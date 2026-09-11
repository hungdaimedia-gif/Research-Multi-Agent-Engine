# YÊU CẦU THIẾT KẾ KỸ THUẬT: CAPCUT DRAFT EXPORT MODULE
- **Dành cho:** Claude (Tech Lead & Solution Architect)
- **Từ:** Anh Hùng (Product Owner) & Antigravity (Senior Engineer)
- **Mã Sprint:** `SPRINT-02-CAPCUT-DRAFT`
- **Thời gian:** 11/09/2026

---

## 1. BỐI CẢNH DỰ ÁN
- Extension `hungdai-flow` v1.0.5 đã được nộp duyệt lên Chrome Web Store.
- Kiến trúc đã chốt tại tài liệu `docs/doc_05.md` (`06 — Chốt CapCut Draft Export`):
  + Không render trong trình duyệt (bỏ WebCodecs/mediabunny trong side panel).
  + Xuất trực tiếp dự án CapCut Desktop qua File System Access API (`showDirectoryPicker` mode 'readwrite').
  + Tận dụng `DownloadItem.filename` từ `chrome.downloads.search` để lấy đường dẫn video tuyệt đối (Y1-bis).
  + Tuyệt đối tuân thủ: ĐIỀU 0 (không xoá), Điều 2 (Gen luôn thắng), Điều 3 (một cửa `buildFlowJobsFromForm`), Điều 4 (chỉ thêm optional), Điều 6 (không nút bấm giả).

---

## 2. NHIỆM VỤ CẦN CLAUDE THIẾT KẾ
Yêu cầu Claude phản hồi chi tiết vào file `specs/01-capcut-draft-spec.md` gồm các mục:

1. **Khung Interface TypeScript (`src/lib/capcutDraft.ts`):**
   - Định nghĩa kiểu dữ liệu đầu vào `CapCutDraftInput`.
   - Interface `buildCapCutProject(input: CapCutDraftInput): CapCutDraftResult`.

2. **Thuật toán Timeline Microseconds (µs):**
   - Cách tính `start` và `duration` trên track video khi có và không có hiệu ứng chuyển cảnh (Transition).
   - Xử lý các trường hợp góc: `duration` clip là `Infinity` hoặc `NaN`.

3. **Cấu trúc ID & References:**
   - Cách liên kết giữa `materials` (videos, speeds, transitions, canvases) với `tracks[].segments`.

4. **Kế hoạch TDD (Unit Test Cases):**
   - Thiết kế 2 bộ dữ liệu test case mẫu chuẩn cho Vitest để Antigravity viết test trước khi viết code.
