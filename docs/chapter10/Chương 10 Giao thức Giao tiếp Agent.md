# Chương 10 Giao thức Giao tiếp Agent

Trong các chương trước, chúng ta đã xây dựng các agent độc lập với đầy đủ chức năng, có khả năng suy luận, gọi công cụ (tool) và ghi nhớ. Tuy nhiên, khi cố gắng xây dựng những hệ thống AI phức tạp hơn, những câu hỏi tự nhiên sẽ nảy sinh: **Làm thế nào để agent tương tác hiệu quả với thế giới bên ngoài? Làm thế nào để nhiều agent cộng tác với nhau?**

Đây chính xác là vấn đề cốt lõi mà các giao thức giao tiếp agent (agent communication protocol) hướng đến giải quyết. Chương này sẽ giới thiệu ba giao thức giao tiếp vào framework HelloAgents: **MCP (Model Context Protocol)** dùng cho giao tiếp chuẩn hóa giữa agent và công cụ, **A2A (Agent-to-Agent Protocol)** dùng cho sự cộng tác ngang hàng giữa các agent, và **ANP (Agent Network Protocol)** dùng để xây dựng các mạng lưới agent quy mô lớn. Ba giao thức này cùng nhau tạo thành tầng hạ tầng cho giao tiếp giữa các agent.

Qua việc học chương này, bạn sẽ nắm vững triết lý thiết kế và kỹ năng thực hành của các giao thức giao tiếp agent, hiểu được những khác biệt trong thiết kế giữa ba giao thức chủ đạo, và học cách chọn giao thức phù hợp để giải quyết các vấn đề thực tế.

## 10.1 Nền tảng của Giao thức Giao tiếp Agent

### 10.1.1 Vì sao cần Giao thức Giao tiếp

Hãy nhớ lại agent ReAct mà chúng ta đã xây dựng ở Chương 7, vốn đã sở hữu khả năng suy luận và gọi công cụ mạnh mẽ. Hãy xem một kịch bản sử dụng điển hình:

```python
from hello_agents import ReActAgent, HelloAgentsLLM
from hello_agents.tools import CalculatorTool, SearchTool

llm = HelloAgentsLLM()
agent = ReActAgent(name="AI Assistant", llm=llm)
agent.add_tool(CalculatorTool())
agent.add_tool(SearchTool())

# Agent có thể hoàn thành nhiệm vụ một cách độc lập
response = agent.run("Search for the latest AI news and calculate the total market value of related companies")
```

Agent này hoạt động tốt, nhưng nó phải đối mặt với ba hạn chế cơ bản. Thứ nhất là **nan đề tích hợp công cụ**: Mỗi khi chúng ta cần truy cập một dịch vụ bên ngoài mới (chẳng hạn như GitHub API, cơ sở dữ liệu, hệ thống tập tin), chúng ta phải viết một lớp Tool chuyên biệt. Điều này không chỉ tốn nhiều công sức, mà các công cụ do những nhà phát triển khác nhau viết ra cũng không thể tương thích với nhau. Thứ hai là **nút thắt cổ chai trong mở rộng khả năng**: Khả năng của agent bị giới hạn trong tập công cụ được định nghĩa trước và không thể khám phá cũng như sử dụng các dịch vụ mới một cách động. Cuối cùng là **thiếu sự cộng tác**: Khi nhiệm vụ đủ phức tạp đến mức cần nhiều agent chuyên biệt cộng tác (chẳng hạn như researcher + writer + editor), chúng ta chỉ có thể điều phối công việc của chúng thông qua việc dàn xếp thủ công.

Hãy hiểu những hạn chế này qua một ví dụ cụ thể hơn. Giả sử bạn muốn xây dựng một trợ lý nghiên cứu thông minh cần:

```python
# Cách tiếp cận truyền thống: Tích hợp thủ công từng dịch vụ
class GitHubTool(BaseTool):
    """Cần viết thủ công adapter cho GitHub API"""
    def run(self, repo_url):
        # Rất nhiều mã gọi API...
        pass

class DatabaseTool(BaseTool):
    """Cần viết thủ công adapter cho cơ sở dữ liệu"""
    def run(self, query):
        # Mã kết nối và truy vấn cơ sở dữ liệu...
        pass

class WeatherTool(BaseTool):
    """Cần viết thủ công adapter cho weather API"""
    def run(self, location):
        # Mã gọi weather API...
        pass

# Mỗi dịch vụ mới đều đòi hỏi lặp lại quá trình này
agent.add_tool(GitHubTool())
agent.add_tool(DatabaseTool())
agent.add_tool(WeatherTool())
```

Cách tiếp cận này có những vấn đề rõ ràng: trùng lặp mã (mỗi công cụ đều phải xử lý HTTP request, xử lý lỗi, xác thực, v.v.), khó bảo trì (khi API thay đổi thì phải sửa tất cả các công cụ liên quan), không thể tái sử dụng (không thể dùng trực tiếp công cụ của những nhà phát triển khác), khả năng mở rộng kém (thêm dịch vụ mới đòi hỏi khối lượng lập trình lớn).

**Giá trị cốt lõi của các giao thức giao tiếp** chính là để giải quyết những vấn đề này. Nó cung cấp một tập các đặc tả giao diện chuẩn hóa, cho phép agent truy cập nhiều loại dịch vụ bên ngoài theo một cách thức thống nhất mà không cần phải viết adapter chuyên biệt cho từng dịch vụ. Điều này giống như giao thức TCP/IP của Internet, cho phép các thiết bị khác nhau giao tiếp với nhau mà không cần viết mã giao tiếp chuyên biệt cho mỗi loại thiết bị.

Với các giao thức giao tiếp, đoạn mã trên có thể được rút gọn thành:

```python
from hello_agents.tools import MCPTool

# Kết nối tới MCP server, tự động lấy được tất cả các công cụ
mcp_tool = MCPTool()  # Server tích hợp sẵn cung cấp các công cụ cơ bản

# Hoặc kết nối tới các MCP server chuyên nghiệp
github_mcp = MCPTool(server_command=["npx", "-y", "@modelcontextprotocol/server-github"])
database_mcp = MCPTool(server_command=["python", "database_mcp_server.py"])

# Agent tự động lấy được tất cả khả năng mà không cần viết adapter thủ công
agent.add_tool(mcp_tool)
agent.add_tool(github_mcp)
agent.add_tool(database_mcp)
```

Những thay đổi mà giao thức giao tiếp mang lại mang tính nền tảng: **Giao diện chuẩn hóa** cho phép các dịch vụ khác nhau cung cấp phương thức truy cập thống nhất, **khả năng tương tác (interoperability)** giúp tích hợp liền mạch các công cụ từ những nhà phát triển khác nhau, **khám phá động (dynamic discovery)** cho phép agent phát hiện các dịch vụ và khả năng mới ngay trong lúc chạy, và **khả năng mở rộng (scalability)** giúp hệ thống dễ dàng bổ sung các mô-đun chức năng mới.

### 10.1.2 So sánh Triết lý Thiết kế của Ba Giao thức

Giao thức giao tiếp agent không phải là một giải pháp đơn lẻ, mà là một loạt các tiêu chuẩn được thiết kế cho những kịch bản giao tiếp khác nhau. Chương này lấy ba giao thức chủ đạo hiện nay là MCP, A2A và ANP làm ví dụ để thực hành. Dưới đây là phần so sánh tổng quan.

**(1) MCP: Cầu nối giữa Agent và Công cụ**

MCP (Model Context Protocol) do đội ngũ Anthropic đề xuất<sup>[1]</sup>, và triết lý thiết kế cốt lõi của nó là **chuẩn hóa phương thức giao tiếp giữa agent và các công cụ/tài nguyên bên ngoài**. Hãy tưởng tượng agent của bạn cần truy cập nhiều dịch vụ khác nhau như hệ thống tập tin, cơ sở dữ liệu, GitHub, Slack, v.v. Cách tiếp cận truyền thống là viết adapter chuyên biệt cho từng dịch vụ, việc này không chỉ tốn công mà còn khó bảo trì. MCP định nghĩa một đặc tả giao thức thống nhất, cho phép mọi dịch vụ được truy cập theo cùng một cách.

Triết lý thiết kế của MCP là "chia sẻ ngữ cảnh (context sharing)". Nó không chỉ là một giao thức RPC (Remote Procedure Call - Gọi Thủ tục Từ xa), mà quan trọng hơn, nó cho phép agent và công cụ chia sẻ thông tin ngữ cảnh phong phú. Như minh họa trong Hình 10.1, khi một agent truy cập một kho mã nguồn, MCP server không chỉ có thể cung cấp nội dung tập tin mà còn cung cấp thông tin ngữ cảnh như cấu trúc mã, quan hệ phụ thuộc, và lịch sử commit, giúp agent đưa ra quyết định thông minh hơn.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-1.png" alt="" width="85%"/>
  <p>Hình 10.1 Triết lý Thiết kế của MCP</p>
</div>

**(2) A2A: Đối thoại giữa các Agent**

Giao thức A2A (Agent-to-Agent Protocol) do đội ngũ Google đề xuất<sup>2</sup>, và triết lý thiết kế cốt lõi của nó là **hiện thực hóa giao tiếp ngang hàng (peer-to-peer) giữa các agent**. Khác với MCP vốn tập trung vào giao tiếp giữa agent và công cụ, A2A tập trung vào cách các agent cộng tác với nhau. Thiết kế này cho phép các agent đối thoại, thương lượng và cộng tác giống như những đội nhóm con người.

Triết lý thiết kế của A2A là "giao tiếp ngang hàng (peer-to-peer communication)". Như minh họa trong Hình 10.2, trong một mạng A2A, mỗi agent vừa là nhà cung cấp dịch vụ vừa là bên tiêu thụ dịch vụ. Các agent có thể chủ động khởi tạo yêu cầu và cũng có thể đáp ứng yêu cầu từ các agent khác. Thiết kế ngang hàng này tránh được nút thắt cổ chai của bộ điều phối tập trung, khiến mạng lưới agent trở nên linh hoạt và dễ mở rộng hơn.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-2.png" alt="" width="85%"/>
  <p>Hình 10.2 Triết lý Thiết kế của A2A</p>
</div>

**(3) ANP: Hạ tầng cho Mạng lưới Agent**

ANP (Agent Network Protocol) là một khung giao thức mang tính khái niệm<sup>3</sup>, hiện được cộng đồng mã nguồn mở duy trì và chưa có một hệ sinh thái trưởng thành. Triết lý thiết kế cốt lõi của nó là **xây dựng hạ tầng cho các mạng lưới agent quy mô lớn**. Nếu MCP giải quyết bài toán "làm thế nào để truy cập công cụ" và A2A giải quyết "làm thế nào để đối thoại với các agent khác", thì ANP giải quyết "làm thế nào để khám phá và kết nối các agent trong những mạng lưới quy mô lớn".

Triết lý thiết kế của ANP là "khám phá dịch vụ phi tập trung (decentralized service discovery)". Trong một mạng chứa hàng trăm hoặc hàng nghìn agent, làm thế nào để một agent tìm được dịch vụ mà nó cần? Như minh họa trong Hình 10.3, ANP cung cấp các cơ chế đăng ký, khám phá và định tuyến dịch vụ, cho phép agent khám phá động các dịch vụ khác trong mạng mà không cần cấu hình trước tất cả các quan hệ kết nối.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-3.png" alt="" width="85%"/>
  <p>Hình 10.3 Triết lý Thiết kế của ANP</p>
</div>

Cuối cùng, trong Bảng 10.1, hãy dùng một bảng so sánh để hiểu rõ hơn sự khác biệt giữa ba giao thức này:

<div align="center">
  <p>Bảng 10.1 So sánh Ba Giao thức</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-1.png" alt="" width="85%"/>
</div>

**(4) Làm thế nào để Chọn Giao thức Phù hợp?**

Các giao thức hiện tại vẫn đang ở giai đoạn phát triển ban đầu. Hệ sinh thái của MCP tương đối trưởng thành, mặc dù tính cập nhật kịp thời của các công cụ khác nhau còn phụ thuộc vào người duy trì. Nên ưu tiên lựa chọn các công cụ MCP được các công ty lớn hậu thuẫn.

Chìa khóa để chọn giao thức nằm ở việc hiểu rõ nhu cầu của bạn:

- Nếu agent của bạn cần truy cập các dịch vụ bên ngoài (tập tin, cơ sở dữ liệu, API), hãy chọn **MCP**
- Nếu bạn cần nhiều agent cộng tác để hoàn thành nhiệm vụ, hãy chọn **A2A**
- Nếu bạn muốn xây dựng một hệ sinh thái agent quy mô lớn, hãy cân nhắc **ANP**

### 10.1.3 Thiết kế Kiến trúc Giao thức Giao tiếp của HelloAgents

Sau khi đã hiểu triết lý thiết kế của ba giao thức, hãy xem cách hiện thực và sử dụng chúng trong framework HelloAgents. Mục tiêu thiết kế của chúng tôi là: **Cho phép người học sử dụng các giao thức này theo cách đơn giản nhất, đồng thời vẫn duy trì đủ độ linh hoạt để xử lý các kịch bản phức tạp**.

Như minh họa trong Hình 10.4, kiến trúc giao thức giao tiếp của HelloAgents áp dụng thiết kế ba tầng, từ dưới lên trên: tầng hiện thực giao thức, tầng đóng gói công cụ, và tầng tích hợp agent.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-4.png" alt="" width="85%"/>
  <p>Hình 10.4 Thiết kế Giao thức Giao tiếp của HelloAgents</p>
</div>

**(1) Tầng Hiện thực Giao thức (Protocol Implementation Layer)**: Tầng này chứa các hiện thực cụ thể của ba giao thức. MCP được hiện thực dựa trên thư viện FastMCP, cung cấp chức năng client và server; A2A được hiện thực dựa trên a2a-sdk chính thức của Google; ANP là hiện thực nhẹ do chúng tôi tự phát triển, cung cấp chức năng khám phá dịch vụ và quản lý mạng. Tất nhiên, hiện cũng đã có một [hiện thực](https://github.com/agent-network-protocol/AgentConnect) chính thức, nhưng xét đến những lần lặp phát triển trong tương lai, ở đây chúng tôi chỉ mô phỏng khái niệm.

**(2) Tầng Đóng gói Công cụ (Tool Encapsulation Layer)**: Tầng này đóng gói các hiện thực giao thức thành một giao diện Tool thống nhất. MCPTool, A2ATool và ANPTool đều kế thừa từ BaseTool, cung cấp phương thức `run()` nhất quán. Thiết kế này cho phép agent sử dụng các giao thức khác nhau theo cùng một cách.

**(3) Tầng Tích hợp Agent (Agent Integration Layer)**: Tầng này là điểm tích hợp giữa agent và giao thức. Tất cả các agent (ReActAgent, SimpleAgent, v.v.) sử dụng các công cụ giao thức thông qua Tool System mà không cần quan tâm đến chi tiết giao thức bên dưới.

### 10.1.4 Mục tiêu Học tập và Trải nghiệm Nhanh của Chương này

Trước tiên hãy xem nội dung học tập của Chương 10:

```
hello_agents/
├── protocols/                          # Mô-đun giao thức giao tiếp
│   ├── mcp/                            # Hiện thực giao thức MCP (Model Context Protocol)
│   │   ├── client.py                   # MCP client (hỗ trợ 5 phương thức truyền tải)
│   │   ├── server.py                   # MCP server (bao bọc FastMCP)
│   │   └── utils.py                    # Hàm tiện ích (create_context/parse_context)
│   ├── a2a/                            # Hiện thực giao thức A2A (Agent-to-Agent Protocol)
│   │   └── implementation.py           # A2A server/client (dựa trên a2a-sdk, phụ thuộc tùy chọn)
│   └── anp/                            # Hiện thực giao thức ANP (Agent Network Protocol)
│       └── implementation.py           # Khám phá/đăng ký dịch vụ ANP (hiện thực mang tính khái niệm)
└── tools/builtin/                      # Mô-đun công cụ tích hợp sẵn
    └── protocol_tools.py               # Lớp bao bọc công cụ giao thức (MCPTool/A2ATool/ANPTool)
```

Đối với nội dung chương này, trọng tâm chủ yếu là ứng dụng, và mục tiêu học tập là có khả năng áp dụng các giao thức trong dự án của riêng bạn. Ngoài ra, vì các giao thức hiện đang ở giai đoạn phát triển ban đầu, không cần dồn quá nhiều công sức để phát minh lại bánh xe. Trước khi bắt đầu thực hành, hãy chuẩn bị môi trường phát triển:

```bash
# Cài đặt framework HelloAgents (phiên bản Chương 10)
pip install "hello-agents[protocol]==0.2.2"

# Cài đặt NodeJS, tham khảo tài liệu trong Additional-Chapter
```

Hãy trải nghiệm chức năng cơ bản của ba giao thức với đoạn mã đơn giản nhất:

```python
from hello_agents.tools import MCPTool, A2ATool, ANPTool

# 1. MCP: Truy cập công cụ
mcp_tool = MCPTool()
result = mcp_tool.run({
    "action": "call_tool",
    "tool_name": "add",
    "arguments": {"a": 10, "b": 20}
})
print(f"MCP calculation result: {result}")  # Output: 30.0

# 2. ANP: Khám phá dịch vụ
anp_tool = ANPTool()
anp_tool.run({
    "action": "register_service",
    "service_id": "calculator",
    "service_type": "math",
    "endpoint": "http://localhost:8080"
})
services = anp_tool.run({"action": "discover_services"})
print(f"Discovered services: {services}")

# 3. A2A: Giao tiếp giữa agent
a2a_tool = A2ATool("http://localhost:5000")
print("A2A tool created successfully")
```

Ví dụ đơn giản này minh họa chức năng cốt lõi của ba giao thức. Trong các phần tiếp theo, chúng ta sẽ học sâu về cách sử dụng chi tiết và các thực hành tốt nhất của từng giao thức.


## 10.2 Thực hành Giao thức MCP

Bây giờ, hãy đi sâu vào MCP và nắm vững cách cho phép agent truy cập các công cụ và tài nguyên bên ngoài.

### 10.2.1 Giới thiệu Khái niệm Giao thức MCP

**(1) MCP: "USB-C" dành cho Agent**

Hãy tưởng tượng agent của bạn có thể cần làm nhiều việc cùng lúc, chẳng hạn như:
- Đọc tài liệu từ hệ thống tập tin cục bộ
- Truy vấn cơ sở dữ liệu PostgreSQL
- Tìm kiếm mã trên GitHub
- Gửi tin nhắn Slack
- Truy cập Google Drive

Theo cách truyền thống, bạn sẽ cần viết mã adapter cho từng dịch vụ, xử lý các API khác nhau, các phương thức xác thực, xử lý lỗi, v.v. Việc này không chỉ tốn công mà còn khó bảo trì. Quan trọng hơn, các nền tảng LLM khác nhau có cách hiện thực function call rất khác nhau, đòi hỏi phải viết lại nhiều mã khi chuyển đổi mô hình.

Sự xuất hiện của MCP đã thay đổi tất cả. Giống như USB-C thống nhất phương thức kết nối cho nhiều loại thiết bị, **MCP thống nhất phương thức tương tác giữa agent và các công cụ bên ngoài**. Dù bạn dùng Claude, GPT, hay các mô hình khác, chỉ cần chúng hỗ trợ giao thức MCP, chúng đều có thể truy cập liền mạch cùng một bộ công cụ và tài nguyên.

**(2) Kiến trúc MCP**

Giao thức MCP áp dụng thiết kế kiến trúc ba tầng gồm Host, Client và Servers. Hãy hiểu cách các thành phần này phối hợp với nhau qua kịch bản trong Hình 10.5.

Giả sử bạn đang dùng Claude Desktop và hỏi: "Trên desktop của tôi có những tài liệu gì?"

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-5.png" alt="" width="85%"/>
  <p>Hình 10.5 Minh họa Trường hợp MCP</p>
</div>

**Trách nhiệm của Kiến trúc Ba tầng:**

1. **Host (Tầng Host)**: Claude Desktop đóng vai trò Host, chịu trách nhiệm nhận câu hỏi của người dùng và tương tác với mô hình Claude. Host là giao diện mà người dùng trực tiếp tương tác, quản lý toàn bộ luồng hội thoại.

2. **Client (Tầng Client)**: Khi mô hình Claude quyết định cần truy cập hệ thống tập tin, MCP Client được tích hợp sẵn trong Host sẽ được kích hoạt. Client chịu trách nhiệm thiết lập kết nối với MCP Server phù hợp, gửi yêu cầu và nhận phản hồi.

3. **Server (Tầng Server)**: MCP Server của hệ thống tập tin được gọi, thực thi thao tác quét tập tin thực tế, truy cập thư mục desktop, và trả về danh sách các tài liệu tìm thấy.

**Luồng Tương tác Hoàn chỉnh:** Câu hỏi của người dùng → Claude Desktop (Host) → Mô hình Claude phân tích → Cần thông tin tập tin → MCP Client kết nối → MCP Server của hệ thống tập tin → Thực thi thao tác → Trả về kết quả → Claude tạo câu trả lời → Hiển thị trên Claude Desktop

Ưu điểm của thiết kế kiến trúc này nằm ở **sự tách biệt các mối quan tâm (separation of concerns)**: Host tập trung vào trải nghiệm người dùng, Client tập trung vào giao tiếp giao thức, và Server tập trung vào việc hiện thực chức năng cụ thể. Nhà phát triển chỉ cần tập trung phát triển MCP Server tương ứng mà không cần quan tâm đến chi tiết hiện thực của Host và Client.

**(3) Các Khả năng Cốt lõi của MCP**

Như minh họa trong Bảng 10.2, giao thức MCP cung cấp ba khả năng cốt lõi, tạo thành một khung truy cập công cụ hoàn chỉnh:

<div align="center">
  <p>Bảng 10.2 Các Khả năng Cốt lõi của MCP</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-2.png" alt="" width="85%"/>
</div>

Sự khác biệt giữa ba khả năng này là: **Tools chủ động** (thực thi thao tác), **Resources thụ động** (cung cấp dữ liệu), **Prompts mang tính chỉ dẫn** (cung cấp mẫu template).

**(4) Quy trình Làm việc của MCP**

Hãy hiểu quy trình làm việc hoàn chỉnh của MCP qua một ví dụ cụ thể, như minh họa trong Hình 10.6:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-6.png" alt="" width="85%"/>
  <p>Hình 10.6 Minh họa Trường hợp MCP</p>
</div>

Một câu hỏi then chốt là: **Claude (hay các LLM khác) quyết định dùng công cụ nào như thế nào?**

Khi người dùng đặt câu hỏi, toàn bộ quy trình chọn công cụ diễn ra như sau:

1. **Giai đoạn Khám phá Công cụ**: Sau khi MCP Client kết nối tới Server, trước tiên nó gọi `list_tools()` để lấy thông tin mô tả của tất cả các công cụ khả dụng (bao gồm tên công cụ, mô tả chức năng, định nghĩa tham số)

2. **Xây dựng Ngữ cảnh**: Client chuyển đổi danh sách công cụ thành định dạng mà LLM có thể hiểu và thêm vào system prompt. Ví dụ:
   ```
   You can use the following tools:
   - read_file(path: str): Read the content of the file at the specified path
   - search_code(query: str, language: str): Search in the codebase
   ```

3. **Suy luận của Mô hình**: LLM phân tích câu hỏi của người dùng và các công cụ khả dụng, quyết định có gọi công cụ hay không và gọi công cụ nào. Quyết định này dựa trên phần mô tả công cụ và ngữ cảnh hội thoại hiện tại

4. **Thực thi Công cụ**: Nếu LLM quyết định dùng một công cụ, Client thực thi công cụ được chọn thông qua MCP Server và lấy kết quả

5. **Tích hợp Kết quả**: Kết quả thực thi công cụ được gửi trở lại cho LLM, LLM kết hợp kết quả để tạo câu trả lời cuối cùng

Quy trình này là **hoàn toàn tự động**, và LLM sẽ quyết định có dùng công cụ hay không và dùng như thế nào dựa trên chất lượng của phần mô tả công cụ. Vì vậy, việc viết mô tả công cụ rõ ràng và chính xác là vô cùng quan trọng.

**(5) Sự khác biệt giữa MCP và Function Calling**

Nhiều nhà phát triển hỏi: **Tôi đã dùng Function Calling rồi, tại sao vẫn cần MCP?** Hãy hiểu sự khác biệt của chúng qua Bảng 10.3.

<div align="center">
  <p>Bảng 10.3 So sánh Function Calling và MCP</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-3.png" alt="" width="85%"/>
</div>

Ở đây chúng ta dùng ví dụ một agent cần truy cập kho GitHub và hệ thống tập tin cục bộ để so sánh chi tiết hai cách hiện thực cùng một nhiệm vụ.

**Cách 1: Dùng Function Calling**

```python
# Bước 1: Định nghĩa hàm cho từng nhà cung cấp LLM
# Định dạng OpenAI
openai_tools = [
    {
        "type": "function",
        "function": {
            "name": "search_github",
            "description": "Search GitHub repositories",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "Search keywords"}
                },
                "required": ["query"]
            }
        }
    }
]

# Định dạng Claude
claude_tools = [
    {
        "name": "search_github",
        "description": "Search GitHub repositories",
        "input_schema": {  # Lưu ý: không phải parameters
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "Search keywords"}
            },
            "required": ["query"]
        }
    }
]

# Bước 2: Tự hiện thực các hàm công cụ
def search_github(query):
    import requests
    response = requests.get(
        "https://api.github.com/search/repositories",
        params={"q": query}
    )
    return response.json()

# Bước 3: Xử lý các định dạng phản hồi khác nhau của từng mô hình
# Phản hồi OpenAI
if response.choices[0].message.tool_calls:
    tool_call = response.choices[0].message.tool_calls[0]
    result = search_github(**json.loads(tool_call.function.arguments))

# Phản hồi Claude
if response.content[0].type == "tool_use":
    tool_use = response.content[0]
    result = search_github(**tool_use.input)
```

**Cách 2: Dùng MCP**

```python
from hello_agents.protocols import MCPClient

# Bước 1: Kết nối tới MCP server do cộng đồng cung cấp (không cần tự hiện thực)
github_client = MCPClient([
    "npx", "-y", "@modelcontextprotocol/server-github"
])

fs_client = MCPClient([
    "npx", "-y", "@modelcontextprotocol/server-filesystem", "."
])

# Bước 2: Phương thức gọi thống nhất (độc lập với mô hình)
async with github_client:
    # Tự động khám phá công cụ
    tools = await github_client.list_tools()

    # Gọi công cụ (giao diện chuẩn hóa)
    result = await github_client.call_tool(
        "search_repositories",
        {"query": "AI agents"}
    )

# Bước 3: Bất kỳ mô hình nào hỗ trợ MCP đều có thể dùng
# OpenAI, Claude, Llama, v.v. đều dùng cùng một MCP client
```

Trước hết, cần làm rõ rằng Function Calling và MCP không cạnh tranh nhau, mà bổ trợ cho nhau. Function Calling là một khả năng cốt lõi của các mô hình ngôn ngữ lớn, phản ánh trí thông minh vốn có của mô hình, giúp mô hình hiểu khi nào cần gọi hàm và tạo ra các tham số gọi tương ứng một cách chính xác. Ngược lại, MCP đóng vai trò một giao thức hạ tầng, giải quyết bài toán kỹ thuật về cách công cụ kết nối với mô hình ở tầng kỹ thuật, mô tả và gọi công cụ theo một cách chuẩn hóa.

Chúng ta có thể dùng một phép so sánh đơn giản để hiểu: Function Calling tương đương với việc học kỹ năng "làm thế nào để gọi điện thoại", bao gồm khi nào bấm số, giao tiếp với đầu bên kia ra sao, và khi nào gác máy. Còn MCP chính là "tiêu chuẩn liên lạc điện thoại" thống nhất toàn cầu, đảm bảo bất kỳ chiếc điện thoại nào cũng có thể gọi thành công tới một chiếc khác.

Sau khi đã hiểu mối quan hệ bổ trợ giữa chúng, tiếp theo hãy xem cách dùng giao thức MCP trong HelloAgents.

### 10.2.2 Sử dụng MCP Client

HelloAgents hiện thực đầy đủ chức năng MCP client dựa trên FastMCP 2.0. Chúng tôi cung cấp cả API bất đồng bộ (asynchronous) và đồng bộ (synchronous) để phù hợp với các kịch bản sử dụng khác nhau. Đối với hầu hết các ứng dụng, API bất đồng bộ được khuyến nghị vì nó xử lý tốt hơn các yêu cầu đồng thời và các thao tác chạy lâu. Dưới đây chúng ta sẽ trình bày thao tác từng bước.

**(1) Kết nối tới MCP Server**

MCP client hỗ trợ nhiều phương thức kết nối, phổ biến nhất là chế độ Stdio (giao tiếp với tiến trình cục bộ thông qua standard input/output):

```python
import asyncio
from hello_agents.protocols import MCPClient

async def connect_to_server():
    # Cách 1: Kết nối tới server hệ thống tập tin do cộng đồng cung cấp
    # npx sẽ tự động tải và chạy gói @modelcontextprotocol/server-filesystem
    client = MCPClient([
        "npx", "-y",
        "@modelcontextprotocol/server-filesystem",
        "."  # Chỉ định thư mục gốc
    ])

    # Dùng async with để đảm bảo kết nối được đóng đúng cách
    async with client:
        # Dùng client ở đây
        tools = await client.list_tools()
        print(f"Available tools: {[t['name'] for t in tools]}")

    # Cách 2: Kết nối tới MCP server Python tùy chỉnh
    client = MCPClient(["python", "my_mcp_server.py"])
    async with client:
        # Dùng client...
        pass

# Chạy hàm bất đồng bộ
asyncio.run(connect_to_server())
```

**(2) Khám phá các Công cụ Khả dụng**

Sau khi kết nối thành công, bước đầu tiên thường là truy vấn xem server cung cấp những công cụ gì:

```python
async def discover_tools():
    client = MCPClient(["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

    async with client:
        # Lấy tất cả các công cụ khả dụng
        tools = await client.list_tools()

        print(f"Server provides {len(tools)} tools:")
        for tool in tools:
            print(f"\nTool name: {tool['name']}")
            print(f"Description: {tool.get('description', 'No description')}")

            # In thông tin tham số
            if 'inputSchema' in tool:
                schema = tool['inputSchema']
                if 'properties' in schema:
                    print("Parameters:")
                    for param_name, param_info in schema['properties'].items():
                        param_type = param_info.get('type', 'any')
                        param_desc = param_info.get('description', '')
                        print(f"  - {param_name} ({param_type}): {param_desc}")

asyncio.run(discover_tools())

# Ví dụ output:
# Server provides 5 tools:
#
# Tool name: read_file
# Description: Read file content
# Parameters:
#   - path (string): File path
#
# Tool name: write_file
# Description: Write file content
# Parameters:
#   - path (string): File path
#   - content (string): File content
```

**(3) Gọi Công cụ**

Khi gọi công cụ, chỉ cần cung cấp tên công cụ và các tham số tuân theo JSON Schema:

```python
async def use_tools():
    client = MCPClient(["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

    async with client:
        # Đọc tập tin
        result = await client.call_tool("read_file", {"path": "my_README.md"})
        print(f"File content:\n{result}")

        # Liệt kê thư mục
        result = await client.call_tool("list_directory", {"path": "."})
        print(f"Current directory files: {result}")

        # Ghi tập tin
        result = await client.call_tool("write_file", {
            "path": "output.txt",
            "content": "Hello from MCP!"
        })
        print(f"Write result: {result}")

asyncio.run(use_tools())
```

Dưới đây là một cách gọi dịch vụ MCP an toàn hơn để tham khảo:

```python
async def safe_tool_call():
    client = MCPClient(["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

    async with client:
        try:
            # Thử đọc một tập tin có thể không tồn tại
            result = await client.call_tool("read_file", {"path": "nonexistent.txt"})
            print(result)
        except Exception as e:
            print(f"Tool call failed: {e}")
            # Có thể chọn thử lại, dùng giá trị mặc định, hoặc báo lỗi cho người dùng

asyncio.run(safe_tool_call())
```

**(4) Truy cập Tài nguyên (Resources)**

Ngoài các công cụ, MCP server còn có thể cung cấp tài nguyên (resources):

```python
# Liệt kê các tài nguyên khả dụng
resources = client.list_resources()
print(f"Available resources: {[r['uri'] for r in resources]}")

# Đọc tài nguyên
resource_content = client.read_resource("file:///path/to/resource")
print(f"Resource content: {resource_content}")
```

**(5) Sử dụng Mẫu Prompt (Prompt Templates)**

MCP server có thể cung cấp các mẫu prompt được định nghĩa trước:

```python
# Liệt kê các prompt khả dụng
prompts = client.list_prompts()
print(f"Available prompts: {[p['name'] for p in prompts]}")

# Lấy nội dung prompt
prompt = client.get_prompt("code_review", {"language": "python"})
print(f"Prompt content: {prompt}")
```

**(6) Ví dụ Hoàn chỉnh: Sử dụng Dịch vụ GitHub MCP**

Hãy xem cách dùng dịch vụ GitHub MCP do cộng đồng cung cấp qua một ví dụ hoàn chỉnh, sử dụng MCP Tools đã được đóng gói:

```python
"""
Ví dụ Dịch vụ GitHub MCP

Lưu ý: Cần thiết lập biến môi trường
    Windows: $env:GITHUB_PERSONAL_ACCESS_TOKEN="your_token_here"
    Linux/macOS: export GITHUB_PERSONAL_ACCESS_TOKEN="your_token_here"
"""

from hello_agents.tools import MCPTool

# Tạo công cụ GitHub MCP
github_tool = MCPTool(
    server_command=["npx", "-y", "@modelcontextprotocol/server-github"]
)

# 1. Liệt kê các công cụ khả dụng
print("📋 Available tools:")
result = github_tool.run({"action": "list_tools"})
print(result)

# 2. Tìm kiếm kho lưu trữ
print("\n🔍 Search repositories:")
result = github_tool.run({
    "action": "call_tool",
    "tool_name": "search_repositories",
    "arguments": {
        "query": "AI agents language:python",
        "page": 1,
        "perPage": 3
    }
})
print(result)

```

### 10.2.3 Giải thích các Phương thức Truyền tải của MCP

Một tính năng quan trọng của giao thức MCP là **tính bất khả tri về truyền tải (transport agnosticism)**. Điều này có nghĩa là bản thân giao thức MCP không phụ thuộc vào phương thức truyền tải cụ thể và có thể chạy trên các kênh giao tiếp khác nhau. HelloAgents, dựa trên FastMCP 2.0, cung cấp hỗ trợ đầy đủ các phương thức truyền tải, cho phép bạn chọn chế độ truyền tải phù hợp nhất dựa trên tình huống thực tế.

**(1) Tổng quan các Phương thức Truyền tải**

`MCPClient` của HelloAgents hỗ trợ năm phương thức truyền tải, mỗi phương thức có trường hợp sử dụng khác nhau, như minh họa trong Bảng 10.4:

<div align="center">
  <p>Bảng 10.4 So sánh các Phương thức Truyền tải của MCP</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-4.png" alt="" width="85%"/>
</div>

**(2) Ví dụ Sử dụng các Phương thức Truyền tải**

```python
from hello_agents.tools import MCPTool

# 1. Memory Transport - Truyền tải trong bộ nhớ (dùng để kiểm thử)
# Không chỉ định tham số, dùng server demo tích hợp sẵn
mcp_tool = MCPTool()

# 2. Stdio Transport - Truyền tải standard input/output (phát triển cục bộ)
# Dùng danh sách lệnh để khởi động server cục bộ
mcp_tool = MCPTool(server_command=["python", "examples/mcp_example_server.py"])

# 3. Stdio Transport with Args - Truyền tải bằng lệnh kèm tham số
# Có thể truyền thêm tham số
mcp_tool = MCPTool(server_command=["python", "examples/mcp_example_server.py", "--debug"])

# 4. Stdio Transport - Server cộng đồng (cách dùng npx)
# Dùng npx để khởi động MCP server của cộng đồng
mcp_tool = MCPTool(server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

# 5. HTTP/SSE/StreamableHTTP Transport
# Lưu ý: MCPTool chủ yếu dành cho truyền tải Stdio và Memory
# Với các phương thức truyền tải từ xa như HTTP/SSE, khuyến nghị dùng trực tiếp MCPClient
```

**(3) Memory Transport (Truyền tải trong Bộ nhớ)**

Trường hợp sử dụng: Unit test, tạo prototype nhanh

```python
from hello_agents.tools import MCPTool

# Dùng server demo tích hợp sẵn (Memory transport)
mcp_tool = MCPTool()

# Liệt kê các công cụ khả dụng
result = mcp_tool.run({"action": "list_tools"})
print(result)

# Gọi công cụ
result = mcp_tool.run({
    "action": "call_tool",
    "tool_name": "add",
    "arguments": {"a": 10, "b": 20}
})
print(result)
```

**(4) Stdio Transport - Truyền tải Standard Input/Output**

Trường hợp sử dụng: Phát triển cục bộ, gỡ lỗi, server bằng script Python

```python
from hello_agents.tools import MCPTool

# Cách 1: Dùng server Python tùy chỉnh
mcp_tool = MCPTool(server_command=["python", "my_mcp_server.py"])

# Cách 2: Dùng server cộng đồng (hệ thống tập tin)
mcp_tool = MCPTool(server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

# Liệt kê công cụ
result = mcp_tool.run({"action": "list_tools"})
print(result)

# Gọi công cụ
result = mcp_tool.run({
    "action": "call_tool",
    "tool_name": "read_file",
    "arguments": {"path": "README.md"}
})
print(result)
```

**(5) HTTP Transport (Truyền tải HTTP)**

Trường hợp sử dụng: Môi trường production, dịch vụ từ xa, kiến trúc microservice

```python
# Lưu ý: MCPTool chủ yếu dành cho truyền tải Stdio và Memory
# Với các phương thức truyền tải từ xa như HTTP/SSE, khuyến nghị dùng MCPClient tầng dưới

import asyncio
from hello_agents.protocols import MCPClient

async def test_http_transport():
    # Kết nối tới HTTP MCP server từ xa
    client = MCPClient("http://api.example.com/mcp")

    async with client:
        # Lấy thông tin server
        tools = await client.list_tools()
        print(f"Remote server tools: {len(tools)} tools")

        # Gọi công cụ từ xa
        result = await client.call_tool("process_data", {
            "data": "Hello, World!",
            "operation": "uppercase"
        })
        print(f"Remote processing result: {result}")

# Lưu ý: Cần một HTTP MCP server thực tế
# asyncio.run(test_http_transport())
```

**(6) SSE Transport - Truyền tải Server-Sent Events**

Trường hợp sử dụng: Giao tiếp thời gian thực, xử lý dạng luồng (streaming), kết nối lâu dài

```python
# Lưu ý: MCPTool chủ yếu dành cho truyền tải Stdio và Memory
# Với truyền tải SSE, khuyến nghị dùng MCPClient tầng dưới

import asyncio
from hello_agents.protocols import MCPClient

async def test_sse_transport():
    # Kết nối tới SSE MCP server
    client = MCPClient(
        "http://localhost:8080/sse",
        transport_type="sse"
    )

    async with client:
        # SSE đặc biệt phù hợp cho xử lý dạng luồng
        result = await client.call_tool("stream_process", {
            "input": "Large data processing request",
            "stream": True
        })
        print(f"Streaming processing result: {result}")

# Lưu ý: Cần MCP server hỗ trợ SSE
# asyncio.run(test_sse_transport())
```

**(7) StreamableHTTP Transport - Truyền tải HTTP dạng Luồng**

Trường hợp sử dụng: Các kịch bản HTTP đòi hỏi giao tiếp luồng hai chiều

```python
# Lưu ý: MCPTool chủ yếu dành cho truyền tải Stdio và Memory
# Với truyền tải StreamableHTTP, khuyến nghị dùng MCPClient tầng dưới

import asyncio
from hello_agents.protocols import MCPClient

async def test_streamable_http_transport():
    # Kết nối tới StreamableHTTP MCP server
    client = MCPClient(
        "http://localhost:8080/mcp",
        transport_type="streamable_http"
    )

    async with client:
        # Hỗ trợ giao tiếp luồng hai chiều
        tools = await client.list_tools()
        print(f"StreamableHTTP server tools: {len(tools)} tools")

# Lưu ý: Cần MCP server hỗ trợ StreamableHTTP
# asyncio.run(test_streamable_http_transport())
```

### 10.2.4 Sử dụng Công cụ MCP trong Agent

Trước đây, chúng ta đã học cách dùng trực tiếp MCP client. Nhưng trong ứng dụng thực tế, chúng ta thường thích để agent **tự động** gọi các công cụ MCP thay vì viết mã gọi thủ công. HelloAgents cung cấp lớp bao bọc `MCPTool`, cho phép MCP server tích hợp liền mạch vào chuỗi công cụ của agent.

**(1) Cơ chế Mở rộng Tự động của Công cụ MCP**

`MCPTool` của HelloAgents có một tính năng: **mở rộng tự động (automatic expansion)**. Khi bạn thêm một công cụ MCP vào Agent, nó sẽ tự động mở rộng tất cả các công cụ do MCP server cung cấp thành các công cụ độc lập, cho phép Agent gọi chúng như các công cụ thông thường.

**Cách 1: Dùng Server Demo Tích hợp Sẵn**

Trước đây chúng ta đã hiện thực các hàm công cụ máy tính, và ở đây chúng ta chuyển đổi chúng thành các dịch vụ MCP. Đây là cách sử dụng đơn giản nhất.

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool

agent = SimpleAgent(name="Assistant", llm=HelloAgentsLLM())

# Không cần cấu hình, tự động dùng server demo tích hợp sẵn
mcp_tool = MCPTool(name="calculator")
agent.add_tool(mcp_tool)
# ✅ Công cụ MCP 'calculator' được mở rộng thành 6 công cụ độc lập

# Agent có thể trực tiếp dùng các công cụ đã mở rộng
response = agent.run("Calculate 25 times 16")
print(response)  # Output: The result of 25 times 16 is 400
```

**Các công cụ sau khi mở rộng tự động**:

- `calculator_add` - Máy tính phép cộng
- `calculator_subtract` - Máy tính phép trừ
- `calculator_multiply` - Máy tính phép nhân
- `calculator_divide` - Máy tính phép chia
- `calculator_greet` - Lời chào thân thiện
- `calculator_get_system_info` - Lấy thông tin hệ thống

Khi Agent gọi, nó chỉ cần cung cấp tham số, ví dụ: `[TOOL_CALL:calculator_multiply:a=25,b=16]`, và hệ thống sẽ tự động xử lý việc chuyển đổi kiểu và gọi MCP.

**Cách 2: Kết nối tới các MCP Server Bên ngoài**

Trong các dự án thực tế, bạn cần kết nối tới các MCP server mạnh mẽ hơn. Các server này có thể là:
- **Server chính thức do cộng đồng cung cấp** (như hệ thống tập tin, GitHub, cơ sở dữ liệu, v.v.)
- **Server tùy chỉnh do bạn tự viết** (đóng gói logic nghiệp vụ)

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool

agent = SimpleAgent(name="File Assistant", llm=HelloAgentsLLM())

# Ví dụ 1: Kết nối tới server hệ thống tập tin do cộng đồng cung cấp
fs_tool = MCPTool(
    name="filesystem",  # Chỉ định tên duy nhất
    description="Access local file system",
    server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."]
)
agent.add_tool(fs_tool)

# Ví dụ 2: Kết nối tới MCP server Python tùy chỉnh
# Về cách viết MCP server tùy chỉnh, xem Phần 10.5
custom_tool = MCPTool(
    name="custom_server",  # Dùng tên khác
    description="Custom business logic server",
    server_command=["python", "my_mcp_server.py"]
)
agent.add_tool(custom_tool)

# Agent giờ đây có thể tự động dùng các công cụ này!
response = agent.run("Please read the my_README.md file and summarize its main content")
print(response)
```

Khi dùng nhiều MCP server, hãy chắc chắn chỉ định một tên khác nhau cho mỗi MCPTool. Tên này sẽ được thêm làm tiền tố vào tên các công cụ đã mở rộng để tránh xung đột. Ví dụ: `name="fs"` sẽ mở rộng thành `fs_read_file`, `fs_write_file`, v.v. Nếu bạn cần tự viết MCP server để đóng gói logic nghiệp vụ cụ thể, xem Phần 10.5.

**(2) Cơ chế Mở rộng Tự động của Công cụ MCP Hoạt động Như thế nào**

Việc hiểu cơ chế mở rộng tự động giúp bạn dùng công cụ MCP tốt hơn. Hãy đi sâu vào cách nó hoạt động:

```python
# Mã của người dùng
fs_tool = MCPTool(name="fs", server_command=[...])
agent.add_tool(fs_tool)

# Điều gì xảy ra bên trong:
# 1. MCPTool kết nối tới server, khám phá 14 công cụ
# 2. Tạo lớp bao bọc cho mỗi công cụ:
#    - fs_read_text_file (tham số: path, tail, head)
#    - fs_write_file (tham số: path, content)
#    - ...
# 3. Đăng ký vào registry công cụ của Agent

# Agent gọi
response = agent.run("Read README.md")

# Bên trong Agent:
# 1. Nhận ra cần gọi fs_read_text_file
# 2. Tạo tham số: path=README.md
# 3. Lớp bao bọc chuyển đổi sang định dạng MCP:
#    {"action": "call_tool", "tool_name": "read_text_file", "arguments": {"path": "README.md"}}
# 4. Gọi MCP server
# 5. Trả về nội dung tập tin
```

Hệ thống tự động chuyển đổi kiểu dựa trên định nghĩa tham số công cụ:

```python
# Agent gọi calculator
agent.run("Calculate 25 times 16")

# Agent tạo ra: a=25,b=16 (chuỗi)
# Hệ thống tự động chuyển đổi thành: {"a": 25.0, "b": 16.0} (số)
# MCP server nhận đúng kiểu số
```

**(3) Trường hợp Thực tế: Trợ lý Tài liệu Thông minh**

Hãy xây dựng một trợ lý tài liệu thông minh hoàn chỉnh. Ở đây chúng ta minh họa bằng một dàn xếp đa agent đơn giản:

```python
"""
Trợ lý Tài liệu Thông minh Cộng tác Đa Agent

Dùng hai SimpleAgent để phân chia công việc:
- Agent1: Chuyên gia tìm kiếm GitHub
- Agent2: Chuyên gia tạo tài liệu
"""
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool
from dotenv import load_dotenv

# Nạp biến môi trường từ tập tin .env
load_dotenv(dotenv_path="../HelloAgents/.env")

print("="*70)
print("Multi-Agent Collaborative Intelligent Document Assistant")
print("="*70)

# ============================================================
# Agent 1: Chuyên gia Tìm kiếm GitHub
# ============================================================
print("\n[Step 1] Creating GitHub search expert...")

github_searcher = SimpleAgent(
    name="GitHub Search Expert",
    llm=HelloAgentsLLM(),
    system_prompt="""You are a GitHub search expert.
Your task is to search GitHub repositories and return results.
Please return clear, structured search results, including:
- Repository name
- Brief description

Keep it concise, don't add extra explanations."""
)

# Thêm công cụ GitHub
github_tool = MCPTool(
    name="gh",
    server_command=["npx", "-y", "@modelcontextprotocol/server-github"]
)
github_searcher.add_tool(github_tool)

# ============================================================
# Agent 2: Chuyên gia Tạo Tài liệu
# ============================================================
print("\n[Step 2] Creating document generation expert...")

document_writer = SimpleAgent(
    name="Document Generation Expert",
    llm=HelloAgentsLLM(),
    system_prompt="""You are a document generation expert.
Your task is to generate structured Markdown reports based on provided information.

The report should include:
- Title
- Introduction
- Main content (listed in points, including project names, descriptions, etc.)
- Summary

Please output the complete Markdown format report content directly, do not use tools to save."""
)

# Thêm công cụ hệ thống tập tin
fs_tool = MCPTool(
    name="fs",
    server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."]
)
document_writer.add_tool(fs_tool)

# ============================================================
# Thực thi Nhiệm vụ
# ============================================================
print("\n" + "="*70)
print("Starting task execution...")
print("="*70)

try:
    # Bước 1: Tìm kiếm GitHub
    print("\n[Step 3] Agent1 searching GitHub...")
    search_task = "Search for GitHub repositories about 'AI agent', return the top 5 most relevant results"

    search_results = github_searcher.run(search_task)

    print("\nSearch results:")
    print("-" * 70)
    print(search_results)
    print("-" * 70)

    # Bước 2: Tạo báo cáo
    print("\n[Step 4] Agent2 generating report...")
    report_task = f"""
Based on the following GitHub search results, generate a Markdown format research report:

{search_results}

Report requirements:
1. Title: # AI Agent Framework Research Report
2. Introduction: Explain this is a GitHub project survey about AI Agents
3. Main findings: List found projects and their features (including names, descriptions, etc.)
4. Summary: Summarize common characteristics of these projects

Please output the complete Markdown format report directly.
"""

    report_content = document_writer.run(report_task)

    print("\nReport content:")
    print("=" * 70)
    print(report_content)
    print("=" * 70)

    # Bước 3: Lưu báo cáo
    print("\n[Step 5] Saving report to file...")
    import os
    try:
        with open("report.md", "w", encoding="utf-8") as f:
            f.write(report_content)
        print("✅ Report saved to report.md")

        # Xác minh tập tin
        file_size = os.path.getsize("report.md")
        print(f"✅ File size: {file_size} bytes")
    except Exception as e:
        print(f"❌ Save failed: {e}")

    print("\n" + "="*70)
    print("Task completed!")
    print("="*70)

except Exception as e:
    print(f"\n❌ Error: {e}")
    import traceback
    traceback.print_exc()

```

Trong quá trình này, `github_searcher` sẽ gọi `gh_search_repositories` để tìm kiếm các dự án GitHub. Kết quả thu được sẽ được trả về cho `document_writer` làm đầu vào, tiếp tục hướng dẫn việc tạo báo cáo, và cuối cùng lưu báo cáo vào report.md.

### 10.2.5 Hệ sinh thái Cộng đồng MCP

Một ưu điểm to lớn của giao thức MCP là **hệ sinh thái cộng đồng phong phú**. Anthropic và các nhà phát triển cộng đồng đã tạo ra một lượng lớn MCP server có sẵn, bao phủ nhiều kịch bản khác nhau như hệ thống tập tin, cơ sở dữ liệu, dịch vụ API, v.v. Điều này có nghĩa là bạn không cần viết adapter công cụ từ đầu và có thể dùng trực tiếp các server đã được kiểm chứng này.

Dưới đây là ba kho tài nguyên cho cộng đồng MCP:

1. **Awesome MCP Servers** (https://github.com/punkpeye/awesome-mcp-servers)
   - Danh sách tuyển chọn các MCP server do cộng đồng duy trì
   - Chứa nhiều server bên thứ ba
   - Phân loại theo chức năng, dễ tìm

2. **MCP Servers Website** (https://mcpservers.org/)
   - Trang web thư mục MCP server chính thức
   - Cung cấp chức năng tìm kiếm và lọc
   - Chứa hướng dẫn sử dụng và ví dụ

3. **Official MCP Servers** (https://github.com/modelcontextprotocol/servers)
   - Server do Anthropic duy trì chính thức
   - Chất lượng cao nhất, tài liệu đầy đủ nhất
   - Chứa hiện thực của các dịch vụ thường dùng

Bảng 10.5 và 10.6 trình bày các MCP server chính thức thường dùng và các MCP server cộng đồng phổ biến:

<div align="center">
  <p>Bảng 10.5 Các MCP Server Chính thức Thường dùng</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-5.png" alt="" width="85%"/>
</div>

<div align="center">
  <p>Bảng 10.6 Các MCP Server Cộng đồng Phổ biến</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-6.png" alt="" width="85%"/>
</div>

Dưới đây là một số trường hợp TODO đặc biệt thú vị để tham khảo:

1. **Kiểm thử Web Tự động (Playwright)**

   ```python
   # Agent có thể tự động:
   # - Mở trình duyệt để truy cập website
   # - Điền và gửi form
   # - Chụp ảnh màn hình để xác minh kết quả
   # - Tạo báo cáo kiểm thử
   playwright_tool = MCPTool(
       name="playwright",
       server_command=["npx", "-y", "@playwright/mcp"]
   )
   ```

2. **Trợ lý Ghi chú Thông minh (Obsidian + Perplexity)**
   ```python
   # Agent có thể:
   # - Tìm kiếm tin tức công nghệ mới nhất (Perplexity)
   # - Sắp xếp thành ghi chú có cấu trúc
   # - Lưu vào kho tri thức Obsidian
   # - Tự động thiết lập liên kết giữa các ghi chú
   ```

3. **Tự động hóa Quản lý Dự án (Jira + GitHub)**
   ```python
   # Agent có thể:
   # - Tạo task Jira từ GitHub Issues
   # - Đồng bộ các commit mã lên Jira
   # - Tự động cập nhật tiến độ Sprint
   # - Tạo báo cáo dự án
   ```

5. **Quy trình Sáng tạo Nội dung (YouTube + Notion + Spotify)**

   ```python
   # Agent có thể:
   # - Lấy phụ đề video YouTube
   # - Tạo bản tóm tắt nội dung
   # - Lưu vào cơ sở dữ liệu Notion
   # - Phát nhạc nền (Spotify)
   ```

Qua phần giải thích của mục này, hy vọng bạn có thể khám phá thêm nhiều trường hợp hiện thực MCP, và những đóng góp cho HelloAgents đều được hoan nghênh! Tiếp theo, hãy tìm hiểu về giao thức A2A.

## 10.3 Thực hành Giao thức A2A

A2A (Agent-to-Agent) là một giao thức hỗ trợ giao tiếp và cộng tác trực tiếp giữa các agent.

### 10.3.1 Động lực Thiết kế Giao thức

Giao thức MCP đã giải quyết sự tương tác giữa agent và công cụ, trong khi giao thức A2A giải quyết bài toán cộng tác giữa các agent. Trong một nhiệm vụ đòi hỏi nhiều agent (như researcher, writer, editor) cộng tác, chúng cần giao tiếp, ủy thác nhiệm vụ, thương lượng khả năng, và đồng bộ trạng thái.

Các giải pháp bộ điều phối tập trung truyền thống (cấu trúc hình sao - star topology) có ba vấn đề chính:

- **Điểm hỏng đơn (Single Point of Failure)**: Bộ điều phối hỏng dẫn đến toàn bộ hệ thống tê liệt.
- **Nút thắt Hiệu năng**: Mọi giao tiếp đều đi qua nút trung tâm, hạn chế khả năng xử lý đồng thời.
- **Khó Mở rộng**: Thêm hoặc sửa đổi agent đòi hỏi thay đổi logic trung tâm.

Giao thức A2A áp dụng kiến trúc ngang hàng (P2P) (cấu trúc lưới - mesh topology), cho phép các agent giao tiếp trực tiếp, giải quyết tận gốc các vấn đề trên. Cốt lõi của nó là hai khái niệm trừu tượng **Task** và **Artifact**, đây là điểm khác biệt lớn nhất của nó so với MCP, như minh họa trong Bảng 10.7.

<div align="center">
  <p>Bảng 10.7 Các Khái niệm Cốt lõi của A2A</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-7.png" alt="" width="85%"/>
</div>

Để hiện thực việc quản lý quá trình cộng tác, A2A định nghĩa một vòng đời chuẩn hóa cho các nhiệm vụ, bao gồm các trạng thái như tạo mới, thương lượng, ủy thác, đang thực hiện, hoàn thành, và thất bại, như minh họa trong Hình 10.7.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-7.png" alt="" width="85%"/>
  <p>Hình 10.7 Vòng đời Nhiệm vụ của A2A</p>
</div>


Cơ chế này giúp các agent thực hiện thương lượng nhiệm vụ, theo dõi tiến độ, và xử lý ngoại lệ.

Vòng đời yêu cầu của A2A là một chuỗi trình bày chi tiết bốn bước chính mà một yêu cầu tuân theo: khám phá agent, xác thực, API gửi tin nhắn, và API luồng gửi tin nhắn. Hình 10.8 dưới đây, mượn từ sơ đồ luồng của trang web chính thức, cho thấy luồng vận hành, minh họa sự tương tác giữa client, A2A server, và authentication server.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-8.png" alt="" width="85%"/>
  <p>Hình 10.8 Vòng đời Yêu cầu của A2A</p>
</div>

### 10.3.2 Thực hành Giao thức A2A

Hầu hết các hiện thực A2A hiện có đều là `Sample Code`, và ngay cả các hiện thực Python cũng khá rườm rà. Vì vậy, ở đây chúng ta chỉ áp dụng một phương pháp mô phỏng các ý tưởng của giao thức, hiện thực một phần chức năng thông qua A2A-SDK.

**(2) Tạo một A2A Agent Đơn giản**

Hãy tạo một A2A agent, một lần nữa dùng trường hợp máy tính để minh họa:

```python
from hello_agents.protocols.a2a.implementation import A2AServer, A2A_AVAILABLE

def create_calculator_agent():
    """Tạo một agent máy tính"""
    if not A2A_AVAILABLE:
        print("❌ A2A SDK not installed, please run: pip install a2a-sdk")
        return None

    print("🧮 Creating calculator agent")

    # Tạo A2A server
    calculator = A2AServer(
        name="calculator-agent",
        description="Professional mathematical calculation agent",
        version="1.0.0",
        capabilities={
            "math": ["addition", "subtraction", "multiplication", "division"],
            "advanced": ["power", "sqrt", "factorial"]
        }
    )

    # Thêm các kỹ năng tính toán cơ bản
    @calculator.skill("add")
    def add_numbers(query: str) -> str:
        """Tính toán phép cộng"""
        try:
            # Phân tích đơn giản định dạng "calculate 5 + 3"
            parts = query.replace("calculate", "").replace("plus", "+").replace("add", "+")
            if "+" in parts:
                numbers = [float(x.strip()) for x in parts.split("+")]
                result = sum(numbers)
                return f"Calculation result: {' + '.join(map(str, numbers))} = {result}"
            else:
                return "Please use format: calculate 5 + 3"
        except Exception as e:
            return f"Calculation error: {e}"

    @calculator.skill("multiply")
    def multiply_numbers(query: str) -> str:
        """Tính toán phép nhân"""
        try:
            parts = query.replace("calculate", "").replace("times", "*").replace("×", "*")
            if "*" in parts:
                numbers = [float(x.strip()) for x in parts.split("*")]
                result = 1
                for num in numbers:
                    result *= num
                return f"Calculation result: {' × '.join(map(str, numbers))} = {result}"
            else:
                return "Please use format: calculate 5 * 3"
        except Exception as e:
            return f"Calculation error: {e}"

    @calculator.skill("info")
    def get_info(query: str) -> str:
        """Lấy thông tin agent"""
        return f"I am {calculator.name}, can perform basic mathematical calculations. Supported skills: {list(calculator.skills.keys())}"

    print(f"✅ Calculator agent created successfully, supported skills: {list(calculator.skills.keys())}")
    return calculator

# Tạo agent
calc_agent = create_calculator_agent()
if calc_agent:
    # Kiểm thử các kỹ năng
    print("\n🧪 Testing agent skills:")
    test_queries = [
        "Get information",
        "Calculate 10 + 5",
        "Calculate 6 * 7"
    ]

    for query in test_queries:
        if "information" in query.lower():
            result = calc_agent.skills["info"](query)
        elif "+" in query:
            result = calc_agent.skills["add"](query)
        elif "*" in query or "×" in query:
            result = calc_agent.skills["multiply"](query)
        else:
            result = "Unknown query type"

        print(f"  📝 Query: {query}")
        print(f"  🤖 Reply: {result}")
        print()
```

**(2) A2A Agent Tùy chỉnh**

Bạn cũng có thể tạo A2A agent của riêng mình, dưới đây là một minh họa đơn giản:

```python
from hello_agents.protocols.a2a.implementation import A2AServer, A2A_AVAILABLE

def create_custom_agent():
    """Tạo agent tùy chỉnh"""
    if not A2A_AVAILABLE:
        print("Please install A2A SDK first: pip install a2a-sdk")
        return None

    # Tạo agent
    agent = A2AServer(
        name="my-custom-agent",
        description="My custom agent",
        capabilities={"custom": ["skill1", "skill2"]}
    )

    # Thêm kỹ năng
    @agent.skill("greet")
    def greet_user(name: str) -> str:
        """Chào người dùng"""
        return f"Hello, {name}! I am a custom agent."

    @agent.skill("calculate")
    def simple_calculate(expression: str) -> str:
        """Tính toán đơn giản"""
        try:
            # Tính toán an toàn (chỉ hỗ trợ các phép toán cơ bản)
            allowed_chars = set('0123456789+-*/(). ')
            if all(c in allowed_chars for c in expression):
                result = eval(expression)
                return f"Calculation result: {expression} = {result}"
            else:
                return "Error: Only basic mathematical operations supported"
        except Exception as e:
            return f"Calculation error: {e}"

    return agent

# Tạo và kiểm thử agent tùy chỉnh
custom_agent = create_custom_agent()
if custom_agent:
    # Kiểm thử các kỹ năng
    print("Testing greeting skill:")
    result1 = custom_agent.skills["greet"]("Zhang San")
    print(result1)

    print("\nTesting calculation skill:")
    result2 = custom_agent.skills["calculate"]("10 + 5 * 2")
    print(result2)
```

### 10.3.3 Sử dụng Công cụ A2A của HelloAgents

HelloAgents cung cấp một giao diện công cụ A2A thống nhất.

**(1) Tạo A2A Agent Server**

Trước tiên, hãy tạo một Agent server:

```python
from hello_agents.protocols import A2AServer
import threading
import time

# Tạo dịch vụ Agent researcher
researcher = A2AServer(
    name="researcher",
    description="Agent responsible for searching and analyzing materials",
    version="1.0.0"
)

# Định nghĩa kỹ năng
@researcher.skill("research")
def handle_research(text: str) -> str:
    """Xử lý các yêu cầu nghiên cứu"""
    import re
    match = re.search(r'research\s+(.+)', text, re.IGNORECASE)
    topic = match.group(1).strip() if match else text

    # Logic nghiên cứu thực tế (được đơn giản hóa ở đây)
    result = {
        "topic": topic,
        "findings": f"Research results about {topic}...",
        "sources": ["Source 1", "Source 2", "Source 3"]
    }
    return str(result)

# Khởi động dịch vụ ở nền
def start_server():
    researcher.run(host="localhost", port=5000)

if __name__ == "__main__":
    server_thread = threading.Thread(target=start_server, daemon=True)
    server_thread.start()

    print("✅ Researcher Agent service started at http://localhost:5000")

    # Giữ chương trình chạy
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        print("\nService stopped")
```

**(2) Tạo A2A Agent Client**

Bây giờ, hãy tạo một client để giao tiếp với server:

```python
from hello_agents.protocols import A2AClient

# Tạo client để kết nối tới Agent researcher
client = A2AClient("http://localhost:5000")

# Gửi yêu cầu nghiên cứu
response = client.execute_skill("research", "research AI applications in healthcare")
print(f"Received response: {response.get('result')}")

# Output:
# Received response: {'topic': 'AI applications in healthcare', 'findings': 'Research results about AI applications in healthcare...', 'sources': ['Source 1', 'Source 2', 'Source 3']}
```

**(3) Tạo Mạng lưới Agent**

Đối với sự cộng tác giữa nhiều Agent, chúng ta có thể kết nối nhiều Agent với nhau:

```python
from hello_agents.protocols import A2AServer, A2AClient
import threading
import time

# 1. Tạo nhiều dịch vụ Agent
researcher = A2AServer(
    name="researcher",
    description="Researcher"
)

@researcher.skill("research")
def do_research(text: str) -> str:
    import re
    match = re.search(r'research\s+(.+)', text, re.IGNORECASE)
    topic = match.group(1).strip() if match else text
    return str({"topic": topic, "findings": f"Research results for {topic}"})

writer = A2AServer(
    name="writer",
    description="Writer"
)

@writer.skill("write")
def write_article(text: str) -> str:
    import re
    match = re.search(r'write\s+(.+)', text, re.IGNORECASE)
    content = match.group(1).strip() if match else text

    # Thử phân tích dữ liệu nghiên cứu
    try:
        data = eval(content)
        topic = data.get("topic", "Unknown topic")
        findings = data.get("findings", "No research results")
    except:
        topic = "Unknown topic"
        findings = content

    return f"# {topic}\n\nBased on research: {findings}\n\nArticle content..."

editor = A2AServer(
    name="editor",
    description="Editor"
)

@editor.skill("edit")
def edit_article(text: str) -> str:
    import re
    match = re.search(r'edit\s+(.+)', text, re.IGNORECASE)
    article = match.group(1).strip() if match else text

    result = {
        "article": article + "\n\n[Edited and optimized]",
        "feedback": "Article quality is good",
        "approved": True
    }
    return str(result)

# 2. Khởi động tất cả dịch vụ
threading.Thread(target=lambda: researcher.run(port=5000), daemon=True).start()
threading.Thread(target=lambda: writer.run(port=5001), daemon=True).start()
threading.Thread(target=lambda: editor.run(port=5002), daemon=True).start()
time.sleep(2)  # Chờ các dịch vụ khởi động

# 3. Tạo các client để kết nối tới từng Agent
researcher_client = A2AClient("http://localhost:5000")
writer_client = A2AClient("http://localhost:5001")
editor_client = A2AClient("http://localhost:5002")

# 4. Quy trình cộng tác
def create_content(topic):
    # Bước 1: Nghiên cứu
    research = researcher_client.execute_skill("research", f"research {topic}")
    research_data = research.get('result', '')

    # Bước 2: Viết
    article = writer_client.execute_skill("write", f"write {research_data}")
    article_content = article.get('result', '')

    # Bước 3: Biên tập
    final = editor_client.execute_skill("edit", f"edit {article_content}")
    return final.get('result', '')

# Cách dùng
result = create_content("AI applications in healthcare")
print(f"\nFinal result:\n{result}")
```

### 10.3.4 Sử dụng Công cụ A2A trong Agent

Bây giờ hãy xem cách tích hợp A2A vào các agent của HelloAgents.

**(1) Sử dụng Lớp bao bọc A2ATool**

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import A2ATool
from dotenv import load_dotenv

load_dotenv()
llm = HelloAgentsLLM()

# Giả sử một dịch vụ Agent researcher đã đang chạy tại http://localhost:5000

# Tạo Agent điều phối
coordinator = SimpleAgent(name="Coordinator", llm=llm)

# Thêm công cụ A2A, kết nối tới Agent researcher
researcher_tool = A2ATool(
    name="researcher",
    description="Researcher Agent, can search and analyze materials",
    agent_url="http://localhost:5000"
)
coordinator.add_tool(researcher_tool)

# Coordinator có thể gọi Agent researcher
response = coordinator.run("Please have the researcher help me research AI applications in education")
print(response)
```

**(2) Trường hợp Thực tế: Hệ thống Chăm sóc Khách hàng Thông minh**

Hãy xây dựng một hệ thống chăm sóc khách hàng thông minh hoàn chỉnh với ba Agent:
- **Receptionist (Lễ tân)**: Phân tích loại câu hỏi của khách hàng
- **Technical Expert (Chuyên gia kỹ thuật)**: Trả lời các câu hỏi kỹ thuật
- **Sales Consultant (Tư vấn bán hàng)**: Trả lời các câu hỏi về bán hàng

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import A2ATool
from hello_agents.protocols import A2AServer
import threading
import time
from dotenv import load_dotenv

load_dotenv()
llm = HelloAgentsLLM()

# 1. Tạo dịch vụ Agent chuyên gia kỹ thuật
tech_expert = A2AServer(
    name="tech_expert",
    description="Technical expert, answers technical questions"
)

@tech_expert.skill("answer")
def answer_tech_question(text: str) -> str:
    import re
    match = re.search(r'answer\s+(.+)', text, re.IGNORECASE)
    question = match.group(1).strip() if match else text
    # Trong ứng dụng thực tế, đây sẽ gọi LLM hoặc cơ sở tri thức
    return f"Technical answer: Regarding '{question}', I suggest you check our technical documentation..."

# 2. Tạo dịch vụ Agent tư vấn bán hàng
sales_advisor = A2AServer(
    name="sales_advisor",
    description="Sales consultant, answers sales questions"
)

@sales_advisor.skill("answer")
def answer_sales_question(text: str) -> str:
    import re
    match = re.search(r'answer\s+(.+)', text, re.IGNORECASE)
    question = match.group(1).strip() if match else text
    return f"Sales answer: Regarding '{question}', we have special offers..."

# 3. Khởi động các dịch vụ
threading.Thread(target=lambda: tech_expert.run(port=6000), daemon=True).start()
threading.Thread(target=lambda: sales_advisor.run(port=6001), daemon=True).start()
time.sleep(2)

# 4. Tạo Agent lễ tân (dùng SimpleAgent của HelloAgents)
receptionist = SimpleAgent(
    name="Receptionist",
    llm=llm,
    system_prompt="""You are a customer service receptionist, responsible for:
1. Analyzing customer question types (technical questions or sales questions)
2. Forwarding questions to appropriate experts
3. Organizing expert answers and returning them to customers

Please remain polite and professional."""
)

# Thêm công cụ chuyên gia kỹ thuật
tech_tool = A2ATool(
    agent_url="http://localhost:6000",
    name="tech_expert",
    description="Technical expert, answers technical-related questions"
)
receptionist.add_tool(tech_tool)

# Thêm công cụ tư vấn bán hàng
sales_tool = A2ATool(
    agent_url="http://localhost:6001",
    name="sales_advisor",
    description="Sales consultant, answers price and purchase-related questions"
)
receptionist.add_tool(sales_tool)

# 5. Xử lý các yêu cầu của khách hàng
def handle_customer_query(query):
    print(f"\nCustomer inquiry: {query}")
    print("=" * 50)
    response = receptionist.run(query)
    print(f"\nCustomer service reply: {response}")
    print("=" * 50)

# Kiểm thử các loại câu hỏi khác nhau
if __name__ == "__main__":
    handle_customer_query("How do I call your API?")
    handle_customer_query("What is the price of the enterprise version?")
    handle_customer_query("How do I integrate it into my Python project?")
```

**(3) Cách dùng Nâng cao: Thương lượng giữa các Agent**

Giao thức A2A cũng hỗ trợ cơ chế thương lượng giữa các Agent:

```python
from hello_agents.protocols import A2AServer, A2AClient
import threading
import time

# Tạo hai Agent cần thương lượng
agent1 = A2AServer(
    name="agent1",
    description="Agent 1"
)

@agent1.skill("propose")
def handle_proposal(text: str) -> str:
    """Xử lý các đề xuất thương lượng"""
    import re

    # Phân tích đề xuất
    match = re.search(r'propose\s+(.+)', text, re.IGNORECASE)
    proposal_str = match.group(1).strip() if match else text

    try:
        proposal = eval(proposal_str)
        task = proposal.get("task")
        deadline = proposal.get("deadline")

        # Đánh giá đề xuất
        if deadline >= 7:  # Cần ít nhất 7 ngày
            result = {"accepted": True, "message": "Proposal accepted"}
        else:
            result = {
                "accepted": False,
                "message": "Timeline too tight",
                "counter_proposal": {"deadline": 7}
            }
        return str(result)
    except:
        return str({"accepted": False, "message": "Invalid proposal format"})

agent2 = A2AServer(
    name="agent2",
    description="Agent 2"
)

@agent2.skill("negotiate")
def negotiate_task(text: str) -> str:
    """Khởi tạo thương lượng"""
    import re

    # Phân tích nhiệm vụ và thời hạn
    match = re.search(r'negotiate\s+task:(.+?)\s+deadline:(\d+)', text, re.IGNORECASE)
    if match:
        task = match.group(1).strip()
        deadline = int(match.group(2))

        # Gửi đề xuất tới agent1
        proposal = {"task": task, "deadline": deadline}
        return str({"status": "negotiating", "proposal": proposal})
    else:
        return str({"status": "error", "message": "Invalid negotiation request"})

# Khởi động các dịch vụ
threading.Thread(target=lambda: agent1.run(port=7000), daemon=True).start()
threading.Thread(target=lambda: agent2.run(port=7001), daemon=True).start()
```

## 10.4 Thực hành Giao thức ANP

Sau khi giao thức MCP giải quyết việc gọi công cụ và giao thức A2A giải quyết sự cộng tác ngang hàng giữa các agent, giao thức ANP tập trung vào giải quyết các bài toán quản lý agent trong môi trường mạng mở, quy mô lớn.

Ở Phần 10.2 và 10.3, chúng ta đã tìm hiểu về MCP (truy cập công cụ) và A2A (cộng tác agent). Bây giờ, hãy tìm hiểu về giao thức ANP (Agent Network Protocol), vốn tập trung vào việc xây dựng **các mạng lưới agent mở, quy mô lớn**.

### 10.4.1 Mục tiêu của Giao thức

Khi một mạng chứa một lượng lớn agent với các chức năng khác nhau (ví dụ: xử lý ngôn ngữ tự nhiên, nhận dạng hình ảnh, phân tích dữ liệu, v.v.), hệ thống phải đối mặt với một loạt thách thức:

- **Khám phá Dịch vụ (Service Discovery)**: Khi một nhiệm vụ mới đến, làm thế nào để nhanh chóng tìm được agent có khả năng xử lý nhiệm vụ đó?
- **Định tuyến Thông minh (Intelligent Routing)**: Nếu nhiều agent có thể xử lý cùng một nhiệm vụ, làm thế nào để chọn cái phù hợp nhất (ví dụ: dựa trên tải, chi phí, v.v.) và điều phối nhiệm vụ tới nó?
- **Mở rộng Động (Dynamic Scaling)**: Làm thế nào để các agent mới gia nhập có thể được các thành viên khác khám phá và gọi tới?

Mục tiêu thiết kế của ANP là cung cấp một cơ chế chuẩn hóa để giải quyết các bài toán khám phá dịch vụ, lựa chọn định tuyến, và khả năng mở rộng mạng lưới nói trên.

Để đạt được các mục tiêu thiết kế, ANP định nghĩa các khái niệm cốt lõi sau, như minh họa trong Bảng 10.8:

<div align="center">
  <p>Bảng 10.8 Các Khái niệm Cốt lõi của ANP</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-8.png" alt="" width="85%"/>
</div>

Chúng ta cũng mượn từ [Hướng dẫn Bắt đầu](https://github.com/agent-network-protocol/AgentNetworkProtocol/blob/main/docs/chinese/ANP入门指南.md) chính thức để giới thiệu thiết kế kiến trúc của ANP, như minh họa trong Hình 10.9

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-9.png" alt="" width="85%"/>
  <p>Hình 10.9 Quy trình Tổng thể của ANP</p>
</div>


Trong sơ đồ luồng này, các bước chính bao gồm:

**1. Khám phá và Khớp Dịch vụ:** Trước tiên, Agent A dùng một dịch vụ khám phá công khai để truy vấn dựa trên mô tả ngữ nghĩa hoặc chức năng nhằm định vị Agent B đáp ứng yêu cầu nhiệm vụ của nó. Dịch vụ khám phá thiết lập một chỉ mục bằng cách thu thập trước các endpoint chuẩn (`.well-known/agent-descriptions`) mà mỗi agent phơi bày ra, qua đó đạt được sự khớp động giữa bên có nhu cầu dịch vụ và bên cung cấp dịch vụ.

**2. Xác thực Danh tính dựa trên DID:** Khi bắt đầu tương tác, Agent A dùng khóa riêng (private key) của mình để ký một yêu cầu chứa DID của chính nó. Sau khi Agent B nhận được, nó phân giải DID để lấy khóa công khai (public key) tương ứng và dùng nó để xác minh tính xác thực của chữ ký và tính toàn vẹn của yêu cầu, qua đó thiết lập giao tiếp đáng tin cậy giữa hai bên.

**3. Thực thi Dịch vụ Chuẩn hóa:** Sau khi xác thực danh tính thành công, Agent B đáp ứng yêu cầu, và hai bên trao đổi dữ liệu hoặc gọi dịch vụ (như đặt chỗ, truy vấn, v.v.) theo các giao diện chuẩn và định dạng dữ liệu được định nghĩa trước. Các quy trình tương tác chuẩn hóa là nền tảng để đạt được khả năng tương tác xuyên nền tảng và xuyên hệ thống.

Tóm lại, cốt lõi của cơ chế này là dùng DID để xây dựng một nền tảng tin cậy phi tập trung và tận dụng các giao thức mô tả chuẩn hóa để đạt được sự khám phá dịch vụ động. Cách tiếp cận này cho phép các agent hình thành các mạng lưới cộng tác trên internet một cách an toàn và hiệu quả mà không cần điều phối tập trung.

### 10.4.2 Sử dụng Khám phá Dịch vụ ANP

**(1) Tạo Trung tâm Khám phá Dịch vụ**

```python
from hello_agents.protocols import ANPDiscovery, register_service

# Tạo trung tâm khám phá dịch vụ
discovery = ANPDiscovery()

# Đăng ký các dịch vụ Agent
register_service(
    discovery=discovery,
    service_id="nlp_agent_1",
    service_name="NLP Processing Expert A",
    service_type="nlp",
    capabilities=["text_analysis", "sentiment_analysis", "ner"],
    endpoint="http://localhost:8001",
    metadata={"load": 0.3, "price": 0.01, "version": "1.0.0"}
)

register_service(
    discovery=discovery,
    service_id="nlp_agent_2",
    service_name="NLP Processing Expert B",
    service_type="nlp",
    capabilities=["text_analysis", "translation"],
    endpoint="http://localhost:8002",
    metadata={"load": 0.7, "price": 0.02, "version": "1.1.0"}
)

print("✅ Service registration completed")
```

**(2) Khám phá Dịch vụ**

```python
from hello_agents.protocols import discover_service

# Tìm theo kiểu
nlp_services = discover_service(discovery, service_type="nlp")
print(f"Found {len(nlp_services)} NLP services")

# Chọn dịch vụ có tải thấp nhất
best_service = min(nlp_services, key=lambda s: s.metadata.get("load", 1.0))
print(f"Best service: {best_service.service_name} (load: {best_service.metadata['load']})")
```

**(3) Xây dựng Mạng lưới Agent**

```python
from hello_agents.protocols import ANPNetwork

# Tạo mạng
network = ANPNetwork(network_id="ai_cluster")

# Thêm các nút
for service in discovery.list_all_services():
    network.add_node(service.service_id, service.endpoint)

# Thiết lập kết nối (dựa trên việc khớp khả năng)
network.connect_nodes("nlp_agent_1", "nlp_agent_2")

stats = network.get_network_stats()
print(f"✅ Network construction completed, total {stats['total_nodes']} nodes")
```

### 10.4.3 Trường hợp Thực tế

Hãy xây dựng một hệ thống điều phối nhiệm vụ phân tán hoàn chỉnh:

```python
from hello_agents.protocols import ANPDiscovery, register_service
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools.builtin import ANPTool
import random
from dotenv import load_dotenv

load_dotenv()
llm = HelloAgentsLLM()

# 1. Tạo trung tâm khám phá dịch vụ
discovery = ANPDiscovery()

# 2. Đăng ký nhiều nút tính toán
for i in range(10):
    register_service(
        discovery=discovery,
        service_id=f"compute_node_{i}",
        service_name=f"Compute Node {i}",
        service_type="compute",
        capabilities=["data_processing", "ml_training"],
        endpoint=f"http://node{i}:8000",
        metadata={
            "load": random.uniform(0.1, 0.9),
            "cpu_cores": random.choice([4, 8, 16]),
            "memory_gb": random.choice([16, 32, 64]),
            "gpu": random.choice([True, False])
        }
    )

print(f"✅ Registered {len(discovery.list_all_services())} compute nodes")

# 3. Tạo Agent điều phối nhiệm vụ
scheduler = SimpleAgent(
    name="Task Scheduler",
    llm=llm,
    system_prompt="""You are an intelligent task scheduler, responsible for:
1. Analyzing task requirements
2. Selecting the most suitable compute node
3. Assigning tasks

When selecting nodes, consider: load, CPU cores, memory, GPU, and other factors."""
)

# Thêm công cụ ANP
anp_tool = ANPTool(
    name="service_discovery",
    description="Service discovery tool, can find and select compute nodes",
    discovery=discovery
)
scheduler.add_tool(anp_tool)

# 4. Phân công nhiệm vụ thông minh
def assign_task(task_description):
    print(f"\nTask: {task_description}")
    print("=" * 50)

    # Để Agent chọn nút một cách thông minh
    response = scheduler.run(f"""
    Please select the most suitable compute node for the following task:
    {task_description}

    Requirements:
    1. List all available nodes
    2. Analyze characteristics of each node
    3. Select the most suitable node
    4. Explain selection reasoning
    """)

    print(response)
    print("=" * 50)

# Kiểm thử các loại nhiệm vụ khác nhau
assign_task("Train a large deep learning model, requires GPU support")
assign_task("Process large amounts of text data, requires high memory")
assign_task("Run lightweight data analysis task")
```

Đây là một ví dụ cân bằng tải (load balancing)

```python
from hello_agents.protocols import ANPDiscovery, register_service
import random

# Tạo trung tâm khám phá dịch vụ
discovery = ANPDiscovery()

# Đăng ký nhiều dịch vụ cùng kiểu
for i in range(5):
    register_service(
        discovery=discovery,
        service_id=f"api_server_{i}",
        service_name=f"API Server {i}",
        service_type="api",
        capabilities=["rest_api"],
        endpoint=f"http://api{i}:8000",
        metadata={"load": random.uniform(0.1, 0.9)}
    )

# Hàm cân bằng tải
def get_best_server():
    """Chọn server có tải thấp nhất"""
    servers = discovery.discover_services(service_type="api")
    if not servers:
        return None

    best = min(servers, key=lambda s: s.metadata.get("load", 1.0))
    return best

# Mô phỏng phân bổ yêu cầu
for i in range(10):
    server = get_best_server()
    print(f"Request {i+1} -> {server.service_name} (load: {server.metadata['load']:.2f})")

    # Cập nhật tải (mô phỏng)
    server.metadata["load"] += 0.1
```

## 10.5 Xây dựng MCP Server Tùy chỉnh

Ở các phần trước, chúng ta đã học cách dùng các dịch vụ MCP có sẵn. Chúng ta cũng đã tìm hiểu về đặc điểm của các giao thức khác nhau. Bây giờ, hãy học cách xây dựng MCP server của riêng mình.

### 10.5.1 Tạo MCP Server Đầu tiên của Bạn

**(1) Vì sao cần Xây dựng MCP Server Tùy chỉnh?**

Mặc dù bạn có thể dùng trực tiếp các dịch vụ MCP công khai, trong nhiều kịch bản ứng dụng thực tế, bạn cần xây dựng MCP server tùy chỉnh để đáp ứng những nhu cầu cụ thể.

Các động lực chính bao gồm:

- **Đóng gói Logic Nghiệp vụ**: Đóng gói các quy trình nghiệp vụ đặc thù của doanh nghiệp hoặc các thao tác phức tạp thành các công cụ MCP chuẩn hóa để agent gọi tới một cách thống nhất.
- **Truy cập Dữ liệu Riêng tư**: Tạo một giao diện hoặc proxy an toàn và có thể kiểm soát để truy cập các cơ sở dữ liệu nội bộ, API, hoặc các nguồn dữ liệu riêng tư khác không thể phơi bày ra mạng công cộng.
- **Tối ưu Hiệu năng**: Tối ưu sâu cho các lệnh gọi tần suất cao hoặc các kịch bản ứng dụng có yêu cầu nghiêm ngặt về độ trễ phản hồi.
- **Mở rộng Tính năng Tùy chỉnh**: Hiện thực các chức năng cụ thể mà dịch vụ MCP tiêu chuẩn không cung cấp, chẳng hạn như tích hợp các mô hình thuật toán độc quyền hoặc kết nối với các thiết bị phần cứng cụ thể.

**(2) Trường hợp Giảng dạy: MCP Server Truy vấn Thời tiết**

Hãy bắt đầu với một server truy vấn thời tiết đơn giản và dần dần học cách phát triển MCP server:

```python
#!/usr/bin/env python3
"""Weather Query MCP Server"""

import json
import requests
import os
from datetime import datetime
from typing import Dict, Any
from hello_agents.protocols import MCPServer

# Tạo MCP server
weather_server = MCPServer(name="weather-server", description="Real weather query service")

CITY_MAP = {
    "Beijing": "Beijing", "Shanghai": "Shanghai", "Guangzhou": "Guangzhou",
    "Shenzhen": "Shenzhen", "Hangzhou": "Hangzhou", "Chengdu": "Chengdu",
    "Chongqing": "Chongqing", "Wuhan": "Wuhan", "Xi'an": "Xi'an",
    "Nanjing": "Nanjing", "Tianjin": "Tianjin", "Suzhou": "Suzhou"
}


def get_weather_data(city: str) -> Dict[str, Any]:
    """Lấy dữ liệu thời tiết từ wttr.in"""
    city_en = CITY_MAP.get(city, city)
    url = f"https://wttr.in/{city_en}?format=j1"
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    data = response.json()
    current = data["current_condition"][0]

    return {
        "city": city,
        "temperature": float(current["temp_C"]),
        "feels_like": float(current["FeelsLikeC"]),
        "humidity": int(current["humidity"]),
        "condition": current["weatherDesc"][0]["value"],
        "wind_speed": round(float(current["windspeedKmph"]) / 3.6, 1),
        "visibility": float(current["visibility"]),
        "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    }


# Định nghĩa hàm công cụ
def get_weather(city: str) -> str:
    """Lấy thời tiết hiện tại cho thành phố được chỉ định"""
    try:
        weather_data = get_weather_data(city)
        return json.dumps(weather_data, ensure_ascii=False, indent=2)
    except Exception as e:
        return json.dumps({"error": str(e), "city": city}, ensure_ascii=False)


def list_supported_cities() -> str:
    """Liệt kê tất cả các thành phố Trung Quốc được hỗ trợ"""
    result = {"cities": list(CITY_MAP.keys()), "count": len(CITY_MAP)}
    return json.dumps(result, ensure_ascii=False, indent=2)


def get_server_info() -> str:
    """Lấy thông tin server"""
    info = {
        "name": "Weather MCP Server",
        "version": "1.0.0",
        "tools": ["get_weather", "list_supported_cities", "get_server_info"]
    }
    return json.dumps(info, ensure_ascii=False, indent=2)


# Đăng ký các công cụ vào server
weather_server.add_tool(get_weather)
weather_server.add_tool(list_supported_cities)
weather_server.add_tool(get_server_info)


if __name__ == "__main__":
    weather_server.run()
```

**(3) Kiểm thử MCP Server Tùy chỉnh**

Sau đó tạo một script kiểm thử:

```python
#!/usr/bin/env python3
"""Test Weather Query MCP Server"""

import asyncio
import json
import sys
import os

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..', 'HelloAgents'))
from hello_agents.protocols.mcp.client import MCPClient


async def test_weather_server():
    server_script = os.path.join(os.path.dirname(__file__), "14_weather_mcp_server.py")
    client = MCPClient(["python", server_script])

    try:
        async with client:
            # Kiểm thử 1: Lấy thông tin server
            info = json.loads(await client.call_tool("get_server_info", {}))
            print(f"Server: {info['name']} v{info['version']}")

            # Kiểm thử 2: Liệt kê các thành phố được hỗ trợ
            cities = json.loads(await client.call_tool("list_supported_cities", {}))
            print(f"Supported cities: {cities['count']} cities")

            # Kiểm thử 3: Truy vấn thời tiết Bắc Kinh
            weather = json.loads(await client.call_tool("get_weather", {"city": "Beijing"}))
            if "error" not in weather:
                print(f"\nBeijing weather: {weather['temperature']}°C, {weather['condition']}")

            # Kiểm thử 4: Truy vấn thời tiết Thâm Quyến
            weather = json.loads(await client.call_tool("get_weather", {"city": "Shenzhen"}))
            if "error" not in weather:
                print(f"Shenzhen weather: {weather['temperature']}°C, {weather['condition']}")

            print("\n✅ All tests completed!")

    except Exception as e:
        print(f"❌ Test failed: {e}")


if __name__ == "__main__":
    asyncio.run(test_weather_server())
```

**(4) Sử dụng MCP Server Tùy chỉnh trong Agent**

```python
"""Using Weather MCP Server in Agent"""

import os
from dotenv import load_dotenv
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool

load_dotenv()


def create_weather_assistant():
    """Tạo trợ lý thời tiết"""
    llm = HelloAgentsLLM()

    assistant = SimpleAgent(
        name="Weather Assistant",
        llm=llm,
        system_prompt="""You are a weather assistant that can query city weather.
Use the get_weather tool to query weather, supports Chinese city names.
"""
    )

    # Thêm công cụ weather MCP
    server_script = os.path.join(os.path.dirname(__file__), "14_weather_mcp_server.py")
    weather_tool = MCPTool(server_command=["python", server_script])
    assistant.add_tool(weather_tool)

    return assistant


def demo():
    """Demo"""
    assistant = create_weather_assistant()

    print("\nQuery Beijing weather:")
    response = assistant.run("How's the weather in Beijing today?")
    print(f"Answer: {response}\n")


def interactive():
    """Chế độ tương tác"""
    assistant = create_weather_assistant()

    while True:
        user_input = input("\nYou: ").strip()
        if user_input.lower() in ['quit', 'exit']:
            break
        response = assistant.run(user_input)
        print(f"Assistant: {response}")


if __name__ == "__main__":
    import sys
    if len(sys.argv) > 1 and sys.argv[1] == "demo":
        demo()
    else:
        interactive()
```

```
🔗 Connecting to MCP server...
✅ Connection successful!
🔌 Connection disconnected
✅ Tool 'mcp_get_weather' registered.
✅ Tool 'mcp_list_supported_cities' registered.
✅ Tool 'mcp_get_server_info' registered.
✅ MCP tool 'mcp' expanded into 3 independent tools

You: I want to query Beijing's weather
🔗 Connecting to MCP server...
✅ Connection successful!
🔌 Connection disconnected
Assistant: The current weather in Beijing is as follows:

- Temperature: 10.0°C
- Feels like: 9.0°C
- Humidity: 94%
- Weather condition: Light rain
- Wind speed: 1.7 m/s
- Visibility: 10.0 km
- Timestamp: October 9, 2025 13:46:40

Please bring rain gear and adjust your clothing according to weather changes.
```

### 10.5.2 Tải MCP Server Lên

Chúng ta đã tạo một MCP server truy vấn thời tiết thực sự. Bây giờ, hãy công bố nó lên nền tảng Smithery để các nhà phát triển trên toàn thế giới có thể dùng dịch vụ của chúng ta.

(1) Smithery là gì?

[Smithery](https://smithery.ai/) là nền tảng công bố chính thức cho các MCP server, tương tự như PyPI của Python hay npm của Node.js. Thông qua Smithery, người dùng có thể:

- 🔍 Khám phá và tìm kiếm các MCP server
- 📦 Cài đặt MCP server chỉ với một cú nhấp
- 📊 Xem thống kê sử dụng và đánh giá của server
- 🔄 Tự động nhận các bản cập nhật server

(2) Chuẩn bị cho Việc Công bố
Trước tiên, chúng ta cần tổ chức dự án thành định dạng công bố chuẩn. Thư mục này đã được tổ chức sẵn trong thư mục `code` để bạn tham khảo:

```
weather-mcp-server/
├── README.md           # Tài liệu dự án
├── LICENSE            # Giấy phép mã nguồn mở
├── Dockerfile         # Cấu hình build Docker (khuyến nghị)
├── pyproject.toml     # Cấu hình dự án Python (bắt buộc)
├── requirements.txt   # Các phụ thuộc Python
├── smithery.yaml      # Tập tin cấu hình Smithery (bắt buộc)
└── server.py          # Tập tin chính của MCP server
```

Lưu ý rằng `smithery.yaml` là tập tin cấu hình cho nền tảng Smithery:
```yaml
name: weather-mcp-server
displayName: Weather MCP Server
description: Real-time weather query MCP server based on HelloAgents framework
version: 1.0.0
author: HelloAgents Team
homepage: https://github.com/yourusername/weather-mcp-server
license: MIT
categories:
  - weather
  - data
tags:
  - weather
  - real-time
  - helloagents
  - wttr
runtime: container
build:
  dockerfile: Dockerfile
  dockerBuildPath: .
startCommand:
  type: http
tools:
  - name: get_weather
    description: Get current weather for a city
  - name: list_supported_cities
    description: List all supported cities
  - name: get_server_info
    description: Get server information
```

Giải thích cấu hình:

- `name`: Định danh duy nhất cho server (chữ thường, phân tách bằng dấu gạch nối)
- `displayName`: Tên hiển thị
- `description`: Mô tả ngắn gọn
- `version`: Số phiên bản (tuân theo semantic versioning)
- `runtime`: Môi trường chạy (python/node)
- `entrypoint`: Tập tin điểm vào
- `tools`: Danh sách công cụ

`pyproject.toml` là tập tin cấu hình chuẩn cho các dự án Python. Smithery yêu cầu tập tin này vì sau đó nó sẽ được đóng gói thành một server:

```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "weather-mcp-server"
version = "1.0.0"
description = "Real-time weather query MCP server based on HelloAgents framework"
readme = "README.md"
license = {text = "MIT"}
authors = [
    {name = "HelloAgents Team", email = "xxx"}
]
requires-python = ">=3.10"
dependencies = [
    "hello-agents>=0.2.1",
    "requests>=2.31.0",
]

[project.urls]
Homepage = "https://github.com/yourusername/weather-mcp-server"
Repository = "https://github.com/yourusername/weather-mcp-server"
"Bug Tracker" = "https://github.com/yourusername/weather-mcp-server/issues"

[tool.setuptools]
py-modules = ["server"]
```


Giải thích cấu hình:

- `[build-system]`: Chỉ định công cụ build (setuptools)
- `[project]`: Metadata của dự án
  - `name`: Tên dự án
  - `version`: Số phiên bản (tuân theo semantic versioning)
  - `dependencies`: Danh sách phụ thuộc của dự án
  - `requires-python`: Yêu cầu về phiên bản Python
- `[project.urls]`: Các liên kết liên quan đến dự án
- `[tool.setuptools]`: Cấu hình setuptools

Mặc dù Smithery tự động tạo Dockerfile, việc cung cấp một Dockerfile tùy chỉnh sẽ đảm bảo triển khai thành công:

```dockerfile
# Multi-stage build for weather-mcp-server
FROM python:3.12-slim-bookworm as base

# Set working directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

# Copy project files
COPY pyproject.toml requirements.txt ./
COPY server.py ./

# Install Python dependencies
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# Set environment variables
ENV PYTHONUNBUFFERED=1
ENV PORT=8081

# Expose port (Smithery uses 8081)
EXPOSE 8081

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import sys; sys.exit(0)"

# Run the MCP server
CMD ["python", "server.py"]
```

Giải thích cấu hình Dockerfile:

- **Base Image**: `python:3.12-slim-bookworm` - Image Python nhẹ
- **Working Directory**: `/app` - Thư mục gốc của ứng dụng
- **Port**: `8081` - Cổng tiêu chuẩn của nền tảng Smithery
- **Start Command**: `python server.py` - Chạy MCP server

Tại đây, chúng ta cần Fork kho `hello-agents`, lấy mã nguồn trong `code`, và tạo một kho tên `weather-mcp-server` bằng GitHub của riêng bạn, thay `yourusername` bằng tên người dùng GitHub của bạn.

(3) Gửi lên Smithery

Mở trình duyệt và truy cập [https://smithery.ai/](https://smithery.ai/). Đăng nhập vào Smithery bằng tài khoản GitHub của bạn. Nhấp nút "Publish Server" trên trang, nhập URL kho GitHub của bạn: `https://github.com/yourusername/weather-mcp-server`, và chờ công bố.

Sau khi công bố hoàn tất, bạn có thể thấy một trang tương tự như thế này, như minh họa trong Hình 10.10:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-10.png" alt="" width="85%"/>
  <p>Hình 10.10 Trang Công bố Thành công của Smithery</p>
</div>



Sau khi server được công bố thành công, người dùng có thể sử dụng nó theo các cách sau:

Cách 1: Thông qua Smithery CLI

```bash
# Cài đặt Smithery CLI
npm install -g @smithery/cli

# Cài đặt server của bạn
smithery install weather-mcp-server
```

Cách 2: Cấu hình trong Claude Desktop

```json
{
  "mcpServers": {
    "weather": {
      "command": "smithery",
      "args": ["run", "weather-mcp-server"]
    }
  }
}
```

Cách 3: Sử dụng trong HelloAgents

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools.builtin.protocol_tools import MCPTool

agent = SimpleAgent(name="Weather Assistant", llm=HelloAgentsLLM())

# Dùng server đã cài qua Smithery
weather_tool = MCPTool(
    server_command=["smithery", "run", "weather-mcp-server"]
)
agent.add_tool(weather_tool)

response = agent.run("How's the weather in Beijing today?")
```

Tất nhiên, đây chỉ là một ví dụ, và còn nhiều cách dùng khác để bạn tự khám phá. Hình 10.11 dưới đây cho thấy thông tin được bao gồm khi một công cụ MCP được công bố thành công, hiển thị tên dịch vụ "Weather", định danh duy nhất `@jjyaoao/weather-mcp-server`, và thông tin trạng thái. Khu vực Tools hiển thị các phương thức chúng ta vừa hiện thực, và khu vực Connect cung cấp thông tin kỹ thuật cần thiết để kết nối và sử dụng dịch vụ này, bao gồm **địa chỉ URL truy cập** của dịch vụ và **các đoạn mã cấu hình** bằng nhiều ngôn ngữ/môi trường. Nếu muốn tìm hiểu thêm, bạn có thể nhấp vào [liên kết](https://smithery.ai/server/@jjyaoao/weather-mcp-server) này.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-11.png" alt="" width="85%"/>
  <p>Hình 10.11 Công cụ MCP Được Công bố Thành công trên Smithery</p>
</div>

Bây giờ là lúc để tạo MCP server của riêng bạn!



## 10.6 Tóm tắt Chương

Chương này đã giới thiệu một cách hệ thống ba giao thức cốt lõi cho giao tiếp agent: MCP, A2A và ANP, đồng thời khám phá triết lý thiết kế, kịch bản ứng dụng, và các phương pháp thực hành của chúng.

**Định vị các Giao thức:**

- **MCP (Model Context Protocol)**: Với vai trò cầu nối giữa agent và công cụ, cung cấp một giao diện truy cập công cụ thống nhất, phù hợp để tăng cường khả năng của từng agent riêng lẻ.
- **A2A (Agent-to-Agent Protocol)**: Với vai trò một hệ thống đối thoại giữa các agent, hỗ trợ giao tiếp trực tiếp và thương lượng nhiệm vụ, phù hợp cho sự cộng tác chặt chẽ trong các đội nhóm quy mô nhỏ.
- **ANP (Agent Network Protocol)**: Với vai trò "internet" cho các agent, cung cấp các cơ chế khám phá dịch vụ, định tuyến, và cân bằng tải, phù hợp để xây dựng các mạng lưới agent mở, quy mô lớn.

**Giải pháp Tích hợp của HelloAgents**

Trong framework `HelloAgents`, ba giao thức này được trừu tượng hóa thống nhất thành các công cụ (Tool), đạt được sự tích hợp liền mạch, cho phép các nhà phát triển linh hoạt bổ sung các khả năng giao tiếp ở các cấp độ khác nhau cho agent:

```python
# Giao diện Tool thống nhất
from hello_agents.tools import MCPTool, A2ATool, ANPTool

# Tất cả các giao thức đều có thể được thêm vào Agent dưới dạng Tool
agent.add_tool(MCPTool(...))
agent.add_tool(A2ATool(...))
agent.add_tool(ANPTool(...))
```

**Tóm tắt Kinh nghiệm Thực hành**

- Ưu tiên dùng các dịch vụ MCP cộng đồng trưởng thành để giảm bớt việc phát triển trùng lặp không cần thiết.
- Chọn giao thức phù hợp dựa trên quy mô hệ thống: A2A được khuyến nghị cho các kịch bản cộng tác quy mô nhỏ, trong khi ANP nên được dùng cho các kịch bản mạng lưới quy mô lớn.

Sau khi hoàn thành chương này, chúng tôi khuyên bạn nên:

1. **Thực hành Trực tiếp**:
   - Xây dựng MCP server của riêng bạn
   - Tạo các hệ thống cộng tác đa agent bằng các giao thức
   - Các chiến lược ứng dụng kết hợp MCP, A2A, và ANP
2. **Học Chuyên sâu**:
   - Đọc tài liệu chính thức của MCP: https://modelcontextprotocol.io
   - Đọc tài liệu chính thức của A2A: https://a2a-protocol.org/latest/
   - Đọc tài liệu chính thức của ANP: https://agent-network-protocol.com/guide/
3. **Tham gia Cộng đồng**:
   - Đóng góp các dịch vụ MCP mới cho cộng đồng
   - Chia sẻ các trường hợp hiện thực agent do bạn tự phát triển
   - Tham gia thảo luận về tiêu chuẩn kỹ thuật cho các giao thức liên quan, hoặc đặt câu hỏi trong Issues hay trực tiếp giúp HelloAgents hỗ trợ các trường hợp ví dụ mới

**Chúc mừng bạn đã hoàn thành Chương 10!**

Bạn giờ đây đã nắm vững kiến thức cốt lõi về các giao thức giao tiếp agent. Hãy tiếp tục phát huy nhé! 🚀

## Bài tập

> **Lưu ý**: Một số bài tập không có đáp án chuẩn. Trọng tâm là bồi dưỡng sự hiểu biết toàn diện và khả năng thực hành của người học về các giao thức giao tiếp agent.

1. Chương này đã giới thiệu ba giao thức giao tiếp agent: MCP, A2A và ANP. Hãy phân tích:

   - Phần 10.1.2 đã so sánh triết lý thiết kế của ba giao thức. Hãy phân tích sâu: Vì sao MCP nhấn mạnh "chia sẻ ngữ cảnh", A2A nhấn mạnh "cộng tác qua đối thoại", và ANP nhấn mạnh "cấu trúc mạng lưới (network topology)"? Các triết lý thiết kế này lần lượt giải quyết những bài toán cốt lõi nào?
   - Giả sử bạn muốn xây dựng một "hệ thống chăm sóc khách hàng thông minh" cần các chức năng sau: (1) Truy cập cơ sở dữ liệu khách hàng và hệ thống đơn hàng; (2) Nhiều agent chăm sóc khách hàng chuyên nghiệp cộng tác để xử lý các vấn đề phức tạp; (3) Hỗ trợ các yêu cầu đồng thời của người dùng ở quy mô lớn. Hãy chọn giao thức phù hợp nhất cho mỗi chức năng và giải thích lý do của bạn.
   - Ba giao thức có thể được dùng kết hợp không? Hãy thiết kế một kịch bản ứng dụng thực tế cho thấy cách dùng đồng thời MCP, A2A, và ANP để xây dựng một hệ thống agent hoàn chỉnh. Vẽ một sơ đồ kiến trúc hệ thống và giải thích trách nhiệm của mỗi giao thức.

2. MCP (Model Context Protocol) là giao thức tiêu chuẩn cho giao tiếp agent-công cụ. Dựa trên nội dung ở Phần 10.2, hãy suy nghĩ sâu:

   > **Lưu ý**: Đây là câu hỏi thực hành, khuyến nghị thao tác thực tế

   - Trong hiện thực MCP server ở Phần 10.2.3, chúng ta đã định nghĩa các phương thức cốt lõi như `list_tools` và `call_tool`. Hãy mở rộng hiện thực này bằng cách thêm một MCP server mới cung cấp các công cụ sau: (1) Công cụ truy vấn cơ sở dữ liệu; (2) Công cụ trực quan hóa dữ liệu; (3) Công cụ tạo báo cáo. Yêu cầu các công cụ có thể cộng tác để hoàn thành các nhiệm vụ phân tích dữ liệu phức tạp.
   - Giao thức MCP hỗ trợ hai khái niệm quan trọng: "Resources" và "Prompts", nhưng chương này chủ yếu tập trung vào "Tools". Hãy tham khảo tài liệu chính thức của MCP để hiểu mục đích thiết kế của Resources và Prompts, và thiết kế một kịch bản ứng dụng cho thấy cách dùng ba khái niệm cốt lõi này để xây dựng một hệ thống agent mạnh mẽ hơn.
   - MCP dùng JSON-RPC 2.0 làm giao thức giao tiếp bên dưới và giao tiếp giữa các tiến trình qua stdio. Hãy phân tích: Thiết kế này có những ưu điểm và hạn chế gì? Nếu bạn cần hỗ trợ các MCP server từ xa (truy cập qua HTTP/WebSocket), hiện thực hiện tại nên được mở rộng như thế nào?

3. A2A (Agent-to-Agent Protocol) hỗ trợ sự cộng tác qua đối thoại giữa các agent. Dựa trên nội dung ở Phần 10.3, hãy hoàn thành bài thực hành mở rộng sau:

   > **Lưu ý**: Đây là câu hỏi thực hành, khuyến nghị thao tác thực tế

   - Trong trường hợp "đội nghiên cứu" ở Phần 10.3.4, researcher và writer cộng tác thông qua giao thức A2A để hoàn thành việc viết bài. Hãy mở rộng trường hợp này bằng cách thêm một agent thứ ba "Reviewer" (Người phản biện), có thể phản biện chất lượng bài viết và đưa ra các đề xuất chỉnh sửa. Thiết kế quy trình cộng tác giữa ba agent và hiện thực mã hoàn chỉnh.
   - Giao thức A2A định nghĩa các loại tin nhắn như `task` và `task_result`. Hãy phân tích: Nếu xảy ra xung đột trong quá trình cộng tác (chẳng hạn hai agent có ý kiến khác nhau về cùng một vấn đề), cơ chế giải quyết xung đột nên được thiết kế như thế nào? Hãy mở rộng giao thức A2A bằng cách thêm các loại tin nhắn như "negotiation" (thương lượng) và "voting" (bỏ phiếu).
   - So sánh giao thức A2A với các framework đa agent như AutoGen và CAMEL đã giới thiệu ở Chương 6: Mối quan hệ giữa A2A với vai trò một giao thức tiêu chuẩn và các framework này là gì? Chúng có thể thay thế cho nhau không? Hãy thiết kế một giải pháp cho phép các agent dựa trên giao thức A2A giao tiếp với các agent trong framework AutoGen.

4. ANP (Agent Network Protocol) hỗ trợ các mạng lưới agent quy mô lớn. Dựa trên nội dung ở Phần 10.4, hãy phân tích sâu:

   - Phần 10.4.2 đã giới thiệu thiết kế cấu trúc mạng lưới của ANP, bao gồm các cấu trúc hình sao (star), lưới (mesh), phân cấp (hierarchical), v.v. Hãy phân tích: Trong những kịch bản nào nên chọn cấu trúc topology nào? Nếu quy mô mạng mở rộng từ 10 agent lên 1000 agent, cấu trúc topology nên tiến hóa như thế nào?
   - Giao thức ANP hỗ trợ các cơ chế "routing" (định tuyến) và "discovery" (khám phá), cho phép các agent tìm được các đối tác cộng tác phù hợp một cách động. Hãy thiết kế một "thuật toán định tuyến thông minh": tự động chọn đường định tuyến tin nhắn tối ưu dựa trên loại nhiệm vụ, khả năng của agent, tải mạng, và các yếu tố khác.
   - Trong trường hợp "thành phố thông minh (smart city)" ở Phần 10.4.4, nhiều agent cộng tác để quản lý các hệ thống của thành phố. Hãy suy nghĩ: Nếu một agent quan trọng (chẳng hạn agent quản lý giao thông) gặp sự cố, toàn bộ hệ thống nên phản ứng như thế nào? Hãy thiết kế một "cơ chế chịu lỗi (fault tolerance mechanism)", bao gồm phát hiện lỗi, chuyển đổi dự phòng, khôi phục trạng thái, và các chức năng khác.

5. Bảo mật và bảo vệ quyền riêng tư của các giao thức giao tiếp agent là những vấn đề then chốt trong ứng dụng thực tế. Hãy suy nghĩ:

   - Trong hiện thực MCP client ở Phần 10.2.4, agent có thể gọi bất kỳ công cụ nào do MCP server cung cấp. Hãy phân tích: Thiết kế này có những rủi ro bảo mật gì? Nếu MCP server cung cấp các thao tác nguy hiểm (chẳng hạn xóa tập tin, thực thi lệnh hệ thống), cơ chế kiểm soát quyền nên được thiết kế như thế nào?
   - Các giao thức A2A và ANP liên quan đến giao tiếp giữa nhiều agent, có thể chứa thông tin nhạy cảm (như dữ liệu riêng tư của người dùng, bí mật kinh doanh). Hãy thiết kế một giải pháp "mã hóa đầu cuối (end-to-end encryption)": đảm bảo tin nhắn không bị nghe lén hoặc giả mạo trong quá trình truyền tải, đồng thời hỗ trợ xác thực danh tính agent và kiểm soát truy cập.
   - Trong các mạng lưới agent quy mô lớn, các agent độc hại có thể gửi thông tin giả, phát động các cuộc tấn công từ chối dịch vụ (denial-of-service), hoặc đánh cắp dữ liệu từ các agent khác. Hãy thiết kế một "hệ thống đánh giá độ tin cậy (trust evaluation system)": đánh giá động độ tin cậy của mỗi agent dựa trên hành vi lịch sử, chất lượng cộng tác, đánh giá của cộng đồng, và các yếu tố khác, và điều chỉnh chiến lược giao tiếp tương ứng.

## Tài liệu Tham khảo

[1] Anthropic. (2024). *Model Context Protocol*. Retrieved October 7, 2025, from https://modelcontextprotocol.io/

[2] The A2A Project. (2025). *A2A Protocol: An open protocol for agent-to-agent communication*. Retrieved October 7, 2025, from https://a2a-protocol.org/

[3] Chang, G., Lin, E., Yuan, C., Cai, R., Chen, B., Xie, X., & Zhang, Y. (2025). *Agent Network Protocol technical white paper*. arXiv. https://doi.org/10.48550/arXiv.2508.00007
