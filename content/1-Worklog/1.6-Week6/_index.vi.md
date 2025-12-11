---
title: "Worklog Tuần 6"

weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---



### Mục tiêu tuần 6:

* Thiết lập kiến trúc dự án theo mô hình Monorepo.
* Xây dựng hạ tầng ban đầu bằng IaC (Infrastructure as Code) với AWS SAM.
* Hoàn thiện Backend MVP cho chức năng CREATE phòng.
* Thiết kế giao diện mockup định hướng cho Frontend.
* Thiết lập CI/CD cơ bản chuẩn bị cho giai đoạn triển khai.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                             | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                                                |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ------------------------------------------------------------- |
| 2   | - Tạo cấu trúc **Monorepo**: backend/, frontend/, infrastructure/ <br> - Hoàn thành **template.yaml** (IaC): DynamoDB + 2 Lambda (CRUD & Search) <br> - Chạy `sam build` | 03/11/2025   | 03/11/2025      | https://docs.aws.amazon.com/serverless-application-model/     |
| 3   | - Viết backend MVP cho **roomCrud.js**: chức năng **CREATE** <br> - Cấu hình `package.json` cho backend (aws-sdk, node-fetch)                                         | 04/11/2025   | 04/11/2025      | https://docs.aws.amazon.com/lambda/                           |
| 4   | - Thiết kế giao diện dạng **wireframe/mockup** cho Home, Search, Post Room (không code UI)                                                                            | 05/11/2025   | 05/11/2025      | https://www.figma.com                                         |
| 5   | - Thiết lập CI/CD: <br> &emsp; + Cấu hình **AWS Secrets** trên GitHub <br> &emsp; + Viết nháp **deploy-backend.yml**                                                  | 06/11/2025   | 06/11/2025      | https://docs.github.com/en/actions                            |

### Kết quả đạt được tuần 6:

* Thiết lập thành công cấu trúc Monorepo chuẩn chỉnh: backend/, frontend/, infrastructure/.
* Hoàn thiện file IaC **template.yaml**:
  * 1 DynamoDB Table: **Rooms**
  * 2 Lambda Functions: **roomCrud.js** và **searchRooms.js**
* Chạy **sam build** thành công — xác nhận hạ tầng sẵn sàng triển khai.
* Backend MVP hoàn thành chức năng **CREATE Room** với Lambda roomCrud.js.
* Thiết kế mockup các trang Home, Search & Post Room → thống nhất bố cục và UI đầu tiên.
* Thiết lập CI/CD bước đầu:
  * Thêm Secrets AWS trên GitHub
  * Tạo file nháp deploy-backend.yml
* **Kết quả chung:** Nền tảng IaC hoàn thiện, backend CRUD (CREATE) hoạt động và hệ thống đã sẵn sàng để triển khai trong tuần tiếp theo.

