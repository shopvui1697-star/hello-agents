# Chương 9 Kỹ thuật Ngữ cảnh

Trong các chương trước, chúng ta đã giới thiệu hệ thống bộ nhớ (memory) và RAG cho agent. Tuy nhiên, để agent có thể "suy nghĩ" và "hành động" một cách ổn định trong các tình huống phức tạp thực tế, chỉ có bộ nhớ và truy xuất (retrieval) là chưa đủ — chúng ta cần một phương pháp luận mang tính kỹ thuật để liên tục và có hệ thống xây dựng "ngữ cảnh" (context) phù hợp cho mô hình. Đây chính là chủ đề của chương này: Kỹ thuật Ngữ cảnh (Context Engineering). Nó tập trung vào việc "làm thế nào để lắp ráp và tối ưu hóa ngữ cảnh đầu vào theo cách có thể tái sử dụng, đo lường được và tiến hóa được trước mỗi lần gọi mô hình", qua đó nâng cao tính đúng đắn, độ bền vững và hiệu quả<sup>[1][2]</sup>.

Để độc giả có thể nhanh chóng trải nghiệm đầy đủ chức năng của chương này, chúng tôi cung cấp một gói Python có thể cài đặt trực tiếp. Bạn có thể cài đặt phiên bản tương ứng với chương này bằng lệnh sau:

```bash
pip install "hello-agents[all]==0.2.8"
```

Chương này chủ yếu giới thiệu các khái niệm cốt lõi và thực hành của kỹ thuật ngữ cảnh, đồng thời bổ sung một trình xây dựng ngữ cảnh (context builder) và hai công cụ hỗ trợ vào framework HelloAgents:

- **ContextBuilder** (`hello_agents/context/builder.py`): Trình xây dựng ngữ cảnh triển khai pipeline GSSC (Gather-Select-Structure-Compress), cung cấp giao diện quản lý ngữ cảnh thống nhất
- **NoteTool** (`hello_agents/tools/builtin/note_tool.py`): Công cụ ghi chú có cấu trúc, hỗ trợ quản lý bộ nhớ bền vững cho agent
- **TerminalTool** (`hello_agents/tools/builtin/terminal_tool.py`): Công cụ terminal, hỗ trợ các thao tác trên hệ thống tệp và truy xuất ngữ cảnh tức thời (just-in-time) cho agent

Các thành phần này cùng nhau tạo nên một giải pháp kỹ thuật ngữ cảnh hoàn chỉnh, là chìa khóa để triển khai quản lý tác vụ dài hạn và tìm kiếm kiểu agent (agentic search), và sẽ được giới thiệu chi tiết trong các phần sau.

Ngoài việc cài đặt framework, bạn còn cần cấu hình LLM API trong `.env`. Các ví dụ trong chương này chủ yếu sử dụng mô hình ngôn ngữ lớn để quản lý ngữ cảnh và ra quyết định thông minh.

Sau khi hoàn tất cấu hình, bạn có thể bắt đầu hành trình học tập của chương này!

## 9.1 Kỹ thuật Ngữ cảnh là gì

Sau nhiều năm Kỹ thuật Prompt (Prompt Engineering) trở thành trọng tâm của AI ứng dụng, một thuật ngữ mới đã nổi lên hàng đầu: **Kỹ thuật Ngữ cảnh (Context Engineering)**. Ngày nay, việc xây dựng hệ thống với mô hình ngôn ngữ không còn chỉ là tìm cách diễn đạt và dùng từ đúng trong prompt, mà là trả lời một câu hỏi vĩ mô hơn: **Cấu hình ngữ cảnh như thế nào thì dễ khiến mô hình tạo ra hành vi mà chúng ta mong đợi nhất?**

Cái gọi là "ngữ cảnh" (context) chỉ tập hợp các token được đưa vào khi lấy mẫu (sampling) một mô hình ngôn ngữ lớn (LLM). Vấn đề kỹ thuật cần giải quyết là **tối ưu hóa tính hữu dụng của những token này** dưới các ràng buộc cố hữu của LLM, nhằm ổn định thu được kết quả mong đợi. Để khai thác LLM một cách hiệu quả, thường cần phải "suy nghĩ theo ngữ cảnh" — tức là: tại bất kỳ lần gọi nào, hãy xem xét toàn bộ trạng thái mà LLM nhìn thấy và dự đoán hành vi mà trạng thái này có thể gây ra.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/9-figures/9-1.webp" alt="" width="85%"/>
  <p>Hình 9.1 Kỹ thuật prompt và Kỹ thuật ngữ cảnh</p>
</div>

Phần này sẽ khám phá kỹ thuật ngữ cảnh mới nổi và cung cấp một mô hình tư duy tinh gọn để xây dựng các agent **có thể kiểm soát và hiệu quả**.

**Kỹ thuật Ngữ cảnh so với Kỹ thuật Prompt**

Như thể hiện trong Hình 9.1, từ góc nhìn của các nhà cung cấp mô hình hàng đầu, kỹ thuật ngữ cảnh là sự tiến hóa tự nhiên của kỹ thuật prompt. Kỹ thuật prompt tập trung vào cách viết và tổ chức các chỉ thị (instruction) cho LLM để có kết quả tốt hơn (như viết system prompt và các chiến lược cấu trúc hóa); trong khi kỹ thuật ngữ cảnh là **cách lập kế hoạch và duy trì "tập thông tin (token) tối ưu" trong giai đoạn suy luận (inference)**, bao gồm không chỉ bản thân prompt mà còn tất cả thông tin khác sẽ đi vào cửa sổ ngữ cảnh (context window).

Trong giai đoạn đầu của kỹ thuật LLM, prompt thường là công việc chính, vì hầu hết các trường hợp sử dụng (ngoài trò chuyện hàng ngày) đều yêu cầu tối ưu prompt được tinh chỉnh cho phân loại đơn lượt hoặc sinh văn bản. Đúng như tên gọi, cốt lõi của kỹ thuật prompt là "cách viết prompt hiệu quả", đặc biệt là system prompt. Tuy nhiên, khi chúng ta bắt đầu kỹ thuật hóa các agent mạnh hơn, hoạt động trong khoảng thời gian dài hơn và qua nhiều vòng suy luận, chúng ta cần các chiến lược có thể quản lý **toàn bộ trạng thái ngữ cảnh** — bao gồm chỉ thị hệ thống, công cụ, MCP (Model Context Protocol), dữ liệu bên ngoài, lịch sử tin nhắn, v.v.

Một agent chạy trong vòng lặp sẽ liên tục sinh ra dữ liệu có thể liên quan đến vòng suy luận tiếp theo. Thông tin này phải được **tinh lọc định kỳ**. Do đó, "nghệ thuật và kỹ thuật" của kỹ thuật ngữ cảnh nằm ở việc **xác định nội dung nào nên đi vào cửa sổ ngữ cảnh có giới hạn** từ "vũ trụ thông tin ứng viên" đang không ngừng mở rộng.

## 9.2 Vì sao Kỹ thuật Ngữ cảnh lại quan trọng

Mặc dù các mô hình ngày càng nhanh hơn và có thể xử lý quy mô dữ liệu lớn hơn, chúng ta quan sát thấy rằng: giống như con người, LLM sẽ "lang thang" hoặc "bối rối" tại một thời điểm nào đó. Các benchmark kiểu "mò kim đáy bể" (needle-in-a-haystack) đã tiết lộ một hiện tượng: **sự mục ruỗng ngữ cảnh (context rot)** — khi số lượng token trong cửa sổ ngữ cảnh tăng lên, khả năng của mô hình trong việc nhớ chính xác thông tin từ ngữ cảnh lại giảm đi.

Các mô hình khác nhau có thể có đường cong suy giảm mượt mà hơn, nhưng đặc điểm này xuất hiện ở hầu hết mọi mô hình. Do đó, **ngữ cảnh phải được nhìn nhận như một tài nguyên hữu hạn với lợi ích cận biên giảm dần**. Cũng như con người có dung lượng bộ nhớ làm việc hữu hạn, LLM cũng có một "ngân sách chú ý" (attention budget). Mỗi token mới tiêu tốn một phần của ngân sách này, nên chúng ta cần cẩn thận hơn về việc token nào nên được cung cấp cho LLM.

Sự khan hiếm này không phải ngẫu nhiên, mà bắt nguồn từ những ràng buộc kiến trúc của LLM. Transformer cho phép mỗi token thiết lập liên kết với **tất cả** token trong ngữ cảnh, về mặt lý thuyết tạo thành \(n^2\) quan hệ chú ý theo cặp. Khi độ dài ngữ cảnh tăng lên, khả năng mô hình hóa các quan hệ theo cặp này của mô hình bị "kéo giãn mỏng đi", tự nhiên tạo ra sự căng thẳng giữa "quy mô ngữ cảnh" và "sự tập trung chú ý". Ngoài ra, các mẫu chú ý (attention pattern) của mô hình đến từ phân phối dữ liệu huấn luyện — chuỗi ngắn thường phổ biến hơn chuỗi dài, nên mô hình có ít kinh nghiệm với "phụ thuộc toàn ngữ cảnh" hơn và có ít tham số chuyên biệt hơn.

Các kỹ thuật như nội suy mã hóa vị trí (position encoding interpolation) có thể cho phép mô hình "thích nghi" với các chuỗi dài hơn so với lúc huấn luyện ở thời điểm suy luận, nhưng phải trả giá bằng một phần độ chính xác trong việc hiểu vị trí token. Nhìn chung, các yếu tố này cùng nhau tạo thành một **gradient hiệu năng** thay vì sự sụp đổ kiểu "vách đá": mô hình vẫn mạnh mẽ trong ngữ cảnh dài, nhưng so với ngữ cảnh ngắn, độ chính xác trong truy xuất thông tin và suy luận tầm xa của nó sẽ suy giảm.

Dựa trên thực tế trên, **kỹ thuật ngữ cảnh có ý thức** trở thành điều tất yếu để xây dựng các agent bền vững.

### 9.2.1 "Giải phẫu" một ngữ cảnh hiệu quả

Dưới ràng buộc "ngân sách chú ý hữu hạn", mục tiêu của kỹ thuật ngữ cảnh xuất sắc là: **tối đa hóa xác suất thu được kết quả mong đợi với càng ít token nhưng mật độ tín hiệu (signal density) càng cao càng tốt**. Trong thực hành, chúng tôi khuyến nghị kỹ thuật hóa xoay quanh các thành phần sau:

- **System Prompt**: Ngôn ngữ rõ ràng và thẳng thắn, với phân cấp thông tin ở độ cao "vừa phải". Các cạm bẫy thường gặp ở hai thái cực:
  - Cứng nhắc quá mức (over-hardcoding): Viết logic if-else phức tạp, dễ vỡ trong prompt, với chi phí bảo trì dài hạn cao và tính mong manh.
  - Quá mơ hồ: Chỉ cung cấp mục tiêu vĩ mô và hướng dẫn khái quát, thiếu **tín hiệu cụ thể** cho đầu ra mong đợi hoặc giả định sai về "ngữ cảnh chung được chia sẻ".
  Nên tổ chức prompt thành các phần (như <background_information>, <instructions>, hướng dẫn công cụ, mô tả đầu ra, v.v.), phân tách bằng XML/Markdown. Bất kể định dạng nào, điều theo đuổi là **"tập thông tin cần thiết tối thiểu" có thể phác họa đầy đủ hành vi mong đợi** ("tối thiểu" không đồng nghĩa với "ngắn nhất"). Trước tiên hãy chạy với mô hình tốt nhất trên prompt tối thiểu, rồi bổ sung chỉ thị và ví dụ rõ ràng dựa trên các dạng lỗi (failure mode).

- **Công cụ (Tools)**: Công cụ định nghĩa hợp đồng (contract) giữa agent và không gian thông tin/hành động, và phải thúc đẩy hiệu quả: chúng phải trả về thông tin **thân thiện với token** đồng thời khuyến khích hành vi agent hiệu quả. Công cụ nên:
  - Có trách nhiệm đơn nhất với độ chồng lấn thấp, ngữ nghĩa giao diện rõ ràng;
  - Bền vững trước lỗi;
  - Có mô tả tham số rõ ràng, không mập mờ, tận dụng đầy đủ thế mạnh của mô hình về diễn đạt và suy luận.
  Một dạng lỗi thường gặp là "bộ công cụ phình to": ranh giới chức năng mờ nhạt, khiến quyết định "dùng công cụ nào" tự nó trở nên mập mờ. **Nếu chính kỹ sư con người không phân biệt được nên dùng công cụ nào, đừng mong agent làm tốt hơn**. Cẩn thận xác định một "Bộ công cụ tối thiểu khả dụng (Minimum Viable Tool Set - MVTS)" thường có thể cải thiện đáng kể tính ổn định và khả năng bảo trì trong các tương tác dài hạn.

- **Ví dụ Few-shot**: Luôn khuyến nghị cung cấp ví dụ, nhưng không khuyến nghị nhồi nhét "tất cả điều kiện biên" vào prompt. Hãy cẩn thận chọn một tập ví dụ **đa dạng và điển hình** khắc họa trực tiếp "hành vi mong đợi". Đối với LLM, **một ví dụ hay đáng giá ngàn lời**.

Nguyên tắc hướng dẫn tổng thể là: **thông tin đầy đủ nhưng gọn gàng**. Như thể hiện trong Hình 9.2, đây là việc truy xuất động đi vào thời gian chạy (runtime).

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/9-figures/9-2.webp" alt="" width="85%"/>
  <p>Hình 9.2 Hiệu chỉnh system prompt</p>
</div>

### 9.2.2 Truy xuất Ngữ cảnh và Tìm kiếm kiểu Agent

Một định nghĩa ngắn gọn: **Agent = LLM tự động gọi công cụ trong một vòng lặp**. Khi năng lực của các mô hình nền tảng tăng lên, mức độ tự chủ của agent có thể được nâng cao: chúng có thể độc lập hơn trong việc khám phá các không gian vấn đề phức tạp và phục hồi từ lỗi.

Thực hành kỹ thuật đang dần chuyển từ "truy xuất một lần trước khi suy luận (truy xuất bằng embedding)" sang "**ngữ cảnh tức thời (Just-in-time - JIT)**". Cách sau không còn nạp trước toàn bộ dữ liệu liên quan, mà duy trì các **tham chiếu nhẹ (lightweight reference)** (đường dẫn tệp, truy vấn lưu trữ, URL, v.v.), nạp dữ liệu cần thiết một cách động thông qua công cụ ở thời gian chạy. Điều này cho phép mô hình viết các truy vấn có mục tiêu, cache những kết quả cần thiết, và phân tích khối lượng dữ liệu lớn bằng các lệnh như <code>head</code>/<code>tail</code> — mà không cần nhồi toàn bộ khối dữ liệu vào ngữ cảnh cùng một lúc. Mẫu hình nhận thức của nó gần với con người hơn: chúng ta không ghi nhớ toàn bộ thông tin, mà dùng các chỉ mục bên ngoài như hệ thống tệp, hộp thư, dấu trang để trích xuất theo nhu cầu.

Ngoài hiệu quả lưu trữ, **siêu dữ liệu (metadata) của tham chiếu** tự nó có thể giúp tinh lọc hành vi: phân cấp thư mục, quy ước đặt tên, dấu thời gian, v.v., tất cả đều ngầm truyền tải "mục đích và tính thời sự". Ví dụ, <code>tests/test_utils.py</code> và <code>src/core/test_utils.py</code> có hàm ý ngữ nghĩa khác nhau.

Cho phép agent tự động điều hướng và truy xuất cũng kích hoạt **tiết lộ dần dần (progressive disclosure)**: mỗi bước tương tác sinh ra ngữ cảnh mới, từ đó lại dẫn dắt quyết định tiếp theo — kích thước tệp gợi ý độ phức tạp, cách đặt tên gợi ý mục đích, dấu thời gian gợi ý mức độ liên quan. Agent có thể xây dựng sự hiểu biết từng lớp một, chỉ giữ lại "tập con cần thiết hiện tại" trong bộ nhớ làm việc, và dùng "ghi chú" để bổ sung bền vững, qua đó duy trì sự tập trung thay vì bị "kéo lê bởi tính toàn diện".

Sự đánh đổi là: khám phá ở thời gian chạy thường chậm hơn truy xuất được tính toán trước, và đòi hỏi thiết kế kỹ thuật "có quan điểm" (opinionated) để đảm bảo mô hình có công cụ và heuristic đúng đắn. Không có hướng dẫn, agent có thể lạm dụng công cụ, đuổi theo ngõ cụt, hoặc bỏ sót thông tin then chốt, gây lãng phí ngữ cảnh.

Trong nhiều tình huống, một **chiến lược lai (hybrid)** hiệu quả hơn: nạp trước một lượng nhỏ ngữ cảnh "giá trị cao" để đảm bảo tốc độ, rồi cho phép agent tiếp tục khám phá tự chủ theo nhu cầu. Việc lựa chọn ranh giới phụ thuộc vào tính động của tác vụ và yêu cầu về tính thời sự. Trong kỹ thuật, bạn có thể nạp trước các tệp như "mô tả quy ước dự án (như README/guide)", đồng thời cung cấp các nguyên hàm (primitive) như <code>glob</code>, <code>grep</code>, cho phép agent truy xuất các tệp cụ thể một cách tức thời, qua đó tránh được chi phí chìm (sunk cost) của các chỉ mục lỗi thời và cây cú pháp phức tạp.

### 9.2.3 Kỹ thuật Ngữ cảnh cho các Tác vụ Dài hạn

Các tác vụ dài hạn (long-horizon) đòi hỏi agent duy trì tính mạch lạc, tính nhất quán ngữ cảnh và định hướng mục tiêu trong các chuỗi hành động vượt quá cửa sổ ngữ cảnh. Ví dụ, di trú (migration) các codebase lớn, nghiên cứu có hệ thống kéo dài hàng giờ. Kỳ vọng tăng vô hạn cửa sổ ngữ cảnh không thể chữa được các vấn đề "ô nhiễm ngữ cảnh" (context pollution) và suy giảm mức độ liên quan, nên cần các phương pháp kỹ thuật đối mặt trực tiếp với những ràng buộc này: **Nén gọn (Compaction)**, **Ghi chú có cấu trúc (Structured note-taking)**, và **Kiến trúc sub-agent (Sub-agent architectures)**.

- **Nén gọn (Compaction)**
  - Định nghĩa: Khi một cuộc trò chuyện tiến gần giới hạn ngữ cảnh, thực hiện tóm tắt trung thực cao (high-fidelity) và khởi động lại một cửa sổ ngữ cảnh mới với bản tóm tắt để duy trì tính mạch lạc tầm xa.
  - Thực hành: Yêu cầu mô hình nén và giữ lại các quyết định kiến trúc, khiếm khuyết chưa giải quyết, chi tiết triển khai, loại bỏ các đầu ra công cụ lặp lại và nhiễu; cửa sổ mới mang theo bản tóm tắt đã nén + một vài hiện vật (artifact) gần đây có liên quan cao (như "các tệp truy cập gần đây").
  - Gợi ý điều chỉnh: Trước tiên tối ưu **recall (độ nhớ lại)** (đảm bảo không bỏ sót thông tin then chốt), rồi tối ưu **precision (độ chính xác)** (loại bỏ nội dung dư thừa); một cách nén "nhẹ nhàng" an toàn là dọn dẹp "các lệnh gọi công cụ và kết quả trong lịch sử sâu".

- **Ghi chú có cấu trúc (Structured note-taking)**
  - Định nghĩa: Còn gọi là "bộ nhớ agent" (agent memory). Agent ghi thông tin then chốt vào **kho lưu trữ bền vững nằm ngoài ngữ cảnh** với tần suất cố định, và kéo lại theo nhu cầu ở các giai đoạn sau.
  - Giá trị: Duy trì trạng thái và phụ thuộc bền vững với chi phí ngữ cảnh cực thấp. Ví dụ, duy trì danh sách TODO, tệp NOTES.md của dự án, chỉ mục các kết luận/phụ thuộc/vật cản then chốt, duy trì tiến độ và tính nhất quán qua hàng chục lệnh gọi công cụ và nhiều lần reset ngữ cảnh.
  - Lưu ý: Cũng hiệu quả không kém trong các tình huống không phải lập trình (như các tác vụ chiến lược dài hạn, quản lý mục tiêu và đếm thống kê trong game/mô phỏng). Kết hợp với <code>MemoryTool</code> từ Chương 8, có thể dễ dàng triển khai bộ nhớ ngoài dựa trên tệp/dựa trên vector và truy xuất ở thời gian chạy.

- **Kiến trúc sub-agent (Sub-agent architectures)**
  - Ý tưởng: Agent chính chịu trách nhiệm lập kế hoạch và tổng hợp ở cấp cao, trong khi nhiều sub-agent chuyên biệt mỗi cái đào sâu, gọi công cụ và khám phá trong các "cửa sổ ngữ cảnh sạch", cuối cùng chỉ trả về **bản tóm tắt cô đọng** (thường 1.000–2.000 token).
  - Lợi ích: Đạt được sự tách bạch mối quan tâm (separation of concerns). Các ngữ cảnh tìm kiếm phức tạp vẫn nằm trong nội bộ sub-agent, trong khi agent chính tập trung vào tích hợp và suy luận; phù hợp với các tác vụ nghiên cứu/phân tích phức tạp cần khám phá song song.
  - Kinh nghiệm: Các hệ thống nghiên cứu đa agent (multi-agent) công khai cho thấy mẫu hình này có ưu thế đáng kể so với baseline đơn agent trong các tác vụ nghiên cứu phức tạp.

Sự đánh đổi giữa các phương pháp có thể tuân theo các quy tắc kinh nghiệm sau:

- **Nén gọn**: Phù hợp với các tác vụ cần tính liên tục dài của cuộc trò chuyện, nhấn mạnh "chuyển tiếp" (relay) ngữ cảnh.
- **Ghi chú có cấu trúc**: Phù hợp với phát triển lặp và nghiên cứu có các mốc/kết quả theo giai đoạn.
- **Kiến trúc sub-agent**: Phù hợp với nghiên cứu và phân tích phức tạp có thể hưởng lợi từ khám phá song song.

Ngay cả khi năng lực mô hình tiếp tục cải thiện, "duy trì tính mạch lạc và sự tập trung trong các tương tác dài" vẫn là một thách thức cốt lõi trong việc xây dựng các agent bền vững. Kỹ thuật ngữ cảnh cẩn thận và có hệ thống sẽ giữ vững giá trị then chốt của nó về lâu dài.

## 9.3 Thực hành trong Hello-Agents: ContextBuilder

Phần này sẽ trình bày chi tiết thực hành kỹ thuật ngữ cảnh trong framework HelloAgents. Chúng ta sẽ dần dần chứng minh cách xây dựng một hệ thống quản lý ngữ cảnh cấp sản xuất (production-grade) từ động lực thiết kế, cấu trúc dữ liệu cốt lõi, chi tiết triển khai đến các trường hợp hoàn chỉnh. Triết lý thiết kế của ContextBuilder là "đơn giản và hiệu quả", loại bỏ độ phức tạp không cần thiết, lựa chọn một cách thống nhất dựa trên điểm số "mức độ liên quan + tính gần đây (recency)", phù hợp với định hướng kỹ thuật về tính mô-đun và khả năng bảo trì của Agent.

### 9.3.1 Động lực và Mục tiêu Thiết kế

Trước khi xây dựng ContextBuilder, trước tiên chúng ta cần làm rõ các mục tiêu thiết kế và giá trị cốt lõi của nó. Một hệ thống quản lý ngữ cảnh xuất sắc nên giải quyết các vấn đề then chốt sau:

1. **Điểm vào thống nhất (Unified Entry)**: Trừu tượng hóa "Gather-Select-Structure-Compress" thành một pipeline có thể tái sử dụng, giảm mã khuôn mẫu lặp lại trong các triển khai Agent. Thiết kế giao diện thống nhất này cho phép các nhà phát triển tránh phải viết lặp lại logic quản lý ngữ cảnh trong mỗi Agent.

2. **Hình thức ổn định (Stable Form)**: Xuất ra một mẫu ngữ cảnh với bộ khung cố định, thuận tiện cho gỡ lỗi, kiểm thử A/B và đánh giá. Chúng tôi áp dụng cấu trúc mẫu theo phân đoạn (sectioned template):
   - `[Role & Policies]`: Làm rõ định vị vai trò và các quy tắc hành vi của Agent
   - `[Task]`: Tác vụ cụ thể hiện cần hoàn thành
   - `[State]`: Trạng thái hiện tại và thông tin ngữ cảnh của Agent
   - `[Evidence]`: Thông tin bằng chứng được truy xuất từ các cơ sở tri thức bên ngoài
   - `[Context]`: Hội thoại lịch sử và các bộ nhớ liên quan
   - `[Output]`: Định dạng và yêu cầu đầu ra mong đợi

3. **Người gác ngân sách (Budget Guardian)**: Giữ lại thông tin giá trị cao nhất có thể trong phạm vi ngân sách token, cung cấp các chiến lược nén dự phòng (fallback) cho các ngữ cảnh vượt giới hạn. Điều này đảm bảo rằng ngay cả trong các tình huống có lượng thông tin khổng lồ, hệ thống vẫn có thể chạy ổn định.

4. **Quy tắc tối thiểu (Minimum Rules)**: Không đưa vào các chiều phân loại như nguồn (source)/độ ưu tiên (priority) để tránh sự gia tăng độ phức tạp. Thực tiễn cho thấy một cơ chế tính điểm đơn giản dựa trên mức độ liên quan và tính gần đây là đủ hiệu quả trong hầu hết các tình huống.

### 9.3.2 Các Cấu trúc Dữ liệu Cốt lõi

Việc triển khai ContextBuilder dựa trên hai cấu trúc dữ liệu cốt lõi định nghĩa cấu hình của hệ thống và các đơn vị thông tin.

(1) ContextPacket: Gói Thông tin Ứng viên

```python
from dataclasses import dataclass
from typing import Optional, Dict, Any
from datetime import datetime

@dataclass
class ContextPacket:
    """Gói thông tin ứng viên

    Attributes:
        content: Nội dung thông tin
        timestamp: Dấu thời gian
        token_count: Số lượng token
        relevance_score: Điểm mức độ liên quan (0.0-1.0)
        metadata: Siêu dữ liệu tùy chọn
    """
    content: str
    timestamp: datetime
    token_count: int
    relevance_score: float = 0.5
    metadata: Optional[Dict[str, Any]] = None

    def __post_init__(self):
        """Xử lý sau khởi tạo"""
        if self.metadata is None:
            self.metadata = {}
        # Đảm bảo điểm mức độ liên quan nằm trong khoảng hợp lệ
        self.relevance_score = max(0.0, min(1.0, self.relevance_score))
```

`ContextPacket` là đơn vị thông tin cơ bản trong hệ thống. Mỗi thông tin ứng viên được đóng gói thành một ContextPacket, chứa các thuộc tính cốt lõi như nội dung, dấu thời gian, số lượng token và điểm mức độ liên quan. Cấu trúc dữ liệu thống nhất này đơn giản hóa logic lựa chọn và sắp xếp về sau.

(2) ContextConfig: Quản lý Cấu hình

```python
@dataclass
class ContextConfig:
    """Cấu hình xây dựng ngữ cảnh

    Attributes:
        max_tokens: Số lượng token tối đa
        reserve_ratio: Tỷ lệ dành riêng cho chỉ thị hệ thống (0.0-1.0)
        min_relevance: Ngưỡng mức độ liên quan tối thiểu
        enable_compression: Có bật nén hay không
        recency_weight: Trọng số tính gần đây (0.0-1.0)
        relevance_weight: Trọng số mức độ liên quan (0.0-1.0)
    """
    max_tokens: int = 3000
    reserve_ratio: float = 0.2
    min_relevance: float = 0.1
    enable_compression: bool = True
    recency_weight: float = 0.3
    relevance_weight: float = 0.7

    def __post_init__(self):
        """Kiểm tra tính hợp lệ của các tham số cấu hình"""
        assert 0.0 <= self.reserve_ratio <= 1.0, "reserve_ratio must be in [0, 1] range"
        assert 0.0 <= self.min_relevance <= 1.0, "min_relevance must be in [0, 1] range"
        assert abs(self.recency_weight + self.relevance_weight - 1.0) < 1e-6, \
            "recency_weight + relevance_weight must equal 1.0"
```

`ContextConfig` đóng gói tất cả các tham số có thể cấu hình, khiến hành vi hệ thống có thể điều chỉnh linh hoạt. Đặc biệt đáng chú ý là tham số `reserve_ratio`, đảm bảo rằng thông tin then chốt như chỉ thị hệ thống luôn có đủ không gian và sẽ không bị các thông tin khác chèn ép ra ngoài.

### 9.3.3 Giải thích Chi tiết Pipeline GSSC

Cốt lõi của ContextBuilder là pipeline GSSC (Gather-Select-Structure-Compress), phân rã quá trình xây dựng ngữ cảnh thành bốn giai đoạn rõ ràng. Hãy đi sâu vào chi tiết triển khai của từng giai đoạn.

(1) Gather: Thu thập Thông tin Đa nguồn

Giai đoạn đầu tiên là thu thập thông tin ứng viên từ nhiều nguồn. Điểm mấu chốt của giai đoạn này là khả năng chịu lỗi (fault tolerance) và tính linh hoạt.

```python
def _gather(
    self,
    user_query: str,
    conversation_history: Optional[List[Message]] = None,
    system_instructions: Optional[str] = None,
    custom_packets: Optional[List[ContextPacket]] = None
) -> List[ContextPacket]:
    """Thu thập tất cả thông tin ứng viên

    Args:
        user_query: Truy vấn của người dùng
        conversation_history: Lịch sử hội thoại
        system_instructions: Chỉ thị hệ thống
        custom_packets: Các gói thông tin tùy chỉnh

    Returns:
        List[ContextPacket]: Danh sách thông tin ứng viên
    """
    packets = []

    # 1. Thêm chỉ thị hệ thống (ưu tiên cao nhất, không tính điểm)
    if system_instructions:
        packets.append(ContextPacket(
            content=system_instructions,
            timestamp=datetime.now(),
            token_count=self._count_tokens(system_instructions),
            relevance_score=1.0,  # Chỉ thị hệ thống luôn được giữ lại
            metadata={"type": "system_instruction", "priority": "high"}
        ))

    # 2. Truy xuất các bộ nhớ liên quan từ hệ thống bộ nhớ
    if self.memory_tool:
        try:
            memory_results = self.memory_tool.run({
                "action": "search",
                "query": user_query,
                "limit": 10,
                "min_importance": 0.3
            })
            # Phân tích kết quả bộ nhớ và chuyển đổi thành ContextPacket
            memory_packets = self._parse_memory_results(memory_results, user_query)
            packets.extend(memory_packets)
        except Exception as e:
            print(f"[WARNING] Memory retrieval failed: {e}")

    # 3. Truy xuất tri thức liên quan từ hệ thống RAG
    if self.rag_tool:
        try:
            rag_results = self.rag_tool.run({
                "action": "search",
                "query": user_query,
                "limit": 5,
                "min_score": 0.3
            })
            # Phân tích kết quả RAG và chuyển đổi thành ContextPacket
            rag_packets = self._parse_rag_results(rag_results, user_query)
            packets.extend(rag_packets)
        except Exception as e:
            print(f"[WARNING] RAG retrieval failed: {e}")

    # 4. Thêm lịch sử hội thoại (chỉ giữ lại N mục gần nhất)
    if conversation_history:
        recent_history = conversation_history[-5:]  # Mặc định giữ lại 5 mục gần nhất
        for msg in recent_history:
            packets.append(ContextPacket(
                content=f"{msg.role}: {msg.content}",
                timestamp=msg.timestamp if hasattr(msg, 'timestamp') else datetime.now(),
                token_count=self._count_tokens(msg.content),
                relevance_score=0.6,  # Mức độ liên quan cơ sở của các tin nhắn lịch sử
                metadata={"type": "conversation_history", "role": msg.role}
            ))

    # 5. Thêm các gói thông tin tùy chỉnh
    if custom_packets:
        packets.extend(custom_packets)

    print(f"[ContextBuilder] Collected {len(packets)} candidate information packages")
    return packets
```

Cách triển khai này thể hiện một số cân nhắc thiết kế quan trọng:

- **Cơ chế chịu lỗi**: Mỗi lệnh gọi nguồn dữ liệu bên ngoài được bọc trong try-except, đảm bảo rằng sự cố của một nguồn đơn lẻ không ảnh hưởng đến toàn bộ quy trình
- **Xử lý ưu tiên**: Chỉ thị hệ thống được đánh dấu là ưu tiên cao, đảm bảo chúng luôn được giữ lại
- **Giới hạn lịch sử**: Lịch sử hội thoại chỉ giữ các mục gần nhất, tránh việc cửa sổ ngữ cảnh bị thông tin lịch sử chiếm dụng

(2) Select: Lựa chọn Thông tin Thông minh

Giai đoạn thứ hai là tính điểm và lựa chọn thông tin ứng viên dựa trên mức độ liên quan và tính gần đây. Đây là cốt lõi của toàn bộ pipeline và trực tiếp quyết định chất lượng của ngữ cảnh cuối cùng.

```python
def _select(
    self,
    packets: List[ContextPacket],
    user_query: str,
    available_tokens: int
) -> List[ContextPacket]:
    """Lựa chọn các gói thông tin liên quan nhất

    Args:
        packets: Danh sách gói thông tin ứng viên
        user_query: Truy vấn của người dùng (để tính mức độ liên quan)
        available_tokens: Số lượng token khả dụng

    Returns:
        List[ContextPacket]: Danh sách gói thông tin đã được lựa chọn
    """
    # 1. Tách chỉ thị hệ thống và các thông tin khác
    system_packets = [p for p in packets if p.metadata.get("type") == "system_instruction"]
    other_packets = [p for p in packets if p.metadata.get("type") != "system_instruction"]

    # 2. Tính số token bị chỉ thị hệ thống chiếm dụng
    system_tokens = sum(p.token_count for p in system_packets)
    remaining_tokens = available_tokens - system_tokens

    if remaining_tokens <= 0:
        print("[WARNING] System instructions have occupied all token budget")
        return system_packets

    # 3. Tính điểm tổng hợp cho các thông tin khác
    scored_packets = []
    for packet in other_packets:
        # Tính điểm mức độ liên quan (nếu chưa được tính)
        if packet.relevance_score == 0.5:  # Giá trị mặc định, cần tính lại
            relevance = self._calculate_relevance(packet.content, user_query)
            packet.relevance_score = relevance

        # Tính điểm tính gần đây
        recency = self._calculate_recency(packet.timestamp)

        # Điểm tổng hợp = trọng số liên quan × mức độ liên quan + trọng số gần đây × tính gần đây
        combined_score = (
            self.config.relevance_weight * packet.relevance_score +
            self.config.recency_weight * recency
        )

        # Lọc bỏ thông tin dưới ngưỡng mức độ liên quan tối thiểu
        if packet.relevance_score >= self.config.min_relevance:
            scored_packets.append((combined_score, packet))

    # 4. Sắp xếp theo điểm giảm dần
    scored_packets.sort(key=lambda x: x[0], reverse=True)

    # 5. Lựa chọn tham lam (greedy): lấp từ điểm cao đến thấp cho đến khi đạt giới hạn token
    selected = system_packets.copy()
    current_tokens = system_tokens

    for score, packet in scored_packets:
        if current_tokens + packet.token_count <= available_tokens:
            selected.append(packet)
            current_tokens += packet.token_count
        else:
            # Ngân sách token đã đầy, dừng lựa chọn
            break

    print(f"[ContextBuilder] Selected {len(selected)} information packages, total {current_tokens} tokens")
    return selected

def _calculate_relevance(self, content: str, query: str) -> float:
    """Tính mức độ liên quan giữa nội dung và truy vấn

    Sử dụng thuật toán chồng lấn từ khóa đơn giản. Trong môi trường sản xuất, có thể thay bằng tính toán độ tương đồng vector.

    Args:
        content: Văn bản nội dung
        query: Văn bản truy vấn

    Returns:
        float: Điểm mức độ liên quan (0.0-1.0)
    """
    # Tách từ (triển khai đơn giản, có thể dùng bộ tách từ phức tạp hơn)
    content_words = set(content.lower().split())
    query_words = set(query.lower().split())

    if not query_words:
        return 0.0

    # Độ tương đồng Jaccard
    intersection = content_words & query_words
    union = content_words | query_words

    return len(intersection) / len(union) if union else 0.0

def _calculate_recency(self, timestamp: datetime) -> float:
    """Tính điểm tính gần đây theo thời gian

    Sử dụng mô hình suy giảm mũ (exponential decay), duy trì điểm cao trong 24 giờ, sau đó suy giảm dần.

    Args:
        timestamp: Dấu thời gian của thông tin

    Returns:
        float: Điểm tính gần đây (0.0-1.0)
    """
    import math

    age_hours = (datetime.now() - timestamp).total_seconds() / 3600

    # Suy giảm mũ: duy trì điểm cao trong 24 giờ, sau đó suy giảm dần
    decay_factor = 0.1  # Hệ số suy giảm
    recency_score = math.exp(-decay_factor * age_hours / 24)

    return max(0.1, min(1.0, recency_score))  # Giới hạn trong khoảng [0.1, 1.0]
```

Thuật toán cốt lõi của giai đoạn lựa chọn thể hiện một số cân nhắc kỹ thuật quan trọng:

- **Cơ chế tính điểm**: Sử dụng tổ hợp có trọng số của mức độ liên quan và tính gần đây, với các trọng số có thể cấu hình
- **Thuật toán tham lam**: Lấp từ điểm cao đến thấp, đảm bảo lựa chọn được thông tin giá trị nhất trong ngân sách hữu hạn
- **Cơ chế lọc**: Lọc bỏ thông tin chất lượng thấp thông qua tham số `min_relevance`

(3) Structure: Đầu ra có Cấu trúc

Giai đoạn thứ ba là tổ chức thông tin đã lựa chọn thành một mẫu ngữ cảnh có cấu trúc.

```python
def _structure(self, selected_packets: List[ContextPacket], user_query: str) -> str:
    """Tổ chức các gói thông tin đã lựa chọn thành mẫu ngữ cảnh có cấu trúc

    Args:
        selected_packets: Danh sách gói thông tin đã lựa chọn
        user_query: Truy vấn của người dùng

    Returns:
        str: Chuỗi ngữ cảnh có cấu trúc
    """
    # Nhóm theo loại
    system_instructions = []
    evidence = []
    context = []

    for packet in selected_packets:
        packet_type = packet.metadata.get("type", "general")

        if packet_type == "system_instruction":
            system_instructions.append(packet.content)
        elif packet_type in ["rag_result", "knowledge"]:
            evidence.append(packet.content)
        else:
            context.append(packet.content)

    # Xây dựng mẫu có cấu trúc
    sections = []

    # [Role & Policies]
    if system_instructions:
        sections.append("[Role & Policies]\n" + "\n".join(system_instructions))

    # [Task]
    sections.append(f"[Task]\n{user_query}")

    # [Evidence]
    if evidence:
        sections.append("[Evidence]\n" + "\n---\n".join(evidence))

    # [Context]
    if context:
        sections.append("[Context]\n" + "\n".join(context))

    # [Output]
    sections.append("[Output]\nPlease provide accurate, evidence-based answers based on the above information.")

    return "\n\n".join(sections)
```

Giai đoạn cấu trúc hóa tổ chức các gói thông tin rời rạc thành các phần rõ ràng. Thiết kế này có một số ưu điểm:

- **Tính dễ đọc**: Các phần rõ ràng giúp cả con người và mô hình dễ hiểu cấu trúc ngữ cảnh hơn
- **Khả năng gỡ lỗi**: Việc khoanh vùng vấn đề dễ dàng hơn, có thể nhanh chóng xác định khu vực nào có thông tin có vấn đề
- **Khả năng mở rộng**: Việc thêm nguồn thông tin mới chỉ cần tạo phần (section) mới

(4) Compress: Nén Dự phòng

Giai đoạn thứ tư là nén các ngữ cảnh vượt giới hạn.

```python
def _compress(self, context: str, max_tokens: int) -> str:
    """Nén ngữ cảnh vượt giới hạn

    Args:
        context: Ngữ cảnh gốc
        max_tokens: Giới hạn token tối đa

    Returns:
        str: Ngữ cảnh đã nén
    """
    current_tokens = self._count_tokens(context)

    if current_tokens <= max_tokens:
        return context  # Không cần nén

    print(f"[ContextBuilder] Context over limit ({current_tokens} > {max_tokens}), executing compression")

    # Nén theo phần: duy trì tính toàn vẹn cấu trúc
    sections = context.split("\n\n")
    compressed_sections = []
    current_total = 0

    for section in sections:
        section_tokens = self._count_tokens(section)

        if current_total + section_tokens <= max_tokens:
            # Giữ lại toàn bộ
            compressed_sections.append(section)
            current_total += section_tokens
        else:
            # Giữ lại một phần
            remaining_tokens = max_tokens - current_total
            if remaining_tokens > 50:  # Giữ lại ít nhất 50 token
                # Cắt ngắn đơn giản (có thể dùng tóm tắt bằng LLM trong môi trường sản xuất)
                truncated = self._truncate_text(section, remaining_tokens)
                compressed_sections.append(truncated + "\n[... Content compressed ...]")
            break

    compressed_context = "\n\n".join(compressed_sections)
    final_tokens = self._count_tokens(compressed_context)
    print(f"[ContextBuilder] Compression complete: {current_tokens} -> {final_tokens} tokens")

    return compressed_context

def _truncate_text(self, text: str, max_tokens: int) -> str:
    """Cắt ngắn văn bản đến số token chỉ định

    Args:
        text: Văn bản gốc
        max_tokens: Số lượng token tối đa

    Returns:
        str: Văn bản đã cắt ngắn
    """
    # Triển khai đơn giản: ước lượng theo tỷ lệ ký tự
    # Nên dùng bộ tách token chính xác trong môi trường sản xuất
    char_per_token = len(text) / self._count_tokens(text) if self._count_tokens(text) > 0 else 4
    max_chars = int(max_tokens * char_per_token)

    return text[:max_chars]

def _count_tokens(self, text: str) -> int:
    """Ước lượng số token của văn bản

    Args:
        text: Nội dung văn bản

    Returns:
        int: Số lượng token
    """
    # Ước lượng đơn giản: tiếng Trung 1 ký tự ≈ 1 token, tiếng Anh 1 từ ≈ 1.3 token
    # Nên dùng bộ tách token thực tế trong môi trường sản xuất
    chinese_chars = sum(1 for ch in text if '一' <= ch <= '鿿')
    english_words = len([w for w in text.split() if w])

    return int(chinese_chars + english_words * 1.3)
```

Thiết kế của giai đoạn nén thể hiện nguyên tắc "duy trì tính toàn vẹn cấu trúc". Ngay cả khi ngân sách token eo hẹp, nó vẫn cố gắng giữ lại thông tin then chốt từ mỗi phần.

### 9.3.4 Ví dụ Sử dụng Hoàn chỉnh

Bây giờ hãy trình bày cách sử dụng ContextBuilder trong các dự án thực tế thông qua một ví dụ hoàn chỉnh.

(1) Cách dùng Cơ bản

```python
from hello_agents.context import ContextBuilder, ContextConfig
from hello_agents.tools import MemoryTool, RAGTool
from hello_agents.core.message import Message
from datetime import datetime

# 1. Khởi tạo các công cụ
memory_tool = MemoryTool(user_id="user123")
rag_tool = RAGTool(knowledge_base_path="./knowledge_base")

# 2. Tạo ContextBuilder
config = ContextConfig(
    max_tokens=3000,
    reserve_ratio=0.2,
    min_relevance=0.2,
    enable_compression=True
)

builder = ContextBuilder(
    memory_tool=memory_tool,
    rag_tool=rag_tool,
    config=config
)

# 3. Chuẩn bị lịch sử hội thoại
conversation_history = [
    Message(content="I'm developing a data analysis tool", role="user", timestamp=datetime.now()),
    Message(content="Great! Data analysis tools usually need to handle large amounts of data. What tech stack do you plan to use?", role="assistant", timestamp=datetime.now()),
    Message(content="I plan to use Python and Pandas, and have completed the CSV reading module", role="user", timestamp=datetime.now()),
    Message(content="Good choice! Pandas is very powerful for data processing. Next you may need to consider data cleaning and transformation.", role="assistant", timestamp=datetime.now()),
]

# 4. Thêm một số bộ nhớ
memory_tool.run({
    "action": "add",
    "content": "User is developing a data analysis tool using Python and Pandas",
    "memory_type": "semantic",
    "importance": 0.8
})

memory_tool.run({
    "action": "add",
    "content": "Completed development of CSV reading module",
    "memory_type": "episodic",
    "importance": 0.7
})

# 5. Xây dựng ngữ cảnh
context = builder.build(
    user_query="How to optimize Pandas memory usage?",
    conversation_history=conversation_history,
    system_instructions="You are a senior Python data engineering consultant. Your answers need to: 1) Provide specific actionable advice 2) Explain technical principles 3) Provide code examples"
)

print("=" * 80)
print("Built context:")
print("=" * 80)
print(context)
print("=" * 80)
```

(2) Minh họa Kết quả Chạy

Sau khi chạy đoạn mã trên, bạn sẽ thấy đầu ra ngữ cảnh có cấu trúc như sau:

```
================================================================================
Built context:
================================================================================
[Role & Policies]
You are a senior Python data engineering consultant. Your answers need to: 1) Provide specific actionable advice 2) Explain technical principles 3) Provide code examples

[Task]
How to optimize Pandas memory usage?

[Evidence]
Core strategies for Pandas memory optimization include:
1. Use appropriate data types (such as category instead of object)
2. Read large files in chunks
3. Use chunksize parameter
---
Data type optimization can significantly reduce memory usage. For example, downgrading int64 to int32 can save 50% memory.

[Context]
user: I'm developing a data analysis tool
assistant: Great! Data analysis tools usually need to handle large amounts of data. What tech stack do you plan to use?
user: I plan to use Python and Pandas, and have completed the CSV reading module
assistant: Good choice! Pandas is very powerful for data processing. Next you may need to consider data cleaning and transformation.
Memory: User is developing a data analysis tool using Python and Pandas
Memory: Completed development of CSV reading module

[Output]
Please provide accurate, evidence-based answers based on the above information.
================================================================================
```

Ngữ cảnh có cấu trúc này chứa tất cả thông tin cần thiết:

- **[Role & Policies]**: Làm rõ vai trò và yêu cầu trả lời của AI
- **[Task]**: Diễn đạt rõ ràng câu hỏi của người dùng
- **[Evidence]**: Tri thức liên quan được truy xuất từ hệ thống RAG
- **[Context]**: Lịch sử hội thoại và các bộ nhớ liên quan, cung cấp đủ thông tin nền tảng
- **[Output]**: Hướng dẫn LLM cách tổ chức câu trả lời

(3) Tích hợp với Agent

Cuối cùng, hãy trình bày cách tích hợp ContextBuilder vào một Agent:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.context import ContextBuilder, ContextConfig
from hello_agents.tools import MemoryTool, RAGTool

class ContextAwareAgent(SimpleAgent):
    """Agent có khả năng nhận biết ngữ cảnh"""

    def __init__(self, name: str, llm: HelloAgentsLLM, **kwargs):
        super().__init__(name=name, llm=llm, system_prompt=kwargs.get("system_prompt", ""))

        # Khởi tạo trình xây dựng ngữ cảnh
        self.memory_tool = MemoryTool(user_id=kwargs.get("user_id", "default"))
        self.rag_tool = RAGTool(knowledge_base_path=kwargs.get("knowledge_base_path", "./kb"))

        self.context_builder = ContextBuilder(
            memory_tool=self.memory_tool,
            rag_tool=self.rag_tool,
            config=ContextConfig(max_tokens=4000)
        )

        self.conversation_history = []

    def run(self, user_input: str) -> str:
        """Chạy Agent, tự động xây dựng ngữ cảnh được tối ưu"""

        # 1. Dùng ContextBuilder để xây dựng ngữ cảnh được tối ưu
        optimized_context = self.context_builder.build(
            user_query=user_input,
            conversation_history=self.conversation_history,
            system_instructions=self.system_prompt
        )

        # 2. Gọi LLM với ngữ cảnh được tối ưu
        messages = [
            {"role": "system", "content": optimized_context},
            {"role": "user", "content": user_input}
        ]
        response = self.llm.invoke(messages)

        # 3. Cập nhật lịch sử hội thoại
        from hello_agents.core.message import Message
        from datetime import datetime

        self.conversation_history.append(
            Message(content=user_input, role="user", timestamp=datetime.now())
        )
        self.conversation_history.append(
            Message(content=response, role="assistant", timestamp=datetime.now())
        )

        # 4. Ghi lại các tương tác quan trọng vào hệ thống bộ nhớ
        self.memory_tool.run({
            "action": "add",
            "content": f"Q: {user_input}\nA: {response[:200]}...",  # Tóm tắt
            "memory_type": "episodic",
            "importance": 0.6
        })

        return response

# Ví dụ sử dụng
agent = ContextAwareAgent(
    name="Data Analysis Consultant",
    llm=HelloAgentsLLM(),
    system_prompt="You are a senior Python data engineering consultant.",
    user_id="user123",
    knowledge_base_path="./data_science_kb"
)

response = agent.run("How to optimize Pandas memory usage?")
print(response)
```

Thông qua cách tiếp cận này, ContextBuilder trở thành "bộ não quản lý ngữ cảnh" của Agent, tự động xử lý việc thu thập, lọc và tổ chức thông tin, cho phép Agent luôn suy luận và sinh nội dung dưới ngữ cảnh tối ưu.

### 9.3.5 Thực hành Tốt nhất và Khuyến nghị Tối ưu

Khi thực sự áp dụng ContextBuilder, các thực hành tốt nhất sau đây đáng lưu ý:

1. **Điều chỉnh động ngân sách token**: Điều chỉnh động `max_tokens` dựa trên độ phức tạp của tác vụ, dùng ngân sách nhỏ hơn cho tác vụ đơn giản, tăng ngân sách cho tác vụ phức tạp.

2. **Tối ưu tính toán mức độ liên quan**: Trong môi trường sản xuất, thay việc chồng lấn từ khóa đơn giản bằng tính toán độ tương đồng vector để nâng cao chất lượng truy xuất.

3. **Cơ chế cache**: Đối với chỉ thị hệ thống và nội dung cơ sở tri thức không thay đổi, triển khai cơ chế cache để tránh tính toán lặp lại.

4. **Giám sát và ghi log**: Ghi lại thông tin thống kê cho mỗi lần xây dựng ngữ cảnh (số lượng thông tin đã lựa chọn, tỷ lệ sử dụng token, v.v.) để phục vụ tối ưu về sau.

5. **Kiểm thử A/B**: Đối với các tham số then chốt (như trọng số mức độ liên quan, trọng số tính gần đây), tìm cấu hình tối ưu thông qua kiểm thử A/B.

## 9.4 NoteTool: Ghi chú có Cấu trúc

NoteTool là một thành phần bộ nhớ ngoài (external memory) có cấu trúc được cung cấp cho các "tác vụ dài hạn". Nó dùng các tệp Markdown làm vật mang tin, với YAML front matter ở phần đầu để ghi thông tin then chốt, và phần thân để ghi trạng thái, kết luận, vật cản và các mục hành động. Thiết kế này kết hợp khả năng đọc của con người, sự thân thiện với kiểm soát phiên bản (version control), và tính dễ tái đưa vào ngữ cảnh, khiến nó trở thành một công cụ quan trọng để xây dựng các agent dài hạn.

### 9.4.1 Triết lý Thiết kế và Các Tình huống Ứng dụng

Trước khi đi sâu vào chi tiết triển khai, hãy tìm hiểu triết lý thiết kế và các tình huống ứng dụng điển hình của NoteTool.

(1) Vì sao chúng ta cần NoteTool?

Ở Chương 8, chúng ta đã giới thiệu MemoryTool, cung cấp năng lực quản lý bộ nhớ mạnh mẽ. Tuy nhiên, MemoryTool chủ yếu tập trung vào **bộ nhớ hội thoại** — bộ nhớ làm việc ngắn hạn, bộ nhớ tình tiết (episodic memory) và bộ nhớ ngữ nghĩa (semantic memory). Đối với các **tác vụ theo dự án** cần theo dõi dài hạn và quản lý có cấu trúc, chúng ta cần một phương pháp ghi chép nhẹ hơn, thân thiện với con người hơn.

NoteTool lấp đầy khoảng trống này bằng cách cung cấp:

- **Ghi chép có cấu trúc**: Dùng định dạng Markdown + YAML, phù hợp cho cả phân tích bằng máy lẫn đọc và chỉnh sửa bởi con người
- **Thân thiện với phiên bản**: Định dạng văn bản thuần túy, tự nhiên hỗ trợ các hệ thống kiểm soát phiên bản như Git
- **Chi phí thấp**: Không cần thao tác cơ sở dữ liệu phức tạp, phù hợp cho việc theo dõi trạng thái nhẹ
- **Phân loại linh hoạt**: Tổ chức ghi chú linh hoạt thông qua `type` và `tags`, hỗ trợ truy xuất đa chiều

(2) Các Tình huống Ứng dụng Điển hình

NoteTool đặc biệt phù hợp với các tình huống sau:

**Tình huống 1: Theo dõi Dự án Dài hạn**

Hãy tưởng tượng một agent đang hỗ trợ một tác vụ tái cấu trúc (refactoring) codebase lớn, có thể mất nhiều ngày hoặc thậm chí nhiều tuần. NoteTool có thể ghi lại:

- `task_state`: Trạng thái và tiến độ tác vụ của giai đoạn hiện tại
- `conclusion`: Các kết luận then chốt sau khi mỗi giai đoạn kết thúc
- `blocker`: Các vấn đề và điểm nghẽn gặp phải
- `action`: Kế hoạch hành động tiếp theo

```python
# Ghi lại trạng thái tác vụ
notes.run({
    "action": "create",
    "title": "Refactoring Project - Phase 1",
    "content": "Completed refactoring of data model layer, test coverage reached 85%. Next will refactor business logic layer.",
    "note_type": "task_state",
    "tags": ["refactoring", "phase1"]
})

# Ghi lại vật cản
notes.run({
    "action": "create",
    "title": "Dependency Conflict Issue",
    "content": "Found some third-party library versions incompatible, need to resolve. Impact scope: 3 modules in business logic layer.",
    "note_type": "blocker",
    "tags": ["dependency", "urgent"]
})
```

**Tình huống 2: Quản lý Tác vụ Nghiên cứu**

Một trợ lý nghiên cứu thông minh đang tiến hành tổng quan tài liệu (literature review) có thể dùng NoteTool để ghi lại:

- Các quan điểm cốt lõi của mỗi bài báo (`conclusion`)
- Các chủ đề cần điều tra sâu (`action`)
- Các tài liệu tham khảo quan trọng (`reference`)

**Tình huống 3: Phối hợp với ContextBuilder**

Trước mỗi vòng hội thoại, Agent có thể truy xuất các ghi chú liên quan thông qua thao tác `search` hoặc `list` và đưa chúng vào ngữ cảnh:

```python
# Trong phương thức run của Agent
def run(self, user_input: str) -> str:
    # 1. Truy xuất các ghi chú liên quan
    relevant_notes = self.note_tool.run({
        "action": "search",
        "query": user_input,
        "limit": 3
    })

    # 2. Chuyển đổi nội dung ghi chú thành ContextPacket
    note_packets = []
    for note in relevant_notes:
        note_packets.append(ContextPacket(
            content=note['content'],
            timestamp=note['updated_at'],
            token_count=self._count_tokens(note['content']),
            relevance_score=0.7,
            metadata={"type": "note", "note_type": note['type']}
        ))

    # 3. Truyền ghi chú khi xây dựng ngữ cảnh
    context = self.context_builder.build(
        user_query=user_input,
        custom_packets=note_packets,
        ...
    )
```

### 9.4.2 Giải thích Chi tiết Định dạng Lưu trữ

NoteTool áp dụng định dạng lai của Markdown + YAML, cân bằng giữa cấu trúc và khả năng đọc.

(1) Định dạng Tệp Ghi chú

Mỗi ghi chú là một tệp `.md` độc lập với định dạng như sau:

```markdown
---
id: note_20250119_153000_0
title: Project Progress - Phase 1
type: task_state
tags: [refactoring, phase1, backend]
created_at: 2025-01-19T15:30:00
updated_at: 2025-01-19T15:30:00
---

# Project Progress - Phase 1

## Completion Status

Completed refactoring of data model layer, main changes include:

1. Unified entity class naming conventions
2. Introduced type hints to improve code maintainability
3. Optimized database query performance

## Test Coverage

- Unit test coverage: 85%
- Integration test coverage: 70%

## Next Steps

1. Refactor business logic layer
2. Resolve dependency conflict issues
3. Increase integration test coverage to 85%
```

Ưu điểm của định dạng này:

- **Siêu dữ liệu YAML**: Có thể phân tích bằng máy, hỗ trợ trích xuất và truy xuất trường (field) chính xác
- **Phần thân Markdown**: Con người đọc được, hỗ trợ định dạng phong phú (tiêu đề, danh sách, khối mã, v.v.)
- **Tên tệp làm ID**: Đơn giản hóa quản lý, tên tệp của mỗi ghi chú là định danh duy nhất của nó

(2) Tệp Chỉ mục

NoteTool duy trì một tệp `notes_index.json` để truy xuất và quản lý ghi chú nhanh chóng:

```json
{
  "note_20250119_153000_0": {
    "id": "note_20250119_153000_0",
    "title": "Project Progress - Phase 1",
    "type": "task_state",
    "tags": ["refactoring", "phase1", "backend"],
    "created_at": "2025-01-19T15:30:00",
    "updated_at": "2025-01-19T15:30:00",
    "file_path": "./notes/note_20250119_153000_0.md"
  }
}
```

Vai trò của tệp chỉ mục này:

- **Truy xuất nhanh**: Không cần mở từng tệp, tìm kiếm trực tiếp từ chỉ mục
- **Quản lý siêu dữ liệu**: Quản lý tập trung siêu dữ liệu của tất cả ghi chú
- **Kiểm tra tính toàn vẹn**: Có thể phát hiện tệp bị thiếu hoặc hỏng

### 9.4.3 Giải thích Chi tiết Các Thao tác Cốt lõi

NoteTool cung cấp bảy thao tác cốt lõi bao trùm việc quản lý toàn bộ vòng đời của ghi chú.

(1) create: Tạo Ghi chú

```python
def _create_note(
    self,
    title: str,
    content: str,
    note_type: str = "general",
    tags: Optional[List[str]] = None
) -> str:
    """Tạo ghi chú

    Args:
        title: Tiêu đề ghi chú
        content: Nội dung ghi chú (định dạng Markdown)
        note_type: Loại ghi chú (task_state/conclusion/blocker/action/reference/general)
        tags: Danh sách nhãn

    Returns:
        str: ID ghi chú
    """
    from datetime import datetime

    # 1. Sinh ID duy nhất
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    note_id = f"note_{timestamp}_{len(self.index)}"

    # 2. Xây dựng siêu dữ liệu
    metadata = {
        "id": note_id,
        "title": title,
        "type": note_type,
        "tags": tags or [],
        "created_at": datetime.now().isoformat(),
        "updated_at": datetime.now().isoformat()
    }

    # 3. Xây dựng nội dung tệp Markdown hoàn chỉnh
    md_content = self._build_markdown(metadata, content)

    # 4. Lưu vào tệp
    file_path = os.path.join(self.workspace, f"{note_id}.md")
    with open(file_path, 'w', encoding='utf-8') as f:
        f.write(md_content)

    # 5. Cập nhật chỉ mục
    metadata["file_path"] = file_path
    self.index[note_id] = metadata
    self._save_index()

    return note_id

def _build_markdown(self, metadata: Dict, content: str) -> str:
    """Xây dựng nội dung tệp Markdown (YAML + phần thân)"""
    import yaml

    # YAML front matter
    yaml_header = yaml.dump(metadata, allow_unicode=True, sort_keys=False)

    # Định dạng kết hợp
    return f"---\n{yaml_header}---\n\n{content}"
```

Ví dụ sử dụng:

```python
from hello_agents.tools import NoteTool

notes = NoteTool(workspace="./project_notes")

note_id = notes.run({
    "action": "create",
    "title": "Refactoring Project - Phase 1",
    "content": """## Completion Status
Completed refactoring of data model layer, test coverage reached 85%.

## Next Steps
Refactor business logic layer""",
    "note_type": "task_state",
    "tags": ["refactoring", "phase1"]
})

print(f"✅ Note created successfully, ID: {note_id}")
```

(2) read: Đọc Ghi chú

```python
def _read_note(self, note_id: str) -> Dict:
    """Đọc nội dung ghi chú

    Args:
        note_id: ID ghi chú

    Returns:
        Dict: Từ điển chứa siêu dữ liệu và nội dung
    """
    if note_id not in self.index:
        raise ValueError(f"Note does not exist: {note_id}")

    file_path = self.index[note_id]["file_path"]

    # Đọc tệp
    with open(file_path, 'r', encoding='utf-8') as f:
        raw_content = f.read()

    # Phân tích siêu dữ liệu YAML và phần thân Markdown
    metadata, content = self._parse_markdown(raw_content)

    return {
        "metadata": metadata,
        "content": content
    }

def _parse_markdown(self, raw_content: str) -> Tuple[Dict, str]:
    """Phân tích tệp Markdown (tách YAML và phần thân)"""
    import yaml

    # Tìm các dấu phân cách YAML
    parts = raw_content.split('---\n', 2)

    if len(parts) >= 3:
        # Có YAML front matter
        yaml_str = parts[1]
        content = parts[2].strip()
        metadata = yaml.safe_load(yaml_str)
    else:
        # Không có siêu dữ liệu, toàn bộ là phần thân
        metadata = {}
        content = raw_content.strip()

    return metadata, content
```

(3) update: Cập nhật Ghi chú

```python
def _update_note(
    self,
    note_id: str,
    title: Optional[str] = None,
    content: Optional[str] = None,
    note_type: Optional[str] = None,
    tags: Optional[List[str]] = None
) -> str:
    """Cập nhật ghi chú

    Args:
        note_id: ID ghi chú
        title: Tiêu đề mới (tùy chọn)
        content: Nội dung mới (tùy chọn)
        note_type: Loại mới (tùy chọn)
        tags: Nhãn mới (tùy chọn)

    Returns:
        str: Thông báo kết quả thao tác
    """
    if note_id not in self.index:
        raise ValueError(f"Note does not exist: {note_id}")

    # 1. Đọc ghi chú hiện có
    note = self._read_note(note_id)
    metadata = note["metadata"]
    old_content = note["content"]

    # 2. Cập nhật các trường
    if title:
        metadata["title"] = title
    if note_type:
        metadata["type"] = note_type
    if tags is not None:
        metadata["tags"] = tags
    if content is not None:
        old_content = content

    # Cập nhật dấu thời gian
    from datetime import datetime
    metadata["updated_at"] = datetime.now().isoformat()

    # 3. Xây dựng lại và lưu
    md_content = self._build_markdown(metadata, old_content)
    file_path = metadata["file_path"]

    with open(file_path, 'w', encoding='utf-8') as f:
        f.write(md_content)

    # 4. Cập nhật chỉ mục
    self.index[note_id] = metadata
    self._save_index()

    return f"✅ Note updated: {metadata['title']}"
```

(4) search: Tìm kiếm Ghi chú

```python
def _search_notes(
    self,
    query: str,
    limit: int = 10,
    note_type: Optional[str] = None,
    tags: Optional[List[str]] = None
) -> List[Dict]:
    """Tìm kiếm ghi chú

    Args:
        query: Từ khóa tìm kiếm
        limit: Giới hạn số lượng trả về
        note_type: Lọc theo loại (tùy chọn)
        tags: Lọc theo nhãn (tùy chọn)

    Returns:
        List[Dict]: Danh sách các ghi chú khớp
    """
    results = []
    query_lower = query.lower()

    for note_id, metadata in self.index.items():
        # Lọc theo loại
        if note_type and metadata.get("type") != note_type:
            continue

        # Lọc theo nhãn
        if tags:
            note_tags = set(metadata.get("tags", []))
            if not note_tags.intersection(tags):
                continue

        # Đọc nội dung ghi chú
        try:
            note = self._read_note(note_id)
            content = note["content"]
            title = metadata.get("title", "")

            # Tìm kiếm trong tiêu đề và nội dung
            if query_lower in title.lower() or query_lower in content.lower():
                results.append({
                    "note_id": note_id,
                    "title": title,
                    "type": metadata.get("type"),
                    "tags": metadata.get("tags", []),
                    "content": content,
                    "updated_at": metadata.get("updated_at")
                })
        except Exception as e:
            print(f"[WARNING] Failed to read note {note_id}: {e}")
            continue

    # Sắp xếp theo thời gian cập nhật
    results.sort(key=lambda x: x["updated_at"], reverse=True)

    return results[:limit]
```

(5) list: Liệt kê Ghi chú

```python
def _list_notes(
    self,
    note_type: Optional[str] = None,
    tags: Optional[List[str]] = None,
    limit: int = 20
) -> List[Dict]:
    """Liệt kê ghi chú (theo thứ tự thời gian cập nhật giảm dần)

    Args:
        note_type: Lọc theo loại (tùy chọn)
        tags: Lọc theo nhãn (tùy chọn)
        limit: Giới hạn số lượng trả về

    Returns:
        List[Dict]: Danh sách siêu dữ liệu ghi chú
    """
    results = []

    for note_id, metadata in self.index.items():
        # Lọc theo loại
        if note_type and metadata.get("type") != note_type:
            continue

        # Lọc theo nhãn
        if tags:
            note_tags = set(metadata.get("tags", []))
            if not note_tags.intersection(tags):
                continue

        results.append(metadata)

    # Sắp xếp theo thời gian cập nhật
    results.sort(key=lambda x: x.get("updated_at", ""), reverse=True)

    return results[:limit]
```

(6) summary: Tóm tắt Ghi chú

```python
def _summary(self) -> Dict[str, Any]:
    """Sinh thống kê tóm tắt ghi chú

    Returns:
        Dict: Thông tin thống kê
    """
    total_count = len(self.index)

    # Đếm theo loại
    type_counts = {}
    for metadata in self.index.values():
        note_type = metadata.get("type", "general")
        type_counts[note_type] = type_counts.get(note_type, 0) + 1

    # Các ghi chú được cập nhật gần đây
    recent_notes = sorted(
        self.index.values(),
        key=lambda x: x.get("updated_at", ""),
        reverse=True
    )[:5]

    return {
        "total_notes": total_count,
        "type_distribution": type_counts,
        "recent_notes": [
            {
                "id": note["id"],
                "title": note.get("title", ""),
                "type": note.get("type"),
                "updated_at": note.get("updated_at")
            }
            for note in recent_notes
        ]
    }
```

(7) delete: Xóa Ghi chú

```python
def _delete_note(self, note_id: str) -> str:
    """Xóa ghi chú

    Args:
        note_id: ID ghi chú

    Returns:
        str: Thông báo kết quả thao tác
    """
    if note_id not in self.index:
        raise ValueError(f"Note does not exist: {note_id}")

    # 1. Xóa tệp
    file_path = self.index[note_id]["file_path"]
    if os.path.exists(file_path):
        os.remove(file_path)

    # 2. Loại bỏ khỏi chỉ mục
    title = self.index[note_id].get("title", note_id)
    del self.index[note_id]
    self._save_index()

    return f"✅ Note deleted: {title}"
```

### 9.4.4 Tích hợp Sâu với ContextBuilder

Sức mạnh thực sự của NoteTool nằm ở việc sử dụng kết hợp với ContextBuilder. Hãy trình bày sự tích hợp này thông qua một nghiên cứu tình huống hoàn chỉnh.

(1) Thiết lập Tình huống

Giả sử chúng ta đang xây dựng một trợ lý dự án dài hạn cần:

1. Ghi lại tiến độ theo giai đoạn của dự án
2. Theo dõi các vấn đề còn tồn đọng
3. Tự động xem lại các ghi chú liên quan trong mỗi cuộc hội thoại
4. Cung cấp các khuyến nghị mạch lạc dựa trên ghi chú lịch sử

(2) Ví dụ Triển khai

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.context import ContextBuilder, ContextConfig, ContextPacket
from hello_agents.tools import MemoryTool, RAGTool, NoteTool
from datetime import datetime

class ProjectAssistant(SimpleAgent):
    """Trợ lý dự án dài hạn, tích hợp NoteTool và ContextBuilder"""

    def __init__(self, name: str, project_name: str, **kwargs):
        super().__init__(name=name, llm=HelloAgentsLLM(), **kwargs)

        self.project_name = project_name

        # Khởi tạo các công cụ
        self.memory_tool = MemoryTool(user_id=project_name)
        self.rag_tool = RAGTool(knowledge_base_path=f"./{project_name}_kb")
        self.note_tool = NoteTool(workspace=f"./{project_name}_notes")

        # Khởi tạo trình xây dựng ngữ cảnh
        self.context_builder = ContextBuilder(
            memory_tool=self.memory_tool,
            rag_tool=self.rag_tool,
            config=ContextConfig(max_tokens=4000)
        )

        self.conversation_history = []

    def run(self, user_input: str, note_as_action: bool = False) -> str:
        """Chạy trợ lý, tự động tích hợp ghi chú"""

        # 1. Truy xuất các ghi chú liên quan từ NoteTool
        relevant_notes = self._retrieve_relevant_notes(user_input)

        # 2. Chuyển đổi ghi chú thành ContextPacket
        note_packets = self._notes_to_packets(relevant_notes)

        # 3. Xây dựng ngữ cảnh được tối ưu
        context = self.context_builder.build(
            user_query=user_input,
            conversation_history=self.conversation_history,
            system_instructions=self._build_system_instructions(),
            custom_packets=note_packets
        )

        # 4. Gọi LLM
        response = self.llm.invoke(context)

        # 5. Nếu cần, ghi lại tương tác dưới dạng ghi chú
        if note_as_action:
            self._save_as_note(user_input, response)

        # 6. Cập nhật lịch sử hội thoại
        self._update_history(user_input, response)

        return response

    def _retrieve_relevant_notes(self, query: str, limit: int = 3) -> List[Dict]:
        """Truy xuất các ghi chú liên quan"""
        try:
            # Ưu tiên truy xuất các ghi chú loại blocker và action
            blockers = self.note_tool.run({
                "action": "list",
                "note_type": "blocker",
                "limit": 2
            })

            # Tìm kiếm chung
            search_results = self.note_tool.run({
                "action": "search",
                "query": query,
                "limit": limit
            })

            # Gộp và khử trùng lặp
            all_notes = {note['note_id']: note for note in blockers + search_results}
            return list(all_notes.values())[:limit]

        except Exception as e:
            print(f"[WARNING] Note retrieval failed: {e}")
            return []

    def _notes_to_packets(self, notes: List[Dict]) -> List[ContextPacket]:
        """Chuyển đổi ghi chú thành các gói ngữ cảnh"""
        packets = []

        for note in notes:
            content = f"[Note: {note['title']}]\n{note['content']}"

            packets.append(ContextPacket(
                content=content,
                timestamp=datetime.fromisoformat(note['updated_at']),
                token_count=len(content) // 4,  # Ước lượng đơn giản
                relevance_score=0.75,  # Ghi chú có mức độ liên quan cao
                metadata={
                    "type": "note",
                    "note_type": note['type'],
                    "note_id": note['note_id']
                }
            ))

        return packets

    def _save_as_note(self, user_input: str, response: str):
        """Lưu tương tác dưới dạng ghi chú"""
        try:
            # Xác định loại ghi chú cần lưu
            if "problem" in user_input.lower() or "blocker" in user_input.lower():
                note_type = "blocker"
            elif "plan" in user_input.lower() or "next" in user_input.lower():
                note_type = "action"
            else:
                note_type = "conclusion"

            self.note_tool.run({
                "action": "create",
                "title": f"{user_input[:30]}...",
                "content": f"## Question\n{user_input}\n\n## Analysis\n{response}",
                "note_type": note_type,
                "tags": [self.project_name, "auto_generated"]
            })

        except Exception as e:
            print(f"[WARNING] Failed to save note: {e}")

    def _build_system_instructions(self) -> str:
        """Xây dựng chỉ thị hệ thống"""
        return f"""You are a long-term assistant for the {self.project_name} project.

Your responsibilities:
1. Provide coherent recommendations based on historical notes
2. Track project progress and pending issues
3. Reference relevant historical notes when answering
4. Provide specific, actionable next-step recommendations

Notes:
- Prioritize issues marked as blockers
- Indicate source of basis in recommendations (notes, memory, or knowledge base)
- Maintain awareness of overall project progress"""

    def _update_history(self, user_input: str, response: str):
        """Cập nhật lịch sử hội thoại"""
        from hello_agents.core.message import Message

        self.conversation_history.append(
            Message(content=user_input, role="user", timestamp=datetime.now())
        )
        self.conversation_history.append(
            Message(content=response, role="assistant", timestamp=datetime.now())
        )

        # Giới hạn độ dài lịch sử
        if len(self.conversation_history) > 10:
            self.conversation_history = self.conversation_history[-10:]

# Ví dụ sử dụng
assistant = ProjectAssistant(
    name="Project Assistant",
    project_name="data_pipeline_refactoring"
)

# Tương tác lần đầu: Ghi lại trạng thái dự án
response = assistant.run(
    "We have completed refactoring of the data model layer, test coverage reached 85%. Next plan is to refactor the business logic layer.",
    note_as_action=True
)

# Tương tác lần hai: Nêu vấn đề
response = assistant.run(
    "When refactoring the business logic layer, I encountered dependency version conflict issues. How should I resolve this?"
)

# Xem tóm tắt ghi chú
summary = assistant.note_tool.run({"action": "summary"})
print(summary)
```

(3) Minh họa Kết quả Chạy

```bash
[ContextBuilder] Collected 8 candidate information packages
[ContextBuilder] Selected 7 information packages, total 3500 tokens

✅ Assistant answer:

I noticed this issue was mentioned in your previously recorded notes. According to the note [Refactoring Project - Phase 1], your current test coverage has reached 85%, which is a good foundation.

Regarding the dependency version conflict issue, I recommend:

1. **Use virtual environment isolation**: Create an independent virtual environment for the business logic layer to avoid dependency conflicts with other modules
2. **Lock versions**: Explicitly specify exact versions of all dependencies in requirements.txt
3. **Use pipdeptree**: Analyze the dependency tree to find the root cause of conflicts

I will mark this issue as a blocker and recommend prioritizing its resolution.

[Source: Note note_20250119_153000_0, Project knowledge base]

---

📋 Note summary:
{
  "total_notes": 2,
  "type_distribution": {
    "action": 1,
    "blocker": 1
  },
  "recent_notes": [
    {
      "id": "note_20250119_154500_1",
      "title": "When refactoring the business logic layer, I encountered dependency version conflict issues...",
      "type": "blocker",
      "updated_at": "2025-01-19T15:45:00"
    },
    {
      "id": "note_20250119_153000_0",
      "title": "We have completed refactoring of the data model layer...",
      "type": "action",
      "updated_at": "2025-01-19T15:30:00"
    }
  ]
}
```

### 9.4.5 Thực hành Tốt nhất

Khi thực sự sử dụng NoteTool, các thực hành tốt nhất sau đây có thể giúp bạn xây dựng các agent dài hạn (long-horizon) mạnh mẽ hơn:

1. **Phân loại ghi chú hợp lý**:
   - `task_state`: Ghi lại tiến độ và trạng thái theo giai đoạn
   - `conclusion`: Ghi lại các kết luận và phát hiện quan trọng
   - `blocker`: Ghi lại các vấn đề gây tắc nghẽn, ưu tiên cao nhất
   - `action`: Ghi lại các kế hoạch hành động tiếp theo
   - `reference`: Ghi lại các tài liệu tham khảo quan trọng

2. **Dọn dẹp và lưu trữ định kỳ**:
   - Với các blocker đã được giải quyết, cập nhật thành conclusion
   - Với các action đã lỗi thời, xóa hoặc cập nhật kịp thời
   - Dùng tag để quản lý phiên bản, chẳng hạn `["v1.0", "completed"]`

3. **Phối hợp với ContextBuilder**:
   - Truy xuất các ghi chú liên quan trước mỗi vòng hội thoại
   - Đặt các điểm liên quan khác nhau dựa trên loại ghi chú (blocker > action > conclusion)
   - Giới hạn số lượng ghi chú để tránh quá tải ngữ cảnh

4. **Cộng tác người-máy**:
   - Ghi chú ở định dạng Markdown mà con người đọc được, hỗ trợ chỉnh sửa thủ công
   - Dùng Git để kiểm soát phiên bản nhằm theo dõi sự tiến hóa của ghi chú
   - Ở các giai đoạn then chốt, xem xét thủ công các ghi chú do Agent tạo ra

5. **Quy trình tự động hóa**:
   - Định kỳ tạo báo cáo tóm tắt ghi chú
   - Tự động tạo tài liệu tiến độ dự án dựa trên ghi chú
   - Đồng bộ nội dung ghi chú sang các hệ thống khác (như Notion, Confluence)

## 9.5 TerminalTool: Truy cập Hệ thống Tệp Tức thời

Trong các chương trước, chúng ta đã giới thiệu MemoryTool và RAGTool, lần lượt cung cấp khả năng ghi nhớ hội thoại và truy xuất tri thức. Tuy nhiên, trong nhiều tình huống thực tế, agent cần **truy cập và khám phá hệ thống tệp một cách tức thời** — xem các tệp log, phân tích cấu trúc codebase, truy xuất các tệp cấu hình, v.v. Đây chính là lúc TerminalTool phát huy tác dụng.

TerminalTool cung cấp cho agent **khả năng thực thi dòng lệnh an toàn**, hỗ trợ các lệnh xử lý hệ thống tệp và văn bản thông dụng, đồng thời đảm bảo an toàn hệ thống thông qua các cơ chế bảo mật nhiều lớp. Thiết kế này hiện thực hóa khái niệm "ngữ cảnh tức thời (Just-in-time - JIT)" đã đề cập ở Mục 9.2.2 — agent không cần nạp trước toàn bộ tệp, mà khám phá và truy xuất theo nhu cầu.

### 9.5.1 Triết lý Thiết kế và Các Cơ chế Bảo mật

(1) Tại sao chúng ta cần TerminalTool?

Khi xây dựng các agent dài hạn (long-horizon), chúng ta thường gặp các tình huống sau:

**Tình huống 1: Khám phá Codebase**

Một trợ lý phát triển cần giúp người dùng hiểu cấu trúc của một codebase lớn:

```python
# Cách truyền thống: Lập chỉ mục trước toàn bộ tệp (chi phí cao, có thể lỗi thời)
rag_tool.add_document("./project/**/*.py")  # Tốn thời gian, chiếm nhiều dung lượng lưu trữ

# Cách của TerminalTool: Khám phá tức thời
terminal.run({"command": "find . -name '*.py' -type f"})  # Nhanh, thời gian thực
terminal.run({"command": "grep -r 'class UserService' ."})  # Định vị chính xác
terminal.run({"command": "head -n 50 src/services/user.py"})  # Xem theo nhu cầu
```

**Tình huống 2: Phân tích Tệp Log**

Một trợ lý vận hành cần phân tích log ứng dụng:

```python
# Kiểm tra kích thước tệp log
terminal.run({"command": "ls -lh /var/log/app.log"})

# Xem các log lỗi mới nhất
terminal.run({"command": "tail -n 100 /var/log/app.log | grep ERROR"})

# Thống kê phân bố các loại lỗi
terminal.run({"command": "grep ERROR /var/log/app.log | cut -d':' -f3 | sort | uniq -c"})
```

**Tình huống 3: Xem trước Tệp Dữ liệu**

Một trợ lý phân tích dữ liệu cần nhanh chóng hiểu cấu trúc của các tệp dữ liệu:

```python
# Xem vài dòng đầu của tệp CSV
terminal.run({"command": "head -n 5 data/sales.csv"})

# Đếm số dòng
terminal.run({"command": "wc -l data/*.csv"})

# Xem tên các cột
terminal.run({"command": "head -n 1 data/sales.csv | tr ',' '\n'"})
```

Đặc điểm chung của các tình huống này là: **cần truy cập hệ thống tệp theo thời gian thực, nhẹ nhàng, thay vì lập chỉ mục trước và vector hóa**. TerminalTool được thiết kế đúng cho quy trình làm việc "khám phá" này.

(2) Giải thích Chi tiết Cơ chế Bảo mật

Cho phép agent thực thi lệnh là một khả năng mạnh mẽ nhưng nguy hiểm. TerminalTool đảm bảo an toàn hệ thống thông qua các cơ chế bảo mật nhiều lớp:

**Lớp thứ nhất: Danh sách trắng Lệnh (Command Whitelist)**

Chỉ cho phép các lệnh chỉ đọc an toàn, hoàn toàn cấm mọi thao tác có thể sửa đổi hệ thống:

```python
ALLOWED_COMMANDS = {
    # File listing and information
    'ls', 'dir', 'tree',
    # File content viewing
    'cat', 'head', 'tail', 'less', 'more',
    # File search
    'find', 'grep', 'egrep', 'fgrep',
    # Text processing
    'wc', 'sort', 'uniq', 'cut', 'awk', 'sed',
    # Directory operations
    'pwd', 'cd',
    # File information
    'file', 'stat', 'du', 'df',
    # Others
    'echo', 'which', 'whereis',
}
```

Nếu agent cố gắng thực thi các lệnh nằm ngoài danh sách trắng, nó sẽ bị từ chối ngay lập tức:

```python
terminal.run({"command": "rm -rf /"})
# ❌ Command not allowed: rm
# Allowed commands: cat, cd, cut, dir, du, ...
```

**Lớp thứ hai: Giới hạn Thư mục Làm việc (Sandbox)**

TerminalTool chỉ có thể truy cập thư mục làm việc được chỉ định và các thư mục con của nó, không thể truy cập các phần khác của hệ thống:

```python
# Chỉ định thư mục làm việc khi khởi tạo
terminal = TerminalTool(workspace="./project")

# Được phép: Truy cập các tệp bên trong thư mục làm việc
terminal.run({"command": "cat ./src/main.py"})  # ✅

# Bị cấm: Truy cập các tệp ngoài thư mục làm việc
terminal.run({"command": "cat /etc/passwd"})  # ❌ Not allowed to access paths outside working directory

# Bị cấm: Thoát ra ngoài thông qua ..
terminal.run({"command": "cd ../../../etc"})  # ❌ Not allowed to access paths outside working directory
```

Cơ chế hộp cát (sandbox) này đảm bảo rằng ngay cả khi hành vi của agent bất thường, nó cũng không thể ảnh hưởng đến các phần khác của hệ thống.

**Lớp thứ ba: Kiểm soát Thời gian chờ (Timeout)**

Mỗi lệnh có một giới hạn thời gian thực thi để ngăn chặn vòng lặp vô hạn hoặc cạn kiệt tài nguyên:

```python
terminal = TerminalTool(
    workspace="./project",
    timeout=30  # 30 second timeout
)

# Nếu thời gian thực thi lệnh vượt quá 30 giây
terminal.run({"command": "find / -name '*.log'"})
# ❌ Command execution timeout (exceeded 30 seconds)
```

**Lớp thứ tư: Giới hạn Kích thước Đầu ra**

Giới hạn kích thước đầu ra của lệnh để ngăn tràn bộ nhớ:

```python
terminal = TerminalTool(
    workspace="./project",
    max_output_size=10 * 1024 * 1024  # 10MB
)

# Nếu đầu ra vượt quá 10MB
terminal.run({"command": "cat huge_file.log"})
# ... (first 10MB of content) ...
# ⚠️ Output truncated (exceeded 10485760 bytes)
```

Thông qua bốn lớp cơ chế bảo mật này, TerminalTool vừa cung cấp khả năng mạnh mẽ vừa tối đa hóa an toàn hệ thống.

### 9.5.2 Giải thích Chi tiết Chức năng Cốt lõi

Việc triển khai TerminalTool tập trung vào hai chức năng cốt lõi: thực thi lệnh và điều hướng thư mục.

(1) Thực thi Lệnh

Phương thức cốt lõi `_execute_command` chịu trách nhiệm thực sự thực thi các lệnh:

```python
def _execute_command(self, command: str) -> str:
    """Thực thi lệnh"""
    try:
        # Thực thi lệnh trong thư mục hiện tại
        result = subprocess.run(
            command,
            shell=True,
            cwd=str(self.current_dir),  # Thực thi trong thư mục làm việc hiện tại
            capture_output=True,
            text=True,
            timeout=self.timeout,
            env=os.environ.copy()
        )

        # Gộp đầu ra chuẩn và lỗi chuẩn
        output = result.stdout
        if result.stderr:
            output += f"\n[stderr]\n{result.stderr}"

        # Kiểm tra kích thước đầu ra
        if len(output) > self.max_output_size:
            output = output[:self.max_output_size]
            output += f"\n\n⚠️ Output truncated (exceeded {self.max_output_size} bytes)"

        # Thêm thông tin mã trả về
        if result.returncode != 0:
            output = f"⚠️ Command return code: {result.returncode}\n\n{output}"

        return output if output else "✅ Command executed successfully (no output)"

    except subprocess.TimeoutExpired:
        return f"❌ Command execution timeout (exceeded {self.timeout} seconds)"
    except Exception as e:
        return f"❌ Command execution failed: {e}"
```

Các điểm mấu chốt của cách triển khai này:

- **Nhận biết thư mục hiện tại**: Dùng tham số `cwd` để thực thi lệnh trong đúng thư mục
- **Xử lý lỗi**: Bắt và gộp lỗi chuẩn, cung cấp thông tin chẩn đoán đầy đủ
- **Kiểm tra mã trả về**: Các mã trả về khác 0 được đánh dấu là cảnh báo
- **Thiết kế chịu lỗi**: Timeout và ngoại lệ được xử lý đúng cách, không khiến agent gặp sự cố

(2) Điều hướng Thư mục

Việc xử lý đặc biệt lệnh `cd` hỗ trợ agent điều hướng trong hệ thống tệp:

```python
def _handle_cd(self, parts: List[str]) -> str:
    """Xử lý lệnh cd"""
    if not self.allow_cd:
        return "❌ cd command is disabled"

    if len(parts) < 2:
        # cd không có tham số, trả về thư mục hiện tại
        return f"Current directory: {self.current_dir}"

    target_dir = parts[1]

    # Xử lý đường dẫn tương đối
    if target_dir == "..":
        new_dir = self.current_dir.parent
    elif target_dir == ".":
        new_dir = self.current_dir
    elif target_dir == "~":
        new_dir = self.workspace
    else:
        new_dir = (self.current_dir / target_dir).resolve()

    # Kiểm tra xem có nằm trong thư mục làm việc không
    try:
        new_dir.relative_to(self.workspace)
    except ValueError:
        return f"❌ Not allowed to access paths outside working directory: {new_dir}"

    # Kiểm tra xem thư mục có tồn tại không
    if not new_dir.exists():
        return f"❌ Directory does not exist: {new_dir}"

    if not new_dir.is_dir():
        return f"❌ Not a directory: {new_dir}"

    # Cập nhật thư mục hiện tại
    self.current_dir = new_dir
    return f"✅ Switched to directory: {self.current_dir}"
```

Thiết kế này hỗ trợ agent khám phá hệ thống tệp theo nhiều bước:

```python
# Bước 1: Xem cấu trúc dự án
terminal.run({"command": "ls -la"})

# Bước 2: Đi vào thư mục mã nguồn
terminal.run({"command": "cd src"})

# Bước 3: Tìm các tệp cụ thể
terminal.run({"command": "find . -name '*service*.py'"})

# Bước 4: Xem nội dung tệp
terminal.run({"command": "cat user_service.py"})
```

### 9.5.3 Các Mẫu hình Sử dụng Điển hình

TerminalTool hỗ trợ nhiều mẫu hình thao tác hệ thống tệp thông dụng.

(1) Điều hướng Khám phá

Agent có thể khám phá codebase từng bước như các nhà phát triển con người:

```python
from hello_agents.tools import TerminalTool

terminal = TerminalTool(workspace="./my_project")

# Bước 1: Xem thư mục gốc của dự án
print(terminal.run({"command": "ls -la"}))
"""
total 24
drwxr-xr-x  6 user  staff   192 Jan 19 16:00 .
drwxr-xr-x  5 user  staff   160 Jan 19 15:30 ..
-rw-r--r--  1 user  staff  1234 Jan 19 15:30 README.md
drwxr-xr-x  4 user  staff   128 Jan 19 15:30 src
drwxr-xr-x  3 user  staff    96 Jan 19 15:30 tests
-rw-r--r--  1 user  staff   456 Jan 19 15:30 requirements.txt
"""

# Bước 2: Xem cấu trúc thư mục mã nguồn
terminal.run({"command": "cd src"})
print(terminal.run({"command": "tree"}))

# Bước 3: Tìm kiếm các mẫu cụ thể
print(terminal.run({"command": "grep -r 'def process' ."}))
```

(2) Phân tích Tệp Dữ liệu

Nhanh chóng hiểu cấu trúc và nội dung của các tệp dữ liệu:

```python
terminal = TerminalTool(workspace="./data")

# Xem vài dòng đầu của tệp CSV
print(terminal.run({"command": "head -n 5 sales_2024.csv"}))
"""
date,product,quantity,revenue
2024-01-01,Widget A,150,4500.00
2024-01-01,Widget B,200,8000.00
2024-01-02,Widget A,180,5400.00
2024-01-02,Widget C,120,3600.00
"""

# Đếm tổng số dòng
print(terminal.run({"command": "wc -l *.csv"}))
"""
  10234 sales_2024.csv
   8567 sales_2023.csv
  18801 total
"""

# Trích xuất và đếm các danh mục sản phẩm
print(terminal.run({"command": "tail -n +2 sales_2024.csv | cut -d',' -f2 | sort | uniq -c"}))
"""
  3456 Widget A
  4123 Widget B
  2655 Widget C
"""
```

(3) Phân tích Tệp Log

Phân tích log ứng dụng theo thời gian thực, nhanh chóng định vị vấn đề:

```python
terminal = TerminalTool(workspace="/var/log")

# Xem các log lỗi mới nhất
print(terminal.run({"command": "tail -n 50 app.log | grep ERROR"}))

# Thống kê phân bố các loại lỗi
print(terminal.run({"command": "grep ERROR app.log | awk '{print $4}' | sort | uniq -c | sort -rn"}))
"""
  245 DatabaseConnectionError
  123 TimeoutException
   67 ValidationError
   34 AuthenticationError
"""

# Tìm log cho một khoảng thời gian cụ thể
print(terminal.run({"command": "grep '2024-01-19 15:' app.log | tail -n 20"}))
```

(4) Phân tích Codebase

Hỗ trợ việc review và hiểu mã nguồn:

```python
terminal = TerminalTool(workspace="./codebase")

# Đếm số dòng mã
print(terminal.run({"command": "find . -name '*.py' -exec wc -l {} + | tail -n 1"}))

# Tìm tất cả các chú thích TODO
print(terminal.run({"command": "grep -rn 'TODO' --include='*.py'"}))

# Tìm định nghĩa của một hàm cụ thể
print(terminal.run({"command": "grep -rn 'def process_data' --include='*.py'"}))

# Xem phần triển khai của hàm
print(terminal.run({"command": "sed -n '/def process_data/,/^def /p' src/processor.py | head -n -1"}))
```

### 9.5.4 Phối hợp với Các Công cụ Khác

Sức mạnh thực sự của TerminalTool nằm ở việc sử dụng cộng tác với MemoryTool, NoteTool và ContextBuilder.

(1) Phối hợp với MemoryTool

Thông tin do TerminalTool khám phá có thể được lưu trữ vào hệ thống bộ nhớ:

```python
# Dùng TerminalTool để khám phá cấu trúc dự án
structure = terminal.run({"command": "tree -L 2 src"})

# Lưu vào bộ nhớ ngữ nghĩa
memory_tool.run({
    "action": "add",
    "content": f"Project structure:\n{structure}",
    "memory_type": "semantic",
    "importance": 0.8,
    "metadata": {"type": "project_structure"}
})
```

(2) Phối hợp với NoteTool

Các phát hiện quan trọng có thể được ghi lại dưới dạng ghi chú có cấu trúc:

```python
# Phát hiện một nút thắt hiệu năng
log_analysis = terminal.run({"command": "grep 'slow query' app.log | tail -n 10"})

# Ghi lại dưới dạng ghi chú blocker
note_tool.run({
    "action": "create",
    "title": "Database Slow Query Issue",
    "content": f"## Problem Description\nFound multiple slow queries affecting system performance\n\n## Log Analysis\n```\n{log_analysis}\n```\n\n## Next Steps\n1. Analyze slow query SQL\n2. Add indexes\n3. Optimize query logic",
    "note_type": "blocker",
    "tags": ["performance", "database"]
})
```

(3) Phối hợp với ContextBuilder

Đầu ra của TerminalTool có thể là một phần của ngữ cảnh:

```python
# Khám phá codebase
code_structure = terminal.run({"command": "ls -R src"})
recent_changes = terminal.run({"command": "git log --oneline -10"})

# Chuyển đổi thành ContextPacket
from hello_agents.context import ContextPacket
from datetime import datetime

packets = [
    ContextPacket(
        content=f"Codebase structure:\n{code_structure}",
        timestamp=datetime.now(),
        token_count=len(code_structure) // 4,
        relevance_score=0.7,
        metadata={"type": "code_structure", "source": "terminal"}
    ),
    ContextPacket(
        content=f"Recent commits:\n{recent_changes}",
        timestamp=datetime.now(),
        token_count=len(recent_changes) // 4,
        relevance_score=0.8,
        metadata={"type": "git_history", "source": "terminal"}
    )
]

# Đưa thông tin này vào khi xây dựng ngữ cảnh
context = context_builder.build(
    user_query="How to refactor the user service module?",
    custom_packets=packets
)
```

## 9.6 Thực hành Agent Dài hạn: Trợ lý Bảo trì Codebase

Bây giờ, hãy tích hợp ContextBuilder, NoteTool và TerminalTool để xây dựng một agent dài hạn (long-horizon) hoàn chỉnh — **Trợ lý Bảo trì Codebase**. Trợ lý này có thể:

1. Khám phá và hiểu cấu trúc codebase
2. Ghi lại các vấn đề đã phát hiện và các điểm cần cải thiện
3. Theo dõi các tác vụ tái cấu trúc (refactoring) dài hạn
4. Duy trì tính mạch lạc dưới các ràng buộc của cửa sổ ngữ cảnh

### 9.6.1 Thiết lập Tình huống và Phân tích Yêu cầu

**Tình huống Nghiệp vụ**

Giả sử chúng ta đang bảo trì một ứng dụng web Python cỡ vừa. Codebase này chứa khoảng 50 tệp Python, được xây dựng bằng framework Flask, bao gồm các module về mô hình dữ liệu, logic nghiệp vụ, giao diện API và các module khác, đồng thời cũng có một số nợ kỹ thuật (technical debt) cần được dọn dẹp dần. Trong tình huống này, chúng ta cần một trợ lý thông minh để giúp khám phá codebase, hiểu cấu trúc dự án, các phụ thuộc (dependency) và phong cách mã; nhận diện các vấn đề trong mã, chẳng hạn như trùng lặp mã, độ phức tạp quá mức, thiếu kiểm thử, v.v.; theo dõi tiến độ tác vụ, ghi lại các hạng mục cần làm, công việc đã hoàn thành và các blocker gặp phải; đồng thời đưa ra các khuyến nghị tái cấu trúc mạch lạc dựa trên ngữ cảnh lịch sử.

**Thách thức và Giải pháp**

Tình huống này đối mặt với một vài thách thức điển hình của tác vụ dài hạn. Đầu tiên là vấn đề thông tin vượt quá cửa sổ ngữ cảnh — toàn bộ codebase có thể chứa hàng chục nghìn dòng mã, không thể đưa hết vào cửa sổ ngữ cảnh cùng một lúc. Chúng ta giải quyết điều này bằng cách dùng TerminalTool để khám phá mã tức thời, theo nhu cầu, chỉ xem các tệp cụ thể khi cần. Thứ hai là thách thức quản lý trạng thái xuyên phiên — các tác vụ tái cấu trúc có thể kéo dài nhiều ngày và cần duy trì tiến độ qua nhiều phiên. Chúng ta xử lý điều này bằng cách dùng NoteTool để ghi lại tiến độ theo giai đoạn, các hạng mục cần làm và các quyết định then chốt. Cuối cùng là vấn đề chất lượng và mức độ liên quan của ngữ cảnh — mỗi cuộc hội thoại cần xem lại thông tin lịch sử liên quan nhưng không được bị nhấn chìm bởi thông tin không liên quan. Chúng ta dùng ContextBuilder để lọc và tổ chức ngữ cảnh một cách thông minh, đảm bảo mật độ tín hiệu cao.

### 9.6.2 Thiết kế Kiến trúc Hệ thống

Trợ lý bảo trì codebase của chúng ta áp dụng kiến trúc ba lớp, như thể hiện trong Hình 9.3:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/9-figures/9-3.png" alt="" width="85%"/>
  <p>Hình 9.3 Kiến trúc ba lớp của trợ lý bảo trì codebase</p>
</div>

### 9.6.3 Triển khai Cốt lõi

Bây giờ hãy triển khai lớp cốt lõi của hệ thống này:

```python
from typing import Dict, Any, List, Optional
from datetime import datetime
import json

from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.context import ContextBuilder, ContextConfig, ContextPacket
from hello_agents.tools import MemoryTool, NoteTool, TerminalTool
from hello_agents.core.message import Message


class CodebaseMaintainer:
    """Trợ lý Bảo trì Codebase - Ví dụ về agent dài hạn (long-horizon)

    Tích hợp ContextBuilder + NoteTool + TerminalTool + MemoryTool
    Hiện thực hóa việc quản lý tác vụ bảo trì codebase xuyên phiên
    """

    def __init__(
        self,
        project_name: str,
        codebase_path: str,
        llm: Optional[HelloAgentsLLM] = None
    ):
        self.project_name = project_name
        self.codebase_path = codebase_path
        self.session_id = f"session_{datetime.now().strftime('%Y%m%d_%H%M%S')}"

        # Khởi tạo LLM
        self.llm = llm or HelloAgentsLLM()

        # Khởi tạo các công cụ
        self.memory_tool = MemoryTool(user_id=project_name)
        self.note_tool = NoteTool(workspace=f"./{project_name}_notes")
        self.terminal_tool = TerminalTool(workspace=codebase_path, timeout=60)

        # Khởi tạo trình xây dựng ngữ cảnh
        self.context_builder = ContextBuilder(
            memory_tool=self.memory_tool,
            rag_tool=None,  # Trường hợp này không dùng RAG
            config=ContextConfig(
                max_tokens=4000,
                reserve_ratio=0.15,
                min_relevance=0.2,
                enable_compression=True
            )
        )

        # Lịch sử hội thoại
        self.conversation_history: List[Message] = []

        # Thống kê
        self.stats = {
            "session_start": datetime.now(),
            "commands_executed": 0,
            "notes_created": 0,
            "issues_found": 0
        }

        print(f"✅ Codebase maintenance assistant initialized: {project_name}")
        print(f"📁 Working directory: {codebase_path}")
        print(f"🆔 Session ID: {self.session_id}")

    def run(self, user_input: str, mode: str = "auto") -> str:
        """Chạy trợ lý

        Args:
            user_input: Đầu vào của người dùng
            mode: Chế độ chạy
                - "auto": Tự động quyết định có dùng công cụ hay không
                - "explore": Tập trung vào khám phá mã
                - "analyze": Tập trung vào phân tích vấn đề
                - "plan": Tập trung vào lập kế hoạch tác vụ

        Returns:
            str: Câu trả lời của trợ lý
        """
        print(f"\n{'='*80}")
        print(f"👤 User: {user_input}")
        print(f"{'='*80}\n")

        # Bước 1: Thực hiện tiền xử lý dựa trên chế độ
        pre_context = self._preprocess_by_mode(user_input, mode)

        # Bước 2: Truy xuất các ghi chú liên quan
        relevant_notes = self._retrieve_relevant_notes(user_input)
        note_packets = self._notes_to_packets(relevant_notes)

        # Bước 3: Xây dựng ngữ cảnh được tối ưu
        context = self.context_builder.build(
            user_query=user_input,
            conversation_history=self.conversation_history,
            system_instructions=self._build_system_instructions(mode),
            custom_packets=note_packets + pre_context
        )

        # Bước 4: Gọi LLM
        print("🤖 Thinking...")
        response = self.llm.invoke(context)

        # Bước 5: Hậu xử lý
        self._postprocess_response(user_input, response)

        # Bước 6: Cập nhật lịch sử hội thoại
        self._update_history(user_input, response)

        print(f"\n🤖 Assistant: {response}\n")
        print(f"{'='*80}\n")

        return response

    def _preprocess_by_mode(
        self,
        user_input: str,
        mode: str
    ) -> List[ContextPacket]:
        """Thực hiện tiền xử lý dựa trên chế độ, thu thập thông tin liên quan"""
        packets = []

        if mode == "explore" or mode == "auto":
            # Chế độ explore: Tự động xem cấu trúc dự án
            print("🔍 Exploring codebase structure...")

            structure = self.terminal_tool.run({"command": "find . -type f -name '*.py' | head -n 20"})
            self.stats["commands_executed"] += 1

            packets.append(ContextPacket(
                content=f"[Codebase Structure]\n{structure}",
                timestamp=datetime.now(),
                token_count=len(structure) // 4,
                relevance_score=0.6,
                metadata={"type": "code_structure", "source": "terminal"}
            ))

        if mode == "analyze":
            # Chế độ analyze: Kiểm tra độ phức tạp và các vấn đề của mã
            print("📊 Analyzing code quality...")

            # Đếm số dòng mã
            loc = self.terminal_tool.run({"command": "find . -name '*.py' -exec wc -l {} + | tail -n 1"})

            # Tìm TODO và FIXME
            todos = self.terminal_tool.run({"command": "grep -rn 'TODO\\|FIXME' --include='*.py' | head -n 10"})

            self.stats["commands_executed"] += 2

            packets.append(ContextPacket(
                content=f"[Code Statistics]\n{loc}\n\n[To-Do Items]\n{todos}",
                timestamp=datetime.now(),
                token_count=(len(loc) + len(todos)) // 4,
                relevance_score=0.7,
                metadata={"type": "code_analysis", "source": "terminal"}
            ))

        if mode == "plan":
            # Chế độ lập kế hoạch: Nạp các ghi chú gần đây
            print("📋 Loading task planning...")

            task_notes = self.note_tool.run({
                "action": "list",
                "note_type": "task_state",
                "limit": 3
            })

            if task_notes:
                content = "\n".join([f"- {note['title']}" for note in task_notes])
                packets.append(ContextPacket(
                    content=f"[Current Tasks]\n{content}",
                    timestamp=datetime.now(),
                    token_count=len(content) // 4,
                    relevance_score=0.8,
                    metadata={"type": "task_plan", "source": "notes"}
                ))

        return packets

    def _retrieve_relevant_notes(self, query: str, limit: int = 3) -> List[Dict]:
        """Truy xuất các ghi chú liên quan"""
        try:
            # Ưu tiên truy xuất các blocker
            blockers = self.note_tool.run({
                "action": "list",
                "note_type": "blocker",
                "limit": 2
            })

            # Tìm kiếm các ghi chú liên quan
            search_results = self.note_tool.run({
                "action": "search",
                "query": query,
                "limit": limit
            })

            # Gộp và khử trùng lặp
            all_notes = {note.get('note_id') or note.get('id'): note for note in (blockers or []) + (search_results or [])}
            return list(all_notes.values())[:limit]

        except Exception as e:
            print(f"[WARNING] Note retrieval failed: {e}")
            return []

    def _notes_to_packets(self, notes: List[Dict]) -> List[ContextPacket]:
        """Chuyển đổi ghi chú thành các gói ngữ cảnh"""
        packets = []

        for note in notes:
            # Đặt các điểm liên quan khác nhau dựa trên loại ghi chú
            relevance_map = {
                "blocker": 0.9,
                "action": 0.8,
                "task_state": 0.75,
                "conclusion": 0.7
            }

            note_type = note.get('type', 'general')
            relevance = relevance_map.get(note_type, 0.6)

            content = f"[Note: {note.get('title', 'Untitled')}]\nType: {note_type}\n\n{note.get('content', '')}"

            packets.append(ContextPacket(
                content=content,
                timestamp=datetime.fromisoformat(note.get('updated_at', datetime.now().isoformat())),
                token_count=len(content) // 4,
                relevance_score=relevance,
                metadata={
                    "type": "note",
                    "note_type": note_type,
                    "note_id": note.get('note_id') or note.get('id')
                }
            ))

        return packets

    def _build_system_instructions(self, mode: str) -> str:
        """Xây dựng chỉ thị hệ thống"""
        base_instructions = f"""You are the codebase maintenance assistant for the {self.project_name} project.

Your core capabilities:
1. Use TerminalTool to explore codebase (ls, cat, grep, find, etc.)
2. Use NoteTool to record discoveries and tasks
3. Provide coherent recommendations based on historical notes

Current session ID: {self.session_id}
"""

        mode_specific = {
            "explore": """
Current mode: Explore codebase

You should:
- Actively use terminal commands to understand code structure
- Identify key modules and files
- Record project architecture in notes
""",
            "analyze": """
Current mode: Analyze code quality

You should:
- Find code issues (duplication, complexity, TODOs, etc.)
- Evaluate code quality
- Record discovered issues as blocker or action notes
""",
            "plan": """
Current mode: Task planning

You should:
- Review historical notes and tasks
- Formulate next action plan
- Update task status notes
""",
            "auto": """
Current mode: Auto decision

You should:
- Flexibly choose strategies based on user needs
- Use tools when needed
- Maintain professionalism and practicality in responses
"""
        }

        return base_instructions + mode_specific.get(mode, mode_specific["auto"])

    def _postprocess_response(self, user_input: str, response: str):
        """Hậu xử lý: Phân tích phản hồi, tự động ghi lại thông tin quan trọng"""

        # Nếu phát hiện vấn đề, tự động tạo ghi chú blocker
        if any(keyword in response.lower() for keyword in ["issue", "bug", "error", "blocker", "problem"]):
            try:
                self.note_tool.run({
                    "action": "create",
                    "title": f"Issue found: {user_input[:30]}...",
                    "content": f"## User Input\n{user_input}\n\n## Issue Analysis\n{response[:500]}...",
                    "note_type": "blocker",
                    "tags": [self.project_name, "auto_detected", self.session_id]
                })
                self.stats["notes_created"] += 1
                self.stats["issues_found"] += 1
                print("📝 Automatically created issue note")
            except Exception as e:
                print(f"[WARNING] Failed to create note: {e}")

        # Nếu là lập kế hoạch tác vụ, tự động tạo ghi chú action
        elif any(keyword in user_input.lower() for keyword in ["plan", "next", "task", "todo"]):
            try:
                self.note_tool.run({
                    "action": "create",
                    "title": f"Task planning: {user_input[:30]}...",
                    "content": f"## Discussion\n{user_input}\n\n## Action Plan\n{response[:500]}...",
                    "note_type": "action",
                    "tags": [self.project_name, "planning", self.session_id]
                })
                self.stats["notes_created"] += 1
                print("📝 Automatically created action plan note")
            except Exception as e:
                print(f"[WARNING] Failed to create note: {e}")

    def _update_history(self, user_input: str, response: str):
        """Cập nhật lịch sử hội thoại"""
        self.conversation_history.append(
            Message(content=user_input, role="user", timestamp=datetime.now())
        )
        self.conversation_history.append(
            Message(content=response, role="assistant", timestamp=datetime.now())
        )

        # Giới hạn độ dài lịch sử (giữ lại 10 vòng hội thoại gần nhất)
        if len(self.conversation_history) > 20:
            self.conversation_history = self.conversation_history[-20:]

    # === Các phương thức tiện ích ===

    def explore(self, target: str = ".") -> str:
        """Khám phá codebase"""
        return self.run(f"Please explore the code structure of {target}", mode="explore")

    def analyze(self, focus: str = "") -> str:
        """Phân tích chất lượng mã"""
        query = f"Please analyze code quality" + (f", focusing on {focus}" if focus else "")
        return self.run(query, mode="analyze")

    def plan_next_steps(self) -> str:
        """Lập kế hoạch các bước tiếp theo"""
        return self.run("Based on current progress, plan next steps", mode="plan")

    def execute_command(self, command: str) -> str:
        """Thực thi lệnh terminal"""
        result = self.terminal_tool.run({"command": command})
        self.stats["commands_executed"] += 1
        return result

    def create_note(
        self,
        title: str,
        content: str,
        note_type: str = "general",
        tags: List[str] = None
    ) -> str:
        """Tạo ghi chú"""
        result = self.note_tool.run({
            "action": "create",
            "title": title,
            "content": content,
            "note_type": note_type,
            "tags": tags or [self.project_name]
        })
        self.stats["notes_created"] += 1
        return result

    def get_stats(self) -> Dict[str, Any]:
        """Lấy thống kê"""
        duration = (datetime.now() - self.stats["session_start"]).total_seconds()

        # Lấy tóm tắt ghi chú
        try:
            note_summary = self.note_tool.run({"action": "summary"})
        except:
            note_summary = {}

        return {
            "session_info": {
                "session_id": self.session_id,
                "project": self.project_name,
                "duration_seconds": duration
            },
            "activity": {
                "commands_executed": self.stats["commands_executed"],
                "notes_created": self.stats["notes_created"],
                "issues_found": self.stats["issues_found"]
            },
            "notes": note_summary
        }

    def generate_report(self, save_to_file: bool = True) -> Dict[str, Any]:
        """Tạo báo cáo phiên làm việc"""
        report = self.get_stats()

        if save_to_file:
            report_file = f"maintainer_report_{self.session_id}.json"
            with open(report_file, 'w', encoding='utf-8') as f:
                json.dump(report, f, ensure_ascii=False, indent=2, default=str)
            report["report_file"] = report_file
            print(f"📄 Report saved: {report_file}")

        return report
```

### 9.6.4 Ví dụ Sử dụng Hoàn chỉnh

Bây giờ hãy minh họa quy trình làm việc của agent dài hạn này thông qua một tình huống sử dụng hoàn chỉnh:

```python
# ========== Khởi tạo Trợ lý ==========

from hello_agents import HelloAgentsLLM

maintainer = CodebaseMaintainer(
    project_name="my_flask_app",
    codebase_path="./my_flask_app",
    llm=HelloAgentsLLM()
)


# ========== Ngày 1: Khám phá Codebase ==========
# 1. Khám phá ban đầu
response = maintainer.explore()
"""
🔍 Exploring codebase structure...
🤖 Thinking...

🤖 Assistant: I have scanned the codebase, this is a Flask web application, mainly containing the following modules:

📁 Project structure:
- app/
  ├── models/      # Data models (User, Product, Order, etc.)
  ├── routes/      # API routes
  ├── services/    # Business logic
  └── utils/       # Utility functions
- tests/           # Test files
- migrations/      # Database migrations
- config.py        # Configuration file

Total approximately 3,500 lines of Python code.

Recommended next steps:
1. Review core model definitions
2. Check routes and API design
3. Evaluate test coverage
"""

# 2. Đi sâu vào một module cụ thể
response = maintainer.run("Please analyze the data model design in the app/models/ directory")
"""
🤖 Assistant: I reviewed the app/models/ directory and found the following data models:

1. User (user.py)
   - Fields: id, username, email, password_hash, created_at
   - Relationships: One-to-many Order
   - Issue: ⚠️ Missing index, email field not set with unique constraint

2. Product (product.py)
   - Fields: id, name, description, price, stock
   - Issue: ✅ Well designed

3. Order (order.py)
   - Fields: id, user_id, product_id, quantity, status
   - Issue: ⚠️ Missing creation time field, not conducive to data analysis

Recommendations:
1. Add unique constraint to User.email
2. Add created_at and updated_at fields to Order
3. Consider adding soft delete support

I have recorded these issues as blocker notes.
"""
# [Automatically created blocker note]


# ========== Ngày 2: Phân tích Chất lượng Mã ==========

# 1. Phân tích chất lượng tổng thể
response = maintainer.analyze()
"""
📊 Analyzing code quality...
🤖 Thinking...

🤖 Assistant: Code quality analysis report:

📊 Code statistics:
- Total lines: 3,542 lines
- Average file length: approximately 150 lines
- Largest file: services/order_service.py (456 lines)

⚠️ Issues found:

1. Code duplication (HIGH):
   - user_service.py and product_service.py have similar CRUD logic
   - Recommendation: Extract base class BaseService

2. Excessive complexity (MEDIUM):
   - order_service.py::process_order method has 8 levels of nesting
   - Recommendation: Split into multiple small methods

3. Lack of tests (HIGH):
   - Test coverage only 45%
   - services/ directory has no tests at all

4. TODO/FIXME (12 items):
   - Most are "optimize later" type comments
   - Recommendation: Clean up or convert to formal tasks

I have recorded these issues as notes and marked priorities.
"""
# [Automatically created multiple blocker notes]

# 2. Xem một vấn đề cụ thể
response = maintainer.run("Please review the process_order method in order_service.py and provide refactoring recommendations")
"""
🤖 Assistant: I reviewed the code of the process_order method:

【Original Code】
def process_order(self, order_id):
    order = Order.query.get(order_id)
    if order:
        if order.status == 'pending':
            product = Product.query.get(order.product_id)
            if product:
                if product.stock >= order.quantity:
                    # ... 8 levels of nesting continue ...

【Problem Analysis】
1. Deep nesting leads to poor readability
2. Lack of error handling
3. Business logic mixed with data access

【Refactoring Recommendations】
def process_order(self, order_id):
    # 1. Early return pattern, reduce nesting
    order = self._get_order(order_id)
    if not order or order.status != 'pending':
        return False

    # 2. Extract methods, single responsibility
    if not self._check_stock(order):
        return False

    # 3. Use transactions to ensure consistency
    with db.transaction():
        self._update_stock(order)
        self._update_order_status(order, 'processing')

    return True

【Next Steps】
1. I have created a refactoring plan note
2. Recommend writing test cases first
3. Refactor gradually, keeping functionality unchanged
"""
# [Automatically created action note]


# ========== Ngày 3: Lập kế hoạch Tác vụ Tái cấu trúc ==========

# 1. Xem lại tiến độ
response = maintainer.plan_next_steps()
"""
📋 Loading task planning...
🤖 Thinking...

🤖 Assistant: Based on the exploration and analysis from the past two days, I have organized refactoring task priorities:

🔴 High priority (complete this week):
1. [blocker] Add unique constraint to User.email
   - Risk: Low, just database migration
   - Effort: 1 hour

2. [blocker] Extract BaseService to reduce code duplication
   - Risk: Medium, affects multiple service classes
   - Effort: Half a day

3. [blocker] Refactor process_order method
   - Risk: High, core business logic
   - Effort: 1 day (including writing tests)

🟡 Medium priority (next week):
4. Add unit tests for services/
5. Clean up TODO comments
6. Add time fields to Order

🟢 Low priority (TBD):
7. Performance optimization
8. Documentation updates

Recommendations:
- This week, first resolve data model issues (tasks 1, 6)
- Next week, conduct code refactoring (tasks 2, 3)
- Continuously increase test coverage (task 4)

I have updated task status notes, you can check progress anytime.
"""

# 2. Tạo thủ công một kế hoạch tái cấu trúc chi tiết
maintainer.create_note(
    title="Weekly Refactoring Plan - Week 1",
    content="""## Objectives
Complete optimization of data model layer

## Task Checklist
- [ ] Add unique constraint to User.email
- [ ] Add created_at, updated_at fields to Order
- [ ] Write database migration scripts
- [ ] Update related test cases

## Schedule
- Monday: Design migration scripts
- Tuesday-Wednesday: Execute migration and test
- Thursday: Update test cases
- Friday: Code Review

## Risks
- Database migration may affect production environment, needs to be executed during off-peak hours
- Existing data may have duplicate emails, need to clean up first
""",
    note_type="task_state",
    tags=["refactoring", "week1", "high_priority"]
)

print("✅ Created detailed refactoring plan")


# ========== Một Tuần Sau: Kiểm tra Tiến độ ==========

# Xem tóm tắt ghi chú
summary = maintainer.note_tool.run({"action": "summary"})
print("📊 Note summary:")
print(json.dumps(summary, indent=2, ensure_ascii=False))
"""
{
  "total_notes": 8,
  "type_distribution": {
    "blocker": 3,
    "action": 2,
    "task_state": 2,
    "conclusion": 1
  },
  "recent_notes": [
    {
      "id": "note_20250119_160000_7",
      "title": "Weekly Refactoring Plan - Week 1",
      "type": "task_state",
      "updated_at": "2025-01-19T16:00:00"
    },
    ...
  ]
}
"""

# Tạo báo cáo hoàn chỉnh
report = maintainer.generate_report()
print("\n📄 Session report:")
print(json.dumps(report, indent=2, ensure_ascii=False))
"""
{
  "session_info": {
    "session_id": "session_20250119_150000",
    "project": "my_flask_app",
    "duration_seconds": 172800  # 2 days
  },
  "activity": {
    "commands_executed": 24,
    "notes_created": 8,
    "issues_found": 3
  },
  "notes": { ... }
}
"""
```

### 9.6.5 Phân tích Hiệu quả Chạy

Thông qua nghiên cứu tình huống hoàn chỉnh này, chúng ta có thể thấy một vài đặc điểm then chốt của các agent dài hạn. Đầu tiên là tính mạch lạc xuyên phiên — agent duy trì tính mạch lạc của tác vụ qua nhiều ngày và nhiều phiên nhờ NoteTool. Các vấn đề được khám phá vào ngày đầu tiên tự động được cân nhắc trong quá trình phân tích của ngày thứ hai, việc lập kế hoạch ngày thứ ba có thể tổng hợp mọi phát hiện của hai ngày trước, và toàn bộ lịch sử được bảo toàn khi kiểm tra lại sau một tuần. Thứ hai là quản lý ngữ cảnh thông minh — ContextBuilder đảm bảo ngữ cảnh chất lượng cao cho mỗi cuộc hội thoại, tự động thu thập các ghi chú liên quan (đặc biệt là loại blocker), điều chỉnh động chiến lược tiền xử lý dựa trên chế độ hội thoại, và chọn thông tin liên quan nhất trong phạm vi ngân sách token.

Đặc điểm thứ ba là truy cập hệ thống tệp tức thời — TerminalTool hỗ trợ khám phá mã linh hoạt mà không cần lập chỉ mục trước toàn bộ codebase, có thể xem nội dung tệp cụ thể ngay lập tức, và hỗ trợ xử lý văn bản phức tạp (grep, awk, v.v.). Thứ tư là quản lý tri thức tự động — hệ thống tự động quản lý tri thức đã khám phá, tự động tạo ghi chú blocker khi phát hiện vấn đề, tự động tạo ghi chú action khi thảo luận về kế hoạch, và tự động lưu trữ thông tin then chốt vào hệ thống bộ nhớ. Cuối cùng là cộng tác người-máy — hệ thống này hỗ trợ các chế độ cộng tác người-máy linh hoạt, trong đó agent có thể tự động hoàn thành việc khám phá và phân tích, con người có thể can thiệp và định hướng thông qua hệ thống ghi chú, và hỗ trợ tạo thủ công các ghi chú lập kế hoạch chi tiết.

Khung cơ bản này có thể được mở rộng thêm, chẳng hạn tích hợp RAGTool để xây dựng chỉ mục vector cho codebase kết hợp với truy xuất ngữ nghĩa, tách thành các explorer, analyzer và planner chuyên biệt để hiện thực hóa cộng tác đa agent, tích hợp các công cụ kiểm thử để tự động xác minh kết quả tái cấu trúc, thực thi các lệnh git qua TerminalTool để theo dõi thay đổi mã, hoặc xây dựng giao diện trực quan bằng Gradio/Streamlit.

## 9.7 Tóm tắt Chương

Trong chương này, chúng ta đã đi sâu khám phá các nền tảng lý thuyết và thực hành kỹ thuật của kỹ thuật ngữ cảnh:

### Cấp độ Lý thuyết

1. **Bản chất của Kỹ thuật Ngữ cảnh**: Sự tiến hóa từ "kỹ thuật prompt" sang "kỹ thuật ngữ cảnh", cốt lõi là quản lý ngân sách chú ý (attention budget) có hạn
2. **Suy giảm Ngữ cảnh (Context Rot)**: Hiểu sự suy giảm hiệu năng do các ngữ cảnh dài gây ra, nhận thức ngữ cảnh là một tài nguyên khan hiếm
3. **Ba Chiến lược Chính**: Nén gọn (Compaction), ghi chú có cấu trúc (structured note-taking), kiến trúc sub-agent (sub-agent architectures)

### Thực hành Kỹ thuật

1. **ContextBuilder**: Hiện thực hóa pipeline GSSC, cung cấp giao diện quản lý ngữ cảnh thống nhất
2. **NoteTool**: Định dạng lai giữa Markdown+YAML, hỗ trợ bộ nhớ dài hạn có cấu trúc
3. **TerminalTool**: Công cụ dòng lệnh an toàn, hỗ trợ truy cập hệ thống tệp tức thời
4. **Agent Dài hạn (Long-Horizon Agent)**: Tích hợp ba công cụ chính, xây dựng trợ lý bảo trì codebase xuyên phiên

### Những Điểm Cốt lõi Cần Nắm

- **Thiết kế Phân lớp**: Truy cập tức thời (TerminalTool) + bộ nhớ phiên (MemoryTool) + ghi chú bền vững (NoteTool)
- **Lọc Thông minh**: Cơ chế chấm điểm dựa trên mức độ liên quan và tính gần đây
- **An toàn Trên hết**: Các cơ chế bảo mật nhiều lớp đảm bảo tính ổn định của hệ thống
- **Cộng tác Người-máy**: Cân bằng giữa tự động hóa và khả năng kiểm soát

Qua việc học chương này, bạn không chỉ nắm được các công nghệ cốt lõi của kỹ thuật ngữ cảnh, mà quan trọng hơn là hiểu cách xây dựng các hệ thống agent có thể duy trì tính mạch lạc và hiệu quả trong những khoảng thời gian dài. Những kỹ năng này sẽ trở thành nền tảng quan trọng để bạn xây dựng các ứng dụng agent cấp sản xuất.

Trong chương tiếp theo, chúng ta sẽ khám phá các giao thức giao tiếp giữa các agent và học cách giúp agent tương tác rộng rãi hơn với thế giới bên ngoài.

## Bài tập

> **Lưu ý**: Một số bài tập không có đáp án chuẩn. Trọng tâm là bồi dưỡng khả năng hiểu toàn diện và năng lực thực hành của người học về kỹ thuật ngữ cảnh và quản lý tác vụ dài hạn.

1. Chương này đã giới thiệu sự khác biệt giữa kỹ thuật ngữ cảnh và kỹ thuật prompt. Hãy phân tích:

   - Mục 9.1 đã đề cập rằng "ngữ cảnh phải được xem như một tài nguyên có hạn với lợi ích cận biên giảm dần". Hãy giải thích hiện tượng "suy giảm ngữ cảnh (context rot)" là gì? Tại sao chúng ta vẫn cần quản lý ngữ cảnh một cách cẩn thận ngay cả khi các mô hình hỗ trợ cửa sổ ngữ cảnh 100K hay thậm chí 200K?
   - Giả sử bạn muốn xây dựng một "trợ lý review mã" cần phân tích một codebase chứa 50 tệp. Hãy so sánh hai chiến lược: (1) Nạp toàn bộ nội dung tệp vào ngữ cảnh cùng một lúc; (2) Dùng ngữ cảnh JIT (Just-in-time), truy xuất tệp theo nhu cầu thông qua công cụ. Phân tích ưu điểm, nhược điểm và các tình huống áp dụng của từng cách.
   - Mục 9.2.1 đã đề cập hai cạm bẫy cực đoan của system prompt: "hard-code quá mức" và "quá mơ hồ". Hãy đưa ra một ví dụ thực tế cho mỗi trường hợp và giải thích cách tìm được sự cân bằng phù hợp.

2. Pipeline GSSC (Gather-Select-Structure-Compress) là công nghệ cốt lõi của chương này. Hãy suy nghĩ sâu:

   > **Lưu ý**: Đây là câu hỏi thực hành, khuyến nghị thao tác thực tế

   - Trong phần triển khai ContextBuilder ở Mục 9.3, bốn giai đoạn mỗi giai đoạn có các trách nhiệm khác nhau. Hãy phân tích: Nếu một giai đoạn nào đó thất bại (chẳng hạn giai đoạn Select chọn thông tin không liên quan, hoặc giai đoạn Compress nén quá mức dẫn đến mất thông tin), điều đó sẽ ảnh hưởng thế nào đến hiệu năng cuối cùng của agent?
   - Dựa trên mã ở Mục 9.3.4, hãy thêm một chức năng "đánh giá chất lượng ngữ cảnh" vào ContextBuilder: Sau mỗi lần xây dựng ngữ cảnh, tự động đánh giá mật độ thông tin, mức độ liên quan và tính đầy đủ của ngữ cảnh, đồng thời đưa ra các gợi ý tối ưu.
   - Giai đoạn "nén" trong pipeline GSSC sử dụng LLM để tóm tắt thông minh. Hãy suy nghĩ: Trong những trường hợp nào chiến lược cắt bớt đơn giản (truncation) hoặc cửa sổ trượt (sliding window) có thể phù hợp hơn so với tóm tắt bằng LLM? Hãy thiết kế một chiến lược nén lai kết hợp ưu điểm của nhiều phương pháp nén.

3. NoteTool và TerminalTool là các công cụ then chốt hỗ trợ các tác vụ dài hạn. Dựa trên Mục 9.4 và 9.5, hãy hoàn thành các bài thực hành mở rộng sau:

   > **Lưu ý**: Đây là câu hỏi thực hành, khuyến nghị thao tác thực tế

   - NoteTool sử dụng một hệ thống ghi chú phân cấp (ghi chú dự án, ghi chú tác vụ, ghi chú tạm thời). Hãy thiết kế một cơ chế "tự động sắp xếp ghi chú": Khi các ghi chú tạm thời tích lũy đến một số lượng nhất định, agent có thể tự động phân tích các ghi chú này, nâng cấp thông tin quan trọng lên thành ghi chú tác vụ hoặc ghi chú dự án, và dọn dẹp các nội dung dư thừa.
   - TerminalTool cung cấp khả năng thao tác hệ thống tệp, nhưng Mục 9.5.2 nhấn mạnh thiết kế bảo mật. Hãy phân tích: Các cơ chế bảo mật hiện tại (xác thực đường dẫn, danh sách trắng lệnh, kiểm tra quyền) đã đủ chưa? Nếu agent cần truy cập các tệp nhạy cảm hoặc thực hiện các thao tác nguy hiểm, thì quy trình "phê duyệt cộng tác người-máy" nên được thiết kế như thế nào?
   - Kết hợp NoteTool và TerminalTool, hãy thiết kế một "trợ lý tái cấu trúc mã thông minh": Có thể phân tích cấu trúc codebase, ghi lại các kế hoạch tái cấu trúc, thực hiện các thao tác tái cấu trúc theo từng bước, và theo dõi tiến độ cũng như các vấn đề gặp phải trong ghi chú. Hãy vẽ một sơ đồ quy trình làm việc hoàn chỉnh.

4. Trong tình huống "quản lý tác vụ dài hạn" ở Mục 9.6, chúng ta đã thấy giá trị của kỹ thuật ngữ cảnh trong các ứng dụng thực tế. Hãy phân tích sâu:

   - Tình huống này sử dụng chiến lược "quản lý ngữ cảnh phân lớp": truy cập tức thời (TerminalTool) + bộ nhớ phiên (MemoryTool) + ghi chú bền vững (NoteTool). Hãy phân tích: Ba lớp này nên phối hợp với nhau như thế nào? Thông tin nào nên đặt vào lớp nào? Làm thế nào để tránh dư thừa và bất nhất thông tin?
   - Giả sử xảy ra một gián đoạn trong quá trình thực thi tác vụ (chẳng hạn hệ thống gặp sự cố, mất kết nối mạng), agent cần khôi phục trạng thái từ ghi chú và tiếp tục thực thi. Hãy thiết kế một cơ chế "khôi phục từ điểm dừng (resume from breakpoint)": Làm thế nào để ghi lại đủ thông tin trạng thái trong ghi chú? Làm thế nào để xác minh rằng trạng thái được khôi phục là chính xác?
   - Các tác vụ dài hạn thường bao gồm việc thực thi song song hoặc tuần tự nhiều tác vụ con. Hãy thiết kế một hệ thống "quản lý phụ thuộc tác vụ": Có thể biểu diễn các mối quan hệ phụ thuộc giữa các tác vụ (chẳng hạn "Tác vụ B phải được thực thi sau khi Tác vụ A hoàn thành"), và tự động lập lịch thứ tự thực thi tác vụ. Hệ thống này nên tích hợp với NoteTool như thế nào?

5. Chương này đã nhiều lần đề cập đến khái niệm "tiết lộ tiệm tiến (progressive disclosure)". Hãy suy nghĩ:

   - Ở Mục 9.2.2, tiết lộ tiệm tiến được mô tả là "mỗi bước tương tác tạo ra ngữ cảnh mới, ngữ cảnh này lại định hướng cho quyết định tiếp theo". Hãy thiết kế một tình huống ứng dụng cụ thể (chẳng hạn viết bài báo học thuật, debug một vấn đề phức tạp), minh họa cách tiết lộ tiệm tiến giúp agent hoàn thành tác vụ hiệu quả hơn.
   - Một rủi ro tiềm ẩn của tiết lộ tiệm tiến là "khám phá kém hiệu quả": Agent có thể lãng phí thời gian vào các chi tiết không quan trọng hoặc bỏ lỡ thông tin then chốt. Hãy thiết kế một cơ chế "định hướng khám phá": Thông qua các quy tắc heuristic hoặc các chiến lược siêu nhận thức (metacognitive), giúp agent đưa ra các quyết định thông minh hơn về "nên khám phá gì tiếp theo".
   - So sánh "tiết lộ tiệm tiến" với cách "nạp toàn bộ ngữ cảnh cùng một lúc" truyền thống: Cách trước có lợi thế rõ rệt trong những loại tác vụ nào? Cách sau có thể phù hợp hơn trong những loại tác vụ nào? Hãy đưa ra ít nhất 3 ví dụ về các loại tác vụ khác nhau.

## Tài liệu tham khảo

[1] Anthropic. Effective Context Engineering for AI Agents. `https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents`

[2] David Kim. Context-Engineering (GitHub). `https://github.com/davidkimai/Context-Engineering`
