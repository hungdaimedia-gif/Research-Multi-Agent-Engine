# RESEARCH & MULTI-AGENT ENGINE

> Trung tâm điều phối nghiên cứu, lập kế hoạch kiến trúc và phối hợp tác vụ đa Agent (Claude, Antigravity và Anh Hùng).

---

## 1. VAI TRÒ CÁC THÀNH VIÊN
- **Chủ dự án (Product Owner):** **Anh Hùng** — Quyết định cuối cùng về tính năng và phê duyệt bản build.
- **Kiến trúc sư trưởng (Tech Lead & Researcher):** **Claude** — Nghiên cứu hệ sinh thái, đối thủ, viết tài liệu đặc tả kỹ thuật (Specs), phản biện và lập kế hoạch.
- **Kỹ sư thi công (Senior Engineer & Implementer):** **Antigravity** — Thực thi mã nguồn, build, test, debug, quản lý Git và nộp Store.

---

## 2. CẤU TRÚC THƯ MỤC
```text
Research-Multi-Agent-Engine/
├── docs/                # Các tài liệu nghiên cứu gốc (00 - 07 từ claude-engine)
├── specs/               # Các bản đặc tả kỹ thuật chi tiết đã được duyệt để thi công
├── tasks/               # Quản lý trạng thái tác vụ giữa các Agent (AGENT_BOARD.json)
└── README.md            # Tài liệu tổng quan này
```

---

## 3. NGUYÊN TẮC PHỐI HỢP (MULTI-AGENT PROTOCOL)
1. **Append-Only (Chỉ cộng thêm file mới):** Mỗi tài liệu nghiên cứu/phản biện tạo một file mới có số thứ tự tăng dần (ví dụ: `04`, `05`, `06`...). Tuyệt đối không sửa đè nội dung file của Agent khác.
2. **File Khóa Trạng Thái (`tasks/AGENT_BOARD.json`):** Agent nào đang nắm cờ `in_progress` thì toàn quyền thực thi, Agent khác không can thiệp để tránh xung đột mã nguồn.
3. **Thực nghiệm thắng suy luận:** Mọi quyết định kiến trúc phải được kiểm chứng bằng thực nghiệm terminal/mã chạy thật.
