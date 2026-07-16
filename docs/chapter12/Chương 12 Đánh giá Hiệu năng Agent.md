# Chương 12 Đánh giá Hiệu năng Agent

Trong các chương trước, chúng ta đã xây dựng các chức năng cốt lõi của framework HelloAgents, hiện thực hóa nhiều paradigm agent khác nhau, hệ thống công cụ, cơ chế bộ nhớ (memory) và huấn luyện học tăng cường (reinforcement learning). Khi xây dựng các hệ thống agent, chúng ta cũng cần giải quyết một vấn đề cốt lõi: **Làm thế nào để đánh giá hiệu năng của agent một cách khách quan?** Cụ thể, chúng ta cần trả lời các câu hỏi sau:

1. Agent có sở hữu những năng lực như kỳ vọng hay không?
2. Nó hoạt động ra sao trên các nhiệm vụ khác nhau?
3. So với các agent khác, nó đang ở mức độ nào?

Chương này sẽ bổ sung một **Hệ thống Đánh giá Hiệu năng** vào HelloAgents. Chúng ta sẽ tìm hiểu sâu về nền tảng lý thuyết của việc đánh giá agent và triển khai các công cụ đánh giá.

## 12.1 Nền tảng Đánh giá Agent

### 12.1.1 Tại sao cần Đánh giá Agent

Hiện tại chúng ta đã có SimpleAgent, vốn đã sở hữu năng lực suy luận và gọi công cụ mạnh mẽ. Hãy cùng xem một kịch bản sử dụng điển hình:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import SearchTool

# Create LLM and agent
llm = HelloAgentsLLM()

# Create a system prompt emphasizing tool use
system_prompt = """You are an AI assistant that can use search tools to obtain the latest information.

When you need to search for information, please use the following format:
[TOOL_CALL:search:search keywords]

For example:
- [TOOL_CALL:search:latest AI news]
- [TOOL_CALL:search:Python programming tutorial]

Please use the search tool to obtain the latest information before answering questions."""

agent = SimpleAgent(name="AI Assistant", llm=llm, system_prompt=system_prompt)

# Add search tool
agent.add_tool(SearchTool())

# Example: Use search tool to answer questions
response = agent.run("What are the latest AI technology development trends?")
print(f"\nAnswer: {response}")
```

Agent này có thể hoạt động bình thường, nhưng chúng ta đối mặt với một vấn đề cốt lõi: Làm thế nào để đánh giá hiệu năng của nó một cách khách quan? Khi chúng ta tối ưu prompt hoặc thay đổi mô hình LLM, làm sao biết được liệu có cải thiện thực sự hay không? Trước khi triển khai lên môi trường production, làm sao đảm bảo độ tin cậy của agent? Tất cả những câu hỏi này đều cần được giải quyết thông qua đánh giá có hệ thống.

Giá trị cốt lõi của việc đánh giá agent nằm ở chỗ cung cấp các phương pháp chuẩn hóa để đo lường năng lực của agent. Thông qua đánh giá, chúng ta có thể định lượng hiệu năng của agent bằng các chỉ số (metric) cụ thể, so sánh khách quan ưu nhược điểm của các phương án thiết kế khác nhau, kịp thời phát hiện điểm yếu của agent trong các kịch bản cụ thể, và chứng minh độ tin cậy của agent với người dùng.

Khác với kiểm thử phần mềm truyền thống, đánh giá agent đối mặt với những thách thức đặc thù. Đầu tiên là tính không chắc chắn của đầu ra (output uncertainty) - cùng một câu hỏi có thể có nhiều câu trả lời đúng, khiến việc phán đoán đúng/sai đơn giản trở nên khó khăn. Thứ hai là sự đa dạng của tiêu chí đánh giá - các nhiệm vụ khác nhau đòi hỏi phương pháp đánh giá khác nhau; việc gọi công cụ cần kiểm tra chữ ký hàm (function signature), trong khi nhiệm vụ hỏi đáp (Q&A) cần đánh giá độ tương đồng ngữ nghĩa. Cuối cùng là chi phí đánh giá cao - mỗi lần đánh giá đòi hỏi rất nhiều lời gọi API, có thể tốn hàng trăm nhân dân tệ hoặc hơn.

Để giải quyết những thách thức này, giới học thuật và công nghiệp đã đề xuất nhiều **Benchmark** (bộ đánh giá chuẩn) chuẩn hóa. Các benchmark này cung cấp tập dữ liệu thống nhất, các chỉ số đánh giá và phương pháp chấm điểm, giúp chúng ta đánh giá và so sánh các hệ thống agent khác nhau dưới cùng một tiêu chuẩn.

### 12.1.2 Tổng quan các Benchmark Đánh giá Chủ đạo

Trong lĩnh vực đánh giá agent đã xuất hiện nhiều benchmark có sức ảnh hưởng. Dưới đây là một số benchmark và chỉ số đánh giá chủ đạo:

**(1) Đánh giá Năng lực Gọi Công cụ (Tool Invocation)**

Gọi công cụ là một trong những năng lực cốt lõi của agent. Agent cần hiểu ý định người dùng, chọn công cụ phù hợp và xây dựng lời gọi hàm chính xác. Các benchmark liên quan bao gồm:

- **BFCL (Berkeley Function Calling Leaderboard)**<sup>[1]</sup>: Được UC Berkeley phát hành, gồm hơn 1120 mẫu kiểm thử, bao phủ bốn loại: simple, multiple, parallel, irrelevance, sử dụng thuật toán khớp AST (AST matching) để đánh giá, quy mô tập dữ liệu vừa phải, cộng đồng năng động.
- **ToolBench**<sup>[2]</sup>: Được Đại học Thanh Hoa phát hành, gồm hơn 16000 kịch bản gọi API thực tế, bao phủ các kịch bản sử dụng công cụ phức tạp trong thế giới thực.
- **API-Bank**<sup>[3]</sup>: Được Microsoft Research phát hành, gồm 53 công cụ API thường dùng, tập trung đánh giá khả năng hiểu và gọi tài liệu API của agent.

**(2) Đánh giá Năng lực Tổng quát**

Đánh giá hiệu năng tổng hợp của agent trong các nhiệm vụ thế giới thực, bao gồm suy luận nhiều bước, ứng dụng kiến thức, hiểu đa phương thức (multimodal), v.v.:

- **GAIA (General AI Assistants)**<sup>[4]</sup>: Được Meta AI và Hugging Face đồng phát hành, gồm 466 bài toán thế giới thực, chia thành các cấp độ khó Level 1/2/3, đánh giá năng lực suy luận nhiều bước, sử dụng công cụ, xử lý tệp, duyệt web, sử dụng thuật toán Quasi Exact Match, các nhiệm vụ mang tính thực tế và toàn diện.
- **AgentBench**<sup>[5]</sup>: Được Đại học Thanh Hoa phát hành, gồm 8 nhiệm vụ thuộc các lĩnh vực khác nhau, đánh giá toàn diện năng lực tổng quát của agent.
- **WebArena**<sup>[6]</sup>: Được CMU phát hành, đánh giá năng lực hoàn thành nhiệm vụ và tương tác web của agent trong môi trường web thực tế.

**(3) Đánh giá Cộng tác Đa-Agent (Multi-Agent Collaboration)**

Đánh giá khả năng làm việc cộng tác của nhiều agent:

- **ChatEval**<sup>[7]</sup>: Đánh giá chất lượng của các hệ thống hội thoại đa-agent.
- **SOTOPIA**<sup>[8]</sup>: Đánh giá năng lực tương tác của agent trong các kịch bản xã hội.
- **Kịch bản Cộng tác Tùy chỉnh**: Các nhiệm vụ đánh giá được thiết kế theo kịch bản ứng dụng cụ thể.

**(4) Các Chỉ số Đánh giá Thường gặp**

Các benchmark khác nhau sử dụng các chỉ số đánh giá khác nhau, thường gặp gồm:

- **Chỉ số Độ chính xác (Accuracy Metrics)**: Accuracy, Exact Match, F1 Score, dùng để đo tính đúng đắn của câu trả lời.
- **Chỉ số Hiệu suất (Efficiency Metrics)**: Response Time, Token Usage, dùng để đo hiệu suất thực thi.
- **Chỉ số Độ bền vững (Robustness Metrics)**: Error Rate, Failure Recovery, dùng để đo khả năng chịu lỗi.
- **Chỉ số Cộng tác (Collaboration Metrics)**: Communication Efficiency, Task Completion, dùng để đo hiệu quả cộng tác.

### 12.1.3 Thiết kế Hệ thống Đánh giá của HelloAgents

Cân nhắc đến đường cong học tập và tính thực tiễn, chương này sẽ tập trung vào các kịch bản đánh giá sau:

1. **BFCL**: Đánh giá năng lực gọi công cụ
   - Lý do lựa chọn: Quy mô tập dữ liệu vừa phải, chỉ số đánh giá rõ ràng, cộng đồng năng động
   - Kịch bản áp dụng: Đánh giá độ chính xác gọi hàm của agent

2. **GAIA**: Đánh giá năng lực trợ lý AI tổng quát
   - Lý do lựa chọn: Nhiệm vụ thực tế, phân cấp độ khó, tính toàn diện cao
   - Kịch bản áp dụng: Đánh giá năng lực giải quyết vấn đề tổng hợp của agent

3. **Đánh giá Chất lượng Sinh Dữ liệu (Data Generation Quality Evaluation)**: Đánh giá chất lượng dữ liệu do LLM sinh ra
   - Lý do lựa chọn: Thông qua tình huống này, trải nghiệm minh họa hoàn chỉnh việc dùng Agent để tạo dữ liệu và đánh giá dữ liệu
   - Kịch bản áp dụng: Đánh giá chất lượng dữ liệu huấn luyện và dữ liệu kiểm thử được sinh ra
   - Phương pháp đánh giá: LLM Judge, Win Rate, kiểm tra thủ công

Thông qua ba kịch bản đánh giá này, chúng ta sẽ xây dựng một hệ thống đánh giá hoàn chỉnh. Hình 12.1 minh họa phương pháp tiếp cận xây dựng hệ thống đánh giá của chúng ta.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-1.png" alt="" width="85%"/>
  <p>Hình 12.1 Kiến trúc Hệ thống Đánh giá của HelloAgents</p>
</div>



### 12.1.4 Mục tiêu Học tập của Chương và Trải nghiệm Nhanh

Trước tiên hãy cùng xem nội dung học tập của Chương 12:

```
hello_agents/
├── evaluation/                         # Evaluation module
│   └── benchmarks/                     # Evaluation benchmark implementation
│       ├── bfcl/                       # BFCL evaluation implementation
│       │   ├── dataset.py              # BFCL dataset loader
│       │   ├── evaluator.py            # BFCL evaluator (AST matching)
│       │   ├── metrics.py              # BFCL-specific metrics
│       │   └── ast_matcher.py          # AST matching algorithm
│       ├── gaia/                       # GAIA evaluation implementation
│       │   ├── dataset.py              # GAIA dataset loader
│       │   ├── evaluator.py            # GAIA evaluator (quasi exact match)
│       │   ├── metrics.py              # GAIA-specific metrics
│       │   └── quasi_exact_match.py    # Quasi exact match algorithm
│       └── data_generation/            # Data generation evaluation implementation
│           ├── dataset.py              # AIME dataset loader
│           ├── llm_judge.py            # LLM Judge evaluator
│           └── win_rate.py             # Win Rate evaluator
└── tools/builtin/                      # Built-in tools module
    ├── bfcl_evaluation_tool.py         # BFCL evaluation tool
    ├── gaia_evaluation_tool.py         # GAIA evaluation tool
    ├── llm_judge_tool.py               # LLM Judge tool
    └── win_rate_tool.py                # Win Rate tool
```

Đối với nội dung chương này, mục tiêu học tập là nắm vững khả năng áp dụng các công cụ đánh giá. Trước tiên hãy chuẩn bị môi trường phát triển:

```bash
# Install HelloAgents framework (Chapter 12 version)
pip install "hello-agents[evaluation]==0.2.7"

# Set environment variables
export HF_TOKEN="your_huggingface_token"     # For GAIA dataset (setup steps will follow)

# Since the official `bfcl-eval` package requires numpy<=2.0.0, which conflicts with HelloAgents main dependencies, separate installation is needed
pip install "numpy==1.26.4" bfcl-eval
```

Trong các phần tiếp theo, chúng ta sẽ tìm hiểu sâu về cách sử dụng và giới thiệu chi tiết của từng phương pháp đánh giá.

## 12.2 BFCL: Đánh giá Năng lực Gọi Công cụ

### 12.2.1 Giới thiệu Benchmark BFCL

BFCL (Berkeley Function Calling Leaderboard) là một benchmark đánh giá năng lực gọi hàm (function calling) do UC Berkeley phát hành<sup>[1]</sup>. Trong các hệ thống agent, gọi công cụ là một trong những năng lực cốt lõi. Agent cần hoàn thành các nhiệm vụ sau:

1. **Hiểu Yêu cầu Nhiệm vụ**: Trích xuất thông tin then chốt từ mô tả bằng ngôn ngữ tự nhiên của người dùng
2. **Chọn Công cụ Phù hợp**: Chọn công cụ thích hợp nhất từ tập công cụ có sẵn
3. **Xây dựng Lời gọi Hàm**: Điền chính xác tên hàm và tham số
4. **Xử lý Kịch bản Phức tạp**: Hỗ trợ các kịch bản nâng cao như gọi nhiều hàm, gọi song song

Benchmark BFCL bao gồm bốn loại đánh giá với độ khó tăng dần. Bắt đầu từ lời gọi hàm đơn cơ bản nhất (Simple), tăng dần đến kịch bản đòi hỏi gọi nhiều hàm (Multiple), rồi đến kịch bản phức tạp đòi hỏi gọi song song nhiều hàm (Parallel), và cuối cùng là kịch bản đòi hỏi phán đoán liệu có cần gọi hàm hay không (Irrelevance). Bốn loại này bao phủ các kịch bản gọi công cụ đa dạng mà agent có thể gặp trong ứng dụng thực tế, như minh họa trong Bảng 12.1:

<div align="center">
  <p>Bảng 12.1 Bốn Loại Đánh giá trong Benchmark BFCL</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-table-1.png" alt="" width="85%"/>
</div>

Quy trình đánh giá BFCL tuân theo các bước kiểm thử benchmark chuẩn: đầu tiên nạp tập dữ liệu và chọn loại đánh giá, sau đó chạy agent để thu được kết quả dự đoán, tiếp theo phân tích kết quả dự đoán thành Cây Cú pháp Trừu tượng (Abstract Syntax Tree - AST), và cuối cùng phán đoán liệu dự đoán có chính xác hay không thông qua thuật toán khớp AST. Toàn bộ quy trình duyệt qua tất cả các mẫu kiểm thử, cuối cùng tính toán các chỉ số đánh giá như độ chính xác và sinh ra báo cáo đánh giá. Quy trình đánh giá hoàn chỉnh được minh họa trong Hình 12.2:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-2.png" alt="" width="85%"/>
  <p>Hình 12.2 Sơ đồ Quy trình Đánh giá BFCL</p>
</div>

**(1) Cấu trúc Tập dữ liệu BFCL**

Tập dữ liệu BFCL sử dụng định dạng JSON, mỗi mẫu kiểm thử chứa các trường sau:

```json
{
  "id": "simple_001",
  "question": "What's the weather like in Beijing today?",
  "function": [
    {
      "name": "get_weather",
      "description": "Get the current weather for a location",
      "parameters": {
        "type": "object",
        "properties": {
          "location": {
            "type": "string",
            "description": "The city name"
          }
        },
        "required": ["location"]
      }
    }
  ],
  "ground_truth": [
    {
      "name": "get_weather",
      "arguments": {
        "location": "Beijing"
      }
    }
  ]
}
```

**Mô tả các Trường Then chốt:**

- `question`: Yêu cầu bằng ngôn ngữ tự nhiên của người dùng
- `function`: Danh sách các hàm có sẵn (bao gồm chữ ký hàm và mô tả)
- `ground_truth`: Đáp án chuẩn (lời gọi hàm kỳ vọng)

**(2) Giải thích Khớp AST**

BFCL sử dụng **Khớp AST (AST Matching - Khớp Cây Cú pháp Trừu tượng)** làm thuật toán đánh giá cốt lõi, vì vậy hãy cùng tìm hiểu chiến lược đánh giá dưới đây.

BFCL sử dụng Cây Cú pháp Trừu tượng (AST) để khớp thông minh, thay vì khớp chuỗi đơn giản. Ý tưởng cốt lõi của khớp AST là: **Phân tích lời gọi hàm thành cây cú pháp, sau đó so sánh cấu trúc cây và giá trị các nút**.

Cho lời gọi hàm dự đoán $P$ và đáp án chuẩn $G$, hàm khớp AST được định nghĩa như sau:

$$
\text{AST\_Match}(P, G) = \begin{cases}
1 & \text{if } \text{AST}(P) \equiv \text{AST}(G) \\
0 & \text{otherwise}
\end{cases}
$$

Trong đó $\text{AST}(x)$ biểu diễn việc phân tích lời gọi hàm thành cây cú pháp trừu tượng, $\equiv$ biểu diễn sự tương đương của cây cú pháp.

Hai cây cú pháp là tương đương nếu chúng thỏa mãn ba điều kiện cốt lõi: tên hàm phải hoàn toàn giống nhau (khớp chính xác), tập cặp khóa-giá trị tham số bằng nhau (bỏ qua thứ tự), và mỗi giá trị tham số tương đương về mặt ngữ nghĩa (ví dụ `2+3` tương đương với `5`). Trong quá trình khớp cụ thể, việc khớp tên hàm đòi hỏi khớp chuỗi chính xác, ví dụ `get_weather` và `get_temperature` được coi là hai hàm khác nhau. Việc khớp tham số sử dụng AST để so sánh thông minh, cho phép thứ tự tham số khác nhau (`f(a=1, b=2)` tương đương với `f(b=2, a=1)`), cho phép biểu thức tương đương (`f(x=2+3)` tương đương với `f(x=5)`), và cũng cho phép biểu diễn chuỗi khác nhau (`f(s="hello")` tương đương với `f(s='hello')`). Đối với kịch bản gọi nhiều hàm, thuật toán khớp đòi hỏi gọi cùng số lượng hàm, mỗi lời gọi hàm đều phải khớp, nhưng thứ tự gọi có thể khác nhau (sử dụng khớp tập hợp).

**Ví dụ Khớp AST:**

```python
# Example 1: Different parameter order (match successful)
Prediction: get_weather(city="Beijing", unit="celsius")
Standard: get_weather(unit="celsius", city="Beijing")
Result: ✅ Match successful

# Example 2: Equivalent expression (match successful)
Prediction: calculate(x=2+3)
Standard: calculate(x=5)
Result: ✅ Match successful

# Example 3: Wrong function name (match failed)
Prediction: get_temperature(city="Beijing")
Standard: get_weather(city="Beijing")
Result: ❌ Match failed

# Example 4: Wrong parameter value (match failed)
Prediction: get_weather(city="Shanghai")
Standard: get_weather(city="Beijing")
Result: ❌ Match failed
```

**(3) Các Chỉ số Đánh giá BFCL**

BFCL sử dụng các chỉ số sau để đánh giá hiệu năng agent:

**1. Độ chính xác (Accuracy)**

Độ chính xác là chỉ số cốt lõi nhất, được định nghĩa là tỷ lệ mẫu khớp AST thành công:

$$
\text{Accuracy} = \frac{1}{N} \sum_{i=1}^{N} \text{AST\_Match}(P_i, G_i)
$$

Trong đó:
- $N$ là tổng số mẫu
- $P_i$ là kết quả dự đoán của mẫu thứ $i$
- $G_i$ là đáp án chuẩn của mẫu thứ $i$
- $\text{AST\_Match}(P_i, G_i) \in \{0, 1\}$ là hàm khớp AST

**2. Tỷ lệ Khớp AST (AST Match Rate)**

Giống với độ chính xác, nhấn mạnh việc sử dụng thuật toán khớp AST:

$$
\text{AST Match Rate} = \text{Accuracy}
$$

**3. Độ chính xác theo Loại (Category-wise Accuracy)**

Với mỗi loại $c \in \{\text{simple}, \text{multiple}, \text{parallel}, \ldots\}$, tính độ chính xác cho loại đó:

$$
\text{Accuracy}_c = \frac{1}{|D_c|} \sum_{i \in D_c} \text{AST\_Match}(P_i, G_i)
$$

Trong đó $D_c$ là tập mẫu của loại $c$, $|D_c|$ là số lượng mẫu trong loại đó.

**4. Độ chính xác có Trọng số (Weighted Accuracy)**

Cân nhắc trọng số độ khó của các loại khác nhau:

$$
\text{Weighted Accuracy} = \sum_{c} w_c \cdot \text{Accuracy}_c
$$

Trong đó $w_c$ là trọng số của loại $c$, thỏa mãn $\sum_c w_c = 1$.

**5. Tỷ lệ Lỗi (Error Rate)**

Tỷ lệ mẫu không gọi hàm chính xác:

$$
\text{Error Rate} = 1 - \text{Accuracy} = \frac{1}{N} \sum_{i=1}^{N} (1 - \text{AST\_Match}(P_i, G_i))
$$

**Diễn giải Chỉ số:**

- **Accuracy = 1.0**: Tất cả các mẫu đều hoàn toàn chính xác
- **Accuracy = 0.8**: 80% mẫu chính xác, 20% mẫu sai
- **Accuracy = 0.0**: Tất cả các mẫu đều sai

**Ví dụ Độ chính xác theo Loại:**

```python
# Assume evaluation results
simple_accuracy = 0.95      # Simple category: 95% correct
multiple_accuracy = 0.82    # Multiple category: 82% correct
parallel_accuracy = 0.68    # Parallel category: 68% correct

# Weighted accuracy (assuming equal weights)
weighted_accuracy = (0.95 + 0.82 + 0.68) / 3 = 0.817
```

**(4) Công cụ Đánh giá Chính thức của BFCL**

BFCL cung cấp công cụ CLI chính thức để đánh giá:

```bash
# Install BFCL evaluation tool
pip install bfcl

# Run official evaluation
bfcl evaluate \
    --model-result-path ./results.json \
    --test-category simple_python
```

Ưu điểm của việc sử dụng công cụ đánh giá chính thức: nó sử dụng thuật toán khớp AST chính thức, kết quả đánh giá hoàn toàn nhất quán với bảng xếp hạng (leaderboard), hỗ trợ tất cả các loại của BFCL v4, và có thể tự động sinh báo cáo đánh giá chi tiết.


### 12.2.2 Lấy Tập dữ liệu BFCL

Tập dữ liệu BFCL có thể được lấy thông qua các phương pháp sau:

**Phương pháp 1: Clone từ Kho GitHub Chính thức (Khuyến nghị)**

Đây là phương pháp đáng tin cậy nhất, thu được tập dữ liệu đầy đủ và ground truth:

```bash
# Clone BFCL repository
git clone https://github.com/ShishirPatil/gorilla.git temp_gorilla
cd temp_gorilla/berkeley-function-call-leaderboard

# View BFCL v4 dataset
ls bfcl_eval/data/
# Output: BFCL_v4_simple_python.json  BFCL_v4_multiple.json  BFCL_v4_parallel.json  ...

# View ground truth
ls bfcl_eval/data/possible_answer/
# Output: BFCL_v4_simple_python.json  BFCL_v4_multiple.json  ...
```

Lý do khuyến nghị phương pháp này: nó chứa ground truth (đáp án chuẩn) đầy đủ, định dạng dữ liệu hoàn toàn nhất quán với công cụ đánh giá chính thức, có thể trực tiếp sử dụng các script đánh giá chính thức, và hỗ trợ phiên bản mới nhất BFCL v4.

**Phương pháp 2: Nạp Dữ liệu Chính thức bằng HelloAgents**

Sau khi clone kho, nạp dữ liệu bằng HelloAgents:

```python
from hello_agents.evaluation import BFCLDataset

# Load BFCL official data
dataset = BFCLDataset(
    bfcl_data_dir="./temp_gorilla/berkeley-function-call-leaderboard/bfcl_eval/data",
    category="simple_python"  # BFCL v4 category
)

# Load data (including test data and ground truth)
data = dataset.load()

print(f"✅ Loaded {len(data)} test samples")
print(f"✅ Loaded {len(dataset.ground_truth)} ground truth")
# Output:
# ✅ Loaded 400 test samples
# ✅ Loaded 400 ground truth
```

Nguyên lý hoạt động của bộ nạp này là: đầu tiên nạp dữ liệu kiểm thử từ `bfcl_eval/data/`, sau đó nạp ground truth từ `bfcl_eval/data/possible_answer/`, tiếp theo tự động hợp nhất dữ liệu kiểm thử và ground truth, và cuối cùng giữ nguyên định dạng dữ liệu BFCL gốc. Các loại tập dữ liệu BFCL v4 có thể xem trong Bảng 12.2.

<div align="center">
  <p>Bảng 12.2 Bốn Loại Đánh giá trong Benchmark BFCL</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-table-2.png" alt="" width="85%"/>
</div>

Bạn cũng có thể xem các loại có sẵn thông qua code:

```python
# Get all supported categories
categories = dataset.get_available_categories()
print(f"Supported categories: {categories}")
# Output: ['simple_python', 'simple_java', 'simple_javascript', 'multiple', ...]
```

### 12.2.3 Triển khai Đánh giá BFCL trong HelloAgents

Bây giờ hãy cùng xem cách triển khai đánh giá BFCL trong framework HelloAgents. Chúng ta cung cấp ba phương pháp sử dụng:

**Phương pháp 1: Sử dụng BFCLEvaluationTool (Khuyến nghị)**

Đây là phương pháp đơn giản nhất, hoàn thành đánh giá, sinh báo cáo và đánh giá chính thức chỉ với một dòng code:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import BFCLEvaluationTool

# 1. Create agent to be evaluated
llm = HelloAgentsLLM()
agent = SimpleAgent(name="TestAgent", llm=llm)

# 2. Create BFCL evaluation tool
bfcl_tool = BFCLEvaluationTool()

# 3. Run evaluation (automatically complete all steps)
results = bfcl_tool.run(
    agent=agent,
    category="simple_python",  # Evaluation category
    max_samples=5              # Number of evaluation samples (0 means all)
)

# 4. View results
print(f"Accuracy: {results['overall_accuracy']:.2%}")
print(f"Correct: {results['correct_samples']}/{results['total_samples']}")
```

**Kết quả Chạy:**

```
============================================================
BFCL One-Click Evaluation
============================================================

Configuration:
   Evaluation category: simple_python
   Sample count: 5
   Agent: TestAgent

============================================================
Step 1: Run HelloAgents Evaluation
============================================================
✅ BFCL dataset loaded
   Data directory: ./temp_gorilla/berkeley-function-call-leaderboard/bfcl_eval/data
   Category: simple_python
   Sample count: 400
   Ground truth count: 400

🔧 Starting BFCL evaluation...
   Progress: 1/5
   Progress: 5/5

✅ BFCL evaluation complete
   Overall accuracy: 100.00%
   simple_python: 100.00% (5/5)

📊 Evaluation results:
   Accuracy: 100.00%
   Correct: 5/5

============================================================
Step 2: Export BFCL Format Results
============================================================
✅ BFCL format results exported
   Output file: ./evaluation_results/bfcl_official/BFCL_v4_simple_python_result.json

============================================================
Step 3: Run BFCL Official Evaluation
============================================================
✅ Result file copied to: ./result/Qwen_Qwen3-8B/BFCL_v4_simple_python_result.json

🔄 Running command: bfcl evaluate --model Qwen/Qwen3-8B --test-category simple_python --partial-eval

============================================================
BFCL Official Evaluation Results
============================================================
📊 Evaluation results summary:
Model,Overall Acc,simple_python
Qwen/Qwen3-8B,100.00,100.00

🎯 Final results:
   Accuracy: 100.00%
   Correct: 5/5

============================================================
Step 4: Generate Evaluation Report
============================================================
📄 Report generated: ./evaluation_reports/bfcl_report_20251011_005938.md

Accuracy: 100.00%
Correct: 5/5
```

**Báo cáo Markdown Sinh Tự động:**

Sau khi đánh giá hoàn tất, một báo cáo Markdown chi tiết sẽ được sinh tự động, bao gồm:

```markdown
# BFCL Evaluation Report
**Generated**: 2025-10-11 00:59:38

## 📊 Evaluation Overview

- **Agent**: TestAgent
- **Evaluation Category**: simple_python
- **Overall Accuracy**: 100.00%
- **Correct Samples**: 5/5

## 📈 Detailed Metrics

### Category Accuracy

- **simple_python**: 100.00% (5/5)

## 📝 Sample Details

| Sample ID | Question | Prediction | Ground Truth | Correct |
|-----------|----------|------------|--------------|---------|
| simple_python_0 | Find the area of a triangle... | [{'name': 'calculate_triangle_area'...}] | [{'function_name': {'base': [10]...}}] | ✅ |
| simple_python_1 | Calculate the factorial of 5... | [{'name': 'calculate_factorial'...}] | [{'function_name': {'number': [5]}}] | ✅ |
...

## 📊 Accuracy Visualization
Accuracy: ██████████████████████████████████████████████████ 100.00%

## 💡 Recommendations
- ✅ Excellent performance! Agent shows outstanding tool calling capabilities.
```

**Phương pháp 2: Sử dụng Script Đánh giá Một Nhấp (One-Click)**

Phù hợp cho việc đánh giá nhanh qua dòng lệnh. Trong các ví dụ code kèm theo chương này, chúng tôi cung cấp `04_run_bfcl_evaluation.py`, hỗ trợ đánh giá trực tiếp qua dòng lệnh:

```bash
# Run evaluation script
python chapter12/04_run_bfcl_evaluation.py --category simple_python --samples 10

# Specify model name (for BFCL official evaluation)
python examples/04_run_bfcl_evaluation.py \
    --category simple_python \
    --samples 10 \
    --model-name "Qwen/Qwen3-8B"
```

Script hỗ trợ ba tham số: `--category` chỉ định loại đánh giá (mặc định simple_python), `--samples` chỉ định số lượng mẫu đánh giá (mặc định 5, 0 nghĩa là tất cả), `--model-name` chỉ định tên mô hình cho đánh giá chính thức BFCL (mặc định Qwen/Qwen3-8B).

**Phương pháp 3: Sử dụng Trực tiếp Dataset và Evaluator**

Phù hợp cho các kịch bản đòi hỏi tùy chỉnh quy trình đánh giá:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.evaluation import BFCLDataset, BFCLEvaluator

# 1. Create agent
llm = HelloAgentsLLM()
agent = SimpleAgent(name="TestAgent", llm=llm)

# 2. Load dataset
dataset = BFCLDataset(
    bfcl_data_dir="./temp_gorilla/berkeley-function-call-leaderboard/bfcl_eval/data",
    category="simple_python"
)
data = dataset.load()

# 3. Create evaluator
evaluator = BFCLEvaluator(
    dataset=dataset,
    category="simple_python",
    evaluation_mode="ast"  # Use AST matching mode
)

# 4. Run evaluation
results = evaluator.evaluate(agent, max_samples=10)

# 5. View results
print(f"Accuracy: {results['overall_accuracy']:.2%}")
print(f"Correct: {results['correct_samples']}/{results['total_samples']}")

# 6. Export BFCL format results (optional)
evaluator.export_to_bfcl_format(
    results,
    output_path="./evaluation_results/my_results.json"
)
```

Thông qua ba phương pháp này, chúng ta có thể chọn phương pháp đánh giá phù hợp dựa trên nhu cầu khác nhau. Nếu bạn chỉ muốn nhanh chóng hiểu hiệu năng agent, sử dụng đánh giá một nhấp của BFCLEvaluationTool là tiện lợi nhất; nếu bạn cần đánh giá hàng loạt hoặc tích hợp vào pipeline CI/CD, sử dụng script dòng lệnh phù hợp hơn; nếu bạn cần tùy chỉnh sâu quy trình đánh giá hoặc tích hợp vào hệ thống của riêng mình, sử dụng trực tiếp Dataset và Evaluator mang lại độ linh hoạt tối đa.




### 12.2.4 Tích hợp Công cụ Đánh giá Chính thức của BFCL

Trước đây chúng ta đã học cách sử dụng chức năng đánh giá tích hợp sẵn của HelloAgents. Thực tế, `BFCLEvaluationTool` đã **tự động tích hợp công cụ đánh giá chính thức của BFCL**, cho phép bạn thu được kết quả đánh giá có tính thẩm quyền và so sánh được.

Toàn bộ quy trình đánh giá bao gồm bốn bước: đầu tiên nạp dữ liệu kiểm thử từ tập dữ liệu BFCL v4, sau đó dùng HelloAgents để chạy đánh giá và thu được kết quả dự đoán của agent, tiếp theo xuất kết quả sang định dạng chính thức của BFCL (JSONL), và cuối cùng dùng script đánh giá chính thức để tính điểm cuối cùng. Quy trình này đảm bảo kết quả đánh giá hoàn toàn nhất quán với bảng xếp hạng BFCL, như minh họa trong Hình 12.3:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-3.png" alt="" width="85%"/>
  <p>Hình 12.3 Quy trình Đánh giá BFCL do HelloAgents Nạp</p>
</div>

Khi sử dụng `BFCLEvaluationTool`, đánh giá chính thức sẽ **chạy tự động** (bật mặc định):

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import BFCLEvaluationTool

# Create agent
llm = HelloAgentsLLM()
agent = SimpleAgent(name="TestAgent", llm=llm)

# Create evaluation tool
bfcl_tool = BFCLEvaluationTool()

# Run evaluation (automatically runs official evaluation)
results = bfcl_tool.run(
    agent=agent,
    category="simple_python",
    max_samples=5,
    # run_official_eval=True  # Default is True, can be omitted
    model_name="Qwen/Qwen3-8B"  # Optional, specify model name
)
```

Công cụ tự động thực thi toàn bộ quy trình đánh giá: đầu tiên chạy đánh giá HelloAgents để thu được kết quả dự đoán, sau đó xuất kết quả sang định dạng BFCL và lưu vào thư mục `evaluation_results/bfcl_official/`, tiếp theo sao chép tệp kết quả sang thư mục `result/{model_name}/` để đáp ứng yêu cầu của công cụ đánh giá chính thức, rồi chạy lệnh đánh giá chính thức của BFCL để tính điểm, và cuối cùng hiển thị kết quả đánh giá chính thức cùng sinh báo cáo đánh giá định dạng Markdown.

**Ví dụ Đầu ra Đánh giá Chính thức:**

```
============================================================
Step 3: Run BFCL Official Evaluation
============================================================

✅ Result file copied to:
   ./result/Qwen_Qwen3-8B/BFCL_v4_simple_python_result.json

🔄 Running command: bfcl evaluate --model Qwen/Qwen3-8B --test-category simple_python --partial-eval

============================================================
BFCL Official Evaluation Results
============================================================

📊 Evaluation results summary:
Model,Overall Acc,simple_python
Qwen/Qwen3-8B,100.00,100.00

🎯 Final results:
   Accuracy: 100.00%
   Correct: 5/5
```

Nếu bạn muốn tự điều khiển quy trình đánh giá theo cách thủ công, bạn có thể tắt đánh giá chính thức tự động:

```python
# Disable official evaluation
results = bfcl_tool.run(
    agent=agent,
    category="simple_python",
    max_samples=5,
    run_official_eval=False  # Disable official evaluation
)

# Then manually run official evaluation
import subprocess
subprocess.run([
    "bfcl", "evaluate",
    "--model", "Qwen/Qwen3-8B",
    "--test-category", "simple_python",
    "--partial-eval"
])
```

Bạn cũng có thể sinh báo cáo theo cách thủ công:

```python
# Run evaluation
results = bfcl_tool.run(agent, category="simple_python", max_samples=5)

# Manually generate report
report = bfcl_tool.generate_report(
    results,
    output_file="./my_reports/custom_report.md"
)

# Print report content
print(report)
```



### 12.2.5 Chi tiết Triển khai các Thành phần Cốt lõi

Trong các phần trước, chúng ta đã học cách sử dụng các công cụ đánh giá BFCL. Bây giờ hãy đi sâu vào cách các thành phần cốt lõi của hệ thống đánh giá HelloAgents được triển khai. Việc hiểu các chi tiết triển khai này không chỉ giúp bạn sử dụng hệ thống đánh giá tốt hơn, mà còn cho phép bạn tùy chỉnh và mở rộng theo nhu cầu của riêng mình.

**(1) BFCLDataset: Bộ Nạp Tập dữ liệu**

BFCLDataset chịu trách nhiệm nạp và quản lý tập dữ liệu BFCL:

````python
class BFCLDataset:
    """BFCL dataset loader"""

    def __init__(self, category: str = "simple", local_data_path: Optional[str] = None):
        self.category = category
        self.local_data_path = local_data_path
        self.data = []

    def load(self) -> List[Dict[str, Any]]:
        """Load dataset"""
        # Load from local first
        if self.local_data_path:
            return self._load_from_local()
        # Otherwise load from Hugging Face
        return self._load_from_huggingface()
````

Vì tập dữ liệu của BFCL nằm trong kho chính thức, cách tiếp cận được khuyến nghị ở đây là trực tiếp clone một bản sao cục bộ để đánh giá. Chỉ khi không tìm thấy thì nó mới nạp từ Hugging Face.

**(2) BFCLEvaluator: Bộ Thực thi Đánh giá**

BFCLEvaluator chịu trách nhiệm thực thi quy trình đánh giá. Cốt lõi của nó là phương thức `evaluate()`, điều phối toàn bộ quy trình đánh giá:

````python
class BFCLEvaluator:
    """BFCL evaluator"""

    def evaluate(self, agent: Any, max_samples: Optional[int] = None) -> Dict[str, Any]:
        """Execute evaluation"""
        results = []

        for item in self.dataset[:max_samples]:
            # 1. Construct prompt
            prompt = self._build_prompt(item)

            # 2. Call agent
            response = agent.run(prompt)

            # 3. Extract function calls
            predicted_calls = self._extract_function_calls(response)

            # 4. Compare with ground truth
            is_correct = self._compare_calls(predicted_calls, item["ground_truth"])

            results.append({
                "id": item["id"],
                "prediction": predicted_calls,
                "ground_truth": item["ground_truth"],
                "is_correct": is_correct
            })

        return {"results": results, "total_samples": len(results)}
````

Thiết kế của bộ đánh giá này chứa ba điểm cốt lõi: đầu tiên là xây dựng prompt, cần chuyển đổi câu hỏi và định nghĩa hàm trong tập dữ liệu thành prompt mà agent có thể hiểu; thứ hai là trích xuất lời gọi hàm, cần trích xuất lời gọi hàm từ phản hồi của agent và hỗ trợ nhiều định dạng (JSON, khối code, v.v.); cuối cùng là khớp AST, sử dụng cây cú pháp trừu tượng để so sánh lời gọi hàm, chính xác hơn so với khớp chuỗi đơn giản.

Hãy cùng xem cách triển khai trích xuất lời gọi hàm:

```python
def _extract_function_calls(self, response: str) -> List[Dict[str, Any]]:
    """Extract function calls from response

    Supports multiple formats:
    1. JSON format: {"name": "func", "arguments": {...}}
    2. Code block format: ```python\nfunc(arg1=val1)\n```
    3. Plain text format: func(arg1=val1)
    """
    calls = []

    # Try JSON parsing
    try:
        json_match = re.search(r'\{.*\}', response, re.DOTALL)
        if json_match:
            data = json.loads(json_match.group())
            if isinstance(data, dict) and "name" in data:
                calls.append(data)
            elif isinstance(data, list):
                calls.extend(data)
    except json.JSONDecodeError:
        pass

    # Try code block extraction
    code_blocks = re.findall(r'```(?:python)?\n(.*?)\n```', response, re.DOTALL)
    for code in code_blocks:
        # Parse Python function calls
        parsed_calls = self._parse_python_calls(code)
        calls.extend(parsed_calls)

    return calls
```

**(3) BFCLMetrics: Bộ Tính Chỉ số**

BFCLMetrics chịu trách nhiệm tính toán các chỉ số đánh giá khác nhau:

````python
class BFCLMetrics:
    """BFCL metrics calculator"""

    def compute_metrics(self, results: List[Dict[str, Any]]) -> Dict[str, Any]:
        """Compute all metrics"""
        return {
            "accuracy": self._compute_accuracy(results),
            "ast_match_rate": self._compute_ast_match_rate(results),
            "parameter_accuracy": self._compute_parameter_accuracy(results),
            "f1_score": self._compute_f1_score(results),
            "category_statistics": self._compute_category_stats(results)
        }
````

**Triển khai Khớp AST**:

Khớp AST là công nghệ cốt lõi của đánh giá BFCL. Nó thông minh hơn khớp chuỗi đơn giản và có thể nhận diện các lời gọi hàm tương đương về ngữ nghĩa:

```python
def _ast_match(self, pred_call: Dict, true_call: Dict) -> bool:
    """Match function calls using AST

    Advantages of AST matching:
    1. Ignore parameter order: func(a=1, b=2) equivalent to func(b=2, a=1)
    2. Recognize equivalent expressions: 2+3 equivalent to 5
    3. Ignore whitespace and format differences
    """
    # 1. Function name must match exactly
    if pred_call.get("name") != true_call.get("name"):
        return False

    # 2. Convert parameters to AST nodes
    pred_args = self._args_to_ast(pred_call.get("arguments", {}))
    true_args = self._args_to_ast(true_call.get("arguments", {}))

    # 3. Compare AST nodes
    return ast.dump(pred_args) == ast.dump(true_args)

def _args_to_ast(self, args: Dict[str, Any]) -> ast.AST:
    """Convert parameter dictionary to AST node"""
    # Construct a virtual function call
    code = f"func({', '.join(f'{k}={repr(v)}' for k, v in args.items())})"
    tree = ast.parse(code)
    return tree.body[0].value  # Return Call node
```

**(4) Đóng gói Công cụ: BFCLEvaluationTool**

Cuối cùng, chúng ta đóng gói các thành phần này thành một Tool để nó có thể được agent gọi trực tiếp:

````python
class BFCLEvaluationTool(Tool):
    """BFCL evaluation tool"""

    def __init__(self, local_data_path: Optional[str] = None):
        super().__init__(
            name="bfcl_evaluation",
            description="Evaluate agent's tool calling capability"
        )
        self.dataset = None
        self.evaluator = None
        self.metrics_calculator = BFCLMetrics()

    def run(self, parameters: Dict[str, Any]) -> str:
        """Execute evaluation"""
        # 1. Load dataset
        self.dataset = BFCLDataset(...)

        # 2. Create evaluator
        self.evaluator = BFCLEvaluator(...)

        # 3. Run evaluation
        results = self.evaluator.evaluate(...)

        # 4. Calculate metrics
        metrics = self.metrics_calculator.compute_metrics(...)

        # 5. Return JSON results
        return json.dumps(results, ensure_ascii=False)
````

Thiết kế của công cụ này tuân theo ba nguyên tắc cốt lõi: đầu tiên kế thừa lớp cơ sở Tool để tuân theo đặc tả công cụ của HelloAgents, đảm bảo tích hợp liền mạch với framework; thứ hai thực hiện kiểm tra tham số nghiêm ngặt, kiểm tra các tham số bắt buộc và cung cấp thông báo lỗi thân thiện, cải thiện trải nghiệm người dùng; cuối cùng định dạng kết quả, trả về chuỗi JSON để dễ dàng phân tích và hiển thị. Thông qua thiết kế mô-đun này, chúng ta đã triển khai một hệ thống đánh giá vừa dễ sử dụng vừa linh hoạt. Người dùng có thể trực tiếp sử dụng giao diện Tool cấp cao để nhanh chóng hoàn thành đánh giá, hoặc đi sâu vào các thành phần cấp thấp để tùy chỉnh đáp ứng nhu cầu đặc biệt.

### 12.2.6 Khuyến nghị Mở rộng và Tối ưu hóa

Qua việc học các phần trước, chúng ta đã nắm vững cách sử dụng HelloAgents để đánh giá BFCL. Cần lưu ý rằng phần triển khai hiện tại của chúng ta là một bản tái hiện đơn giản dựa trên SimpleAgent, chủ yếu hoàn thành chức năng đánh giá BFCL cơ bản. Trong ứng dụng thực tế, benchmark BFCL chứa nhiều cấp độ khó và kịch bản khác nhau. Để đạt điểm cao hơn trên bảng xếp hạng, cần tối ưu hóa và mở rộng thêm.

**(1) Giới hạn của Triển khai Hiện tại**

Triển khai SimpleAgent hiện tại của chúng ta chủ yếu tập trung vào việc xây dựng quy trình đánh giá, còn nhiều dư địa cải thiện về năng lực gọi công cụ. SimpleAgent sử dụng định dạng gọi công cụ tùy chỉnh `[TOOL_CALL:tool_name:parameters]`, đòi hỏi LLM phải chủ động học và sử dụng. Trong các kịch bản phức tạp, hiệu năng có thể không sánh bằng các agent sử dụng function calling gốc (native). Ngoài ra, hiện tại chúng ta chỉ kiểm thử các loại cơ bản như simple_python. Đối với các kịch bản phức tạp hơn như multiple, parallel, irrelevance, vẫn cần tối ưu hóa có mục tiêu.

**(2) Các Hướng Cải thiện Điểm số BFCL**

Để cải thiện thêm điểm số đánh giá BFCL, bạn có thể bắt đầu từ các hướng sau. Đầu tiên là tối ưu hóa năng lực gọi công cụ của agent - cân nhắc sử dụng các LLM hỗ trợ function calling gốc (như GPT-4, Claude, v.v.), hoặc cải thiện prompt để giúp LLM hiểu tốt hơn định dạng gọi công cụ. Thứ hai là mở rộng thư viện công cụ - các bài kiểm thử BFCL liên quan đến nhiều loại hàm khác nhau, bạn có thể triển khai trước các loại công cụ phổ biến dựa trên đặc điểm tập dữ liệu kiểm thử để cải thiện độ bao phủ công cụ của agent. Thứ ba là thiết kế các chiến lược khác nhau cho các cấp độ khó khác nhau - ví dụ, trong kịch bản multiple agent cần lập kế hoạch chuỗi gọi công cụ nhiều bước, trong kịch bản parallel chúng cần nhận diện các lời gọi công cụ có thể thực thi song song, trong kịch bản irrelevance chúng cần phán đoán liệu có thực sự cần gọi công cụ hay không.

**(3) Khuyến nghị Thực hành**

Đối với các nhà phát triển muốn đạt kết quả tốt hơn trên BFCL, các chiến lược thực hành sau được khuyến nghị. Đầu tiên, bắt đầu từ loại simple, đảm bảo lời gọi hàm đơn cơ bản hoạt động ổn định - đây là nền tảng cho việc tối ưu hóa tiếp theo. Sau đó, dần dần kiểm thử các loại phức tạp hơn như multiple, parallel, phân tích các trường hợp thất bại, tìm ra điểm yếu của agent. Trong quá trình tối ưu hóa, bạn có thể tham khảo các mô hình đạt điểm cao trên bảng xếp hạng BFCL, học hỏi ý tưởng thiết kế và kỹ thuật tối ưu hóa của họ. Đồng thời, khuyến nghị sử dụng các công cụ đánh giá chính thức để xác thực, đảm bảo kết quả sau tối ưu hóa nhất quán với tiêu chuẩn của bảng xếp hạng.

Dưới đây là một số gợi ý để xử lý thêm trong quá trình đánh giá:

**1. Đánh giá Tiệm tiến (Progressive Evaluation)**

Bắt đầu từ mẫu nhỏ, tăng dần số lượng mẫu:

```python
# Step 1: Quick test (5 samples)
results_quick = bfcl_tool.run(agent, category="simple_python", max_samples=5)

# Step 2: Medium-scale test (50 samples)
if results_quick['overall_accuracy'] > 0.8:
    results_medium = bfcl_tool.run(agent, category="simple_python", max_samples=50)

# Step 3: Full evaluation (all samples)
if results_medium['overall_accuracy'] > 0.8:
    results_full = bfcl_tool.run(agent, category="simple_python", max_samples=0)
```

**2. Đánh giá Đa Loại (Multi-Category Evaluation)**

Đánh giá các nhiệm vụ với độ khó khác nhau:

```python
categories = ["simple_python", "multiple", "parallel", "irrelevance"]

for category in categories:
    print(f"\nEvaluating category: {category}")
    results = bfcl_tool.run(agent, category=category, max_samples=10)
    print(f"Accuracy: {results['overall_accuracy']:.2%}")
```

**3. Đánh giá So sánh (Comparative Evaluation)**

So sánh các agent với cấu hình khác nhau:

```python
# Configuration 1: Default prompt
agent1 = SimpleAgent(name="Agent-Default", llm=llm)
results1 = bfcl_tool.run(agent1, category="simple_python", max_samples=10)

# Configuration 2: Optimized prompt
agent2 = SimpleAgent(name="Agent-Optimized", llm=llm)
# ... Set optimized system prompt ...
results2 = bfcl_tool.run(agent2, category="simple_python", max_samples=10)

# Compare results
print(f"Default configuration accuracy: {results1['overall_accuracy']:.2%}")
print(f"Optimized configuration accuracy: {results2['overall_accuracy']:.2%}")
```

Nếu kết quả đánh giá của bạn tốt, hãy cân nhắc gửi lên bảng xếp hạng chính thức của BFCL!

**Bước 1: Chuẩn bị Tài liệu Nộp**

1. Tài liệu mô tả mô hình
2. Các tệp kết quả đánh giá (tất cả các loại)
3. Phương thức truy cập mô hình (API hoặc liên kết mã nguồn mở)

**Bước 2: Nộp lên GitHub**

Truy cập kho chính thức của BFCL và gửi Pull Request theo hướng dẫn:

- Kho: https://github.com/ShishirPatil/gorilla
- Hướng dẫn nộp: Tham khảo `CONTRIBUTING.md`

**Bước 3: Chờ Duyệt**

Đội ngũ BFCL sẽ duyệt bản nộp của bạn và xác minh độ chính xác của kết quả. Sau khi được phê duyệt, mô hình của bạn sẽ xuất hiện trên bảng xếp hạng chính thức!



## 12.3 GAIA: Đánh giá Năng lực Trợ lý AI Tổng quát

### 12.3.1 Giới thiệu Benchmark GAIA

GAIA (General AI Assistants) là một benchmark đánh giá do Meta AI và Hugging Face đồng phát hành, tập trung đánh giá **năng lực tổng quát** của các trợ lý AI<sup>[2]</sup>. Khác với BFCL tập trung vào gọi công cụ, GAIA đánh giá hiệu năng tổng hợp của agent trong các nhiệm vụ thế giới thực.

Triết lý thiết kế của GAIA là: **Các bài toán thế giới thực thường đòi hỏi ứng dụng tổng hợp nhiều năng lực khác nhau**. Một trợ lý AI xuất sắc không chỉ cần gọi công cụ, mà còn cần:

- **Suy luận Nhiều bước (Multi-step Reasoning)**: Phân rã bài toán phức tạp thành nhiều bài toán con
- **Ứng dụng Kiến thức (Knowledge Application)**: Tận dụng kiến thức có sẵn và cơ sở kiến thức bên ngoài
- **Hiểu Đa phương thức (Multimodal Understanding)**: Xử lý nhiều đầu vào như văn bản, hình ảnh, tệp
- **Duyệt Web (Web Browsing)**: Lấy thông tin mới nhất từ internet
- **Thao tác Tệp (File Operations)**: Đọc và xử lý các tệp với nhiều định dạng khác nhau

**(1) Cấu trúc Tập dữ liệu GAIA**

Sau khi hiểu triết lý đánh giá của GAIA, hãy đi sâu vào cấu trúc cụ thể của tập dữ liệu GAIA. GAIA chứa 466 bài toán thế giới thực được thiết kế kỹ lưỡng. Các bài toán này được chia thành ba cấp độ khó dựa trên độ phức tạp và số bước suy luận cần thiết, từ các nhiệm vụ suy luận không bước đơn giản đến các nhiệm vụ khó đòi hỏi suy luận phức tạp nhiều bước, bao phủ toàn diện các kịch bản đa dạng mà agent có thể gặp trong ứng dụng thực tế, như minh họa trong Bảng 12.3:

<div align="center">
  <p>Bảng 12.3 Phân bố Cấp độ Khó của Tập dữ liệu GAIA</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-table-3.png" alt="" width="85%"/>
</div>

Đối với các ví dụ mẫu của tập dữ liệu GAIA, tham khảo đoạn code dưới đây:

```json
{
  "task_id": "gaia_001",
  "Question": "What is the total population of the top 3 most populous cities in California?",
  "Level": 2,
  "Final answer": "12847521",
  "file_name": "",
  "file_path": "",
  "Annotator Metadata": {
    "Steps": [
      "Search for most populous cities in California",
      "Get population data for top 3 cities",
      "Sum the populations"
    ],
    "Number of steps": 3,
    "How long did this take?": "5 minutes",
    "Tools": ["web_search", "calculator"]
  }
}
```

**Mô tả các Trường Then chốt:**
- `Question`: Mô tả câu hỏi
- `Level`: Cấp độ khó (1-3)
- `Final answer`: Đáp án chuẩn (có thể là số, văn bản, hoặc tệp)
- `file_name/file_path`: Tệp đính kèm (nếu có)
- `Annotator Metadata`: Metadata do người chú thích cung cấp (các bước suy luận, công cụ cần thiết, v.v.)

**(2) Giới thiệu Quasi Exact Match**

GAIA sử dụng thuật toán đánh giá **Quasi Exact Match** (Khớp Gần-Chính-xác), đây là tiêu chuẩn đánh giá do GAIA định nghĩa chính thức. Ý tưởng cốt lõi của thuật toán này là: **Trước tiên chuẩn hóa câu trả lời, sau đó thực hiện khớp chính xác**.

Cho câu trả lời dự đoán $A_{\text{pred}}$ và đáp án chuẩn $A_{\text{true}}$, hàm quasi exact match được định nghĩa như sau:

$$
\text{Quasi\_Exact\_Match}(A_{\text{pred}}, A_{\text{true}}) = \begin{cases}
1 & \text{if } \mathcal{N}(A_{\text{pred}}) = \mathcal{N}(A_{\text{true}}) \\
0 & \text{otherwise}
\end{cases}
$$

Trong đó $\mathcal{N}(\cdot)$ là hàm chuẩn hóa, áp dụng các quy tắc khác nhau dựa trên loại câu trả lời.

Hàm chuẩn hóa áp dụng các quy tắc khác nhau dựa trên loại câu trả lời. Đối với kiểu số, loại bỏ dấu phân cách phẩy (`1,000` → `1000`) và ký hiệu đơn vị (`$100` → `100`, `50%` → `50`), ví dụ `"$1,234.56"` chuẩn hóa thành `"1234.56"`. Đối với kiểu chuỗi, chuyển thành chữ thường (`"Apple"` → `"apple"`), loại bỏ mạo từ (`"the apple"` → `"apple"`), loại bỏ khoảng trắng thừa (`"hello  world"` → `"hello world"`) và loại bỏ dấu câu ở cuối (`"hello."` → `"hello"`), ví dụ `"The United States"` chuẩn hóa thành `"united states"`. Đối với kiểu danh sách, tách các phần tử theo dấu phẩy, áp dụng chuẩn hóa chuỗi cho từng phần tử, sắp xếp theo bảng chữ cái rồi ghép lại, ví dụ `"Paris, London, Berlin"` chuẩn hóa thành `"berlin,london,paris"`.

**Ví dụ Chuẩn hóa:**

```python
# Numeric answer
Original answer: "$1,234.56"
Normalized: "1234.56"

# String answer
Original answer: "The United States of America"
Normalized: "united states of america"

# List answer
Original answer: "Paris, London, Berlin"
Normalized: "berlin, london, paris"
```

**(3) Các Chỉ số Đánh giá GAIA**

GAIA sử dụng các chỉ số sau để đánh giá hiệu năng agent:

**1. Tỷ lệ Khớp Chính xác (Exact Match Rate)**

Tỷ lệ khớp chính xác là chỉ số cốt lõi của GAIA, được định nghĩa là tỷ lệ mẫu quasi exact match thành công:

$$
\text{Exact Match Rate} = \frac{1}{N} \sum_{i=1}^{N} \text{Quasi\_Exact\_Match}(A_{\text{pred},i}, A_{\text{true},i})
$$

Trong đó:
- $N$ là tổng số mẫu
- $A_{\text{pred},i}$ là câu trả lời dự đoán của mẫu thứ $i$
- $A_{\text{true},i}$ là đáp án chuẩn của mẫu thứ $i$
- $\text{Quasi\_Exact\_Match}(\cdot, \cdot) \in \{0, 1\}$ là hàm quasi exact match

**2. Độ chính xác theo Cấp độ (Level-wise Accuracy)**

Với mỗi cấp độ khó $\ell \in \{1, 2, 3\}$, tính độ chính xác cho cấp độ đó:

$$
\text{Accuracy}_\ell = \frac{1}{|D_\ell|} \sum_{i \in D_\ell} \text{Quasi\_Exact\_Match}(A_{\text{pred},i}, A_{\text{true},i})
$$

Trong đó $D_\ell$ là tập mẫu của cấp độ khó $\ell$, $|D_\ell|$ là số lượng mẫu ở cấp độ đó.

**3. Tỷ lệ Suy giảm theo Độ khó (Difficulty Progression Drop Rate)**

Đo mức suy giảm hiệu năng của agent khi độ khó tăng lên:

$$
\text{Drop Rate}_{\ell \to \ell+1} = \frac{\text{Accuracy}_\ell - \text{Accuracy}_{\ell+1}}{\text{Accuracy}_\ell}
$$

- $\text{Drop Rate}_{1 \to 2}$: Tỷ lệ suy giảm từ Level 1 đến Level 2
- $\text{Drop Rate}_{2 \to 3}$: Tỷ lệ suy giảm từ Level 2 đến Level 3

**4. Số bước Suy luận Trung bình (Average Reasoning Steps)**

Đánh giá số bước trung bình mà agent cần để hoàn thành nhiệm vụ:

$$
\text{Avg Steps} = \frac{1}{N_{\text{correct}}} \sum_{i \in \text{Correct}} \text{steps}_i
$$

Trong đó $N_{\text{correct}}$ là số lượng mẫu trả lời đúng, $\text{steps}_i$ là số bước suy luận của mẫu thứ $i$.

**Diễn giải Chỉ số:**

- **Exact Match Rate = 1.0**: Tất cả các mẫu đều hoàn toàn chính xác
- **Exact Match Rate = 0.5**: 50% mẫu chính xác, 50% mẫu sai
- **Drop Rate = 0.3**: Độ khó tăng khiến độ chính xác giảm 30%
- **Drop Rate = 0.0**: Độ khó tăng không ảnh hưởng đến độ chính xác (trường hợp lý tưởng)

**Ví dụ Đánh giá:**

Giả sử chúng ta đánh giá 10 mẫu, kết quả có thể tham khảo trong Bảng 12.4:

<div align="center">
  <p>Bảng 12.4 Phân bố Cấp độ Khó của Tập dữ liệu GAIA</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-table-4.png" alt="" width="85%"/>
</div>

Để tính các chỉ số cho trường hợp này, tham khảo script Python dưới đây:

```python
# 1. Exact match rate
total_samples = 10
correct_samples = 7  # Samples 1,2,3,5,6,8,9
exact_match_rate = correct_samples / total_samples = 0.70  # 70%

# 2. Level-wise accuracy
level_1_correct = 3  # Samples 1,2,3
level_1_total = 3
level_1_accuracy = 3 / 3 = 1.00  # 100%

level_2_correct = 2  # Samples 5,6
level_2_total = 3
level_2_accuracy = 2 / 3 = 0.67  # 67%

level_3_correct = 2  # Samples 8,9
level_3_total = 4
level_3_accuracy = 2 / 4 = 0.50  # 50%

# 3. Difficulty progression drop rate
drop_rate_1_to_2 = (1.00 - 0.67) / 1.00 = 0.33  # 33%
drop_rate_2_to_3 = (0.67 - 0.50) / 0.67 = 0.25  # 25%

print(f"Exact match rate: {exact_match_rate:.2%}")  # 70.00%
print(f"Level 1 accuracy: {level_1_accuracy:.2%}")  # 100.00%
print(f"Level 2 accuracy: {level_2_accuracy:.2%}")  # 66.67%
print(f"Level 3 accuracy: {level_3_accuracy:.2%}")  # 50.00%
print(f"Level 1→2 drop rate: {drop_rate_1_to_2:.2%}")  # 33.00%
print(f"Level 2→3 drop rate: {drop_rate_2_to_3:.2%}")  # 25.00%
```

**Phân tích Kết quả:**

- **Hiệu năng Tổng thể**: Tỷ lệ khớp chính xác 70%, hiệu năng tốt
- **Độ nhạy với Độ khó**: Giảm 33% từ Level 1 đến Level 2, cho thấy sự suy giảm đáng kể ở các nhiệm vụ độ khó trung bình
- **Biên Năng lực**: Độ chính xác Level 3 là 50%, cho thấy còn dư địa cải thiện ở các nhiệm vụ phức tạp

Tỷ lệ suy giảm càng lớn, sự suy giảm năng lực của agent khi xử lý các nhiệm vụ phức tạp càng rõ rệt.

**(4) System Prompt Chính thức của GAIA**

GAIA yêu cầu sử dụng một system prompt cụ thể để đảm bảo đầu ra của mô hình tuân theo định dạng đánh giá:

```python
GAIA_SYSTEM_PROMPT = """You are a general AI assistant. I will ask you a question. Report your thoughts, and finish your answer with the following template: FINAL ANSWER: [YOUR FINAL ANSWER].

YOUR FINAL ANSWER should be a number OR as few words as possible OR a comma separated list of numbers and/or strings.

If you are asked for a number, don't use comma to write your number neither use units such as $ or percent sign unless specified otherwise.

If you are asked for a string, don't use articles, neither abbreviations (e.g. for cities), and write the digits in plain text unless specified otherwise.

If you are asked for a comma separated list, apply the above rules depending of whether the element to be put in the list is a number or a string."""
```

GAIA có yêu cầu nghiêm ngặt về định dạng câu trả lời: câu trả lời phải được đưa ra theo định dạng `FINAL ANSWER: [answer]`; đối với câu trả lời kiểu số, không dùng dấu phân cách phẩy và ký hiệu đơn vị; đối với câu trả lời kiểu chuỗi, không dùng mạo từ và chữ viết tắt; đối với câu trả lời kiểu danh sách, dùng dấu phẩy phân cách và sắp xếp theo bảng chữ cái.

### 12.3.2 Lấy Tập dữ liệu GAIA

**Lưu ý Quan trọng**: GAIA là một **Tập dữ liệu Có kiểm soát (Gated Dataset)**, đòi hỏi phải đăng ký quyền truy cập trước trên HuggingFace.

**Bước 1: Đăng ký Quyền Truy cập**

1. Truy cập https://huggingface.co/datasets/gaia-benchmark/GAIA
2. Nhấn nút "Request access"
3. Điền biểu mẫu đăng ký (thường được phê duyệt trong vài giây)
4. Lấy HuggingFace Token của bạn: https://huggingface.co/settings/tokens

**Bước 2: Cấu hình Biến Môi trường**

Thêm HuggingFace Token của bạn vào tệp `.env`:

```bash
# HuggingFace API configuration
HF_TOKEN=hf_your_token_here
```

**Phương pháp 1: Tự động Tải xuống bằng HelloAgents (Khuyến nghị)**

HelloAgents tự động xử lý việc tải xuống và lưu cache tập dữ liệu GAIA:

```python
from hello_agents.evaluation import GAIADataset
import os

# Ensure HF_TOKEN is set, this line is not needed if .env is configured
os.environ["HF_TOKEN"] = "hf_your_token_here"

# Automatically download to ./data/gaia/
dataset = GAIADataset(
    dataset_name="gaia-benchmark/GAIA",
    split="validation",  # or "test"
    level=1  # Optional: 1, 2, 3, None(all)
)
items = dataset.load()

print(f"Loaded {len(items)} test samples")
# Output: Loaded 53 test samples (Level 1)
```

**Nguyên lý Hoạt động**:

- Ở lần chạy đầu tiên, dùng `snapshot_download` để tải toàn bộ tập dữ liệu về `./data/gaia/`
- Tập dữ liệu chứa 114 tệp (câu hỏi, hình ảnh, PDF, v.v.)
- Các lần sử dụng sau nạp trực tiếp từ cục bộ, rất nhanh

**Cấu trúc Thư mục Tập dữ liệu**:
```
./data/gaia/
├── 2023/
│   ├── validation/
│   │   ├── metadata.jsonl  (165 questions)
│   │   ├── *.png, *.pdf, *.csv, *.xlsx  (attachment files)
│   └── test/
│       ├── metadata.jsonl  (301 questions)
│       └── ... (attachment files)
├── GAIA.py
└── README.md
```

**Phương pháp 2: Tải xuống Thủ công**

Nếu bạn muốn tải xuống tập dữ liệu theo cách thủ công:

```python
from huggingface_hub import snapshot_download
import os

# Set Token
os.environ["HF_TOKEN"] = "hf_your_token_here"

# Download dataset
snapshot_download(
    repo_id="gaia-benchmark/GAIA",
    repo_type="dataset",
    local_dir="./data/gaia",
    token=os.getenv("HF_TOKEN")
)
```

**Xem Thống kê Tập dữ liệu**:

```python
# View dataset statistics
stats = dataset.get_statistics()
print(f"Total samples: {stats['total_samples']}")
print(f"Level distribution: {stats['level_distribution']}")
# Output:
# Total samples: 165
# Level distribution: {1: 53, 2: 62, 3: 50}
```


### 12.3.3 Triển khai Đánh giá GAIA trong HelloAgents

Tương tự BFCL, chúng ta cung cấp hai phương pháp đánh giá, **Phương pháp 1** được khuyến nghị.

**Phương pháp 1: Đánh giá Một Nhấp bằng GAIAEvaluationTool**

Đây là phương pháp đơn giản nhất, tự động hoàn thành việc tải xuống tập dữ liệu, thực thi đánh giá, xuất kết quả và sinh báo cáo:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import GAIAEvaluationTool

# GAIA official system prompt (from paper)
GAIA_SYSTEM_PROMPT = """You are a general AI assistant. I will ask you a question. Report your thoughts, and finish your answer with the following template: FINAL ANSWER: [YOUR FINAL ANSWER].

YOUR FINAL ANSWER should be a number OR as few words as possible OR a comma separated list of numbers and/or strings.

If you are asked for a number, don't use comma to write your number neither use units such as $ or percent sign unless specified otherwise.

If you are asked for a string, don't use articles, neither abbreviations (e.g. for cities), and write the digits in plain text unless specified otherwise.

If you are asked for a comma separated list, apply the above rules depending of whether the element to be put in the list is a number or a string."""

# 1. Create agent (using GAIA official system prompt)
llm = HelloAgentsLLM()
agent = SimpleAgent(
    name="TestAgent",
    llm=llm,
    system_prompt=GAIA_SYSTEM_PROMPT  # Key: Use GAIA official prompt
)

# 2. Create GAIA evaluation tool
gaia_tool = GAIAEvaluationTool()

# 3. One-click run evaluation
results = gaia_tool.run(
    agent=agent,
    level=1,  # Level 1: Simple tasks
    max_samples=5,  # Evaluate 5 samples
    export_results=True,  # Export GAIA format results
    generate_report=True  # Generate evaluation report
)

# 4. View results
print(f"Exact match rate: {results['exact_match_rate']:.2%}")
print(f"Partial match rate: {results['partial_match_rate']:.2%}")
print(f"Correct: {results['exact_matches']}/{results['total_samples']}")
```

**Kết quả Chạy:**

```
============================================================
GAIA One-Click Evaluation
============================================================

Configuration:
   Agent: TestAgent
   Difficulty level: 1
   Sample count: 5

============================================================
Step 1: Run HelloAgents Evaluation
============================================================
   Downloading from HuggingFace: gaia-benchmark/GAIA
   📥 Downloading GAIA dataset...
   ✓ Dataset download complete
   ✓ Loaded 165 samples
✅ GAIA dataset loaded
   Data source: gaia-benchmark/GAIA
   Split: validation
   Level: 1
   Sample count: 53

🌟 Starting GAIA evaluation...
   Sample count: 5
   Progress: 5/5
✅ GAIA evaluation complete
   Exact match rate: 80.00%
   Partial match rate: 80.00%

============================================================
Step 2: Export GAIA Format Results
============================================================
✅ GAIA format results exported
   Output file: evaluation_results\gaia_official\gaia_level1_result_20251011_012648.jsonl
   Sample count: 5
   Includes reasoning trace: True
📄 Submission guide generated: evaluation_results\gaia_official\SUBMISSION_GUIDE_20251011_012648.md

============================================================
Step 3: Generate Evaluation Report
============================================================
📄 Report generated: evaluation_reports\gaia_report_20251011_012648.md

============================================================
🎯 Final Results
============================================================
   Exact match rate: 80.00%
   Partial match rate: 80.00%
   Correct: 4/5
```

Sau khi đánh giá hoàn tất, ba loại tệp được sinh tự động: đầu tiên là tệp kết quả định dạng GAIA (`evaluation_results/gaia_official/gaia_level1_result_*.jsonl`), sử dụng định dạng JSONL (mỗi dòng một đối tượng JSON), có thể dùng trực tiếp để nộp lên bảng xếp hạng GAIA; thứ hai là tệp hướng dẫn nộp (`evaluation_results/gaia_official/SUBMISSION_GUIDE_*.md`), chứa các bước nộp chi tiết, mô tả định dạng tệp kết quả và lưu ý; cuối cùng là báo cáo đánh giá (`evaluation_reports/gaia_report_*.md`), chứa tóm tắt kết quả đánh giá, các chỉ số chi tiết, chi tiết mẫu và biểu đồ trực quan hóa.

**Lưu ý**: Nếu bạn thấy kết quả đánh giá được sinh ra chưa như ý (ví dụ độ chính xác thấp), điều này là bình thường. Mặc dù Level 1 là các nhiệm vụ suy luận một bước, agent vẫn cần năng lực gọi công cụ (như công cụ tìm kiếm, máy tính, v.v.) để trả lời chính xác các câu hỏi. SimpleAgent hiện tại của chúng ta chủ yếu dùng để minh họa quy trình đánh giá, còn nhiều dư địa cải thiện về năng lực gọi công cụ.

**Phương pháp 2: Sử dụng Dataset + Evaluator (Tùy chỉnh Linh hoạt)**

Nếu bạn cần điều khiển tinh vi hơn, bạn có thể trực tiếp sử dụng các thành phần cấp thấp:

```python
from hello_agents.evaluation import GAIADataset, GAIAEvaluator

# 1. Load dataset
dataset = GAIADataset(level=1)
items = dataset.load()
print(f"Loaded {len(items)} samples")

# 2. Create evaluator
evaluator = GAIAEvaluator(dataset=dataset, level=1)

# 3. Run evaluation
results = evaluator.evaluate(agent, max_samples=5)

# 4. Export GAIA format results
evaluator.export_to_gaia_format(
    results,
    "gaia_results.jsonl",
    include_reasoning=True
)
```

Báo cáo đánh giá được sinh ra (`gaia_report_*.md`) có thể tham khảo tệp dưới đây:

```markdown
# GAIA Evaluation Report

**Generated**: 2025-10-11 01:26:48

## 📊 Evaluation Overview

- **Agent**: TestAgent
- **Difficulty Level**: 1
- **Total Samples**: 2
- **Exact Matches**: 1
- **Partial Matches**: 1
- **Exact Match Rate**: 50.00%
- **Partial Match Rate**: 50.00%

## 📈 Detailed Metrics

### Level-wise Accuracy

- **Level 1**: 50.00% exact / 50.00% partial (1/2)

## 📝 Sample Details (First 10)

| Task ID | Level | Predicted Answer | Correct Answer | Exact Match | Partial Match |
|---------|-------|------------------|----------------|-------------|---------------|
| e1fc63a2-da7a-432f-be78-7c4a95598703 | 1 | 24000 | 17 | ❌ | ❌ |
| 8e867cd7-cff9-4e6c-867a-ff5ddc2550be | 1 | 3 | 3 | ✅ | ✅ |

## 📊 Accuracy Visualization

Exact match: █████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░ 50.00%
Partial match: █████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░ 50.00%


## 💡 Recommendations

- ⚠️ Average performance, needs improvement.
- 💡 Suggest checking tool usage and multi-step reasoning capabilities.
```

**Kết quả Định dạng GAIA Được sinh ra (`gaia_level1_result_*.jsonl`):**

```json
{"task_id": "e1fc63a2-da7a-432f-be78-7c4a95598703", "model_answer": "24000", "reasoning_trace": "24000"}
{"task_id": "8e867cd7-cff9-4e6c-867a-ff5ddc2550be", "model_answer": "3", "reasoning_trace": "3"}
```

### 12.3.4 Nộp Kết quả lên Bảng Xếp hạng Chính thức của GAIA

Sau khi chạy đánh giá bằng GAIAEvaluationTool, các tệp cần thiết cho việc nộp và hướng dẫn nộp chi tiết được sinh ra trong thư mục `evaluation_results/gaia_official/`.

1. **Tệp Kết quả Định dạng GAIA**: `gaia_level1_result_*.jsonl`
   ```json
   {"task_id": "xxx", "model_answer": "answer", "reasoning_trace": "reasoning process"}
   {"task_id": "yyy", "model_answer": "answer", "reasoning_trace": "reasoning process"}
   ```

2. **Tệp Hướng dẫn Nộp**: `SUBMISSION_GUIDE_*.md`

Mở tệp `SUBMISSION_GUIDE_*.md` được sinh tự động, tệp này chứa hướng dẫn nộp đầy đủ:

Cụ thể, mở trình duyệt và truy cập:
```
https://huggingface.co/spaces/gaia-benchmark/leaderboard
```

Như minh họa trong Hình 12.4, điền thông tin vào biểu mẫu nộp:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-4.png" alt="" width="85%"/>
  <p>Hình 12.4 Sơ đồ Quy trình Đánh giá BFCL</p>
</div>

Trước khi nộp, bạn có thể kiểm tra thủ công tệp JSONL được sinh ra:

```python
import json

# Read result file
with open("evaluation_results/gaia_official/gaia_level1_result_*.jsonl", "r") as f:
    for line in f:
        result = json.loads(line)
        print(f"Task ID: {result['task_id']}")
        print(f"Answer: {result['model_answer']}")
        print(f"Reasoning: {result['reasoning_trace']}")
        print("-" * 50)
```

### 12.3.5 Chi tiết Triển khai các Thành phần Cốt lõi

Việc triển khai hệ thống đánh giá GAIA tương tự như BFCL, nhưng có một số thiết kế đặc biệt cho việc đánh giá năng lực tổng quát.

**(1) GAIADataset: Bộ Nạp Dữ liệu Đa phương thức**

Đặc điểm đặc biệt của tập dữ liệu GAIA là nó chứa dữ liệu đa phương thức (văn bản, tệp, hình ảnh, v.v.):

````python
class GAIADataset:
    """GAIA dataset loader

    Supports loading GAIA dataset from HuggingFace (gated dataset)
    """

    def __init__(
        self,
        level: Optional[int] = None,
        split: str = "validation",
        local_data_dir: Optional[str] = None
    ):
        self.level = level
        self.split = split
        self.local_data_dir = local_data_dir or "./data/gaia"
        self.data = []

    def load(self) -> List[Dict[str, Any]]:
        """Load dataset"""
        # Download from HuggingFace
        items = self._load_from_huggingface()

        # Filter by level
        if self.level:
            items = [item for item in items if item.get("level") == self.level]

        self.data = items
        return items

    def _load_from_huggingface(self) -> List[Dict[str, Any]]:
        """Download GAIA dataset from HuggingFace"""
        from huggingface_hub import snapshot_download
        import json

        # Download dataset
        repo_id = "gaia-benchmark/GAIA"
        local_dir = snapshot_download(
            repo_id=repo_id,
            repo_type="dataset",
            local_dir=self.local_data_dir,
            local_dir_use_symlinks=False
        )

        # Load JSONL file
        data_file = Path(local_dir) / "2023" / self.split / "metadata.jsonl"
        items = []
        with open(data_file, 'r', encoding='utf-8') as f:
            for line in f:
                item = json.loads(line)
                items.append(self._standardize_item(item))

        return items
````

**(2) GAIAEvaluator: Triển khai Thuật toán Đánh giá Chính thức của GAIA**

Đánh giá GAIA sử dụng thuật toán **Quasi Exact Match**, đòi hỏi logic chuẩn hóa và khớp câu trả lời đặc biệt:

````python
class GAIAEvaluator:
    """GAIA evaluator

    Implements GAIA official Quasi Exact Match evaluation algorithm
    """

    def evaluate(self, agent: Any, max_samples: Optional[int] = None) -> Dict[str, Any]:
        """Execute evaluation"""
        dataset_items = self.dataset.load()

        if max_samples:
            dataset_items = dataset_items[:max_samples]

        results = []
        for i, item in enumerate(dataset_items, 1):
            # 1. Construct prompt
            prompt = self._build_prompt(item["question"], item)

            # 2. Call agent
            response = agent.run(prompt)

            # 3. Extract answer (GAIA format: FINAL ANSWER: [answer])
            predicted_answer = self._extract_answer(response)

            # 4. Normalize answer (GAIA official rules)
            normalized_pred = self._normalize_answer(predicted_answer)
            normalized_truth = self._normalize_answer(item["final_answer"])

            # 5. Quasi exact match
            exact_match = (normalized_pred == normalized_truth)

            results.append({
                "task_id": item["task_id"],
                "predicted": predicted_answer,
                "expected": item["final_answer"],
                "exact_match": exact_match,
                "level": item.get("level", 0)
            })

        return self._format_results(results)
````

GAIA sử dụng các quy tắc chuẩn hóa cụ thể để xử lý các loại câu trả lời khác nhau:

```python
def _normalize_answer(self, answer: str) -> str:
    """Normalize answer string (GAIA official normalization rules)

    Rules:
    1. Numbers: Remove comma separators and unit symbols
    2. Strings: Remove articles, convert to lowercase, remove extra spaces
    3. Lists: Comma-separated, sorted alphabetically
    """
    if not answer:
        return ""

    answer = answer.strip()

    # Check if it's a comma-separated list
    if ',' in answer:
        parts = [self._normalize_single_answer(p.strip()) for p in answer.split(',')]
        parts.sort()  # GAIA requires alphabetical sorting
        return ','.join(parts)
    else:
        return self._normalize_single_answer(answer)

def _normalize_single_answer(self, answer: str) -> str:
    """Normalize single answer (answer without commas)"""
    answer = answer.strip().lower()

    # Remove common articles
    articles = ['the', 'a', 'an']
    words = answer.split()
    if words and words[0] in articles:
        words = words[1:]
        answer = ' '.join(words)

    # Remove currency symbols and percent signs
    answer = answer.replace('$', '').replace('%', '').replace('€', '').replace('£', '')

    # Remove comma separators in numbers
    answer = re.sub(r'(\d),(\d)', r'\1\2', answer)

    # Remove extra spaces
    answer = ' '.join(answer.split())

    # Remove trailing punctuation
    answer = answer.rstrip('.,;:!?')

    return answer
```

GAIA yêu cầu định dạng đầu ra của mô hình là `FINAL ANSWER: [answer]`:

```python
def _extract_answer(self, response: str) -> str:
    """Extract answer from response (GAIA format)

    GAIA requires answer format: FINAL ANSWER: [answer]
    """
    # First try to extract GAIA official format answer
    final_answer_pattern = r'FINAL ANSWER:\s*(.+?)(?:\n|$)'
    match = re.search(final_answer_pattern, response, re.IGNORECASE | re.MULTILINE)
    if match:
        answer = match.group(1).strip()
        # Remove possible brackets
        answer = answer.strip('[]')
        return answer

    # Fallback: Look for other answer markers
    answer_patterns = [
        r'答案[：:]\s*(.+)',
        r'最终答案[：:]\s*(.+)',
        r'Final answer[：:]\s*(.+)',
        r'Answer[：:]\s*(.+)',
    ]

    for pattern in answer_patterns:
        match = re.search(pattern, response, re.IGNORECASE)
        if match:
            return match.group(1).strip()

    # If no marker found, return last non-empty line
    lines = response.strip().split('\n')
    for line in reversed(lines):
        line = line.strip()
        if line and not line.startswith('#'):
            return line

    return response.strip()
```

Sau khi đánh giá hoàn tất, có thể xuất sang định dạng JSONL mà GAIA chính thức yêu cầu:

```python
def export_to_gaia_format(
    self,
    results: Dict[str, Any],
    output_path: Union[str, Path],
    include_reasoning: bool = True
) -> None:
    """Export to GAIA official format (JSONL)

    GAIA required format:
    {"task_id": "xxx", "model_answer": "answer", "reasoning_trace": "reasoning process"}
    """
    output_path = Path(output_path)
    output_path.parent.mkdir(parents=True, exist_ok=True)

    with open(output_path, 'w', encoding='utf-8') as f:
        for result in results.get("detailed_results", []):
            entry = {
                "task_id": result["task_id"],
                "model_answer": result["predicted"]
            }

            if include_reasoning:
                entry["reasoning_trace"] = result.get("response", result["predicted"])

            f.write(json.dumps(entry, ensure_ascii=False) + '\n')
```

**(3) GAIAEvaluationTool: Công cụ Đánh giá Một Nhấp**

GAIAEvaluationTool đóng gói toàn bộ quy trình đánh giá, cung cấp chức năng đánh giá một nhấp:

````python
class GAIAEvaluationTool(Tool):
    """GAIA evaluation tool

    Provides one-click evaluation functionality:
    1. Run HelloAgents evaluation
    2. Export GAIA format results
    3. Generate evaluation report
    4. Generate submission guide
    """

    def run(
        self,
        agent: Any,
        level: Optional[int] = None,
        max_samples: Optional[int] = None,
        local_data_dir: Optional[str] = None,
        export_results: bool = True,
        generate_report: bool = True
    ) -> Dict[str, Any]:
        """Execute GAIA one-click evaluation"""
        # Step 1: Run HelloAgents evaluation
        results = self._run_evaluation(agent, level, max_samples, local_data_dir)

        # Step 2: Export GAIA format results
        if export_results:
            self._export_results(results)

        # Step 3: Generate evaluation report
        if generate_report:
            self.generate_report(results)

        return results
````

GAIAEvaluationTool tự động sinh báo cáo đánh giá:

```python
def generate_report(
    self,
    results: Dict[str, Any],
    output_file: Optional[Union[str, Path]] = None
) -> str:
    """Generate evaluation report"""
    report = f"""# GAIA Evaluation Report

**Generated**: {datetime.now().strftime("%Y-%m-%d %H:%M:%S")}

## 📊 Evaluation Overview

- **Agent**: {results.get("agent_name", "Unknown")}
- **Difficulty Level**: {results.get("level_filter") or 'All'}
- **Total Samples**: {results.get("total_samples", 0)}
- **Exact Matches**: {results.get("exact_matches", 0)}
- **Exact Match Rate**: {results.get("exact_match_rate", 0):.2%}

## 📈 Detailed Metrics

### Level-wise Accuracy

{self._format_level_metrics(results.get("level_metrics", {}))}

## 📝 Sample Details (First 10)

{self._format_sample_details(results.get("detailed_results", [])[:10])}

## 📊 Accuracy Visualization

{self._format_visualization(results.get("exact_match_rate", 0))}

## 💡 Recommendations

{self._format_suggestions(results.get("exact_match_rate", 0))}
"""

    # Save report
    if output_file is None:
        output_dir = Path("./evaluation_reports")
        output_dir.mkdir(parents=True, exist_ok=True)
        output_file = output_dir / f"gaia_report_{datetime.now().strftime('%Y%m%d_%H%M%S')}.md"

    with open(output_file, 'w', encoding='utf-8') as f:
        f.write(report)

    return report
```

## 12.4 Đánh giá Chất lượng Sinh Dữ liệu

Trong quá trình phát triển hệ thống AI, dữ liệu huấn luyện chất lượng cao là nền tảng cho hiệu năng của hệ thống. Phần này giới thiệu cách sử dụng framework HelloAgents để đánh giá chất lượng của dữ liệu được sinh ra, lấy ví dụ về việc sinh bài toán toán học theo phong cách AIME (American Invitational Mathematics Examination)<sup>[9]</sup>.

AIME là một cuộc thi toán học có độ khó trung bình do Hiệp hội Toán học Hoa Kỳ (Mathematical Association of America - MAA) tổ chức, nằm giữa AMC 10/12 và Olympic Toán học Hoa Kỳ (USAMO). Các bài toán AIME có những đặc điểm riêng biệt: đáp án của mỗi bài là một số nguyên trong khoảng từ 0 đến 999, các bài toán bao phủ nhiều lĩnh vực toán học bao gồm đại số, hình học, lý thuyết số, tổ hợp và xác suất, đòi hỏi suy luận nhiều bước nhưng không liên quan đến lý thuyết cao cấp, và có độ khó vừa phải (tương đương các bài toán AIME số 6-9). Những đặc điểm này khiến các bài toán AIME trở thành một benchmark lý tưởng để đánh giá chất lượng sinh bài toán toán học: định dạng đáp án thống nhất tạo điều kiện cho việc đánh giá tự động, và độ khó vừa phải phù hợp cho việc sinh dữ liệu quy mô lớn. Chúng ta sử dụng tập dữ liệu `TianHongZXY/aime-1983-2025` trên HuggingFace làm tham chiếu, tập này chứa hơn 900 bài toán AIME thực từ năm 1983 đến 2025, cung cấp các mẫu tham chiếu phong phú cho việc sinh và đánh giá của chúng ta.

### 12.4.1 Tổng quan các Phương pháp Đánh giá

Trong đánh giá chất lượng sinh dữ liệu, chúng ta áp dụng ba phương pháp đánh giá bổ trợ lẫn nhau: LLM Judge (LLM làm giám khảo), Win Rate (tỷ lệ thắng) và Manual Verification (kiểm chứng thủ công). Có hai lý do quan trọng để lựa chọn ba phương pháp này. Thứ nhất, từ góc độ phương pháp luận, đây là những phương án đánh giá tự động thường được sử dụng trong lĩnh vực agent hiện nay và là thực tiễn chủ đạo trong nhiều bài báo học thuật, có sự công nhận rộng rãi và nền tảng thực tiễn. Thứ hai, từ góc độ tính ứng dụng, ba phương pháp này phù hợp một cách tự nhiên với kịch bản đánh giá của chúng ta: LLM Judge và Win Rate được dùng để đánh giá chất lượng sinh bài toán (đánh giá đa chiều từ tính đúng đắn, độ rõ ràng, mức độ khớp độ khó, v.v.), trong khi Manual Verification được dùng để đánh giá chất lượng sinh đáp án (kiểm chứng độ chính xác của đáp án thông qua các chuyên gia con người), sự phân chia công việc này rất hợp lý và dễ hiểu.

Dưới đây chúng ta giới thiệu chi tiết cách triển khai cụ thể của ba phương pháp đánh giá này. Luồng triển khai của toàn bộ case study được minh họa trong Hình 12.5:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-5.png" alt="" width="85%"/>
  <p>Hình 12.5 Sơ đồ Luồng Đánh giá Chất lượng Sinh Dữ liệu</p>
</div>

**(1) Đánh giá LLM Judge**

**Động lực Thiết kế**: Trong đánh giá chất lượng sinh dữ liệu, chúng ta cần đánh giá nhanh chóng và nhất quán chất lượng của một lượng lớn bài toán được sinh ra. Đánh giá thủ công truyền thống, mặc dù chính xác, nhưng tốn kém và kém hiệu quả, khó đáp ứng nhu cầu sinh dữ liệu quy mô lớn. LLM Judge, bằng cách sử dụng các mô hình ngôn ngữ lớn làm giám khảo, có thể tự động đánh giá chất lượng dữ liệu được sinh ra từ nhiều chiều, không chỉ cải thiện đáng kể hiệu quả đánh giá mà còn duy trì tính nhất quán trong các tiêu chuẩn đánh giá. Quan trọng hơn, LLM Judge có thể cung cấp các lý do chấm điểm chi tiết và các gợi ý cải thiện, giúp chúng ta hiểu được điểm mạnh và điểm yếu của dữ liệu được sinh ra và cung cấp định hướng cho việc tối ưu hóa sau này.

Trong phần triển khai của chúng ta, LLM Judge đánh giá chất lượng bài toán AIME từ bốn chiều then chốt:

<div align="center">
  <p>Bảng 12.5 Các Chiều Đánh giá LLM Judge cho Bài toán AIME</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-table-5.png" alt="" width="85%"/>
</div>

Sau khi thu được điểm số từ bốn chiều, chúng ta cần tổng hợp các điểm số này thành các chỉ số đánh giá tổng thể. Chúng ta định nghĩa ba chỉ số then chốt để đo lường mức độ chất lượng của các bài toán được sinh ra:

**Các Chỉ số Đánh giá**:

**1. Điểm Trung bình (Average Score)**: Tính điểm trung bình của tất cả các bài toán trên bốn chiều, phản ánh mức độ chất lượng tổng thể của các bài toán được sinh ra.
$$
\text{Average Score} = \frac{1}{N} \sum_{i=1}^{N} \frac{\sum_{d=1}^{4} S_{i,d}}{4}
$$

**2. Tỷ lệ Đạt (Pass Rate)**: Đếm tỷ lệ các bài toán có điểm trung bình từ 3.5 trở lên, phản ánh sự đảm bảo chất lượng cơ bản của các bài toán được sinh ra.

$$
\text{Pass Rate} = \frac{|\{i : \text{Score}_i \geq 3.5\}|}{N}
$$

**3. Tỷ lệ Xuất sắc (Excellent Rate)**: Đếm tỷ lệ các bài toán có điểm trung bình từ 4.5 trở lên, phản ánh tỷ lệ chất lượng cao của các bài toán được sinh ra.

$$
\text{Excellent Rate} = \frac{|\{i : \text{Score}_i \geq 4.5\}|}{N}
$$

Trong đó:
- $N$ là tổng số bài toán được đánh giá
- $S_{i,d}$ là điểm số của bài toán thứ $i$ trên chiều thứ $d$ (1-5 điểm)
- $\text{Score}_i$ là điểm trung bình của bài toán thứ $i$ (trung bình của bốn điểm chiều)

Ba chỉ số này phản ánh chất lượng sinh dữ liệu từ các góc độ khác nhau: điểm trung bình cho mức độ tổng thể, tỷ lệ đạt đảm bảo chất lượng cơ bản, tỷ lệ xuất sắc đo lường năng lực tạo ra đầu ra chất lượng cao.

**(2) Đánh giá Win Rate**

**Động lực Thiết kế**: Mặc dù LLM Judge có thể cung cấp chấm điểm tuyệt đối đa chiều, chúng ta cũng cần một chỉ số đánh giá tương đối để đo lường khoảng cách chất lượng giữa các bài toán được sinh ra và các bài toán thực. Đánh giá Win Rate, thông qua so sánh từng cặp, để LLM trực tiếp phán đoán bên nào tốt hơn giữa bài toán được sinh ra và bài toán thực. So sánh tương đối này phù hợp với thói quen phán đoán của con người hơn so với chấm điểm tuyệt đối, và có thể dễ dàng hơn trong việc phát hiện các ưu nhược điểm tương đối của các bài toán được sinh ra. Trong trường hợp lý tưởng, nếu chất lượng của các bài toán được sinh ra gần với các bài toán thực, Win Rate nên ở khoảng 50% (tức là bài toán được sinh ra và bài toán thực mỗi bên có tỷ lệ thắng 50%). Chỉ số này đơn giản và trực quan, cho phép nhanh chóng phán đoán mức độ chất lượng tổng thể của hệ thống sinh dữ liệu.

Trong phần triển khai của chúng ta, đánh giá Win Rate được tiến hành thông qua luồng được minh họa trong Hình 12.6:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-6.png" alt="" width="85%"/>
  <p>Hình 12.6 Sơ đồ Luồng Đánh giá Chất lượng Sinh Dữ liệu</p>
</div>

Trong đánh giá so sánh từng cặp, mỗi lần so sánh tạo ra ba kết quả khả dĩ: bài toán được sinh ra thắng (Win), bài toán thực thắng (Loss), hoặc hòa (Tie). Chúng ta đánh giá chất lượng của các bài toán được sinh ra bằng cách đếm tỷ lệ của ba kết quả này:

**Các Chỉ số Đánh giá**:

**1. Win Rate (Tỷ lệ thắng)**: Tỷ lệ các bài toán được sinh ra được phán đoán là tốt hơn, phản ánh ưu điểm của các bài toán được sinh ra so với các bài toán thực.

$$
\text{Win Rate} = \frac{\text{Wins}}{\text{Total Comparisons}}
$$

**2. Loss Rate (Tỷ lệ thua)**: Tỷ lệ các bài toán thực được phán đoán là tốt hơn, phản ánh nhược điểm của các bài toán được sinh ra so với các bài toán thực.

$$
\text{Loss Rate} = \frac{\text{Losses}}{\text{Total Comparisons}}
$$

**3. Tie Rate (Tỷ lệ hòa)**: Tỷ lệ được phán đoán là chất lượng tương đương, phản ánh sự tương đồng giữa các bài toán được sinh ra và các bài toán thực.

$$
\text{Tie Rate} = \frac{\text{Ties}}{\text{Total Comparisons}}
$$

Trong đó Total Comparisons là tổng số lần so sánh, Wins, Losses và Ties lần lượt là số lần bài toán được sinh ra thắng, thua và hòa. Ba chỉ số này thỏa mãn: Win Rate + Loss Rate + Tie Rate = 100%.

**Kết quả Lý tưởng**: Win Rate ≈ 50% (cho thấy chất lượng sinh dữ liệu gần với các bài toán thực). Nếu Win Rate thấp hơn đáng kể so với 50%, điều đó cho thấy chất lượng bài toán được sinh ra kém hơn các bài toán thực và chiến lược sinh dữ liệu cần được tối ưu hóa; nếu Win Rate cao hơn đáng kể so với 50%, có thể cho thấy các bài toán được sinh ra vượt qua các bài toán thực ở một số khía cạnh, hoặc có sự thiên lệch trong các tiêu chuẩn đánh giá.

**(3) Kiểm chứng Thủ công**

**Động lực Thiết kế**: Mặc dù LLM Judge và Win Rate có thể tự động đánh giá chất lượng bài toán, đối với các bài toán toán học đòi hỏi suy luận logic chặt chẽ, kiểm chứng thủ công vẫn là điều không thể thiếu. Đặc biệt khi đánh giá chất lượng sinh đáp án, cần các chuyên gia con người để kiểm chứng độ chính xác của đáp án, tính đầy đủ của các bước giải, và tính chặt chẽ của suy luận toán học. Ngoài ra, kiểm chứng thủ công có thể phát hiện các vấn đề mà đánh giá tự động có thể bỏ sót, chẳng hạn như các yếu tố chủ quan như tính sáng tạo và tính thú vị của bài toán. Để cải thiện hiệu quả và trải nghiệm kiểm chứng thủ công, chúng ta đã phát triển một giao diện Web dựa trên Gradio, cho phép người kiểm chứng thuận tiện duyệt qua các bài toán, chấm điểm, chú thích trạng thái, và thêm nhận xét, giảm đáng kể rào cản cho việc kiểm chứng thủ công.

Trong phần triển khai của chúng ta, kiểm chứng thủ công được tiến hành thông qua các bước sau:

1. Đọc bài toán, đáp án, lời giải
2. Chấm điểm (1-5 điểm): tính đúng đắn, độ rõ ràng, mức độ khớp độ khó, tính đầy đủ
3. Chú thích trạng thái:
   - ✅ approved (đã duyệt)
   - ❌ rejected (bị từ chối)
   - 🔄 needs_revision (cần chỉnh sửa)
4. Thêm nhận xét

### 12.4.2 Kiến trúc Hệ thống

Hệ thống sinh và đánh giá dữ liệu áp dụng thiết kế mô-đun hóa:

```
data_generation/
├── aime_generator.py              # AIME problem generator
├── human_verification_ui.py       # Manual verification interface
├── run_complete_evaluation.py     # Complete evaluation flow
│
├── generated_data/                # Generated data
│   ├── aime_generated_XXXXXX.json
│   └── generation_report_XXXXXX.md
│
└── evaluation_results/            # Evaluation results
    └── XXXXXX/
        ├── llm_judge/
        ├── win_rate/
        └── comprehensive_report.md
```

Hệ thống chứa bốn thành phần cốt lõi: Đầu tiên là AIMEGenerator (bộ sinh bài toán), sử dụng framework HelloAgents để sinh các bài toán theo phong cách AIME, hỗ trợ sinh theo lô và lưu tiến độ, và tự động xử lý giới hạn tốc độ API; thứ hai là LLMJudgeTool (công cụ đánh giá LLM Judge), cung cấp đánh giá chất lượng theo 4 chiều, tự động sinh kết quả JSON và báo cáo Markdown; thứ ba là WinRateTool (công cụ đánh giá Win Rate), tính toán tỷ lệ thắng, tỷ lệ thua, và tỷ lệ hòa thông qua đánh giá so sánh từng cặp; cuối cùng là HumanVerificationUI (giao diện kiểm chứng thủ công), dựa trên giao diện Web Gradio, hỗ trợ chấm điểm và chú thích trạng thái.

### 12.4.3 Triển khai Bộ sinh Bài toán AIME

```python
class AIMEGenerator:
    """AIME Problem Generator"""

    def __init__(
        self,
        llm: HelloAgentsLLM = None,
        delay_seconds: float = 1.0,
        use_reference_examples: bool = True,
        reference_dataset: str = "TianHongZXY/aime-1983-2025"
    ):
        self.llm = llm or HelloAgentsLLM()
        self.agent = SimpleAgent(
            name="AIME Generator",
            llm=self.llm,
            system_prompt="You are a professional mathematics competition problem designer."
        )
        self.delay_seconds = delay_seconds
        self.use_reference_examples = use_reference_examples

        # Load reference examples from 900+ AIME problems (1983-2025)
        if use_reference_examples:
            dataset = load_dataset(reference_dataset, split="test")
            self.reference_examples = list(dataset)
```

Mục tiêu của chúng ta là sinh ra một tập dữ liệu có phong cách tương tự, vì vậy chúng ta chọn ngẫu nhiên các mẫu tham chiếu từ hơn 900 bài toán AIME thực (1983-2025)

Thiết kế prompt sinh dữ liệu (tiếng Anh):

```python
GENERATION_PROMPT = """You are a professional mathematics competition problem designer, skilled in creating AIME (American Invitational Mathematics Examination) style problems.

【Reference Example】(For style reference only, please generate a completely different problem)
Problem: {example_problem}
Answer: {example_answer}

AIME Problem Characteristics:
1. Answer: An integer between 0 and 999
2. Topics: Algebra, Geometry, Number Theory, Combinatorics, Probability, etc.
3. Style: Requires multi-step reasoning, but no advanced theory
4. Difficulty: Medium to hard (similar to AIME problems 6-9)

Please generate a **completely different** AIME-style mathematics problem, including:
1. Problem statement (clear and complete, different from the reference)
2. Answer (an integer between 0 and 999, different from the reference)
3. Detailed solution (including all reasoning steps)
4. Topic classification (Algebra/Geometry/Number Theory/Combinatorics/Probability)

Please output in the following JSON format:
{
    "problem": "Problem statement in English",
    "answer": 123,
    "solution": "Detailed solution steps in English",
    "topic": "Algebra"
}
"""
```

Chúng ta chọn sinh các bài toán bằng tiếng Anh vì bốn lý do quan trọng: thứ nhất là tính nhất quán với các bài toán AIME thực (AIME là một cuộc thi bằng tiếng Anh, sinh các bài toán tiếng Anh là hợp lý hơn), thứ hai là đảm bảo tính công bằng trong đánh giá (đánh giá LLM Judge công bằng hơn khi tiếng Anh so với tiếng Anh), thứ ba là tạo điều kiện quốc tế hóa (các bài toán tiếng Anh có thể được sử dụng rộng rãi hơn), và cuối cùng là tránh các vấn đề dịch thuật (không cần lo lắng về độ chính xác của việc dịch Trung-Anh).

Triển khai sinh theo lô:

```python
def generate_and_save(self, num_problems: int = 30, output_dir: str = "data_generation/generated_data"):
    """Generate and save problems with intelligent delay"""
    # Clean old checkpoints
    for file in os.listdir(output_dir):
        if file.startswith("checkpoint_") and file.endswith(".json"):
            os.remove(os.path.join(output_dir, file))

    # Generate with tqdm progress bar
    with tqdm(total=num_problems, desc="Generating AIME problems", unit="problem") as pbar:
        last_call_time = 0

        for i in range(num_problems):
            # Ensure minimum delay between API calls
            if last_call_time > 0:
                elapsed = time.time() - last_call_time
                if elapsed < self.delay_seconds:
                    wait_time = self.delay_seconds - elapsed
                    time.sleep(wait_time)

            # Generate problem (randomly select reference example)
            start_time = time.time()
            problem = self.generate_single()
            last_call_time = time.time()
            generation_time = last_call_time - start_time

            # Update progress bar
            pbar.set_postfix({
                "topic": problem.get('topic', 'N/A'),
                "answer": problem.get('answer', 'N/A'),
                "time": f"{generation_time:.1f}s"
            })
            pbar.update(1)

    return generated_data_path
```

Hỗ trợ công thức toán học LaTeX:

Các bài toán AIME được sinh ra chứa các công thức toán học LaTeX (chẳng hạn `$\frac{a}{b}$`, `$\sqrt{x}$`), đòi hỏi xử lý phân tích JSON đặc biệt:

```python
def _parse_response(self, response: str) -> Dict[str, Any]:
    """Parse LLM response (supports LaTeX mathematical formulas)"""
    import re

    # Extract JSON part
    if "```json" in response:
        json_str = response.split("```json")[1].split("```")[0].strip()
    else:
        json_str = response.strip()

    try:
        problem_data = json.loads(json_str)
    except json.JSONDecodeError:
        # Fix LaTeX escape issue: convert \frac to \\frac
        # Regular expression: find unescaped backslashes
        fixed_json_str = re.sub(r'(?<!\\)\\(?!["\\/bfnrtu])', r'\\\\', json_str)
        problem_data = json.loads(fixed_json_str)

    return problem_data
```

Các dấu gạch chéo ngược trong các công thức LaTeX (chẳng hạn `\frac`, `\sqrt`) là các ký tự escape bất hợp lệ trong JSON, gây ra lỗi phân tích:
```
Invalid \escape: line 4 column 185 (char 375)
```

Bằng cách sử dụng biểu thức chính quy để thay thế các dấu gạch chéo ngược chưa được escape bằng dấu gạch chéo ngược kép, khiến chúng trở nên hợp lệ trong JSON.

### 12.4.4 Công cụ Đánh giá LLM Judge

Công cụ LLM Judge sử dụng LLM làm giám khảo để tiến hành đánh giá đa chiều đối với các bài toán được sinh ra.

```python
class LLMJudgeTool(Tool):
    """LLM Judge evaluation tool"""

    def run(self, params: Dict[str, Any]) -> str:
        """Run LLM Judge evaluation"""
        # 1. Load generated data
        gen_dataset = AIDataset(dataset_type="generated", data_path=params["generated_data_path"])
        gen_problems = gen_dataset.load()

        # 2. Load reference data (AIME 2025)
        ref_dataset = AIDataset(dataset_type="real", year=2025)
        ref_problems = ref_dataset.load()

        # 3. Create evaluator
        evaluator = LLMJudgeEvaluator(llm=self.llm, judge_model=params.get("judge_model", "gpt-4o"))

        # 4. Run evaluation
        results = evaluator.evaluate_batch(gen_problems, max_samples=params.get("max_samples"))

        # 5. Save results
        evaluator.export_results(results, result_file)

        # 6. Generate report
        self._generate_report(results, report_file)

        return json.dumps({"status": "success", "metrics": results["metrics"]})
```

**Prompt Đánh giá**:

```python
EVALUATION_PROMPT = """Please evaluate the quality of the following AIME mathematics problem.

Problem:
{problem}

Answer: {answer}

Solution:
{solution}

Please score from the following 4 dimensions (1-5 points):

1. **Correctness**: Is the mathematical logic correct, is the answer accurate
2. **Clarity**: Is the problem statement clear, is the solution easy to understand
3. **Difficulty Match**: Does the difficulty match AIME standards (medium to hard)
4. **Completeness**: Are the solution steps complete, does it include necessary reasoning

Please output in the following JSON format:
{
    "correctness": 5,
    "clarity": 4,
    "difficulty_match": 4,
    "completeness": 5,
    "comments": "Evaluation reason"
}
"""
```

**Ví dụ Báo cáo Đánh giá**:

```markdown
# LLM Judge Evaluation Report

## Overall Score

- **Average Total Score**: 4.2/5.0
- **Pass Rate**: 85.0% (≥3.5 points)
- **Excellent Rate**: 40.0% (≥4.5 points)

## Dimension Scores

| Dimension | Average Score | Rating |
|------|--------|------|
| Correctness | 4.3/5.0 | Good ⭐⭐⭐⭐ |
| Clarity | 4.1/5.0 | Good ⭐⭐⭐⭐ |
| Difficulty Match | 4.0/5.0 | Good ⭐⭐⭐⭐ |
| Completeness | 4.4/5.0 | Good ⭐⭐⭐⭐ |
```

### 12.4.5 Công cụ Đánh giá Win Rate

Công cụ Win Rate đánh giá chất lượng của dữ liệu được sinh ra so với các bài toán thực thông qua so sánh từng cặp.

```python
class WinRateTool(Tool):
    """Win Rate evaluation tool"""

    def run(self, params: Dict[str, Any]) -> str:
        """Run Win Rate evaluation"""
        # 1. Load generated data
        gen_dataset = AIDataset(dataset_type="generated", data_path=params["generated_data_path"])
        gen_problems = gen_dataset.load()

        # 2. Load reference data (AIME 2025)
        ref_dataset = AIDataset(dataset_type="real", year=2025)
        ref_problems = ref_dataset.load()

        # 3. Create evaluator
        evaluator = WinRateEvaluator(llm=self.llm, judge_model=params.get("judge_model", "gpt-4o"))

        # 4. Run evaluation
        results = evaluator.evaluate_win_rate(gen_problems, ref_problems, num_comparisons=params.get("num_comparisons"))

        # 5. Save results and report
        evaluator.export_results(results, result_file)
        self._generate_report(results, report_file)

        return json.dumps({"status": "success", "metrics": results["metrics"]})
```

AIDataset chịu trách nhiệm nạp dữ liệu được sinh ra và dữ liệu bài toán AIME thực, hỗ trợ hai loại dữ liệu:

```python
class AIDataset:
    """AI dataset loader

    Supports two data types:
    1. generated: Generated data (JSON format)
    2. real: AIME real problems (loaded from HuggingFace)
    """

    def __init__(
        self,
        dataset_type: str = "generated",
        data_path: Optional[str] = None,
        year: Optional[int] = None
    ):
        self.dataset_type = dataset_type
        self.data_path = data_path
        self.year = year  # Only for real type, default 2025

    def load(self) -> List[Dict[str, Any]]:
        """Load dataset"""
        if self.dataset_type == "generated":
            return self._load_generated_data()
        elif self.dataset_type == "real":
            return self._load_real_data()

    def _load_real_data(self) -> List[Dict[str, Any]]:
        """Load AIME 2025 real problems from HuggingFace"""
        from huggingface_hub import snapshot_download

        # Use AIME 2025 dataset
        repo_id = "math-ai/aime25"

        # Download dataset
        local_dir = snapshot_download(
            repo_id=repo_id,
            repo_type="dataset"
        )

        # Read JSONL file
        data_file = list(Path(local_dir).glob("*.jsonl"))[0]
        data = []
        with open(data_file, 'r', encoding='utf-8') as f:
            for line in f:
                if line.strip():
                    data.append(json.loads(line))

        # Unify data format (AIME 2025 uses lowercase field names)
        problems = []
        for idx, item in enumerate(data):
            problem = {
                "problem_id": item.get("id", f"aime_2025_{idx}"),
                "problem": item.get("problem", ""),
                "answer": item.get("answer", ""),
                "solution": item.get("solution", ""),  # AIME 2025 has no solution field
            }
            problems.append(problem)

        return problems
```

Chúng ta chọn chỉ sử dụng tập dữ liệu AIME 2025 vì bốn lý do: thứ nhất là tính kịp thời của dữ liệu (2025 là dữ liệu cuộc thi AIME mới nhất), thứ hai là đơn giản hóa việc bảo trì (chỉ bảo trì một tập dữ liệu, mã ngắn gọn hơn), thứ ba là định dạng thống nhất (định dạng JSONL, tên trường thống nhất về chữ thường), và cuối cùng là tính đại diện đầy đủ (30 bài toán đủ để đánh giá chất lượng sinh dữ liệu).

**Prompt So sánh**:

```python
COMPARISON_PROMPT = """Please compare the quality of the following two AIME mathematics problems and judge which is better.

【Problem A - Generated Problem】
Problem: {problem_a}
Answer: {answer_a}
Solution: {solution_a}

【Problem B - AIME Real Problem】
Problem: {problem_b}
Answer: {answer_b}
Solution: {solution_b}

Please compare from the following aspects:
1. Rigor of mathematical logic
2. Clarity of problem statement
3. Reasonableness of difficulty
4. Completeness of solution

Please output in the following JSON format:
{
    "winner": "A" or "B" or "Tie",
    "reason": "Judgment reason"
}
"""
```

**Ví dụ Báo cáo Đánh giá**:

```markdown
# Win Rate Evaluation Report

## Win Rate Statistics

| Metric | Value | Percentage |
|------|------|--------|
| Generated Data Wins | 9 times | 45.0% |
| AIME Real Problems Win | 8 times | 40.0% |
| Tie | 3 times | 15.0% |

**Win Rate**: 45.0%

✅ **Good**: Generated data quality is close to reference data (gap <10%).
```

### 12.4.6 Giao diện Kiểm chứng Thủ công

Sử dụng Gradio để tạo giao diện Web, hỗ trợ kiểm chứng thủ công các bài toán được sinh ra.

```python
class HumanVerificationUI:
    """Manual verification interface"""

    def launch(self, share: bool = False):
        """Launch Gradio interface"""
        with gr.Blocks(title="AIME Problem Manual Verification") as demo:
            gr.Markdown("# 🎯 AIME Problem Manual Verification System")

            with gr.Row():
                with gr.Column(scale=2):
                    # Problem display area
                    problem_text = gr.Textbox(label="Problem Description", lines=5, interactive=False)
                    answer_text = gr.Textbox(label="Answer", interactive=False)
                    solution_text = gr.Textbox(label="Solution Process", lines=10, interactive=False)

                with gr.Column(scale=1):
                    # Scoring area
                    correctness_slider = gr.Slider(1, 5, value=3, step=1, label="Correctness")
                    clarity_slider = gr.Slider(1, 5, value=3, step=1, label="Clarity")
                    difficulty_slider = gr.Slider(1, 5, value=3, step=1, label="Difficulty Match")
                    completeness_slider = gr.Slider(1, 5, value=3, step=1, label="Completeness")

                    # Status selection
                    status_radio = gr.Radio(
                        choices=["approved", "rejected", "needs_revision"],
                        value="approved",
                        label="Status"
                    )

                    # Verification button
                    verify_btn = gr.Button("✅ Submit Verification", variant="primary")

            demo.launch(share=share, server_name="127.0.0.1", server_port=7860)
```

**Cách Sử dụng**:

```bash
# Launch manual verification interface
python data_generation/human_verification_ui.py data_generation/generated_data/aime_generated_XXXXXX.json

# Open browser and visit
http://127.0.0.1:7860
```

Hiệu quả cuối cùng có thể tham khảo trong Hình 12.7. Đối với tính đúng đắn của bài toán, kiểm tra thủ công là tốt nhất:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/12-figures/12-7.png" alt="" width="85%"/>
  <p>Hình 12.7 Trang Kiểm chứng Thủ công Bài toán AIME</p>
</div>

**Quy trình Kiểm chứng**:

1. Mở giao diện kiểm chứng trong trình duyệt
2. Đọc bài toán, đáp án, lời giải
3. Chấm điểm từ 4 chiều (1-5 điểm)
4. Chọn trạng thái kiểm chứng (approved/rejected/needs_revision)
5. Thêm nhận xét (tùy chọn)
6. Nhấp "Submit Verification"
7. Xem bài toán tiếp theo

**Lưu Kết quả Kiểm chứng**:

Kết quả kiểm chứng được tự động lưu dưới dạng `<data_path>_verifications.json`:

```json
{
  "gen_aime_1": {
    "problem_id": "gen_aime_1",
    "scores": {
      "correctness": 5,
      "clarity": 4,
      "difficulty_match": 4,
      "completeness": 5
    },
    "total_score": 4.5,
    "status": "approved",
    "comments": "Problem quality is very good, logic is rigorous",
    "verified_at": "2025-01-10T12:00:00"
  }
}
```

### 12.4.7 Quy trình Đánh giá Hoàn chỉnh

Tích hợp tất cả các phương pháp đánh giá vào một quy trình hoàn chỉnh.

```python
def run_complete_evaluation(
    num_problems: int = 30,
    delay_seconds: float = 3.0
):
    """
    Run complete evaluation flow

    Args:
        num_problems: Number of problems to generate
        delay_seconds: Delay between each generation (seconds), avoid API rate limit
    """
    # Step 1: Generate AIME problems
    generator = AIMEGenerator(delay_seconds=delay_seconds)
    generated_data_path = generator.generate_and_save(
        num_problems=num_problems,
        output_dir="data_generation/generated_data"
    )

    # Step 2: Evaluation
    # Create evaluation result directory
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    evaluation_dir = f"data_generation/evaluation_results/{timestamp}"
    os.makedirs(evaluation_dir, exist_ok=True)
    os.makedirs(os.path.join(evaluation_dir, "llm_judge"), exist_ok=True)
    os.makedirs(os.path.join(evaluation_dir, "win_rate"), exist_ok=True)

    # Create LLM
    llm = HelloAgentsLLM()

    # Step 2.1: LLM Judge evaluation
    llm_judge_result = None
    try:
        llm_judge_tool = LLMJudgeTool(llm=llm)
        llm_judge_result_json = llm_judge_tool.run({
            "generated_data_path": generated_data_path,
            "reference_year": 2025,
            "max_samples": num_problems,
            "output_dir": os.path.join(evaluation_dir, "llm_judge"),
            "judge_model": "gpt-4o"
        })
        llm_judge_result = json.loads(llm_judge_result_json)
    except Exception as e:
        print(f"❌ LLM Judge evaluation failed: {e}")

    # Step 2.2: Win Rate evaluation
    win_rate_result = None
    try:
        win_rate_tool = WinRateTool(llm=llm)
        win_rate_result_json = win_rate_tool.run({
            "generated_data_path": generated_data_path,
            "reference_year": 2025,
            "num_comparisons": min(num_problems, 20),
            "output_dir": os.path.join(evaluation_dir, "win_rate"),
            "judge_model": "gpt-4o"
        })
        win_rate_result = json.loads(win_rate_result_json)
    except Exception as e:
        print(f"❌ Win Rate evaluation failed: {e}")

    # Step 3: Generate comprehensive report
    comprehensive_report_path = None
    if llm_judge_result or win_rate_result:
        comprehensive_report_path = os.path.join(evaluation_dir, "comprehensive_report.md")
        report = generate_comprehensive_report(
            generated_data_path,
            llm_judge_result,
            win_rate_result
        )
        with open(comprehensive_report_path, 'w', encoding='utf-8') as f:
            f.write(report)

    return {
        "generated_data_path": generated_data_path,
        "llm_judge_result": llm_judge_result,
        "win_rate_result": win_rate_result,
        "comprehensive_report_path": comprehensive_report_path
    }
```

**Cách Chạy**:

```bash
# Basic usage (default 3 second delay)
python data_generation/run_complete_evaluation.py 30

# Custom delay (recommended 3-5 seconds, avoid API rate limit)
python data_generation/run_complete_evaluation.py 30 3.0

# Parameter explanation:
# - 30: Number of problems to generate
# - 3.0: Delay between each generation (seconds)

# Explanation:
# - Generation phase: Randomly select reference examples from 900+ AIME real problems (1983-2025)
# - Evaluation phase: Quality comparison with AIME 2025 real problems
# - Dataset source: math-ai/aime25 (JSONL format)
```

**Ví dụ Đầu ra**:

```
================================================================================
🚀 AIME Data Generation and Evaluation Complete Flow
================================================================================

Configuration:
  - Number of problems to generate: 30
  - API delay: 3.0 seconds/problem
  - Generation reference data: TianHongZXY/aime-1983-2025 (900+ problems)
  - Evaluation reference: AIME 2025 real problems

================================================================================
📝 Step 1: Generate AIME Problems
================================================================================
📚 Load AIME real problem dataset: TianHongZXY/aime-1983-2025
   ✓ Loaded 963 reference problems

🎯 Start generating AIME problems
   Target quantity: 30
   Generation model: gpt-4o
   Delay setting: 3.0 seconds/problem

Generating AIME problems:  100%|██████████| 30/30 [01:30<00:00, 3.00s/problem, topic=Algebra, answer=123, time=3.0s]

✅ Step 1 complete! Generated data saved at: data_generation/generated_data/aime_generated_20250110_120000.json

🎯 Step 2.1: LLM Judge Evaluation (vs AIME 2025)

✅ LLM Judge evaluation complete!
   Average total score: 4.2/5.0
   Pass rate: 85.0%

🏆 Step 2.2: Win Rate Evaluation (vs AIME 2025)

✅ Win Rate evaluation complete!
   Win Rate: 45.0%

================================================================================
📊 Step 3: Generate Comprehensive Report
================================================================================

✅ Comprehensive report saved: data_generation/evaluation_results/20250110_120000/comprehensive_report.md

================================================================================
🎉 Complete Evaluation Flow Finished!
================================================================================

📁 Output Files:
   - Generated data: data_generation/generated_data/aime_generated_20250110_120000.json
   - Evaluation result directory: data_generation/evaluation_results/20250110_120000
   - LLM Judge report: data_generation/evaluation_results/20250110_120000/llm_judge/llm_judge_report_20250110_120000.md
   - Win Rate report: data_generation/evaluation_results/20250110_120000/win_rate/win_rate_report_20250110_120000.md
   - Comprehensive report: data_generation/evaluation_results/20250110_120000/comprehensive_report.md

💡 Next Steps:
   1. View comprehensive report: data_generation/evaluation_results/20250110_120000/comprehensive_report.md
   2. Run manual verification: python data_generation/human_verification_ui.py data_generation/generated_data/aime_generated_20250110_120000.json
```

### 12.4.8 Báo cáo Đánh giá Tổng hợp

Hệ thống tự động sinh ra các báo cáo đánh giá tổng hợp, tóm tắt tất cả các kết quả đánh giá. Dưới đây là một báo cáo ví dụ:

```markdown
# AIME Data Generation and Evaluation Comprehensive Report

## 1. Basic Information

- **Generation Time**: 2025-01-10 12:00:00
- **Number of Generated Problems**: 30
- **Reference AIME Year**: 2025

## 2. Data Generation Statistics

### Topic Distribution

| Topic | Quantity | Proportion |
|------|------|------|
| Algebra | 10 | 33.3% |
| Geometry | 8 | 26.7% |
| Number Theory | 7 | 23.3% |
| Combinatorics | 3 | 10.0% |
| Probability | 2 | 6.7% |

## 3. LLM Judge Evaluation Results

### Overall Score

- **Average Total Score**: 4.2/5.0
- **Pass Rate**: 85.0% (≥3.5 points)
- **Excellent Rate**: 40.0% (≥4.5 points)

### Dimension Scores

| Dimension | Average Score | Rating |
|------|--------|------|
| Correctness | 4.3/5.0 | Good ⭐⭐⭐⭐ |
| Clarity | 4.1/5.0 | Good ⭐⭐⭐⭐ |
| Difficulty Match | 4.0/5.0 | Good ⭐⭐⭐⭐ |
| Completeness | 4.4/5.0 | Good ⭐⭐⭐⭐ |

## 4. Win Rate Evaluation Results

### Win Rate Statistics

| Metric | Value | Percentage |
|------|------|--------|
| Generated Data Wins | 9 times | 45.0% |
| AIME Real Problems Win | 8 times | 40.0% |
| Tie | 3 times | 15.0% |

**Win Rate**: 45.0%

✅ **Good**: Generated data quality is close to reference data (gap <10%).

## 5. Comprehensive Conclusion

Based on the results of LLM Judge and Win Rate evaluation methods:

1. **LLM Judge Evaluation**: Average quality of generated data is **4.2/5.0**
2. **Win Rate Evaluation**: Win rate of generated data relative to AIME 2025 real problems is **45.0%**

✅ **Conclusion**: Generated data quality is **excellent**, reaching or exceeding AIME real problem level. Can be used for practical applications.

## 6. Improvement Suggestions

- ✅ Continue maintaining current generation strategy
- ✅ Can consider increasing generation quantity
- ✅ Recommend manual verification to ensure quality

## 7. Next Steps

1. **Manual Verification**: Run `python data_generation/human_verification_ui.py <data_path>` for manual verification
2. **View Detailed Results**:
   - LLM Judge detailed report
   - Win Rate detailed report
3. **Data Usage**: If quality is satisfactory, generated data can be used for training or testing
```

Dựa trên kinh nghiệm sử dụng thực tế, tóm tắt các nội dung sau:

Trong sinh dữ liệu, hãy sử dụng thời gian trễ phù hợp (2-3 giây) để tránh giới hạn tốc độ API, bật lưu checkpoint để tránh tổn thất do gián đoạn, đầu tiên kiểm thử với các lô nhỏ (10) để xác nhận không có vấn đề trước khi sinh dữ liệu quy mô lớn, và thường xuyên kiểm tra chất lượng sinh dữ liệu để điều chỉnh prompt kịp thời. Trong chiến lược đánh giá, khuyến nghị kết hợp các phương pháp LLM Judge và Win Rate, trong đó LLM Judge được dùng để đánh giá chất lượng tuyệt đối, Win Rate để so sánh chất lượng tương đối, và kiểm chứng thủ công để kiểm soát chất lượng cuối cùng. Đối với tiêu chuẩn chất lượng, khuyến nghị điểm trung bình LLM Judge trên 4.0/5.0, Win Rate trên 45% (gần 50%), tỷ lệ đạt trên 80%, và tỷ lệ đạt kiểm chứng thủ công trên 90%. Trong tối ưu hóa lặp, điều chỉnh prompt sinh dữ liệu dựa trên kết quả đánh giá, phân tích các vấn đề phổ biến trong các bài toán điểm thấp, tham khảo ưu điểm của các bài toán điểm cao, và liên tục cải thiện chiến lược sinh dữ liệu.

Qua việc học phần này, chúng ta đã nắm vững cách sử dụng framework HelloAgents để đánh giá chất lượng sinh dữ liệu, bao gồm ba phương pháp: đánh giá LLM Judge, đánh giá Win Rate, và kiểm chứng thủ công. Hệ thống đánh giá hoàn chỉnh này có thể đảm bảo chất lượng cao của dữ liệu được sinh ra, cung cấp hỗ trợ dữ liệu đáng tin cậy cho việc huấn luyện và kiểm thử hệ thống AI.

Đối với đánh giá LLM Judge và Win Rate, HelloAgents cũng đã tích hợp các công cụ và cung cấp mã ví dụ hoàn chỉnh. Nếu bạn quan tâm đến các chi tiết triển khai cụ thể của hai phương pháp đánh giá này, bạn cũng có thể tham khảo mã ví dụ.

## 12.5 Tổng kết Chương

Trong chương này, chúng ta đã xây dựng một hệ thống đánh giá hiệu năng hoàn chỉnh cho framework HelloAgents. Hãy cùng ôn lại các nội dung cốt lõi đã học:

**(1) Tổng quan Hệ thống Đánh giá**

Chúng ta đã thiết lập một hệ thống đánh giá ba tầng, bao phủ toàn diện các chiều năng lực khác nhau của agent. Đầu tiên là đánh giá năng lực gọi công cụ (BFCL), tập trung đánh giá độ chính xác gọi hàm của agent, bao gồm bốn loại simple, multiple, parallel, irrelevance, sử dụng công nghệ khớp AST để đánh giá chính xác. Thứ hai là đánh giá năng lực tổng quát (GAIA), đánh giá năng lực giải quyết vấn đề tổng hợp của agent, bao gồm ba cấp độ khó với 466 bài toán thế giới thực, tập trung vào suy luận nhiều bước, sử dụng công cụ, xử lý tệp và các năng lực khác. Thứ ba là đánh giá chất lượng sinh dữ liệu (AIME), đánh giá chất lượng dữ liệu do LLM sinh ra, sử dụng các phương pháp LLM Judge và Win Rate, hỗ trợ kiểm chứng thủ công và sinh báo cáo tổng hợp, đảm bảo dữ liệu được sinh ra đạt tiêu chuẩn chất lượng của dữ liệu tham chiếu.

**(2) Các Điểm Kỹ thuật Cốt lõi**

Trong triển khai kỹ thuật, chúng ta đã áp dụng sáu điểm kỹ thuật cốt lõi. Đầu tiên là thiết kế mô-đun hóa, hệ thống đánh giá áp dụng kiến trúc ba tầng: tầng dữ liệu (Dataset chịu trách nhiệm nạp và quản lý dữ liệu), tầng đánh giá (Evaluator chịu trách nhiệm thực thi quy trình đánh giá), và tầng chỉ số (Metrics chịu trách nhiệm tính toán các chỉ số đánh giá khác nhau). Thứ hai là đóng gói công cụ, tất cả các chức năng đánh giá được đóng gói dưới dạng Tools, có thể được agent gọi trực tiếp, tích hợp vào các workflow, hoặc sử dụng thông qua giao diện thống nhất. Thứ ba là công nghệ khớp AST, sử dụng khớp cây cú pháp trừu tượng cho các lời gọi hàm, thông minh hơn khớp chuỗi đơn giản, có khả năng bỏ qua thứ tự tham số, nhận diện các biểu thức tương đương, và bỏ qua sự khác biệt về định dạng. Thứ tư là hỗ trợ đa phương thức (multimodal), đánh giá GAIA hỗ trợ câu hỏi văn bản, tệp đính kèm, đầu vào hình ảnh và các dữ liệu đa phương thức khác. Thứ năm là đánh giá LLM Judge, sử dụng LLM làm giám khảo để đánh giá chất lượng dữ liệu được sinh ra, cung cấp chấm điểm đa chiều (tính đúng đắn, độ rõ ràng, mức độ khớp độ khó, tính đầy đủ), quy trình đánh giá tự động, các báo cáo đánh giá chi tiết, và hỗ trợ tùy chỉnh các chiều và tiêu chuẩn đánh giá. Thứ sáu là đánh giá so sánh Win Rate, đánh giá chất lượng sinh dữ liệu thông qua so sánh từng cặp (dữ liệu được sinh ra so với dữ liệu tham chiếu), LLM phán đoán bên nào tốt hơn và tính toán thống kê tỷ lệ thắng, gần 50% cho thấy chất lượng tương đương.

**(3) Các Hướng Mở rộng**

Dựa trên hệ thống đánh giá của chương này, bạn có thể mở rộng theo bốn hướng. Đầu tiên là thêm các benchmark đánh giá mới, có thể tham khảo các mẫu triển khai của BFCL và GAIA, triển khai ba thành phần Dataset, Evaluator, Metrics, và đóng gói dưới dạng Tool để sử dụng. Thứ hai là tùy chỉnh các chỉ số đánh giá, thêm các phương pháp tính toán chỉ số mới trong lớp Metrics, thiết kế các chỉ số theo các kịch bản ứng dụng cụ thể. Thứ ba là tích hợp vào quy trình CI/CD, tự động chạy đánh giá khi commit mã, thiết lập ngưỡng hiệu năng để ngăn suy giảm hiệu năng, sinh các báo cáo đánh giá và lưu trữ. Thứ tư là mở rộng đánh giá sinh dữ liệu, hỗ trợ nhiều loại dữ liệu hơn (mã, hội thoại, tài liệu, v.v.), thêm nhiều chiều đánh giá hơn (tính sáng tạo, tính đa dạng, v.v.), tích hợp nhiều tập dữ liệu tham chiếu hơn, hỗ trợ đánh giá so sánh đa mô hình.

**Chúc mừng bạn đã hoàn thành Chương 12!** 🎉

Đánh giá là một phần quan trọng trong phát triển agent, nó cho phép chúng ta:

- Đo lường khách quan năng lực của agent
- Phát hiện và khắc phục các vấn đề
- Liên tục cải thiện hệ thống

Trong chương tiếp theo, chúng ta sẽ khám phá cách áp dụng framework HelloAgents vào các dự án thực tế.

**Hãy tiếp tục cố gắng!** 💪

## Bài tập

> **Gợi ý**: Một số bài tập không có đáp án chuẩn, tập trung vào việc rèn luyện sự hiểu biết tổng hợp và năng lực thực hành của người học trong đánh giá hiệu năng agent.

1. Chương này đã giới thiệu nhiều benchmark đánh giá agent. Hãy phân tích:

   - Trong Mục 12.1.2, các benchmark đánh giá như BFCL, GAIA, AgentBench đã được giới thiệu. Hãy so sánh BFCL và GAIA: Chúng đánh giá những năng lực cốt lõi nào của agent tương ứng? Tại sao BFCL sử dụng thuật toán khớp AST trong khi GAIA sử dụng Quasi Exact Match? Ưu và nhược điểm của hai phương pháp đánh giá này là gì?
   - Giả sử bạn muốn xây dựng một "hệ thống chăm sóc khách hàng thông minh" cần đánh giá các năng lực sau: (1) độ chính xác của việc hiểu ý định người dùng; (2) tính đúng đắn của việc gọi các API backend; (3) sự thân thiện và tính chuyên nghiệp của các phản hồi; (4) tính bền vững (robustness) trong xử lý các tình huống ngoại lệ. Hãy chọn hoặc thiết kế các chỉ số và phương pháp đánh giá phù hợp cho từng năng lực.
   - Trong Mục 12.1.1, đã đề cập rằng đánh giá agent đối mặt với ba thách thức lớn: "tính bất định của đầu ra", "sự đa dạng của tiêu chuẩn đánh giá", và "chi phí đánh giá cao". Hãy đề xuất các giải pháp cụ thể cho từng thách thức và phân tích tính khả thi cũng như các hạn chế của các giải pháp.

2. BFCL (Berkeley Function Calling Leaderboard) là một benchmark quan trọng để đánh giá năng lực gọi công cụ. Dựa trên nội dung Mục 12.2, hãy suy nghĩ sâu:

   > **Gợi ý**: Đây là một câu hỏi thực hành, khuyến nghị thao tác thực tế

   - Trong thuật toán khớp AST ở Mục 12.2.3, chúng ta phán đoán liệu các lời gọi hàm có đúng hay không bằng cách so sánh các cây cú pháp trừu tượng. Hãy phân tích: Tại sao khớp AST phù hợp hơn khớp chuỗi đơn giản? Trong những tình huống nào khớp AST có thể tạo ra phán đoán sai (dương tính giả hoặc âm tính giả)? Làm thế nào để cải thiện thuật toán khớp AST để tăng độ chính xác?
   - Tập dữ liệu BFCL chứa bốn loại: simple, multiple, parallel, irrelevance. Hãy thiết kế 2-3 mẫu kiểm thử mới cho từng loại, yêu cầu có khả năng kiểm thử các trường hợp biên hoặc các kịch bản dễ gây lỗi dưới loại đó.
   - Hãy mở rộng bộ đánh giá BFCL dựa trên mã trong Mục 12.2.4, thêm các chức năng sau: (1) hỗ trợ đánh giá thứ tự thực thi của các lời gọi công cụ (đối với nhiều lời gọi công cụ có phụ thuộc); (2) đánh giá hiệu quả gọi công cụ (chẳng hạn liệu có sử dụng số lượng lời gọi tối thiểu hay không); (3) sinh báo cáo phân tích lỗi chi tiết (chẳng hạn loại lỗi nào phổ biến nhất).

3. GAIA (General AI Assistants) đánh giá năng lực tổng hợp của agent. Dựa trên nội dung Mục 12.3, hãy hoàn thành bài thực hành mở rộng sau:

   > **Gợi ý**: Đây là một câu hỏi thực hành, khuyến nghị thao tác thực tế

   - Trong Mục 12.3.2, ba cấp độ khó của GAIA (Level 1/2/3) đã được giới thiệu. Hãy phân tích: Sự khác biệt giữa ba cấp độ này về độ phức tạp nhiệm vụ, các năng lực yêu cầu, tiêu chuẩn đánh giá, v.v. là gì? Nếu thiết kế Level 4 (độ khó cực cao), nó nên bao gồm những loại nhiệm vụ nào?
   - GAIA sử dụng thuật toán "Quasi Exact Match" để đánh giá độ đúng của đáp án. Hãy phân tích: Phương pháp này xử lý sự đa dạng của đáp án như thế nào (chẳng hạn "42", "forty-two", "42.0" đều phải được coi là đúng)? Trong những tình huống nào quasi exact match có thể không đủ? Hãy thiết kế một thuật toán khớp đáp án thông minh hơn có khả năng xử lý các đáp án tương đương về mặt ngữ nghĩa.
   - Hãy triển khai một "tập đánh giá GAIA tùy chỉnh" dựa trên mã trong Mục 12.3.4: chọn một lĩnh vực cụ thể (chẳng hạn y tế, pháp lý, tài chính), thiết kế 10 câu hỏi thế giới thực, và triển khai quy trình đánh giá hoàn chỉnh. Yêu cầu các câu hỏi bao phủ các cấp độ khó khác nhau, và cung cấp đáp án chuẩn cùng các tiêu chí chấm điểm.

4. LLM Judge là một phương pháp mới nổi sử dụng các mô hình ngôn ngữ lớn để đánh giá. Dựa trên nội dung Mục 12.4, hãy phân tích sâu:

   - Trong Mục 12.4.2, chúng ta đã sử dụng GPT-4 làm giám khảo để đánh giá chất lượng phản hồi của agent. Hãy phân tích: LLM Judge có những ưu điểm gì so với khớp quy tắc truyền thống hoặc tính toán chỉ số? Nó có những thiên lệch hoặc hạn chế tiềm ẩn nào (chẳng hạn ưu tiên một số phong cách phản hồi nhất định, nhạy cảm với độ dài)?
   - Thiết kế tiêu chí chấm điểm của LLM Judge là rất quan trọng. Hãy thiết kế các tiêu chí chấm điểm chi tiết (bao gồm các chiều chấm điểm, trọng số, ví dụ) cho ba kịch bản đánh giá khác nhau sau: (1) đánh giá chất lượng sinh mã; (2) đánh giá chất lượng viết sáng tạo; (3) đánh giá chất lượng tài liệu kỹ thuật.
   - Trong Mục 12.4.3, đã đề cập rằng có thể sử dụng nhiều LLM Judge để đánh giá theo kiểu "hội đồng bồi thẩm" (jury-style). Hãy thiết kế một "hệ thống đánh giá đa giám khảo": sử dụng 3-5 LLM khác nhau (chẳng hạn GPT-4, Claude, Qwen) làm giám khảo, làm thế nào để tổng hợp điểm số của chúng? Làm thế nào để xử lý sự bất đồng giữa các giám khảo? Làm thế nào để phát hiện và lọc bỏ các điểm số bất thường?

5. Ứng dụng thực tế của đánh giá agent cần cân nhắc nhiều khía cạnh. Hãy suy nghĩ:

   - Trong các dự án thực tế, đánh giá thường cần cân bằng giữa "chi phí đánh giá" và "chất lượng đánh giá". Hãy thiết kế một "chiến lược đánh giá phân tầng": (1) đánh giá nhanh (chi phí thấp, cho việc lặp phát triển hàng ngày); (2) đánh giá tiêu chuẩn (chi phí trung bình, cho giai đoạn trước phát hành); (3) đánh giá toàn diện (chi phí cao, cho các bản cập nhật lớn hoặc phát hành công khai). Mỗi tầng nên bao gồm những mục đánh giá nào? Làm thế nào để thiết kế quy trình đánh giá?
   - Hiệu năng của agent có thể thay đổi theo thời gian (chẳng hạn sự thay đổi của các API bên ngoài phụ thuộc, sự thay đổi nhu cầu người dùng). Hãy thiết kế một "hệ thống đánh giá liên tục": có khả năng định kỳ tự động chạy đánh giá, giám sát xu hướng thay đổi hiệu năng của agent, và cảnh báo kịp thời khi hiệu năng suy giảm. Hệ thống này nên bao gồm những thành phần nào? Làm thế nào để thiết kế các quy tắc cảnh báo?
   - Kết quả đánh giá cần được trình bày rõ ràng cho các đối tượng khác nhau (chẳng hạn nhà phát triển, quản lý sản phẩm, người dùng). Hãy thiết kế một "hệ thống sinh báo cáo đánh giá": có khả năng tự động sinh các báo cáo với các mức độ chi tiết khác nhau dựa trên loại đối tượng. Báo cáo cho nhà phát triển nên bao gồm những chi tiết kỹ thuật nào? Báo cáo cho quản lý sản phẩm nên làm nổi bật những chỉ số nghiệp vụ nào? Báo cáo cho người dùng nên được đơn giản hóa và trực quan hóa như thế nào?

## Tài liệu tham khảo

[1] Patil, S. G., Zhang, T., Wang, X., & Gonzalez, J. E. (2023). Gorilla: Large Language Model Connected with Massive APIs. arXiv preprint arXiv:2305.15334.

[2] Qin, Y., Liang, S., Ye, Y., Zhu, K., Yan, L., Lu, Y., ... & Sun, M. (2023). ToolLLM: Facilitating Large Language Models to Master 16000+ Real-world APIs. arXiv preprint arXiv:2307.16789.

[3] Li, M., Zhao, Y., Yu, B., Song, F., Li, H., Yu, H., ... & Li, Y. (2023). Api-bank: A comprehensive benchmark for tool-augmented llms. arXiv preprint arXiv:2304.08244.

[4] Mialon, G., Dessì, R., Lomeli, M., Nalmpantis, C., Pasunuru, R., Raileanu, R., ... & Scialom, T. (2023). GAIA: a benchmark for General AI Assistants. arXiv preprint arXiv:2311.12983.

[5] Liu, X., Yu, H., Zhang, H., Xu, Y., Lei, X., Lai, H., ... & Zhang, D. (2023). AgentBench: Evaluating LLMs as Agents. arXiv preprint arXiv:2308.03688.

[6] Zhou, S., Xu, F. F., Zhu, H., Zhou, X., Lo, R., Sridhar, A., ... & Neubig, G. (2023). WebArena: A Realistic Web Environment for Building Autonomous Agents. arXiv preprint arXiv:2307.13854.

[7] Chan, C. M., Chen, W., Su, Y., Yu, J., Xue, W., Zhang, S., ... & Liu, Z. (2023). ChatEval: Towards Better LLM-based Evaluators through Multi-Agent Debate. arXiv preprint arXiv:2308.07201.

[8] Zhou, X., Zhu, H., Mathur, L., Zhang, R., Yu, H., Qi, Z., ... & Neubig, G. (2023). SOTOPIA: Interactive Evaluation for Social Intelligence in Language Agents. arXiv preprint arXiv:2310.11667.

[9] Mathematical Association of America. (2024). American Invitational Mathematics Examination (AIME). Retrieved from https://www.maa.org/math-competitions/invitational-competitions/aime

