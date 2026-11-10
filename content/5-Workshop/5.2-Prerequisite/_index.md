---
title: "Chuẩn bị (IAM, region, công cụ)"
date: "2026-10-03"
weight: 2
chapter: false
pre: " <b> 5.2 </b> "
---
## Yêu cầu chuẩn bị

- Một tài khoản AWS dùng cho thử nghiệm (không dùng tài khoản production)
- Một địa chỉ email thật để nhận thư kiểm thử
- Máy tính Windows có PowerShell, đã cài Git
- Region sử dụng trong toàn bộ workshop: **Asia Pacific (Singapore) `ap-southeast-1`**

#### 1. Quyền IAM

Tạo policy sau và gắn vào user dùng để triển khai workshop.
(IAM → Policies → Create policy → tab **JSON** → dán nội dung → đặt tên
`OrderWorkshopDeployPolicy`).

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "WorkshopServices",
      "Effect": "Allow",
      "Action": [
        "cloudformation:*",
        "lambda:*",
        "sqs:*",
        "dynamodb:*",
        "states:*",
        "sns:*",
        "ses:*",
        "cloudwatch:*",
        "logs:*",
        "events:*",
        "s3:*"
      ],
      "Resource": "*"
    },
    {
      "Sid": "WorkshopIam",
      "Effect": "Allow",
      "Action": [
        "iam:CreateRole",
        "iam:DeleteRole",
        "iam:GetRole",
        "iam:PassRole",
        "iam:TagRole",
        "iam:UntagRole",
        "iam:AttachRolePolicy",
        "iam:DetachRolePolicy",
        "iam:PutRolePolicy",
        "iam:DeleteRolePolicy",
        "iam:GetRolePolicy",
        "iam:ListRolePolicies",
        "iam:ListAttachedRolePolicies"
      ],
      "Resource": "*"
    }
  ]
}
```

> Policy này rộng để thuận tiện cho môi trường thử nghiệm. Với môi trường thật,
> hãy giới hạn `Resource` theo từng tài nguyên cụ thể.

![Tạo IAM policy](/images/5-Workshop/5.2-Prerequisite/create-policy.png)

#### 2. Cài đặt công cụ

Mở PowerShell và chạy:

```powershell
winget install -e --id OpenJS.NodeJS.LTS
winget install -e --id Amazon.AWSCLI
winget install -e --id Amazon.SAM-CLI
```

Đóng và mở lại PowerShell, rồi kiểm tra:

```powershell
node -v
aws --version
sam --version
```

#### 3. Kết nối máy với tài khoản AWS

1. Vào IAM → Users → chọn user → **Security credentials** → **Create access key**
   (use case: Command Line Interface).
2. Cấu hình profile:

```powershell
aws configure --profile order-workshop
$env:AWS_PROFILE = "order-workshop"
aws sts get-caller-identity
```

Nhập Access Key, Secret Key, region `ap-southeast-1`, output `json`.
Kết quả của lệnh cuối phải hiển thị thông tin tài khoản của bạn.

![Kiểm tra danh tính](/images/5-Workshop/5.2-Prerequisite/caller-identity.png)

> Không đưa Access Key lên GitHub. Hãy xóa hoặc vô hiệu hóa key sau khi hoàn thành workshop.

#### 4. Xác minh email trong Amazon SES

Tài khoản SES mới ở chế độ **sandbox**: chỉ gửi được thư **từ và đến** các địa chỉ đã xác minh.

1. Vào **Amazon SES** → **Identities** → **Create identity**.
2. Chọn **Email address**, nhập email của bạn, chọn **Create identity**.
3. Mở email xác nhận từ AWS và bấm vào liên kết.
4. Trạng thái identity chuyển sang **Verified**.

![SES verified](/images/5-Workshop/5.2-Prerequisite/ses-verified.png)

#### 5. Đặt cảnh báo chi phí

1. Vào **Billing and Cost Management** → **Budgets** → **Create budget**.
2. Chọn **Zero spend budget** hoặc **Monthly cost budget** (ví dụ 5 USD).
3. Nhập email nhận cảnh báo.

![Tạo budget](/images/5-Workshop/5.2-Prerequisite/budget.png)

#### 6. Tạo thư mục dự án

```powershell
mkdir order-system
cd order-system
mkdir src
```

Ở các mục tiếp theo, ta sẽ thêm `template.yaml` và mã nguồn Lambda vào thư mục này.