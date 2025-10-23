# 🚀 TOÀN TẬP OPENAI API — XÂY DỰNG HỆ THỐNG AI THỰC TẾ

*(Dành cho kỹ sư AI/LLM Developer – chuẩn phỏng vấn kỹ thuật)*

---

## 🧠 PHẦN 1: KIẾN TRÚC & CƠ CHẾ HOẠT ĐỘNG CỦA OPENAI API

---

### 🧩 1.1 Kiến trúc cơ bản

OpenAI API là **nền tảng đám mây** cho phép gọi các mô hình ngôn ngữ (GPT, embedding, moderation…) qua REST API hoặc thư viện Python.

* Thành phần chính:

  * **Model** → Tên mô hình (ví dụ `"gpt-4o-mini"`, `"text-embedding-3-large"`)
  * **Messages** → Danh sách hội thoại (`system`, `user`, `assistant`)
  * **Response** → Đầu ra mô hình (text hoặc JSON)

### 🧩 1.2 Cấu trúc một API call cơ bản

```python
from openai import OpenAI
client = OpenAI(api_key="YOUR_API_KEY")

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "user", "content": "Who developed ChatGPT?"}
    ]
)

print(response.choices[0].message.content)
```

**Output:**

```
ChatGPT was developed by OpenAI, an artificial intelligence research lab.
```

🧠 *Phỏng vấn:*

> “Cấu trúc một request của OpenAI API gồm gì?”
> ✅ Trả lời: “Model + Messages + (tuỳ chọn) response_format, tools, temperature, max_tokens...”

---

### 🧩 1.3 Định dạng JSON có cấu trúc

Dùng `response_format` để ép mô hình trả JSON chuẩn — rất quan trọng khi tích hợp với backend.

```python
response = client.chat.completions.create(
  model="gpt-4o-mini",
  messages=[
    {"role": "user", "content": "List 5 trees with their scientific names in JSON."}
  ],
  response_format={"type": "json_object"}
)

print(response.choices[0].message.content)
```

**Output:**

```json
{
  "trees": [
    {"commonName": "Oak", "scientificName": "Quercus"},
    {"commonName": "Maple", "scientificName": "Acer"},
    {"commonName": "Pine", "scientificName": "Pinus"},
    {"commonName": "Birch", "scientificName": "Betula"},
    {"commonName": "Willow", "scientificName": "Salix"}
  ]
}
```

---

## ⚙️ PHẦN 2: LẬP TRÌNH HỆ THỐNG AI — LÀM CHỦ FUNCTION CALLING

---

### 🧠 2.1 Khái niệm Function Calling

**Function Calling** là cơ chế cho phép mô hình **tự động gọi các hàm Python hoặc API ngoài** để lấy thông tin chính xác hơn, giúp mô hình từ “chatbot tĩnh” trở thành “agent động”.

---

### 🧩 2.2 Cấu trúc một function

```python
function_definition = [{
  "type": "function",
  "function": {
    "name": "extract_job_info",
    "description": "Get job information from text",
    "parameters": {
      "type": "object",
      "properties": {
        "job": {"type": "string", "description": "Job title"},
        "location": {"type": "string", "description": "Location"}
      },
      "required": ["job", "location"]
    }
  }
}]
```

---

### 🧩 2.3 Gọi function trong Chat Completion

```python
response = client.chat.completions.create(
  model="gpt-4o-mini",
  messages=[{"role": "user", "content": "We are hiring a Data Scientist in San Francisco, CA."}],
  tools=function_definition
)

print(response.choices[0].message.tool_calls[0].function.arguments)
```

**Output:**

```json
{"job": "Data Scientist", "location": "San Francisco, CA"}
```

🧠 *Phỏng vấn:*

> “Function calling dùng để làm gì?”
> ✅ Trả lời: “Để chuyển đầu ra ngôn ngữ tự nhiên thành dữ liệu có cấu trúc, hoặc gọi API ngoài.”

---

### 🧩 2.4 Gọi nhiều hàm song song (Parallel Function Calling)

```python
function_definition.append({
  "type": "function",
  "function": {
    "name": "get_timezone",
    "description": "Return timezone for a location",
    "parameters": {"type": "object", "properties": {"timezone": {"type": "string"}}}
  }
})

response = client.chat.completions.create(
  model="gpt-4o-mini",
  messages=messages,
  tools=function_definition
)

print(response.choices[0].message.tool_calls[0].function.arguments)
print(response.choices[0].message.tool_calls[1].function.arguments)
```

🧠 *Kỹ năng phỏng vấn nâng cao:*

> “Làm sao để ép mô hình chỉ gọi 1 hàm cụ thể?”
> ✅ Dùng `tool_choice={"type": "function", "function": {"name": "extract_job_info"}}`

---

### 🧩 2.5 Gọi API ngoài qua `requests`

```python
import requests

def get_artwork(keyword):
    url = "https://api.artic.edu/api/v1/artworks/search"
    params = {"q": keyword}
    return requests.get(url, params=params).text
```

Sau đó gọi:

```python
if response.choices[0].finish_reason == "tool_calls":
    call = response.choices[0].message.tool_calls[0].function
    if call.name == "get_artwork":
        kw = json.loads(call.arguments)["artwork keyword"]
        print(get_artwork(kw))
```

💡 *Ứng dụng thực tế:*

* Chatbot truy vấn cơ sở dữ liệu
* AI travel planner (gọi API thời tiết, địa điểm)
* AI hỗ trợ HR (trích xuất dữ liệu từ JD)

---

## 🧰 PHẦN 3: QUẢN LÝ LỖI, RATE LIMITS & TOKEN

---

### ⚠️ 3.1 Error Handling

Các lỗi phổ biến trong OpenAI API:

| Loại lỗi              | Nguyên nhân           | Cách xử lý                      |
| --------------------- | --------------------- | ------------------------------- |
| `AuthenticationError` | Sai API key           | Kiểm tra key, load từ env       |
| `RateLimitError`      | Gọi quá nhiều request | Dùng batching, retry            |
| `BadRequestError`     | Sai cấu trúc messages | Kiểm tra role hợp lệ            |
| `APIConnectionError`  | Mất kết nối           | Thử lại hoặc báo lỗi người dùng |

---

### ⚙️ 3.2 Retry tự động với `tenacity`

```python
from tenacity import retry, stop_after_attempt, wait_random_exponential

@retry(wait=wait_random_exponential(min=1, max=60), stop=stop_after_attempt(6))
def get_response(model, message):
    return client.chat.completions.create(
        model=model,
        messages=[message]
    )
```

💡 *Phỏng vấn:*

> “Nếu gặp lỗi 429 Rate Limit, bạn làm gì?”
> ✅ “Tôi dùng retry có exponential backoff hoặc gom batch các request.”

---

### 🧩 3.3 Batching

```python
countries = ["USA", "Ireland", "India"]
messages = [{"role": "system", "content": "Return each country and its capital."}]
[ messages.append({"role": "user", "content": c}) for c in countries ]

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=messages
)
```

---

### 🔢 3.4 Tính số tokens (giới hạn độ dài input)

```python
import tiktoken
encoding = tiktoken.encoding_for_model("gpt-4o-mini")
prompt = "Tokens can be words or subwords."
print(len(encoding.encode(prompt)))
```

🧠 *Phỏng vấn:*

> “Nếu mô hình trả incomplete output, nguyên nhân?”
> ✅ “Hết token output hoặc prompt quá dài.”

---

## 🧱 PHẦN 4: MODERATION & SAFETY

---

### 🔒 4.1 Moderation API

Kiểm tra input có vi phạm chính sách không (bạo lực, thù ghét, tình dục, v.v.)

```python
moderation_response = client.moderations.create(input="Someone explodes in the game.")
print(moderation_response.results[0].categories.violence)
```

➡️ Output: `True`

---

### 🧱 4.2 Prompt Guardrails

Giới hạn chủ đề hoặc chặn “prompt injection”.

```python
user_request = "Describe the Exploding Kittens card game."
messages = [
  {"role": "system", "content": "You can only talk about chess. If not, say 'not allowed'."},
  {"role": "user", "content": user_request}
]

response = client.chat.completions.create(model="gpt-4o-mini", messages=messages)
print(response.choices[0].message.content)
```

➡️ Output: “Apologies, but the topic is not allowed.”

---

### 🧠 4.3 Validation & Adversarial Testing

* Test model với **prompt độc hại hoặc mâu thuẫn** để phát hiện lỗi.
* Thực hành kiểm thử với Hugging Face datasets (`openai/evals`).

```python
response = client.chat.completions.create(
  model="gpt-4o-mini",
  messages=[
    {"role": "system", "content": "You analyze movie reviews."},
    {"role": "user", "content": "This movie had great actors but bad story."}
])
print(response.choices[0].message.content)
```

➡️ Output: “The sentiment is negative.”

---

### 🔑 4.4 Bảo mật API key

* Lưu trong biến môi trường (`os.getenv("OPENAI_API_KEY")`)
* Không commit vào GitHub
* Xoay key định kỳ

```bash
export OPENAI_API_KEY="your_key_here"
```

---

## 🧩 PHẦN 5: BEST PRACTICES TRONG SẢN XUẤT

| Hạng mục              | Thực hành tốt                        |
| --------------------- | ------------------------------------ |
| **Error Handling**    | Luôn try-except chi tiết             |
| **Moderation**        | Kiểm tra input trước khi gửi model   |
| **Rate Limit**        | Batch + retry có backoff             |
| **Prompt Control**    | Rõ vai trò system/user               |
| **Validation**        | Adversarial testing trước production |
| **Key Management**    | Lưu qua KMS hoặc Vault               |
| **Human in the Loop** | Có người kiểm định với case nhạy cảm |

---

## 🎯 CÂU HỎI

| Câu hỏi                            | Trả lời mẫu                                        |
| ---------------------------------- | -------------------------------------------------- |
| Function calling dùng làm gì?      | Tạo dữ liệu có cấu trúc hoặc gọi API ngoài.        |
| Làm sao giảm lỗi RateLimitError?   | Batch, retry exponential, giảm token.              |
| Moderation API có vai trò gì?      | Phát hiện nội dung vi phạm chính sách.             |
| Làm sao đảm bảo JSON hợp lệ?       | Dùng `response_format="json_object"`.              |
| Khác nhau giữa system & user role? | System định hướng, user cung cấp input.            |
| Prompt injection là gì?            | Khi user cố gắng thay đổi hành vi model trái phép. |
| Cách bảo mật API key?              | Lưu trong biến môi trường, không push lên repo.    |

---
