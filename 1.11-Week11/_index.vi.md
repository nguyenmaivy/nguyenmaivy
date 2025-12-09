---
title: "Worklog Tuần 11"

weight: 1
chapter: false
pre: " <b> 1.11. </b> "
---

### Mục tiêu tuần 11:

* Nâng cao chất lượng hệ thống thông qua việc viết Unit Test cho Backend và E2E Test cho Frontend.
* Xây dựng pipeline phân tích dữ liệu và hệ thống Business Intelligence (BI) cơ bản.
* Hoàn thiện và tối ưu quy trình CI/CD cho cả Frontend và Backend.
* Rà soát và đảm bảo an toàn hệ thống thông qua việc kiểm tra quyền IAM theo nguyên tắc **Least Privilege**.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                       | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo                                |
| --- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ------------------------------------------------- |
| 2   | - Viết **Backend Unit Tests** bằng **Jest** cho các Lambda cốt lõi: `roomCrud.js`, `searchRooms.js`                                                                                          | 08/12/2025   | 08/12/2025      | https://jestjs.io/                                |
| 3   | - Viết **Frontend E2E Tests** bằng **Cypress** cho 3 luồng chính: <br> &emsp; + Đăng nhập <br> &emsp; + Tìm kiếm/Lọc <br> &emsp; + Đăng tin                                                  | 09/12/2025   | 09/12/2025      | https://www.cypress.io/                           |
| 4   | - Thiết lập **AWS Athena** trên dữ liệu được export từ S3 <br> - Viết file **athena-queries.sql**: <br> &emsp; + Giá thuê trung bình theo quận <br> &emsp; + Top 5 phòng có lượt xem cao nhất | 10/12/2025   | 10/12/2025      | https://docs.aws.amazon.com/athena/               |
| 5   | - Cấu hình **Amazon QuickSight** <br> - Xây dựng một dashboard trực quan hóa kết quả truy vấn từ Athena                                                                                       | 11/12/2025   | 11/12/2025      | https://docs.aws.amazon.com/quicksight/           |
| 6   | - Tối ưu file **deploy-frontend.yml**: Build Next.js → Upload S3 → Invalidate CloudFront <br> - Rà soát lại **IAM Permissions** cho các Lambda theo nguyên tắc Least Privilege              | 12/12/2025   | 12/12/2025      | https://docs.github.com/en/actions                |

### Kết quả đạt được tuần 11:

* Hoàn thiện **Unit Test cho Backend**:
  * `roomCrud.js`
  * `searchRooms.js`
* Hoàn thành **E2E Test cho Frontend bằng Cypress**:
  * Đăng nhập
  * Tìm kiếm & Lọc
  * Đăng tin
* Hệ thống ổn định hơn với **tỷ lệ bao phủ test đạt khoảng 70%**.
* Xây dựng thành công **Data Pipeline & BI**:
  * Thiết lập AWS Athena trên dữ liệu xuất từ S3.
  * Viết các câu truy vấn:
    * Giá thuê trung bình theo quận
    * Top 5 phòng có lượt xem cao nhất
  * Xây dựng Dashboard trực quan trên Amazon QuickSight.
* Hoàn thiện **CI/CD tự động**:
  * Deploy Frontend tự động (Build → S3 → CloudFront)
  * Rà soát, tối ưu pipeline Backend.
* Đã rà soát toàn bộ **IAM Permissions** theo nguyên tắc **đặc quyền tối thiểu (Least Privilege)**.
* **Kết quả tổng thể:** Hệ thống đã sẵn sàng ở mức hoàn chỉnh, có đầy đủ kiểm thử, phân tích dữ liệu và triển khai tự động.
