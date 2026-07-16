# Kiến thức bổ sung về Kỹ thuật Ngữ cảnh

## Giới thiệu

Vì sao gần đây kỹ thuật ngữ cảnh (context engineering) lại một lần nữa trở nên nóng bỏng? Điều này bắt nguồn từ cuộc trò chuyện của Jeff, nhà sáng lập kiêm CEO của Chroma, trên [podcast](https://youware.app/project/7529x70z4p) Len Space.
Chroma là bá chủ mã nguồn mở trong lĩnh vực cơ sở dữ liệu vector (vector database). Ngay cả bài báo Voyager lừng danh cũng sử dụng nó.
Tiêu đề cuộc trò chuyện của CEO Jeff xoay quanh quan điểm "RAG is dead" (RAG đã chết). Trong video, ông đã chỉ ra rất rõ ràng những giới hạn của RAG ban đầu và tầm quan trọng của context engineering ngày nay.

![alt text](./images/Extra02-figures/image-1.png)


Trong chương này, trước tiên chúng ta sẽ trình bày một cách toàn diện về khái niệm "kỹ thuật ngữ cảnh" (context engineering),
và ở cuối bài viết sẽ bàn về quan điểm "RAG is dead".



## Kỹ thuật ngữ cảnh là gì?

Chúng ta có thể lấy một phép so sánh: Agent giống như một loại [hệ điều hành kiểu mới](https://www.youtube.com/watch?si=-aKY-x57ILAmWTdw&t=620&v=LCEmiRjPEtQ&feature=youtu.be&ref=blog.langchain.com). LLM giống như CPU, còn [cửa sổ ngữ cảnh (context window)](https://docs.anthropic.com/en/docs/build-with-claude/context-windows?ref=blog.langchain.com) của nó giống như RAM, đóng vai trò là bộ nhớ làm việc của mô hình. Giống như RAM, [dung lượng](https://lilianweng.github.io/posts/2023-06-23-agent/?ref=blog.langchain.com) của cửa sổ ngữ cảnh LLM là hữu hạn, không thể xử lý ngữ cảnh đến từ đủ mọi nguồn. Và kỹ thuật ngữ cảnh, giống như hệ điều hành quản lý RAM của CPU, sẽ quản lý cửa sổ ngữ cảnh của LLM, quyết định lúc nào thì nạp nội dung gì vào. [Karpathy đã tổng kết rất hay](https://x.com/karpathy/status/1937902205765607626?ref=blog.langchain.com):
_"Kỹ thuật ngữ cảnh là... nghệ thuật và khoa học tinh tế trong việc nạp vào cửa sổ ngữ cảnh lượng thông tin vừa đủ, chuẩn xác cho bước tiếp theo."_

![llm_context_engineering](https://blog.langchain.com/content/images/2025/07/image-1.png)
















## [Khái niệm về kỹ thuật ngữ cảnh](https://blog.langchain.com/context-engineering-for-agents/`)

![alt text](./images/Extra02-figures/image-2.png)

Context (ngữ cảnh) chính là tất cả những gì mà mô hình "nhìn thấy". Thực ra mô hình không chỉ dựa vào prompt (câu lệnh đầu vào) mà chúng ta nhập để trả lời câu hỏi, mà còn kết hợp với những thông tin khác để sinh ra câu trả lời. Kỹ thuật ngữ cảnh là khái niệm bao trùm áp dụng cho một số loại ngữ cảnh khác nhau:

- <strong>Instructions (ngữ cảnh chỉ dẫn)</strong>: prompt engineering như prompt, bộ nhớ, các ví dụ mẫu ít ỏi (few-shot), v.v., bao gồm:
  - System prompt (câu lệnh hệ thống): định nghĩa vai trò, quy tắc hành xử và phong cách phản hồi của AI
  - Chỉ dẫn của người dùng: mô tả nhiệm vụ cụ thể và các yêu cầu
  - Ví dụ few-shot: các ví dụ đầu vào - đầu ra, giúp hiểu rõ định dạng mong đợi
  - Mô tả công cụ: quy cách và hướng dẫn sử dụng của hàm hoặc công cụ
  - Ràng buộc định dạng: yêu cầu về định dạng và cấu trúc của đầu ra
- <strong>Knowledge (ngữ cảnh tri thức)</strong>: rag như sự kiện, cơ sở tri thức, v.v., bao gồm:
  - Tri thức chuyên ngành: thông tin sự kiện của một ngành hoặc lĩnh vực chuyên môn cụ thể
  - Bộ nhớ: sở thích của người dùng, lịch sử tương tác và bản ghi phiên làm việc
  - Cơ sở tri thức: lấy thông tin liên quan từ cơ sở dữ liệu hoặc kho tri thức
  - Dữ liệu thời gian thực: thông tin trạng thái hiện tại được cập nhật động
- <strong>Tools (ngữ cảnh công cụ)</strong>: agent như mô tả công cụ và phản hồi khi gọi công cụ, bao gồm:
  - Kết quả gọi hàm: phản hồi API hoặc kết quả truy vấn
  - Trạng thái thực thi công cụ: phản hồi thành công, thất bại hoặc lỗi
  - Chuỗi công cụ nhiều bước: quan hệ phụ thuộc và truyền dữ liệu giữa các công cụ
  - Lịch sử thực thi: bản ghi và kết quả của các lần gọi công cụ






### Ví dụ — Trợ lý thông minh của ứng dụng du lịch


![alt text](./images/Extra02-figures/image-5.png)


Để phân biệt rõ ràng bốn khái niệm này, chúng ta hãy đặt ra một tình huống thực tế thống nhất, rồi xem mỗi phương pháp giải quyết vấn đề này như thế nào.

<strong>Tình huống: Trợ lý thông minh của một ứng dụng du lịch</strong>

<strong>Nhu cầu người dùng:</strong> "Hãy giúp tôi lên kế hoạch một chuyến du lịch gia đình ba ngày đến Bắc Kinh. Chúng tôi gồm hai người lớn và một đứa trẻ 5 tuổi, thích lịch sử văn hóa, cũng muốn có một số hoạt động thư giãn thú vị. Tổng ngân sách của chúng tôi là 8000 tệ."

---

#### 1. Kỹ thuật prompt (Prompt Engineering)

Đây là phương pháp cơ bản nhất, trực tiếp nhất. Cốt lõi của nó là <strong>làm thế nào để đặt một câu hỏi hay cho mô hình ngôn ngữ (LLM)</strong>, với hy vọng nó chỉ dựa vào kho tri thức chung nội tại của mình cũng có thể đưa ra câu trả lời tốt nhất.

*   <strong>Tư tưởng cốt lõi:</strong> Tối ưu hóa chỉ dẫn (Prompt) đưa vào mô hình, để nó xuất ra kết quả phù hợp hơn với kỳ vọng.
*   <strong>Cách thức hoạt động:</strong>
    1.  Nhà phát triển hoặc người dùng cẩn thận cấu trúc toàn bộ nhu cầu thành một prompt chi tiết.
    2.  Gửi trực tiếp prompt này đến một mô hình ngôn ngữ lớn thông dụng (như GPT-4).
    3.  Mô hình hoàn toàn dựa vào tri thức nội tại tính đến ngày huấn luyện (chẳng hạn năm 2023) để trả lời.

*   <strong>Ví dụ:</strong>
    ```
    你是一位专业的旅行规划师。请为北京一个为期三天的家庭旅行设计一份详细行程。
    
    # 家庭成员
    - 2个成年人
    - 1个5岁的儿童
    
    # 兴趣偏好
    - 历史文化（故宫、长城等）
    - 轻松有趣的儿童活动
    
    # 预算
    - 总预算不超过8000元人民币，请给出大致的费用估算。
    
    # 输出要求
    - 每日行程安排（上午、下午、晚上）
    - 交通建议
    - 餐饮推荐（包含适合儿童的餐厅）
    - 预算明细
    ```

*   <strong>Giới hạn:</strong>
    *   <strong>Thông tin lỗi thời:</strong> Không thể cung cấp giá vé, giờ mở cửa thời gian thực hoặc thông tin giao thông mới nhất.
    *   <strong>Thông tin không chính xác:</strong> Ước tính ngân sách có thể rất sơ sài, vì nó không biết giá khách sạn và vé máy bay hiện tại.
    *   <strong>Thiếu cá nhân hóa:</strong> Không thể đề xuất dựa trên sở thích lịch sử của người dùng.
    *   <strong>"Nói bừa một cách nghiêm túc":</strong> Có thể bịa ra một số "khu vui chơi trẻ em" hoặc nhà hàng không tồn tại.

---

#### 2. Sinh tăng cường bằng truy xuất (RAG)

Để giải quyết vấn đề "tri thức cũ kỹ" của kỹ thuật prompt, RAG đã đưa vào <strong>cơ sở tri thức bên ngoài</strong>.

*   <strong>Tư tưởng cốt lõi:</strong> Trước khi sinh ra câu trả lời, hãy truy xuất thông tin liên quan từ một cơ sở dữ liệu cụ thể, đáng tin cậy, sau đó cung cấp những thông tin này cùng với câu hỏi của người dùng cho mô hình.
*   <strong>Cách thức hoạt động:</strong>
    1.  <strong>Chuẩn bị cơ sở tri thức:</strong> Chuẩn bị sẵn một cơ sở dữ liệu chứa các cẩm nang du lịch mới nhất, giới thiệu điểm tham quan, danh sách khách sạn, đánh giá nhà hàng (chẳng hạn một loạt PDF, trang web hoặc bản ghi cơ sở dữ liệu).
    2.  <strong>Truy xuất (Retrieve):</strong> Khi người dùng đặt câu hỏi, hệ thống trước tiên tìm kiếm trong cơ sở tri thức các đoạn tài liệu liên quan đến "du lịch gia đình Bắc Kinh", "điểm tham quan lịch sử văn hóa".
    3.  <strong>Tăng cường (Augment):</strong> Ghép thông tin truy xuất được (ví dụ: "Giá vé Cố Cung mới nhất là 60 tệ, đóng cửa thứ Hai", "Universal Studios Bắc Kinh là hoạt động gia đình nổi tiếng") với câu hỏi gốc của người dùng thành một prompt mới, phong phú hơn về nội dung.
    4.  <strong>Sinh (Generate):</strong> Gửi prompt đã được tăng cường này đến LLM, để nó sinh ra lịch trình dựa trên những tài liệu "tươi mới" này.

*   <strong>Ví dụ:</strong>
    Hệ thống tìm thấy ba đoạn văn bản trong cơ sở tri thức nội bộ: A) giờ mở cửa và giá vé trên trang web chính thức của Cố Cung; B) một bài blog về "dẫn con đi Thiên Đàn"; C) một danh sách "khách sạn thân thiện với gia đình ở Bắc Kinh".
    Sau đó, nó phát lệnh cho LLM: "Dựa vào những thông tin sau: [nội dung các đoạn văn bản A, B, C], hãy lên kế hoạch một chuyến du lịch gia đình ba ngày đến Bắc Kinh cho người dùng, ngân sách 8000 tệ."

*   <strong>Giới hạn:</strong>
    *   <strong>Phản hồi thụ động:</strong> Nó chỉ có thể trả lời dựa trên thông tin bạn cung cấp, không thể chủ động thực hiện nhiệm vụ. Nó không thể đi "tra" vé máy bay, mà chỉ có thể dùng thông tin vé máy bay "có sẵn" trong cơ sở dữ liệu của bạn.
    *   <strong>Tương tác một chiều:</strong> Hoàn thành một lần truy xuất và sinh là kết thúc, không thể thực hiện suy luận và hành động nhiều bước.
    *   <strong>Phụ thuộc vào cơ sở tri thức:</strong> Hiệu quả tốt hay xấu phụ thuộc nặng nề vào chất lượng và tần suất cập nhật của cơ sở tri thức.

---

#### 3. Agent (Tác nhân thông minh)

Agent giúp AI tiến hóa từ một "robot hỏi đáp" thành một <strong>"người hành động" biết suy nghĩ, biết sử dụng công cụ</strong>.

*   <strong>Tư tưởng cốt lõi:</strong> Trao cho mô hình một vòng lặp "suy nghĩ - hành động" (Reasoning-Action Loop), để nó có thể tự chủ lên kế hoạch các bước, sử dụng công cụ bên ngoài (như API) để hoàn thành các nhiệm vụ phức tạp.
*   <strong>Cách thức hoạt động:</strong>
    1.  <strong>Suy nghĩ và lên kế hoạch:</strong> LLM (đóng vai trò là bộ não của Agent) sau khi nhận được nhu cầu của người dùng, sẽ suy nghĩ trước: "Để hoàn thành nhiệm vụ này, tôi cần: 1. Tra giá vé máy bay và khách sạn; 2. Tra vé điểm tham quan; 3. Lên kế hoạch lộ trình; 4. Tổng hợp thành lịch trình."
    2.  <strong>Chọn công cụ (Action):</strong> Nó quyết định sử dụng công cụ đầu tiên: `search_flight_api(from="上海", to="北京", date="...")`.
    3.  <strong>Quan sát kết quả (Observation):</strong> API trả về giá vé máy bay: 5000 tệ.
    4.  <strong>Suy nghĩ lại:</strong> "Vé máy bay tốn 5000, ngân sách còn lại 3000. Tôi cần tìm khách sạn có giá dưới 800 tệ mỗi đêm."
    5.  <strong>Hành động lại:</strong> Sử dụng công cụ `search_hotel_api(city="北京", price_max=800, family_friendly=true)`.
    6.  Vòng lặp này sẽ tiếp tục cho đến khi nó thu thập được tất cả thông tin cần thiết và cuối cùng hoàn thành việc lên kế hoạch.

*   <strong>Ví dụ:</strong>
    Trợ lý này sẽ làm việc như một trợ lý người thật:
    *   "Được, tôi đang tra cứu cho bạn... Tôi phát hiện vé máy bay đi Bắc Kinh vào thứ Sáu tuần sau khoảng 5000 tệ."
    *   "Xét đến ngân sách, tôi đã lọc ra cho bạn vài khách sạn gia đình có đánh giá rất tốt và giá 600-800 tệ/đêm."
    *   "Vé Cố Cung đã được tra qua `ticket_api`, trẻ em miễn phí. Tôi đã thêm thông tin này vào lịch trình."

*   <strong>Giới hạn:</strong>
    *   <strong>Phức tạp và không ổn định:</strong> Đường đi hành vi của Agent không cố định, có thể mắc lỗi (chẳng hạn rơi vào vòng lặp, sử dụng sai công cụ), việc gỡ lỗi và kiểm soát rất khó khăn.
    *   <strong>Chi phí cao:</strong> Mỗi bước suy nghĩ và gọi công cụ đều có thể là một lần gọi API LLM, chi phí khá cao.

---

#### 4. Kỹ thuật ngữ cảnh (Context Engineering)

Kỹ thuật ngữ cảnh là <strong>một bộ môn vĩ mô hơn, nghiêm ngặt hơn</strong>, nó tập trung vào <strong>làm thế nào để xây dựng "cửa sổ ngữ cảnh" tối ưu cho mô hình (dù là RAG đơn giản hay Agent phức tạp)</strong>. Nó là sự tối ưu hóa và nâng tầm của tất cả các phương pháp nêu trên.

*   <strong>Tư tưởng cốt lõi:</strong> Thiết kế và sắp xếp một cách tinh tế toàn bộ thông tin đưa vào ngữ cảnh của mô hình (chỉ dẫn, dữ liệu truy xuất được, lịch sử hội thoại, đầu ra công cụ, v.v.), nhằm đạt được đầu ra hiệu quả nhất, đáng tin cậy nhất. Đây là một khoa học về "cho ăn gì" và "cho ăn như thế nào".
*   <strong>Cách thức hoạt động:</strong>
    Nó không phải là một loại hệ thống độc lập, mà là phương pháp luận để tối ưu hóa RAG và Agent. Quay lại ví dụ lên kế hoạch du lịch:
    1.  <strong>Giai đoạn thu thập (Gather):</strong>
        *   <strong>Truy xuất song song:</strong> Không chỉ truy xuất từ kho cẩm nang du lịch (RAG), mà đồng thời còn:
            *   Gọi `weather_api` để tra thời tiết Bắc Kinh vài ngày tới.
            *   Gọi `events_api` để tra xem có triển lãm hoặc hoạt động đặc biệt nào cho trẻ em không.
            *   Truy xuất từ cơ sở dữ liệu hồ sơ người dùng (CRM) rằng "người dùng này lần du lịch trước đã đặt vé bảo tàng".
            *   Với câu hỏi mơ hồ "hoạt động thư giãn thú vị" của người dùng, thực hiện tìm kiếm đa hướng, bao gồm "công viên giải trí Bắc Kinh", "bảo tàng khoa học Bắc Kinh", "buổi biểu diễn phù hợp với trẻ em".
    2.  <strong>Giai đoạn sàng lọc và nén (Glean & Compact):</strong>
        *   <strong>Sắp xếp lại thứ tự:</strong> Nó phát hiện dự báo thời tiết cho thấy ngày thứ hai có mưa, nên giảm mức ưu tiên của Vạn Lý Trường Thành ngoài trời, nâng trọng số đề xuất cho bảo tàng khoa học trong nhà.
        *   <strong>Nén:</strong> Nó sẽ không ném cả một bài đánh giá khách sạn dài dằng dặc cho mô hình, mà trích xuất thông tin then chốt: "Khách sạn này có khu vui chơi trẻ em, cung cấp nôi cho em bé."
        *   <strong>Định dạng:</strong> Nó tích hợp tất cả thông tin thu thập được, lộn xộn (thời tiết, vé máy bay, sở thích người dùng, giới thiệu điểm tham quan) thành một đối tượng JSON có cấu trúc cao, ngắn gọn rõ ràng.
    3.  <strong>Bàn giao cuối cùng:</strong> Cuối cùng, nó giao gói ngữ cảnh "hoàn hảo" này cho bộ não của Agent (LLM), chỉ dẫn có thể là: "Hãy dựa trên dữ liệu có cấu trúc đã được xác thực, đã được sắp xếp này `[JSON object]`, sinh ra lịch trình cuối cùng cho người dùng."

*   <strong>Ví dụ:</strong>
    Sản phẩm của kỹ thuật ngữ cảnh không phải là lịch trình đưa trực tiếp cho người dùng, mà là "bản đồ tác chiến" tối ưu để mô hình xem. Vì đã qua tối ưu hóa của kỹ thuật ngữ cảnh, công việc của Agent trở nên cực kỳ đơn giản và hiệu quả, nó không cần phải tự mình vất vả thử sai từng bước một, mà dựa trên một bản báo cáo tóm tắt hoàn hảo để trực tiếp tiến hành việc sinh ra kế hoạch cuối cùng.

#### Tổng kết so sánh

| Khái niệm | Tư tưởng cốt lõi | Cách thức hoạt động | Giới hạn |
| :--- | :--- | :--- | :--- |
| <strong>Kỹ thuật prompt</strong> | Hỏi đúng câu hỏi | Thiết kế cẩn thận một Prompt hoàn hảo | Tri thức lỗi thời, không thể tương tác với thế giới bên ngoài |
| <strong>RAG</strong> | Cung cấp tài liệu tham khảo | Trước khi hỏi, truy xuất thông tin liên quan từ cơ sở tri thức | Phản hồi thụ động, không thể thực hiện nhiệm vụ, phụ thuộc vào cơ sở tri thức |
| <strong>Agent</strong> | Trao khả năng hành động | Sử dụng công cụ, hoàn thành nhiệm vụ qua vòng lặp "suy nghĩ - hành động" | Phức tạp, không ổn định, chi phí cao |
| <strong>Kỹ thuật ngữ cảnh</strong> | Tạo ra đầu vào hoàn hảo | Thu thập, sàng lọc, nén, định dạng toàn bộ thông tin một cách hệ thống, cung cấp ngữ cảnh tối ưu cho mô hình | Là một phương pháp luận/bộ môn, chứ không phải một hệ thống cụ thể, triển khai phức tạp |

Nói một cách đơn giản, chúng là sự nâng cấp dần dần về năng lực:
*   <strong>Kỹ thuật prompt</strong> là <strong>người đối thoại</strong>.
*   <strong>RAG</strong> là một <strong>người đối thoại</strong> mang theo một cuốn sách để tra cứu.
*   <strong>Agent</strong> là một <strong>trợ lý</strong> có thể gọi điện thoại, lên mạng tra cứu tài liệu, giúp bạn đặt vé.
*   <strong>Kỹ thuật ngữ cảnh</strong> là <strong>tổng tham mưu trưởng</strong> đứng sau vị trợ lý này, chịu trách nhiệm thu thập và sắp xếp trước tất cả thông tin tình báo, đảm bảo trợ lý có thể đưa ra quyết định sáng suốt nhất.



## Vì sao Context Engineering lại xuất hiện?

![alt text](./images/Extra02-figures/image-3.png)


Khi LLM ngày càng giỏi hơn trong suy luận và gọi công cụ, sự quan tâm của mọi người đối với Agent tăng lên mạnh mẽ. Agent đan xen việc gọi LLM và gọi công cụ, thường được dùng cho các nhiệm vụ chạy trong thời gian dài. Agent sử dụng phản hồi của công cụ để quyết định bước hành động tiếp theo.


Tuy nhiên, các nhiệm vụ chạy trong thời gian dài và phản hồi gọi công cụ tích lũy có nghĩa là Agent thường sử dụng một lượng lớn token. Điều này có thể dẫn đến nhiều vấn đề: có thể vượt quá kích thước cửa sổ ngữ cảnh, làm tăng chi phí/độ trễ hoặc làm giảm hiệu năng của Agent.

Khi cửa sổ ngữ cảnh ngày càng dài, ban đầu chúng ta cứ tưởng rằng "ném tất cả lịch sử hội thoại và tài liệu vào mô hình" là có thể giải quyết được vấn đề bộ nhớ. Nhưng thực nghiệm cho thấy thực tế phức tạp hơn tưởng tượng nhiều. Khi độ dài ngữ cảnh tăng lên, mô hình ngày càng khó duy trì tính chính xác và nhất quán của thông tin, biểu hiện giống như "<strong>bộ nhớ mục nát</strong>".

![alt text](./images/Extra02-figures/image-4.png)

Những hiện tượng này trong nghiên cứu của Chroma được gọi là Context Rot — tức là hiệu năng của mô hình bị "mục nát" trong ngữ cảnh dài. Đây chính là nguyên nhân căn bản khiến vai trò Context Engineer ra đời: cần có người đối kháng và sửa chữa sự "mục nát ngữ cảnh" này, thông qua việc cắt tỉa, nén, tái tổ chức và tăng cường truy xuất, để mô hình duy trì được hiệu năng đáng tin cậy trong nguồn lực chú ý hữu hạn.


## Thách thức ngữ cảnh

Thách thức ngữ cảnh chủ yếu tồn tại ở [bốn phương diện](https://www.dbreunig.com/2025/06/22/how-contexts-fail-and-how-to-fix-them.html?ref=blog.langchain.com), được mô tả lần lượt là:

- Ô nhiễm ngữ cảnh - Khi ảo giác lọt vào ngữ cảnh
- Phân tán ngữ cảnh - Khi ngữ cảnh áp đảo dữ liệu huấn luyện
- Nhiễu loạn ngữ cảnh - Khi ngữ cảnh thừa thãi ảnh hưởng đến phản hồi
- Xung đột ngữ cảnh - Khi các phần của ngữ cảnh không nhất quán







### Context Poisoning: When a Hallucination Makes It into the Context

Ô nhiễm ngữ cảnh (Context Poisoning) chỉ việc ảo giác (hallucination, tức thông tin sai hoặc bịa đặt do mô hình sinh ra) hoặc các sai sót khác lọt vào cửa sổ ngữ cảnh và bị tham chiếu lặp đi lặp lại, từ đó nhúng thông tin sai vào, khiến hiệu năng của tác nhân (agent) trật bánh. Tình huống này sẽ "đầu độc" các phần then chốt, như mục tiêu hoặc bản tóm tắt, khiến mô hình cố chấp với những mục tiêu bất khả thi hoặc không liên quan, dẫn đến hành vi lặp lại, vô nghĩa.

### Context Distraction: When the Context Overwhelms the Training

Phân tán ngữ cảnh (Context Distraction) xảy ra khi ngữ cảnh tăng trưởng quá dài (ví dụ vượt quá 100 nghìn token), khiến mô hình quá phụ thuộc vào các chi tiết lịch sử mà bỏ qua tri thức tiền huấn luyện hoặc khả năng sinh ra giải pháp mới lạ. Điều này gây ra các hành động lặp lại thay vì giải quyết vấn đề một cách sáng tạo, và hiệu năng đã suy giảm ngay cả trước khi cửa sổ ngữ cảnh đầy tải.


Khi đối mặt với đầu vào hàng trăm nghìn token, mô hình không thể nhớ đều tất cả thông tin như một ổ đĩa cứng. Thực nghiệm phát hiện rằng đầu vào bản tinh gọn (chỉ vài trăm token) lại có biểu hiện tốt hơn đầu vào đầy đủ (hàng trăm nghìn token). Kết quả nghiên cứu cho thấy mô hình có biểu hiện trên bản tinh gọn tốt hơn rõ rệt so với bản đầy đủ. Điều này cho thấy khi đầu vào quá dài, quá nhiều nhiễu, thì ngay cả mô hình tiên tiến nhất cũng rất khó nắm bắt được thông tin then chốt.

### Context Confusion: When Superfluous Context Influences the Response

Nhiễu loạn ngữ cảnh (Context Confusion) chỉ việc thông tin không liên quan hoặc thừa thãi (như định nghĩa công cụ dư thừa) bị đưa vào ngữ cảnh, buộc mô hình phải cân nhắc nó, từ đó sinh ra phản hồi kém tối ưu. Ngay cả khi nội dung thừa là vô hại, nó cũng làm loãng sự tập trung và giảm chất lượng.
Trong hội thoại và tài liệu thực tế, thường tồn tại "nhiễu" có ngữ nghĩa tương tự nhưng không liên quan. Trong ngữ cảnh ngắn, mô hình có thể phân biệt được, nhưng trong ngữ cảnh dài lại dễ bị dẫn dắt sai hơn. Điều này đòi hỏi phải có người thực hiện việc sàng lọc và khử nhiễu ngữ cảnh, để mô hình tập trung vào thông tin thực sự liên quan. Trong ngữ cảnh dài, mô hình không chỉ phải tìm ra thông tin liên quan, mà còn phải phân biệt được "đâu mới là cây kim (needle) đúng, đâu chỉ là thứ gây nhiễu".

### Context Clash: When Parts of the Context Disagree

Xung đột ngữ cảnh (Context Clash) là dạng nghiêm trọng hơn của nhiễu loạn, chỉ việc các thông tin trong ngữ cảnh mâu thuẫn với nhau (như công cụ hoặc sự kiện mới mâu thuẫn với nội dung hiện có), từ đó phá hoại suy luận, thường vì mô hình bị khóa chặt vào các giả định ban đầu. Điều này có tính phá hoại lớn hơn so với chỉ đơn thuần là không liên quan: "This is a more problematic version of Context Confusion: the bad context here isn't irrelevant, it directly conflicts with other information in the prompt." Trong các tương tác nhiều bước, sai sót ban đầu sẽ lan truyền, mô hình phụ thuộc vào những tiền đề có khiếm khuyết.


 Thiếu độ tin cậy "kiểu máy tính"
Chúng ta mong muốn LLM cho ra đầu ra có chất lượng nhất quán. Ngay cả với nhiệm vụ sao chép đơn giản nhất, mô hình cũng sẽ mắc lỗi khi đầu vào dài. Nó không phải là một bộ xử lý ký hiệu từng chữ từng bit, mà là một bộ sinh ngôn ngữ vận hành theo xác suất. Do đó không thể kỳ vọng nó xử lý ngữ cảnh dài một cách chính xác như cơ sở dữ liệu hay máy tính, mà buộc phải nhờ đến thiết kế có cấu trúc để bù đắp.




Do đó, việc quản lý cửa sổ ngữ cảnh hiệu quả và kỹ thuật ngữ cảnh là điều không thể thiếu.







## Chiến lược kỹ thuật ngữ cảnh

Phần trước đã đề cập ngữ cảnh phải đối mặt với rất nhiều thách thức, vậy làm thế nào để khắc phục chúng? Điều này phải dựa vào kỹ thuật ngữ cảnh. Trong đó, các chiến lược của kỹ thuật ngữ cảnh chủ yếu chia thành bốn loại: ghi (lưu trữ), chọn, nén và cô lập.

![alt text](./images/Extra02-figures/image-6.png)

### Ghi ngữ cảnh

<strong>Ghi ngữ cảnh</strong> nghĩa là lưu nó ở ngoài cửa sổ ngữ cảnh để hỗ trợ Agent thực hiện nhiệm vụ.
Chủ yếu chia thành hai loại:
- <strong>Bảng nháp tạm thời</strong>
Một không gian làm việc tạm thời, ghi lại suy luận trung gian của mô hình, giúp quá trình suy nghĩ trở nên hiển thị. Việc ghi chú qua "bảng nháp tạm thời" là một phương pháp lưu giữ thông tin lâu dài trong khi Agent thực hiện nhiệm vụ. Ý tưởng là lưu thông tin ở ngoài cửa sổ ngữ cảnh, để Agent có thể sử dụng được.
- <strong>Bộ nhớ</strong>
Agent kết hợp ngữ cảnh mới phát sinh (new context) với bộ nhớ đã có (existing memories), sau khi xử lý thì ghi thành bộ nhớ được cập nhật (updated memory).

![alt text](./images/Extra02-figures/image-8.png)



### Chọn ngữ cảnh

Khi lượng thông tin ngày càng lớn, việc chọn như thế nào còn quan trọng hơn việc lưu trữ như thế nào. Chọn ngữ cảnh chính là ở mỗi lần gọi mô hình, từ tất cả các nguồn thông tin sẵn có, chọn ra phần thực sự liên quan để đưa vào cửa sổ.

Các ngữ cảnh cụ thể có thể chọn gồm:

- <strong>Bảng nháp tạm thời (Scratchpad)</strong>: chính là bảng nháp tạm thời đã đề cập ở trên, đóng vai trò là không gian "bộ nhớ làm việc" của mô hình, dùng để ghi lại quá trình suy luận, kết quả trung gian và các bước suy nghĩ. Trong các nhiệm vụ nhiều bước, mô hình có thể ghi trạng thái suy luận hiện tại, các nhiệm vụ con đã hoàn thành, các vấn đề chờ xử lý, v.v. vào bảng nháp tạm thời, thuận tiện cho việc tham chiếu và điều chỉnh chiến lược ở các bước sau.

- <strong>Bộ nhớ (Memory)</strong>: bao gồm hai tầng là bộ nhớ ngắn hạn và bộ nhớ dài hạn. Bộ nhớ ngắn hạn lưu giữ lịch sử hội thoại và thông tin ngữ cảnh trong phiên hiện tại, đảm bảo tính liền mạch của hội thoại; còn bộ nhớ dài hạn lưu trữ sở thích người dùng, mẫu hình tương tác lịch sử, cài đặt cá nhân hóa và các thông tin bền vững xuyên phiên khác, giúp mô hình cung cấp trải nghiệm dịch vụ cá nhân hóa và nhất quán hơn.

- <strong>Công cụ (Tools)</strong>: Trong hệ thống Agent, bản thân công cụ chính là một loại ngữ cảnh. Khi mô hình gọi API, plugin hoặc hàm bên ngoài, nó phải hiểu mô tả của công cụ (bao gồm thuyết minh chức năng, yêu cầu tham số, định dạng trả về, v.v.), và trong tình huống phù hợp chọn đúng công cụ. Kết quả phản hồi sau khi gọi công cụ cũng sẽ trở thành đầu vào ngữ cảnh mới, chỉ dẫn cho quyết định bước tiếp theo của mô hình. Tính khả dụng của công cụ, trạng thái thực thi, lịch sử gọi đều là thông tin ngữ cảnh quan trọng.

- <strong>Tri thức (Knowledge)</strong>: chủ yếu chỉ cơ sở tri thức bên ngoài trong RAG (sinh tăng cường bằng truy xuất). Bao gồm dữ liệu có cấu trúc (như bảng cơ sở dữ liệu), tài liệu phi cấu trúc (như tài liệu kỹ thuật, sổ tay sản phẩm), kết quả truy xuất ngữ nghĩa trong cơ sở dữ liệu vector, v.v. Những tri thức bên ngoài này bù đắp cho hạn chế về tính thời sự của dữ liệu huấn luyện và độ phủ tri thức chưa đủ của mô hình, thông qua việc truy xuất động thông tin liên quan để tăng cường độ chính xác và tính chuyên nghiệp trong câu trả lời của mô hình.

### Nén ngữ cảnh




![alt text](./images/Extra02-figures/image-9.png)


Nén ngữ cảnh liên quan đến việc chỉ giữ lại các token cần thiết để thực hiện nhiệm vụ, thông qua việc giảm thông tin dư thừa để tối ưu hóa hiệu suất sử dụng cửa sổ ngữ cảnh.

#### Tóm tắt ngữ cảnh

<strong>Tóm tắt hội thoại:</strong>
Trong các tương tác nhiều lượt kéo dài, việc giữ lại đầy đủ toàn bộ lịch sử hội thoại sẽ nhanh chóng tiêu tốn cửa sổ ngữ cảnh. Thông qua kỹ thuật tóm tắt hội thoại, có thể nén các lượt hội thoại ban đầu thành dạng tóm tắt ngắn gọn, giữ lại thông tin then chốt (như sở thích người dùng, quyết định quan trọng, vấn đề chờ giải quyết, v.v.), đồng thời loại bỏ nội dung xã giao dư thừa và lặp lại. Như vậy vừa có thể duy trì tính liền mạch của hội thoại, vừa chừa đủ không gian cho các tương tác mới.

<strong>Tóm tắt công cụ:</strong>
Việc gọi công cụ thường trả về một lượng lớn dữ liệu thô (như phản hồi API đầy đủ, kết quả truy vấn cơ sở dữ liệu, v.v.). Thông qua tóm tắt công cụ, có thể trích xuất và giữ lại các trường kết quả liên quan nhất, lọc bỏ metadata, thông tin gỡ lỗi và các nội dung không cần thiết khác. Ví dụ, API thời tiết có thể trả về các tham số khí tượng chi tiết, nhưng sau khi tóm tắt chỉ giữ lại thông tin cốt lõi như nhiệt độ, tình trạng thời tiết, giảm đáng kể lượng token tiêu thụ.

#### Cắt tỉa ngữ cảnh

<strong>Cắt tỉa dựa trên quy tắc:</strong>
Có thể sử dụng phương pháp heuristic mã hóa cứng để chủ động xóa ngữ cảnh lỗi thời hoặc ưu tiên thấp. Các chiến lược thường gặp gồm:
- Xóa các tin nhắn cũ hơn khỏi lịch sử hội thoại, giữ lại N lượt hội thoại gần nhất
- Loại bỏ bản ghi các nhiệm vụ con đã hoàn thành, chỉ giữ lại thông tin liên quan đến nhiệm vụ hiện tại
- Xóa dữ liệu tạm thời đã hết hạn hoặc kết quả gọi công cụ đã mất hiệu lực

<strong>Cắt tỉa thông minh:</strong>
Phương pháp cao cấp hơn có thể dựa trên điểm số độ liên quan để chọn động những đoạn ngữ cảnh nào cần giữ lại. Thông qua tính toán độ tương đồng ngữ nghĩa hoặc chấm điểm mức độ quan trọng, ưu tiên giữ lại thông tin liên quan nhất đến nhiệm vụ hiện tại, tự động loại bỏ các nội dung lịch sử có độ liên quan thấp.


### Cô lập ngữ cảnh

Cô lập ngữ cảnh liên quan đến việc tách ngữ cảnh ra để hỗ trợ Agent thực hiện nhiệm vụ.

#### Kiến trúc đa Agent

![alt text](./images/Extra02-figures/image-10.png)

<strong>Tách biệt mối quan tâm:</strong>
Chia một nhiệm vụ lớn phức tạp thành nhiều nhiệm vụ con độc lập, mỗi nhiệm vụ con do một Agent chuyên trách phụ trách. Thiết kế này tuân theo nguyên tắc trách nhiệm đơn nhất, khiến mỗi Agent tập trung vào một lĩnh vực cụ thể, nâng cao khả năng bảo trì và khả năng mở rộng của toàn hệ thống.

<strong>Đặc tính cô lập của Agent:</strong>
Mỗi Agent con sở hữu tài nguyên và cấu hình độc lập:
- <strong>Bộ công cụ chuyên dụng</strong>: Mỗi Agent chỉ có thể truy cập những công cụ cụ thể cần thiết để hoàn thành nhiệm vụ của nó, tránh việc công cụ tràn lan dẫn đến khó khăn trong lựa chọn
- <strong>Chỉ dẫn hệ thống độc lập</strong>: System prompt được tùy chỉnh cho nhiệm vụ cụ thể, làm rõ định vị vai trò và quy tắc hành xử của Agent
- <strong>Cửa sổ ngữ cảnh được cô lập</strong>: Mỗi Agent duy trì không gian ngữ cảnh riêng, không can thiệp lẫn nhau, tránh thông tin không liên quan gây ô nhiễm

<strong>Cơ chế cộng tác của Agent:</strong>
Nhiều Agent giao tiếp và truyền dữ liệu với nhau qua các giao diện rõ ràng, Agent chủ điều khiển hoặc tầng định tuyến chịu trách nhiệm phân bổ nhiệm vụ và tích hợp kết quả, tạo thành luồng công việc phối hợp.

#### Cô lập môi trường thực thi

![alt text](./images/Extra02-figures/image-11.png)

<strong>Tách biệt ngữ cảnh và thực thi:</strong>
Cô lập môi trường thực thi mã khỏi cửa sổ ngữ cảnh của LLM, LLM không cần trực tiếp tiếp xúc với toàn bộ dữ liệu đầu ra thô của tất cả công cụ.

<strong>Thiết kế tầng xử lý:</strong>
Thêm một tầng xử lý giữa việc thực thi công cụ và LLM:
- Công cụ được thực thi trong môi trường sandbox độc lập, tạo ra đầu ra thô
- Tầng xử lý lọc, chuyển đổi và tóm tắt kết quả thô
- Chỉ truyền thông tin then chốt đã tinh chế vào ngữ cảnh của LLM

Sự cô lập này vừa nâng cao tính an toàn, vừa giảm lượng token tiêu thụ, khiến LLM có thể tập trung vào quyết định cấp cao thay vì xử lý chi tiết cấp thấp.




### Tổng kết

Bốn hành động của kỹ thuật ngữ cảnh — ghi, chọn, nén, cô lập — không phải là những kỹ xảo rời rạc, mà là một bộ phương pháp hệ thống.
Chúng lần lượt giải quyết các vấn đề mất thông tin, thông tin dư thừa, quá tải thông tin và xung đột thông tin.
Khi bốn chiến lược này được thực thi một cách hệ thống, Agent có thể vận hành ổn định trong môi trường phức tạp.


## Triển khai kỹ thuật ngữ cảnh


Sử dụng LangSmith và LangGraph để thực hiện kỹ thuật ngữ cảnh, nội dung cụ thể của phần này có thể tham khảo Chương 9.



## Tổng kết và suy ngẫm: RAG is Dead?

![alt text](./images/Extra02-figures/image.png)


Jeff chủ yếu phê phán việc RAG truyền thống gán ghép cưỡng bức ba khái niệm khác nhau là "Truy xuất (Retrieval), Tăng cường (Augmented), Sinh (Generation)" lại với nhau, dẫn đến sự hỗn loạn về khái niệm và mơ hồ trong thực hành. Nhìn nhận lại RAG từ góc độ kỹ thuật ngữ cảnh, ta có thể phân tách nó thành các bước rõ ràng hơn:

<strong>RAG truyền thống vs Góc độ kỹ thuật ngữ cảnh (RAG cao cấp):</strong>

| Giai đoạn | RAG truyền thống | Phương pháp kỹ thuật ngữ cảnh |
|------|---------|----------------|
| <strong>Truy xuất</strong> | Tìm kiếm độ tương đồng vector đơn giản | Truy xuất lai: kết hợp nhiều chiến lược như truy xuất vector, khớp từ khóa, sắp xếp lại thứ tự |
| <strong>Lọc</strong> | Thường thiếu hoặc sơ sài | Lọc thông minh: loại bỏ nội dung dư thừa, lỗi thời hoặc không liên quan đến nhiệm vụ |
| <strong>Sắp xếp</strong> | Dựa trên một điểm số tương đồng duy nhất | Sắp xếp đa chiều: cân nhắc các yếu tố như độ liên quan, độ mới, độ đáng tin cậy, ưu tiên đưa vào thông tin then chốt nhất |
| <strong>Đánh giá</strong> | Thiếu đánh giá hệ thống hóa | Xây dựng tập dữ liệu vàng, đánh giá định lượng chất lượng truy xuất, độ chính xác câu trả lời và hiệu suất sử dụng ngữ cảnh |

<strong>Cải tiến cốt lõi:</strong>
- <strong>Đa dạng hóa chiến lược truy xuất</strong>: Không còn phụ thuộc vào truy xuất vector đơn nhất, mà tùy theo đặc điểm nhiệm vụ kết hợp sử dụng các kỹ thuật như truy xuất dày (dense retrieval), truy xuất thưa (sparse retrieval), sắp xếp lại theo ngữ nghĩa (semantic reranking)
- <strong>Ưu tiên chất lượng ngữ cảnh</strong>: Nhấn mạnh rằng thứ đưa vào LLM không phải "càng nhiều càng tốt", mà "càng chuẩn xác càng tốt", thông qua lọc và sắp xếp để đảm bảo ngữ cảnh có chất lượng cao
- <strong>Tối ưu hóa vòng khép kín</strong>: Thông qua tập dữ liệu đánh giá để liên tục lặp lại tối ưu hóa chiến lược truy xuất, quy tắc lọc và thuật toán sắp xếp, hình thành một quy trình công trình hóa có thể đo lường, có thể cải thiện

Góc nhìn này biến RAG từ một quy trình hộp đen thành một vấn đề kỹ thuật ngữ cảnh có thể phân tách, có thể tối ưu hóa, khiến nó mang tính khả thi và khả năng mở rộng cao hơn.

Do đó, kỹ thuật ngữ cảnh vừa là một thực hành công trình có hệ thống, vừa là một nghệ thuật cần cân nhắc đánh đổi. Nó đòi hỏi chúng ta phán đoán chính xác 4 câu hỏi sau trong biển thông tin khổng lồ:

- <strong>Write (Ghi)</strong> —— Những thông tin nào nên được đưa vào ngữ cảnh?
- <strong>Select (Chọn)</strong> —— Những nội dung nào liên quan nhất và cần thiết nhất?
- <strong>Compress (Nén)</strong> —— Những nội dung nào có thể tóm tắt hoặc đơn giản hóa?
- <strong>Isolate (Cô lập)</strong> —— Những nội dung nào cần tách ra không gian độc lập?

Chỉ khi hiểu được những câu hỏi này, mới có thể thực hiện kỹ thuật ngữ cảnh hiệu quả, đạt được sự kết hợp hoàn hảo giữa nghệ thuật và công trình.

![alt text](./images/Extra02-figures/image-12.png)


## Tài liệu tham khảo


沧海九粟. 上下文工程：优化 Agent 效能的关键技术[EB/OL]. (2025-07-10)[2025-10-21]. https://www.bilibili.com/video/BV1w3GNzeEHb/?spm_id_from=333.1387.upload.video_card.click&vd_source=0f47ed6b43bae0b240e774a8fd72e3e4


Drew Breunig. How Long Contexts Fail[EB/OL]. (2025-06-22)[2025-10-21]. https://www.dbreunig.com/2025/06/22/how-contexts-fail-and-how-to-fix-them.html?ref=blog.langchain.com


Latent.Space, Jeff Huber, Swyx. RAG is Dead, Context Engineering is King[EB/OL]. (2025-08-19)[2025-10-21]. https://www.latent.space/p/chroma

万字拆解. RAG已死吗？上下文工程（context engineer）为何为王？[EB/OL]. (2025-09-03)[2025-10-21]. https://www.woshipm.com/ai/6264065.html
