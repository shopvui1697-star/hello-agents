# Chương 1 Làm quen với Agent

Chào mừng bạn đến với thế giới của các Agent! Trong kỷ nguyên ngày nay, khi làn sóng trí tuệ nhân tạo đang càn quét khắp toàn cầu, **Agent** (tác tử thông minh) đã trở thành một trong những khái niệm cốt lõi thúc đẩy sự chuyển đổi công nghệ và đổi mới ứng dụng. Dù khát vọng của bạn là trở thành một nhà nghiên cứu hay kỹ sư trong lĩnh vực AI, hay bạn mong muốn hiểu sâu về những công nghệ tiên tiến nhất với tư cách một người quan sát, thì việc nắm vững bản chất của Agent sẽ là một phần không thể thiếu trong hệ thống kiến thức của bạn.

Vì vậy, trong chương này, hãy cùng quay về những điều cơ bản và cùng nhau khám phá một số câu hỏi: Agent là gì? Nó có những loại chính nào? Nó tương tác với thế giới mà chúng ta đang sống như thế nào? Thông qua những thảo luận này, chúng tôi hy vọng sẽ đặt một nền móng vững chắc cho việc học tập và khám phá của bạn trong tương lai.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-0.png" alt="Figure description" width="90%"/>
  <p>Hình 1.1 Vòng lặp tương tác cơ bản giữa Agent và môi trường</p>
</div>

## 1.1 Agent là gì?

Khi khám phá bất kỳ khái niệm phức tạp nào, tốt nhất là nên bắt đầu từ một định nghĩa ngắn gọn. Trong lĩnh vực trí tuệ nhân tạo, một Agent được định nghĩa là bất kỳ thực thể nào có thể cảm nhận **Môi trường (Environment)** của nó thông qua **Cảm biến (Sensors)**, và **tự chủ (autonomously)** thực hiện **Hành động (Actions)** thông qua **Cơ cấu chấp hành (Actuators)** để đạt được những mục tiêu cụ thể.

Định nghĩa này bao hàm bốn yếu tố cơ bản về sự tồn tại của một Agent. Môi trường là thế giới bên ngoài nơi Agent hoạt động. Đối với một chiếc xe tự hành, môi trường là giao thông đường bộ biến đổi liên tục; đối với một thuật toán giao dịch, môi trường là thị trường tài chính luôn thay đổi. Agent không tách rời khỏi môi trường — nó liên tục cảm nhận trạng thái của môi trường thông qua các cảm biến của mình. Camera, micrô, radar, hay các luồng dữ liệu được trả về bởi nhiều **Giao diện Lập trình Ứng dụng (API — Application Programming Interfaces)** đều là phần mở rộng cho năng lực cảm nhận của nó.

Sau khi thu thập thông tin, Agent cần thực hiện hành động để tác động đến môi trường, thay đổi trạng thái của nó thông qua các cơ cấu chấp hành. Cơ cấu chấp hành có thể là các thiết bị vật lý (như cánh tay robot hoặc vô-lăng) hoặc các công cụ ảo (như thực thi mã hoặc gọi một dịch vụ).

Tuy nhiên, điều thực sự mang lại "trí tuệ" cho một Agent chính là **Tính tự chủ (Autonomy)** của nó. Một Agent không đơn thuần là một chương trình thụ động phản ứng với các kích thích bên ngoài hay chấp hành nghiêm ngặt các chỉ lệnh được thiết lập sẵn; nó có thể tự đưa ra quyết định độc lập dựa trên những gì nó cảm nhận và trạng thái nội tại của mình để đạt được các mục tiêu thiết kế. Vòng lặp khép kín từ cảm nhận đến hành động này tạo nên nền tảng cho mọi hành vi của Agent, như minh họa trong Hình 1.1.

### 1.1.1 Agent dưới góc nhìn truyền thống

Trước làn sóng **Mô hình Ngôn ngữ Lớn (LLM — Large Language Models)** hiện nay, các nhà tiên phong trong lĩnh vực trí tuệ nhân tạo đã dành hàng thập kỷ để khám phá và xây dựng khái niệm "Agent". Những mô thức mà ngày nay chúng ta gọi là "Agent truyền thống" không phải là một khái niệm tĩnh, đơn nhất, mà đã trải qua một lộ trình tiến hóa rõ ràng từ đơn giản đến phức tạp, từ phản ứng thụ động đến học tập chủ động.

Điểm khởi đầu của quá trình tiến hóa này là **Agent Phản xạ Đơn giản (Simple Reflex Agent)**, loại có cấu trúc đơn giản nhất. Cốt lõi ra quyết định của chúng bao gồm các quy tắc "điều kiện - hành động" được các kỹ sư thiết kế một cách tường minh, như minh họa trong Hình 1.2. Một bộ điều nhiệt tự động kinh điển hoạt động theo cách này: nếu cảm biến nhận thấy nhiệt độ phòng cao hơn giá trị cài đặt, nó sẽ kích hoạt hệ thống làm mát.

Loại Agent này hoàn toàn dựa vào đầu vào cảm nhận hiện tại và không có khả năng ghi nhớ hay dự đoán. Nó giống như một bản năng được số hóa — đáng tin cậy và hiệu quả, nhưng vì thế mà không thể xử lý các tác vụ phức tạp đòi hỏi hiểu bối cảnh. Những hạn chế của nó làm nảy sinh một câu hỏi then chốt: Agent nên làm gì nếu trạng thái hiện tại của môi trường là không đủ để làm cơ sở duy nhất cho việc ra quyết định?

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-1.png" alt="Figure description" width="90%"/>
  <p>Hình 1.2 Sơ đồ logic ra quyết định của một Agent phản xạ đơn giản</p>
</div>

Để trả lời câu hỏi này, các nhà nghiên cứu đã đưa ra khái niệm "trạng thái" và phát triển **Agent Phản xạ Dựa trên Mô hình (Model-Based Reflex Agents)**. Loại Agent này có một **Mô hình Thế giới (World Model)** nội tại được dùng để theo dõi và hiểu các khía cạnh của môi trường mà nó không thể cảm nhận trực tiếp. Nó cố gắng trả lời: "Thế giới bây giờ đang như thế nào?" Ví dụ, một chiếc xe tự hành đang đi qua một đường hầm, dù camera của nó tạm thời không thể nhìn thấy chiếc xe phía trước, thì mô hình nội tại của nó vẫn duy trì phán đoán về sự tồn tại, tốc độ và vị trí ước tính của chiếc xe đó. Mô hình nội tại này mang lại cho Agent một dạng "trí nhớ" sơ khai, khiến cho các quyết định của nó không còn chỉ phụ thuộc vào cảm nhận tức thời mà dựa trên một sự hiểu biết mạch lạc và đầy đủ hơn về trạng thái của thế giới.

Tuy nhiên, chỉ hiểu về thế giới thôi thì chưa đủ — một Agent cần có mục tiêu rõ ràng. Điều này dẫn đến sự phát triển của **Agent Dựa trên Mục tiêu (Goal-Based Agents)**. Khác với hai loại trước, hành vi của chúng không còn là phản ứng thụ động với môi trường mà là chủ động và có chủ đích lựa chọn những hành động có thể dẫn đến một trạng thái tương lai cụ thể. Câu hỏi mà loại Agent này cần trả lời là: "Tôi nên làm gì để đạt được mục tiêu của mình?" Một ví dụ kinh điển là hệ thống định vị GPS: mục tiêu của bạn là đến văn phòng, và Agent sẽ lên kế hoạch một lộ trình tối ưu bằng cách sử dụng các thuật toán tìm kiếm (chẳng hạn A*) dựa trên dữ liệu bản đồ (mô hình thế giới). Năng lực cốt lõi của loại Agent này thể hiện ở khả năng cân nhắc và lập kế hoạch cho tương lai.

Đi xa hơn nữa, các mục tiêu trong thế giới thực thường không đơn nhất. Chúng ta không chỉ muốn đến văn phòng mà còn muốn thời gian ngắn nhất, lộ trình tiết kiệm nhiên liệu nhất, và tránh tắc đường. Khi cần cân bằng nhiều mục tiêu, **Agent Dựa trên Độ hữu dụng (Utility-Based Agents)** xuất hiện. Chúng gán một giá trị độ hữu dụng (utility) cho mọi trạng thái khả dĩ của thế giới, biểu thị mức độ thỏa mãn. Mục tiêu cốt lõi của Agent không còn đơn giản là đạt được một trạng thái cụ thể mà là tối đa hóa độ hữu dụng kỳ vọng. Nó cần trả lời một câu hỏi phức tạp hơn: "Hành vi nào sẽ mang lại cho tôi kết quả thỏa mãn nhất?" Kiến trúc này cho phép Agent học cách cân bằng các mục tiêu xung đột, khiến các quyết định của nó gần với lựa chọn lý trí của con người hơn.

Cho đến nay, những Agent mà chúng ta đã thảo luận, dù ngày càng phức tạp về mặt chức năng, thì cốt lõi logic ra quyết định của chúng vẫn dựa vào tri thức tiên nghiệm của những người thiết kế là con người, dù đó là quy tắc, mô hình, hay hàm độ hữu dụng. Vậy điều gì sẽ xảy ra nếu một Agent có thể tự học thông qua tương tác với môi trường mà không cần dựa vào những thiết lập sẵn?

Đây chính là ý tưởng cốt lõi của **Agent Học tập (Learning Agents)**, và **Học Tăng cường (RL — Reinforcement Learning)** là con đường tiêu biểu nhất để hiện thực hóa ý tưởng này. Một Agent học tập bao gồm một thành phần hiệu năng (chính là các loại Agent mà chúng ta đã thảo luận ở trên) và một thành phần học tập. Thành phần học tập liên tục điều chỉnh chiến lược ra quyết định của thành phần hiệu năng bằng cách quan sát kết quả của các hành động mà thành phần hiệu năng thực hiện trong môi trường.

Hãy tưởng tượng một AI đang học chơi cờ. Ban đầu nó có thể đi những nước cờ ngẫu nhiên, nhưng khi cuối cùng thắng một ván, hệ thống sẽ cho nó một phần thưởng tích cực. Qua vô số ván tự chơi, thành phần học tập dần dần khám phá ra những nước đi nào có nhiều khả năng dẫn đến chiến thắng cuối cùng. AlphaGo Zero là một thành tựu mang tính cột mốc của triết lý này. Trong ván cờ vây phức tạp, thông qua học tăng cường, nó đã khám phá ra nhiều chiến lược hiệu quả vượt qua cả tri thức hiện có của con người.

Từ những bộ điều nhiệt đơn giản đến những chiếc xe có mô hình nội tại, đến hệ thống định vị có thể lập kế hoạch lộ trình, đến những nhà ra quyết định biết cân nhắc lợi hại, và cuối cùng đến những cỗ máy học tập có thể tự tiến hóa qua kinh nghiệm. Lộ trình tiến hóa này thể hiện quỹ đạo phát triển mà trí tuệ nhân tạo truyền thống đã trải qua trong việc xây dựng trí tuệ máy móc. Chúng đã đặt một nền tảng vững chắc và cần thiết cho sự hiểu biết của chúng ta về những mô thức Agent tiên tiến hơn ngày nay.

### 1.1.2 Mô thức mới được thúc đẩy bởi Mô hình Ngôn ngữ Lớn

Sự xuất hiện của các mô hình ngôn ngữ lớn, tiêu biểu là **GPT (Generative Pre-trained Transformer)**, đang thay đổi đáng kể phương pháp xây dựng và ranh giới năng lực của các Agent. Các Agent LLM được thúc đẩy bởi mô hình ngôn ngữ lớn có cơ chế ra quyết định cốt lõi khác biệt về bản chất so với Agent truyền thống, từ đó mang lại cho chúng một loạt các đặc tính hoàn toàn mới.

Sự chuyển biến này có thể thấy rõ qua sự so sánh giữa hai loại trên nhiều chiều khác nhau như động cơ cốt lõi, nguồn tri thức, và phương thức tương tác, như trình bày trong Bảng 1.1. Nói ngắn gọn, năng lực của Agent truyền thống bắt nguồn từ việc lập trình tường minh và xây dựng tri thức của các kỹ sư, và mô thức hành vi của chúng mang tính tất định và có giới hạn; trong khi đó, Agent LLM, thông qua việc tiền huấn luyện (pre-training) trên lượng dữ liệu khổng lồ, đã thu nhận được các mô hình thế giới ngầm ẩn và những năng lực trỗi dậy (emergent) mạnh mẽ, cho phép chúng xử lý các tác vụ phức tạp theo một cách linh hoạt và tổng quát hơn.

<div align="center">
  <p>Bảng 1.1 So sánh cốt lõi giữa Agent truyền thống và Agent được thúc đẩy bởi LLM</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-2.png" alt="Figure description" width="90%"/>
</div>

Sự khác biệt này cho phép Agent LLM xử lý trực tiếp các chỉ lệnh bằng ngôn ngữ tự nhiên ở cấp độ cao, mơ hồ và giàu bối cảnh. Hãy lấy một "trợ lý du lịch thông minh" làm ví dụ để minh họa.

Trước khi Agent LLM xuất hiện, việc lên kế hoạch cho một chuyến đi thường có nghĩa là người dùng phải tự tay chuyển đổi qua lại giữa nhiều ứng dụng chuyên dụng (như ứng dụng thời tiết, bản đồ, các trang web đặt chỗ), với bản thân người dùng đóng vai trò tích hợp thông tin và ra quyết định. Tuy nhiên, một Agent LLM có thể tích hợp toàn bộ quá trình này. Khi nhận được một chỉ lệnh mơ hồ như "lên kế hoạch cho một chuyến đi tới Hạ Môn", cách thức làm việc của nó phản ánh những điểm sau:

- **Lập kế hoạch và Suy luận (Planning and Reasoning)**: Agent trước tiên phân rã mục tiêu cấp cao này thành một loạt các tác vụ con hợp logic, ví dụ: `[Xác nhận sở thích du lịch] -> [Truy vấn thông tin điểm đến] -> [Phác thảo lịch trình] -> [Đặt vé và chỗ ở]`. Đây là một quá trình lập kế hoạch nội tại, được điều khiển bởi mô hình.
- **Sử dụng Công cụ (Tool Use)**: Khi thực thi kế hoạch, Agent nhận diện các lỗ hổng thông tin và chủ động gọi các công cụ bên ngoài để lấp đầy chúng. Ví dụ, nó sẽ gọi một giao diện truy vấn thời tiết để lấy thời tiết theo thời gian thực, và dựa trên thông tin "dự báo có mưa", nó sẽ có xu hướng gợi ý các hoạt động trong nhà trong các bước lập kế hoạch tiếp theo.
- **Điều chỉnh Động (Dynamic Adjustment)**: Trong quá trình tương tác, Agent coi phản hồi của người dùng (chẳng hạn "khách sạn này vượt quá ngân sách") như những ràng buộc mới và điều chỉnh các hành động tiếp theo cho phù hợp, tìm kiếm lại và gợi ý những lựa chọn đáp ứng yêu cầu mới. Toàn bộ quá trình "**kiểm tra thời tiết → điều chỉnh lịch trình → đặt khách sạn**" thể hiện khả năng của nó trong việc điều chỉnh động hành vi dựa trên bối cảnh.

Tóm lại, chúng ta đang chuyển từ việc phát triển các công cụ tự động hóa chuyên biệt sang việc xây dựng những hệ thống có thể tự chủ giải quyết vấn đề. Cốt lõi không còn là viết mã mà là dẫn dắt một "bộ não" tổng quát để lập kế hoạch, hành động, và học tập.

### 1.1.3 Các loại Agent

Sau khi điểm lại quá trình tiến hóa của Agent ở trên, phần này sẽ phân loại Agent từ ba chiều bổ trợ cho nhau.

(1) **Phân loại dựa trên Kiến trúc Ra quyết định Nội tại**

Chiều phân loại đầu tiên dựa trên độ phức tạp của kiến trúc ra quyết định nội tại của Agent. Góc nhìn này được đề xuất một cách hệ thống trong cuốn "Artificial Intelligence: A Modern Approach" (Trí tuệ Nhân tạo: Một Cách tiếp cận Hiện đại)<sup>[1]</sup>. Như đã mô tả trong Mục 1.1.1, bản thân lộ trình tiến hóa của các Agent truyền thống đã cấu thành nên bậc thang phân loại kinh điển nhất, bao trùm từ các Agent **phản ứng (reactive)** đơn giản đến các Agent **dựa trên mô hình (model-based)** có đưa vào mô hình nội tại, rồi đến các Agent **dựa trên mục tiêu (goal-based)** và **dựa trên độ hữu dụng (utility-based)** có tầm nhìn xa hơn. Ngoài ra, **năng lực học tập (learning capability)** là một siêu năng lực (meta-capability) có thể được trao cho tất cả các loại trên, cho phép chúng tự cải thiện qua kinh nghiệm.

(2) **Phân loại dựa trên Thời gian và Tính phản ứng**

Ngoài độ phức tạp của kiến trúc nội tại, các Agent còn có thể được phân loại theo chiều thời gian của quá trình xử lý ra quyết định. Góc nhìn này tập trung vào việc liệu một Agent hành động ngay lập tức sau khi nhận thông tin hay hành động sau khi lập kế hoạch có cân nhắc. Điều này hé lộ một sự đánh đổi cốt lõi trong thiết kế Agent: sự cân bằng giữa **Tính phản ứng (Reactivity)**, vốn theo đuổi tốc độ, và **Sự cân nhắc (Deliberation)**, vốn theo đuổi lời giải tối ưu, như minh họa trong Hình 1.3.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-3.png" alt="Figure description" width="90%"/>
  <p>Hình 1.3 Mối quan hệ giữa thời gian ra quyết định và chất lượng của Agent</p>
</div>

- **Agent Phản ứng (Reactive Agents)**

Loại Agent này đưa ra phản hồi gần như tức thời trước các kích thích của môi trường với độ trễ ra quyết định cực thấp. Chúng thường tuân theo một ánh xạ trực tiếp từ cảm nhận đến hành động, không hoặc rất ít lập kế hoạch cho tương lai. Các Agent **phản ứng đơn giản** và **dựa trên mô hình** đã đề cập ở trên thuộc về loại này.

Lợi thế cốt lõi của chúng nằm ở **tốc độ nhanh và chi phí tính toán thấp**, điều này có ý nghĩa quyết định trong các môi trường động đòi hỏi ra quyết định nhanh. Ví dụ, hệ thống túi khí của xe phải phản ứng trong vòng vài mili giây sau va chạm — bất kỳ độ trễ nào cũng có thể dẫn đến hậu quả nghiêm trọng; tương tự, các robot giao dịch tần suất cao phải dựa vào việc ra quyết định phản ứng để nắm bắt những cơ hội thị trường thoáng qua. Tuy nhiên, cái giá của tốc độ này là sự "thiển cận". Do thiếu lập kế hoạch dài hạn, các Agent phản ứng dễ rơi vào tối ưu cục bộ và khó hoàn thành những tác vụ phức tạp đòi hỏi sự phối hợp qua nhiều bước.

- **Agent Cân nhắc (Deliberative Agents)**

Ngược lại với Agent phản ứng, các Agent cân nhắc (hay lập kế hoạch) thực hiện việc tư duy và lập kế hoạch phức tạp trước khi hành động. Chúng không phản ứng ngay lập tức với các cảm nhận mà trước tiên sử dụng mô hình thế giới nội tại của mình để khám phá một cách hệ thống nhiều khả năng tương lai, đánh giá hậu quả của các chuỗi hành động khác nhau, với hy vọng tìm ra một con đường tối ưu để đạt được mục tiêu. Các Agent **dựa trên mục tiêu** và **dựa trên độ hữu dụng** là những Agent cân nhắc tiêu biểu.

Quá trình ra quyết định của chúng có thể được ví như một kỳ thủ cờ. Họ không chỉ nhìn vào nước đi trước mắt mà còn dự liệu các phản ứng khả dĩ của đối thủ và hoạch định các nước đi tiếp theo, thậm chí trước hàng chục nước. Khả năng cân nhắc này cho phép chúng xử lý các tác vụ phức tạp đòi hỏi tầm nhìn dài hạn, chẳng hạn như soạn thảo một kế hoạch kinh doanh hay lên kế hoạch cho một chuyến đi xa. Lợi thế của chúng nằm ở tính chiến lược và khả năng nhìn xa trong các quyết định. Tuy nhiên, mặt trái của lợi thế này là chi phí thời gian và tính toán cao. Trong những môi trường thay đổi nhanh chóng, khi một Agent cân nhắc vẫn còn đang chìm sâu trong suy nghĩ, thì thời điểm tốt nhất để hành động có thể đã trôi qua từ lâu.

- **Agent Lai (Hybrid Agents)**

Các tác vụ phức tạp trong thế giới thực thường đòi hỏi cả phản ứng tức thời lẫn lập kế hoạch dài hạn. Ví dụ, trợ lý du lịch thông minh mà chúng ta đã đề cập trước đó cần điều chỉnh các gợi ý dựa trên phản hồi tức thời của người dùng (chẳng hạn "khách sạn này quá đắt") (tính phản ứng), đồng thời cũng phải có khả năng lên kế hoạch một lịch trình du lịch nhiều ngày hoàn chỉnh (sự cân nhắc). Do đó, các Agent lai đã ra đời, nhằm kết hợp ưu điểm của cả hai và đạt được sự cân bằng giữa phản ứng và lập kế hoạch.

Một kiến trúc lai kinh điển là thiết kế phân tầng: tầng dưới là một mô-đun phản ứng nhanh xử lý các tình huống khẩn cấp và các hành động cơ bản; tầng trên là một mô-đun lập kế hoạch cân nhắc chịu trách nhiệm hoạch định các mục tiêu dài hạn. Các Agent LLM hiện đại thể hiện một chế độ lai linh hoạt hơn. Chúng thường vận hành theo vòng lặp "tư duy - hành động - quan sát", khéo léo tích hợp cả hai chế độ:

- **Suy luận (Reasoning)**: Trong giai đoạn "tư duy", LLM phân tích tình huống hiện tại và lập kế hoạch cho hành động hợp lý tiếp theo. Đây là một quá trình cân nhắc.
- **Hành động và Quan sát (Acting & Observing)**: Trong các giai đoạn "hành động" và "quan sát", Agent tương tác với các công cụ bên ngoài hoặc môi trường và ngay lập tức nhận phản hồi. Đây là một quá trình phản ứng.

Thông qua cách tiếp cận này, Agent phân rã một tác vụ lớn đòi hỏi lập kế hoạch dài hạn thành một loạt các vi vòng lặp "lập kế hoạch - phản ứng". Điều này cho phép nó linh hoạt ứng phó với những thay đổi tức thời của môi trường, đồng thời cuối cùng hoàn thành các mục tiêu dài hạn phức tạp thông qua các bước mạch lạc.

**(3) Phân loại dựa trên Biểu diễn Tri thức**

Đây là một chiều phân loại cơ bản hơn, khám phá xem tri thức mà Agent sử dụng để ra quyết định tồn tại dưới hình thức nào trong "tâm trí" của chúng. Câu hỏi này nằm ở trung tâm của một cuộc tranh luận đã kéo dài hơn nửa thế kỷ trong lĩnh vực trí tuệ nhân tạo và đã định hình nên hai nền văn hóa AI hoàn toàn khác biệt.

- **AI Biểu tượng (Symbolic AI)**

Chủ nghĩa biểu tượng (symbolism), thường được gọi là trí tuệ nhân tạo truyền thống, có một niềm tin cốt lõi: trí tuệ bắt nguồn từ các phép toán logic trên các biểu tượng (symbols). Biểu tượng ở đây là những thực thể mà con người có thể đọc hiểu (chẳng hạn như từ ngữ, khái niệm), và các phép toán tuân theo những quy tắc logic nghiêm ngặt, như minh họa ở phía bên trái Hình 1.4. Điều này giống như một người thủ thư tỉ mỉ sắp xếp tri thức của thế giới thành các cơ sở quy tắc và đồ thị tri thức rõ ràng.

Ưu điểm chính của nó nằm ở tính minh bạch và khả năng diễn giải. Vì các bước suy luận là tường minh, quá trình ra quyết định của nó có thể được truy vết đầy đủ, điều này có ý nghĩa quyết định trong các lĩnh vực rủi ro cao như tài chính và y tế. Tuy nhiên, "gót chân Achilles" của nó nằm ở sự mong manh: nó dựa vào một hệ thống quy tắc hoàn chỉnh, nhưng trong thế giới thực đầy sự mơ hồ và ngoại lệ, bất kỳ tình huống mới nào không được bao phủ đều có thể dẫn đến hỏng hóc hệ thống, đây chính là cái gọi là "nút thắt cổ chai trong thu nhận tri thức" (knowledge acquisition bottleneck).

- **AI Cận biểu tượng (Sub-symbolic AI)**

Chủ nghĩa cận biểu tượng (sub-symbolism), hay chủ nghĩa kết nối (connectionism), đưa ra một bức tranh hoàn toàn khác. Ở đây, tri thức không phải là các quy tắc tường minh mà được phân bố ngầm ẩn trong một mạng lưới phức tạp gồm vô số nơ-ron, biểu thị các mẫu hình thống kê học được từ lượng dữ liệu khổng lồ. Mạng nơ-ron và học sâu (deep learning) là những đại diện của nó.

Như minh họa ở giữa Hình 1.4, nếu AI biểu tượng là một người thủ thư, thì AI cận biểu tượng giống như một đứa trẻ đang bập bẹ tập nói. Chúng không học cách nhận diện con mèo bằng cách học các quy tắc như "mèo có bốn chân, có lông, và kêu meo meo", mà sau khi xem hàng nghìn bức ảnh mèo, mạng nơ-ron trong "bộ não" của chúng có thể nhận ra mẫu hình thị giác của khái niệm "mèo". Sức mạnh của cách tiếp cận này nằm ở năng lực nhận dạng mẫu hình và tính bền vững của nó đối với dữ liệu nhiễu. Nó có thể dễ dàng xử lý dữ liệu phi cấu trúc như hình ảnh và âm thanh, vốn là những tác vụ cực kỳ khó khăn đối với AI biểu tượng.

Tuy nhiên, năng lực trực giác mạnh mẽ này cũng đi kèm với sự mờ đục. Các hệ thống cận biểu tượng thường được xem như một **Hộp đen (Black Box)**. Nó có thể nhận diện một con mèo trong bức ảnh với độ chính xác đáng kinh ngạc, nhưng nếu bạn hỏi nó "tại sao anh cho rằng đây là một con mèo?", thì nhiều khả năng nó không thể đưa ra một lời giải thích hợp logic. Ngoài ra, nó thể hiện kém trong các tác vụ suy luận logic thuần túy và đôi khi tạo ra những ảo giác (hallucinations) nghe có vẻ hợp lý nhưng lại sai về mặt sự thật.

- **AI Nơ-ron-Biểu tượng (Neuro-Symbolic AI)**

Trong một thời gian dài, hai trường phái chủ nghĩa biểu tượng và chủ nghĩa cận biểu tượng phát triển như hai đường thẳng song song. Để khắc phục những hạn chế của hai mô thức trên, một ý tưởng "hòa giải vĩ đại" bắt đầu xuất hiện, đó chính là AI nơ-ron-biểu tượng, còn được gọi là mô hình lai nơ-ron-biểu tượng. Mục tiêu của nó là hợp nhất ưu điểm của cả hai mô thức, tạo ra một Agent lai vừa có thể học từ dữ liệu như mạng nơ-ron, vừa có thể thực hiện suy luận logic như các hệ thống biểu tượng. Nó cố gắng bắc cầu qua khoảng cách giữa cảm nhận và nhận thức, giữa trực giác và lý trí. Lý thuyết hệ thống kép mà nhà kinh tế học đoạt giải Nobel Daniel Kahneman đề xuất trong cuốn sách "Thinking, Fast and Slow" (Tư duy Nhanh và Chậm) của ông cung cấp một phép ẩn dụ tuyệt vời để hiểu về chủ nghĩa nơ-ron-biểu tượng<sup>[2]</sup>, như minh họa trong Hình 1.4:

- **Hệ thống 1 (System 1)** là một chế độ tư duy nhanh, trực giác, song song, tương tự như năng lực nhận dạng mẫu hình mạnh mẽ của AI cận biểu tượng.
- **Hệ thống 2 (System 2)** là tư duy cân nhắc chậm rãi, có phương pháp, dựa trên logic, giống hệt như quá trình suy luận của AI biểu tượng.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-4.png" alt="Figure description" width="90%"/>
  <p>Hình 1.4 Các mô thức biểu diễn tri thức của chủ nghĩa biểu tượng, chủ nghĩa cận biểu tượng, và mô hình lai nơ-ron-biểu tượng</p>
</div>

Trí tuệ của con người bắt nguồn từ sự phối hợp làm việc của hai hệ thống này. Tương tự, một AI thực sự vững chắc cũng cần kết hợp sức mạnh của cả hai. Các Agent được thúc đẩy bởi mô hình ngôn ngữ lớn là một ví dụ thực tiễn xuất sắc của chủ nghĩa nơ-ron-biểu tượng. Cốt lõi của nó là một mạng nơ-ron khổng lồ, mang lại cho nó năng lực nhận dạng mẫu hình và sinh ngôn ngữ. Tuy nhiên, khi hoạt động, nó tạo ra một loạt các bước trung gian có cấu trúc, chẳng hạn như các suy nghĩ, kế hoạch, hay lời gọi API, tất cả đều là những biểu tượng tường minh, có thể thao tác. Thông qua cách tiếp cận này, nó đạt được một sự hòa quyện sơ khởi giữa cảm nhận và nhận thức, giữa trực giác và lý trí.

## 1.2 Cấu thành và Nguyên lý Vận hành của Agent

### 1.2.1 Định nghĩa Môi trường Tác vụ

Để hiểu một Agent vận hành như thế nào, trước tiên chúng ta phải hiểu **môi trường tác vụ (task environment)** mà nó hoạt động trong đó. Trong lĩnh vực trí tuệ nhân tạo, **mô hình PEAS** thường được sử dụng để mô tả chính xác một môi trường tác vụ, phân tích **Thước đo Hiệu năng (Performance measure), Môi trường (Environment), Cơ cấu chấp hành (Actuators), và Cảm biến (Sensors)** của nó. Lấy trợ lý du lịch thông minh đã đề cập ở trên làm ví dụ, Bảng 1.2 dưới đây cho thấy cách sử dụng mô hình PEAS để đặc tả môi trường tác vụ của nó.

<div align="center">
  <p>Bảng 1.2 Mô tả PEAS của trợ lý du lịch thông minh</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-6.png" alt="Figure description" width="90%"/>
</div>

Trong thực tế, môi trường số mà các Agent LLM vận hành trong đó thể hiện một số đặc tính phức tạp có ảnh hưởng trực tiếp đến việc thiết kế Agent.

Thứ nhất, môi trường thường **chỉ quan sát được một phần (partially observable)**. Ví dụ, khi một trợ lý du lịch truy vấn các chuyến bay, nó không thể lấy được toàn bộ thông tin ghế ngồi theo thời gian thực từ tất cả các hãng hàng không cùng một lúc. Nó chỉ có thể thấy dữ liệu một phần được trả về bởi API đặt vé máy bay mà nó gọi, điều này đòi hỏi Agent phải có khả năng ghi nhớ (nhớ các tuyến đã truy vấn) và khám phá (thử các ngày truy vấn khác nhau).

Thứ hai, kết quả của các hành động không phải lúc nào cũng mang tính tất định. Dựa trên khả năng dự đoán được của kết quả, môi trường có thể được chia thành **tất định (deterministic)** và **ngẫu nhiên (stochastic)**. Môi trường tác vụ của một trợ lý du lịch là một môi trường ngẫu nhiên điển hình. Khi nó tìm kiếm giá vé, hai lần gọi liền kề nhau có thể trả về giá vé và số ghế còn lại khác nhau, đòi hỏi Agent phải có khả năng xử lý sự bất định, giám sát các thay đổi, và ra quyết định kịp thời.

Ngoài ra, có thể có các tác nhân khác trong môi trường, tạo thành một môi trường **đa Agent (multi-agent)**. Đối với một trợ lý du lịch, hành vi đặt chỗ của những người dùng khác, các script tự động khác, và thậm chí cả hệ thống định giá động của các hãng hàng không đều là những "Agent" khác trong môi trường. Hành động của chúng (ví dụ, đặt chiếc vé giảm giá cuối cùng) trực tiếp thay đổi trạng thái của môi trường mà trợ lý du lịch đang vận hành, đặt ra những yêu cầu cao hơn về khả năng phản ứng nhanh và lựa chọn chiến lược của Agent.

Cuối cùng, gần như tất cả các tác vụ đều diễn ra trong các môi trường **tuần tự (sequential)** và **động (dynamic)**. "Tuần tự" có nghĩa là các hành động hiện tại ảnh hưởng đến tương lai; trong khi "động" có nghĩa là bản thân môi trường có thể thay đổi trong khi Agent đang ra quyết định. Điều này đòi hỏi vòng lặp "cảm nhận - tư duy - hành động - quan sát" của Agent phải có khả năng thích ứng nhanh chóng và linh hoạt với một thế giới liên tục thay đổi.

### 1.2.2 Cơ chế Vận hành của Agent

Sau khi định nghĩa môi trường tác vụ mà một Agent vận hành trong đó, hãy cùng khám phá cơ chế vận hành cốt lõi của nó. Một Agent không hoàn thành tác vụ trong một lần mà tương tác với môi trường thông qua một vòng lặp liên tục. Cơ chế cốt lõi này được gọi là **Vòng lặp Agent (Agent Loop)**. Như minh họa trong Hình 1.5, vòng lặp này mô tả quá trình tương tác động giữa Agent và môi trường, tạo nên nền tảng cho hành vi tự chủ của nó.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-5.png" alt="Figure description" width="90%"/>
  <p>Hình 1.5 Vòng lặp cơ bản của tương tác Agent-môi trường</p>
</div>

Vòng lặp này chủ yếu bao gồm các giai đoạn liên kết với nhau sau đây:

1. **Cảm nhận (Perception)**: Đây là điểm khởi đầu của vòng lặp. Agent tiếp nhận thông tin đầu vào từ môi trường thông qua các cảm biến của nó (ví dụ, các cổng lắng nghe API, các giao diện nhập liệu của người dùng). Thông tin này, tức là **Quan sát (Observation)**, có thể là chỉ lệnh ban đầu của người dùng hoặc phản hồi về những thay đổi trạng thái môi trường do hành động trước đó gây ra.
2. **Tư duy (Thought)**: Sau khi tiếp nhận thông tin quan sát, Agent bước vào giai đoạn ra quyết định cốt lõi của nó. Đối với Agent LLM, đây thường là một quá trình suy luận nội tại được điều khiển bởi mô hình ngôn ngữ lớn. Như thể hiện trong hình, giai đoạn "tư duy" có thể được chia nhỏ hơn thành hai khâu then chốt:
   - **Lập kế hoạch (Planning)**: Dựa trên các quan sát hiện tại và trí nhớ nội tại của nó, Agent cập nhật sự hiểu biết của mình về tác vụ và môi trường, đồng thời xây dựng hoặc điều chỉnh một kế hoạch hành động. Việc này có thể bao gồm phân rã các mục tiêu phức tạp thành một loạt các tác vụ con cụ thể hơn.
   - **Lựa chọn Công cụ (Tool Selection)**: Dựa trên kế hoạch hiện tại, Agent lựa chọn công cụ phù hợp nhất từ thư viện công cụ khả dụng của mình để thực thi bước tiếp theo và xác định các tham số cụ thể cần thiết để gọi công cụ đó.
3. **Hành động (Action)**: Sau khi hoàn tất việc ra quyết định, Agent thực thi các hành động cụ thể thông qua các cơ cấu chấp hành của nó. Việc này thường biểu hiện dưới dạng gọi một công cụ đã được lựa chọn (chẳng hạn như trình thông dịch mã hoặc API công cụ tìm kiếm), qua đó tác động đến môi trường với ý định thay đổi trạng thái của nó.

Hành động không phải là điểm kết thúc của vòng lặp. Hành động của Agent gây ra một **thay đổi trạng thái** trong **môi trường**, sau đó tạo ra một **quan sát** mới như là phản hồi kết quả. Quan sát mới này được hệ thống cảm nhận của Agent nắm bắt trong vòng lặp kế tiếp, tạo thành một vòng lặp khép kín "cảm nhận - tư duy - hành động - quan sát" liên tục. Chính thông qua việc liên tục lặp lại vòng lặp này mà Agent dần dần đẩy tác vụ tiến triển, tiến hóa từ trạng thái ban đầu hướng tới trạng thái mục tiêu.

### 1.2.3 Cảm nhận và Hành động của Agent

Trong thực tiễn kỹ thuật, để cho phép các LLM điều khiển hiệu quả vòng lặp này, chúng ta cần một **Giao thức Tương tác (Interaction Protocol)** rõ ràng nhằm điều tiết việc trao đổi thông tin giữa nó và môi trường.

Trong nhiều framework Agent hiện đại, giao thức này được thể hiện qua việc định nghĩa có cấu trúc cho mỗi đầu ra của Agent. Đầu ra của Agent không còn là một phản hồi ngôn ngữ tự nhiên đơn thuần mà là một đoạn văn bản tuân theo một định dạng cụ thể, thể hiện tường minh quá trình suy luận nội tại và quyết định cuối cùng của nó.

Cấu trúc này thường chứa hai phần cốt lõi:

- **Tư duy (Thought)**: Đây là một "ảnh chụp nhanh" về quá trình ra quyết định nội tại của Agent. Nó diễn đạt bằng ngôn ngữ tự nhiên cách Agent phân tích tình huống hiện tại, xem xét lại các kết quả quan sát từ bước trước, tiến hành tự phản tỉnh và phân rã vấn đề, và cuối cùng hoạch định hành động cụ thể tiếp theo.
- **Hành động (Action)**: Đây là thao tác cụ thể mà Agent quyết định áp đặt lên môi trường dựa trên quá trình tư duy của nó, thường được biểu diễn dưới dạng một lời gọi hàm.

Ví dụ, một Agent đang lên kế hoạch cho một chuyến đi có thể tạo ra đầu ra được định dạng như sau:

```Bash
Thought: The user wants to know the weather in Beijing. I need to call the weather query tool.
Action: get_weather("Beijing")
```

Trường `Action` ở đây cấu thành một chỉ lệnh gửi đến thế giới bên ngoài. Một **Trình phân tích cú pháp (Parser)** bên ngoài sẽ nắm bắt chỉ lệnh này và gọi hàm `get_weather` tương ứng.

Sau khi hành động được thực thi, môi trường trả về một kết quả. Ví dụ, hàm `get_weather` có thể trả về một đối tượng JSON chứa dữ liệu thời tiết chi tiết. Tuy nhiên, dữ liệu thô có thể đọc được bởi máy (chẳng hạn như JSON) thường chứa những thông tin dư thừa mà LLM không cần tập trung vào, và định dạng đó không phù hợp với thói quen xử lý ngôn ngữ tự nhiên của nó.

Do đó, một trách nhiệm quan trọng của hệ thống cảm nhận là đóng vai trò của một cảm biến: xử lý và đóng gói đầu ra thô này thành văn bản ngôn ngữ tự nhiên ngắn gọn, rõ ràng, tức là quan sát.

```Bash
Observation: Beijing's current weather is sunny, temperature 25 degrees Celsius, light breeze.
```

Đoạn văn bản `Observation` này được phản hồi trở lại cho Agent như là thông tin đầu vào chính cho vòng lặp kế tiếp, để nó tiến hành một vòng `Thought` và `Action` mới.

Tóm lại, thông qua vòng lặp chặt chẽ được cấu thành từ Thought (Tư duy), Action (Hành động), và Observation (Quan sát) này, các Agent LLM có thể kết hợp hiệu quả năng lực suy luận ngôn ngữ nội tại của chúng với thông tin thực và năng lực thao tác công cụ từ môi trường bên ngoài.

## 1.3 Trải nghiệm Thực hành: Xây dựng Agent Đầu tiên của Bạn trong 5 Phút

Trong các phần trước, chúng ta đã tìm hiểu về môi trường tác vụ của Agent, cơ chế vận hành cốt lõi, và mô thức tương tác `Thought-Action-Observation`. Mặc dù kiến thức lý thuyết là quan trọng, nhưng cách học tốt nhất là thông qua thực hành. Trong phần này, chúng ta sẽ hướng dẫn bạn xây dựng một trợ lý du lịch thông minh hoạt động được từ đầu chỉ với vài dòng mã Python đơn giản. Quá trình này sẽ tuân theo vòng lặp lý thuyết mà chúng ta vừa học, cho phép bạn trực tiếp trải nghiệm cách một Agent "tư duy" và tương tác với các "công cụ" bên ngoài. Hãy bắt đầu nào!

Trong trường hợp này, mục tiêu của chúng ta là xây dựng một trợ lý du lịch thông minh có thể xử lý các tác vụ theo từng bước. Tác vụ của người dùng cần giải quyết được định nghĩa là: "Xin chào, làm ơn giúp tôi kiểm tra thời tiết hôm nay ở Bắc Kinh, rồi gợi ý một điểm du lịch phù hợp dựa trên thời tiết." Để hoàn thành tác vụ này, Agent phải thể hiện năng lực lập kế hoạch logic rõ ràng. Nó cần trước tiên gọi công cụ truy vấn thời tiết và dùng kết quả quan sát thu được làm cơ sở cho bước tiếp theo. Trong vòng lặp kế tiếp, nó gọi công cụ gợi ý điểm du lịch để đi đến đề xuất cuối cùng.

### 1.3.1 Chuẩn bị

Để truy cập các web API từ một chương trình Python, chúng ta cần một thư viện HTTP. `requests` là lựa chọn phổ biến và dễ dùng nhất trong cộng đồng Python. `tavily-python` là một client API tìm kiếm AI mạnh mẽ dùng để lấy kết quả tìm kiếm web theo thời gian thực, có thể lấy được bằng cách đăng ký trên [trang web chính thức](https://www.tavily.com/). `openai` là SDK Python chính thức do OpenAI cung cấp để gọi các dịch vụ mô hình ngôn ngữ lớn như GPT. Vui lòng cài đặt chúng trước bằng lệnh sau:

```bash
pip install requests tavily-python openai
```

(1) Mẫu Chỉ lệnh

Chìa khóa để điều khiển một LLM thực thụ nằm ở **Kỹ thuật Prompt (Prompt Engineering)**. Chúng ta cần thiết kế một "mẫu chỉ lệnh" cho LLM biết nó nên đóng vai trò gì, nó có những công cụ nào, và cách định dạng tư duy cũng như hành động của nó. Đây là "cẩm nang hướng dẫn" cho Agent của chúng ta, sẽ được truyền cho LLM dưới dạng `system_prompt`.

```
AGENT_SYSTEM_PROMPT = """
You are an intelligent travel assistant. Your task is to analyze user requests and use available tools to solve problems step by step.

# Available Tools:
- `get_weather(city: str)`: Query real-time weather for a specified city.
- `get_attraction(city: str, weather: str)`: Search for recommended tourist attractions based on city and weather.

# Output Format Requirements:
Each response must strictly follow this format, containing one Thought-Action pair:

Thought: [Your thinking process and next step plan]
Action: [The specific action you want to execute]

Action format must be one of the following:
1. Call a tool: function_name(arg_name="arg_value")
2. Finish task: Finish[final answer]

# Important Notes:
- Output only one Thought-Action pair each time
- Action must be on the same line, do not break lines
- When you have collected enough information to answer the user's question, you must use Action: Finish[final answer] format to end

Let's begin!
"""
```

(2) Công cụ 1: Truy vấn Thời tiết Thực

Chúng ta sẽ sử dụng dịch vụ truy vấn thời tiết miễn phí `wttr.in`, dịch vụ này có thể trả về dữ liệu thời tiết cho một thành phố được chỉ định ở định dạng JSON. Dưới đây là mã để hiện thực công cụ này:

```python
import requests

def get_weather(city: str) -> str:
    """
    Query real weather information by calling the wttr.in API.
    """
    # API endpoint, we request data in JSON format
    url = f"https://wttr.in/{city}?format=j1"

    try:
        # Make network request
        response = requests.get(url)
        # Check if response status code is 200 (success)
        response.raise_for_status()
        # Parse returned JSON data
        data = response.json()

        # Extract current weather conditions
        current_condition = data['current_condition'][0]
        weather_desc = current_condition['weatherDesc'][0]['value']
        temp_c = current_condition['temp_C']

        # Format as natural language return
        return f"{city} current weather: {weather_desc}, temperature {temp_c} degrees Celsius"

    except requests.exceptions.RequestException as e:
        # Handle network errors
        return f"Error: Network problem encountered when querying weather - {e}"
    except (KeyError, IndexError) as e:
        # Handle data parsing errors
        return f"Error: Failed to parse weather data, city name may be invalid - {e}"
```

(3) Công cụ 2: Tìm kiếm và Gợi ý Điểm Du lịch

Chúng ta sẽ định nghĩa một công cụ mới `get_attraction` để tìm kiếm trên internet các điểm du lịch phù hợp dựa trên thành phố và điều kiện thời tiết:

```python
import os
from tavily import TavilyClient

def get_attraction(city: str, weather: str) -> str:
    """
    Based on city and weather, use Tavily Search API to search and return optimized attraction recommendations.
    """
    # 1. Read API key from environment variable
    api_key = os.environ.get("TAVILY_API_KEY")
    if not api_key:
        return "Error: TAVILY_API_KEY environment variable not configured."

    # 2. Initialize Tavily client
    tavily = TavilyClient(api_key=api_key)

    # 3. Construct a precise query
    query = f"'{city}' most worthwhile tourist attractions and reasons in '{weather}' weather"

    try:
        # 4. Call API, include_answer=True will return a comprehensive answer
        response = tavily.search(query=query, search_depth="basic", include_answer=True)

        # 5. Tavily's returned results are already very clean and can be used directly
        # response['answer'] is a summary answer based on all search results
        if response.get("answer"):
            return response["answer"]

        # If there's no comprehensive answer, format raw results
        formatted_results = []
        for result in response.get("results", []):
            formatted_results.append(f"- {result['title']}: {result['content']}")

        if not formatted_results:
             return "Sorry, no relevant tourist attraction recommendations found."

        return "Based on search, found the following information for you:\n" + "\n".join(formatted_results)

    except Exception as e:
        return f"Error: Problem occurred when executing Tavily search - {e}"
```

Cuối cùng, chúng ta đặt tất cả các hàm công cụ vào một dictionary để vòng lặp chính có thể gọi:

```python
# Put all tool functions into a dictionary for easy subsequent calling
available_tools = {
    "get_weather": get_weather,
    "get_attraction": get_attraction,
}
```

### 1.3.2 Kết nối với Mô hình Ngôn ngữ Lớn

Hiện nay, nhiều nhà cung cấp dịch vụ LLM (bao gồm OpenAI, Azure, và nhiều framework dịch vụ mô hình mã nguồn mở như Ollama, vLLM, v.v.) tuân theo các đặc tả giao diện tương tự như OpenAI API. Sự chuẩn hóa này mang lại rất nhiều thuận tiện cho các nhà phát triển. Năng lực ra quyết định tự chủ của Agent đến từ LLM. Chúng ta sẽ hiện thực một client vạn năng `OpenAICompatibleClient` có thể kết nối tới bất kỳ dịch vụ LLM nào tương thích với đặc tả giao diện OpenAI.

```python
from openai import OpenAI

class OpenAICompatibleClient:
    """
    A client for calling any LLM service compatible with the OpenAI interface.
    """
    def __init__(self, model: str, api_key: str, base_url: str):
        self.model = model
        self.client = OpenAI(api_key=api_key, base_url=base_url)

    def generate(self, prompt: str, system_prompt: str) -> str:
        """Call LLM API to generate response."""
        print("Calling large language model...")
        try:
            messages = [
                {'role': 'system', 'content': system_prompt},
                {'role': 'user', 'content': prompt}
            ]
            response = self.client.chat.completions.create(
                model=self.model,
                messages=messages,
                stream=False
            )
            answer = response.choices[0].message.content
            print("Large language model responded successfully.")
            return answer
        except Exception as e:
            print(f"Error occurred when calling LLM API: {e}")
            return "Error: Error occurred when calling language model service."
```

Để khởi tạo lớp này, bạn cần cung cấp ba thông tin: `API_KEY`, `BASE_URL`, và `MODEL_ID`. Các giá trị cụ thể phụ thuộc vào nhà cung cấp dịch vụ mà bạn sử dụng (chẳng hạn như OpenAI chính thức, Azure, hoặc các mô hình cục bộ như Ollama). Nếu bạn chưa có quyền truy cập vào những thông tin này, bạn có thể tham khảo [Cấu hình Môi trường](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra07-环境配置.md).

### 1.3.3 Thực thi Vòng lặp Hành động

Vòng lặp chính dưới đây sẽ tích hợp tất cả các thành phần và điều khiển LLM ra quyết định thông qua các prompt được định dạng.

```python
import re

# --- 1. Configure LLM client ---
# Please replace this with the corresponding credentials and address for the service you use
API_KEY = "YOUR_API_KEY"
BASE_URL = "YOUR_BASE_URL"
MODEL_ID = "YOUR_MODEL_ID"
TAVILY_API_KEY="YOUR_Tavily_KEY"
os.environ['TAVILY_API_KEY'] = "YOUR_TAVILY_API_KEY"

llm = OpenAICompatibleClient(
    model=MODEL_ID,
    api_key=API_KEY,
    base_url=BASE_URL
)

# --- 2. Initialize ---
user_prompt = "Hello, please help me check today's weather in Beijing, and then recommend a suitable tourist attraction based on the weather."
prompt_history = [f"User request: {user_prompt}"]

print(f"User input: {user_prompt}\n" + "="*40)

# --- 3. Run main loop ---
for i in range(5): # Set maximum number of loops
    print(f"--- Loop {i+1} ---\n")

    # 3.1. Build Prompt
    full_prompt = "\n".join(prompt_history)

    # 3.2. Call LLM for thinking
    llm_output = llm.generate(full_prompt, system_prompt=AGENT_SYSTEM_PROMPT)
    # Truncate extra Thought-Action pairs that the model may generate
    match = re.search(r'(Thought:.*?Action:.*?)(?=\n\s*(?:Thought:|Action:|Observation:)|\Z)', 
                    llm_output, re.DOTALL)
    if match:
        truncated = match.group(1).strip()
        if truncated != llm_output.strip():
            llm_output = truncated
            print("Truncated extra Thought-Action pairs")
    print(f"Model output:\n{llm_output}\n")
    prompt_history.append(llm_output)

    # 3.3. Parse and execute action
    action_match = re.search(r"Action: (.*)", llm_output, re.DOTALL)
    if not action_match:
        observation = "Error: No action found. Please explicitly use Action: finish(...) or other actions."
        observation_str = f"Observation: {observation}"
        print(f"{observation_str}\n" + "="*40)
        prompt_history.append(observation_str)
        continue
    action_str = action_match.group(1).strip()

    if action_str.startswith("Finish"):
        final_answer = re.match(r"Finish\[(.*)\]", action_str).group(1)
        print(f"Task completed, final answer: {final_answer}")
        break

    tool_name = re.search(r"(\w+)\(", action_str).group(1)
    args_str = re.search(r"\((.*)\)", action_str).group(1)
    kwargs = dict(re.findall(r'(\w+)="([^"]*)"', args_str))

    if tool_name in available_tools:
        observation = available_tools[tool_name](**kwargs)
    else:
        observation = f"Error: Undefined tool '{tool_name}'"

    # 3.4. Record observation results
    observation_str = f"Observation: {observation}"
    print(f"{observation_str}\n" + "="*40)
    prompt_history.append(observation_str)
```

Thông qua các bước trên, chúng ta đã xây dựng được một Agent hoàn chỉnh được điều khiển bởi một LLM thực thụ. Cốt lõi của nó nằm ở sự kết hợp giữa "công cụ" và "kỹ thuật prompt", đây chính là tinh túy thiết kế của các framework Agent chủ đạo hiện nay (chẳng hạn như LangChain, LlamaIndex, v.v.).

### 1.3.4 Phân tích Trường hợp Chạy thực tế

Đầu ra dưới đây minh họa đầy đủ một quá trình thực thi Agent thành công. Thông qua việc phân tích vòng lặp ba vòng này, chúng ta có thể thấy rõ các năng lực cốt lõi của Agent trong việc giải quyết vấn đề.

```bash
User input: Hello, please help me check today's weather in Beijing, and then recommend a suitable tourist attraction based on the weather.
========================================
--- Loop 1 ---

Calling large language model...
Large language model responded successfully.
Model output:
Thought: First need to get Beijing's weather today, then recommend tourist attractions based on the weather.
Action: get_weather(city="Beijing")

Observation: Beijing current weather: Sunny, temperature 26 degrees Celsius
========================================
--- Loop 2 ---

Calling large language model...
Large language model responded successfully.
Model output:
Thought: Now I know Beijing's weather today is sunny with moderate temperature, I can recommend a suitable tourist attraction based on this information.
Action: get_attraction(city="Beijing", weather="Sunny")

Observation: The most worthwhile tourist attractions in Beijing on sunny days are the Summer Palace for its beautiful lake views and ancient architecture. Another recommendation is the Great Wall for its spectacular scenery and historical significance.
========================================
--- Loop 3 ---

Calling large language model...
Large language model responded successfully.
Model output:
Thought: I have obtained two attraction suggestions suitable for sunny days, now I can provide a satisfactory response to the user based on this information.
Action: Finish[Today's weather in Beijing is sunny with a temperature of 26 degrees Celsius, very suitable for outdoor activities. I recommend you visit the Summer Palace to enjoy the beautiful lake views and ancient architecture, or go to the Great Wall to experience its spectacular scenery and profound historical significance. Hope you have a pleasant trip!]

Task completed, final answer: Today's weather in Beijing is sunny with a temperature of 26 degrees Celsius, very suitable for outdoor activities. I recommend you visit the Summer Palace to enjoy the beautiful lake views and ancient architecture, or go to the Great Wall to experience its spectacular scenery and profound historical significance. Hope you have a pleasant trip!
```

Trường hợp trợ lý du lịch đơn giản này tập trung minh họa bốn năng lực cơ bản của một Agent dựa trên mô thức `Thought-Action-Observation`: phân rã tác vụ, gọi công cụ, hiểu bối cảnh, và tổng hợp kết quả. Chính thông qua sự lặp lại liên tục của vòng lặp này mà Agent có thể biến một ý định mơ hồ của người dùng thành một loạt các bước cụ thể, có thể thực thi được, và cuối cùng đạt được mục tiêu.

## 1.4 Các Mô hình Cộng tác của Ứng dụng Agent

Trong phần trước, chúng ta đã hiểu sâu về vòng lặp vận hành nội tại của một Agent bằng cách tự tay xây dựng một Agent. Tuy nhiên, trong các kịch bản ứng dụng rộng hơn, vai trò của chúng ta đang ngày càng chuyển thành người dùng và người cộng tác. Dựa trên vai trò của Agent trong các tác vụ và mức độ tự chủ, các mô hình cộng tác của nó chủ yếu được chia thành hai loại: một là như một công cụ hiệu quả được tích hợp sâu vào quy trình làm việc của chúng ta; loại còn lại là như một người cộng tác tự chủ làm việc cùng các Agent khác để hoàn thành các mục tiêu phức tạp.

### 1.4.1 Agent như Công cụ cho Nhà phát triển

Trong mô hình này, các Agent được tích hợp sâu vào quy trình làm việc của nhà phát triển như những công cụ hỗ trợ mạnh mẽ. Chúng tăng cường chứ không thay thế vai trò của nhà phát triển, tự động hóa những tác vụ tẻ nhạt, lặp đi lặp lại để nhà phát triển có thể tập trung hơn vào công việc sáng tạo cốt lõi. Cách tiếp cận cộng tác người-máy này cải thiện rất nhiều hiệu suất và chất lượng của việc phát triển phần mềm.

Hiện nay, thị trường đã chứng kiến sự xuất hiện của nhiều công cụ hỗ trợ lập trình AI xuất sắc. Mặc dù tất cả đều cải thiện hiệu suất phát triển, chúng khác nhau về con đường hiện thực và trọng tâm chức năng:

- **GitHub Copilot**: Là một trong những sản phẩm có ảnh hưởng nhất trong lĩnh vực này, Copilot được phát triển chung bởi GitHub và OpenAI. Nó được tích hợp sâu vào các trình soạn thảo chủ đạo như Visual Studio Code và nổi tiếng với năng lực tự động hoàn thành mã mạnh mẽ. Khi nhà phát triển viết mã, Copilot có thể đưa ra gợi ý theo thời gian thực cho toàn bộ dòng hoặc thậm chí cả khối hàm. Trong những năm gần đây, nó cũng đã mở rộng năng lực lập trình hội thoại thông qua Copilot Chat, cho phép nhà phát triển giải quyết các vấn đề lập trình thông qua trò chuyện ngay trong trình soạn thảo.
- **Claude Code**: Claude Code là một trợ lý lập trình AI được phát triển bởi Anthropic, được thiết kế để giúp nhà phát triển hoàn thành hiệu quả các tác vụ lập trình trong terminal thông qua các chỉ lệnh bằng ngôn ngữ tự nhiên. Nó có thể hiểu được cấu trúc codebase hoàn chỉnh, thực hiện các thao tác như chỉnh sửa mã, kiểm thử, và gỡ lỗi, đồng thời hỗ trợ phát triển toàn trình từ mô tả chức năng đến hiện thực mã. Claude Code cũng cung cấp một chế độ headless (không giao diện) phù hợp cho CI, các pre-commit hook, các build script, và các kịch bản tự động hóa khác, mang lại cho nhà phát triển một trải nghiệm lập trình dòng lệnh mạnh mẽ.
- **Trae**: Là một công cụ lập trình AI mới nổi, Trae tập trung cung cấp cho nhà phát triển các dịch vụ sinh mã và tối ưu hóa mã thông minh. Nó phân tích các mẫu hình mã thông qua công nghệ học sâu và có thể cung cấp cho nhà phát triển những gợi ý mã chính xác cùng các giải pháp tái cấu trúc (refactoring) tự động. Đặc điểm nổi bật của Trae là thiết kế nhẹ và khả năng phản hồi nhanh, đặc biệt phù hợp cho các kịch bản đòi hỏi lặp thường xuyên và tạo nguyên mẫu (prototyping) nhanh chóng.
- **Cursor**: Khác với các công cụ trên vốn chủ yếu tồn tại dưới dạng plugin hoặc tính năng tích hợp, Cursor đã chọn một con đường tích hợp hơn — bản thân nó là một trình soạn thảo mã AI-native (thuần AI). Thay vì thêm chức năng AI vào các trình soạn thảo có sẵn, nó đã biến việc tương tác với AI thành một tính năng cốt lõi ngay từ giai đoạn thiết kế. Ngoài các năng lực sinh mã và trò chuyện hàng đầu, nó nhấn mạnh việc để AI hiểu được bối cảnh của toàn bộ codebase, qua đó đạt được khả năng hỏi đáp, tái cấu trúc, và gỡ lỗi sâu hơn.

Tất nhiên, còn nhiều công cụ xuất sắc khác không được liệt kê ở đây, nhưng tất cả đều chỉ về một xu hướng rõ ràng: AI đang tích hợp sâu vào toàn bộ vòng đời phát triển phần mềm, định hình lại một cách sâu sắc ranh giới hiệu suất và các mô thức phát triển của kỹ thuật phần mềm thông qua việc xây dựng các quy trình cộng tác người-máy hiệu quả.

### 1.4.2 Agent như Người cộng tác Tự chủ

Khác với việc đóng vai công cụ hỗ trợ con người, mô hình tương tác thứ hai nâng mức độ tự động hóa của Agent lên một tầm hoàn toàn mới: người cộng tác tự chủ. Trong mô hình này, chúng ta không còn hướng dẫn AI thực hiện từng hành động một cách tuần tự nữa mà giao phó cho nó một mục tiêu cấp cao. Agent, giống như một thành viên thực thụ của đội dự án, tự chủ lập kế hoạch, suy luận, thực thi, và phản tỉnh cho đến khi cuối cùng đưa ra kết quả. Sự chuyển đổi từ trợ lý sang người cộng tác này đã đưa các Agent LLM đi sâu hơn vào tầm nhìn của công chúng. Nó đánh dấu sự tiến hóa trong mối quan hệ của chúng ta với AI từ "ra lệnh - chấp hành" sang "giao phó mục tiêu". Các Agent không còn là những công cụ thụ động mà là những chủ thể chủ động theo đuổi mục tiêu.

Hiện nay, các cách tiếp cận để đạt được sự cộng tác tự chủ này đang nở rộ, với nhiều framework và sản phẩm xuất sắc xuất hiện, từ BabyAGI và AutoGPT thời kỳ đầu đến những framework trưởng thành hơn hiện nay như CrewAI, AutoGen, MetaGPT, và LangGraph, cùng nhau thúc đẩy sự phát triển nhanh chóng trong lĩnh vực này. Mặc dù các hiện thực cụ thể khác nhau rất nhiều, các mô thức kiến trúc của chúng có thể được tóm lược một cách khái quát thành vài hướng chủ đạo:

1. **Vòng lặp Tự chủ Đơn-Agent (Single-Agent Autonomous Loop)**: Đây là một mô thức tiêu biểu thời kỳ đầu, được đại diện bởi các mô hình như **AgentGPT**. Cốt lõi của nó là một Agent tổng quát liên tục tự đưa ra prompt cho chính mình và lặp qua vòng lặp khép kín "tư duy - lập kế hoạch - thực thi - phản tỉnh" để hoàn thành một mục tiêu cấp cao mang tính mở.
2. **Cộng tác Đa-Agent (Multi-Agent Collaboration)**: Đây hiện là hướng khám phá chủ đạo nhất, nhằm giải quyết các vấn đề phức tạp bằng cách mô phỏng các mô hình cộng tác của đội ngũ con người. Nó có thể được chia nhỏ hơn thành các chế độ khác nhau: **Hội thoại Nhập vai (Role-Playing Dialogue)**: Giống như framework **CAMEL**, vốn gán các vai trò rõ ràng và các giao thức giao tiếp cho hai Agent (ví dụ, "lập trình viên" và "quản lý sản phẩm"), cho phép chúng cùng nhau hoàn thành tác vụ trong một cuộc hội thoại có cấu trúc. **Quy trình Có tổ chức (Organized Workflow)**: Giống như **MetaGPT** và **CrewAI**, vốn mô phỏng một "đội ảo" với sự phân công lao động rõ ràng (chẳng hạn như một công ty phần mềm hoặc một nhóm tư vấn). Mỗi Agent có các trách nhiệm và quy trình làm việc (SOP) được thiết lập sẵn, cộng tác theo cách phân cấp hoặc tuần tự để tạo ra những đầu ra phức tạp chất lượng cao (chẳng hạn như codebase hoàn chỉnh hoặc báo cáo nghiên cứu). **AutoGen** và **AgentScope** cung cấp các chế độ hội thoại linh hoạt hơn, cho phép nhà phát triển tùy chỉnh các mạng lưới tương tác phức tạp giữa các Agent.
3. **Kiến trúc Luồng Điều khiển Nâng cao (Advanced Control Flow Architecture)**: Các framework như **LangGraph** tập trung nhiều hơn vào việc cung cấp cho các Agent những nền tảng kỹ thuật cấp thấp mạnh mẽ hơn. Chúng mô hình hóa quá trình thực thi của Agent như một đồ thị trạng thái (state graph), cho phép hiện thực các quy trình phức tạp như vòng lặp, phân nhánh, quay lui (backtracking), và can thiệp của con người một cách linh hoạt và đáng tin cậy hơn.

Những mô thức kiến trúc khác nhau này cùng nhau thúc đẩy các Agent tự chủ đi từ những khái niệm lý thuyết hướng tới các ứng dụng thực tiễn rộng lớn hơn, cho phép chúng xử lý những tác vụ thực tế ngày càng phức tạp. Trong các chương tiếp theo, chúng ta cũng sẽ trải nghiệm những khác biệt và ưu điểm giữa các loại framework khác nhau.

### 1.4.3 Sự khác biệt giữa Workflow và Agent

Sau khi hiểu hai mô hình của Agent là "công cụ" và "người cộng tác", cần thiết phải thảo luận về sự khác biệt giữa Workflow (Quy trình làm việc) và Agent. Mặc dù cả hai đều nhằm đạt được sự tự động hóa tác vụ, nhưng logic nền tảng, đặc tính cốt lõi, và các kịch bản áp dụng của chúng khác nhau về bản chất.

Nói một cách đơn giản, **Workflow khiến AI thực thi các chỉ lệnh theo từng bước, trong khi Agent trao cho AI sự tự do để tự chủ đạt được mục tiêu.**

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-18.png" alt="Figure description" width="90%"/>
  <p>Hình 1.6 Sự khác biệt giữa Workflow và Agent</p>
</div>

Như minh họa trong Hình 1.6, workflow là một mô thức tự động hóa truyền thống mà cốt lõi là **việc điều phối (orchestration) có cấu trúc, được định nghĩa trước một loạt các tác vụ hoặc bước**. Về bản chất, nó là một lưu đồ (flowchart) chính xác, tĩnh, quy định thao tác nào cần thực thi trong điều kiện nào và theo thứ tự nào. Một trường hợp điển hình: quy trình phê duyệt hoàn ứng chi phí của một công ty. Nhân viên nộp phiếu hoàn ứng (kích hoạt) -> Nếu số tiền nhỏ hơn 500 tệ, được quản lý bộ phận phê duyệt trực tiếp -> Nếu số tiền lớn hơn 500 tệ, trước tiên được quản lý bộ phận phê duyệt, sau đó chuyển tiếp cho CFO phê duyệt -> Sau khi được phê duyệt, thông báo cho phòng tài chính thực hiện thanh toán. Mọi bước và mọi điều kiện phán đoán của toàn bộ quy trình đều được thiết lập trước một cách chính xác.

Khác với workflow, các Agent dựa trên mô hình ngôn ngữ lớn là những **hệ thống tự chủ, hướng mục tiêu**. Chúng không chỉ thực thi các chỉ lệnh được thiết lập sẵn mà còn có thể hiểu môi trường ở một mức độ nhất định, suy luận, xây dựng kế hoạch, và thực hiện hành động một cách động để đạt được mục tiêu cuối cùng. Các LLM đóng vai trò "bộ não" trong quá trình này. Một ví dụ điển hình là trợ lý du lịch thông minh mà chúng ta đã viết trong Mục 1.3. Khi chúng ta đưa cho nó một chỉ lệnh mới, ví dụ: **"Xin chào, làm ơn giúp tôi kiểm tra thời tiết hôm nay ở Bắc Kinh, rồi gợi ý một điểm du lịch phù hợp dựa trên thời tiết."** Quá trình xử lý của nó thể hiện đầy đủ tính tự chủ của nó:

1. **Lập kế hoạch và Gọi công cụ:** Agent trước tiên phân rã tác vụ thành hai bước: ① Truy vấn thời tiết; ② Gợi ý điểm du lịch dựa trên thời tiết. Sau đó, nó tự chủ lựa chọn và gọi "API truy vấn thời tiết", truyền "Bắc Kinh" làm tham số.
2. **Suy luận và Ra quyết định:** Giả sử API trả về "trời nắng, gió nhẹ." Bộ não LLM của Agent sẽ suy luận dựa trên thông tin này: "Ngày nắng phù hợp cho các hoạt động ngoài trời." Sau đó, dựa trên phán đoán này, nó sẽ lọc ra các điểm du lịch ngoài trời ở Bắc Kinh từ cơ sở tri thức của nó hoặc thông qua các công cụ tìm kiếm, chẳng hạn như Tử Cấm Thành, Di Hòa Viên, Công viên Thiên Đàn, v.v.
3. **Sinh Kết quả:** Cuối cùng, Agent sẽ tổng hợp thông tin và đưa ra một câu trả lời hoàn chỉnh, mang tính nhân văn: "Thời tiết hôm nay ở Bắc Kinh trời nắng và gió nhẹ, rất phù hợp cho các hoạt động ngoài trời. Tôi gợi ý bạn ghé thăm Di Hòa Viên, nơi bạn có thể chèo thuyền trên hồ Côn Minh và tận hưởng phong cảnh vườn ngự uyển tuyệt đẹp."

Trong quá trình này, không có các quy tắc mã hóa cứng như `if weather=sunny then recommend Summer Palace`. Nếu thời tiết là "mưa", Agent sẽ tự chủ suy luận và gợi ý các địa điểm trong nhà như Bảo tàng Quốc gia hoặc Bảo tàng Thủ đô. **Khả năng suy luận và ra quyết định một cách động dựa trên thông tin thời gian thực này chính là giá trị cốt lõi của Agent.**

## 1.5 Tóm tắt Chương

Trong chương này, chúng ta đã bắt đầu một hành trình nhập môn để khám phá về Agent. Hành trình của chúng ta khởi đầu từ những câu hỏi cơ bản nhất:

- **Agent được thúc đẩy bởi mô hình ngôn ngữ lớn là gì?** Trước tiên chúng ta đã làm rõ định nghĩa của chúng và hiểu rằng các Agent hiện đại là những thực thể có năng lực. Chúng không còn chỉ là những script thực thi các chương trình được thiết lập sẵn mà là những nhà ra quyết định có khả năng suy luận tự chủ và sử dụng công cụ.
- **Agent hoạt động như thế nào?** Chúng ta đã đi sâu vào cơ chế vận hành của tương tác Agent-môi trường. Chúng ta đã học được rằng vòng lặp khép kín liên tục này là nền tảng để Agent xử lý thông tin, ra quyết định, tác động đến môi trường, và điều chỉnh hành vi dựa trên phản hồi.
- **Làm thế nào để xây dựng một Agent?** Đây là phần thực hành cốt lõi của chương này. Lấy "trợ lý du lịch thông minh" làm ví dụ, chúng ta đã xây dựng một Agent hoàn chỉnh được điều khiển bởi một LLM thực thụ.
- **Các mô thức ứng dụng chủ đạo của Agent là gì?** Cuối cùng, chúng ta đã hướng tầm nhìn của mình về những lĩnh vực ứng dụng rộng lớn hơn. Chúng ta đã khám phá hai mô hình tương tác Agent chủ đạo: một là "công cụ cho nhà phát triển" được đại diện bởi GitHub Copilot và Cursor, giúp tăng cường quy trình làm việc của con người; loại còn lại là "người cộng tác tự chủ" được đại diện bởi các framework như CrewAI, MetaGPT, và AgentScope, có thể độc lập hoàn thành các mục tiêu cấp cao. Chúng ta cũng đã giải thích sự khác biệt giữa Workflow và Agent.

Thông qua việc học chương này, chúng ta đã thiết lập được một khung nhận thức nền tảng về Agent. Vậy, nó đã tiến hóa từng bước ra sao từ ý tưởng ban đầu cho đến hiện tại? Trong chương tiếp theo, chúng ta sẽ khám phá lịch sử phát triển của Agent — một hành trình ngược dòng về cội nguồn sắp bắt đầu!

## Bài tập

> **Lưu ý**: Một số bài tập dưới đây không có đáp án chuẩn. Trọng tâm là bồi dưỡng tư duy phản biện sâu sắc và năng lực thực hành của người học đối với các hệ thống Agent.

1. Hãy phân tích xem **chủ thể** trong bốn `trường hợp (cases)` sau đây có đủ điều kiện là một Agent hay không. Nếu có, nó thuộc loại Agent nào (có thể phân tích từ nhiều chiều phân loại), và giải thích lập luận của bạn:

   `Trường hợp A`: **Một siêu máy tính tuân theo kiến trúc von Neumann**, với năng lực tính toán đỉnh lên đến 2 EFlops mỗi giây

   `Trường hợp B`: **Hệ thống lái tự động của Tesla** đang chạy trên đường cao tốc thì đột nhiên phát hiện một chướng ngại vật phía trước và cần đưa ra quyết định phanh hoặc chuyển làn trong vòng vài mili giây

   `Trường hợp C`: **AlphaGo** đang đấu với một kỳ thủ con người và cần đánh giá cục diện hiện tại và hoạch định chiến lược tối ưu cho hàng chục nước đi phía trước

   `Trường hợp D`: **ChatGPT đóng vai một tổng đài chăm sóc khách hàng thông minh** đang xử lý một khiếu nại của người dùng và cần truy vấn thông tin đơn hàng, phân tích nguyên nhân của vấn đề, đưa ra giải pháp, và xoa dịu cảm xúc của người dùng

2. Giả sử bạn cần thiết kế một môi trường tác vụ cho một "huấn luyện viên thể hình thông minh". Agent này có thể:
   - Giám sát dữ liệu sinh lý của người dùng như nhịp tim và cường độ vận động thông qua các thiết bị đeo được
   - Điều chỉnh động kế hoạch tập luyện dựa trên mục tiêu thể hình của người dùng (giảm mỡ / tăng cơ / cải thiện sức bền)
   - Cung cấp hướng dẫn bằng giọng nói theo thời gian thực và chỉnh sửa động tác trong quá trình người dùng tập luyện
   - Đánh giá hiệu quả tập luyện và đưa ra khuyến nghị về chế độ ăn uống

   Hãy sử dụng mô hình PEAS để mô tả đầy đủ môi trường tác vụ của Agent này và phân tích môi trường này có những đặc tính gì (chẳng hạn như quan sát được một phần, ngẫu nhiên, động, v.v.).

3. Một công ty thương mại điện tử đang cân nhắc hai cách tiếp cận để xử lý các yêu cầu hoàn tiền hậu mãi:

   Cách tiếp cận A (`Workflow`): Thiết kế một quy trình cố định, ví dụ:

   A.1 Đối với các sản phẩm thông thường trong vòng 7 ngày, số tiền `< 100 tệ` được tự động phê duyệt; `100-500 tệ` được nhân viên chăm sóc khách hàng xét duyệt; `> 500 tệ` cần cấp trên phê duyệt; các sản phẩm đặc biệt (chẳng hạn như hàng đặt làm theo yêu cầu) luôn bị từ chối

   A.2 Đối với các sản phẩm quá 7 ngày, bất kể số tiền, chúng chỉ có thể được nhân viên chăm sóc khách hàng xét duyệt hoặc cấp trên phê duyệt;

   Cách tiếp cận B (`Agent`): Xây dựng một hệ thống Agent hiểu các chính sách hoàn tiền, phân tích hành vi lịch sử của người dùng, đánh giá tình trạng sản phẩm, và tự chủ quyết định có phê duyệt hoàn tiền hay không

   Hãy phân tích:
   - Ưu điểm và nhược điểm của hai cách tiếp cận này là gì?
   - Trong những trường hợp nào thì `Workflow` phù hợp hơn? Khi nào thì `Agent` có lợi thế? Nếu bạn là người đứng đầu công ty thương mại điện tử này, bạn sẽ ưa chuộng cách tiếp cận nào hơn?
   - Có tồn tại một Cách tiếp cận C nào có thể kết hợp cả hai cách tiếp cận để đạt được sự bổ trợ điểm mạnh cho nhau hay không?

4. Dựa trên trợ lý du lịch thông minh trong Mục 1.3, hãy cân nhắc cách bổ sung các tính năng sau đây (bạn chỉ cần mô tả ý tưởng thiết kế hoặc thử hiện thực bằng mã ở mức sâu hơn):

   > **Gợi ý**: Hãy suy nghĩ về cách sửa đổi vòng lặp `Thought-Action-Observation` để hiện thực các tính năng này.

   - Bổ sung tính năng "trí nhớ" cho phép Agent ghi nhớ sở thích của người dùng (chẳng hạn như thích các điểm tham quan lịch sử và văn hóa, khoảng ngân sách, v.v.)
   - Khi vé của điểm du lịch được gợi ý đã bán hết, Agent có thể tự động gợi ý các lựa chọn thay thế
   - Nếu người dùng liên tiếp từ chối 3 gợi ý, Agent có thể phản tỉnh và điều chỉnh chiến lược gợi ý của mình

5. Lý thuyết "Hệ thống 1" (trực giác nhanh) và "Hệ thống 2" (suy luận chậm) của Kahneman<sup>[2]</sup> cung cấp một phép ẩn dụ tốt cho AI nơ-ron-biểu tượng. Trước tiên hãy hình dung một kịch bản ứng dụng Agent cụ thể, sau đó giải thích trong kịch bản đó:

   > **Gợi ý**: Trợ lý chẩn đoán y tế, robot tư vấn pháp lý, hệ thống kiểm soát rủi ro tài chính, v.v., đều là những kịch bản ứng dụng phổ biến

   - Những tác vụ nào nên do "Hệ thống 1" xử lý?
   - Những tác vụ nào nên do "Hệ thống 2" xử lý?
   - Hai hệ thống này phối hợp làm việc như thế nào để đạt được mục tiêu cuối cùng?

6. Mặc dù các hệ thống Agent được thúc đẩy bởi mô hình ngôn ngữ lớn thể hiện những năng lực mạnh mẽ, chúng vẫn còn nhiều hạn chế. Hãy phân tích các câu hỏi sau:
   - Tại sao các Agent hoặc hệ thống Agent đôi khi lại tạo ra "ảo giác" (sinh ra thông tin nghe có vẻ hợp lý nhưng thực tế lại sai)?
   - Trong trường hợp ở Mục 1.3, chúng ta đã đặt số vòng lặp tối đa là 5. Nếu không có giới hạn này, Agent có thể gặp phải những vấn đề gì?
   - Làm thế nào để đánh giá mức độ "thông minh" của một Agent? Chỉ sử dụng thước đo độ chính xác thôi đã đủ chưa?

## Tài liệu tham khảo

[1] RUSSELL S, NORVIG P. Artificial Intelligence: A Modern Approach[M]. 4th ed. London: Pearson, 2020.

[2] KAHNEMAN D. Thinking, Fast and Slow[M]. New York: Farrar, Straus and Giroux, 2011.

---

## 💬 Thảo luận & Giao lưu

Có câu hỏi trong khi học chương này? Muốn chia sẻ những hiểu biết với các người học khác?

**📝 Truy cập GitHub Discussions:**
- [💬 Thảo luận & Hỏi đáp về Bài tập](https://github.com/datawhalechina/Hello-Agents/discussions)
- Tại đây bạn có thể:
  - ✅ Đặt câu hỏi về các bài tập
  - ✅ Chia sẻ lời giải và ý tưởng của bạn
  - ✅ Trao đổi kinh nghiệm với các người học khác
  - ✅ Nhận trợ giúp và phản hồi từ cộng đồng

**💡 Mẹo:** Ở cuối mỗi trang cũng có một khu vực bình luận để thảo luận trực tiếp!

---
