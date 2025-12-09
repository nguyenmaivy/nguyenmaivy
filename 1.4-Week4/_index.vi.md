---
title: "Worklog Tuần 4"

weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---


### Mục tiêu tuần 4:

* Hiểu về các dịch vụ lưu trữ trên AWS
* Biết cách triển khai hệ thống Backup, thực hành Import/Export máy ảo và triển khai Storage Gateway

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Tìm hiểu Amazon Simple Storage Service (S3) với tính năng Access Point và Storage Class của S3 <br>&emsp; - Tìm hiểu S3 Static Website & CORS, Control Access, Object Key & Performance, Glacier                                           | 20/10/2025   | 20/10/2025 |<https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html> | 
| 3   | - Tìm hiểu Snow Family, Storage Gateway, Backup <br>                | 21/10/2025   | 21/10/2025      | <https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html> |
| 4   | - **Thực hành:** triển khai AWS Backup cho hệ thống <br>&emsp; + Tạo S3 Bucket và triển khai hạ tầng <br>&emsp; + Tạo Backup Plan, thiết lập thông báo, kiểm tra hoạt động                                                                   | 22/10/2025   | 22/10/2025      | <https://000013.awsstudygroup.com/> |
| 5   | - **Thực hành:** Export/Import máy ảo <br>&emsp; + Chuẩn bị máy ảo <br>&emsp; + Import máy ảo vào AWS <br>&emsp; +     Export EC2 Instance từ AWS                                                              | 23/10/2025   | 23/10/2025      | <https://000014.awsstudygroup.com/> |
| 6   | - **Thực hành:** <br>&emsp; + Triển khai file storage gateway: Tạo Storage Gateway, File Shares và kết nối File Shares ở máy On-primise <br>&emsp; + Thiết lập hệ thống lưu trữ dữ liệu chung cho hạ tầng Windows                          | 24/10/2025   | 24/10/2025      | <https://000024.awsstudygroup.com/> |


### Kết quả đạt được tuần 4:

* Hiểu Amazon Simple Storage Service (S3) là gì và nắm được các nhóm tính năng cơ bản:
  * Amazon S3 Access Point
  * S3 Static Website & CORS
  * Control Access, Object Key & Performance, Glacier
* Hiểu các dịch vụ Snow Family, Storage Gateway, Backup
* Đã tạo và cấu hình S3 Bucket và triển khai hạ tầng, thiết lập thông báo và kiểm tra hoạt động thành công.
* Tạo Storage Gateway, File Shares thành công và có khả năng kết nối file shares ở máy On-premise
* Có khả năng Import máy ảo vào AWS và export EC2 Instance từ AWS
  * Export máy ảo từ On-premise
  * Tải máy ảo lên AWS
  * Triển khai EC2 Instance từ AMI
* Có khả năng export EC2 Instance từ AWS
  * Thiết lập ACL cho S3 Bucket
  * Export máy ảo từ EC2 Instance
* Thiết lập hệ thống lưu trữ dữ liệu chung cho hạ tầng Windows:
  * Tạo môi trường thực hành để tạo file share mới
  * Kiểm tra và giám sát hiệu năng 
  * Kích hoạt các các thành phần để có thể triển khai FSX trên Windows như: hạn ngạch bộ nhớ của người dùng, chia sẻ truy cập liên tục,...

