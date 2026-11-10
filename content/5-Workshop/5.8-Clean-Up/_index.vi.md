---
title: "Clean up"
date: "2026-11-06"
weight: 8
chapter: false
pre: " <b> 5.8 </b> "
---
## Mục tiêu

Xóa toàn bộ tài nguyên đã tạo để không phát sinh chi phí, đặc biệt là hai API
công khai ở mục 5.7.

#### 1. Xóa stack chính

```powershell
sam delete --stack-name order-system --region ap-southeast-1
```

Trả lời `y` cho các câu hỏi xác nhận. Lệnh này xóa stack CloudFormation cùng
SQS, DLQ, DynamoDB, Lambda, Step Functions, SNS, CloudWatch alarm, dashboard và
HTTP API.

Kiểm tra stack đã biến mất:

```powershell
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE DELETE_FAILED --query "StackSummaries[].StackName" --output text
```

![Kết quả xóa stack](/images/5-Workshop/5.8-Cleanup/delete-stack.png)

#### 2. Xóa log group của Lambda

Log group do Lambda tự tạo nên không bị xóa cùng stack:

```powershell
$groups = aws logs describe-log-groups --log-group-name-prefix /aws/lambda/order-system --query "logGroups[].logGroupName" --output text
$groups -split "\s+" | Where-Object { $_ } | ForEach-Object { aws logs delete-log-group --log-group-name $_ }
```

#### 3. Xóa stack quản lý của SAM (tùy chọn)

SAM tạo thêm stack `aws-sam-cli-managed-default` kèm một S3 bucket chứa mã
triển khai. Chi phí rất nhỏ nhưng bạn có thể dọn: vào **S3**, mở bucket có tên
bắt đầu bằng `aws-sam-cli-managed-default`, chọn **Empty**, sau đó vào
**CloudFormation** xóa stack `aws-sam-cli-managed-default`.

#### 4. Xóa email đã xác minh trong SES (tùy chọn)

Vào **Amazon SES** → **Identities** → chọn email → **Delete**.

#### 5. Thu hồi quyền truy cập

Làm bước này **sau cùng**, vì sau khi xóa khóa thì CLI không dùng được nữa:

1. Vào **IAM** → **Users** → chọn user → **Security credentials** → **Deactivate** rồi **Delete** access key.
2. Gỡ policy `OrderWorkshopDeployPolicy` khỏi user, rồi xóa policy nếu không dùng nữa.
3. Xóa profile trên máy: mở file `%USERPROFILE%\.aws\credentials` và `config`, xóa phần `order-workshop`.
4. Xóa biến môi trường trong PowerShell:

```powershell
Remove-Item Env:API_KEY, Env:CUSTOMER_EMAIL -ErrorAction SilentlyContinue
```

#### 6. Kiểm tra lần cuối

1. Vào Console của **Lambda**, **SQS**, **DynamoDB**, **Step Functions**, **API Gateway**, **CloudWatch** (Alarms, Dashboards): không còn tài nguyên `order-system` nào.
2. Vào **Billing and Cost Management** → **Cost Explorer**, kiểm tra lại sau 1 đến 2 ngày để chắc chắn chi phí về 0.
3. Giữ lại ngân sách trong **AWS Budgets** nếu bạn còn dùng tài khoản này.

![Kiểm tra chi phí](/images/5-Workshop/5.8-Cleanup/cost-check.png)

## Checklist dọn dẹp

| Việc | Xong |
|---|---|
| Xóa stack `order-system` | ☐ |
| Xóa log group Lambda | ☐ |
| Xóa stack và bucket của SAM (tùy chọn) | ☐ |
| Xóa identity SES (tùy chọn) | ☐ |
| Xóa access key và policy IAM | ☐ |
| Xóa profile và biến môi trường trên máy | ☐ |
| Kiểm tra Cost Explorer sau 1 đến 2 ngày | ☐ |
