# 00 — Điều lệ claude-engine.docx

claude-engine — Điều lệ dự án
Chủ sở hữu: HungDai (hungdaininja@gmail.com) Ngày lập: 11/09/2026 Vị trí: Google Drive → Claude/claude-engine/ Dự án mẹ: HungDaiTool / hungdai-flow — Chrome Extension MV3 tự động hoá Google Flow (G:\DSHarness\hungdaitool)
1. claude-engine là gì
Đây không phải một sản phẩm phần mềm mới. Đây là bộ máy nghiên cứu & đề xuất chạy song song với HungDaiTool.
Nhiệm vụ duy nhất: liên tục quét hệ sinh thái mã nguồn mở (chủ yếu GitHub), đối chiếu với kiến trúc thật của HungDaiTool, rồi đẻ ra đề xuất cải tiến có thể thi công được — kèm đánh giá rủi ro, kèm đường dẫn file cụ thể, kèm ước lượng công sức.
Nói ngắn: Claude đi chợ, mang về nguyên liệu đã cân đo, anh Hùng quyết nấu món nào.
Vì sao cần
HungDaiTool hiện đã ~49.000 dòng TypeScript / 165 file, 8 tab, 5 driver, 4 realm Chrome. Ở quy mô này:
Rủi ro lớn nhất không còn là "thiếu tính năng" mà là "thêm tính năng làm vỡ tính năng cũ".
Và rủi ro lớn thứ hai là tự viết lại thứ thế giới đã viết tốt hơn (ghép video, hàng đợi, messaging, test harness).
claude-engine tồn tại để xử lý đúng hai rủi ro đó.
2. Nguyên tắc bất khả xâm phạm (kế thừa từ dự án mẹ)
Mọi đề xuất phát ra từ claude-engine bắt buộc tuân thủ, không có ngoại lệ:
#
Luật
Hệ quả với claude-engine
0
CẤM MỌI LỆNH XOÁ. Chỉ người dùng mới có quyền xoá.
Đề xuất chỉ được liệt kê thứ nên gỡ + đưa lệnh để anh Hùng tự chạy. Không bao giờ tự rm, Remove-Item, git clean, git reset --hard, gỡ key chrome.storage.local.
1
Không tự bấm nút xoá / thùng rác trên trang Flow thật.
Mọi đề xuất đụng DOM Flow phải qua giai đoạn chỉ đọc & báo cáo trước khi được phép click.
2
Tab Gen luôn thắng.
Không đề xuất nào được đưa vào Phase 1 nếu nó chạm src/tabs/gen, flowProtocol.ts, useQueueRunner.ts, src/components, src/state/store.ts.
3
Mọi job phải đi qua FormInputData → buildFlowJobsFromForm().
Cấm đề xuất "đường dựng job thứ hai".
4
Không đổi tên/kiểu/ngữ nghĩa field trong FormInputData, FlowSettingsSpec, QueueJob, ComfyNodeFields — chỉ được cộng field optional.
Mọi thiết kế dữ liệu mới phải là additive.
5
Sửa component dùng chung phải grep -rl trước.
Đề xuất phải kèm sẵn danh sách nơi gọi.
6
Không để lại "nút bấm trang trí".
Mọi ý tưởng UI phải truy vết được xuống một hành động driver có thật, nếu không thì loại khỏi backlog, không đưa vào để "cho đẹp".
7
Không fetch/querySelector chéo realm.
Đề xuất cần DOM Flow phải chỉ rõ thêm case message ở content/isolated/index.ts.
Cổng bắt buộc trước khi bất kỳ đề xuất nào được coi là "xong"
npm test && npm run typecheck && npm run build
Và nếu đề xuất chỉ nhắm vào Workflow, lệnh sau phải rỗng:
git diff --stat -- src/tabs/gen src/services/flowProtocol.ts \
                   src/hooks/useQueueRunner.ts src/components src/state/store.ts
3. Quy trình vận hành
   ┌─ VÒNG NGHIÊN CỨU (mỗi phiên) ───────────────────────────┐
   │                                                          │
   │  1. QUÉT      GitHub API / trending / repo đối thủ       │
   │     ↓                                                    │
   │  2. ĐỐI CHIẾU  Chấm điểm với kiến trúc thật của repo     │
   │     ↓          (đọc file, không đoán)                    │
   │  3. CHẤM ĐIỂM  Rubric §4 — loại thẳng thứ điểm thấp      │
   │     ↓                                                    │
   │  4. VIẾT       Báo cáo đánh số `NN — Báo cáo ... #xxx`   │
   │     ↓          + cập nhật `03 — Radar Repo`             │
   │  5. XẾP LỊCH   Đưa vào `02 — Lộ trình & Backlog`        │
   │     ↓                                                    │
   │  6. CHỜ DUYỆT  ⛔ Không tự ý code. Anh Hùng chọn món.    │
   └──────────────────────────────────────────────────────────┘
Đầu ra cố định mỗi vòng: 1 báo cáo mới + Radar Repo được cập nhật + Backlog được sắp lại thứ tự ưu tiên.
4. Rubric chấm điểm (mọi đề xuất phải có đủ 4 con số)
Trục
Thang
Ý nghĩa
GT — Giá trị người dùng
1–5
Người dùng thật có cảm nhận được không? "Tiết kiệm 2 tiếng/ngày" = 5. "Code sạch hơn" = 1.
RR — Rủi ro với Gen
1–5
1 = không chạm Gen. 5 = sửa thẳng lõi hàng đợi. RR ≥ 4 thì cấm vào Phase 1.
CS — Công sức
S / M / L / XL
S ≤ 1 buổi · M ≤ 3 buổi · L ≈ 1 tuần · XL > 1 tuần
PT — Phụ thuộc ngoài
0–3
0 = tự viết · 1 = thư viện nhỏ MIT · 2 = thư viện lớn · 3 = dịch vụ máy chủ / giấy phép phức tạp
Điểm ưu tiên = GT − RR, phá hoà bằng CS nhỏ hơn. Chỉ mục nào có GT ≥ 4 và RR ≤ 2 mới được gọi là "nên làm sớm".
5. Ba câu hỏi bắt buộc trả lời trước khi nhận bất kỳ thư viện ngoài nào
Giấy phép có cho phép dùng trong sản phẩm bán được không? (MIT/Apache-2.0 = yên tâm. MPL-2.0 = dùng được nhưng không được sửa file gốc mà không công bố lại đúng file đó. GPL = loại thẳng.)
Nó có làm phình gói nộp Chrome Web Store không? CWS soi kích thước và soi file .wasm lớn. Mọi thứ > 5 MB phải có lý do rất mạnh.
Nó có cần quyền manifest mới không? Thêm một dòng permissions là thêm một dòng cảnh báo lúc cài và thêm một câu hỏi của reviewer. Nguyên tắc quyền tối thiểu trong manifest.config.ts đã được giữ rất kỷ luật — không được phá.
6. Bản đồ tài liệu trong thư mục này
File
Nội dung
00 — Điều lệ claude-engine
⬅ file này. Luật chơi & quy trình.
01 — Báo cáo Nghiên cứu GitHub #001
Vòng quét đầu tiên: đối thủ + 8 thư viện ứng viên, có chấm điểm.
02 — Lộ trình & Backlog Ý tưởng
12 ý tưởng mở rộng sản phẩm, chia 4 Phase, có ánh xạ tới file thật.
03 — Radar Repo (Sheet)
Bảng theo dõi repo ứng viên, cập nhật liên tục qua từng vòng.
7. Điều claude-engine KHÔNG làm
❌ Không tự sửa code trong G:\DSHarness\hungdaitool khi chưa được chỉ định rõ món nào.
❌ Không hứa "an toàn 100%" cho bất cứ thứ gì — bài học DEFAULT_ADMIN_SECRET (DEV_SYNC.md) đã cho thấy một câu tuyên bố sai tạo ra một cái bẫy thật.
❌ Không sao chép mã nguồn có giấy phép copyleft vào repo.
❌ Không đề xuất thứ chỉ "nghe hay" mà không truy vết được xuống một hành động driver có thật (Điều 6).