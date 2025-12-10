---
title: "Blog 5"

weight: 1
chapter: false
pre: " <b> 3.6. </b> "
---


# Cách HashiCorp thực hiện chuyển đổi liên vùng liền mạch với Amazon Application Recovery Controller 

Tác giả: Dmitriy Novikov

Xuất bản: Ngày 25 JUL 2025

[Amazon Application Recovery Controller (ARC)](https://aws.amazon.com/blogs/architecture/category/networking-content-delivery/amazon-application-recovery-controller/), [Customer Solutions](https://aws.amazon.com/blogs/architecture/category/post-types/customer-solutions/) 

*Bài viết này được đồng sáng tác bởi Brandon Raabe, Kỹ sư Cao cấp về Độ tin cậy của Site tại HashiCorp.*

Trong các hệ thống đám mây, chỉ vài phút ngừng hoạt động cũng có thể gây ra tác động đáng kể đến hoạt động kinh doanh và làm xói mòn niềm tin của khách hàng. [HashiCorp](https://www.hashicorp.com/en), một công ty hàng đầu về phần mềm tự động hóa cơ sở hạ tầng đa đám mây, đã phải đối mặt với thách thức quan trọng này khi Nền tảng Đám mây HashiCorp (HCP) của họ được mở rộng quy mô để phục vụ khách hàng doanh nghiệp với các yêu cầu nghiêm ngặt về tính khả dụng. Khi sự cố gián đoạn cục bộ đe dọa tính liên tục của dịch vụ, quá trình chuyển đổi dự phòng phức tạp giữa các mục DNS, khối lượng công việc và cơ sở dữ liệu trên khắp các Vùng AWS  đã trở thành một quy trình dễ xảy ra lỗi, đòi hỏi sự phối hợp chặt chẽ. Bài viết này ghi lại cách nhóm Kỹ thuật Độ tin cậy Site (SRE) của HashiCorp đã chuyển đổi khả năng phục hồi sau thảm họa bằng cách triển khai [Bộ điều khiển Khôi phục Ứng dụng Amazon](https://aws.amazon.com/application-recovery-controller/) (ARC), tạo ra một giải pháp không chỉ đơn giản hóa đáng kể việc chuyển đổi dự phòng giữa các Vùng mà còn cung cấp một phương thức chuẩn hóa để truyền tải ngữ cảnh Vùng đến các dịch vụ phân tán của họ.

Trong bài viết này, chúng tôi thảo luận về hành trình của HashiCorp từ các quy trình chuyển đổi dự phòng thủ công, gây căng thẳng sang phương pháp hợp lý, tự tin, giúp thay đổi căn bản cách họ thực hiện các cam kết về khả năng phục hồi cấp doanh nghiệp.

## Thách thức với phục hồi sau thảm họa trong cơ sở hạ tầng đa đám mây
Đội ngũ SRE của HashiCorp nhận thấy rằng khi nền tảng đám mây của họ được mở rộng để phục vụ các khối lượng công việc quan trọng của doanh nghiệp, phương pháp phục hồi sau thảm họa của họ cần được nâng cấp. Các quy trình thủ công hiện tại đòi hỏi sự phối hợp chính xác trên nhiều hệ thống trong các tình huống ngừng hoạt động vốn đã căng thẳng, điều này có thể dẫn đến các biến chứng tiềm ẩn khi tốc độ và độ chính xác là yếu tố quan trọng nhất. Sự cố ngừng hoạt động theo khu vực đặt ra những thách thức đặc biệt: nếu các mặt phẳng điều khiển cho các dịch vụ quan trọng không khả dụng, chính các công cụ cần thiết để thực hiện phục hồi có thể không truy cập được.

ARC nổi lên như một giải pháp lý tưởng với kiến ​​trúc độc đáo: một mặt phẳng dữ liệu có tính khả dụng cao có thể truy cập thông qua các điểm cuối trong năm Khu Vực riêng biệt, do đó cơ chế phục hồi vẫn hoạt động ngay cả trong những gián đoạn đáng kể của Khu Vực. Bằng cách sử dụng [AWS SDK](https://builder.aws.com/build/tools) để giao tiếp với ARC, HashiCorp đã đạt được một số lợi thế quan trọng. Họ có thể áp dụng các phương pháp cơ sở hạ tầng dưới dạng code (IaC) vào quy trình phục hồi sau thảm họa, tự động hóa việc kiểm tra các quy trình chuyển đổi dự phòng và tích hợp khả năng phục hồi liền mạch với các công cụ vận hành hiện có. Giải pháp này đã chuyển đổi quy trình phục hồi sau thảm họa của họ từ một quy trình thủ công chuyên biệt thành một quy trình được mã hóa, có thể lặp lại được tích hợp trong các hoạt động nền tảng của họ.

## Yêu cầu và cân nhắc về kiến ​​trúc
Sau khi đánh giá nhiều phương pháp phục hồi sau thảm họa, HashiCorp đã thiết lập ba yêu cầu cốt lõi cho giải pháp của mình. Thứ nhất, trong khi vẫn duy trì sự phán đoán của con người khi khởi tạo chuyển đổi dự phòng, quá trình thực thi cần được tiến hành mà không cần sự can thiệp bổ sung của người vận hành sau khi nó được kích hoạt. Thiết kế vòng lặp con người này bảo toàn việc ra quyết định có chủ đích, đồng thời giảm thiểu các bước thủ công dễ xảy ra lỗi trong quá trình triển khai.

Thứ hai, kiến ​​trúc cần có khả năng phục hồi vượt trội trước chính những lỗi mà nó được thiết kế để giảm thiểu. Các giải pháp chuyển đổi dự phòng DNS truyền thống bộc lộ một lỗ hổng nghiêm trọng: sự phụ thuộc vào các mặt phẳng điều khiển Vùng đơn lẻ có thể không khả dụng trong thời gian ngừng hoạt động. ARC đã giải quyết vấn đề này thông qua kiến ​​trúc phân tán, kết nối [Amazon Route 53](https://aws.amazon.com/route53/) với một cơ chế điều khiển linh hoạt, được kích hoạt bởi các kiểm tra tình trạng Route 53, có thể truy cập thông qua nhiều điểm cuối Vùng. Điều này có nghĩa là bản thân hệ thống chuyển đổi dự phòng vẫn khả dụng ngay cả khi Vùng chính ngừng hoạt động.

Thứ ba, giải pháp cần đáp ứng hoặc vượt quá các chỉ số Mục tiêu Điểm Phục hồi (RPO) và Mục tiêu Thời gian Phục hồi (RTO) hiện có của HashiCorp - ngưỡng mất dữ liệu và thời gian ngừng hoạt động tối đa có thể chấp nhận được. Sử dụng ARC, nhóm SRE đã lên kế hoạch không chỉ đạt được các mục tiêu này mà còn tạo ra những cải tiến đáng kể, giảm thiểu tác động tiềm ẩn đến khách hàng trong các sự kiện Khu vực và củng cố khả năng phục hồi cấp doanh nghiệp của HashiCorp.

## Tổng quan về giải pháp
Để chuyển đổi tư thế phục hồi sau thảm họa, nhóm SRE của HashiCorp đã thiết kế một kiến ​​trúc tập trung vào ARC và được bổ sung bởi một dịch vụ điều phối được xây dựng riêng. Kiến trúc này kết nối liền mạch quyết định khởi tạo chuyển đổi dự phòng của con người với các hoạt động kỹ thuật phức tạp cần thiết để chuyển đổi lưu lượng giữa các Khu vực với sự gián đoạn tối thiểu.

Trọng tâm của giải pháp là một dịch vụ chuyển đổi dự phòng tùy chỉnh, đóng vai trò là lớp điều phối cho các chuyển đổi Khu vực. Dịch vụ này duy trì chi tiết cấu hình cho cụm ARC và cung cấp một giao diện duy nhất, được kiểm soát để khởi tạo các chuyển đổi Khu vực. Khi được kích hoạt, dịch vụ sẽ thiết lập kết nối an toàn đến các điểm cuối API ARC và thực hiện quy trình làm việc hai bước: đầu tiên là vô hiệu hóa các điều khiển định tuyến cho Khu vực chính, sau đó bật các điều khiển đó cho Khu vực phụ. Cách tiếp cận tuần tự này cung cấp một quá trình chuyển đổi lưu lượng sạch mà không có các tình huống phân chia bộ não hoặc mất kết nối.

Kiến trúc DNS đã trải qua một quá trình phát triển chiến lược để hỗ trợ khả năng mới này. HashiCorp đã cấu hình lại các điểm cuối đầu vào quan trọng của họ thành các cặp bản ghi dự phòng Route 53, mỗi cặp bao gồm một bản ghi chính và một bản ghi phụ. Mỗi bản ghi được liên kết với một kiểm tra tình trạng để theo dõi trạng thái của một điều khiển định tuyến ARC—kết nối hiệu quả dịch vụ DNS toàn cầu của AWS với điều khiển định tuyến ARC. Các bản ghi chính phân giải đến các điểm cuối trong Vùng chính, và các bản ghi phụ trỏ đến cơ sở hạ tầng tương ứng trong Vùng dự phòng. Khi các điều khiển định tuyến thay đổi trạng thái, các kiểm tra tình trạng liên quan sẽ tự động kích hoạt Route 53 để điều chỉnh các mẫu phân giải DNS, chuyển hướng lưu lượng đến cơ sở hạ tầng Vùng phù hợp.

HashiCorp duy trì Vùng phụ của mình ở cấu hình dự phòng ấm, với các dịch vụ thiết yếu đang chạy nhưng không chủ động phục vụ lưu lượng máy khách cho đến khi xảy ra sự kiện chuyển đổi dự phòng. Để đảm bảo nhận thức liền mạch về trạng thái Vùng trên toàn hệ thống phân tán, nhóm đã triển khai cơ chế báo hiệu sử dụng các bản ghi DNS TXT được thiết kế đặc biệt. Các bản ghi này được liên kết với cùng các điều khiển định tuyến ARC như các điểm cuối dịch vụ chính, tạo ra một chỉ báo trạng thái toàn cầu có thể khám phá. Các dịch vụ có thể truy vấn các bản ghi TXT này để xác định động Vùng hiện đang hoạt động và điều chỉnh định tuyến nội bộ, sao chép và hành vi vận hành của chúng cho phù hợp — giảm nhu cầu về một hệ thống phân phối cấu hình riêng biệt và đảm bảo tất cả các thành phần đều có cái nhìn nhất quán về trạng thái hiện tại của Vùng.

Sơ đồ sau minh họa quy trình phục hồi sau thảm họa.

> *Image 1. *
>
> 
Kiến trúc này kết hợp sự giám sát của con người để khởi tạo các chuyển đổi khu vực quan trọng với việc thực thi hoàn toàn tự động sau khi quyết định được đưa ra. Việc sử dụng mặt phẳng điều khiển phân tán toàn cầu của ARC loại bỏ các phụ thuộc vào một khu vực duy nhất, nếu không có thể gây ảnh hưởng đến cơ chế chuyển đổi dự phòng trong trường hợp mất điện khu vực.

## Khung quyết định vận hành cho chuyển đổi dự phòng khu vực
Quy trình chuyển đổi dự phòng khu vực của HashiCorp cân bằng giữa việc giám sát tự động với việc ra quyết định có chủ đích của con người. Nền tảng quan sát toàn diện của họ liên tục theo dõi tình trạng của khu vực, tự động cảnh báo nhóm ứng phó sự cố khi phát hiện bất thường. Khi cảnh báo được kích hoạt, giao thức quản lý sự cố sẽ được kích hoạt, với một chỉ huy sự cố nhanh chóng tập hợp các chuyên gia để đánh giá tình hình.

Nhóm tuân theo một khung đánh giá có cấu trúc để xác định xem việc chuyển đổi dự phòng có cần thiết hay không: xác nhận sự cố là đặc thù của khu vực, xác minh rằng các thành phần dự phòng trong khu vực không thể giảm thiểu sự cố và đánh giá xem thời gian phục hồi dự kiến ​​của khu vực có vượt quá ngưỡng tác động chấp nhận được của khách hàng hay không. Cách tiếp cận này ngăn chặn các chuyển đổi khu vực không cần thiết đồng thời cung cấp hành động nhanh chóng khi thực sự cần thiết.

Sau khi quyết định chuyển đổi dự phòng được đưa ra, một nhà điều hành được ủy quyền sẽ khởi tạo quy trình thông qua một lệnh gọi API duy nhất đến dịch vụ điều phối của họ, sau đó dịch vụ này sẽ giao tiếp với ARC để thực hiện chuỗi thay đổi điều khiển định tuyến phức tạp. Thiết kế này bảo toàn sự phán đoán của con người trong các quyết định quan trọng, đồng thời sử dụng tự động hóa để thực hiện chính xác, nhờ đó HashiCorp có thể phản ứng một cách tự tin và nhất quán trong các tình huống mất điện cục bộ áp lực cao.

## Kiểm thử phục hồi sau thảm họa
HashiCorp duy trì khả năng sẵn sàng hoạt động thông qua chương trình kiểm thử phục hồi sau thảm họa hàng tháng được tổ chức bài bản trong môi trường tích hợp của họ. Một tuần trước mỗi bài kiểm tra theo lịch trình, nhóm sẽ thông báo cho tất cả các bên liên quan để xác nhận nhận thức và sự tham gia của toàn bộ tổ chức. Vào ngày kiểm tra, họ tuân thủ các giao thức sự cố chính thức, tạo ra các kênh liên lạc chuyên dụng để quan sát và cộng tác minh bạch.

Việc thực hiện kiểm thử phản ánh quy trình chuyển đổi dự phòng sản xuất của họ: người vận hành khởi tạo chuỗi phục hồi thông qua API của họ, kích hoạt các điều khiển định tuyến ARC để chuyển lưu lượng sang Vùng thứ cấp. Điểm khác biệt của phương pháp tiếp cận của HashiCorp là phương pháp xác thực toàn diện. Nhóm kiểm thử xác minh các dịch vụ quan trọng trong Vùng thứ cấp và sau đó chuyển đổi dự phòng trở lại Vùng chính với xác thực tiếp theo. Kiểm thử hai chiều này xác nhận cả quy trình chuyển đổi dự phòng và quay lại dự phòng đều hoạt động đáng tin cậy.

Mỗi bài tập kết thúc bằng một cuộc hồi cứu có cấu trúc, trong đó nhóm ghi lại các quan sát và xác định các cơ hội cải tiến. Bằng cách coi những bài kiểm tra này là kinh nghiệm học tập hơn là các hoạt động tuân thủ, HashiCorp đã thiết lập một chu trình cải tiến liên tục cho khả năng phục hồi sau thảm họa của họ. Những hiểu biết sâu sắc từ các cuộc tập trận thường xuyên này đã dẫn đến nhiều cải tiến trong việc triển khai ARC và quy trình vận hành, do đó nhóm của họ có thể phản ứng tự tin trong các sự cố mất điện thực tế bằng các quy trình đã được thực hành và có thể dự đoán được.

## Kết luận
Sự ​​hợp tác giữa HashiCorp và AWS thông qua ARC đã cách mạng hóa khả năng phục hồi sau thảm họa của HashiCorp. Việc chuyển đổi khu vực trước đây đòi hỏi việc thao tác bản ghi DNS cẩn thận bởi các nhà điều hành chuyên biệt giờ đây được thực hiện thông qua một lệnh gọi API duy nhất, với việc chuyển đổi lưu lượng trong vòng vài giây và việc truyền tải hoàn tất trong khoảng 2 phút. Sự đơn giản hóa đáng kể này, đạt được nhờ tích hợp kiến ​​trúc ARC linh hoạt với dịch vụ điều phối tùy chỉnh của HashiCorp, không chỉ cải thiện các chỉ số phục hồi mà còn củng cố các cam kết về khả năng phục hồi cấp doanh nghiệp của họ.

ARC đã giải quyết một thách thức cơ bản của hệ thống phân tán bằng cách cung cấp một cơ chế đáng tin cậy để các dịch vụ xác định Khu vực đang hoạt động. Bằng cách liên kết các điều khiển định tuyến ARC với các bản ghi TXT chuyên biệt, HashiCorp đã tạo ra một chỉ báo toàn cầu nhất quán cho phép các dịch vụ tự động điều chỉnh hành vi của chúng mà không cần các hệ thống điều phối bổ sung—đơn giản hóa kiến ​​trúc và giảm sự phụ thuộc.

Quan trọng nhất, việc triển khai này đã dân chủ hóa khả năng phục hồi sau thảm họa trong HashiCorp, chuyển đổi nó từ một khả năng chuyên biệt thành một quy trình chuẩn hóa có thể thực thi thông qua việc luân phiên trực thường xuyên. Các điểm cuối có tính khả dụng cao của giải pháp trên nhiều Khu vực đảm bảo cơ chế phục hồi vẫn hoạt động ngay cả trong những sự cố ngừng hoạt động nghiêm trọng - giải quyết một lỗ hổng nghiêm trọng trong phương pháp tiếp cận trước đây của họ.

Đối với khách hàng doanh nghiệp của HashiCorp, những cải tiến này chuyển trực tiếp thành giá trị kinh doanh: giảm thời gian phục hồi trong các sự kiện Khu vực, tăng cường sự tự tin vận hành và đảm bảo rằng các công cụ quản lý cơ sở hạ tầng quan trọng của họ sẽ luôn khả dụng ngay cả trong những sự cố gián đoạn đám mây lớn. Khi HashiCorp tiếp tục tinh chỉnh phương pháp tiếp cận của mình thông qua thử nghiệm nghiêm ngặt và cải tiến liên tục, việc triển khai ARC của họ chứng minh cách phục hồi thảm họa được thiết kế chu đáo có thể phát triển từ một chính sách bảo hiểm đơn thuần thành một lợi thế cạnh tranh chiến lược.

Để tìm hiểu thêm, hãy truy cập [Amazon Application Recovery Controller](https://aws.amazon.com/application-recovery-controller/), [AWS Multi-Region Capabilities](https://repost.aws/articles/AR02pJIdoARYKX6Rhkdra-Zg/aws-multi-region-capabilities) và [AWS Multi-Region Fundamentals](https://docs.aws.amazon.com/prescriptive-guidance/latest/aws-multi-region-fundamentals/introduction.html).


