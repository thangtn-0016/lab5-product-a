# LAB-5 — Product A: break task, GitHub Project, SWP và báo cáo tiến độ

**Dự án:** Product A — 水産物卸売市場 業務支援システム (hệ thống hỗ trợ nghiệp vụ chợ đầu mối thủy sản)
**Học viên:** Trần Ngọc Thắng · GitHub `thangtn-0016`
**Đề:** Product A

## Tổng quan

- **Backlog break từ:** bộ detail design **LAB-4 đã nộp ngày 2026-10-12** (37 màn, 12 nhóm tính năng F001–F012, 7 ADR, 3 flow). Bản chép nguyên nằm ở [`docs/`](docs/).
- **PREMISE có thay đổi:** có — 1 epic mới (F000), 6 chỗ sửa thiết kế nhỏ, 3 lỗi mới, 12 giả định, 7 điểm chốt hướng; không thêm / bỏ màn nào. Nguồn: tự phát hiện khi break (chưa nhận góp ý review LAB-4). Xem [`PREMISE.md`](PREMISE.md).
- **Quy mô:** **13 epic · 117 issue · 284 sub-issue** (414 item trên Project), 44 issue mang ràng buộc đặc thù Product A, 11 issue làm rõ.
- **Estimate:** **1.078 SWP** (sau khi người kiểm hiệu chỉnh) ≈ **205 giờ** theo velocity thật.
- **Sprint đã chạy:** **Sprint 1** (22/09 → 05/10/2026) — chạy thật, trạng thái lấy theo commit code LAB-3: xong 183 / 227 SWP (81%). Sprint 2 (20/10 → 02/11) đã xếp 144 SWP.

## Link

| | |
|---|---|
| Repo | https://github.com/thangtn-0016/lab5-product-a |
| GitHub Project | https://github.com/users/thangtn-0016/projects/2 |
| Báo cáo HTML của skill chấm SWP | [`reports/estimate-swp-lab5-product-a-20261017T160606Z.html`](reports/estimate-swp-lab5-product-a-20261017T160606Z.html) (tải về rồi mở bằng trình duyệt) |

## Cấu trúc repo

| File / thư mục | Nội dung |
|---|---|
| `README.md` | File này: tổng quan, link, ánh xạ spec → epic |
| [`PREMISE.md`](PREMISE.md) | Khác biệt so với LAB-4: spec thêm / sửa, giả định bổ sung, hướng đã chốt |
| [`task-breakdown.md`](task-breakdown.md) | Nguyên tắc chia task, spec → issue, hạng mục ràng buộc đặc thù, chỗ hở spec phát hiện khi break |
| [`estimate.md`](estimate.md) | Issue mẫu đã kiểm, cách quy ra giờ, tổng theo epic, so với estimate LAB-1 |
| [`progress-report.md`](progress-report.md) | Báo cáo tiến độ Sprint 1: trạng thái, burndown, rủi ro trễ, kế hoạch Sprint 2 |
| [`AI-report.md`](AI-report.md) | Dùng AI ở đâu, review output AI thế nào, điểm đặc biệt |
| [`reports/`](reports/) | Báo cáo HTML của skill `tkm:estimate-swp` (một file, tự chứa) |
| [`docs/`](docs/) | Bản chép markdown của bộ detail design LAB-4 đã nộp — mốc so sánh, không sửa. Đường dẫn `docs/…` trong issue trỏ vào đây. Ảnh chụp và wireframe HTML không chép sang |
| `docs/lab4-README.md`, `docs/lab4-PREMISE.md` | README và PREMISE của bài LAB-4 |

## Spec → epic

| Spec (LAB-4, hoặc dòng PREMISE) | Epic trên Project | Ghi chú |
|---|---|---|
| PREMISE **S-01** (không có spec LAB-4) · `docs/system/architecture.md`, ADR-001/002/004 | [F000 #1](https://github.com/thangtn-0016/lab5-product-a/issues/1) — Nền tảng kỹ thuật | Epic mới ở LAB-5; 5/7 issue đã Done ở Sprint 1 |
| `docs/features/F001_AdminAndPermissions/` · SCR002–004 | [F001 #28](https://github.com/thangtn-0016/lab5-product-a/issues/28) — Quản trị tài khoản & phân quyền | Tách trách nhiệm, maker-checker (`can_approve`), MFA phân tầng |
| `docs/features/F002_ParticipantsAndQualification/` · SCR005, 006, 036 · ADR-006 · FL-03 | [F002 #64](https://github.com/thangtn-0016/lab5-product-a/issues/64) — Người tham gia & tư cách | **Ràng buộc đặc thù:** 許可 / 承認 hai luồng trạng thái riêng |
| `docs/features/F003_LotsAndInspection/` · SCR007–009 · ADR-003 | [F003 #103](https://github.com/thangtn-0016/lab5-product-a/issues/103) — Lô hàng & đánh giá | Đánh giá chất lượng là quyết định của người; offline hẹp |
| `docs/features/F004_TradeRecording/` · SCR010–013 · FL-01 | [F004 #138](https://github.com/thangtn-0016/lab5-product-a/issues/138) — Ghi nhận giao dịch | **Ràng buộc đặc thù:** 相対取引 là luồng chính; せり chỉ nhập tay |
| `docs/features/F005_DeliveryTracking/` · SCR014–017 | [F005 #160](https://github.com/thangtn-0016/lab5-product-a/issues/160) — Theo dõi giao hàng | Hoàn tất ở cấp giao dịch; chứa lỗi B-01…B-04 |
| `docs/features/F006_ReconciliationAndClosing/` · SCR018–021 · ADR-005 | [F006 #198](https://github.com/thangtn-0016/lab5-product-a/issues/198) — Đối chiếu & chốt sổ | Chốt sổ cảnh báo không chặn; chứa lỗi B-05 |
| `docs/features/F007_PostCloseCorrection/` · SCR022–024 · FL-02 | [F007 #224](https://github.com/thangtn-0016/lab5-product-a/issues/224) — Điều chỉnh sau chốt | Lịch sử không sửa đè; một cấp duyệt |
| `docs/features/F008_IncentiveAndRateVersion/` · SCR025–027 | [F008 #245](https://github.com/thangtn-0016/lab5-product-a/issues/245) — Tiền khuyến khích & phiên bản 料率 | **Ràng buộc đặc thù: 完納奨励金** (110/100, làm tròn xuống JPY, quá hạn = 0); 料率 có phiên bản, trần 4 lần / năm |
| `docs/features/F009_ReportsAndExport/` · SCR028–030 | [F009 #286](https://github.com/thangtn-0016/lab5-product-a/issues/286) — Báo cáo & xuất dữ liệu | Báo cáo ngày xong trước 11:00 JST; kế toán CSV một chiều |
| `docs/features/F010_Notifications/` · SCR001, 031–033, 035 | [F010 #324](https://github.com/thangtn-0016/lab5-product-a/issues/324) — Thông báo | Chỉ in-app + email; việc quan trọng trong 5 phút |
| `docs/features/F011_AuditLog/` · SCR034 | [F011 #353](https://github.com/thangtn-0016/lab5-product-a/issues/353) — Nhật ký kiểm toán | Append-only, lưu 7 năm |
| `docs/features/F012_MarketManagement/` · SCR037 · ADR-007 | [F012 #381](https://github.com/thangtn-0016/lab5-product-a/issues/381) — Quản lý chợ & quy tắc | **Ràng buộc đặc thù:** quy tắc vận hành riêng từng chợ là dữ liệu có phiên bản |

Chi tiết từng issue trong mỗi epic: [`task-breakdown.md`](task-breakdown.md) mục 2.
