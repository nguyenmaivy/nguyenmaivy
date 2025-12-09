---
title: "Worklog Tuần 7"

weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:

* Triển khai hạ tầng AWS cho dự án (Lambda, API Gateway, DynamoDB).
* Xây dựng Frontend MVP (Base UI) và hoàn thiện form đăng tin.
* Tích hợp API giữa Frontend ↔ API Gateway ↔ Lambda ↔ DynamoDB.
* Viết các chức năng backend cơ bản phục vụ tính năng tìm kiếm phòng.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                 | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                                             |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ---------------------------------------------------------- |
| 2   | - Deployment hạ tầng AWS bằng **sam deploy** lần đầu <br> - Tạo Lambda, API Gateway, DynamoDB <br> - Thêm API Gateway Endpoint vào `frontend/.env.local`                  | 10/11/2025   | 10/11/2025      | https://cloudjourney.awsstudygroup.com/                   |
| 3   | - Code giao diện **Layout (`layout.js`)** <br> - Code giao diện **Trang Đăng tin (`post-room/page.js`)** <br> - Cài đặt & cấu hình **Tailwind CSS**                      | 11/11/2025   | 11/11/2025      |                                                            |
| 4   | - Viết **API Proxy** trong Next.js tại `frontend/api/proxy/`                                                                        | 12/11/2025   | 12/11/2025      | https://nextjs.org/docs                                   |
| 5   | - Tích hợp form đăng tin: kết nối trang `post-room` với **Lambda roomCrud.js** thông qua API Proxy                                  | 13/11/2025   | 13/11/2025      | https://docs.aws.amazon.com/lambda/                       |
| 6   | - Viết backend cho tìm kiếm phòng: tạo file **searchRooms.js** (chức năng READ – lấy tất cả phòng từ DynamoDB)                      | 14/11/2025   | 14/11/2025      | https://docs.aws.amazon.com/amazondynamodb/               |
| 7   | - Kiểm thử toàn bộ luồng: Frontend → API Proxy → API Gateway → Lambda → DynamoDB <br> - Fix lỗi và hoàn thiện MVP                   | 15/11/2025   | 15/11/2025      |                                                            |

### Kết quả đạt được tuần 7:

* Triển khai hạ tầng AWS thành công bằng **sam deploy** (Lambda, DynamoDB, API Gateway).
* Cấu hình môi trường Frontend (`.env.local`) với API Gateway Endpoint.
* Hoàn thiện giao diện cơ bản (MVP):
  * Layout
  * Trang đăng tin (post-room)
  * Tailwind CSS hoạt động ổn định
* Viết API Proxy trong Next.js để kết nối với Lambda.
* Tích hợp thành công form đăng tin:
  * Gửi dữ liệu → API Proxy → API Gateway → Lambda → DynamoDB.
* Hoàn thiện backend xử lý tìm kiếm cơ bản (`searchRooms.js` – READ).
* Toàn bộ hệ thống đã có thể:
  * Truy cập website
  * Đăng tin phòng
  * Lưu dữ liệu vào DynamoDB thành công

