**Sinh lịch trình du lịch theo ràng buộc thực tế (ngân sách, sở thích, thời gian, nhịp độ) chỉ từ một file JSON — xuất ra Markdown.**

---

# 2) Vấn đề và thị trường hiện nay

* Lịch trình trên blog/Itinerary mẫu **thiếu cá nhân hoá**
* Planner thủ công **mất thời gian** (search, ráp lịch, kiểm tra trùng/đi quá xa, canh bữa/giờ mở cửa).
* Người dùng cần lịch trình chi tiết ví dụ như lộ trình, budget, ....

---

# 3) Mục tiêu hướng tới. Ai sử dụng?

* **Cá nhân/nhóm nhỏ**: tự set trip ngắn ngày (2–5 ngày), muốn tối ưu thời gian/chi phí.
* **Các bên cung cấp dịch vụ đặt tour**: cần tạo nháp lịch trình theo yêu cầu khách, sau đó tinh chỉnh thủ công.
---

# 4) Kịch bản sử dụng (User Story)

> “Tôi có 3 ngày ở Đà Nẵng, thích hải sản & view, ngân sách tầm 6 triệu cho 2 người, sợ lịch ‘tham quan chớp nhoáng’. Tôi muốn kế hoạch **theo ngày/giờ** và **2 lựa chọn**: rẻ & thoải mái. Tôi in ra để bàn với bạn đồng hành.”

---

# 5) Giải pháp (Solution Overview)

* **Đầu vào**: `trip.json` (city, ngày, budget, interests, exclusions, pace, dietary, must_do, ghi chú thời tiết).
* **Sinh lịch trình**: Azure OpenAI (Chat Completions) + **prompt tiếng Việt** đã ràng buộc **schema JSON** + hai mode:

  * **Budget** (ưu tiên đi bộ/công cộng, tối ưu chi phí)
  * **Comfort** (ưu tiên taxi/ride-hailing, nhịp độ thoải mái)
* **Validator**: luật thời gian cơ bản (tuần tự, không trùng slot, min time cho tham quan/di chuyển, phát hiện chi phí âm, cảnh báo ngày quá dài).
* **Đầu ra**: `itinerary.md` (bảng giờ/hoạt động/khu vực/ghi chú/chi phí), kèm **Ghi chú kiểm tra** & **Packing list** rule-based.

---
