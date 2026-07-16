# Tổng hợp câu hỏi phỏng vấn LLM & VLM & Agent

Tài liệu này là bộ tổng hợp các câu hỏi phỏng vấn "kiến thức nền tảng bắt buộc" (八股 - những kiến thức lý thuyết kinh điển thường được hỏi) được biên soạn trong quá trình chuẩn bị cho kỳ tuyển dụng mùa thu 2025.

Các vị trí mà tác giả chủ yếu ứng tuyển bao gồm: Kỹ sư thuật toán mô hình lớn (LLM), Kỹ sư Agent, Kỹ sư phát triển AI, Kỹ sư đánh giá thuật toán, v.v., với các công ty phỏng vấn chủ yếu là các doanh nghiệp Internet lớn và vừa trong nước (Trung Quốc). Vì vậy, độ sâu và độ rộng của các câu hỏi trong tài liệu này đều xoay quanh yêu cầu của những vị trí này, nội dung bao trùm toàn bộ chuỗi công nghệ (full-stack) từ lý thuyết cốt lõi của LLM/VLM, đến phát triển ứng dụng RAG/Agent, rồi đến kỹ thuật căn chỉnh RLHF và đánh giá mô hình/Agent. Tất cả câu hỏi đều được tổng hợp từ những trải nghiệm phỏng vấn kỹ thuật trực tuyến có thật qua nhiều lần.

【Gợi ý sử dụng】
Tài liệu này chỉ dùng cho mục đích học tập và tham khảo. Để đạt hiệu quả tốt nhất, tôi mạnh mẽ khuyến nghị bạn hãy tự suy nghĩ độc lập về từng câu hỏi trước, cố gắng xây dựng câu trả lời của riêng mình, sau đó mới đối chiếu với hướng tư duy tham khảo mà tài liệu cung cấp để rà soát và bổ sung những chỗ còn thiếu. Không chỉ biết "cái gì", mà còn phải hiểu "tại sao". Học thuộc lòng một cách máy móc là phương pháp kém hiệu quả nhất.

Chúc mọi người tìm việc thuận lợi, ai cũng nhận được Offer như ý!

---

### 1. Kiến thức nền tảng LLM

1.  Hãy giải thích chi tiết cơ chế self-attention (tự chú ý) trong mô hình Transformer hoạt động như thế nào? Tại sao nó phù hợp để xử lý chuỗi dài hơn so với RNN?
2.  Positional encoding (mã hóa vị trí) là gì? Trong Transformer, tại sao nó lại cần thiết? Hãy liệt kê ít nhất hai cách triển khai.
3.  Hãy giới thiệu chi tiết về ROPE, so sánh với mã hóa vị trí tuyệt đối, ưu và nhược điểm của nó lần lượt là gì?
4.  Bạn có biết sự khác biệt giữa MHA, MQA, GQA không? Hãy giải thích chi tiết.
5.  Hãy so sánh một vài kiến trúc LLM phổ biến, ví dụ Encoder-Only, Decoder-Only và Encoder-Decoder, đồng thời nêu rõ loại tác vụ mà mỗi loại giỏi nhất.
6.  Scaling Laws (định luật mở rộng quy mô) là gì? Nó tiết lộ mối quan hệ gì giữa hiệu năng mô hình, khối lượng tính toán và khối lượng dữ liệu? Điều này có ý nghĩa chỉ đạo gì đối với việc nghiên cứu và phát triển LLM?
7.  Ở giai đoạn suy luận (inference) của LLM, có những chiến lược giải mã (decoding) phổ biến nào? Hãy giải thích nguyên lý cùng ưu nhược điểm của Greedy Search, Beam Search, Top-K Sampling và Nucleus Sampling (Top-P).
8.  Tokenization (phân tách token) là gì? Hãy so sánh hai thuật toán phân tách từ con (subword) chủ đạo là BPE và WordPiece.
9.  Bạn nghĩ sự khác biệt lớn nhất giữa NLP và LLM là gì? Hai cái này có điểm gì chung và điểm gì khác nhau?
10. Chính quy hóa (regularization) L1 và L2 lần lượt là gì, và mỗi loại phù hợp sử dụng trong tình huống nào?
11. "Khả năng đột hiện" (emergent ability) là một hiện tượng rất được quan tâm ở các mô hình lớn, bạn hiểu khái niệm này như thế nào? Nó thường xuất hiện khi quy mô mô hình đạt đến mức độ nào?
12. Bạn có tìm hiểu về hàm kích hoạt (activation function) không, bạn biết những hàm kích hoạt nào thường dùng trong LLM? Tại sao lại chọn dùng chúng?
13. Mô hình hỗn hợp chuyên gia (MoE) làm thế nào để mở rộng hiệu quả quy mô tham số của mô hình mà không làm tăng đáng kể chi phí suy luận? Hãy trình bày sơ lược nguyên lý hoạt động của nó.
14. Khi huấn luyện một LLM ở quy mô hàng trăm hoặc hàng nghìn tỷ tham số, bạn sẽ đối mặt với những thách thức chính nào về mặt kỹ thuật và thuật toán? (Ví dụ: bộ nhớ hiển thị (VRAM), giao tiếp, tính bất ổn định khi huấn luyện, v.v.)
15. Bạn đã tìm hiểu qua những framework mã nguồn mở nào? Bạn đã nghiên cứu kỹ các bài báo của Qwen, Deepseek chưa, hãy nói xem điểm đổi mới trong đó chủ yếu thể hiện ở đâu?
16. Gần đây bạn đã đọc những bài báo LLM tiên tiến nào, hãy nói về các phương pháp liên quan của nó, nhắm vào vấn đề gì, đề xuất phương pháp gì, có những thí nghiệm so sánh (đối chứng) nào?

---

### 2. Kiến thức nền tảng VLM

1.  Thách thức cốt lõi của mô hình lớn đa phương thức (multimodal, như VLM) là gì? Tức là làm thế nào để thực hiện việc căn chỉnh (alignment) và hợp nhất (fusion) hiệu quả thông tin từ các phương thức khác nhau (như thị giác và ngôn ngữ)?
2.  Hãy giải thích nguyên lý hoạt động của mô hình CLIP. Nó kết nối hình ảnh và văn bản thông qua học tương phản (contrastive learning) như thế nào?
3.  Các mô hình như LLaVA hay MiniGPT-4 kết nối một bộ mã hóa thị giác (Vision Encoder) đã được huấn luyện trước với một mô hình ngôn ngữ lớn (LLM) như thế nào? Hãy mô tả thiết kế kiến trúc then chốt của chúng.
4.  Tinh chỉnh theo chỉ dẫn thị giác (visual instruction tuning) là gì? Tại sao nói nó là bước then chốt để VLM có được khả năng đối thoại và tuân theo chỉ dẫn tốt?
5.  Khi xử lý dữ liệu đa phương thức như video, so với ảnh tĩnh, VLM cần giải quyết thêm những vấn đề nào? (Ví dụ, làm thế nào để biểu diễn thông tin thời gian - temporal information?)
6.  Hãy giải thích ý nghĩa của Grounding trong lĩnh vực VLM. Chúng ta đánh giá như thế nào việc một VLM có thể ánh xạ mô tả văn bản một cách chính xác đến một vùng cụ thể trong hình ảnh?
7.  Hãy so sánh ít nhất các mô thức kiến trúc VLM khác nhau (như bộ mã hóa dùng chung - shared encoder vs. hợp nhất chú ý xuyên phương thức - cross-modal attention fusion) và phân tích ưu nhược điểm của chúng.
8.  Trong ứng dụng VLM, làm thế nào để xử lý ảnh đầu vào có độ phân giải cao? Điều này sẽ mang lại những thách thức gì về mặt tính toán và thiết kế mô hình?
9.  Khi tạo nội dung, VLM cũng gặp vấn đề "ảo giác" (Hallucination), nhưng hình thức biểu hiện của nó khác gì so với LLM thuần văn bản? Hãy nêu ví dụ minh họa.
10. Ngoài mô tả ảnh và hỏi đáp thị giác (VQA), bạn còn có thể liệt kê những hướng ứng dụng tiên tiến hoặc có tiềm năng nào của VLM?
11. Bạn đã từng làm về mặt tinh chỉnh (fine-tuning) liên quan đến VLM chưa? Mô hình nào?

---

### 3. Kiến thức nền tảng RLHF

1.  So với SFT truyền thống, RLHF nhằm giải quyết những vấn đề cốt lõi nào trong mô hình ngôn ngữ? Tại sao nói bản thân SFT không đủ để đạt được mục tiêu "căn chỉnh" (alignment) mà chúng ta kỳ vọng?
2.  Hãy trình bày chi tiết ba giai đoạn cốt lõi của quy trình RLHF kinh điển. Ở mỗi giai đoạn, đầu vào là gì, đầu ra là gì, và mục tiêu then chốt của giai đoạn đó là gì?
3.  Ở giai đoạn huấn luyện RM (Reward Model), chúng ta thường thu thập dữ liệu so sánh theo cặp (pairwise comparison), thay vì để người gán nhãn cho trực tiếp một điểm số tuyệt đối cho câu trả lời. Bạn cho rằng ưu điểm chính và nhược điểm tiềm ẩn của cách làm này lần lượt là gì?
4.  Thiết kế mô hình phần thưởng (reward model) là cực kỳ quan trọng. Kiến trúc mô hình của nó thường được lựa chọn như thế nào? Nó có quan hệ gì với LLM mà cuối cùng chúng ta muốn tối ưu? Khi huấn luyện mô hình phần thưởng, hàm mất mát (loss function) thường dùng là gì? Hãy giải thích nguyên lý toán học đằng sau nó (ví dụ, có thể kết hợp mô hình Bradley-Terry để giải thích).
5.  Ở giai đoạn thứ ba của RLHF, PPO là thuật toán học tăng cường (reinforcement learning) chủ đạo nhất. Tại sao lại chọn PPO, mà không phải các thuật toán policy gradient đơn giản hơn khác (như REINFORCE) hoặc các thuật toán dòng Q-learning? Số hạng phạt phân kỳ KL (KL divergence penalty) trong PPO đóng vai trò then chốt gì?
6.  Nếu trong quá trình huấn luyện PPO, hệ số β của số hạng phạt phân kỳ KL được đặt quá lớn hoặc quá nhỏ, thì mỗi trường hợp sẽ dẫn đến vấn đề gì? Bạn sẽ điều chỉnh siêu tham số (hyperparameter) này thông qua thí nghiệm và quan sát như thế nào?
7.  "Gian lận phần thưởng / tấn công phần thưởng" (Reward Hacking) là gì? Hãy đưa ra một ví dụ kết hợp với một tình huống ứng dụng LLM cụ thể, và thảo luận một vài chiến lược giảm thiểu khả thi.
8.  Quy trình RLHF phức tạp và không ổn định. Những năm gần đây đã xuất hiện một số giải pháp thay thế, ví dụ DPO. Hãy giải thích tư tưởng cốt lõi của DPO, và so sánh sự khác biệt chính cùng ưu điểm của nó so với RLHF truyền thống (dựa trên PPO).
9.  Hãy tưởng tượng, mô hình RLHF bạn huấn luyện xong thể hiện xuất sắc trong đánh giá offline, điểm số mô hình phần thưởng rất cao, nhưng sau khi lên môi trường thực tế, phản hồi của người dùng cho thấy câu trả lời của nó ngày càng "rập khuôn", nịnh nọt và thiếu thông tin hữu ích. Bạn cho rằng nguyên nhân có thể là gì? Bạn sẽ bắt tay phân tích và giải quyết vấn đề này từ những khía cạnh nào?
10. Bạn có biết GRPO của Deepseek không, sự khác biệt chính giữa nó và PPO là gì? Ưu nhược điểm là gì?
11. Bạn đã nghe qua GSPO và DAPO chưa? Chúng khác gì so với GRPO?
12. Làm thế nào để giải quyết vấn đề gán tín dụng (credit assignment)? Phần thưởng ở cấp độ token và cấp độ chuỗi (sequence) khác nhau như thế nào?
13. Ngoài phản hồi của con người, chúng ta còn có thể tận dụng phản hồi của chính AI để căn chỉnh, tức RLAIF. Hãy nói về cách hiểu của bạn về RLAIF, tiềm năng và rủi ro của nó lần lượt là gì?

---

### 4. Agent

1.  Bạn định nghĩa một tác nhân thông minh (Agent) dựa trên LLM như thế nào? Nó thường được cấu thành từ những thành phần cốt lõi nào?
2.  Hãy giải thích chi tiết framework ReAct. Nó kết hợp chuỗi suy luận (chain-of-thought) với hành động (action) như thế nào để hoàn thành các tác vụ phức tạp?
3.  Trong thiết kế Agent, "khả năng lập kế hoạch" (planning) là cực kỳ quan trọng. Hãy nói về hiện nay có những phương pháp chủ đạo nào có thể trao khả năng lập kế hoạch cho LLM? (Ví dụ CoT, ToT, GoT, v.v.)
4.  Memory (bộ nhớ) là một mô-đun then chốt của Agent. Vậy làm thế nào để thiết kế hệ thống bộ nhớ ngắn hạn và bộ nhớ dài hạn cho Agent? Có thể tận dụng những công cụ hoặc kỹ thuật bên ngoài nào?
5.  Tool Use (sử dụng công cụ) là một con đường hiệu quả để mở rộng năng lực của Agent. Hãy giải thích LLM học cách gọi API hoặc công cụ bên ngoài như thế nào? (Có thể giải thích từ góc độ Function Calling.)
6.  Hãy so sánh hai framework phát triển Agent phổ biến, như LangChain và LlamaIndex. Tình huống ứng dụng cốt lõi của chúng khác nhau ở đâu?
7.  Khi xây dựng một Agent phức tạp, bạn cho rằng thách thức chính yếu nhất là gì?
8.  Hệ thống đa tác nhân (multi-agent system) là gì? Việc để nhiều LLM Agent phối hợp làm việc có lợi thế gì so với một Agent đơn lẻ? Và nó sẽ đưa vào những sự phức tạp mới nào?
9.  Khi một Agent cần thực thi tác vụ trong môi trường thực tế hoặc mô phỏng (như robot, trò chơi), nó có sự khác biệt bản chất gì so với một Agent thuần túy dựa trên công cụ phần mềm?
10. Làm thế nào để đảm bảo hành vi của một Agent là an toàn, có thể kiểm soát và phù hợp với ý định của con người? Trong thiết kế Agent, có những phương pháp bảo đảm căn chỉnh (alignment) nào?
11. Bạn có tìm hiểu về framework A2A không? Nó khác gì so với framework Agent thông thường, hãy chọn một điểm khác biệt then chốt nhất để giải thích.
12. Bạn đã dùng qua những framework Agent nào? Bạn lựa chọn (selection) như thế nào? Chỉ số đánh giá cho tình huống cuối cùng của bạn là gì?
13. Bạn đã từng tinh chỉnh (fine-tune) năng lực Agent chưa? Bộ dữ liệu được thu thập như thế nào?

---

### 5. RAG

1.  Hãy giải thích nguyên lý hoạt động của RAG. So với việc tinh chỉnh (fine-tuning) LLM trực tiếp, RAG chủ yếu giải quyết vấn đề gì? Có những ưu điểm nào?
2.  Một pipeline RAG hoàn chỉnh bao gồm những bước then chốt nào? Hãy mô tả chi tiết toàn bộ quá trình từ chuẩn bị dữ liệu đến sinh (generation) cuối cùng.
3.  Khi xây dựng kho tri thức (knowledge base), chiến lược chia khối văn bản (chunking) là cực kỳ quan trọng. Bạn sẽ lựa chọn kích thước khối (chunk size) và độ dài chồng lấn (overlap) phù hợp như thế nào? Đằng sau đó có sự đánh đổi (trade-off) gì?
4.  Làm thế nào để lựa chọn một mô hình embedding (nhúng) phù hợp? Đánh giá một mô hình Embedding tốt hay dở có những chỉ số nào?
5.  Ngoài truy xuất vector (vector retrieval) cơ bản, bạn còn biết những kỹ thuật nào có thể nâng cao chất lượng truy xuất của RAG?
6.  Hãy giải thích vấn đề "Lost in the Middle". Nó mô tả hiện tượng gì trong RAG? Có phương pháp nào để giảm thiểu vấn đề này?
7.  Làm thế nào để đánh giá toàn diện hiệu năng của một hệ thống RAG? Hãy đề xuất các chỉ số đánh giá lần lượt từ hai giai đoạn truy xuất (retrieval) và sinh (generation).
8.  Trong tình huống nào, bạn sẽ chọn dùng cơ sở dữ liệu đồ thị (graph database) hoặc đồ thị tri thức (knowledge graph) để tăng cường hoặc thay thế truy xuất cơ sở dữ liệu vector truyền thống?
9.  Quy trình RAG truyền thống là "truy xuất trước rồi sinh sau", bạn có tìm hiểu về một số mô thức RAG phức tạp hơn không, ví dụ như thực hiện truy xuất nhiều lần hoặc truy xuất thích ứng (adaptive retrieval) trong quá trình sinh?
10. Hệ thống RAG có thể đối mặt với những thách thức nào khi triển khai thực tế?
11. Bạn có tìm hiểu về hệ thống tìm kiếm (search system) không? Nó khác gì so với RAG?
12. Bạn có biết hoặc đã dùng qua những framework RAG mã nguồn mở nào, ví dụ Ragflow? Làm thế nào để lựa chọn cho tình huống phù hợp?

---

### 6. Đánh giá mô hình và đánh giá Agent

1.  Tại sao các chỉ số đánh giá NLP truyền thống (như BLEU, ROUGE) lại tồn tại nhiều hạn chế lớn khi đánh giá chất lượng sinh của LLM hiện đại?
2.  Hãy giới thiệu một vài bộ benchmark tổng hợp cho LLM đang được sử dụng rộng rãi trong ngành hiện nay, và nêu rõ trọng tâm của từng loại. (Ví dụ: MMLU, Big-Bench, HumanEval)
3.  "LLM-as-a-Judge" là gì? Sử dụng LLM để đánh giá đầu ra của một LLM khác có những ưu điểm và thiên kiến (bias) tiềm ẩn nào?
4.  Làm thế nào để thiết kế một phương án đánh giá nhằm đo lường một năng lực cụ thể của LLM, ví dụ như "mức độ đúng sự thật / mức độ ảo giác", "khả năng suy luận" hoặc "tính an toàn"?
5.  Tại sao đánh giá một Agent lại khó khăn và phức tạp hơn so với đánh giá một LLM cơ bản? Các chiều đánh giá có gì khác nhau?
6.  Bạn biết những benchmark nào chuyên dùng để đánh giá năng lực Agent? Những benchmark này thường xây dựng môi trường thử nghiệm và tác vụ như thế nào?
7.  Khi đánh giá tình hình hoàn thành tác vụ của một Agent, ngoài tính đúng đắn của kết quả cuối cùng, còn có những chỉ số quá trình (process metrics) nào đáng chú ý? (Ví dụ: hiệu suất, chi phí, tính bền vững - robustness)
8.  Kiểm thử đội đỏ (red teaming) là gì? Nó đóng vai trò gì trong việc phát hiện các lỗ hổng an toàn và thiên kiến của LLM và Agent?
9.  Khi thực hiện đánh giá thủ công (human evaluation), làm thế nào để thiết kế tiêu chí và quy trình đánh giá hợp lý, nhằm đảm bảo tính khách quan và nhất quán của kết quả đánh giá?
10. Làm thế nào để liên tục giám sát và đánh giá hiệu năng của một ứng dụng LLM hoặc dịch vụ Agent đã được triển khai lên môi trường thực tế, nhằm ứng phó với sự suy giảm hiệu năng hoặc trôi dạt hành vi (behavior drift) có thể xảy ra?

---

### 7. Triển vọng và phát triển của LLM

1.  Bạn cho rằng LLM hiện tại còn cách trí tuệ nhân tạo tổng quát (AGI) bao xa? Năng lực còn thiếu then chốt nhất là gì?
2.  Từ GPT-4 đến các mô hình tương lai, bạn cho rằng sự hợp nhất đa phương thức sẽ đi về đâu? Chỉ đơn thuần là sự kết hợp giữa văn bản và hình ảnh, hay sẽ mở rộng đến nhiều chiều giác quan hơn?
3.  Bạn nhìn nhận thế nào về sự cạnh tranh và cùng tồn tại giữa hệ sinh thái mô hình mã nguồn mở và mô hình mã nguồn đóng? Lợi thế của mỗi bên là gì, và tương lai sẽ tiến hóa ra sao?
4.  Khi năng lực mô hình được tăng cường, "mô hình thế giới" (world model) hay khả năng mô phỏng nội tại của LLM cũng rất được quan tâm. Bạn hiểu khái niệm này như thế nào? Nó có ý nghĩa gì đối với việc thực hiện suy luận và lập kế hoạch ở cấp độ cao hơn?
5.  "Dữ liệu" là nhiên liệu để huấn luyện LLM. Bạn cho rằng dữ liệu tổng hợp nhân tạo (synthetic data) chất lượng cao sẽ đóng vai trò như thế nào trong việc huấn luyện mô hình trong tương lai?
6.  Trí tuệ hiện thân (Embodied AI), tức sự kết hợp giữa LLM và robot, được cho là làn sóng tiếp theo của AI. Bạn cho rằng LLM sẽ trao quyền năng cho robot như thế nào, và sẽ mang lại những thách thức nào?
7.  Cá nhân hóa là một hướng quan trọng của ứng dụng LLM. Trong quá trình hiện thực hóa một Agent hoặc trợ lý được cá nhân hóa cao độ, chúng ta nên cân bằng giữa hiệu quả, quyền riêng tư và an toàn như thế nào?
8.  Bạn cho rằng kiến trúc Transformer sẽ thống trị lĩnh vực này lâu dài không? Hay bạn nhìn thấy tiềm năng của các kiến trúc mới như mô hình không gian trạng thái (SSM, như Mamba)?
9.  Nhìn về tương lai 3-5 năm tới, bạn cho rằng công nghệ LLM và Agent có khả năng nhất sẽ đi đầu trong việc hiện thực hóa ứng dụng mang tính đột phá ở ngành hoặc lĩnh vực nào? Tại sao?

---

### 8. Khác

1.  Bạn cho rằng nút thắt lớn nhất hiện đang hạn chế năng lực và sự phổ cập của Agent là gì? (Ví dụ: năng lực mô hình, chi phí, độ tin cậy, hay điều gì khác?)
2.  Trong nửa năm qua, bài báo về Agent nào hoặc dự án mã nguồn mở nào để lại cho bạn ấn tượng sâu sắc nhất? Tại sao?
3.  Bạn nhìn nhận thế nào về "khả năng đột hiện" (emergent ability) trong lĩnh vực Agent? Chúng ta nên theo đuổi một mô hình nền tảng mạnh mẽ hơn, hay một kiến trúc Agent tinh xảo hơn?
4.  Bạn cho rằng trong 1-2 năm tới, công nghệ Agent có khả năng nhất sẽ đi đầu trong việc hiện thực hóa thương mại quy mô lớn ở ngành hoặc tình huống nào?
5.  Nếu để bạn tự do khám phá, bạn muốn tạo ra một Agent như thế nào nhất để giải quyết vấn đề gì?
6.  Đối với những người mới muốn bước vào lĩnh vực Agent, bạn sẽ đưa ra lời khuyên gì cho họ? Nên tập trung học những công nghệ nào?
7.  Tóm lại, bạn cho rằng một kỹ sư AI Agent hàng đầu nên có những tố chất cốt lõi nào?
8.  Bình thường bạn có dùng AI không, dùng để làm gì? Nếu tôi muốn dùng AI, ví dụ trong lĩnh vực coding, bạn có lời khuyên gì cho tôi?
