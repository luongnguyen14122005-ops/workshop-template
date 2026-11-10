---
title: "Chống xử lý trùng"
date: "2026-11-06"
weight: 4
chapter: false
pre: " <b> 5.4 </b> "
---
## Mục tiêu

Hàng đợi SQS đảm bảo giao tin nhắn **ít nhất một lần** (at-least-once), nghĩa là
đôi khi cùng một đơn hàng có thể được giao hai lần. Nếu không xử lý, khách sẽ
nhận email trùng. Ta dùng **DynamoDB** làm "sổ ghi": mỗi `orderId` chỉ được xử
lý một lần. Kỹ thuật này gọi là **idempotency**.

#### 1. Cập nhật template.yaml

Thêm bảng DynamoDB và hàm Lambda vào `template.yaml`. Giữ nguyên `OrdersDLQ`
và `OrdersQueue`, thay phần còn lại như sau:

```yaml
  OrdersTable:
    Type: AWS::DynamoDB::Table
    Properties:
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: orderId
          AttributeType: S
      KeySchema:
        - AttributeName: orderId
          KeyType: HASH

  ProcessOrderFn:
    Type: AWS::Serverless::Function
    Properties:
      Handler: index.handler
      Runtime: nodejs22.x
      CodeUri: src/
      Timeout: 10
      Environment:
        Variables:
          TABLE_NAME: !Ref OrdersTable
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref OrdersTable
      Events:
        FromSqs:
          Type: SQS
          Properties:
            Queue: !GetAtt OrdersQueue.Arn
            BatchSize: 1

Outputs:
  QueueUrl:
    Value: !Ref OrdersQueue
  DlqUrl:
    Value: !Ref OrdersDLQ
  TableName:
    Value: !Ref OrdersTable
```

> `VisibilityTimeout` của hàng đợi (60 giây) lớn hơn nhiều lần `Timeout` của
> Lambda (10 giây). Đây là cấu hình khuyến nghị khi Lambda đọc từ SQS.

#### 2. Viết mã Lambda

Tạo file `src/index.mjs`:

```javascript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, PutCommand } from "@aws-sdk/lib-dynamodb";

const db = DynamoDBDocumentClient.from(new DynamoDBClient({}));

export const handler = async (event) => {
  for (const record of event.Records) {
    const order = JSON.parse(record.body); // JSON sai se gay loi -> retry -> DLQ

    try {
      await db.send(new PutCommand({
        TableName: process.env.TABLE_NAME,
        Item: {
          orderId: order.orderId,
          status: "PROCESSING",
          createdAt: Date.now(),
        },
        // Chi ghi neu orderId chua ton tai
        ConditionExpression: "attribute_not_exists(orderId)",
      }));
    } catch (e) {
      if (e.name === "ConditionalCheckFailedException") {
        console.log("Don trung, bo qua:", order.orderId);
        continue;
      }
      throw e;
    }

    console.log("Da xu ly:", order.orderId);
  }
};
```

Điểm mấu chốt là `ConditionExpression`: DynamoDB chỉ cho ghi nếu `orderId`
chưa tồn tại, nên lần ghi thứ hai sẽ bị từ chối và Lambda bỏ qua đơn đó.

> Ở mục 5.5 ta sẽ bổ sung trạng thái đơn (`PROCESSING` → `DONE`) để xử lý
> trường hợp đã ghi sổ nhưng bước gửi email thất bại.

#### 3. Triển khai lại

```powershell
sam build
sam deploy
```

![Deploy lần hai](/images/5-Workshop/5.4-Idempotency/deploy-result.png)

#### 4. Kiểm thử

Lấy các biến cần dùng:

```powershell
$q = aws cloudformation describe-stacks --stack-name order-system --query "Stacks[0].Outputs[?OutputKey=='QueueUrl'].OutputValue" --output text
$dlq = aws cloudformation describe-stacks --stack-name order-system --query "Stacks[0].Outputs[?OutputKey=='DlqUrl'].OutputValue" --output text
$table = aws cloudformation describe-stacks --stack-name order-system --query "Stacks[0].Outputs[?OutputKey=='TableName'].OutputValue" --output text
```

Tạo file nội dung đơn hàng:

```powershell
'{"orderId":"A001"}' | Out-File -Encoding ascii msg1.json
```

**Trường hợp 1: đơn hợp lệ**

```powershell
aws sqs send-message --queue-url $q --message-body file://msg1.json
aws dynamodb scan --table-name $table
```

Bảng DynamoDB có một dòng `A001` với `status = PROCESSING`.

![Dữ liệu trong DynamoDB](/images/5-Workshop/5.4-Idempotency/dynamodb-item.png)

**Trường hợp 2: gửi trùng cùng đơn**

```powershell
aws sqs send-message --queue-url $q --message-body file://msg1.json
sam logs --stack-name order-system --name ProcessOrderFn --tail
```

Log hiển thị dòng `Don trung, bo qua: A001`. Nhấn `Ctrl+C` để thoát chế độ theo dõi log.
Bảng DynamoDB vẫn chỉ có một dòng `A001`.

![Log báo đơn trùng](/images/5-Workshop/5.4-Idempotency/duplicate-log.png)

**Trường hợp 3: tin nhắn lỗi đi vào DLQ**

```powershell
aws sqs send-message --queue-url $q --message-body "khong-phai-json"
```

Lambda lỗi ở `JSON.parse`, SQS thử lại sau mỗi 60 giây. Sau khoảng 3 đến 4 phút,
tin nhắn xuất hiện trong DLQ (Console → DLQ → **Poll for messages**).

![Tin nhắn lỗi trong DLQ](/images/5-Workshop/5.4-Idempotency/dlq-message.png)

## Kiểm tra kết quả

| Trường hợp | Kết quả mong đợi |
|---|---|
| Đơn hợp lệ | Có một dòng trong DynamoDB |
| Gửi trùng | Log báo trùng, không thêm dòng mới |
| Tin nhắn không phải JSON | Sau vài lần thử, tin nhắn nằm trong DLQ |

## Xử lý sự cố

- **Lambda không chạy**: vào Console → Lambda → hàm `ProcessOrderFn` → tab **Configuration → Triggers**, kiểm tra trigger SQS đang ở trạng thái *Enabled*.
- **Lỗi quyền DynamoDB**: xem log, kiểm tra phần `Policies` trong template đã dùng đúng `!Ref OrdersTable`.
- **`sam logs` không hiện gì**: chờ vài giây rồi gửi lại tin nhắn, log có độ trễ nhỏ.
