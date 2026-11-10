---
title: "Giới thiệu và kiến trúc"
date: "2026-10-03"
weight: 1
chapter: false
pre: " <b> 5.1 </b> "
---
## Tổng quan

Trong một ứng dụng thương mại điện tử, bước đặt hàng kéo theo nhiều tác vụ phía sau: ghi nhận đơn, gửi email xác nhận, thông báo cho quản trị viên. Nếu xử lý tất cả trong cùng một yêu cầu (đồng bộ), hệ thống dễ chậm khi lượng đơn tăng, và một dịch vụ phụ bị lỗi có thể làm mất đơn hoặc gửi email trùng.

Workshop này xây dựng luồng xử lý đơn hàng theo hướng **event-driven, chịu lỗi và quan sát được** trên AWS, sau đó đo đạc để so sánh với phương án xử lý đồng bộ.

## Bạn sẽ làm gì

- Tách bước nhận đơn khỏi bước xử lý bằng **Amazon SQS** và hàng đợi lỗi (**DLQ**)
- Chống xử lý trùng bằng **Amazon DynamoDB** (idempotency)
- Điều phối quy trình gửi email và thông báo bằng **AWS Step Functions**, **Amazon SES**, **Amazon SNS**
- Theo dõi hệ thống và nhận cảnh báo với **Amazon CloudWatch**
- Giả lập lỗi và tải để đánh giá hiệu quả của kiến trúc

## Kiến trúc

{{< figure src="/images/5-Workshop/5.1-Introduction/architecture.png" alt="Kiến trúc tổng thể" >}}

## Chỉ số đánh giá

- Thời gian phản hồi của API đặt hàng (trung bình, p95)
- Tỉ lệ đơn bị mất khi có lỗi
- Số email gửi trùng
- Thời gian từ lúc xảy ra lỗi đến lúc quản trị viên nhận cảnh báo

## Thời lượng và chi phí

Khoảng 2–3 giờ. Phần lớn dịch vụ nằm trong free tier hoặc có chi phí rất thấp ở
quy mô thử nghiệm. Hãy thực hiện bước dọn dẹp ở cuối để tránh phát sinh phí.