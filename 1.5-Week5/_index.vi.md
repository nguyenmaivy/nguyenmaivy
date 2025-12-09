---
title: "Worklog Tuần 5"

weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

* Hiểu các dịch vụ bảo mật cốt lõi trên AWS
* Thực hành cấu hình Security Hub, tối ưu EC2 và quản lý tài nguyên bằng Tag/IAM.
* Thiết lập IAM User/Group/Role, Permission Boundary và xác định mục tiêu – kiến trúc đề tài.
* Chọn đề tài dự án và xác định các yêu cầu của dự án


### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Làm quen với các dịch vụ bảo mật trên AWS: <br>&emsp; + Mô hình chia sẻ trải nghiệm <br>&emsp; + Amazon Cognito <br>&emsp; + AWS Organization <br>&emsp; + AWS Identity Center (SSO) <br>&emsp; + AWS KMS <br>                                                                                             | 03/11/2025   | 03/11/2025      |
| 3   | - **Thực hành:** <br>&emsp; + Kích hoạt Security Hub và kiểm tra đánh giá theo từng bộ tiêu chuẩn  <br>&emsp; + Tối ưu chi phí EC2 với Lambda                                          | 04/11/2025   | 04/11/2025      | <https://000018.awsstudygroup.com/> <br>&emsp; <https://000022.awsstudygroup.com/> |
| 4   | - **Thực hành:** <br>&emsp; + Quản lý tài nguyên bằng Tag và Resource Groups <br>&emsp; + Quản lý truy cập vào dịch vụ EC2 resource tag với AWS IAM | 05/11/2025   | 05/11/2025     | <https://000027.awsstudygroup.com/> <br>&emsp; <https://000028.awsstudygroup.com/> |
| 5   | - - **Thực hành:** <br>&emsp; + Giới hạn quyền của User với IAM permission boundary <br>&emsp; + Mã hóa ở trạng thái lưu trữ với AWS KMS                 | 06/11/2025  | 06/11/2025      | <https://000030.awsstudygroup.com/> <br>&emsp; <https://000033.awsstudygroup.com/> |
| 6   | - Chọn đề tài và xác định mục tiêu của đề tài <br> - Thiết kế kiến trúc hệ thống <br> - **Thực hành:** <br>&emsp; + Tạo IAM Group và IAM user và cấu hình Role Condition <br>&emsp; + Cấp quyền cho ứng dụng truy cập dịch vụ AWS với IAM Role                                                                                         | 07/11/2025  | 09/11/2025      | <https://000044.awsstudygroup.com/> <br>&emsp; <https://000048.awsstudygroup.com/> |


### Kết quả đạt được tuần 5:

* Hiểu và nắm được các nhóm dịch vụ bảo mật trên AWS:
  * Mô hình chia sẻ trải nghiệm (Share Responsibility Model)
  * Amazon Identity and access management
  * Amazon Cognito
  * AWS Organization
  * AWS Identity Center (SSO)
  * AWS KMS
* Thực hành và vận hành các dịch vụ bảo mật
  * Kích hoạt và làm quen với AWS Security Hub.
  * Thực hiện kiểm tra, đánh giá bảo mật theo các bộ tiêu chuẩn và AWS Best Practices.
  * Nhận diện được các nhóm cảnh báo (Findings) và mức độ ưu tiên xử lý.
* Thực hành tối ưu chi phí EC2 bằng cách sử dụng AWS Lambda để tự động tắt/bật EC2 theo lịch. Xác định được những tài nguyên không cần thiết có thể tối ưu.
* Quản lý tài nguyên AWS bằng Tag và Resource Groups
  * Thiết lập hệ thống Tag chuẩn (Environment, Owner, Project, CostCenter).
  * Áp dụng Tag cho EC2, S3, Lambda và các tài nguyên khác.
  * Tạo Resource Groups để theo dõi và quản lý tài nguyên theo từng nhóm hoặc dự án
* Quản lý truy cập bằng IAM và điều kiện Tag
  * Cấu hình chính sách IAM theo điều kiện (Condition) gắn với Tag.
  * Giới hạn quyền truy cập EC2 theo Resource Tag nhằm đảm bảo phân tách môi trường và giảm rủi ro thao tác nhầm lẫn.
* Giới hạn quyền người dùng bằng Permission Boundary
  * Thiết lập Permission Boundary cho IAM User/Role.
  * Đảm bảo người dùng không thể tự cấp quyền vượt giới hạn cho phép.
  * Mã hóa dữ liệu với AWS KMS
  * Cấu hình mã hóa dữ liệu ở trạng thái lưu trữ (at-rest) cho S3, EBS và các dịch vụ liên quan.
  * Tạo và quản lý Key, phân quyền sử dụng Key thông qua chính sách KMS.
* Lựa chọn được đề tài dự án và xác định mục tiêu, phạm vi và yêu cầu đầu ra của đề tài
* Phác thảo kiến trúc tổng quan ứng dụng/dự án. Xác định các dịch vụ AWS sử dụng, quy trình phân quyền, mã hóa và logging.
* Thực hành IAM nâng cao
  * Tạo IAM Group và IAM User tương ứng với từng nhóm chức năng.
  * Cấu hình IAM Role và thiết lập các điều kiện truy cập (IP/Tag/Time).
  * Cấp quyền cho ứng dụng truy cập vào dịch vụ AWS thông qua IAM Role.



