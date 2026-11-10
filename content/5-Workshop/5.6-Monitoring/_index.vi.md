---
title: "Giám sát"
date: "2026-11-06"
weight: 6
chapter: false
pre: " <b> 5.6 </b> "
---
## Mục tiêu

Một hệ thống chịu lỗi chỉ có giá trị khi người vận hành **biết** lỗi đang xảy
ra. Ở mục này ta thêm:

- Ba **CloudWatch Alarm**: có tin nhắn trong DLQ, có execution thất bại, Lambda báo lỗi.
- Một **Dashboard** hiển thị toàn bộ luồng xử lý trên một màn hình.

Cảnh báo được gửi qua cùng SNS topic ở mục 5.5, nên email nhận cảnh báo là email
admin đã xác nhận trước đó.

#### 1. Thêm alarm vào template.yaml

Chèn khối sau vào cuối phần `Resources`, ngay trước dòng `Outputs:`
(giữ đúng thụt lề 2 dấu cách):

```yaml
  DlqAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: order-dlq-has-messages
      AlarmDescription: Co tin nhan trong DLQ
      Namespace: AWS/SQS
      MetricName: ApproximateNumberOfMessagesVisible
      Dimensions:
        - Name: QueueName
          Value: !GetAtt OrdersDLQ.QueueName
      Statistic: Maximum
      Period: 60
      EvaluationPeriods: 1
      Threshold: 0
      ComparisonOperator: GreaterThanThreshold
      TreatMissingData: notBreaching
      AlarmActions:
        - !Ref OrderTopic

  SfnFailedAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: order-sfn-failed
      AlarmDescription: Co execution Step Functions that bai
      Namespace: AWS/States
      MetricName: ExecutionsFailed
      Dimensions:
        - Name: StateMachineArn
          Value: !Ref OrderStateMachine
      Statistic: Sum
      Period: 60
      EvaluationPeriods: 1
      Threshold: 0
      ComparisonOperator: GreaterThanThreshold
      TreatMissingData: notBreaching
      AlarmActions:
        - !Ref OrderTopic

  LambdaErrorAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: order-lambda-errors
      AlarmDescription: Lambda ProcessOrderFn bao loi
      Namespace: AWS/Lambda
      MetricName: Errors
      Dimensions:
        - Name: FunctionName
          Value: !Ref ProcessOrderFn
      Statistic: Sum
      Period: 60
      EvaluationPeriods: 1
      Threshold: 0
      ComparisonOperator: GreaterThanThreshold
      TreatMissingData: notBreaching
      AlarmActions:
        - !Ref OrderTopic

  OrderDashboard:
    Type: AWS::CloudWatch::Dashboard
    Properties:
      DashboardName: order-workshop
      DashboardBody: !Sub |
        {
          "widgets": [
            { "type": "metric", "x": 0, "y": 0, "width": 12, "height": 6,
              "properties": { "title": "SQS - tin nhan dang cho", "region": "${AWS::Region}",
                "view": "timeSeries", "stat": "Maximum", "period": 60,
                "metrics": [
                  ["AWS/SQS", "ApproximateNumberOfMessagesVisible", "QueueName", "${OrdersQueue.QueueName}", {"label": "Hang doi chinh"}],
                  ["AWS/SQS", "ApproximateNumberOfMessagesVisible", "QueueName", "${OrdersDLQ.QueueName}", {"label": "DLQ"}]
                ] } },
            { "type": "metric", "x": 12, "y": 0, "width": 12, "height": 6,
              "properties": { "title": "SQS - tuoi cua tin nhan cu nhat (giay)", "region": "${AWS::Region}",
                "view": "timeSeries", "stat": "Maximum", "period":
