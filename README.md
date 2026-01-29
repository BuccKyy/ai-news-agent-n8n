
# 🤖 AI News Automation Agent (v9.0)

Chào mừng bạn đến với hệ thống tự động hóa tin tức AI toàn diện. Hệ thống này được thiết kế để hoạt động 24/7 trên nền tảng đám mây (Hugging Face / n8n), tự động thu thập, phân tích và gửi những tin tức AI quan trọng nhất trực tiếp đến Telegram của bạn.

---

## 🏗 Cấu Trúc Hoạt Động

Luồng xử lý của Workflow được chia thành 5 giai đoạn chính:

```mermaid
graph TD
    A[🕒 Schedule Trigger] -->|Mỗi 30 phút| B(📡 RSS Feeds Collection)
    B --> C{🧹 Smart Filter & Deduplication}
    C -->|Rác / Trùng lặp| X[🗑 Discard]
    C -->|Tin mới| D[🧠 AI Analysis (Groq/Llama3)]
    D --> E{🎯 Scoring & Classification}
    E -->|Điểm < 7| X
    E -->|Điểm >= 7| F[🚀 Telegram Notification]
```

---

## 🧩 Chi Tiết Các Thành Phần

### 1. Thu Thập (Collection Layer)
- **Nguồn tin**: Hệ thống quét hơn 20 nguồn tin uy tín nhất thế giới về AI:
  - **Blog Chính Chủ**: OpenAI, Google AI, DeepMind, Anthropic, Hugging Face.
  - **Học Thuật**: ArXiv (AI, CV, NLP, ML).
  - **Tạp Chí**: MIT Tech Review, TechCrunch AI, VentureBeat.
  - **YouTube**: Two Minute Papers, Yannic Kilcher, AI Explained (tự động nhận diện Video).

### 2. Bộ Lọc Thông Minh (Smart Timing & Clean)
- **Cơ chế**: Sử dụng Javascript để xử lý dữ liệu thô.
- **Tính năng**:
  - **Chống trùng lặp (Deduplication)**: Lưu lại các link đã gửi vào bộ nhớ tĩnh (Static Memory) để không bao giờ gửi lại tin cũ.
  - **Khung giờ hoạt động**: Chỉ xử lý tin tức từ **7:00 sáng đến 24:00 đêm**. Tin tức ban đêm sẽ được tích lại và xử lý vào sáng hôm sau (tránh làm phiền lúc ngủ).

### 3. Phân Tích AI (Intelligence Layer)
- **Mô hình**: Sử dụng **Llama-3-8b-instant** thông qua Groq API (tốc độ cực nhanh).
- **Nhiệm vụ**:
  - Đọc Title và Description của bài viết.
  - Tóm tắt nội dung sang tiếng Việt (2-3 câu).
  - Chấm điểm độ quan trọng (Score 1-10).
  - Phân loại chủ đề (LLM, Computer Vision, AI Tools...).
  - Đánh giá cảm xúc (Tích cực/Tiêu cực) và Tác động (Cao/Trung bình/Thấp).

### 4. Định Tuyến & Thông Báo (Notification Layer)
- **Bộ lọc điểm số**: Chỉ những bài viết có điểm **Score >= 7** (Tin quan trọng/Đột phá) mới được đi tiếp. Tin rác hoặc tin ít quan trọng sẽ bị loại bỏ.
- **Telegram Bot**:
  - Gửi tin nhắn được định dạng HTML đẹp mắt.
  - Kèm theo nút bấm: **📖 Đọc ngay**, **⭐ Lưu lại**, **❌ Bỏ qua**.
  - Cơ chế **Split In Batches**: Gửi từng tin một, cách nhau 15 giây để tránh bị Telegram chặn (Rate Limit).

---

## 🛠 Hướng Dẫn Cài Đặt & Sử Dụng

### 1. File Workflow
Trong thư mục này bạn sẽ thấy 2 file chính:
- **`My workflow 9.json`**: Bản gốc, dùng cho chạy Local (trên máy tính). Có chức năng lưu file CSV vào ổ cứng.
- **`My workflow 9_cloud.json`**: Bản tối ưu cho Cloud (Hugging Face). Đã loại bỏ các node ghi file CSV (vì trên cloud không có ổ cứng bền vững) để tránh lỗi. **Khuyên dùng bản này.**

### 2. Cách Import lên Cloud
1. Truy cập n8n của bạn: `https://[tên-space].hf.space`
2. Vào menu **Workflows** -> **Import from File**.
3. Chọn file **`My workflow 9_cloud.json`**.
4. Vào **Credentials** -> Kết nối lại **Telegram API** (nhập Bot Token mới).
5. Bật nút **Active** (Góc phải trên cùng).

### 3. Tùy Chỉnh (Nâng Cao)
Nếu bạn muốn thay đổi ngưỡng lọc tin (ví dụ chỉ nhận tin cực hot 9/10):
- Mở node **`Filter Score >= 7`**.
- Sửa số `7` thành `9`.

---

## 🗑 Ghi Chú Về Dọn Dẹp
Các file script Python (`.py`) và các file JSON cũ trong thư mục này là tàn dư của quá trình phát triển. Bạn có thể xóa chúng đi nếu muốn tiết kiệm dung lượng, chỉ cần giữ lại file JSON workflow và README này là đủ.

*Tài liệu được tạo tự động bởi trợ lý AI của bạn. Chúc bạn có những trải nghiệm tuyệt vời!*
