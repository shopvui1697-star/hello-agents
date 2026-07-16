# Câu hỏi thường gặp (FAQ) Hello-Agents Datawhale

> Bài viết này được tổng hợp dựa trên phần hỏi đáp (QA) tại buổi livestream ngày 2024-12-01, cùng với các câu hỏi thu thập được trong quá trình xây dựng khóa học kỳ đầu tiên, dùng làm tài liệu đọc mở rộng cho \
> khóa học "Hello-Agents Nhập môn từ con số 0" của Datawhale.\
> Khuyến nghị học kèm với tài liệu khóa học chính:
> - 🔗 [Tài liệu khóa học](https://datawhalechina.github.io/hello-agents/#/)
> - ⌛️ Video khóa học (sắp ra mắt)

---

## 1. Kiến trúc đa Agent (đa tác tử) và điều phối song song

<strong>Q1. Hệ thống đa Agent làm thế nào để thực hiện "đa luồng song song"? Đối với các bước có thể chạy song song do Agent lập kế hoạch tác vụ tách ra, làm sao để nhiều Agent thực thi tự nhận tác vụ và tự động xử lý các phụ thuộc? Có framework (bộ khung) sẵn có nào không?</strong>

- Tóm tắt các điểm chính:
  - Trước tiên, để "Agent lập kế hoạch tác vụ" thực hiện việc tách phụ thuộc giữa các tác vụ, tạo thành các tác vụ con có thể chạy song song và các tác vụ con có phụ thuộc.
  - Lớp thực thi có thể được thiết kế thành nhiều Agent chuyên trách, mỗi Agent chỉ xử lý tác vụ con mà mình phụ trách.
  - Điều phối song song thường được thực hiện thông qua hàng đợi (queue) / thăm dò API (API polling), v.v. Đa số các tình huống cần tùy chỉnh kết hợp với nghiệp vụ, chưa có giải pháp "một nút bấm" hoàn toàn thông dụng.
- Chỉ dẫn khóa học: các chương liên quan đến mô thức đa Agent và kiến trúc hệ thống (phần mô thức kinh điển + thực chiến framework).

---

## 2. Hệ sinh thái framework và giao thức giao tiếp

### 2.1 Các framework chủ đạo và vị thế của Hello-Agents

<strong>Q2. Hiện nay có những framework Agents chủ đạo nào? Hello-Agents chủ yếu giải quyết vấn đề gì?</strong>

- Tóm tắt các điểm chính:
  - Việc so sánh hệ thống và nhịp độ cập nhật của các framework chủ đạo được thảo luận tập trung ở chương 6 của khóa học, ở đây không liệt kê lại.
  - <strong>Vị thế của Hello-Agents</strong>: lấy giảng dạy và học tập làm chính, nhấn mạnh "cấu trúc rõ ràng + có thể triển khai thực tế + dễ suy ra và áp dụng rộng", giúp người mới xây dựng được khung kiến thức và thực hành Agent hoàn chỉnh.
- Chỉ dẫn khóa học: Chương 6 "Thực hành phát triển framework".

<strong>Q7. Hello-Agents nhìn có vẻ đầy đủ tính năng, nếu muốn dùng cho môi trường production (sản xuất) thì đại khái còn cần bổ sung những năng lực gì?</strong>

- Tóm tắt các điểm chính:
  - Bản thân framework thiên về "giảng dạy + có thể dùng được", để thực sự triển khai vào production còn cần phát triển thêm (secondary development) kết hợp với nghiệp vụ.
  - Các điểm tăng cường cốt lõi thường nằm ở:
    - Mô hình hóa tri thức nghiệp vụ và hiểu bối cảnh (scenario);
    - Cơ chế ghi log, giám sát (monitoring), đánh giá và rollback (khôi phục về trạng thái trước) ổn định hơn;
    - Tối ưu hiệu năng và kiểm soát chi phí.
- Chỉ dẫn khóa học: phần framework + chương thực chiến dự án.

### 2.2 Sự kết hợp giữa Hello-Agents với LangGraph / các framework khác

<strong>Q4. Hello-Agents làm thế nào để sử dụng kết hợp với LangGraph? Ai gọi ai?</strong>

<strong>Q11 / Q15. Muốn biết làm thế nào dùng A2A để kết hợp Hello-Agents với LangGraph?</strong>

- Tóm tắt các điểm chính (trả lời gộp):
  - Có thể dùng <strong>giao thức A2A + Agent Card</strong> để phơi bày (expose) "một Agent trong một framework" thành "năng lực từ xa" (remote capability) mà framework khác có thể gọi được.
  - Tương tự Function Calling: node của LangGraph có thể coi Agent trong Hello-Agents như một "hàm từ xa" để gọi, và ngược lại cũng vậy.
- Chỉ dẫn khóa học: Chương 10 "Giao thức giao tiếp giữa các Agent". Chương 6 "Thực hành phát triển framework".

<strong>Q17. Trong quá trình học có giới thiệu DeepResearch và các framework mã nguồn mở / có sẵn khác không?</strong>

- Tóm tắt các điểm chính:
  - DeepResearch thuộc nội dung Workflow (luồng công việc), sẽ được bổ sung sau.
  - Các nội dung liên quan đến framework được tập trung ở <strong>chương 5, chương 6</strong>, từ "đơn framework" đến "hệ sinh thái đa framework" đều được đề cập.
- Chỉ dẫn khóa học: chương 5, chương 6 (framework và các trường hợp ứng dụng).

<strong>Q18. Khi viết Agent bằng LangGraph, cảm giác giống "kịch bản luồng công việc của mô hình lớn" hơn, để thực sự triển khai thành dự án thì cần cân nhắc những gì? Trong khóa học có không?</strong>

- Tóm tắt các điểm chính:
  - Từ "kịch bản (script)" đến "dự án" cần cân nhắc thêm: ranh giới module, quản lý cấu hình, giám sát và đánh giá, khôi phục lỗi, cũng như các vấn đề kỹ thuật như phối hợp làm việc nhóm, v.v.
  - <strong>Phần 4</strong> của khóa học chuyên dẫn dắt mọi người xây dựng một dự án hoàn chỉnh từ con số 0, có thể đối chiếu thực hành.
- Chỉ dẫn khóa học: Phần 4 "Xây dựng dự án".

### 2.3 Giao thức giao tiếp: A2A & ANP

<strong>Q5. A2A và ANP giảng hơi nhanh, có thể chi tiết hơn và kèm theo ví dụ code không?</strong>

- Tóm tắt các điểm chính:
  - Trong khóa học sẽ triển khai từng bước theo trình tự "động cơ → trừu tượng hóa → trường của giao thức → ví dụ code".
  - Khuyến nghị đọc bổ sung chương "Giao thức giao tiếp giữa các Agent" trong "Học Agent qua thực hành phát triển ứng dụng" của Datawhale (phần giảng giải của tác giả trong cộng đồng ANP).
- Chỉ dẫn khóa học: Chương 10; có thể liên kết đến tài liệu chính thức của Datawhale trong phần "Đọc mở rộng" của README.

<strong>Q6. Khi thực chiến, nên khuyến nghị triển khai cục bộ (local) hay dùng trực tiếp API?</strong>

- Tóm tắt các điểm chính:
  - <strong>Chương 7</strong> của khóa học đưa ra đồng thời hai lộ trình: triển khai cục bộ và API trên đám mây (cloud).
  - Giai đoạn học: ưu tiên API (ngưỡng thấp), khi cần kiểm soát chi phí hoặc triển khai offline thì mới cân nhắc dùng cục bộ.
  - Chỉ dẫn khóa học: Chương 10

Chỉ dẫn khóa học các nội dung liên quan:
- 🔗 Chương 5 "Xây dựng Agent dựa trên nền tảng low-code". Nhấn để chuyển đến: [Chương 5](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter5/第五章%20基于低代码平台的智能体搭建.md)
- 🔗 Chương 6 "Thực hành phát triển framework". Nhấn để chuyển đến: [Chương 6](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter6/第六章%20框架开发实践.md)
- 🔗 Chương 7 "Xây dựng framework Agent của bạn". Nhấn để chuyển đến: [Chương 7](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter7/第七章%20构建你的Agent框架.md)
- 🔗 Chương 10 "Giao thức giao tiếp giữa các Agent". Nhấn để chuyển đến: [Chương 10](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter10/第十章%20智能体通信协议.md)


---

## 3. Vị thế khóa học, lộ trình học và đối tượng phù hợp

<strong>Q9. Các khóa học tiếp theo sẽ được đặt trên những nền tảng nào?</strong>

- Tóm tắt các điểm chính:
  - Kho lưu trữ Github (code và tài liệu khóa học)  
  - Datawhale trên B站 (Bilibili) (video)  
  - Trang web chính thức của Datawhale (cổng vào khóa học dạng văn bản và hình ảnh)  

<strong>Q10. Hoàn toàn không có nền tảng có phù hợp để học không?</strong>

- Tóm tắt các điểm chính:
  - Có thể học, nhưng cần "chậm lại một chút + xem nhiều phần bóc tách code hơn".
  - Các bạn không có nền tảng Python cần bỏ thêm công sức để bổ túc cú pháp, bản thân khóa học không hoàn toàn ở mức độ khó "không cần lập trình".
  - Về Python, khuyến nghị khóa học độc quyền của Datawhale: 🔗 [Cách học Python thông minh](https://datawhalechina.github.io/learn-python-the-smart-way-v2/)
  - Với những ai đã có nền tảng khóa học Python, lộ trình học khóa này được khuyến nghị như sau:
    1. Cấu hình môi trường, lời mở đầu
    2. 🔗 [Chương 1 Làm quen với Agent](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter1/第一章%20初识智能体.md)
    3. 🔗 [Chương 2 Lịch sử phát triển của Agent](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter2/第二章%20智能体发展史.md)
    4. 🔗 [Chương 3 Nền tảng mô hình ngôn ngữ lớn](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter3/第三章%20大语言模型基础.md)
    5. 🔗 [Chương 4 Xây dựng mô thức kinh điển của Agent](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter4/第四章%20智能体经典范式构建.md)
    6. 🔗 [Chương 5 Xây dựng Agent dựa trên nền tảng low-code](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter5/第五章%20基于低代码平台的智能体搭建.md)
    7. 🔗 [Chương 6 Thực hành phát triển ứng dụng framework](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter6/第六章%20框架开发实践.md)

<strong>Q12. Bài 1.1 và 2.1 của khóa học nhìn hơi giống nhau, mỗi bài có trọng tâm riêng là gì?</strong>

- Tóm tắt các điểm chính:
  - <strong>2.1</strong>: Hệ thống hóa Agent theo dòng thời gian phát triển, và mở rộng đến các trường hợp như hệ chuyên gia (expert system), thiên về "mạch lịch sử + góc nhìn toàn cảnh".  
  - <strong>1.1</strong>: Là phần dẫn nhập mở đầu, giới thiệu đơn giản các khái niệm cơ bản và bối cảnh.

<strong>Q13. Các bạn đã đi làm thì học thế nào? Học xong áp dụng vào công việc ra sao? Có gì khác so với các công cụ luồng công việc như n8n?</strong>

- Tóm tắt các điểm chính:
  - Gợi ý học tập: kết hợp với tình huống nghiệp vụ của chính mình, ưu tiên làm một ứng dụng Agent "nhỏ mà hoàn chỉnh", dù chỉ thay thế một đoạn nhỏ trong quy trình công việc.
  - Khác biệt cốt lõi so với luồng công việc (n8n, v.v.):
    - Luồng công việc: quy trình và nhánh về cơ bản là cố định, dùng để giải quyết các tác vụ "có cấu trúc tương đối chắc chắn".
    - Agent: phù hợp với các tác vụ phức tạp hơn, bất định hơn, có thể tự đưa ra quyết định ở một mức độ nhất định trong lúc chạy (gọi công cụ, lập kế hoạch tác vụ con, v.v.).
- Chỉ dẫn khóa học: các chương khái niệm + phần mô thức kinh điển và thực chiến dự án.

<strong>Q14. Trong khóa học có những chương nào chuyên nói về kiểm thử và đánh giá Agent?</strong>

- Tóm tắt các điểm chính:
  - Có chương đánh giá riêng (như chương 12), và cũng sẽ hình thành các trường hợp thực hành "đánh giá theo vòng khép kín" trong các chương khác.
- Chỉ dẫn khóa học: chương chuyên đề về kiểm thử và đánh giá + thực hành dự án liên quan.
- 🔗 [Chương 12](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter12/第十二章%20智能体性能评估.md)

<strong>Q16. Làm các thí nghiệm trong khóa học, cấu hình phần cứng tối thiểu là gì? Có cần card đồ họa không?</strong>

- Tóm tắt các điểm chính:
  - <strong>Chương Agentic RL</strong>: khuyến nghị môi trường GPU có bộ nhớ hiển thị (VRAM) ≥ 4G sẽ có trải nghiệm tốt hơn.
  - Các chương khác chỉ cần dùng API là có thể học trọn vẹn toàn bộ, không bắt buộc phải có GPU cục bộ.
- Chỉ dẫn khóa học: phần giải thích ở các chương liên quan đến Agentic RL.

<strong>Q20. Sinh viên đang theo học có phù hợp để học môn này không?</strong>

- Tóm tắt các điểm chính:
  - Rất phù hợp, hãy coi nó như một dự án nhập môn cho hướng giao thoa "AI + kỹ thuật phần mềm".
  - Khuyến nghị kết hợp với đồ án tốt nghiệp / dự án nghiên cứu nhỏ của bản thân để nâng cao giá trị thực hành.
  - Thành quả đồ án tốt nghiệp có thể được gửi vào dự án này thông qua hình thức tạo pull request (pr):
    - 🔗 [Co-creation-projects](https://github.com/datawhalechina/hello-agents/tree/main/Co-creation-projects).

---

## 4. Tri thức và công cụ: RAG / KAG / đồ thị tri thức / RL, v.v.

<strong>Q3. Xây dựng Agent có nhất định phải dùng đến RAG không?</strong>

- Tóm tắt các điểm chính:
  - RAG không phải là lựa chọn bắt buộc của Agent, mà là một loại "công cụ tri thức bên ngoài" thường gặp.
  - <strong>Chương 8</strong> của khóa học sẽ triển khai một phương án RAG từ con số 0, giúp hiểu sự khác biệt về mặt thiết kế hệ thống giữa "có RAG / không có RAG".
- Chỉ dẫn khóa học: Chương 8.
- 🔗 [Chương 8 Bộ nhớ và truy hồi](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter8/第八章%20记忆与检索.md)

<strong>Q21. KAG / đồ thị tri thức có quan hệ gì với Agent?</strong>

- Tóm tắt các điểm chính:
  - Có thể hiểu KAG / đồ thị tri thức như một "công cụ" hoặc "nền tảng tri thức" của Agent:
    - Agent chịu trách nhiệm ra quyết định và gọi;
    - Đồ thị tri thức cung cấp tri thức có cấu trúc và năng lực truy hồi.

<strong>Q22. RL, LLM, RLHF theo tiêu chuẩn phân loại "Agent kiểu học tập và Agent dựa trên LLMs" ở chương 1 thì lần lượt thuộc loại nào?</strong>

- Tóm tắt các điểm chính:
  - RL, LLM, RLHF đều giống <strong>thành phần cấu thành hoặc kỹ thuật triển khai</strong> của Agent hơn, chứ không phải một "loại Agent" riêng biệt.
  - Ví dụ: LLM có thể làm bộ não của Agent; RL / RLHF có thể được dùng để huấn luyện hoặc tinh chỉnh (fine-tune) chiến lược của Agent.

---

## 5. Hiệu năng và quản lý ngữ cảnh (context)

<strong>Q19. Prompt hệ thống của các tác vụ phức tạp rất dài, mỗi lần gọi tốn nhiều token, khiến phản hồi chậm đi; ngữ cảnh càng dài thì vấn đề này càng nghiêm trọng, nên cân bằng thế nào?</strong>

- Tóm tắt các điểm chính:
  - Mấu chốt nằm ở "cắt tỉa và quản lý ngữ cảnh": lịch sử hội thoại, kết quả gọi công cụ, prompt hệ thống, v.v. cần được giữ lại theo lớp, theo cấp độ.
  - Có thể giảm áp lực token thông qua "tóm tắt + module bộ nhớ + ngữ cảnh kiểu truy hồi".
  - <strong>Chương 9</strong> của khóa học chuyên nói về các chiến lược xử lý ngữ cảnh.
- Chỉ dẫn khóa học: Chương 9.
- 🔗 [Chương 9 Kỹ thuật ngữ cảnh](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter9/第九章%20上下文工程.md)

---

## 6. Dự án và phát triển sự nghiệp

<strong>Q8. Mấy dự án ví dụ trong khóa học có thể viết vào CV (sơ yếu lý lịch) không?</strong>

- Tóm tắt các điểm chính:
  - Có thể, với điều kiện bạn thực sự hiểu và giải thích rõ được:
    - Dự án giải quyết vấn đề gì;
    - Đã dùng những thiết kế Agent nào;
    - Đã làm những công việc cụ thể gì trong gọi công cụ / đánh giá / triển khai.
  - Khuyến nghị mô tả các dự án này trong CV theo cấu trúc "vấn đề - giải pháp - kết quả", và đính kèm liên kết Github.

---

## 7. Các vấn đề liên quan đến cấu hình môi trường, sử dụng mô hình và gọi API

<strong>Q17. API trong khóa học được thiết lập thế nào, có xảy ra trường hợp gọi thất bại</strong>

- Tóm tắt các điểm chính:
  - Mô hình API mà dự án khóa học hỗ trợ:
    - [Inference API của Silicon Flow (硅基流动)](https://modelscope.cn/models);
    - [Deepseek API](https://platform.deepseek.com/usage);
    - [OpenAI API](https://platform.openai.com/docs/quickstart);
    - Khác ...
  - Quy trình cấu hình: lấy API_KEY, MODEL_ID, BASE_URL rồi thiết lập trong biến môi trường ở file `.env`.
  - Cách lấy API mô hình của cộng đồng modelscope https://www.modelscope.cn/models/Qwen/Qwen3-VL-8B-Instruct
    - Nhấn vào kho mô hình (模型库), tìm mô hình hỗ trợ API-Inference, nhấn vào để vào trang chi tiết mô hình, tìm API-Inference
    - ![alt text](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra04-figures/3f1b68eedc9d9e556fbb51358bf49f9d.png)
    - ![alt text](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra04-figures/e7dd177f-4867-4af0-bd0e-03771a3a040e.png)


<strong>Q18. Khi triển khai luồng công việc ReAct, công cụ tìm kiếm web serpApi này còn có phương án thay thế nào không?</strong>

- Tóm tắt các điểm chính:
  - Cần "vượt tường lửa" (khoa học lên mạng), nếu không thể vượt tường lửa thì đổi phương án;
  - Có thể cân nhắc các công cụ tìm kiếm khác như: duckduckgo, googlesearch, v.v.

<strong>Q21. Mô hình suy luận đang dùng chỉ hỗ trợ đầu ra dạng luồng (streaming), không thể vào vòng lặp tiếp theo của Agent</strong>

- Tóm tắt các điểm chính:
  - Các mô hình suy luận như DeepSeek, Qwen mặc định chỉ cung cấp API dạng luồng (streaming), cần tự nối (ghép) đúng cách chứ không phải chỉ nối chuỗi (string concatenation) đơn giản (cách nối cụ thể có thể tham khảo chương 7)

<strong>Q23. Liên kết bộ nhớ với cơ sở tri thức (knowledge base), không biết nên hiểu thế nào.</strong>

- Tóm tắt các điểm chính:
  - Cách tôi hiểu là: lấy một ví dụ, thông thường agent có bộ nhớ ngắn hạn và bộ nhớ dài hạn. Bộ nhớ ngắn hạn tương đương với những việc chúng ta làm trong một ngày, chúng ta có thể trực tiếp đưa vào mô hình dưới dạng input thông qua ngữ cảnh. Nhưng bộ nhớ dài hạn tương đương với việc ghi chép, chúng ta không thể đưa ngần ấy thông tin vào ngữ cảnh cùng một lúc, nên chúng ta làm một công cụ để mô hình gọi, mô hình có thể sinh ra một query rồi thông qua RAG để truy hồi trong cơ sở tri thức của chúng ta, tức là bộ nhớ dài hạn.

<strong>Q24. Nguồn mirror Thanh Hoa (清华源) báo lỗi yêu cầu, 403</strong>

- Tóm tắt các điểm chính:
  - Do nguyên nhân mạng, đổi sang Đại học Khoa học và Công nghệ Trung Quốc (中科大):
    - https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
    - https://mirrors.ustc.edu.cn/anaconda/pkgs/free/

<strong>Q25. Gọi API mô hình báo lỗi 401</strong>

- Tóm tắt các điểm chính:
  - Số dư mô hình không đủ, cần nạp tiền.

**Q27. Mô hình lớn mã nguồn mở Hugging Face báo lỗi Connection aborted.**

- Tóm tắt các điểm chính:
  - Do nguyên nhân mạng, khuyến nghị cấu hình bằng HF_ENDPOINT
    - Ví dụ xem chi tiết tại code/chapter3/Qwen.py
    - https://hf-mirror.com
   
Nếu báo lỗi (MaxRetryError("HTTPSConnectionPool(host='huggingface.co', port=443) này
có thể thêm vào code
```
import os
os.environ["HF_ENDPOINT"] = "https://hf-mirror.com"
```
hoặc trực tiếp trên dòng lệnh
(Linux\mac)
```bash
export HF_ENDPOINT="https://hf-mirror.com"
```
(win's powshell)
```bash
$env:HF_ENDPOINT = "https://hf-mirror.com"
```


## 8. Vấn đề nền tảng toán học

<strong>Q22. Trong công thức xác suất, làm sao hiểu P(w_2∣w_1)</strong>

- Tóm tắt các điểm chính:
  - Đây là xác suất có điều kiện trong xác suất học, tức là xác suất w_2 xảy ra trong điều kiện w_1 đã xảy ra

## 9. Các vấn đề khác

### 9.1 Liên quan đến đồ án tốt nghiệp

<strong>Q26. Sau khi nộp đồ án tốt nghiệp, làm thế nào để trình bày trong CV và kho cá nhân?</strong>

- Tóm tắt các điểm chính:
  - Xem chi tiết mục 5 trong [Chương 16](https://github.com/datawhalechina/hello-agents/blob/main/docs/chapter16/第十六章%20毕业设计.md)
  - Trong CV khuyến nghị viết như sau (ví dụ):
    - "Trợ lý lập kế hoạch hành trình du lịch thông minh dựa trên framework Hello-Agents"
      - Phụ trách: thiết kế vai trò Agent, điều phối gọi công cụ, truy hồi RAG và đánh giá hội thoại
      - Hiệu quả: tự động sinh phương án hành trình nhiều ngày, hỗ trợ ràng buộc ngân sách và sở thích cá nhân hóa
      - Liên kết: `https://github.com/<your-id>/hello-agents/tree/main/projects/<your-folder>`
  - Khi phỏng vấn hãy nói rõ trọng tâm:
    - Vì sao lại thiết kế cấu trúc Agent như vậy;
    - Đã dùng những phương pháp đánh giá nào;
    - Đã gặp những vấn đề gì (chẳng hạn ngữ cảnh quá dài, chi phí gọi, v.v.) và bạn đã xử lý ra sao.

---
