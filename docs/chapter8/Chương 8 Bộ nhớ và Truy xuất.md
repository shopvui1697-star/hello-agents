# Chương 8 Bộ nhớ và Truy xuất

Trong các chương trước, chúng ta đã xây dựng kiến trúc cơ bản của framework HelloAgents, hiện thực hóa nhiều mô hình (paradigm) agent khác nhau cùng hệ thống công cụ (tool). Tuy nhiên, framework của chúng ta vẫn còn thiếu một năng lực then chốt: **bộ nhớ (memory)**. Nếu một agent không thể ghi nhớ các tương tác trước đó hoặc học hỏi từ kinh nghiệm lịch sử, hiệu năng của nó sẽ bị hạn chế rất nhiều trong các cuộc hội thoại liên tục hoặc các tác vụ phức tạp.

Chương này sẽ bổ sung hai năng lực cốt lõi cho HelloAgents dựa trên framework đã xây dựng ở Chương 7: **Hệ thống Bộ nhớ (Memory System)** và **Sinh tăng cường bằng truy xuất (Retrieval-Augmented Generation - RAG)**. Chúng ta sẽ áp dụng phương pháp "mở rộng framework + phổ cập kiến thức", tìm hiểu sâu nền tảng lý thuyết của Memory và RAG trong quá trình xây dựng, và cuối cùng hiện thực hóa một hệ thống agent với năng lực ghi nhớ và truy xuất tri thức hoàn chỉnh.


## 8.1 Từ Khoa học Nhận thức đến Bộ nhớ của Agent

### 8.1.1 Cảm hứng từ Hệ thống Bộ nhớ của Con người

Trước khi xây dựng hệ thống bộ nhớ cho agent, hãy cùng tìm hiểu từ góc độ khoa học nhận thức về cách con người xử lý và lưu trữ thông tin. Bộ nhớ của con người là một hệ thống nhận thức đa cấp, không chỉ lưu trữ thông tin mà còn phân loại và tổ chức thông tin dựa trên tầm quan trọng, thời gian và ngữ cảnh. Tâm lý học nhận thức cung cấp một khung lý thuyết kinh điển để hiểu cấu trúc và các tiến trình của bộ nhớ<sup>[1]</sup>, như minh họa trong Hình 8.1.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-1.png" alt="Human Memory System Structure" width="85%"/>
  <p>Hình 8.1 Cấu trúc phân cấp của Hệ thống Bộ nhớ Con người</p>
</div>

Theo nghiên cứu của tâm lý học nhận thức, bộ nhớ của con người có thể được chia thành các cấp độ sau:

1. **Bộ nhớ giác quan (Sensory Memory)**: Thời gian tồn tại rất ngắn (0.5-3 giây), dung lượng khổng lồ, chịu trách nhiệm tạm thời lưu trữ toàn bộ thông tin mà các giác quan tiếp nhận
2. **Bộ nhớ làm việc (Working Memory)**: Thời gian tồn tại ngắn (15-30 giây), dung lượng hạn chế (7±2 mục), chịu trách nhiệm xử lý thông tin trong tác vụ hiện tại
3. **Bộ nhớ dài hạn (Long-term Memory)**: Thời gian tồn tại lâu (có thể kéo dài cả đời), dung lượng gần như không giới hạn, được chia nhỏ thêm thành:
   - **Bộ nhớ thủ tục (Procedural Memory)**: Kỹ năng và thói quen (chẳng hạn như đi xe đạp)
   - **Bộ nhớ khai báo (Declarative Memory)**: Tri thức có thể diễn đạt bằng ngôn ngữ, được chia nhỏ thêm thành:
     - **Bộ nhớ ngữ nghĩa (Semantic Memory)**: Tri thức và khái niệm chung (chẳng hạn như "Paris là thủ đô của Pháp")
     - **Bộ nhớ tình tiết (Episodic Memory)**: Trải nghiệm và sự kiện cá nhân (chẳng hạn như "nội dung cuộc họp hôm qua")

### 8.1.2 Tại sao Agent cần Memory và RAG

Dựa trên thiết kế của hệ thống bộ nhớ con người, chúng ta có thể hiểu tại sao agent cũng cần những năng lực ghi nhớ tương tự. Một đặc trưng quan trọng của trí thông minh con người là khả năng ghi nhớ những trải nghiệm trong quá khứ, học hỏi từ chúng, và áp dụng những kinh nghiệm này vào các tình huống mới. Tương tự, một agent thực sự thông minh cũng cần có năng lực ghi nhớ. Đối với các agent dựa trên LLM, chúng thường đối mặt với hai hạn chế cơ bản: **quên trạng thái hội thoại** và **giới hạn của tri thức tích hợp sẵn**.

(1) Hạn chế 1: Quên hội thoại do tính phi trạng thái (statelessness)

Các mô hình ngôn ngữ lớn hiện nay, dù mạnh mẽ, được thiết kế theo hướng **phi trạng thái (stateless)**. Điều này có nghĩa là mỗi yêu cầu của người dùng (hoặc mỗi lệnh gọi API) là một phép tính độc lập, không liên quan đến nhau. Bản thân mô hình không tự động "ghi nhớ" nội dung của cuộc hội thoại trước đó. Điều này gây ra một số vấn đề:

1. **Mất ngữ cảnh (Context Loss)**: Trong các cuộc hội thoại dài, thông tin quan trọng ở giai đoạn đầu có thể bị mất do giới hạn cửa sổ ngữ cảnh (context window)
2. **Thiếu cá nhân hóa**: Agent không thể ghi nhớ sở thích, thói quen hoặc nhu cầu cụ thể của người dùng
3. **Năng lực học hỏi hạn chế**: Không thể học hỏi và cải thiện từ những thành công hay thất bại trong quá khứ
4. **Vấn đề nhất quán**: Có thể đưa ra các câu trả lời mâu thuẫn trong hội thoại nhiều lượt

Hãy cùng hiểu vấn đề này qua một ví dụ cụ thể:

```python
# How to use Agent from Chapter 7
from hello_agents import SimpleAgent, HelloAgentsLLM

agent = SimpleAgent(name="Learning Assistant", llm=HelloAgentsLLM())

# First conversation
response1 = agent.run("My name is Zhang San, I'm learning Python and have mastered basic syntax")
print(response1)  # "Great! Python basic syntax is an important foundation for programming..."
 
# Second conversation (new session, such as after restarting the program and creating a new Agent)
agent = SimpleAgent(name="Learning Assistant", llm=HelloAgentsLLM())
response2 = agent.run("Do you remember my learning progress?")
print(response2)  # "Sorry, I don't know your learning progress..."
```

Lưu ý rằng `SimpleAgent` từ Chương 7 lưu tạm cuộc hội thoại hiện tại trong `_history` bên trong cùng một thực thể (instance), do đó các lượt liên tiếp trong cùng một tiến trình và cùng một thực thể có thể mang theo ngữ cảnh gần đây. Tuy nhiên, lịch sử này chỉ là một danh sách tin nhắn tạm thời. Nó không được lưu bền vững (persist) qua các phiên và không hỗ trợ truy xuất dài hạn, quên (forgetting) hay củng cố (consolidation).

Để giải quyết vấn đề này, framework của chúng ta cần đưa vào một hệ thống bộ nhớ.

(2) Hạn chế 2: Giới hạn của tri thức tích hợp sẵn trong mô hình

Ngoài việc quên lịch sử hội thoại, một hạn chế cốt lõi khác của LLM là tri thức của chúng mang tính **tĩnh và hữu hạn**. Tri thức này hoàn toàn đến từ dữ liệu huấn luyện, kéo theo một loạt vấn đề:

1. **Tính thời sự của tri thức**: Các mô hình lớn có một mốc thời gian cắt (cutoff date) của dữ liệu huấn luyện và không thể truy cập thông tin mới nhất
2. **Tri thức chuyên biệt theo lĩnh vực**: Các mô hình tổng quát có thể thiếu độ sâu cần thiết trong những lĩnh vực cụ thể
3. **Độ chính xác về mặt sự kiện**: Giảm ảo giác (hallucination) của mô hình thông qua việc xác minh bằng truy xuất
4. **Khả năng diễn giải (Explainability)**: Cung cấp nguồn thông tin để tăng độ tin cậy của câu trả lời

Để vượt qua hạn chế này, công nghệ RAG ra đời. Ý tưởng cốt lõi của nó là truy xuất thông tin liên quan nhất từ một cơ sở tri thức bên ngoài (chẳng hạn như tài liệu, cơ sở dữ liệu, API) trước khi mô hình sinh câu trả lời, và cung cấp thông tin này cho mô hình dưới dạng ngữ cảnh.

### 8.1.3 Thiết kế Kiến trúc Hệ thống Memory và RAG

Dựa trên nền tảng framework đã thiết lập ở Chương 7 và cảm hứng từ khoa học nhận thức, chúng ta đã thiết kế một kiến trúc hệ thống memory và RAG phân lớp, như minh họa trong Hình 8.2. Kiến trúc này không chỉ tham khảo cấu trúc phân cấp của hệ thống bộ nhớ con người mà còn cân nhắc đầy đủ tính mở rộng của việc triển khai kỹ thuật. Trong triển khai, chúng ta thiết kế memory và RAG thành hai công cụ độc lập: `memory_tool` chịu trách nhiệm lưu trữ và duy trì thông tin tương tác trong quá trình hội thoại, còn `rag_tool` chịu trách nhiệm truy xuất thông tin liên quan từ các cơ sở tri thức do người dùng cung cấp làm ngữ cảnh và có thể tự động lưu các kết quả truy xuất quan trọng vào hệ thống bộ nhớ.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-2.png" alt="HelloAgents Memory and RAG System Architecture" width="95%"/>
  <p>Hình 8.2 Kiến trúc tổng thể của Hệ thống Memory và RAG của HelloAgents</p>
</div>

Hệ thống bộ nhớ áp dụng thiết kế kiến trúc bốn lớp:

```
HelloAgents Memory System
├── Infrastructure Layer
│   ├── MemoryManager - Memory manager (unified scheduling and coordination)
│   ├── MemoryItem - Memory data structure (standardized memory items)
│   ├── MemoryConfig - Configuration management (system parameter settings)
│   └── BaseMemory - Memory base class (common interface definition)
├── Memory Types Layer
│   ├── WorkingMemory - Working memory (temporary information, TTL management)
│   ├── EpisodicMemory - Episodic memory (specific events, time series)
│   ├── SemanticMemory - Semantic memory (abstract knowledge, graph relationships)
│   └── PerceptualMemory - Perceptual memory (multimodal data)
├── Storage Backend Layer
│   ├── QdrantVectorStore - Vector storage (high-performance semantic retrieval)
│   ├── Neo4jGraphStore - Graph storage (knowledge graph management)
│   └── SQLiteDocumentStore - Document storage (structured persistence)
└── Embedding Service Layer
    ├── DashScopeEmbedding - Tongyi Qianwen embedding (cloud API)
    ├── LocalTransformerEmbedding - Local embedding (offline deployment)
    └── TFIDFEmbedding - TFIDF embedding (lightweight fallback)
```

Hệ thống RAG tập trung vào việc thu nhận và sử dụng tri thức bên ngoài:

```
HelloAgents RAG System
├── Document Processing Layer
│   ├── DocumentProcessor - Document processor (multi-format parsing)
│   ├── Document - Document object (metadata management)
│   └── Pipeline - RAG pipeline (end-to-end processing)
├── Embedding Layer
│   └── Unified Embedding Interface - Reuses memory system's embedding service
├── Vector Storage Layer
│   └── QdrantVectorStore - Vector database (namespace isolation)
└── Intelligent Q&A Layer
    ├── Multi-strategy Retrieval - Vector retrieval + MQE + HyDE
    ├── Context Construction - Intelligent fragment merging and truncation
    └── LLM-Enhanced Generation - Accurate Q&A based on context
```

### 8.1.4 Mục tiêu Học tập và Trải nghiệm Nhanh

Trước tiên hãy cùng xem nội dung học tập cốt lõi của Chương 8:

```
hello-agents/
├── hello_agents/
│   ├── memory/                   # Memory system module
│   │   ├── base.py               # Basic data structures (MemoryItem, MemoryConfig, BaseMemory)
│   │   ├── manager.py            # Memory manager (unified coordination and scheduling)
│   │   ├── embedding.py          # Unified embedding service (DashScope/Local/TFIDF)
│   │   ├── types/                # Memory type implementations
│   │   │   ├── working.py        # Working memory (TTL management, pure in-memory)
│   │   │   ├── episodic.py       # Episodic memory (event sequence, SQLite+Qdrant)
│   │   │   ├── semantic.py       # Semantic memory (knowledge graph, Qdrant+Neo4j)
│   │   │   └── perceptual.py     # Perceptual memory (multimodal, SQLite+Qdrant)
│   │   ├── storage/              # Storage backend implementations
│   │   │   ├── qdrant_store.py   # Qdrant vector storage (high-performance vector retrieval)
│   │   │   ├── neo4j_store.py    # Neo4j graph storage (knowledge graph management)
│   │   │   └── document_store.py # SQLite document storage (structured persistence)
│   │   └── rag/                  # RAG system
│   │       ├── pipeline.py       # RAG pipeline (end-to-end processing)
│   │       └── document.py       # Document processor (multi-format parsing)
│   └── tools/builtin/            # Extended built-in tools
│       ├── memory_tool.py        # Memory tool (Agent memory capability)
│       └── rag_tool.py           # RAG tool (intelligent Q&A capability)
└──
```

**Bắt đầu Nhanh: Cài đặt Framework HelloAgents**

Để bạn đọc có thể nhanh chóng trải nghiệm đầy đủ chức năng của chương này, chúng tôi cung cấp một gói Python có thể cài đặt trực tiếp. Bạn có thể cài phiên bản tương ứng với chương này bằng các lệnh sau:

```bash
# If you encounter model unavailability in version 0.2.0, please refer to issue#320 or switch to version 0.2.9 for testing.
pip install "hello-agents[all]==0.2.0"
python -m spacy download zh_core_web_sm
python -m spacy download en_core_web_sm
```

Ngoài ra, bạn cần cấu hình API cho cơ sở dữ liệu đồ thị (graph database), cơ sở dữ liệu vector (vector database), LLM và giải pháp Embedding trong file `.env`. Trong hướng dẫn này, Qdrant được dùng cho cơ sở dữ liệu vector, Neo4J cho cơ sở dữ liệu đồ thị, và nền tảng Bailian được ưu tiên cho Embedding. Nếu không có API nào khả dụng, bạn có thể chuyển sang giải pháp mô hình triển khai cục bộ (local deployment).

```bash
# ================================
# Qdrant Vector Database Configuration - Get API key: https://cloud.qdrant.io/
# ================================
# Use Qdrant cloud service (recommended)
QDRANT_URL=https://your-cluster.qdrant.tech:6333
QDRANT_API_KEY=your_qdrant_api_key_here

# Or use local Qdrant (requires Docker)
# QDRANT_URL=http://localhost:6333
# QDRANT_API_KEY=

# Qdrant collection configuration
QDRANT_COLLECTION=hello_agents_vectors
QDRANT_VECTOR_SIZE=384
QDRANT_DISTANCE=cosine
QDRANT_TIMEOUT=30

# ================================
# Neo4j Graph Database Configuration - Get API key: https://neo4j.com/cloud/aura/
# ================================
# Use Neo4j Aura cloud service (recommended)
NEO4J_URI=neo4j+s://your-instance.databases.neo4j.io
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=your_neo4j_password_here

# Or use local Neo4j (requires Docker)
# NEO4J_URI=bolt://localhost:7687
# NEO4J_USERNAME=neo4j
# NEO4J_PASSWORD=hello-agents-password

# Neo4j connection configuration
NEO4J_DATABASE=neo4j
NEO4J_MAX_CONNECTION_LIFETIME=3600
NEO4J_MAX_CONNECTION_POOL_SIZE=50
NEO4J_CONNECTION_TIMEOUT=60

# ==========================
# Embedding Configuration Example - Get from Alibaba Cloud Console: https://dashscope.aliyun.com/
# ==========================
# - If empty, dashscope defaults to text-embedding-v3; local defaults to sentence-transformers/all-MiniLM-L6-v2
EMBED_MODEL_TYPE=dashscope
EMBED_MODEL_NAME=
EMBED_API_KEY=
EMBED_BASE_URL=
```

Việc học trong chương này có thể thực hiện theo hai cách:

1. **Học qua trải nghiệm (Experiential Learning)**: Cài đặt trực tiếp framework bằng `pip`, chạy mã ví dụ, và nhanh chóng trải nghiệm các chức năng khác nhau
2. **Học chuyên sâu (Deep Learning)**: Theo nội dung của chương, hiện thực hóa từng thành phần từ đầu, và hiểu sâu triết lý thiết kế cũng như chi tiết triển khai của framework

Chúng tôi khuyến nghị áp dụng lộ trình học "trải nghiệm trước, hiện thực hóa sau". Trong chương này, chúng tôi cung cấp các file kiểm thử (test) hoàn chỉnh. Bạn có thể viết lại các hàm cốt lõi và chạy test để xác minh xem cách triển khai của mình có đúng hay không.

Theo các nguyên tắc thiết kế đã thiết lập ở Chương 7, chúng ta đóng gói năng lực memory và RAG thành các công cụ (tool) chuẩn thay vì tạo ra các lớp Agent mới. Trước khi bắt đầu, hãy dành 30 giây để trải nghiệm việc xây dựng một agent với năng lực memory và RAG bằng Hello-agents!

```python
# Configure the LLM API in .env in the same folder
from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.tools import MemoryTool, RAGTool

# Create LLM instance
llm = HelloAgentsLLM()

# Create Agent
agent = SimpleAgent(
    name="Intelligent Assistant",
    llm=llm,
    system_prompt="You are an AI assistant with memory and knowledge retrieval capabilities"
)

# Create tool registry
tool_registry = ToolRegistry()

# Add memory tool
memory_tool = MemoryTool(user_id="user123")
tool_registry.register_tool(memory_tool)

# Add RAG tool
rag_tool = RAGTool(knowledge_base_path="./knowledge_base")
tool_registry.register_tool(rag_tool)

# Configure tools for Agent
agent.tool_registry = tool_registry

# Start conversation
response = agent.run("Hello! Please remember my name is Zhang San, I am a Python developer")
print(response)
```

Nếu mọi thứ được cấu hình đúng, bạn có thể thấy nội dung sau:

```bash
[OK] SQLite database tables and indexes created
[OK] SQLite document storage initialized: ./memory_data\memory.db
INFO:hello_agents.memory.storage.qdrant_store:✅ Successfully connected to Qdrant cloud service: https://0c517275-2ad0-4442-8309-11c36dc7e811.us-east-1-1.aws.cloud.qdrant.io:6333
INFO:hello_agents.memory.storage.qdrant_store:✅ Using existing Qdrant collection: hello_agents_vectors
INFO:hello_agents.memory.types.semantic:✅ Embedding model ready, dimension: 1024
INFO:hello_agents.memory.types.semantic:✅ Qdrant vector database initialization complete
INFO:hello_agents.memory.storage.neo4j_store:✅ Successfully connected to Neo4j cloud service: neo4j+s://851b3a28.databases.neo4j.io
INFO:hello_agents.memory.types.semantic:✅ Neo4j graph database initialization complete
INFO:hello_agents.memory.storage.neo4j_store:✅ Neo4j index creation complete
INFO:hello_agents.memory.types.semantic:✅ Neo4j graph database initialization complete
INFO:hello_agents.memory.types.semantic:🏥 Database health status: Qdrant=✅, Neo4j=✅
INFO:hello_agents.memory.types.semantic:✅ Loaded Chinese spaCy model: zh_core_web_sm
INFO:hello_agents.memory.types.semantic:✅ Loaded English spaCy model: en_core_web_sm
INFO:hello_agents.memory.types.semantic:📚 Available language models: Chinese, English
INFO:hello_agents.memory.types.semantic:Enhanced semantic memory initialization complete (using Qdrant+Neo4j professional databases)
INFO:hello_agents.memory.manager:MemoryManager initialization complete, enabled memory types: ['working', 'episodic', 'semantic']
✅ Tool 'memory' registered.
INFO:hello_agents.memory.storage.qdrant_store:✅ Successfully connected to Qdrant cloud service: https://0c517275-2ad0-4442-8309-11c36dc7e811.us-east-1-1.aws.cloud.qdrant.io:6333
INFO:hello_agents.memory.storage.qdrant_store:✅ Using existing Qdrant collection: rag_knowledge_base
✅ RAG tool initialization successful: namespace=default, collection=rag_knowledge_base
✅ Tool 'rag' registered.
Hello, Zhang San! Nice to meet you. As a Python developer, you must be passionate about programming. If you have any technical questions or need to discuss Python-related topics, feel free to reach out to me anytime. I'll do my best to help you. Is there anything I can help you with right now?
```

## 8.2 Hệ thống Bộ nhớ: Trao cho Agent khả năng Ghi nhớ

### 8.2.1 Quy trình Làm việc của Hệ thống Bộ nhớ

Trước khi bước vào giai đoạn triển khai mã, chúng ta cần định nghĩa trước quy trình làm việc (workflow) của hệ thống bộ nhớ. Quy trình này tham khảo mô hình bộ nhớ trong khoa học nhận thức và ánh xạ mỗi giai đoạn nhận thức tới các thành phần và thao tác kỹ thuật cụ thể. Hiểu được mối quan hệ ánh xạ này sẽ giúp ích cho việc triển khai mã về sau.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-3.png" alt="Memory Formation Process" width="90%"/>
  <p>Hình 8.3 Tiến trình Nhận thức của việc Hình thành Bộ nhớ</p>
</div>

Như minh họa trong Hình 8.3, theo nghiên cứu khoa học nhận thức, sự hình thành bộ nhớ của con người trải qua các giai đoạn sau:

1. **Mã hóa (Encoding)**: Chuyển đổi thông tin được cảm nhận thành dạng có thể lưu trữ
2. **Lưu trữ (Storage)**: Lưu thông tin đã mã hóa vào hệ thống bộ nhớ
3. **Truy xuất (Retrieval)**: Trích xuất thông tin liên quan từ bộ nhớ khi cần
4. **Củng cố (Consolidation)**: Chuyển bộ nhớ ngắn hạn thành bộ nhớ dài hạn
5. **Quên (Forgetting)**: Xóa thông tin không quan trọng hoặc lỗi thời

Dựa trên cảm hứng này, chúng ta đã thiết kế một hệ thống bộ nhớ hoàn chỉnh cho HelloAgents. Ý tưởng cốt lõi của nó là mô phỏng cách não người xử lý các loại thông tin khác nhau, chia bộ nhớ thành nhiều mô-đun chuyên biệt và thiết lập một cơ chế quản lý thông minh. Hình 8.4 trình bày chi tiết quy trình làm việc của hệ thống này, bao gồm các khâu then chốt như thêm bộ nhớ, truy xuất, củng cố và quên.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-4.png" alt="Memory System Workflow" width="95%"/>
  <p>Hình 8.4 Quy trình Làm việc Hoàn chỉnh của Hệ thống Bộ nhớ HelloAgents</p>
</div>

Hệ thống bộ nhớ của chúng ta bao gồm bốn loại mô-đun bộ nhớ khác nhau, mỗi loại được tối ưu cho các kịch bản ứng dụng và vòng đời cụ thể:

Đầu tiên là **Bộ nhớ làm việc (Working Memory)**, đóng vai trò "bộ nhớ ngắn hạn" của agent, chủ yếu dùng để lưu trữ thông tin ngữ cảnh của cuộc hội thoại hiện tại. Để đảm bảo truy cập và phản hồi tốc độ cao, dung lượng của nó được cố ý giới hạn (ví dụ mặc định 50 mục), và vòng đời của nó gắn với một phiên duy nhất, tự động xóa sau khi phiên kết thúc.

Thứ hai là **Bộ nhớ tình tiết (Episodic Memory)**, chịu trách nhiệm lưu trữ dài hạn các sự kiện tương tác cụ thể và các trải nghiệm học hỏi của agent. Khác với bộ nhớ làm việc, bộ nhớ tình tiết chứa thông tin ngữ cảnh phong phú và hỗ trợ truy xuất hồi tưởng theo chuỗi thời gian hoặc theo chủ đề, đóng vai trò nền tảng để agent "xem lại" và học hỏi từ những trải nghiệm trong quá khứ.

Tương ứng với các sự kiện cụ thể là **Bộ nhớ ngữ nghĩa (Semantic Memory)**, lưu trữ tri thức, khái niệm và quy tắc trừu tượng hơn. Ví dụ, sở thích của người dùng học được qua hội thoại, các chỉ dẫn cần tuân theo lâu dài, hoặc các điểm tri thức chuyên ngành đều phù hợp để lưu trữ ở đây. Phần bộ nhớ này có tính bền vững và tầm quan trọng cao, là cốt lõi để agent hình thành "hệ thống tri thức" và thực hiện suy luận liên tưởng.

Cuối cùng, để tương tác với các phương tiện đa phương tiện (multimedia) ngày càng phong phú, chúng ta đưa vào **Bộ nhớ tri giác (Perceptual Memory)**. Mô-đun này xử lý riêng thông tin đa phương thức (multimodal) như hình ảnh và âm thanh, và hỗ trợ truy xuất xuyên phương thức (cross-modal). Vòng đời của nó được quản lý động dựa trên tầm quan trọng của thông tin và không gian lưu trữ khả dụng.

### 8.2.2 Trải nghiệm Nhanh: Bắt đầu với Tính năng Bộ nhớ trong 30 giây

Trước khi đi sâu vào chi tiết triển khai, hãy nhanh chóng trải nghiệm các chức năng cơ bản của hệ thống bộ nhớ:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.tools import MemoryTool

# Create Agent with memory capability
llm = HelloAgentsLLM()
agent = SimpleAgent(name="Memory Assistant", llm=llm)

# Create memory tool
memory_tool = MemoryTool(user_id="user123")
tool_registry = ToolRegistry()
tool_registry.register_tool(memory_tool)
agent.tool_registry = tool_registry

# Experience memory features
print("=== Adding Multiple Memories ===")

# Add first memory
result1 = memory_tool.execute("add", content="User Zhang San is a Python developer focusing on machine learning and data analysis", memory_type="semantic", importance=0.8)
print(f"Memory 1: {result1}")

# Add second memory
result2 = memory_tool.execute("add", content="Li Si is a frontend engineer skilled in React and Vue.js development", memory_type="semantic", importance=0.7)
print(f"Memory 2: {result2}")

# Add third memory
result3 = memory_tool.execute("add", content="Wang Wu is a product manager responsible for user experience design and requirements analysis", memory_type="semantic", importance=0.6)
print(f"Memory 3: {result3}")

print("\n=== Searching Specific Memories ===")
# Search for frontend-related memories
print("🔍 Searching 'frontend engineer':")
result = memory_tool.execute("search", query="frontend engineer", limit=3)
print(result)

print("\n=== Memory Summary ===")
result = memory_tool.execute("summary")
print(result)
```

### 8.2.3 Giải thích Chi tiết về MemoryTool

Bây giờ hãy áp dụng cách tiếp cận từ trên xuống (top-down), bắt đầu từ các thao tác cụ thể mà MemoryTool hỗ trợ và dần dần đi sâu vào phần triển khai bên dưới. MemoryTool, với vai trò là giao diện thống nhất của hệ thống bộ nhớ, tuân theo mô hình kiến trúc "một điểm vào thống nhất, xử lý phân tán":

````python
def execute(self, action: str, **kwargs) -> str:
    """Execute memory operation

    Supported operations:
    - add: Add memory (supports 4 types: working/episodic/semantic/perceptual)
    - search: Search memory
    - summary: Get memory summary
    - stats: Get statistics
    - update: Update memory
    - remove: Delete memory
    - forget: Forget memory (multiple strategies)
    - consolidate: Consolidate memory (short-term → long-term)
    - clear_all: Clear all memories
    """

    if action == "add":
        return self._add_memory(**kwargs)
    elif action == "search":
        return self._search_memory(**kwargs)
    elif action == "summary":
        return self._get_summary(**kwargs)
    # ... other operations
````

Thiết kế giao diện `execute` thống nhất này đơn giản hóa cách gọi của Agent. Thao tác cụ thể được chỉ định thông qua tham số `action`, và `**kwargs` cho phép mỗi thao tác có các yêu cầu tham số khác nhau. Ở đây chúng ta sẽ liệt kê một vài thao tác quan trọng:

(1) Thao tác 1: add

Thao tác `add` là nền tảng của hệ thống bộ nhớ. Nó mô phỏng quá trình não người mã hóa thông tin được cảm nhận thành bộ nhớ. Trong triển khai, chúng ta không chỉ cần lưu trữ nội dung bộ nhớ mà còn thêm thông tin ngữ cảnh phong phú cho mỗi bộ nhớ. Thông tin này sẽ đóng vai trò quan trọng trong việc truy xuất và quản lý về sau.

````python
def _add_memory(
    self,
    content: str = "",
    memory_type: str = "working",
    importance: float = 0.5,
    file_path: str = None,
    modality: str = None,
    **metadata
) -> str:
    """Add memory"""
    try:
        # Ensure session ID exists
        if self.current_session_id is None:
            self.current_session_id = f"session_{datetime.now().strftime('%Y%m%d_%H%M%S')}"

        # Perceptual memory file support
        if memory_type == "perceptual" and file_path:
            inferred = modality or self._infer_modality(file_path)
            metadata.setdefault("modality", inferred)
            metadata.setdefault("raw_data", file_path)

        # Add session information to metadata
        metadata.update({
            "session_id": self.current_session_id,
            "timestamp": datetime.now().isoformat()
        })

        memory_id = self.memory_manager.add_memory(
            content=content,
            memory_type=memory_type,
            importance=importance,
            metadata=metadata,
            auto_classify=False
        )

        return f"✅ Memory added (ID: {memory_id[:8]}...)"

    except Exception as e:
        return f"❌ Failed to add memory: {str(e)}"
````

Đoạn này chủ yếu hiện thực hóa ba nhiệm vụ then chốt: quản lý tự động ID phiên (đảm bảo mỗi bộ nhớ đều có quy thuộc phiên rõ ràng), xử lý thông minh dữ liệu đa phương thức (tự động suy ra loại file và lưu metadata liên quan), và bổ sung tự động thông tin ngữ cảnh (thêm dấu thời gian và thông tin phiên cho mỗi bộ nhớ). Trong đó, tham số `importance` (mặc định 0.5) dùng để đánh dấu mức độ quan trọng của bộ nhớ, với khoảng giá trị 0.0-1.0. Cơ chế này mô phỏng việc não người đánh giá tầm quan trọng của các loại thông tin khác nhau. Thiết kế này cho phép Agent tự động phân biệt các cuộc hội thoại ở các khoảng thời gian khác nhau và cung cấp thông tin ngữ cảnh phong phú cho việc truy xuất và quản lý về sau.

Với mỗi loại bộ nhớ, chúng ta cung cấp các ví dụ sử dụng khác nhau:

```python
# 1. Working Memory - Temporary information, limited capacity
memory_tool.execute("add",
    content="User just asked a question about Python functions",
    memory_type="working",
    importance=0.6
)

# 2. Episodic Memory - Specific events and experiences
memory_tool.execute("add",
    content="On March 15, 2024, user Zhang San completed their first Python project",
    memory_type="episodic",
    importance=0.8,
    event_type="milestone",
    location="Online learning platform"
)

# 3. Semantic Memory - Abstract knowledge and concepts
memory_tool.execute("add",
    content="Python is an interpreted, object-oriented programming language",
    memory_type="semantic",
    importance=0.9,
    knowledge_type="factual"
)

# 4. Perceptual Memory - Multimodal information
memory_tool.execute("add",
    content="User uploaded a Python code screenshot containing function definitions",
    memory_type="perceptual",
    importance=0.7,
    modality="image",
    file_path="./uploads/code_screenshot.png"
)
```

(2) Thao tác 2: search

Thao tác `search` là chức năng cốt lõi của hệ thống bộ nhớ. Nó cần nhanh chóng tìm ra nội dung liên quan nhất với truy vấn trong một lượng lớn bộ nhớ. Nó bao gồm nhiều bước như hiểu ngữ nghĩa, tính toán độ liên quan, và sắp xếp kết quả.

````python
def _search_memory(
    self,
    query: str,
    limit: int = 5,
    memory_types: List[str] = None,
    memory_type: str = None,
    min_importance: float = 0.1
) -> str:
    """Search memory"""
    try:
        # Parameter standardization
        if memory_type and not memory_types:
            memory_types = [memory_type]

        results = self.memory_manager.retrieve_memories(
            query=query,
            limit=limit,
            memory_types=memory_types,
            min_importance=min_importance
        )

        if not results:
            return f"🔍 No memories found related to '{query}'"

        # Format results
        formatted_results = []
        formatted_results.append(f"🔍 Found {len(results)} related memories:")

        for i, memory in enumerate(results, 1):
            memory_type_label = {
                "working": "Working Memory",
                "episodic": "Episodic Memory",
                "semantic": "Semantic Memory",
                "perceptual": "Perceptual Memory"
            }.get(memory.memory_type, memory.memory_type)

            content_preview = memory.content[:80] + "..." if len(memory.content) > 80 else memory.content
            formatted_results.append(
                f"{i}. [{memory_type_label}] {content_preview} (Importance: {memory.importance:.2f})"
            )

        return "\n".join(formatted_results)

    except Exception as e:
        return f"❌ Failed to search memory: {str(e)}"
````

Thao tác search được thiết kế để hỗ trợ cả dạng tham số số ít lẫn số nhiều (`memory_type` và `memory_types`), cho phép người dùng diễn đạt nhu cầu theo cách tự nhiên nhất. Trong đó, tham số `min_importance` (mặc định 0.1) dùng để lọc bỏ các bộ nhớ chất lượng thấp. Về cách dùng chức năng search, bạn có thể tham khảo ví dụ này:

```python
# Basic search
result = memory_tool.execute("search", query="Python programming", limit=5)

# Search by specifying memory type
result = memory_tool.execute("search",
    query="learning progress",
    memory_type="episodic",
    limit=3
)

# Multi-type search
result = memory_tool.execute("search",
    query="function definition",
    memory_types=["semantic", "episodic"],
    min_importance=0.5
)
```

(3) Thao tác 3: forget

Cơ chế quên (forgetting) là tính năng mang tính khoa học nhận thức nhất. Nó mô phỏng quá trình quên có chọn lọc của não người và hỗ trợ ba chiến lược: dựa trên tầm quan trọng (xóa các bộ nhớ không quan trọng), dựa trên thời gian (xóa các bộ nhớ lỗi thời), và dựa trên dung lượng (xóa các bộ nhớ ít quan trọng nhất khi lưu trữ gần đạt giới hạn).

````python
def _forget(self, strategy: str = "importance_based", threshold: float = 0.1, max_age_days: int = 30) -> str:
    """Forget memories (supports multiple strategies)"""
    try:
        count = self.memory_manager.forget_memories(
            strategy=strategy,
            threshold=threshold,
            max_age_days=max_age_days
        )
        return f"🧹 Forgot {count} memories (strategy: {strategy})"
    except Exception as e:
        return f"❌ Failed to forget memories: {str(e)}"
````

**Cách sử dụng ba chiến lược quên:**

```python
# 1. Importance-based forgetting - Delete memories below importance threshold
memory_tool.execute("forget",
    strategy="importance_based",
    threshold=0.2
)

# 2. Time-based forgetting - Delete memories older than specified days
memory_tool.execute("forget",
    strategy="time_based",
    max_age_days=30
)

# 3. Capacity-based forgetting - Delete least important when memory count exceeds limit
memory_tool.execute("forget",
    strategy="capacity_based",
    threshold=0.3
)
```

(4) Thao tác 4: consolidate

````python
def _consolidate(self, from_type: str = "working", to_type: str = "episodic", importance_threshold: float = 0.7) -> str:
    """Consolidate memories (promote important short-term memories to long-term memories)"""
    try:
        count = self.memory_manager.consolidate_memories(
            from_type=from_type,
            to_type=to_type,
            importance_threshold=importance_threshold,
        )
        return f"🔄 Consolidated {count} memories to long-term memory ({from_type} → {to_type}, threshold={importance_threshold})"
    except Exception as e:
        return f"❌ Failed to consolidate memories: {str(e)}"
````

Thao tác consolidate mượn ý tưởng củng cố bộ nhớ (memory consolidation) trong khoa học thần kinh, mô phỏng quá trình não người chuyển bộ nhớ ngắn hạn thành bộ nhớ dài hạn. Thiết lập mặc định là chuyển các bộ nhớ làm việc có tầm quan trọng vượt quá 0.7 thành bộ nhớ tình tiết. Ngưỡng này đảm bảo chỉ những thông tin thực sự quan trọng mới được lưu giữ lâu dài. Toàn bộ quá trình được tự động hóa; người dùng không cần tự chọn các bộ nhớ cụ thể. Hệ thống tự động nhận diện những bộ nhớ đáp ứng tiêu chí và thực hiện chuyển đổi loại.

**Ví dụ sử dụng việc củng cố bộ nhớ:**

```python
# Convert important working memories to episodic memories
memory_tool.execute("consolidate",
    from_type="working",
    to_type="episodic",
    importance_threshold=0.7
)

# Convert important episodic memories to semantic memories
memory_tool.execute("consolidate",
    from_type="episodic",
    to_type="semantic",
    importance_threshold=0.8
)
```

Thông qua sự phối hợp của các thao tác cốt lõi này, MemoryTool xây dựng một hệ thống quản lý vòng đời bộ nhớ hoàn chỉnh. Từ tạo lập, truy xuất, tóm tắt cho đến quên, củng cố và quản lý bộ nhớ, nó tạo thành một hệ thống quản lý bộ nhớ thông minh khép kín, trao cho Agent năng lực ghi nhớ thực sự giống con người.

### 8.2.4 Giải thích Chi tiết về MemoryManager

Sau khi hiểu thiết kế giao diện của MemoryTool, hãy đi sâu vào phần triển khai bên dưới để xem MemoryTool phối hợp với MemoryManager như thế nào. Thiết kế phân lớp này thể hiện nguyên tắc tách biệt mối quan tâm (separation of concerns) trong kỹ thuật phần mềm. MemoryTool tập trung vào giao diện người dùng và xử lý tham số, còn MemoryManager chịu trách nhiệm về logic quản lý bộ nhớ cốt lõi.

MemoryTool tạo một thực thể MemoryManager trong quá trình khởi tạo và bật các mô-đun bộ nhớ thuộc các loại khác nhau dựa trên cấu hình. Thiết kế này cho phép người dùng lựa chọn kích hoạt loại bộ nhớ nào dựa trên nhu cầu cụ thể, đảm bảo tính đầy đủ về chức năng đồng thời tránh tiêu tốn tài nguyên không cần thiết.

````python
class MemoryTool(Tool):
    """Memory tool - Provides memory functionality for Agent"""

    def __init__(
        self,
        user_id: str = "default_user",
        memory_config: MemoryConfig = None,
        memory_types: List[str] = None
    ):
        super().__init__(
            name="memory",
            description="Memory tool - Can store and retrieve conversation history, knowledge, and experience"
        )

        # Initialize memory manager
        self.memory_config = memory_config or MemoryConfig()
        self.memory_types = memory_types or ["working", "episodic", "semantic"]

        self.memory_manager = MemoryManager(
            config=self.memory_config,
            user_id=user_id,
            enable_working="working" in self.memory_types,
            enable_episodic="episodic" in self.memory_types,
            enable_semantic="semantic" in self.memory_types,
            enable_perceptual="perceptual" in self.memory_types
        )
````

MemoryManager, với vai trò là bộ điều phối cốt lõi của hệ thống bộ nhớ, chịu trách nhiệm quản lý các loại mô-đun bộ nhớ khác nhau và cung cấp một giao diện thao tác thống nhất.

````python
class MemoryManager:
    """Memory manager - Unified memory operation interface"""

    def __init__(
        self,
        config: Optional[MemoryConfig] = None,
        user_id: str = "default_user",
        enable_working: bool = True,
        enable_episodic: bool = True,
        enable_semantic: bool = True,
        enable_perceptual: bool = False
    ):
        self.config = config or MemoryConfig()
        self.user_id = user_id

        # Initialize storage and retrieval components
        self.store = MemoryStore(self.config)
        self.retriever = MemoryRetriever(self.store, self.config)

        # Initialize various types of memory
        self.memory_types = {}

        if enable_working:
            self.memory_types['working'] = WorkingMemory(self.config, self.store)

        if enable_episodic:
            self.memory_types['episodic'] = EpisodicMemory(self.config, self.store)

        if enable_semantic:
            self.memory_types['semantic'] = SemanticMemory(self.config, self.store)

        if enable_perceptual:
            self.memory_types['perceptual'] = PerceptualMemory(self.config, self.store)
````

### 8.2.5 Bốn Loại Bộ nhớ

Bây giờ hãy đi sâu vào phần triển khai cụ thể của bốn loại bộ nhớ. Mỗi loại bộ nhớ có đặc trưng và kịch bản ứng dụng riêng:

(1) Bộ nhớ làm việc (Working Memory)

Bộ nhớ làm việc là phần năng động nhất của hệ thống bộ nhớ. Nó chịu trách nhiệm lưu trữ thông tin tạm thời trong phiên hội thoại hiện tại. Trọng tâm thiết kế của bộ nhớ làm việc là truy cập nhanh và tự động dọn dẹp, điều này đảm bảo tốc độ phản hồi và hiệu quả sử dụng tài nguyên của hệ thống.

Bộ nhớ làm việc áp dụng giải pháp lưu trữ hoàn toàn trong bộ nhớ (in-memory), kết hợp với cơ chế TTL (Time To Live) để tự động dọn dẹp. Ưu điểm của thiết kế này là tốc độ truy cập cực nhanh, nhưng cũng có nghĩa là nội dung của bộ nhớ làm việc sẽ mất sau khi hệ thống khởi động lại. Đặc trưng này hoàn toàn phù hợp với định vị của bộ nhớ làm việc: lưu trữ thông tin tạm thời và dễ biến mất (volatile).

````python
class WorkingMemory:
    """Working memory implementation
    Features:
    - Limited capacity (default 50 items) + TTL automatic cleanup
    - Pure in-memory storage, extremely fast access
    - Hybrid retrieval: TF-IDF vectorization + keyword matching
    """

    def __init__(self, config: MemoryConfig):
        self.max_capacity = config.working_memory_capacity or 50
        self.max_age_minutes = config.working_memory_ttl or 60
        self.memories = []

    def add(self, memory_item: MemoryItem) -> str:
        """Add working memory"""
        self._expire_old_memories()  # Expiration cleanup

        if len(self.memories) >= self.max_capacity:
            self._remove_lowest_priority_memory()  # Capacity management

        self.memories.append(memory_item)
        return memory_item.id

    def retrieve(self, query: str, limit: int = 5, **kwargs) -> List[MemoryItem]:
        """Hybrid retrieval: TF-IDF vectorization + keyword matching"""
        self._expire_old_memories()

        # Try TF-IDF vector retrieval
        vector_scores = self._try_tfidf_search(query)

        # Calculate comprehensive score
        scored_memories = []
        for memory in self.memories:
            vector_score = vector_scores.get(memory.id, 0.0)
            keyword_score = self._calculate_keyword_score(query, memory.content)

            # Hybrid scoring
            base_relevance = vector_score * 0.7 + keyword_score * 0.3 if vector_score > 0 else keyword_score
            time_decay = self._calculate_time_decay(memory.timestamp)
            importance_weight = 0.8 + (memory.importance * 0.4)

            final_score = base_relevance * time_decay * importance_weight
            if final_score > 0:
                scored_memories.append((final_score, memory))

        scored_memories.sort(key=lambda x: x[0], reverse=True)
        return [memory for _, memory in scored_memories[:limit]]
````

Việc truy xuất bộ nhớ làm việc áp dụng chiến lược truy xuất lai (hybrid retrieval). Trước tiên nó cố gắng dùng vector hóa TF-IDF để truy xuất ngữ nghĩa, và nếu thất bại, nó quay lại đối sánh từ khóa (keyword matching). Thiết kế này đảm bảo dịch vụ truy xuất đáng tin cậy trong nhiều môi trường khác nhau. Thuật toán tính điểm kết hợp độ tương đồng ngữ nghĩa, suy giảm theo thời gian (time decay) và trọng số tầm quan trọng. Công thức điểm cuối cùng là: `(độ tương đồng × suy giảm thời gian) × (0.8 + tầm quan trọng × 0.4)`.

(2) Bộ nhớ tình tiết (Episodic Memory)

Bộ nhớ tình tiết chịu trách nhiệm lưu trữ các sự kiện và trải nghiệm cụ thể. Trọng tâm thiết kế của nó là duy trì tính toàn vẹn của sự kiện và các mối quan hệ theo trình tự thời gian. Bộ nhớ tình tiết áp dụng giải pháp lưu trữ lai SQLite + Qdrant. SQLite chịu trách nhiệm lưu trữ dữ liệu có cấu trúc và các truy vấn phức tạp, còn Qdrant chịu trách nhiệm truy xuất vector hiệu quả.

````python
class EpisodicMemory:
    """Episodic memory implementation
    Features:
    - SQLite+Qdrant hybrid storage architecture
    - Supports time series and session-level retrieval
    - Structured filtering + semantic vector retrieval
    """

    def __init__(self, config: MemoryConfig):
        self.doc_store = SQLiteDocumentStore(config.database_path)
        self.vector_store = QdrantVectorStore(config.qdrant_url, config.qdrant_api_key)
        self.embedder = create_embedding_model_with_fallback()
        self.sessions = {}  # Session index

    def add(self, memory_item: MemoryItem) -> str:
        """Add episodic memory"""
        # Create episode object
        episode = Episode(
            episode_id=memory_item.id,
            session_id=memory_item.metadata.get("session_id", "default"),
            timestamp=memory_item.timestamp,
            content=memory_item.content,
            context=memory_item.metadata
        )

        # Update session index
        session_id = episode.session_id
        if session_id not in self.sessions:
            self.sessions[session_id] = []
        self.sessions[session_id].append(episode.episode_id)

        # Persistent storage (SQLite + Qdrant)
        self._persist_episode(episode)
        return memory_item.id

    def retrieve(self, query: str, limit: int = 5, **kwargs) -> List[MemoryItem]:
        """Hybrid retrieval: structured filtering + semantic vector retrieval"""
        # 1. Structured pre-filtering (time range, importance, etc.)
        candidate_ids = self._structured_filter(**kwargs)

        # 2. Vector semantic retrieval
        hits = self._vector_search(query, limit * 5, kwargs.get("user_id"))

        # 3. Comprehensive scoring and sorting
        results = []
        for hit in hits:
            if self._should_include(hit, candidate_ids, kwargs):
                score = self._calculate_episode_score(hit)
                memory_item = self._create_memory_item(hit)
                results.append((score, memory_item))

        results.sort(key=lambda x: x[0], reverse=True)
        return [item for _, item in results[:limit]]

    def _calculate_episode_score(self, hit) -> float:
        """Episodic memory scoring algorithm"""
        vec_score = float(hit.get("score", 0.0))
        recency_score = self._calculate_recency(hit["metadata"]["timestamp"])
        importance = hit["metadata"].get("importance", 0.5)

        # Scoring formula: (vector similarity × 0.8 + temporal recency × 0.2) × importance weight
        base_relevance = vec_score * 0.8 + recency_score * 0.2
        importance_weight = 0.8 + (importance * 0.4)

        return base_relevance * importance_weight
````

Phần triển khai truy xuất của bộ nhớ tình tiết thể hiện một cơ chế tính điểm đa yếu tố phức tạp. Nó không chỉ xét đến độ tương đồng ngữ nghĩa mà còn kết hợp cân nhắc tính gần đây về mặt thời gian (temporal recency), và cuối cùng được điều chỉnh bằng trọng số tầm quan trọng. Công thức tính điểm là: `(độ tương đồng vector × 0.8 + tính gần đây thời gian × 0.2) × (0.8 + tầm quan trọng × 0.4)`, đảm bảo kết quả truy xuất vừa liên quan về mặt ngữ nghĩa vừa liên quan về mặt thời gian.

(3) Bộ nhớ ngữ nghĩa (Semantic Memory)

Bộ nhớ ngữ nghĩa là phần phức tạp nhất của hệ thống bộ nhớ. Nó chịu trách nhiệm lưu trữ các khái niệm, quy tắc và tri thức trừu tượng. Trọng tâm thiết kế của bộ nhớ ngữ nghĩa là biểu diễn có cấu trúc của tri thức và năng lực suy luận thông minh. Bộ nhớ ngữ nghĩa áp dụng kiến trúc lai giữa cơ sở dữ liệu đồ thị Neo4j và cơ sở dữ liệu vector Qdrant. Thiết kế này cho phép hệ thống vừa thực hiện truy xuất ngữ nghĩa nhanh vừa thực hiện suy luận quan hệ phức tạp bằng đồ thị tri thức (knowledge graph).

````python
class SemanticMemory(BaseMemory):
    """Semantic memory implementation

    Features:
    - Uses HuggingFace Chinese pre-trained models for text embedding
    - Vector retrieval for fast similarity matching
    - Knowledge graph storage for entities and relationships
    - Hybrid retrieval strategy: vector + graph + semantic reasoning
    """

    def __init__(self, config: MemoryConfig, storage_backend=None):
        super().__init__(config, storage_backend)

        # Embedding model (unified provision)
        self.embedding_model = get_text_embedder()

        # Professional database storage
        self.vector_store = QdrantConnectionManager.get_instance(**qdrant_config)
        self.graph_store = Neo4jGraphStore(**neo4j_config)

        # Entity and relation cache
        self.entities: Dict[str, Entity] = {}
        self.relations: List[Relation] = []

        # NLP processor (supports Chinese and English)
        self.nlp = self._init_nlp()
````

Quá trình thêm bộ nhớ ngữ nghĩa thể hiện quy trình làm việc hoàn chỉnh của việc xây dựng đồ thị tri thức. Hệ thống không chỉ lưu trữ nội dung bộ nhớ mà còn tự động trích xuất các thực thể (entity) và mối quan hệ (relationship) để xây dựng các biểu diễn tri thức có cấu trúc:

```python
def add(self, memory_item: MemoryItem) -> str:
    """Add semantic memory"""
    # 1. Generate text embedding
    embedding = self.embedding_model.encode(memory_item.content)

    # 2. Extract entities and relations
    entities = self._extract_entities(memory_item.content)
    relations = self._extract_relations(memory_item.content, entities)

    # 3. Store to Neo4j graph database
    for entity in entities:
        self._add_entity_to_graph(entity, memory_item)

    for relation in relations:
        self._add_relation_to_graph(relation, memory_item)

    # 4. Store to Qdrant vector database
    metadata = {
        "memory_id": memory_item.id,
        "entities": [e.entity_id for e in entities],
        "entity_count": len(entities),
        "relation_count": len(relations)
    }

    self.vector_store.add_vectors(
        vectors=[embedding.tolist()],
        metadata=[metadata],
        ids=[memory_item.id]
    )
```

Việc truy xuất bộ nhớ ngữ nghĩa hiện thực hóa một chiến lược tìm kiếm lai, kết hợp năng lực hiểu ngữ nghĩa của truy xuất vector và năng lực suy luận quan hệ của truy xuất đồ thị:

```python
def retrieve(self, query: str, limit: int = 5, **kwargs) -> List[MemoryItem]:
    """Retrieve semantic memory"""
    # 1. Vector retrieval
    vector_results = self._vector_search(query, limit * 2, user_id)

    # 2. Graph retrieval
    graph_results = self._graph_search(query, limit * 2, user_id)

    # 3. Hybrid ranking
    combined_results = self._combine_and_rank_results(
        vector_results, graph_results, query, limit
    )

    return combined_results[:limit]
```

Thuật toán xếp hạng lai áp dụng một cơ chế tính điểm đa yếu tố:

```python
def _combine_and_rank_results(self, vector_results, graph_results, query, limit):
    """Hybrid ranking of results"""
    combined = {}

    # Merge vector and graph retrieval results
    for result in vector_results:
        combined[result["memory_id"]] = {
            **result,
            "vector_score": result.get("score", 0.0),
            "graph_score": 0.0
        }

    for result in graph_results:
        memory_id = result["memory_id"]
        if memory_id in combined:
            combined[memory_id]["graph_score"] = result.get("similarity", 0.0)
        else:
            combined[memory_id] = {
                **result,
                "vector_score": 0.0,
                "graph_score": result.get("similarity", 0.0)
            }

    # Calculate hybrid score
    for memory_id, result in combined.items():
        vector_score = result["vector_score"]
        graph_score = result["graph_score"]
        importance = result.get("importance", 0.5)

        # Base relevance score
        base_relevance = vector_score * 0.7 + graph_score * 0.3

        # Importance weight [0.8, 1.2]
        importance_weight = 0.8 + (importance * 0.4)

        # Final score: similarity * importance weight
        combined_score = base_relevance * importance_weight
        result["combined_score"] = combined_score

    # Sort and return
    sorted_results = sorted(
        combined.values(),
        key=lambda x: x["combined_score"],
        reverse=True
    )

    return sorted_results[:limit]
```

Công thức tính điểm cho bộ nhớ ngữ nghĩa là: `(độ tương đồng vector × 0.7 + độ tương đồng đồ thị × 0.3) × (0.8 + tầm quan trọng × 0.4)`. Ý tưởng cốt lõi của thiết kế này là:

- **Trọng số truy xuất vector (0.7)**: Độ tương đồng ngữ nghĩa là yếu tố chính, đảm bảo kết quả truy xuất liên quan về mặt ngữ nghĩa với truy vấn
- **Trọng số truy xuất đồ thị (0.3)**: Suy luận quan hệ đóng vai trò bổ trợ, phát hiện các liên hệ ẩn giữa các khái niệm
- **Khoảng trọng số tầm quan trọng [0.8, 1.2]**: Tránh để tầm quan trọng ảnh hưởng quá mức đến thứ hạng theo độ tương đồng, giữ được độ chính xác của truy xuất

(4) Bộ nhớ tri giác (Perceptual Memory)

Bộ nhớ tri giác hỗ trợ lưu trữ và truy xuất dữ liệu ở nhiều phương thức (modality) như văn bản, hình ảnh và âm thanh. Nó áp dụng chiến lược lưu trữ tách biệt theo phương thức, tạo các bộ sưu tập (collection) vector độc lập cho dữ liệu của các phương thức khác nhau. Thiết kế này tránh được vấn đề không khớp chiều (dimension mismatch) đồng thời đảm bảo độ chính xác của truy xuất:

````python
class PerceptualMemory(BaseMemory):
    """Perceptual memory implementation

    Features:
    - Supports multimodal data (text, images, audio, etc.)
    - Cross-modal similarity search
    - Semantic understanding of perceptual data
    - Supports content generation and retrieval
    """

    def __init__(self, config: MemoryConfig, storage_backend=None):
        super().__init__(config, storage_backend)

        # Multimodal encoders
        self.text_embedder = get_text_embedder()
        self._clip_model = self._init_clip_model()  # Image encoding
        self._clap_model = self._init_clap_model()  # Audio encoding

        # Modality-separated vector storage
        self.vector_stores = {
            "text": QdrantConnectionManager.get_instance(
                collection_name="perceptual_text",
                vector_size=self.vector_dim
            ),
            "image": QdrantConnectionManager.get_instance(
                collection_name="perceptual_image",
                vector_size=self._image_dim
            ),
            "audio": QdrantConnectionManager.get_instance(
                collection_name="perceptual_audio",
                vector_size=self._audio_dim
            )
        }
````

Việc truy xuất bộ nhớ tri giác hỗ trợ cả chế độ cùng phương thức (same-modality) lẫn xuyên phương thức (cross-modality). Truy xuất cùng phương thức sử dụng các bộ mã hóa (encoder) chuyên biệt để đối sánh chính xác, còn truy xuất xuyên phương thức đòi hỏi cơ chế căn chỉnh ngữ nghĩa (semantic alignment) phức tạp hơn:

```python
def retrieve(self, query: str, limit: int = 5, **kwargs) -> List[MemoryItem]:
    """Retrieve perceptual memory (can filter modality; same-modality vector retrieval + time/importance fusion)"""
    user_id = kwargs.get("user_id")
    target_modality = kwargs.get("target_modality")
    query_modality = kwargs.get("query_modality", target_modality or "text")

    # Same-modality vector retrieval
    try:
        query_vector = self._encode_data(query, query_modality)
        store = self._get_vector_store_for_modality(target_modality or query_modality)

        where = {"memory_type": "perceptual"}
        if user_id:
            where["user_id"] = user_id
        if target_modality:
            where["modality"] = target_modality

        hits = store.search_similar(
            query_vector=query_vector,
            limit=max(limit * 5, 20),
            where=where
        )
    except Exception:
        hits = []

    # Fusion ranking (vector similarity + temporal recency + importance weight)
    results = []
    for hit in hits:
        vector_score = float(hit.get("score", 0.0))
        recency_score = self._calculate_recency_score(hit["metadata"]["timestamp"])
        importance = hit["metadata"].get("importance", 0.5)

        # Scoring algorithm
        base_relevance = vector_score * 0.8 + recency_score * 0.2
        importance_weight = 0.8 + (importance * 0.4)
        combined_score = base_relevance * importance_weight

        results.append((combined_score, self._create_memory_item(hit)))

    results.sort(key=lambda x: x[0], reverse=True)
    return [item for _, item in results[:limit]]
```

Công thức tính điểm cho bộ nhớ tri giác là: `(độ tương đồng vector × 0.8 + tính gần đây thời gian × 0.2) × (0.8 + tầm quan trọng × 0.4)`. Cơ chế tính điểm của bộ nhớ tri giác cũng hỗ trợ truy xuất xuyên phương thức, đạt được sự căn chỉnh ngữ nghĩa của dữ liệu ở các phương thức khác nhau như văn bản, hình ảnh và âm thanh thông qua một không gian vector thống nhất. Khi thực hiện truy xuất xuyên phương thức, hệ thống tự động điều chỉnh trọng số tính điểm để đảm bảo tính đa dạng và độ chính xác của kết quả truy xuất. Ngoài ra, việc tính tính gần đây thời gian trong bộ nhớ tri giác áp dụng mô hình suy giảm theo hàm mũ (exponential decay):

```python
def _calculate_recency_score(self, timestamp: str) -> float:
    """Calculate temporal recency score"""
    try:
        memory_time = datetime.fromisoformat(timestamp)
        current_time = datetime.now()
        age_hours = (current_time - memory_time).total_seconds() / 3600

        # Exponential decay: maintain high score within 24 hours, then gradually decay
        decay_factor = 0.1  # Decay coefficient
        recency_score = math.exp(-decay_factor * age_hours / 24)

        return max(0.1, recency_score)  # Maintain minimum base score of 0.1
    except Exception:
        return 0.5  # Default medium score
```

Mô hình suy giảm theo thời gian này mô phỏng đường cong quên (forgetting curve) trong bộ nhớ con người, đảm bảo hệ thống bộ nhớ tri giác có thể ưu tiên truy xuất những nội dung bộ nhớ liên quan hơn về mặt thời gian.

## 8.3 Hệ thống RAG: Tăng cường Truy xuất Tri thức

### 8.3.1 Kiến thức Nền tảng về RAG

Trước khi đi sâu vào phần triển khai hệ thống RAG của HelloAgents, hãy cùng tìm hiểu các khái niệm cơ bản, lịch sử phát triển và nguyên lý cốt lõi của công nghệ RAG. Vì cuốn sách này không được xây dựng dựa trên nền tảng RAG, ở đây chúng ta chỉ ôn lại nhanh các khái niệm liên quan để hiểu rõ hơn các lựa chọn kỹ thuật và những đổi mới trong thiết kế hệ thống.

(1) RAG là gì?

Sinh tăng cường bằng truy xuất (Retrieval-Augmented Generation - RAG) là một công nghệ kết hợp truy xuất thông tin (information retrieval) và sinh văn bản (text generation). Ý tưởng cốt lõi của nó là: trước khi sinh câu trả lời, trước tiên truy xuất thông tin liên quan từ một cơ sở tri thức bên ngoài, sau đó cung cấp thông tin đã truy xuất cho mô hình ngôn ngữ lớn dưới dạng ngữ cảnh, từ đó sinh ra câu trả lời chính xác và đáng tin cậy hơn.

Do đó, Retrieval-Augmented Generation có thể được phân tách thành ba từ. **Retrieval (Truy xuất)** đề cập đến việc truy vấn nội dung liên quan từ cơ sở tri thức; **Augmented (Tăng cường)** nghĩa là tích hợp kết quả truy xuất vào prompt để hỗ trợ mô hình sinh; **Generation (Sinh)** xuất ra câu trả lời vừa chính xác vừa minh bạch.

(2) Quy trình Làm việc Cơ bản

Một quy trình làm việc RAG hoàn chỉnh chủ yếu được chia thành hai giai đoạn cốt lõi. Trong **giai đoạn chuẩn bị dữ liệu (data preparation stage)**, hệ thống xây dựng tri thức bên ngoài thành một cơ sở dữ liệu có thể truy xuất thông qua **trích xuất dữ liệu (data extraction)**, **phân đoạn văn bản (text segmentation)** và **vector hóa (vectorization)**. Sau đó, trong **giai đoạn ứng dụng (application stage)**, hệ thống phản hồi **truy vấn (query)** của người dùng, **truy xuất (retrieve)** thông tin liên quan từ cơ sở dữ liệu, **đưa nó vào prompt (inject into the prompt)**, và cuối cùng thúc đẩy mô hình ngôn ngữ lớn **sinh câu trả lời (generate answers)**.

(3) Lịch sử Phát triển

Giai đoạn một: Naive RAG (2020-2021). Đây là giai đoạn phôi thai của công nghệ RAG, với quy trình trực tiếp và đơn giản, thường được gọi là chế độ "Truy xuất - Đọc" (Retrieve-Read). **Phương pháp truy xuất**: Chủ yếu dựa vào các thuật toán đối sánh từ khóa truyền thống như `TF-IDF` hoặc `BM25`. Các phương pháp này tính tần suất từ (term frequency) và tần suất tài liệu (document frequency) để đánh giá độ liên quan, có hiệu quả đối sánh theo mặt chữ tốt, nhưng khó hiểu được độ tương đồng ngữ nghĩa. **Chế độ sinh**: Ghép trực tiếp nội dung tài liệu đã truy xuất vào ngữ cảnh prompt mà không xử lý, rồi gửi cho mô hình sinh.

Giai đoạn hai: Advanced RAG (2022-2023). Với sự trưởng thành của cơ sở dữ liệu vector và công nghệ nhúng văn bản (text embedding), RAG bước vào giai đoạn phát triển nhanh. Các nhà nghiên cứu và nhà phát triển đã đưa vào một lượng lớn kỹ thuật tối ưu trong các giai đoạn "truy xuất" và "sinh". **Phương pháp truy xuất**: Chuyển sang truy xuất ngữ nghĩa dựa trên **nhúng dày đặc (dense embedding)**. Bằng cách chuyển văn bản thành các vector nhiều chiều, mô hình có thể hiểu và đối sánh độ tương đồng ngữ nghĩa, không chỉ là từ khóa. **Chế độ sinh**: Đưa vào nhiều kỹ thuật tối ưu, chẳng hạn như viết lại truy vấn (query rewriting), chia khối tài liệu (document chunking), xếp hạng lại (reranking), v.v.

Giai đoạn ba: Modular RAG (2023-đến nay). Dựa trên nền advanced RAG, các hệ thống RAG hiện đại tiếp tục phát triển theo hướng mô-đun hóa, tự động hóa và thông minh hóa. Các phần khác nhau của hệ thống được thiết kế thành các mô-đun độc lập có thể cắm-và-chạy (pluggable), có thể tổ hợp (composable) để thích ứng với các kịch bản ứng dụng đa dạng và phức tạp hơn. **Phương pháp truy xuất**: Chẳng hạn như truy xuất lai (hybrid retrieval), mở rộng đa truy vấn (multi-query expansion), nhúng tài liệu giả định (hypothetical document embedding), v.v. **Chế độ sinh**: Suy luận chuỗi tư duy (chain-of-thought reasoning), tự phản tỉnh và tự sửa (self-reflection and correction), v.v.

### 8.3.2 Nguyên lý Hoạt động của Hệ thống RAG

Trước khi đi sâu vào chi tiết triển khai, chúng ta có thể dùng một lưu đồ để phác thảo quy trình làm việc hoàn chỉnh của hệ thống RAG trong HelloAgents:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-5.png" alt="RAG System Core Principle" width="85%"/>
  <p>Hình 8.5 Nguyên lý Hoạt động Cốt lõi của Hệ thống RAG</p>
</div>

Như minh họa trong Hình 8.5, nó thể hiện hai chế độ làm việc chính của hệ thống RAG:
1. **Quy trình xử lý dữ liệu (Data Processing Workflow)**: Xử lý và lưu trữ các tài liệu tri thức. Ở đây chúng ta dùng công cụ `Markitdown`, với ý tưởng thiết kế là chuyển đổi thống nhất tất cả các nguồn tri thức bên ngoài đầu vào thành định dạng Markdown để xử lý.
2. **Quy trình truy vấn và sinh (Query and Generation Workflow)**: Truy xuất thông tin liên quan dựa trên truy vấn và sinh câu trả lời.

### 8.3.3 Trải nghiệm Nhanh: Bắt đầu với Tính năng RAG trong 30 giây

Hãy nhanh chóng trải nghiệm các chức năng cơ bản của hệ thống RAG:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.tools import RAGTool

# Create Agent with RAG capability
llm = HelloAgentsLLM()
agent = SimpleAgent(name="Knowledge Assistant", llm=llm)

# Create RAG tool
rag_tool = RAGTool(
    knowledge_base_path="./knowledge_base",
    collection_name="test_collection",
    rag_namespace="test"
)

tool_registry = ToolRegistry()
tool_registry.register_tool(rag_tool)
agent.tool_registry = tool_registry

# Experience RAG features
# Add first knowledge
result1 = rag_tool.execute("add_text",
    text="Python is a high-level programming language first released by Guido van Rossum in 1991. Python's design philosophy emphasizes code readability and concise syntax.",
    document_id="python_intro")
print(f"Knowledge 1: {result1}")

# Add second knowledge
result2 = rag_tool.execute("add_text",
    text="Machine learning is a branch of artificial intelligence that uses algorithms to enable computers to learn patterns from data. It mainly includes three types: supervised learning, unsupervised learning, and reinforcement learning.",
    document_id="ml_basics")
print(f"Knowledge 2: {result2}")

# Add third knowledge
result3 = rag_tool.execute("add_text",
    text="RAG (Retrieval-Augmented Generation) is an AI technology that combines information retrieval and text generation. It enhances the generation capability of large language models by retrieving relevant knowledge.",
    document_id="rag_concept")
print(f"Knowledge 3: {result3}")


print("\n=== Search Knowledge ===")
result = rag_tool.execute("search",
    query="History of Python programming language",
    limit=3,
    min_score=0.1
)
print(result)

print("\n=== Knowledge Base Statistics ===")
result = rag_tool.execute("stats")
print(result)
```

Tiếp theo, chúng ta sẽ đi sâu vào phần triển khai cụ thể của hệ thống RAG trong HelloAgents.

### 8.3.4 Thiết kế Kiến trúc Hệ thống RAG

Trong phần này, chúng ta áp dụng cách tiếp cận khác với phần giải thích hệ thống bộ nhớ. Bởi vì `Memory_tool` là một phần triển khai mang tính hệ thống, còn RAG trong thiết kế của chúng ta được định nghĩa là một công cụ có thể được tổ chức thành một pipeline. Kiến trúc cốt lõi của hệ thống RAG của chúng ta có thể được tóm tắt thành mô hình thiết kế "năm lớp bảy bước":

```
User Layer: RAGTool unified interface
  ↓
Application Layer: Intelligent Q&A, search, management
  ↓
Processing Layer: Document parsing, chunking, vectorization
  ↓
Storage Layer: Vector database, document storage
  ↓
Foundation Layer: Embedding model, LLM, database
```

Ưu điểm của thiết kế phân lớp này là mỗi lớp có thể được tối ưu và thay thế độc lập trong khi vẫn duy trì được sự ổn định của toàn hệ thống. Ví dụ, bạn có thể dễ dàng chuyển mô hình embedding từ sentence-transformers sang API Bailian mà không ảnh hưởng đến logic nghiệp vụ ở lớp trên. Tương tự, mã của quy trình xử lý hoàn toàn có thể tái sử dụng, và bạn cũng có thể chọn các phần mình cần và đưa vào dự án của riêng mình. RAGTool đóng vai trò là điểm vào thống nhất của hệ thống RAG, cung cấp một giao diện API gọn gàng.

````python
class RAGTool(Tool):
    """RAG tool

    Provides complete RAG capabilities:
    - Add multi-format documents (PDF, Office, images, audio, etc.)
    - Intelligent retrieval and recall
    - LLM-enhanced Q&A
    - Knowledge base management
    """

    def __init__(
        self,
        knowledge_base_path: str = "./knowledge_base",
        qdrant_url: str = None,
        qdrant_api_key: str = None,
        collection_name: str = "rag_knowledge_base",
        rag_namespace: str = "default"
    ):
        # Initialize RAG pipeline
        self._pipelines: Dict[str, Dict[str, Any]] = {}
        self.llm = HelloAgentsLLM()

        # Create default pipeline
        default_pipeline = create_rag_pipeline(
            qdrant_url=self.qdrant_url,
            qdrant_api_key=self.qdrant_api_key,
            collection_name=self.collection_name,
            rag_namespace=self.rag_namespace
        )
        self._pipelines[self.rag_namespace] = default_pipeline
````

Toàn bộ quy trình xử lý như sau:
```
Any format document → MarkItDown conversion → Markdown text → Intelligent chunking → Vectorization → Storage and retrieval
```

(1) Tải Tài liệu Đa phương thức

Một trong những ưu điểm cốt lõi của hệ thống RAG là năng lực xử lý tài liệu đa phương thức mạnh mẽ. Hệ thống dùng MarkItDown làm engine chuyển đổi tài liệu thống nhất, hỗ trợ hầu hết mọi định dạng tài liệu phổ biến. MarkItDown là một công cụ chuyển đổi tài liệu đa năng mã nguồn mở của Microsoft. Nó là thành phần cốt lõi của hệ thống RAG HelloAgents, chịu trách nhiệm chuyển đổi thống nhất các tài liệu ở bất kỳ định dạng nào thành văn bản Markdown có cấu trúc. Dù đầu vào là PDF, Word, Excel, hình ảnh hay âm thanh, cuối cùng nó đều sẽ được chuyển thành định dạng Markdown chuẩn, rồi đi vào quy trình chia khối (chunking), vector hóa và lưu trữ thống nhất.

```python
def _convert_to_markdown(path: str) -> str:
    """
    Universal document reader using MarkItDown with enhanced PDF processing.
    Core function: Convert documents of any format to Markdown text

    Supported formats:
    - Documents: PDF, Word, Excel, PowerPoint
    - Images: JPG, PNG, GIF (via OCR)
    - Audio: MP3, WAV, M4A (via transcription)
    - Text: TXT, CSV, JSON, XML, HTML
    - Code: Python, JavaScript, Java, etc.
    """
    if not os.path.exists(path):
        return ""

    # Use enhanced processing for PDF files
    ext = (os.path.splitext(path)[1] or '').lower()
    if ext == '.pdf':
        return _enhanced_pdf_processing(path)

    # Use MarkItDown unified conversion for other formats
    md_instance = _get_markitdown_instance()
    if md_instance is None:
        return _fallback_text_reader(path)

    try:
        result = md_instance.convert(path)
        markdown_text = getattr(result, "text_content", None)
        if isinstance(markdown_text, str) and markdown_text.strip():
            print(f"[RAG] MarkItDown conversion successful: {path} -> {len(markdown_text)} chars Markdown")
            return markdown_text
        return ""
    except Exception as e:
        print(f"[WARNING] MarkItDown conversion failed {path}: {e}")
        return _fallback_text_reader(path)
```

(2) Chiến lược Chia khối Thông minh

Sau khi chuyển đổi bằng MarkItDown, tất cả tài liệu được thống nhất thành định dạng Markdown chuẩn. Điều này cung cấp một nền tảng có cấu trúc cho việc chia khối thông minh về sau. HelloAgents hiện thực hóa một chiến lược chia khối thông minh dành riêng cho định dạng Markdown, tận dụng đầy đủ các đặc trưng có cấu trúc của Markdown để phân đoạn chính xác.

Quy trình chia khối nhận biết cấu trúc Markdown:

```
Standard Markdown text → Heading hierarchy parsing → Paragraph semantic segmentation → Token calculation chunking → Overlap strategy optimization → Vectorization preparation
       ↓                ↓              ↓            ↓           ↓            ↓
   Unified format      #/##/###      Semantic boundary  Size control  Information continuity  Embedding vector
   Clear structure     Hierarchy recognition  Integrity guarantee  Retrieval optimization  Context preservation  Similarity matching
```

Vì tất cả tài liệu đã được chuyển thành định dạng Markdown, hệ thống có thể sử dụng cấu trúc tiêu đề (heading) của Markdown (#, ##, ###, v.v.) để phân đoạn ngữ nghĩa chính xác:

```python
def _split_paragraphs_with_headings(text: str) -> List[Dict]:
    """Split paragraphs based on heading hierarchy, maintaining semantic integrity"""
    lines = text.splitlines()
    heading_stack: List[str] = []
    paragraphs: List[Dict] = []
    buf: List[str] = []
    char_pos = 0

    def flush_buf(end_pos: int):
        if not buf:
            return
        content = "\n".join(buf).strip()
        if not content:
            return
        paragraphs.append({
            "content": content,
            "heading_path": " > ".join(heading_stack) if heading_stack else None,
            "start": max(0, end_pos - len(content)),
            "end": end_pos,
        })

    for ln in lines:
        raw = ln
        if raw.strip().startswith("#"):
            # Process heading line
            flush_buf(char_pos)
            level = len(raw) - len(raw.lstrip('#'))
            title = raw.lstrip('#').strip()

            if level <= 0:
                level = 1
            if level <= len(heading_stack):
                heading_stack = heading_stack[:level-1]
            heading_stack.append(title)

            char_pos += len(raw) + 1
            continue

        # Accumulate paragraph content
        if raw.strip() == "":
            flush_buf(char_pos)
            buf = []
        else:
            buf.append(raw)
        char_pos += len(raw) + 1

    flush_buf(char_pos)

    if not paragraphs:
        paragraphs = [{"content": text, "heading_path": None, "start": 0, "end": len(text)}]

    return paragraphs
```

Dựa trên việc phân đoạn theo đoạn của Markdown, hệ thống tiếp tục thực hiện chia khối thông minh dựa trên số lượng token. Vì đầu vào đã là văn bản Markdown có cấu trúc, hệ thống có thể kiểm soát ranh giới khối (chunk boundary) chính xác hơn, đảm bảo mỗi khối vừa phù hợp để xử lý vector hóa vừa duy trì tính toàn vẹn của cấu trúc Markdown:

```python
def _chunk_paragraphs(paragraphs: List[Dict], chunk_tokens: int, overlap_tokens: int) -> List[Dict]:
    """Intelligent chunking based on token count"""
    chunks: List[Dict] = []
    cur: List[Dict] = []
    cur_tokens = 0
    i = 0

    while i < len(paragraphs):
        p = paragraphs[i]
        p_tokens = _approx_token_len(p["content"]) or 1

        if cur_tokens + p_tokens <= chunk_tokens or not cur:
            cur.append(p)
            cur_tokens += p_tokens
            i += 1
        else:
            # Generate current chunk
            content = "\n\n".join(x["content"] for x in cur)
            start = cur[0]["start"]
            end = cur[-1]["end"]
            heading_path = next((x["heading_path"] for x in reversed(cur) if x.get("heading_path")), None)

            chunks.append({
                "content": content,
                "start": start,
                "end": end,
                "heading_path": heading_path,
            })

            # Build overlap section
            if overlap_tokens > 0 and cur:
                kept: List[Dict] = []
                kept_tokens = 0
                for x in reversed(cur):
                    t = _approx_token_len(x["content"]) or 1
                    if kept_tokens + t > overlap_tokens:
                        break
                    kept.append(x)
                    kept_tokens += t
                cur = list(reversed(kept))
                cur_tokens = kept_tokens
            else:
                cur = []
                cur_tokens = 0

    # Process last chunk
    if cur:
        content = "\n\n".join(x["content"] for x in cur)
        start = cur[0]["start"]
        end = cur[-1]["end"]
        heading_path = next((x["heading_path"] for x in reversed(cur) if x.get("heading_path")), None)

        chunks.append({
            "content": content,
            "start": start,
            "end": end,
            "heading_path": heading_path,
        })

    return chunks
```

Đồng thời, để tương thích với các ngôn ngữ khác nhau, hệ thống hiện thực hóa một thuật toán ước lượng token cho văn bản hỗn hợp Trung-Anh, điều này rất quan trọng để kiểm soát chính xác kích thước khối:

```python
def _approx_token_len(text: str) -> int:
    """Approximate token length estimation, supports Chinese-English mixed text"""
    # CJK characters counted as 1 token each
    cjk = sum(1 for ch in text if _is_cjk(ch))
    # Other characters counted by whitespace tokenization
    non_cjk_tokens = len([t for t in text.split() if t])
    return cjk + non_cjk_tokens

def _is_cjk(ch: str) -> bool:
    """Determine if character is CJK"""
    code = ord(ch)
    return (
        0x4E00 <= code <= 0x9FFF or  # CJK Unified Ideographs
        0x3400 <= code <= 0x4DBF or  # CJK Extension A
        0x20000 <= code <= 0x2A6DF or # CJK Extension B
        0x2A700 <= code <= 0x2B73F or # CJK Extension C
        0x2B740 <= code <= 0x2B81F or # CJK Extension D
        0x2B820 <= code <= 0x2CEAF or # CJK Extension E
        0xF900 <= code <= 0xFAFF      # CJK Compatibility Ideographs
    )
```

(3) Nhúng Thống nhất và Lưu trữ Vector

Mô hình embedding là cốt lõi của hệ thống RAG. Nó chịu trách nhiệm chuyển văn bản thành các vector nhiều chiều, giúp máy tính có thể hiểu và so sánh độ tương đồng ngữ nghĩa của văn bản. Năng lực truy xuất của hệ thống RAG phần lớn phụ thuộc vào chất lượng của mô hình embedding và hiệu quả của việc lưu trữ vector. HelloAgents hiện thực hóa một giao diện embedding thống nhất. Vì mục đích minh họa, ở đây chúng ta dùng API Bailian. Nếu chưa cấu hình, bạn có thể chuyển sang mô hình cục bộ `all-MiniLM-L6-v2`. Nếu cả hai giải pháp đều không được hỗ trợ, thuật toán TF-IDF cũng được cấu hình như một phương án dự phòng (fallback). Trong sử dụng thực tế, bạn có thể thay thế bằng mô hình hoặc API mình muốn, hoặc thử mở rộng nội dung framework nhé~

```python
def index_chunks(
    store = None,
    chunks: List[Dict] = None,
    cache_db: Optional[str] = None,
    batch_size: int = 64,
    rag_namespace: str = "default"
) -> None:
    """
    Index markdown chunks with unified embedding and Qdrant storage.
    Uses Bailian API with fallback to sentence-transformers.
    """
    if not chunks:
        print("[RAG] No chunks to index")
        return

    # Use unified embedding model
    embedder = get_text_embedder()
    dimension = get_dimension(384)

    # Create default Qdrant storage
    if store is None:
        store = _create_default_vector_store(dimension)
        print(f"[RAG] Created default Qdrant store with dimension {dimension}")

    # Preprocess Markdown text for better embedding quality
    processed_texts = []
    for c in chunks:
        raw_content = c["content"]
        processed_content = _preprocess_markdown_for_embedding(raw_content)
        processed_texts.append(processed_content)

    print(f"[RAG] Embedding start: total_texts={len(processed_texts)} batch_size={batch_size}")

    # Batch encoding
    vecs: List[List[float]] = []
    for i in range(0, len(processed_texts), batch_size):
        part = processed_texts[i:i+batch_size]
        try:
            # Use unified embedder (handles caching internally)
            part_vecs = embedder.encode(part)

            # Standardize to List[List[float]] format
            if not isinstance(part_vecs, list):
                if hasattr(part_vecs, "tolist"):
                    part_vecs = [part_vecs.tolist()]
                else:
                    part_vecs = [list(part_vecs)]

            # Process vector format and dimension
            for v in part_vecs:
                try:
                    if hasattr(v, "tolist"):
                        v = v.tolist()
                    v_norm = [float(x) for x in v]

                    # Dimension check and adjustment
                    if len(v_norm) != dimension:
                        print(f"[WARNING] Vector dimension anomaly: expected {dimension}, actual {len(v_norm)}")
                        if len(v_norm) < dimension:
                            v_norm.extend([0.0] * (dimension - len(v_norm)))
                        else:
                            v_norm = v_norm[:dimension]

                    vecs.append(v_norm)
                except Exception as e:
                    print(f"[WARNING] Vector conversion failed: {e}, using zero vector")
                    vecs.append([0.0] * dimension)

        except Exception as e:
            print(f"[WARNING] Batch {i} encoding failed: {e}")
            # Implement retry mechanism
            # ... retry logic ...

        print(f"[RAG] Embedding progress: {min(i+batch_size, len(processed_texts))}/{len(processed_texts)}")
```

### 8.3.5 Các Chiến lược Truy xuất Nâng cao

Năng lực truy xuất của hệ thống RAG là năng lực cạnh tranh cốt lõi của nó. Trong ứng dụng thực tế, có thể có sự khác biệt về cách diễn đạt giữa truy vấn của người dùng và nội dung thực tế trong tài liệu, dẫn đến việc các tài liệu liên quan không được truy xuất. Để giải quyết vấn đề này, HelloAgents hiện thực hóa ba chiến lược truy xuất nâng cao bổ trợ lẫn nhau: Mở rộng Đa truy vấn (Multi-Query Expansion - MQE), Nhúng Tài liệu Giả định (Hypothetical Document Embeddings - HyDE), và một khung truy xuất mở rộng thống nhất.

(1) Mở rộng Đa truy vấn (MQE)

Mở rộng Đa truy vấn (Multi-Query Expansion - MQE) là một kỹ thuật cải thiện độ bao phủ truy xuất (recall) bằng cách sinh ra các truy vấn đa dạng tương đương về mặt ngữ nghĩa. Cái nhìn cốt lõi của phương pháp này là: cùng một câu hỏi có thể có nhiều cách diễn đạt khác nhau, và các cách diễn đạt khác nhau có thể khớp với các tài liệu liên quan khác nhau. Ví dụ, "cách học Python" có thể được mở rộng thành "hướng dẫn Python cho người mới bắt đầu", "phương pháp học Python", "cẩm nang lập trình Python", và các truy vấn khác. Bằng cách thực thi song song các truy vấn mở rộng này và gộp kết quả, hệ thống có thể bao phủ một phạm vi tài liệu liên quan rộng hơn, tránh bỏ sót thông tin quan trọng do khác biệt về cách diễn đạt.

Ưu điểm của MQE là nó có thể tự động hiểu nhiều ý nghĩa khả dĩ của truy vấn người dùng, đặc biệt hiệu quả với các truy vấn mơ hồ hoặc các truy vấn thuật ngữ chuyên ngành. Hệ thống dùng LLM để sinh các truy vấn mở rộng, đảm bảo tính đa dạng và độ liên quan ngữ nghĩa của các mở rộng:

```python
def _prompt_mqe(query: str, n: int) -> List[str]:
    """Use LLM to generate diverse query expansions"""
    try:
        from ...core.llm import HelloAgentsLLM
        llm = HelloAgentsLLM()
        prompt = [
            {"role": "system", "content": "You are a retrieval query expansion assistant. Generate semantically equivalent or complementary diverse queries. Use Chinese, keep it short, avoid punctuation."},
            {"role": "user", "content": f"Original query: {query}\nPlease provide {n} differently phrased queries, one per line."}
        ]
        text = llm.invoke(prompt)
        lines = [ln.strip("- \t") for ln in (text or "").splitlines()]
        outs = [ln for ln in lines if ln]
        return outs[:n] or [query]
    except Exception:
        return [query]
```

(2) Nhúng Tài liệu Giả định (HyDE)

Nhúng Tài liệu Giả định (Hypothetical Document Embeddings - HyDE) là một kỹ thuật truy xuất mang tính đổi mới. Ý tưởng cốt lõi của nó là "dùng câu trả lời để tìm câu trả lời". Các phương pháp truy xuất truyền thống dùng câu hỏi để khớp tài liệu, nhưng thường có sự khác biệt trong phân bố của câu hỏi và câu trả lời trong không gian ngữ nghĩa—câu hỏi thường là câu nghi vấn, còn nội dung tài liệu là câu trần thuật. HyDE để LLM sinh trước một đoạn câu trả lời giả định, sau đó dùng đoạn câu trả lời này để truy xuất các tài liệu thực, từ đó thu hẹp khoảng cách ngữ nghĩa giữa truy vấn và tài liệu.

Ưu điểm của phương pháp này là các câu trả lời giả định gần với câu trả lời thực hơn trong không gian ngữ nghĩa, nhờ đó có thể khớp chính xác hơn với các tài liệu liên quan. Ngay cả khi nội dung của câu trả lời giả định không hoàn toàn chính xác, các thuật ngữ then chốt, khái niệm và phong cách diễn đạt mà nó chứa vẫn có thể hướng dẫn hiệu quả hệ thống truy xuất tìm ra các tài liệu đúng. Đặc biệt với các truy vấn thuộc lĩnh vực chuyên môn, HyDE có thể sinh các tài liệu giả định chứa thuật ngữ chuyên ngành, cải thiện đáng kể độ chính xác truy xuất:

```python
def _prompt_hyde(query: str) -> Optional[str]:
    """Generate hypothetical document to improve retrieval"""
    try:
        from ...core.llm import HelloAgentsLLM
        llm = HelloAgentsLLM()
        prompt = [
            {"role": "system", "content": "Based on the user's question, first write a possible answer paragraph for use as a query document in vector retrieval (no analysis process)."},
            {"role": "user", "content": f"Question: {query}\nPlease directly write a medium-length, objective paragraph containing key terminology."}
        ]
        return llm.invoke(prompt)
    except Exception:
        return None
```

(3) Khung Truy xuất Mở rộng

HelloAgents tích hợp hai chiến lược MQE và HyDE vào một khung truy xuất mở rộng thống nhất. Hệ thống cho phép người dùng lựa chọn kích hoạt chiến lược nào dựa trên kịch bản cụ thể thông qua các tham số `enable_mqe` và `enable_hyde`: với các kịch bản đòi hỏi độ bao phủ cao, cả hai chiến lược có thể được bật đồng thời; với các kịch bản nhạy cảm về hiệu năng, chỉ dùng truy xuất cơ bản.

Cơ chế cốt lõi của truy xuất mở rộng là quy trình ba bước "mở rộng-truy xuất-gộp". Trước tiên, hệ thống sinh nhiều truy vấn mở rộng dựa trên truy vấn gốc (bao gồm các truy vấn đa dạng do MQE sinh và các tài liệu giả định do HyDE sinh); sau đó, nó thực thi truy xuất vector song song cho từng truy vấn mở rộng để thu được một nhóm tài liệu ứng viên (candidate pool); cuối cùng, nó gộp tất cả kết quả thông qua khử trùng lặp và sắp xếp theo điểm, trả về top-k tài liệu liên quan nhất. Điểm khéo léo của thiết kế này là nó mở rộng nhóm ứng viên thông qua tham số `candidate_pool_multiplier` (mặc định là 4), đảm bảo có đủ tài liệu ứng viên để sàng lọc, đồng thời tránh trả về nội dung trùng lặp thông qua khử trùng lặp thông minh.

```python
def search_vectors_expanded(
    store = None,
    query: str = "",
    top_k: int = 8,
    rag_namespace: Optional[str] = None,
    only_rag_data: bool = True,
    score_threshold: Optional[float] = None,
    enable_mqe: bool = False,
    mqe_expansions: int = 2,
    enable_hyde: bool = False,
    candidate_pool_multiplier: int = 4,
) -> List[Dict]:
    """
    Search with query expansion using unified embedding and Qdrant.
    """
    if not query:
        return []

    # Create default storage
    if store is None:
        store = _create_default_vector_store()

    # Query expansion
    expansions: List[str] = [query]

    if enable_mqe and mqe_expansions > 0:
        expansions.extend(_prompt_mqe(query, mqe_expansions))
    if enable_hyde:
        hyde_text = _prompt_hyde(query)
        if hyde_text:
            expansions.append(hyde_text)

    # Deduplication and trimming
    uniq: List[str] = []
    for e in expansions:
        if e and e not in uniq:
            uniq.append(e)
    expansions = uniq[: max(1, len(uniq))]

    # Allocate candidate pool
    pool = max(top_k * candidate_pool_multiplier, 20)
    per = max(1, pool // max(1, len(expansions)))

    # Build RAG data filter
    where = {"memory_type": "rag_chunk"}
    if only_rag_data:
        where["is_rag_data"] = True
        where["data_source"] = "rag_pipeline"
    if rag_namespace:
        where["rag_namespace"] = rag_namespace

    # Collect results from all expanded queries
    agg: Dict[str, Dict] = {}
    for q in expansions:
        qv = embed_query(q)
        hits = store.search_similar(
            query_vector=qv,
            limit=per,
            score_threshold=score_threshold,
            where=where
        )
        for h in hits:
            mid = h.get("metadata", {}).get("memory_id", h.get("id"))
            s = float(h.get("score", 0.0))
            if mid not in agg or s > float(agg[mid].get("score", 0.0)):
                agg[mid] = h

    # Sort by score and return
    merged = list(agg.values())
    merged.sort(key=lambda x: float(x.get("score", 0.0)), reverse=True)
    return merged[:top_k]
```

Trong ứng dụng thực tế, việc kết hợp sử dụng ba chiến lược này cho hiệu quả tốt nhất. MQE xuất sắc trong việc xử lý vấn đề đa dạng cách diễn đạt, HyDE xuất sắc trong việc xử lý vấn đề khoảng cách ngữ nghĩa, và khung thống nhất đảm bảo chất lượng và tính đa dạng của kết quả. Với các truy vấn tổng quát, nên bật MQE; với các truy vấn thuộc lĩnh vực chuyên môn, nên bật đồng thời cả MQE và HyDE; với các kịch bản nhạy cảm về hiệu năng, chỉ dùng truy xuất cơ bản hoặc chỉ dùng MQE.

Tất nhiên, còn nhiều phương pháp thú vị khác. Đây chỉ là một phần giới thiệu mở rộng phù hợp dành cho mọi người. Trong các kịch bản sử dụng thực tế, bạn cũng cần thử nghiệm để tìm ra giải pháp phù hợp với vấn đề.

## 8.4 Xây dựng Trợ lý Hỏi-Đáp Tài liệu Thông minh

Trong các phần trước, chúng ta đã trình bày chi tiết thiết kế và triển khai hệ thống bộ nhớ và hệ thống RAG của HelloAgents. Giờ đây, hãy cùng minh họa qua một tình huống thực hành hoàn chỉnh cách kết hợp hữu cơ hai hệ thống này để xây dựng một trợ lý hỏi-đáp tài liệu thông minh.

### 8.4.1 Bối cảnh và Mục tiêu của Tình huống

Trong công việc thực tế, chúng ta thường cần xử lý một lượng lớn tài liệu kỹ thuật, các bài báo nghiên cứu, cẩm nang sản phẩm và các file PDF khác. Các phương pháp đọc tài liệu truyền thống kém hiệu quả, khó nhanh chóng định vị thông tin then chốt, chưa nói đến việc thiết lập liên hệ giữa các tri thức.

Tình huống này sẽ dùng tài liệu PDF public beta `Happy-LLM-0727.pdf` từ một hướng dẫn thực hành mô hình lớn khác của Datawhale là Happy-LLM làm ví dụ để xây dựng một **ứng dụng Web dựa trên Gradio**, minh họa cách dùng RAGTool và MemoryTool để xây dựng một trợ lý học tập tương tác hoàn chỉnh. File PDF có thể lấy từ [liên kết này](https://github.com/datawhalechina/happy-llm/releases/download/v1.0.1/Happy-LLM-0727.pdf).

Chúng ta mong muốn hiện thực hóa các chức năng sau:

1. **Xử lý tài liệu thông minh**: Dùng MarkItDown để chuyển đổi thống nhất từ PDF sang Markdown, chiến lược chia khối thông minh dựa trên cấu trúc Markdown, vector hóa và xây dựng chỉ mục (index) hiệu quả

2. **Hỏi-đáp truy xuất nâng cao**: Mở rộng Đa truy vấn (MQE) để cải thiện độ bao phủ, Nhúng Tài liệu Giả định (HyDE) để cải thiện độ chính xác truy xuất, hỏi-đáp thông minh nhận biết ngữ cảnh

3. **Quản lý bộ nhớ đa cấp**: Bộ nhớ làm việc quản lý tác vụ học tập và ngữ cảnh hiện tại, bộ nhớ tình tiết ghi lại các sự kiện học tập và lịch sử truy vấn, bộ nhớ ngữ nghĩa lưu trữ tri thức khái niệm và sự thấu hiểu, bộ nhớ tri giác xử lý đặc trưng tài liệu và thông tin đa phương thức

4. **Hỗ trợ học tập cá nhân hóa**: Đề xuất cá nhân hóa dựa trên lịch sử học tập, củng cố bộ nhớ và quên có chọn lọc, sinh báo cáo học tập và theo dõi tiến độ

Để minh họa rõ hơn quy trình làm việc của toàn hệ thống, Hình 8.6 thể hiện các mối quan hệ và luồng dữ liệu giữa năm bước. Năm bước tạo thành một vòng khép kín hoàn chỉnh: Bước 1 ghi thông tin từ các tài liệu PDF đã xử lý vào hệ thống bộ nhớ, kết quả truy xuất của Bước 2 cũng được ghi vào hệ thống bộ nhớ, Bước 3 minh họa đầy đủ các chức năng của hệ thống bộ nhớ (thêm, truy xuất, củng cố, quên), Bước 4 tích hợp RAG và Memory để cung cấp định tuyến thông minh (intelligent routing), và Bước 5 thu thập tất cả thông tin thống kê để sinh báo cáo học tập.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-6.png" alt="" width="85%"/>
  <p>Hình 8.6 Quy trình Thực thi Năm bước của Trợ lý Hỏi-Đáp Thông minh</p>
</div>

Tiếp theo, chúng ta sẽ minh họa cách hiện thực hóa ứng dụng Web này. Toàn bộ ứng dụng được chia thành ba phần cốt lõi:

1. **Lớp trợ lý cốt lõi (PDFLearningAssistant)**: Đóng gói logic gọi của RAGTool và MemoryTool
2. **Giao diện Web Gradio**: Cung cấp giao diện tương tác thân thiện cho người dùng, phần này có thể tham khảo mã ví dụ để học
3. **Các chức năng cốt lõi khác**: Ghi chú, xem lại bài học, xem thống kê, và sinh báo cáo

### 8.4.2 Triển khai Lớp Trợ lý Cốt lõi

Trước tiên, chúng ta triển khai lớp trợ lý cốt lõi `PDFLearningAssistant`, đóng gói logic gọi của RAGTool và MemoryTool.

(1) Khởi tạo Lớp

```python
class PDFLearningAssistant:
    """Intelligent document Q&A assistant"""

    def __init__(self, user_id: str = "default_user"):
        """Initialize learning assistant

        Args:
            user_id: User ID, used to isolate data for different users
        """
        self.user_id = user_id
        self.session_id = f"session_{datetime.now().strftime('%Y%m%d_%H%M%S')}"

        # Initialize tools
        self.memory_tool = MemoryTool(user_id=user_id)
        self.rag_tool = RAGTool(rag_namespace=f"pdf_{user_id}")

        # Learning statistics
        self.stats = {
            "session_start": datetime.now(),
            "documents_loaded": 0,
            "questions_asked": 0,
            "concepts_learned": 0
        }

        # Currently loaded document
        self.current_document = None
```

Trong quá trình khởi tạo này, chúng ta đã đưa ra một số quyết định thiết kế then chốt:

**Khởi tạo MemoryTool**: Hiện thực hóa việc cô lập bộ nhớ ở cấp người dùng thông qua tham số `user_id`. Bộ nhớ học tập của các người dùng khác nhau hoàn toàn độc lập, và mỗi người dùng có không gian bộ nhớ làm việc, bộ nhớ tình tiết, bộ nhớ ngữ nghĩa và bộ nhớ tri giác riêng.

**Khởi tạo RAGTool**: Hiện thực hóa việc cô lập namespace của cơ sở tri thức thông qua tham số `rag_namespace`. Sử dụng `f"pdf_{user_id}"` làm namespace, mỗi người dùng có cơ sở tri thức PDF độc lập của riêng mình.

**Quản lý phiên**: `session_id` được dùng để theo dõi toàn bộ tiến trình của một phiên học tập đơn lẻ, tạo thuận lợi cho việc xem lại và phân tích hành trình học tập về sau.

**Thông tin thống kê**: Từ điển `stats` ghi lại các chỉ số học tập then chốt để sinh báo cáo học tập.

(2) Tải Tài liệu PDF

```python
def load_document(self, pdf_path: str) -> Dict[str, Any]:
    """Load PDF document into knowledge base

    Args:
        pdf_path: PDF file path

    Returns:
        Dict: Result containing success and message
    """
    if not os.path.exists(pdf_path):
        return {"success": False, "message": f"File does not exist: {pdf_path}"}

    start_time = time.time()

    # [RAGTool] Process PDF: MarkItDown conversion → Intelligent chunking → Vectorization
    result = self.rag_tool.execute(
        "add_document",
        file_path=pdf_path,
        chunk_size=1000,
        chunk_overlap=200
    )

    process_time = time.time() - start_time

    if result.get("success", False):
        self.current_document = os.path.basename(pdf_path)
        self.stats["documents_loaded"] += 1

        # [MemoryTool] Record to learning memory
        self.memory_tool.execute(
            "add",
            content=f"Loaded document 《{self.current_document}》",
            memory_type="episodic",
            importance=0.9,
            event_type="document_loaded",
            session_id=self.session_id
        )

        return {
            "success": True,
            "message": f"Loading successful! (Time: {process_time:.1f}s)",
            "document": self.current_document
        }
    else:
        return {
            "success": False,
            "message": f"Loading failed: {result.get('error', 'Unknown error')}"
        }
```

Chúng ta có thể hoàn thành việc xử lý PDF chỉ với một dòng mã:

```python
result = self.rag_tool.execute(
    "add_document",
    file_path=pdf_path,
    chunk_size=1000,
    chunk_overlap=200
)
```

Lệnh gọi này kích hoạt toàn bộ quy trình xử lý của RAGTool (chuyển đổi MarkItDown, xử lý nâng cao, chia khối thông minh, lưu trữ vector hóa). Các chi tiết nội bộ này đã được giới thiệu chi tiết trong Mục 8.3. Chúng ta chỉ cần tập trung vào:

- **Loại thao tác**: `"add_document"` - Thêm tài liệu vào cơ sở tri thức
- **Đường dẫn file**: `file_path` - Đường dẫn tới file PDF
- **Tham số chia khối**: `chunk_size=1000, chunk_overlap=200` - Kiểm soát việc chia khối văn bản
- **Kết quả trả về**: Từ điển chứa trạng thái xử lý và thông tin thống kê

Sau khi tài liệu được tải thành công, chúng ta dùng MemoryTool để ghi nó vào bộ nhớ tình tiết:

```python
self.memory_tool.execute(
    "add",
    content=f"Loaded document 《{self.current_document}》",
    memory_type="episodic",
    importance=0.9,
    event_type="document_loaded",
    session_id=self.session_id
)
```

**Tại sao dùng bộ nhớ tình tiết?** Bởi vì đây là một sự kiện cụ thể, có dấu thời gian, phù hợp để ghi lại bằng bộ nhớ tình tiết. Tham số `session_id` liên kết sự kiện này với phiên học tập hiện tại, tạo thuận lợi cho việc xem lại hành trình học tập về sau.

Bản ghi bộ nhớ này đặt nền tảng cho các dịch vụ cá nhân hóa về sau:

- Người dùng hỏi "Trước đây tôi đã tải những tài liệu nào?" → Truy xuất từ bộ nhớ tình tiết
- Hệ thống có thể theo dõi hành trình học tập của người dùng và việc sử dụng tài liệu

### 8.4.3 Chức năng Hỏi-Đáp Thông minh

Sau khi tài liệu được tải, người dùng có thể đặt câu hỏi về tài liệu. Chúng ta hiện thực hóa một phương thức `ask` để xử lý các câu hỏi của người dùng:

```python
def ask(self, question: str, use_advanced_search: bool = True) -> str:
    """Ask questions about the document

    Args:
        question: User question
        use_advanced_search: Whether to use advanced retrieval (MQE + HyDE)

    Returns:
        str: Answer
    """
    if not self.current_document:
        return "⚠️ Please load a document first!"

    # [MemoryTool] Record question to working memory
    self.memory_tool.execute(
        "add",
        content=f"Question: {question}",
        memory_type="working",
        importance=0.6,
        session_id=self.session_id
    )

    # [RAGTool] Use advanced retrieval to get answer
    answer = self.rag_tool.execute(
        "ask",
        question=question,
        limit=5,
        enable_advanced_search=use_advanced_search,
        enable_mqe=use_advanced_search,
        enable_hyde=use_advanced_search
    )

    # [MemoryTool] Record to episodic memory
    self.memory_tool.execute(
        "add",
        content=f"Learning about '{question}'",
        memory_type="episodic",
        importance=0.7,
        event_type="qa_interaction",
        session_id=self.session_id
    )

    self.stats["questions_asked"] += 1

    return answer
```

Khi chúng ta gọi `self.rag_tool.execute("ask", ...)`, RAGTool bên trong thực thi quy trình truy xuất nâng cao sau:

1. **Mở rộng Đa truy vấn (MQE)**:

   ```python
   # Generate diverse queries
   expanded_queries = self._generate_multi_queries(question)
   # For example, for "What is a large language model?", it might generate:
   # - "What is the definition of a large language model?"
   # - "Please explain large language models"
   # - "What does LLM mean?"
   ```

   MQE sinh các truy vấn tương đương về mặt ngữ nghĩa nhưng được diễn đạt khác nhau thông qua LLM, hiểu ý định của người dùng từ nhiều góc độ, cải thiện độ bao phủ (recall) lên 30%-50%.

2. **Nhúng Tài liệu Giả định (HyDE)**:

   - Sinh các tài liệu câu trả lời giả định, bắc cầu qua khoảng cách ngữ nghĩa giữa truy vấn và tài liệu
   - Dùng vector của các câu trả lời giả định để truy xuất

Phần triển khai nội bộ của các kỹ thuật truy xuất nâng cao này đã được giới thiệu chi tiết trong Mục 8.3.5.

### 8.4.4 Các Chức năng Cốt lõi Khác

Ngoài việc tải tài liệu và hỏi-đáp thông minh, chúng ta còn cần hiện thực hóa các chức năng như ghi chú, xem lại bài học, xem thống kê và sinh báo cáo:

```python
def add_note(self, content: str, concept: Optional[str] = None):
    """Add learning note"""
    self.memory_tool.execute(
        "add",
        content=content,
        memory_type="semantic",
        importance=0.8,
        concept=concept or "general",
        session_id=self.session_id
    )
    self.stats["concepts_learned"] += 1

def recall(self, query: str, limit: int = 5) -> str:
    """Review learning journey"""
    result = self.memory_tool.execute(
        "search",
        query=query,
        limit=limit
    )
    return result

def get_stats(self) -> Dict[str, Any]:
    """Get learning statistics"""
    duration = (datetime.now() - self.stats["session_start"]).total_seconds()
    return {
        "Session Duration": f"{duration:.0f}s",
        "Documents Loaded": self.stats["documents_loaded"],
        "Questions Asked": self.stats["questions_asked"],
        "Learning Notes": self.stats["concepts_learned"],
        "Current Document": self.current_document or "Not loaded"
    }

def generate_report(self, save_to_file: bool = True) -> Dict[str, Any]:
    """Generate learning report"""
    memory_summary = self.memory_tool.execute("summary", limit=10)
    rag_stats = self.rag_tool.execute("stats")

    duration = (datetime.now() - self.stats["session_start"]).total_seconds()
    report = {
        "session_info": {
            "session_id": self.session_id,
            "user_id": self.user_id,
            "start_time": self.stats["session_start"].isoformat(),
            "duration_seconds": duration
        },
        "learning_metrics": {
            "documents_loaded": self.stats["documents_loaded"],
            "questions_asked": self.stats["questions_asked"],
            "concepts_learned": self.stats["concepts_learned"]
        },
        "memory_summary": memory_summary,
        "rag_status": rag_stats
    }

    if save_to_file:
        report_file = f"learning_report_{self.session_id}.json"
        with open(report_file, 'w', encoding='utf-8') as f:
            json.dump(report, f, ensure_ascii=False, indent=2, default=str)
        report["report_file"] = report_file

    return report
```

Các phương thức này lần lượt hiện thực hóa:

- **add_note**: Lưu ghi chú học tập vào bộ nhớ ngữ nghĩa
- **recall**: Truy xuất hành trình học tập từ hệ thống bộ nhớ
- **get_stats**: Lấy thông tin thống kê của phiên hiện tại
- **generate_report**: Sinh báo cáo học tập chi tiết và lưu dưới dạng file JSON

### 8.4.5 Minh họa Hiệu quả Chạy thực tế

Tiếp theo là phần minh họa hiệu quả chạy thực tế. Như minh họa trong Hình 8.7, sau khi vào trang chính, bạn cần khởi tạo trợ lý trước, tức là thực hiện các thao tác tải cơ sở dữ liệu, mô hình, API và các thứ khác của chúng ta. Sau đó truyền vào tài liệu PDF và nhấn để tải tài liệu.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-7.png" alt="" width="85%"/>
  <p>Hình 8.7 Trang chính của Trợ lý Hỏi-Đáp</p>
</div>

Chức năng đầu tiên là hỏi-đáp thông minh, có thể truy xuất dựa trên các tài liệu đã tải lên và trả về nguồn tham chiếu cùng các tính toán độ tương đồng của các tài liệu liên quan. Đây là phần minh họa năng lực của công cụ RAG, như minh họa trong Hình 8.8.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-8.png" alt="" width="85%"/>
  <p>Hình 8.8 Trang chính của Trợ lý Hỏi-Đáp</p>
</div>

Chức năng thứ hai là ghi chú học tập. Như minh họa trong Hình 8.9, bạn có thể chọn các khái niệm liên quan và viết nội dung ghi chú. Phần này sử dụng công cụ Memory và sẽ lưu các ghi chú cá nhân của bạn vào cơ sở dữ liệu để dễ dàng thống kê và trả về báo cáo học tập tổng thể về sau.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-9.png" alt="" width="85%"/>
  <p>Hình 8.9 Trang chính của Trợ lý Hỏi-Đáp</p>
</div>

Cuối cùng là phần thống kê tiến độ học tập và sinh báo cáo. Như minh họa trong Hình 8.10, chúng ta có thể thấy số lượng tài liệu đã tải, số lượng câu hỏi đã đặt, và số lượng ghi chú trong quá trình sử dụng trợ lý. Cuối cùng, các kết quả hỏi-đáp và ghi chú của chúng ta được tổ chức thành một tài liệu JSON và trả về.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-10.png" alt="" width="85%"/>
  <p>Hình 8.10 Trang chính của Trợ lý Hỏi-Đáp</p>
</div>

Thông qua tình huống trợ lý hỏi-đáp này, chúng ta đã minh họa cách dùng RAGTool và MemoryTool để xây dựng một **hệ thống hỏi-đáp tài liệu thông minh dựa trên Web** hoàn chỉnh. Mã hoàn chỉnh có thể tìm thấy tại `code/chapter8/11_Q&A_Assistant.py`. Sau khi khởi động, truy cập `http://localhost:7860` để sử dụng trợ lý học tập thông minh này.

Bạn đọc được khuyến khích tự tay chạy tình huống này, trải nghiệm các năng lực của RAG và Memory, và mở rộng, tùy chỉnh trên nền tảng này để xây dựng các ứng dụng thông minh đáp ứng nhu cầu của riêng mình!

## 8.5 Tổng kết Chương và Triển vọng

Trong chương này, chúng ta đã thành công bổ sung hai năng lực cốt lõi cho framework HelloAgents: hệ thống bộ nhớ và hệ thống RAG.

Với những bạn đọc muốn học tập và ứng dụng sâu nội dung của chương này, chúng tôi đưa ra các gợi ý sau:

1. Từ số không đến số một, tự tay thiết kế một mô-đun bộ nhớ cơ bản và dần lặp lại để bổ sung các tính năng phức tạp hơn.

2. Thử nghiệm và đánh giá các mô hình embedding và chiến lược truy xuất khác nhau trong dự án để tìm ra giải pháp tối ưu cho các tác vụ cụ thể.

3. Áp dụng hệ thống memory và RAG đã học vào một dự án cá nhân thực tế, kiểm thử và cải thiện năng lực trong thực hành.

Khám phá Nâng cao

1. Theo dõi và nghiên cứu các kho lưu trữ (repository) memory và RAG tiên tiến, học hỏi các cách triển khai xuất sắc.
2. Khám phá khả năng áp dụng kiến trúc RAG vào các kịch bản đa phương thức (văn bản + hình ảnh) hoặc xuyên phương thức.
3. Tham gia dự án mã nguồn mở HelloAgents, đóng góp ý tưởng và mã của bạn.

Qua việc học chương này, bạn không chỉ nắm được công nghệ triển khai hệ thống Memory và RAG, mà quan trọng hơn, đã hiểu cách chuyển đổi lý thuyết khoa học nhận thức thành các giải pháp kỹ thuật thực tiễn. Cách tư duy liên ngành này sẽ đặt nền tảng vững chắc cho sự phát triển tiếp theo của bạn trong lĩnh vực AI.

Cuối cùng, hãy tổng kết toàn bộ hệ thống tri thức của chương này qua một sơ đồ tư duy (mind map), như minh họa trong Hình 8.11:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-11.png" alt="" width="85%"/>
  <p>Hình 8.11 Tổng kết Tri thức Chương 8 của Hello-agents</p>
</div>

Chương này đã minh họa các năng lực của hệ thống bộ nhớ và công nghệ RAG của framework HelloAgents. Chúng ta đã thành công xây dựng một trợ lý học tập thực sự "thông minh". Kiến trúc này có thể dễ dàng mở rộng sang các kịch bản ứng dụng khác, chẳng hạn như chăm sóc khách hàng, hỗ trợ kỹ thuật, trợ lý cá nhân và các lĩnh vực khác.

Trong chương tiếp theo, chúng ta sẽ tiếp tục khám phá cách cải thiện hơn nữa chất lượng hội thoại và trải nghiệm người dùng của agent thông qua kỹ thuật ngữ cảnh (context engineering). Hãy đón chờ!

## Bài tập

> **Lưu ý**: Một số bài tập không có đáp án chuẩn. Trọng tâm là rèn luyện sự hiểu biết toàn diện và năng lực thực hành về hệ thống bộ nhớ và công nghệ RAG cho người học.

1. Chương này đã giới thiệu bốn loại bộ nhớ: bộ nhớ làm việc, bộ nhớ tình tiết, bộ nhớ ngữ nghĩa và bộ nhớ tri giác. Hãy phân tích:

   - Trong Mục 8.2.5, mỗi loại bộ nhớ có một công thức tính điểm riêng. Hãy so sánh cơ chế tính điểm của bộ nhớ tình tiết và bộ nhớ ngữ nghĩa, và giải thích tại sao bộ nhớ tình tiết nhấn mạnh "tính gần đây thời gian" hơn (trọng số 0.2), trong khi bộ nhớ ngữ nghĩa nhấn mạnh "truy xuất đồ thị" hơn (trọng số 0.3)?
   - Nếu bạn phải thiết kế một "trợ lý quản lý sức khỏe cá nhân" (cần ghi lại dữ liệu ăn uống, tập luyện, giấc ngủ của người dùng, và đưa ra lời khuyên sức khỏe), bạn sẽ kết hợp bốn loại bộ nhớ này như thế nào? Hãy thiết kế các kịch bản ứng dụng cụ thể cho mỗi loại bộ nhớ.
   - Bộ nhớ làm việc dùng cơ chế TTL (Time To Live) để tự động dọn dẹp dữ liệu hết hạn. Hãy suy nghĩ: trong những trường hợp nào các bộ nhớ làm việc quan trọng nên được "củng cố" thành bộ nhớ dài hạn? Làm thế nào để thiết kế một điều kiện kích hoạt củng cố tự động?

2. Trong hệ thống RAG ở Mục 8.3, chúng ta dùng MarkItDown để chuyển đổi thống nhất các tài liệu đa định dạng sang Markdown. Hãy suy nghĩ sâu:

   > **Lưu ý**: Đây là câu hỏi thực hành, khuyến khích thao tác thực tế

   - Chiến lược chia khối thông minh hiện tại dựa trên phân cấp tiêu đề Markdown (#, ##, ###) để phân đoạn. Nếu xử lý các tài liệu không có cấu trúc tiêu đề rõ ràng (chẳng hạn như tiểu thuyết, điều khoản pháp luật), chiến lược chia khối nên được tối ưu như thế nào? Hãy thử hiện thực hóa một thuật toán chia khối dựa trên "ranh giới ngữ nghĩa (semantic boundaries)".
   - Mục 8.3.5 đã giới thiệu hai chiến lược truy xuất nâng cao: MQE (Mở rộng Đa truy vấn) và HyDE (Nhúng Tài liệu Giả định). Hãy chọn một kịch bản thực tế (chẳng hạn như hỏi-đáp tài liệu kỹ thuật, truy xuất tri thức y khoa), so sánh sự khác biệt về hiệu quả giữa truy xuất cơ bản, MQE và HyDE, và phân tích các kịch bản áp dụng phù hợp của từng loại.
   - Chất lượng truy xuất của hệ thống RAG phần lớn phụ thuộc vào việc chọn mô hình embedding. Hãy so sánh ba giải pháp embedding được đề cập trong chương này (API Bailian, Transformer cục bộ, TF-IDF) theo các chiều như độ chính xác, tốc độ, chi phí, khả năng triển khai ngoại tuyến (offline), v.v., và đưa ra khuyến nghị lựa chọn.

3. Cơ chế "quên" của hệ thống bộ nhớ là một thiết kế quan trọng mô phỏng nhận thức con người. Dựa trên MemoryTool ở Mục 8.2.3, hãy hoàn thành bài thực hành mở rộng sau:

   > **Lưu ý**: Đây là câu hỏi thực hành, khuyến khích thao tác thực tế

   - Hiện tại, ba chiến lược quên được cung cấp: dựa trên tầm quan trọng, dựa trên thời gian, và dựa trên dung lượng. Hãy thiết kế và hiện thực hóa một chiến lược "quên thông minh" xét đến một cách toàn diện các yếu tố như tầm quan trọng, tần suất truy cập, suy giảm theo thời gian, v.v., sử dụng tính điểm có trọng số để quyết định bộ nhớ nào nên bị quên.
   - Trong các hệ thống agent chạy dài hạn, cơ sở dữ liệu bộ nhớ có thể tích lũy một lượng lớn dữ liệu. Hãy thiết kế một cơ chế "lưu trữ bộ nhớ (memory archiving)": chuyển những bộ nhớ lâu không dùng nhưng có thể có giá trị sang lưu trữ lạnh (cold storage), và khôi phục chúng khi cần. Cơ chế này nên được tích hợp với bốn loại bộ nhớ hiện có như thế nào?
   - Hãy suy nghĩ: Nếu agent cần "quên" một số thông tin nhạy cảm (chẳng hạn như dữ liệu riêng tư của người dùng), liệu chỉ xóa nó khỏi cơ sở dữ liệu có đủ không? Trong trường hợp dùng cơ sở dữ liệu vector và cơ sở dữ liệu đồ thị, làm thế nào để đảm bảo dữ liệu được xóa hoàn toàn?

4. Trong tình huống "Trợ lý Học tập Thông minh" ở Mục 8.4, chúng ta đã kết hợp MemoryTool và RAGTool. Hãy phân tích sâu:

   - Phương thức `ask_question()` trong tình huống này dùng cả truy xuất RAG lẫn truy xuất bộ nhớ. Hãy phân tích: trong những trường hợp nào nên ưu tiên RAG? Trong những trường hợp nào nên ưu tiên Memory? Làm thế nào để thiết kế một cơ chế "định tuyến thông minh (intelligent routing)" tự động chọn phương pháp truy xuất phù hợp nhất?
   - Báo cáo học tập hiện tại (`generate_report()`) chỉ chứa thông tin thống kê. Hãy mở rộng chức năng này và thiết kế một bộ sinh báo cáo học tập thông minh hơn: có thể phân tích quỹ đạo học tập của người dùng, nhận diện các điểm mù tri thức (knowledge blind spots), và đề xuất nội dung học tiếp theo. Cần những loại bộ nhớ và chiến lược truy xuất nào cho việc này?
   - Giả sử bạn muốn triển khai trợ lý học tập này thành một dịch vụ Web đa người dùng, trong đó mỗi người dùng có bộ nhớ và cơ sở tri thức độc lập. Hãy thiết kế một giải pháp cô lập dữ liệu: làm thế nào để hiện thực hóa việc cô lập dữ liệu ở cấp người dùng trong Qdrant và Neo4j? Làm thế nào để tối ưu hiệu năng truy xuất trong kịch bản đa người dùng?

5. Bộ nhớ ngữ nghĩa dùng cơ sở dữ liệu đồ thị Neo4j để lưu trữ đồ thị tri thức. Hãy suy nghĩ:

   - Trong phần triển khai bộ nhớ ngữ nghĩa ở Mục 8.2.5, hệ thống tự động trích xuất các thực thể và mối quan hệ để xây dựng đồ thị tri thức. Hãy phân tích: việc trích xuất tự động này chính xác đến mức nào? Trong những trường hợp nào có thể trích xuất sai các thực thể hoặc mối quan hệ? Làm thế nào để thiết kế một cơ chế "đánh giá chất lượng đồ thị tri thức"?
   - Một ưu điểm quan trọng của đồ thị tri thức là hỗ trợ suy luận quan hệ phức tạp. Hãy thiết kế một kịch bản truy vấn tận dụng đầy đủ năng lực truy vấn đồ thị của Neo4j (chẳng hạn như quan hệ đa bước - multi-hop, tìm đường - path finding) để hoàn thành các tác vụ mà truy xuất vector thuần túy không thể hoàn thành.
   - So sánh chiến lược lai "truy xuất vector + truy xuất đồ thị" của bộ nhớ ngữ nghĩa với truy xuất vector thuần túy: trong những loại truy vấn nào truy xuất đồ thị có thể mang lại sự cải thiện hiệu năng đáng kể? Hãy minh họa bằng các ví dụ cụ thể.

## Tài liệu tham khảo

[1] Atkinson, R. C., & Shiffrin, R. M. (1968). Human memory: A proposed system and its control processes. In *Psychology of learning and motivation* (Vol. 2, pp. 89-195). Academic press.

