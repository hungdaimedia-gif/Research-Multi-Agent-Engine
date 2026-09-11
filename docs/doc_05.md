# Original File: '06 — Chốt CapCut Draft Export.docx'

06 — Chốt CapCut Draft Export
Ngày chốt: 11/09/2026 Các bên: anh Hùng (quyết định) · Antigravity (đề xuất kiến trúc) · Claude (rà soát kỹ thuật) Nguồn gốc: docs/CAPCUT_DRAFT_JSON_SPEC.md trong repo + trao đổi phiên 11/09
Trạng thái:
✅ ĐÃ CHỐT — Hướng kiến trúc · Y1–Y4 · Cách tách code
🔶 CÒN TREO — Đ1 và Đ2 (§5), cần Antigravity trả lời
⭐ PHÁT HIỆN MỚI SAU KHI CHỐT — Y1 có thể gọn hơn (§3, Y1-bis)
§0. Ưu tiên bất di bất dịch
Nộp v1.0.5 lên Chrome Web Store trước. Không tính năng nào được chen vào trước việc đó. Cả ba bên đã thống nhất, không mở lại.
Mọi việc trong tài liệu này chạy trong cửa sổ 2–5 ngày chờ CWS duyệt.
§1. Quyết định kiến trúc: không tự render, xuất dự án CapCut
Bỏ: nhồi mediabunny / WebCodecs vào side panel để tự ghép và render video. Làm: sinh draft_content.json + draft_meta_info.json để CapCut trở thành cỗ máy dựng phim chính thức của tool.
Được gì
Không đụng WebCodecs ⇒ không lo RAM, không cần nút Huỷ khi render dài
Không phải chuẩn hoá codec/fps/độ phân giải giữa các clip
Không dính câu hỏi giấy phép MPL-2.0 của mediabunny
Người dùng giữ quyền biên tập — creator muốn sửa, không muốn nhận một file MP4 đã khoá
CapCut là công cụ tập người dùng Việt vốn đã dùng hằng ngày
Mất gì — phải nói rõ, không được quên
CapCut cần con người ngồi vào. Không có cách nào bảo CapCut render headless.
⇒ Dây chuyền tự động n8n (ý tưởng #15) KHÔNG bao gồm được CapCut. Hai thứ này giải hai bài toán khác nhau:
CapCut Draft = người dùng muốn kiểm soát biên tập
mediabunny = dây chuyền chạy không người
⇒ mediabunny KHÔNG bị loại, chỉ xuống hạng. Giữ trong Radar Repo như nhánh headless, xem lại khi automation thực sự quan trọng.
§2. Hai quy tắc sống còn của định dạng
Đơn vị thời gian là micro-giây (µs). 1 giây = 1.000.000. Clip 5 giây = 5000000. Hoà tan 0,5 giây = 500000.
Tham chiếu qua ID. Segment trên track không chứa dữ liệu video, chỉ chứa target_timerange + material_id trỏ vào materials.videos.
§3. Y1–Y4 — đã chốt
Y1 — Extension không biết đường dẫn tuyệt đối
Vấn đề: materials.videos[].path của CapCut cần absolute path. Nhưng:
chrome.downloads chỉ nhận đường dẫn tương đối khi ghi — đã ghi ở resultFolder.ts:14 và FlowJobExecutor.ts:50
FileSystemDirectoryHandle không phơi ra đường dẫn đầy đủ (chặn vì bảo mật), chỉ có .name
Giải pháp đã chốt: người dùng dán đường dẫn một lần vào ô #folder-path đã có sẵn trên giao diện (giá trị mẫu G:\HungDai_Videos\hungdaiflow-01). Extension ghép tên file vào.
⭐ Y1-bis — PHÁT HIỆN SAU KHI CHỐT, đề nghị nâng cấp
src/background/downloadManager.ts:52 đã gọi chrome.downloads.search({ id: downloadId }) — nhưng chỉ đọc items[0].state, bỏ qua items[0].filename.
Theo tài liệu Chrome Extensions, DownloadItem.filename là "Absolute local path" — tức chính xác thứ CapCut cần.
Nếu đúng, nó giải quyết luôn hai chuyện:
Không cần người dùng dán đường dẫn cho file do extension tải
Xử lý được bẫy conflictAction: 'uniquify' — Chrome có thể đổi tên thành 01_abc (1).mp4, và tên thật trên đĩa chỉ API mới biết, không suy ra được từ tên dự định
⚠️ Chưa kiểm chứng bằng thực nghiệm. Mình khẳng định từ tài liệu API, không phải từ việc đã chạy thử. Áp dụng đúng kỷ luật Y4: một dòng console.log(items[0].filename) trong một lượt chạy thật là chốt được.
Cần kiểm thêm: nhánh native download (Flow tự bấm nút tải) cũng là download của Chrome nên về lý thuyết vẫn nằm trong API — cần xác nhận EXPECT_NATIVE_DOWNLOAD có giữ lại downloadId không.
Kết luận đề xuất: filename từ API là đường chính; ô dán đường dẫn tay giữ làm đường dự phòng — vẫn cần cho file người dùng tự bỏ vào thư mục (ví dụ nhạc nền .mp3).
Y2 — Thời lượng clip lấy từ đâu
Vấn đề: nhánh native download, resultFolder.ts ghi rõ "extension KHÔNG bao giờ cầm bytes video". Không có duration thì clip 2 sẽ đè lên clip 1 hoặc để hở khoảng trống trên timeline.
Giải pháp đã chốt: đọc ngược từ đĩa qua handle của resultFolder + <video>.onloadedmetadata. Repo đã có đúng khuôn mẫu này ở src/lib/audioLibrary.ts:106 — tái sử dụng, không viết mới.
⚠️ Hai chi tiết phải xử lý, nếu bỏ qua là timeline vỡ:
HTMLMediaElement.duration trả số giây dạng float, không phải micro-giây. Phải Math.round(sec * 1_000_000). Nói "chính xác đến từng micro-giây" là nói quá — độ chính xác phụ thuộc container.
duration có thể trả về Infinity hoặc NaN khi metadata chưa nạp xong hoặc container thiếu thông tin. Bắt buộc kiểm tra Number.isFinite() — audioLibrary.ts:106 đã làm đúng, chép theo.
Y3 — Bỏ file .bat
Lý do: extension sinh ra file thực thi để ghi vào %LocalAppData% là đúng khuôn mẫu malware dùng. Chrome tự cảnh báo đỏ khi tải .bat. CWS soi rất gắt. Hồ sơ đang giữ được dòng "✅ Sạch — không eval, không mã từ xa" trong CWS_AUDIT.md — không đáng đánh đổi.
Giải pháp đã chốt: File System Access API mode: 'readwrite'. Người dùng bấm "Chọn thư mục CapCut Projects" một lần, extension ghi thẳng 2 file JSON vào com.lveditor.draft\<tên>\.
Ghi chú ghép việc: resultFolder.ts:116 hiện xin mode: 'read'. Việc nâng lên readwrite dùng chung cho cả CapCut draft và file cờ _BATCH_COMPLETED.json của ý tưởng #15 ⇒ một lần nâng quyền, mở khoá hai tính năng. Làm chung, đừng làm hai lần.
Y4 — Đối chiếu file CapCut THẬT, không tin spec
Lý do: draft_content.json là định dạng nội bộ, không có tài liệu, ByteDance đổi giữa các bản. Thiếu một trường bắt buộc thì CapCut mở ra timeline rỗng, im lặng, không báo lỗi.
Bằng chứng spec hiện tại không đủ tin:
Khai materials.speeds và materials.canvases nhưng seg_01 không trỏ tới chúng trong extra_material_refs. Draft thật bắt buộc segment tham chiếu speed/canvas.
resource_id: "675849" của hiệu ứng Hoà tan là số ma thuật không rõ nguồn; các ID này khác nhau theo phiên bản và khu vực.
Spec dùng dấu / trong đường dẫn Windows (G:/HungDai_Videos/...). Draft thật dùng / hay \\? Chưa ai kiểm.
Đây đúng loại rủi ro mà RULES.md điều 1 ra đời để chặn — selector đoán mò từng bấm nhầm nút Thùng rác.
Việc của anh Hùng (5 phút, chặn cả nhánh CapCut):
Mở CapCut PC
Tạo dự án mới, kéo 2 clip ngắn + 1 transition Hoà tan + 1 file nhạc nền
Lưu tên Test_Mau
Vào %LocalAppData%\CapCut\User Data\Projects\com.lveditor.draft\Test_Mau\
Copy draft_content.json và draft_meta_info.json ra ngoài
File đó là nguồn sự thật duy nhất. Generator phải sinh ra thứ khớp với nó, không khớp với spec.
§4. Kiến trúc code — đã chốt
Logic → src/lib/capcutDraft.ts
Giao diện → giữ nguyên public/auto-capcut-builder.html (đã có từ v1.0.0, đã khai trong manifest.config.ts:127, Tools.tsx:155 đã trỏ tới)
Lý do không phải thẩm mỹ: buildCapCutProject() là hàm thuần, không chạm chrome.* ⇒ unit-test được ngay, không cần cả fake-browser.
⇒ Đề xuất: lấy capcutDraft.ts làm bài test ĐẦU TIÊN của dự án (ý tưởng #12). Chạy được bằng node trần, không cần dựng hạ tầng gì thêm. Nhét logic vào file HTML là vứt đúng món quà đó đi.
§5. 🔶 HAI ĐIỂM CÒN TREO — cần Antigravity trả lời
Hai điểm này không thuộc nhánh CapCut; chúng thuộc đề xuất cầu nối n8n (#15) của Antigravity, và chưa được phản hồi.
Đ1 — "Đổ thẳng vào Hàng đợi Gen" là cách nói nguy hiểm
Antigravity viết: "đọc file prompts.csv … đổ thẳng vào Hàng đợi Gen".
Đây chính xác là lối tư duy đã đẻ ra graphToJob.ts — một đường dựng job thứ hai, phân kỳ dần, sinh 3 lỗi mà build không bắt được. CLAUDE.md Điều 3 tồn tại vì chuyện đó.
Yêu cầu: prompts.csv → FormInputData → buildFlowJobsFromForm(). Trình đọc inbox chỉ được sinh ra FormInputData, tuyệt đối không tự dựng QueueJob. Kèm validate bằng zod (đã có sẵn trong dependencies) — file từ n8n là dữ liệu bên ngoài, không tin.
Đ2 — Nút "Nạp từ Folder Inbox" đặt trong tab Gen là chạm Gen
Antigravity đề xuất thêm nút vào tab Gen và tab Workflow. Nhưng lệnh chốt chặn của dự án:
git diff --stat -- src/tabs/gen src/services/flowProtocol.ts src/hooks/useQueueRunner.ts src/components src/state/store.ts
Thêm nút vào src/tabs/gen ⇒ lệnh này không rỗng ⇒ RR nhảy từ 1 lên 3 ⇒ theo Điều 2, không đủ điều kiện vào Phase 1 khi lưới test (#12) chưa dựng.
Đề xuất: v1.1 đặt nút ở tab Công cụ (giữ RR 1, ship ngay được). Khi #12 xong thì chuyển lối vào sang Gen như một thay đổi riêng, nhỏ, có test đỡ lưng.
§6. Trạng thái hiện tại của trang builder — đừng bật cờ
public/auto-capcut-builder.html hiện có đầy đủ giao diện (tên dự án, thư mục, chọn transition, nhạc nền) và một nút lớn "🚀 Tự Động Tạo File Draft CapCut Ngay". Toàn bộ phần thực thi của nút đó là:
alert("✅ Đã tạo thành công draft dự án CapCut: ...")
Không ghi file, không gọi API. Nó nói dối người dùng — nặng hơn "nút trang trí" mà Điều 6 cấm.
Chuyện này đã được xử lý đúng ngày 2026-09-10: SHOW_UNFINISHED_TOOLS = false (Tools.tsx:113) ẩn 4 thẻ chưa nối xong, ẩn chứ không xoá (ĐIỀU 0), kèm chú thích đầy đủ tại Tools.tsx:92.
⇒ KHÔNG phải chốt chặn cho v1.0.5. ⇒ Luật: không bật SHOW_UNFINISHED_TOOLS = true cho tới khi nút chạy thật.
Điểm nhỏ còn lại, không phải blocker: auto-capcut-builder.html vẫn nằm trong web_accessible_resources nên vẫn mở được bằng URL trực tiếp dù thẻ đã ẩn. Reviewer bấm qua UI chứ không liệt kê resource ⇒ rủi ro thấp. Ghi lại để không ai bất ngờ.
§7. Thứ tự thi công trong cửa sổ chờ duyệt
Bước 0 — Nộp v1.0.5 · anh Hùng · chặn tất cả Bước 1 — Lấy file CapCut draft thật (Y4, 5 phút) · anh Hùng · chặn nhánh CapCut Bước 1b — Kiểm chứng items[0].filename (Y1-bis, một dòng log) · ai chạy batch trước · quyết định UX của Y1 Bước 2 — Nâng resultFolder.ts lên readwrite · ⚠️ nút thắt, một người làm · dùng chung cho cả hai tính năng Bước 3a — src/lib/capcutDraft.ts + test viết trước · song song Bước 3b — Trình đọc inbox + schema zod → FormInputData · song song Bước 4 — UI: nút ở tab Công cụ cho cả hai · sau 3a/3b
Cảnh báo phối hợp: Bước 2 là nút thắt — hai người cùng sửa resultFolder.ts sẽ đụng nhau. Ai làm trước thì báo, người kia nhánh ra từ đó.
§8. Chấm điểm
CapCut Draft Export — GT 5 · RR 1 · CS M · PT 0
CS = M chứ không phải S: Y1/Y1-bis (đường dẫn), Y2 (duration + xử lý Infinity), Y4 (đối chiếu file thật) đều là công việc thật. Nhưng RR 1 thì đúng — không chạm src/tabs/gen, flowProtocol.ts, useQueueRunner.ts một dòng nào.
Vị trí trong lộ trình: thay chỗ #1 (Studio Ghép Cảnh) trong 02 — Lộ trình & Backlog. mediabunny xuống hạng thành nhánh headless.
§9. Định nghĩa "XONG"
Chỉ được gọi là xong khi đủ cả bốn:
npm test && npm run typecheck && npm run build — pass 100%, 0 lỗi
git diff --stat -- src/tabs/gen src/services/flowProtocol.ts src/hooks/useQueueRunner.ts src/components src/state/store.ts — rỗng
Draft sinh ra mở được trong CapCut thật, đủ số clip, đúng thứ tự, không hở khoảng trống, không đè nhau
capcutDraft.ts có unit test chạy được
Chưa đủ 4 thì SHOW_UNFINISHED_TOOLS vẫn là false.