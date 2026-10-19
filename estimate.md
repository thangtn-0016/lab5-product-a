# Estimate — SWP và giờ

> Product A · Người thực hiện: Trần Ngọc Thắng · 2026-10-17

## 1. Báo cáo HTML của skill

- Đường dẫn: [`reports/estimate-swp-lab5-product-a-20261017T160606Z.html`](reports/estimate-swp-lab5-product-a-20261017T160606Z.html) (tải về rồi mở bằng trình duyệt; file tự chứa, không gọi mạng).
- Skill: `tkm:estimate-swp` (Takumi), nguồn = [GitHub Project](https://github.com/users/thangtn-0016/projects/2), cờ `--include-closed --no-push-kpi --level medium`.
- Phạm vi chấm: **284 sub-issue** (việc lá) được chấm trực tiếp; 117 issue và 13 epic là "container", SWP = tổng con, không chấm riêng.
- Kết quả: **967 SWP** (AI chấm) → **1.078 SWP** sau khi người kiểm hiệu chỉnh; trạng thái hiệu chỉnh `calibrated`. 20 việc độ tin cậy thấp, 0 việc đề nghị tách.
- Skill chỉ đọc, không ghi lên Project. SWP và giờ trên Project là do tôi dùng AI Agent ghi sau khi đã kiểm (script ghi field `SWP`, `Giờ ước lượng` từ dữ liệu nhúng trong báo cáo HTML).

## 2. Issue mẫu đã kiểm

Skill tự chọn 5 issue mẫu (nhỏ nhất, trung vị, lớn nhất, xa trung vị nhất, độ tin cậy thấp nhất). Tôi kiểm từng issue theo hai bước: chọn mức khó (rất dễ / dễ / vừa / khó / rất khó) rồi chọn điểm cụ thể; skill quy điểm đó về Scope × Complexity gần với bản AI nhất.

| Issue | Skill chấm (Scope × Complexity → SWP) | Tôi chỉnh | Lý do |
|---|---|---|---|
| [F000-01-c #5](https://github.com/thangtn-0016/lab5-product-a/issues/5) README chạy local + `.env.local.example` | 1 × 0 → **1** | → 5 × 1 = **6**, chỉ issue này | README chạy local khó hơn AI nghĩ: phải thử lại Supabase local, seed, biến môi trường. Không áp cho cả nhóm — 8 việc cùng nhóm là soạn câu hỏi / cập nhật tài liệu, giữ 1 SWP |
| [F008-13-a #285](https://github.com/thangtn-0016/lab5-product-a/issues/285) Seed 料率 gốc + tài khoản demo maker/checker | 2 × 1 → **3** | → 5 × 1 = **6**, **áp cho cả nhóm** (DB · feature · nhiều thành phần · có trạng thái) | AI chấm thấp các việc DB có nhiều thành phần và trạng thái: seed phải phối hợp nhiều tài khoản maker/checker, role và cờ `can_approve` (PREMISE L5-G07); cùng nhóm là migration kèm RLS kèm view. Quy tắc lan ra **20 việc cùng nhóm** (+2 nấc Scope mỗi việc) |
| [F012-02-b #387](https://github.com/thangtn-0016/lab5-product-a/issues/387) Migration đổi PK `business_days` + cập nhật FK, RPC, view | 5 × 3 → **10** | Giữ **10** | Đồng ý với mức AI chấm (vừa, 10): chạm mọi reader / writer của ngày nghiệp vụ và khó hoàn tác |
| [F000-02-b #8](https://github.com/thangtn-0016/lab5-product-a/issues/8) RPC nghiệp vụ lõi + trigger chặn ngày đã chốt | 5 × 3 → **10** | Giữ **10** | Đồng ý với mức AI chấm (vừa, 10). Việc này đã làm thật ở Sprint 1 |
| [F006-03-b #206](https://github.com/thangtn-0016/lab5-product-a/issues/206) Cập nhật điều kiện cảnh báo chốt sổ theo hoàn tất cấp giao dịch | 2 × 2 → **3** | → 2 × 5 = **6**, chỉ issue này | Khó vì định nghĩa "chưa giao xong" còn phụ thuộc bản sửa B-01 / B-02 và chỗ hở spec (PREMISE S-05), không phải vì mọi RPC đều khó — không lan ra 22 việc API cùng nhóm |

Tổng thay đổi: 2 việc chỉnh riêng + 20 việc theo quy tắc nhóm + 1 việc mẫu của chính nhóm đó = **23 việc đổi điểm**, tổng **+111 SWP**.

## 3. Cách quy ra giờ

**Cách chọn: quy đổi SWP → giờ bằng một hệ số chung**, không ước giờ trực tiếp từng issue.

- **Vì sao:** SWP đo khối lượng, không phụ thuộc người ước. Ước giờ thẳng từng issue (284 lần đoán) sẽ trộn năng lực người làm vào từng con số và không kiểm lại được. Một hệ số chung thì sai ở đâu cũng sai đều, đo lại velocity là sửa được toàn bộ.
- **Quy tắc:** `Giờ ước lượng = SWP × 0,19`, làm tròn 0,1 giờ.
- **Hệ số 0,19 lấy từ đâu:** Sprint 1 xong **183 SWP** (45 sub-issue có commit) trong **~35 giờ thật** (quick-report LAB-3 ghi 30–40 giờ, lấy giữa) → 35 / 183 = 0,19 giờ/SWP. Đây là giờ **một người làm cùng AI**, chất lượng prototype, **chưa viết test tự động**.
- **Áp dụng nhất quán:** cùng một hệ số cho cả 414 item (sub-issue lấy SWP của nó; issue / epic lấy tổng SWP con), ghi vào field `Giờ ước lượng` trên Project bằng script, không chỉnh tay dòng nào.
- **Giới hạn đã biết:** hệ số lạc quan vì Sprint 1 không viết test. Sẽ đo lại sau Sprint 2 (sprint có test) và ghi đè cả field — xem rủi ro ở [`progress-report.md`](progress-report.md).

## 4. Tổng theo epic

| Epic | Số issue | Số sub-issue | Tổng SWP | Tổng giờ | Ghi chú |
|---|---:|---:|---:|---:|---|
| [F000 #1](https://github.com/thangtn-0016/lab5-product-a/issues/1) Nền tảng kỹ thuật | 7 | 19 | 76 | 14,4 | 61 SWP đã Done ở Sprint 1; còn CI và khung test (PREMISE S-01) |
| [F001 #28](https://github.com/thangtn-0016/lab5-product-a/issues/28) Quản trị tài khoản & phân quyền | 10 | 25 | 84 | 16,0 | Có view tên người lập dùng chung cho 5 màn duyệt (S-02) |
| [F002 #64](https://github.com/thangtn-0016/lab5-product-a/issues/64) Người tham gia & tư cách | 10 | 28 | 113 | 21,5 | Hai máy trạng thái 許可 / 承認 tách riêng (ràng buộc đặc thù) |
| [F003 #103](https://github.com/thangtn-0016/lab5-product-a/issues/103) Lô hàng & đánh giá | 10 | 24 | 78 | 14,8 | 2 lỗi mới (S-08) |
| [F004 #138](https://github.com/thangtn-0016/lab5-product-a/issues/138) Ghi nhận giao dịch | 6 | 15 | 60 | 11,4 | 相対取引 luồng chính; 26 SWP đã Done |
| [F005 #160](https://github.com/thangtn-0016/lab5-product-a/issues/160) Theo dõi giao hàng | 11 | 26 | 73 | 13,9 | Chứa B-01…B-04 |
| [F006 #198](https://github.com/thangtn-0016/lab5-product-a/issues/198) Đối chiếu & chốt sổ | 8 | 17 | 61 | 11,6 | Chứa B-05 (bản vá 0042) |
| [F007 #224](https://github.com/thangtn-0016/lab5-product-a/issues/224) Điều chỉnh sau chốt | 7 | 13 | 40 | 7,6 | Chưa làm gì |
| [F008 #245](https://github.com/thangtn-0016/lab5-product-a/issues/245) Tiền khuyến khích & 料率 | 13 | 27 | 94 | 17,9 | **完納奨励金**; 3 issue làm rõ chặn nghiệm thu |
| [F009 #286](https://github.com/thangtn-0016/lab5-product-a/issues/286) Báo cáo & xuất dữ liệu | 10 | 27 | 101 | 19,2 | 12 báo cáo + lô kế toán CSV |
| [F010 #324](https://github.com/thangtn-0016/lab5-product-a/issues/324) Thông báo | 8 | 20 | 93 | 17,7 | Có hàng đợi offline dùng chung |
| [F011 #353](https://github.com/thangtn-0016/lab5-product-a/issues/353) Nhật ký kiểm toán | 8 | 19 | 83 | 15,8 | 1 lỗi mới (S-08) |
| [F012 #381](https://github.com/thangtn-0016/lab5-product-a/issues/381) Quản lý chợ & quy tắc | 9 | 24 | 122 | 23,2 | Epic lớn nhất; đổi PK `business_days` |
| **Tổng** | **117** | **284** | **1.078** | **204,8** | Đã Done 183 SWP · còn lại 895 SWP ≈ 170 giờ |


## 5. So với estimate LAB-1

LAB-1 ước bottom-up **1.210,5 MD** cho một đội nhiều vai trò, không dùng AI. LAB-5 có **1.078 SWP ≈ 205 giờ ≈ 26 MD** (8 giờ/ngày) theo velocity một người làm cùng AI. LAB-1 không ghi tầng từng màn, nên so theo khối thay vì ép về từng epic.

| Phần | LAB-1 | LAB-5 | Chênh | Vì sao |
|---|---:|---:|---:|---|
| Chức năng: 35 màn + 15 engine / quy tắc → epic F001–F012 | 658,5 MD | 1.002 SWP ≈ 190 giờ ≈ 24 MD | −96% | (1) Giờ LAB-5 là giờ AIDD đo thật ở Sprint 1, LAB-1 là giờ đội làm thủ công; (2) LAB-1 tính QA, UX, Arch ngay trong đơn giá mỗi màn, LAB-5 chỉ tính phần có issue; (3) hệ số đo trên sprint không viết test → lạc quan (xem mục 3) |
| Phạm vi LAB-1 không có: F012 quản lý chợ + SCR036 xét duyệt tư cách | 0 | F012 122 SWP + phần SCR036 trong F002 | thêm | LAB-4 thêm (PREMISE LAB-4 P-02, P-03) để vá lỗ hổng đề nêu; LAB-5 giữ |
| Hạ tầng, CI/CD, kiến trúc → F000 | 116 MD (43 + 73) | 76 SWP ≈ 14 giờ | −98% | F000 chỉ gồm việc dựng code và CI; thiết kế kiến trúc đã làm ở LAB-4 dưới dạng tài liệu, không thành issue (PREMISE S-01) |
| Việc xuyên suốt khác: PM & giao tiếp JP, kiểm thử hệ thống / UAT, đào tạo, chuyển dữ liệu, chốt requirement, BA | 436 MD | 0 (11 issue làm rõ = 25 SWP là phần duy nhất chạm tới) | không so được | Backlog LAB-5 là backlog phát triển; các việc này không break thành issue code. Muốn báo giá phải cộng lại riêng |
| **Tổng** | **1.210,5 MD** | **≈ 26 MD** | — | **Không dùng con số LAB-5 để báo giá.** Nó cho biết khối lượng phát triển còn lại ở tốc độ AIDD một người; LAB-1 vẫn là cơ sở báo giá cho đội thật |

Điều so sánh cho thấy: phần **chức năng** co mạnh khi có AI, còn phần **xuyên suốt** (46% của LAB-1) không co theo — đó là chỗ báo giá AIDD dễ sai nhất nếu chỉ nhân giờ code.
