# Hướng dẫn thực chiến xây dựng Agent với Dify:<br>Xây dựng trợ lý cá nhân toàn năng từ con số 0 (hướng dẫn chi tiết từng bước)

<div align="center">
  <img src="https://github.com/Tasselszcx.png" width="80" height="80" style="border-radius: 50%;" />
  <br />
  <strong>Tác giả:</strong> <a href="https://github.com/Tasselszcx">Tasselszcx</a>
  <br />
  <em>Hướng dẫn nguyên gốc | Hướng dẫn chi tiết từng bước | Thực hành hoàn chỉnh</em>
</div>

## 1. Cài đặt các plugin cần thiết

Trước khi xây dựng Agent (tác nhân thông minh), bạn cần hoàn tất việc cài đặt các plugin cần thiết và cấu hình MCP. Như minh họa ở Hình 1, hãy làm theo chỉ dẫn bằng chữ trong hình để cài đặt từng bước các plugin cần cho chương này.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image1.jpg" alt="插件安装示意图" width="90%"/>
  <p>Hình 1 Sơ đồ minh họa cài đặt plugin</p>
</div>

## 2. Cấu hình MCP (Model Context Protocol)

Chúng ta sẽ không đi sâu vào nguyên lý chi tiết của MCP ở đây, mà tập trung trình bày cách sử dụng dịch vụ MCP được triển khai trên cloud. Trong ví dụ này, chúng ta dùng chợ MCP của cộng đồng ModelScope (魔搭社区) trong nước để minh họa, các bước cụ thể như sau:

<strong>(1) Truy cập cộng đồng ModelScope</strong>: [https://www.modelscope.cn/home](https://www.modelscope.cn/home)

<strong>(2) Đăng ký tài khoản và đăng nhập</strong>, như minh họa ở Hình 2

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image2.jpg" alt="ModelScope注册登录界面" width="90%"/>
  <p>Hình 2 Giao diện đăng ký và đăng nhập ModelScope</p>
</div>

<strong>(3) Truy cập trang cấu hình MCP của Amap (Gaode Map - 高德地图)</strong>
   - Sau khi đăng nhập, làm theo Hình 3, nhấp từng bước để vào trang cấu hình MCP của Amap
   - Trang hiển thị sẽ như Hình 4

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image3.jpg" alt="高德地图MCP入口指引" width="90%"/>
  <p>Hình 3 Hướng dẫn lối vào MCP của Amap</p>
</div>

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image4.jpg" alt="高德地图MCP配置页面" width="90%"/>
  <p>Hình 4 Trang cấu hình MCP của Amap</p>
</div>

<strong>(4) Truy cập Nền tảng mở Amap (Amap Open Platform)</strong>: [https://console.amap.com/dev/index](https://console.amap.com/dev/index)
   - Làm theo chỉ dẫn bằng chữ trong Hình 5 để tạo ứng dụng mới

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image5.jpg" alt="高德开放平台新建应用" width="90%"/>
  <p>Hình 5 Tạo ứng dụng mới trên Nền tảng mở Amap</p>
</div>

<strong>(5) Tạo api_key</strong>
   - Như minh họa ở Hình 6, tạo api_key từng bước
   - Nhập api_key vừa tạo vào ô viền đỏ ở Hình 4, khi đó sẽ hiển thị cấu hình thành công
   - Trang cấu hình thành công như Hình 7

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image6.jpg" alt="创建api_key步骤" width="90%"/>
  <p>Hình 6 Các bước tạo api_key</p>
</div>

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image7.jpg" alt="MCP配置成功页面" width="90%"/>
  <p>Hình 7 Trang cấu hình MCP thành công</p>
</div>

<strong>Đến đây, toàn bộ quá trình cấu hình MCP của Amap đã hoàn tất!</strong>

## 3. Thiết kế Agent và trình diễn hiệu quả

Trong ví dụ này, chúng ta sẽ tạo một trợ lý cá nhân toàn diện, bao gồm các mô-đun chức năng sau:

- Hỏi đáp đời sống hàng ngày
- Tối ưu và trau chuốt câu chữ (copywriting)
- Sinh nội dung đa phương thức (hình ảnh, video)
- Tích hợp công cụ MCP (Amap, gợi ý ẩm thực, tin tức thời sự)
- Truy vấn dữ liệu và phân tích trực quan hóa

Kiến trúc điều phối (orchestration) tổng thể của Agent được thể hiện ở Hình 8.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image8.jpg" alt="智能体编排架构图" width="90%"/>
  <p>Hình 8 Sơ đồ kiến trúc điều phối của Agent</p>
</div>

Dưới đây giới thiệu cách xây dựng Chatflow cho một Agent như vậy:

### （1）Tạo ứng dụng Chatflow trống

- Làm theo Hình 9 và Hình 10, tạo ứng dụng Chatflow trống từng bước

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image9.jpg" alt="创建Chatflow步骤1" width="90%"/>
  <p>Hình 9 Tạo Chatflow bước 1</p>
</div>

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image10.jpg" alt="创建Chatflow步骤2" width="90%"/>
  <p>Hình 10 Tạo Chatflow bước 2</p>
</div>

### （2）Tạo bộ phân loại câu hỏi (Question Classifier)

- Trước tiên tạo một bộ phân loại câu hỏi để phân loại các câu hỏi đầu vào
- Nội dung cần điền cho bộ phân loại như minh họa ở Hình 11

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image11.jpg" alt="问题分类器配置" width="80%"/>
  <p>Hình 11 Cấu hình bộ phân loại câu hỏi</p>
</div>

### （3）Triển khai mô-đun trợ lý đời sống hàng ngày

Đây là một mô-đun hội thoại cơ bản, cấu hình mô hình ngôn ngữ lớn (LLM) và công cụ thời gian, đóng vai trò là dịch vụ hỏi đáp thông dụng làm phương án dự phòng (fallback).

<strong>Giải thích cấu hình</strong>:
- Giải thích cấu hình và cách nối dây tham khảo Hình 12
- Các node trong flow cụ thể lần lượt là "Bắt đầu - Bộ phân loại câu hỏi - LLM - Trả lời trực tiếp"
- <strong>Về sau chúng ta sẽ trực tiếp dùng node flow để mô tả flow của từng mô-đun</strong>

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image12.jpg" alt="日常助手模块配置" width="90%"/>
  <p>Hình 12 Cấu hình mô-đun trợ lý đời sống hàng ngày</p>
</div>

<strong>system_prompt của node LLM như sau</strong>:
```
# Role: 日常问题咨询专家

## Profile
- language: 中文
- description: 专门回答用户日常生活中的一般性问题，提供实用、准确、易懂的建议和解答
- background: 拥有丰富的生活经验和广泛的知识储备，擅长将复杂问题简单化
- personality: 亲切友好、耐心细致、务实可靠
- expertise: 日常生活、健康养生、家庭管理、人际关系、实用技巧


## Skills

1. 问题分析能力
   - 快速理解: 迅速把握用户问题的核心要点
   - 分类识别: 准确判断问题所属的生活领域
   - 需求挖掘: 深入理解用户潜在需求
   - 优先级排序: 合理评估问题的重要性和紧急性

2. 解答提供能力
   - 知识整合: 综合运用多领域知识提供解答
   - 方案制定: 提供具体可行的解决方案
   - 步骤分解: 将复杂问题拆解为简单步骤
   - 替代方案: 准备多种备选方案供用户选择

3. 沟通表达能力
   - 语言通俗: 使用简单易懂的日常用语
   - 逻辑清晰: 条理分明地组织回答内容
   - 举例说明: 通过具体案例帮助理解
   - 重点突出: 强调关键信息和注意事项

## Rules

1. 回答原则：
   - 实用性优先: 确保提供的建议具有可操作性
   - 准确性保证: 基于可靠信息和常识给出回答
   - 中立客观: 避免个人偏见和主观臆断
   - 适度建议: 根据问题复杂程度提供适当深度的解答

2. 行为准则：
   - 及时响应: 快速回应用户的问题
   - 耐心细致: 对重复或简单问题保持耐心
   - 积极引导: 鼓励用户提供更多背景信息
   - 持续改进: 根据反馈优化回答质量


## Workflows

- 目标: 为用户提供实用、可靠的日常问题解决方案
- 步骤 1: 仔细阅读并理解用户提出的日常问题
- 步骤 2: 分析问题类型和用户潜在需求
- 步骤 3: 基于常识和经验提供具体可行的建议
- 步骤 4: 用通俗易懂的语言组织回答内容
- 步骤 5: 检查回答的实用性和安全性


## Initialization
作为日常问题咨询专家，你必须遵守上述Rules，按照Workflows执行任务。
```

<strong>Hiệu quả trình diễn</strong>:
Như minh họa ở Hình 13:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image13.png" alt="日常助手演示效果" width="80%"/>
  <p>Hình 13 Hiệu quả trình diễn trợ lý đời sống hàng ngày</p>
</div>

### （4）Triển khai mô-đun tối ưu câu chữ (copywriting)

Theo báo cáo dữ liệu của OpenAI, hơn 60% người dùng sử dụng ChatGPT cho các tác vụ liên quan đến tối ưu văn bản, bao gồm trau chuốt, chỉnh sửa, viết mở rộng, viết rút gọn, v.v. Vì vậy, tối ưu câu chữ là kịch bản nhu cầu tần suất cao, và chúng ta sẽ đưa nó vào làm mô-đun chức năng cốt lõi thứ hai.

<strong>Cấu hình cụ thể</strong>:
- Các node trong flow cụ thể lần lượt là "Bắt đầu - Bộ phân loại câu hỏi - LLM - Trả lời trực tiếp", giống như (3)

<strong>system_prompt của node LLM như sau</strong>:
```
# 一、 角色人设（Role）
你是一位专业的文案优化专家，拥有丰富的营销文案写作和优化经验，擅长提升文案的吸引力、转化率和可读性。你的视角是站在目标受众和营销目标的角度，专业度边界限于文案优化领域，不涉及技术实现或产品开发。

# 二、 背景（Background）
用户提供了一段原始文案，需要你对其进行优化，以提升其整体效果。背景信息包括：文案可能用于营销、品牌推广或信息传达等场景，但具体用途未详细说明。已知条件是用户希望文案更吸引人、清晰或具有说服力，但未提供原始文案内容，因此你需要基于通用优化原则工作。

# 三、 任务目标（Task）
- 分析并优化文案的结构、语言和风格，使其更符合目标受众的偏好。
- 提升文案的吸引力、可读性和转化潜力，确保信息传达清晰。
- 根据常见优化原则（如简洁性、情感共鸣、行动号召等）进行调整，不涉及内容重写，除非必要。
- 在保持核心信息的前提下，适当扩展和丰富文案内容，提供更全面的优化版本。

# 四、 限制提示（Limit）
- 避免改变原始文案的核心信息或意图，除非用户明确要求。
- 不要添加虚构或无关内容，确保优化基于逻辑和最佳实践。
- 避免使用过于技术性或专业术语，除非目标受众是专业人士。
- 不涉及对图片、布局或其他非文本元素的优化。

# 五、 输出格式要求（Example）
输出应为优化后的文案文本，结构清晰，语言流畅，内容详实。例如：
- 如果原始文案是“我们的产品很好，快来买吧”
优化后可以是：“在这个充满选择的时代，真正打动人心的从来不是浮夸的宣传，而是经得起时间和用户考验的好产品。我们的产品正是如此。它不仅在设计上注重细节与品质，更在功能上不断打磨与创新，只为给每一位用户带来更好的使用体验。无论是外观的质感，还是性能的稳定，我们始终坚持高标准严要求，力求让每一位选择我们的顾客都能感受到物超所值的惊喜。
我们深知，购买一款产品，不仅仅是一次简单的消费，更是一种对生活方式的选择。因此，我们从选材、工艺到售后服务的每一个环节，都倾注了满满的诚意与专业，用心守护您的每一次体验。无论您是追求实用、注重品质，还是想要与众不同的个性化，我们的产品都能为您提供理想的解决方案。
现在，就让我们用行动来证明一切。真正的好产品，不需要过多修饰，它本身就是最好的代言人。立即行动，选择我们，让品质改变生活，从此拥有与众不同的体验！”
- 输出应直接呈现优化内容，无需额外解释或注释，除非用户要求。请确保优化后的文案内容更加丰富和完整，优化后的文案文本须超过500字。
```


<strong>Hiệu quả trình diễn</strong>:
Như minh họa ở Hình 14:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image14.png" alt="文案优化演示效果" width="80%"/>
  <p>Hình 14 Hiệu quả trình diễn tối ưu câu chữ</p>
</div>

### （5）Mô-đun sinh nội dung đa phương thức (hình ảnh, video)

Sinh hình ảnh và video là một kịch bản ứng dụng tần suất cao khác. Cùng với sự tiến hóa của các mô hình như Doubao (豆包) sinh ảnh, Google Imagen, cũng như những đột phá về công nghệ sinh video như Kling (可灵), Google Veo 3, OpenAI Sora 2, chất lượng của việc sinh nội dung đa phương thức đã đạt đến mức có thể ứng dụng thực tế.

<strong>Cấu hình sinh hình ảnh</strong>:
- Ví dụ này sử dụng plugin Doubao để thực hiện sinh hình ảnh và video
- Về quyền sinh hình ảnh, video của plugin Doubao và cách lấy api_key, vui lòng tham khảo bài blog này, giải thích cực kỳ rõ ràng, khuyến nghị xem trực tiếp phần 3 và 4 trong blog:
  [https://blog.csdn.net/sjkflw121150/article/details/148480867#:~:text=3.-,%E8%B0%83%E7%94%A8Doubao%E6%96%87%E7%94%9F%E5%9B%BE%E5%B7%A5%E5%85%B7,-%E8%B0%83%E7%94%A8%20Doubao](https://blog.csdn.net/sjkflw121150/article/details/148480867#:~:text=3.-,%E8%B0%83%E7%94%A8Doubao%E6%96%87%E7%94%9F%E5%9B%BE%E5%B7%A5%E5%85%B7,-%E8%B0%83%E7%94%A8%20Doubao)
- Tham khảo Hình 15, tạo phần flow sinh ảnh với Doubao
- Các node trong flow lần lượt là "Bắt đầu - Bộ phân loại câu hỏi - Doubao T2I - Trả lời trực tiếp"

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image15.jpg" alt="豆包生图flow配置" width="90%"/>
  <p>Hình 15 Cấu hình flow sinh ảnh với Doubao</p>
</div>

<strong>Hiệu quả sinh ảnh</strong>:
Như minh họa ở Hình 16:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image16.png" alt="豆包生图效果展示" width="80%"/>
  <p>Hình 16 Trình diễn hiệu quả sinh ảnh với Doubao</p>
</div>

<strong>Cấu hình sinh video</strong>:
- Sinh video tương tự như sinh hình ảnh, chỉ cần kích hoạt quyền text-to-video (văn bản sang video) trong Volcano Engine (火山引擎) là được, xem giải thích ở Hình 17
- Các node trong flow text-to-video lần lượt là "Bắt đầu - Bộ phân loại câu hỏi - Doubao T2V - Trả lời trực tiếp"

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image17.jpg" alt="文生视频权限开通" width="90%"/>
  <p>Hình 17 Kích hoạt quyền text-to-video</p>
</div>

<strong>Hiệu quả sinh video</strong>:
Như minh họa ở Hình 18:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image18.png" alt="豆包生视频效果展示" width="80%"/>
  <p>Hình 18 Trình diễn hiệu quả sinh video với Doubao</p>
</div>

### （6）Tích hợp công cụ MCP (Amap, gợi ý ẩm thực, tin tức thời sự)

Ở phần trước chúng ta đã hoàn thành việc cấu hình MCP, bây giờ sẽ tích hợp nó vào Agent.

<strong>Các bước cấu hình</strong> (tham khảo Hình 19):

1. Chọn node Agent hỗ trợ gọi MCP
2. Chọn chế độ ReAct
3. Thêm công cụ "Lấy timestamp"
4. Cấu hình dịch vụ MCP (tìm đến Hình 7, chọn chế độ SSE, xóa tiền tố mcp-server rồi sao chép các thông tin còn lại qua)
5. Điền prompt (câu lệnh gợi ý) tương ứng

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image19.jpg" alt="MCP工具集成配置步骤" width="90%"/>
  <p>Hình 19 Các bước cấu hình tích hợp công cụ MCP</p>
</div>

<strong>Cấu hình cụ thể</strong>:
- Thông tin điền cuối cùng cho node Agent có thể tham khảo Hình 20
- Các node trong flow gọi dịch vụ MCP lần lượt là "Bắt đầu - Bộ phân loại câu hỏi - Agent - Trả lời trực tiếp"

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image20.jpg" alt="Agent节点配置详情" width="50%"/>
  <p>Hình 20 Chi tiết cấu hình node Agent</p>
</div>

<strong>Trình diễn hiệu quả</strong>:
- Hiệu quả trợ lý Amap: như minh họa ở Hình 21

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image21.png" alt="高德助手效果展示" width="80%"/>
  <p>Hình 21 Trình diễn hiệu quả trợ lý Amap</p>
</div>

- Hiệu quả trợ lý ẩm thực: như minh họa ở Hình 22

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image22.png" alt="饮食助手效果展示" width="80%"/>
  <p>Hình 22 Trình diễn hiệu quả trợ lý ẩm thực</p>
</div>

- Hiệu quả trợ lý tin tức: như minh họa ở Hình 23

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image23.png" alt="新闻助手效果展示" width="50%"/>
  <p>Hình 23 Trình diễn hiệu quả trợ lý tin tức</p>
</div>

### （7）Mô-đun truy vấn và phân tích dữ liệu

<strong>Mô-đun truy vấn và phân tích dữ liệu</strong>

Xử lý dữ liệu là một trong những năng lực quan trọng của Agent. Mô-đun này minh họa cách kết nối cơ sở dữ liệu trong Dify để thực hiện truy vấn dữ liệu và phân tích trực quan hóa.

Trước tiên cài đặt plugin công cụ truy vấn dữ liệu, ví dụ này sử dụng plugin `rookie-text2data`. Điểm mấu chốt của truy vấn dữ liệu là cung cấp cho mô hình lớn cấu trúc bảng và thông tin trường (field) rõ ràng, để nó có thể sinh ra câu lệnh truy vấn SQL chính xác. Các cách làm phổ biến bao gồm:

- Cung cấp trực tiếp câu lệnh DDL của bảng dữ liệu
- Cung cấp mô tả về mối tương ứng giữa tên bảng và tên trường

Cấu hình thông tin kết nối cơ sở dữ liệu (địa chỉ IP, tên cơ sở dữ liệu, cổng, tài khoản, mật khẩu, v.v.), như minh họa ở Hình 24. Kết quả truy vấn cần được node mô hình lớn xử lý sắp xếp, chuyển đổi thành đầu ra ngôn ngữ tự nhiên dễ hiểu.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image24.png" alt="数据库配置" width="50%"/>
  <p>Hình 24 Cấu hình cơ sở dữ liệu</p>
</div>

<strong>Thiết lập prompt:</strong>
```
# 一、 角色人设（Role）
您是一位专业的数据查询师，擅长数据整理，具有清晰的逻辑思维和简洁表达能力。

# 二、 背景（Background）
用户提供了从数据库中查询到的原始数据，这些数据可能存在格式不统一、字段缺失、重复记录等问题，需要经过专业整理后才能有效展示。

# 三、 任务目标（Task）
1. 对原始数据进行归纳和整理
2. 按照正确的逻辑对数据进行分类和排序
3. 数据展示突出关键信息和数据洞察
4. 提供易于理解的数据展示

# 四、 限制提示（Limit）
1. 不得随意删除重要数据
2. 避免使用过于复杂或专业的统计术语
3. 不得篡改原始数据的真实值
4. 避免展示过多冗余信息，保持简洁明了
5. 不得泄露敏感数据或个人隐私信息

# 五、 输出格式要求（Example）
 数据概览：简要说明数据内容即可
```

Trình diễn hiệu quả như minh họa ở Hình 25:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image25.png" alt="数据查询助手效果" width="80%"/>
  <p>Hình 25 Trợ lý truy vấn dữ liệu</p>
</div>

<strong>Thiết lập prompt:</strong>
```
# 一、 角色人设（Role）
你是一位专业的数据分析师，具备数据整理、清洗和可视化能力，能够从原始数据中提取关键信息并转化为直观的可视化展示。

# 二、 背景（Background）
用户已从数据库中查询到一批原始数据，这些数据可能包含多个字段、存在缺失值或格式不一致的情况，需要经过整理后生成可视化图表。

# 三、 任务目标（Task）
#工作流程
1. 数据分析
按照合理的规则进行数据分析整理总结
2. 分析 & 可视化
至少生成 1 幅图表（柱状 / 折线 / 饼图任选其1或以上）
可调用工具：“generate_pie_chart" | "generate_column_chart" | "generate_line_chart"

# 四、 限制提示（Limit）
1. 避免使用过于复杂的图表类型，确保可视化结果易于理解
2. 不要忽略数据质量问题，必须进行必要的数据清洗
3. 避免在可视化中使用过多颜色或元素，保持简洁明了
4. 不要遗漏关键数据的标注和说明
5.必须进行总结和图表生成，不管数据多少

# 五、 输出格式要求（Example）
请按照以下格式输出：
1. 数据概况总结（不要输出字段名称，不要分点，一小段话就行）
2. 展示生成的图表
```

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra03-figures/image26.png" alt="数据分析助手效果" width="80%"/>
  <p>Hình 26 Trợ lý phân tích dữ liệu</p>
</div>

Điểm khác biệt duy nhất của phần trợ lý phân tích dữ liệu này là chúng ta đã thêm công cụ trực quan hóa dữ liệu, tức là các plugin công cụ sinh biểu đồ BI như "generate_pie_chart" | "generate_column_chart" | "generate_line_chart". Tin rằng ở phần trước mọi người đều đã cài đặt theo yêu cầu, nên có thể trực tiếp thêm và kích hoạt sử dụng, đồng thời bổ sung mô tả tương ứng như trong prompt ở trên là được. Phần này về sau mọi người có thể tự kết nối với SQL để thử nghiệm, nên sẽ không nói dông dài thêm nữa~

---

<strong>Đến đây, chúng ta đã hoàn thành một trợ lý cá nhân siêu Agent với chức năng toàn diện.</strong>

Trợ lý này bao trùm nhiều khía cạnh của cuộc sống:
- Khi cần quần áo mới, có thể để Doubao sinh thiết kế
- Trước khi ra ngoài, có thể để trợ lý Amap lên kế hoạch lộ trình
- Khi không biết ăn gì, có thể nhận gợi ý ẩm thực
- Khi muốn nắm tình hình học tập, có thể tiến hành phân tích dữ liệu

<strong>Agent này có thể xử lý đủ loại tác vụ trong công việc và cuộc sống, mong được thấy mọi người xây dựng thêm nhiều trợ lý Agent cá nhân sáng tạo hơn nữa.</strong>

## Tài liệu tham khảo
1. Cộng đồng ModelScope. https://www.modelscope.cn/home

2. Nền tảng mở Amap. https://console.amap.com/dev/index

3. sjkflw121150. Những cạm bẫy khi xây dựng trợ lý sinh ảnh AI bằng Dify! Blog CSDN. https://blog.csdn.net/sjkflw121150/article/details/148480867#:~:text=3.-,%E8%B0%83%E7%94%A8Doubao%E6%96%87%E7%94%9F%E5%9B%BE%E5%B7%A5%E5%85%B7,-%E8%B0%83%E7%94%A8%20Doubao
