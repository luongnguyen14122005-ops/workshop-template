---
title: "Thực nghiệm lỗi và tải"
weight: 7
chapter: false
pre: " <b> 5.7 </b> "
---
## Mục tiêu

So sánh hai cách xử lý đơn hàng trong cùng điều kiện:

| | Đồng bộ (đối chứng) | Bất đồng bộ (đề xuất) |
|---|---|---|
| Luồng | API ghi DynamoDB rồi gửi SES ngay trong request | API chỉ đưa đơn vào SQS; Lambda và Step Functions xử lý sau |
| Chống trùng | Không | Có (DynamoDB) |
| Thử lại, DLQ, cảnh báo | Không | Có |

Chỉ số: phản hồi API (trung bình, p95), đơn hoàn tất, đơn thất bại hoặc mất,
thời gian hoàn tất xử lý, số email dư khi gửi trùng, và thời gian có cảnh báo
khi lỗi.

> **Giới hạn của SES sandbox:** tối đa 1 email/giây. Email gửi đến
> `success@simulator.amazonses.com` (hộp thư giả lập của SES) không tính vào
> hạn mức 200 email/ngày nhưng vẫn bị giới hạn 1 email/giây. Vì vậy mức tải thử
> là 20 và 50 đơn. Đây cũng là một kết quả đáng ghi nhận: nút thắt nằm ở dịch
> vụ gửi email chứ không phải ở kiến trúc.

#### 1. Cập nhật template.yaml

**a) Thêm tham số** vào phần `Parameters`:

```yaml
  ApiKey:
    Type: String
    NoEcho: true
    MinLength: 16
    Description: Khoa bi mat cho header x-api-key
```

**b) Sửa state machine.** Trong `SendCustomerEmail`, thay khối `Retry` để chịu
được việc SES giới hạn tốc độ:

```yaml
            Retry:
              - ErrorEquals: [States.ALL]
                IntervalSeconds: 2
                MaxAttempts: 6
                BackoffRate: 2
                JitterStrategy: FULL
```

Thay toàn bộ trạng thái `MarkDone` để ghi thêm thời điểm hoàn tất và đếm số
email đã gửi:

```yaml
          MarkDone:
            Type: Task
            Resource: arn:aws:states:::dynamodb:updateItem
            Parameters:
              TableName: ${TableName}
              Key:
                orderId:
                  S.$: $.orderId
              UpdateExpression: "SET #s = :v, doneAt = :t ADD emailCount :one"
              ExpressionAttributeNames:
                "#s": status
              ExpressionAttributeValues:
                ":v":
                  S: DONE
                ":t":
                  S.$: $$.State.EnteredTime
                ":one":
                  N: "1"
            End: true
```

> Với `MaxAttempts: 6` và hệ số 2, một đơn bị lỗi vĩnh viễn (ví dụ email chưa
> xác minh) sẽ mất tối đa khoảng 2 phút mới được đánh dấu `FAILED`, thay vì
> 20 đến 30 giây như ở mục 5.5.

**c) Thêm hai hàm API** vào cuối `Resources` (trước `Outputs:`):

```yaml
  SyncOrderFn:
    Type: AWS::Serverless::Function
    Properties:
      Handler: sync-order.handler
      Runtime: nodejs22.x
      CodeUri: src/
      MemorySize: 256
      Timeout: 15
      Environment:
        Variables:
          TABLE_NAME: !Ref OrdersTable
          SENDER_EMAIL: !Ref SenderEmail
          API_KEY: !Ref ApiKey
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref OrdersTable
        - Statement:
            - Effect: Allow
              Action: ses:SendEmail
              Resource: "*"
      Events:
        Post:
          Type: HttpApi
          Properties:
            Path: /sync
            Method: POST

  AsyncOrderFn:
    Type: AWS::Serverless::Function
    Properties:
      Handler: async-order.handler
      Runtime: nodejs22.x
      CodeUri: src/
      MemorySize: 256
      Timeout: 15
      Environment:
        Variables:
          QUEUE_URL: !Ref OrdersQueue
          API_KEY: !Ref ApiKey
      Policies:
        - SQSSendMessagePolicy:
            QueueName: !GetAtt OrdersQueue.QueueName
      Events:
        Post:
          Type: HttpApi
          Properties:
            Path: /async
            Method: POST
```

**d) Thêm đầu ra** vào phần `Outputs`:

```yaml
  ApiUrl:
    Value: !Sub "https://${ServerlessHttpApi}.execute-api.${AWS::Region}.amazonaws.com"
```

#### 2. Viết mã

**src/sync-order.mjs** (phương án đối chứng: không thử lại, không chống trùng):

```javascript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, UpdateCommand } from "@aws-sdk/lib-dynamodb";
import { SESv2Client, SendEmailCommand } from "@aws-sdk/client-sesv2";

const db = DynamoDBDocumentClient.from(new DynamoDBClient({}));
const ses = new SESv2Client({});
const ID = /^[A-Za-z0-9_-]{1,64}$/;

export const handler = async (event) => {
  if (event.headers?.["x-api-key"] !== process.env.API_KEY)
    return { statusCode: 403, body: "forbidden" };

  let order;
  try { order = JSON.parse(event.body); }
  catch { return { statusCode: 400, body: "invalid json" }; }
  if (typeof order?.orderId !== "string" || !ID.test(order.orderId) || !order.customerEmail)
    return { statusCode: 400, body: "invalid order" };

  const key = { orderId: order.orderId };
  try {
    await db.send(new UpdateCommand({
      TableName: process.env.TABLE_NAME,
      Key: key,
      UpdateExpression: "SET #s = :r, createdAt = if_not_exists(createdAt, :c)",
      ExpressionAttributeNames: { "#s": "status" },
      ExpressionAttributeValues: { ":r": "RECEIVED", ":c": Date.now() },
    }));

    await ses.send(new SendEmailCommand({
      FromEmailAddress: process.env.SENDER_EMAIL,
      Destination: { ToAddresses: [order.customerEmail] },
      Content: { Simple: {
        Subject: { Data: `Xac nhan don hang ${order.orderId}` },
        Body: { Text: { Data: `Cam on ban. Don hang ${order.orderId} da duoc ghi nhan.` } },
      } },
    }));

    await db.send(new UpdateCommand({
      TableName: process.env.TABLE_NAME,
      Key: key,
      UpdateExpression: "SET #s = :d, doneAt = :t ADD emailCount :one",
      ExpressionAttributeNames: { "#s": "status" },
      ExpressionAttributeValues: { ":d": "DONE", ":t": new Date().toISOString(), ":one": 1 },
    }));
    return { statusCode: 200, body: JSON.stringify({ orderId: order.orderId, status: "DONE" }) };
  } catch (e) {
    console.error("Loi xu ly don", order.orderId, e.name);
    return { statusCode: 500, body: "error" };
  }
};
```

**src/async-order.mjs** (phương án đề xuất: nhận đơn rồi trả lời ngay):

```javascript
import { SQSClient, SendMessageCommand } from "@aws-sdk/client-sqs";

const sqs = new SQSClient({});
const ID = /^[A-Za-z0-9_-]{1,64}$/;

export const handler = async (event) => {
  if (event.headers?.["x-api-key"] !== process.env.API_KEY)
    return { statusCode: 403, body: "forbidden" };

  let order;
  try { order = JSON.parse(event.body); }
  catch { return { statusCode: 400, body: "invalid json" }; }
  if (typeof order?.orderId !== "string" || !ID.test(order.orderId) || !order.customerEmail)
    return { statusCode: 400, body: "invalid order" };

  await sqs.send(new SendMessageCommand({
    QueueUrl: process.env.QUEUE_URL,
    MessageBody: JSON.stringify({
      orderId: order.orderId,
      customerEmail: order.customerEmail,
      acceptedAt: Date.now(),
    }),
  }));
  return { statusCode: 202, body: JSON.stringify({ orderId: order.orderId, status: "ACCEPTED" }) };
};
```

**Sửa một dòng trong `src/index.mjs`** để thời gian hoàn tất được tính từ lúc
API nhận đơn:

```javascript
// truoc
Item: { orderId: order.orderId, status: "RECEIVED", createdAt: Date.now() },
// sau
Item: { orderId: order.orderId, status: "RECEIVED", createdAt: order.acceptedAt ?? Date.now() },
```

#### 3. Triển khai

Tạo một khóa bí mật và triển khai lại (lần này cần `--guided` vì có tham số mới):

```powershell
[guid]::NewGuid().ToString("N")     # copy chuoi nay lam ApiKey
sam validate
sam build
sam deploy --guided
```

Giữ nguyên các giá trị cũ, nhập `ApiKey` khi được hỏi. Lấy các biến dùng chung:

```powershell
$api   = aws cloudformation describe-stacks --stack-name order-system --query "Stacks[0].Outputs[?OutputKey=='ApiUrl'].OutputValue" --output text
$table = aws cloudformation describe-stacks --stack-name order-system --query "Stacks[0].Outputs[?OutputKey=='TableName'].OutputValue" --output text
$sm    = aws cloudformation describe-stacks --stack-name order-system --query "Stacks[0].Outputs[?OutputKey=='StateMachineArn'].OutputValue" --output text
$env:API_KEY = "<ApiKey-ban-vua-tao>"
$env:AWS_REGION = "ap-southeast-1"
```

![Kết quả deploy](/images/5-Workshop/5.7-Experiments/deploy-result.png)

> Không đưa `ApiKey` và `samconfig.toml` lên GitHub. Hai đường API này công
> khai trên Internet (chỉ được bảo vệ bằng khóa), nên sẽ bị xóa ở mục 5.8.

#### 4. Chuẩn bị công cụ đo

Trong thư mục `order-system`:

```powershell
mkdir tools
cd tools
npm init -y
npm install @aws-sdk/client-dynamodb @aws-sdk/lib-dynamodb
```

**tools/loadtest.mjs** gửi N đơn với C yêu cầu đồng thời và đo thời gian phản hồi:

```javascript
// Cach dung: node loadtest.mjs <url> <tien-to> <so-don> <dong-thoi>
const [url, prefix, n, c] = process.argv.slice(2);
const N = Number(n), C = Number(c);
const key = process.env.API_KEY;
const email = process.env.CUSTOMER_EMAIL ?? "success@simulator.amazonses.com";
if (!url || !key) { console.error("Thieu url hoac API_KEY"); process.exit(1); }

const lat = [];
let ok = 0, fail = 0, next = 1;

async function worker() {
  while (true) {
    const i = next++;
    if (i > N) return;
    const body = JSON.stringify({ orderId: `${prefix}-${i}`, customerEmail: email });
    const t0 = performance.now();
    try {
      const r = await fetch(url, {
        method: "POST",
        headers: { "content-type": "application/json", "x-api-key": key },
        body,
      });
      await r.text();
      lat.push(performance.now() - t0);
      r.ok ? ok++ : fail++;
    } catch { fail++; }
  }
}

await Promise.all(Array.from({ length: C }, worker));
lat.sort((a, b) => a - b);
const avg = lat.length ? lat.reduce((s, x) => s + x, 0) / lat.length : NaN;
const p95 = lat.length ? lat[Math.min(lat.length - 1, Math.ceil(lat.length * 0.95) - 1)] : NaN;
console.log(`${url} | gui=${N} dong thoi=${C} | thanh cong=${ok} loi=${fail} | API avg=${avg.toFixed(0)}ms p95=${p95.toFixed(0)}ms`);
```

**tools/report.mjs** đọc DynamoDB để tổng hợp kết quả xử lý theo tiền tố:

```javascript
// Cach dung: node report.mjs <ten-bang> <tien-to> <so-don-da-gui>
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, ScanCommand } from "@aws-sdk/lib-dynamodb";

const [table, prefix, n] = process.argv.slice(2);
const db = DynamoDBDocumentClient.from(new DynamoDBClient({}));

const items = [];
let key;
do {
  const r = await db.send(new ScanCommand({
    TableName: table,
    ConsistentRead: true,
    FilterExpression: "begins_with(orderId, :p)",
    ExpressionAttributeValues: { ":p": `${prefix}-` },
    ExclusiveStartKey: key,
  }));
  items.push(...(r.Items ?? []));
  key = r.LastEvaluatedKey;
} while (key);

const status = {};
for (const i of items) status[i.status] = (status[i.status] ?? 0) + 1;
const extra = items.reduce((s, i) => s + Math.max(0, (i.emailCount ?? 0) - 1), 0);
const d = items.filter(i => i.doneAt && i.createdAt)
  .map(i => Date.parse(i.doneAt) - i.createdAt).sort((a, b) => a - b);
const avg = d.length ? Math.round(d.reduce((s, x) => s + x, 0) / d.length) : null;
const p95 = d.length ? d[Math.min(d.length - 1, Math.ceil(d.length * 0.95) - 1)] : null;

console.log(`Tien to: ${prefix} | gui: ${n} | tim thay: ${items.length} | mat: ${Number(n) - items.length}`);
console.log("Trang thai:", status);
console.log("Email du (gui trung):", extra);
console.log(`Thoi gian hoan tat (ms): avg=${avg} p95=${p95} (so don: ${d.length})`);
if (items.length <= 10)
  console.table(items.map(i => ({ orderId: i.orderId, status: i.status, emailCount: i.emailCount ?? 0 })));
```

## Thí nghiệm 1: độ trễ và tải

Chạy khởi động nóng (tránh tính cả thời gian khởi động lạnh của Lambda), rồi đo
20 đơn. Giữa các lần chạy phải chờ để SES kịp "nguội" (giới hạn 1 email/giây):

```powershell
$env:CUSTOMER_EMAIL = "success@simulator.amazonses.com"
node loadtest.mjs "$api/sync"  warm-s 3 1
node loadtest.mjs "$api/async" warm-a 3 1
Start-Sleep -Seconds 60

node loadtest.mjs "$api/sync" s20 20 10
Start-Sleep -Seconds 60
node loadtest.mjs "$api/async" a20 20 10
Start-Sleep -Seconds 120

node report.mjs $table s20 20
node report.mjs $table a20 20
```

Lặp lại với 50 đơn (`s50`, `a50`, chờ khoảng 4 phút trước khi chạy `report`).
Nên chạy mỗi mức tải ít nhất 3 lần và lấy trung bình.

![Kết quả loadtest](/images/5-Workshop/5.7-Experiments/loadtest-result.png)
![Kết quả report](/images/5-Workshop/5.7-Experiments/report-result.png)

Cần đọc ra:

- `thanh cong` và `loi` của từng API: đường đồng bộ có bị lỗi khi SES giới hạn tốc độ không?
- `Trang thai` trong report: bao nhiêu đơn `DONE`, `FAILED`, `RECEIVED` (bị kẹt), và bao nhiêu đơn `mat`.
- `API avg/p95` (khách thấy) so với `Thoi gian hoan tat` (xử lý xong).

## Thí nghiệm 2: gửi trùng đơn

Gửi cùng một `orderId` 5 lần vào mỗi đường:

```powershell
$h = @{ "x-api-key" = $env:API_KEY }
$body = '{"orderId":"dup-s","customerEmail":"success@simulator.amazonses.com"}'
1..5 | ForEach-Object {
  Invoke-RestMethod -Method Post -Uri "$api/sync" -Headers $h -ContentType "application/json" -Body $body
  Start-Sleep -Seconds 1
}

$body = '{"orderId":"dup-a","customerEmail":"success@simulator.amazonses.com"}'
1..5 | ForEach-Object {
  Invoke-RestMethod -Method Post -Uri "$api/async" -Headers $h -ContentType "application/json" -Body $body
  Start-Sleep -Seconds 1
}

Start-Sleep -Seconds 40
node ..\tools\report.mjs $table dup 2
```

(Nếu đang ở thư mục `tools` thì dùng `node report.mjs $table dup 2`.) Bảng kết
quả cho thấy `emailCount` của từng đơn. Theo thiết kế, đường đồng bộ gửi email
mỗi lần nhận yêu cầu, còn đường bất đồng bộ chỉ xử lý đơn đầu tiên.

![Kết quả gửi trùng](/images/5-Workshop/5.7-Experiments/duplicate-result.png)

## Thí nghiệm 3: lỗi và cảnh báo

Dùng địa chỉ email chưa xác minh để SES từ chối, gửi 5 đơn mỗi đường:

```powershell
$env:CUSTOMER_EMAIL = "chua-xac-minh@example.com"
node loadtest.mjs "$api/sync"  f-s 5 1
node loadtest.mjs "$api/async" f-a 5 1
```

Chờ khoảng 3 phút để các execution thất bại hết các lần thử lại, rồi:

```powershell
node report.mjs $table f-s 5
node report.mjs $table f-a 5
```

Quan sát:

- Đường đồng bộ: khách nhận lỗi 500 ngay, đơn nằm lại ở `RECEIVED`, **không có cảnh báo nào** (đối chứng được cố ý để đơn giản, không có giám sát).
- Đường bất đồng bộ: API vẫn trả 202, đơn được đánh dấu `FAILED`, admin nhận email cảnh báo và alarm CloudWatch kích hoạt.

**Đo thời gian có cảnh báo.** Mốc của cảnh báo trực tiếp (SNS) là thời điểm
execution kết thúc:

```powershell
aws stepfunctions list-executions --state-machine-arn $sm --status-filter FAILED --max-results 5 --query "executions[].[name,startDate,stopDate]" --output table
```

Mốc của alarm CloudWatch:

```powershell
aws cloudwatch describe-alarm-history --alarm-name order-sfn-failed --history-item-type StateUpdate --max-items 5 --query "AlarmHistoryItems[].[Timestamp,HistorySummary]" --output table
```

Lấy hiệu giữa `stopDate` và `Timestamp` để biết alarm trễ hơn cảnh báo trực tiếp
bao lâu (chú ý hai bảng có thể hiển thị múi giờ khác nhau).

**Đo đường DLQ** (tin nhắn lỗi định dạng): ghi lại thời điểm gửi rồi so với
lịch sử alarm `order-dlq-has-messages`:

```powershell
(Get-Date).ToUniversalTime().ToString("o")
aws sqs send-message --queue-url (aws cloudformation describe-stacks --stack-name order-system --query "Stacks[0].Outputs[?OutputKey=='QueueUrl'].OutputValue" --output text) --message-body "khong-phai-json"
aws cloudwatch describe-alarm-history --alarm-name order-dlq-has-messages --history-item-type StateUpdate --max-items 3 --query "AlarmHistoryItems[].[Timestamp,HistorySummary]" --output table
```

![Execution thất bại](/images/5-Workshop/5.7-Experiments/failed-execution.png)
![Lịch sử alarm](/images/5-Workshop/5.7-Experiments/alarm-history.png)

## Tổng hợp kết quả

Điền số đo thật của bạn vào bảng (với mỗi mức tải 20 và 50 đơn):

| Chỉ số | Đồng bộ | Bất đồng bộ |
|---|---|---|
| Phản hồi API trung bình (ms) | | |
| Phản hồi API p95 (ms) | | |
| Yêu cầu bị lỗi tại API | | |
| Đơn hoàn tất (`DONE`) | | |
| Đơn `FAILED` hoặc kẹt `RECEIVED` | | |
| Đơn mất (không có bản ghi) | | |
| Thời gian hoàn tất trung bình / p95 (ms) | | |
| Email dư khi gửi trùng 5 lần | | |
| Lỗi có được cảnh báo không, sau bao lâu | | |

Chi phí: sau 1 đến 2 ngày, vào **Billing → Cost Explorer** lọc theo dịch vụ để
ghi nhận chi phí thực tế của toàn bộ thí nghiệm.

## Nhận xét và hạn chế

- Hãy nhận xét theo đúng số liệu bạn đo được, kể cả khi kiến trúc bất đồng bộ không thắng ở mọi chỉ số (ví dụ thời gian hoàn tất thường dài hơn vì xử lý chạy nền).
- SES sandbox (1 email/giây) là nút thắt chung của cả hai phương án. Để thử tải lớn hơn (100 đến 200 đơn trở lên), cần xin chuyển SES ra khỏi sandbox, hoặc tăng `MaxAttempts` của bước gửi email.
- Đơn `FAILED` hiện chưa có cơ chế tự xử lý lại: do `orderId` đã được ghi sổ và dùng làm tên execution, gửi lại cùng mã sẽ bị coi là trùng. Đây là hướng cải tiến (thêm quy trình xử lý lại thủ công hoặc dùng mã thử lại riêng).
- Mỗi mức tải chỉ chạy vài lần, kết quả bị ảnh hưởng bởi thời điểm chạy và khởi động lạnh của Lambda, nên chỉ mang tính tham khảo.