---
title: "Blog 2"

weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# Vòng đời phát triển dựa trên AI: Tái định hình ngành kỹ thuật phần mềm

Các nhà lãnh đạo kinh doanh và công nghệ luôn nỗ lực để nâng cao năng suất, tăng tốc độ phát triển, thúc đẩy thử nghiệm, rút ngắn thời gian đưa sản phẩm ra thị trường (TTM), và cải thiện trải nghiệm của lập trình viên. Những mục tiêu “ngôi sao dẫn đường” này thúc đẩy sự đổi mới trong các phương pháp phát triển phần mềm. Sự đổi mới này ngày càng được thúc đẩy bởi trí tuệ nhân tạo. Đặc biệt, các công cụ AI tạo sinh như [Amazon Q Developer](https://aws.amazon.com/q/developer/) và [Kiro](https://kiro.dev/) đã bắt đầu cách mạng hóa cách phần mềm được tạo ra. Hiện tại, các tổ chức đang ứng dụng AI vào phát triển phần mềm theo hai hướng chính: phát triển hỗ trợ bởi AI, nơi AI hỗ trợ các nhiệm vụ cụ thể như viết tài liệu, gợi ý mã và kiểm thử; và phát triển tự động hóa bởi AI, nơi AI được kỳ vọng tạo ra toàn bộ ứng dụng dựa trên yêu cầu người dùng mà không cần sự can thiệp của con người. Cả hai hướng tiếp cận này đều cho kết quả chưa tối ưu về tốc độ và chất lượng phần mềm — và đây chính là vấn đề mà AI-DLC đặt mục tiêu giải quyết.

## Tại sao chúng ta cần một cách tiếp cận mang tính chuyển đổi đối với AI trong phần mềm?

Phương pháp phát triển phần mềm hiện tại được xây dựng cho các quy trình dài hạn, do con người dẫn dắt, nơi product owner, developer và architect dành phần lớn thời gian cho các hoạt động không cốt lõi như lập kế hoạch, họp hành, và thực hiện các nghi thức trong vòng đời phát triển phần mềm (SDLC). Việc chỉ đơn thuần tích hợp AI như một trợ lý không chỉ hạn chế khả năng của nó mà còn củng cố những điểm kém hiệu quả đã lỗi thời. Để thực sự khai thác sức mạnh của AI và đạt được các mục tiêu năng suất mang tính chiến lược, chúng ta cần tái hình dung toàn bộ cách tiếp cận vòng đời phát triển phần mềm.

Để đạt được kết quả mang tính đột phá, chúng ta cần đặt AI vào vai trò cộng tác viên và đồng đội trung tâm trong quy trình phát triển, đồng thời tận dụng khả năng của nó xuyên suốt vòng đời phần mềm. Đây là lý do chúng tôi giới thiệu [AI-Driven Development Lifecycle (AI-DLC)](https://prod.d13rzhkk8cj2z0.amplifyapp.com/) — một phương pháp mới được thiết kế nhằm tích hợp hoàn toàn khả năng của AI vào mọi khía cạnh của quá trình phát triển phần mềm.

## AI Driven Development Life Cycle (AI-DLC) là gì?
AI-DLC là một cách tiếp cận mới trong phát triển phần mềm, lấy AI làm trung tâm và tập trung vào hai yếu tố quan trọng:
- Thực thi được hỗ trợ bởi AI với sự giám sát của con người:
 AI xây dựng kế hoạch làm việc chi tiết một cách có hệ thống, chủ động yêu cầu làm rõ và hướng dẫn khi cần, đồng thời chuyển các quyết định quan trọng cho con người. Điều này rất cần thiết bởi chỉ con người mới có sự hiểu biết sâu về bối cảnh và yêu cầu kinh doanh để đưa ra lựa chọn chính xác.
- Cộng tác nhóm năng động:
 Trong khi AI xử lý các nhiệm vụ thường xuyên và tẻ nhạt, đội ngũ con người tập trung làm việc trong môi trường cộng tác để giải quyết vấn đề theo thời gian thực, sáng tạo và đưa ra quyết định nhanh chóng. Sự chuyển dịch từ làm việc riêng lẻ sang hợp tác năng lượng cao giúp tăng tốc đổi mới và tốc độ triển khai.

![](/images/3-BlogsTranslated/3.2-blog2/aidlc-image01.png.png)

Hai yếu tố này cho phép bạn phát triển phần mềm nhanh hơn mà không phải đánh đổi chất lượng.

## AI-DLC hoạt động như thế nào?

Cốt lõi của AI-DLC là AI bắt đầu và điều hướng quy trình làm việc thông qua một mô hình tư duy mới:
![](/images/3-BlogsTranslated/3.2-blog2/aidlc-image02.png)
Mô hình này hoạt động theo khuôn mẫu: AI tạo ra kế hoạch, đặt câu hỏi làm rõ để hiểu bối cảnh, và chỉ triển khai giải pháp sau khi nhận được xác nhận từ con người. Quy trình này được lặp lại liên tục cho mọi hoạt động trong vòng đời phát triển phần mềm (SDLC), giúp tạo ra một tầm nhìn thống nhất và phương pháp tiếp cận nhất quán cho tất cả các quy trình phát triển.

Với mô hình tư duy này làm nền tảng, quá trình phát triển phần mềm trong AI-DLC diễn ra qua ba giai đoạn đơn giản:

- **Giai đoạn Khởi tạo (Inception):**
AI chuyển đổi mục tiêu kinh doanh thành yêu cầu chi tiết, user stories và các đơn vị công việc thông qua “Mob Elaboration” – nơi cả nhóm cùng tham gia xác nhận câu hỏi và đề xuất của AI.
- **Giai đoạn Xây dựng (Construction):**
Dựa trên bối cảnh đã được xác thực ở giai đoạn Khởi tạo, AI đề xuất kiến trúc logic, mô hình miền (domain models), giải pháp mã nguồn và kiểm thử thông qua “Mob Construction” – nơi nhóm đưa ra làm rõ về các quyết định kỹ thuật và lựa chọn kiến trúc theo thời gian thực.
- **Giai đoạn Vận hành (Operations):**
AI sử dụng toàn bộ bối cảnh đã tích lũy từ các giai đoạn trước để quản lý hạ tầng dưới dạng mã (infrastructure as code) và triển khai hệ thống, với sự giám sát của đội ngũ.

Mỗi giai đoạn cung cấp ngữ cảnh phong phú hơn cho giai đoạn tiếp theo, giúp AI đưa ra các đề xuất ngày càng chính xác và hiệu quả hơn.

![](/images/3-BlogsTranslated/3.2-blog2/aidlc-image03.png)

AI lưu trữ và duy trì ngữ cảnh liên tục xuyên suốt các giai đoạn bằng cách lưu kế hoạch, yêu cầu và các tài liệu thiết kế vào kho dự án của bạn, giúp công việc được tiếp tục liền mạch qua nhiều phiên làm việc khác nhau.

AI-DLC giới thiệu thuật ngữ và nghi thức mới để phản ánh cách tiếp cận cộng tác cao và do AI dẫn dắt. Các “sprint” truyền thống được thay thế bằng “bolt” – các chu kỳ làm việc ngắn hơn, cường độ cao hơn, tính theo giờ hoặc ngày thay vì tuần; các Epics được thay thế bằng Units of Work. Sự thay đổi thuật ngữ này nhấn mạnh trọng tâm của phương pháp: tốc độ và khả năng triển khai liên tục. Tương tự, các thuật ngữ Agile quen thuộc khác cũng được tái định nghĩa để phù hợp với quy trình lấy AI làm trung tâm, tạo ra một hệ thống ngôn ngữ phản ánh đúng phương pháp đổi mới trong phát triển phần mềm này.

## Lợi ích của phương pháp này là gì?

- **Tốc độ (Velocity):** Lợi ích quan trọng nhất mà AI-DLC mang lại là tăng tốc độ phát triển. AI nhanh chóng tạo và tinh chỉnh các tài liệu như yêu cầu, user stories, thiết kế, mã nguồn và kiểm thử, giúp product owner, kiến trúc sư và lập trình viên hoàn thành công việc trong vài giờ hoặc vài ngày thay vì hàng tuần như trước.
- **Đổi mới (Innovation):** Nhờ AI đảm nhiệm phần lớn khối lượng công việc nặng, đội ngũ có thêm thời gian tập trung vào đổi mới, khám phá các giải pháp sáng tạo và mở rộng giới hạn của những gì có thể thực hiện.
- **Chất lượng (Quality):** Thông qua việc liên tục làm rõ yêu cầu, đội ngũ xây dựng chính xác những gì họ mong muốn thay vì một phiên bản diễn giải mơ hồ của AI. Kết quả là sản phẩm phù hợp hơn với mục tiêu kinh doanh. AI nâng cao chất lượng bằng cách nhất quán áp dụng các tiêu chuẩn riêng của tổ chức — như quy tắc viết mã, mẫu thiết kế, yêu cầu bảo mật — đồng thời tạo ra bộ kiểm thử toàn diện. Tích hợp AI xuyên suốt quy trình giúp cải thiện tính thống nhất và khả năng truy vết từ yêu cầu đến triển khai.
- **Khả năng phản ứng với thị trường (Market Responsiveness)**: Nhờ chu kỳ phát triển nhanh, AI-DLC cho phép phản hồi nhanh với nhu cầu thị trường và phản hồi của người dùng, từ đó thích ứng với yêu cầu hiệu quả hơn.
- **Trải nghiệm lập trình viên (Developer Experience):** AI-DLC thay đổi trải nghiệm của lập trình viên bằng cách chuyển trọng tâm từ việc viết mã lặp đi lặp lại sang giải quyết các vấn đề cốt lõi. AI giảm gánh nặng nhận thức bằng cách xử lý công việc thủ công, trong khi lập trình viên có được hiểu biết sâu hơn về nghiệp vụ và thấy rõ tác động trực tiếp của công việc họ đối với giá trị kinh doanh — từ đó tăng mức độ hài lòng.

## Làm thế nào để bắt đầu với phương pháp này?

Hãy bắt đầu hành trình AI-DLC của bạn thông qua ba hướng tiếp cận rõ ràng: đọc bản [white paper chi tiết về AI-DLC](https://prod.d13rzhkk8cj2z0.amplifyapp.com/), tìm hiểu cách [Amazon Q Developer rules](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/context-project-rules.html) và [các quy trình tùy chỉnh Kiro](https://kiro.dev/docs/steering/) có thể giúp bạn triển khai AI-DLC một cách nhất quán trong tổ chức, hoặc liên hệ với đội ngũ AWS của bạn để trao đổi về cách điều chỉnh AI-DLC phù hợp với nhu cầu đặc thù của tổ chức.

Tương lai của phát triển phần mềm đã bắt đầu. Chúng tôi rất háo hức được giúp bạn tận dụng AI không chỉ để xây dựng hệ thống nhanh hơn mà còn đảm bảo tính chính xác và chất lượng thông qua sự giám sát và cộng tác quan trọng của con người. Hãy bắt đầu hành trình AI-DLC của bạn ngay hôm nay và tham gia cộng đồng ngày càng lớn mạnh của các tổ chức đang chuyển đổi phương pháp phát triển của họ thông qua đổi mới do AI dẫn dắt.