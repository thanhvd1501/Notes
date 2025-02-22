
# **Tóm tắt bài học về mã hóa mật khẩu trong Spring Boot với BCrypt**

## **1. Giới thiệu về mã hóa mật khẩu**
- Mật khẩu lưu dưới dạng **plain text** (chưa mã hóa) rất nguy hiểm, dễ bị hacker lấy cắp.
- Cách mã hóa mật khẩu phổ biến trước đây là sử dụng **hàm băm (Hash Function)** như **MD5**.
- MD5 có nhược điểm lớn là dễ bị **tấn công Rainbow Table** nên không còn an toàn.
- Thuật toán thay thế an toàn hơn là **BCrypt**, hiện được sử dụng rộng rãi trong hệ thống bảo mật.

## **2. Tổng quan về thuật toán băm (Hash Function)**
- Hàm băm nhận đầu vào và tạo ra một giá trị cố định (hash).
- Đặc điểm của hàm băm:
  - **Không thể đảo ngược** (không thể lấy lại mật khẩu gốc từ hash).
  - **Cùng đầu vào luôn cho cùng một đầu ra** (MD5 bị lộ dễ bị tấn công).
  - **Nhạy cảm với thay đổi nhỏ** (chỉ cần thay đổi một ký tự thì hash sẽ khác hoàn toàn).
- Vấn đề bảo mật:
  - Hash có thể bị dò tìm bằng **Rainbow Table** – danh sách hash đã được tính trước.
  - Giải pháp là thêm **Salt** (muối) vào trước khi băm để giảm nguy cơ bị tấn công.

## **3. Giới thiệu về BCrypt**
- **BCrypt** là thuật toán băm mật khẩu có sẵn trong **Spring Security**.
- Điểm đặc biệt:
  - BCrypt **tự động thêm Salt** vào chuỗi hash.
  - Cùng một mật khẩu nhưng mỗi lần mã hóa sẽ tạo ra một chuỗi hash khác nhau.
  - Hash chứa thông tin về phiên bản thuật toán, độ phức tạp, Salt và giá trị đã mã hóa.

## **4. Cách tích hợp BCrypt trong Spring Boot**

### **Bước 1: Thêm dependency**
Thêm thư viện **Spring Security Crypto** vào `pom.xml`:
```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-crypto</artifactId>
    <version>5.7.1</version>
</dependency>
```

### **Bước 2: Tạo `PasswordEncoder`**
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

### **Bước 3: Mã hóa mật khẩu trước khi lưu vào database**
```java
user.setPassword(passwordEncoder.encode(user.getPassword()));
userRepository.save(user);
```

### **Bước 4: Kiểm tra mật khẩu khi đăng nhập**
```java
boolean isPasswordMatch = passwordEncoder.matches(rawPassword, encodedPassword);
```

## **5. Cách xác thực mật khẩu khi user đăng nhập**

### **Tạo `AuthenticationService`**
```java
@Service
public class AuthenticationService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    public boolean authenticate(String username, String rawPassword) {
        User user = userRepository.findByUsername(username)
                      .orElseThrow(() -> new UsernameNotFoundException("User not found"));
        return passwordEncoder.matches(rawPassword, user.getPassword());
    }
}
```

### **Tạo API đăng nhập (`AuthenticationController`)**
```java
@RestController
@RequestMapping("/auth")
public class AuthenticationController {

    @Autowired
    private AuthenticationService authenticationService;

    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody AuthRequest request) {
        boolean isAuthenticated = authenticationService.authenticate(request.getUsername(), request.getPassword());
        return ResponseEntity.ok(new AuthResponse(isAuthenticated));
    }
}
```

## **6. Kiểm tra kết quả**
- Nếu đăng nhập với đúng username/password → Trả về `true`.
- Nếu sai mật khẩu → Trả về `false`.

## **7. Hướng đi tiếp theo**
- Hiện tại, hệ thống chỉ xác thực username/password trực tiếp.
- Thực tế, hệ thống API thường sử dụng **JWT Token** để xác thực, tránh gửi mật khẩu nhiều lần.
- Video tiếp theo sẽ hướng dẫn cách tạo **JWT Token**, ký điện tử và xác thực người dùng.

---

## **Kết luận**
- Sử dụng **BCryptPasswordEncoder** để bảo vệ mật khẩu an toàn.
- **Không bao giờ lưu plain text password** trong database.
- Dùng `passwordEncoder.matches()` để xác thực mật khẩu.
- Hướng dẫn tiếp theo sẽ tập trung vào **JWT Authentication**.

🔥 **Đừng quên Like & Subscribe để cập nhật video mới nhất nhé!** 🚀
