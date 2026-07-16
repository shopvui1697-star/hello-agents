# Chương 14 Agent Nghiên cứu Chuyên sâu Tự động

Trong dự án trợ lý du lịch ở Chương 13, chúng ta đã trải nghiệm cách ứng dụng HelloAgents vào một sản phẩm multi-agent. Trong chương này, chúng ta tiếp tục tiến lên, tập trung vào **các ứng dụng thiên về tri thức (knowledge-intensive)**: **xây dựng một trợ lý agent có thể tự động thực thi các tác vụ nghiên cứu chuyên sâu.**

So với việc lập kế hoạch du lịch, độ khó của nghiên cứu chuyên sâu nằm ở sự phân kỳ liên tục của thông tin, sự cập nhật nhanh chóng của các dữ kiện, và yêu cầu cao của người dùng đối với nguồn trích dẫn. Để tạo ra các báo cáo nghiên cứu đáng tin cậy, chúng ta cần trang bị cho agent ba năng lực cốt lõi:

**(1) Phân tích Vấn đề**: Phân rã các chủ đề mở của người dùng thành các câu truy vấn có thể tìm kiếm được.

**(2) Thu thập Thông tin Nhiều Vòng**: Liên tục khai thác tài liệu bằng cách kết hợp các search API khác nhau, đồng thời khử trùng lặp và tích hợp chúng.

**(3) Phản chiếu và Tổng hợp**: Xác định các lỗ hổng tri thức dựa trên kết quả từng giai đoạn, quyết định có nên tiếp tục tìm kiếm hay không, và tạo ra các bản tóm tắt có cấu trúc.

## 14.1 Tổng quan Dự án và Thiết kế Kiến trúc

### 14.1.1 Tại sao Chúng ta Cần một Trợ lý Nghiên cứu Chuyên sâu

Trong kỷ nguyên bùng nổ thông tin, mỗi ngày chúng ta đều cần nhanh chóng hiểu các công nghệ, khái niệm hoặc sự kiện mới. Các phương pháp nghiên cứu truyền thống có một số điểm nhức nhối. Đầu tiên là **quá tải thông tin**. Các công cụ tìm kiếm trả về hàng nghìn kết quả, và bạn phải nhấp vào từng liên kết một, đọc rất nhiều nội dung để tìm ra thông tin hữu ích. Thứ hai là **thiếu cấu trúc**. Ngay cả khi bạn tìm được thông tin liên quan, thông tin đó thường rời rạc và thiếu sự tổ chức có hệ thống. Cuối cùng là **lao động lặp đi lặp lại**. Mỗi khi nghiên cứu một chủ đề mới, bạn phải lặp lại quy trình "tìm kiếm → đọc → tóm tắt → tổ chức".

Đây chính là vấn đề mà trợ lý nghiên cứu chuyên sâu cần giải quyết. Nó không chỉ là một công cụ tìm kiếm, mà là một trợ lý nghiên cứu có thể tự chủ lập kế hoạch, thực thi và tổng hợp.

**Giá trị Cốt lõi của Trợ lý Nghiên cứu Chuyên sâu:**

1. **Tiết kiệm Thời gian**: Nén 1-2 giờ công việc nghiên cứu xuống còn 5-10 phút
2. **Nâng cao Chất lượng**: Quy trình nghiên cứu có hệ thống, tránh bỏ sót thông tin quan trọng
3. **Có thể Truy vết**: Ghi lại tất cả kết quả tìm kiếm và nguồn để dễ dàng kiểm chứng và trích dẫn
4. **Có thể Mở rộng**: Dễ dàng thêm các công cụ tìm kiếm, nguồn dữ liệu và công cụ phân tích mới

### 14.1.2 Tổng quan Kiến trúc Kỹ thuật

Hệ thống này vẫn áp dụng **kiến trúc tách biệt front-end và back-end** cổ điển, như minh họa trong Hình 14.1.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-1.png" alt="" width="85%"/>
  <p>Hình 14.1 Kiến trúc Kỹ thuật của Trợ lý Nghiên cứu Chuyên sâu</p>
</div>

Hệ thống được thiết kế theo kiến trúc bốn tầng:

**Tầng Front-End (Vue3+TypeScript)**: Giao diện hộp thoại modal toàn màn hình, trực quan hóa kết quả bằng Markdown

**Tầng Back-End (FastAPI)**: Định tuyến API (`/research/stream`)

**Tầng Agent (HelloAgents)**: Ba Agent chuyên biệt (TODO Planner, Task Summarizer, Report Writer) + Hai công cụ cốt lõi (SearchTool, NoteTool)

**Tầng Dịch vụ Bên ngoài**: Các công cụ tìm kiếm + các nhà cung cấp LLM

Hãy xem một yêu cầu nghiên cứu hoàn chỉnh chảy qua hệ thống như thế nào, như minh họa trong Hình 14.2:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-2.png" alt="" width="85%"/>
  <p>Hình 14.2 Quy trình Luồng Dữ liệu của Trợ lý Nghiên cứu Chuyên sâu</p>
</div>

1. **Người dùng Nhập liệu**: Người dùng nhập chủ đề nghiên cứu ở front-end
2. **Front-End Gửi**: Front-end kết nối tới `/research/stream` qua SSE
3. **Back-End Nhận**: FastAPI nhận yêu cầu, tạo trạng thái nghiên cứu
4. **Giai đoạn Lập kế hoạch**: Gọi Agent lập kế hoạch nghiên cứu, phân rã thành 3 tác vụ con
5. **Giai đoạn Thực thi**: Thực thi lần lượt từng tác vụ con
   - Dùng SearchTool để tìm kiếm
   - Gọi Agent tóm tắt tác vụ để tóm tắt
   - Dùng NoteTool để ghi lại kết quả
6. **Giai đoạn Báo cáo**: Gọi Agent tạo báo cáo, tích hợp tất cả các bản tóm tắt
7. **Trả về Theo luồng**: Đẩy tiến độ và kết quả về front-end qua SSE
8. **Front-End Hiển thị**: Front-end cập nhật trạng thái tác vụ, thanh tiến độ, nhật ký và báo cáo theo thời gian thực

Cấu trúc thư mục của dự án như sau:

```
helloagents-deepresearch/
├── backend/                    # Back-end code
│   ├── src/
│   │   ├── agent.py           # Core coordinator
│   │   ├── main.py            # FastAPI entry
│   │   ├── models.py          # Data models
│   │   ├── prompts.py         # Prompt templates
│   │   ├── config.py          # Configuration management
│   │   └── services/          # Service layer
│   │       ├── planner.py     # Planning service
│   │       ├── summarizer.py  # Summarization service
│   │       ├── reporter.py    # Report service
│   │       └── search.py      # Search service
│   ├── .env                   # Environment variables
│   ├── pyproject.toml         # Dependency management
│   └── workspace/             # Research notes
│
└── frontend/                   # Front-end code
    ├── src/
    │   ├── App.vue            # Main component
    │   ├── components/        # UI components
    │   │   └── ResearchModal.vue
    │   └── composables/       # Composable functions
    │       └── useResearch.ts
    ├── package.json           # npm dependencies
    └── vite.config.ts         # Build configuration
```

### 14.1.3 Trải nghiệm Nhanh: Chạy Dự án trong 5 Phút

Trước khi đi sâu vào các chi tiết triển khai, hãy chạy dự án trước để xem kết quả cuối cùng. Bằng cách này bạn sẽ có được hiểu biết trực quan về toàn bộ hệ thống.

Bạn có thể kiểm tra phiên bản bằng các lệnh sau:

```bash
python --version  # Should show Python 3.10.x or higher
node --version    # Should show v16.x.x or higher
npm --version     # Should show 8.x.x or higher
```

**(1) Khởi động Back-End**

```bash
# 1. Enter back-end directory
cd helloagents-deepresearch/backend

# 2. Install dependencies
# Method 1: Using uv (recommended, faster Python package manager)
uv sync

# Method 2: Using pip
pip install -e .

# 3. Configure environment variables
cp .env.example .env

# 4. Edit .env file, fill in your API keys
# Open .env file with your favorite editor
# At minimum, configure:
# - LLM_PROVIDER (e.g., openai, deepseek, qwen)
# - LLM_API_KEY (your LLM API key)
# - SEARCH_API (e.g., duckduckgo, tavily)

# 5. Start back-end
python src/main.py
```

Nếu mọi thứ bình thường, bạn sẽ thấy output tương tự như:

```
INFO:     Started server process [12345]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

**(2) Khởi động Front-End**

Mở một cửa sổ terminal mới:

```bash
# 1. Enter front-end directory
cd helloagents-deepresearch/frontend

# 2. Install dependencies
npm install

# 3. Start front-end
npm run dev
```

Nếu mọi thứ bình thường, bạn sẽ thấy output tương tự như:

```
  VITE v5.0.0  ready in 500 ms

  ➜  Local:   http://localhost:5174/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

**(3) Bắt đầu Nghiên cứu**

Mở trình duyệt và truy cập `http://localhost:5174`. Bạn sẽ thấy một thẻ nhập liệu (input card) được căn giữa, như minh họa trong Hình 14.3. Nhập một chủ đề nghiên cứu, ví dụ `What kind of organization is Datawhale?`, chọn một công cụ tìm kiếm (nếu đã cấu hình nhiều công cụ), và nhấp vào nút "Start Research".

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-3.png" alt="" width="85%"/>
  <p>Hình 14.3 Trang Tìm kiếm của Trợ lý Nghiên cứu Chuyên sâu</p>
</div>

Như minh họa trong Hình 14.4, hệ thống sẽ tự động mở rộng ra toàn màn hình, với thông tin nghiên cứu hiển thị bên trái và tiến độ cùng kết quả nghiên cứu hiển thị theo thời gian thực bên phải. Toàn bộ quá trình nghiên cứu mất khoảng 1-3 phút, tùy thuộc vào độ phức tạp của chủ đề và tốc độ phản hồi của công cụ tìm kiếm.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-4.png" alt="" width="85%"/>
  <p>Hình 14.4 Trợ lý Nghiên cứu Chuyên sâu ở Chế độ Mở rộng</p>
</div>

Sau khi nghiên cứu hoàn tất, bạn sẽ thấy:

- **Danh sách Tác vụ**: Hiển thị tất cả các tác vụ con và trạng thái của chúng
- **Nhật ký Tiến độ**: Hiển thị tất cả các thao tác trong quá trình nghiên cứu
- **Báo cáo Cuối cùng**: Báo cáo Markdown có cấu trúc chứa các bản tóm tắt của tất cả các tác vụ con và trích dẫn nguồn

Bây giờ bạn đã chạy thành công trợ lý nghiên cứu chuyên sâu và có được hiểu biết trực quan về hệ thống.

## 14.2 Mô hình Nghiên cứu Hướng TODO (TODO-Driven)

### 14.2.1 Nghiên cứu Hướng TODO là gì

Các công cụ tìm kiếm truyền thống chỉ có thể trả lời các câu hỏi đơn lẻ, trong khi nghiên cứu chuyên sâu cần trả lời một loạt các câu hỏi liên quan. Mô hình nghiên cứu hướng TODO phân rã các chủ đề nghiên cứu phức tạp thành nhiều tác vụ con (TODO), thực thi từng cái một, và tích hợp các kết quả.

Ý tưởng cốt lõi của mô hình này là: **Chuyển đổi tác vụ phức tạp "nghiên cứu" thành một quy trình "lập kế hoạch → thực thi → tích hợp"**.

Hãy hiểu sự chuyển đổi này qua một ví dụ. Giả sử bạn muốn nghiên cứu "What kind of organization is Datawhale?". Phương pháp tìm kiếm truyền thống là:

```
User input: What kind of organization is Datawhale?
Search engine: Returns 10-20 links
User: Click on links one by one, read content, take notes
Result: Fragmented information, lacking systematization
```

Vấn đề của cách tiếp cận này là mỗi liên kết chỉ bao phủ một khía cạnh của chủ đề, thiếu cấu trúc có hệ thống, và đòi hỏi phải tổ chức và tóm tắt thủ công.

**Cách tiếp cận Hướng TODO: Nghiên cứu có Hệ thống**

```
User input: What kind of organization is Datawhale?

System planning:
  ├─ TODO 1: Basic information about Datawhale (organizational positioning)
  ├─ TODO 2: Main projects of Datawhale (core content)
  ├─ TODO 3: Community culture of Datawhale (values)
  └─ TODO 4: Influence of Datawhale (social contribution)

System execution:
  For each TODO:
    1. Search for relevant materials
    2. Summarize key information
    3. Record source citations

System integration:
  Generate structured report:
    ├─ Part 1: Organizational positioning (from TODO 1)
    ├─ Part 2: Core content (from TODO 2)
    ├─ Part 3: Values (from TODO 3)
    ├─ Part 4: Social contribution (from TODO 4)
    └─ References: All source citations
```

Ưu điểm của cách tiếp cận này là nó phân rã các chủ đề phức tạp thành các câu hỏi con rõ ràng, ghi lại kết quả tìm kiếm và bản tóm tắt cho từng tác vụ con để dễ dàng truy vết, và quy trình nghiên cứu có hệ thống giúp tránh bỏ sót thông tin quan trọng. Nó cũng dễ dàng thêm các tác vụ con mới hoặc điều chỉnh thứ tự thực thi.

Một hệ thống nghiên cứu hướng TODO hoàn chỉnh chứa ba yếu tố cốt lõi:

**(1) Bộ lập kế hoạch Thông minh (TODO Planner)**: Chịu trách nhiệm phân rã các chủ đề nghiên cứu thành các tác vụ con. Một bộ lập kế hoạch tốt cần hiểu các khía cạnh chính và mục tiêu nghiên cứu của chủ đề, phân rã chủ đề thành 3-5 tác vụ con (quá ít sẽ không bao phủ hết, quá nhiều sẽ dư thừa), và thiết kế các truy vấn tìm kiếm phù hợp cho từng tác vụ con.

**(2) Bộ thực thi Tác vụ (Task Executor)**: Chịu trách nhiệm thực thi từng tác vụ con. Bộ thực thi cần dùng công cụ tìm kiếm để lấy tài liệu liên quan, trích xuất thông tin then chốt và loại bỏ nội dung dư thừa, đồng thời lưu tất cả trích dẫn nguồn để dễ dàng kiểm chứng.

**(3) Người viết Báo cáo (Report Writer)**: Chịu trách nhiệm tích hợp kết quả của tất cả các tác vụ con. Bộ tạo báo cáo cần tổ chức nội dung theo trình tự logic, hợp nhất thông tin trùng lặp, và thêm trích dẫn nguồn cho từng luận điểm.

Trong trường hợp của chúng ta, quy trình nghiên cứu hướng TODO được minh họa trong Hình 14.5:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-5.png" alt="" width="85%"/>
  <p>Hình 14.5 Quy trình Nghiên cứu Hướng TODO</p>
</div>

Toàn bộ quy trình là tuyến tính, nhưng mỗi giai đoạn có đầu vào và đầu ra rõ ràng. Thiết kế này giúp hệ thống dễ hiểu và dễ debug.

### 14.2.2 Quy trình Nghiên cứu Ba Giai đoạn

Quy trình nghiên cứu hướng TODO được chia thành ba giai đoạn: Lập kế hoạch, Thực thi và Báo cáo. Mỗi giai đoạn có một Agent chuyên trách.

**(1) Giai đoạn 1: Lập kế hoạch (Planning)**

Mục tiêu của giai đoạn lập kế hoạch là phân rã chủ đề nghiên cứu thành 3-5 tác vụ con. Hệ thống nhận chủ đề nghiên cứu và ngày hiện tại làm đầu vào, và xuất ra một danh sách tác vụ con ở định dạng JSON. Mỗi tác vụ con chứa ba trường: title (tiêu đề tác vụ), intent (ý định nghiên cứu), và query (truy vấn tìm kiếm).

Agent lập kế hoạch nghiên cứu áp dụng các chiến lược phân rã khác nhau dựa trên đặc điểm chủ đề, thường bắt đầu từ các khái niệm cơ bản, sau đó tìm hiểu hiện trạng kỹ thuật, ứng dụng thực tế và xu hướng phát triển, và tiến hành phân tích so sánh khi cần thiết. Ví dụ, đối với "What kind of organization is Datawhale?", Agent lập kế hoạch có thể tạo ra các tác vụ con sau:

```json
[
  {
    "title": "Basic information about Datawhale",
    "intent": "Understand Datawhale's organizational positioning, founding time, development history",
    "query": "Datawhale organization introduction history 2024"
  },
  {
    "title": "Main projects of Datawhale",
    "intent": "Understand Datawhale's core open source projects and tutorials",
    "query": "Datawhale projects tutorials open source 2024"
  },
  ...
]
```

Một kế hoạch tốt nên toàn diện, logic rõ ràng, có truy vấn chính xác, và số lượng mục hợp lý.

**(2) Giai đoạn 2: Thực thi (Execution)**

Giai đoạn thực thi thực hiện từng tác vụ con một, tìm kiếm và tóm tắt tài liệu liên quan. Hệ thống nhận danh sách tác vụ con và cấu hình công cụ tìm kiếm làm đầu vào, và xuất ra một bản tóm tắt (định dạng Markdown) cùng danh sách trích dẫn nguồn cho mỗi tác vụ con. Quy trình thực thi như sau:

Đối với mỗi tác vụ con, bộ thực thi sẽ:

1. **Tìm kiếm tài liệu**: Dùng công cụ tìm kiếm đã cấu hình để thực thi việc tìm kiếm

   ```python
   search_results = search_tool.run({
       "input": task.query,
       "backend": "tavily",
       "mode": "structured",
       "max_results": 5
   })
   ```

2. **Lấy kết quả tìm kiếm**: Trích xuất title, URL, snippet

   ```json
   {
     "results": [
       {
         "title": "What is a Multimodal Model?",
         "url": "https://example.com/multimodal-model",
         "snippet": "A multimodal model is an AI model that can process multiple types of data..."
       },
       ...
     ]
   }
   ```

3. **Gọi Agent tóm tắt**: Tóm tắt kết quả tìm kiếm

   ```python
   summary = summarizer_agent.run(
       task=task,
       search_results=search_results
   )
   ```

4. **Ghi lại bản tóm tắt và nguồn**: Lưu vào NoteTool

   ```python
   note_tool.run({
       "action": "create",
       "title": task.title,
       "content": f"## {task.title}\n\n{summary}\n\n## Sources\n{sources}",
       "tags": ["research", "summary"]
   })
   ```

Agent tóm tắt tác vụ sẽ trích xuất các luận điểm cốt lõi từ mỗi kết quả tìm kiếm, hợp nhất các thông tin tương tự, giữ lại các con số, ngày tháng, tên gọi và các dữ liệu then chốt khác, và thêm trích dẫn nguồn cho từng luận điểm. Ví dụ, đối với kết quả tìm kiếm của "Basic information about Datawhale", Agent tóm tắt có thể tạo ra:

```markdown
## Basic Information about Datawhale

Datawhale is an open source organization focused on data science and AI, founded in 2018[1]. The organization's core mission is "for the learner, grow together with learners", committed to building a pure learning community[2].

**Core Positioning:**

1. **Open Source Education Platform**: Provides high-quality AI and data science learning resources[1]
2. **Learner Community**: Gathers tens of thousands of AI learners and practitioners[3]
3. **Knowledge Sharing**: Advocates open source spirit, all content is completely free and open[2]

**Development History:**

- **2018**: Datawhale was founded, released first open source tutorial[1]
- **2020**: Became one of the leading AI learning communities in China[3]
- **2024**: Released 50+ open source projects, impacting 100,000+ learners[4]

## Sources

[1] https://github.com/datawhalechina
[2] https://datawhale.club/about
[3] https://www.zhihu.com/org/datawhale
[4] https://datawhale.cn
```

Trong quá trình thực thi, hệ thống sẽ đẩy thông tin tiến độ về front-end theo thời gian thực:

```json
{
  "type": "status",
  "message": "Searching: Basic information about Datawhale"
}
```

```json
{
  "type": "status",
  "message": "Summarizing search results..."
}
```

```json
{
  "type": "task",
  "task": {
    "id": 1,
    "title": "Basic information about Datawhale",
    "status": "completed"
  }
}
```

**(3) Giai đoạn 3: Báo cáo (Reporting)**

Mục tiêu của giai đoạn báo cáo là tích hợp các bản tóm tắt của tất cả các tác vụ con và tạo ra báo cáo cuối cùng. Hệ thống nhận các bản tóm tắt của tất cả các tác vụ con và chủ đề nghiên cứu làm đầu vào, và xuất ra báo cáo cuối cùng ở định dạng Markdown. Báo cáo chứa năm phần: tiêu đề, tổng quan, phân tích chi tiết từng tác vụ con, tóm tắt, và tài liệu tham khảo. Ví dụ, đối với "What kind of organization is Datawhale?", báo cáo cuối cùng có thể là:

```markdown
# What Kind of Organization is Datawhale?

## Overview

This report systematically researched the open source organization Datawhale, covering four aspects: basic information, main projects, community culture, and influence.

## 1. Basic Information about Datawhale

Datawhale is an open source organization focused on data science and AI, founded in 2018...

(Insert summary of subtask 1 here)

## 2. Main Projects of Datawhale

Datawhale has released multiple high-quality open source tutorials, including Hello-Agents, Joyful-Pandas, etc...

(Insert summary of subtask 2 here)
...
## Summary

Through this research, we learned about Datawhale's organizational positioning, core projects, community culture, and social contributions. Datawhale is a pure learning community that has made important contributions to AI education.

## References

[1] https://github.com/datawhalechina
[2] https://datawhale.club/about
...
```

Agent tạo báo cáo sẽ tổ chức nội dung theo trình tự logic của các tác vụ con, thêm một phần tổng quan ngắn gọn ở đầu, hợp nhất thông tin trùng lặp, thống nhất định dạng Markdown, và tổ chức tất cả các trích dẫn nguồn vào phần tài liệu tham khảo.

## 14.3 Thiết kế Hệ thống Agent

### 14.3.1 Phân chia Trách nhiệm của Agent

Trong trợ lý nghiên cứu chuyên sâu, chúng ta đã thiết kế ba Agent chuyên biệt, mỗi Agent chịu trách nhiệm cho một tác vụ cụ thể. Điều này giúp mỗi Agent trở nên đơn giản, dễ hiểu và dễ bảo trì.

Trong Chương 7, chúng ta đã học cách dùng `SimpleAgent` để xây dựng agent. Triết lý thiết kế của `SimpleAgent` đơn giản và trực tiếp: mỗi khi phương thức `run()` được gọi, Agent phân tích câu hỏi của người dùng, quyết định có gọi công cụ hay không, rồi trả về kết quả. Thiết kế này rất hiệu quả khi xử lý các tác vụ đơn giản, nhưng khi đối mặt với các tác vụ phức tạp như nghiên cứu chuyên sâu, chúng ta cần tiếp tục sử dụng cách tiếp cận hợp tác multi-agent.

Như minh họa trong Bảng 14.1, ba Agent lần lượt chịu trách nhiệm về lập kế hoạch, tóm tắt, và tạo báo cáo.

<div align="center">
  <p>Bảng 14.1 Phân chia Trách nhiệm của Ba Agent</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-table-1.png" alt="" width="85%"/>
</div>

Hãy giới thiệu chi tiết thiết kế của từng Agent.

**Agent 1: Chuyên gia Lập kế hoạch Nghiên cứu (TODO Planner)**

**Trách nhiệm**: Phân rã chủ đề nghiên cứu thành 3-5 tác vụ con

**Triết lý Thiết kế**: Tác vụ cốt lõi của chuyên gia lập kế hoạch nghiên cứu là hiểu chủ đề nghiên cứu của người dùng, phân tích các khía cạnh chính của chủ đề, rồi tạo ra một loạt các tác vụ con. Quá trình này tương tự như giai đoạn "brainstorming" (động não) của các nhà nghiên cứu con người trước khi bắt đầu nghiên cứu.

**Thiết kế Prompt**:

```python
todo_planner_instructions = """
You are a research planning expert. Your task is to decompose the user's research topic into 3-5 subtasks.

Current date: {current_date}

Research topic: {research_topic}

Please analyze this research topic and decompose it into 3-5 subtasks. Each subtask should:
1. Cover an important aspect of the topic
2. Have a clear research objective
3. Be able to find relevant materials through search engines

Please return the subtask list in JSON format, each subtask containing:
- title: Task title (concise and clear)
- intent: Task intent (why research this)
- query: Search query (query string for search engines, can use English for better search results)

Example output:
[
  {{
    "title": "What is a multimodal model",
    "intent": "Understand the basic concepts of multimodal models to lay the foundation for subsequent research",
    "query": "multimodal model definition concept 2024"
  }},
  ...
]

Please ensure:
1. Number of subtasks is between 3-5
2. Subtasks have logical relationships (e.g., from basics to applications, from current status to trends)
3. Search queries can accurately find relevant materials
4. Only return JSON, do not include other text
"""
```

**Các Điểm Thiết kế Then chốt**: Prompt bao gồm ngày hiện tại để lấy thông tin mới nhất, yêu cầu rõ ràng output ở định dạng JSON để dễ phân tích, giúp Agent hiểu output mong đợi thông qua các ví dụ, và nhấn mạnh các ràng buộc như số lượng tác vụ con và các mối quan hệ logic.

**Mã Triển khai**:

ToolAwareSimpleAgent ở đây là một phần mở rộng của SimpleAgent. Bạn có thể tìm hiểu về nó trong Mục 14.3.2, không cần đi sâu vào ở đây.

```python
class PlanningService:
    def __init__(self, llm: HelloAgentsLLM):
        self._agent = ToolAwareSimpleAgent(
            name="TODO Planner",
            system_prompt="You are a research planning expert",
            llm=llm,
            tool_call_listener=self._on_tool_call
        )

    def plan_todo_list(self, state: SummaryState) -> List[TodoItem]:
        prompt = todo_planner_instructions.format(
            current_date=get_current_date(),
            research_topic=state.research_topic,
        )

        response = self._agent.run(prompt)
        tasks_payload = self._extract_tasks(response)

        todo_items = []
        for idx, item in enumerate(tasks_payload, start=1):
            task = TodoItem(
                id=idx,
                title=item["title"],
                intent=item["intent"],
                query=item["query"],
            )
            todo_items.append(task)

        return todo_items

    def _extract_tasks(self, response: str) -> List[dict]:
        """Extract JSON from Agent response"""
        # Use regex to extract JSON part
        json_match = re.search(r'\[.*\]', response, re.DOTALL)
        if json_match:
            json_str = json_match.group(0)
            return json.loads(json_str)
        else:
            raise ValueError("Unable to extract JSON from response")
```

**Agent 2: Chuyên gia Tóm tắt Tác vụ (Task Summarizer)**

**Trách nhiệm**: Tóm tắt kết quả tìm kiếm, trích xuất thông tin then chốt

**Triết lý Thiết kế**: Tác vụ cốt lõi của chuyên gia tóm tắt tác vụ là đọc kết quả tìm kiếm, trích xuất thông tin then chốt, và trình bày nó theo cách có cấu trúc. Quá trình này tương tự như việc các nhà nghiên cứu con người ghi chú sau khi đọc tài liệu.

**Thiết kế Prompt**:

```python
task_summarizer_instructions = """
You are a task summarization expert. Your task is to summarize search results and extract key information.

Task title: {task_title}
Task intent: {task_intent}
Search query: {task_query}

Search results:
{search_results}

Please carefully read the above search results, extract key information, and return a summary in Markdown format.

The summary should include:
1. **Core Viewpoints**: Core viewpoints and conclusions from search results
2. **Key Data**: Important numbers, dates, names, etc.
3. **Source Citations**: Add source citations for each viewpoint (using [1], [2], etc.)

Please ensure:
1. Summary is concise and clear, avoiding redundancy
2. Retain important details and data
3. Add source citations for each viewpoint
4. Use Markdown format (headings, lists, bold, etc.)

Example output:
## Core Viewpoints

Multimodal models are AI models that can process multiple types of data[1]. Unlike traditional unimodal models, multimodal models can simultaneously understand text, images, audio, etc.[2].

**Key Features:**
- Cross-modal understanding[1]
- Unified representation[3]
- End-to-end training[2]

## Sources

[1] https://example.com/source1
[2] https://example.com/source2
[3] https://example.com/source3
"""
```

**Các Điểm Thiết kế Then chốt**: Prompt bao gồm tiêu đề tác vụ, ý định, truy vấn và các ngữ cảnh khác để giúp Agent hiểu tác vụ, yêu cầu rõ ràng output phải bao gồm luận điểm cốt lõi, dữ liệu then chốt, và trích dẫn nguồn, nhấn mạnh việc thêm trích dẫn nguồn cho từng luận điểm, và giúp Agent hiểu định dạng output mong đợi thông qua các ví dụ.

**Mã Triển khai**:

```python
class SummarizationService:
    def __init__(self, llm: HelloAgentsLLM):
        self._agent = ToolAwareSimpleAgent(
            name="Task Summarizer",
            system_prompt="You are a task summarization expert",
            llm=llm,
            tool_call_listener=self._on_tool_call
        )

    def summarize_task(
        self,
        task: TodoItem,
        search_results: List[dict]
    ) -> str:
        # Format search results
        formatted_sources = self._format_sources(search_results)

        prompt = task_summarizer_instructions.format(
            task_title=task.title,
            task_intent=task.intent,
            task_query=task.query,
            search_results=formatted_sources,
        )

        summary = self._agent.run(prompt)
        return summary

    def _format_sources(self, search_results: List[dict]) -> str:
        """Format search results"""
        formatted = []
        for idx, result in enumerate(search_results, start=1):
            formatted.append(
                f"[{idx}] {result['title']}\n"
                f"URL: {result['url']}\n"
                f"Snippet: {result['snippet']}\n"
            )
        return "\n".join(formatted)
```

**Agent 3: Chuyên gia Viết Báo cáo (Report Writer)**

**Trách nhiệm**: Tích hợp các bản tóm tắt của tất cả các tác vụ con và tạo báo cáo cuối cùng

**Triết lý Thiết kế**: Tác vụ cốt lõi của chuyên gia viết báo cáo là tích hợp các bản tóm tắt của tất cả các tác vụ con thành một báo cáo có cấu trúc. Quá trình này tương tự như việc các nhà nghiên cứu con người viết báo cáo nghiên cứu sau khi hoàn thành tất cả các cuộc điều tra.

**Thiết kế Prompt**:

```python
report_writer_instructions = """
You are a report writing expert. Your task is to integrate the summaries of all subtasks and generate a structured research report.

Research topic: {research_topic}

Subtask summaries:
{task_summaries}

Please integrate all the above subtask summaries and generate a structured research report.

The report should include:
1. **Title**: Research topic
2. **Overview**: Briefly introduce the research topic and report structure (2-3 paragraphs)
3. **Detailed Analysis of Each Subtask**: Organize in logical order (using level-2 headings)
4. **Summary**: Summarize the main findings of the research (1-2 paragraphs)
5. **References**: All source citations (grouped by subtask)

Please ensure:
1. Report structure is clear and logically coherent
2. Eliminate duplicate information
3. Retain all source citations
4. Use Markdown format

Example output:
# Latest Advances in Multimodal Large Models

## Overview

This report systematically researched the latest advances in multimodal large models...

## 1. What is a Multimodal Model

(Insert summary of subtask 1 here)

## 2. What are the Latest Multimodal Models

(Insert summary of subtask 2 here)

...

## Summary

Through this research, we learned about...

## References

### Task 1: What is a Multimodal Model
[1] https://example.com/source1
...
"""
```

**Các Điểm Thiết kế Then chốt**: Prompt yêu cầu rõ ràng báo cáo phải bao gồm tiêu đề, tổng quan, phân tích chi tiết, tóm tắt, tài liệu tham khảo và các cấu trúc khác, nhấn mạnh việc tổ chức nội dung theo trình tự logic, yêu cầu hợp nhất thông tin trùng lặp để loại bỏ dư thừa, và giữ lại tất cả các trích dẫn nguồn.

**Mã Triển khai**:

```python
class ReportingService:
    def __init__(self, llm: HelloAgentsLLM):
        self._agent = ToolAwareSimpleAgent(
            name="Report Writer",
            system_prompt="You are a report writing expert",
            llm=llm,
            tool_call_listener=self._on_tool_call
        )

    def generate_report(
        self,
        research_topic: str,
        task_summaries: List[Tuple[TodoItem, str]]
    ) -> str:
        # Format subtask summaries
        formatted_summaries = self._format_summaries(task_summaries)

        prompt = report_writer_instructions.format(
            research_topic=research_topic,
            task_summaries=formatted_summaries,
        )

        report = self._agent.run(prompt)
        return report

    def _format_summaries(
        self,
        task_summaries: List[Tuple[TodoItem, str]]
    ) -> str:
        """Format subtask summaries"""
        formatted = []
        for idx, (task, summary) in enumerate(task_summaries, start=1):
            formatted.append(
                f"## Task {idx}: {task.title}\n"
                f"Intent: {task.intent}\n\n"
                f"{summary}\n"
            )
        return "\n".join(formatted)
```

### 14.3.2 Thiết kế ToolAwareSimpleAgent

Trong Chương 7, chúng ta đã triển khai `SimpleAgent`, đây là Agent cơ bản của framework HelloAgents. Nhưng trong trợ lý nghiên cứu chuyên sâu, chúng ta cần một Agent có thể **ghi lại các lần gọi công cụ (tool calls)**. Đây là nguồn gốc của `ToolAwareSimpleAgent`.

Trong trợ lý nghiên cứu chuyên sâu, chúng ta cần ghi lại trạng thái gọi công cụ của mỗi Agent để phục vụ cho:

1. **Debug**: Xem Agent đã gọi những công cụ nào và truyền vào những tham số gì
2. **Ghi nhật ký (Logging)**: Ghi lại tất cả các thao tác trong quá trình nghiên cứu
3. **Phân tích**: Phân tích các mẫu hành vi của Agent
4. **Hiển thị Tiến độ**: Hiển thị theo thời gian thực Agent đang làm gì

`SimpleAgent` bản thân nó không hỗ trợ lắng nghe các lần gọi công cụ, vì vậy chúng ta cần mở rộng nó.

`ToolAwareSimpleAgent` bổ sung một tham số `tool_call_listener` trên nền `SimpleAgent`. Đây là một hàm callback được gọi mỗi khi một công cụ được gọi.

**Ví dụ Sử dụng:**

```python
from hello_agents import ToolAwareSimpleAgent

def tool_listener(call_info):
    print(f"Agent: {call_info['agent_name']}")
    print(f"Tool: {call_info['tool_name']}")
    print(f"Parameters: {call_info['parsed_parameters']}")
    print(f"Result: {call_info['result']}")

agent = ToolAwareSimpleAgent(
    name="Research Assistant",
    system_prompt="You are a research assistant",
    llm=llm,
    tool_call_listener=tool_listener
)
```

`ToolAwareSimpleAgent` kế thừa từ `SimpleAgent` và ghi đè (override) phương thức `_execute_tool_call`:

```python
class ToolAwareSimpleAgent(SimpleAgent):
    def __init__(
        self,
        name: str,
        system_prompt: str,
        llm: HelloAgentsLLM,
        tool_registry: Optional[ToolRegistry] = None,
        tool_call_listener: Optional[Callable] = None,
    ):
        super().__init__(
            name=name,
            system_prompt=system_prompt,
            llm=llm,
            tool_registry=tool_registry,
        )
        self._tool_call_listener = tool_call_listener

    def _execute_tool_call(self, tool_name: str, parameters: str) -> str:
        """Execute tool call and notify listener"""
        # Parse parameters
        parsed_parameters = self._parse_parameters(parameters)

        # Call tool
        result = super()._execute_tool_call(tool_name, parameters)

        # Notify listener
        if self._tool_call_listener:
            self._tool_call_listener({
                "agent_name": self.name,
                "tool_name": tool_name,
                "parsed_parameters": parsed_parameters,
                "result": result,
            })

        return result
```

Trong trợ lý nghiên cứu chuyên sâu, chúng ta dùng `ToolAwareSimpleAgent` để ghi lại tất cả các lần gọi công cụ của Agent:

```python
class DeepResearchAgent:
    def __init__(self, config: Configuration):
        self.config = config
        self.llm = HelloAgentsLLM(...)

        # Create tool call listener
        def tool_listener(call_info):
            self._emit_event({
                "type": "tool_call",
                "agent": call_info["agent_name"],
                "tool": call_info["tool_name"],
                "parameters": call_info["parsed_parameters"],
            })

        # Create three Agents, all using the same listener
        self.planner = PlanningService(self.llm, tool_listener)
        self.summarizer = SummarizationService(self.llm, tool_listener)
        self.reporter = ReportingService(self.llm, tool_listener)
```

Bằng cách này, tất cả các lần gọi công cụ của Agent đều được ghi lại và đẩy về front-end qua SSE, hiển thị cho người dùng theo thời gian thực.

### 14.3.3 Chế độ Hợp tác của Agent

Ba Agent có mối quan hệ **hợp tác tuần tự (sequential collaboration)**, như minh họa trong Hình 14.6.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-6.png" alt="" width="85%"/>
  <p>Hình 14.6 Quy trình Hợp tác của Agent</p>
</div>

Các đặc điểm của chế độ hợp tác tuần tự là:

1. **Quy trình Tuyến tính**: Các Agent thực thi theo một thứ tự cố định
2. **Đầu vào và Đầu ra Rõ ràng**: Đầu vào của mỗi Agent đến từ đầu ra của Agent trước đó
3. **Không có Đồng thời (Concurrency)**: Chỉ một Agent làm việc tại một thời điểm

`DeepResearchAgent` là bộ điều phối cốt lõi của toàn bộ hệ thống, chịu trách nhiệm điều phối ba Agent:

```python
class DeepResearchAgent:
    def run(self, research_topic: str) -> str:
        # 1. Planning stage
        self._emit_event({"type": "status", "message": "Planning research tasks..."})
        todo_list = self.planner.plan_todo_list(research_topic)
        self._emit_event({"type": "tasks", "tasks": todo_list})

        # 2. Execution stage
        task_summaries = []
        for task in todo_list:
            self._emit_event({
                "type": "status",
                "message": f"Researching: {task.title}"
            })

            # Search
            search_results = self.search_service.search(task.query)

            # Summarize
            summary = self.summarizer.summarize_task(task, search_results)
            task_summaries.append((task, summary))

            self._emit_event({
                "type": "task_completed",
                "task_id": task.id
            })

        # 3. Reporting stage
        self._emit_event({"type": "status", "message": "Generating report..."})
        report = self.reporter.generate_report(research_topic, task_summaries)
        self._emit_event({"type": "report", "content": report})

        return report
```

## 14.4 Tích hợp Hệ thống Công cụ

### 14.4.1 Mở rộng SearchTool

Trong Chương 7, chúng ta đã triển khai phiên bản cơ bản của `SearchTool`, tích hợp các công cụ tìm kiếm Tavily và SerpApi, minh họa ý tưởng thiết kế của tìm kiếm đa nguồn. Trong trợ lý nghiên cứu chuyên sâu của chương này, chúng ta mở rộng thêm các khả năng của `SearchTool`, bổ sung DuckDuckGo, Perplexity, SearXNG và các công cụ tìm kiếm khác, và triển khai chế độ Advanced (kết hợp nhiều công cụ tìm kiếm). Tìm kiếm là chức năng cốt lõi nhất của trợ lý nghiên cứu chuyên sâu, và những mở rộng này giúp hệ thống thích ứng với các tình huống sử dụng và nhu cầu khác nhau.

Như minh họa trong Bảng 14.2, các công cụ tìm kiếm được thêm vào lần này có các đặc điểm và tình huống áp dụng khác nhau.

<div align="center">
  <p>Bảng 14.2 So sánh Đa Công cụ Tìm kiếm</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-table-2.png" alt="" width="85%"/>
</div>

Chúng ta sẽ không thảo luận riêng về cách mở rộng nữa. Bạn có thể tham khảo mã nguồn và các trường hợp mở rộng trong Chương 7 để triển khai. `SearchTool` cung cấp một giao diện tìm kiếm thống nhất. Bất kể dùng công cụ tìm kiếm nào, cách gọi đều giống nhau.

Trong trợ lý nghiên cứu chuyên sâu, chúng ta chọn công cụ tìm kiếm thông qua file cấu hình:

```python
# config.py
class SearchAPI(str, Enum):
    TAVILY = "tavily"
    DUCKDUCKGO = "duckduckgo"
    PERPLEXITY = "perplexity"
    SEARXNG = "searxng"
    ADVANCED = "advanced"

class Configuration(BaseModel):
    search_api: SearchAPI = SearchAPI.DUCKDUCKGO
    # ...
```

```python
# .env
SEARCH_API=tavily
```

Bằng cách này, người dùng có thể chọn công cụ tìm kiếm bằng cách sửa file `.env` mà không cần sửa mã.

Kết quả mà `SearchTool` trả về là một dictionary chứa:

- `results`: Danh sách kết quả tìm kiếm, mỗi kết quả chứa title, URL, snippet
- `backend`: Công cụ tìm kiếm được sử dụng
- `answer`: Câu trả lời do AI tạo ra (chỉ có ở Perplexity)
- `notices`: Thông tin thông báo (như giới hạn API, lỗi, v.v.)

Dưới đây là một số xử lý cho các trường hợp đặc biệt.

Kết quả tìm kiếm có thể chứa các URL trùng lặp, chúng ta cần khử trùng lặp:

```python
def deduplicate_sources(sources: List[dict]) -> List[dict]:
    """Remove duplicate URLs"""
    seen_urls = set()
    unique_sources = []

    for source in sources:
        if source["url"] not in seen_urls:
            seen_urls.add(source["url"])
            unique_sources.append(source)

    return unique_sources
```

Kết quả tìm kiếm có thể chứa một lượng lớn văn bản, chúng ta cần giới hạn số lượng token cho mỗi nguồn:

```python
def limit_source_tokens(source: dict, max_tokens: int = 2000) -> dict:
    """Limit the number of tokens for a source"""
    snippet = source["snippet"]

    # Simple token estimation: 1 token is approximately 4 characters
    max_chars = max_tokens * 4

    if len(snippet) > max_chars:
        snippet = snippet[:max_chars] + "..."

    return {
        **source,
        "snippet": snippet
    }
```

### 14.4.2 Sử dụng NoteTool

Trong trợ lý nghiên cứu chuyên sâu, chúng ta dùng `NoteTool` để lưu trữ bền vững (persist) tiến độ nghiên cứu. `NoteTool` là một công cụ tích hợp sẵn được đưa vào ở Chương 9, dùng để tạo, đọc, cập nhật và xóa các ghi chú (note).

Trong quá trình nghiên cứu, chúng ta cần ghi lại kết quả tìm kiếm, bản tóm tắt, và báo cáo nghiên cứu cuối cùng cho từng tác vụ con. Thông tin này cần được lưu trữ bền vững ra đĩa để khi bị gián đoạn có thể tiếp tục nghiên cứu từ tiến độ gần nhất, đồng thời cũng thuận tiện cho việc xem tất cả các thao tác trong quá trình nghiên cứu và phân tích chất lượng cũng như hiệu quả của nghiên cứu.

`NoteTool` lưu các ghi chú trong thư mục workspace được chỉ định, mỗi ghi chú là một file Markdown. Tên file của ghi chú là ID tác vụ, và nội dung bao gồm tiêu đề tác vụ, ý định tác vụ, truy vấn tìm kiếm, kết quả tìm kiếm, và bản tóm tắt.

Style của file được tạo ra cuối cùng sẽ có cấu trúc cây như sau:

```
workspace/
├── notes/
│   ├── 1.md  # Notes for task 1
│   ├── 2.md  # Notes for task 2
│   ├── 3.md  # Notes for task 3
│   └── ...
└── reports/
    └── final_report.md  # Final report
```

Trong trợ lý nghiên cứu chuyên sâu, chúng ta dùng `NoteTool` để ghi lại tiến độ nghiên cứu của từng tác vụ con:

```python
class NotesService:
    def __init__(self, workspace: str):
        self.note_tool = NoteTool(workspace=workspace)

    def save_task_summary(
        self,
        task: TodoItem,
        search_results: List[dict],
        summary: str
    ):
        """Save task summary"""
        # Format note content
        content = self._format_note_content(
            task=task,
            search_results=search_results,
            summary=summary
        )

        # Create note
        self.note_tool.run({
            "action": "create",
            "title": f"Task {task.id}: {task.title}",
            "content": content,
            "tags": ["research", "summary"]
        })

    def _format_note_content(
        self,
        task: TodoItem,
        search_results: List[dict],
        summary: str
    ) -> str:
        """Format note content"""
        content = f"# Task {task.id}: {task.title}\n\n"
        content += f"## Task Information\n\n"
        content += f"- **Intent**: {task.intent}\n"
        content += f"- **Query**: {task.query}\n\n"

        content += f"## Search Results\n\n"
        for idx, result in enumerate(search_results, start=1):
            content += f"[{idx}] {result['title']}\n"
            content += f"URL: {result['url']}\n"
            content += f"Snippet: {result['snippet']}\n\n"

        content += f"## Summary\n\n{summary}\n"

        return content
```

### 14.4.3 Quản lý Công cụ với ToolRegistry

`ToolRegistry` là bộ đăng ký công cụ (tool registry) của framework HelloAgents, cũng được hỗ trợ trong Chương 7 của chúng ta, dùng để quản lý việc đăng ký và gọi tất cả các công cụ. Trong trợ lý nghiên cứu chuyên sâu, chúng ta dùng `ToolRegistry` để quản lý `SearchTool` và `NoteTool`.

Trước khi tạo một Agent, chúng ta cần đăng ký công cụ trước:

```python
from hello_agents import ToolAwareSimpleAgent
from hello_agents.tools import ToolRegistry
from hello_agents.tools import SearchTool
from hello_agents.tools import NoteTool

# Create tools
search_tool = SearchTool(backend="hybrid")
note_tool = NoteTool(workspace="./workspace/notes")

# Create registry
registry = ToolRegistry()

# Register tools
registry.register_tool(search_tool)
registry.register_tool(note_tool)

# Create Agent
agent = ToolAwareSimpleAgent(
    name="Research Assistant",
    system_prompt="You are a research assistant",
    llm=llm,
    tool_registry=registry
)
```

Khi một Agent cần gọi một công cụ, nó sẽ tạo ra một chỉ thị gọi công cụ (tool call instruction), như minh họa trong Hình 14.7.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-7.png" alt="" width="85%"/>
  <p>Hình 14.7 Quy trình Gọi Công cụ</p>
</div>

**Quy trình Gọi Công cụ**:

1. **Agent tạo chỉ thị**: Agent tạo ra chỉ thị gọi công cụ, chẳng hạn `[TOOL_CALL:search_tool:{"input": "Datawhale organization", "backend": "tavily"}]`
2. **Phân tích chỉ thị**: `ToolRegistry` phân tích chỉ thị, trích xuất tên công cụ và các tham số
3. **Tìm công cụ**: `ToolRegistry` tìm công cụ tương ứng dựa trên tên công cụ
4. **Gọi công cụ**: Gọi phương thức `run` của công cụ, truyền vào các tham số
5. **Trả về kết quả**: Công cụ trả về kết quả thực thi
6. **Định dạng kết quả**: Định dạng kết quả thành chuỗi và trả về cho Agent

## 14.5 Triển khai Tầng Dịch vụ

Mục này sẽ giới thiệu chi tiết việc triển khai các dịch vụ cốt lõi, bao gồm PlanningService, SummarizationService, ReportingService, và SearchService. Các dịch vụ này là cầu nối kết nối Agent và công cụ, chịu trách nhiệm về logic nghiệp vụ cụ thể.

### 14.5.1 Dịch vụ Lập kế hoạch Tác vụ

`PlanningService` chịu trách nhiệm gọi Agent lập kế hoạch nghiên cứu để phân rã chủ đề nghiên cứu thành các tác vụ con. Đây là bước đầu tiên và quan trọng nhất của toàn bộ quy trình nghiên cứu.

**(1) Cách tiếp cận Triển khai**

Các trách nhiệm cốt lõi của nó là:

1. **Xây dựng Prompt lập kế hoạch**: Xây dựng Prompt dựa trên chủ đề nghiên cứu và ngày hiện tại
2. **Gọi Agent lập kế hoạch**: Gọi Agent TODO Planner để tạo danh sách tác vụ con
3. **Phân tích phản hồi JSON**: Trích xuất danh sách tác vụ con ở định dạng JSON từ phản hồi của Agent
4. **Xác thực định dạng tác vụ con**: Đảm bảo mỗi tác vụ con chứa các trường bắt buộc (title, intent, query)

```python
import re
import json
from typing import List, Callable, Optional
from datetime import datetime

from hello_agents import HelloAgentsLLM
from hello_agents import ToolAwareSimpleAgent
from models import TodoItem, SummaryState
from prompts import todo_planner_instructions

class PlanningService:
    """Task planning service"""

    def __init__(
        self,
        llm: HelloAgentsLLM,
        tool_call_listener: Optional[Callable] = None
    ):
        self._llm = llm
        self._tool_call_listener = tool_call_listener

        # Create planning Agent
        self._agent = ToolAwareSimpleAgent(
            name="TODO Planner",
            system_prompt="You are a research planning expert, skilled at decomposing complex research topics into clear subtasks.",
            llm=llm,
            tool_call_listener=tool_call_listener
        )

    def plan_todo_list(self, state: SummaryState) -> List[TodoItem]:
        """Plan TODO list

        Args:
            state: Research state, containing research topic

        Returns:
            Subtask list
        """
        # Build Prompt
        prompt = todo_planner_instructions.format(
            current_date=self._get_current_date(),
            research_topic=state.research_topic,
        )

        # Call Agent
        response = self._agent.run(prompt)

        # Parse JSON
        tasks_payload = self._extract_tasks(response)

        # Validate and create TodoItem
        todo_items = []
        for idx, item in enumerate(tasks_payload, start=1):
            # Validate required fields
            if not all(key in item for key in ["title", "intent", "query"]):
                raise ValueError(f"Task {idx} is missing required fields")

            task = TodoItem(
                id=idx,
                title=item["title"],
                intent=item["intent"],
                query=item["query"],
            )
            todo_items.append(task)

        return todo_items

    def _get_current_date(self) -> str:
        """Get current date"""
        return datetime.now().strftime("%Y-%m-%d")

    def _extract_tasks(self, response: str) -> List[dict]:
        """Extract JSON from Agent response

        The Agent's response may contain extra text, such as:
        "Okay, I will plan the following tasks for you:\n[{...}, {...}]\nThese tasks cover..."

        We need to extract the JSON part.
        """
        # Method 1: Use regex to extract JSON array
        json_match = re.search(r'\[.*\]', response, re.DOTALL)
        if json_match:
            json_str = json_match.group(0)
            try:
                return json.loads(json_str)
            except json.JSONDecodeError as e:
                raise ValueError(f"JSON parsing failed: {e}")

        # Method 2: If no JSON array is found, try to parse the entire response directly
        try:
            return json.loads(response)
        except json.JSONDecodeError:
            raise ValueError("Unable to extract JSON from response")
```

**(2) Phân tích và Xác thực JSON**

JSON do Agent trả về có thể chứa văn bản thừa hoặc lỗi định dạng, nên chúng ta cần logic phân tích mạnh mẽ (robust):

**Các Vấn đề Thường gặp**:

1. **Chứa văn bản thừa**: Agent có thể thêm văn bản giải thích trước và sau JSON
2. **Lỗi định dạng**: JSON có thể thiếu dấu ngoặc kép, dấu phẩy, v.v.
3. **Thiếu trường**: Một số tác vụ con có thể thiếu các trường bắt buộc

**Giải pháp**:

1. **Dùng regex**: Trích xuất phần JSON
2. **Nhiều chiến lược phân tích**: Đầu tiên thử trích xuất mảng JSON, sau đó thử phân tích trực tiếp
3. **Xác thực trường**: Đảm bảo mỗi tác vụ con chứa các trường bắt buộc

**Ví dụ**:

```python
# Agent response example 1: Contains extra text
response1 = """
Okay, I will plan the following tasks for you:

[
  {
    "title": "What is a multimodal model",
    "intent": "Understand basic concepts",
    "query": "multimodal model definition"
  },
  {
    "title": "Latest multimodal models",
    "intent": "Understand technical status",
    "query": "latest multimodal models 2024"
  }
]

These tasks cover the basic information and core projects of the Datawhale organization.
"""

# Extract JSON
tasks1 = service._extract_tasks(response1)
# Result: [{"title": "Basic information about Datawhale", ...}, ...]

# Agent response example 2: Pure JSON
response2 = """
[
  {"title": "Basic information about Datawhale", "intent": "Understand organizational positioning", "query": "Datawhale organization introduction"},
  {"title": "Main projects of Datawhale", "intent": "Understand core content", "query": "Datawhale projects tutorials 2024"}
]
"""

# Extract JSON
tasks2 = service._extract_tasks(response2)
# Result: [{"title": "What is a multimodal model", ...}, ...]
```

**(3) Đánh giá Chất lượng Lập kế hoạch**

Một kế hoạch tốt nên đáp ứng các tiêu chí sau:

1. **Bao phủ toàn diện**: Bao phủ tất cả các khía cạnh quan trọng của chủ đề
2. **Logic rõ ràng**: Các mối quan hệ logic rõ ràng giữa các tác vụ con
3. **Truy vấn chính xác**: Truy vấn tìm kiếm có thể tìm chính xác tài liệu liên quan
4. **Số lượng phù hợp**: 3-5 tác vụ con

Chúng ta có thể thêm một phương thức đánh giá:

```python
def evaluate_plan(self, todo_items: List[TodoItem]) -> dict:
    """Evaluate planning quality

    Returns:
        Evaluation results, including score and suggestions
    """
    score = 100
    suggestions = []

    # Check quantity
    if len(todo_items) < 3:
        score -= 20
        suggestions.append("Too few subtasks, may miss important information")
    elif len(todo_items) > 5:
        score -= 10
        suggestions.append("Too many subtasks, may have redundancy")

    # Check query quality
    for task in todo_items:
        if len(task.query.split()) < 2:
            score -= 10
            suggestions.append(f"Query for task '{task.title}' is too simple")

    # Check logical relationships
    # (More complex logic checks can be added here)

    return {
        "score": score,
        "suggestions": suggestions
    }
```

### 14.5.2 Dịch vụ Tóm tắt

`SummarizationService` chịu trách nhiệm gọi Agent tóm tắt tác vụ để tóm tắt kết quả tìm kiếm. Đây là mắt xích cốt lõi của quy trình nghiên cứu và quyết định chất lượng của nghiên cứu.

Các trách nhiệm của nó là:

1. **Định dạng kết quả tìm kiếm**: Định dạng kết quả tìm kiếm thành văn bản dễ đọc
2. **Xây dựng Prompt tóm tắt**: Xây dựng Prompt dựa trên thông tin tác vụ và kết quả tìm kiếm
3. **Gọi Agent tóm tắt**: Gọi Agent Task Summarizer để tạo bản tóm tắt
4. **Trích xuất trích dẫn nguồn**: Trích xuất trích dẫn nguồn từ bản tóm tắt

Mã cốt lõi:

```python
from typing import List, Callable, Optional, Tuple

from hello_agents import HelloAgentsLLM
from hello_agents import ToolAwareSimpleAgent
from models import TodoItem
from prompts import task_summarizer_instructions

class SummarizationService:
    """Summarization service"""

    def __init__(
        self,
        llm: HelloAgentsLLM,
        tool_call_listener: Optional[Callable] = None
    ):
        self._llm = llm
        self._tool_call_listener = tool_call_listener

        # Create summarization Agent
        self._agent = ToolAwareSimpleAgent(
            name="Task Summarizer",
            system_prompt="You are a task summarization expert, skilled at extracting key information from search results.",
            llm=llm,
            tool_call_listener=tool_call_listener
        )

    def summarize_task(
        self,
        task: TodoItem,
        search_results: List[dict]
    ) -> Tuple[str, List[str]]:
        """Summarize task

        Args:
            task: Task information
            search_results: Search results list

        Returns:
            (Summary text, source URL list)
        """
        # Format search results
        formatted_sources = self._format_sources(search_results)

        # Build Prompt
        prompt = task_summarizer_instructions.format(
            task_title=task.title,
            task_intent=task.intent,
            task_query=task.query,
            search_results=formatted_sources,
        )

        # Call Agent
        summary = self._agent.run(prompt)

        # Extract source URLs
        source_urls = [result["url"] for result in search_results]

        return summary, source_urls

    def _format_sources(self, search_results: List[dict]) -> str:
        """Format search results

        Format search results into readable text, including:
        - Serial number
        - Title
        - URL
        - Snippet
        """
        formatted = []
        for idx, result in enumerate(search_results, start=1):
            formatted.append(
                f"[{idx}] {result['title']}\n"
                f"URL: {result['url']}\n"
                f"Snippet: {result['snippet']}\n"
            )
        return "\n".join(formatted)
```

### Thiết kế Cấu trúc Báo cáo

Báo cáo cuối cùng nên bao gồm các phần sau:

## References

### Task 1: What is a Multimodal Model
- https://example.com/multimodal-model-definition
...

### Task 2: What are the Latest Multimodal Models
- https://example.com/gpt4v
...
...

### 14.5.3 Dịch vụ Tạo Báo cáo

`ReportingService` chịu trách nhiệm gọi Agent tạo báo cáo để tích hợp các bản tóm tắt của tất cả các tác vụ con. Đây là bước cuối cùng của quy trình nghiên cứu, tạo ra báo cáo nghiên cứu cuối cùng.

Các trách nhiệm của nó là:

1. **Định dạng các bản tóm tắt tác vụ con**: Định dạng tất cả các bản tóm tắt tác vụ con thành một định dạng thống nhất
2. **Xây dựng Prompt báo cáo**: Xây dựng Prompt dựa trên chủ đề nghiên cứu và các bản tóm tắt tác vụ con
3. **Gọi Agent báo cáo**: Gọi Agent Report Writer để tạo báo cáo cuối cùng
4. **Tổ chức trích dẫn**: Tổ chức tất cả các trích dẫn nguồn vào phần tài liệu tham khảo

**Triển khai Mã Cốt lõi**:

```python
from typing import List, Callable, Optional, Tuple

from hello_agents import HelloAgentsLLM
from hello_agents import ToolAwareSimpleAgent
from models import TodoItem
from prompts import report_writer_instructions

class ReportingService:
    """Report generation service"""

    def __init__(
        self,
        llm: HelloAgentsLLM,
        tool_call_listener: Optional[Callable] = None
    ):
        self._llm = llm
        self._tool_call_listener = tool_call_listener

        # Create report Agent
        self._agent = ToolAwareSimpleAgent(
            name="Report Writer",
            system_prompt="You are a report writing expert, skilled at integrating information and generating structured reports.",
            llm=llm,
            tool_call_listener=tool_call_listener
        )

    def generate_report(
        self,
        research_topic: str,
        task_summaries: List[Tuple[TodoItem, str, List[str]]]
    ) -> str:
        """Generate final report

        Args:
            research_topic: Research topic
            task_summaries: Subtask summary list, each element is (task, summary, source URL list)

        Returns:
            Final report (Markdown format)
        """
        # Format subtask summaries
        formatted_summaries = self._format_summaries(task_summaries)

        # Build Prompt
        prompt = report_writer_instructions.format(
            research_topic=research_topic,
            task_summaries=formatted_summaries,
        )

        # Call Agent
        report = self._agent.run(prompt)

        return report

    def _format_summaries(
        self,
        task_summaries: List[Tuple[TodoItem, str, List[str]]]
    ) -> str:
        """Format subtask summaries

        Format all subtask summaries into a unified format, including:
        - Task serial number
        - Task title
        - Task intent
        - Summary content
        - Source URLs
        """
        formatted = []
        for idx, (task, summary, source_urls) in enumerate(task_summaries, start=1):
            formatted.append(
                f"## Task {idx}: {task.title}\n\n"
                f"**Intent**: {task.intent}\n\n"
                f"{summary}\n\n"
                f"**Sources**:\n"
            )
            for url in source_urls:
                formatted.append(f"- {url}\n")
            formatted.append("\n")

        return "".join(formatted)
```

### 14.5.4 Dịch vụ Điều phối Tìm kiếm

`SearchService` chịu trách nhiệm điều phối các công cụ tìm kiếm, thực thi việc tìm kiếm, và trả về kết quả. Đây là cầu nối kết nối Agent và SearchTool. Ở đây chúng ta không áp dụng hình thức thông thường là để SimpleAgent trực tiếp gọi công cụ, mà thay vào đó trả về kết quả thực thi của SearchTool cho Agent thông qua một tầng trung gian, điều này giúp Agent tập trung hơn vào việc xử lý thông tin đã thu được.

Các trách nhiệm của nó là:

1. **Điều phối công cụ tìm kiếm**: Chọn công cụ tìm kiếm dựa trên cấu hình
2. **Thực thi tìm kiếm**: Gọi SearchTool để thực thi việc tìm kiếm
3. **Xử lý kết quả**: Khử trùng lặp, giới hạn token, định dạng
4. **Xử lý lỗi**: Xử lý các tình huống tìm kiếm thất bại

Mã cốt lõi:

```python
from typing import List, Optional
import logging

from hello_agents.tools import SearchTool
from config import Configuration

logger = logging.getLogger(__name__)

class SearchService:
    """Search scheduling service"""

    def __init__(self, config: Configuration):
        self.config = config

        # Create SearchTool
        self.search_tool = SearchTool(backend="hybrid")

    def search(
        self,
        query: str,
        max_results: int = 5
    ) -> List[dict]:
        """Execute search

        Args:
            query: Search query
            max_results: Maximum number of results

        Returns:
            Search results list
        """
        try:
            # Call SearchTool
            raw_response = self.search_tool.run({
                "input": query,
                "backend": self.config.search_api.value,
                "mode": "structured",
                "max_results": max_results
            })

            # Extract results
            results = raw_response.get("results", [])

            # Process results
            results = self._deduplicate_sources(results)
            results = self._limit_source_tokens(results)

            logger.info(f"Search successful: {query}, returned {len(results)} results")

            return results

        except Exception as e:
            logger.error(f"Search failed: {query}, error: {e}")
            return []

    def _deduplicate_sources(self, sources: List[dict]) -> List[dict]:
        """Remove duplicate URLs"""
        seen_urls = set()
        unique_sources = []

        for source in sources:
            url = source.get("url", "")
            if url and url not in seen_urls:
                seen_urls.add(url)
                unique_sources.append(source)

        return unique_sources

    def _limit_source_tokens(
        self,
        sources: List[dict],
        max_tokens_per_source: int = 2000
    ) -> List[dict]:
        """Limit the number of tokens per source"""
        limited_sources = []

        for source in sources:
            snippet = source.get("snippet", "")

            # Simple token estimation: 1 token is approximately 4 characters
            max_chars = max_tokens_per_source * 4

            if len(snippet) > max_chars:
                snippet = snippet[:max_chars] + "..."

            limited_sources.append({
                **source,
                "snippet": snippet
            })

        return limited_sources
```

Chọn công cụ tìm kiếm dựa trên cấu hình, như minh họa trong Hình 14.8:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-8.png" alt="" width="85%"/>
  <p>Hình 14.8 Quy trình Điều phối Công cụ Tìm kiếm</p>
</div>

**Logic Điều phối**:

1. **Đọc cấu hình**: Đọc cấu hình `SEARCH_API` từ file `.env`
2. **Chọn engine**: Chọn công cụ tìm kiếm dựa trên cấu hình (tavily, duckduckgo, perplexity, v.v.)
3. **Thực thi tìm kiếm**: Gọi SearchTool để thực thi việc tìm kiếm
4. **Xử lý kết quả**: Khử trùng lặp, giới hạn token, định dạng
5. **Trả về kết quả**: Trả về kết quả tìm kiếm đã xử lý

Để nâng cao hiệu quả và giảm chi phí, chúng ta có thể thêm bộ nhớ đệm (cache) kết quả tìm kiếm:

```python
import hashlib
import json
from pathlib import Path

class SearchService:
    def __init__(self, config: Configuration):
        self.config = config
        self.search_tool = SearchTool(backend="hybrid")

        # Cache directory
        self.cache_dir = Path("./cache/search")
        self.cache_dir.mkdir(parents=True, exist_ok=True)

    def search(
        self,
        query: str,
        max_results: int = 5,
        use_cache: bool = True
    ) -> List[dict]:
        """Execute search (with cache)"""
        # Generate cache key
        cache_key = self._generate_cache_key(query, max_results)
        cache_file = self.cache_dir / f"{cache_key}.json"

        # Try to read from cache
        if use_cache and cache_file.exists():
            logger.info(f"Reading search results from cache: {query}")
            with open(cache_file, "r", encoding="utf-8") as f:
                return json.load(f)

        # Execute search
        results = self._execute_search(query, max_results)

        # Save to cache
        if use_cache and results:
            with open(cache_file, "w", encoding="utf-8") as f:
                json.dump(results, f, ensure_ascii=False, indent=2)

        return results

    def _generate_cache_key(self, query: str, max_results: int) -> str:
        """Generate cache key"""
        # Generate MD5 hash using query and max results
        content = f"{query}_{max_results}_{self.config.search_api.value}"
        return hashlib.md5(content.encode()).hexdigest()
```

Thông qua bốn dịch vụ cốt lõi (PlanningService, SummarizationService, ReportingService, SearchService), chúng ta đã xây dựng một quy trình nghiên cứu hoàn chỉnh. Các dịch vụ này mỗi cái đảm nhiệm nhiệm vụ riêng và hợp tác thông qua các giao diện rõ ràng, hiện thực hóa một quy trình tự động từ chủ đề nghiên cứu đến báo cáo cuối cùng.

## 14.6 Thiết kế Tương tác Front-End

Trong các mục trước, chúng ta đã triển khai hệ thống back-end hoàn chỉnh. Mục này sẽ giới thiệu chi tiết thiết kế tương tác front-end, bao gồm giao diện hộp thoại modal toàn màn hình, hiển thị tiến độ theo thời gian thực, và trực quan hóa kết quả nghiên cứu.

### 14.6.1 Thiết kế Giao diện Hộp thoại Modal Toàn màn hình

Trợ lý nghiên cứu chuyên sâu áp dụng thiết kế giao diện hộp thoại modal toàn màn hình, có các ưu điểm sau:

1. **Trải nghiệm nhập tâm (immersive)**: Hiển thị toàn màn hình, tránh xao nhãng, tập trung vào nghiên cứu
2. **Phân cấp rõ ràng**: Trang chính và trang nghiên cứu tách biệt, phân cấp rõ ràng
3. **Dễ đóng**: Nhấp vào nút đóng hoặc nhấn phím ESC để quay lại trang chính
4. **Thiết kế responsive**: Thích ứng với các kích thước màn hình khác nhau

Như minh họa trong Hình 14.9, hộp thoại modal toàn màn hình chứa các phần sau:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-9.png" alt="" width="85%"/>
  <p>Hình 14.9 Giao diện Hộp thoại Modal Toàn màn hình</p>
</div>

**Các Thành phần Giao diện**:

1. **Thanh trên cùng (Top bar)**: Chứa chủ đề nghiên cứu và nút đóng
2. **Vùng tiến độ (Progress area)**: Hiển thị tiến độ nghiên cứu hiện tại (lập kế hoạch, thực thi, báo cáo)
3. **Vùng nội dung (Content area)**: Hiển thị kết quả nghiên cứu (định dạng Markdown)
4. **Thanh dưới cùng (Bottom bar)**: Hiển thị thông tin trạng thái (như "Researching...", "Completed")

Phần triển khai Vue tương ứng như sau (ResearchModal.vue):

```vue
<template>
  <div v-if="isOpen" class="modal-overlay" @click.self="close">
    <div class="modal-container">
      <!-- Top bar -->
      <div class="modal-header">
        <h2>{{ researchTopic }}</h2>
        <button @click="close" class="close-button">
          <svg><!-- Close icon --></svg>
        </button>
      </div>

      <!-- Progress area -->
      <div class="progress-section">
        <div class="progress-bar">
          <div
            class="progress-fill"
            :style="{ width: progressPercentage + '%' }"
          ></div>
        </div>
        <div class="progress-text">{{ progressText }}</div>
      </div>

      <!-- Content area -->
      <div class="content-section">
        <div v-if="isLoading" class="loading-spinner">
          <div class="spinner"></div>
          <p>Researching, please wait...</p>
        </div>

        <div v-else class="markdown-content" v-html="renderedMarkdown"></div>
      </div>

      <!-- Bottom bar -->
      <div class="modal-footer">
        <span class="status-text">{{ statusText }}</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { marked } from 'marked'

interface Props {
  isOpen: boolean
  researchTopic: string
}

const props = defineProps<Props>()
const emit = defineEmits<{
  close: []
}>()

// State
const isLoading = ref(true)
const progressPercentage = ref(0)
const progressText = ref('Preparing...')
const statusText = ref('Researching...')
const markdownContent = ref('')

// Render Markdown
const renderedMarkdown = computed(() => {
  return marked(markdownContent.value)
})

// Close modal
const close = () => {
  emit('close')
}

// Listen for ESC key
const handleKeydown = (e: KeyboardEvent) => {
  if (e.key === 'Escape') {
    close()
  }
}

// Add keyboard listener on mount
watch(() => props.isOpen, (isOpen) => {
  if (isOpen) {
    document.addEventListener('keydown', handleKeydown)
  } else {
    document.removeEventListener('keydown', handleKeydown)
  }
})
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}
...
</style>
```

Để thích ứng với các kích thước màn hình khác nhau, chúng ta thêm các media query:

```css
/* Tablet devices */
@media (max-width: 768px) {
  .modal-container {
    width: 95vw;
    height: 95vh;
  }

  .modal-header,
  .progress-section,
  .content-section,
  .modal-footer {
    padding: 15px 20px;
  }
}

/* Mobile devices */
@media (max-width: 480px) {
  .modal-container {
    width: 100vw;
    height: 100vh;
    border-radius: 0;
  }

  .modal-header h2 {
    font-size: 18px;
  }
}
```

### 14.6.2 Hiển thị Tiến độ Theo thời gian Thực

Trợ lý nghiên cứu chuyên sâu dùng SSE để hiện thực hóa việc hiển thị tiến độ theo thời gian thực. SSE là một công nghệ đẩy dữ liệu từ máy chủ (server push) cho phép máy chủ chủ động gửi dữ liệu tới client, điều này cũng đã được giải thích trong chương về giao thức (protocol).

Như minh họa trong Hình 14.10, quy trình SSE bao gồm các bước sau:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-10.png" alt="" width="85%"/>
  <p>Hình 14.10 Quy trình SSE</p>
</div>

**Mô tả Quy trình**:

1. **Client khởi tạo yêu cầu**: Gửi yêu cầu POST tới `/api/research`, chứa chủ đề nghiên cứu
2. **Server thiết lập kết nối SSE**: Trả về phản hồi `text/event-stream`
3. **Server đẩy tiến độ**: Định kỳ đẩy tiến độ nghiên cứu (lập kế hoạch, thực thi, báo cáo)
4. **Client nhận tiến độ**: Lắng nghe các sự kiện SSE, cập nhật giao diện
5. **Nghiên cứu hoàn tất**: Server đẩy báo cáo cuối cùng, đóng kết nối

Nếu bạn muốn dùng SSE trong các dự án front-end và back-end, bạn cũng cần thực hiện các cấu hình sau.

**Endpoint SSE của Back-End FastAPI**:

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from typing import AsyncGenerator
import asyncio
import json

app = FastAPI()

async def research_stream(topic: str) -> AsyncGenerator[str, None]:
    """Research streaming generator

    Generate SSE format data:
    data: {"type": "progress", "data": {...}}

    """
    try:
        # 1. Planning stage
        yield f"data: {json.dumps({'type': 'progress', 'stage': 'planning', 'percentage': 10, 'text': 'Planning research tasks...'})}\n\n"

        # Call PlanningService
        todo_items = await planning_service.plan_todo_list(topic)

        yield f"data: {json.dumps({'type': 'plan', 'data': [item.dict() for item in todo_items]})}\n\n"

        # 2. Execution stage
        task_summaries = []
        for idx, task in enumerate(todo_items, start=1):
            # Update progress
            percentage = 10 + (idx / len(todo_items)) * 70
            yield f"data: {json.dumps({'type': 'progress', 'stage': 'executing', 'percentage': percentage, 'text': f'Researching task {idx}/{len(todo_items)}: {task.title}'})}\n\n"

            # Search
            search_results = await search_service.search(task.query)

            # Summarize
            summary, source_urls = await summarization_service.summarize_task(task, search_results)

            task_summaries.append((task, summary, source_urls))

            # Push task summary
            yield f"data: {json.dumps({'type': 'task_summary', 'task_id': task.id, 'summary': summary})}\n\n"

        # 3. Reporting stage
        yield f"data: {json.dumps({'type': 'progress', 'stage': 'reporting', 'percentage': 90, 'text': 'Generating final report...'})}\n\n"

        # Generate report
        report = await reporting_service.generate_report(topic, task_summaries)

        # Push final report
        yield f"data: {json.dumps({'type': 'report', 'data': report})}\n\n"

        # Complete
        yield f"data: {json.dumps({'type': 'progress', 'stage': 'completed', 'percentage': 100, 'text': 'Research complete!'})}\n\n"

    except Exception as e:
        # Error handling
        yield f"data: {json.dumps({'type': 'error', 'message': str(e)})}\n\n"

@app.post("/api/research")
async def research(request: ResearchRequest):
    """Research endpoint (SSE)"""
    return StreamingResponse(
        research_stream(request.topic),
        media_type="text/event-stream",
        headers={
            "Cache-Control": "no-cache",
            "Connection": "keep-alive",
        }
    )
```

**Front-End Dùng EventSource để Nhận SSE**:

```typescript
// composables/useResearch.ts
import { ref } from 'vue'

export function useResearch() {
  const isLoading = ref(false)
  const progressPercentage = ref(0)
  const progressText = ref('')
  const markdownContent = ref('')
  const error = ref<string | null>(null)

  const startResearch = (topic: string) => {
    isLoading.value = true
    error.value = null

    // Create EventSource
    const eventSource = new EventSource(`/api/research?topic=${encodeURIComponent(topic)}`)

    // Listen for messages
    eventSource.onmessage = (event) => {
      const data = JSON.parse(event.data)

      switch (data.type) {
        case 'progress':
          progressPercentage.value = data.percentage
          progressText.value = data.text
          break

        case 'plan':
          // Display planning results
          console.log('Planning results:', data.data)
          break

        case 'task_summary':
          // Append task summary to Markdown
          markdownContent.value += `\n\n## Task ${data.task_id}\n\n${data.summary}`
          break

        case 'report':
          // Display final report
          markdownContent.value = data.data
          break

        case 'error':
          error.value = data.message
          eventSource.close()
          isLoading.value = false
          break

        case 'completed':
          eventSource.close()
          isLoading.value = false
          break
      }
    }

    // Error handling
    eventSource.onerror = (err) => {
      console.error('SSE error:', err)
      error.value = 'Connection failed, please retry'
      eventSource.close()
      isLoading.value = false
    }
  }

  return {
    isLoading,
    progressPercentage,
    progressText,
    markdownContent,
    error,
    startResearch,
  }
}
```

**Sử dụng trong Component**:

```vue
<script setup lang="ts">
import { useResearch } from '@/composables/useResearch'

const {
  isLoading,
  progressPercentage,
  progressText,
  markdownContent,
  error,
  startResearch
} = useResearch()

const handleStartResearch = (topic: string) => {
  startResearch(topic)
}
</script>
```

### 14.6.3 Trực quan hóa Kết quả Nghiên cứu

Kết quả nghiên cứu được hiển thị ở định dạng Markdown, bao gồm tiêu đề, đoạn văn, danh sách, trích dẫn và các phần tử khác. Chúng ta dùng thư viện `marked` để chuyển đổi Markdown thành HTML và thêm các style tùy chỉnh.

**Render Markdown**:

```typescript
import { marked } from 'marked'

// Configure marked
marked.setOptions({
  breaks: true,  // Support line breaks
  gfm: true,     // Support GitHub Flavored Markdown
})

// Render
const renderedHtml = marked(markdownContent.value)
```

Các báo cáo nghiên cứu chứa một lượng lớn trích dẫn nguồn, mà chúng ta cần xử lý đặc biệt:

```markdown
## References

### Task 1: Basic Information about Datawhale
- [Datawhale GitHub](https://github.com/datawhalechina)
- [Datawhale Official Website](https://datawhale.club)

### Task 2: Main Projects of Datawhale
- [Hello-Agents Tutorial](https://github.com/datawhalechina/Hello-Agents)
...
```

Thông qua giao diện hộp thoại modal toàn màn hình, hiển thị tiến độ theo thời gian thực bằng SSE, và trực quan hóa kết quả bằng Markdown, chúng ta đã xây dựng một giao diện front-end thân thiện với người dùng. Người dùng có thể thấy rõ tiến độ nghiên cứu và xem kết quả nghiên cứu ở định dạng đẹp mắt.

## 14.7 Tóm tắt Chương

Trong chương này, chúng ta đã xây dựng một hệ thống agent nghiên cứu chuyên sâu tự động hoàn chỉnh từ đầu. Hãy cùng điểm lại các nội dung cốt lõi:

**(1) Mô hình Nghiên cứu Hướng TODO**

Chúng ta đã đề xuất một mô hình nghiên cứu mới - nghiên cứu hướng TODO. Mô hình này phân rã các chủ đề nghiên cứu phức tạp thành các tác vụ con có thể thực thi và hoàn thành nghiên cứu qua ba giai đoạn:

- **Giai đoạn lập kế hoạch**: Phân rã chủ đề nghiên cứu thành 3-5 tác vụ con, mỗi tác vụ con chứa tiêu đề, ý định, và truy vấn tìm kiếm
- **Giai đoạn thực thi**: Thực thi việc tìm kiếm và tóm tắt cho từng tác vụ con, tạo ra tri thức có cấu trúc
- **Giai đoạn báo cáo**: Tích hợp các bản tóm tắt của tất cả các tác vụ con, tạo ra báo cáo nghiên cứu cuối cùng

Các ưu điểm của mô hình này là:

1. **Khả năng kiểm soát cao**: Mỗi tác vụ con có mục tiêu và phạm vi rõ ràng
2. **Chất lượng đáng tin cậy**: Các Agent chuyên biệt đảm bảo chất lượng ở từng giai đoạn
3. **Dễ debug**: Có thể debug từng tác vụ con riêng lẻ
4. **Khả năng mở rộng tốt**: Có thể dễ dàng thêm các tác vụ con mới hoặc sửa các tác vụ con hiện có

**(2) Hệ thống Hợp tác Ba Agent**

Chúng ta đã thiết kế ba Agent chuyên biệt, mỗi Agent đảm nhiệm nhiệm vụ riêng:

- **TODO Planner (Chuyên gia Lập kế hoạch Nghiên cứu)**: Chịu trách nhiệm phân rã chủ đề nghiên cứu thành các tác vụ con
- **Task Summarizer (Chuyên gia Tóm tắt Tác vụ)**: Chịu trách nhiệm tóm tắt kết quả tìm kiếm cho từng tác vụ con
- **Report Writer (Chuyên gia Viết Báo cáo)**: Chịu trách nhiệm tích hợp các bản tóm tắt của tất cả các tác vụ con và tạo báo cáo cuối cùng

Các ưu điểm của thiết kế này là:

1. **Trách nhiệm rõ ràng**: Mỗi Agent tập trung vào một tác vụ cụ thể
2. **Tối ưu hóa Prompt**: Có thể tùy chỉnh Prompt chuyên biệt cho từng Agent
3. **Dễ bảo trì**: Sửa một Agent không ảnh hưởng đến các Agent khác
4. **Đảm bảo chất lượng**: Mỗi Agent là một "chuyên gia" trong lĩnh vực của mình

**(3) Thiết kế ToolAwareSimpleAgent**

Chúng ta đã mở rộng `SimpleAgent` của framework HelloAgents và triển khai `ToolAwareSimpleAgent`. Agent này có năng lực lắng nghe các lần gọi công cụ và có thể:

- **Lắng nghe các lần gọi công cụ**: Lắng nghe từng lần gọi công cụ thông qua các hàm callback
- **Phản hồi theo thời gian thực**: Đẩy thông tin gọi công cụ về front-end theo thời gian thực
- **Hỗ trợ debug**: Ghi lại tất cả các lần gọi công cụ để dễ dàng debug

Agent này đã được tích hợp vào framework HelloAgents và có thể tái sử dụng trong các dự án khác.

**(4) Tích hợp Hệ thống Công cụ**

Chúng ta đã tận dụng đầy đủ hệ thống công cụ của framework HelloAgents:

- **SearchTool**: Được mở rộng để hỗ trợ nhiều công cụ tìm kiếm hơn (Tavily, DuckDuckGo, Perplexity, v.v.)
- **NoteTool**: Lưu trữ bền vững tiến độ nghiên cứu, hỗ trợ khôi phục và kiểm tra (auditing)
- **ToolRegistry**: Quản lý thống nhất tất cả các công cụ, hỗ trợ mở rộng tùy chỉnh

Thông qua thiết kế dựa trên cấu hình, người dùng có thể dễ dàng chuyển đổi công cụ tìm kiếm mà không cần sửa mã.

**(5) Triển khai Dịch vụ Cốt lõi**

Chúng ta đã triển khai bốn dịch vụ cốt lõi kết nối Agent và công cụ:

- **PlanningService**: Gọi Agent lập kế hoạch, phân tích JSON, xác thực định dạng
- **SummarizationService**: Gọi Agent tóm tắt, xử lý kết quả tìm kiếm, trích xuất nguồn
- **ReportingService**: Gọi Agent báo cáo, tích hợp các bản tóm tắt, tạo báo cáo
- **SearchService**: Điều phối các công cụ tìm kiếm, xử lý kết quả, suy giảm khi lỗi (error degradation), cache kết quả

Các dịch vụ này mỗi cái đảm nhiệm nhiệm vụ riêng và hợp tác thông qua các giao diện rõ ràng, hiện thực hóa một quy trình tự động từ chủ đề nghiên cứu đến báo cáo cuối cùng.

**(6) Thiết kế Tương tác Front-End**

Chúng ta đã thiết kế một giao diện front-end thân thiện với người dùng:

- **Hộp thoại modal toàn màn hình**: Trải nghiệm nhập tâm, phân cấp rõ ràng
- **Tiến độ theo thời gian thực bằng SSE**: Hiển thị tiến độ nghiên cứu theo thời gian thực, trải nghiệm người dùng tốt
- **Trực quan hóa Markdown**: Định dạng đẹp mắt, cấu trúc rõ ràng

Thông qua ngăn xếp công nghệ Vue 3 + TypeScript + SSE, chúng ta đã triển khai một ứng dụng web hiện đại.

Kiến thức này không chỉ áp dụng được cho các trợ lý nghiên cứu chuyên sâu, mà còn có thể áp dụng cho các ứng dụng AI khác. Chúng tôi hy vọng độc giả có thể khám phá thêm nhiều khả năng dựa trên chương này và xây dựng các hệ thống AI mạnh mẽ hơn.

Ở chương tiếp theo, chúng ta sẽ xây dựng một hệ thống multi-agent kết hợp với một game engine (công cụ trò chơi) - Cyber Town, khám phá các mẫu tương tác và hợp tác phức tạp giữa các Agent. Hãy đón chờ nhé!
