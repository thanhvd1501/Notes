# 🤖 HUGGING FACE & NLP

### (Dành cho AI / ML / NLP Engineer)

---

## 🧩 I. HUGGING FACE CƠ BẢN

| Khái niệm            | Mô tả                                                              | Câu trả lời mẫu phỏng vấn                                                                       |
| -------------------- | ------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------- |
| **Hugging Face**     | Nền tảng mã nguồn mở cho cộng đồng AI chia sẻ model, dataset, app. | “Là nơi tập trung mô hình AI (NLP, CV, Audio), dữ liệu và API phục vụ inference và huấn luyện.” |
| **Hugging Face Hub** | Website nơi bạn tìm kiếm, tải và upload model/dataset.             | “Hub cung cấp model cards, dataset cards và API tích hợp dễ dàng qua Python.”                   |
| **Thư viện chính**   | `transformers`, `datasets`, `huggingface_hub`                      | “transformers cho model, datasets cho dữ liệu, hub để quản lý model/dataset.”                   |

---

## ⚙️ II. TRANSFORMERS LIBRARY

### 1️⃣ `pipeline` – Cách nhanh nhất để chạy mô hình

```python
from transformers import pipeline
pipe = pipeline("text-generation", model="openai-community/gpt2")
print(pipe("What if AI")[0]["generated_text"])
```

✅ **Câu hỏi:**

> “pipeline trong Hugging Face là gì?”

💬 **Trả lời:**

> “Là interface cấp cao giúp load model, tokenizer, và thực thi inference chỉ bằng vài dòng code.”

---

### 2️⃣ Các loại pipeline phổ biến

| Task                       | Mô hình tiêu biểu                                 | Ứng dụng                               |
| -------------------------- | ------------------------------------------------- | -------------------------------------- |
| `text-classification`      | `distilbert-base-uncased-finetuned-sst-2-english` | Phân loại sentiment, spam              |
| `summarization`            | `sshleifer/distilbart-cnn-12-6`                   | Tóm tắt văn bản                        |
| `question-answering`       | `distilbert-base-cased-distilled-squad`           | Trả lời câu hỏi theo ngữ cảnh          |
| `zero-shot-classification` | `facebook/bart-large-mnli`                        | Phân loại văn bản không cần huấn luyện |
| `text-generation`          | `gpt2`, `mistral`, `llama-3`                      | Sinh văn bản, chatbot                  |

---

## 💬 III. TEXT CLASSIFICATION

### 💡 Ví dụ

```python
from transformers import pipeline
classifier = pipeline("text-classification", model="distilbert-base-uncased-finetuned-sst-2-english")
print(classifier("I love pineapple on pizza!"))
```

> Output → `[{'label': 'POSITIVE', 'score': 0.99}]`

### 💭 Các dạng classification:

| Loại                         | Mục đích                              | Ví dụ                                     |
| ---------------------------- | ------------------------------------- | ----------------------------------------- |
| **Sentiment Analysis**       | Cảm xúc tích cực / tiêu cực           | “Bad service!” → Negative                 |
| **Grammar Check**            | Kiểm tra ngữ pháp                     | “He eat pizza” → Sai                      |
| **QNLI**                     | Câu trả lời có khớp với câu hỏi không | “Seattle in Washington?” ✅                |
| **Zero-shot classification** | Phân loại vào nhóm chưa học           | Văn bản → “sales”, “support”, “marketing” |

> 💬 **Câu hỏi:**
> “Zero-shot classification khác gì supervised?”
> ✅ **Trả lời:** “Zero-shot không cần huấn luyện nhãn sẵn – model dựa vào ngữ nghĩa của nhãn để phân loại.”

---

## 🧾 IV. TEXT SUMMARIZATION

### ✍️ Khái niệm:

| Kiểu            | Đặc điểm                | Ứng dụng                    |
| --------------- | ----------------------- | --------------------------- |
| **Extractive**  | Chọn câu gốc quan trọng | Báo cáo tài chính, hợp đồng |
| **Abstractive** | Tạo lại câu mới cô đọng | Tóm tắt tin tức, blog       |

### 🔹 Ví dụ:

```python
from transformers import pipeline
summarizer = pipeline("summarization", model="sshleifer/distilbart-cnn-12-6")
summary = summarizer("Data Science is a multidisciplinary field...", max_new_tokens=60)
print(summary[0]['summary_text'])
```

💬 **Câu hỏi:**

> “Điểm khác biệt giữa extractive và abstractive summarization là gì?”
> ✅ “Extractive chọn câu gốc, abstractive tạo câu mới — abstractive tự nhiên hơn nhưng đòi hỏi tài nguyên cao hơn.”

---

## 📚 V. DOCUMENT Q&A (Question Answering on PDFs)

### 📄 Quy trình:

1️⃣ Dùng `pypdf` để đọc nội dung PDF
2️⃣ Tạo `pipeline("question-answering")`
3️⃣ Truyền `context` = nội dung file, `question` = câu hỏi

```python
from pypdf import PdfReader
from transformers import pipeline

reader = PdfReader("policy.pdf")
context = "".join([p.extract_text() for p in reader.pages])

qa = pipeline("question-answering", model="distilbert-base-cased-distilled-squad")
print(qa(question="How many holidays per year?", context=context)["answer"])
```

💬 **Câu hỏi:**

> “Bạn xây dựng hệ thống hỏi đáp tài liệu thế nào?”
> ✅ “Extract text bằng `pypdf`, dùng `QA pipeline` của Hugging Face để trả lời dựa trên context.”

---

## 🧠 VI. AUTOMODEL & AUTOTOKENIZER

### 🔹 Khi cần kiểm soát chi tiết hơn:

```python
from transformers import AutoModelForSequenceClassification, AutoTokenizer, pipeline

model = AutoModelForSequenceClassification.from_pretrained("distilbert-base-uncased-finetuned-sst-2-english")
tokenizer = AutoTokenizer.from_pretrained("distilbert-base-uncased-finetuned-sst-2-english")

custom_pipe = pipeline("sentiment-analysis", model=model, tokenizer=tokenizer)
print(custom_pipe("This course is amazing!"))
```

### 💬 So sánh nhanh:

| Tiêu chí        | `pipeline`       | `AutoModel` + `AutoTokenizer` |
| --------------- | ---------------- | ----------------------------- |
| Mức độ đơn giản | Dễ dùng          | Phức tạp hơn                  |
| Linh hoạt       | Hạn chế          | Toàn quyền kiểm soát          |
| Dùng khi nào?   | Demo, thử nghiệm | Fine-tuning, production       |

---

## 🧩 VII. HUGGING FACE DATASETS

```python
from datasets import load_dataset
data = load_dataset("IVN-RIN/BioBERT_Italian", split="train")

# Lọc dữ liệu
filtered = data.filter(lambda row: "bella" in row['text'])
print(filtered)
```

💬 **Câu hỏi:**

> “Hugging Face Datasets dùng định dạng gì?”
> ✅ “Apache Arrow – lưu dữ liệu dạng cột, giúp truy vấn nhanh và tiết kiệm bộ nhớ.”

---

## 🚀 VIII. CÂU HỎI THƯỜNG GẶP

| Câu hỏi                                      | Cách trả lời ngắn gọn                                         |
| -------------------------------------------- | ------------------------------------------------------------- |
| Hugging Face là gì?                          | Cộng đồng mã nguồn mở chia sẻ model, dataset, công cụ AI.     |
| pipeline trong transformers là gì?           | Interface giúp chạy mô hình nhanh với cấu hình sẵn.           |
| Khác nhau giữa pipeline và AutoModel?        | pipeline = tiện, AutoModel = linh hoạt.                       |
| Zero-shot classification là gì?              | Phân loại mà model chưa được huấn luyện nhãn sẵn.             |
| Summarization có mấy loại?                   | 2 loại: Extractive và Abstractive.                            |
| Làm sao đọc PDF và hỏi đáp nội dung?         | Dùng `pypdf` + `pipeline(question-answering)`.                |
| Datasets của Hugging Face dùng định dạng gì? | Apache Arrow.                                                 |
| Transformer model xử lý ngôn ngữ thế nào?    | Dùng cơ chế self-attention để học mối quan hệ giữa các token. |

---
