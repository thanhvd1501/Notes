## 🔧 1. THÔNG SỐ LLM & GENERATION

### **Q1: Temperature là bao nhiêu? Tại sao chọn giá trị đó?**

**A:** 
- **Temperature = 0.7** (trong `services/langchain_service.py` và `core/chat_engine.py`)
- **Lý do:**
  - **0.7** là giá trị cân bằng giữa tính sáng tạo và độ chính xác
  - Quá thấp (< 0.5): Response quá cứng nhắc, lặp lại
  - Quá cao (> 0.9): Response quá ngẫu nhiên, không nhất quán
  - **0.7** phù hợp cho chatbot du lịch: cần thông tin chính xác nhưng vẫn tự nhiên, thân thiện

### **Q2: Max Tokens là bao nhiêu? Tại sao?**

**A:**
- **Max Tokens = 900** (trong cả LangChain và traditional RAG)
- **Lý do:**
  - Đủ dài để trả lời chi tiết (lịch trình, món ăn, thông tin địa điểm)
  - Không quá dài để tránh tốn token và thời gian xử lý
  - Phù hợp với context window của GPT-4o-mini
  - Cân bằng giữa chất lượng response và chi phí API

### **Q3: Model nào được sử dụng? Tại sao chọn model đó?**

**A:**
- **Model: GPT-4o-mini** (có thể config qua `DEPLOYMENT_NAME` env var)
- **Lý do:**
  - **Chi phí thấp**: Phù hợp cho prototype và demo
  - **Tốc độ nhanh**: Response time tốt cho real-time chat
  - **Chất lượng đủ**: Đủ khả năng hiểu tiếng Việt và tạo response tự nhiên
  - **Flexible**: Có thể dễ dàng chuyển sang GPT-4 nếu cần chất lượng cao hơn

---

## 🎯 2. THÔNG SỐ RAG & RETRIEVAL

### **Q4: RAG_TOP_K là bao nhiêu? Tại sao chọn 5 documents?**

**A:**
- **RAG_TOP_K = 5** (trong `config/settings.py`)
- **Lý do:**
  - **Đủ thông tin**: 5 documents cung cấp đủ context để trả lời câu hỏi
  - **Không quá nhiều**: Tránh làm loãng context, giảm noise
  - **Cân bằng token**: 5 docs không làm vượt quá context window
  - **Thực nghiệm**: Sau khi test, 5 docs cho kết quả tốt nhất cho domain du lịch

### **Q5: Intent Threshold là bao nhiêu? Tại sao chọn 0.18?**

**A:**
- **INTENT_THRESHOLD = 0.18** (trong `config/settings.py`)
- **Lý do:**
  - **Cosine distance threshold**: ChromaDB trả về distance (0 = giống nhất, 1 = khác nhất)
  - **0.18** nghĩa là chỉ match intent khi distance < 0.18 (tức là similarity > 82%)
  - **Độ chính xác cao**: Threshold thấp đảm bảo chỉ match khi thực sự giống
  - **Tránh false positive**: Nếu threshold cao hơn (0.3-0.4) sẽ match nhầm nhiều intent
  - **Thực nghiệm**: Test với nhiều câu hỏi, 0.18 cho precision tốt nhất

### **Q6: Trong bước extract data (RAG retrieval), có threshold nào không?**

**A:**
- **Không có explicit threshold** cho RAG retrieval
- **Cách hoạt động:**
  - ChromaDB trả về top-k documents dựa trên cosine similarity
  - Không filter theo distance threshold (khác với intent detection)
  - **Lý do:**
    - RAG cần linh hoạt hơn: có thể không có document nào giống 100%
    - Top-k đảm bảo luôn có documents để tham khảo
    - LLM sẽ tự quyết định sử dụng thông tin nào từ retrieved docs
  - **Nếu muốn filter**: Có thể thêm `where={"distance": {"$lt": 0.5}}` trong query

### **Q7: Chunk size và overlap là bao nhiêu khi seed data vào ChromaDB?**

**A:**
- **Hiện tại: KHÔNG có chunking tự động** trong code
- **Cách seed hiện tại:**
  - Mỗi row trong CSV (`vietnam_travel_docs.csv`) được thêm trực tiếp như 1 document
  - Không split, không overlap
  - **Lý do:**
    - Data trong CSV đã được chuẩn bị sẵn (có thể đã chunked trước)
    - Mỗi row là 1 đoạn văn hoàn chỉnh về 1 chủ đề (địa điểm, món ăn, etc.)
  - **Nếu cần chunking:**
    - Có thể dùng LangChain's `RecursiveCharacterTextSplitter`
    - **Chunk size đề xuất**: 500-1000 characters (phù hợp với embedding model)
    - **Overlap đề xuất**: 50-100 characters (để không mất context giữa các chunks)

---

## 🧠 3. EMBEDDING & VECTOR DATABASE

### **Q8: Embedding model nào được dùng? Dimension là bao nhiêu?**

**A:**
- **Model: `sentence-transformers/all-MiniLM-L6-v2`**
- **Dimension: 384**
- **Lý do chọn:**
  - **Nhẹ và nhanh**: Model nhỏ, phù hợp cho real-time chat
  - **Hỗ trợ đa ngôn ngữ**: Tốt với tiếng Việt
  - **Dimension 384**: Đủ để capture semantic meaning, không quá lớn
  - **Local deployment**: Có thể tải về và chạy local (không cần API)
  - **Chi phí thấp**: Không tốn tiền như OpenAI embeddings

### **Q9: ChromaDB được dùng như thế nào? Có những collection nào?**

**A:**
- **3 collections chính:**
  1. **`vietnam_travel_v2`**: Knowledge base về du lịch Việt Nam (RAG)
  2. **`chat_memory_v2`**: Lưu lịch sử chat (memory recall)
  3. **`intent_bank_v2`**: Lưu các intent mẫu (intent detection)
- **Lý do tách collection:**
  - **Tách biệt mục đích**: Mỗi collection phục vụ 1 mục đích riêng
  - **Dễ quản lý**: Có thể xóa/update từng collection độc lập
  - **Performance**: Query nhanh hơn khi collection nhỏ hơn
  - **Metadata khác nhau**: Mỗi collection có metadata structure riêng

---

## 💾 4. DATABASE & STORAGE

### **Q10: Dùng database gì để lưu data? Tại sao?**

**A:**
- **2 loại database:**
  1. **ChromaDB (Vector Database)**: Lưu embeddings và documents
     - **Lý do**: Cần vector search cho RAG và semantic search
  2. **SQLite (Relational Database)**: Lưu interaction logs
     - **File: `travel_chatbot_logs.db`**
     - **Lý do:**
       - **Đơn giản**: Không cần server riêng, file-based
       - **Đủ cho analytics**: SQLite đủ mạnh cho logging và analytics
       - **Dễ deploy**: Không cần setup database server
       - **Phù hợp prototype**: Đủ cho demo và thuyết trình

### **Q11: Có lưu history chat không? Làm thế nào?**

**A:**
- **Có, lưu ở 2 nơi:**
  1. **Streamlit Session State** (`st.session_state.messages`):
     - Lưu trong memory của session hiện tại
     - **Mất khi refresh/đóng tab**: Chỉ tồn tại trong session
     - **Dùng để**: Hiển thị chat UI, pass vào LLM context
  2. **ChromaDB Memory Collection** (`chat_memory_v2`):
     - Lưu vĩnh viễn với embeddings
     - **Dùng để**: Semantic search các cuộc hội thoại trước
     - **Metadata**: role (user/assistant), city, timestamp
  3. **SQLite** (`interactions` table):
     - Lưu metadata: user_input, city, dates, intent, RAG usage
     - **Dùng để**: Analytics, không dùng để recall conversation

### **Q12: Có bước load history không? Làm thế nào?**

**A:**
- **Có, nhưng chỉ trong session:**
  - **Streamlit tự động persist** `st.session_state.messages` trong session
  - **Khi user refresh**: Streamlit giữ lại session state (nếu không clear)
  - **Khi mở tab mới**: Session mới → không có history
- **Memory recall (semantic search):**
  - **Không load toàn bộ history**
  - **Chỉ recall relevant memories**: `ChromaService.recall_memories(user_input, k=3)`
  - **Top-3 similar conversations** được thêm vào context
  - **Lý do**: Tránh context quá dài, chỉ lấy thông tin liên quan

---

## 🔄 5. MEMORY & CONTEXT MANAGEMENT

### **Q13: MEMORY_RECALL_K là bao nhiêu? Tại sao?**

**A:**
- **MEMORY_RECALL_K = 3** (trong `config/settings.py`)
- **Lý do:**
  - **Đủ context**: 3 memories cung cấp đủ thông tin về cuộc hội thoại trước
  - **Không quá nhiều**: Tránh làm loãng context hiện tại
  - **Cân bằng token**: 3 memories không làm vượt context window
  - **Thực nghiệm**: Test cho thấy 3 là số lượng tối ưu

### **Q14: Conversation history được giữ bao nhiêu messages?**

**A:**
- **12 messages** (last 12 messages)
- **Trong code:**
  - LangChain: `ConversationBufferWindowMemory(k=12)`
  - Traditional RAG: `conversation_history[-12:]`
- **Lý do:**
  - **Đủ context**: 12 messages ≈ 6 cặp Q&A, đủ để hiểu context
  - **Không quá dài**: Tránh vượt context window và tốn token
  - **Cân bằng**: Đủ để maintain conversation flow, không quá nặng

---

## 🏗️ 6. ARCHITECTURE & DESIGN

### **Q15: Tại sao có 2 cách implement RAG (LangChain vs Traditional)?**

**A:**
- **Fallback pattern:**
  1. **LangChain RAG** (ưu tiên): Dùng `ConversationalRetrievalChain`
     - **Ưu điểm**: Tự động, dễ maintain, tích hợp memory tốt
  2. **Traditional RAG** (fallback): Manual retrieve + LLM call
     - **Khi nào dùng**: Khi LangChain không available hoặc fail
     - **Ưu điểm**: Không phụ thuộc LangChain, control tốt hơn
- **Lý do:**
  - **Resilience**: Hệ thống vẫn hoạt động nếu LangChain có vấn đề
  - **Flexibility**: Có thể tắt LangChain nếu muốn
  - **Learning**: Hiểu rõ cách RAG hoạt động ở cả 2 levels

### **Q16: Nguồn dữ liệu du lịch từ đâu?**

**A:**
- **File CSV: `data/vietnam_travel_docs.csv`**
- **Cấu trúc**: Có các cột: `id`, `title`, `city`, `text/description`, `source`
- **Nguồn có thể:**
  - Tổng hợp từ các website du lịch
  - Wikipedia về các địa điểm Việt Nam
  - Blog du lịch, review
  - Tài liệu chính thức về du lịch
- **Lưu ý**: Cần đảm bảo bản quyền và chất lượng data

---

## 📊 7. PERFORMANCE & OPTIMIZATION

### **Q17: Có cache không? Cache ở đâu?**

**A:**
- **Có, Streamlit cache:**
  - `@st.cache_resource`: Cache embedding model (không reload mỗi lần)
  - `@st.cache_data`: Có thể cache data loading (nếu cần)
- **ChromaDB persistence:**
  - ChromaDB tự động persist data vào disk
  - Không cần reload embeddings mỗi lần
- **Lý do:**
  - **Performance**: Embedding model lớn, cache giúp khởi động nhanh
  - **Cost**: Không cần regenerate embeddings mỗi lần

### **Q18: Có xử lý lỗi không? Xử lý như thế nào?**

**A:**
- **Có, nhiều lớp:**
  1. **Try-catch blocks**: Bọc các API calls và operations
  2. **Fallback mechanisms**: LangChain → Traditional RAG → Direct LLM
  3. **Default values**: Nếu không có data, dùng defaults
  4. **Error messages**: Hiển thị lỗi thân thiện cho user
- **Ví dụ:**
  - Nếu RAG fail → fallback to direct LLM
  - Nếu weather API fail → bỏ qua, không block response
  - Nếu embedding fail → return empty results

---

## 🎨 8. UI/UX

### **Q19: Voice input/output được implement như thế nào?**

**A:**
- **Speech-to-Text**: Google Speech Recognition API
  - Language: `vi-VN` (tiếng Việt)
  - Format: WAV, OGG, WebM, MP3
- **Text-to-Speech**: Google Text-to-Speech (gTTS)
  - Language: `vi` (tiếng Việt)
  - Optional: Có thể bật/tắt trong UI
- **Lý do chọn Google:**
  - **Free tier**: Đủ cho demo
  - **Hỗ trợ tiếng Việt tốt**: Accuracy cao
  - **Dễ integrate**: API đơn giản

---

## 🔐 9. SECURITY & CONFIGURATION

### **Q20: API keys được quản lý như thế nào?**

**A:**
- **Environment variables** hoặc **Streamlit secrets**
- **Các keys cần:**
  - `OPENAI_API_KEY`: OpenAI API
  - `OPENWEATHERMAP_API_KEY`: Weather data
  - `PIXABAY_API_KEY`: Images
  - `PLACES_API_KEY`: Google Places (geocoding)
- **Best practices:**
  - Không hardcode trong code
  - Dùng `.env` file (local) hoặc Streamlit secrets (deploy)
  - Không commit keys vào git

---

## 📈 10. ANALYTICS & MONITORING

### **Q21: Có tracking/logging gì không?**

**A:**
- **SQLite logging** (`LoggerService`):
  - Lưu mỗi interaction: timestamp, user_input, city, dates, intent, RAG usage, sources_count
  - **Dùng để**: Analytics, debug, improve system
- **UI Analytics Dashboard**:
  - Hiển thị stats: total interactions, RAG usage rate, popular cities, etc.
  - Charts: Plotly visualizations

---

## 🚀 11. DEPLOYMENT & SCALABILITY

### **Q22: Hệ thống có thể scale không? Làm thế nào?**

**A:**
- **Hiện tại: Single instance** (phù hợp prototype)
- **Có thể scale:**
  1. **Horizontal scaling**: Deploy nhiều instances, dùng load balancer
  2. **Database**: Chuyển SQLite → PostgreSQL, ChromaDB → cloud version
  3. **Caching**: Redis cho session state
  4. **API**: Tách backend (FastAPI) và frontend (Streamlit)
- **Bottlenecks hiện tại:**
  - Embedding generation (CPU-bound)
  - LLM API calls (network-bound)
  - ChromaDB queries (I/O-bound)

---

## 💡 12. FUTURE IMPROVEMENTS

### **Q23: Có kế hoạch cải thiện gì không?**

**A:**
- **Short-term:**
  - Thêm chunking strategy cho documents
  - Fine-tune embedding model cho domain du lịch
  - Thêm web search fallback khi không có RAG results
- **Long-term:**
  - Multi-language support
  - User authentication & profiles
  - FastAPI backend
  - Docker containerization
  - CI/CD pipeline

---

## 📝 TÓM TẮT CÁC THÔNG SỐ QUAN TRỌNG

| Thông số | Giá trị | Vị trí trong code |
|----------|---------|-------------------|
| **Temperature** | 0.7 | `services/langchain_service.py:46`, `core/chat_engine.py:188` |
| **Max Tokens** | 900 | `services/langchain_service.py:47`, `core/chat_engine.py:187` |
| **Model** | gpt-4o-mini | `config/settings.py:23` |
| **RAG_TOP_K** | 5 | `config/settings.py:41` |
| **INTENT_THRESHOLD** | 0.18 | `config/settings.py:42` |
| **MEMORY_RECALL_K** | 3 | `config/settings.py:43` |
| **Conversation History** | 12 messages | `services/langchain_service.py:74`, `core/chat_engine.py:179` |
| **Embedding Model** | all-MiniLM-L6-v2 | `config/settings.py:32` |
| **Embedding Dimension** | 384 | `config/settings.py:33` |
| **Chunk Size** | N/A (no chunking) | - |
| **Overlap** | N/A (no chunking) | - |

---

## ⚡ 13. PERFORMANCE & LATENCY

### **Q24: Response time trung bình là bao nhiêu? Có bottleneck nào không?**

**A:**
- **Response time ước tính:**
  - **Intent detection**: ~100-200ms (embedding + vector search)
  - **RAG retrieval**: ~200-300ms (embedding + vector search)
  - **LLM generation**: ~2-5 giây (phụ thuộc vào OpenAI API)
  - **Total**: ~3-6 giây cho 1 response đầy đủ
- **Bottlenecks:**
  1. **LLM API call**: Chiếm 70-80% thời gian (network latency + generation)
  2. **Embedding generation**: ~100-200ms mỗi lần (CPU-bound)
  3. **ChromaDB query**: ~50-100ms (I/O-bound)
- **Optimization:**
  - Cache embeddings (đã implement với `@st.cache_resource`)
  - Parallel API calls cho weather/images (nếu có thể)
  - Timeout settings: 8-10 giây cho external APIs

### **Q25: Có timeout settings không? Giá trị là bao nhiêu?**

**A:**
- **Có, timeout cho các API calls:**
  - **Weather API**: 8 giây (`services/weather_service.py:65`)
  - **Image API (Pixabay)**: 8 giây (`services/image_service.py:32`)
  - **Geocoding API**: 10 giây (`services/geocoding_service.py:24`)
  - **Restaurant API (Google Places)**: 10 giây (`services/restaurant_service.py:27`)
- **Lý do:**
  - **Đủ thời gian**: 8-10 giây đủ cho hầu hết API calls
  - **Không block quá lâu**: Tránh user phải chờ quá lâu
  - **Fallback**: Nếu timeout, hệ thống vẫn tiếp tục với data có sẵn

### **Q26: Có retry mechanism không? Xử lý rate limiting như thế nào?**

**A:**
- **Hiện tại: KHÔNG có retry mechanism tự động**
- **Xử lý lỗi:**
  - Try-catch blocks bọc tất cả API calls
  - Fallback mechanisms: Nếu API fail → dùng data local hoặc bỏ qua
  - Error messages thân thiện cho user
- **Rate limiting:**
  - **Chưa implement**: Dựa vào API provider's rate limits
  - **OpenAI**: Có rate limits theo tier (free/paid)
  - **OpenWeatherMap**: 60 calls/phút (free tier)
  - **Google Places**: 1000 requests/ngày (free tier)
- **Cải thiện có thể:**
  - Implement exponential backoff retry
  - Rate limiting middleware
  - Queue system cho API calls

---

## 💰 14. COST & TOKEN USAGE

### **Q27: Ước tính chi phí cho mỗi request là bao nhiêu?**

**A:**
- **Token usage ước tính mỗi request:**
  - **Input tokens**: ~500-1000 tokens (user input + context + RAG docs)
  - **Output tokens**: ~200-400 tokens (response ~900 max tokens)
  - **Total**: ~700-1400 tokens/request
- **Chi phí (GPT-4o-mini):**
  - **Input**: $0.15 per 1M tokens
  - **Output**: $0.60 per 1M tokens
  - **Cost/request**: ~$0.0005-0.001 (0.5-1 cent)
  - **1000 requests**: ~$0.50-1.00
- **Các chi phí khác:**
  - **Embedding**: Free (local model)
  - **ChromaDB**: Free (local)
  - **External APIs**: Free tier đủ cho demo
- **Tối ưu chi phí:**
  - Giới hạn context length (12 messages)
  - Giới hạn RAG docs (top-5)
  - Cache embeddings

### **Q28: Có tracking token usage không? Làm thế nào monitor cost?**

**A:**
- **Hiện tại: KHÔNG có tracking tự động**
- **Có thể track:**
  - OpenAI API response có `usage` field (tokens used)
  - Có thể log vào SQLite database
  - Streamlit có thể hiển thị usage stats
- **Monitor cost:**
  - OpenAI Dashboard: Xem usage theo ngày/tháng
  - Set up billing alerts
  - Log token usage vào database để analyze
- **Cải thiện có thể:**
  - Add token usage tracking vào `LoggerService`
  - Dashboard hiển thị cost per request
  - Budget alerts

---

## 🧪 15. TESTING & EVALUATION

### **Q29: Có test cases không? Làm thế nào evaluate chất lượng response?**

**A:**
- **Hiện tại: Manual testing**
  - Test với các câu hỏi mẫu
  - Kiểm tra accuracy của intent detection
  - Kiểm tra RAG retrieval quality
- **Evaluation metrics có thể dùng:**
  1. **Accuracy**: % câu hỏi được trả lời đúng
  2. **Relevance**: RAG docs có liên quan không
  3. **Completeness**: Response có đầy đủ thông tin không
  4. **Latency**: Response time
  5. **User satisfaction**: Feedback từ users
- **Cải thiện có thể:**
  - Unit tests cho các services
  - Integration tests cho RAG pipeline
  - A/B testing cho các thông số (temperature, top-k, etc.)
  - Automated evaluation với test dataset

### **Q30: Có validation cho data quality không?**

**A:**
- **Basic validation:**
  - Check API keys tồn tại
  - Check file paths tồn tại
  - Try-catch cho data loading
- **Data quality checks:**
  - CSV files: Check columns, empty rows
  - ChromaDB: Check collection exists, dimension match
  - Embeddings: Check dimension consistency
- **Cải thiện có thể:**
  - Schema validation cho CSV data
  - Data quality metrics (completeness, accuracy)
  - Automated data validation pipeline

---

## 🏛️ 16. TECHNICAL DECISIONS

### **Q31: Tại sao chọn Streamlit thay vì Flask/FastAPI + React?**

**A:**
- **Streamlit advantages:**
  - **Rapid prototyping**: Build UI nhanh, ít code
  - **Python-native**: Không cần học frontend framework
  - **Built-in components**: Chat, forms, charts sẵn có
  - **Perfect for demo**: Phù hợp cho prototype và thuyết trình
- **Trade-offs:**
  - **Less flexible**: Khó customize UI phức tạp
  - **Performance**: Không tối ưu bằng custom frontend
  - **Scalability**: Khó scale như FastAPI + React
- **Khi nào nên chuyển:**
  - Khi cần UI phức tạp hơn
  - Khi cần scale lớn
  - Khi cần separation of concerns (backend/frontend)

### **Q32: Tại sao chọn ChromaDB thay vì Pinecone/Weaviate/Qdrant?**

**A:**
- **ChromaDB advantages:**
  - **Local deployment**: Không cần cloud service, free
  - **Easy setup**: Install và chạy ngay, không cần config phức tạp
  - **Python-native**: API đơn giản, dễ integrate
  - **Persistent storage**: Tự động lưu vào disk
  - **Perfect for prototype**: Đủ mạnh cho demo, không tốn chi phí
- **Trade-offs:**
  - **Scalability**: Khó scale như cloud solutions
  - **Performance**: Có thể chậm hơn với dataset lớn
  - **Features**: Ít features hơn các cloud solutions
- **Khi nào nên chuyển:**
  - Khi cần scale lớn (millions of documents)
  - Khi cần distributed deployment
  - Khi cần advanced features (hybrid search, etc.)

### **Q33: Tại sao dùng SentenceTransformers thay vì OpenAI embeddings?**

**A:**
- **SentenceTransformers advantages:**
  - **Free**: Không tốn chi phí
  - **Local**: Chạy local, không phụ thuộc API
  - **Fast**: Nhanh cho real-time chat
  - **Good quality**: Đủ tốt cho domain du lịch
  - **Privacy**: Data không gửi lên cloud
- **Trade-offs:**
  - **Quality**: Có thể kém hơn OpenAI embeddings (text-embedding-3-small)
  - **Dimension**: 384 vs 1536 (OpenAI) - nhỏ hơn
  - **Language support**: Có thể kém hơn với một số ngôn ngữ
- **Khi nào nên chuyển:**
  - Khi cần quality cao hơn
  - Khi có budget cho OpenAI embeddings
  - Khi cần dimension lớn hơn

---

## 🚧 17. LIMITATIONS & CHALLENGES

### **Q34: Hệ thống có limitations gì? Những thách thức đã gặp?**

**A:**
- **Limitations:**
  1. **Language**: Chủ yếu hỗ trợ tiếng Việt, chưa đa ngôn ngữ
  2. **Domain**: Chỉ hỗ trợ du lịch Việt Nam
  3. **Data**: Phụ thuộc vào quality của CSV data
  4. **Scalability**: Single instance, chưa scale được
  5. **Real-time updates**: Data không update real-time
  6. **Context window**: Giới hạn 12 messages, có thể mất context dài hạn
- **Challenges đã gặp:**
  1. **Intent detection accuracy**: Phải tune threshold nhiều lần
  2. **RAG quality**: Phải test nhiều top-k values
  3. **API rate limits**: Phải handle fallbacks
  4. **Memory management**: Balance giữa context và token usage
  5. **Error handling**: Phải cover nhiều edge cases

### **Q35: Có xử lý edge cases không? Ví dụ?**

**A:**
- **Có, nhiều edge cases được xử lý:**
  1. **Missing API keys**: Fallback to local data hoặc skip feature
  2. **API failures**: Try-catch, return default/empty results
  3. **Invalid city names**: AI resolution, fallback suggestions
  4. **No RAG results**: Fallback to direct LLM
  5. **Empty conversation history**: Initialize với system prompt
  6. **Date parsing errors**: Default to today
  7. **ChromaDB errors**: Graceful degradation
  8. **Embedding failures**: Return empty results, không crash
- **Ví dụ cụ thể:**
  - User hỏi về thành phố không tồn tại → AI resolution
  - Weather API fail → Bỏ qua, không block response
  - RAG không tìm thấy docs → LLM dùng knowledge base

---

## 🔄 18. COMPARISON & ALTERNATIVES

### **Q36: So sánh với các approach khác (fine-tuning, prompt engineering thuần)?**

**A:**
- **RAG vs Fine-tuning:**
  - **RAG (hiện tại)**:
    - ✅ Dễ update data (chỉ cần update ChromaDB)
    - ✅ Không cần retrain model
    - ✅ Có thể trace sources
    - ❌ Phụ thuộc vào retrieval quality
    - ❌ Context window limit
  - **Fine-tuning**:
    - ✅ Model học được domain knowledge
    - ✅ Không cần retrieval step
    - ❌ Cần retrain khi update data
    - ❌ Khó trace sources
    - ❌ Tốn chi phí training
- **RAG vs Prompt Engineering:**
  - **RAG (hiện tại)**:
    - ✅ Có thể tham khảo external data
    - ✅ Có thể update knowledge base
    - ❌ Phức tạp hơn
  - **Prompt Engineering**:
    - ✅ Đơn giản hơn
    - ❌ Không thể tham khảo external data
    - ❌ Khó update knowledge
- **Kết luận**: RAG phù hợp cho use case này vì cần external data và dễ update

### **Q37: Tại sao không dùng LangChain hoàn toàn mà có traditional RAG fallback?**

**A:**
- **Lý do:**
  1. **Resilience**: Hệ thống vẫn hoạt động nếu LangChain có bug/issue
  2. **Flexibility**: Có thể tắt LangChain nếu muốn
  3. **Learning**: Hiểu rõ cách RAG hoạt động ở cả 2 levels
  4. **Control**: Traditional RAG cho control tốt hơn
  5. **Debugging**: Dễ debug hơn khi có 2 implementations
- **Trade-off:**
  - Code phức tạp hơn
  - Phải maintain 2 code paths
  - Nhưng đảm bảo reliability

---

## 🌐 19. MULTI-LANGUAGE & INTERNATIONALIZATION

### **Q38: Hệ thống có hỗ trợ đa ngôn ngữ không?**

**A:**
- **Hiện tại: Chủ yếu tiếng Việt**
  - **System prompt**: Tiếng Việt
  - **Embedding model**: Hỗ trợ đa ngôn ngữ nhưng optimize cho tiếng Việt
  - **Voice**: `vi-VN` (tiếng Việt)
  - **LLM**: Có thể hiểu tiếng Anh nhưng response chủ yếu tiếng Việt
- **Có thể mở rộng:**
  - Thêm language detection
  - Multi-language prompts
  - Language-specific embeddings
  - Translation layer

---

## 📊 20. MONITORING & DEBUGGING

### **Q39: Làm thế nào debug khi có vấn đề? Có logging không?**

**A:**
- **Logging:**
  - **SQLite logs**: Lưu mỗi interaction với metadata
  - **Console logs**: Print statements trong code
  - **Streamlit logs**: Streamlit tự động log errors
- **Debugging tools:**
  - **Analytics dashboard**: Xem stats, popular queries
  - **ChromaDB queries**: Có thể query trực tiếp ChromaDB
  - **Session state**: Streamlit session state inspector
- **Cải thiện có thể:**
  - Structured logging (JSON format)
  - Log levels (DEBUG, INFO, WARNING, ERROR)
  - Centralized logging service
  - Error tracking (Sentry, etc.)

### **Q40: Có metrics nào để monitor hệ thống không?**

**A:**
- **Hiện tại: Basic metrics trong analytics dashboard**
  - Total interactions
  - RAG usage rate
  - Popular cities
  - Intent distribution
- **Metrics nên có:**
  - Response time (p50, p95, p99)
  - Error rate
  - Token usage
  - API call success rate
  - User satisfaction (nếu có feedback)
- **Cải thiện có thể:**
  - Real-time monitoring dashboard
  - Alerts cho errors/performance issues
  - A/B testing framework

---

## 🎯 TIPS CHO BUỔI THUYẾT TRÌNH

1. **Nhấn mạnh các quyết định thiết kế**: Giải thích rõ lý do chọn mỗi thông số
2. **Chuẩn bị demo**: Show các thông số trong action
3. **So sánh alternatives**: Nói về các lựa chọn khác và tại sao không chọn
4. **Thực nghiệm**: Nếu có, mention về quá trình test và tune các thông số
5. **Trade-offs**: Nói về trade-offs giữa các lựa chọn (cost vs quality, speed vs accuracy)
6. **Limitations**: Thành thật về limitations và cách xử lý
7. **Future work**: Nói về kế hoạch cải thiện
8. **Demo scenarios**: Chuẩn bị các câu hỏi mẫu để demo

---

**Chúc bạn thuyết trình thành công! 🎉**


