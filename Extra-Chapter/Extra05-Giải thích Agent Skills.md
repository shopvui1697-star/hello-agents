# Agent Skills và MCP: Hai mô hình mở rộng năng lực cho Agent

## Lời mở đầu: Sau MCP, chúng ta còn cần gì nữa?

Ở chương mười, chúng ta đã đi sâu tìm hiểu cách MCP (Model Context Protocol) giải quyết bài toán kết nối giữa Agent (tác nhân thông minh) với các công cụ bên ngoài thông qua một giao thức chuẩn hóa. Bạn đã học được cách để Agent truy cập cơ sở dữ liệu, hệ thống tệp, dịch vụ API và nhiều loại tài nguyên khác nhau thông qua MCP. Hãy cùng ôn lại một kịch bản sử dụng MCP điển hình:

```python
from hello_agents import ReActAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool

llm = HelloAgentsLLM()
agent = ReActAgent(name="数据分析助手", llm=llm)

# Kết nối tới máy chủ MCP của cơ sở dữ liệu
db_mcp = MCPTool(server_command=["python", "database_mcp_server.py"])
agent.add_tool(db_mcp)

# Bây giờ Agent đã có thể truy cập cơ sở dữ liệu
response = agent.run("查询员工表中薪资最高的前10名员工")
```

Đoạn mã này hoạt động rất tốt, Agent đã kết nối thành công tới cơ sở dữ liệu. Nhưng khi bạn thử xử lý các tác vụ phức tạp hơn, bạn sẽ phát hiện ra một số vấn đề tinh vi:

```python
# Một yêu cầu phức tạp hơn
response = agent.run("""
分析公司内部谁的话语权最高？
需要综合考虑：
1. 管理层级和下属数量
2. 薪资水平和涨薪幅度
3. 任职时长和稳定性
4. 跨部门影响力
""")
```

Tác vụ này cần thực hiện nhiều lần truy vấn cơ sở dữ liệu, và kết quả của mỗi lần truy vấn sẽ ảnh hưởng đến chiến lược cho lần truy vấn tiếp theo. Điều quan trọng hơn là nó đòi hỏi Agent phải có <strong>kiến thức lĩnh vực (domain knowledge)</strong>: biết cách đánh giá "tiếng nói/quyền phát ngôn", biết nên phân tích dữ liệu từ những chiều nào, biết cách kết hợp kết quả của nhiều truy vấn để rút ra kết luận.



Tại thời điểm này, bạn sẽ gặp phải hai vấn đề mang tính căn bản:

<strong>Vấn đề thứ nhất là bùng nổ ngữ cảnh (context explosion)</strong>. Để Agent có thể truy vấn cơ sở dữ liệu một cách linh hoạt, máy chủ MCP thường sẽ phơi bày hàng chục thậm chí hàng trăm công cụ (các bảng khác nhau, các phương thức truy vấn khác nhau). Toàn bộ JSON Schema đầy đủ của các công cụ này sẽ được nạp vào prompt hệ thống (system prompt) ngay khi kết nối được thiết lập, có thể chiếm tới hàng chục nghìn token. Theo phản hồi từ các nhà phát triển trong cộng đồng, chỉ riêng việc nạp một máy chủ Playwright MCP đã chiếm 8% của cửa sổ ngữ cảnh 200k, và con số này sẽ tích lũy nhanh chóng trong các cuộc hội thoại nhiều lượt, dẫn đến chi phí tăng vọt và năng lực suy luận giảm sút.

<strong>Vấn đề thứ hai là hố ngăn cách về năng lực (capability gap)</strong>. MCP đã giải quyết được bài toán "có thể kết nối", nhưng chưa giải quyết được bài toán "biết cách sử dụng". Có năng lực kết nối cơ sở dữ liệu không đồng nghĩa với việc Agent biết cách viết ra câu SQL hiệu quả và an toàn; có thể truy cập hệ thống tệp không có nghĩa là nó hiểu được cấu trúc mã nguồn và quy chuẩn phát triển của một dự án cụ thể. Điều này giống như việc cấp cho một lập trình viên mới quyền truy cập vào mọi hệ thống, nhưng lại không cung cấp cẩm nang vận hành và các thực hành tốt nhất (best practices).

Đây chính là vấn đề cốt lõi mà <strong>Agent Skills</strong> muốn giải quyết. Đầu năm 2025, sau khi ra mắt MCP, Anthropic đã tiến thêm một bước và đề xuất khái niệm Agent Skills, gây được sự chú ý rộng rãi trong ngành. Có nhà phát triển bình luận rằng: "Skills và MCP là hai thứ khác nhau. Skills là kiến thức lĩnh vực, nói cho mô hình biết nên làm như thế nào, về bản chất là một dạng Prompt cấp cao; còn MCP thì kết nối tới công cụ và dữ liệu bên ngoài." Cũng có người cho rằng: "Từ Function Call đến Tool Call đến MCP rồi đến Skills, cốt lõi cũng không khác nhau nhiều, chỉ là sự tiến hóa tối ưu hóa về mặt thực hành kỹ thuật và hình thức thể hiện."

Vậy thì, Agent Skills rốt cuộc là gì? Nó khác biệt về bản chất với MCP như thế nào? Hai công nghệ này là quan hệ cạnh tranh hay bổ trợ cho nhau? Chương này sẽ đi sâu thảo luận những vấn đề đó.



## Agent Skills là gì?

### Triết lý thiết kế cốt lõi

<strong>Agent Skills là một định dạng chuẩn hóa để đóng gói kiến thức mang tính quy trình (procedural knowledge)</strong>. Nếu như MCP cung cấp cho Agent "đôi tay" để thao tác công cụ, thì Skills cung cấp "cẩm nang vận hành" hay "SOP (Quy trình tác nghiệp chuẩn)", dạy cho Agent cách sử dụng những công cụ đó một cách đúng đắn.

Triết lý thiết kế này xuất phát từ một sự thấu hiểu đơn giản nhưng sâu sắc: <strong>Tính kết nối (Connectivity) và Năng lực (Capability) nên được tách biệt</strong>. MCP tập trung vào cái trước, Skills tập trung vào cái sau. Sự phân tách trách nhiệm này mang lại những lợi thế rõ ràng về mặt kiến trúc:

- <strong>Trách nhiệm của MCP</strong>: Cung cấp giao diện truy cập chuẩn hóa, giúp Agent có thể "với tới" dữ liệu và công cụ của thế giới bên ngoài.
- <strong>Trách nhiệm của Skills</strong>: Cung cấp kiến thức chuyên môn về lĩnh vực, nói cho Agent biết trong một tình huống cụ thể "phải kết hợp sử dụng các công cụ này như thế nào".

Hãy dùng một phép so sánh để hiểu: MCP giống như cổng USB hoặc trình điều khiển (driver), nó định nghĩa cách thiết bị kết nối; còn Skills giống như một ứng dụng phần mềm, nó định nghĩa cách sử dụng những thiết bị đã kết nối đó để hoàn thành các tác vụ cụ thể. Bạn có thể sở hữu một trình điều khiển máy in đầy đủ tính năng (MCP), nhưng nếu không có ai chỉ cho bạn cách thiết lập lề trang và in hai mặt trong Word (Skill), bạn vẫn không thể hoàn thành công việc in ấn một cách hiệu quả.

### Tiết lộ tiệm tiến: Phá giải thế bí về ngữ cảnh

Đổi mới cốt lõi nhất của Agent Skills là <strong>cơ chế tiết lộ tiệm tiến (Progressive Disclosure)</strong>. Cơ chế này phân chia thông tin kỹ năng thành ba tầng, Agent nạp dần từng bước theo nhu cầu, vừa đảm bảo không bỏ sót chi tiết khi cần thiết, vừa tránh việc nhồi nhét quá nhiều nội dung vào cửa sổ ngữ cảnh cùng một lúc.

<div align="center">
  <img src="./images/Extra05-figures/image1.png" alt="渐进式披露三层架构" width="90%"/>
  <p>Hình 1 Kiến trúc ba tầng của cơ chế tiết lộ tiệm tiến trong Agent Skills</p>
</div>


#### Tầng thứ nhất: Siêu dữ liệu (Metadata)

Trong thiết kế của Skills, mỗi kỹ năng được lưu trong một thư mục độc lập, cốt lõi là một tệp Markdown có tên `SKILL.md`. Tệp này bắt buộc phải bắt đầu bằng phần Frontmatter theo định dạng YAML, dùng để định nghĩa thông tin cơ bản của kỹ năng.

Khi Agent khởi động, nó sẽ quét toàn bộ các thư mục kỹ năng đã được cài đặt, <strong>chỉ đọc phần Frontmatter của mỗi tệp `SKILL.md`</strong>, và nạp những siêu dữ liệu này vào prompt hệ thống. Theo dữ liệu đo thực tế, siêu dữ liệu của mỗi kỹ năng chỉ tiêu tốn khoảng <strong>100 token</strong>. Ngay cả khi bạn cài đặt 50 kỹ năng, mức tiêu thụ ngữ cảnh ban đầu cũng chỉ khoảng 5.000 token.

Điều này tạo nên sự tương phản rõ rệt với cách thức hoạt động của MCP. Trong một triển khai MCP điển hình, khi máy khách (client) kết nối tới một máy chủ, nó thường sẽ lấy toàn bộ JSON Schema đầy đủ của tất cả các công cụ khả dụng thông qua yêu cầu `tools/list`, và có thể tiêu tốn ngay hàng chục nghìn token.

#### Tầng thứ hai: Phần thân kỹ năng (Instructions)

Khi Agent, thông qua việc phân tích yêu cầu của người dùng, xác định rằng một kỹ năng nào đó có liên quan cao đến tác vụ hiện tại, nó sẽ chuyển sang nạp tầng thứ hai. Lúc này, Agent sẽ đọc toàn bộ nội dung tệp `SKILL.md` của kỹ năng đó, nạp vào ngữ cảnh các chỉ dẫn chi tiết, các lưu ý, ví dụ, v.v.

Tại thời điểm này, Agent đã có được toàn bộ ngữ cảnh cần thiết để hoàn thành tác vụ: cấu trúc cơ sở dữ liệu, mẫu truy vấn, các lưu ý, v.v. Mức tiêu thụ token của phần nội dung này phụ thuộc vào độ phức tạp của các chỉ dẫn, thường nằm trong khoảng từ 1.000 đến 5.000 token.

#### Tầng thứ ba: Tài nguyên bổ sung (Scripts & References)

Đối với các kỹ năng phức tạp hơn, `SKILL.md` có thể tham chiếu tới các tệp khác nằm trong cùng thư mục: các tập lệnh (script), tệp cấu hình, tài liệu tham khảo, v.v. Agent <strong>chỉ nạp những tài nguyên này khi cần thiết</strong>.

Ví dụ, cấu trúc tệp của một kỹ năng xử lý PDF có thể như sau:

```
skills/pdf-processing/
├── SKILL.md              # 主技能文件
├── parse_pdf.py          # PDF 解析脚本
├── forms.md              # 表单填写指南（仅在填表任务时加载）
└── templates/            # PDF 模板文件
    ├── invoice.pdf
    └── report.pdf
```

Trong `SKILL.md`, bạn có thể tham chiếu tài nguyên bổ sung như sau:

- Khi cần thực hiện phân tích cú pháp PDF, Agent sẽ chạy tập lệnh `parse_pdf.py`
- Khi gặp tác vụ điền biểu mẫu, mới nạp tệp `forms.md` để tìm hiểu các bước chi tiết
- Các tệp mẫu (template) chỉ được truy cập khi cần tạo ra tài liệu theo định dạng cụ thể

Thiết kế này có hai lợi thế then chốt:

1. <strong>Dung lượng kiến thức không giới hạn</strong>: Thông qua các tập lệnh và tệp bên ngoài, kỹ năng có thể "mang theo" lượng kiến thức vượt xa giới hạn ngữ cảnh. Ví dụ, một kỹ năng phân tích dữ liệu có thể đính kèm một tệp dữ liệu 1GB và một tập lệnh truy vấn; Agent truy cập dữ liệu bằng cách thực thi tập lệnh, mà không cần nạp toàn bộ tập dữ liệu vào ngữ cảnh.

2. <strong>Thực thi mang tính tất định (deterministic)</strong>: Các tác vụ như tính toán phức tạp, chuyển đổi dữ liệu, phân tích định dạng, v.v. được giao cho mã nguồn thực thi, tránh được vấn đề bất định và ảo giác (hallucination) trong quá trình LLM sinh nội dung.

### Hiệu quả của tiết lộ tiệm tiến: Từ 16k xuống còn 500 Token

Các trường hợp thực tế do các nhà phát triển trong cộng đồng chia sẻ đã chứng minh đầy đủ sức mạnh của tiết lộ tiệm tiến. Trong một kịch bản thực tế:

- <strong>Cách làm MCP truyền thống</strong>: Kết nối trực tiếp tới một máy chủ MCP chứa lượng lớn định nghĩa công cụ, việc nạp ban đầu tiêu tốn <strong>16.000 token</strong>
- <strong>Sau khi đóng gói bằng Skills</strong>: Tạo một Skill đơn giản làm "cổng vào" (gateway), chỉ mô tả chức năng trong phần Frontmatter, mức tiêu thụ ban đầu chỉ còn <strong>500 token</strong>

Khi Agent xác định rằng cần sử dụng kỹ năng đó, nó mới nạp các chỉ dẫn chi tiết và gọi tới các công cụ MCP bên dưới theo nhu cầu. Kiến trúc này không chỉ giảm mạnh chi phí ban đầu, mà còn giúp việc quản lý ngữ cảnh trong suốt cuộc hội thoại trở nên chính xác và hiệu quả hơn.

## Agent Skills vs MCP: Khác biệt bản chất và quan hệ hợp tác

Bây giờ, chúng ta có thể so sánh một cách hệ thống sự khác biệt về bản chất giữa hai công nghệ này.

<div align="center">
  <img src="./images/Extra05-figures/image2.png" alt="MCP与Agent Skills设计哲学对比" width="90%"/>
  <p>Hình 2 So sánh triết lý thiết kế giữa MCP và Agent Skills</p>
</div>

### Hiểu sự khác biệt từ góc nhìn kỹ thuật

Hãy cùng hiểu sự khác biệt này thông qua một ví dụ cụ thể. Giả sử bạn muốn xây dựng một Agent để giúp đội ngũ thực hiện việc rà soát mã (code review):

<strong>Trách nhiệm của MCP</strong>:

```python
# MCP cung cấp quyền truy cập chuẩn hóa tới GitHub
github_mcp = MCPTool(server_command=["npx", "-y", "@modelcontextprotocol/server-github"])

# Các công cụ mà MCP phơi bày (ví dụ rút gọn):
# - list_pull_requests(repo, state)
# - get_pull_request_details(pr_number)
# - list_pr_comments(pr_number)
# - create_pr_comment(pr_number, body)
# - get_file_content(repo, path, ref)
# - list_pr_files(pr_number)
```

MCP giúp Agent "có thể" truy cập GitHub, có thể gọi các API này. Nhưng nó không biết "nên" làm gì.

<strong>Trách nhiệm của Skills</strong>:

```markdown
---
name: code-review-workflow
description: 执行标准的代码审查流程，包括检查代码风格、安全问题、测试覆盖率等
---

# 代码审查工作流

## 审查清单

当执行代码审查时，按以下步骤进行：

1. **获取 PR 信息**：调用 `get_pull_request_details` 了解变更背景
2. **分析变更文件**：调用 `list_pr_files` 获取文件列表
3. **逐文件审查**：
   - 对于 `.py` 文件：检查是否符合 PEP 8，是否有明显的性能问题
   - 对于 `.js/.ts` 文件：检查是否有未处理的 Promise，是否使用了废弃的 API
   - 对于测试文件：验证是否覆盖了新增的代码路径
4. **安全检查**：
   - 是否硬编码了敏感信息（密钥、密码）
   - 是否有 SQL 注入或 XSS 风险
5. **提供反馈**：
   - 严重问题：使用 `create_pr_comment` 直接评论
   - 建议改进：在总结中提出

## 公司特定规范

- 所有数据库查询必须使用参数化查询
- API 端点必须有权限验证装饰器
- 新功能必须附带单元测试（覆盖率 > 80%）

## 示例评论模板

**严重问题**：

⚠️ 安全风险：第 45 行直接拼接 SQL 字符串，存在注入风险。
建议改用参数化查询：`cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))`

```

Skills nói cho Agent biết "nên" làm gì, tổ chức quy trình rà soát như thế nào, cần chú ý tới những quy chuẩn đặc thù nào của công ty. Nó là nơi chứa đựng kiến thức lĩnh vực và các thực hành tốt nhất.



### Sự khác biệt bản chất trong chiến lược quản lý ngữ cảnh

<div align="center">
  <img src="./images/Extra05-figures/image3.png" alt="MCP急切加载 vs Skills惰性加载" width="90%"/>
  <p>Hình 3 So sánh nạp háo hức (eager) của MCP với nạp lười (lazy) của Skills</p>
</div>


### Bổ trợ chứ không cạnh tranh: Kiến trúc lai Skills + MCP

Sau khi hiểu được sự khác biệt giữa hai công nghệ, chúng ta sẽ nhận ra: <strong>Skills và MCP không phải là quan hệ cạnh tranh, mà là quan hệ bổ trợ cho nhau</strong>. Thực hành tốt nhất là kết hợp cả hai, tạo thành một kiến trúc phân tầng:

<div align="center">
  <img src="./images/Extra05-figures/image4.png" alt="Skills + MCP 混合架构" width="90%"/>
  <p>Hình 4 Thiết kế kiến trúc lai Skills + MCP</p>
</div>
<strong>Quy trình làm việc điển hình</strong>:

1. Người dùng hỏi: "Phân tích trong nội bộ công ty ai có tiếng nói cao nhất"
2. <strong>Tầng Skills</strong> nhận diện đây là một tác vụ phân tích dữ liệu, nạp kỹ năng `mysql-employees-analysis`
3. <strong>Tầng Skills</strong> dựa theo chỉ dẫn của kỹ năng, phân rã tác vụ thành các bước con: truy vấn quan hệ quản lý, so sánh lương, thời gian công tác, v.v.
4. <strong>Tầng MCP</strong> thực thi các truy vấn SQL cụ thể, trả về kết quả
5. <strong>Tầng Skills</strong> dựa theo kiến thức lĩnh vực trong kỹ năng, diễn giải dữ liệu và tạo ra phân tích tổng hợp
6. Trả về câu trả lời có cấu trúc cho người dùng

Ưu điểm của kiến trúc này là:

- <strong>Phân tách mối quan tâm (separation of concerns)</strong>: MCP tập trung vào "năng lực", Skills tập trung vào "trí tuệ"
- <strong>Tối ưu chi phí</strong>: Nạp tiệm tiến giảm mạnh mức tiêu thụ token
- <strong>Khả năng bảo trì</strong>: Logic nghiệp vụ (Skills) tách rời khỏi hạ tầng (MCP)
- <strong>Khả năng tái sử dụng</strong>: Cùng một máy chủ MCP có thể được nhiều Skills sử dụng

## Triển khai kỹ thuật: Cách tạo và sử dụng Skills

### Giải thích chi tiết quy chuẩn SKILL.md

Hãy cùng tìm hiểu sâu về cấu trúc chuẩn của tệp `SKILL.md`:

```markdown
---
# === 必需字段 ===
name: skill-name
  # 技能的唯一标识符，使用 kebab-case 命名

description: >
  简洁但精确的描述，说明：
  1. 这个技能做什么
  2. 什么时候应该使用它
  3. 它的核心价值是什么
  # 注意：description 是智能体选择技能的唯一依据，必须写清楚！

# === 可选字段 ===
version: 1.0.0
  # 语义化版本号

allowed_tools: [tool1, tool2]
  # 此技能可以调用的工具列表（白名单）

required_context: [context_item1]
  # 此技能需要的上下文信息

license: MIT
  # 许可协议

author: Your Name <email@example.com>
  # 作者信息

tags: [database, analysis, sql]
  # 便于分类和搜索的标签
---

# 技能标题

## 概述
（对技能的详细介绍，包括使用场景、技术背景等）

## 前置条件
（使用此技能需要的环境配置、依赖项等）

## 工作流程
（详细的步骤说明，告诉智能体如何执行任务）

## 最佳实践
（经验总结、注意事项、常见陷阱等）

## 示例
（具体的使用案例，帮助智能体理解）

## 故障排查
（常见问题和解决方案）
```

### Các nguyên tắc để viết Skills chất lượng cao

Dựa theo tài liệu chính thức của Anthropic và các thực hành tốt nhất trong cộng đồng, để viết được những Skills hiệu quả cần tuân theo các nguyên tắc sau:

#### 1. Description chính xác

`description` là yếu tố then chốt cho việc ra quyết định của Agent. Nó cần:

- <strong>Định nghĩa chính xác phạm vi áp dụng</strong>: Tránh những mô tả mơ hồ như "giúp xử lý dữ liệu"
- <strong>Chứa các từ khóa kích hoạt</strong>: Để Agent có thể khớp với ý định của người dùng
- <strong>Nêu rõ giá trị độc đáo</strong>: Phân biệt với các kỹ năng khác

❌ <strong>description không tốt</strong>:
```yaml
description: 处理数据库查询
```

✅ <strong>description tốt</strong>:
```yaml
description: >
  将中文业务问题转换为 SQL 查询并分析 MySQL employees 示例数据库。
  适用于员工信息查询、薪资统计、部门分析、职位变动历史等场景。
  当用户询问关于员工、薪资、部门的数据时使用此技能。
```

#### 2. Mô-đun hóa và trách nhiệm đơn nhất

Một Skill nên tập trung vào một lĩnh vực hoặc một loại tác vụ rõ ràng. Nếu một Skill cố gắng làm quá nhiều việc, sẽ dẫn đến:

- Description quá rộng, độ chính xác khi khớp giảm sút
- Nội dung chỉ dẫn quá dài, lãng phí ngữ cảnh
- Khó bảo trì và cập nhật

<strong>Khuyến nghị</strong>: Thay vì tạo ra một kỹ năng "phân tích dữ liệu tổng quát", tốt hơn là tạo nhiều kỹ năng chuyên biệt:
- `mysql-employees-analysis`: Chuyên phân tích cơ sở dữ liệu employees
- `sales-data-analysis`: Chuyên phân tích dữ liệu bán hàng
- `user-behavior-analysis`: Chuyên phân tích dữ liệu hành vi người dùng

#### 3. Nguyên tắc ưu tiên tính tất định

Đối với các tác vụ phức tạp, cần thực thi chính xác, hãy ưu tiên sử dụng tập lệnh thay vì phụ thuộc vào việc LLM sinh nội dung. Ví dụ, trong kịch bản xuất dữ liệu, thay vì để LLM sinh ra nội dung nhị phân của tệp Excel (dễ sai sót), tốt hơn là viết một tập lệnh chuyên dụng để xử lý tác vụ này; trong SKILL.md chỉ cần hướng dẫn Agent khi nào thì gọi tập lệnh này là được.

#### 4. Chiến lược tiết lộ tiệm tiến

Tận dụng hợp lý cấu trúc ba tầng, phân tầng thông tin theo mức độ quan trọng và tần suất sử dụng:

- <strong>Phần thân SKILL.md</strong>: Đặt quy trình làm việc cốt lõi, các mẫu thường dùng
- <strong>Tài liệu bổ sung</strong> (như `advanced.md`): Đặt các cách dùng nâng cao, các trường hợp biên (edge cases)
- <strong>Tệp dữ liệu</strong>: Đặt dữ liệu tham khảo cỡ lớn, truy vấn theo nhu cầu thông qua tập lệnh

### Trường hợp thực hành: Giải thích chi tiết Skill phân tích nhân viên MySQL

Hãy cùng tìm hiểu ứng dụng cụ thể của Agent Skills thông qua một trường hợp thực tế từ cộng đồng Anthropic. Kỹ năng này được dùng để phân tích cơ sở dữ liệu mẫu `employees` chính thức của MySQL.

#### Cấu trúc tệp của kỹ năng

```
skills/mysql-employees-analysis/
├── SKILL.md          # 主技能文件（包含元数据和详细指令）
└── db_schema.sql     # 数据库结构参考（可选，按需加载）
```

#### Ví dụ nội dung cốt lõi của SKILL.md

Phần Frontmatter (tầng siêu dữ liệu) của kỹ năng này:

```markdown
---
name: mysql-employees-analysis
description: >
  将中文业务问题转换为 SQL 查询并分析 MySQL employees 示例数据库。
  适用于员工信息查询（如"工号12345的员工信息"）、
  薪资统计（如"平均薪资最高的部门"）、
  部门分析（如"各部门人数分布"）、
  职位变动历史（如"某员工的晋升路径"）等场景。
version: 1.0.0
allowed_tools: [execute_sql]
tags: [database, mysql, sql, employees, analysis]
---

# MySQL 员工数据库分析技能

## 概述

这个技能专门用于分析 MySQL 官方提供的 `employees` 示例数据库。
该数据库包含约 300,000 名虚拟员工的记录，涵盖 1985-2000 年的数据。

**核心能力**：
- 理解中文自然语言的业务问题
- 转换为高效的 SQL 查询
- 执行查询并解读结果
- 提供业务洞察和数据解读

## 数据库结构

### 核心表结构

| 表名           | 说明         | 关键字段                                                     |
| -------------- | ------------ | ------------------------------------------------------------ |
| `employees`    | 员工基本信息 | emp_no, birth_date, first_name, last_name, gender, hire_date |
| `salaries`     | 薪资历史     | emp_no, salary, from_date, to_date                           |
| `titles`       | 职位历史     | emp_no, title, from_date, to_date                            |
| `dept_emp`     | 员工部门关系 | emp_no, dept_no, from_date, to_date                          |
| `dept_manager` | 部门经理     | emp_no, dept_no, from_date, to_date                          |
| `departments`  | 部门信息     | dept_no, dept_name                                           |

### 关键约定

⚠️ **重要**：`to_date = '9999-01-01'` 表示"当前有效"的记录。
查询"当前"状态时（如现任员工、当前薪资），必须加此过滤条件。

完整的表结构参见：`db_schema.sql`

## 工作流程

### 第一步：理解需求

仔细分析用户的中文描述，识别：
- **查询目标**：要查什么数据？（员工、薪资、部门...）
- **筛选条件**：有什么限制？（特定部门、时间范围、薪资区间...）
- **聚合维度**：需要统计吗？（平均值、总数、排名...）
- **时间范围**：是历史数据还是当前状态？

### 第二步：构建 SQL

根据需求选择合适的查询模式（见下方"常见查询模式"）。

**编写原则**：
1. 使用明确的表别名（如 `e` for employees）
2. JOIN 时优先使用主键/外键
3. 注意日期过滤（特别是 `to_date`）
4. 合理使用索引字段
5. 大结果集要加 LIMIT

### 第三步：执行查询

调用 `execute_sql` 工具执行构建好的 SQL。

```python
# 示例调用（智能体会自动转换为工具调用）
result = execute_sql(query="SELECT ...")


### 第四步：解读结果

将查询结果转化为自然语言回答：
- 用表格呈现结构化数据
- 突出关键数据点
- 提供业务洞察（如趋势、异常）
- 如果结果为空，说明可能的原因

## 常见查询模式

### 模式 1：基础信息查询


-- 查询特定员工的基本信息
SELECT emp_no, CONCAT(first_name, ' ', last_name) AS full_name,
       gender, birth_date, hire_date
FROM employees
WHERE emp_no = <员工号>;


### 模式 2：当前状态查询


-- 查询当前薪资最高的员工（TOP 10）
SELECT e.emp_no,
       CONCAT(e.first_name, ' ', e.last_name) AS name,
       s.salary
FROM employees e
JOIN salaries s ON e.emp_no = s.emp_no
WHERE s.to_date = '9999-01-01'  -- 当前薪资
ORDER BY s.salary DESC
LIMIT 10;


### 模式 3：历史趋势分析


-- 查询某员工的薪资变化历史
SELECT emp_no, salary, from_date, to_date,
       salary - LAG(salary) OVER (ORDER BY from_date) AS increase
FROM salaries
WHERE emp_no = <员工号>
ORDER BY from_date;


### 模式 4：跨表关联查询


-- 查询各部门的平均薪资（当前）
SELECT d.dept_name,
       COUNT(DISTINCT de.emp_no) AS emp_count,
       ROUND(AVG(s.salary), 2) AS avg_salary
FROM departments d
JOIN dept_emp de ON d.dept_no = de.dept_no
JOIN salaries s ON de.emp_no = s.emp_no
WHERE de.to_date = '9999-01-01'  -- 当前在职
  AND s.to_date = '9999-01-01'   -- 当前薪资
GROUP BY d.dept_name
ORDER BY avg_salary DESC;


### 模式 5：复杂业务分析


-- 分析"话语权"：综合管理层级、薪资、任职时长
WITH manager_hierarchy AS (
    -- 统计每个经理管理的下属数
    SELECT dm.emp_no, COUNT(de.emp_no) AS subordinate_count
    FROM dept_manager dm
    JOIN dept_emp de ON dm.dept_no = de.dept_no
    WHERE dm.to_date = '9999-01-01'
      AND de.to_date = '9999-01-01'
      AND de.emp_no != dm.emp_no
    GROUP BY dm.emp_no
),
current_salary AS (
    -- 当前薪资
    SELECT emp_no, salary
    FROM salaries
    WHERE to_date = '9999-01-01'
),
tenure AS (
    -- 任职时长（年）
    SELECT emp_no,
           TIMESTAMPDIFF(YEAR, hire_date, CURDATE()) AS years_employed
    FROM employees
)
SELECT e.emp_no,
       CONCAT(e.first_name, ' ', e.last_name) AS name,
       COALESCE(mh.subordinate_count, 0) AS team_size,
       cs.salary,
       t.years_employed,
       -- 简单的话语权评分（可根据业务调整权重）
       (COALESCE(mh.subordinate_count, 0) * 10 +
        cs.salary / 1000 +
        t.years_employed * 5) AS influence_score
FROM employees e
JOIN current_salary cs ON e.emp_no = cs.emp_no
JOIN tenure t ON e.emp_no = t.emp_no
LEFT JOIN manager_hierarchy mh ON e.emp_no = mh.emp_no
WHERE cs.salary > 60000  -- 过滤低薪员工
ORDER BY influence_score DESC
LIMIT 20;


## 注意事项

### ⚠️ 时间字段的正确处理

- <strong>当前状态</strong>：必须使用 `to_date = '9999-01-01'` 过滤
- <strong>历史查询</strong>：注意 `from_date` 和 `to_date` 的范围
- <strong>时间计算</strong>：使用 `TIMESTAMPDIFF`、`DATEDIFF` 等函数

### ⚠️ 性能优化

- <strong>大表 JOIN</strong>：优先使用索引字段（emp_no, dept_no）
- <strong>聚合查询</strong>：合理使用 GROUP BY 和 HAVING
- <strong>结果限制</strong>：对于展示类查询，添加 LIMIT 限制
- <strong>子查询优化</strong>：复杂查询使用 WITH (CTE) 提高可读性和性能

### ⚠️ 数据质量

- <strong>NULL 值处理</strong>：使用 COALESCE 或 IFNULL 处理空值
- <strong>重复记录</strong>：注意员工可能多次调岗，查询时考虑去重
- <strong>数据范围</strong>：数据库只包含 1985-2000 年的数据，查询时注意时间边界

## 故障排查

<strong>问题 1：查询结果为空</strong>
- 检查是否正确使用了 `to_date = '9999-01-01'`
- 验证员工号或部门号是否存在
- 检查日期范围是否合理

<strong>问题 2：查询速度慢</strong>
- 检查是否缺少索引字段的 WHERE 条件
- 考虑将复杂查询拆分为多步
- 使用 EXPLAIN 分析查询计划

<strong>问题 3：统计数据不准确</strong>
- 注意区分"历史"和"当前"状态
- 检查 JOIN 条件是否遗漏
- 验证聚合函数的使用是否正确
```

Tệp SKILL.md này minh họa cấu trúc của một kỹ năng hoàn chỉnh:
- Siêu dữ liệu rõ ràng (Agent dùng để phát hiện và khớp)
- Mô tả đầy đủ về cấu trúc cơ sở dữ liệu
- Hướng dẫn quy trình làm việc chi tiết
- Nhiều ví dụ mẫu truy vấn phong phú (các mẫu SQL có thể tái sử dụng trực tiếp)
- Các lưu ý và xử lý sự cố thiết thực

#### Hiệu quả sử dụng của kỹ năng

Khi người dùng đặt câu hỏi cho một Agent hỗ trợ Agent Skills (như Claude Desktop, Claude Code):

**Câu hỏi của người dùng**:
> "Phân tích trong nội bộ công ty ai có tiếng nói cao nhất? Cần cân nhắc tổng hợp cấp bậc quản lý, mức lương và thời gian công tác."

<div align="center">
  <img src="./images/Extra05-figures/image5.png" alt="Agent Skills工作流程" width="90%"/>
  <p>Hình 5 Sơ đồ minh họa quy trình làm việc hoàn chỉnh của Agent Skills</p>
</div>


**Ví dụ đầu ra**:

| Xếp hạng | Mã NV | Họ tên               | Quy mô đội | Lương    | Số năm công tác | Điểm ảnh hưởng |
| -------- | ----- | -------------------- | ---------- | -------- | --------------- | -------------- |
| 1        | 110022 | Margareta Markovitch | 45         | 152,710  | 18              | 692.71         |
| 2        | 110039 | Vishwani Minakawa    | 38         | 138,273  | 16              | 598.27         |
| 3        | 110085 | Ebru Alpin           | 32         | 124,054  | 15              | 519.05         |

**Những thấu hiểu then chốt**:
1. Nhân viên có tiếng nói cao nhất thường quản lý đội ngũ lớn (30+ người), lương thuộc top 1% (>120 nghìn), thời gian công tác trên 15 năm
2. Ảnh hưởng của các quản lý phòng ban vượt xa nhân viên thông thường, quy mô quản lý là yếu tố then chốt
3. Những nhân viên lương cao gắn bó lâu dài, dù không giữ chức vụ quản lý, cũng có tiếng nói khá mạnh

Trong suốt quá trình, kỹ năng đã cung cấp:
- **Kiến thức lĩnh vực**: Cách đánh giá "tiếng nói" (quy mô quản lý + lương + thời gian công tác)
- **Hướng dẫn kỹ thuật**: Cách viết SQL hiệu quả (dùng CTE, hàm cửa sổ, JOIN nhiều bảng)
- **Hiểu biết nghiệp vụ**: Cách diễn giải dữ liệu và tạo ra những thấu hiểu



### Chia sẻ và tái sử dụng Skills

Một đặc tính quan trọng khác của Agent Skills là **tính cộng đồng hóa**. Anthropic đã thiết lập một kho Skills chính thức:

**Kho kỹ năng chính thức**: https://github.com/anthropics/skills

Tính đến năm 2025, đã có hàng trăm kỹ năng do cộng đồng đóng góp, bao trùm:

- **Công cụ phát triển**: Thiết kế frontend, kiểm thử API, rà soát mã, quy trình làm việc Git
- **Phân tích dữ liệu**: Truy vấn SQL, trực quan hóa dữ liệu, phân tích thống kê
- **Xử lý tài liệu**: Phân tích cú pháp PDF, sinh Markdown, viết tài liệu kỹ thuật
- **Quy trình nghiệp vụ**: Quản lý dự án, hỗ trợ khách hàng, rà soát tuân thủ

Việc sử dụng kỹ năng của cộng đồng rất đơn giản:

```bash
# 克隆官方技能库
git clone https://github.com/anthropics/skills.git

# 复制需要的技能到你的项目
cp -r skills/frontend-design ./my-project/skills/

# 智能体会自动发现并加载
```

Bạn cũng có thể chia sẻ kỹ năng của riêng mình:

```bash
# 发布到 GitHub
cd my-custom-skill
git init
git add SKILL.md
git commit -m "Add custom SQL analysis skill"
git remote add origin https://github.com/yourname/my-skill.git
git push -u origin main

# 其他开发者可以直接使用
# git clone https://github.com/yourname/my-skill.git
```

## Động thái ngành và sự tiến hóa của hệ sinh thái

### Tiến trình chuẩn hóa và sự hỗ trợ từ các nhà cung cấp

Mặc dù Agent Skills do Anthropic đề xuất, nhưng triết lý thiết kế của nó đang ảnh hưởng đến toàn ngành.

**Anthropic Claude**:
- Claude Desktop và Claude Code hỗ trợ Skills một cách nguyên bản (native)
- Cung cấp SDK và công cụ phát triển chính thức
- Duy trì kho kỹ năng chính thức

**Phản ứng của OpenAI**:
Mặc dù OpenAI chưa chính thức áp dụng thuật ngữ "Skills", nhưng trong bản cập nhật tháng 3 năm 2025, ChatGPT đã đưa vào những khái niệm tương tự:

- **Tăng cường Custom Instructions**: Hỗ trợ các chỉ dẫn nhiều bước phức tạp hơn
- **Memory và Context Profiles**: Cho phép người dùng lưu và tái sử dụng kiến thức thuộc một lĩnh vực cụ thể
- **Tính năng "cơ sở tri thức" của GPTs**: Có thể đính kèm tài liệu và tập lệnh, nạp theo nhu cầu

Về bản chất, những tính năng này là các hình thức triển khai khác nhau của triết lý Skills.

**Google Vertex AI**:
Google đã đưa vào **"Grounding with Functions"** trong các mô hình Gemini, cho phép nhà phát triển định nghĩa các "gói hàm" (Function Packages), mỗi gói bao gồm:

- Định nghĩa hàm (tương tự tools của MCP)
- Hướng dẫn sử dụng (tương tự instructions của Skills)
- Ví dụ (examples)

Thiết kế này rất giống với kiến trúc lai Skills + MCP.



### Tính tất yếu của kiến trúc phân tầng

Tổng hợp quan điểm từ nhiều phía, chúng tôi cho rằng: **Skills và MCP đại diện cho hai tầng tất yếu phải tách biệt trong kiến trúc Agent**. Khi độ phức tạp của các hệ thống Agent tăng lên, sự phân tầng này là không thể tránh khỏi:

```
应用层（Application Layer）
  ↓ Agent Skills
  ↓ 领域知识、工作流、最佳实践

传输层（Transport Layer）
  ↓ MCP
  ↓ 标准化接口、工具调用、资源访问

基础设施层（Infrastructure Layer）
  ↓ 数据库、API、文件系统、外部服务
```

Điều này hoàn toàn nhất quán với lộ trình tiến hóa của kiến trúc phần mềm truyền thống (từ monolithic đến phân tầng đến microservices), chỉ là được diễn giải lại một lần nữa trong lĩnh vực AI.





### Xu hướng chuẩn hóa

Cùng với việc ngành ngày càng coi trọng công nghệ Agent, chúng tôi dự đoán những xu hướng sau:

**1. Hợp nhất giao thức**

Trong tương lai có thể xuất hiện một giao thức mô tả năng lực Agent thống nhất, dung hòa tính kết nối của MCP và khả năng biểu đạt tri thức của Skills:

```yaml
# 未来的统一协议示例（假想）
apiVersion: agent.io/v1
kind: Capability
metadata:
  name: enterprise-data-analysis
spec:
  transport:
    protocol: mcp
    server: database-mcp-server
    tools: [query, schema]
  knowledge:
    type: skill
    workflow: data-analysis-workflow.md
    examples: examples/
```

**2. Thị trường hóa và hệ sinh thái**

Tương tự như NPM, PyPI, trong tương lai có thể xuất hiện một hệ thống quản lý gói cho năng lực Agent:

```bash
# 假想的未来命令
agent-cli install @anthropic/frontend-design-skill
agent-cli install @google/data-analysis-suite
agent-cli install @openai/code-review-assistant
```

Nhà phát triển có thể phát hành, chia sẻ, bán các Skills và máy chủ MCP của riêng mình, hình thành một hệ sinh thái thịnh vượng.

**3. Tự động phát hiện năng lực**

Agent có thể phát triển các cơ chế tự động phát hiện và học các năng lực mới:

```python
# 未来的智能体可能具备自主学习能力
agent = SelfEvolvingAgent()

# 智能体在执行任务时发现缺少某种能力
response = agent.run("生成 3D 建模文件")

# 智能体自动搜索并安装相关 Skill
# [内部日志] 检测到未知任务类型：3D建模
# [内部日志] 搜索技能库...发现 "blender-3d-modeling" skill
# [内部日志] 请求用户授权安装...已授权
# [内部日志] 技能安装完成，重新执行任务
```

### Thách thức và rủi ro

Đồng thời, chúng ta cũng cần cảnh giác với những rủi ro tiềm ẩn:

<strong>Thách thức về bảo mật</strong>:
- Skills chứa các tập lệnh có thể thực thi, tồn tại rủi ro tiêm mã (code injection)
- Máy chủ MCP có thể phơi bày các giao diện dữ liệu nhạy cảm
- Độ tin cậy của các kỹ năng bên thứ ba khó có thể kiểm chứng

<strong>Ô nhiễm ngữ cảnh</strong>:
- Khi số lượng Skills tăng lên, ngay cả siêu dữ liệu cũng có thể chiếm dụng một lượng lớn ngữ cảnh
- Cần có cơ chế lập chỉ mục và truy xuất kỹ năng thông minh hơn

<strong>Rủi ro phân mảnh</strong>:
- Mặc dù MCP đang được chuẩn hóa, nhưng định dạng Skills vẫn chưa được thống nhất
- Các nhà cung cấp khác nhau có thể tung ra những quy chuẩn Skills không tương thích với nhau



## Tổng kết

Agent Skills và MCP đại diện cho hai tầng trừu tượng then chốt trong ngăn xếp công nghệ (tech stack) của Agent:

- <strong>MCP (Model Context Protocol)</strong>: Giải quyết bài toán "tính kết nối", là giao diện chuẩn hóa để Agent tương tác với thế giới bên ngoài, tương đương với "hệ thần kinh" hoặc "đôi tay"
- <strong>Agent Skills</strong>: Giải quyết bài toán "năng lực", là sự đóng gói kiến thức lĩnh vực và quy trình làm việc, tương đương với "vỏ não" hoặc "cẩm nang vận hành"

Hai công nghệ này không phải là quan hệ cạnh tranh, mà là quan hệ bổ trợ cho nhau:

<div align="center">
  <img src="./images/Extra05-figures/image6.png" alt="MCP与Agent Skills总结对比" width="90%"/>
  <p>Hình 6 Tổng kết so sánh toàn diện giữa MCP và Agent Skills</p>
</div>


<strong>Những thấu hiểu then chốt</strong>:

1. <strong>Kiến trúc phân tầng là xu hướng tất yếu</strong>: Khi độ phức tạp của các hệ thống Agent tăng lên, sự phân tách giữa "tầng kết nối" và "tầng tri thức" là không thể tránh khỏi

2. <strong>Hiệu quả ngữ cảnh là mâu thuẫn cốt lõi</strong>: Cơ chế tiết lộ tiệm tiến của Skills giảm mức tiêu thụ token xuống hơn 90%, đây là lợi thế kỹ thuật lớn nhất của nó

3. <strong>Dân chủ hóa kiến thức lĩnh vực</strong>: Skills giúp cả những người không phải nhà phát triển cũng có thể đóng góp năng lực cho Agent, điều này sẽ mở rộng đáng kể biên giới của các ứng dụng AI

4. <strong>Kiến trúc lai là thực hành tốt nhất</strong>: Trong các ứng dụng cấp doanh nghiệp, MCP cung cấp kết nối hạ tầng, Skills cung cấp logic nghiệp vụ, kết hợp cả hai mới có thể xây dựng được hệ thống Agent hiệu quả và dễ bảo trì

<strong>Khuyến nghị thực hành</strong>:

- Đối với <strong>kết nối dịch vụ bên ngoài</strong> (cơ sở dữ liệu, API, dịch vụ đám mây), ưu tiên sử dụng MCP
- Đối với <strong>quy trình làm việc phức tạp</strong> (tác vụ nhiều bước, kiến thức chuyên môn lĩnh vực), ưu tiên sử dụng Skills
- Trong các kịch bản <strong>ngữ cảnh bị giới hạn</strong> (hội thoại dài, nhiều công cụ), sử dụng Skills để quản lý tiệm tiến
- Khi xây dựng <strong>Agent cấp doanh nghiệp</strong>, áp dụng kiến trúc phân tầng MCP + Skills

Qua việc học chương này, bạn sẽ có thể:
- Hiểu được sự khác biệt bản chất và quan hệ hợp tác giữa Agent Skills và MCP
- Nắm vững cơ chế tiết lộ tiệm tiến của Skills cùng những ưu điểm của nó
- Viết được các tệp SKILL.md chất lượng cao
- Lựa chọn và kết hợp hai công nghệ một cách hợp lý trong các dự án thực tế
- Xây dựng được hệ thống Agent phân tầng rõ ràng, hiệu quả và dễ bảo trì

Công nghệ Agent vẫn đang tiến hóa nhanh chóng. MCP đã trở thành tiêu chuẩn thực tế (de facto) của tầng kết nối, và triết lý của Skills cũng đang ảnh hưởng đến toàn ngành. Nắm vững hai công nghệ này sẽ giúp bạn xây dựng những ứng dụng Agent mạnh mẽ và thực dụng hơn trong làn sóng AI.

---

## Tài liệu tham khảo

1. Tài liệu chính thức Anthropic Agent Skills: https://docs.anthropic.com/en/docs/agent-skills
2. Kho GitHub Anthropic Skills: https://github.com/anthropics/skills
3. Quy chuẩn Model Context Protocol: https://modelcontextprotocol.io/
4. Blog của Anthropic: Improving Frontend Design Through Skills: https://www.claude.com/blog/improving-frontend-design-through-skills
5. Chương mười: Giao thức truyền thông giữa các Agent (hello-agents)
