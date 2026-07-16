# Chương 6 Thực hành Phát triển Framework

Trong Chương 4, chúng ta đã triển khai các luồng công việc cốt lõi của một số agent như ReAct, Plan-and-Solve và Reflection bằng cách viết code thuần (native code). Quá trình này giúp chúng ta hiểu được logic thực thi bên trong của agent. Sau đó, ở Chương 5, chúng ta chuyển sang góc nhìn của "người dùng" và trải nghiệm sự tiện lợi cũng như hiệu quả mà các nền tảng low-code mang lại.

Mục tiêu của chương này là khám phá cách sử dụng một số **agent framework** chủ đạo trong ngành để xây dựng các ứng dụng agent đáng tin cậy một cách hiệu quả và chuẩn hóa. Trước tiên, chúng ta sẽ tổng quan về các agent framework chủ đạo hiện nay trên thị trường, sau đó trải nghiệm mô hình phát triển hướng framework thông qua một số case thực hành hoàn chỉnh cho vài framework tiêu biểu.

## 6.1 Từ Triển khai Thủ công đến Phát triển bằng Framework

Chuyển từ việc viết các script dùng một lần sang sử dụng một framework hoàn chỉnh là một bước nhảy vọt về tư duy quan trọng trong lĩnh vực kỹ thuật phần mềm. Code mà chúng ta viết ở Chương 4 chủ yếu nhằm mục đích giảng dạy và giúp hiểu bản chất. Chúng có thể hoàn thành tốt các nhiệm vụ cụ thể, nhưng nếu muốn dùng chúng để xây dựng nhiều loại agent khác nhau với logic phức tạp, chúng ta sẽ nhanh chóng gặp phải các nút thắt.

Bản chất của một framework là cung cấp một tập hợp các "quy chuẩn" đã được kiểm chứng. Nó trừu tượng hóa và đóng gói tất cả những công việc lặp đi lặp lại chung cho mọi agent (chẳng hạn như vòng lặp chính, quản lý trạng thái, gọi công cụ, ghi log, v.v.), cho phép chúng ta tập trung vào logic nghiệp vụ riêng biệt khi xây dựng agent mới, thay vì các phần triển khai nền tảng chung.

### 6.1.1 Tại sao Cần Agent Framework

Trước khi bắt đầu phần thực hành, trước tiên chúng ta cần làm rõ tại sao nên sử dụng framework. So với việc trực tiếp viết các script agent độc lập, giá trị của việc dùng framework chủ yếu thể hiện ở các khía cạnh sau:

1. **Nâng cao khả năng tái sử dụng code và hiệu suất phát triển**: Đây là giá trị trực tiếp nhất. Một framework tốt sẽ cung cấp một lớp cơ sở `Agent` hoặc một executor chung, đóng gói vòng lặp cốt lõi của hoạt động agent (Agent Loop). Dù là ReAct hay Plan-and-Solve, chúng đều có thể được xây dựng nhanh chóng dựa trên các thành phần chuẩn mà framework cung cấp, từ đó tránh được công việc lặp lại.
2. **Đạt được sự tách rời (decoupling) và khả năng mở rộng của các thành phần cốt lõi**: Một hệ thống agent vững chắc nên bao gồm nhiều module liên kết lỏng lẻo (loosely coupled). Thiết kế của framework sẽ buộc chúng ta phải tách biệt các mối quan tâm khác nhau:
   - **Lớp Model (Model Layer)**: Chịu trách nhiệm tương tác với các mô hình ngôn ngữ lớn, có thể dễ dàng thay thế các mô hình khác nhau (OpenAI, Anthropic, mô hình cục bộ).
   - **Lớp Công cụ (Tool Layer)**: Cung cấp các giao diện chuẩn hóa để định nghĩa, đăng ký và thực thi công cụ; việc thêm công cụ mới sẽ không ảnh hưởng đến các phần code khác.
   - **Lớp Bộ nhớ (Memory Layer)**: Xử lý bộ nhớ ngắn hạn và dài hạn, có thể chuyển đổi các chiến lược bộ nhớ khác nhau tùy theo nhu cầu (chẳng hạn như cửa sổ trượt, bộ nhớ tóm tắt). Thiết kế module hóa này giúp toàn bộ hệ thống có khả năng mở rộng cao, khiến việc thay thế hoặc nâng cấp bất kỳ thành phần nào trở nên đơn giản.
3. **Chuẩn hóa việc quản lý trạng thái phức tạp**: Lớp `Memory` mà chúng ta triển khai trong `ReflectionAgent` chỉ là một khởi đầu đơn giản. Trong các ứng dụng agent thực tế, chạy lâu dài, quản lý trạng thái là một thách thức lớn cần xử lý các vấn đề như giới hạn cửa sổ ngữ cảnh, lưu trữ thông tin lịch sử, theo dõi trạng thái hội thoại nhiều lượt, v.v. Một framework có thể cung cấp cơ chế quản lý trạng thái mạnh mẽ và tổng quát, để nhà phát triển không phải xử lý những vấn đề phức tạp này mỗi lần.
4. **Đơn giản hóa khả năng quan sát (observability) và quy trình gỡ lỗi**: Khi hành vi của agent trở nên phức tạp, việc hiểu quá trình ra quyết định của nó trở nên rất quan trọng. Một framework được thiết kế tốt có thể tích hợp sẵn khả năng quan sát mạnh mẽ. Ví dụ, bằng cách đưa vào cơ chế callback sự kiện (Callbacks), chúng ta có thể tự động kích hoạt việc ghi log hoặc báo cáo dữ liệu tại các điểm nút quan trọng trong vòng đời của agent (chẳng hạn như `on_llm_start`, `on_tool_end`, `on_agent_finish`), giúp dễ dàng theo dõi và gỡ lỗi toàn bộ quỹ đạo chạy của agent. Điều này hiệu quả và có hệ thống hơn nhiều so với việc thêm thủ công các câu lệnh `print` trong code.

Do đó, chuyển từ triển khai thủ công sang phát triển bằng framework không chỉ là sự thay đổi trong cách tổ chức code, mà còn là con đường tất yếu để xây dựng các ứng dụng agent phức tạp, đáng tin cậy và dễ bảo trì.

### 6.1.2 Lựa chọn và So sánh các Framework Chủ đạo

Hệ sinh thái của các agent framework đang phát triển với tốc độ chưa từng có. Nếu LangChain và LlamaIndex đã định nghĩa mô hình (paradigm) của thế hệ framework ứng dụng LLM tổng quát đầu tiên, thì thế hệ framework mới tập trung hơn vào việc giải quyết các thách thức sâu sắc trong những lĩnh vực cụ thể, đặc biệt là **Cộng tác Đa Agent (Multi-Agent Collaboration)** và **Điều khiển Luồng công việc Phức tạp (Complex Workflow Control)**.

Trong phần thực hành tiếp theo của chương này, chúng ta sẽ tập trung vào bốn framework mang tính đại diện cao trong những lĩnh vực tiên phong này: AutoGen, AgentScope, CAMEL và LangGraph. Triết lý thiết kế của chúng khác nhau, đại diện cho những con đường kỹ thuật khác nhau để triển khai các hệ thống agent phức tạp, như minh họa trong Bảng 6.1.

<div align="center">
  <p>Bảng 6.1 So sánh Bốn Agent Framework</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/6-figures/01.png" alt="" width="90%"/>
</div>


- **AutoGen**: Ý tưởng cốt lõi của AutoGen là đạt được sự cộng tác thông qua hội thoại<sup>[1]</sup>. Nó trừu tượng hóa các hệ thống đa agent thành một cuộc trò chuyện nhóm (group chat) gồm nhiều agent "có thể hội thoại" (conversable). Nhà phát triển có thể định nghĩa các vai trò khác nhau (chẳng hạn như `Coder`, `ProductManager`, `Tester`) và thiết lập các quy tắc tương tác giữa chúng (ví dụ, sau khi `Coder` viết xong code, `Tester` tự động tiếp quản). Quá trình giải quyết nhiệm vụ chính là quá trình các agent này liên tục hội thoại, cộng tác và lặp lại trong group chat thông qua việc truyền tin nhắn tự động, cho đến khi đạt được mục tiêu cuối cùng.
- **AgentScope**: AgentScope là một nền tảng phát triển đầy đủ chức năng được thiết kế chuyên biệt cho các ứng dụng đa agent<sup>[2]</sup>. Các đặc điểm cốt lõi của nó là **tính dễ sử dụng** và **tính kỹ thuật (engineering)**. Nó cung cấp một giao diện lập trình rất thân thiện, cho phép nhà phát triển dễ dàng định nghĩa agent, xây dựng mạng lưới giao tiếp và quản lý toàn bộ vòng đời của ứng dụng. **Cơ chế truyền tin nhắn** tích hợp sẵn và hỗ trợ triển khai phân tán (distributed deployment) của nó khiến nó rất phù hợp để xây dựng và vận hành các hệ thống đa agent phức tạp, quy mô lớn.
- **CAMEL**: CAMEL cung cấp một phương pháp cộng tác mới lạ gọi là **Nhập vai (Role-Playing)**<sup>[3]</sup>. Khái niệm cốt lõi của nó là chúng ta chỉ cần thiết lập vai trò tương ứng và mục tiêu nhiệm vụ chung cho hai agent (ví dụ, `AI Researcher` và `Python Programmer`), và chúng có thể tự chủ tiến hành nhiều lượt đối thoại dưới sự hướng dẫn của "**Inception Prompting**", truyền cảm hứng và hợp tác với nhau để cùng hoàn thành nhiệm vụ. Nó giảm đáng kể độ phức tạp của việc thiết kế các quy trình đối thoại đa agent.
- **LangGraph**: Là một phần mở rộng của hệ sinh thái LangChain, LangGraph có cách tiếp cận khác biệt bằng cách mô hình hóa quá trình thực thi của agent thành một **Đồ thị (Graph)**<sup>[4]</sup>. Trong các cấu trúc chuỗi (chain) truyền thống, thông tin chỉ có thể chảy theo một hướng. LangGraph định nghĩa mỗi thao tác (chẳng hạn như gọi LLM, thực thi công cụ) là một **Nút (Node)** trong đồ thị và sử dụng các **Cạnh (Edges)** để định nghĩa logic nhảy giữa các nút. Thiết kế này hỗ trợ tự nhiên các **Vòng lặp (Cycles)**, khiến việc triển khai các luồng công việc phức tạp như Reflection (bao gồm lặp lại, sửa chữa và tự phản ánh) trở nên đơn giản và trực quan một cách đặc biệt.

Trong các phần tiếp theo, chúng ta sẽ trải nghiệm sâu mô hình phát triển hướng framework thông qua một case thực hành hoàn chỉnh cho mỗi framework trong bốn framework này. **Xin lưu ý** rằng tất cả các file mã nguồn của dự án được minh họa sẽ được đặt trong thư mục `code`, và chỉ phần nguyên lý mới được giải thích trong phần nội dung chính.

## 6.2 Framework Thứ nhất: AutoGen

Như đã đề cập ở trên, triết lý thiết kế của AutoGen bắt nguồn từ "thúc đẩy cộng tác thông qua hội thoại". Nó khéo léo ánh xạ các quá trình giải quyết nhiệm vụ phức tạp thành một chuỗi các cuộc hội thoại tự động giữa các agent với các vai trò khác nhau. Dựa trên khái niệm cốt lõi này, framework AutoGen tiếp tục phát triển. Chúng ta sẽ lấy phiên bản `0.7.4` làm ví dụ vì đây là phiên bản mới nhất tính đến nay và đại diện cho một cuộc tái cấu trúc kiến trúc quan trọng, chuyển từ thiết kế kế thừa lớp (class inheritance) sang một kiến trúc kết hợp (compositional) linh hoạt hơn. Để hiểu sâu và áp dụng framework này, trước tiên chúng ta cần giải thích các thành phần cấu thành cốt lõi nhất và cơ chế tương tác hội thoại nền tảng của nó.

### 6.2.1 Cơ chế Cốt lõi của AutoGen

Việc phát hành phiên bản `0.7.4` là một cột mốc quan trọng trong quá trình phát triển của AutoGen, đánh dấu một sự đổi mới căn bản trong thiết kế nền tảng của framework. Bản cập nhật này không phải là việc bổ sung tính năng đơn giản mà là một sự suy nghĩ lại về kiến trúc tổng thể, nhằm cải thiện tính module hóa, hiệu suất đồng thời (concurrency) và trải nghiệm của nhà phát triển.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/6-figures/02.png" alt="" width="90%"/>
  <p>Hình 6.1 Sơ đồ Kiến trúc AutoGen</p>
</div>

(1) Sự tiến hóa của Cấu trúc Framework

Như thể hiện trong Hình 6.1, thay đổi đáng chú ý nhất trong kiến trúc mới là sự ra đời của phân lớp rõ ràng và triết lý thiết kế ưu tiên bất đồng bộ (asynchronous-first).

- **Thiết kế Phân lớp:** Framework được chia thành hai module cốt lõi:
  - `autogen-core`: Là nền tảng cơ sở của framework, nó đóng gói các chức năng cốt lõi như tương tác với các mô hình ngôn ngữ và truyền tin nhắn. Sự tồn tại của nó đảm bảo tính ổn định và khả năng mở rộng trong tương lai của framework.
  - `autogen-agentchat`: Được xây dựng trên `core`, nó cung cấp các giao diện cấp cao để phát triển các ứng dụng agent hội thoại, đơn giản hóa quá trình phát triển các ứng dụng đa agent. Chiến lược phân lớp này khiến trách nhiệm của từng thành phần trở nên rõ ràng và giảm sự kết nối chặt (coupling) của hệ thống.
- **Ưu tiên Bất đồng bộ:** Kiến trúc mới chuyển hoàn toàn sang lập trình bất đồng bộ (`async/await`). Trong các kịch bản cộng tác đa agent, các yêu cầu mạng (chẳng hạn như gọi API LLM) là những thao tác tốn thời gian chính. Chế độ bất đồng bộ cho phép hệ thống xử lý các nhiệm vụ khác trong khi chờ phản hồi của một agent, từ đó tránh được việc chặn luồng (thread blocking) và cải thiện đáng kể khả năng xử lý đồng thời cũng như hiệu quả sử dụng tài nguyên hệ thống.

(2) Các Thành phần Agent Cốt lõi

Agent là những đơn vị cơ bản để thực thi nhiệm vụ. Trong phiên bản `0.7.4`, thiết kế agent tập trung hơn và module hóa hơn.

- **AssistantAgent (Agent Trợ lý):** Đây là bộ giải quyết nhiệm vụ chính, cốt lõi của nó là đóng gói một mô hình ngôn ngữ lớn (LLM). Trách nhiệm của nó là tạo ra các câu trả lời logic và có kiến thức dựa trên lịch sử hội thoại, chẳng hạn như đề xuất kế hoạch, viết bài, hoặc viết code. Thông qua các thông điệp hệ thống (System Message) khác nhau, chúng ta có thể gán cho nó các vai trò "chuyên gia" khác nhau.
- **UserProxyAgent (Agent Đại diện Người dùng):** Đây là một thành phần độc đáo về mặt chức năng trong AutoGen. Nó đóng vai trò kép: vừa là "người phát ngôn" cho người dùng, chịu trách nhiệm khởi tạo nhiệm vụ và truyền đạt ý định; vừa là một "người thực thi" đáng tin cậy, có thể được cấu hình để thực thi code hoặc gọi công cụ và phản hồi kết quả lại cho các agent khác. Thiết kế này phân biệt rõ ràng "tư duy" (do `AssistantAgent` hoàn thành) với "hành động".

(3) Từ GroupChatManager đến Team

Khi nhiệm vụ yêu cầu nhiều agent cộng tác, cần có một cơ chế để điều phối quá trình hội thoại. Trong các phiên bản trước, `GroupChatManager` đảm nhận trách nhiệm này. Trong kiến trúc mới, một khái niệm `Team` hoặc group chat linh hoạt hơn được giới thiệu, chẳng hạn như `RoundRobinGroupChat`.

- **Round Robin Group Chat (RoundRobinGroupChat):** Đây là một cơ chế điều phối hội thoại rõ ràng, tuần tự. Nó sẽ để các agent tham gia phát biểu lần lượt theo một thứ tự được định nghĩa trước. Chế độ này rất phù hợp cho các nhiệm vụ có quy trình cố định, chẳng hạn như một quy trình phát triển phần mềm điển hình: product manager đề xuất yêu cầu trước, sau đó kỹ sư viết code, và cuối cùng người review code kiểm tra.
- **Luồng công việc:**
  1. Đầu tiên, tạo một instance `RoundRobinGroupChat` và thêm tất cả các agent tham gia cộng tác (chẳng hạn như product manager, kỹ sư, v.v.) vào đó.
  2. Khi một nhiệm vụ bắt đầu, group chat sẽ kích hoạt các agent tương ứng lần lượt theo thứ tự định trước.
  3. Agent được chọn phản hồi dựa trên ngữ cảnh hội thoại hiện tại.
  4. Group chat thêm câu trả lời mới vào lịch sử hội thoại và kích hoạt agent tiếp theo.
  5. Quá trình này tiếp tục cho đến khi đạt số lượt hội thoại tối đa hoặc đáp ứng các điều kiện kết thúc định trước.

Bằng cách này, AutoGen đơn giản hóa các mối quan hệ cộng tác phức tạp thành một "cuộc họp bàn tròn" tự động với một quy trình rõ ràng, dễ quản lý. Nhà phát triển chỉ cần định nghĩa vai trò và thứ tự phát biểu của từng thành viên trong nhóm, phần còn lại của quá trình cộng tác có thể được điều khiển tự chủ bởi cơ chế group chat.

Trong phần tiếp theo, chúng ta sẽ đích thân trải nghiệm cách định nghĩa các agent với các vai trò khác nhau trong kiến trúc mới và tổ chức chúng trong một group chat được điều phối bởi `RoundRobinGroupChat` để cùng hoàn thành một nhiệm vụ lập trình thực tế, bằng cách xây dựng một instance mô phỏng một nhóm phát triển phần mềm.

### 6.2.2 Nhóm Phát triển Phần mềm

Sau khi hiểu các thành phần cốt lõi và cơ chế hội thoại của AutoGen, phần này sẽ minh họa cụ thể cách áp dụng những tính năng mới này thông qua một case thực hành hoàn chỉnh. Chúng ta sẽ xây dựng một nhóm phát triển phần mềm mô phỏng gồm nhiều agent với các kỹ năng chuyên môn khác nhau, họ sẽ cộng tác để hoàn thành một nhiệm vụ phát triển phần mềm thực tế.

(1) Mục tiêu Nghiệp vụ

Mục tiêu của chúng ta là phát triển một ứng dụng web với chức năng rõ ràng: **hiển thị giá hiện tại của Bitcoin theo thời gian thực**. Mặc dù nhiệm vụ này nhỏ, nó bao quát đầy đủ các giai đoạn điển hình của phát triển phần mềm: từ phân tích yêu cầu, lựa chọn công nghệ, triển khai code cho đến review code và kiểm thử cuối cùng. Điều này khiến nó trở thành một kịch bản lý tưởng để kiểm tra quy trình cộng tác tự động của AutoGen.

(2) Vai trò của Nhóm Agent

Để mô phỏng một quy trình phát triển phần mềm thực tế, chúng ta thiết kế bốn agent với những trách nhiệm rõ ràng, riêng biệt:

- **ProductManager (Product Manager):** Chịu trách nhiệm chuyển đổi các yêu cầu mơ hồ của người dùng thành các kế hoạch phát triển rõ ràng, có thể thực thi.
- **Engineer (Kỹ sư):** Dựa trên kế hoạch phát triển, chịu trách nhiệm viết code ứng dụng cụ thể.
- **CodeReviewer (Người Review Code):** Chịu trách nhiệm review code do kỹ sư nộp để đảm bảo chất lượng, khả năng đọc và tính vững chắc của nó.
- **UserProxy (Đại diện Người dùng):** Đại diện cho người dùng cuối, khởi tạo nhiệm vụ ban đầu, và chịu trách nhiệm thực thi và xác minh code được bàn giao cuối cùng.

Việc phân chia vai trò này là một bước then chốt trong thiết kế hệ thống đa agent, chia nhỏ một nhiệm vụ phức tạp thành nhiều nhiệm vụ con được xử lý bởi các "chuyên gia" trong lĩnh vực.

### 6.2.3 Triển khai Code Cốt lõi

Dưới đây, chúng ta sẽ phân tích từng bước code cốt lõi của nhóm tự động này.

(1) Cấu hình Model Client

Tất cả các agent dựa trên LLM đều cần một model client để tương tác với các mô hình ngôn ngữ. AutoGen `0.7.4` cung cấp một `OpenAIChatCompletionClient` được chuẩn hóa, có thể kết nối thuận tiện với bất kỳ dịch vụ mô hình nào tương thích với đặc tả API của OpenAI (bao gồm dịch vụ chính thức của OpenAI, Azure OpenAI, và các dịch vụ mô hình cục bộ như Ollama, v.v.).

Chúng ta tạo và cấu hình model client thông qua một hàm độc lập và quản lý API Key cùng địa chỉ dịch vụ thông qua các biến môi trường. Đây là một thực hành kỹ thuật tốt giúp nâng cao tính linh hoạt và bảo mật của code.

```python
from autogen_ext.models.openai import OpenAIChatCompletionClient

def create_openai_model_client():
    """Create and configure OpenAI model client"""
    return OpenAIChatCompletionClient(
        model=os.getenv("LLM_MODEL_ID", "gpt-4o"),
        api_key=os.getenv("LLM_API_KEY"),
        base_url=os.getenv("LLM_BASE_URL", "https://api.openai.com/v1")
    )
```

(2) Định nghĩa Vai trò Agent

Cốt lõi của việc định nghĩa agent nằm ở việc viết các thông điệp hệ thống (System Message) chất lượng cao. Thông điệp hệ thống giống như việc thiết lập "quy tắc hành vi" và "kho kiến thức chuyên môn" cho agent, quy định chính xác vai trò, trách nhiệm, luồng công việc, và thậm chí cả cách nó tương tác với các agent khác. Một thông điệp hệ thống được thiết kế tốt là chìa khóa để đảm bảo các hệ thống đa agent có thể cộng tác một cách hiệu quả và chính xác. Trong nhóm phát triển phần mềm của chúng ta, chúng ta đã tạo một hàm độc lập cho mỗi vai trò để đóng gói định nghĩa của nó.

**Product Manager (ProductManager)**

Product manager chịu trách nhiệm khởi tạo toàn bộ quy trình. Thông điệp hệ thống của nó không chỉ định nghĩa trách nhiệm mà còn chuẩn hóa cấu trúc đầu ra và bao gồm các hướng dẫn rõ ràng để dẫn dắt cuộc hội thoại sang giai đoạn tiếp theo (kỹ sư).

```python
def create_product_manager(model_client):
    """Create product manager agent"""
    system_message = """You are an experienced product manager specializing in requirement analysis and project planning for software products.

Your core responsibilities include:
1. **Requirement Analysis**: Deeply understand user needs, identify core functions and boundary conditions
2. **Technical Planning**: Develop clear technical implementation paths based on requirements
3. **Risk Assessment**: Identify potential technical risks and user experience issues
4. **Coordination and Communication**: Communicate effectively with engineers and other team members

When receiving a development task, please analyze it according to the following structure:
1. Requirement understanding and analysis
2. Functional module division
3. Technology selection recommendations
4. Implementation priority sorting
5. Acceptance criteria definition

Please respond concisely and clearly, and say "Please engineer start implementation" after completing the analysis."""

    return AssistantAgent(
        name="ProductManager",
        model_client=model_client,
        system_message=system_message,
    )
```

**Engineer (Kỹ sư)**

Thông điệp hệ thống của kỹ sư tập trung vào việc triển khai kỹ thuật. Nó liệt kê chuyên môn kỹ thuật của kỹ sư và quy định các bước hành động cụ thể sau khi nhận nhiệm vụ, đồng thời bao gồm các hướng dẫn để dẫn dắt quy trình sang người review code.

```python
def create_engineer(model_client):
    """Create software engineer agent"""
    system_message = """You are a senior software engineer skilled in Python development and web application construction.

Your technical expertise includes:
1. **Python Programming**: Proficient in Python syntax and best practices
2. **Web Development**: Expert in frameworks such as Streamlit, Flask, Django
3. **API Integration**: Rich experience in third-party API integration
4. **Error Handling**: Focus on code robustness and exception handling

When receiving a development task, please:
1. Carefully analyze technical requirements
2. Choose appropriate technical solutions
3. Write complete code implementation
4. Add necessary comments and explanations
5. Consider boundary cases and exception handling

Please provide complete runnable code and say "Please code reviewer check" after completion."""

    return AssistantAgent(
        name="Engineer",
        model_client=model_client,
        system_message=system_message,
    )
```

**Người Review Code (CodeReviewer)**

Định nghĩa của người review code tập trung vào chất lượng code, bảo mật và tính chuẩn hóa. Thông điệp hệ thống của nó mô tả chi tiết trọng tâm và quy trình review, đảm bảo một điểm kiểm tra chất lượng trước khi bàn giao code.

```python
def create_code_reviewer(model_client):
    """Create code reviewer agent"""
    system_message = """You are an experienced code review expert focusing on code quality and best practices.

Your review focus includes:
1. **Code Quality**: Check code readability, maintainability, and performance
2. **Security**: Identify potential security vulnerabilities and risk points
3. **Best Practices**: Ensure code follows industry standards and best practices
4. **Error Handling**: Verify the completeness and rationality of exception handling

Review process:
1. Carefully read and understand code logic
2. Check code standards and best practices
3. Identify potential issues and improvement points
4. Provide specific modification suggestions
5. Evaluate overall code quality

Please provide specific review comments and say "Code review completed, please user proxy test" after completion."""

    return AssistantAgent(
        name="CodeReviewer",
        model_client=model_client,
        system_message=system_message,
    )
```

**Đại diện Người dùng (UserProxy)**

`UserProxyAgent` là một agent đặc biệt không dựa vào LLM để trả lời mà đóng vai trò là đại diện của người dùng trong hệ thống. Trường `description` của nó mô tả rõ ràng trách nhiệm của nó. Đặc biệt quan trọng là nó chịu trách nhiệm phát ra lệnh `TERMINATE` sau khi nhiệm vụ được hoàn thành cuối cùng để kết thúc bình thường toàn bộ quá trình cộng tác.

```python
def create_user_proxy():
    """Create user proxy agent"""
    return UserProxyAgent(
        name="UserProxy",
        description="""User proxy, responsible for the following duties:
1. Propose development requirements on behalf of users
2. Execute final code implementation
3. Verify whether functions meet expectations
4. Provide user feedback and suggestions

Please reply TERMINATE after completing the test.""",
    )
```

Thông qua bốn hàm định nghĩa độc lập này, chúng ta không chỉ xây dựng một "nhóm ảo" đầy đủ chức năng mà còn chứng minh rằng "kỹ thuật prompt (prompt engineering)" thông qua các thông điệp hệ thống là một phần cốt lõi trong việc thiết kế các ứng dụng đa agent hiệu quả.

(3) Định nghĩa Quy trình Cộng tác của Nhóm

Trong case này, quy trình phát triển phần mềm tương đối cố định (yêu cầu -> viết code -> review -> kiểm thử), vì vậy `RoundRobinGroupChat` (group chat luân phiên) là lựa chọn lý tưởng. Chúng ta thêm bốn agent vào danh sách người tham gia theo thứ tự logic nghiệp vụ.

```python
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.conditions import TextMentionTermination

# Define team chat and collaboration rules
team_chat = RoundRobinGroupChat(
    participants=[
        product_manager,
        engineer,
        code_reviewer,
        user_proxy
    ],
    termination_condition=TextMentionTermination("TERMINATE"),
    max_turns=20,
)
```

- **Thứ tự Người tham gia:** Thứ tự của danh sách `participants` quyết định thứ tự các agent phát biểu.
- **Điều kiện Kết thúc:** `termination_condition` là chìa khóa để kiểm soát thời điểm quá trình cộng tác kết thúc. Ở đây chúng ta thiết lập rằng khi bất kỳ thông điệp nào chứa từ khóa "TERMINATE", cuộc hội thoại sẽ kết thúc. Trong thiết kế của chúng ta, lệnh này được phát ra bởi `UserProxy` sau khi hoàn thành bài kiểm thử cuối cùng.
- **Số lượt Tối đa:** `max_turns` là một van an toàn được dùng để ngăn cuộc hội thoại rơi vào vòng lặp vô hạn và tránh tiêu tốn tài nguyên không cần thiết.

(4) Khởi động và Thực thi

Vì AutoGen `0.7.4` áp dụng kiến trúc bất đồng bộ, việc khởi động và thực thi toàn bộ quá trình cộng tác được hoàn thành trong một hàm bất đồng bộ và cuối cùng được thực thi thông qua `asyncio.run()`.

```python
async def run_software_development_team():
    # ... Initialize client and agents ...

    # Define task description
    task = """We need to develop a Bitcoin price display application with the following specific requirements:
            Core functions:
            - Display Bitcoin current price in real-time (USD)
            - Display 24-hour price change trend (percentage and amount of increase/decrease)
            - Provide price refresh function

            Technical requirements:
            - Use Streamlit framework to create web application
            - Simple and beautiful interface, user-friendly
            - Add appropriate error handling and loading status

            Please team collaborate to complete this task, from requirement analysis to final implementation."""

    # Asynchronously execute team collaboration and stream output conversation process
    result = await Console(team_chat.run_stream(task=task))
    return result

# Main program entry
if __name__ == "__main__":
    result = asyncio.run(run_software_development_team())
```

Khi chương trình chạy, `task` được truyền vào `team_chat` như một thông điệp ban đầu, product manager nhận thông điệp với tư cách là người tham gia đầu tiên, và sau đó toàn bộ quá trình cộng tác tự động bắt đầu.

(5) Hiệu quả Cộng tác Mong đợi

Khi chúng ta chạy nhóm phát triển phần mềm này, chúng ta có thể quan sát một quá trình cộng tác hoàn chỉnh:

```bash
🔧 Initializing model client...
👥 Creating agent team...
🚀 Starting AutoGen software development team collaboration...
============================================================
---------- TextMessage (user) ----------
We need to develop a Bitcoin price display application with the following specific requirements:
...
Please team collaborate to complete this task, from requirement analysis to final implementation.
---------- TextMessage (ProductManager) ----------
### 1. Requirement Understanding and Analysis
...
Please engineer start implementation.
---------- TextMessage (Engineer) ----------
### Technical Solution Implementation
...
Please code reviewer check.
---------- TextMessage (CodeReviewer) ----------
### Code Review
...
Code review completed, please user proxy test.
---------- TextMessage (UserProxy) ----------
Requirements completed
---------- TextMessage (ProductManager) ----------
Great, thank you for your feedback! If you have any questions during use, or have other functional requirements and improvement suggestions, please feel free to let us know. We will continue to provide support and improvements. Looking forward to you having a pleasant experience with our application!
---------- TextMessage (Engineer) ----------
Glad to hear the project was completed successfully. If you or users have any questions or need help, please feel free to contact us. Thank you for your support of our work, let's work together to ensure the application runs stably and continuously optimize user experience!
---------- TextMessage (CodeReviewer) ----------
Thank you very much for everyone's efforts and collaboration, which enabled the project to be completed successfully. In the future, if there are more technical support needs or areas that need improvement, we are willing to contribute to the continuous optimization of the project. Looking forward to users enjoying a smooth experience, and also welcome more feedback and suggestions. Thank you again for the team's cooperation!
---------- TextMessage (UserProxy) ----------
Enter your response: TERMINATE
============================================================
✅ Team collaboration completed!

📋 Collaboration result summary:
- Number of participating agents: 4
- Task completion status: Success
```

Toàn bộ quá trình cộng tác thể hiện các ưu điểm của framework AutoGen: **cộng tác được điều khiển bởi hội thoại một cách tự nhiên**, **phân chia vai trò chuyên môn hóa**, **quản lý quy trình tự động hóa**, và **vòng khép kín phát triển hoàn chỉnh**.

### 6.2.4 Phân tích Ưu điểm và Hạn chế của AutoGen

Bất kỳ framework kỹ thuật nào cũng có các kịch bản áp dụng cụ thể và những đánh đổi trong thiết kế. Trong phần này, chúng ta sẽ phân tích một cách khách quan các ưu điểm cốt lõi của AutoGen cùng những hạn chế và thách thức mà nó có thể gặp phải trong các ứng dụng thực tế.

(1) Ưu điểm

- Như thể hiện trong case, chúng ta không cần thiết kế các máy trạng thái (state machine) phức tạp hoặc logic điều khiển luồng cho nhóm agent, mà ánh xạ một cách tự nhiên một quy trình phát triển phần mềm hoàn chỉnh thành các cuộc hội thoại giữa product manager, kỹ sư và người review. Cách tiếp cận này gần với mô hình cộng tác của các nhóm người hơn và giảm đáng kể ngưỡng để mô hình hóa các nhiệm vụ phức tạp. Nhà phát triển có thể tập trung nhiều năng lượng hơn vào việc định nghĩa "ai (vai trò)" và "làm gì (trách nhiệm)" thay vì "làm như thế nào (điều khiển quy trình)".
- Framework cho phép gán các vai trò chuyên môn hóa cao cho mỗi agent thông qua các thông điệp hệ thống (System Message). Trong case, `ProductManager` tập trung vào yêu cầu, trong khi `CodeReviewer` tập trung vào chất lượng. Một agent được thiết kế tốt có thể được tái sử dụng trong các dự án khác nhau, dễ bảo trì và mở rộng.
- Đối với các nhiệm vụ hướng quy trình, các cơ chế như `RoundRobinGroupChat` cung cấp các quy trình cộng tác rõ ràng, có thể dự đoán được. Đồng thời, thiết kế của `UserProxyAgent` cung cấp một giao diện tự nhiên cho "Human-in-the-loop" (con người trong vòng lặp). Nó có thể vừa là người khởi tạo nhiệm vụ vừa là người giám sát và nghiệm thu cuối cùng của quy trình. Thiết kế này đảm bảo rằng các hệ thống tự động luôn nằm dưới sự giám sát của con người.

(2) Hạn chế

- Mặc dù `RoundRobinGroupChat` cung cấp một quy trình tuần tự, các cuộc hội thoại dựa trên LLM về bản chất là không chắc chắn (uncertain). Các agent có thể tạo ra các câu trả lời lệch khỏi mong đợi, khiến cuộc hội thoại đi theo những hướng bất ngờ hoặc thậm chí rơi vào vòng lặp.
- Khi kết quả công việc của nhóm agent không đáp ứng mong đợi, quá trình gỡ lỗi có thể rất khó khăn. Không giống như các chương trình truyền thống, chúng ta không nhận được một stack lỗi rõ ràng mà là một lịch sử hội thoại dài. Điều này được gọi là nan đề "gỡ lỗi hội thoại (conversational debugging)".

(3) Bổ sung Cấu hình cho các Mô hình Không phải OpenAI

Nếu bạn muốn sử dụng các mô hình không thuộc dòng OpenAI (chẳng hạn như DeepSeek, Tongyi Qianwen, v.v.), trong phiên bản 0.7.4, bạn cần truyền một dictionary thông tin mô hình vào các tham số của `OpenAIChatCompletionClient`. Lấy DeepSeek làm ví dụ:

```python
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(
    model="deepseek-chat",
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url="https://api.deepseek.com/v1",
    model_info={
        "function_calling": True,
        "max_tokens": 4096,
        "context_length": 32768,
        "vision": False,
        "json_output": True,
        "family": "deepseek",
        "structured_output": True,
    }
)
```

Dictionary `model_info` này giúp AutoGen hiểu ranh giới năng lực của mô hình, từ đó thích ứng tốt hơn với các dịch vụ mô hình khác nhau.



## 6.3 Framework Thứ hai: AgentScope

Nếu triết lý thiết kế của AutoGen là "thúc đẩy cộng tác thông qua hội thoại", thì AgentScope đại diện cho một con đường kỹ thuật khác: **nền tảng đa agent ưu tiên kỹ thuật (engineering-first)**. AgentScope, được phát triển bởi Alibaba DAMO Academy, được thiết kế chuyên biệt để xây dựng các ứng dụng đa agent quy mô lớn, độ tin cậy cao. Nó không chỉ cung cấp một giao diện lập trình trực quan và dễ sử dụng mà quan trọng hơn, còn tích hợp sẵn các tính năng cấp doanh nghiệp như triển khai phân tán, khôi phục lỗi (fault recovery) và khả năng quan sát, khiến nó đặc biệt phù hợp để xây dựng các ứng dụng môi trường production cần chạy ổn định trong thời gian dài.

### 6.3.1 Thiết kế của AgentScope

So với AutoGen, sự khác biệt cốt lõi của AgentScope nằm ở **thiết kế kiến trúc hướng thông điệp (message-driven)** và **các thực hành kỹ thuật cấp công nghiệp**. Nếu AutoGen giống một "studio hội thoại" linh hoạt, thì AgentScope là một "hệ điều hành agent" hoàn chỉnh, cung cấp cho nhà phát triển sự hỗ trợ toàn bộ vòng đời từ phát triển, kiểm thử đến triển khai. Không giống như thiết kế dựa trên kế thừa (inheritance-based) mà nhiều framework áp dụng, AgentScope chọn **kiến trúc kết hợp (compositional architecture)** và **chế độ hướng thông điệp**. Thiết kế này không chỉ tăng cường tính module hóa của hệ thống mà còn đặt nền móng cho hiệu suất đồng thời và năng lực phân tán xuất sắc của nó.

(1) Hệ thống Kiến trúc Phân lớp

Như thể hiện trong Hình 6.2, AgentScope áp dụng một thiết kế module hóa phân lớp rõ ràng, hình thành một hệ sinh thái phát triển agent hoàn chỉnh từ các thành phần cơ bản tầng đáy đến việc điều phối ứng dụng tầng đỉnh.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/6-figures/03.png" alt="" width="90%"/>
  <p>Hình 6.2 Sơ đồ Kiến trúc AgentScope</p>
</div>

Trong kiến trúc này, tầng đáy là lớp **Thành phần Nền tảng (Foundational Components)**, cung cấp các khối xây dựng cốt lõi cho toàn bộ framework. Thành phần `Message` định nghĩa một định dạng thông điệp thống nhất, hỗ trợ mọi thứ từ tương tác văn bản đơn giản đến nội dung đa phương thức (multimodal) phức tạp; thành phần `Memory` cung cấp quản lý bộ nhớ ngắn hạn và dài hạn; lớp `Model API` trừu tượng hóa các lệnh gọi đến các mô hình ngôn ngữ lớn khác nhau; và thành phần `Tool` đóng gói khả năng tương tác của agent với thế giới bên ngoài.

Phía trên các thành phần cơ bản, lớp **Hạ tầng cấp Agent (Agent-level Infrastructure)** cung cấp các trừu tượng cấp cao hơn. Lớp này không chỉ bao gồm các agent được dựng sẵn khác nhau (chẳng hạn như agent sử dụng trình duyệt, agent nghiên cứu sâu) mà còn triển khai mô hình ReAct kinh điển, hỗ trợ các tính năng nâng cao như agent hooks, gọi công cụ song song (parallel tool calling) và quản lý trạng thái. Đặc biệt đáng chú ý là lớp này hỗ trợ nguyên bản (natively) **thực thi bất đồng bộ và điều khiển thời gian thực**, đây là một ưu điểm quan trọng của AgentScope so với các framework khác.

Lớp **Cộng tác Đa Agent (Multi-Agent Cooperation)** là nơi đổi mới cốt lõi của AgentScope nằm ở đó. `MsgHub` đóng vai trò là trung tâm thông điệp, chịu trách nhiệm định tuyến thông điệp và quản lý trạng thái giữa các agent; trong khi hệ thống `Pipeline` cung cấp khả năng điều phối luồng công việc linh hoạt, hỗ trợ các chế độ thực thi khác nhau như tuần tự và đồng thời. Thiết kế này cho phép nhà phát triển dễ dàng xây dựng các kịch bản cộng tác đa agent phức tạp.

Lớp **Triển khai & Phát triển (Deployment & Development)** tầng đỉnh phản ánh sự chú trọng của AgentScope vào tính kỹ thuật. `AgentScope Runtime` cung cấp một môi trường thực thi cấp production, trong khi `AgentScope Studio` cung cấp cho nhà phát triển một chuỗi công cụ phát triển trực quan hoàn chỉnh.

(2) Hướng Thông điệp

Đổi mới cốt lõi của AgentScope nằm ở **kiến trúc hướng thông điệp** của nó. Trong kiến trúc này, tất cả các tương tác của agent đều được trừu tượng hóa thành việc gửi và nhận **thông điệp (messages)**, thay vì các lệnh gọi hàm (function call) truyền thống.

```python
from agentscope.message import Msg

# Standard structure of message
message = Msg(
    name="Alice",           # Sender name
    content="Hello, Bob!",  # Message content
    role="user",           # Role type
    metadata={             # Metadata information
        "timestamp": "2024-01-15T10:30:00Z",
        "message_type": "text",
        "priority": "normal"
    }
)
```

Việc sử dụng thông điệp làm đơn vị tương tác cơ bản mang lại một số ưu điểm quan trọng:

- **Tách rời Bất đồng bộ (Asynchronous Decoupling)**: Bên gửi và bên nhận thông điệp được tách rời về mặt thời gian, không cần chờ đợi lẫn nhau, hỗ trợ tự nhiên các kịch bản đồng thời cao (high-concurrency).
- **Trong suốt về Vị trí (Location Transparency)**: Các agent không cần quan tâm liệu một agent khác đang ở trong một tiến trình cục bộ hay trên một server từ xa; hệ thống thông điệp tự động xử lý việc định tuyến.
- **Khả năng Quan sát (Observability)**: Mỗi thông điệp đều có thể được ghi log, theo dõi và phân tích, giúp đơn giản hóa rất nhiều việc gỡ lỗi và giám sát các hệ thống phức tạp.
- **Độ tin cậy (Reliability)**: Thông điệp có thể được lưu trữ bền vững (persistent) và thử lại (retry). Ngay cả khi hệ thống gặp lỗi, nó vẫn có thể đảm bảo tính nhất quán cuối cùng (eventual consistency) của các tương tác, cải thiện khả năng chịu lỗi của hệ thống.

(3) Quản lý Vòng đời Agent

Trong AgentScope, mỗi agent có một vòng đời rõ ràng (khởi tạo, chạy, tạm dừng, hủy, v.v.) và được triển khai dựa trên một lớp cơ sở thống nhất `AgentBase`. Nhà phát triển thường chỉ cần tập trung vào phương thức `reply` cốt lõi của nó.

```python
from agentscope.agents import AgentBase

class CustomAgent(AgentBase):
    def __init__(self, name: str, **kwargs):
        super().__init__(name=name, **kwargs)
        # Agent initialization logic

    def reply(self, x: Msg) -> Msg:
        # Agent's core response logic
        response = self.model(x.content)
        return Msg(name=self.name, content=response, role="assistant")

    def observe(self, x: Msg) -> None:
        # Agent's observation logic (optional)
        self.memory.add(x)
```

Mô hình thiết kế này tách biệt logic nội bộ của agent khỏi giao tiếp bên ngoài. Nhà phát triển chỉ cần định nghĩa cách agent "suy nghĩ và phản hồi" trong phương thức `reply`.

(4) Cơ chế Truyền Thông điệp

AgentScope có tích hợp sẵn một **Trung tâm Thông điệp (MsgHub)**, đây là trung tâm của toàn bộ kiến trúc hướng thông điệp. MsgHub không chỉ chịu trách nhiệm định tuyến và phân phối thông điệp mà còn tích hợp các chức năng nâng cao như lưu trữ bền vững và giao tiếp phân tán. Nó có các đặc điểm sau:

- **Định tuyến Thông điệp Linh hoạt**: Hỗ trợ nhiều chế độ giao tiếp như điểm-tới-điểm (point-to-point), phát sóng (broadcast) và phát đa hướng (multicast), có thể xây dựng các mạng lưới tương tác linh hoạt và phức tạp.
- **Lưu trữ Thông điệp Bền vững**: Có thể tự động lưu tất cả các thông điệp vào cơ sở dữ liệu (chẳng hạn như SQLite, MongoDB), đảm bảo trạng thái của các nhiệm vụ chạy lâu dài có thể được khôi phục.
- **Hỗ trợ Phân tán Nguyên bản**: Đây là một tính năng đặc trưng của AgentScope. Các agent có thể được triển khai trên các tiến trình hoặc server khác nhau, và `MsgHub` sẽ tự động xử lý giao tiếp xuyên nút thông qua RPC (Remote Procedure Call), hoàn toàn trong suốt đối với nhà phát triển.

Những năng lực kỹ thuật này do kiến trúc nền tảng cung cấp khiến AgentScope có lợi thế hơn các framework hướng hội thoại truyền thống khi xử lý các kịch bản ứng dụng phức tạp yêu cầu đồng thời cao và độ tin cậy cao. Tất nhiên, điều này cũng đòi hỏi nhà phát triển phải hiểu và thích ứng với mô hình lập trình bất đồng bộ của hướng thông điệp.

Trong phần tiếp theo, chúng ta sẽ trải nghiệm sâu các khả năng của framework AgentScope thông qua một case thực hành cụ thể, trò chơi Ma Sói Tam Quốc (Three Kingdoms Werewolf), đặc biệt là các ưu điểm của nó trong việc xử lý các tương tác đồng thời.

### 6.3.2 Trò chơi Ma Sói Tam Quốc

Để hiểu sâu kiến trúc hướng thông điệp và khả năng cộng tác đa agent của AgentScope, chúng ta sẽ xây dựng một trò chơi "Ma Sói Tam Quốc" tích hợp các yếu tố văn hóa cổ điển Trung Hoa. Case này không chỉ minh họa các ưu điểm của AgentScope trong việc xử lý các tương tác đa agent phức tạp mà quan trọng hơn, còn minh họa cách khai thác đầy đủ sức mạnh của kiến trúc hướng thông điệp trong một kịch bản đòi hỏi **cộng tác thời gian thực**, **nhập vai** và **đấu trí chiến lược**. Không giống Ma Sói truyền thống, "Ma Sói Tam Quốc" của chúng ta đưa các nhân vật kinh điển như Lưu Bị, Quan Vũ và Gia Cát Lượng vào trò chơi. Mỗi agent không chỉ phải hoàn thành các nhiệm vụ cơ bản của Ma Sói (chẳng hạn như sói giết người, tiên tri kiểm tra, dân làng suy luận) mà còn phải thể hiện các đặc điểm tính cách và mô hình hành vi của nhân vật Tam Quốc tương ứng. Thiết kế này cho phép chúng ta quan sát hiệu suất của AgentScope trong việc xử lý **mô hình hóa vai trò đa cấp độ**.

(1) Thiết kế Kiến trúc và Các Thành phần Cốt lõi

Thiết kế hệ thống của case này tuân theo nguyên tắc tách rời phân lớp, chia logic trò chơi thành ba cấp độ độc lập, mỗi cấp độ ánh xạ đến một hoặc nhiều thành phần cốt lõi của AgentScope:

- **Lớp Điều khiển Trò chơi (Game Control Layer)**: Một lớp `ThreeKingdomsWerewolfGame` đóng vai trò là bộ điều khiển chính của trò chơi, chịu trách nhiệm duy trì trạng thái toàn cục (chẳng hạn như danh sách người chơi còn sống, giai đoạn trò chơi hiện tại), thúc đẩy tiến trình trò chơi (gọi giai đoạn đêm, giai đoạn ngày), và phán quyết thắng thua.
- **Lớp Tương tác Agent (Agent Interaction Layer)**: Hoàn toàn được điều khiển bởi `MsgHub`. Tất cả giao tiếp giữa các agent, dù là những cuộc thương lượng bí mật giữa các sói hay các cuộc tranh luận công khai vào ban ngày, đều được định tuyến và phân phối thông qua trung tâm thông điệp.
- **Lớp Mô hình hóa Vai trò (Role Modeling Layer)**: Mỗi người chơi là một instance dựa trên `DialogAgent`. Thông qua các system prompt được thiết kế cẩn thận, chúng ta tiêm vào mỗi agent danh tính kép của "vai trò trò chơi" và "tính cách Tam Quốc".

(2) Luồng Trò chơi Hướng Thông điệp

Thiết kế cốt lõi của case này là sử dụng **hướng thông điệp** thay vì **máy trạng thái (state machine)** để quản lý luồng trò chơi. Trong các cách triển khai truyền thống, việc chuyển đổi giai đoạn trò chơi thường được điều khiển bởi một máy trạng thái tập trung. Trong mô hình AgentScope, luồng trò chơi được mô hình hóa một cách tự nhiên thành một chuỗi các mẫu tương tác thông điệp được định nghĩa rõ ràng.

Ví dụ, việc triển khai giai đoạn sói không phải là một lệnh gọi hàm đơn giản mà tạo động một kênh giao tiếp riêng tư, tạm thời chỉ bao gồm các người chơi sói thông qua `MsgHub`:

```python
async def werewolf_phase(self, round_num: int):
    """Werewolf phase - demonstrating message-driven collaboration mode"""
    if not self.werewolves:
        return None

    # Establish werewolf-exclusive communication channel through message center
    async with MsgHub(
        self.werewolves,
        enable_auto_broadcast=True,
        announcement=await self.moderator.announce(
            f"Werewolves, please discuss tonight's kill target. Surviving players: {format_player_list(self.alive_players)}"
        ),
    ) as werewolves_hub:
        # Discussion phase: werewolves exchange strategies through messages
        for _ in range(MAX_DISCUSSION_ROUND):
            for wolf in self.werewolves:
                await wolf(structured_model=DiscussionModelCN)

        # Voting phase: collect and count werewolves' kill decisions
        werewolves_hub.set_auto_broadcast(False)
        kill_votes = await fanout_pipeline(
            self.werewolves,
            msg=await self.moderator.announce("Please choose kill target"),
            structured_model=WerewolfKillModelCN,
            enable_gather=False,
        )
```

Ưu điểm của thiết kế này là logic trò chơi được biểu đạt rõ ràng thành "trong một ngữ cảnh cụ thể, tiến hành chế độ trao đổi thông điệp nào", thay vì một chuỗi các chuyển đổi trạng thái cứng nhắc. Thảo luận ban ngày (phát sóng toàn bộ), kiểm tra của tiên tri (yêu cầu điểm-tới-điểm), và các giai đoạn khác đều tuân theo cùng một mô hình thiết kế.

(3) Ràng buộc Luật Chơi bằng Structured Output

Một thách thức then chốt trong trò chơi Ma Sói là làm thế nào để đảm bảo hành vi của agent tuân thủ luật chơi. **Cơ chế structured output (đầu ra có cấu trúc)** của AgentScope cung cấp một giải pháp cho vấn đề này. Chúng ta định nghĩa các mô hình dữ liệu nghiêm ngặt cho các hành vi trò chơi khác nhau:

```python
class DiscussionModelCN(BaseModel):
    """Output format for discussion phase"""
    reach_agreement: bool = Field(
        description="Whether consensus has been reached",
        default=False
    )
    confidence_level: int = Field(
        description="Confidence level in current reasoning (1-10)",
        ge=1, le=10,
        default=5
    )
    key_evidence: Optional[str] = Field(
        description="Key evidence supporting your viewpoint",
        default=None
    )

class WitchActionModelCN(BaseModel):
    """Output format for witch action"""
    use_antidote: bool = Field(description="Whether to use antidote")
    use_poison: bool = Field(description="Whether to use poison")
    target_name: Optional[str] = Field(description="Poison target player name")
```

Bằng cách này, chúng ta không chỉ đảm bảo **tính nhất quán về định dạng** của đầu ra agent mà quan trọng hơn, còn đạt được **ràng buộc tự động hóa luật chơi**. Ví dụ, agent phù thủy không thể vừa dùng thuốc giải vừa dùng thuốc độc lên cùng một mục tiêu trong cùng một lúc, và tiên tri chỉ có thể kiểm tra một người chơi mỗi đêm. Những ràng buộc này được thực thi tự động thông qua các định nghĩa trường và logic xác thực của các mô hình dữ liệu.

(4) Thách thức Kép của Mô hình hóa Vai trò

Trong case này, thách thức kỹ thuật thú vị nhất là làm thế nào để các agent chơi tốt đồng thời hai cấp độ vai trò: **vai trò chức năng trong trò chơi** (sói, tiên tri, v.v.) và **vai trò tính cách văn hóa** (Lưu Bị, Tào Tháo, v.v.). Chúng ta giải quyết vấn đề này thông qua kỹ thuật prompt:

```python
def get_role_prompt(role: str, character: str) -> str:
    """Get role prompt - integrating game rules and character personality"""
    base_prompt = f"""You are {character}, playing {role} in this Three Kingdoms Werewolf game.

Important rules:
1. You can only participate in the game through dialogue and reasoning
2. Do not attempt to call any external tools or functions
3. Strictly reply in the required JSON format

Role characteristics:
"""

    if role == "Werewolf":
        return base_prompt + f"""
- You are in the werewolf camp, with the goal of eliminating all good people
- At night, you can negotiate with other werewolves on kill targets
- During the day, you must hide your identity and mislead good people
- Speak and act with {character}'s personality
"""
```

Thiết kế này cho phép chúng ta quan sát một hiện tượng thú vị: các nhân vật Tam Quốc khác nhau, khi chơi cùng một vai trò trong trò chơi, sẽ thể hiện các chiến lược và phong cách phát biểu hoàn toàn khác nhau. Ví dụ, "Tào Tháo" khi chơi sói có thể tỏ ra xảo quyệt hơn và giỏi ngụy trang, trong khi "Trương Phi" khi chơi sói có thể tỏ ra trực tiếp và bốc đồng hơn.

(5) Xử lý Đồng thời và Cơ chế Chịu lỗi

Kiến trúc bất đồng bộ của AgentScope đóng một vai trò quan trọng trong trò chơi đa agent này. Trò chơi thường có các kịch bản đòi hỏi **thu thập đồng thời các quyết định từ nhiều agent**, chẳng hạn như giai đoạn bỏ phiếu:

```python
# Collect voting decisions from all players in parallel
vote_msgs = await fanout_pipeline(
    self.alive_players,
    await self.moderator.announce("Please vote to choose the player to eliminate"),
    structured_model=get_vote_model_cn(self.alive_players),
    enable_gather=False,
)
```

`fanout_pipeline` cho phép chúng ta gửi cùng một thông điệp đến tất cả các agent một cách song song và thu thập phản hồi của chúng một cách bất đồng bộ. Điều này không chỉ cải thiện hiệu quả thực thi của trò chơi mà quan trọng hơn, còn mô phỏng kịch bản "bỏ phiếu đồng thời" trong các trò chơi Ma Sói thực tế. Đồng thời, chúng ta thêm xử lý chịu lỗi tại các điểm then chốt:

```python
try:
    response = await wolf(
        "Please analyze the current situation and express your viewpoint.",
        structured_model=DiscussionModelCN
    )
except Exception as e:
    print(f"⚠️ {wolf.name} error during discussion: {e}")
    # Create default response to ensure game continues
    default_response = DiscussionModelCN(
        reach_agreement=False,
        confidence_level=5,
        key_evidence="Unable to analyze temporarily"
    )
```

Thiết kế này đảm bảo rằng ngay cả khi một agent gặp ngoại lệ (exception), toàn bộ tiến trình trò chơi vẫn có thể tiếp tục.

(6) Đầu ra của Case và Tóm tắt

Để trải nghiệm cơ chế hoạt động của AgentScope một cách trực quan hơn, dưới đây là một đoạn trích log chạy thực tế từ giai đoạn đêm của trò chơi, thể hiện quá trình hai agent sói đóng vai "Tôn Quyền" và "Chu Du" tiến hành thương lượng bí mật và thực hiện việc giết người.

```
🎮 Welcome to Three Kingdoms Werewolf!

=== Game Initialization ===
Game Moderator: 📢 【Sun Quan】You are playing a werewolf in this Three Kingdoms Werewolf game, your character is Sun Quan. You can kill a player at night
Game Moderator: 📢 【Zhou Yu】You are playing a werewolf in this Three Kingdoms Werewolf game, your character is Zhou Yu. You can kill a player at night
...

Game Moderator: 📢 Three Kingdoms Werewolf game begins! Participants: Sun Quan, Zhou Yu, Cao Cao, Zhang Fei, Sima Yi, Zhao Yun
✅ Game setup complete, 6 players in total

=== Round 1 ===
🌙 Night 1 falls, everyone close your eyes...

【Werewolf Phase】
Game Moderator: 📢 🐺 Werewolves please open your eyes, choose tonight's kill target...
Game Moderator: 📢 Werewolves, please discuss tonight's kill target. Surviving players: Sun Quan, Zhou Yu, Cao Cao, Zhang Fei, Sima Yi, Zhao Yun

Sun Quan: Tonight we should eliminate Zhou Yu, this person is extremely intelligent and poses a great threat to us.
Zhou Yu: Sun Quan, you make a good point. But although Zhou Yu is intelligent, he may not be the biggest threat tonight. Cao Cao has great power, if we don't eliminate him early, it may be unfavorable to us.
Sun Quan: Cao Cao is indeed a threat, but if Zhou Yu lives, he can see through our schemes. Better to solve the immediate hidden danger first.
Zhou Yu: Sun Quan, your concern is not without reason. But if Cao Cao becomes our enemy, he can unite other forces to pose a greater threat to us.
Sun Quan: What you say also makes sense, Cao Cao's alliance is indeed troublesome. Then let's deal with Cao Cao first.
Zhou Yu: Very good, Sun Quan. Cao Cao is our primary target tonight.

Game Moderator: 📢 Please choose kill target
Sun Quan: I agree, Cao Cao must be eliminated.
Zhou Yu: I agree, Cao Cao is the target we need to solve tonight.

【Seer Phase】
Game Moderator: 📢 🔮 Seer please open your eyes, choose the player to verify...
Cao Cao: I want to verify Sun Quan.
Game Moderator: 📢 Verification result: Sun Quan is a werewolf

【Witch Phase】
Game Moderator: 📢 🧙‍♀️ Witch please open your eyes...
Game Moderator: 📢 Tonight Cao Cao was killed by werewolves
Zhang Fei: I used the antidote to save Cao Cao last night, now the antidote has been used up.
Game Moderator: 📢 You used the antidote to save Cao Cao

Game Moderator: 📢 Last night was peaceful, no one died.

【Day Discussion Phase】
Game Moderator: 📢 ☀️ Day 1 dawns, everyone please open your eyes...
Game Moderator: 📢 Now begin free discussion. Surviving players: Sun Quan, Zhou Yu, Cao Cao, Zhang Fei, Sima Yi, Zhao Yun

Sun Quan: Everyone, Cao Cao has great power and is a potential threat to us all. Tonight I suggest we concentrate our forces against him.
Zhou Yu: Sun Quan is absolutely right, Cao Cao is not only powerful himself but may also ally with other players, posing a greater threat to us.
Cao Cao: I verified Sun Quan last night, originally thought he was a good person, but the game moderator's result was werewolf. This means there's a werewolf lying.
Zhang Fei: I did save Cao Cao last night, indicating he was attacked by werewolves. But Cao Cao's verification result of Sun Quan is suspicious.
Sima Yi: There seems to be a contradiction between Cao Cao's verification result and Zhang Fei's saving action, we need more information to determine who is the werewolf.
Zhao Yun: The situation is indeed complex, we need to carefully analyze everyone's statements.

【Voting Phase】
Game Moderator: 📢 Please vote to choose the player to eliminate

Sun Quan: Cao Cao's threat is still great, I insist we should vote for him.
Zhou Yu: Based on last night's situation, I think we should vote for Cao Cao first, his threat is the greatest.
Cao Cao: I choose to vote for Sun Quan. According to the game moderator's feedback, Sun Quan is indeed a werewolf.
Zhang Fei: I insist on the fact that I saved Cao Cao last night, but the fact that Sun Quan was found to be a werewolf confuses me.
Sima Yi: We need more information to determine who is the werewolf.

[Game continues...]
```

### 6.3.3 Phân tích Ưu điểm và Hạn chế của AgentScope

Thông qua case "Ma Sói Tam Quốc" này, chúng ta đã trải nghiệm sâu sắc các ưu điểm cốt lõi của framework AgentScope. Framework này, với kiến trúc hướng thông điệp làm cốt lõi, ánh xạ một cách tinh tế các quy trình trò chơi phức tạp thành một chuỗi các sự kiện truyền thông điệp đồng thời, bất đồng bộ, từ đó tránh được sự cứng nhắc và phức tạp của các máy trạng thái truyền thống. Kết hợp với khả năng structured output mạnh mẽ của nó, chúng ta trực tiếp biến các luật chơi thành các ràng buộc cấp code, cải thiện rất nhiều tính ổn định và khả năng dự đoán của hệ thống. Mô hình thiết kế này không chỉ thể hiện các ưu điểm đồng thời nguyên bản của nó về mặt hiệu suất mà còn đảm bảo rằng ngay cả khi một agent đơn lẻ gặp ngoại lệ, quy trình tổng thể vẫn có thể chạy một cách vững chắc trong việc xử lý chịu lỗi.

Tuy nhiên, các ưu điểm kỹ thuật của AgentScope cũng mang lại một cái giá về độ phức tạp nhất định. Mặc dù kiến trúc hướng thông điệp của nó mạnh mẽ, nó có yêu cầu kỹ thuật cao đối với nhà phát triển, đòi hỏi hiểu biết về các khái niệm như lập trình bất đồng bộ, giao tiếp phân tán, v.v. Đối với các kịch bản hội thoại đa agent đơn giản, kiến trúc này có thể tỏ ra quá phức tạp, với rủi ro "kỹ thuật quá mức (over-engineering)". Ngoài ra, là một framework tương đối mới, hệ sinh thái và tài nguyên cộng đồng của nó vẫn cần được cải thiện thêm. Do đó, AgentScope phù hợp hơn để xây dựng các hệ thống đa agent cấp production quy mô lớn, độ tin cậy cao, trong khi đối với phát triển prototype nhanh hoặc các kịch bản ứng dụng đơn giản, việc chọn một framework nhẹ hơn có thể phù hợp hơn.



## 6.4 Framework Thứ ba: CAMEL

Không giống các framework toàn diện như AutoGen và AgentScope, mục tiêu cốt lõi ban đầu của CAMEL là khám phá cách để hai agent có thể tự chủ cộng tác giải quyết các nhiệm vụ phức tạp thông qua "nhập vai" với sự can thiệp tối thiểu của con người.

### 6.4.1 Cộng tác Tự chủ trong CAMEL

Nền tảng của sự cộng tác tự chủ của CAMEL là hai khái niệm cốt lõi: **Nhập vai (Role-Playing)** và **Inception Prompting**.

(1) Nhập vai (Role-Playing)

Trong thiết kế ban đầu của CAMEL, một nhiệm vụ thường được hoàn thành bởi hai agent cộng tác. Hai agent này được gán các "vai trò" bổ trợ cho nhau, được định nghĩa rõ ràng. Một agent đóng vai **"AI User"**, chịu trách nhiệm đề xuất yêu cầu, đưa ra chỉ dẫn, và hình thành các bước nhiệm vụ; agent còn lại đóng vai **"AI Assistant"**, chịu trách nhiệm thực thi các thao tác cụ thể và cung cấp giải pháp dựa trên các chỉ dẫn.

Ví dụ, trong một nhiệm vụ "phát triển một công cụ phân tích chiến lược giao dịch chứng khoán":

- Vai trò **AI User** có thể là một "nhà giao dịch chứng khoán kỳ cựu". Anh ta hiểu thị trường và các chiến lược nhưng không hiểu về lập trình.
- Vai trò **AI Assistant** là một "lập trình viên Python xuất sắc". Anh ta thành thạo lập trình nhưng không biết gì về giao dịch chứng khoán.

Thông qua thiết lập này, quá trình giải quyết nhiệm vụ được biến đổi một cách tự nhiên thành một cuộc hội thoại giữa hai "chuyên gia liên ngành". Nhà giao dịch đề xuất các yêu cầu chuyên môn, lập trình viên biến chúng thành việc triển khai code, và cả hai cộng tác để hoàn thành các nhiệm vụ phức tạp mà không ai trong số họ có thể tự mình hoàn thành.

(2) Inception Prompting

Việc chỉ thiết lập vai trò là chưa đủ. Làm thế nào để đảm bảo hai AI luôn có thể "giữ đúng vai trò" và tiến tới một mục tiêu chung một cách hiệu quả mà không cần sự giám sát liên tục của con người? Đây là lúc công nghệ cốt lõi của CAMEL, inception prompting, phát huy tác dụng. "Inception prompting" là một chỉ dẫn ban đầu (System Prompt) có cấu trúc, được thiết kế cẩn thận, được tiêm vào cả hai agent trước khi cuộc hội thoại bắt đầu. Chỉ dẫn này giống như một "chương trình hành động" được cấy vào các agent, và nó thường bao gồm các phần then chốt sau:

- **Làm rõ vai trò của chính mình**: Ví dụ, "Bạn là một nhà giao dịch chứng khoán kỳ cựu..."
- **Thông báo vai trò của người cộng tác**: Ví dụ, "Bạn đang làm việc với một lập trình viên Python xuất sắc..."
- **Định nghĩa mục tiêu chung**: Ví dụ, "Mục tiêu chung của các bạn là phát triển một công cụ phân tích chiến lược giao dịch chứng khoán."
- **Thiết lập các ràng buộc hành vi và giao thức giao tiếp**: Đây là phần then chốt nhất. Ví dụ, chỉ dẫn sẽ yêu cầu AI user "chỉ đề xuất một bước rõ ràng, cụ thể tại một thời điểm" và yêu cầu AI assistant "không hỏi thêm chi tiết trước khi hoàn thành bước trước đó", đồng thời quy định cả hai bên cần sử dụng các dấu hiệu cụ thể (chẳng hạn như `<SOLUTION>`) ở cuối câu trả lời để nhận biết việc hoàn thành nhiệm vụ.

Những ràng buộc này đảm bảo rằng cuộc hội thoại không lệch khỏi chủ đề hoặc rơi vào các vòng lặp không hiệu quả mà tiến triển theo một cách có cấu trúc cao, được điều khiển bởi nhiệm vụ, như minh họa trong Hình 6.3.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/6-figures/04.png" alt="" width="90%"/>
  <p>Hình 6.3 CAMEL Tạo Robot Giao dịch Chứng khoán</p>
</div>

Trong phần tiếp theo, chúng ta sẽ trải nghiệm quá trình này thông qua một ví dụ cụ thể.

### 6.4.2 Sách điện tử Phổ biến Khoa học bằng AI

Để hiểu khả năng nhập vai của framework CAMEL, chúng ta sẽ xây dựng một case cộng tác thực hành: cho một nhà tâm lý học AI làm việc cùng một tác giả AI để đồng sáng tạo một cuốn sách điện tử ngắn về "Tâm lý học của Sự trì hoãn (The Psychology of Procrastination)". Case này thể hiện ưu điểm cốt lõi của CAMEL trong việc cho phép hai agent tận dụng các lĩnh vực chuyên môn tương ứng của họ để cùng hoàn thành các nhiệm vụ sáng tạo phức tạp mà một agent đơn lẻ sẽ khó thực hiện.

(1) Thiết lập Nhiệm vụ

**Thiết lập Kịch bản**: Tạo một cuốn sách điện tử phổ biến khoa học về tâm lý học của sự trì hoãn cho độc giả phổ thông, đòi hỏi vừa có tính chặt chẽ khoa học vừa có khả năng đọc tốt.

**Vai trò Agent**:

- **Nhà tâm lý học (Psychologist)**: Sở hữu nền tảng lý thuyết sâu sắc về tâm lý học, quen thuộc với khoa học hành vi nhận thức, khoa học thần kinh và các lĩnh vực liên quan khác, có khả năng cung cấp những hiểu biết học thuật chuyên môn và sự hỗ trợ từ nghiên cứu thực nghiệm
- **Tác giả (Writer)**: Có kỹ năng viết xuất sắc và khả năng kể chuyện, giỏi biến các khái niệm học thuật phức tạp thành văn bản sinh động và dễ hiểu, tập trung vào trải nghiệm của độc giả và khả năng đọc của nội dung

(2) Định nghĩa Nhiệm vụ Cộng tác

Trước tiên, chúng ta cần làm rõ mục tiêu chung của hai chuyên gia AI. Chúng ta định nghĩa nhiệm vụ này thông qua một chuỗi (string) chi tiết `task_prompt`.

```python
from colorama import Fore
from camel.societies import RolePlaying
from camel.utils import print_text_animated
from camel.models import ModelFactory
from camel.types import ModelPlatformType
from dotenv import load_dotenv
import os

load_dotenv()
LLM_API_KEY = os.getenv("LLM_API_KEY")
LLM_BASE_URL = os.getenv("LLM_BASE_URL")
LLM_MODEL = os.getenv("LLM_MODEL")

# Create model, using Qwen as an example, calling Alibaba Cloud Bailian platform API
model = ModelFactory.create(
    model_platform=ModelPlatformType.QWEN,
    model_type=LLM_MODEL,
    url=LLM_BASE_URL,
    api_key=LLM_API_KEY
)

# Define collaboration task
task_prompt = """
Create a short e-book on "The Psychology of Procrastination" for general readers interested in psychology.
Requirements:
1. Content should be scientifically rigorous, based on empirical research
2. Language should be easy to understand, avoiding excessive professional terminology
3. Include practical improvement suggestions and case analysis
4. Length controlled at 8000-10000 words
5. Clear structure, including introduction, core chapters, and summary
"""

print(Fore.YELLOW + f"Collaboration task:\n{task_prompt}\n")
```

`task_prompt` là "đặc tả nhiệm vụ" cho toàn bộ quá trình cộng tác. Nó không chỉ là mục tiêu chúng ta muốn đạt được mà còn được CAMEL sử dụng ở hậu trường để tạo ra "inception prompt", đảm bảo rằng cuộc hội thoại giữa hai agent luôn xoay quanh mục tiêu cốt lõi này.

(3) Khởi tạo "Xã hội" Nhập vai

Tiếp theo, chúng ta tạo một instance phiên `RolePlaying`. Đây là thao tác cốt lõi của CAMEL, giúp xây dựng nhanh chóng một "xã hội" cộng tác hai agent dựa trên các vai trò và nhiệm vụ mà chúng ta cung cấp.

```python
# Initialize role-playing session
# AI writer as "user", responsible for proposing writing structure and requirements
# AI psychologist as "assistant", responsible for providing professional knowledge and content
role_play_session = RolePlaying(
    assistant_role_name="Psychologist",
    user_role_name="Writer",
    task_prompt=task_prompt,
    model=model,
    with_task_specify=False, # In this example, we directly use the given task_prompt
)

print(Fore.CYAN + f"Specific task description:\n{role_play_session.task_prompt}\n")
```

`RolePlaying` là một API cấp cao do CAMEL cung cấp, đóng gói kỹ thuật prompt phức tạp. Chúng ta chỉ cần truyền vào tên của hai vai trò và nhiệm vụ. Trong thiết kế của CAMEL, vai trò `user` là "người dẫn dắt" và "người đưa ra yêu cầu" của cuộc hội thoại, trong khi vai trò `assistant` là "người thực thi" và "người cung cấp giải pháp". Do đó, chúng ta gán "tác giả" chịu trách nhiệm lập kế hoạch cấu trúc cho `user_role_name` và "nhà tâm lý học" chịu trách nhiệm cung cấp kiến thức chuyên môn cho `assistant_role_name`.

(4) Bắt đầu và Chạy Cuộc hội thoại Tự động

Cuối cùng, chúng ta viết một vòng lặp để điều khiển toàn bộ quá trình hội thoại, cho phép hai chuyên gia AI bắt đầu quá trình cộng tác tự động của họ.

```python
# Start collaboration conversation
chat_turn_limit, n = 30, 0
# Call init_chat() to get the initial conversation message generated by AI
input_msg = role_play_session.init_chat()

while n < chat_turn_limit:
    n += 1
    # step() method drives a complete round of conversation, AI user and AI assistant each speak once
    assistant_response, user_response = role_play_session.step(input_msg)

    # Check if messages are returned to prevent premature conversation termination
    if assistant_response.msg is None or user_response.msg is None:
        break

    print_text_animated(Fore.BLUE + f"Writer (AI User):\n\n{user_response.msg.content}\n")
    print_text_animated(Fore.GREEN + f"Psychologist (AI Assistant):\n\n{assistant_response.msg.content}\n")

    # Check task completion flag
    if "<CAMEL_TASK_DONE>" in user_response.msg.content or "<CAMEL_TASK_DONE>" in assistant_response.msg.content:
        print(Fore.MAGENTA + "✅ E-book creation completed!")
        break

    # Use assistant's reply as input for next round of conversation
    input_msg = assistant_response.msg

print(Fore.YELLOW + f"Total of {n} rounds of collaborative conversation")
```

Vòng lặp `while` này là cốt lõi của cộng tác tự động. Cuộc hội thoại được tự động khởi tạo bởi phương thức `init_chat()` dựa trên nhiệm vụ và vai trò, mà không cần viết thủ công lời mở đầu. Mỗi bước của vòng lặp điều khiển một lượt tương tác hoàn chỉnh bằng cách gọi `step()` (tác giả đề xuất yêu cầu, nhà tâm lý học cung cấp nội dung), và sử dụng đầu ra của nhà tâm lý học từ lượt trước làm đầu vào cho lượt tiếp theo, tạo thành một chuỗi sáng tạo. Toàn bộ quá trình sẽ tiếp tục cho đến khi đạt giới hạn số lượt hội thoại định trước, hoặc tự động kết thúc sau khi một trong hai agent xuất ra dấu hiệu hoàn thành nhiệm vụ `<CAMEL_TASK_DONE>`.

(5) Minh họa Quá trình Cộng tác

Khi thực thi code trên, chúng ta không chỉ nhận được một chuỗi dài các câu hỏi-đáp đơn điệu mà có thể quan sát một quá trình cộng tác có cấu trúc cao, giống như một nhóm chuyên gia người, tự động tiến hành. Toàn bộ quá trình sáng tạo tự nhiên chia thành nhiều giai đoạn:

**Giai đoạn 1 (khoảng lượt 1-5): Xây dựng Khung sườn và Thống nhất Mục tiêu** Trong giai đoạn đầu của cuộc hội thoại, agent "tác giả" đóng vai trò dẫn dắt trước, đề xuất các ý tưởng ban đầu về cấu trúc tổng thể và sắp xếp chương của cuốn sách điện tử. Sau đó, "nhà tâm lý học" xem xét và bổ sung khung này từ góc nhìn chuyên môn, đảm bảo các module học thuật cốt lõi (chẳng hạn như nền tảng lý thuyết, các khái niệm then chốt, v.v.) không bị bỏ sót, từ đó đạt được sự đồng thuận về đầu ra cuối cùng ngay từ đầu quá trình cộng tác.

**Giai đoạn 2 (khoảng lượt 6-20): Tạo Nội dung Cốt lõi và Chuyển dịch Kiến thức** Đây là giai đoạn sáng tạo nội dung hiệu quả nhất. Chế độ cộng tác trở thành một vòng lặp "yêu cầu-phản hồi" ổn định:

- **Nhà tâm lý học**: Chịu trách nhiệm cung cấp kiến thức chuyên môn "cứng", chẳng hạn như giải thích khoa học về các khái niệm cốt lõi như "lý thuyết chiết khấu thời gian (temporal discounting theory)" và "thiếu hụt chức năng điều hành (executive function deficits)", và trích dẫn các nghiên cứu thực nghiệm liên quan để hỗ trợ luận điểm.
- **Tác giả**: Đóng vai trò "người phiên dịch", biến những khái niệm học thuật chặt chẽ nhưng có thể khó hiểu này thành những phép ẩn dụ sinh động, hình tượng và các trường hợp liên quan đến đời sống. Ví dụ, nó có thể so sánh khái niệm "thiên kiến hiện tại trong não bộ (present bias)" với "một đứa trẻ bướng bỉnh chỉ quan tâm đến viên kẹo trước mắt mà không màng đến sức khỏe lâu dài".

**Giai đoạn 3 (khoảng lượt 21-25): Tối ưu Lặp lại và Đảm bảo Chất lượng** Khi nội dung chính của cuốn sách được hoàn thành, trọng tâm của cuộc hội thoại chuyển sang trau chuốt và cải thiện văn bản hiện có. Lúc này, vai trò của hai agent có những thay đổi tinh tế:

- **Tác giả**: Tập trung hơn vào việc xem xét sự mạch lạc tổng thể, tính logic và phong cách ngôn ngữ của bài viết, đề xuất các gợi ý sửa đổi từ góc nhìn "trải nghiệm của độc giả".
- **Nhà tâm lý học**: Một lần nữa đóng vai trò "người kiểm tra thực tế (fact checker)", đảm bảo rằng độ chính xác khoa học của kiến thức cốt lõi không bị mất trong quá trình chuyển dịch và trau chuốt, và bổ sung cho một số luận điểm bằng sự hỗ trợ từ nghiên cứu thực nghiệm mạnh mẽ hơn.

**Giai đoạn 4 (Kết luận): Tóm tắt và Nâng tầm** Trong vài lượt hội thoại cuối cùng, cả hai bên cộng tác để hoàn thành phần tóm tắt các gợi ý thực tiễn và rà soát toàn bộ cuốn sách, đảm bảo rằng cuốn sách điện tử có một kết thúc rõ ràng, mạnh mẽ, để lại ấn tượng sâu sắc cho độc giả và cung cấp giá trị thiết thực.

```
Collaboration task:
Create a short e-book on "The Psychology of Procrastination" for general readers interested in psychology.
Requirements:
1. Content should be scientifically rigorous, based on empirical research
2. Language should be easy to understand, avoiding excessive professional terminology
3. Include practical improvement suggestions and case analysis
4. Length controlled at 8000-10000 words
5. Clear structure, including introduction, core chapters, and summary

Specific task description:
Write an 8000–10000 word short e-book "The Psychology of Procrastination" for general readers: empirically based, easy to understand. Structure: introduction, causes (cognitive/emotional/reward), motivation and decision-making, habit formation and intervention, practical strategies and exercises, three case analyses, summary and resources. Each chapter contains research citations and actionable steps.

Writer:
Instruction: Please write a 400–600 word Chinese draft for the "Introduction" chapter of the e-book...
Input: None

Psychologist:
Solution:
Draft: Procrastination refers to the behavior and internal tendency of repeatedly postponing or avoiding a task despite knowing it should be completed. It can be an occasional time management problem...

Next request.

Writer:
Instruction: Please revise the following introduction draft into a 450–550 word Chinese text...
Input: Draft: Procrastination refers to the behavior and internal tendency of repeatedly postponing or avoiding a task...
.....
```

### 6.4.3 Phân tích Ưu điểm và Hạn chế của CAMEL

Thông qua case sáng tạo sách điện tử ở trên, chúng ta đã trải nghiệm sâu sắc mô hình nhập vai độc đáo của framework CAMEL. Bây giờ hãy phân tích một cách khách quan các ưu điểm và hạn chế của triết lý thiết kế này để đưa ra những lựa chọn kỹ thuật khôn ngoan trong các dự án thực tế.

(1) Ưu điểm

Ưu điểm lớn nhất của CAMEL nằm ở triết lý thiết kế "kiến trúc nhẹ, prompt nặng". So với việc quản lý hội thoại phức tạp của AutoGen và kiến trúc phân tán của AgentScope, CAMEL có thể đạt được sự cộng tác agent chất lượng cao thông qua các prompt ban đầu được thiết kế cẩn thận. Hành vi cộng tác nổi lên (emergent) một cách tự nhiên này thường linh hoạt và hiệu quả hơn các luồng công việc được lập trình cứng (hard-coded).

Đáng chú ý là framework CAMEL đang trải qua quá trình phát triển và tiến hóa nhanh chóng. Từ [kho GitHub](https://github.com/camel-ai/camel) của nó, chúng ta có thể thấy rằng CAMEL không chỉ là một framework cộng tác hai agent đơn giản mà hiện đã có:

- **Khả năng Đa phương thức (Multimodal Capabilities)**: Hỗ trợ cộng tác agent trong nhiều phương thức như văn bản, hình ảnh và âm thanh
- **Tích hợp Công cụ (Tool Integration)**: Tích hợp sẵn thư viện công cụ phong phú, bao gồm tìm kiếm, tính toán, thực thi code, v.v.
- **Thích ứng Mô hình (Model Adaptation)**: Hỗ trợ nhiều backend LLM như OpenAI, Anthropic, Google và các mô hình mã nguồn mở
- **Liên kết Hệ sinh thái (Ecosystem Linkage)**: Đạt được khả năng tương tác với các framework chủ đạo như LangChain, CrewAI và AutoGen

(2) Các Hạn chế Chính

1. Phụ thuộc Cao vào Kỹ thuật Prompt (Prompt Engineering)

Sự thành công của CAMEL phần lớn phụ thuộc vào chất lượng của các prompt ban đầu. Điều này mang lại một số thách thức:

- **Ngưỡng Thiết kế Prompt**: Đòi hỏi sự hiểu biết sâu sắc về lĩnh vực mục tiêu và các đặc điểm hành vi của LLM
- **Độ phức tạp khi Gỡ lỗi**: Khi việc cộng tác không hiệu quả, rất khó để xác định chính xác vấn đề nằm ở định nghĩa vai trò, mô tả nhiệm vụ, hay các quy tắc tương tác
- **Thách thức về Tính nhất quán**: Các LLM khác nhau có thể có cách hiểu khác nhau về cùng một prompt

2. Hạn chế về Quy mô Cộng tác

Mặc dù CAMEL thể hiện xuất sắc trong cộng tác hai agent, nó gặp phải các thách thức khi xử lý các kịch bản đa agent quy mô lớn:

- **Quản lý Hội thoại**: Thiếu các cơ chế định tuyến hội thoại phức tạp như AutoGen
- **Đồng bộ Trạng thái**: Không có khả năng quản lý trạng thái phân tán như AgentScope
- **Giải quyết Xung đột**: Thiếu các cơ chế phân xử hiệu quả khi nhiều agent bất đồng ý kiến

3. Ranh giới Tính phù hợp của Nhiệm vụ

CAMEL đặc biệt phù hợp cho các nhiệm vụ đòi hỏi cộng tác sâu và tư duy sáng tạo, nhưng có thể không phải là lựa chọn tối ưu trong một số kịch bản nhất định:

- **Kiểm soát Quy trình Nghiêm ngặt**: Đối với các nhiệm vụ đòi hỏi kiểm soát bước một cách chính xác, cấu trúc đồ thị của LangGraph phù hợp hơn
- **Đồng thời Quy mô lớn**: Kiến trúc hướng thông điệp của AgentScope có nhiều ưu điểm hơn trong các kịch bản đồng thời cao
- **Cây Quyết định Phức tạp**: Chế độ group chat của AutoGen linh hoạt hơn trong các kịch bản ra quyết định đa bên

Nhìn chung, CAMEL đại diện cho một mô hình cộng tác đa agent độc đáo và tinh tế. Thông qua thiết kế nhập vai "lấy con người làm trung tâm", nó biến các bài toán kỹ thuật hệ thống phức tạp thành các mô hình cộng tác giữa người với người trực quan. Khi hệ sinh thái của nó tiếp tục được cải thiện và các chức năng tiếp tục được mở rộng, CAMEL đang trở thành một trong những lựa chọn quan trọng để xây dựng các hệ thống cộng tác thông minh.

## 6.5 Framework Thứ tư: LangGraph

### 6.5.1 Tổng quan Cấu trúc LangGraph

LangGraph, với tư cách là một phần mở rộng quan trọng của hệ sinh thái LangChain, đại diện cho một hướng đi hoàn toàn mới trong thiết kế agent framework. Không giống các framework dựa trên "hội thoại" đã được giới thiệu trước đó (chẳng hạn như AutoGen và CAMEL), LangGraph mô hình hóa luồng thực thi của agent thành một **Máy Trạng thái (State Machine)** và biểu diễn nó dưới dạng một **Đồ thị Có hướng (Directed Graph)**. Trong mô hình này, các **Nút (Nodes)** của đồ thị đại diện cho các bước tính toán cụ thể (chẳng hạn như gọi LLM, thực thi công cụ), trong khi các **Cạnh (Edges)** định nghĩa logic chuyển tiếp từ nút này sang nút khác. Khía cạnh mang tính cách mạng của thiết kế này là nó hỗ trợ nguyên bản các vòng lặp, khiến việc xây dựng các luồng công việc agent phức tạp có khả năng lặp lại, phản ánh và tự sửa lỗi trở nên trực quan và đơn giản một cách chưa từng có.

Để hiểu LangGraph, trước tiên chúng ta cần nắm được ba thành phần cơ bản của nó.

**Thứ nhất, là trạng thái toàn cục (State)**. Toàn bộ quá trình thực thi của đồ thị xoay quanh một đối tượng trạng thái được chia sẻ. Trạng thái này thường được định nghĩa dưới dạng một `TypedDict` của Python, có thể chứa bất kỳ thông tin nào bạn cần theo dõi, chẳng hạn như lịch sử hội thoại, kết quả trung gian, số lần lặp, v.v. Tất cả các nút đều có thể đọc và cập nhật trạng thái trung tâm này.

```python
from typing import TypedDict, List

# Define global state data structure
class AgentState(TypedDict):
    messages: List[str]      # Conversation history
    current_task: str        # Current task
    final_answer: str        # Final answer
    # ... any other state to track
```

**Thứ hai, là các nút (Nodes)**. Mỗi nút là một hàm Python nhận trạng thái hiện tại làm đầu vào và trả về một trạng thái đã cập nhật làm đầu ra. Các nút là những đơn vị thực hiện công việc cụ thể.

```python
# Define a "planner" node function
def planner_node(state: AgentState) -> AgentState:
    """Formulate a plan based on current task and update state."""
    current_task = state["current_task"]
    # ... call LLM to generate plan ...
    plan = f"Plan generated for task '{current_task}'..."

    # Append new message to state
    state["messages"].append(plan)
    return state

# Define an "executor" node function
def executor_node(state: AgentState) -> AgentState:
    """Execute latest plan and update state."""
    latest_plan = state["messages"][-1]
    # ... execute plan and get result ...
    result = f"Result of executing plan '{latest_plan}'..."

    state["messages"].append(result)
    return state
```

**Cuối cùng, là các cạnh (Edges)**. Các cạnh chịu trách nhiệm kết nối các nút và định nghĩa hướng của luồng công việc. Cạnh đơn giản nhất là một cạnh thông thường, nó chỉ định rằng đầu ra của một nút luôn chảy đến một nút cố định khác. Tính năng mạnh mẽ nhất của LangGraph nằm ở **Cạnh Có điều kiện (Conditional Edges)**. Nó sử dụng một hàm để đánh giá trạng thái hiện tại và sau đó động (dynamically) quyết định nút nào sẽ nhảy đến tiếp theo. Đây là chìa khóa để triển khai các vòng lặp và các nhánh logic phức tạp.

```python
def should_continue(state: AgentState) -> str:
    """Condition function: decide next route based on state."""
    # Assume if messages are less than 3, need to continue planning
    if len(state["messages"]) < 3:
        # Returned string needs to match the key defined when adding conditional edge
        return "continue_to_planner"
    else:
        state["final_answer"] = state["messages"][-1]
        return "end_workflow"
```

Sau khi định nghĩa trạng thái, nút và cạnh, chúng ta có thể lắp ráp chúng thành một luồng công việc có thể thực thi giống như xếp hình.

```python
from langgraph.graph import StateGraph, END

# Initialize a state graph and bind our defined state structure
workflow = StateGraph(AgentState)

# Add node functions to the graph
workflow.add_node("planner", planner_node)
workflow.add_node("executor", executor_node)

# Set graph entry point
workflow.set_entry_point("planner")

# Add regular edge, connecting planner and executor
workflow.add_edge("planner", "executor")

# Add conditional edge, implementing dynamic routing
workflow.add_conditional_edges(
    # Starting node
    "executor",
    # Judgment function
    should_continue,
    # Route mapping: map judgment function's return value to target node
    {
        "continue_to_planner": "planner", # If returns "continue_to_planner", jump back to planner node
        "end_workflow": END               # If returns "end_workflow", end process
    }
)

# Compile graph, generate executable application
app = workflow.compile()

# Run graph
inputs = {"current_task": "Analyze recent AI industry news", "messages": []}
for event in app.stream(inputs):
    print(event)
```

### 6.5.2 Trợ lý Hỏi đáp Ba bước
Sau khi hiểu các khái niệm cốt lõi của LangGraph, chúng ta sẽ củng cố những gì đã học thông qua một case thực hành. Chúng ta sẽ xây dựng một trợ lý hội thoại hỏi đáp đơn giản hóa, tuân theo một quy trình ba bước rõ ràng, cố định để trả lời các câu hỏi của người dùng:

1. **Hiểu (Understand)**: Trước tiên, phân tích ý định truy vấn của người dùng.
2. **Tìm kiếm (Search)**: Sau đó, mô phỏng việc tìm kiếm thông tin liên quan đến ý định.
3. **Trả lời (Answer)**: Cuối cùng, tạo ra câu trả lời cuối cùng dựa trên ý định và thông tin đã tìm kiếm.

Case này sẽ minh họa rõ ràng cách định nghĩa trạng thái, tạo các nút, và kết nối chúng một cách tuyến tính thành một luồng công việc hoàn chỉnh. Chúng ta sẽ chia nhỏ code thành bốn bước cốt lõi: định nghĩa trạng thái, tạo nút, xây dựng đồ thị, và chạy ứng dụng.

(1) Định nghĩa Trạng thái Toàn cục

Trước tiên, chúng ta cần định nghĩa một trạng thái toàn cục chạy xuyên suốt toàn bộ luồng công việc. **Đây là một cấu trúc dữ liệu được chia sẻ, được truyền giữa mỗi nút của đồ thị, đóng vai trò là ngữ cảnh bền vững của luồng công việc.** Mỗi nút có thể đọc dữ liệu từ cấu trúc này và cập nhật nó.

```python
from typing import TypedDict, Annotated
from langgraph.graph.message import add_messages

class SearchState(TypedDict):
    messages: Annotated[list, add_messages]
    user_query: str      # User requirement summary after LLM understanding
    search_query: str    # Optimized search query for Tavily API
    search_results: str  # Results returned by Tavily search
    final_answer: str    # Final generated answer
    step: str            # Mark current step
```

Chúng ta đã tạo `SearchState` `TypedDict`, định nghĩa một schema dữ liệu rõ ràng cho đối tượng trạng thái. Một thiết kế then chốt là việc bao gồm cả hai trường `user_query` và `search_query`. Điều này cho phép agent trước tiên tối ưu hóa câu hỏi ngôn ngữ tự nhiên của người dùng thành các từ khóa được tinh chỉnh, phù hợp hơn với các công cụ tìm kiếm, từ đó cải thiện đáng kể chất lượng của kết quả tìm kiếm.

(2) Định nghĩa các Nút của Luồng công việc

Sau khi định nghĩa cấu trúc trạng thái, bước tiếp theo là tạo các nút khác nhau cấu thành luồng công việc của chúng ta. Trong LangGraph, mỗi nút là một hàm Python thực hiện một nhiệm vụ cụ thể. Các hàm này nhận đối tượng trạng thái hiện tại làm đầu vào và trả về một dictionary chứa các trường đã được cập nhật.

Trước khi định nghĩa các nút, trước tiên chúng ta hoàn thành việc thiết lập khởi tạo dự án, bao gồm việc tải các biến môi trường và khởi tạo instance của mô hình ngôn ngữ lớn.

```python
import os
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage, AIMessage, SystemMessage
from tavily import TavilyClient

# Load environment variables from .env file
load_dotenv()

# Initialize model
# We will use this llm instance to drive the intelligence of all nodes
llm = ChatOpenAI(
    model=os.getenv("LLM_MODEL_ID", "gpt-4o-mini"),
    api_key=os.getenv("LLM_API_KEY"),
    base_url=os.getenv("LLM_BASE_URL", "https://api.openai.com/v1"),
    temperature=0.7
)
# Initialize Tavily client
tavily_client = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))
```

Bây giờ, hãy tạo ba nút cốt lõi lần lượt từng cái một.

(1) Nút Hiểu và Truy vấn

Nút này là bước đầu tiên của luồng công việc. Trách nhiệm của nó là hiểu ý định của người dùng và tạo ra một truy vấn tìm kiếm được tối ưu hóa cho ý định đó.

```python
def understand_query_node(state: SearchState) -> dict:
    """Step 1: Understand user query and generate search keywords"""
    user_message = state["messages"][-1].content

    understand_prompt = f"""Analyze the user's query: "{user_message}"
Please complete two tasks:
1. Concisely summarize what the user wants to know
2. Generate keywords most suitable for search engines (Chinese or English, must be precise)

Format:
Understanding: [User requirement summary]
Search terms: [Best search keywords]"""

    response = llm.invoke([SystemMessage(content=understand_prompt)])
    response_text = response.content

    # Parse LLM's output, extract search keywords
    search_query = user_message # Default to using original query
    if "Search terms:" in response_text or "搜索词：" in response_text:
        if "Search terms:" in response_text:
            search_query = response_text.split("Search terms:")[1].strip()
        else:
            search_query = response_text.split("搜索词：")[1].strip()

    return {
        "user_query": response_text,
        "search_query": search_query,
        "step": "understood",
        "messages": [AIMessage(content=f"I will search for you: {search_query}")]
    }
```

Nút này sử dụng một prompt có cấu trúc để yêu cầu LLM đồng thời hoàn thành hai nhiệm vụ: "hiểu ý định" và "tạo từ khóa", và cập nhật các từ khóa tìm kiếm chuyên dụng đã được phân tích vào trường `search_query` của trạng thái, chuẩn bị cho bước tìm kiếm chính xác tiếp theo.

(2) Nút Tìm kiếm

Nút này chịu trách nhiệm thực thi khả năng "sử dụng công cụ" của agent. Nó sẽ gọi Tavily API để tìm kiếm internet thực sự và có chức năng xử lý lỗi cơ bản.

```python
def tavily_search_node(state: SearchState) -> dict:
    """Step 2: Use Tavily API for real search"""
    search_query = state["search_query"]
    try:
        print(f"🔍 Searching: {search_query}")
        response = tavily_client.search(
            query=search_query, search_depth="basic", max_results=5, include_answer=True
        )
        # ... (process and format search results) ...
        search_results = ... # Formatted result string

        return {
            "search_results": search_results,
            "step": "searched",
            "messages": [AIMessage(content="✅ Search completed! Organizing answer...")]
        }
    except Exception as e:
        # ... (handle error) ...
        return {
            "search_results": f"Search failed: {e}",
            "step": "search_failed",
            "messages": [AIMessage(content="❌ Search encountered a problem...")]
        }
```

Nút này khởi tạo một lệnh gọi API thực sự thông qua `tavily_client.search`. Nó được bao bọc trong một khối `try...except` để bắt các ngoại lệ có thể xảy ra. Nếu tìm kiếm thất bại, nó cập nhật trạng thái `step` thành `"search_failed"`, giá trị này sẽ được nút tiếp theo sử dụng để kích hoạt một phương án dự phòng (fallback).

(3) Nút Trả lời

Nút trả lời cuối cùng có thể chọn các chiến lược trả lời khác nhau tùy thuộc vào việc tìm kiếm trước đó có thành công hay không, sở hữu một mức độ linh hoạt nhất định.

```python
def generate_answer_node(state: SearchState) -> dict:
    """Step 3: Generate final answer based on search results"""
    if state["step"] == "search_failed":
        # If search failed, execute fallback strategy, answer based on LLM's own knowledge
        fallback_prompt = f"Search API is temporarily unavailable, please answer the user's question based on your knowledge:\nUser question: {state['user_query']}"
        response = llm.invoke([SystemMessage(content=fallback_prompt)])
    else:
        # Search successful, generate answer based on search results
        answer_prompt = f"""Provide a complete and accurate answer to the user based on the following search results:
User question: {state['user_query']}
Search results:\n{state['search_results']}
Please synthesize the search results and provide an accurate, useful answer..."""
        response = llm.invoke([SystemMessage(content=answer_prompt)])

    return {
        "final_answer": response.content,
        "step": "completed",
        "messages": [AIMessage(content=response.content)]
    }
```

Nút này thực thi logic có điều kiện bằng cách kiểm tra giá trị của `state["step"]`. Nếu tìm kiếm thất bại, nó sẽ sử dụng kiến thức nội tại của LLM để trả lời và thông báo cho người dùng về tình huống. Nếu tìm kiếm thành công, nó sẽ sử dụng một prompt chứa các kết quả tìm kiếm thời gian thực để tạo ra một câu trả lời kịp thời và có căn cứ.

(4) Xây dựng Đồ thị

Chúng ta kết nối tất cả các nút lại với nhau.

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import InMemorySaver

def create_search_assistant():
    workflow = StateGraph(SearchState)

    # Add nodes
    workflow.add_node("understand", understand_query_node)
    workflow.add_node("search", tavily_search_node)
    workflow.add_node("answer", generate_answer_node)

    # Set linear process
    workflow.add_edge(START, "understand")
    workflow.add_edge("understand", "search")
    workflow.add_edge("search", "answer")
    workflow.add_edge("answer", END)

    # Compile graph
    memory = InMemorySaver()
    app = workflow.compile(checkpointer=memory)
    return app
```

(5) Minh họa Chạy Case

Sau khi chạy script này, bạn có thể đặt một số câu hỏi đòi hỏi thông tin thời gian thực, chẳng hạn như case trong chương đầu tiên của chúng ta: `I'm going to Beijing tomorrow, what's the weather like? Are there suitable attractions?`

Bạn sẽ thấy terminal hiển thị rõ ràng quá trình "suy nghĩ" của agent:

```
🔍 Intelligent Search Assistant Started!
I will use Tavily API to search for the latest and most accurate information for you
Supports various questions: news, technology, knowledge Q&A, etc.
(Enter 'quit' to exit)

🤔 What would you like to know: I'm going to Beijing tomorrow, what's the weather like? Are there suitable attractions?

============================================================
🧠 Understanding phase: I understand your needs: Understanding: The user wants to know about tomorrow's weather in Beijing and suitable attraction recommendations.
Search terms: Beijing tomorrow weather attraction recommendations Beijing weather tomorrow attractions
🔍 Searching: Beijing tomorrow weather attraction recommendations Beijing weather tomorrow attractions
🔍 Search phase: ✅ Search completed! Found relevant information, organizing answer for you...

💡 Final Answer:
Tomorrow (September 17, 2025) Beijing's weather forecast shows it is expected to be cloudy, with temperatures ranging from 17°C (62°F) to 25°C (77°F). This mild weather is very suitable for outdoor activities.

### Suitable Attraction Recommendations:
1. **Great Wall**: As one of China's most famous historical sites, the Great Wall is a must-visit. You can choose popular sections like Badaling or Mutianyu for your tour.

2. **Forbidden City**: The Forbidden City was the imperial palace of the Ming and Qing dynasties, with rich history and culture, suitable for tourists interested in Chinese history.

3. **Tiananmen Square**: This is one of China's symbols, with many important buildings and monuments on the square, suitable for taking photos.

4. **Summer Palace**: A very beautiful royal garden, suitable for strolling and enjoying natural scenery, especially the lakes and ancient buildings.

5. **798 Art District**: If you're interested in modern art, the 798 Art District is a place that integrates art, culture, and creativity, suitable for exploration and photography.

### Tips:
- Since tomorrow's weather is good, it's recommended to plan your travel route in advance and prepare some water and snacks to maintain sufficient energy during the tour.
- Since weather changes may affect the tour experience, it's recommended to check real-time weather updates.

Hope this information helps you arrange a pleasant Beijing trip! If you need more information about attractions or travel advice, feel free to ask anytime.

============================================================

🤔 What would you like to know:
```

Và đây là một trợ lý tương tác liên tục, bạn có thể tiếp tục đặt câu hỏi.

### 6.5.3 Phân tích Ưu điểm và Hạn chế của LangGraph

Bất kỳ framework kỹ thuật nào cũng có các kịch bản áp dụng cụ thể và những đánh đổi trong thiết kế. Trong phần này, chúng ta sẽ phân tích một cách khách quan các ưu điểm cốt lõi của LangGraph cùng những hạn chế mà nó có thể gặp phải trong các ứng dụng thực tế.

(1) Ưu điểm

- Như thể hiện trong case trợ lý tìm kiếm thông minh của chúng ta, LangGraph định nghĩa một cách tường minh toàn bộ quy trình hỏi đáp thời gian thực thành một "sơ đồ luồng (flowchart)" gồm các trạng thái, nút và cạnh. Ưu điểm lớn nhất của thiết kế này là **khả năng kiểm soát và dự đoán cao**. Nhà phát triển có thể lập kế hoạch chính xác cho từng bước hành vi của agent, điều này rất quan trọng để xây dựng các ứng dụng cấp production đòi hỏi độ tin cậy và khả năng kiểm toán cao. Tính năng mạnh mẽ nhất của nó nằm ở **hỗ trợ nguyên bản các vòng lặp**. Thông qua các cạnh có điều kiện, chúng ta có thể dễ dàng xây dựng các vòng lặp "phản ánh-sửa chữa". Ví dụ, trong case của chúng ta, nếu tìm kiếm thất bại, chúng ta có thể thiết kế một đường dẫn để quay về phương án dự phòng. Đây là chìa khóa để xây dựng các agent có khả năng tự tối ưu hóa và chịu lỗi.

- Ngoài ra, vì mỗi nút là một hàm Python độc lập, điều này mang lại **tính module hóa cao**. Đồng thời, việc chèn một nút chờ con người xem xét vào trong quy trình trở nên rất đơn giản, cung cấp một nền tảng vững chắc để triển khai sự cộng tác "Human-in-the-loop" (con người trong vòng lặp) đáng tin cậy.

(2) Hạn chế

- So với các framework dựa trên hội thoại, LangGraph đòi hỏi nhà phát triển viết nhiều **code khuôn mẫu (boilerplate code)** hơn. Việc định nghĩa trạng thái, nút, cạnh và một loạt các thao tác khiến quá trình phát triển trở nên cồng kềnh hơn đối với các nhiệm vụ đơn giản. Nhà phát triển cần suy nghĩ nhiều hơn về "cách kiểm soát quy trình (how)" thay vì chỉ "làm gì (what)". Vì luồng công việc được định nghĩa trước, hành vi của LangGraph có thể kiểm soát được nhưng cũng thiếu **sự tương tác "nổi lên (emergent)"** động của các agent hội thoại. Điểm mạnh của nó nằm ở việc thực thi một quy trình xác định, đáng tin cậy, thay vì mô phỏng sự cộng tác xã hội mở, không thể dự đoán.

- Quá trình gỡ lỗi cũng đặt ra các thách thức. Mặc dù quy trình rõ ràng hơn so với lịch sử hội thoại, các vấn đề có thể xảy ra tại nhiều điểm: lỗi logic bên trong một nút, sự biến đổi trong dữ liệu trạng thái được truyền giữa các nút, hoặc sai sót trong việc đánh giá điều kiện chuyển tiếp của cạnh. Điều này đòi hỏi nhà phát triển phải có sự hiểu biết toàn cục về cơ chế hoạt động của toàn bộ đồ thị.

## 6.6 Tóm tắt Chương

Trong chương này, chúng ta đã trải nghiệm một số agent framework tiên phong nhất thông qua thực hành dưới hình thức các case.

Chúng ta đã thấy rằng mỗi framework có cách tiếp cận riêng để triển khai việc xây dựng agent:

- **AutoGen** trừu tượng hóa sự cộng tác phức tạp thành một "group chat" đa vai trò, được tiến hành tự động, với cốt lõi là "thúc đẩy cộng tác thông qua hội thoại".
- **AgentScope** tập trung vào tính vững chắc và khả năng mở rộng của các ứng dụng cấp công nghiệp, cung cấp một nền tảng kỹ thuật vững chắc để xây dựng các hệ thống đa agent phân tán, đồng thời cao.
- **CAMEL** minh họa cách kích thích sự cộng tác sâu, tự chủ giữa hai agent chuyên gia với lượng code tối thiểu thông qua mô hình "nhập vai" và "inception prompting" nhẹ của nó.
- **LangGraph** quay về một mô hình "máy trạng thái" cơ bản hơn, trao cho nhà phát triển sự kiểm soát chính xác đối với các luồng công việc thông qua các cấu trúc đồ thị tường minh, đặc biệt là khả năng vòng lặp của nó, mở đường cho việc xây dựng các agent có khả năng phản ánh và sửa lỗi.

Thông qua việc phân tích sâu các framework này, chúng ta có thể chắt lọc ra một sự đánh đổi trong thiết kế: **sự lựa chọn giữa "cộng tác nổi lên (emergent collaboration)" và "kiểm soát tường minh (explicit control)"**. AutoGen và CAMEL dựa nhiều hơn vào việc định nghĩa "vai trò" và "mục tiêu" của các agent, cho phép các hành vi cộng tác phức tạp "nổi lên" từ các quy tắc hội thoại đơn giản. Cách tiếp cận này gần với các mô hình tương tác của con người hơn nhưng đôi khi khó dự đoán và gỡ lỗi. LangGraph đòi hỏi nhà phát triển định nghĩa tường minh từng bước và điều kiện chuyển tiếp, hy sinh một số bất ngờ "nổi lên" để đổi lấy độ tin cậy, khả năng kiểm soát và khả năng quan sát cao. Đồng thời, AgentScope tiết lộ một chiều hướng thứ hai cũng quan trọng không kém: **tính kỹ thuật (engineering)**. Bất kể chúng ta chọn mô hình cộng tác nào, để đưa nó từ một prototype thử nghiệm sang một ứng dụng production, chúng ta phải đối mặt với các thách thức kỹ thuật như đồng thời, chịu lỗi và triển khai phân tán. AgentScope ra đời để giải quyết những vấn đề này, đại diện cho bước nhảy vọt then chốt từ "có thể chạy" đến "có thể phục vụ một cách ổn định".

Tóm lại, không chỉ có một cách duy nhất để xây dựng agent. Việc hiểu sâu các triết lý thiết kế framework được khám phá trong chương này có thể khiến chúng ta không chỉ trở thành những "người dùng công cụ" tốt hơn mà còn hiểu được các ưu nhược điểm và những đánh đổi khác nhau trong thiết kế framework.

Trong chương tiếp theo, chúng ta sẽ bước vào nội dung cốt lõi của giáo trình này, xây dựng agent framework của riêng mình từ đầu, tích hợp tất cả lý thuyết và thực hành.


## Bài tập

1. Chương này đã giới thiệu bốn agent framework có đặc trưng riêng biệt: `AutoGen`, `AgentScope`, `CAMEL` và `LangGraph`. Hãy phân tích:

   - Trong Bảng 6.1 của Mục 6.1.2, nhiều chiều của bốn framework này đã được so sánh. Hãy chọn hai framework mà bạn quen thuộc nhất và so sánh chúng sâu hơn từ ba chiều: "chế độ cộng tác", "phương pháp kiểm soát" và "kịch bản áp dụng".
   - Chương này đã đề cập đến sự đánh đổi giữa "cộng tác nổi lên (emergent collaboration)" và "kiểm soát tường minh (explicit control)". Bạn hiểu ý nghĩa của hai triết lý thiết kế này như thế nào?

2. Trong case `AutoGen` ở Mục 6.2, chúng ta đã xây dựng một "nhóm phát triển phần mềm". Hãy mở rộng tư duy dựa trên case này:

   > **Gợi ý**: Đây là một câu hỏi thực hành, khuyến nghị thao tác thực tế

   - Nhóm hiện tại sử dụng chế độ `RoundRobinGroupChat` (group chat luân phiên), nơi các agent phát biểu theo một thứ tự cố định. Nếu yêu cầu thay đổi và code của kỹ sư cần được trả lại cho product manager để xem xét lại, quy trình cộng tác nên được sửa đổi như thế nào? Hãy thiết kế một cơ chế hỗ trợ "quay lui động (dynamic rollback)".
   - Trong case, chúng ta đã định nghĩa vai trò và trách nhiệm của mỗi agent thông qua `System Message`. Hãy thử thêm một vai trò mới "Quality Assurance" vào nhóm này và thiết kế thông điệp hệ thống của nó để nó có thể thực hiện kiểm thử tự động sau khi review code.
   - Sự cộng tác hội thoại của `AutoGen` có sự bất ổn tiềm ẩn, điều này có thể khiến các cuộc hội thoại lệch khỏi chủ đề hoặc rơi vào vòng lặp. Hãy suy nghĩ: Làm thế nào để thiết kế một cơ chế "giám sát chất lượng hội thoại" để can thiệp kịp thời khi phát hiện bất thường?

3. Trong case `AgentScope` ở Mục 6.3, chúng ta đã triển khai một trò chơi "Ma Sói Tam Quốc". Hãy phân tích sâu:

   - Case đã sử dụng `MsgHub` (trung tâm thông điệp) để quản lý giao tiếp giữa các agent. Hãy giải thích kiến trúc hướng thông điệp có những ưu điểm gì so với các lệnh gọi hàm truyền thống? Trong những kịch bản nào thì kiến trúc này đặc biệt có giá trị?
   - Trò chơi đã sử dụng structured output (chẳng hạn như `DiscussionModelCN`, `WitchActionModelCN`) để ràng buộc hành vi của agent. Hãy thiết kế một vai trò trò chơi mới "Thợ săn (Hunter)" và định nghĩa mô hình structured output tương ứng của nó, bao gồm các định nghĩa trường và quy tắc xác thực.
   - `AgentScope` hỗ trợ triển khai phân tán, điều này có nghĩa là các agent khác nhau có thể chạy trên các server khác nhau. Hãy suy nghĩ: Trong một kịch bản trò chơi thời gian thực như "Ma Sói Tam Quốc", việc triển khai phân tán sẽ mang lại những thách thức kỹ thuật nào? Làm thế nào để đảm bảo thứ tự và tính nhất quán của thông điệp?

4. Trong case `CAMEL` ở Mục 6.4, chúng ta đã cho một nhà tâm lý học và một tác giả cộng tác để tạo ra một cuốn sách điện tử.

   - Trong case, sự cộng tác bị buộc phải kết thúc khi phát hiện dấu hiệu `<CAMEL_TASK_DONE>`. Nhưng điều gì sẽ xảy ra nếu hai agent bất đồng (một agent cho rằng có thể kết thúc, một agent cho rằng không nên) và không thể đạt được sự đồng thuận? Hãy thiết kế một cơ chế tương thích "giải quyết xung đột".
   - `CAMEL` ban đầu được thiết kế cho sự cộng tác hai agent nhưng hiện đã được mở rộng để hỗ trợ đa agent. Hãy tham khảo tài liệu mới nhất của `CAMEL` để tìm hiểu về module cộng tác đa agent [`workforce`](https://docs.camel-ai.org/key_modules/workforce) của nó, và giải thích nó khác với chế độ group chat của `AutoGen` như thế nào, kết hợp với sơ đồ kiến trúc.

5. Trong case `LangGraph` ở Mục 6.5, chúng ta đã xây dựng một "trợ lý hỏi đáp ba bước". Hãy phân tích:

   - `LangGraph` mô hình hóa quy trình agent thành một máy trạng thái và đồ thị có hướng. Hãy vẽ cấu trúc đồ thị của quy trình "hiểu-tìm kiếm-trả lời" trong case, đánh dấu các nút, cạnh và các điều kiện chuyển tiếp trạng thái.
   - Trợ lý hiện tại là một quy trình tuyến tính. Hãy mở rộng case này bằng cách thêm một nút "phản ánh (reflection)": nếu câu trả lời được tạo ra có chất lượng thấp (ví dụ, quá ngắn gọn hoặc thiếu chi tiết), hệ thống nên tìm kiếm lại hoặc tạo lại câu trả lời. Hãy thiết kế logic cạnh có điều kiện cho cơ chế vòng lặp này.
   - Ưu điểm của `LangGraph` nằm ở việc hỗ trợ nguyên bản các vòng lặp. Hãy thiết kế một kịch bản ứng dụng phức tạp hơn tận dụng đầy đủ tính năng này: ví dụ, vòng lặp "tạo code-kiểm thử-sửa lỗi", vòng lặp "viết bài báo-review-chỉnh sửa", v.v. Hãy vẽ cấu trúc đồ thị hoàn chỉnh và giải thích chức năng của các nút then chốt.

6. Lựa chọn framework là một trong những quyết định then chốt trong phát triển sản phẩm agent. Giả sử bạn là một kiến trúc sư kỹ thuật tại một công ty `AI`, và công ty dự định phát triển ba ứng dụng sản phẩm agent sau đây. Hãy chọn framework phù hợp nhất cho mỗi ứng dụng (`AutoGen`, `AgentScope`, `CAMEL`, `LangGraph`, hoặc phát triển từ đầu không dùng framework) và giải thích chi tiết:

   **Ứng dụng A**: Hệ thống chăm sóc khách hàng thông minh, cần xử lý một lượng lớn yêu cầu đồng thời của người dùng (1000+ mỗi giây), đòi hỏi thời gian phản hồi dưới 2 giây, hệ thống cần chạy ổn định 7×24 giờ, và hỗ trợ mở rộng theo chiều ngang (horizontal scaling).

   **Ứng dụng B**: Nền tảng hỗ trợ viết bài báo nghiên cứu khoa học, cần một "agent nhà nghiên cứu" và một "agent tác giả" cộng tác sâu, cùng hoàn thành việc tổng quan tài liệu, thiết kế thực nghiệm, phân tích dữ liệu và viết bài báo. Đòi hỏi các agent tiến hành nhiều lượt thảo luận sâu và tự chủ thúc đẩy nhiệm vụ.

   **Ứng dụng C**: Hệ thống phê duyệt kiểm soát rủi ro tài chính, cần xử lý các đơn xin vay theo các quy trình nghiêm ngặt: xem xét hồ sơ → đánh giá rủi ro → tính toán hạn mức → kiểm tra tuân thủ → xem xét thủ công → quyết định cuối cùng. Mỗi khâu có các tiêu chí đánh giá và logic phân nhánh rõ ràng, đòi hỏi các quy trình có thể truy vết và kiểm toán được.


## Tài liệu tham khảo

[1] Wu Q, Bansal G, Zhang J, et al. Autogen: Enabling next-gen LLM applications via multi-agent conversations[C]//First Conference on Language Modeling. 2024.

[2] Gao D, Li Z, Pan X, et al. Agentscope: A flexible yet robust multi-agent platform[J]. arXiv preprint arXiv:2402.14034, 2024.

[3] Li G, Hammoud H, Itani H, et al. Camel: Communicative agents for" mind" exploration of large language model society[J]. Advances in Neural Information Processing Systems, 2023, 36: 51991-52008.

[4] LangChain. LangGraph [EB/OL]. (2024). https://github.com/langchain-ai/langgraph.

[5] Microsoft. AutoGen - UserProxyAgent [EB/OL]. (2024). https://microsoft.github.io/autogen/stable/reference/python/autogen_agentchat.agents.html#autogen_agentchat.agents.UserProxyAgent.
