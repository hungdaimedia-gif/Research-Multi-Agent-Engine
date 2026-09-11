# ĐẶC TẢ KỸ THUẬT: `src/lib/capcutDraft.ts`
**Module:** CapCut Draft Export cho hungdai-flow  
**Người viết:** Claude (Tech Lead & Solution Architect)  
**Đối tượng đọc:** Antigravity (Senior Engineer) & Anh Hùng (Product Owner)  
**Ngày:** 11/09/2026  

> ⚠️ **Lưu ý quan trọng về độ tin cậy:** Định dạng `draft_content.json` / `draft_meta_info.json` của CapCut **không phải là định dạng có tài liệu chính thức từ công ty**. Cấu trúc dưới đây được tổng hợp từ hiểu biết chung về cách các công cụ cộng đồng (kiểu `pyJianYingDraft` và tương tự) đã suy ngược (reverse-engineer) định dạng này qua nhiều phiên bản CapCut/JianYing. Cấu trúc **có thể lệch** so với phiên bản CapCut Desktop cụ thể mà anh Hùng đang dùng, và **có thể thay đổi giữa các bản cập nhật CapCut**.
>
> **Antigravity bắt buộc phải làm bước sau trước khi code:** xuất thử 1 dự án CapCut thật (3 clip, có nhạc, có transition) từ CapCut Desktop, mở file `draft_content.json` thật đó lên, và **đối chiếu từng field** với đặc tả này. Đặc tả này nên được dùng làm khung sườn (scaffold) + bộ test, không nên được coi là "chân lý tuyệt đối" cho tới khi đối chiếu xong với ít nhất 1 file mẫu thật.

---

## 0. Vị trí trong kiến trúc (tuân thủ Điều 2 & Điều 3)

```text
src/lib/capcutDraft.ts        ← module mới, thuần logic, KHÔNG đụng core gen
src/lib/capcutDraft.types.ts  ← các type/interface
src/lib/capcutDraft.test.ts   ← unit test (Vitest/Jest tuỳ stack hiện tại)
```

Module này là **Pure Function**, không import bất kỳ thứ gì từ `src/tabs/gen`, `flowProtocol.ts`, `useQueueRunner.ts`, `src/state/store.ts`. Việc ghi ra ổ đĩa qua `showDirectoryPicker()` là một lớp **riêng biệt** (ví dụ `src/lib/capcutWriter.ts`) gọi `buildCapCutProject()` rồi mới ghi file — tách biệt để giữ hàm build là pure và test được không cần Chrome API.

---

## 1. Interface TypeScript đầu vào

```typescript
// src/lib/capcutDraft.types.ts

/** Đơn vị thời gian dùng xuyên suốt module: MICROSECONDS (µs). 1s = 1_000_000µs */
export type Microseconds = number;

export type AspectRatio = '16:9' | '9:16' | '1:1' | '4:3' | '21:9';

export interface CanvasConfig {
  ratio: AspectRatio;
  width: number;   // ví dụ 1920
  height: number;  // ví dụ 1080
  fps?: number;     // mặc định 30 nếu không truyền
}

export type TransitionType =
  | 'dissolve'      // Hòa tan
  | 'fade_black'    // Fade qua màu đen
  | 'push'          // Đẩy
  | 'wipe'          // Quét
  | 'zoom';         // Phóng to

export interface TransitionSpec {
  type: TransitionType;
  duration: Microseconds;
  /** true nếu transition này áp cho cạnh SAU của clip hiện tại (nối với clip kế tiếp) */
  appliesToNextClip?: boolean; // mặc định true
}

export interface VideoClipInput {
  /** Đường dẫn tuyệt đối trên ổ cứng, lấy từ DownloadItem.filename (xem mục Y1-bis) */
  absolutePath: string;
  /** Thời lượng gốc của file clip, tính bằng µs */
  durationUs: Microseconds;
  width: number;
  height: number;
  /** Transition áp dụng NGAY SAU clip này (nối sang clip kế tiếp). Bỏ trống nếu là clip cuối hoặc không có transition. */
  transitionAfter?: TransitionSpec;
  /** Cắt trong khoảng [trimInUs, trimOutUs] của clip gốc, nếu không truyền thì lấy toàn bộ clip */
  trimInUs?: Microseconds;
  trimOutUs?: Microseconds;
}

export interface AudioTrackInput {
  absolutePath: string;
  durationUs: Microseconds;
  /** 0.0 – 1.0 */
  volume: number;
  /** Nếu nhạc nền dài hơn timeline video, có cắt bớt cho vừa không. Mặc định true. */
  trimToVideoLength?: boolean;
}

export interface CapCutDraftInput {
  /** Tên dự án — cũng là tên thư mục sẽ tạo trong CapCut Projects */
  projectName: string;
  canvas: CanvasConfig;
  clips: VideoClipInput[];           // >= 1 phần tử
  backgroundAudio?: AudioTrackInput; // optional
}
```

---

## 2. Thuật toán dựng Timeline (`draft_content.json`)

### 2.1. Quy tắc tính thời gian tuyến tính

Gọi:
- `clip[i].durationUs` = thời lượng đã trim của clip thứ i
- `T[i]` = thời lượng transition đứng *ngay sau* clip i (0 nếu không có)

**Trường hợp KHÔNG có transition** (chuỗi nối tiếp thuần):
```text
start[0] = 0
start[i] = start[i-1] + clip[i-1].durationUs   (i >= 1)
totalDuration = Σ clip[i].durationUs
```

**Trường hợp CÓ transition:**
```text
start[0] = 0
start[i] = start[i-1] + clip[i-1].durationUs - T[i-1]   (i >= 1, T[i-1] là transition sau clip i-1)
totalDuration = Σ clip[i].durationUs - Σ T[i]  (với mọi transition tồn tại)
```

### 2.2. Sinh UUID v4 và liên kết `materials` ↔ `tracks[].segments[].material_id`
- Dùng `crypto.randomUUID().toUpperCase()`.
- Mỗi video clip $\rightarrow$ 1 object trong `materials.videos[]`.
- Mỗi transition $\rightarrow$ 1 object trong `materials.transitions[]`.
- Audio nền $\rightarrow$ 1 object trong `materials.audios[]`.
- `tracks[0]` (video) chứa các `segments[]` trỏ vào `material_id` tương ứng và `extra_material_refs` trỏ vào transition.

---

## 3. Cấu trúc tối giản `draft_meta_info.json`

```json
{
  "draft_id": "AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEEE",
  "draft_name": "<projectName>",
  "draft_fold_path": "<đường dẫn tuyệt đối tới thư mục dự án>",
  "draft_root_path": "<đường dẫn gốc CapCut Projects>",
  "draft_cover": "",
  "draft_json_file": "draft_content.json",
  "tm_draft_create": 0,
  "tm_draft_modified": 0,
  "draft_duration": 0,
  "draft_fps": 30,
  "draft_width": 1920,
  "draft_height": 1080,
  "draft_type": "",
  "draft_version": "1.0.0"
}
```

---

## 4. Kế hoạch Unit Test (TDD)
Bao gồm:
1. **Test Case 1 (Cơ bản):** 3 clip nối tiếp, không transition, không audio. Kiểm tra `duration`, `start` timecode, map material_id, không có audio track.
2. **Test Case 2 (Nâng cao):** 3 clip + transition hòa tan (dissolve 0.5s) + audio nền có trim. Kiểm tra overlap timecode, `extra_material_refs`, canvas 9:16.
3. **Edge cases:** ném lỗi khi `clips.length === 0`, hoặc khi `transition` dài hơn độ dài clip.

---

## 5. Checklist trước khi code thật
- [ ] Lấy 1 file `draft_content.json` thật từ CapCut Desktop (dự án 3 clip + nhạc + transition) để đối chiếu field-by-field.
- [ ] Xác nhận đơn vị timestamp trong `draft_meta_info.json`.
- [ ] Viết `computeTimeline()` như 1 hàm riêng, cô lập công thức overlap.
- [ ] Viết `capcutWriter.ts` (lớp ghi file qua `showDirectoryPicker`) tách biệt hoàn toàn khỏi `capcutDraft.ts`.
