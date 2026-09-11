# Ý TƯỞNG NGHIÊN CỨU #016: HỆ THỐNG ĐA TÀI KHOẢN GOOGLE FLOW & CẦU NỐI TỰ ĐỘNG HÓA N8N

> **Mã ý tưởng:** `#16 - Multi-Account Cookie/Credit Pool & N8N Integration Bridge`  
> **Nguồn cảm hứng:** Nghiên cứu thực tế từ case study của Xuân Linh (`workflowfree.com`, tháng 09/2026)  
> **Người tổng hợp:** Antigravity (Senior Engineer) & Anh Hùng (Product Owner)  
> **Mục tiêu:** Giải quyết triệt để bài toán khan hiếm Credit Veo/Flow cho người dùng, đồng thời đón đầu tập người dùng tự động hóa n8n để biến `hungdai-flow` thành trạm lắp ráp CapCut Draft hoàn chỉnh.

---

## 1. NGUYÊN LÝ KỸ THUẬT CỐT LÕI (REVERSE-ENGINEERED)

Từ quy trình thực tế của cộng đồng, cơ chế xác thực và quản lý tài nguyên của Google Flow vận hành như sau:

1. **Endpoint lấy phiên đăng nhập (Session & Cookie):**
   - URL: `https://labs.google/fx/api/auth/session`
   - Phương thức: `GET`
   - Dữ liệu trả về: Chứa Bearer Access Token hoặc Cookie Auth phiên làm việc của Google Account hiện tại.

2. **Endpoint truy vấn thông tin tác giả & số dư Credit (Google Internal API):**
   - URL: `https://aisandbox-pa.googleapis.com/...` (hoặc các route con kiểm tra hạn mức `/v1/user/credits` hoặc `/v1/user/info`).
   - Header: Cần đính kèm `Authorization: Bearer <Token>` lấy từ session ở bước 1.
   - Dữ liệu trả về: ID người dùng, email, hạn mức credit miễn phí còn lại trong ngày (thông thường 50 credit/tài khoản Gmail cá nhân).

3. **Thời gian sống của Cookie:**
   - Phiên làm việc kéo dài từ **12 - 18 tiếng**. Hết thời gian này Cookie sẽ hết hạn và cần người dùng mở lại tab để làm mới phiên.

---

## 2. BA HƯỚNG ÁP DỤNG CỤ THỂ VÀO DỰ ÁN HUNGDAI-FLOW

### Hướng A: "Bể Credit Đa Tài Khoản" Tích Hợp Sẵn (Built-in Multi-Account Pool)
*Thay vì bắt người dùng tự cài n8n, server, Google Sheets cồng kềnh, Extension tự quản lý nội bộ.*

- **Cách làm kỹ thuật:**
  1. Người dùng mở nhiều Profile Chrome (Profile 1, 2, 3...), mỗi Profile đăng nhập 1 Gmail khác nhau.
  2. Tại mỗi Profile, `hungdai-flow` tự động bắt sự kiện khi người dùng vào Google Flow, âm thầm đọc `labs.google/fx/api/auth/session` và lưu Token/Credit vào `chrome.storage.local`.
  3. Sử dụng cơ chế đồng bộ nhẹ (qua Supabase hoặc file JSON nội bộ do người dùng xuất ra), Side Panel sẽ hiển thị Dashboard tổng:
     ```text
     📊 TỔNG CREDIT KHẢ DỤNG HÔM NAY: 250 / 300
     - Profile 1 (hungdai01@gmail.com): 50 credits (Còn hạn 14h)
     - Profile 2 (hungdai02@gmail.com): 50 credits (Còn hạn 12h)
     - Profile 3 (hungdai03@gmail.com): 40 credits (Đang gen)
     ```
  4. Người dùng biết chính xác profile nào còn credit để mở tab đó chạy tiếp.

---

### Hướng B: Cầu Nối N8N / Google Sheets Sang CapCut Draft (N8N-to-CapCut Bridge)
*Đón đầu toàn bộ dân MMO đang chạy n8n tự động gen hàng trăm clip nhưng bị tắc ở khâu dựng video.*

- **Vấn đề của dân n8n:** n8n gọi API tải clip về Google Drive hoặc thư mục máy tính, nhưng không thể tạo được dự án CapCut PC để chèn nhạc nền và chuyển cảnh tự động.
- **Cách làm kỹ thuật trong hungdai-flow:**
  1. Tại tab **Công cụ (Tools)**, tạo một mục con: **"Ghép Video Từ Danh Sách N8N / Sheets"**.
  2. Người dùng cung cấp:
     - Thư mục chứa các file `.mp4` đã tải về máy tính (chọn qua `window.showDirectoryPicker()`).
     - (Tùy chọn) File `prompts.csv` hoặc link Google Sheets danh sách clip/kịch bản.
     - File nhạc nền `.mp3` và chọn hiệu ứng chuyển cảnh (ví dụ: Hòa tan - Dissolve).
  3. Bấm nút: **"⚡ Xuất Sang CapCut Draft Ngay"**.
  4. Module `src/lib/capcutDraft.ts` sẽ:
     - Quét toàn bộ video trong thư mục.
     - Lấy duration từng clip qua `<video>.onloadedmetadata`.
     - Tự động sinh `draft_content.json` và `draft_meta_info.json` theo chuẩn file thật `0831` (đã giải mã).
     - Ghi thẳng vào `D:\capcut\lưu trư tạm thoi\CapCut Drafts\<Tên_Dự_Án>\`.
  5. Người dùng mở CapCut PC lên là video 10-15 phút đã được dựng sẵn, chỉ việc bấm Render!

---

### Hướng C: Cảnh Báo Hết Credit & Gợi Ý Đổi Tab Thông Minh (Credit Guard & Switcher)
- Khi đang chạy Hàng đợi Batch:
  - Nếu API Google trả về mã lỗi hết credit hoặc số dư rơi về `0`:
  - `QueueEngine.ts` lập tức tạm dừng (Pause) hàng đợi một cách êm đẹp.
  - Hiện popup thông báo: *"Tài khoản trên Tab hiện tại đã hết credit. Vui lòng chuyển sang Tab của Profile 2 và bấm 'Tiếp Tục' để chạy nốt các prompt còn lại!"*
  - Giúp không bị mất các prompt đang đợi trong danh sách.

---

## 3. LỘ TRÌNH TRIỂN KHAI KHUYẾN NGHỊ

1. **Sprint hiện tại (SPRINT-02):**
   - Tập trung hoàn thiện module lõi `src/lib/capcutDraft.ts` (Tạo dự án CapCut Desktop chuẩn từ file local).
2. **Sprint tiếp theo (SPRINT-03):**
   - Thêm tính năng **Hướng B (N8N-to-CapCut Bridge)** vào tab Công cụ.
   - Viết bài hướng dẫn / video demo cách kết hợp `n8n + hungdai-flow + CapCut` để hút người dùng từ các cộng đồng tự động hóa (như nhóm của Xuân Linh).
3. **Sprint nâng cao (SPRINT-04):**
   - Nghiên cứu cơ chế bắt Token Google Flow tự động để hiển thị số dư Credit trực tiếp trên thanh Side Panel (Hướng A).
