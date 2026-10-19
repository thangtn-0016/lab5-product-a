# AI-report — LAB-5

> Công cụ: Claude Code (Opus) làm agent điều phối, kèm bộ skill Takumi. Toàn bộ LAB-5 làm trong một phiên ngày 2026-10-17.

## 1. Dùng AI ở công việc nào

| Công việc | Skill / agent | Cách làm |
|---|---|---|
| **Break task** | 3 subagent chạy song song (Claude Code `general-purpose`) | Tôi viết một brief chung: nguồn LAB-4, quy tắc chia (epic = feature, issue = hành vi có ranh giới, sub-issue 0,5–2 ngày), bắt buộc tách ràng buộc đặc thù, ghi chỗ hở spec, định dạng JSON đầu ra. Mỗi agent đọc spec của 4 feature (agent đầu thêm F000) và trả về một file JSON. Một script gộp 3 file, bỏ issue trùng, bổ sung bối cảnh cho sub-issue |
| **Tạo backlog trên GitHub** | Claude Code viết script Python gọi GitHub GraphQL qua `gh` | Tạo repo public, label, 414 issue, nối 401 quan hệ sub-issue, đóng 58 issue đã Done, dựng Project (field Status, Sprint, SWP, Giờ ước lượng), gắn Status / Sprint. Script lưu trạng thái để chạy lại không tạo trùng |
| **Chấm SWP** | Skill **`tkm:estimate-swp`** (+ `tkm:confidence`) | Skill đọc Project (chỉ đọc), chọn 284 việc lá, chia 5 lô cho 5 worker (model sonnet) chấm Scope × Complexity; validator của skill kiểm hợp đồng; skill chọn 5 issue mẫu, **tôi kiểm từng issue** (mức khó → điểm); skill hiệu chỉnh và xuất một báo cáo HTML |
| **Ghi SWP đã chốt vào Project** | Claude Code chạy script `write-swp-hours-sprint2-to-project` | Agent đọc dữ liệu đã kiểm **nhúng trong báo cáo HTML** (không đọc lại output thô của worker), ghi `finalSWP` vào field **SWP** cho 414 item (container = tổng con, khớp báo cáo), ghi **Giờ ước lượng** = SWP × 0,19, và gắn Sprint 2. Skill không tự ghi — đúng ranh giới chỉ đọc của skill |
| **Báo cáo tiến độ** | Claude Code + git log | Burndown dựng từ ngày commit thật của code LAB-3, ánh xạ từng sub-issue Done về commit tương ứng |
| **Viết PREMISE, task-breakdown, estimate, README** | Claude Code | AI soạn từ dữ liệu (backlog JSON, payload SWP, git log); bảng của `task-breakdown.md` sinh bằng script từ cùng dữ liệu đã tạo issue để không lệch với Project. Tôi rà và quyết các điểm chọn hướng |

## 2. Review output của AI

| Bước | AI làm sai / thiếu gì | Phát hiện thế nào | Xử lý |
|---|---|---|---|
| Break task | Hai nhóm cùng tạo issue cho một việc: nới RLS `app_users` (F001-09 và F006-09) | Đọc danh sách chỗ hở spec của 3 nhóm, thấy cùng một lỗ | Gộp về F001-09, chuyển sub-issue seed sang, trỏ lại chỗ hở spec |
| Break task | Hai nhóm chọn ngược nhau cho cùng câu hỏi: kiểm tư cách theo `current_date` (giao hàng) hay ngày nghiệp vụ (chốt giao dịch) | Đối chiếu cột `why` của các chỗ hở spec | Thống nhất thành một giả định: kiểm theo ngày xảy ra sự kiện (PREMISE L5-G01) |
| Break task | Body sub-issue quá ngắn (trung bình ~66 ký tự) — worker chấm SWP sẽ thiếu bằng chứng | Thống kê độ dài body trước khi tạo issue | Script gắn thêm issue cha, mục tiêu, hoạt động, spec tham chiếu vào mọi sub-issue |
| Break task | Một nhóm đánh dấu F010-01, F011-01/02 "đã xây" trái với brief tôi đưa ("F008–F011 không xây") | Agent tự báo trong kết quả | Kiểm lại với git: agent đúng — trang chủ SCR001 (thuộc F010) có từ commit `338098c` (23/09), bảng nhật ký từ migration `0002` (22/09), log lần thử bị chặn từ `0040` (24/09). Brief của tôi sai, giữ kết quả của agent |
| Chấm SWP | 16 việc tài liệu bị gán discipline `Unknown` (rubric không có nhóm "tài liệu"), 9 việc trong đó lại ghi độ tin cậy medium / high | Validator của skill từ chối payload (`Unknown discipline requires low confidence`) | Hạ 9 việc về `low` theo đúng luật hợp đồng, không đổi điểm; ghi vào phần lưu ý của báo cáo HTML |
| Chấm SWP | Một worker báo "DONE 62" cho lô 57 việc | So độ phủ input / output trước khi gộp | Output thực tế đủ đúng 57, không trùng, không thừa — chỉ là con số báo sai |
| Chấm SWP | AI chấm thấp các việc DB nhiều thành phần (seed maker/checker, migration + RLS + view) và việc README chạy local | Kiểm 5 issue mẫu | Nâng nhóm DB 21 việc +2 nấc Scope; README 1 → 6 và #206 3 → 6 chỉ riêng issue đó (chi tiết ở [`estimate.md`](estimate.md) mục 2). Tổng 967 → 1.078 SWP |
| Viết tài liệu | AI viết lý do không có bằng chứng: giải thích 8 ngày không commit bằng "tự kiểm tay, việc ngoài dự án"; viết lý do hiệu chỉnh như lời tôi ("tôi đã làm việc này ở LAB-3") | Rà từng câu có số / lý do so với nguồn | Xoá; chỉ giữ điều kiểm được (git log trống) và đúng nội dung phương án tôi đã chọn |
| Viết tài liệu | AI ghi "44 SWP trượt toàn là test" | Liệt kê lại 14 sub-issue trượt | Sửa: 31 SWP test + 13 SWP implement còn dở + 1 SWP sửa lỗi |
| Tạo Project | Script lỗi: cú pháp Python 3.10 trên máy 3.9; label `bug` đã có sẵn (HTTP 422); thêm item trùng bị GitHub từ chối; Sprint 1 nằm trong `completedIterations` | Lỗi khi chạy | Sửa script, đọc lại item / iteration có sẵn trước khi ghi; chạy lại không tạo trùng |

## 3. Điểm đặc biệt

- **Hiệu quả nhất:** chia break task cho 3 agent song song theo một brief có định dạng JSON cố định — 3 nhóm xong trong ~8 phút, gộp bằng script, rồi tạo 414 issue bằng GraphQL gộp lô trong ~10 phút. Bảng trong `task-breakdown.md` sinh từ cùng file dữ liệu nên số liệu luôn khớp Project.
- **Bước người kiểm có tác dụng thật:** 3 / 5 issue mẫu tôi chỉnh lên, và một chỉnh sửa lan ra 20 việc DB. Không có bước này, estimate thấp hơn 111 SWP (~10%).
- **Lệch so với hợp đồng skill (có ghi lại):** hợp đồng `estimate-swp` muốn worker không có công cụ, dữ liệu nhúng thẳng vào prompt. Với 284 việc (~120 KB), tôi cho worker đọc đúng 2 file (rubric + gói của nó) và ghi 1 file — đỡ token nhưng là một lệch nhỏ, đã ghi trong phần lưu ý của báo cáo HTML. Rủi ro thấp vì nội dung issue do chính tôi tạo.
- **Chi phí:** 3 agent break task ~675 nghìn token; 5 worker chấm SWP ~611 nghìn token (số đọc từ thông báo hoàn thành của từng agent).
- **Hạn chế đã biết:**
  - Hệ số giờ 0,19 giờ/SWP đo trên một sprint không viết test → lạc quan; phải đo lại sau Sprint 2.
  - Burndown Sprint 1 ánh xạ sub-issue về commit theo ngày, không theo giờ; một commit gom nhiều sub-issue.
  - Rubric SWP không có discipline cho việc tài liệu / làm rõ → các việc này rơi vào `Unknown` và độ tin cậy thấp. Lần sau nên ghi rõ trong body issue là việc tài liệu để worker phân loại đúng `documentation`.
- **Cải thiện được nếu có thêm thời gian:** chạy `--level high` để một model thứ hai chấm độc lập các việc rủi ro và lộ chỗ hai bên lệch nhau; viết khung test ngay ở Sprint 1 thay vì để thành nợ.
