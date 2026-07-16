# Thực hành post-training cho Trợ lý Du lịch: Từ giao thức sản phẩm đến SFT, DPO và khâu hoàn thiện bằng Rerank

Dự án này đến từ `helloagents-trip-planner` mà tôi đang duy trì. Nó không phải là một dự án nghiên cứu (paper), cũng không nhằm mục đích leo bảng xếp hạng. Tôi xem nó như một bài tập kỹ thuật: làm sao để một trợ lý du lịch trông có vẻ biết trò chuyện dần dần được cải tiến đến mức có thể ghép nối front-end/back-end, có thể bị các quy tắc bắt lỗi, và vẫn có thể tiếp tục sửa.

Lập kế hoạch du lịch trông có vẻ đơn giản, nhưng thực ra rất dễ đánh lừa. Bản Demo đầu tiên thường trông khá đẹp: người dùng nói "Tôi muốn đi Hàng Châu chơi 4 ngày, ngân sách 3500", mô hình rất nhanh có thể viết ra các điểm tham quan, khách sạn, nhà hàng và những lưu ý. Nhưng khi thực sự ghép nối với front-end/back-end thì rắc rối bắt đầu: ngân sách là cho cả chuyến hay bình quân đầu người, khách sạn tính theo mấy đêm, vé tham quan có phải nhân với số người không, nhà hàng có thực sự đến từ danh sách ứng viên do công cụ cung cấp không, và ngày cuối cùng rốt cuộc có cần sắp xếp bữa tối hay không.

Bài viết Extra-Chapter này chỉ viết bản rút gọn: con đường này đã đi qua như thế nào, những thử nghiệm nào có ích, những thử nghiệm nào về sau chứng minh là không cần thiết. Các lệnh, cấu hình và bản lưu trữ đầy đủ được đặt trong kho lưu trữ (repository) của dự án.

Dự án và tài liệu đi kèm:

- Kho dự án Trợ lý Du lịch: [helloagents-trip-planner](https://github.com/nameless0120/helloagents-trip-planner)
- Giáo trình post-training đầy đủ: [Giáo trình thực hành post-training cho Trợ lý Du lịch](https://github.com/nameless0120/helloagents-trip-planner/blob/main/training/docs/%E6%95%99%E7%A8%8B/%E6%97%85%E8%A1%8C%E5%8A%A9%E6%89%8B%E5%90%8E%E8%AE%AD%E7%BB%83%E5%AE%9E%E6%88%98%E6%95%99%E7%A8%8B.md)
- Dữ liệu đi kèm: `helloagents-后训练数据`, liên kết ổ đĩa mạng (netdisk): <https://pan.baidu.com/s/5oNsK7pwQnqzQEUg5ykb09Q>

Chạy đến cuối, tôi thấy tuyến đường thực ra rất mộc mạc: **Prompt cố định giao thức, SFT học cấu trúc, DPO học sở thích (preference), Rerank chọn câu trả lời ổn định hơn trong số các ứng viên.**

![Sơ đồ tổng quan tuyến chính của post-training](./images/Extra12-figures/01-后训练主线总览图.png)

---

## Mục lục

- [Chương 1: Trước tiên hãy xem một ví dụ so sánh trước/sau](#chương-1-trước-tiên-hãy-xem-một-ví-dụ-so-sánh-trướcsau)
- [Chương 2: Vì sao không thể vừa vào đã huấn luyện ngay](#chương-2-vì-sao-không-thể-vừa-vào-đã-huấn-luyện-ngay)
- [Chương 3: Trước tiên hãy sửa giao thức sản phẩm](#chương-3-trước-tiên-hãy-sửa-giao-thức-sản-phẩm)
- [Chương 4: Đóng băng tập đánh giá (eval set)](#chương-4-đóng-băng-tập-đánh-giá-eval-set)
- [Chương 5: Gỡ lỗi Prompt là để tìm ranh giới](#chương-5-gỡ-lỗi-prompt-là-để-tìm-ranh-giới)
- [Chương 6: Sinh dữ liệu SFT và kiểm định (audit)](#chương-6-sinh-dữ-liệu-sft-và-kiểm-định-audit)
- [Chương 7: Huấn luyện LoRA SFT nhiều giai đoạn](#chương-7-huấn-luyện-lora-sft-nhiều-giai-đoạn)
- [Bổ sung Chương 7: Vì sao full-parameter SFT không trở thành tuyến chính](#bổ-sung-chương-7-vì-sao-full-parameter-sft-không-trở-thành-tuyến-chính)
- [Chương 8: Best-of-N Replay và SFT Rerank](#chương-8-best-of-n-replay-và-sft-rerank)
- [Chương 9: DPO học sở thích, chỉ số cốt lõi đổi thành PlannerSoft](#chương-9-dpo-học-sở-thích-chỉ-số-cốt-lõi-đổi-thành-plannersoft)
- [Chương 10: Rerank đa ứng viên cuối cùng](#chương-10-rerank-đa-ứng-viên-cuối-cùng)
- [Chương 11: So sánh với tham chiếu bên ngoài MiMo như thế nào](#chương-11-so-sánh-với-tham-chiếu-bên-ngoài-mimo-như-thế-nào)
- [Bad Case Gallery: Ba lỗi đáng giá nhất](#bad-case-gallery-ba-lỗi-đáng-giá-nhất)
- [Chương 12: Những kinh nghiệm đọng lại từ thí nghiệm này](#chương-12-những-kinh-nghiệm-đọng-lại-từ-thí-nghiệm-này)
- [Tài nguyên tái hiện](#tài-nguyên-tái-hiện)

---

## Chương 1: Trước tiên hãy xem một ví dụ so sánh trước/sau

Trước tiên chưa nói đến LoRA, chưa nói đến DPO, cũng chưa nói đến rerank. Hãy xem một yêu cầu bình thường:

> Một người đi Hàng Châu chơi 4 ngày, di chuyển bằng taxi, ở khách sạn hạng phổ thông (economy), thích ẩm thực và các địa danh nổi bật của thành phố, tổng ngân sách khoảng 3500 tệ, và không được vượt quá.

Mô hình cơ sở (base model) không phải hoàn toàn không biết viết. Nó viết được Tây Hồ, chùa Linh Ẩn, ban công thành phố (city balcony), cũng cho ra khách sạn và nhà hàng. Nhưng nhìn kỹ sẽ thấy không chắc chắn: lịch trình khá rỗng, số đêm khách sạn không ổn định, nhà hàng lặp lại, ngân sách cũng cách quá xa mục tiêu người dùng. Nó trông giống một kế hoạch du lịch, nhưng thực sự đưa cho người dùng thì hơi thiếu tin cậy.

Phiên bản sau post-training cũng không phải điểm tuyệt đối, ví dụ ngân sách ăn uống vẫn còn một sai số nhỏ 60 tệ. Nhưng ít nhất nó bắt đầu giống một lịch trình được sắp xếp nghiêm túc: số đêm khách sạn ổn định, mật độ điểm tham quan bình thường, nhà hàng không còn lặp lại một mạch, ngân sách cũng gần hơn với ràng buộc cứng (hard constraint) mà người dùng đưa ra.

| Điểm quan sát | Mô hình cơ sở | Sau post-training |
| --- | --- | --- |
| Mật độ lịch trình | 4 ngày cơ bản mỗi ngày 1 điểm tham quan, khá rỗng | Mỗi ngày 2 đến 3 điểm tham quan, bao phủ Tây Hồ, Đoạn Kiều, Lôi Phong Tháp, chùa Linh Ẩn, Tây Khê, Thanh Hà Phường, v.v. |
| Khách sạn | 3 ngày đầu có khách sạn, ngày thứ 4 ghi là "không lưu trú", nhưng trong ngân sách lại tính theo 4 đêm | 3 đêm đầu ổn định dùng cùng một khách sạn phổ thông, ngân sách tính theo 3 đêm |
| Ẩm thực | KFC lặp lại khá nhiều, bữa trưa/tối luân phiên kém | Nhà hàng có luân phiên, bao gồm món Hàng Châu, hải sản, đồ nướng, quán mì và một ít đồ ăn nhanh |
| Ngân sách | Báo 1840 tệ, cách quá xa hard budget 3500 tệ | Báo 2500 tệ, rơi vào cận dưới của khoảng chấp nhận được |
| Đánh giá bằng quy tắc | Bắt được 9 loại lỗi | Hiện tại chủ yếu còn sai số nhỏ về ngân sách ăn uống |

![Sơ đồ so sánh trước/sau của Case mở đầu](./images/Extra12-figures/02-开篇Case前后对比图.png)

Tôi muốn nói rõ trước: post-training không phải để mô hình viết văn án cho đẹp, mà là để nó giống hơn một Planner có thể đặt vào quy trình sản phẩm. Số đêm lưu trú, grounding ẩm thực (căn cứ vào ứng viên thực tế), quan hệ ngân sách, ngày tháng thời tiết, xuất ra JSON, những thứ này trông có vẻ vụn vặt, nhưng trong sản phẩm thực tế thì chính chúng dễ hỏng việc nhất.

---

## Chương 2: Vì sao không thể vừa vào đã huấn luyện ngay

Ban đầu tôi cũng rất muốn huấn luyện ngay. Chạy LoRA cho cảm giác tiến độ nhất: dữ liệu chuẩn bị xong, script khởi động, loss bắt đầu tụt xuống, trông như dự án đang tiến về phía trước.

Về sau mới phát hiện thứ tự này bị ngược.

Nếu các sự thật nghiệp vụ (business facts) chưa được cố định, huấn luyện chỉ khiến sự hỗn loạn được học một cách ổn định hơn. Người dùng nói "ngân sách 3000", mô hình phải biết đây là ngân sách cả chuyến hay ngân sách bình quân đầu người; giá khách sạn là mỗi phòng mỗi đêm, không phải tổng giá cả chuyến; vé tham quan phải nhân với số người đi cùng; nhà hàng không được bịa ra vô căn cứ, tốt nhất phải đến từ danh sách ứng viên do công cụ cung cấp. Chỉ dựa vào việc nhắc đi nhắc lại trong prompt thì cứu được một phần, nhưng không cứu được cả chuỗi.

Vì vậy về sau tôi đã không đi theo con đường này:

```text
写 prompt -> 造数据 -> 训练 -> 看指标
```

Mà là:

```text
前后端协议改造
  -> 冻结 standard / hard 评测集
  -> prompt 调试和失败画像
  -> 强模型生成 SFT 数据
  -> 数据审计与 LLaMA-Factory 导出
  -> LoRA SFT 多阶段训练
  -> Best-of-N Replay
  -> DPO 偏好训练
  -> 规则评测、切片对比和 checkpoint 选择
  -> 多候选 Rerank 收尾
```

Chậm thì có chậm, nhưng cái lợi rất thực tế: mỗi vòng đều biết mình đang sửa cái gì, và cũng biết chỗ nào lại bị làm hỏng.

---

## Chương 3: Trước tiên hãy sửa giao thức sản phẩm

Lúc mới bắt đầu làm trợ lý du lịch, tôi cũng rất dễ ném hết mọi vấn đề cho mô hình: bắt nó từ ngôn ngữ tự nhiên đoán số người, đoán cách hiểu ngân sách (budget caliber), đoán số đêm lưu trú, rồi lại đoán giá vé và giá nhà hàng. Bản đầu tiên chạy được, nhưng huấn luyện về sau sẽ rất đau đớn, vì các "sự thật" trong dữ liệu huấn luyện vốn dĩ đã bấp bênh.

Về sau, quyết định đầu tiên tôi đưa ra rất mộc mạc: **đừng để mô hình đoán các sự thật nghiệp vụ.**

Front-end không còn chỉ gửi lên một đoạn văn bản tự do, mà gửi lên một cách tường minh:

- `party`: người lớn, trẻ em, người già, tổng số người, loại hình đi lại;
- `budget_constraint`: số tiền, loại tiền tệ, khoảng ngân sách, mức ngân sách (budget tier), độ mạnh của ràng buộc;
- `travel_days`, phương thức di chuyển, sở thích lưu trú, sở thích sở thích, và các trường có cấu trúc khác.

Back-end cũng không còn nhồi hết kết quả công cụ vào prompt, mà trước tiên biên dịch (compile) thành `PlannerContext`. Ngữ cảnh này sẽ nói rõ cho mô hình biết: lần này đi mấy người, ngân sách là cả chuyến hay bình quân đầu người, khách sạn tính theo mấy đêm, mỗi điểm tham quan, nhà hàng, khách sạn ứng viên đến từ đâu, gợi ý giá (price hint) và chiến lược ngân sách là gì, và cuối cùng đầu ra phải thỏa mãn JSON shape nào.

![Sơ đồ cải tạo giao thức sản phẩm](./images/Extra12-figures/03-产品协议改造图.png)

Nếu muốn theo mã nguồn để xem, có thể tìm theo manh mối này: kiểu (type) form front-end, schema request back-end, biên dịch PlannerContext, phân tích (parse) và kiểm tra (validate) đầu ra. Đường dẫn tệp cụ thể được đặt trong giáo trình đầy đủ của `helloagents-trip-planner`, ở đây không triển khai chi tiết.

Có lớp giao thức này rồi thì nhiệm vụ của post-training mới trở nên hẹp lại: mô hình không còn viết kế hoạch du lịch theo cảm tính nữa, mà lựa chọn trong số các ứng viên có cấu trúc, và xuất ra JSON hợp lệ.

---

## Chương 4: Đóng băng tập đánh giá (eval set)

Kết quả huấn luyện không hiểu nổi, rất nhiều khi không phải vấn đề của mô hình, mà là đề thi cứ thay đổi liên tục.

Trợ lý du lịch đặc biệt dễ như vậy: hôm nay ứng viên bản đồ thay đổi, ngày mai thời tiết thay đổi, ngày kia logic sinh ngân sách lại thay đổi. Cuối cùng bạn không phân biệt được là mô hình mạnh lên hay đề thi dễ đi.

Vì vậy tôi cố định tập đánh giá trước.

![Sơ đồ đóng băng tập đánh giá](./images/Extra12-figures/04-评测集冻结图.png)

Tôi chia đánh giá thành hai loại:

| Tập đánh giá | Tác dụng |
| --- | --- |
| `standard eval` | Xem độ ổn định dưới các yêu cầu bình thường, ví dụ thành phố thông thường, ngân sách thông thường, sở thích thông thường |
| `hard eval` | Chủ động phóng đại điểm khó, ví dụ nhiều người, người già trẻ em, ngân sách nghiêm ngặt, sở thích tiêu cực (negative preference), chế độ ăn đặc biệt |

Ở đây còn một chi tiết: về sau chiến lược truy hồi (retrieval) và việc sửa chữa ngữ cảnh thay đổi, quả thực cần dựng lại ngữ cảnh đánh giá, nhưng không nên lấy mẫu lại (re-sample) yêu cầu người dùng. Cách làm của tôi là giữ nguyên request signature (chữ ký yêu cầu) không đổi, chỉ dựng lại ứng viên công cụ và ngữ cảnh. Như vậy vừa giữ được tính so sánh được, vừa sửa được dữ liệu bẩn trong ngữ cảnh cũ.

Bộ frozen eval này về sau luôn được dùng để chọn mô hình. Theo cách gọi nghiêm ngặt của paper, nó giống một validation set (tập kiểm định) hơn, không phải blind test (kiểm định mù). Vì vậy tôi chỉ gọi nó là "đánh giá theo giai đoạn trên tập đánh giá cố định", chứ không đóng gói thành một bài kiểm định mù độc lập.

Cuối cùng khi làm dữ liệu hoàn thiện DPO, tôi đã kiểm tra riêng độ chồng lấp chữ ký: `selected_eval_signature_overlap = 0`. Tức là, prompt đánh giá không lọt vào dữ liệu huấn luyện.

---

## Chương 5: Gỡ lỗi Prompt là để tìm ranh giới

Sau khi giao thức front-end/back-end và tập đánh giá ổn định, mới bước vào gỡ lỗi prompt.

Ở đây tôi không tìm một "prompt thần thánh". Điều tôi quan tâm hơn là: những vấn đề nào prompt có thể cứu, những vấn đề nào bắt buộc phải giao cho dữ liệu, quy tắc và kỹ thuật.

Vài vòng prompt đầu tiên đại khái giải quyết ba loại vấn đề:

| Vòng | Mục tiêu chính | Kết luận |
| --- | --- | --- |
| Giao thức đầu ra | Ngày tháng không được loạn, bữa ăn không được thiếu, trường khách sạn không được bấp bênh, JSON không được cụt nửa chừng | prompt có thể cải thiện rõ rệt độ ổn định của shape, nhưng vẫn phải phối hợp với parser và validation |
| Grounding ẩm thực | Nhà hàng phải đến từ ứng viên, không viết "quán ăn vặt gần đó" "nhà hàng đặc sản địa phương" | Viết trong prompt vẫn chưa đủ, đánh giá cũng phải bắt được đầu ra chưa grounded |
| Tuyến đường giả chính xác (伪精确) | Không viết "đi bộ 10 phút" "taxi 15 phút" mà công cụ chưa từng cung cấp | Loại hallucination này thích hợp hơn để chặn lại trong quy tắc đầu ra |

Bước này thứ thực sự hữu ích không phải bản thân prompt, mà là bức chân dung thất bại (failure profile). Nhìn nhiều bad case, vấn đề sẽ tự phân tầng: schema, ngày tháng, thiếu bữa ăn thích hợp cho shape validation; nhà hàng và điểm tham quan không grounded thì phải dựa vào prompt và quy tắc cùng nhau ép; quan hệ ngân sách quá phức tạp thì tốt nhất tách thành hai phần là tính lại bằng kỹ thuật và lựa chọn bằng mô hình; độ thỏa mãn sở thích chưa đủ thì đi bổ sung dữ liệu.

Prompt điều chỉnh đến cuối, điều quan trọng nhất không phải là viết dài thêm nữa, mà là biết nên dừng ở đâu.

---

## Chương 6: Sinh dữ liệu SFT và kiểm định (audit)

Sinh dữ liệu SFT dễ khiến người ta lơ là cảnh giác nhất.

Mô hình mạnh (strong model) quả thực có thể sinh ra kế hoạch du lịch trông rất tươm tất, nhưng "tươm tất" không đồng nghĩa với "huấn luyện được". Nếu trong đầu ra của teacher, cách hiểu ngân sách sai, nhà hàng không grounded, khách sạn mỗi ngày đổi loạn xạ, thì mô hình học sinh (student model) sẽ học ổn định hơn, và cũng sẽ sai một cách ổn định hơn.

Vì vậy tôi tách việc sinh dữ liệu ra làm:

1. Trước tiên dry-run phân bố yêu cầu, xem thành phố, số ngày, ngân sách, loại hình đi cùng có hợp lý không.
2. Tiếp theo dry-run `PlannerContext`, xác nhận ứng viên công cụ, gợi ý giá, thời tiết và chiến lược ngân sách đều biên dịch ra được.
3. Sinh theo lô nhỏ, trước tiên chạy 20 đến 100 bản, đừng vừa vào đã sinh cả nghìn bản.
4. Mỗi lần gọi mô hình mạnh đều ghi lại usage, manifest và cấu hình run.
5. Sinh xong thì kiểm định trước, rồi mới đưa vào tập huấn luyện.

![Sơ đồ phễu sinh và kiểm định dữ liệu SFT](./images/Extra12-figures/05-SFT数据生成与审计漏斗图.png)

Khi kiểm định, trước tiên xem bộ lọc cứng (hard filter):

| Mục lọc | Vì sao quan trọng |
| --- | --- |
| JSON / schema hợp lệ | Back-end phải phân tích được |
| Ngày tháng và số ngày nhất quán | Kế hoạch du lịch không được thiếu ngày, thừa ngày, sai ngày |
| Khách sạn và nhà hàng grounded | Không được bịa ứng viên vô căn cứ |
| Ẩm thực không lặp lại | Không được ăn liền mấy bữa cùng một quán ăn nhanh |
| Ràng buộc cứng ngân sách (hard constraint) | Ngân sách cứng không được vượt |
| Quan hệ ngân sách hợp lý | Số đêm khách sạn, số người mua vé, quy mô ẩm thực phải đúng |

Còn một điểm dễ bỏ qua: dữ liệu cũ đừng tiếc mà giữ. Đầu dự án có một loạt dữ liệu SFT cũ, nhìn cục bộ thì khá sạch, nhưng đến từ cách hiểu ngân sách cũ. Về sau tôi chọn lưu trữ toàn bộ, không tiếp tục vá víu để dùng nữa. Quyết định này lúc đó hơi đau, nhưng nhìn lại là đúng. Post-training sợ nhất là "giao thức mới + dữ liệu theo cách hiểu cũ" trộn lẫn với nhau, bề mặt thì mô hình học được nhiều mẫu hơn, thực chất lại học được những quy tắc mâu thuẫn nhau.

---

## Chương 7: Huấn luyện LoRA SFT nhiều giai đoạn

Có dữ liệu rồi thì mới thực sự bước vào LoRA SFT.

Tuyến này dùng Qwen2.5-7B-Instruct để làm LoRA. Huấn luyện không hoàn thành trong một vòng, mà đẩy tiến qua nhiều giai đoạn. Nguyên tắc của tôi là: **cố gắng ít thay đổi biến số cùng lúc nhất có thể.**

Các thiết lập lớp nền cơ bản giữ ổn định: LoRA r32, ngữ cảnh dài (long context), bf16, tích lũy gradient. Ở đây điều quan trọng nhất không phải học thuộc bảng tham số, mà là hiểu vì sao mỗi vòng sau chỉ đổi một biến số chính. Trong `PlannerContext` có điểm tham quan, khách sạn, nhà hàng, thời tiết và chiến lược ngân sách, nén ngắn ngữ cảnh sẽ cắt cụt tín hiệu ngay; rank quá nhỏ thì giao thức JSON dài và lựa chọn ứng viên cũng học không ổn định.

Thứ thực sự phải điều chỉnh đi chỉnh lại là ba loại: dữ liệu, learning rate (tốc độ học), số vòng huấn luyện (epoch).

| Giai đoạn | Điểm xuất phát | Dữ liệu | Tham số chính | Muốn giải quyết gì |
| --- | --- | --- | --- | --- |
| Clean SFT xương sống (main clean) | Qwen2.5-7B-Instruct | `main_clean` | `lr=8e-5 / 6e-5`, `epoch=4` | Trước tiên học ổn giao thức TripPlan |
| Bổ sung huấn luyện hỗn hợp ngân sách thực (usage700) | Huấn luyện tiếp từ adapter `lr6e-5` | main clean + realbudget usage700 | `lr=2e-5`, `epoch=1` | Bổ sung việc sử dụng ngân sách và cách hiểu ngân sách thực |
| Bổ sung huấn luyện chẩn đoán tận dụng ngân sách (patch700) | Huấn luyện tiếp từ adapter `lr6e-5` | budget utilization patch 700 | `lr=1e-5`, `epoch=2` | Xem dữ liệu bổ sung kiểu tận dụng ngân sách rốt cuộc đẩy được đến đâu |
| Replay SFT Rerank 600 (Best-of-N 600) | Huấn luyện tiếp từ adapter usage700 | old replay + Best-of-N winner | `lr=1e-5`, lưu nửa vòng | Tiêm vào các ứng viên tốt hơn được quy tắc lọc ra |
| Replay SFT Rerank 1200 (Best-of-N 1200) | Huấn luyện tiếp từ Best-of-N 600 final | old replay + nhiều Best-of-N winner hơn | `lr=1e-5`, lưu nửa vòng | Tăng tỷ lệ winner, xem có tiếp tục cải thiện không |

![Sơ đồ dòng thời gian huấn luyện LoRA nhiều giai đoạn](./images/Extra12-figures/06-LoRA多阶段训练时间线图.png)

Ở đây có một cái hố tôi đã giẫm phải: `adapter_name_or_path` không phải là `resume_from_checkpoint`. Nó chỉ lấy LoRA adapter xuất ra ở vòng trước để làm warm-start, trạng thái optimizer sẽ không đi tiếp từ vòng trước. Tức là, mỗi giai đoạn đều sẽ dùng lại learning rate và scheduler trong cấu hình hiện tại.

Điều này ngược lại lại thích hợp cho thí nghiệm theo giai đoạn. Năng lực học được ở vòng trước lưu trong adapter, vòng sau dùng learning rate nhỏ hơn để tiếp tục sửa các vấn đề cục bộ.

Learning rate cứ giảm dần một mạch, cũng là vì lý do này:

```text
main clean:       6e-5 / 8e-5
usage700 mixed:   2e-5
patch700 only:    1e-5
Best-of-N replay: 1e-5
DPO closing:      1e-6 到 1.5e-6 级别
```

Càng về sau, dữ liệu càng giống như đang sửa vấn đề cục bộ. Learning rate quá cao, chỉ số ngân sách có thể lên được, nhưng grounding ẩm thực, tính liên tục của lưu trú hoặc ngày tháng thời tiết lại tụt xuống.

### Bổ sung Chương 7: Vì sao full-parameter SFT không trở thành tuyến chính

Sau khi viết xong tuyến chính LoRA, tôi lại bổ sung thêm một lần full-parameter SFT (SFT toàn tham số). Không phải muốn lật đổ tuyến đường phía trước, mà chỉ muốn xem giới hạn trên: nếu không chỉ huấn luyện adapter, mà động đến toàn mô hình, liệu Planner du lịch có tăng thêm một mức nữa không?

Lần này dùng `Qwen2.5-7B-Instruct` và dữ liệu clean SFT bản đầu tiên, giữ ngữ cảnh dài, chạy 6 epoch. Huấn luyện chạy được, nhưng chi phí lập tức tăng lên: 6 card 40GB một mẻ khoảng 7 tiếng, mô hình lưu ra 28GB. So với một LoRA adapter, đây đã không còn là cùng một cảm giác lặp (iteration feel) nữa.

Kết quả khá thú vị. Full-parameter không phải là vô dụng, nó quả thực đẩy planner soft lên:

| Chỉ số | Full Instruct toàn tham số | LoRA cùng bản | Chênh lệch |
| --- | ---: | ---: | ---: |
| Hard pass | 94.4% | 95.8% | -1.4pp |
| Planner Soft | 48.0% | 44.7% | +3.3pp |
| Ngân sách tính lại Soft | 33.8% | 30.7% | +3.1pp |
| Đa dạng ẩm thực | 82.6% | 76.4% | +6.2pp |
| Độ khớp sở thích ngân sách | 67.8% | 66.9% | +0.9pp |
| Tổng ngân sách nhất quán | 68.0% | 72.3% | -4.3pp |
| Ràng buộc ngân sách người dùng | 88.8% | 91.6% | -2.8pp |

![Sơ đồ so sánh full-parameter SFT và LoRA](./images/Extra12-figures/18-全参SFT与LoRA对比图.png)

Trên tập standard, Planner Soft của phiên bản full-parameter tăng từ 48.0% lên 57.5%, mức cải thiện này rất rõ ràng. Nó sẵn lòng viết lịch trình phong phú hơn, ẩm thực cũng không dễ lặp lại. Nói cách khác, cập nhật full-parameter quả thực đã động đến "thói quen lập kế hoạch" của mô hình, không chỉ học định dạng ở bề mặt.

Nhưng cuối cùng tôi vẫn không đưa full-parameter vào tuyến chính. Lý do rất đơn giản: điều khó nhất của dự án này không phải là để mô hình viết nhiều hơn một chút, mà là để nó đừng tính sai trong một đống ràng buộc cứng. Tổng ngân sách, ngân sách người dùng, số đêm khách sạn, vé tham quan tính theo số người, những thứ này giống vấn đề quy tắc kỹ thuật và phân bố dữ liệu hơn. Full-parameter khiến mô hình linh hoạt hơn, nhưng linh hoạt không đồng nghĩa với ổn định hơn.

Với dự án này, cuối cùng tôi vẫn thiên về dùng LoRA hơn:

- Lượng dữ liệu cỡ nghìn, không phải mấy trăm nghìn bản dữ liệu chỉ dẫn (instruction data). Dung lượng full-parameter quá lớn, dễ học luôn cả phong cách cục bộ vào.
- Trọng tâm nhiệm vụ là sao chép ngữ cảnh dài, xuất schema, grounding ứng viên và quy tắc ngân sách. LoRA đã có thể nén những thứ này đến mức rất cao.
- Chi phí lặp chênh lệch quá lớn. LoRA huấn luyện, lưu, roll back (quay lui), đánh giá đều nhẹ, full-parameter mỗi lần thử đều phải nghiêm túc sắp xếp card và dọn dung lượng.
- Những chỗ thực sự có thể tăng điểm về sau, phần lớn nằm ở làm sạch dữ liệu, khai thác bad case, DPO pair và rerank. Full-parameter sẽ làm chậm nhịp độ này.

Vì vậy vị trí của thí nghiệm full-parameter lần này trong giáo trình rất rõ ràng: **nó cho thấy Planner Soft vẫn còn không gian tăng; cũng cho thấy dự án này không thể dựa vào full-parameter để ép cứng.**

Full-parameter có thể chơi, đặc biệt khi muốn làm một kết quả offline đẹp mắt. Nhưng dự án này cần bổ sung dữ liệu, chạy đánh giá, roll back so sánh đi làm lại nhiều lần, tôi vẫn sẽ chọn LoRA + DPO + Rerank.

---

## Chương 8: Best-of-N Replay và SFT Rerank

Sau khi SFT học ổn giao thức, tôi làm Best-of-N replay trước, rồi mới làm rerank ở giai đoạn trình diễn cuối cùng. Tên hơi giống nhau, nhưng chúng không phải cùng một thứ.

![Sơ đồ Best-of-N Replay và Rerank](./images/Extra12-figures/07-Best-of-N-Replay与Rerank图.png)

Best-of-N Replay là quy trình xây dựng dữ liệu huấn luyện: cùng một `PlannerContext`, cho mô hình hiện tại lấy mẫu nhiều câu trả lời, dùng bộ đánh giá quy tắc (rule evaluator) chọn một cái tốt hơn, rồi xuất winner thành dữ liệu SFT cho vòng sau.

```text
PlannerContext
  -> t=0.2 / 0.5 / 0.8 多温度采样
  -> 每个候选跑 rule metrics
  -> 优先选 hardpass 候选
  -> 再看预算、餐饮尺度、多样性等软奖励
  -> winner 进入下一轮 SFT
```

Rerank cuối cùng là quy trình lúc suy luận (inference): cùng một prompt sinh ra nhiều ứng viên, không còn ghi winner ngược lại vào tập huấn luyện nữa, mà trực tuyến (online) chọn một câu trả lời ổn định hơn từ hồ ứng viên để trả về cho người dùng.

Cả hai quy trình này cuối cùng đều phải quay lại xem trên frozen eval. Quy tắc chọn winner chắc chắn có thiên lệch (bias), nếu reward quá thiên về một chỉ số nào đó, mô hình có thể trở nên bảo thủ, cũng có thể hy sinh trải nghiệm. Chỉ nhìn điểm số của một winner đơn lẻ thì rất dễ mừng quá sớm.

Sau khi giai đoạn SFT tiếp nhận ứng viên đa nhiệt độ (multi-temperature) + rerank bằng quy tắc, một vài phiên bản tổng thể lên được một bậc:

| Phiên bản | hardpass | softpass | softpass ngân sách tính lại | Số học ngân sách | Sở thích ngân sách | Quan hệ ngân sách | Quy mô ẩm thực |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| SFT checkpoint giữa kỳ + rerank (ckpt104) | 98.0 | 65.6 | 54.6 | 81.2 | 77.0 | 86.4 | 88.8 |
| Best-of-N 1200 replay + rerank (final1200) | 98.2 | 66.8 | 54.6 | 78.0 | 78.4 | 85.0 | 88.0 |
| Best-of-N 600 replay bản cuối + rerank (old600final) | 98.2 | 66.2 | 59.2 | 78.4 | 75.4 | 87.0 | 89.4 |

![Sơ đồ so sánh SFT Rerank](./images/Extra12-figures/09-sft-rerank-comparison.png)

Đến đây, giai đoạn SFT có thể khép lại. Lợi ích của việc tiếp tục thêm SFT đã trở nên cùn đi, mức tăng chính về sau nên đến từ dữ liệu sở thích (preference data) và lựa chọn ứng viên.

---

## Chương 9: DPO học sở thích, chỉ số cốt lõi đổi thành PlannerSoft

SFT đã có thể viết ổn cái vỏ của TripPlan, nhưng giữa các câu trả lời hợp lệ cũng có tốt xấu. Hai kế hoạch đều qua schema, đều tìm được khách sạn và nhà hàng, một cái có thể rất tiết kiệm nhưng không giống chuyến du lịch mà người dùng muốn, một cái khác thì ngân sách khớp hơn, ẩm thực ít lặp lại hơn, điểm tham quan cũng thuận hơn.

Sự đánh đổi kiểu này rất khó học ổn từ một mẫu teacher đơn lẻ, DPO thuận tay hơn.

Tôi không dùng DPO như một công cụ tăng cường vạn năng. Ở đây nó chỉ làm một việc: **trong số các ứng viên đã qua ải hardpass, học xem cái nào giống một lịch trình tốt hơn.**

### DPO pair phải qua ngưỡng cứng trước

chosen / rejected của DPO pair không được làm bừa. JSON tồi đối với JSON tốt, kiểu pair này đương nhiên có tín hiệu với mô hình, nhưng cái nó học được là định dạng, không phải sở thích. Trong dự án này, hữu ích hơn là kiểu pair dưới đây:

```text
同一个 PlannerContext
  -> chosen: schema 过、hardpass 过、planner soft 过
  -> rejected: schema 过、hardpass 过，但预算/重复/偏好没过
```

Mô hình huấn luyện ra như vậy mới là đang học lựa chọn giữa các kế hoạch hợp lệ, chứ không phải học lại cách viết JSON.

Còn một lằn ranh đáy: không được khai thác training pair từ tập đánh giá đóng băng. Trong dữ liệu hoàn thiện ngân sách đã làm riêng bộ lọc chữ ký:

```text
frozen eval signature count = 497
selected eval signature overlap = 0
```

Bước này rất phiền, nhưng không thể bỏ. Nếu không thì điểm nhìn có vẻ đẹp, nhưng thực ra là đang học thuộc đề.

![Sơ đồ sàng lọc mẫu DPO và chống rò rỉ](./images/Extra12-figures/12-DPO样本筛选与防泄漏图.png)

### Chỉ số chính đổi thành PlannerSoft

Về sau tôi ngày càng thấy, softpass thông thường vẫn chưa đủ. Đầu ra của trợ lý du lịch không phải một câu trắc nghiệm, mà là một kế hoạch người dùng có thể thực sự đem đi dùng. Vì vậy chỉ số chính dần chuyển thành `planner soft`: độ khớp ngân sách, lặp lại ẩm thực, lặp lại điểm tham quan, quan hệ ngân sách, những thứ này đều phải xem.

![Sơ đồ phân rã chỉ số PlannerSoft](./images/Extra12-figures/16-PlannerSoft指标分解图.png)

Tuyến đường vài vòng DPO đại khái là:

| Giai đoạn | Mục đích | Kết luận |
| --- | --- | --- |
| DPO sở thích độ tin cậy cao chạy thử | Trước tiên xác minh DPO ngữ cảnh dài có chạy thông không | Quy trình chạy thông, về sau bắt đầu đổi chỉ số |
| DPO quy tắc PlannerSoft | Chuyển mục tiêu tối ưu từ hardpass sang planner soft | Chọn ra điểm xuất phát DPO đầu tiên có thể tiếp tục mở rộng dữ liệu (ckpt25) |
| PlannerSoft mở rộng dữ liệu + neo Direct | Mở rộng dữ liệu planner soft, đồng thời giữ lại direct preference | Có được baseline mở rộng dữ liệu ổn định hơn (ckpt126) |
| PlannerSoft Clean nâng cấp sinh đơn (single generation) | Dùng dữ liệu clean quy mô lớn hơn để huấn luyện tiếp | Bản sinh đơn tốt nhất (260519 ckpt138) |
| DPO hoàn thiện ngân sách | Nhắm vào ngân sách thiên bảo thủ, vượt chi, lặp lại để xây dựng clean pair | Sinh đơn không tiếp tục tăng, nhưng hồ ứng viên thích hợp cho rerank hơn (260520 / 260521) |

DPO loss cũng đừng so cứng giữa các batch. Vài vòng pair đầu rất dễ phân biệt, loss thấp, accuracy cao; pair hoàn thiện ngân sách thì gần nhau hơn, cả chosen và rejected đều là kế hoạch hardpass, chỉ khác nhau ở việc sử dụng ngân sách, lặp lại và sở thích, loss cao hơn một chút ngược lại là bình thường.

![Sơ đồ DPO loss không thể so sánh giữa các batch](./images/Extra12-figures/15-DPO-loss跨批次不可比图.png)

Về sau tôi coi trọng hơn hai tín hiệu: reward accuracy có tăng lên ổn định không, và planner soft cùng các chỉ số liên quan đến ngân sách trên frozen eval có thực sự chuyển động không. Log huấn luyện cho bạn biết mẻ này có hỏng không, còn đánh giá mới cho bạn biết mẻ này có ích không.

---

## Chương 10: Rerank đa ứng viên cuối cùng

Nửa sau của DPO dễ bị đọc sai nhất. Bản sinh đơn ổn định nhất là bản PlannerSoft Clean kia, tức là 260519 ckpt138; hai vòng hoàn thiện ngân sách về sau (260520 ckpt66, 260521 ckpt64) không đẩy điểm sinh đơn tiếp tục lên cao.

Nhưng trình diễn cuối cùng không phải sinh đơn, mà là rerank đa ứng viên. Giá trị của việc hoàn thiện ngân sách chủ yếu thể hiện ở hồ ứng viên: nó không nhất thiết làm cho câu trả lời phát đầu tiên đạt điểm cao hơn, nhưng dễ lấy mẫu ra được câu trả lời tốt có thể được quy tắc chọn hơn.

![Sơ đồ so sánh sinh đơn và Rerank đa ứng viên](./images/Extra12-figures/17-单生成与多候选Rerank对比图.png)

Vài nhóm chỉ số cuối cùng đại khái là:

| Phiên bản | hardpass | planner soft | ngân sách tính lại soft |
| --- | ---: | ---: | ---: |
| Baseline mở rộng dữ liệu (ckpt126) | 98.4% | 66.9% | 48.5% |
| PlannerSoft Clean sinh đơn (260519 ckpt138) | 98.4% | 71.5% | 50.9% |
| Hoàn thiện ngân sách vòng 1 sinh đơn (260520 ckpt66) | 99.0% | 70.1% | 48.3% |
| Hoàn thiện ngân sách vòng 2 sinh đơn (260521 ckpt64) | 98.2% | 69.7% | 47.6% |
| Hoàn thiện ngân sách vòng 2 + rerank 4 ứng viên (260521 ckpt64) | **99.4%** | **80.6%** | **68.2%** |

![Sơ đồ hiệu quả Rerank hoàn thiện DPO](./images/Extra12-figures/10-dpo-rerank-closing.png)

Vì vậy không thể đơn giản nói "mẻ cuối cùng sinh đơn tốt nhất". Tôi sẽ ghi nhớ thế này:

- Sinh đơn tốt nhất: bản PlannerSoft Clean sinh đơn (260519 ckpt138).
- Sinh nhiều rerank tốt nhất: hoàn thiện ngân sách vòng 2 + rerank 4 ứng viên (260521 ckpt64).
- Bản chủ lực để trình diễn: 260521 ckpt64 rerank n4, 500 bản planner soft `80.6%`, hard split planner soft `77.0%`.

Sinh đơn nhìn vào chất lượng trung bình của một lần lấy mẫu; rerank nhìn vào trong hồ ứng viên có câu trả lời tốt hơn không, và quy tắc có chọn nó ra được không. Cả hai có thể không phải cùng một checkpoint.

---

## Chương 11: So sánh với tham chiếu bên ngoài MiMo như thế nào

Cuối cùng có thể thêm một mô hình mạnh bên ngoài làm tham chiếu, nhưng phần này nhất định phải viết rõ cách hiểu (caliber). MiMo không phải là checkpoint trong tuyến huấn luyện LoRA của chúng ta, cũng không phải leaderboard dưới nghiêm ngặt cùng một bộ script, cùng một phiên bản quy tắc. Cách dùng thích hợp hơn là: xem nó cho chúng ta biết mô hình mạnh đại khái sẽ mạnh ở đâu, chỗ nào không hoàn toàn ăn khớp với quy tắc cục bộ.

Tôi còn lấy API bên ngoài MiMo v2.5 Pro để làm đánh giá tham chiếu. Để tránh giới hạn độ dài đầu ra ảnh hưởng đến kết luận, lúc đó tôi đã phóng to max token lên một chút. Đặt chung với bản cục bộ cuối cùng để xem, đại khái như thế này:

| Mô hình | hardpass | planner soft | ngân sách tính lại soft | Sở thích ngân sách | Độ khớp ngân sách tính lại |
| --- | ---: | ---: | ---: | ---: | ---: |
| MiMo v2.5 Pro (phóng to max token) | 98.8% | 78.7% | **76.6%** | 85.5% | **82.4%** |
| Bản cục bộ cuối cùng (260521 ckpt64 rerank n4) | **99.4%** | **80.6%** | 68.2% | **86.0%** | 73.4% |

![Sơ đồ so sánh tham chiếu bên ngoài MiMo](./images/Extra12-figures/11-mimo-reference-comparison.png)

Bảng này tôi sẽ nhìn thế này:

- Bản cục bộ cuối cùng, dưới cách hiểu quy tắc của dự án này, `hardpass` và `planner soft` đã đuổi kịp và cao hơn một chút so với tham chiếu MiMo.
- Ngân sách tính lại soft và độ khớp ngân sách tính lại của MiMo vẫn mạnh hơn, cho thấy việc kiểm soát tổng ngân sách nó làm ổn định hơn.
- Quan hệ ngân sách và quy mô ẩm thực của MiMo trong báo cáo giai đoạn đầu không được cao lắm, chủ yếu là vì nó đưa ra chi phí ăn uống bình quân đầu người thực tế hơn, nhưng những chi phí ăn uống này đôi khi thấp hơn cận dưới của mức quy tắc hiện tại của chúng ta.

Vì vậy tôi sẽ không viết thành "vượt qua MiMo toàn diện". Cách nói ổn thỏa hơn là: dưới bộ đánh giá đóng băng và cách hiểu quy tắc của dự án này, planner soft của mô hình cục bộ cuối cùng đã ngang bằng với tham chiếu mô hình mạnh; độ khớp ngân sách vẫn còn khoảng cách, về sau nếu thực sự muốn tiếp tục làm, thì bổ sung sự phối hợp giữa kiểm soát tổng ngân sách và mức ngân sách (budget tier).


---

## Bad Case Gallery: Ba lỗi đáng giá nhất

Thứ hữu ích nhất trong post-training, thường không phải các mẫu thành công đẹp mắt nhất, mà là những bad case bị tát vào mặt hết lần này đến lần khác. Cuối cùng tôi giữ lại ba loại: tổng ngân sách sai, ẩm thực lặp lại hoặc không grounded, số đêm khách sạn và số phòng tính không đúng.

### 1. Tổng ngân sách sai: trông như sai sổ nhỏ, thực ra sẽ đâm thủng trực tiếp hard budget

Ví dụ đầu tiên là Tế Nam 3 ngày, 1 người đi công tác tiện thể chơi, hard budget 3200 tệ. Trường ngân sách trong đầu ra của mô hình trông như thế này:

```json
{
  "total_attractions": 240,
  "total_hotels": 2600,
  "total_meals": 1748,
  "total_transportation": 400,
  "total": 5088
}
```

Vấn đề có hai lớp.

Lớp thứ nhất là ngay số học đã sai: `240 + 2600 + 1748 + 400 = 4988`, nhưng mô hình viết thành `5088`. Lớp thứ hai phiền hơn, sau khi rule tính lại theo chi phí ăn uống mỗi ngày thì phát hiện, tổng ẩm thực thực tế lẽ ra là `1928`, không phải `1748`. Tức là, nó vừa viết sai các mục con, vừa viết sai tổng số, lại còn vượt qua hard budget 3200.

Cái rule bắt được là:

```text
budget_arithmetic_inconsistent: part_sum=4988, total=5088, diff=-100
meal_budget_inconsistent: expected_total_meals=1928, reported_total_meals=1748
budget_hard_constraint_exceeded: requested_budget=3200, total=5088
```

Về sau sửa thế nào? Tôi không tiếp tục kỳ vọng mô hình tự mình tính chuẩn hết mọi khoản, mà tách đánh giá ngân sách thành hai bộ: một bộ xem `budget.total` do mô hình khai báo, một bộ dùng điểm tham quan, khách sạn, ẩm thực, di chuyển để tính lại `recomputed_budget`. Trong huấn luyện và rerank cũng tách số học ngân sách, quan hệ ngân sách, độ khớp ngân sách ra để xem. Như vậy mô hình có thể tiếp tục phụ trách các lựa chọn và lịch trình, còn sổ sách thì để quy tắc đỡ đáy (bù trừ ở cuối).

### 2. Ẩm thực lặp lại: không phải không được ăn cùng một quán, mà là không được lười một mạch

Ví dụ thứ hai là Trường Sa 4 ngày, thân thiện với gia đình và người già. Mô hình quả thực đã tìm được nhà hàng, nhưng chọn quá lười:

```text
2026-03-08 lunch  新长福(世嘉店)
2026-03-08 dinner 新长福(世嘉店)
2026-03-10 dinner 新长福(世嘉店)
2026-03-11 dinner 新长福(世嘉店)
```

Cùng một ngày bữa trưa và bữa tối cùng một quán, sau đó lại tiếp tục lặp lại. Với mô hình đây có thể là "nhà hàng địa phương ổn thỏa", nhưng với người dùng, 4 ngày ăn đi ăn lại cùng một quán thì rất không giống một lịch trình được làm nghiêm túc.

Cái rule bắt được là:

```text
meal_same_day_lunch_dinner_repeat: 2026-03-08 lunch/dinner 都是新长福(世嘉店)
meal_repeat_too_many: name_key=新长福, count=4, max_allowed=3
meal_diversity_ok=False
```

Về sau sửa thế nào? Mảng ẩm thực này không thể chỉ viết "gợi ý nhà hàng đặc sản". Tôi thêm ba loại ràng buộc: bữa trưa và bữa tối cùng ngày không được lặp lại, cùng một họ nhà hàng không được vượt quá giới hạn trên, nhà hàng tốt nhất đến từ ứng viên công cụ chứ không phải viết chung chung "quán ăn vặt gần đó". Trong Best-of-N Replay và rerank cuối cùng, cũng đưa lặp lại ẩm thực, đa dạng và grounding vào tín hiệu sắp xếp. Thay đổi này rất thực dụng, vì nó không yêu cầu mô hình sinh ra câu trả lời hoàn hảo ngay lần đầu, chỉ cần trong hồ ứng viên có một phiên bản ít lặp lại hơn, rerank sẽ vớt nó ra được.

### 3. Số đêm khách sạn sai: số phòng còn dễ bị mô hình bỏ sót hơn số ngày

Ví dụ thứ ba là Bắc Kinh 3 ngày, 3 người bạn, ở khách sạn cao cấp. Lịch trình là 2026-01-07 đến 2026-01-09, nên cần 2 đêm; 3 người lớn thông thường cần 2 phòng. Khách sạn mỗi ngày trong đầu ra của mô hình như thế này:

```text
2026-01-07 华北宾馆 estimated_cost=1500
2026-01-08 华北宾馆 estimated_cost=1500
2026-01-09 无住宿
budget.total_hotels=3000
```

Thoạt nhìn không có vấn đề: 2 đêm, mỗi đêm 1500, tổng cộng 3000. Nhưng nó bỏ sót số phòng. 3 người bạn không phải 1 phòng ở 2 đêm, mà theo chiến lược lẽ ra phải tính 2 phòng ở 2 đêm, ngân sách khách sạn ít nhất phải bao phủ `1500 * 2 rooms * 2 nights = 6000`.

Cái rule bắt được là:

```text
hotel_budget_underestimated:
  lodging_nights=2
  party_total=3
  room_count=2
  expected_min_total_hotels=6000
  reported_total_hotels=3000
```

Về sau sửa thế nào? Loại vấn đề này không thể trông cậy mô hình "hiểu một chút về số người" trong ngôn ngữ tự nhiên. Front-end và back-end phải truyền tường minh `party.total`, lớp chiến lược back-end phải biên dịch ra `room_count` và `lodging_nights`, trong đánh giá thì dùng `hotel_budget_covers_nights` để bắt ra. Dữ liệu huấn luyện cũng không thể chỉ viết các mẫu đơn giản một người một phòng, nếu không mô hình sẽ mặc định "giá khách sạn = một phòng một đêm * số đêm", vừa gặp đi cùng bạn bè, gia đình, người già là bỏ sót sổ.

Ba bad case này về sau cơ bản trở thành thứ tự truy vết của tôi: trước tiên xem sổ có cộng đúng không, rồi xem ẩm thực có lười không, cuối cùng xem khách sạn có bao phủ theo số đêm và số phòng không. Rất nhiều vấn đề mô hình trông có vẻ huyền bí, tách đến đây thực ra đều khá mộc mạc.

---

## Chương 12: Những kinh nghiệm đọng lại từ thí nghiệm này

Làm lần này xong, điều tôi chắc chắn nhất là: post-training cho Agent không đơn giản như "tạo thêm chút dữ liệu rồi huấn luyện lại". Cái thực sự rắc rối là ghép cho khớp giao thức sản phẩm, dữ liệu, huấn luyện, đánh giá và chiến lược suy luận.

Cuối cùng những thứ tôi ghi lại không phức tạp:

1. **Trước tiên sửa giao thức sản phẩm.** Những trường có thể gửi lên dưới dạng có cấu trúc, đừng để mô hình đoán.
2. **Biên dịch ngữ cảnh cho tốt.** Mô hình nên lựa chọn dựa trên ứng viên, chứ không phải viết sự thật vô căn cứ.
3. **Đóng băng tập đánh giá trước.** Huấn luyện có thể đổi dữ liệu, đánh giá thì đừng mỗi vòng đổi đề.
4. **Gỡ lỗi Prompt dùng để tìm ranh giới.** Cái prompt giải quyết được thì viết vào prompt, cái giải quyết không được thì giao cho dữ liệu, quy tắc hoặc kỹ thuật.
5. **Mô hình mạnh sinh dữ liệu, nhưng teacher bắt buộc phải được kiểm định.** Tươm tất không đồng nghĩa với huấn luyện được.
6. **LoRA SFT đẩy tiến theo giai đoạn.** Mỗi vòng cố gắng chỉ giải quyết một loại vấn đề chính.
7. **Best-of-N Replay và Rerank cuối cùng phải phân biệt rõ.** Cái trước tạo dữ liệu huấn luyện, cái sau chọn câu trả lời lúc suy luận.
8. **DPO chỉ học sở thích giữa các ứng viên hợp lệ.** Đừng coi pair sửa định dạng là pair sở thích.
9. **Dữ liệu huấn luyện phải tránh tập đánh giá đóng băng.** Dự án thí nghiệm cũng chẳng cần học thuộc đề, kiểm tra chồng lấp rất rẻ.
10. **Chỉ số tách ra xem.** hardpass, planner soft, quan hệ ngân sách, ngân sách tính lại đừng nhào thành một điểm tổng.

Cuối cùng tôi sẽ khép lại thế này:

> Cái gì có thể cấu trúc hóa thì giao cho kỹ thuật, cái gì có thể quy tắc hóa thì làm thành đánh giá, phần còn lại thực sự cần mô hình học, thì mới đưa vào SFT hoặc huấn luyện sở thích.

Mô hình huấn luyện ra như vậy đương nhiên sẽ nói hay hơn, nhưng trọng điểm không nằm ở đó. Quan trọng hơn là, nó có thể ghép vào front-end/back-end; khi có vấn đề, quy tắc có thể định vị; vòng sau vẫn có thể tiếp tục sửa.

## Tài nguyên tái hiện

Nếu muốn tái hiện, bắt đầu từ những tài liệu này là được:

- Kho dự án Trợ lý Du lịch: [https://github.com/nameless0120/helloagents-trip-planner](https://github.com/nameless0120/helloagents-trip-planner)
- Giáo trình đầy đủ: [Giáo trình thực hành post-training cho Trợ lý Du lịch](https://github.com/nameless0120/helloagents-trip-planner/blob/main/training/docs/%E6%95%99%E7%A8%8B/%E6%97%85%E8%A1%8C%E5%8A%A9%E6%89%8B%E5%90%8E%E8%AE%AD%E7%BB%83%E5%AE%9E%E6%88%98%E6%95%99%E7%A8%8B.md)
- Dữ liệu đi kèm: `helloagents-后训练数据`, <https://pan.baidu.com/s/5oNsK7pwQnqzQEUg5ykb09Q>

Chi tiết phần cứng và script bài này không triển khai. Nói ngắn gọn, tuyến chính này phụ thuộc vào LoRA ngữ cảnh dài; nếu cắt ngắn ngữ cảnh, đổi sang QLoRA hoặc giảm nhỏ rank, cũng chạy được, nhưng khi đó không còn là tuyến đường trong bài viết nữa. Khi tái hiện thí nghiệm, hãy lấy cấu hình và bản lưu trữ trong kho `helloagents-trip-planner` làm chuẩn.
