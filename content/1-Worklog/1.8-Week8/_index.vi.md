---
title: "Worklog Tuần 8"

weight: 1
chapter: false
pre: " <b> 1.8. </b> "
---


### Mục tiêu tuần 8:

* Tích hợp bản đồ và geocoding vào hệ thống để hiển thị vị trí phòng.
* Hoàn thiện chức năng tìm kiếm & lọc theo nhiều tiêu chí.
* Kiểm thử toàn bộ chức năng chính trước khi chuyển sang giai đoạn tiếp theo.
* Chuẩn bị Proposal Review (nếu cần).

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                          | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                                             |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------ | --------------- | ---------------------------------------------------------- |
| 2   | - Backend: Tích hợp **Vietmap Geocoding API** trong `roomCrud.js` để lưu tọa độ khi đăng tin <br> - Frontend: Tích hợp **Vietmap-gl** trong `RoomMap.js`          | 17/11/2025   | 17/11/2025      | https://maps.vietmap.vn/                                   |
| 3   | - Backend: Hoàn thiện logic **lọc theo giá, quận, khoảng cách** trong `searchRooms.js`                                                                             | 18/11/2025   | 18/11/2025      | https://docs.aws.amazon.com/amazondynamodb/                |
| 4   | - Frontend: Xây dựng trang **Search (`search/page.js`)** hoàn chỉnh với filters <br> - Hiển thị kết quả trên **bản đồ + danh sách**                               | 19/11/2025   | 19/11/2025      | https://nextjs.org/docs                                    |
| 5   | - Kiểm thử thủ công: đăng tin (đã tích hợp tọa độ), tìm kiếm, lọc                                                             | 20/11/2025   | 20/11/2025      |                                                            |
| 6   | - Xây dựng Proposal Review: Tổng hợp tính năng đã hoàn thành, chuẩn bị tài liệu/trình bày (nếu cần)                                                                | 21/11/2025   | 21/11/2025      |                                                            |

### Kết quả đạt được tuần 8:

* Tích hợp thành công Vietmap Geocoding:  
  * Khi đăng tin, hệ thống tự động lấy tọa độ (lat/lng).  
  * Lưu vào DynamoDB qua Lambda.

* Frontend đã hiển thị bản đồ bằng **Vietmap-gl**, có thể hiển thị vị trí phòng trên bản đồ.

* Hoàn thiện chức năng tìm kiếm & lọc:
  * Lọc theo giá  
  * Lọc theo quận  
  * Lọc theo khoảng cách  
  * Tối ưu truy vấn trong Lambda `searchRooms.js`

* Trang Search đã hoàn chỉnh:
  * Giao diện filters  
  * Danh sách kết quả  
  * Hiển thị vị trí trên bản đồ

* Kiểm thử:
  * Đăng tin → lấy tọa độ → hiển thị đúng  
  * Tìm kiếm và lọc chạy đúng logic  
  * UI + API hoạt động đồng bộ

* Hệ thống tìm kiếm + bản đồ đã hoạt động hoàn chỉnh — **cốt lõi của ứng dụng đã chạy ổn định**.