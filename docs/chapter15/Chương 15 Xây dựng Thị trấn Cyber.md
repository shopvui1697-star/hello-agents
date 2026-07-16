# Chương 15 Xây dựng Thị trấn Cyber

Trong chương này, chúng ta sẽ khám phá một hướng đi hoàn toàn mới: **kết hợp công nghệ agent với game engine để xây dựng một thị trấn AI tràn đầy sức sống**.

Bạn còn nhớ những NPC sống động trong "The Sims" hay "Animal Crossing" không? Chúng có tính cách, ký ức và các mối quan hệ xã hội của riêng mình. Thị trấn Cyber trong chương này sẽ là một dự án tương tự, nhưng khác với các trò chơi truyền thống, NPC của chúng ta có "trí thông minh" thực sự - chúng có thể hiểu được cuộc trò chuyện của người chơi, ghi nhớ các tương tác trong quá khứ và phản ứng khác nhau dựa trên mức độ thiện cảm. Thị trấn Cyber trong chương này bao gồm các tính năng cốt lõi sau:

**(1) Hệ thống Đối thoại NPC Thông minh**: Người chơi có thể trò chuyện bằng ngôn ngữ tự nhiên với NPC, và NPC sẽ phản hồi dựa trên thiết lập vai trò và ký ức của mình.

**(2) Hệ thống Ký ức**: NPC có ký ức ngắn hạn và dài hạn, có thể ghi nhớ lịch sử tương tác với người chơi.

**(3) Hệ thống Thiện cảm**: Thái độ của NPC đối với người chơi thay đổi theo các tương tác, từ người lạ đến quen biết, từ thân thiện đến thân mật.

**(4) Tương tác Game hóa**: Người chơi có thể di chuyển tự do trong một cảnh văn phòng phong cách pixel 2D và tương tác với các NPC khác nhau.

**(5) Hệ thống Ghi log Thời gian thực**: Tất cả các cuộc trò chuyện và tương tác đều được ghi lại để dễ dàng gỡ lỗi và phân tích.

## 15.1 Tổng quan Dự án và Thiết kế Kiến trúc

### 15.1.1 Vì sao Xây dựng một Thị trấn AI

NPC trong các trò chơi truyền thống thường chỉ có thể nói những câu thoại cố định hoặc có các tương tác hạn chế thông qua cây hội thoại được định sẵn. Ngay cả trong những trò chơi RPG phức tạp nhất, đối thoại của NPC cũng được biên kịch viết sẵn trước. Cách tiếp cận này có thể kiểm soát được nhưng thiếu "trí thông minh" và "sức sống" thực sự.

Hãy tưởng tượng nếu NPC trong game có thể hiểu bất cứ điều gì bạn nói, không còn bị giới hạn ở các tùy chọn định sẵn. Bạn có thể giao tiếp với NPC bằng ngôn ngữ tự nhiên. NPC sẽ nhớ những gì bạn nói lần trước, mối quan hệ của bạn, và thậm chí cả sở thích của bạn. Mỗi NPC có nghề nghiệp, tính cách và phong cách nói chuyện của riêng mình. Thái độ của NPC đối với bạn thay đổi theo các tương tác, từ người lạ đến bạn bè, thậm chí là bạn thân.

Đây chính là khả năng mới mà công nghệ AI mang lại cho trò chơi. Bằng cách kết hợp các mô hình ngôn ngữ lớn với game engine, chúng ta có thể tạo ra những NPC thực sự "sống". Đây không chỉ là một minh họa kỹ thuật, mà còn là sự khám phá về hình thái trò chơi trong tương lai. Trong các trò chơi giáo dục, NPC có thể đóng vai các nhân vật lịch sử và nhà khoa học, tiến hành giảng dạy tương tác với học sinh. Trong các văn phòng ảo, NPC có thể đóng vai đồng nghiệp và người cố vấn, cung cấp sự trợ giúp và lời khuyên. NPC cũng có thể đóng vai người bạn đồng hành, tiến hành giao tiếp cảm xúc với người dùng, ứng dụng trong lĩnh vực sức khỏe tâm thần. Tất nhiên, ứng dụng trực tiếp nhất là thêm NPC AI vào các trò chơi truyền thống để nâng cao trải nghiệm người chơi.

### 15.1.2 Tổng quan Kiến trúc Kỹ thuật

Thị trấn Cyber áp dụng kiến trúc tách biệt **game engine + dịch vụ back-end**, được chia thành bốn lớp, như minh họa trong Hình 15.1.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-1.png" alt="" width="85%"/>
  <p>Hình 15.1 Kiến trúc Kỹ thuật của Thị trấn Cyber</p>
</div>

Lớp front-end sử dụng game engine Godot 4.5, chịu trách nhiệm render game, điều khiển người chơi, hiển thị NPC và giao diện UI đối thoại. Godot là một game engine 2D/3D mã nguồn mở, rất phù hợp để phát triển nhanh các trò chơi phong cách pixel. Lớp back-end sử dụng framework FastAPI, chịu trách nhiệm định tuyến API, quản lý trạng thái NPC, xử lý đối thoại và ghi log. FastAPI là một web framework Python hiện đại với hiệu năng xuất sắc và dễ phát triển. Lớp agent sử dụng framework HelloAgents do chúng ta tự phát triển, chịu trách nhiệm về trí thông minh NPC, quản lý ký ức và tính toán thiện cảm. Mỗi NPC là một instance của SimpleAgent với ký ức và trạng thái độc lập. Lớp dịch vụ bên ngoài cung cấp các khả năng LLM, lưu trữ vector và lưu trữ dữ liệu bền vững, bao gồm LLM API, cơ sở dữ liệu vector Qdrant và cơ sở dữ liệu quan hệ SQLite.

Quy trình luồng dữ liệu được minh họa trong Hình 15.2:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-2.png" alt="" width="85%"/>
  <p>Hình 15.2 Quy trình Luồng Dữ liệu</p>
</div>

Người chơi nhấn phím E trong Godot để tương tác với NPC, và Godot gửi yêu cầu đối thoại đến back-end FastAPI thông qua HTTP API. Back-end gọi SimpleAgent của HelloAgents để xử lý cuộc đối thoại, Agent truy xuất lịch sử liên quan từ hệ thống ký ức, sau đó gọi LLM để sinh ra câu trả lời. Back-end cập nhật trạng thái NPC và thiện cảm, ghi log vào console và file, cuối cùng trả câu trả lời về cho front-end Godot. Godot hiển thị câu trả lời của NPC và cập nhật UI, hoàn thành một vòng lặp tương tác hoàn chỉnh.

Cấu trúc dự án như sau, giúp bạn dễ dàng định vị mã nguồn:

```
Helloagents-AI-Town/
├── helloagents-ai-town/           # Godot game project
│   ├── project.godot              # Godot project configuration
│   ├── scenes/                    # Game scenes
│   │   ├── main.tscn              # Main scene (office)
│   │   ├── player.tscn            # Player character
│   │   ├── npc.tscn               # NPC character
│   │   └── dialogue_ui.tscn       # Dialogue UI
│   ├── scripts/                   # GDScript scripts
│   │   ├── main.gd                # Main scene logic
│   │   ├── player.gd              # Player control
│   │   ├── npc.gd                 # NPC behavior
│   │   ├── dialogue_ui.gd         # Dialogue UI logic
│   │   ├── api_client.gd          # API client
│   │   └── config.gd              # Configuration management
│   └── assets/                    # Game assets
│       ├── characters/            # Character sprites
│       ├── interiors/             # Interior scenes
│       ├── ui/                    # UI materials
│       └── audio/                 # Sound effects and music
│
└── backend/                       # Python back-end
    ├── main.py                    # FastAPI main program
    ├── agents.py                  # NPC Agent system
    ├── relationship_manager.py    # Affection management
    ├── state_manager.py           # State management
    ├── logger.py                  # Logging system
    ├── config.py                  # Configuration management
    ├── models.py                  # Data models
    ├── requirements.txt           # Python dependencies
    └── .env.example               # Environment variable example
```

Thiết kế kiến trúc chi tiết và luồng dữ liệu sẽ được giới thiệu trong các phần tiếp theo.

### 15.1.3 Trải nghiệm Nhanh: Chạy Dự án trong 5 Phút

Trước khi đi sâu vào các chi tiết triển khai, hãy chạy dự án trước để xem kết quả cuối cùng. Bằng cách này, bạn sẽ có được hiểu biết trực quan về toàn bộ hệ thống.

**Yêu cầu Môi trường:**

- Godot 4.2 trở lên
- Python 3.10 trở lên
- Khóa API LLM (OpenAI, DeepSeek, Zhipu, v.v.)

**Lấy Dự án:**

Bạn có thể xem `code/chapter15/Helloagents-AI-Town`, hoặc clone toàn bộ kho lưu trữ hello-agents từ GitHub.

**Khởi động Back-End:**

```bash
# 1. Enter backend directory
cd Helloagents-AI-Town/backend

# 2. Install dependencies
pip install -r requirements.txt

# 3. Configure environment variables
cp .env.example .env
# Edit .env file, fill in your API key

# 4. Start back-end service
python main.py
```

Sau khi khởi động thành công, bạn sẽ thấy kết quả đầu ra sau:

```
============================================================
🎮 Cyber Town back-end service starting...
============================================================
✅ All services started!
📡 API address: http://0.0.0.0:8000
📚 API documentation: http://0.0.0.0:8000/docs
============================================================
```

**Khởi động Godot:**

Việc cài đặt Godot rất đơn giản. Windows cung cấp file `.exe` trực tiếp, và Mac cũng cung cấp file `.dmg`. Bạn có thể tải trực tiếp từ trang web chính thức ([Windows](https://godotengine.org/download/windows/) / [Mac](https://godotengine.org/download/macos/))

Mở Godot engine, nhấn nút "Import", duyệt đến `Helloagents-AI-Town/helloagents-ai-town/scenes/main.tscn`, và nhấn "Import and Edit". Sau khi Godot import xong tài nguyên, nhấn `F5` hoặc nhấn nút "Run" để khởi động trò chơi.

**Trải nghiệm các Tính năng Cốt lõi:**

Sau khi trò chơi khởi động, bạn sẽ thấy một cảnh văn phòng Datawhale phong cách pixel, như minh họa trong Hình 15.3.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-3.png" alt="" width="85%"/>
  <p>Hình 15.3 Cảnh Game của Thị trấn Cyber</p>
</div>

Sử dụng các phím WASD để di chuyển nhân vật người chơi. Khi bạn đi lại gần một NPC, màn hình sẽ hiển thị lời nhắc "Press E to interact". Sau khi nhấn phím E, một hộp thoại sẽ hiện lên, và bạn có thể nhập bất cứ điều gì bạn muốn nói, như minh họa trong Hình 15.4.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-4.png" alt="" width="85%"/>
  <p>Hình 15.4 Giao diện Đối thoại với NPC</p>
</div>

NPC sẽ phản hồi dựa trên thiết lập vai trò của mình (kỹ sư Python, quản lý sản phẩm, nhà thiết kế UI) và lịch sử tương tác của bạn. Khi cuộc trò chuyện tiến triển, thiện cảm của NPC đối với bạn sẽ dần tăng lên, từ "người lạ" đến "quen biết", rồi đến "thân thiện", "thân mật", và thậm chí là "bạn thân".

**Hệ thống thiện cảm được triển khai ở back-end**. Mỗi cuộc trò chuyện điều chỉnh giá trị thiện cảm dựa trên nội dung tin nhắn của người chơi và phân tích cảm xúc. Mặc dù giá trị thiện cảm không được hiển thị trực tiếp trong giao diện game front-end, tất cả các thay đổi thiện cảm đều được ghi lại chi tiết trong log back-end. Bạn có thể xem các thay đổi thiện cảm của mỗi cuộc trò chuyện trong file `backend/logs/dialogue_YYYY-MM-DD.log`. File log ghi lại thông tin chi tiết cho mỗi cuộc trò chuyện, bao gồm: giá trị thiện cảm hiện tại, các ký ức liên quan được truy xuất, câu trả lời của NPC, mức thay đổi thiện cảm (+2.0, +3.0, v.v.), lý do thay đổi (chào hỏi thân thiện, giao tiếp bình thường, v.v.), và kết quả phân tích cảm xúc (tích cực, trung tính, v.v.). Thiết kế này cho phép các nhà phát triển theo dõi rõ ràng sự phát triển mối quan hệ giữa NPC và người chơi, đồng thời cũng cung cấp một nền tảng dữ liệu để sau này thêm giao diện UI thiện cảm vào front-end.

Tất cả các cuộc trò chuyện đều được ghi lại trong các file log back-end. Bạn có thể xem chúng theo thời gian thực bằng lệnh sau:

```bash
# In the backend directory
python view_logs.py
```

Trải nghiệm đơn giản này minh họa các tính năng cốt lõi của Thị trấn AI. Tiếp theo, chúng ta sẽ đi sâu vào cách triển khai các tính năng này.

## 15.2 Hệ thống NPC Agent

### 15.2.1 SimpleAgent Dựa trên HelloAgents

Trong Thị trấn Cyber, mỗi NPC là một agent độc lập. Chúng ta sử dụng SimpleAgent từ framework HelloAgents để triển khai trí thông minh NPC. SimpleAgent là một triển khai agent nhẹ, đóng gói các chức năng cốt lõi như gọi LLM, quản lý tin nhắn và gọi công cụ (tool call).

Hãy nhớ lại SimpleAgent mà chúng ta đã học ở Chương 7. Cốt lõi của nó là một vòng lặp đối thoại đơn giản: nhận tin nhắn người dùng, gọi LLM để sinh câu trả lời, trả về kết quả. Trong Thị trấn Cyber, chúng ta cần tạo một instance SimpleAgent cho mỗi NPC và cấu hình các system prompt độc đáo cho chúng, mang lại cho mỗi NPC những tính cách và thiết lập vai trò khác nhau.

Hãy xem cách tạo một NPC Agent. Đầu tiên, chúng ta cần định nghĩa thông tin cơ bản của NPC, bao gồm ID, tên, nghề nghiệp và tính cách. Sau đó, chúng ta xây dựng system prompt dựa trên thông tin này, để LLM đóng vai NPC này. Cuối cùng, chúng ta tạo một instance SimpleAgent và cấu hình hệ thống ký ức.

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.memory import MemoryManager, WorkingMemory, EpisodicMemory

def create_npc_agent(npc_id: str, name: str, role: str, personality: str):
    """Create NPC Agent"""
    # Build system prompt
    system_prompt = f"""You are {name}, a {role}.
Your personality traits: {personality}

You work in the Datawhale office, working with colleagues to promote the development of the open source community.
Please have natural conversations with players based on your role and personality.
Remember your previous conversations to maintain dialogue coherence.
"""

    # Create LLM instance
    llm = HelloAgentsLLM()

    # Create memory manager
    memory_manager = MemoryManager(
        working_memory=WorkingMemory(capacity=10, ttl_minutes=120),
        episodic_memory=EpisodicMemory(
            db_path=f"memory_data/{npc_id}_episodic.db",
            collection_name=f"{npc_id}_memories"
        )
    )

    # Create Agent
    agent = SimpleAgent(
        name=name,
        llm=llm,
        system_prompt=system_prompt,
        memory_manager=memory_manager
    )

    return agent
```

Đoạn code này minh họa cách tạo một NPC Agent. System prompt định nghĩa danh tính và tính cách của NPC, và trình quản lý ký ức cho phép NPC ghi nhớ lịch sử trò chuyện với người chơi. WorkingMemory là ký ức ngắn hạn với dung lượng 10 tin nhắn và thời gian lưu giữ 120 phút. EpisodicMemory là ký ức dài hạn, sử dụng cơ sở dữ liệu SQLite và cơ sở dữ liệu vector Qdrant để lưu trữ, và có thể truy xuất các cuộc trò chuyện lịch sử liên quan.

Quy trình làm việc của NPC Agent được minh họa trong Hình 15.5:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-5.png" alt="" width="85%"/>
  <p>Hình 15.5 Quy trình Làm việc của NPC Agent</p>
</div>

### 15.2.2 Thiết lập Vai trò NPC và Thiết kế Prompt

Một NPC tốt cần có tính cách và thiết lập vai trò rõ ràng. Trong Thị trấn Cyber, chúng ta đã thiết kế ba NPC đại diện cho những nghề nghiệp và tính cách khác nhau.

**Trương Tam - Kỹ sư Python**

Trương Tam là một kỹ sư Python cao cấp chịu trách nhiệm phát triển cốt lõi framework HelloAgents. Anh ấy có tính cách nghiêm túc, nói thẳng, và thích sử dụng thuật ngữ kỹ thuật. Anh ấy có yêu cầu cao về chất lượng code và thường chia sẻ các mẹo lập trình và best practice.

```python
npc_zhang = {
    "npc_id": "zhang_san",
    "name": "Zhang San",
    "role": "Python Engineer",
    "personality": "Rigorous, professional, likes to share technical knowledge. Speaks directly, focuses on code quality."
}
```

**Lý Tứ - Quản lý Sản phẩm**

Lý Tứ là một quản lý sản phẩm giàu kinh nghiệm chịu trách nhiệm lập kế hoạch sản phẩm và thiết kế trải nghiệm người dùng của framework HelloAgents. Anh ấy có tính cách hướng ngoại, giỏi giao tiếp, và luôn có thể suy nghĩ từ góc độ của người dùng. Anh ấy thích thảo luận về thiết kế sản phẩm và nhu cầu người dùng, và thường hỏi "tại sao".

```python
npc_li = {
    "npc_id": "li_si",
    "name": "Li Si",
    "role": "Product Manager",
    "personality": "Outgoing, good at communication, focuses on user experience. Likes to think from the user's perspective."
}
```

**Vương Ngũ - Nhà thiết kế UI**

Vương Ngũ là một nhà thiết kế UI sáng tạo chịu trách nhiệm thiết kế giao diện và trình bày trực quan của framework HelloAgents. Anh ấy có tính cách hiền hòa, thẩm mỹ độc đáo, và có cảm nhận nhạy bén về màu sắc và bố cục. Anh ấy thích thảo luận về các khái niệm thiết kế và thẩm mỹ, và thường chia sẻ cảm hứng thiết kế.

```python
npc_wang = {
    "npc_id": "wang_wu",
    "name": "Wang Wu",
    "role": "UI Designer",
    "personality": "Gentle, creative, unique aesthetics. Focuses on visual presentation and user experience."
}
```

Ba NPC này có những đặc điểm rõ ràng. Người chơi có thể chọn tương tác với các NPC khác nhau dựa trên sở thích của mình. Trương Tam có thể dạy bạn các kỹ năng lập trình, Lý Tứ có thể thảo luận về thiết kế sản phẩm với bạn, và Vương Ngũ có thể chia sẻ cảm hứng thiết kế.

### 15.2.3 Tích hợp Hệ thống Ký ức

Hệ thống ký ức là chìa khóa cho trí thông minh của NPC. Một NPC có thể ghi nhớ các cuộc trò chuyện trong quá khứ sẽ khiến người chơi cảm thấy chân thực và thú vị hơn. Chúng ta sử dụng `WorkingMemory` và `EpisodicMemory` của HelloAgents để xây dựng ký ức ngắn hạn và dài hạn.

Ký ức ngắn hạn lưu trữ nội dung trò chuyện gần đây với dung lượng hạn chế và tự động dọn dẹp theo thời gian. Vai trò của nó là duy trì tính mạch lạc của đối thoại, cho phép NPC hiểu được ngữ cảnh. Ví dụ, khi người chơi nói "Nó màu gì?", NPC cần tìm từ ký ức ngắn hạn xem "nó" đề cập đến điều gì.

Ký ức dài hạn lưu trữ toàn bộ lịch sử trò chuyện, sử dụng cơ sở dữ liệu vector để truy xuất ngữ nghĩa. Khi người chơi đề cập đến một chủ đề, NPC có thể truy xuất các cuộc trò chuyện lịch sử liên quan từ ký ức dài hạn, nhớ lại nội dung đã thảo luận trước đó. Ví dụ, khi người chơi nói "Bạn có nhớ dự án chúng ta đã thảo luận lần trước không?", NPC có thể tìm thấy các bản ghi trò chuyện liên quan từ ký ức dài hạn.

Kiến trúc của hệ thống ký ức được minh họa trong Hình 15.6:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-6.png" alt="" width="85%"/>
  <p>Hình 15.6 Kiến trúc Hệ thống Ký ức</p>
</div>

Trong sử dụng thực tế, Agent trước tiên lấy các cuộc trò chuyện gần đây từ ký ức ngắn hạn, sau đó truy xuất các cuộc trò chuyện lịch sử liên quan từ ký ức dài hạn, gửi thông tin này cùng nhau đến LLM, và sinh ra các câu trả lời chính xác và cá nhân hóa hơn.

```python
# Agent's dialogue processing flow
def process_dialogue(agent, player_message):
    # 1. Get recent conversations from short-term memory
    recent_messages = agent.memory_manager.working_memory.get_recent_messages(5)

    # 2. Retrieve relevant history from long-term memory
    relevant_memories = agent.memory_manager.episodic_memory.search(
        query=player_message,
        top_k=3
    )

    # 3. Build context
    context = {
        "recent": recent_messages,
        "relevant": relevant_memories
    }

    # 4. Call Agent to generate reply
    reply = agent.run(player_message, context=context)

    # 5. Save to memory system
    agent.memory_manager.add_interaction(player_message, reply)

    return reply
```

Quy trình này đảm bảo rằng NPC có thể ghi nhớ lịch sử tương tác với người chơi và phản ánh nó trong các cuộc trò chuyện.

### 15.2.4 Sinh Đối thoại Hàng loạt: Chế độ Tải nhẹ

Trong vận hành thực tế, một vấn đề đã nhanh chóng được phát hiện: khi nhiều người chơi đồng thời trò chuyện với các NPC khác nhau, back-end cần xử lý đồng thời nhiều yêu cầu LLM. Mỗi yêu cầu cần gọi API, điều này không chỉ làm tăng chi phí mà còn có thể gây ra lỗi yêu cầu hoặc độ trễ do giới hạn đồng thời.

Để giải quyết vấn đề này, chúng ta đã thiết kế một **hệ thống sinh đối thoại hàng loạt**. Ý tưởng cốt lõi là: gộp nhiều yêu cầu đối thoại NPC thành một lần gọi LLM, để LLM sinh tất cả các câu trả lời NPC cùng một lúc. Điều này giống như "món ăn chế biến sẵn" của nhà hàng - chuẩn bị theo lô trước, sử dụng trực tiếp khi cần, giảm đáng kể chi phí và độ trễ.

Quy trình làm việc của việc sinh hàng loạt được minh họa trong Hình 15.7:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-7.png" alt="" width="85%"/>
  <p>Hình 15.7 Chế độ Sinh Hàng loạt so với Chế độ Truyền thống</p>
</div>

Việc triển khai bộ sinh hàng loạt rất thông minh. Chúng ta xây dựng một prompt đặc biệt yêu cầu LLM sinh tất cả các đối thoại NPC cùng một lúc và trả về chúng ở định dạng JSON. Bằng cách này, một lần gọi API có thể lấy được tất cả các câu trả lời NPC, giảm chi phí xuống còn 1/3 so với ban đầu và giảm đáng kể độ trễ.

```python
class NPCBatchGenerator:
    """Generator for batch generating NPC dialogues"""

    def __init__(self):
        self.llm = HelloAgentsLLM()
        self.npc_configs = NPC_ROLES  # All NPC configurations

    def generate_batch_dialogues(self, context: Optional[str] = None) -> Dict[str, str]:
        """Batch generate dialogues for all NPCs

        Args:
            context: Scene context (such as "morning work time", "lunch time", etc.)

        Returns:
            Dict[str, str]: Mapping from NPC names to dialogue content
        """
        # Build batch generation prompt
        prompt = self._build_batch_prompt(context)

        # One LLM call generates all dialogues
        response = self.llm.invoke([
            {"role": "system", "content": "You are a game NPC dialogue generator, skilled at creating natural and realistic office dialogues."},
            {"role": "user", "content": prompt}
        ])

        # Parse JSON response
        dialogues = json.loads(response)
        # Return format: {"Zhang San": "...", "Li Si": "...", "Wang Wu": "..."}

        return dialogues

    def _build_batch_prompt(self, context: Optional[str] = None) -> str:
        """Build batch generation prompt"""
        # Automatically infer scene based on time
        if context is None:
            context = self._get_current_context()

        # Build NPC descriptions
        npc_descriptions = []
        for name, cfg in self.npc_configs.items():
            desc = f"- {name}({cfg['title']}): {cfg['activity']} at {cfg['location']}, personality {cfg['personality']}"
            npc_descriptions.append(desc)

        npc_desc_text = "\n".join(npc_descriptions)

        prompt = f"""Please generate current dialogues or behavior descriptions for 3 NPCs in the Datawhale office.

【Scene】{context}

【NPC Information】
{npc_desc_text}

【Generation Requirements】
1. Generate 1 sentence for each NPC (20-40 characters)
2. Content should match role settings, current activities, and scene atmosphere
3. Can be self-talk, work status description, or simple thoughts
4. Should be natural and realistic, like real office colleagues
5. **Must strictly return in JSON format**

【Output Format】(strictly follow)
{{"Zhang San": "...", "Li Si": "...", "Wang Wu": "..."}}

【Example Output】
{{"Zhang San": "This bug is really annoying, been debugging for two hours...", "Li Si": "Hmm, the priority of this feature needs to be re-evaluated.", "Wang Wu": "The latte art on this coffee is really nice, inspiration is coming!"}}

Please generate (only return JSON, no other content):
"""
        return prompt
```

Điểm mấu chốt của thiết kế này là việc xây dựng prompt. Chúng ta yêu cầu rõ ràng LLM trả về định dạng JSON và cung cấp ví dụ đầu ra. LLM sẽ sinh các câu trả lời chặt chẽ theo định dạng này, và chúng ta chỉ cần phân tích JSON để lấy tất cả các đối thoại NPC.

Việc sinh hàng loạt có một lợi ích bổ sung: tất cả các đối thoại NPC đều được sinh trong cùng một ngữ cảnh, nên chúng có một mức độ tương quan nhất định. Ví dụ, nếu Trương Tam đang gỡ lỗi một bug, Lý Tứ có thể đề cập đến việc giúp xem qua; nếu Vương Ngũ đang thiết kế giao diện, Trương Tam có thể nói anh ấy sẽ kiểm tra bản thiết kế sau. Điều này làm cho bầu không khí của toàn bộ văn phòng chân thực và mạch lạc hơn.

Tất nhiên, việc sinh hàng loạt cũng có một số hạn chế. Nó phù hợp hơn để sinh "đối thoại nền" hoặc "tự nói chuyện" của NPC thay vì các tương tác trực tiếp với người chơi. Đối với các cuộc trò chuyện do người chơi khởi xướng, chúng ta vẫn sử dụng các Agent riêng lẻ để xử lý chúng nhằm đảm bảo các câu trả lời cá nhân hóa và chính xác. Việc sinh hàng loạt chủ yếu được sử dụng trong các tình huống sau:

1. **Đối thoại nền NPC**: NPC đang làm gì và nói gì khi người chơi vào cảnh
2. **Cập nhật định kỳ**: Cập nhật trạng thái và đối thoại NPC theo các khoảng thời gian đều đặn
3. **Bầu không khí cảnh**: Sinh các đối thoại khác nhau dựa trên thời gian (sáng, trưa, tối)
4. **Giảm chi phí**: Sử dụng việc sinh hàng loạt để giảm tần suất gọi API trong các tình huống đồng thời cao

**Chế độ Hỗn hợp: Sinh Hàng loạt + Phản hồi Tức thời**

Trong triển khai thực tế, chúng ta đã áp dụng một chế độ hỗn hợp kết hợp việc sinh hàng loạt và phản hồi tức thời. Thiết kế này rất thông minh, đảm bảo cả hiệu quả lẫn chất lượng tương tác.

Cụ thể, hệ thống định kỳ chạy việc sinh hàng loạt ở nền, sinh "đối thoại nền" cho tất cả các NPC trong cảnh hiện tại. Những đối thoại này được cache, và khi người chơi đến gần NPC nhưng chưa khởi xướng tương tác, NPC sẽ hiển thị những đối thoại nền này, chẳng hạn như "Đang gỡ lỗi code...", "Đang đọc tài liệu sản phẩm...", v.v. Điều này khiến NPC trông có vẻ "sống" thay vì là các mô hình tĩnh.

Tuy nhiên, khi người chơi nhấn phím E để khởi xướng tương tác, hệ thống ngay lập tức chuyển sang chế độ phản hồi tức thời. Tại thời điểm này, back-end gọi Agent chuyên dụng của NPC, sinh các câu trả lời cá nhân hóa dựa trên tin nhắn cụ thể của người chơi, ký ức lịch sử và mức độ thiện cảm. Quá trình này diễn ra theo thời gian thực, đảm bảo rằng câu trả lời của NPC có độ liên quan cao với đầu vào của người chơi.

```python
# Hybrid mode implementation in main.py
@app.post("/dialogue")
async def dialogue(request: DialogueRequest):
    """Handle player-NPC dialogue (instant response mode)"""
    npc_id = request.npc_id
    player_message = request.player_message
    player_name = request.player_name

    # Get NPC Agent (each NPC has an independent Agent)
    agent = npc_agents.get(npc_id)
    if not agent:
        raise HTTPException(status_code=404, detail="NPC not found")

    # Instantly generate personalized reply
    # Here we don't use batch generation, but call Agent's run method
    reply = agent.run(player_message)

    # Update affection
    affinity_change = relationship_manager.update_affinity(
        npc_id, player_name, player_message, reply
    )

    return {
        "npc_reply": reply,
        "affinity_score": affinity_change["score"],
        "affinity_level": affinity_change["level"]
    }

# Background task: periodically batch generate background dialogues
async def background_dialogue_update():
    """Background task: update NPC background dialogues every 5 minutes"""
    while True:
        try:
            # Use batch generator to generate background dialogues for all NPCs
            batch_generator = get_batch_generator()
            dialogues = batch_generator.generate_batch_dialogues()

            # Update to state manager
            for npc_name, dialogue in dialogues.items():
                state_manager.update_npc_background_dialogue(npc_name, dialogue)

            print(f"✅ Background dialogue update complete: {len(dialogues)} NPCs")
        except Exception as e:
            print(f"❌ Background dialogue update failed: {e}")

        # Wait 5 minutes
        await asyncio.sleep(300)
```

Những ưu điểm của chế độ hỗn hợp này rất rõ ràng:

1. **Giảm chi phí**: Đối thoại nền sử dụng việc sinh hàng loạt, một lần gọi sinh tất cả các đối thoại NPC, chi phí thấp
2. **Đảm bảo chất lượng**: Các tương tác của người chơi sử dụng phản hồi tức thời, mỗi câu trả lời đều được cá nhân hóa, chất lượng cao
3. **Nâng cao trải nghiệm**: NPC luôn có "đối thoại nền", trông rất sống động; các tương tác của người chơi có câu trả lời chính xác, trải nghiệm tốt
4. **Điều chỉnh linh hoạt**: Có thể điều chỉnh động tần suất sinh hàng loạt dựa trên tải máy chủ

Thông qua sự kết hợp giữa việc sinh hàng loạt và phản hồi tức thời, chúng ta đã triển khai một hệ thống NPC vừa hiệu quả vừa thông minh. Trong điều kiện bình thường, người chơi không cảm nhận được bất kỳ sự khác biệt nào, nhưng chi phí và hiệu năng back-end được tối ưu hóa đáng kể. Cách tiếp cận thiết kế này cũng có thể được áp dụng cho các tình huống khác đòi hỏi một số lượng lớn các lần gọi AI.

## 15.3 Thiết kế Hệ thống Thiện cảm

### 15.3.1 Phân loại Cấp độ Thiện cảm

Trong Thị trấn Cyber, thái độ của NPC đối với người chơi thay đổi theo các tương tác. Chúng ta đã thiết kế một hệ thống thiện cảm năm cấp độ, từ người lạ đến bạn thân, mỗi cấp độ có khoảng điểm khác nhau và các biểu hiện hành vi tương ứng.

Ý tưởng cốt lõi của hệ thống thiện cảm là: bằng cách định lượng mối quan hệ giữa NPC và người chơi, làm cho các câu trả lời của NPC chân thực và có tầng lớp hơn. Khi người chơi lần đầu vào game, tất cả các NPC đều có thái độ người lạ đối với người chơi, với các câu trả lời lịch sự nhưng xa cách. Khi các cuộc trò chuyện tiến triển, nếu người chơi cư xử thân thiện, thiện cảm của NPC sẽ dần tăng lên, và các câu trả lời sẽ trở nên thân thiết và chi tiết hơn.

Chúng ta chia thiện cảm thành năm cấp độ, mỗi cấp độ tương ứng với một khoảng điểm, như minh họa trong Hình 15.8:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-8.png" alt="" width="85%"/>
  <p>Hình 15.8 Phân loại Cấp độ Thiện cảm</p>
</div>

- **Người lạ (0-20 điểm)**: NPC vừa mới gặp người chơi, thái độ lịch sự nhưng giữ khoảng cách. Các câu trả lời ngắn gọn, sẽ không chủ động chia sẻ thông tin cá nhân.

- **Quen biết (21-40 điểm)**: NPC bắt đầu nhớ người chơi, sẵn sàng trao đổi đơn giản. Các câu trả lời trở nên tự nhiên hơn, thỉnh thoảng chia sẻ một số thông tin liên quan đến công việc.

- **Thân thiện (41-60 điểm)**: NPC coi người chơi như một người bạn, sẵn sàng chia sẻ nhiều thông tin hơn. Các câu trả lời chi tiết hơn, sẽ chủ động hỏi thăm tình hình của người chơi.

- **Thân mật (61-80 điểm)**: NPC rất tin tưởng người chơi, sẵn sàng chia sẻ các chủ đề riêng tư. Các câu trả lời tràn đầy nhiệt tình, sẽ cung cấp sự trợ giúp và lời khuyên cho người chơi.

- **Bạn thân (81-100 điểm)**: NPC coi người chơi như người bạn tốt nhất, tâm sự mọi thứ. Các câu trả lời rất thân thiết, sẽ chia sẻ những suy nghĩ và cảm xúc trong lòng.

Thiết kế này cho phép người chơi cảm nhận rõ ràng sự thay đổi trong mối quan hệ của họ với NPC, đồng thời cũng cung cấp một nền tảng cho lối chơi sau này. Ví dụ, chỉ sau khi đạt đến một mức độ thiện cảm nhất định thì NPC mới chia sẻ một số thông tin đặc biệt hoặc cung cấp các nhiệm vụ đặc biệt.

### 15.3.2 Logic Tính toán Thiện cảm

Việc tính toán thiện cảm cần cân nhắc nhiều yếu tố. Chúng ta không thể đơn giản cộng một điểm số cố định cho mỗi cuộc trò chuyện, điều đó sẽ làm cho hệ thống trông máy móc và không chân thực. Một hệ thống thiện cảm tốt cần có khả năng nhận diện thái độ của người chơi và điều chỉnh điểm số một cách động dựa trên nội dung trò chuyện.

Trong Thị trấn Cyber, chúng ta sử dụng LLM để phân tích nội dung trò chuyện, đánh giá xem thái độ của người chơi là thân thiện, trung tính hay không thân thiện. Sau đó chúng ta điều chỉnh điểm thiện cảm dựa trên kết quả đánh giá. Quá trình này là tự động, người chơi không cần cố ý chọn các tùy chọn, làm cho các tương tác tự nhiên hơn.

Quy trình tính toán thiện cảm được minh họa trong Hình 15.9:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-9.png" alt="" width="85%"/>
  <p>Hình 15.9 Quy trình Tính toán Thiện cảm</p>
</div>

```python
class RelationshipManager:
    """Affection manager"""

    def __init__(self):
        self.affinity_data = {}  # Store affection data
        self.llm = HelloAgentsLLM()  # For analyzing conversations

    def analyze_sentiment(self, player_message: str, npc_reply: str) -> int:
        """Analyze conversation sentiment, return affection change value"""
        prompt = f"""Analyze the player's attitude in the following conversation:
Player: {player_message}
NPC: {npc_reply}

Please judge if the player's attitude is:
1. Friendly (+5 points): Polite, enthusiastic, expressing thanks or agreement
2. Neutral (+2 points): Normal inquiry or statement
3. Unfriendly (-3 points): Rude, indifferent, critical or negative

Only return the number, no other content."""

        response = self.llm.think([{"role": "user", "content": prompt}])
        try:
            score_change = int(response.strip())
            return max(-3, min(5, score_change))  # Limit between -3 and 5
        except:
            return 2  # Default neutral

    def update_affinity(self, npc_id: str, player_name: str,
                       player_message: str, npc_reply: str) -> dict:
        """Update affection"""
        key = f"{npc_id}_{player_name}"

        # Get current affection
        if key not in self.affinity_data:
            self.affinity_data[key] = {
                "score": 0,
                "level": "Stranger",
                "interaction_count": 0
            }

        # Analyze conversation sentiment
        score_change = self.analyze_sentiment(player_message, npc_reply)

        # Update score
        current_score = self.affinity_data[key]["score"]
        new_score = max(0, min(100, current_score + score_change))

        # Update level
        level = self.get_affinity_level(new_score)

        # Update data
        self.affinity_data[key].update({
            "score": new_score,
            "level": level,
            "interaction_count": self.affinity_data[key]["interaction_count"] + 1
        })

        return self.affinity_data[key]

    def get_affinity_level(self, score: int) -> str:
        """Get affection level based on score"""
        if score <= 20:
            return "Stranger"
        elif score <= 40:
            return "Familiar"
        elif score <= 60:
            return "Friendly"
        elif score <= 80:
            return "Intimate"
        else:
            return "Close Friend"
```

Việc triển khai này sử dụng LLM để phân tích nội dung trò chuyện, tự động đánh giá thái độ của người chơi và điều chỉnh thiện cảm. Thiết kế này làm cho hệ thống thiện cảm thông minh và tự nhiên hơn, người chơi không cần cố ý lấy lòng NPC, chỉ cần giao tiếp bình thường.

### 15.3.3 Thiện cảm Ảnh hưởng đến Đối thoại

Thiện cảm không chỉ là một con số, nó nên thực sự ảnh hưởng đến hành vi của NPC. Trong Thị trấn Cyber, chúng ta điều chỉnh system prompt của NPC để NPC điều chỉnh phong cách trả lời dựa trên các cấp độ thiện cảm hiện tại.

Khi thiện cảm thấp, NPC duy trì thái độ lịch sự nhưng xa cách. Khi thiện cảm tăng lên, NPC trở nên nhiệt tình và nói nhiều hơn. Sự thay đổi này được thực hiện bằng cách điều chỉnh động system prompt.

```python
def create_npc_agent_with_affinity(npc_id: str, name: str, role: str,
                                   personality: str, affinity_level: str):
    """Create NPC Agent with affection"""

    # Adjust prompts based on affection level
    affinity_prompts = {
        "Stranger": "You just met this player, be polite but not overly enthusiastic. Keep replies brief and professional.",
        "Familiar": "You already know this player, can have normal exchanges. Replies should be natural and friendly.",
        "Friendly": "You treat this player as a friend, willing to share more information. Replies should be detailed and enthusiastic.",
        "Intimate": "You trust this player very much, can share private topics. Replies should be full of care.",
        "Close Friend": "You treat this player as your best friend, talk about everything. Replies should be cordial and sincere."
    }

    system_prompt = f"""You are {name}, a {role}.
Your personality traits: {personality}

Current relationship with player: {affinity_level}
{affinity_prompts.get(affinity_level, affinity_prompts["Stranger"])}

You work in the Datawhale office, working with colleagues to promote the development of the open source community.
Please reply naturally based on your role, personality, and relationship with the player.
"""

    # Create Agent
    llm = HelloAgentsLLM()
    agent = SimpleAgent(
        name=name,
        llm=llm,
        system_prompt=system_prompt
    )

    return agent
```

Thiết kế này làm cho hành vi của NPC thay đổi động theo thiện cảm. Người chơi có thể cảm nhận rõ ràng rằng khi các tương tác tăng lên, thái độ của NPC đối với họ đang dần thay đổi, nâng cao đáng kể tính đắm chìm và niềm vui của trò chơi.

## 15.4 Triển khai Dịch vụ Back-End

### 15.4.1 Cấu trúc Ứng dụng FastAPI

Back-end của Thị trấn Cyber được xây dựng bằng framework FastAPI, chịu trách nhiệm xử lý các yêu cầu từ front-end Godot, gọi các NPC Agent của HelloAgents, quản lý trạng thái NPC và thiện cảm, và ghi log. Một cấu trúc ứng dụng rõ ràng làm cho code dễ bảo trì và mở rộng hơn.

Ứng dụng FastAPI của chúng ta áp dụng thiết kế mô-đun hóa, tách các chức năng khác nhau vào các file khác nhau, như minh họa trong Hình 15.10:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-10.png" alt="" width="85%"/>
  <p>Hình 15.10 Cấu trúc Ứng dụng Back-End</p>
</div>

Hãy bắt đầu với `main.py`, file entry của ứng dụng FastAPI:

```python
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, Field
from typing import Optional
import uvicorn

from agents import NPCAgentManager
from relationship_manager import RelationshipManager
from state_manager import StateManager
from logger import DialogueLogger
from config import settings

# Create FastAPI application
app = FastAPI(
    title="Cyber Town Back-End Service",
    description="AI NPC dialogue system based on HelloAgents",
    version="1.0.0"
)

# Configure CORS, allow Godot front-end access
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Production environment should limit specific domains
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Initialize各个managers
agent_manager = NPCAgentManager()
relationship_manager = RelationshipManager()
state_manager = StateManager()
dialogue_logger = DialogueLogger()

@app.on_event("startup")
async def startup_event():
    """Initialization on application startup"""
    print("=" * 60)
    print("🎮 Cyber Town back-end service starting...")
    print("=" * 60)

    # Initialize NPC Agents
    agent_manager.initialize_npcs()
    print("✅ NPC Agents initialized")

    # Initialize state manager
    state_manager.initialize_npcs()
    print("✅ State manager initialized")

@app.get("/")
async def root():
    """Health check"""
    return {
        "status": "running",
        "message": "Cyber Town back-end service is running",
        "version": "1.0.0",
        "npcs": state_manager.get_npc_count()
    }

if __name__ == "__main__":
    uvicorn.run(
        app,
        host=settings.HOST,
        port=settings.PORT,
        log_level="info"
    )
```

File chương trình chính này định nghĩa cấu trúc cơ bản của ứng dụng FastAPI, cấu hình middleware CORS để cho phép các yêu cầu cross-origin, và khởi tạo các trình quản lý khi khởi động. Tiếp theo chúng ta sẽ triển khai các route API cụ thể.

### 15.4.2 Thiết kế Route API

Back-end của Thị trấn Cyber cần cung cấp một số endpoint API cốt lõi để xử lý các yêu cầu từ front-end Godot. Chúng ta thêm các route này vào `main.py`.

**Lấy Trạng thái NPC**

API này trả về trạng thái hiện tại của tất cả các NPC, bao gồm vị trí, có bận hay không, v.v.:

```python
from models import NPCStatusResponse

@app.get("/npcs/status", response_model=NPCStatusResponse)
async def get_npc_status():
    """Get status of all NPCs"""
    npcs = state_manager.get_all_npc_states()
    return {"npcs": npcs}

@app.get("/npcs/{npc_id}/status")
async def get_single_npc_status(npc_id: str):
    """Get status of a single NPC"""
    npc = state_manager.get_npc_state(npc_id)
    if not npc:
        raise HTTPException(status_code=404, detail=f"NPC {npc_id} does not exist")
    return npc
```

**Giao diện Đối thoại**

Đây là API cốt lõi nhất, xử lý các cuộc trò chuyện giữa người chơi và NPC:

```python
from models import DialogueRequest, DialogueResponse

@app.post("/dialogue", response_model=DialogueResponse)
async def dialogue(request: DialogueRequest):
    """Handle player-NPC dialogue"""
    # 1. Verify NPC exists
    if not agent_manager.has_npc(request.npc_id):
        raise HTTPException(status_code=404, detail=f"NPC {request.npc_id} does not exist")

    # 2. Check if NPC is busy
    if state_manager.is_npc_busy(request.npc_id):
        raise HTTPException(status_code=409, detail=f"NPC {request.npc_id} is talking with another player")

    # 3. Mark NPC as busy
    state_manager.set_npc_busy(request.npc_id, True)

    try:
        # 4. Get current affection
        affinity_info = relationship_manager.get_affinity(
            request.npc_id,
            request.player_name
        )

        # 5. Call Agent to generate reply
        agent = agent_manager.get_agent(request.npc_id, affinity_info["level"])
        reply = agent.run(request.player_message)

        # 6. Update affection
        new_affinity = relationship_manager.update_affinity(
            request.npc_id,
            request.player_name,
            request.player_message,
            reply
        )

        # 7. Record log
        dialogue_logger.log_dialogue(
            npc_id=request.npc_id,
            player_name=request.player_name,
            player_message=request.player_message,
            npc_reply=reply,
            affinity_info=new_affinity
        )

        # 8. Return reply
        return DialogueResponse(
            npc_reply=reply,
            affinity_level=new_affinity["level"],
            affinity_score=new_affinity["score"]
        )

    except Exception as e:
        dialogue_logger.log_error(f"Dialogue processing failed: {str(e)}")
        raise HTTPException(status_code=500, detail=f"Dialogue processing failed: {str(e)}")

    finally:
        # 9. Release NPC status
        state_manager.set_npc_busy(request.npc_id, False)
```

**Truy vấn Thiện cảm**

API này cho phép truy vấn thiện cảm giữa người chơi và NPC:

```python
from models import AffinityInfo

@app.get("/affinity/{npc_id}/{player_name}", response_model=AffinityInfo)
async def get_affinity(npc_id: str, player_name: str):
    """Get player-NPC affection"""
    if not agent_manager.has_npc(npc_id):
        raise HTTPException(status_code=404, detail=f"NPC {npc_id} does not exist")

    affinity = relationship_manager.get_affinity(npc_id, player_name)
    return affinity
```

Luồng gọi route API được minh họa trong Hình 15.11:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-11.png" alt="" width="85%"/>
  <p>Hình 15.11 Luồng Gọi API</p>
</div>

### 15.4.3 Quản lý Trạng thái và Hệ thống Ghi log

**Trình quản lý Trạng thái**

Trình quản lý trạng thái chịu trách nhiệm theo dõi trạng thái hiện tại của mỗi NPC, bao gồm vị trí, có bận hay không, hành động hiện tại, v.v. Điều này quan trọng để ngăn ngừa các vấn đề đồng thời, chẳng hạn như tránh việc một NPC trò chuyện với nhiều người chơi cùng lúc.

```python
# state_manager.py
from typing import Dict, List, Optional
from datetime import datetime

class StateManager:
    """NPC state manager"""

    def __init__(self):
        self.npc_states: Dict[str, dict] = {}

    def initialize_npcs(self):
        """Initialize NPC states"""
        npcs = [
            {
                "npc_id": "zhang_san",
                "name": "Zhang San",
                "role": "Python Engineer",
                "position": {"x": 300, "y": 200}
            },
            {
                "npc_id": "li_si",
                "name": "Li Si",
                "role": "Product Manager",
                "position": {"x": 500, "y": 200}
            },
            {
                "npc_id": "wang_wu",
                "name": "Wang Wu",
                "role": "UI Designer",
                "position": {"x": 700, "y": 200}
            }
        ]

        for npc in npcs:
            self.npc_states[npc["npc_id"]] = {
                **npc,
                "is_busy": False,
                "current_action": "idle",
                "last_interaction": None
            }

    def get_npc_state(self, npc_id: str) -> Optional[dict]:
        """Get NPC state"""
        return self.npc_states.get(npc_id)

    def get_all_npc_states(self) -> List[dict]:
        """Get all NPC states"""
        return list(self.npc_states.values())

    def is_npc_busy(self, npc_id: str) -> bool:
        """Check if NPC is busy"""
        npc = self.npc_states.get(npc_id)
        return npc["is_busy"] if npc else False

    def set_npc_busy(self, npc_id: str, busy: bool):
        """Set NPC busy status"""
        if npc_id in self.npc_states:
            self.npc_states[npc_id]["is_busy"] = busy
            if busy:
                self.npc_states[npc_id]["last_interaction"] = datetime.now().isoformat()

    def get_npc_count(self) -> int:
        """Get NPC count"""
        return len(self.npc_states)
```

**Hệ thống Ghi log**

Hệ thống ghi log triển khai đầu ra kép: console và file. Điều này giúp thuận tiện để xem theo thời gian thực và lưu các bản ghi lịch sử.

```python
# logger.py
import logging
from datetime import datetime
from pathlib import Path

class DialogueLogger:
    """Dialogue logger"""

    def __init__(self, log_dir: str = "logs"):
        self.log_dir = Path(log_dir)
        self.log_dir.mkdir(exist_ok=True)

        # Create log file name (by date)
        today = datetime.now().strftime("%Y-%m-%d")
        log_file = self.log_dir / f"dialogue_{today}.log"

        # Configure logging
        self.logger = logging.getLogger("DialogueLogger")
        self.logger.setLevel(logging.INFO)

        # Console handler
        console_handler = logging.StreamHandler()
        console_handler.setLevel(logging.INFO)
        console_formatter = logging.Formatter(
            '%(asctime)s - %(levelname)s - %(message)s',
            datefmt='%H:%M:%S'
        )
        console_handler.setFormatter(console_formatter)

        # File handler
        file_handler = logging.FileHandler(log_file, encoding='utf-8')
        file_handler.setLevel(logging.INFO)
        file_formatter = logging.Formatter(
            '%(asctime)s - %(levelname)s - %(message)s',
            datefmt='%Y-%m-%d %H:%M:%S'
        )
        file_handler.setFormatter(file_formatter)

        # Add handlers
        self.logger.addHandler(console_handler)
        self.logger.addHandler(file_handler)

    def log_dialogue(self, npc_id: str, player_name: str,
                    player_message: str, npc_reply: str,
                    affinity_info: dict):
        """Log dialogue"""
        log_message = f"""
{'='*60}
NPC: {npc_id}
Player: {player_name}
Player message: {player_message}
NPC reply: {npc_reply}
Affection: {affinity_info['level']} ({affinity_info['score']}/100)
Interaction count: {affinity_info['interaction_count']}
{'='*60}
"""
        self.logger.info(log_message)

    def log_error(self, error_message: str):
        """Log error"""
        self.logger.error(error_message)
```

Hệ thống ghi log này hiển thị nội dung đối thoại theo thời gian thực trên console đồng thời lưu nó vào file. Log của mỗi ngày được lưu trong các file riêng biệt để dễ dàng phân tích sau này.

### 15.4.4 Hiểu về Hệ thống Cảnh (Scene) của Godot

Trước khi bắt đầu xây dựng các cảnh game, chúng ta cần hiểu trước về các khái niệm cốt lõi của Godot - Scene (Cảnh) và Node (Nút). Đây là điểm khác biệt lớn nhất giữa Godot và các game engine khác, đồng thời cũng là một trong những tính năng mạnh mẽ nhất của nó.

**Node là gì?**

Node là những khối xây dựng cơ bản nhất trong Godot. Bạn có thể coi các node như những viên gạch Lego, mỗi node có một chức năng cụ thể. Ví dụ, node Sprite2D được dùng để hiển thị hình ảnh, node AudioStreamPlayer được dùng để phát âm thanh, và node CharacterBody2D được dùng để xử lý chuyển động vật lý của nhân vật. Godot cung cấp hàng trăm loại node khác nhau, mỗi loại tập trung làm tốt một việc.

Các node có thể tạo thành các mối quan hệ cha-con, hình thành cấu trúc cây. Node cha có thể ảnh hưởng đến node con, ví dụ, di chuyển một node cha sẽ đồng thời di chuyển tất cả các node con, ẩn một node cha sẽ đồng thời ẩn tất cả các node con. Mối quan hệ phân cấp này cho phép chúng ta dễ dàng tổ chức và quản lý các đối tượng game phức tạp.

**Scene là gì?**

Một scene là một tập hợp các node, được lưu trong file .tscn. Bạn có thể coi một scene như một "prefab" (thành phần đúc sẵn). Ví dụ, chúng ta có thể tạo một scene "player" chứa tất cả các node liên quan như sprite nhân vật, collision body, hiệu ứng âm thanh, v.v. Sau đó sử dụng scene này nhiều lần trong game, mỗi lần sử dụng sẽ tạo ra một instance độc lập.

Sức mạnh của scene nằm ở khả năng tái sử dụng và mô-đun hóa của chúng. Chúng ta có thể instance hóa một scene bên trong một scene khác, tạo thành các cấu trúc lồng nhau. Ví dụ, scene chính có thể chứa scene player, nhiều scene NPC, và scene UI. Việc sửa đổi scene NPC sẽ tự động ảnh hưởng đến tất cả các instance NPC, đơn giản hóa đáng kể việc phát triển và bảo trì.

**Một Ví dụ Đơn giản**

Hãy dùng một ví dụ đơn giản để hiểu về scene và node. Giả sử chúng ta muốn tạo một scene "player":

```
Player (CharacterBody2D)  ← Root node, responsible for physics movement
├─ AnimatedSprite2D       ← Child node, displays character animation
├─ CollisionShape2D       ← Child node, defines collision shape
└─ Camera2D               ← Child node, camera follows player
```

Scene này chứa 4 node tạo thành cấu trúc cây. CharacterBody2D là node gốc, ba node còn lại là các node con của nó. Chúng ta có thể thêm script vào mỗi node để điều khiển hành vi của nó, hoặc thêm một script vào node gốc để điều phối tất cả các node con.

Khi chúng ta instance hóa scene Player này trong scene chính, Godot tạo ra một bản sao của toàn bộ cây node này. Chúng ta có thể tạo nhiều instance player, mỗi instance là độc lập với vị trí, trạng thái và hành vi riêng của nó.

**Ưu điểm của việc Instance hóa Scene**

Trong Thị trấn Cyber, chúng ta có ba NPC: Trương Tam, Lý Tứ và Vương Ngũ. Nếu không sử dụng hệ thống scene, chúng ta sẽ phải tạo các node, thiết lập thuộc tính và viết script cho từng NPC riêng biệt, dẫn đến rất nhiều công việc lặp lại. Sử dụng hệ thống scene, chúng ta chỉ cần tạo một scene NPC chung, sau đó instance hóa nó ba lần, thiết lập tên và thông tin vai trò khác nhau thông qua các tham số script.

Lợi ích của thiết kế này là: nếu chúng ta muốn thêm một tính năng mới cho tất cả các NPC (chẳng hạn như hiển thị bong bóng đối thoại phía trên đầu của chúng), chúng ta chỉ cần sửa đổi scene NPC, và tất cả các instance sẽ tự động có được tính năng này.

## 15.5 Xây dựng Cảnh Game Godot

**Vì sao Chọn Godot làm Game Engine?**

Trong số nhiều game engine, chúng ta đã chọn Godot 4.5 làm engine front-end, chủ yếu dựa trên những cân nhắc sau:

(1) **Godot có lợi thế tự nhiên trong phát triển game 2D**. Thị trấn Cyber là một trò chơi phong cách pixel 2D nhìn từ trên xuống (top-down). Engine 2D của Godot rất trưởng thành, cung cấp các loại node được thiết kế đặc biệt cho game 2D như TileMap, AnimatedSprite2D, CharacterBody2D, v.v. Hiệu quả phát triển cao hơn nhiều so với các engine như Unity. Hệ thống Scene của Godot cho phép chúng ta đóng gói các thành phần như người chơi, NPC và UI thành các scene độc lập, sau đó instance hóa chúng trong scene chính. Thiết kế dựa trên thành phần này rất phù hợp với nhu cầu của chúng ta.

(2) **Godot hoàn toàn mã nguồn mở và miễn phí**. Godot sử dụng giấy phép MIT, không có phí bản quyền hoặc chia sẻ doanh thu, điều này rất thân thiện với các dự án giảng dạy và dự án mã nguồn mở. Bạn có thể tự do sửa đổi mã nguồn engine và thương mại hóa game mà không phải lo lắng về các vấn đề bản quyền. Ngược lại, mặc dù Unity mạnh mẽ, nhưng nó đã đưa ra chính sách phí runtime vào năm 2024, gây ra tranh cãi rộng rãi trong cộng đồng nhà phát triển.

(3) **Godot có chi phí học tập cực kỳ thấp**. Godot sử dụng GDScript làm ngôn ngữ scripting chính, một ngôn ngữ có kiểu động (dynamically typed) tương tự Python với cú pháp ngắn gọn dễ hiểu và đường cong học tập rất thoải mái. Đối với những độc giả đã quen thuộc với Python, việc học GDScript hầu như không có rào cản - khai báo biến, định nghĩa hàm, luồng điều khiển và các cú pháp khác đều rất giống Python. Bạn thậm chí có thể bắt đầu viết các script game trong vài giờ. Cấu trúc cây node của Godot cũng rất trực quan, bạn có thể trực quan nhìn thấy các mối quan hệ phân cấp của scene trong editor, điều này rất thân thiện với người mới bắt đầu.

(4) **Godot tích hợp rất đơn giản với back-end Python**. Godot có node HTTPRequest tích hợp sẵn có thể dễ dàng giao tiếp với back-end FastAPI qua HTTP. Chúng ta chỉ cần tạo một script API client đóng gói tất cả các lệnh gọi API để có thể gọi các khả năng AI của back-end trong game. Kiến trúc tách biệt front-end và back-end này cho phép chúng ta phát triển và kiểm thử độc lập logic game và logic AI, cải thiện đáng kể hiệu quả phát triển.

Tất nhiên, Godot cũng có một số hạn chế. Ví dụ, khả năng 3D của Godot vẫn còn tụt hậu so với Unreal Engine và Unity. Nếu bạn muốn phát triển các trò chơi 3D quy mô lớn, bạn có thể cần cân nhắc các engine khác. Nhưng đối với game 2D, game indie và các dự án giảng dạy, Godot là một lựa chọn tuyệt vời.

### 15.5.1 Thiết kế Cảnh và Tổ chức Tài nguyên

Sau khi hiểu về hệ thống scene của Godot, hãy xem thiết kế cảnh của Thị trấn Cyber. Toàn bộ trò chơi bao gồm bốn scene cốt lõi: Main (scene chính), Player (người chơi), NPC (nhân vật không phải người chơi), và DialogueUI (giao diện đối thoại). Mỗi scene là một mô-đun độc lập có thể được chỉnh sửa và kiểm thử riêng, sau đó kết hợp lại để tạo thành một trò chơi hoàn chỉnh.

Việc tổ chức scene của Thị trấn Cyber áp dụng thiết kế mô-đun hóa. Trước tiên chúng ta tạo ba scene cơ bản: Player (người chơi), NPC (nhân vật không phải người chơi), và DialogueUI (giao diện đối thoại). Sau đó trong Main (scene chính), chúng ta instance hóa và kết hợp các scene này. Điều đặc biệt đáng chú ý là ba NPC (Trương Tam, Lý Tứ, Vương Ngũ) đều là các instance của cùng một scene NPC, chỉ khác nhau ở thông tin vai trò được thiết lập thông qua các tham số script.

Trước tiên hãy xem cấu trúc của bốn scene cốt lõi, như minh họa trong Hình 15.12:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-12.png" alt="" width="85%"/>
  <p>Hình 15.12 Bốn Scene Cốt lõi của Thị trấn Cyber</p>
</div>

Sơ đồ này thể hiện bốn scene độc lập và cấu trúc bên trong của chúng. **Scene 1 (Main)** là scene chính, chứa hình nền (Sprite2D), instance player, node tổ chức NPCs (bên dưới có ba instance NPC), instance giao diện đối thoại, node tổ chức walls, và nhạc nền. Lưu ý rằng Player, NPC_Zhang, NPC_Li, NPC_Wang, và DialogueUI ở đây là các instance scene, không phải các node thông thường. **Scene 2 (Player)** định nghĩa cấu trúc nhân vật người chơi, bao gồm animation, collision, camera, và hai node hiệu ứng âm thanh. **Scene 3 (NPC)** là một template chung - Trương Tam, Lý Tứ và Vương Ngũ đều là các instance của scene này, chứa collision, animation, khu vực tương tác, và hai label. **Scene 4 (DialogueUI)** là một node CanvasLayer chứa Panel và các phần tử UI khác nhau.

Quá trình instance hóa scene có thể hiểu theo cách này: Chúng ta đã tạo file scene NPC.tscn trong editor Godot, định nghĩa cấu trúc node của NPC. Sau đó trong scene Main, chúng ta "instance hóa" scene NPC này ba lần, tạo ra ba bản sao độc lập được đặt tên lần lượt là NPC_Zhang, NPC_Li, và NPC_Wang. Mỗi bản sao có vị trí và trạng thái riêng của nó, nhưng chúng chia sẻ cùng một cấu trúc node. Nếu chúng ta sửa đổi NPC.tscn, chẳng hạn như thêm một node hiệu ứng âm thanh mới cho NPC, cả ba instance sẽ tự động có được hiệu ứng âm thanh này.

Các bước để tạo các scene này trong Godot như sau:

1. **Tạo scene Player**: Tạo scene mới, chọn CharacterBody2D làm node gốc, thêm các node con AnimatedSprite2D, CollisionShape2D, Camera2D, InteractSound, và RunningSound, lưu thành Player.tscn.

2. **Tạo scene NPC**: Tạo scene mới, chọn CharacterBody2D làm node gốc, thêm các node con CollisionShape2D, AnimatedSprite2D, InteractionArea (Area2D với CollisionShape2D bên dưới), NameLabel, và DialogueLabel, lưu thành NPC.tscn.

3. **Tạo scene DialogueUI**: Tạo scene mới, chọn CanvasLayer làm node gốc, thêm node con Panel, dưới Panel thêm NPCName, NPCTitle, DialogueText (RichTextLabel), PlayerInput (LineEdit), SendButton, và CloseButton, lưu thành DialogueUI.tscn.

4. **Tạo scene Main**: Tạo scene mới, chọn Node2D làm node gốc, thêm Background (Sprite2D) làm hình nền, dưới Background thêm trang trí cá voi, sau đó instance hóa scene Player, tạo node NPCs và instance hóa scene NPC ba lần bên dưới nó, instance hóa scene DialogueUI, tạo node Walls để tổ chức các collision tường, cuối cùng thêm AudioStreamPlayer để phát nhạc nền.

Ưu điểm của phương pháp tổ chức scene này là: mỗi scene độc lập và có thể được kiểm thử riêng; các NPC sử dụng instance của cùng một scene, sửa đổi một lần ảnh hưởng đến tất cả các NPC; các scene giao tiếp thông qua các tín hiệu (signal) với độ ghép nối thấp, dễ bảo trì và mở rộng.

### 15.5.2 Triển khai Điều khiển Người chơi

Nhân vật người chơi là một trong những phần tử quan trọng nhất trong game. Chúng ta cần triển khai điều khiển di chuyển WASD, chuyển đổi animation, phát hiện va chạm, tương tác với NPC, và hệ thống hiệu ứng âm thanh.

Cấu trúc scene player bao gồm: một CharacterBody2D làm node gốc, chịu trách nhiệm về chuyển động vật lý và va chạm; một AnimatedSprite2D hiển thị animation nhân vật; một CollisionShape2D định nghĩa hình dạng va chạm; một Camera2D theo dõi người chơi; hai AudioStreamPlayer lần lượt phát hiệu ứng âm thanh tương tác và hiệu ứng âm thanh đi bộ.

Script điều khiển người chơi `player.gd` triển khai logic di chuyển, tương tác và hiệu ứng âm thanh:

```python
extends CharacterBody2D

# Movement speed
@export var speed: float = 200.0

# Currently interactable NPC
var nearby_npc: Node = null

# Interaction state (disable movement during interaction)
var is_interacting: bool = false

# Node references
@onready var animated_sprite: AnimatedSprite2D = $AnimatedSprite2D
@onready var camera: Camera2D = $Camera2D

# Sound effect references
@onready var interact_sound: AudioStreamPlayer = null
@onready var running_sound: AudioStreamPlayer = null

# Walking sound effect state
var is_playing_running_sound: bool = false

func _ready():
    # Add to player group (important! NPCs need this group to identify player)
    add_to_group("player")

    # Get sound effect nodes (optional, won't error if doesn't exist)
    interact_sound = get_node_or_null("InteractSound")
    running_sound = get_node_or_null("RunningSound")

    # Enable camera
    camera.enabled = true

    # Play default animation
    if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
        animated_sprite.play("idle")

func _physics_process(_delta: float):
    # If interacting, disable movement
    if is_interacting:
        velocity = Vector2.ZERO
        move_and_slide()
        # Play idle animation
        if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")
        # Stop walking sound effect
        stop_running_sound()
        return

    # Get input direction
    var input_direction = Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")

    # Set velocity
    velocity = input_direction * speed

    # Move
    move_and_slide()

    # Update animation and direction
    update_animation(input_direction)

    # Update walking sound effect
    update_running_sound(input_direction)

func update_animation(direction: Vector2):
    """Update character animation (supports 4 directions)"""
    if animated_sprite.sprite_frames == null:
        return

    # Play animation based on movement direction
    if direction.length() > 0:
        # Moving - determine main direction
        if abs(direction.x) > abs(direction.y):
            # Left-right movement
            if direction.x > 0:
                # Right
                if animated_sprite.sprite_frames.has_animation("walk_right"):
                    animated_sprite.play("walk_right")
                    animated_sprite.flip_h = false
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
                    animated_sprite.flip_h = false
            else:
                # Left
                if animated_sprite.sprite_frames.has_animation("walk_left"):
                    animated_sprite.play("walk_left")
                    animated_sprite.flip_h = false
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
                    animated_sprite.flip_h = true
        else:
            # Up-down movement
            if direction.y > 0:
                # Down
                if animated_sprite.sprite_frames.has_animation("walk_down"):
                    animated_sprite.play("walk_down")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
            else:
                # Up
                if animated_sprite.sprite_frames.has_animation("walk_up"):
                    animated_sprite.play("walk_up")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
    else:
        # Idle
        if animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")

func _input(event: InputEvent):
    # Press E key to interact with NPC
    if event is InputEventKey:
        if event.pressed and not event.echo:
            if event.keycode == KEY_E or event.keycode == KEY_ENTER:
                if nearby_npc != null:
                    interact_with_npc()

func interact_with_npc():
    """Interact with nearby NPC"""
    if nearby_npc != null:
        # Play interaction sound effect
        if interact_sound:
            interact_sound.play()

        # Send signal to dialogue system
        get_tree().call_group("dialogue_system", "start_dialogue", nearby_npc.npc_name)

func set_nearby_npc(npc: Node):
    """Set nearby NPC"""
    nearby_npc = npc

func set_interacting(interacting: bool):
    """Set interaction state"""
    is_interacting = interacting
    if interacting:
        # Stop walking sound effect
        stop_running_sound()

func update_running_sound(direction: Vector2):
    """Update walking sound effect"""
    if running_sound == null:
        return

    # If moving
    if direction.length() > 0:
        # If sound effect not playing yet, start playing
        if not is_playing_running_sound:
            running_sound.play()
            is_playing_running_sound = true
    else:
        # If stopped moving, stop sound effect
        stop_running_sound()

func stop_running_sound():
    """Stop walking sound effect"""
    if running_sound and is_playing_running_sound:
        running_sound.stop()
        is_playing_running_sound = false
```

Script này triển khai điều khiển người chơi hoàn chỉnh. Người chơi sử dụng các phím WASD (hoặc phím mũi tên) để di chuyển, và nhân vật phát các animation 4 hướng tương ứng (walk_up/down/left/right) dựa trên hướng di chuyển. Khi người chơi đến gần một NPC, NPC gọi `set_nearby_npc()` để thiết lập chính nó thành một đối tượng có thể tương tác, và người chơi có thể nhấn phím E để kích hoạt tương tác. Trong quá trình tương tác, hiệu ứng âm thanh được phát, và `call_group()` thông báo cho hệ thống đối thoại bắt đầu cuộc trò chuyện. Trong khi đối thoại, `set_interacting(true)` vô hiệu hóa chuyển động của người chơi, được khôi phục sau khi đối thoại kết thúc. Hiệu ứng âm thanh đi bộ tự động phát khi người chơi di chuyển và tự động dừng khi dừng lại.

### 15.5.3 Hành vi và Tương tác NPC

NPC cần triển khai ba chức năng cốt lõi: đi tuần và lang thang ngẫu nhiên trong cảnh, phản hồi các tương tác của người chơi, và hiển thị bong bóng đối thoại. Chúng ta sử dụng Area2D để phát hiện xem người chơi có ở gần NPC hay không. Khi người chơi vào phạm vi tương tác, người chơi được thông báo, và nhấn phím E để bắt đầu cuộc trò chuyện.

Cấu trúc scene NPC bao gồm: CharacterBody2D làm node gốc; CollisionShape2D định nghĩa hình dạng va chạm của NPC; AnimatedSprite2D hiển thị animation NPC; InteractionArea (Area2D) phát hiện người chơi vào phạm vi tương tác, với CollisionShape2D bên dưới định nghĩa phạm vi tương tác; NameLabel hiển thị tên NPC; DialogueLabel hiển thị bong bóng đối thoại.

Script NPC `npc.gd` triển khai logic đi tuần, tương tác và bong bóng đối thoại:

```python
extends CharacterBody2D

# NPC information
@export var npc_name: String = "Zhang San"
@export var npc_title: String = "Python Engineer"

# NPC appearance configuration
@export var sprite_frames: SpriteFrames = null  # Custom sprite frame resource

# NPC movement configuration
@export var move_speed: float = 50.0  # Movement speed
@export var wander_enabled: bool = true  # Whether to enable patrol
@export var wander_range: float = 200.0  # Patrol range
@export var wander_interval_min: float = 3.0  # Minimum patrol interval (seconds)
@export var wander_interval_max: float = 8.0  # Maximum patrol interval (seconds)

# Current dialogue content (obtained from back-end)
var current_dialogue: String = ""

# Node references
@onready var animated_sprite: AnimatedSprite2D = $AnimatedSprite2D
@onready var interaction_area: Area2D = $InteractionArea
@onready var name_label: Label = $NameLabel
@onready var dialogue_label: Label = $DialogueLabel

# Player reference
var player: Node = null

# Patrol-related variables
var wander_target: Vector2 = Vector2.ZERO  # Patrol target position
var wander_timer: float = 0.0  # Patrol timer
var is_wandering: bool = false  # Whether currently patrolling
var is_interacting: bool = false  # Whether currently interacting with player
var spawn_position: Vector2 = Vector2.ZERO  # Spawn position

func _ready():
    # Add to npcs group
    add_to_group("npcs")

    # Set NPC name
    name_label.text = npc_name

    # Connect interaction area signals
    interaction_area.body_entered.connect(_on_body_entered)
    interaction_area.body_exited.connect(_on_body_exited)

    # Initialize dialogue label
    dialogue_label.text = ""
    dialogue_label.visible = false

    # Set custom sprite frames (if any)
    if sprite_frames != null:
        animated_sprite.sprite_frames = sprite_frames

    # Play default animation
    if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
        animated_sprite.play("idle")

    # Record spawn position
    spawn_position = global_position

    # Initialize patrol timer
    if wander_enabled:
        wander_timer = randf_range(wander_interval_min, wander_interval_max)
        choose_new_wander_target()

func _on_body_entered(body: Node2D):
    """Player enters interaction range"""
    if body.is_in_group("player"):
        player = body

        if player.has_method("set_nearby_npc"):
            player.set_nearby_npc(self)

func _on_body_exited(body: Node2D):
    """Player leaves interaction range"""
    if body.is_in_group("player"):
        if player != null and player.has_method("set_nearby_npc"):
            player.set_nearby_npc(null)
        player = null

func update_dialogue(dialogue: String):
    """Update NPC dialogue content"""
    current_dialogue = dialogue
    dialogue_label.text = dialogue
    dialogue_label.visible = true

    # Hide dialogue after 10 seconds
    await get_tree().create_timer(10.0).timeout
    dialogue_label.visible = false

func _physics_process(delta: float):
    """Physics update - handle movement"""
    # If interacting with player, stop movement
    if is_interacting:
        velocity = Vector2.ZERO
        move_and_slide()
        # Play idle animation
        if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")
        return

    # If patrol not enabled, don't move
    if not wander_enabled:
        return

    # Update patrol timer
    wander_timer -= delta

    # If timer ends, choose new target and start moving
    if wander_timer <= 0:
        choose_new_wander_target()
        wander_timer = randf_range(wander_interval_min, wander_interval_max)

    # If patrolling, move to target
    if is_wandering:
        # Check if reached target
        if global_position.distance_to(wander_target) < 10:
            # Reached target, stop movement
            is_wandering = false
            velocity = Vector2.ZERO
            move_and_slide()
            # Play idle animation
            if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
                animated_sprite.play("idle")
        else:
            # Continue moving to target
            var direction = (wander_target - global_position).normalized()
            velocity = direction * move_speed
            move_and_slide()
            # Update animation
            update_animation(direction)
    else:
        # Stop movement
        velocity = Vector2.ZERO
        move_and_slide()
        # Play idle animation
        if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")

func choose_new_wander_target():
    """Choose new patrol target"""
    # Randomly choose a point near spawn position
    var offset = Vector2(
        randf_range(-wander_range, wander_range),
        randf_range(-wander_range, wander_range)
    )
    wander_target = spawn_position + offset
    is_wandering = true

func update_animation(direction: Vector2):
    """Update animation"""
    if animated_sprite.sprite_frames == null:
        return

    if direction.length() > 0:
        # Movement animation
        if abs(direction.x) > abs(direction.y):
            # Left-right movement
            if direction.x > 0:
                if animated_sprite.sprite_frames.has_animation("walk_right"):
                    animated_sprite.play("walk_right")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
                    animated_sprite.flip_h = false
            else:
                if animated_sprite.sprite_frames.has_animation("walk_left"):
                    animated_sprite.play("walk_left")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
                    animated_sprite.flip_h = true
        else:
            # Up-down movement
            if direction.y > 0:
                if animated_sprite.sprite_frames.has_animation("walk_down"):
                    animated_sprite.play("walk_down")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
            else:
                if animated_sprite.sprite_frames.has_animation("walk_up"):
                    animated_sprite.play("walk_up")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
    else:
        # Idle animation
        if animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")

func set_interacting(interacting: bool):
    """Set interaction state"""
    is_interacting = interacting
```

Script này triển khai hành vi NPC hoàn chỉnh. NPC đi tuần ngẫu nhiên trong phạm vi `wander_range` xung quanh vị trí sinh ra của chúng, chọn một điểm mục tiêu mới và di chuyển đến đó sau mỗi `wander_interval_min` đến `wander_interval_max` giây. Trong khi di chuyển, các animation 4 hướng (walk_up/down/left/right) được phát, và khi đến mục tiêu, chúng dừng lại và phát animation idle. Khi người chơi vào InteractionArea, NPC gọi phương thức `set_nearby_npc(self)` của người chơi, thiết lập chính nó thành một đối tượng có thể tương tác. Sau khi người chơi nhấn phím E, hệ thống đối thoại gọi phương thức `set_interacting(true)` của NPC, và NPC dừng di chuyển. Sau khi đối thoại kết thúc, `set_interacting(false)` được gọi, và NPC tiếp tục đi tuần. Scene chính định kỳ gọi phương thức `update_dialogue()` để cập nhật bong bóng đối thoại của NPC, hiển thị nội dung đối thoại tự chủ giữa các NPC.

## 15.6 Triển khai Giao tiếp Front-End và Back-End

### 15.6.1 Đóng gói API Client

Front-end Godot cần giao tiếp với back-end FastAPI qua HTTP. Chúng ta tạo một script API client `api_client.gd`, đóng gói tất cả các lệnh gọi API, và thiết lập nó thành một singleton AutoLoad (tự động tải) để các script khác có thể sử dụng thuận tiện.

API client sử dụng node HTTPRequest của Godot để gửi các yêu cầu HTTP. HTTPRequest là một node bất đồng bộ không chặn game sau khi gửi yêu cầu, mà thông báo hoàn thành yêu cầu thông qua các signal. Điều này đảm bảo tính mượt mà của game - ngay cả khi độ trễ mạng cao, cũng không có hiện tượng giật lag. Chúng ta sử dụng cơ chế signal để thông báo cho các script khác về các phản hồi API thay vì sử dụng await, cho phép nhiều script đồng thời lắng nghe cùng một phản hồi API.

```python
# api_client.gd
extends Node

# Signal definitions
signal chat_response_received(npc_name: String, message: String)
signal chat_error(error_message: String)
signal npc_status_received(dialogues: Dictionary)
signal npc_list_received(npcs: Array)

# HTTP request nodes
var http_chat: HTTPRequest
var http_status: HTTPRequest
var http_npcs: HTTPRequest

func _ready():
    # Create HTTP request nodes
    http_chat = HTTPRequest.new()
    http_status = HTTPRequest.new()
    http_npcs = HTTPRequest.new()

    add_child(http_chat)
    add_child(http_status)
    add_child(http_npcs)

    # Connect signals
    http_chat.request_completed.connect(_on_chat_request_completed)
    http_status.request_completed.connect(_on_status_request_completed)
    http_npcs.request_completed.connect(_on_npcs_request_completed)

# ==================== Chat API ====================
func send_chat(npc_name: String, message: String) -> void:
    """Send chat request"""
    var data = {
        "npc_name": npc_name,
        "message": message
    }

    var json_string = JSON.stringify(data)
    var headers = ["Content-Type: application/json"]

    var error = http_chat.request(
        Config.API_CHAT,
        headers,
        HTTPClient.METHOD_POST,
        json_string
    )

    if error != OK:
        print("[ERROR] Failed to send chat request: ", error)
        chat_error.emit("Network request failed")

func _on_chat_request_completed(_result: int, response_code: int, _headers: PackedStringArray, body: PackedByteArray) -> void:
    """Handle chat response"""
    if response_code != 200:
        print("[ERROR] Chat request failed: HTTP ", response_code)
        chat_error.emit("Server error: " + str(response_code))
        return

    var json = JSON.new()
    var parse_result = json.parse(body.get_string_from_utf8())

    if parse_result != OK:
        print("[ERROR] Failed to parse response")
        chat_error.emit("Response parsing failed")
        return

    var response = json.data

    if response.has("success") and response["success"]:
        var npc_name = response["npc_name"]
        var msg = response["message"]
        print("[INFO] Received NPC reply: ", npc_name, " -> ", msg)
        chat_response_received.emit(npc_name, msg)
    else:
        chat_error.emit("Chat failed")

# ==================== NPC Status API ====================
func get_npc_status() -> void:
    """Get NPC status"""
    # Check if request is being processed
    if http_status.get_http_client_status() != HTTPClient.STATUS_DISCONNECTED:
        print("[WARN] NPC status request is being processed, skipping this request")
        return

    var error = http_status.request(Config.API_NPC_STATUS)

    if error != OK:
        print("[ERROR] Failed to get NPC status: ", error)

func _on_status_request_completed(_result: int, response_code: int, _headers: PackedStringArray, body: PackedByteArray) -> void:
    """Handle NPC status response"""
    if response_code != 200:
        print("[ERROR] NPC status request failed: HTTP ", response_code)
        return

    var json = JSON.new()
    var parse_result = json.parse(body.get_string_from_utf8())

    if parse_result != OK:
        print("[ERROR] Failed to parse NPC status")
        return

    var response = json.data

    if response.has("dialogues"):
        var dialogues = response["dialogues"]
        print("[INFO] Received NPC status update: ", dialogues.size(), " NPCs")
        npc_status_received.emit(dialogues)

# ==================== NPC List API ====================
func get_npc_list() -> void:
    """Get NPC list"""
    var error = http_npcs.request(Config.API_NPCS)

    if error != OK:
        print("[ERROR] Failed to get NPC list: ", error)

func _on_npcs_request_completed(_result: int, response_code: int, _headers: PackedStringArray, body: PackedByteArray) -> void:
    """Handle NPC list response"""
    if response_code != 200:
        print("[ERROR] NPC list request failed: HTTP ", response_code)
        return

    var json = JSON.new()
    var parse_result = json.parse(body.get_string_from_utf8())

    if parse_result != OK:
        print("[ERROR] Failed to parse NPC list")
        return

    var response = json.data

    if response.has("npcs"):
        var npcs = response["npcs"]
        print("[INFO] Received NPC list: ", npcs.size(), " NPCs")
        npc_list_received.emit(npcs)
```

API client này đóng gói ba chức năng cốt lõi: gửi yêu cầu chat (`send_chat`), lấy trạng thái NPC (`get_npc_status`), và lấy danh sách NPC (`get_npc_list`). Tất cả các yêu cầu HTTP đều bất đồng bộ, thông báo kết quả phản hồi thông qua các signal. Chúng ta đã tạo các node HTTPRequest độc lập cho mỗi API, cho phép nhiều yêu cầu được gửi đồng thời mà không ảnh hưởng lẫn nhau. Các URL API được lấy từ singleton Config để quản lý thống nhất thuận tiện. Hệ thống đối thoại lắng nghe signal `chat_response_received` để nhận phản hồi từ NPC, và scene chính lắng nghe signal `npc_status_received` để cập nhật bong bóng đối thoại của NPC.

### 15.6.2 Triển khai UI Đối thoại

UI đối thoại là giao diện cho tương tác giữa người chơi và NPC. Chúng ta cần thiết kế một hộp thoại đơn giản và đẹp mắt chứa tên NPC, chức danh, hiển thị nội dung đối thoại, hộp nhập liệu và các nút bấm.

Cấu trúc UI đối thoại được thể hiện trong Hình 15.13:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-13.png" alt="" width="85%"/>
  <p>Hình 15.13 Cấu trúc UI Đối thoại</p>
</div>

Thiết kế UI đối thoại rất đơn giản. DialogueUI là một node CanvasLayer, nghĩa là nó sẽ luôn hiển thị trên cùng của màn hình game và không bị che khuất bởi các đối tượng game khác. Panel là nền của hộp thoại, được neo ở dưới cùng của màn hình. Bên dưới Panel là 6 phần tử UI được đặt trực tiếp: NPCName hiển thị tên của NPC, NPCTitle hiển thị chức danh, DialogueText sử dụng RichTextLabel để hiển thị nội dung đối thoại (hỗ trợ định dạng rich text), PlayerInput là một LineEdit cho người chơi nhập liệu, và SendButton cùng CloseButton được dùng để gửi tin nhắn và đóng hộp thoại tương ứng.

Script UI đối thoại `dialogue_ui.gd` triển khai logic giao diện đối thoại:

```python
# dialogue_ui.gd
extends CanvasLayer

# UI node references
@onready var panel = $Panel
@onready var npc_name_label = $Panel/NPCName
@onready var npc_title_label = $Panel/NPCTitle
@onready var dialogue_text = $Panel/DialogueText
@onready var input_field = $Panel/PlayerInput
@onready var send_button = $Panel/SendButton
@onready var close_button = $Panel/CloseButton

# API client
var api_client: Node = null

# Current NPC in dialogue
var current_npc_name: String = ""

func _ready():
    # Hide dialogue box on initialization
    visible = false

    # Connect button signals
    send_button.pressed.connect(_on_send_button_pressed)
    close_button.pressed.connect(_on_close_button_pressed)
    input_field.text_submitted.connect(_on_text_submitted)

    # Get API client
    api_client = get_node_or_null("/root/APIClient")

func start_dialogue(npc_name: String):
    """Start dialogue with NPC"""
    current_npc_name = npc_name

    # Set NPC information
    npc_name_label.text = npc_name
    npc_title_label.text = get_npc_title(npc_name)

    # Clear dialogue content
    dialogue_text.clear()
    dialogue_text.append_text("[color=gray]Conversation with " + npc_name + " started...[/color]\n")

    # Clear input field
    input_field.text = ""

    # Show dialogue box
    show_dialogue()

    # Focus input field
    input_field.grab_focus()

func show_dialogue():
    """Show dialogue box"""
    visible = true

    # Notify player to enter interaction state (disable movement)
    var player = get_tree().get_first_node_in_group("player")
    if player and player.has_method("set_interacting"):
        player.set_interacting(true)

func hide_dialogue():
    """Hide dialogue box"""
    visible = false
    current_npc_name = ""

    # Notify player to exit interaction state (enable movement)
    var player = get_tree().get_first_node_in_group("player")
    if player and player.has_method("set_interacting"):
        player.set_interacting(false)

func _on_send_button_pressed():
    """Send button clicked"""
    send_message()

func _on_close_button_pressed():
    """Close button clicked"""
    hide_dialogue()

func _on_text_submitted(_text: String):
    """Input field enter pressed"""
    send_message()

func send_message():
    """Send message"""
    var message = input_field.text.strip_edges()

    if message.is_empty():
        return

    if current_npc_name.is_empty():
        return

    # Display player message
    dialogue_text.append_text("\n[color=cyan]Player:[/color] " + message + "\n")

    # Clear input field
    input_field.text = ""

    # Disable input
    input_field.editable = false
    send_button.disabled = true

    # Send API request
    if api_client:
        api_client.send_chat_request(current_npc_name, message)

func on_chat_response_received(npc_name: String, response: String):
    """Received NPC reply"""
    if npc_name == current_npc_name:
        # Display NPC reply
        dialogue_text.append_text("[color=yellow]" + npc_name + ":[/color] " + response + "\n")

        # Enable input
        input_field.editable = true
        send_button.disabled = false
        input_field.grab_focus()

func get_npc_title(npc_name: String) -> String:
    """Get NPC title"""
    var titles = {
        "Zhang San": "Python Engineer",
        "Li Si": "Product Manager",
        "Wang Wu": "UI Designer"
    }
    return titles.get(npc_name, "")
```

UI đối thoại này triển khai chức năng đối thoại hoàn chỉnh. Người chơi có thể nhập và gửi tin nhắn, và UI sử dụng phương thức append_text của RichTextLabel để hiển thị nội dung đối thoại, hỗ trợ định dạng rich text (màu sắc, in đậm, v.v.). Tất cả các lệnh gọi API đều bất đồng bộ, vô hiệu hóa hộp nhập liệu trong khi chờ phản hồi để ngăn gửi trùng lặp. Khi hộp thoại được hiển thị, nó thông báo cho người chơi vào trạng thái tương tác, vô hiệu hóa chuyển động, và khôi phục chuyển động khi đóng lại.

### 15.6.3 Tích hợp Scene Chính

Cuối cùng, chúng ta cần tích hợp tất cả các chức năng trong scene chính: điều khiển người chơi, tương tác NPC, UI đối thoại, và cập nhật trạng thái NPC. Script scene chính `main.gd` điều phối các thành phần này và định kỳ lấy trạng thái NPC từ back-end để cập nhật bong bóng đối thoại của NPC.

```python
# main.gd
extends Node2D

# NPC node references
@onready var npc_zhang: Node2D = $NPCs/NPC_Zhang
@onready var npc_li: Node2D = $NPCs/NPC_Li
@onready var npc_wang: Node2D = $NPCs/NPC_Wang

# API client
var api_client: Node = null

# NPC status update timer
var status_update_timer: float = 0.0

func _ready():
    print("[INFO] Main scene initialization")

    # Get API client
    api_client = get_node_or_null("/root/APIClient")
    if api_client:
        api_client.npc_status_received.connect(_on_npc_status_received)

        # Immediately get NPC status once
        api_client.get_npc_status()
    else:
        print("[ERROR] API client not found")

func _process(delta: float):
    # Periodically update NPC status
    status_update_timer += delta
    if status_update_timer >= Config.NPC_STATUS_UPDATE_INTERVAL:
        status_update_timer = 0.0
        if api_client:
            api_client.get_npc_status()

func _on_npc_status_received(dialogues: Dictionary):
    """Received NPC status update"""
    print("[INFO] Update NPC status: ", dialogues)

    # Update each NPC's dialogue
    for npc_name in dialogues:
        var dialogue = dialogues[npc_name]
        update_npc_dialogue(npc_name, dialogue)

func update_npc_dialogue(npc_name: String, dialogue: String):
    """Update specified NPC's dialogue"""
    var npc_node = get_npc_node(npc_name)
    if npc_node and npc_node.has_method("update_dialogue"):
        npc_node.update_dialogue(dialogue)

func get_npc_node(npc_name: String) -> Node2D:
    """Get NPC node by name"""
    match npc_name:
        "Zhang San":
            return npc_zhang
        "Li Si":
            return npc_li
        "Wang Wu":
            return npc_wang
        _:
            return null
```

Chức năng cốt lõi của script scene chính là định kỳ lấy trạng thái NPC từ back-end. Trong `_ready()`, chúng ta lấy một tham chiếu đến singleton APIClient và kết nối signal `npc_status_received`. Sau đó chúng ta ngay lập tức gọi `get_npc_status()` để lấy trạng thái NPC một lần. Trong `_process()`, chúng ta sử dụng một bộ đếm thời gian để gọi `get_npc_status()` sau mỗi `Config.NPC_STATUS_UPDATE_INTERVAL` giây (mặc định 30 giây). Khi nhận được cập nhật trạng thái NPC, hàm callback `_on_npc_status_received()` duyệt qua tất cả các NPC và gọi phương thức `update_dialogue()` của chúng để cập nhật bong bóng đối thoại. Bằng cách này, ngay cả khi người chơi không tương tác với NPC, họ vẫn có thể thấy đối thoại tự chủ giữa các NPC.

Quy trình giao tiếp front-end và back-end hoàn chỉnh được thể hiện trong Hình 15.14:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-14.png" alt="" width="85%"/>
  <p>Hình 15.14 Quy trình Giao tiếp Front-End và Back-End Hoàn chỉnh</p>
</div>

Đến đây, tất cả các chức năng giao tiếp front-end và back-end đã được triển khai. Người chơi có thể di chuyển tự do trong game, tương tác với NPC, và trò chuyện bằng ngôn ngữ tự nhiên. Đồng thời, scene chính định kỳ lấy trạng thái NPC từ back-end, cập nhật bong bóng đối thoại của NPC, và hiển thị đối thoại tự chủ giữa các NPC. Toàn bộ hệ thống sử dụng cơ chế signal để giao tiếp, với sự liên kết lỏng lẻo giữa các thành phần, giúp dễ dàng bảo trì và mở rộng.

## 15.7 Tổng kết và Triển vọng

### 15.7.1 Ôn tập Chương

Trong chương này, chúng ta đã hoàn thành một dự án thị trấn AI đầy đủ - Thị trấn Cyber (Cyber Town). Dự án này kết hợp framework HelloAgents với game engine Godot để tạo ra một thế giới ảo sống động. Hãy cùng ôn lại nội dung cốt lõi mà chúng ta đã học.

**Thiết kế Kiến trúc Kỹ thuật**

Chúng ta đã áp dụng một kiến trúc tách biệt gồm game engine + dịch vụ back-end, tách biệt việc render front-end, logic back-end, và trí tuệ AI thành các tầng khác nhau. Godot xử lý đồ họa game và tương tác người chơi, FastAPI xử lý các dịch vụ API và quản lý trạng thái, và HelloAgents xử lý trí tuệ NPC và hệ thống bộ nhớ. Thiết kế phân tầng này cho phép mỗi phần được phát triển và kiểm thử độc lập, đồng thời cũng cung cấp một nền tảng tốt cho việc mở rộng trong tương lai.

**Hệ thống Agent NPC**

Chúng ta đã sử dụng SimpleAgent của HelloAgents để tạo các agent độc lập cho mỗi NPC. Mỗi NPC có thiết lập vai trò, đặc điểm tính cách, và hệ thống bộ nhớ riêng. Thông qua các system prompt được thiết kế cẩn thận, chúng ta đã biến Zhang San thành một kỹ sư Python nghiêm túc, Li Si thành một product manager giỏi giao tiếp, và Wang Wu thành một nhà thiết kế UI đầy sáng tạo. Những NPC này không chỉ có thể hiểu đối thoại của người chơi mà còn phản hồi theo đặc điểm vai trò của chúng.

**Hệ thống Bộ nhớ và Thiện cảm**

Chúng ta đã triển khai một hệ thống bộ nhớ hai tầng: bộ nhớ ngắn hạn duy trì tính mạch lạc của đối thoại, và bộ nhớ dài hạn lưu trữ toàn bộ lịch sử tương tác. Thông qua truy xuất ngữ nghĩa trong vector database, các NPC có thể nhớ lại các chủ đề đã thảo luận trước đó. Hệ thống thiện cảm cho phép thái độ của NPC đối với người chơi thay đổi theo tương tác, từ người lạ đến bạn thân, với các biểu hiện hành vi khác nhau ở mỗi cấp độ. Những thiết kế này khiến các NPC trở nên chân thực và thú vị hơn.

**Xây dựng Scene Game**

Chúng ta đã sử dụng Godot để tạo ra một scene văn phòng phong cách pixel, triển khai điều khiển người chơi, đi lang thang của NPC, phát hiện tương tác, và UI đối thoại. Thông qua thiết kế mô-đun của hệ thống scene, chúng ta có thể dễ dàng thêm NPC mới, scene mới, và chức năng mới. Cú pháp ngắn gọn của GDScript khiến việc triển khai logic game trở nên trực quan và hiệu quả.

**Giao tiếp Front-End và Back-End**

Chúng ta đã sử dụng HTTP REST API để triển khai giao tiếp giữa front-end Godot và back-end FastAPI. Thông qua các yêu cầu bất đồng bộ và hệ thống signal, chúng ta đảm bảo tính mượt mà của game - ngay cả khi độ trễ mạng cao, trải nghiệm người chơi cũng không bị ảnh hưởng. Việc đóng gói API client cho phép các script khác gọi các dịch vụ back-end một cách thuận tiện, và việc triển khai UI đối thoại cho phép người chơi giao tiếp tự nhiên với các NPC.

Công nghệ (technology stack) của dự án được thể hiện trong Hình 15.15:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-15.png" alt="" width="85%"/>
  <p>Hình 15.15 Technology Stack của Thị trấn Cyber</p>
</div>

### 15.7.2 Hướng Mở rộng

Thị trấn Cyber chỉ là một điểm khởi đầu - có rất nhiều hướng để mở rộng. Những mở rộng này không chỉ có thể nâng cao sự thú vị của game mà còn khám phá thêm nhiều khả năng của công nghệ AI trong game.

**(1) Hỗ trợ Nhiều người chơi Trực tuyến**

Hiện tại, Thị trấn Cyber là một game một người chơi, nhưng chúng ta có thể mở rộng nó thành một game nhiều người chơi trực tuyến. Nhiều người chơi có thể đồng thời vào cùng một văn phòng và tương tác với các NPC và những người chơi khác. Điều này đòi hỏi phải giới thiệu WebSocket cho giao tiếp thời gian thực và database để lưu trữ dữ liệu người chơi và trạng thái NPC. Các NPC có thể nhớ các tương tác với những người chơi khác nhau và duy trì mức độ thiện cảm độc lập cho mỗi người chơi.

**(2) Hệ thống Nhiệm vụ**

Chúng ta có thể thiết kế một hệ thống nhiệm vụ cho các NPC. Khi thiện cảm của người chơi với một NPC đạt đến một mức độ nhất định, NPC sẽ cung cấp các nhiệm vụ đặc biệt. Ví dụ, Zhang San có thể yêu cầu người chơi giúp gỡ lỗi code, Li Si có thể yêu cầu người chơi thu thập phản hồi người dùng, và Wang Wu có thể yêu cầu người chơi đánh giá các đề xuất thiết kế. Hoàn thành nhiệm vụ có thể nhận được phần thưởng và tăng thêm thiện cảm.

**(3) Tương tác NPC với NPC**

Hiện tại, các NPC chỉ tương tác với người chơi, nhưng chúng ta có thể cho phép các NPC tương tác với nhau. Zhang San có thể thảo luận yêu cầu sản phẩm với Li Si, Li Si có thể thảo luận thiết kế giao diện với Wang Wu, và Wang Wu có thể thảo luận triển khai kỹ thuật với Zhang San. Những tương tác này có thể diễn ra tự động ở nền, và người chơi có thể quan sát đối thoại giữa các NPC, khiến toàn bộ thế giới trở nên sống động hơn.

**(4) Hệ thống Cảm xúc**

Ngoài thiện cảm, chúng ta có thể thêm một hệ thống cảm xúc phức tạp hơn cho các NPC. Các NPC có thể có các trạng thái cảm xúc khác nhau như vui vẻ, buồn bã, tức giận, và phấn khích, những trạng thái này ảnh hưởng đến phong cách phản hồi và hành vi của NPC. Ví dụ, khi một NPC đang có tâm trạng tốt, họ sẽ sẵn lòng chia sẻ thông tin hơn; khi tâm trạng không tốt, họ có thể khá lạnh lùng.

**(5) Hệ thống Sự kiện Động**

Chúng ta có thể thiết kế các sự kiện động để làm cho thế giới game phong phú hơn. Ví dụ, định kỳ tổ chức các cuộc họp nhóm nơi tất cả các NPC và người chơi tụ họp để thảo luận tiến độ dự án; hoặc tổ chức các bữa tiệc sinh nhật mừng sinh nhật của một NPC; hoặc các nhiệm vụ khẩn cấp đòi hỏi sự hợp tác của mọi người. Những sự kiện này có thể tăng sự đa dạng và thú vị của game.

**(6) Thế giới Lớn hơn**

Hiện tại, Thị trấn Cyber chỉ có một scene văn phòng, nhưng chúng ta có thể mở rộng thành một thế giới lớn hơn. Chúng ta có thể thêm các scene khác nhau như quán cà phê, thư viện, và công viên, mỗi scene có các NPC và phương thức tương tác khác nhau. Người chơi có thể di chuyển giữa các scene khác nhau và khám phá một thế giới ảo rộng lớn hơn.

**(7) Học tập Cá nhân hóa**

Các NPC có thể học các sở thích và thói quen của mỗi người chơi. Ví dụ, nếu một người chơi thường xuyên thảo luận về Python với Zhang San, NPC sẽ nhớ rằng người chơi quan tâm đến lập trình và sẽ chủ động chia sẻ nội dung liên quan trong tương lai. Nếu một người chơi thích chơi game vào ban đêm, NPC sẽ nhớ thói quen thời gian này và trở nên tích cực hơn vào ban đêm.

### 15.7.3 Suy ngẫm và Triển vọng

Thị trấn Cyber thể hiện tiềm năng to lớn của công nghệ AI trong game. Các NPC trong game truyền thống bị giới hạn bởi các cây đối thoại và kịch bản định sẵn, trong khi các NPC AI có thể hiểu và tạo ra ngôn ngữ tự nhiên, có những cuộc trò chuyện thực sự với người chơi. Điều này không chỉ nâng cao sự đắm chìm của game mà còn mang lại những khả năng mới cho thiết kế game.

Tuy nhiên, các NPC AI cũng đối mặt với một số thách thức. Đầu tiên là vấn đề chi phí - mỗi cuộc trò chuyện đòi hỏi gọi API của LLM, điều này phát sinh một khoản phí nhất định. Đối với các game nhiều người chơi trực tuyến lớn, chi phí này có thể rất cao. Thứ hai là vấn đề độ trễ - suy luận LLM cần thời gian, và nếu độ trễ mạng cao, người chơi có thể phải chờ vài giây để thấy phản hồi của NPC. Cuối cùng, có vấn đề kiểm soát nội dung - nội dung do LLM tạo ra có thể không hoàn toàn kiểm soát được, đòi hỏi các prompt được thiết kế tốt và các cơ chế lọc nội dung.

Bất chấp những thách thức này, tương lai của các NPC AI vẫn đầy hứa hẹn. Khi công nghệ LLM phát triển, tốc độ suy luận sẽ nhanh hơn và chi phí sẽ thấp hơn. Các LLM nhỏ cục bộ (localized) cũng đang phát triển nhanh chóng - trong tương lai, chúng có thể chạy trực tiếp trên thiết bị của người chơi, hoàn toàn không cần yêu cầu mạng. Sự kết hợp giữa công nghệ AI và game sẽ mang lại cho người chơi những trải nghiệm chưa từng có.

Trong chương dự án tốt nghiệp của Phần 5, chúng ta sẽ học cách xây dựng các agent tổng quát bằng cách sử dụng single-agent và multi-agent - đây sẽ là thời gian sáng tạo của bạn, hãy đón chờ!
