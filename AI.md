# 🧠 TOÀN TẬP: KIẾN TRÚC HỆ THỐNG AI & OPENAI API

---

## ⚙️ **CHƯƠNG 1 – NỀN TẢNG VỀ HỆ THỐNG AI**

---

### 1.1. Khái niệm cơ bản

#### 🤖 Trí tuệ nhân tạo (AI)

Là **khả năng của máy tính mô phỏng hành vi thông minh của con người**: hiểu, suy luận, học hỏi, và hành động.

**Phân loại chính:**

| Loại AI      | Đặc điểm                      | Ví dụ           |
| ------------ | ----------------------------- | --------------- |
| *Narrow AI*  | Chỉ làm 1 việc cụ thể         | ChatGPT, Siri   |
| *General AI* | Có tư duy tổng quát như người | (chưa đạt được) |
| *Super AI*   | Vượt trí tuệ con người        | (giả thuyết)    |

---

### 1.2. Các thành phần chính trong một hệ thống AI hiện đại

1. **Dữ liệu (Data Layer)**
   → Thu thập, làm sạch, chuẩn hóa.
   → Đóng vai trò “nhiên liệu” cho mọi mô hình.

2. **Mô hình (Model Layer)**
   → Nơi huấn luyện, fine-tune, và inference.
   → Dựa trên kiến trúc Transformer, LLM, CNN,...

3. **Hạ tầng (Infrastructure)**
   → GPU/TPU, Vector DB, API Gateway, Cache Layer.

4. **Lớp ứng dụng (Application Layer)**
   → Gồm chatbot, recommender, document Q&A, automation...

---

### 1.3. Chu trình phát triển hệ thống AI

```text
Data → Preprocessing → Model Training → Evaluation → Deployment → Monitoring
```

**Chi tiết từng bước:**

* **Preprocessing**: loại bỏ nhiễu, tokenization, embedding.
* **Training**: tối ưu hàm mất mát (loss), điều chỉnh hyperparameters.
* **Evaluation**: đo bằng Accuracy, F1, BLEU, ROUGE, nDCG...
* **Deployment**: đóng gói model → API / container / service.
* **Monitoring**: theo dõi drift, log, chi phí, latency.

---

### 1.4. Pipeline AI cơ bản

Ví dụ hệ thống **AI Q&A nội bộ**:

```
User → Query → Embedding → Vector Search → Context Retrieval → GPT Response
```

* **Embedding**: chuyển text thành vector ngữ nghĩa.
* **Vector DB**: lưu các vector (Chroma, Pinecone,...).
* **LLM (GPT)**: sinh câu trả lời có ngữ cảnh.

---

### 1.5. Các loại hệ thống AI thường gặp

| Hệ thống           | Mục tiêu                | Công nghệ chính        |
| ------------------ | ----------------------- | ---------------------- |
| Chatbot thông minh | Trò chuyện tự nhiên     | GPT, RAG               |
| Recommender System | Gợi ý cá nhân hóa       | Embeddings, Similarity |
| Document Assistant | Hỏi đáp từ tài liệu     | Vector DB + LLM        |
| AI Agent           | Tự động thực thi tác vụ | Function Calling       |
| Speech-to-Text     | Nhận diện giọng nói     | Whisper, ASR models    |

---

### 🧠 Câu hỏi phỏng vấn:

* “Pipeline AI từ input đến output hoạt động thế nào?”
* “Khác biệt giữa model-based và retrieval-based system?”
* “Vì sao cần monitoring khi triển khai model?”

---

## 🤖 **CHƯƠNG 2 – OPENAI API & ỨNG DỤNG TRONG HỆ THỐNG AI**

---

### 2.1. Kiến trúc OpenAI API

Gồm 3 lớp:

1. **Chat Models** – sinh ngôn ngữ tự nhiên (GPT-4, GPT-4o-mini).
2. **Embedding Models** – mã hóa vector (text-embedding-3).
3. **Moderation Models** – phát hiện nội dung nhạy cảm.

Mỗi lần gọi API là một **“Completion Request”**.

---

### 2.2. Chat Completion API

```python
from openai import OpenAI
client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "You are a Python tutor."},
        {"role": "user", "content": "Explain the difference between list and tuple."}
    ]
)
print(response.choices[0].message.content)
```

🧩 **Cấu trúc hội thoại:**

* `system` → định hướng hành vi.
* `user` → câu hỏi.
* `assistant` → phản hồi của mô hình.

---

### 2.3. Response Format – JSON có cấu trúc

Giúp tích hợp trực tiếp với backend.

```python
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Return a JSON of 3 colors with RGB."}],
    response_format={"type": "json_object"}
)
```

Output:

```json
{
  "colors": [
    {"name": "red", "rgb": [255, 0, 0]},
    {"name": "green", "rgb": [0, 255, 0]},
    {"name": "blue", "rgb": [0, 0, 255]}
  ]
}
```

---

### 2.4. Function Calling – cho phép GPT gọi API thật

**Mục tiêu:** GPT không chỉ “nói” mà còn “hành động”.

```python
tools = [{
  "type": "function",
  "function": {
    "name": "get_weather",
    "description": "Return weather info for a city",
    "parameters": {"type": "object", "properties": {"city": {"type": "string"}}}
  }
}]

response = client.chat.completions.create(
  model="gpt-4o-mini",
  messages=[{"role": "user", "content": "What's the weather in Tokyo?"}],
  tools=tools
)
print(response.choices[0].message.tool_calls)
```

Khi model gọi `get_weather`, backend có thể thực thi API thật.

---

### 2.5. Temperature, Top_p, Max_tokens

| Tham số         | Ảnh hưởng                              | Dùng khi               |
| --------------- | -------------------------------------- | ---------------------- |
| **temperature** | Độ sáng tạo (0 = chính xác, 1 = tự do) | Viết content           |
| **top_p**       | Xác suất chọn token (nucleus sampling) | Giữ đa dạng ngôn ngữ   |
| **max_tokens**  | Giới hạn độ dài output                 | Ngăn model nói quá dài |

---

### 2.6. Moderation – kiểm duyệt an toàn

```python
mod = client.moderations.create(input="I want to build a bomb.")
print(mod.results[0].categories)
```

Nếu kết quả `violence=True` → chặn yêu cầu.

---

### 2.7. Cost & Token Tracking

```python
import tiktoken
enc = tiktoken.encoding_for_model("gpt-4o-mini")
tokens = len(enc.encode("Hello world"))
print(tokens)
```

→ Mỗi model có giá/token khác nhau.
Ví dụ `gpt-4o-mini`:

* Input: $0.15 / 1M tokens
* Output: $0.6 / 1M tokens

---

### 🧠 Câu hỏi phỏng vấn:

* “Temperature = 0.8 nghĩa là gì?”
* “Function calling hoạt động thế nào?”
* “Moderation có bắt buộc không?”
* “Token là gì và vì sao phải quản lý chi phí?”

---

## 🧱 **CHƯƠNG 3 – XÂY DỰNG HỆ THỐNG AI HOÀN CHỈNH**

---

### 3.1. Mô hình kiến trúc tổng thể

```
User → API Gateway → LLM Layer → Vector Database → External Tools → Response
```

Các lớp chính:

1. **Frontend / Input Layer** – người dùng hoặc API nhận input.
2. **LLM Layer (OpenAI)** – xử lý logic, gọi function.
3. **Vector Layer (Chroma / Pinecone)** – tìm ngữ cảnh.
4. **External Tools** – API, DB, search engine.
5. **Monitoring Layer** – log, token, lỗi, cache.

---

### 3.2. Xây dựng Document Q&A System (RAG pipeline)

```python
from openai import OpenAI
import chromadb
client = OpenAI()

# 1️⃣ Tạo ChromaDB
chroma_client = chromadb.PersistentClient(path="./knowledge")
collection = chroma_client.create_collection(name="docs")

# 2️⃣ Thêm tài liệu
collection.add(ids=["1"], documents=["AI is transforming the world of automation."])

# 3️⃣ Embed câu hỏi
query = "What is the impact of AI on automation?"
query_emb = client.embeddings.create(model="text-embedding-3-small", input=query).data[0].embedding

# 4️⃣ Tìm kiếm vector gần nhất
results = collection.query(query_embeddings=[query_emb], n_results=1)
context = results["documents"][0][0]

# 5️⃣ Gửi câu hỏi kèm ngữ cảnh cho GPT
prompt = f"Context: {context}\nQuestion: {query}"
answer = client.chat.completions.create(model="gpt-4o-mini", messages=[{"role": "user", "content": prompt}])
print(answer.choices[0].message.content)
```

🧠 Đây là **RAG (Retrieval-Augmented Generation)** — kiến trúc phổ biến nhất trong phỏng vấn kỹ sư LLM.

---

### 3.3. Logging, Retry & Error Handling

```python
from tenacity import retry, stop_after_attempt, wait_random_exponential

@retry(wait=wait_random_exponential(min=1, max=60), stop=stop_after_attempt(5))
def get_answer(prompt):
    return client.chat.completions.create(model="gpt-4o-mini", messages=[{"role": "user", "content": prompt}])
```

→ Dùng **exponential backoff** để tránh rate-limit.

---

### 3.4. Triển khai Production

**Các thành phần thực tế:**

* FastAPI / Flask → REST API service.
* Redis / PostgreSQL → cache và logs.
* Supervisor / Docker → chạy bền vững.
* Prometheus / Grafana → theo dõi latency & usage.

---

### 3.5. Checklist phỏng vấn về hệ thống AI

| Chủ đề           | Câu hỏi phỏng vấn điển hình                       |
| ---------------- | ------------------------------------------------- |
| API              | “Hãy mô tả flow của OpenAI Chat Completion.”      |
| Embedding        | “Embedding dùng trong semantic search thế nào?”   |
| RAG              | “RAG khác gì fine-tune?”                          |
| Function calling | “Cách dùng function calling để truy cập DB?”      |
| Monitoring       | “Theo dõi drift & chi phí model ra sao?”          |
| Prompting        | “Khác biệt giữa zero-shot và few-shot prompting?” |

---

## 🎯 TỔNG KẾT CHO PHỎNG VẤN

1. **Biết pipeline tổng thể:**
   `Input → Embedding → Vector Search → Context → GPT Answer`.

2. **Hiểu rõ từng thành phần:**
   Chat Completion / Embedding / Moderation / Function Calling.

3. **Nắm best practices:**
   Guardrails, Retry, Token limits, Cost control.

4. **Thực hành thực tế:**
   Viết được app mini dùng OpenAI API + Vector DB.

---

## 💼 Mini Project bạn nên làm để chuẩn bị phỏng vấn

✅ **AI Resume Reviewer:**

* Input: file CV → GPT phân tích điểm mạnh, yếu, khuyến nghị cải thiện.
* Output: JSON structured + lời khuyên.
* Sử dụng: `chat.completions`, `response_format`, `temperature=0.2`.

✅ **RAG Chatbot nội bộ:**

* Ingest PDF → Embed → Vector DB → GPT.
* Gắn thêm logging & retry.

---
