# Extra07 - Cấu hình môi trường

> Phần này sẽ hướng dẫn bạn cấu hình môi trường đầy đủ cần thiết để chạy FirstAgentTest.py. Đoạn code này triển khai một trợ lý du lịch thông minh, minh họa mô hình triển khai Agent dựa trên việc gọi công cụ (tool calling).

## Một, Yêu cầu môi trường

### 1.1 Yêu cầu phiên bản Python

- **Python 3.10+** (khuyến nghị sử dụng Python 3.10 hoặc phiên bản cao hơn)
- Hệ điều hành được hỗ trợ: Windows, macOS, Linux

### 1.2 Giải thích về đoạn code mục tiêu

Mục tiêu của chúng ta là chạy thành công file `code\chapter1\FirstAgentTest.py` của dự án. Đoạn code này triển khai:

- Chức năng trợ lý du lịch thông minh
- Công cụ tra cứu thời tiết (dựa trên wttr.in API)
- Công cụ gợi ý địa điểm tham quan (dựa trên Tavily Search API)
- Gọi LLM tương thích OpenAI
- Luồng thực thi Agent theo mô thức ReAct

## Hai, Cấu hình API

### 2.1 Cấu hình API mô hình ngôn ngữ lớn

#### Lựa chọn một: AIHubmix API (khuyến nghị)

AIHubmix là một nền tảng tổng hợp mô hình AI đặt tại bang Delaware, Hoa Kỳ, tích hợp các mô hình ngôn ngữ lớn chủ đạo trên thị trường, các mô hình mới phát hành thường có thể sử dụng được trong vòng một tuần. Nền tảng này kết nối trực tiếp với API gốc của các nhà cung cấp dịch vụ đám mây lớn (như OpenAI qua Azure, Anthropic qua AWS, Google qua giao diện chính thức, v.v.), triển khai theo kiến trúc cụm (cluster) của Google Cloud tại Hoa Kỳ, có năng lực cân bằng tải đa node (multi-node load balancing), thể hiện xuất sắc về mặt ổn định và tốc độ phản hồi.

> Hạn mức miễn phí mà nền tảng cung cấp có thể đáp ứng nhu cầu học tập của chúng ta.

1. **Vào trang web chính thức AIHubmix**

   Dùng trình duyệt truy cập [trang web chính thức AIHubmix](https://aihubmix.com/?aff=Igcn/)

   ![image1](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra07-figures/image1.png)

2. **Hoàn tất đăng ký tài khoản**

   Lần đầu sử dụng cần đăng ký tài khoản. Nhấn vào nút đăng ký ở góc trên bên phải, hỗ trợ đăng ký bằng email hoặc số điện thoại.

3. **Duyệt các mô hình khả dụng**

   Sau khi đăng ký thành công, truy cập [Trung tâm mô hình](https://aihubmix.com/models) để xem tất cả các mô hình khả dụng. Trong điều kiện lọc, chọn nhãn `免费` (miễn phí) để xem danh sách các mô hình miễn phí mà nền tảng cung cấp. Khuyến nghị chọn `coding-glm-4.7-free` hoặc các mô hình miễn phí khác tương thích định dạng OpenAI.

   ![image2](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra07-figures/image2.png)

4. **Lấy chứng chỉ API**

   Vào trang [Quản lý khóa API](https://console.aihubmix.com/token), hệ thống mặc định sẽ sinh ra một khóa khả dụng. Bạn cũng có thể nhấn nút `创建 Key` (Tạo Key) để tùy chỉnh tên khóa và sinh khóa mới.

   ![image3](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra07-figures/image3.png)
   
   Vui lòng lưu giữ cẩn thận các thông tin cấu hình sau:
   - API Key: `your_api_key`
   - Base URL: `https://aihubmix.com/v1`
   - Mô hình khuyến nghị: `coding-glm-4.7-free`





#### Lựa chọn hai: ModelScope

ModelScope là nhà cung cấp dịch vụ mô hình lớn hàng đầu tại Trung Quốc, cung cấp dịch vụ API với tỷ lệ hiệu năng/giá tốt. Ở đây chúng ta lấy Qwen làm ví dụ, bạn có thể lấy từ [ModelScope](https://modelscope.cn/docs/model-service/API-Inference/intro), nó cung cấp API định dạng tương thích (OpenAI) miễn phí của dòng Qwen, mỗi ngày miễn phí 2000 lượt gọi.

Vui lòng đảm bảo bạn có một tài khoản ModelScope đã đăng ký hợp lệ và có thể sử dụng. Để sinh API KEY riêng của bạn, có thể tham khảo hình minh họa của chúng tôi.

![image4](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra07-figures/image4.png)

![image5](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra07-figures/image5.png)

Token SDK trong hình chính là API KEY của chúng ta.

> Lưu ý, cần vào **dịch vụ mô hình** (模型服务) liên kết trước với [tài khoản Alibaba Cloud](https://modelscope.cn/docs/accounts/aliyun-binding-and-authorization), nếu không api sẽ hiển thị không thể sử dụng

**Phạm vi mô hình có thể chọn**

Trong [kho mô hình](https://modelscope.cn/models?filter=inference_type&page=1) của ModelScope, chọn suy luận API-Inference, các mô hình bên trong đều có thể chọn, chúng ta có thể trải nghiệm mô hình Llama-70B mới nhất được chưng cất dữ liệu (data distillation) từ DeepSeek-R1.

![image6](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra07-figures/image6.png)

Định dạng cuối cùng cần thiết giống với thông tin cấu hình của AIHubmix (Key, URL, tên mô hình)



### 2.2 Cấu hình Tavily Search API

Tavily là một API tìm kiếm được thiết kế chuyên cho các ứng dụng AI, dùng cho chức năng gợi ý địa điểm tham quan.

1. **Truy cập nền tảng Tavily**

   Mở trình duyệt, truy cập [Tavily](https://tavily.com/)

   ![image7](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra07-figures/image7.png)

2. **Đăng ký và lấy khóa API**

   ![image8](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra07-figures/image8.png)

   1. Đăng ký tài khoản
   2. Lấy API Key trong bảng điều khiển (console)
   3. Ghi lại API Key: `your_tavily_key`

## Ba, Cấu hình môi trường Python

### 3.1 Cài đặt Python (nếu chưa cài)

**Người dùng Windows:**
1. Truy cập [trang web chính thức Python](https://www.python.org/downloads/)
2. Tải phiên bản Python 3.10+
3. Khi cài đặt hãy tích chọn "Add Python to PATH"

**Người dùng macOS:**
```bash
# Cài đặt bằng Homebrew
brew install python@3.10
```

**Người dùng Linux:**
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install python3.10 python3.10-pip python3.10-venv

# CentOS/RHEL
sudo yum install python3.10 python3.10-pip
```

### 3.2 Xác minh cài đặt Python

```bash
python --version
# hoặc
python3 --version
```

Đảm bảo hiển thị Python 3.10 hoặc phiên bản cao hơn.

## Bốn, Cấu hình môi trường dự án

### 4.1 Tạo môi trường ảo (virtual environment) (khuyến nghị)

```bash
# Vào thư mục dự án
cd "hello-agents"

# Tạo môi trường ảo
python -m venv venv

# Kích hoạt môi trường ảo
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate
```

### 4.2 Cài đặt các gói phụ thuộc (dependencies)

```bash
# Cài đặt các phụ thuộc cốt lõi
pip install requests>=2.31.0
pip install tavily-python>=0.3.0
pip install openai>=1.0.0

# Tùy chọn: cài đặt các gói thường dùng khác
pip install python-dotenv>=1.0.0
```

### 4.3 Cấu hình biến môi trường

#### Cách một: Sử dụng file .env (khuyến nghị)

Tạo file `.env` ở thư mục gốc của dự án:

```bash
# Tạo file .env ở thư mục gốc của dự án
touch .env  # Linux/macOS
# Hoặc tạo thủ công trong Windows
```

Chỉnh sửa file `.env`, thêm nội dung sau:

```env
# Cấu hình Tavily API
TAVILY_API_KEY=your_tavily_api_key

# Cấu hình API mô hình ngôn ngữ lớn (chọn một trong hai)
# Lựa chọn một: AIHubmix
OPENAI_API_KEY=your_aihubmix_api_key
OPENAI_BASE_URL=https://aihubmix.com/v1
MODEL_NAME=xxxx

# Lựa chọn hai: Modelscope
# OPENAI_API_KEY=your_modelscope_api_key
# OPENAI_BASE_URL=https://api-inference.modelscope.cn/v1/
# MODEL_NAME=xxxx
```

#### Cách hai: Biến môi trường của hệ thống

Sau đây là phương án biến môi trường dài hạn, cũng có thể nạp tạm thời trong terminal.

**Windows:**
1. Nhấn chuột phải "This PC" (此电脑) → "Properties" (属性) → "Advanced system settings" (高级系统设置)
2. Nhấn "Environment Variables" (环境变量)
3. Trong "User variables" (用户变量) thêm:
   - `TAVILY_API_KEY`: `your_tavily_api_key`

**macOS/Linux:**
```bash
# Chỉnh sửa ~/.bashrc hoặc ~/.zshrc
export TAVILY_API_KEY="your_tavily_api_key"

# Làm cho cấu hình có hiệu lực
source ~/.bashrc
```

## Năm, Cấu hình code

### 5.1 Chỉnh sửa cấu hình FirstAgentTest.py

Mở file `code/chapter1/FirstAgentTest.py`, tìm phần cấu hình ở dòng 143-148:

```python
# --- 1. 配置LLM客户端 ---
# 请根据您使用的服务，将这里替换成对应的凭证和地址
API_KEY = "YOUR_API_KEY"
BASE_URL = "YOUR_BASE_URL"
MODEL_ID = "YOUR_MODEL_ID"
os.environ['TAVILY_API_KEY'] = "YOUR_TAVILY_API_KEY"
```

**Thay bằng cấu hình thực tế của bạn:**

#### Ví dụ cấu hình khi dùng AIHubmix:
```python
API_KEY = "your_aihubmix_api_key"
BASE_URL = "https://aihubmix.com/v1"
MODEL_ID = "coding-glm-4.7-free"
os.environ['TAVILY_API_KEY'] = "YOUR_TAVILY_API_KEY"
```

## Sáu, Chạy và xác minh

### 6.1 Kiểm thử kết nối mạng

Trước tiên kiểm thử tính thông suốt của từng API:

```python
# 测试天气 API
import requests
response = requests.get("https://wttr.in/Beijing?format=j1")
print("天气API状态:", response.status_code)

# 测试 Tavily API
from tavily import TavilyClient
tavily = TavilyClient(api_key="your_tavily_key")
try:
    result = tavily.search("test", search_depth="basic")
    print("Tavily API 连接成功")
except Exception as e:
    print("Tavily API 错误:", e)

# 测试 LLM API - AIHubmix
from openai import OpenAI
client = OpenAI(
    api_key="your_aihubmix_api_key",
    base_url="https://aihubmix.com/v1"
)
try:
    response = client.chat.completions.create(
        model="coding-glm-4.7-free",
        messages=[{"role": "user", "content": "Hello"}],
        max_tokens=10
    )
    print("LLM API 连接成功:", response.choices[0].message.content)
except Exception as e:
    print("LLM API 错误:", e)

# 测试 LLM API - ModelScope（如果您使用的是 ModelScope，请取消注释并替换配置）
# from openai import OpenAI
# client = OpenAI(
#     api_key="your_modelscope_api_key",
#     base_url="https://api-inference.modelscope.cn/v1/"
# )
# try:
#     response = client.chat.completions.create(
#         model="Qwen/Qwen2.5-72B-Instruct",
#         messages=[{"role": "user", "content": "Hello"}],
#         max_tokens=10
#     )
#     print("LLM API 连接成功:", response.choices[0].message.content)
# except Exception as e:
#     print("LLM API 错误:", e)
```

### 6.2 Chạy chương trình hoàn chỉnh

```bash
# 确保在正确目录
cd "hello-agents\code\chapter1"

# 运行程序
python FirstAgentTest.py
```

### 6.3 Đầu ra mong đợi

Khi chương trình chạy thành công, bạn sẽ thấy đầu ra tương tự như sau:

```
用户输入: 你好，请帮我查询一下今天北京的天气，然后根据天气推荐一个合适的旅游景点。
========================================
--- 循环 1 ---

正在调用大语言模型...
大语言模型响应成功。
模型输出:
Thought: 用户想要查询北京的天气，然后根据天气情况推荐合适的旅游景点。我需要先调用get_weather工具查询北京的天气情况。
Action: get_weather(city="北京")

Observation: 北京当前天气：Clear，气温15摄氏度
========================================
--- 循环 2 ---

正在调用大语言模型...
大语言模型响应成功。
模型输出:
Thought: 现在我知道了北京的天气是晴朗的，气温15摄氏度，这是一个很适合户外活动的天气。接下来我需要根据这个天气情况推荐合适的旅游景点。
Action: get_attraction(city="北京", weather="Clear，气温15摄氏度")

Observation: 根据搜索，为您找到以下信息：...
========================================
任务完成，最终答案: 根据查询，北京今天天气晴朗，气温15摄氏度，非常适合户外游览。推荐您去...
```


## Bảy, Xử lý các vấn đề thường gặp

### 7.1 Vấn đề cài đặt phụ thuộc

**Vấn đề: pip cài đặt chậm**

Giải pháp: sử dụng nguồn mirror trong nước
```bash
# 临时使用清华镜像
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple requests tavily-python openai

# 永久配置镜像源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

**Vấn đề: ModuleNotFoundError**

Giải pháp:
```bash
# 确认虚拟环境已激活
# 重新安装缺失的包
pip install requests tavily-python openai python-dotenv
```

### 7.2 Vấn đề gọi API

**Vấn đề: Tavily API trả về lỗi**

Nguyên nhân có thể:
- API Key chưa được thiết lập đúng
- Hạn mức API đã dùng hết
- Vấn đề kết nối mạng

Giải pháp:
```python
# 检查环境变量
import os
print("TAVILY_API_KEY:", os.environ.get('TAVILY_API_KEY'))

# 测试 API 连接
from tavily import TavilyClient
client = TavilyClient(api_key="your_key")
result = client.search("test")
```



## Tám, Tổng kết

Sau khi hoàn tất cấu hình môi trường, khuyến nghị:

1. Hiểu cấu trúc code của FirstAgentTest.py
2. Thử chỉnh sửa System Prompt để quan sát hiệu quả
3. Thêm các hàm công cụ mới
4. Triển khai logic Agent phức tạp hơn

Làm theo các bước trong tài liệu này, bạn sẽ có thể chạy thành công đoạn code trợ lý du lịch thông minh, và hiểu được nguyên lý triển khai Agent dựa trên việc gọi công cụ.
