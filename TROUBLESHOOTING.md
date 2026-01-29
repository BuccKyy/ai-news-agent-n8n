# 🔧 Troubleshooting Guide - Tech News AI Agent

## 🚨 Các Vấn Đề Thường Gặp & Cách Fix

---

## 1️⃣ Telegram Issues

### ❌ Lỗi: "Bad Request: chat not found"

**Nguyên nhân:** Bot chưa được start bởi user

**Cách fix:**
1. Mở Telegram
2. Tìm bot của bạn: `@your_bot_username`
3. Click "Start" hoặc gửi `/start`
4. Test lại workflow trong n8n

---

### ❌ Lỗi: "Unauthorized"

**Nguyên nhân:** Bot token không đúng

**Cách fix:**
1. Quay lại @BotFather
2. Gửi `/token` và chọn bot
3. Copy token mới
4. Trong n8n: Settings → Credentials → Edit Telegram credential
5. Paste token mới → Save
6. Test lại

---

### ❌ Tin nhắn không format đẹp / hiển thị code

**Nguyên nhân:** Markdown syntax bị lỗi trong message

**Cách fix:**
Kiểm tra trong node "Format for Telegram":
- Dấu `*` phải thành cặp cho bold
- Dấu `_` phải thành cặp cho italic
- Link phải đúng format: `[text](url)`
- Không có `*`, `_`, `[` trong text thông thường → escape bằng `\`

**Code fix:**
```javascript
// Trong "Format for Telegram" node
// Escape special characters
const escapeMarkdown = (text) => {
  return text.replace(/([*_\[\]()~`>#+\-=|{}.!])/g, '\\$1');
};

const title = escapeMarkdown(data.title);
```

---

### ❌ Nhận quá nhiều tin / Spam

**Cách fix 1: Tăng threshold score**
- Node "Filter Score >= 7" → đổi thành `>= 8` hoặc `>= 9`

**Cách fix 2: Giới hạn số tin mỗi lần**
Trong node "Format for Telegram", thêm:
```javascript
// Chỉ lấy top 3 tin mỗi lần
const limitedItems = items.slice(0, 3);
```

**Cách fix 3: Tăng interval**
- Node "Schedule" → đổi 30 phút thành 60 phút hoặc 2 giờ

---

## 2️⃣ Gemini API Issues

### ❌ Lỗi: "API key not valid"

**Cách fix:**
1. Vào: https://aistudio.google.com/app/apikey
2. Kiểm tra API key còn active không
3. Nếu hết hạn → tạo key mới
4. Update trong n8n credentials

---

### ❌ Lỗi: "Quota exceeded"

**Nguyên nhân:** Vượt quá limit free tier

**Cách fix:**
1. Check quota: https://aistudio.google.com
2. Giảm frequency (60 phút thay vì 30 phút)
3. Giảm số items xử lý mỗi lần:
```javascript
// Trong "Clean Data" node
return uniqueItems.slice(0, 10); // Chỉ xử lý 10 items
```

---

### ❌ Lỗi: "Rate limit exceeded"

**Cách fix:** Thêm delay giữa các requests

Thêm node "Split Out" sau "Clean Data":
- Batch Size: 1
- Add delay: 2 seconds

Hoặc trong Code node:
```javascript
// Thêm delay function
const delay = ms => new Promise(resolve => setTimeout(resolve, ms));

for (const item of items) {
  // Process item
  await delay(2000); // 2 giây delay
}
```

---

### ❌ Response không parse được JSON

**Nguyên nhân:** Gemini trả về text thừa hoặc format sai

**Cách fix:** Cải thiện prompt trong node "Gemini Analysis"

**Prompt tốt hơn:**
```
Analyze this tech article and return ONLY a JSON object (no other text):

Title: {{ $json.title }}
Description: {{ $json.description }}

Return this EXACT format:
{"summary": "...", "score": 8, "category": "AI", "key_points": ["point1", "point2"]}

Rules:
- summary: 2-3 sentences in Vietnamese
- score: 1-10 integer
- category: one of [AI, Web, Mobile, Cloud, DevOps, Security, Other]
- key_points: array of 2-3 strings

RETURN ONLY THE JSON, NO OTHER TEXT.
```

**Cải thiện parser** trong node "Parse AI Response":
```javascript
try {
  let text = item.json.response || item.json.text || '';
  
  // Remove markdown code blocks
  text = text.replace(/```json\n?/g, '').replace(/```\n?/g, '');
  
  // Extract JSON object
  const jsonMatch = text.match(/\{[\s\S]*\}/);
  if (!jsonMatch) {
    console.log('No JSON found, skipping');
    continue;
  }
  
  const parsed = JSON.parse(jsonMatch[0]);
  
  // Validate structure
  if (!parsed.summary || !parsed.score || !parsed.category) {
    console.log('Invalid JSON structure, skipping');
    continue;
  }
  
  // Process item...
} catch (error) {
  console.log('Parse error:', error.message);
  continue;
}
```

---

## 3️⃣ RSS Feed Issues

### ❌ Lỗi: "Request failed with status code 403"

**Nguyên nhân:** Website block bot/crawler

**Cách fix:** Thêm User-Agent header

Trong RSS node → Options → Headers:
```json
{
  "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36"
}
```

---

### ❌ RSS feed không trả về data / Empty items

**Cách fix:**
1. Test RSS URL trong browser trước
2. Một số feed yêu cầu authentication
3. Thử dùng alternative RSS service như: https://rss.app/ hoặc https://feedrabbit.com/

---

### ❌ Feed update chậm / Tin cũ

**Nguyên nhân:** RSS feed cache

**Cách fix:**
- Thêm query param để bypass cache:
```
https://example.com/feed?t={{Date.now()}}
```

- Hoặc dùng HTTP Request node thay vì RSS node:
```javascript
// Fetch RSS manually
const response = await fetch(rssUrl, {
  headers: { 'Cache-Control': 'no-cache' }
});
```

---

## 4️⃣ n8n Workflow Issues

### ❌ Workflow không chạy tự động

**Check list:**
1. ✅ Workflow có toggle "Active" không?
2. ✅ Schedule node có đúng cron expression không?
3. ✅ n8n server có đang chạy không?

**Cách fix:**
```bash
# Check n8n process
ps aux | grep n8n

# Nếu không chạy, restart:
n8n start

# Hoặc với PM2:
pm2 restart n8n

# Hoặc Docker:
docker restart n8n
```

---

### ❌ Workflow execution bị timeout

**Nguyên nhân:** Xử lý quá nhiều items cùng lúc

**Cách fix:**
1. Settings → Execution Timeout → tăng lên (default 120s → 300s)
2. Hoặc giảm số items:
```javascript
// Trong các node, limit items
return items.slice(0, 20);
```

---

### ❌ Lỗi: "Workflow has no active webhook"

**Cách fix:**
- Node "Schedule Trigger" phải là node đầu tiên
- Kiểm tra connections giữa các nodes
- Delete và tạo lại Schedule node

---

### ❌ Data không flow giữa các nodes

**Cách fix:**
1. Click "Execute Workflow" để xem data flow
2. Hover qua connections để xem data
3. Check console logs: n8n settings → Enable debug logs

---

## 5️⃣ Google Sheets Issues

### ❌ Authentication failed / Unauthorized

**Cách fix:**
1. n8n Credentials → Google Sheets
2. Click "Reconnect"
3. Complete OAuth flow lại

---

### ❌ Data không save vào sheet

**Check:**
1. Sheet ID có đúng không?
2. Sheet name có đúng không? (case-sensitive)
3. Columns trong sheet có match với data không?

**Cách fix:**
Trong node "Save to Google Sheets":
- Columns Mapping Mode: "Auto Map Input Data"
- Hoặc manual map từng column

---

### ❌ Lỗi: "The caller does not have permission"

**Cách fix:**
1. Open Google Sheet
2. Share → Add n8n service account email
3. Give "Editor" permission

---

## 6️⃣ Performance Issues

### ❌ Workflow chạy quá chậm

**Optimization tips:**

**1. Parallel processing:**
```javascript
// Thay vì sequential
for (const item of items) {
  await processItem(item);
}

// Dùng parallel
await Promise.all(items.map(item => processItem(item)));
```

**2. Reduce API calls:**
- Batch items before calling Gemini
- Cache results

**3. Limit items:**
```javascript
// Chỉ xử lý top 30 items
const topItems = items
  .sort((a, b) => new Date(b.pubDate) - new Date(a.pubDate))
  .slice(0, 30);
```

---

### ❌ Memory leak / n8n crashes

**Cách fix:**
1. Restart n8n định kỳ:
```bash
# Với PM2
pm2 restart n8n --cron-restart="0 0 * * *"  # Restart lúc midnight
```

2. Giảm số workflow executions được lưu:
- Settings → Executions → Save only error executions

---

## 7️⃣ Debugging Tips

### 🔍 Cách debug hiệu quả:

**1. Execute từng node riêng:**
- Click node → "Execute Node"
- Xem output data

**2. Add debug nodes:**
```javascript
// Thêm Code node để log data
console.log('Current data:', JSON.stringify($json, null, 2));
return $input.all();
```

**3. Check execution logs:**
- Executions list → Click execution
- Xem từng step

**4. Test với static data:**
- Tắt Schedule trigger
- Dùng Manual trigger với test data

---

## 8️⃣ Advanced Issues

### ❌ Duplicate notifications

**Cách fix:** Implement state management

```javascript
// Dùng n8n's static data
const workflow = this.getWorkflowStaticData('global');
const sentUrls = workflow.sentUrls || [];

// Check if URL was already sent
const newItems = items.filter(item => !sentUrls.includes(item.url));

// Update state
workflow.sentUrls = [
  ...sentUrls.slice(-1000), // Keep last 1000
  ...newItems.map(i => i.url)
];

return newItems;
```

---

### ❌ Timezone issues

**Cách fix:**
```javascript
// Format với timezone Vietnam
const date = new Date().toLocaleString('vi-VN', { 
  timeZone: 'Asia/Ho_Chi_Minh',
  dateStyle: 'short',
  timeStyle: 'short'
});
```

---

## 📞 Cần Hỗ Trợ Thêm?

**n8n Community:**
- Forum: https://community.n8n.io
- Discord: https://discord.gg/n8n

**Documentation:**
- n8n Docs: https://docs.n8n.io
- Gemini API: https://ai.google.dev/docs

**GitHub Issues:**
Nếu tìm thấy bug trong workflow này, có thể báo cáo để cải thiện!

---

## ✅ Checklist Khi Gặp Lỗi

1. ☐ Check n8n server có chạy không?
2. ☐ Check workflow có "Active" không?
3. ☐ Test từng node riêng lẻ
4. ☐ Xem execution logs
5. ☐ Check credentials còn valid không?
6. ☐ Check API quotas
7. ☐ Test với ít items trước (slice 0-5)
8. ☐ Xem console logs
9. ☐ Restart n8n nếu cần
10. ☐ Google error message cụ thể

---

**Happy debugging! 🐛→✅**
