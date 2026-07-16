# Chương 7 Xây dựng Framework Agent của bạn

Trong các chương trước, chúng ta đã giải thích những nguyên lý nền tảng của agent và trải nghiệm sự tiện lợi trong phát triển mà các framework chủ đạo mang lại. Bắt đầu từ chương này, chúng ta sẽ bước vào một giai đoạn đầy thử thách và giá trị hơn: **xây dựng một framework agent từ đầu—HelloAgents**.

Để đảm bảo tính liên tục và khả năng tái lập của quá trình học, HelloAgents sẽ được phát triển theo các phiên bản lặp (version iteration). Mỗi chương sẽ bổ sung các module chức năng mới dựa trên chương trước, đồng thời tích hợp và hiện thực hóa các kiến thức liên quan đến agent. Cuối cùng, chúng ta sẽ dùng chính framework tự xây dựng này để triển khai một cách hiệu quả các tình huống ứng dụng nâng cao trong các chương sau của cuốn sách.

## 7.1 Thiết kế Kiến trúc Tổng thể của Framework

### 7.1.1 Tại sao cần tự xây dựng Framework Agent

Trong bối cảnh công nghệ agent đang phát triển nhanh chóng ngày nay, trên thị trường đã có rất nhiều framework Agent trưởng thành. Vậy tại sao chúng ta vẫn cần xây dựng một framework mới từ đầu?

(1) Sự lặp lại nhanh chóng và những hạn chế của các framework trên thị trường

Lĩnh vực agent là một mảng phát triển nhanh, nơi các khái niệm mới không ngừng xuất hiện. Mỗi framework có định vị và cách hiểu riêng về thiết kế agent, nhưng các điểm kiến thức cốt lõi của agent thì luôn nhất quán.

- **Sự phức tạp của việc trừu tượng hóa quá mức**: Nhiều framework đưa vào vô số lớp trừu tượng (abstraction layer) và tùy chọn cấu hình nhằm theo đuổi tính tổng quát. Lấy LangChain làm ví dụ, mặc dù cơ chế gọi chuỗi (chain invocation) của nó linh hoạt, nhưng đường cong học tập lại dốc đối với người mới, thường đòi hỏi phải hiểu nhiều khái niệm mới hoàn thành được những tác vụ đơn giản.
- **Sự bất ổn do lặp lại nhanh chóng**: Các framework thương mại thường xuyên thay đổi giao diện API để giành thị phần. Nhà phát triển thường gặp phải sự bực bội khi code không chạy được sau khi nâng cấp phiên bản, khiến chi phí bảo trì luôn ở mức cao.
- **Logic hiện thực dạng hộp đen**: Nhiều framework đóng gói logic cốt lõi quá chặt, khiến nhà phát triển khó hiểu được cơ chế hoạt động nội bộ của Agent và thiếu khả năng tùy biến sâu. Khi gặp vấn đề, họ chỉ có thể dựa vào tài liệu và sự hỗ trợ của cộng đồng; đặc biệt nếu cộng đồng không đủ sôi động, phản hồi có thể mất rất nhiều thời gian mà không ai thúc đẩy, ảnh hưởng đến hiệu suất phát triển về sau.
- **Sự phức tạp của các dependency**: Các framework trưởng thành thường kéo theo một lượng lớn gói phụ thuộc (dependency package), với kích thước gói cài đặt lớn, có thể gây ra vấn đề xung đột dependency khi cần phối hợp với code của các dự án khác.

(2) Bước nhảy về năng lực từ "người dùng" sang "người xây dựng"

Tự xây dựng framework Agent thực chất là một quá trình chuyển hóa từ "người dùng" (user) sang "người xây dựng" (builder). Giá trị mà sự chuyển hóa này mang lại là lâu dài.

- **Hiểu sâu về nguyên lý hoạt động của Agent**: Bằng cách tự tay hiện thực từng thành phần, nhà phát triển có thể thực sự hiểu quá trình tư duy của Agent, cơ chế gọi công cụ (tool invocation), cũng như ưu nhược điểm và sự khác biệt của các mẫu thiết kế (design pattern) khác nhau.
- **Giành được toàn quyền kiểm soát**: Một framework tự xây dựng đồng nghĩa với việc kiểm soát hoàn toàn từng dòng code, cho phép tinh chỉnh chính xác theo nhu cầu cụ thể mà không bị ràng buộc bởi triết lý thiết kế của framework bên thứ ba.
- **Rèn luyện năng lực thiết kế hệ thống**: Quá trình xây dựng framework liên quan đến các kỹ năng kỹ thuật phần mềm cốt lõi như thiết kế module hóa, trừu tượng hóa giao diện và xử lý lỗi, những điều có giá trị lớn cho sự phát triển lâu dài của nhà phát triển.

(3) Sự cần thiết của nhu cầu tùy biến và làm chủ sâu

Trong các ứng dụng thực tế, nhu cầu đối với agent khác nhau rất nhiều giữa các tình huống, thường đòi hỏi phải phát triển thứ cấp dựa trên các framework tổng quát.

- **Nhu cầu tối ưu cho các lĩnh vực cụ thể**: Các lĩnh vực chuyên biệt như tài chính, y tế và giáo dục thường cần các mẫu prompt được nhắm mục tiêu, tích hợp công cụ đặc thù và chiến lược bảo mật được tùy chỉnh.
- **Kiểm soát chính xác về hiệu năng và tài nguyên**: Trong môi trường production, có những yêu cầu nghiêm ngặt về thời gian phản hồi, mức sử dụng bộ nhớ và khả năng xử lý đồng thời. Các giải pháp "một cỡ vừa cho tất cả" của các framework tổng quát thường không đáp ứng được các nhu cầu tinh chỉnh.
- **Yêu cầu về tính minh bạch cho việc học và giảng dạy**: Trong tình huống giảng dạy của chúng ta, người học mong muốn thấy rõ từng bước của quá trình xây dựng agent và hiểu cơ chế hoạt động của các mô hình (paradigm) khác nhau, điều này đòi hỏi framework phải có khả năng quan sát (observability) và khả năng diễn giải (interpretability) cao.

### 7.1.2 Triết lý Thiết kế của Framework HelloAgents

Xây dựng một framework Agent mới không phải là chuyện số lượng tính năng, mà là liệu triết lý thiết kế có thực sự giải quyết được những điểm nhức nhối của các framework hiện có hay không. Thiết kế của framework HelloAgents xoay quanh một câu hỏi cốt lõi: Làm thế nào để người học vừa có thể bắt đầu nhanh chóng, vừa hiểu sâu nguyên lý hoạt động của Agent?

Khi lần đầu tiếp xúc với bất kỳ framework trưởng thành nào, bạn có thể bị thu hút bởi các tính năng phong phú của nó, nhưng bạn sẽ sớm phát hiện một vấn đề: để hoàn thành một tác vụ đơn giản, bạn thường phải hiểu hơn một chục khái niệm khác nhau như Chain, Agent, Tool, Memory, Retriever, v.v. Mỗi khái niệm lại có lớp trừu tượng riêng, khiến đường cong học tập trở nên cực kỳ dốc. Mặc dù sự phức tạp này mang lại chức năng mạnh mẽ, nó cũng trở thành rào cản cho người mới bắt đầu. Framework HelloAgents cố gắng tìm sự cân bằng giữa tính hoàn chỉnh về chức năng và sự thân thiện với việc học, hình thành nên bốn triết lý thiết kế cốt lõi.

(1) Cân bằng giữa nhẹ nhàng (lightweight) và thân thiện với giảng dạy

Một framework học tập xuất sắc phải có khả năng đọc hiểu hoàn chỉnh. HelloAgents tách code cốt lõi theo từng chương, dựa trên một nguyên tắc đơn giản: bất kỳ nhà phát triển nào có một nền tảng lập trình nhất định đều có thể hiểu đầy đủ nguyên lý hoạt động của framework trong một khoảng thời gian hợp lý. Về quản lý dependency, framework áp dụng chiến lược tối giản. Ngoài SDK chính thức của OpenAI và một vài thư viện cơ bản cần thiết, không đưa vào bất kỳ dependency nặng nề nào. Khi gặp vấn đề, chúng ta có thể trực tiếp định vị được ngay code của chính framework mà không cần tìm câu trả lời trong các mối quan hệ dependency phức tạp.

(2) Lựa chọn thực dụng dựa trên API tiêu chuẩn

API của OpenAI đã trở thành một tiêu chuẩn của ngành, và hầu như tất cả các nhà cung cấp LLM chủ đạo đều đang nỗ lực tương thích với giao diện này. HelloAgents chọn xây dựng trên tiêu chuẩn này thay vì phát minh lại một giao diện trừu tượng khác. Quyết định này được thúc đẩy chủ yếu bởi vài điểm. Thứ nhất là sự đảm bảo về tính tương thích. Sau khi thành thạo việc sử dụng HelloAgents, khi chuyển sang các framework khác hoặc tích hợp nó vào các dự án hiện có, logic gọi API ở tầng dưới hoàn toàn nhất quán. Thứ hai là giảm chi phí học tập. Bạn không cần học các mô hình khái niệm mới vì tất cả các thao tác đều dựa trên các giao diện tiêu chuẩn mà bạn đã quen thuộc.

(3) Thiết kế cẩn trọng lộ trình học tập tiệm tiến

HelloAgents cung cấp một lộ trình học tập rõ ràng. Chúng ta sẽ lưu code học tập của mỗi chương thành một phiên bản lịch sử có thể tải về qua pip, nên không cần lo lắng về chi phí sử dụng code, bởi vì mọi chức năng cốt lõi đều do chính bạn viết. Thiết kế này cho phép bạn tiến lên theo nhu cầu và nhịp độ của riêng mình. Mỗi lần nâng cấp đều tự nhiên, không có bước nhảy khái niệm hay khoảng trống trong việc hiểu. Đáng nói là nội dung của chương này cũng được xây dựng dựa trên nội dung của sáu chương trước. Tương tự, chương này cũng đặt nền móng framework cho việc học các kiến thức nâng cao ở các chương sau.

(4) Trừu tượng hóa "Công cụ" thống nhất: Mọi thứ đều là Tool

Để triển khai triệt để triết lý nhẹ nhàng và thân thiện với giảng dạy, HelloAgents đã thực hiện một sự đơn giản hóa then chốt trong kiến trúc: ngoại trừ lớp Agent cốt lõi, mọi thứ đều là Tool (công cụ). Memory, RAG (Retrieval-Augmented Generation - Sinh có tăng cường truy xuất), RL (Reinforcement Learning - Học tăng cường), MCP (Protocol - giao thức), và các module khác vốn cần được học độc lập trong nhiều framework khác, tất cả đều được trừu tượng hóa thống nhất thành một "công cụ" trong HelloAgents. Ý định ban đầu của thiết kế này là loại bỏ các lớp trừu tượng không cần thiết, đưa người học trở về với logic cốt lõi trực quan nhất là "agent gọi công cụ", từ đó thực sự đạt được sự thống nhất giữa việc bắt đầu nhanh và hiểu sâu.

### 7.1.3 Mục tiêu Học tập của Chương này

Trước tiên hãy xem nội dung học tập cốt lõi của Chương 7:

```
hello-agents/
├── hello_agents/
│   │
│   ├── core/                     # Tầng framework cốt lõi
│   │   ├── agent.py              # Lớp cơ sở Agent
│   │   ├── llm.py                # Giao diện thống nhất HelloAgentsLLM
│   │   ├── message.py            # Hệ thống message
│   │   ├── config.py             # Quản lý cấu hình
│   │   └── exceptions.py         # Hệ thống ngoại lệ
│   │
│   ├── agents/                   # Tầng hiện thực Agent
│   │   ├── simple_agent.py       # Hiện thực SimpleAgent
│   │   ├── react_agent.py        # Hiện thực ReActAgent
│   │   ├── reflection_agent.py   # Hiện thực ReflectionAgent
│   │   └── plan_solve_agent.py   # Hiện thực PlanAndSolveAgent
│   │
│   ├── tools/                    # Tầng hệ thống công cụ
│   │   ├── base.py               # Lớp cơ sở Tool
│   │   ├── registry.py           # Cơ chế đăng ký công cụ
│   │   ├── chain.py              # Hệ thống quản lý chuỗi công cụ
│   │   ├── async_executor.py     # Bộ thực thi công cụ bất đồng bộ
│   │   └── builtin/              # Tập công cụ tích hợp sẵn
│   │       ├── calculator.py     # Công cụ máy tính
│   │       └── search.py         # Công cụ tìm kiếm
└──
```

Trước khi bắt đầu viết code cụ thể, chúng ta cần thiết lập một bản thiết kế kiến trúc rõ ràng. Thiết kế kiến trúc của HelloAgents tuân theo các nguyên tắc cốt lõi "phân tầng tách rời (layered decoupling), trách nhiệm đơn nhất (single responsibility), giao diện thống nhất (unified interface)", giúp duy trì tổ chức code và thuận tiện cho việc mở rộng nội dung theo từng chương.

**Bắt đầu nhanh: Cài đặt Framework HelloAgents**

Để bạn đọc có thể nhanh chóng trải nghiệm đầy đủ chức năng của chương này, chúng tôi cung cấp một gói Python có thể cài đặt trực tiếp. Bạn có thể cài đặt phiên bản tương ứng với chương này bằng lệnh sau:

```bash
# Liên kết xem code framework hello-agents: https://github.com/jjyaoao/helloagents
# Phiên bản Python cần >= 3.10
pip install "hello-agents==0.1.1"
```

Việc học chương này có thể thực hiện theo hai cách:

1. **Học trải nghiệm**: Cài đặt trực tiếp framework bằng `pip`, chạy code ví dụ và nhanh chóng trải nghiệm các chức năng khác nhau
2. **Học chuyên sâu**: Theo nội dung của chương này, hiện thực từng thành phần từ đầu và hiểu sâu ý tưởng thiết kế cũng như chi tiết triển khai của framework

Chúng tôi khuyến nghị áp dụng lộ trình học "trải nghiệm trước, hiện thực sau". Trong chương này, chúng tôi cung cấp các file kiểm thử (test file) hoàn chỉnh. Bạn có thể tự viết lại các chức năng cốt lõi và chạy các bài kiểm thử để xác minh xem cách hiện thực của mình có đúng không. Phương pháp học này vừa đảm bảo tính thực tiễn vừa đảm bảo hiệu quả học tập. Nếu bạn muốn hiểu sâu chi tiết triển khai của framework hoặc muốn tham gia phát triển framework, bạn có thể truy cập [kho GitHub này](https://github.com/jjyaoao/helloagents).

Trước khi bắt đầu, hãy trải nghiệm việc xây dựng một agent đơn giản bằng Hello-agents trong 30 giây!

```python
# Cấu hình LLM API trong file .env ở thư mục cùng cấp. Bạn có thể tham khảo .env.example trong thư mục code, hoặc tái sử dụng file .env từ các case của chương trước.
from hello_agents import SimpleAgent, HelloAgentsLLM
from dotenv import load_dotenv

# Nạp các biến môi trường
load_dotenv()

# Tạo instance LLM - framework tự động phát hiện provider
llm = HelloAgentsLLM()

# Hoặc chỉ định provider thủ công (tùy chọn)
# llm = HelloAgentsLLM(provider="modelscope")

# Tạo SimpleAgent
agent = SimpleAgent(
    name="AI Assistant",
    llm=llm,
    system_prompt="You are a helpful AI assistant"
)

# Hội thoại cơ bản
response = agent.run("Hello! Please introduce yourself")
print(response)

# Thêm chức năng công cụ (tùy chọn)
from hello_agents.tools import CalculatorTool
calculator = CalculatorTool()
# Cần hiện thực MySimpleAgent trong phần 7.4.1 để gọi, các chương sau sẽ hỗ trợ cách gọi này
# agent.add_tool(calculator)

# Bây giờ bạn có thể dùng công cụ
response = agent.run("Please help me calculate 2 + 3 * 4")
print(response)

# Xem lịch sử hội thoại
print(f"Number of historical messages: {len(agent.get_history())}")
```



## 7.2 Mở rộng HelloAgentsLLM

Nội dung của phần này sẽ là một bản nâng cấp lặp dựa trên `HelloAgentsLLM` đã được tạo ở Phần 4.1.3. Chúng ta sẽ biến client cơ bản này thành một trung tâm gọi mô hình có khả năng thích ứng cao hơn. Việc nâng cấp này chủ yếu xoay quanh ba mục tiêu sau:

1. **Hỗ trợ đa provider**: Đạt được sự chuyển đổi liền mạch giữa nhiều nhà cung cấp dịch vụ LLM chủ đạo như OpenAI, ModelScope, Zhipu AI, v.v., tránh việc framework bị ràng buộc với một nhà cung cấp cụ thể.
2. **Tích hợp mô hình cục bộ**: Giới thiệu VLLM và Ollama, hai giải pháp triển khai cục bộ hiệu năng cao, như một sự bổ sung cấp production cho giải pháp Hugging Face Transformers ở Phần 3.2.3, đáp ứng nhu cầu về bảo mật dữ liệu và kiểm soát chi phí.
3. **Cơ chế phát hiện tự động**: Thiết lập cơ chế nhận diện tự động cho phép framework suy luận thông minh loại dịch vụ LLM đang được sử dụng dựa trên thông tin môi trường, đơn giản hóa quá trình cấu hình của người dùng.

### 7.2.1 Hỗ trợ Nhiều Provider

Lớp `HelloAgentsLLM` mà chúng ta định nghĩa trước đây đã có thể kết nối với bất kỳ dịch vụ nào tương thích với giao diện OpenAI thông qua hai tham số cốt lõi `api_key` và `base_url`. Về lý thuyết điều này đảm bảo tính phổ quát, nhưng trong ứng dụng thực tế, các nhà cung cấp dịch vụ khác nhau có sự khác biệt về cách đặt tên biến môi trường, địa chỉ API mặc định và các mô hình được khuyến nghị. Nếu mỗi lần chuyển đổi nhà cung cấp dịch vụ người dùng đều phải tra cứu và chỉnh sửa code thủ công, thì sẽ ảnh hưởng lớn đến hiệu suất phát triển. Để giải quyết vấn đề này, chúng ta đưa vào `provider`. Ý tưởng cải tiến là: để `HelloAgentsLLM` xử lý các chi tiết cấu hình của các nhà cung cấp dịch vụ khác nhau ở bên trong, từ đó cung cấp cho người dùng một trải nghiệm gọi thống nhất và gọn gàng. Chúng ta sẽ trình bày chi tiết cách triển khai cụ thể ở Phần 7.2.3 "Cơ chế phát hiện tự động". Ở đây, trước tiên chúng ta tập trung vào cách sử dụng cơ chế này để mở rộng framework.

Dưới đây, chúng ta sẽ minh họa cách thêm hỗ trợ cho nền tảng ModelScope bằng cách kế thừa `HelloAgentsLLM`. Chúng tôi hy vọng bạn đọc không chỉ học cách "sử dụng" framework mà còn nắm được cách "mở rộng" nó. Trực tiếp sửa đổi mã nguồn của các thư viện đã cài đặt không phải là cách làm được khuyến nghị vì nó khiến việc nâng cấp thư viện về sau trở nên khó khăn.

(1) Tạo lớp LLM tùy chỉnh và kế thừa

Giả sử chúng ta có một file `my_llm.py` trong thư mục dự án. Trước tiên chúng ta import lớp cơ sở `HelloAgentsLLM` từ thư viện `hello_agents`, sau đó tạo một lớp mới tên là `MyLLM` kế thừa từ nó.

```python
# my_llm.py
import os
from typing import Optional
from openai import OpenAI
from hello_agents import HelloAgentsLLM

class MyLLM(HelloAgentsLLM):
    """
    Một client LLM tùy chỉnh, bổ sung hỗ trợ cho ModelScope thông qua kế thừa.
    """
    pass # Tạm để trống
```

(2) Ghi đè phương thức `__init__` để hỗ trợ Provider mới

Tiếp theo, chúng ta ghi đè (override) phương thức `__init__` trong lớp `MyLLM`. Mục tiêu của chúng ta là: khi người dùng truyền vào `provider="modelscope"`, thực thi logic tùy chỉnh của chúng ta; nếu không, gọi logic gốc của lớp cha `HelloAgentsLLM`, để nó tiếp tục hỗ trợ các provider tích hợp sẵn khác như OpenAI.

```python
class MyLLM(HelloAgentsLLM):
    def __init__(
        self,
        model: Optional[str] = None,
        api_key: Optional[str] = None,
        base_url: Optional[str] = None,
        provider: Optional[str] = "auto",
        **kwargs
    ):
        # Kiểm tra xem provider có phải là 'modelscope' mà ta muốn xử lý hay không
        if provider == "modelscope":
            print("Using custom ModelScope Provider")
            self.provider = "modelscope"

            # Phân tích thông tin xác thực (credentials) của ModelScope
            self.api_key = api_key or os.getenv("MODELSCOPE_API_KEY")
            self.base_url = base_url or "https://api-inference.modelscope.cn/v1/"

            # Xác thực rằng credentials tồn tại
            if not self.api_key:
                raise ValueError("ModelScope API key not found. Please set MODELSCOPE_API_KEY environment variable.")

            # Đặt model mặc định và các tham số khác
            self.model = model or os.getenv("LLM_MODEL_ID") or "Qwen/Qwen2.5-VL-72B-Instruct"
            self.temperature = kwargs.get('temperature', 0.7)
            self.max_tokens = kwargs.get('max_tokens')
            self.timeout = kwargs.get('timeout', 60)

            # Tạo instance client OpenAI với các tham số đã lấy được
            self._client = OpenAI(api_key=self.api_key, base_url=self.base_url, timeout=self.timeout)

        else:
            # Nếu không phải modelscope, dùng logic gốc của lớp cha để xử lý
            super().__init__(model=model, api_key=api_key, base_url=base_url, provider=provider, **kwargs)

```

Đoạn code này minh họa ý tưởng của "ghi đè": chúng ta chặn trường hợp `provider="modelscope"` và xử lý nó một cách đặc biệt. Đối với tất cả các trường hợp khác, chúng ta trả lại cho lớp cha thông qua `super().__init__(...)`, giữ nguyên toàn bộ chức năng gốc của framework.

(3) Sử dụng lớp `MyLLM` tùy chỉnh

Giờ đây, chúng ta có thể sử dụng lớp `MyLLM` của riêng mình trong logic nghiệp vụ của dự án giống hệt như dùng `HelloAgentsLLM` nguyên bản.

Trước tiên, cấu hình API key của ModelScope trong file `.env`:

```bash
# file .env
MODELSCOPE_API_KEY="your-modelscope-api-key"
```

Sau đó, import và sử dụng `MyLLM` trong chương trình chính:

```python
# my_main.py
from dotenv import load_dotenv
from my_llm import MyLLM # Lưu ý: import lớp của chính chúng ta ở đây

# Nạp các biến môi trường
load_dotenv()

# Khởi tạo client đã ghi đè của chúng ta và chỉ định provider
llm = MyLLM(provider="modelscope")

# Chuẩn bị các message
messages = [{"role": "user", "content": "Hello, please introduce yourself."}]

# Thực hiện gọi, các phương thức think và các phương thức khác được kế thừa từ lớp cha, không cần ghi đè
response_stream = llm.think(messages)

# In phản hồi
print("ModelScope Response:")
for chunk in response_stream:
    # Chunk đã được in trong my_llm, ở đây chỉ pass
    # print(chunk, end="", flush=True)
    pass
```

Qua các bước trên, chúng ta đã mở rộng thành công chức năng mới cho thư viện `hello-agents` mà không cần sửa đổi mã nguồn của nó. Phương pháp này không chỉ đảm bảo tính sạch sẽ và dễ bảo trì của code mà còn đảm bảo rằng chức năng tùy chỉnh của chúng ta sẽ không bị mất khi nâng cấp thư viện `hello-agents` trong tương lai.

### 7.2.2 Gọi Mô hình Cục bộ

Ở Phần 3.2.3, chúng ta đã học cách sử dụng thư viện Hugging Face Transformers để chạy các mô hình mã nguồn mở cục bộ. Phương pháp này rất phù hợp cho việc học nhập môn và kiểm chứng chức năng, nhưng cách hiện thực ở tầng dưới của nó có hiệu năng hạn chế khi xử lý các yêu cầu đồng thời cao (high-concurrency) và thường không phải là lựa chọn hàng đầu cho môi trường production.

Để đạt được dịch vụ suy luận mô hình hiệu năng cao, cấp production ngay tại cục bộ, cộng đồng đã tạo ra những công cụ xuất sắc như VLLM và Ollama. Chúng cải thiện đáng kể thông lượng (throughput) và hiệu suất vận hành của mô hình thông qua các kỹ thuật như continuous batching và PagedAttention, đồng thời đóng gói các mô hình thành dịch vụ API tương thích với tiêu chuẩn OpenAI. Điều này có nghĩa là chúng ta có thể tích hợp chúng vào `HelloAgentsLLM` một cách liền mạch.

**VLLM**

VLLM là một thư viện Python hiệu năng cao được thiết kế cho suy luận LLM. Thông qua các công nghệ tiên tiến như PagedAttention, nó có thể đạt thông lượng cao hơn gấp nhiều lần so với cách hiện thực Transformers tiêu chuẩn. Dưới đây là các bước hoàn chỉnh để triển khai một dịch vụ VLLM cục bộ:

Trước tiên, bạn cần cài đặt VLLM phù hợp với môi trường phần cứng của mình (đặc biệt là phiên bản CUDA). Khuyến nghị làm theo [tài liệu chính thức](https://docs.vllm.ai/en/latest/getting_started/installation.html) của nó để cài đặt nhằm tránh các vấn đề không tương thích phiên bản.

```python
pip install vllm
```

Sau khi cài đặt, dùng lệnh sau để khởi động một dịch vụ API tương thích OpenAI. VLLM sẽ tự động tải trọng số (weights) của mô hình được chỉ định từ Hugging Face Hub (nếu chúng chưa tồn tại cục bộ). Chúng ta vẫn dùng mô hình Qwen1.5-0.5B-Chat làm ví dụ:

```
# Khởi động dịch vụ VLLM và nạp mô hình Qwen1.5-0.5B-Chat
python -m vllm.entrypoints.openai.api_server \
    --model Qwen/Qwen1.5-0.5B-Chat \
    --host 0.0.0.0 \
    --port 8000
```

Sau khi dịch vụ khởi động, nó sẽ cung cấp một API tương thích OpenAI tại địa chỉ `http://localhost:8000/v1`.

**Ollama**

Ollama đơn giản hóa hơn nữa việc quản lý và triển khai mô hình cục bộ bằng cách đóng gói việc tải mô hình, cấu hình và khởi động dịch vụ thành một lệnh duy nhất, khiến nó rất phù hợp để bắt đầu nhanh. Truy cập [trang web chính thức](https://ollama.com) của Ollama để tải và cài đặt client cho hệ điều hành của bạn.

Sau khi cài đặt, mở terminal và thực thi lệnh sau để tải và chạy một mô hình (dùng Llama 3 làm ví dụ). Ollama sẽ tự động xử lý việc tải mô hình, đóng gói dịch vụ và cấu hình tăng tốc phần cứng.

```
# Lần chạy đầu tiên sẽ tự động tải mô hình, các lần chạy sau sẽ trực tiếp khởi động dịch vụ
ollama run llama3
```

Khi bạn thấy dấu nhắc tương tác của mô hình trong terminal, điều đó cho biết dịch vụ đã khởi động thành công ở chế độ nền. Ollama theo mặc định sẽ hiển thị một giao diện API tương thích OpenAI tại địa chỉ `http://localhost:11434/v1`.

**Tích hợp với `HelloAgentsLLM`**

Vì cả VLLM và Ollama đều tuân theo các API tiêu chuẩn ngành, việc tích hợp chúng vào `HelloAgentsLLM` là rất đơn giản. Chúng ta chỉ cần coi chúng như một `provider` mới khi khởi tạo client.

Ví dụ, kết nối tới một dịch vụ **VLLM** đang chạy cục bộ:

```python
llm_client = HelloAgentsLLM(
    provider="vllm",
    model="Qwen/Qwen1.5-0.5B-Chat", # Phải khớp với model được chỉ định khi khởi động dịch vụ
    base_url="http://localhost:8000/v1",
    api_key="vllm" # Dịch vụ cục bộ thường không cần API Key thật, có thể điền bất kỳ chuỗi không rỗng nào
)
```

Hoặc, bằng cách đặt các biến môi trường và để client tự động phát hiện, đạt được sự thay đổi code bằng không:

```bash
# Đặt trong file .env
LLM_BASE_URL="http://localhost:8000/v1"
LLM_API_KEY="vllm"

# Trực tiếp khởi tạo trong code Python
llm_client = HelloAgentsLLM() # Sẽ tự động phát hiện là vllm
```

Tương tự, việc kết nối tới một dịch vụ **Ollama** cục bộ cũng đơn giản như vậy:

```python
llm_client = HelloAgentsLLM(
    provider="ollama",
    model="llama3", # Phải khớp với model được chỉ định trong `ollama run`
    base_url="http://localhost:11434/v1",
    api_key="ollama" # Dịch vụ cục bộ cũng không cần Key thật
)
```

Thông qua thiết kế thống nhất này, code cốt lõi của agent chúng ta không cần bất kỳ sửa đổi nào để tự do chuyển đổi giữa các API đám mây và các mô hình cục bộ. Điều này cung cấp sự linh hoạt lớn cho việc phát triển ứng dụng, triển khai, kiểm soát chi phí và bảo vệ quyền riêng tư dữ liệu về sau.

### 7.2.3 Cơ chế Phát hiện Tự động

Để giảm thiểu tối đa gánh nặng cấu hình của người dùng và tuân theo nguyên tắc "quy ước hơn cấu hình" (convention over configuration), `HelloAgentsLLM` thiết kế bên trong hai phương thức phụ trợ cốt lõi: `_auto_detect_provider` và `_resolve_credentials`. Chúng phối hợp với nhau, trong đó `_auto_detect_provider` chịu trách nhiệm suy luận nhà cung cấp dịch vụ dựa trên thông tin môi trường, còn `_resolve_credentials` hoàn tất việc cấu hình tham số cụ thể dựa trên kết quả suy luận.

Phương thức `_auto_detect_provider` chịu trách nhiệm tự động suy luận nhà cung cấp dịch vụ dựa trên thông tin môi trường, theo thứ tự ưu tiên sau:

1. **Ưu tiên cao nhất: Kiểm tra các biến môi trường của các nhà cung cấp dịch vụ cụ thể** Đây là căn cứ phán đoán trực tiếp và đáng tin cậy nhất. Framework sẽ lần lượt kiểm tra xem các biến môi trường như `MODELSCOPE_API_KEY`, `OPENAI_API_KEY`, `ZHIPU_API_KEY`, v.v. có tồn tại hay không. Một khi tìm thấy bất kỳ biến nào, nó sẽ ngay lập tức xác định nhà cung cấp dịch vụ tương ứng.

2. **Ưu tiên cao thứ hai: Xác định dựa trên `base_url`** Nếu người dùng chưa đặt key của một nhà cung cấp dịch vụ cụ thể nhưng đã đặt `LLM_BASE_URL` tổng quát, framework sẽ phân tích URL này thay thế.

   - **Khớp tên miền (Domain Matching)**: Nhận diện các nhà cung cấp dịch vụ đám mây bằng cách kiểm tra xem URL có chứa các chuỗi đặc trưng như `"api-inference.modelscope.cn"`, `"api.openai.com"`, v.v. hay không.

   - **Khớp cổng (Port Matching)**: Nhận diện các giải pháp triển khai cục bộ bằng cách kiểm tra xem URL có chứa các cổng tiêu chuẩn của dịch vụ cục bộ như `:11434` (Ollama), `:8000` (VLLM), v.v. hay không.

3. **Phán đoán phụ trợ: Phân tích định dạng API Key** Trong một số trường hợp, nếu cả hai phương pháp trên đều không xác định được, framework sẽ thử phân tích định dạng của biến môi trường tổng quát `LLM_API_KEY`. Ví dụ, API key của một số nhà cung cấp dịch vụ có tiền tố cố định hoặc định dạng mã hóa độc nhất. Tuy nhiên, vì phương pháp này có thể có sự mơ hồ (ví dụ, nhiều nhà cung cấp dịch vụ có định dạng key tương tự nhau), nên nó có mức ưu tiên thấp hơn và chỉ được dùng như một biện pháp phụ trợ.

Một số code then chốt như sau:

```python
def _auto_detect_provider(self, api_key: Optional[str], base_url: Optional[str]) -> str:
    """
    Tự động phát hiện provider LLM
    """
    # 1. Kiểm tra biến môi trường của các provider cụ thể (ưu tiên cao nhất)
    if os.getenv("MODELSCOPE_API_KEY"): return "modelscope"
    if os.getenv("OPENAI_API_KEY"): return "openai"
    if os.getenv("ZHIPU_API_KEY"): return "zhipu"
    # ... Kiểm tra biến môi trường của các nhà cung cấp dịch vụ khác

    # Lấy các biến môi trường tổng quát
    actual_api_key = api_key or os.getenv("LLM_API_KEY")
    actual_base_url = base_url or os.getenv("LLM_BASE_URL")

    # 2. Xác định dựa trên base_url
    if actual_base_url:
        base_url_lower = actual_base_url.lower()
        if "api-inference.modelscope.cn" in base_url_lower: return "modelscope"
        if "open.bigmodel.cn" in base_url_lower: return "zhipu"
        if "localhost" in base_url_lower or "127.0.0.1" in base_url_lower:
            if ":11434" in base_url_lower: return "ollama"
            if ":8000" in base_url_lower: return "vllm"
            return "local" # Các cổng cục bộ khác

    # 3. Phán đoán phụ trợ dựa trên định dạng API key
    if actual_api_key:
        if actual_api_key.startswith("ms-"): return "modelscope"
        # ... Các phán đoán định dạng key khác

    # 4. Mặc định trả về 'auto', dùng cấu hình tổng quát
    return "auto"
```

Một khi `provider` được xác định (dù do người dùng chỉ định hay tự động phát hiện), phương thức `_resolve_credentials` sẽ tiếp quản để xử lý cấu hình khác biệt của các nhà cung cấp dịch vụ. Nó sẽ chủ động tìm kiếm các biến môi trường tương ứng dựa trên giá trị của `provider` và đặt `base_url` mặc định cho nó. Một số hiện thực then chốt như sau:

```python
def _resolve_credentials(self, api_key: Optional[str], base_url: Optional[str]) -> tuple[str, str]:
    """Phân giải API key và base_url dựa trên provider"""
    if self.provider == "openai":
        resolved_api_key = api_key or os.getenv("OPENAI_API_KEY") or os.getenv("LLM_API_KEY")
        resolved_base_url = base_url or os.getenv("LLM_BASE_URL") or "https://api.openai.com/v1"
        return resolved_api_key, resolved_base_url

    elif self.provider == "modelscope":
        resolved_api_key = api_key or os.getenv("MODELSCOPE_API_KEY") or os.getenv("LLM_API_KEY")
        resolved_base_url = base_url or os.getenv("LLM_BASE_URL") or "https://api-inference.modelscope.cn/v1/"
        return resolved_api_key, resolved_base_url

    # ... Logic cho các nhà cung cấp dịch vụ khác
```

Hãy trải nghiệm sự tiện lợi mà việc phát hiện tự động mang lại qua một ví dụ đơn giản. Giả sử một người dùng muốn dùng dịch vụ Ollama cục bộ, họ chỉ cần cấu hình file `.env` như sau:

```bash
LLM_BASE_URL="http://localhost:11434/v1"
LLM_MODEL_ID="llama3"
```

Họ hoàn toàn không cần cấu hình `LLM_API_KEY` hay chỉ định `provider` trong code. Sau đó, trong code Python, họ chỉ cần khởi tạo `HelloAgentsLLM`:

```python
from dotenv import load_dotenv
from hello_agents import HelloAgentsLLM

load_dotenv()

# Không cần truyền provider, framework sẽ tự động phát hiện
llm = HelloAgentsLLM()
# Log nội bộ của framework sẽ hiển thị provider được phát hiện là 'ollama'

# Các phương thức gọi tiếp theo hoàn toàn không thay đổi
messages = [{"role": "user", "content": "Hello!"}]
for chunk in llm.think(messages):
    print(chunk, end="")

```

Trong quá trình này, phương thức `_auto_detect_provider` suy luận thành công `provider` là `"ollama"` bằng cách phân tích `"localhost"` và `:11434` trong `LLM_BASE_URL`. Tiếp đó, phương thức `_resolve_credentials` đặt các tham số mặc định đúng cho Ollama.

So với hiện thực cơ bản ở Phần 4.1.3, HelloAgentsLLM hiện tại có những ưu điểm nổi bật sau:

<div align="center">
  <p>Bảng 7.1 So sánh Tính năng của Các Phiên bản HelloAgentLLM Khác nhau</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/7-figures/table-01.png" alt="" width="90%"/>
</div>

Như thể hiện trong Bảng 7.1 ở trên, sự tiến hóa này thể hiện một nguyên tắc quan trọng của thiết kế framework: **bắt đầu đơn giản, dần dần hoàn thiện**. Chúng ta nâng cao tính hoàn chỉnh về chức năng trong khi vẫn giữ giao diện đơn giản.



## 7.3 Hiện thực Giao diện của Framework

Trong phần trước, chúng ta đã xây dựng `HelloAgentsLLM`, một thành phần cốt lõi giải quyết vấn đề then chốt là giao tiếp với các mô hình ngôn ngữ lớn. Tuy nhiên, nó vẫn cần một loạt các giao diện và thành phần hỗ trợ để xử lý luồng dữ liệu, quản lý cấu hình, xử lý ngoại lệ, và cung cấp một cấu trúc rõ ràng, thống nhất cho việc xây dựng ứng dụng ở tầng trên. Phần này sẽ trình bày ba file cốt lõi sau:

- `message.py`: Định nghĩa định dạng message thống nhất trong framework, đảm bảo sự chuẩn hóa trong việc truyền thông tin giữa các agent và mô hình.
- `config.py`: Cung cấp một giải pháp quản lý cấu hình tập trung, giúp hành vi của framework dễ điều chỉnh và mở rộng.
- `agent.py`: Định nghĩa lớp cơ sở trừu tượng (`Agent`) cho tất cả các agent, cung cấp một giao diện và đặc tả thống nhất để hiện thực các loại agent khác nhau trong tương lai.

### 7.3.1 Lớp Message

Trong sự tương tác giữa các agent và các mô hình ngôn ngữ lớn, lịch sử hội thoại là ngữ cảnh (context) rất quan trọng. Để quản lý thông tin này một cách chuẩn hóa, chúng ta thiết kế một lớp `Message` đơn giản. Nó sẽ được mở rộng trong chương kỹ thuật ngữ cảnh (context engineering) sau này.

```python
"""Hệ thống message"""
from typing import Optional, Dict, Any, Literal
from datetime import datetime
from pydantic import BaseModel

# Định nghĩa kiểu role của message, giới hạn các giá trị của nó
MessageRole = Literal["user", "assistant", "system", "tool"]

class Message(BaseModel):
    """Lớp Message"""

    content: str
    role: MessageRole
    timestamp: datetime = None
    metadata: Optional[Dict[str, Any]] = None

    def __init__(self, content: str, role: MessageRole, **kwargs):
        super().__init__(
            content=content,
            role=role,
            timestamp=kwargs.get('timestamp', datetime.now()),
            metadata=kwargs.get('metadata', {})
        )

    def to_dict(self) -> Dict[str, Any]:
        """Chuyển đổi sang định dạng dictionary (định dạng OpenAI API)"""
        return {
            "role": self.role,
            "content": self.content
        }

    def __str__(self) -> str:
        return f"[{self.role}] {self.content}"
```

Thiết kế của lớp này có vài điểm then chốt. Thứ nhất, chúng ta giới hạn nghiêm ngặt các giá trị của trường `role` chỉ còn bốn loại: `"user"`, `"assistant"`, `"system"`, `"tool"` thông qua `typing.Literal`, điều này tương ứng trực tiếp với đặc tả của OpenAI API và đảm bảo an toàn kiểu (type safety). Ngoài hai trường cốt lõi `content` và `role`, chúng ta còn thêm `timestamp` và `metadata`, dành chỗ cho việc ghi log và mở rộng tính năng trong tương lai. Cuối cùng, phương thức `to_dict()` là một trong những chức năng cốt lõi của nó, chịu trách nhiệm chuyển đổi đối tượng `Message` được dùng nội bộ thành định dạng dictionary tương thích với OpenAI API, thể hiện nguyên tắc thiết kế "phong phú bên trong, tương thích bên ngoài".

### 7.3.2 Lớp Config

Trách nhiệm của lớp `Config` là tập trung các tham số cấu hình bị hard-code trong code và hỗ trợ đọc từ các biến môi trường.

```python
"""Quản lý cấu hình"""
import os
from typing import Optional, Dict, Any
from pydantic import BaseModel

class Config(BaseModel):
    """Lớp cấu hình HelloAgents"""

    # Cấu hình LLM
    default_model: str = "gpt-3.5-turbo"
    default_provider: str = "openai"
    temperature: float = 0.7
    max_tokens: Optional[int] = None

    # Cấu hình hệ thống
    debug: bool = False
    log_level: str = "INFO"

    # Cấu hình khác
    max_history_length: int = 100

    @classmethod
    def from_env(cls) -> "Config":
        """Tạo cấu hình từ các biến môi trường"""
        return cls(
            debug=os.getenv("DEBUG", "false").lower() == "true",
            log_level=os.getenv("LOG_LEVEL", "INFO"),
            temperature=float(os.getenv("TEMPERATURE", "0.7")),
            max_tokens=int(os.getenv("MAX_TOKENS")) if os.getenv("MAX_TOKENS") else None,
        )

    def to_dict(self) -> Dict[str, Any]:
        """Chuyển đổi sang dictionary"""
        return self.dict()
```

Thứ nhất, chúng ta chia các mục cấu hình một cách logic thành `Cấu hình LLM`, `Cấu hình hệ thống`, v.v., làm cho cấu trúc rõ ràng ngay từ cái nhìn đầu tiên. Thứ hai, mỗi mục cấu hình đều có một giá trị mặc định hợp lý, đảm bảo framework có thể hoạt động với cấu hình bằng không (zero configuration). Cốt lõi nhất là phương thức lớp (class method) `from_env()`, cho phép người dùng ghi đè cấu hình mặc định bằng cách đặt các biến môi trường mà không cần sửa code, điều này đặc biệt hữu ích khi triển khai đến các môi trường khác nhau.

### 7.3.3 Lớp cơ sở trừu tượng Agent

Lớp `Agent` là mức trừu tượng cao nhất (top-level abstraction) của toàn bộ framework. Nó định nghĩa các hành vi và thuộc tính chung mà một agent nên có, nhưng không quan tâm đến các cách hiện thực cụ thể. Chúng ta hiện thực nó thông qua module `abc` (Abstract Base Classes) của Python, module này buộc tất cả các hiện thực agent cụ thể (như `SimpleAgent`, `ReActAgent`, v.v. ở các chương sau) phải tuân theo cùng một "giao diện".

```python
"""Lớp cơ sở Agent"""
from abc import ABC, abstractmethod
from typing import Optional, Any
from .message import Message
from .llm import HelloAgentsLLM
from .config import Config

class Agent(ABC):
    """Lớp cơ sở Agent"""

    def __init__(
        self,
        name: str,
        llm: HelloAgentsLLM,
        system_prompt: Optional[str] = None,
        config: Optional[Config] = None
    ):
        self.name = name
        self.llm = llm
        self.system_prompt = system_prompt
        self.config = config or Config()
        self._history: list[Message] = []

    @abstractmethod
    def run(self, input_text: str, **kwargs) -> str:
        """Chạy Agent"""
        pass

    def add_message(self, message: Message):
        """Thêm message vào lịch sử"""
        self._history.append(message)

    def clear_history(self):
        """Xóa lịch sử"""
        self._history.clear()

    def get_history(self) -> list[Message]:
        """Lấy lịch sử"""
        return self._history.copy()

    def __str__(self) -> str:
        return f"Agent(name={self.name}, provider={self.llm.provider})"
```

Thiết kế của lớp này thể hiện nguyên tắc trừu tượng hóa trong lập trình hướng đối tượng. Thứ nhất, nó được định nghĩa là một lớp trừu tượng không thể được khởi tạo trực tiếp bằng cách kế thừa `ABC`. Hàm khởi tạo `__init__` của nó định nghĩa rõ ràng các dependency cốt lõi của một Agent: tên, instance LLM, system prompt và cấu hình. Phần quan trọng nhất là phương thức `run` được trang trí (decorate) bằng `@abstractmethod`, buộc tất cả các lớp con phải hiện thực phương thức này, từ đó đảm bảo rằng tất cả các agent đều có một điểm truy cập thực thi (execution entry point) thống nhất. Ngoài ra, lớp cơ sở còn cung cấp các phương thức quản lý lịch sử chung, phối hợp với lớp `Message`, phản ánh sự liên kết giữa các thành phần.

Đến đây, chúng ta đã hoàn thành việc thiết kế và hiện thực các thành phần cơ bản cốt lõi của framework `HelloAgents`.

## 7.4 Hiện thực các Mô hình Agent trong Framework

Nội dung của phần này sẽ tiến hành tái cấu trúc (refactoring) theo hướng framework dựa trên ba mô hình Agent kinh điển (ReAct, Plan-and-Solve, Reflection) đã được xây dựng ở Chương 4, và bổ sung SimpleAgent như một mô hình hội thoại cơ bản. Chúng ta sẽ biến các hiện thực Agent độc lập này thành các thành phần framework dựa trên một kiến trúc thống nhất. Việc tái cấu trúc này chủ yếu xoay quanh ba mục tiêu cốt lõi sau:

1. **Cải thiện có hệ thống về kỹ thuật prompt (Prompt Engineering)**: Tối ưu sâu các prompt từ Chương 4, chuyển từ hướng tác vụ cụ thể sang thiết kế tổng quát hóa, đồng thời tăng cường ràng buộc về định dạng và định nghĩa vai trò.
2. **Chuẩn hóa và thống nhất giao diện và định dạng**: Thiết lập một lớp cơ sở Agent thống nhất và giao diện chạy chuẩn hóa, tất cả các Agent tuân theo cùng các tham số khởi tạo, chữ ký phương thức (method signature) và cơ chế quản lý lịch sử.
3. **Khả năng tùy biến có tính cấu hình cao**: Hỗ trợ các mẫu prompt, tham số cấu hình và chiến lược thực thi do người dùng tự định nghĩa.

### 7.4.1 SimpleAgent

SimpleAgent là hiện thực Agent cơ bản nhất, minh họa cách xây dựng một agent hội thoại hoàn chỉnh trên nền tảng framework. Chúng ta sẽ mở rộng lớp `SimpleAgent` hiện có và ghi đè các phương thức cốt lõi của nó để xây dựng một phiên bản dễ mở rộng hơn. Trước tiên, tạo một file `my_simple_agent.py` trong thư mục dự án của bạn:

```python
# my_simple_agent.py
from typing import Optional, Iterator
from hello_agents import SimpleAgent, HelloAgentsLLM, Config, Message

class MySimpleAgent(SimpleAgent):
    """
    Agent hội thoại đơn giản được viết lại
    Minh họa cách xây dựng một Agent tùy chỉnh bằng cách mở rộng SimpleAgent
    """

    def __init__(
        self,
        name: str,
        llm: HelloAgentsLLM,
        system_prompt: Optional[str] = None,
        config: Optional[Config] = None,
        tool_registry: Optional['ToolRegistry'] = None,
        enable_tool_calling: bool = True
    ):
        super().__init__(name, llm, system_prompt, config)
        self.tool_registry = tool_registry
        self.enable_tool_calling = enable_tool_calling and tool_registry is not None
        print(f"✅ {name} initialization complete, tool calling: {'enabled' if self.enable_tool_calling else 'disabled'}")
```

Tiếp theo, chúng ta cần ghi đè phương thức `run`. SimpleAgent hỗ trợ chức năng gọi công cụ tùy chọn, điều này cũng thuận tiện cho việc mở rộng ở các chương sau:

```python
# Tiếp tục thêm vào my_simple_agent.py
import re

class MySimpleAgent(SimpleAgent):
    # ... phương thức __init__ ở trên

    def run(self, input_text: str, max_tool_iterations: int = 3, **kwargs) -> str:
        """
        Phương thức run được viết lại - hiện thực logic hội thoại đơn giản, hỗ trợ gọi công cụ tùy chọn
        """
        print(f"🤖 {self.name} is processing: {input_text}")

        # Xây dựng danh sách message
        messages = []

        # Thêm system message (có thể bao gồm thông tin công cụ)
        enhanced_system_prompt = self._get_enhanced_system_prompt()
        messages.append({"role": "system", "content": enhanced_system_prompt})

        # Thêm các message lịch sử
        for msg in self._history:
            messages.append({"role": msg.role, "content": msg.content})

        # Thêm message hiện tại của người dùng
        messages.append({"role": "user", "content": input_text})

        # Nếu không bật gọi công cụ, dùng logic hội thoại đơn giản
        if not self.enable_tool_calling:
            response = self.llm.invoke(messages, **kwargs)
            self.add_message(Message(input_text, "user"))
            self.add_message(Message(response, "assistant"))
            print(f"✅ {self.name} response complete")
            return response

        # Logic hỗ trợ nhiều vòng gọi công cụ
        return self._run_with_tools(messages, input_text, max_tool_iterations, **kwargs)

    def _get_enhanced_system_prompt(self) -> str:
        """Xây dựng system prompt được tăng cường, bao gồm thông tin công cụ"""
        base_prompt = self.system_prompt or "You are a helpful AI assistant."

        if not self.enable_tool_calling or not self.tool_registry:
            return base_prompt

        # Lấy mô tả công cụ
        tools_description = self.tool_registry.get_tools_description()
        if not tools_description or tools_description == "No tools available":
            return base_prompt

        tools_section = "\n\n## Available Tools\n"
        tools_section += "You can use the following tools to help answer questions:\n"
        tools_section += tools_description + "\n"

        tools_section += "\n## Tool Calling Format\n"
        tools_section += "When you need to use a tool, please use the following format:\n"
        tools_section += "`[TOOL_CALL:{tool_name}:{parameters}]`\n"
        tools_section += "For example: `[TOOL_CALL:search:Python programming]` or `[TOOL_CALL:memory:recall=user information]`\n\n"
        tools_section += "Tool calling results will be automatically inserted into the conversation, and then you can continue answering based on the results.\n"

        return base_prompt + tools_section
```

Bây giờ chúng ta hiện thực logic cốt lõi của việc gọi công cụ:

```python
# Tiếp tục thêm vào my_simple_agent.py
class MySimpleAgent(SimpleAgent):
    # ... các phương thức ở trên

    def _run_with_tools(self, messages: list, input_text: str, max_tool_iterations: int, **kwargs) -> str:
        """Logic chạy hỗ trợ gọi công cụ"""
        current_iteration = 0
        final_response = ""

        while current_iteration < max_tool_iterations:
            # Gọi LLM
            response = self.llm.invoke(messages, **kwargs)

            # Kiểm tra xem có lệnh gọi công cụ nào không
            tool_calls = self._parse_tool_calls(response)

            if tool_calls:
                print(f"🔧 Detected {len(tool_calls)} tool calls")
                # Thực thi tất cả các lệnh gọi công cụ và thu thập kết quả
                tool_results = []
                clean_response = response

                for call in tool_calls:
                    result = self._execute_tool_call(call['tool_name'], call['parameters'])
                    tool_results.append(result)
                    # Xóa các dấu (marker) gọi công cụ khỏi phản hồi
                    clean_response = clean_response.replace(call['original'], "")

                # Xây dựng message chứa kết quả công cụ
                messages.append({"role": "assistant", "content": clean_response})

                # Thêm kết quả công cụ
                tool_results_text = "\n\n".join(tool_results)
                messages.append({"role": "user", "content": f"Tool execution results:\n{tool_results_text}\n\nPlease provide a complete answer based on these results."})

                current_iteration += 1
                continue

            # Không có lệnh gọi công cụ, đây là câu trả lời cuối cùng
            final_response = response
            break

        # Nếu vượt quá số vòng lặp tối đa, lấy phản hồi cuối cùng
        if current_iteration >= max_tool_iterations and not final_response:
            final_response = self.llm.invoke(messages, **kwargs)

        # Lưu vào lịch sử
        self.add_message(Message(input_text, "user"))
        self.add_message(Message(final_response, "assistant"))
        print(f"✅ {self.name} response complete")

        return final_response

    def _parse_tool_calls(self, text: str) -> list:
        """Phân tích các lệnh gọi công cụ trong văn bản"""
        pattern = r'\[TOOL_CALL:([^:]+):([^\]]+)\]'
        matches = re.findall(pattern, text)

        tool_calls = []
        for tool_name, parameters in matches:
            tool_calls.append({
                'tool_name': tool_name.strip(),
                'parameters': parameters.strip(),
                'original': f'[TOOL_CALL:{tool_name}:{parameters}]'
            })

        return tool_calls

    def _execute_tool_call(self, tool_name: str, parameters: str) -> str:
        """Thực thi lệnh gọi công cụ"""
        if not self.tool_registry:
            return f"❌ Error: Tool registry not configured"

        try:
            # Phân tích tham số thông minh
            if tool_name == 'calculator':
                # Công cụ calculator truyền trực tiếp biểu thức
                result = self.tool_registry.execute_tool(tool_name, parameters)
            else:
                # Các công cụ khác dùng phân tích tham số thông minh
                param_dict = self._parse_tool_parameters(tool_name, parameters)
                tool = self.tool_registry.get_tool(tool_name)
                if not tool:
                    return f"❌ Error: Tool '{tool_name}' not found"
                result = tool.run(param_dict)

            return f"🔧 Tool {tool_name} execution result:\n{result}"

        except Exception as e:
            return f"❌ Tool call failed: {str(e)}"

    def _parse_tool_parameters(self, tool_name: str, parameters: str) -> dict:
        """Phân tích thông minh các tham số của công cụ"""
        param_dict = {}

        if '=' in parameters:
            # Định dạng: key=value hoặc action=search,query=Python
            if ',' in parameters:
                # Nhiều tham số: action=search,query=Python,limit=3
                pairs = parameters.split(',')
                for pair in pairs:
                    if '=' in pair:
                        key, value = pair.split('=', 1)
                        param_dict[key.strip()] = value.strip()
            else:
                # Tham số đơn: key=value
                key, value = parameters.split('=', 1)
                param_dict[key.strip()] = value.strip()
        else:
            # Truyền trực tiếp tham số, suy luận thông minh dựa trên loại công cụ
            if tool_name == 'search':
                param_dict = {'query': parameters}
            elif tool_name == 'memory':
                param_dict = {'action': 'search', 'query': parameters}
            else:
                param_dict = {'input': parameters}

        return param_dict
```

Chúng ta cũng có thể thêm chức năng phản hồi dạng luồng (streaming response) và các phương thức tiện lợi vào Agent tùy chỉnh:

```python
# Tiếp tục thêm vào my_simple_agent.py
class MySimpleAgent(SimpleAgent):
    # ... các phương thức ở trên

    def stream_run(self, input_text: str, **kwargs) -> Iterator[str]:
        """
        Phương thức chạy dạng luồng tùy chỉnh
        """
        print(f"🌊 {self.name} starting streaming processing: {input_text}")

        messages = []

        if self.system_prompt:
            messages.append({"role": "system", "content": self.system_prompt})

        for msg in self._history:
            messages.append({"role": msg.role, "content": msg.content})

        messages.append({"role": "user", "content": input_text})

        # Gọi LLM theo luồng
        full_response = ""
        print("📝 Real-time response: ", end="")
        for chunk in self.llm.stream_invoke(messages, **kwargs):
            full_response += chunk
            print(chunk, end="", flush=True)
            yield chunk

        print()  # Xuống dòng

        # Lưu toàn bộ hội thoại vào lịch sử
        self.add_message(Message(input_text, "user"))
        self.add_message(Message(full_response, "assistant"))
        print(f"✅ {self.name} streaming response complete")

    def add_tool(self, tool) -> None:
        """Thêm công cụ vào Agent (phương thức tiện lợi)"""
        if not self.tool_registry:
            from hello_agents import ToolRegistry
            self.tool_registry = ToolRegistry()
            self.enable_tool_calling = True

        self.tool_registry.register_tool(tool)
        print(f"🔧 Tool '{tool.name}' added")

    def has_tools(self) -> bool:
        """Kiểm tra xem có công cụ khả dụng không"""
        return self.enable_tool_calling and self.tool_registry is not None

    def remove_tool(self, tool_name: str) -> bool:
        """Gỡ bỏ công cụ (phương thức tiện lợi)"""
        if self.tool_registry:
            self.tool_registry.unregister(tool_name)
            return True
        return False

    def list_tools(self) -> list:
        """Liệt kê tất cả các công cụ khả dụng"""
        if self.tool_registry:
            return self.tool_registry.list_tools()
        return []
```

Tạo một file kiểm thử `test_simple_agent.py`:

```python
# test_simple_agent.py
from dotenv import load_dotenv
from hello_agents import HelloAgentsLLM, ToolRegistry
from hello_agents.tools import CalculatorTool
from my_simple_agent import MySimpleAgent

# Nạp các biến môi trường
load_dotenv()

# Tạo instance LLM
llm = HelloAgentsLLM()

# Test 1: Agent hội thoại cơ bản (không có công cụ)
print("=== Test 1: Basic Conversation ===")
basic_agent = MySimpleAgent(
    name="Basic Assistant",
    llm=llm,
    system_prompt="You are a friendly AI assistant, please answer questions in a concise and clear manner."
)

response1 = basic_agent.run("Hello, please introduce yourself")
print(f"Basic conversation response: {response1}\n")

# Test 2: Agent có công cụ
print("=== Test 2: Tool-Enhanced Conversation ===")
tool_registry = ToolRegistry()
calculator = CalculatorTool()
tool_registry.register_tool(calculator)

enhanced_agent = MySimpleAgent(
    name="Enhanced Assistant",
    llm=llm,
    system_prompt="You are an intelligent assistant that can use tools to help users.",
    tool_registry=tool_registry,
    enable_tool_calling=True
)

response2 = enhanced_agent.run("Please help me calculate 15 * 8 + 32")
print(f"Tool-enhanced response: {response2}\n")

# Test 3: Phản hồi dạng luồng
print("=== Test 3: Streaming Response ===")
print("Streaming response: ", end="")
for chunk in basic_agent.stream_run("Please explain what artificial intelligence is"):
    pass  # Nội dung đã được in theo thời gian thực trong stream_run

# Test 4: Thêm công cụ động
print("\n=== Test 4: Dynamic Tool Management ===")
print(f"Before adding tool: {basic_agent.has_tools()}")
basic_agent.add_tool(calculator)
print(f"After adding tool: {basic_agent.has_tools()}")
print(f"Available tools: {basic_agent.list_tools()}")

# Xem lịch sử hội thoại
print(f"\nConversation history: {len(basic_agent.get_history())} messages")
```

Trong phần này, bằng cách kế thừa lớp cơ sở `Agent`, chúng ta đã xây dựng thành công một agent hội thoại cơ bản đầy đủ chức năng `MySimpleAgent` tuân theo đặc tả của framework. Nó không chỉ hỗ trợ hội thoại cơ bản mà còn có khả năng gọi công cụ tùy chọn, phản hồi dạng luồng và các phương thức quản lý công cụ tiện lợi.

### 7.4.2 ReActAgent

ReActAgent dựa trên framework giữ nguyên logic cốt lõi trong khi cải thiện tổ chức code và khả năng bảo trì, chủ yếu thông qua việc tối ưu prompt và tích hợp với hệ thống công cụ của framework.

(1) Cải thiện Mẫu Prompt

Giữ nguyên các yêu cầu định dạng ban đầu, nhấn mạnh rằng "chỉ có thể thực thi một bước tại một thời điểm" để tránh nhầm lẫn, và làm rõ các tình huống sử dụng của hai loại Action.

```python
MY_REACT_PROMPT = """You are an AI assistant with reasoning and action capabilities. You can analyze problems through thinking, then call appropriate tools to obtain information, and finally provide accurate answers.

## Available Tools
{tools}

## Workflow
Please respond strictly in the following format, executing only one step at a time:

Thought: Analyze the current problem and think about what information is needed or what action to take.
Action: Choose an action, the format must be one of the following:
- `{{tool_name}}[{{tool_input}}]` - Call specified tool
- `Finish[final answer]` - When you have enough information to give a final answer

## Important Reminders
1. Each response must include both Thought and Action parts
2. Tool call format must strictly follow: tool_name[parameters]
3. Only use Finish when you are confident you have enough information to answer the question
4. If the information returned by the tool is insufficient, continue using other tools or different parameters of the same tool

## Current Task
**Question:** {question}

## Execution History
{history}

Now begin your reasoning and action:
"""
```

(2) Hiện thực Đầy đủ ReActAgent được Viết lại

Tạo một file `my_react_agent.py` để viết lại ReActAgent:

```python
# my_react_agent.py
import re
from typing import Optional, List, Tuple
from hello_agents import ReActAgent, HelloAgentsLLM, Config, Message, ToolRegistry

class MyReActAgent(ReActAgent):
    """
    ReAct Agent được viết lại - Agent kết hợp suy luận và hành động
    """

    def __init__(
        self,
        name: str,
        llm: HelloAgentsLLM,
        tool_registry: ToolRegistry,
        system_prompt: Optional[str] = None,
        config: Optional[Config] = None,
        max_steps: int = 5,
        custom_prompt: Optional[str] = None
    ):
        super().__init__(name, llm, system_prompt, config)
        self.tool_registry = tool_registry
        self.max_steps = max_steps
        self.current_history: List[str] = []
        self.prompt_template = custom_prompt if custom_prompt else MY_REACT_PROMPT
        print(f"✅ {name} initialization complete, max steps: {max_steps}")
```

Ý nghĩa của các tham số khởi tạo của nó như sau:

- `name`: Tên của Agent.
- `llm`: Instance của `HelloAgentsLLM`, chịu trách nhiệm giao tiếp với mô hình ngôn ngữ lớn.
- `tool_registry`: Instance của `ToolRegistry`, dùng để quản lý và thực thi các công cụ khả dụng cho Agent.
- `system_prompt`: System prompt, dùng để thiết lập vai trò và các nguyên tắc hành vi của Agent.
- `config`: Đối tượng cấu hình, dùng để truyền các thiết lập ở cấp framework.
- `max_steps`: Số bước thực thi tối đa của vòng lặp ReAct, ngăn chặn vòng lặp vô hạn.
- `custom_prompt`: Mẫu prompt tùy chỉnh, dùng để thay thế prompt ReAct mặc định.

ReActAgent dựa trên framework phân rã quá trình thực thi thành các bước rõ ràng:

```python
def run(self, input_text: str, **kwargs) -> str:
    """Chạy ReAct Agent"""
    self.current_history = []
    current_step = 0

    print(f"\n🤖 {self.name} starting to process question: {input_text}")

    while current_step < self.max_steps:
        current_step += 1
        print(f"\n--- Step {current_step} ---")

        # 1. Xây dựng prompt
        tools_desc = self.tool_registry.get_tools_description()
        history_str = "\n".join(self.current_history)
        prompt = self.prompt_template.format(
            tools=tools_desc,
            question=input_text,
            history=history_str
        )

        # 2. Gọi LLM
        messages = [{"role": "user", "content": prompt}]
        response_text = self.llm.invoke(messages, **kwargs)

        # 3. Phân tích output
        thought, action = self._parse_output(response_text)

        # 4. Kiểm tra điều kiện hoàn thành
        if action and action.startswith("Finish"):
            final_answer = self._parse_action_input(action)
            self.add_message(Message(input_text, "user"))
            self.add_message(Message(final_answer, "assistant"))
            return final_answer

        # 5. Thực thi lệnh gọi công cụ
        if action:
            tool_name, tool_input = self._parse_action(action)
            observation = self.tool_registry.execute_tool(tool_name, tool_input)
            self.current_history.append(f"Action: {action}")
            self.current_history.append(f"Observation: {observation}")

    # Đã đạt số bước tối đa
    final_answer = "Sorry, I cannot complete this task within the limited number of steps."
    self.add_message(Message(input_text, "user"))
    self.add_message(Message(final_answer, "assistant"))
    return final_answer
```

Thông qua việc tái cấu trúc trên, chúng ta đã tích hợp thành công mô hình ReAct vào framework. Cải tiến cốt lõi nằm ở việc tận dụng giao diện `ToolRegistry` thống nhất và cải thiện tính ổn định trong việc thực thi vòng lặp suy nghĩ-hành động của agent thông qua một mẫu prompt có tính cấu hình, chặt chẽ hơn. Đối với các test case của ReAct, do cần gọi công cụ, code kiểm thử được cung cấp ở cuối tài liệu.

### 7.4.3 ReflectionAgent

Vì các loại Agent này đã hiện thực logic cốt lõi ở Chương 4, ở đây chỉ cung cấp các Prompt tương ứng. Khác với các prompt chuyên dụng cho việc sinh code ở Chương 4, phiên bản framework áp dụng thiết kế tổng quát hóa, khiến nó phù hợp với nhiều tình huống khác nhau như sinh văn bản, phân tích và sáng tạo, đồng thời hỗ trợ tùy biến sâu bởi người dùng thông qua tham số `custom_prompts`.

```python
DEFAULT_PROMPTS = {
    "initial": """
Please complete the task according to the following requirements:

Task: {task}

Please provide a complete and accurate answer.
""",
    "reflect": """
Please carefully review the following answer and identify possible problems or areas for improvement:

# Original Task:
{task}

# Current Answer:
{content}

Please analyze the quality of this answer, point out deficiencies, and provide specific improvement suggestions.
If the answer is already good, please respond "No improvement needed".
""",
    "refine": """
Please improve your answer based on the feedback:

# Original Task:
{task}

# Previous Answer:
{last_attempt}

# Feedback:
{feedback}

Please provide an improved answer.
"""
}
```

Bạn có thể thử xây dựng MyReflectionAgent của riêng mình dựa trên code từ Chương 4 và hiện thực ReAct ở trên. Dưới đây là một đoạn code kiểm thử để xác minh ý tưởng.

```python
# test_reflection_agent.py
from dotenv import load_dotenv
from hello_agents import HelloAgentsLLM
from my_reflection_agent import MyReflectionAgent

load_dotenv()
llm = HelloAgentsLLM()

# Dùng các prompt tổng quát mặc định
general_agent = MyReflectionAgent(name="My Reflection Assistant", llm=llm)

# Dùng các prompt sinh code tùy chỉnh (tương tự Chương 4)
code_prompts = {
    "initial": "You are a Python expert, please write a function: {task}",
    "reflect": "Please review the algorithm efficiency of the code:\nTask: {task}\nCode: {content}",
    "refine": "Please optimize the code based on feedback:\nTask: {task}\nFeedback: {feedback}"
}
code_agent = MyReflectionAgent(
    name="My Code Generation Assistant",
    llm=llm,
    custom_prompts=code_prompts
)

# Test cách sử dụng
result = general_agent.run("Write a short article about the development history of artificial intelligence")
print(f"Final result: {result}")
```

### 7.4.4 PlanAndSolveAgent

Khác với output kế hoạch dạng văn bản tự do ở Chương 4, phiên bản framework bắt buộc Planner xuất kế hoạch ở định dạng danh sách Python và cung cấp một cơ chế xử lý ngoại lệ hoàn chỉnh để đảm bảo các bước tiếp theo được thực thi ổn định. Prompt Plan-and-Solve dựa trên framework:

````bash
# Mẫu prompt planner mặc định
DEFAULT_PLANNER_PROMPT = """
You are a top AI planning expert. Your task is to decompose complex problems raised by users into an action plan consisting of multiple simple steps.
Please ensure that each step in the plan is an independent, executable subtask and is strictly arranged in logical order.
Your output must be a Python list, where each element is a string describing a subtask.

Question: {question}

Please output your plan strictly in the following format:
```python
["Step 1", "Step 2", "Step 3", ...]
```
"""

# Mẫu prompt executor mặc định
DEFAULT_EXECUTOR_PROMPT = """
You are a top AI execution expert. Your task is to solve problems step by step strictly according to the given plan.
You will receive the original question, the complete plan, and the steps and results completed so far.
Please focus on solving the "current step" and only output the final answer for that step, without any additional explanations or dialogue.

# Original Question:
{question}

# Complete Plan:
{plan}

# Historical Steps and Results:
{history}

# Current Step:
{current_step}

Please only output the answer for the "current step":
"""
````

Phần này vẫn cung cấp một file kiểm thử toàn diện `test_plan_solve_agent.py`, mà bạn có thể tự thiết kế và hiện thực.

```python
# test_plan_solve_agent.py
from dotenv import load_dotenv
from hello_agents.core.llm import HelloAgentsLLM
from my_plan_solve_agent import MyPlanAndSolveAgent

# Nạp các biến môi trường
load_dotenv()

# Tạo instance LLM
llm = HelloAgentsLLM()

# Tạo PlanAndSolveAgent tùy chỉnh
agent = MyPlanAndSolveAgent(
    name="My Planning Execution Assistant",
    llm=llm
)

# Test bài toán phức tạp
question = "A fruit store sold 15 apples on Monday. The number of apples sold on Tuesday was twice that of Monday. The number sold on Wednesday was 5 less than Tuesday. How many apples were sold in total over these three days?"

result = agent.run(question)
print(f"\nFinal result: {result}")

# Xem lịch sử hội thoại
print(f"Conversation history: {len(agent.get_history())} messages")
```

Cuối cùng, bạn có thể thêm một prompt mới và thử hiện thực `custom_prompt` để nạp các prompt tùy chỉnh.

```python
# Tạo các prompt tùy chỉnh chuyên dụng cho các bài toán toán học
math_prompts = {
    "planner": """
You are a math problem planning expert. Please decompose the math problem into calculation steps:

Question: {question}

Output format:
python
["Calculation step 1", "Calculation step 2", "Sum total"]

""",
    "executor": """
You are a math calculation expert. Please calculate the current step:

Question: {question}
Plan: {plan}
History: {history}
Current step: {current_step}

Please only output the numerical result:
"""
}

# Tạo Agent chuyên về toán học dùng các prompt tùy chỉnh
math_agent = MyPlanAndSolveAgent(
    name="Math Calculation Assistant",
    llm=llm,
    custom_prompts=math_prompts
)

# Test bài toán toán học
math_result = math_agent.run(question)
print(f"Math-specific Agent result: {math_result}")
```

Như thể hiện trong Bảng 7.2, thông qua việc tái cấu trúc theo hướng framework này, chúng ta không chỉ giữ nguyên chức năng cốt lõi của các mô hình Agent khác nhau từ Chương 4 mà còn cải thiện đáng kể tổ chức code, khả năng bảo trì và khả năng mở rộng. Tất cả các Agent giờ đây đều chia sẻ một hạ tầng thống nhất trong khi vẫn giữ được các đặc điểm và ưu điểm riêng.

<div align="center">
  <p>Bảng 7.2 So sánh các Hiện thực Agent qua các Chương</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/7-figures/table-02.png" alt="" width="90%"/>
</div>

### 7.4.5 FunctionCallAgent

FunctionCallAgent là một Agent được giới thiệu trong hello-agents sau phiên bản 0.2.8, dựa trên cơ chế gọi hàm (function calling) nguyên bản của OpenAI. Nó minh họa cách xây dựng một Agent bằng khả năng gọi hàm của OpenAI.
Nó hỗ trợ các tính năng sau:

- _build_tool_schemas: Xây dựng schema gọi hàm của OpenAI thông qua mô tả công cụ
- _extract_message_content: Trích xuất nội dung văn bản từ phản hồi của OpenAI
- _parse_function_call_arguments: Phân tích các tham số dạng chuỗi JSON do mô hình trả về
- _convert_parameter_types: Chuyển đổi kiểu tham số

Những tính năng này cho phép sử dụng khả năng OpenAI Function Calling nguyên bản, cung cấp tính robust (mạnh mẽ, chịu lỗi) tốt hơn so với các cách tiếp cận bị ràng buộc bởi prompt.

```python
def _invoke_with_tools(self, messages: list[dict[str, Any]], tools: list[dict[str, Any]], tool_choice: Union[str, dict], **kwargs):
        """Gọi client OpenAI ở tầng dưới để thực thi các lệnh gọi hàm"""
        client = getattr(self.llm, "_client", None)
        if client is None:
            raise RuntimeError("HelloAgentsLLM client not properly initialized, cannot execute function calls.")

        client_kwargs = dict(kwargs)
        client_kwargs.setdefault("temperature", self.llm.temperature)
        if self.llm.max_tokens is not None:
            client_kwargs.setdefault("max_tokens", self.llm.max_tokens)

        return client.chat.completions.create(
            model=self.llm.model,
            messages=messages,
            tools=tools,
            tool_choice=tool_choice,
            **client_kwargs,
        )

# Logic nội bộ bao bọc function calling nguyên bản của OpenAI
# Ví dụ function calling nguyên bản của OpenAI
from openai import OpenAI
client = OpenAI()
tools = [
  {
    "type": "function",
    "function": {
      "name": "get_current_weather",
      "description": "Get the current weather in a given location",
      "parameters": {
        "type": "object",
        "properties": {
          "location": {
            "type": "string",
            "description": "The city and state, e.g. San Francisco, CA",
          },
          "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]},
        },
        "required": ["location"],
      },
    }
  }
]
messages = [{"role": "user", "content": "What's the weather like in Boston today?"}]
completion = client.chat.completions.create(
  model="gpt-5",
  messages=messages,
  tools=tools,
  tool_choice="auto"
)

print(completion)
```

## 7.5 Hệ thống Công cụ

Nội dung của phần này sẽ khám phá sâu về thiết kế và hiện thực của hệ thống công cụ dựa trên hạ tầng Agent đã được xây dựng ở phần trước. Chúng ta sẽ bắt đầu từ việc xây dựng hạ tầng và dần đi sâu vào thiết kế phát triển tùy chỉnh. Mục tiêu học tập của phần này xoay quanh ba khía cạnh cốt lõi sau:

1. **Trừu tượng hóa và quản lý công cụ thống nhất**: Thiết lập một lớp cơ sở Tool chuẩn hóa và cơ chế đăng ký ToolRegistry để cung cấp hạ tầng thống nhất cho việc phát triển, đăng ký, khám phá và thực thi công cụ.

2. **Phát triển công cụ theo hướng thực hành**: Lấy công cụ tính toán toán học làm ví dụ điển hình, minh họa cách thiết kế và hiện thực các công cụ tùy chỉnh, cho phép bạn đọc nắm được toàn bộ quá trình phát triển công cụ.

3. **Các chiến lược tích hợp và tối ưu nâng cao**: Thông qua thiết kế công cụ tìm kiếm đa nguồn, minh họa cách tích hợp nhiều dịch vụ bên ngoài, hiện thực việc lựa chọn backend thông minh, hợp nhất kết quả và chịu lỗi, phản ánh tư duy thiết kế của hệ thống công cụ trong các tình huống phức tạp.

### 7.5.1 Thiết kế Lớp cơ sở Tool và Cơ chế Đăng ký

Khi xây dựng một hệ thống công cụ có thể mở rộng, trước tiên chúng ta cần thiết lập một tập hợp hạ tầng chuẩn hóa. Hạ tầng này bao gồm lớp cơ sở Tool, registry ToolRegistry và các cơ chế quản lý công cụ.

(1) Thiết kế Trừu tượng của Lớp cơ sở Tool

Lớp cơ sở Tool là trừu tượng cốt lõi của toàn bộ hệ thống công cụ, định nghĩa các đặc tả giao diện mà tất cả các công cụ phải tuân theo:

````python
class Tool(ABC):
    """Lớp cơ sở Tool"""

    def __init__(self, name: str, description: str):
        self.name = name
        self.description = description

    @abstractmethod
    def run(self, parameters: Dict[str, Any]) -> str:
        """Thực thi công cụ"""
        pass

    @abstractmethod
    def get_parameters(self) -> List[ToolParameter]:
        """Lấy các định nghĩa tham số của công cụ"""
        pass
````
Thiết kế này thể hiện ý tưởng cốt lõi của thiết kế hướng đối tượng: thông qua giao diện phương thức `run` thống nhất, tất cả các công cụ đều có thể được thực thi theo một cách nhất quán, nhận tham số dạng dictionary và trả về kết quả dạng chuỗi, đảm bảo tính nhất quán của framework. Đồng thời, các công cụ có khả năng tự mô tả. Thông qua phương thức `get_parameters`, chúng có thể cho biết rõ ràng với bên gọi rằng chúng cần những tham số nào. Cơ chế nội quan (introspection) này cung cấp nền tảng cho việc tạo tài liệu tự động và xác thực tham số. Việc thiết kế các metadata như name và description mang lại cho hệ thống công cụ khả năng khám phá (discoverability) và khả năng hiểu (understandability) tốt.

(2) Hệ thống Định nghĩa Tham số ToolParameter

Để hỗ trợ việc xác thực tham số phức tạp và tạo tài liệu, chúng ta thiết kế lớp ToolParameter:

````python
class ToolParameter(BaseModel):
    """Định nghĩa tham số của công cụ"""
    name: str
    type: str
    description: str
    required: bool = True
    default: Any = None
````
Thiết kế này cho phép các công cụ mô tả chính xác các yêu cầu tham số của chúng, hỗ trợ kiểm tra kiểu, thiết lập giá trị mặc định và tạo tài liệu tự động.

(3) Hiện thực ToolRegistry

ToolRegistry là trung tâm quản lý của hệ thống công cụ, cung cấp các chức năng cốt lõi như đăng ký, khám phá và thực thi công cụ. Trong phần này, chúng ta chủ yếu dùng các chức năng sau:

````python
class ToolRegistry:
    """Registry công cụ của HelloAgents"""

    def __init__(self):
        self._tools: dict[str, Tool] = {}
        self._functions: dict[str, dict[str, Any]] = {}

    def register_tool(self, tool: Tool):
        """Đăng ký đối tượng Tool"""
        if tool.name in self._tools:
            print(f"⚠️ Warning: Tool '{tool.name}' already exists and will be overwritten.")
        self._tools[tool.name] = tool
        print(f"✅ Tool '{tool.name}' registered.")

    def register_function(self, name: str, description: str, func: Callable[[str], str]):
        """
        Trực tiếp đăng ký một hàm như một công cụ (phương thức tiện lợi)

        Args:
            name: Tên công cụ
            description: Mô tả công cụ
            func: Hàm công cụ, nhận tham số chuỗi, trả về kết quả chuỗi
        """
        if name in self._functions:
            print(f"⚠️ Warning: Tool '{name}' already exists and will be overwritten.")

        self._functions[name] = {
            "description": description,
            "func": func
        }
        print(f"✅ Tool '{name}' registered.")
````
ToolRegistry hỗ trợ hai cách đăng ký:

1. **Đăng ký đối tượng Tool**: Phù hợp với các công cụ phức tạp, hỗ trợ định nghĩa và xác thực tham số đầy đủ
2. **Đăng ký hàm trực tiếp**: Phù hợp với các công cụ đơn giản, nhanh chóng tích hợp các hàm có sẵn

(4) Cơ chế Khám phá và Quản lý Công cụ

Registry cung cấp các chức năng quản lý công cụ phong phú:

````python
def get_tools_description(self) -> str:
    """Lấy chuỗi mô tả đã định dạng của tất cả các công cụ khả dụng"""
    descriptions = []

    # Mô tả đối tượng Tool
    for tool in self._tools.values():
        descriptions.append(f"- {tool.name}: {tool.description}")

    # Mô tả công cụ dạng hàm
    for name, info in self._functions.items():
        descriptions.append(f"- {name}: {info['description']}")

    return "\n".join(descriptions) if descriptions else "No tools available"
````
Chuỗi mô tả được tạo bởi phương thức này có thể được dùng trực tiếp để xây dựng prompt của Agent, cho Agent biết những công cụ nào đang khả dụng.

### 7.5.2 Phát triển Công cụ Tùy chỉnh

Với hạ tầng đã sẵn sàng, hãy xem cách phát triển một công cụ tùy chỉnh hoàn chỉnh. Công cụ tính toán toán học là một ví dụ tốt vì nó đơn giản và trực quan. Cách trực tiếp nhất là dùng chức năng đăng ký hàm của ToolRegistry.

Hãy tạo một công cụ tính toán toán học tùy chỉnh. Trước tiên, tạo `my_calculator_tool.py` trong thư mục dự án của bạn:

```python
# my_calculator_tool.py
import ast
import operator
import math
from hello_agents import ToolRegistry

def my_calculate(expression: str) -> str:
    """Hàm tính toán toán học đơn giản"""
    if not expression.strip():
        return "Calculation expression cannot be empty"

    # Các phép toán cơ bản được hỗ trợ
    operators = {
        ast.Add: operator.add,      # +
        ast.Sub: operator.sub,      # -
        ast.Mult: operator.mul,     # *
        ast.Div: operator.truediv,  # /
    }

    # Các hàm cơ bản được hỗ trợ
    functions = {
        'sqrt': math.sqrt,
        'pi': math.pi,
    }

    try:
        node = ast.parse(expression, mode='eval')
        result = _eval_node(node.body, operators, functions)
        return str(result)
    except:
        return "Calculation failed, please check expression format"

def _eval_node(node, operators, functions):
    """Đánh giá biểu thức đơn giản hóa"""
    if isinstance(node, ast.Constant):
        return node.value
    elif isinstance(node, ast.BinOp):
        left = _eval_node(node.left, operators, functions)
        right = _eval_node(node.right, operators, functions)
        op = operators.get(type(node.op))
        return op(left, right)
    elif isinstance(node, ast.Call):
        func_name = node.func.id
        if func_name in functions:
            args = [_eval_node(arg, operators, functions) for arg in node.args]
            return functions[func_name](*args)
    elif isinstance(node, ast.Name):
        if node.id in functions:
            return functions[node.id]

def create_calculator_registry():
    """Tạo registry công cụ chứa calculator"""
    registry = ToolRegistry()

    # Đăng ký hàm calculator
    registry.register_function(
        name="my_calculator",
        description="Simple mathematical calculation tool, supports basic operations (+,-,*,/) and sqrt function",
        func=my_calculate
    )

    return registry
```

Công cụ không chỉ hỗ trợ các phép toán số học cơ bản mà còn bao gồm các hàm và hằng số toán học thường dùng, đáp ứng nhu cầu của hầu hết các tình huống tính toán. Bạn cũng có thể tự mở rộng file này để tạo một hàm tính toán hoàn chỉnh hơn. Chúng tôi cung cấp một file kiểm thử `test_my_calculator.py` để giúp bạn xác minh chức năng:

```python
# test_my_calculator.py
from dotenv import load_dotenv
from my_calculator_tool import create_calculator_registry

# Nạp các biến môi trường
load_dotenv()

def test_calculator_tool():
    """Test công cụ calculator tùy chỉnh"""

    # Tạo registry chứa calculator
    registry = create_calculator_registry()

    print("🧪 Testing Custom Calculator Tool\n")

    # Các test case đơn giản
    test_cases = [
        "2 + 3",           # Phép cộng cơ bản
        "10 - 4",          # Phép trừ cơ bản
        "5 * 6",           # Phép nhân cơ bản
        "15 / 3",          # Phép chia cơ bản
        "sqrt(16)",        # Căn bậc hai
    ]

    for i, expression in enumerate(test_cases, 1):
        print(f"Test {i}: {expression}")
        result = registry.execute_tool("my_calculator", expression)
        print(f"Result: {result}\n")

def test_with_simple_agent():
    """Test tích hợp với SimpleAgent"""
    from hello_agents import HelloAgentsLLM

    # Tạo client LLM
    llm = HelloAgentsLLM()

    # Tạo registry chứa calculator
    registry = create_calculator_registry()

    print("🤖 Integration Test with SimpleAgent:")

    # Mô phỏng tình huống SimpleAgent dùng công cụ
    user_question = "Please help me calculate sqrt(16) + 2 * 3"

    print(f"User question: {user_question}")

    # Dùng công cụ để tính toán
    calc_result = registry.execute_tool("my_calculator", "sqrt(16) + 2 * 3")
    print(f"Calculation result: {calc_result}")

    # Xây dựng câu trả lời cuối cùng
    final_messages = [
        {"role": "user", "content": f"The calculation result is {calc_result}, please answer the user's question in natural language: {user_question}"}
    ]

    print("\n🎯 SimpleAgent's answer:")
    response = llm.think(final_messages)
    for chunk in response:
        print(chunk, end="", flush=True)
    print("\n")

if __name__ == "__main__":
    test_calculator_tool()
    test_with_simple_agent()
```

Thông qua ví dụ về công cụ tính toán toán học đơn giản hóa này, chúng ta đã học được cách nhanh chóng phát triển các công cụ tùy chỉnh: viết một hàm tính toán đơn giản, đăng ký nó thông qua ToolRegistry, rồi tích hợp nó với SimpleAgent. Để quan sát trực quan hơn, Hình 7.1 được cung cấp ở đây để hiểu rõ logic vận hành của code.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/7-figures/01.png" alt="" width="90%"/>
  <p>Hình 7.1 Quy trình làm việc của SimpleAgent dựa trên HelloAgents</p>
</div>

### 7.5.3 Công cụ Tìm kiếm Đa nguồn

Trong các ứng dụng thực tế, chúng ta thường cần tích hợp nhiều dịch vụ bên ngoài để cung cấp chức năng mạnh mẽ hơn. Công cụ tìm kiếm là một ví dụ điển hình, tích hợp nhiều công cụ tìm kiếm (search engine) để cung cấp thông tin thực tế đầy đủ hơn. Ở Chương 1, chúng ta đã dùng API tìm kiếm của Tavily, và ở Chương 4, chúng ta đã dùng API tìm kiếm của SerpApi. Vì vậy, lần này chúng ta dùng hai API này để hiện thực chức năng tìm kiếm đa nguồn. Nếu bạn chưa cài đặt các dependency Python tương ứng, bạn có thể chạy script sau:

```bash
pip install "hello-agents[search]==0.1.1"
```

(1) Thiết kế Giao diện Thống nhất cho Công cụ Tìm kiếm

SearchTool tích hợp sẵn trong framework HelloAgents minh họa cách thiết kế một công cụ tìm kiếm đa nguồn nâng cao:

````python
class SearchTool(Tool):
    """
    Công cụ tìm kiếm lai (hybrid) thông minh

    Hỗ trợ nhiều backend công cụ tìm kiếm, lựa chọn thông minh nguồn tìm kiếm tốt nhất:
    1. Chế độ lai (hybrid) - Lựa chọn thông minh TAVILY hoặc SERPAPI
    2. Tavily API (tavily) - Tìm kiếm AI chuyên nghiệp
    3. SerpApi (serpapi) - Tìm kiếm Google truyền thống
    """

    def __init__(self, backend: str = "hybrid", tavily_key: Optional[str] = None, serpapi_key: Optional[str] = None):
        super().__init__(
            name="search",
            description="An intelligent web search engine. Supports hybrid search mode, automatically selects the best search source."
        )
        self.backend = backend
        self.tavily_key = tavily_key or os.getenv("TAVILY_API_KEY")
        self.serpapi_key = serpapi_key or os.getenv("SERPAPI_API_KEY")
        self.available_backends = []
        self._setup_backends()
````
Ý tưởng cốt lõi của thiết kế này là tự động lựa chọn backend tìm kiếm tốt nhất dựa trên các API key và thư viện dependency khả dụng.

(2) Chiến lược Tích hợp các Nguồn Tìm kiếm TAVILY và SERPAPI

Framework hiện thực logic lựa chọn backend thông minh:

````python
def _search_hybrid(self, query: str) -> str:
    """Tìm kiếm lai - lựa chọn thông minh nguồn tìm kiếm tốt nhất"""
    # Ưu tiên Tavily (tìm kiếm được tối ưu cho AI)
    if "tavily" in self.available_backends:
        try:
            return self._search_tavily(query)
        except Exception as e:
            print(f"⚠️ Tavily search failed: {e}")
            # Nếu Tavily thất bại, thử SerpApi
            if "serpapi" in self.available_backends:
                print("🔄 Switching to SerpApi search")
                return self._search_serpapi(query)

    # Nếu Tavily không khả dụng, dùng SerpApi
    elif "serpapi" in self.available_backends:
        try:
            return self._search_serpapi(query)
        except Exception as e:
            print(f"⚠️ SerpApi search failed: {e}")

    # Nếu cả hai đều không khả dụng, nhắc người dùng cấu hình API
    return "❌ No available search sources, please configure TAVILY_API_KEY or SERPAPI_API_KEY environment variables"
````
Thiết kế này thể hiện khái niệm cốt lõi của các hệ thống có tính sẵn sàng cao (high-availability): thông qua cơ chế suy giảm (degradation), hệ thống có thể suy giảm dần từ nguồn tìm kiếm tối ưu sang các giải pháp thay thế khả dụng. Khi tất cả các nguồn tìm kiếm đều không khả dụng, nó nhắc rõ ràng người dùng cấu hình các API key đúng.

(3) Định dạng Thống nhất các Kết quả Tìm kiếm

Các công cụ tìm kiếm khác nhau trả về kết quả ở các định dạng khác nhau. Framework xử lý điều này thông qua một phương thức định dạng thống nhất:

````python
def _search_tavily(self, query: str) -> str:
    """Tìm kiếm bằng Tavily"""
    response = self.tavily_client.search(
        query=query,
        search_depth="basic",
        include_answer=True,
        max_results=3
    )

    result = f"🎯 Tavily AI search results: {response.get('answer', 'No direct answer found')}\n\n"

    for i, item in enumerate(response.get('results', [])[:3], 1):
        result += f"[{i}] {item.get('title', '')}\n"
        result += f"    {item.get('content', '')[:200]}...\n"
        result += f"    Source: {item.get('url', '')}\n\n"

    return result
````

Dựa trên triết lý thiết kế của framework, chúng ta có thể tạo công cụ tìm kiếm nâng cao của riêng mình. Lần này chúng ta dùng cách tiếp cận dựa trên lớp (class-based) để minh họa các phương pháp hiện thực khác nhau. Tạo `my_advanced_search.py`:

```python
# my_advanced_search.py
import os
from typing import Optional, List, Dict, Any
from hello_agents import ToolRegistry

class MyAdvancedSearchTool:
    """
    Lớp công cụ tìm kiếm nâng cao tùy chỉnh
    Minh họa các mẫu thiết kế cho tích hợp đa nguồn và lựa chọn thông minh
    """

    def __init__(self):
        self.name = "my_advanced_search"
        self.description = "Intelligent search tool, supports multiple search sources, automatically selects best results"
        self.search_sources = []
        self._setup_search_sources()

    def _setup_search_sources(self):
        """Thiết lập các nguồn tìm kiếm khả dụng"""
        # Kiểm tra tính khả dụng của Tavily
        if os.getenv("TAVILY_API_KEY"):
            try:
                from tavily import TavilyClient
                self.tavily_client = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))
                self.search_sources.append("tavily")
                print("✅ Tavily search source enabled")
            except ImportError:
                print("⚠️ Tavily library not installed")

        # Kiểm tra tính khả dụng của SerpApi
        if os.getenv("SERPAPI_API_KEY"):
            try:
                import serpapi
                self.search_sources.append("serpapi")
                print("✅ SerpApi search source enabled")
            except ImportError:
                print("⚠️ SerpApi library not installed")

        if self.search_sources:
            print(f"🔧 Available search sources: {', '.join(self.search_sources)}")
        else:
            print("⚠️ No available search sources, please configure API keys")

    def search(self, query: str) -> str:
        """Thực thi tìm kiếm thông minh"""
        if not query.strip():
            return "❌ Error: Search query cannot be empty"

        # Kiểm tra xem có nguồn tìm kiếm khả dụng không
        if not self.search_sources:
            return """❌ No available search sources, please configure one of the following API keys:

1. Tavily API: Set environment variable TAVILY_API_KEY
   Get it at: https://tavily.com/

2. SerpAPI: Set environment variable SERPAPI_API_KEY
   Get it at: https://serpapi.com/

Restart the program after configuration."""

        print(f"🔍 Starting intelligent search: {query}")

        # Thử nhiều nguồn tìm kiếm, trả về kết quả tốt nhất
        for source in self.search_sources:
            try:
                if source == "tavily":
                    result = self._search_with_tavily(query)
                    if result and "not found" not in result.lower():
                        return f"📊 Tavily AI search results:\n\n{result}"

                elif source == "serpapi":
                    result = self._search_with_serpapi(query)
                    if result and "not found" not in result.lower():
                        return f"🌐 SerpApi Google search results:\n\n{result}"

            except Exception as e:
                print(f"⚠️ {source} search failed: {e}")
                continue

        return "❌ All search sources failed, please check network connection and API key configuration"

    def _search_with_tavily(self, query: str) -> str:
        """Tìm kiếm bằng Tavily"""
        response = self.tavily_client.search(query=query, max_results=3)

        if response.get('answer'):
            result = f"💡 AI direct answer: {response['answer']}\n\n"
        else:
            result = ""

        result += "🔗 Related results:\n"
        for i, item in enumerate(response.get('results', [])[:3], 1):
            result += f"[{i}] {item.get('title', '')}\n"
            result += f"    {item.get('content', '')[:150]}...\n\n"

        return result

    def _search_with_serpapi(self, query: str) -> str:
        """Tìm kiếm bằng SerpApi"""
        import serpapi

        search = serpapi.GoogleSearch({
            "q": query,
            "api_key": os.getenv("SERPAPI_API_KEY"),
            "num": 3
        })

        results = search.get_dict()

        result = "🔗 Google search results:\n"
        if "organic_results" in results:
            for i, res in enumerate(results["organic_results"][:3], 1):
                result += f"[{i}] {res.get('title', '')}\n"
                result += f"    {res.get('snippet', '')}\n\n"

        return result

def create_advanced_search_registry():
    """Tạo registry chứa công cụ tìm kiếm nâng cao"""
    registry = ToolRegistry()

    # Tạo instance công cụ tìm kiếm
    search_tool = MyAdvancedSearchTool()

    # Đăng ký phương thức của công cụ tìm kiếm như một hàm
    registry.register_function(
        name="advanced_search",
        description="Advanced search tool, integrates Tavily and SerpAPI multiple search sources, provides more comprehensive search results",
        func=search_tool.search
    )

    return registry
```

Tiếp theo, chúng ta có thể test công cụ mà chính chúng ta đã viết. Tạo `test_advanced_search.py`:

```python
# test_advanced_search.py
from dotenv import load_dotenv
from my_advanced_search import create_advanced_search_registry, MyAdvancedSearchTool

# Nạp các biến môi trường
load_dotenv()

def test_advanced_search():
    """Test công cụ tìm kiếm nâng cao"""

    # Tạo registry chứa công cụ tìm kiếm nâng cao
    registry = create_advanced_search_registry()

    print("🔍 Testing Advanced Search Tool\n")

    # Các truy vấn test
    test_queries = [
        "History of Python programming language",
        "Latest developments in artificial intelligence",
        "2024 technology trends"
    ]

    for i, query in enumerate(test_queries, 1):
        print(f"Test {i}: {query}")
        result = registry.execute_tool("advanced_search", query)
        print(f"Result: {result}\n")
        print("-" * 60 + "\n")

def test_api_configuration():
    """Test kiểm tra cấu hình API"""
    print("🔧 Testing API Configuration Check:")

    # Trực tiếp tạo instance công cụ tìm kiếm
    search_tool = MyAdvancedSearchTool()

    # Nếu API chưa được cấu hình, sẽ hiển thị lời nhắc cấu hình
    result = search_tool.search("machine learning algorithms")
    print(f"Search result: {result}")

def test_with_agent():
    """Test tích hợp với Agent"""
    print("\n🤖 Integration Test with Agent:")
    print("Advanced search tool is ready and can be integrated with Agent")

    # Hiển thị mô tả công cụ
    registry = create_advanced_search_registry()
    tools_desc = registry.get_tools_description()
    print(f"Tool description:\n{tools_desc}")

if __name__ == "__main__":
    test_advanced_search()
    test_api_configuration()
    test_with_agent()
```

Thông qua thực hành thiết kế công cụ tìm kiếm nâng cao này, chúng ta đã học được cách dùng các lớp (class) để xây dựng các hệ thống công cụ phức tạp. So với cách tiếp cận dạng hàm, cách tiếp cận dạng lớp phù hợp hơn với các công cụ cần duy trì trạng thái (state) (chẳng hạn như API client, thông tin cấu hình).

### 7.5.4 Các Tính năng Nâng cao của Hệ thống Công cụ

Sau khi nắm được việc phát triển công cụ cơ bản và tích hợp đa nguồn, hãy khám phá các tính năng nâng cao của hệ thống công cụ. Những tính năng này giúp hệ thống công cụ chạy ổn định trong các môi trường production phức tạp và cung cấp các khả năng mạnh mẽ hơn cho các Agent.

(1) Cơ chế Gọi theo Chuỗi Công cụ (Tool Chain)

Trong các ứng dụng thực tế, các Agent thường cần kết hợp nhiều công cụ để hoàn thành các tác vụ phức tạp. Chúng ta có thể thiết kế một trình quản lý chuỗi công cụ (tool chain manager) để hỗ trợ tình huống này, mượn khái niệm đồ thị (graph) đã đề cập ở Chương 6:

```python
# tool_chain_manager.py
from typing import List, Dict, Any, Optional
from hello_agents import ToolRegistry

class ToolChain:
    """Chuỗi công cụ - hỗ trợ thực thi tuần tự nhiều công cụ"""

    def __init__(self, name: str, description: str):
        self.name = name
        self.description = description
        self.steps: List[Dict[str, Any]] = []

    def add_step(self, tool_name: str, input_template: str, output_key: str = None):
        """
        Thêm bước thực thi công cụ

        Args:
            tool_name: Tên công cụ
            input_template: Mẫu đầu vào, hỗ trợ thay thế biến
            output_key: Tên khóa cho kết quả đầu ra, dùng để tham chiếu trong các bước sau
        """
        self.steps.append({
            "tool_name": tool_name,
            "input_template": input_template,
            "output_key": output_key or f"step_{len(self.steps)}_result"
        })

    def execute(self, registry: ToolRegistry, initial_input: str, context: Dict[str, Any] = None) -> str:
        """Thực thi chuỗi công cụ"""
        context = context or {}
        context["input"] = initial_input

        print(f"🔗 Starting tool chain execution: {self.name}")

        for i, step in enumerate(self.steps, 1):
            tool_name = step["tool_name"]
            input_template = step["input_template"]
            output_key = step["output_key"]

            # Thay thế các biến trong mẫu
            try:
                tool_input = input_template.format(**context)
            except KeyError as e:
                return f"❌ Tool chain execution failed: Template variable {e} not found"

            print(f"  Step {i}: Using {tool_name} to process '{tool_input[:50]}...'")

            # Thực thi công cụ
            result = registry.execute_tool(tool_name, tool_input)
            context[output_key] = result

            print(f"  ✅ Step {i} completed, result length: {len(result)} characters")

        # Trả về kết quả của bước cuối cùng
        final_result = context[self.steps[-1]["output_key"]]
        print(f"🎉 Tool chain '{self.name}' execution completed")
        return final_result

class ToolChainManager:
    """Trình quản lý chuỗi công cụ"""

    def __init__(self, registry: ToolRegistry):
        self.registry = registry
        self.chains: Dict[str, ToolChain] = {}

    def register_chain(self, chain: ToolChain):
        """Đăng ký chuỗi công cụ"""
        self.chains[chain.name] = chain
        print(f"✅ Tool chain '{chain.name}' registered")

    def execute_chain(self, chain_name: str, input_data: str, context: Dict[str, Any] = None) -> str:
        """Thực thi chuỗi công cụ được chỉ định"""
        if chain_name not in self.chains:
            return f"❌ Tool chain '{chain_name}' does not exist"

        chain = self.chains[chain_name]
        return chain.execute(self.registry, input_data, context)

    def list_chains(self) -> List[str]:
        """Liệt kê tất cả các chuỗi công cụ"""
        return list(self.chains.keys())

# Ví dụ sử dụng
def create_research_chain() -> ToolChain:
    """Tạo một chuỗi công cụ nghiên cứu: tìm kiếm -> tính toán -> tổng hợp"""
    chain = ToolChain(
        name="research_and_calculate",
        description="Search for information and perform related calculations"
    )

    # Bước 1: Tìm kiếm thông tin
    chain.add_step(
        tool_name="search",
        input_template="{input}",
        output_key="search_result"
    )

    # Bước 2: Thực hiện tính toán dựa trên kết quả tìm kiếm (nếu cần)
    chain.add_step(
        tool_name="my_calculator",
        input_template="Calculate relevant values based on the following information: {search_result}",
        output_key="calculation_result"
    )

    return chain
```

(2) Hỗ trợ Thực thi Công cụ Bất đồng bộ

Đối với các thao tác công cụ tốn thời gian, chúng ta có thể cung cấp hỗ trợ thực thi bất đồng bộ (asynchronous):

```python
# async_tool_executor.py
import asyncio
import concurrent.futures
from typing import Dict, Any, List, Callable
from hello_agents import ToolRegistry

class AsyncToolExecutor:
    """Bộ thực thi công cụ bất đồng bộ"""

    def __init__(self, registry: ToolRegistry, max_workers: int = 4):
        self.registry = registry
        self.executor = concurrent.futures.ThreadPoolExecutor(max_workers=max_workers)

    async def execute_tool_async(self, tool_name: str, input_data: str) -> str:
        """Thực thi bất đồng bộ một công cụ đơn"""
        loop = asyncio.get_event_loop()

        def _execute():
            return self.registry.execute_tool(tool_name, input_data)

        result = await loop.run_in_executor(self.executor, _execute)
        return result

    async def execute_tools_parallel(self, tasks: List[Dict[str, str]]) -> List[str]:
        """Thực thi nhiều công cụ song song"""
        print(f"🚀 Starting parallel execution of {len(tasks)} tool tasks")

        # Tạo các tác vụ bất đồng bộ
        async_tasks = []
        for task in tasks:
            tool_name = task["tool_name"]
            input_data = task["input_data"]
            async_task = self.execute_tool_async(tool_name, input_data)
            async_tasks.append(async_task)

        # Chờ tất cả các tác vụ hoàn thành
        results = await asyncio.gather(*async_tasks)

        print(f"✅ All tool tasks completed")
        return results

    def __del__(self):
        """Dọn dẹp tài nguyên"""
        if hasattr(self, 'executor'):
            self.executor.shutdown(wait=True)

# Ví dụ sử dụng
async def test_parallel_execution():
    """Test thực thi công cụ song song"""
    from hello_agents import ToolRegistry

    registry = ToolRegistry()
    # Giả sử các công cụ search và calculator đã được đăng ký

    executor = AsyncToolExecutor(registry)

    # Định nghĩa các tác vụ song song
    tasks = [
        {"tool_name": "search", "input_data": "Python programming"},
        {"tool_name": "search", "input_data": "machine learning"},
        {"tool_name": "my_calculator", "input_data": "2 + 2"},
        {"tool_name": "my_calculator", "input_data": "sqrt(16)"},
    ]

    # Thực thi song song
    results = await executor.execute_tools_parallel(tasks)

    for i, result in enumerate(results):
        print(f"Task {i+1} result: {result[:100]}...")
```

Dựa trên kinh nghiệm thiết kế và hiện thực ở trên, chúng ta có thể tổng kết các khái niệm cốt lõi của việc phát triển hệ thống công cụ: Ở cấp độ thiết kế, mỗi công cụ nên tuân theo nguyên tắc trách nhiệm đơn nhất, tập trung vào chức năng cụ thể trong khi vẫn duy trì tính đồng nhất của giao diện, và coi việc xử lý ngoại lệ toàn diện cùng với xác thực đầu vào ưu tiên bảo mật (security-first) là những yêu cầu cơ bản. Về mặt tối ưu hiệu năng, dùng thực thi bất đồng bộ để cải thiện khả năng xử lý đồng thời, đồng thời quản lý hợp lý các kết nối bên ngoài và tài nguyên hệ thống.



## 7.6 Tổng kết Chương

Trước khi tổng kết chính thức, chúng tôi muốn chia sẻ một tin vui với mọi người: Đối với tất cả các phương thức và chức năng được hiện thực trong chương này, các test case hoàn chỉnh đều được cung cấp trong kho GitHub. Bạn có thể truy cập [liên kết này](https://github.com/datawhalechina/hello-agents/tree/main/code/chapter7) để xem và chạy các đoạn code kiểm thử này. Thư mục này chứa các minh họa của bốn mô hình Agent, các bài kiểm thử tích hợp của hệ thống công cụ, các ví dụ sử dụng các tính năng nâng cao, và các trải nghiệm Agent tương tác. Nếu bạn muốn xác minh xem cách hiện thực của mình có đúng không hoặc muốn hiểu sâu cách sử dụng thực tế của framework, những test case này sẽ là những tài liệu tham khảo giá trị.

Nhìn lại chương này, chúng ta đã hoàn thành một tác vụ đầy thử thách: từng bước một, chúng ta đã xây dựng một framework agent cơ bản—HelloAgents. Quá trình này luôn tuân theo các nguyên tắc cốt lõi "phân tầng tách rời, trách nhiệm đơn nhất, và giao diện thống nhất".

Trong việc hiện thực cụ thể framework, chúng ta đã hiện thực lại bốn mô hình Agent kinh điển. Từ chế độ hội thoại cơ bản của SimpleAgent đến sự kết hợp giữa suy luận và hành động của ReActAgent; từ sự tự phản chiếu và tối ưu lặp của ReflectionAgent đến việc phân rã lập kế hoạch và thực thi theo từng bước của PlanAndSolveAgent. Hệ thống công cụ, với vai trò là cốt lõi của việc mở rộng năng lực Agent, là một thực hành kỹ thuật hoàn chỉnh.

Quan trọng hơn, việc xây dựng Chương 7 không phải là điểm kết thúc mà cung cấp nền tảng kỹ thuật cần thiết cho việc học sâu hơn ở các chương sau. Chúng ta đã cân nhắc đầy đủ khả năng mở rộng của nội dung tiếp theo ngay từ thiết kế ban đầu, dành sẵn các giao diện và điểm mở rộng cần thiết để hiện thực các tính năng nâng cao. Giao diện LLM thống nhất, hệ thống message chuẩn hóa, và cơ chế đăng ký công cụ mà chúng ta đã thiết lập cùng nhau tạo thành một nền tảng kỹ thuật hoàn chỉnh. Điều này cho phép chúng ta bình thản hơn khi học các chủ đề nâng cao hơn ở các chương sau: hệ thống memory và RAG của Chương 8 sẽ mở rộng ranh giới năng lực của Agent dựa trên nền tảng này; kỹ thuật ngữ cảnh (context engineering) của Chương 9 sẽ đi sâu vào cơ chế xử lý message mà chúng ta đã thiết lập; giao thức agent (agent protocol) của Chương 10 sẽ đòi hỏi việc mở rộng các công cụ mới.

Tiếp theo, chúng ta sẽ cùng nhau khám phá cách thêm hệ thống RAG và cơ chế Memory vào framework. Hãy đón chờ Chương 8!


## Bài tập

1. Chương này đã xây dựng framework `HelloAgents` và giải thích "tại sao chúng ta cần tự xây dựng framework Agent". Hãy phân tích:

   - Phần 7.1.1 đã đề cập bốn hạn chế chính của các framework chủ đạo hiện nay. Kết hợp với trải nghiệm thực tế của bạn khi dùng một framework trong [bài tập Chương 6](../chapter6/第六章%20框架开发实践.md#习题) hoặc các dự án thực tế, giải thích những vấn đề này ảnh hưởng đến hiệu suất phát triển như thế nào.
   - `HelloAgents` đề xuất triết lý thiết kế "mọi thứ đều là công cụ", trừu tượng hóa các module như `Memory`, `RAG`, và `MCP` thành các công cụ. Thiết kế này có những ưu điểm gì? Có hạn chế nào không? Hãy cho ví dụ.
   - So sánh code agent được hiện thực từ đầu ở Chương 4 với hiện thực framework ở chương này, framework mang lại những cải tiến cụ thể nào? Nếu bạn phải thiết kế một framework, bạn sẽ ưu tiên những nguyên tắc thiết kế nào?

2. Ở Phần 7.2, chúng ta đã mở rộng `HelloAgentsLLM` để hỗ trợ nhiều nhà cung cấp mô hình và việc gọi mô hình cục bộ.

   > <strong>Gợi ý</strong>: Đây là một bài tập thực hành, khuyến nghị thao tác trực tiếp

   - Tham khảo ví dụ ở Phần 7.2.1, hãy thử thêm hỗ trợ cho một nhà cung cấp mô hình mới vào `HelloAgentsLLM` (chẳng hạn như `Gemini`, `Anthropic`, `Kim`). Hiện thực nó thông qua kế thừa và bật khả năng tự động phát hiện các biến môi trường của nhà cung cấp đó.
   - Phần 7.2.3 đã giới thiệu ba mức ưu tiên của cơ chế phát hiện tự động. Hãy phân tích: Nếu cả `OPENAI_API_KEY` và `LLM_BASE_URL="http://localhost:11434/v1"` đều được đặt, framework cuối cùng sẽ chọn provider nào? Thiết kế ưu tiên này có hợp lý không?
   - Ngoài `VLLM` và `Ollama` được giới thiệu trong chương này, còn có các giải pháp triển khai mô hình cục bộ khác như `SGLang`. Trước tiên hãy tìm kiếm và tìm hiểu thông tin cơ bản cùng đặc điểm của `SGLang`, sau đó so sánh `VLLM`, `SGLang`, và `Ollama` về mặt dễ sử dụng, mức tiêu thụ tài nguyên, tốc độ suy luận và độ chính xác suy luận.

3. Ở Phần 7.3, chúng ta đã hiện thực lớp `Message`, lớp `Config`, và lớp cơ sở `Agent`. Hãy phân tích:

   - Lớp `Message` dùng `BaseModel` của `Pydantic` để xác thực dữ liệu. Thiết kế này có những ưu điểm gì trong các ứng dụng thực tế?
   - Lớp cơ sở `Agent` định nghĩa hai phương thức: `run` và `_execute`, trong đó `run` là giao diện công khai và `_execute` là một phương thức trừu tượng. Mẫu thiết kế này được gọi là gì? Nó có những lợi ích gì?
   - Trong lớp `Config`, chúng ta đã dùng mẫu singleton (singleton pattern). Hãy giải thích mẫu singleton là gì, tại sao việc quản lý cấu hình cần dùng mẫu singleton, và những vấn đề gì sẽ nảy sinh nếu không dùng mẫu singleton.

4. Ở Phần 7.4, chúng ta đã hiện thực bốn mô hình `Agent` theo hướng framework.

   > <strong>Gợi ý</strong>: Đây là một bài tập thực hành, khuyến nghị thao tác trực tiếp

   - So sánh `ReActAgent` được hiện thực từ đầu ở Chương 4 với `ReActAgent` dựa trên framework ở chương này, liệt kê 3 cải tiến cụ thể và giải thích những cải tiến này nâng cao khả năng bảo trì và khả năng mở rộng của code như thế nào.
   - `ReflectionAgent` hiện thực một vòng lặp "thực thi-phản chiếu-tối ưu". Hãy mở rộng hiện thực này bằng cách thêm một cơ chế "chấm điểm chất lượng": Sau mỗi lần phản chiếu, để `LLM` chấm điểm output của phiên bản hiện tại, và chỉ tiếp tục tối ưu nếu điểm số dưới một ngưỡng; nếu không thì kết thúc sớm.
   - Hãy thiết kế và hiện thực một mô hình `Agent` mới gọi là `Tree-of-Thought Agent`, nó phải kế thừa từ lớp cơ sở `Agent` và có khả năng sinh ra nhiều đường tư duy (thinking path) khả dĩ ở mỗi bước, rồi chọn đường tối ưu để tiếp tục.

5. Ở Phần 7.5, chúng ta đã xây dựng hệ thống công cụ. Hãy cân nhắc các câu hỏi sau:

   - Lớp `BaseTool` định nghĩa một phương thức trừu tượng `execute` mà tất cả các công cụ phải hiện thực. Hãy giải thích tại sao tất cả các công cụ nên bị buộc phải hiện thực một giao diện thống nhất. Nếu một công cụ cần trả về nhiều giá trị (chẳng hạn như một công cụ tìm kiếm trả về tiêu đề, tóm tắt và liên kết), nó nên được thiết kế như thế nào?
   - Phần 7.5.3 đã hiện thực chuỗi công cụ (`ToolChain`). Hãy thiết kế một tình huống ứng dụng thực tế cần nối chuỗi ít nhất 3 công cụ và vẽ sơ đồ luồng thực thi của chuỗi công cụ.
   - Bộ thực thi công cụ bất đồng bộ (`AsyncToolExecutor`) dùng một thread pool để thực thi các công cụ song song. Hãy phân tích: Trong những trường hợp nào việc thực thi công cụ song song có thể mang lại cải thiện hiệu năng?

6. Khả năng mở rộng của framework là một trong những cân nhắc quan trọng trong thiết kế. Bây giờ bạn cần mở rộng framework `HelloAgents` để hiện thực một số tính năng và đặc tính mới thú vị.

   - Trước tiên, thêm một tính năng "xuất dạng luồng" (streaming output) vào `HelloAgents` để `Agent` có thể trả về các kết quả trung gian theo thời gian thực khi sinh phản hồi (tương tự như hiệu ứng đánh máy trong giao diện người dùng của `ChatGPT`). Hãy thiết kế phương án hiện thực cho tính năng này và giải thích những lớp và phương thức nào cần được sửa đổi.
   - Sau đó thêm một tính năng "quản lý hội thoại đa lượt" (multi-turn conversation management) vào framework, có thể tự động quản lý lịch sử hội thoại, hỗ trợ phân nhánh (branching) và quay lui (backtracking) hội thoại. Bạn sẽ thiết kế điều này như thế nào? Cần những lớp mới nào? Làm thế nào để tích hợp với hệ thống `Message` hiện có?
   - Cuối cùng, hãy thiết kế một "hệ thống plugin" cho `HelloAgents` cho phép các nhà phát triển bên thứ ba mở rộng chức năng framework thông qua các plugin (chẳng hạn như thêm các loại `Agent` mới, các loại công cụ mới, v.v.) mà không cần sửa đổi code cốt lõi của framework. Hãy vẽ sơ đồ kiến trúc của hệ thống plugin và giải thích các giao diện then chốt.
