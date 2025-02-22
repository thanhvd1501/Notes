
# 🔍 **Giải thích chi tiết về chuỗi BCrypt**

`BCrypt` là một thuật toán băm mật khẩu bảo mật cao, được thiết kế để **chống lại các cuộc tấn công brute-force** bằng cách sử dụng một cơ chế đặc biệt có tên là **Adaptive Hashing** (băm thích ứng). Khi bạn băm một mật khẩu bằng `BCrypt`, nó sẽ tạo ra một chuỗi (hash) có định dạng đặc biệt.

Ví dụ một chuỗi BCrypt:
```
$2a$10$7EqJtq98hPqEX7fNZaFWoOhi7h5EE2pIXFNSaZKbT/S5x/qG9uTq.
```

---

## **📌 Cấu trúc của chuỗi BCrypt**
Chuỗi hash của BCrypt thường có độ dài **60 ký tự** và được chia thành **4 phần chính**:

### **1️⃣ Prefix (Tiền tố) – Thuật toán được sử dụng**
```
$2a$
```
👉 Đây là **version (phiên bản) của thuật toán BCrypt**.

| Giá trị  | Ý nghĩa |
|----------|--------|
| `$2a$`   | Phiên bản gốc của BCrypt |
| `$2b$`   | Phiên bản cải tiến, sửa lỗi bảo mật trên BSD |
| `$2y$`   | Phiên bản dành riêng cho OpenBSD |
| `$2x$`   | Phiên bản có lỗi, không nên dùng |

> 💡 **Lưu ý**: Nếu không chỉ định gì, `Spring Security` sẽ dùng phiên bản `$2a$` hoặc `$2b$` tùy phiên bản Spring.

---

### **2️⃣ Cost Factor (Độ phức tạp – số vòng lặp)**
```
$10$
```
👉 Đây là số lần lặp (work factor), biểu thị độ mạnh của thuật toán.

- Giá trị này càng lớn thì **việc mã hóa càng chậm** nhưng cũng **càng khó bẻ khóa**.
- Nếu `cost = 10`, thì số vòng tính toán sẽ là **2¹⁰ = 1024 vòng lặp**.
- Nếu `cost = 12`, thì số vòng sẽ là **2¹² = 4096 vòng**, khiến việc bẻ khóa chậm hơn rất nhiều.

> 💡 **Gợi ý**:  
> - Nếu hệ thống cần bảo mật cao, đặt `cost = 12 - 14`.  
> - Nếu cần hiệu suất nhanh hơn, đặt `cost = 10 - 11`.

---

### **3️⃣ Salt (Muối) – 22 ký tự ngẫu nhiên**
```
7EqJtq98hPqEX7fNZaFWoO
```
👉 **Salt là một chuỗi ngẫu nhiên được tạo ra trong mỗi lần mã hóa**.

- Salt giúp tránh **Rainbow Table Attack** (tấn công bảng tra cứu băm).
- Mỗi lần băm, `BCrypt` sẽ **tạo một salt mới**, đảm bảo rằng **cùng một mật khẩu nhưng hash sẽ luôn khác nhau**.

---

### **4️⃣ Hashed Password (Chuỗi đã mã hóa)**
```
hi7h5EE2pIXFNSaZKbT/S5x/qG9uTq.
```
👉 Đây là phần chứa **mật khẩu đã băm**, được tạo bằng thuật toán `Blowfish`.

- Phần này có **31 ký tự**.
- Mật khẩu đã băm được tạo dựa trên:
  - Mật khẩu gốc (`raw password`).
  - Salt (muối).
  - Số vòng lặp (`cost factor`).

---

## **🔬 Ví dụ: Cách tạo chuỗi BCrypt trong Java**
```java
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

public class BcryptExample {
    public static void main(String[] args) {
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder(12); // Cost factor = 12
        String rawPassword = "mypassword";
        String hashedPassword = encoder.encode(rawPassword);

        System.out.println("Mật khẩu đã băm: " + hashedPassword);
    }
}
```
🔹 **Mỗi lần chạy sẽ ra một hash khác nhau**, vì `BCrypt` sinh `salt` ngẫu nhiên!

Ví dụ:
```
$2a$12$N2PZT1kTnI6p5vhZq3YQpeHkxdzOqHiv3Uklq6M2AsV7H5Y5E5HkG
$2a$12$84JHAc9H9Bk3I3qz/OHD3.4UlD/VB3wWMo9Ar.XplEUVzHZTnsFZW
```

✅ **Dù cùng mật khẩu "mypassword", nhưng hash mỗi lần sinh ra đều khác!**

---

## **🔄 Kiểm tra mật khẩu với BCrypt**
```java
String rawPassword = "mypassword";
boolean isMatch = encoder.matches(rawPassword, hashedPassword);
System.out.println("Mật khẩu đúng không? " + isMatch);
```
📌 **Cách hoạt động:**
- `matches()` sẽ lấy mật khẩu nhập vào, **trích xuất `Salt` từ hash cũ** và băm lại mật khẩu với Salt đó.
- Nếu kết quả băm mới **trùng** với hash đã lưu, nghĩa là mật khẩu đúng.

---

## **📌 Tại sao BCrypt an toàn hơn MD5, SHA-256?**
| Thuật toán  | Tốc độ băm | Có dùng Salt? | Dễ bị tấn công Rainbow Table? | Có bảo vệ chống Brute Force? |
|------------|-----------|---------------|------------------------------|-----------------------------|
| **MD5**    | Rất nhanh ⚡ | ❌ Không      | ✅ Dễ bị tấn công            | ❌ Không                     |
| **SHA-256**| Nhanh ⚡   | ❌ Không      | ✅ Dễ bị tấn công            | ❌ Không                     |
| **BCrypt** | Chậm 🛡️   | ✅ Có         | ❌ Không                      | ✅ Có bảo vệ (Cost Factor)   |

> **📌 BCrypt có 2 tính năng bảo mật quan trọng:**
> 1. **Adaptive Hashing (Thích ứng)** – Có thể tăng `Cost Factor` để tăng số vòng băm, làm hacker mất nhiều thời gian hơn.
> 2. **Built-in Salt (Muối sẵn có)** – Mỗi lần băm luôn khác nhau, giúp chống lại tấn công Rainbow Table.

---

## **📝 Kết luận**
✔ **BCrypt tạo ra một chuỗi 60 ký tự bảo mật cao, chứa đầy đủ thông tin về thuật toán, salt và mật khẩu băm.**  
✔ **Cùng một mật khẩu nhưng hash luôn khác nhau, giúp bảo vệ chống lại tấn công Rainbow Table.**  
✔ **Số vòng lặp (`Cost Factor`) có thể điều chỉnh để làm tăng độ khó khi brute-force.**  
✔ **Spring Security hỗ trợ `BCryptPasswordEncoder`, giúp dễ dàng tích hợp vào ứng dụng.**

🔐 **➡ Vì vậy, BCrypt là lựa chọn tốt nhất để mã hóa mật khẩu trong hệ thống hiện đại!** 🚀
