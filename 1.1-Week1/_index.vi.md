---
title: "Worklog Tuần 1"

weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---



### Mục tiêu tuần 1:

* Kết nối, làm quen với các thành viên trong First Cloud Journey.
* Hiểu dịch vụ AWS cơ bản, cách dùng console & CLI.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - Làm quen với các thành viên FCJ <br> - Đọc và lưu ý các nội quy, quy định tại đơn vị thực tập                                                                                             | 29/09/2025   | 29/09/2025      | |
| 3   | - Tìm hiểu AWS và các loại dịch vụ <br>&emsp; + Compute <br>&emsp; + Storage <br>&emsp; + Networking <br>&emsp; + Database <br>&emsp; + ... <br>                                            | 30/09/2025   | 30/09/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 4   | - Tạo AWS Free Tier account <br> - Tìm hiểu AWS Console & AWS CLI <br> - **Thực hành:** <br>&emsp; + Tạo AWS account <br>&emsp; + Cài AWS CLI & cấu hình <br> &emsp; + Cách sử dụng AWS CLI | 01/10/2025   | 01/10/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 5   | - Tìm hiểu EC2 cơ bản: <br>&emsp; + Instance types <br>&emsp; + AMI <br>&emsp; + EBS <br>&emsp; + Security Group <br>&emsp; + ... <br> - Các cách remote SSH vào EC2 <br> - Tìm hiểu Elastic IP   <br>                  | 02/10/2025   | 02/10/2025      | <https://cloudjourney.awsstudygroup.com/> |
| 6   | - **Thực hành:** <br>&emsp; + Tạo EC2 instance (T2.micro, Amazon Linux 2, Key Pair mới) <br>&emsp; + Cấu hình **Security Group** để mở cổng SSH <br>&emsp; + Kết nối SSH từ máy cá nhân <br>&emsp; + Gắn **EBS volume** & mount vào EC2                                                                                         | 03/10/2025   | 03/10/2025      | <https://cloudjourney.awsstudygroup.com/> |

<br>

---

### Kết quả đạt được tuần 1:

* **Mối quan hệ:** Đã làm quen và kết nối với các thành viên trong nhóm **First Cloud Journey (FCJ)**.
* **Kiến thức cơ bản AWS:**
  * Hiểu **AWS là gì** và nắm được các nhóm dịch vụ cơ bản:
    * Compute (**EC2**), Storage (**S3, EBS**), Networking (**VPC, Security Group**), Database (**RDS**), v.v.
* **Thực hành AWS Account & Tools:**
  * Đã tạo và cấu hình **AWS Free Tier account** thành công.
  * Làm quen với **AWS Management Console** và biết cách tìm, truy cập, sử dụng dịch vụ từ giao diện web.
  * Cài đặt và cấu hình **AWS CLI** trên máy tính bao gồm:
    * **Access Key, Secret Key, Region** mặc định.
* **Sử dụng EC2:**
  * Nắm được các khái niệm cơ bản về **EC2 (Instance types, AMI, EBS, Security Group)**.
  * Đã tạo thành công một **EC2 instance** (t2.micro) và thực hiện kết nối **SSH** từ máy cá nhân.
  * Biết cách quản lý và gắn **EBS volume** vào EC2 instance.
* **Sử dụng AWS CLI:**
  * Sử dụng AWS CLI để thực hiện các thao tác cơ bản như:
    * Kiểm tra thông tin tài khoản & cấu hình (vd: `aws configure list`).
    * Lấy danh sách region (vd: `aws ec2 describe-regions`).
    * Xem dịch vụ EC2 (vd: `aws ec2 describe-instances`).
    * Tạo và quản lý key pair.
* **Tổng hợp:** Có khả năng kết nối giữa giao diện web và CLI để quản lý tài nguyên AWS song song.
