# Chương 16 Đồ án Tốt nghiệp - Xây dựng Ứng dụng Multi-Agent của Riêng Bạn

Xin chúc mừng bạn đã đến với chương cuối cùng của giáo trình Hello-Agents! Trong 15 chương trước, chúng ta đã xây dựng framework HelloAgents từ con số không và tìm hiểu về các khái niệm cốt lõi của agent, nhiều mô hình (paradigm) khác nhau, hệ thống công cụ (tool), cơ chế bộ nhớ (memory), giao thức giao tiếp (communication protocol), huấn luyện học tăng cường (reinforcement learning), và đánh giá hiệu năng. Trong các Chương 13-15, chúng ta cũng đã minh họa cách tích hợp toàn bộ kiến thức đã học thông qua ba đồ án thực hành hoàn chỉnh (Trợ lý Du lịch Thông minh, Agent Nghiên cứu Chuyên sâu Tự động, và Cyber Town).

Giờ đây, đã đến lúc bạn trở thành một người xây dựng hệ thống agent thực thụ! Chương này sẽ hướng dẫn bạn **xây dựng ứng dụng multi-agent của riêng mình** và chia sẻ thành quả của bạn với cộng đồng thông qua hợp tác mã nguồn mở (open-source).

## 16.1 Ý nghĩa của Đồ án Tốt nghiệp

### 16.1.1 Tại sao Cần Làm Đồ án Tốt nghiệp

Cách tốt nhất để học công nghệ không phải là đọc giáo trình, mà là **thực hành trực tiếp**. Qua các chương trước, bạn đã nắm vững các kiến thức lý thuyết và công cụ kỹ thuật để xây dựng hệ thống agent. Tuy nhiên, thách thức thực sự nằm ở chỗ: **Làm thế nào để áp dụng kiến thức này vào các vấn đề thực tế? Làm thế nào để thiết kế một hệ thống hoàn chỉnh? Làm thế nào để xử lý các trường hợp biên (edge case) và ngoại lệ khác nhau?**

Giá trị cốt lõi của đồ án tốt nghiệp là rèn luyện khả năng ứng dụng tổng hợp của bạn, tích hợp một cách có chọn lọc toàn bộ kiến thức đã học trước đây (các paradigm của agent, hệ thống công cụ, cơ chế bộ nhớ, giao thức giao tiếp, v.v.) vào một đồ án hoàn chỉnh.

Thông qua việc học tập và thực hành trong chương này, chúng tôi hy vọng bạn có thể tự mình thiết kế và triển khai một ứng dụng agent hoàn chỉnh, sử dụng thành thạo các chức năng khác nhau của framework HelloAgents, nắm vững các thao tác cơ bản với Git và GitHub, học cách viết tài liệu dự án rõ ràng, tham gia phát triển hợp tác trong cộng đồng mã nguồn mở, và cuối cùng có được một tác phẩm kỹ thuật mà bạn có thể trưng bày.

### 16.1.2 Hình thức của Đồ án Tốt nghiệp

Đồ án tốt nghiệp của bạn sẽ được nộp vào kho lưu trữ dự án đồng sáng tạo (co-creation) của Hello-Agents (thư mục `Co-creation-projects`) dưới hình thức một **dự án mã nguồn mở**. Các yêu cầu cụ thể như sau:

1. **Đặt tên Dự án**: Sử dụng định dạng `{tên-người-dùng-GitHub-của-bạn}-{tên-dự-án}`, ví dụ `jjyaoao-CodeReviewAgent`

2. **Nội dung Dự án**:
   - Một Jupyter Notebook có thể chạy được (tệp `.ipynb`) hoặc một script Python
   - Danh sách phụ thuộc (dependency) đầy đủ (`requirements.txt`)
   - Tài liệu README rõ ràng (`README.md`)
   - Tùy chọn: video demo, ảnh chụp màn hình, tập dữ liệu (dataset), v.v.

3. **Phương thức Nộp**: Nộp qua GitHub Pull Request (PR)

4. **Quy trình Đánh giá**: Các thành viên cộng đồng sẽ đánh giá mã của bạn, đưa ra đề xuất cải tiến, và hợp nhất (merge) vào kho lưu trữ chính sau khi được phê duyệt

## 16.2 Hướng dẫn Chọn Chủ đề Dự án

### 16.2.1 Nguyên tắc Chọn Chủ đề

Một đồ án tốt nghiệp tốt nên mang tính thực tiễn, giải quyết các vấn đề thực tế thay vì làm công nghệ chỉ để làm công nghệ. Chúng ta cần theo đuổi việc hoàn thành trong thời gian và nguồn lực hạn chế, đồng thời thể hiện rõ năng lực kỹ thuật của bạn.

### 16.2.2 Các Hướng Chủ đề Được Đề xuất

Dưới đây là một số hướng dự án được đề xuất - bạn có thể chọn một trong số đó hoặc tự đề xuất ý tưởng của riêng mình:

**(1) Công cụ Nâng cao Năng suất**

- **Trợ lý Đánh giá Mã Thông minh**: Tự động phân tích chất lượng mã, phát hiện các lỗi (bug) tiềm ẩn, đưa ra đề xuất tối ưu hóa
- **Trình Tạo Tài liệu Thông minh**: Tự động tạo tài liệu API và hướng dẫn sử dụng dựa trên mã
- **Trợ lý Họp Thông minh**: Ghi lại nội dung cuộc họp, tạo biên bản họp, trích xuất các mục hành động (action item)
- **Trợ lý Email Thông minh**: Tự động phân loại email, tạo bản nháp trả lời, nhắc nhở về những vấn đề quan trọng

**(2) Hỗ trợ Học tập**

- **Bạn học Thông minh**: Đề xuất tài nguyên học tập dựa trên tiến độ học tập, tạo bài tập, giải đáp thắc mắc
- **Trợ lý Bài báo Thông minh**: Giúp tìm tài liệu, tóm tắt bài báo, tạo trích dẫn (citation)
- **Gia sư Lập trình Thông minh**: Cung cấp bài tập lập trình, đánh giá mã, lập kế hoạch lộ trình học tập
- **Trợ lý Học Ngôn ngữ Thông minh**: Cung cấp luyện hội thoại, sửa ngữ pháp, mở rộng vốn từ vựng

**(3) Sáng tạo và Giải trí**

- **Trình Tạo Truyện Thông minh**: Tạo tiểu thuyết, kịch bản, thơ ca dựa trên đầu vào của người dùng
- **NPC Game Thông minh**: Tạo các nhân vật game có cá tính, có thể trò chuyện tự nhiên với người chơi
- **Gợi ý Âm nhạc Thông minh**: Gợi ý âm nhạc dựa trên tâm trạng và bối cảnh, tạo danh sách phát (playlist)
- **Trợ lý Công thức Nấu ăn Thông minh**: Gợi ý công thức dựa trên nguyên liệu và khẩu vị, tạo danh sách mua sắm

**(4) Phân tích Dữ liệu**

- **Nhà Phân tích Dữ liệu Thông minh**: Tự động phân tích dữ liệu, tạo biểu đồ trực quan hóa, viết báo cáo phân tích
- **Phân tích Chứng khoán Thông minh**: Phân tích dữ liệu chứng khoán và cảm xúc tin tức (news sentiment), đưa ra lời khuyên đầu tư
- **Giám sát Dư luận Thông minh**: Giám sát mạng xã hội và các trang tin tức, phân tích xu hướng dư luận
- **Phân tích Cạnh tranh Thông minh**: Thu thập thông tin đối thủ cạnh tranh, phân tích so sánh, tạo báo cáo

**(5) Dịch vụ Đời sống**

- **Trợ lý Sức khỏe Thông minh**: Ghi lại dữ liệu sức khỏe, đưa ra lời khuyên sức khỏe, lập kế hoạch tập luyện
- **Trợ lý Tài chính Thông minh**: Ghi lại thu chi, phân tích thói quen chi tiêu, đưa ra lời khuyên tài chính
- **Trợ lý Mua sắm Thông minh**: So sánh giá cả, gợi ý sản phẩm, tạo danh sách mua sắm
- **Điều khiển Nhà Thông minh**: Điều khiển các thiết bị nhà thông minh thông qua ngôn ngữ tự nhiên

### 16.2.3 Ví dụ về Chọn Chủ đề

Hãy minh họa cách chọn chủ đề và thiết kế một dự án thông qua một ví dụ cụ thể.

**Tên Dự án**: Trợ lý Đánh giá Mã Thông minh (CodeReviewAgent)

**Phân tích Vấn đề**: Đánh giá mã (code review) là một phần quan trọng trong phát triển phần mềm, nhưng việc đánh giá thủ công tốn thời gian và dễ bỏ sót vấn đề. Các công cụ phân tích tĩnh (static analysis) hiện có chỉ có thể tìm lỗi cú pháp và không thể hiểu được logic của mã, do đó cần một trợ lý thông minh có thể hiểu ngữ nghĩa của mã và cung cấp phân tích chuyên sâu.

**Chức năng Cốt lõi**: Dự án này sẽ triển khai phân tích chất lượng mã (kiểm tra phong cách mã, quy ước đặt tên, độ đầy đủ của chú thích), phát hiện lỗi tiềm ẩn (phát hiện lỗi logic, vấn đề điều kiện biên, rò rỉ tài nguyên), đề xuất tối ưu hóa hiệu năng (xác định các nút thắt hiệu năng, đề xuất giải pháp tối ưu), quét lỗ hổng bảo mật (phát hiện SQL injection, XSS và các vấn đề bảo mật khác), và đề xuất các thực hành tốt nhất (best practice) (đề xuất cải tiến dựa trên đặc điểm ngôn ngữ và các mẫu thiết kế - design pattern).

**Kết quả Mong đợi**: Sản phẩm bàn giao cuối cùng sẽ là một Jupyter Notebook có thể chạy được, minh họa quy trình đánh giá hoàn chỉnh, hỗ trợ các ngôn ngữ chính thống như Python và JavaScript, có khả năng tạo báo cáo đánh giá ở định dạng Markdown có cấu trúc, và cung cấp các ví dụ mã cụ thể cùng các đề xuất cải tiến.

## 16.3 Chuẩn bị Môi trường Phát triển

### 16.3.1 Cài đặt các Công cụ Cần thiết

Trước khi bắt đầu phát triển, hãy đảm bảo môi trường phát triển của bạn đã cài đặt các công cụ sau:

**(1) Môi trường Python**

```bash
# Install HelloAgents
pip install "hello-agents[all]"
```

**(2) Git và GitHub**

```bash
# Check Git version
git --version

# Configure Git user information
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Configure GitHub SSH key (recommended)
# 1. Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# 2. Add public key to GitHub
# Copy the content of ~/.ssh/id_ed25519.pub
# Add in GitHub Settings > SSH and GPG keys

# 3. Test connection
ssh -T git@github.com
```

**(3) Jupyter Notebook**

```bash
# Install Jupyter
pip install jupyter notebook

# Or use JupyterLab (recommended)
pip install jupyterlab

# Start Jupyter
jupyter lab
```

### 16.3.2 Fork Kho lưu trữ Dự án

**Bước 1: Fork Kho lưu trữ**

1. Truy cập kho lưu trữ Hello-Agents: https://github.com/datawhalechina/hello-agents
2. Nhấp vào nút "Fork" ở góc trên bên phải, như được hiển thị trong khung màu đỏ ở Hình 16.1
3. Chọn tài khoản GitHub của bạn và tạo bản Fork

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/hello-agents/main/docs/images/16-figures/16-1.png" alt="" width="85%"/>
  <p>Hình 16.1 Các bước Fork Kho lưu trữ</p>
</div>

**Bước 2: Clone về Máy cục bộ**

```bash
# As shown in Figure 16.2, clone your forked repository
git clone git@github.com:your-username/hello-agents.git

# Enter project directory
cd Hello-Agents

# Add upstream repository (for syncing updates)
git remote add upstream https://github.com/datawhalechina/hello-agents.git

# View remote repositories
git remote -v
```

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/hello-agents/main/docs/images/16-figures/16-2.png" alt="" width="85%"/>
  <p>Hình 16.2 Clone Kho lưu trữ về Máy cục bộ</p>
</div>

**Bước 3: Tạo Nhánh Phát triển**

```bash
# Create and switch to new branch
git checkout -b feature/your-project-name

# For example:
git checkout -b feature/code-review-agent
```

### 16.3.3 Cấu trúc Thư mục Dự án

Tạo thư mục dự án của bạn trong thư mục `Co-creation-projects`:

```bash
# Enter co-creation projects directory
cd Co-creation-projects

# Create project folder (format: GitHub-username-project-name)
mkdir your-username-project-name

# For example:
mkdir jjyaoao-CodeReviewAgent

# Enter project directory
cd jjyaoao-CodeReviewAgent
```

Cấu trúc dự án được đề xuất:

```
jjyaoao-CodeReviewAgent/
├── README.md              # Project documentation
├── requirements.txt       # Python dependency list
├── main.ipynb            # Main Jupyter Notebook
├── data/                 # Data files (optional)
│   ├── sample_code.py
│   └── test_cases.json
├── outputs/              # Output results (optional)
│   ├── review_report.md
│   └── screenshots/
├── src/                  # Source code (optional, if code is extensive)
│   ├── agents/
│   ├── tools/
│   └── utils/
└── .env.example          # Environment variable template
```

## 16.4 Hướng dẫn Phát triển Dự án

### 16.4.1 Viết Tài liệu README

README là bộ mặt của dự án của bạn. Một README tốt nên chứa các nội dung sau:

```markdown
# Project Name

> One-sentence description of your project

## 📝 Project Introduction

Detailed introduction to your project:
- What problem does it solve?
- What are its special features?
- What scenarios is it suitable for?

## ✨ Core Features

- [ ] Feature 1: Description
- [ ] Feature 2: Description
- [ ] Feature 3: Description

## 🛠️ Technology Stack

- HelloAgents framework
- Agent paradigms used (e.g., ReAct, Plan-and-Solve, etc.)
- Tools and APIs used
- Other dependency libraries

## 🚀 Quick Start

### Environment Requirements

- Python 3.10+
- Other requirements

### Install Dependencies


pip install -r requirements.txt


### Configure API Keys


# Create .env file
cp .env.example .env

# Edit .env file and fill in your API keys


### Run Project


# Start Jupyter Notebook
jupyter lab

# Open main.ipynb and run


## 📖 Usage Examples

Show how to use your project, preferably with code examples and results.

## 🎯 Project Highlights

- Highlight 1: Explanation
- Highlight 2: Explanation
- Highlight 3: Explanation

## 📊 Performance Evaluation

If you have evaluation results, display them here:
- Accuracy: XX%
- Response time: XX seconds
- Other metrics

## 🔮 Future Plans

- [ ] Feature 1 to be implemented
- [ ] Feature 2 to be implemented
- [ ] Parts to be optimized

## 🤝 Contribution Guidelines

Issues and Pull Requests are welcome!

## 📄 License

MIT License

## 👤 Author

- GitHub: [@your-username](https://github.com/your-username)
- Email: your.email@example.com (optional)

## 🙏 Acknowledgments

Thanks to the Datawhale community and Hello-Agents project!
```

### 16.4.2 Viết requirements.txt

Liệt kê tất cả các phụ thuộc Python cần thiết cho dự án:

```txt
# Core dependencies
hello-agents[all]>=0.2.7

# Visualization (if needed)
matplotlib>=3.7.0
plotly>=5.14.0

# Web framework (if needed)
fastapi>=0.109.0
uvicorn>=0.27.0
```

### 16.4.3 Phát triển Jupyter Notebook

**(1) Đề xuất về Cấu trúc Notebook**

Một Jupyter Notebook tốt nên chứa các phần sau:

```python
# ========================================
# Part 1: Project Introduction
# ========================================

"""
# Project Name

## Project Introduction
Brief introduction to project goals and features

## Author Information
- Name: XXX
- GitHub: @XXX
- Date: 2025-XX-XX
"""

# ========================================
# Part 2: Environment Configuration
# ========================================

# Install dependencies
!pip install -q hello-agents[all]

# Import necessary libraries
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import BaseTool
import os
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# ========================================
# Part 3: Tool Definition
# ========================================

class CustomTool(BaseTool):
    """Custom tool class"""

    name = "tool_name"
    description = "Tool description"

    def run(self, query: str) -> str:
        """Tool execution logic"""
        # Implement your tool logic
        return "Result"

# ========================================
# Part 4: Agent Construction
# ========================================

# Create LLM
llm = HelloAgentsLLM()

# Create agent
agent = SimpleAgent(
    name="Agent Name",
    llm=llm,
    system_prompt="System prompt"
)

# Add tools
agent.add_tool(CustomTool())

# ========================================
# Part 5: Feature Demonstration
# ========================================

# Example 1: Basic functionality
print("=== Example 1: Basic Functionality ===")
result = agent.run("User input")
print(result)

# Example 2: Complex scenario
print("\n=== Example 2: Complex Scenario ===")
result = agent.run("Complex user input")
print(result)

# ========================================
# Part 6: Performance Evaluation (Optional)
# ========================================

# Evaluation code
# ...

# ========================================
# Part 7: Summary and Outlook
# ========================================

"""
## Project Summary

### Implemented Features
- Feature 1
- Feature 2

### Challenges Encountered
- Challenge 1 and solution
- Challenge 2 and solution

### Future Improvement Directions
- Improvement 1
- Improvement 2
"""
```

### 16.4.4 Kiểm thử Dự án của Bạn

Trước khi nộp, hãy sử dụng danh sách kiểm tra (checklist) này để xác định xem dự án của bạn có đáp ứng các yêu cầu nộp hay không:

```markdown
- [ ] Code runs normally without errors
- [ ] README documentation is complete with clear instructions
- [ ] requirements.txt contains all dependencies
- [ ] Clear usage examples provided
- [ ] Code has appropriate comments
- [ ] Output results meet expectations
- [ ] Common exception cases handled
- [ ] Project structure is clear with standardized file naming
- [ ] Large files properly handled (see next section)
```

### 16.4.5 Hướng dẫn Xử lý Tệp Lớn

**⚠️ Quan trọng: Tránh làm Kho lưu trữ Chính Quá lớn**

Để giữ cho kho lưu trữ chính Hello-Agents gọn nhẹ, hãy tuân theo các hướng dẫn xử lý tệp lớn sau:

**(1) Giới hạn Kích thước Tệp**

- **Tổng kích thước dự án**: Không vượt quá 5MB
- **Cấm nộp trực tiếp**: Tệp video, tập dữ liệu lớn, tệp mô hình (model)

**(2) Các Giải pháp Xử lý Tệp Lớn**

Nếu dự án của bạn chứa các tệp lớn (tập dữ liệu, video, mô hình, v.v.), hãy sử dụng các giải pháp sau:

**Giải pháp 1: Sử dụng Liên kết Ngoài (Khuyến nghị)**

Tải các tệp lớn lên các nền tảng bên ngoài và cung cấp liên kết tải xuống trong README:

```markdown
## Datasets

The datasets used in this project are large. Please download from the following links:

- Dataset 1: [Baidu Netdisk](link) Extraction code: xxxx
- Dataset 2: [Google Drive](link)
- Demo video: [Bilibili](link) / [YouTube](link)
```

Các nền tảng bên ngoài được đề xuất:
- **Tập dữ liệu**: Baidu Netdisk, Google Drive, Kaggle, HuggingFace Datasets
- **Video**: Bilibili, YouTube, Tencent Video
- **Mô hình**: HuggingFace Models, ModelScope
- **Hình ảnh**: GitHub Issues, dịch vụ lưu trữ hình ảnh (image hosting)

**Giải pháp 2: Tạo Kho lưu trữ Độc lập**

Nếu dự án có nhiều tài nguyên, hãy cân nhắc tạo một kho lưu trữ dữ liệu độc lập:

```markdown
## Project Resources

Due to the large amount of data and demo resources, a separate resource repository has been created:

- Resource repository: https://github.com/your-username/project-name-resources
- Contains: Datasets, demo videos, model files, test data, etc.

### Usage

\`\`\`bash
# Clone resource repository
git clone https://github.com/your-username/project-name-resources.git

# Copy data to project directory
cp -r project-name-resources/data ./data
\`\`\`
```

**Giải pháp 3: Sử dụng Dữ liệu Mẫu**

Chỉ cung cấp dữ liệu mẫu quy mô nhỏ trong kho lưu trữ chính:

```python
# Explain in README
## Data Description

- `data/sample.csv`: Sample data (100 records)
- Complete dataset (100,000 records) download from [here](link)
```

**(3) Ví dụ về Thực hành Tốt nhất**

```
your-username-project-name/
├── README.md              # Contains external resource links
├── requirements.txt
├── main.ipynb
├── .gitignore            # Ignore large files
├── data/
│   └── sample.csv        # Sample data only (<1MB)
└── outputs/
    └── demo_result.png   # Demo results only (<1MB)
```

Phần giải thích trong README:

```markdown
## Data and Resources

### Sample Data
Project includes small-scale sample data for quick testing (located in `data/sample.csv`)

### Complete Dataset
Complete dataset (500MB) download from the following link:
- Baidu Netdisk: [Link] Extraction code: xxxx
- Extract to `data/` directory after download

### Demo Video
- Bilibili: [Project Demo Video](link)
- YouTube: [Demo Video](link)
```

## 16.5 Nộp Pull Request

### 16.5.1 Nộp Mã lên GitHub

**Bước 1: Kiểm tra Các Thay đổi**

```bash
# View modified files
git status
```

**Bước 2: Thêm Tệp**

```bash
# Add all modified files
git add .

# Or add specific files
git add Co-creation-projects/your-username-project-name/
```

**Bước 3: Commit Các Thay đổi**

Thông điệp commit nên tuân theo định dạng sau:

```bash
# Format: type: brief description
git commit -m "feat: Add XXX graduation project"
```

**Quy ước về Loại Commit:**

- `feat`: Tính năng hoặc dự án mới (dùng loại này cho đồ án tốt nghiệp)
- `fix`: Sửa lỗi (bug fix)
- `docs`: Cập nhật tài liệu
- `style`: Điều chỉnh định dạng mã (không ảnh hưởng đến chức năng)
- `refactor`: Tái cấu trúc mã (refactoring)
- `test`: Liên quan đến kiểm thử
- `chore`: Các thay đổi khác (ví dụ: cập nhật phụ thuộc)

**Bước 4: Push lên GitHub**

```bash
# Push to your forked repository
git push origin feature/your-project-name
```

### 16.5.2 Tạo Pull Request

**Bước 1: Truy cập GitHub**

1. Truy cập kho lưu trữ đã fork của bạn: `https://github.com/your-username/hello-agents`
2. Nhấp vào tab "Pull requests", như được hiển thị trong Hình 16.3
3. Nhấp vào nút "New pull request"

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/hello-agents/main/docs/images/16-figures/16-3.png" alt="" width="85%"/>
  <p>Hình 16.3 Tạo Pull Request</p>
</div>

**Bước 2: Chọn Nhánh**

- Base repository: `datawhalechina/hello-agents`
- Base branch: `main`
- Head repository: `your-username/hello-agents`
- Compare branch: `feature/your-project-name`

**Bước 3: Điền Thông tin PR**

**⚠️ Quan trọng: Định dạng Tiêu đề PR Thống nhất**

Để dễ dàng quản lý và truy xuất, tất cả tiêu đề PR của đồ án tốt nghiệp phải tuân theo định dạng sau:

```
[Graduation Project] Project Name - Brief Description
```

Ví dụ:
- `[Graduation Project] CodeReviewAgent - Intelligent Code Review Assistant`
- `[Graduation Project] StudyBuddy - AI Learning Partner`
- `[Graduation Project] DataAnalyst - Intelligent Data Analyst`

**Mẫu Mô tả PR:**

```markdown
## Project Information

- **Project Name**: XXX
- **Author**: @your-username
- **Project Type**: Productivity Tool/Learning Assistance/Creative Entertainment/Data Analysis/Life Service

## Project Introduction

Brief description of your project (2-3 sentences)

## Core Features

- [ ] Feature 1
- [ ] Feature 2
- [ ] Feature 3

## Technical Highlights

- Used XXX paradigm
- Implemented XXX functionality
- Optimized XXX performance

## Demo Effects

(Optional) Add screenshots or GIFs to showcase project effects

## Self-Check List

- [ ] Code runs normally
- [ ] README documentation complete
- [ ] requirements.txt complete
- [ ] Clear usage examples provided
- [ ] Code has appropriate comments

## Other Notes

(Optional) Other content that needs explanation
```

**Bước 4: Nộp PR**

Như được hiển thị trong Hình 16.4, nhấp vào nút "Create pull request" để nộp.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/hello-agents/main/docs/images/16-figures/16-4.png" alt="" width="85%"/>
  <p>Hình 16.4 Nộp Pull Request</p>
</div>

### 16.5.3 Phản hồi các Bình luận Đánh giá

Sau khi nộp PR, các thành viên cộng đồng sẽ đánh giá mã của bạn và đưa ra đề xuất. Vui lòng phản hồi kịp thời:

1. **Xem Bình luận**: Kiểm tra các bình luận của người đánh giá (reviewer) trên trang PR
2. **Sửa Mã**: Sửa mã dựa trên các đề xuất
3. **Nộp Bản cập nhật**:
   ```bash
   git add .
   git commit -m "fix: Modify XXX based on review comments"
   git push origin feature/your-project-name
   ```
4. **Trả lời Bình luận**: Trả lời người đánh giá trên GitHub, giải thích các thay đổi của bạn

## 16.6 Trưng bày Dự án Mẫu

Để giúp bạn hiểu rõ hơn các yêu cầu của đồ án tốt nghiệp, dưới đây là một dự án mẫu hoàn chỉnh. Đừng lo lắng - những ý tưởng sáng tạo nhỏ cũng có thể được đưa vào. Bất kỳ tác phẩm nào bạn tự tạo ra đều đáng được trân trọng.

**Thông tin Dự án**

- **Tên Dự án**: CodeReviewAgent
- **Tác giả**: @jjyaoao
- **Đường dẫn Dự án**: `Co-creation-projects/jjyaoao-CodeReviewAgent/`

**Cấu trúc Dự án**

```
jjyaoao-CodeReviewAgent/
├── README.md              # Project documentation
├── requirements.txt       # Dependency list
├── main.ipynb            # Main program (includes quick demo and full features)
├── .env.example          # Environment variable example
├── .gitignore            # Git ignore rules
├── data/
│   └── sample_code.py    # Sample code
└── outputs/
    └── review_report.md  # Sample report
```

**Đoạn Mã Cốt lõi (main.ipynb)**

```python
# ========================================
# Intelligent Code Review Assistant
# ========================================

from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.tools import Tool, ToolParameter
from typing import Dict, Any, List
import ast
import os

# ========================================
# 0. Configure LLM Parameters
# ========================================

os.environ["LLM_MODEL_ID"] = "Qwen/Qwen2.5-72B-Instruct"
os.environ["LLM_API_KEY"] = "your_api_key_here"
os.environ["LLM_BASE_URL"] = "https://api-inference.modelscope.cn/v1/"
os.environ["LLM_TIMEOUT"] = "60"

# ========================================
# 1. Define Code Analysis Tools
# ========================================

class CodeAnalysisTool(Tool):
    """Code static analysis tool"""

    def __init__(self):
        super().__init__(
            name="code_analysis",
            description="Analyze Python code structure, complexity, and potential issues"
        )

    def run(self, parameters: Dict[str, Any]) -> str:
        """Analyze code and return results"""
        code = parameters.get("code", "")
        if not code:
            return "Error: Code cannot be empty"

        try:
            tree = ast.parse(code)
            functions = [node for node in ast.walk(tree)
                        if isinstance(node, ast.FunctionDef)]
            classes = [node for node in ast.walk(tree)
                      if isinstance(node, ast.ClassDef)]

            result = {
                "Number of functions": len(functions),
                "Number of classes": len(classes),
                "Lines of code": len(code.split('\n')),
                "Function list": [f.name for f in functions],
                "Class list": [c.name for c in classes]
            }
            return str(result)
        except SyntaxError as e:
            return f"Syntax error: {str(e)}"

    def get_parameters(self) -> List[ToolParameter]:
        return [
            ToolParameter(
                name="code",
                type="string",
                description="Python code to analyze",
                required=True
            )
        ]

class StyleCheckTool(Tool):
    """Code style checking tool"""

    def __init__(self):
        super().__init__(
            name="style_check",
            description="Check if code complies with PEP 8 standards"
        )

    def run(self, parameters: Dict[str, Any]) -> str:
        """Check code style"""
        code = parameters.get("code", "")
        if not code:
            return "Error: Code cannot be empty"

        issues = []
        lines = code.split('\n')
        for i, line in enumerate(lines, 1):
            if len(line) > 79:
                issues.append(f"Line {i}: Exceeds 79 characters")
            if line.startswith(' ') and not line.startswith('    '):
                if len(line) - len(line.lstrip()) not in [0, 4, 8, 12]:
                    issues.append(f"Line {i}: Non-standard indentation")

        if not issues:
            return "Code style is good, complies with PEP 8 standards"
        return "Found the following issues:\n" + "\n".join(issues)

    def get_parameters(self) -> List[ToolParameter]:
        return [
            ToolParameter(
                name="code",
                type="string",
                description="Python code to check",
                required=True
            )
        ]

# ========================================
# 2. Create Tool Registry and Agent
# ========================================

# Create tool registry
tool_registry = ToolRegistry()
tool_registry.register_tool(CodeAnalysisTool())
tool_registry.register_tool(StyleCheckTool())

# Initialize LLM
llm = HelloAgentsLLM()

# Define system prompt
system_prompt = """You are an experienced code review expert. Your tasks are:

1. Use code_analysis tool to analyze code structure
2. Use style_check tool to check code style
3. Based on analysis results, provide detailed review report

The review report should include:
- Code structure analysis
- Style issues
- Potential bugs
- Performance optimization suggestions
- Best practice recommendations

Please output the report in Markdown format."""

# Create agent
agent = SimpleAgent(
    name="Code Review Assistant",
    llm=llm,
    system_prompt=system_prompt,
    tool_registry=tool_registry
)

# ========================================
# 3. Run Example
# ========================================

# Read sample code
with open("data/sample_code.py", "r", encoding="utf-8") as f:
    sample_code = f.read()

print("=== Code to Review ===")
print(sample_code)
print("\n" + "="*50 + "\n")

# Execute code review
print("=== Starting Code Review ===")
review_result = agent.run(f"Please review the following Python code:\n\n```python\n{sample_code}\n```")

print(review_result)

# Save review report
with open("outputs/review_report.md", "w", encoding="utf-8") as f:
    f.write(review_result)

print("\nReview report saved to outputs/review_report.md")
```

**Ví dụ README.md**

```markdown
# CodeReviewAgent - Intelligent Code Review Assistant

> Intelligent code review tool based on HelloAgents framework

## 📝 Project Introduction

CodeReviewAgent is an intelligent code review assistant that can automatically analyze Python code quality, discover potential issues, and provide optimization suggestions.

### Core Features

- ✅ Code structure analysis: Count functions, classes, lines of code, etc.
- ✅ Style checking: Check compliance with PEP 8 standards
- ✅ Intelligent suggestions: Provide in-depth analysis and optimization suggestions based on LLM
- ✅ Report generation: Generate review reports in Markdown format

## 🛠️ Technology Stack

- HelloAgents framework (SimpleAgent + ToolRegistry)
- Python AST module (code parsing)
- ModelScope API (Qwen2.5-72B model)

## 🚀 Quick Start

### Install Dependencies

\`\`\`bash
pip install -r requirements.txt
\`\`\`

### Configure LLM Parameters

**Method 1: Use .env file**

\`\`\`bash
cp .env.example .env
# Edit .env file and fill in your API key
\`\`\`

**Method 2: Set directly in Notebook**

The project is pre-configured with ModelScope API and can run directly. To modify, edit the configuration code in Part 1 of main.ipynb.

### Run Project

\`\`\`bash
jupyter lab
# Open main.ipynb and run all cells
\`\`\`

## 📖 Usage Example

1. Place code to review in `data/sample_code.py`
2. Run `main.ipynb`
3. View generated review report `outputs/review_report.md`

## 🎯 Project Highlights

- **Automation**: No need for manual line-by-line checking, automatically discovers issues
- **Intelligence**: Uses LLM to understand code semantics and provide in-depth suggestions
- **Extensibility**: Easy to add new checking rules and tools

## 👤 Author

- GitHub: [@jjyaoao](https://github.com/jjyaoao)
- Project link: [CodeReviewAgent](https://github.com/datawhalechina/hello-agents/tree/main/Co-creation-projects/jjyaoao-CodeReviewAgent)

## 🙏 Acknowledgments

Thanks to the Datawhale community and Hello-Agents project!
```

## 16.7 Tổng kết và Triển vọng

Bằng việc hoàn thành đồ án tốt nghiệp, bạn hẳn đã nắm vững quy trình hoàn chỉnh của việc thiết kế hệ thống agent: thiết kế kiến trúc hệ thống từ yêu cầu, sử dụng thành thạo các chức năng và thành phần khác nhau của framework HelloAgents, phát triển các công cụ tùy chỉnh để mở rộng khả năng của agent, hoàn thành toàn bộ quá trình phát triển dự án từ phân tích yêu cầu đến triển khai mã, học cách sử dụng Git và GitHub cho hợp tác mã nguồn mở, và viết tài liệu kỹ thuật rõ ràng.

Trong dự án này, chúng ta đã xây dựng framework HelloAgents từ con số không và sử dụng nó để triển khai nhiều ứng dụng thực tế. Hoàn thành đồ án tốt nghiệp mới chỉ là khởi đầu. Bạn có thể tiếp tục đào sâu việc học các paradigm và thuật toán agent khác, kỹ thuật prompt (prompt engineering) và kỹ thuật ngữ cảnh (context engineering), các cơ chế cộng tác multi-agent, và các kiến thức lý thuyết khác. Bạn cũng có thể mở rộng bộ công nghệ (technology stack) của mình bằng cách học phát triển web để xây dựng các ứng dụng hoàn chỉnh, học cơ sở dữ liệu để triển khai lưu trữ dữ liệu bền vững (data persistence), và học triển khai (deployment) để đưa ứng dụng lên trực tuyến. Bạn cũng có thể liên tục tối ưu hóa dự án của mình bằng cách thêm nhiều tính năng hơn, tối ưu hóa hiệu năng và trải nghiệm người dùng, và cải thiện việc kiểm thử cũng như tài liệu. Quan trọng hơn, hãy tích cực tham gia đóng góp cho cộng đồng bằng cách giúp đỡ những người học khác, tham gia phát triển framework Hello-Agents, và chia sẻ kinh nghiệm cũng như hiểu biết của bạn.

Từ agent đơn giản ở Chương 1 cho đến việc giờ đây có thể tự mình xây dựng các ứng dụng multi-agent hoàn chỉnh, bạn đã trải qua một hành trình học tập đầy thú vị. Nhưng đây không phải là điểm kết thúc - đây là một khởi đầu mới.

Công nghệ AI đang thay đổi nhanh chóng, và lĩnh vực agent tràn đầy những khả năng vô hạn. Chúng tôi hy vọng bạn có thể giữ vững sự tò mò và không ngừng học hỏi các công nghệ mới, dũng cảm sử dụng công nghệ AI để giải quyết các vấn đề thực tế và tạo ra giá trị, sẵn lòng chia sẻ kinh nghiệm và thành tựu của bạn với cộng đồng, và không ngừng trau chuốt tác phẩm của mình để theo đuổi sự xuất sắc.

Cuối cùng, cảm ơn bạn đã đọc trọn vẹn dự án này. Chúng tôi hy vọng bạn đã thu được điều gì đó trong quá trình học tập và rằng bạn có thể áp dụng những gì đã học vào các dự án thực tế, tạo ra những ứng dụng agent tuyệt vời. Tương lai của AI tràn đầy những khả năng vô hạn - hãy cùng nhau khám phá và sáng tạo!

**Hãy nhớ: Cách học tốt nhất là thông qua thực hành trực tiếp!**

Giờ đây, hãy bắt đầu xây dựng ứng dụng agent của riêng bạn! Chúng tôi mong chờ được thấy những tác phẩm xuất sắc của bạn trong thư mục Co-creation-projects!

Nếu bạn thấy dự án Hello-Agents hữu ích, hãy cho chúng tôi một ⭐Star!

---
<div align="center">
  <strong>🎓 Xin chúc mừng bạn đã hoàn thành giáo trình Hello-Agents! 🎉</strong>
</div>
