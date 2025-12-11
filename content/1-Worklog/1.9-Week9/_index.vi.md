---
title: "Worklog Tuần 9"

weight: 1
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9:

* Hoàn thiện hệ thống xác thực người dùng bằng AWS Cognito.
* Phân quyền cơ bản giữa User và Owner.
* Xây dựng chức năng Chat và Favorite (Yêu thích).
* Tối ưu trải nghiệm người dùng với giao diện Responsive.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                                 | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo                                     |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ------------------------------------------------------ |
| 2   | - **AWS Cognito Setup:** Tạo User Pool và App Client <br> - Tích hợp **AWS Amplify** vào Next.js                                                                                                            | 24/11/2025   | 24/11/2025      | https://docs.aws.amazon.com/cognito/                  |
| 3   | - Xây dựng UI cho **Sign Up / Sign In** (sử dụng component/form thư viện) <br> - Thiết lập **Cognito Groups** để phân quyền User vs Owner                                            | 25/11/2025   | 25/11/2025      | https://docs.amplify.aws/                             |
| 4   | - Backend: Hoàn thiện logic `chatMessage.js` (Lưu/Lấy tin nhắn giữa 2 user) <br> - Backend: Hoàn thiện logic `favorite.js` (Thêm/Xóa phòng yêu thích)                                                    | 26/11/2025   | 26/11/2025      | https://docs.aws.amazon.com/lambda/                  |
| 5   | - Frontend: Xây dựng **Chat Modal/Page** (`ChatModal.js`, `chat/page.js`) <br> - Frontend: Tích hợp nút **Favorite** trên `RoomCard.js`                                                                  | 27/11/2025   | 27/11/2025      | https://nextjs.org/docs                              |
| 6   | - Tối ưu hóa UI: đảm bảo các trang chính **Responsive (Mobile-first)**                                                                                                                                    | 28/11/2025   | 28/11/2025      |                                                      |
| 7   | - Kiểm thử toàn bộ luồng: Đăng ký → Đăng nhập → Chat → Favorite → Phân quyền                                                                                                                             | 29/11/2025   | 29/11/2025      |                                                      |

### Kết quả đạt được tuần 9:

* Tích hợp thành công **AWS Cognito**:
  * User có thể **Đăng ký / Đăng nhập**.
* Hoàn thiện chức năng **Chat mini**:
  * Lưu và lấy tin nhắn giữa 2 người dùng.
  * Giao diện Chat Modal hoạt động ổn định.
* Hoàn thiện chức năng **Favorite (Yêu thích)**:
  * Thêm/Xóa phòng yêu thích.
  * Hiển thị trạng thái yêu thích trên RoomCard.
* Giao diện được tối ưu **Responsive**, hoạt động tốt trên Tablet và Desktop.
* **Kết quả tổng thể:** Hệ thống đã có đầy đủ tính năng tương tác người dùng (Auth, Chat, Favorite).

