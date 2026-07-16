# Chương 13 Trợ lý Du lịch Thông minh

Trong các chương trước, chúng ta đã xây dựng framework HelloAgents từ đầu, hiện thực các chức năng cốt lõi bao gồm nhiều mô thức Agent khác nhau, hệ thống công cụ (tool), cơ chế bộ nhớ (memory), giao tiếp qua protocol, và đánh giá hiệu năng. Bắt đầu từ chương này, chúng ta sẽ bước vào một giai đoạn hoàn toàn mới: **tích hợp toàn bộ kiến thức đã học để xây dựng các ứng dụng thực tế hoàn chỉnh.**

Bạn còn nhớ agent đầu tiên mà chúng ta xây dựng ở Chương 1 không? Đó là một trợ lý du lịch thông minh đơn giản, minh họa các nguyên lý cơ bản của vòng lặp `Thought-Action-Observation` (Suy nghĩ - Hành động - Quan sát). Trợ lý du lịch thông minh trong chương này sẽ là một dự án hoàn chỉnh, bao gồm các chức năng cốt lõi sau:

**(1) Lập kế hoạch hành trình thông minh**: Người dùng nhập điểm đến, ngày tháng, sở thích và các thông tin khác, hệ thống sẽ tự động tạo ra một kế hoạch hành trình hoàn chỉnh bao gồm các điểm tham quan, ăn uống và khách sạn.

**(2) Trực quan hóa trên bản đồ**: Đánh dấu vị trí các điểm tham quan trên bản đồ và vẽ tuyến đường tham quan, giúp hành trình trở nên rõ ràng, dễ nhìn.

**(3) Tính toán ngân sách**: Tự động tính toán chi phí vé, khách sạn, ăn uống và di chuyển, hiển thị chi tiết ngân sách.

**(4) Chỉnh sửa hành trình**: Hỗ trợ thêm, xóa và điều chỉnh các điểm tham quan, cập nhật bản đồ theo thời gian thực.

**(5) Chức năng xuất file**: Hỗ trợ xuất ra PDF hoặc hình ảnh, tiện cho việc lưu trữ và chia sẻ.

## 13.1 Tổng quan Dự án và Thiết kế Kiến trúc

### 13.1.1 Tại sao chúng ta cần một Trợ lý Du lịch Thông minh

Lập kế hoạch cho một chuyến đi vừa thú vị vừa gây bực bội. Bạn cần tìm kiếm thông tin về các điểm tham quan trên mạng, so sánh các hướng dẫn du lịch khác nhau, kiểm tra dự báo thời tiết, đặt khách sạn, tính toán ngân sách và lên kế hoạch tuyến đường. Quá trình này có thể mất vài giờ, thậm chí vài ngày. Và ngay cả sau khi bỏ ra ngần ấy thời gian, bạn cũng không chắc liệu hành trình đã lập có hợp lý không, có bỏ sót điểm tham quan quan trọng nào không, hay ngân sách có chính xác không.

Các phương pháp lập kế hoạch du lịch truyền thống có một số điểm khó chịu. Đầu tiên là **thông tin phân tán**. Thông tin về điểm tham quan nằm trên các trang du lịch, thông tin thời tiết nằm trên các trang thời tiết, thông tin khách sạn nằm trên các trang đặt phòng - bạn cần chuyển đổi qua lại giữa nhiều trang web và tự tổng hợp các thông tin này. Thứ hai là **thiếu cá nhân hóa**. Hầu hết các hướng dẫn du lịch đều mang tính chung chung và không xét đến sở thích cá nhân, giới hạn ngân sách, thời gian du lịch cùng nhiều yếu tố khác của bạn. Cuối cùng là **khó điều chỉnh**. Khi bạn muốn sửa hành trình, có thể bạn phải lập lại kế hoạch cho toàn bộ chuyến đi, bởi vì thứ tự các điểm tham quan, sắp xếp thời gian và ngân sách đều liên quan chặt chẽ với nhau.

Công nghệ AI mang lại những khả năng mới để giải quyết các vấn đề này. Hãy tưởng tượng bạn chỉ cần nói với hệ thống "Tôi muốn đi Bắc Kinh 3 ngày, thích lịch sử văn hóa, ngân sách tầm trung", và hệ thống có thể tự động tạo ra một kế hoạch hành trình hoàn chỉnh cho bạn, bao gồm mỗi ngày tham quan điểm nào, ăn ở đâu, ở khách sạn nào, và cần bao nhiêu ngân sách. Hơn nữa, kế hoạch này có thể điều chỉnh được - bạn có thể xóa những điểm tham quan mình không thích, điều chỉnh thứ tự tham quan, và hệ thống sẽ tự động cập nhật bản đồ và ngân sách.

Đây chính là trợ lý du lịch thông minh mà chúng ta muốn xây dựng. Nó không chỉ là một minh họa kỹ thuật, mà là một ứng dụng thực sự hữu ích. Thông qua dự án này, bạn sẽ học được cách áp dụng công nghệ AI vào các vấn đề thực tế, cách thiết kế hệ thống đa Agent (multi-agent), và cách xây dựng các ứng dụng Web hoàn chỉnh.

### 13.1.2 Tổng quan Kiến trúc Kỹ thuật

Hệ thống áp dụng **kiến trúc tách biệt front-end và back-end** cổ điển, được chia thành bốn tầng, như minh họa ở Hình 13.1:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-1.png" alt="" width="85%"/>
  <p>Hình 13.1 Kiến trúc Kỹ thuật của Trợ lý Du lịch Thông minh</p>
</div>

**(1) Tầng Front-end (Vue3+TypeScript)**: Chịu trách nhiệm tương tác với người dùng và hiển thị dữ liệu, bao gồm nhập biểu mẫu, hiển thị kết quả và trực quan hóa bản đồ.

**(2) Tầng Back-end (FastAPI)**: Chịu trách nhiệm định tuyến API, kiểm tra dữ liệu (validation) và logic nghiệp vụ.

**(3) Tầng Agent (HelloAgents)**: Chịu trách nhiệm phân rã tác vụ, gọi công cụ và tổng hợp kết quả. Bao gồm 4 Agent chuyên biệt.

**(4) Tầng Dịch vụ Bên ngoài**: Cung cấp dữ liệu và năng lực, bao gồm Amap API, Unsplash API và LLM API.

Luồng dữ liệu diễn ra như sau: Người dùng điền biểu mẫu ở front-end → Back-end kiểm tra dữ liệu → Gọi hệ thống agent → Các Agent lần lượt gọi các Agent tìm kiếm điểm tham quan, truy vấn thời tiết, gợi ý khách sạn, lập kế hoạch hành trình → Mỗi Agent gọi các API bên ngoài thông qua giao thức MCP → Tổng hợp kết quả và trả về front-end → Front-end kết xuất (render) và hiển thị.

Cấu trúc tham chiếu của dự án như sau, được cung cấp để dễ dàng định vị mã nguồn:
```
helloagents-trip-planner/
├── backend/                    # Backend code
│   ├── app/
│   │   ├── agents/            # Agent implementation
│   │   ├── api/               # API routes
│   │   ├── models/            # Data models
│   │   ├── services/          # Service layer
│   │   └── config.py          # Configuration file
│   └── requirements.txt       # Python dependencies
│
└── frontend/                   # Frontend code
    ├── src/
    │   ├── views/             # Page components
    │   ├── services/          # API services
    │   ├── types/             # Type definitions
    │   └── router/            # Route configuration
    └── package.json           # npm dependencies
```

Thiết kế kiến trúc chi tiết và luồng dữ liệu sẽ được giới thiệu trong các phần tiếp theo.

### 13.1.3 Trải nghiệm Nhanh: Chạy Dự án trong 5 Phút

Trước khi đi sâu vào các chi tiết hiện thực, hãy chạy thử dự án để xem hiệu quả cuối cùng. Bằng cách này, bạn sẽ có được sự hiểu biết trực quan về toàn bộ hệ thống.

**Yêu cầu môi trường:**

- Python 3.10 trở lên
- Node.js 16.0 trở lên
- npm 8.0 trở lên

**Lấy các API Key:**

Bạn cần chuẩn bị các API key sau:

- LLM API (OpenAI, DeepSeek, v.v.)
- Amap Web Service Key: Truy cập https://console.amap.com/ để đăng ký và tạo một ứng dụng
- Unsplash Access Key: Truy cập https://unsplash.com/developers để đăng ký và tạo một ứng dụng

Đặt tất cả các API key vào file `.env`.

Khởi động back-end:

```bash
# 1. Enter backend directory
cd helloagents-trip-planner/backend

# 2. Install dependencies
pip install -r requirements.txt

# 3. Configure environment variables
cp .env.example .env
# Edit .env file, fill in your API keys

# 4. Start backend service
uvicorn app.api.main:app --reload
# or
python run.py
```

Sau khi khởi động thành công, truy cập http://localhost:8000/docs để xem tài liệu API.

Mở một cửa sổ terminal mới:

```bash
# 1. Enter frontend directory
cd helloagents-trip-planner/frontend

# 2. Install dependencies
npm install

# 3. Start frontend service
npm run dev
```

Sau khi khởi động thành công, truy cập http://localhost:5173 để sử dụng ứng dụng.

Trải nghiệm các chức năng cốt lõi:

Đầu tiên, điền vào biểu mẫu trên trang chủ các thông tin: thành phố điểm đến, ngày du lịch, sở thích, ngân sách, loại phương tiện di chuyển và loại chỗ ở. Sau khi nhấn nút "Start Planning" (Bắt đầu Lập kế hoạch), hệ thống sẽ hiển thị thanh tiến trình đang tải và nhanh chóng tạo ra một trang kết quả, như minh họa ở Hình 13.2.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-2.png" alt="" width="85%"/>
  <p>Hình 13.2 Trang Tiến trình Lập kế hoạch của Trợ lý Du lịch</p>
</div>

Sau khi tải thành công, trang sẽ hiển thị rõ ràng tổng quan hành trình, chi tiết ngân sách, bản đồ các điểm tham quan, chi tiết hành trình hàng ngày và thông tin thời tiết, như minh họa ở Hình 13.3 và 13.4.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-3.png" alt="" width="85%"/>
  <p>Hình 13.3 Trang Hoàn thành Lập kế hoạch của Trợ lý Du lịch</p>
</div>

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-4.png" alt="" width="85%"/>
  <p>Hình 13.4 Trang Hoàn thành Lập kế hoạch của Trợ lý Du lịch</p>
</div>

Nếu người dùng cần điều chỉnh theo ý riêng, họ có thể nhấn nút "Edit Itinerary" (Chỉnh sửa Hành trình) để tự do điều chỉnh thứ tự các điểm tham quan hoặc xóa một số điểm tham quan nhất định, như minh họa ở Hình 13.5. Sau khi lập kế hoạch xong, thông qua menu thả xuống "Export Itinerary" (Xuất Hành trình), kế hoạch cuối cùng có thể được lưu dễ dàng thành file hình ảnh hoặc PDF để tiện tham khảo bất cứ lúc nào.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-5.png" alt="" width="85%"/>
  <p>Hình 13.5 Trang Hoàn thành Lập kế hoạch của Trợ lý Du lịch</p>
</div>

## 13.2 Thiết kế Mô hình Dữ liệu

### 13.2.1 Luồng Dữ liệu trong Ứng dụng Web

Khi xây dựng một trợ lý du lịch thông minh, chúng ta cần giải quyết một vấn đề cốt lõi: **Làm thế nào để biểu diễn và truyền tải dữ liệu kế hoạch du lịch?**

Chúng ta cần hiểu cách dữ liệu luân chuyển trong một ứng dụng Web hoàn chỉnh. Hãy tưởng tượng điều gì xảy ra khi người dùng nhấn nút "Start Planning" trong trình duyệt?

Dữ liệu biểu mẫu mà người dùng điền ở front-end (điểm đến, ngày tháng, ngân sách, v.v.) cần được gửi tới máy chủ back-end thông qua các HTTP request. Sau khi back-end nhận được dữ liệu, nó sẽ gọi hệ thống agent để xử lý. Các agent sau đó sẽ gọi các dịch vụ bên ngoài như Amap API và Unsplash API để lấy dữ liệu. Định dạng dữ liệu mà các API bên ngoài trả về là khác nhau - có nơi dùng `lng`, có nơi dùng `lon`, và có nơi dùng `longitude`. Cuối cùng, back-end cần trả dữ liệu đã xử lý về front-end, rồi front-end kết xuất nó thành trang mà người dùng nhìn thấy.

Trong quá trình này, dữ liệu trải qua nhiều lần biến đổi: Biểu mẫu front-end → HTTP request → Đối tượng Python ở back-end → Phản hồi API bên ngoài → Đối tượng Python ở back-end → HTTP response → Đối tượng TypeScript ở front-end → Hiển thị trên trang. Nếu không có một định dạng dữ liệu thống nhất, mỗi bước biến đổi đều có thể xảy ra lỗi. Đây chính là lý do vì sao chúng ta cần **mô hình dữ liệu (data model)**.

### 13.2.2 Từ Dictionary đến Mô hình Pydantic

Hãy bắt đầu từ nguyên mẫu đơn giản ở Chương 1. Trong nguyên mẫu đó, chúng ta đã dùng dictionary của Python để biểu diễn dữ liệu điểm tham quan:

```python
# Chapter 1 approach: using dictionaries
attraction = {
    "name": "Forbidden City",
    "location": {"lng": 116.397128, "lat": 39.916527},
    "price": 60
}

# Access data
lng = attraction["location"]["lng"]
```

Cách tiếp cận này tiện lợi ở giai đoạn nguyên mẫu, nhưng sẽ gặp nhiều vấn đề trong các dự án thực tế. Đầu tiên là vấn đề **tên trường không nhất quán**. Dữ liệu vị trí mà Amap API trả về là một chuỗi như `"116.397128,39.916527"`, cần được tách thủ công thành kinh độ và vĩ độ. Unsplash API có thể dùng `longitude` và `latitude`. Nếu chúng ta dùng dictionary ở khắp nơi trong mã, chúng ta phải xử lý những khác biệt này ở mọi chỗ.

Thứ hai là vấn đề **an toàn kiểu (type safety)**. Giả sử chúng ta vô tình gán `price` là một chuỗi `"60"`, điều này sẽ không báo lỗi ngay lập tức trong Python, nhưng sẽ gây ra vấn đề khi tính tổng ngân sách. Tệ hơn, loại lỗi này chỉ có thể được phát hiện lúc chạy (runtime), và thông báo lỗi có thể rất khó định vị.

Cuối cùng là vấn đề **khả năng bảo trì**. Khi chúng ta cần thêm một trường mới cho điểm tham quan (chẳng hạn `rating`), chúng ta phải sửa nhiều chỗ trong mã. Nếu bỏ sót chỗ nào đó, nó sẽ dẫn đến việc dữ liệu không nhất quán.

Pydantic cung cấp một giải pháp. Nó là một thư viện kiểm tra dữ liệu (data validation) của Python, cho phép chúng ta định nghĩa cấu trúc dữ liệu bằng các lớp (class) và tự động xử lý việc kiểm tra, chuyển đổi và tuần tự hóa (serialization). Hãy xem một ví dụ đơn giản:

```python
from pydantic import BaseModel, Field

class Location(BaseModel):
    longitude: float = Field(..., description="Longitude")
    latitude: float = Field(..., description="Latitude")

class Attraction(BaseModel):
    name: str
    location: Location
    ticket_price: int = 0

# Create object
attraction = Attraction(
    name="Forbidden City",
    location=Location(longitude=116.397128, latitude=39.916527),
    ticket_price=60
)

# Type-safe access
lng = attraction.location.longitude  # IDE will provide code completion
```

Cách tiếp cận này có một số lợi ích. Đầu tiên, nếu chúng ta truyền vào sai kiểu (chẳng hạn gán `ticket_price` là một chuỗi), Pydantic sẽ lập tức ném ra một ngoại lệ (exception) cho biết lỗi ở đâu. Thứ hai, IDE có thể cung cấp gợi ý hoàn thành mã (code completion) và kiểm tra kiểu dựa trên định nghĩa kiểu, giảm đáng kể lỗi chính tả. Cuối cùng, khi cần sửa cấu trúc dữ liệu, chúng ta chỉ cần sửa định nghĩa lớp, và tất cả những chỗ sử dụng lớp này sẽ tự động cập nhật.

### 13.2.3 Các Khái niệm Cốt lõi của Pydantic

Trước khi đi sâu vào việc thiết kế các mô hình dữ liệu của mình, hãy cùng tìm hiểu một số khái niệm cốt lõi của Pydantic. Nền tảng của Pydantic là lớp `BaseModel`, và tất cả các mô hình dữ liệu đều cần kế thừa từ lớp này. Mỗi trường có thể chỉ định một kiểu, và Pydantic sẽ tự động thực hiện việc kiểm tra và chuyển đổi kiểu.

Việc định nghĩa trường sử dụng hàm `Field`, hàm này có thể chỉ định giá trị mặc định, mô tả, quy tắc kiểm tra, v.v. `...` cho biết trường này là bắt buộc - nếu trường này không được cung cấp khi tạo đối tượng, Pydantic sẽ ném ra một ngoại lệ. Chúng ta cũng có thể dùng `Optional` để biểu thị trường tùy chọn, hoặc trực tiếp cung cấp giá trị mặc định.

```python
from pydantic import BaseModel, Field
from typing import Optional, List

class Attraction(BaseModel):
    name: str = Field(..., description="Attraction name")  # Required
    rating: float = Field(default=0.0, ge=0, le=5)  # Default value, range validation
    visit_duration: int = Field(default=60, gt=0)  # Greater than 0
    description: Optional[str] = None  # Optional field
```

Pydantic cũng hỗ trợ các mô hình lồng nhau (nested model) và danh sách (list). Chúng ta có thể dùng một mô hình khác làm kiểu của một trường trong một mô hình, cho phép chúng ta xây dựng các cấu trúc dữ liệu phức tạp. Ví dụ, một điểm tham quan chứa thông tin vị trí, và một hành trình chứa nhiều điểm tham quan.

```python
class DayPlan(BaseModel):
    date: str
    attractions: List[Attraction]  # Attraction list
    hotel: Optional[Hotel] = None  # Optional hotel information
```

Một trong những tính năng mạnh mẽ nhất là **bộ kiểm tra tùy chỉnh (custom validator)**. Đôi khi định dạng dữ liệu mà các API bên ngoài trả về không đáp ứng yêu cầu của chúng ta, và chúng ta có thể dùng decorator `field_validator` để tùy chỉnh logic kiểm tra và chuyển đổi. Ví dụ, nhiệt độ mà Amap trả về là một chuỗi như `"16°C"`, và chúng ta cần chuyển nó thành một số:

```python
from pydantic import field_validator

class WeatherInfo(BaseModel):
    temperature: int

    @field_validator('temperature', mode='before')
    def parse_temperature(cls, v):
        """Parse temperature string: "16°C" -> 16"""
        if isinstance(v, str):
            v = v.replace('°C', '').replace('℃', '').strip()
            return int(v)
        return v
```

Bộ kiểm tra này sẽ tự động thực thi trước khi tạo đối tượng, chuyển chuỗi thành số nguyên. Nhờ đó chúng ta không cần xử lý định dạng nhiệt độ thủ công ở mọi chỗ trong mã.

### 13.2.4 Thiết kế Mô hình Từ dưới lên (Bottom-Up)

Bây giờ hãy bắt đầu thiết kế các mô hình dữ liệu cho trợ lý du lịch thông minh. Một nguyên tắc thiết kế tốt là **từ dưới lên (bottom-up)**: đầu tiên định nghĩa các mô hình cơ bản nhất, sau đó dần dần kết hợp chúng thành các cấu trúc phức tạp. Ưu điểm của cách tiếp cận này là mỗi mô hình đều đơn giản, dễ hiểu và dễ bảo trì.

Mô hình cơ bản nhất là **thông tin vị trí**. Dù là điểm tham quan, khách sạn hay nhà hàng, tất cả đều cần thông tin vị trí. Chúng ta định nghĩa một lớp `Location` để biểu diễn tọa độ kinh độ và vĩ độ:

```python
class Location(BaseModel):
    """Location information (longitude and latitude coordinates)"""
    longitude: float = Field(..., description="Longitude", ge=-180, le=180)
    latitude: float = Field(..., description="Latitude", ge=-90, le=90)
```

Ở đây chúng ta dùng kiểm tra khoảng giá trị (`ge` nghĩa là lớn hơn hoặc bằng, `le` nghĩa là nhỏ hơn hoặc bằng) để đảm bảo giá trị kinh độ và vĩ độ nằm trong khoảng hợp lý.

Tiếp theo là **thông tin điểm tham quan**. Một điểm tham quan chứa thông tin về tên, địa chỉ, vị trí, thời lượng tham quan, mô tả, đánh giá, hình ảnh và giá vé. Lưu ý rằng chúng ta dùng `Location` làm kiểu của một trường, đây là một mô hình lồng nhau:

```python
class Attraction(BaseModel):
    """Attraction information"""
    name: str = Field(..., description="Attraction name")
    address: str = Field(..., description="Address")
    location: Location = Field(..., description="Longitude and latitude coordinates")
    visit_duration: int = Field(..., description="Recommended visit duration (minutes)", gt=0)
    description: str = Field(..., description="Attraction description")
    category: Optional[str] = Field(default="Attraction", description="Attraction category")
    rating: Optional[float] = Field(default=None, ge=0, le=5, description="Rating")
    image_url: Optional[str] = Field(default=None, description="Image URL")
    ticket_price: int = Field(default=0, ge=0, description="Ticket price (yuan)")
```

Tương tự, chúng ta định nghĩa **thông tin bữa ăn** và **thông tin khách sạn**. Các mô hình này có cấu trúc tương tự nhau, đều chứa thông tin cơ bản như tên, địa chỉ, vị trí và chi phí:

```python
class Meal(BaseModel):
    """Meal information"""
    type: str = Field(..., description="Meal type: breakfast/lunch/dinner/snack")
    name: str = Field(..., description="Meal name")
    address: Optional[str] = Field(default=None, description="Address")
    location: Optional[Location] = Field(default=None, description="Longitude and latitude coordinates")
    description: Optional[str] = Field(default=None, description="Description")
    estimated_cost: int = Field(default=0, description="Estimated cost (yuan)")

class Hotel(BaseModel):
    """Hotel information"""
    name: str = Field(..., description="Hotel name")
    address: str = Field(default="", description="Hotel address")
    location: Optional[Location] = Field(default=None, description="Hotel location")
    price_range: str = Field(default="", description="Price range")
    rating: str = Field(default="", description="Rating")
    distance: str = Field(default="", description="Distance to attractions")
    type: str = Field(default="", description="Hotel type")
    estimated_cost: int = Field(default=0, description="Estimated cost (yuan/night)")
```

**Thông tin ngân sách** là một mô hình đặc biệt, không chứa thông tin vị trí, mà chứa bản tổng hợp các khoản chi phí khác nhau:

```python
class Budget(BaseModel):
    """Budget information"""
    total_attractions: int = Field(default=0, description="Total attraction ticket cost")
    total_hotels: int = Field(default=0, description="Total hotel cost")
    total_meals: int = Field(default=0, description="Total meal cost")
    total_transportation: int = Field(default=0, description="Total transportation cost")
    total: int = Field(default=0, description="Total cost")
```

Bây giờ chúng ta có thể kết hợp các mô hình cơ bản này để xây dựng một **hành trình hàng ngày**. Một hành trình hàng ngày chứa ngày, mô tả, phương tiện di chuyển, sắp xếp chỗ ở, khách sạn, danh sách điểm tham quan và danh sách bữa ăn:

```python
class DayPlan(BaseModel):
    """Daily itinerary"""
    date: str = Field(..., description="Date")
    day_index: int = Field(..., description="Day number (starting from 0)")
    description: str = Field(..., description="Daily itinerary description")
    transportation: str = Field(..., description="Transportation method")
    accommodation: str = Field(..., description="Accommodation arrangement")
    hotel: Optional[Hotel] = Field(default=None, description="Hotel information")
    attractions: List[Attraction] = Field(default_factory=list, description="Attraction list")
    meals: List[Meal] = Field(default_factory=list, description="Meal arrangements")
```

Lưu ý rằng chúng ta dùng `List[Attraction]` để biểu diễn danh sách điểm tham quan, và `default_factory=list` nghĩa là giá trị mặc định là một danh sách rỗng.

**Thông tin thời tiết** cần được xử lý đặc biệt bởi vì định dạng nhiệt độ mà Amap trả về không theo chuẩn. Chúng ta dùng một bộ kiểm tra tùy chỉnh để xử lý điều này:

```python
class WeatherInfo(BaseModel):
    """Weather information"""
    date: str = Field(..., description="Date")
    day_weather: str = Field(..., description="Daytime weather")
    night_weather: str = Field(..., description="Nighttime weather")
    day_temp: int = Field(..., description="Daytime temperature (Celsius)")
    night_temp: int = Field(..., description="Nighttime temperature (Celsius)")
    wind_direction: str = Field(..., description="Wind direction")
    wind_power: str = Field(..., description="Wind power")

    @field_validator('day_temp', 'night_temp', mode='before')
    def parse_temperature(cls, v):
        """Parse temperature string: "16°C" -> 16"""
        if isinstance(v, str):
            v = v.replace('°C', '').replace('℃', '').replace('°', '').strip()
            try:
                return int(v)
            except ValueError:
                return 0  # Error tolerance
        return v
```

Cuối cùng, chúng ta định nghĩa **kế hoạch du lịch hoàn chỉnh**. Đây là mô hình cấp cao nhất chứa tất cả thông tin:

```python
class TripPlan(BaseModel):
    """Travel plan"""
    city: str = Field(..., description="Destination city")
    start_date: str = Field(..., description="Start date")
    end_date: str = Field(..., description="End date")
    days: List[DayPlan] = Field(default_factory=list, description="Daily itinerary")
    weather_info: List[WeatherInfo] = Field(default_factory=list, description="Weather information")
    overall_suggestions: str = Field(..., description="Overall suggestions")
    budget: Optional[Budget] = Field(default=None, description="Budget information")
```

Như vậy, chúng ta đã hoàn thành việc thiết kế toàn bộ mô hình dữ liệu. Từ `Location` cơ bản nhất, đến `Attraction`, `Meal`, `Hotel`, rồi đến `DayPlan`, và cuối cùng là `TripPlan`, tạo thành một cấu trúc phân cấp rõ ràng.

### 13.2.5 Ứng dụng Mô hình Dữ liệu trong Ứng dụng Web

Bây giờ hãy xem các mô hình dữ liệu này được sử dụng như thế nào trong ứng dụng Web thực tế. Trong FastAPI, các mô hình Pydantic có thể được dùng trực tiếp làm định nghĩa kiểu cho request và response. FastAPI sẽ tự động thực hiện kiểm tra dữ liệu, tuần tự hóa và tạo tài liệu.

```python
from fastapi import FastAPI
from app.models.schemas import TripPlanRequest, TripPlan

app = FastAPI()

@app.post("/api/trip/plan", response_model=TripPlan)
async def create_trip_plan(request: TripPlanRequest) -> TripPlan:
    """
    Create travel plan

    FastAPI automatically:
    1. Validates request data (TripPlanRequest)
    2. Validates response data (TripPlan)
    3. Generates OpenAPI documentation
    """
    trip_plan = await generate_trip_plan(request)
    return trip_plan
```

Khi người dùng gửi một POST request tới `/api/trip/plan`, FastAPI sẽ tự động chuyển đổi dữ liệu JSON thành một đối tượng `TripPlanRequest`. Nếu định dạng dữ liệu không đúng (chẳng hạn thiếu trường bắt buộc hoặc kiểu không khớp), FastAPI sẽ tự động trả về lỗi 400 và cho người dùng biết lỗi ở đâu.

Ở front-end, chúng ta cũng cần định nghĩa các kiểu TypeScript tương ứng. Mặc dù TypeScript và Python là các ngôn ngữ khác nhau, nhưng cấu trúc dữ liệu là như nhau:

```typescript
interface Location {
  longitude: number;
  latitude: number;
}

interface Attraction {
  name: string;
  address: string;
  location: Location;
  visit_duration: number;
  ticket_price: number;
}

interface TripPlan {
  city: string;
  start_date: string;
  end_date: string;
  days: DayPlan[];
}
```

Nhờ đó, front-end và back-end dùng chung một định dạng dữ liệu thống nhất. Khi back-end trả về một đối tượng `TripPlan`, front-end có thể dùng trực tiếp mà không cần chuyển đổi gì. Việc kiểm tra kiểu của TypeScript cũng có thể giúp chúng ta tránh được nhiều lỗi.

## 13.3 Thiết kế Cộng tác Đa Agent

### 13.3.1 Tại sao chúng ta cần Đa Agent

Ở Chương 7, chúng ta đã học cách xây dựng agent bằng SimpleAgent. Triết lý thiết kế của SimpleAgent đơn giản và trực tiếp: mỗi lần phương thức `run()` được gọi, Agent phân tích câu hỏi của người dùng, quyết định có gọi công cụ hay không, rồi trả về kết quả. Thiết kế này rất hiệu quả khi xử lý các tác vụ đơn giản, nhưng khi đối mặt với các tác vụ như lập kế hoạch du lịch, một số vấn đề sẽ nảy sinh.

Nếu chúng ta dùng một Agent duy nhất để hoàn thành việc lập kế hoạch du lịch, Agent này cần làm những gì? Đầu tiên, nó cần tìm kiếm thông tin điểm tham quan, điều này đòi hỏi gọi công cụ tìm kiếm POI của Amap. Sau đó, nó cần truy vấn thông tin thời tiết, điều này đòi hỏi gọi công cụ truy vấn thời tiết. Tiếp theo, nó cần tìm kiếm thông tin khách sạn, điều này lại đòi hỏi gọi công cụ tìm kiếm POI. Cuối cùng, nó cần tổng hợp tất cả thông tin này để tạo ra một kế hoạch du lịch hoàn chỉnh.

Nghe thì đơn giản, nhưng trong thao tác thực tế, vấn đề đầu tiên gặp phải là: **giới hạn của việc gọi công cụ**. SimpleAgent chỉ có thể thực thi một công cụ cho mỗi lần gọi `run()`. Điều này nghĩa là chúng ta cần gọi phương thức `run()` nhiều lần, mỗi lần xử lý một tác vụ. Nhưng điều này mang lại một vấn đề mới: làm thế nào để truyền thông tin giữa nhiều lần gọi? Làm thế nào để truyền thông tin điểm tham quan lấy được từ lần gọi đầu tiên sang lần gọi thứ hai? Chúng ta cần quản lý thủ công các kết quả trung gian này, và mã trở nên rất phức tạp.

Tất nhiên, chúng ta có thể dùng ReactAgent để giải quyết vấn đề này. ReactAgent có thể thực thi nhiều công cụ trong một lần gọi, và nó sẽ tự động thực hiện nhiều vòng suy nghĩ và hành động. Nhưng điều này mang lại vấn đề mới: **chi phí thời gian**. Mỗi vòng suy nghĩ của ReactAgent đều cần gọi LLM. Nếu cần gọi ba công cụ, cần ít nhất ba vòng suy nghĩ, nghĩa là ít nhất ba lần gọi LLM. Hơn nữa, các lần gọi này là tuần tự - lần tiếp theo chỉ có thể bắt đầu sau khi lần trước hoàn thành, nên tổng thời gian sẽ rất dài.

Vấn đề thứ hai là **độ phức tạp của prompt**. Nếu chúng ta muốn một Agent hoàn thành tất cả các tác vụ, chúng ta cần mô tả chi tiết logic thực thi của từng tác vụ trong prompt. Ví dụ:

```python
COMPLEX_PROMPT = """You are a travel planning assistant. You need to:
1. Use maps_text_search to search for attractions, keywords determined by user preferences
2. Use maps_weather to query weather, get weather forecast for the next few days
3. Use maps_text_search to search for hotels, type determined by user needs
4. Integrate all information to generate travel plan, including daily attractions, dining, accommodation arrangements
Note: Must execute in order, each tool can only be called once, output must be in JSON format...
"""
```

Loại prompt này có một số vấn đề. Đầu tiên là **khó bảo trì**. Nếu chúng ta muốn sửa logic tìm kiếm điểm tham quan (chẳng hạn thêm lọc theo đánh giá), chúng ta phải sửa toàn bộ prompt, điều này dễ ảnh hưởng đến các phần khác. Thứ hai là **dễ mắc lỗi**. LLM cần hiểu đồng thời yêu cầu của nhiều tác vụ, và rất dễ nhầm lẫn định dạng và tham số của các tác vụ khác nhau. Cuối cùng là **khó gỡ lỗi (debug)**. Khi kế hoạch được tạo ra không đúng như kỳ vọng, rất khó biết phần nào bị sai - liệu việc tìm kiếm điểm tham quan không chính xác, việc truy vấn thời tiết thất bại, hay logic tổng hợp có vấn đề?

Đối mặt với những vấn đề này, một ý tưởng tự nhiên là: liệu chúng ta có thể phân rã các tác vụ phức tạp thành nhiều tác vụ đơn giản và để các Agent khác nhau mỗi cái làm việc của mình không? Đây chính là ý tưởng cốt lõi của việc cộng tác đa Agent.

Hãy tưởng tượng một công ty du lịch trong thế giới thực. Khi bạn đến một công ty du lịch để tư vấn về kế hoạch du lịch, bạn sẽ không chỉ được một người phục vụ. Thường sẽ có một chuyên viên tư vấn điểm tham quan chuyên trách gợi ý các điểm tham quan; một chuyên viên tư vấn khách sạn chịu trách nhiệm đặt khách sạn; và một người lập kế hoạch hành trình chịu trách nhiệm tổng hợp tất cả thông tin thành một hành trình hoàn chỉnh. Mỗi người tập trung vào lĩnh vực chuyên môn của mình, và cuối cùng người lập kế hoạch hành trình tổng hợp tất cả thông tin. Sự phân công và cộng tác này hiệu quả hơn nhiều so với việc để một người làm tất cả mọi thứ.

### 13.3.2 Thiết kế Vai trò của Agent

Dựa trên nguyên tắc phân rã tác vụ, chúng ta đã thiết kế bốn Agent chuyên biệt, như minh họa ở Hình 13.6:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-6.png" alt="" width="85%"/>
  <p>Hình 13.6 Luồng Cộng tác Đa Agent</p>
</div>

- **AttractionSearchAgent (Chuyên gia Tìm kiếm Điểm tham quan)** tập trung vào việc tìm kiếm thông tin điểm tham quan. Nó chỉ cần hiểu sở thích của người dùng (chẳng hạn "lịch sử văn hóa", "cảnh quan thiên nhiên"), sau đó gọi công cụ tìm kiếm POI của Amap và trả về một danh sách các điểm tham quan liên quan. Prompt của nó rất đơn giản, chỉ cần giải thích cách chọn từ khóa dựa trên sở thích và cách gọi công cụ.

- **WeatherQueryAgent (Chuyên gia Truy vấn Thời tiết)** tập trung vào việc truy vấn thông tin thời tiết. Nó chỉ cần biết tên thành phố, sau đó gọi công cụ truy vấn thời tiết và trả về dự báo thời tiết cho vài ngày tới. Tác vụ của nó rất rõ ràng và hầu như không có lỗi.

- **HotelAgent (Chuyên gia Gợi ý Khách sạn)** tập trung vào việc tìm kiếm thông tin khách sạn. Nó cần hiểu nhu cầu chỗ ở của người dùng (chẳng hạn "tiết kiệm", "sang trọng"), sau đó gọi công cụ tìm kiếm POI và trả về một danh sách các khách sạn đáp ứng yêu cầu.

- **PlannerAgent (Chuyên gia Lập kế hoạch Hành trình)** chịu trách nhiệm tổng hợp tất cả thông tin. Nó nhận đầu ra từ ba Agent trước, cộng với yêu cầu ban đầu của người dùng (ngày tháng, ngân sách, v.v.), sau đó tạo ra một kế hoạch du lịch hoàn chỉnh. Nó không cần gọi bất kỳ công cụ bên ngoài nào, chỉ cần tập trung vào việc tổng hợp thông tin và sắp xếp hành trình.

Bây giờ hãy thiết kế chi tiết vai trò và prompt cho từng Agent. Khi thiết kế prompt, chúng ta cần xem xét một số câu hỏi then chốt: Agent này cần đầu vào gì? Nó nên tạo ra đầu ra gì? Nó cần gọi công cụ nào? Nó có thể gặp phải vấn đề gì?

Tác vụ của **AttractionSearchAgent** là tìm kiếm điểm tham quan dựa trên sở thích của người dùng. Đầu vào của nó là tên thành phố và sở thích của người dùng (chẳng hạn "lịch sử văn hóa", "cảnh quan thiên nhiên"). Nó cần gọi công cụ `amap_maps_text_search` với các tham số là từ khóa và thành phố. Đầu ra của nó là một danh sách điểm tham quan, bao gồm tên, địa chỉ, đánh giá và các thông tin khác.

```python
ATTRACTION_AGENT_PROMPT = """You are an attraction search expert.

**Tool Call Format:**
`[TOOL_CALL:amap_maps_text_search:keywords=attraction,city=city_name]`

**Examples:**
- `[TOOL_CALL:amap_maps_text_search:keywords=attraction,city=Beijing]`
- `[TOOL_CALL:amap_maps_text_search:keywords=museum,city=Shanghai]`

**Important:**
- Must use tools to search, don't fabricate information
- Search for attractions in {city} based on user preferences ({preferences})
"""
```

Prompt này ngắn gọn nhưng chứa tất cả thông tin cần thiết. Nó giải thích rõ ràng định dạng gọi công cụ, cung cấp các ví dụ cụ thể, và nhấn mạnh hai nguyên tắc quan trọng: phải dùng công cụ (không được bịa) và tìm kiếm dựa trên sở thích của người dùng.

Tác vụ của **WeatherQueryAgent** đơn giản hơn, chỉ cần truy vấn thời tiết. Đầu vào của nó là tên thành phố, và đầu ra là thông tin thời tiết.

```python
WEATHER_AGENT_PROMPT = """You are a weather query expert.

**Tool Call Format:**
`[TOOL_CALL:amap_maps_weather:city=city_name]`

Please query weather information for {city}.
"""
```

Tác vụ của **HotelAgent** là tìm kiếm khách sạn. Đầu vào của nó là tên thành phố và loại chỗ ở, và đầu ra là một danh sách khách sạn.

```python
HOTEL_AGENT_PROMPT = """You are a hotel recommendation expert.

**Tool Call Format:**
`[TOOL_CALL:amap_maps_text_search:keywords=hotel,city=city_name]`

Please search for {accommodation} hotels in {city}.
"""
```

**PlannerAgent** là phức tạp nhất bởi vì nó cần tổng hợp tất cả thông tin. Đầu vào của nó là yêu cầu của người dùng và đầu ra từ ba Agent trước, và đầu ra là một kế hoạch du lịch hoàn chỉnh (định dạng JSON).

```python
PLANNER_AGENT_PROMPT = """You are an itinerary planning expert.

**Output Format:**
Strictly return in the following JSON format:
{
  "city": "city name",
  "start_date": "YYYY-MM-DD",
  "end_date": "YYYY-MM-DD",
  "days": [...],
  "weather_info": [...],
  "overall_suggestions": "overall suggestions",
  "budget": {...}
}

**Planning Requirements:**
1. weather_info must include weather for each day
2. Temperature as pure numbers (without °C)
3. Arrange 2-3 attractions per day
4. Consider attraction distance and visit time
5. Include breakfast, lunch, and dinner
6. Provide practical suggestions
7. Include budget information
"""
```

### 13.3.3 Luồng Cộng tác của Agent

Bây giờ hãy xem bốn Agent này cộng tác như thế nào để hoàn thành tác vụ lập kế hoạch du lịch. Toàn bộ luồng có thể được chia thành năm bước:

```python
class TripPlannerAgent:
    def __init__(self):
        self.attraction_agent = SimpleAgent(name="Attraction Search", prompt=ATTRACTION_PROMPT)
        self.weather_agent = SimpleAgent(name="Weather Query", prompt=WEATHER_PROMPT)
        self.hotel_agent = SimpleAgent(name="Hotel Recommendation", prompt=HOTEL_PROMPT)
        self.planner_agent = SimpleAgent(name="Itinerary Planning", prompt=PLANNER_PROMPT)

    def plan_trip(self, request: TripPlanRequest) -> TripPlan:
        # Step 1: Attraction search
        attraction_response = self.attraction_agent.run(
            f"Please search for {request.preferences} attractions in {request.city}"
        )

        # Step 2: Weather query
        weather_response = self.weather_agent.run(
            f"Please query weather for {request.city}"
        )

        # Step 3: Hotel recommendation
        hotel_response = self.hotel_agent.run(
            f"Please search for {request.accommodation} hotels in {request.city}"
        )

        # Step 4: Integrate and generate plan
        planner_query = self._build_planner_query(
            request, attraction_response, weather_response, hotel_response
        )
        planner_response = self.planner_agent.run(planner_query)

        # Step 5: Parse JSON
        trip_plan = self._parse_trip_plan(planner_response)
        return trip_plan
```

Luồng này thực thi bốn bước một cách tuần tự, với đầu ra của mỗi bước làm đầu vào cho bước tiếp theo. Lưu ý rằng chúng ta dùng các mô hình Pydantic `TripPlanRequest` và `TripPlan` đã định nghĩa ở Phần 13.2.

### 13.3.4 Xây dựng Query

PlannerAgent cần tổng hợp tất cả thông tin. Query này cần bao gồm tất cả thông tin cần thiết và được tổ chức một cách rõ ràng, có trật tự để LLM có thể hiểu chính xác.

```python
def _build_planner_query(
    self,
    request: TripPlanRequest,
    attraction_response: str,
    weather_response: str,
    hotel_response: str
) -> str:
    """Build query for planning Agent"""
    return f"""
Please generate a {request.days}-day travel plan for {request.city} based on the following information:

**User Requirements:**
- Destination: {request.city}
- Dates: {request.start_date} to {request.end_date}
- Days: {request.days} days
- Preferences: {request.preferences}
- Budget: {request.budget}
- Transportation: {request.transportation}
- Accommodation: {request.accommodation}

**Attraction Information:**
{attraction_response}

**Weather Information:**
{weather_response}

**Hotel Information:**
{hotel_response}

Please generate a detailed travel plan, including daily attraction arrangements, dining recommendations, accommodation information, and budget details.
"""
```

Thông qua thiết kế cộng tác đa Agent này, chúng ta phân rã một tác vụ lập kế hoạch du lịch phức tạp thành bốn tác vụ con đơn giản. Mỗi Agent tập trung vào lĩnh vực chuyên môn của mình, đồng thời cũng đặt nền tảng tốt cho việc mở rộng tính năng trong tương lai (chẳng hạn thêm Agent gợi ý nhà hàng, Agent lập kế hoạch di chuyển).

## 13.4 Chi tiết Tích hợp Công cụ MCP

### 13.4.1 Tại sao không Gọi API Trực tiếp

Ở Phần 13.3, chúng ta đã thiết kế bốn Agent cộng tác trên tác vụ lập kế hoạch du lịch. Trong đó, AttractionSearchAgent, WeatherQueryAgent và HotelAgent đều cần gọi API của Amap để lấy dữ liệu. Một câu hỏi tự nhiên là: tại sao không gọi trực tiếp HTTP API của Amap trong Agent?

Hãy xem việc gọi API trực tiếp trông như thế nào. Amap cung cấp một API tìm kiếm POI, và chúng ta cần xây dựng các HTTP request, truyền tham số và phân tích phản hồi:

```python
import requests

def search_poi(keywords: str, city: str, api_key: str):
    """Directly call Amap POI search API"""
    url = "https://restapi.amap.com/v3/place/text"
    params = {
        "keywords": keywords,
        "city": city,
        "key": api_key,
        "output": "json"
    }
    response = requests.get(url, params=params)
    data = response.json()
    return data
```

Cách tiếp cận này trông có vẻ đơn giản, nhưng sẽ gặp một số vấn đề trong quá trình sử dụng thực tế. Đầu tiên là **Agent không thể gọi một cách tự chủ**. Trong framework HelloAgents của chúng ta, các Agent gọi công cụ bằng cách nhận diện các dấu hiệu gọi công cụ (tool call marker) trong prompt (chẳng hạn `[TOOL_CALL:tool_name:arg1=value1]`). Nếu chúng ta gọi API trực tiếp trong mã, Agent sẽ mất khả năng ra quyết định tự chủ và trở thành một lệnh gọi hàm đơn giản.

Thứ hai là **truyền tham số phức tạp**. API của Amap có rất nhiều tham số. Ví dụ, tìm kiếm POI có hơn chục tham số như `keywords`, `city`, `types`, `offset`, `page`, v.v. Nếu chúng ta muốn Agent dùng các tham số này một cách linh hoạt, chúng ta cần giải thích chi tiết ý nghĩa và định dạng của từng tham số trong prompt, điều này sẽ làm cho prompt trở nên rất phức tạp.

Thứ ba là **phân tích phản hồi khó khăn**. Dữ liệu mà Amap API trả về ở định dạng JSON với cấu trúc tương đối phức tạp. Chúng ta cần viết mã để phân tích dữ liệu này và trích xuất các trường mình cần. Nếu định dạng phản hồi của API thay đổi, chúng ta cần sửa mã phân tích.

Cuối cùng là **quản lý công cụ hỗn loạn**. Amap cung cấp hơn chục API khác nhau (tìm kiếm POI, truy vấn thời tiết, lập kế hoạch tuyến đường, v.v.). Nếu chúng ta viết một hàm cho mỗi API rồi đăng ký thủ công nó vào danh sách công cụ của Agent, mã sẽ trở nên rất dài dòng. Và khi chúng ta muốn thêm một API mới, chúng ta cần sửa nhiều chỗ.

### 13.4.2 Tích hợp Amap MCP

MCP (Model Context Protocol) là một giao thức chuẩn hóa do Anthropic đề xuất để kết nối các LLM và công cụ bên ngoài. Phần này sẽ giới thiệu cách tích hợp máy chủ Amap MCP trong dự án. Dự án của chúng ta dùng `amap-mcp-server`, đây là một máy chủ MCP được hiện thực bằng Node.js:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-7.png" alt="" width="85%"/>
  <p>Hình 13.7 Các Công cụ của amap-mcp-server</p>
</div>

Máy chủ Amap MCP cung cấp nhiều công cụ khác nhau, chủ yếu được chia thành các nhóm sau, như minh họa ở Bảng 13.1:

<div align="center">
  <p>Bảng 13.1 Các Nhóm Công cụ Amap MCP</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-table-1.png" alt="" width="85%"/>
</div>

Thông qua giao thức MCP, chúng ta có thể dễ dàng tích hợp trong HelloAgents:

```python
from hello_agents.tools import MCPTool
from app.config import get_settings

settings = get_settings()

# Create MCP tool
mcp_tool = MCPTool(
    name="amap_mcp",
    command="npx",
    args=["-y", "@sugarforever/amap-mcp-server"],
    env={"AMAP_API_KEY": settings.amap_api_key},
    auto_expand=True
)
```

Đoạn mã này làm gì? Đầu tiên, `command` và `args` chỉ định cách khởi động máy chủ MCP. `npx -y @sugarforever/amap-mcp-server` sẽ tải và chạy gói `amap-mcp-server` từ kho lưu trữ npm. Tham số `env` truyền các biến môi trường, ở đây chúng ta truyền Amap API key.

**Lưu ý:** Một số ví dụ trong tài liệu này dùng `npx` để khởi chạy các dịch vụ MCP (Model Context Protocol). Tuy nhiên, trong kho mã tương ứng với phần nội dung này, chúng ta thực sự dùng `uvx`. Điều quan trọng cần lưu ý là `npx` và `uvx` chia sẻ các nguyên lý thiết kế gần như giống hệt nhau - điểm khác biệt duy nhất nằm ở hệ sinh thái của chúng: `npx` nhắm đến JavaScript/Node.js (các gói từ npm), trong khi `uvx` nhắm đến Python (các gói từ PyPI). Không có sự hơn kém nào giữa hai phương pháp. Vui lòng chọn theo nhu cầu của bạn khi sử dụng.

Khi chúng ta tạo đối tượng `MCPTool`, nó sẽ khởi động tiến trình máy chủ MCP ở nền và giao tiếp với máy chủ thông qua đầu vào/đầu ra chuẩn (stdin/stdout). Đây là một đặc điểm của giao thức MCP: dùng giao tiếp liên tiến trình (inter-process communication) thay vì HTTP, hiệu quả hơn và dễ quản lý hơn.

Tham số then chốt nhất là `auto_expand=True`. Khi được đặt là True, `MCPTool` sẽ tự động truy vấn xem máy chủ MCP cung cấp những công cụ nào, rồi tạo một đối tượng Tool độc lập cho mỗi công cụ. Đây là lý do vì sao chúng ta chỉ tạo một `MCPTool`, nhưng Agent lại có được 16 công cụ. Hãy xem quá trình này:

```python
# Create one MCPTool
mcp_tool = MCPTool(..., auto_expand=True)
agent.add_tool(mcp_tool)

# Agent actually gets 16 tools!
print(list(agent.tools.keys()))
# ['amap_maps_text_search', 'amap_maps_weather', ...]
```

Như minh họa ở Hình 13.8, giả sử người dùng muốn tìm kiếm điểm tham quan ở Bắc Kinh. AttractionSearchAgent nhận query "Vui lòng tìm kiếm các điểm tham quan lịch sử văn hóa ở Bắc Kinh". Agent phân tích query này và quyết định gọi công cụ `amap_maps_text_search` với các tham số `keywords=attraction, city=Beijing`.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-8.png" alt="" width="85%"/>
  <p>Hình 13.8 Luồng Gọi Công cụ MCP</p>
</div>

Agent tạo ra một dấu hiệu gọi công cụ: `[TOOL_CALL:amap_maps_text_search:keywords=attraction,city=Beijing]`. Framework HelloAgents phân tích dấu hiệu này, trích xuất tên công cụ và tham số, sau đó gọi đối tượng Tool tương ứng.

Đối tượng Tool được `MCPTool` tự động tạo ra, và nó sẽ gửi yêu cầu gọi tới máy chủ MCP. Cụ thể, nó sẽ xây dựng một thông điệp định dạng JSON-RPC và gửi tới tiến trình máy chủ thông qua stdin:

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "amap_maps_text_search",
    "arguments": {
      "keywords": "attraction",
      "city": "Beijing"
    }
  }
}
```

Máy chủ MCP nhận thông điệp này, phân tích các tham số, rồi gọi HTTP API của Amap. Nó sẽ xây dựng một HTTP request, thêm API key, gửi request, và nhận phản hồi.

Amap API trả về dữ liệu định dạng JSON chứa danh sách điểm tham quan, địa chỉ, tọa độ và các thông tin khác. Máy chủ MCP phân tích dữ liệu này, trích xuất các trường then chốt, sau đó xây dựng một thông điệp phản hồi, trả về `MCPTool` thông qua stdout:

```json
{
  "jsonrpc": "2.0",
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Found the following attractions:\n1. Forbidden City Museum - Address: No. 4 Jingshan Front Street, Dongcheng District\n2. Temple of Heaven Park - Address: Tiantan Road, Dongcheng District\n..."
      }
    ]
  }
}
```

`MCPTool` nhận phản hồi, trích xuất nội dung văn bản, và trả về Agent. Agent dùng kết quả này làm đầu ra của lệnh gọi công cụ và tiếp tục tạo ra câu trả lời cuối cùng.

Quá trình này trông có vẻ phức tạp, nhưng đối với Agent, nó chỉ cần biết rằng có một công cụ tên là `amap_maps_text_search` có thể tìm kiếm điểm tham quan. Tất cả các chi tiết ở tầng dưới đều được giao thức MCP và `MCPTool` đóng gói.

### 13.4.3 Chia sẻ Thực thể (Instance) MCP

Trong hệ thống đa Agent của chúng ta, ba Agent đều cần dùng các công cụ Amap. Vậy mỗi Agent nên tạo thực thể `MCPTool` riêng của mình, hay chia sẻ cùng một thực thể?

Nếu mỗi Agent tạo một thực thể `MCPTool`, điều này nghĩa là sẽ có ba tiến trình máy chủ chạy đồng thời. Mỗi tiến trình sẽ gọi Amap API một cách độc lập, điều này có thể vượt quá giới hạn tần suất (rate limit) của API. Hơn nữa, nhiều tiến trình sẽ chiếm dụng nhiều bộ nhớ và tài nguyên CPU hơn.

Một cách tiếp cận tốt hơn là để tất cả các Agent chia sẻ cùng một thực thể `MCPTool`. Bằng cách này, chỉ cần khởi động một tiến trình máy chủ MCP, và tất cả các lệnh gọi API đều đi qua tiến trình này. Điều này không chỉ tiết kiệm tài nguyên mà còn cho phép kiểm soát tốt hơn tần suất gọi API.

Trong mã, chúng ta tạo một thực thể `MCPTool` trong hàm khởi tạo (constructor) của `TripPlannerAgent`, sau đó thêm nó vào danh sách công cụ của mỗi Agent con:

```python
class TripPlannerAgent:
    def __init__(self):
        settings = get_settings()
        self.llm = HelloAgentsLLM()

        # Create shared MCP tool instance (create only once)
        self.mcp_tool = MCPTool(
            name="amap_mcp",
            command="npx",
            args=["-y", "@sugarforever/amap-mcp-server"],
            env={"AMAP_API_KEY": settings.amap_api_key},
            auto_expand=True
        )

        # Create multiple Agents, sharing the same MCP tool
        self.attraction_agent = SimpleAgent(
            name="AttractionSearchAgent",
            llm=self.llm,
            system_prompt=ATTRACTION_AGENT_PROMPT
        )
        self.attraction_agent.add_tool(self.mcp_tool)  # Share

        self.weather_agent = SimpleAgent(
            name="WeatherQueryAgent",
            llm=self.llm,
            system_prompt=WEATHER_AGENT_PROMPT
        )
        self.weather_agent.add_tool(self.mcp_tool)  # Share

        self.hotel_agent = SimpleAgent(
            name="HotelAgent",
            llm=self.llm,
            system_prompt=HOTEL_AGENT_PROMPT
        )
        self.hotel_agent.add_tool(self.mcp_tool)  # Share
```

Bằng cách này, cả ba Agent đều có thể dùng 16 công cụ của Amap, nhưng chỉ có một tiến trình máy chủ MCP đang chạy ở bên dưới. Khi chúng ta gọi phương thức `plan_trip` của `TripPlannerAgent`, ba Agent sẽ lần lượt gọi công cụ, và tất cả các request đều được gửi tới Amap API thông qua cùng một máy chủ MCP.

### 13.4.4 Tích hợp Unsplash Image API

Ngoài Amap, chúng ta cũng cần lấy hình ảnh cho các điểm tham quan để làm cho kế hoạch du lịch trở nên sống động và trực quan hơn. Chúng ta dùng Unsplash API để tìm kiếm hình ảnh điểm tham quan. Lưu ý rằng Unsplash là một dịch vụ nước ngoài và là một trong số ít các image API có thể dùng miễn phí, nên kết quả tìm kiếm có thể không đủ chính xác. Trong các dự án thực tế, bạn có thể cân nhắc dùng image API POI của Bing, Baidu hoặc Amap, nhưng các dịch vụ này thường yêu cầu trả phí.

Việc tích hợp Unsplash API tương đối đơn giản. Chúng ta tạo một lớp `UnsplashService` để đóng gói các lệnh gọi API:

```python
# backend/app/services/unsplash_service.py
import requests
from typing import Optional, List, Dict
import logging

logger = logging.getLogger(__name__)

class UnsplashService:
    """Unsplash image service"""

    def __init__(self, access_key: str):
        self.access_key = access_key
        self.base_url = "https://api.unsplash.com"

    def search_photos(self, query: str, per_page: int = 10) -> List[Dict]:
        """Search for images"""
        try:
            url = f"{self.base_url}/search/photos"
            params = {
                "query": query,
                "per_page": per_page,
                "client_id": self.access_key
            }

            response = requests.get(url, params=params, timeout=10)
            response.raise_for_status()

            data = response.json()
            results = data.get("results", [])

            # Extract image URLs
            photos = []
            for result in results:
                photos.append({
                    "url": result["urls"]["regular"],
                    "description": result.get("description", ""),
                    "photographer": result["user"]["name"]
                })

            return photos

        except Exception as e:
            logger.error(f"Image search failed: {e}")
            return []

    def get_photo_url(self, query: str) -> Optional[str]:
        """Get single image URL"""
        photos = self.search_photos(query, per_page=1)
        return photos[0].get("url") if photos else None
```

Lớp dịch vụ này cung cấp hai phương thức: `search_photos` tìm kiếm nhiều hình ảnh, và `get_photo_url` lấy URL của một hình ảnh đơn lẻ. Chúng ta dùng dịch vụ này trong route API để lấy hình ảnh cho mỗi điểm tham quan:

```python
# backend/app/api/routes/trip.py
from app.services.unsplash_service import UnsplashService

unsplash_service = UnsplashService(settings.unsplash_access_key)

@router.post("/plan", response_model=TripPlan)
async def create_trip_plan(request: TripPlanRequest) -> TripPlan:
    # Generate travel plan
    trip_plan = trip_planner_agent.plan_trip(request)

    # Get images for each attraction
    for day in trip_plan.days:
        for attraction in day.attractions:
            if not attraction.image_url:
                image_url = unsplash_service.get_photo_url(
                    f"{attraction.name} {trip_plan.city}"
                )
                attraction.image_url = image_url

    return trip_plan
```

Lưu ý rằng chúng ta không đóng gói Unsplash thành một Tool hay công cụ MCP, mà gọi nó trực tiếp trong route API. Đây là vì việc tìm kiếm hình ảnh không đòi hỏi khả năng ra quyết định thông minh của Agent, nó chỉ là một bước làm giàu dữ liệu (data enhancement) đơn giản. Nếu bạn muốn Agent tự chủ quyết định có cần hình ảnh hay không hoặc chọn các nguồn hình ảnh khác nhau, bạn có thể cân nhắc đóng gói nó thành một Tool.

## 13.5 Chi tiết Phát triển Front-End

### 13.5.1 Kiến trúc Web Tách biệt Front-end và Back-end

Trước khi bắt đầu phát triển front-end, chúng ta cần hiểu mô hình kiến trúc của các ứng dụng Web hiện đại. Trong thời kỳ phát triển Web ban đầu, front-end và back-end được trộn lẫn với nhau. Ví dụ, các công nghệ như PHP và JSP có template HTML và mã logic nghiệp vụ được viết trong cùng một file. Cách tiếp cận này tiện lợi trong các dự án nhỏ, nhưng gặp nhiều vấn đề trong các dự án lớn: các lập trình viên front-end và back-end cần phối hợp thường xuyên, mã khó tái sử dụng, và việc kiểm thử khó khăn.

Các ứng dụng Web hiện đại nói chung áp dụng kiến trúc **tách biệt front-end và back-end**. Back-end chỉ chịu trách nhiệm cung cấp các API interface và trả về dữ liệu ở định dạng JSON. Front-end là một ứng dụng độc lập, gọi các API back-end thông qua các HTTP request, lấy dữ liệu, rồi kết xuất các trang. Kiến trúc này có một số ưu điểm rõ ràng: front-end và back-end có thể được phát triển, triển khai và kiểm thử một cách độc lập; front-end có thể là một ứng dụng Web, ứng dụng di động hay ứng dụng desktop, tất cả đều dùng chung một bộ API back-end; front-end có thể dùng các framework và bộ công cụ (toolchain) hiện đại để mang lại trải nghiệm người dùng tốt hơn.

Trong dự án trợ lý du lịch thông minh của chúng ta, back-end được hiện thực bằng Python và FastAPI, cung cấp một API interface cốt lõi `POST /api/trip/plan` để nhận các yêu cầu du lịch và trả về các kế hoạch du lịch. Front-end được hiện thực bằng Vue 3 và TypeScript, và là một ứng dụng đơn trang (SPA - single-page application). Người dùng điền biểu mẫu trong trình duyệt, nhấn nút "Start Planning", front-end gửi một HTTP request tới back-end, chờ phản hồi, rồi kết xuất trang kết quả. Trong suốt quá trình này, trang không bị làm mới (refresh), và trải nghiệm người dùng rất mượt mà.

Việc lựa chọn technology stack (bộ công nghệ) của front-end cần xem xét một số yếu tố: hiệu suất phát triển, hiệu năng, hệ sinh thái và đường cong học tập (learning curve). Như minh họa ở Bảng 13.2, dự án đã chọn technology stack sau:

<div align="center">
  <p>Bảng 13.2 Technology Stack của Front-End</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/13-figures/13-table-2.png" alt="" width="85%"/>
</div>

Cấu trúc thư mục của dự án như sau:

```
frontend/
├── src/
│   ├── views/              # Page components
│   │   ├── Home.vue        # Home page (form)
│   │   └── Result.vue      # Result page
│   ├── services/           # API services
│   │   └── api.ts
│   ├── types/              # Type definitions
│   │   └── index.ts
│   ├── router/             # Router configuration
│   │   └── index.ts
│   ├── App.vue
│   └── main.ts
├── package.json
├── vite.config.ts
└── tsconfig.json
```

Thư mục `views` lưu trữ các page component, thư mục `services` lưu trữ logic gọi API, thư mục `types` lưu trữ các định nghĩa kiểu TypeScript, và thư mục `router` lưu trữ cấu hình định tuyến (router).

### 13.5.2 Định nghĩa Kiểu (Type Definitions)

Ở Phần 13.2, chúng ta đã dùng Pydantic để định nghĩa các mô hình dữ liệu ở back-end, chẳng hạn `Location`, `Attraction`, `DayPlan`, `TripPlan`, v.v. Ở front-end, chúng ta cần định nghĩa các kiểu TypeScript tương ứng.

Hãy xem cách định nghĩa các kiểu này. Đầu tiên là kiểu `Location` cơ bản nhất, biểu diễn tọa độ kinh độ và vĩ độ:

```typescript
// frontend/src/types/index.ts
export interface Location {
  longitude: number
  latitude: number
}
```

Định nghĩa kiểu này tương ứng chính xác với mô hình Pydantic ở back-end. Lưu ý rằng TypeScript dùng từ khóa `interface` để định nghĩa kiểu, kiểu của trường được phân tách bằng dấu hai chấm, và không cần giá trị mặc định.

Tiếp theo là kiểu `Attraction`, biểu diễn thông tin điểm tham quan:

```typescript
export interface Attraction {
  name: string
  address: string
  location: Location
  visit_duration: number
  description: string
  category?: string
  rating?: number
  image_url?: string
  ticket_price?: number
}
```

Lưu ý rằng ở đây chúng ta dùng kiểu `Location` làm kiểu của một trường, đây là một kiểu lồng nhau. Dấu chấm hỏi `?` biểu thị một trường tùy chọn, tương ứng với `Optional` trong mô hình Pydantic ở back-end.

Tương tự, chúng ta định nghĩa các kiểu như `Meal`, `Hotel`, `Budget`, `WeatherInfo`, v.v. Cuối cùng là kiểu cấp cao nhất `TripPlan`:

```typescript
export interface TripPlan {
  city: string
  start_date: string
  end_date: string
  days: DayPlan[]
  weather_info: WeatherInfo[]
  overall_suggestions: string
  budget?: Budget
}
```

Còn có kiểu request `TripPlanRequest`, tương ứng với mô hình request ở back-end:

```typescript
export interface TripPlanRequest {
  city: string
  start_date: string
  end_date: string
  days: number
  preferences: string
  budget: string
  transportation: string
  accommodation: string
}
```

Các định nghĩa kiểu này để làm gì? Đầu tiên, khi chúng ta gọi API, TypeScript sẽ kiểm tra xem dữ liệu chúng ta truyền vào có tuân theo kiểu `TripPlanRequest` hay không. Nếu chúng ta vô tình viết `days` thành một chuỗi, TypeScript sẽ lập tức báo lỗi. Thứ hai, khi chúng ta nhận phản hồi API, TypeScript sẽ kiểm tra xem dữ liệu phản hồi có tuân theo kiểu `TripPlan` hay không. Nếu cấu trúc dữ liệu của back-end thay đổi, front-end sẽ lập tức phát hiện ra. Cuối cùng, IDE có thể cung cấp gợi ý hoàn thành mã dựa trên các định nghĩa kiểu. Khi chúng ta gõ `tripPlan.`, IDE sẽ tự động liệt kê tất cả các trường khả dụng.

### 13.5.3 Đóng gói Dịch vụ API

Với các định nghĩa kiểu, chúng ta có thể đóng gói các lệnh gọi API. Chúng ta tạo một file `api.ts` và dùng Axios để gửi các HTTP request:

```typescript
import axios from 'axios'
import type { TripPlanRequest, TripPlan } from '../types'

const api = axios.create({
  baseURL: 'http://localhost:8000/api',
  timeout: 120000, // 2-minute timeout
  headers: {
    'Content-Type': 'application/json'
  }
})
```

Ở đây chúng ta tạo một thực thể Axios và cấu hình base URL, timeout và các request header. Tại sao timeout được đặt là 2 phút? Bởi vì việc tạo một kế hoạch du lịch đòi hỏi gọi nhiều Agent, mỗi Agent cần gọi LLM và các API bên ngoài, và toàn bộ quá trình có thể mất 10-30 giây. Nếu timeout quá ngắn, request sẽ bị gián đoạn.

Tiếp theo chúng ta thêm các interceptor. Các interceptor có thể thực thi một số logic chung trước khi gửi request và sau khi nhận response, chẳng hạn ghi log, xử lý lỗi, xác thực, v.v.:

```typescript
// Request interceptor
api.interceptors.request.use(
  config => {
    console.log('Sending request:', config)
    return config
  },
  error => Promise.reject(error)
)

// Response interceptor
api.interceptors.response.use(
  response => {
    console.log('Received response:', response)
    return response
  },
  error => {
    console.error('Request failed:', error)
    return Promise.reject(error)
  }
)
```

Cuối cùng, chúng ta định nghĩa hàm API, đây là điểm vào (entry point) duy nhất để front-end gọi back-end:

```typescript
// Generate travel plan
export const generateTripPlan = async (request: TripPlanRequest): Promise<TripPlan> => {
  const response = await api.post<TripPlan>('/trip/plan', request)
  return response.data
}
```

Lưu ý chữ ký kiểu (type signature) của hàm này: tham số có kiểu `TripPlanRequest`, và giá trị trả về có kiểu `Promise<TripPlan>`. Điều này nghĩa là TypeScript sẽ kiểm tra xem các tham số mà bên gọi truyền vào có đáp ứng yêu cầu hay không, và cũng sẽ kiểm tra xem việc sử dụng giá trị trả về có đúng hay không.

### 13.5.4 Thiết kế Biểu mẫu Trang chủ (Home Form)

Trang Home là điểm vào của người dùng, chứa một biểu mẫu để người dùng điền các yêu cầu du lịch. Chúng ta dùng Composition API của Vue 3 để tổ chức mã:

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { message } from 'ant-design-vue'
import { generateTripPlan } from '@/services/api'
import type { TripPlanRequest } from '@/types'

const router = useRouter()
const loading = ref(false)
const loadingProgress = ref(0)
const loadingStatus = ref('')

const formData = ref<TripPlanRequest>({
  city: '',
  start_date: '',
  end_date: '',
  days: 3,
  preferences: 'History and Culture',
  budget: 'Medium',
  transportation: 'Public Transportation',
  accommodation: 'Budget Hotel'
})
</script>
```

Ở đây chúng ta dùng `ref` để tạo các biến phản ứng (reactive variable). `formData` là dữ liệu biểu mẫu, có kiểu `TripPlanRequest`. `loading` cho biết có đang tải hay không, `loadingProgress` cho biết tiến trình tải, và `loadingStatus` cho biết văn bản trạng thái tải.

Logic gửi biểu mẫu như sau:

```typescript
const handleSubmit = async () => {
  loading.value = true
  loadingProgress.value = 0

  // Simulate progress updates
  const progressInterval = setInterval(() => {
    if (loadingProgress.value < 90) {
      loadingProgress.value += 10
      if (loadingProgress.value <= 30) loadingStatus.value = '🔍 Searching for attractions...'
      else if (loadingProgress.value <= 50) loadingStatus.value = '🌤️ Querying weather...'
      else if (loadingProgress.value <= 70) loadingStatus.value = '🏨 Recommending hotels...'
      else loadingStatus.value = '📋 Generating itinerary...'
    }
  }, 500)

  try {
    const response = await generateTripPlan(formData.value)
    clearInterval(progressInterval)
    loadingProgress.value = 100
    router.push({ name: 'result', state: { tripPlan: response } })
  } catch (error) {
    clearInterval(progressInterval)
    message.error('Failed to generate plan, please try again')
  } finally {
    loading.value = false
  }
}
```

Đoạn mã này làm một số việc. Đầu tiên, nó đặt `loading` thành true để hiển thị trạng thái đang tải. Sau đó, nó khởi động một bộ hẹn giờ (timer) cập nhật thanh tiến trình và văn bản trạng thái mỗi 500 mili-giây. Đây là một tiến trình mô phỏng bởi vì chúng ta không thể biết chính xác tiến độ xử lý của back-end. Nhưng điều này giúp người dùng biết rằng hệ thống đang làm việc, thay vì bị treo.

Tiếp theo, nó gọi hàm `generateTripPlan` để gửi API request. Đây là một thao tác bất đồng bộ (asynchronous), và chúng ta dùng `await` để chờ phản hồi. Nếu request thành công, xóa bộ hẹn giờ, đặt tiến trình thành 100%, rồi điều hướng tới trang kết quả và truyền dữ liệu kế hoạch du lịch. Nếu request thất bại, hiển thị một thông báo lỗi. Cuối cùng, dù thành công hay thất bại, đặt `loading` thành false để ẩn trạng thái đang tải.

Phần template dùng các component của Ant Design Vue:

```vue
<template>
  <div class="home-container">
    <div class="page-header">
      <h1 class="page-title">✈️ Intelligent Travel Assistant</h1>
      <p class="page-subtitle">AI-Powered Personalized Travel Planning</p>
    </div>

    <a-card class="form-card">
      <a-form :model="formData" @finish="handleSubmit">
        <a-form-item label="Destination City" name="city" :rules="[{ required: true }]">
          <a-input v-model:value="formData.city" placeholder="e.g., Beijing" />
        </a-form-item>

        <!-- More form items... -->

        <a-form-item>
          <a-button type="primary" html-type="submit" size="large" :loading="loading">
            Start Planning
          </a-button>
        </a-form-item>

        <!-- Loading progress bar -->
        <a-form-item v-if="loading">
          <a-progress :percent="loadingProgress" status="active" />
          <p>{{ loadingStatus }}</p>
        </a-form-item>
      </a-form>
    </a-card>
  </div>
</template>
```

Lưu ý directive `v-model:value`, nó thực hiện ràng buộc dữ liệu hai chiều (two-way data binding). Khi người dùng gõ vào ô nhập, `formData.city` tự động cập nhật. Khi giá trị của `formData.city` thay đổi, nội dung ô nhập cũng tự động cập nhật.

### 13.5.5 Hiển thị Trang Kết quả (Result Page)

Trang Result là cốt lõi của toàn bộ ứng dụng, hiển thị kế hoạch du lịch được tạo ra. Trang này bao gồm một số phần: tổng quan hành trình, chi tiết ngân sách, trực quan hóa bản đồ, chi tiết hành trình hàng ngày và thông tin thời tiết.

Đầu tiên là trực quan hóa bản đồ. Chúng ta dùng Amap JS API để đánh dấu vị trí các điểm tham quan trên bản đồ:

```typescript
import AMapLoader from '@amap/amap-jsapi-loader'

const initMap = async () => {
  const AMap = await AMapLoader.load({
    key: 'your_amap_web_key',
    version: '2.0'
  })

  map = new AMap.Map('amap-container', {
    zoom: 12,
    center: [116.397128, 39.916527]
  })

  // Add attraction markers
  tripPlan.value.days.forEach((day) => {
    day.attractions.forEach((attraction, index) => {
      const marker = new AMap.Marker({
        position: [attraction.location.longitude, attraction.location.latitude],
        title: attraction.name,
        label: { content: `${index + 1}`, direction: 'top' }
      })
      map.add(marker)
    })
  })
}
```

Đoạn mã này đầu tiên tải Amap SDK, rồi tạo một thực thể bản đồ, và cuối cùng lặp qua tất cả các điểm tham quan để tạo một marker cho mỗi điểm. Vị trí của marker là tọa độ kinh độ và vĩ độ của điểm tham quan, được lấy từ đối tượng `Attraction` ở back-end.

Chức năng xuất file dùng các thư viện `html2canvas` và `jsPDF`. `html2canvas` có thể chuyển đổi các phần tử DOM thành Canvas, sau đó chúng ta có thể xuất Canvas thành hình ảnh hoặc PDF:

```typescript
import html2canvas from 'html2canvas'
import jsPDF from 'jspdf'

// Export as image
const exportAsImage = async () => {
  const element = document.getElementById('trip-plan-content')
  const canvas = await html2canvas(element, { scale: 2 })
  const link = document.createElement('a')
  link.download = `${tripPlan.value.city} Travel Plan.png`
  link.href = canvas.toDataURL()
  link.click()
}

// Export as PDF
const exportAsPDF = async () => {
  const element = document.getElementById('trip-plan-content')
  const canvas = await html2canvas(element, { scale: 2 })
  const imgData = canvas.toDataURL('image/png')
  const pdf = new jsPDF('p', 'mm', 'a4')
  const imgWidth = 210
  const imgHeight = (canvas.height * imgWidth) / canvas.width
  pdf.addImage(imgData, 'PNG', 0, 0, imgWidth, imgHeight)
  pdf.save(`${tripPlan.value.city} Travel Plan.pdf`)
}
```

Thông qua các công nghệ front-end này, chúng ta đã hiện thực một ứng dụng Web hoàn chỉnh. Người dùng có thể điền biểu mẫu trong trình duyệt, gửi request, chờ AI tạo ra kế hoạch du lịch, sau đó xem các sắp xếp hành trình chi tiết, thấy vị trí các điểm tham quan trên bản đồ, và xuất thành hình ảnh hoặc PDF. Toàn bộ quá trình mượt mà và tự nhiên - đây chính là sức hấp dẫn của các ứng dụng Web hiện đại.

## 13.6 Chi tiết Hiện thực Tính năng

Phần này giới thiệu các hiện thực tính năng cốt lõi của trợ lý du lịch thông minh, bao gồm tính toán ngân sách, thanh tiến trình tải, chỉnh sửa hành trình, chức năng xuất file và điều hướng bên (side navigation).

### 13.6.1 Tính năng Tính toán Ngân sách

Khi lập kế hoạch cho một chuyến đi, ngân sách là một yếu tố cần cân nhắc rất quan trọng. Người dùng cần biết chuyến đi này sẽ tốn khoảng bao nhiêu tiền và tiền sẽ được chi vào đâu. Trợ lý du lịch thông minh của chúng ta cung cấp chức năng tính toán ngân sách tự động, chia chi phí thành bốn nhóm lớn: vé điểm tham quan, chỗ ở khách sạn, ăn uống và di chuyển.

Logic tính toán ngân sách được hiện thực ở đâu? Chúng ta chọn hiện thực nó trong PlannerAgent ở back-end. Tại sao không tính toán ở front-end? Bởi vì việc ước tính ngân sách cần dựa trên giá vé điểm tham quan, khoảng giá khách sạn, tiêu chuẩn ăn uống và các thông tin khác, tất cả những thứ này đều đã được PlannerAgent lấy được khi tạo hành trình. Nếu tính toán ở front-end, chúng ta sẽ phải lặp lại logic này, và nó có thể không chính xác.

Trong prompt của PlannerAgent, chúng ta yêu cầu rõ ràng LLM tạo ra thông tin ngân sách:

```python
PLANNER_AGENT_PROMPT = """
You are an itinerary planning expert.

**Output Format:**
Strictly return in the following JSON format:
{
  ...
  "budget": {
    "total_attractions": 180,
    "total_hotels": 1200,
    "total_meals": 480,
    "total_transportation": 200,
    "total": 2060
  }
}

**Planning Requirements:**
...
7. Include budget information, estimate based on attraction tickets, hotel prices, dining standards, and transportation methods
"""
```

LLM sẽ ước tính chi phí của từng khoản dựa trên các điểm tham quan, khách sạn và sắp xếp ăn uống trong hành trình. Ví dụ, nếu hành trình bao gồm Tử Cấm Thành (vé 60 tệ), Thiên Đàn (vé 15 tệ) và Di Hòa Viên (vé 30 tệ), thì tổng chi phí vé điểm tham quan là 105 tệ. Nếu là chuyến đi 3 ngày 2 đêm với khách sạn tiết kiệm (300 tệ mỗi đêm), thì tổng chi phí khách sạn là 600 tệ.

Ở front-end, chúng ta dùng component Statistic của Ant Design Vue để hiển thị thông tin ngân sách. Component này được thiết kế chuyên biệt để hiển thị dữ liệu thống kê và hỗ trợ hiệu ứng động cho số, tiền tố/hậu tố, kiểu tùy chỉnh, v.v.:

```vue
<a-card v-if="tripPlan.budget" title="💰 Budget Details">
  <a-row :gutter="16">
    <a-col :span="6">
      <a-statistic title="Attraction Tickets" :value="tripPlan.budget.total_attractions" suffix="yuan" />
    </a-col>
    <a-col :span="6">
      <a-statistic title="Hotel Accommodation" :value="tripPlan.budget.total_hotels" suffix="yuan" />
    </a-col>
    <a-col :span="6">
      <a-statistic title="Dining Expenses" :value="tripPlan.budget.total_meals" suffix="yuan" />
    </a-col>
    <a-col :span="6">
      <a-statistic title="Transportation" :value="tripPlan.budget.total_transportation" suffix="yuan" />
    </a-col>
  </a-row>
  <a-divider />
  <a-row>
    <a-col :span="24" style="text-align: center;">
      <a-statistic
        title="Estimated Total Cost"
        :value="tripPlan.budget.total"
        suffix="yuan"
        :value-style="{ color: '#cf1322', fontSize: '32px', fontWeight: 'bold' }"
      />
    </a-col>
  </a-row>
</a-card>
```

Đoạn mã này dùng bố cục lưới (grid layout) (`a-row` và `a-col`) để hiển thị bốn khoản chi phí cạnh nhau. Mỗi khoản chi phí dùng một component `a-statistic` để hiển thị tiêu đề và giá trị. Cuối cùng, một đường phân cách (`a-divider`) ngăn cách chúng, và bên dưới hiển thị tổng chi phí bằng phông chữ đỏ lớn để nhấn mạnh.

Lưu ý việc kết xuất có điều kiện `v-if="tripPlan.budget"`. Bởi vì thông tin ngân sách là tùy chọn (được định nghĩa là `Optional[Budget]` trong mô hình Pydantic), nếu LLM không tạo ra thông tin ngân sách, card này sẽ không được hiển thị. Điều này phản ánh khả năng dung sai lỗi của front-end đối với dữ liệu.

### 13.6.2 Thanh Tiến trình Tải (Loading Progress Bar)

Việc tạo một kế hoạch du lịch là một thao tác tốn thời gian. Back-end cần lần lượt gọi AttractionSearchAgent, WeatherQueryAgent, HotelAgent và PlannerAgent, và mỗi Agent cần gọi LLM và các API bên ngoài. Toàn bộ quá trình có thể mất 10-30 giây. Nếu người dùng nhấn nút "Start Planning" mà trang không có phản hồi gì, người dùng sẽ nghĩ hệ thống bị treo và có thể làm mới trang hoặc nhấn liên tục.

Để cải thiện trải nghiệm người dùng, chúng ta đã thêm một thanh tiến trình tải và các thông báo trạng thái. Hiện tại, nó chỉ là tiến trình mô phỏng, nhưng nó giúp người dùng biết rằng hệ thống đang làm việc.

```typescript
const loading = ref(false)
const loadingProgress = ref(0)
const loadingStatus = ref('')

const handleSubmit = async () => {
  loading.value = true
  loadingProgress.value = 0

  // Simulate progress updates
  const progressInterval = setInterval(() => {
    if (loadingProgress.value < 90) {
      loadingProgress.value += 10
      if (loadingProgress.value <= 30) loadingStatus.value = '🔍 Searching for attractions...'
      else if (loadingProgress.value <= 50) loadingStatus.value = '🌤️ Querying weather...'
      else if (loadingProgress.value <= 70) loadingStatus.value = '🏨 Recommending hotels...'
      else loadingStatus.value = '📋 Generating itinerary...'
    }
  }, 500)

  try {
    const response = await generateTripPlan(formData.value)
    clearInterval(progressInterval)
    loadingProgress.value = 100
    loadingStatus.value = '✅ Complete!'
    router.push({ name: 'result', state: { tripPlan: response } })
  } catch (error) {
    clearInterval(progressInterval)
    message.error('Failed to generate plan')
  } finally {
    loading.value = false
  }
}
```

### 13.6.3 Tính năng Chỉnh sửa Hành trình

Mặc dù các kế hoạch du lịch do AI tạo ra rất thông minh, chúng có thể không đáp ứng hoàn toàn nhu cầu cá nhân của người dùng. Ví dụ, người dùng có thể không thích một điểm tham quan nào đó và muốn xóa nó, hoặc muốn điều chỉnh thứ tự các điểm tham quan. Chúng ta cung cấp tính năng chỉnh sửa hành trình cho phép người dùng tùy chỉnh hành trình của mình.

Cốt lõi của tính năng chỉnh sửa là **quản lý trạng thái (state management)**. Chúng ta cần duy trì hai trạng thái: kế hoạch hành trình hiện tại và kế hoạch hành trình gốc. Khi người dùng vào chế độ chỉnh sửa, chúng ta lưu một bản sao của kế hoạch gốc. Nếu người dùng hủy chỉnh sửa, chúng ta khôi phục kế hoạch gốc. Nếu người dùng lưu thay đổi, chúng ta cập nhật kế hoạch hiện tại:

```typescript
const editMode = ref(false)
const originalPlan = ref<TripPlan | null>(null)

// Enter edit mode
const toggleEditMode = () => {
  editMode.value = true
  originalPlan.value = JSON.parse(JSON.stringify(tripPlan.value))
}
```

Lưu ý rằng chúng ta dùng `JSON.parse(JSON.stringify(...))` để sao chép sâu (deep copy) đối tượng. Tại sao không gán trực tiếp? Bởi vì các đối tượng trong JavaScript là kiểu tham chiếu (reference type) - nếu chúng ta gán trực tiếp, `originalPlan` và `tripPlan` sẽ trỏ tới cùng một đối tượng, và việc sửa cái này sẽ ảnh hưởng đến cái kia. Việc sao chép sâu tạo ra một bản sao hoàn toàn độc lập.

Logic để di chuyển điểm tham quan là hoán đổi vị trí của hai phần tử trong mảng:

```typescript
// Move attraction
const moveAttraction = (dayIndex: number, attractionIndex: number, direction: 'up' | 'down') => {
  const attractions = tripPlan.value.days[dayIndex].attractions
  const newIndex = direction === 'up' ? attractionIndex - 1 : attractionIndex + 1

  if (newIndex >= 0 && newIndex < attractions.length) {
    [attractions[attractionIndex], attractions[newIndex]] =
    [attractions[newIndex], attractions[attractionIndex]]
  }
}
```

Điều này dùng cú pháp gán bằng phân rã (destructuring assignment) của ES6 để hoán đổi hai phần tử. `[a, b] = [b, a]` là một cách thanh lịch để hoán đổi mà không cần biến tạm.

Việc xóa điểm tham quan dùng phương thức `splice` của mảng:

```typescript
// Delete attraction
const deleteAttraction = (dayIndex: number, attractionIndex: number) => {
  tripPlan.value.days[dayIndex].attractions.splice(attractionIndex, 1)
}
```

Khi lưu thay đổi, chúng ta cần khởi tạo lại bản đồ bởi vì vị trí các điểm tham quan có thể đã thay đổi:

```typescript
// Save changes
const saveChanges = () => {
  editMode.value = false
  message.success('Changes saved')
  initMap()  // Reinitialize map
}

// Cancel editing
const cancelEdit = () => {
  if (originalPlan.value) {
    tripPlan.value = originalPlan.value
  }
  editMode.value = false
}
```

Trong template, chúng ta hiển thị UI khác nhau dựa trên giá trị của `editMode`. Trong chế độ chỉnh sửa, các nút lên, xuống và xóa được hiển thị bên cạnh mỗi điểm tham quan:

```vue
<div v-if="editMode" class="edit-buttons">
  <a-button size="small" @click="moveAttraction(dayIndex, index, 'up')">Up</a-button>
  <a-button size="small" @click="moveAttraction(dayIndex, index, 'down')">Down</a-button>
  <a-button size="small" danger @click="deleteAttraction(dayIndex, index)">Delete</a-button>
</div>
```

### 13.6.4 Chức năng Xuất File

Sau khi người dùng tạo ra một kế hoạch du lịch ưng ý, họ có thể muốn lưu nó hoặc chia sẻ nó với bạn bè. Chúng ta cung cấp hai phương thức xuất: xuất thành hình ảnh và xuất thành PDF.

Cốt lõi của chức năng xuất file là thư viện `html2canvas`. Thư viện này có thể chuyển đổi các phần tử DOM thành Canvas, sau đó chúng ta có thể xuất Canvas thành hình ảnh. Nhưng ở đây có một thách thức kỹ thuật: bản đồ được kết xuất bằng Canvas, và `html2canvas` gặp vấn đề tương thích khi xử lý Canvas lồng nhau.

Chúng ta đã thử nhiều giải pháp, bao gồm việc chuyển Canvas bản đồ thành hình ảnh trước khi xuất, nhưng do cơ chế kết xuất Canvas của Amap và các hạn chế cross-origin, giải pháp này không giải quyết hoàn toàn được vấn đề. Trong các dự án thực tế, bạn có thể cần cân nhắc các giải pháp thay thế sau:

1. **Dùng API bản đồ tĩnh của Amap**: Gọi công cụ `maps_staticmap` để tạo hình ảnh bản đồ tĩnh thay thế cho bản đồ động
2. **Xuất riêng biệt**: Xuất bản đồ và nội dung hành trình riêng biệt, sau đó gộp chúng ở back-end
3. **Dùng dịch vụ chụp màn hình**: Dùng các trình duyệt headless như Puppeteer để chụp màn hình ở phía máy chủ
4. **Đơn giản hóa nội dung xuất**: Ẩn bản đồ khi xuất, chỉ xuất nội dung văn bản

Trong hiện thực hiện tại, chúng ta áp dụng cách tiếp cận đơn giản hóa, tạm thời ẩn phần bản đồ khi xuất và chỉ xuất nội dung văn bản và thông tin điểm tham quan của hành trình. Mặc dù đây không phải là giải pháp lý tưởng, nó đảm bảo chức năng xuất file có thể sử dụng được.

Logic để xuất thành hình ảnh rất đơn giản:

```typescript
import html2canvas from 'html2canvas'

const exportAsImage = async () => {
  const element = document.getElementById('trip-plan-content')
  if (!element) return

  const canvas = await html2canvas(element, {
    backgroundColor: '#ffffff',
    scale: 2,
    useCORS: true
  })

  const link = document.createElement('a')
  link.download = `${tripPlan.value.city} Travel Plan.png`
  link.href = canvas.toDataURL('image/png')
  link.click()
  message.success('Export successful!')
}
```

`scale: 2` nghĩa là dùng độ phân giải gấp 2 lần, làm cho hình ảnh xuất ra rõ nét hơn. `useCORS: true` cho phép tải hình ảnh cross-origin, điều này quan trọng đối với hình ảnh điểm tham quan (từ Unsplash).

Việc xuất thành PDF đòi hỏi thêm các bước: đầu tiên chuyển thành Canvas, rồi chuyển thành hình ảnh, và cuối cùng thêm vào PDF:

```typescript
import jsPDF from 'jspdf'

const exportAsPDF = async () => {
  // First capture map image
  await captureMapImage()

  const element = document.getElementById('trip-plan-content')
  if (!element) return

  const canvas = await html2canvas(element, {
    backgroundColor: '#ffffff',
    scale: 2,
    useCORS: true,
    allowTaint: true
  })

  // Restore map
  restoreMap()

  const pdf = new jsPDF('p', 'mm', 'a4')
  const imgData = canvas.toDataURL('image/png')
  const imgWidth = 210  // A4 width
  const imgHeight = (canvas.height * imgWidth) / canvas.width

  pdf.addImage(imgData, 'PNG', 0, 0, imgWidth, imgHeight)
  pdf.save(`${tripPlan.value.city} Travel Plan.pdf`)
  message.success('Export successful!')
}
```

Ở đây chúng ta cần tính toán chiều cao của hình ảnh để duy trì tỷ lệ khung hình. Chiều rộng của giấy A4 là 210mm, và chúng ta tính toán chiều cao tương ứng dựa trên tỷ lệ khung hình của Canvas.

### 13.6.5 Điều hướng Bên và Nhảy tới Anchor

Trang Result có rất nhiều nội dung, bao gồm tổng quan hành trình, chi tiết ngân sách, bản đồ, hành trình hàng ngày, thông tin thời tiết, v.v. Nếu người dùng muốn nhảy nhanh tới một mục nào đó, họ phải cuộn một quãng dài. Chúng ta cung cấp tính năng điều hướng bên và nhảy tới anchor, cho phép người dùng định vị nhanh chóng.

Điều hướng bên dùng component Menu của Ant Design Vue:

```vue
<a-menu
  v-model:selectedKeys="[activeSection]"
  mode="inline"
  @click="scrollToSection"
>
  <a-menu-item key="overview">📋 Itinerary Overview</a-menu-item>
  <a-menu-item key="budget">💰 Budget Details</a-menu-item>
  <a-menu-item key="map">🗺️ Map</a-menu-item>
  <a-menu-item key="days">📅 Daily Itinerary</a-menu-item>
  <a-menu-item key="weather">🌤️ Weather</a-menu-item>
</a-menu>
```

Khi nhấn vào một mục menu, gọi hàm `scrollToSection`:

```typescript
const activeSection = ref('overview')

// Scroll to specified section
const scrollToSection = ({ key }: { key: string }) => {
  activeSection.value = key
  const element = document.getElementById(key)
  if (element) {
    element.scrollIntoView({ behavior: 'smooth', block: 'start' })
  }
}
```

`scrollIntoView` là một API gốc của trình duyệt có thể cuộn một phần tử vào vùng hiển thị. `behavior: 'smooth'` nghĩa là cuộn mượt mà thay vì nhảy tức thì. `block: 'start'` nghĩa là đỉnh của phần tử căn với đỉnh của vùng hiển thị.

Ở các phần khác nhau của trang, chúng ta cần thêm các id tương ứng:

```vue
<div id="overview">
  <!-- Itinerary overview content -->
</div>

<div id="budget">
  <!-- Budget details content -->
</div>

<div id="map">
  <!-- Map content -->
</div>
```

Bằng cách này, khi người dùng nhấn vào một mục menu trong điều hướng bên, trang sẽ cuộn mượt mà tới mục tương ứng.

Thông qua việc hiện thực các tính năng này, trợ lý du lịch thông minh của chúng ta không chỉ tạo ra các kế hoạch du lịch mà còn cung cấp các tính năng tương tác phong phú: tính toán ngân sách giúp người dùng hiểu chi phí, thanh tiến trình tải giúp việc chờ đợi bớt lo lắng, chỉnh sửa hành trình giúp kế hoạch cá nhân hóa hơn, chức năng xuất file cho phép chia sẻ và lưu trữ kế hoạch, và điều hướng bên giúp các trang dài dễ duyệt hơn. Sự kết hợp của các tính năng này tạo thành một ứng dụng Web hoàn chỉnh, thân thiện với người dùng và thiết thực.

## 13.7 Kết luận

Xin chúc mừng bạn đã hoàn thành Chương 13!

Thông qua chương này, bạn không chỉ học được cách xây dựng một ứng dụng trợ lý du lịch thông minh hoàn chỉnh, mà quan trọng hơn, bạn đã nắm vững:

1. **Tư duy Thiết kế Hệ thống**: Cách phân rã các vấn đề phức tạp thành nhiều tác vụ đơn giản
2. **Năng lực Thực hành Kỹ thuật**: Cách biến kiến thức lý thuyết thành mã có thể chạy được
3. **Năng lực Phát triển Full-Stack**: Cách tích hợp các technology stack front-end và back-end
4. **Phát triển Ứng dụng AI**: Cách dùng các LLM để xây dựng các ứng dụng thực tế

Dự án này là một điểm khởi đầu, không phải điểm kết thúc. Dựa trên dự án này, bạn có thể:

- Thêm nhiều tính năng hơn
- Tối ưu hóa trải nghiệm người dùng
- Mở rộng sang các lĩnh vực khác (chẳng hạn trợ lý mua sắm thông minh, trợ lý học tập thông minh, v.v.)
- Triển khai lên môi trường production để phục vụ người dùng thực

Cách học tốt nhất là thông qua thực hành. Đừng chỉ đọc mã - hãy tự mình sửa đổi, mở rộng và tối ưu hóa nó. Mỗi lần thực hành sẽ làm sâu sắc thêm hiểu biết của bạn về các hệ thống đa Agent.

Chúc bạn thành công trên hành trình phát triển ứng dụng AI của mình!
