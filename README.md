# BÀI 8:
## PHẦN 1: PHÂN TÍCH HẬU QUẢ & LỖI KỸ THUẬT

### 1. 04 Secrets nhạy cảm bị phơi bày và hậu quả thảm họa
Dựa vào đoạn code trong file `server.js`, 4 mã bí mật đang bị hardcode trực tiếp bao gồm:
*   `OPENAI_API_KEY`: Chuỗi khóa `"sk-proj-998822abcdef77..."`
    *   *Hậu quả:* Hacker dùng bot quét được trong vài giây, chiếm quyền gọi API để đào coin/bán lại băng thông, làm trừ sạch tiền trong thẻ tín dụng của Trưởng nhóm (thực tế đã bị trừ 1,200 USD).
*   `STEAM_WEB_API_KEY`: Chuỗi khóa `"882310AAFFBBDD"`
    *   *Hậu quả:* Hacker thao túng dữ liệu người chơi, vật phẩm ảo trong game CyberRealm, phá hoại danh tiếng của studio trên cổng Steamworks.
*   `MONGO_URI`: Chuỗi kết nối `"mongodb+str://admin_root:SuperSecretPass2026!..."`
    *   *Hậu quả:* Lộ toàn bộ tài khoản quản trị Cơ sở dữ liệu, hacker có thể xóa sạch data game hoặc tống tiền dữ liệu người dùng.
*   `JWT_SECRET`: Chuỗi khóa `"secret_for_game_super_private_token_key_9988"`
    *   *Hậu quả:* Hacker tự sinh được mã Token giả lập để đăng nhập giả mạo (Bypass Auth) vào bất kỳ tài khoản người chơi hay admin nào trong hệ thống mà không cần mật khẩu.

### 2. Vì sao dù xóa commit hoặc chuyển repository sang Private sau 10 phút thì chìa khóa vẫn coi như đã bị lộ và phải thu hồi (Revoke) ngay?
*   **Cơ chế hoạt động của GitHub Scraper Bot:** Các bot quét của hacker liên tục lắng nghe và bắt sự kiện (Event Push) trên toàn cầu theo thời gian thực. Tốc độ thu thập dữ liệu của chúng chỉ mất từ **3 đến 5 giây** ngay sau khi mã nguồn xuất hiện trên repository public.
*   **Tính chất của Git History:** Dù có xóa commit hay đổi sang Private sau 10 phút, dữ liệu bí mật vẫn nằm lại trong lịch sử lưu trữ (Git History/Cache) hoặc đã bị bot tải về máy chủ của hacker từ giây thứ 5. Vì vậy, các chìa khóa này đã hoàn toàn mất tính an toàn và bắt buộc phải thực hiện lệnh **Thu hồi (Revoke/Hủy hiệu lực)** trên các nền tảng (OpenAI, MongoDB, Steam) để tạo mã mới ngay lập tức.

---

## PHẦN 2: SỬA LỖI & CHUẨN HÓA AN NINH

### 3. Nội dung tệp `.env` đặt trên máy cục bộ
```env
OPENAI_API_KEY=sk-proj-998822abcdef77ff88990008877665544332211
STEAM_WEB_API_KEY=882310AAFFBBDD
MONGO_URI=mongodb+str://admin_root:SuperSecretPass2026!@cluster0.pixelforge.net/cyberrealm_db
JWT_SECRET=secret_for_game_super_private_token_key_9988
```

### 4. Viết lại đoạn mã trong file `server.js` an toàn
```javascript
// tệp server.js (ĐÃ VÁ LỖI)
const express = require('express');
const app = express();

// Sử dụng biến môi trường qua process.env thay vì hardcode trực tiếp
const OPENAI_API_KEY = process.env.OPENAI_API_KEY;
const STEAM_WEB_API_KEY = process.env.STEAM_WEB_API_KEY;
const MONGO_URI = process.env.MONGO_URI;
const JWT_SECRET = process.env.JWT_SECRET;

app.listen(3000, () => console.log('Game Server running on port 3000'));
```

### 5. Nội dung tệp `.gitignore`
```text
# Chặn tệp chứa cấu hình bí mật thật để không bao giờ bị đẩy lên Git
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# Thư mục thư viện bên thứ ba
node_modules/
```

### 6. Nội dung tệp `.env.example` để push lên GitHub cho nhóm sử dụng
```env
# Mẫu cấu hình môi trường cho dự án CyberRealm - Tuyệt đối để trống giá trị thật
OPENAI_API_KEY=your_openai_api_key_here
STEAM_WEB_API_KEY=your_steam_web_api_key_here
MONGO_URI=your_mongodb_connection_string_here
JWT_SECRET=your_jwt_secret_key_here
```
