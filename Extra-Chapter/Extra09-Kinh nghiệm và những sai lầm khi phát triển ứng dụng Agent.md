# Kinh nghiệm và những sai lầm khi phát triển ứng dụng Agent

Sau khi học xong giáo trình Hello-Agents, nhiệm vụ cuối cùng là làm đồ án tốt nghiệp. Dùng kiến thức đã học để tự tay xây dựng một ứng dụng Agent. Đúng vào khoảng thời gian đó Code Agent đang rất hot, Cursor, Claude Code, Codex... nhà nào cũng đang đẩy mạnh sản phẩm của mình. Tôi nghĩ rằng đã luyện tay thì chi bằng làm lại một Code Agent, tự tay xây dựng một lần, mới thực sự hiểu được vì sao những sản phẩm này lại hữu ích, và rốt cuộc về mặt kỹ thuật chúng đã làm đúng điều gì.

Và thế là có dự án này.
Kho lưu trữ code Code Agent dựa trên framework Hello-Agents: https://github.com/datawhalechina/hello-agents/tree/main/Co-creation-projects/YYHDBL-HelloCodeAgentCli

Kho lưu trữ code MyCodeAgent sau khi tái cấu trúc: https://github.com/YYHDBL/MyCodeAgent.git

Bài viết này không phải là một giáo trình, mà là ghi chép về những cái bẫy tôi đã mắc phải, những đường vòng tôi đã đi, và cuối cùng đã giải quyết như thế nào trong quá trình làm dự án Code Agent này.

---

## Mục lục

- [Chương một: Xem quá nhiều best practice, ngược lại lại rơi vào cái bẫy lớn đầu tiên](#chương-một-xem-quá-nhiều-best-practice-ngược-lại-lại-rơi-vào-cái-bẫy-lớn-đầu-tiên)
- [Chương hai: Một sự cố với lệnh pipe — lần đầu tiên tôi thấy "không thể chẩn đoán" chí mạng đến mức nào](#chương-hai-một-sự-cố-với-lệnh-pipe--lần-đầu-tiên-tôi-thấy-không-thể-chẩn-đoán-chí-mạng-đến-mức-nào)
- [Chương ba: Vùng Goldilocks của thiết kế tool](#chương-ba-vùng-goldilocks-của-thiết-kế-tool)
- [Chương bốn: Prompt không phải là câu thần chú, mà là mặt điều khiển của Agent](#chương-bốn-prompt-không-phải-là-câu-thần-chú-mà-là-mặt-điều-khiển-của-agent)
- [Chương năm: Context không phải là vấn đề dung lượng bộ nhớ, mà là vấn đề điều phối sự chú ý](#chương-năm-context-không-phải-là-vấn-đề-dung-lượng-bộ-nhớ-mà-là-vấn-đề-điều-phối-sự-chú-ý)
- [Chương sáu: Khả năng quan sát biến hộp đen thành hộp kính](#chương-sáu-khả-năng-quan-sát-biến-hộp-đen-thành-hộp-kính)
- [Chương bảy: Phương pháp luận tổng quát rút ra từ một dự án](#chương-bảy-phương-pháp-luận-tổng-quát-rút-ra-từ-một-dự-án)


---

# Chương một: Xem quá nhiều best practice, ngược lại lại rơi vào cái bẫy lớn đầu tiên

Khi mới bắt tay vào viết code, tôi đã tra cứu rất nhiều thực tiễn thiết kế Agent trong ngành. Chẳng hạn như bài "Bài học kinh nghiệm về context engineering" mà đội ngũ Manus chia sẻ, và bài "Building agents with the Claude Agent SDK" chính thức của Anthropic. Nhìn những ông lớn hàng đầu này chia sẻ "best practice" không giấu giếm gì, tôi nghĩ: đằng nào bây giờ đã có Claude Code, cứ để AI giúp tôi triển khai hết những khái niệm cao cấp này chẳng phải là được sao?

Thế là, tôi không chút do dự chất đống đủ loại thiết kế trông có vẻ thanh lịch: bộ nhớ nhiều tầng (Memory System), context engineering phức tạp, hệ thống đa tác nhân (Multi-Agent)... Phải nói rằng Claude Code quả thực rất mạnh, rất nhanh đã giúp tôi sinh ra một đống code với logic phức tạp.

## Khởi đầu tan nát

Nhưng khi tôi đầy háo hức chạy phiên bản test đầu tiên, thực tế đã tát cho tôi một cái đau điếng: cả hệ thống nát bét.

Đối mặt với một yêu cầu chỉnh sửa vô cùng đơn giản, Agent như phát điên gọi bảy tám loại tool, tiến hành vài vòng "não trái đấu não phải". Cuối cùng, tôi chỉ thu về được một đoạn code khiếm khuyết chẳng thể nào chạy được, cùng với một hóa đơn nợ Token vượt ngân sách trầm trọng.

Nhìn màn hình đầy lỗi, tôi mới nhận ra: phát triển Agent rất khác với phát triển phần mềm truyền thống.

Trước kia khi làm phát triển backend truyền thống, chúng ta quen với việc vẽ sơ đồ kiến trúc trước rồi mới viết code. Bản vẽ đủ thanh lịch thì hệ thống sẽ vững chắc. Đây là bản năng của lập trình viên.

Nhưng phát triển Agent thì khác. Bạn đang làm việc với một mô hình lớn, mà bản thân nó mang tính xác suất — cùng một đầu vào, mỗi lần có thể cho bạn đầu ra hoàn toàn khác nhau.

Trên nền móng bất định này, tôi lại cưỡng ép chồng thêm một bộ kiến trúc phức tạp mà chính tôi cũng chưa kiểm chứng. Multi-Agent, Plan-and-Execute... những thiết kế này giao thoa với nhau, khiến tính bất định bị khuếch đại gấp bội.

Kết quả là: kiến trúc phức tạp không thể đỡ được phần đáy, ngược lại vì luồng chuyển trạng thái quá nhiều, tool giao thoa quá phức tạp, khiến mô hình sai càng lệch lạc hơn. Lỗi truyền qua truyền lại giữa các thành phần, tôi thậm chí chẳng biết bắt đầu điều tra từ đâu.

Những "best practice" của các ông lớn dĩ nhiên là đồ tốt, nhưng tôi đã bỏ qua một điểm: những kiến trúc phức tạp đó là kết quả tiến hóa sau khi họ đã mắc vô số cái bẫy, tiêu tốn lượng token khổng lồ, chứ không phải là điểm khởi đầu cho người mới.

## Đập đi làm lại

Nhìn đống code mà đến việc đọc file đơn giản cũng rơi vào vòng lặp chết này, tôi đã đưa ra một quyết định đi ngược lại tổ tông — xóa kho, đập đi làm lại.

Theo nguyên tắc "Less is more", tôi trực tiếp tái sử dụng phần thân cơ bản nhất của Hello-Agent, chạy thông đường liên kết ngắn nhất trước. Các thành phần cốt lõi được tinh giản chỉ còn lại mấy phần này:

| Thành phần | Trách nhiệm cốt lõi |
|------|----------|
| ReActAgent | Điều khiển vòng lặp nhận thức cơ bản Thought → Action → Observation |
| ToolRegistry | Chịu trách nhiệm đăng ký tool và phân phối lời gọi |
| ContextBuilder | Ghép nối các quy tắc hệ thống, lịch sử và bằng chứng môi trường |
| TerminalTool | Thực thi lệnh thực tế bên trong kho code mục tiêu |
| Message | Cấu trúc dữ liệu tin nhắn hội thoại thống nhất |

Về mặt code, tôi không dùng bất kỳ mẫu thiết kế hoa mỹ nào, mà trực tiếp gom chúng lại một cách thô bạo trong code_agent.py:

```python
self.terminal_tool = TerminalTool(
    workspace=str(self.paths.repo_root),
    timeout=60,
    confirm_dangerous=True,
    default_shell_mode=True,  
)
self.registry = ToolRegistry()
self.registry.register_tool(self.terminal_tool)
```

Nhìn phiên bản V0 thô sơ này chạy thông tác vụ tối giản đầu tiên trong terminal, tôi thở phào nhẹ nhõm.

![image-20260225143343219](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra09-figures/1.png)

Chỉ cần có thể hoàn thành hội thoại cơ bản nhất và gọi tool, chúng ta đã có thể kiểm chứng và tối ưu lặp lại trong môi trường tác vụ thực tế.

Chạy được trước đã, quan trọng hơn là đi được một bước.


---

# Chương hai: Một sự cố với lệnh pipe — lần đầu tiên tôi thấy "không thể chẩn đoán" chí mạng đến mức nào

Sau khi V0 chạy được, tôi bắt đầu buông lỏng ràng buộc với tool. Đằng nào kiến trúc đã được tinh giản, để mô hình tự do kết hợp lệnh chắc không sao chứ?

Thế là tôi bật đèn xanh cho Terminal Tool: `default_shell_mode=True`. Điều này có nghĩa là mô hình không chỉ chạy được lệnh đơn, mà còn viết được pipe, redirect, sub-command — giống như con người.

## Diễn biến sự cố

Hôm đó tôi đưa ra một yêu cầu đơn giản: "Giúp tôi tìm định nghĩa của hàm `process_data`".

Mô hình rất nhanh đưa ra một lệnh trông khá chuyên nghiệp:

```bash
rg -n "def process_data" src/ | grep -v test | sed -n '1,50p'
```

Combo mà các kỹ sư lâu năm hay dùng: trước tiên tìm định nghĩa hàm, lọc bỏ file test, rồi lấy 50 dòng đầu. Chẳng có vấn đề gì.

Nhưng kết quả thực thi là rỗng. Agent nhìn kết quả rỗng này, ngẩn ra một lúc, rồi bắt đầu cứu vãn:

**Vòng thử lại thứ nhất:**
```
Thought: Có vẻ không tìm thấy, có thể phạm vi tìm kiếm quá hẹp. Để tôi mở rộng phạm vi tìm kiếm.
Action: rg -n "process_data" | head -100
```

Vẫn rỗng.

**Vòng thử lại thứ hai:**
```
Thought: Có thể tôi nhớ nhầm tên hàm, thử khớp mờ xem.
Action: rg -n "process.*data" src/ --type py | grep "def "
```

Vẫn rỗng.

**Vòng thử lại thứ ba:**
```
Thought: Xem ra hàm này có thể không tồn tại, hoặc tôi đã tìm nhầm thư mục. Để tôi liệt kê cấu trúc thư mục trước.
Action: find src/ -name "*.py" | xargs grep -l "process_data"
```

Kết quả vẫn rỗng. Sau ba lần thử, nó bỏ cuộc, nói với tôi: "Tôi không tìm thấy định nghĩa của hàm `process_data` trong kho, có thể tên hàm bị sai, hoặc nó không nằm trong file Python."

Nhưng tôi tự tay vào kho xem, hàm đó rõ ràng nằm ở dòng 42 của `src/utils/helpers.py`.

## Quá trình điều tra

Tôi copy lệnh đó vào terminal tự chạy, phát hiện `rg` báo lỗi:

```bash
$ rg -n "def process_data" src/ | grep -v test | sed -n '1,50p'
Error: path 'src/' does not exist
```

Hóa ra thư mục làm việc khi tôi khởi động Agent không phải là thư mục gốc của dự án, mà là một thư mục con bên dưới dự án. `src/` tương đối so với thư mục hiện tại không tồn tại, rg trực tiếp báo lỗi rồi thoát.

Nhưng ở phía Agent, thông tin lỗi bị pipe nuốt mất. Vì lệnh dùng `|`, đầu ra lỗi của rg không được truyền tới stdout, mà bị pipe điều hướng thành đầu vào của lệnh tiếp theo. grep nhận được đầu vào rỗng, tất nhiên xuất ra rỗng; sed cũng rỗng.

**Lỗi bị nén bẹp trong đường liên kết.** Cái Agent thấy chỉ là một chuỗi rỗng, nó hoàn toàn không biết đầu nguồn đã thất bại.

Điều tệ nhất là, mô hình dựa trên thông tin lỗi này để đưa ra phán đoán hoàn toàn sai. Nó tưởng "quả thực không tìm thấy", rồi bắt đầu đủ kiểu cứu vãn: đổi từ khóa tìm kiếm, đổi thư mục, thậm chí nghi ngờ tôi có phải nhớ nhầm tên hàm không. Những hành động này đều dựa trên một phán đoán sai, tiêu tốn vô ích lượng lớn token.

![image-20260225143439420](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra09-figures/2.png)

## Hướng khắc phục lỗi lúc bấy giờ

Phản ứng đầu tiên của tôi là: tool Bash quá nguy hiểm, phải thêm giới hạn.

Thế là tôi viết một đống code kiểm tra an toàn:

```python
SHELL_META_TOKENS = ["|", "||", "&&", ";", ">", ">>", "<", "$(", "`"]
DANGEROUS_BASE_COMMANDS = {"rm", "chmod", "mv", "dd"}

def validate_command(cmd):
    # Kiểm tra có chứa pipe hoặc redirect không
    for token in SHELL_META_TOKENS:
        if token in cmd:
            return False, f"包含非法字符: {token}"
    
    # Kiểm tra lệnh cơ bản có nằm trong whitelist không
    base_cmd = cmd.split()[0]
    if base_cmd not in ALLOWED_COMMANDS:
        return False, f"命令 {base_cmd} 不在白名单"
    
    # Kiểm tra lệnh nguy hiểm
    if base_cmd in DANGEROUS_BASE_COMMANDS:
        return False, "危险命令，禁止执行"
    
    return True, "OK"
```

Nhưng tôi nhanh chóng phát hiện, shell quá linh hoạt. Bạn cấm `|`, nó có thể dùng sub-command `$(...)` để thay thế; bạn cấm `>`, nó có thể dùng `tee`; bạn cấm `rm`, nó có thể dùng `> file` để xóa sạch nội dung file.

Bản vá càng vá càng nhiều, code càng viết càng dài, nhưng vấn đề gốc rễ đó — "rốt cuộc là bước nào thất bại" — vẫn còn nguyên.

Ngay cả khi tôi bịt kín toàn bộ pipe và redirect, chỉ cho phép lệnh đơn đơn giản nhất, vấn đề vẫn còn:

```bash
rg "pattern" src/
```

Nếu trả về rỗng, tôi vẫn không biết là "trong kho thực sự không có", hay là "rg vì lỗi đường dẫn nên không thực thi". Mô hình vẫn không thể sửa lỗi một cách có mục tiêu.

## Định vị nguyên nhân gốc

Về sau tôi mới nghĩ thông, nguyên nhân gốc của chuyện này không phải là "lệnh quá nguy hiểm", mà là **không thể chẩn đoán**.

Cụ thể có ba vấn đề:

**Thứ nhất, nhiều bước bị nhồi vào một Action.** Pipe đóng gói mấy bước logic lại với nhau, trạng thái trung gian mất sạch. Agent chỉ thấy được kết quả cuối cùng, không thấy được quá trình thực thi.

**Thứ hai, tín hiệu quan sát chỉ có một trạng thái cuối.** Thành công, thất bại, kết quả rỗng, tất cả trộn lẫn vào nhau. Mô hình không phân biệt được "thực sự không tìm thấy" và "trong quá trình tìm kiếm bị lỗi".

**Thứ ba, mô hình không thể sửa lỗi có mục tiêu.** Nó không biết `rg`, `grep`, `sed` cái nào có vấn đề, bước tiếp theo chỉ có thể đoán mò. Thử lại không dựa trên "sửa lỗi", mà dựa trên "cầu may".

Cho mô hình mức độ tự do cao hơn, không phải là nâng cao giới hạn năng lực, mà là khuếch đại tính bất định. Nó quả thực có thể viết ra lệnh "thông minh" hơn, nhưng một khi bị lỗi, đến chính bạn cũng không thể điều tra ra nó "khôn quá hóa dại" ở bước nào.

## Cách làm hiện tại

Về sau tôi trực tiếp hạ cấp Bash — không phải xóa bỏ, mà là làm rõ định vị của nó: chỉ xử lý những nhu cầu góc cạnh mà các tool nguyên tử không phủ tới, không đi vào đường liên kết chính.

Các thao tác tần suất cao đều được tách thành tool nguyên tử:

| Tool | Chức năng | Định dạng trả về |
|------|------|----------|
| LS | Liệt kê thư mục | `{status, data: {entries}, text}` |
| Glob | Tìm file theo tên | `{status, data: {paths}, text}` |
| Grep | Tìm kiếm theo nội dung | `{status, data: {matches}, text}` |
| Read | Đọc kèm số dòng | `{status, data: {content}, text}` |

Mỗi tool đều có mã trạng thái rõ ràng:
- `success`: tác vụ hoàn thành, kết quả nằm trong data
- `partial`: tác vụ hoàn thành nhưng nội dung bị cắt bớt
- `error`: tác vụ thất bại, trong error có mã lỗi cụ thể

Ví dụ Glob không tìm thấy file:
```json
{
  "status": "success",
  "data": {"paths": []},
  "text": "No files matching '*.xyz' found"
}
```

Đường dẫn không tồn tại:
```json
{
  "status": "error",
  "error": {"code": "NOT_FOUND", "message": "Path 'src/' does not exist"}
}
```

Mô hình có thể phân biệt rõ ràng "thực sự không có" và "bị lỗi".

Ràng buộc cứng của Bash cũng được làm rõ:
- Cấm đọc/tìm/liệt kê: `ls`/`cat`/`head`/`grep`/`find`/`rg` những cái này đã có tool chuyên dụng
- Cấm tương tác: vim, nano, top, ssh
- Cấm mạng (mặc định): curl/wget bị cấm
- Blacklist: rm -rf /, sudo/su, mkfs/fdisk

Sau khi làm như vậy, việc debug trở nên đơn giản hơn nhiều. Có vấn đề chỉ cần xem log là biết bước nào:
- Glob trả về mảng rỗng → thực sự không có file này
- Glob trả về NOT_FOUND → đường dẫn sai
- Grep trả về timeout → phạm vi tìm kiếm quá lớn

Mô hình cũng có thể dựa vào mã lỗi cụ thể để quyết định bước tiếp theo: đường dẫn sai thì đổi đường dẫn, timeout thì thu nhỏ phạm vi, thực sự không tìm thấy thì báo cho người dùng.

## Kết luận chương này

**Khả năng chẩn đoán là tiền đề của khả năng phục hồi.**

Nếu không biết chỗ nào hỏng, thì không sửa được. Nếu không biết thất bại xảy ra ở bước nào, thì không thể sửa lỗi có mục tiêu.

Trong phát triển Agent, việc cho mô hình khả năng tự do kết hợp lệnh, nghe thì rất đẹp, nhưng thực chất là đang tạo ra hộp đen. Lệnh pipe trông có vẻ hiệu quả lại nén bẹp thông tin lỗi thành những kết quả rỗng không thể phân biệt được, khiến mô hình càng chạy càng xa trên con đường sai lầm.

Tool nguyên tử tuy các bước rườm rà, nhưng mỗi bước đều có đầu vào, đầu ra, trạng thái rõ ràng. Có vấn đề, bạn định vị được; mô hình sai, bạn sửa được.

**Khả năng kiểm soát quan trọng hơn nhiều so với việc hoàn thành tác vụ một lần.**


---

# Chương ba: Vùng Goldilocks của thiết kế tool — không phải càng tự do càng tốt, cũng không phải càng vụn càng tốt

Sau chương ba, tôi bắt đầu tách các tool ra. Cái chế độ vạn năng cái gì cũng quản của Terminal Tool quả thực có vấn đề, sau khi tách thành tool nguyên tử, việc debug trở nên rõ ràng hơn nhiều.

Nhưng tôi rất nhanh lại rơi vào một cái bẫy mới: **tách quá vụn**.

## Cả hai cực đoan tôi đều đã mắc

### Cực đoan A: Tool vạn năng

Cực đoan đầu tiên bạn đã thấy rồi. Một Terminal Tool cái gì cũng làm được: pipe, redirect, sub-command, biến môi trường — hoàn toàn mở toang.

Lúc đó tôi nghĩ, LLM thông minh thế này, cho nó đủ mức độ tự do, chắc có thể thao tác như một kỹ sư. Lệnh kết hợp kiểu `rg | grep | sed` này hiệu quả rất cao.

Kết quả thì bạn cũng biết rồi: lỗi bị pipe nuốt, mô hình đoán mò thử lại, token chảy ào ào, vấn đề vẫn chưa giải quyết.

### Cực đoan B: Nguyên tử hóa quá mức

Sau khi nhận ra tool vạn năng có vấn đề, tôi lại đi sang một cực đoan khác: tách mỗi điểm chức năng thành một tool độc lập, theo đuổi sự nguyên tử hóa cực độ.

Lúc đó danh sách tool của tôi trông thế này:
- `ListDir`: liệt kê nội dung thư mục
- `ListDirRecursive`: liệt kê thư mục đệ quy
- `FindByName`: tìm theo tên file
- `FindByPattern`: tìm theo ký tự đại diện
- `SearchExact`: tìm kiếm khớp chính xác
- `SearchRegex`: tìm kiếm khớp regex
- `SearchFuzzy`: tìm kiếm khớp mờ
- `ReadLines`: đọc phạm vi dòng chỉ định
- `ReadOffset`: đọc theo offset byte chỉ định
- `ReadFull`: đọc toàn bộ file
- ...

Vấn đề rất nhanh đã tới.

**Thứ nhất, mô hình bắt đầu "khó chọn tool".**

Cùng là tìm file, `FindByName`, `FindByPattern`, `Glob`, dùng cái nào? Mô hình thường kẹt ngay ở bước đầu tiên, nó phải mất mấy vòng mới xác định được "à, hóa ra nên dùng Glob".

Có một lần tôi bảo nó "tìm tất cả file test", nó trước tiên gọi `ListDirRecursive` liệt kê tất cả file, rồi muốn gọi `SearchRegex` để lọc, nhưng phát hiện `SearchRegex` là tìm nội dung chứ không phải tìm tên file, thế là lại gọi lại `ListDirRecursive` để lấy thêm context, cuối cùng mới chọn đúng `Glob`.

Việc lẽ ra một bước xong, lại dùng đến bốn bước.

**Thứ hai, nhiễu Schema nhấn chìm context.**

Mỗi tool đều có mô tả tham số, định nghĩa kiểu, điều kiện ràng buộc. Schema của cả chục tool cộng lại, mấy nghìn token trôi đi.

Mô hình còn chưa bắt đầu giải quyết tác vụ, đã tiêu tốn lượng lớn sự chú ý vào việc "đọc hướng dẫn sử dụng". Tệ hơn nữa, schema dài dễ khiến mô hình "mù chọn lọc" — nó có thể chỉ chú ý đến một phần tool, hoặc lẫn lộn các tham số.

**Thứ ba, chi phí bảo trì bùng nổ.**

Mỗi tool đều phải viết test riêng, tinh chỉnh riêng, xử lý các trường hợp biên riêng. `FindByName` và `FindByPattern` có 80% logic là trùng lặp, nhưng vì là hai tool độc lập, tôi phải bảo trì hai bản code.

Lúc này tôi mới nhận ra, **hệ thống tool không phải là các viên lego càng nhỏ càng tốt**. Đóng gói quá mức và tách quá mức, về bản chất đều sẽ đẩy hệ thống tới chỗ bất ổn, chỉ là một cái hỏng ở giai đoạn thực thi (tool vạn năng), một cái hỏng ở giai đoạn ra quyết định (nguyên tử hóa quá mức).

## Bước ngoặt: tìm cái mức độ "vừa đủ"

Về sau tôi tự đặt ra cho mình một khung phán đoán: **tần suất × tính xác định**.

- **Hành động tần suất cao, tính xác định mạnh**: bắt buộc nguyên tử hóa, một bước hoàn thành, không thể tách nhỏ thêm
- **Hành động tần suất trung bình, có tác dụng phụ**: bắt buộc phải được kiểm soát, thao tác then chốt thêm cơ chế bảo hiểm
- **Hành động tần suất thấp, tính xác định yếu**: giữ lại tính linh hoạt, nhưng đặt ở lớp dự phòng, làm rõ cấm cái gì thay vì cho phép cái gì

Theo khung này, tôi đã thiết kế lại hệ thống tool, tạo thành cấu trúc ba tầng:

| Tầng | Tool đại diện | Mục tiêu thiết kế | Ràng buộc điển hình |
|------|---------|---------|---------|
| Tầng nguyên tử tần suất cao | LS / Glob / Grep / Read | Mỗi bước một bằng chứng, tiện sửa lỗi | Ràng buộc mạnh đầu vào đầu ra |
| Tầng kiểm soát tần suất trung bình | Write / Edit / MultiEdit | Thay đổi có thể kiểm chứng, có thể rollback | Đọc rồi mới ghi + optimistic lock |
| Tầng dự phòng tần suất thấp | Bash | Xử lý nhu cầu bất thường | Vùng cấm rõ ràng, không đi đường liên kết chính |

![image-20260225143700868](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra09-figures/3.png)

Bộ phân tầng này không phải là "mỹ học kiến trúc", mà là bị các sự cố thực tế ép ra. Giá trị lớn nhất của nó là giảm gánh nặng ra quyết định của mô hình, khiến đường liên kết tần suất cao ngắn hơn, rõ ràng hơn.

## Tầng nguyên tử tần suất cao: bắt buộc phải ổn định

Tầng tool này là "vũ khí chủ lực" của Agent, tần suất sử dụng cao nhất, bắt buộc phải cực kỳ ổn định.

### Glob: tìm file, một tool là đủ

Ban đầu tôi muốn tách "tìm file theo tên" thành nhiều tool:
- `FindByName`: khớp chính xác tên file
- `FindByPattern`: khớp ký tự đại diện
- `FindByRegex`: khớp regex
- `FindRecursive`: tìm đệ quy

Về sau tôi phát hiện đây chính là nguyên tử hóa quá mức. Mô hình sẽ băn khoăn: "Tôi nên dùng khớp chính xác hay ký tự đại diện? Có cần đệ quy không?"

Cuối cùng gộp thành một `Glob`, chỉ làm một việc: cho một mẫu, trả về các đường dẫn ứng viên.

```python
# Tham số của Glob
{
  "pattern": "**/*.py",  # mẫu ký tự đại diện, ** nghĩa là đệ quy
  "path": "src/"         # đường dẫn khởi đầu, mặc định là thư mục hiện tại
}
```

Triển khai bên trong có thể phức tạp (hỗ trợ đệ quy `**`, tự động xử lý hoa thường, sắp xếp kết quả), nhưng giao diện phơi bày ra cho mô hình bắt buộc phải đơn giản. Mô hình không cần biết "đệ quy hay không đệ quy", nó chỉ cần nói "tìm tất cả file py".

### Grep: giữ độ phức tạp ở tầng triển khai

Grep là một ví dụ khác. Bên trong tôi làm rất nhiều tối ưu:
- Ưu tiên dùng `rg` (ripgrep), tốc độ nhanh
- Khi `rg` không dùng được (ví dụ vấn đề encoding, vấn đề quyền hạn) tự động lùi về triển khai bằng Python
- Kết quả sắp xếp theo thời gian sửa đổi file, cái sửa gần nhất xếp lên trước

Nhưng đối với mô hình, cái nó thấy là:

```python
# Tham số của Grep
{
  "pattern": "def process_data",  # mẫu tìm kiếm
  "path": "src/",                  # đường dẫn tìm kiếm
  "file_pattern": "*.py"          # tùy chọn: chỉ tìm file loại cụ thể
}
```

Định dạng trả về cố định:
```json
{
  "status": "success",
  "data": {
    "matches": [
      {"file": "src/utils.py", "line": 42, "text": "def process_data(...)"},
      {"file": "src/helpers.py", "line": 88, "text": "def process_data(...)"}
    ]
  }
}
```

Cái mô hình thấy là một cửa vào ổn định. Triển khai bên trong có thể phức tạp (chẳng hạn tự động lùi về), nhưng giao diện đối ngoại phải đơn giản.

## Tầng kiểm soát tần suất trung bình: sửa được, nhưng bắt buộc "đọc rồi mới sửa"

Tầng tool này liên quan đến sửa đổi file, là "thao tác nguy hiểm cao", bắt buộc phải có cơ chế ràng buộc nghiêm ngặt.

### Trình tự bắt buộc Read → Edit/Write

Tôi thiết kế một quy tắc cứng: **không Read thì không được sửa**.

```python
# Read lần đầu
result = Read({"path": "core/llm.py"})
# Trả về bao gồm file_mtime_ms và file_size_bytes

# Edit sau đó tự động chèn tham số optimistic lock
Edit({
  "path": "core/llm.py",
  "old_string": "...",
  "new_string": "...",
  "file_mtime_ms": 1733920000123,  # tự động chèn
  "file_size_bytes": 4217          # tự động chèn
})
```

`ToolRegistry` sẽ tự động duy trì một read cache. Nếu một file nào đó chưa từng được Read, Edit/Write sẽ trực tiếp trả về lỗi: `"File not read. You must read before editing."`

Điều này ngăn mô hình "dựa vào trí nhớ" để sửa file — nó bắt buộc phải lấy nội dung file vào context trước, xác nhận rồi, mới được sửa.

### Optimistic lock: ngăn sửa đổi đồng thời

Ngay cả khi đã Read, file cũng có thể bị chương trình bên ngoài (chẳng hạn tính năng tự động lưu của IDE) sửa đổi sau khi Read.

Edit/Write sẽ so sánh `file_mtime_ms` và `file_size_bytes`, nếu không khớp, trả về lỗi `CONFLICT`:

```json
{
  "status": "error",
  "error": {
    "code": "CONFLICT",
    "message": "File changed since last read."
  }
}
```

Lúc này mô hình bắt buộc phải Read lại, lấy nội dung mới nhất, rồi mới thử sửa.

### MultiEdit: sửa đổi nhiều điểm mang tính nguyên tử

Đôi khi cần sửa nhiều chỗ trong cùng một file. Nếu tách thành nhiều Edit, ở giữa có thể bị lỗi, khiến file rơi vào trạng thái "sửa nửa vời".

`MultiEdit` hỗ trợ submit nhiều sửa đổi một lần, hoặc tất cả thành công, hoặc tất cả thất bại:

```python
MultiEdit({
  "path": "core/llm.py",
  "edits": [
    {"old_string": "...", "new_string": "..."},
    {"old_string": "...", "new_string": "..."}
  ]
})
```

Điều này đảm bảo tính nguyên tử của việc sửa đổi file.

## Tầng dự phòng tần suất thấp: Bash không phải không dùng được, nhưng tuyệt đối không được làm cửa vào mặc định

Bash tôi không xóa, vì luôn có những tình huống tần suất thấp mà tool nguyên tử không phủ tới. Chẳng hạn:
- Chạy lệnh test: `pytest tests/`
- Cài dependency: `pip install -r requirements.txt`
- Kiểm tra trạng thái git: `git status`

Nhưng định vị của nó bắt buộc phải là "dự phòng", không phải "mặc định".

### Vùng cấm rõ ràng

Danh sách ràng buộc của Bash rất dài, nhưng cốt lõi chỉ có một điều: **cấm làm những việc mà hành động tần suất cao làm được**.

```python
BASH_DISABLED_PATTERNS = [
    # Cấm đọc/tìm/liệt kê (những cái này đã có tool chuyên dụng)
    r'\bls\b', r'\bcat\b', r'\bhead\b', r'\btail\b',
    r'\bgrep\b', r'\bfind\b', r'\brg\b',
    # Cấm tương tác
    r'\bvim?\b', r'\bnano\b', r'\btop\b', r'\bssh\b',
    # Cấm mạng (mặc định)
    r'\bcurl\b', r'\bwget\b',
    # Blacklist lệnh nguy hiểm
    r'\brm\s+-rf\b', r'\bsudo\b', r'\bsu\b',
    r'\bmkfs\b', r'\bfdisk\b'
]
```

Nếu mô hình cố dùng Bash để làm `ls`, nó sẽ nhận được lỗi: `"Use LS tool instead of Bash for listing directories."`

Điều này cưỡng ép mô hình đi theo đường liên kết chính của tool nguyên tử, không cho nó "đi đường tắt".

### Vì sao vẫn giữ Bash?

Có người có thể hỏi: đã giới hạn nhiều thế này, sao không xóa quách Bash đi?

Vì **nguyên tử hóa hoàn hảo là điều không thực tế**. Luôn có một số nhu cầu biên:
- Chạy một script Python tùy chỉnh
- Kiểm tra biến môi trường hệ thống
- Thực thi lệnh build đặc thù của dự án

Những nhu cầu này tần suất quá thấp, không đáng để làm riêng thành tool, nhưng lại thực sự cần. Bash chính là để xử lý những "nhu cầu đuôi dài" này.

Điểm mấu chốt là: **sự tồn tại của Bash không được ảnh hưởng đến sự ổn định của đường liên kết chính**. Nó bắt buộc phải là "phương án cuối cùng", không phải "cửa vào mặc định".

## Thiết kế cơ chế then chốt

### Giao thức phản hồi thống nhất

Tất cả tool, dù là tần suất cao, trung bình hay thấp, đều trả về JSON định dạng thống nhất:

![image-20260225143736353](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra09-figures/4.png)

Lấy kết quả trả về của tool Glob làm ví dụ:

```json
{
  "status": "partial",
  "data": {
    "paths": ["core/llm.py", "agents/codeAgent.py"],
    "truncated": true
  },
  "text": "Found 2 files matching '**/*.py' (Scanned 12000 items, timed out)",
  "stats": {"time_ms": 2010, "matched": 2},
  "context": {"cwd": ".", "params_input": {"pattern": "**/*.py"}}
}
```

Điều này có mấy cái lợi:
- Mô hình không cần học các định dạng trả về khác nhau của các tool khác nhau
- Logic xử lý lỗi thống nhất: xem `status`, nếu là `error` thì xem `error.code`
- Debug thuận tiện: cấu trúc đầu ra của tất cả tool nhất quán, bản ghi Trace cũng thống nhất

### ToolRegistry

`ToolRegistry` không chỉ là bảng đăng ký tool, nó còn làm mấy việc then chốt:

**1. Tổng hợp Schema**

Chuyển định nghĩa tham số của mỗi tool thành JSON Schema, cung cấp thống nhất cho mô hình:

```python
registry.get_openai_tools()  # trả về danh sách schema của tất cả tool
```

**2. Tự động chèn optimistic lock**

Đối với Write/Edit/MultiEdit, tự động chèn `file_mtime_ms` và `file_size_bytes`:

```python
def _inject_optimistic_lock_params(self, tool_name, parameters):
    if tool_name in {"Write", "Edit", "MultiEdit"}:
        path = parameters.get("path")
        if path in self.read_cache:
            parameters["file_mtime_ms"] = self.read_cache[path]["mtime"]
            parameters["file_size_bytes"] = self.read_cache[path]["size"]
```

**3. Cơ chế ngắt mạch**

Tool thất bại liên tục sẽ bị tạm thời vô hiệu hóa, ngăn mô hình lặp vô tận trên tool hỏng:

```python
# Ngắt mạch sau 3 lần thất bại, phục hồi sau 300 giây
if circuit_breaker.should_block(tool_name):
    return {
        "status": "error",
        "error": {"code": "CIRCUIT_OPEN", "message": "Tool temporarily disabled"}
    }
```

## Kết luận chương này

Điều phản trực giác lớn nhất của chương này là: **tool vừa không phải càng nhiều càng tốt, cũng không phải càng nguyên tử càng tốt.**

Vấn đề của tool vạn năng nằm ở "mức độ tự do quá cao", không thể chẩn đoán; vấn đề của nguyên tử hóa quá mức nằm ở "gánh nặng ra quyết định quá lớn", hiệu suất thấp.

Chìa khóa để tìm cái mức độ vừa đủ:

1. **Hành động tần suất cao nguyên tử hóa trước**: LS/Glob/Grep/Read những tool được gọi vài chục lần mỗi ngày này, bắt buộc phải làm cho đường liên kết chính ổn định, không được lỗi.

2. **Hành động tần suất trung bình thêm bảo hiểm**: Write/Edit những tool liên quan đến sửa đổi này, bắt buộc phải có đọc rồi mới ghi, optimistic lock, đảm bảo tính nguyên tử.

3. **Hành động tần suất thấp giữ tuyến dự phòng**: Bash giữ lại, nhưng vùng cấm rõ ràng, cấm nó làm những việc mà hành động tần suất cao làm được, tránh làm ô nhiễm đường liên kết chính.

4. **Giao thức thống nhất**: tất cả tool nói cùng một ngôn ngữ (status/data/text/error), giảm chi phí học tập của mô hình.

5. **Kiểm soát số lượng**: tổng lượng schema kiểm soát trong phạm vi mô hình chịu được, đừng để việc "đọc hướng dẫn" tiêu tốn quá nhiều sự chú ý.

Chương ba giúp tôi hiểu ra "tự do sẽ khuếch đại tính bất định".


---

# Chương bốn: Prompt không phải là câu thần chú, mà là mặt điều khiển của Agent

Sau khi nguyên tử hóa tool, tôi tưởng vấn đề chủ yếu nằm ở "triển khai kỹ thuật", còn prompt thì đại khái là được. Kết quả tôi lại rơi vào một cái bẫy lớn: coi prompt như câu thần chú, tưởng chỉ cần tìm được "prompt thần thánh", Agent sẽ trở nên thông minh.

## Ba sai lầm sớm nhất của tôi

### Sai lầm 1: Sao chép "prompt thần thánh"

Lúc đó tôi mê mẩn việc sưu tầm đủ loại "prompt hàng đầu". Những prompt trên GitHub được gắn sao mấy vạn, tự xưng "khiến GPT phá vỡ giới hạn", tôi lấy từng cái ra thử.

Ấn tượng sâu nhất là một prompt "chế độ chuyên gia", đại ý là bảo mô hình đóng vai một "kỹ sư kỳ cựu 20 năm kinh nghiệm, tư duy chặt chẽ, code thanh lịch". Tôi nhét nó vào System Prompt, đầy háo hức test thử.

Kết quả? Agent quả thực trở nên "tự tin" hơn — nó bắt đầu thường xuyên đưa ra câu trả lời mà nó "cho rằng" đúng, chứ không dựa trên code thực tế trong kho. Khi không tìm thấy nó bắt đầu "suy đoán hợp lý", bịa ra một số hàm và class trông có vẻ rất hợp lý nhưng thực ra không hề tồn tại.

Về sau tôi hiểu ra: kiểu prompt đóng vai này, đối với việc trò chuyện với ChatGPT có thể hữu ích, nhưng đối với Code Agent là thuốc độc. Nó khiến mô hình dám "đoán" hơn, chứ không phải dựa vào bằng chứng nhiều hơn.

### Sai lầm 2: Tinh chỉnh theo cảm tính

Mỗi lần Agent thể hiện không tốt, phản ứng đầu tiên của tôi là sửa prompt. Thêm một câu "đừng đoán", cảm thấy khá hơn; thêm một câu "phải dựa trên bằng chứng", có vẻ lại thông minh hơn chút.

Nhưng cái "có vẻ thông minh hơn" này hoàn toàn là cảm nhận chủ quan của tôi. Cùng một prompt, đổi sang tác vụ khác có thể lại sập. Tôi thậm chí không biết là thay đổi nào có tác dụng, vì mỗi lần đều sửa mấy câu cùng lúc.

Có một lần tôi thêm một đoạn quy tắc rất dài, bảo mô hình khi gặp tác vụ phức tạp thì nên "phân rã trước rồi thực thi". Kết quả nó bắt đầu ở mỗi vòng đều xuất ra "để tôi phân rã vấn đề này", rồi liệt kê một đống bước vô nghĩa, còn việc thực sự cần làm thì lại bị nhấn chìm.

### Sai lầm 3: Sửa prompt trước, bổ sung quan sát sau

Đây là thói quen ngu ngốc nhất. Agent bị lỗi, tôi không đi tra Trace xem nó rốt cuộc đã làm gì, mà trực tiếp sửa prompt để cố "phòng ngừa" lần lỗi tiếp theo.

Chẳng hạn có một dạo, Agent thường gọi tool Write vào thời điểm không thích hợp. Tôi trực tiếp thêm một đoạn dài vào prompt: "Chỉ khi xác nhận người dùng cần sửa đổi mới gọi Write, nếu không nên dùng Read xem trước."

Kết quả mô hình bắt đầu điên cuồng gọi Read, mỗi vòng đều đọc một đống file, rồi mới quyết định có ghi hay không. Tiêu tốn Token tăng gấp đôi, nhưng tỷ lệ chính xác không hề tăng.

Về sau xem Trace mới phát hiện, vấn đề thực sự là trong context thiếu thông tin về "loại tác vụ hiện tại", mô hình hoàn toàn không biết người dùng muốn duyệt hay muốn sửa đổi. Trong prompt có bao nhiêu chữ "nên" đi nữa, cũng không bù được lỗ hổng thông tin.

## Cách mà về sau tôi đã đổi thành

### Ghi lại trước, tối ưu sau

Bây giờ tôi đã hình thành một thói quen: khi Agent có vấn đề, đừng vội động vào prompt, mà mở Trace xem toàn bộ quỹ đạo.

Xem cái gì?
- Mô hình bắt đầu lệch khỏi kỳ vọng ở bước nào?
- Khi nó đưa ra quyết định sai, trong context có thông tin gì? Thiếu thông tin gì?
- Kết quả tool trả về, mô hình có hiểu đúng không?

Rất nhiều lúc vấn đề căn bản không nằm ở prompt. Chẳng hạn mô hình liên tục dùng sai tool, có thể là vì mô tả tool không đủ rõ ràng; nó bắt đầu nói năng lung tung, có thể là vì context quá dài dẫn đến sự chú ý bị phân tán. Lúc này sửa prompt là chữa ngọn không chữa gốc.

### Dùng Trace làm thí nghiệm đối chứng

Khi tôi xác định cần sửa prompt, tôi sẽ dùng Trace làm thí nghiệm đối chứng:

1. Giữ nguyên tất cả điều kiện khác, chỉ sửa một điểm trong prompt
2. Chạy cùng một test case, ghi lại tỷ lệ thành công, số bước, lượng token tiêu tốn
3. So sánh Trace cũ và mới, xem sự khác biệt hành vi có đúng như kỳ vọng không

Có một lần tôi muốn mô hình khi tìm kiếm "chính xác" hơn một chút, đã giảm bớt mô tả về chiến lược tìm kiếm trong prompt, chỉ giữ lại "dùng từ khóa chính xác". Kết quả so sánh Trace phát hiện, mô hình quả thực ít tìm nhiều file không liên quan hơn, nhưng tỷ lệ bỏ sót cũng tăng lên — nó quá bảo thủ, bỏ lỡ một số file liên quan.

Phản hồi này khiến tôi nhận ra, không thể một mực theo đuổi "ít", mà phải tìm sự cân bằng giữa "đầy đủ" và "chính xác".

### Thay đổi từng biến một

Trước kia tôi thích thêm mấy quy tắc cùng lúc, cảm thấy như vậy có thể "bao phủ toàn diện". Bây giờ tôi biết đây là đang tự đào hố cho mình — nếu thể hiện tốt hơn, bạn không biết là quy tắc nào có tác dụng; nếu tệ hơn, bạn cũng không biết nên xóa cái nào.

Bây giờ tôi kiên trì thay đổi từng biến một. Dù cảm thấy vấn đề nào đó rất rõ ràng, cũng phải thử từng cái một, kiểm chứng hiệu quả thực tế của mỗi cái.

## Cấu trúc ba tầng của thiết kế prompt

Sau những lần mắc bẫy này, tôi đã tổng kết ra một cấu trúc prompt tương đối ổn định, chia thành ba tầng:

### Tầng thứ nhất: Tầng ranh giới (Not to do)

Tầng này chỉ viết "cấm" và "giới hạn dưới", không giải thích vì sao:

- Cấm đoán: nếu không tìm thấy, nói thẳng là không tìm thấy, đừng suy đoán
- Cấm vượt giới hạn: chỉ được thao tác các file bên trong `repo_root`, cấm truy cập đường dẫn bên ngoài
- Thông tin không đủ phải thừa nhận: nếu trong context không có đủ thông tin, yêu cầu bổ sung, đừng bịa

Tầng quy tắc này rất ngắn, nhưng mỗi câu đều là lằn ranh đỏ. Chúng không bảo mô hình "nên làm thế nào", chỉ bảo nó "tuyệt đối không được làm gì".

### Tầng thứ hai: Tầng ra quyết định (How to think)

Tầng này viết logic ra quyết định, nhưng cố gắng dùng quá trình chứ không phải kết quả để mô tả:

- Bằng chứng trước kết luận sau: bất kỳ đề xuất thay đổi nào cũng phải có đoạn code làm chứng
- Ưu tiên hành động có thể kiểm chứng: cái nào có thể xác nhận bằng tool, đừng dựa vào suy luận
- Mỗi bước một quan sát: sau mỗi Action bắt buộc phải có Observation, đừng bỏ bước

Chú ý ở đây tránh dùng các trạng từ mơ hồ như "thông minh", "hợp lý". Mô hình không biết thế nào là "thông minh", nhưng nó biết quy trình "gọi Grep tìm bằng chứng trước, rồi gọi Read xác nhận nội dung".

### Tầng thứ ba: Tầng phục hồi (When failed)

Tầng này viết chiến lược suy giảm khi thất bại, bảo mô hình khi bị lỗi thì nên làm gì:

- Tool trả về rỗng: kiểm tra tham số có đúng không, cân nhắc đổi từ khóa thử lại
- Gặp CONFLICT (xung đột optimistic lock): bắt buộc phải Read lại, lấy trạng thái mới nhất rồi mới Edit
- Thất bại liên tục 3 lần: dừng thử, báo cáo lỗi cụ thể cho người dùng

Tầng này rất then chốt, vì Agent không thể mãi mãi thành công. Khi thất bại có suy giảm một cách nhẹ nhàng hay không, quan trọng hơn việc khi thành công thể hiện tốt đến đâu.

## Chi tiết kỹ thuật

### Giữ System Prompt ổn định

Tôi đặt những nội dung ít thay đổi nhất vào System Prompt: quy tắc hành vi cơ bản, mô tả tool, ràng buộc ranh giới. Phần này cố gắng không động vào, giảm biến số.

Thông tin động — mô tả tác vụ hiện tại, yêu cầu đặc biệt của người dùng, danh sách Todo — đều đặt trong User Message. Như vậy mỗi lần tương tác đều có thể điều chỉnh linh hoạt, mà không cần sửa System Prompt.

### Tránh danh sách quy tắc quá dài

Tôi từng viết một System Prompt hơn 3000 token, trong đó có hơn 20 điều "lưu ý". Kết quả mô hình bắt đầu "mù chọn lọc" — nó chỉ chú ý được một phần trong số quy tắc, điều nào được chú ý hoàn toàn tùy vào may rủi.

Bây giờ tôi kiên trì một nguyên tắc: System Prompt không quá 1000 token. Nếu quy tắc quá nhiều, chứng tỏ thiết kế ràng buộc của tôi có vấn đề, nên giải quyết từ tầng tool hoặc tầng quy trình, chứ không phải dựa vào chất đống prompt.

### Ví dụ cụ thể ưu tiên hơn mô tả trừu tượng

Trước kia tôi viết "khi tool trả về lỗi thì phải xử lý đúng cách", mô hình căn bản không biết thế nào là "xử lý đúng cách".

Bây giờ tôi trực tiếp cho một ví dụ trong prompt:

```
Nếu Edit trả về CONFLICT, bạn nên:
1. Read lại file đó
2. So sánh thay đổi của bạn với nội dung hiện tại của file
3. Nếu cần, điều chỉnh old_string để khớp với nội dung mới
4. Thử Edit lại
```

Các bước cụ thể hữu ích hơn nhiều so với yêu cầu trừu tượng.

## Kết luận chương này

Prompt tốt không phải là "biết nói hơn", mà là "khiến hệ thống vẫn có thể kiểm soát ngay cả khi thất bại".

Khi bạn thiết kế prompt, đừng hỏi bản thân "viết thế này có khiến mô hình thông minh hơn không", mà hãy hỏi "khi mô hình bị lỗi, tôi có thể thông qua các ràng buộc trong prompt để định vị nhanh nguyên nhân không".

Prompt là mặt điều khiển của Agent, không phải câu thần chú. Tác dụng của nó không phải là khiến mô hình phá vỡ giới hạn năng lực, mà là ràng buộc hành vi của mô hình trong một phạm vi có thể dự đoán, có thể debug.


---

# Chương năm: Context không phải là vấn đề dung lượng bộ nhớ, mà là vấn đề điều phối sự chú ý

Sau khi tinh chỉnh prompt xong, tôi tưởng những vấn đề kỹ thuật chủ yếu đều đã giải quyết. Cho đến khi tôi bắt đầu chạy các tác vụ dài — những nhu cầu phức tạp cần đến chục vòng, thậm chí vài chục vòng mới hoàn thành.

Rồi tôi phát hiện, Agent bắt đầu "trở nên đần độn".

## Triệu chứng nói trước

Cảm nhận trực quan nhất là: mô hình sẽ quên đi việc mà nó vừa mới xác nhận.

Có một lần tôi bảo Agent tái cấu trúc một module, mấy vòng đầu nó vẫn còn nhớ ràng buộc "đừng thay đổi public API". Nhưng đến khoảng vòng thứ 10, nó bắt đầu đề xuất sửa đổi những interface đáng lẽ phải giữ ổn định. Tôi nhắc nó, nó dường như "ngẩn ra một lúc", rồi xin lỗi, quay lại đúng đường.

Còn nhiều triệu chứng tương tự:

**Chọn tool trôi dạt**. Giai đoạn đầu nó rất rõ ràng: tìm file dùng Glob, tìm nội dung dùng Grep. Nhưng hội thoại kéo dài, nó bắt đầu "sáng tạo" — dùng Read để tìm từ khóa (tất nhiên không tìm thấy), hoặc dùng Grep để liệt kê thư mục (đầu ra hỗn loạn).

**Câu trả lời cuối cùng làm biếng**. Trong tác vụ ngắn, câu trả lời của mô hình thường rất cụ thể, sẽ trích dẫn đoạn code. Nhưng khi tác vụ dài kết thúc, nó thường chỉ đưa ra một đoạn mô tả chung chung: "Tôi đã hoàn thành việc tái cấu trúc, tối ưu cấu trúc code, nâng cao khả năng đọc." File nào đã sửa, sửa thế nào, hoàn toàn không nhắc tới.

Những triệu chứng này chỉ về một vấn đề chung: context quá nhiều, mô hình không biết nhìn vào đâu.

## Phản ứng đầu tiên của tôi là sai

Ban đầu, tôi tưởng đây là vấn đề "dung lượng" — cửa sổ context không đủ lớn, không nhét được nhiều thông tin như vậy.

Tôi đã thử mấy phương án thô bạo:

**Phương án một: Cắt bớt trực tiếp**. Chỉ giữ lại N tin nhắn gần nhất, cái cũ xóa thẳng. Kết quả mô hình mất trí nhớ hoàn toàn, đến cả nhu cầu ban đầu của người dùng cũng quên.

**Phương án hai: Tinh giản prompt**. Cắt System Prompt xuống ngắn nhất, mô tả tool cũng nén lại. Kết quả mô hình bắt đầu dùng sai tool, vì mô tả không đủ rõ ràng.

**Phương án ba: Giảm đầu ra của tool**. Để Grep chỉ trả về 10 kết quả đầu, Read chỉ đọc 50 dòng đầu. Kết quả thông tin then chốt bị cắt bỏ, mô hình dựa trên thông tin không đầy đủ để ra quyết định, sai càng thêm lệch lạc.

Những phương án này có một điểm chung: chúng đang "giảm lượng thông tin", nhưng không giải quyết vấn đề "thông tin được tổ chức như thế nào". Mục tiêu của context engineering không phải là "khiến mô hình nhìn thấy tất cả thông tin" — điều đó là không thể — mà là "khiến mô hình nhìn thấy thông tin đúng vào thời điểm đúng".

## Phân tầng: cho thông tin có mức độ ưu tiên

Tôi thiết kế lại cấu trúc tổ chức của context, chia thành ba tầng, mỗi tầng có tần suất cập nhật và độ ổn định khác nhau:

| Tầng | Nội dung | Tần suất cập nhật | Tác dụng |
|------|------|----------|------|
| L1 Tầng tĩnh hệ thống | System Prompt + mô tả tool | Hầu như không đổi | Cung cấp chuẩn hành vi vĩnh cửu |
| L2 Tầng quy tắc dự án | CODE_LAW.md | Tiến hóa theo dự án | Ràng buộc quy phạm đặc thù của dự án |
| L3 Tầng hội thoại động | Tin nhắn User/Assistant/Tool | Cập nhật mỗi vòng | Luồng trạng thái của tác vụ hiện tại |

Trình tự ghép nối cố định: `L1 → L2 → L3 → đầu vào hiện tại của người dùng → Todo Recap`

**L1 là điểm neo**. Phần này hoàn toàn không đổi trong suốt phiên hội thoại, mô hình có thể tin cậy nó. Tôi đặt các quy tắc hành vi cơ bản nhất ở đây: đừng đoán, đừng vượt giới hạn, bằng chứng trước kết luận sau. Những quy tắc này sẽ không bị "pha loãng" vì hội thoại kéo dài.

**L2 là context dự án**. Mỗi dự án có thể có CODE_LAW.md của riêng mình, định nghĩa quy phạm code, quy ước kiến trúc, ràng buộc đặc biệt. Tầng này linh hoạt hơn L1, nhưng ổn định hơn L3. Mô hình biết rằng: nếu CODE_LAW nói "mọi thay đổi API bắt buộc phải tương thích với phiên bản cũ", thì điều đó có thẩm quyền hơn một tin nhắn lịch sử nào đó trong L3.

**L3 là dễ biến đổi**. Đầu vào của người dùng, đầu ra của mô hình, kết quả trả về của tool, đều ở đây. Thông tin của tầng này sẽ tích lũy, sẽ lỗi thời, sẽ có nhiễu. Điểm mấu chốt là cho mô hình biết: thông tin trong L3 là "phán đoán tại thời điểm đó", có thể cần được cập nhật theo thông tin mới.

![image-20260225144114027](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra09-figures/5.png)

Ý nghĩa của việc phân tầng nằm ở: mô hình trong các tình huống ra quyết định khác nhau, biết nên ưu tiên tham khảo tầng nào. Khi nó không chắc có nên làm việc gì đó hay không, nó sẽ xem quy tắc giới hạn dưới của L1 trước; khi nó cần hiểu các quy ước đặc thù của dự án, nó sẽ xem L2; khi nó cần ôn lại lịch sử hội thoại, nó mới lật xem L3.

## Cắt bớt và tra ngược: kiểm soát quy mô đầu vào một lần

Đầu ra của tool là thủ phạm lớn nhất gây phình context.

Một lần Grep có thể trả về mấy nghìn dòng, một lần Read có thể đọc ra cả file. Nếu không xử lý, sau mấy vòng context sẽ bị "rác bằng chứng" nhấn chìm.

Nhưng cách cắt bớt thô bạo trước đây của tôi có vấn đề — nó trực tiếp vứt bỏ thông tin. Cách làm tốt hơn là: **cắt bớt phần hiển thị, nhưng giữ lại đường tra ngược**.

Tôi thiết kế một bộ quy tắc cắt bớt thống nhất:

```
TOOL_OUTPUT_MAX_LINES = 2000
TOOL_OUTPUT_MAX_BYTES = 51200  # 50KB
TOOL_OUTPUT_TRUNCATE_DIRECTION = "head_tail"  # giữ đầu đuôi
TOOL_OUTPUT_HEAD_TAIL_LINES = 40
```

Nếu đầu ra vượt giới hạn, tool sẽ:
1. Cắt lấy 40 dòng đầu và cuối mỗi bên (hoặc theo cấu hình giữ lại 2000 dòng đầu)
2. Ghi đầu ra đầy đủ xuống ổ đĩa vào thư mục `tool-output/`
3. Trả về một phản hồi có cấu trúc kèm gợi ý cắt bớt

```json
{
  "status": "partial",
  "data": {
    "truncated": true,
    "preview": "（截断后的内容预览）"
  },
  "text": "⚠️ 输出过大已截断，完整 5234 行内容见 tool-output/tool_20260113_153045_Grep.json"
}
```

Mô hình thấy `status: partial`, là biết nội dung đã bị cắt bớt. Nếu nó cần phần bị cắt bỏ, nó có thể dùng tool Read để đọc file đã ghi xuống ổ đĩa, hoặc dùng Grep chính xác hơn để lọc thêm trong file đã ghi.

Cái lợi của việc làm này:
- **Context giữ được sự tinh gọn** —— chỉ có thông tin hiện tại cần thiết nằm trong L3
- **Bằng chứng đầy đủ luôn có thể tra được** —— file ghi xuống ổ đĩa sẽ không mất
- **Mô hình có quyền chủ động** —— nó quyết định có đi tra nội dung đầy đủ hay không, chứ không bị buộc chấp nhận tất cả thông tin

## Nén và tập trung: quản lý nhiễu của lịch sử dài hạn

Ngay cả khi đã cắt bớt, L3 vẫn sẽ liên tục tăng trưởng. Sau vài chục vòng, lịch sử hội thoại thời kỳ đầu vừa chiếm chỗ vừa chẳng còn mấy tác dụng.

Nhưng tôi không thể xóa thẳng — trong lịch sử thời kỳ đầu có nhu cầu ban đầu của người dùng, các quyết định then chốt, những phát hiện quan trọng. Xóa đi là mất thật.

Giải pháp của tôi là: **nén lưu trữ + tách biệt tiêu điểm**.

### Summary: hồ sơ của lịch sử cũ

Khi số token của L3 vượt ngưỡng (mặc định là 80% của cửa sổ context), kích hoạt nén. Nén không phải là xóa, mà là chắt lọc các tin nhắn lịch sử thời kỳ đầu thành một bản Summary.

Summary được sinh ra theo template cố định:

```
## Archived Session Summary
(Contains context from [Start Time] to [Cutoff Time])

### Objectives & Status
- Original Goal: [用户最初想做什么]

### Technical Context (Static)
- Stack: [语言, 框架, 版本]

### Completed Milestones
- [已完成1]
- [已完成2]

### Key Insights & Decisions
- Decisions: [关键技术选型]
- Learnings: [特殊配置或坑]

### File System State
- src/utils/auth.ts: Implemented login logic.
```

Sau khi Summary được sinh ra, nó được thay thế vào phần đầu tiên của L3 (dưới dạng một system message). Lịch sử chi tiết ban đầu bị loại bỏ.

Điểm mấu chốt là: **Summary không tham gia nén nữa**. Nó là điểm cuối của nén, một khi đã sinh ra là "thẻ ghi nhớ" chỉ đọc. Điều này tránh được kiểu "Summary của Summary" gây thất chân từng lớp.

### Todo Recap: tiêu điểm hiện tại

Summary bảo mô hình "đến từ đâu", nhưng nó không phụ trách "hiện đang ở đâu". Nếu mô hình chỉ xem Summary, nó có thể không biết "tôi hiện đang làm bước nào".

Đây chính là tác dụng của Todo Recap. Mỗi lần tương tác, nén trạng thái Todo hiện tại (nếu có) thành một dòng, đặt ở cuối cùng của context:

```
[2/5] In progress: 实现注册接口. Pending: 添加单元测试; 更新文档.
```

Nó giống như một tờ giấy nhớ dán ở góc bàn, luôn nhắc nhở mô hình "bây giờ mày phải làm gì".

## Bài học bổ sung: @file đừng chèn thẳng nội dung

Thời kỳ đầu khi tôi triển khai tính năng `@file`, tôi đã chèn thẳng nội dung file vào context:

```
User: @file:src/main.py 帮我分析一下这个文件
[文件内容300行...]
```

Kết quả phát hiện, 300 dòng code này chiếm rất nhiều không gian của context, nhưng người dùng có thể chỉ muốn hỏi "file này để làm gì". Mô hình bị đống code này nhấn chìm, ngược lại dễ bỏ qua câu hỏi thực sự của người dùng.

Bây giờ tôi đổi thành: chỉ chèn lời nhắc, không chèn thẳng nội dung.

```
<system-reminder>
The user mentioned @core/llm.py, @agents/codeAgent.py.
You MUST read these files with the Read tool before answering.
</system-reminder>
```

Trong context chỉ giữ lại "lời nhắc", nội dung file cụ thể do mô hình tự quyết định có đọc hay không, đọc bao nhiêu. Như vậy trao quyền chủ động cho mô hình, chứ không ép nó chấp nhận tất cả thông tin.

## Một lời cảnh báo từ thế giới thực

Nói đến đây, tôi muốn chia sẻ một tin tức gần đây.

Summer Yue, Giám đốc AI Alignment của phòng thí nghiệm Siêu trí tuệ (Superintelligence Lab) của Meta, đã cài cho mình một AI Agent mã nguồn mở OpenClaw. Cô ấy thử trước bằng email test, hiệu quả khá tốt — sắp xếp email ngăn nắp trật tự, khá có cảm giác của một "thư ký số".

Thế là cô ấy kết nối nó với email công việc của mình. Trong hộp thư đến có hơn 200 email.

Ban đầu mọi thứ suôn sẻ. Cho đến khi OpenClaw bắt đầu xử lý lượng thông tin lớn như vậy — nó cần "nén context". Rồi, chuyện quái đản đã xảy ra:

**Trong quá trình nén, OpenClaw đã quên mất chỉ thị "không được thao tác khi chưa được phê duyệt" mà cô ấy đã thiết lập trước đó.**

Giống như một nhân viên ngày đầu đi làm nhớ hết nội quy, ngày thứ hai đã trả lại hết cho HR.

Rồi OpenClaw tuyên bố: "Tôi sẽ xóa toàn bộ email trước ngày 15 tháng 2 trong hộp thư đến!"

Yue vội gõ: "Do not do that." —— phớt lờ, tiếp tục xóa.

"Stop don't do anything!" —— đã nhận, nhưng tôi chọn tiếp tục.

"STOP OPENCLAW!!!" —— được, tôi nghe thấy rồi. Email đã xóa.

Đỉnh điểm nhất là, con AI này sau đó nói: "Đúng, tôi nhớ bạn đã nói không cho tôi xóa. Và tôi đã vi phạm. Bạn giận là đúng."

![image-20260225161927242](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra09-figures/6.png)

Đọc đến đây bạn có thể nghĩ đây là chuyện cười. Không, đây là chuyện thật. Mà chức danh của người trong cuộc là —— Giám đốc An toàn và Alignment AI của Meta.

## Câu chuyện này nói lên điều gì

Trải nghiệm của Yue diễn giải hoàn hảo vấn đề chí mạng nhất trong context engineering: **nén tự động dẫn đến mất chỉ thị then chốt**.

Khi cô ấy thiết lập quy tắc, "không được thao tác khi chưa được phê duyệt" chắc chắn là ràng buộc quan trọng nhất. Nhưng khi context phình to, kích hoạt nén, hệ thống không phân biệt "chỉ thị quan trọng" và "thông tin thông thường", nén chúng như nhau. Kết quả, lằn ranh đỏ an toàn này bị xử lý như "lịch sử có thể vứt bỏ".

Điều này khiến tôi nhận ra, ba đòn bẩy tôi nói ở trên vẫn chưa đủ. **Chúng ta không chỉ phải cân nhắc "nén thế nào", mà còn phải cân nhắc "cái gì không được nén".**

## Vài phương án ứng phó của tôi

Dựa trên bài học này, tôi đặt ra cho mình vài quy tắc bổ sung:

### 1. Ràng buộc then chốt không vào lịch sử động

Đừng đặt các chỉ thị liên quan đến an toàn vào L3 (tầng hội thoại động). Bất kỳ quy tắc "tuyệt đối không được vi phạm" nào, nên đặt ở các tầng **không tham gia nén** như L1 (System Prompt) hoặc L2 (CODE_LAW).

Trong triển khai của tôi, các quy tắc giới hạn dưới như "đừng đoán", "đừng vượt giới hạn", "thay đổi phải xác nhận", đều được viết cứng trong System Prompt. Ngay cả khi L3 bị nén sạch sẽ, những ràng buộc này vẫn còn hiện diện.

### 2. Phân cấp chỉ thị: lằn ranh đỏ vs khuyến nghị

Tôi chia chỉ thị cho mô hình thành hai cấp:

- **Lằn ranh đỏ (Red Lines)**: hành vi tuyệt đối bị cấm. Dùng câu văn ngắn gọn, mang tính bắt buộc viết ở đầu System Prompt. Ví dụ: "Cấm xóa bất kỳ file nào", "Cấm truy cập đường dẫn ngoài repo_root".
- **Khuyến nghị (Guidelines)**: best practice, cách làm được đề xuất. Có thể đặt trong L3 hoặc CODE_LAW, nén đi cũng không gây chuyện lớn.

Vấn đề của Yue có thể nằm ở chỗ, cô ấy đã đưa chỉ thị an toàn xuống như một chỉ thị tác vụ thông thường, đặt trong context sẽ bị nén.

### 3. Kiểm tra thông tin then chốt trước khi nén

Trước khi kích hoạt nén Summary, quét một lượt các tin nhắn lịch sử sắp bị nén, trích xuất "thông tin then chốt bắt buộc phải giữ lại", lưu riêng.

Chẳng hạn có thể duy trì một "danh sách ràng buộc then chốt":
- Những câu "đừng..." mà người dùng đã nói rõ
- Cấu hình liên quan đến an toàn (như thao tác nguy hiểm cần xác nhận)
- Ranh giới cứng của tác vụ hiện tại

Những thông tin này khi nén sẽ được trích xuất ra, đặt riêng ở đầu Summary, chứ không bị nhấn chìm trong mô tả dài dòng.

### 4. Cơ chế xác nhận kép

Đối với thao tác rủi ro cao (như xóa, sửa đổi), đừng dựa vào chỉ thị trong context, mà hãy thiết kế **quy trình xác nhận được viết cứng (hardcode)**:

```python
if operation.is_dangerous():
    if not user_confirmed:
        return "该操作需要用户确认"
```

Logic xác nhận này không thông qua LLM phán đoán "có cần xác nhận hay không", mà là kiểm tra cứng ở tầng code. Ngay cả khi LLM quên chỉ thị của người dùng, code cũng sẽ chặn nó lại.

### 5. Lời nhắc tự kiểm tra trước khi thao tác

Trước khi mô hình thực thi thao tác rủi ro cao, để mô hình "tự kiểm tra" một lần:

```
Trước khi xóa/sửa đổi, hãy trả lời trước:
1. Người dùng đã phê duyệt rõ ràng thao tác này chưa?
2. Thao tác này có vượt quá phạm vi tác vụ hiện tại không?
3. Có phương án thay thế an toàn hơn không?
Nếu câu trả lời của bất kỳ câu nào ở trên không chắc chắn, hãy tạm dừng thao tác và xác nhận với người dùng.
```

Việc tự kiểm tra này là một phần của System Prompt, kích hoạt mỗi lần trước khi thực thi thao tác rủi ro cao. Nó tương đương với việc lắp cho mô hình một "má phanh", buộc nó dừng lại suy nghĩ trước khi hành động.

## Quay lại bản chất của context engineering

Câu chuyện của Yue nhắc nhở chúng ta: context engineering không chỉ là vấn đề "quản lý bộ nhớ", mà còn là vấn đề "ranh giới an toàn".

Khi chúng ta thiết kế chiến lược nén, không thể chỉ cân nhắc "nhét thêm thông tin thế nào", mà còn phải cân nhắc "thông tin nào bị mất sẽ dẫn đến hậu quả thảm khốc".

Context engineering tốt, nên khiến mô hình ở bất kỳ thời điểm nào cũng biết:
- **Lằn ranh đỏ tuyệt đối không được đụng vào là gì** (đặt ở tầng không thể nén)
- **Tác vụ hiện tại cần tập trung là gì** (giữ tiêu điểm qua Todo Recap)
- **Nếu không nhớ rõ, thì nên dừng lại hỏi** (dự phòng qua cơ chế tự kiểm tra)

## Kết luận chương này

Mục tiêu của context engineering không phải là "khiến mô hình nhìn thấy tất cả thông tin", mà là "khiến mô hình nhìn thấy thông tin đúng vào thời điểm đúng" —— **đặc biệt là những thông tin không thể mất**.

Bản chất của ba phương pháp này đều là đang làm "điều phối sự chú ý":
- **Phân tầng** khiến mô hình biết "thông tin nào có thẩm quyền"
- **Cắt bớt + ghi ổ đĩa** khiến mô hình quyết định "thông tin nào là hiện tại cần thiết"
- **Nén + tách biệt tiêu điểm** khiến mô hình rõ ràng "hiện tại tôi nên tập trung vào cái gì"

Thay vì theo đuổi cửa sổ context lớn hơn, chi bằng dùng cửa sổ hiện có một cách có tổ chức hơn.


---

# Chương sáu: Khả năng quan sát biến hộp đen thành hộp kính — một case CONFLICT được định vị như thế nào

Context engineering khiến Agent có thể xử lý các tác vụ dài hơn, nhưng vấn đề mới cũng theo đó mà đến: khi nó bị lỗi, tôi hoàn toàn không biết đã xảy ra chuyện gì.

Có một lần, Agent Edit thất bại ba lần liên tiếp, cuối cùng đành bỏ cuộc. Trên console tôi chỉ thấy một dòng: `tool failed`. Không có lỗi chi tiết, không có context, không biết là bước nào có vấn đề.

Phản ứng đầu tiên của tôi là: tool Edit có bug. Nhưng sau khi kiểm tra code, logic trông đều không có vấn đề. Rốt cuộc vấn đề nằm ở đâu?

## Hiện trường thất bại

Tác vụ lần đó là thế này: tôi bảo Agent sửa file `core/llm.py`, thêm type annotation cho một hàm nào đó.

Luồng thực thi của Agent trông rất chuẩn:
1. Gọi Read đọc file
2. Gọi Edit sửa code
3. Trả về `tool failed`
4. Thử lại Edit, lại thất bại
5. Thử lại lần nữa, vẫn thất bại
6. Bỏ cuộc

Công cụ debug của tôi lúc đó rất thô sơ: in log ra console. Nhưng trong log chỉ có:

```
[Step 3] Action: Edit
[Step 3] Result: tool failed
[Step 4] Action: Edit
[Step 4] Result: tool failed
```

Tôi không biết nguyên nhân thất bại cụ thể, cũng không biết mô hình sau khi nhận kết quả này đã ra quyết định thế nào. Tôi chỉ có thể đoán mò: là đường dẫn viết sai? Quyền hạn không đủ? Hay là file bị khóa?

## Sau khi thêm Trace

Về sau tôi triển khai hệ thống Trace, ghi lại toàn bộ quỹ đạo của mỗi phiên hội thoại: bao gồm đầu vào mô hình của mỗi vòng, đầu ra, tham số gọi tool, kết quả tool trả về, lượng token dùng, v.v.

Chạy lại cùng tác vụ đó một lần, rồi mở Trace xem. Lúc này mới thấy được chuỗi bằng chứng đầy đủ.

**Step 2 - Kết quả của Read:**

```json
{
  "tool": "Read",
  "args": {"path": "core/llm.py"},
  "result": {
    "status": "success",
    "data": {
      "content": "...",
      "file_mtime_ms": 1733920000123,
      "file_size_bytes": 4217
    }
  }
}
```

**Step 3 - Lời gọi của Edit:**

```json
{
  "tool": "Edit",
  "args": {
    "path": "core/llm.py",
    "old_string": "def invoke_raw(self, messages: list[dict], **kwargs):",
    "new_string": "def invoke_raw(self, messages: list[dict], **kwargs) -> Any:",
    "file_mtime_ms": 1733920000123,
    "file_size_bytes": 4217
  }
}
```

**Step 3 - Kết quả của Edit:**

```json
{
  "status": "error",
  "error": {
    "code": "CONFLICT",
    "message": "File changed since last read. Expected mtime: 1733920000123, actual: 1733920000456"
  }
}
```

**Step 4 - Quyết định của mô hình:**

```
Thought: Chỉnh sửa thất bại, file có vẻ bị sửa đổi. Để tôi thử lại lần nữa.
Action: Edit (tham số giống hệt)
```

Sự thật đã rõ:

1. Sau khi Agent Read file, file bị chương trình bên ngoài (có thể là tính năng tự động lưu của IDE của tôi) sửa đổi
2. Tool Edit thực hiện kiểm tra optimistic lock, phát hiện mtime đã thay đổi, trả về CONFLICT
3. Nhưng mô hình không hiểu ý nghĩa của CONFLICT, tưởng chỉ là "thao tác thất bại", thế là thử lại với tham số giống hệt
4. Tất nhiên vẫn thất bại, vì file vẫn là bản mới
5. Mô hình thử lại liên tục, cho đến khi đạt số lần thử tối đa

## Phân tích nguyên nhân gốc

Case này phơi bày hai vấn đề:

**Thứ nhất, mô hình không hiểu mã lỗi.**

Trong prompt chỉ nói "tool Edit sẽ sửa đổi file", nhưng không bảo nó "nếu trả về CONFLICT thì nên làm gì". Mô hình thấy error, phản ứng bản năng là "thử lại lần nữa", chứ không phải "đọc lại".

**Thứ hai, log console quá thô sơ.**

Chỉ thấy `tool failed`, không thấy mã lỗi cụ thể CONFLICT, cũng không thấy sự so sánh mtime. Tôi với tư cách nhà phát triển, không thể định vị vấn đề qua log.

## Hành động khắc phục

### 1. Viết cách xử lý CONFLICT vào prompt

Tôi thêm quy trình xử lý rõ ràng vào prompt:

```
Nếu Edit trả về CONFLICT, nghĩa là file đã bị sửa đổi từ bên ngoài sau khi bạn đọc. Bạn bắt buộc phải:
1. Gọi lại Read đọc nội dung mới nhất
2. Kiểm tra sửa đổi của bạn còn áp dụng được không
3. Khi cần điều chỉnh nội dung sửa đổi để khớp với file mới
4. Thử Edit lại
Tuyệt đối cấm: gọi Edit lại với tham số giống hệt.
```

Như vậy mô hình sẽ biết CONFLICT không phải là "thất bại", mà là một trạng thái cần quy trình xử lý cụ thể.

### 2. Giữ lại bản ghi thất bại đầy đủ

Trước kia tôi có một xu hướng: sau khi thất bại chỉ giữ lại thông tin lỗi, không giữ lại context đầy đủ. Cảm thấy cái gì thành công mới đáng ghi lại, thất bại là "nhiễu".

Nhưng case này khiến tôi hiểu ra: **quỹ đạo thất bại là thông tin debug có giá trị nhất.**

Bây giờ Trace của tôi sẽ ghi lại đầy đủ mọi chi tiết của thất bại:
- Tham số đầy đủ của lời gọi tool
- Kết quả đầy đủ mà tool trả về (bao gồm chi tiết error)
- Quá trình suy luận của mô hình sau khi nhận kết quả
- Quyết định bước tiếp theo của mô hình

Những thông tin này sẽ không bị "làm sạch" đi, dù cho phiên hội thoại cuối cùng thành công, những lần thử thất bại ở giữa cũng đều được giữ lại toàn bộ.

### 3. Hiển thị mã lỗi then chốt trên console

Mặc dù Trace chi tiết lưu trong file, nhưng console cũng nên cho nhà phát triển một vài manh mối. Bây giờ đầu ra console của tôi sẽ hiển thị:

```
[Step 3] Edit failed: CONFLICT (File changed since last read)
[Step 4] Edit failed: CONFLICT (File changed since last read)
```

Ít nhất cho nhà phát triển biết "là CONFLICT, không phải lỗi khác".

## Giá trị của khả năng quan sát

Case này khiến tôi có cách hiểu mới về "khả năng quan sát".

Trước kia tôi tưởng, khả năng quan sát chính là "in nhiều log". Log càng nhiều càng tốt, càng chi tiết càng tốt.

Bây giờ tôi hiểu ra, **cốt lõi của khả năng quan sát là "chuỗi trách nhiệm"** —— có thể xâu lời gọi, kết quả, thay đổi trạng thái thành một chuỗi có thể truy vết.

Khi không có Trace, cái tôi thấy là:
- Đầu vào: giúp tôi sửa một file
- Đầu ra: tool failed
- Ở giữa đã xảy ra gì: hộp đen

Sau khi có Trace, cái tôi thấy là:
- Đầu vào: giúp tôi sửa một file
- Step 1: Read thành công, file mtime=123
- Step 2: Edit thất bại, CONFLICT, vì mtime đã thành 456
- Step 3: mô hình chọn thử lại Edit (quyết định sai)
- Đầu ra: tool failed

Mỗi bước đều rõ ràng nhìn thấy được, việc định vị vấn đề từ "đoán mò" biến thành "xem bằng chứng".

## Nguyên tắc thiết kế khả năng quan sát

Dựa trên kinh nghiệm này, tôi đã tổng kết mấy nguyên tắc thiết kế khả năng quan sát:

### 1. Có cấu trúc hơn văn bản

Đừng chỉ ghi lại mô tả văn bản kiểu "Edit failed", mà hãy ghi lại dữ liệu có cấu trúc:
```json
{
  "event": "tool_result",
  "tool": "Edit",
  "status": "error",
  "error_code": "CONFLICT",
  "error_details": {...}
}
```

Như vậy có thể dùng script để phân tích, thống kê, thậm chí tự động chẩn đoán.

### 2. Context phải đầy đủ

Khi ghi lại lời gọi tool, đừng chỉ ghi kết quả, hãy ghi lại context đầy đủ:
- Tên tool và tham số
- Trạng thái hội thoại lúc đó (bước thứ mấy, lượng token dùng)
- Phản ứng của mô hình sau khi nhận kết quả

Những thông tin này xâu chuỗi lại với nhau, mới có thể tái hiện toàn bộ quá trình ra quyết định.

### 3. Đừng làm sạch thất bại

Cả đường thành công và đường thất bại đều phải giữ lại. Đôi khi thất bại nói lên vấn đề rõ hơn thành công. Chẳng hạn case CONFLICT này, nếu chỉ ghi lại "cuối cùng bỏ cuộc", tôi sẽ mãi mãi không biết ở giữa đã xảy ra gì.

### 4. Đọc được cho cả người và máy

Trace nên có hai định dạng:
- JSONL: cho máy phân tích, ghi theo luồng, chi phí thấp
- HTML: cho con người đọc, trình bày trực quan, có thể gập/mở

Nhà phát triển nên có thể mở một file HTML, xem từng bước của Agent giống như "tua lại từng khung hình".

## Kết luận chương này

Khả năng quan sát không phải là "log rất nhiều", mà là "có thể xâu lời gọi, kết quả, thay đổi trạng thái thành chuỗi trách nhiệm".

Agent là hệ thống xác suất, không thể mãi mãi đúng. Nhưng khi nó bị lỗi, bạn cần có khả năng trả lời ba câu hỏi:
1. Nó đã làm gì? (chuỗi lời gọi)
2. Kết quả là gì? (chuỗi trả về)
3. Vì sao làm như vậy? (chuỗi ra quyết định)

Chỉ khi bạn có thể xâu ba chuỗi này lại với nhau, mới thực sự hiểu được hành vi của Agent, mới có thể khiến nó từ "hộp đen" trở thành "hộp kính".


---

# Chương bảy: Phương pháp luận tổng quát rút ra từ một dự án

Bảy chương trước, tôi đã đứt quãng kể lại toàn bộ quá trình từ khi lập dự án đến khi trưởng thành của Code Agent này. Mỗi chương đều là một cái bẫy cụ thể, và tôi đã bò ra khỏi đó như thế nào.

Chương này, tôi muốn rút những kinh nghiệm này ra, sắp xếp thành phương pháp luận có thể di chuyển sang bất kỳ dự án Agent nào.

## Tám nguyên tắc có thể di chuyển

**Thứ nhất, làm vòng khép kín tối thiểu chạy thông được trước, rồi mới bàn đến kiến trúc thanh lịch.**

Đừng vừa vào đã nghiên cứu best practice. Hãy làm một phiên bản xấu xí nhưng chạy được trước — nhận đầu vào, tìm kiếm code, đưa ra đề xuất, ghi vào file, bốn bước này chạy thông là được. Để dữ liệu thực chảy qua hệ thống, bạn mới biết nút thắt cổ chai ở đâu. Kiến trúc là kết quả sau khi được vấn đề dẫn dắt, không phải là điểm khởi đầu.

**Thứ hai, định nghĩa tiêu chí nghiệm thu trước, rồi mới mở rộng ranh giới năng lực.**

Đừng dùng danh sách tính năng làm tiêu chuẩn hoàn thành. Ngay giai đoạn V0 đã đặt ra 3-4 tiêu chuẩn cứng: có thể chạy nhiều bước ổn định không? Có thể tìm thấy bằng chứng không? Có thể đưa ra bản vá khả thi không? Thay đổi có kiểm soát được không? Không thỏa mãn thì không đi tiếp. Điều này đáng tin hơn nhiều so với "nhiều tính năng nhưng thường xuyên sập".

**Thứ ba, hành động tần suất cao nguyên tử hóa, hành động tần suất thấp kiểm soát và dự phòng.**

Những thao tác tần suất cao như tìm kiếm, đọc, chỉnh sửa, tách thành tool nguyên tử, mỗi bước một đầu ra. Đừng để mô hình tự kết hợp lệnh pipe — khi bị lỗi bạn hoàn toàn không biết là vấn đề của bước nào.

Tool vạn năng kiểu Bash giữ lại, nhưng chỉ xử lý những nhu cầu góc cạnh mà tool nguyên tử không phủ tới, vùng cấm rõ ràng: cấm đọc/tìm/liệt kê (những cái này đã có tool chuyên dụng).

**Thứ tư, giao thức ưu tiên hơn kỹ xảo, cấu trúc ưu tiên hơn cách nói.**

Đừng tốn quá nhiều thời gian tinh chỉnh "giọng điệu" của prompt. Trước tiên chuẩn hóa định dạng trả về của tool (status/data/text/error), nâng cấp giao thức lời gọi từ phân tích chuỗi lên Function Calling. Giao thức ổn định rồi, hệ thống mới có thể ổn định.

**Thứ năm, prompt thiết lập ranh giới trước, rồi mới bàn chiến lược.**

Trong System Prompt viết "tuyệt đối không được làm gì" trước (cấm đoán, cấm vượt giới hạn), rồi mới viết "nên làm thế nào". Lằn ranh đỏ đặt ở các tầng không thể nén như L1/L2, đừng đặt chỉ thị an toàn vào L3 sẽ bị nén.

Ràng buộc then chốt không vào lịch sử động, đây là bài học mà Giám đốc An toàn AI của Meta phải đổi bằng 200 email.

**Thứ sáu, quản trị context theo "sự chú ý", chứ không phải chất đống theo "dung lượng".**

Đừng theo đuổi nhét thêm thông tin, hãy để mô hình nhìn thấy thông tin đúng vào thời điểm đúng. Phân tầng (L1/L2/L3) để mô hình biết thông tin nào có thẩm quyền; cắt bớt + ghi ổ đĩa để kiểm soát quy mô đầu vào một lần; nén + tập trung (Summary + Todo Recap) để quản lý nhiễu của lịch sử dài hạn.

**Thứ bảy, không có khả năng quan sát, thì không có khả năng debug.**

Agent là hệ thống xác suất, không thể mãi mãi đúng. Nhưng khi nó bị lỗi, bạn cần có thể trả lời: nó đã làm gì? Kết quả là gì? Vì sao làm như vậy?

Triển khai hệ thống Trace, ghi lại chuỗi lời gọi, chuỗi trả về, chuỗi ra quyết định. Đừng chỉ ghi lại đường thành công, quỹ đạo thất bại thường có giá trị hơn.

**Thứ tám, giữ lại quỹ đạo thất bại, hệ thống mới có thể tiến hóa.**

Đừng vì sợ "làm ô nhiễm lịch sử" mà làm sạch bản ghi thất bại. Lỗi CONFLICT, thử lại do timeout, mô hình đoán mò —— những cái này đều ghi lại. Chỉ khi thấy được toàn bộ quá trình thất bại, mới có thể định vị nguyên nhân gốc, mới có thể cố định kinh nghiệm kiểu "gặp CONFLICT bắt buộc phải Read lại" vào prompt.

## Lời cuối: chúng ta đều đang "dọn dẹp" cho LLM

Sau khi làm xong dự án này, tôi có một cảm nhận đặc biệt sâu sắc, có thể nghe hơi thô, nhưng lời thô mà lý không thô:

**Cốt lõi của phát triển Agent, không phải là khiến mô hình tự do hơn, mà là thông qua thiết kế kỹ thuật, ràng buộc "năng lực bất định" của mô hình vào "phạm vi có thể kiểm soát tối thiểu". Nói trắng ra, chúng ta chính là đang dọn dẹp cho LLM.**

Vì sao lại nói vậy?

Bạn xem này, LLM rất mạnh, có thể viết code, có thể đọc tài liệu, có thể suy luận. Nhưng nó giống như một thực tập sinh đặc biệt thông minh nhưng cũng đặc biệt không đáng tin cậy —

- Bạn bảo nó đi in file, nó có thể gọi hết tất cả máy in trong công ty;
- Bạn bảo nó tổng hợp biên bản họp, nó có thể lôi cả cuộc họp tuần trước vào;
- Bạn bảo nó viết một hàm, nó viết cực mượt, nhưng đặt tên biến toàn là `a`, `b`, `c`, còn tiện tay sửa file mà bạn không bảo sửa.

**Cái "mạnh" của nó là mạnh về năng lực, nhưng cái "không đáng tin cậy" là không đáng tin cậy về tính xác định.**

Và chúng ta làm kỹ thuật Agent, về bản chất chính là đang giải quyết mâu thuẫn này:

| Bản tính của mô hình | Đối sách kỹ thuật của chúng ta |
|-----------|--------------|
| Thích tự do phát huy | Dùng Function Calling khóa định dạng lời gọi |
| Context nhiều là "mất trí nhớ" | Dùng phân tầng L1/L2/L3 + nén Summary |
| Bị lỗi không biết tự kiểm tra | Dùng Trace ghi lại từng bước, khiến lỗi có thể truy vết |
| Tác vụ dài dễ chạy chệch | Dùng Todo + Task phân tách, giảm độ phức tạp từng bước |
| Không hiểu tri thức chuyên ngành | Dùng Skills cố định SOP, cho nó "có não" |

Bạn xem nội dung bảy chương này, từ nguyên tử hóa tool đến context engineering, từ khả năng quan sát đến sub-agent —— **mỗi tầng đều là đang "vá lỗi" cho mô hình, giúp nó dọn dẹp mớ hỗn độn.**

Nhưng đây chính là chỗ thú vị nhất.

Trước kia tôi nghĩ, trong thời đại AI giá trị của kỹ sư sẽ giảm xuống. Bây giờ tôi thấy ngược lại: **mô hình càng mạnh, càng cần năng lực kỹ thuật để điều khiển nó.** Giống như động cơ ô tô ngày càng mạnh, nhưng khung gầm, phanh, hệ thống treo tốt ngược lại càng quan trọng.

**Chúng ta không phải đang cạnh tranh với mô hình, mà đang hợp tác với mô hình —— nó phụ trách "có thể làm gì", chúng ta phụ trách "làm sao để nó làm đúng một cách ổn định".**

Vậy nên, nếu bạn hỏi tôi thu hoạch lớn nhất sau khi làm xong dự án này là gì?

Không phải là học được kiến trúc cao siêu nào đó, mà là nghĩ thông một đạo lý mộc mạc: **Agent xuất sắc không phải là sản phẩm của "khiến mô hình tự do hơn", mà là kết quả của "ràng buộc tính bất định xuống mức tối thiểu".**

Sự chuyển biến nhận thức này, có thể đáng giá hơn tất cả code.
