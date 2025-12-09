## 1. Core code vs Infrastructure code trong Spring Boot

### Core code là gì trong Java/Spring?

Theo ebook, core code là phần có thể chạy chỉ với CPU + RAM, **không cần DB, không cần network, không cần framework**. 

Trong Spring Boot, core code thường là:

* Các **domain model**: `Order`, `Product`, `Customer`, `Money`, `EmailAddress`…
* **Domain service / application service thuần Java**: không dùng `EntityManager`, không gọi `RestTemplate/WebClient`, không đọc file, không log ra ngoài, v.v.
* Các **interface** trừu tượng cho hạ tầng: `OrderRepository`, `PaymentGateway`, `MailSender`, `Clock`…

Đặc điểm:

* Không (hoặc gần như không) có annotation Spring (`@Entity`, `@Repository`, `@RestController`, `@Autowired`…).
* Có thể đặt trong 1 module riêng `core` hoặc package `com.yourapp.core` và **build thành JAR độc lập** vẫn chạy test được.

Ví dụ core entity:

```java
public final class Order {
    private final Long ebookId;
    private final String email;
    private final int quantity;
    private final int pricePerUnit;
    private final int totalAmount;

    public Order(Long ebookId, String email, int quantity, int pricePerUnit) {
        if (ebookId == null || ebookId <= 0) throw new IllegalArgumentException("ebookId > 0");
        if (!email.matches("^[^@]+@[^@]+$")) throw new IllegalArgumentException("Email invalid");
        if (quantity <= 0) throw new IllegalArgumentException("Quantity > 0");
        if (pricePerUnit < 0) throw new IllegalArgumentException("Price invalid");

        this.ebookId = ebookId;
        this.email = email;
        this.quantity = quantity;
        this.pricePerUnit = pricePerUnit;
        this.totalAmount = quantity * pricePerUnit;
    }

    // getters…
}
```

Interface repository (core):

```java
public interface OrderRepository {
    Long save(Order order);
    Order getById(Long id);
}
```

Ở đây không hề có Spring, JPA, HTTP gì cả → đúng nghĩa core.

---

### Infrastructure code là gì trong Spring?

Ebook định nghĩa: bất cứ thứ gì không thỏa điều kiện core (cần hệ thống bên ngoài, cần môi trường đặc biệt) là infrastructure. 

Trong Spring Boot, infrastructure thường là:

* `@RestController`, `@Controller`, `@Filter` → phụ thuộc Servlet container, HTTP.
* `@Repository`, class dùng `EntityManager`, Spring Data JPA.
* Adapter gửi mail (`JavaMailSender`), call HTTP (`WebClient`, `RestTemplate`), Kafka, RabbitMQ, S3…
* Cấu hình Spring, `@Configuration`, `@Bean`, `application.yml`…

Ví dụ implement repository bằng Spring Data JPA:

```java
@Entity
@Table(name = "orders")
class OrderEntity {
    @Id @GeneratedValue
    private Long id;
    private Long ebookId;
    private String email;
    private int quantity;
    private int amount;
    // getters/setters
}

interface SpringDataOrderJpa extends JpaRepository<OrderEntity, Long> {
}
```

Adapter triển khai `OrderRepository` (hạ tầng):

```java
@Component
class JpaOrderRepository implements OrderRepository {

    private final SpringDataOrderJpa jpa;

    JpaOrderRepository(SpringDataOrderJpa jpa) {
        this.jpa = jpa;
    }

    @Override
    public Long save(Order order) {
        OrderEntity e = new OrderEntity();
        e.setEbookId(order.getEbookId());
        e.setEmail(order.getEmail());
        e.setQuantity(order.getQuantity());
        e.setAmount(order.getTotalAmount());
        return jpa.save(e).getId();
    }

    @Override
    public Order getById(Long id) {
        OrderEntity e = jpa.findById(id)
                .orElseThrow(() -> new RuntimeException("Order not found"));
        int pricePerUnit = e.getAmount() / e.getQuantity();
        return new Order(e.getEbookId(), e.getEmail(), e.getQuantity(), pricePerUnit);
    }
}
```

Đây rõ ràng là infra: cần DB, Spring, JPA mới chạy.

---

## 2. Tại sao không nên trộn Core + Infrastructure trong Spring Boot?

Ebook liệt kê một loạt vấn đề khi trộn.  Ta map sang Spring Boot:

1. **Khó đọc logic nghiệp vụ**

   * Anti-pattern rất hay gặp: `@RestController` vừa validate, vừa tính toán nghiệp vụ, vừa gọi JPA, vừa map DTO.
   * Khi bạn đọc method controller 200 dòng, lẫn lộn `@Valid`, `orderRepository.save`, `EntityManager`, `BigDecimal`… rất khó nhìn ra “câu chuyện nghiệp vụ”.

2. **Coupling chặt với công nghệ**

   * Nếu logic `OrderService` dùng thẳng `JpaRepository` hoặc `EntityManager`, mai mốt đổi sang MongoDB, JDBC thuần, hoặc multi-tenancy sẽ rất đau.
   * Ebook nhấn mạnh: đổi DB, đổi framework sẽ phải sửa logic khắp nơi nếu không tách. 

3. **Khó test (TDD gần như bất khả thi)**

   * Service gọi thẳng JPA, WebClient, `SecurityContextHolder`, `HttpServletRequest` → phải spin cả Spring context + DB nhúng mới test được, test chậm và flaky. 
   * Nếu tách core: unit test chạy 5–10ms, fake repo in-memory, không cần Spring Boot.

4. **Dữ liệu không nhất quán**

   * Khi business rule nằm tản mát ở controller, service, repository, validator… sẽ rất dễ quên validate ở một đường luồng nào đó → lưu bậy vào DB. Ebook nói rõ chuyện này. 

5. **Bảo trì cực khổ**

   * Thay đổi schema hay business rule phải grep và sửa ở nhiều chỗ.
   * Kiểu code smell “transaction script everywhere” mà ebook mô tả. 

---

## 3. Layered Architecture trong Spring Boot (Domain / Application / Infrastructure)

Ebook đề xuất kiến trúc 3 lớp: Domain – Application – Infrastructure. 
Ta map sang Spring Boot:

### 3.1 Domain layer (core nhất)

* Thuần Java: `com.yourapp.domain.*`
* Gồm:

  * Entity/Aggregate: `Order`, `Ebook`, `Cart`, `Money`…
  * Value Object: `EmailAddress`, `OrderId`, `EbookId`…
  * Domain Service (nếu logic không thuộc riêng 1 entity).

> Không dùng Spring, không JPA, không log framework, không DB.

### 3.2 Application layer (core nhưng gần use case)

* `com.yourapp.application.*`
* Gồm:

  * Application service / use case: `PlaceOrderService`, `RegisterCustomerService`…
  * Interface của outbound ports: `OrderRepository`, `EbookPriceProvider`, `MailSender`…

Đây là nơi orchestrate: gọi domain, gọi repo (qua interface), commit transaction (conceptually).

Có thể để thuần Java, hoặc chấp nhận một chút annotation như `@Transactional` / `@Service` nhưng cố giữ không dính hạ tầng (ko dùng `RestTemplate`, `EntityManager` trực tiếp).

### 3.3 Infrastructure layer

* `com.yourapp.adapter.*` hoặc `com.yourapp.infrastructure.*`
* Chia nhỏ:

  * `adapter.web`: `@RestController`, `@ControllerAdvice`, DTO mapper.
  * `adapter.persistence`: JPA entity, repository implement, Flyway scripts.
  * `adapter.messaging`: Kafka listener/producer.
  * `adapter.external`: call third party APIs.

Các adapter **implements** port (interface) ở Application/Domain.

Dependency 1 chiều:

```text
domain  <-  application  <-  infrastructure
```

Infra phụ thuộc ngược vào interface ở core, chứ không phải core phụ thuộc infra – đúng tinh thần DIP mà ebook nhắc. 

---

## 4. Hexagonal Architecture (Ports & Adapters) trong Spring Boot

Ebook giải khá kỹ về Hexagonal: Port = cổng giao tiếp, Adapter = bộ chuyển đổi cụ thể (web, DB, external service…). 

Trong Spring Boot:

### 4.1 Primary port (input)

* Là **API nghiệp vụ** mà thế giới bên ngoài gọi vào:

  * Interface `PlaceOrderUseCase` trong application layer:

```java
public interface PlaceOrderUseCase {
    Long placeOrder(PlaceOrderCommand command);
}
```

* Primary adapter:

  * `@RestController`:

```java
@RestController
@RequestMapping("/orders")
class OrderController {

    private final PlaceOrderUseCase placeOrder;

    OrderController(PlaceOrderUseCase placeOrder) {
        this.placeOrder = placeOrder;
    }

    @PostMapping
    public ResponseEntity<?> create(@RequestBody PlaceOrderRequest req) {
        Long id = placeOrder.placeOrder(
                new PlaceOrderCommand(req.getEbookId(), req.getEmail(), req.getQuantity())
        );
        return ResponseEntity.ok(new OrderCreatedResponse(id));
    }
}
```

Controller chỉ map JSON ↔ command, không xử lý nghiệp vụ.

### 4.2 Secondary port (output)

* Là interface mà core cần để “nói chuyện” ra ngoài:

  * `OrderRepository`, `MailSender`, `PaymentGateway`, `Clock`, `IdGenerator`… 

* Secondary adapter: `JpaOrderRepository`, `SmtpMailSender`, `StripePaymentGateway`, `SystemClock`… dùng Spring + lib cụ thể.

Khi đổi DB hoặc đổi cổng thanh toán, bạn chỉ viết adapter khác, không đụng domain + application.

---

## 5. Domain Model & Repository Pattern trong Spring Boot

Ebook dành hẳn 1 chương nói về Domain Model và Repository, mình đã minh họa ở trên bằng Java.

Tóm quy tắc áp dụng vào Spring Boot:

1. **Domain entity không phải JPA entity**

   * Tách lớp `Order` (domain) khỏi `OrderEntity` (JPA).
   * Tránh lạm dụng Active Record kiểu `orderEntity.place()` chứa luôn logic domain – nó vừa phụ thuộc JPA vừa khó test.

2. **Repository interface nằm ở core**

   * `OrderRepository` trong module core, không dùng Spring Data annotation.

3. **Repository implementation nằm ở infra**

   * `JpaOrderRepository` dùng `OrderEntity` + `SpringDataOrderJpa` để map sang domain.

4. **Đặt business rule vào Domain / Application, không đặt lẫn trong `@Repository`**

   * `@Repository` chỉ nên map data, không chứa if/else nghiệp vụ.

---

## 6. Read Model / View Model trong Spring Boot

Ebook cảnh báo: đừng lạm dụng domain entity cho mục đích đọc (UI/report), dễ phình to, lộ quá nhiều behavior, và coupling. 

Trong Spring Boot:

* **Write model**: Domain entity + repository → dùng cho command (tạo/sửa/xoá).
* **Read model**: DTO, projection, view model → dùng cho query/hiển thị.

Ví dụ:

```java
public record OrderSummaryView(
        Long id,
        String email,
        int quantity,
        int totalAmount
) {}
```

Read repository:

```java
public interface OrderReadRepository {
    List<OrderSummaryView> findLatestOrders();
}
```

Implementation ở infra có thể dùng:

* Spring Data projection:

```java
interface OrderSummaryProjection {
    Long getId();
    String getEmail();
    int getQuantity();
    int getAmount();
}

interface SpringDataOrderReadJpa extends JpaRepository<OrderEntity, Long> {
    @Query("select o.id as id, o.email as email, o.quantity as quantity, o.amount as amount from OrderEntity o order by o.id desc")
    List<OrderSummaryProjection> findLatest();
}
```

Adapter map sang view:

```java
@Component
class JpaOrderReadRepository implements OrderReadRepository {

    private final SpringDataOrderReadJpa jpa;

    JpaOrderReadRepository(SpringDataOrderReadJpa jpa) {
        this.jpa = jpa;
    }

    @Override
    public List<OrderSummaryView> findLatestOrders() {
        return jpa.findLatest().stream()
                .map(p -> new OrderSummaryView(
                        p.getId(), p.getEmail(), p.getQuantity(), p.getAmount()))
                .toList();
    }
}
```

Controller trả ra `OrderSummaryView` cho FE, không expose domain entity.

---

## 7. Refactor ví dụ “đặt hàng e-book” sang Spring Boot

Ebook có ví dụ controller Laravel trộn đủ thứ logic: query DB, tính tiền, insert, set session. 
Ta làm bản “bẩn” tương đương trong Spring Boot:

```java
@RestController
class DirtyOrderController {

    @Autowired
    private JdbcTemplate jdbc;

    @PostMapping("/orders")
    public ResponseEntity<?> orderEbook(@RequestBody OrderRequest req, HttpSession session) {
        Long ebookId = req.getEbookId();
        Integer price = jdbc.queryForObject(
                "select price from ebooks where id = ?",
                Integer.class, ebookId
        );

        int quantity = req.getQuantity();
        int amount = price * quantity;

        KeyHolder kh = new GeneratedKeyHolder();
        jdbc.update(con -> {
            PreparedStatement ps = con.prepareStatement(
                    "insert into orders(email, quantity, amount) values(?,?,?)",
                    Statement.RETURN_GENERATED_KEYS);
            ps.setString(1, req.getEmail());
            ps.setInt(2, quantity);
            ps.setInt(3, amount);
            return ps;
        }, kh);

        Long orderId = kh.getKey().longValue();
        session.setAttribute("currentOrderId", orderId);
        return ResponseEntity.ok(orderId);
    }
}
```

Refactor theo ebook:

1. **Domain**: `Order` (đã viết ở trên).
2. **Port**: `OrderRepository`, `EbookPriceProvider`…
3. **Application service**:

```java
public record PlaceOrderCommand(Long ebookId, String email, int quantity) {}

@Service // hoặc để thuần Java rồi có adapter gọi
class PlaceOrderService implements PlaceOrderUseCase {

    private final EbookPriceProvider priceProvider;
    private final OrderRepository orderRepository;

    PlaceOrderService(EbookPriceProvider priceProvider, OrderRepository orderRepository) {
        this.priceProvider = priceProvider;
        this.orderRepository = orderRepository;
    }

    @Override
    public Long placeOrder(PlaceOrderCommand cmd) {
        int price = priceProvider.getPrice(cmd.ebookId());
        Order order = new Order(cmd.ebookId(), cmd.email(), cmd.quantity(), price);
        return orderRepository.save(order);
    }
}
```

4. **Infra – price provider & repo** như đã minh họa (JPA, JDBC…).

5. **Controller sạch**:

```java
@RestController
@RequestMapping("/orders")
class OrderController {

    private final PlaceOrderUseCase placeOrder;

    OrderController(PlaceOrderUseCase placeOrder) {
        this.placeOrder = placeOrder;
    }

    @PostMapping
    public ResponseEntity<OrderCreatedResponse> create(@RequestBody PlaceOrderRequest req,
                                                       HttpSession session) {
        Long orderId = placeOrder.placeOrder(
                new PlaceOrderCommand(req.getEbookId(), req.getEmail(), req.getQuantity())
        );
        session.setAttribute("currentOrderId", orderId);
        return ResponseEntity.ok(new OrderCreatedResponse(orderId));
    }
}
```

Giờ:

* Controller không biết gì về DB.
* `PlaceOrderService` không biết gì về HTTP, session, JPA.
* Repository và Provider là adapter hạ tầng.

Đây chính là kết quả mà ebook hướng tới khi refactor ví dụ Laravel sang Domain Model + Repository. 

---

## 8. Cách tổ chức project Spring Boot theo ebook

Nếu bạn dùng Maven/Gradle multi-module, có thể chia:

```text
your-app
├── core-domain          (Java thuần: entity, value object, domain services)
├── core-application     (use case, port interface)
├── adapter-persistence  (Spring, JPA, Flyway… implements ports)
├── adapter-web          (Spring MVC / WebFlux controllers, DTO)
└── boot-app             (Spring Boot main @SpringBootApplication, wiring beans)
```

Nếu một module thì tổ chức theo package:

```text
com.yourapp
  ├─ domain
  ├─ application
  └─ infrastructure
       ├─ web
       ├─ persistence
       └─ messaging
```

Nguyên tắc giống ebook: **core không import gì từ infra; infra import core**. 

---

## 9. Checklist nhanh áp dụng vào dự án Spring Boot

Lấy tinh thần ebook và đổi sang checklist:

1. Mỗi khi viết business logic mới:

   * Có thể test bằng unit test **mà không cần Spring context hay DB** không?
   * Nếu không → bạn đang trộn infra.

2. Controller:

   * Chỉ validate + map request/response + gọi use case.
   * Không thao tác JPA, không viết SQL, không chứa if/else business.

3. Service:

   * Chỉ dùng **ports (interface)**: `OrderRepository`, `MailSender`…
   * Không dùng trực tiếp `EntityManager`, `RestTemplate`, `JavaMailSender`, `JdbcTemplate`.

4. Domain:

   * Entity/VO đảm bảo invariant ngay trong constructor / method (như Order trong ebook). 

5. Repository:

   * Interface ở core, implementation ở infra.
   * Không nhét business rule vào `@Repository`.

6. Read model:

   * Dùng DTO/projection riêng cho query phức tạp.
   * Không bắt UI tái sử dụng aggregate domain nặng nề.

Làm được những điều này là bạn đã áp dụng 90% tư tưởng trong ebook “Kiến trúc ứng dụng web” vào Java Spring Boot rồi.

---
