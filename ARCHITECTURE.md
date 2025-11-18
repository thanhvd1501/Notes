# Travel Chat Bot AI - Modular Architecture Documentation

## 📁 Project Structure

```
TRAVEL_CHAT_BOT_AI/
│
├── core/                          # Core chatbot logic
│   ├── __init__.py
│   ├── chat_engine.py            # Main conversation orchestration
│   ├── intent_detector.py        # Intent detection using ChromaDB
│   └── entity_extractor.py       # Extract city, dates from text
│
├── services/                      # External services & integrations
│   ├── __init__.py
│   ├── chroma_service.py         # ChromaDB RAG operations
│   ├── langchain_service.py      # LangChain RAG chains & memory
│   ├── voice_service.py          # Speech-to-Text & Text-to-Speech
│   ├── logger_service.py         # SQLite logging
│   ├── weather_service.py        # OpenWeatherMap API
│   ├── geocoding_service.py      # Location lookup & maps
│   ├── image_service.py          # Pixabay image API
│   ├── food_service.py           # Food recommendations
│   └── restaurant_service.py     # Restaurant recommendations
│
├── ui/                            # Streamlit user interface
│   ├── __init__.py
│   └── app.py                    # Main UI application
│
├── config/                        # Configuration management
│   ├── __init__.py
│   └── settings.py               # Environment variables & constants
│
├── utils/                         # Utility functions
│   ├── __init__.py
│   ├── text_processing.py        # Text parsing utilities
│   └── date_utils.py             # Date handling utilities
│
├── data/                          # Data files (CSV, etc.)
│   ├── vietnam_foods.csv
│   ├── restaurants_vn.csv
│   └── vietnam_travel_docs.csv
│
├── chromadb_data/                 # ChromaDB persistent storage
│
├── main.py                        # Application entry point
├── requirements.txt               # Python dependencies
├── travel_chatbot_logs.db        # SQLite database
└── README.md                      # Project documentation
```

## 🧩 Module Responsibilities

### **core/** - Core Chatbot Logic

#### `chat_engine.py`
- **Purpose**: Main orchestration of conversation flow
- **Responsibilities**:
  - Process user messages
  - Coordinate intent detection, RAG retrieval, and LLM generation
  - Prioritize LangChain RAG chain if available, fallback to traditional RAG
  - Manage conversation context and memory (both LangChain and ChromaDB)
  - Log interactions

#### `intent_detector.py`
- **Purpose**: Detect user intent from text
- **Responsibilities**:
  - Use ChromaDB semantic matching to detect intents
  - Handle intent-specific responses (weather, food, itinerary)
  - Fallback to RAG when intent not detected

#### `entity_extractor.py`
- **Purpose**: Extract structured information from user text
- **Responsibilities**:
  - Extract city names, dates from natural language
  - Validate travel-related topics
  - Resolve ambiguous location names via AI

### **services/** - External Services

#### `chroma_service.py`
- **Purpose**: ChromaDB vector database operations
- **Responsibilities**:
  - Initialize ChromaDB client and collections
  - Generate embeddings using SentenceTransformers
  - RAG query: retrieve relevant documents
  - Memory management: store/recall conversations
  - Intent bank: semantic intent matching

#### `langchain_service.py`
- **Purpose**: LangChain integration for RAG and memory management
- **Responsibilities**:
  - Initialize LangChain components (ChatOpenAI, Chroma vectorstore)
  - Create ConversationalRetrievalChain for RAG
  - Manage ConversationBufferWindowMemory
  - Generate responses with RAG using LangChain chains
  - Fallback to traditional RAG if LangChain unavailable

#### `voice_service.py`
- **Purpose**: Voice input/output processing
- **Responsibilities**:
  - Speech-to-Text: convert audio to text (Google Speech Recognition)
  - Text-to-Speech: convert text to audio (gTTS)
  - Audio format conversion (WAV, OGG, WebM, MP3)

#### `logger_service.py`
- **Purpose**: Interaction logging
- **Responsibilities**:
  - Log user interactions to SQLite
  - Track RAG usage, intent detection, sources
  - Provide analytics data

#### `weather_service.py`
- **Purpose**: Weather forecast data
- **Responsibilities**:
  - Fetch weather data from OpenWeatherMap API
  - Format forecast for display
  - Handle date ranges

#### `geocoding_service.py`
- **Purpose**: Location services
- **Responsibilities**:
  - Geocode city names to coordinates
  - Display interactive maps using PyDeck

#### `image_service.py`
- **Purpose**: Image retrieval
- **Responsibilities**:
  - Fetch images from Pixabay API
  - Get city and food images

#### `food_service.py` & `restaurant_service.py`
- **Purpose**: Food and restaurant recommendations
- **Responsibilities**:
  - Query CSV data for local foods/restaurants
  - Fallback to GPT when CSV data unavailable

### **config/** - Configuration

#### `settings.py`
- **Purpose**: Centralized configuration management
- **Responsibilities**:
  - Load environment variables
  - Provide default values
  - Support Streamlit secrets integration

### **ui/** - User Interface

#### `app.py`
- **Purpose**: Streamlit application interface
- **Responsibilities**:
  - Render UI components (hero, sidebar, chat, analytics)
  - Handle user interactions
  - Coordinate services and core modules
  - Display results and analytics

### **utils/** - Utilities

#### `text_processing.py`
- **Purpose**: Text parsing utilities
- **Functions**: Extract days, split foods, clean text

#### `date_utils.py`
- **Purpose**: Date handling
- **Functions**: Parse date ranges, validate dates

## 🔄 System Flow

### **Complete Flow Diagram**

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER INPUT                               │
│              (Text Input or Voice Recording)                      │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    VOICE SERVICE (if voice)                      │
│  • Convert audio → WAV                                          │
│  • Speech-to-Text (Google Speech Recognition)                   │
│  • Output: Text string                                          │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ENTITY EXTRACTOR                              │
│  • Extract city name                                            │
│  • Extract start_date, end_date                                  │
│  • Validate travel-related topic                                 │
│  • Output: {city, start_date, end_date, is_travel_related}      │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
                    ┌───────┴───────┐
                    │               │
            ┌───────▼───────┐   ┌───▼──────────────┐
            │  INTENT      │   │  RAG QUERY        │
            │  DETECTOR    │   │  (ChromaDB)       │
            └───────┬───────┘   └───┬──────────────┘
                    │               │
                    ▼               ▼
            ┌───────────────────────────────┐
            │   INTENT MATCHED?             │
            │   (weather_query,             │
            │    food_query, etc.)          │
            └───────┬───────────────┬───────┘
                    │               │
            ┌───────▼───────┐   ┌───▼──────────────┐
            │  YES          │   │  NO              │
            │  Handle       │   │  Need RAG?       │
            │  Intent       │   │  (Yes, proceed)  │
            │  Directly     │   │                  │
            └───────┬───────┘   └───┬──────────────┘
                    │               │
                    │               │ (Only 1 RAG process,
                    │               │  choose implementation)
                    │               │
                    │               ▼
                    │       ┌───────────────┐
                    │       │  LangChain    │
                    │       │  Available?   │
                    │       └───────┬───────┘
                    │               │
                    │       ┌───────┴───────┐
                    │       │               │
                    │   ┌───▼───────┐   ┌───▼──────────────┐
                    │   │ YES       │   │ NO               │
                    │   │ LangChain │   │ Traditional RAG  │
                    │   │ RAG Chain │   │ (manual retrieve │
                    │   │           │   │  + LLM call)     │
                    │   └───┬───────┘   └───┬──────────────┘
                    │       │               │
                    │       └───────┬───────┘
                    │               │
                    │               │ (Both paths do RAG,
                    │               │  just different ways)
                    │               │
                    └───────────────┴───────┘
                                    │
                                    ▼
                    ┌───────────────────────────────┐
                    │   RAG RETRIEVAL               │
                    │   (Query ChromaDB)            │
                    └───────┬───────────────────────┘
                            │
                            ▼
                    ┌───────────────────────────────┐
                    │   DOCUMENTS FOUND?            │
                    │   (sources_count > 0?)        │
                    └───────┬───────────────┬───────┘
                            │               │
                    ┌───────▼───────┐   ┌───▼──────────────┐
                    │  YES          │   │  NO              │
                    │  (Has docs)   │   │  (No docs found) │
                    │               │   │                  │
                    │  Build RAG    │   │  Fallback to     │
                    │  context +    │   │  Direct LLM      │
                    │  Memory       │   │  (no RAG aug)    │
                    │  recall       │   │  + Memory only   │
                    └───────┬───────┘   └───┬──────────────┘
                            │               │
                            └───────┬───────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CHAT ENGINE                                   │
│  • If RAG docs found: Build context (RAG + memory)             │
│  • If no RAG docs: Use direct LLM (memory only, no RAG aug)   │
│  • Generate prompt with/without augmentation                    │
│  • Call OpenAI LLM (via LangChain or direct)                   │
│  • Format response                                              │
│  • Output: {response, intent, rag_used, sources_count}         │
│    - rag_used = True if sources_count > 0                      │
│    - rag_used = False if no documents found                    │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    MEMORY STORAGE                                │
│  • Save user message to ChromaDB memory collection              │
│  • Save assistant response to memory                            │
│  • Store metadata (city, timestamp, role)                       │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    LOGGER SERVICE                                │
│  • Log interaction to SQLite                                    │
│  • Track: user_input, city, dates, intent, RAG usage            │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ADDITIONAL SERVICES                           │
│  • Weather Service: Fetch forecast                              │
│  • Geocoding Service: Get coordinates & show map               │
│  • Image Service: Fetch city/food images                        │
│  • Food Service: Get local foods                               │
│  • Restaurant Service: Get restaurant recommendations          │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    UI RENDERING                                  │
│  • Display chat messages                                        │
│  • Show sources (RAG documents)                                │
│  • Display weather, map, images, foods                          │
│  • Text-to-Speech (if enabled)                                 │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    USER SEES RESPONSE                           │
└─────────────────────────────────────────────────────────────────┘
```

> **💡 Lưu ý về RAG trong flow:**
> 
> Không phải có "nhiều RAG" chạy song song. Flow chỉ có **1 quyết định RAG**:
> - Nếu **có intent matched** (weather, food, etc.) → Xử lý trực tiếp, **KHÔNG dùng RAG**
> - Nếu **không có intent** → **CẦN RAG** để tìm thông tin từ knowledge base
> 
> Sau khi quyết định cần RAG, hệ thống chọn **cách triển khai**:
> - **LangChain RAG Chain** (ưu tiên): Sử dụng ConversationalRetrievalChain tự động
> - **Traditional RAG** (fallback): Retrieve documents thủ công + gọi LLM trực tiếp
> 
> Cả hai cách đều làm RAG, chỉ khác nhau về **cách thức triển khai**.
> 
> **⚠️ Trường hợp không tìm thấy thông tin:**
> - Sau khi RAG retrieval, nếu **không tìm thấy documents** (sources_count = 0):
>   - Hệ thống **fallback về Direct LLM** (không có RAG augmentation)
>   - Chỉ dùng **memory recall** (nếu có) + **LLM knowledge** để trả lời
>   - **KHÔNG có web search** hay external document retrieval
>   - LLM sẽ dựa vào kiến thức có sẵn của nó để trả lời câu hỏi

### **Detailed Step-by-Step Flow**

1. **User Input**
   - User types text OR records voice
   - If voice: `VoiceService.speech_to_text()` converts audio → text

2. **Entity Extraction**
   - `EntityExtractor.extract_city_and_dates()` parses text
   - Extracts: city name, start_date, end_date
   - Validates: `is_travel_related()` checks if query is travel-related

3. **Intent Detection**
   - `IntentDetector.detect_intent()` queries ChromaDB intent collection
   - If intent matched (distance < threshold):
     - Handle directly (weather_query → WeatherService, food_query → FoodService)
   - If no intent matched:
     - Proceed to RAG + LLM generation

4. **RAG Retrieval & Generation**
   - **If LangChain available**:
     - `LangChainService.generate_with_rag()` uses ConversationalRetrievalChain
     - Chain automatically retrieves documents and generates response
     - Uses ConversationBufferWindowMemory for conversation context
     - **If no documents found**: Falls back to `_generate_direct_llm()` (no RAG augmentation)
   - **If LangChain unavailable** (fallback):
     - `ChromaService.rag_query()` generates embedding for user text
     - Queries ChromaDB travel knowledge base
     - Returns top-k relevant documents with metadata
     - **If no documents found** (sources_count = 0): Proceeds to direct LLM call

5. **Check Documents Found**
   - **If documents found** (sources_count > 0):
     - `rag_used = True`
     - Build RAG context augmentation with retrieved documents
   - **If no documents found** (sources_count = 0):
     - `rag_used = False`
     - Skip RAG context augmentation
     - Fallback to direct LLM call (no RAG augmentation)
     - **Note**: Currently NO web search or external document retrieval

6. **Memory Recall**
   - **LangChain Memory**: ConversationBufferWindowMemory maintains last 12 messages
   - **ChromaDB Memory**: `ChromaService.recall_memories()` finds similar past conversations
   - Both memory systems work together for comprehensive context
   - Memory recall works regardless of RAG results

7. **LLM Generation**
   - **If RAG docs found**:
     - **LangChain path**: Chain handles prompt building with RAG context automatically
     - **Traditional path**: `ChatEngine.process_message()` builds augmented prompt:
       - System prompt + **RAG context** + memory recall
       - Calls OpenAI API with conversation history
   - **If no RAG docs found**:
     - **Direct LLM call**: System prompt + **memory recall only** (no RAG augmentation)
     - LLM relies on its own knowledge + conversation memory
   - Generates response

8. **Memory Storage**
   - **LangChain Memory**: Automatically updated by ConversationalRetrievalChain
   - **ChromaDB Memory**: Save user message and assistant response to ChromaDB memory collection
   - Both memory systems store conversation for future context recall

9. **Logging**
   - `LoggerService.log_interaction()` saves to SQLite
   - Tracks: timestamp, user_input, city, dates, intent, RAG usage, sources

10. **Additional Services**
   - Weather: Fetch forecast for city/date range
   - Geocoding: Get coordinates and display map
   - Images: Fetch city and food images
   - Food/Restaurants: Get recommendations

10. **UI Display**
    - Render chat messages
    - Show RAG sources
    - Display weather, map, images, foods
    - Optional TTS audio playback

## 🚀 Usage

### **Run Application**
```bash
streamlit run main.py
```

### **Environment Variables**
Create `.env` file or use Streamlit secrets:
```env
OPENAI_API_KEY=your_key
OPENWEATHERMAP_API_KEY=your_key
PIXABAY_API_KEY=your_key
PLACES_API_KEY=your_key
```

### **Key Features**
- ✅ Modular architecture (easy to extend)
- ✅ Clean separation of concerns
- ✅ RAG with ChromaDB + LangChain integration
- ✅ LangChain ConversationalRetrievalChain for enhanced RAG
- ✅ Dual memory system (LangChain + ChromaDB)
- ✅ Voice input/output
- ✅ Intent detection
- ✅ Memory management
- ✅ Analytics dashboard

## 📊 Benefits of Modular Architecture

1. **Maintainability**: Each module has a single responsibility
2. **Testability**: Services can be tested independently
3. **Extensibility**: Easy to add new features (e.g., FastAPI backend)
4. **Scalability**: Services can be deployed separately
5. **Reusability**: Services can be used in other projects

## 🔧 Future Enhancements

- FastAPI REST API backend
- WebSocket for real-time chat
- User authentication & profiles
- Multi-language support
- Advanced analytics dashboard
- Docker containerization
- CI/CD pipeline

