# 🧠 TOÀN TẬP PYTHON CHO DEVELOPER (TỪ CƠ BẢN → PHỎNG VẤN)

---

## ⚙️ **CHƯƠNG 1 – LÀM CHỦ CƠ BẢN PYTHON**

---

### 1.1. Vì sao chọn Python?

* **Đơn giản, dễ đọc** – gần ngôn ngữ tự nhiên.
* **Miễn phí, mã nguồn mở** – cộng đồng cực lớn.
* **Thư viện đa dạng:** NumPy, Pandas, TensorFlow, FastAPI...
* **Ứng dụng:** Automation, AI, Web, Data Science, ML, API.

📘 **Phỏng vấn hỏi:**

> “Tại sao AI Engineer chọn Python?”
> ✅ Vì cú pháp ngắn, thư viện mạnh cho xử lý dữ liệu & machine learning.

---

### 1.2. Chạy code & cú pháp cơ bản

```python
print("Hello Python")
print(4 + 5)
print(10 - 4)
print(7 / 3)   # Chia thực
print(7 // 3)  # Chia lấy phần nguyên
print(7 % 3)   # Lấy dư
```

🔹 Dấu `#` là comment (không được chạy).
🔹 Python chạy theo **thứ tự dòng từ trên xuống**.

---

### 1.3. Biến & kiểu dữ liệu

```python
name = "George"
age = 29
balance = 58.50
is_active = True
```

| Kiểu    | Ví dụ        | Giải thích |
| ------- | ------------ | ---------- |
| `int`   | 25           | Số nguyên  |
| `float` | 25.5         | Số thực    |
| `str`   | "Python"     | Chuỗi      |
| `bool`  | True / False | Boolean    |

👉 *Python là dynamically typed — không cần khai báo kiểu trước.*

---

### 1.4. Hàm `type()` để kiểm tra kiểu

```python
print(type(name))      # <class 'str'>
print(type(balance))   # <class 'float'>
print(type(is_active)) # <class 'bool'>
```

---

### 1.5. Toán tử cơ bản

| Toán tử | Ý nghĩa     | Ví dụ         |
| ------- | ----------- | ------------- |
| `+`     | Cộng        | `2 + 3 = 5`   |
| `-`     | Trừ         | `5 - 2 = 3`   |
| `*`     | Nhân        | `4 * 2 = 8`   |
| `/`     | Chia thực   | `7 / 2 = 3.5` |
| `//`    | Chia nguyên | `7 // 2 = 3`  |
| `%`     | Chia dư     | `7 % 2 = 1`   |
| `**`    | Lũy thừa    | `3 ** 2 = 9`  |

---

## ✍️ **CHƯƠNG 2 – CẤU TRÚC DỮ LIỆU**

---

### 2.1. String – Chuỗi ký tự

#### Cú pháp:

```python
my_text = "Hello, I'm a Developer."
```

✅ Dùng **" hoặc '** đều được.
✅ Dùng `"""` cho **chuỗi nhiều dòng**.

#### Một số hàm xử lý:

```python
text = "Python For Developers"
print(text.lower())  # python for developers
print(text.upper())  # PYTHON FOR DEVELOPERS
print(text.replace("Developers", "Everyone"))
```

🧠 **Ứng dụng thực tế:** Làm sạch text trong NLP, đổi format, xử lý log.

---

### 2.2. List – Danh sách có thứ tự

```python
prices = [10, 20, 30, 15, 25]
print(prices[0])     # 10
print(prices[-1])    # 25
```

| Phép thao tác | Ví dụ              | Kết quả         |
| ------------- | ------------------ | --------------- |
| Cắt list      | `prices[:3]`       | `[10,20,30]`    |
| Cắt có bước   | `prices[::2]`      | `[10,30,25]`    |
| Duyệt list    | `for p in prices:` | In từng phần tử |

✅ List **mutable** (có thể thay đổi giá trị).

---

### 2.3. Dictionary – Từ điển key-value

```python
products = {"A1": 10, "B2": 25, "C3": 40}
print(products["B2"])  # 25
```

#### Thao tác:

```python
products["D4"] = 50       # Thêm phần tử
products["A1"] = 15       # Cập nhật
print(products.keys())    # dict_keys(['A1', 'B2', 'C3', 'D4'])
print(products.values())  # dict_values([15,25,40,50])
```

✅ Dùng trong AI cho lookup nhanh hoặc lưu config model.

---

### 2.4. Set – Tập hợp không trùng lặp

```python
ids = {"A", "B", "C", "A"}
print(ids)  # {'A', 'B', 'C'}
```

* Không có thứ tự
* Không trùng lặp
* Không truy cập bằng chỉ số

Dùng `sorted(ids)` để sắp xếp → trả về list.

---

### 2.5. Tuple – Dữ liệu bất biến

```python
coords = (10, 20, 30)
print(coords[1])  # 20
```

✅ Không thể thay đổi (immutable).
📘 Thường dùng để lưu dữ liệu cố định (tọa độ, cấu hình...).

---

### 2.6. Tóm tắt cấu trúc dữ liệu

| Kiểu  | Mutable | Duplicates | Ordered | Subset   |
| ----- | ------- | ---------- | ------- | -------- |
| List  | ✅       | ✅          | ✅       | ✅        |
| Dict  | ✅       | ✅          | ✅       | Theo key |
| Set   | ✅       | ❌          | ❌       | ❌        |
| Tuple | ❌       | ✅          | ✅       | ✅        |

---

## 🔁 **CHƯƠNG 3 – CẤU TRÚC ĐIỀU KIỆN & VÒNG LẶP**

---

### 3.1. Câu lệnh điều kiện (if / elif / else)

```python
sales_target = 350
units_sold = 325

if units_sold >= sales_target:
    print("Target achieved")
elif units_sold >= 320:
    print("Almost achieved")
else:
    print("Target not achieved")
```

🧠 **Indentation cực kỳ quan trọng** – sai thụt dòng sẽ lỗi ngay!

---

### 3.2. Toán tử so sánh

| Toán tử | Nghĩa         |
| ------- | ------------- |
| `==`    | Bằng          |
| `!=`    | Khác          |
| `>`     | Lớn hơn       |
| `<`     | Nhỏ hơn       |
| `>=`    | Lớn hoặc bằng |
| `<=`    | Nhỏ hoặc bằng |

---

### 3.3. Vòng lặp `for`

```python
prices = [9.99, 8.99, 35.25, 1.50, 5.75]

for price in prices:
    if price > 10:
        print("More than $10")
    elif price < 5:
        print("Less than $5")
    else:
        print(price)
```

---

### 3.4. Vòng lặp `while`

```python
stock = 10
purchases = 0

while purchases < stock:
    purchases += 1
    print("Remaining:", stock - purchases)
```

💡 Dừng vô hạn → dùng `break`.

---

### 3.5. Từ khóa logic

| Keyword | Chức năng          | Ví dụ              |
| ------- | ------------------ | ------------------ |
| `and`   | Cả hai đúng        | `x > 5 and y < 10` |
| `or`    | Một trong hai đúng | `x > 5 or y < 10`  |
| `not`   | Phủ định           | `not x == 5`       |
| `in`    | Kiểm tra tồn tại   | `'A' in ['A','B']` |

---

### 3.6. Workflow kết hợp điều kiện + vòng lặp

```python
expensive = []
products = {"A": 10, "B": 25, "C": 40}

for key, value in products.items():
    if value > 20:
        expensive.append(key)

print(expensive)  # ['B', 'C']
```

---

### 3.7. Range() và counter

```python
count = 0
for i in range(1, 6):
    count += 1
print(count)  # 5
```

---

## 🚀 **CHƯƠNG 4 – MỞ RỘNG & PHỎNG VẤN**

---

### 4.1. Các hàm Python thường gặp trong phỏng vấn

| Hàm               | Mục đích         | Ví dụ                        |
| ----------------- | ---------------- | ---------------------------- |
| `len()`           | Đếm số phần tử   | `len([1,2,3])`               |
| `sum()`           | Tổng             | `sum([1,2,3])`               |
| `min()` / `max()` | Giá trị nhỏ/lớn  | `min([1,5,2])`               |
| `sorted()`        | Sắp xếp          | `sorted(set)`                |
| `enumerate()`     | Duyệt kèm chỉ số | `for i,v in enumerate(list)` |
| `zip()`           | Kết hợp 2 list   | `zip(names, ages)`           |

---

### 4.2. Mẹo code chuyên nghiệp

```python
# List comprehension
squares = [x**2 for x in range(5)]
# Dict comprehension
prices = {"A": 10, "B": 20}
discount = {k: v*0.9 for k, v in prices.items()}
```

---

### 4.3. Câu hỏi phỏng vấn mẫu

| Chủ đề          | Câu hỏi                             | Gợi ý trả lời                           |
| --------------- | ----------------------------------- | --------------------------------------- |
| Python Basics   | Khác list và tuple?                 | List mutable, tuple immutable           |
| Data Structures | Dùng dict khi nào?                  | Khi cần tra cứu nhanh theo key          |
| Control Flow    | `if` và `elif` khác gì?             | `elif` chỉ chạy khi điều kiện trước sai |
| Loop            | Khi nào dùng `while` thay vì `for`? | Khi chưa biết trước số lần lặp          |
| Logic           | `and` vs `or`?                      | `and` yêu cầu cả hai điều kiện đúng     |

---

### 4.4. Mini Project luyện tập

✅ **Product Filter Tool**

```python
products = {"A": 10, "B": 25, "C": 40}
filtered = {k: v for k, v in products.items() if v > 20}
print(filtered)
```

✅ **Even Number Counter**

```python
numbers = [1, 2, 3, 4, 5, 6]
even = [n for n in numbers if n % 2 == 0]
print(even)
```

---
