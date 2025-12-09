---
title: "Worklog Tuần 3"

weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

* Hiểu toàn diện về Amazon EC2 và các loại instance.
* Biết cách sử dụng AMI, tạo backup, quản lý key pair.
* Nắm được EBS và các tính năng lưu trữ đi kèm.
* Hiểu cơ chế Auto Scaling và khả năng mở rộng tài nguyên tự động.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                 | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Giới thiệu Amazon EC2 <br> - Tìm hiểu EC2 Instance Types (General, Compute, Memory, Storage Optimized…)                                                | 13/10/2025   | 13/10/2025      | <https://000004.awsstudygroup.com/> |
| 3   | - Tìm hiểu AMI (Amazon Machine Image) <br> - Backup & Restore EC2 <br> - Tạo & quản lý Key Pair                                                          | 14/10/2025   | 14/10/2025      | <https://000013.awsstudygroup.com/> |
| 4   | - Tìm hiểu Elastic Block Store (EBS): <br> &emsp; + Volume types <br> &emsp; + Snapshot <br> &emsp; + Gắn/Tháo volume                                   | 15/10/2025   | 15/10/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 5   | - Tìm hiểu EC2 Auto Scaling: <br> &emsp; + Launch Template <br> &emsp; + Auto Scaling Group <br> &emsp; + Scaling Policies                                | 16/10/2025   | 16/10/2025      | <https://000006.awsstudygroup.com/> |
| 6   | - Tổng hợp kiến thức EC2 <br> - Thực hành thiết lập Auto Scaling Group cơ bản                                                                             | 17/10/2025   | 17/10/2025      | <https://000006.awsstudygroup.com/> |

### Kết quả đạt được tuần 3:

* Hiểu rõ các khái niệm và thành phần của EC2:
  * Instance Types và use case từng loại.
  * On-Demand, Reserved, Spot Instances.

* Làm chủ AMI:
  * Tạo AMI từ EC2 đang chạy.
  * Sao lưu & phục hồi từ AMI.
  * Xử lý lỗi khi khởi tạo từ AMI.

* Nắm được cơ chế quản lý Key Pair:
  * SSH key pair.
  * Khôi phục EC2 khi mất key pair.

* Hiểu đầy đủ về EBS:
  * Các loại volume (gp2/gp3, io1/io2, st1, sc1).
  * Snapshot & restore.
  * Tăng dung lượng volume.

* Nắm được Auto Scaling:
  * Tạo Launch Template.
  * Cấu hình Auto Scaling Group.
  * Hiểu Scaling Policies (Target Tracking, Step Scaling…).
  * Hiểu cơ chế tăng/giảm instance tự động.

* Sẵn sàng kết hợp EC2 + VPC để triển khai mô hình hạ tầng hoàn chỉnh.
