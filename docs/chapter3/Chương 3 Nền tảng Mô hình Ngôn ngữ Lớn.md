# Chương 3 Nền tảng Mô hình Ngôn ngữ Lớn

Hai chương đầu đã giới thiệu định nghĩa và lịch sử phát triển của Agent. Chương này sẽ tập trung hoàn toàn vào bản thân các mô hình ngôn ngữ lớn để trả lời một câu hỏi then chốt: Các Agent hiện đại hoạt động như thế nào? Chúng ta sẽ bắt đầu từ định nghĩa cơ bản của mô hình ngôn ngữ, và thông qua việc tìm hiểu những nguyên lý này, đặt một nền tảng vững chắc để hiểu cách các LLM có được kho kiến thức phong phú và năng lực suy luận mạnh mẽ.

## 3.1 Mô hình Ngôn ngữ và Kiến trúc Transformer

### 3.1.1 Từ N-gram đến RNN

**Mô hình Ngôn ngữ (Language Model - LM)** là cốt lõi của xử lý ngôn ngữ tự nhiên, và nhiệm vụ cơ bản của nó là tính xác suất xuất hiện của một chuỗi từ (tức là một câu). Một mô hình ngôn ngữ tốt có thể cho chúng ta biết loại câu nào là trôi chảy và tự nhiên. Trong các hệ thống đa Agent, mô hình ngôn ngữ là nền tảng để Agent hiểu chỉ thị của con người và tạo ra phản hồi. Phần này sẽ điểm lại quá trình tiến hóa từ các phương pháp thống kê cổ điển đến các mô hình học sâu hiện đại, đặt nền tảng vững chắc để hiểu kiến trúc Transformer ở phần sau.

**(1) Mô hình Ngôn ngữ Thống kê và Ý tưởng N-gram**

Trước khi học sâu (deep learning) trỗi dậy, các phương pháp thống kê là dòng chủ đạo của mô hình ngôn ngữ. Ý tưởng cốt lõi là: xác suất xuất hiện của một câu bằng tích của các xác suất có điều kiện của từng từ trong câu. Đối với một câu S được cấu thành từ các từ $w_1,w_2,\cdots,w_m$, xác suất P(S) của nó có thể được biểu diễn như sau:

$$P(S)=P(w_1,w_2,…,w_m)=P(w_1)⋅P(w_2∣w_1)⋅P(w_3∣w_1,w_2)⋯P(w_m∣w_1,…,w_{m−1})$$

Công thức này được gọi là quy tắc chuỗi (chain rule) của xác suất. Tuy nhiên, việc tính trực tiếp công thức này gần như là không thể, bởi vì các xác suất có điều kiện như $P(w_m∣w_1,\cdots,w_{m−1})$ quá khó ước lượng từ một kho ngữ liệu (corpus), vì chuỗi từ $w_1,\cdots,w_{m−1}$ có thể chưa bao giờ xuất hiện trong dữ liệu huấn luyện.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-0.png" alt="Figure description" width="90%"/>
  <p>Hình 3.1 Sơ đồ minh họa giả định Markov</p>
</div>

Để giải quyết vấn đề này, các nhà nghiên cứu đã đưa ra **Giả định Markov (Markov Assumption)**. Ý tưởng cốt lõi của nó là: chúng ta không cần truy ngược lại toàn bộ lịch sử của một từ; chúng ta có thể giả định gần đúng rằng xác suất xuất hiện của một từ chỉ liên quan đến $n−1$ từ đứng trước nó, như minh họa trong Hình 3.1. Các mô hình ngôn ngữ được xây dựng dựa trên giả định này được gọi là **mô hình N-gram**. Ở đây, "N" đại diện cho kích thước cửa sổ ngữ cảnh (context window) mà chúng ta xem xét. Hãy xem một số ví dụ phổ biến nhất để hiểu khái niệm này:

- **Bigram (khi N=2)**: Đây là trường hợp đơn giản nhất, khi chúng ta giả định rằng sự xuất hiện của một từ chỉ liên quan đến một từ đứng ngay trước nó. Do đó, xác suất có điều kiện phức tạp $P(w_i∣w_1,\cdots,w_{i−1})$ trong quy tắc chuỗi có thể được xấp xỉ thành một dạng dễ tính hơn:

$$P(w_{i}∣w_{1},…,w_{i−1})≈P(w_{i}∣w_{i−1})$$

- **Trigram (khi N=3)**: Tương tự, chúng ta giả định rằng sự xuất hiện của một từ chỉ liên quan đến hai từ đứng trước nó:

$$P(w_i∣w_1,…,w_{i−1})≈P(w_i∣w_{i−2},w_{i−1})$$

Các xác suất này có thể được tính thông qua **Ước lượng Hợp lý Cực đại (Maximum Likelihood Estimation - MLE)** trên các kho ngữ liệu lớn. Thuật ngữ này nghe có vẻ phức tạp, nhưng ý tưởng của nó rất trực quan: cái gì có khả năng xuất hiện nhất chính là cái mà chúng ta thấy thường xuyên nhất trong dữ liệu. Ví dụ, đối với mô hình Bigram, chúng ta muốn tính xác suất $P(w_i∣w_{i−1})$ rằng từ tiếp theo là $w_i$ sau khi từ $w_{i−1}$ xuất hiện. Theo ước lượng hợp lý cực đại, xác suất này có thể được ước tính thông qua phép đếm đơn giản:

$$P(w_i∣w_{i−1})=\frac{Count(w_{i−1},w_i)}{Count(w_{i−1})}$$

Ở đây, hàm `Count()` biểu thị "phép đếm":

- $Count(w_{i−1},w_i)$: biểu thị tổng số lần cặp từ $(w_{i−1},w_i)$ xuất hiện liên tiếp trong kho ngữ liệu.
- $Count(w_{i−1})$: biểu thị tổng số lần từ đơn $w_{i−1}$ xuất hiện trong kho ngữ liệu.

Ý nghĩa của công thức là: chúng ta dùng "số lần cặp từ $Count(w_{i−1},w_i)$ xuất hiện" chia cho "tổng số lần từ $Count(w_{i−1})$ xuất hiện" làm ước lượng gần đúng cho $P(w_i∣w_{i−1})$.

Để làm cho quá trình này cụ thể hơn, hãy thực hiện một phép tính thủ công. Giả sử chúng ta có một kho ngữ liệu nhỏ chỉ chứa hai câu sau: `datawhale agent learns`, `datawhale agent works`. Mục tiêu của chúng ta là: sử dụng mô hình Bigram (N=2), ước lượng xác suất xuất hiện của câu `datawhale agent learns`. Theo giả định Bigram, mỗi lần chúng ta xem xét các cặp từ liên tiếp (tức là các cặp từ).

**Bước 1: Tính xác suất của từ đầu tiên** $P(datawhale)$ Đây là số lần `datawhale` xuất hiện chia cho tổng số từ. `datawhale` xuất hiện 2 lần, và tổng số từ là 6.

$$P(\text{datawhale}) = \frac{\text{Number of "datawhale" in total corpus}}{\text{Total number of words in corpus}} = \frac{2}{6} \approx 0.333$$

**Bước 2: Tính xác suất có điều kiện** $P(agent∣datawhale)$ Đây là số lần cặp từ `datawhale agent` xuất hiện chia cho tổng số lần `datawhale` xuất hiện. `datawhale agent` xuất hiện 2 lần, `datawhale` xuất hiện 2 lần.

$$P(\text{agent}|\text{datawhale}) =  \frac{\text{Count}(\text{datawhale agent})}{\text{Count}(\text{datawhale})} =  \frac{2}{2} = 1$$

**Bước 3: Tính xác suất có điều kiện** $P(learns∣agent)$ Đây là số lần cặp từ `agent learns` xuất hiện chia cho tổng số lần `agent` xuất hiện. `agent learns` xuất hiện 1 lần, `agent` xuất hiện 2 lần.

$$P(\text{learns}|\text{agent}) =  \frac{\text{Count(agent learns)}}{\text{Count(agent)}} =  \frac{1}{2} = 0.5$$

**Cuối cùng: Nhân các xác suất** Vậy, xác suất gần đúng của toàn bộ câu là:

$$P(\text{datawhale agent learns}) \approx  P(\text{datawhale}) \cdot  P(\text{agent}|\text{datawhale}) \cdot  P(\text{learns}|\text{agent}) \approx  0.333 \cdot 1 \cdot 0.5 \approx 0.167$$

```Python
import collections

# Kho ngữ liệu ví dụ, nhất quán với kho ngữ liệu trong phần giải thích trường hợp ở trên
corpus = "datawhale agent learns datawhale agent works"
tokens = corpus.split()
total_tokens = len(tokens)

# --- Bước 1: Tính P(datawhale) ---
count_datawhale = tokens.count('datawhale')
p_datawhale = count_datawhale / total_tokens
print(f"Step 1: P(datawhale) = {count_datawhale}/{total_tokens} = {p_datawhale:.3f}")

# --- Bước 2: Tính P(agent|datawhale) ---
# Trước tiên tính các bigram cho các bước tiếp theo
bigrams = zip(tokens, tokens[1:])
bigram_counts = collections.Counter(bigrams)
count_datawhale_agent = bigram_counts[('datawhale', 'agent')]
# count_datawhale đã được tính ở bước 1
p_agent_given_datawhale = count_datawhale_agent / count_datawhale
print(f"Step 2: P(agent|datawhale) = {count_datawhale_agent}/{count_datawhale} = {p_agent_given_datawhale:.3f}")

# --- Bước 3: Tính P(learns|agent) ---
count_agent_learns = bigram_counts[('agent', 'learns')]
count_agent = tokens.count('agent')
p_learns_given_agent = count_agent_learns / count_agent
print(f"Step 3: P(learns|agent) = {count_agent_learns}/{count_agent} = {p_learns_given_agent:.3f}")

# --- Cuối cùng: Nhân các xác suất ---
p_sentence = p_datawhale * p_agent_given_datawhale * p_learns_given_agent
print(f"Finally: P('datawhale agent learns') ≈ {p_datawhale:.3f} * {p_agent_given_datawhale:.3f} * {p_learns_given_agent:.3f} = {p_sentence:.3f}")

>>>
Step 1: P(datawhale) = 2/6 = 0.333
Step 2: P(agent|datawhale) = 2/2 = 1.000
Step 3: P(learns|agent) = 1/2 = 0.500
Finally: P('datawhale agent learns') ≈ 0.333 * 1.000 * 0.500 = 0.167
```

Mô hình N-gram, tuy đơn giản và hiệu quả, nhưng có hai khiếm khuyết chí mạng:

1. **Thưa thớt Dữ liệu (Data Sparsity)**: Nếu một chuỗi từ chưa bao giờ xuất hiện trong kho ngữ liệu, ước lượng xác suất của nó sẽ là 0, điều này rõ ràng là vô lý. Mặc dù có thể giảm nhẹ bằng các kỹ thuật làm mượt (smoothing), nhưng không thể loại bỏ hoàn toàn.
2. **Khả năng Tổng quát hóa Kém**: Mô hình không thể hiểu được sự tương đồng ngữ nghĩa giữa các từ. Ví dụ, ngay cả khi mô hình đã thấy `agent learns` nhiều lần trong kho ngữ liệu, nó cũng không thể tổng quát hóa kiến thức này sang các từ tương đồng về mặt ngữ nghĩa. Khi chúng ta tính xác suất của `robot learns`, nếu từ `robot` chưa bao giờ xuất hiện, hoặc nếu tổ hợp `robot learns` chưa bao giờ xuất hiện, thì xác suất do mô hình tính ra cũng sẽ bằng không. Mô hình không thể hiểu được sự tương đồng ngữ nghĩa giữa `agent` và `robot`.

**(2) Mô hình Ngôn ngữ Mạng Nơ-ron và Word Embedding**

Khiếm khuyết cơ bản của mô hình N-gram là chúng coi các từ như những ký hiệu rời rạc, biệt lập. Để khắc phục vấn đề này, các nhà nghiên cứu đã chuyển sang mạng nơ-ron và đề xuất một ý tưởng: biểu diễn các từ bằng các vector liên tục. Vào năm 2003, **Mô hình Ngôn ngữ Mạng Nơ-ron Truyền thẳng (Feedforward Neural Network Language Model)** do Bengio và cộng sự đề xuất là một cột mốc trong lĩnh vực này<sup>[1]</sup>.

Ý tưởng cốt lõi của nó có thể chia thành hai bước:

1. **Xây dựng không gian ngữ nghĩa**: Tạo một không gian vector liên tục nhiều chiều, sau đó ánh xạ mỗi từ trong bộ từ vựng thành một điểm trong không gian đó. Điểm này (tức là vector) được gọi là **Word Embedding** (nhúng từ) hay word vector (vector từ). Trong không gian này, các từ tương đồng về mặt ngữ nghĩa sẽ có các vector gần nhau về vị trí. Ví dụ, các vector của `agent` và `robot` sẽ rất gần nhau, trong khi các vector của `agent` và `apple` sẽ ở xa nhau.
2. **Học ánh xạ từ ngữ cảnh sang từ tiếp theo**: Tận dụng khả năng khớp (fitting) mạnh mẽ của mạng nơ-ron để học một hàm số. Đầu vào của hàm này là các word vector của $n−1$ từ đứng trước, và đầu ra là phân phối xác suất của mỗi từ trong bộ từ vựng xuất hiện sau ngữ cảnh hiện tại.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-1.png" alt="Figure description" width="90%"/>
  <p>Hình 3.2 Sơ đồ minh họa kiến trúc mô hình ngôn ngữ mạng nơ-ron</p>
</div>

Như minh họa trong Hình 3.2, trong kiến trúc này, word embedding được học một cách tự động trong quá trình huấn luyện mô hình. Để hoàn thành nhiệm vụ "dự đoán từ tiếp theo," mô hình liên tục điều chỉnh vị trí vector của mỗi từ, cuối cùng làm cho các vector này chứa đựng thông tin ngữ nghĩa phong phú. Một khi chúng ta chuyển các từ thành vector, chúng ta có thể sử dụng các công cụ toán học để đo lường mối quan hệ giữa chúng. Phương pháp được sử dụng phổ biến nhất là **Độ tương đồng Cosine (Cosine Similarity)**, đo lường sự tương đồng của chúng bằng cách tính cosine của góc giữa hai vector.

$$\text{similarity}(\vec{a}, \vec{b}) = \cos(\theta) = \frac{\vec{a} \cdot \vec{b}}{|\vec{a}| |\vec{b}|}$$

Ý nghĩa của công thức này là:

- Nếu hai vector có cùng hướng chính xác, góc là 0°, giá trị cosine là 1, biểu thị sự tương quan hoàn toàn.
- Nếu hai vector trực giao, góc là 90°, giá trị cosine là 0, biểu thị không có mối quan hệ.
- Nếu hai vector có hướng hoàn toàn ngược nhau, góc là 180°, giá trị cosine là -1, biểu thị sự tương quan âm hoàn toàn.

Thông qua phương pháp này, word vector không chỉ có thể nắm bắt các mối quan hệ đơn giản như "từ đồng nghĩa" mà còn nắm bắt được các mối quan hệ loại suy phức tạp hơn.

Một ví dụ nổi tiếng minh họa các mối quan hệ ngữ nghĩa được word vector nắm bắt: `vector('King') - vector('Man') + vector('Woman')` Kết quả của phép toán vector này lại gần một cách đáng kinh ngạc với vị trí của `vector('Queen')` trong không gian vector. Điều này giống như thực hiện một phép dịch ngữ nghĩa: chúng ta bắt đầu từ điểm "vua," trừ đi vector của "nam giới," cộng thêm vector của "nữ giới," và cuối cùng đến vị trí của "nữ hoàng." Điều này chứng minh rằng word embedding có thể học được các khái niệm trừu tượng như "giới tính" và "hoàng gia."

```Python
import numpy as np

# Giả sử chúng ta đã học được các word vector 2D đơn giản hóa
embeddings = {
    "king": np.array([0.9, 0.8]),
    "queen": np.array([0.9, 0.2]),
    "man": np.array([0.7, 0.9]),
    "woman": np.array([0.7, 0.3])
}

def cosine_similarity(vec1, vec2):
    dot_product = np.dot(vec1, vec2)
    norm_product = np.linalg.norm(vec1) * np.linalg.norm(vec2)
    return dot_product / norm_product

# king - man + woman
result_vec = embeddings["king"] - embeddings["man"] + embeddings["woman"]

# Tính độ tương đồng giữa vector kết quả và "queen"
sim = cosine_similarity(result_vec, embeddings["queen"])

print(f"Result vector of king - man + woman: {result_vec}")
print(f"Similarity of this result with 'queen': {sim:.4f}")

>>>
Result vector of king - man + woman: [0.9 0.2]
Similarity of this result with 'queen': 1.0000
```

Mô hình ngôn ngữ mạng nơ-ron đã giải quyết thành công vấn đề tổng quát hóa kém của mô hình N-gram thông qua word embedding. Tuy nhiên, chúng vẫn có một hạn chế tương tự như N-gram: cửa sổ ngữ cảnh là cố định. Chúng chỉ có thể xem xét một số lượng từ đứng trước cố định, điều này đặt nền móng cho các mạng nơ-ron hồi quy có thể xử lý các chuỗi có độ dài tùy ý.

**(3) Mạng Nơ-ron Hồi quy (RNN) và Mạng Bộ nhớ Ngắn hạn Dài (LSTM)**

Mặc dù mô hình ngôn ngữ mạng nơ-ron ở phần trước đã đưa vào word embedding để giải quyết vấn đề tổng quát hóa, nhưng giống như mô hình N-gram, cửa sổ ngữ cảnh của nó có kích thước cố định. Để dự đoán từ tiếp theo, nó chỉ có thể thấy $n−1$ từ đứng trước, còn thông tin lịch sử sớm hơn sẽ bị loại bỏ. Điều này rõ ràng không phù hợp với cách con người chúng ta hiểu ngôn ngữ. Để phá vỡ giới hạn của cửa sổ cố định, **Mạng Nơ-ron Hồi quy (Recurrent Neural Networks - RNN)** đã xuất hiện, với một ý tưởng cốt lõi rất trực quan: thêm khả năng "ghi nhớ" vào mạng<sup>[2]</sup>.

Như minh họa trong Hình 3.3, thiết kế của RNN đưa vào một vector **trạng thái ẩn (hidden state)**, mà chúng ta có thể hiểu là bộ nhớ ngắn hạn của mạng. Tại mỗi bước xử lý chuỗi, mạng đọc từ đầu vào hiện tại và kết hợp nó với bộ nhớ của nó từ thời điểm trước đó (tức là trạng thái ẩn từ bước thời gian trước), sau đó tạo ra một bộ nhớ mới (tức là trạng thái ẩn của bước thời gian hiện tại) để truyền sang thời điểm tiếp theo. Quá trình mang tính chu kỳ này cho phép thông tin liên tục lan truyền ngược về sau qua chuỗi.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-2.png" alt="Figure description" width="90%"/>
  <p>Hình 3.3 Sơ đồ minh họa cấu trúc RNN</p>
</div>

Tuy nhiên, RNN tiêu chuẩn có một vấn đề nghiêm trọng trong thực tế: **Vấn đề Phụ thuộc Dài hạn (Long-term Dependency Problem)**. Trong quá trình huấn luyện, mô hình cần điều chỉnh các trọng số ở sâu trong mạng dựa trên các sai số ở đầu ra thông qua thuật toán lan truyền ngược (backpropagation). Đối với RNN, độ dài của chuỗi chính là độ sâu của mạng. Khi chuỗi rất dài, các gradient trải qua nhiều phép nhân trong quá trình lan truyền ngược, điều này khiến giá trị gradient nhanh chóng tiến về không (**tiêu biến gradient - gradient vanishing**) hoặc trở nên cực kỳ lớn (**bùng nổ gradient - gradient explosion**). Tiêu biến gradient khiến mô hình không thể học một cách hiệu quả tác động của thông tin ở đầu chuỗi lên các đầu ra ở sau, khiến việc nắm bắt các phụ thuộc dài hạn trở nên khó khăn.

Để giải quyết vấn đề phụ thuộc dài hạn, **Mạng Bộ nhớ Ngắn hạn Dài (Long Short-Term Memory - LSTM)** đã được thiết kế<sup>[3]</sup>. LSTM là một loại RNN đặc biệt, và đổi mới cốt lõi của nó nằm ở việc đưa vào **Trạng thái Ô (Cell State)** và một **Cơ chế Cổng (Gating Mechanism)** tinh vi. Trạng thái ô có thể được xem như một đường dẫn thông tin độc lập với trạng thái ẩn, cho phép thông tin truyền đi mượt mà hơn giữa các bước thời gian. Cơ chế cổng bao gồm một vài mạng nơ-ron nhỏ có thể học cách chọn lọc cho thông tin đi qua, từ đó kiểm soát việc thêm vào và loại bỏ thông tin trong trạng thái ô. Các cổng này bao gồm:

- **Cổng Quên (Forget Gate)**: Quyết định thông tin nào cần loại bỏ khỏi trạng thái ô của thời điểm trước đó.
- **Cổng Đầu vào (Input Gate)**: Quyết định thông tin mới nào từ đầu vào hiện tại cần lưu trữ vào trạng thái ô.
- **Cổng Đầu ra (Output Gate)**: Quyết định thông tin nào cần xuất ra trạng thái ẩn dựa trên trạng thái ô hiện tại.

### 3.1.2 Phân tích Kiến trúc Transformer

Trong phần trước, chúng ta đã thấy rằng RNN và LSTM xử lý dữ liệu tuần tự bằng cách đưa vào các cấu trúc hồi quy, điều này ở một mức độ nào đó đã giải quyết vấn đề nắm bắt các phụ thuộc dài hạn. Tuy nhiên, phương pháp tính toán hồi quy này cũng mang lại những nút thắt cổ chai mới: nó buộc phải xử lý dữ liệu một cách tuần tự. Phép tính tại bước thời gian t phải chờ bước thời gian t−1 hoàn thành mới có thể bắt đầu. Điều này có nghĩa là RNN không thể thực hiện tính toán song song quy mô lớn và kém hiệu quả khi xử lý các chuỗi dài, điều này giới hạn rất nhiều sự cải thiện về quy mô mô hình và tốc độ huấn luyện. Transformer được đề xuất bởi nhóm nghiên cứu của Google vào năm 2017<sup>[4]</sup>. Nó hoàn toàn từ bỏ cấu trúc hồi quy và thay vào đó dựa hoàn toàn vào một cơ chế gọi là **Chú ý (Attention)** để nắm bắt các phụ thuộc bên trong chuỗi, từ đó đạt được tính toán song song thực sự.

**(1) Cấu trúc Encoder-Decoder Tổng thể**

Mô hình Transformer ban đầu được thiết kế cho nhiệm vụ đầu-cuối (end-to-end) là dịch máy. Như minh họa trong Hình 3.4, ở cấp độ vĩ mô, nó tuân theo một kiến trúc **Encoder-Decoder** cổ điển.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-3.png" alt="Figure description" width="50%"/>
  <p>Hình 3.4 Sơ đồ kiến trúc Transformer tổng thể</p>
</div>

Chúng ta có thể hiểu cấu trúc này như một đội ngũ có sự phân công rõ ràng:

1. **Encoder (Bộ mã hóa)**: Nhiệm vụ là "**hiểu**" toàn bộ câu đầu vào. Nó đọc tất cả các token đầu vào (khái niệm này sẽ được giới thiệu trong Phần 3.2.2) và cuối cùng tạo ra một biểu diễn vector giàu thông tin ngữ cảnh cho mỗi token.
2. **Decoder (Bộ giải mã)**: Nhiệm vụ là "**tạo ra**" câu đích. Nó tham chiếu đến phần văn bản trước đó mà nó đã tạo ra và "tham vấn" kết quả hiểu biết của encoder để tạo ra từ tiếp theo.

Để thực sự hiểu Transformer hoạt động như thế nào, phương pháp tốt nhất là tự mình triển khai nó. Trong phần này, chúng ta sẽ áp dụng cách tiếp cận "từ trên xuống" (top-down): trước tiên, chúng ta xây dựng khung mã (code framework) hoàn chỉnh của Transformer, định nghĩa tất cả các lớp (class) và phương thức (method) cần thiết. Sau đó, giống như hoàn thành một trò chơi ghép hình, chúng ta sẽ triển khai từng chức năng cụ thể của các lớp này.

```Python
import torch
import torch.nn as nn
import math

# --- Các module giữ chỗ (placeholder), sẽ được triển khai trong các phần sau ---

class PositionalEncoding(nn.Module):
    """
    Module mã hóa vị trí
    """
    def forward(self, x):
        pass

class MultiHeadAttention(nn.Module):
    """
    Module cơ chế chú ý đa đầu
    """
    def forward(self, query, key, value, mask):
        pass

class PositionWiseFeedForward(nn.Module):
    """
    Module mạng truyền thẳng theo vị trí
    """
    def forward(self, x):
        pass

# --- Lớp cốt lõi của Encoder ---

class EncoderLayer(nn.Module):
    def __init__(self, d_model, num_heads, d_ff, dropout):
        super(EncoderLayer, self).__init__()
        self.self_attn = MultiHeadAttention() # Sẽ triển khai sau
        self.feed_forward = PositionWiseFeedForward() # Sẽ triển khai sau
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x, mask):
        # Kết nối phần dư (residual connection) và chuẩn hóa lớp (layer normalization) sẽ được giải thích chi tiết trong Phần 3.1.2.4
        # 1. Tự chú ý đa đầu (multi-head self-attention)
        attn_output = self.self_attn(x, x, x, mask)
        x = self.norm1(x + self.dropout(attn_output))

        # 2. Mạng truyền thẳng (feed-forward network)
        ff_output = self.feed_forward(x)
        x = self.norm2(x + self.dropout(ff_output))

        return x

# --- Lớp cốt lõi của Decoder ---

class DecoderLayer(nn.Module):
    def __init__(self, d_model, num_heads, d_ff, dropout):
        super(DecoderLayer, self).__init__()
        self.self_attn = MultiHeadAttention() # Sẽ triển khai sau
        self.cross_attn = MultiHeadAttention() # Sẽ triển khai sau
        self.feed_forward = PositionWiseFeedForward() # Sẽ triển khai sau
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.norm3 = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)

    def forward(self, x, encoder_output, src_mask, tgt_mask):
        # 1. Tự chú ý đa đầu có mặt nạ (trên chính nó)
        attn_output = self.self_attn(x, x, x, tgt_mask)
        x = self.norm1(x + self.dropout(attn_output))

        # 2. Chú ý chéo (cross-attention) (trên đầu ra của encoder)
        cross_attn_output = self.cross_attn(x, encoder_output, encoder_output, src_mask)
        x = self.norm2(x + self.dropout(cross_attn_output))

        # 3. Mạng truyền thẳng
        ff_output = self.feed_forward(x)
        x = self.norm3(x + self.dropout(ff_output))

        return x
```

**(2) Từ Tự Chú ý đến Chú ý Đa đầu**

Bây giờ, hãy điền vào module then chốt nhất trong bộ khung: cơ chế chú ý.

Hãy tưởng tượng chúng ta đang đọc câu này: "The agent learns because **it** is intelligent." (Agent học vì **nó** thông minh). Khi chúng ta đọc từ "**it**" được in đậm, để hiểu nó chỉ đến cái gì, não của chúng ta vô thức đặt nhiều chú ý hơn vào từ "agent" ở phía trước câu. Cơ chế **Tự Chú ý (Self-Attention)** là một mô hình hóa toán học của hiện tượng này. Nó cho phép mô hình xem xét tất cả các từ khác trong câu khi xử lý mỗi từ và gán các "trọng số chú ý" khác nhau cho những từ này. Trọng số của một từ càng cao thì mối liên hệ của nó với từ hiện tại càng mạnh, và thông tin của nó nên chiếm tỷ trọng càng lớn trong biểu diễn của từ hiện tại.

Để triển khai quá trình trên, cơ chế tự chú ý đưa vào ba vai trò có thể học được cho mỗi vector token đầu vào:

- **Query (Q) - Truy vấn**: Đại diện cho token hiện tại, đang chủ động "truy vấn" các token khác để lấy thông tin.
- **Key (K) - Khóa**: Đại diện cho "nhãn" hoặc "chỉ mục" của các token trong câu có thể được truy vấn.
- **Value (V) - Giá trị**: Đại diện cho "nội dung" hoặc "thông tin" mà bản thân token mang theo.

Ba vector này đều thu được bằng cách nhân vector word embedding gốc với ba ma trận trọng số khác nhau, có thể học được ($W^Q,W^K,W^V$). Toàn bộ quá trình tính toán có thể được chia thành các bước sau, mà chúng ta có thể hình dung như một bài thi mở sách hiệu quả:

- Chuẩn bị "đề thi" và "tài liệu": Đối với mỗi từ trong câu, tạo ra các vector $Q,K,V$ của nó thông qua các ma trận trọng số.
- Tính điểm liên quan: Để tính biểu diễn mới của từ $A$, dùng vector $Q$ của từ $A$ thực hiện phép nhân điểm (dot product) với các vector $K$ của tất cả các từ trong câu (bao gồm cả chính $A$). Điểm số này phản ánh tầm quan trọng của các từ khác đối với việc hiểu từ $A$.
- Ổn định hóa và chuẩn hóa: Chia tất cả các điểm số thu được cho một hệ số tỷ lệ $\sqrt{d_{k}}$ ($d_{k}$ là số chiều của vector $K$) để tránh gradient quá nhỏ, sau đó dùng hàm Softmax để chuyển các điểm số thành các trọng số có tổng bằng 1, đây chính là quá trình chuẩn hóa.
- Tổng có trọng số: Nhân các trọng số thu được ở bước trước với vector $V$ tương ứng của mỗi từ, sau đó cộng tất cả các kết quả lại. Vector cuối cùng chính là biểu diễn mới của từ $A$ sau khi tích hợp thông tin ngữ cảnh toàn cục.

Quá trình này có thể được tóm tắt bằng một công thức ngắn gọn:

$$\text{Attention}(Q,K,V)=\text{softmax}\left(\frac{QK^{T}}{\sqrt{d_{k}}}\right)V$$

Nếu chỉ thực hiện một phép tính chú ý (tức là đơn đầu - single-head), mô hình có thể chỉ học được cách tập trung vào một loại mối liên hệ. Ví dụ, khi xử lý "it," nó có thể chỉ học được cách tập trung vào chủ ngữ. Nhưng các mối quan hệ trong ngôn ngữ rất phức tạp, và chúng ta muốn mô hình đồng thời tập trung vào nhiều mối quan hệ (như quan hệ chỉ định, quan hệ thì, quan hệ phụ thuộc, v.v.). Cơ chế chú ý đa đầu (multi-head attention) đã ra đời. Ý tưởng của nó rất đơn giản: thay vì làm tất cả cùng một lúc, hãy chia thành nhiều nhóm, làm riêng biệt, rồi hợp nhất lại.

Nó chia các vector Q, K, V gốc thành h phần theo chiều (h là số "đầu" - head), và mỗi phần thực hiện độc lập một phép tính chú ý đơn đầu. Điều này giống như có h "chuyên gia" khác nhau xem xét câu từ các góc độ khác nhau, mỗi chuyên gia nắm bắt một mối quan hệ đặc trưng khác nhau. Cuối cùng, "ý kiến" (tức là các vector đầu ra) của h chuyên gia này được nối lại (concatenate), rồi được tích hợp thông qua một phép biến đổi tuyến tính để thu được đầu ra cuối cùng.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-4.png" alt="Figure description" width="50%"/>
  <p>Hình 3.5 Cơ chế chú ý đa đầu</p>
</div>

Như minh họa trong Hình 3.5, thiết kế này cho phép mô hình đồng thời chú ý đến thông tin từ các vị trí khác nhau và các không gian con biểu diễn (representation subspace) khác nhau, tăng cường đáng kể năng lực biểu đạt của mô hình. Dưới đây là một triển khai đơn giản của chú ý đa đầu để tham khảo.

```Python
class MultiHeadAttention(nn.Module):
    """
    Module cơ chế chú ý đa đầu
    """
    def __init__(self, d_model, num_heads):
        super(MultiHeadAttention, self).__init__()
        assert d_model % num_heads == 0, "d_model must be divisible by num_heads"

        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads

        # Định nghĩa các lớp biến đổi tuyến tính cho Q, K, V và đầu ra
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        self.W_o = nn.Linear(d_model, d_model)

    def scaled_dot_product_attention(self, Q, K, V, mask=None):
        # 1. Tính điểm chú ý (QK^T)
        attn_scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.d_k)

        # 2. Áp dụng mặt nạ (nếu được cung cấp)
        # Đặt các vị trí có mask bằng 0 thành một số âm rất nhỏ, để chúng tiến về 0 sau softmax
        if mask is not None:
            attn_scores = attn_scores.masked_fill(mask == 0, -1e9)

        # 3. Tính trọng số chú ý (Softmax)
        attn_probs = torch.softmax(attn_scores, dim=-1)

        # 4. Tổng có trọng số (weights * V)
        output = torch.matmul(attn_probs, V)
        return output

    def split_heads(self, x):
        # Biến đổi shape của đầu vào x từ (batch_size, seq_length, d_model)
        # sang (batch_size, num_heads, seq_length, d_k)
        batch_size, seq_length, d_model = x.size()
        return x.view(batch_size, seq_length, self.num_heads, self.d_k).transpose(1, 2)

    def combine_heads(self, x):
        # Biến đổi shape của đầu vào x từ (batch_size, num_heads, seq_length, d_k)
        # trở lại (batch_size, seq_length, d_model)
        batch_size, num_heads, seq_length, d_k = x.size()
        return x.transpose(1, 2).contiguous().view(batch_size, seq_length, self.d_model)

    def forward(self, Q, K, V, mask=None):
        # 1. Thực hiện các phép biến đổi tuyến tính trên Q, K, V
        Q = self.split_heads(self.W_q(Q))
        K = self.split_heads(self.W_k(K))
        V = self.split_heads(self.W_v(V))

        # 2. Tính chú ý theo tích điểm có tỷ lệ (scaled dot-product attention)
        attn_output = self.scaled_dot_product_attention(Q, K, V, mask)

        # 3. Kết hợp các đầu ra đa đầu và thực hiện biến đổi tuyến tính cuối cùng
        output = self.W_o(self.combine_heads(attn_output))
        return output
```

**(3) Mạng Nơ-ron Truyền thẳng**

Trong mỗi lớp Encoder và Decoder, sau lớp con tự chú ý đa đầu là một **Mạng Truyền thẳng theo Vị trí (Position-wise Feed-Forward Network - FFN)**. Nếu vai trò của lớp chú ý là "tổng hợp động" (dynamically aggregate) thông tin liên quan từ toàn bộ chuỗi, thì vai trò của mạng truyền thẳng là trích xuất các đặc trưng bậc cao hơn từ thông tin đã được tổng hợp này.

Điểm mấu chốt của cái tên này là "theo vị trí" (position-wise). Nó có nghĩa là mạng truyền thẳng này tác động độc lập lên mỗi vector token trong chuỗi. Nói cách khác, đối với một chuỗi có độ dài `seq_len`, FFN này thực tế được gọi `seq_len` lần, mỗi lần xử lý một token. Quan trọng là, tất cả các vị trí đều chia sẻ cùng một bộ trọng số mạng. Thiết kế này vừa duy trì khả năng xử lý độc lập từng vị trí, vừa giảm đáng kể số lượng tham số của mô hình. Cấu trúc của mạng này rất đơn giản, bao gồm hai phép biến đổi tuyến tính và một hàm kích hoạt ReLU:

$$\mathrm{FFN}(x)=\max\left(0, xW_{1}+b_{1}\right) W_{2}+b_{2}$$

Trong đó $x$ là đầu ra của lớp con chú ý. $W_1,b_1,W_2,b_2$ là các tham số có thể học được. Thông thường, số chiều đầu ra `d_ff` của lớp tuyến tính thứ nhất lớn hơn nhiều so với số chiều đầu vào `d_model` (ví dụ, `d_ff = 4 * d_model`), sau đó qua kích hoạt ReLU, nó được ánh xạ trở lại về số chiều `d_model` thông qua lớp tuyến tính thứ hai. Thiết kế "mở rộng rồi thu nhỏ" này được cho là giúp mô hình học được các biểu diễn đặc trưng phong phú hơn.

Trong bộ khung PyTorch của chúng ta, có thể triển khai module này với đoạn mã sau:

```Python
class PositionWiseFeedForward(nn.Module):
    """
    Module mạng truyền thẳng theo vị trí
    """
    def __init__(self, d_model, d_ff, dropout=0.1):
        super(PositionWiseFeedForward, self).__init__()
        self.linear1 = nn.Linear(d_model, d_ff)
        self.dropout = nn.Dropout(dropout)
        self.linear2 = nn.Linear(d_ff, d_model)
        self.relu = nn.ReLU()

    def forward(self, x):
        # shape của x: (batch_size, seq_len, d_model)
        x = self.linear1(x)
        x = self.relu(x)
        x = self.dropout(x)
        x = self.linear2(x)
        # shape đầu ra cuối cùng: (batch_size, seq_len, d_model)
        return x
```

**(4) Kết nối Phần dư và Chuẩn hóa Lớp**

Trong mỗi lớp encoder và decoder của Transformer, tất cả các module con (như chú ý đa đầu và mạng truyền thẳng) đều được bọc bởi một thao tác `Add & Norm`. Sự kết hợp này đảm bảo rằng Transformer có thể huấn luyện một cách ổn định.

Thao tác này gồm hai phần:

- **Kết nối Phần dư (Residual Connection - Add)**: Thao tác này cộng trực tiếp đầu vào `x` của module con với đầu ra `Sublayer(x)` của module con. Cấu trúc này giải quyết vấn đề **Tiêu biến Gradient (Vanishing Gradients)** trong các mạng nơ-ron sâu. Trong quá trình lan truyền ngược, gradient có thể đi vòng qua module con và lan truyền tiến về phía trước một cách trực tiếp, đảm bảo rằng ngay cả khi mạng có nhiều lớp, mô hình vẫn có thể được huấn luyện hiệu quả. Công thức của nó có thể được biểu diễn là: $\text{Output} = x + \text{Sublayer}(x)$.
- **Chuẩn hóa Lớp (Layer Normalization - Norm)**: Thao tác này chuẩn hóa tất cả các đặc trưng của một mẫu đơn lẻ, làm cho giá trị trung bình của nó bằng 0 và phương sai bằng 1. Điều này giải quyết vấn đề **Dịch chuyển Hiệp phương sai Nội bộ (Internal Covariate Shift)** trong quá trình huấn luyện mô hình, giữ cho phân phối đầu vào của mỗi lớp ổn định, từ đó đẩy nhanh sự hội tụ của mô hình và cải thiện tính ổn định của quá trình huấn luyện.

**3.1.2.5 Mã hóa Vị trí**

Chúng ta đã hiểu rằng cốt lõi của Transformer là cơ chế tự chú ý, cơ chế này nắm bắt các phụ thuộc bằng cách tính toán mối quan hệ giữa hai token bất kỳ trong một chuỗi. Tuy nhiên, phương pháp tính toán này có một vấn đề cố hữu: nó không chứa bất kỳ thông tin nào về thứ tự hay vị trí của token. Đối với tự chú ý, hai chuỗi "agent learns" và "learns agent" là hoàn toàn tương đương, bởi vì nó chỉ quan tâm đến mối quan hệ giữa các token và bỏ qua cách sắp xếp của chúng. Để giải quyết vấn đề này, Transformer đã đưa vào **Mã hóa Vị trí (Positional Encoding)**.

Ý tưởng cốt lõi của mã hóa vị trí là thêm vào mỗi vector token embedding trong chuỗi đầu vào một "vector vị trí" bổ sung biểu thị thông tin vị trí tuyệt đối và tương đối của nó. Vector vị trí này không phải được học mà được tính trực tiếp thông qua một công thức toán học cố định. Bằng cách này, ngay cả khi hai token (ví dụ, hai token đều là `agent`) có cùng embedding, nhưng vì chúng ở các vị trí khác nhau trong câu, nên các vector mà chúng cuối cùng đưa vào mô hình Transformer sẽ trở nên độc nhất do được cộng thêm các mã hóa vị trí khác nhau. Mã hóa vị trí được đề xuất trong bài báo gốc sử dụng các hàm sin và cosin để tạo ra, với công thức như sau:

$$PE_{(pos,2i)}=\sin\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right)，$$

$$PE_{(pos,2i+1)}=\cos\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right)$$

Trong đó:

- $pos$ là vị trí của token trong chuỗi (ví dụ, $0$, $1$, $2$, ...)
- $i$ là chỉ số chiều trong vector vị trí (từ $0$ đến $d_{\text{model}}/2$)
- $d_{\text{model}}$ là số chiều của vector word embedding (nhất quán với những gì chúng ta đã định nghĩa trong mô hình)

Bây giờ, hãy triển khai module `PositionalEncoding` và hoàn thành phần cuối cùng của bộ khung mã Transformer của chúng ta.

```Python
class PositionalEncoding(nn.Module):
    """
    Thêm mã hóa vị trí vào các vector word embedding của chuỗi đầu vào.
    """
    def __init__(self, d_model: int, dropout: float = 0.1, max_len: int = 5000):
        super().__init__()
        self.dropout = nn.Dropout(p=dropout)

        # Tạo một ma trận mã hóa vị trí đủ dài
        position = torch.arange(max_len).unsqueeze(1)
        div_term = torch.exp(torch.arange(0, d_model, 2) * (-math.log(10000.0) / d_model))

        # kích thước của pe (positional encoding) là (max_len, d_model)
        pe = torch.zeros(max_len, d_model)

        # Các chiều chẵn dùng sin, các chiều lẻ dùng cos
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)

        # Đăng ký pe làm buffer, để nó không bị coi là tham số mô hình nhưng sẽ di chuyển cùng mô hình (ví dụ, to(device))
        self.register_buffer('pe', pe.unsqueeze(0))

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        # x.size(1) là độ dài chuỗi đầu vào hiện tại
        # Thêm mã hóa vị trí vào vector đầu vào
        x = x + self.pe[:, :x.size(1)]
        return self.dropout(x)
```

Phần này chủ yếu giúp hiểu cấu trúc vĩ mô của Transformer và các chi tiết vận hành của từng module bên trong. Vì mục đích là bổ sung hệ thống kiến thức về mô hình lớn trong việc học Agent, nên chúng ta sẽ không tiếp tục triển khai sâu hơn. Đến đây, chúng ta đã đặt một nền tảng kiến trúc vững chắc để hiểu các mô hình ngôn ngữ lớn hiện đại. Trong phần tiếp theo, chúng ta sẽ khám phá kiến trúc Decoder-Only và xem nó đã tiến hóa như thế nào dựa trên các ý tưởng của Transformer.

### 3.1.3 Kiến trúc Decoder-Only

Trong phần trước, chúng ta đã tự tay xây dựng một mô hình Transformer hoàn chỉnh, mô hình này thể hiện xuất sắc trong nhiều kịch bản đầu-cuối. Nhưng khi nhiệm vụ chuyển sang việc xây dựng một mô hình tổng quát có thể trò chuyện với con người, sáng tạo, và đóng vai trò là bộ não của một Agent, thì có lẽ chúng ta không cần một cấu trúc phức tạp như vậy.

Triết lý thiết kế của Transformer là "hiểu trước, tạo ra sau." Encoder chịu trách nhiệm hiểu sâu toàn bộ câu đầu vào, hình thành một bộ nhớ ngữ cảnh chứa thông tin toàn cục, sau đó decoder tạo ra bản dịch dựa trên bộ nhớ này. Nhưng khi OpenAI phát triển **GPT (Generative Pre-trained Transformer)**, họ đã đề xuất một ý tưởng đơn giản hơn<sup>[5]</sup>: **Chẳng phải nhiệm vụ cốt lõi của ngôn ngữ là dự đoán từ có khả năng xuất hiện tiếp theo cao nhất hay sao?**

Dù là trả lời câu hỏi, viết truyện, hay tạo mã, về bản chất đều là việc thêm nội dung hợp lý nhất từng từ một vào sau một chuỗi văn bản có sẵn. Dựa trên ý tưởng này, GPT đã thực hiện một sự đơn giản hóa táo bạo: **Nó hoàn toàn từ bỏ encoder và chỉ giữ lại phần decoder.** Đây chính là nguồn gốc của kiến trúc **Decoder-Only**.

Chế độ làm việc của kiến trúc Decoder-Only được gọi là **Tự Hồi quy (Autoregressive)**. Thuật ngữ nghe có vẻ chuyên môn này thực ra mô tả một quá trình rất đơn giản:

1. Cung cấp cho mô hình một đoạn văn bản khởi đầu (ví dụ, "Datawhale Agent is").
2. Mô hình dự đoán từ có khả năng xuất hiện tiếp theo cao nhất (ví dụ, "a").
3. Mô hình thêm từ "a" mà nó vừa tạo ra vào cuối văn bản đầu vào, tạo thành một đầu vào mới ("Datawhale Agent is a").
4. Dựa trên đầu vào mới này, mô hình lại dự đoán từ tiếp theo (ví dụ, "powerful").
5. Liên tục lặp lại quá trình này cho đến khi một câu hoàn chỉnh được tạo ra hoặc đạt đến điều kiện dừng.

Mô hình giống như đang chơi trò "nối từ," không ngừng "xem lại" nội dung mà nó đã viết, rồi suy nghĩ xem từ tiếp theo nên là gì.

Bạn có thể hỏi: trong quá trình huấn luyện, mô hình thường được cung cấp toàn bộ chuỗi văn bản cùng một lúc, vậy làm thế nào để đảm bảo rằng khi học dự đoán token tiếp theo, nó không "nhìn trộm" đáp án ở phía sau?

Câu trả lời là **Tự Chú ý có Mặt nạ (Masked Self-Attention)**. Trong kiến trúc Decoder-Only, cơ chế này trở nên cực kỳ quan trọng. Nguyên lý hoạt động của nó rất khéo léo:

Trong quá trình huấn luyện, mặc dù cả một chuỗi văn bản có thể được đưa vào mô hình một cách song song, nhưng sau khi cơ chế tự chú ý tính ma trận điểm chú ý (tức là điểm chú ý của mỗi từ đối với tất cả các từ khác) và trước khi chuẩn hóa Softmax, mô hình áp dụng một mặt nạ nhân quả (causal mask). Mặt nạ này thay thế các điểm số tương ứng với tất cả các token đứng sau vị trí hiện tại bằng một số âm rất lớn. Khi ma trận này đi qua Softmax, xác suất tại những vị trí đó trở thành 0. Bằng cách này, khi mô hình tính biểu diễn tại bất kỳ vị trí nào, về mặt toán học nó bị ngăn không cho chú ý đến thông tin sau vị trí đó.

Trong quá trình tạo sinh, tình huống còn trực tiếp hơn: các token tương lai chưa được tạo ra, nên mô hình chỉ có thể sử dụng nội dung đã tạo ra làm ngữ cảnh và dự đoán token tiếp theo từng bước một. Tự chú ý có mặt nạ giữ cho mục tiêu huấn luyện nhất quán với quá trình tạo sinh tự hồi quy, đảm bảo rằng mô hình luôn chỉ dựa vào thông tin trước vị trí hiện tại.

**Ưu điểm của Kiến trúc Decoder-Only**

Kiến trúc có vẻ đơn giản này đã mang lại thành công to lớn, với các ưu điểm bao gồm:

- **Mục tiêu Huấn luyện Thống nhất**: Nhiệm vụ duy nhất của mô hình là "dự đoán từ tiếp theo," một mục tiêu đơn giản rất phù hợp cho việc tiền huấn luyện (pre-training) trên dữ liệu văn bản không gán nhãn khổng lồ.
- **Cấu trúc Đơn giản, Dễ Mở rộng**: Ít thành phần hơn đồng nghĩa với việc dễ mở rộng quy mô hơn. Các mô hình khổng lồ ngày nay như GPT-4, Llama, và các mô hình khác với hàng trăm tỷ hoặc thậm chí hàng nghìn tỷ tham số đều dựa trên kiến trúc gọn gàng này.
- **Phù hợp Tự nhiên với Nhiệm vụ Tạo sinh**: Chế độ làm việc tự hồi quy của nó phù hợp hoàn hảo với tất cả các nhiệm vụ tạo sinh (đối thoại, viết lách, tạo mã, v.v.), đây cũng là lý do cốt lõi khiến nó có thể trở thành nền tảng để xây dựng các Agent tổng quát.

Tóm lại, kiến trúc Decoder-Only đã tiến hóa từ decoder của Transformer, thông qua mô hình đơn giản là "dự đoán từ tiếp theo," đã mở ra kỷ nguyên mô hình ngôn ngữ lớn mà chúng ta đang sống ngày nay.

## 3.2 Tương tác với Mô hình Ngôn ngữ Lớn

### 3.2.1 Kỹ thuật Prompt (Prompt Engineering)

Nếu chúng ta ví mô hình ngôn ngữ lớn như một "bộ não" cực kỳ tài năng, thì **Prompt** chính là ngôn ngữ mà chúng ta dùng để giao tiếp với "bộ não" này. Kỹ thuật prompt (prompt engineering) là ngành nghiên cứu cách thiết kế các prompt chính xác để dẫn dắt mô hình tạo ra các phản hồi mà chúng ta mong đợi. Đối với việc xây dựng Agent, một prompt được thiết kế cẩn thận có thể làm cho sự cộng tác và phân công giữa các Agent trở nên hiệu quả.

**(1) Các Tham số Lấy mẫu của Mô hình**

Khi sử dụng các mô hình lớn, bạn thường thấy các tham số có thể cấu hình như `Temperature`. Bản chất của chúng là điều chỉnh chiến lược lấy mẫu (sampling strategy) đối với "phân phối xác suất" của mô hình để phù hợp với nhu cầu của các kịch bản cụ thể. Cấu hình các tham số phù hợp có thể cải thiện hiệu năng của Agent trong các kịch bản cụ thể.

Phân phối xác suất truyền thống được tính bằng công thức Softmax: $p_i = \frac{e^{z_i}}{\sum_{j=1}^k e^{z_j}}$. Bản chất của các tham số lấy mẫu là "điều chỉnh lại" hoặc "cắt bớt" phân phối dựa trên các chiến lược khác nhau, từ đó thay đổi token tiếp theo mà mô hình lớn xuất ra.

`Temperature`: Nhiệt độ là một tham số then chốt kiểm soát tính "ngẫu nhiên" và "xác định" của đầu ra mô hình. Nguyên lý của nó là đưa vào một hệ số nhiệt độ $T\gt0$, viết lại Softmax thành $p_i^{(T)} = \frac{e^{z_i / T}}{\sum_{j=1}^k e^{z_j / T}}$.

Khi T giảm, phân phối trở nên "dốc hơn," trọng số của các mục có xác suất cao được khuếch đại thêm, tạo ra văn bản "bảo thủ" hơn với tỷ lệ lặp lại cao hơn. Khi T tăng, phân phối trở nên "phẳng hơn," trọng số của các mục có xác suất thấp tăng lên, tạo ra nội dung "đa dạng" hơn nhưng có thể thiếu mạch lạc.

- Nhiệt độ thấp (0 $\leqslant$ Temperature $\lt$ 0.3): Đầu ra "chính xác, xác định" hơn. Kịch bản áp dụng: Các nhiệm vụ dựa trên sự thật (factual): như hỏi đáp, tính toán dữ liệu, tạo mã; Các kịch bản đòi hỏi chặt chẽ: diễn giải văn bản pháp lý, viết tài liệu kỹ thuật, giải thích khái niệm học thuật, v.v.

- Nhiệt độ trung bình (0.3 $\leqslant$ Temperature $\lt$ 0.7): Đầu ra "cân bằng, tự nhiên." Kịch bản áp dụng: Trò chuyện hàng ngày: như tương tác chăm sóc khách hàng, chatbot; Sáng tạo thông thường: như viết email, viết nội dung quảng cáo sản phẩm, sáng tác truyện đơn giản.

- Nhiệt độ cao (0.7 $\leqslant$ Temperature $\lt$ 2): Đầu ra "sáng tạo, phân kỳ." Kịch bản áp dụng: Các nhiệm vụ sáng tạo: như sáng tác thơ, hình thành ý tưởng truyện khoa học viễn tưởng, động não khẩu hiệu quảng cáo, cảm hứng nghệ thuật; Tư duy phân kỳ (divergent thinking).

`Top-k`: Nguyên lý của nó là sắp xếp tất cả các token theo xác suất từ cao đến thấp, lấy k token đầu tiên để tạo thành một "tập ứng viên," sau đó "chuẩn hóa" xác suất của k token đã lọc: $ \hat{p}_i = \frac{p_i}{\sum_{j \in \text{candidate set}} p_j}$

- Sự khác biệt và liên hệ với lấy mẫu theo nhiệt độ: Lấy mẫu theo nhiệt độ điều chỉnh phân phối xác suất của tất cả các token (mượt hoặc dốc) thông qua nhiệt độ T, mà không thay đổi số lượng token ứng viên (vẫn xem xét toàn bộ N). Lấy mẫu Top-k giới hạn số lượng token ứng viên (chỉ giữ lại k token có xác suất cao nhất) thông qua giá trị k, sau đó lấy mẫu từ chúng. Khi k=1, đầu ra hoàn toàn xác định, thoái hóa thành "lấy mẫu tham lam" (greedy sampling).

`Top-p`: Nguyên lý của nó là sắp xếp tất cả các token theo xác suất từ cao đến thấp, bắt đầu từ token đầu tiên sau khi sắp xếp, dần dần tích lũy xác suất cho đến khi tổng tích lũy lần đầu tiên đạt hoặc vượt ngưỡng p: $\sum_{i \in S} p_{(i)} \geq p$. Tại thời điểm này, tất cả các token được đưa vào trong quá trình tích lũy tạo thành "tập nhân" (nucleus set), và cuối cùng tập nhân được chuẩn hóa.

- Sự khác biệt và liên hệ với Top-k: So với Top-k có kích thước cắt cố định, Top-p có thể thích ứng động với đặc điểm "đuôi dài" (long tail) của các phân phối khác nhau, có khả năng thích ứng tốt hơn với các trường hợp cực đoan của phân phối xác suất không đồng đều.

Trong quá trình tạo sinh văn bản, khi Top-p, Top-k, và hệ số nhiệt độ được thiết lập đồng thời, các tham số này phối hợp với nhau theo kiểu lọc phân tầng, với thứ tự ưu tiên: điều chỉnh nhiệt độ → Top-k → Top-p. Nhiệt độ điều chỉnh độ dốc tổng thể của phân phối, Top-k trước tiên giữ lại k ứng viên có xác suất cao nhất, sau đó Top-p chọn tập nhỏ nhất có xác suất tích lũy ≥ p từ kết quả Top-k làm tập ứng viên cuối cùng. Tuy nhiên, thông thường chỉ cần chọn một trong hai Top-k hoặc Top-p là đủ; nếu cả hai đều được thiết lập, tập ứng viên thực tế là giao của cả hai.
Lưu ý rằng nếu nhiệt độ được đặt bằng 0, thì Top-k và Top-p trở nên không liên quan vì Token có khả năng nhất sẽ là Token được dự đoán tiếp theo; nếu Top-k được đặt bằng 1, thì nhiệt độ và Top-p cũng trở nên không liên quan vì chỉ có một Token vượt qua tiêu chí Top-k và nó sẽ là Token được dự đoán tiếp theo.

**(2) Prompt Zero-shot, One-shot, và Few-shot**

Dựa trên số lượng ví dụ (Exemplars) mà chúng ta cung cấp cho mô hình, prompt có thể được chia thành ba loại. Để hiểu rõ hơn, hãy dùng một nhiệm vụ phân loại cảm xúc làm ví dụ, với mục tiêu là để mô hình phán đoán sắc thái cảm xúc của một đoạn văn bản (như tích cực, tiêu cực, hoặc trung tính).

**Prompt Zero-shot** Điều này có nghĩa là chúng ta không cung cấp cho mô hình bất kỳ ví dụ nào và yêu cầu trực tiếp nó hoàn thành nhiệm vụ dựa trên chỉ thị. Điều này được hưởng lợi từ khả năng tổng quát hóa mạnh mẽ mà mô hình có được sau khi tiền huấn luyện trên dữ liệu khổng lồ.

Trường hợp: Chúng ta trực tiếp đưa chỉ thị cho mô hình, yêu cầu nó hoàn thành nhiệm vụ phân loại cảm xúc.

```Python
Text: Datawhale's AI Agent course is excellent!
Sentiment: Positive
```

**Prompt One-shot** Chúng ta cung cấp cho mô hình một ví dụ hoàn chỉnh, cho nó thấy định dạng nhiệm vụ và phong cách đầu ra mong đợi.

Trường hợp: Trước tiên chúng ta đưa cho mô hình một cặp "câu hỏi-câu trả lời" hoàn chỉnh làm minh họa, sau đó đặt câu hỏi mới của chúng ta.

```Python
Text: This restaurant's service is too slow.
Sentiment: Negative

Text: Datawhale's AI Agent course is excellent!
Sentiment:
```

Mô hình sẽ bắt chước định dạng của ví dụ đã cho và hoàn thành với "Positive" cho đoạn văn bản thứ hai.

**Prompt Few-shot** Chúng ta cung cấp nhiều ví dụ, điều này cho phép mô hình hiểu chính xác hơn các chi tiết, ranh giới, và sắc thái tinh tế của nhiệm vụ, từ đó đạt hiệu năng tốt hơn.

Trường hợp: Chúng ta cung cấp nhiều ví dụ bao quát các tình huống khác nhau, cho phép mô hình có hiểu biết toàn diện hơn về nhiệm vụ.

```Python
Text: This restaurant's service is too slow.
Sentiment: Negative

Text: This movie's plot is very bland.
Sentiment: Neutral

Text: Datawhale's AI Agent course is excellent!
Sentiment:
```

Mô hình sẽ tổng hợp tất cả các ví dụ và phân loại chính xác hơn cảm xúc của câu cuối cùng là "Positive."

**(3) Tác động của Tinh chỉnh theo Chỉ thị (Instruction Tuning)**

Các mô hình GPT ban đầu (như GPT-3) chủ yếu là các mô hình "hoàn thành văn bản" (text completion); chúng giỏi việc viết tiếp văn bản dựa trên phần văn bản trước đó nhưng không nhất thiết giỏi việc hiểu và thực thi các chỉ thị của con người.

**Tinh chỉnh theo Chỉ thị (Instruction Tuning)** là một kỹ thuật tinh chỉnh (fine-tuning) sử dụng một lượng lớn dữ liệu ở định dạng "chỉ thị-câu trả lời" để huấn luyện thêm cho các mô hình đã được tiền huấn luyện. Sau khi tinh chỉnh theo chỉ thị, các mô hình có thể hiểu và tuân theo chỉ thị của người dùng tốt hơn. Tất cả các mô hình mà chúng ta sử dụng trong công việc và học tập hàng ngày ngày nay (như `ChatGPT`, `DeepSeek`, `Qwen`) đều là các mô hình đã được tinh chỉnh theo chỉ thị trong các dòng mô hình của chúng.

- **Prompt cho các mô hình "hoàn thành văn bản" (bạn cần dùng prompt few-shot để "dạy" mô hình phải làm gì):**

```Plain
This is a program that translates English to Chinese.
English: Hello
Chinese: 你好
English: How are you?
Chinese:
```

- **Prompt cho các mô hình "đã tinh chỉnh theo chỉ thị" (bạn có thể đưa chỉ thị trực tiếp):**

```Plain
Please translate the following English to Chinese:
How are you?
```

Sự xuất hiện của tinh chỉnh theo chỉ thị đã đơn giản hóa rất nhiều cách chúng ta tương tác với mô hình, làm cho việc dùng chỉ thị ngôn ngữ tự nhiên trực tiếp, rõ ràng trở nên khả thi.

**(4) Các Kỹ thuật Prompt Cơ bản**

**Đóng vai (Role-playing)** Bằng cách gán cho mô hình một vai trò cụ thể, chúng ta có thể dẫn dắt phong cách phản hồi, giọng điệu, và phạm vi kiến thức của nó, làm cho đầu ra của nó phù hợp hơn với nhu cầu của kịch bản cụ thể.

```Plain
# Trường hợp
You are now a senior Python programming expert. Please explain what GIL (Global Interpreter Lock) is in Python in a way that even a beginner can understand.
```

**Ví dụ trong Ngữ cảnh (In-context Example)** Điều này nhất quán với ý tưởng của prompt few-shot. Bằng cách cung cấp các ví dụ đầu vào-đầu ra rõ ràng trong prompt, chúng ta "dạy" mô hình cách xử lý các yêu cầu của mình, điều này đặc biệt hiệu quả khi xử lý các định dạng phức tạp hoặc các nhiệm vụ có phong cách cụ thể.

```Plain
# Trường hợp
I need you to extract product names and user sentiment from product reviews. Please output strictly in the JSON format below.

Review: The screen display of this "Stardust" laptop is amazing, but I don't really like its keyboard feel.
Output: {"product_name": "Stardust Laptop", "sentiment": "Mixed"}

Review: The "SoundMove" headphones I just bought have great sound quality, and the battery life exceeded my expectations!
Output:
```

**(5) Chuỗi Tư duy (Chain-of-Thought)**

Đối với các vấn đề phức tạp đòi hỏi suy luận logic, tính toán, hoặc tư duy nhiều bước, việc yêu cầu trực tiếp mô hình đưa ra câu trả lời thường dẫn đến sai sót. **Chuỗi Tư duy (Chain-of-Thought - CoT)** là một kỹ thuật prompt mạnh mẽ giúp cải thiện năng lực suy luận của mô hình trên các nhiệm vụ phức tạp bằng cách dẫn dắt nó "tư duy từng bước một."

Điểm mấu chốt để triển khai CoT là thêm một cụm từ dẫn dắt đơn giản vào trong prompt, chẳng hạn như "hãy suy nghĩ từng bước một" hoặc "Let's think step by step."

```Plain
# Prompt Chuỗi Tư duy
A basketball team won 60% of their 80 games in one season. In the next season, they played 15 games and won 12. What is the total winning percentage for both seasons?
Please think step by step and solve.

>>>
(Mô hình có thể xuất ra)
Okay, let's calculate step by step.
Step 1: Calculate the number of games won in the first season.
80 games * 60% = 48 games.
Step 2: Calculate the total number of games and total wins for both seasons.
Total games = 80 + 15 = 95 games.
Total wins = 48 + 12 = 60 games.
Step 3: Calculate the total winning percentage.
Total winning percentage = (Total wins / Total games) * 100% = (60 / 95) * 100% ≈ 63.16%.
So, the total winning percentage for both seasons is approximately 63.16%.
```

Bằng cách hiển thị rõ ràng quá trình suy luận của mình, mô hình không chỉ dễ dàng đi đến câu trả lời đúng hơn mà còn làm cho phản hồi của nó đáng tin cậy hơn và dễ để chúng ta kiểm tra, sửa chữa hơn.

### 3.2.2 Tách Token Văn bản (Text Tokenization)

Chúng ta biết rằng về bản chất, máy tính chỉ có thể hiểu các con số. Do đó, trước khi đưa văn bản ngôn ngữ tự nhiên cho các mô hình ngôn ngữ lớn, nó phải được chuyển đổi trước thành định dạng số mà mô hình có thể xử lý. Quá trình chuyển đổi chuỗi văn bản thành chuỗi số này được gọi là **Tách Token (Tokenization)**. Vai trò của một **Bộ tách token (Tokenizer)** là định nghĩa một bộ quy tắc để chia văn bản thô thành các đơn vị nhỏ nhất, mà chúng ta gọi là **Token**.

**3.2.2.1 Tại sao Cần Tách Token**

Các nhiệm vụ xử lý ngôn ngữ tự nhiên thời kỳ đầu có thể áp dụng các chiến lược tách token đơn giản:

-   **Dựa trên từ (Word-based)**: Chia trực tiếp các câu thành các từ bằng dấu cách hoặc dấu câu. Phương pháp này trực quan nhưng đối mặt với những thách thức đáng kể:
    -   **Bùng nổ Từ vựng và OOV**: Từ vựng của một ngôn ngữ là rất lớn. Nếu mỗi từ được coi là một token độc lập, bộ từ vựng trở nên khó quản lý. Tệ hơn, mô hình không thể xử lý bất kỳ từ nào không xuất hiện trong bộ từ vựng của nó (ví dụ, "DatawhaleAgent"). Hiện tượng này được gọi là vấn đề "Ngoài Từ vựng" ("Out-Of-Vocabulary" - OOV).
    -   **Thiếu Liên kết Ngữ nghĩa**: Mô hình gặp khó khăn trong việc nắm bắt các mối quan hệ ngữ nghĩa giữa các từ tương đồng về hình thái. Ví dụ, "look," "looks," và "looking" được coi là ba token hoàn toàn khác nhau, mặc dù chúng chia sẻ một ý nghĩa cốt lõi chung. Tương tự, ngữ nghĩa của các từ tần suất thấp trong dữ liệu huấn luyện không thể được học đầy đủ do chúng xuất hiện hiếm.

-   **Dựa trên ký tự (Character-based)**: Chia văn bản thành từng ký tự riêng lẻ. Phương pháp này có bộ từ vựng rất nhỏ (ví dụ, các chữ cái tiếng Anh, số, và dấu câu) và do đó tránh được vấn đề OOV. Tuy nhiên, nhược điểm của nó là hầu hết các ký tự riêng lẻ thiếu ý nghĩa ngữ nghĩa độc lập. Mô hình phải bỏ ra nhiều công sức hơn để học cách kết hợp các ký tự thành các từ có nghĩa, dẫn đến việc học kém hiệu quả.

Để cân bằng giữa kích thước bộ từ vựng và biểu đạt ngữ nghĩa, các mô hình ngôn ngữ lớn hiện đại áp dụng rộng rãi các thuật toán **Tách Token Từ con (Subword Tokenization)**. Ý tưởng cốt lõi là giữ các từ phổ biến (như "agent") làm các token đơn lẻ, hoàn chỉnh trong khi chia nhỏ các từ ít phổ biến (như "Tokenization") thành các mảnh từ con có nghĩa (chẳng hạn như "Token" và "ization"). Cách tiếp cận này không chỉ kiểm soát kích thước của bộ từ vựng mà còn cho phép mô hình hiểu và tạo ra các từ mới bằng cách kết hợp các từ con.

**3.2.2.2 Phân tích Thuật toán Byte-Pair Encoding**

Byte-Pair Encoding (BPE) là một trong những thuật toán tách token từ con chủ đạo nhất<sup>[6]</sup>, được các mô hình dòng GPT áp dụng. Ý tưởng cốt lõi của nó rất ngắn gọn và có thể được hiểu như một quá trình hợp nhất "tham lam" (greedy):

1. **Khởi tạo**: Khởi tạo bộ từ vựng bằng tất cả các ký tự cơ bản xuất hiện trong kho ngữ liệu.
2. **Hợp nhất Lặp**: Trong kho ngữ liệu, đếm tần suất của tất cả các cặp token liền kề, tìm cặp có tần suất cao nhất, hợp nhất chúng thành một token mới, và thêm nó vào bộ từ vựng.
3. **Lặp lại**: Lặp lại bước 2 cho đến khi kích thước bộ từ vựng đạt đến ngưỡng đã định trước.

**Minh họa Trường hợp:** Giả sử kho ngữ liệu nhỏ của chúng ta là `{"hug": 1, "pug": 1, "pun": 1, "bun": 1}`, và chúng ta muốn xây dựng một bộ từ vựng có kích thước 10. Quá trình huấn luyện BPE có thể được biểu diễn bằng Bảng 3.1:

<div align="center">
  <p>Bảng 3.1 Ví dụ về Quá trình Hợp nhất của Thuật toán BPE</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/3-figures/1757249275674-5.png" alt="Figure description" width="90%"/>
</div>

Sau khi quá trình huấn luyện kết thúc, khi kích thước bộ từ vựng đạt đến 10, chúng ta có được các quy tắc tách token mới. Bây giờ, đối với một từ chưa từng thấy là "bug," bộ tách token trước tiên sẽ kiểm tra xem "bug" có trong bộ từ vựng không và thấy là không có; sau đó kiểm tra "bu" và thấy không có; cuối cùng kiểm tra "b" và "ug," thấy cả hai đều có, và do đó chia nó thành `['b', 'ug']`.

Dưới đây chúng ta dùng một đoạn mã Python đơn giản để mô phỏng quá trình trên:

```Python
import re, collections

def get_stats(vocab):
    """Đếm tần suất các cặp token"""
    pairs = collections.defaultdict(int)
    for word, freq in vocab.items():
        symbols = word.split()
        for i in range(len(symbols)-1):
            pairs[symbols[i],symbols[i+1]] += freq
    return pairs

def merge_vocab(pair, v_in):
    """Hợp nhất các cặp token"""
    v_out = {}
    bigram = re.escape(' '.join(pair))
    p = re.compile(r'(?<!\S)' + bigram + r'(?!\S)')
    for word in v_in:
        w_out = p.sub(''.join(pair), word)
        v_out[w_out] = v_in[word]
    return v_out

# Chuẩn bị kho ngữ liệu, thêm </w> vào cuối mỗi từ để biểu thị kết thúc, và tách các ký tự
vocab = {'h u g </w>': 1, 'p u g </w>': 1, 'p u n </w>': 1, 'b u n </w>': 1}
num_merges = 4 # Đặt số lần hợp nhất

for i in range(num_merges):
    pairs = get_stats(vocab)
    if not pairs:
        break
    best = max(pairs, key=pairs.get)
    vocab = merge_vocab(best, vocab)
    print(f"Merge {i+1}: {best} -> {''.join(best)}")
    print(f"New vocabulary (partial): {list(vocab.keys())}")
    print("-" * 20)

>>>
Merge 1: ('u', 'g') -> ug
New vocabulary (partial): ['h ug </w>', 'p ug </w>', 'p u n </w>', 'b u n </w>']
--------------------
Merge 2: ('ug', '</w>') -> ug</w>
New vocabulary (partial): ['h ug</w>', 'p ug</w>', 'p u n </w>', 'b u n </w>']
--------------------
Merge 3: ('u', 'n') -> un
New vocabulary (partial): ['h ug</w>', 'p ug</w>', 'p un </w>', 'b un </w>']
--------------------
Merge 4: ('un', '</w>') -> un</w>
New vocabulary (partial): ['h ug</w>', 'p ug</w>', 'p un</w>', 'b un</w>']
--------------------
```

Đoạn mã này minh họa rõ ràng cách thuật toán BPE dần dần xây dựng và mở rộng bộ từ vựng bằng cách hợp nhất lặp các cặp token liền kề có tần suất cao nhất.

Nhiều thuật toán về sau là các tối ưu hóa dựa trên BPE. Trong số đó, WordPiece và SentencePiece do Google phát triển là hai thuật toán có ảnh hưởng nhất.

- **WordPiece**: Thuật toán được mô hình BERT của Google áp dụng<sup>[7]</sup>. Nó rất giống với BPE, nhưng tiêu chí để hợp nhất token không phải là "tần suất cao nhất" mà là "tối đa hóa sự cải thiện xác suất của mô hình ngôn ngữ trên kho ngữ liệu." Nói đơn giản, nó ưu tiên hợp nhất các cặp token có thể tối đa hóa sự cải thiện "độ trôi chảy" của toàn bộ kho ngữ liệu.
- **SentencePiece**: Một công cụ tách token mã nguồn mở của Google<sup>[8]</sup>, được các mô hình dòng Llama áp dụng. Đặc điểm lớn nhất của nó là coi dấu cách như các ký tự thông thường (thường được biểu diễn bằng dấu gạch dưới `_`). Điều này làm cho quá trình tách token và giải mã hoàn toàn thuận nghịch (reversible) và độc lập với ngôn ngữ cụ thể (ví dụ, nó không cần biết rằng tiếng Trung không dùng dấu cách để phân tách từ).

**3.2.2.3 Ý nghĩa của Bộ tách Token đối với Nhà phát triển**

Hiểu các chi tiết của thuật toán tách token không phải là mục tiêu, nhưng với tư cách là một nhà phát triển Agent, hiểu tác động thực tế của bộ tách token là điều quan trọng, vì nó liên quan trực tiếp đến hiệu năng, chi phí, và độ ổn định của Agent:

- **Giới hạn Cửa sổ Ngữ cảnh**: Cửa sổ ngữ cảnh của mô hình (như 8K, 128K) được tính theo **số lượng Token**, không phải số ký tự hay số từ. Cùng một văn bản có thể có số lượng Token khác nhau rất lớn ở các ngôn ngữ khác nhau (như tiếng Trung và tiếng Anh) hoặc với các bộ tách token khác nhau. Quản lý chính xác độ dài đầu vào và tránh vượt quá giới hạn ngữ cảnh là nền tảng để xây dựng các Agent có bộ nhớ dài hạn.
- **Chi phí API**: Hầu hết các API mô hình tính phí dựa trên số lượng Token. Hiểu cách văn bản của bạn sẽ được tách token là một bước then chốt trong việc ước tính và kiểm soát chi phí vận hành của Agent.
- **Các Bất thường về Hiệu năng Mô hình**: Đôi khi hành vi kỳ lạ của mô hình bắt nguồn từ việc tách token. Ví dụ, mô hình có thể giỏi tính `2 + 2` nhưng lại có thể sai với `2+2` (không có dấu cách), bởi vì cái sau có thể bị bộ tách token coi là một token độc lập, không phổ biến. Tương tự, một từ với chữ cái đầu được viết hoa khác nhau có thể bị chia thành các chuỗi Token hoàn toàn khác nhau, ảnh hưởng đến việc hiểu của mô hình. Việc cân nhắc những "cạm bẫy" này khi thiết kế prompt và phân tích cú pháp đầu ra của mô hình giúp cải thiện tính bền vững (robustness) của Agent.

### 3.2.3 Gọi Mô hình Ngôn ngữ Lớn Mã nguồn Mở

Trong Chương 1 của cuốn sách này, chúng ta đã tương tác với các mô hình ngôn ngữ lớn thông qua API để điều khiển các Agent của mình. Đây là một phương pháp nhanh chóng và tiện lợi, nhưng không phải là phương pháp duy nhất. Đối với nhiều kịch bản đòi hỏi xử lý dữ liệu nhạy cảm, vận hành ngoại tuyến (offline), hoặc kiểm soát chi phí tinh vi, việc triển khai các mô hình ngôn ngữ lớn trực tiếp tại chỗ (local) trở nên rất quan trọng.

**Hugging Face Transformers** là một thư viện mã nguồn mở mạnh mẽ cung cấp các giao diện tiêu chuẩn hóa để tải và sử dụng hàng chục nghìn mô hình đã được tiền huấn luyện. Chúng ta sẽ sử dụng nó để hoàn thành phần thực hành này.

**Cấu hình Môi trường và Lựa chọn Mô hình**: Để đảm bảo hầu hết độc giả có thể chạy trơn tru trên máy tính cá nhân, chúng tôi đã cố ý chọn một mô hình quy mô nhỏ nhưng mạnh mẽ: `Qwen/Qwen1.5-0.5B-Chat`. Đây là một mô hình đối thoại với khoảng 500 triệu tham số được mã nguồn mở bởi Học viện DAMO của Alibaba. Nó có kích thước nhỏ, hiệu năng xuất sắc, và rất phù hợp để học nhập môn và triển khai tại chỗ.

Trước tiên, hãy đảm bảo bạn đã cài đặt các thư viện cần thiết:

```Plain
pip install transformers torch
```

Trong thư viện `transformers`, chúng ta thường sử dụng các lớp `AutoModelForCausalLM` và `AutoTokenizer` để tự động tải các trọng số và bộ tách token khớp với mô hình. Đoạn mã sau sẽ tự động tải các tệp mô hình và cấu hình bộ tách token cần thiết từ Hugging Face Hub, việc này có thể mất một chút thời gian tùy thuộc vào tốc độ mạng của bạn.

```Python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

# Chỉ định ID mô hình
model_id = "Qwen/Qwen1.5-0.5B-Chat"

# Thiết lập thiết bị, ưu tiên GPU
device = "cuda" if torch.cuda.is_available() else "cpu"
print(f"Using device: {device}")

# Tải bộ tách token
tokenizer = AutoTokenizer.from_pretrained(model_id)

# Tải mô hình và chuyển nó sang thiết bị đã chỉ định
model = AutoModelForCausalLM.from_pretrained(model_id).to(device)

print("Model and tokenizer loaded!")
```

Hãy tạo một prompt đối thoại. Mô hình Qwen1.5-Chat tuân theo một mẫu đối thoại (dialogue template) cụ thể. Sau đó, chúng ta có thể sử dụng `tokenizer` đã tải ở bước trước để chuyển đổi prompt văn bản thành các ID số (tức là Token ID) mà mô hình có thể hiểu.

```Python
# Chuẩn bị đầu vào đối thoại
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello, please introduce yourself."}
]

# Dùng mẫu của tokenizer để định dạng đầu vào
text = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True
)

# Mã hóa văn bản đầu vào
model_inputs = tokenizer([text], return_tensors="pt").to(device)

print("Encoded input text:")
print(model_inputs)

>>>
{'input_ids': tensor([[151644, 8948, 198, 2610, 525, 264,  10950, 17847, 13,151645, 198, 151644, 872, 198, 108386, 37945, 100157, 107828,1773, 151645, 198, 151644, 77091, 198]], device='cuda:0'), 'attention_mask': tensor([[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]],
       device='cuda:0')}
```

Bây giờ chúng ta có thể gọi phương thức `generate()` của mô hình để tạo ra câu trả lời. Mô hình sẽ xuất ra một chuỗi Token ID đại diện cho câu trả lời của nó.

Cuối cùng, chúng ta cần sử dụng phương thức `decode()` của bộ tách token để dịch các ID số này trở lại thành văn bản mà con người có thể đọc được.

```Python
# Dùng mô hình để tạo câu trả lời
# max_new_tokens kiểm soát số lượng Token mới tối đa mà mô hình có thể tạo ra
generated_ids = model.generate(
    model_inputs.input_ids,
    max_new_tokens=512
)

# Cắt bỏ phần đầu vào khỏi các Token ID đã tạo
# Bằng cách này chúng ta chỉ giải mã phần mới do mô hình tạo ra
generated_ids = [
    output_ids[len(input_ids):] for input_ids, output_ids in zip(model_inputs.input_ids, generated_ids)
]

# Giải mã các Token ID đã tạo
response = tokenizer.batch_decode(generated_ids, skip_special_tokens=True)[0]

print("\nModel's answer:")
print(response)

>>>
My name is Tongyi Qianwen, a pre-trained language model developed by Alibaba Cloud. I can answer questions, create text, express opinions, and write code. My main functions are to provide help in multiple fields, including but not limited to: language understanding, text generation, machine translation, question-answering systems, etc. Is there anything I can help you with?
```

Sau khi chạy toàn bộ đoạn mã, bạn sẽ thấy phần giới thiệu về mô hình Qwen do mô hình tạo ra trên máy tính cá nhân của mình. Xin chúc mừng, bạn đã triển khai và chạy thành công một mô hình ngôn ngữ lớn mã nguồn mở tại chỗ!

### 3.2.4 Lựa chọn Mô hình

Trong phần trước, chúng ta đã chạy thành công một mô hình ngôn ngữ mã nguồn mở nhỏ tại chỗ. Điều này tự nhiên đặt ra một câu hỏi then chốt đối với các nhà phát triển Agent: trong bối cảnh hiện tại với hàng trăm mô hình đua nở, chúng ta nên chọn mô hình phù hợp nhất cho các nhiệm vụ cụ thể như thế nào?

Việc lựa chọn một mô hình ngôn ngữ không đơn giản là theo đuổi "lớn nhất, mạnh nhất" mà là một quá trình ra quyết định cân bằng giữa hiệu năng, chi phí, tốc độ, và phương thức triển khai. Phần này trước tiên sẽ sắp xếp một số yếu tố cân nhắc then chốt khi lựa chọn mô hình, sau đó điểm lại các mô hình mã nguồn đóng và mã nguồn mở chủ đạo hiện nay.

Vì công nghệ mô hình ngôn ngữ lớn đang ở giai đoạn phát triển nhanh chóng, với các mô hình và phiên bản mới liên tục xuất hiện và tốc độ lặp lại (iteration) cực kỳ nhanh, phần này cố gắng cung cấp một cái nhìn tổng quan về các mô hình chủ đạo hiện nay và các yếu tố cân nhắc khi lựa chọn tại thời điểm viết, nhưng độc giả nên lưu ý rằng các phiên bản mô hình cụ thể và dữ liệu hiệu năng được đề cập có thể thay đổi theo thời gian, và chỉ liệt kê một số công trình, không phải toàn diện. Chúng tôi tập trung nhiều hơn vào việc giới thiệu các đặc điểm kỹ thuật cốt lõi, xu hướng phát triển, và các nguyên tắc lựa chọn chung trong phát triển Agent.

**3.2.4.1 Các Yếu tố Cân nhắc Then chốt khi Lựa chọn Mô hình**

Khi lựa chọn một mô hình ngôn ngữ lớn cho Agent của bạn, bạn có thể đánh giá tổng hợp từ các khía cạnh sau:

- **Hiệu năng và Năng lực**: Đây là yếu tố cân nhắc cốt lõi. Các mô hình khác nhau giỏi ở các nhiệm vụ khác nhau; một số giỏi suy luận logic và tạo mã, trong khi những mô hình khác giỏi hơn về viết sáng tạo hoặc dịch đa ngôn ngữ. Bạn có thể tham khảo một số bảng xếp hạng benchmark công khai (như LMSys Chatbot Arena Leaderboard) để đánh giá năng lực tổng hợp của mô hình.
- **Chi phí**: Đối với các mô hình mã nguồn đóng, chi phí chủ yếu thể hiện ở phí gọi API, thường được tính theo số lượng Token. Đối với các mô hình mã nguồn mở, chi phí thể hiện ở phần cứng (GPU, bộ nhớ) và vận hành cần thiết cho việc triển khai tại chỗ. Cần đưa ra lựa chọn dựa trên mức sử dụng dự kiến và ngân sách của ứng dụng.
- **Tốc độ (Độ trễ - Latency)**: Đối với các Agent đòi hỏi tương tác thời gian thực (như chăm sóc khách hàng, NPC trong game), tốc độ phản hồi của mô hình là rất quan trọng. Một số mô hình nhẹ hoặc đã được tối ưu hóa (như GPT-3.5 Turbo, Claude 3.5 Sonnet) thể hiện tốt hơn về độ trễ.
- **Cửa sổ Ngữ cảnh**: Giới hạn trên của số lượng Token mà mô hình có thể xử lý cùng một lúc. Đối với các Agent cần hiểu các tài liệu dài, phân tích kho mã (code repository), hoặc duy trì bộ nhớ đối thoại dài hạn, việc chọn một mô hình có cửa sổ ngữ cảnh lớn hơn (như 128K Token hoặc cao hơn) là cần thiết.
- **Phương thức Triển khai**: Sử dụng API là đơn giản và tiện lợi nhất, nhưng dữ liệu cần được gửi đến bên thứ ba và phải tuân theo các điều khoản của nhà cung cấp dịch vụ. Triển khai tại chỗ có thể đảm bảo quyền riêng tư dữ liệu và mức độ tự chủ cao nhất, nhưng có yêu cầu cao hơn về kỹ thuật và phần cứng.
- **Hệ sinh thái và Chuỗi công cụ (Toolchain)**: Mức độ phổ biến của một mô hình cũng quyết định sự trưởng thành của hệ sinh thái xung quanh nó. Các mô hình chủ đạo thường có sự hỗ trợ cộng đồng, hướng dẫn, mô hình đã tiền huấn luyện, công cụ tinh chỉnh, và các framework phát triển tương thích (như LangChain, LlamaIndex, Hugging Face Transformers) phong phú hơn, điều này có thể đẩy nhanh đáng kể quá trình phát triển và giảm độ khó. Chọn một mô hình có cộng đồng năng động và chuỗi công cụ đầy đủ giúp dễ tìm giải pháp và tài nguyên hơn khi gặp vấn đề.
- **Khả năng Tinh chỉnh và Tùy biến**: Đối với các Agent cần xử lý dữ liệu chuyên ngành hoặc thực hiện các nhiệm vụ cụ thể, khả năng tinh chỉnh mô hình là rất quan trọng. Một số mô hình cung cấp các giao diện và công cụ tinh chỉnh tiện lợi, cho phép các nhà phát triển tùy biến việc huấn luyện bằng bộ dữ liệu của riêng họ, cải thiện đáng kể hiệu năng và độ chính xác của mô hình trong các kịch bản cụ thể. Các mô hình mã nguồn mở thường cung cấp tính linh hoạt lớn hơn về mặt này.
- **An toàn và Đạo đức**: Với việc ứng dụng rộng rãi các mô hình ngôn ngữ lớn, các rủi ro an toàn tiềm ẩn và các vấn đề đạo đức của chúng ngày càng nổi bật. Khi lựa chọn mô hình, hãy cân nhắc hiệu năng của chúng về mặt thiên kiến (bias), độc hại (toxicity), ảo giác (hallucination), v.v., và mức độ đầu tư của nhà cung cấp dịch vụ hoặc cộng đồng mã nguồn mở vào an toàn mô hình và AI có trách nhiệm. Đối với các ứng dụng hướng đến công chúng hoặc liên quan đến thông tin nhạy cảm, an toàn mô hình và tuân thủ đạo đức là những yếu tố cân nhắc không thể bỏ qua.

**3.2.4.2 Tổng quan về các Mô hình Mã nguồn Đóng**

Các mô hình mã nguồn đóng thường đại diện cho công nghệ AI tiên tiến nhất hiện nay và cung cấp các dịch vụ API ổn định, dễ sử dụng, khiến chúng trở thành lựa chọn hàng đầu để xây dựng các Agent hiệu năng cao.

1. **Dòng OpenAI GPT**: Từ GPT-3 mở ra kỷ nguyên mô hình lớn, đến ChatGPT giới thiệu RLHF (Học tăng cường từ Phản hồi của Con người - Reinforcement Learning from Human Feedback) và đạt được sự căn chỉnh (alignment) với ý định của con người, đến GPT-4 mở ra kỷ nguyên đa phương thức (multimodal), OpenAI liên tục dẫn dắt sự phát triển của ngành. Mô hình GPT-5 mới nhất tiếp tục nâng năng lực đa phương thức và trí tuệ tổng quát lên những tầm cao mới, xử lý liền mạch các đầu vào văn bản, âm thanh, và hình ảnh và tạo ra các đầu ra tương ứng, với tốc độ phản hồi và tính tự nhiên được cải thiện đáng kể, đặc biệt xuất sắc trong đối thoại giọng nói thời gian thực.
2. **Dòng Google Gemini**: Các mô hình dòng Gemini của Google DeepMind là đại diện cho tính đa phương thức bản địa (native multimodality), với đặc điểm cốt lõi là xử lý thống nhất nhiều phương thức bao gồm văn bản, mã, âm thanh/video, và hình ảnh, và có lợi thế trong việc xử lý thông tin khổng lồ với cửa sổ ngữ cảnh siêu dài. Gemini Ultra là mô hình mạnh nhất của nó, phù hợp cho các nhiệm vụ cực kỳ phức tạp; Gemini Pro phù hợp cho một loạt các nhiệm vụ rộng, cung cấp hiệu năng và hiệu quả cao; Gemini Nano được tối ưu hóa cho việc triển khai trên thiết bị. Các mô hình dòng Gemini 2.5 mới nhất, như Gemini 2.5 Pro và Gemini 2.5 Flash, tiếp tục cải thiện năng lực suy luận và cửa sổ ngữ cảnh, đặc biệt là Gemini 2.5 Flash với tốc độ suy luận nhanh hơn và hiệu quả về chi phí, phù hợp cho các kịch bản đòi hỏi phản hồi nhanh.
3. **Dòng Anthropic Claude**: Anthropic là một công ty tập trung vào an toàn AI và AI có trách nhiệm. Các mô hình dòng Claude của họ đã ưu tiên an toàn AI ngay từ giai đoạn thiết kế, nổi tiếng về độ tin cậy trong việc xử lý các tài liệu dài, giảm các đầu ra có hại, và tuân theo chỉ thị, được các ứng dụng doanh nghiệp đặc biệt ưa chuộng. Dòng Claude 3 bao gồm Claude 3 Opus (thông minh nhất, hiệu năng mạnh nhất), Claude 3 Sonnet (lựa chọn cân bằng giữa hiệu năng và tốc độ), và Claude 3 Haiku (mô hình nhanh nhất, nhỏ gọn nhất, phù hợp cho tương tác gần thời gian thực). Các mô hình dòng Claude 4 mới nhất, như Claude 4 Opus, đã có những tiến bộ đáng kể về trí tuệ tổng quát, suy luận phức tạp, và tạo mã, tiếp tục cải thiện năng lực trong việc xử lý ngữ cảnh dài và các nhiệm vụ đa phương thức.
4. **Các Mô hình Chủ đạo trong nước (Trung Quốc)**: Trung Quốc đã nổi lên với nhiều mô hình mã nguồn đóng có tính cạnh tranh trong lĩnh vực mô hình ngôn ngữ lớn, tiêu biểu là Baidu ERNIE Bot, Tencent Hunyuan, Huawei Pangu-α, iFlytek SparkDesk, và Moonshot AI. Những mô hình trong nước này có lợi thế tự nhiên trong xử lý tiếng Trung và trao quyền sâu sắc cho các ngành công nghiệp bản địa.

**3.2.4.3 Tổng quan về các Mô hình Mã nguồn Mở**

Các mô hình mã nguồn mở cung cấp cho các nhà phát triển mức độ linh hoạt, minh bạch, và tự chủ cao nhất, xúc tác cho một hệ sinh thái cộng đồng thịnh vượng. Chúng cho phép các nhà phát triển triển khai tại chỗ, thực hiện tinh chỉnh tùy biến, và có toàn quyền kiểm soát mô hình.

- **Dòng Meta Llama**: Dòng Llama của Meta là một cột mốc quan trọng trong các mô hình ngôn ngữ lớn mã nguồn mở. Dòng này đã trở thành nền tảng cho nhiều dự án phái sinh và nghiên cứu nhờ hiệu năng tổng hợp xuất sắc, thỏa thuận cấp phép mở, và sự hỗ trợ cộng đồng mạnh mẽ. Dòng Llama 4 được phát hành vào tháng 4 năm 2025, là những mô hình đầu tiên của Meta áp dụng kiến trúc Hỗn hợp Chuyên gia (Mixture of Experts - MoE), giúp cải thiện đáng kể hiệu quả tính toán bằng cách chỉ kích hoạt các phần của mô hình cần thiết để xử lý các nhiệm vụ cụ thể. Dòng này bao gồm ba mô hình có định vị khác biệt rõ rệt: Llama 4 Scout hỗ trợ cửa sổ ngữ cảnh 10 triệu token được thiết kế cho phân tích tài liệu dài và triển khai trên di động. Llama 4 Maverick tập trung vào năng lực đa phương thức, xuất sắc trong lập trình, suy luận phức tạp, và hỗ trợ đa ngôn ngữ. Llama 4 Behemoth vượt trội hơn các đối thủ cạnh tranh trong nhiều benchmark STEM và là mô hình mạnh nhất của Meta hiện nay.
- **Dòng Mistral AI**: Mistral AI đến từ Pháp nổi tiếng với thiết kế mô hình "kích thước nhỏ, hiệu năng cao." Mô hình mới nhất của họ là Mistral Medium 3.1 được phát hành vào tháng 8 năm 2025, với độ chính xác và tốc độ phản hồi được cải thiện đáng kể trong các nhiệm vụ như tạo mã, suy luận STEM, và hỏi đáp liên ngành, với hiệu năng benchmark vượt trội hơn Claude Sonnet 3.7 và Llama 4 Maverick cùng các mô hình tương tự khác. Nó có năng lực đa phương thức bản địa, có thể đồng thời xử lý các đầu vào hỗn hợp hình ảnh và văn bản, và có một "lớp thích ứng giọng điệu" (tone adaptation layer) tích hợp sẵn để giúp các doanh nghiệp dễ dàng hơn trong việc đạt được các đầu ra phù hợp với thương hiệu.
- **Các Lực lượng Mã nguồn Mở trong nước (Trung Quốc)**: Các nhà sản xuất và viện nghiên cứu trong nước cũng đang tích cực đón nhận mã nguồn mở, như dòng **Qwen (Tongyi Qianwen)** của Alibaba và dòng **ChatGLM** do Đại học Thanh Hoa hợp tác với Zhipu AI. Chúng cung cấp năng lực tiếng Trung mạnh mẽ và đã xây dựng các cộng đồng năng động xung quanh mình.

Đối với các nhà phát triển Agent, các mô hình mã nguồn đóng cung cấp sự tiện lợi "dùng ngay" (out-of-the-box), trong khi các mô hình mã nguồn mở trao cho chúng ta "sự tự do tùy biến." Hiểu các đặc điểm và các mô hình tiêu biểu của hai phe này là bước đầu tiên để đưa ra các lựa chọn kỹ thuật khôn ngoan cho các dự án Agent của chúng ta.

## 3.3 Định luật Tỷ lệ (Scaling Laws) và Các Giới hạn của Mô hình Ngôn ngữ Lớn

Các Mô hình Ngôn ngữ Lớn (LLM) đã đạt được những tiến bộ đáng chú ý trong những năm gần đây, với ranh giới năng lực không ngừng mở rộng và các kịch bản ứng dụng ngày càng phong phú. Tuy nhiên, đằng sau những thành tựu này là một sự hiểu biết sâu sắc về mối quan hệ giữa quy mô mô hình, lượng dữ liệu, và tài nguyên tính toán, cụ thể là **Định luật Tỷ lệ (Scaling Laws)**. Đồng thời, với tư cách là một công nghệ mới nổi, các LLM cũng đối mặt với nhiều thách thức và giới hạn. Phần này sẽ khám phá sâu những khái niệm cốt lõi này, nhằm giúp độc giả hiểu một cách toàn diện các ranh giới năng lực của LLM, từ đó phát huy điểm mạnh và tránh điểm yếu khi xây dựng Agent.

### 3.3.1 Định luật Tỷ lệ (Scaling Laws)

**Định luật Tỷ lệ (Scaling Laws)** là một trong những phát hiện quan trọng nhất trong lĩnh vực mô hình ngôn ngữ lớn những năm gần đây. Chúng tiết lộ rằng có những mối quan hệ lũy thừa (power-law) có thể dự đoán được giữa hiệu năng của mô hình với số lượng tham số của mô hình, lượng dữ liệu huấn luyện, và tài nguyên tính toán. Phát hiện này cung cấp sự chỉ dẫn lý thuyết cho sự phát triển liên tục của các mô hình ngôn ngữ lớn, làm rõ logic nền tảng rằng việc tăng đầu tư tài nguyên có thể cải thiện hiệu năng của mô hình một cách có hệ thống.

Nghiên cứu phát hiện rằng trong các hệ tọa độ log-log, hiệu năng của mô hình (thường được đo bằng Loss) thể hiện các mối quan hệ lũy thừa mượt mà với cả ba yếu tố: số lượng tham số, lượng dữ liệu, và lượng tính toán<sup>[9]</sup>. Nói đơn giản, chỉ cần chúng ta liên tục và tăng theo tỷ lệ ba yếu tố này, hiệu năng của mô hình sẽ cải thiện một cách có thể dự đoán và mượt mà mà không có các nút thắt cổ chai rõ ràng. Phát hiện này cung cấp sự chỉ dẫn rõ ràng cho việc thiết kế và huấn luyện các mô hình lớn: trong phạm vi ràng buộc về tài nguyên, hãy tối đa hóa quy mô mô hình và lượng dữ liệu huấn luyện càng nhiều càng tốt.

Nghiên cứu ban đầu tập trung nhiều hơn vào việc tăng số lượng tham số của mô hình, nhưng "Định luật Chinchilla" (Chinchilla Law) do DeepMind đề xuất vào năm 2022 đã đưa ra những hiệu chỉnh quan trọng<sup>[10]</sup>. Định luật này chỉ ra rằng dưới một ngân sách tính toán cho trước, để đạt được hiệu năng tối ưu, **tồn tại một tỷ lệ tối ưu giữa số lượng tham số của mô hình và lượng dữ liệu huấn luyện**. Cụ thể, các mô hình tối ưu nên nhỏ hơn so với những gì thường được tin trước đây nhưng cần được huấn luyện với nhiều dữ liệu hơn rất nhiều. Ví dụ, một mô hình Chinchilla 70 tỷ tham số, vì được huấn luyện với lượng dữ liệu gấp 4 lần GPT-3 (175 tỷ tham số), thực tế lại vượt trội hơn mô hình sau. Phát hiện này đã hiệu chỉnh nhận thức phiến diện "càng lớn càng tốt," nhấn mạnh tầm quan trọng của hiệu quả dữ liệu, và định hướng cho việc thiết kế nhiều mô hình lớn hiệu quả về sau (như dòng Llama).

Sản phẩm đáng kinh ngạc nhất của định luật tỷ lệ là "sự trỗi dậy của năng lực" (capability emergence). Cái gọi là sự trỗi dậy của năng lực đề cập đến việc khi quy mô của mô hình đạt đến một ngưỡng nhất định, nó đột nhiên thể hiện những năng lực hoàn toàn mới mà không tồn tại hoặc thể hiện kém ở các mô hình quy mô nhỏ. Ví dụ, các năng lực như **Chuỗi Tư duy (Chain-of-Thought)**, **Tuân theo Chỉ thị (Instruction Following)**, suy luận nhiều bước, tạo mã, v.v. đều chỉ xuất hiện một cách đáng kể sau khi số lượng tham số của mô hình đạt đến hàng chục hoặc thậm chí hàng trăm tỷ. Hiện tượng này cho thấy các mô hình ngôn ngữ lớn không chỉ đơn thuần ghi nhớ và đọc thuộc lòng; chúng có thể đã hình thành một mức độ trừu tượng hóa và năng lực suy luận sâu sắc hơn nào đó trong quá trình học. Đối với các nhà phát triển Agent, sự trỗi dậy của năng lực có nghĩa là việc lựa chọn một mô hình có quy mô đủ lớn là điều kiện tiên quyết để đạt được các năng lực ra quyết định và lập kế hoạch tự chủ phức tạp.

### 3.3.2 Ảo giác của Mô hình (Model Hallucination)

**Ảo giác của Mô hình (Model Hallucination)** thường đề cập đến nội dung do các mô hình ngôn ngữ lớn tạo ra mà mâu thuẫn với sự thật khách quan, đầu vào của người dùng, hoặc thông tin ngữ cảnh, hoặc tạo ra các sự thật, thực thể, hay sự kiện không tồn tại. Bản chất của ảo giác là các mô hình "bịa đặt" thông tin một cách quá tự tin trong quá trình tạo sinh thay vì truy xuất hoặc suy luận một cách chính xác. Theo hình thức biểu hiện, ảo giác có thể được chia thành nhiều loại<sup>[11]</sup>, chẳng hạn như:

- **Ảo giác Sự thật (Factual Hallucinations)**: Mô hình tạo ra thông tin không nhất quán với các sự thật của thế giới thực.
- **Ảo giác Trung thực (Faithfulness Hallucinations)**: Trong các nhiệm vụ như tóm tắt văn bản và dịch thuật, nội dung được tạo ra không phản ánh trung thực ý nghĩa của văn bản nguồn.
- **Ảo giác Nội tại (Intrinsic Hallucinations)**: Nội dung do mô hình tạo ra mâu thuẫn trực tiếp với thông tin đầu vào.

Sự sản sinh ảo giác là kết quả của nhiều yếu tố cùng tác động. Thứ nhất, dữ liệu huấn luyện có thể chứa thông tin sai lệch hoặc mâu thuẫn. Thứ hai, cơ chế tạo sinh tự hồi quy của mô hình quyết định rằng nó chỉ dự đoán token có khả năng xuất hiện tiếp theo cao nhất mà không có một module kiểm tra sự thật tích hợp sẵn. Cuối cùng, khi đối mặt với các nhiệm vụ đòi hỏi suy luận phức tạp, mô hình có thể mắc lỗi trong chuỗi logic, từ đó "bịa đặt" ra các kết luận sai. Ví dụ: một Agent lập kế hoạch du lịch có thể đề xuất một địa điểm tham quan không tồn tại hoặc đặt vé với số hiệu chuyến bay không chính xác.

Ngoài ra, các mô hình ngôn ngữ lớn còn đối mặt với những thách thức như tính kịp thời của kiến thức không đủ và các thiên kiến trong dữ liệu huấn luyện. Năng lực của mô hình ngôn ngữ lớn đến từ dữ liệu huấn luyện của chúng. Điều này có nghĩa là kiến thức mà mô hình sở hữu là những tư liệu mới nhất tại thời điểm dữ liệu huấn luyện của nó được thu thập. Đối với các sự kiện xảy ra sau ngày này, các khái niệm mới xuất hiện, hoặc các sự thật mới nhất, mô hình sẽ không thể nhận thức hoặc trả lời chính xác. Đồng thời, dữ liệu huấn luyện thường chứa nhiều thiên kiến và định kiến khác nhau từ xã hội loài người. Khi các mô hình học trên dữ liệu này, chúng không thể tránh khỏi việc hấp thụ và phản ánh những thiên kiến này<sup>[12]</sup>.

Để cải thiện độ tin cậy của mô hình ngôn ngữ lớn, các nhà nghiên cứu và nhà phát triển đang tích cực khám phá nhiều phương pháp để phát hiện và giảm nhẹ ảo giác:

1. **Cấp độ Dữ liệu**: Giảm ảo giác ngay từ nguồn thông qua việc làm sạch dữ liệu chất lượng cao, đưa vào kiến thức sự thật, và Học tăng cường từ Phản hồi của Con người (Reinforcement Learning from Human Feedback - RLHF)<sup>[13]</sup>.
2. **Cấp độ Mô hình**: Khám phá các kiến trúc mô hình mới hoặc cho phép mô hình biểu đạt sự không chắc chắn về nội dung được tạo ra.
3. **Cấp độ Suy luận và Tạo sinh**:
   1. **Tạo sinh Tăng cường bằng Truy xuất (Retrieval-Augmented Generation - RAG)**<sup>[14]</sup>: Đây hiện là một trong những phương pháp hiệu quả để giảm nhẹ ảo giác. Các hệ thống RAG truy xuất thông tin liên quan từ các cơ sở tri thức bên ngoài (như cơ sở dữ liệu tài liệu, trang web) trước khi tạo sinh, sau đó sử dụng thông tin đã truy xuất làm ngữ cảnh để dẫn dắt mô hình tạo ra các câu trả lời dựa trên sự thật.
   2. **Suy luận Nhiều bước và Kiểm chứng**: Dẫn dắt mô hình thực hiện suy luận nhiều bước và tiến hành tự kiểm tra hoặc kiểm chứng bên ngoài ở mỗi bước.
   3. **Đưa vào các Công cụ Bên ngoài**: Cho phép mô hình gọi các công cụ bên ngoài (như công cụ tìm kiếm, máy tính, trình thông dịch mã) để lấy thông tin thời gian thực hoặc thực hiện các phép tính chính xác.

Mặc dù vấn đề ảo giác khó có thể loại bỏ hoàn toàn trong ngắn hạn, nhưng thông qua các chiến lược trên, tần suất xuất hiện và tác động của chúng có thể được giảm đáng kể, cải thiện độ tin cậy và tính thực dụng của mô hình ngôn ngữ lớn trong các ứng dụng thực tế.

## 3.4 Tóm tắt Chương

Chương này đã giới thiệu kiến thức nền tảng cần thiết để xây dựng Agent, tập trung vào các mô hình ngôn ngữ lớn (LLM) với tư cách là thành phần cốt lõi của chúng. Nội dung bắt đầu từ sự phát triển của mô hình ngôn ngữ thời kỳ đầu, trình bày chi tiết kiến trúc Transformer, và giới thiệu các phương pháp tương tác với LLM. Cuối cùng, chương này đã sắp xếp các hệ sinh thái mô hình chủ đạo hiện nay, các mô thức phát triển, và các giới hạn cố hữu của chúng.

**Điểm lại Kiến thức Cốt lõi:**

- **Sự Tiến hóa của Mô hình và Kiến trúc Cốt lõi**: Chương này đã truy nguyên từ các mô hình ngôn ngữ thống kê (N-gram) đến các mô hình mạng nơ-ron (RNN, LSTM), rồi đến kiến trúc Transformer đã đặt nền móng cho các LLM hiện đại. Thông qua việc triển khai mã theo hướng "từ trên xuống" (top-down), chương này đã mổ xẻ các thành phần cốt lõi của Transformer và giải thích vai trò then chốt của cơ chế tự chú ý trong tính toán song song và nắm bắt các phụ thuộc dài hạn.
- **Các Phương pháp Tương tác với Mô hình**: Chương này đã giới thiệu hai khía cạnh cốt lõi của việc tương tác với LLM: Kỹ thuật Prompt (Prompt Engineering) và Tách Token (Tokenization). Cái trước dẫn dắt hành vi của mô hình, cái sau là nền tảng để hiểu cách mô hình xử lý đầu vào. Thông qua thực hành triển khai và chạy các mô hình mã nguồn mở tại chỗ, kiến thức lý thuyết đã được áp dụng vào các thao tác thực tế.
- **Hệ sinh thái Mô hình và Lựa chọn**: Chương này đã sắp xếp một cách có hệ thống các yếu tố then chốt cần cân nhắc khi lựa chọn mô hình cho Agent và tổng quan về các đặc điểm và định vị của các mô hình mã nguồn đóng tiêu biểu là OpenAI GPT và Google Gemini cùng các mô hình mã nguồn mở tiêu biểu là Llama và Mistral.
- **Các Định luật và Giới hạn**: Chương này đã khám phá các định luật tỷ lệ thúc đẩy sự cải thiện năng lực của LLM và giải thích các nguyên lý nền tảng. Đồng thời, chương này cũng đã phân tích các giới hạn cố hữu của mô hình như ảo giác sự thật và kiến thức lỗi thời, điều này rất quan trọng để xây dựng các Agent đáng tin cậy, bền vững.

**Từ Nền tảng LLM đến Xây dựng Agent:**

Nền tảng LLM của chương này chủ yếu giúp mọi người hiểu rõ hơn quá trình ra đời và phát triển của các mô hình lớn, điều này cũng chứa đựng một số suy nghĩ về thiết kế Agent. Ví dụ, làm thế nào để thiết kế các prompt hiệu quả nhằm dẫn dắt việc lập kế hoạch và ra quyết định của Agent, làm thế nào để chọn các mô hình phù hợp dựa trên yêu cầu nhiệm vụ, và làm thế nào để thêm các cơ chế kiểm chứng vào quy trình làm việc của Agent nhằm tránh ảo giác của mô hình—các giải pháp cho những vấn đề này đều được xây dựng dựa trên nền tảng của chương này. Giờ đây chúng ta đã sẵn sàng để chuyển từ lý thuyết sang thực hành. Trong chương tiếp theo, chúng ta sẽ bắt đầu khám phá việc xây dựng các mô thức Agent kinh điển, áp dụng kiến thức đã học trong chương này vào thiết kế Agent thực tế.

## Bài tập

1. Trong xử lý ngôn ngữ tự nhiên, các mô hình ngôn ngữ đã tiến hóa từ các mô hình thống kê sang các mô hình mạng nơ-ron.

   - Hãy sử dụng kho ngữ liệu nhỏ được cung cấp trong chương này (`datawhale agent learns`, `datawhale agent works`) để tính xác suất của câu `agent works` dưới mô hình Bigram
   - Giả định cốt lõi của các mô hình N-gram là giả định Markov. Hãy giải thích ý nghĩa của giả định này và các mô hình N-gram có những giới hạn cơ bản nào?
   - Các mô hình ngôn ngữ mạng nơ-ron (RNN/LSTM) và Transformer lần lượt khắc phục các giới hạn của mô hình N-gram như thế nào? Ưu điểm tương ứng của chúng là gì?

2. Kiến trúc Transformer<sup>[4]</sup> là nền tảng của các mô hình ngôn ngữ lớn hiện đại. Trong đó:

   > **Gợi ý**: Có thể kết hợp phần triển khai mã trong Phần 3.1.2 của chương này để hỗ trợ việc hiểu

   - Ý tưởng cốt lõi của cơ chế Tự Chú ý (Self-Attention) là gì?
   - Tại sao Transformer có thể xử lý các chuỗi một cách song song trong khi RNN phải xử lý tuần tự? Mã hóa Vị trí (Positional Encoding) đóng vai trò gì?
   - Sự khác biệt giữa kiến trúc Decoder-Only và kiến trúc Encoder-Decoder hoàn chỉnh là gì? Tại sao các mô hình ngôn ngữ lớn chủ đạo hiện nay đều áp dụng kiến trúc Decoder-Only?

3. Các thuật toán tách token từ con của văn bản là một công nghệ then chốt đối với các mô hình ngôn ngữ lớn, chịu trách nhiệm chuyển đổi văn bản thành các chuỗi token mà mô hình có thể xử lý. Tại sao chúng ta không thể trực tiếp sử dụng "ký tự" hoặc "từ" làm đơn vị đầu vào của mô hình? Thuật toán BPE (Byte Pair Encoding) giải quyết vấn đề gì?

4. Phần 3.2.3 của chương này đã giới thiệu cách triển khai các mô hình ngôn ngữ lớn mã nguồn mở tại chỗ. Hãy hoàn thành phần thực hành và phân tích sau:

   > **Gợi ý**: Đây là một câu hỏi thực hành trực tiếp; khuyến nghị thao tác thực tế

   - Theo hướng dẫn của chương này, triển khai một mô hình mã nguồn mở nhẹ tại chỗ (khuyến nghị [Qwen3-0.6B](https://modelscope.cn/models/Qwen/Qwen3-0.6B)), thử điều chỉnh các tham số lấy mẫu và quan sát tác động của chúng đến đầu ra
   - Chọn một nhiệm vụ cụ thể (như phân loại văn bản, trích xuất thông tin, tạo mã, v.v.), thiết kế và so sánh các chiến lược prompt khác nhau (như Zero-shot, Few-shot, Chuỗi Tư duy) và sự khác biệt về hiệu quả của chúng đối với kết quả đầu ra
   - So sánh các mô hình mã nguồn đóng và các mô hình mã nguồn mở từ các khía cạnh hiệu năng, chi phí, khả năng kiểm soát, quyền riêng tư, v.v.
   - Nếu bạn muốn xây dựng một Agent chăm sóc khách hàng cấp doanh nghiệp, bạn sẽ chọn loại mô hình nào? Cần cân nhắc những yếu tố nào?

5. Ảo giác của Mô hình<sup>[11]</sup> là một trong những giới hạn then chốt của các mô hình ngôn ngữ lớn hiện nay. Chương này đã giới thiệu các phương pháp để giảm nhẹ ảo giác (như tạo sinh tăng cường bằng truy xuất, suy luận nhiều bước, gọi công cụ bên ngoài)

   - Hãy chọn một phương pháp và giải thích nguyên lý hoạt động cùng các kịch bản áp dụng của nó
   - Nghiên cứu các công trình và bài báo tiên tiến—có những phương pháp khác để giảm nhẹ ảo giác của mô hình không, và chúng có những cải tiến và ưu điểm gì?

6. Giả sử bạn muốn thiết kế một Agent hỗ trợ đọc bài báo có thể giúp các nhà nghiên cứu đọc và hiểu nhanh các bài báo học thuật, bao gồm: tóm tắt nội dung cốt lõi của nghiên cứu trong bài báo, trả lời các câu hỏi về bài báo, trích xuất thông tin then chốt, so sánh quan điểm của các bài báo khác nhau, v.v. Hãy trả lời:

   - Bạn sẽ chọn mô hình nào làm mô hình nền tảng khi thiết kế Agent? Cần cân nhắc những yếu tố nào khi lựa chọn?
   - Làm thế nào để thiết kế các prompt nhằm dẫn dắt mô hình hiểu tốt hơn các bài báo học thuật? Các bài báo học thuật thường rất dài và có thể vượt quá giới hạn cửa sổ ngữ cảnh của mô hình—bạn sẽ giải quyết vấn đề này như thế nào?
   - Nghiên cứu học thuật đòi hỏi tính chặt chẽ, nghĩa là chúng ta cần đảm bảo thông tin do Agent tạo ra là chính xác, khách quan, và trung thực với văn bản gốc. Bạn nghĩ cần thêm những thiết kế nào vào hệ thống để đạt được yêu cầu này tốt hơn?

## Tài liệu tham khảo

[1] Bengio, Y., Ducharme, R., Vincent, P., & Jauvin, C. (2003). A neural probabilistic language model. *Journal of Machine Learning Research*, 3, 1137-1155.

[2] Elman, J. L. (1990). Finding structure in time. *Cognitive Science*, 14(2), 179-211.

[3] Hochreiter, S., & Schmidhuber, J. (1997). Long short-term memory. *Neural Computation*, 9(8), 1735-1780.

[4] Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N., ... & Polosukhin, I. (2017). Attention is all you need. In *Advances in neural information processing systems* (pp. 5998-6008).

[5] Radford, A., Narasimhan, K., Salimans, T., & Sutskever, I. (2018). Improving language understanding by generative pre-training. OpenAI.

[6] Gage, P. (1994). A new algorithm for data compression. *C Users Journal*, *12*(2), 23-38.

[7] Schuster, M., & Nakajima, K. (2012, March). Japanese and korean voice search. In *2012 IEEE international conference on acoustics, speech and signal processing (ICASSP)* (pp. 5149-5152). IEEE.

[8] Kudo, T., & Richardson, J. (2018). SentencePiece: A simple and language independent subword tokenizer and detokenizer for neural text processing. *arXiv preprint arXiv:1808.06226*.

[9] Kaplan, J., McCandlish, S., Henighan, T., Brown, T. B., Chess, B., Child, R., ... & Amodei, D. (2020). Scaling Laws for Neural Language Models. arXiv preprint arXiv:2001.08361.

[10] Hoffmann, J., Borgeaud, E., Mensch, A., Buchatskaya, E., Cai, T., Rutherford, R., ... & Sifre, L. (2022). Training Compute-Optimal Large Language Models. arXiv preprint arXiv:2203.07678.

[11] Ji, Z., Lee, N., Fries, R., Yu, T., & Su, D. (2023). Survey of Hallucination in Large Language Models.

[12] Bender, E. M., Gebru, T., McMillan-Major, A., & Mitchell, M. (2021). On the Dangers of Stochastic Parrots: Can Language Models Be Too Big? .

[13] Christiano, P., Leike, J., Brown, T. B., Martic, M., Legg, S., & Amodei, D. (2017). Deep reinforcement learning from human preferences. *arXiv preprint arXiv:1706.03741*.

[14] Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goswami, N., ... & Kiela, D. (2020). Retrieval-augmented generation for knowledge-intensive NLP tasks. In *Advances in neural information processing systems* (pp. 9459-9474).
