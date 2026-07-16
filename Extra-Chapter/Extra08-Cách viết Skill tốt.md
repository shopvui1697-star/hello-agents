# 05. Cách viết một Skill tốt

> Skill là gì? Làm sao để viết một skill tốt?
> Chúng ta sẽ đi theo tư duy thiết kế của skill-creator để tìm ra câu trả lời.
> Mục tiêu của bài viết này là: đọc xong nó, bạn sẽ hiểu được các thực hành tốt nhất (best practices) khi viết skill.

---

![Mục lục](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra08-figures/toc.png)

## I. Skill là gì?

### 1.1 Định nghĩa

Skill là một thư mục, bên trong chứa các tài nguyên như tài liệu chỉ dẫn, tài liệu tham khảo, script có thể thực thi, v.v. Khi AI nhận được nó, AI có thể đảm nhận được một công việc cụ thể mà vốn dĩ nó không biết làm.

Ví dụ, trong một thư mục skill `pdf-editor` có thể có một tài liệu chỉ dẫn thao tác "cách xử lý PDF", một script Python để xoay PDF, một tài liệu tham khảo API — AI không cần phải tìm bất cứ thứ gì từ bên ngoài nữa, mọi thứ đều đã có sẵn trong thư mục này.

Khái niệm này không giới hạn ở một sản phẩm cụ thể nào. Dù là Codex, Claude hay một AI Agent nào khác, bản chất của skill đều giống nhau. Bạn có thể hiểu nó như một **plugin năng lực** của AI — cắm vào thì AI có thêm một sở trường; rút ra thì AI vẫn là trợ lý đa năng như cũ.

### 1.2 Hình thái tối giản

Một skill tối thiểu chỉ cần một file:

```
my-skill/
└── SKILL.md
```

Cấu trúc của `SKILL.md` rất đơn giản — phần nửa trên cho AI biết "khi nào thì dùng tôi", phần nửa dưới cho AI biết "cụ thể phải làm thế nào":

```yaml
---
name: my-skill                    # ← Phần nửa trên: metadata
description: >-                   #    AI dựa vào đây để quyết định có kích hoạt skill này không
  Khi người dùng cần làm một việc nào đó, hãy dùng skill này.
---

Phần nửa dưới: chỉ dẫn thao tác     # ← AI chỉ đọc đến đây sau khi kích hoạt skill
Thực hiện theo các bước sau...
```

Phần nửa trên gọi là **frontmatter** (đoạn YAML nằm giữa hai dấu `---`), gồm hai trường `name` và `description`. Mỗi khi bắt đầu một cuộc hội thoại, AI sẽ quét frontmatter của tất cả các skill đã cài đặt, dựa vào description để phán đoán "skill này có liên quan đến yêu cầu hiện tại không" — đây là **căn cứ duy nhất** để skill được kích hoạt.

Phần nửa dưới gọi là **body** (phần nội dung Markdown), là chỉ dẫn thao tác chỉ được nạp vào sau khi skill được kích hoạt. Nếu skill không được kích hoạt, AI sẽ không bao giờ đọc đến phần này.

### 1.3 Cấu trúc đầy đủ

Khi một skill trở nên phức tạp, chỉ dựa vào một file SKILL.md là chưa đủ.

Ví dụ bạn muốn làm một skill "xử lý PDF": trong SKILL.md đã viết quy trình xử lý, nhưng đoạn code xoay PDF lần nào cũng giống nhau, mỗi lần lại bắt AI viết lại vừa lãng phí thời gian vừa có thể sai sót — chi bằng đặt sẵn một script Python đã viết hoàn chỉnh. Hay ví dụ skill "trình tạo dự án frontend": mỗi lần đều cần một bộ file mẫu HTML/React, chi bằng đặt sẵn một thư mục template để AI sao chép ra rồi sửa.

Vì vậy một thư mục skill đầy đủ có thể bao gồm những thứ sau:

```
skill-name/
├── SKILL.md                  # [Bắt buộc] File đầu vào: frontmatter + body
├── agents/
│   └── openai.yaml           # [Khuyến nghị] "Danh thiếp" của skill
├── scripts/                  # [Tùy chọn] Script có thể thực thi
├── references/               # [Tùy chọn] Tài liệu tham khảo
└── assets/                   # [Tùy chọn] Template sản phẩm đầu ra
```

Giải thích từng phần:

- **SKILL.md** — file bắt buộc duy nhất, đã giới thiệu ở phần trên

- **scripts/** — các chương trình đã viết sẵn, AI không cần phải hiểu nó, chỉ cần gọi shell để thực thi là được. Ví dụ `scripts/rotate_pdf.py`, AI chỉ cần chạy `python rotate_pdf.py input.pdf 90` là xoay được PDF, không cần mỗi lần lại viết lại logic xoay. Phù hợp với những thao tác **yêu cầu kết quả phải chính xác, không thể để AI tùy ý sáng tạo**

- **references/** — tài liệu tham khảo mà AI cần tra cứu trong quá trình làm việc. Ví dụ một skill "truy vấn BigQuery", AI cần biết công ty có những bảng nào, mỗi bảng có những trường gì, những thông tin này được đặt trong `references/schema.md`, khi cần AI mới đọc. Điểm khác biệt với scripts là: references là để AI **đọc**, còn scripts là để AI **thực thi**

- **assets/** — không phải để AI xem, mà là các file được dùng trực tiếp trong sản phẩm cuối cùng. Ví dụ một skill "trình tạo dự án frontend", trong `assets/frontend-template/` đặt sẵn một bộ code mẫu HTML/React, AI trực tiếp sao chép bộ template này ra rồi sửa trên đó. Hay ví dụ `assets/logo.png` là logo công ty, khi AI tạo trang web sẽ trực tiếp tham chiếu đến nó. AI không cần "hiểu" một tấm ảnh logo, chỉ cần biết nó ở đâu, khi nào thì đặt vào

- **agents/openai.yaml** — "danh thiếp" của skill. Nhiều sản phẩm AI sẽ hiển thị một danh sách skill trên giao diện để người dùng chọn hoặc tìm kiếm. File này lưu trữ chính là tên hiển thị, mô tả ngắn, biểu tượng, v.v. hiển thị trong danh sách đó. Nó không ảnh hưởng đến hành vi của AI, thuần túy dùng cho giao diện sản phẩm

---

## II. Bạn đang viết chỉ dẫn cho người, hay đang viết chỉ dẫn cho AI?

Biết được skill là gì rồi, bước tiếp theo là viết một cái. Nhưng skill mà đa số mọi người viết ra lần đầu tiên đều mắc cùng một vấn đề.

Hãy xem một ví dụ. Giả sử bạn muốn làm một skill "review code", bạn có thể sẽ viết như thế này:

```markdown
---
name: code-review
description: Skill review code
---

# Code Review Skill

## Bối cảnh
Skill này được đúc kết từ nhiều năm kinh nghiệm review code của nhóm, nhằm nâng cao chất lượng code và hiệu quả cộng tác của nhóm.

## Nguyên tắc review
- Giữ giọng điệu chuyên nghiệp, mang tính xây dựng
- Tập trung vào chất lượng code chứ không phải phong cách cá nhân
- Cân bằng giữa sự nghiêm khắc và linh hoạt

## Cách sử dụng
Khi người dùng gửi code, hãy review code một cách toàn diện và đưa ra đề xuất cải thiện. Chú ý giữ thái độ thân thiện và khích lệ.

## Lịch sử phiên bản
- v1.0: Phiên bản khởi đầu
- v1.1: Bổ sung hỗ trợ cho Python
```

Nếu đây là một tài liệu nhóm dành cho con người đọc, thì nó viết khá tốt — có bối cảnh, có nguyên tắc, có cách sử dụng, thậm chí còn có cả lịch sử phiên bản.

Nhưng người đọc của skill là AI. Hãy nhìn lại nó bằng góc nhìn này:

- **"Đúc kết từ nhiều năm kinh nghiệm của nhóm"** — AI không quan tâm skill này có nguồn gốc từ đâu, nó chỉ cần biết **bây giờ phải làm thế nào**
- **"Giữ giọng điệu chuyên nghiệp, mang tính xây dựng"** — con người đọc vào thì nắm được một cảm giác đại khái, nhưng AI sẽ triển khai "chuyên nghiệp" và "mang tính xây dựng" thành vô số tổ hợp, mỗi lần output đều khác nhau
- **"Cân bằng giữa sự nghiêm khắc và linh hoạt"** — người review giàu kinh nghiệm biết khi nào nên nghiêm khắc khi nào nên linh hoạt, nhưng AI không có trực giác này, câu này coi như chưa nói gì
- **"Review toàn diện, đưa ra đề xuất cải thiện"** — đây là kỳ vọng đối với người review, nhưng cái AI cần là: kiểm tra cái gì trước? Kiểm tra cái gì sau? Vấn đề nào bắt buộc phải chỉ ra? Vấn đề nào có thể bỏ qua?
- **"Lịch sử phiên bản"** — mỗi lần AI được đánh thức đều hoàn toàn mới, v1.0 hay v1.1 chẳng có ý nghĩa gì với nó
- **description chỉ viết "skill review code"** — AI dựa vào description để phán đoán có kích hoạt hay không, năm chữ "skill review code" quá mơ hồ: người dùng nói "xem giúp tôi đoạn code này" có kích hoạt không? "Hàm này hiệu năng thế nào" có kích hoạt không?

Từng điều nhìn riêng lẻ đều không phải là "sai", nhưng tất cả đều được viết cho con người đọc. **Vấn đề không nằm ở chỗ viết chưa đủ nhiều, mà nằm ở chỗ viết sai đối tượng.**

Vậy cách viết đúng trông như thế nào? Chúng ta hãy xem một câu trả lời có sẵn — skill-creator của codex. Nó là một "skill để tạo skill", và chính bản thân SKILL.md của nó là một tài liệu về "cách viết chỉ dẫn cho AI" theo best practices.

---

## III. Khung tổng thể của skill-creator

Mở file SKILL.md của skill-creator (khoảng 370 dòng), trước khi đi sâu vào bất kỳ chi tiết nào, chúng ta hãy thiết lập một nhận thức tổng thể về nó.

Vấn đề mà skill-creator cần giải quyết chỉ có một: **làm sao trong một cửa sổ ngữ cảnh (context window) có hạn, đưa cho AI những chỉ dẫn hiệu quả nhất?**

Xoay quanh vấn đề này, nó đưa ra một hệ thống thiết kế hoàn chỉnh, có thể hiểu qua ba tầng.

### Tầng thứ nhất: Ràng buộc căn bản — Sự súc tích

Cửa sổ ngữ cảnh của AI là có hạn, và được dùng chung (system prompt, lịch sử hội thoại, metadata của tất cả các skill đã cài đặt đều nằm trong đó). Skill của bạn chiếm càng nhiều, thì phần còn lại cho các mục đích khác càng ít. Vì vậy nguyên tắc đầu tiên của skill-creator chính là: **mỗi câu chữ đều phải xứng đáng với số token mà nó chiếm dụng**.

### Tầng thứ hai: Hai chiều thiết kế

Dưới ràng buộc "súc tích" này, khi viết skill bạn đối mặt với hai quyết định cốt lõi:

**Chiều thứ nhất: Đặt thông tin ở đâu?**

Không phải mọi thông tin đều cần được nạp ngay từ đầu. skill-creator thiết kế một kiến trúc phân tầng ba cấp, cho phép những thông tin khác nhau đi vào ngữ cảnh vào những thời điểm khác nhau:

![Cấu trúc chuẩn của Skill và nạp ba cấp](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra08-figures/skill-structure.png)

- **L1 (metadata)**: luôn nằm trong ngữ cảnh, khoảng 100 từ — AI dựa vào nó để phán đoán có kích hoạt skill này hay không
- **L2 (SKILL.md body)**: chỉ nạp sau khi được kích hoạt, giữ trong khoảng dưới 5k từ — chỉ dẫn thao tác
- **L3 (scripts/references/assets)**: dùng theo nhu cầu, không giới hạn — trong đó scripts được thực thi mà không nạp vào, chi phí token bằng không

Điều này giải quyết vấn đề "làm sao dùng ít token nhất để chứa nhiều thông tin nhất".

**Chiều thứ hai: Cho AI bao nhiêu quyền tự do?**

Không phải mọi tác vụ đều phù hợp để AI tự do sáng tạo.

Lấy một ví dụ: để AI viết một bài blog kỹ thuật, mười người viết ra mười phong cách khác nhau đều được — bạn chỉ cần đưa ra định hướng, còn viết cụ thể thế nào thì để AI tự quyết định. Đây chính là **quyền tự do cao**.

Nhưng để AI tạo một file cấu hình YAML thì lại khác. Ví dụ file `openai.yaml` mà skill-creator cần tạo, bên trong có một trường `short_description`, yêu cầu 25-64 ký tự, viết hoa chữ cái đầu, không được có dấu ngoặc kép. AI viết thành 65 ký tự? Không được, giao diện sản phẩm sẽ cắt bớt. Viết thành 24 ký tự? Không được, không qua được kiểm tra (validation). Quên viết hoa chữ cái đầu? Giao diện hiển thị không nhất quán. Loại tác vụ này chỉ lệch một ký tự là có vấn đề, bạn không thể để AI tự do sáng tạo, mà phải dùng script để khóa chặt định dạng — đây chính là **quyền tự do thấp**. Loại tác vụ này gọi là "thao tác dễ vỡ" (fragile operation): không phải nó phức tạp, mà là **làm đúng chỉ có một cách, làm sai có cả trăm cách**.

![Quang phổ quyền tự do](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra08-figures/freedom-spectrum.png)

Điều này giải quyết vấn đề "làm sao cân bằng giữa tính linh hoạt của AI và độ tin cậy của output".

### Tầng thứ ba: Quy trình triển khai

Có nguyên tắc và kiến trúc rồi, cuối cùng skill-creator đưa ra một quy trình tạo skill gồm sáu bước, biến tư duy thiết kế thành các bước thao tác có thể thực thi:

![Quy trình tạo sáu bước](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra08-figures/creation-flow.png)

Hiểu → Lập kế hoạch → Khởi tạo → Chỉnh sửa → Kiểm tra → Lặp lại (iterate). Trong đó script xuyên suốt cả quy trình, tạo thành một chuỗi đảm bảo chất lượng mang tính xác định (deterministic):

![Mối quan hệ tương tác giữa các file](https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra08-figures/file-interaction.png)

### Tổng quan khung

Mối quan hệ giữa ba tầng:

```
Súc tích (ràng buộc căn bản)                          → Chương IV
 ├── Đặt thông tin ở đâu? → Kiến trúc phân tầng ba cấp → Chương V
 ├── Cho AI bao nhiêu quyền tự do? → Quang phổ tự do và script → Chương VI
 └── Triển khai thế nào? → Quy trình tạo sáu bước      → Chương VII
```

Mỗi chương tiếp theo đều được triển khai trong khung này.

---

## IV. Ràng buộc căn bản: Sự súc tích
> Vị trí trong khung: Tầng thứ nhất

### 4.1 Ràng buộc cốt lõi

Cửa sổ ngữ cảnh của AI giống như một bàn làm việc — số tài liệu mà nó có thể trải ra cùng một lúc là có hạn. Mà trên bàn làm việc này đã có sẵn không ít thứ rồi: quy tắc riêng của hệ thống, những lời người dùng đã nói trước đó, phần giới thiệu của tất cả các skill đã cài đặt. Skill của bạn một khi được kích hoạt, nội dung của nó cũng phải trải lên đó. Bàn làm việc chỉ lớn có vậy, bạn chiếm càng nhiều, không gian dành cho những thứ khác càng ít.

Vì vậy skill-creator đã viết điều này thành nguyên tắc đầu tiên:

> The context window is a public good. Skills share the context window with everything else Codex needs: system prompt, conversation history, other Skills' metadata, and the actual user request.

Vì không gian bàn làm việc có hạn, vậy khi viết skill làm sao phán đoán một đoạn nội dung có nên đưa vào hay không? skill-creator đưa ra một giả định tiền đề: **bản thân AI vốn đã rất thông minh rồi, bạn chỉ cần bổ sung những thứ mà nó không biết.**

> Default assumption: Codex is already very smart. Only add context Codex doesn't already have.

Dựa trên giả định này, trước khi viết mỗi đoạn nội dung hãy tự hỏi hai câu:

- "AI có phải đã biết điều này rồi không?" — ví dụ "vòng lặp for trong Python viết thế nào", AI đương nhiên biết, không cần dạy
- "Đoạn nội dung này có xứng đáng chiếm không gian trên bàn làm việc không?" — một đoạn giải thích 200 chữ, liệu có thể thay thế bằng một ví dụ code 10 dòng không?

**Suy luận thực tiễn**: Dùng ví dụ súc tích thay cho lời giải thích dài dòng. Một ví dụ code tốt hơn ba đoạn mô tả bằng chữ.

### 4.2 Cái gì không nên đưa vào Skill?

Skill-creator đã liệt kê rõ ràng một **danh sách cấm**:

> A skill should only contain essential files that directly support its functionality. Do NOT create extraneous documentation or auxiliary files.

Những file không nên có:
- README.md
- INSTALLATION_GUIDE.md
- QUICK_REFERENCE.md
- CHANGELOG.md

> The skill should only contain the information needed for an AI agent to do the job at hand. It should not contain auxiliary context about the process that went into creating it, setup and testing procedures, user-facing documentation, etc. Creating additional documentation files just adds clutter and confusion.

Lý do rất đơn giản: người đọc của skill là AI, không phải lập trình viên. AI không cần những "tài liệu hỗ trợ cho con người" như hướng dẫn cài đặt, nhật ký cập nhật, tài liệu tra cứu nhanh. Mỗi file thừa đều là nhiễu.

### 4.3 Khi viết ràng buộc, "không làm gì" chính xác hơn "làm gì"

Sự súc tích không chỉ là "viết ít", mà còn bao gồm "viết đúng". Hãy xem một ví dụ.

Khi skill-creator tạo skill `laotou-thought-style` (một skill về phong cách viết), nó đã **không** viết:

```
Hãy viết bằng giọng điệu ấm áp, tiết chế, sâu sắc.
```

Kiểu mô tả tích cực (positive) này nhìn thì có vẻ rõ ràng, nhưng đối với AI, mức độ "ấm áp", sự cân bằng giữa "tiết chế" và "sâu sắc" — tất cả đều là những khoảng mơ hồ.

Cái mà nó làm là viết một **danh sách anti-pattern** (`references/anti-patterns.md`):

| Đừng làm thế này | Triệu chứng | Cách sửa |
|-----------|------|--------|
| Chồng chất nhân vật | Xuất hiện liên tục nhiều tên và lời thoại | Giữ lại một cảnh xung đột, bổ sung phần đúc kết trừu tượng |
| Chỉ có triết lý sáo rỗng mà không có hành động | Cả bài toàn "phải kiên trì, phải nỗ lực" | Đổi thành một bước nhỏ có thể làm ngay hôm nay |
| Giảng đạo lý trực tiếp | Mở đầu đã nói quy luật | Trải bối cảnh đời thường trước |
| Kết thúc quá gắt | Kết bài "phải thay đổi!" | Đổi thành "từ từ thôi", "vậy là được" |
| Tuyệt đối hóa quá mức | "mãi mãi", "chắc chắn" | Thêm từ giới hạn "phần lớn thời gian", "thường thì" |

**Mỗi điều đều cụ thể, có thể phát hiện được, và có phương án sửa chữa rõ ràng.**

Nguyên lý đằng sau:

```
"Làm gì" → Mô tả một vùng khả thi vô hạn lớn → AI đi lang thang ngẫu nhiên trong đó
"Không làm gì" → Vẽ ranh giới trên vùng khả thi → Không gian hành vi của AI bị thu hẹp vào phạm vi bạn muốn
```

Bản thân skill-creator cũng tuân theo nguyên tắc này — SKILL.md của nó dùng một phần lớn dung lượng để nói "cái gì không nên viết" (What to Not Include in a Skill), thay vì nói chung chung "viết nội dung cho tốt".

Khi bạn viết xong SKILL.md, hãy làm một "bài kiểm tra đảo ngược": mỗi chỉ dẫn tích cực, liệu có thể viết lại thành dạng "đừng làm X" không? Nếu có thể, thì sau khi viết lại thường sẽ chính xác hơn.

### 4.4 Thống nhất dùng giọng mệnh lệnh

skill-creator yêu cầu phần nội dung của SKILL.md thống nhất dùng **giọng mệnh lệnh/nguyên thể** (Always use imperative/infinitive form). Đây không phải là sở thích thẩm mỹ, mà là để giảm sự mơ hồ — câu mệnh lệnh tự bản chất chính là chỉ dẫn.

---

## V. Chiều thiết kế thứ nhất: Đặt thông tin ở đâu?
> Vị trí trong khung: Tầng thứ hai — Chiều thứ nhất

Trong phần tổng quan khung ở chương III, chúng ta đã thấy toàn cảnh của kiến trúc phân tầng ba cấp. Chương này sẽ triển khai chi tiết của nó.

### 5.1 Nạp lũy tiến ba cấp (progressive loading)

Định nghĩa nguyên văn của skill-creator về ba cấp:

> 1. **Metadata (name + description)** - Always in context (~100 words)
> 2. **SKILL.md body** - When skill triggers (<5k words)
> 3. **Bundled resources** - As needed by Codex (Unlimited because scripts can be executed without reading into context window)

| Cấp | Nội dung | Khi nào nằm trong ngữ cảnh | Chi phí token |
|------|------|--------------|-----------|
| **L1** | frontmatter (name + description) | **Luôn luôn** | ~100 từ |
| **L2** | SKILL.md body | Nạp sau khi kích hoạt | <5k từ |
| **L3** | scripts/ references/ assets/ | Nạp theo nhu cầu | Không giới hạn |

**Về bản chất đây là một hệ thống quản lý entropy thông tin**:

- **L1 là bộ lọc** — sàng lọc ra đúng cái skill cần dùng hiện tại từ hàng chục skill đã cài đặt. description không chính xác → kích hoạt nhầm hoặc bỏ sót kích hoạt
- **L2 là sổ tay thao tác** — sau khi kích hoạt sẽ cho AI biết phải làm thế nào. Quá dài → sự chú ý bị pha loãng. Body giữ trong khoảng dưới 500 dòng
- **L3 là hộp công cụ** — chỉ mở khi cần. Trong đó scripts/ hiệu quả nhất — **thực thi mà không nạp vào**, chi phí token bằng không

### 5.2 Frontmatter: Toàn bộ nguồn gốc của cơ chế kích hoạt

Frontmatter chỉ có hai trường bắt buộc: `name` và `description`. Nhưng cách viết description cực kỳ quan trọng:

> This is the primary triggering mechanism for your skill, and helps Codex understand when to use the skill.

description của chính skill-creator được viết như thế này:

```yaml
description: Guide for creating effective skills. This skill should be used when
  users want to create a new skill (or update an existing skill) that extends
  Codex's capabilities with specialized knowledge, workflows, or tool integrations.
```

Nó không chỉ nói "làm gì" (creating effective skills), mà còn nói "khi nào dùng" (when users want to create a new skill or update an existing skill).

**Quy tắc then chốt**:
- Đặt tất cả thông tin "when to use" trong description, **đừng đặt trong body**. Body chỉ được nạp sau khi kích hoạt, lúc đó Codex đã quyết định dùng rồi, thông tin "khi nào dùng" đã muộn
- Đừng đặt các trường khác ngoài `name` và `description` trong frontmatter (trừ `license`, `allowed-tools`, `metadata`)

Một ví dụ description tốt (skill docx):

> "Comprehensive document creation, editing, and analysis with support for tracked changes, comments, formatting preservation, and text extraction. Use when Codex needs to work with professional documents (.docx files) for: (1) Creating new documents, (2) Modifying or editing content, (3) Working with tracked changes, (4) Adding comments, or any other document tasks"

### 5.3 Sự khác biệt bản chất giữa bốn loại tài nguyên đóng gói

Hiểu được sự khác biệt của bốn loại tài nguyên này chính là chìa khóa để hiểu toàn bộ hệ thống skill:

#### Scripts (`scripts/`)

Code có thể thực thi (Python/Bash, v.v.), dùng cho những tác vụ cần **độ tin cậy mang tính xác định** hoặc phải viết lại nhiều lần.

- **Khi nào cần**: cùng một đoạn code lần nào cũng phải viết lại, hoặc cần output đáng tin cậy mang tính xác định
- **Ví dụ**: `scripts/rotate_pdf.py` dùng cho tác vụ xoay PDF
- **Ưu điểm cốt lõi**: tiết kiệm token, mang tính xác định, có thể thực thi mà không nạp vào cửa sổ ngữ cảnh
- **Lưu ý**: script đôi khi vẫn cần được Codex đọc, để vá lỗi hoặc thích ứng môi trường

#### References (`references/`)

Tài liệu và tài liệu tham khảo, được nạp vào ngữ cảnh khi cần, hỗ trợ quá trình suy nghĩ của Codex.

- **Khi nào cần**: tài liệu chi tiết mà Codex cần tham khảo khi làm việc
- **Ví dụ**: `references/finance.md` (schema tài chính), `references/api_docs.md` (đặc tả API), `references/policies.md` (chính sách công ty)
- **Công dụng**: schema database, tài liệu API, kiến thức lĩnh vực, chính sách công ty, hướng dẫn quy trình chi tiết
- **Ưu điểm cốt lõi**: giữ cho SKILL.md tinh gọn, chỉ nạp khi Codex phán đoán là cần
- **Best practice**: nếu file rất lớn (>10k từ), hãy đưa mẫu tìm kiếm grep vào trong SKILL.md
- **Tránh trùng lặp**: thông tin chỉ nên tồn tại trong SKILL.md **hoặc** trong file references, không được có ở cả hai nơi. Thông tin chi tiết ưu tiên đặt trong references, SKILL.md chỉ giữ lại chỉ dẫn quy trình cốt lõi và hướng dẫn workflow

#### Assets (`assets/`)

Không phải file để nạp vào ngữ cảnh, mà là tài nguyên được dùng trực tiếp trong sản phẩm đầu ra của Codex.

- **Khi nào cần**: những file mà skill cần dùng trong output cuối cùng
- **Ví dụ**: `assets/logo.png` (tài nguyên thương hiệu), `assets/slides.pptx` (template PPT), `assets/frontend-template/` (mẫu HTML/React), `assets/font.ttf` (font chữ)
- **Công dụng**: template, hình ảnh, biểu tượng, code mẫu, font chữ, tài liệu ví dụ — những thứ này sẽ được sao chép hoặc chỉnh sửa
- **Ưu điểm cốt lõi**: tách tài nguyên output ra khỏi tài liệu, Codex có thể sử dụng chúng mà không cần nạp vào ngữ cảnh

#### Agents metadata (`agents/openai.yaml`) (khuyến nghị)

Metadata hướng đến UI, không phải để AI đọc, mà để frontend sản phẩm đọc:

- Bao gồm các trường `display_name`, `short_description`, `default_prompt`, v.v.
- Được sinh ra một cách xác định thông qua script `generate_openai_yaml.py`, thay vì viết tay
- Sau khi cập nhật SKILL.md phải kiểm tra xem `agents/openai.yaml` còn khớp không, nếu lỗi thời thì sinh lại
- Định nghĩa trường chi tiết xem tại `references/openai_yaml.md`

### 5.4 Ba mô hình thực chiến của tiết lộ lũy tiến (progressive disclosure)

Skill-creator đưa ra ba mô hình cụ thể để tách nội dung ra thành references:

**Pattern 1: Hướng dẫn cấp cao + file tham khảo**

```markdown
# PDF Processing

## Quick start
Extract text with pdfplumber:
[code example]

## Advanced features
- **Form filling**: See [FORMS.md](FORMS.md) for complete guide
- **API reference**: See [REFERENCE.md](REFERENCE.md) for all methods
- **Examples**: See [EXAMPLES.md](EXAMPLES.md) for common patterns
```

Codex chỉ nạp FORMS.md, REFERENCE.md hoặc EXAMPLES.md khi cần.

**Pattern 2: Tổ chức theo lĩnh vực**

Skill đa lĩnh vực/đa biến thể, tách theo lĩnh vực để tránh nạp nội dung không liên quan:

```
bigquery-skill/
├── SKILL.md (overview and navigation)
└── reference/
    ├── finance.md (revenue, billing metrics)
    ├── sales.md (opportunities, pipeline)
    ├── product.md (API usage, features)
    └── marketing.md (campaigns, attribution)
```

Khi người dùng hỏi về chỉ số bán hàng, Codex chỉ đọc `sales.md`.

Cũng áp dụng cho các tình huống đa framework/đa biến thể:

```
cloud-deploy/
├── SKILL.md (workflow + provider selection)
└── references/
    ├── aws.md (AWS deployment patterns)
    ├── gcp.md (GCP deployment patterns)
    └── azure.md (Azure deployment patterns)
```

**Pattern 3: Chi tiết có điều kiện**

Chức năng cơ bản hiển thị trực tiếp, chức năng nâng cao liên kết theo nhu cầu:

```markdown
# DOCX Processing

## Creating documents
Use docx-js for new documents. See [DOCX-JS.md](DOCX-JS.md).

## Editing documents
For simple edits, modify the XML directly.

**For tracked changes**: See [REDLINING.md](REDLINING.md)
**For OOXML details**: See [OOXML.md](OOXML.md)
```

### 5.5 Hai hướng dẫn tránh bẫy quan trọng

1. **Tránh tham chiếu lồng nhau sâu** — tất cả file reference nên được liên kết trực tiếp từ SKILL.md, đừng lồng nhau kiểu A → B → C
2. **File dài thì thêm mục lục** — file reference vượt quá 100 dòng cần thêm TOC ở đầu, để Codex tiện xem trước toàn cảnh

### 5.6 Những lỗi đặt sai tầng thường gặp

| Lỗi | Hậu quả | Cách sửa |
|------|------|------|
| Đặt điều kiện kích hoạt trong body | body chỉ được nạp sau khi kích hoạt, đã muộn | Đặt vào description của frontmatter |
| "When to Use This Skill" viết trong body | Như trên, Codex đã quyết định dùng rồi mới thấy | Chuyển sang description |
| Nhồi chi tiết tham khảo vào SKILL.md | body phình to, mật độ thông tin giảm | Tách sang references/, body chỉ đặt link tham chiếu |
| Thao tác mang tính xác định viết thành chỉ dẫn văn bản | AI mỗi lần lại hiểu lại từ đầu, có thể sai | Đóng gói thành scripts/, thực thi mà không nạp vào |
| Các file references tham chiếu lẫn nhau | AI cần nhảy nhiều bước để lấy thông tin | Tất cả references liên kết trực tiếp từ SKILL.md |
| Nội dung SKILL.md và references trùng lặp | Lãng phí token, khi cập nhật có thể không nhất quán | Thông tin chỉ tồn tại ở một nơi |

---

## VI. Chiều thiết kế thứ hai: Cho AI bao nhiêu quyền tự do?
> Vị trí trong khung: Tầng thứ hai — Chiều thứ hai

Biết được thông tin nên đặt ở đâu, nên ràng buộc thế nào rồi, câu hỏi tiếp theo là: **AI làm gì, script làm gì?**

AI rất giỏi trong việc hiểu ngữ nghĩa, sinh văn bản, làm những công việc sáng tạo. Nhưng nó không giỏi kiểm soát định dạng chính xác, ràng buộc độ dài, quy tắc đặt tên — những "thao tác dễ vỡ" này.

### 6.1 Ba nấc quyền tự do

Skill-creator dùng một **quang phổ quyền tự do** để xử lý sự không đồng đều này (xem hình khung ở chương III):

> Think of Codex as exploring a path: a narrow bridge with cliffs needs specific guardrails (low freedom), while an open field allows many routes (high freedom).

**Tự do cao** (chỉ dẫn văn bản): khi có nhiều phương pháp đều khả thi, quyết định phụ thuộc vào ngữ cảnh, dùng heuristic để dẫn dắt.

**Tự do trung bình** (pseudo-code/script có tham số): có best practice nhưng cho phép biến hóa, cấu hình ảnh hưởng đến hành vi.

**Tự do thấp** (script cụ thể, ít tham số): thao tác dễ vỡ dễ sai, tính nhất quán cực kỳ quan trọng, phải tuân theo một trình tự cụ thể.

Logic cốt lõi:

```
Tác vụ càng dễ vỡ (dễ sai) → quyền tự do càng thấp → dùng script để khóa chặt
Tác vụ càng linh hoạt (nhiều phương án đều đúng) → quyền tự do càng cao → dùng văn bản để dẫn dắt
```

### 6.2 Phân bổ quyền tự do của chính skill-creator

| Tác vụ | Quyền tự do | Cách hiện thực |
|------|--------|---------|
| Hiểu nhu cầu người dùng và đặt câu hỏi | Cao | Hướng dẫn văn bản trong SKILL.md |
| Lập kế hoạch cấu trúc nội dung skill | Trung bình | Template + đề xuất mô hình kiểu trắc nghiệm |
| Khởi tạo cấu trúc thư mục | **Thấp** | Script `init_skill.py` |
| Sinh openai.yaml | **Thấp** | Script `generate_openai_yaml.py` |
| Viết nội dung SKILL.md | Cao | Hướng dẫn nguyên tắc + gợi ý cách viết |
| Kiểm tra kết quả cuối cùng | **Thấp** | Script `quick_validate.py` |

### 6.3 Hai hướng sai lầm

**Sai lầm 1: Cho tác vụ dễ vỡ quá nhiều quyền tự do**

```markdown
# Sai
Hãy sinh một file openai.yaml, bao gồm display_name và short_description.

# Hậu quả: short_description có thể vượt quá giới hạn 64 ký tự, chữ hoa/thường có thể không nhất quán
```

Cách làm của Skill-creator: dùng script `generate_openai_yaml.py` để khóa chặt định dạng. AI chỉ cung cấp giá trị tham số, script đảm bảo output tuân thủ.

**Sai lầm 2: Cho tác vụ sáng tạo quá nhiều ràng buộc**

```markdown
# Sai
Đoạn thứ nhất bắt buộc bắt đầu bằng "hôm qua", đoạn thứ hai bắt buộc chứa "về bản chất", đoạn cuối kết thúc bằng "từ từ thôi".

# Hậu quả: văn bản sinh ra giống như trò chơi điền từ
```

Cách làm của Skill-creator: đưa ra tỷ lệ cấu trúc (tầng bối cảnh ≤30%, tầng nguyên lý 30-40%), nhưng không khóa chặt từ ngữ cụ thể.

### 6.4 Tiêu chí phán đoán

Hai câu hỏi:
1. **Làm sai thì hậu quả nghiêm trọng đến mức nào?** — càng nghiêm trọng → quyền tự do càng thấp
2. **Có bao nhiêu cách làm "đúng"?** — càng nhiều → quyền tự do càng cao

### 6.5 Hiện thực quyền tự do thấp: Ba script của skill-creator

Hiểu được quang phổ quyền tự do rồi, ta sẽ hiểu vì sao skill-creator có ba script — chúng chính là hiện thực cụ thể của "quyền tự do thấp" (mối quan hệ tương tác giữa các script xem hình khung ở chương III).

**`init_skill.py` (đảm bảo đầu vào, 398 dòng)**

Công cụ khung (scaffolding) để khởi tạo thư mục skill mới, tương tự như `create-react-app` đối với dự án React:

```bash
scripts/init_skill.py <skill-name> --path <output-directory> \
  [--resources scripts,references,assets] [--examples] \
  [--interface key=value]
```

Chức năng cốt lõi:
- Tạo thư mục skill
- Sinh template SKILL.md với các placeholder TODO (TODO là "bài điền vào chỗ trống" cho Codex xem)
- Gọi `generate_openai_yaml.py` để sinh `agents/openai.yaml` (truyền vào display_name, short_description, default_prompt do AI sinh ra thông qua `--interface key=value`)
- Tùy chọn tạo các thư mục con `scripts/`, `references/`, `assets/`
- Tùy chọn thêm file ví dụ (`--examples`)
- Tích hợp sẵn `normalize_skill_name()` tự động chuẩn hóa bất kỳ đầu vào nào của người dùng thành hyphen-case

Ví dụ sử dụng:
```bash
scripts/init_skill.py my-skill --path skills/public
scripts/init_skill.py my-skill --path skills/public --resources scripts,references
scripts/init_skill.py my-skill --path skills/public --resources scripts --examples
```

**`generate_openai_yaml.py` (đảm bảo định dạng, 226 dòng)**

Chuyên phụ trách việc sinh và cập nhật `agents/openai.yaml`:

- Đọc tên skill từ frontmatter của SKILL.md
- Tự động chuyển hyphen-case thành Title Case (`my-cool-skill` → `My Cool Skill`)
- Tích hợp sẵn từ điển viết tắt (GH, MCP, API, v.v. giữ nguyên chữ hoa) và từ điển thương hiệu (openai → OpenAI)
- Tự động sinh `short_description` 25-64 ký tự
- Hỗ trợ `--interface key=value` để ghi đè bất kỳ trường nào

```bash
scripts/generate_openai_yaml.py <path/to/skill-folder> --interface key=value
```

**`quick_validate.py` (đảm bảo đầu ra, 102 dòng)**

"Nhân viên kiểm định chất lượng" sau khi skill được tạo:

```bash
scripts/quick_validate.py <path/to/skill-folder>
```

Nội dung kiểm tra:
- SKILL.md có tồn tại không
- Định dạng YAML frontmatter có hợp lệ không
- `name`: có phải hyphen-case không, ≤ 64 ký tự, không có dấu gạch nối liên tiếp/ở đầu cuối
- `description`: có tồn tại không, không có dấu ngoặc nhọn, ≤ 1024 ký tự
- Chỉ cho phép 5 khóa frontmatter là `name`, `description`, `license`, `allowed-tools`, `metadata`

### 6.6 Chuỗi đảm bảo chất lượng

Ba script tạo thành một **chuỗi đảm bảo mang tính xác định**, kẹp lấy bước sáng tạo ở giữa:

```
init_skill.py (đảm bảo đầu vào)
  Chuẩn hóa tên + tạo cấu trúc thư mục + sinh template
  → Đảm bảo điểm xuất phát đúng
       ↓
  AI viết một cách sáng tạo (quyền tự do cao)
  → Nội dung SKILL.md, references, scripts tùy chỉnh
       ↓
quick_validate.py (đảm bảo đầu ra)
  Kiểm tra định dạng frontmatter + quy tắc đặt tên + ràng buộc độ dài
  → Đảm bảo điểm kết thúc tuân thủ
```

Nhận thức then chốt: script là loại "thực thi mà không nạp vào" — **chi phí token bằng không**. Bạn có thể đóng gói bất kỳ logic xác định phức tạp nào vào script mà không cần lo nó chiếm ngữ cảnh. Đây chính là lý do vì sao skill-creator giao toàn bộ những thao tác vụn vặt nhưng dễ vỡ như chuyển đổi tên gọi (từ điển viết tắt, từ điển thương hiệu), ràng buộc độ dài (25-64 ký tự), kiểm tra định dạng cho script.

### 6.7 Cái gì nên đóng gói thành script?

```
Kết quả mỗi lần thực thi phải giống nhau     → Script
Liên quan đến định dạng/độ dài chính xác     → Script
Liên quan đến chuyển đổi quy tắc đặt tên      → Script
Cần khớp với quy tắc kiểm tra                → Script
Cùng một đoạn code lần nào cũng phải viết lại → Script

Cần hiểu ngữ cảnh                            → Chỉ dẫn văn bản
Có nhiều cách làm hợp lý                     → Chỉ dẫn văn bản
Cần phán đoán sáng tạo                       → Chỉ dẫn văn bản
```

Script đôi khi vẫn cần được Codex đọc (để vá lỗi hoặc thích ứng môi trường), nhưng phần lớn thời gian chúng là loại "thực thi mà không nạp vào".

---

## VII. Triển khai: Quy trình tạo skill sáu bước
> Vị trí trong khung: Tầng thứ ba

Có nguyên tắc và kiến trúc ở phần trước rồi, cuối cùng skill-creator đưa ra một quy trình tạo skill sáu bước, biến tư duy thiết kế thành các bước thao tác có thể thực thi (xem hình khung ở chương III).

### 7.0 Quy tắc đặt tên

Trước khi bắt đầu, hãy xác định tên gọi trước:

- Chỉ dùng chữ thường, số và dấu gạch nối; chuẩn hóa tên do người dùng cung cấp thành hyphen-case (ví dụ "Plan Mode" → `plan-mode`)
- Tên ≤ 64 ký tự
- Ưu tiên dùng cụm từ ngắn gọn, bắt đầu bằng động từ để mô tả hành động
- Khi cần dùng tên tool làm namespace (ví dụ `gh-address-comments`, `linear-address-issue`)
- Tên thư mục skill trùng khớp hoàn toàn với tên skill

### 7.1 Step 1: Hiểu skill — dùng ví dụ cụ thể để tạo đồng thuận

> Skip this step only when the skill's usage patterns are already clearly understood.

Để tạo một skill hiệu quả, trước tiên phải hiểu rõ **các ví dụ sử dụng cụ thể**. Những hiểu biết này có thể đến từ ví dụ do người dùng cung cấp, cũng có thể đến từ ví dụ được sinh ra rồi được người dùng xác nhận.

Lấy việc xây dựng skill image-editor làm ví dụ, có thể hỏi người dùng:

- "Skill image-editor nên hỗ trợ những chức năng gì? Chỉnh sửa, xoay, còn gì nữa không?"
- "Có thể cho một vài ví dụ về việc sử dụng skill này không?"
- "Tôi có thể nghĩ đến việc người dùng sẽ nói 'khử mắt đỏ cho tấm ảnh này' hoặc 'xoay tấm hình này'. Còn cách sử dụng nào khác không?"
- "Người dùng sẽ nói câu gì để kích hoạt skill này?"

**Lưu ý**: Đừng hỏi quá nhiều câu cùng một lúc. Hỏi câu quan trọng nhất trước, rồi hỏi tiếp theo nhu cầu.

**Dấu hiệu hoàn thành**: đã có nhận thức rõ ràng về những chức năng mà skill nên hỗ trợ.

### 7.2 Step 2: Lập kế hoạch nội dung skill có thể tái sử dụng

Với mỗi ví dụ cụ thể, hãy làm hai phân tích:
1. Nếu làm việc này từ đầu, thì cần những gì?
2. Trong đó cái nào sẽ được dùng đi dùng lại nhiều lần?

Những thứ được dùng đi dùng lại → đóng gói thành scripts/references/assets.

skill-creator đưa ra ba trường hợp phân tích điển hình:

**Trường hợp 1: skill `pdf-editor`** (người dùng hỏi "xoay giúp tôi cái PDF này")
- Xoay PDF lần nào cũng phải viết lại cùng một đoạn code
- → đóng gói thành `scripts/rotate_pdf.py`

**Trường hợp 2: skill `frontend-webapp-builder`** (người dùng hỏi "làm giúp tôi một todo app" hoặc "làm một dashboard theo dõi số bước chân")
- Viết webapp frontend lần nào cũng cần cùng một bộ code mẫu HTML/React
- → đóng gói thành thư mục template `assets/hello-world/`

**Trường hợp 3: skill `big-query`** (người dùng hỏi "hôm nay có bao nhiêu người dùng đăng nhập?")
- Truy vấn BigQuery lần nào cũng phải khám phá lại schema và mối quan hệ của các bảng
- → đóng gói thành `references/schema.md`

**Dấu hiệu hoàn thành**: đã liệt kê danh sách tất cả các tài nguyên có thể tái sử dụng cần đưa vào (scripts, references, assets).

### 7.3 Step 3: Khởi tạo skill

> When creating a new skill from scratch, always run the `init_skill.py` script.

Ở đây dùng từ "always" — không phải "khuyến nghị", mà là "luôn luôn". Lý do:
- Cấu trúc thư mục do script sinh ra đảm bảo tuân thủ quy chuẩn
- Nhắc nhở TODO trong template đảm bảo không bỏ sót trường bắt buộc
- Ràng buộc định dạng của `agents/openai.yaml` (độ dài trường, quy tắc dấu ngoặc) nếu viết tay rất dễ sai

Đây là **ứng dụng trực tiếp của nguyên tắc quyền tự do thấp**: khởi tạo là một thao tác dễ vỡ, dùng script để loại bỏ khả năng sai sót.

Sau khi khởi tạo:
- Tùy chỉnh SKILL.md và thêm tài nguyên theo nhu cầu
- Nếu đã dùng `--examples`, thay thế hoặc xóa các file placeholder

### 7.4 Step 4: Chỉnh sửa skill

Đây là bước cốt lõi nhất, chia làm hai giai đoạn:

#### Giai đoạn một: Hiện thực tài nguyên có thể tái sử dụng trước

Bắt đầu từ các tài nguyên đã lập kế hoạch ở Step 2: hiện thực các file `scripts/`, `references/`, `assets/`.

Lưu ý:
- Bước này có thể cần đầu vào của người dùng (ví dụ skill `brand-guidelines` cần người dùng cung cấp tài nguyên thương hiệu)
- Script mới thêm **bắt buộc phải được kiểm thử bằng cách chạy thực tế**, đảm bảo không có bug và output đúng như mong đợi
- Nếu có nhiều script tương tự nhau, chỉ cần kiểm thử một mẫu đại diện để tạo sự tin tưởng
- Nếu đã dùng `--examples`, xóa các file placeholder không cần thiết. Chỉ tạo những thư mục tài nguyên thực sự cần

#### Giai đoạn hai: Cập nhật SKILL.md

**Cách viết Frontmatter**:

```yaml
---
name: skill-name
description: >-
  Mô tả skill làm gì + cụ thể khi nào dùng.
  Đặt tất cả thông tin "when to use" ở đây, đừng đặt trong body.
---
```

**Cách viết Body**:

Viết chỉ dẫn thao tác cho một instance Codex khác. Bao gồm những thông tin hữu ích nhưng không hiển nhiên đối với Codex: kiến thức thủ tục (procedural knowledge), chi tiết lĩnh vực, cách sử dụng các tài nguyên có thể tái sử dụng.

Thống nhất dùng **giọng mệnh lệnh/nguyên thể**.

### 7.5 Step 5: Kiểm tra skill

```bash
scripts/quick_validate.py <path/to/skill-folder>
```

Kiểm tra định dạng YAML frontmatter, các trường bắt buộc, quy tắc đặt tên. Nếu không đạt thì sửa xong rồi chạy lại.

### 7.6 Step 6: Lặp lại (iterate)

> After testing the skill, users may request improvements. Often this happens right after using the skill, with fresh context of how the skill performed.

Workflow lặp lại:
1. Sử dụng skill trên tác vụ thực tế
2. Phát hiện chỗ nào bị đuối hoặc kém hiệu quả
3. Tìm ra SKILL.md hoặc tài nguyên đóng gói nên được cập nhật thế nào
4. Triển khai thay đổi và kiểm thử lại

Một skill tốt không được viết xong trong một lần. Skill laotou-thought-style mà skill-creator tạo ra, sau lần sinh đầu tiên đã lặp lại `short_description` và `default_prompt` của `openai.yaml` — từ mô tả chung chung trở thành chỉ dẫn thao tác chính xác hơn.

---

## VIII. Tổng kết

Quay lại câu hỏi ban đầu: làm sao viết ra một skill tốt?

Nhìn lại toàn bộ khung:

```
Ràng buộc căn bản: Súc tích (Chương IV)
 ├── Đặt thông tin ở đâu? → Phân tầng ba cấp, nạp theo nhu cầu (Chương V)
 ├── Cho AI bao nhiêu quyền tự do? → Thao tác dễ vỡ dùng script khóa chặt, công việc sáng tạo dùng văn bản dẫn dắt (Chương VI)
 └── Triển khai thế nào? → Quy trình sáu bước: Hiểu → Lập kế hoạch → Khởi tạo → Chỉnh sửa → Kiểm tra → Lặp lại (Chương VII)
```

**Skill là viết chỉ dẫn cho AI, chứ không phải cho người. Dùng ít token nhất, ở đúng tầng, đưa cho AI những ràng buộc chính xác nhất, để nó tự do phát huy trong phạm vi ranh giới.**
