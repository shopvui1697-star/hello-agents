# Chương 5 Xây dựng Agent với Nền tảng Low-Code

Trong chương trước, thông qua việc viết mã Python, chúng ta đã tự tay triển khai từ đầu nhiều quy trình làm việc (workflow) agent kinh điển, bao gồm ReAct, Plan-and-Solve và Reflection. Quá trình này đã đặt một nền tảng kỹ thuật vững chắc cho chúng ta và giúp chúng ta hiểu sâu về các cơ chế nội tại của agent. Tuy nhiên, đối với một lĩnh vực đang phát triển nhanh chóng, việc phát triển thuần bằng mã không phải lúc nào cũng là lựa chọn hiệu quả nhất, đặc biệt trong những tình huống cần nhanh chóng kiểm chứng ý tưởng hoặc khi những nhà phát triển không chuyên muốn tham gia vào việc xây dựng.

## 5.1 Sự trỗi dậy của việc xây dựng dựa trên nền tảng

Khi công nghệ trưởng thành, chúng ta thấy ngày càng nhiều năng lực được "nền tảng hóa". Giống như việc phát triển website đã tiến hóa từ việc viết tay HTML/CSS/JS sang việc sử dụng các nền tảng xây dựng website như WordPress và Wix, việc xây dựng agent cũng đã đón nhận một làn sóng nền tảng hóa. Chương này sẽ tập trung vào cách sử dụng các nền tảng low-code trực quan, mô-đun hóa để nhanh chóng và trực quan xây dựng, gỡ lỗi và triển khai các ứng dụng agent, chuyển trọng tâm của chúng ta từ "chi tiết triển khai" sang "logic nghiệp vụ".

### 5.1.1 Vì sao cần đến nền tảng Low-Code

"Tự phát minh lại bánh xe" là điều rất quan trọng đối với việc học sâu, nhưng trong công việc thực tế theo đuổi hiệu quả kỹ thuật và sự đổi mới, chúng ta thường cần đứng trên vai những người khổng lồ. Mặc dù trong Chương 4 chúng ta đã đóng gói các lớp có thể tái sử dụng như `ReActAgent` và `PlanAndSolveAgent`, nhưng khi logic nghiệp vụ trở nên phức tạp, chi phí bảo trì và chu kỳ phát triển của mã thuần túy sẽ tăng lên nhanh chóng. Sự xuất hiện của các nền tảng low-code chính là để giải quyết những điểm nhức nhối này.

Giá trị cốt lõi của chúng chủ yếu thể hiện ở các khía cạnh sau:

1. **Hạ thấp rào cản kỹ thuật**: Các nền tảng low-code đóng gói những chi tiết kỹ thuật phức tạp (như gọi API, quản lý trạng thái, kiểm soát đồng thời) thành những "node" hoặc "module" dễ hiểu. Người dùng không cần thành thạo lập trình; họ chỉ cần kéo thả và kết nối các node này để xây dựng những workflow mạnh mẽ. Điều này cho phép những nhân sự phi kỹ thuật như quản lý sản phẩm, nhà thiết kế và chuyên gia nghiệp vụ tham gia vào việc thiết kế và tạo ra agent, mở rộng đáng kể ranh giới của sự đổi mới.
2. **Nâng cao hiệu quả phát triển**: Đối với các nhà phát triển chuyên nghiệp, nền tảng cũng có thể mang lại sự cải thiện hiệu quả to lớn. Trong giai đoạn đầu của một dự án, khi một ý tưởng cần được kiểm chứng nhanh hoặc một nguyên mẫu cần được xây dựng, việc sử dụng nền tảng low-code có thể hoàn thành công việc mà bình thường mất nhiều ngày lập trình chỉ trong vài giờ hoặc thậm chí vài phút. Các nhà phát triển có thể đầu tư nhiều năng lượng hơn vào việc tổ chức logic nghiệp vụ và tối ưu hóa prompt engineering thay vì triển khai kỹ thuật cấp thấp.
3. **Cung cấp khả năng trực quan hóa và quan sát tốt hơn**: So với việc in log ra terminal, các nền tảng trực quan tự nhiên cung cấp khả năng trực quan hóa đầu-cuối cho quỹ đạo chạy của agent. Bạn có thể thấy rõ dữ liệu chảy giữa các node như thế nào, khâu nào tốn nhiều thời gian nhất, và lệnh gọi công cụ nào thất bại. Trải nghiệm gỡ lỗi trực quan này là điều mà việc phát triển bằng mã thuần túy không thể sánh được.
4. **Chuẩn hóa và tích lũy best practice**: Các nền tảng low-code xuất sắc thường tích hợp sẵn nhiều best practice của ngành. Ví dụ, chúng cung cấp các template ReAct dựng sẵn, các engine truy xuất knowledge base đã được tối ưu, các đặc tả tích hợp công cụ chuẩn hóa, v.v. Điều này không những giúp các nhà phát triển tránh "giẫm phải mìn" mà còn giúp việc hợp tác nhóm mượt mà hơn, vì mọi người đều phát triển dựa trên cùng một bộ tiêu chuẩn và thành phần.

Nói tóm lại, các nền tảng low-code không nhằm thay thế mã mà cung cấp một mức độ trừu tượng cao hơn. Chúng cho phép chúng ta giải phóng bản thân khỏi việc triển khai cấp thấp phức tạp và tập trung hơn vào chính logic "tư duy" và "hành động" của agent, từ đó biến ý tưởng thành hiện thực nhanh hơn và tốt hơn.

### 5.1.2 Lựa chọn nền tảng Low-Code

Hiện nay, thị trường nền tảng low-code cho agent và ứng dụng LLM đang trong tình trạng phát triển bùng nổ, mỗi nền tảng đều có định vị và ưu thế riêng. Việc chọn nền tảng nào thường phụ thuộc vào nhu cầu cốt lõi, nền tảng kỹ thuật và mục tiêu cuối cùng của dự án của bạn. Trong các phần tiếp theo của chương này, chúng ta sẽ tập trung giới thiệu và thực hành một số nền tảng tiêu biểu: Coze, Dify, FastGPT và n8n. Trước đó, hãy giới thiệu ngắn gọn về chúng.

**Coze**

- **Định vị cốt lõi**: Được phát triển bởi ByteDance, Coze<sup>[1]</sup> tập trung vào trải nghiệm xây dựng Agent zero-code/low-code, cho phép những người dùng không có nền tảng lập trình dễ dàng tạo ra sản phẩm.
- **Phân tích tính năng**: Coze có một giao diện trực quan cực kỳ thân thiện. Người dùng có thể tạo agent bằng cách kéo thả plugin, cấu hình knowledge base và thiết lập workflow, hệt như xếp hình LEGO. Nó tích hợp sẵn một thư viện plugin vô cùng phong phú và hỗ trợ xuất bản một chạm lên các nền tảng chủ đạo như Douyin, Feishu và WeChat Official Accounts, giúp đơn giản hóa đáng kể quá trình phân phối.
- **Đối tượng mục tiêu**: Người dùng nhập môn ứng dụng AI, quản lý sản phẩm, nhân sự vận hành và các nhà sáng tạo cá nhân muốn nhanh chóng biến ý tưởng thành sản phẩm tương tác.

**Dify**

- **Định vị cốt lõi**: Dify là một nền tảng phát triển và vận hành ứng dụng LLM mã nguồn mở, đầy đủ tính năng<sup>[2]</sup>, nhằm cung cấp cho các nhà phát triển một giải pháp trọn gói từ xây dựng nguyên mẫu đến triển khai production.
- **Phân tích tính năng**: Nó tích hợp các khái niệm về dịch vụ backend và vận hành mô hình, hỗ trợ nhiều năng lực như Agent workflow, RAG Pipeline, gán nhãn dữ liệu và fine-tuning. Đối với các ứng dụng cấp doanh nghiệp theo đuổi tính chuyên nghiệp, ổn định và khả năng mở rộng, Dify cung cấp một nền tảng vững chắc.
- **Đối tượng mục tiêu**: Các nhà phát triển có nền tảng kỹ thuật nhất định, các đội nhóm cần xây dựng ứng dụng AI cấp doanh nghiệp có khả năng mở rộng.

**FastGPT**

- **Định vị cốt lõi**: FastGPT là một nền tảng hỏi đáp knowledge base và công cụ xây dựng Agent mã nguồn mở dựa trên LLM<sup>[3]</sup>, tập trung vào việc cung cấp các giải pháp RAG (Retrieval-Augmented Generation - Tạo sinh tăng cường truy xuất) dễ sử dụng và năng lực điều phối workflow trực quan.
- **Phân tích tính năng**: Ưu thế cốt lõi của FastGPT nằm ở việc tối ưu hóa cực độ cho các tình huống hỏi đáp knowledge base. Nó cung cấp một pipeline RAG hoàn chỉnh từ nhập dữ liệu, tự động chia nhỏ văn bản, lưu trữ vector hóa đến truy xuất thông minh, và hỗ trợ điều phối các luồng hội thoại phức tạp và workflow Agent thông qua giao diện trực quan (module Flow). Nền tảng áp dụng thiết kế trung lập về mô hình, kết nối linh hoạt với nhiều mô hình lớn chủ đạo trong nước và quốc tế như OpenAI, Claude và Tongyi Qianwen, đồng thời cung cấp các giao diện API toàn diện và một chợ plugin để tích hợp nhanh với các hệ thống hiện có như WeChat Work, DingTalk và Feishu.
- **Đối tượng mục tiêu**: Các nhà phát triển và đội nhóm doanh nghiệp vừa và nhỏ muốn nhanh chóng xây dựng dịch vụ khách hàng thông minh, trợ lý tri thức nội bộ, hoặc robot hỏi đáp tài liệu dựa trên knowledge base riêng tư, cũng như những người đam mê công nghệ quan tâm đến RAG muốn hạ thấp rào cản triển khai.

**n8n**

- **Định vị cốt lõi**: Về bản chất, n8n là một công cụ tự động hóa workflow mã nguồn mở<sup>[4]</sup>, không phải là một nền tảng LLM thuần túy. Trong những năm gần đây, nó đã tích cực tích hợp các năng lực AI.

- **Phân tích tính năng**: Sức mạnh của n8n nằm ở khả năng "kết nối". Nó có hàng trăm node dựng sẵn có thể dễ dàng kết nối nhiều dịch vụ SaaS, cơ sở dữ liệu và API thành các quy trình nghiệp vụ tự động hóa phức tạp. Bạn có thể nhúng các node LLM vào quy trình này, biến nó thành một phần của toàn bộ chuỗi tự động hóa. Mặc dù nó không chuyên về chức năng LLM như hai nền tảng đầu tiên, nhưng năng lực tự động hóa tổng quát của nó là độc nhất. Tuy nhiên, đường cong học tập của nó cũng tương đối dốc.

- **Đối tượng mục tiêu**: Các nhà phát triển và doanh nghiệp cần tích hợp sâu năng lực AI vào các quy trình nghiệp vụ hiện có và đạt được sự tự động hóa được tùy chỉnh cao.

Trong các tiểu mục tiếp theo, chúng ta sẽ lần lượt thực hành với các nền tảng này, và cảm nhận rõ nét hơn sức hấp dẫn riêng của chúng thông qua các thao tác thực tế.

## 5.2 Nền tảng Một: Coze
Coze là một công cụ tạo AI agent cực kỳ tuyệt vời! Nó cũng đồng thời là nền tảng agent được sử dụng rộng rãi nhất trên thị trường hiện nay. Với giao diện trực quan và các module chức năng phong phú, nền tảng này cho phép người dùng dễ dàng tạo ra nhiều loại ứng dụng agent khác nhau, chẳng hạn như chatbot có thể trò chuyện với bạn, cỗ máy sáng tạo tự động viết truyện, và thậm chí trực tiếp giúp bạn biến truyện thành MV phim! Một trong những điểm nổi bật của nó là năng lực tích hợp hệ sinh thái mạnh mẽ. Các agent đã phát triển có thể được xuất bản một chạm lên các nền tảng chủ đạo như WeChat, Feishu và Doubao, đạt được việc triển khai đa nền tảng liền mạch. Đối với người dùng doanh nghiệp, Coze cũng cung cấp các giao diện API linh hoạt, hỗ trợ tích hợp năng lực agent vào các hệ thống nghiệp vụ hiện có, đạt được việc xây dựng ứng dụng AI theo kiểu "xếp hình".
### 5.2.1 Các module chức năng của Coze
(1) Tổng quan giao diện nền tảng

Giới thiệu bố cục tổng thể: Gần đây, Coze lại cập nhật giao diện UI của mình, như trong Hình 5.1. Giờ đây thanh bên ngoài cùng bên trái là không gian phát triển của trang chủ nền tảng Coze, bao gồm phát triển dự án cốt lõi, thư viện tài nguyên, đánh giá hiệu quả và cấu hình không gian. Khu vực phía dưới là không gian tài liệu hỗ trợ cho việc phát triển Coze, bao gồm các template chính thức để sao chép một chạm, ưu thế lớn nhất của Coze - một cửa hàng plugin phong phú và đa dạng, cộng đồng agent lớn nhất với vô số lựa chọn, quản lý API để kiểm thử API, cũng như tài liệu hướng dẫn chi tiết và quản lý tổng quát dành cho doanh nghiệp. Bên phải có bốn template. Trên cùng là thông báo cập nhật mới nhất của Coze, cho bạn biết về tiến độ mới nhất của Coze để bạn có thể tìm hiểu về các công cụ và tính năng mới nhất. Bên dưới đó là hướng dẫn cho người mới bắt đầu. Nhấp vào đó và bạn sẽ thấy tài liệu hướng dẫn cho người mới bắt đầu, và bạn có thể bắt đầu xây dựng agent trong vài phút. Tiếp theo là những người bạn theo dõi và các đề xuất agent. Tại đây bạn cũng có thể theo dõi các nhà phát triển AI yêu thích và đánh dấu các agent của họ để sử dụng cho riêng mình.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-01.png" alt="Image description" width="90%"/>
  <p>Hình 5.1 Sơ đồ tổng thể của nền tảng Coze Agent</p>
</div>

(2) Giới thiệu chức năng cốt lõi

Đầu tiên, chúng ta nhấp vào dấu cộng trên thanh bên trái để thấy điểm truy cập tạo agent. Hiện tại có hai loại ứng dụng AI: một là tạo agent, và loại còn lại gọi là ứng dụng (application). Trong đó, agent được chia thành chế độ lập kế hoạch tự chủ đơn-agent (single-agent), chế độ luồng hội thoại đơn-agent, và chế độ đa-agent (multi-agent). Ứng dụng AI cũng được chia thành hai loại: bạn không chỉ có thể thiết kế giao diện người dùng cho desktop và web, mà còn có thể dễ dàng xây dựng giao diện cho mini-program và H5, như trong Hình 5.2.
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-02.png" alt="Image description" width="90%"/>
  <p>Hình 5.2 Điểm truy cập tạo Coze Agent</p>
</div>
Không gian dự án là kho lưu trữ agent của bạn, nơi lưu trữ tất cả các agent hoặc ứng dụng mà bạn đã phát triển hoặc sao chép. Đây cũng là nơi bạn sẽ ghé thăm thường xuyên nhất khi phát triển agent trong Coze, như trong Hình 5.3.
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-03.png" alt="Image description" width="90%"/>
  <p>Hình 5.3 Không gian dự án Coze Agent</p>
</div>
Thư viện tài nguyên là kho vũ khí cốt lõi của bạn để phát triển Coze agent. Thư viện tài nguyên lưu trữ các workflow, knowledge base, card, thư viện prompt và một loạt các công cụ khác để phát triển agent. Loại agent bạn có thể tạo ra trước tiên phụ thuộc vào năng lực của mô hình, nhưng quan trọng nhất, nó phụ thuộc vào cách bạn trang bị "trang bị và kỹ năng" cho agent. Mô hình quyết định giới hạn dưới của agent, nhưng thư viện tài nguyên Coze cho bạn giới hạn trên vô hạn cho năng lực của agent, cho phép bạn phát triển theo ý tưởng, trí tưởng tượng và sự sáng tạo của riêng mình, như trong Hình 5.4.
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-04.png" alt="Image description" width="90%"/>
  <p>Hình 5.4 Thư viện tài nguyên Coze Agent</p>
</div>
Cấu hình không gian bao gồm một kênh quản lý thống nhất cho agent, plugin, workflow và các kênh xuất bản, cũng như quản lý mô hình nơi bạn có thể thấy các mô hình lớn khác nhau mà bạn gọi, như trong Hình 5.5.
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-05.png" alt="Image description" width="90%"/>
  <p>Hình 5.5 Các kênh xuất bản Coze Agent</p>
</div>
Nếu phải tóm tắt đơn giản về việc phát triển agent của Coze, tôi sẽ so sánh nó với các thành phần khác nhau của một trò chơi. Việc kết hợp từng phần để tạo ra những agent tuyệt vời rất giống với việc chơi một "trò chơi". Mỗi khi bạn hoàn thành một agent, cảm giác như đánh bại một con boss và thu được rất nhiều thứ, dù đó là "kinh nghiệm" hay "trang bị".

- Workflow: Bản đồ lộ trình vượt màn
- Luồng hội thoại: Đối thoại NPC để vượt màn
- Plugin: Thẻ kỹ năng nhân vật
- Knowledge base: Bách khoa toàn thư trò chơi
- Card: Thanh vật phẩm nhanh
- Prompt: Phím điều khiển chuyển động nhân vật
- Database: "Lưu game trên đám mây"
- Quản lý xuất bản: Người kiểm duyệt màn chơi
- Quản lý mô hình: Thư viện nhân vật trò chơi hoặc hệ thống tạo nhân vật
- Đánh giá hiệu quả: Hệ thống chấm điểm màn chơi




### 5.2.2 Xây dựng trợ lý "Bản tin AI hằng ngày"

**Mô tả tình huống:** Case thực hành này nhằm phân tích sâu năng lực tích hợp plugin của nền tảng Coze và hướng dẫn độc giả xây dựng từ đầu một agent "Bản tin AI hằng ngày" mạnh mẽ. Agent này có thể tự động thu thập các tin tức nổi bật, bài báo học thuật và cập nhật dự án mã nguồn mở mới nhất trong lĩnh vực AI từ nhiều nguồn thông tin (bao gồm 36Kr, Huxiu, IT Home, InfoQ, GitHub, arXiv) và tổng hợp chúng thành một bản tin sinh động, súc tích theo cách có cấu trúc và chuyên nghiệp.

Thông qua case này, bạn sẽ nắm vững một cách có hệ thống các kỹ năng cốt lõi sau:

  * **Tổng hợp thông tin đa nguồn:** Sử dụng hệ sinh thái plugin của Coze để đạt được sự tích hợp liền mạch các luồng dữ liệu xuyên nền tảng, xuyên loại hình.
  * **Định nghĩa hành vi Agent:** Thông qua thiết lập vai trò và prompt engineering, kiểm soát chính xác việc thực thi tác vụ và tạo sinh nội dung của agent để đảm bảo đầu ra đáp ứng các tiêu chuẩn chuyên nghiệp đã định.
  * **Xây dựng workflow tự động hóa:** Học cách liên kết nhiều bước như thu thập dữ liệu, xử lý nội dung và đầu ra được định dạng thành một workflow tự động hóa hiệu quả.



**Bước 1: Thêm và cấu hình các plugin nguồn thông tin**

Nhiệm vụ hàng đầu khi xây dựng agent "Bản tin AI hằng ngày" là kết nối nó với các nguồn thông tin phong phú và uy tín. Trên nền tảng Coze, điều này được thực hiện bằng cách thêm và cấu hình các plugin tương ứng.

1.  **Tích hợp plugin:** Trong thư viện plugin của Coze, tìm kiếm và thêm các plugin cần thiết. Ví dụ, đăng ký các RSS feed từ các nền tảng truyền thông thông qua plugin **RSS** (như trong Hình 5.6), theo dõi các dự án mã nguồn mở thông qua plugin **GitHub** (như trong Hình 5.7), và lấy các kết quả nghiên cứu học thuật mới nhất thông qua plugin **arXiv** (như trong Hình 5.8).

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-06.png" alt="Image description" width="90%"/>
  <p>Hình 5.6 Plugin nguồn RSS cho các nền tảng truyền thông</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-07.png" alt="Image description" width="90%"/>
  <p>Hình 5.7 Plugin GitHub</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-08.png" alt="Image description" width="90%"/>
  <p>Hình 5.8 Plugin Arxiv</p>
</div>

2.  **Cấu hình cá nhân hóa:** Thực hiện cấu hình chi tiết cho từng plugin để đảm bảo nó có thể lấy chính xác dữ liệu cần thiết. Ví dụ, trong plugin RSS, nhập các liên kết đăng ký RSS cụ thể cho các website như 36Kr và Huxiu; trong plugin GitHub, thiết lập số lượng truy vấn từ khóa và cài đặt cập nhật mới nhất cần theo dõi; trong plugin arXiv, định nghĩa các từ khóa quan tâm như "LLM", "AI", v.v., và định nghĩa số lượng cũng như cài đặt cập nhật mới nhất.

```
RSS Link Configuration

- **36Kr:** https://www.36kr.com/feed
- **Huxiu:** https://rss.huxiu.com/
- **IT Home:** http://www.ithome.com/rss/
- **InfoQ:** https://feed.infoq.com/ai-ml-data-eng/

GitHub Plugin Configuration

- q:AI
- per_page:10
- sort:updated

Arxiv Plugin Configuration

- count: 5
- search_query: AI
- sort_by: 2
```

3.  **Điều phối và kết nối:** Trong giao diện điều phối trực quan của agent, sử dụng các plugin nguồn thông tin đã cấu hình này (như `rss_24Hbj`, `searchRepository`, `arxiv`, v.v.) làm các node đầu vào dữ liệu và kết nối chúng với các module xử lý logic tiếp theo (như module **Large Model** / Mô hình lớn) để xây dựng một đường dẫn xử lý dữ liệu hoàn chỉnh, như trong Hình 5.9.
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-09.png" alt="Image description" width="90%"/>
  <p>Hình 5.9 Lưu đồ điều phối Bản tin AI hằng ngày</p>
</div>


**Bước 2: Thiết lập vai trò và prompt của Agent**

Thiết lập vai trò và viết prompt là những bước cốt lõi trong việc định nghĩa hành vi và chất lượng đầu ra của agent. Bước này nhằm chuyển các chỉ dẫn trừu tượng thành các tác vụ cụ thể mà agent có thể hiểu và thực thi.

(1) Thiết lập vai trò

Chúng ta thiết lập agent thành một **biên tập viên truyền thông công nghệ cấp cao và có uy tín**. Vai trò này mang lại cho agent một định vị chuyên nghiệp rõ ràng, cho phép nó bắt chước lối tư duy của các biên tập viên chuyên nghiệp trong quá trình sáng tạo nội dung sau này, thực hiện sàng lọc, tổng hợp và tóm tắt thông tin một cách hiệu quả.

(2) Viết và cấu trúc hóa prompt

Prompt là cẩm nang chỉ dẫn để agent thực thi tác vụ. Chúng ta chia chúng thành **System Prompt và User Prompt** để đảm bảo các chỉ dẫn rõ ràng, đầy đủ và có thể kiểm soát.

**System Prompt**

System prompt được dùng để định nghĩa các nguyên tắc hành vi dài hạn và đặc tả định dạng đầu ra của agent.

```
# Role
You are a senior and authoritative technology media editor, skilled at efficiently and precisely integrating and creating highly professional technology briefs, with deep analytical and integration capabilities especially in AI field technical developments, cutting-edge academic research results, and popular open-source projects.

## Workflow
### Daily Report Output Format
1. The daily report should prominently display "AI Daily Report", "by@jasonhuang", and the current date at the beginning, for example: "AI Daily Report | September 24, 2025 | by@jasonhuang".
2. <!!!important!!!> Add a unique Emoji symbol at the beginning of each title based on the different content of each AI technology news, each AI academic paper, and each AI open-source project.
3. All output content must be highly relevant to AI, LLM, AIGC, large models, and other technical topics, firmly excluding any irrelevant information, advertisements, and marketing content.
4. Must provide the original link for each item (including AI technology news, AI academic papers, AI open-source projects).
5. Provide a brief and precise summary description for each news item or project output.
```

**User Prompt**

User prompt được dùng để định nghĩa các chỉ dẫn tác vụ cụ thể và các nguồn dữ liệu.

```
- **Information Extraction and Integration:** From input sources `{{articles}}`, `{{articles1}}`, `{{articles2}}`, and `{{articles3}}`, filter and extract article titles and corresponding links related to AI, large models, AIGC, LLM, and other topics, and organize them into the **"AI Technology News"** module.
- **Academic Paper Summary:** From input source `{{arxiv}}`, based on fields `arxiv_title` and `arxiv_link`, summarize and organize the latest paper content to form the **"AI Academic Papers"** module.
- **Open-Source Project Filtering:** From input source `{{GitHub}}`, filter out the **5 most prominent and influential AI open-source projects**. Extract the titles and corresponding links of these projects and organize them into the **"AI Open-Source Projects"** module.

# Attention
- Strictly follow the daily report output format defined in the system prompt.
- The total output content should be: **10 AI technology news items, 5 AI academic papers, 5 AI open-source projects**.
```



**Bước 3: Kiểm thử, gỡ lỗi và xuất bản đa kênh**

Sau khi hoàn thành việc xây dựng logic cốt lõi của agent, phải thực hiện kiểm thử và gỡ lỗi nghiêm ngặt để đảm bảo đầu ra của nó đáp ứng kỳ vọng.

**Xem trước khi chạy:** Chạy agent trong giao diện xem trước của nền tảng Coze và quan sát nội dung bản tin mà nó tạo ra.

```
# AI Daily Report by@jasonhuang 2025-09-24

## 🚀 AI Technology News

🤖 **Zhiyuan Robot GO-1 Universal Embodied Foundation Model Fully Open-Sourced**
Link: https://36kr.com/p/3479085489708163?f=rss
Summary: Zhiyuan Robot announced the full open-sourcing of its GO-1 universal embodied foundation model, providing powerful AI foundational capabilities for the robotics field.

🔬 **Microsoft Overcomes Data Center Chip Cooling Bottleneck: Microfluidics + AI Precision Cooling**
Link: https://www.ithome.com/0/885/391.htm
Summary: Microsoft achieves precise temperature control of data center chips through the combination of microfluidic technology and AI algorithms, improving energy efficiency.
......

## 📚 AI Academic Papers

🧪 **Lyra: Generative 3D Scene Reconstruction via Video Diffusion Model Self-Distillation**
Link: http://arxiv.org/pdf/2509.19296v1
Summary: Proposes an innovative framework for 3D scene generation through video diffusion model self-distillation, without requiring multi-view training data.

📊 **The ICML 2023 Ranking Experiment: Examining Author Self-Assessment in ML/AI Peer Review**
Link: http://arxiv.org/pdf/2408.13430v3
Summary: Studies the effectiveness of author self-assessment in machine learning conference review processes and proposes methods to improve review mechanisms.
......

## 💻 AI Open-Source Projects

🤖 **llmling-agent - Multi-Agent Workflow Framework**
Link: https://github.com/phil65/llmling-agent
Summary: Multi-agent interaction framework supporting YAML configuration and programming methods, integrating MCP and ACP protocol support.

🚌 **College_EV_AI_Transportation - Campus AI Electric Transportation System**
Link: https://github.com/LuisMc2005v/College_EV_AI_Transportation
Summary: AI-driven campus electric transportation optimization system, achieving real-time tracking and efficient carpooling services.
......
```

Hãy kiểm tra cẩn thận độ chính xác nội dung, tính đầy đủ của định dạng và phong cách ngôn ngữ của bản tin. Nếu phát hiện những phần không đáp ứng kỳ vọng, hãy quay lại giai đoạn cấu hình prompt hoặc plugin để điều chỉnh chi tiết. Ví dụ, nếu nội dung chưa đủ súc tích, hãy sửa yêu cầu tóm tắt trong prompt; nếu việc thu thập dữ liệu không chính xác, hãy kiểm tra các tham số cấu hình plugin.

Xuất bản đa kênh: Coze cung cấp khả năng xuất bản agent lên nhiều nền tảng ứng dụng chủ đạo (như WeChat, Doubao, Feishu, v.v.) chỉ với một chạm, mở rộng đáng kể các tình huống ứng dụng của agent, như trong Hình 5.10.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-10.png" alt="Image description" width="90%"/>
  <p>Hình 5.10 Các kênh xuất bản đa dạng của nền tảng Coze</p>
</div>

Sau khi agent được xuất bản, chúng ta có thể thấy AI agent mà mình đã tạo trong cửa hàng Coze, và nó cũng có thể được tích hợp vào các ứng dụng AI để cung cấp dịch vụ cho người dùng, như trong Hình 5.11 và 5.12. Đây cũng là [Liên kết trải nghiệm Agent Bản tin AI hằng ngày](https://www.coze.cn/store/agent/7506052197071962153?bot_id=true&bid=6hkt3je8o2g16)

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-11.png" alt="Image description" width="90%"/>
  <p>Hình 5.11 AI Agent - Bản tin AI hằng ngày</p>
</div>

Hơn nữa, chúng ta có thể nhấp vào [liên kết trải nghiệm](https://www.coze.cn/store/project/7458678213078777893?from=store_search_suggestion&bid=6gu3cmr7k5g1i) này để xem Bản tin AI hằng ngày trong ứng dụng AI.
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-12.png" alt="Image description" width="90%"/>
  <p>Hình 5.12 Bản tin AI hằng ngày trong ứng dụng AI</p>
</div>
**Cấu hình xuất bản:** Nếu bạn muốn xuất bản agent của riêng mình, bạn cũng cần cấu hình tên, ảnh đại diện và lời chào phù hợp cho agent trước khi xuất bản để mang lại trải nghiệm người dùng thân thiện hơn, như trong Hình 5.13 và 5.14.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-13.png" alt="Image description" width="90%"/>
  <p>Hình 5.13 Cấu hình thông tin cơ bản cho Agent</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/coze-14.png" alt="Image description" width="90%"/>
  <p>Hình 5.14 Cấu hình lời chào mở đầu và câu hỏi gợi ý cho Agent</p>
</div>


### 5.2.3 Phân tích ưu điểm và hạn chế của Coze

**Ưu điểm:**

  * **Hệ sinh thái plugin mạnh mẽ:** Ưu thế cốt lõi của nền tảng Coze nằm ở thư viện plugin phong phú, cho phép agent dễ dàng truy cập các dịch vụ và nguồn dữ liệu bên ngoài, đạt được khả năng mở rộng chức năng cao.
  * **Điều phối trực quan trực giác:** Nền tảng cung cấp giao diện điều phối workflow trực quan với rào cản thấp. Người dùng có thể xây dựng các workflow phức tạp thông qua "kéo và thả" mà không cần kiến thức lập trình sâu, giảm đáng kể độ khó phát triển.
  * **Kiểm soát prompt linh hoạt:** Thông qua thiết lập vai trò chính xác và viết prompt, người dùng có thể kiểm soát chi tiết hành vi và việc tạo sinh nội dung của agent, đạt được đầu ra được tùy chỉnh cao. Nó cũng hỗ trợ quản lý prompt và template, tạo thuận lợi lớn cho các nhà phát triển trong việc phát triển agent.
  * **Triển khai đa nền tảng thuận tiện:** Hỗ trợ xuất bản cùng một agent lên các nền tảng ứng dụng khác nhau, đạt được sự tích hợp và ứng dụng đa nền tảng liền mạch. Hơn nữa, Coze liên tục tích hợp các nền tảng mới vào hệ sinh thái của mình, với ngày càng nhiều nhà sản xuất điện thoại và phần cứng dần hỗ trợ việc xuất bản agent Coze.

**Hạn chế:**

  * **Không hỗ trợ MCP:** Tôi cho rằng đây là điều chí mạng nhất. Mặc dù chợ plugin của Coze cực kỳ phong phú và hấp dẫn, việc không hỗ trợ MCP có thể trở thành một xiềng xích cản trở sự phát triển của nó. Nếu được mở ra, đó sẽ lại là một tính năng đột phá.
  * **Độ phức tạp cao của một số cấu hình plugin:** Đối với các plugin cần API Key hoặc các tham số nâng cao khác, người dùng có thể cần một số nền tảng kỹ thuật để hoàn thành cấu hình đúng. Việc điều phối workflow phức tạp cũng không phải là thứ có thể thành thạo với con số không; nó đòi hỏi một số kiến thức nền tảng về JavaScript hoặc Python.
  * **Không thể trực tiếp nhập file JSON:** Trước đây, ứng dụng không có chức năng xuất/nhập, nhưng phiên bản trả phí hiện đã có. Tuy nhiên, file xuất/nhập không phải là file JSON như Dify hay N8n; nó là một file ZIP. Điều này có nghĩa là bạn chỉ có thể xuất từ ứng dụng rồi nhập file ZIP. Tuy nhiên, bạn có thể dùng một mẹo: trong giao diện bố cục, nhấn Ctrl+A để chọn tất cả, sau đó Ctrl+C để sao chép bố cục, rồi dán nó vào một workflow trống khác hoặc các workflow khác.


## 5.3 Nền tảng Hai: Dify
### 5.3.1 Giới thiệu về Dify và hệ sinh thái của nó

Dify là một nền tảng phát triển ứng dụng mô hình ngôn ngữ lớn (LLM) mã nguồn mở, tích hợp các khái niệm về Backend as a Service (BaaS) và LLMOps, cung cấp hỗ trợ toàn quy trình từ thiết kế nguyên mẫu đến triển khai production, như trong Hình 5.15. Nó áp dụng kiến trúc mô-đun hóa phân tầng, chia thành tầng dữ liệu, tầng phát triển, tầng điều phối và tầng nền tảng, với mỗi tầng được tách rời để dễ dàng mở rộng.

Dify có tính trung lập và tương thích cao về mô hình: dù là mô hình mã nguồn mở hay thương mại, người dùng có thể tích hợp chúng thông qua cấu hình đơn giản và gọi năng lực suy luận của chúng qua một giao diện thống nhất. Nó tích hợp sẵn hỗ trợ cho hàng trăm LLM mã nguồn mở hoặc độc quyền, bao gồm các mô hình như GPT, Deepseek, Llama, cũng như bất kỳ mô hình nào tương thích với OpenAI API.

Đồng thời, Dify hỗ trợ triển khai cục bộ (khởi động một chạm bằng Docker Compose chính thức) và triển khai đám mây. Người dùng có thể chọn tự triển khai Dify trong môi trường cục bộ/riêng tư (đảm bảo quyền riêng tư dữ liệu) hoặc sử dụng dịch vụ đám mây SaaS chính thức (được trình bày chi tiết trong phần mô hình kinh doanh bên dưới). Tính linh hoạt trong triển khai này khiến nó phù hợp cho các môi trường intranet doanh nghiệp có yêu cầu bảo mật hoặc các nhóm nhà phát triển có yêu cầu về sự tiện lợi trong vận hành.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-01.png" alt="Image description" width="90%"/>
  <p>Hình 5.15 Trang web chính thức của Dify</p>
</div>

Hệ sinh thái plugin Marketplace: Dify Marketplace cung cấp chức năng quản lý plugin trọn gói và triển khai một chạm, cho phép các nhà phát triển khám phá, mở rộng hoặc gửi plugin, mang lại nhiều khả năng hơn cho cộng đồng, như trong Hình 5.16.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-02.png" alt="Image description" width="90%"/>
  <p>Hình 5.16 Hệ sinh thái plugin Dify Marketplace</p>
</div>
Marketplace bao gồm:


- Models (Mô hình)
- Tools (Công cụ)
- Agent Strategies (Chiến lược Agent)
- Extensions (Tiện ích mở rộng)
- Bundles (Gói)

Hiện tại, Dify Marketplace có hơn 8.677 plugin bao phủ nhiều chức năng và tình huống ứng dụng khác nhau. Trong đó, các plugin được đề xuất chính thức bao gồm:
- Google Search: langgenius/google
- Azure OpenAI: langgenius/azure_openai
- Notion: langgenius/notion
- DuckDuckGo: langgenius/duckduckgo


Dify cung cấp hỗ trợ phát triển mạnh mẽ cho các nhà phát triển plugin, bao gồm chức năng gỡ lỗi từ xa cộng tác liền mạch với các IDE phổ biến, đòi hỏi thiết lập môi trường tối thiểu. Các nhà phát triển có thể kết nối với dịch vụ SaaS của Dify trong khi chuyển tiếp tất cả các thao tác plugin đến môi trường cục bộ để kiểm thử. Cách tiếp cận thân thiện với nhà phát triển này nhằm trao quyền cho các nhà sáng tạo plugin và đẩy nhanh sự đổi mới trong hệ sinh thái Dify. Đây cũng là lý do Dify có thể trở thành một trong những nền tảng agent thành công nhất hiện nay, bởi vì mô hình đều có thể tích hợp, prompt và điều phối đều có thể sao chép, nhưng sự hiện diện và phong phú của các plugin công cụ trực tiếp quyết định liệu agent của bạn có thể đạt được kết quả tốt hơn hay các chức năng mạnh mẽ ngoài mong đợi hay không.

### 5.3.2 Xây dựng một trợ lý cá nhân Super Agent
> **✨✨ Hướng dẫn thao tác chi tiết**: Vui lòng tham khảo **[Hướng dẫn thao tác từng bước tạo Dify Agent](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra03-Dify智能体创建保姆级操作流程.md)**

Trong case Coze trước đó, chúng ta đã xây dựng một agent bản tin AI hằng ngày. Mặc dù chức năng của nó rõ ràng, nhưng năng lực tạo bản tin đơn lẻ của nó có phần hạn chế. Phần này sẽ sử dụng Dify để xây dựng một trợ lý cá nhân super agent đầy đủ chức năng, bao phủ nhiều tình huống như hỏi đáp hằng ngày, tối ưu hóa văn bản, tạo sinh đa phương thức (multimodal) và phân tích dữ liệu. Trước khi bắt đầu, hãy tìm hiểu ngắn gọn về giao diện chính và các module chức năng của Dify.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-14.png" alt="Image description" width="90%"/>
  <p>Hình 5.17 Trang chủ xây dựng Dify Agent</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-18.png" alt="Image description" width="90%"/>
  <p>Hình 5.18 Thư viện template chính thức của Dify</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-15.png" alt="Image description" width="90%"/>
  <p>Hình 5.19 Knowledge Base của Dify</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-16.png" alt="Image description" width="90%"/>
  <p>Hình 5.20 Chợ plugin của Dify</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-17.png" alt="Image description" width="90%"/>
  <p>Hình 5.21 Cấu hình mô hình lớn của Dify</p>
</div>

**(1) Tạo plugin và cấu hình MCP**

Trước khi xây dựng agent, phải hoàn thành việc cài đặt plugin cần thiết và cấu hình MCP trước. Như trong Hình 5.22, đây là các plugin cốt lõi cần thiết cho case này.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-19.png" alt="Image description" width="90%"/>
  <p>Hình 5.22 Cấu hình cài đặt plugin Dify</p>
</div>

Các plugin được đánh dấu bằng khung màu đỏ trong hình cần được tìm kiếm và cài đặt từ chợ plugin của Dify. Người dùng có thể nhấp để xem chi tiết nhằm hiểu các chức năng cụ thể của từng plugin.

Tiếp theo, cấu hình MCP (Model Context Protocol). Chúng ta sẽ không đi sâu vào nguyên lý chi tiết của MCP ở đây; chúng ta sẽ tập trung minh họa cách sử dụng các dịch vụ MCP được triển khai trên đám mây. Case này sử dụng chợ MCP của cộng đồng ModelScope trong nước để minh họa, như trong Hình 5.23.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-20.png" alt="Image description" width="90%"/>
  <p>Hình 5.23 Chợ MCP của cộng đồng ModelScope</p>
</div>

Mở chợ MCP của cộng đồng ModelScope và chọn loại hosted (được lưu trữ). Lấy Amap MCP làm ví dụ, sau khi vào trang chủ của nó, chọn chế độ SSE ở phía bên phải và nhấp vào cấu hình kết nối để tạo một JSON cấu hình MCP chuyên dụng, như trong Hình 5.24. MCP hỗ trợ nhiều chế độ giao tiếp, nhưng việc sử dụng giao tiếp chế độ SSE trong Dify mượt mà và ổn định hơn, nên khuyến nghị dùng chế độ SSE.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-21.png" alt="Image description" width="90%"/>
  <p>Hình 5.24 Ví dụ cấu hình Amap MCP</p>
</div>

**(2) Thiết kế Agent và trình diễn hiệu quả**

Case này sẽ tạo một trợ lý cá nhân toàn diện bao phủ các module chức năng sau:

- Hỏi đáp đời sống hằng ngày
- Trau chuốt và tối ưu hóa văn bản
- Tạo sinh nội dung đa phương thức (hình ảnh, video)
- Truy vấn dữ liệu và phân tích trực quan hóa
- Tích hợp công cụ MCP (Amap, gợi ý ăn uống, thông tin tin tức)

Kiến trúc điều phối agent tổng thể được thể hiện trong Hình 5.25.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-12.png" alt="Image description" width="90%"/>
  <p>Hình 5.25 Điều phối Agent</p>
</div>

Đối với kiến trúc đa-agent, chúng ta sử dụng một bộ phân loại câu hỏi (question classifier) để định tuyến thông minh. Trong bộ phân loại, định nghĩa các chức năng cốt lõi và phạm vi tác vụ cho từng agent để đảm bảo yêu cầu của người dùng có thể được phân phối chính xác đến các module xử lý tương ứng.

**Module Trợ lý hằng ngày**

Đây là một module hội thoại cơ bản được cấu hình với một mô hình ngôn ngữ lớn và các công cụ thời gian, đóng vai trò là dịch vụ hỏi đáp tổng quát dự phòng.

Cấu hình prompt:
```
# Role: Daily Question Consultation Expert

## Profile
- language: Chinese
- description: Specializes in answering general questions in users' daily lives, providing practical, accurate, and easy-to-understand advice and answers
- background: Possesses rich life experience and extensive knowledge reserves, skilled at simplifying complex problems
- personality: Kind and friendly, patient and meticulous, pragmatic and reliable
- expertise: Daily life, health and wellness, family management, interpersonal relationships, practical tips


## Skills

1. Problem Analysis Ability
   - Quick Understanding: Rapidly grasp the core points of user questions
   - Classification Recognition: Accurately judge the life domain to which the question belongs
   - Demand Mining: Deeply understand users' potential needs
   - Priority Sorting: Reasonably assess the importance and urgency of problems

2. Answer Providing Ability
   - Knowledge Integration: Comprehensively apply multi-domain knowledge to provide answers
   - Solution Formulation: Provide specific and feasible solutions
   - Step Decomposition: Break down complex problems into simple steps
   - Alternative Solutions: Prepare multiple backup solutions for users to choose from

3. Communication and Expression Ability
   - Popular Language: Use simple and easy-to-understand everyday language
   - Clear Logic: Organize answer content in a well-organized manner
   - Illustrative Examples: Help understanding through specific cases
   - Highlight Key Points: Emphasize key information and precautions

## Rules

1. Answer Principles:
   - Practicality First: Ensure the advice provided is actionable
   - Accuracy Guarantee: Give answers based on reliable information and common sense
   - Neutral and Objective: Avoid personal bias and subjective assumptions
   - Moderate Advice: Provide appropriate depth of answers based on problem complexity

2. Code of Conduct:
   - Timely Response: Quickly respond to users' questions
   - Patient and Meticulous: Maintain patience with repetitive or simple questions
   - Active Guidance: Encourage users to provide more background information
   - Continuous Improvement: Optimize answer quality based on feedback


## Workflows

- Goal: Provide users with practical and reliable daily problem solutions
- Step 1: Carefully read and understand the daily questions raised by users
- Step 2: Analyze the problem type and users' potential needs
- Step 3: Provide specific and feasible suggestions based on common sense and experience
- Step 4: Organize answer content in easy-to-understand language
- Step 5: Check the practicality and safety of the answer


## Initialization
As a daily question consultation expert, you must abide by the above Rules and execute tasks according to Workflows.
```

Phần trình diễn hiệu quả được thể hiện trong Hình 5.26:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-03.png" alt="Image description" width="90%"/>
  <p>Hình 5.26 Trợ lý hằng ngày</p>
</div>

**Module Tối ưu hóa văn bản**

Theo báo cáo dữ liệu của OpenAI, hơn 60% người dùng sử dụng ChatGPT cho các tác vụ liên quan đến tối ưu hóa văn bản, bao gồm trau chuốt, sửa đổi, mở rộng và rút gọn. Do đó, tối ưu hóa văn bản là một tình huống nhu cầu tần suất cao, và chúng ta biến nó thành module chức năng cốt lõi thứ hai.

Cấu hình prompt:
```
# I. Role Setting (Role)
You are a professional copywriting optimization expert with rich experience in marketing copywriting and optimization, skilled at improving the attractiveness, conversion rate, and readability of copy. Your perspective is from the angle of the target audience and marketing goals, with professional boundaries limited to the copywriting optimization field, not involving technical implementation or product development.

# II. Background
The user has provided a piece of original copy that needs your optimization to improve its overall effectiveness. Background information includes: the copy may be used for marketing, brand promotion, or information communication scenarios, but the specific use is not detailed. The known condition is that the user hopes the copy is more attractive, clear, or persuasive, but has not provided the original copy content, so you need to work based on general optimization principles.

# III. Task Objectives (Task)
- Analyze and optimize the structure, language, and style of the copy to make it more in line with the preferences of the target audience.
- Improve the attractiveness, readability, and conversion potential of the copy, ensuring clear information delivery.
- Make adjustments according to common optimization principles (such as conciseness, emotional resonance, call to action, etc.), without content rewriting unless necessary.
- While maintaining core information, appropriately expand and enrich copy content to provide a more comprehensive optimized version.

# IV. Limitation Prompts (Limit)
- Avoid changing the core information or intent of the original copy unless explicitly requested by the user.
- Do not add fictional or irrelevant content, ensuring optimization is based on logic and best practices.
- Avoid using overly technical or professional terminology unless the target audience is professionals.
- Do not involve optimization of images, layouts, or other non-text elements.

# V. Output Format Requirements (Example)
The output should be optimized copy text with clear structure, fluent language, and substantial content. For example:
- If the original copy is "Our product is very good, come and buy it"
The optimized version can be: "In this era full of choices, what truly touches people's hearts is never exaggerated propaganda, but good products that can withstand the test of time and users. Our product is exactly that. It not only pays attention to details and quality in design but also continuously polishes and innovates in function, just to bring a better user experience to every user. Whether it's the texture of the appearance or the stability of performance, we always adhere to high standards and strict requirements, striving to make every customer who chooses us feel the surprise of value for money.
We deeply understand that purchasing a product is not just a simple consumption but a choice of lifestyle. Therefore, from material selection, craftsmanship to after-sales service, we have poured full sincerity and professionalism into every link, carefully guarding your every experience. Whether you pursue practicality, value quality, or want unique personalization, our products can provide you with ideal solutions.
Now, let us prove everything with action. A truly good product does not need too much embellishment; it itself is the best spokesperson. Act now, choose us, let quality change life, and have a different experience from now on!"
- The output should directly present optimized content without additional explanations or annotations unless requested by the user. Please ensure that the optimized copy content is richer and more complete, and the optimized copy text must exceed 500 words.
```

Phần trình diễn hiệu quả được thể hiện trong Hình 5.27:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-04.png" alt="Image description" width="90%"/>
  <p>Hình 5.27 Trợ lý văn bản</p>
</div>

**Module Tạo sinh đa phương thức**

Tạo sinh hình ảnh và video là một tình huống ứng dụng tần suất cao khác. Với sự tiến hóa của các mô hình như Jimeng tạo ảnh và Google Imagen, cũng như những đột phá trong công nghệ tạo sinh video như Keling, Google Veo 3, seedance2.0 và OpenAI Sora 2, chất lượng tạo sinh nội dung đa phương thức đã đạt đến mức thực dụng.

Case này sử dụng plugin Jimeng để triển khai tạo sinh hình ảnh và video. Các bước cấu hình như sau:

1. Thêm plugin tạo ảnh/video Jimeng vào workflow
2. Cấu hình tham số (như tỉ lệ ảnh 1:1, lựa chọn mô hình seedream4.5/5.0 và seedance1.5/2.0)
3. Xuất file đã tạo

Ở đây chúng ta sử dụng plugin của Jimeng để gọi các mô hình mới nhất, cụ thể là seedream5.0 và seedance2.0. Chất lượng của cả hình ảnh và video đều đã được cải thiện đáng kể. Chúng ta cũng sử dụng vòng lặp (loop) để truy xuất tác vụ video, và bổ sung phần minh họa case bằng việc truy xuất tham số và phán đoán điều kiện. Để biết chi tiết, vui lòng tham khảo [workflow hoàn chỉnh](code/chapter5/HelloAgent_difyCase.yml).

Cấu hình và hiệu quả tạo ảnh được thể hiện trong Hình 5.28 và 5.29.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-13.png" alt="Image description" width="90%"/>
  <p>Hình 5.28 Cài đặt tạo ảnh</p>
</div>
<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-05.png" alt="Image description" width="90%"/>
  <p>Hình 5.29 Trợ lý tạo ảnh</p>
</div>

Hiệu quả tạo sinh video được thể hiện trong Hình 5.30.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-06.png" alt="Image description" width="90%"/>
  <p>Hình 5.30 Trợ lý video</p>

  <p><a href="https://pub-f5ed2046361c4244878e5984bdb564de.r2.dev/9af7c33d-5c82-4b14-8fb3-a4e426e8ee5a.mp4">Nhấp để xem video demo</a></p>
</div>

**Module Truy vấn và phân tích dữ liệu**

Xử lý dữ liệu là một trong những năng lực quan trọng của agent. Module này minh họa cách kết nối cơ sở dữ liệu trong Dify để triển khai truy vấn dữ liệu và phân tích trực quan hóa.

Đầu tiên, cài đặt plugin công cụ truy vấn dữ liệu; case này sử dụng plugin `rookie-text2data`. Chìa khóa của việc truy vấn dữ liệu là cung cấp cho mô hình lớn cấu trúc bảng và thông tin trường rõ ràng để nó có thể tạo ra các câu lệnh truy vấn SQL chính xác. Các cách làm phổ biến bao gồm:

- Trực tiếp cung cấp câu lệnh DDL của bảng dữ liệu
- Cung cấp mô tả về sự tương ứng giữa tên bảng và tên trường

Cấu hình thông tin kết nối cơ sở dữ liệu (địa chỉ IP, tên cơ sở dữ liệu, cổng, tài khoản, mật khẩu, v.v.), như trong Hình 5.31. Kết quả truy vấn cần được tổ chức thông qua một node mô hình lớn và chuyển đổi thành đầu ra ngôn ngữ tự nhiên dễ hiểu.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-22.png" alt="Image description" width="90%"/>
  <p>Hình 5.31 Cấu hình cơ sở dữ liệu</p>
</div>

Cài đặt prompt:

```
# I. Role Setting (Role)
You are a professional data query specialist, skilled at data organization, with clear logical thinking and concise expression ability.

# II. Background
The user has provided raw data queried from the database. This data may have issues such as inconsistent formats, missing fields, and duplicate records, and needs professional organization before effective display.

# III. Task Objectives (Task)
1. Summarize and organize raw data
2. Classify and sort data according to correct logic
3. Data display highlights key information and data insights
4. Provide easy-to-understand data display

# IV. Limitation Prompts (Limit)
1. Must not arbitrarily delete important data
2. Avoid using overly complex or professional statistical terminology
3. Must not tamper with the true values of raw data
4. Avoid displaying too much redundant information, keep it concise and clear
5. Must not leak sensitive data or personal privacy information

# V. Output Format Requirements (Example)
 Data Overview: Simply briefly explain the data content
```

Phần trình bày hiệu quả được thể hiện trong Hình 5.32:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-07.png" alt="Image description" width="90%"/>
  <p>Hình 5.32 Trợ lý truy vấn dữ liệu</p>
</div>

Cài đặt prompt:

```
# I. Role Setting (Role)
You are a professional data analyst with data organization, cleaning, and visualization capabilities, able to extract key information from raw data and transform it into intuitive visual displays.

# II. Background
The user has queried a batch of raw data from the database. This data may contain multiple fields, missing values, or inconsistent formats, and needs to be organized before generating visualization charts.

# III. Task Objectives (Task)
# Workflow
1. Data Analysis
Analyze, organize, and summarize data according to reasonable rules
2. Analysis & Visualization
Generate at least 1 chart (choose one or more from bar / line / pie chart)
Can call tools: "generate_pie_chart" | "generate_column_chart" | "generate_line_chart"

# IV. Limitation Prompts (Limit)
1. Avoid using overly complex chart types, ensure visualization results are easy to understand
2. Do not ignore data quality issues, must perform necessary data cleaning
3. Avoid using too many colors or elements in visualization, keep it concise and clear
4. Do not omit labeling and explanation of key data
5. Must perform summary and chart generation, regardless of data volume

# V. Output Format Requirements (Example)
Please output in the following format:
1. Data overview summary (do not output field names, do not list points, just a short paragraph)
2. Display generated charts
```

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-08.png" alt="Image description" width="90%"/>
  <p>Hình 5.33 Trợ lý phân tích dữ liệu</p>
</div>

Điểm khác biệt duy nhất trong trợ lý phân tích dữ liệu là chúng ta đã thêm các công cụ trực quan hóa dữ liệu, cụ thể là các plugin công cụ tạo biểu đồ BI "generate_pie_chart" | "generate_column_chart" | "generate_line_chart". Nếu bạn đã cài đặt những công cụ này theo yêu cầu trước đó, bạn có thể trực tiếp thêm và sử dụng chúng, đồng thời thêm các mô tả tương ứng như trong prompt ở trên.

**Tích hợp công cụ MCP**

Cuối cùng là ứng dụng tích hợp các công cụ MCP. Chúng ta đã hoàn thành cấu hình MCP ở trên, giờ đây chúng ta sẽ tích hợp nó vào agent. Các bước cấu hình như sau:

1. Chọn một chiến lược agent hỗ trợ gọi MCP
2. Chọn chế độ ReAct
3. Cấu hình dịch vụ MCP (lưu ý xóa tiền tố `mcp-server`, chọn chế độ SSE)
4. Điền các prompt tương ứng

Giao diện cấu hình được thể hiện trong Hình 5.34.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-23.png" alt="Image description" width="90%"/>
  <p>Hình 5.34 Cấu hình MCP cho Agent</p>
</div>

Hiệu quả của trợ lý Amap, trợ lý ăn uống và trợ lý tin tức lần lượt được thể hiện trong Hình 5.35, 5.36 và 5.37.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-09.png" alt="Image description" width="90%"/>
  <p>Hình 5.35 Trợ lý Amap</p>
</div>

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-10.png" alt="Image description" width="90%"/>
  <p>Hình 5.36 Trợ lý ăn uống</p>
</div>

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/dify-11.png" alt="Image description" width="90%"/>
  <p>Hình 5.37 Trợ lý tin tức</p>
</div>

Đến đây, chúng ta đã hoàn thành một trợ lý cá nhân super agent đầy đủ chức năng. Trợ lý này bao phủ nhiều khía cạnh của cuộc sống: khi bạn cần quần áo mới, bạn có thể để Doubao tạo thiết kế; trước khi ra ngoài, bạn có thể để trợ lý Amap lập kế hoạch tuyến đường; khi không biết ăn gì, bạn có thể nhận gợi ý ăn uống; khi muốn tìm hiểu tình hình học tập, bạn có thể thực hiện phân tích dữ liệu. Agent này có thể xử lý nhiều tác vụ công việc và cuộc sống khác nhau, và chúng tôi mong được thấy mọi người xây dựng nhiều trợ lý agent cá nhân sáng tạo hơn.

### 5.3.3 Phân tích ưu điểm và hạn chế của Dify

Với tư cách là một nền tảng phát triển ứng dụng AI hàng đầu, Dify thể hiện những ưu thế đáng kể ở nhiều khía cạnh:

1. Ưu điểm cốt lõi

- Trải nghiệm phát triển Full-Stack: Dify tích hợp các RAG pipeline, AI workflow, quản lý mô hình và các chức năng khác vào một nền tảng, cung cấp trải nghiệm phát triển trọn gói
- Cân bằng giữa Low-Code và khả năng mở rộng cao: Dify đạt được sự cân bằng tốt giữa sự tiện lợi của phát triển low-code và tính linh hoạt của phát triển chuyên nghiệp

- Bảo mật và tuân thủ cấp doanh nghiệp: Dify cung cấp mã hóa AES-256, kiểm soát quyền RBAC và nhật ký kiểm toán, đáp ứng các yêu cầu nghiêm ngặt về bảo mật và tuân thủ

- Năng lực tích hợp công cụ phong phú: Dify hỗ trợ hơn 9000 công cụ và tiện ích mở rộng API, cung cấp khả năng mở rộng chức năng rộng rãi
- Cộng đồng mã nguồn mở năng động: Dify có một cộng đồng mã nguồn mở năng động, cung cấp tài nguyên học tập và hỗ trợ phong phú



2. Hạn chế chính
- Đường cong học tập dốc: Đối với những người dùng hoàn toàn không có nền tảng kỹ thuật, vẫn còn một đường cong học tập nhất định

- Nút thắt hiệu năng: Có thể đối mặt với các thách thức về hiệu năng trong các tình huống đồng thời cao, cần được tối ưu hóa phù hợp. Các thành phần cốt lõi phía máy chủ của hệ thống Dify được triển khai bằng Python, có hiệu năng tương đối kém so với các ngôn ngữ như C++, Golang và Rust

- Hỗ trợ đa phương thức chưa đầy đủ: Hiện tại chủ yếu tập trung vào xử lý văn bản, với hỗ trợ hạn chế cho hình ảnh, video, HTML, v.v.

- Chi phí phiên bản doanh nghiệp cao: Giá phiên bản doanh nghiệp của Dify tương đối cao, có thể vượt quá ngân sách của các đội nhỏ

- Vấn đề tương thích API: Định dạng API của Dify không tương thích với OpenAI, điều này có thể hạn chế việc tích hợp với một số hệ thống bên thứ ba


## 5.4 Nền tảng Ba: FastGPT

### 5.4.1 Giới thiệu FastGPT và các tính năng cốt lõi

FastGPT là một nền tảng hỏi đáp knowledge base và công cụ xây dựng Agent mã nguồn mở dựa trên LLM. Định vị cốt lõi của nó là "Engine năng suất AI cấp doanh nghiệp" (Enterprise-grade AI Productivity Engine), tập trung vào việc cung cấp các giải pháp RAG (Retrieval-Augmented Generation) dễ sử dụng và năng lực điều phối workflow trực quan. Không giống như trải nghiệm zero-code của Coze và năng lực phát triển full-stack của Dify, FastGPT coi hỏi đáp knowledge base là công dân hạng nhất, tối ưu hóa sâu xoay quanh chuỗi hoàn chỉnh "nhập dữ liệu — chia nhỏ thông minh — truy xuất vector — tạo sinh hội thoại".

Khi bạn truy cập trang web chính thức của FastGPT, điều đầu tiên bạn thấy là tuyên ngôn sản phẩm súc tích và mạnh mẽ của nó — "Enterprise-grade AI Productivity Engine", nhấn mạnh việc xây dựng các AI Agent cấp doanh nghiệp an toàn, có thể kiểm soát, như trong Hình 5.38.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-01.png" alt="Image description" width="90%"/>
  <p>Hình 5.38 Trang chủ website chính thức của FastGPT</p>
</div>

Sau khi đăng nhập vào nền tảng, bạn có thể thấy bố cục không gian làm việc rõ ràng của nó. Thanh điều hướng bên trái chia các chức năng cốt lõi thành bốn module: Cổng hội thoại (Dialog Portal), Không gian làm việc (Workspace), Knowledge Base và Tài khoản (Account). Trong đó, module Agent được chia thành ba loại: Workflow, Dialog Agent và Dialog Agent V2 (Beta), giúp người dùng thuận tiện chọn chế độ xây dựng phù hợp dựa trên tình huống nghiệp vụ của họ. Khu vực chính cung cấp một điểm truy cập nhanh "Tạo từ Template" (Create from Template), với các template chính thức tích hợp sẵn như Sales Training Master, Document Translation Assistant và Industry Trend Insight Briefing; bên dưới là danh sách Agent của riêng người dùng, như trong Hình 5.39.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-02.png" alt="Image description" width="90%"/>
  <p>Hình 5.39 Giao diện chính của nền tảng FastGPT</p>
</div>

Về các tùy chọn tài khoản và gói dịch vụ, FastGPT cung cấp một phiên bản miễn phí để các nhà phát triển cá nhân dùng thử. Phiên bản miễn phí bao gồm 100 credit, 600 chỉ mục knowledge base, 1 thành viên nhóm, 10 Agent, 3 knowledge base, lưu giữ bản ghi hội thoại trong 30 ngày, tốc độ gọi 30 QPM, và khả năng tải lên 5 file mỗi file 50MB cùng một lúc, như trong Hình 5.40. Đối với các doanh nghiệp và đội nhóm vừa và nhỏ, nền tảng cũng cung cấp các gói nâng cấp trả phí để đáp ứng nhu cầu đồng thời và lưu trữ cao hơn.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-03.png" alt="Image description" width="90%"/>
  <p>Hình 5.40 Gói miễn phí và mức sử dụng của FastGPT</p>
</div>

Năng lực cạnh tranh cốt lõi của FastGPT nằm ở năng lực knowledge base mạnh mẽ của nó. Nền tảng hỗ trợ nhập nhiều định dạng file, bao gồm các loại tài liệu phổ biến như Word, Markdown và PDF. Như trong Hình 5.41, trong "test General Knowledge Base", chúng ta có thể tải lên nhiều file như Introduction to Deep Learning, Getting Started with Machine Learning và Bidding Document Text. Hệ thống tự động chia nhỏ và lập chỉ mục các file, và khi trạng thái hiển thị "Ready", chúng có thể được truy xuất và tham chiếu trong các cuộc hội thoại.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-12.png" alt="Image description" width="90%"/>
  <p>Hình 5.41 Quản lý file Knowledge Base của FastGPT</p>
</div>

Ở cấp độ xử lý file, FastGPT cung cấp cấu hình tham số chi tiết. Như trong Hình 5.42, người dùng có thể chọn giữa phương thức xử lý "Chunk Storage" (Lưu trữ theo khối) hoặc "Q&A Pair Extraction" (Trích xuất cặp hỏi đáp), thiết lập các điều kiện chia nhỏ (như kích hoạt chia nhỏ khi độ dài văn bản gốc vượt quá 1000 ký tự), và bật nhiều tùy chọn tăng cường chỉ mục, bao gồm thêm tiêu đề vào chỉ mục, tự động tạo chỉ mục bổ sung và lập chỉ mục hình ảnh tự động. Đối với các tài liệu có nội dung văn bản và hình ảnh trộn lẫn phong phú (như sách giáo khoa và báo cáo nghiên cứu), tính năng lập chỉ mục hình ảnh tự động đặc biệt quan trọng, vì nó cho phép mô hình lớn hiểu và tham chiếu thông tin trực quan trong tài liệu khi trả lời.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-14.png" alt="Image description" width="90%"/>
  <p>Hình 5.42 Cài đặt tham số xử lý dữ liệu Knowledge Base</p>
</div>

Sau khi tải lên, người dùng có thể xem nội dung cụ thể của các khối file. Như trong Hình 5.43, lấy "English Grade 4 Lower Semester Full Electronic Book.pdf" làm ví dụ, nền tảng hiển thị bản xem trước văn bản của từng khối, trong khi bảng metadata bên phải hiển thị các thông tin quan trọng như kích thước file (62MB), độ dài văn bản gốc (37.797 ký tự), chế độ xử lý (lưu trữ theo khối) và trạng thái lập chỉ mục hình ảnh. Việc hiển thị khối minh bạch này tạo thuận lợi cho các nhà phát triển trong việc gỡ lỗi và tối ưu hóa knowledge base.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-13.png" alt="Image description" width="90%"/>
  <p>Hình 5.43 Chi tiết khối file Knowledge Base và metadata</p>
</div>

Ngoài knowledge base, FastGPT cũng bắt kịp các xu hướng hệ sinh thái trong việc tích hợp công cụ. Nền tảng hỗ trợ native (nguyên bản) các công cụ MCP (Model Context Protocol), và người dùng có thể quản lý thống nhất nhiều dịch vụ MCP khác nhau trong module "My Tools" (Công cụ của tôi). Như trong Hình 5.44, dưới thư mục "ai Finance", chúng ta đã cấu hình nhiều công cụ MCP bao gồm Chinese Trend Aggregation, Real-time Stock MCP, QieMan Fund MCP, Minimax-MCP và BI Chart Tool. Những công cụ này sẽ trao cho agent năng lực gọi dữ liệu thời gian thực bên ngoài và các dịch vụ chuyên nghiệp.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-04.png" alt="Image description" width="90%"/>
  <p>Hình 5.44 Quản lý công cụ MCP của FastGPT</p>
</div>

### 5.4.2 Xây dựng "Trợ lý cố vấn đầu tư thông minh"

**Mô tả tình huống:** Case này sẽ sử dụng nền tảng FastGPT, kết hợp knowledge base, công cụ MCP và điều phối workflow, để xây dựng một "Trợ lý cố vấn đầu tư thông minh" chuyên nghiệp. Trợ lý này có thể trả lời các câu hỏi về lý thuyết đầu tư tài chính, truy vấn báo giá cổ phiếu thời gian thực, tiến hành đánh giá hồ sơ rủi ro của người dùng, và tạo ra các báo cáo phân tích chiến lược đầu tư được cá nhân hóa. Thông qua case này, bạn sẽ nắm vững mô hình phát triển cốt lõi của FastGPT: xây dựng knowledge base, tích hợp công cụ MCP, điều phối workflow trực quan và thiết kế tương tác hội thoại nhiều lượt.

**Bước 1: Cấu hình các công cụ MCP**

Một trong những năng lực cốt lõi của trợ lý cố vấn đầu tư thông minh là lấy dữ liệu tài chính thời gian thực. Trong FastGPT, chúng ta đạt được điều này bằng cách kết nối các công cụ MCP.

Case này cần hai loại dịch vụ MCP sau:

1. **Truy vấn báo giá cổ phiếu thời gian thực**: Dùng để lấy giá thời gian thực của từng cổ phiếu, biến động giá, khối lượng giao dịch và các dữ liệu khác.
2. **Dữ liệu tài chính và tạo biểu đồ**: Dùng để lấy dữ liệu tài chính vĩ mô và tạo các biểu đồ trực quan.

Như trong Hình 5.45, chúng ta có thể tìm thấy "Visual Chart MCP Server" trong chợ MCP của cộng đồng ModelScope. Dịch vụ này được phát triển dựa trên TypeScript, tương thích với giao thức MCP, và cung cấp năng lực tạo biểu đồ vùng, biểu đồ cột, biểu đồ tròn và nhiều loại biểu đồ khác, biến đổi dữ liệu khô khan thành các kết quả trực quan trực giác.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-05.png" alt="Image description" width="90%"/>
  <p>Hình 5.45 Visual Chart MCP Server của cộng đồng ModelScope</p>
</div>

Ngoài ra, như trong Hình 5.46, nền tảng Alibaba Cloud Bailian cũng cung cấp các dịch vụ MCP chính thức phong phú. Trong trang quản lý MCP, chúng ta có thể tìm thấy các dịch vụ MCP tài chính như "Today's Investment - Financial..." và "QieMan", cũng như các công cụ như truy vấn báo giá cổ phiếu thời gian thực và Wanxiang - Video Generation. Sau khi thêm các dịch vụ này vào thư viện công cụ MCP của FastGPT, agent có thể gọi chúng theo nhu cầu trong các cuộc hội thoại.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-06.png" alt="Image description" width="90%"/>
  <p>Hình 5.46 Quản lý MCP của Alibaba Cloud Bailian</p>
</div>

Trong giao diện cấu hình công cụ MCP của FastGPT, sau khi điền các địa chỉ dịch vụ và thông tin xác thực tương ứng, việc tích hợp công cụ hoàn tất. Mỗi công cụ MCP có thể được cấu hình với mô tả độc lập và các tham số gọi, giúp agent dễ hiểu mục đích của từng công cụ khi ra quyết định.

**Bước 2: Thiết kế workflow cố vấn đầu tư thông minh**

Sau khi hoàn thành cấu hình công cụ, bước vào giai đoạn điều phối workflow cốt lõi. FastGPT cung cấp giao diện điều phối Flow trực quan, nơi người dùng có thể xây dựng các luồng hội thoại phức tạp bằng cách kéo node và kết nối các cạnh.

Như trong Hình 5.47, workflow hoàn chỉnh của "Trợ lý cố vấn đầu tư thông minh" bao gồm nhiều nhánh xử lý: nhận diện ý định người dùng, truy xuất knowledge base, thu thập bảng câu hỏi rủi ro, gọi công cụ MCP và tạo báo cáo. Toàn bộ workflow trình bày một cấu trúc mô-đun rõ ràng với dữ liệu chảy có trật tự giữa các node khác nhau. Cách tiếp cận điều phối trực quan này cho phép các nhà phát triển hiểu và gỡ lỗi trực quan các đường dẫn quyết định của agent.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-07.png" alt="Image description" width="90%"/>
  <p>Hình 5.47 Điều phối workflow Trợ lý cố vấn đầu tư thông minh</p>
</div>

Logic cốt lõi của workflow như sau:

1. **Node nhận diện ý định**: Đầu tiên xác định loại đầu vào của người dùng. Nếu là truy vấn khái niệm tài chính, định tuyến đến nhánh truy xuất knowledge base; nếu là truy vấn cổ phiếu, định tuyến đến nhánh gọi công cụ MCP; nếu là chẩn đoán đầu tư, đi vào quy trình thu thập bảng câu hỏi rủi ro.
2. **Chuyên gia giáo dục kiến thức đầu tư**: Kết nối với một knowledge base tài chính đã dựng sẵn để truy xuất các lý thuyết đầu tư, giải thích khái niệm và nghiên cứu tình huống liên quan.
3. **Chuyên gia phân tích đánh giá rủi ro**: Hướng dẫn người dùng qua một bảng câu hỏi đánh giá rủi ro thông qua các node nhập form, bao gồm các chiều như độ tuổi, kinh nghiệm đầu tư, mức thu nhập hằng tháng, khả năng chịu đựng rủi ro và mục tiêu đầu tư. Dữ liệu sau đó được chuyển đến các node mô hình lớn tiếp theo để phân tích môi trường thị trường, phân tích cơ bản và tâm lý tin tức, tích hợp hồ sơ người dùng, dữ liệu thị trường và thông tin tin tức để gọi mô hình lớn tạo ra một báo cáo phân tích chiến lược đầu tư có cấu trúc.
4. **Chuyên gia tình báo tin tức thị trường**: Gọi MCP cổ phiếu thời gian thực hoặc MCP tạo biểu đồ dựa trên nhu cầu người dùng để lấy dữ liệu bên ngoài.
5. **Chuyên gia tư vấn tổng quát**: Phản hồi các truy vấn đơn giản của người dùng như "xin chào", "bạn khỏe không", v.v.

**Bước 3: Cấu hình prompt và knowledge base**

Trong FastGPT, việc cấu hình System Prompt cũng quan trọng không kém. Đối với trợ lý cố vấn đầu tư thông minh, chúng ta cần thiết lập một vai trò cố vấn đầu tư tài chính chuyên nghiệp:

```
# I. Role Setting (Role)
You are a professional financial investment advisor with rich experience in risk assessment and portfolio management. Your professional background includes finance, behavioral finance, and investment psychology, enabling you to analyze users' risk tolerance from multiple dimensions. Your tone should be professional, neutral, and easy to understand, avoiding overly complex financial jargon to ensure that ordinary investors can easily understand your analysis.

# II. Background
Users will provide the following information: age, investment experience, monthly income level, maximum tolerable loss, and investment goals. This information is the foundation for risk assessment. You need to analyze this data based on general financial principles (such as lifecycle theory and risk-return matching principles). The scenario limitation is that you cannot access other personal information about the user (such as total assets, family status), so the analysis should focus on the provided information and avoid excessive speculation.

# III. Task Objectives (Task)
Based on the user's provided age, investment experience, monthly income level, maximum tolerable loss, and investment goals, conduct a comprehensive risk assessment.
Output a clear risk level assessment result (e.g., conservative, moderate, aggressive, etc.), and briefly explain the reasoning.
Ensure the analysis logic is coherent and easy for users to understand and apply.

# IV. Limitation Prompts (Limit)
Avoid providing specific financial product recommendations (such as specific stock or fund names); only discuss general asset classes.
Do not make any guaranteed return promises or predictions; emphasize that investment carries risks.
Avoid using overly technical or specialized language to ensure output is friendly to non-professional investors.
Do not expand user information based on assumptions or speculation; only analyze using the provided age, investment experience, monthly income, maximum loss tolerance, and investment goals.
Output must not contain any discriminatory, biased, or strongly subjective statements; maintain objectivity and neutrality.

# V. Output Format Requirements (Example)
Output should be organized according to the following structure:
**Risk Level Assessment**: [e.g., Moderate]
**Assessment Reasoning**: Briefly explain the analysis based on user's age, investment experience, income, loss tolerance, and investment goals.
**Risk Reminder**: Reiterate investment risks and encourage users to adjust based on their own circumstances.
```

Đồng thời, chúng ta cần cấu hình một knowledge base tài chính cho trợ lý. Tải lên knowledge base các tài liệu về nền tảng đầu tư, phân tích báo cáo tài chính và diễn giải các chỉ số kinh tế vĩ mô, và hoàn thành việc chia nhỏ và lập chỉ mục theo quy trình được thể hiện trong Hình 5.41~5.43. Như vậy, khi người dùng hỏi các câu hỏi mang tính khái niệm như "Sự khác biệt giữa hệ số P/E và P/B là gì", agent có thể truy xuất các định nghĩa chính xác và phân tích so sánh từ knowledge base thay vì hoàn toàn dựa vào kiến thức tiền huấn luyện của mô hình lớn, từ đó giảm thiểu hiệu quả nguy cơ ảo giác (hallucination).

**Bước 4: Kiểm thử và trình diễn hiệu quả**

Sau khi hoàn thành cấu hình workflow và prompt, chúng ta có thể kiểm thử trong giao diện hội thoại của FastGPT. Như trong Hình 5.48, lời chào mở đầu của trợ lý cố vấn đầu tư thông minh giới thiệu rõ ràng ba tính năng chính của nó: nắm vững lý thuyết đầu tư tài chính, tin tức và dữ liệu thị trường thời gian thực, và các khuyến nghị phân bổ tài sản dựa trên đánh giá hồ sơ rủi ro. Giao diện cũng cung cấp các nút thao tác nhanh để người dùng kích hoạt các tác vụ phổ biến chỉ với một chạm.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-08.png" alt="Image description" width="50%"/>
  <p>Hình 5.48 Giao diện hội thoại Trợ lý cố vấn đầu tư thông minh</p>
</div>

Khi người dùng nhấp vào "Tiến hành đánh giá tài sản để nhận lời khuyên đầu tư", trợ lý lần lượt mở ra bảng câu hỏi đánh giá rủi ro, thu thập thông tin về độ tuổi, kinh nghiệm đầu tư, mức thu nhập hằng tháng, mức tổn thất tối đa có thể chịu đựng và mục tiêu đầu tư của người dùng. Dựa trên thông tin này, trợ lý tạo ra một báo cáo phân tích chiến lược đầu tư hoàn chỉnh.

Như trong Hình 5.49, báo cáo chứa các module cốt lõi sau:

- **Đánh giá rủi ro người dùng**: Dựa trên kết quả bảng câu hỏi, phân tích mức độ chịu đựng rủi ro của người dùng (ví dụ: mức độ vừa phải).
- **Khuyến nghị tỉ lệ phân bổ tài sản**: Trình bày tỉ lệ phân bổ của các lớp tài sản chính như cổ phiếu, trái phiếu và tiền mặt dưới dạng biểu đồ tròn trực quan (ví dụ: cổ phiếu 45%, trái phiếu 40%, tiền mặt 15%).
- **Phân tích cơ bản thị trường**: Cung cấp nhận định thị trường dựa trên môi trường kinh tế vĩ mô hiện tại và các xu hướng ngành.
- **Chiến lược tái cân bằng**: Cung cấp các khuyến nghị về tái cân bằng danh mục định kỳ, bao gồm chu kỳ tái cân bằng và các điều kiện kích hoạt.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-09.png" alt="Image description" width="90%"/>
  <p>Hình 5.49 Báo cáo phân tích chiến lược đầu tư</p>
</div>

Đối với các tình huống truy vấn dữ liệu thời gian thực, như trong Hình 5.50, khi người dùng hỏi "Truy vấn thông tin giá cổ phiếu hiện tại của Kweichow Moutai", agent tự động gọi công cụ MCP (`get_stock_quote_realtime`) để lấy dữ liệu thị trường thời gian thực. Kết quả trả về bao gồm tiêu đề, nguồn dữ liệu, các điểm nổi bật chính (giá mở cửa, giá cao nhất, khoảng giá trong phiên, khối lượng giao dịch, tổng vốn hóa thị trường, vốn hóa lưu hành, v.v.), cũng như phân tích tác động tiềm năng và các hành động được đề xuất. Đầu ra có cấu trúc, chuyên nghiệp này phản ánh giá trị thực tiễn của năng lực gọi công cụ của Agent.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-10.png" alt="Image description" width="50%"/>
  <p>Hình 5.50 Truy vấn báo giá cổ phiếu thời gian thực</p>
</div>

Về mặt giải thích khái niệm, như trong Hình 5.51, khi người dùng hỏi "Sự khác biệt giữa hệ số P/E và P/B là gì", trợ lý cung cấp một phân tích so sánh có hệ thống dựa trên knowledge base và sự hiểu biết của mô hình lớn: bắt đầu từ định nghĩa, nó giải thích chi tiết các phương pháp tính toán của P/E Ratio và P/B Ratio; so sánh chúng từ bốn chiều (cơ sở tính toán, ngành áp dụng, thông tin phản ánh, hạn chế); và cuối cùng đưa ra lời khuyên ứng dụng thực tế về khi nào nên tập trung vào hệ số P/E so với hệ số P/B. Đầu ra có cấu trúc tốt, logic chặt chẽ này là một ưu thế điển hình của mô hình lớn được tăng cường bởi RAG trong hỏi đáp lĩnh vực chuyên biệt.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/fastgpt-11.png" alt="Image description" width="50%"/>
  <p>Hình 5.51 Phân tích khái niệm hệ số P/E và P/B</p>
</div>

### 5.4.3 Phân tích ưu điểm và hạn chế của FastGPT

Thông qua thực hành xây dựng "Trợ lý cố vấn đầu tư thông minh" ở trên, chúng ta có thể hình thành một hiểu biết toàn diện về nền tảng FastGPT.

**Ưu điểm:**

- **Trải nghiệm Knowledge Base tối ưu:** Ưu thế cốt lõi của FastGPT nằm ở sự tinh chỉnh sâu sắc của RAG pipeline. Từ tải file, chia nhỏ thông minh, tăng cường chỉ mục, nhận diện hình ảnh đến truy xuất và thu hồi (recall), mỗi khâu đều cung cấp các tùy chọn cấu hình chi tiết và một giao diện gỡ lỗi minh bạch. Đối với các tình huống cần xây dựng hệ thống hỏi đáp dựa trên knowledge base riêng tư (như trợ lý tri thức doanh nghiệp, dịch vụ khách hàng thông minh và tư vấn lĩnh vực chuyên nghiệp), FastGPT cung cấp một trải nghiệm out-of-the-box tuyệt vời.
- **Hỗ trợ MCP native:** Không giống Coze, FastGPT hỗ trợ nguyên bản giao thức MCP, cho phép tích hợp liền mạch với một số lượng lớn dịch vụ MCP từ các hệ sinh thái như ModelScope và Alibaba Cloud Bailian. Điều này có nghĩa là khả năng mở rộng công cụ của agent không còn bị giới hạn ở thư viện plugin tích hợp sẵn của nền tảng; các nhà phát triển có thể tự do tích hợp bất kỳ công cụ bên thứ ba nào tuân thủ các tiêu chuẩn MCP.
- **Thiết kế trung lập về mô hình:** FastGPT hỗ trợ tích hợp linh hoạt với nhiều mô hình lớn chủ đạo trong nước và quốc tế như OpenAI, Claude, Tongyi Qianwen và DeepSeek. Người dùng có thể tự do chuyển đổi mô hình nền tảng dựa trên nhu cầu nghiệp vụ và cân nhắc chi phí, tránh nguy cơ bị ràng buộc vào một nhà cung cấp mô hình duy nhất.
- **Điều phối workflow trực quan:** Module Flow cung cấp một giao diện điều phối dựa trên node trực quan, nơi logic đa nhánh phức tạp (như nhận diện ý định, thu thập bảng câu hỏi và tạo báo cáo trong case này) có thể được xây dựng nhanh chóng thông qua kéo-thả, hạ thấp rào cản cho những người không phải là nhà phát triển.

**Hạn chế:**

- **Hệ sinh thái template tương đối yếu:** So với cửa hàng plugin phong phú của Coze và Marketplace của Dify với hơn 8.000 plugin, số lượng template chính thức và công cụ tích hợp sẵn của FastGPT tương đối hạn chế. Mặc dù giao thức MCP bù đắp một phần cho khuyết điểm này, việc tìm kiếm và cấu hình các dịch vụ MCP phù hợp vẫn đặt ra một ngưỡng nhất định cho người dùng phi kỹ thuật.
- **Hạn mức phiên bản miễn phí eo hẹp:** Phiên bản miễn phí chỉ cung cấp 100 credit và tốc độ gọi 30 QPM, cạn kiệt nhanh chóng đối với các nhà phát triển cần kiểm thử và lặp lại thường xuyên. Các giới hạn về số lượng chỉ mục knowledge base và số lượng thành viên nhóm cũng khiến phiên bản miễn phí khó hỗ trợ sự hợp tác nhóm ngay cả ở quy mô vừa phải.
- **Cộng đồng và tài liệu vẫn đang hoàn thiện:** Là một dự án mã nguồn mở tương đối trẻ, mức độ năng động của cộng đồng và tính đầy đủ của tài liệu tiếng Anh của FastGPT vẫn tụt hậu so với các nền tảng trưởng thành như Dify và n8n. Khi gặp các trường hợp biên (edge case), bạn có thể cần đào sâu vào mã nguồn hoặc tìm sự trợ giúp từ cộng đồng.

Nhìn chung, FastGPT là một nền tảng có tính cạnh tranh cao trong lĩnh vực hỏi đáp knowledge base. Nếu nhu cầu cốt lõi của bạn là xây dựng một hệ thống hỏi đáp thông minh dựa trên tài liệu riêng tư và bạn muốn kiểm soát chi tiết toàn bộ RAG pipeline, FastGPT là một lựa chọn đáng thử. Đối với các tình huống cần một hệ sinh thái plugin mạnh mẽ hoặc tự động hóa quy trình nghiệp vụ phức tạp, bạn có thể bổ trợ nó bằng Dify hoặc n8n.


## 5.5 Nền tảng Bốn: n8n

Như chúng ta đã giới thiệu trước đó, bản sắc cốt lõi của n8n là một nền tảng tự động hóa workflow tổng quát, không phải là một công cụ xây dựng ứng dụng LLM thuần túy. Hiểu được điều này là chìa khóa để làm chủ n8n. Khi sử dụng n8n để xây dựng các ứng dụng thông minh, thực chất chúng ta đang thiết kế một quy trình tự động hóa lớn hơn, và mô hình ngôn ngữ lớn chỉ là một (hoặc nhiều) "node xử lý" mạnh mẽ trong quy trình này.

### 5.5.1 Node và Workflow của n8n

Thế giới của n8n được cấu thành từ hai khái niệm cơ bản nhất: **Node** và **Workflow**.

- **Node**: Node là đơn vị nhỏ nhất thực hiện các thao tác cụ thể trong một workflow. Bạn có thể coi nó như một "khối xây dựng" với các chức năng cụ thể. n8n cung cấp hàng trăm node dựng sẵn bao phủ nhiều thao tác phổ biến khác nhau, từ gửi email, đọc ghi cơ sở dữ liệu, gọi API đến xử lý file. Mỗi node đều có đầu vào và đầu ra, và cung cấp một giao diện cấu hình trực quan. Các node có thể được chia thành hai loại:
  - **Trigger Node (Node kích hoạt)**: Đây là điểm khởi đầu của toàn bộ workflow, chịu trách nhiệm khởi động quy trình. Ví dụ, "khi nhận được một email Gmail mới", "được kích hoạt mỗi giờ một lần", hoặc "khi nhận được một yêu cầu Webhook". Một workflow phải có một và chỉ một node kích hoạt.
  - **Regular Node (Node thông thường)**: Chịu trách nhiệm xử lý dữ liệu và logic cụ thể. Ví dụ, "đọc bảng tính Google Sheets", "gọi mô hình OpenAI", hoặc "chèn một bản ghi vào cơ sở dữ liệu".
- **Workflow**: Workflow là một lưu đồ tự động hóa được cấu thành từ nhiều node được kết nối với nhau. Nó định nghĩa đường dẫn hoàn chỉnh về cách dữ liệu bắt đầu từ node kích hoạt, đi qua từng bước giữa các node khác nhau, được xử lý, và cuối cùng hoàn thành tác vụ đã định. Dữ liệu được truyền giữa các node ở định dạng JSON có cấu trúc, cho phép chúng ta kiểm soát chính xác đầu vào và đầu ra của từng khâu.


Sức mạnh thực sự của n8n nằm ở năng lực "kết nối" mạnh mẽ của nó. Nó có thể liên kết các ứng dụng và dịch vụ vốn tách biệt (như CRM nội bộ của công ty, các nền tảng mạng xã hội bên ngoài, cơ sở dữ liệu của bạn và các mô hình ngôn ngữ lớn) để đạt được tự động hóa quy trình nghiệp vụ đầu-cuối mà trước đây đòi hỏi phải lập trình phức tạp. Trong phần thực hành sắp tới, chúng ta sẽ tự mình trải nghiệm cách sử dụng hệ thống node và workflow này để xây dựng một ứng dụng tự động hóa được tích hợp năng lực AI.

### 5.5.2 Xây dựng trợ lý email thông minh

Về việc cấu hình môi trường và cách sử dụng cơ bản nhất của n8n, tài liệu đã được tạo trong thư mục `Additional-Chapter` của dự án, nên chúng ta sẽ không giới thiệu quá nhiều ở đây. Trong phần trước, chúng ta đã tìm hiểu các khái niệm cơ bản của n8n. Case này sẽ minh họa rõ ràng sự khác biệt cốt lõi giữa AI Agent hiện đại và workflow tự động hóa truyền thống. Các quy trình truyền thống mang tính tuyến tính, trong khi Agent mà chúng ta sắp xây dựng sẽ có thể nhận email của người dùng, "tư duy" thông qua một **node AI Agent** cốt lõi, tự chủ hiểu ý định người dùng, ra quyết định và lựa chọn trong số nhiều "công cụ" khả dụng, và cuối cùng tự động tạo và gửi các phản hồi có độ liên quan cao.

Toàn bộ quy trình mô phỏng một logic ra quyết định cao cấp hơn: `Nhận -> AI Agent (Tư duy -> Quyết định -> Gọi công cụ) -> Trả lời`, như trong Hình 5.52.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-01.png" alt="Image description" width="90%"/>
  <p>Hình 5.52 Sơ đồ kiến trúc Agent email thông minh tích hợp</p>
</div>

Không giống phương pháp truyền thống chia công cụ thành nhiều sub-workflow, node `AI Agent` của n8n cho phép chúng ta tích hợp các thành phần như mô hình ngôn ngữ lớn (LLM), bộ nhớ (memory) và công cụ (tools) trong một giao diện thống nhất, đơn giản hóa đáng kể quá trình xây dựng.

Toàn bộ quá trình xây dựng được chia thành hai bước cốt lõi:

1. **Chuẩn bị "bộ nhớ" cho Agent**: Tạo một quy trình độc lập để nạp một knowledge base riêng tư cho Agent.
2. **Xây dựng thân chính của Agent**: Tạo workflow chính để nhận email, tư duy và trả lời.

### 5.5.3 Xây dựng Knowledge Base riêng tư cho Agent

Để Agent có thể trả lời các câu hỏi về những lĩnh vực cụ thể (như thông tin cá nhân của bạn hoặc tài liệu dự án), trước tiên chúng ta cần chuẩn bị cho nó một "bộ não bên ngoài", tức là một knowledge base vector.

Trong n8n, chúng ta có thể sử dụng node `Simple Vector Store` để nhanh chóng xây dựng một knowledge base trong bộ nhớ (in-memory). Quá trình chuẩn bị này thường chỉ cần chạy một lần khi cập nhật tri thức.

**(1) Định nghĩa nguồn tri thức**

Đầu tiên, chúng ta sử dụng node `Code` để lưu trữ văn bản tri thức thô của mình. Đây là một cách đơn giản và nhanh chóng; trong các dự án thực tế, dữ liệu cũng có thể đến từ file, cơ sở dữ liệu, v.v.

- **Node**: `Code`
- **Nội dung**: Viết tri thức của bạn ở định dạng JSON.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-02.png" alt="Screenshot of knowledge base JSON text filled in Code node" width="90%"/>
  <p>Hình 5.53 Định nghĩa nguồn tri thức trong node Code</p>
</div>

```javascript
return [
  {
    "doc_id": "work-schedule-001",
    "content": "My working hours are Monday to Friday, 9 AM to 5 PM. The timezone is Australian Eastern Standard Time (AEST)."
  },
  {
    "doc_id": "off-hours-policy-001",
    "content": "During non-working hours (including weekends and public holidays), I cannot reply to emails immediately."
  },
  {
    "doc_id": "auto-reply-instruction-001",
    "content": "If an email is received during non-working hours, the AI assistant should inform the sender that the email has been received and I will process and reply as soon as possible between 9 AM and 5 PM on the next working day."
  }
];
```

**(2) Vector hóa văn bản (Embeddings)**

Máy tính không thể trực tiếp hiểu văn bản và cần chuyển đổi nó thành vector. Chúng ta sử dụng node `Embeddings` để hoàn thành công việc "phiên dịch" này.

- **Node**: `Embeddings Google Gemini`, chọn mô hình là `gemini-embedding-exp-03-07`. Ở đây chúng ta sử dụng Google API để minh họa; nếu bạn không biết cách lấy Google API, bạn có thể tham khảo tài liệu chính thức.
- **Cấu hình**: Kết nối nó sau node `Code`, và nó sẽ tự động chuyển đổi văn bản được truyền từ thượng nguồn thành dữ liệu vector.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-03.png" alt="" width="90%"/>
  <p>Hình 5.54 Vector hóa dữ liệu trong Code</p>
</div>

**(3) Lưu vào Vector Storage**

Cuối cùng, chúng ta lưu tri thức đã được vector hóa vào một cơ sở dữ liệu trong bộ nhớ, như trong Hình 5.55.

- **Node**: `Simple Vector Store`
- **Cấu hình**:
  - **Chế độ vận hành (Operation Mode)**: `Insert Documents` (chế độ ghi).
  - **Memory Key**: Đặt cho knowledge base này một tên duy nhất, ví dụ `my-dailytime`. Key này tương đương với "tên bảng" của cơ sở dữ liệu, và Agent sẽ sử dụng nó để tìm thông tin sau này.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-04.png" alt="" width="90%"/>
  <p>Hình 5.55 Lưu dữ liệu từ Code vào Vector Storage</p>
</div>

Sau khi hoàn thành cấu hình, **thực thi thủ công quy trình này một lần**. Sau khi thành công, tri thức riêng tư của bạn được nạp vào bộ nhớ của n8n, như trong Hình 5.56.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-05.png" alt="" width="90%"/>
  <p>Hình 5.56 Workflow nạp Knowledge Base hoàn chỉnh</p>
</div>

### 5.5.4 Tạo Workflow chính của Agent

Với các công cụ đã sẵn sàng, giờ đây chúng ta bắt đầu xây dựng quy trình chính của Agent. Nó sẽ chịu trách nhiệm nhận email, tư duy và ra quyết định, gọi các công cụ mà chúng ta vừa tạo vào đúng thời điểm, và cuối cùng thực thi việc trả lời email.

(1) Cấu hình Gmail Trigger

Tạo một workflow mới có tên `Agent: Customer Support`. Sử dụng node `Gmail` làm trigger, thiết lập **Event** của nó thành `Message Received`, và cấu hình tài khoản email của bạn. Bằng cách này, mỗi khi một email mới vào hộp thư đến, workflow sẽ tự động được kích hoạt, như trong Hình 5.57.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-06.png" alt="" width="90%"/>
  <p>Hình 5.57 Tạo node Gmail</p>
</div>

Quá trình cấu hình có thể tham khảo [tài liệu chính thức của n8n](https://docs.n8n.io/integrations/builtin/credentials/google/oauth-single-service/?utm_source=n8n_app&utm_medium=credential_settings&utm_campaign=create_new_credentials_modal#enable-apis). API của Gmail được cấu hình [tại đây](https://console.cloud.google.com/apis/library/gmail.googleapis.com?project=apt-entropy-471905-b9). Bạn cần tạo thông tin xác thực (credentials), chọn loại Web application, và cuối cùng lấy client ID và client secret cần thiết. Bạn cũng cần thêm OAuth Redirect URL do n8n cung cấp vào các authorized redirect URIs. Đồng thời, bạn cũng cần thêm địa chỉ email của chính mình trong mục Add users tại [Audience](https://console.cloud.google.com/auth/audience?project=apt-entropy-471905-b9). Trang cấu hình cuối cùng được thể hiện trong Hình 5.58.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-07.png" alt="" width="90%"/>
  <p>Hình 5.58 Tài khoản Gmail được nạp thành công</p>
</div>

Giờ đây chúng ta có thể nhấp vào `Fetch Test Event` để lấy email, như trong Hình 5.59!

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-08.png" alt="" width="90%"/>
  <p>Hình 5.59 Lấy email theo thời gian thực</p>
</div>

(2) Cấu hình node AI Agent

Đây là bộ não của toàn bộ workflow. Kéo một node `AI Agent` từ menu node và cấu hình nó như sau:

- **Chat Model**: Kết nối mô hình ngôn ngữ lớn mà bạn chọn, chẳng hạn như `Google Gemini Chat Model`. Đây là "lõi tư duy" của Agent.
- **Memory**: Kết nối một node `Simple Memory`. Điều này cho phép Agent ghi nhớ lịch sử hội thoại trước đó khi xử lý nhiều email qua lại trong cùng một luồng email (email thread).
- **Tools**: Chúng ta có thể kết nối nhiều công cụ ở đây. Trong case của chúng ta, chúng ta kết nối hai công cụ:
  1. `SerpAPI`: Đây là API mà chúng ta đã sử dụng trong case ở Chương 4, trao cho Agent khả năng tìm kiếm thông tin công khai trên mạng.
  2. `Simple Vector Store`: Trao cho Agent khả năng truy vấn knowledge base riêng tư mà chúng ta đã tạo trong phần đầu tiên.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-09.png" alt="" width="90%"/>
  <p>Hình 5.60 Cài đặt node AI Agent</p>
</div>

Đây là bước đầu tiên của việc "tư duy" của Agent. Thêm một node `Gemini` (hoặc node LLM khác), thiết lập chế độ thành `Chat`. Mục tiêu của chúng ta là để nó phân tích nội dung email và phán đoán ý định người dùng. Thiết kế Prompt là cực kỳ quan trọng; một chỉ dẫn rõ ràng có thể giúp LLM hoàn thành tác vụ chính xác hơn. Chúng ta truyền phần thân và tiêu đề email (`{{ $json.snippet }}{{ $json.Subject }}`) làm biến vào Prompt. Nếu bạn không có API, bạn có thể vào [Google AI Studio](https://aistudio.google.com/prompts/new_chat) và nhấp Get API key để tạo một API khả dụng.

Đối với node AI Agent, chúng ta chủ yếu cần điền vào các phần `User Message` và `System Message`, như trong Hình 5.61.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-10.png" alt="" width="90%"/>
  <p>Hình 5.61 Chi tiết node AI Agent</p>
</div>

Dưới đây là Prompt được sử dụng trong case của chúng ta:

```json
# Prompt (User Message)
# Context Information
- Current Time: {{ new Date().toLocaleString('en-AU', { timeZone: 'Australia/Sydney', hour12: false }) }} (Sydney, Australia time)
- Sender: {{ $json.From }}
- Subject: {{ $json.Subject }}
- Email Body: {{ $json.snippet }}

# System Message
# Role and Goal
You are a 24/7 on-call, professional and efficient AI email assistant. Your task is: to do your best to answer all questions in emails using public information at the first opportunity, and add contextual status reminders at the beginning of replies based on my work schedule.

# Context Information
- Current Time: {{ new Date().toLocaleString('en-AU', { timeZone: 'Australia/Sydney', hour12: false }) }} (Sydney, Australia time)
- Email information is in the input data.

# Available Tools
- Simple Vector Store2: Used to query my exact working hours (e.g., Monday to Friday, 9 AM to 5 PM).
- SerpAPI: **[Primary Information Source]** Prioritize using this tool to search the internet to answer specific questions in emails.

# Execution Steps
1.  **Analyze the Question**: First, carefully read the email content and extract the sender's core question.

2.  **Parallel Information Gathering**: Execute the following two operations simultaneously to collect information:
    a. Use the `SerpAPI` tool to search online for answers to the sender's questions.
    b. Use the `Simple Vector Store2` tool to get my set exact working hours.

3.  **Draft Core Reply**: Based on the information collected by `SerpAPI`, clearly and directly answer the sender's question. This part will serve as the main body of the email reply.

4.  **Add Status Prefix and Integrate**:
    a. Compare "Current Time" with the working hours I obtained from the tool.
    b. **If currently "Non-working Hours"**: Create a status reminder prefix. This prefix **must include** the specific working hours obtained from `Simple Vector Store2`.
        * **Prefix Example**: "Hello, thank you for your email. You have contacted me during my non-working hours (my working hours are: [insert queried working hours here]). I will personally review this email on the next working day. In the meantime, here is a preliminary reply found for you based on public information:**<br><br>---<br><br>**"
    c. **If currently "Working Hours"**: Just use a simple greeting.
        * **Prefix Example**: "Hello, regarding your question, the reply is as follows:**<br><br>---<br><br>**"
    d. Concatenate the generated prefix and the core reply you drafted (result of step 3) to form the final email body.

5.  **Formatted Output**: You must output the finally generated email content in a strict JSON format. The format is as follows, do not add any additional explanations or text:
    {
      "shouldReply": true,
      "subject": "Re: [Original Email Subject]",
      "body": "[Here is the concatenated, complete email reply body, **all line breaks must use HTML <br> tags**]"
    }

# Rules and Restrictions
- **Always Try to Answer First**: At any time, your primary task is to use `SerpAPI` to provide valuable replies to users.
- **Must Declare Status**: If replying during non-working hours, you must clearly state this at the beginning of the email and attach my exact working hours.
- **Information Sources Must Be Accurate**: Working hours must strictly follow the results of `Simple Vector Store2`; question answers mainly come from `SerpAPI`, do not fabricate information.
- **Output Format**: **In the final output JSON, all line breaks in the `body` field must use `<br>` tags, not `\n`.**
```

(3) Cấu hình các công cụ của Agent

Đối với công cụ `Simple Vector Store`, chúng ta cần thực hiện các cấu hình then chốt để đảm bảo nó có thể "đọc" chính xác tri thức mà chúng ta đã lưu trước đó:

- **Chế độ vận hành (Operation Mode)**: `Retrieve Documents (As Tool for AI Agent)` (chế độ đọc dưới dạng công cụ).
- **Memory Key**: Phải điền **chính xác cùng một** Key như trong phần đầu tiên, tức là `my_private_knowledge`.
- **Embeddings**: Phải sử dụng **chính xác cùng một** mô hình `Embeddings Google Gemini` như trong phần đầu tiên.

Chỉ khi `Memory Key` và mô hình `Embeddings` hoàn toàn nhất quán thì Agent mới có thể sử dụng đúng "chìa khóa" và "ngôn ngữ" để truy cập knowledge base, như trong Hình 5.62.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-11.png" alt="" width="90%"/>
  <p>Hình 5.62 Cấu hình công cụ Simple Vector Store</p>
</div>

Tham số Description là định nghĩa mô tả của công cụ khi AI Agent gọi nó. Dưới đây là Prompt tương ứng:

```json
This is the Simple Vector Store2 tool, used to query my personal information, especially my working hours and email reply policy. When you need to determine whether it is currently working hours, or need to inform the other party when I will reply to emails, you must use this tool.
```

Đối với Memory, điều duy nhất cần lưu ý là ở đây chúng ta sử dụng tên luồng (thread name) của mỗi hộp thư làm định danh duy nhất để đảm bảo tính duy nhất của việc lưu trữ. Key được thiết lập thành `{{ $('Gmail').item.json.threadId }}`



(4) Gửi phản hồi cuối cùng

Bước cuối cùng là thực thi. Kết nối đầu ra của node `AI Agent` với một node `Gmail`, thiết lập **Operation** thành `Send`. Sử dụng các biểu thức của n8n để liên kết người nhận, tiêu đề và phần thân với các trường tương ứng trong dữ liệu JSON được xuất ra bởi `AI Agent` nhằm đạt được việc trả lời email tự động, như trong Hình 5.63.

- **To**: `{{ $('Gmail').item.json.From }}` (hoặc trường người gửi trong các trigger khác)
- **Subject**: `Re:  {{ $('Gmail').item.json.Subject }}`
- **Message**: `{{ $json.output }}`

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-12.png" alt="" width="90%"/>
  <p>Hình 5.63 Sơ đồ công cụ phản hồi cuối cùng</p>
</div>

Và khi việc gửi thành công, bạn cũng có thể nhận được thông tin email phản hồi thực trong hộp thư cá nhân của mình, như trong Hình 5.64.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-13.png" alt="" width="90%"/>
  <p>Hình 5.64 Định dạng email phản hồi trong hộp thư cá nhân</p>
</div>

Đến đây, một dịch vụ khách hàng thông minh tích hợp dựa trên node `AI Agent` đã hoàn thành. Bạn có thể gửi một email thử nghiệm để kiểm chứng kết quả hoạt động của nó. Kiến trúc này có khả năng mở rộng cực kỳ mạnh mẽ. Trong tương lai, bạn có thể trực tiếp thêm nhiều công cụ hơn (như lịch, cơ sở dữ liệu, CRM, v.v.) vào node `AI Agent`. Bạn chỉ cần dạy Agent cách sử dụng chúng trong Prompt để liên tục trao cho Agent của bạn những năng lực mạnh mẽ hơn.

### 5.5.5 Phân tích ưu điểm và hạn chế của n8n

Thông qua việc thực hành xây dựng một trợ lý email thông minh từ đầu, chúng ta đã có được một hiểu biết trực quan về phương thức làm việc của n8n. Là một nền tảng tự động hóa low-code mạnh mẽ, n8n thể hiện xuất sắc trong việc trao quyền cho việc phát triển ứng dụng Agent, nhưng nó không phải là vạn năng. Như trong Bảng 5.2, chúng ta sẽ phân tích một cách khách quan các ưu điểm và những hạn chế tiềm tàng của nó.

<div align="center">
  <p>Bảng 5.2 Tóm tắt ưu điểm và hạn chế của nền tảng n8n</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/5-figures/n8n-14.png" alt="" width="90%"/>
</div>

Đầu tiên, ưu điểm nổi bật nhất của n8n nằm ở **hiệu quả phát triển** của nó. Nó trừu tượng hóa logic phức tạp thành các workflow trực quan, dễ hiểu. Dù là việc nhận email, ra quyết định AI, gọi công cụ hay trả lời cuối cùng, toàn bộ luồng dữ liệu và chuỗi xử lý đều rõ ràng trong nháy mắt trên canvas. Đặc tính low-code này hạ thấp đáng kể rào cản kỹ thuật, cho phép các nhà phát triển nhanh chóng xây dựng và kiểm chứng logic cốt lõi của Agent, rút ngắn đáng kể khoảng cách từ ý tưởng đến nguyên mẫu.

Thứ hai, nền tảng **mạnh mẽ và có tính tích hợp cao**. n8n có một thư viện node dựng sẵn phong phú có thể dễ dàng kết nối hàng trăm dịch vụ phổ biến như Gmail và Google Gemini. Quan trọng hơn, node `AI Agent` nâng cao của nó tích hợp cao độ việc quản lý mô hình, bộ nhớ và công cụ, cho phép chúng ta triển khai việc ra quyết định tự chủ phức tạp chỉ với một node, điều này thanh lịch và mạnh mẽ hơn nhiều so với việc định tuyến thủ công đa node truyền thống. Đồng thời, đối với những tình huống mà các chức năng dựng sẵn không thể bao phủ, node `Code` cũng cung cấp sự linh hoạt để viết mã tùy chỉnh, đảm bảo giới hạn trên của chức năng.

Cuối cùng, ở cấp độ **triển khai và vận hành**, n8n hỗ trợ **triển khai riêng tư (private deployment)**, và hiện tại nó là một giải pháp Agent riêng tư tương đối đơn giản có thể triển khai một phiên bản hoàn chỉnh của dự án. Điều này cực kỳ quan trọng đối với các doanh nghiệp coi trọng bảo mật và quyền riêng tư dữ liệu. Chúng ta có thể triển khai toàn bộ dịch vụ trên máy chủ của riêng mình để đảm bảo rằng các thông tin nhạy cảm như email nội bộ và dữ liệu khách hàng không rời khỏi môi trường của chính chúng ta, cung cấp một nền tảng vững chắc cho tính tuân thủ của các ứng dụng Agent.

Tất nhiên, mọi công cụ đều có sự đánh đổi của nó. Trong khi tận hưởng sự tiện lợi mà n8n mang lại, chúng ta cũng phải nhận ra các hạn chế của nó.

Đằng sau **hiệu quả phát triển** là **việc gỡ lỗi và xử lý lỗi tương đối cồng kềnh**. Khi các workflow trở nên phức tạp, một khi xảy ra lỗi định dạng dữ liệu, các nhà phát triển có thể cần kiểm tra đầu vào và đầu ra của từng node một để định vị vấn đề, điều này đôi khi không trực tiếp như việc đặt breakpoint trong mã.

Về mặt chức năng, hạn chế lớn nhất được phản ánh ở **tính không bền vững (non-persistence) của bộ lưu trữ tích hợp sẵn**. `Simple Memory` và `Simple Vector Store` mà chúng ta đã sử dụng trong case đều dựa trên bộ nhớ (memory-based), nghĩa là một khi dịch vụ n8n khởi động lại, toàn bộ lịch sử hội thoại và knowledge base sẽ bị mất. Điều này là chí mạng đối với các ứng dụng môi trường production. Do đó, trong triển khai thực tế, chúng phải được thay thế bằng các cơ sở dữ liệu bền vững bên ngoài như Redis và Pinecone, điều này cũng làm tăng thêm chi phí cấu hình và bảo trì bổ sung.

Ngoài ra, về mặt **triển khai và vận hành** cũng như hợp tác nhóm, **kiểm soát phiên bản và hợp tác nhiều người của n8n chưa trưởng thành bằng mã truyền thống**. Mặc dù các workflow có thể được xuất ra dưới dạng file JSON để quản lý, việc so sánh các thay đổi của chúng kém rõ ràng hơn nhiều so với `git diff` mã, và việc nhiều người cùng chỉnh sửa một workflow đồng thời rất dễ gây ra xung đột.

Cuối cùng, về **hiệu năng**, n8n hoàn toàn có thể đáp ứng phần lớn các tác vụ tự động hóa doanh nghiệp và các tác vụ Agent tần suất trung bình đến thấp. Tuy nhiên, đối với những tình huống cần xử lý các yêu cầu đồng thời siêu cao, cơ chế lập lịch node của nó có thể mang lại một số chi phí hiệu năng nhất định, có thể hơi thua kém các dịch vụ được triển khai bằng mã thuần túy.

## 5.6 Tóm tắt chương

Chương này giới thiệu một cách có hệ thống các khái niệm, phương pháp và thực hành xây dựng ứng dụng agent dựa trên các nền tảng low-code, đánh dấu bước chuyển tiếp quan trọng của chúng ta từ "viết mã thủ công" sang "phát triển dựa trên nền tảng".

Trong phần đầu tiên, chúng ta đã trình bày chi tiết về bối cảnh và giá trị của sự trỗi dậy của các nền tảng low-code. So với các agent được triển khai thuần bằng mã trong Chương 4, các nền tảng low-code hạ thấp đáng kể rào cản kỹ thuật, nâng cao hiệu quả phát triển và cung cấp trải nghiệm gỡ lỗi trực quan tốt hơn thông qua các phương pháp trực quan hóa và mô-đun hóa. "Mức độ trừu tượng cao hơn" này cho phép các nhà phát triển tập trung năng lượng vào logic nghiệp vụ và prompt engineering thay vì các chi tiết triển khai cấp thấp.

Tiếp theo, chúng ta đã thực hành sâu bốn nền tảng tiêu biểu có đặc trưng riêng biệt:

**Coze** nổi bật với trải nghiệm thân thiện zero-code và hệ sinh thái plugin phong phú. Thông qua case "Bản tin AI hằng ngày", chúng ta đã trải nghiệm cách nhanh chóng tổng hợp thông tin đa nguồn thông qua cấu hình kéo-thả và xuất bản lên nhiều nền tảng chủ đạo chỉ với một chạm. Coze đặc biệt phù hợp cho những người dùng không có nền tảng kỹ thuật và các tình huống cần nhanh chóng kiểm chứng ý tưởng, nhưng các hạn chế của nó về việc không hỗ trợ MCP và không thể xuất các file cấu hình chuẩn hóa cũng đáng lưu ý.

**Dify**, với tư cách là một nền tảng cấp doanh nghiệp mã nguồn mở, thể hiện năng lực phát triển full-stack. Case "Trợ lý cá nhân Super Agent" bao phủ nhiều module như hỏi đáp hằng ngày, tối ưu hóa văn bản, tạo sinh đa phương thức, phân tích dữ liệu và tích hợp công cụ MCP, minh họa đầy đủ năng lực điều phối mạnh mẽ của Dify trong các tình huống nghiệp vụ phức tạp. Chợ plugin phong phú của nó (8000+), các phương thức triển khai linh hoạt và các tính năng bảo mật cấp doanh nghiệp khiến nó trở thành một lựa chọn lý tưởng cho các nhà phát triển chuyên nghiệp và các đội nhóm doanh nghiệp. Tuy nhiên, đường cong học tập tương đối dốc và các thách thức về hiệu năng trong các tình huống đồng thời cao cũng cần được cân nhắc.

**FastGPT** nổi bật với trải nghiệm knowledge base RAG tối ưu, trở thành một đối thủ cạnh tranh mạnh mẽ trong các tình huống hỏi đáp lĩnh vực chuyên biệt. Thông qua case "Trợ lý cố vấn đầu tư thông minh", chúng ta đã trải nghiệm mô hình phát triển hoàn chỉnh từ xây dựng knowledge base, tích hợp công cụ MCP đến điều phối workflow trực quan. Sự kiểm soát chi tiết của FastGPT đối với việc chia nhỏ file, tăng cường chỉ mục và nhận diện hình ảnh mang lại cho nó những ưu thế độc đáo trong các tình huống trợ lý tri thức doanh nghiệp và dịch vụ khách hàng thông minh. Tuy nhiên, hệ sinh thái template tương đối yếu và hạn mức phiên bản miễn phí eo hẹp cũng ràng buộc hiệu suất của nó trong các tình huống nghiệp vụ phức tạp hơn.

**n8n** mở ra một con đường khác với năng lực "kết nối" độc đáo của mình. Thông qua case "Trợ lý email thông minh", chúng ta đã thấy cách nhúng liền mạch năng lực AI vào các quy trình tự động hóa nghiệp vụ phức tạp. Node AI Agent của n8n tích hợp cao độ các mô hình, bộ nhớ và công cụ, và kết hợp với hàng trăm node dựng sẵn của nó, có thể đạt được các giải pháp tự động hóa được tùy chỉnh cao. Việc nó hỗ trợ triển khai riêng tư đặc biệt quan trọng đối với các doanh nghiệp coi trọng bảo mật dữ liệu. Tuy nhiên, tính không bền vững của bộ lưu trữ tích hợp sẵn và sự chưa trưởng thành của việc kiểm soát phiên bản đòi hỏi việc xử lý kỹ thuật bổ sung trong môi trường production.

Thông qua việc thực hành so sánh bốn nền tảng, chúng ta có thể rút ra các gợi ý lựa chọn sau:
- **Kiểm chứng nguyên mẫu nhanh, người dùng phi kỹ thuật**: Ưu tiên Coze
- **Ứng dụng cấp doanh nghiệp, logic nghiệp vụ phức tạp, tạo sinh đa phương thức**: Ưu tiên Dify
- **Hệ thống hỏi đáp dựa trên knowledge base riêng tư, dịch vụ khách hàng thông minh**: Ưu tiên FastGPT
- **Tích hợp nghiệp vụ sâu, quy trình tự động hóa tổng quát**: Ưu tiên n8n

Điều đáng nhấn mạnh là các nền tảng low-code không nhằm thay thế việc phát triển bằng mã mà cung cấp một lựa chọn bổ trợ. Trong các dự án thực tế, chúng ta có thể chuyển đổi linh hoạt tùy theo nhu cầu của các giai đoạn khác nhau: dùng các nền tảng low-code để nhanh chóng kiểm chứng ý tưởng, dùng mã để đạt được sự kiểm soát chi tiết; dùng nền tảng để xử lý các quy trình chuẩn hóa, dùng mã để xử lý logic đặc biệt. Tư duy "phát triển kết hợp (hybrid development)" này là best practice cho kỹ thuật agent.

Trong chương tiếp theo, chúng ta sẽ khám phá sâu hơn các framework agent cấp thấp hơn để giúp độc giả xây dựng các ứng dụng đáng tin cậy và thú vị hơn.


## Bài tập

1. Chương này giới thiệu bốn nền tảng low-code có đặc trưng riêng biệt: `Coze`, `Dify`, `FastGPT` và `n8n`. Hãy phân tích:

   - Sự khác biệt về định vị cốt lõi và triết lý thiết kế giữa bốn nền tảng này là gì? Chúng lần lượt giải quyết những điểm nhức nhối nào trong việc phát triển agent?
   - Các nền tảng low-code và phát triển thuần bằng mã đều có ưu và nhược điểm riêng. Ngoài ra, còn có một chế độ "phát triển kết hợp (hybrid development)" trong đó một số chức năng được triển khai bằng nền tảng và một số bằng mã. Hãy suy nghĩ xem mỗi chế độ trong ba chế độ phát triển này phù hợp với những tình huống nào? Hãy cho ví dụ.

2. Trong case `Coze` ở Phần 5.2, chúng ta đã xây dựng một agent "Bản tin AI hằng ngày". Hãy mở rộng tư duy dựa trên case này:

   > **Gợi ý**: Đây là một câu hỏi thực hành, khuyến nghị thao tác thực tế

   - Việc tạo bản tin hiện tại được kích hoạt thụ động (người dùng chủ động hỏi). Làm thế nào để cải tạo agent này để nó có thể tự động tạo bản tin và đẩy chúng đến các nhóm Feishu hoặc tài khoản công khai WeChat được chỉ định vào 8 giờ sáng mỗi ngày?
   - Chất lượng của bản tin phụ thuộc rất nhiều vào thiết kế prompt. Hãy thử tối ưu hóa prompt trong Phần 5.2.2 để làm cho bản tin được tạo ra chuyên nghiệp hơn, với cấu trúc rõ ràng hơn, hoặc thêm các chức năng mới như "phân tích điểm nóng" và "dự đoán xu hướng".
   - Việc `Coze` hiện chưa hỗ trợ giao thức `MCP` được coi là một hạn chế quan trọng (tại thời điểm viết các bài tập này, mặc dù `feature-mcp` đã có trong [`Coze Studio Q4 2025 Product Roadmap`](https://github.com/coze-dev/coze-studio/issues/2218), nó vẫn chưa được triển khai). Hãy mô tả ngắn gọn giao thức `MCP` là gì? Vì sao nó quan trọng? Nếu `Coze` hỗ trợ `MCP` trong tương lai, nó sẽ mang lại những khả năng mới nào?

3. Trong case `Dify` ở Phần 5.3, chúng ta đã xây dựng một "Trợ lý cá nhân Super Agent" đầy đủ chức năng. Hãy phân tích sâu:

   - Case sử dụng một "bộ phân loại câu hỏi (question classifier)" để định tuyến thông minh, phân phối các loại yêu cầu khác nhau đến các sub-agent khác nhau. Những ưu điểm của kiến trúc đa-agent này là gì? Nếu bạn không sử dụng bộ phân loại mà để một agent đơn lẻ xử lý tất cả các tác vụ, bạn sẽ gặp phải những vấn đề gì?
   - Module truy vấn dữ liệu cần cung cấp cho mô hình lớn thông tin cấu trúc bảng rõ ràng. Nếu cơ sở dữ liệu có 50 bảng, mỗi bảng 20 trường, việc đưa trực tiếp tất cả các câu lệnh `DDL` vào prompt sẽ khiến ngữ cảnh (context) quá dài. Hãy thiết kế một giải pháp thông minh hơn để giải quyết vấn đề này.
   - `Dify` hỗ trợ cả chế độ triển khai cục bộ và triển khai đám mây. Hãy so sánh sự khác biệt giữa hai chế độ này về bảo mật dữ liệu, chi phí, hiệu năng và độ khó bảo trì, và giải thích các tình huống áp dụng của mỗi chế độ.

4. Trong case `FastGPT` ở Phần 5.4, chúng ta đã xây dựng một "Trợ lý cố vấn đầu tư thông minh". Hãy phân tích sâu:

   - Ưu điểm cốt lõi của FastGPT là sự tối ưu hóa sâu sắc RAG pipeline của nó. Hãy so sánh việc xử lý knowledge base của FastGPT (chia nhỏ file, tăng cường chỉ mục, nhận diện hình ảnh) với chức năng knowledge base của Dify. Sự khác biệt về triết lý thiết kế và các tình huống áp dụng giữa hai bên là gì?
   - Case sử dụng các công cụ MCP để lấy dữ liệu cổ phiếu thời gian thực và tạo các biểu đồ trực quan. Nếu FastGPT không hỗ trợ MCP nguyên bản, bạn sẽ đạt được cùng chức năng đó như thế nào? Hãy đề xuất một giải pháp thay thế.
   - Phiên bản miễn phí của FastGPT chỉ có 100 credit và 30 QPM. Đối với một đội khởi nghiệp cần phục vụ 1000 người dùng, bạn sẽ thiết kế một giải pháp cân bằng giữa chi phí và hiệu năng như thế nào?

5. Trong case `n8n` ở Phần 5.5, chúng ta đã xây dựng một "Trợ lý email thông minh". Hãy suy nghĩ về các câu hỏi sau:

   > **Gợi ý**: Đây là một câu hỏi thực hành, khuyến nghị thao tác thực tế

   - `Simple Vector Store` và `Simple Memory` được sử dụng trong case đều dựa trên bộ nhớ, và dữ liệu sẽ bị mất sau khi khởi động lại dịch vụ. Hãy tham khảo tài liệu `n8n`, thử thay thế chúng bằng các giải pháp lưu trữ bền vững (như `Pinecone`, `Redis`, v.v.), và giải thích quá trình cấu hình.
   - Trợ lý email hiện tại chỉ có thể xử lý email văn bản. Nếu email do người dùng gửi chứa các tệp đính kèm (như tài liệu `PDF`, hình ảnh), bạn sẽ mở rộng workflow này như thế nào để agent có thể hiểu nội dung tệp đính kèm và đưa ra các phản hồi tương ứng?
   - Ưu điểm cốt lõi của `n8n` nằm ở năng lực "kết nối" của nó. Hãy thiết kế một tình huống tự động hóa phức tạp hơn: khi một khách hàng đặt hàng trên một nền tảng thương mại điện tử, tự động kích hoạt một loạt các thao tác (gửi email xác nhận, cập nhật cơ sở dữ liệu tồn kho, thông báo cho hệ thống logistics, ghi lại thông tin khách hàng vào `CRM`). Hãy vẽ sơ đồ kết nối node của workflow và giải thích các cấu hình then chốt.

6. Prompt engineering cũng cực kỳ quan trọng trong các nền tảng low-code. Chương này trình bày nhiều case thiết kế prompt trên các nền tảng. Hãy phân tích:

   - So sánh các thiết kế prompt trong Phần 5.2.2 (`Coze`), Phần 5.3.2 (`Dify`), Phần 5.4.2 (`FastGPT`) và Phần 5.5.4 (`n8n`). Sự khác biệt về cấu trúc, phong cách và trọng tâm là gì? Những khác biệt này có liên quan đến đặc trưng của nền tảng không?
   - Trong "Module Tối ưu hóa văn bản" của `Dify`, prompt yêu cầu đầu ra "vượt quá 500 từ". Yêu cầu cứng nhắc về độ dài đầu ra này có hợp lý không? Trong những tình huống nào nên giới hạn độ dài đầu ra, và trong những tình huống nào nên cho phép mô hình tự do thể hiện?

7. Công cụ và plugin là các phương thức mở rộng năng lực cốt lõi của các nền tảng low-code. Hãy suy nghĩ:

   - `Coze` có một cửa hàng plugin phong phú, `Dify` có một chợ plugin 8000+, `FastGPT` hỗ trợ nguyên bản giao thức MCP, và `n8n` có hàng trăm node dựng sẵn. Nếu không nền tảng nào trong bốn nền tảng này có một công cụ cụ thể mà bạn cần (chẳng hạn như "kết nối với `API` hệ thống nội bộ của công ty"), bạn sẽ giải quyết nó như thế nào?
   - Trong Phần 5.3.2, chúng ta đã sử dụng giao thức `MCP` để tích hợp các dịch vụ như Amap và gợi ý ăn uống. Hãy nghiên cứu và giải thích: Sự khác biệt giữa giao thức `MCP` và `RESTful API` truyền thống cũng như `Tool Calling` là gì? Vì sao `MCP` được gọi là "tiêu chuẩn mới" cho việc gọi công cụ của agent?
   - Giả sử bạn muốn phát triển một plugin tùy chỉnh cho `Dify` để nó có thể gọi hệ thống knowledge base nội bộ của công ty bạn. Hãy tham khảo tài liệu phát triển plugin của `Dify` và phác thảo quy trình phát triển cùng các điểm kỹ thuật then chốt.

8. Lựa chọn nền tảng là một trong những quyết định then chốt cho sự thành công của các sản phẩm agent. Giả sử bạn là trưởng nhóm kỹ thuật của một công ty khởi nghiệp, và công ty dự định phát triển ba ứng dụng AI sau đây. Hãy chọn nền tảng phù hợp nhất cho mỗi ứng dụng (`Coze`, `Dify`, `FastGPT`, `n8n`, hoặc phát triển thuần bằng mã) và giải thích chi tiết:

   **Ứng dụng A**: Một mini-program "Trợ lý viết lách AI" dành cho người dùng C-end, cần được ra mắt nhanh chóng để kiểm chứng nhu cầu thị trường, với ngân sách hạn chế, và đội nhóm chỉ có 1 kỹ sư front-end và 1 quản lý sản phẩm.

   **Ứng dụng B**: Một "Hệ thống rà soát hợp đồng thông minh" dành cho khách hàng doanh nghiệp, cần xử lý các tài liệu pháp lý nhạy cảm, yêu cầu dữ liệu không được rời khỏi môi trường riêng tư của khách hàng, và cần tích hợp sâu với hệ thống OA hiện có cùng hệ thống quản lý tài liệu của khách hàng.

   **Ứng dụng C**: Một "Công cụ nâng cao hiệu quả R&D" nội bộ, cần tự động hóa nhiều khâu quy trình R&D như rà soát mã, tạo báo cáo kiểm thử, theo dõi bug và đồng bộ tiến độ dự án. Đội nhóm có năng lực kỹ thuật mạnh.

   Đối với mỗi ứng dụng, hãy phân tích từ các chiều sau (bao gồm nhưng không giới hạn):

   > **Gợi ý**: Liệu năng lực nền tảng có đáp ứng yêu cầu hay không, tốc độ ra mắt nhanh đến đâu, chi phí phát triển, chi phí vận hành, độ khó của việc lặp lại sau này, không gian cho việc mở rộng chức năng trong tương lai

   - Tính khả thi kỹ thuật
   - Hiệu quả phát triển
   - Kiểm soát chi phí
   - Khả năng bảo trì
   - Khả năng mở rộng
   - Bảo mật dữ liệu và tuân thủ

## Tài liệu tham khảo

[1] Coze - Nền tảng phát triển ứng dụng AI thế hệ mới. https://www.coze.cn/

[2] Dify - Nền tảng phát triển ứng dụng LLM mã nguồn mở. https://dify.ai/

[3] FastGPT - Nền tảng hỏi đáp knowledge base và công cụ xây dựng Agent mã nguồn mở. https://fastgpt.io/en/

[4] n8n - Công cụ tự động hóa workflow. https://n8n.io/
