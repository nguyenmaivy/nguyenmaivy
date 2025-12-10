---
title: "Blog 4"

weight: 1
chapter: false
pre: " <b> 3.4. </b> "
---


# Cách Zapier chạy các tác vụ riêng biệt trên AWS Lambda và nâng cấp các chức năng ở quy mô lớn

Tác giả: Anton Aleksandrov, Raúl Negrón-Otero, Ankush Kalra, Vítek Urbanec và Chandresh Patel 
Xuất bản: Ngày 25 tháng 07 năm 2025
[Advanced (300)](https://aws.amazon.com/blogs/architecture/category/learning-levels/advanced-300/), [Amazon CloudWatch](https://aws.amazon.com/blogs/architecture/category/management-tools/amazon-cloudwatch/), [Amazon Elastic Kubernetes Service](https://aws.amazon.com/blogs/architecture/category/compute/amazon-kubernetes-service/), [Architecture](https://aws.amazon.com/blogs/architecture/category/architecture/), [AWS Lambda](https://aws.amazon.com/blogs/architecture/category/compute/aws-lambda/), [Customer Solutions](https://aws.amazon.com/blogs/architecture/category/post-types/customer-solutions/), [Monitoring and observability](https://aws.amazon.com/blogs/architecture/category/management-and-governance/monitoring-and-observability/), [Serverless](https://aws.amazon.com/blogs/architecture/category/serverless/) 

[Zapier](https://zapier.com/) là nhà cung cấp dịch vụ tự động hóa không cần mã hàng đầu, với khách hàng sử dụng giải pháp của họ để tự động hóa quy trình làm việc và di chuyển dữ liệu trên hơn 8.000 ứng dụng như Slack, Salesforce, Asana và Dropbox. Zapier vận hành các quy trình tự động hóa này thông qua các tích hợp được gọi là Zap, được triển khai bằng kiến ​​trúc serverless chạy trên [Amazon Web Services](https://aws.amazon.com/) (AWS). Mỗi Zap được hỗ trợ bởi một hàm [AWS Lambda](https://aws.amazon.com/lambda/).

Trong bài viết này, bạn sẽ tìm hiểu cách Zapier xây dựng kiến ​​trúc serverless của mình, tập trung vào ba khía cạnh chính: sử dụng hàm Lambda để xây dựng các Zap riêng biệt, vận hành hơn một trăm nghìn hàm Lambda thông qua cơ sở hạ tầng mặt phẳng điều khiển của Zapier, và tăng cường khả năng bảo mật đồng thời giảm thiểu nỗ lực bảo trì bằng cách đưa các nâng cấp hàm tự động và quy trình dọn dẹp vào kiến ​​trúc nền tảng của họ.

## Kiến trúc môi trường thời gian chạy an toàn và biệt lập
Các Zap do người dùng Zapier tạo ra triển khai logic nghiệp vụ dành riêng cho từng đối tượng thuê, do đó chúng yêu cầu sự cô lập tính toán giữa các đối tượng thuê. Mã triển khai một Zap không thể chia sẻ môi trường thực thi với mã triển khai một Zap khác. Hơn nữa, cùng một loại Zap được sử dụng bởi hai đối tượng thuê khác nhau cũng không thể chia sẻ môi trường thực thi.

Để đạt được mức độ cô lập cần thiết, đội ngũ kỹ thuật của Zapier đã áp dụng [AWS Lambda](https://aws.amazon.com/lambda/) , một dịch vụ điện toán serverless chạy code để phản hồi các sự kiện và tự động quản lý tài nguyên điện toán đám mây. [Chi phí vận hành](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) tối thiểu, tính khả dụng cao tích hợp sẵn [](), [khả năng mở rộng tự động](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html), [mức độ cô lập cao](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html) và [mô hình trả tiền theo mức sử dụng](https://aws.amazon.com/lambda/pricing/) đã khiến Lambda trở thành lựa chọn hoàn hảo cho trường hợp sử dụng này. Hiện tại, kiến ​​trúc của Zapier đang chạy hơn một trăm nghìn hàm Lambda để hỗ trợ quy trình tích hợp của khách hàng.

Vì được hỗ trợ bởi các [microVM Firecracker](https://firecracker-microvm.github.io/) mã nguồn mở, mỗi hàm được cô lập hoàn toàn với các hàm khác. Hơn nữa, mỗi môi trường thực thi thuộc về cùng một hàm (đôi khi được gọi là các thể hiện hàm) cũng được cô lập khỏi các môi trường thực thi khác. Sơ đồ cấu trúc kiến ​​trúc sau đây sử dụng các đường màu đỏ để biểu diễn ranh giới cô lập. Mỗi môi trường thực thi của mỗi hàm đều được cô lập với các môi trường ngang hàng và có các tài nguyên ảo riêng như đĩa, bộ nhớ và CPU. Để biết thêm chi tiết, vui lòng đọc phần. [Bảo mật trong AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-security.html)

<!-- **Kiến trúc giải pháp bây giờ như sau:** -->

> *Hình 1. *
 
Mặt phẳng điều khiển của Zapier được thiết kế dựa trên [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/) (Amazon EKS). Một cơ sở dữ liệu được chỉ định sẽ được sử dụng để duy trì kho dữ liệu chức năng được cập nhật. Bất cứ khi nào người dùng tạo một Zap mới, mặt phẳng điều khiển sẽ tạo một hàm Lambda tương ứng và lưu trữ tham chiếu trong cơ sở dữ liệu kho dữ liệu. Khi một Zap được kích hoạt, mặt phẳng điều khiển sẽ truy xuất thông tin về hàm Lambda liên quan và gọi hàm đó để tạo điều kiện thuận lợi cho quy trình tích hợp, như minh họa trong sơ đồ sau.

> *Hình 2. *

## Hiểu về quy trình ngừng sử dụng runtime

Khi xây dựng kiến ​​trúc sử dụng hệ thống điện toán không máy chủ truyền thống, các kỹ sư đám mây là những người chịu trách nhiệm cập nhật hệ điều hành và phần mềm trên các phiên bản điện toán của họ, đồng thời áp dụng các bản vá bảo mật và bảo trì. Với kiến ​​trúc serverless và các hàm Lambda, các bản vá bảo mật và nâng cấp  [runtime](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html) nhỏ được AWS tự động xử lý, điều này có nghĩa là khách hàng có thể tập trung vào việc mang lại giá trị kinh doanh thay vì gánh nặng quản lý cơ sở hạ tầng không chuyên biệt.

Khi một phiên bản runtime được quản lý Lambda chính kết thúc vòng đời, AWS sẽ khởi tạo [quy trình ngừng sử dụng](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html#runtime-support-policy) thông qua [AWS Health Dashboard](https://docs.aws.amazon.com/health/latest/ug/aws-health-dashboard-status.html) và gửi email trực tiếp đến các khách hàng bị ảnh hưởng. Vì các runtime đã ngừng sử dụng cuối cùng sẽ mất quyền truy cập vào các bản cập nhật bảo mật và hỗ trợ, các tổ chức phải nâng cấp lên các phiên bản runtime được hỗ trợ để tránh các rủi ro bảo mật tiềm ẩn. Tìm hiểu thêm về  [mô hình chia sẻ trách nhiệm](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html#runtimes-shared-responsibility), [việc sử dụng runtime sau khi ngừng sử dụng](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html#runtime-deprecation-levels) và [nhận thông báo ngừng sử dụng runtime](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html#runtime-deprecation-notify)

Do cơ sở người dùng và độ phức tạp về kiến ​​trúc của Zapier - và do đó số lượng Zap - ngày càng tăng, việc duy trì tất cả các chức năng trên các phiên bản runtime chính mới nhất trở thành một nhiệm vụ tốn nhiều công sức. Các yếu tố góp phần hàng đầu là:

- Số lượng hàm lớn. Vào thời kỳ đỉnh cao, nền tảng Zapier đã chạy Zap bằng hàng trăm nghìn hàm Lambda riêng biệt. Khoảng 35% trong số các hàm này sử dụng một môi trường thời gian chạy đã được lên lịch ngừng hoạt động trong 12 tháng tới.
  
- Zapier đã thiết kế môi trường mặt phẳng dữ liệu của họ theo hướng tạm thời – mặt phẳng điều khiển tạo và xóa các hàm Lambda theo yêu cầu và quản lý vòng đời của chúng một cách linh hoạt. Việc xác định chủ sở hữu cụ thể cho từng hàm bị ảnh hưởng không phải lúc nào cũng đơn giản.
  
- Bảo mật là tối quan trọng tại Zapier và việc nâng cấp thời gian chạy của các hàm bị ảnh hưởng trước ngày ngừng hoạt động là điều bắt buộc. Không có thời điểm nào các hàm Zapier có thể sử dụng thời gian chạy sau ngày ngừng hoạt động. Đây là một nhiệm vụ đòi hỏi thêm tài nguyên.
  
- Quá trình nâng cấp không nên có bất kỳ tác động nào đến trải nghiệm của khách hàng cuối. Không có thời điểm nào trải nghiệm của khách hàng bị ảnh hưởng.

Với thời gian triển khai ngắn, khối lượng công việc lớn và các yêu cầu nghiêm ngặt về việc không ảnh hưởng đến trải nghiệm của khách hàng, nhóm Kỹ thuật Nền tảng của Zapier đã đảm nhận thử thách này để duy trì trạng thái bảo mật cao trong kiến ​​trúc nền tảng của họ.

### Áp dụng giải pháp
Giải pháp này bao gồm ba luồng công việc:
1. Giảm thiểu rủi ro bằng cách phân tích kiến ​​trúc, xác định và dọn dẹp các chức năng không sử dụng.
2. Ưu tiên nâng cấp bằng cách xác định các chức năng quan trọng và có tác động lớn nhất.
3. Trao quyền cho các nhóm kỹ thuật bằng các công cụ và kiến ​​thức tự động để hợp lý hóa quy trình nâng cấp trong tương lai.
   
### Xác định và dọn dẹp các chức năng không sử dụng
Bước đầu tiên trong việc hợp lý hóa quy trình nâng cấp là xác định và loại bỏ các chức năng không sử dụng. Điều này đã giảm tổng số chức năng trong kiến ​​trúc của Zapier cần nâng cấp, loại bỏ khối lượng công việc không cần thiết cho nhóm.
Zapier bắt đầu bằng cách bổ sung thông tin thời gian chạy vào kho chức năng bằng cách sử dụng  [AWS Trusted Advisor](https://aws.amazon.com/premiumsupport/technology/trusted-advisor/) và [Amazon Cloud IntelligenceTrusted Advisor dashboards](https://catalog.workshops.aws/awscid/en-US/dashboards/advanced/trusted-advisor), như minh họa trong sơ đồ sau.

> *Hình 3. *

Điều này có nghĩa là nhóm có thể xây dựng một danh mục chi tiết các chức năng đang chạy trên các môi trường thời gian chạy sắp bị loại bỏ. Sử dụng  [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), nhóm nền tảng của Zapier bắt đầu theo dõi các số liệu như  [số lần gọi](https://docs.aws.amazon.com/lambda/latest/dg/monitoring-metrics-types.html#invocation-metrics). Họ xác định chức năng nào đang hoạt động, chức năng nào không được sử dụng trong thời gian dài và chức năng nào không có chủ sở hữu đang hoạt động và có thể bị xóa.

Một trong những cơ chế chính để xác thực quyền sở hữu trong tổ chức là sử dụng  [thẻ tài nguyên](https://docs.aws.amazon.com/whitepapers/latest/tagging-best-practices/what-are-tags.html). Các chức năng đang hoạt động nhưng không có quyền sở hữu rõ ràng sẽ được gắn cờ để xem xét thêm trước khi xóa. Các chức năng được xác nhận là không sử dụng hoặc không có chủ sở hữu đang hoạt động sẽ được đánh dấu để xóa. Việc xóa các chức năng như vậy cho phép Zapier đơn giản hóa đáng kể kiến ​​trúc của họ và giảm số lượng chức năng cần nâng cấp.

## Ưu tiên nâng cấp
Với số lượng chức năng cần nâng cấp ít hơn, nhóm nền tảng của Zapier đã ưu tiên nâng cấp chức năng dựa trên mô hình sử dụng, mức độ quan trọng và tác động tiềm ẩn đến khách hàng. Ba hạng mục ưu tiên chính là:
- **Chức năng hướng đến khách hàng** – Bất kỳ chức năng nào liên quan trực tiếp đến việc thực thi Zap của người dùng đều được đánh dấu là ưu tiên cao. Những chức năng này phải được nâng cấp trước để tránh gián đoạn dịch vụ.
  
- **Các chức năng cơ sở hạ tầng backend** – Các chức năng nội bộ hỗ trợ hoạt động hệ thống được đánh giá dựa trên tầm quan trọng của chúng đối với sự ổn định của nền tảng.
  
- **Các chức năng khối lượng lớn** – Các chức năng có tần suất thực thi cao nhất được ưu tiên vì việc nâng cấp chúng sẽ có tác động lớn nhất đến việc giảm thiểu rủi ro vận hành.
Sử dụng các yếu tố này, nhóm nền tảng của Zapier đã tạo ra một lộ trình nâng cấp, đảm bảo các tài sản quan trọng được xử lý trước tiên đồng thời giảm thiểu các gián đoạn tiềm ẩn.

Tham khảo mục [Truy xuất dữ liệu về các chức năng Lambda sử dụng thời gian chạy đã lỗi thời](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-list-deprecated.html) trong Hướng dẫn dành cho Nhà phát triển Lambda để tìm hiểu cách xác định các chức năng Lambda được sử dụng phổ biến nhất và thường xuyên nhất trong kiến ​​trúc không máy chủ của bạn.

## Trao quyền cho các nhóm kỹ thuật bằng các công cụ và kiến ​​thức tự động

Để đảm bảo quy trình nâng cấp diễn ra suôn sẻ và hiệu quả trên toàn bộ kiến ​​trúc không máy chủ, nhóm Zapier đã trao quyền cho các nhóm kỹ thuật bằng các hướng dẫn rõ ràng và các giải pháp tự động. Nền tảng kết hợp hai phương pháp chính: các chức năng do Terraform quản lý và một công cụ canary thời gian chạy Lambda được xây dựng riêng. **Việc triển khai và áp dụng các công cụ và phương pháp này đã giúp giảm 95% số lượng chức năng sử dụng các thời gian chạy sắp lỗi thời.**

Đối với các chức năng được quản lý thông qua  [infrastructure-as-code](https://aws.amazon.com/what-is/iac/) (IaC), nhóm Zapier đã phát triển các mô-đun Terraform chuẩn hóa, chỉ định các phiên bản thời gian chạy được hỗ trợ. Các nhóm phát triển đã triển khai các mô-đun này trong cấu hình của họ:

```yaml
resource "aws_lambda_function" "example" {
    runtime = "python3.13"  # Updated to supported runtime
}
```
Sau khi áp dụng phiên bản mô-đun mới, các nhóm đã xác thực các thay đổi bằng cách kiểm tra thời gian chạy mới trong môi trường dàn dựng và theo dõi đầu ra của Terraform plan để đảm bảo cập nhật phiên bản thời gian chạy chính xác.
Để quản lý hiệu quả hầu hết các hàm Lambda trong kiến ​​trúc của mình, Zapier đã phát triển bộ công cụ Lambda runtime canary. Sử dụng giải pháp này, họ đã tự động hóa quy trình nâng cấp thời gian chạy cho hàng nghìn hàm Lambda đang hoạt động với sự can thiệp thủ công tối thiểu. Bộ công cụ triển khai một số tính năng chính:
- **Được thiết kế để chuyển đổi lưu lượng dần dần** với cơ chế định tuyến tích hợp sẵn của Lambda thông qua  [phiên bản](https://docs.aws.amazon.com/lambda/latest/dg/configuration-versions.html) hàm và  [đặt bí danh](https://docs.aws.amazon.com/lambda/latest/dg/configuration-aliases.html). Công cụ có thể chuyển đổi dần dần phân phối lưu lượng từ phiên bản hàm cũ sang phiên bản hàm mới. Trong quá trình chuyển đổi lưu lượng dần dần này, hệ thống sẽ theo dõi các  [số liệu CloudWatch](https://docs.aws.amazon.com/lambda/latest/dg/monitoring-metrics-types.html) để tìm lỗi và tự động khôi phục nếu tỷ lệ lỗi vượt quá ngưỡng chấp nhận được.
- **Chiến lược nâng cấp tối ưu** triển khai nâng cấp trực tiếp cho các hàm ít được sử dụng bằng cách sử dụng giá trị cờ được lưu trữ trong bộ nhớ đệm để phát hiện các sự cố tiềm ẩn trong lần gọi hàm đầu tiên sau khi nâng cấp. Nếu lần gọi này không thành công, mặt phẳng điều khiển sẽ thử lại bằng phiên bản hàm trước đó. Nếu lệnh gọi lại thành công, mặt phẳng điều khiển của Zapier sẽ khởi tạo lệnh khôi phục, giả định rằng lỗi rất có thể là do nâng cấp thời gian chạy. Sau khi khôi phục, nó sẽ ghi lại lỗi và cảnh báo các bên liên quan.
  
- **Việc tích hợp với cơ sở hạ tầng hiện có** sử dụng giao diện quản trị và hàng đợi tác vụ để tự động chuyển đổi lưu lượng. Sổ cái cơ sở dữ liệu duy trì việc theo dõi trạng thái chức năng và thông tin khôi phục.
  
- **Các biện pháp kiểm soát vận hành** cung cấp khả năng khôi phục thủ công và triển khai các công tắc điều khiển tập trung để quản lý quy trình. Sau khi một chức năng được nâng cấp lên thời gian chạy mới và không phát hiện hoạt động khôi phục nào trong khoảng thời gian đã đặt, một tác vụ cắt tỉa tự động sẽ dọn dẹp các phiên bản cũ hơn.
  
Công cụ Lambda canary của Zapier, thông qua việc tích hợp chuyển đổi lưu lượng dần dần, giám sát CloudWatch theo thời gian thực và cơ chế khôi phục tự động, đã thiết lập một khuôn khổ bền vững để quản lý các nâng cấp thời gian chạy trên toàn bộ kiến ​​trúc không máy chủ của họ. Cách tiếp cận này không chỉ tự động hóa quy trình nâng cấp và giảm thiểu rủi ro vận hành mà còn tạo ra một giải pháp có khả năng mở rộng, cung cấp các bản nâng cấp thời gian chạy liên tục, ngăn chặn việc sử dụng các thời gian chạy đã lỗi thời tại bất kỳ thời điểm nào. Bằng cách cho phép cập nhật thời gian chạy hàm liên tục với mức độ gián đoạn tối thiểu đối với trải nghiệm người dùng cuối, Zapier duy trì tính bảo mật và ổn định trong khi yêu cầu can thiệp thủ công tối thiểu. Khung này quản lý hiệu quả cơ sở hạ tầng không máy chủ đang phát triển của họ, mang lại cả tính bảo mật và hiệu quả vận hành cho các bản cập nhật thời gian chạy trong tương lai.

## Kết luận
Trong bài viết này, bạn đã tìm hiểu cách Zapier thiết kế [nền tảng phần mềm dưới dạng dịch vụ](https://aws.amazon.com/what-is/saas/) (SaaS) của mình để cung cấp môi trường thực thi an toàn, biệt lập bằng AWS Lambda và Amazon EKS, cho phép khách hàng tạo ra hàng trăm nghìn Zap. Bạn đã tìm hiểu cách đội ngũ Zapier triển khai quy trình nâng cấp thời gian chạy hàm trên quy mô lớn và giảm 95% số lượng hàm đang chạy trên các thời gian chạy sắp bị loại bỏ. Bạn đã thấy các phương pháp hay nhất đã được thiết lập và các kỹ thuật giúp Zapier duy trì trạng thái bảo mật cao mà không ảnh hưởng đến trải nghiệm của khách hàng.

Sử dụng các liên kết sau để tìm hiểu thêm về thời gian chạy Lambda và nâng cấp các hàm của bạn lên phiên bản thời gian chạy mới nhất:

- [Tài liệu về thời gian chạy Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)
- [Truy xuất dữ liệu về các hàm Lambda sử dụng thời gian chạy đã lỗi thời](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-list-deprecated.html)
- [Quản lý các bản nâng cấp thời gian chạy AWS Lambda](https://aws.amazon.com/blogs/compute/managing-aws-lambda-runtime-upgrades/) trong Blog AWS Compute
- [Các điều khiển quản lý thời gian chạy AWS Lambda](https://aws.amazon.com/blogs/compute/introducing-aws-lambda-runtime-management-controls/) trong Blog AWS Compute



