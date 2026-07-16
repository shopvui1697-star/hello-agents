# Chương 11 Agentic-RL

## 11.1 Từ huấn luyện LLM đến Agentic RL

Ở các chương trước, chúng ta đã triển khai nhiều mô hình (paradigm) Agent khác nhau cùng các giao thức giao tiếp. Tuy nhiên, khi Agent phải xử lý những nhiệm vụ phức tạp hơn, chúng lại hoạt động kém hiệu quả, và điều này tự nhiên dẫn tới những câu hỏi: **Làm sao để Agent có khả năng suy luận mạnh hơn? Làm sao để Agent học cách sử dụng công cụ tốt hơn? Làm sao để Agent có khả năng tự cải thiện?**

Đây chính là vấn đề cốt lõi mà Agentic RL (huấn luyện Agent dựa trên reinforcement learning) hướng tới giải quyết. Chương này sẽ giới thiệu năng lực huấn luyện bằng reinforcement learning vào framework HelloAgents, giúp bạn có thể huấn luyện các Agent với những năng lực cao cấp như suy luận và sử dụng công cụ. Chúng ta sẽ bắt đầu từ những kiến thức nền tảng về huấn luyện LLM, rồi dần đi sâu vào các kỹ thuật thực hành như Supervised Fine-Tuning (SFT) và Group Relative Policy Optimization (GRPO), cuối cùng xây dựng một pipeline huấn luyện Agent hoàn chỉnh.

### 11.1.1 Từ Reinforcement Learning đến Agentic RL

Trong mục 2.4.2 của Chương 2, chúng ta đã giới thiệu các Agent dựa trên reinforcement learning. Reinforcement Learning (RL - học tăng cường) là một mô hình học tập tập trung vào việc giải quyết các bài toán ra quyết định tuần tự. Nó học cách tối đa hóa phần thưởng dài hạn thông qua sự tương tác trực tiếp giữa Agent và môi trường, học theo kiểu "thử và sai".

Bây giờ, hãy áp dụng framework này cho các Agent LLM. Hãy xem xét một Agent giải toán cần trả lời những câu hỏi như thế này:

```
Question: Janet's ducks lay 16 eggs per day. She eats three for breakfast
every morning and bakes muffins for her friends every day with four.
She sells the remainder at the farmers' market daily for $2 per fresh
duck egg. How much in dollars does she make every day at the farmers' market?
```

Bài toán này đòi hỏi suy luận nhiều bước: trước hết tính số trứng còn lại mỗi ngày của Janet (16 - 3 - 4 = 9), rồi tính thu nhập của cô ấy (9 × 2 = 18). Chúng ta có thể ánh xạ nhiệm vụ này vào framework reinforcement learning:

- **Agent**: Hệ thống suy luận dựa trên LLM
- **Environment (Môi trường)**: Bài toán và hệ thống kiểm tra
- **State (Trạng thái)**: Mô tả bài toán hiện tại và các bước suy luận đã có
- **Action (Hành động)**: Sinh ra bước suy luận tiếp theo hoặc câu trả lời cuối cùng
- **Reward (Phần thưởng)**: Câu trả lời có đúng hay không (đúng +1, sai 0)

Các phương pháp học có giám sát (supervised learning) truyền thống có ba hạn chế cốt lõi: thứ nhất, chất lượng dữ liệu quyết định hoàn toàn chất lượng huấn luyện, mô hình chỉ có thể bắt chước dữ liệu huấn luyện nên khó vượt qua được; thứ hai, thiếu khả năng khám phá, chỉ học một cách thụ động những lối đi mà con người cung cấp; thứ ba, khó tối ưu các mục tiêu dài hạn, không thể tối ưu một cách chính xác các quá trình trung gian của suy luận nhiều bước.

Reinforcement learning mang đến những khả năng mới. Bằng cách cho phép Agent tự sinh ra nhiều câu trả lời ứng viên và nhận phần thưởng dựa trên độ đúng đắn, chúng có thể học được đâu là những lối suy luận tốt hơn, đâu là những bước then chốt, thậm chí khám phá ra những phương pháp giải quyết vấn đề tốt hơn cả chú thích của con người<sup>[8]</sup>. Đây chính là ý tưởng cốt lõi của Agentic RL: coi LLM như một policy (chính sách) có thể học được, nhúng nó vào vòng lặp cảm nhận - quyết định - thực thi của Agent, và tối ưu hiệu suất của nhiệm vụ nhiều bước thông qua reinforcement learning.

### 11.1.2 Bức tranh toàn cảnh về huấn luyện LLM

Trước khi đi sâu vào Agentic RL, chúng ta cần hiểu trước quy trình đầy đủ của việc huấn luyện LLM. Sự ra đời của một LLM mạnh mẽ (chẳng hạn GPT, Claude, Qwen) thường trải qua hai giai đoạn chính: Pretraining (tiền huấn luyện) và Post-training (hậu huấn luyện). Như minh họa ở Hình 11.1, hai giai đoạn này tạo nên con đường tiến hóa hoàn chỉnh của LLM từ "mô hình ngôn ngữ" thành "trợ lý hội thoại".

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-1.png" alt="" width="85%"/>
  <p>Hình 11.1 Bức tranh toàn cảnh về huấn luyện LLM</p>
</div>

**Giai đoạn Pretraining** là giai đoạn đầu tiên của việc huấn luyện LLM, với mục tiêu giúp mô hình học được các mẫu ngôn ngữ cơ bản và kiến thức về thế giới. Giai đoạn này sử dụng lượng dữ liệu văn bản khổng lồ (thường ở mức TB) và huấn luyện mô hình thông qua học tự giám sát (self-supervised learning), trong đó tín hiệu huấn luyện được xây dựng từ chính văn bản, chẳng hạn dự đoán từ tiếp theo dựa trên ngữ cảnh trước đó. Nhiệm vụ pretraining phổ biến nhất là Causal Language Modeling, còn gọi là Next Token Prediction (dự đoán token tiếp theo).

Cho một chuỗi văn bản $x_1, x_2, ..., x_t$, mô hình cần dự đoán từ tiếp theo $x_{t+1}$:

$$
\mathcal{L}_{\text{pretrain}} = -\sum_{t=1}^{T} \log P(x_t | x_1, x_2, ..., x_{t-1}; \theta)
$$

Trong đó $\theta$ là các tham số của mô hình, $P(x_t | x_1, ..., x_{t-1}; \theta)$ là phân phối xác suất của từ tiếp theo do mô hình dự đoán, và mục tiêu là tối thiểu hóa negative log-likelihood, tức là tối đa hóa xác suất dự đoán đúng từ. Ví dụ, cho văn bản "The cat sat on the", mô hình cần dự đoán rằng từ tiếp theo nhiều khả năng nhất là "mat". Thông qua việc huấn luyện trên lượng văn bản khổng lồ, mô hình dần học được các quy tắc ngữ pháp (chuỗi từ nào là hợp lệ), kiến thức ngữ nghĩa (mối quan hệ giữa các từ), kiến thức về thế giới (thông tin sự thật về thế giới), và các khả năng suy luận cơ bản.

Đặc điểm của giai đoạn pretraining là: lượng dữ liệu khổng lồ, chi phí tính toán cao, học được khả năng hiểu và sinh ngôn ngữ tổng quát, và sử dụng các mục tiêu tự giám sát được xây dựng từ văn bản chưa gán nhãn thay vì dữ liệu nhiệm vụ được gán nhãn thủ công.

**Giai đoạn Post-training** nhằm khắc phục những thiếu sót của mô hình đã được pretraining. Mặc dù các mô hình pretraining có năng lực ngôn ngữ mạnh mẽ, nhưng chúng chỉ là những mô hình "dự đoán từ tiếp theo" và không biết cách tuân theo chỉ dẫn của con người, sinh ra những câu trả lời hữu ích, vô hại và trung thực, từ chối các yêu cầu không phù hợp, và tương tác với con người theo kiểu hội thoại. Giai đoạn post-training nhằm giải quyết những vấn đề này và căn chỉnh (align) mô hình với các sở thích và giá trị của con người.

Post-training thường bao gồm ba bước. Bước đầu tiên là **Supervised Fine-Tuning (SFT)**<sup>[15]</sup>, với mục tiêu giúp mô hình học cách tuân theo chỉ dẫn và định dạng hội thoại. Dữ liệu huấn luyện gồm các cặp (prompt, completion), và mục tiêu huấn luyện tương tự như pretraining, vẫn là tối đa hóa xác suất của đầu ra đúng:

$$
\mathcal{L}_{\text{SFT}} = -\sum_{i=1}^{N} \log P(y_i | x_i; \theta)
$$

Trong đó $x_i$ là prompt đầu vào, $y_i$ là đầu ra mong đợi, và $N$ là số lượng mẫu huấn luyện. Đặc điểm của SFT là: lượng dữ liệu nhỏ hơn, cần gán nhãn thủ công, cho kết quả nhanh, chủ yếu học các định dạng nhiệm vụ và năng lực cơ bản.

Bước thứ hai là **Reward Modeling (RM - mô hình hóa phần thưởng)**. Mặc dù các mô hình SFT có thể tuân theo chỉ dẫn, nhưng chất lượng của câu trả lời sinh ra lại không đồng đều. Chúng ta cần một cách để đánh giá chất lượng câu trả lời, đó chính là vai trò của reward model (mô hình phần thưởng)<sup>[13,14]</sup>. Dữ liệu huấn luyện reward model gồm dữ liệu so sánh sở thích (preference comparison), chứa hai câu trả lời cho cùng một câu hỏi, một câu tốt hơn (chosen) và một câu tệ hơn (rejected). Mục tiêu huấn luyện reward model là học được sở thích của con người:

$$
\mathcal{L}_{\text{RM}} = -\mathbb{E}_{(x, y_w, y_l)} [\log \sigma(r_\phi(x, y_w) - r_\phi(x, y_l))]
$$

Trong đó $r_\phi(x, y)$ là reward model, đầu vào là cặp (prompt, câu trả lời), đầu ra là điểm chất lượng; $y_w$ là câu trả lời tốt hơn (chosen), $y_l$ là câu trả lời tệ hơn (rejected), $\sigma$ là hàm sigmoid, và mục tiêu là làm cho reward model cho điểm cao hơn với những câu trả lời tốt hơn.

Bước thứ ba là **Reinforcement Learning Fine-tuning (tinh chỉnh bằng học tăng cường)**. Với reward model, chúng ta có thể dùng reinforcement learning để tối ưu mô hình ngôn ngữ sinh ra những câu trả lời chất lượng cao hơn. Thuật toán kinh điển nhất là PPO (Proximal Policy Optimization)<sup>[1]</sup>, với mục tiêu huấn luyện:

$$
J_{\text{PPO}} = \mathbb{E}_{x, y \sim \pi_\theta} [r_\phi(x, y)] - \beta \cdot D_{KL}(\pi_\theta || \pi_{\text{ref}})
$$

Trong đó $\pi_\theta$ là policy hiện tại, tức là mô hình ngôn ngữ, $\pi_{\text{ref}}$ là policy tham chiếu, trong kịch bản này có thể là mô hình SFT, $r_\phi(x, y)$ là điểm của reward model, $D_{KL}$ là KL divergence (độ phân kỳ KL), nhằm ngăn mô hình đi chệch quá xa, và $\beta$ là hệ số cân bằng. Ý nghĩa của hàm mục tiêu này là: tối đa hóa phần thưởng đồng thời không đi chệch quá xa so với mô hình ban đầu.

RLHF (Reinforcement Learning from Human Feedback)<sup>[5]</sup> truyền thống đòi hỏi một lượng lớn dữ liệu gán nhãn sở thích thủ công, vốn rất tốn kém. Để giảm chi phí, các nhà nghiên cứu đề xuất RLAIF (Reinforcement Learning from AI Feedback)<sup>[7]</sup>, sử dụng các mô hình AI mạnh mẽ (như GPT-4) để thay thế người gán nhãn. Quy trình RLAIF là: dùng mô hình SFT sinh ra nhiều câu trả lời ứng viên, dùng mô hình AI mạnh để chấm điểm và xếp hạng các câu trả lời, dùng điểm của AI để huấn luyện reward model, dùng reward model để thực hiện reinforcement learning. Các thí nghiệm cho thấy hiệu quả của RLAIF gần bằng hoặc thậm chí vượt RLHF, trong khi chi phí giảm đáng kể<sup>[11]</sup>.

### 11.1.3 Triết lý cốt lõi của Agentic RL

Sau khi đã hiểu quy trình huấn luyện cơ bản của LLM, hãy cùng xem sự khác biệt giữa Agentic RL và các phương pháp huấn luyện truyền thống. Post-training truyền thống (mà chúng ta gọi là PBRFT: Preference-Based Reinforcement Fine-Tuning) chủ yếu tập trung vào tối ưu chất lượng hội thoại một lượt: cho một câu hỏi của người dùng, mô hình sinh ra một câu trả lời, rồi nhận một phần thưởng dựa trên chất lượng câu trả lời. Cách tiếp cận này phù hợp để tối ưu các trợ lý hội thoại, nhưng đối với những nhiệm vụ Agent đòi hỏi suy luận nhiều bước, sử dụng công cụ và lập kế hoạch dài hạn thì nó tỏ ra thiếu sót.

**Agentic RL** là một mô hình (paradigm) mới coi LLM như một policy có thể học được, được nhúng vào một vòng lặp ra quyết định tuần tự. Trong framework này, Agent cần tương tác với thế giới bên ngoài trong các môi trường động, thực thi các hành động nhiều bước để hoàn thành các nhiệm vụ phức tạp, nhận phản hồi trung gian để dẫn dắt các quyết định về sau, và tối ưu phần thưởng tích lũy dài hạn thay vì phần thưởng một bước.

Hãy hiểu sự khác biệt này qua một ví dụ cụ thể. Trong kịch bản PBRFT, người dùng hỏi "Hãy giải thích reinforcement learning là gì", mô hình sinh ra một câu trả lời hoàn chỉnh, rồi chấm điểm trực tiếp dựa trên chất lượng câu trả lời. Trong kịch bản Agentic RL, người dùng yêu cầu "Hãy giúp tôi phân tích chất lượng mã của repository GitHub này", Agent cần trải qua nhiều bước: đầu tiên gọi GitHub API để lấy thông tin repository, lấy được thành công cấu trúc repository và danh sách file, nhận thưởng +0.1; sau đó đọc các file mã chính, lấy được thành công nội dung mã, nhận thưởng +0.1; tiếp theo phân tích chất lượng mã một cách hợp lý, nhận thưởng +0.2; cuối cùng sinh ra báo cáo phân tích chất lượng cao, nhận thưởng +0.6. Tổng phần thưởng là sự tích lũy của tất cả các bước: 1.0.

Như có thể thấy, các đặc điểm then chốt của Agentic RL là tương tác nhiều bước, mỗi hành động thay đổi trạng thái môi trường, mỗi bước có thể nhận phản hồi, và tối ưu chất lượng hoàn thành tổng thể của nhiệm vụ.

Reinforcement learning thường được hình thức hóa bằng framework Markov Decision Process (MDP - quá trình quyết định Markov). Một MDP được định nghĩa bởi bộ năm phần tử $(S, A, P, R, \gamma)$: không gian trạng thái $S$, không gian hành động $A$, hàm chuyển trạng thái $P(s'|s,a)$, hàm phần thưởng $R(s,a)$, và hệ số chiết khấu (discount factor) $\gamma$. Bảng dưới đây so sánh PBRFT và Agentic RL từ góc nhìn MDP.

<div align="center">
  <p>Bảng 11.1 So sánh PBRFT và Agentic RL</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-1.png" alt="" width="85%"/>
</div>

Về mặt trạng thái, trạng thái $s_0$ của PBRFT chỉ gồm prompt của người dùng, khoảng thời gian $T=1$ (một bước), trạng thái không thay đổi, có thể biểu diễn là $s_0 = \text{prompt}$. Trong khi đó, trạng thái $s_t$ của Agentic RL chứa các quan sát và ngữ cảnh lịch sử, khoảng thời gian $T \gg 1$ (nhiều bước), trạng thái tiến hóa theo các hành động, có thể biểu diễn là $s_t = (\text{prompt}, o_1, o_2, ..., o_t)$, trong đó $o_t$ là quan sát ở bước $t$ (chẳng hạn kết quả trả về của công cụ, phản hồi của môi trường, v.v.).

Về mặt hành động, không gian hành động của PBRFT chỉ có sinh văn bản, một loại hành động duy nhất, biểu diễn là $a = y \sim \pi_\theta(y|s_0)$. Trong khi đó, không gian hành động của Agentic RL bao gồm sinh văn bản, gọi công cụ, thao tác môi trường và các loại khác, biểu diễn là $a_t \in \{a_t^{\text{text}}, a_t^{\text{tool}}\}$, ví dụ $a_t^{\text{text}}$ là sinh quá trình suy nghĩ hoặc câu trả lời, $a_t^{\text{tool}}$ là gọi máy tính, công cụ tìm kiếm và các công cụ khác.

Về mặt hàm chuyển trạng thái, PBRFT không có chuyển trạng thái, biểu diễn là $P(s'|s,a) = \delta(s' - s_{\text{terminal}})$. Trong khi đó, trạng thái của Agentic RL thay đổi động dựa trên các hành động và môi trường, biểu diễn là $s_{t+1} \sim P(s_{t+1}|s_t, a_t)$, ví dụ sau khi gọi công cụ tìm kiếm, trạng thái sẽ bao gồm kết quả tìm kiếm.

Về mặt phần thưởng, PBRFT chỉ có phần thưởng một bước $r(s_0, a)$, chỉ được cho ở cuối nhiệm vụ, biểu diễn là $R_{\text{PBRFT}} = r(s_0, y)$, thường do reward model cho: $r(s_0, y) = r_\phi(s_0, y)$. Trong khi đó, Agentic RL có phần thưởng nhiều bước $r(s_t, a_t)$, có thể cho phần thưởng một phần ở các bước trung gian, biểu diễn là:

$$
R_{\text{Agentic}} = \sum_{t=0}^{T} \gamma^t r(s_t, a_t)
$$

Trong đó $\gamma \in [0,1]$ là hệ số chiết khấu, $r(s_t, a_t)$ có thể là sparse reward (phần thưởng thưa, chỉ cho khi hoàn thành nhiệm vụ, chẳng hạn câu trả lời đúng +1), dense reward (phần thưởng dày, cho ở mỗi bước, chẳng hạn gọi công cụ thành công +0.1), hoặc kết hợp cả hai.

Về mặt hàm mục tiêu, PBRFT tối đa hóa phần thưởng kỳ vọng một bước:

$$
J_{\text{PBRFT}}(\theta) = \mathbb{E}_{s_0, y \sim \pi_\theta} [r(s_0, y)]
$$

Trong khi đó, Agentic RL tối đa hóa phần thưởng chiết khấu tích lũy:

$$
J_{\text{Agentic}}(\theta) = \mathbb{E}_{\tau \sim \pi_\theta} \left[\sum_{t=0}^{T} \gamma^t r(s_t, a_t)\right]
$$

Trong đó $\tau = (s_0, a_0, s_1, a_1, ..., s_T)$ là quỹ đạo (trajectory) hoàn chỉnh.

Sự chuyển đổi này không chỉ là khác biệt về chi tiết kỹ thuật, mà là một sự thay đổi cơ bản trong tư duy. Tư duy PBRFT tập trung vào "làm sao để mô hình sinh ra câu trả lời đơn lẻ tốt hơn", tối ưu chất lượng câu trả lời, chú trọng biểu đạt ngôn ngữ, ra quyết định một bước. Trong khi đó, tư duy Agentic RL tập trung vào "làm sao để Agent hoàn thành các nhiệm vụ phức tạp", tối ưu việc hoàn thành nhiệm vụ, chú trọng chiến lược hành động, lập kế hoạch nhiều bước. Sự chuyển đổi này cho phép LLM tiến hóa từ "trợ lý hội thoại" thành "Agent tự chủ", có khả năng chủ động tìm kiếm thông tin, biết khi nào và làm sao để dùng công cụ bên ngoài, sẵn sàng thực hiện những bước trung gian dường như "vòng vo" vì mục tiêu cuối cùng, và học hỏi từ sai lầm.

Agentic RL nhằm trang bị cho các Agent LLM sáu năng lực cốt lõi, như minh họa ở Hình 11.2.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-2.png" alt="" width="85%"/>
  <p>Hình 11.2 Sáu năng lực cốt lõi của Agentic RL</p>
</div>

**Reasoning (Suy luận)** là quá trình rút ra kết luận một cách logic từ thông tin cho trước, đây là năng lực cốt lõi của Agent. Các phương pháp prompting CoT truyền thống dựa vào các ví dụ few-shot với khả năng khái quát hóa hạn chế; SFT chỉ có thể bắt chước các mẫu suy luận trong dữ liệu huấn luyện, khó lòng đổi mới. Ưu điểm của reinforcement learning là học được các chiến lược suy luận hiệu quả thông qua thử-sai, khám phá những lối suy luận không có trong dữ liệu huấn luyện, học được khi nào cần suy nghĩ sâu và khi nào có thể trả lời nhanh. Các nhiệm vụ suy luận có thể được mô hình hóa như những bài toán ra quyết định tuần tự. Cho câu hỏi $q$, Agent cần sinh ra chuỗi suy luận $c = (c_1, c_2, ..., c_n)$ và câu trả lời cuối cùng $a$. Hàm phần thưởng thường được thiết kế là $r(q, c, a) = 1$ nếu $a = a^*$ và ngược lại là $0$, với mục tiêu huấn luyện $\max_\theta \mathbb{E}_{q, (c,a) \sim \pi_\theta} [r(q, c, a)]$. Thông qua cách tiếp cận này, mô hình học được cách sinh ra các chuỗi suy luận chất lượng cao, chứ không chỉ ghi nhớ các câu trả lời.

**Tool Use (Sử dụng công cụ)** là khả năng của Agent gọi các công cụ bên ngoài để hoàn thành nhiệm vụ. Trong các nhiệm vụ sử dụng công cụ, không gian hành động mở rộng thành $a_t \in \{a_t^{\text{think}}, a_t^{\text{tool}}\}$, trong đó $a_t^{\text{think}}$ là sinh quá trình suy nghĩ, $a_t^{\text{tool}} = (\text{tool\_name}, \text{arguments})$ là gọi công cụ. Reinforcement learning cho phép Agent học được khi nào dùng công cụ, chọn công cụ nào, và làm sao kết hợp nhiều công cụ. Ví dụ, khi giải toán, Agent cần học được khi nào dùng máy tính, khi nào dùng trình thông dịch mã, và khi nào suy luận trực tiếp.

**Memory (Bộ nhớ)** là khả năng của Agent lưu giữ và tái sử dụng thông tin trong quá khứ, điều này cực kỳ quan trọng đối với các nhiệm vụ dài hạn. Cửa sổ ngữ cảnh (context window) của LLM có giới hạn, và các chiến lược truy xuất tĩnh (như RAG) không thể được tối ưu cho nhiệm vụ. Reinforcement learning cho phép Agent học các chiến lược quản lý bộ nhớ: quyết định thông tin nào đáng ghi nhớ, khi nào cập nhật bộ nhớ, và khi nào xóa thông tin lỗi thời. Điều này tương tự bộ nhớ làm việc (working memory) của con người, khi chúng ta chủ động quản lý thông tin trong não bộ, giữ lại thông tin quan trọng và quên đi thông tin không liên quan.

**Planning (Lập kế hoạch)** là khả năng lập ra các chuỗi hành động để đạt được mục tiêu. CoT truyền thống là tư duy tuyến tính và không thể quay lui; kỹ thuật prompt sử dụng các mẫu lập kế hoạch tĩnh vốn khó thích ứng với các tình huống mới. Reinforcement learning cho phép Agent học lập kế hoạch động: khám phá các chuỗi hành động hiệu quả thông qua thử-sai, học cách cân bằng lợi ích ngắn hạn và dài hạn. Ví dụ, trong các nhiệm vụ nhiều bước, Agent có thể cần thực hiện một số bước dường như "vòng vo" trước, chẳng hạn thu thập thông tin, trước khi cuối cùng hoàn thành nhiệm vụ.

**Self-Improvement (Tự cải thiện)** là khả năng của Agent xem xét lại đầu ra của chính mình, sửa lỗi, và tối ưu chiến lược. Reinforcement learning cho phép Agent học tự phản tỉnh: nhận diện lỗi của chính mình, phân tích nguyên nhân thất bại, và điều chỉnh chiến lược. Năng lực này cho phép Agent liên tục cải thiện mà không cần can thiệp của con người, tương tự việc con người "học hỏi từ sai lầm".

**Perception (Cảm nhận)** là khả năng hiểu thông tin đa phương thức (multimodal). Ví dụ, reinforcement learning có thể tăng cường năng lực suy luận thị giác, cho phép mô hình học cách dùng các công cụ thị giác và học lập kế hoạch thị giác. Điều này cho phép Agent không chỉ hiểu văn bản mà còn hiểu và thao tác trong thế giới thị giác.

### 11.1.4 Thiết kế Agentic RL của HelloAgents

Sau khi đã hiểu triết lý cốt lõi của Agentic RL, hãy cùng xem cách triển khai những năng lực này trong framework HelloAgents.

Về lựa chọn công nghệ, chúng ta tích hợp framework TRL (Transformer Reinforcement Learning)<sup>[9]</sup> và chọn mô hình Qwen3-0.6B<sup>[10]</sup>. TRL là thư viện reinforcement learning của Hugging Face, trưởng thành và ổn định, đầy đủ tính năng, và dễ tích hợp. Qwen3-0.6B là mô hình ngôn ngữ nhỏ của Alibaba Cloud, với 0.6B tham số phù hợp để huấn luyện trên GPU thông thường, hiệu năng xuất sắc, mã nguồn mở và miễn phí.

Module Agentic RL của HelloAgents áp dụng thiết kế kiến trúc bốn lớp, như minh họa ở Hình 11.3.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-3.png" alt="" width="85%"/>
  <p>Hình 11.3 Kiến trúc Agentic RL của HelloAgents</p>
</div>

Lớp dưới cùng là **Dataset Layer (Lớp tập dữ liệu)**, chứa lớp `GSM8KDataset`, hàm `create_sft_dataset()`, và hàm `create_rl_dataset()`, chịu trách nhiệm nạp dữ liệu và chuyển đổi định dạng. Lớp thứ hai là **Reward Function Layer (Lớp hàm phần thưởng)**, chứa lớp cơ sở `MathRewardFunction`, `AccuracyReward` (phần thưởng độ chính xác), `LengthPenaltyReward` (phạt độ dài), `StepReward` (phần thưởng bước), và các hàm tạo tiện lợi `create_*_reward()`, chịu trách nhiệm định nghĩa thế nào là hành vi tốt. Lớp thứ ba là **Trainer Layer (Lớp huấn luyện)**, chứa `SFTTrainerWrapper` và `GRPOTrainerWrapper`, chịu trách nhiệm về logic huấn luyện cụ thể và hỗ trợ LoRA. Lớp trên cùng là **Unified Interface Layer (Lớp giao diện thống nhất)**, cung cấp công cụ huấn luyện thống nhất `RLTrainingTool`, hỗ trợ bốn thao tác: `action="train"` (huấn luyện mô hình), `action="load_dataset"` (nạp tập dữ liệu), `action="create_reward"` (tạo hàm phần thưởng), `action="evaluate"` (đánh giá mô hình).

### 11.1.5 Ví dụ bắt đầu nhanh

Trước khi đi sâu vào học, hãy nhanh chóng trải nghiệm quy trình huấn luyện hoàn chỉnh. Vì chương này có nhiều nội dung lý thuyết và việc gỡ lỗi thực hành khá rườm rà, nên chúng ta tập trung vào học cách ứng dụng thay vì xây dựng công cụ. Trước hết cài đặt framework HelloAgents:

```bash
# Install HelloAgents framework (Chapter 11 version)
pip install "hello-agents[rl]==0.2.5"

# Or install from source
cd HelloAgents
pip install -e ".[rl]"
```

Sau đó chạy ví dụ huấn luyện nhanh:

```python
import sys
import json

from hello_agents.tools import RLTrainingTool

# Create RL training tool
rl_tool = RLTrainingTool()

# 1. Quick test: SFT training (10 samples, 1 epoch)
sft_result_str = rl_tool.run({
    "action": "train",
    "algorithm": "sft",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/quick_test_sft",
    "max_samples": 10,      # Only use 10 samples for quick test
    "num_epochs": 1,        # Only train 1 epoch
    "batch_size": 2,
    "use_lora": True        # Use LoRA to accelerate training
})

sft_result = json.loads(sft_result_str)
print(f"\n✓ SFT training completed, model saved at: {sft_result['output_dir']}")

# 2. GRPO training (5 samples, 1 epoch)
grpo_result_str = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",  # Use base model
    "output_dir": "./models/quick_test_grpo",
    "max_samples": 5,       # Only use 5 samples for quick test
    "num_epochs": 1,
    "batch_size": 2,        # Must be divisible by num_generations(8), use 2
    "use_lora": True
})

grpo_result = json.loads(grpo_result_str)
print(f"\n✓ GRPO training completed, model saved at: {grpo_result['output_dir']}")

# 3. Evaluate model
eval_result_str = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/quick_test_grpo",
    "max_samples": 10,      # Evaluate on 10 test samples
    "use_lora": True
})

eval_result = json.loads(eval_result_str)
print(f"\n✓ Evaluation completed:")
print(f"  - Accuracy: {eval_result['accuracy']}")
print(f"  - Average reward: {eval_result['average_reward']}")
print(f"  - Test samples: {eval_result['num_samples']}")

print("\n" + "=" * 50)
print("🎉 Congratulations! You have completed training your first Agentic RL model!")
print("=" * 50)
print(f"\nModel paths:")
print(f"  SFT model: {sft_result['output_dir']}")
print(f"  GRPO model: {grpo_result['output_dir']}")
```

Ví dụ nhanh này minh họa quy trình huấn luyện hoàn chỉnh: huấn luyện SFT giúp mô hình học các định dạng suy luận cơ bản và các mẫu hội thoại, huấn luyện GRPO tối ưu chiến lược suy luận thông qua reinforcement learning để cải thiện độ chính xác, và đánh giá mô hình đánh giá hiệu quả huấn luyện trên tập kiểm tra. Ngoài ra, việc độ chính xác rất thấp sau khi chạy là bình thường, vì mô hình mới chỉ thấy 0.7% mẫu huấn luyện và chỉ chạy một epoch.

## 11.2 Tập dữ liệu và hàm phần thưởng

Tập dữ liệu và hàm phần thưởng là hai nền tảng của việc huấn luyện reinforcement learning. Tập dữ liệu định nghĩa những nhiệm vụ mà Agent cần học, còn hàm phần thưởng định nghĩa thế nào là hành vi tốt. Trong mục này, chúng ta sẽ học cách chuẩn bị dữ liệu huấn luyện và thiết kế hàm phần thưởng.

### 11.2.1 Tập dữ liệu suy luận toán học GSM8K

Suy luận toán học là một nhiệm vụ lý tưởng để đánh giá năng lực suy luận của LLM. Thứ nhất, các bài toán có câu trả lời đúng rõ ràng, có thể đánh giá tự động mà không cần gán nhãn thủ công hay reward model phức tạp. Thứ hai, giải toán đòi hỏi phân rã bài toán và suy diễn từng bước, đây là kịch bản điển hình của suy luận nhiều bước. Cuối cùng, năng lực suy luận học được có thể chuyển giao sang các lĩnh vực khác với khả năng khái quát hóa mạnh. Ngược lại, các nhiệm vụ hỏi-đáp mở (chẳng hạn "Làm sao để học lập trình?") có chất lượng câu trả lời khó đánh giá một cách khách quan và đòi hỏi gán nhãn thủ công nhiều.

GSM8K (Grade School Math 8K)<sup>[4]</sup> là một tập dữ liệu bài toán đố (word problem) toán tiểu học chất lượng cao. Như minh họa ở Bảng 11.2, tập dữ liệu chứa 7.473 mẫu huấn luyện và 1.319 mẫu kiểm tra, với độ khó ở mức toán tiểu học (lớp 2-8), loại bài là bài toán đố, đòi hỏi 2-8 bước suy luận để đi đến đáp án.

<div align="center">
  <p>Bảng 11.2 Thống kê tập dữ liệu GSM8K</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-2.png" alt="" width="85%"/>
</div>

Hãy xem một bài toán GSM8K điển hình:

```
Question: Natalia sold clips to 48 of her friends in April, and then she sold half
      as many clips in May. How many clips did Natalia sell altogether in April
      and May?

Answer: Natalia sold 48/2 = <<48/2=24>>24 clips in May.
      Natalia sold 48+24 = <<48+24=72>>72 clips altogether in April and May.
      #### 72

Final Answer: 72
```

Bài toán này đòi hỏi hai bước suy luận: trước hết tính số lượng bán trong tháng Năm (một nửa của 48), rồi tính tổng (tháng Tư + tháng Năm). Phần `<<48/2=24>>` trong câu trả lời là ký hiệu đánh dấu các bước tính toán trung gian, và `#### 72` đánh dấu câu trả lời cuối cùng.

Tập dữ liệu GSM8K cần được chuyển đổi sang các định dạng khác nhau để thích ứng với các phương pháp huấn luyện khác nhau, như minh họa ở Hình 11.4.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-4.png" alt="" width="85%"/>
  <p>Hình 11.4 Chuyển đổi định dạng dữ liệu GSM8K</p>
</div>


Định dạng gốc đến trực tiếp từ tập dữ liệu, chứa câu hỏi và câu trả lời (kèm các bước giải), phù hợp cho con người đọc. Định dạng SFT được dùng cho supervised fine-tuning, chuyển câu hỏi thành prompt định dạng hội thoại, với lời giải hoàn chỉnh làm completion. Ví dụ:

```python
{
    "prompt": "<|im_start|>user\nNatalia sold clips to 48 of her friends...<|im_end|>\n<|im_start|>assistant\n",
    "completion": "Let me solve this step by step.\n\nStep 1: ...\n\nFinal Answer: 72<|im_end|>"
}
```

Các điểm mấu chốt là sử dụng mẫu hội thoại của mô hình (chẳng hạn ký hiệu `<|im_start|>` của Qwen), prompt chứa câu hỏi của người dùng, completion chứa toàn bộ quá trình giải và câu trả lời. Như vậy mô hình có thể học cách định dạng đầu ra và cách suy luận từng bước.

Định dạng RL được dùng cho reinforcement learning, chỉ cung cấp câu hỏi và câu trả lời đúng, không có quá trình giải. Ví dụ:

```python
{
    "prompt": "<|im_start|>user\nNatalia sold clips to 48 of her friends...<|im_end|>\n<|im_start|>assistant\n",
    "ground_truth": "72"
}
```

Các điểm mấu chốt là prompt giống như SFT, nhưng ground_truth chỉ chứa câu trả lời cuối cùng (dùng để tính phần thưởng), và mô hình phải tự sinh ra toàn bộ quá trình suy luận. Thiết kế này buộc mô hình phải học suy luận tự chủ thay vì chỉ ghi nhớ câu trả lời một cách đơn giản.

Như minh họa ở Bảng 11.3, ba định dạng đều có công dụng riêng.

<div align="center">
  <p>Bảng 11.3 So sánh các định dạng dữ liệu</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-3.png" alt="" width="85%"/>
</div>

HelloAgents cung cấp các hàm nạp tập dữ liệu tiện lợi. Hãy nạp và xem tập dữ liệu qua đoạn mã sau:

```python
from hello_agents.tools import RLTrainingTool
import json

# Create tool
rl_tool = RLTrainingTool()

# 1. Load SFT format dataset
sft_result = rl_tool.run({
    "action": "load_dataset",
    "format": "sft",
    "max_samples": 5  # Only load 5 samples to view
})
sft_data = json.loads(sft_result)

print(f"Dataset size: {sft_data['dataset_size']}")
print(f"Data format: {sft_data['format']}")
print(f"Sample keys: {sft_data['sample_keys']}")

# 2. Load RL format dataset
rl_result = rl_tool.run({
    "action": "load_dataset",
    "format": "rl",
    "max_samples": 5
})
rl_data = json.loads(rl_result)

print(f"Dataset size: {rl_data['dataset_size']}")
print(f"Data format: {rl_data['format']}")
print(f"Sample keys: {rl_data['sample_keys']}")
```

Như có thể thấy, định dạng SFT chứa toàn bộ quá trình giải để học có giám sát; định dạng RL chỉ chứa câu trả lời cuối cùng, và mô hình phải tự sinh ra quá trình suy luận. Tham số `max_samples` kiểm soát số lượng mẫu được nạp, thuận tiện cho việc kiểm thử nhanh.

### 11.2.2 Thiết kế hàm phần thưởng

Hàm phần thưởng là cốt lõi của reinforcement learning, định nghĩa thế nào là "hành vi tốt". Một hàm phần thưởng tốt có thể dẫn dắt Agent học được các chiến lược đúng đắn, trong khi một hàm phần thưởng kém có thể dẫn tới thất bại trong huấn luyện hoặc học phải những hành vi sai.

Trong reinforcement learning, hàm phần thưởng $r(s, a)$ hoặc $r(s, a, s')$ gán một phần thưởng dạng số cho mỗi hành động của Agent. Mục tiêu của Agent là tối đa hóa phần thưởng tích lũy:

$$
J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta} \left[\sum_{t=0}^{T} \gamma^t r(s_t, a_t)\right]
$$

Đối với các nhiệm vụ suy luận toán học, chúng ta có thể đơn giản hóa thành:

$$
r(q, a) = f(a, a^*)
$$

Trong đó $q$ là câu hỏi, $a$ là câu trả lời do mô hình sinh ra, $a^*$ là câu trả lời đúng, và $f$ là hàm đánh giá.

Thiết kế hàm phần thưởng ảnh hưởng trực tiếp tới hiệu quả huấn luyện. Hàm phần thưởng tốt cần định nghĩa rõ ràng thế nào là thành công, cung cấp tín hiệu gradient, không tạo ra phương sai (variance) quá lớn, và dễ điều chỉnh và kết hợp. Hàm phần thưởng kém có thể chỉ cho phần thưởng ở cuối nhiệm vụ mà không có phản hồi trung gian, gặp phải reward hacking khi Agent tìm ra cách "gian lận" để nhận phần thưởng cao, có nhiều mục tiêu mâu thuẫn nhau, hoặc có phương sai quá lớn khiến không hội tụ.

HelloAgents cung cấp ba hàm phần thưởng dựng sẵn, có thể dùng riêng lẻ hoặc kết hợp, như minh họa ở Hình 11.5.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-5.png" alt="" width="85%"/>
  <p>Hình 11.5 Thiết kế hàm phần thưởng</p>
</div>

**(1) Accuracy Reward (Phần thưởng độ chính xác)**

Accuracy Reward (AccuracyReward) là hàm phần thưởng cơ bản nhất, chỉ quan tâm câu trả lời có đúng hay không. Định nghĩa toán học:

$$
r_{\text{acc}}(a, a^*) = \begin{cases}
1 & \text{if } a = a^* \\
0 & \text{otherwise}
\end{cases}
$$

Trong đó $a$ là câu trả lời do mô hình sinh ra và $a^*$ là câu trả lời đúng. Đây là một hàm phần thưởng nhị phân, được 1 điểm cho câu trả lời đúng và 0 cho câu trả lời sai.

Việc triển khai đòi hỏi xử lý trích xuất và so sánh câu trả lời. Đầu ra của mô hình có thể chứa lượng lớn văn bản, và chúng ta cần trích xuất câu trả lời cuối cùng. Các phương pháp trích xuất phổ biến gồm: tìm số nằm sau "Final Answer:", tìm số nằm sau ký hiệu "####", dùng biểu thức chính quy (regular expression) để trích xuất số cuối cùng. Việc so sánh câu trả lời cần xử lý độ chính xác số học (chẳng hạn 72.0 và 72 nên được coi là như nhau), chuyển đổi đơn vị (chẳng hạn 1000 và 1k), và khác biệt định dạng (chẳng hạn "72" và "seventy-two").

Ví dụ sử dụng:

```python
from hello_agents.tools import RLTrainingTool
import json
rl_tool = RLTrainingTool()

# Create accuracy reward function
reward_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "accuracy"
})
reward_data = json.loads(reward_result)

print(f"Reward type: {reward_data['reward_type']}")
print(f"Description: {reward_data['description']}")

# Note: The create_reward operation of RLTrainingTool returns configuration information,
# the actual reward function will be automatically created and used during training
```

Đầu ra:

```json
Prediction: 72, Ground truth: 72, Reward: 1.0
Prediction: 72.0, Ground truth: 72, Reward: 1.0
Prediction: 73, Ground truth: 72, Reward: 0.0
```

Ưu điểm của phần thưởng độ chính xác: đơn giản và trực tiếp, dễ hiểu và dễ triển khai, phù hợp với các nhiệm vụ có câu trả lời đúng rõ ràng. Nhược điểm: phần thưởng thưa, chỉ những câu trả lời hoàn toàn đúng mới nhận được thưởng, không thể phân biệt "gần đúng" với "hoàn toàn sai", có thể dẫn tới thiếu phản hồi hiệu quả trong giai đoạn đầu huấn luyện.

**(2) Length Penalty (Phạt độ dài)**

Length Penalty (LengthPenaltyReward) khuyến khích mô hình sinh ra câu trả lời súc tích, tránh dài dòng. Định nghĩa toán học:

$$
r_{\text{length}}(a, a^*, l) = r_{\text{acc}}(a, a^*) - \alpha \cdot \max(0, l - l_{\text{target}})
$$

Trong đó $l$ là độ dài của văn bản sinh ra (số ký tự hoặc số token), $l_{\text{target}}$ là độ dài mục tiêu, và $\alpha$ là hệ số phạt (mặc định 0.001). Phạt độ dài chỉ được áp dụng khi câu trả lời đúng, tránh trường hợp mô hình sinh ra câu trả lời sai nhưng ngắn để giảm hình phạt.

Lý do thiết kế: nếu câu trả lời sai, phần thưởng là 0 (bất kể độ dài); nếu câu trả lời đúng và độ dài hợp lý, phần thưởng là 1; nếu câu trả lời đúng nhưng quá dài, phần thưởng là $1 - \alpha \cdot (l - l_{\text{target}})$. Ví dụ, độ dài mục tiêu 200 ký tự, độ dài thực tế 500 ký tự, hệ số phạt 0.001, thì phần thưởng là $1 - 0.001 \times (500 - 200) = 0.7$.

Ví dụ sử dụng:

```python
# Create length penalty reward function
reward_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "length_penalty",
    "max_length": 1024,      # Maximum length
    "penalty_weight": 0.001  # Penalty weight
})
reward_data = json.loads(reward_result)

print(f"Reward type: {reward_data['reward_type']}")
print(f"Description: {reward_data['description']}")
print(f"Max length: {reward_data['max_length']}")
print(f"Penalty weight: {reward_data['penalty_weight']}")
```

Đầu ra:

```
Prediction: 72, Ground truth: 72, Length: 50, Reward: 1.000
Prediction: 72, Ground truth: 72, Length: 200, Reward: 1.000
Prediction: 72, Ground truth: 72, Length: 500, Reward: 0.700
Prediction: 73, Ground truth: 72, Length: 50, Reward: 0.000
```

Ưu điểm của phạt độ dài: khuyến khích biểu đạt súc tích, tránh mô hình sinh ra nội dung dư thừa, có thể kiểm soát chi phí suy luận (đầu ra ngắn hơn nghĩa là tiêu thụ ít token hơn). Nhược điểm: có thể ức chế suy luận chi tiết, cần điều chỉnh cẩn thận hệ số phạt, độ dài tối ưu khác nhau rất nhiều giữa các nhiệm vụ.

**(3) Step Reward (Phần thưởng bước)**

Step Reward (StepReward) khuyến khích mô hình sinh ra các bước suy luận rõ ràng, cải thiện tính diễn giải được (interpretability). Định nghĩa toán học:

$$
r_{\text{step}}(a, a^*, s) = r_{\text{acc}}(a, a^*) + \beta \cdot s
$$

Trong đó $s$ là số bước suy luận được phát hiện và $\beta$ là hệ số phần thưởng bước (mặc định 0.1). Tương tự, phần thưởng bước chỉ được cho khi câu trả lời đúng.

Các phương pháp phát hiện bước gồm: tìm các ký hiệu "Step 1:", "Step 2:", đếm ký tự xuống dòng, dùng biểu thức chính quy để khớp các mẫu suy luận. Ví dụ, một câu trả lời đúng với 3 bước rõ ràng nhận phần thưởng $1 + 0.1 \times 3 = 1.3$.

Ví dụ sử dụng:

```python
# Create step reward function
reward_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "step",
    "step_bonus": 0.1  # 0.1 reward per step
})
reward_data = json.loads(reward_result)

print(f"Reward type: {reward_data['reward_type']}")
print(f"Description: {reward_data['description']}")
print(f"Step bonus: {reward_data['step_bonus']}")
```

Đầu ra:

```
Prediction: 72, Ground truth: 72, Steps: 0, Reward: 1.00
Prediction: 72, Ground truth: 72, Steps: 2, Reward: 1.20
Prediction: 72, Ground truth: 72, Steps: 5, Reward: 1.50
Prediction: 73, Ground truth: 72, Steps: 5, Reward: 0.00
```

Ưu điểm của phần thưởng bước: khuyến khích suy luận diễn giải được, câu trả lời sinh ra dễ kiểm chứng và gỡ lỗi hơn, giúp mô hình học tư duy có hệ thống. Nhược điểm: có thể khiến mô hình sinh ra các bước dư thừa để nhận nhiều phần thưởng hơn, cần cân bằng số lượng bước với chất lượng câu trả lời, việc phát hiện bước có thể không chính xác.

Trong các ứng dụng thực tế, chúng ta thường kết hợp nhiều hàm phần thưởng để cân bằng các mục tiêu khác nhau. Các chiến lược kết hợp phổ biến gồm:

**Accuracy + Length Penalty**: Khuyến khích câu trả lời đúng và súc tích, phù hợp với hệ thống hội thoại và hệ thống hỏi-đáp. Công thức:

$$
r = r_{\text{acc}} - \alpha \cdot \max(0, l - l_{\text{target}})
$$

**Accuracy + Step Reward**: Khuyến khích quá trình suy luận chi tiết, phù hợp với các kịch bản giáo dục và AI có thể giải thích được. Công thức:

$$
r = r_{\text{acc}} + \beta \cdot s
$$

**Cân bằng ba chiều**: Tối ưu tổng hợp chất lượng câu trả lời, tính súc tích, và tính diễn giải được. Công thức:

$$
r = r_{\text{acc}} - \alpha \cdot \max(0, l - l_{\text{target}}) + \beta \cdot s
$$

Các trọng số $\alpha$ và $\beta$ cần được điều chỉnh cẩn thận để tránh một mục tiêu chi phối quá mức.

Ví dụ sử dụng:

```python
# Combined reward function: accuracy + length penalty + step reward
# Note: RLTrainingTool currently supports single reward type
# Combined rewards need to be specified through reward_fn parameter in training configuration
# This shows how to configure different types of reward functions

# Accuracy reward
accuracy_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "accuracy"
})
print("Accuracy reward:", json.loads(accuracy_result)['description'])

# Length penalty reward
length_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "length_penalty",
    "max_length": 1024,
    "penalty_weight": 0.001
})
print("Length penalty reward:", json.loads(length_result)['description'])

# Step reward
step_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "step",
    "step_bonus": 0.1
})
print("Step reward:", json.loads(step_result)['description'])
```

Đầu ra:

```
Combined reward: 1.200
  - Accuracy: 1.0
  - Length penalty: -0.100
  - Step reward: +0.3
```

Như minh họa ở Bảng 11.4, các hàm phần thưởng khác nhau phù hợp với các kịch bản ứng dụng khác nhau.

<div align="center">
  <p>Bảng 11.4 So sánh các hàm phần thưởng</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-4.png" alt="" width="85%"/>
</div>

### 11.2.3 Tập dữ liệu và hàm phần thưởng tùy chỉnh

Mặc dù HelloAgents cung cấp tập dữ liệu GSM8K và các hàm phần thưởng thông dụng, nhưng trong các ứng dụng thực tế bạn có thể cần dùng tập dữ liệu của riêng mình hoặc thiết kế các hàm phần thưởng cụ thể. Mục này sẽ giới thiệu cách mở rộng framework.

Trước khi dùng tập dữ liệu tùy chỉnh, bạn cần hiểu các yêu cầu dữ liệu cho hai định dạng huấn luyện:

**Định dạng SFT**: Dùng cho supervised fine-tuning, cần chứa các trường sau:
- `prompt`: Prompt đầu vào (chứa các thông điệp system và user)
- `completion`: Đầu ra mong đợi
- `text`: Toàn bộ văn bản hội thoại (tùy chọn)

**Định dạng RL**: Dùng cho reinforcement learning, cần chứa các trường sau:
- `question`: Câu hỏi gốc
- `prompt`: Prompt đầu vào (chứa các thông điệp system và user)
- `ground_truth`: Câu trả lời đúng
- `full_answer`: Câu trả lời hoàn chỉnh (bao gồm quá trình suy luận)

**(1) Chuyển đổi với format_math_dataset**

Phương pháp đơn giản nhất là chuẩn bị dữ liệu thô chứa các trường `question` và `answer`, rồi dùng hàm `format_math_dataset()` để chuyển đổi tự động:

```python
from datasets import Dataset
from hello_agents.rl import format_math_dataset

# 1. Prepare raw data
custom_data = [
    {
        "question": "What is 2+2?",
        "answer": "2+2=4. #### 4"
    },
    {
        "question": "What is 5*3?",
        "answer": "5*3=15. #### 15"
    },
    {
        "question": "What is 10+7?",
        "answer": "10+7=17. #### 17"
    }
]

# 2. Convert to Dataset object
raw_dataset = Dataset.from_list(custom_data)

# 3. Convert to SFT format
sft_dataset = format_math_dataset(
    dataset=raw_dataset,
    format_type="sft",
    model_name="Qwen/Qwen3-0.6B"
)
print(f"SFT dataset: {len(sft_dataset)} samples")
print(f"Fields: {sft_dataset.column_names}")

# 4. Convert to RL format
rl_dataset = format_math_dataset(
    dataset=raw_dataset,
    format_type="rl",
    model_name="Qwen/Qwen3-0.6B"
)
print(f"RL dataset: {len(rl_dataset)} samples")
print(f"Fields: {rl_dataset.column_names}")
```

**(2) Truyền trực tiếp tập dữ liệu tùy chỉnh**

Khi dùng RLTrainingTool, bạn có thể truyền trực tiếp một tập dữ liệu tùy chỉnh qua tham số `custom_dataset`:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# SFT training
result = rl_tool.run({
    "action": "train",
    "algorithm": "sft",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/custom_sft",
    "num_epochs": 3,
    "batch_size": 4,
    "use_lora": True,
    "custom_dataset": sft_dataset  # Directly pass custom dataset
})

# GRPO training
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/custom_grpo",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True,
    "custom_dataset": rl_dataset  # Directly pass custom dataset
})
```

**(3) Đăng ký tập dữ liệu tùy chỉnh (Khuyến nghị)**

Đối với các tập dữ liệu cần dùng nhiều lần, nên đăng ký:

```python
# 1. Register dataset
rl_tool.register_dataset("my_math_dataset", rl_dataset)

# 2. Use registered dataset
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "dataset": "my_math_dataset",  # Use registered dataset name
    "output_dir": "./models/custom_grpo",
    "num_epochs": 2,
    "use_lora": True
})
```

Các hàm phần thưởng được dùng để đánh giá chất lượng của câu trả lời do mô hình sinh ra. Hàm phần thưởng tùy chỉnh cần tuân theo chữ ký (signature) sau:

```python
from typing import List
import re

def custom_reward_function(
    completions: List[str],
    **kwargs
) -> List[float]:
    """
    Custom reward function

    Args:
        completions: List of completion texts generated by the model
        **kwargs: Other parameters, typically including:
            - ground_truth: List of correct answers
            - Other dataset fields

    Returns:
        List of reward values (each value between 0.0-1.0)
    """
    ground_truths = kwargs.get("ground_truth", [])
    rewards = []

    for completion, truth in zip(completions, ground_truths):
        reward = 0.0

        # Extract answer
        numbers = re.findall(r'-?\d+\.?\d*', completion)
        if numbers:
            try:
                pred = float(numbers[-1])
                truth_num = float(truth)
                error = abs(pred - truth_num)

                # Give different rewards based on error
                if error < 0.01:
                    reward = 1.0  # Completely correct
                elif error < 1.0:
                    reward = 0.8  # Very close
                elif error < 5.0:
                    reward = 0.5  # Close

                # Extra reward: encourage showing reasoning steps
                if "step" in completion.lower() or "=" in completion:
                    reward += 0.1

            except ValueError:
                reward = 0.0

        rewards.append(min(reward, 1.0))  # Limit maximum value to 1.0

    return rewards
```

Có hai cách để dùng hàm phần thưởng tùy chỉnh:

**(1) Truyền trực tiếp**

```python
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/custom_grpo",
    "custom_dataset": rl_dataset,
    "custom_reward": custom_reward_function  # Directly pass reward function
})
```

**(2) Đăng ký (Khuyến nghị)**

```python
# 1. Register reward function
rl_tool.register_reward_function("my_reward", custom_reward_function)

# 2. Use registered reward function
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "dataset": "my_math_dataset",
    "output_dir": "./models/custom_grpo"
    # Reward function will automatically use registered function with same name as dataset
})
```

Dưới đây là một ví dụ hoàn chỉnh về tập dữ liệu và hàm phần thưởng tùy chỉnh:

```python
from datasets import Dataset
from hello_agents.tools import RLTrainingTool
from hello_agents.rl import format_math_dataset
import re
from typing import List

# 1. Prepare custom data
custom_data = [
    {"question": "What is 2+2?", "answer": "2+2=4. #### 4"},
    {"question": "What is 5+3?", "answer": "5+3=8. #### 8"},
    {"question": "What is 10+7?", "answer": "10+7=17. #### 17"}
]

# 2. Convert to training format
raw_dataset = Dataset.from_list(custom_data)
rl_dataset = format_math_dataset(raw_dataset, format_type="rl")

# 3. Define custom reward function
def tolerant_reward(completions: List[str], **kwargs) -> List[float]:
    """Reward function with tolerance"""
    ground_truths = kwargs.get("ground_truth", [])
    rewards = []

    for completion, truth in zip(completions, ground_truths):
        numbers = re.findall(r'-?\d+\.?\d*', completion)
        if numbers:
            try:
                pred = float(numbers[-1])
                truth_num = float(truth)
                error = abs(pred - truth_num)

                if error < 0.01:
                    reward = 1.0
                elif error < 5.0:
                    reward = 0.5
                else:
                    reward = 0.0
            except ValueError:
                reward = 0.0
        else:
            reward = 0.0

        rewards.append(reward)

    return rewards

# 4. Create tool and register
rl_tool = RLTrainingTool()
rl_tool.register_dataset("my_dataset", rl_dataset)
rl_tool.register_reward_function("my_dataset", tolerant_reward)

# 5. Train
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "dataset": "my_dataset",
    "output_dir": "./models/custom_grpo",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True
})
```

## 11.3 Huấn luyện SFT

Supervised Fine-Tuning (SFT) là bước đầu tiên của việc huấn luyện reinforcement learning và là nền tảng quan trọng nhất. SFT giúp mô hình học được định dạng cơ bản của nhiệm vụ, các mẫu hội thoại, và năng lực suy luận sơ khởi. Nếu không có nền tảng của SFT, việc thực hiện reinforcement learning trực tiếp thường thất bại vì mô hình thậm chí không biết cả định dạng đầu ra cơ bản.

### 11.3.1 Tại sao cần SFT

Trước khi bắt đầu reinforcement learning, chúng ta cần thực hiện huấn luyện SFT trước. Điều này là vì mặc dù các mô hình pretraining có năng lực ngôn ngữ mạnh mẽ, nhưng chúng không biết cách hoàn thành các nhiệm vụ cụ thể. Mục tiêu huấn luyện của các mô hình pretraining là dự đoán từ tiếp theo, chứ không phải giải toán hay dùng công cụ. Định dạng đầu ra của các mô hình pretraining là văn bản tự do, trong khi chúng ta cần đầu ra có cấu trúc (chẳng hạn "Step 1: ..., Step 2: ..., Final Answer: ..."). Các mô hình pretraining chưa thấy dữ liệu liên quan đến nhiệm vụ và không biết một quá trình suy luận "tốt" là như thế nào.

Vai trò của SFT là dạy cho mô hình các quy tắc cơ bản của nhiệm vụ. Thứ nhất, học định dạng đầu ra, giúp mô hình biết cách tổ chức câu trả lời (chẳng hạn dùng các ký hiệu "Step 1", "Final Answer"). Thứ hai, học các mẫu suy luận, học cách phân rã bài toán và suy diễn từng bước thông qua các ví dụ. Thứ ba, thiết lập năng lực cơ sở, cung cấp một điểm khởi đầu hợp lý cho reinforcement learning về sau. Cuối cùng, giảm không gian khám phá, reinforcement learning không cần bắt đầu từ con số không mà có thể tối ưu dựa trên SFT.

Hãy hiểu tầm quan trọng của SFT qua một thí nghiệm so sánh. Giả sử chúng ta dùng trực tiếp một mô hình pretraining để giải các bài toán GSM8K:

```python
from transformers import AutoTokenizer, AutoModelForCausalLM

# Load pretrained model
model_name = "Qwen/Qwen3-0.6B"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name)

# Test question
question = """Natalia sold clips to 48 of her friends in April, and then she sold half as many clips in May. How many clips did Natalia sell altogether in April and May?"""

# Construct input
prompt = f"<|im_start|>user\n{question}<|im_end|>\n<|im_start|>assistant\n"
inputs = tokenizer(prompt, return_tensors="pt")

# Generate answer
outputs = model.generate(**inputs, max_new_tokens=200)
response = tokenizer.decode(outputs[0], skip_special_tokens=False)

print("Pretrained model's answer:")
print(response)
```

Đầu ra của mô hình pretraining có thể là:

```bash
<|im_start|>user
Natalia sold clips to 48 of her friends in April, and then she sold half as many clips in May. How many clips did Natalia sell altogether in April and May?<|im_end|>
<|im_start|>assistant
I can help you with that. Let me think about this problem. Natalia sold clips to her friends. In April she sold 48 clips. In May she sold half as many. So we need to find out how many she sold in May and then add them together. But I'm not sure how to calculate half of 48. Maybe I should use a calculator? Or maybe I can just estimate...
```

Như có thể thấy, mặc dù mô hình pretraining hiểu câu hỏi, nhưng đầu ra rất dài dòng, thiếu cấu trúc, không có câu trả lời rõ ràng, và có quá trình suy luận lộn xộn. Đầu ra như vậy không thể dùng cho reinforcement learning vì chúng ta không thể trích xuất câu trả lời, không thể đánh giá chất lượng, và không thể cung cấp tín hiệu phần thưởng hiệu quả.

Bây giờ hãy xem mô hình sau khi SFT:

```python
# Load SFT model (assuming already trained)
sft_model_path = "./models/sft_model"
sft_model = AutoModelForCausalLM.from_pretrained(sft_model_path)

# Use the same question
outputs = sft_model.generate(**inputs, max_new_tokens=200)
response = tokenizer.decode(outputs[0], skip_special_tokens=False)

print("SFT model's answer:")
print(response)
```

Đầu ra của mô hình SFT:

```bash
<|im_start|>user
Natalia sold clips to 48 of her friends in April, and then she sold half as many clips in May. How many clips did Natalia sell altogether in April and May?<|im_end|>
<|im_start|>assistant
Let me solve this step by step.

Step 1: Calculate clips sold in May
Natalia sold half as many clips in May as in April.
Clips in May = 48 / 2 = 24

Step 2: Calculate total clips
Total = April + May = 48 + 24 = 72

Final Answer: 72<|im_end|>
```

Như có thể thấy, đầu ra của mô hình SFT có cấu trúc rõ ràng (dùng các ký hiệu "Step 1", "Step 2", "Final Answer"), suy luận đúng, câu trả lời rõ ràng, và định dạng thống nhất. Đầu ra như vậy có thể dùng cho reinforcement learning vì chúng ta có thể trích xuất câu trả lời, tính phần thưởng, và tối ưu chiến lược.

Như minh họa ở Hình 11.6, SFT là cây cầu nối từ các mô hình pretraining đến reinforcement learning.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-6.png" alt="" width="85%"/>
  <p>Hình 11.6 Vai trò của SFT trong pipeline huấn luyện</p>
</div>

### 11.3.2 LoRA: Tinh chỉnh hiệu quả về tham số

Việc tinh chỉnh (fine-tuning) trực tiếp toàn bộ mô hình đòi hỏi tài nguyên tính toán và bộ nhớ đáng kể. Đối với Qwen3-0.6B (0.6B tham số), tinh chỉnh toàn phần (full fine-tuning) đòi hỏi khoảng 12GB bộ nhớ (FP16) hoặc 24GB bộ nhớ (FP32). Đối với các mô hình lớn hơn (chẳng hạn 7B, 13B), tinh chỉnh toàn phần gần như bất khả thi trên các GPU cấp tiêu dùng.

LoRA (Low-Rank Adaptation)<sup>[3]</sup> là một phương pháp tinh chỉnh hiệu quả về tham số, chỉ huấn luyện một số ít tham số bổ sung trong khi giữ nguyên (đóng băng) các tham số của mô hình gốc. Ý tưởng cốt lõi của LoRA là: các thay đổi tham số trong quá trình tinh chỉnh mô hình có thể được biểu diễn bằng các ma trận hạng thấp (low-rank matrices).

Giả sử ma trận trọng số của mô hình gốc là $W \in \mathbb{R}^{d \times k}$, và trọng số sau khi tinh chỉnh là $W' = W + \Delta W$. LoRA giả định $\Delta W$ có thể được phân rã thành tích của hai ma trận hạng thấp:

$$
\Delta W = BA
$$

Trong đó $B \in \mathbb{R}^{d \times r}$, $A \in \mathbb{R}^{r \times k}$, $r \ll \min(d, k)$ là hạng (rank).

Trong quá trình lan truyền tiến (forward propagation), đầu ra là:

$$
h = Wx + \Delta Wx = Wx + BAx
$$

Các tham số $W$ của mô hình gốc vẫn được đóng băng, chỉ huấn luyện $B$ và $A$.

So sánh số lượng tham số: số lượng tham số của mô hình gốc là $d \times k$, số lượng tham số LoRA là $d \times r + r \times k = r(d + k)$. Khi $r \ll \min(d, k)$, số lượng tham số LoRA nhỏ hơn nhiều so với mô hình gốc. Ví dụ, với $d=4096, k=4096, r=8$, số lượng tham số của mô hình gốc là $4096 \times 4096 = 16,777,216$, số lượng tham số LoRA là $8 \times (4096 + 4096) = 65,536$, giảm tham số 256 lần!

Vì vậy, chúng ta có thể tổng kết các ưu điểm của LoRA: giảm đáng kể mức sử dụng bộ nhớ, tốc độ huấn luyện nhanh hơn, dễ triển khai, và ngăn ngừa overfitting. Tuy nhiên, hiệu quả huấn luyện thường kém hơn đôi chút so với tinh chỉnh toàn bộ tham số.

Như minh họa ở Bảng 11.5, so sánh hiệu quả của LoRA ở các quy mô mô hình khác nhau.

<div align="center">
  <p>Bảng 11.5 So sánh LoRA và tinh chỉnh toàn phần</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-5.png" alt="" width="85%"/>
</div>

Các siêu tham số (hyperparameter) then chốt của LoRA gồm: rank (r), kiểm soát hạng của các ma trận LoRA, càng lớn thì khả năng biểu đạt càng mạnh nhưng càng nhiều tham số, giá trị điển hình 4-64, mặc định 8; Alpha ($\alpha$), hệ số co giãn LoRA, cập nhật thực tế là $\Delta W = \frac{\alpha}{r} BA$, kiểm soát cường độ ảnh hưởng của LoRA, giá trị điển hình bằng rank; target_modules, chỉ định áp dụng LoRA cho các lớp nào, thường chọn các lớp attention (q_proj, k_proj, v_proj, o_proj), cũng có thể bao gồm các lớp MLP (gate_proj, up_proj, down_proj).

### 11.3.3 Thực hành huấn luyện SFT

Bây giờ hãy thực hiện huấn luyện SFT bằng HelloAgents. Quy trình huấn luyện hoàn chỉnh gồm: chuẩn bị tập dữ liệu, cấu hình LoRA, thiết lập các tham số huấn luyện, bắt đầu huấn luyện, và lưu mô hình.

Ví dụ huấn luyện cơ bản:

```python
from hello_agents.tools import RLTrainingTool

# Create training tool
rl_tool = RLTrainingTool()

# SFT training
result = rl_tool.run({
    # Training configuration
    "action": "train",
    "algorithm": "sft",

    # Model configuration
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/sft_model",

    # Data configuration
    "max_samples": 100,     # Use 100 samples for quick test

    # Training parameters
    "num_epochs": 3,        # Train for 3 epochs
    "batch_size": 4,        # Batch size
    "learning_rate": 5e-5,  # Learning rate

    # LoRA configuration
    "use_lora": True,       # Use LoRA
    "lora_rank": 8,         # LoRA rank
    "lora_alpha": 16,       # LoRA alpha
})

print(f"\n✓ Training completed!")
print(f"  - Model save path: {result['model_path']}")
print(f"  - Training samples: {result['num_samples']}")
print(f"  - Training epochs: {result['num_epochs']}")
print(f"  - Final loss: {result['final_loss']:.4f}")
```

Nếu loss dần giảm trong quá trình huấn luyện, điều đó cho thấy mô hình đang học.

**(1) Chi tiết các tham số huấn luyện**

Hãy hiểu ý nghĩa và các gợi ý điều chỉnh cho từng tham số huấn luyện một cách chi tiết.

**Tham số dữ liệu**:

- `max_samples`: Số lượng mẫu huấn luyện được dùng. Để kiểm thử nhanh, dùng 100-1000 mẫu; để huấn luyện đầy đủ, khuyến nghị dùng toàn bộ dữ liệu (7473 mẫu). Nhiều dữ liệu hơn thường mang lại kết quả tốt hơn, nhưng thời gian huấn luyện cũng dài hơn.
- `split`: Tập con của tập dữ liệu, mặc định "train". Có thể đặt thành "train[:1000]" để chỉ dùng 1000 mẫu đầu tiên.

**Tham số huấn luyện**:

- `num_epochs`: Số epoch huấn luyện. 1 epoch nghĩa là duyệt qua toàn bộ tập dữ liệu một lần. Quá ít (1-2 epoch) có thể underfit, quá nhiều (>10 epoch) có thể overfit. Khuyến nghị bắt đầu từ 3 epoch, quan sát đường cong loss và điều chỉnh.
- `batch_size`: Số lượng mẫu dùng cho mỗi lần cập nhật. Càng lớn càng ổn định nhưng dùng nhiều bộ nhớ hơn. Khuyến nghị điều chỉnh dựa trên bộ nhớ: 4GB bộ nhớ dùng batch_size=1-2, 8GB bộ nhớ dùng batch_size=4-8, 16GB bộ nhớ dùng batch_size=8-16.
- `learning_rate`: Learning rate, kiểm soát độ lớn bước cập nhật tham số. Quá nhỏ (1e-6) hội tụ chậm, quá lớn (1e-3) có thể không hội tụ. SFT khuyến nghị 5e-5, LoRA có thể lớn hơn đôi chút (1e-4).

**Tham số LoRA**:

- `use_lora`: Có dùng LoRA hay không. Khuyến nghị luôn bật trừ khi có đủ bộ nhớ.
- `lora_rank`: Hạng LoRA, kiểm soát khả năng biểu đạt. 4-8 phù hợp với nhiệm vụ nhỏ, 16-32 phù hợp với nhiệm vụ phức tạp, 64 phù hợp với tinh chỉnh quy mô lớn.
- `lora_alpha`: Hệ số co giãn LoRA, thường đặt bằng 2 lần rank. Khi rank=8, alpha=16; khi rank=16, alpha=32.

**Tham số optimizer**:

- `optimizer`: Loại optimizer, mặc định "adamw". AdamW là lựa chọn phổ biến nhất, cũng có thể thử "sgd" hoặc "adafactor".
- `weight_decay`: Suy giảm trọng số (weight decay), ngăn ngừa overfitting. Mặc định 0.01, có thể thử 0.001-0.1.
- `warmup_ratio`: Tỷ lệ warmup của learning rate. Learning rate tăng tuyến tính trong warmup_ratio bước đầu tiên, rồi suy giảm tuyến tính. Mặc định 0.1 (warmup 10% bước đầu tiên).

**(2) Ví dụ huấn luyện hoàn chỉnh**

Hãy thực hiện một lần huấn luyện SFT hoàn chỉnh dùng toàn bộ dữ liệu và các thực hành tốt nhất:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# Complete SFT training
result = rl_tool.run({
    "action": "train",
    "algorithm": "sft",

    # Model configuration
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/sft_full",

    # Data configuration
    "max_samples": None,    # Use all data (7473 samples)

    # Training parameters
    "num_epochs": 3,
    "batch_size": 8,
    "learning_rate": 5e-5,
    "warmup_ratio": 0.1,
    "weight_decay": 0.01,

    # LoRA configuration
    "use_lora": True,
    "lora_rank": 16,        # Use larger rank
    "lora_alpha": 32,
    "lora_target_modules": ["q_proj", "k_proj", "v_proj", "o_proj"],

    # Other configurations
    "save_steps": 500,      # Save every 500 steps
    "logging_steps": 100,   # Log every 100 steps
    "eval_steps": 500,      # Evaluate every 500 steps
})

print(f"Training completed! Model saved at: {result['model_path']}")
```

Cấu hình này phù hợp để huấn luyện trên GPU có 8GB bộ nhớ, ước tính mất 30-60 phút.

**(3) Giám sát và gỡ lỗi huấn luyện**

Trong quá trình huấn luyện, chúng ta cần giám sát ba chỉ số then chốt. Loss nên dần giảm; nếu không giảm, learning rate có thể quá nhỏ hoặc dữ liệu có thể có vấn đề; nếu giảm rồi tăng, learning rate có thể quá lớn hoặc có thể xảy ra overfitting. Gradient Norm nên nằm trong khoảng hợp lý 0.1-10; quá lớn (>100) cho thấy gradient bùng nổ (gradient explosion) và cần giảm learning rate; quá nhỏ (<0.01) cho thấy gradient biến mất (gradient vanishing) và cần kiểm tra cấu hình mô hình. Learning Rate nên thay đổi theo chiến lược warmup, tăng tuyến tính trong 10% bước đầu tiên, rồi suy giảm tuyến tính về 0.

Các vấn đề thường gặp trong quá trình huấn luyện và cách giải quyết: khi hết bộ nhớ (out of memory), giảm batch_size hoặc max_length, dùng gradient accumulation hoặc mô hình nhỏ hơn; khi huấn luyện chậm, tăng batch_size, giảm tần suất ghi log, hoặc dùng huấn luyện độ chính xác hỗn hợp (mixed precision); khi loss không giảm, tăng learning rate, kiểm tra định dạng dữ liệu, hoặc tăng số epoch huấn luyện; khi overfitting, tăng weight_decay, giảm số epoch huấn luyện, hoặc dùng nhiều dữ liệu hơn.

### 11.3.4 Đánh giá mô hình

Sau khi huấn luyện xong, chúng ta cần đánh giá hiệu quả của mô hình. Các chỉ số đánh giá gồm:

- **Accuracy (Độ chính xác)**: Tỷ lệ câu trả lời hoàn toàn đúng, chỉ số trực tiếp nhất, phạm vi 0-1, càng cao càng tốt.

- **Average Reward (Phần thưởng trung bình)**: Phần thưởng trung bình trên tất cả các mẫu, xem xét tổng hợp độ chính xác, độ dài, số bước và các yếu tố khác, phạm vi phụ thuộc vào thiết kế hàm phần thưởng.

- **Reasoning Quality (Chất lượng suy luận)**: Độ rõ ràng và tính logic của quá trình suy luận, đòi hỏi đánh giá thủ công hoặc mô hình đánh giá chuyên biệt.

Dùng HelloAgents để đánh giá mô hình:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# Evaluate SFT model
eval_result = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/sft_full",
    "max_samples": 100,     # Evaluate on 100 test samples
    "use_lora": True,
})

eval_data = json.loads(eval_result)
print(f"\nEvaluation results:")
print(f"  - Accuracy: {eval_data['accuracy']}")
print(f"  - Average reward: {eval_data['average_reward']}")
print(f"  - Test samples: {eval_data['num_samples']}")
```

Đối với các mô hình nhỏ như Qwen3-0.6B, việc đạt độ chính xác 40-50% trên GSM8K sau SFT là bình thường. Thông qua reinforcement learning, chúng ta có thể cải thiện thêm lên 60-70%.

Để hiểu rõ hơn hiệu quả của SFT, chúng ta có thể so sánh các mô hình ở các giai đoạn khác nhau:

```python
# Evaluate pretrained model (without SFT)
base_result = rl_tool.run({
    "action": "evaluate",
    "model_path": "Qwen/Qwen3-0.6B",
    "max_samples": 100,
    "use_lora": False,
})
base_data = json.loads(base_result)

# Evaluate SFT model
sft_result = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/sft_full",
    "max_samples": 100,
    "use_lora": True,
})
sft_data = json.loads(sft_result)

# Compare results
print("Model comparison:")
print(f"Pretrained model accuracy: {base_data['accuracy']}")
print(f"SFT model accuracy: {sft_data['accuracy']}")
```

Trong mục này, chúng ta đã học về tầm quan trọng của SFT (học định dạng, thiết lập năng lực cơ sở), nguyên lý của LoRA (phân rã hạng thấp, hiệu quả về tham số), thực hành huấn luyện SFT (cấu hình tham số, giám sát huấn luyện), và đánh giá mô hình (độ chính xác, phân tích so sánh).

## 11.4 Huấn luyện GRPO

Sau khi hoàn thành huấn luyện SFT, chúng ta đã có được một mô hình có khả năng sinh ra các câu trả lời có cấu trúc. Tuy nhiên, mô hình SFT mới chỉ học cách "bắt chước" quá trình suy luận trong dữ liệu huấn luyện chứ chưa thực sự học cách "tư duy". Reinforcement learning có thể cho phép mô hình tối ưu chiến lược suy luận thông qua thử-sai, từ đó vượt qua chất lượng của dữ liệu huấn luyện.

### 11.4.1 Từ PPO đến GRPO

Trong lĩnh vực reinforcement learning, PPO (Proximal Policy Optimization)<sup>[1]</sup> là một trong những thuật toán kinh điển nhất. PPO đảm bảo tính ổn định trong huấn luyện bằng cách giới hạn độ lớn của việc cập nhật policy. Tuy nhiên, PPO có một số vấn đề trong huấn luyện LLM: nó đòi hỏi huấn luyện một Value Model (mô hình giá trị), làm tăng độ phức tạp huấn luyện và mức sử dụng bộ nhớ; nó đòi hỏi duy trì đồng thời bốn mô hình (Policy Model, Reference Model, Value Model, Reward Model), khiến việc triển khai kỹ thuật phức tạp; huấn luyện không ổn định, dễ xảy ra reward collapse (sụp đổ phần thưởng) hoặc policy degradation (thoái hóa policy).

GRPO (Group Relative Policy Optimization)<sup>[2]</sup> là một biến thể PPO được đơn giản hóa, thiết kế riêng cho LLM. Ý tưởng cốt lõi của GRPO là: không cần Value Model, dùng phần thưởng tương đối theo nhóm (group-relative reward) thay cho phần thưởng tuyệt đối; đơn giản hóa quy trình huấn luyện, chỉ cần Policy Model và Reference Model; cải thiện tính ổn định huấn luyện, giảm nguy cơ reward collapse.

Hãy hiểu nguyên lý của GRPO qua các công thức toán học. Hàm mục tiêu của PPO là:

$$
J_{\text{PPO}}(\theta) = \mathbb{E}_{s,a \sim \pi_\theta} \left[ \min\left( \frac{\pi_\theta(a|s)}{\pi_{\text{old}}(a|s)} A(s,a), \text{clip}\left(\frac{\pi_\theta(a|s)}{\pi_{\text{old}}(a|s)}, 1-\epsilon, 1+\epsilon\right) A(s,a) \right) \right]
$$

Trong đó $A(s,a)$ là hàm lợi thế (advantage function), đòi hỏi Value Model để ước lượng:

$$
A(s,a) = Q(s,a) - V(s) = r(s,a) + \gamma V(s') - V(s)
$$

Hàm mục tiêu của GRPO được đơn giản hóa thành:

$$
J_{\text{GRPO}}(\theta) = \mathbb{E}_{s,a \sim \pi_\theta} \left[ \frac{\pi_\theta(a|s)}{\pi_{\text{ref}}(a|s)} \cdot (r(s,a) - \bar{r}_{\text{group}}) \right] - \beta \cdot D_{KL}(\pi_\theta || \pi_{\text{ref}})
$$

Trong đó $\bar{r}_{\text{group}}$ là phần thưởng trung bình của nhóm và $\beta$ là hệ số phạt KL divergence. Các khác biệt then chốt là: GRPO dùng $r(s,a) - \bar{r}_{\text{group}}$ thay cho hàm lợi thế $A(s,a)$, không cần Value Model; GRPO dùng phần thưởng tương đối theo nhóm, giảm phương sai phần thưởng; GRPO thêm phạt KL divergence, ngăn policy đi chệch quá xa.

Như minh họa ở Hình 11.7, so sánh quy trình huấn luyện của PPO và GRPO.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-7.png" alt="" width="85%"/>
  <p>Hình 11.7 Quy trình huấn luyện PPO và GRPO</p>
</div>

Như có thể thấy, GRPO loại bỏ việc huấn luyện Value Model, đơn giản hóa quy trình rất nhiều.

Như minh họa ở Bảng 11.6, so sánh chi tiết giữa PPO và GRPO.

<div align="center">
  <p>Bảng 11.6 So sánh PPO và GRPO</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-6.png" alt="" width="85%"/>
</div>



Đối với huấn luyện LLM, GRPO là lựa chọn tốt hơn vì nó đơn giản hơn, ổn định hơn, và có mức sử dụng bộ nhớ thấp hơn.

### 11.4.2 Thực hành huấn luyện GRPO

Bây giờ hãy thực hiện huấn luyện GRPO bằng HelloAgents. Điều kiện tiên quyết để huấn luyện GRPO là hoàn thành huấn luyện SFT, vì GRPO đòi hỏi một policy khởi đầu hợp lý.

Ví dụ huấn luyện GRPO cơ bản:

```python
from hello_agents.tools import RLTrainingTool

# Create training tool
rl_tool = RLTrainingTool()

# GRPO training
result = rl_tool.run({
    # Training configuration
    "action": "train",
    "algorithm": "grpo",

    # Model configuration
    "model_name": "./models/sft_full",  # Start from SFT model
    "output_dir": "./models/grpo_model",

    # Data configuration
    "max_samples": 100,     # Use 100 samples for quick test

    # Training parameters
    "num_epochs": 3,
    "batch_size": 4,
    "learning_rate": 1e-5,  # GRPO learning rate usually smaller than SFT

    # GRPO-specific parameters
    "num_generations": 4,   # Generate 4 answers per question
    "kl_coef": 0.05,        # KL divergence penalty coefficient

    # LoRA configuration
    "use_lora": True,
    "lora_rank": 16,
    "lora_alpha": 32,

    # Reward function configuration
    "reward_type": "accuracy",  # Use accuracy reward
})

print(f"\n✓ Training completed!")
print(f"  - Model save path: {result['model_path']}")
print(f"  - Training samples: {result['num_samples']}")
print(f"  - Training epochs: {result['num_epochs']}")
print(f"  - Average reward: {result['average_reward']:.4f}")
```

Nếu phần thưởng trung bình dần tăng và KL divergence duy trì trong khoảng hợp lý trong quá trình huấn luyện GRPO, điều đó cho thấy huấn luyện đang diễn ra bình thường.

GRPO có một số tham số riêng cần được hiểu và điều chỉnh.

**Tham số sinh (Generation Parameters)**:

- `num_generations`: Sinh bao nhiêu câu trả lời cho mỗi câu hỏi. Càng nhiều càng tốt, nhưng chi phí tính toán cũng cao hơn. Giá trị điển hình là 4-8. Mục đích của việc sinh nhiều câu trả lời là để tính phần thưởng tương đối theo nhóm và tăng tính đa dạng của tín hiệu huấn luyện.
- `max_new_tokens`: Số token tối đa được sinh cho mỗi câu trả lời. Quá ít có thể cắt cụt câu trả lời, quá nhiều lãng phí tính toán. Khuyến nghị 256-512.
- `temperature`: Nhiệt độ sinh, kiểm soát tính ngẫu nhiên. 0 nghĩa là giải mã tham lam (greedy decoding), 1 nghĩa là lấy mẫu chuẩn. GRPO khuyến nghị 0.7-1.0, duy trì một mức độ khám phá nhất định.

**Tham số tối ưu (Optimization Parameters)**:

- `learning_rate`: Learning rate của GRPO thường nhỏ hơn SFT vì chúng ta không muốn đi chệch quá xa so với mô hình SFT. Khuyến nghị 1e-5 đến 5e-5.
- `kl_coef`: Hệ số phạt KL divergence, kiểm soát độ lớn của việc cập nhật policy. Quá nhỏ (0.01) có thể khiến policy đi chệch quá xa, quá lớn (0.5) có thể giới hạn việc học. Khuyến nghị 0.05-0.1.
- `clip_range`: Khoảng cắt tỷ lệ policy, tương tự epsilon của PPO. Khuyến nghị 0.2.

**Tham số phần thưởng (Reward Parameters)**:

- `reward_type`: Loại hàm phần thưởng, có thể là "accuracy", "length_penalty", "step", hoặc "combined".
- `reward_config`: Cấu hình bổ sung cho hàm phần thưởng, chẳng hạn độ dài mục tiêu cho phạt độ dài, hệ số cho phần thưởng bước, v.v.

Hãy thực hiện một lần huấn luyện GRPO hoàn chỉnh dùng toàn bộ dữ liệu và các thực hành tốt nhất:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# Complete GRPO training
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",

    # Model configuration
    "model_name": "./models/sft_full",
    "output_dir": "./models/grpo_full",

    # Data configuration
    "max_samples": None,    # Use all data

    # Training parameters
    "num_epochs": 3,
    "batch_size": 4,
    "learning_rate": 1e-5,
    "warmup_ratio": 0.1,

    # GRPO-specific parameters
    "num_generations": 4,
    "max_new_tokens": 512,
    "temperature": 0.8,
    "kl_coef": 0.05,
    "clip_range": 0.2,

    # LoRA configuration
    "use_lora": True,
    "lora_rank": 16,
    "lora_alpha": 32,

    # Reward function configuration
    "reward_type": "combined",
    "reward_config": {
        "components": [
            {"type": "accuracy", "weight": 1.0},
            {"type": "length_penalty", "weight": 0.5, "target_length": 200},
            {"type": "step", "weight": 0.3, "step_bonus": 0.1}
        ]
    },

    # Other configurations
    "save_steps": 500,
    "logging_steps": 100,
})

print(f"Training completed! Model saved at: {result['model_path']}")
```

### 11.4.3 Phân tích quy trình huấn luyện GRPO

Hãy hiểu sâu về quy trình huấn luyện của GRPO và xem điều gì xảy ra ở mỗi bước.

**(1) Vòng lặp huấn luyện (Training Loop)**

Vòng lặp huấn luyện của GRPO gồm các bước sau:

1. **Sampling Phase (Giai đoạn lấy mẫu)**: Với mỗi câu hỏi, dùng policy hiện tại để sinh nhiều câu trả lời (`num_generations`). Những câu trả lời này tạo thành một "nhóm" (group) để tính phần thưởng tương đối.

2. **Reward Calculation (Tính phần thưởng)**: Tính phần thưởng $r_i$ cho mỗi câu trả lời được sinh ra. Phần thưởng có thể là độ chính xác, phạt độ dài, phần thưởng bước, hoặc kết hợp của chúng.

3. **Relative Reward (Phần thưởng tương đối)**: Tính phần thưởng trung bình của nhóm $\bar{r} = \frac{1}{N}\sum_{i=1}^{N} r_i$, rồi tính phần thưởng tương đối $\hat{r}_i = r_i - \bar{r}$. Lợi ích của điều này là giảm phương sai phần thưởng và làm cho huấn luyện ổn định hơn.

4. **Policy Update (Cập nhật policy)**: Dùng phần thưởng tương đối để cập nhật policy, đồng thời thêm phạt KL divergence để ngăn policy đi chệch quá xa so với mô hình tham chiếu.

5. **Repeat (Lặp lại)**: Lặp lại các bước trên cho đến khi hoàn thành tất cả các epoch huấn luyện.

Hãy hiểu qua một ví dụ cụ thể:

```python
# Assume we have a question
question = "What is 48 + 24?"

# Generate 4 answers
answers = [
    "48 + 24 = 72. Final Answer: 72",      # Correct
    "48 + 24 = 72. Final Answer: 72",      # Correct
    "48 + 24 = 70. Final Answer: 70",      # Incorrect
    "Let me think... 72. Final Answer: 72" # Correct but verbose
]

# Calculate rewards (assuming using accuracy + length penalty)
rewards = [1.0, 1.0, 0.0, 0.8]  # 4th answer penalized for verbosity

# Calculate group average reward
avg_reward = (1.0 + 1.0 + 0.0 + 0.8) / 4 = 0.7

# Calculate relative rewards
relative_rewards = [
    1.0 - 0.7 = 0.3,   # Correct and concise, positive relative reward
    1.0 - 0.7 = 0.3,   # Correct and concise, positive relative reward
    0.0 - 0.7 = -0.7,  # Incorrect, negative relative reward
    0.8 - 0.7 = 0.1    # Correct but verbose, smaller relative reward
]

# Policy update: increase probability of first two answers, decrease probability of third answer
```

Như có thể thấy, cơ chế phần thưởng tương đối khuyến khích mô hình sinh ra những câu trả lời "tốt hơn mức trung bình" thay vì chỉ đơn thuần theo đuổi phần thưởng cao. Điều này có thể giảm phương sai phần thưởng và cải thiện tính ổn định huấn luyện.

**(2) Phạt KL Divergence**

Phạt KL divergence là thành phần then chốt của GRPO, ngăn policy đi chệch quá xa so với mô hình tham chiếu. KL divergence được định nghĩa là:

$$
D_{KL}(\pi_\theta || \pi_{\text{ref}}) = \mathbb{E}_{s,a \sim \pi_\theta} \left[ \log \frac{\pi_\theta(a|s)}{\pi_{\text{ref}}(a|s)} \right]
$$

Trong thực tế, chúng ta tính KL divergence cho mỗi token, rồi cộng lại:

$$
D_{KL} = \sum_{t=1}^{T} \log \frac{\pi_\theta(a_t|s, a_{<t})}{\pi_{\text{ref}}(a_t|s, a_{<t})}
$$

KL divergence càng lớn, sự khác biệt giữa policy hiện tại và mô hình tham chiếu càng lớn. Bằng cách thêm số hạng phạt KL divergence $-\beta \cdot D_{KL}$, chúng ta giới hạn độ lớn của việc cập nhật policy, tránh "quên" kiến thức đã học trong giai đoạn SFT.

Việc chọn `kl_coef` ($\beta$) rất quan trọng:

- Quá nhỏ (0.01): Policy có thể đi chệch quá xa, gây lộn xộn định dạng đầu ra hoặc suy giảm chất lượng
- Quá lớn (0.5): Việc cập nhật policy bị giới hạn, học chậm, khó vượt qua mô hình SFT
- Khuyến nghị (0.05-0.1): Cân bằng giữa khám phá và ổn định

**(3) Giám sát huấn luyện**

Trong quá trình huấn luyện GRPO, chúng ta cần giám sát các chỉ số sau:

- **Average Reward (Phần thưởng trung bình)**: Nên dần tăng. Nếu phần thưởng không tăng, learning rate có thể quá nhỏ, phạt KL quá lớn, hoặc thiết kế hàm phần thưởng không hợp lý. Nếu phần thưởng tăng rồi giảm, có thể là overfitting hoặc reward collapse.

- **KL Divergence**: Nên duy trì trong khoảng hợp lý (0.01-0.1). Nếu KL divergence quá lớn (>0.5), policy đi chệch quá xa, cần tăng kl_coef hoặc giảm learning rate. Nếu KL divergence quá nhỏ (<0.001), policy gần như không cập nhật, cần giảm kl_coef hoặc tăng learning rate.

- **Accuracy (Độ chính xác)**: Nên dần cải thiện. Đây là chỉ số trực quan nhất, phản ánh năng lực thực tế của mô hình.

- **Generation Quality (Chất lượng sinh)**: Cần kiểm tra thủ công các câu trả lời được sinh ra để đảm bảo định dạng đúng và suy luận rõ ràng.

HelloAgents tích hợp hai công cụ giám sát huấn luyện chủ đạo: Weights & Biases (wandb) và TensorBoard.

**Cách 1: Dùng Weights & Biases (Khuyến nghị)**

Weights & Biases hiện là nền tảng theo dõi thí nghiệm machine learning phổ biến nhất, cung cấp các tính năng trực quan hóa và quản lý thí nghiệm mạnh mẽ.

```python
import os

# 1. Set up wandb (need to register account first: https://wandb.ai)
os.environ["WANDB_PROJECT"] = "hello-agents-grpo"  # Project name
os.environ["WANDB_LOG_MODEL"] = "false"            # Don't upload model files

# 2. Enable wandb in training configuration
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/grpo_monitored",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True,
    # wandb will automatically log all training metrics
})

# After training completes, visit https://wandb.ai to view training curves
```

wandb sẽ tự động ghi log các chỉ số sau:
- `train/reward`: Phần thưởng trung bình
- `train/kl`: KL divergence
- `train/loss`: Loss huấn luyện
- `train/learning_rate`: Learning rate
- `train/epoch`: Epoch huấn luyện

**Cách 2: Dùng TensorBoard**

TensorBoard là công cụ trực quan hóa do TensorFlow cung cấp, cũng hỗ trợ huấn luyện PyTorch.

```python
# 1. TensorBoard logs will be automatically created in output_dir during training
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/grpo_tb",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True,
})

# 2. Launch TensorBoard to view training curves
# Run in command line:
# tensorboard --logdir=./models/grpo_tb
# Then visit http://localhost:6006
```

**Cách 3: Giám sát ngoại tuyến (Không cần công cụ bên ngoài)**

Nếu bạn không muốn dùng wandb hoặc TensorBoard, bạn cũng có thể giám sát qua log huấn luyện:

```python
# Training process will print detailed logs
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/grpo_simple",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True,
})

# Log example:
# Epoch 1/2 | Step 100/500 | Reward: 0.45 | KL: 0.023 | Loss: 1.234
# Epoch 1/2 | Step 200/500 | Reward: 0.52 | KL: 0.031 | Loss: 1.156
# ...
```

Trong huấn luyện GRPO, bạn có thể gặp một số vấn đề. Khi phần thưởng không tăng, có thể là do learning rate quá nhỏ hoặc phạt KL quá lớn giới hạn việc cập nhật policy, hoặc thiết kế hàm phần thưởng không hợp lý hay chất lượng mô hình SFT quá kém. Trong trường hợp này, hãy tăng learning rate (từ 1e-5 lên 5e-5), giảm kl_coef (từ 0.1 xuống 0.05), kiểm tra hàm phần thưởng, hoặc huấn luyện lại mô hình SFT.

Khi KL divergence bùng nổ (vượt 0.5 hoặc thậm chí 1.0) gây lộn xộn định dạng câu trả lời được sinh ra, thường là do learning rate quá lớn hoặc phạt KL quá nhỏ, hoặc hàm phần thưởng quá quyết liệt (aggressive). Bạn có thể giảm learning rate (từ 5e-5 xuống 1e-5), tăng kl_coef (từ 0.05 lên 0.1), điều chỉnh hàm phần thưởng, hoặc dùng gradient clipping.

Khi chất lượng sinh suy giảm (độ chính xác cải thiện nhưng định dạng lộn xộn, suy luận không rõ ràng), có thể là do hàm phần thưởng chỉ tập trung vào độ chính xác mà bỏ qua các chỉ số chất lượng khác, hoặc phạt KL quá nhỏ khiến mô hình đi chệch quá xa so với SFT, hoặc xảy ra overfitting. Trong trường hợp này, hãy dùng hàm phần thưởng kết hợp để tối ưu đồng thời nhiều chỉ số, tăng kl_coef để duy trì tính nhất quán, giảm số epoch huấn luyện, hoặc tăng dữ liệu huấn luyện.

Huấn luyện GRPO có mức sử dụng bộ nhớ cao hơn SFT vì nó cần sinh nhiều câu trả lời đồng thời và lưu đầu ra của mô hình tham chiếu, dễ bị OOM. Bạn có thể giảm num_generations (từ 8 xuống 4), batch_size (từ 4 xuống 2), hoặc max_new_tokens (từ 512 xuống 256), hoặc dùng gradient checkpointing và huấn luyện độ chính xác hỗn hợp để giảm nhẹ.

## 11.5 Đánh giá và phân tích mô hình

Sau khi huấn luyện xong, chúng ta cần đánh giá toàn diện hiệu năng của mô hình, không chỉ nhìn vào độ chính xác như một chỉ số đơn lẻ, mà còn phân tích sâu chất lượng suy luận, các mẫu lỗi, khả năng khái quát hóa, v.v. của mô hình. Mục này sẽ giới thiệu cách đánh giá và phân tích một cách có hệ thống các mô hình Agentic RL.

### 11.5.1 Hệ thống chỉ số đánh giá

Một hệ thống đánh giá tốt cần đa chiều, đo lường năng lực của mô hình từ các góc độ khác nhau. Chúng ta chia các chỉ số đánh giá thành ba nhóm: chỉ số độ chính xác, chỉ số hiệu quả, và chỉ số chất lượng.

**(1) Chỉ số độ chính xác (Accuracy Metrics)**

Chỉ số độ chính xác đo lường việc mô hình có thể đi đến câu trả lời đúng hay không.

**Accuracy (Độ chính xác)**: Chỉ số cơ bản nhất, tỷ lệ câu trả lời hoàn toàn đúng. Công thức tính:
$$
\text{Accuracy} = \frac{\text{Number of correct answers}}{\text{Total number of questions}}
$$

Ưu điểm là đơn giản và trực quan, dễ hiểu và dễ so sánh. Nhược điểm là không thể phân biệt "gần đúng" với "hoàn toàn sai", có thể quá thô đối với các nhiệm vụ phức tạp.

**Top-K Accuracy (Độ chính xác Top-K)**: Sinh K câu trả lời, tính là đúng nếu có ít nhất một câu đúng. Công thức tính:
$$
\text{Accuracy@K} = \frac{\text{Number of questions with at least one correct answer}}{\text{Total number of questions}}
$$

Chỉ số này phản ánh "tiềm năng" của mô hình, tức là liệu có thể tìm được câu trả lời đúng thông qua lấy mẫu nhiều lần hay không.

**Numerical Error (Sai số số học)**: Đối với các bài toán, có thể tính sai số giữa giá trị dự đoán và giá trị thực. Công thức tính:

$$
\text{Error} = \frac{1}{N} \sum_{i=1}^{N} |y_i - \hat{y}_i|
$$

Chỉ số này có thể phân biệt "gần đúng" (ví dụ, dự đoán 72.5, thực tế 72) với "hoàn toàn sai" (ví dụ, dự đoán 100, thực tế 72).

**(2) Chỉ số hiệu quả (Efficiency Metrics)**

Chỉ số hiệu quả đo lường chi phí sinh câu trả lời.

**Average Length (Độ dài trung bình)**: Số token trung bình trong các câu trả lời được sinh ra. Công thức tính:

$$
\text{Avg Length} = \frac{1}{N} \sum_{i=1}^{N} |y_i|
$$

Câu trả lời ngắn hơn nghĩa là chi phí suy luận thấp hơn và tốc độ phản hồi nhanh hơn.

**Reasoning Steps (Số bước suy luận)**: Số bước suy luận chứa trong các câu trả lời. Công thức tính:

$$
\text{Avg Steps} = \frac{1}{N} \sum_{i=1}^{N} s_i
$$

Số bước phù hợp (2-5 bước) cho thấy mô hình có thể phân rã bài toán một cách có hệ thống; quá nhiều bước có thể cho thấy suy luận dư thừa.

**Inference Time (Thời gian suy luận)**: Thời gian cần để sinh một câu trả lời. Chỉ số này quan trọng trong triển khai thực tế, ảnh hưởng đến trải nghiệm người dùng.

**(3) Chỉ số chất lượng (Quality Metrics)**

Chỉ số chất lượng đo lường tính dễ đọc và tính diễn giải được của câu trả lời.

**Format Correctness (Độ đúng định dạng)**: Câu trả lời có tuân theo định dạng mong đợi hay không (ví dụ, chứa các ký hiệu như "Step 1", "Final Answer"). Công thức tính:
$$
\text{Format Correctness} = \frac{\text{Number of correctly formatted answers}}{\text{Total number of answers}}
$$

Định dạng đúng là yêu cầu cơ bản; những câu trả lời có định dạng lộn xộn khó sử dụng ngay cả khi kết quả đúng.

**Reasoning Coherence (Tính mạch lạc của suy luận)**: Các bước suy luận có mạch lạc về mặt logic hay không. Chỉ số này thường đòi hỏi đánh giá thủ công hoặc các mô hình đánh giá chuyên biệt.

**Explainability (Tính giải thích được)**: Câu trả lời có dễ hiểu và dễ kiểm chứng hay không. Những câu trả lời có các bước rõ ràng dễ giải thích hơn những câu trả lời trực tiếp đưa ra kết quả.

Như minh họa ở Bảng 11.7, so sánh các chỉ số khác nhau.

<div align="center">
  <p>Bảng 11.7 So sánh các chỉ số đánh giá</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-7.png" alt="" width="85%"/>
</div>


### 11.5.2 Thực hành đánh giá

HelloAgents cung cấp chức năng đánh giá toàn diện, có khả năng tính nhiều chỉ số cùng lúc.

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# Comprehensive evaluation
print("=" * 50)
print("Comprehensive GRPO Model Evaluation")
print("=" * 50)

result = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/grpo_full",
    "max_samples": 200,
    "use_lora": True,

    # Evaluation configuration
    "metrics": [
        "accuracy",           # Accuracy
        "accuracy_at_k",      # Top-K accuracy
        "average_length",     # Average length
        "average_steps",      # Average steps
        "format_correctness", # Format correctness
    ],
    "k": 3,  # Top-3 accuracy
})

# Parse results
eval_data = json.loads(result)

# Print results
print(f"\nEvaluation results:")
print(f"  Accuracy: {eval_data['accuracy']}")
print(f"  Average reward: {eval_data['average_reward']}")
print(f"  Test samples: {eval_data['num_samples']}")
```

Chúng ta có thể so sánh hiệu năng của mô hình pretraining, mô hình SFT, và mô hình GRPO:

```python
# Evaluate three models
models = [
    ("Pretrained Model", "Qwen/Qwen3-0.6B", False),
    ("SFT Model", "./models/sft_full", True),
    ("GRPO Model", "./models/grpo_full", True),
]

results = []
for name, path, use_lora in models:
    print(f"\nEvaluating {name}...")
    result = rl_tool.run({
        "action": "evaluate",
        "model_path": path,
        "max_samples": 200,
        "use_lora": use_lora,
        "metrics": ["accuracy", "average_length", "format_correctness"],
    })
    results.append((name, result))

# Print comparison table
print("\n" + "=" * 70)
print(f"{'Model':<15} {'Accuracy':<12} {'Avg Length':<15} {'Format Correct':<12}")
print("=" * 70)
for name, result in results:
    print(f"{name:<15} {result['accuracy']:<12.2%} {result['average_length']:<15.1f} {result['format_correctness']:<12.2%}")
print("=" * 70)
```

### 11.5.3 Phân tích lỗi

Chỉ biết độ chính xác thôi thì chưa đủ; chúng ta cần phân tích sâu xem mô hình dễ mắc lỗi ở những loại bài toán nào, từ đó dẫn dắt các cải tiến về sau. Lỗi của mô hình có thể chia thành bốn nhóm: lỗi tính toán (calculation error - các bước suy luận đúng nhưng tính toán sai, ví dụ "48/2=25", cho thấy năng lực tính toán số học chưa đủ), lỗi suy luận (reasoning error - logic suy luận sai dẫn tới cách tiếp cận giải sai, ví dụ cộng trước rồi mới chia thay vì chia trước rồi cộng, cho thấy năng lực suy luận logic chưa đủ), lỗi hiểu (comprehension error - không hiểu đúng bài toán, ví dụ câu hỏi hỏi "tổng cộng" nhưng chỉ tính một phần, cho thấy năng lực hiểu ngôn ngữ chưa đủ), lỗi định dạng (format error - câu trả lời đúng nhưng định dạng không đạt yêu cầu, ví dụ thiếu ký hiệu "Final Answer:", cho thấy việc học định dạng chưa đủ).

Ví dụ phân tích lỗi:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# Evaluate and collect error samples
result = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/grpo_full",
    "max_samples": 200,
    "use_lora": True,
    "return_details": True,  # Return detailed results
})

# Analyze error samples
errors = result['errors']  # Error sample list
print(f"Total errors: {len(errors)}")

# Classify by error type
error_types = {
    "Calculation Error": 0,
    "Reasoning Error": 0,
    "Comprehension Error": 0,
    "Format Error": 0,
}

for error in errors:
    question = error['question']
    prediction = error['prediction']
    ground_truth = error['ground_truth']

    # Simple error classification logic (may need more complex analysis in practice)
    if "Final Answer:" not in prediction:
        error_types["Format Error"] += 1
    elif "Step" in prediction:
        # Has reasoning steps, may be calculation or reasoning error
        # More detailed analysis needed here
        error_types["Calculation Error"] += 1
    else:
        error_types["Comprehension Error"] += 1

# Print error distribution
print("\nError type distribution:")
for error_type, count in error_types.items():
    percentage = count / len(errors) * 100
    print(f"  {error_type}: {count} ({percentage:.1f}%)")
```

Ví dụ đầu ra:

```bash
Total errors: 76

Error type distribution:
  Calculation Error: 32 (42.1%)
  Reasoning Error: 18 (23.7%)
  Comprehension Error: 22 (28.9%)
  Format Error: 4 (5.3%)
```

Như có thể thấy, lỗi tính toán là loại lỗi chủ yếu (42.1%), cho thấy năng lực tính toán số học của mô hình cần được củng cố. Lỗi định dạng hiếm gặp (5.3%), cho thấy huấn luyện SFT đã hiệu quả. Chúng ta cũng có thể phân tích hiệu năng của mô hình trên các bài toán có độ khó khác nhau:

```python
# Group by number of reasoning steps
step_groups = {
    "Easy (1-2 steps)": [],
    "Medium (3-4 steps)": [],
    "Hard (5+ steps)": [],
}

for sample in result['details']:
    steps = sample['ground_truth_steps']  # Number of steps in true answer
    correct = sample['correct']

    if steps <= 2:
        step_groups["Easy (1-2 steps)"].append(correct)
    elif steps <= 4:
        step_groups["Medium (3-4 steps)"].append(correct)
    else:
        step_groups["Hard (5+ steps)"].append(correct)

# Calculate accuracy for each group
print("\nAccuracy at different difficulty levels:")
for group_name, results in step_groups.items():
    if len(results) > 0:
        accuracy = sum(results) / len(results)
        print(f"  {group_name}: {accuracy:.2%} ({len(results)} samples)")
```

Ví dụ đầu ra:

```bash
Accuracy at different difficulty levels:
  Easy (1-2 steps): 78.50% (85 samples)
  Medium (3-4 steps): 58.30% (96 samples)
  Hard (5+ steps): 31.60% (19 samples)
```

Như có thể thấy, mô hình hoạt động tốt trên các bài toán dễ (78.5%) nhưng kém trên các bài toán khó (31.6%). Điều này cho thấy năng lực suy luận nhiều bước của mô hình cần cải thiện.

### 11.5.4 Các hướng cải thiện

Dựa trên kết quả đánh giá và phân tích, chúng ta có thể xác định các hướng cải thiện cho mô hình, như minh họa ở Hình 11.8.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-8.png" alt="" width="85%"/>
  <p>Hình 11.8 Quy trình lặp cải thiện mô hình</p>
</div>

Đây là một quá trình lặp liên tục: huấn luyện mô hình → đánh giá hiệu năng → phân tích lỗi → xác định vấn đề → chọn hướng cải thiện → huấn luyện lại. Thông qua nhiều lần lặp, hiệu năng của mô hình sẽ liên tục cải thiện.

## 11.6 Thực hành pipeline huấn luyện hoàn chỉnh

Ở các mục trước, chúng ta đã học riêng lẻ về chuẩn bị dữ liệu, huấn luyện SFT, huấn luyện GRPO, và đánh giá mô hình. Bây giờ, hãy tích hợp những kiến thức này để hoàn thành một pipeline huấn luyện Agentic RL đầu-cuối (end-to-end).

### 11.6.1 Pipeline huấn luyện đầu-cuối

Một pipeline huấn luyện Agentic RL hoàn chỉnh gồm các giai đoạn sau: chuẩn bị dữ liệu, huấn luyện SFT, đánh giá SFT, huấn luyện GRPO, đánh giá GRPO, và triển khai mô hình. Như minh họa ở Hình 11.9.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-9.png" alt="" width="85%"/>
  <p>Hình 11.9 Pipeline huấn luyện đầu-cuối</p>
</div>

Hãy triển khai pipeline này qua một script hoàn chỉnh:

```python
"""
Complete Agentic RL Training Pipeline
End-to-end example from data preparation to model deployment
"""

from hello_agents.tools import RLTrainingTool
import json
from datetime import datetime

class AgenticRLPipeline:
    """Agentic RL Training Pipeline"""

    def __init__(self, config_path="config.json"):
        """
        Initialize training pipeline

        Args:
            config_path: Configuration file path
        """
        self.rl_tool = RLTrainingTool()
        self.config = self.load_config(config_path)
        self.results = {}

    def load_config(self, config_path):
        """Load configuration file"""
        with open(config_path, 'r') as f:
            return json.load(f)

    def log(self, message):
        """Log message"""
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        print(f"[{timestamp}] {message}")

    def stage1_prepare_data(self):
        """Stage 1: Data Preparation"""
        self.log("=" * 50)
        self.log("Stage 1: Data Preparation")
        self.log("=" * 50)

        # Load and check dataset
        result = self.rl_tool.run({
            "action": "load_dataset",
            "format": "sft",
            "max_samples": self.config["data"]["max_samples"],
        })

        # Parse JSON result
        dataset_info = json.loads(result)

        self.log(f"✓ Dataset loaded")
        self.log(f"  - Samples: {dataset_info['dataset_size']}")
        self.log(f"  - Format: {dataset_info['format']}")
        self.log(f"  - Data columns: {', '.join(dataset_info['sample_keys'])}")

        self.results["data"] = dataset_info

        return dataset_info

    def stage2_sft_training(self):
        """Stage 2: SFT Training"""
        self.log("\n" + "=" * 50)
        self.log("Stage 2: SFT Training")
        self.log("=" * 50)

        sft_config = self.config["sft"]

        result = self.rl_tool.run({
            "action": "train",
            "algorithm": "sft",
            "model_name": self.config["model"]["base_model"],
            "output_dir": sft_config["output_dir"],
            "max_samples": self.config["data"]["max_samples"],
            "num_epochs": sft_config["num_epochs"],
            "batch_size": sft_config["batch_size"],
            "use_lora": True,
            # Training monitoring configuration
            "use_wandb": self.config.get("monitoring", {}).get("use_wandb", False),
            "use_tensorboard": self.config.get("monitoring", {}).get("use_tensorboard", True),
            "wandb_project": self.config.get("monitoring", {}).get("wandb_project", None),
        })

        # Parse JSON result
        result_data = json.loads(result)

        self.log(f"✓ SFT training completed")
        self.log(f"  - Model path: {result_data['output_dir']}")
        self.log(f"  - Status: {result_data['status']}")

        self.results["sft_training"] = result_data

        return result_data["output_dir"]

    def stage3_sft_evaluation(self, model_path):
        """Stage 3: SFT Evaluation"""
        self.log("\n" + "=" * 50)
        self.log("Stage 3: SFT Evaluation")
        self.log("=" * 50)

        result = self.rl_tool.run({
            "action": "evaluate",
            "model_path": model_path,
            "max_samples": self.config["eval"]["max_samples"],
            "use_lora": True,
        })
        eval_data = json.loads(result)

        self.log(f"✓ SFT evaluation completed")
        self.log(f"  - Accuracy: {eval_data['accuracy']}")
        self.log(f"  - Average reward: {eval_data['average_reward']}")

        self.results["sft_evaluation"] = eval_data

        return eval_data

    def stage4_grpo_training(self, sft_model_path):
        """Stage 4: GRPO Training"""
        self.log("\n" + "=" * 50)
        self.log("Stage 4: GRPO Training")
        self.log("=" * 50)

        grpo_config = self.config["grpo"]

        result = self.rl_tool.run({
            "action": "train",
            "algorithm": "grpo",
            "model_name": sft_model_path,
            "output_dir": grpo_config["output_dir"],
            "max_samples": self.config["data"]["max_samples"],
            "num_epochs": grpo_config["num_epochs"],
            "batch_size": grpo_config["batch_size"],
            "use_lora": True,
            # Training monitoring configuration
            "use_wandb": self.config.get("monitoring", {}).get("use_wandb", False),
            "use_tensorboard": self.config.get("monitoring", {}).get("use_tensorboard", True),
            "wandb_project": self.config.get("monitoring", {}).get("wandb_project", None),
        })

        # Parse JSON result
        result_data = json.loads(result)

        self.log(f"✓ GRPO training completed")
        self.log(f"  - Model path: {result_data['output_dir']}")
        self.log(f"  - Status: {result_data['status']}")

        self.results["grpo_training"] = result_data

        return result_data["output_dir"]

    def stage5_grpo_evaluation(self, model_path):
        """Stage 5: GRPO Evaluation"""
        self.log("\n" + "=" * 50)
        self.log("Stage 5: GRPO Evaluation")
        self.log("=" * 50)

        result = self.rl_tool.run({
            "action": "evaluate",
            "model_path": model_path,
            "max_samples": self.config["eval"]["max_samples"],
            "use_lora": True,
        })
        eval_data = json.loads(result)

        self.log(f"✓ GRPO evaluation completed")
        self.log(f"  - Accuracy: {eval_data['accuracy']}")
        self.log(f"  - Average reward: {eval_data['average_reward']}")

        self.results["grpo_evaluation"] = eval_data

        return eval_data

    def stage6_save_results(self):
        """Stage 6: Save Results"""
        self.log("\n" + "=" * 50)
        self.log("Stage 6: Save Results")
        self.log("=" * 50)

        # Save training results
        results_path = "training_results.json"
        with open(results_path, 'w') as f:
            json.dump(self.results, f, indent=2)

        self.log(f"✓ Results saved to: {results_path}")

    def run(self):
        """Run complete pipeline"""
        try:
            # Stage 1: Data preparation
            self.stage1_prepare_data()

            # Stage 2: SFT training
            sft_model_path = self.stage2_sft_training()

            # Stage 3: SFT evaluation
            self.stage3_sft_evaluation(sft_model_path)

            # Stage 4: GRPO training
            grpo_model_path = self.stage4_grpo_training(sft_model_path)

            # Stage 5: GRPO evaluation
            self.stage5_grpo_evaluation(grpo_model_path)

            # Stage 6: Save results
            self.stage6_save_results()

            self.log("\n" + "=" * 50)
            self.log("✓ Training pipeline completed!")
            self.log("=" * 50)

        except Exception as e:
            self.log(f"\n✗ Training failed: {str(e)}")
            raise

# Usage example
if __name__ == "__main__":
    # Create configuration file
    config = {
        "model": {
            "base_model": "Qwen/Qwen3-0.6B"
        },
        "data": {
            "max_samples": 1000  # Use 1000 samples
        },
        "sft": {
            "output_dir": "./models/sft_model",
            "num_epochs": 3,
            "batch_size": 8,
        },
        "grpo": {
            "output_dir": "./models/grpo_model",
            "num_epochs": 3,
            "batch_size": 4,
        },
        "eval": {
            "max_samples": 200,
            "sft_accuracy_threshold": 0.40  # SFT accuracy threshold
        },
        "monitoring": {
            "use_wandb": False,  # Whether to use Wandb
            "use_tensorboard": True,  # Whether to use TensorBoard
            "wandb_project": "agentic-rl-pipeline"  # Wandb project name
        }
    }

    # Save configuration
    with open("config.json", 'w') as f:
        json.dump(config, f, indent=2)

    # Run training pipeline
    pipeline = AgenticRLPipeline("config.json")
    pipeline.run()
```

Chạy script này, bạn sẽ thấy toàn bộ quá trình huấn luyện.

Mẹo khi chạy:

**Bắt đầu nhỏ (Start Small)**: Đừng bắt đầu huấn luyện với toàn bộ dữ liệu ngay một lúc. Trước hết dùng 100-1000 mẫu để lặp nhanh, kiểm chứng quy trình và tham số, rồi mở rộng quy mô sau khi xác nhận hiệu quả. Điều này có thể tiết kiệm đáng kể thời gian và tài nguyên tính toán.

**Kiểm tra chất lượng dữ liệu (Data Quality Check)**: Kiểm tra chất lượng dữ liệu trước khi huấn luyện, đảm bảo định dạng đúng, câu trả lời chính xác, và không có mẫu trùng lặp. Bạn có thể dùng đoạn mã sau:

```python
def check_data_quality(dataset):
    """Check data quality"""
    issues = []

    # Check required fields
    required_fields = ["prompt", "completion"]
    for field in required_fields:
        if field not in dataset.column_names:
            issues.append(f"Missing field: {field}")

    # Check null values
    for i, sample in enumerate(dataset):
        if not sample["prompt"] or not sample["completion"]:
            issues.append(f"Sample {i} contains null values")

    # Check duplicates
    prompts = [s["prompt"] for s in dataset]
    duplicates = len(prompts) - len(set(prompts))
    if duplicates > 0:
        issues.append(f"Found {duplicates} duplicate samples")

    return issues

# Usage
issues = check_data_quality(dataset)
if issues:
    print("Data quality issues:")
    for issue in issues:
        print(f"  - {issue}")
else:
    print("✓ Data quality check passed")
```

**Tăng cường dữ liệu (Data Augmentation)**: Nếu lượng dữ liệu không đủ, hãy cân nhắc tăng cường dữ liệu, chẳng hạn viết lại câu hỏi (giữ nguyên câu trả lời), sinh các câu hỏi tương tự, hoặc dịch ngược (back translation). Nhưng cần cẩn thận duy trì chất lượng dữ liệu và tránh đưa vào nhiễu.

### 11.6.2 Điều chỉnh siêu tham số

Điều chỉnh siêu tham số (hyperparameter) là chìa khóa để cải thiện hiệu năng mô hình. Dưới đây là một số chiến lược điều chỉnh thường dùng.

**(1) Grid Search (Tìm kiếm lưới)**

Grid Search là phương pháp điều chỉnh đơn giản nhất, duyệt qua tất cả các tổ hợp tham số và chọn ra bộ tốt nhất.

```python
# Define parameter grid
param_grid = {
    "learning_rate": [1e-5, 5e-5, 1e-4],
    "lora_rank": [8, 16, 32],
    "kl_coef": [0.05, 0.1, 0.2],
}

best_accuracy = 0
best_params = None

# Traverse all combinations
for lr in param_grid["learning_rate"]:
    for rank in param_grid["lora_rank"]:
        for kl in param_grid["kl_coef"]:
            print(f"Testing parameters: lr={lr}, rank={rank}, kl={kl}")

            # Train model
            result = rl_tool.run({
                "action": "train",
                "algorithm": "grpo",
                "learning_rate": lr,
                "lora_rank": rank,
                "kl_coef": kl,
                # Other parameters...
            })

            # Evaluate model
            eval_result = rl_tool.run({
                "action": "evaluate",
                "model_path": result["model_path"],
            })

            # Update best parameters
            if eval_result["accuracy"] > best_accuracy:
                best_accuracy = eval_result["accuracy"]
                best_params = {"lr": lr, "rank": rank, "kl": kl}

print(f"Best parameters: {best_params}")
print(f"Best accuracy: {best_accuracy:.2%}")
```

Ưu điểm của grid search là đơn giản và trực tiếp, có thể tìm được tối ưu toàn cục. Nhược điểm là chi phí tính toán cao, không thực tế khi có nhiều tham số.

**(2) Random Search (Tìm kiếm ngẫu nhiên)**

Random Search lấy mẫu ngẫu nhiên các tổ hợp tham số, hiệu quả hơn grid search.

```python
import random

# Define parameter ranges
param_ranges = {
    "learning_rate": (1e-6, 1e-4),  # Log-uniform distribution
    "lora_rank": [4, 8, 16, 32, 64],
    "kl_coef": (0.01, 0.5),
}

best_accuracy = 0
best_params = None

# Random sampling N times
N = 10
for i in range(N):
    # Randomly sample parameters
    lr = 10 ** random.uniform(-6, -4)  # Log-uniform
    rank = random.choice(param_ranges["lora_rank"])
    kl = random.uniform(0.01, 0.5)

    print(f"[{i+1}/{N}] Testing parameters: lr={lr:.2e}, rank={rank}, kl={kl:.3f}")

    # Train and evaluate (same as above)
    # ...

print(f"Best parameters: {best_params}")
print(f"Best accuracy: {best_accuracy:.2%}")
```

Ưu điểm của random search là hiệu quả cao, phù hợp với các không gian tham số lớn. Nhược điểm là có thể bỏ lỡ lời giải tối ưu.

**(3) Bayesian Optimization (Tối ưu Bayes)**

Bayesian Optimization dùng các mô hình xác suất để dẫn dắt việc tìm kiếm, thông minh hơn. Có thể dùng các thư viện như Optuna:

```python
import optuna

def objective(trial):
    """Optimization objective function"""
    # Sample parameters
    lr = trial.suggest_loguniform("learning_rate", 1e-6, 1e-4)
    rank = trial.suggest_categorical("lora_rank", [8, 16, 32])
    kl = trial.suggest_uniform("kl_coef", 0.01, 0.5)

    # Train model
    result = rl_tool.run({
        "action": "train",
        "algorithm": "grpo",
        "learning_rate": lr,
        "lora_rank": rank,
        "kl_coef": kl,
        # Other parameters...
    })

    # Evaluate model
    eval_result = rl_tool.run({
        "action": "evaluate",
        "model_path": result["model_path"],
    })

    return eval_result["accuracy"]

# Create study
study = optuna.create_study(direction="maximize")
study.optimize(objective, n_trials=20)

# Print best parameters
print(f"Best parameters: {study.best_params}")
print(f"Best accuracy: {study.best_value:.2%}")
```

Ưu điểm của tối ưu Bayes là hiệu quả về mẫu cao, có thể nhanh chóng tìm được tham số tốt. Nhược điểm là triển khai phức tạp, đòi hỏi các thư viện bổ sung.

Như minh họa ở Bảng 11.8, so sánh các phương pháp điều chỉnh khác nhau.

<div align="center">
  <p>Bảng 11.8 So sánh các phương pháp điều chỉnh siêu tham số</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-8.png" alt="" width="85%"/>
</div>

### 11.6.3 Huấn luyện phân tán

Khi lượng dữ liệu và quy mô mô hình tăng lên, việc huấn luyện trên một GPU trở nên rất chậm. Lúc này chúng ta cần dùng huấn luyện phân tán (distributed training) để tăng tốc quá trình huấn luyện. HelloAgents dựa trên TRL và Hugging Face Accelerate, hỗ trợ tự nhiên huấn luyện phân tán đa GPU và đa node.

**Gợi ý lựa chọn giải pháp**:

- **Một máy nhiều GPU (2-8 card)**: Dùng DDP, đơn giản và hiệu quả
- **Mô hình lớn (>7B)**: Dùng DeepSpeed ZeRO-2 hoặc ZeRO-3
- **Cụm đa node**: Dùng DeepSpeed ZeRO-3 + Offload

**(1) Cấu hình Accelerate**

Trước hết cần tạo file cấu hình Accelerate. Chạy lệnh sau:

```bash
accelerate config
```

Chọn cấu hình theo các gợi ý:

```
In which compute environment are you running?
> This machine

Which type of machine are you using?
> multi-GPU

How many different machines will you use?
> 1

Do you wish to optimize your script with torch dynamo?
> NO

Do you want to use DeepSpeed?
> YES

Which DeepSpeed config file do you want to use?
> ZeRO-2

How many GPU(s) should be used for distributed training?
> 4
```

Việc này sẽ sinh ra một file cấu hình tại `~/.cache/huggingface/accelerate/default_config.yaml`.

**(2) Huấn luyện với DDP**

**Data Parallel (DDP - song song dữ liệu)** là giải pháp phân tán đơn giản nhất, mỗi GPU giữ một bản sao đầy đủ của mô hình, dữ liệu được chia đều trên các GPU.

**File cấu hình Accelerate** (`multi_gpu_ddp.yaml`):

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: MULTI_GPU
num_processes: 4  # Number of GPUs
machine_rank: 0
num_machines: 1
gpu_ids: all
mixed_precision: fp16
```

**Script huấn luyện** (không cần chỉnh sửa):

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# Training code remains unchanged
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/grpo_ddp",
    "num_epochs": 3,
    "batch_size": 4,  # Batch size per GPU
    "use_lora": True,
})
```

**Khởi chạy huấn luyện**:

```bash
# Using configuration file
accelerate launch --config_file multi_gpu_ddp.yaml train_script.py

# Or directly specify parameters
accelerate launch --num_processes 4 --mixed_precision fp16 train_script.py
```

**(3) Huấn luyện với DeepSpeed ZeRO**

**DeepSpeed ZeRO** giảm đáng kể mức sử dụng bộ nhớ bằng cách phân mảnh (sharding) các trạng thái optimizer, gradient, và tham số mô hình, hỗ trợ các mô hình và batch size lớn hơn.

**File cấu hình ZeRO-2** (`deepspeed_zero2.yaml`):

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: DEEPSPEED
num_processes: 4
machine_rank: 0
num_machines: 1
gpu_ids: all
mixed_precision: fp16
deepspeed_config:
  gradient_accumulation_steps: 4
  gradient_clipping: 1.0
  offload_optimizer_device: none
  offload_param_device: none
  zero3_init_flag: false
  zero_stage: 2  # ZeRO-2
```

**File cấu hình ZeRO-3** (`deepspeed_zero3.yaml`):

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: DEEPSPEED
num_processes: 4
machine_rank: 0
num_machines: 1
gpu_ids: all
mixed_precision: fp16
deepspeed_config:
  gradient_accumulation_steps: 4
  gradient_clipping: 1.0
  offload_optimizer_device: cpu  # Offload optimizer states to CPU
  offload_param_device: cpu      # Offload parameters to CPU
  zero3_init_flag: true
  zero_stage: 3  # ZeRO-3
```

**Khởi chạy huấn luyện**:

```bash
# ZeRO-2
accelerate launch --config_file deepspeed_zero2.yaml train_script.py

# ZeRO-3
accelerate launch --config_file deepspeed_zero3.yaml train_script.py
```

Như minh họa ở Bảng 11.9, đây là bảng so sánh bộ nhớ khi huấn luyện mô hình Qwen3-0.6B bằng các phương pháp khác nhau:

<div align="center">
  <p>Bảng 11.9 So sánh bộ nhớ (Mô hình Qwen3-0.6B)</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-9.png" alt="" width="85%"/>
</div>

**(4) Huấn luyện đa node**

Đối với huấn luyện quy mô cực lớn, có thể dùng nhiều node (máy).

**Cấu hình node chính** (`multi_node_main.yaml`):

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: DEEPSPEED
num_processes: 16  # 4 nodes x 4 GPUs
machine_rank: 0    # Main node
num_machines: 4
main_process_ip: 192.168.1.100  # Main node IP
main_process_port: 29500
gpu_ids: all
mixed_precision: fp16
deepspeed_config:
  zero_stage: 3
  offload_optimizer_device: cpu
  offload_param_device: cpu
```

**Cấu hình node phụ (worker)** (chỉnh `machine_rank` thành 1, 2, 3):

```yaml
machine_rank: 1  # Worker node 1
# Other configurations same
```

**Khởi chạy huấn luyện**:

```bash
# On main node
accelerate launch --config_file multi_node_main.yaml train_script.py

# On worker node 1
accelerate launch --config_file multi_node_worker1.yaml train_script.py

# On worker node 2
accelerate launch --config_file multi_node_worker2.yaml train_script.py

# On worker node 3
accelerate launch --config_file multi_node_worker3.yaml train_script.py
```

**(5) Thực hành tốt nhất cho huấn luyện phân tán**

**1. Điều chỉnh Batch Size**

Trong huấn luyện phân tán, tổng batch size = `per_device_batch_size × num_gpus × gradient_accumulation_steps`

```python
# Single GPU: batch_size=4, gradient_accumulation=4, total_batch=16
# 4GPU DDP: batch_size=4, gradient_accumulation=1, total_batch=16 (keep consistent)
```

**2. Co giãn Learning Rate**

Dùng quy tắc co giãn tuyến tính: `lr_new = lr_base × sqrt(total_batch_size_new / total_batch_size_base)`

```python
# Baseline: single GPU, batch=16, lr=5e-5
# 4GPU: batch=64, lr=5e-5 × sqrt(64/16) = 1e-4
```

**3. Giám sát và gỡ lỗi**

```python
# Enable verbose logging
export ACCELERATE_LOG_LEVEL=INFO

# Enable NCCL debugging (multi-node)
export NCCL_DEBUG=INFO

# Check GPU utilization
watch -n 1 nvidia-smi
```

### 11.6.4 Triển khai production

Sau khi huấn luyện xong, chúng ta cần triển khai mô hình lên môi trường production. Dưới đây là một số gợi ý triển khai.

**(1) Xuất mô hình (Model Export)**

Gộp (merge) các trọng số LoRA vào mô hình cơ sở để dễ triển khai hơn:

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import PeftModel

# Load base model
base_model = AutoModelForCausalLM.from_pretrained("Qwen/Qwen3-0.6B")

# Load LoRA weights
model = PeftModel.from_pretrained(base_model, "./models/grpo_model")

# Merge weights
merged_model = model.merge_and_unload()

# Save merged model
merged_model.save_pretrained("./models/merged_model")

# Save tokenizer
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen3-0.6B")
tokenizer.save_pretrained("./models/merged_model")

print("✓ Model exported to: ./models/merged_model")
```

**(2) Tối ưu suy luận (Inference Optimization)**

Dùng các kỹ thuật lượng tử hóa (quantization) và tối ưu để tăng tốc suy luận:

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

# Load model (using 8-bit quantization)
model = AutoModelForCausalLM.from_pretrained(
    "./models/merged_model",
    load_in_8bit=True,  # 8-bit quantization
    device_map="auto",  # Auto device allocation
)

tokenizer = AutoTokenizer.from_pretrained("./models/merged_model")

# Inference
def generate_answer(question):
    prompt = f"<|im_start|>user\n{question}<|im_end|>\n<|im_start|>assistant\n"
    inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

    outputs = model.generate(
        **inputs,
        max_new_tokens=512,
        temperature=0.7,
        do_sample=True,
    )

    response = tokenizer.decode(outputs[0], skip_special_tokens=False)
    return response

# Test
question = "What is 48 + 24?"
answer = generate_answer(question)
print(answer)
```

**(3) Dịch vụ API (API Service)**

Tạo dịch vụ suy luận dùng FastAPI:

```python
from fastapi import FastAPI
from pydantic import BaseModel
from transformers import AutoModelForCausalLM, AutoTokenizer

app = FastAPI()

# Load model
model = AutoModelForCausalLM.from_pretrained("./models/merged_model")
tokenizer = AutoTokenizer.from_pretrained("./models/merged_model")

class Question(BaseModel):
    text: str
    max_tokens: int = 512

class Answer(BaseModel):
    text: str
    confidence: float

@app.post("/generate", response_model=Answer)
def generate(question: Question):
    """Generate answer"""
    prompt = f"<|im_start|>user\n{question.text}<|im_end|>\n<|im_start|>assistant\n"
    inputs = tokenizer(prompt, return_tensors="pt")

    outputs = model.generate(
        **inputs,
        max_new_tokens=question.max_tokens,
        temperature=0.7,
        return_dict_in_generate=True,
        output_scores=True,
    )

    response = tokenizer.decode(outputs.sequences[0], skip_special_tokens=False)

    # Calculate confidence (simplified version)
    confidence = 0.8  # Should actually be calculated based on output probabilities

    return Answer(text=response, confidence=confidence)

# Run: uvicorn api:app --host 0.0.0.0 --port 8000
```



## 11.7 Tóm tắt chương

Trong chương này, chúng ta đã học một cách có hệ thống lý thuyết và thực hành của Agentic RL, từ các khái niệm cơ bản đến pipeline huấn luyện hoàn chỉnh, từ chuẩn bị dữ liệu đến triển khai mô hình. Hãy cùng điểm lại những nội dung chính của chương này.

**(1) Bản chất của Agentic RL**

Agentic RL coi LLM như một policy có thể học được, nhúng nó vào vòng lặp cảm nhận - quyết định - thực thi của Agent, tối ưu hiệu năng của Agent trong các nhiệm vụ nhiều bước thông qua reinforcement learning. Sự khác biệt cốt lõi của nó so với PBRFT truyền thống (Preference-Based Reinforcement Fine-Tuning) nằm ở:

- **Bản chất nhiệm vụ**: Từ tối ưu hội thoại một lượt đến ra quyết định tuần tự nhiều bước
- **Không gian trạng thái**: Từ prompt tĩnh đến các trạng thái môi trường tiến hóa động
- **Không gian hành động**: Từ sinh văn bản thuần túy đến văn bản + công cụ + thao tác môi trường
- **Thiết kế phần thưởng**: Từ đánh giá chất lượng một bước đến lợi ích tích lũy dài hạn
- **Mục tiêu tối ưu**: Từ chất lượng phản hồi ngắn hạn đến thành công nhiệm vụ dài hạn

**(2) Sáu năng lực cốt lõi**

Agentic RL nhằm tăng cường sáu năng lực cốt lõi của Agent:

1. **Reasoning (Suy luận)**: Suy diễn logic nhiều bước, học các chiến lược suy luận
2. **Tool Use (Sử dụng công cụ)**: Gọi API/công cụ, học khi nào và làm sao để dùng
3. **Memory (Bộ nhớ)**: Lưu giữ thông tin dài hạn, học quản lý bộ nhớ
4. **Planning (Lập kế hoạch)**: Lập kế hoạch chuỗi hành động, học lập kế hoạch động
5. **Self-Improvement (Tự cải thiện)**: Tối ưu qua tự phản tỉnh, học hỏi từ sai lầm
6. **Perception (Cảm nhận)**: Hiểu đa phương thức, suy luận và dùng công cụ thị giác

**(3) Pipeline huấn luyện**

Pipeline huấn luyện Agentic RL hoàn chỉnh gồm:

1. **Pretraining (Tiền huấn luyện)**: Học kiến thức ngôn ngữ trên văn bản quy mô lớn (thường dùng các mô hình pretraining sẵn có)
2. **Supervised Fine-Tuning (SFT)**: Học định dạng nhiệm vụ và năng lực suy luận cơ bản
3. **Reinforcement Learning (RL)**: Tối ưu chiến lược suy luận thông qua thử-sai, vượt qua chất lượng dữ liệu huấn luyện

Trong số này, SFT là nền tảng, RL là sự tăng cường. Không có nền tảng SFT, RL khó thành công; không có tối ưu RL, mô hình chỉ có thể bắt chước dữ liệu huấn luyện.

Nếu bạn muốn học sâu về Agentic RL, khuyến nghị theo lộ trình sau:

**Giai đoạn nền tảng**

1. **Kiến thức cơ bản về Reinforcement Learning**: Học các khái niệm cơ bản như MDP, policy gradient, PPO
2. **Kiến thức cơ bản về LLM**: Hiểu các công nghệ như Transformer, pretraining, fine-tuning
3. **Thực hành HelloAgents**: Chạy đoạn mã ví dụ trong chương này, hiểu pipeline hoàn chỉnh

**Giai đoạn nâng cao**

1. **Đào sâu TRL**: Học cách triển khai của thư viện TRL, hiểu chi tiết các thuật toán như SFT và GRPO
2. **Tập dữ liệu tùy chỉnh**: Huấn luyện mô hình bằng tập dữ liệu của riêng bạn
3. **Hàm phần thưởng tùy chỉnh**: Thiết kế các hàm phần thưởng phù hợp với nhiệm vụ của bạn
4. **Điều chỉnh tham số**: Điều chỉnh siêu tham số một cách có hệ thống, cải thiện hiệu năng mô hình

**Giai đoạn chuyên gia**

1. **Suy luận nhiều bước**: Nghiên cứu các nhiệm vụ suy luận chuỗi dài
2. **Học công cụ (Tool Learning)**: Giúp Agent học sử dụng công cụ
3. **Multi-Agent (Đa tác nhân)**: Nghiên cứu sự cộng tác đa Agent
4. **Bài báo tiên tiến**: Đọc các bài báo nghiên cứu mới nhất, theo dõi tiến bộ ở tuyến đầu



Chúng tôi hy vọng chương này giúp bạn hiểu và làm chủ công nghệ Agentic RL, áp dụng kiến thức này vào các dự án của riêng bạn, và xây dựng các hệ thống Agent thông minh hơn!



## Tài liệu tham khảo

[1] Schulman, J., Wolski, F., Dhariwal, P., Radford, A., & Klimov, O. (2017). Proximal Policy Optimization Algorithms. *arXiv preprint arXiv:1707.06347*.

[2] Shao, Z., Wang, P., Zhu, Q., Xu, R., Song, J., Zhang, M., ... & Guo, D. (2024). DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open Language Models. *arXiv preprint arXiv:2402.03300*.

[3] Hu, E. J., Shen, Y., Wallis, P., Allen-Zhu, Z., Li, Y., Wang, S., ... & Chen, W. (2021). LoRA: Low-Rank Adaptation of Large Language Models. *arXiv preprint arXiv:2106.09685*.

[4] Cobbe, K., Kosaraju, V., Bavarian, M., Chen, M., Jun, H., Kaiser, L., ... & Schulman, J. (2021). Training Verifiers to Solve Math Word Problems. *arXiv preprint arXiv:2110.14168*.

[5] Ouyang, L., Wu, J., Jiang, X., Almeida, D., Wainwright, C., Mishkin, P., ... & Lowe, R. (2022). Training language models to follow instructions with human feedback. *Advances in Neural Information Processing Systems*, 35, 27730-27744.

[6] Rafailov, R., Sharma, A., Mitchell, E., Ermon, S., Manning, C. D., & Finn, C. (2023). Direct Preference Optimization: Your Language Model is Secretly a Reward Model. *arXiv preprint arXiv:2305.18290*.

[7] Lee, H., Phatale, S., Mansoor, H., Lu, K., Mesnard, T., Bishop, C., ... & Rastogi, A. (2023). RLAIF: Scaling Reinforcement Learning from Human Feedback with AI Feedback. *arXiv preprint arXiv:2309.00267*.

[8] Wei, J., Wang, X., Schuurmans, D., Bosma, M., Ichter, B., Xia, F., ... & Zhou, D. (2022). Chain-of-Thought Prompting Elicits Reasoning in Large Language Models. *Advances in Neural Information Processing Systems*, 35, 24824-24837.

[9] von Werra, L., Belkada, Y., Tunstall, L., Beeching, E., Thrush, T., Lambert, N., & Huang, S. (2020). TRL: Transformer Reinforcement Learning. *GitHub repository*. https://github.com/huggingface/trl

[10] Qwen Team. (2025). Qwen3 Technical Report. *arXiv preprint arXiv:2505.09388*.

[11] Bai, Y., Jones, A., Ndousse, K., Askell, A., Chen, A., DasSarma, N., ... & Kaplan, J. (2022). Training a Helpful and Harmless Assistant with Reinforcement Learning from Human Feedback. *arXiv preprint arXiv:2204.05862*.

[12] Wang, X., Wei, J., Schuurmans, D., Le, Q., Chi, E., Narang, S., ... & Zhou, D. (2022). Self-Consistency Improves Chain of Thought Reasoning in Language Models. *arXiv preprint arXiv:2203.11171*.

[13] Christiano, P. F., Leike, J., Brown, T., Martic, M., Legg, S., & Amodei, D. (2017). Deep Reinforcement Learning from Human Preferences. *Advances in Neural Information Processing Systems*, 30.

[14] Stiennon, N., Ouyang, L., Wu, J., Ziegler, D., Lowe, R., Voss, C., ... & Christiano, P. F. (2020). Learning to summarize with human feedback. *Advances in Neural Information Processing Systems*, 33, 3008-3021.

[15] Ziegler, D. M., Stiennon, N., Wu, J., Brown, T. B., Radford, A., Amodei, D., ... & Irving, G. (2019). Fine-Tuning Language Models from Human Preferences. *arXiv preprint arXiv:1909.08593*.

## Bài tập

> **Lưu ý**: Một số bài tập không có đáp án chuẩn; trọng tâm là nuôi dưỡng sự hiểu biết toàn diện và năng lực thực hành của người học về Agentic RL và huấn luyện Agent.

1. Chương này đã giới thiệu sự tiến hóa từ huấn luyện LLM đến Agentic RL. Hãy phân tích:

   - Trong Bảng 11.1 của mục 11.1.3, những khác biệt giữa PBRFT (Preference-Based Reinforcement Fine-Tuning) và Agentic RL dưới framework MDP đã được so sánh. Hãy giải thích sâu: Tại sao không gian trạng thái của Agentic RL $s_t = (\text{prompt}, o_1, o_2, ..., o_t)$ bao gồm các quan sát lịch sử, trong khi trạng thái của PBRFT $s_0 = \text{prompt}$ chỉ gồm prompt ban đầu? Sự khác biệt này có tác động gì đến quá trình huấn luyện và kết quả cuối cùng?
   - Giả sử bạn muốn huấn luyện một "trợ lý gỡ lỗi mã thông minh" cần: (1) phân tích mã để tìm bug; (2) tra cứu tài liệu để hiểu cách dùng API; (3) sửa mã; (4) chạy kiểm thử để xác minh hiệu quả sửa lỗi. Hãy ánh xạ nhiệm vụ này vào framework reinforcement learning, định nghĩa rõ ràng không gian trạng thái, không gian hành động, hàm phần thưởng, và hàm chuyển trạng thái.
   - Mục 11.1.1 đã đề cập rằng học có giám sát truyền thống có hạn chế là "khó tối ưu các mục tiêu dài hạn". Hãy thiết kế một nhiệm vụ suy luận nhiều bước cụ thể (chẳng hạn chứng minh toán học, giải quyết vấn đề phức tạp), chứng minh tại sao học có giám sát chật vật khi tối ưu các bước trung gian, trong khi reinforcement learning có thể giải quyết vấn đề này thông qua phần thưởng trễ (delayed reward).

2. SFT (Supervised Fine-Tuning) và GRPO (Group Relative Policy Optimization) là hai phương pháp huấn luyện cốt lõi trong chương này. Dựa trên các mục 11.2 và 11.3, hãy suy nghĩ sâu:

   > **Lưu ý**: Đây là câu hỏi thực hành, khuyến nghị thao tác thực tế

   - Trong đoạn mã huấn luyện SFT ở mục 11.2.4, chúng ta đã dùng công nghệ LoRA (Low-Rank Adaptation) để giảm số tham số huấn luyện. Hãy phân tích: Ý tưởng cốt lõi của LoRA là gì? Tại sao nó có thể đạt hiệu quả gần bằng tinh chỉnh toàn bộ tham số với một số ít tham số (chẳng hạn 0.16%)? Trong những trường hợp nào nên chọn LoRA thay vì tinh chỉnh toàn bộ tham số?
   - Thuật toán GRPO (mục 11.3) có những ưu điểm gì so với thuật toán PPO truyền thống? Hãy so sánh quy trình huấn luyện của cả hai, phân tích cách GRPO đơn giản hóa quy trình huấn luyện và cải thiện tính ổn định thông qua "phần thưởng tương đối theo nhóm". Nếu áp dụng GRPO cho các nhiệm vụ khác (chẳng hạn sinh mã, tối ưu hội thoại), cần những điều chỉnh gì?
   - Dựa trên đoạn mã ở mục 11.2.5, hãy mở rộng pipeline huấn luyện SFT, thêm các tính năng sau: (1) hỗ trợ huấn luyện dữ liệu hội thoại nhiều lượt; (2) thêm các chiến lược tăng cường dữ liệu (chẳng hạn viết lại từ đồng nghĩa, điều chỉnh độ khó); (3) triển khai giám sát trực quan hóa quá trình huấn luyện (chẳng hạn đường cong loss, đánh giá chất lượng mẫu).

3. Thiết kế hàm phần thưởng là một thách thức cốt lõi của Agentic RL. Dựa trên mục 11.3.3, hãy hoàn thành phần thực hành mở rộng sau:

   > **Lưu ý**: Đây là câu hỏi thực hành, khuyến nghị thao tác thực tế

   - Trong mục 11.3.3, chúng ta đã thiết kế một phần thưởng nhị phân đơn giản cho các bài toán GSM8K (đúng +1, sai 0). Hãy thiết kế một hàm phần thưởng tinh tế hơn có thể: (1) cho phần thưởng một phần đối với những câu trả lời đúng một phần; (2) chấm điểm mức độ hợp lý của quá trình suy luận; (3) phạt các lối giải quá dài dòng hoặc kém hiệu quả. Hàm phần thưởng này nên được triển khai như thế nào?
   - Thiết kế hàm phần thưởng thường đòi hỏi kiến thức lĩnh vực (domain knowledge). Hãy thiết kế các hàm phần thưởng cho ba nhiệm vụ Agent khác nhau sau đây: (1) trợ lý sinh mã (cần xét độ đúng đắn của mã, tính dễ đọc, hiệu quả); (2) Agent hội thoại chăm sóc khách hàng (cần xét tỷ lệ giải quyết vấn đề, sự hài lòng của người dùng, thời gian phản hồi); (3) AI chơi game (cần xét tỷ lệ thắng, tính đa dạng chiến lược, tính bền vững đối kháng).
   - Trong các ứng dụng thực tế, hàm phần thưởng có thể gặp vấn đề "reward hacking": Agent tìm ra đường tắt để nhận phần thưởng cao nhưng thực chất không hoàn thành nhiệm vụ. Hãy đưa ra ví dụ về hiện tượng này và thiết kế các cơ chế phòng vệ để tránh reward hacking.

4. Trong tình huống "Huấn luyện Agent suy luận toán học" ở mục 11.4, chúng ta đã thấy pipeline huấn luyện hoàn chỉnh. Hãy phân tích sâu:

   - Tình huống này dùng tập dữ liệu GSM8K để huấn luyện và đánh giá. Hãy phân tích: Tập dữ liệu này có những đặc điểm gì? Nó phù hợp để huấn luyện loại năng lực suy luận nào? Nếu huấn luyện một Agent có khả năng xử lý các bài toán phức tạp hơn (chẳng hạn toán cao cấp, chứng minh toán học), thì tập dữ liệu và các phương pháp huấn luyện nên được mở rộng như thế nào?
   - Trong các kết quả huấn luyện ở mục 11.4.3, chúng ta quan sát thấy độ chính xác cải thiện trên tập huấn luyện, nhưng có thể có nguy cơ overfitting. Hãy thiết kế một kế hoạch "đánh giá năng lực khái quát hóa": Làm sao để kiểm tra xem mô hình có thực sự học được suy luận toán học hay chỉ ghi nhớ dữ liệu huấn luyện? Làm sao để cải thiện năng lực khái quát hóa thông qua các kỹ thuật như regularization, tăng cường dữ liệu?
   - Việc huấn luyện trong tình huống này là ngoại tuyến (offline - dùng các tập dữ liệu thu thập sẵn). Hãy thiết kế một kế hoạch "học trực tuyến (online learning)": Agent liên tục thu thập phản hồi của người dùng trong quá trình sử dụng thực tế và tự động cập nhật mô hình. Kế hoạch này cần xét những thách thức kỹ thuật nào (chẳng hạn kiểm soát chất lượng dữ liệu, catastrophic forgetting - quên thảm khốc, đảm bảo an toàn)?

5. Một ứng dụng quan trọng của Agentic RL là giúp Agent học sử dụng công cụ. Hãy suy nghĩ:

   - Mục 11.1.3 đã đề cập rằng Agentic RL phù hợp để tối ưu các nhiệm vụ "đòi hỏi suy luận nhiều bước, sử dụng công cụ, lập kế hoạch dài hạn". Hãy thiết kế một kế hoạch "học công cụ (tool learning)": Cho một tập công cụ (chẳng hạn công cụ tìm kiếm, máy tính, trình thực thi mã), làm sao để huấn luyện Agent học cách chọn công cụ phù hợp vào thời điểm phù hợp? Hàm phần thưởng nên được thiết kế như thế nào?
   - Việc sử dụng công cụ thường liên quan đến các phụ thuộc phức tạp (chẳng hạn "phải gọi công cụ A trước để lấy thông tin rồi mới gọi công cụ B"). Hãy thiết kế một kế hoạch "reinforcement learning phân cấp (hierarchical)": policy cấp cao chịu trách nhiệm lập kế hoạch nhiệm vụ, policy cấp thấp chịu trách nhiệm gọi công cụ. Làm sao để huấn luyện cấu trúc phân cấp này? Làm sao để phối hợp các mục tiêu tối ưu của cấp cao và cấp thấp?
   - Trong các ứng dụng thực tế, số lượng công cụ có thể rất lớn (chẳng hạn 50+ API), và việc huấn luyện trực tiếp có thể gặp vấn đề "hiệu quả khám phá thấp". Hãy thiết kế một kế hoạch "học theo chương trình (curriculum learning)": bắt đầu huấn luyện từ các nhiệm vụ đơn giản (dùng ít công cụ), rồi dần tăng độ khó nhiệm vụ và số lượng công cụ. Kế hoạch này nên thiết kế trình tự chương trình như thế nào? Làm sao để đánh giá xem Agent đã sẵn sàng bước vào giai đoạn tiếp theo hay chưa?
