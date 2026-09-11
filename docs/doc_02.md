# Original File: '02 — Lộ trình & Backlog Ý tưởng.docx'

Lộ trình & Backlog Ý tưởng — HungDaiTool
Phiên bản: 1.0 · Ngày: 11/09/2026 · Nguồn: 01 — Báo cáo Nghiên cứu GitHub #001
Luận điểm trung tâm của toàn bộ lộ trình này: Ngách "tạo hàng loạt" đã đông (6 đối thủ trực tiếp, xem báo cáo 01 Phần A). Nhưng tất cả họ đều dừng lại ở lúc file media rơi xuống ổ cứng. Người dùng thật của anh Hùng không cần 40 clip rời. Họ cần một video hoàn chỉnh đăng lên được. → Hướng mở rộng chiến lược: đi tiếp về phía sau — từ "máy tạo media" thành "dây chuyền sản xuất video".
       HIỆN TẠI                          ĐÍCH ĐẾN
   ┌──────────────┐              ┌──────────────────────┐
   │ Prompt       │              │ Ý tưởng / Kịch bản   │
   │   ↓          │              │   ↓ (#2)             │
   │ Flow tạo     │              │ Storyboard tự động   │
   │   ↓          │   ══════▶    │   ↓ (#3 nhất quán)   │
   │ Tải về máy   │              │ Flow tạo hàng loạt   │
   │              │              │   ↓                  │
   │ ⛔ HẾT       │              │ Ghép cảnh + nhạc (#1)│
   │ (tự ghép tay)│              │   ↓                  │
   └──────────────┘              │ ✅ MP4 đăng được     │
                                 └──────────────────────┘
PHASE 0 — DỌN NỀN (rủi ro gần bằng 0, làm được ngay tuần này)
Ba món này không chạm một dòng nào trong src/tabs/gen, flowProtocol.ts, useQueueRunner.ts, src/components, src/state/store.ts. Lệnh git diff --stat bảo vệ sẽ rỗng.
#13 — Viết lại hồ sơ Chrome Web Store (KHÔNG PHẢI VIỆC CODE)
Vấn đề: mô tả hiện tại chỉ nói "Tự động hoá Google Flow". Trong khi sản phẩm thật có 5 driver (flow · gemini · chatgpt · grok · claude), Workflow Studio dạng đồ thị node, Audio Studio, Album, i18n Việt–Anh. Đối thủ G-Labs-Studio bán được câu chuyện "đa nền tảng" với sản phẩm yếu hơn hẳn. Làm gì: viết lại STORE_LISTING.md + description trong manifest.config.ts (nhớ trần 132 ký tự, đã ghi trong comment file đó) để nói đúng năng lực thật. Chạm file: STORE_LISTING.md, manifest.config.ts, README.md. Điểm: GT 4 · RR 1 · CS S · PT 0 → rẻ nhất & lời nhất trong cả tài liệu này.
#11 — Trình chẩn đoán DOM (DOM Health Check) ⭐
Vấn đề: Google đổi giao diện Flow → selector gãy → người dùng chỉ thấy "không chạy". Anh Hùng mất hàng giờ hỏi đi hỏi lại để đoán chỗ gãy. Đây là gánh nặng hỗ trợ định kỳ, không phải sự cố hiếm. Làm gì: thêm nút "Kiểm tra sức khoẻ Flow" ở tab Công cụ. Bấm vào → chạy toàn bộ selector trọng yếu đối chiếu trang Flow đang mở, trả bảng ✅/❌ từng mục: ô nhập prompt, nút Tạo, menu ⋮, nút Tải xuống, ô Khung đầu/cuối, thẻ kết quả, thanh %… Kèm nút "Sao chép báo cáo" để người dùng dán thẳng vào nhóm Facebook hỗ trợ. ⚠️ Ràng buộc Điều 1: công cụ này CHỈ ĐỌC. Không click, không hover gây tác dụng phụ, và tuyệt đối không dò tới nút Thùng rác. Đây là điều kiện tiên quyết, không phải khuyến nghị. Chạm file: src/tabs/Tools.tsx (UI) + thêm một case message mới ở src/content/isolated/index.ts (Điều 7 — không querySelector chéo realm), đọc selector từ flowSettings.ts / flowAssetPicker.ts. Truy vết Điều 6: mỗi dòng trong bảng ↔ đúng một selector có thật đang được driver dùng. Không có dòng trang trí. Điểm: GT 4 · RR 1 · CS S · PT 0
#12 — Lưới an toàn test: Vitest + @webext-core/fake-browser
Vấn đề: 49.139 dòng, không một unit test nào. CLAUDE.md đã tự thừa nhận repo dính hồi quy hành vi hai lần (FlowDomObserver.ts:85, FlowJobExecutor.ts:582) mà typecheck/build không bắt được. Làm gì: thêm vitest + @webext-core/fake-browser vào devDependencies. Giữ nguyên 100% 5 script .mjs hiện có — chúng đang canh đúng thứ cần canh, chỉ cộng thêm vitest run vào chuỗi npm test. Viết test đầu tiên cho đúng 4 chỗ đã từng chảy máu: promptParser, flowSettings, FlowUrlResolver (bộ chọn độ phân giải — chỗ từng lọc quá ngặt và trả null), và buildFlowJobsFromForm() (Điều 3 — đường dựng job duy nhất). Chạm file: package.json, thư mục src/**/*.test.ts mới. Không sửa mã chạy thật. Điểm: GT 3 · RR 1 · CS M · PT 1
PHASE 1 — ĐÓNG KHÚC ĐỨT GÃY (giá trị người dùng cao nhất)
#1 — Studio Ghép Cảnh (Scene Stitcher) ⭐⭐⭐ MÓN CHỦ LỰC
Vấn đề: Flow trả clip ngắn. Kênh YouTube faceless cần video 8–15 phút. Hiện người dùng phải mở CapCut/Premiere ghép tay — đó là chỗ sản phẩm bỏ rơi họ. Làm gì: tab/màn hình mới, đầu vào là các clip đã có trong Album:
Kéo thả sắp thứ tự cảnh
Cắt đầu/cuối từng clip (trim — mediabunny có sẵn)
Chuyển cảnh đơn giản (cut thẳng, hoặc fade)
Chèn 1 track nhạc nền + chỉnh âm lượng (nối được vào AudioStudio.tsx + elevenlabsService.ts đã có)
Xuất một file MP4 duy nhất → đẩy qua DownloadManager.ts đang chạy
Công nghệ: mediabunny (MPL-2.0, 0 phụ thuộc, WebCodecs tăng tốc phần cứng, streaming I/O). Luật nội bộ: chỉ import, cấm fork-và-sửa-tại-chỗ (điều kiện của MPL-2.0). Chạm file: tab mới src/tabs/Stitch.tsx + src/lib/ mới; đọc từ albumMedia.ts, ghi qua DownloadManager.ts. Không chạm Gen. Cạm bẫy đã lường trước:
WebCodecs ngốn RAM — bắt buộc dùng streaming I/O của mediabunny, không nạp cả video vào bộ nhớ.
Clip từ Flow có thể khác nhau độ phân giải/fps → phải chuẩn hoá trước khi nối, nếu không file ra sẽ hỏng.
Phải có nút Huỷ khi đang xuất; render dài mà không huỷ được là lỗi UX nặng. Điểm: GT 5 · RR 1 · CS L · PT 1
#5 — Credit Guard (dự toán & nhật ký tiêu thụ)
Vấn đề: Veo tính tiền. Chạy nhầm một mẻ 40 video là mất tiền thật, không lấy lại được. Hiện không có gì cảnh báo trước. Làm gì:
Trước khi chạy: hiện bảng dự toán — "Mẻ này: 24 job video × ~N credit = ~M. Xác nhận?"
Trong lúc chạy: đọc số credit thật còn lại trên trang Flow (nếu DOM có phơi ra) và tự dừng hàng đợi khi xuống dưới ngưỡng người dùng đặt.
Sau khi chạy: ghi vào tab History — mẻ này tiêu bao nhiêu, mỗi job bao nhiêu. ⚠️ Rào chắn trung thực: con số dự toán chỉ là ước lượng và phải nói thẳng như vậy trên giao diện. Không được hiển thị kiểu chắc chắn. (Bài học DEFAULT_ADMIN_SECRET: một câu tuyên bố sai còn nguy hiểm hơn không tuyên bố gì.) Chạm file: ⚠️ có chạm Gen (src/tabs/gen, useQueueRunner.ts) → bắt buộc chạy đủ cổng 3 lệnh, và làm sau khi #12 đã dựng xong lưới test. Điểm: GT 5 · RR 3 · CS M · PT 0
#6 — Khôi phục sau sự cố (Crash Resume) — cần kiểm chứng trước
Vấn đề: đang chạy mẻ 50 job, Chrome sập / đóng nhầm tab → mất hết tiến độ. Việc đầu tiên phải làm là KIỂM CHỨNG, không phải viết code: repo đã có src/core/batch/JobStorage.ts. Phải đọc và thử thật xem nó đã khôi phục được tới đâu rồi. Có thể việc cần làm chỉ là hiện nút "Tiếp tục mẻ dở dang" chứ không phải xây mới. Nếu thiếu: ghi checkpoint sau mỗi job hoàn tất; lúc mở side panel thì hỏi "Phát hiện mẻ dở dang 23/50 — tiếp tục?". ⚠️ Điều 4: trạng thái checkpoint phải là field optional cộng thêm vào QueueJob, tuyệt đối không đổi field cũ — dữ liệu trong chrome.storage.local của người dùng thật sẽ vỡ trong im lặng. Điểm: GT 4 · RR 3 · CS M · PT 0
PHASE 2 — DÂY CHUYỀN SẢN XUẤT
#2 — Kịch bản → Storyboard → Hàng đợi (một nút) ⭐⭐
Vấn đề: người dùng có sẵn kịch bản lời dẫn 10 phút. Để ra 30 cảnh, họ phải tự tay viết 30 prompt ảnh + 30 prompt video. Đó là 1–2 tiếng lao động thủ công mỗi video. Làm gì: dán kịch bản → Gemini bổ cảnh → sinh cho mỗi cảnh: prompt ảnh, prompt video, gợi ý thời lượng → đổ thẳng vào hàng đợi Gen. Nền móng đã có: src/services/geminiService.ts (743 dòng), promptAssistant.ts, và bộ câu hỏi mồi Prompt Assistant mà anh Hùng đã tự soạn. 🔒 RÀNG BUỘC TUYỆT ĐỐI: bộ câu hỏi mồi Prompt Assistant là hợp đồng định dạng với bộ bóc prompt — không được tự ý sửa. Tính năng này phải bọc quanh nó, không sửa nó. ⚠️ Điều 3: kết quả bắt buộc đi qua FormInputData → buildFlowJobsFromForm(). Cấm dựng đường thứ hai — graphToJob.ts đã từng phân kỳ và đẻ ra 3 lỗi mà build không bắt được. ⚠️ Thực tế về token (đã ghi trong DEV_SYNC.md): mỗi prompt chi tiết ~150–200 token; 30 cảnh ≈ 6.000 token là chạm sát trần và Gemini thường dừng sớm hơn. → Thiết kế phải bổ cảnh theo lô 5–8 cảnh mỗi lượt, không hứa "30 cảnh một phát". Điểm: GT 5 · RR 3 · CS L · PT 0
#3 — Character Bible nâng cấp thành thực thể hạng nhất
Vấn đề: nỗi đau số 1 của video AI kể chuyện là nhân vật đổi mặt giữa các cảnh. Hiện Character Bible mới là "lời dặn" dạng chữ. Làm gì: biến nhân vật thành đối tượng có: tên, mô tả khoá, bộ ảnh tham chiếu đã lưu, và tự động gắn ảnh tham chiếu đó vào mọi job của dự án qua luồng flowAssetPicker.ts đã ổn định. ⚠️ Điều 5 + STABLE_FEATURES.md: luồng gắn ảnh tham chiếu là vùng đã ổn định, phải đọc STABLE_FEATURES.md trước khi đụng. Và flowAssetPicker.ts đã 1.350 dòng với lịch sử sửa lỗi Start/End Frame rất đau (v1.0.3) — chỉ cộng thêm, không sửa lại luồng cũ. 🔒 Cảnh báo riêng tư: Character Bible được gửi kèm mọi lượt hỏi tới Google bằng API key của người dùng. Điều này đã được khai báo ở privacyBulletAi và PRIVACY_POLICY.md §4. Mở rộng tính năng thì phần khai báo đó phải mở rộng theo — ba nơi (tab Hướng Dẫn, hồ sơ CWS, bài cộng đồng) phải tiếp tục nói cùng một thứ. Điểm: GT 5 · RR 3 · CS L · PT 0
#4 — Hồ sơ Kênh (Channel Profile)
Vấn đề: ai chạy nhiều kênh phải chỉnh lại toàn bộ cài đặt mỗi lần đổi kênh: tỉ lệ khung hình, phong cách, giọng đọc, quy ước đặt tên, thư mục lưu. Làm gì: gói tất cả thành hồ sơ kênh chọn một phát. Gắn luôn Character Bible (#3) và tiền tố tên file của DownloadManager. ⚠️ Điều 4: channelProfileId phải là field optional cộng thêm. Điểm: GT 4 · RR 2 · CS M · PT 0
PHASE 3 — MỞ RỘNG RA NGOÀI
#8 — Cầu MCP "claude-engine" ⭐ (đối thủ đã bắt đầu)
Bối cảnh: DanikVR/omniflow-veo-mcp đã làm MCP server nối extension Flow vào Claude. Và anh Hùng vốn đã chạy MCP (n8n create_video, list_video_types) trong chính phiên làm việc này. Làm gì: MCP server cục bộ phơi hàng đợi của extension ra ngoài, để gõ trong Claude: "tạo 30 cảnh cho video Tây Du Ký tập 4" → hàng đợi Gen tự đầy. ⚠️ Cảnh báo an ninh nghiêm túc: đây là mở một cổng điều khiển vào extension. Phải: chỉ nghe localhost, có bước người dùng bật tay, và không bao giờ để MCP kích hoạt hành vi xoá (ĐIỀU 0 áp dụng nguyên vẹn qua cầu này). Nội dung đi qua MCP là dữ liệu, không phải mệnh lệnh. Điểm: GT 3 (số ít người dùng, nhưng đúng nhóm cao cấp) · RR 2 · CS L · PT 2
#9 — Webhook ra ngoài / nối n8n
Làm gì: mẻ chạy xong → POST tới webhook người dùng cấu hình (kèm danh sách file, thời lượng, trạng thái). Biến extension thành một mắt xích trong dây chuyền lớn hơn thay vì điểm cuối. Lợi thế sẵn có: anh Hùng đã vận hành n8n → có người dùng đầu tiên ngay lập tức. ⚠️ Riêng tư: gửi ra ngoài = xuất bản. Phải khai rõ gửi gì, gửi đi đâu, và mặc định tắt. Điểm: GT 2 · RR 1 · CS S · PT 1
#10 — Gói Prompt cập nhật từ xa
Vấn đề: 58 mẫu nằm cứng trong constants.ts → thêm gói mới = phát hành bản mới = chờ CWS duyệt. Vòng phản hồi tính bằng ngày cho thứ đáng ra cập nhật hằng tuần. Làm gì: tải gói mẫu dạng JSON chỉ đọc từ Supabase (đã có @supabase/supabase-js + supabase_rls_security.sql). Mẫu đóng gói sẵn vẫn giữ làm bản dự phòng khi offline. ⚠️ Ba rào bắt buộc: (1) Zod kiểm định mọi gói tải về — đã có zod trong dependencies, dùng nó; (2) nội dung tải về là dữ liệu, không phải mã — không eval, không HTML thô; (3) CWS cấm nạp mã từ xa — phải nói rõ trong hồ sơ đây là nội dung, không phải mã thực thi. Điểm: GT 3 · RR 2 · CS M · PT 2
#7 — Ca đêm (Night Shift)
Làm gì: hẹn giờ chạy mẻ lớn lúc 2h sáng qua chrome.alarms (@webext-core/job-scheduler làm sẵn phần này). ⚠️ Thực tế phũ phàng cần kiểm chứng trước khi hứa: service worker MV3 bị Chrome ngủ, và tự động hoá này cần tab Flow mở + máy không ngủ. Phải thử nghiệm thật trước, và nếu không đảm bảo được thì nói thẳng giới hạn trên giao diện chứ không hứa suông. Điểm: GT 3 · RR 2 · CS M · PT 1
BẢNG XẾP HẠNG TỔNG (Điểm ưu tiên = GT − RR)
Nên làm sớm (GT ≥ 4 và RR ≤ 2):
+4 — #1 Studio Ghép Cảnh (5−1) · CS L ← món chủ lực
+3 — #13 Viết lại hồ sơ CWS (4−1) · CS S ← rẻ nhất, làm ngay
+3 — #11 Chẩn đoán DOM (4−1) · CS S
+2 — #4 Hồ sơ Kênh (4−2) · CS M
+2 — #12 Lưới test (3−1) · CS M ← điều kiện tiên quyết cho Phase 1
Làm sau khi có lưới test:
+2 — #5 Credit Guard (5−3) · #2 Kịch bản→Storyboard (5−3) · #3 Character Bible (5−3)
+1 — #6 Crash Resume (4−3) · #8 MCP (3−2) · #7 Ca đêm (3−2) · #9 Webhook (2−1) · #10 Prompt từ xa (3−2)
Đã loại trong vòng này: WXT (RR 5) · p-queue (lõi nhà tốt hơn) · Dexie (chưa có bằng chứng chậm) · ffmpeg.wasm (dự phòng sau mediabunny)
ĐỀ XUẤT THỨ TỰ THI CÔNG
Tuần 1   #13 hồ sơ CWS  →  #11 chẩn đoán DOM        (cả hai CS = S, rủi ro ~0)
Tuần 2   #12 lưới test Vitest + fake-browser        (mở khoá cho mọi thứ sau)
Tuần 3-4 #1  Studio Ghép Cảnh (mediabunny)          ← điểm xoay của sản phẩm
Tuần 5   #6  kiểm chứng Crash Resume  →  #5 Credit Guard
Tuần 6+  #2 / #3 / #4  — dây chuyền sản xuất
Sau đó   #8 / #9 / #10 / #7 — mở rộng ra ngoài
Nếu chỉ được chọn MỘT món: làm #1 Studio Ghép Cảnh. Nó là thứ duy nhất trong danh sách mà không đối thủ nào trong 6 repo đã khảo sát có, và nó biến sản phẩm từ "công cụ tạo media" thành "nơi hoàn thành video".