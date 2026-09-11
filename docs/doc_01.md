# Original File: '01 — Báo cáo Nghiên cứu GitHub #001.docx'

Báo cáo Nghiên cứu GitHub #001
Ngày quét: 11/09/2026 · Phạm vi: đối thủ cùng ngách + thư viện nền tảng cho Chrome MV3 + xử lý media trên trình duyệt Cơ sở đối chiếu: đã đọc trực tiếp package.json, manifest.config.ts, cây src/ (165 file / 49.139 dòng), README.md, CHANGELOG.md, DEV_SYNC.md của repo thật.
⚠️ Ghi chú về số liệu: số sao (★) dưới đây chỉ ghi ở nơi mình đọc được chính xác qua GitHub API. Chỗ nào không đọc được thì mình ghi chỉ số khác (số repo phụ thuộc, số contributor) chứ không bịa số sao.
PHẦN A — BỐI CẢNH CẠNH TRANH
Ngách "tự động hoá Google Flow / Veo hàng loạt" đã có người làm. Đây là tin tốt (thị trường có thật) và tin xấu (không còn là đại dương xanh).
A1. trgkyle/veo-automation-user-guide — 43★
Đối thủ gần nhất. Mô tả: "Chrome extension that automates batch video and image generation on Google Flow AI VEO3. Process multiple prompts simultaneously, configure your workflow, and automatically download generated content." → Trùng gần như 1:1 với mô tả ngắn của hungdai-flow. Đáng chú ý: repo công khai chỉ là user guide (tài liệu), mã nguồn đóng — cùng mô hình phát hành với mình.
A2. Shivanshu85/Google-Flow-Automation — 9★
Nhắc đích danh Veo và Nano Banana Pro, nhấn vào "queue prompts, batch process, bulk download". Nhắm content creator & marketer.
A3. trgkyle/vids-automation-user-guide — 3★
Cùng tác giả A1 nhưng đánh sang Google Vids, và có thêm voice-over tự động. → Tín hiệu: đối thủ đang mở rộng sang lồng tiếng. Mình đã có elevenlabsService.ts + AudioStudio.tsx, tức là đang cùng hướng — nhưng họ tuyên bố trước.
A4. duckmartians/G-Labs-Studio — 1★
"One desktop app to batch-generate images & videos across every major AI — Google Flow (Veo 3.1, Omni Flash, Nano Banana), Grok Imagine, Meta AI and ChatGPT GPT Image 2." → Chiến lược desktop app đa nền tảng. Mình đã có 5 driver (flow · gemini · chatgpt · grok · claude) nên lợi thế đa nền tảng của mình mạnh hơn họ, chỉ là chưa marketing đúng mức.
A5. amzbase-com/veo-automation-extension — 0★
Bán điểm khác biệt bằng tài liệu đa ngôn ngữ. Mình vừa ra i18n Việt–Anh ở 1.0.4 → ngang bằng.
A6. DanikVR/omniflow-veo-mcp — 0★ ⭐ TÍN HIỆU CHIẾN LƯỢC
"MCP server for Google Flow (Veo 3 / Omni 1.1): batch video generation, extend, scenes and references from Claude. Bridge for the OmniFlow Chrome extension."
→ Có người đã nối extension Flow vào Claude qua MCP. Đây là hướng đi mà mình có lợi thế lớn nhưng chưa động tới: anh Hùng vốn đã chạy MCP (n8n create_video, list_video_types) trong chính phiên làm việc này. Xem ý tưởng #8 — MCP Bridge ở tài liệu 02.
A7. jeffstric/ZJT — 218★ (tham chiếu, không cùng ngách)
Nền tảng mã nguồn mở làm phim ngắn chuyên nghiệp: kịch bản → storyboard → tổng hợp video, tự động hoá toàn chuỗi. → Đây là hình dạng tương lai của cái mình đang làm. Họ mạnh ở "chuỗi sản xuất", mình mạnh ở "tay chân điều khiển Flow thật". Hai thứ ghép được.
Kết luận Phần A
Lợi thế phòng thủ của hungdai-flow không nằm ở "tạo hàng loạt" — chỗ đó đã đông. Nó nằm ở ba thứ đối thủ chưa ai có đủ cả ba:
Slate.js Fiber RPC Bridge — can thiệp đúng data model của Flow chứ không giả lập gõ phím.
5 driver trong một — Flow, Gemini, ChatGPT, Grok, Claude.
Workflow Studio dạng đồ thị node (@xyflow/react) — chưa đối thủ nào trong danh sách có.
→ Chiến lược đề xuất: đừng đua tính năng "batch". Đua ở khúc SAU khi đã có media (ghép cảnh, nhất quán nhân vật, xuất bản) — chỗ đó chưa ai chiếm.
PHẦN B — THƯ VIỆN ỨNG VIÊN
B1. ⭐⭐⭐ mediabunny — Vanilagy/mediabunny
Giấy phép: MPL-2.0 · Phụ thuộc: 0 · Kích thước: nhỏ nhất ~5 kB gzip (tree-shakable)
Nó là gì: bộ công cụ media thuần TypeScript chạy thẳng trong trình duyệt. Đọc/ghi/chuyển đổi MP4, MOV, WebM, MKV, HLS, WAV, MP3, FLAC. Mã hoá & giải mã 25+ codec bằng WebCodecs API có tăng tốc phần cứng. Có API Conversion sẵn: transmux, transcode, resize, xoay, crop, resample, trim. I/O dạng streaming nên xử lý được file lớn mà không nổ RAM.
Vì sao quan trọng với mình: Google Flow trả về clip ngắn. Người dùng thật của anh Hùng (kênh YouTube faceless) cần video 8–15 phút. Hiện tại sau khi tải xong, họ phải mở CapCut/Premiere ghép tay. Đó là khúc đứt gãy lớn nhất của sản phẩm — mình giao "nguyên liệu" chứ chưa giao "món ăn".
mediabunny cho phép ghép/cắt/chèn nhạc/xuất MP4 ngay trong extension, không cần máy chủ, không cần cài gì thêm.
Cảnh báo giấy phép (quan trọng): MPL-2.0 là copyleft cấp file. Dùng như thư viện trong sản phẩm thương mại đóng: được. Nhưng nếu sửa file nguồn của mediabunny thì đúng những file đó phải được công bố lại. → Luật nội bộ: chỉ import, tuyệt đối không fork-và-sửa-tại-chỗ.
Chấm điểm: GT 5 · RR 1 (tính năng mới hoàn toàn, không chạm Gen) · CS L · PT 1 → Kết luận: ỨNG VIÊN SỐ 1. Ưu tiên cao nhất vòng này.
B2. ffmpeg.wasm — ffmpegwasm/ffmpeg.wasm
Giấy phép: MIT · 43 bản phát hành · 72 contributor · 83,7% mã C
Nó là gì: cổng FFmpeg sang WebAssembly.
Vì sao mình NÊN CÂN NHẮC KỸ TRƯỚC KHI DÙNG — ba vấn đề cụ thể với bối cảnh Chrome MV3 + Chrome Web Store:
Bản đa luồng (core-mt) cần SharedArrayBuffer, mà SharedArrayBuffer cần trang được cross-origin isolated (header COOP/COEP). Trang side panel của extension không dễ đạt điều kiện đó. Rơi về bản đơn luồng thì chậm hơn nhiều.
Kích thước file .wasm rất lớn (hàng chục MB). Đây là vấn đề trực tiếp với gói nộp CWS — xem §5 câu hỏi 2 trong Điều lệ.
Nó là cả một FFmpeg, trong khi nhu cầu thật của mình chỉ là "nối n clip + chèn 1 track nhạc + xuất MP4".
Chấm điểm: GT 4 · RR 1 · CS L · PT 2 → Kết luận: PHƯƠNG ÁN DỰ PHÒNG. Chỉ dùng nếu mediabunny thiếu một codec cụ thể mà Flow trả về. Không phải lựa chọn mặc định.
B3. ⭐⭐ Dexie.js — dexie/Dexie.js
Giấy phép: Apache-2.0 · README ghi "100.000 website/app đang dùng"
Nó là gì: lớp bọc IndexedDB, có khai báo schema kèm phiên bản, truy vấn kiểu .where(), thao tác hàng loạt, và đi vòng qua các lỗi triển khai IndexedDB của từng trình duyệt.
Vì sao quan trọng với mình: repo đã dùng IndexedDB ở 4 chỗ — src/lib/albumMedia.ts, assetLibrary.ts, audioLibrary.ts, resultFolder.ts — nhưng dùng API thô. Khi Album tích tới hàng trăm video, API thô sẽ cho ba loại đau: nâng cấp schema thủ công, truy vấn chậm, và lỗi trình duyệt khó tái hiện.
Cảnh báo: đây là thay thế hạ tầng lưu trữ đang chạy thật, tức là đụng vào dữ liệu người dùng đã có. Bắt buộc phải có đường di trú đọc-được-cả-hai-định-dạng, và tuyệt đối không xoá dữ liệu cũ (ĐIỀU 0).
Chấm điểm: GT 2 (người dùng không thấy trực tiếp) · RR 3 · CS M · PT 1 → Kết luận: HOÃN. Chỉ làm khi Album thật sự chậm. Chưa có bằng chứng chậm thì chưa động — đây đúng nghĩa "sửa thứ chưa hỏng".
B4. ⭐⭐⭐ webext-core — aklinker1/webext-core
Giấy phép: MIT · 88 bản phát hành · ~3.000 repo phụ thuộc
Bộ gồm nhiều gói nhỏ, 3 gói dính thẳng vào điểm yếu hiện tại của mình:
@webext-core/fake-browser — ứng viên mạnh nhất trong nhóm
Bản cài đặt giả lập toàn bộ API chrome.* chạy trong bộ nhớ, dành riêng cho việc test.
Phát hiện thẳng thắn về repo mình: npm test hiện chạy 5 script tự viết (check-secrets, scan-orphan-fields, check-frame-to-frame, check-node-image-chain, test-shared). Chúng là lưới an toàn tốt cho đúng vùng chúng canh — nhưng không có bộ chạy test thật, không có một unit test nào cho 49.139 dòng code. CLAUDE.md đã tự thừa nhận: "typecheck và build chỉ bắt lỗi KIỂU — chúng không hề bắt được hồi quy hành vi, mà repo này đã dính hai lần".
Lý do gốc khiến chưa test được là: hầu hết logic đều chạm chrome.*, mà chrome.* không tồn tại trong Node. fake-browser gỡ đúng nút thắt đó.
@webext-core/messaging
Lớp bọc type-safe cho messaging. Mình đang có 4 realm và src/content/isolated/index.ts đã phình tới 1.053 dòng chủ yếu vì bảng switch phân loại message. Type-safe messaging biến lỗi sai tên message từ lỗi lúc chạy thành lỗi lúc biên dịch.
@webext-core/job-scheduler
Lập lịch tác vụ lặp qua Alarms API → nền tảng cho ý tưởng #7 Hẹn giờ chạy đêm.
Chấm điểm (fake-browser + Vitest): GT 3 (gián tiếp, nhưng cứu khỏi hồi quy mà người dùng chịu trận) · RR 1 (chỉ thêm devDependency, không đụng mã chạy thật) · CS M · PT 1 → Kết luận: LÀM SỚM. Rẻ nhất, an toàn nhất, và trả nợ đúng chỗ repo đã đau hai lần.
B5. webext-bridge — serversideup/webext-bridge
Giấy phép: MIT
Cùng bài toán với @webext-core/messaging nhưng có hỗ trợ ngữ cảnh window (main world) — đúng chỗ slateBridge.ts / assetBridge.ts / interceptor.ts đang chạy.
→ Kết luận: ĐỂ DÀNH. Nếu quyết làm type-safe messaging thì so hai anh này; webext-bridge nhỉnh hơn ở main world, webext-core nhỉnh hơn vì đi kèm fake-browser. Không dùng cả hai.
B6. WXT — wxt-dev/wxt
Giấy phép: MIT · 257 bản phát hành · 3.591 repo phụ thuộc · 290 contributor
"Next-gen framework for developing web extensions — như Nuxt nhưng cho Web Extension." Có HMR thật, entrypoint theo file, auto-import, tự động publish, phân tích bundle, hỗ trợ mọi trình duyệt (mở đường sang Edge/Firefox).
Vì sao đáng để mắt: hai cái bẫy tốn thời gian nhất mà CLAUDE.md đã ghi lại đều là bẫy của công cụ build, không phải bẫy logic:
"Vite chia chunk: Gen.tsx build vào dist/assets/index.html-<hash>.js, không phải index-<hash>.js"
Và trong bộ nhớ dự án: npm run dev biến dist/ thành vỏ CRXJS DEV MODE và làm chết side panel của extension đang cài.
WXT xử lý đúng lớp vấn đề này bằng dev mode riêng biệt.
Nhưng: đây là thay toàn bộ hệ thống build của một dự án 49k dòng đang chạy sản xuất hằng ngày. Vi phạm tinh thần Điều 2 (Gen luôn thắng) nếu làm vội.
Chấm điểm: GT 1 (người dùng không thấy gì) · RR 5 · CS XL · PT 2 → Kết luận: KHÔNG LÀM BÂY GIỜ. Ghi vào Radar, xem lại nếu CRXJS ngừng phát triển hoặc khi có nhu cầu thật sự ra bản Firefox/Edge.
B7. Vitest
Bộ chạy test cho hệ Vite — mà repo đã dùng Vite 5, nên hoà nhập gần như không ma sát: dùng chung vite.config.ts, chung cách phân giải TypeScript.
→ Kết luận: LÀM CÙNG B4. Giữ nguyên 100% 5 script .mjs hiện có (chúng đang canh đúng thứ cần canh — ĐIỀU 0 tinh thần: không gỡ thứ đang bảo vệ mình), chỉ cộng thêm vitest vào chuỗi npm test.
B8. p-queue — sindresorhus/p-queue
Giấy phép: MIT. README nói rõ: "dự án đã hoàn chỉnh tính năng, không có kế hoạch phát triển tiếp".
Hàng đợi Promise có kiểm soát đồng thời và giới hạn tốc độ.
→ Kết luận: KHÔNG DÙNG — và đây là một "không" có chủ ý. src/core/queue/QueueEngine.ts + useQueueRunner.ts của mình đã làm đúng việc này và làm nhiều hơn: chọn/mở tab đích, jitter delay chống rate-limit, 2 lane song song Ảnh/Video, mutex ghi file ở background. Thay bằng p-queue là đánh đổi lõi đã tôi luyện lấy một thư viện tổng quát hơn nhưng biết ít hơn về bài toán — vi phạm thẳng Điều 2.
Thứ duy nhất đáng học từ nó: khái niệm intervalCap (giới hạn N tác vụ mỗi T mili-giây). Đây là mô hình sạch hơn jitter delay ngẫu nhiên nếu sau này cần khai báo hạn mức kiểu "tối đa 10 video/giờ". Học ý tưởng, không nhập thư viện.
PHẦN C — PHÁT HIỆN VỀ CHÍNH REPO MÌNH
Phần này không đến từ GitHub mà từ việc đọc trực tiếp mã nguồn. Ghi lại vì nó quyết định thứ tự ưu tiên.
C1. Không có một unit test thật nào cho 49.139 dòng
Đã nói ở B4. Đây là rủi ro số 1 của dự án ở thời điểm hiện tại, lớn hơn mọi tính năng còn thiếu.
C2. 97 lời gọi chrome.storage.local rải khắp src/
Không có lớp bọc, không có kiểu, không có nơi tập trung khai báo khoá. Điều 4 (không đổi tên/kiểu/ngữ nghĩa field) đang được bảo vệ bằng kỷ luật con người chứ không bằng máy. Một grep sót là một lần vỡ dữ liệu trong im lặng — và DEV_SYNC.md đã ghi đúng nỗi lo này khi bàn về ADVANCED_MODE_KEY.
C3. Ba file quá khổ đang tích nợ
src/tabs/Workflow.tsx — 3.033 dòng
src/tabs/workflow/canvas/WorkflowCanvas.tsx — 1.567 dòng
src/tabs/workflow/canvas/WorkflowAiSidebar.tsx — 1.516 dòng
Cả ba đều thuộc Workflow — tức là có thể tách an toàn mà không chạm Gen (lệnh git diff --stat bảo vệ sẽ vẫn rỗng). Đây là món "dọn nhà" rủi ro thấp nhất có thể làm.
C4. 58 mẫu prompt đang nằm cứng trong constants.ts
Hệ quả: thêm một gói prompt mới = phát hành bản mới = chờ Chrome Web Store duyệt. Vòng phản hồi tính bằng ngày, trong khi đây đúng ra phải là thứ cập nhật được hằng tuần. Xem ý tưởng #10.
C5. Điểm mạnh bị marketing bỏ quên
Mô tả CWS hiện tại chỉ nói "Tự động hoá Google Flow". Nhưng repo có 5 driver (flow · gemini · chatgpt · grok · claude), Workflow Studio dạng node, Audio Studio, Album, i18n song ngữ. Đối thủ A4 bán được câu chuyện "đa nền tảng" với sản phẩm yếu hơn. → Đây là việc sửa chữ, không phải sửa code. Rẻ nhất trong toàn bộ báo cáo này.
PHẦN D — KẾT LUẬN VÒNG #001
Ba việc nên làm trước, xếp theo tỉ lệ giá trị/rủi ro:
Vitest + @webext-core/fake-browser — rẻ, không chạm mã chạy thật, trả nợ đúng chỗ đã đau 2 lần. (GT 3 · RR 1 · CS M)
Studio Ghép Cảnh bằng mediabunny — tính năng mới hoàn toàn, không chạm Gen, và đóng lại khúc đứt gãy lớn nhất của sản phẩm. (GT 5 · RR 1 · CS L)
Trình chẩn đoán DOM (ý tưởng #11) — tự viết, không cần thư viện, giảm mạnh gánh nặng hỗ trợ mỗi khi Google đổi giao diện Flow. (GT 4 · RR 1 · CS S)
Ba việc KHÔNG làm, và lý do:
WXT — đổi build hệ thống đang chạy sản xuất, RR 5.
p-queue — lõi nhà mình đã tốt hơn cho đúng bài toán này.
Dexie — chưa có bằng chứng Album chậm; sửa thứ chưa hỏng là tự tạo rủi ro.
Vòng #002 sẽ quét: thư viện nhất quán nhân vật (character consistency), phụ đề/caption tự động, và khảo sát sâu mô hình MCP bridge của DanikVR/omniflow-veo-mcp.