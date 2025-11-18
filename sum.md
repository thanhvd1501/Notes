# BÁO CÁO PHÂN TÍCH TOÀN DIỆN - TRAVEL CHAT BOT AI

## 1. TỔNG QUAN HỆ THỐNG

### 1.1 Loại chatbot
**Mây Lang Thang** là một **RAG-powered Travel Assistant Chatbot** chuyên về du lịch Việt Nam với các đặc điểm:

- **Chatbot du lịch thông minh**: Tư vấn lịch trình, địa điểm, ẩm thực, thời tiết cho các điểm đến tại Việt Nam
- **RAG (Retrieval-Augmented Generation)**: Kết hợp ChromaDB vector database để truy xuất tri thức từ knowledge base
- **Multi-modal**: Hỗ trợ cả text và voice input/output (Speech-to-Text & Text-to-Speech)
- **Context-aware**: Hệ thống memory kép (LangChain + ChromaDB) để nhớ ngữ cảnh hội thoại

### 1.2 Dạng ứng dụng
- **Web Application** sử dụng **Streamlit** framework
- Chạy trên local hoặc deploy lên Streamlit Cloud/Heroku
- Interface: Browser-based với UI hiện đại, responsive

### 1.3 Điểm đặc biệt & mục tiêu chính

**Điểm nổi bật:**
- ✅ **Kiến trúc modular** rất rõ ràng (core, services, ui, config, utils)
- ✅ **Dual RAG system**: LangChain RAG (ưu tiên) + Traditional RAG (fallback)
- ✅ **Dual Memory**: LangChain ConversationBufferWindowMemory + ChromaDB long-term memory
- ✅ **Intent Detection**: Phát hiện ý định qua semantic search trên ChromaDB
- ✅ **Multi-service integration**: Weather API, Geocoding, Image API, Food/Restaurant data
- ✅ **Voice-enabled**: STT/TTS cho trải nghiệm hands-free
- ✅ **Analytics dashboard**: Logging và thống kê truy vấn vào SQLite

**Mục tiêu:**
Cung cấp trợ lý du lịch AI toàn diện, thay thế hướng dẫn viên thật với khả năng tư vấn 24/7, cá nhân hóa theo ngữ cảnh và ngân sách.

---

## 2. TECH STACK & THỨ VIỆN CHÍNH

### 2.1 Ngôn ngữ & Framework
- **Python 3.8+**
- **Streamlit 1.50.0**: Web UI framework

### 2.2 AI/LLM Stack
```
OpenAI API (gpt-4o-mini):
├── Chat completions cho conversation
├── Entity extraction (city, dates)
├── Intent handling
└── Travel content generation

LangChain Stack:
├── langchain>=0.3.0: Core framework
├── langchain-openai>=0.2.0: OpenAI integration
├── langchain-community>=0.3.0: Community integrations
└── ConversationalRetrievalChain: RAG with memory
```

**Vai trò**: LangChain làm orchestration layer cho RAG pipeline, tự động hóa việc retrieve documents + generate response + manage memory.

### 2.3 Vector DB / RAG
```
ChromaDB 1.2.1:
├── Persistent vector storage
├── 3 collections: travel_docs, memory, intent_bank
└── Sentence Transformers embeddings

Sentence Transformers:
├── Model: all-MiniLM-L6-v2 (384 dimensions)
├── Local caching tại data/all-MiniLM-L6-v2/
└── Fast semantic search
```

**Vai trò**: ChromaDB lưu trữ tri thức du lịch, memory cuộc trò chuyện, và intent patterns. Sentence Transformers tạo embeddings cho semantic matching.

### 2.4 External APIs & Services
```
OpenWeatherMap API:
└── Dự báo thời tiết 5 ngày (forecast endpoint)

Google Places API:
└── (Dự phòng - chưa thấy sử dụng nhiều trong code)

Pixabay API:
└── Lấy hình ảnh minh họa cho city/food

Geopy:
└── Geocoding địa danh thành tọa độ (lat/lon)

PyDeck:
└── Render bản đồ 3D interactive
```

### 2.5 Voice Processing
```
SpeechRecognition:
└── Google Speech Recognition API (STT)

gTTS (Google Text-to-Speech):
└── Chuyển text → audio (TTS)

Pydub + FFmpeg:
└── Audio format conversion (WebM/OGG/MP3 → WAV)
```

### 2.6 Data & Storage
```
SQLite:
└── travel_chatbot_logs.db - Log interactions

CSV Files:
├── vietnam_foods.csv (11 cities)
├── restaurants_vn.csv (fallback data)
└── vietnam_travel_docs.csv (~20+ documents)

Pandas:
└── Data manipulation và analytics
```

### 2.7 UI & Visualization
```
Streamlit 1.50.0:
└── Main UI framework

Streamlit-mic-recorder:
└── Voice recording widget

Plotly 6.3.1:
└── Interactive charts (analytics dashboard)
```

### 2.8 Utilities
```
python-dotenv: Environment variables
Requests: HTTP API calls
```

---

## 3. CẤU TRÚC THƯ MỤC & KIẾN TRÚC TỔNG THỂ

### 3.1 Cây thư mục (high-level)

```
Travel_Chat_Bot_AI/
│
├── core/                          # 🧠 Business Logic Layer
│   ├── chat_engine.py             # Orchestrator chính
│   ├── entity_extractor.py        # Trích xuất city, dates
│   └── intent_detector.py         # Phát hiện ý định user
│
├── services/                      # 🔧 Service Layer (External APIs)
│   ├── chroma_service.py          # ChromaDB operations (RAG, Memory, Intent)
│   ├── langchain_service.py       # LangChain RAG chains & memory
│   ├── voice_service.py           # STT/TTS processing
│   ├── logger_service.py          # SQLite logging
│   ├── weather_service.py         # OpenWeatherMap API
│   ├── geocoding_service.py       # Geopy geocoding + PyDeck maps
│   ├── image_service.py           # Pixabay image API
│   ├── food_service.py            # Food recommendations (CSV + AI fallback)
│   └── restaurant_service.py      # Restaurant data
│
├── ui/                            # 🎨 Presentation Layer
│   └── app.py                     # Streamlit UI (953 lines)
│
├── config/                        # ⚙️ Configuration Layer
│   └── settings.py                # Centralized settings + env vars
│
├── utils/                         # 🛠️ Utility Functions
│   ├── text_processing.py         # Text parsing (extract days, split foods)
│   └── date_utils.py              # Date handling
│
├── data/                          # 📊 Data Assets
│   ├── vietnam_foods.csv          # 11 cities × món ăn
│   ├── restaurants_vn.csv         # Restaurant fallback
│   ├── vietnam_travel_docs.csv    # ~20 documents (RAG knowledge base)
│   ├── all-MiniLM-L6-v2/          # Local embedding model cache
│   └── chroma_storage/            # ChromaDB persistent data
│
├── chromadb_data/                 # 🗄️ ChromaDB Persistent Storage
│   └── (collections: vietnam_travel_v2, chat_memory_v2, intent_bank_v2)
│
├── main.py                        # 🚀 Entry Point
├── requirements.txt               # 📦 Dependencies (265 lines)
├── .env                           # 🔐 Environment variables
├── travel_chatbot_logs.db         # 📝 SQLite logs
├── ARCHITECTURE.md                # 📖 Architecture documentation
└── README.md                      # 📘 User guide
```

### 3.2 Giải thích từng layer

#### **core/** - Lớp nghiệp vụ chatbot
- [chat_engine.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/core/chat_engine.py:0:0-0:0): **Trái tim** của hệ thống
  - Orchestrate toàn bộ flow: entity extraction → intent detection → RAG → LLM generation
  - Quản lý conversation history
  - Điều phối các services (weather, map, food, etc.)
  - Return response + metadata (rag_used, sources_count, intent)

- [entity_extractor.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/core/entity_extractor.py:0:0-0:0): Trích xuất thông tin có cấu trúc
  - Extract: city name, start_date, end_date từ natural language
  - Normalize city names (Hà Nội, Đà Nẵng, Sài Gòn...)
  - Check [is_travel_related()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/core/entity_extractor.py:165:4-202:32) để filter off-topic queries
  - AI-based extraction qua OpenAI API

- [intent_detector.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/core/intent_detector.py:0:0-0:0): Phát hiện ý định đơn giản
  - Detect: `weather_query`, `food_query`, `itinerary_request`
  - Semantic matching qua ChromaDB intent collection
  - Handle intent trực tiếp (không cần RAG nếu match)

#### **services/** - Lớp tích hợp dịch vụ bên ngoài

- [chroma_service.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/services/chroma_service.py:0:0-0:0) (620 lines): **Service phức tạp nhất**
  - Initialize ChromaDB client + 3 collections
  - [rag_query()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/services/chroma_service.py:109:4-174:25): Query tri thức du lịch, return (docs, context)
  - [rag_query_enhanced()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/services/chroma_service.py:365:4-467:25): RAG với city filtering
  - [add_to_memory()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/services/chroma_service.py:176:4-197:57) + [recall_memories()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/services/chroma_service.py:199:4-252:21): Long-term memory
  - [get_intent()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/services/chroma_service.py:254:4-284:19): Intent detection qua semantic search
  - Embedding generation: SentenceTransformer

- [langchain_service.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/services/langchain_service.py:0:0-0:0) (265 lines): **LangChain orchestration**
  - Initialize: ChatOpenAI LLM + HuggingFace embeddings + Chroma vectorstore
  - [create_rag_chain()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/services/langchain_service.py:107:4-155:23): Tạo ConversationalRetrievalChain
  - [generate_with_rag()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/services/langchain_service.py:157:4-216:74): RAG generation (ưu tiên path)
  - [_generate_direct_llm()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/services/langchain_service.py:218:4-251:13): Fallback khi không có docs
  - ConversationBufferWindowMemory (k=12 messages)

- [voice_service.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/services/voice_service.py:0:0-0:0): Voice I/O
  - [speech_to_text()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/services/voice_service.py:77:4-95:23): Audio bytes → text (Google Speech Recognition)
  - [text_to_speech()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/services/voice_service.py:97:4-108:23): Text → base64 audio (gTTS)
  - [convert_to_wav()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/services/voice_service.py:42:4-75:23): Convert WebM/OGG/MP3 → WAV (pydub/ffmpeg)

- [weather_service.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/services/weather_service.py:0:0-0:0): OpenWeatherMap integration
  - [get_forecast()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/services/weather_service.py:48:4-105:75): Lấy dự báo 5 ngày
  - AI fallback nếu không tìm thấy city

- [geocoding_service.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/services/geocoding_service.py:0:0-0:0): Location services
  - Geocode city → (lat, lon) qua Geopy
  - Render PyDeck 3D map

- [image_service.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/services/image_service.py:0:0-0:0): Pixabay API
  - `get_city_image()` + `get_food_images()`

- [food_service.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/services/food_service.py:0:0-0:0) + [restaurant_service.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/services/restaurant_service.py:0:0-0:0): Local data
  - Query CSV files
  - AI fallback nếu không có data

- [logger_service.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/services/logger_service.py:0:0-0:0): SQLite logging
  - Log interactions với metadata (city, dates, intent, rag_used)

#### **ui/** - Giao diện người dùng

- [app.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/ui/app.py:0:0-0:0) (953 lines): **Streamlit application**
  - Hero section với search form
  - Chat interface với message history
  - Sidebar: settings, voice toggle, display options
  - Analytics tab: Query statistics, top cities (Plotly charts)
  - Display: weather, map, images, foods, sources
  - Voice recording button (streamlit-mic-recorder)

#### **config/** - Cấu hình tập trung

- [settings.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/config/settings.py:0:0-0:0): Centralized configuration class
  - Load từ [.env](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/.env:0:0-0:0) hoặc Streamlit secrets
  - API keys: OPENAI, OPENWEATHERMAP, PIXABAY, PLACES
  - Paths: DATA_DIR, CHROMA_DIR, DB_PATH
  - Constants: RAG_TOP_K=5, INTENT_THRESHOLD=0.18, MEMORY_RECALL_K=3
  - **SYSTEM_PROMPT**: System prompt dài cho LLM (format 6 mục: Địa điểm, Thời gian, Ẩm thực, Chi phí, Góc chụp, Mẹo hay)

#### **utils/** - Utilities

- [text_processing.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/utils/text_processing.py:0:0-0:0): Extract số ngày, split foods
- [date_utils.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/utils/date_utils.py:0:0-0:0): Parse date ranges

#### **data/** - Dữ liệu

- [vietnam_travel_docs.csv](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/data/vietnam_travel_docs.csv:0:0-0:0): ~20 documents về các địa điểm du lịch (Hạ Long, Huế, Đà Nẵng, Sapa, Hà Nội, Nha Trang, Phong Nha, Đà Lạt, Mũi Né, Phú Quốc, Tây Nguyên, Củ Chi, Mekong, Ninh Bình, Hội An...)
- [vietnam_foods.csv](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/data/vietnam_foods.csv:0:0-0:0): 11 cities với món ăn đặc trưng
- Mỗi document có: id, title, location, content, source

#### **chromadb_data/** - Vector DB storage

- Persistent storage cho ChromaDB
- 3 collections:
  - `vietnam_travel_v2`: RAG knowledge base
  - `chat_memory_v2`: Conversation memory
  - `intent_bank_v2`: Intent patterns

### 3.3 So sánh ARCHITECTURE.md vs Code thực tế

**Khớp hoàn toàn:**
- ✅ Cấu trúc thư mục đúng như mô tả
- ✅ Flow diagram chính xác
- ✅ LangChain integration đã được implement
- ✅ Dual memory system hoạt động
- ✅ Intent detection qua ChromaDB

**Điểm khác biệt nhỏ:**
- Google Places API ít được sử dụng (chủ yếu dùng Geopy)
- Một số tính năng như restaurant service chưa tích hợp Google Places, vẫn dùng CSV fallback

---

## 4. LUỒNG XỬ LÝ CHÍNH (MAIN FLOW)

### 4.1 Entry Point

**File chạy chính:** [main.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/main.py:0:0-0:0)
```python
from ui.app import run_app
if __name__ == "__main__":
    run_app()
```

**Lệnh chạy:**
```bash
streamlit run main.py
```

### 4.2 Luồng tương tác từ User đến Response

```
┌─────────────────────────────────────────────────────────────────┐
│ STEP 1: USER INPUT (ui/app.py)                                 │
│ - Text input: st.chat_input()                                  │
│ - Voice input: mic_recorder() → audio bytes                    │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 2: VOICE PROCESSING (nếu có voice)                        │
│ - VoiceService.speech_to_text(audio_bytes)                    │
│ - Convert audio → WAV → Google Speech Recognition             │
│ - Output: text string                                          │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 3: ENTITY EXTRACTION (core/entity_extractor.py)          │
│ - EntityExtractor.extract_city_and_dates(user_text)           │
│ - Gọi OpenAI API để parse: {city, start_date, end_date}       │
│ - Validate: is_travel_related() → filter off-topic            │
│ - Normalize city names (Hà Nội, Đà Nẵng...)                   │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 4: INTENT DETECTION (core/intent_detector.py)            │
│ - IntentDetector.detect_intent(user_text)                     │
│ - Query ChromaDB intent_bank collection (semantic search)     │
│ - Threshold < 0.18 → intent matched                           │
│ - Intents: weather_query, food_query, itinerary_request       │
└────────────────────────────┬────────────────────────────────────┘
                             │
                    ┌────────┴────────┐
                    │                 │
            ┌───────▼────────┐   ┌────▼──────────────┐
            │ Intent Matched │   │ No Intent         │
            │ (weather/food) │   │ → Need RAG        │
            └───────┬────────┘   └────┬──────────────┘
                    │                 │
                    │                 ▼
                    │         ┌──────────────────────┐
                    │         │ STEP 5: RAG QUERY    │
                    │         │ (Priority: LangChain)│
                    │         └────┬─────────────────┘
                    │              │
                    │              ▼
                    │      ┌────────────────────┐
                    │      │ LangChain Available?│
                    │      └────┬───────────────┘
                    │           │
                    │      ┌────┴─────┐
                    │      │          │
                    │  ┌───▼───┐  ┌───▼──────────┐
                    │  │ YES   │  │ NO           │
                    │  │       │  │ Traditional  │
                    │  └───┬───┘  │ RAG          │
                    │      │      └───┬──────────┘
                    │      │          │
                    │      └──────┬───┘
                    │             │
                    │             ▼
                    │    ┌─────────────────────────┐
                    │    │ Query ChromaDB          │
                    │    │ vietnam_travel_v2       │
                    │    │ (top-k=5 documents)     │
                    │    └────┬────────────────────┘
                    │         │
                    │         ▼
                    │    ┌─────────────────────────┐
                    │    │ Documents Found?        │
                    │    │ (sources_count > 0?)    │
                    │    └────┬────────────────────┘
                    │         │
                    │    ┌────┴─────┐
                    │    │          │
                    │ ┌──▼───┐  ┌───▼─────────┐
                    │ │ YES  │  │ NO          │
                    │ │      │  │ Direct LLM  │
                    │ └──┬───┘  │ (no RAG)    │
                    │    │      └───┬─────────┘
                    │    │          │
                    └────┴──────────┴──────────┐
                                               │
                                               ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 6: MEMORY RECALL (services/chroma_service.py)            │
│ - ChromaService.recall_memories(user_text, k=3)               │
│ - Query chat_memory_v2 collection                             │
│ - Return similar past conversations                            │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 7: LLM GENERATION (core/chat_engine.py)                  │
│                                                                 │
│ IF LangChain RAG Chain:                                        │
│   - ConversationalRetrievalChain.invoke()                     │
│   - Auto retrieve + generate với memory                       │
│                                                                 │
│ IF Traditional RAG:                                            │
│   - Build context: RAG docs + memory recall                    │
│   - Augment system prompt với context                         │
│   - Call OpenAI API: chat.completions.create()               │
│   - Temperature=0.7, max_tokens=900                            │
│                                                                 │
│ IF No docs (Direct LLM):                                       │
│   - System prompt + memory only (no RAG augmentation)         │
│   - LLM dựa vào internal knowledge                            │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 8: MEMORY STORAGE                                         │
│ - LangChain memory: Auto-updated by chain                     │
│ - ChromaDB memory: add_to_memory(user_msg + assistant_msg)   │
│ - Metadata: role, city, timestamp                             │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 9: LOGGING (services/logger_service.py)                  │
│ - Log to SQLite: travel_chatbot_logs.db                       │
│ - Fields: timestamp, user_input, city, dates, intent,         │
│           rag_used, sources_count                              │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 10: ADDITIONAL SERVICES (chat_engine._call_additional_   │
│          services)                                              │
│ - WeatherService.get_forecast() → 5-day forecast             │
│ - GeocodingService.geocode_city() → (lat, lon) + map         │
│ - ImageService.get_city_image() → Pixabay images             │
│ - FoodService.get_foods_with_fallback() → món ăn đặc sản     │
│ - RestaurantService.get_restaurants() → nhà hàng              │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ STEP 11: UI DISPLAY (ui/app.py)                               │
│ - Render chat messages (user + assistant)                     │
│ - Display RAG sources (expandable với title + snippet)       │
│ - Show weather forecast                                        │
│ - Show PyDeck 3D map                                           │
│ - Display city/food images                                     │
│ - List foods + restaurants                                     │
│ - TTS playback (nếu enabled): VoiceService.text_to_speech()  │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
                 ┌───────────────────────┐
                 │ USER SEES RESPONSE    │
                 └───────────────────────┘
```

### 4.3 RAG Pipeline Chi Tiết

**Bước Ingest Data (One-time setup):**
```python
# Không thấy script ingest rõ ràng, nhưng dựa vào code:
# 1. Load vietnam_travel_docs.csv
# 2. Generate embeddings cho mỗi document (SentenceTransformer)
# 3. Add vào ChromaDB collection "vietnam_travel_v2"
```

**Bước Retrieve (Query time):**
```python
# services/chroma_service.py: rag_query()
1. User query → embedding (SentenceTransformer)
2. Query ChromaDB.vietnam_travel_v2 với cosine similarity
3. Return top-k=5 documents với metadata + distances
4. Format thành context string: [src:ID|source] text...
```

**Bước Generate:**
```python
# LangChain path:
ConversationalRetrievalChain tự động:
1. Retrieve documents từ vectorstore
2. Build prompt với context + chat_history
3. Generate response qua ChatOpenAI
4. Return answer + source_documents

# Traditional path:
1. Augment system prompt: SYSTEM_PROMPT + RAG context + memory
2. Build messages: [system, ...history, user]
3. Call OpenAI API
4. Return response
```

---

## 5. CHI TIẾT CÁC MODULE QUAN TRỌNG

### 5.1 core/chat_engine.py - Orchestrator chính

**Class:** [ChatEngine](cci:2://file:///d:/CB/Travel_Chat_Bot_AI/core/chat_engine.py:17:0-345:9)

**Input:** `user_input: str, conversation_history: List[Dict]`

**Output:** 
```python
{
    "response": str,              # Câu trả lời của chatbot
    "intent": Optional[str],      # Intent detected
    "rag_used": bool,             # Có dùng RAG không?
    "sources_count": int,         # Số documents retrieved
    "memory_used": bool,          # Có recall memory không?
    "city": Optional[str],        # City extracted
    "start_date": datetime,       # Ngày bắt đầu
    "end_date": datetime,         # Ngày kết thúc
    "rag_docs": List[Dict],       # RAG documents
    "additional_services": Dict   # Weather, map, images, foods...
}
```

**Trách nhiệm:**
1. Check [is_travel_related()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/core/entity_extractor.py:165:4-202:32) → reject nếu off-topic
2. Extract entities: city, dates
3. Detect intent → handle trực tiếp nếu match
4. Nếu không có intent → RAG + LLM generation
5. Memory recall + storage
6. Logging
7. Điều phối additional services (weather, map, food...)
8. Return comprehensive result

**Business Logic chính:**
- Format response theo 6 mục (Địa điểm, Thời gian, Ẩm thực, Chi phí, Góc chụp, Mẹo hay) khi tư vấn lịch trình
- Augmentation context từ RAG + memory
- Fallback mechanism: LangChain → Traditional RAG → Direct LLM

### 5.2 services/chroma_service.py - Vector DB Operations

**Class:** [ChromaService](cci:2://file:///d:/CB/Travel_Chat_Bot_AI/services/chroma_service.py:17:0-617:20)

**Key Methods:**

```python
_initialize():
    # Setup ChromaDB client, load embedding model, create collections

get_embedding(text: str) → List[float]:
    # Generate 384-dim embedding qua SentenceTransformer

rag_query(user_text: str, k=5) → (List[Dict], str):
    # Query vietnam_travel_v2, return docs + context string

rag_query_enhanced(user_text, k=5, target_city=None):
    # RAG với city filtering (nếu có target_city)

add_to_memory(text, role, city, extra_meta):
    # Lưu conversation vào chat_memory_v2

recall_memories(user_text, k=3) → List[Dict]:
    # Semantic search trên chat_memory_v2

get_intent(user_text, threshold=0.18) → Optional[str]:
    # Query intent_bank_v2, return intent nếu distance < threshold

_preload_intents():
    # Preload intent patterns vào intent_bank_v2
```

**Business Logic:**
- **3 collections riêng biệt** cho RAG, memory, intent
- Embedding model được cache local để tránh download lại
- Safe collection handling với dimension validation
- Memory có metadata: role, city, timestamp

### 5.3 services/langchain_service.py - LangChain Orchestration

**Class:** [LangChainService](cci:2://file:///d:/CB/Travel_Chat_Bot_AI/services/langchain_service.py:21:0-262:43)

**Key Methods:**

```python
_initialize():
    # Setup ChatOpenAI, HuggingFaceEmbeddings, Chroma vectorstore
    # Initialize ConversationBufferWindowMemory (k=12)

create_rag_chain() → ConversationalRetrievalChain:
    # Tạo RAG chain với custom prompt
    # Retriever: vectorstore.as_retriever(k=5)
    # Return source documents

generate_with_rag(user_input, conversation_history) → Dict:
    # Main generation method
    # Try RAG chain → fallback to direct LLM
    # Check documents found → set rag_used flag

_generate_direct_llm(user_input, conversation_history) → Dict:
    # Direct LLM call (no RAG augmentation)
    # Only memory + LLM internal knowledge

add_to_memory(user_input, assistant_response):
    # Add to LangChain memory
```

**Business Logic:**
- **Priority system**: RAG chain (có docs) → Direct LLM (no docs)
- Custom system prompt với format reminder
- Chat history conversion: Dict → LangChain Messages
- Source documents formatting

### 5.

# BÁO CÁO PHÂN TÍCH CODEBASE - TRAVEL CHAT BOT AI

## 1. TỔNG QUAN HỆ THỐNG

### Loại chatbot
**RAG-powered Travel Assistant** chuyên du lịch Việt Nam với:
- Chatbot tư vấn lịch trình, địa điểm, ẩm thực, thời tiết
- RAG (ChromaDB + LangChain) truy xuất tri thức từ knowledge base
- Multi-modal: Text + Voice (STT/TTS)
- Dual memory: LangChain + ChromaDB

### Dạng ứng dụng
- **Streamlit Web App** (browser-based)
- Entry point: [main.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/main.py:0:0-0:0) → `streamlit run main.py`
- URL: http://localhost:8501

### Điểm đặc biệt
✅ Kiến trúc modular rõ ràng (core/services/ui/config/utils)  
✅ Dual RAG: LangChain (ưu tiên) + Traditional (fallback)  
✅ Dual Memory: ConversationBufferWindowMemory + ChromaDB  
✅ Intent Detection qua semantic search  
✅ Multi-service: Weather, Geocoding, Images, Food  
✅ Voice-enabled, Analytics dashboard

---

## 2. TECH STACK

### AI/LLM
- **OpenAI API** (gpt-4o-mini): Chat, entity extraction, generation
- **LangChain**: ConversationalRetrievalChain, memory management
- **Sentence Transformers**: all-MiniLM-L6-v2 (384-dim embeddings)

### Vector DB / RAG
- **ChromaDB 1.2.1**: 3 collections (travel_docs, memory, intent_bank)
- Persistent storage tại [chromadb_data/](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/chromadb_data:0:0-0:0)

### Web & UI
- **Streamlit 1.50.0**: Main framework
- **Plotly**: Analytics charts
- **PyDeck**: Interactive 3D maps

### External APIs
- **OpenWeatherMap**: Dự báo thời tiết 5 ngày
- **Pixabay**: Hình ảnh city/food
- **Geopy**: Geocoding

### Voice
- **SpeechRecognition**: Google STT
- **gTTS**: Google TTS
- **Pydub + FFmpeg**: Audio conversion

### Storage
- **SQLite**: Logging ([travel_chatbot_logs.db](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/travel_chatbot_logs.db:0:0-0:0))
- **CSV**: Foods, restaurants, travel docs

---

## 3. CẤU TRÚC THƯ MỤC

```
Travel_Chat_Bot_AI/
├── core/                      # Business logic
│   ├── chat_engine.py         # Orchestrator chính
│   ├── entity_extractor.py    # Extract city, dates
│   └── intent_detector.py     # Detect intent
│
├── services/                  # External integrations
│   ├── chroma_service.py      # ChromaDB (RAG, memory, intent)
│   ├── langchain_service.py   # LangChain RAG + memory
│   ├── voice_service.py       # STT/TTS
│   ├── logger_service.py      # SQLite logging
│   ├── weather_service.py     # OpenWeatherMap
│   ├── geocoding_service.py   # Geopy + PyDeck
│   ├── image_service.py       # Pixabay
│   ├── food_service.py        # Food data
│   └── restaurant_service.py  # Restaurant data
│
├── ui/                        # Presentation
│   └── app.py                 # Streamlit UI (953 lines)
│
├── config/
│   └── settings.py            # Config + env vars
│
├── utils/
│   ├── text_processing.py
│   └── date_utils.py
│
├── data/
│   ├── vietnam_travel_docs.csv    # ~20 docs RAG knowledge
│   ├── vietnam_foods.csv          # 11 cities
│   └── all-MiniLM-L6-v2/          # Local embedding model
│
├── chromadb_data/             # Vector DB storage
├── main.py                    # Entry point
├── requirements.txt           # 265 lines dependencies
└── .env                       # API keys
```

**Vai trò:**
- **core/**: Nghiệp vụ chatbot (orchestration, entity, intent)
- **services/**: Tích hợp APIs bên ngoài
- **ui/**: Giao diện Streamlit
- **config/**: Cấu hình tập trung
- **data/**: Knowledge base + CSV data

---

## 4. LUỒNG XỬ LÝ CHÍNH

```
User Input (Text/Voice)
    ↓
Voice Processing (STT nếu có) → Text
    ↓
Entity Extraction (city, dates) → OpenAI API
    ↓
is_travel_related? → Reject nếu off-topic
    ↓
Intent Detection → ChromaDB semantic search
    ↓
    ├─→ Intent Match (weather/food) → Handle trực tiếp
    └─→ No Intent → RAG + LLM
            ↓
        LangChain Available?
            ├─→ YES: ConversationalRetrievalChain
            │        ├─→ Docs found → RAG generation
            │        └─→ No docs → Direct LLM
            └─→ NO: Traditional RAG
                    ├─→ Query ChromaDB (top-k=5)
                    ├─→ Recall memory (k=3)
                    ├─→ Build context (RAG + memory)
                    └─→ OpenAI API generation
    ↓
Memory Storage (LangChain + ChromaDB)
    ↓
Logging → SQLite
    ↓
Additional Services (Weather, Map, Images, Food)
    ↓
UI Display (Chat, Sources, Weather, Map, Images)
    ↓
TTS (nếu enabled)
```

---

## 5. MODULE QUAN TRỌNG

### [core/chat_engine.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/core/chat_engine.py:0:0-0:0) - Orchestrator
- **Input**: `user_input, conversation_history`
- **Output**: `{response, intent, rag_used, sources_count, city, dates, rag_docs, additional_services}`
- **Logic**: 
  - Check travel-related
  - Extract entities → Detect intent
  - RAG (LangChain priority) → LLM generation
  - Điều phối additional services
  - Format theo 6 mục (Địa điểm, Thời gian, Ẩm thực, Chi phí, Góc chụp, Mẹo)

### [services/chroma_service.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/services/chroma_service.py:0:0-0:0) - Vector DB
- **3 collections**: `vietnam_travel_v2`, `chat_memory_v2`, `intent_bank_v2`
- **Methods**: [rag_query()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/services/chroma_service.py:109:4-174:25), [add_to_memory()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/services/chroma_service.py:176:4-197:57), [recall_memories()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/services/chroma_service.py:199:4-252:21), [get_intent()](cci:1://file:///d:/CB/Travel_Chat_Bot_AI/services/chroma_service.py:254:4-284:19)
- **Embedding**: SentenceTransformer (384-dim)

### [services/langchain_service.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/services/langchain_service.py:0:0-0:0) - LangChain
- **ConversationalRetrievalChain**: Auto RAG + memory
- **ConversationBufferWindowMemory**: k=12 messages
- **Fallback**: Direct LLM nếu no docs

### [ui/app.py](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/ui/app.py:0:0-0:0) - Streamlit UI
- Hero section + search form
- Chat interface
- Sidebar: Voice toggle, display options
- Analytics tab: Query stats, top cities
- Display: Sources, weather, map, images, foods

---

## 6. CẤU HÌNH & MÔI TRƯỜNG

### Biến môi trường (.env)
```
OPENAI_API_KEY=           # OpenAI API key
OPENAI_ENDPOINT=          # Custom endpoint
DEPLOYMENT_NAME=          # Model name (gpt-4o-mini)
OPENWEATHERMAP_API_KEY=   # Weather API
PIXABAY_API_KEY=          # Image API
PLACES_API_KEY=           # Google Places (optional)
CHROMA_PERSIST_DIR=       # ChromaDB storage path
```

### Settings quan trọng
- `RAG_TOP_K = 5`: Số documents retrieve
- `INTENT_THRESHOLD = 0.18`: Ngưỡng detect intent
- `MEMORY_RECALL_K = 3`: Số memories recall
- `SYSTEM_PROMPT`: Dài, định nghĩa format 6 mục

---

## 7. CÁCH CHẠY PROJECT

### Bước 1: Setup môi trường
```bash
python -m venv venv
venv\Scripts\activate  # Windows
pip install -r requirements.txt
```

### Bước 2: Cấu hình .env
Tạo file [.env](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/.env:0:0-0:0) với các API keys cần thiết (xem section 6)

### Bước 3: Chạy app
```bash
streamlit run main.py
```
→ Mở browser tại http://localhost:8501

### Khởi tạo dữ liệu
- ChromaDB tự động init khi chạy lần đầu
- [vietnam_travel_docs.csv](cci:7://file:///d:/CB/Travel_Chat_Bot_AI/data/vietnam_travel_docs.csv:0:0-0:0) được load vào collection `vietnam_travel_v2`
- Embedding model download từ HuggingFace (lần đầu) và cache local

---

## 8. ĐÁNH GIÁ CHẤT LƯỢNG CODE

### Điểm mạnh ⭐
✅ **Separation of concerns xuất sắc**: core/services/ui rõ ràng  
✅ **Modular**: Dễ test, maintain, extend  
✅ **Production-ready**: Error handling, logging, fallback  
✅ **Documentation tốt**: README, ARCHITECTURE.md chi tiết  
✅ **Modern stack**: LangChain, ChromaDB, Streamlit  
✅ **Dual system**: RAG, Memory có backup  

### Điểm yếu ⚠️
⚠️ **Thiếu type hints** ở một số chỗ  
⚠️ **Không có unit tests** (chưa thấy folder tests/)  
⚠️ **Hard-coded values** (threshold, k values trong code)  
⚠️ **Data ingest không tự động**: Phải load manual  
⚠️ **Error messages tiếng Việt**: Khó debug cho dev quốc tế  

---

## 9. GỢI Ý CẢI TIẾN

### 🔥 Ưu tiên cao (Dễ làm trước)

**1. Thêm Type Hints toàn bộ**
```python
# Trước
def rag_query(self, user_text, k=5):

# Sau  
def rag_query(self, user_text: str, k: int = 5) -> Tuple[List[Dict], str]:
```

**2. Tách config constants**
```python
# config/constants.py
class RAGConfig:
    TOP_K = 5
    INTENT_THRESHOLD = 0.18
    MEMORY_RECALL_K = 3
```

**3. Thêm error logging đầy đủ**
```python
import logging
logger = logging.getLogger(__name__)
logger.error(f"RAG query failed: {e}", exc_info=True)
```

**4. Viết ingest script**
```python
# scripts/ingest_data.py
def ingest_travel_docs():
    # Load CSV → Generate embeddings → Add to ChromaDB
```

**5. Thêm .env.example**
```bash
cp .env .env.example
# Remove sensitive values từ .env.example
```

### 🎯 Ưu tiên trung bình

**6. Cải thiện RAG quality**
- Chunking strategy tốt hơn (hiện tại load toàn bộ doc)
- Hybrid search (semantic + keyword)
- Reranking documents

**7. Prompt engineering**
- Few-shot examples trong system prompt
- Chain-of-thought prompting
- Output parser cho structured response

**8. Caching**
```python
@st.cache_data(ttl=3600)
def get_weather_forecast(city, date):
    # Cache weather API calls
```

**9. User feedback loop**
- Thumbs up/down cho responses
- Lưu feedback → Improve RAG/prompts

### 🚀 Tính năng mới

**10. FastAPI Backend**
- Tách logic ra REST API
- Streamlit → Frontend only
- Scalable deployment

**11. User authentication**
- Streamlit auth hoặc OAuth
- Personal memory per user
- Saved itineraries

**12. Advanced analytics**
- User behavior tracking
- Popular destinations
- Response quality metrics

**13. Multi-language**
- English interface
- Auto-detect language
- Translate responses

**14. Testing suite**
```
tests/
├── test_chat_engine.py
├── test_chroma_service.py
├── test_entity_extractor.py
└── test_langchain_service.py
```

**15. CI/CD Pipeline**
```yaml
# .github/workflows/test.yml
- pytest
- linting (ruff/black)
- type checking (mypy)
```

---

## 10. TÓM TẮT

**Project này là một RAG chatbot được xây dựng RẤT TỐT với:**
- Kiến trúc modular, separation of concerns xuất sắc
- Tech stack hiện đại (LangChain, ChromaDB, Streamlit)
- Production-ready với error handling, logging, fallback
- Documentation đầy đủ

**Cần cải thiện:**
- Type hints, unit tests
- Config management tốt hơn
- RAG quality (chunking, hybrid search)
- Scalability (FastAPI backend, user auth)

**Phù hợp cho:** Production deployment sau khi thêm tests và optimize RAG quality.