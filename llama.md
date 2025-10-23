# 🦙 TOÀN TẬP VỀ LLAMA 3

### (Cấu trúc, cách chạy local, tham số, chat roles, structured output, multi-turn, prompt engineering, lỗi thường gặp & Q&A phỏng vấn)

---

## 🧩 1. Giới thiệu về LLaMA

### 1.1 Nguồn gốc & mục tiêu

**LLaMA (Large Language Model Meta AI)** là dòng mô hình mã nguồn mở của Meta, đối trọng với GPT.
Phiên bản mới nhất — **LLaMA 3** — có kích thước từ 8B đến 70B tham số, hỗ trợ:

* Nhiều ngôn ngữ (đa ngữ).
* Chat format tương thích OpenAI API (`messages`, `roles`).
* Chạy local bằng `llama.cpp` (file `.gguf`).
* Xuất JSON & multi-turn conversation.

### 1.2 Ưu điểm nổi bật

| Tính năng                | Lợi ích                                      |
| ------------------------ | -------------------------------------------- |
| Mã nguồn mở              | Tự do tùy biến, fine-tune local              |
| Hiệu suất cao            | Hỗ trợ quantization (Q4, Q5, Q8) để chạy CPU |
| API tương tự OpenAI      | Dễ tích hợp pipeline cũ                      |
| Structured output (JSON) | Đầu ra chuẩn hóa, dễ dùng cho backend        |
| Chat role system         | Giữ ngữ cảnh hội thoại tự nhiên              |

---

## ⚙️ 2. Cài đặt & chạy mô hình LLaMA local

### 2.1 Cài thư viện

```bash
pip install llama-cpp-python
```

### 2.2 Tải mô hình `.gguf`

Tải từ Hugging Face hoặc Meta (tùy license).
Ví dụ: `TheBloke/Llama-3-8B-Instruct-GGUF`.

---

### 2.3 Chạy inference đầu tiên

```python
from llama_cpp import Llama

llm = Llama(model_path="llama-3-8b-instruct.Q4_K_M.gguf")

out = llm("Explain the concept of reinforcement learning in 3 bullet points.")
print(out["choices"][0]["text"])
```

📘 **Giải thích:**

* `model_path` → file `.gguf` đã tải.
* Output là dict giống OpenAI response.
* Nội dung nằm ở `choices[0]['text']`.

🧠 **Hỏi phỏng vấn:**

> “LLaMA 3 có khác gì GPT không?”
> ✅ “Về kiến trúc Transformer là tương tự, nhưng LLaMA là open-source, có thể chạy local và tùy chỉnh, còn GPT là closed-source, chỉ chạy qua API.”

---

## 🎛️ 3. Các tham số điều khiển đầu ra

### 3.1 Tham số cơ bản

| Tham số       | Ý nghĩa                              | Ảnh hưởng                        |
| ------------- | ------------------------------------ | -------------------------------- |
| `temperature` | Mức độ sáng tạo (0–1.5)              | Thấp = chính xác, cao = sáng tạo |
| `top_k`       | Giới hạn số token được chọn mỗi bước | Giảm ngẫu nhiên khi nhỏ          |
| `top_p`       | Xác suất tích lũy cắt ngẫu nhiên     | 0.9 → phổ biến                   |
| `max_tokens`  | Giới hạn độ dài output               | Tránh output quá dài             |

---

### 3.2 Thực hành

```python
brief = llm(
    "Describe an electric car.",
    temperature=0.2, top_k=1, top_p=0.5, max_tokens=20
)
creative = llm(
    "Describe an electric car.",
    temperature=0.9, top_k=50, top_p=0.95, max_tokens=80
)
print("Brief:", brief["choices"][0]["text"])
print("Creative:", creative["choices"][0]["text"])
```

💡 Kinh nghiệm:

* Dùng `temperature < 0.3` cho **task chính xác (tóm tắt, trích thông tin)**.
* Dùng `temperature ~ 0.8` cho **task sáng tạo (viết content, brainstorming)**.

🧠 **Hỏi phỏng vấn:**

> “Khác biệt giữa top-k và top-p sampling?”
> ✅ “Top-k chọn K token xác suất cao nhất, top-p chọn theo ngưỡng xác suất tích lũy (nhiều hoặc ít token tùy context).”

---

## 💬 4. Chat role system (System/User/Assistant)

### 4.1 Cấu trúc chat

LLaMA 3 hỗ trợ **chat completion API tương tự OpenAI**, gồm 3 vai trò:

| Role        | Vai trò                    |
| ----------- | -------------------------- |
| `system`    | Định hướng hành vi mô hình |
| `user`      | Câu hỏi / đầu vào          |
| `assistant` | Câu trả lời của model      |

---

### 4.2 Ví dụ thực tế

```python
messages = [
    {"role": "system", "content": "You are a helpful data science tutor."},
    {"role": "user", "content": "Explain PCA in simple terms."}
]
out = llm.create_chat_completion(messages=messages)
print(out["choices"][0]["message"]["content"])
```

🧠 **Giải thích:**

* Lệnh `create_chat_completion` duy trì **ngữ cảnh hội thoại**.
* Role `system` nên được dùng đầu tiên để định hướng “nhân cách” model.

📘 *Nguồn:* `chapter1-llama.pdf` – phần Chat Roles.

---

## 🧱 5. Structured Output (JSON, Schema)

### 5.1 Xuất JSON thô

```python
resp = llm.create_chat_completion(
    messages=[
        {"role": "system", "content": "You are a weather API returning JSON only."},
        {"role": "user", "content": "Weather in Tokyo now?"}
    ],
    response_format="json_object"
)
print(resp["choices"][0]["message"]["content"])
```

📘 **Giải thích:**

* `response_format="json_object"` ép model trả JSON hợp lệ.
* Dùng để tích hợp với app/backend dễ dàng.

---

### 5.2 Định schema JSON (có cấu trúc)

```python
schema = {
  "type": "object",
  "properties": {
    "temperature": {"type": "string"},
    "humidity": {"type": "string"},
    "condition": {"type": "string"}
  },
  "required": ["temperature", "condition"]
}

resp = llm.create_chat_completion(
    messages=[{"role": "user", "content": "Give Tokyo weather"}],
    response_format={"type": "json_schema", "json_schema": schema}
)
print(resp["choices"][0]["message"]["content"])
```

💡 **Ứng dụng:**

* Chatbot data → API JSON trả kết quả có cấu trúc.
* Gắn với FastAPI hoặc Streamlit để hiển thị bảng dữ liệu.

🧠 **Hỏi phỏng vấn:**

> “Tại sao structured output quan trọng?”
> ✅ “Giúp model tương tác trực tiếp với hệ thống lập trình, tránh parsing lỗi, tăng tính tự động hóa.”

---

## 🔁 6. Multi-turn Conversation (hội thoại nhiều lượt)

### 6.1 Vấn đề

Model cần “nhớ” ngữ cảnh trước (history).
Nếu không, mỗi câu hỏi mới bị coi như hội thoại mới.

---

### 6.2 Cách làm: lưu `messages` lịch sử

```python
class ChatSession:
    def __init__(self, llm):
        self.llm = llm
        self.history = [{"role": "system", "content": "You are an AI assistant."}]
    
    def ask(self, text):
        self.history.append({"role": "user", "content": text})
        out = self.llm.create_chat_completion(messages=self.history)
        msg = out["choices"][0]["message"]
        self.history.append(msg)
        return msg["content"]

session = ChatSession(llm)
print(session.ask("Who is Alan Turing?"))
print(session.ask("What was his main contribution?"))
```

📘 *Nguồn:* `llama.txt` & `chapter2-llama.pdf`.

🧠 **Lợi ích:**

* Duy trì ngữ cảnh.
* Mô phỏng hội thoại tự nhiên.
* Dễ mở rộng cho chatbot có bộ nhớ ngắn hạn.

---

## 🧠 7. Prompt Engineering nâng cao

### 7.1 Zero-shot Prompting

Chỉ cung cấp hướng dẫn mô tả nhiệm vụ.

```python
prompt = "Classify the following text as Positive, Negative, or Neutral:\n\n'The movie was boring.'"
print(llm(prompt))
```

💡 *Mẹo:*
Thêm “**INSTRUCTION:**” hoặc “**QUESTION:**” giúp model phân tách rõ input vs yêu cầu.

---

### 7.2 Few-shot Prompting

Cung cấp **mẫu ví dụ** để model học cách phản hồi.

```python
prompt = """Translate English to French:

EN: Hello
FR: Bonjour
EN: Good morning
FR:"""
print(llm(prompt))
```

🧠 *Phỏng vấn:*

> “Few-shot giúp gì so với zero-shot?”
> ✅ “Nó cho mô hình hiểu format và ngữ cảnh mong muốn.”

---

### 7.3 Stop Sequences

Giúp **dừng output** tại chuỗi cụ thể (vd: `"Q:"`, `"User:"`).

```python
out = llm(
    "Q: What is 2+2?\nA:",
    stop=["Q:"]
)
print(out["choices"][0]["text"])
```

💡 *Ứng dụng:* tránh “lạc đề” khi model tạo thêm câu hỏi giả.

---

## 🚫 8. Lỗi thường gặp & cách khắc phục

| Lỗi                        | Nguyên nhân                               | Cách khắc phục                                         |
| -------------------------- | ----------------------------------------- | ------------------------------------------------------ |
| Output lộn xộn, không JSON | Không đặt `response_format="json_object"` | Thêm `response_format` hoặc prompt “Return valid JSON” |
| Model quên ngữ cảnh        | Không lưu `messages`                      | Lưu `history` toàn hội thoại                           |
| Câu trả lời quá dài        | Không set `max_tokens` hoặc `stop`        | Giới hạn `max_tokens`                                  |
| Quá chậm                   | File GGUF quá lớn                         | Dùng quantization Q4_K_M                               |

---

## 💼 9. Câu hỏi thường gặp
| Câu hỏi                             | Trả lời gợi ý                                                                 |
| ----------------------------------- | ----------------------------------------------------------------------------- |
| LLaMA là gì?                        | Là họ mô hình ngôn ngữ mã nguồn mở của Meta, tương tự GPT.                    |
| LLaMA 3 có gì nổi bật?              | Hiệu suất cao, open-source, tương thích OpenAI API, hỗ trợ JSON & multi-turn. |
| temperature ảnh hưởng thế nào?      | Quyết định mức ngẫu nhiên – thấp chính xác, cao sáng tạo.                     |
| top-k vs top-p khác nhau ra sao?    | top-k chọn K token xác suất cao, top-p chọn theo ngưỡng xác suất tích lũy.    |
| Chat roles là gì?                   | system định hướng hành vi, user cung cấp câu hỏi, assistant trả lời.          |
| Structured output quan trọng không? | Rất – giúp ứng dụng backend đọc kết quả chính xác & tự động.                  |
| Stop sequences dùng khi nào?        | Khi cần dừng sinh văn bản ở một vị trí cố định.                               |
| Multi-turn hoạt động ra sao?        | Lưu lịch sử `messages` và truyền lại cho model để duy trì ngữ cảnh.           |

---

