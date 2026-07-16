# Agent Self-Evolution: Bốn loại vòng lặp khép kín trong quá trình tự tiến hóa của Agent

## Lời mở đầu: Vì sao Agent cần tự tiến hóa?

Ở các chương trước, chúng ta đã tìm hiểu về ReAct, Reflection, MCP, Agent Skills, hệ thống ghi nhớ (memory), kỹ thuật ngữ cảnh (context engineering) và Agentic RL. Khi đặt tất cả những năng lực này cạnh nhau, một câu hỏi rất tự nhiên sẽ nảy sinh: nếu mỗi lần Agent (trí tuệ thông minh) đều phải mò mẫm lại từ những sai lầm tương tự, mỗi lần đều phải tra cứu lại cùng một tài liệu, mỗi lần đều phải viết lại cùng một bộ các bước thao tác, thì thực ra nó vẫn chưa thật sự "trưởng thành".

**Agent Self-Evolution** (Agent tự tiến hóa) bàn đúng về vấn đề này: liệu Agent có thể tích lũy lại các tín hiệu như quỹ đạo tương tác, phản hồi nhiệm vụ, sự chỉnh sửa của người dùng, kết quả thực thi công cụ, kinh nghiệm tập thể, và để những thứ được tích lũy đó tiếp tục ảnh hưởng đến hành vi về sau hay không?

"Tự tiến hóa" (self-evolution) ở đây không đồng nghĩa với bộ nhớ dài hạn đơn thuần, cũng không phải là việc cài đặt thủ công một plugin. Một định nghĩa thực dụng hơn là:

> Agent tự tiến hóa là một hệ thống Agent có khả năng, dựa trên chính quỹ đạo tương tác, phản hồi nhiệm vụ hay tín hiệu môi trường của nó, liên tục cập nhật ngữ cảnh (context), bộ nhớ (memory), kỹ năng (skill), công cụ, quy trình làm việc (workflow), mã nguồn hoặc tham số mô hình, và để những cập nhật đó ảnh hưởng đến hiệu quả thực hiện nhiệm vụ trong tương lai.

Định nghĩa này có ba điểm mấu chốt.

1. **Được thúc đẩy bởi kinh nghiệm**: cập nhật đến từ nhiệm vụ thực tế, phản hồi khi thực thi, sự sửa lỗi của người dùng, kết quả đánh giá hoặc tín hiệu môi trường, chứ không phải cấu hình thủ công một lần rồi thôi.
2. **Có hiệu lực liên tục**: cập nhật sẽ đi vào bộ nhớ, kho kỹ năng, quy trình làm việc, mã nguồn hoặc tham số, và tiếp tục phát huy tác dụng trong các nhiệm vụ tương lai.
3. **Có thể đánh giá, có thể khôi phục (rollback)**: mức độ tự tiến hóa càng mạnh thì càng cần đến bộ đánh giá (evaluator), ghi chép phiên bản, sandbox, kiểm soát quyền và cơ chế rollback.

Chương này được tổ chức theo hướng "vòng lặp tiến hóa được đặt ở đâu, đối tượng tiến hóa là gì", chia thành bốn loại: **vòng lặp ngữ cảnh tích hợp sẵn**, **vòng lặp tài sản hóa kỹ năng**, **vòng lặp giám sát bên ngoài hoặc trí tuệ tập thể**, và **vòng lặp tự sửa đổi tham số, mã nguồn hoặc quy trình làm việc**. Mỗi loại chọn ra một số phương pháp hoặc kỹ thuật tiêu biểu, **tổng cộng 10 dự án**.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra10-figures/agent-evolution.png" alt="Agent Self-Evolution: four closed loops and ten representative projects" width="88%"/>
  <p><strong>Hình 1</strong> Agent Self-Evolution: Bốn loại vòng lặp khép kín và 10 dự án tiêu biểu. Ràng buộc cắt ngang: bất kỳ lộ trình tiến hóa nào cũng phải đặt trên nền tảng đánh giá, phiên bản, rollback, quyền hạn và quản trị chuỗi cung ứng.</p>
</div>

## Tổng quan bốn loại vòng lặp khép kín

Trước tiên hãy dùng một bảng để thiết lập góc nhìn toàn cục. Bảng này có thể giúp mọi người nhanh chóng nhận định: một hệ thống tự tiến hóa rốt cuộc đang thay đổi cái gì, dựa vào phản hồi nào để thay đổi, và sau khi thay đổi thì ảnh hưởng đến nhiệm vụ tiếp theo như thế nào.


| Loại             | Phương pháp hoặc kỹ thuật tiêu biểu                                                                                                                                                                   | Điểm cốt lõi                                                                                                     |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Vòng lặp ngữ cảnh tích hợp sẵn        | 1. [Hermes Agent](https://github.com/NousResearch/hermes-agent) <br>2. [Agent Zero](https://github.com/agent0ai/agent-zero)                                                         | 1. Hermes đưa bộ nhớ, truy hồi phiên và tạo kỹ năng vào ngay bên trong bản thân Agent. <br>2. Agent Zero hình thành ngữ cảnh liên tục thông qua dự án, lịch sử trò chuyện, công cụ và các sub-agent.                              |
| Vòng lặp tài sản hóa kỹ năng        | 1. [Darwin Skill](https://github.com/alchaincyf/darwin-skill) <br>2. [JiuwenClaw](https://github.com/openJiuwen-ai/jiuwenclaw) <br>3. [EvoSkill](https://github.com/sentient-agi/EvoSkill) | 1. Darwin Skill xem SKILL.md như một tài sản có thể đánh giá, có thể rollback. <br>2. JiuwenClaw tối ưu Skill dựa trên phản hồi trong lúc chạy (runtime). <br>3. EvoSkill sinh ra, kiểm thử và giữ lại các biến thể kỹ năng từ quỹ đạo thất bại. |
| Vòng lặp giám sát bên ngoài hoặc trí tuệ tập thể    | 1. [Ultron](https://github.com/modelscope/ultron) <br>2. [OpenSpace](https://github.com/HKUDS/OpenSpace) <br>3. [SkillClaw](https://github.com/AMAP-ML/SkillClaw)                          | 1. Ultron chưng cất kinh nghiệm cá nhân thành bộ nhớ tập thể, kỹ năng và Harness. <br>2. OpenSpace, với tư cách dịch vụ tiến hóa bên ngoài, duy trì phả hệ phiên bản của kỹ năng. <br>3. SkillClaw hợp nhất kinh nghiệm xuyên phiên, xuyên thiết bị, xuyên người dùng thành kỹ năng dùng chung.   |
| Vòng lặp tự sửa đổi tham số, mã nguồn hoặc quy trình làm việc | 1. [OpenClaw-RL](https://github.com/Gen-Verse/OpenClaw-RL) <br>2. [Agent Lightning](https://github.com/microsoft/agent-lightning)                                                   | 1. OpenClaw-RL chuyển phản hồi hội thoại thực tế thành tín hiệu RL/OPD bất đồng bộ. <br>2. Agent Lightning tách rời việc thực thi Agent khỏi việc huấn luyện RL.                             |


Bốn loại này, đi từ trên xuống dưới, thường đồng nghĩa với giới hạn năng lực ngày càng cao và rủi ro kỹ thuật cũng ngày càng lớn. Sự tiến hóa ở tầng ngữ cảnh và tầng kỹ năng dễ kiểm toán (audit) và rollback nhất; tầng trí tuệ tập thể bắt đầu liên quan đến lưu trữ dùng chung, quyền hạn và quyền riêng tư; còn việc tự sửa đổi ở tầng tham số, mã nguồn hoặc quy trình làm việc là gần nhất với "chính sách đã thay đổi", nhưng cũng phụ thuộc nhiều nhất vào bộ đánh giá đáng tin cậy và môi trường thực thi được cô lập.

## I. Vòng lặp ngữ cảnh tích hợp sẵn: Để Agent học ngay trong vòng lặp chính của chính nó

Tư tưởng cốt lõi của vòng lặp ngữ cảnh tích hợp sẵn là: không trực tiếp sửa đổi tham số mô hình, mà để Agent ghi kinh nghiệm vào bộ nhớ, văn bản phản tư (reflection), chỉ mục phiên hoặc thư mục kỹ năng, rồi lấy ra dùng lại trong nhiệm vụ kế tiếp.

Ưu điểm của loại hệ thống này là nhẹ, triển khai nhanh, khả năng diễn giải cao. Chúng không cần cụm huấn luyện (training cluster), cũng không cần hạ tầng RL trực tuyến phức tạp. Nhược điểm cũng rất rõ ràng: nếu năng lực mô hình nền tảng không đủ, thì chỉ dựa vào ngữ cảnh và bộ nhớ sẽ rất khó vượt qua giới hạn; nếu không có cơ chế quản trị, thì bộ nhớ sai và phản tư sai cũng sẽ liên tục làm ô nhiễm hành vi về sau.

### 1. Hermes Agent: Đưa vòng lặp học tập vào ngay bên trong bản thân Agent

Địa chỉ dự án: [https://github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)

Hermes Agent là dự án kỹ thuật đáng được triển khai riêng nhất trong loại này. Nó hướng đến lộ trình [self-improving AI agent](https://hermes-agent.nousresearch.com/docs) (AI agent tự cải thiện): thế mạnh không nằm ở một chức năng đơn lẻ nào đó, mà ở chỗ nhồi nhiều cơ chế "sẽ trở nên vững chắc trong quá trình sử dụng" vào cùng một vòng lặp hội thoại chính. Vài mảnh ghép điển hình bao gồm: bộ nhớ bền vững do chính Agent chủ động duy trì (`MEMORY.md` / `USER.md` v.v.), truy hồi xuyên phiên được kích hoạt theo nhu cầu (SQLite + FTS5, kết hợp với tóm tắt), tải kỹ năng theo kiểu tiệm tiến cùng việc kỹ năng được tạo ra hoặc viết lại ngay trong lúc sử dụng, và về mặt an toàn thì có phê duyệt lệnh cùng cô lập terminal đa backend, v.v.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra10-figures/hermes-banner.png" alt="Hermes Agent project banner" width="88%"/>
  <p><strong>Hình 2</strong> Hermes Agent</p>
</div>

Có thể hình dung sự tự tiến hóa của Hermes như một vòng lặp khép kín nhẹ, bám sát vòng lặp chính: **nhiệm vụ người dùng → vòng lặp chính của Hermes → công cụ / terminal / cổng tin nhắn (message gateway) → phản hồi khi thực thi và chỉnh sửa của người dùng**, đầu ra tách nhánh ghi vào **bộ nhớ được biên tập (curated memory)** và **Skills**; bộ nhớ thông qua **session_search** cùng với kỹ năng đều dồn vào **ngữ cảnh của lượt kế tiếp**, rồi lại quay về vòng lặp chính.

Ý nghĩa của lộ trình này nằm ở chỗ "học tập" diễn ra ngay trên đường gọi thực tế hằng ngày. Chẳng hạn sau khi kết thúc một lần gỡ lỗi phức tạp, Agent có thể ghi thói quen dùng lệnh, ràng buộc môi trường, nguyên nhân thất bại và lộ trình khắc phục cuối cùng vào bộ nhớ hoặc kỹ năng; về sau khi gặp lại vấn đề cùng loại, nó không cần mò mẫm cả không gian tìm kiếm từ con số không.

Tuy nhiên, đối tượng cập nhật chính của Hermes vẫn là **ngữ cảnh, bộ nhớ và kỹ năng**, chứ không phải cập nhật trọng số mô hình theo kiểu trực tuyến. Vì vậy nó phù hợp hơn khi được xem như "hình thái kỹ thuật trưởng thành của vòng lặp ngữ cảnh tích hợp sẵn", chứ không phải một hệ thống tự huấn luyện ở tầng trọng số.

### 2. Agent Zero: Đưa bộ nhớ dự án, công cụ động và sub-agent vào vòng lặp chính

Địa chỉ dự án: [https://github.com/agent0ai/agent-zero](https://github.com/agent0ai/agent-zero)  

Agent Zero phù hợp để xếp vào **vòng lặp ngữ cảnh tích hợp sẵn**. Nó hướng đến các nhiệm vụ thực tế với tư cách một **[dynamic, organic agentic framework](https://github.com/agent0ai/agent-zero)** (framework agentic linh hoạt, hữu cơ): **không biến Agent thành một script mục đích đơn**, mà cấp cho nó một môi trường ở cấp hệ điều hành có thể sử dụng (terminal, thực thi mã, tệp, tự động hóa trình duyệt, v.v.), và cho phép **tạo ra hoặc viết lại công cụ theo nhu cầu** trong quá trình nhiệm vụ tiến triển. Điều này không liên quan đến việc tinh chỉnh trọng số ngoại tuyến, mà giống với "trạng thái bàn làm việc" tích lũy dần theo dự án và phiên hơn.

Về mặt kỹ thuật, có vài khối lắp ghép liên quan trực tiếp đến vòng lặp khép kín: **Project** cô lập không gian làm việc, phần mô tả, bộ nhớ, khóa (key), cơ sở tri thức, kho mã (repo) và preset mô hình, có thể clone repo vào ngữ cảnh dự án độc lập; **Skills** tuân theo quy ước `SKILL.md` mở, có thể bật theo phạm vi toàn cục, theo dự án hoặc theo phiên hiện tại; **Agent Profiles** dùng để chuyển đổi hành vi, ghi đè prompt, chuỗi công cụ (toolchain) và cấu hình mô hình mà không cần viết lại toàn bộ hệ thống; về mặt cộng tác thì nhấn mạnh việc Agent cấp trên tạo ra các **subordinate agents** (agent cấp dưới), để mỗi cấp dưới tự giữ một ngữ cảnh nhỏ hơn rồi báo cáo kết quả về. Nó còn hỗ trợ MCP, plugin cùng bố cục `prompts/`, `tools/` có thể kiểm tra được; phía Web UI còn có Universal Canvas, chú thích trình duyệt cùng các năng lực trực quan hóa hướng đến sự đồng tiến hóa giữa người và máy (xem chi tiết trong tài liệu chính thức).

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra10-figures/agent-zero-banner.png" alt="Agent Zero framework banner" width="88%"/>
  <p><strong>Hình 3</strong> Agent Zero</p>
</div>

Dưới cùng một góc nhìn, đường hồi lưu chính có thể nén thành một chuỗi: **nhiệm vụ và mục tiêu → Project → Chat → Skills / MCP / plugin → agent cấp dưới → phản hồi và đầu ra → Project** (các năng lực sản phẩm như canvas, chú thích trình duyệt ở đây được lược bỏ).

Agent Zero và Hermes ở cùng một cấp độ: cả hai đều là Agent host phía cá nhân hoặc nhóm theo kiểu "làm dày ngữ cảnh theo mức độ sử dụng", chứ không phải một thuật toán đơn lẻ từ bài báo nào đó. Agent Zero còn mang màu sắc chồng chéo của tài sản hóa kỹ năng (`SKILL.md`) và phân công đa Agent, nhưng chương này vẫn đặt nhãn chính của nó là vòng lặp ngữ cảnh tích hợp sẵn, bởi vì sự tăng trưởng dễ kiểm toán nhất trước tiên diễn ra ngay trong ranh giới dự án, trạng thái phiên, cấu hình công cụ và dấu vết thực thi thực tế.

## II. Vòng lặp tài sản hóa kỹ năng: Để kinh nghiệm lắng đọng thành Skill có thể tái sử dụng

Tư tưởng cốt lõi của vòng lặp tài sản hóa kỹ năng là: hiện thực hóa (externalize) kinh nghiệm thành các tài sản kỹ năng có thể đọc, có thể quản lý phiên bản, có thể kiểm thử, có thể di chuyển (migrate).

Trong hệ sinh thái Agent Skills, `SKILL.md` không chỉ là một tệp mô tả, nó có thể trở thành bộ nhớ thủ tục (procedural memory) của Agent: khi nào dùng, thực thi thế nào, gọi những script nào, tuân thủ những ràng buộc nào, xử lý ngoại lệ ra sao. Điểm mấu chốt của việc tự tiến hóa ở tầng kỹ năng chính là làm cho những kỹ năng này không còn chỉ dựa vào việc bảo trì thủ công, mà có thể được đánh giá, viết lại, xác minh và rollback.

### 3. Darwin Skill: Tối ưu Skill bằng cơ chế đánh giá và ratchet

Địa chỉ dự án: [https://github.com/alchaincyf/darwin-skill](https://github.com/alchaincyf/darwin-skill)

Dưới đây ta bắt đầu loại vòng lặp thứ hai từ Darwin Skill: `SKILL.md` được xem một cách rõ ràng như một tài sản có thể đo lường, có thể lặp cải tiến. Chủ trương của Darwin Skill về tối ưu SKILL có thể tóm gọn thành "**tối ưu Agent Skills của bạn như huấn luyện một mô hình**", phương pháp luận đối chiếu trực tiếp với [autoresearch](https://github.com/karpathy/autoresearch) của Karpathy: dùng mục tiêu định lượng được để thúc đẩy thay đổi, và dùng **bánh cóc (ratchet)** để chỉ giữ lại những phần cải thiện xác minh được, còn lại thì **git revert** đi, tránh để baseline âm thầm xấu đi theo thời gian.

Về cơ chế, nó được tách thành vài nguyên tắc cứng: **tài sản đơn có thể chỉnh sửa** (mỗi lần chỉ sửa một `SKILL.md` cần tối ưu), **đánh giá kép** (phía cấu trúc thiên về phân tích tĩnh, phía hiệu quả thì phải chạy trên các prompt kiểm thử để xem đầu ra), **chấm điểm độc lập** (dùng sub-agent chấm điểm, giảm bớt tình trạng "tự sửa tự chấm"), **con người trong vòng lặp (human-in-the-loop)** (tạm dừng giữa các giai đoạn, hiển thị diff và biến động điểm số rồi mới tiếp tục). Phía xác minh hiệu quả sẽ dùng đến tập kiểm thử kiểu như `test-prompts.json`; tổng điểm 100 đến từ **rubric tám chiều**, trong đó tỷ lệ phân bổ điểm giữa chiều cấu trúc và chiều hiệu quả được viết rất rõ ràng (trong phần hiệu quả thì "biểu hiện thực đo" có trọng số cao nhất).

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra10-figures/daerwin-banner.png" alt="Darwin Skill project banner" width="88%"/>
  <p><strong>Hình 4</strong> Darwin Skill</p>
</div>

Hình dưới nhất quán với **Core Loop** của nó, tương ứng với Evaluate → Improve → Validate → Confirm → Keep or Revert.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra10-figures/darwin-loop.png" alt="Darwin Skill core evaluation loop" width="88%"/>
  <p><strong>Hình 5</strong> Darwin Skill: Vòng lặp khép kín Evaluate → Improve → Validate → Confirm → Keep or Revert</p>
</div>

Quy trình này có thể tách thành năm bước:

1. **Evaluate**: Phân tích cấu trúc và xác minh hiệu quả đối với `SKILL.md` mục tiêu, tổng hợp thành điểm có trọng số tám chiều.
2. **Improve**: Tìm ra chiều có điểm thấp nhất, sinh ra một vòng viết lại có trọng tâm và đề xuất thay đổi.
3. **Validate**: Kiểm tra lại trên tập prompt kiểm thử (như `test-prompts.json`) hoặc nhiệm vụ thực tế tương đương.
4. **Confirm**: Hiển thị diff và biến động điểm số, để con người xác nhận có bước sang vòng tiếp theo hay Skill tiếp theo hay không.
5. **Keep or Revert**: Nếu tổng điểm mới cao hơn mức tốt nhất hiện tại thì giữ lại, ngược lại thì rollback; ratchet đảm bảo baseline hiệu quả đơn điệu không giảm.

Rubric tám chiều chia tổng điểm thành nhiều chiều (tỷ lệ phân bổ điểm giữa phía cấu trúc và phía hiệu quả xem hình minh họa); "biểu hiện thực đo" có trọng số cao nhất trong nhóm chiều hiệu quả.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra10-figures/darwin-score.png" alt="Darwin Skill eight-dimension rubric" width="88%"/>
  <p><strong>Hình 6</strong> Darwin Skill: Hệ thống chấm điểm tám chiều (tổng điểm 100)</p>
</div>

Thiết kế quan trọng nhất của Darwin Skill không phải là "tự động sửa kỹ năng", mà là **chỉ để những cải thiện xác minh được ở lại**. Bất kỳ toolchain nào tương thích với host Agent Skill / `SKILL.md` mở đều có thể tích hợp, bao gồm Claude Code, Codex, OpenClaw, Trae, CodeBuddy v.v.; trong đó Darwin Skill đóng vai trò một bộ tối ưu kỹ năng tương đối độc lập.

### 4. JiuwenClaw: Tự tiến hóa kỹ năng ở thời điểm chạy (runtime)

Địa chỉ dự án: [https://github.com/openJiuwen-ai/jiuwenclaw](https://github.com/openJiuwen-ai/jiuwenclaw)  
Tài liệu tự tiến hóa kỹ năng: [https://github.com/openJiuwen-ai/jiuwenclaw/blob/develop/docs/en/SkillSelfEvolution.md](https://github.com/openJiuwen-ai/jiuwenclaw/blob/develop/docs/en/SkillSelfEvolution.md)

Khẩu hiệu của JiuwenClaw hướng đến việc sử dụng đồng hành dài hạn là "Understands You. Evolves With You": **Autonomous Evolution** (tiến hóa tự chủ) tức là liên tục cải thiện các kỹ năng liên quan dựa trên phản hồi khi người dùng không hài lòng hoặc khi thực thi bị lỗi. Về mặt hiện thực, nó dựa vào **SkillCallOperator** để thống lĩnh việc đọc, ghi và hợp nhất, cùng với **SignalDetector** (dựa trên quy tắc, **không gọi LLM**) giám sát các manh mối sửa lỗi trong kết quả công cụ và cách diễn đạt của người dùng; những sự kiện có thể quy về kỹ năng hiện tại được giao cho **SkillEvolutionManager** điều phối việc quét và sinh ra, còn **SkillOptimizer** khi cần thay đổi sẽ gọi LLM để viết ra các mục tiến hóa (evolution entry), đưa vào **`evolutions.json`** trước, rồi vào thời điểm thích hợp thì **solidify** (cố kết) hợp nhất trở lại **`SKILL.md`**, cũng có thể kích hoạt thủ công qua **`/evolve`**. Các tín hiệu thuộc loại thất bại có xu hướng được ghi vào mục kiểu **Troubleshooting**, còn sự chỉnh sửa của người dùng thì thường được sắp xếp thành **Examples** hơn. Mỗi kỹ năng có thư mục riêng dưới workspace (đường dẫn điển hình dạng `~/.jiuwenclaw/workspace/agent/skills/<skill_name>/`), trong cấu hình có thể bật **`evolution_auto_scan`**, sau khi kết thúc một lượt công cụ cũng có thể bổ sung bản ghi tiến hóa ở nền (background).

Xét về mặt phân loại, nó vẫn phù hợp nhất để xếp vào **vòng lặp tài sản hóa kỹ năng**: phần gia tăng bền vững rơi vào `SKILL.md` và `evolutions.json`, chứ không phải một bản tóm tắt hội thoại dùng một lần.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra10-figures/jiuwenclaw.png" alt="JiuwenClaw overview and skill evolution" width="88%"/>
  <p><strong>Hình 7</strong> JiuwenClaw: Tổng quan dự án và bối cảnh tự tiến hóa kỹ năng ở runtime</p>
</div>

Một pipeline khớp với **Evolution flow** trong tài liệu của kho mã: **hội thoại người dùng hoặc thực thi công cụ → SignalDetector → SkillEvolutionManager → SkillOptimizer → `evolutions.json` → solidify → SkillCallOperator** (cái sau thống nhất việc đọc ghi và tải **`SKILL.md`** đã hợp nhất trước khi gọi ở lượt kế tiếp).

So với Darwin Skill: Darwin nhấn mạnh hơn vào bộ đánh giá đầy đủ kiểu "như huấn luyện mô hình", chấm điểm tám chiều, ratchet và con người trong vòng lặp mạnh mẽ; còn JiuwenClaw nhúng sự tiến hóa vào ngay trong hội thoại thường ngày và vòng lặp công cụ, được thúc đẩy bởi tín hiệu, mục tiến hóa vào JSON trước rồi mới cố kết vào tài liệu, trọng tâm kỹ thuật nằm ở **khả dụng trực tuyến (online availability) và lặp cải tiến nhanh**. Nó cũng không tránh khỏi chồng chéo với **vòng lặp ngữ cảnh tích hợp sẵn** (ngay trong cùng một lượt hội thoại đã phải phát hiện phản hồi), nhưng chương này vẫn lấy tài sản kỹ năng làm nhãn chính, vì phần gia tăng dài hạn có thể kiểm toán chủ yếu nằm trong tệp kỹ năng và bản ghi tiến hóa.

### 5. EvoSkill: Tiến hóa biến thể kỹ năng từ quỹ đạo thất bại

Địa chỉ dự án: [https://github.com/sentient-agi/EvoSkill](https://github.com/sentient-agi/EvoSkill)  
Địa chỉ bài báo: [https://arxiv.org/abs/2603.02766](https://arxiv.org/abs/2603.02766)

EvoSkill hướng đến việc tự động khám phá và cải thiện kỹ năng cho **coding agent** (agent viết mã): trong bối cảnh kho mã, nó mở rộng tư tưởng kiểu [GEPA](https://github.com/sentient-agi/gepa-plus) "dựa vào phản hồi để sửa một chỗ prompt" thành sự lặp cải tiến trên **toàn bộ agent program** (có thể đồng thời đề xuất nhiều biến dị (mutation) của **skill** và **system prompt**), chấm điểm trên **tập xác thực để riêng ra (held-out validation set)**, mỗi vòng các ứng viên xuất sắc sẽ đi vào vòng tiếp theo dưới dạng **trạng thái chương trình hoàn toàn mới**, chứ không chỉ vá đắp tại chỗ một đoạn văn.

Vòng lặp tự phục vụ được tách trong tài liệu thành năm phần: **Base Agent** dùng cấu hình tốt nhất hiện tại chạy các mẫu trong bộ chuẩn (benchmark); **Proposer** đối chiếu các ví dụ thất bại để đề xuất thay đổi có trọng tâm; **Generator** viết ra tệp kỹ năng mới hoặc viết lại system prompt; **Evaluator** chấm điểm phiên bản mới trên dữ liệu held-out; **Frontier** duy trì Top-N bộ chương trình có biểu hiện tốt nhất, và quản lý phiên bản dưới dạng nhánh Git **`program/*`** (còn có nhãn **`frontier/*`** đánh dấu các thành viên tiền tuyến), tiện cho việc kiểm toán và rollback qua `evoskill diff`, `evoskill skills`. Về mặt kỹ thuật thì được điều khiển qua **`evoskill init`** / **`evoskill run`**, tập dữ liệu phần lớn là CSV có đáp án chuẩn, có thể viết `.evoskill/task.md` theo nhiệm vụ, chế độ thực thi hỗ trợ chạy trên máy cục bộ, Docker hoặc sandbox từ xa Daytona; host harness bao phủ Claude Code, OpenCode, OpenHands, Goose, Codex CLI v.v. Trong cấu hình tiến hóa có thể chọn **`skill_only`** hoặc **`prompt_only`**, việc chấm điểm hỗ trợ nhiều loại **scorer** như khớp theo quy tắc, LLM-as-judge, script v.v.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra10-figures/evoskill-framework.png" alt="EvoSkill discovery framework" width="88%"/>
  <p><strong>Hình 8</strong> EvoSkill: Framework đồng tiến hóa kỹ năng và prompt hướng đến coding Agent</p>
</div>

Khác với Darwin Skill thiên về "một SKILL.md duy nhất + ratchet tám chiều", JiuwenClaw thiên về tín hiệu hội thoại trực tuyến, EvoSkill gần với "tiến hóa nguyên gói cấu hình Agent do bộ chuẩn ngoại tuyến (offline benchmark) dẫn dắt" hơn: bằng chứng thành bại đến từ đánh giá lặp lại được, đầu ra là thư mục skills có thể sao chép cùng các artifact kiểu `program.yaml`, phù hợp cho các nhóm muốn thu hoạch trợ lý viết mã từ mô hình đa dụng thành một **pipeline chuyên biệt**.

## III. Vòng lặp giám sát bên ngoài hoặc trí tuệ tập thể: Để kinh nghiệm chảy qua các Agent

Hai loại vòng lặp đầu chủ yếu phục vụ cho một Agent đơn lẻ. Đến vòng lặp giám sát bên ngoài hoặc trí tuệ tập thể, kinh nghiệm bắt đầu chảy xuyên phiên, xuyên thiết bị, xuyên người dùng, xuyên Agent.

Giá trị của loại hệ thống này rất trực tiếp: một cái hố mà một Agent đã sa vào thì không nên để tất cả các Agent phải sa lại lần nữa; một quy trình làm việc mà một nhóm đã xác thực thì không nên chỉ nằm trong lịch sử cục bộ của một người nào đó. Cái giá của nó cũng rất trực tiếp: kinh nghiệm dùng chung cần đến quyền hạn, khử nhận dạng (desensitization), quản trị phiên bản, cổng kiểm soát chất lượng (quality gate) và kiểm toán.

### 6. Trí tuệ tập thể Ultron: Memory Hub, Skill Hub, Harness Hub

Địa chỉ dự án: [https://github.com/modelscope/ultron](https://github.com/modelscope/ultron)  
Địa chỉ demo: [https://writtingforfun-ultron.ms.show/dashboard](https://writtingforfun-ultron.ms.show/dashboard)

Ultron là một **self-evolving collective intelligence** (trí tuệ tập thể tự tiến hóa) hướng đến Agent đa dụng: chưng cất kinh nghiệm rải rác trong từng phiên thành **tri thức tập thể dễ truy hồi và tái sử dụng**. Ba năng lực nổi bật với bên ngoài là: **bộ nhớ tập thể phân tầng**, **kỹ năng tập thể có thể tự tiến hóa theo bằng chứng**, và **bản thiết kế (blueprint) Harness có thể chia sẻ** (nhập một lần cả bộ persona, bộ nhớ và tổ hợp kỹ năng). Phía máy chủ **Trajectory Hub** tiếp nhận các quỹ đạo `.jsonl`: phân đoạn nhiệm vụ, chỉ số `ms_agent.trajectory`, khử trùng lặp bằng vân tay tăng dần (incremental fingerprint) và trích xuất ở nền; các quỹ đạo chất lượng cao còn có thể dùng cho **SFT / tự huấn luyện** (có thể kết nối với các framework huấn luyện như [Twinkle](https://github.com/modelscope/twinkle) Workbench), từ đó **giảm chi phí gọi mô hình đầu cuối** ở phía định tuyến (routing). Đây là một vòng lặp khép kín dài hạn "quỹ đạo vào kho → bộ nhớ và kỹ năng sinh trưởng → phân phối bản thiết kế → nhiều quỹ đạo hồi lưu về", chứ không phải chỉ dựa vào chồng chất prompt dài.

Phần dưới sẽ trình bày tách ra theo ba khối năng lực bảng điều khiển: Memory Hub, Skill Hub, Harness Hub; tổng thể vẫn thuộc loại vòng lặp thứ ba: kinh nghiệm rời khỏi phiên đơn máy, đi vào tầng dùng chung có thể quản trị.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra10-figures/ultron-banner.png" alt="Ultron collective intelligence banner" width="88%"/>
  <p><strong>Hình 9</strong> Ultron: Bộ nhớ tập thể, kỹ năng tập thể và Harness dùng chung</p>
</div>

**Memory Hub** đảm nhận vai trò "kho sự kiện có thể triệu hồi" ở phía tập thể. Các điểm năng lực bao gồm: phân tầng **HOT / WARM / COLD** và tái cân bằng theo số lần trúng (hit count), truy hồi ngữ nghĩa bằng vector kết hợp trọng số theo tầng, các cấp tóm tắt **L0 / L1 / Full** (truy hồi trước tiên trả về tóm tắt ngắn để tiết kiệm token, kéo toàn văn khi cần), tự động phân loại theo kiểu khi tải lên, hợp nhất các vector gần trùng và sắp xếp hàng loạt, mở rộng truy hồi theo ý định (intent), suy giảm độ nóng theo hàm mũ theo thời gian, cùng phát hiện **PII** tiếng Trung và tiếng Anh dựa trên **Presidio** rồi khử nhận dạng trước khi vào kho. Hình dưới tương ứng với góc nhìn "duyệt và truy hồi bộ nhớ phân tầng" trong bảng điều khiển.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra10-figures/ultron-memory-hub.png" alt="Ultron Memory Hub" width="88%"/>
  <p><strong>Hình 10</strong> Ultron Memory Hub: Duyệt và truy hồi bộ nhớ tập thể phân tầng</p>
</div>

Về phía **Skill Hub**, vừa có các gói kỹ năng nội bộ được **chưng cất** từ bộ nhớ điểm nóng (hot memory), vừa đối tiếp với các chỉ mục bên ngoài như **ModelScope Skill Hub** để thực hiện khám phá thống nhất. Lộ trình **Skill self-evolution** nhấn mạnh: các bộ nhớ liên quan trước tiên hình thành cụm ngữ nghĩa (semantic cluster), rồi **kết tinh (crystallize)** thành kỹ năng quy trình nhiều bước, sau khi bằng chứng tích lũy thì **tái kết tinh**; kết hợp với **provenance-grounded verification** (xác minh dựa trên nguồn gốc) và **structure-score upgrade gate** (cổng nâng cấp theo điểm cấu trúc), tránh để kỹ năng sau khi tiến hóa **thụt lùi** về điểm cấu trúc. Nó cũng có quan hệ kế thừa với các tư tưởng tiến hóa kỹ năng tập thể như [SkillClaw](https://github.com/AMAP-ML/SkillClaw), có thể đọc đối chiếu với mục SkillClaw ở phần sau của chương.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra10-figures/ultron-skill-hub.png" alt="Ultron Skill Hub" width="88%"/>
  <p><strong>Hình 11</strong> Ultron Skill Hub: Cổng thống nhất cho kỹ năng chưng cất nội bộ và kỹ năng chỉ mục bên ngoài</p>
</div>

**Harness Hub** biến "persona hội thoại dùng xong bỏ" thành tài sản có thể quản lý phiên bản: có thể phát hành một **Agent profile** hoàn chỉnh (persona, bộ nhớ, kỹ năng hợp nhất một thể) thành bản thiết kế nhập được bằng mã ngắn, và hỗ trợ **đồng bộ hai chiều** giữa workspace và máy chủ để tiện tiếp nối trên nhiều thiết bị. Nó còn cung cấp nhiều **Soul presets** (tổ hợp vai trò, MBTI, cung hoàng đạo, v.v.) để lắp ghép tài nguyên workspace. Hình dưới tương ứng với bối cảnh lắp ghép và phát hành Harness trong bảng điều khiển.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra10-figures/ultron-harness-hub.png" alt="Ultron Harness Hub" width="88%"/>
  <p><strong>Hình 12</strong> Ultron Harness Hub: Lắp ghép persona cùng bộ nhớ, kỹ năng và phát hành bản thiết kế nhập được</p>
</div>

Phía tích hợp chủ yếu hướng đến các nhà phát triển "đã có sẵn instance dịch vụ Ultron, chỉ cần nối host Agent vào"; trong Showcase có đưa ra các script nhập một lần cho OpenClaw, Hermes, Nanobot v.v. Loại như **Darwin Skill** vốn là bộ tối ưu thiên về kiểm toán tài sản SKILL đơn máy và ratchet, cũng có thể bổ sung cho tầng bộ nhớ / kỹ năng dùng chung của Ultron: cái trước giữ vững tính giải thích được của thay đổi, cái sau khuếch đại bán kính tái sử dụng xuyên phiên, xuyên instance.

Nếu thu hẹp vấn đề vào một câu: Ultron cố gắng giảm nhẹ ba loại nỗi đau Session-bound (bị ràng buộc bởi phiên) điển hình (kinh nghiệm chết theo phiên, chi phí sa hố lặp lại bị khuếch đại theo số instance, persona không di chuyển được). Nó không phải để thay thế một framework Agent nào đó, mà chồng thêm lên trên nó một tầng hạ tầng tập thể **có thể truy hồi, có thể tiến hóa, có thể phân phối**.

### 7. OpenSpace: Dịch vụ tiến hóa bên ngoài và phả hệ phiên bản

Địa chỉ dự án: [https://github.com/HKUDS/OpenSpace](https://github.com/HKUDS/OpenSpace)

OpenSpace hướng đến hình thái "host Agent + engine tiến hóa kỹ năng gắn ngoài": lộ trình tích hợp phổ biến là cấu hình **OpenSpace MCP** vào các toolchain như Claude Code, Codex, OpenClaw, nanobot, Cursor, để phía OpenSpace đảm nhận việc khám phá kỹ năng, giám sát thực thi và đồng bộ đám mây, và lấy **self-evolving engine** (engine tự tiến hóa) làm câu chuyện sản phẩm: **thất bại có thể kích hoạt sửa chữa, mô hình thành công có thể lắng đọng, chất lượng kỹ năng có thể được đo lường liên tục**.

Về cơ chế tiến hóa, nó xem kỹ năng như đối tượng lặp cải tiến liên tục, phân biệt ba tuyến chính: **FIX** (vá tại chỗ phần mô tả lỗi thời hoặc mất hiệu lực), **DERIVED** (phái sinh biến thể tăng cường hoặc chuyên biệt từ kỹ năng cha), **CAPTURED** (trích xuất một quy trình hoàn toàn mới, có thể tái sử dụng từ một lần thực thi thành công). Nguồn kích hoạt bao gồm **Post-Execution Analysis** (phân tích sau thực thi) khi nhiệm vụ kết thúc, **Tool Degradation** (tuần tra suy giảm công cụ) khi tỷ lệ thành công của công cụ tụt xuống, và **Metric Monitor** quét các chỉ số sức khỏe theo chu kỳ, từ đó hình thành vòng lặp khép kín "thực thi → bằng chứng → bản vá hoặc phiên bản kỹ năng mới". Chiến lược viết lại thiên về **thay đổi tối thiểu ở cấp diff**, thất bại thì có thể tự động thử lại; về phía phiên bản dùng **DAG** để ghi phả hệ và so sánh, **Dashboard** cục bộ có thể duyệt đồ thị tiến hóa kỹ năng và lịch sử thực thi. Registry đám mây hỗ trợ khả năng nhìn thấy công khai, theo nhóm hoặc riêng tư, và đối tiếp với truy hồi cộng đồng, nhập một lần.

Về chiều tập thể, nó nhất quán với câu chuyện của loại vòng lặp thứ ba: **cải thiện của một Agent đơn lẻ có thể lan tỏa đến các instance khác thông qua registry dùng chung**, tương đương với việc đặt hạ tầng tiến hóa ra bên ngoài host, để nhiều đầu tái sử dụng cùng một kho kỹ năng và stack giám sát.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra10-figures/openspace-framework.png" alt="OpenSpace framework" width="88%"/>
  <p><strong>Hình 13</strong> OpenSpace: Dịch vụ tiến hóa bên ngoài và phả hệ phiên bản</p>
</div>

Cách dùng điển hình là nhiều instance hoặc nhiều người dùng đồng thời tích hợp vào cùng một dịch vụ OpenSpace hoặc cùng một kho kỹ năng đám mây: quỹ đạo và bằng chứng thực thi được tụ hợp ở phía engine, phiên bản kỹ năng lặp cải tiến theo tín hiệu chất lượng và giữ lại phả hệ, rồi được kéo về từng host qua MCP hoặc CLI, tránh để mỗi Agent tự duy trì riêng một thư mục kỹ năng tĩnh mong manh.

### 8. SkillClaw: Để kỹ năng tiến hóa xuyên phiên, xuyên thiết bị, xuyên người dùng

Địa chỉ dự án: [https://github.com/AMAP-ML/SkillClaw](https://github.com/AMAP-ML/SkillClaw)  
Bài báo: [https://arxiv.org/abs/2604.08377](https://arxiv.org/abs/2604.08377)

SkillClaw hướng đến **collective skill evolution** (tiến hóa kỹ năng tập thể): lắng đọng cách dùng trong các hội thoại thực tế thành **`SKILL.md`** có thể tái sử dụng, và chia sẻ cùng một vòng lặp tiến hóa qua nhiều phiên của một người dùng, nhiều thiết bị, nhiều instance Agent cho đến giữa các thành viên trong nhóm, nhấn mạnh tự động tiến hóa, khử trùng lặp và kiểm chứng chất lượng xuyên phiên (có thể cấu hình **PRM** theo nhu cầu). Phía host tương thích với các chuỗi phổ biến như Hermes, OpenClaw, Codex, Claude Code, QwenPaw và bất kỳ API tương thích OpenAI nào.

Kiến trúc chia thành hai phần: **Client Proxy** là proxy API cục bộ, chặn **`/v1/chat/completions`** và **`/v1/messages`**, ghi lại các sản phẩm của phiên và duy trì kho kỹ năng cục bộ mà không làm gián đoạn nhịp hội thoại, chỉ cần bật riêng nó là hoàn tất việc tích hợp. **Evolve Server** (`evolve_server`) là tùy chọn: đọc dữ liệu phiên từ lưu trữ dùng chung, sinh ra hoặc viết lại kỹ năng rồi ghi trở lại; engine tiến hóa hỗ trợ hai chế độ **`workflow`** (pipeline LLM ba giai đoạn cố định Summarize → Aggregate → Execute) và **`agent`** (trực tiếp sửa kỹ năng dựa trên workspace của OpenClaw). Client và server chỉ gặp nhau qua cùng một tầng lưu trữ (**thư mục cục bộ / OSS / S3**), do đó cá nhân có thể chạy proxy trước, rồi gắn dịch vụ tiến hóa trên máy đơn hoặc từ xa; còn với kịch bản nhóm thì để nhiều client trỏ về cùng một lưu trữ và do một dịch vụ tiến hóa thống nhất tiêu hóa các quỹ đạo.

Hermes trên nhiều máy của cùng một người dùng, hoặc host Agent của nhiều thành viên trong cùng một nhóm, có thể ghi các phiên của mình vào lưu trữ dùng chung rồi để phía server hợp nhất, khử trùng lặp và phân phối kỹ năng phiên bản mới trở lại từng instance, đây chính là hình thái triển khai thực tế điển hình của "xuyên phiên, xuyên thiết bị, xuyên người dùng" trong loại vòng lặp thứ ba.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra10-figures/skillclaw-framework.png" alt="SkillClaw architecture" width="88%"/>
  <p><strong>Hình 14</strong> SkillClaw: Framework tổng thể của Client Proxy, lưu trữ dùng chung và Evolve Server</p>
</div>

Lấy ví dụ: đoạn gỡ lỗi React tích lũy trong môi trường gia đình, đoạn vận hành Kubernetes tích lũy trong môi trường văn phòng, đoạn phân tích log do OpenClaw chạy trên máy chủ, nếu thiếu một tầng kỹ năng dùng chung thì sẽ cô lập với nhau; sau khi tích hợp SkillClaw, chúng đi vào cùng một vòng lặp tiến hóa và kiểm chứng, rồi hồi lưu về các host khác nhau theo nhu cầu, tránh việc trả lại cùng một chi phí thử-sai lặp lại trong nội bộ fleet (đội máy).

## IV. Tự sửa đổi tham số, mã nguồn hoặc quy trình làm việc: Để bản thân chính sách thay đổi

Loại cuối cùng là vòng lặp mạnh nhất và cũng nặng nhất: hệ thống không chỉ sửa ngữ cảnh, bộ nhớ hay kỹ năng, mà bắt đầu sửa tham số mô hình, mã nguồn Agent, tô-pô quy trình làm việc hoặc thuật toán ứng viên.

Giới hạn trên của loại hệ thống này rất cao, vì nó có thể thật sự thay đổi bản thân chính sách; nhưng rủi ro cũng lớn nhất, bởi vì một khi bộ đánh giá có lỗ hổng, thiết kế phần thưởng không hợp lý, sandbox không đủ, thì hệ thống có thể học được cách "chiều lòng bộ đánh giá" thay vì thật sự mạnh lên.

### 9. OpenClaw-RL: Từ phản hồi hội thoại thực tế đến RL trực tuyến

Địa chỉ dự án: [https://github.com/Gen-Verse/OpenClaw-RL](https://github.com/Gen-Verse/OpenClaw-RL)  
Báo cáo kỹ thuật: [https://arxiv.org/abs/2603.10165](https://arxiv.org/abs/2603.10165)

OpenClaw-RL chồng hai tuyến chính trong cùng một stack: **Track 1 (Personal Agent)** nhúng mô hình chính sách tự lưu trữ (self-hosted policy model) vào [OpenClaw](https://openclaw.ai), đối ngoại vẫn giữ API tương thích OpenAI, chặn hội thoại đa lượt trực tuyến và chuyển tương tác thành tín hiệu huấn luyện, Serving và tối ưu ở nền không cản trở lẫn nhau; **Track 2 (General Agentic RL)** thì mở rộng hạ tầng RL bất đồng bộ đến các môi trường nặng hơn như terminal, GUI, SWE, gọi công cụ, đẩy cao quy mô rollout song song.

Vòng lặp khép kín được tách theo README thành bốn vòng lặp bất đồng bộ: **Agent serving**, **rollout collection**, **PRM / Judge evaluation**, **policy training** (bao gồm **LoRA**). Phía Rollout chia tin nhắn hội thoại thành **main-line** (tuyến chính) có thể huấn luyện và **side** (tuyến phụ), và xem phản hồi của người dùng, môi trường hoặc công cụ ở nhịp kế tiếp như **next-state** tự nhiên; Judge / PRM chấm điểm bất đồng bộ, có thể biểu quyết đa số theo nhu cầu rồi mới xếp hàng. **Binary RL (GRPO)** dùng mô hình phần thưởng quá trình (process reward model) kết hợp phản hồi trạng thái kế tiếp để tạo phần thưởng vô hướng theo từng bước; **OPD (On-Policy Distillation)** dùng văn bản hindsight để dựng giáo viên tăng cường, đưa ra tín hiệu định hướng qua khoảng cách log-probability giữa thầy và trò; ngoài ra còn có công thức **Combine / Hybrid** hợp nhất cả hai. Triển khai có thể chọn đa card cục bộ, [Tinker](https://thinkingmachines.ai/tinker/), [Fireworks AI](https://fireworks.ai/) v.v.; cũng có thể nối RL head vào OpenClaw của riêng bạn qua extension chính thức.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra10-figures/openclaw-rl-framework.png" alt="OpenClaw-RL asynchronous loops" width="88%"/>
  <p><strong>Hình 15</strong> OpenClaw-RL: Kiến trúc bất đồng bộ Serving · Rollout · PRM/Judge · Training</p>
</div>

Loại hệ thống này gần nhất với "vừa trò chuyện vừa huấn luyện", nhưng một khi phản hồi của người dùng thực tế đi vào đường gradient, thì bắt buộc phải triển khai song song sự đồng thuận có hiểu biết (informed consent), chu kỳ lưu giữ dữ liệu, khử nhận dạng quyền riêng tư, đối phó với việc gaming phần thưởng (reward gaming) và rollback an toàn; nếu không thì chỉ số kỹ thuật rất dễ bị rủi ro tuân thủ và pháp lý bù trừ hết.

### 10. Agent Lightning: Tách rời việc thực thi Agent khỏi việc huấn luyện RL

Địa chỉ dự án: [https://github.com/microsoft/agent-lightning](https://github.com/microsoft/agent-lightning)  
Bài báo: [https://arxiv.org/abs/2508.03680](https://arxiv.org/abs/2508.03680)  
Tài liệu: [https://microsoft.github.io/agent-lightning/](https://microsoft.github.io/agent-lightning/)

Agent Lightning thu gọn chuỗi huấn luyện thành ba khối: phía **Agent / Tracer** chỉ cần xâm nhập tối thiểu (gọi tường minh `agl.emit_xxx()` hoặc để tracer tự động bắt prompt, gọi công cụ và phần thưởng), sự kiện được chuẩn hóa thành **span** có cấu trúc; **LightningStore** cache thống nhất định nghĩa nhiệm vụ, snapshot tài nguyên và quỹ đạo, tương đương một mặt đọc-ghi duy nhất ở phía huấn luyện; **Trainer + Algorithm** đọc span, sản xuất tài nguyên đã cập nhật (như prompt tinh chỉnh hoặc trọng số chính sách), rồi hồi lưu về đầu suy luận (inference). Khe thuật toán vừa có thể cắm RL, vừa có thể cắm tối ưu prompt tự động, tinh chỉnh có giám sát v.v. theo cùng một hợp đồng span.

Tài liệu chính thức nhấn mạnh có thể cùng tồn tại với các bộ điều phối như LangChain, OpenAI Agent SDK, AutoGen, CrewAI, Microsoft Agent Framework, cũng có thể dùng trên chuỗi gọi OpenAI trần không có framework; trong kịch bản đa Agent có thể **chọn lọc** chỉ tính gradient cho một số vai trò, mà không cần viết lại toàn bộ tô-pô thành một đồ thị huấn luyện duy nhất.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/Extra-Chapter/images/Extra10-figures/agent-lightning-architecture.svg" alt="Agent Lightning architecture" width="90%"/>
  <p><strong>Hình 16</strong> Agent Lightning: LightningStore tách rời việc thực thi trực tuyến và thuật toán ngoại tuyến</p>
</div>

So với OpenClaw-RL: cái trước bám sát OpenClaw Serving + stack huấn luyện RL bất đồng bộ; Agent Lightning giống một "xe buýt quỹ đạo" đa dụng hơn, mục tiêu là để **Trainer / Algorithm** có thể cắm-rút, còn phía host thì chú trọng giữ lại ngữ nghĩa framework vốn có.

## Các vấn đề kỹ thuật then chốt của hệ thống tự tiến hóa

Bất kể là loại vòng lặp khép kín nào, khi thật sự triển khai đều sẽ gặp bốn vấn đề chung.

### 1. Bộ đánh giá quan trọng hơn bộ sinh (generator)

Ảo tưởng nguy hiểm nhất của hệ thống tự tiến hóa là: "chỉ cần biết tự động sửa thì sẽ tự động tốt lên." Thực tế lại hoàn toàn ngược lại. Sự sửa đổi tự động không có bộ đánh giá chỉ là tự động tạo ra bất định.

Ratchet của Darwin Skill, worker xác minh của SkillClaw, cổng nâng cấp theo điểm cấu trúc của Ultron, PRM/Judge của OpenClaw-RL, phân tích sau thực thi và tuần tra chỉ số của OpenSpace, tất cả đều nói lên cùng một đạo lý: **cốt lõi của tự tiến hóa không phải là thay đổi, mà là chứng minh rằng thay đổi không làm hệ thống xấu đi.**

### 2. Khả năng rollback là hạ tầng của tự tiến hóa

Ưu thế của tầng kỹ năng nằm ở tính có thể quản lý phiên bản một cách tự nhiên: `SKILL.md` có thể diff, có thể git commit, có thể rollback. Ngược lại, cập nhật ở tầng tham số tuy tiềm năng lớn hơn, nhưng giải thích và rollback đều nặng hơn. Một nguyên tắc thực dụng là:

```
Việc gì giải quyết được ở tầng bộ nhớ trước thì đừng vội sửa kỹ năng;
Việc gì giải quyết được ở tầng kỹ năng trước thì đừng vội sửa mã nguồn;
Việc gì giải quyết được ở tầng mã nguồn/quy trình làm việc trước thì đừng vội cập nhật trọng số trực tuyến.
```

### 3. Kinh nghiệm dùng chung cần được quản trị

Ultron, OpenSpace, SkillClaw đều đưa kinh nghiệm cá nhân vào tầng dùng chung. Việc dùng chung sẽ mang lại hiệu ứng mạng, nhưng cũng mang lại rủi ro ô nhiễm. Một kỹ năng sai, một mẩu bộ nhớ chứa thông tin riêng tư, một đoạn phiên bị ô nhiễm bởi prompt injection, nếu đi vào kho dùng chung, thì có thể ảnh hưởng đến Agent của cả nhóm.

Vì vậy, vòng lặp trí tuệ tập thể ít nhất cần bốn năng lực mặc định: phân tầng quyền hạn, khử nhận dạng PII, xác minh ứng viên, kiểm toán phiên bản.

### 4. An toàn chuỗi cung ứng kỹ năng không thể bù đắp về sau

Agent Skills đã trở thành định dạng đóng gói năng lực để tái sử dụng xuyên hệ sinh thái. Một kỹ năng có thể bao gồm `SKILL.md`, script, tài liệu tham khảo, template, phụ thuộc từ xa và chỉ thị thực thi. Nó vừa giống tài liệu, vừa giống gói phần mềm, lại còn có thể ảnh hưởng đến bộ nhớ dài hạn và hành vi gọi công cụ.

Điều này có nghĩa là chợ kỹ năng (skill market) và việc chia sẻ kỹ năng không chỉ được nhìn ở khía cạnh "viết có tốt hay không", mà còn phải xem: có đọc các đường dẫn nhạy cảm hay không, có thực thi lệnh nguy hiểm hay không, có tải script từ xa hay không, có ghi secret vào đầu ra hay không, có tìm cách làm ô nhiễm kỹ năng hoặc bộ nhớ khác hay không.

## Lộ trình thực hành: Từ vòng lặp nhẹ tiến tới tự tiến hóa mạnh

Nếu bạn muốn đưa self-evolution vào dự án Agent của mình, có thể triển khai theo bốn giai đoạn.

### Giai đoạn một: Làm vòng lặp ngữ cảnh tích hợp sẵn trước

Mục tiêu là để Agent có được năng lực trưởng thành cơ bản nhất: biết ghi lại sở thích người dùng, biết tổng kết kinh nghiệm thất bại, biết truy hồi các phiên lịch sử, biết lắng đọng nhiệm vụ phức tạp thành các bước thao tác.

Dự án có thể tham khảo là [Hermes Agent](https://github.com/NousResearch/hermes-agent). Trọng tâm của giai đoạn này không phải là nhồi lịch sử vô hạn trở lại prompt, mà là để ngữ cảnh dự án, lịch sử phiên, trạng thái công cụ và thông tin tự tham chiếu dần hình thành một cơ chế ổn định.

### Giai đoạn hai: Lắng đọng kinh nghiệm thành Skill

Khi kinh nghiệm bắt đầu lặp lại, thì nên nâng từ tầng bộ nhớ lên tầng kỹ năng. Kỹ năng có cấu trúc hơn bộ nhớ, cũng dễ di chuyển sang các Agent khác hơn.

Dự án có thể tham khảo là [Darwin Skill](https://github.com/alchaincyf/darwin-skill). Nếu bạn đã đang dùng các host hỗ trợ `SKILL.md` như OpenClaw, Nanobot, Hermes, Codex hoặc Claude Code, thì vòng lặp tài sản hóa kỹ năng thường là bước có tỷ suất đầu tư trên hiệu quả (ROI) cao nhất.

### Giai đoạn ba: Đưa vào tầng dùng chung và trí tuệ tập thể

Khi nhiều người dùng, nhiều thiết bị, nhiều Agent đều đang tích lũy kinh nghiệm, thì cần một tầng dùng chung để khử trùng lặp, xác minh, hợp nhất và phân phối.

Các dự án có thể tham khảo bao gồm [Ultron](https://github.com/modelscope/ultron), [OpenSpace](https://github.com/HKUDS/OpenSpace) và [SkillClaw](https://github.com/AMAP-ML/SkillClaw). Giai đoạn này cần thiết kế trước quyền riêng tư, quyền hạn và kiểm toán vào ngay từ đầu.

### Giai đoạn bốn: Thận trọng thử tự sửa đổi tham số, mã nguồn hoặc quy trình làm việc

Chỉ khi bộ đánh giá, sandbox, quản trị phiên bản và cơ chế rollback đủ trưởng thành, thì mới thích hợp thử các vòng lặp nặng hơn như [OpenClaw-RL](https://github.com/Gen-Verse/OpenClaw-RL), [Agent Lightning](https://github.com/microsoft/agent-lightning) vốn nối tín hiệu đánh giá vào việc cập nhật tham số hoặc chính sách có cấu trúc.

Nếu bạn chỉ muốn để trợ lý cá nhân hiểu bạn hơn, thì có thể không cần đến RL trực tuyến; còn nếu bạn muốn Agent tối ưu việc sử dụng công cụ, thao tác GUI, sửa lỗi SWE hoặc lựa chọn quy trình làm việc trong một lượng lớn nhiệm vụ thực tế, thì việc tự sửa đổi ở tầng tham số hoặc quy trình làm việc mới dần trở nên cần thiết.

## Tổng kết

Agent Self-Evolution không phải là một kỹ thuật đơn lẻ, mà là một tập hợp các vòng lặp khép kín từ nhẹ đến nặng:

```
Tự tiến hóa ngữ cảnh/bộ nhớ
  → Tự tiến hóa tài sản kỹ năng
    → Tự tiến hóa kinh nghiệm tập thể
      → Tự tiến hóa tham số, mã nguồn hoặc quy trình làm việc
```

**Hermes Agent** và **Agent Zero** đại diện cho vòng lặp ngữ cảnh tích hợp sẵn; **Darwin Skill**, **JiuwenClaw**, **EvoSkill** đại diện cho vòng lặp tài sản hóa kỹ năng; **Ultron**, **OpenSpace**, **SkillClaw** đại diện cho việc lắng đọng và phân phối ở phía trí tuệ tập thể; **OpenClaw-RL** nối tương tác trực tuyến vào RL hoặc OPD, còn **Agent Lightning** thì dùng LightningStore để tách rời việc quan sát thực thi khỏi Trainer có thể cắm-rút.

Với đa số nhóm kỹ thuật, lộ trình ổn thỏa nhất không phải là làm cập nhật trọng số trực tuyến ngay từ đầu, mà là làm tốt trước các vòng lặp bộ nhớ và kỹ năng "có thể giải thích, có thể đánh giá, có thể rollback". Cái thật sự hạn chế việc triển khai tự tiến hóa thường không phải là mô hình chưa đủ thông minh, mà là đánh giá, sandbox, quyền hạn, quản trị phiên bản và an toàn chuỗi cung ứng còn chưa đủ vững chắc.

## Tài liệu tham khảo

[1] Hermes Agent. [https://github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)  
[2] Agent Zero. [https://github.com/agent0ai/agent-zero](https://github.com/agent0ai/agent-zero)  
[3] Darwin Skill. [https://github.com/alchaincyf/darwin-skill](https://github.com/alchaincyf/darwin-skill)  
[4] JiuwenClaw. [https://github.com/openJiuwen-ai/jiuwenclaw](https://github.com/openJiuwen-ai/jiuwenclaw)  
[5] EvoSkill. [https://github.com/sentient-agi/EvoSkill](https://github.com/sentient-agi/EvoSkill)  
[6] Ultron. [https://github.com/modelscope/ultron](https://github.com/modelscope/ultron)  
[7] OpenSpace. [https://github.com/HKUDS/OpenSpace](https://github.com/HKUDS/OpenSpace)  
[8] SkillClaw. [https://github.com/AMAP-ML/SkillClaw](https://github.com/AMAP-ML/SkillClaw)  
[9] OpenClaw-RL. [https://github.com/Gen-Verse/OpenClaw-RL](https://github.com/Gen-Verse/OpenClaw-RL)  
[10] Agent Lightning. [https://github.com/microsoft/agent-lightning](https://github.com/microsoft/agent-lightning)
