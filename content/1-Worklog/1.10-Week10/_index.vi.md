---
title: "Worklog Tuần 10"

weight: 1
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10:

* Thiết lập cơ chế export dữ liệu từ DynamoDB sang S3 phục vụ phân tích.
* Tiếp tục sửa lỗi và tối ưu hệ thống.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc                                                                                                  | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo                                      |
| --- | ---------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ------------------------------------------------------- |
| 2   | - Nghiên cứu cơ chế export dữ liệu từ DynamoDB sang S3                                                     | 01/12/2025   | 01/12/2025      | https://docs.aws.amazon.com/amazondynamodb/             |
| 3   | - Viết script export dữ liệu DynamoDB sang S3 (JSON/CSV)                                                   | 02/12/2025   | 02/12/2025      | https://docs.aws.amazon.com/AmazonS3/                   |
| 4   | - Thiết lập lịch trình export dữ liệu phục vụ backup & phân tích                                           | 03/12/2025   | 03/12/2025      | https://docs.aws.amazon.com/scheduler/                 |
| 5   | - Sửa lỗi Backend: API, xử lý dữ liệu, phân quyền                                                         | 04/12/2025   | 04/12/2025      |                                                        |
| 6   | - Sửa lỗi Frontend: hiển thị dữ liệu, form, trải nghiệm người dùng                                        | 05/12/2025   | 05/12/2025      | https://nextjs.org/docs                                |
| 7   | - Kiểm thử tổng thể hệ thống sau khi sửa lỗi và export dữ liệu                                             | 06/12/2025   | 06/12/2025      |                                                        |

### Kết quả đạt được tuần 10:

* Thiết lập thành công cơ chế **export dữ liệu từ DynamoDB sang S3**:
  * Dữ liệu được lưu trữ dưới dạng **JSON/CSV**.
  * Phục vụ **backup định kỳ** và **phân tích dữ liệu**.
* Hoàn thành việc **sửa lỗi Backend & Frontend**:
  * Ổn định các API chính.
  * Cải thiện hiển thị giao diện người dùng.
* Kiểm thử lại toàn hệ thống sau khi cập nhật.
* **Kết quả tổng thể:** Hệ thống vận hành ổn định hơn, dữ liệu được sao lưu an toàn và sẵn sàng cho các bước phân tích tiếp theo.
