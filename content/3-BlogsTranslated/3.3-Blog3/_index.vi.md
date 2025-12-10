---
title: "Blog 3"

weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# Khắc phục sự rối loạn trong phát triển với các tác nhân tùy chỉnh của Amazon Q Developer CLI.

Là một nhà phát triển đã tận dụng sức mạnh của Giao thức [Ngữ cảnh Mô hình (MCP)](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-mcp-config-CLI.html) để nâng cao quy trình làm việc, tôi vô cùng hào hứng khi thấy [Amazon Q Developer CLI bổ sung các tác nhân tùy chỉnh (custom agents)](https://aws.amazon.com/about-aws/whats-new/2025/07/amazon-q-developer-cli-custom-agents/). Tính năng mới này đưa các khả năng mà tôi đã quen thuộc lên một tầm cao mới, cho phép tôi quản lý các ngữ cảnh phát triển khác nhau một cách liền mạch và dễ dàng chuyển đổi giữa chúng.

[Trong bài viết trước](https://aws.amazon.com/blogs/devops/extend-the-amazon-q-developer-cli-with-mcp/), tôi đã thảo luận về cách các máy chủ MCP đã cách mạng hóa phương thức tôi tương tác với các dịch vụ AWS, cơ sở dữ liệu và các công cụ thiết yếu khác. Việc tích hợp MCP vào Amazon Q Developer cho phép tôi truy vấn các lược đồ cơ sở dữ liệu, tự động hóa việc triển khai cơ sở hạ tầng, và nhiều hơn thế nữa. Tuy nhiên, khi tôi bắt đầu xử lý nhiều dự án, mỗi dự án lại có các stack công nghệ và yêu cầu riêng biệt, tôi nhận thấy mình cần một phương pháp tiếp cận có cấu trúc hơn để quản lý các môi trường phát triển đa dạng này.

Đây chính là lúc các tác nhân tùy chỉnh xuất hiện. Với tính năng mới này, tôi có thể tạo và sử dụng một tác nhân tùy chỉnh bằng cách kết hợp các công cụ, lời nhắc (prompt), ngữ cảnh và quyền công cụ cụ thể cho các tác vụ phù hợp với từng giai đoạn phát triển. Trong bài viết này, tôi sẽ giải thích cách cấu hình một tác nhân tùy chỉnh cho phát triển front-end và back-end, giúp tôi dễ dàng tối ưu hóa Amazon Q Developer cho từng nhiệm vụ.

## Bối Cảnh

Hãy tưởng tượng tôi đang làm việc trên một ứng dụng web đa tầng. Ứng dụng này có giao diện front-end React viết bằng Typescript và giao diện back-end FastAPI viết bằng Python. Ngoài tôi ra, nhóm còn có một nhà thiết kế sử dụng Figma và một quản trị viên cơ sở dữ liệu quản lý database PostgreSQL. Có những khác biệt tinh tế trong cách tôi giao tiếp với nhà thiết kế và quản trị viên cơ sở dữ liệu. Ví dụ, khi tôi thảo luận về một "table" (bảng) với nhà thiết kế, tôi có khả năng đang đề cập đến một bảng HTML và cách trang web được cấu trúc. Tuy nhiên, khi tôi thảo luận về một "table" với quản trị viên cơ sở dữ liệu, tôi có khả năng đang nói về một bảng SQL và cách dữ liệu được lưu trữ.

Trong quá khứ, tôi đã cấu hình cả [máy chủ Figma Dev Mode MCP](https://www.figma.com/blog/introducing-figmas-dev-mode-mcp-server/) và [máy chủ Amazon Aurora PostgreSQL MCP](https://github.com/awslabs/mcp/blob/main/src/postgres-mcp-server) trong môi trường của mình. Mặc dù điều này cho phép tôi dễ dàng làm việc với mã front-end hoặc back-end, nhưng nó đã tạo ra một số thách thức. Nếu tôi hỏi Amazon Q Developer: "Tôi có bao nhiêu tables?", Amazon Q Developer sẽ phải đoán xem tôi đang nói về bảng HTML hay bảng SQL. Nếu câu hỏi là về HTML, nó nên sử dụng máy chủ Figma. Nếu câu hỏi là về SQL, nó nên sử dụng máy chủ Aurora. Đây không phải là một hạn chế kỹ thuật, mà là một hạn chế về ngôn ngữ. Giống như việc tôi phải điều chỉnh các giả định của mình khi nói chuyện với nhà thiết kế và quản trị viên cơ sở dữ liệu, Amazon Q Developer cũng phải thực hiện những điều chỉnh tương tự.

Giải pháp là các tác nhân tùy chỉnh (custom agents) của Amazon Q Developer CLI. Các tác nhân tùy chỉnh cho phép tôi tối ưu hóa cấu hình của Q Developer cho từng tình huống. Hãy cùng tìm hiểu qua cấu hình front-end và back-end của tôi để hiểu rõ tác động.

## Tác Nhân Front-end

Tác nhân tùy chỉnh front-end của tôi được tối ưu hóa cho việc phát triển web front-end sử dụng React và Figma. Ví dụ mã sau đây là cấu hình cho tác nhân front-end của tôi, được lưu trữ tại `~/.aws/amazonq/cli-agents/front-end.json`. Hãy cùng thảo luận về các phần chính của cấu hình này.
- `mcpServers` – Tại đây, tôi đã cấu hình Máy chủ Figma Dev Mode MCP. Máy chủ này đơn giản là giao tiếp với Ứng dụng Thiết kế Web Figma được cài đặt cục bộ. Lưu ý rằng điều này sẽ thay thế cấu hình MCP đã được lưu trữ trong` ~/.aws/amazonq/mcp.json`.
- `tools` và `allowedTools` – Hai phần này có liên quan đến nhau, vì vậy tôi sẽ thảo luận chúng cùng lúc. tools định nghĩa các công cụ có sẵn cho Amazon Q Developer trong khi `allowedTools` định nghĩa các công cụ được tin cậy, nói cách khác, Q Developer có thể sử dụng tất cả các công cụ đã được cấu hình, và nó không cần phải hỏi ý kiến tôi để sử dụng `fs_read`, `fs_write`, và `@Figma`. `@Figma` cho phép Amazon Q Developer sử dụng tất cả các công cụ Figma mà không cần hỏi xin phép. Chi tiết hơn sẽ có trong phần tiếp theo.
- `resources` – Tại đây tôi đã cấu hình các tệp nên được thêm vào ngữ cảnh (context). Tôi đã bao gồm tệp README.md (được lưu trữ trong thư mục dự án) và các tùy chọn riêng của tôi cho React (được lưu trữ trong hồ sơ cá nhân). Bạn có thể đọc thêm trong phần [quản lý ngữ cảnh của hướng dẫn sử dụng](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-context-profiles.html).
- hooks – Ngoài các tài nguyên, tôi cũng đã bao gồm một hook. Hook này sẽ chạy một lệnh và chèn kết quả của nó vào ngữ cảnh tại thời điểm chạy (runtime). Trong ví dụ này, tôi đang thêm `git status` hiện tại. Bạn có thể đọc thêm trong phần [hook ngữ cảnh của hướng dẫn sử dụng](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line-context-hooks.html).
```bash
{
  "name": "front-end",
  "description": "Optimized for front-end web development using React and Figma",
  "mcpServers": {
    "Figma": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "http://127.0.0.1:3845/sse"
      ]
    }
  },
  "tools": ["*"],
  "allowedTools": [
    "fs_read",
    "fs_write",
    "report_issues",
    "@Figma"
  ],
  "resources": [
    "file://README.md",
    "file://~/.aws/amazonq/react-preferences.md"
  ],
  "hooks": {
    "agentSpawn": [
      {
        "command": "git status"
      }
    ]
  }
}
```

## Tác Nhân Back-end

Tác nhân tùy chỉnh back-end của tôi được tối ưu hóa cho việc phát triển back-end với Python và PostgreSQL. Ví dụ mã sau đây là cấu hình cho tác nhân back-end của tôi, được lưu trữ tại `~/.aws/amazonq/cli-agents/back-end.json`. Thay vì mô tả lại các phần như đã làm trước đó, tôi sẽ tập trung vào sự khác biệt giữa tác nhân front-end và back-end.
- `mcpServers` – Ở đây, tôi đã cấu hình [Máy chủ Amazon Aurora PostgreSQL MCP](https://github.com/awslabs/mcp/blob/main/src/postgres-mcp-server). Điều này cho phép Amazon Q Developer truy vấn cơ sở dữ liệu `dev` của tôi để tìm hiểu về lược đồ (schema). Hãy chú ý rằng tôi đã cấu hình một kết nối chỉ đọc (read-only) để đảm bảo rằng tôi không vô tình cập nhật cơ sở dữ liệu.
- `tools` và `allowedTools` – Một lần nữa, tôi đã cho phép Amazon Q Developer sử dụng tất cả các công cụ. Tuy nhiên, hãy lưu ý rằng tôi hạn chế hơn về những công cụ được tin cậy. Amazon Q Developer sẽ cần hỏi xin phép để sử dụng `fs_write` (ghi tệp) hoặc @`PostgreSQL/run_query` (chạy truy vấn SQL). Lưu ý rằng tôi có thể cho phép toàn bộ máy chủ MCP (như tôi đã làm với Figma) hoặc các công cụ cụ thể (như tôi đã làm ở đây).
- `resources` – Tương tự, tôi đã bao gồm tệp README.md (được lưu trữ trong thư mục dự án) và các tùy chọn riêng của tôi cho Python và SQL (cả hai đều được lưu trữ trong hồ sơ cá nhân). Lưu ý rằng tôi cũng có thể sử dụng các mẫu glob (glob patterns) ở đây. Ví dụ, `file://.amazonq/rules/**/*.md` sẽ bao gồm các quy tắc được tạo bởi các plugin IDE của Amazon Q Developer.
- `hooks` – Cuối cùng, tôi cũng đã đưa vào hook cho cả front-end và back-end. Tuy nhiên, tôi có thể đã đưa vào các tùy chọn dành riêng cho dự án như `npm run` cho front-end và `pip freeze` cho back-end.
```bash
{
  "name": "back-end",
  "description": "Optimized for back-end development with Python and PostgreSQL",
  "mcpServers": {
    "PostgreSQL": {
      "command": "uvx",
      "args": [
        "awslabs.postgres-mcp-server@latest",
        "--resource_arn", "arn:aws:rds:us-east-1:xxxxxxxxxxxx:cluster:xxxxxx",
        "--secret_arn", "arn:aws:secretsmanager:us-east-1:xxxxxxxxxxxx:secret:rds!cluster-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx-xxxxxx",
        "--database", "dev",
        "--region", "us-east-1",
        "--readonly", "True"
      ]
    }
  },
  "tools": ["*"],
  "allowedTools": [
    "fs_read",
    "report_issues",
    "@PostgreSQL/get_table_schema"
  ],
  "resources": [
    "file://README.md",
    "file://~/.aws/amazonq/python-preferences.md",
    "file://~/.aws/amazonq/sql-preferences.md"
  ],
  "hooks": {
    "agentSpawn": [
      {
        "command": "git status"
      }
    ]
  }
}
```
## Sử Dụng Các Tác Nhân Tùy Chỉnh

Sức mạnh thực sự của các tác nhân (agents) trở nên rõ ràng khi tôi cần chuyển đổi giữa các ngữ cảnh phát triển khác nhau này. Giờ đây, tôi chỉ cần chạy lệnh `q chat --agent front-end` khi tôi làm việc với React và Figma, hoặc `q chat --agent back-end` khi tôi làm việc với Python và SQL. Amazon Q Developer sẽ tự động cấu hình tác nhân chính xác với tất cả các tùy chọn cá nhân của tôi.

Trong hình ảnh sau đây, bạn có thể thấy rõ sự khác biệt trong cấu hình của Amazon Q Developer CLI. Hãy chú ý rằng tác nhân Front-end có thêm công cụ Figma, trong khi Tác nhân Back-end có thêm công cụ PostgreSQL. Ngoài ra tác nhân Front-end tin cậy (trusts) `fs_write` và tất cả các công cụ Figma trong khi tác nhân Back-end sẽ hỏi xin phép để sử dụng `fs_write` và chỉ tin cậy một trong hai công cụ PostgreSQL.

![](/images/3-BlogsTranslated/3.1-blog3/custom-agents-image01.png)

Tương tự, hãy xem cấu hình ngữ cảnh ở cả tác tử front-end và back-end. Trong hình dưới đây, tôi đã thêm các tùy chọn yêu thích của mình cho React trong việc phát triển front-end, và các tùy chọn về Python và SQL cho việc phát triển back-end.

![](/images/3-BlogsTranslated/3.1-blog3/custom-agents-image02.png)

Như bạn có thể thấy, các tác tử tùy chỉnh cho phép tôi tối ưu hóa Amazon Q Developer CLI cho từng nhiệm vụ. Tất nhiên, tác tử front-end và back-end chỉ là một ví dụ. Bạn có thể có tác tử dành cho lập trình và kiểm thử, tác tử cho khoa học dữ liệu và phân tích, v.v. Các tác tử tùy chỉnh cho phép bạn điều chỉnh cấu hình phù hợp với hầu hết mọi nhiệm vụ.

## Kết luận

Các tác tử tùy chỉnh của Amazon Q Developer CLI đại diện cho một bước tiến đáng kể trong việc quản lý các môi trường phát triển phức tạp. Bằng cách cho phép các lập trình viên chuyển đổi liền mạch giữa các ngữ cảnh khác nhau, chúng loại bỏ gánh nặng tư duy khi phải tự tay cấu hình lại công cụ và quyền truy cập cho từng nhiệm vụ. Bạn đã sẵn sàng tối ưu hóa quy trình phát triển của mình chưa? [Hãy bắt đầu với Amazon Q Developer](https://aws.amazon.com/q/developer/getting-started/) ngay hôm nay.

