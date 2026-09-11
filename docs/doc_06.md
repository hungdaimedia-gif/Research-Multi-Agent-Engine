# Original File: '07 — Cập nhật chốt_ Đ1_Đ2 đóng & phân công Bước 2.docx'

07 — Cập nhật chốt: Đ1/Đ2 đóng & phân công Bước 2
Ngày: 11/09/2026 · Loại: Phụ lục cho 06 — Chốt CapCut Draft Export
⚠️ Tài liệu này thay thế §5 của 06. Google Docs không sửa nội dung tại chỗ được, nên 06 vẫn ghi Đ1/Đ2 là "🔶 CÒN TREO" — thông tin đó nay đã cũ. Ai đọc 06 thì đọc tiếp file này.
§1. Hai điểm treo — ĐÃ ĐÓNG
Đ1 — CLOSED ✅
Antigravity xác nhận tuân thủ Điều 3: dữ liệu từ prompts.csv (do n8n/Drive sinh ra) bắt buộc đi qua:
prompts.csv → Zod validation → FormInputData → buildFlowJobsFromForm()
Tuyệt đối không tạo đường tắt đổ thẳng vào hàng đợi. Không còn nguy cơ tái hiện phân kỳ kiểu graphToJob.ts.
Đ2 — CLOSED ✅
Nút "Nạp từ Folder Inbox" đặt tại src/tabs/Tools.tsx. Không chạm src/tabs/Gen.tsx.
Lệnh chốt chặn giữ nguyên yêu cầu rỗng 100%:
git diff --stat -- src/tabs/gen src/services/flowProtocol.ts src/hooks/useQueueRunner.ts src/components src/state/store.ts
Mức rủi ro giữ ở RR 1.
Y1-bis & Y2 — được xác nhận ✅
Lưu DownloadItem.filename khi tải xong để có đường dẫn tuyệt đối; ô #folder-path giữ làm dự phòng
Number.isFinite() + Math.round(sec * 1_000_000) theo khuôn mẫu src/lib/audioLibrary.ts:106
§2. Phân công Bước 2 — chống đụng file
Antigravity nhận Bước 2: nâng src/lib/resultFolder.ts từ mode: 'read' (dòng 116) lên readwrite.
Trình tự: anh Hùng nộp v1.0.5 → Antigravity làm Bước 2 → chạy đủ 3 cổng → thông báo → Claude mới bắt đầu nhánh của mình.
Nút thắt được gỡ. Không ai chạm resultFolder.ts cho tới khi có thông báo.
§3. Ba lưu ý MỚI, phát sinh từ chính các quyết định vừa chốt
Việc chốt Y1-bis (lưu đường dẫn tuyệt đối) kéo theo ba hệ quả chưa ai bàn. Cả ba đều nhỏ nếu xử lý ngay từ đầu, và đều khó gỡ nếu để lọt.
L1 — Đường dẫn tuyệt đối chứa tên tài khoản Windows
DownloadItem.filename trả về dạng C:\Users\<TênNgườiDùng>\Downloads\HungDaiFlow_Outputs\01_abc.mp4.
Chuỗi này chứa tên tài khoản Windows của người dùng. Nó vô hại khi nằm trong máy, nhưng rò rỉ ngay nếu lọt vào một trong ba chỗ sau — mà cả ba đều đang nằm trong kế hoạch:
File cờ _BATCH_COMPLETED.json — n8n đọc file này, và n8n có thể đẩy tiếp lên Drive/Telegram/webhook
Báo cáo "Sao chép báo cáo" của ý tưởng #11 (Chẩn đoán DOM) — người dùng dán thẳng vào nhóm Facebook công khai
console.log trong bản phát hành
Yêu cầu: đường dẫn tuyệt đối chỉ được dùng bên trong máy, cho đúng mục đích sinh draft_content.json. Mọi thứ xuất ra ngoài (file cờ, báo cáo hỗ trợ, log) chỉ được chứa tên file, không chứa đường dẫn đầy đủ.
Bối cảnh: PRIVACY_POLICY.md §4 đang giữ được tuyên bố "Không đi qua bất kỳ máy chủ nào của chúng tôi". L1 không phá tuyên bố đó — nhưng nó tạo ra một đường rò sang phía người dùng tự đẩy đi, tinh vi hơn và không ai để ý.
L2 — Lưu filename vào storage phải là field optional cộng thêm
Điều 4: không đổi tên/kiểu/ngữ nghĩa field trong FormInputData, FlowSettingsSpec, QueueJob, ComfyNodeFields.
QueueJob/asset hiện đã có filename (tên dự định, tương đối — xem DownloadManager.ts:110-111). Đường dẫn tuyệt đối là thứ khác, ngữ nghĩa khác.
⇒ Không được ghi đè lên filename đang có. Phải là field mới, optional, tên khác hẳn — ví dụ absolutePath?: string. Ghi đè sẽ làm vỡ trong im lặng mọi job đã lưu trong chrome.storage.local của người dùng thật.
L3 — Y1-bis chạm lõi tải, capture phải thuần additive
Chỗ lấy filename là src/background/downloadManager.ts:52, trong callback của chrome.downloads.search. Giá trị đã nằm sẵn trong tay (items[0]), chỉ là hiện code chỉ đọc items[0].state.
Nhưng file này là mutex ghi file tuần tự vào ổ đĩa của background worker — thứ tồn tại để triệt tiêu race condition và nhầm tên file.
Ràng buộc:
Chỉ đọc thêm một trường, không đổi một nhánh điều kiện nào
Không đụng luồng state === 'complete' / 'interrupted', không đụng pollTimer, không đụng timeout 90 giây
Chạy npm test (có test-shared.mjs canh vùng dùng chung Ảnh/Video) — đây là loại thay đổi "nhỏ và chắc chắn an toàn" mà repo đã dính hồi quy hai lần
Ghi chú phối hợp: downloadManager.ts là file khác với resultFolder.ts của Bước 2 ⇒ làm song song được, không đụng nhau. Nhưng vẫn nên báo trước khi vào, vì cả hai đều nằm trong lõi tải.
§4. Trạng thái hiện tại
Đang chặn tất cả:
Nộp v1.0.5 — anh Hùng
Chặn riêng nhánh CapCut:
Lấy file draft_content.json thật từ CapCut (5 phút) — anh Hùng
Chạy được ngay sau khi nộp:
Bước 2: resultFolder.ts → readwrite — Antigravity (đã nhận, sẽ thông báo khi xong)
Bước 1b: kiểm chứng items[0].filename bằng một dòng log trong lượt chạy thật — ai chạy batch trước, file riêng nên không đụng Bước 2
Chờ tín hiệu:
src/lib/capcutDraft.ts + unit test — Claude, chờ file CapCut thật (Y4) và thông báo Bước 2
Trình đọc inbox + schema Zod → FormInputData — chờ Bước 2
Không đổi:
SHOW_UNFINISHED_TOOLS giữ false cho tới khi đủ 4 điều kiện "XONG" ở 06 §9