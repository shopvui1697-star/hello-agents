# Giới thiệu và Thực hành GUI Agent — Hành trình khám phá thế hệ tương tác người-máy tiếp theo

## Lời mở đầu: Khi AI học cách "nhìn" màn hình

Hãy tưởng tượng một tình huống như thế này: bạn nói với điện thoại "giúp tôi đặt một vé tàu cao tốc đi Thượng Hải vào ngày mai, hạng ghế hai, khởi hành khoảng 10 giờ sáng", rồi AI tự động mở APP đường sắt 12306, điền điểm đi, điểm đến và ngày tháng, lọc ra các chuyến tàu phù hợp, hoàn tất việc đặt vé và thanh toán — toàn bộ quá trình không cần bạn thao tác thủ công, AI giống như một trợ lý thực thụ, "nhìn" màn hình, "hiểu" giao diện, "nhấn" nút.

Đây không phải khoa học viễn tưởng, mà là hiện thực mà **GUI Agent (tác tử giao diện người dùng đồ họa)** đang biến thành sự thật.

Trong hai thập kỷ qua, phương án chủ đạo cho tự động hóa doanh nghiệp là **RPA (tự động hóa quy trình bằng robot)**. Tuy nhiên, RPA có một điểm yếu chí mạng: nó phụ thuộc vào các bộ chọn phần tử UI cố định (Selectors), một khi giao diện thay đổi dù chỉ một chút, kịch bản sẽ mất hiệu lực. Sự mong manh này dẫn đến chi phí bảo trì khổng lồ.

Còn sự xuất hiện của GUI Agent đã thay đổi hoàn toàn cục diện này. Nó không đơn thuần "phát lại" các kịch bản định sẵn, mà giống như con người, thông qua **cảm nhận thị giác** để hiểu nội dung màn hình, thông qua **năng lực suy luận của mô hình ngôn ngữ lớn** để lập kế hoạch đường đi thao tác, tự chủ hoàn thành nhiệm vụ trong môi trường phần mềm động và chưa biết trước.

Chương này sẽ đưa bạn tìm hiểu sâu về nguyên lý kỹ thuật của GUI Agent, và thông qua ba trường hợp (case) thực hành, giúp bạn thực sự nắm vững cách sử dụng và triển khai các hệ thống tác tử tiên tiến này.

## Phần thứ nhất: Giới thiệu kỹ thuật về GUI Agent

### 1.1 GUI Agent là gì?

**GUI Agent (tác tử giao diện người dùng đồ họa)** là một loại hệ thống AI có khả năng tự chủ hiểu và thao tác giao diện đồ họa. Khác với việc gọi API truyền thống hay công cụ dòng lệnh, GUI Agent tương tác trực tiếp với giao diện đồ họa mà con người sử dụng — bất kể đó là APP điện thoại, phần mềm máy tính để bàn hay ứng dụng web.

#### 1.1.1 Sự chuyển đổi mô hình (paradigm) từ RPA sang AI Agent

Hãy cùng hiểu sự chuyển đổi này thông qua một phép so sánh:

| Chiều cạnh       | RPA truyền thống                                      | GUI Agent (AI Agent)                          |
| ---------------- | ----------------------------------------------------- | --------------------------------------------- |
| **Nguyên lý hoạt động** | Phát lại kịch bản dựa trên bộ chọn cố định (như XPath, ID) | Thao tác tự chủ dựa trên hiểu thị giác và suy luận của mô hình ngôn ngữ |
| **Tính thích ứng** | Giao diện thay đổi là mất hiệu lực              | Thích ứng được với thay đổi giao diện, có độ đàn hồi ngữ nghĩa |
| **Lập kế hoạch nhiệm vụ** | Cần con người định sẵn từng bước thao tác     | Tự chủ phân rã nhiệm vụ theo chỉ thị ngôn ngữ tự nhiên |
| **Khả năng đa nền tảng** | Cần viết kịch bản chuyên biệt cho từng nền tảng | Phương án thị giác dùng chung, tự nhiên xuyên nền tảng |
| **Chi phí bảo trì** | Cực cao (UI thay đổi phải viết lại kịch bản)     | Thấp (mô hình tự động thích ứng)              |

**Khác biệt cốt lõi**: RPA là "tự động hóa mong manh", còn GUI Agent là "tự chủ thông minh".

#### 1.1.2 Vì sao GUI Agent bỗng nhiên nổi lên?

Sự bùng nổ của GUI Agent không phải ngẫu nhiên, mà là kết quả của việc nhiều lĩnh vực kỹ thuật cùng trưởng thành đồng bộ. Trước hết là bước tiến đột phá của các mô hình lớn đa phương thức (multimodal). Bắt đầu từ những mô hình như GPT-4o, Claude 3.5 Sonnet, Qwen-VL, các mô hình lớn không chỉ hiểu được chữ, mà còn "nhìn hiểu" được hình ảnh, điều này cung cấp cho GUI Agent một "đôi mắt" mạnh mẽ. Khi bạn đưa cho các mô hình này một ảnh chụp màn hình (screenshot), chúng có thể nhận diện chính xác "đây là một nút đăng nhập", "chỗ này có một ô tìm kiếm", thậm chí hiểu được bố cục giao diện phức tạp.

Quan trọng hơn nữa là bước đột phá về năng lực định vị. Các mô hình thị giác thời kỳ đầu giống như một người cận thị — nó biết trên màn hình có một cái nút, nhưng không nói rõ được cái nút nằm ở đâu. Còn các mô hình mới nhất (như GUI-Owl, Qwen-VL) qua huấn luyện chuyên biệt, có thể xuất ra chính xác tọa độ màn hình $(x, y)$ của phần tử UI, điều này giúp Agent không chỉ "nhìn thấy", mà còn "nhấn trúng".

Cuối cùng là sự thay đổi về chất trong năng lực suy luận. Năng lực tư duy chuỗi (Chain of Thought) của mô hình ngôn ngữ lớn đã trao cho Agent một "bộ não". Nó có thể phân rã chỉ thị mơ hồ như "đặt một vé tàu cao tốc cho ngày mai" thành các bước cụ thể như "mở APP → chọn ngày → nhập địa điểm → lọc chuyến tàu → xác nhận thanh toán", và không ngừng phản tư, sửa lỗi trong quá trình thực thi.

### 1.2 Kiến trúc kỹ thuật cốt lõi của GUI Agent

Một hệ thống GUI Agent hoàn chỉnh có thể được phân rã thành ba mô-đun cốt lõi: **Cảm nhận (Perception)** → **Suy luận (Reasoning)** → **Hành động (Action)**. Đây là một hệ thống ra quyết định tự chủ khép kín.

<div align="center">
  <img src="./images/Extra06-figures/image1.png" alt="GUI Agent 三层架构" width="90%"/>
  <p>Hình 1 Vòng lặp khép kín cảm nhận-suy luận-hành động của GUI Agent</p>
</div>


#### 1.2.1 Tầng cảm nhận: Máy móc "nhìn thấy" màn hình như thế nào

Tầng cảm nhận chịu trách nhiệm chuyển đổi thông tin màn hình thành dữ liệu mà máy móc có thể hiểu. Hiện tại chủ yếu có hai hướng kỹ thuật, chúng đại diện cho hai triết lý thiết kế hoàn toàn khác nhau.

Hướng thứ nhất là cảm nhận có cấu trúc dựa trên DOM hoặc cây khả năng truy cập (accessibility tree). Phương pháp này lấy cấu trúc nội bộ của ứng dụng thông qua API hệ thống — chẳng hạn cây HTML DOM của trang web, hoặc View Hierarchy của ứng dụng Android. Giống như cung cấp cho Agent một "bản vẽ kiến trúc", nó có thể biết chính xác loại và vị trí của từng nút, ô văn bản. Ưu điểm của phương pháp này là chính xác và hiệu quả, nhưng vấn đề cũng rất rõ ràng: nhiều ứng dụng hiện đại vốn không hề để lộ những thông tin có cấu trúc này. Giao diện vẽ bằng Canvas, game, phần mềm màn hình từ xa, đối với phương án dựa trên DOM đều là "hộp đen". Hơn nữa phương pháp này làm mất thông tin bố cục thị giác, rất khó hiểu được quan hệ không gian giữa các phần tử, khả năng tương thích đa nền tảng cũng rất kém.

Hướng thứ hai là cảm nhận thuần thị giác, đây cũng là hướng tiên tiến nhất hiện nay. Agent trực tiếp chụp ảnh màn hình, dùng mô hình thị giác lớn (VLM) để "nhìn" màn hình như con người. Tính phổ dụng của phương pháp này cực mạnh — bất kể giao diện của bạn được xây bằng công nghệ gì, chỉ cần hiển thị được trên màn hình, Agent đều có thể hiểu. Quan trọng hơn, nó có "độ đàn hồi ngữ nghĩa". Ngay cả khi một cái nút chuyển từ màu xanh sang màu lục, hoặc vị trí dịch chuyển đôi chút, Agent dựa trên thị giác vẫn có thể nhận diện qua ngữ nghĩa rằng "đây là nút đăng nhập". RPA truyền thống gặp tình huống này sẽ mất hiệu lực, nhưng GUI Agent lại xử lý dễ dàng. Tất nhiên, phương án thuần thị giác cũng có thách thức, khó khăn lớn nhất là độ chính xác định vị — mô hình không chỉ phải nhận diện nút là gì, mà còn phải xuất ra tọa độ màn hình chính xác của nó.

#### 1.2.2 Tầng suy luận: Quá trình ra quyết định của bộ não

Tầng suy luận là "bộ não" của GUI Agent, chịu trách nhiệm chuyển hóa chỉ thị trừu tượng của người dùng thành chuỗi thao tác cụ thể. Ở đây liên quan đến vài năng lực then chốt.

Trước hết là năng lực phân rã nhiệm vụ. Khi bạn nói với Agent "giúp tôi đặt một vé tàu cao tốc đi Thượng Hải vào ngày mai, hạng ghế hai, khởi hành khoảng 10 giờ sáng", nó cần hiểu logic phức tạp đằng sau câu nói này. Agent sẽ tự động tách nhu cầu mơ hồ này thành một loạt bước cụ thể: mở APP 12306 → nhấn "Đặt vé tàu" → nhập điểm đi "Bắc Kinh" → nhập điểm đến "Thượng Hải" → chọn ngày "ngày mai" → nhấn tra cứu → lọc chuyến tàu (hạng ghế hai + khoảng 10 giờ sáng) → chọn chuyến tàu phù hợp → nhấn đặt vé → điền thông tin hành khách → xác nhận thanh toán. Quá trình phân rã này phụ thuộc vào sự hiểu biết của mô hình ngôn ngữ lớn về kiến thức thông thường và quy trình nghiệp vụ.

Tinh vi hơn nữa là cơ chế chuỗi tư duy (chain of thought). Để nâng cao tỷ lệ thành công cho các nhiệm vụ phức tạp, GUI Agent hiện đại sẽ sinh ra "độc thoại nội tâm" trước mỗi bước thao tác. Ví dụ màn hình hiện tại là trang chủ 12306, mục tiêu của người dùng là đặt vé tàu cao tốc, Agent sẽ phân tích trước: "Tôi thấy trên màn hình có các lựa chọn 'Đặt vé tàu', 'Tra cứu đơn hàng', cần nhấn 'Đặt vé tàu' mới vào được quy trình mua vé." Sau đó ra quyết định: "Nhấn nút 'Đặt vé tàu' tại tọa độ (540, 320)." Quá trình tư duy tường minh này không chỉ khiến hành vi của Agent dễ giải thích hơn, mà còn giảm đáng kể sự tích lũy sai số trong các thao tác nhiều bước.

Cuối cùng là năng lực phản tư và sửa lỗi. Nếu sau khi Agent nhấn nút "tra cứu" mà phát hiện không xuất hiện danh sách chuyến tàu như mong đợi, thay vào đó lại hiện lời nhắc "vui lòng chọn ngày khởi hành", nó sẽ lập tức nhận ra: "Tôi đã bỏ sót bước chọn ngày." Rồi điều chỉnh chiến lược: "Nhấn vào bộ chọn ngày trước, chọn ngày mai, rồi tra cứu lại." Năng lực tự sửa lỗi này giúp Agent có thể ứng phó với đủ loại tình huống bất ngờ trong thế giới thực.

#### 1.2.3 Tầng thực thi: Từ quyết định đến hành động

Tầng thực thi là "đôi tay" của GUI Agent, chịu trách nhiệm chuyển hóa quyết định của mô hình thành thao tác hệ thống thực tế.

Khác với không gian mở của việc sinh văn bản, không gian hành động của thao tác GUI là hữu hạn và rõ ràng. Nhấn, nhấn đúp, giữ lâu, vuốt, nhập liệu, cuộn, kéo thả — những hành động cơ bản này tạo thành nền tảng cho mọi thao tác phức tạp. Mỗi loại hành động đều có tham số đặc trưng riêng, chẳng hạn nhấn cần tọa độ (x, y), vuốt cần điểm đầu và điểm cuối (x1, y1, x2, y2), nhập liệu cần nội dung văn bản.

Ở đây có một chi tiết kỹ thuật then chốt: sự chuyển đổi hệ tọa độ. Mô hình thị giác (như Qwen-VL) thường xuất ra tọa độ chuẩn hóa (0-1000), còn độ phân giải màn hình thực tế của điện thoại hay máy tính có thể là 1920x1080. Tầng thực thi phải thực hiện ánh xạ tọa độ chính xác, chuyển đầu ra của mô hình thành tọa độ vật lý. Hơn nữa các thiết bị khác nhau còn có DPI và tỷ lệ phóng đại hệ thống khác nhau, tất cả đều cần được tính đến. Một hàm ánh xạ đơn giản có thể như sau: trước tiên chia tọa độ chuẩn hóa cho 1000, rồi nhân với chiều rộng và chiều cao thực tế của màn hình, cuối cùng làm tròn để được tọa độ vật lý.

Phức tạp hơn là việc thích ứng đa nền tảng. Trên Android, mọi thao tác đều được thực hiện bằng cách gửi lệnh qua ADB (Android Debug Bridge), chẳng hạn `adb shell input tap 500 1000` để thực hiện nhấn, `adb shell input swipe 500 1000 500 500` để thực hiện vuốt. Trên iOS, cần thông qua libimobiledevice hoặc WDA (WebDriverAgent) để hiện thực chức năng tương tự. Còn trong môi trường máy tính để bàn Windows, Mac, Linux, thường dùng các thư viện Python như pyautogui, pynput để điều khiển trực tiếp chuột và bàn phím. Cùng một hành động "nhấn", cách hiện thực trên các nền tảng khác nhau lại hoàn toàn khác nhau, tầng thực thi cần cung cấp giao diện trừu tượng thống nhất cho từng nền tảng.

### 1.3 So sánh toàn cảnh các framework mã nguồn mở chủ đạo

Năm 2024-2025 là thời kỳ bùng nổ của GUI Agent, các công ty công nghệ lớn và cơ quan nghiên cứu đua nhau mã nguồn mở framework của mình. Hãy cùng so sánh một cách hệ thống vài dự án tiêu biểu nhất:

<div align="center">
  <img src="./images/Extra06-figures/image2.png" alt="主流GUI Agent框架对比" width="90%"/>
  <p>Hình 2 Biểu đồ radar so sánh toàn cảnh các framework GUI Agent chủ đạo</p>
</div>


### 1.4 Kịch bản ứng dụng và giới hạn kỹ thuật

#### 1.4.1 Năm kịch bản ứng dụng tiêu biểu

Tiềm năng ứng dụng của GUI Agent vượt xa trí tưởng tượng của chúng ta. Trong lĩnh vực buồng lái thông minh (intelligent cockpit), nhu cầu tương tác bằng giọng nói trong quá trình lái xe đang bùng nổ. Hãy tưởng tượng khi đang lái xe bạn nói "dẫn đường đến quán cà phê gần nhất, và trước khi đến 10 phút giúp tôi đặt một ly latte", GUI Agent có thể phối hợp APP dẫn đường và APP giao đồ ăn xuyên ứng dụng, hiểu được logic thời gian phức tạp, còn thích ứng được với sự khác biệt UI của hệ thống xe từ các thương hiệu khác nhau. Đây chính là điều mà hệ thống xe truyền thống khó lòng làm được.

Trong lĩnh vực kiểm thử phần mềm, GUI Agent mang lại sự thay đổi mang tính cách mạng. Kiểm thử tự động truyền thống phụ thuộc vào các công cụ như Selenium, mỗi lần thay đổi UI đều phải cập nhật kịch bản kiểm thử, chi phí bảo trì cực cao. Còn GUI Agent có thể tự thích ứng với thay đổi UI — ngay cả khi vị trí nút được điều chỉnh, màu sắc thay đổi, Agent vẫn tìm được phần tử đúng qua nhận diện ngữ nghĩa. Nó còn có thể thực hiện kiểm thử hồi quy thị giác, tự động phát hiện bất thường UI, thậm chí chủ động tiến hành kiểm thử thăm dò, phát hiện những trường hợp biên mà kỹ sư kiểm thử con người có thể bỏ sót.

Kịch bản RPA cấp doanh nghiệp là một thị trường khổng lồ khác. RPA truyền thống không thể xử lý những hệ thống cũ kỹ không có API, nhưng GUI Agent thì có thể. Trích xuất dữ liệu từ Excel, điền vào hệ thống ERP, gửi email thông báo — toàn bộ quy trình làm việc xuyên hệ thống có thể được tự động hóa hoàn toàn. Đối với những hệ thống di sản đã chạy hai ba chục năm, không có bất kỳ giao diện hiện đại nào, GUI Agent cuối cùng đã mang lại khả năng tự động hóa.

Trong đời sống cá nhân, GUI Agent có thể trở thành một trợ lý thông minh thực thụ. Đăng nội dung định giờ lên nhiều nền tảng mạng xã hội, mỗi sáng tự động tổng hợp tin tức, thời tiết, lịch trình, ghi lại dữ liệu vận động và thói quen ăn uống — những công việc số lặp đi lặp lại này đều có thể giao cho Agent hoàn thành. Còn đối với người dùng khiếm thị, khuyết tật vận động, GUI Agent càng mở ra cánh cửa đến một thế giới mới. Điều khiển điện thoại hoàn toàn bằng giọng nói, đọc thông minh nội dung màn hình, chuyển thao tác phức tạp thành chỉ thị đơn giản — những chức năng này đang giúp công nghệ thực sự đến được với mọi người.

#### 1.4.2 Ba giới hạn lớn của kỹ thuật hiện tại

Nhưng chúng ta cũng phải tỉnh táo nhận thức rằng, kỹ thuật GUI Agent vẫn đang ở giai đoạn phát triển sơ khai, đối mặt với một số thách thức thực chất.

Đáng lo ngại nhất là rủi ro an toàn và ảo giác (hallucination). Vấn đề ảo giác của mô hình ngôn ngữ lớn ở GUI Agent có thể dẫn đến hậu quả nghiêm trọng. Người dùng yêu cầu "dọn dẹp màn hình nền", Agent có thể hiểu nhầm thành xóa toàn bộ tập tin; một con số sai trong thao tác chuyển khoản có thể gây tổn thất kinh tế. Các phương án giảm thiểu hiện tại bao gồm: bắt buộc yêu cầu xác nhận thủ công đối với thao tác rủi ro cao, ghi lại nhật ký thao tác chi tiết và hỗ trợ hoàn tác (rollback), cũng như kiểm thử đầy đủ trong môi trường sandbox. Nhưng đây đều là giải pháp tình thế, việc giải quyết triệt để vấn đề ảo giác của mô hình vẫn cần thời gian.

Vấn đề chi phí và hiệu suất cũng không thể xem nhẹ. Mỗi bước thao tác đều cần gọi mô hình lớn để suy luận, nếu dùng API đám mây, chi phí sẽ tăng tuyến tính theo số lần gọi. Một nhiệm vụ phức tạp có thể cần hàng chục lần lặp, tổng thời gian khá dài. Triển khai mô hình nhỏ tại chỗ có thể giảm chi phí, nhưng độ chính xác sẽ giảm đi ít nhiều. Cache thao tác, nhận dạng mẫu, kiến trúc lai (nhiệm vụ đơn giản dùng RPA, nhiệm vụ phức tạp dùng AI) là các hướng đang được thăm dò, nhưng vẫn chưa hình thành thông lệ tốt nhất trưởng thành.

Cuối cùng là nút thắt độ chính xác. Ngay cả hệ thống tốt nhất, tỷ lệ thành công trong kịch bản thực tế cũng chỉ đạt 40-50%. Định vị phần tử của giao diện phức tạp, xử lý nội dung động (quảng cáo, cửa sổ bật lên), sự tích lũy lỗi của các nhiệm vụ chuỗi dài — đây đều là những khó khăn kỹ thuật rất thực. Các hướng đột phá bao gồm mô hình thị giác lớn mạnh hơn, tối ưu chiến lược thao tác qua học tăng cường (reinforcement learning), cùng thiết kế cộng tác "con người trong vòng lặp" (Human-in-the-loop). Nhưng để nâng từ 50% lên mức thương mại hóa khả dụng 90%, có lẽ vẫn cần một khoảng thời gian nữa.

---

## Phần thứ hai: Hướng dẫn thực hành GUI Agent

Sau khi học lý thuyết, hãy cùng thực sự nắm vững cách sử dụng và triển khai GUI Agent qua hai trường hợp (case) thực hành có độ khó tăng dần.

### Thực hành một: Trải nghiệm trực tuyến Mobile-Agent (không rào cản)

#### 2.1.1 Truy cập Demo trực tuyến

Mobile-Agent-v3 không chỉ hỗ trợ điện thoại, mà còn thao tác được cả máy tính. Như thể hiện ở Hình 3, trong trang Demo của ModelScope, ta chuyển lựa chọn thiết bị ở góc trên bên trái sang "máy tính", là có thể vào môi trường trải nghiệm PC Agent.

**Lựa chọn một: ModelScope Demo** (khuyến nghị)
Liên kết: https://modelscope.cn/studios/wangjunyang/Mobile-Agent-v3

**Lựa chọn hai: Aliyun Bailian (Bách Luyện Alibaba Cloud)**
Liên kết: https://bailian.console.aliyun.com/next?tab=demohouse#/experience/adk-computer-use/pc

Cả hai nền tảng này đều cung cấp **môi trường điện thoại đám mây/máy tính đám mây**, không cần triển khai tại chỗ vẫn trải nghiệm được đầy đủ chức năng.



### 2.1.2 Giới thiệu các chức năng giao diện

Sau khi vào trang, bạn sẽ thấy giao diện thao tác như Hình 3. Để đảm bảo trải nghiệm nhất quán, hãy nhất định thực hiện các **thiết lập then chốt** sau:

1. **Chọn thiết bị**: Trong menu thả xuống ở góc trên bên trái, xác nhận chọn **"máy tính"** (chứ không phải điện thoại).
2. **Xem trước màn hình nền**: Cửa sổ bên phải hiển thị màn hình Windows 10 mà đám mây phân bổ cho bạn, đã cài sẵn các phần mềm cơ bản như Office, trình duyệt.
3. **Khu tương tác**: Góc dưới bên trái là khu nhập chỉ thị, quá trình tư duy (Thinking Process) và các bước thao tác của Agent sẽ hiển thị trong hộp thoại phía trên.

<div align="center">
  <img src="./images/Extra06-figures/image3.png" alt="ModelScope Demo界面" width="90%"/>
  <p>Hình 3 Giải thích giao diện Demo trực tuyến Mobile-Agent-v3</p>
</div>

Trong giao diện này, bạn có thể trực tiếp điều khiển Agent thực hiện các thao tác văn phòng, chỉ có điều thời gian sử dụng hiện tại còn hạn chế.



### 2.1.3 Diễn tập nhiệm vụ điển hình

Dựa trên các năng lực định sẵn mà giao diện cung cấp, khuyến nghị người mới bắt đầu từ hai loại nhiệm vụ sau:

- **Điều khiển cấp hệ thống**: Thử để Agent chỉnh sửa thiết lập hệ thống.
  - *Ví dụ chỉ thị*: "Đặt màu hệ thống sang **chế độ sáng (light mode)**."
  - *Điểm quan sát*: Agent có thể mở "menu Start -> Cài đặt -> Cá nhân hóa" như con người hay không.
- **Văn phòng xuyên ứng dụng**: Thử để Agent liên kết trình duyệt và phần mềm văn phòng.
  - *Ví dụ chỉ thị*: "Tìm giá cổ phiếu Alibaba trong trình duyệt Edge, sau đó tạo một bảng tính mới trong WPS, điền tên công ty và giá cổ phiếu hiện tại."
  - *Điểm quan sát*: Agent có xử lý chính xác việc chuyển đổi ngữ cảnh xuyên phần mềm từ "tìm thông tin" sang "nhập thông tin" hay không.



### 2.1.4 Kỹ thuật prompt: Cách điều khiển PC Agent

Trong kịch bản GUI, một Prompt chất lượng cao là chìa khóa của thành công. Kết hợp với kịch bản văn phòng nêu trên, chúng tôi đúc kết ba kỹ thuật cốt lõi:

1. **Xác định rõ ranh giới ứng dụng (Explicit Context)**
   - Tránh chỉ thị chung chung như "viết một đoạn giới thiệu".
   - **Cách viết khuyến nghị**: "Viết một đoạn giới thiệu trong **tài liệu WPS Office**……"
   - *Phân tích*: Chỉ rõ tên phần mềm (App Name) có thể giảm thời gian Agent phải tìm kiếm công cụ.
2. **Phân rã theo chuỗi bước (Chain of Steps)**
   - Đừng cố nhồi toàn bộ logic phức tạp vào một câu.
   - **Cách viết khuyến nghị**: "Bước một, mở Edge tìm kiếm……; bước hai, sau khi xác nhận trang web đã tải xong, trích xuất dữ liệu……; bước ba, mở Excel dán vào."
   - *Phân tích*: Thao tác GUI có tính tuần tự nghiêm ngặt, chỉ thị chia bước có thể giảm đáng kể tỷ lệ lỗi thực thi.
3. **Mô tả thuộc tính thị giác (Visual Attributes)**
   - Agent thao tác bằng cách "nhìn" màn hình, dùng đặc trưng thị giác để mô tả sẽ hiệu quả hơn.
   - **Cách viết khuyến nghị**: "Nhấn **nút lưu màu xanh** ở góc trên bên phải" hoặc "Đổi màu chữ sang **màu đỏ**".





#### 2.1.5 Giá trị và giới hạn của trải nghiệm trực tuyến

Giá trị lớn nhất của Demo trực tuyến mà ModelScope cung cấp nằm ở **trải nghiệm không rào cản**. Bạn không cần cấu hình bất kỳ môi trường nào, không cần chuẩn bị điện thoại, thậm chí không cần tải xuống bất kỳ phần mềm nào, mà vẫn cảm nhận trực tiếp được sự kỳ diệu của GUI Agent. Điều này rất hữu ích cho việc nhanh chóng kiểm chứng ý tưởng, tìm hiểu ranh giới của công nghệ.

Nhưng môi trường trực tuyến cũng có giới hạn của nó. Trước hết là **vấn đề quyền riêng tư**, mọi thao tác đều diễn ra trên máy ảo đám mây, bạn không thể truy cập dữ liệu cá nhân thực. Tiếp đến là **giới hạn chức năng**, môi trường ảo chỉ cài sẵn một số APP thông dụng, không thể kiểm thử các kịch bản ứng dụng đặc thù. Cuối cùng là **sự khác biệt hiệu suất**, độ trễ suy luận trên đám mây sẽ cao hơn một chút so với triển khai tại chỗ.

Vì vậy, trải nghiệm trực tuyến phù hợp làm điểm khởi đầu để học và khám phá, nhưng nếu muốn ứng dụng GUI Agent trong kịch bản thực tế, bạn cần thử triển khai tại chỗ. Chính thức Mobile-Agent-v3 có cung cấp một [hướng dẫn](https://github.com/X-PLUG/MobileAgent/blob/main/Mobile-Agent-v3/README_zh.md), bạn có thể tự thử.

Tiếp theo, Thực hành hai sẽ đưa bạn dùng AutoGLM mà **Zhipu (Trí Phổ)** vừa mã nguồn mở gần đây để bước vào thế giới sâu hơn này.

---

### Thực hành hai: Triển khai tại chỗ AutoGLM và thực chiến trên điện thoại

Trải nghiệm trực tuyến giúp ta cảm nhận được năng lực của GUI Agent, nhưng sức mạnh thực sự nằm ở việc triển khai trên thiết bị của chính mình, điều khiển các ứng dụng thực. AutoGLM là một framework rất phù hợp để lập trình viên cá nhân nhập môn, kiến trúc của nó rõ ràng, tài liệu đầy đủ, quá trình triển khai tương đối đơn giản.

Mục tiêu của thực hành này là triển khai AutoGLM trên máy tính của bạn, kết nối điện thoại Android của bạn, rồi để AI giúp bạn hoàn thành một số nhiệm vụ thực — chẳng hạn tự động trả lời tin nhắn WeChat, hoặc định giờ làm mới một APP nào đó để lấy dữ liệu mới nhất.

#### 2.2.1 Chuẩn bị môi trường: Bạn cần những gì

Việc triển khai Open-AutoGLM cần hai thiết bị cốt lõi: một máy tính có thể chạy Python, và một điện thoại Android. Cấu hình máy tính không cần quá cao, vì AutoGLM hỗ trợ gọi API đám mây, không nhất thiết phải chạy mô hình lớn tại chỗ. Nếu bạn định dùng API đám mây (như GLM-4V của Zhipu), một chiếc laptop bình thường là đủ. Nhưng nếu bạn muốn trải nghiệm phương án hoàn toàn tại chỗ, thì một GPU có ít nhất 8GB bộ nhớ đồ họa sẽ giúp trải nghiệm tốt hơn nhiều.

Về điện thoại, Android 7.0 trở lên đều được, không cần quyền Root. Người dùng iPhone tạm thời chưa dùng được, vì tính đóng của iOS khiến phương án gỡ lỗi ADB không thể áp dụng trực tiếp.

Về môi trường phần mềm, bạn cần cài Python 3.10 trở lên, cùng công cụ ADB (Android Debug Bridge). ADB là cây cầu kết nối máy tính và điện thoại, mọi thao tác chụp màn hình, nhấn, vuốt đều phải thực hiện qua nó.

**Cài đặt công cụ ADB (macOS / Linux):** Tùy theo hệ thống của bạn, thực thi lệnh sau trong terminal:

```bash
# macOS 使用 Homebrew
brew install android-platform-tools

# Linux (Ubuntu/Debian)
sudo apt install android-tools-adb
```

Người dùng Windows thường có thể tải trực tiếp gói nén Platform Tools và cấu hình biến môi trường. [Tham khảo](https://blog.csdn.net/x2584179909/article/details/108319973)



#### 2.2.2 Bước một: Cài đặt Open-AutoGLM

Nếu bạn có **Claude Code**, bạn có thể cấu hình [GLM Coding Plan](https://bigmodel.cn/glm-coding) rồi nhập prompt sau để triển khai nhanh:

```
访问文档，为我安装 AutoGLM
https://raw.githubusercontent.com/zai-org/Open-AutoGLM/refs/heads/main/README.md
```

Nếu không có CLI tương tự, hãy làm theo các bước thủ công dưới đây:



Mở terminal dòng lệnh, trước tiên clone kho mã của Open-AutoGLM:

```bash
git clone https://github.com/zai-org/Open-AutoGLM.git
cd Open-AutoGLM
```

Tiếp theo cài đặt các phụ thuộc. Ngoài các gói phụ thuộc cơ bản, **nhất định phải thực thi lệnh cài đặt của dự án** để đảm bảo mọi mô-đun được gọi đúng cách:

```bash
# 1. 安装基础依赖
pip install -r requirements.txt

# 2. 以编辑模式安装项目本身 (关键步骤)
pip install -e .

# 3. (可选) 如果你是开发者，需要额外安装开发依赖
pip install -e ".[dev]"
```

Quá trình này thường mất vài phút, tùy theo tốc độ mạng của bạn. Sau khi cài xong, bạn cần cấu hình khóa API. Nếu dùng API GLM-4V của Zhipu, trước tiên đến nền tảng mở Zhipu đăng ký tài khoản và lấy API Key, rồi tạo một tập tin `.env` trong thư mục gốc của dự án:

```bash
# .env 文件内容
GLM_API_KEY=your_api_key_here
```



[AutoGLM-Phone-9B · Kho mô hình](https://modelscope.cn/models/ZhipuAI/AutoGLM-Phone-9B)



#### 2.2.3 Bước hai: Kết nối điện thoại Android của bạn

Bây giờ đến bước then chốt: giúp máy tính có thể "nhìn thấy" và "điều khiển" điện thoại của bạn. Việc này cần ba bước nhỏ: bật chế độ nhà phát triển, bật gỡ lỗi USB, và **cài đặt ADB Keyboard**.

**1. Kích hoạt chế độ nhà phát triển & gỡ lỗi USB** Trên điện thoại Android, vào "Cài đặt" → "Giới thiệu điện thoại", tìm "Số phiên bản", **nhấn liên tục 7 lần** (hoặc đến khi xuất hiện lời nhắc), bạn sẽ thấy thông báo "Bạn đã ở chế độ nhà phát triển". Quay lại giao diện Cài đặt chính, vào "Tùy chọn nhà phát triển", tìm "Gỡ lỗi USB" và **bật** lên.

**2. Cài đặt ADB Keyboard (bắt buộc)** Để AI có thể nhập chữ trên điện thoại, chúng ta cần cài đặt bàn phím ADB chuyên dụng.

- Địa chỉ tải: https://github.com/senzhk/ADBKeyBoard/raw/master/ADBKeyboard.apk

Sau khi cài, nhớ vào "phương thức nhập" trong cài đặt điện thoại, kích hoạt và chuyển sang **ADB Keyboard**.

**3. Xác minh kết nối** Dùng cáp dữ liệu USB kết nối điện thoại với máy tính (nhấn "Cho phép" khi hộp cấp quyền hiện lên trên điện thoại). Nhập vào terminal máy tính:

Bash

```
adb devices
```

Nếu mọi thứ bình thường, bạn sẽ thấy số sê-ri thiết bị:

```
List of devices attached
ABC12345    device
```

Nếu hiển thị `device`, chúc mừng bạn, kết nối phần cứng đã thông! Nếu hiển thị `unauthorized`, hãy kiểm tra xem màn hình điện thoại có hiện hộp xác nhận cấp quyền hay không.



Đối với người dùng Windows, có thể còn cần cài trình điều khiển (driver) cho điện thoại. Hầu hết điện thoại các thương hiệu (như Xiaomi, Huawei, OPPO) đều tự động cài driver khi kết nối máy tính, nhưng nếu gặp vấn đề, bạn có thể lên trang chủ tải driver USB tương ứng.

<div align="center">
  <img src="./images/Extra06-figures/image4.png" alt="手机ADB连接配置步骤" width="90%"/>
  <p>Hình 4 Quy trình cấu hình kết nối ADB đầy đủ cho điện thoại Android</p>
</div>


#### 2.2.4 Bước ba: Chạy nhiệm vụ đầu tiên của bạn

Sau khi kết nối thành công, hãy cùng thực thi một nhiệm vụ đơn giản nhưng thiết thực.

Có hai cách kết nối gọi API trực tiếp:

**1. Zhipu BigModel**

- Tài liệu: https://docs.bigmodel.cn/cn/api/introduction
- `--base-url`: `https://open.bigmodel.cn/api/paas/v4`
- `--model`: `autoglm-phone`
- `--apikey`: Đăng ký API Key của bạn trên nền tảng Zhipu

**2. ModelScope (Cộng đồng Moda)**

- Tài liệu: https://modelscope.cn/models/ZhipuAI/AutoGLM-Phone-9B
- `--base-url`: `https://api-inference.modelscope.cn/v1`
- `--model`: `ZhipuAI/AutoGLM-Phone-9B`
- `--apikey`: Đăng ký API Key của bạn trên nền tảng ModelScope

Readme chính thức có cung cấp một giao diện dòng lệnh, bạn có thể nhập trực tiếp:

```bash
# 使用智谱 BigModel
python main.py --base-url https://open.bigmodel.cn/api/paas/v4 --model "autoglm-phone" --apikey "your-bigmodel-api-key" "打开美团搜索附近的火锅店"

# 使用 ModelScope
python main.py --base-url https://api-inference.modelscope.cn/v1 --model "ZhipuAI/AutoGLM-Phone-9B" --apikey "your-modelscope-api-key" "打开美团搜索附近的火锅店"
```

Sau khi thực thi lệnh này, AutoGLM sẽ khởi động quy trình suy luận. Bạn sẽ thấy nhật ký được xuất ra theo thời gian thực trên terminal, đồng thời màn hình điện thoại sẽ bắt đầu tự động thao tác. Toàn bộ quá trình đại khái như sau:

Trước tiên, AutoGLM sẽ chụp ảnh màn hình hiện tại qua ADB, gửi hình ảnh cho mô hình phân tích. Mô hình sẽ nhận diện toàn bộ biểu tượng APP trên màn hình, và định vị vị trí của "Meituan" ở cấp độ pixel. Sau đó AutoGLM gửi lệnh nhấn, đánh thức ứng dụng qua `adb shell input tap x y`.

Sau khi chờ Meituan khởi động, AutoGLM lại chụp màn hình lần nữa. Lần này mục tiêu của nó là tìm "thanh tìm kiếm" ở phía trên trang chủ. Sau khi nhận diện và nhấn vào ô tìm kiếm, **nó sẽ gọi ADB Keyboard mà chúng ta đã cài ở giai đoạn chuẩn bị môi trường**, nhập chuỗi ký tự "lẩu gần đây" vào, cuối cùng tự động nhấn nút tìm kiếm.

Toàn bộ quá trình thường mất 15-20 giây (nhiệm vụ tìm kiếm có hơi nhiều bước một chút), thời gian cụ thể tùy thuộc vào tốc độ suy luận của mô hình và độ trễ mạng. Nếu bạn dùng API đám mây, thời gian "tư duy" của mỗi bước khoảng 2-3 giây. Nếu là mô hình triển khai tại chỗ, một GPU cấu hình tốt có thể nén thời gian mỗi bước xuống còn khoảng 1 giây.



---

## Tổng kết và triển vọng

Qua hai thực hành có mức độ tăng dần này, chúng ta đã trải nghiệm trọn vẹn toàn bộ quá trình của GUI Agent từ trình diễn trực tuyến đến triển khai tại chỗ. Demo trực tuyến của Mobile-Agent giúp ta nhanh chóng hiểu được các khả năng của công nghệ, thực chiến trên điện thoại của AutoGLM giúp ta nắm vững kỹ năng triển khai thực tế, còn phương án đầu cuối (on-device) của GLM-ZERO thì cho thấy tương lai của bảo vệ quyền riêng tư và ứng dụng ngoại tuyến.

Kỹ thuật GUI Agent vẫn đang tiến hóa nhanh chóng. Hệ thống hiện tại tuy đã có thể xử lý phần lớn nhiệm vụ hằng ngày, nhưng về độ chính xác, tốc độ suy luận và kiểm soát chi phí vẫn còn nhiều không gian để cải thiện. Cùng với sự tiến bộ liên tục của mô hình thị giác lớn, cũng như sự phát triển của chip suy luận đầu cuối, chúng ta có lý do để tin rằng GUI Agent sẽ trở thành một mô hình (paradigm) quan trọng của tương tác người-máy trong tương lai.

Có lẽ trong tương lai không xa, mỗi người đều sẽ sở hữu một trợ lý số thực sự thông minh, nó không chỉ hiểu được ý định của bạn, mà còn vượt qua các ứng dụng và nền tảng khác nhau, giúp bạn hoàn thành đủ loại công việc lặp đi lặp lại. Khi đó, những kịch bản tự động hóa mà hôm nay chúng ta phải vất vả viết ra, đều sẽ trở thành một câu chỉ thị ngôn ngữ tự nhiên đơn giản.

Tương lai này, thực ra đã đang trên đường đến rồi.

---

## Tài liệu tham khảo

1. Bài báo Mobile-Agent-v3: https://arxiv.org/abs/2508.15144
2. Open-AutoGLM GitHub: https://github.com/zai-org/Open-AutoGLM
3. Dự án UI-TARS: https://github.com/bytedance/UI-TARS
