# 🤖 Hướng Dẫn Xây Dựng AI News Agent với n8n + Gemini + Telegram

## 📋 Tổng Quan

Hệ thống này sẽ:
- Tự động thu thập tin tức công nghệ & AI từ nhiều nguồn
- Dùng Gemini API để phân tích và đánh giá độ quan trọng
- Gửi real-time qua Telegram khi có tin quan trọng
- Lưu trữ backup trong Google Sheets

---

## 🛠️ BƯỚC 1: Setup Telegram Bot

### 1.1. Tạo Bot với BotFather

1. Mở Telegram và tìm **@BotFather**
2. Gửi lệnh: `/newbot`
3. Đặt tên cho bot (ví dụ: "Tech News AI")
4. Đặt username cho bot (phải kết thúc bằng "bot", ví dụ: `technews_daily_bot`)
5. BotFather sẽ trả về **Bot Token** (dạng: `123456789:ABCdefGHIjklMNOpqrsTUVwxyz`)
   - ⚠️ **LƯU LẠI TOKEN NÀY** - bạn sẽ cần nó!

### 1.2. Lấy Chat ID của bạn

1. Tìm bot **@userinfobot** trên Telegram
2. Start chat với bot này
3. Nó sẽ hiển thị **Chat ID** của bạn (dạng số, ví dụ: `123456789`)
4. ⚠️ **LƯU LẠI CHAT ID NÀY**

---

## 🔑 BƯỚC 2: Chuẩn Bị API Keys

### 2.1. Gemini API Key

Vì bạn đã có Gemini Pro:
1. Truy cập: https://aistudio.google.com/app/apikey
2. Tạo API key mới
3. ⚠️ **LƯU LẠI API KEY**

### 2.2. Google Sheets (Optional - để backup)

1. Tạo Google Sheet mới: https://sheets.google.com
2. Đặt tên: "Tech News Archive"
3. Tạo các cột: `Date | Time | Source | Title | Summary | Score | URL | Category`

---

## ⚙️ BƯỚC 3: Cài Đặt n8n (Nếu chưa có)

Vì bạn dùng macOS:

```bash
# Cài qua npm (recommended)
npm install n8n -g

# Hoặc dùng Docker
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n

# Chạy n8n
n8n start
```

Mở browser: http://localhost:5678

---

## 🔄 BƯỚC 4: Tạo n8n Workflow

### 4.1. Import Workflow JSON

Tôi sẽ cung cấp file JSON để bạn import trực tiếp vào n8n. File này bao gồm:

**Workflow Components:**

1. **Schedule Trigger** - Chạy mỗi 30 phút
2. **RSS Feed Readers** - Đọc từ nhiều nguồn tin
3. **HTTP Request** - Lấy thêm tin từ API (Hacker News, Reddit)
4. **Merge Data** - Gộp tất cả nguồn tin
5. **Remove Duplicates** - Loại bỏ tin trùng lặp
6. **Gemini AI** - Phân tích và scoring
7. **Filter** - Chỉ lấy tin có score ≥ 7
8. **Format Message** - Format đẹp cho Telegram
9. **Telegram Send** - Gửi tin
10. **Google Sheets** - Lưu backup

---

## 📝 BƯỚC 5: Workflow JSON Configuration

### 5.1. Nguồn Tin Đề Xuất

**RSS Feeds (Miễn phí):**
```
Hacker News: https://hnrss.org/frontpage
TechCrunch AI: https://techcrunch.com/category/artificial-intelligence/feed/
The Verge: https://www.theverge.com/rss/index.xml
MIT Tech Review: https://www.technologyreview.com/feed
ArXiv CS.AI: http://export.arxiv.org/rss/cs.AI
Dev.to: https://dev.to/feed
GitHub Trending: https://mshibanami.github.io/GitHubTrendingRSS/daily/all.xml
```

**API Sources (Miễn phí):**
```
Hacker News API: https://hacker-news.firebaseio.com/v0/topstories.json
Reddit API: https://www.reddit.com/r/MachineLearning/hot.json?limit=20
```

### 5.2. Cấu Hình Chi Tiết Từng Node

#### Node 1: Schedule Trigger
```json
{
  "rule": "*/30 * * * *",
  "type": "Interval"
}
```

#### Node 2-4: RSS Feed Readers
Mỗi node RSS:
- URL: [paste RSS URL từ danh sách trên]
- Item Limit: 10

#### Node 5: Merge
- Mode: "Combine All"
- Combine Mode: "Append"

#### Node 6: Remove Duplicates
- Compare: "URL" field

#### Node 7: Gemini AI Analysis
```
Prompt template:

Phân tích bài viết công nghệ sau và trả về JSON:

Title: {{$json.title}}
Description: {{$json.description}}
URL: {{$json.url}}

Yêu cầu:
1. Tóm tắt nội dung chính (2-3 câu, tiếng Việt)
2. Đánh giá độ quan trọng (1-10) dựa trên:
   - Tính mới mẻ của công nghệ
   - Tác động đến ngành
   - Hữu ích cho developers
3. Phân loại: AI, Web, Mobile, Cloud, DevOps, Security, Other

Trả về ĐÚNG format JSON này:
{
  "summary": "tóm tắt tiếng Việt",
  "score": 8,
  "category": "AI",
  "key_points": ["điểm chính 1", "điểm chính 2"]
}
```

Configuration:
- Model: `gemini-pro` hoặc `gemini-1.5-pro`
- Temperature: 0.3
- Max Tokens: 500

#### Node 8: Filter
```javascript
// Code Filter
return $json.ai_response.score >= 7;
```

#### Node 9: Format Telegram Message
```javascript
// JavaScript Code
const item = $input.all()[0].json;
const ai = item.ai_response;

const message = `
🔥 *${item.title}*

📊 Score: ${ai.score}/10 | 📂 ${ai.category}

📝 *Tóm tắt:*
${ai.summary}

💡 *Key Points:*
${ai.key_points.map(p => `• ${p}`).join('\n')}

🔗 [Đọc thêm](${item.url})

⏰ ${new Date().toLocaleString('vi-VN')}
`;

return { message };
```

#### Node 10: Telegram Send
- Chat ID: [your chat ID]
- Text: `{{$json.message}}`
- Parse Mode: "Markdown"
- Disable Web Page Preview: false

#### Node 11: Google Sheets (Optional)
- Spreadsheet ID: [your sheet ID]
- Sheet Name: "Sheet1"
- Columns: Date, Time, Source, Title, Summary, Score, URL, Category

---

## 🎯 BƯỚC 6: Tối Ưu Workflow

### 6.1. Thêm Error Handling

Thêm **Error Trigger** node để xử lý lỗi:
- Nếu RSS feed down → skip và tiếp tục
- Nếu Gemini API lỗi → retry 3 lần
- Gửi cảnh báo qua Telegram nếu workflow fail

### 6.2. Rate Limiting

Để tránh spam Telegram:
```javascript
// Thêm node Function trước Telegram Send
const items = $input.all();

// Chỉ gửi tối đa 5 tin mỗi lần chạy
return items.slice(0, 5);
```

### 6.3. Caching

Thêm SQLite hoặc file-based cache để tránh gửi tin trùng:
```javascript
// Pseudo-code
const sentURLs = readFromCache();
const newItems = items.filter(item => !sentURLs.includes(item.url));
updateCache(newItems.map(i => i.url));
return newItems;
```

---

## 🚀 BƯỚC 7: Deploy & Monitor

### 7.1. Chạy n8n Persistently

**Option 1: PM2 (Recommended for macOS)**
```bash
npm install -g pm2
pm2 start n8n
pm2 save
pm2 startup
```

**Option 2: Docker với restart policy**
```bash
docker run -d \
  --name n8n \
  --restart always \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

### 7.2. Monitoring

Thêm daily report qua Telegram:
- Số tin đã xử lý
- Số tin đã gửi
- Top categories
- Workflow uptime

---

## 🎨 BƯỚC 8: Nâng Cao (Optional)

### 8.1. Thêm Telegram Commands

Tạo bot interactive:
- `/summary` - Tóm tắt tin trong ngày
- `/category AI` - Xem tin theo category
- `/score 9` - Chỉ xem tin score cao
- `/pause` - Tạm dừng notifications
- `/resume` - Tiếp tục

### 8.2. Personalization

Dùng Gemini để học preference:
```
Phân tích 30 bài viết gần đây mà user đã đọc (click link).
Tìm pattern về topics, keywords, style mà user thích.
Điều chỉnh scoring algorithm theo preference.
```

### 8.3. Multi-language Support

Thêm option để nhận tin bằng tiếng Anh hoặc tiếng Việt.

---

## 📊 BƯỚC 9: Sample Workflow JSON

Tôi đã tạo file JSON hoàn chỉnh mà bạn có thể import trực tiếp vào n8n.

**File:** `tech-news-agent-workflow.json`

**Cách import:**
1. Mở n8n (http://localhost:5678)
2. Click "Workflows" → "Import"
3. Paste JSON content hoặc upload file
4. Điền các credentials:
   - Gemini API Key
   - Telegram Bot Token
   - Telegram Chat ID
   - (Optional) Google Sheets credentials

---

## ⚠️ Lưu Ý Quan Trọng

### Rate Limits:
- **Gemini Pro**: ~60 requests/phút (free tier có thể thấp hơn)
- **Telegram Bot**: 30 messages/giây (quá đủ)
- **RSS Feeds**: Không giới hạn, nhưng nên cache

### Chi Phí:
- Gemini Pro: Miễn phí với quota hào phóng
- n8n: Miễn phí (self-hosted)
- Telegram: Miễn phí
- **Tổng: $0/tháng** ✅

### Privacy:
- Bot token là private key - không share
- Chat ID cũng là sensitive
- Cân nhắc encrypt Google Sheets nếu chứa data nhạy cảm

---

## 🐛 Troubleshooting

### Lỗi thường gặp:

**1. Telegram "Bot token invalid"**
- Kiểm tra lại token từ BotFather
- Đảm bảo không có space hoặc ký tự thừa

**2. "Chat not found"**
- Phải start chat với bot trước
- Gửi `/start` cho bot của bạn

**3. Gemini API "quota exceeded"**
- Kiểm tra quota tại Google AI Studio
- Giảm frequency (60 phút thay vì 30 phút)
- Thêm retry logic với exponential backoff

**4. RSS feed timeout**
- Tăng timeout setting trong HTTP Request node
- Thêm fallback sources

**5. Tin bị trùng lặp**
- Kiểm tra "Remove Duplicates" node
- Implement persistent cache

---

## 📚 Resources Thêm

- **n8n Documentation**: https://docs.n8n.io
- **Telegram Bot API**: https://core.telegram.org/bots/api
- **Gemini API Docs**: https://ai.google.dev/docs
- **RSS Best Practices**: https://www.rssboard.org/rss-specification

---

## 🎓 Next Steps

Sau khi setup xong workflow cơ bản:

1. **Tuần 1**: Monitor và điều chỉnh scoring logic
2. **Tuần 2**: Thêm more sources phù hợp với interest
3. **Tuần 3**: Implement caching và deduplication
4. **Tuần 4**: Thêm personalization dựa trên reading history

---

## 💡 Pro Tips

1. **Bắt đầu đơn giản**: Chỉ 2-3 RSS feeds đầu tiên
2. **Test riêng từng node**: Dùng "Execute Node" để debug
3. **Log everything**: Lưu logs để analyze sau
4. **Backup workflow**: Export JSON thường xuyên
5. **Join n8n community**: https://community.n8n.io

---

**Chúc bạn build thành công! 🚀**

Nếu có vấn đề gì trong quá trình setup, cứ hỏi nhé!
