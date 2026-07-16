# Chương 4 Xây dựng các Mô hình Agent Kinh điển

Trong chương trước, chúng ta đã tìm hiểu sâu về các mô hình ngôn ngữ lớn với vai trò là "bộ não" của các agent hiện đại. Chúng ta đã học về kiến trúc Transformer bên trong của chúng, các phương pháp tương tác với chúng, và các giới hạn năng lực của chúng. Bây giờ, đã đến lúc biến kiến thức lý thuyết này thành thực hành và tự tay xây dựng các agent.

Năng lực cốt lõi của một agent hiện đại nằm ở khả năng kết nối sức mạnh suy luận của các mô hình ngôn ngữ lớn với thế giới bên ngoài. Nó có thể tự chủ hiểu được ý định của người dùng, phân rã các nhiệm vụ phức tạp, và đạt được mục tiêu bằng cách gọi một loạt các "công cụ" (tool) như trình thông dịch mã nguồn, công cụ tìm kiếm, và API để lấy thông tin và thực thi các thao tác. Tuy nhiên, agent không phải là toàn năng; chúng cũng phải đối mặt với những thách thức từ vấn đề "ảo giác" (hallucination) vốn có trong các mô hình lớn, các vòng lặp suy luận tiềm ẩn trong các nhiệm vụ phức tạp, và việc sử dụng công cụ sai, những điều tạo nên các giới hạn năng lực của agent.

Để tổ chức tốt hơn các quá trình "tư duy" và "hành động" của agent, ngành công nghiệp đã cho ra đời nhiều mô hình kiến trúc kinh điển. Trong chương này, chúng ta sẽ tập trung vào ba mô hình tiêu biểu nhất và triển khai chúng từng bước từ đầu:

- **ReAct (Reasoning and Acting):** Một mô hình kết hợp chặt chẽ giữa "tư duy" và "hành động", cho phép agent vừa làm vừa suy nghĩ và điều chỉnh linh hoạt.
- **Plan-and-Solve:** Một mô hình "suy nghĩ trước khi hành động", trong đó agent trước tiên tạo ra một kế hoạch hành động hoàn chỉnh và sau đó thực thi nó một cách nghiêm ngặt.
- **Reflection:** Một mô hình trang bị cho agent khả năng "phản tư" (reflection), tối ưu hóa kết quả thông qua việc tự phê bình và sửa lỗi.

Sau khi hiểu về những điều này, bạn có thể hỏi: với nhiều framework xuất sắc như LangChain và LlamaIndex đã có sẵn, tại sao lại phải "phát minh lại bánh xe"? Câu trả lời nằm ở chỗ mặc dù các framework trưởng thành có những ưu điểm đáng kể về hiệu suất kỹ thuật, việc sử dụng trực tiếp các công cụ trừu tượng hóa cao không giúp chúng ta hiểu được cơ chế thiết kế cơ sở hoạt động như thế nào hoặc chúng mang lại lợi ích gì. Thứ hai, quá trình này phơi bày các thách thức kỹ thuật trong các dự án. Các framework xử lý nhiều vấn đề cho chúng ta, chẳng hạn như phân tích định dạng đầu ra của mô hình, thử lại các lệnh gọi công cụ thất bại, và ngăn agent rơi vào các vòng lặp vô hạn. Tự tay xử lý những vấn đề này là cách trực tiếp nhất để trau dồi năng lực thiết kế hệ thống. Cuối cùng, và quan trọng nhất, nắm vững các nguyên tắc thiết kế cho phép bạn thực sự chuyển đổi từ một "người dùng" framework thành một "người sáng tạo" ứng dụng thông minh. Khi các thành phần tiêu chuẩn không thể đáp ứng nhu cầu phức tạp của bạn, bạn sẽ có khả năng tùy chỉnh sâu hoặc thậm chí xây dựng một agent hoàn toàn mới từ đầu.

## 4.1 Chuẩn bị Môi trường và Định nghĩa Công cụ Cơ bản

Trước khi bắt đầu xây dựng, chúng ta cần thiết lập môi trường phát triển và định nghĩa một số thành phần cơ bản. Điều này sẽ giúp chúng ta tránh lặp lại công việc và tập trung hơn vào logic cốt lõi khi triển khai các mô hình khác nhau sau này.

### 4.1.1 Cài đặt các Dependency

Phần thực hành của cuốn sách này sẽ chủ yếu sử dụng ngôn ngữ Python, và khuyến nghị dùng Python 3.10 trở lên. Trước tiên, hãy đảm bảo bạn đã cài đặt thư viện `openai` để tương tác với các mô hình ngôn ngữ lớn, và thư viện `python-dotenv` để quản lý an toàn các API key của chúng ta.

Chạy lệnh sau trong terminal của bạn:

```bash
pip install openai python-dotenv
```

### 4.1.2 Cấu hình API Key

Để làm cho mã nguồn của chúng ta phổ quát hơn, chúng ta sẽ cấu hình thống nhất các thông tin liên quan đến dịch vụ mô hình (model ID, API key, địa chỉ dịch vụ) trong các biến môi trường.

1. Trong thư mục gốc của dự án, tạo một file có tên `.env`.
2. Trong file này, thêm nội dung sau. Bạn có thể trỏ nó đến dịch vụ chính thức của OpenAI hoặc bất kỳ dịch vụ cục bộ/bên thứ ba nào tương thích với giao diện OpenAI tùy theo nhu cầu của bạn.
3. Nếu bạn thực sự không biết cách lấy nó, bạn có thể tham khảo [Cấu hình Môi trường](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra07-环境配置.md).

```bash
# .env file
LLM_API_KEY="YOUR-API-KEY"
LLM_MODEL_ID="YOUR-MODEL"
LLM_BASE_URL="YOUR-URL"
```

Mã nguồn của chúng ta sẽ tự động tải các cấu hình này từ file này.

### 4.1.3 Đóng gói Hàm Gọi LLM Cơ bản

Để làm cho cấu trúc mã nguồn rõ ràng hơn và dễ tái sử dụng hơn, hãy định nghĩa một lớp client LLM chuyên dụng. Lớp này sẽ đóng gói tất cả các chi tiết của việc tương tác với các dịch vụ mô hình, cho phép logic chính của chúng ta tập trung hơn vào việc xây dựng agent.

```python
import os
from openai import OpenAI
from dotenv import load_dotenv
from typing import List, Dict

# Load environment variables from .env file
load_dotenv()

class HelloAgentsLLM:
    """
    A customized LLM client for the book "Hello Agents".
    It is used to call any service compatible with the OpenAI interface and uses streaming responses by default.
    """
    def __init__(self, model: str = None, apiKey: str = None, baseUrl: str = None, timeout: int = None):
        """
        Initialize the client. Prioritize passed parameters; if not provided, load from environment variables.
        """
        self.model = model or os.getenv("LLM_MODEL_ID")
        apiKey = apiKey or os.getenv("LLM_API_KEY")
        baseUrl = baseUrl or os.getenv("LLM_BASE_URL")
        timeout = timeout or int(os.getenv("LLM_TIMEOUT", 60))
        
        if not all([self.model, apiKey, baseUrl]):
            raise ValueError("Model ID, API key, and service address must be provided or defined in the .env file.")

        self.client = OpenAI(api_key=apiKey, base_url=baseUrl, timeout=timeout)

    def think(self, messages: List[Dict[str, str]], temperature: float = 0) -> str:
        """
        Call the large language model to think and return its response.
        """
        print(f"🧠 Calling {self.model} model...")
        try:
            response = self.client.chat.completions.create(
                model=self.model,
                messages=messages,
                temperature=temperature,
                stream=True,
            )
            
            # Handle streaming response
            print("✅ Large language model response successful:")
            collected_content = []
            for chunk in response:
                if not chunk.choices:
                    continue
                content = chunk.choices[0].delta.content or ""
                print(content, end="", flush=True)
                collected_content.append(content)
            print()  # Newline after streaming output ends
            return "".join(collected_content)

        except Exception as e:
            print(f"❌ Error occurred when calling LLM API: {e}")
            return None

# --- Client Usage Example ---
if __name__ == '__main__':
    try:
        llmClient = HelloAgentsLLM()
        
        exampleMessages = [
            {"role": "system", "content": "You are a helpful assistant that writes Python code."},
            {"role": "user", "content": "Write a quicksort algorithm"}
        ]
        
        print("--- Calling LLM ---")
        responseText = llmClient.think(exampleMessages)
        if responseText:
            print("\n\n--- Complete Model Response ---")
            print(responseText)

    except ValueError as e:
        print(e)


>>>
--- Calling LLM ---
🧠 Calling xxxxxx model...
✅ Large language model response successful:
Quicksort is a very efficient sorting algorithm...
```



## 4.2 ReAct

Sau khi đã chuẩn bị client LLM, chúng ta sẽ xây dựng mô hình agent đầu tiên và kinh điển nhất: **ReAct (Reason + Act)**. ReAct được đề xuất bởi Shunyu Yao vào năm 2022<sup>[1]</sup>. Ý tưởng cốt lõi của nó là mô phỏng cách con người giải quyết vấn đề bằng cách kết hợp một cách rõ ràng **Suy luận (Reasoning)** và **Hành động (Acting)** để tạo thành một vòng lặp "tư duy-hành động-quan sát".

### 4.2.1 Quy trình Hoạt động của ReAct

Trước khi ReAct xuất hiện, các phương pháp chủ đạo có thể được chia thành hai loại: một là loại "tư duy thuần túy", chẳng hạn như **Chain-of-Thought**, có thể hướng dẫn mô hình thực hiện suy luận logic phức tạp nhưng không thể tương tác với thế giới bên ngoài và dễ mắc phải ảo giác về sự thật; loại còn lại là loại "hành động thuần túy", trong đó mô hình trực tiếp xuất ra các hành động để thực thi nhưng thiếu khả năng lập kế hoạch và sửa lỗi.

Sự khéo léo của ReAct nằm ở việc nhận ra rằng **tư duy và hành động bổ trợ cho nhau**. Tư duy hướng dẫn hành động, trong khi kết quả hành động ngược lại điều chỉnh tư duy. Vì mục đích này, mô hình ReAct sử dụng một kỹ thuật prompt engineering đặc biệt để hướng dẫn mô hình sao cho mỗi bước đầu ra của nó tuân theo một quỹ đạo cố định:

- **Thought (Tư duy):** Đây là "độc thoại nội tâm" của agent. Nó phân tích tình huống hiện tại, phân rã nhiệm vụ, xây dựng kế hoạch tiếp theo, hoặc suy ngẫm về kết quả của bước trước.
- **Action (Hành động):** Đây là hành động cụ thể mà agent quyết định thực hiện, thường là gọi một công cụ bên ngoài, chẳng hạn như `Search['Điện thoại mới nhất của Huawei']`.
- **Observation (Quan sát):** Đây là kết quả được trả về từ công cụ bên ngoài sau khi thực thi `Action`, chẳng hạn như một bản tóm tắt kết quả tìm kiếm hoặc một giá trị trả về từ API.

Agent sẽ liên tục lặp lại vòng lặp **Thought -> Action -> Observation** này, nối các kết quả quan sát mới vào lịch sử để tạo thành một ngữ cảnh liên tục mở rộng, cho đến khi nó xác định trong `Thought` rằng nó đã tìm được câu trả lời cuối cùng và sau đó xuất ra kết quả. Quá trình này tạo thành một sự hiệp lực mạnh mẽ: **suy luận làm cho hành động có mục đích hơn, trong khi hành động cung cấp cơ sở thực tế cho suy luận.**

Chúng ta có thể biểu diễn quá trình này một cách hình thức, như minh họa trong Hình 4.1. Cụ thể, tại mỗi bước thời gian $t$, chính sách của agent (tức là mô hình ngôn ngữ lớn $\pi$) tạo ra tư duy hiện tại $th_t$ và hành động $a_t$ dựa trên câu hỏi ban đầu $q$ và quỹ đạo lịch sử của tất cả các bước "hành động-quan sát" trước đó $((a_1,o_1),\dots,(a_{t-1},o_{t-1}))$:

$$\left(th_t,a_t\right)=\pi\left(q,(a_1,o_1),\ldots,(a_{t-1},o_{t-1})\right)$$

Sau đó, công cụ $T$ trong môi trường thực thi hành động $a_t$ và trả về một kết quả quan sát mới $o_t$:

$$o_t = T(a_t)$$

Vòng lặp này tiếp tục, nối các cặp $(a_t,o_t)$ mới vào lịch sử cho đến khi mô hình xác định trong tư duy $th_t$ rằng nhiệm vụ đã hoàn thành.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-1.png" alt="Think-Act-Observe synergistic loop in ReAct paradigm" width="90%"/>
  <p>Hình 4.1 Vòng lặp Hiệp lực Tư duy-Hành động-Quan sát trong Mô hình ReAct</p>
</div>

Cơ chế này đặc biệt phù hợp với các tình huống sau:

- **Các nhiệm vụ yêu cầu kiến thức bên ngoài**: Chẳng hạn như truy vấn thông tin thời gian thực (thời tiết, tin tức, giá cổ phiếu), tìm kiếm kiến thức trong các lĩnh vực chuyên môn, v.v.
- **Các nhiệm vụ yêu cầu tính toán chính xác**: Ủy thác các bài toán toán học cho các công cụ máy tính để tránh lỗi tính toán của LLM.
- **Các nhiệm vụ yêu cầu tương tác với API**: Chẳng hạn như thao tác cơ sở dữ liệu, gọi API của một dịch vụ để hoàn thành các chức năng cụ thể.

Do đó, chúng ta sẽ xây dựng một agent ReAct có khả năng **sử dụng các công cụ bên ngoài** để trả lời các câu hỏi mà các mô hình ngôn ngữ lớn không thể trực tiếp trả lời chỉ với cơ sở kiến thức của chính chúng. Ví dụ: "Điện thoại mới nhất của Huawei là gì? Những điểm nổi bật chính của nó là gì?" Câu hỏi này yêu cầu agent hiểu rằng nó cần tìm kiếm trực tuyến, gọi công cụ để tìm kiếm kết quả, và tóm tắt câu trả lời.

### 4.2.2 Định nghĩa và Triển khai Công cụ

Nếu các mô hình ngôn ngữ lớn là bộ não của một agent, thì **Công cụ (Tool)** là "tay chân" của nó để tương tác với thế giới bên ngoài. Để mô hình ReAct thực sự giải quyết được các vấn đề mà chúng ta đặt ra, agent cần có khả năng gọi các công cụ bên ngoài.

Đối với mục tiêu được đặt ra trong phần này—trả lời các câu hỏi về "điện thoại mới nhất của Huawei"—chúng ta cần cung cấp cho agent một công cụ tìm kiếm web. Ở đây chúng ta chọn **SerpApi**, cung cấp các kết quả tìm kiếm Google có cấu trúc thông qua một API và có thể trực tiếp trả về "hộp tóm tắt câu trả lời" (answer summary box) hoặc thông tin knowledge graph chính xác.

Trước tiên, bạn cần cài đặt thư viện:

```bash
pip install google-search-results
```

Đồng thời, bạn cần truy cập [trang web chính thức của SerpApi](https://serpapi.com/) để đăng ký một tài khoản miễn phí, lấy API key của bạn, và thêm nó vào file `.env` trong thư mục gốc của dự án:

```bash
# .env file
# ... (Keep previous LLM configuration)
SERPAPI_API_KEY="YOUR_SERPAPI_API_KEY"
```

Tiếp theo, chúng ta sẽ định nghĩa và quản lý công cụ này thông qua mã nguồn. Chúng ta sẽ tiến hành từng bước: trước tiên triển khai chức năng cốt lõi của công cụ, sau đó xây dựng một trình quản lý công cụ tổng quát.

(1) Triển khai Logic Cốt lõi của Công cụ Tìm kiếm

Một công cụ được định nghĩa tốt cần chứa ba yếu tố cốt lõi sau:

1. **Tên (Name)**: Một định danh ngắn gọn, duy nhất để agent gọi trong `Action`, chẳng hạn như `Search`.
2. **Mô tả (Description)**: Một mô tả ngôn ngữ tự nhiên rõ ràng giải thích mục đích của công cụ này. **Đây là phần quan trọng nhất của toàn bộ cơ chế** bởi vì mô hình ngôn ngữ lớn sẽ dựa vào mô tả này để xác định khi nào sử dụng công cụ nào.
3. **Logic Thực thi (Execution Logic)**: Hàm hoặc phương thức thực sự thực hiện nhiệm vụ.

Công cụ đầu tiên của chúng ta là hàm `search`, nhận vào một chuỗi truy vấn và sau đó trả về kết quả tìm kiếm.

```python
from serpapi import SerpApiClient

def search(query: str) -> str:
    """
    A practical web search engine tool based on SerpApi.
    It intelligently parses search results, prioritizing direct answers or knowledge graph information.
    """
    print(f"🔍 Executing [SerpApi] web search: {query}")
    try:
        api_key = os.getenv("SERPAPI_API_KEY")
        if not api_key:
            return "Error: SERPAPI_API_KEY not configured in .env file."

        params = {
            "engine": "google",
            "q": query,
            "api_key": api_key,
            "gl": "cn",  # Country code
            "hl": "zh-cn", # Language code
        }
        
        client = SerpApiClient(params)
        results = client.get_dict()
        
        # Intelligent parsing: prioritize finding the most direct answer
        if "answer_box_list" in results:
            return "\n".join(results["answer_box_list"])
        if "answer_box" in results and "answer" in results["answer_box"]:
            return results["answer_box"]["answer"]
        if "knowledge_graph" in results and "description" in results["knowledge_graph"]:
            return results["knowledge_graph"]["description"]
        if "organic_results" in results and results["organic_results"]:
            # If no direct answer, return summaries of the first three organic results
            snippets = [
                f"[{i+1}] {res.get('title', '')}\n{res.get('snippet', '')}"
                for i, res in enumerate(results["organic_results"][:3])
            ]
            return "\n\n".join(snippets)
        
        return f"Sorry, no information found about '{query}'."

    except Exception as e:
        return f"Error occurred during search: {e}"
```

Trong đoạn mã trên, trước tiên nó kiểm tra xem có tồn tại thông tin `answer_box` (hộp tóm tắt câu trả lời của Google) hoặc `knowledge_graph` (knowledge graph) hay không. Nếu có, nó trực tiếp trả về những câu trả lời chính xác nhất này. Nếu không, nó quay lại trả về tóm tắt của ba kết quả tìm kiếm thông thường đầu tiên. Việc "phân tích thông minh" này có thể cung cấp đầu vào thông tin chất lượng cao hơn cho LLM.

(2) Xây dựng một Trình Thực thi Công cụ Tổng quát

Khi một agent cần sử dụng nhiều công cụ (ví dụ, ngoài tìm kiếm, nó có thể cũng cần tính toán, truy vấn cơ sở dữ liệu, v.v.), chúng ta cần một trình quản lý thống nhất để đăng ký và điều phối các công cụ này. Vì điều này, chúng ta tạo một lớp `ToolExecutor`.

```python
from typing import Dict, Any

class ToolExecutor:
    """
    A tool executor responsible for managing and executing tools.
    """
    def __init__(self):
        self.tools: Dict[str, Dict[str, Any]] = {}

    def registerTool(self, name: str, description: str, func: callable):
        """
        Register a new tool in the toolbox.
        """
        if name in self.tools:
            print(f"Warning: Tool '{name}' already exists and will be overwritten.")
        self.tools[name] = {"description": description, "func": func}
        print(f"Tool '{name}' registered.")

    def getTool(self, name: str) -> callable:
        """
        Get a tool's execution function by name.
        """
        return self.tools.get(name, {}).get("func")

    def getAvailableTools(self) -> str:
        """
        Get a formatted description string of all available tools.
        """
        return "\n".join([
            f"- {name}: {info['description']}" 
            for name, info in self.tools.items()
        ])

```

(3) Kiểm thử

Bây giờ, chúng ta sẽ đăng ký công cụ `search` trong `ToolExecutor` và mô phỏng một lệnh gọi để xác minh rằng toàn bộ quá trình hoạt động bình thường.

```python
# --- Tool Initialization and Usage Example ---
if __name__ == '__main__':
    # 1. Initialize tool executor
    toolExecutor = ToolExecutor()

    # 2. Register our practical search tool
    search_description = "A web search engine. Use this tool when you need to answer questions about current events, facts, and information not found in your knowledge base."
    toolExecutor.registerTool("Search", search_description, search)

    # 3. Print available tools
    print("\n--- Available Tools ---")
    print(toolExecutor.getAvailableTools())

    # 4. Agent's Action call, this time we ask a real-time question
    print("\n--- Execute Action: Search['What is NVIDIA's latest GPU model'] ---")
    tool_name = "Search"
    tool_input = "What is NVIDIA's latest GPU model"

    tool_function = toolExecutor.getTool(tool_name)
    if tool_function:
        observation = tool_function(tool_input)
        print("--- Observation ---")
        print(observation)
    else:
        print(f"Error: Tool named '{tool_name}' not found.")

>>>
Tool 'Search' registered.

--- Available Tools ---
- Search: A web search engine. Use this tool when you need to answer questions about current events, facts, and information not found in your knowledge base.

--- Execute Action: Search['What is NVIDIA's latest GPU model'] ---
🔍 Executing [SerpApi] web search: What is NVIDIA's latest GPU model
--- Observation ---
[1] GeForce RTX 50 Series Graphics Cards
GeForce RTX™ 50 Series GPUs are powered by NVIDIA Blackwell architecture, bringing new gameplay for gamers and creators. RTX 50 Series has powerful AI computing power, bringing upgraded experience and more realistic graphics.

[2] Compare GeForce Series Latest Generation and Previous Generation Graphics Cards
Compare the latest RTX 30 series graphics cards with previous RTX 20 series, GTX 10 and 900 series graphics cards. View specifications, features, technical support, etc.

[3] GeForce Graphics Cards | NVIDIA
DRIVE AGX. Powerful in-vehicle computing power for AI-driven intelligent vehicle systems · Clara AGX. AI computing for innovative medical devices and imaging. Gaming and Creation. GeForce. Explore graphics cards, gaming solutions, AI ...
```

Đến đây, chúng ta đã trang bị cho agent một công cụ `Search` kết nối với internet thế giới thực, cung cấp một nền tảng vững chắc cho vòng lặp ReAct tiếp theo.



### 4.2.3 Triển khai Mã nguồn của Agent ReAct

Bây giờ, chúng ta sẽ lắp ráp tất cả các thành phần độc lập—client LLM và trình thực thi công cụ—để xây dựng một agent ReAct hoàn chỉnh. Chúng ta sẽ đóng gói logic cốt lõi của nó thông qua một lớp `ReActAgent`. Để dễ hiểu, chúng ta sẽ chia nhỏ quá trình triển khai lớp này thành các phần chính sau để giải thích.

(1) Thiết kế System Prompt

Prompt là nền tảng của toàn bộ cơ chế ReAct, cung cấp các hướng dẫn vận hành cho mô hình ngôn ngữ lớn. Chúng ta cần thiết kế cẩn thận một template sẽ chèn động các công cụ có sẵn, câu hỏi của người dùng, và lịch sử tương tác của các bước trung gian.

```bash
# ReAct Prompt Template
REACT_PROMPT_TEMPLATE = """
Please note that you are an intelligent assistant capable of calling external tools.

Available tools are as follows:
{tools}

Please respond strictly in the following format:

Thought: Your thinking process, used to analyze problems, decompose tasks, and plan the next action.
Action: The action you decide to take, must be in one of the following formats:
- {{tool_name}}[{{tool_input}}]`: Call an available tool.
- `Finish[final answer]`: When you believe you have obtained the final answer.
- When you have collected enough information to answer the user's final question, you must use `Finish[final answer]` after the Action: field to output the final answer.

Now, please start solving the following problem:
Question: {question}
History: {history}
"""
```

Template này định nghĩa quy tắc tương tác giữa agent và LLM:

- **Định nghĩa Vai trò**: "You are an intelligent assistant capable of calling external tools" thiết lập vai trò của LLM.
- **Danh sách Công cụ (`{tools}`)**: Thông báo cho LLM biết nó có những "tay chân" nào để sử dụng.
- **Quy ước Định dạng (`Thought`/`Action`)**: Đây là phần quan trọng nhất, buộc đầu ra của LLM phải có cấu trúc để chúng ta có thể phân tích chính xác ý định của nó thông qua mã nguồn.
- **Ngữ cảnh Động (`{question}`/`{history}`)**: Chèn câu hỏi gốc của người dùng và lịch sử tương tác được tích lũy liên tục, cho phép LLM đưa ra quyết định dựa trên ngữ cảnh đầy đủ.

(2) Triển khai Vòng lặp Cốt lõi

Cốt lõi của `ReActAgent` là một vòng lặp liên tục "định dạng prompt -> gọi LLM -> thực thi hành động -> tích hợp kết quả" cho đến khi nhiệm vụ hoàn thành hoặc đạt đến giới hạn số bước tối đa.

```python
class ReActAgent:
    def __init__(self, llm_client: HelloAgentsLLM, tool_executor: ToolExecutor, max_steps: int = 5):
        self.llm_client = llm_client
        self.tool_executor = tool_executor
        self.max_steps = max_steps
        self.history = []

    def run(self, question: str):
        """
        Run the ReAct agent to answer a question.
        """
        self.history = [] # Reset history for each run
        current_step = 0

        while current_step < self.max_steps:
            current_step += 1
            print(f"--- Step {current_step} ---")

            # 1. Format prompt
            tools_desc = self.tool_executor.getAvailableTools()
            history_str = "\n".join(self.history)
            prompt = REACT_PROMPT_TEMPLATE.format(
                tools=tools_desc,
                question=question,
                history=history_str
            )

            # 2. Call LLM to think
            messages = [{"role": "user", "content": prompt}]
            response_text = self.llm_client.think(messages=messages)

            if not response_text:
                print("Error: LLM failed to return a valid response.")
                break

            # ... (Subsequent parsing, execution, integration steps)

```

Phương thức `run` là điểm khởi đầu của agent. Vòng lặp `while` của nó tạo thành phần chính của mô hình ReAct, và tham số `max_steps` là một van an toàn quan trọng để ngăn agent rơi vào vòng lặp vô hạn và cạn kiệt tài nguyên.

(3) Triển khai Trình phân tích Đầu ra

LLM trả về văn bản thuần túy, và chúng ta cần trích xuất chính xác `Thought` và `Action` từ đó. Điều này được thực hiện thông qua một số hàm phân tích phụ trợ, thường sử dụng biểu thức chính quy (regular expression).

```python
# (These methods are part of the ReActAgent class)
    def _parse_output(self, text: str):
        """Parse LLM output to extract Thought and Action.
        """
        # Thought: match until Action: or end of text
        thought_match = re.search(r"Thought:\s*(.*?)(?=\nAction:|$)", text, re.DOTALL)
        # Action: match until end of text
        action_match = re.search(r"Action:\s*(.*?)$", text, re.DOTALL)
        thought = thought_match.group(1).strip() if thought_match else None
        action = action_match.group(1).strip() if action_match else None
        return thought, action

    def _parse_action(self, action_text: str):
        """Parse Action string to extract tool name and input.
        """
        match = re.match(r"(\w+)\[(.*)\]", action_text, re.DOTALL)
        if match:
            return match.group(1), match.group(2)
        return None, None
```

- `_parse_output`: Chịu trách nhiệm tách hai phần chính `Thought` và `Action` từ phản hồi hoàn chỉnh của LLM.
- `_parse_action`: Chịu trách nhiệm phân tích thêm chuỗi `Action`, ví dụ, trích xuất tên công cụ `Search` và đầu vào công cụ `Điện thoại mới nhất của Huawei` từ `Search[Điện thoại mới nhất của Huawei]`.

(4) Gọi và Thực thi Công cụ

```python
# (This logic is inside the while loop of the run method)
            # 3. Parse LLM output
            thought, action = self._parse_output(response_text)

            if thought:
                print(f"Thought: {thought}")

            if not action:
                print("Warning: Failed to parse valid Action, process terminated.")
                break

            # 4. Execute Action
            if action.startswith("Finish"):
                # If it's a Finish instruction, extract the final answer and end
                final_answer = re.match(r"Finish\[(.*)\]", action).group(1)
                print(f"🎉 Final Answer: {final_answer}")
                return final_answer

            tool_name, tool_input = self._parse_action(action)
            if not tool_name or not tool_input:
                # ... Handle invalid Action format ...
                continue

            print(f"🎬 Action: {tool_name}[{tool_input}]")

            tool_function = self.tool_executor.getTool(tool_name)
            if not tool_function:
                observation = f"Error: Tool named '{tool_name}' not found."
            else:
                observation = tool_function(tool_input) # Call real tool

```

Đoạn mã này là trung tâm thực thi của `Action`. Trước tiên nó kiểm tra xem đó có phải là lệnh `Finish` hay không; nếu vậy, quá trình kết thúc. Nếu không, nó lấy hàm công cụ tương ứng thông qua `tool_executor` và thực thi nó để lấy `observation`.

(5) Tích hợp Kết quả Quan sát

Bước cuối cùng, và là chìa khóa để tạo thành một vòng lặp khép kín, là thêm chính `Action` và `Observation` sau khi thực thi công cụ trở lại lịch sử, cung cấp ngữ cảnh mới cho vòng lặp tiếp theo.

```python
# (This logic follows tool invocation, at the end of the while loop)
            print(f"👀 Observation: {observation}")

            # Add this round's Action and Observation to history
            self.history.append(f"Action: {action}")
            self.history.append(f"Observation: {observation}")

        # Loop ends
        print("Maximum steps reached, process terminated.")
        return None
```

Bằng cách nối `Observation` vào `self.history`, agent có thể "nhìn thấy" kết quả của hành động trước đó khi tạo prompt trong vòng lặp tiếp theo, và tiến hành tư duy và lập kế hoạch mới tương ứng.

(6) Ví dụ Chạy và Phân tích

Kết hợp tất cả các phần trên, chúng ta có được lớp `ReActAgent` hoàn chỉnh. Ví dụ chạy mã hoàn chỉnh có thể được tìm thấy trong thư mục `code` của kho mã đi kèm cuốn sách này.

Dưới đây là một bản ghi chạy thực tế:

```
Tool 'Search' registered.

--- Step 1 ---
🧠 Calling xxxxxx model...
✅ Large language model response successful:
Thought: To answer this question, I need to search for Huawei's latest released phone model and its main features. This information may be outside my existing knowledge base, so I need to use a search engine to obtain the latest data.
Action: Search[Huawei latest phone model and main selling points]
🤔 Thought: To answer this question, I need to search for Huawei's latest released phone model and its main features. This information may be outside my existing knowledge base, so I need to use a search engine to obtain the latest data.
🎬 Action: Search[Huawei latest phone model and main selling points]
🔍 Executing [SerpApi] web search: Huawei latest phone model and main selling points
👀 Observation: [1] Huawei Phones - Huawei Official Website
Smartphones ; Mate Series. Extraordinary Flagship · HUAWEI Mate XTs. Extraordinary Master ; Pura Series. Pioneer Imaging · HUAWEI Pura 80 Pro+ ; Pocket Series. Aesthetic New Chapter. HUAWEI Pocket 2 ; nova Series. Professional Portrait.

[2] Which Huawei phone has the best value in 2025? Huawei phone recommendations and market analysis ...
Now the biggest selling point of Huawei phones is only the HarmonyOS system, and the controversial brand faith. Here are several Huawei series phones worth buying, choose according to different budgets:. Huawei's currently most popular, also equipped with ...

[3] Which new Huawei phone has the best value in 2025? 10 new Huawei phone recommendations
Choosing Huawei mainly recommends high-end phones, Mate 70 and Pura 70 series are the latest flagship models. HUAWEI Mate 70. Advantages are, photography configuration is still top-level, full focal length coverage, suitable for professional photography, excellent workmanship, outdoor drop resistance ...

--- Step 2 ---
🧠 Calling xxxxxx model...
✅ Large language model response successful:
Thought: Based on the search results, Huawei's latest flagship models include Mate 70 and Pura 80 Pro+. To determine the latest model and its main selling points, I will focus on this information. From the provided links, both Mate 70 series and Pura 80 Pro+ are recently released products, but which one is "latest" needs further confirmation. Meanwhile, I can extract their main selling points from this information.
Action: Finish[According to the latest information, Huawei's latest phones may be HUAWEI Pura 80 Pro+ or HUAWEI Mate 70. Among them, HUAWEI Mate 70's main selling points include top-level photography configuration, full focal length coverage, suitable for professional photography, excellent workmanship, and good outdoor drop resistance. While HUAWEI Pura 80 Pro+ emphasizes pioneer imaging technology.]
🤔 Thought: Based on the search results, Huawei's latest flagship models include Mate 70 and Pura 80 Pro+. To determine the latest model and its main selling points, I will focus on this information. From the provided links, both Mate 70 series and Pura 80 Pro+ are recently released products, but which one is "latest" needs further confirmation. Meanwhile, I can extract their main selling points from this information.
🎉 Final Answer: According to the latest information, Huawei's latest phones may be HUAWEI Pura 80 Pro+ or HUAWEI Mate 70. Among them, HUAWEI Mate 70's main selling points include top-level photography configuration, full focal length coverage, suitable for professional photography, excellent workmanship, and good outdoor drop resistance. While HUAWEI Pura 80 Pro+ emphasizes pioneer imaging technology.
```

Từ đầu ra trên, chúng ta có thể thấy agent thể hiện rõ ràng chuỗi tư duy của nó: trước tiên nó nhận ra kiến thức của mình không đủ và cần sử dụng công cụ tìm kiếm; sau đó, nó suy luận và tóm tắt dựa trên kết quả tìm kiếm, đưa ra câu trả lời cuối cùng trong vòng hai bước.

Cần lưu ý rằng vì kiến thức của mô hình và thông tin trên internet liên tục được cập nhật, kết quả chạy của bạn có thể không hoàn toàn giống với kết quả này. Tính đến ngày 8 tháng 9 năm 2025, khi phần này được viết, HUAWEI Mate 70 và HUAWEI Pura 80 Pro+ được đề cập trong kết quả tìm kiếm thực sự là các dòng điện thoại flagship mới nhất của Huawei tại thời điểm đó. Điều này chứng minh đầy đủ năng lực mạnh mẽ của mô hình ReAct trong việc xử lý các vấn đề nhạy cảm về thời gian.

### 4.2.4 Đặc điểm, Hạn chế và Kỹ thuật Gỡ lỗi của ReAct

Bằng cách tự tay triển khai một agent ReAct, chúng ta không chỉ nắm vững quy trình hoạt động của nó mà còn nên có sự hiểu biết sâu sắc hơn về các cơ chế bên trong của nó. Bất kỳ mô hình kỹ thuật nào cũng có những điểm nổi bật và những lĩnh vực cần cải thiện; phần này sẽ tổng kết về ReAct.

(1) Các Đặc điểm Chính của ReAct

1. **Khả năng diễn giải cao**: Một trong những ưu điểm lớn nhất của ReAct là tính minh bạch. Thông qua chuỗi `Thought`, chúng ta có thể thấy rõ "hành trình tư duy" của agent tại mỗi bước—tại sao nó chọn công cụ này và nó dự định làm gì tiếp theo. Điều này rất quan trọng để hiểu, tin tưởng và gỡ lỗi hành vi của agent.
2. **Khả năng Lập kế hoạch Động và Sửa lỗi**: Không giống như các mô hình tạo ra kế hoạch hoàn chỉnh một lần, ReAct là "đi một bước, nhìn một bước". Nó điều chỉnh động các `Thought` và `Action` tiếp theo dựa trên `Observation` thu được từ thế giới bên ngoài tại mỗi bước. Nếu kết quả tìm kiếm trước đó không thỏa đáng, nó có thể chỉnh sửa các từ khóa tìm kiếm trong bước tiếp theo và thử lại.
3. **Khả năng Hiệp lực Công cụ**: Mô hình ReAct kết hợp một cách tự nhiên năng lực suy luận của các mô hình ngôn ngữ lớn với năng lực thực thi của các công cụ bên ngoài. LLM chịu trách nhiệm hoạch định chiến lược (lập kế hoạch và suy luận), công cụ chịu trách nhiệm giải quyết các vấn đề cụ thể (tìm kiếm, tính toán), và cả hai làm việc hiệp lực, phá vỡ các giới hạn vốn có của các LLM đơn lẻ về tính kịp thời của kiến thức, độ chính xác tính toán, v.v.

(2) Các Hạn chế Vốn có của ReAct

1. **Phụ thuộc Mạnh vào Năng lực của chính LLM**: Sự thành công của quá trình ReAct phụ thuộc rất nhiều vào các năng lực tổng thể của LLM cơ sở. Nếu khả năng suy luận logic, khả năng tuân theo hướng dẫn, hoặc khả năng xuất đầu ra có định dạng của LLM không đủ, thì dễ tạo ra lập kế hoạch sai trong giai đoạn `Thought` hoặc tạo ra các lệnh không tuân theo định dạng trong giai đoạn `Action`, khiến toàn bộ quá trình bị gián đoạn.
2. **Vấn đề Hiệu suất Thực thi**: Do tính chất từng bước của nó, việc hoàn thành một nhiệm vụ thường yêu cầu nhiều lệnh gọi LLM. Mỗi lệnh gọi đi kèm với độ trễ mạng và chi phí tính toán. Đối với các nhiệm vụ phức tạp yêu cầu nhiều bước, vòng lặp "tư duy-hành động" tuần tự này có thể dẫn đến tổng thời gian và chi phí cao.
3. **Tính dễ vỡ của Prompt**: Hoạt động ổn định của toàn bộ cơ chế được xây dựng trên một template prompt được thiết kế cẩn thận. Bất kỳ thay đổi nhỏ nào trong template, thậm chí là sự khác biệt trong cách diễn đạt, đều có thể ảnh hưởng đến hành vi của LLM. Ngoài ra, không phải tất cả các mô hình đều có thể luôn tuân theo các định dạng đã đặt trước, làm tăng tính không chắc chắn trong các ứng dụng thực tế.
4. **Có thể Rơi vào Tối ưu Cục bộ**: Chế độ ra quyết định từng bước có nghĩa là agent thiếu một kế hoạch toàn cục, dài hạn. Nó có thể chọn một con đường có vẻ đúng trong ngắn hạn nhưng không tối ưu về lâu dài do `Observation` tức thời, hoặc thậm chí rơi vào một vòng lặp "xoay tại chỗ" trong một số trường hợp.

(3) Kỹ thuật Gỡ lỗi

Khi agent ReAct bạn xây dựng hoạt động không như mong đợi, bạn có thể gỡ lỗi từ các khía cạnh sau:

- **Kiểm tra Prompt Hoàn chỉnh**: Trước mỗi lệnh gọi LLM, in ra prompt hoàn chỉnh cuối cùng đã được định dạng chứa toàn bộ lịch sử. Đây là cách trực tiếp nhất để truy vết nguồn gốc các quyết định của LLM.
- **Phân tích Đầu ra Thô**: Khi việc phân tích đầu ra thất bại (ví dụ, biểu thức chính quy không khớp với `Action`), hãy chắc chắn in ra văn bản thô, chưa được xử lý mà LLM trả về. Điều này có thể giúp bạn xác định xem LLM không tuân theo định dạng hay logic phân tích của bạn bị sai.
- **Xác minh Đầu vào và Đầu ra của Công cụ**: Kiểm tra xem `tool_input` do agent tạo ra có đúng định dạng mà hàm công cụ mong đợi hay không, và cũng đảm bảo `observation` được công cụ trả về ở định dạng mà agent có thể hiểu và xử lý.
- **Điều chỉnh Ví dụ trong Prompt (Few-shot Prompting)**: Nếu mô hình thường xuyên mắc lỗi, bạn có thể thêm một hoặc hai trường hợp "Thought-Action-Observation" thành công hoàn chỉnh vào prompt để hướng dẫn mô hình tuân theo hướng dẫn của bạn tốt hơn thông qua các ví dụ.
- **Thử các Mô hình hoặc Tham số Khác nhau**: Chuyển sang một mô hình mạnh hơn hoặc điều chỉnh tham số `temperature` (thường được đặt thành 0 để đảm bảo tính xác định của đầu ra) đôi khi có thể trực tiếp giải quyết vấn đề.

## 4.3 Plan-and-Solve

Sau khi nắm vững ReAct, mô hình agent ra quyết định phản ứng, từng bước này, tiếp theo chúng ta sẽ khám phá một phương pháp có phong cách rất khác biệt nhưng cũng mạnh mẽ không kém: **Plan-and-Solve**. Đúng như tên gọi, mô hình này chia rõ ràng việc xử lý nhiệm vụ thành hai giai đoạn: **Lập kế hoạch trước, sau đó Giải quyết**.

Nếu ReAct giống như một thám tử giàu kinh nghiệm suy luận từng bước dựa trên các manh mối tại hiện trường (Observation) và điều chỉnh hướng điều tra bất cứ lúc nào; thì Plan-and-Solve giống như một kiến trúc sư trước tiên phải vẽ một bản thiết kế hoàn chỉnh (Plan) trước khi bắt đầu xây dựng, sau đó xây dựng nghiêm ngặt theo bản thiết kế (Solve). Thực tế, nhiều chế độ Agent của các công cụ mô hình lớn mà chúng ta sử dụng hiện nay đều kết hợp mô hình thiết kế này.

### 4.3.1 Nguyên lý Hoạt động của Plan-and-Solve

Plan-and-Solve Prompting được đề xuất bởi Lei Wang vào năm 2023<sup>[2]</sup>. Động lực cốt lõi của nó là giải quyết vấn đề chain-of-thought dễ bị "chệch hướng" khi xử lý các vấn đề phức tạp, nhiều bước.

Không giống như ReAct, tích hợp tư duy và hành động tại mỗi bước, Plan-and-Solve tách toàn bộ quá trình thành hai giai đoạn cốt lõi, như minh họa trong Hình 4.2:

1. **Giai đoạn Lập kế hoạch (Planning Phase)**: Trước tiên, agent nhận được câu hỏi hoàn chỉnh của người dùng. Nhiệm vụ đầu tiên của nó không phải là trực tiếp giải quyết vấn đề hoặc gọi công cụ, mà là **phân rã vấn đề và xây dựng một kế hoạch hành động rõ ràng, từng bước**. Bản thân kế hoạch này là sản phẩm của một lệnh gọi mô hình ngôn ngữ lớn.
2. **Giai đoạn Giải quyết (Solving Phase)**: Sau khi có được kế hoạch hoàn chỉnh, agent bước vào giai đoạn thực thi. Nó sẽ **thực thi nghiêm ngặt theo các bước trong kế hoạch, từng bước một**. Việc thực thi mỗi bước có thể là một lệnh gọi LLM độc lập hoặc xử lý kết quả của bước trước, cho đến khi tất cả các bước trong kế hoạch được hoàn thành và có được câu trả lời cuối cùng.

Chiến lược "lập kế hoạch trước khi hành động" này cho phép agent duy trì tính nhất quán về mục tiêu cao hơn khi xử lý các nhiệm vụ phức tạp yêu cầu lập kế hoạch dài hạn, tránh bị lạc trong các bước trung gian.

Chúng ta có thể biểu diễn quá trình hai giai đoạn này một cách hình thức. Trước tiên, mô hình lập kế hoạch $\pi_{\text{plan}}$ tạo ra một kế hoạch $P = (p_1, p_2, \dots, p_n)$ chứa $n$ bước dựa trên câu hỏi gốc $q$:

$$
P = \pi_{\text{plan}}(q)
$$

Sau đó, trong giai đoạn thực thi, mô hình thực thi $\pi_{\text{solve}}$ sẽ hoàn thành các bước trong kế hoạch từng bước một. Đối với bước thứ $i$, việc tạo ra lời giải $s_i$ của nó sẽ phụ thuộc vào câu hỏi gốc $q$, kế hoạch hoàn chỉnh $P$, và kết quả thực thi của tất cả các bước trước đó $(s_1, \dots, s_{i-1})$:

$$
s_i = \pi_{\text{solve}}(q, P, (s_1, \dots, s_{i-1}))
$$

Câu trả lời cuối cùng là kết quả thực thi của bước cuối cùng $s_n$.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-2.png" alt="Two-stage workflow of Plan-and-Solve paradigm" width="90%"/>
  <p>Hình 4.2 Quy trình Hai Giai đoạn của Mô hình Plan-and-Solve</p>
</div>

Plan-and-Solve đặc biệt phù hợp với các nhiệm vụ phức tạp có tính cấu trúc mạnh, có thể được phân rã rõ ràng, chẳng hạn như:

- **Các bài toán đố toán nhiều bước**: Cần liệt kê các bước tính toán trước, sau đó giải quyết từng bước một.
- **Viết báo cáo tích hợp nhiều nguồn thông tin**: Cần lập kế hoạch cấu trúc báo cáo trước (giới thiệu, nguồn dữ liệu A, nguồn dữ liệu B, tóm tắt), sau đó điền nội dung từng phần một.
- **Các nhiệm vụ tạo mã**: Cần hình dung cấu trúc của các hàm, lớp, và module trước, sau đó triển khai từng phần một.

### 4.3.2 Giai đoạn Lập kế hoạch

Để làm nổi bật các ưu điểm của mô hình Plan-and-Solve trong các nhiệm vụ suy luận có cấu trúc, chúng ta sẽ không sử dụng công cụ mà hoàn thành một nhiệm vụ suy luận thông qua thiết kế prompt.

Đặc điểm của loại nhiệm vụ này là câu trả lời không thể có được thông qua một truy vấn hoặc tính toán duy nhất; vấn đề trước tiên phải được phân rã thành một loạt các bước con mạch lạc về mặt logic, sau đó giải quyết theo thứ tự. Điều này chính xác tận dụng năng lực cốt lõi "lập kế hoạch trước, thực thi sau" của Plan-and-Solve.

**Vấn đề mục tiêu của chúng ta là:** "Một cửa hàng trái cây bán được 15 quả táo vào thứ Hai. Số táo bán được vào thứ Ba gấp đôi thứ Hai. Số bán được vào thứ Tư ít hơn thứ Ba 5 quả. Tổng cộng có bao nhiêu quả táo được bán trong ba ngày này?"

Vấn đề này không đặc biệt khó đối với các mô hình ngôn ngữ lớn, nhưng nó chứa một chuỗi logic rõ ràng để tham khảo. Đối với một số câu đố logic thực tế, nếu mô hình lớn không thể suy luận ra câu trả lời chính xác với chất lượng cao, bạn có thể tham khảo mô hình thiết kế này để thiết kế Agent của riêng mình hoàn thành nhiệm vụ. Agent cần:

1. **Giai đoạn Lập kế hoạch**: Trước tiên, phân rã vấn đề thành ba bước tính toán độc lập (tính doanh số thứ Ba, tính doanh số thứ Tư, tính tổng doanh số).
2. **Giai đoạn Thực thi**: Sau đó, tuân theo nghiêm ngặt kế hoạch, thực thi tính toán từng bước, và sử dụng kết quả của mỗi bước làm đầu vào cho bước tiếp theo, cuối cùng có được tổng số.

Mục tiêu của giai đoạn lập kế hoạch là để mô hình ngôn ngữ lớn nhận được vấn đề gốc và xuất ra một kế hoạch hành động rõ ràng, từng bước. Kế hoạch này phải có cấu trúc để mã nguồn của chúng ta có thể dễ dàng phân tích và thực thi từng bước một. Do đó, prompt mà chúng ta thiết kế cần nói rõ với mô hình về vai trò và nhiệm vụ của nó và cung cấp một ví dụ về định dạng đầu ra.

````python
PLANNER_PROMPT_TEMPLATE = """
You are a top AI planning expert. Your task is to decompose complex problems posed by users into an action plan consisting of multiple simple steps.
Please ensure that each step in the plan is an independent, executable subtask and is strictly arranged in logical order.
Your output must be a Python list, where each element is a string describing a subtask.

Question: {question}

Please strictly output your plan in the following format, with ```python and ``` as prefix and suffix being necessary:
```python
["Step 1", "Step 2", "Step 3", ...]
```
"""
````

Prompt này đảm bảo chất lượng và tính ổn định của đầu ra thông qua các điểm sau:
- **Thiết lập Vai trò**: "Top AI planning expert" kích hoạt các năng lực chuyên môn của mô hình.
- **Mô tả Nhiệm vụ**: Định nghĩa rõ ràng mục tiêu "phân rã vấn đề".
- **Ràng buộc Định dạng**: Buộc đầu ra phải là một chuỗi ở định dạng Python list, điều này đơn giản hóa rất nhiều công việc phân tích mã nguồn tiếp theo, làm cho nó ổn định và đáng tin cậy hơn so với việc phân tích ngôn ngữ tự nhiên.

Tiếp theo, chúng ta đóng gói logic prompt này vào một lớp `Planner`, đây cũng chính là bộ lập kế hoạch của chúng ta.

```python
# Assume the HelloAgentsLLM class in llm_client.py is already defined
# from llm_client import HelloAgentsLLM

class Planner:
    def __init__(self, llm_client):
        self.llm_client = llm_client

    def plan(self, question: str) -> list[str]:
        """
        Generate an action plan based on user question.
        """
        prompt = PLANNER_PROMPT_TEMPLATE.format(question=question)

        # To generate a plan, we build a simple message list
        messages = [{"role": "user", "content": prompt}]

        print("--- Generating Plan ---")
        # Use streaming output to get the complete plan
        response_text = self.llm_client.think(messages=messages) or ""

        print(f"✅ Plan Generated:\n{response_text}")

        # Parse the list string output by LLM
        try:
            # Find content between ```python and ```
            plan_str = response_text.split("```python")[1].split("```")[0].strip()
            # Use ast.literal_eval to safely execute the string and convert it to a Python list
            plan = ast.literal_eval(plan_str)
            return plan if isinstance(plan, list) else []
        except (ValueError, SyntaxError, IndexError) as e:
            print(f"❌ Error parsing plan: {e}")
            print(f"Raw response: {response_text}")
            return []
        except Exception as e:
            print(f"❌ Unknown error occurred while parsing plan: {e}")
            return []
```

### 4.3.3 Trình Thực thi và Quản lý Trạng thái

Sau khi bộ lập kế hoạch (`Planner`) tạo ra một bản thiết kế hành động rõ ràng, chúng ta cần một trình thực thi (`Executor`) để hoàn thành các nhiệm vụ trong kế hoạch từng bước một. Trình thực thi không chỉ chịu trách nhiệm gọi mô hình ngôn ngữ lớn để giải quyết từng vấn đề con mà còn đóng một vai trò quan trọng: **quản lý trạng thái (state management)**. Nó phải ghi lại kết quả thực thi của mỗi bước và cung cấp chúng làm ngữ cảnh cho các bước tiếp theo, đảm bảo thông tin lưu thông trơn tru trong suốt chuỗi nhiệm vụ.

Prompt của trình thực thi khác với prompt của bộ lập kế hoạch. Mục tiêu của nó không phải là phân rã vấn đề mà là **tập trung vào giải quyết bước hiện tại dựa trên ngữ cảnh hiện có**. Do đó, prompt cần bao gồm các thông tin chính sau:

- **Câu hỏi Gốc**: Đảm bảo mô hình luôn hiểu được mục tiêu cuối cùng.
- **Kế hoạch Hoàn chỉnh**: Cho mô hình hiểu vị trí của bước hiện tại trong toàn bộ nhiệm vụ.
- **Các Bước và Kết quả Lịch sử**: Cung cấp công việc đã hoàn thành cho đến nay làm đầu vào trực tiếp cho bước hiện tại.
- **Bước Hiện tại**: Hướng dẫn rõ ràng cho mô hình biết nhiệm vụ cụ thể nào nó cần giải quyết ngay bây giờ.

```python
EXECUTOR_PROMPT_TEMPLATE = """
You are a top AI execution expert. Your task is to strictly follow the given plan and solve the problem step by step.
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
```

Chúng ta đóng gói logic thực thi vào lớp `Executor`. Lớp này sẽ lặp qua kế hoạch, gọi LLM, và duy trì một lịch sử (trạng thái).

```python
class Executor:
    def __init__(self, llm_client):
        self.llm_client = llm_client

    def execute(self, question: str, plan: list[str]) -> str:
        """
        Execute step by step according to the plan and solve the problem.
        """
        history = "" # String to store historical steps and results

        print("\n--- Executing Plan ---")

        for i, step in enumerate(plan):
            print(f"\n-> Executing step {i+1}/{len(plan)}: {step}")

            prompt = EXECUTOR_PROMPT_TEMPLATE.format(
                question=question,
                plan=plan,
                history=history if history else "None", # If it's the first step, history is empty
                current_step=step
            )

            messages = [{"role": "user", "content": prompt}]

            response_text = self.llm_client.think(messages=messages) or ""

            # Update history for the next step
            history += f"Step {i+1}: {step}\nResult: {response_text}\n\n"

            print(f"✅ Step {i+1} completed, result: {response_text}")

        # After the loop ends, the last step's response is the final answer
        final_answer = response_text
        return final_answer
```

Bây giờ chúng ta đã xây dựng riêng biệt `Planner` chịu trách nhiệm "lập kế hoạch" và `Executor` chịu trách nhiệm "thực thi". Bước cuối cùng là tích hợp hai thành phần này vào một agent thống nhất `PlanAndSolveAgent` và trao cho nó năng lực giải quyết vấn đề hoàn chỉnh. Chúng ta sẽ tạo một lớp chính `PlanAndSolveAgent` có trách nhiệm rất rõ ràng: nhận một client LLM, khởi tạo bộ lập kế hoạch và trình thực thi bên trong, và cung cấp một phương thức `run` đơn giản để khởi động toàn bộ quá trình.

```python
class PlanAndSolveAgent:
    def __init__(self, llm_client):
        """
        Initialize the agent and create planner and executor instances.
        """
        self.llm_client = llm_client
        self.planner = Planner(self.llm_client)
        self.executor = Executor(self.llm_client)

    def run(self, question: str):
        """
        Run the agent's complete process: plan first, then execute.
        """
        print(f"\n--- Starting to Process Question ---\nQuestion: {question}")

        # 1. Call planner to generate plan
        plan = self.planner.plan(question)

        # Check if plan was successfully generated
        if not plan:
            print("\n--- Task Terminated --- \nUnable to generate valid action plan.")
            return

        # 2. Call executor to execute plan
        final_answer = self.executor.execute(question, plan)

        print(f"\n--- Task Completed ---\nFinal Answer: {final_answer}")
```

Thiết kế của lớp `PlanAndSolveAgent` này thể hiện nguyên tắc "kết hợp thay vì kế thừa" (composition over inheritance). Bản thân nó không chứa logic phức tạp mà đóng vai trò như một bộ điều phối (orchestrator), gọi rõ ràng các thành phần bên trong của nó để hoàn thành nhiệm vụ.

### 4.3.4 Ví dụ Chạy và Phân tích

Mã hoàn chỉnh cũng có thể được tìm thấy trong thư mục `code` của kho mã đi kèm cuốn sách này; ở đây chúng ta chỉ minh họa kết quả cuối cùng.

````bash
--- Starting to Process Question ---
Question: A fruit store sold 15 apples on Monday. The number of apples sold on Tuesday was twice that of Monday. The number sold on Wednesday was 5 fewer than Tuesday. How many apples were sold in total over these three days?
--- Generating Plan ---
🧠 Calling xxxx model...
✅ Large language model response successful:
```python
["Calculate Monday's apple sales: 15", "Calculate Tuesday's apple sales: Monday's quantity × 2 = 15 × 2 = 30", "Calculate Wednesday's apple sales: Tuesday's quantity - 5 = 30 - 5 = 25", "Calculate total sales for three days: Monday + Tuesday + Wednesday = 15 + 30 + 25 = 70"]
```
✅ Plan Generated:
```python
["Calculate Monday's apple sales: 15", "Calculate Tuesday's apple sales: Monday's quantity × 2 = 15 × 2 = 30", "Calculate Wednesday's apple sales: Tuesday's quantity - 5 = 30 - 5 = 25", "Calculate total sales for three days: Monday + Tuesday + Wednesday = 15 + 30 + 25 = 70"]
```

--- Executing Plan ---

-> Executing step 1/4: Calculate Monday's apple sales: 15
🧠 Calling xxxx model...
✅ Large language model response successful:
15
✅ Step 1 completed, result: 15

-> Executing step 2/4: Calculate Tuesday's apple sales: Monday's quantity × 2 = 15 × 2 = 30
🧠 Calling xxxx model...
✅ Large language model response successful:
30
✅ Step 2 completed, result: 30

-> Executing step 3/4: Calculate Wednesday's apple sales: Tuesday's quantity - 5 = 30 - 5 = 25
🧠 Calling xxxx model...
✅ Large language model response successful:
25
✅ Step 3 completed, result: 25

-> Executing step 4/4: Calculate total sales for three days: Monday + Tuesday + Wednesday = 15 + 30 + 25 = 70
🧠 Calling xxxx model...
✅ Large language model response successful:
70
✅ Step 4 completed, result: 70

--- Task Completed ---
Final Answer: 70
````

Từ nhật ký đầu ra trên, chúng ta có thể thấy rõ quy trình hoạt động của mô hình Plan-and-Solve:

1. **Giai đoạn Lập kế hoạch**: Agent trước tiên gọi `Planner` và phân rã thành công bài toán đố phức tạp thành một Python list chứa bốn bước logic. Kế hoạch có cấu trúc này đặt nền tảng cho việc thực thi tiếp theo.
2. **Giai đoạn Thực thi**: `Executor` thực thi nghiêm ngặt từng bước một theo kế hoạch được tạo ra. Trong mỗi bước, nó sử dụng các kết quả lịch sử làm ngữ cảnh, đảm bảo việc truyền tải thông tin chính xác (ví dụ, bước 2 sử dụng đúng kết quả "15" của bước 1, và bước 3 cũng sử dụng đúng kết quả "30" của bước 2).
3. **Kết quả**: Toàn bộ quá trình rõ ràng về mặt logic với các bước rõ ràng, và agent đưa ra chính xác câu trả lời đúng "70".

## 4.4 Reflection

Trong các mô hình ReAct và Plan-and-Solve mà chúng ta đã triển khai, một khi agent hoàn thành một nhiệm vụ, quy trình hoạt động của nó kết thúc. Tuy nhiên, các câu trả lời ban đầu mà chúng tạo ra, dù là quỹ đạo hành động hay kết quả cuối cùng, có thể chứa lỗi hoặc có chỗ để cải thiện. Ý tưởng cốt lõi của cơ chế Reflection là giới thiệu một **vòng lặp tự sửa lỗi hậu kỳ** cho agent, cho phép nó xem xét lại công việc của mình, phát hiện các thiếu sót, và tối ưu hóa lặp đi lặp lại, giống như con người vẫn làm.

### 4.4.1 Ý tưởng Cốt lõi của Cơ chế Reflection

Nguồn cảm hứng cho cơ chế Reflection đến từ quá trình học tập của con người: chúng ta đọc dò sau khi hoàn thành bản nháp đầu tiên và kiểm tra lại sau khi giải một bài toán. Ý tưởng này được thể hiện trong nhiều nghiên cứu, chẳng hạn như framework Reflexion được đề xuất bởi Shinn, Noah vào năm 2023<sup>[3]</sup>. Quy trình hoạt động cốt lõi của nó có thể được tóm tắt thành một vòng lặp ba bước ngắn gọn: **Thực thi -> Phản tư -> Tinh chỉnh (Execute -> Reflect -> Refine)**.

1. **Thực thi (Execution)**: Trước tiên, agent cố gắng hoàn thành nhiệm vụ bằng các phương pháp quen thuộc (như ReAct hoặc Plan-and-Solve), tạo ra một lời giải sơ bộ hoặc quỹ đạo hành động. Điều này có thể được xem như một "bản nháp đầu tiên".
2. **Phản tư (Reflection)**: Tiếp theo, agent bước vào giai đoạn phản tư. Nó gọi một instance mô hình ngôn ngữ lớn độc lập, hoặc một instance với các prompt đặc biệt, để đóng vai trò một "người đánh giá" (reviewer). "Người đánh giá" này xem xét "bản nháp đầu tiên" được tạo ra ở bước một và đánh giá nó từ nhiều khía cạnh, chẳng hạn như:
   - **Lỗi Thực tế**: Có nội dung nào mâu thuẫn với lẽ thường hoặc các sự thật đã biết không?
   - **Sai sót Logic**: Có sự không nhất quán hoặc mâu thuẫn nào trong quá trình suy luận không?
   - **Vấn đề Hiệu suất**: Có một con đường trực tiếp hơn, ngắn gọn hơn để hoàn thành nhiệm vụ không?
   - **Thiếu Thông tin**: Có một số ràng buộc hoặc khía cạnh quan trọng của vấn đề bị bỏ sót không? Dựa trên đánh giá, nó tạo ra **Phản hồi (Feedback)** có cấu trúc, chỉ ra các vấn đề cụ thể và đề xuất cải thiện.
3. **Tinh chỉnh (Refinement)**: Cuối cùng, agent sử dụng "bản nháp đầu tiên" và "phản hồi" làm ngữ cảnh mới, gọi lại mô hình ngôn ngữ lớn, và yêu cầu nó chỉnh sửa bản nháp đầu tiên dựa trên nội dung phản hồi, tạo ra một "bản chỉnh sửa" hoàn thiện hơn.

Như minh họa trong Hình 4.3, vòng lặp này có thể được lặp lại nhiều lần cho đến khi giai đoạn phản tư không còn tìm thấy vấn đề mới hoặc đạt đến giới hạn số lần lặp đã đặt trước. Chúng ta có thể biểu diễn quá trình tối ưu hóa lặp này một cách hình thức. Giả sử $O_i$ là đầu ra được tạo ra bởi lần lặp thứ $i$ ($O_0$ là đầu ra ban đầu), mô hình phản tư $\pi_{\text{reflect}}$ tạo ra phản hồi $F_i$ cho $O_i$:
$$
F_i = \pi_{\text{reflect}}(\text{Task}, O_i)
$$
Sau đó, mô hình tinh chỉnh $\pi_{\text{refine}}$ kết hợp nhiệm vụ gốc, đầu ra của phiên bản trước, và phản hồi để tạo ra đầu ra của phiên bản mới $O_{i+1}$:
$$
O_{i+1} = \pi_{\text{refine}}(\text{Task}, O_i, F_i)
$$



<div align="center">
<img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-3.png" alt="Execute-Reflect-Refine iterative loop in Reflection mechanism" width="70%"/>
<p>Hình 4.3 Vòng lặp Lặp đi lặp lại Thực thi-Phản tư-Tinh chỉnh trong Cơ chế Reflection</p>
</div>



So với hai mô hình trước, giá trị của Reflection nằm ở:

- Nó cung cấp cho agent một vòng lặp sửa lỗi nội bộ, làm cho nó không còn phụ thuộc hoàn toàn vào phản hồi từ công cụ bên ngoài (Observation của ReAct), do đó có thể sửa các lỗi logic và chiến lược ở cấp độ cao hơn.
- Nó biến việc thực thi nhiệm vụ một lần thành một quá trình tối ưu hóa liên tục, cải thiện đáng kể tỷ lệ thành công cuối cùng và chất lượng câu trả lời cho các nhiệm vụ phức tạp.
- Nó xây dựng một **"bộ nhớ ngắn hạn" (short-term memory)** tạm thời cho agent. Toàn bộ quỹ đạo "thực thi-phản tư-tinh chỉnh" tạo thành một bản ghi kinh nghiệm quý giá; agent không chỉ biết câu trả lời cuối cùng mà còn nhớ nó đã lặp đi lặp lại như thế nào từ một bản nháp đầu tiên có sai sót đến phiên bản cuối cùng. Hơn nữa, hệ thống bộ nhớ này cũng có thể là **đa phương thức (multimodal)**, cho phép agent phản tư và chỉnh sửa các đầu ra ngoài văn bản (như mã, hình ảnh, v.v.), đặt nền tảng cho việc xây dựng các agent đa phương thức mạnh mẽ hơn.

### 4.4.2 Thiết lập Trường hợp và Thiết kế Module Bộ nhớ

Để thể hiện cơ chế Reflection trong thực tế, chúng ta sẽ giới thiệu một cơ chế quản lý bộ nhớ, bởi vì phản tư thường tương ứng với việc lưu trữ và truy xuất thông tin. Nếu ngữ cảnh đủ dài, việc để "người đánh giá" trực tiếp lấy tất cả thông tin rồi phản tư thường đưa vào rất nhiều thông tin dư thừa. Trong bước thực hành này, chúng ta chủ yếu hoàn thành **việc tạo mã và tối ưu hóa lặp đi lặp lại**.

Nhiệm vụ mục tiêu cho bước này là: "Viết một hàm Python để tìm tất cả các số nguyên tố từ 1 đến n."

Nhiệm vụ này là một tình huống tuyệt vời để kiểm thử cơ chế Reflection:

1. **Tồn tại Con đường Tối ưu hóa Rõ ràng**: Mã ban đầu được tạo bởi mô hình ngôn ngữ lớn có khả năng là một triển khai đơn giản nhưng không hiệu quả.
2. **Điểm Phản tư Rõ ràng**: Thông qua phản tư, các vấn đề như "độ phức tạp thời gian quá cao" hoặc "tính toán dư thừa" có thể được phát hiện.
3. **Hướng Tối ưu hóa Rõ ràng**: Dựa trên phản hồi, nó có thể được tối ưu hóa thành một phiên bản lặp hiệu quả hơn hoặc một phiên bản sử dụng mô hình ghi nhớ (memoization).

Cốt lõi của Reflection nằm ở việc lặp lại, và điều kiện tiên quyết cho việc lặp lại là khả năng ghi nhớ các lần thử trước đó và phản hồi nhận được. Do đó, một module "bộ nhớ ngắn hạn" là thiết yếu để triển khai mô hình này. Module bộ nhớ này sẽ chịu trách nhiệm lưu trữ toàn bộ quỹ đạo của mỗi vòng lặp "thực thi-phản tư".

```python
from typing import List, Dict, Any, Optional

class Memory:
    """
    A simple short-term memory module for storing the agent's action and reflection trajectory.
    """

    def __init__(self):
        """
        Initialize an empty list to store all records.
        """
        self.records: List[Dict[str, Any]] = []

    def add_record(self, record_type: str, content: str):
        """
        Add a new record to memory.

        Parameters:
        - record_type (str): Type of record ('execution' or 'reflection').
        - content (str): Specific content of the record (e.g., generated code or reflection feedback).
        """
        record = {"type": record_type, "content": content}
        self.records.append(record)
        print(f"📝 Memory updated, added a '{record_type}' record.")

    def get_trajectory(self) -> str:
        """
        Format all memory records into a coherent string text for building prompts.
        """
        trajectory_parts = []
        for record in self.records:
            if record['type'] == 'execution':
                trajectory_parts.append(f"--- Previous Attempt (Code) ---\n{record['content']}")
            elif record['type'] == 'reflection':
                trajectory_parts.append(f"--- Reviewer Feedback ---\n{record['content']}")

        return "\n\n".join(trajectory_parts)

    def get_last_execution(self) -> Optional[str]:
        """
        Get the most recent execution result (e.g., the latest generated code).
        Returns None if it doesn't exist.
        """
        for record in reversed(self.records):
            if record['type'] == 'execution':
                return record['content']
        return None
```

Thiết kế của lớp `Memory` này tương đối ngắn gọn, với cấu trúc chính như sau:

- Sử dụng một danh sách `records` để lưu trữ từng hành động và phản tư theo thứ tự.
- Phương thức `add_record` chịu trách nhiệm thêm các mục mới vào bộ nhớ.
- Phương thức `get_trajectory` là cốt lõi; nó "tuần tự hóa" (serialize) quỹ đạo bộ nhớ thành một đoạn văn bản có thể được chèn trực tiếp vào các prompt tiếp theo, cung cấp ngữ cảnh đầy đủ cho việc phản tư và tối ưu hóa của mô hình.
- `get_last_execution` giúp thuận tiện trong việc lấy "bản nháp đầu tiên" mới nhất để phản tư.



### 4.4.3 Triển khai Mã nguồn của Agent Reflection

Với module `Memory` làm nền tảng, bây giờ chúng ta có thể tiến hành xây dựng logic cốt lõi của `ReflectionAgent`. Quy trình hoạt động của toàn bộ agent sẽ xoay quanh vòng lặp "thực thi-phản tư-tinh chỉnh" mà chúng ta đã thảo luận trước đó và hướng dẫn mô hình ngôn ngữ lớn đóng các vai trò khác nhau thông qua các prompt được thiết kế cẩn thận.

(1) Thiết kế Prompt

Không giống như các mô hình trước, cơ chế Reflection yêu cầu nhiều prompt cho các vai trò khác nhau phối hợp với nhau.

1. **Prompt Thực thi Ban đầu**: Đây là prompt cho lần thử đầu tiên của agent để giải quyết vấn đề, với nội dung tương đối đơn giản, chỉ yêu cầu mô hình hoàn thành nhiệm vụ được chỉ định.

```bash
INITIAL_PROMPT_TEMPLATE = """
You are a senior Python programmer. Please write a Python function according to the following requirements.
Your code must include a complete function signature, docstring, and follow PEP 8 coding standards.

Requirement: {task}

Please output the code directly without any additional explanations.
"""
```

2. **Prompt Phản tư**: Prompt này là linh hồn của cơ chế Reflection. Nó hướng dẫn mô hình đóng vai trò một "người đánh giá mã", phân tích một cách phê phán mã được tạo ở vòng trước, và cung cấp phản hồi cụ thể, có thể hành động được.

````bash
REFLECT_PROMPT_TEMPLATE = """
You are an extremely strict code review expert and senior algorithm engineer with ultimate requirements for code performance.
Your task is to review the following Python code and focus on finding its main bottlenecks in <strong>algorithm efficiency</strong>.

# Original Task:
{task}

# Code to Review:
```python
{code}
```

Please analyze the time complexity of this code and consider whether there is an <strong>algorithmically superior</strong> solution to significantly improve performance.
If one exists, please clearly point out the deficiencies of the current algorithm and propose specific, feasible algorithm improvement suggestions (e.g., using sieve method instead of trial division).
Only if the code has reached optimality at the algorithm level can you answer "no improvement needed."

Please output your feedback directly without any additional explanations.
"""
````

3. **Prompt Tinh chỉnh**: Sau khi nhận được phản hồi, prompt này sẽ hướng dẫn mô hình chỉnh sửa và tối ưu hóa mã gốc dựa trên nội dung phản hồi.

````bash
REFINE_PROMPT_TEMPLATE = """
You are a senior Python programmer. You are optimizing your code based on feedback from a code review expert.

# Original Task:
{task}

# Your Previous Code Attempt:
{last_code_attempt}
Reviewer's Feedback:
{feedback}

Please generate an optimized new version of the code based on the reviewer's feedback.
Your code must include a complete function signature, docstring, and follow PEP 8 coding standards.
Please output the optimized code directly without any additional explanations.
"""
````

(2) Đóng gói và Triển khai Agent

Bây giờ, chúng ta sẽ tích hợp bộ logic prompt này và module `Memory` vào lớp `ReflectionAgent`.

```python
# Assume llm_client.py and memory.py are already defined
# from llm_client import HelloAgentsLLM
# from memory import Memory

class ReflectionAgent:
    def __init__(self, llm_client, max_iterations=3):
        self.llm_client = llm_client
        self.memory = Memory()
        self.max_iterations = max_iterations

    def run(self, task: str):
        print(f"\n--- Starting to Process Task ---\nTask: {task}")

        # --- 1. Initial Execution ---
        print("\n--- Performing Initial Attempt ---")
        initial_prompt = INITIAL_PROMPT_TEMPLATE.format(task=task)
        initial_code = self._get_llm_response(initial_prompt)
        self.memory.add_record("execution", initial_code)

        # --- 2. Iterative Loop: Reflection and Refinement ---
        for i in range(self.max_iterations):
            print(f"\n--- Iteration {i+1}/{self.max_iterations} ---")

            # a. Reflection
            print("\n-> Performing Reflection...")
            last_code = self.memory.get_last_execution()
            reflect_prompt = REFLECT_PROMPT_TEMPLATE.format(task=task, code=last_code)
            feedback = self._get_llm_response(reflect_prompt)
            self.memory.add_record("reflection", feedback)

            # b. Check if stopping is needed
            if "no improvement needed" in feedback.lower():
                print("\n✅ Reflection considers code needs no improvement, task completed.")
                break

            # c. Refinement
            print("\n-> Performing Refinement...")
            refine_prompt = REFINE_PROMPT_TEMPLATE.format(
                task=task,
                last_code_attempt=last_code,
                feedback=feedback
            )
            refined_code = self._get_llm_response(refine_prompt)
            self.memory.add_record("execution", refined_code)

        final_code = self.memory.get_last_execution()
        print(f"\n--- Task Completed ---\nFinal Generated Code:\n```python\n{final_code}\n```")
        return final_code

    def _get_llm_response(self, prompt: str) -> str:
        """A helper method for calling LLM and getting complete streaming response."""
        messages = [{"role": "user", "content": prompt}]
        response_text = self.llm_client.think(messages=messages) or ""
        return response_text

```

### 4.4.4 Ví dụ Chạy và Phân tích

Mã hoàn chỉnh cũng có thể được tìm thấy trong thư mục `code` của kho mã đi kèm cuốn sách này; ở đây chúng ta cung cấp một ví dụ đầu ra.

````python
--- Starting to Process Task ---
Task: Write a Python function to find all prime numbers between 1 and n.

--- Performing Initial Attempt ---
🧠 Calling xxxxxx model...
✅ Large language model response successful:
```python
def find_primes(n):
    ...
    return primes
```
📝 Memory updated, added an 'execution' record.

--- Iteration 1/2 ---

-> Performing Reflection...
🧠 Calling xxxxxx model...
✅ Large language model response successful:
The current code has a time complexity of O(n * sqrt(n)). While this implementation is acceptable for smaller n values, performance will significantly degrade when n is very large. The main bottleneck is that each number needs trial division checking, leading to high time overhead.

It is recommended to use the Sieve of Eratosthenes algorithm, which has a time complexity of O(n log(log n)) and can significantly improve the efficiency of finding prime numbers.

Improved code as follows:
```python
def find_primes(n):
    ...
    return primes
```
📝 Memory updated, added a 'reflection' record.

-> Performing Refinement...
🧠 Calling xxxxxx model...
✅ Large language model response successful:
```python
def find_primes(n):
    ...
    return primes
```
📝 Memory updated, added an 'execution' record.

--- Iteration 2/2 ---

-> Performing Reflection...
🧠 Calling xxxxxx model...
✅ Large language model response successful:
The current code uses the Sieve of Eratosthenes with time complexity O(n log log n) and space complexity O(n). This algorithm is already very efficient for finding all prime numbers between 1 and n, and usually requires no further optimization. However, in some specific scenarios, the following improvements can be considered:

1. <strong>Segmented Sieve</strong>: Suitable for cases where n is very large but memory is limited. Divide the interval into multiple small segments, process each segment separately with the sieve method, reducing memory usage.
2. <strong>Odd Number Sieve</strong>: Except for 2, all prime numbers are odd. When initializing the `is_prime` array, only mark odd numbers, which can reduce space complexity by half while reducing some unnecessary calculations.

However, these improvements are not necessary for most application scenarios because the standard Sieve of Eratosthenes is already efficient enough. Therefore, in general cases, <strong>no improvement needed</strong>.
📝 Memory updated, added a 'reflection' record.

✅ Reflection considers code needs no improvement, task completed.

--- Task Completed ---
Final Generated Code:
```python
def find_primes(n):
    """
    Finds all prime numbers between 1 and n using the Sieve of Eratosthenes algorithm.

    :param n: The upper limit of the range to find prime numbers.
    :return: A list of all prime numbers between 1 and n.
    """
    if n < 2:
        return []

    is_prime = [True] * (n + 1)
    is_prime[0] = is_prime[1] = False

    p = 2
    while p * p <= n:
        if is_prime[p]:
            for i in range(p * p, n + 1, p):
                is_prime[i] = False
        p += 1

    primes = [num for num in range(2, n + 1) if is_prime[num]]
    return primes
```
````

Ví dụ chạy này minh họa cách cơ chế Reflection thúc đẩy agent thực hiện tối ưu hóa sâu:

1. **"Phê bình" Hiệu quả là Điều kiện Tiên quyết cho Tối ưu hóa**: Trong vòng phản tư đầu tiên, vì chúng ta sử dụng một prompt "cực kỳ nghiêm khắc" và "tập trung vào hiệu suất thuật toán", agent không hài lòng với mã ban đầu đúng về mặt chức năng mà chỉ ra chính xác nút thắt cổ chai về độ phức tạp thời gian `O(n * sqrt(n))` của nó và đề xuất các cải thiện ở cấp độ thuật toán—Sàng Eratosthenes (Sieve of Eratosthenes).
2. **Cải thiện Lặp đi lặp lại**: Sau khi nhận được phản hồi rõ ràng, agent triển khai thành công một phương pháp sàng hiệu quả hơn trong giai đoạn tinh chỉnh, giảm độ phức tạp thuật toán xuống `O(n log log n)`, hoàn thành lần tự lặp có ý nghĩa đầu tiên.
3. **Hội tụ và Kết thúc**: Trong vòng phản tư thứ hai, đối mặt với phương pháp sàng vốn đã hiệu quả, agent thể hiện kiến thức sâu sắc hơn. Nó không chỉ khẳng định hiệu quả của thuật toán hiện tại mà thậm chí còn đề cập đến các hướng tối ưu hóa nâng cao hơn như sàng phân đoạn (segmented sieve), nhưng cuối cùng đưa ra phán đoán đúng đắn là "không cần cải thiện trong các trường hợp thông thường". Phán đoán này kích hoạt điều kiện kết thúc của chúng ta, cho phép quá trình tối ưu hóa hội tụ.

Trường hợp này chứng minh đầy đủ rằng giá trị của một cơ chế Reflection được thiết kế tốt không chỉ nằm ở việc sửa lỗi mà quan trọng hơn là **thúc đẩy các lời giải đạt được những cải thiện từng bước về chất lượng và hiệu suất**, khiến nó trở thành một trong những công nghệ chủ chốt để xây dựng các agent phức tạp, chất lượng cao.

### 4.4.5 Phân tích Chi phí-Lợi ích của Cơ chế Reflection

Mặc dù cơ chế Reflection thể hiện xuất sắc trong việc cải thiện chất lượng lời giải cho nhiệm vụ, năng lực này không phải là không có chi phí. Trong các ứng dụng thực tế, chúng ta cần cân nhắc giữa các lợi ích mà nó mang lại và các chi phí tương ứng.

(1) Các Chi phí Chính

1. **Tăng Chi phí Gọi Mô hình**: Đây là chi phí trực tiếp nhất. Mỗi lần lặp yêu cầu ít nhất hai lệnh gọi mô hình ngôn ngữ lớn bổ sung (một cho phản tư, một cho tinh chỉnh). Nếu lặp nhiều vòng, chi phí gọi API và tiêu thụ tài nguyên tính toán sẽ tăng theo cấp số nhân.

2. **Tăng Đáng kể Độ trễ Nhiệm vụ**: Reflection là một quá trình tuần tự; mỗi vòng tinh chỉnh phải chờ vòng phản tư trước đó hoàn thành. Điều này kéo dài đáng kể tổng thời gian nhiệm vụ, làm cho nó không phù hợp với các tình huống có yêu cầu cao về thời gian thực.

3. **Tăng Độ phức tạp của Prompt Engineering**: Như trường hợp của chúng ta minh họa, sự thành công của Reflection phần lớn phụ thuộc vào các prompt chất lượng cao, có mục tiêu. Việc thiết kế và gỡ lỗi các prompt hiệu quả cho các giai đoạn khác nhau như "thực thi", "phản tư", và "tinh chỉnh" đòi hỏi nhiều công sức phát triển hơn.

(2) Các Lợi ích Cốt lõi

1. **Bước nhảy vọt về Chất lượng Lời giải**: Lợi ích lớn nhất là nó có thể tối ưu hóa lặp đi lặp lại một lời giải ban đầu "đạt yêu cầu" thành một lời giải cuối cùng "xuất sắc". Cải thiện này từ đúng về chức năng đến hiệu quả về hiệu suất, từ logic thô sơ đến logic chặt chẽ, là rất quan trọng trong nhiều nhiệm vụ then chốt.

2. **Tăng cường Tính bền vững và Độ tin cậy**: Thông qua các vòng lặp tự sửa lỗi nội bộ, agent có thể phát hiện và sửa các sai sót logic tiềm ẩn, lỗi thực tế, hoặc việc xử lý các trường hợp biên không phù hợp trong lời giải ban đầu, cải thiện đáng kể độ tin cậy của kết quả cuối cùng.

Tóm lại, cơ chế Reflection là một chiến lược "đánh đổi chi phí lấy chất lượng" điển hình. Nó rất phù hợp với các tình huống **có yêu cầu cực kỳ cao về chất lượng, độ chính xác, và độ tin cậy của kết quả cuối cùng, và có yêu cầu tương đối thoải mái về tính thời gian thực của việc hoàn thành nhiệm vụ**. Ví dụ:

- Tạo mã nghiệp vụ quan trọng hoặc báo cáo kỹ thuật.
- Tiến hành suy luận logic phức tạp trong nghiên cứu khoa học.
- Các hệ thống hỗ trợ ra quyết định yêu cầu phân tích và lập kế hoạch sâu.

Ngược lại, nếu tình huống ứng dụng yêu cầu phản hồi nhanh, hoặc một câu trả lời "gần đúng" đã đủ, thì việc sử dụng các mô hình nhẹ hơn như ReAct hoặc Plan-and-Solve có thể là một lựa chọn hiệu quả hơn về chi phí.

## 4.5 Tóm tắt Chương

Trong chương này, dựa trên kiến thức về mô hình ngôn ngữ lớn đã nắm vững ở Chương 3, chúng ta đã lập trình và triển khai ba mô hình xây dựng agent kinh điển của ngành từ đầu thông qua việc "tự chế tạo bánh xe": ReAct, Plan-and-Solve, và Reflection. Chúng ta không chỉ khám phá các nguyên lý hoạt động cốt lõi của chúng mà còn hiểu sâu sắc các ưu điểm, hạn chế, và các tình huống áp dụng tương ứng của chúng thông qua các trường hợp thực hành cụ thể.

**Ôn tập Kiến thức Cốt lõi:**

1. ReAct: Chúng ta đã xây dựng một agent ReAct có thể tương tác với thế giới bên ngoài. Thông qua vòng lặp động "tư duy-hành động-quan sát", nó đã sử dụng thành công các công cụ tìm kiếm để trả lời các câu hỏi thời gian thực mà cơ sở kiến thức của chính nó không thể bao phủ. Các ưu điểm cốt lõi của nó nằm ở **khả năng thích ứng với môi trường** và **khả năng sửa lỗi động**, khiến nó trở thành lựa chọn hàng đầu để xử lý các nhiệm vụ khám phá yêu cầu đầu vào từ công cụ bên ngoài.
2. Plan-and-Solve: Chúng ta đã triển khai một agent Plan-and-Solve lập kế hoạch trước rồi thực thi, và sử dụng nó để giải các bài toán đố toán yêu cầu suy luận nhiều bước. Nó phân rã các nhiệm vụ phức tạp thành các bước rõ ràng, sau đó thực thi chúng từng bước một. Các ưu điểm cốt lõi của nó nằm ở **tính cấu trúc** và **tính ổn định**, đặc biệt phù hợp để xử lý các nhiệm vụ có con đường logic xác định và suy luận nội bộ chuyên sâu.
3. Reflection (Tự Phản tư và Lặp lại): Chúng ta đã xây dựng một agent Reflection với năng lực tự tối ưu hóa. Bằng cách giới thiệu vòng lặp lặp đi lặp lại "thực thi-phản tư-tinh chỉnh", nó đã tối ưu hóa thành công một lời giải mã ban đầu không hiệu quả thành một phiên bản hiệu suất cao vượt trội về mặt thuật toán. Giá trị cốt lõi của nó nằm ở việc **cải thiện đáng kể chất lượng lời giải**, phù hợp với các tình huống có yêu cầu cực kỳ cao về độ chính xác và độ tin cậy của kết quả.

Ba mô hình được khám phá trong chương này đại diện cho ba chiến lược khác nhau để agent giải quyết vấn đề, như minh họa trong Bảng 4.1. Trong các ứng dụng thực tế, việc chọn mô hình nào phụ thuộc vào các yêu cầu cốt lõi của nhiệm vụ:

<div align="center">
<p>Bảng 4.1 Chiến lược Lựa chọn cho Các Vòng lặp Agent Khác nhau</p>
<img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-4.png" alt="" width="70%"/>
</div>

Đến đây, chúng ta đã nắm vững các công nghệ cốt lõi để xây dựng các agent riêng lẻ. Để chuyển hóa kiến thức và có được những hiểu biết sâu sắc hơn về các ứng dụng thực tế, trong phần tiếp theo chúng ta sẽ khám phá cách sử dụng các nền tảng low-code khác nhau và các giải pháp mã nhẹ để xây dựng agent.

## Bài tập

> **Lưu ý**: Một số bài tập không có đáp án chuẩn; trọng tâm là trau dồi sự hiểu biết toàn diện và khả năng thực hành của người học trong việc thiết kế mô hình agent.

1. Chương này đã giới thiệu ba mô hình agent kinh điển: `ReAct`, `Plan-and-Solve`, và `Reflection`. Hãy phân tích:

   - Những khác biệt bản chất trong cách ba mô hình này tổ chức "tư duy" và "hành động" là gì?
   - Nếu bạn thiết kế một "trợ lý điều khiển nhà thông minh" (cần điều khiển đèn, điều hòa, rèm cửa, và các thiết bị khác, và tự động điều chỉnh dựa trên thói quen của người dùng), bạn sẽ chọn mô hình nào làm kiến trúc cơ sở? Tại sao?
   - Ba mô hình này có thể kết hợp được không? Nếu có, hãy thử thiết kế một kiến trúc agent mô hình lai (hybrid) và giải thích các tình huống áp dụng của nó.

2. Trong triển khai `ReAct` ở Phần 4.2, chúng ta đã sử dụng biểu thức chính quy để phân tích đầu ra của mô hình ngôn ngữ lớn (như `Thought` và `Action`). Hãy suy nghĩ:

   - Những tính dễ vỡ tiềm ẩn nào tồn tại trong phương pháp phân tích hiện tại? Trong những trường hợp nào nó có thể thất bại?
   - Ngoài biểu thức chính quy, có những giải pháp phân tích đầu ra bền vững hơn nào?
   - Hãy thử chỉnh sửa mã trong chương này để sử dụng một định dạng đầu ra đáng tin cậy hơn, và so sánh ưu nhược điểm của hai cách tiếp cận.

3. Việc gọi công cụ là một trong những năng lực cốt lõi của các agent hiện đại. Dựa trên thiết kế `ToolExecutor` ở Phần 4.2.2, hãy hoàn thành bài thực hành mở rộng sau:

   > **Lưu ý**: Đây là một bài thực hành thực tế; khuyến nghị nên thực sự viết mã.

   - Thêm một công cụ "máy tính" (calculator) vào agent `ReAct` để nó có thể xử lý các bài toán tính toán phức tạp (chẳng hạn như "Tính kết quả của `(123 + 456) × 789 / 12 = ?`").
   - Thiết kế và triển khai một cơ chế xử lý "thất bại trong việc chọn công cụ": khi agent liên tục gọi sai công cụ hoặc cung cấp sai tham số, hệ thống nên hướng dẫn nó sửa lỗi như thế nào?
   - Hãy suy nghĩ: Nếu số lượng công cụ có thể gọi tăng lên 50 hoặc thậm chí 100, phương pháp mô tả công cụ hiện tại có còn hoạt động hiệu quả không? Từ góc độ kỹ thuật, làm thế nào chúng ta có thể tối ưu hóa cơ chế tổ chức và truy xuất công cụ khi số lượng công cụ có thể gọi tăng lên đáng kể theo nhu cầu nghiệp vụ?

4. Mô hình `Plan-and-Solve` phân rã các nhiệm vụ thành hai giai đoạn: "lập kế hoạch" và "thực thi". Hãy phân tích sâu:

   - Trong triển khai ở Phần 4.3, kế hoạch được tạo ra trong giai đoạn lập kế hoạch là "tĩnh" (tạo ra một lần, không thể sửa đổi). Nếu trong quá trình thực thi phát hiện rằng một bước nào đó không thể hoàn thành hoặc kết quả không đáp ứng kỳ vọng, làm thế nào để thiết kế một cơ chế "lập kế hoạch lại động" (dynamic replanning)?
   - So sánh `Plan-and-Solve` với `ReAct`: Khi xử lý một nhiệm vụ như "đặt một chuyến công tác từ Bắc Kinh đến Thượng Hải (bao gồm chuyến bay, khách sạn, thuê xe)", mô hình nào phù hợp hơn? Tại sao?
   - Hãy thử thiết kế một hệ thống "lập kế hoạch phân cấp" (hierarchical planning): trước tiên tạo một kế hoạch trừu tượng cấp cao, sau đó tạo các kế hoạch con chi tiết cho mỗi bước cấp cao. Thiết kế này có những ưu điểm gì?

5. Cơ chế `Reflection` cải thiện chất lượng đầu ra thông qua vòng lặp "thực thi-phản tư-tinh chỉnh". Hãy suy nghĩ:

   - Trong trường hợp tạo mã ở Phần 4.4, cùng một mô hình được sử dụng cho các giai đoạn khác nhau. Nếu sử dụng hai mô hình khác nhau (ví dụ, sử dụng một mô hình mạnh hơn cho phản tư và một mô hình nhanh hơn cho thực thi), điều đó sẽ có tác động gì?
   - Điều kiện kết thúc cho cơ chế `Reflection` là "phản hồi chứa **no improvement needed**" hoặc "đạt đến số lần lặp tối đa". Thiết kế này có hợp lý không? Có thể thiết kế một điều kiện kết thúc thông minh hơn không?
   - Giả sử bạn muốn xây dựng một "trợ lý viết bài báo học thuật" có thể tạo bản nháp và liên tục tối ưu hóa nội dung bài báo. Hãy thiết kế một cơ chế Reflection đa chiều phản tư và cải thiện từ nhiều góc độ như logic đoạn văn, tính đổi mới của phương pháp, cách diễn đạt ngôn ngữ, và chuẩn trích dẫn.

6. Prompt engineering là một công nghệ chủ chốt ảnh hưởng đến hiệu quả cuối cùng của agent. Chương này đã minh họa nhiều template prompt được thiết kế cẩn thận. Hãy phân tích:

   - So sánh prompt `ReAct` ở Phần 4.2.3 và prompt `Plan-and-Solve` ở Phần 4.3.2; chúng rõ ràng có những khác biệt đáng kể trong thiết kế cấu trúc. Những khác biệt này phục vụ logic cốt lõi của các mô hình tương ứng của chúng như thế nào?
   - Trong prompt `Reflection` ở Phần 4.4.3, chúng ta đã sử dụng một thiết lập vai trò như "bạn là một chuyên gia đánh giá mã cực kỳ nghiêm khắc". Hãy thử chỉnh sửa thiết lập vai trò này (chẳng hạn như đổi thành "bạn là một người duy trì dự án mã nguồn mở coi trọng khả năng đọc mã"), quan sát các thay đổi trong kết quả đầu ra, và tổng kết tác động của các thiết lập vai trò đối với hành vi của agent.
   - Việc thêm các ví dụ `few-shot` vào prompt thường có thể cải thiện đáng kể khả năng của mô hình trong việc tuân theo các định dạng cụ thể. Hãy thử thêm các ví dụ `few-shot` vào một trong các agent trong chương này và so sánh hiệu quả.

7. Một công ty khởi nghiệp thương mại điện tử hiện nay hy vọng sử dụng một "agent chăm sóc khách hàng" để thay thế nhân viên chăm sóc khách hàng nhằm giảm chi phí và nâng cao hiệu quả. Nó cần có các chức năng sau:

   a. Hiểu lý do yêu cầu hoàn tiền của người dùng

   b. Truy vấn thông tin đơn hàng và trạng thái vận chuyển của người dùng

   c. Đánh giá thông minh xem có nên chấp thuận hoàn tiền hay không dựa trên chính sách công ty

   d. Tạo một email trả lời phù hợp và gửi đến email của người dùng

   e. Nếu quyết định đánh giá có phần gây tranh cãi (độ tự tin dưới ngưỡng), có thể tự phản tư và đưa ra các đề xuất thận trọng hơn

   Với tư cách là product manager của sản phẩm này:
   - Bạn sẽ chọn mô hình nào (hoặc tổ hợp mô hình nào) từ chương này làm kiến trúc cốt lõi của hệ thống?
   - Hệ thống này cần những công cụ nào? Hãy liệt kê ít nhất 3 công cụ và mô tả chức năng của chúng.
   - Làm thế nào để thiết kế prompt nhằm đảm bảo rằng các quyết định của agent vừa phù hợp với lợi ích của công ty vừa duy trì thái độ thân thiện với người dùng?
   - Sản phẩm này có thể đối mặt với những rủi ro và thách thức nào sau khi ra mắt? Làm thế nào có thể giảm thiểu những rủi ro này thông qua các biện pháp kỹ thuật?

## Tài liệu tham khảo

[1] Yao S, Zhao J, Yu D, et al. React: Synergizing reasoning and acting in language models[C]//International Conference on Learning Representations (ICLR). 2023.

[2] Wang L, Xu W, Lan Y, et al. Plan-and-solve prompting: Improving zero-shot chain-of-thought reasoning by large language models[J]. arXiv preprint arXiv:2305.04091, 2023.

[3] Shinn N, Cassano F, Gopinath A, et al. Reflexion: Language agents with verbal reinforcement learning[J]. Advances in Neural Information Processing Systems, 2023, 36: 8634-8652.
