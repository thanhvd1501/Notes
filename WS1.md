**Sinh lịch trình du lịch theo ràng buộc thực tế (ngân sách, sở thích, thời gian, nhịp độ) chỉ từ một file JSON — xuất ra Markdown dễ in, có validator logic để tránh lịch “ảo tưởng”.**

---

# 2) Vấn đề & khoảng trống thị trường (Problem → Gap)

* Lịch trình trên blog/Itinerary mẫu **thiếu cá nhân hoá**: ngân sách, khẩu vị, lịch di chuyển không trùng nhu cầu.
* Planner thủ công **mất thời gian** (search, ráp lịch, kiểm tra trùng/đi quá xa, canh bữa/giờ mở cửa).
* Nhiều tool AI hiện có **sinh nội dung “đẹp” nhưng thiếu kiểm tra logic** (slot chồng lấn, di chuyển 5′ giữa hai nơi cách xa 10km).
* Người dùng cần **MVP chạy tại chỗ** (không phụ thuộc web service), đầu vào **rõ ràng, có cấu trúc** → dễ tích hợp quy trình nội bộ.

**Khoảng trống**: Một CLI gọn, **đầu vào có cấu trúc** → **đầu ra chuẩn in**, có **validator** kiểm các lỗi thời gian cơ bản, cho **hai phương án** (Budget vs Comfort) để lựa chọn nhanh.

---

# 3) Ai sử dụng? (Target Users)

* **Cá nhân/nhóm nhỏ**: tự set trip ngắn ngày (2–5 ngày), muốn tối ưu thời gian/chi phí.
* **Travel ops/agent nhỏ**: cần tạo nháp lịch trình theo yêu cầu khách, sau đó tinh chỉnh thủ công.
* **Marketing team**: xuất nhanh “itinerary card” cho chiến dịch nội dung.

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

# 6) Kiến trúc & luồng dữ liệu (Architecture & Flow)

**CLI** → đọc `trip.json` → **Prompt Builder** (đưa ràng buộc & schema JSON) → **Azure OpenAI (Deployment)** → nhận JSON lịch trình
→ **Validator** (Python) → **Renderer** → `itinerary.md` (+ notes & packing).

```
[trip.json] --> [CLI] --> [Prompt Builder] --> [Azure OpenAI]
                                     |--> JSON Plan
JSON Plan --> [Validator Rules] --> [Markdown Renderer] --> itinerary.md
               (cảnh báo)                 (2 phương án + notes + packing)
```

* Công nghệ: Python 3.10+, `openai` (AzureOpenAI client), `python-dotenv`.
* Triển khai: cục bộ; không lưu dữ liệu người dùng ra server bên thứ ba ngoài Azure OpenAI.

---

# 7) Điểm khác biệt (Differentiators)

* **Ràng buộc cứng** ngay trong prompt (schema JSON + hướng dẫn slot tuần tự).
* **Validator** hậu kỳ: loại lỗi “lịch thần tốc”, “slot chồng lấn”.
* **Hai phương án** cho cùng input → giúp ra quyết định nhanh.
* **Markdown dễ in** → chia sẻ/duyệt nhanh; không cần app UI phức tạp.

---

# 8) Demo kịch bản (2–3 phút)

1. Mở `trip.json` (Đà Nẵng 3N2Đ) → chỉ các trường chính (dates, amount, interests, must_do).
2. Chạy:

   ```bash
   python itinerary-generate.py --trip trip.json --out itinerary.md --title "Đà Nẵng 3N2Đ"
   ```
3. Mở `itinerary.md`:

   * Xem **Phương án A — Budget** (nhiều đi bộ/công cộng, chi phí thấp hơn).
   * Xem **Phương án B — Comfort** (taxi, nhịp độ thoải mái).
   * Cuộn đến **Ghi chú kiểm tra** (ví dụ cảnh báo ngày >15h, slot ngắn bất thường).
   * **Packing list** gợi ý (rule-based, dựa trên interests/weather_notes).

> **Plan B (nếu mạng/endpoint lỗi):** Dùng `trip.json` demo & show sẵn một `itinerary.md` đã tạo trước; sau đó giải thích kiến trúc & mã nguồn.

---

# 9) Mô-đun & mã nguồn (High-level Code)

* `itinerary-generate.py` (single-file):

  * **ENV/Client**: `AzureOpenAI(api_version, azure_endpoint, api_key, deployment)`.
  * **PROMPTS (VN)**: `SYSTEM_PLANNER`, `USER_TEMPLATE` (schema JSON + ràng buộc).
  * **build_plan(mode)**: gọi API, ép JSON an toàn.
  * **validate_itinerary(plan)**: luật thời gian, chi phí.
  * **render_markdown**: xuất 2 phương án + notes + packing list.
* `.env.example`: khai báo `AZURE_OPENAI_ENDPOINT | KEY | DEPLOYMENT | API_VERSION`.

---

# 10) Rủi ro & phương án giảm thiểu (Risks & Mitigations)

* **Model đôi khi trả về JSON lỗi format** → trích JSON robust + retry thủ công.
* **Thông tin “giờ mở cửa/đi lại”** không realtime → trình bày là **ước tính**; người dùng vẫn cần check lại.
* **Giới hạn token** → cắt text, không nhét dữ liệu thừa; chỉ một file JSON gọn.
* **Độ trễ mạng/500 server** → dùng `AzureOpenAI` (đúng api_version), có script test kết nối; chuẩn bị sẵn output dự phòng.

---

# 11) Riêng tư & đạo đức (Privacy & Ethics)

* Không lưu vị trí cụ thể/ID người dùng; dữ liệu ở máy người dùng, chỉ gửi **nội dung tối thiểu** lên Azure để tạo lịch.
* Không khẳng định “chính xác tuyệt đối” chi phí/giờ giấc; luôn ghi chú “ước tính, cần kiểm tra thực tế”.

---

# 12) Đo lường thành công (KPIs)

* **Precision vận hành**: % lịch trình **không trùng slot**, **không WARNING nghiêm trọng**.
* **Hữu ích**: % người dùng chọn được 1 trong 2 phương án sau 1 lần chạy.
* **Tốc độ**: 1–2 phút để sinh lịch 3 ngày.
* **Chỉnh sửa thủ công ít**: trung bình < 5 thay đổi/itinerary.

---

# 13) Lộ trình 5 tuần (Roadmap)

* **W1 – Foundation**: Schema `trip.json`, client AzureOpenAI, prompt VN v1, chạy được Budget/Comfort.
* **W2 – Quality**: Chuẩn JSON, robust extract, Markdown renderer, packing list rule-based.
* **W3 – Dual Options**: Hoàn thiện khác biệt Budget/Comfort (chi phí/nhịp độ/di chuyển).
* **W4 – Validator**: Bổ sung luật thời gian, cảnh báo tổng; flags CLI; mẫu thành phố.
* **W5 – Ship**: README, bộ demo `trip.json` (BKK/SGN/HN/PQ), in-friendly layout, script test API.

---

# 14) Chi phí vận hành (Cost)

* Mỗi lần sinh **2 kế hoạch** (Budget & Comfort).
* Chi phí phụ thuộc model (ví dụ gpt-4o-mini) & độ dài prompt/response; với 3 ngày nội dung gọn → **rẻ** cho MVP.
* Có thể cache đầu ra theo `hash(trip.json)` để tránh gọi lặp.

---

# 15) Mở rộng tương lai (Next)

* **Song ngữ VN/EN** (`--lang`) & preset thành phố (POI gợi ý).
* **Tích hợp dữ liệu thời tiết/thời gian di chuyển** (API công khai) → điều chỉnh slot tự động.
* **Web UI nhẹ** (FastAPI/Next.js) + chia sẻ link.
* **Tối ưu “pacing” thông minh**: learn từ phản hồi người dùng (reinforcement-like heuristics).

---

# 16) Lệnh chạy mẫu (đưa thẳng lên slide)

```bash
pip install openai python-dotenv
cp .env.example .env
# AZURE_OPENAI_ENDPOINT=https://<resource>.openai.azure.com/
# AZURE_OPENAI_API_KEY=<key>
# AZURE_OPENAI_DEPLOYMENT=<deployment>
# AZURE_OPENAI_API_VERSION=2024-08-01-preview

python itinerary-generate.py --trip trip.json --out itinerary.md --title "Đà Nẵng 3N2Đ"
```

---

# 17) Q&A Cheat-sheet

* **“Vì sao CLI, không Web?”** → Nhanh, nhẹ, demo offline; dễ nhúng pipeline nội bộ. Web là bước tiếp theo.
* **“Tính chính xác thời gian di chuyển?”** → Mô phỏng/ước tính; validator chỉ check logic hình thức. Có thể tích hợp API Maps ở bản nâng cao.
* **“Khác gì ChatGPT trả lời tự do?”** → Ràng buộc **schema JSON + validator** → lịch có cấu trúc, giảm “ảo tưởng”, xuất chuẩn in.
* **“Bảo mật?”** → Không lưu dữ liệu; chỉ gửi prompt tối thiểu tới Azure; người dùng giữ file cục bộ.

---
