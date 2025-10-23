# 🧠 TOÀN TẬP KIẾN THỨC EMBEDDING, VECTOR DATABASE & SEMANTIC SEARCH

### (Từ cơ bản đến xây dựng hệ thống AI thực tế & phỏng vấn kỹ thuật)

---

## 🌱 CHƯƠNG 1: NỀN TẢNG VỀ EMBEDDINGS

### 🧩 1. Embedding là gì?

**Định nghĩa:**
Embeddings là **biểu diễn số (vector)** của văn bản, hình ảnh, âm thanh... trong **không gian nhiều chiều** (vector space), trong đó các điểm gần nhau mang nghĩa tương tự nhau.

* Ví dụ:

  * “cat” và “dog” → gần nhau trong vector space.
  * “car” và “apple” → xa nhau.

👉 Giúp máy tính “hiểu nghĩa” thay vì chỉ “nhìn từ”.

**Mô hình tạo embedding phổ biến:**

* OpenAI: `text-embedding-3-small`, `text-embedding-3-large`
* Sentence-BERT (SBERT)
* Instructor / E5 (Hugging Face)

---

### ⚙️ 2. Cấu trúc vector Embedding

Mỗi văn bản → 1 vector gồm **1536 giá trị float** (với `text-embedding-3-small`).

Ví dụ:

```python
from openai import OpenAI
client = OpenAI()

response = client.embeddings.create(
    model="text-embedding-3-small",
    input="AI helps automate tasks"
)
print(len(response.data[0].embedding))  # 1536
```

* Mỗi phần tử ≈ “tọa độ” thể hiện ý nghĩa.
* Dữ liệu này có thể được so sánh bằng các hàm khoảng cách.

---

### 📏 3. Đo độ tương đồng (Similarity Metrics)

#### ✅ Cosine Similarity

* Phổ biến nhất.
* Tính bằng công thức:
  [
  \text{similarity} = \frac{A \cdot B}{||A|| \times ||B||}
  ]
  (Tích vô hướng chia cho tích độ dài vector)

Giá trị:

* 1 → giống hệt
* 0 → khác hoàn toàn

Python:

```python
from scipy.spatial import distance
similarity = 1 - distance.cosine(v1, v2)
```

---

### 💡 4. Ứng dụng của Embeddings

| Ứng dụng                                 | Mô tả                                                     |
| ---------------------------------------- | --------------------------------------------------------- |
| **Semantic Search**                      | Tìm nội dung “cùng nghĩa” thay vì khớp từ khóa            |
| **Recommendation**                       | Gợi ý nội dung tương tự                                   |
| **Classification**                       | Gán nhãn cho văn bản dựa trên độ tương đồng               |
| **Clustering**                           | Gom nhóm văn bản tương tự                                 |
| **RAG (Retrieval-Augmented Generation)** | Lấy thông tin ngữ cảnh để tăng chất lượng trả lời của LLM |

---

## 🔎 CHƯƠNG 2: ỨNG DỤNG EMBEDDING TRONG AI

---

### 🧭 A. SEMANTIC SEARCH (Tìm kiếm ngữ nghĩa)

**Mục tiêu:**
Tìm các văn bản “gần nghĩa” nhất với câu truy vấn.

**Pipeline cơ bản:**

1. Embed toàn bộ văn bản dữ liệu.
2. Embed câu truy vấn.
3. Tính cosine similarity giữa query và mỗi văn bản.
4. Trả về top N kết quả gần nhất.

**Ví dụ:**

```python
from openai import OpenAI
from scipy.spatial import distance
import numpy as np

client = OpenAI()

# Embed 2 văn bản
docs = ["I love AI", "I like cooking pasta"]
embs = [client.embeddings.create(model="text-embedding-3-small", input=d).data[0].embedding for d in docs]

# Query
query_emb = client.embeddings.create(model="text-embedding-3-small", input="Artificial intelligence is great").data[0].embedding

# Tính similarity
similarities = [1 - distance.cosine(query_emb, e) for e in embs]
print(np.argmax(similarities))  # -> chỉ mục văn bản gần nhất
```

---

### 🧩 B. RECOMMENDATION SYSTEM (Hệ gợi ý)

Giống semantic search, nhưng:

* Query = embedding của **item người dùng đã xem**.
* Tính similarity để gợi ý các item khác.

**Gợi ý trên nhiều hành vi:**

```python
import numpy as np
user_vector = np.mean([v1, v2, v3], axis=0)
```

→ Trung bình embedding các bài viết đã xem → tìm các item gần nhất.

---

### 🧾 C. CLASSIFICATION (Phân loại không giám sát)

Zero-shot classification bằng Embedding.

**Ý tưởng:**

1. Embed các nhãn (label).
2. Embed văn bản cần phân loại.
3. Tính khoảng cách, gán nhãn gần nhất.

```python
labels = ["Tech", "Business", "Sports"]
label_embs = [embed(l) for l in labels]
article_emb = embed("AI startup raises $10M")

pred = np.argmin([distance.cosine(article_emb, le) for le in label_embs])
print(labels[pred])  # Tech
```

💬 *Phỏng vấn:*

> “Làm sao phân loại văn bản mà không có dữ liệu huấn luyện?”
> ✅ Trả lời: “Bằng cách sử dụng zero-shot classification dựa trên Embeddings và cosine similarity.”

---

## 🗄️ CHƯƠNG 3: VECTOR DATABASES

---

### 📘 1. Vấn đề với cách tiếp cận cũ

* Mỗi embedding có 1536 float ≈ 13 KB.
* Với 100k văn bản → >1 GB RAM chỉ để lưu embedding.
* So sánh tuyến tính (O(n)) rất chậm.
  💡 → Cần giải pháp lưu & truy vấn tối ưu: **Vector Databases**.

---

### 🧱 2. Khái niệm Vector Database

**Chức năng:**

* Lưu trữ vector embeddings + metadata.
* Cho phép truy vấn theo ngữ nghĩa (similarity search).
* Tối ưu hóa bằng index (HNSW, FAISS…).

**Ví dụ:** ChromaDB, Pinecone, Weaviate, Milvus, Qdrant.

---

### ⚙️ 3. Kiến trúc cơ bản

| Thành phần     | Vai trò                           |
| -------------- | --------------------------------- |
| **Embeddings** | Vector của văn bản                |
| **Documents**  | Nội dung nguồn                    |
| **Metadata**   | Thông tin phụ (ID, type, year...) |
| **Indexes**    | Cấu trúc giúp tìm kiếm nhanh      |

---

### 🧩 4. Cài đặt ChromaDB

**Cài đặt:**

```bash
pip install chromadb openai
```

**Kết nối DB:**

```python
import chromadb
from chromadb.utils.embedding_functions import OpenAIEmbeddingFunction

client = chromadb.PersistentClient(path="./chroma_db")
collection = client.create_collection(
    name="articles",
    embedding_function=OpenAIEmbeddingFunction(model_name="text-embedding-3-small", api_key="...")
)
```

---

### 📚 5. Thêm dữ liệu vào DB

```python
collection.add(
    ids=["doc1", "doc2"],
    documents=["AI changes the world", "Cooking with friends is fun"]
)
```

📌 Tự động tạo embeddings khi thêm dữ liệu.

---

### 🔍 6. Truy vấn dữ liệu

```python
results = collection.query(
    query_texts=["artificial intelligence"],
    n_results=2
)
print(results["documents"])
```

Kết quả trả về:

* `ids`: ID tài liệu tương ứng
* `documents`: văn bản gốc
* `distances`: khoảng cách cosine

---

### 🔄 7. Cập nhật / Xóa dữ liệu

```python
# Update
collection.update(ids=["doc1"], documents=["Updated text"])

# Upsert (thêm nếu chưa có)
collection.upsert(ids=["doc3"], documents=["New document"])

# Xóa
collection.delete(ids=["doc2"])
```

---

### 🧠 8. Lọc kết quả (Filtering)

Thêm metadata:

```python
collection.update(
    ids=["doc1"],
    metadatas=[{"type": "movie", "release_year": 2022}]
)
```

Lọc khi query:

```python
results = collection.query(
    query_texts=["romantic comedies"],
    n_results=3,
    where={"type": "movie", "release_year": {"$gt": 2020}}
)
```

---

### 💸 9. Ước tính chi phí embedding

```python
import tiktoken
enc = tiktoken.encoding_for_model("text-embedding-3-small")
tokens = sum(len(enc.encode(text)) for text in documents)
cost = 0.00002 * tokens / 1000
print(f"Tokens: {tokens}, Cost: ${cost:.5f}")
```

---

## 🚀 CHƯƠNG 4: XÂY DỰNG ỨNG DỤNG THỰC TẾ

---

### 💬 A. Document Q&A System

**Mục tiêu:** Trả lời câu hỏi từ PDF.

```python
from pypdf import PdfReader
from transformers import pipeline

reader = PdfReader("policy.pdf")
text = "".join([page.extract_text() for page in reader.pages])

qa = pipeline("question-answering", model="distilbert-base-cased-distilled-squad")
print(qa(question="How many holidays?", context=text))
```

> Ứng dụng: Chatbot HR, pháp lý, nội quy công ty.

---

### 🔍 B. Semantic Search + ChromaDB

* Lưu embeddings của văn bản.
* Khi query: embed câu hỏi → tìm top N văn bản gần nhất.
* Cung cấp lại cho mô hình ngôn ngữ → trả lời chính xác hơn.

💡 Đây là nguyên lý **RAG (Retrieval-Augmented Generation)**.

---

### 🎯 C. Recommendation Engine

* Embed bài viết người dùng đã đọc.
* Tính trung bình vector.
* Query ChromaDB để lấy top tương tự.

---

## 💼 CHƯƠNG 5: CÂU HỎI THỰC TẾ

| Chủ đề                | Câu hỏi                               | Trả lời mẫu                                             |
| --------------------- | ------------------------------------- | ------------------------------------------------------- |
| **Embeddings**        | Embedding là gì?                      | Vector biểu diễn ý nghĩa văn bản.                       |
| **Cosine similarity** | Vì sao dùng cosine distance?          | So sánh hướng vector, bỏ qua độ dài.                    |
| **Vector DB**         | Lợi ích của Vector DB?                | Lưu & truy vấn embedding hiệu quả, mở rộng quy mô.      |
| **ChromaDB**          | ChromaDB khác Pinecone thế nào?       | Chroma là open-source, Pinecone là dịch vụ quản lý.     |
| **RAG**               | Retrieval-Augmented Generation là gì? | Kết hợp truy vấn vector + LLM để trả lời chính xác hơn. |
| **Ứng dụng thực tế**  | Làm sao xây hệ thống Q&A PDF?         | pypdf → extract text → embed → ChromaDB → query.        |

---
