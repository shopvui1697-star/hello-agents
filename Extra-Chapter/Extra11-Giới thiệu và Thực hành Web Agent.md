# Giới thiệu và Thực hành Web Agent — Dạy AI biết "lên mạng"

## Lời mở đầu: Khi AI học được cách duyệt web

Hãy tưởng tượng một tình huống như thế này: bạn nói với AI "hãy giúp tôi tìm trên các trang du lịch phổ biến ba chuyến bay thẳng từ Bắc Kinh đi Singapore vào thứ Ba tuần sau với giá dưới 3000 tệ, so sánh rồi dùng thẻ tín dụng tôi đã lưu để đặt chuyến bay có lợi nhất" — và rồi AI thực sự mở một trình duyệt, tự động điều hướng qua quy trình đặt vé, xử lý đủ loại pop-up và trang xác minh trên đường đi, điền thông tin hành khách, và cuối cùng trao mã xác nhận đơn hàng vào tay bạn.

Đây không phải là khoa học viễn tưởng, mà là **Web Agent (tác nhân thông minh trên web)** đang dần trở thành hiện thực. Nếu như GUI Agent được giới thiệu ở Extra06 đã dạy AI cách thao tác các ứng dụng trên điện thoại và máy tính để bàn, thì chương này sẽ giới thiệu người họ hàng gần của nó trong thế giới web — một tác nhân thông minh (Agent) lấy trình duyệt (browser) làm bề mặt hành động chính.

Web xứng đáng có riêng một chương độc lập vì hai lý do. Thứ nhất, **phần lớn dữ liệu và luồng công việc có giá trị nhất trên thế giới đều chạy trên web** — nhiều hơn hẳn so với App di động hay phần mềm máy tính. Thứ hai, **Web là một bài toán kỹ thuật độc lập**: những thách thức về cảm nhận, hành động và độ tin cậy của nó về bản chất khác với GUI Agent trên di động/máy tính. Bạn không thể cứ cắm Mobile-Agent-v3 vào Chrome rồi coi nó là Web Agent — con đường đó không đi tới đâu.

Lời hứa của chương này là, sau khi đọc xong bạn sẽ:

- Hiểu sự khác biệt bản chất giữa Web Agent với RPA truyền thống và GUI Agent phổ thông
- Nắm được ba chiến lược cảm nhận chủ đạo (DOM, thị giác, hỗn hợp), và hiểu vì sao các hệ thống cấp sản xuất phổ biến đều đi theo hướng hỗn hợp
- Thông qua việc gọi SDK, cũng như tích hợp như một công cụ (tool) vào framework (khung phần mềm) **HelloAgents** đã xây dựng ở chương 7, tự tay sử dụng một dịch vụ Web Agent cấp sản xuất — **TinyFish**
- Học được cách chẩn đoán và né tránh các cơ chế chống bot (anti-crawler) thường gặp
- Biết khi nào **không nên** sử dụng Web Agent kiểu dịch vụ được lưu trữ (托管型)

---

## Phần một: Giới thiệu kiến thức về công nghệ Web Agent

### 1.1 Web Agent là gì?

**Web Agent** là một loại tác nhân thông minh tự chủ (autonomous agent) lấy trình duyệt web làm bề mặt hành động chính. Nó cảm nhận trang web thông qua một tổ hợp nào đó của DOM, cây khả năng truy cập (accessibility tree) và ảnh chụp màn hình, dùng mô hình ngôn ngữ lớn (LLM) để suy luận bước tiếp theo nên làm gì, rồi thực thi hành động trong một phiên trình duyệt thực — nhấp chuột, nhập liệu, cuộn trang, chuyển trang.

Từ khóa ở đây là **tự chủ**. Một script crawler viết cứng bằng Playwright không phải là Web Agent, nó chỉ là một đoạn chương trình mong manh, chỉ cần trang web đích thay đổi vị trí nút một chút là nó sập ngay. Còn Web Agent thì quyết định động hành động tiếp theo tại thời điểm chạy (runtime) dựa trên những gì nó "nhìn thấy".

#### 1.1.1 Web Agent vs RPA vs GUI Agent

Mạch huyết thống công nghệ này rất quan trọng. Chúng ta đặt cả ba cạnh nhau để so sánh:

| Chiều so sánh | RPA truyền thống (Selenium, UiPath) | GUI Agent (Mobile-Agent, AutoGLM) | Web Agent (Browser-Use, TinyFish) |
| --- | --- | --- | --- |
| **Bề mặt chính** | Web hoặc máy tính để bàn | Di động + máy tính, chủ yếu dựa vào ảnh chụp | Trình duyệt web |
| **Cách cảm nhận** | Bộ chọn DOM (XPath, CSS) | Thị giác (VLM phân tích ảnh chụp) | DOM + cây khả năng truy cập + thị giác (hỗn hợp) |
| **Cơ chế hành động** | Script cố định | Nhấp chuột dựa trên tọa độ | Hỗn hợp giữa bộ chọn, tọa độ và mục tiêu ngữ nghĩa |
| **Có thích ứng được thay đổi UI không** | Không — hỏng ngay lập tức | Được — thị giác có độ đàn hồi ngữ nghĩa | Được — nhiều tín hiệu neo (anchor) |
| **Khả năng đa nền tảng** | Hạn chế | Đa nền tảng một cách tự nhiên | Bản thân Web vốn đã đa nền tảng |
| **Xử lý xác thực / phiên** | Thủ công | Hạn chế | Công dân hạng nhất (first-class citizen) |
| **Ý thức về chống bot** | Gần như không có | Không liên quan | Cực kỳ quan trọng |

Hai điểm mấu chốt nhất:

1. **Web Agent không phải là "GUI Agent bị giới hạn trên Chrome"**. Nó tận dụng thông tin có cấu trúc từ DOM, cây khả năng truy cập và tầng mạng (network layer) — những thông tin này hoặc không tồn tại trong môi trường di động/máy tính, hoặc GUI Agent chọn không dùng. Web Agent cấp sản xuất nhất định là hỗn hợp: thị giác phụ trách hiểu bố cục, DOM/AX-tree phụ trách định vị chính xác, chuyển đổi tùy theo tình huống cụ thể của hành động.
2. **Web có những đối thủ riêng của nó**. Cloudflare, DataDome, PerimeterX, nhận dạng vân tay (fingerprint), phân tích đặc trưng sinh trắc học hành vi — những thứ này đều không tồn tại trên GUI Agent di động/máy tính. Một Web Agent không tính đến chống bot thì trong môi trường sản xuất chính là một Web Agent trả về kết quả rỗng.

#### 1.1.2 Vì sao Web Agent bùng nổ đột ngột vào giai đoạn 2024–2026?

Ba lực lượng hội tụ vào cùng một thời điểm:

1. **Mô hình lớn đa phương thức (multimodal) có thể đọc hiểu đồng thời cả ảnh chụp và DOM**. GPT-4o, Claude Sonnet 4, Gemini 2.5, Qwen-VL — khả năng định vị thị giác đối với các phần tử UI của chúng đều đã đủ mạnh, có thể điều khiển trình duyệt mà không cần huấn luyện thị giác chuyên biệt.
2. **Hạ tầng trình duyệt không đầu (headless browser) đã trưởng thành**. Playwright và Chrome DevTools Protocol (CDP) cho phép chúng ta khởi động một cách rẻ tiền một phiên Chromium thực ở bất kỳ đâu trên thế giới, kèm theo các bản vá tàng hình (stealth), định tuyến proxy và điều khiển từ xa.
3. **Nỗi đau thực tế đủ lớn**. Một nửa dữ liệu trên thế giới nằm sau những trang web không có API — giá cả, tồn kho, hồ sơ công khai của chính phủ, kho lưu trữ tin tức, đủ loại dashboard nội bộ mà các nhà cung cấp triển khai cho doanh nghiệp. Giá trị kinh tế của việc tự động hóa các luồng công việc này là khổng lồ, và chỉ có Web Agent mới lấp được khoảng trống đó.

Chúng ta còn vừa đúng lúc bắt kịp thời điểm các phòng thí nghiệm AI lớn bắt đầu trực tiếp sản phẩm hóa năng lực này — Computer Use của Anthropic, Operator của OpenAI, Project Mariner của Google. "Web Agent" đang nhanh chóng trở thành năng lực nền tảng mà mỗi sản phẩm AI-native đều sẽ tích hợp.

### 1.2 Kiến trúc công nghệ cốt lõi

Giống như mọi GUI Agent, Web Agent cũng là một vòng lặp khép kín **cảm nhận → suy luận → hành động**. Nhưng mỗi tầng đều có những điểm tinh tế đặc thù của Web.

#### 1.2.1 Tầng cảm nhận: ba chiến lược, mỗi cái đều có điểm yếu chí mạng

Trong ngành có ba chiến lược cảm nhận chủ đạo, mỗi chiến lược đều có điểm yếu mà hai chiến lược còn lại vừa vặn bù đắp được:

**Chiến lược A — dựa trên DOM**: phân tích HTML và cây khả năng truy cập của trang. Vừa nhanh vừa chính xác, có thể trực tiếp cho bạn biết loại phần tử, nội dung văn bản, bộ chọn. **Tình huống thất bại**: trang là SPA render bằng Canvas (Google Docs, Figma, v.v.), hoặc trang cố tình làm rối DOM, hoặc nội dung có ý nghĩa được render vào Shadow DOM mà crawler không đọc được.

**Chiến lược B — dựa trên thị giác**: chụp màn hình → đưa vào mô hình lớn thị giác → mô hình nhận diện phần tử cần nhấp và xuất ra tọa độ. **Tình huống thất bại**: mô hình đọc sai chữ nhỏ, ảo giác về vị trí phần tử, trên trang 4K có những thứ nó không nhìn thấy được. Hơn nữa chi phí cao — mỗi bước đều phải gọi một lần suy luận VLM.

**Chiến lược C — hỗn hợp (kẻ chiến thắng trong môi trường sản xuất)**: dùng thị giác để hiểu bố cục và suy luận ngữ cảnh ("trong ba ô tìm kiếm này, cái nào mới là cái tôi cần?"), rồi dùng DOM hoặc cây khả năng truy cập để định vị chính xác ("đây là phần tử cụ thể cần nhấp"). Tất cả các Web Agent cấp sản xuất nghiêm túc — Operator, Computer Use, Browser-Use, TinyFish — cuối cùng đều đi tới hướng hỗn hợp.

Vì sao ngành lại hội tụ về điểm này? Dữ liệu kinh nghiệm lên tiếng: trên các luồng công việc sản xuất thường gặp, những hệ thống trưởng thành đã qua phương án hỗn hợp + trình duyệt tàng hình + gia cố chống bot đã đạt tỷ lệ thành công ổn định quanh mức 90%; còn phương án thuần thị giác, chưa gia cố thì trên các benchmark học thuật đối kháng kiểu WebArena thường chỉ đạt 30–40%. Lưu ý là hai con số này đo lường **những thứ khác nhau** — benchmark học thuật cố tình dựng lên các chuỗi thao tác dài, tình huống hiểm hóc; còn hệ thống sản xuất đã được tinh chỉnh có mục tiêu rất nhiều trên các luồng nghiệp vụ thực. Nhưng cả hai đường cong đều cho thấy cùng một điều: phương án hỗn hợp rõ ràng chiếm ưu thế về mặt kỹ thuật.

#### 1.2.2 Tầng suy luận: bài toán về ký ức, phản tư và trạng thái

Thách thức lớn nhất của Web ở tầng suy luận là **đột biến trạng thái (state mutation)**. Trang web sẽ thay đổi ngay trước mắt bạn:

- Cuộn vô hạn (infinite scroll) sẽ liên tục tải nội dung mới khi bạn cuộn xuống đáy
- Hai giây sau khi trang tải xong, một hộp thoại modal bật lên, che mất cái nút bạn muốn nhấp
- Sau khi nhấp "Thêm vào giỏ hàng", trang render lại, ảnh chụp nhanh DOM trước đó của bạn lập tức mất hiệu lực
- Luồng đăng nhập sẽ đi qua ba trang trung gian mới đưa bạn tới dashboard

Một Web Agent đạt chuẩn cần phải có:

- **Phân rã nhiệm vụ** — chia "giúp tôi tìm chuyến bay thứ Bảy giá dưới 3000" thành một loạt các bước cụ thể
- **Phản tư (reflection)** — nhận ra một hành động không tạo ra hiệu quả như mong đợi, và có thể khôi phục
- **Ký ức (memory)** — khi lật sang trang thứ hai vẫn nhớ nội dung kết quả tìm kiếm ở trang đầu
- **Theo dõi trạng thái** — phân biệt "nút biến mất vì tôi đã nhấp nó" và "nút biến mất vì trang bị sập"

Đây chính là sân khấu để **mô hình ReAct** của chương 4 tỏa sáng. Về bản chất Web Agent chính là một vòng lặp ReAct, trong đó không gian hành động là `{click, type, scroll, navigate, wait, extract}`, và quan sát (observation) là trạng thái trang tiếp theo.

#### 1.2.3 Tầng hành động: những điểm khó đặc thù trên Web

Không gian hành động thoạt nhìn khá giống GUI Agent di động — nhấp chuột, nhập liệu, cuộn trang. Nhưng tầng Web còn có thêm:

- **Trạng thái phiên (session state)**: cookies, localStorage, sessionStorage cần được duy trì xuyên suốt các hành động (hoặc cố tình xóa sạch)
- **Lịch sử điều hướng**: quay lại, tiến tới, tab mới, chuyển tab
- **Tải xuống và tải lên tệp**: không đi qua DOM
- **Duyệt qua iframe và Shadow DOM**: một trang web không phải là một tài liệu, mà là một cây các tài liệu
- **Các mối quan tâm ở tầng mạng**: sau khi bạn nhấp "Gửi", có thể phải đợi một XHR hoàn thành mới tính là "đã xong"

Web Agent chỉ dùng tọa độ sẽ va phải từng giới hạn kể trên. Đây chính là lý do phương án hỗn hợp trở thành chuẩn mực sản xuất.

### 1.3 Những bài toán khó chỉ Web mới có

Những vấn đề dưới đây, Extra06 không đề cập và cũng không thể đề cập, bởi vì chúng căn bản không tồn tại trên di động và máy tính:

**Cơ chế chống bot**. Cloudflare, DataDome, PerimeterX, Akamai Bot Manager. Chúng theo dõi vân tay TLS của bạn, chữ ký các đối tượng JavaScript của trình duyệt, mẫu di chuyển chuột, thời điểm thao tác. Một script Playwright ngây thơ sẽ bị chặn trong vài giây. Web Agent cấp sản xuất cần **trình duyệt tàng hình** (một Chromium đã được vá, trông không thể phân biệt với trình duyệt của người dùng thật), thường còn phải phối hợp với proxy IP dân cư (residential IP).

**Xác thực và duy trì phiên**. Luồng OAuth, xác thực hai lớp, cookie phiên, token "ghi nhớ đăng nhập", mã captcha khi đăng nhập. Một Web Agent không đăng nhập được thì chỉ có thể nhìn thấy các trang web công khai. Phương án có kho lưu trữ thông tin đăng nhập được mã hóa (vault) là mô hình mới nổi.

**Nội dung động và điều kiện tranh chấp (race condition)**. Cuộn vô hạn, ảnh tải lười (lazy loading), modal xuất hiện sau 800ms, animation chặn thao tác nhấp. Tác nhân phải biết khi nào trang được tính là "sẵn sàng", mà "sẵn sàng" thì không có định nghĩa rõ ràng.

**SPA nặng JS**. Khi bạn truy cập một trang React hay Vue hiện đại, HTML ban đầu về cơ bản là rỗng — DOM thực sự do JavaScript render ra sau ba giây. Crawler ngây thơ không nhìn thấy nội dung nào cả.

**Mạng, địa lý và giới hạn tần suất (rate limit)**. Một số nội dung bị giới hạn theo khu vực địa lý, một số API giới hạn tần suất theo IP, một số trang sẽ âm thầm hạ cấp phản hồi khi phát hiện "quá nhiều" yêu cầu từ cùng một nguồn.

**Chi phí và độ trễ**. Mỗi bước hành động là một lần gọi LLM + một lượt đi-về của trình duyệt. Một nhiệm vụ tự động hóa 10 bước có thể mất 30–60 giây, chỉ riêng chi phí token của LLM đã là 0.10–1.00 đô la. Đây chính là lý do ngành ra sức theo đuổi những mô hình nhỏ hơn, nhanh hơn, được tinh chỉnh chuyên cho Web.

### 1.4 Bức tranh toàn cảnh năm 2026: so sánh bốn nhóm người chơi

Hệ sinh thái Web Agent năm 2026 đại khái có thể chia thành bốn nhóm, mỗi nhóm tồn tại đều tương ứng với một sự đánh đổi thực tế:

| Nhóm | Dự án tiêu biểu | Ưu điểm | Nhược điểm |
| --- | --- | --- | --- |
| **Tự động hóa trình duyệt thô** | Playwright, Puppeteer, Selenium | Nhanh, có thể lặp lại, miễn phí, kiểm soát hoàn toàn | Không có AI, mong manh, mỗi lần UI thay đổi đều phải sửa script |
| **Web Agent AI mã nguồn mở** | Browser-Use, Skyvern, WebVoyager, AgentE | Miễn phí, có thể độ (mod) lại, hoàn toàn minh bạch | Tự triển khai, tự xử lý chống bot, proxy, hạ tầng, khả năng quan sát (observability) |
| **API computer-use** | Anthropic Computer Use, OpenAI Operator, Google Project Mariner | Năng lực suy luận tổng quát mạnh, mô hình tiên phong | Đắt, UX có lập trường riêng, không có chống bot hay định tuyến địa lý tích hợp sẵn, thường lấy máy tính để bàn làm chính chứ không thuần Web |
| **API tự động hóa Web kiểu dịch vụ** | **TinyFish**, Browserbase, Apify, Bright Data | Trình duyệt tàng hình + proxy + vòng lặp tác nhân đóng gói trọn gói, vài phút là chạy được; tự có khả năng quan sát | Phụ thuộc nhà cung cấp, ít quyền kiểm soát vòng lặp tác nhân bên trong hơn, tính phí theo nhiệm vụ |

Đánh giá thành thật: mỗi nhóm thắng ở một trục khác nhau. Bạn là nhà nghiên cứu, muốn nghiên cứu hành vi của Agent? Dùng mã nguồn mở. Bạn là lập trình viên cá nhân, cào một trang nhỏ không có chống bot? Playwright thuần là đủ. Bạn đang làm tính năng cho một sản phẩm, cần "dùng ngay ra khỏi hộp" trên các trang trong thế giới thực (bao gồm cả chống bot)? API dịch vụ có thể tiết kiệm cho bạn vài tuần.

Ở phần hai chúng ta sẽ dùng TinyFish để làm bài thực hành — thiết kế API của nó là sạch sẽ nhất trong nhóm này, và nó phơi bày rõ ràng vòng lặp tác nhân, tiện cho việc học.

### 1.5 Các kịch bản ứng dụng trong thế giới thực

Dưới đây là danh sách (chưa đầy đủ) những nơi Web Agent đã được đưa vào môi trường sản xuất:

- **Giám sát thương mại điện tử**: giá cả, tồn kho, sản phẩm đối thủ, tổng hợp đánh giá
- **Làm giàu dữ liệu B2B và tạo khách hàng tiềm năng (lead)**: thu thập thông tin từ các trang kiểu LinkedIn, xây dựng cơ sở dữ liệu công ty
- **RPA nội bộ cho các hệ thống Web cũ (legacy)**: hàng ngàn công ty vẫn chạy nghiệp vụ cốt lõi trên các hệ thống thuần UI không có API, Web Agent cuối cùng đã cho phép tự động hóa chúng
- **Đảm bảo chất lượng và kiểm thử tự động**: kiểm thử hồi quy (regression testing), kiểm toán khả năng truy cập, tuân thủ nội dung
- **Tác nhân nghiên cứu (chủ đề chương 14)**: một tác nhân nghiên cứu không thể duyệt web thì mù mất một nửa, Web Agent là backend tự nhiên cho các hệ thống kiểu DeepResearch
- **Luồng công việc du lịch và đặt chỗ (chủ đề chương 13)**: trợ lý du lịch sau khi "thực sự có thể hoàn tất việc đặt chỗ" thay vì "chỉ có thể đưa ra gợi ý", giá trị sử dụng sẽ tăng gấp 10 lần
- **Tự động hóa cá nhân**: theo dõi một tin tuyển dụng, giám sát hàng bổ sung, theo dõi các thông báo công khai của chính phủ

### 1.6 Những giới hạn thực tế của công nghệ hiện tại

Xin hãy thành thật với độc giả của bạn. Web Agent vừa gây hào hứng, vừa có ranh giới của nó:

- **Tỷ lệ thành công: benchmark vs sản xuất là hai đường cong**. Web Agent cấp sản xuất (phương án hỗn hợp + tàng hình + gia cố chống bot) trên các luồng nghiệp vụ thường gặp đã có thể ổn định quanh mức **90%**; nhưng trên các benchmark học thuật đối kháng được dựng có chủ đích như WebArena, VisualWebArena, ngay cả hệ thống tốt nhất cũng vẫn đang không ngừng đuổi theo. Nói cách khác: những gì có thể làm được trong vòng lặp kỹ thuật khép kín thì nhiều hơn nhiều so với những gì đánh giá học thuật cho thấy — nhưng nhiệm vụ càng hiểm hóc, chuỗi thao tác càng dài thì khoảng cách càng lớn.
- **Chống bot là một trò mèo vờn chuột luôn dịch chuyển**. Phương án hữu dụng tháng trước tháng này có thể đã bị phát hiện. Cuộc đối kháng của các nhà cung cấp không bao giờ ngừng.
- **Chi phí là có thật**. API dịch vụ trung bình mỗi nhiệm vụ 0.10–1.00 đô la. Tự triển khai thì tiết kiệm tiền, đốt thời gian kỹ thuật.
- **Captcha là trần cứng**. Nhà cung cấp có quảng cáo thế nào đi nữa, Web Agent hiện đại **vẫn chưa thể giải được reCAPTCHA hoặc hCaptcha một cách đáng tin cậy**. Cách làm thực dụng là thiết kế nhiệm vụ né tránh nó, chứ không phải cứng đầu chống chọi.
- **Cái giá của ảo giác là tiền thật**. Một trường biểu mẫu điền sai có thể khiến bưu kiện được gửi tới sai địa chỉ. Các hành động rủi ro cao bắt buộc phải có điểm kiểm tra có con người trong vòng lặp (human-in-the-loop).

---

## Phần hai: Hướng dẫn thực hành

Chúng ta đi từ nông đến sâu theo ba bước:

1. **Bắt đầu siêu tốc** — chạy thông lệnh gọi TinyFish đầu tiên trong vòng 5 phút, không framework, không môi trường
2. **Quan sát tác nhân làm việc** — dùng URL phát trực tiếp (live) để trực quan hóa gỡ lỗi mỗi lần chạy
3. **Tích hợp vào HelloAgents** — đóng gói TinyFish thành một `Tool`, tích hợp vào tác nhân ReAct trong framework của chương 7

### 2.1 Bắt đầu siêu tốc: Web Agent đầu tiên của bạn trong vòng 5 phút

TinyFish là một API Web Agent kiểu dịch vụ được lưu trữ. Bạn đưa cho nó một URL và một đoạn **mục tiêu (goal)** bằng ngôn ngữ tự nhiên, nó trả về JSON có cấu trúc.

#### Bước 1: Lấy API key

1. Đăng ký tài khoản tại `agent.tinyfish.ai`
2. Vào trang API Keys, nhấp **Create API Key**
3. Sao chép key — nó chỉ hiển thị một lần duy nhất
4. Viết vào shell của bạn:

```bash
export TINYFISH_API_KEY="sk-tinyfish-..."
```

#### Bước 2: Cài đặt SDK

```bash
# Python
pip install tinyfish

# hoặc TypeScript
npm install @tiny-fish/sdk
```

#### Bước 3: Nhiệm vụ tự động hóa đầu tiên của bạn

**Python:**

```python
from tinyfish import TinyFish, CompleteEvent

client = TinyFish()  # tự động đọc TINYFISH_API_KEY từ biến môi trường

with client.agent.stream(
    url="https://scrapeme.live/shop",
    goal="Trích xuất tên và giá của 2 sản phẩm đầu tiên, trả về dưới dạng JSON.",
) as stream:
    for event in stream:
        if isinstance(event, CompleteEvent):
            print(event.result_json)
```

**TypeScript:**

```typescript
import { TinyFish, EventType } from "@tiny-fish/sdk";

const client = new TinyFish();

const stream = await client.agent.stream({
  url: "https://scrapeme.live/shop",
  goal: "Trích xuất tên và giá của 2 sản phẩm đầu tiên, trả về dưới dạng JSON.",
});

for await (const event of stream) {
  if (event.type === EventType.COMPLETE) {
    console.log(event.result);
  }
}
```

**HTTP thô (không dùng SDK):**

```bash
curl -N -X POST https://agent.tinyfish.ai/v1/automation/run-sse \
  -H "X-API-Key: $TINYFISH_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://scrapeme.live/shop",
    "goal": "Trích xuất tên và giá của 2 sản phẩm đầu tiên"
  }'
```

Chạy lên, bạn sẽ thấy các Server-Sent Events chảy vào terminal theo thời gian thực:

```
{"type": "STARTED", "run_id": "abc123"}
{"type": "STREAMING_URL", "run_id": "abc123", "streaming_url": "https://tf-abc123.fra0-tinyfish.unikraft.app/stream/0"}
{"type": "PROGRESS", "run_id": "abc123", "purpose": "Visit the page to extract product information"}
{"type": "PROGRESS", "run_id": "abc123", "purpose": "Check for product information on the page"}
{"type": "COMPLETE", "run_id": "abc123", "status": "COMPLETED", "result": {
  "products": [
    {"name": "Bulbasaur", "price": "$63.00"},
    {"name": "Ivysaur", "price": "$87.00"}
  ]
}}
```

Vậy đó — bạn vừa chạy xong một Web Agent. Những gì diễn ra đằng sau lần gọi này: một trình duyệt Chromium thực được khởi động trong trung tâm dữ liệu, điều hướng tới trang đích, phân tích bố cục, nhận diện các thẻ sản phẩm, trích xuất dữ liệu, và stream tiến độ trả về cho bạn.

#### Giải phẫu một yêu cầu (request)

Các trường JSON mà endpoint `/run-sse` của TinyFish nhận (**các trường trọng điểm đã được in đậm**):

| Trường | Kiểu | Mô tả |
| --- | --- | --- |
| **`url`** | string | Trang khởi đầu |
| **`goal`** | string | Chỉ dẫn bằng ngôn ngữ tự nhiên |
| `output_schema` | object | Tập con JSON Schema tùy chọn, ràng buộc cấu trúc trả về |
| `browser_profile` | `"lite"` \| `"stealth"` | Mặc định `lite` |
| `proxy_config` | object | Tùy chọn `{enabled, type, country_code}` |
| `use_vault` | boolean | Có dùng thông tin đăng nhập đã lưu để hoàn tất đăng nhập hay không |
| `credential_item_ids` | string[] | Giới hạn vào các thông tin đăng nhập cụ thể trong vault |

Ba endpoint, ba chế độ:

- `POST /v1/automation/run` — đồng bộ, chặn cho đến khi hoàn tất, trả về kết quả cuối cùng
- `POST /v1/automation/run-async` — gửi rồi quên (fire-and-forget), trả về `run_id` ngay lập tức
- `POST /v1/automation/run-sse` — stream, đẩy các sự kiện tiến độ về cho bạn theo thời gian thực

Nhiệm vụ ngắn (< 30 giây) dùng đồng bộ; công việc theo lô (batch) dùng bất đồng bộ; khi cần hiển thị tiến độ thời gian thực trên UI thì dùng SSE.

### 2.2 Quan sát tác nhân làm việc — URL phát trực tiếp

Một tính năng hữu ích nhất về mặt sư phạm của TinyFish là **`streaming_url`**: mỗi lần chạy đều sinh ra một URL cho phép bạn **xem trình duyệt theo thời gian thực**. Bạn có thể nhúng nó vào iframe trong sản phẩm của mình, hoặc trực tiếp mở một Tab trình duyệt để vừa gỡ lỗi vừa xem.

```python
from tinyfish import TinyFish, CompleteEvent

client = TinyFish()

with client.agent.stream(
    url="https://scrapeme.live/shop",
    goal="Trích xuất tên và giá của 3 sản phẩm đầu tiên",
    on_streaming_url=lambda e: print(f"\nXem trực tiếp: {e.streaming_url}\n"),
    on_progress=lambda e: print(f"  → {e.purpose}"),
) as stream:
    for event in stream:
        if isinstance(event, CompleteEvent):
            print("\nKết quả cuối cùng:", event.result_json)
```

Chạy lên gần như tức thì bạn có được một URL có thể nhấp. Mở nó trong trình duyệt — những gì bạn thấy chính là **phiên trình duyệt thực mà tác nhân đang điều khiển**. Trang đang tải, chuột đang di chuyển, các trường đang được điền. Đây là khoảnh khắc mà đa số độc giả sẽ thốt lên "trời ơi, nó thực sự đang làm".

Và đây không chỉ là một chiêu trò để phô diễn. Luồng phát trực tiếp là công cụ **hữu dụng nhất** để gỡ lỗi Web Agent:

- Một lần chạy trả về kết quả rỗng? Mở luồng trực tiếp — rất có thể bạn sẽ thấy một trang chặn Cloudflare "Checking your browser", hoặc một pop-up ngoài dự kiến
- Tác nhân nhấp nhầm nút? Xem luồng trực tiếp, sửa goal
- Đang làm sản phẩm hướng tới người dùng? Nhúng URL trực tiếp vào iframe, để người dùng xem công việc diễn ra

### 2.3 Cách viết goal cho tốt

Riêng trường `goal` gần như là toàn bộ bề mặt API của TinyFish — viết tốt nó chính là ranh giới phân cách giữa tỷ lệ thành công 90% và 30%.

Mô hình tư duy mà TinyFish khuyến nghị chính thức: **hãy coi tác nhân như một trợ lý "thực thi theo nghĩa đen", đang ngồi trước trình duyệt**. Nó có thể nhìn thấy mọi thứ trên màn hình, có thể ra tay — nhưng nó không thể đoán ý bạn.

Một goal xuất sắc bao gồm tối đa bảy thành phần:

| Thành phần | Ví dụ |
| --- | --- |
| **Mục tiêu (Objective)** | "Trích xuất thông tin giá" |
| **Phạm vi (Target)** | "Từ bảng giá" |
| **Trường (Fields)** | "Tên gói, phí hàng tháng, tính năng bao gồm" |
| **Cấu trúc (Schema)** | "Trả về JSON: `[{plan: string, price_monthly: number}]`" |
| **Các bước (Steps)** | "Trước tiên đóng banner cookie" |
| **Rào chắn (Guardrails)** | "Không nhấp bất kỳ nút 'Mua ngay' nào" |
| **Trường hợp biên (Edge cases)** | "Nếu giá hiển thị là 'Liên hệ chúng tôi', đặt là null" |

Quá trình tiến hóa từ tệ nhất đến tốt nhất:

**Mơ hồ (chắc chắn thất bại):**
```
"Lấy giá của trang này"
```

**Khá hơn (có thể chạy được):**
```
"Trích xuất tên sản phẩm, giá và tình trạng tồn kho"
```

**Chất lượng cấp sản xuất:**
```
1. Đợi trang tải hoàn toàn.
2. Nếu có banner đồng ý cookie, nhấp "Chấp nhận tất cả".
3. Định vị khu vực giá.
4. Với mỗi gói, trích xuất: tên gói, phí hàng tháng (số), tính năng bao gồm (mảng chuỗi).

Không nhấp bất kỳ nút mua hoặc thanh toán nào.
Nếu gói hiển thị "Liên hệ chúng tôi", đặt monthly_price là null.

Trả về JSON: [{"plan": string, "monthly_price": number | null, "features": string[]}]
```

Trong các thử nghiệm của chính TinyFish, với cùng một nhiệm vụ thì goal cụ thể **hoàn thành nhanh hơn 4.9 lần**, **trả về dữ liệu thừa ít hơn 16 lần**. Viết goal chính là kỹ thuật Prompt kiểu mới, và cũng giống mọi kỹ thuật Prompt, hiệu ứng lãi kép rất rõ.

### 2.4 Món chính: tích hợp TinyFish vào HelloAgents

Phần này là phần cốt lõi của chương. Gọi API trực tiếp tất nhiên là được — nhưng thông điệp cốt lõi mà cuốn sách này, đặc biệt là chương 4 và chương 7, luôn truyền tải là: **khi dịch vụ bên ngoài trở thành một công cụ trong framework của chính bạn, sức mạnh thực sự mới xuất hiện**. Đây chính là ranh giới phân cách giữa "sử dụng AI" và "dùng AI để xây dựng".

Chúng ta sẽ đóng gói TinyFish thành một `Tool` của HelloAgents, tích hợp vào `ReActAgent`.

> **Gợi ý**: HelloAgents là framework đồng hành bạn đã xây dựng ở chương 7. Nếu chưa cài: `pip install hello-agents`. Muốn có phiên bản khớp hoàn toàn với phần chính văn, bạn có thể chuyển sang nhánh `learn_version` trên GitHub.

#### Bước 1: Định nghĩa `TinyFishWebTool`

Chúng ta viết một công cụ, phơi bày năng lực tự động hóa Web thành một năng lực duy nhất. Tác nhân dùng ngôn ngữ tự nhiên mô tả điều mình muốn làm, công cụ phụ trách gọi API và trả về JSON có cấu trúc.

```python
# tools/tinyfish_tool.py
import json
import os
from typing import Any, Dict, List

from tinyfish import (
    TinyFish,
    BrowserProfile,
    ProxyConfig,
    ProxyCountryCode,
)

from hello_agents.tools import Tool, ToolParameter


class TinyFishWebTool(Tool):
    """Công cụ cho phép tác nhân ReAct điều khiển trình duyệt thực bằng ngôn ngữ tự nhiên."""

    def __init__(self, api_key: str | None = None):
        super().__init__(
            name="web_automation",
            description=(
                "Sử dụng ngôn ngữ tự nhiên để tự động hóa bất kỳ trang web nào. Nhập vào một chuỗi JSON gồm hai trường bắt buộc: "
                "`url` (trang khởi đầu) và `goal` (mô tả nhiệm vụ rõ ràng, cụ thể). "
                "Trường tùy chọn: `stealth` (giá trị boolean, dành cho các trang có bảo vệ chống bot), "
                "`country` (US/GB/CA/DE/FR/JP/AU, dùng cho định tuyến địa lý). "
                "Trả về JSON có cấu trúc do tác nhân trích xuất, hoặc mô tả lỗi."
            ),
        )
        self.client = TinyFish(
            api_key=api_key or os.environ["TINYFISH_API_KEY"],
        )

    def run(self, parameters: Dict[str, Any]) -> str:
        # ToolRegistry sẽ bọc văn bản đầu vào của ReAct thành {"input": "..."}
        raw = parameters.get("input", "")
        try:
            params = json.loads(raw)
        except json.JSONDecodeError:
            return json.dumps(
                {"error": "Đầu vào phải là một chuỗi JSON hợp lệ"},
                ensure_ascii=False,
            )

        url = params.get("url")
        goal = params.get("goal")
        if not url or not goal:
            return json.dumps(
                {"error": "Thiếu trường bắt buộc url hoặc goal"},
                ensure_ascii=False,
            )

        kwargs: Dict[str, Any] = {"url": url, "goal": goal}
        if params.get("stealth"):
            kwargs["browser_profile"] = BrowserProfile.STEALTH
        if (country := params.get("country")):
            kwargs["proxy_config"] = ProxyConfig(
                enabled=True,
                country_code=ProxyCountryCode(country),
            )

        # Dùng run đồng bộ — vòng lặp ReAct phải lấy được kết quả rồi mới tiếp tục.
        # Nhiệm vụ dài có thể chuyển sang dùng queue + polling.
        run = self.client.agent.run(**kwargs)

        if run.status.value != "COMPLETED" or run.result is None:
            err = run.error.message if run.error else "Thất bại không xác định"
            return json.dumps(
                {"error": err, "status": run.status.value},
                ensure_ascii=False,
            )

        return json.dumps(
            {"data": run.result, "run_id": run.run_id},
            ensure_ascii=False,
        )

    def get_parameters(self) -> List[ToolParameter]:
        return [
            ToolParameter(
                name="input",
                type="string",
                description=(
                    "Chuỗi JSON, các trường: url (bắt buộc), goal (bắt buộc), "
                    "stealth (tùy chọn), country (tùy chọn)"
                ),
                required=True,
            )
        ]
```

Có vài chỗ trong code đáng đặc biệt lưu ý:

1. **`description` của công cụ là thông tin duy nhất mà LLM nhìn thấy khi quyết định có gọi nó hay không**. Hãy viết mô tả như thể bạn đang giao việc cho một lập trình viên non tay chưa từng đọc code của bạn — nói rõ đầu vào, đầu ra, khi nào thì dùng.
2. **Công cụ luôn trả về chuỗi (string)**. ReAct là văn bản vào, văn bản ra. Chúng ta tuần tự hóa (serialize) kết quả thành JSON để tác nhân có thể tiếp tục suy luận ở bước "Thought" tiếp theo.
3. **`stealth` và `country` không phải mặc định bật mà là tham số tùy chọn**. Hãy để LLM tự quyết định — thông qua mô tả công cụ để nói cho nó biết khi nào thì nên bật.

#### Bước 2: Tích hợp công cụ vào tác nhân ReAct

```python
# main.py
from hello_agents import ReActAgent, HelloAgentsLLM, ToolRegistry
from tools.tinyfish_tool import TinyFishWebTool

llm = HelloAgentsLLM()  # đọc cấu hình provider từ .env
registry = ToolRegistry()
registry.register_tool(TinyFishWebTool())

agent = ReActAgent(
    agent_name="research_assistant",
    llm=llm,
    tool_registry=registry,
)

result = agent.run(
    "Tra cứu giá hiện tại của iPhone 17 Pro trên cửa hàng chính hãng Apple và trên JD, "
    "sau khi tính đến phí vận chuyển ghi trên trang sản phẩm hãy cho tôi biết bên nào rẻ hơn."
)
print(result)
```

Chạy lên, vòng lặp ReAct đại khái sẽ trải qua quá trình như thế này:

1. **Thought (suy nghĩ)**: "Tôi cần lấy giá từ hai trang khác nhau. Tôi nên gọi web_automation hai lần."
2. **Action (hành động)**: `web_automation({"url": "https://www.apple.com/.../iphone-17-pro", "goal": "Trích xuất giá khởi điểm của iPhone 17 Pro. Trả về JSON: {price_cny: number, free_shipping: boolean}"})`
3. **Observation (quan sát)**: `{"data": {"price_cny": 9999, "free_shipping": true}}`
4. **Thought**: "Bây giờ lấy giá JD. JD có chống bot — tôi nên bật stealth."
5. **Action**: `web_automation({"url": "https://item.jd.com/...", "goal": "...", "stealth": true})`
6. **Observation**: `{"data": {"price_cny": 9799, "free_shipping": true}}`
7. **Thought**: "Cả hai bên đều miễn phí vận chuyển. JD rẻ hơn 200 tệ."
8. **Final Answer**: "JD hiện rẻ hơn 200 tệ so với Apple chính hãng: JD 9.799 tệ, Apple chính hãng 9.999 tệ, cả hai đều miễn phí vận chuyển."

Điều vừa xảy ra chính là thông điệp cốt lõi mà chương 7 muốn truyền tải: **dịch vụ bên ngoài trở thành công dân hạng nhất bước vào framework của chính bạn**. Vòng lặp ReAct không đổi, LLM cũng không đổi. Chúng ta chỉ thêm một công cụ, và tác nhân của bạn đã có được năng lực hành động trên mạng công khai. Đây chính là **tính tổ hợp (composability)**.

#### Bước 3: Đưa nó lên cấp sản xuất

Phiên bản trên đủ để trình diễn. Môi trường sản xuất thực còn phải thêm vài thứ nữa:

**1. Xác minh nội dung, đừng chỉ nhìn trạng thái**

Một lần chạy `COMPLETED` cũng có thể trả về rác, nếu tác nhân va phải một cú chặn mềm (trang thử thách Cloudflare, captcha, render "Truy cập bị từ chối" thành nội dung chính). Luôn phải kiểm tra **nội dung kết quả**:

```python
def is_real_success(result: dict | None) -> bool:
    if not result:
        return False
    s = json.dumps(result, ensure_ascii=False).lower()
    failure_signals = ["captcha", "blocked", "access denied", "could not", "unable to"]
    return not any(signal in s for signal in failure_signals)
```

**2. Cache được thì cache**

Một lần tự động hóa web 30 giây là đắt đỏ. Nếu trong cùng một phiên mà tác nhân ReAct yêu cầu cùng một URL hai lần, thì nên trả về kết quả đã cache. (Hệ thống ký ức ở chương 8 là điểm tựa phù hợp.)

**3. Đặt timeout**

TinyFish bên trong có timeout 10 phút cho mỗi lần chạy, nhưng công cụ của bạn nên thất bại sớm hơn — đa số nhiệm vụ có ý nghĩa hoàn thành trong 10–60 giây. Vượt quá thời gian này, phần lớn là bị kẹt ở trang thử thách rồi.

**4. Dùng luồng trực tiếp để quan sát**

Ghi lại `streaming_url` của mỗi lần chạy vào log. Khi môi trường sản xuất gặp sự cố, bản ghi hình lần chạy là công cụ nhanh nhất để định vị lỗi.

### 2.5 Ứng phó với chống bot

Sớm muộn gì tác nhân của bạn cũng sẽ va phải một trang trả về kết quả rỗng hoặc lỗi 403. Đây là quy trình chẩn đoán:

**Bước 1 — xác nhận vấn đề đúng là do chống bot**. Mở `streaming_url` của lần chạy thất bại, tìm các đặc điểm sau:

| Bạn thấy gì | Nguyên nhân khả năng cao |
| --- | --- |
| Trang Cloudflare "Checking your browser" | Phát hiện bot của Cloudflare |
| Pop-up hoặc chuyển hướng của DataDome | DataDome |
| Trang trắng hoặc quay vòng mãi | Chặn dựa trên IP hoặc vân tay |
| Captcha (reCAPTCHA, hCaptcha) | Captcha — trần cứng |
| "Access Denied" / 403 | Chặn theo IP hoặc User-Agent |
| Đáng lẽ thấy nội dung thì lại hiện tường đăng nhập | Chống bot dựa trên phiên |

**Bước 2 — bật tàng hình và proxy cùng lúc**. Tàng hình thay đổi vân tay trình duyệt, proxy thay đổi IP. Nhà cung cấp chống bot sẽ liên kết cả hai tín hiệu — chỉ đổi một cái thường là không đủ:

```python
run = client.agent.run(
    url="https://protected-site.com",
    goal="Trích xuất giá sản phẩm",
    browser_profile=BrowserProfile.STEALTH,
    proxy_config=ProxyConfig(enabled=True, country_code=ProxyCountryCode.US),
)
```

**Bước 3 — làm cho tác nhân trông giống con người hơn**. Một số trang quan tâm đến hành vi chứ không chỉ vân tay. Trong goal hãy thêm:

- Đóng banner cookie một cách tường minh
- Đợi trang tải xong rồi mới trích xuất
- Mô tả phần tử bằng thị giác ("nút 'Thêm vào giỏ hàng' màu xanh"), đừng dùng bộ chọn
- Quy trình nhiều bước dùng bước đánh số, để tác nhân tự chậm lại

**Bước 4 — thực sự không được thì đổi cách chơi**. Một số trang chặn crawler quyết liệt, nhưng lại hào phóng cung cấp RSS, sitemap, API công khai. Bỏ ra năm phút lật qua, tiết kiệm được mấy ngày thử-và-sai. Nếu dữ liệu thực sự chỉ nằm trong trang đã định dạng sau bức tường phí, hãy tự hỏi: nguồn thông tin gốc (thông báo, thông cáo báo chí, trang nhà cung cấp) liệu có xuất hiện ở nơi phòng thủ yếu hơn không.

Một ví dụ hoàn chỉnh, đã được gia cố:

```python
from tinyfish import (
    TinyFish, BrowserProfile, ProxyConfig, ProxyCountryCode,
    CompleteEvent, RunStatus,
)

client = TinyFish()

with client.agent.stream(
    url="https://protected-site.com/pricing",
    browser_profile=BrowserProfile.STEALTH,
    proxy_config=ProxyConfig(enabled=True, country_code=ProxyCountryCode.US),
    goal="""
        1. Đợi trang tải hoàn toàn.
        2. Đóng bất kỳ banner đồng ý cookie / GDPR nào.
        3. Đợi 1 giây trước khi tiếp tục.
        4. Định vị khu vực giá (thường là dạng lưới hoặc bảng thẻ).
        5. Với mỗi gói, trích xuất: tên gói, phí hàng tháng, phí hàng năm (nếu có).

        Nếu xuất hiện trang kiểm tra bảo mật hoặc Cloudflare, hãy đợi nó tự động vượt qua.
        Nếu thấy trang Access Denied hoặc CAPTCHA, trả về {"error": "blocked"}.
        Không nhấp bất kỳ nút mua hoặc thanh toán nào.

        Trả về JSON: [{"plan": "Pro", "monthly_price": 49, "annual_price": 39}]
    """,
    on_streaming_url=lambda e: print(f"Xem trực tiếp: {e.streaming_url}"),
    on_progress=lambda e: print(f"  → {e.purpose}"),
) as stream:
    for event in stream:
        if isinstance(event, CompleteEvent):
            if event.status == RunStatus.COMPLETED:
                print("Kết quả:", event.result_json)
            else:
                print("Thất bại:", event.error.message if event.error else "unknown")
```

**Tuyên bố thành thật quan trọng**: bao gồm cả TinyFish, không có Web Agent nào có thể giải một cách đáng tin cậy captcha hiện đại (reCAPTCHA v2/v3, hCaptcha). Nếu một trang bật captcha cho bạn, thì hiện tại đó là một bức tường cứng. Cách làm đúng là thiết kế lần chạy của bạn sao cho không kích hoạt nó, chứ không phải cứng đầu chống chọi.

### 2.6 Món thêm: gọi TinyFish qua MCP (tương ứng chương 10)

Chương 10 giới thiệu **MCP (Model Context Protocol)** — một giao thức mở do Anthropic đề xuất, cho phép trợ lý AI kết nối với các công cụ và dữ liệu bên ngoài. TinyFish phơi bày một máy chủ MCP. Điều này có nghĩa là, bạn có thể trao cho bất kỳ trợ lý nào tương thích MCP (Claude Desktop, Cursor, Windsurf, Claude Code) năng lực điều khiển trình duyệt — mà bạn không cần viết một dòng code nào.

Cài đặt bằng một dòng lệnh:

```bash
# Claude Code
npx -y install-mcp@latest https://agent.tinyfish.ai/mcp --client claude-code

# Claude Desktop
npx -y install-mcp@latest https://agent.tinyfish.ai/mcp --client claude

# Cursor
npx -y install-mcp@latest https://agent.tinyfish.ai/mcp --client cursor
```

Hoặc thêm thủ công vào `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "tinyfish": {
      "url": "https://agent.tinyfish.ai/mcp"
    }
  }
}
```

Sau khi khởi động lại, trợ lý của bạn sẽ có thêm các công cụ như `run_web_automation`, `search`, `fetch_content`, `create_browser_session`. Rồi bạn có thể nói thẳng với Claude:

> "Hãy cào song song trang giá của 5 đối thủ cạnh tranh sau đây, sắp xếp kết quả thành bảng."

Claude sẽ dùng TinyFish để hoàn thành việc này ở phía sau, thực thi theo lô, phản hồi tiến độ theo thời gian thực.

Đây chính là phần thưởng **xây một lần, dùng mọi nơi** mà chương 10 đã báo trước. MCP với vai trò là giao thức phân phối, đã biến mỗi công cụ thành một năng lực mà **mọi trợ lý AI đều có thể trực tiếp sử dụng**.

---

## Phần ba: Suy ngẫm tổng hợp

### 3.1 Khi nào **không nên** dùng Web Agent kiểu dịch vụ

Để cho thành thật: Web Agent kiểu dịch vụ không phải lúc nào cũng là lựa chọn đúng.

- **Có API chính thức** — hãy dùng API. API nhanh hơn, rẻ hơn, đáng tin cậy hơn
- **Cào tần suất cao trên trang không có chống bot** — Playwright thuần là đủ, rẻ hơn vài bậc độ lớn
- **Luồng công việc then chốt mà cái giá của lỗi rất cao** — bất kể bạn dùng công cụ gì, luôn phải thêm điểm kiểm tra có con người trong vòng lặp ở các bước then chốt
- **Công cụ nội bộ mà bạn kiểm soát hoàn toàn** — thì thêm luôn một endpoint API cho xong

Khi nào thì dùng Web Agent kiểu dịch vụ: trang đích ở bên ngoài, không có API, có bảo vệ chống bot, quy trình nhiều bước, **và thời gian kỹ thuật đáng giá hơn đơn giá mỗi nhiệm vụ**.

### 3.2 Hướng đi tiếp theo của Web Agent

Lĩnh vực này đi rất nhanh. Vài điều đáng chú ý trong 12–24 tháng tới:

- **Mô hình nhỏ hơn, nhanh hơn, tinh chỉnh chuyên cho Web**. Dùng mô hình tiên phong để "nhấp nút màu xanh" là dùng dao mổ trâu giết gà. Thế hệ tác nhân chuyên gia OS tiếp theo — chẳng hạn các mô hình sau Qwen-VL-2.5-VL — sẽ giảm mạnh độ trễ và chi phí của mỗi bước hành động.
- **Phiên xác thực trở thành công dân hạng nhất**. Tích hợp Vault (cầu nối tới trình quản lý mật khẩu được mã hóa) bắt đầu đi vào thực tế. Điều này sẽ mở khóa làn sóng ứng dụng tác nhân tiếp theo — ngân hàng, y tế, công cụ nội bộ, những kịch bản mà đăng nhập là cửa ngõ.
- **Benchmark học thuật tiến gần mực nước sản xuất**. Tỷ lệ thành công sản xuất trên các luồng nghiệp vụ thường gặp đã ổn định quanh mức 90%; điểm số trên các benchmark đối kháng như WebArena cũng đang leo dốc nhanh, khoảng cách đang bị nén lại đồng thời từ cả hai đầu kỹ thuật và mô hình.
- **Web Agent cục bộ (local)**. Các nhiệm vụ nhạy cảm về riêng tư (hồ sơ y tế, thuế) cần tác nhân chạy cục bộ. VLM cục bộ kết hợp Chromium cục bộ đã trông có vẻ khả thi.
- **Đánh giá Web Agent trưởng thành hóa**. Phương pháp luận đánh giá được trình bày ở chương 12 vừa vặn áp dụng được — sẽ sớm thấy các bộ đánh giá chuẩn hóa hướng tới Web Agent cấp sản xuất.

### 3.3 Đi tiếp theo hướng nào

Nếu chương này khơi gợi được hứng thú của bạn, dưới đây là vài con đường để tiếp tục:

- **Kết hợp chương 8 (ký ức)**: để Web Agent của bạn nhớ được hôm qua đã cào được gì, hôm nay chỉ lấy phần tăng thêm (incremental)
- **Kết hợp chương 12 (đánh giá)**: gắn cho Web Agent của bạn việc theo dõi tỷ lệ thành công. Bạn sẽ nhanh chóng nắm rõ trang nào cần stealth, goal nào cần tinh chỉnh thêm
- **Kết hợp chương 13 và chương 14**: trợ lý du lịch và tác nhân DeepResearch một khi mở rộng không gian hành động tới "trình duyệt thực" sẽ biến đổi về chất. Thử dùng Web Agent làm backend để viết lại một trong số đó
- **Các giải pháp mã nguồn mở đáng nghiên cứu**: Browser-Use (Python), Skyvern (Python), WebVoyager (benchmark nghiên cứu), AgentE (hạ tầng tác nhân mã nguồn mở)
- **Benchmark**: WebArena, VisualWebArena, Mind2Web, WebVoyager-2024 — đọc hết các bài báo một lượt, bạn sẽ rõ Web Agent năm 2026 đang ở mực nước nào

---

## Lời kết

Suy cho cùng, Web Agent chẳng qua chỉ là một vòng lặp ReAct — có điều bề mặt hành động của nó là trình duyệt. Mọi thứ bạn học được ở các chương trước — cảm nhận, suy luận, hành động, ký ức, đánh giá — đều áp dụng trực tiếp được. Cái mới là **môi trường đối kháng** này: sự thù địch của các trang web thực đối với tự động hóa, thứ mà App di động và phần mềm máy tính không có, chính sự thù địch đó là lý do kỹ thuật Web Agent trở thành một môn học độc lập.

Tin tốt là: bạn không cần giải quyết tất cả những thứ này từ con số không. Những nguyên hàm (primitive) cấp sản xuất như TinyFish tồn tại chính là vì điều đó. Với vai trò một người xây dựng tác nhân AI-native, công việc của bạn là **tổ hợp** những nguyên hàm này thành thứ gì đó hữu ích — và làm việc đó trong một framework mà bạn **hiểu rõ từ đầu đến cuối** của chính mình.

Hãy đi xây dựng thôi. Web mở là bề mặt hành động lớn nhất thế giới, và chưa bao giờ có thời điểm nào tốt hơn lúc này để dạy một tác nhân biết cách sử dụng nó.

---

*Lời cảm ơn: Cảm ơn đội ngũ TinyFish đã đóng góp các chi tiết kỹ thuật cho chương này. Tất cả các chi tiết API đều được đối chiếu từ [tài liệu chính thức của TinyFish](https://docs.tinyfish.ai).*
