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

### 2.1. Quy tắc tính thời gian tuyến tính (XÁC NHẬN THỰC NGHIỆM TỪ DỰ ÁN THẬT 0831)

Từ phân tích file `0831/draft_content.json` trên máy thật (2 clips + transition `is_overlap: true` + 1 audio track):
- Clip 0: start = 0, duration = 35,133,333 µs
- Clip 1: start = 35,133,333 µs, duration = 29,700,000 µs
- Tổng duration = 64,833,333 µs (đúng bằng $35,133,333 + 29,700,000$)

**KẾT LUẬN CỐT LÕI:**
1. Trong CapCut Desktop, các segment trên Video Track được xếp **nối tiếp liên tục (contiguous)**:
   ```text
   start[0] = 0
   start[i] = start[i-1] + clip[i-1].durationUs   (với mọi i >= 1)
   totalDuration = Σ clip[i].durationUs
   ```
2. **Transition KHÔNG làm co ngắn timeline hay trừ bớt start[i]**:
   - Transition được khai báo trong `materials.transitions` với `is_overlap: true`.
   - UUID của transition được đưa vào `extra_material_refs` của Segment đứng TRƯỚC (Segment $i-1$).
   - CapCut tự động render hiệu ứng chuyển cảnh lấn sang 2 đầu clip mà không làm dịch chuyển timecode của segment sau.

### 2.2. Bắt buộc: `source_timerange` KHÔNG ĐƯỢC null
- Trong CapCut Desktop, nếu `source_timerange: null` thì CapCut sẽ báo lỗi hỏng dự án.
- Phải luôn truyền:
  ```json
  "source_timerange": {
    "start": 0,
    "duration": 35133333
  }
  ```

### 2.3. Bắt buộc: `extra_material_refs` cho mỗi Segment
Mỗi segment video bắt buộc phải có ít nhất:
1. `materials.speeds`: `{ id: uuidSpeed, type: "speed", mode: 0, speed: 1.0, curve_speed: null }`
2. `materials.canvases`: `{ id: uuidCanvas, type: "canvas_color", color: "", blur: 0, image: "", album_image: "", image_id: "", image_name: "", source_platform: 0, team_id: "" }`
3. Nếu có transition sau clip: UUID của `materials.transitions` được thêm vào `extra_material_refs`.

---

## 3. Cấu trúc tối giản `draft_meta_info.json`

```json
{
  "draft_id": "AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEEE",
  "draft_name": "<projectName>",
  "draft_fold_path": "D:/capcut/lưu trư tạm thoi/CapCut Drafts/<projectName>",
  "draft_root_path": "D:\\capcut\\lưu trư tạm thoi\\CapCut Drafts",
  "draft_cover": "",
  "draft_json_file": "draft_content.json",
  "tm_draft_create": 1789088328801047,
  "tm_draft_modified": 1789088335902705,
  "tm_duration": 64833333,
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
1. **Test Case 1 (Cơ bản):** 3 clip nối tiếp, không transition, không audio. Kiểm tra `duration`, `start` timecode nối tiếp chuẩn xác, `source_timerange` không null, `speeds` và `canvases` refs đầy đủ.
2. **Test Case 2 (Có Transition):** 2 clip + transition nối tiếp. Kiểm tra `is_overlap: true`, transition UUID có trong `materials.transitions` và nằm trong `extra_material_refs` của clip 0, start timecode của clip 1 không bị trừ âm.
3. **Test Case 3 (Audio Track):** Có nhạc nền với `trimToVideoLength: true`. Kiểm tra track audio có duration bằng tổng video duration, source_timerange chuẩn.
4. **Edge cases:** Báo lỗi hoặc ném ngoại lệ khi `clips.length === 0`, hoặc clip có duration <= 0.

---

## 5. Trạng thái Checklist
- [x] Lấy file `draft_content.json` và `draft_meta_info.json` thật từ CapCut Desktop (`0831`) đối chiếu field-by-field.
- [x] Xác nhận đơn vị timestamp là microseconds (µs).
- [x] Làm rõ cơ chế overlap transition trong `draft_content.json`.
- [ ] Xây dựng `src/lib/capcutDraft.types.ts` và `src/lib/capcutDraft.ts`.
- [ ] Viết test Vitest `src/lib/capcutDraft.test.ts`.
