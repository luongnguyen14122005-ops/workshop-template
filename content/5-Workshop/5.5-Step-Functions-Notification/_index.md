---
title: "Điều phối và thông báo"
weight: 5
chapter: false
pre: " <b> 5.5 </b> "
---
## Mục tiêu

Thay vì để một hàm Lambda làm hết mọi việc, ta dùng **AWS Step Functions** để
điều phối quy trình xử lý đơn hàng. Mỗi bước có cơ chế **thử lại (Retry)** và
**bắt lỗi (Catch)** rõ ràng, trạng thái từng đơn được ghi lại, và quản trị viên
được báo ngay qua **Amazon SNS** khi có đơn thất bại.

Quy trình:

1. Lambda nhận tin nhắn từ SQS, ghi sổ DynamoDB (`RECEIVED`) và khởi động state machine.
2. State machine gửi email xác nhận cho khách bằng **Amazon SES** (thử lại tối đa 3 lần).
3. Thành công: cập nhật trạng thái `DONE`.
4. Thất bại: cập nhật `FAILED`, gửi cảnh báo qua SNS, kết thúc ở trạng thái lỗi.

![Sơ đồ quy trình](/images/5-Workshop/5.5-Orchestration/workflow.png)

#### 1. Thay toàn bộ template.yaml

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: Order workshop - xu ly don hang event-driven

Parameters:
  SenderEmail:
    Type: String
    Description: Email da xac minh trong SES, dung lam nguoi gui
  AdminEmail:
    Type: String
    Description: Email nhan canh bao khi xu ly don that bai

Resources:
  OrdersDLQ:
    Type: AWS::SQS::Queue
    Properties:
      MessageRetentionPeriod: 1209600

  OrdersQueue:
    Type: AWS::SQS::Queue
    Properties:
      VisibilityTimeout: 60
      RedrivePolicy:
        deadLetterTargetArn: !GetAtt OrdersDLQ.Arn
        maxReceiveCount: 3

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

  OrderTopic:
    Type: AWS::SNS::Topic
    Properties:
      Subscription:
        - Protocol: email
          Endpoint: !Ref AdminEmail

  OrderStateMachine:
    Type: AWS::Serverless::StateMachine
    Properties:
      DefinitionSubstitutions:
        TableName: !Ref OrdersTable
        TopicArn: !Ref OrderTopic
        SenderEmail: !Ref SenderEmail
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref OrdersTable
        - SNSPublishMessagePolicy:
            TopicName: !GetAtt OrderTopic.TopicName
        - Statement:
            - Effect: Allow
              Action: ses:SendEmail
              Resource: "*"
      Definition:
        StartAt: SendCustomerEmail
        States:
          SendCustomerEmail:
            Type: Task
            Resource: arn:aws:states:::aws-sdk:sesv2:sendEmail
            Parameters:
              FromEmailAddress: ${SenderEmail}
              Destination:
                ToAddresses.$: States.Array($.customerEmail)
              Content:
                Simple:
                  Subject:
                    Data.$: "States.Format('Xac nhan don hang {}', $.orderId)"
                  Body:
                    Text:
                      Data.$: "States.Format('Cam on ban. Don hang {} da duoc ghi nhan.', $.orderId)"
            ResultPath: null
            Retry:
              - ErrorEquals: [States.ALL]
                IntervalSeconds: 2
                MaxAttempts: 3
                BackoffRate: 2
            Catch:
              - ErrorEquals: [States.ALL]
                ResultPath: $.error
                Next: MarkFailed
            Next: MarkDone

          MarkDone:
            Type: Task
            Resource: arn:aws:states:::dynamodb:updateItem
            Parameters:
              TableName: ${TableName}
              Key:
                orderId:
                  S.$: $.orderId
              UpdateExpression: "SET #s = :v"
              ExpressionAttributeNames:
                "#s": status
              ExpressionAttributeValues:
                ":v":
                  S: DONE
            End: true

          MarkFailed:
            Type: Task
            Resource: arn:aws:states:::dynamodb:updateItem
            Parameters:
              TableName: ${TableName}
              Key:
                orderId:
                  S.$: $.orderId
              UpdateExpression: "SET #s = :v"
              ExpressionAttributeNames:
                "#s": status
              ExpressionAttributeValues:
                ":v":
                  S: FAILED
            ResultPath: null
            Next: NotifyAdminFailure

          NotifyAdminFailure:
            Type: Task
            Resource: arn:aws:states:::sns:publish
            Parameters:
              TopicArn: ${TopicArn}
              Subject: Canh bao - xu ly don hang that bai
              Message.$: "States.Format('Don hang {} xu ly that bai. Kiem tra execution trong Step Functions.', $.orderId)"
            ResultPath: null
            Next: OrderFailed

          OrderFailed:
            Type: Fail
            Error: OrderProcessingFailed
            Cause: Khong gui duoc email xac nhan

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
          STATE_MACHINE_ARN: !Ref OrderStateMachine
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref OrdersTable
        - StepFunctionsExecutionPolicy:
            StateMachineName: !GetAtt OrderStateMachine.Name
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
  StateMachineArn:
    Value: !Ref OrderStateMachine
```

> Quyền `ses:SendEmail` để `Resource: "*"` cho đơn giản trong môi trường thử
> nghiệm. Với môi trường thật, hãy giới hạn theo identity của người gửi.

#### 2. Thay toàn bộ src/index.mjs

```javascript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import {
  DynamoDBDocumentClient,
  PutCommand,
  GetCommand,
} from "@aws-sdk/lib-dynamodb";
import { SFNClient, StartExecutionCommand } from "@aws-sdk/client-sfn";

const db = DynamoDBDocumentClient.from(new DynamoDBClient({}));
const sfn = new SFNClient({});

export const handler = async (event) => {
  for (const record of event.Records) {
    const order = JSON.parse(record.body); // JSON sai -> loi -> retry -> DLQ

    // Buoc 1: ghi so, moi orderId chi duoc ghi mot lan
    try {
      await db.send(new PutCommand({
        TableName: process.env.TABLE_NAME,
        Item: { orderId: order.orderId, status: "RECEIVED", createdAt: Date.now() },
        ConditionExpression: "attribute_not_exists(orderId)",
      }));
    } catch (e) {
      if (e.name !== "ConditionalCheckFailedException") throw e;

      const { Item } = await db.send(new GetCommand({
        TableName: process.env.TABLE_NAME,
        Key: { orderId: order.orderId },
      }));
      if (Item?.status !== "RECEIVED") {
        console.log("Don trung, bo qua:", order.orderId);
        continue;
      }
      // Con o trang thai RECEIVED: lan truoc chua khoi dong duoc quy trinh -> thu lai
    }

    // Buoc 2: khoi dong quy trinh. Ten execution = orderId nen khong the chay trung.
    try {
      await sfn.send(new StartExecutionCommand({
        stateMachineArn: process.env.STATE_MACHINE_ARN,
        name: order.orderId,
        input: JSON.stringify(order),
      }));
      console.log("Da khoi dong quy trinh:", order.orderId);
    } catch (e) {
      if (e.name === "ExecutionAlreadyExists") {
        console.log("Quy trinh da ton tai, bo qua:", order.orderId);
        continue;
      }
      throw e;
    }
  }
};
```

> `orderId` dùng làm tên execution nên chỉ nên gồm chữ, số, dấu `-` và `_`
> (tối đa 80 ký tự).

#### 3. Triển khai

```powershell
sam build
sam deploy --guided
```

Giữ các câu trả lời như lần trước. Khi được hỏi hai tham số, nhập:

| Tham số | Giá trị |
|---|---|
| SenderEmail | Email đã xác minh ở mục 5.2 |
| AdminEmail | Email nhận cảnh báo (có thể là cùng email trên) |

![Kết quả deploy](/images/5-Workshop/5.5-Orchestration/deploy-result.png)

**Xác nhận đăng ký SNS:** AWS gửi một email tiêu đề *AWS Notification -
Subscription Confirmation* đến `AdminEmail`. Mở email và bấm **Confirm
subscription**, nếu không bạn sẽ không nhận được cảnh báo.

![Xác nhận SNS](/images/5-Workshop/5.5-Orchestration/sns-confirm.png)

#### 4. Kiểm thử

Lấy các biến cần dùng:

```powershell
$q = aws cloudformation describe-stacks --stack-name order-system --query "Stacks[0].Outputs[?OutputKey=='QueueUrl'].OutputValue" --output text
$table = aws cloudformation describe-stacks --stack-name order-system --query "Stacks[0].Outputs[?OutputKey=='TableName'].OutputValue" --output text
$email = "email-da-xac-minh-cua-ban@example.com"
```

**Trường hợp 1: đơn thành công**

```powershell
@{ orderId = "B001"; customerEmail = $email } | ConvertTo-Json -Compress | Out-File -Encoding ascii msg-b001.json
aws sqs send-message --queue-url $q --message-body file://msg-b001.json
```

Kiểm tra:

1. Hộp thư nhận được email *Xac nhan don hang B001*.
2. Vào **Step Functions** → **State machines** → chọn state machine → execution tên `B001` → trạng thái **Succeeded**, mở **Graph view** để thấy các bước màu xanh.
3. `aws dynamodb scan --table-name $table` cho thấy `B001` có `status = DONE`.

![Execution thành công](/images/5-Workshop/5.5-Orchestration/execution-success.png)

**Trường hợp 2: gửi trùng đơn**

```powershell
aws sqs send-message --queue-url $q --message-body file://msg-b001.json
sam logs --stack-name order-system --name ProcessOrderFn --tail
```

Log hiển thị `Don trung, bo qua: B001`, không có email thứ hai.
Nhấn `Ctrl+C` để thoát.

**Trường hợp 3: lỗi gửi email**

Dùng một địa chỉ **chưa xác minh** trong SES. Vì tài khoản đang ở chế độ
sandbox, SES sẽ từ chối:

```powershell
@{ orderId = "B002"; customerEmail = "chua-xac-minh@example.com" } | ConvertTo-Json -Compress | Out-File -Encoding ascii msg-b002.json
aws sqs send-message --queue-url $q --message-body file://msg-b002.json
```

Đợi khoảng 20 đến 30 giây (3 lần thử lại cách nhau 2, 4, 8 giây), rồi kiểm tra:

1. Execution `B002` ở trạng thái **Failed**. Trong Graph view, bước `SendCustomerEmail` báo lỗi, luồng đi qua `MarkFailed` và `NotifyAdminFailure`.
2. Email cảnh báo *Canh bao - xu ly don hang that bai* đến `AdminEmail`.
3. DynamoDB: `B002` có `status = FAILED`.

![Execution thất bại](/images/5-Workshop/5.5-Orchestration/execution-failed.png)
![Email cảnh báo](/images/5-Workshop/5.5-Orchestration/alert-email.png)

## Kiểm tra kết quả

| Trường hợp | Kết quả mong đợi |
|---|---|
| Đơn hợp lệ | Email đến khách, execution Succeeded, trạng thái `DONE` |
| Gửi trùng | Không có email thứ hai, log báo trùng |
| Email không gửi được | Thử lại 3 lần, execution Failed, trạng thái `FAILED`, admin nhận cảnh báo |

## Xử lý sự cố

- **Không nhận được email xác nhận đơn**: kiểm tra `SenderEmail` và email khách đều ở trạng thái *Verified* trong SES, và xem thư mục spam.
- **Không nhận được cảnh báo**: kiểm tra đã bấm *Confirm subscription* ở SNS chưa (Console → SNS → Subscriptions, trạng thái phải là *Confirmed*).
- **Không có execution nào**: xem log của `ProcessOrderFn`; nếu có lỗi quyền, kiểm tra phần `Policies` trong template.
- **Execution báo lỗi `AccessDenied` ở SES**: kiểm tra role của state machine có `ses:SendEmail`.