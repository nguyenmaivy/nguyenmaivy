---
title: "Bản đề xuất"

weight: 2
chapter: false
pre: " <b> 2. </b> "
---


# Roommate
## Giải pháp AWS Serverless hợp nhất cho quản lý và tìm kiếm phòng trọ theo thời gian thực


### 1. Tóm tắt điều hành
**RoomMate** là một nền tảng tìm kiếm và quản lý phòng trọ được thiết kế cho sinh viên và chủ trọ, tận dụng tối đa kiến trúc **Serverless** và **AWS Free Tier**. Nền tảng cho phép chủ trọ dễ dàng đăng tin, quản lý dữ liệu phòng trọ (thêm, sửa, xóa) và nhận báo cáo định kỳ. Sinh viên có thể tìm kiếm, lọc theo nhiều tiêu chí (vị trí, giá, tiện ích), lưu phòng yêu thích và trò chuyện trực tiếp với chủ trọ qua tính năng chat mini tích hợp. Kiến trúc sử dụng Next.js cho giao diện người dùng, API Gateway + Lambda cho backend, DynamoDB để lưu trữ dữ liệu chính,... để xác thực an toàn. Điểm khác biệt là việc tích hợp openAI để tự động hóa các tác vụ như gửi thông báo email/Telegram khi có phòng mới phù hợp hoặc tạo báo cáo thống kê, giúp tăng tính tương tác và độ tin cậy của hệ thống.

### 2. Tuyên bố vấn đề  
*Vấn đề hiện tại*  
- Các phương thức tìm phòng trọ truyền thống (bảng tin, nhóm mạng xã hội) thường **thiếu hệ thống lọc chi tiết**, thông tin không tập trung, và dễ bị nhiễu bởi tin rác.
- Việc liên hệ giữa sinh viên và chủ trọ thường phải qua điện thoại/Zalo, **tạo ra "friction" (rào cản giao tiếp)** và khó theo dõi lịch sử.
- **Thiếu cơ chế thông báo tự động** khi có phòng mới phù hợp với nhu cầu cụ thể của sinh viên.
- Chủ trọ **thiếu công cụ phân tích** về nhu cầu thị trường, số lượng người xem tin đăng để tối ưu hóa việc cho thuê. 

*Giải pháp*  

Nền tảng **RoomMate** cung cấp một giải pháp tìm kiếm và quản lý trọ tập trung, tích hợp tính năng **chat mini** nội bộ và **thông báo tự động** (qua n8n) dựa trên tiêu chí cá nhân.
- **Kiến trúc Serverless AWS**: Sử dụng **API Gateway + Lambda, DynamoDB** (cho dữ liệu phòng, user, chat), và S3 (cho hình ảnh) để đảm bảo khả năng mở rộng (scale) với chi phí tối ưu (tận dụng Free Tier).
- **Tính năng nổi bật**: Khác biệt so với các ứng dụng chat/rao vặt chung chung, **RoomMate** tập trung vào trải nghiệm tốt cho người thuê khi áp dụng chatbot AI để gợi ý các phòng trọ theo yêu cầu cá nhân.

*Lợi ích và hoàn vốn đầu tư (ROI)*  
- **Lợi ích**: Cung cấp giải pháp rất thực tế cho sinh viên. Giảm rào cản giao tiếp (friction) giữa hai bên qua chat nội bộ. Tạo nền tảng có tính mở rộng cao (dễ dàng thêm bản đồ, review). Đảm bảo đạt đủ 5 mục tiêu cốt lõi của một dự án công nghệ (serverless, CI/CD, monitoring, security, data pipeline).
- **Hoàn vốn đầu tư (ROI)**: Chi phí phát triển và vận hành cực thấp do tận dụng tối đa AWS Free Tier (Lambda, DynamoDB, S3), giảm thiểu chi phí quản lý server. Giá trị mang lại là một sản phẩm hoàn chỉnh, thiết thực, có khả năng nổi bật nhờ tính năng tự động hóa và chat nội bộ.

### 3. Kiến trúc giải pháp  
Nền tảng ứng dụng kiến trúc AWS Serverless để xây dựng hệ thống quản lý và phân tích dữ liệu. Hệ thống sử dụng Amazon API Gateway và AWS Lambda để xử lý nghiệp vụ. Giao diện web được phân phối toàn cầu qua CloudFront, hoạt động như điểm truy cập duy nhất cho cả nội dung tĩnh và các yêu cầu API. Dữ liệu được lưu trữ an toàn trong Amazon S3 (cho file/hình ảnh) với các bucket được bảo vệ, trong khi Amazon DynamoDB đảm nhiệm lưu trữ dữ liệu cấu trúc. Quy trình triển khai được tự động hóa hoàn toàn thông qua AWS CodePipeline lấy mã nguồn từ GitHub. Toàn bộ hoạt động được giám sát bằng Amazon CloudWatch.
 

![Roommate Platform Architecture](/images/2-Proposal/platfrom_architecture.png)

*Dịch vụ AWS sử dụng*
- *AWS Lambda*: Thực thi các hàm xử lý nghiệp vụ (Backend Business Logic).
- *Amazon API Gateway*: Tiếp nhận, xác thực và định tuyến các yêu cầu API từ ứng dụng web đến AWS Lambda.
- *Amazon S3*: S3 - Frontend lưu trữ nội dung tĩnh của ứng dụng web (đích đến của CodePipeline). S3 - Image Storage lưu trữ dữ liệu file và hình ảnh với cơ chế bảo vệ (S3 - Protected).
- *Amazon DynamoDB*: Lưu trữ dữ liệu cấu trúc (NoSQL), bao gồm dữ liệu người dùng và thông tin thiết bị.
- *AWS CodePipeline*: Tự động hóa quy trình CI/CD cho toàn bộ hệ thống, triển khai giao diện web lên S3 - Frontend.
- *Amazon Cognito*: Quản lý danh tính và quyền truy cập (xác thực) cho người dùng, bảo vệ API Gateway.
- *Amazon CloudFront*: Phân phối nội dung toàn cầu cho ứng dụng web (từ S3 - Frontend) và định tuyến các yêu cầu API đến API Gateway.
- *Amazon CloudWatch*: Giám sát, ghi log và cung cấp cảnh báo cho toàn bộ hệ thống.


*Thiết kế thành phần*  
- *Giao diện người dùng*: Ứng dụng web được lưu trữ tại S3 - Frontend và phân phối toàn cầu thông qua CloudFront.
- *Tầng API*: API Gateway kết hợp Lambda xử lý các yêu cầu nghiệp vụ sau khi người dùng được xác thực bởi Cognito.
- *Lưu trữ dữ liệu*: Dữ liệu file và hình ảnh lưu trong S3 - Image Storage (B3 - Image Storage). Dữ liệu cấu trúc, người dùng và thiết bị lưu trong DynamoDB.
- *Pipeline triển khai*: AWS CodePipeline tự động hóa việc build, test và deploy các thay đổi từ GitHub đến S3 Frontend và các tài nguyên Lambda/API Gateway.
- *Bảo mật & Giám sát*: Dữ liệu trong S3 được bảo vệ. Toàn bộ hoạt động hệ thống được giám sát qua Amazon CloudWatch, và quyền truy cập được quản lý bởi Amazon Cognito.


### 4. Triển khai kỹ thuật  
*Các giai đoạn triển khai*  
Dự án được triển khai theo 4 giai đoạn:
1. Nghiên cứu & thiết kế kiến trúc: Tìm hiểu API Gateway, Lambda, DynamoDB, S3, CloudFront, Cognito và CodePipeline. Thiết kế kiến trúc Serverless theo mô hình phân lớp (Frontend – API – Data – CI/CD).
2. Ước tính chi phí & đánh giá khả thi: Sử dụng AWS Pricing Calculator để ước tính chi phí của S3, Lambda, API Gateway, DynamoDB và CloudFront. Điều chỉnh kiến trúc dựa trên chi phí dự kiến và yêu cầu bảo mật.
3. Tối ưu kiến trúc & quy trình triển khai: Tối ưu luồng API Gateway → Lambda → DynamoDB, tinh chỉnh CloudFront cho phân phối toàn cầu, thiết lập S3 nhiều bucket (Frontend và Image Storage) và cấu hình CodePipeline để rút ngắn thời gian triển khai.
4. Phát triển – kiểm thử – triển khai: Phát triển backend Lambda, xây dựng API Gateway, lập trình giao diện web và upload lên S3–Frontend, tạo DynamoDB schema, thiết lập Cognito, hoàn thiện CodePipeline, kiểm thử end-to-end và đưa hệ thống vào vận hành.


*Yêu cầu kỹ thuật*  
Ứng dụng Web & Phân phối nội dung
- Ứng dụng web phải build dạng static để lưu trữ tại S3 – Frontend.
- CloudFront bắt buộc cấu hình để phân phối nội dung toàn cầu và định tuyến API đến API Gateway.
- File và hình ảnh người dùng phải được lưu tại S3 – Image Storage với chính sách bảo vệ.

Tầng API & Logic xử lý
- Tất cả yêu cầu API đi qua API Gateway, được xác thực bằng Cognito User Pool.
- AWS Lambda xử lý nghiệp vụ backend (CRUD dữ liệu, upload metadata, validate thông tin…).
- API phải tuân theo mô hình REST và trả về JSON.

Lưu trữ dữ liệu
- Dữ liệu cấu trúc phải lưu trong DynamoDB, thiết kế bảng hỗ trợ truy vấn nhanh (Partition Key / Sort Key).
- Bucket S3 phải cấu trúc thành 2 phần:
  * S3 – Frontend: chứa file build của trang web.
  * S3 – Image Storage: chứa file người dùng upload, có quyền truy cập giới hạn qua presigned URL.

CI/CD và Tự động hóa
- Toàn bộ quá trình build – test – deploy được tự động qua AWS CodePipeline, nguồn từ GitHub.
- Pipeline phải triển khai đồng thời frontend (S3–Frontend) và backend (Lambda + API Gateway).
- Mỗi thay đổi cần được theo dõi qua CloudWatch Logs.

Bảo mật & Giám sát
- Amazon Cognito quản lý đăng nhập và phân quyền.
- CloudWatch theo dõi log Lambda, cảnh báo lỗi API Gateway, và giám sát trạng thái hệ thống.
- S3 phải bật chính sách bảo mật (Block Public Access).
- API phải yêu cầu xác thực, chỉ CloudFront được phép truy cập trực tiếp.

### 5. Lộ trình & Mốc triển khai  
- *Trước thực tập (Tháng 0)*: 1 tháng lên kế hoạch và đánh giá trạm cũ.  
- *Thực tập (Tháng 1–3)*:  
    - Tháng 1: Học các kiến thức cơ bản của AWS.
      - Nắm vững các kiến thức cơ bản và một số dịch vụ AWS nền tảng (API Gateway, S3, Lambda,... )
      - Hoàn thiện kiến thức về CI/CD, logging và security cơ bản.
    - Tháng 2: Thiết kế và điều chỉnh kiến trúc. 
      - Thiết kế kiến trúc tổng thể của hệ thống.
      - Tối ưu các thành phần: frontend hosting, backend API, storage, bảo mật.
      - Điều chỉnh mô hình theo phản hồi từ mentor và yêu cầu thực tế. 
    - Tháng 3: Triển khai, kiểm thử, đưa vào sử dụng.
      - Triển khai hạ tầng trên AWS theo kiến trúc đã hoàn thiện.
      - Kiểm thử chức năng và hiệu năng (unit test, integration test).
      - Khắc phục lỗi, tối ưu chi phí và tài nguyên.
      - Đưa hệ thống vào vận hành thử nghiệm (pilot).  
- *Sau triển khai*: 
  - Nghiên cứu thêm trong vòng 1 năm.  
  - Theo dõi, tối ưu và bảo trì hệ thống định kỳ.
  - Nghiên cứu thêm các dịch vụ nâng cao (Auto Scaling, WAF, CloudFormation, EKS…).
  - Đề xuất các cải tiến và tính năng mới dựa trên nhu cầu thực tế.

### 6. Ước tính ngân sách  
Có thể xem chi phí trên [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=621f38b12a1ef026842ba2ddfe46ff936ed4ab01)  
Hoặc tải [tệp ước tính ngân sách](../attachments/budget_estimation.pdf).  

*Chi phí hạ tầng*  
- AWS CloudFront: 0.00 (20-50 GB).  
- AWS Lambda: 0,00 USD/tháng (50.000 request, 100GB lưu trữ).
- S3 - Fontend bucket: 0,00 USD/tháng (6 GB).
- S3 - Image Storage: 0.35-0.6 USD/tháng (15-25 GB).
- Amazon API Gateway: 0.02 USD/tháng (20.000 request).
- Amazon DynamoDB: 0.00 USD/tháng (<10.000 read/write, < 1GB).
- Amazon Cognito: 0.00 USD ( < 50 User).
- AWS CodePipeline (CI/CD từ GitHub): 0 USD (20-50 build/tháng).
- Amazon CloudWatch Logs: 0.5-3.0 (2-15 GB Logs).
  

*Tổng*: 0,87-1.12 USD/tháng, 10.44-13.44 USD/12 tháng.

### 7. Đánh giá rủi ro  
*Ma trận rủi ro*  
- Mất mạng: Ảnh hưởng trung bình, xác suất trung bình.
- Vượt ngân sách: Ảnh hưởng trung bình, xác suất thấp.
- Lỗi chatbox realtime: Ảnh hưởng trung bình, xác suất trung bình
- Lỗi tích hợp bản đồ hoặc hết hạn key: Ảnh hưởng trung bình, xác suất thấp
 
*Chiến lược giảm thiểu*  
- Mạng: Lưu trữ cục bộ trên Raspberry Pi với Docker.  
- Cảm biến: Kiểm tra định kỳ, dự phòng linh kiện.  
- Chi phí: Cảnh báo ngân sách AWS, tối ưu dịch vụ.  

*Kế hoạch dự phòng*  
- Quay lại thu thập thủ công nếu AWS gặp sự cố.
- Sử dụng CloudFormation để khôi phục cấu hình liên quan đến chi phí.
- Chuyển sang chế độ hiển thị địa chỉ dạng text hoặc sử dụng API dự phòng (Mapbox / OpenStreetMap).
- Khi chatbox bị gián đoạn, tự động chuyển sang form liên hệ hoặc tin nhắn offline.
 

### 8. Kết quả kỳ vọng  
*Cải tiến kỹ thuật*: Dữ liệu và phân tích thời gian thực thay thế quy trình thủ công. Có thể mở rộng tới 10–15 trạm.  
*Giá trị dài hạn*: Nền tảng dữ liệu 1 năm cho nghiên cứu AI, có thể tái sử dụng cho các dự án tương lai.