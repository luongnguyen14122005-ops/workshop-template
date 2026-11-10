---
title: "Hàng đợi và DLQ"
weight: 3
chapter: false
pre: " <b> 5.3 </b> "
---
## Mục tiêu

Tạo hàng đợi chính nhận đơn hàng và một hàng đợi lỗi (**Dead-Letter Queue, DLQ**)
để giữ lại các tin nhắn xử lý thất bại, thay vì để chúng bị mất.

Ba khái niệm cần nhớ:

- **Visibility timeout**: thời gian tin nhắn bị ẩn sau khi một consumer nhận nó. Nếu consumer không xóa tin nhắn trong thời gian này, tin nhắn hiện lại để được thử lần nữa.
- **maxReceiveCount**: số lần tin nhắn được nhận mà không bị xóa trước khi chuyển sang DLQ.
- **DLQ**: hàng đợi chứa các tin nhắn lỗi để kiểm tra và xử lý lại.

#### 1. Tạo file template.yaml

Trong thư mục `order-system`, tạo file `template.yaml`:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: Order workshop - SQS va DLQ

Resources:
  OrdersDLQ:
    Type: AWS::SQS::Queue
    Properties:
      MessageRetentionPeriod: 1209600   # giu tin nhan loi 14 ngay

  OrdersQueue:
    Type: AWS::SQS::Queue
    Properties:
      VisibilityTimeout: 60
      RedrivePolicy:
        deadLetterTargetArn: !GetAtt OrdersDLQ.Arn
        maxReceiveCount: 3

Outputs:
  QueueUrl:
    Value: !Ref OrdersQueue
  DlqUrl:
    Value: !Ref OrdersDLQ
```

#### 2. Triển khai

```powershell
sam build
sam deploy --guided
```

Trả lời các câu hỏi như sau:

| Câu hỏi | Trả lời |
|---|---|
| Stack Name | `order-system` |
| AWS Region | `ap-southeast-1` |
| Confirm changes before deploy | `N` |
| Allow SAM CLI IAM role creation | `Y` |
| Disable rollback | `N` |
| Save arguments to samconfig.toml | `Y` |

Khi hoàn tất, SAM in ra `QueueUrl` và `DlqUrl`.

![Kết quả deploy](/images/5-Workshop/5.3-SQS-DLQ/deploy-result.png)

#### 3. Kiểm tra trên Console

1. Vào **Amazon SQS** → **Queues**.
2. Bạn thấy hai hàng đợi có tên bắt đầu bằng `order-system-`.
3. Mở hàng đợi chính → tab **Dead-letter queue**: hàng đợi lỗi đã được gắn, **Maximum receives = 3**.

![Hàng đợi SQS](/images/5-Workshop/5.3-SQS-DLQ/queues.png)

#### 4. Thử để tin nhắn rơi vào DLQ

Lấy địa chỉ hai hàng đợi vào biến:

```powershell
$q = aws cloudformation describe-stacks --stack-name order-system --query "Stacks[0].Outputs[?OutputKey=='QueueUrl'].OutputValue" --output text
$dlq = aws cloudformation describe-stacks --stack-name order-system --query "Stacks[0].Outputs[?OutputKey=='DlqUrl'].OutputValue" --output text
```

Gửi một tin nhắn thử:

```powershell
aws sqs send-message --queue-url $q --message-body "test-dlq"
```

Nhận tin nhắn nhiều lần mà **không xóa** (tham số `--visibility-timeout 0` làm tin nhắn hiện lại ngay):

```powershell
1..4 | ForEach-Object {
  aws sqs receive-message --queue-url $q --visibility-timeout 0 --wait-time-seconds 2
  Start-Sleep -Seconds 1
}
```

Ba lần đầu trả về tin nhắn, lần thứ tư không còn gì. Đợi khoảng một phút rồi kiểm tra DLQ:

```powershell
aws sqs get-queue-attributes --queue-url $dlq --attribute-names ApproximateNumberOfMessages
```

Hoặc vào Console → hàng đợi DLQ → **Send and receive messages** → **Poll for messages**.

![Tin nhắn trong DLQ](/images/5-Workshop/5.3-SQS-DLQ/dlq-message.png)

> Các số liệu của SQS là xấp xỉ và có thể trễ vài chục giây.

## Kiểm tra kết quả

- Có hai hàng đợi, hàng chính trỏ tới DLQ với `maxReceiveCount = 3`.
- Tin nhắn bị nhận quá 3 lần mà không xóa nằm trong DLQ.