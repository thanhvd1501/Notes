## Chức năng thi thử HSK

Có 2 role: **ADMIN (giáo viên)** và **USER (thí sinh)**.

### I. Admin / CMS

1. **Quản lý đề**

   * Tạo / sửa / xoá đề HSK (code, level, thời gian, mô tả).
   * Upload file audio (.mp3) gắn với phần Listening.
   * Upload ảnh câu hỏi.

2. **Quản lý cấu trúc đề**

   * Tạo các **section**: Nghe – Phần 1…4; Đọc – Phần 1…4.
   * Nhập **question** + **options**:

     * Câu nghe: có thể không hiển thị text, chỉ hiển thị hình + lựa chọn A/B/C.
     * Câu đọc: hiển thị text, hình, hoặc bảng.
   * Nhập **đáp án** dựa trên answer key.

3. **Import nhanh**

   * Một màn hình “Import” để seed DB từ script SQL/JSON.

4. **Xem kết quả & thống kê**

   * Danh sách lượt làm bài theo đề.
   * Xem chi tiết từng attempt, câu nào sai, tỉ lệ đúng từng câu.
   * Export CSV.

---

### II. Thí sinh

1. **Danh sách đề HSK**

   * Lọc theo level (HSK1, HSK2…).
   * Hiển thị thông tin: cấp độ, thời lượng, số câu.

2. **Màn hình “Giới thiệu đề”**

   * Hướng dẫn, thời lượng 40 phút, cấu trúc nghe/đọc.
   * Nút “Bắt đầu làm bài” → tạo `exam_attempt`.

3. **Thi Nghe (Listening)**

   * Giao diện player:

     * Phát file audio .
     * Hiển thị câu hỏi + hình tương ứng theo thứ tự.
   * Logic:

     * Cho learner tự nghe và chọn.
     * Advanced: dùng `audio_offset_ms` để auto highlight câu.

4. **Thi Đọc (Reading)**

   * Render layout:

     * Phần 1: bảng hình – từ (21–25).
     * Phần 2–4: dạng matching / chọn đáp án.
   * Mỗi câu là single choice: A–F…; bấm vào là lưu ngay `attempt_answer` (autosave).

5. **Timer & điều hướng**

   * Global countdown 40 phút.
   * Thanh progress (Câu 1/40).

6. **Nộp bài & chấm điểm**

   * Khi hết giờ hoặc user bấm “Nộp bài”:

     * Tính điểm từng câu dựa trên `question_answer`.
     * Ghi `total_score`, `listening_score`, `reading_score` vào `exam_attempt`.

7. **Xem kết quả & review**

   * Màn hình kết quả:

     * Tổng điểm, % đúng, phân tách nghe/đọc.
     * Danh sách câu:

       * Câu, đáp án đã chọn, đáp án đúng.
