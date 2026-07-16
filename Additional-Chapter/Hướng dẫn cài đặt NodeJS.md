# Hướng dẫn cài đặt Node.js và npx

## 📋 Mục lục

- [Tại sao cần cài đặt Node.js?](#tại-sao-cần-cài-đặt-nodejs)
- [Hướng dẫn cài đặt trên Windows](#hướng-dẫn-cài-đặt-trên-windows)
- [Hướng dẫn cài đặt trên macOS](#hướng-dẫn-cài-đặt-trên-macos)
- [Hướng dẫn cài đặt trên Linux](#hướng-dẫn-cài-đặt-trên-linux)
- [Xác minh cài đặt](#xác-minh-cài-đặt)
- [Các câu hỏi thường gặp](#các-câu-hỏi-thường-gặp)

---

## Tại sao cần cài đặt Node.js?

Trong phần học về giao thức MCP ở Chương 10, chúng ta cần sử dụng các máy chủ MCP do cộng đồng cung cấp. Hầu hết các máy chủ này được viết bằng JavaScript/TypeScript và cần môi trường chạy Node.js.

**Sau khi cài đặt Node.js, bạn sẽ có được**:
- ✅ **node**: môi trường chạy (runtime) JavaScript
- ✅ **npm**: trình quản lý gói của Node (Node Package Manager)
- ✅ **npx**: trình thực thi gói npm (tự động tải về và chạy các gói npm)

**Vai trò của npx**:
```bash
# 传统方式：需要先安装再运行
npm install -g @modelcontextprotocol/server-filesystem
server-filesystem

# 使用npx：自动下载并运行（推荐）
npx @modelcontextprotocol/server-filesystem
```

---

## Hướng dẫn cài đặt trên Windows

### Cách 1: Bộ cài đặt chính thức (khuyến nghị)

#### Bước 1: Tải bộ cài đặt

Truy cập trang chủ Node.js: https://nodejs.org/

Bạn sẽ thấy hai phiên bản:
- **LTS (phiên bản hỗ trợ dài hạn)**: khuyến nghị cho hầu hết người dùng ✅
- **Current (phiên bản mới nhất)**: chứa các tính năng mới nhất

**Khuyến nghị tải phiên bản LTS** (ví dụ: 20.x.x LTS)

#### Bước 2: Chạy chương trình cài đặt

1. Nhấp đúp vào tập tin `.msi` đã tải về
2. Nhấp "Next" để bắt đầu cài đặt
3. Chấp nhận thỏa thuận cấp phép
4. Chọn đường dẫn cài đặt (để mặc định là được)
5. **Quan trọng**: đảm bảo tích chọn các tùy chọn sau:
   - ✅ Node.js runtime
   - ✅ npm package manager
   - ✅ Add to PATH (tự động thêm vào biến môi trường)
6. Nhấp "Install" để bắt đầu cài đặt
7. Chờ cài đặt hoàn tất, nhấp "Finish"

#### Bước 3: Xác minh cài đặt

Mở **PowerShell** hoặc **Command Prompt** (CMD), nhập:

```powershell
# 检查Node.js版本
node -v
# 应该显示：v20.x.x

# 检查npm版本
npm -v
# 应该显示：10.x.x

# 检查npx版本
npx -v
# 应该显示：10.x.x
```

Nếu tất cả đều hiển thị số phiên bản bình thường thì việc cài đặt đã thành công! ✅

---

## Hướng dẫn cài đặt trên macOS

### Cách 1: Bộ cài đặt chính thức

#### Bước 1: Tải bộ cài đặt

Truy cập: https://nodejs.org/

Tải tập tin `.pkg` của **phiên bản LTS**

#### Bước 2: Cài đặt

1. Nhấp đúp vào tập tin `.pkg`
2. Thao tác theo hướng dẫn của trình cài đặt
3. Nhập mật khẩu quản trị viên
4. Hoàn tất cài đặt

#### Bước 3: Xác minh cài đặt

Mở **Terminal (cửa sổ dòng lệnh)**, nhập:

```bash
node -v
npm -v
npx -v
```

---

## Hướng dẫn cài đặt trên Linux

### Ubuntu/Debian

#### Cách 1: Sử dụng kho lưu trữ NodeSource (khuyến nghị)

```bash
# 更新包列表
sudo apt update

# 安装curl（如果还没有）
sudo apt install -y curl

# 添加NodeSource仓库（Node.js 20.x LTS）
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# 安装Node.js和npm
sudo apt install -y nodejs

# 验证安装
node -v
npm -v
npx -v
```

#### Cách 2: Sử dụng apt (phiên bản có thể cũ hơn)

```bash
sudo apt update
sudo apt install -y nodejs npm
```

---

### CentOS/RHEL/Fedora

```bash
# 添加NodeSource仓库
curl -fsSL https://rpm.nodesource.com/setup_20.x | sudo bash -

# 安装Node.js
sudo yum install -y nodejs

# 验证安装
node -v
npm -v
npx -v
```

---

### Arch Linux

```bash
# 使用pacman安装
sudo pacman -S nodejs npm

# 验证安装
node -v
npm -v
npx -v
```

---

## Xác minh cài đặt

### Các bước xác minh đầy đủ

Sau khi cài đặt xong, chạy các lệnh sau để xác minh đầy đủ:

```bash
# 1. 检查版本
node -v
npm -v
npx -v

# 2. 测试Node.js
node -e "console.log('Node.js 工作正常！')"

# 3. 测试npm
npm --version

# 4. 测试npx（运行一个简单的包）
npx cowsay "Hello MCP!"
```

### Kết quả mong đợi

```
v20.11.0
10.2.4
10.2.4
Node.js 工作正常！
10.2.4
 _____________
< Hello MCP! >
 -------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

---

## Kiểm tra kết nối máy chủ MCP

Sau khi cài đặt xong, hãy kiểm tra kết nối đến máy chủ MCP của cộng đồng:

### Kiểm tra máy chủ hệ thống tập tin

```bash
# 使用npx运行文件系统MCP服务器
npx -y @modelcontextprotocol/server-filesystem .
```

Nếu bạn thấy thông tin khởi động của máy chủ thì mọi thứ đều bình thường!

### Kiểm tra trong Python

Tạo tập lệnh kiểm tra `test_mcp.py`:

```python
import asyncio
from hello_agents.protocols import MCPClient

async def test():
    client = MCPClient([
        "npx", "-y",
        "@modelcontextprotocol/server-filesystem",
        "."
    ])
    
    async with client:
        tools = await client.list_tools()
        print(f"✅ 成功连接！可用工具: {[t['name'] for t in tools]}")

asyncio.run(test())
```

Chạy:

```bash
python test_mcp.py
```

---

## Các câu hỏi thường gặp

### Q1: Sau khi cài đặt không tìm thấy lệnh

**Windows**:
```powershell
# 检查环境变量
echo $env:PATH

# 手动添加Node.js到PATH
# 1. 右键"此电脑" -> "属性"
# 2. "高级系统设置" -> "环境变量"
# 3. 在"系统变量"中找到"Path"
# 4. 添加：C:\Program Files\nodejs\
```

**macOS/Linux**:
```bash
# 检查环境变量
echo $PATH

# 添加到~/.bashrc 或 ~/.zshrc
export PATH="/usr/local/bin:$PATH"
source ~/.bashrc  # 或 source ~/.zshrc
```

---

### Q2: npm rất chậm

Sử dụng nguồn máy chủ gương (mirror) trong nước (mirror Taobao):

```bash
# 临时使用
npm install --registry=https://registry.npmmirror.com

# 永久设置
npm config set registry https://registry.npmmirror.com

# 验证
npm config get registry
```

---

### Q3: Lỗi quyền truy cập của npx

**Windows**:
```powershell
# 以管理员身份运行PowerShell
```

**macOS/Linux**:
```bash
# 不要使用sudo运行npx
# 如果遇到权限问题，修复npm全局目录权限
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

---

### Q4: Xung đột phiên bản

Nếu bạn cần quản lý nhiều phiên bản Node.js, khuyến nghị sử dụng công cụ quản lý phiên bản:

**Windows**: [nvm-windows](https://github.com/coreybutler/nvm-windows)

```powershell
# 安装nvm-windows后
nvm install 20.11.0
nvm use 20.11.0
```

**macOS/Linux**: [nvm](https://github.com/nvm-sh/nvm)

```bash
# 安装nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# 安装Node.js
nvm install 20
nvm use 20
```

---

### Q5: npx tải gói rất chậm

```bash
# 方式1：使用国内镜像
npx --registry=https://registry.npmmirror.com @modelcontextprotocol/server-filesystem

# 方式2：先全局安装，再使用
npm install -g @modelcontextprotocol/server-filesystem
server-filesystem
```

---

## Bước tiếp theo

Sau khi cài đặt xong, bạn có thể:

1. ✅ Chạy `code/02_Connect2MCP.py` để kiểm tra kết nối máy khách MCP
2. ✅ Khám phá các máy chủ MCP của cộng đồng: https://github.com/modelcontextprotocol/servers
3. ✅ Tiếp tục học các nội dung khác của Chương 10

---

## Tài liệu tham khảo

- **Trang chủ Node.js**: https://nodejs.org/
- **Tài liệu npm**: https://docs.npmjs.com/
- **Tài liệu npx**: https://docs.npmjs.com/cli/v10/commands/npx
- **Danh sách máy chủ MCP**: https://github.com/modelcontextprotocol/servers
- **Mirror npm Taobao**: https://npmmirror.com/

---

**Chúc bạn học tập vui vẻ!** 🎉

Nếu có thắc mắc, vui lòng tham khảo phần các câu hỏi thường gặp hoặc tra cứu tài liệu chính thức.
