
# 🔒 **Tóm tắt bài học về Authentication bằng JWT trong Spring Boot**

## **1. Giới thiệu**  
Trong bài trước, chúng ta đã tìm hiểu về cách **mã hóa mật khẩu người dùng** một cách an toàn và xác thực mật khẩu đó. Hôm nay, chúng ta tiếp tục với **cách sử dụng JSON Web Token (JWT) để xác thực người dùng trong hệ thống Identity Service**.

---

## **2. Token-Based Authentication là gì?**  
Trong hệ thống truyền thống, khi một client muốn xác thực với server, họ sẽ gửi **username và password** mỗi lần request. Điều này gây ra bất tiện và không an toàn. Vì vậy, **Token-Based Authentication** ra đời để thay thế.

🔹 **Ví dụ thực tế:**  
- Bạn là sinh viên và có **thẻ sinh viên**.
- Để vào thư viện, bạn cần một **thẻ thư viện** do trường cấp.
- Khi vào thư viện, bạn chỉ cần trình **thẻ thư viện**, không cần thẻ sinh viên nữa.

💡 **Áp dụng vào Authentication:**  
- Khi user đăng nhập, hệ thống sẽ cấp một **JWT Token**.
- Các request sau, user chỉ cần gửi **JWT Token** để xác thực.

---

## **3. JWT là gì?**  
**JWT (JSON Web Token)** là một **chuỗi JSON được mã hóa** chứa thông tin xác thực.  
JWT bao gồm **3 phần chính**:

### **1️⃣ Header**  
Chứa thông tin về loại token và thuật toán ký:  
```json
{
  "alg": "HS512",
  "typ": "JWT"
}
```

### **2️⃣ Payload (Body)**  
Chứa các **claims (dữ liệu xác thực)** như user, thời gian hết hạn:
```json
{
  "sub": "user123",
  "iss": "your-app.com",
  "iat": 1711042340,
  "exp": 1711045940,
  "role": "admin"
}
```

### **3️⃣ Signature (Chữ ký số)**  
Là **hash của Header + Payload + Secret Key**, đảm bảo token không bị thay đổi.
```
HMACSHA512( base64UrlEncode(header) + "." + base64UrlEncode(payload), secret )
```

➡ **Nếu ai đó thay đổi payload, chữ ký sẽ không khớp, token bị từ chối!**

---

## **4. Cách tạo JWT trong Spring Boot**  

### **📌 Bước 1: Thêm thư viện JWT vào `pom.xml`**  
```xml
<dependency>
    <groupId>com.nimbusds</groupId>
    <artifactId>nimbus-jose-jwt</artifactId>
    <version>9.31</version>
</dependency>
```

### **📌 Bước 2: Tạo Token**  
```java
public String generateToken(String username) {
    JWSSigner signer = new MACSigner(SECRET_KEY);
    
    JWTClaimsSet claimsSet = new JWTClaimsSet.Builder()
        .subject(username)
        .issuer("your-app.com")
        .issueTime(new Date())
        .expirationTime(new Date(System.currentTimeMillis() + 3600000)) // 1 giờ
        .claim("role", "user") // Thêm claim
        .build();

    SignedJWT signedJWT = new SignedJWT(
        new JWSHeader(JWSAlgorithm.HS512), claimsSet);

    signedJWT.sign(signer);

    return signedJWT.serialize(); // Trả về chuỗi JWT
}
```

### **📌 Bước 3: Verify Token**
```java
public boolean verifyToken(String token) {
    try {
        SignedJWT signedJWT = SignedJWT.parse(token);
        JWSVerifier verifier = new MACVerifier(SECRET_KEY);

        return signedJWT.verify(verifier) && 
               new Date().before(signedJWT.getJWTClaimsSet().getExpirationTime());
    } catch (Exception e) {
        return false;
    }
}
```

---

## **5. Cách sử dụng JWT trong Authentication Service**  

- Khi user đăng nhập thành công, **hệ thống cấp JWT Token** thay vì trả về `true/false`.
- User sử dụng JWT này trong **các request tiếp theo** bằng cách **gửi trong Header Authorization**:
```
Authorization: Bearer <jwt_token>
```

**📌 Ví dụ API Login trả về JWT**  
```java
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody AuthRequest request) {
    if (authenticationService.authenticate(request.getUsername(), request.getPassword())) {
        String token = jwtService.generateToken(request.getUsername());
        return ResponseEntity.ok(new AuthResponse(token, true));
    } else {
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
    }
}
```

---

## **6. Kiểm tra Token bằng JWT Debugger**  
Bạn có thể **giải mã JWT Token** bằng cách dán token vào trang web:  
🔗 [jwt.io](https://jwt.io)

- Header: **Thuật toán SHA512**
- Payload: **Thông tin user, thời gian hết hạn**
- Signature: **Chữ ký đảm bảo tính toàn vẹn**

➡ **Nếu có ai thay đổi payload, chữ ký sẽ không khớp, token bị từ chối!**

---

## **7. Cách bảo vệ Secret Key và nâng cao bảo mật**  

🔹 **Không lưu Secret Key trong code**, hãy đặt trong **application.properties**:
```properties
jwt.secret=YOUR_RANDOM_SECRET_KEY_256BIT
```
🔹 **Sử dụng xác thực bằng Public/Private Key (RSA)** để tăng tính bảo mật.

---

## **📌 Kết luận**  
✔ **JWT giúp xác thực user một cách bảo mật, không cần gửi username/password mỗi lần.**  
✔ **Có thể dễ dàng kiểm tra token mà không cần truy vấn database.**  
✔ **Chữ ký số đảm bảo token không thể bị chỉnh sửa.**  
✔ **Cần bảo vệ Secret Key và thiết lập thời gian hết hạn hợp lý.**  
