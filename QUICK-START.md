# ⚡ Quick Start Checklist - Tech News AI Agent

## ✅ 5 Bước Setup Nhanh (15 phút)

### 🔹 BƯỚC 1: Tạo Telegram Bot (3 phút)
1. Mở Telegram → tìm `@BotFather`
2. Gửi: `/newbot`
3. Đặt tên bot: `Tech News AI`
4. Đặt username: `your_name_tech_bot`
5. **Copy Bot Token** → Lưu lại
6. Tìm `@userinfobot` → **Copy Chat ID** → Lưu lại
7. Vào chat với bot của bạn → gửi `/start`

---

### 🔹 BƯỚC 2: Lấy Gemini API Key (2 phút)
1. Vào: https://aistudio.google.com/app/apikey
2. Click "Create API Key"
3. **Copy API Key** → Lưu lại

---

### 🔹 BƯỚC 3: Setup n8n (5 phút)

**Mở n8n:**
```bash
# Nếu chưa cài
npm install n8n -g

# Chạy n8n
n8n start
```

Browser: http://localhost:5678

**Import Workflow:**
1. Click "Workflows" → "Add workflow" → menu "..." → "Import from File"
2. Upload file: `tech-news-workflow.json`
3. Workflow sẽ hiển thị

---

### 🔹 BƯỚC 4: Cấu hình Credentials (3 phút)

**A. Gemini API:**
1. Click node "Gemini Analysis"
2. Click "Credential for Google PaLM"
3. "+ Create New Credential"
4. Paste **Gemini API Key**
5. Click "Save"

**B. Telegram Bot:**
1. Click node "Send to Telegram"
2. Click "Credential for Telegram"
3. "+ Create New Credential"
4. Paste **Bot Token**
5. Click "Save"
6. Trong node settings, điền **Chat ID** của bạn
7. Click "Execute Node" để test

**C. Google Sheets (Optional):**
1. Tạo Google Sheet mới
2. Copy Sheet ID từ URL (giữa `/d/` và `/edit`)
3. Click node "Save to Google Sheets"
4. Follow OAuth flow để connect Google account
5. Paste Sheet ID vào node settings

---

### 🔹 BƯỚC 5: Test & Activate (2 phút)

**Test workflow:**
1. Click node "Schedule Every 30min"
2. Click "Execute Node"
3. Chờ ~1-2 phút
4. Check Telegram → bạn sẽ nhận được tin!

**Activate workflow:**
1. Click toggle "Inactive" → "Active" ở góc trên
2. Workflow sẽ tự chạy mỗi 30 phút

---

## 🎯 Bạn đã xong! 

Từ giờ bạn sẽ nhận tin công nghệ quan trọng qua Telegram mỗi 30 phút!

---

## 🔧 Tùy Chỉnh Nhanh

### Thay đổi tần suất:
- Click node "Schedule Every 30min"
- Đổi `30` thành `60` (mỗi giờ) hoặc `15` (15 phút)

### Thay đổi ngưỡng score:
- Click node "Filter Score >= 7"
- Đổi `7` thành `8` (chỉ tin rất quan trọng) hoặc `6` (nhiều tin hơn)

### Thêm/bớt nguồn tin:
- Xóa node RSS không cần
- Hoặc add node RSS mới với URL từ guide chính

---

## 🆘 Troubleshooting Nhanh

**❌ Không nhận được tin:**
- Check workflow có "Active" không?
- Test từng node bằng "Execute Node"
- Check Telegram Chat ID đúng chưa?

**❌ Gemini API error:**
- Check API key có đúng không?
- Check quota tại: https://aistudio.google.com

**❌ RSS feed lỗi:**
- Bỏ qua, đợi lần chạy tiếp theo
- Hoặc thay RSS URL khác

---

## 📝 Ghi Chú

**Credentials cần lưu:**
```
Telegram Bot Token: __________________
Telegram Chat ID: __________________
Gemini API Key: __________________
Google Sheet ID (optional): __________________
```

**URLs quan trọng:**
- n8n: http://localhost:5678
- Gemini Console: https://aistudio.google.com
- Telegram Bot: https://t.me/your_bot_username

---

**🎉 Chúc mừng! Bạn đã có AI Agent riêng rồi!**

Next: Đọc file `ai-news-agent-guide.md` để tùy chỉnh nâng cao.
