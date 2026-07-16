# Chương 2 Lịch sử phát triển của Agent

Để hiểu sâu vì sao các Agent hiện đại lại có hình thái như ngày nay, cũng như nguồn gốc của những triết lý thiết kế cốt lõi, chương này sẽ ngược dòng lịch sử: bắt đầu từ thời kỳ kinh điển của trí tuệ nhân tạo, khám phá xem "trí tuệ" thuở sơ khai được định nghĩa như thế nào trong các hệ thống quy tắc dựa trên logic và ký hiệu; sau đó chứng kiến bước chuyển mình lớn từ mô hình trí tuệ đơn lẻ, tập trung sang tư duy trí tuệ phân tán, cộng tác; và cuối cùng hiểu được cách mà mô thức "học tập" đã hoàn toàn thay đổi cách Agent thu nhận năng lực, khai sinh ra những Agent hiện đại mà chúng ta thấy ngày nay.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-00.png" alt="Figure description" width="90%"/>
  <p>Hình 2.1 Nấc thang tiến hóa của AI Agent</p>
</div>

Như Hình 2.1 minh họa, **sự ra đời của mỗi mô thức mới đều nhằm giải quyết những "điểm đau" cốt lõi hoặc những hạn chế căn bản của mô thức thế hệ trước.** Trong khi các giải pháp mới mang lại bước nhảy vọt về năng lực, chúng cũng đồng thời tạo ra những "hạn chế" mới mà tại thời điểm đó rất khó vượt qua, và những hạn chế này lại đặt nền móng cho sự ra đời của mô thức thế hệ tiếp theo. Việc hiểu quá trình lặp lại "được thúc đẩy bởi vấn đề" này giúp chúng ta nắm bắt sâu sắc hơn những lý do sâu xa và tính tất yếu lịch sử đằng sau các lựa chọn công nghệ Agent hiện đại.

## 2.1 Các Agent thuở sơ khai dựa trên Ký hiệu và Logic

Những khám phá ban đầu trong lĩnh vực trí tuệ nhân tạo chịu ảnh hưởng sâu sắc từ logic toán học và các nguyên lý nền tảng của khoa học máy tính. Ở thời đại đó, các nhà nghiên cứu nhìn chung đều giữ một niềm tin: trí tuệ con người, đặc biệt là khả năng suy luận logic, có thể được nắm bắt và tái hiện bằng các hệ thống ký hiệu được hình thức hóa. Ý tưởng cốt lõi này đã khai sinh ra mô thức quan trọng đầu tiên của trí tuệ nhân tạo — Chủ nghĩa ký hiệu (Symbolicism), còn được gọi là "Logic AI" hay "AI truyền thống".

Theo quan điểm của chủ nghĩa ký hiệu, cốt lõi của hành vi thông minh là thao tác trên các ký hiệu dựa trên một tập hợp quy tắc rõ ràng. Do đó, một Agent có thể được xem như một hệ thống ký hiệu vật lý: nó biểu diễn thế giới bên ngoài thông qua các ký hiệu nội tại và hoạch định hành động thông qua suy luận logic. "Trí tuệ" của các Agent trong thời đại này hoàn toàn đến từ cơ sở tri thức và các quy tắc suy luận được lập trình sẵn bởi những người thiết kế, chứ không phải được thu nhận qua học tập tự chủ.

### 2.1.1 Giả thuyết Hệ thống Ký hiệu Vật lý

Nền tảng lý thuyết của thời đại chủ nghĩa ký hiệu là **Giả thuyết Hệ thống Ký hiệu Vật lý (Physical Symbol System Hypothesis, PSSH)**<sup>[1]</sup>, được **Allen Newell** và **Herbert A. Simon** cùng đề xuất năm 1976. Hai nhà khoa học đoạt giải Turing này đã cung cấp định hướng lý thuyết và tiêu chí để hiện thực hóa trí tuệ nhân tạo tổng quát trên máy tính thông qua giả thuyết này.

Giả thuyết bao gồm hai luận điểm cốt lõi:

1. **Luận điểm về tính đầy đủ (Sufficiency Assertion)**: Bất kỳ hệ thống ký hiệu vật lý nào cũng có đủ phương tiện để tạo ra hành vi thông minh tổng quát.
2. **Luận điểm về tính cần thiết (Necessity Assertion)**: Bất kỳ hệ thống nào có khả năng thể hiện hành vi thông minh tổng quát về bản chất đều phải là một hệ thống ký hiệu vật lý.

Một hệ thống ký hiệu vật lý ở đây chỉ một hệ thống có thể tồn tại trong thế giới vật lý, được cấu thành từ một tập hợp các ký hiệu phân biệt được và một chuỗi các tiến trình thao tác trên những ký hiệu này, với các thành phần cấu thành như minh họa trong Hình 2.2. Các ký hiệu này có thể được kết hợp thành những cấu trúc phức tạp hơn (như các biểu thức), trong khi các tiến trình có thể tạo ra, sửa đổi, sao chép và hủy bỏ những cấu trúc ký hiệu này.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-0.png" alt="Figure description" width="90%"/>
  <p>Hình 2.2 Các thành phần cấu thành của một hệ thống ký hiệu vật lý</p>
</div>

Nói ngắn gọn, PSSH đã mạnh dạn tuyên bố: **Bản chất của trí tuệ là sự tính toán và xử lý các ký hiệu.**

Giả thuyết này có ảnh hưởng sâu rộng. Nó đã biến việc nghiên cứu bài toán triết học mơ hồ và phức tạp về tâm trí con người thành một bài toán cụ thể có thể được kỹ thuật hóa và hiện thực hóa trên máy tính. Nó gieo vào lòng các nhà nghiên cứu trí tuệ nhân tạo thời kỳ đầu một niềm tin mạnh mẽ rằng chỉ cần tìm ra cách đúng đắn để biểu diễn tri thức và thiết kế các thuật toán suy luận hiệu quả, chúng ta chắc chắn có thể tạo ra trí tuệ máy sánh ngang con người. Gần như tất cả các nghiên cứu trong thời đại chủ nghĩa ký hiệu, từ hệ chuyên gia đến hoạch định tự động, đều được tiến hành dưới sự dẫn dắt của giả thuyết này.

### 2.1.2 Hệ chuyên gia

Dưới ảnh hưởng trực tiếp của giả thuyết hệ thống ký hiệu vật lý, **Hệ chuyên gia (Expert Systems)** đã trở thành thành tựu ứng dụng quan trọng và thành công nhất của thời đại chủ nghĩa ký hiệu. Mục tiêu cốt lõi của hệ chuyên gia là mô phỏng khả năng giải quyết vấn đề của các chuyên gia con người trong những lĩnh vực cụ thể. Bằng cách mã hóa tri thức và kinh nghiệm của chuyên gia thành các chương trình máy tính, chúng có thể đưa ra kết luận hoặc khuyến nghị sánh ngang, thậm chí vượt qua các chuyên gia con người khi đối mặt với những vấn đề tương tự.

Một hệ chuyên gia điển hình thường bao gồm một số thành phần cốt lõi như cơ sở tri thức, động cơ suy luận và giao diện người dùng, với kiến trúc tổng quát như minh họa trong Hình 2.3.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-1.png" alt="Figure description" width="90%"/>
  <p>Hình 2.3 Kiến trúc tổng quát của hệ chuyên gia</p>
</div>

Kiến trúc này thể hiện rõ triết lý thiết kế tách biệt tri thức khỏi suy luận, một đặc điểm quan trọng của AI theo chủ nghĩa ký hiệu.

**Cơ sở tri thức và Động cơ suy luận**

"Trí tuệ" của hệ chuyên gia chủ yếu đến từ hai thành phần cốt lõi của nó: cơ sở tri thức và động cơ suy luận.

- **Cơ sở tri thức (Knowledge Base)**: Đây là trung tâm lưu trữ tri thức của hệ chuyên gia, dùng để lưu trữ tri thức và kinh nghiệm của chuyên gia trong lĩnh vực. **Biểu diễn tri thức (Knowledge Representation)** là chìa khóa để xây dựng cơ sở tri thức. Trong hệ chuyên gia, phương pháp biểu diễn tri thức được sử dụng phổ biến nhất là **Quy tắc sản xuất (Production Rules)**, tức một chuỗi các câu điều kiện dưới dạng "IF-THEN". Ví dụ: IF bệnh nhân có triệu chứng sốt AND ho THEN có thể bị nhiễm trùng đường hô hấp. Những quy tắc này liên kết các tình huống cụ thể (phần IF, điều kiện) với các kết luận hoặc hành động tương ứng (phần THEN, kết luận). Một hệ chuyên gia phức tạp có thể chứa hàng trăm hoặc hàng nghìn quy tắc như vậy, cùng nhau tạo thành một mạng lưới tri thức khổng lồ.
- **Động cơ suy luận (Inference Engine)**: Động cơ suy luận là động cơ tính toán cốt lõi của hệ chuyên gia. Nó là một chương trình tổng quát có nhiệm vụ tìm và áp dụng các quy tắc liên quan trong cơ sở tri thức dựa trên các sự kiện do người dùng cung cấp, từ đó rút ra các kết luận mới. Động cơ suy luận chủ yếu hoạt động theo hai cách:
  - **Suy diễn tiến (Forward Chaining)**: Bắt đầu từ các sự kiện đã biết, liên tục đối chiếu phần IF của các quy tắc, kích hoạt các kết luận ở phần THEN, và bổ sung các kết luận mới vào cơ sở sự kiện cho đến khi cuối cùng rút ra được mục tiêu hoặc không còn quy tắc mới nào được đối chiếu. Đây là phương pháp suy luận "được thúc đẩy bởi dữ liệu".
  - **Suy diễn lùi (Backward Chaining)**: Bắt đầu từ một mục tiêu giả định (chẳng hạn "bệnh nhân có bị viêm phổi hay không"), tìm các quy tắc có thể rút ra mục tiêu đó, sau đó lấy phần IF của quy tắc đó làm mục tiêu con mới, đệ quy theo cách này cho đến khi tất cả các mục tiêu con đều có thể được chứng minh bằng các sự kiện đã biết. Đây là phương pháp suy luận "được thúc đẩy bởi mục tiêu".

**Trường hợp ứng dụng và Phân tích: Hệ thống MYCIN**

MYCIN là một trong những hệ chuyên gia nổi tiếng và có ảnh hưởng nhất trong lịch sử, được Đại học Stanford phát triển vào những năm 1970<sup>[2]</sup>. Nó được thiết kế để hỗ trợ các bác sĩ chẩn đoán nhiễm trùng máu do vi khuẩn và khuyến nghị các phác đồ điều trị kháng sinh phù hợp.

- **Nguyên lý hoạt động**: MYCIN thu thập triệu chứng, tiền sử bệnh và kết quả xét nghiệm của bệnh nhân thông qua tương tác hỏi-đáp với bác sĩ. Cơ sở tri thức của nó chứa khoảng 600 quy tắc "IF-THEN" do các chuyên gia y tế cung cấp. Động cơ suy luận chủ yếu hoạt động theo lối suy diễn lùi: bắt đầu từ mục tiêu cao nhất là "xác định tác nhân gây bệnh", nó suy ngược lại xem cần những bằng chứng và điều kiện nào, sau đó đặt câu hỏi cho bác sĩ để thu thập thông tin này. Quy trình làm việc đơn giản hóa của nó được minh họa trong Hình 2.4.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-2.png" alt="Figure description" width="90%"/>
  <p>Hình 2.4 Sơ đồ minh họa quá trình suy diễn lùi của MYCIN</p>
</div>

- **Xử lý sự không chắc chắn**: Chẩn đoán y khoa đầy rẫy sự không chắc chắn. Một đổi mới quan trọng của MYCIN là đưa vào khái niệm **Hệ số chắc chắn (Certainty Factor, CF)**, sử dụng một giá trị số nằm giữa -1 và 1 để biểu diễn độ tin cậy của một kết luận. Điều này cho phép hệ thống xử lý tri thức y khoa không chắc chắn, mơ hồ và đưa ra các kết quả chẩn đoán kèm đánh giá độ tin cậy, gần gũi với thế giới thực hơn so với logic Boolean đơn thuần.
- **Thành tựu và Ý nghĩa**: Trong một cuộc đánh giá, hiệu suất của MYCIN trong chẩn đoán nhiễm trùng máu vượt qua các bác sĩ không chuyên khoa và thậm chí đạt tới trình độ của các chuyên gia con người. Thành công của nó đã chứng minh hùng hồn tính hiệu lực của giả thuyết hệ thống ký hiệu vật lý: thông qua kỹ thuật tri thức tỉ mỉ và suy luận ký hiệu, máy móc quả thực có thể thể hiện "trí tuệ" xuất sắc trong những lĩnh vực chuyên môn có độ phức tạp cao. MYCIN không chỉ là một cột mốc trong lịch sử phát triển của hệ chuyên gia mà còn mở đường cho các ứng dụng thương mại của trí tuệ nhân tạo trong nhiều lĩnh vực dọc khác nhau sau này.

### 2.1.3 SHRDLU

Nếu như hệ chuyên gia thể hiện "chiều sâu" của AI ký hiệu trong các lĩnh vực chuyên môn, thì dự án SHRDLU<sup>[3]</sup> do **Terry Winograd** phát triển trong giai đoạn 1968-1970 đã đạt được một bước đột phá mang tính cách mạng về "chiều rộng". Như minh họa trong Hình 2.5, SHRDLU hướng tới xây dựng một Agent thông minh toàn diện có thể tương tác trôi chảy với con người thông qua ngôn ngữ tự nhiên trong môi trường vi mô của "thế giới khối hộp" (blocks world). "Thế giới khối hộp" là một không gian ảo ba chiều được mô phỏng chứa các khối hộp có hình dạng, màu sắc và kích thước khác nhau, cùng với một cánh tay robot ảo có thể gắp và di chuyển chúng. Người dùng ra lệnh hoặc đặt câu hỏi cho SHRDLU bằng ngôn ngữ tự nhiên, và SHRDLU thực hiện hành động trong thế giới ảo hoặc đưa ra phản hồi bằng văn bản.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-3.png" alt="Figure description" width="90%"/>
  <p>Hình 2.5 Giao diện tương tác "thế giới khối hộp" của SHRDLU</p>
</div>

SHRDLU thu hút sự chú ý rộng rãi vào thời điểm đó chủ yếu vì nó là hệ thống đầu tiên tích hợp nhiều module trí tuệ nhân tạo độc lập (như phân tích ngôn ngữ, hoạch định, ghi nhớ) vào một hệ thống thống nhất và khiến chúng phối hợp làm việc với nhau:

- **Hiểu ngôn ngữ tự nhiên**: SHRDLU có thể phân tích các câu tiếng Anh có cấu trúc phức tạp và mơ hồ. Nó không chỉ hiểu được các mệnh lệnh trực tiếp (như `Pick up a big red block.`) mà còn xử lý được các chỉ dẫn phức tạp hơn, chẳng hạn:
  - Phân giải tham chiếu: `Find a block which is taller than the one you are holding and put it into the box.` Trong chỉ dẫn này, hệ thống cần hiểu rằng `the one you are holding` chỉ đối tượng đang được cánh tay robot gắp giữ.
  - Ghi nhớ ngữ cảnh: Người dùng có thể nói `Grasp the pyramid.`, rồi hỏi `What does the box contain?`, và hệ thống có thể trả lời bằng cách liên kết ngữ cảnh.
- **Hoạch định và Hành động**: Sau khi hiểu chỉ dẫn, SHRDLU có thể tự chủ hoạch định một chuỗi các hành động cần thiết để hoàn thành nhiệm vụ. Ví dụ, nếu chỉ dẫn là "đặt khối màu xanh dương lên khối màu đỏ", và trên khối màu đỏ đã có sẵn một khối màu xanh lá khác, hệ thống sẽ hoạch định chuỗi hành động "trước tiên dời khối xanh lá đi, sau đó đặt khối xanh dương lên".
- **Ghi nhớ và Hỏi-đáp**: SHRDLU có ghi nhớ về môi trường của nó và về hành vi của chính nó. Người dùng có thể đặt câu hỏi về điều này, chẳng hạn:
  - Hỏi về trạng thái thế giới: `Is there a large block behind a pyramid?`
  - Hỏi về lịch sử hành vi: `Did you touch any pyramid before you put the green one on the little cube?`
  - Hỏi về động cơ hành vi: `Why did you pick up the red block?` SHRDLU có thể trả lời: `BECAUSE YOU ASKED ME TO.`

Vị thế lịch sử và ảnh hưởng của SHRDLU chủ yếu thể hiện ở ba khía cạnh:

- **Mô hình mẫu về trí tuệ toàn diện**: Trước SHRDLU, nghiên cứu AI phần lớn tập trung vào các chức năng đơn lẻ. Nó là hệ thống đầu tiên tích hợp nhiều module AI như hiểu ngôn ngữ, suy luận hoạch định và ghi nhớ hành động vào một hệ thống thống nhất. Thiết kế vòng lặp khép kín "cảm nhận - suy nghĩ - hành động" của nó đã đặt nền móng cho nghiên cứu Agent hiện đại.
- **Phổ biến phương pháp nghiên cứu thế giới vi mô**: Thành công của nó đã chứng minh tính khả thi của việc khám phá và kiểm chứng các nguyên lý cơ bản của những Agent phức tạp trong một môi trường được đơn giản hóa với các quy tắc rõ ràng, một phương pháp đã ảnh hưởng sâu sắc đến các nghiên cứu robot học và hoạch định AI sau này.
- **Khơi dậy sự lạc quan và Suy ngẫm**: Thành công của SHRDLU đã khơi dậy những kỳ vọng lạc quan ban đầu về AGI, nhưng năng lực của nó bị giới hạn nghiêm ngặt trong thế giới khối hộp. Hạn chế này đã khơi mào cho những suy ngẫm lâu dài trong lĩnh vực AI về sự khác biệt giữa "xử lý ký hiệu" và "hiểu biết thực sự", phơi bày những thách thức sâu sắc trên con đường đến trí tuệ tổng quát.

### 2.1.4 Những thách thức căn bản mà Chủ nghĩa ký hiệu đối mặt

Mặc dù đạt được những thành tựu đáng kể trong các dự án thời kỳ đầu, bắt đầu từ những năm 1980, AI ký hiệu đã vấp phải những khó khăn căn bản vốn có trong phương pháp luận của nó khi chuyển từ "thế giới vi mô" sang thế giới thực mở và phức tạp. Những khó khăn này chủ yếu có thể được tổng hợp thành hai loại lớn:

**(1) Tri thức thường thức và Nút thắt cổ chai trong thu nhận tri thức**

"Trí tuệ" của các Agent ký hiệu hoàn toàn phụ thuộc vào chất lượng và tính đầy đủ của cơ sở tri thức của chúng. Tuy nhiên, việc xây dựng một cơ sở tri thức có thể hỗ trợ tương tác với thế giới thực đã chứng tỏ là một nhiệm vụ vô cùng gian nan, chủ yếu thể hiện ở hai khía cạnh:

- **Nút thắt cổ chai trong thu nhận tri thức**: Tri thức của hệ chuyên gia cần được các chuyên gia con người và các kỹ sư tri thức xây dựng thông qua những quy trình phỏng vấn, chắt lọc và mã hóa tẻ nhạt. Quá trình này tốn kém, mất thời gian và khó mở rộng quy mô. Quan trọng hơn, phần lớn tri thức của chuyên gia con người là ngầm ẩn và mang tính trực giác, khó có thể được diễn đạt rõ ràng thành các quy tắc "IF-THEN". Việc cố gắng ký hiệu hóa thủ công toàn bộ tri thức của cả thế giới được xem là một nhiệm vụ gần như bất khả thi.
- **Vấn đề thường thức (Common-sense Problem)**: Hành vi của con người dựa trên một nền tảng thường thức khổng lồ (ví dụ "nước thì ướt", "dây thừng có thể kéo nhưng không thể đẩy"), nhưng các hệ thống ký hiệu hoàn toàn không biết gì về điều này trừ khi được mã hóa một cách tường minh. Việc thiết lập một cơ sở tri thức hoàn chỉnh cho thường thức rộng lớn và mơ hồ vẫn là một thách thức lớn cho đến tận ngày nay. Dự án Cyc<sup>[4]</sup>, sau nhiều thập kỷ nỗ lực, vẫn có kết quả và ứng dụng rất hạn chế.

**(2) Vấn đề khung (Frame Problem) và Tính giòn của hệ thống**

Bên cạnh những thách thức ở tầng tri thức, chủ nghĩa ký hiệu còn vấp phải những nan đề logic khi xử lý một thế giới biến đổi động.

- **Vấn đề khung (Frame Problem)**: Trong một thế giới động, làm thế nào để xác định một cách hiệu quả những gì đã không thay đổi sau khi một Agent thực hiện một hành động là một câu đố logic<sup>[5]</sup>. Việc khai báo tường minh tất cả các trạng thái bất biến cho mỗi hành động là bất khả thi về mặt tính toán, thế nhưng con người lại có thể dễ dàng bỏ qua những thay đổi không liên quan mà chẳng tốn công sức.
- **Tính giòn (Brittleness)**: Các hệ thống ký hiệu hoàn toàn dựa vào các quy tắc định trước, khiến hành vi của chúng rất "giòn". Một khi gặp bất kỳ thay đổi nhỏ nào hay tình huống mới nằm ngoài các quy tắc, hệ thống có thể hoàn toàn thất bại, không thể thích ứng linh hoạt như con người. Thành công của SHRDLU chính là bởi nó hoạt động trong một thế giới khép kín với bộ quy tắc hoàn chỉnh, trong khi thế giới thực đầy rẫy những ngoại lệ.

## 2.2 Xây dựng Chatbot dựa trên Quy tắc

Sau khi khám phá những thách thức lý thuyết của chủ nghĩa ký hiệu, trong phần này chúng ta sẽ trải nghiệm một cách trực quan cách các hệ thống dựa trên quy tắc hoạt động thông qua một bài thực hành lập trình cụ thể. Chúng ta sẽ thử tái hiện ELIZA, một chatbot thời kỳ đầu vô cùng có ảnh hưởng trong lịch sử trí tuệ nhân tạo.

### 2.2.1 Triết lý thiết kế của ELIZA

ELIZA là một chương trình máy tính được nhà khoa học máy tính của MIT là **Joseph Weizenbaum** phát hành năm 1966<sup>[6]</sup>, một trong những nỗ lực nổi tiếng thời kỳ đầu trong lĩnh vực xử lý ngôn ngữ tự nhiên. ELIZA không phải là một chương trình đơn lẻ mà là một framework có thể chạy các "kịch bản" (script) khác nhau. Trong đó, kịch bản được biết đến rộng rãi và thành công nhất là "DOCTOR", mô phỏng một nhà trị liệu tâm lý theo trường phái Rogers phi định hướng.

Cách thức hoạt động của ELIZA vô cùng khéo léo: nó không bao giờ trực tiếp trả lời câu hỏi hay cung cấp thông tin mà nhận diện các từ khóa trong đầu vào của người dùng, sau đó áp dụng một tập hợp các quy tắc biến đổi định trước để chuyển các câu nói của người dùng thành những câu hỏi mở. Ví dụ, khi người dùng nói "I am sad about my boyfriend" (Tôi buồn về bạn trai của mình), ELIZA có thể nhận diện từ khóa "I am sad about..." và áp dụng một quy tắc để sinh ra phản hồi: "Why are you sad about your boyfriend?" (Tại sao bạn lại buồn về bạn trai của mình?).

Triết lý thiết kế của Weizenbaum không phải là tạo ra một Agent có thể thực sự "hiểu" cảm xúc của con người; trái lại, ông muốn chứng minh rằng thông qua một số kỹ thuật biến đổi câu đơn giản, máy móc có thể tạo ra một ảo giác về "trí tuệ" và "sự đồng cảm" mà hoàn toàn không hiểu gì về nội dung cuộc trò chuyện. Tuy nhiên, điều khiến ông ngạc nhiên là nhiều người tương tác với ELIZA (bao gồm cả cô thư ký của ông) đã nảy sinh sự phụ thuộc về mặt cảm xúc với nó, tin sâu sắc rằng nó có thể hiểu họ.

Mục tiêu thực hành của phần này là tái hiện cơ chế cốt lõi của ELIZA để hiểu sâu về những ưu điểm và hạn chế căn bản của phương pháp dựa trên quy tắc này.

### 2.2.2 Đối chiếu mẫu và Thay thế văn bản

Luồng thuật toán của ELIZA dựa trên **Đối chiếu mẫu và Thay thế văn bản (Pattern Matching and Text Substitution)**, có thể được phân rã rõ ràng thành bốn bước sau:

1. **Nhận diện và Xếp hạng từ khóa:** Cơ sở quy tắc thiết lập một mức độ ưu tiên cho mỗi từ khóa (như `mother`, `dreamed`, `depressed`). Khi đầu vào chứa nhiều từ khóa, chương trình chọn quy tắc tương ứng với từ khóa có mức ưu tiên cao nhất để xử lý.
2. **Quy tắc phân rã:** Sau khi tìm thấy một từ khóa, chương trình sử dụng các quy tắc phân rã với ký tự đại diện (`*`) để bắt lấy phần còn lại của câu.
   1. **Ví dụ quy tắc**: `* my *`
   2. **Đầu vào người dùng**: `"My mother is afraid of me"`
   3. **Kết quả bắt được**: `["", "mother is afraid of me"]`
3. **Quy tắc lắp ráp lại:** Chương trình chọn một trong một tập hợp các quy tắc lắp ráp lại được liên kết với quy tắc phân rã để sinh ra phản hồi (thường được chọn ngẫu nhiên để tăng tính đa dạng), và tùy chọn sử dụng nội dung đã bắt được ở bước trước.
   1. **Ví dụ quy tắc**: `"Tell me more about your family."`
   2. **Đầu ra được sinh ra**: `"Tell me more about your family."`
4. **Chuyển đổi đại từ:** Trước khi lắp ráp lại, chương trình thực hiện chuyển đổi đại từ đơn giản (như `I` → `you`, `my` → `your`) để duy trì tính mạch lạc của cuộc trò chuyện.

Toàn bộ quy trình làm việc có thể được biểu diễn bằng một ý tưởng mã giả đơn giản:

```Python
FUNCTION generate_response(user_input):
    // 1. Split user input into words
    words = SPLIT(user_input)

    // 2. Find the highest priority keyword rule
    best_rule = FIND_BEST_RULE(words)
    IF best_rule is NULL:
        RETURN a_generic_response() // For example: "Please go on."

    // 3. Use rule to decompose user input
    decomposed_parts = DECOMPOSE(user_input, best_rule.decomposition_pattern)
    IF decomposition_failed:
        RETURN a_generic_response()

    // 4. Perform pronoun conversion on decomposed parts
    transformed_parts = TRANSFORM_PRONOUNS(decomposed_parts)

    // 5. Use reassembly rules to generate response
    response = REASSEMBLE(transformed_parts, best_rule.reassembly_patterns)

    RETURN response
```

Thông qua cơ chế này, ELIZA đã thành công đơn giản hóa bài toán hiểu ngôn ngữ tự nhiên phức tạp thành một trò chơi đối chiếu mẫu dựa trên quy tắc, có thể thao tác được.

### 2.2.3 Hiện thực hóa Logic cốt lõi

Bây giờ, chúng ta sẽ chuyển các nguyên lý kỹ thuật được mô tả ở phần trước thành một hàm Python đơn giản, có thể chạy được. Đoạn mã sau đây hiện thực hóa một phiên bản thu nhỏ của ELIZA, chứa một tập hợp quy tắc nhỏ nhưng đủ để minh họa cơ chế hoạt động cốt lõi của nó.

```Python
import re
import random

# Định nghĩa cơ sở quy tắc: mẫu (biểu thức chính quy) -> danh sách mẫu phản hồi
rules = {
    r'I need (.*)': [
        "Why do you need {0}?",
        "Would it really help you to get {0}?",
        "Are you sure you need {0}?"
    ],
    r'Why don\'t you (.*)\?': [
        "Do you really think I don't {0}?",
        "Perhaps eventually I will {0}.",
        "Do you really want me to {0}?"
    ],
    r'Why can\'t I (.*)\?': [
        "Do you think you should be able to {0}?",
        "If you could {0}, what would you do?",
        "I don't know -- why can't you {0}?"
    ],
    r'I am (.*)': [
        "Did you come to me because you are {0}?",
        "How long have you been {0}?",
        "How do you feel about being {0}?"
    ],
    r'.* mother .*': [
        "Tell me more about your mother.",
        "What was your relationship with your mother like?",
        "How do you feel about your mother?"
    ],
    r'.* father .*': [
        "Tell me more about your father.",
        "How did your father make you feel?",
        "What has your father taught you?"
    ],
    r'.*': [
        "Please tell me more.",
        "Let's change focus a bit... Tell me about your family.",
        "Can you elaborate on that?"
    ]
}

# Định nghĩa quy tắc chuyển đổi đại từ
pronoun_swap = {
    "i": "you", "you": "i", "me": "you", "my": "your",
    "am": "are", "are": "am", "was": "were", "i'd": "you would",
    "i've": "you have", "i'll": "you will", "yours": "mine",
    "mine": "yours"
}

def swap_pronouns(phrase):
    """
    Thực hiện chuyển đổi ngôi thứ nhất/ngôi thứ hai cho các đại từ trong cụm từ đầu vào
    """
    words = phrase.lower().split()
    swapped_words = [pronoun_swap.get(word, word) for word in words]
    return " ".join(swapped_words)

def respond(user_input):
    """
    Sinh phản hồi dựa trên cơ sở quy tắc
    """
    for pattern, responses in rules.items():
        match = re.search(pattern, user_input, re.IGNORECASE)
        if match:
            # Bắt lấy phần đã khớp
            captured_group = match.group(1) if match.groups() else ''
            # Thực hiện chuyển đổi đại từ
            swapped_group = swap_pronouns(captured_group)
            # Chọn ngẫu nhiên một mẫu và định dạng
            response = random.choice(responses).format(swapped_group)
            return response
    # Nếu không có quy tắc cụ thể nào khớp, dùng quy tắc ký tự đại diện cuối cùng
    return random.choice(rules[r'.*'])

# Vòng lặp trò chuyện chính
if __name__ == '__main__':
    print("Therapist: Hello! How can I help you today?")
    while True:
        user_input = input("You: ")
        if user_input.lower() in ["quit", "exit", "bye"]:
            print("Therapist: Goodbye. It was nice talking to you.")
            break
        response = respond(user_input)
        print(f"Therapist: {response}")

>>>
Therapist: Hello! How can I help you today?
You: I am feeling sad today.
Therapist: How long have you been feeling sad today?
You: I need some help with my project.
Therapist: Are you sure you need some help with your project?
You: My mother is not happy with my work.
Therapist: Tell me more about your mother.
You: quit
Therapist: Goodbye. It was nice talking to you.
```

Thông qua bài thực hành lập trình trên, chúng ta có thể tổng kết một cách trực quan những hạn chế căn bản của các hệ thống dựa trên quy tắc, và đây chính là những minh chứng trực tiếp cho các thách thức lý thuyết của chủ nghĩa ký hiệu đã thảo luận ở Mục `2.1.4`:

- **Thiếu khả năng hiểu ngữ nghĩa**: Hệ thống không hiểu nghĩa của từ. Ví dụ, khi đối mặt với đầu vào "I am **not** happy" (Tôi **không** vui), nó vẫn sẽ máy móc khớp với quy tắc `I am (.*)` và sinh ra một phản hồi sai về mặt ngữ nghĩa vì nó không thể hiểu vai trò của từ phủ định "not".
- **Không có ghi nhớ ngữ cảnh**: Hệ thống là **phi trạng thái (stateless)**, mỗi phản hồi chỉ dựa trên câu đầu vào đơn lẻ hiện tại, không thể tiến hành các cuộc trò chuyện nhiều lượt mạch lạc.
- **Vấn đề khả năng mở rộng của quy tắc**: Việc cố gắng thêm nhiều quy tắc hơn dẫn đến sự tăng trưởng bùng nổ về kích thước của cơ sở quy tắc, và việc quản lý xung đột cũng như xử lý mức ưu tiên giữa các quy tắc trở nên vô cùng phức tạp, cuối cùng khiến hệ thống trở nên khó bảo trì.

Tuy nhiên, dù có những khiếm khuyết rõ ràng này, ELIZA đã tạo ra "**hiệu ứng ELIZA (ELIZA effect)**" nổi tiếng vào thời điểm đó, với nhiều người dùng tin rằng nó có thể hiểu họ. Ảo giác về trí tuệ này chủ yếu bắt nguồn từ các chiến lược trò chuyện khéo léo của nó (như đóng vai một người đặt câu hỏi thụ động, sử dụng các mẫu câu mở) và tâm lý phóng chiếu cảm xúc bẩm sinh của con người.

Bài thực hành ELIZA đã phơi bày rõ ràng mâu thuẫn cốt lõi của phương pháp chủ nghĩa ký hiệu: màn thể hiện có vẻ thông minh của hệ thống hoàn toàn phụ thuộc vào các quy tắc được lập trình sẵn bởi những người thiết kế. Tuy nhiên, khi đối mặt với vô số khả năng của ngôn ngữ trong thế giới thực, phương pháp liệt kê tường tận này chắc chắn sẽ không thể mở rộng quy mô. Hệ thống không có sự hiểu biết thực sự, mà chỉ thực thi các thao tác ký hiệu, và đây chính là gốc rễ của tính giòn của nó.

## 2.3 Học thuyết Xã hội của Tâm trí của Marvin Minsky

Sự khám phá chủ nghĩa ký hiệu và bài thực hành ELIZA cùng chỉ ra một vấn đề: một động cơ suy luận đơn lẻ, tập trung được xây dựng thông qua các quy tắc định trước dường như khó có thể dẫn đến trí tuệ thực sự. Bất kể cơ sở quy tắc lớn đến đâu, hệ thống luôn tỏ ra cứng nhắc và giòn khi đối mặt với sự mơ hồ, phức tạp và những biến đổi vô tận của thế giới thực. Nan đề này đã thúc đẩy một số nhà tư tưởng hàng đầu suy ngẫm về triết lý thiết kế căn bản nhất của trí tuệ nhân tạo. Trong số đó, **Marvin Minsky** đã không tiếp tục cố gắng thêm nhiều quy tắc hơn vào một lõi suy luận đơn lẻ mà đề xuất một câu hỏi mang tính cách mạng trong cuốn sách **"Xã hội của Tâm trí" (The Society of Mind)**<sup>[7]</sup> của mình: "Thủ thuật kỳ diệu nào khiến chúng ta trở nên thông minh? Thủ thuật đó là chẳng có thủ thuật nào cả. Sức mạnh của trí tuệ bắt nguồn từ sự đa dạng vô cùng của chúng ta, chứ không phải từ bất kỳ một nguyên lý đơn lẻ, hoàn hảo nào."

### 2.3.1 Suy ngẫm về các Mô hình trí tuệ đơn nhất, chỉnh thể

Từ những năm 1970 đến những năm 1980, những hạn chế của chủ nghĩa ký hiệu ngày càng trở nên rõ ràng. Dù hệ chuyên gia đạt được thành công trong các lĩnh vực dọc chuyên biệt cao, chúng vẫn không thể sở hữu thường thức như một đứa trẻ; dù SHRDLU có thể thể hiện xuất sắc trong thế giới khối hộp khép kín, nó vẫn không thể hiểu bất cứ điều gì bên ngoài thế giới đó; dù ELIZA có thể bắt chước trò chuyện, nó vẫn hoàn toàn không biết gì về chính nội dung cuộc trò chuyện. Những hệ thống này đều tuân theo một phương pháp thiết kế từ trên xuống (top-down): một bộ xử lý trung tâm toàn tri xử lý thông tin và ra quyết định theo một tập hợp quy tắc logic thống nhất.

Đối mặt với sự thất bại phổ biến này, Minsky bắt đầu đặt ra một loạt câu hỏi căn bản:

- **"Sự hiểu biết" là gì?** Khi chúng ta nói rằng chúng ta hiểu một câu chuyện, đây có phải là một năng lực đơn lẻ không? Hay thực ra nó là kết quả của hàng chục tiến trình tinh thần khác nhau cùng làm việc với nhau, chẳng hạn như khả năng hình dung, khả năng suy luận logic, khả năng cộng hưởng cảm xúc, và thường thức về các mối quan hệ xã hội?
- **"Thường thức" là gì?** Thường thức có phải là một cơ sở tri thức khổng lồ chứa hàng triệu quy tắc logic (như dự án Cyc đã thử) không? Hay nó là một mạng lưới phân tán được đan dệt từ vô số kinh nghiệm cụ thể và các mảnh quy tắc đơn giản?
- **Nên xây dựng Agent như thế nào?** Chúng ta nên tiếp tục theo đuổi một hệ thống logic hoàn hảo, thống nhất, hay nên thừa nhận rằng bản thân trí tuệ là một mớ hỗn tạp "không hoàn hảo" được cấu thành từ nhiều bộ phận đơn giản khác nhau về chức năng, thậm chí mâu thuẫn với nhau?

Những câu hỏi này trực tiếp nhắm vào những nhược điểm cốt lõi của các mô hình trí tuệ đơn nhất, chỉnh thể. Những mô hình như vậy cố gắng giải quyết mọi vấn đề bằng một cơ chế biểu diễn và suy luận thống nhất, nhưng điều này khác xa cách chúng ta quan sát trí tuệ tự nhiên (đặc biệt là trí tuệ con người) vận hành. Minsky cho rằng việc cưỡng ép nhồi nhét các hoạt động tinh thần đa dạng vào một khuôn khổ logic cứng nhắc chính là nguyên nhân gốc rễ khiến nghiên cứu trí tuệ nhân tạo thời kỳ đầu bị đình trệ.

Dựa trên sự suy ngẫm này, Minsky đã đề xuất một quan niệm mang tính lật đổ: ông không còn xem tâm trí như một cấu trúc phân tầng kiểu kim tự tháp mà xem nó như một "xã hội" được làm phẳng, đầy rẫy sự tương tác và cộng tác.

### 2.3.2 Trí tuệ như là sự Cộng tác

Trong khuôn khổ lý thuyết của Minsky, định nghĩa về một agent khác với những Agent hiện đại mà chúng ta đã thảo luận ở Chương 1. Ở đây, một agent chỉ một tiến trình tinh thần cực kỳ đơn giản, chuyên biệt, mà bản thân nó "không có trí tuệ". Ví dụ, một agent `LINE-FINDER` chịu trách nhiệm nhận diện các đường thẳng, hay một agent `GRASP` chịu trách nhiệm gắp giữ.

Những agent đơn giản này được tổ chức lại để tạo thành các **Cơ quan (Agencies)** mạnh mẽ hơn. Một cơ quan là một nhóm các agent làm việc cùng nhau để hoàn thành một nhiệm vụ phức tạp hơn. Ví dụ, một cơ quan `BUILD` chịu trách nhiệm xây dựng khối hộp có thể được cấu thành từ nhiều agent hoặc cơ quan cấp thấp hơn như `SEE`, `FIND`, `GET` và `PUT`. Chúng ảnh hưởng lẫn nhau thông qua các tín hiệu kích hoạt và ức chế phi tập trung, tạo thành luồng điều khiển động.

**Sự trồi hiện (Emergence)** là chìa khóa để hiểu học thuyết xã hội của tâm trí. Hành vi thông minh phức tạp, có chủ đích không phải được hoạch định trước bởi một agent cấp cao nào đó mà tự phát trồi lên từ các tương tác cục bộ giữa vô số agent đơn giản ở tầng đáy.

Hãy dùng nhiệm vụ kinh điển "xây tháp khối hộp" làm ví dụ để minh họa quá trình này, như thể hiện trong Hình 2.6. Khi một mục tiêu cấp cao (chẳng hạn "Tôi muốn xây một tòa tháp") xuất hiện, nó kích hoạt một cơ quan cấp cao gọi là `BUILD-TOWER`.

1. Cơ quan `BUILD-TOWER` không biết cách thực thi các hành động vật lý cụ thể; vai trò duy nhất của nó là kích hoạt các cơ quan cấp dưới của nó, chẳng hạn như `BUILDER`.
2. Cơ quan `BUILDER` cũng rất đơn giản; nó có thể chỉ chứa logic vòng lặp: chừng nào tòa tháp chưa hoàn thành thì kích hoạt cơ quan `ADD-BLOCK`.
3. Cơ quan `ADD-BLOCK` chịu trách nhiệm điều phối các nhiệm vụ con cụ thể hơn; nó lần lượt kích hoạt ba cơ quan con: `FIND-BLOCK`, `GET-BLOCK` và `PUT-ON-TOP`.
4. Mỗi cơ quan con lại được cấu thành từ các agent cấp thấp hơn nữa. Ví dụ, cơ quan `GET-BLOCK` kích hoạt agent `SEE-SHAPE` trong hệ thống thị giác và các agent `REACH` và `GRASP` trong hệ thống vận động.

Trong quá trình này, không một agent hay cơ quan đơn lẻ nào có kế hoạch tổng thể cho toàn bộ nhiệm vụ. `GRASP` chỉ chịu trách nhiệm gắp giữ; nó không biết một tòa tháp là gì; `BUILDER` chỉ chịu trách nhiệm lặp; nó không biết cách điều khiển cánh tay. Tuy nhiên, khi xã hội được cấu thành từ vô số agent "không có trí tuệ" này tương tác với nhau thông qua các quy tắc kích hoạt và ức chế đơn giản, một hành vi có vẻ vô cùng thông minh — xây một tháp khối hộp — sẽ tự nhiên trồi hiện.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-4.png" alt="Figure description" width="90%"/>
  <p>Hình 2.6 Sơ đồ minh họa cơ chế trồi hiện của hành vi xây tháp khối hộp trong "xã hội của tâm trí"</p>
</div>

### 2.3.3 Cảm hứng lý thuyết cho các Hệ đa Agent

Ảnh hưởng sâu rộng nhất của học thuyết xã hội của tâm trí là nó đã cung cấp một nền tảng khái niệm quan trọng cho **Trí tuệ nhân tạo phân tán (Distributed Artificial Intelligence, DAI)** và sau này là **Hệ đa Agent (Multi-Agent Systems, MAS)**. Nó thúc đẩy các nhà nghiên cứu suy nghĩ:

**Nếu trí tuệ bên trong một tâm trí trồi hiện thông qua sự cộng tác của vô số agent đơn giản, thì liệu một "trí tuệ tập thể" mạnh mẽ hơn có thể trồi hiện thông qua sự cộng tác giữa nhiều thực thể tính toán độc lập, tách biệt về mặt vật lý (máy tính, robot) hay không?**

Việc đặt ra câu hỏi này đã trực tiếp chuyển trọng tâm nghiên cứu từ "làm thế nào để xây dựng một agent đơn lẻ vạn năng" sang "làm thế nào để thiết kế một nhóm agent cộng tác hiệu quả". Cụ thể, xã hội của tâm trí đã trực tiếp truyền cảm hứng cho nghiên cứu MAS ở những khía cạnh sau:

- **Điều khiển phi tập trung (Decentralized Control)**: Cốt lõi của học thuyết là không có bộ điều khiển trung tâm. Ý tưởng này được lĩnh vực MAS hoàn toàn kế thừa, và làm thế nào để thiết kế các cơ chế điều phối và chiến lược phân bổ nhiệm vụ mà không có nút trung tâm đã trở thành một trong những chủ đề nghiên cứu cốt lõi của MAS.
- **Tính toán trồi hiện (Emergent Computation)**: Các giải pháp cho những vấn đề phức tạp có thể tự phát trồi lên từ các quy tắc tương tác cục bộ đơn giản. Điều này đã truyền cảm hứng cho vô số thuật toán dựa trên sự trồi hiện trong MAS, chẳng hạn như thuật toán đàn kiến và tối ưu hóa bầy đàn, để giải quyết các bài toán tối ưu hóa và tìm kiếm phức tạp.
- **Tính xã hội của Agent (Agent Sociality)**: Học thuyết của Minsky nhấn mạnh các tương tác giữa các agent (kích hoạt, ức chế). Lĩnh vực MAS đã mở rộng thêm điều này, nghiên cứu một cách hệ thống các ngôn ngữ giao tiếp giữa các agent (như ACL), các giao thức tương tác (như mạng hợp đồng), các chiến lược thương lượng, các mô hình niềm tin, và thậm chí cả các cấu trúc tổ chức, từ đó kiến tạo nên những xã hội tính toán thực sự.

Có thể nói, học thuyết "xã hội của tâm trí" của Minsky đã cung cấp một khuôn khổ phân tích quan trọng để các nhà nghiên cứu AI hiểu cấu trúc bên trong của "trí tuệ tập thể". Nó cung cấp cho các nhà nghiên cứu sau này một góc nhìn hoàn toàn mới để khám phá các hệ thống phức tạp được cấu thành từ những agent tính toán độc lập, tự chủ và có năng lực xã hội, chính thức mở màn cho nghiên cứu hệ đa Agent.

## 2.4 Sự tiến hóa của các Mô thức học tập và các Agent hiện đại

Học thuyết "xã hội của tâm trí" đã thảo luận ở trên đã chỉ đường cho trí tuệ tập thể và sự cộng tác phi tập trung ở tầng triết học, nhưng con đường hiện thực hóa vẫn còn chưa rõ ràng. Đồng thời, những thách thức căn bản mà chủ nghĩa ký hiệu bộc lộ khi xử lý sự phức tạp của thế giới thực cũng cho thấy rằng một trí tuệ thực sự vững chắc không thể được xây dựng chỉ dựa trên các quy tắc được lập trình sẵn.

Hai luồng suy nghĩ này cùng chỉ đến một câu hỏi: Nếu trí tuệ không thể được thiết kế hoàn toàn, thì liệu nó có thể được học hay không?

Câu hỏi này đã mở ra kỷ nguyên "học tập" của trí tuệ nhân tạo. Mục tiêu cốt lõi của nó không còn là mã hóa tri thức một cách thủ công nữa mà là xây dựng các hệ thống có thể tự động thu nhận tri thức và năng lực từ kinh nghiệm và dữ liệu. Phần này sẽ theo dấu sự tiến hóa của mô thức này: từ nền tảng học tập do chủ nghĩa liên kết đặt ra, đến học tập tương tác đạt được bởi học tăng cường, cho đến các Agent hiện đại được điều khiển bởi các mô hình ngôn ngữ lớn ngày nay.

### 2.4.1 Từ Ký hiệu đến Liên kết

Là một phản ứng trực tiếp đối với những hạn chế của chủ nghĩa ký hiệu, **Chủ nghĩa liên kết (Connectionism)** đã tái xuất vào những năm 1980. Khác với triết lý thiết kế từ trên xuống dựa vào các quy tắc logic tường minh của chủ nghĩa ký hiệu, chủ nghĩa liên kết là một phương pháp từ dưới lên (bottom-up) lấy cảm hứng từ việc mô phỏng cấu trúc mạng nơ-ron của não bộ sinh học<sup>[8]</sup>. Các ý tưởng cốt lõi của nó có thể được tổng kết như sau:

1. **Biểu diễn tri thức phân tán**: Tri thức không được lưu trữ trong một cơ sở tri thức nào đó dưới dạng các ký hiệu hay quy tắc tường minh mà được lưu trữ một cách phân tán dưới dạng các trọng số liên kết giữa vô số đơn vị xử lý đơn giản (tức các nơ-ron nhân tạo). Bản thân mẫu hình liên kết của toàn bộ mạng lưới cấu thành nên tri thức.
2. **Các đơn vị xử lý đơn giản**: Mỗi nơ-ron chỉ thực hiện những phép tính rất đơn giản, chẳng hạn như nhận các đầu vào có trọng số từ các nơ-ron khác, xử lý chúng thông qua một hàm kích hoạt, rồi xuất kết quả cho nơ-ron tiếp theo.
3. **Điều chỉnh trọng số thông qua học tập**: Trí tuệ của hệ thống không đến từ các chương trình phức tạp được những người thiết kế viết sẵn mà đến từ quá trình "học tập". Bằng cách được tiếp xúc với vô số mẫu, hệ thống tự động điều chỉnh lặp đi lặp lại các trọng số liên kết giữa các nơ-ron theo một thuật toán học nào đó (như lan truyền ngược - backpropagation), dần dần khiến đầu ra của toàn bộ mạng lưới tiến gần đến mục tiêu mong muốn.

Dưới mô thức này, các Agent không còn là những cỗ máy suy luận logic thụ động thực thi quy tắc nữa mà là những hệ thống thích ứng có khả năng tự tối ưu hóa thông qua kinh nghiệm. Như thể hiện trong Hình 2.7, điều này đại diện cho một sự chuyển dịch căn bản trong ý tưởng cốt lõi về việc xây dựng Agent. Chủ nghĩa ký hiệu cố gắng mã hóa tường minh tri thức của con người vào máy móc, trong khi chủ nghĩa liên kết cố gắng tạo ra những cỗ máy có thể học tri thức như con người.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-5.png" alt="Figure description" width="90%"/>
  <p>Hình 2.7 So sánh mô thức chủ nghĩa ký hiệu và chủ nghĩa liên kết</p>
</div>

Sự trỗi dậy của chủ nghĩa liên kết, đặc biệt là thành công của học sâu (deep learning) trong thế kỷ 21, đã trang bị cho các Agent những năng lực nhận thức và nhận dạng mẫu mạnh mẽ, cho phép chúng hiểu thế giới trực tiếp từ dữ liệu thô (như hình ảnh, âm thanh, văn bản), điều không thể tưởng tượng được trong thời đại chủ nghĩa ký hiệu. Tuy nhiên, làm thế nào để các Agent học được cách đưa ra những quyết định tuần tự tối ưu trong tương tác động với môi trường lại cần đến sự bổ sung từ một mô thức học tập khác.

### 2.4.2 Các Agent dựa trên Học tăng cường

Chủ nghĩa liên kết chủ yếu giải quyết các bài toán nhận thức (ví dụ "Trong bức ảnh này có gì?"), nhưng nhiệm vụ cốt lõi hơn của Agent là ra quyết định (ví dụ "Trong tình huống này tôi nên làm gì?"). **Học tăng cường (Reinforcement Learning, RL)** chính là mô thức học tập tập trung giải quyết các bài toán quyết định tuần tự. Nó không học trực tiếp từ các tập dữ liệu tĩnh đã được gán nhãn mà học cách tối đa hóa lợi ích dài hạn của mình thông qua tương tác trực tiếp giữa Agent và môi trường, học bằng cách "thử và sai".

Lấy AlphaGo làm ví dụ, quá trình học tự chơi (self-play) cốt lõi của nó là một hiện thân kinh điển của học tăng cường<sup>[9]</sup>. Trong quá trình này, AlphaGo (Agent) quan sát thế cờ hiện tại (trạng thái môi trường) và quyết định đặt quân cờ tiếp theo ở đâu (hành động). Sau khi một ván cờ kết thúc, dựa trên kết quả thắng-thua, nó nhận được một tín hiệu rõ ràng: thắng là phần thưởng dương, thua là phần thưởng âm. Thông qua hàng triệu ván tự chơi như vậy, AlphaGo liên tục điều chỉnh chiến lược nội tại của mình, dần dần học được trong những thế cờ nào thì việc chọn hành động nào là dễ dẫn đến chiến thắng cuối cùng nhất. Quá trình này hoàn toàn tự chủ, không dựa vào sự hướng dẫn trực tiếp từ các biên bản ván cờ của con người.

Cơ chế học tập tối ưu hóa hành vi của chính mình thông qua tương tác với môi trường và dựa trên các tín hiệu phản hồi này chính là khuôn khổ cốt lõi của học tăng cường. Dưới đây chúng ta sẽ trình bày chi tiết các thành phần cấu thành cơ bản và phương thức làm việc của nó.

Khuôn khổ học tăng cường có thể được mô tả bằng một số yếu tố cốt lõi:

- **Agent**: Người học và người ra quyết định. Trong ví dụ AlphaGo, đó là chương trình ra quyết định của nó.
- **Môi trường (Environment)**: Mọi thứ bên ngoài Agent, đối tượng mà Agent tương tác. Đối với AlphaGo, đó là luật cờ vây và đối thủ.
- **Trạng thái (State, S)**: Một mô tả cụ thể về môi trường tại một thời điểm nhất định, cơ sở cho việc ra quyết định của Agent. Ví dụ, vị trí hiện tại của tất cả các quân cờ trên bàn cờ.
- **Hành động (Action, A)**: Các thao tác mà Agent có thể thực hiện dựa trên trạng thái hiện tại. Ví dụ, đặt một quân cờ vào một vị trí hợp lệ trên bàn cờ.
- **Phần thưởng (Reward, R)**: Một tín hiệu vô hướng (scalar) được môi trường phản hồi lại cho Agent sau khi Agent thực hiện một hành động, dùng để đánh giá chất lượng của hành động đó trong một trạng thái cụ thể. Ví dụ, khi kết thúc một ván cờ, chiến thắng nhận được phần thưởng +1, thất bại nhận được phần thưởng -1.

Dựa trên các yếu tố cốt lõi trên, các Agent học tăng cường liên tục lặp lại trong một vòng lặp khép kín "cảm nhận - hành động - học tập", với phương thức làm việc được thể hiện trong Hình 2.8.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-6.png" alt="Figure description" width="90%"/>
  <p>Hình 2.8 Vòng lặp tương tác cốt lõi của học tăng cường</p>
</div>

Các bước cụ thể của vòng lặp này như sau:

1. Tại bước thời gian t, Agent quan sát trạng thái hiện tại $S_{t}$ của môi trường.
2. Dựa trên trạng thái $S_{t}$, Agent chọn một hành động $A_{t}$ theo **Chính sách (Policy, π)** nội tại của nó và thực thi hành động đó. Chính sách về bản chất là một ánh xạ từ trạng thái sang hành động, định nghĩa hành vi của Agent.
3. Sau khi nhận hành động $A_{t}$, môi trường chuyển sang một trạng thái mới $S_{t+1}$.
4. Đồng thời, môi trường phản hồi lại cho Agent một phần thưởng tức thời $R_{t+1}$.
5. Agent sử dụng phản hồi này (trạng thái mới $S_{t+1}$ và phần thưởng $R_{t+1}$) để cập nhật và tối ưu hóa chính sách nội tại của mình nhằm đưa ra những quyết định tốt hơn trong tương lai. Quá trình cập nhật này chính là học tập.

Mục tiêu học tập của Agent không phải là tối đa hóa phần thưởng tức thời tại một bước thời gian nào đó mà là tối đa hóa **Phần thưởng tích lũy (Cumulative Reward)** từ thời điểm hiện tại đến tương lai, còn được gọi là **Lợi ích thu về (Return)**. Điều này có nghĩa là Agent cần có "tầm nhìn xa"; đôi khi để thu được phần thưởng lớn hơn trong tương lai, nó cần hy sinh phần thưởng tức thời hiện tại (ví dụ, chiến lược "thí quân" trong cờ vây). Thông qua việc liên tục khám phá, thu thập phản hồi và tối ưu hóa chính sách trong vòng lặp trên, Agent cuối cùng có thể học được cách đưa ra quyết định tự chủ và hoạch định dài hạn trong những môi trường động phức tạp.

### 2.4.3 Tiền huấn luyện dựa trên Dữ liệu quy mô lớn

Học tăng cường đã trang bị cho các Agent khả năng học các chiến lược ra quyết định từ tương tác, nhưng điều này thường đòi hỏi lượng dữ liệu tương tác khổng lồ đặc thù cho từng nhiệm vụ, dẫn đến việc các Agent thiếu tri thức tiên nghiệm ở giai đoạn đầu học tập và phải xây dựng sự hiểu biết về nhiệm vụ từ con số không. Dù là thường thức mà chủ nghĩa ký hiệu cố gắng mã hóa thủ công hay tri thức nền tảng mà con người dựa vào khi ra quyết định, cả hai đều bị thiếu vắng trong các Agent RL. Làm thế nào để các Agent có được sự hiểu biết rộng về thế giới trước khi bắt đầu học các nhiệm vụ cụ thể? Giải pháp cho vấn đề này cuối cùng đã xuất hiện trong lĩnh vực **Xử lý ngôn ngữ tự nhiên (Natural Language Processing, NLP)**, với cốt lõi là **Tiền huấn luyện (Pre-training)** dựa trên dữ liệu quy mô lớn.

**Từ các Nhiệm vụ cụ thể đến các Mô hình tổng quát**

Trước khi mô thức tiền huấn luyện xuất hiện, các mô hình xử lý ngôn ngữ tự nhiên truyền thống thường được huấn luyện từ đầu một cách độc lập cho từng nhiệm vụ cụ thể đơn lẻ (như phân tích cảm xúc, dịch máy) trên các tập dữ liệu quy mô nhỏ đến vừa được gán nhãn đặc biệt. Chế độ này dẫn đến một số vấn đề: các mô hình có phạm vi tri thức hẹp, khó khái quát hóa tri thức đã học được trong một nhiệm vụ sang một nhiệm vụ khác, và mỗi nhiệm vụ mới đều đòi hỏi công sức lớn của con người để gán nhãn dữ liệu. Việc đề xuất mô thức Tiền huấn luyện và Tinh chỉnh (Pre-training and Fine-tuning) đã hoàn toàn thay đổi tình trạng này. Ý tưởng cốt lõi của nó được chia thành hai bước:

1. **Giai đoạn Tiền huấn luyện**: Trước tiên, huấn luyện một mô hình mạng nơ-ron siêu lớn trên một kho ngữ liệu tổng quát chứa lượng dữ liệu văn bản khổng lồ ở cấp độ internet thông qua **Học tự giám sát (Self-supervised Learning)**. Mục tiêu của giai đoạn này không phải là hoàn thành bất kỳ nhiệm vụ cụ thể nào mà là học các mẫu hình vốn có, các cấu trúc ngữ pháp, tri thức thực tế và logic ngữ cảnh của bản thân ngôn ngữ. Mục tiêu phổ biến nhất là "dự đoán từ tiếp theo".
2. **Giai đoạn Tinh chỉnh**: Sau khi hoàn thành tiền huấn luyện, mô hình này đã học được lượng tri thức phong phú liên quan đến tập dữ liệu. Tiếp theo, đối với các nhiệm vụ hạ nguồn (downstream) cụ thể, chỉ cần một lượng nhỏ dữ liệu đã gán nhãn cho nhiệm vụ đó để tinh chỉnh mô hình, cho phép nó thích ứng với nhiệm vụ tương ứng.

Như thể hiện trong Hình 2.9, điều này minh họa một cách trực quan toàn bộ quá trình tiền huấn luyện và tinh chỉnh: dữ liệu văn bản tổng quát tạo thành một mô hình nền tảng thông qua học tự giám sát, sau đó tinh chỉnh với dữ liệu nhiệm vụ cụ thể để cuối cùng thích ứng với nhiều nhiệm vụ hạ nguồn khác nhau.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-7.png" alt="Figure description" width="90%"/>
  <p>Hình 2.9 Sơ đồ minh họa mô thức "tiền huấn luyện - tinh chỉnh"</p>
</div>

**Sự ra đời của các Mô hình ngôn ngữ lớn và Năng lực trồi hiện**

Thông qua tiền huấn luyện trên hàng nghìn tỷ văn bản, các trọng số mạng nơ-ron của các mô hình ngôn ngữ lớn thực chất đã kiến tạo nên một mô hình ngầm ẩn được nén ở mức độ cao về tri thức của thế giới. Nó giải quyết vấn đề "nút thắt cổ chai trong thu nhận tri thức" gây đau đầu nhất của thời đại chủ nghĩa ký hiệu theo một cách hoàn toàn mới. Điều đáng ngạc nhiên hơn là, khi quy mô của mô hình (số lượng tham số, khối lượng dữ liệu, khả năng tính toán) vượt qua một ngưỡng nhất định, chúng bắt đầu thể hiện những **Năng lực trồi hiện (Emergent Abilities)** không ngờ tới mà không được huấn luyện trực tiếp, chẳng hạn như:

- **Học trong ngữ cảnh (In-context Learning)**: Không cần điều chỉnh trọng số mô hình, chỉ cần cung cấp **một vài ví dụ (Few-shot)** hoặc thậm chí **không có ví dụ nào (Zero-shot)** trong đầu vào, mô hình có thể hiểu và hoàn thành các nhiệm vụ mới.
- **Suy luận theo chuỗi tư duy (Chain-of-Thought)**: Bằng cách hướng dẫn mô hình xuất ra quá trình suy luận từng bước trước khi trả lời các câu hỏi phức tạp, độ chính xác của nó trên các nhiệm vụ suy luận logic, số học và thường thức có thể được cải thiện đáng kể.

Sự xuất hiện của những năng lực này đánh dấu rằng các LLM không còn chỉ là các mô hình ngôn ngữ nữa; chúng đã tiến hóa thành những thành phần đóng vai trò kép, vừa là kho tri thức khổng lồ vừa là động cơ suy luận tổng quát.

Đến đây, trong dòng chảy lịch sử phát triển Agent, một số mảnh ghép công nghệ then chốt đã đều xuất hiện: chủ nghĩa ký hiệu cung cấp khuôn khổ cho suy luận logic, chủ nghĩa liên kết và học tăng cường cung cấp năng lực học tập và ra quyết định, trong khi các mô hình ngôn ngữ lớn cung cấp tri thức thế giới và năng lực suy luận tổng quát chưa từng có được thu nhận thông qua tiền huấn luyện. Trong phần tiếp theo, chúng ta sẽ thấy các công nghệ này được tích hợp như thế nào trong thiết kế của các Agent hiện đại.

### 2.4.4 Các Agent dựa trên Mô hình ngôn ngữ lớn

Với sự phát triển nhanh chóng của công nghệ mô hình ngôn ngữ lớn, các Agent lấy LLM làm trung tâm đã trở thành một mô thức mới trong lĩnh vực trí tuệ nhân tạo. Chúng không chỉ có thể hiểu và sinh ngôn ngữ của con người mà quan trọng hơn, còn có thể tự chủ cảm nhận, hoạch định, quyết định và thực thi nhiệm vụ thông qua tương tác với môi trường.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-8.png" alt="Figure description" width="90%"/>
  <p>Hình 2.10 Kiến trúc thành phần cốt lõi của các Agent được điều khiển bởi LLM</p>
</div>

Như đã mô tả ở Chương 1, sự tương tác giữa các Agent và môi trường có thể được trừu tượng hóa thành một vòng lặp cốt lõi. Các Agent được điều khiển bởi LLM hoàn thành nhiệm vụ thông qua một quá trình vòng lặp khép kín được lặp lại liên tục, trong đó nhiều module cùng làm việc với nhau. Quá trình này tuân theo kiến trúc thể hiện trong Hình 2.10, với các bước cụ thể như sau:

1. **Cảm nhận (Perception)**: Quá trình bắt đầu với **Module cảm nhận (Perception Module)**. Nó nhận đầu vào thô từ **Môi trường (Environment)** thông qua các cảm biến, tạo thành các **Quan sát (Observations)**. Thông tin quan sát này (chẳng hạn như chỉ dẫn của người dùng, dữ liệu do API trả về, hoặc những thay đổi trong trạng thái môi trường) là điểm khởi đầu cho việc ra quyết định của Agent và sẽ được chuyển đến giai đoạn suy nghĩ sau khi được xử lý.
2. **Suy nghĩ (Thought)**: Đây là lõi nhận thức của Agent, tương ứng với công việc phối hợp của **Module hoạch định (Planning Module)** và **Mô hình ngôn ngữ lớn (LLM)** trong sơ đồ.
   - **Hoạch định và Phân rã**: Trước tiên, module hoạch định nhận thông tin quan sát và xây dựng các chiến lược ở cấp cao. Thông qua các cơ chế như **Phản tư (Reflection)** và **Tự phê bình (Self-criticism)**, nó phân rã các mục tiêu vĩ mô thành những bước cụ thể hơn, có thể thực thi được.
   - **Suy luận và Ra quyết định**: Tiếp theo, **LLM** với vai trò là trung tâm điều phối nhận các chỉ dẫn từ module hoạch định và tương tác với module **Bộ nhớ (Memory)** để tích hợp thông tin lịch sử. LLM thực hiện suy luận sâu và cuối cùng quyết định thao tác cụ thể cần thực thi tiếp theo, thường được biểu hiện dưới dạng một **Lệnh gọi công cụ (Tool Call)**.
3. **Hành động (Action)**: Sau khi việc ra quyết định hoàn tất, giai đoạn hành động bắt đầu, được quản lý bởi **Module thực thi (Execution Module)**. Các chỉ dẫn gọi công cụ do LLM sinh ra được gửi đến module thực thi. Module này phân tích các chỉ dẫn, chọn và gọi các công cụ phù hợp từ hộp công cụ **Sử dụng công cụ (Tool Use)** (như bộ thực thi mã, công cụ tìm kiếm, API, v.v.) để tương tác với môi trường hoặc thực thi nhiệm vụ. Sự tương tác thực tế với môi trường này chính là **Hành động (Action)** của Agent.
4. **Quan sát (Observation)** và Vòng lặp: Hành động thay đổi trạng thái của môi trường và tạo ra kết quả.
   - Sau khi công cụ được thực thi, một **Kết quả công cụ (Tool Result)** được trả về cho LLM, cấu thành phản hồi trực tiếp về hiệu quả của hành động. Đồng thời, hành động của Agent làm thay đổi môi trường, tạo ra một **trạng thái môi trường** hoàn toàn mới.
   - "Kết quả công cụ" và "trạng thái môi trường mới" này cùng nhau cấu thành một vòng **Quan sát (Observation)** mới. Quan sát mới này lại được module cảm nhận nắm bắt, đồng thời LLM **cập nhật bộ nhớ (Memory Update)** dựa trên kết quả hành động, từ đó khởi động vòng lặp "cảm nhận - suy nghĩ - hành động" tiếp theo.

Cơ chế cộng tác theo module này cùng với vòng lặp được lặp lại liên tục cấu thành nên quy trình làm việc cốt lõi của các Agent được điều khiển bởi LLM trong việc giải quyết các vấn đề phức tạp.

### 2.4.5 Tổng quan các Cột mốc then chốt trong phát triển Agent

Lịch sử phát triển của các Agent trí tuệ nhân tạo không phải là một con đường một làn thẳng tắp mà là một quá trình đan xen, cạnh tranh và hòa nhập của một số trường phái tư tưởng cốt lõi qua hơn nửa thế kỷ. Việc hiểu quá trình này giúp chúng ta thấu hiểu những cội nguồn sâu xa của sự hình thành mô thức kiến trúc Agent hiện nay.

Trong đó, ba xu hướng lớn đã thống trị các mô thức nghiên cứu ở những thời kỳ khác nhau:

1. **Chủ nghĩa ký hiệu (Symbolism)**: Với đại diện là những người tiên phong như **Herbert A. Simon** và **Marvin Minsky**, cho rằng cốt lõi của trí tuệ nằm ở việc thao tác ký hiệu và suy luận logic. Ý tưởng này đã khai sinh ra SHRDLU, hệ thống có thể hiểu các chỉ dẫn ngôn ngữ tự nhiên, các hệ chuyên gia được điều khiển bởi tri thức, và máy tính "Deep Blue" đã đạt được thành công vang dội trong môn cờ vua.
2. **Chủ nghĩa liên kết (Connectionism)**: Cảm hứng của nó đến từ việc mô phỏng mạng nơ-ron của não bộ. Mặc dù sự phát triển thời kỳ đầu bị hạn chế, dưới sự thúc đẩy của các nhà nghiên cứu như **Geoffrey Hinton**, thuật toán lan truyền ngược đã đặt nền móng cho sự phục hưng của mạng nơ-ron. Cuối cùng, cùng với sự xuất hiện của kỷ nguyên học sâu, ý tưởng này đã trở thành dòng chính thông qua các mô hình như mạng nơ-ron tích chập và Transformer.
3. **Chủ nghĩa hành vi (Behaviorism)**: Nhấn mạnh rằng các Agent học các chiến lược tối ưu thông qua tương tác với môi trường và thử sai, hiện thân hiện đại của nó là học tăng cường. Từ TD-Gammon thời kỳ đầu đến AlphaGo, hệ thống đã kết hợp với học sâu và đánh bại các kỳ thủ hàng đầu của con người, trường phái này đã trang bị cho các Agent khả năng học các hành vi ra quyết định phức tạp từ kinh nghiệm.

Bước vào thập niên 2020, các trường phái tư tưởng này đã hòa nhập sâu sắc theo những cách chưa từng có. Các mô hình ngôn ngữ lớn với đại diện là dòng GPT bản thân chúng là sản phẩm của chủ nghĩa liên kết nhưng đã trở thành "bộ não" cốt lõi để thực thi suy luận ký hiệu, gọi công cụ và ra quyết định hoạch định, hình thành một kiến trúc Agent hiện đại kết hợp các phương pháp nơ-ron và ký hiệu (neural-symbolic). Để hệ thống hóa việc điểm lại bối cảnh phát triển này, Hình 2.11 dưới đây tổ chức các lý thuyết, dự án và sự kiện then chốt trong lịch sử phát triển của các Agent trí tuệ nhân tạo từ những năm 1950 đến nay, cung cấp cho độc giả một cái nhìn tổng quan toàn cục rõ ràng như một sự củng cố kiến thức của chương này.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-9.png" alt="Figure description" width="90%"/>
  <p>Hình 2.11 Dòng thời gian tiến hóa phát triển của Agent (phiên bản chưa đầy đủ)</p>
</div>

Nhờ những đột phá trong các mô hình ngôn ngữ lớn, ngăn xếp công nghệ Agent thể hiện sự sôi động và đa dạng chưa từng có. Hình 2.12 cho thấy một bức tranh toàn cảnh điển hình của ngăn xếp công nghệ trong lĩnh vực AI Agent hiện nay, bao quát mọi khía cạnh từ các mô hình ở tầng nền tảng đến các ứng dụng ở tầng trên.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/2-figures/1757246501849-10.png" alt="Figure description" width="90%"/>
  <p>Hình 2.12 Tổng quan ngăn xếp công nghệ AI Agent</p>
</div>

Sơ đồ ngăn xếp công nghệ này được Letta phát hành vào tháng 11 năm 2024<sup>[10]</sup>. Nó phân tầng và phân loại các công cụ, nền tảng và dịch vụ liên quan đến AI Agent, cung cấp một tài liệu tham khảo có giá trị để hiểu bối cảnh thị trường hiện tại và việc lựa chọn công nghệ.

## 2.5 Tổng kết chương

Chương này đã điểm lại bối cảnh lịch sử của sự phát triển Agent, khám phá quá trình từ khi ra đời đến khi tiến hóa của các ý tưởng cốt lõi của nó, bao quát một số cuộc cách mạng mô thức then chốt trong lĩnh vực trí tuệ nhân tạo:

- **Sự khám phá và những hạn chế của Chủ nghĩa ký hiệu**: Bắt đầu từ thời đại kinh điển của trí tuệ nhân tạo, chương này đã giải thích cách các Agent thời kỳ đầu với đại diện là các hệ chuyên gia đã cố gắng mô phỏng trí tuệ thông qua "tri thức + suy luận". Bằng cách tự tay xây dựng một chatbot dựa trên quy tắc, chúng ta đã trải nghiệm sâu sắc ranh giới năng lực của mô thức này và những thách thức căn bản mà nó phải đối mặt.
- **Sự trỗi dậy của tư duy Trí tuệ phân tán**: Đã khám phá học thuyết "xã hội của tâm trí" của Marvin Minsky. Ý tưởng mang tính cách mạng này đã hé lộ rằng trí tuệ chỉnh thể phức tạp có thể trồi hiện từ các tương tác của những đơn vị cục bộ đơn giản, cung cấp cảm hứng triết học quan trọng cho nghiên cứu hệ đa Agent sau này.
- **Sự tiến hóa của các Mô thức học tập**: Đã chứng kiến những thay đổi căn bản trong cách các Agent thu nhận năng lực. Từ chủ nghĩa liên kết trang bị cho các Agent khả năng nhận thức thế giới, đến học tăng cường cho phép chúng học cách ra quyết định tối ưu trong tương tác với môi trường, cho đến các mô hình ngôn ngữ lớn (LLM) dựa trên tiền huấn luyện dữ liệu quy mô lớn cung cấp cho chúng tri thức thế giới và năng lực suy luận tổng quát chưa từng có.
- **Sự ra đời của các Agent hiện đại**: Cuối cùng, chúng ta đã phân tích các Agent được điều khiển bởi LLM. Thông qua việc phân tích các thành phần cốt lõi của chúng (mô hình, bộ nhớ, hoạch định, công cụ, v.v.) và các nguyên lý hoạt động, chúng ta đã hiểu cách các ý tưởng công nghệ khác nhau trong lịch sử đã đạt được sự tích hợp công nghệ trong kiến trúc Agent hiện đại.

Thông qua việc học chương này, chúng ta không chỉ hiểu các Agent hiện đại được giới thiệu ở Chương 1 đến từ đâu mà còn thiết lập một khuôn khổ nhận thức vĩ mô về sự tiến hóa của công nghệ Agent. Chúng ta có thể nhận ra rằng sự phát triển của Agent không phải là một sự lặp lại công nghệ đơn giản mà là một cuộc cách mạng tư tưởng về cách định nghĩa "trí tuệ", thu nhận "tri thức" và đưa ra "quyết định".

Vì cốt lõi của các Agent hiện đại là các mô hình ngôn ngữ lớn, việc hiểu sâu các nguyên lý nền tảng của chúng là vô cùng quan trọng. Chương tiếp theo sẽ tập trung vào chính bản thân các mô hình ngôn ngữ lớn, khám phá các khái niệm cơ bản của chúng, đặt một nền móng vững chắc cho các ứng dụng nâng cao sau này trong các hệ đa Agent.

## Bài tập

> **Lưu ý**: Một số bài tập dưới đây không có đáp án chuẩn, nhằm giúp người học thiết lập sự hiểu biết hệ thống về lịch sử phát triển Agent và bồi dưỡng năng lực nhận thức công nghệ theo hướng "học từ lịch sử".

1. Giả thuyết Hệ thống Ký hiệu Vật lý<sup>[1]</sup> là nền tảng lý thuyết của thời đại chủ nghĩa ký hiệu. Hãy phân tích:

   - "Luận điểm về tính đầy đủ" và "luận điểm về tính cần thiết" của giả thuyết này có ý nghĩa gì?
   - Kết hợp với nội dung của chương này, hãy giải thích những vấn đề nào mà các Agent ký hiệu gặp phải trong thực tế đã thách thức "tính đầy đủ" của giả thuyết này?
   - Các Agent được điều khiển bởi mô hình ngôn ngữ lớn có phù hợp với Giả thuyết Hệ thống Ký hiệu Vật lý không?

2. Hệ chuyên gia MYCIN<sup>[2]</sup> đã đạt được thành công đáng kể trong lĩnh vực chẩn đoán y khoa nhưng cuối cùng lại không được ứng dụng rộng rãi trong thực hành lâm sàng. Hãy suy nghĩ:

   > **Gợi ý**: Có thể phân tích từ nhiều góc độ bao gồm công nghệ, đạo đức, pháp lý, sự chấp nhận của người dùng, v.v.

   - Bên cạnh "nút thắt cổ chai trong thu nhận tri thức" và "tính giòn" đã đề cập trong chương này, còn những yếu tố nào khác có thể đã cản trở việc ứng dụng các hệ chuyên gia trong những lĩnh vực rủi ro cao như y khoa?
   - Nếu bây giờ bạn phải thiết kế một Agent chẩn đoán y khoa, bạn sẽ thiết kế hệ thống như thế nào để khắc phục những hạn chế của MYCIN?
   - Ngày nay, trong những lĩnh vực dọc nào thì các hệ chuyên gia dựa trên quy tắc vẫn là một lựa chọn tốt hơn so với học sâu? Hãy cho ví dụ.

3. Ở Mục 2.2, chúng ta đã hiện thực hóa một phiên bản đơn giản của chatbot ELIZA. Hãy mở rộng trên cơ sở này:

   > **Gợi ý**: Đây là một câu hỏi thực hành; khuyến khích thực sự viết mã

   - Thêm 3-5 quy tắc mới cho ELIZA để nó có thể xử lý các kịch bản trò chuyện đa dạng hơn (chẳng hạn như thảo luận về công việc, học tập, sở thích, v.v.)
   - Hiện thực hóa một chức năng "ghi nhớ ngữ cảnh" đơn giản: cho phép ELIZA ghi nhớ các thông tin then chốt mà người dùng đề cập trong cuộc trò chuyện (như tên, tuổi, nghề nghiệp) và tham chiếu chúng trong các cuộc trò chuyện sau đó
   - So sánh ELIZA mà bạn đã mở rộng với [ChatGPT](https://chatgpt.com/), liệt kê ít nhất 3 chiều khác biệt về bản chất
   - Tại sao phương pháp dựa trên quy tắc lại gặp phải vấn đề "bùng nổ tổ hợp" (combinatorial explosion) và khó khăn trong việc mở rộng quy mô và bảo trì khi xử lý các cuộc trò chuyện lĩnh vực mở? Bạn có thể giải thích bằng phương pháp toán học không?

4. Marvin Minsky đã đề xuất một quan điểm mang tính cách mạng trong học thuyết "xã hội của tâm trí"<sup>[7]</sup>: trí tuệ bắt nguồn từ sự cộng tác của vô số agent đơn giản, chứ không phải một hệ thống hoàn hảo đơn lẻ.

   - Trong ví dụ "xây tháp khối hộp" ở Hình 2.6, điều gì sẽ xảy ra với toàn bộ hệ thống nếu agent `GRASP` đột nhiên gặp sự cố? Kiến trúc phi tập trung này có những ưu điểm và nhược điểm gì?
   - Hãy so sánh học thuyết "xã hội của tâm trí" với một số hệ đa Agent hiện nay (như [CAMEL-Workforce](https://docs.camel-ai.org/key_modules/workforce), [MetaGPT](https://github.com/FoundationAgents/MetaGPT), [CrewAI](https://github.com/crewAIInc/crewAI)), giữa chúng có những mối liên hệ và khác biệt gì?
   - Marvin Minsky cho rằng các agent có thể là những tiến trình đơn giản "không có trí tuệ", thế nhưng các mô hình ngôn ngữ lớn và Agent hiện nay thường sở hữu năng lực suy luận mạnh mẽ. Điều này có nghĩa là học thuyết "xã hội của tâm trí" không còn áp dụng được trong kỷ nguyên mô hình ngôn ngữ lớn nữa hay không?

5. Học tăng cường và học có giám sát là hai mô thức học tập khác nhau. Hãy phân tích:

   - Dùng ví dụ AlphaGo để giải thích cơ chế "học thử và sai" của học tăng cường hoạt động như thế nào
   - Tại sao học tăng cường lại đặc biệt phù hợp với các bài toán quyết định tuần tự? Có sự khác biệt bản chất nào về yêu cầu dữ liệu giữa nó và học có giám sát?
   - Bây giờ chúng ta cần huấn luyện một Agent để chơi Super Mario. Nếu sử dụng học có giám sát và học tăng cường một cách tương ứng, mỗi phương pháp cần dữ liệu gì? Phương pháp nào phù hợp hơn cho nhiệm vụ này?
   - Trong quá trình huấn luyện các mô hình ngôn ngữ lớn, học tăng cường đóng vai trò then chốt gì?

6. Mô thức tiền huấn luyện - tinh chỉnh là một đột phá quan trọng trong lĩnh vực trí tuệ nhân tạo hiện đại. Hãy suy nghĩ sâu:

   - Tại sao tiền huấn luyện lại giải quyết được vấn đề "nút thắt cổ chai trong thu nhận tri thức" của thời đại chủ nghĩa ký hiệu? Có sự khác biệt bản chất nào trong các phương pháp biểu diễn tri thức?
   - Phần lớn tri thức của các mô hình tiền huấn luyện đến từ dữ liệu internet; điều này có thể mang lại những vấn đề gì? Làm thế nào để giảm thiểu những vấn đề này?
   - Bạn có nghĩ rằng mô thức "tiền huấn luyện - tinh chỉnh" có thể bị thay thế bởi một mô thức mới nào đó không? Hay nó sẽ tồn tại lâu dài?

7. Giả sử bạn muốn thiết kế một "trợ lý review mã thông minh" có thể tự động review các lần gửi mã (Pull Request), tóm tắt logic hiện thực của mã, kiểm tra chất lượng mã, phát hiện các bug tiềm ẩn và đề xuất các gợi ý cải tiến.

   - Nếu thiết kế hệ thống này trong thời đại chủ nghĩa ký hiệu (những năm 1980), bạn sẽ hiện thực hóa nó như thế nào? Bạn sẽ gặp phải những khó khăn gì?
   - Nếu trong kỷ nguyên học sâu nhưng chưa có các mô hình ngôn ngữ lớn (khoảng năm 2015), bạn sẽ hiện thực hóa nó như thế nào?
   - Trong kỷ nguyên hiện tại của các mô hình ngôn ngữ lớn và Agent, bạn sẽ thiết kế kiến trúc của Agent này như thế nào? Nó nên bao gồm những module nào (tham khảo Hình 2.10)?
   - So sánh các giải pháp của ba thời đại này, hãy giải thích cách sự tiến hóa của công nghệ Agent đã khiến nhiệm vụ này thay đổi từ "gần như bất khả thi" thành "khả thi"

## Tài liệu tham khảo

[1] NEWELL A, SIMON H A. Computer science as empirical inquiry: symbols and search[J]. Communications of the ACM, 1976, 19(3): 113-126.

[2] BUCHANAN B G, SHORTLIFFE E H, ed. Rule-based expert systems: the MYCIN experiments of the Stanford Heuristic Programming Project[M]. Reading, Mass.: Addison-Wesley, 1984.

[3] WINOGRAD T. Understanding natural language[M]. New York: Academic Press, 1972.

[4] LENAT D B, GUHA R V. Cyc: a midterm report[J]. AI magazine, 1990, 11(3): 32.

[5] MCCARTHY J, HAYES P J. Some philosophical problems from the standpoint of artificial intelligence[C]//MELTZER B, MICHIE D, ed. Machine intelligence 4. Edinburgh: Edinburgh University Press, 1969: 463-502.

[6] WEIZENBAUM J. ELIZA: a computer program for the study of natural language communication between man and machine[J]. Communications of the ACM, 1966, 9(1): 36-45.

[7] MINSKY M. The society of mind[M]. New York: Simon & Schuster, 1986.

[8] RUMELHART D E, MCCLELLAND J L, PDP RESEARCH GROUP. Parallel distributed processing: explorations in the microstructure of cognition[M]. Cambridge, MA: MIT Press, 1986.

[9] SILVER D, HUANG A, MADDISON C J, ed. Mastering the game of Go with deep neural networks and tree search[J]. Nature, 2016, 529(7587): 484-489.

[10] LETTA. The AI agents stack[EB/OL]. (2024-11) [2025-09-07]. https://www.letta.com/blog/ai-agents-stack.
