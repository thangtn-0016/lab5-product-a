# LAB-4 — Bộ specs Product A

Product A là hệ thống hỗ trợ nghiệp vụ chợ đầu mối thủy sản (水産物卸売市場 業務支援システム). Bộ specs này đặc tả đủ 35 màn của LAB-1 và thêm 2 màn mới. Trong 37 màn, 11 màn đã chạy thật ở prototype LAB-3: https://lab3-product-a.vercel.app. Bộ này là đầu vào để chia task ở LAB-5.

File này mô tả cấu trúc thư mục `specs/`: file nào kế thừa từ bộ spec-standard, file nào mới tạo theo đề bài, và quy ước đặt mã dùng chung.

## Về bộ spec-standard đã dùng

Mình không tìm được tài liệu nào tên "spec-standard" trong phần đề bài và tài liệu đang có. Vì vậy mình dùng bộ template spec chuẩn của Takumi, bộ agent kit mình dùng suốt từ LAB-1 (skill `rebuild-spec`). Bộ đó quy định layout `docs/system · generated · flows · features · screens · decisions` và 41 file template. Đây là một giả định, đã ghi ở [PREMISE.md](./PREMISE.md) mục G1. Nếu công ty có bộ chuẩn khác thì phần nội dung vẫn dùng lại được, chỉ phải đổi khung.

Cách áp template:
- Heading giữ **nguyên tiếng Anh** như template, để nhìn là biết mục nào của chuẩn. Nội dung viết tiếng Việt, thuật ngữ nghiệp vụ tiếng Nhật giữ nguyên (`相対取引`, `確定`, `許可`…).
- Mục nào không áp dụng thì vẫn giữ heading và ghi "Không áp dụng — lý do", không xoá đi.
- Chỗ cần thêm ngoài template thì đặt thành mục con, có ghi "(mục bổ sung)" ở heading.

## Cây thư mục

```
specs/
├── README.md                 ← file này                                  [mới theo đề]
├── PREMISE.md                ← quyết định bổ sung sau LAB-1               [mới theo đề]
├── AI-report.md              ← cách dùng AI ở LAB-4                       [mới theo đề]
└── docs/                     ← toàn bộ specs chi tiết, layout spec-standard
    ├── system/               ← tầng hệ thống (5 file)
    │   ├── overview.md
    │   ├── architecture.md
    │   ├── business-rules.md
    │   ├── permissions.md
    │   └── glossary.md
    ├── decisions/            ← ADR-001 … ADR-007, mỗi quyết định một file
    ├── flows/                ← FL-01 … FL-03, luồng nghiệp vụ xuyên màn
    ├── generated/            ← danh mục tổng hợp
    │   ├── feature-list.md
    │   ├── screen-list.md
    │   ├── screen-flow.md
    │   ├── entities.md       ← database diagram + đối chiếu bảng thật
    │   ├── erd.mmd           ← ERD đầy đủ cột, sinh từ schema sống
    │   └── schema-snapshot.json
    ├── features/             ← 12 feature × 4 file
    │   └── F001_AdminAndPermissions/ … F012_MarketManagement/
    │       ├── business-context.md
    │       ├── technical-spec.md
    │       ├── screens.md
    │       └── edge-cases.md
    ├── screens/              ← 37 screen spec chi tiết
    │   └── SCR001_RoleHome/spec.md … SCR037_MarketManagement/spec.md
    └── design/               ← wireframe + ảnh chụp bản deploy
        ├── wireframes/       ← 4 wireframe HTML + README.md giải thích
        └── screenshots/      ← 23 ảnh từ bản chạy thật LAB-3 + README.md giải thích
```

## Kế thừa hay mới — từng phần

| Phần | Nguồn khuôn | Template spec-standard dùng |
|---|---|---|
| `README.md`, `PREMISE.md`, `AI-report.md` | Mới, theo yêu cầu đề bài LAB-4 | — |
| `docs/system/overview.md` | Kế thừa | `system-overview-template.md` |
| `docs/system/architecture.md` | Kế thừa, thêm mục con chi tiết | `architecture-template.md` |
| `docs/system/business-rules.md` | Kế thừa | `business-rules-template.md` |
| `docs/system/permissions.md` | Kế thừa, thêm phụ lục ma trận màn × chức danh | `permissions-template.md` |
| `docs/system/glossary.md` | Kế thừa, thêm cột tiếng Việt, xếp theo nhóm nghiệp vụ | `glossary-template.md` |
| `docs/decisions/ADR-*.md` | Kế thừa vị trí `docs/decisions/ADR-*`; 4 phần Bối cảnh · Lựa chọn · Phương án đã bỏ và vì sao · Hệ quả (4 ADR có thêm Câu hỏi còn mở), giữ từ LAB-4 | — (chuẩn chỉ quy định vị trí) |
| `docs/flows/FL-*.md` | Kế thừa | `process-flow-template.md` |
| `docs/generated/feature-list.md` | Kế thừa | `feature-list-template.md` |
| `docs/generated/screen-list.md` | Kế thừa | `screen-list-template.md` |
| `docs/generated/screen-flow.md` | Kế thừa | `screen-flow-template.md` |
| `docs/generated/entities.md` | Kế thừa, thêm mục đối chiếu thiết kế ↔ bảng thật | `data-model-template.md` |
| `docs/features/F###/` (4 file) | Kế thừa bộ 4 file bắt buộc mỗi feature | `business-context`, `technical-spec`, `screens`, `edge-cases` |
| `docs/screens/SCR###/spec.md` | Kế thừa, thêm 3 mục con (xem dưới) | `screen-spec-template.md` |
| `docs/design/` | **Mới theo đề** — chuẩn không có chỗ cho wireframe | — |
| `generated/erd.mmd`, `schema-snapshot.json` | Mới — dữ liệu thô để tái lập ERD từ database thật | — |

Một số mục trong screen spec không có trong template. Các mục này đều có heading kết thúc bằng *(mục bổ sung)*:
- **Dependencies & Risks** (cả 37 spec): màn này phụ thuộc gì, rủi ro gì, kể cả lỗi thật đã tìm thấy trong code.
- **Unresolved Questions** (cả 37 spec): những điểm phải hỏi khách hoặc phải quyết trước LAB-5.
- **Error Catalogue** (13 spec): bảng mã lỗi đầy đủ của màn, nằm trong mục `Validation & Error Feedback`. Các spec còn lại để bảng mã lỗi thẳng trong mục đó.
- **Read Calls** (24 spec): các lần đọc dữ liệu không gắn với nút submit nào.
- **Field Labels & Defaults** (11 spec): nhãn VI/JA và giá trị mặc định của từng ô.
- **State Machine** (1 spec): máy trạng thái riêng của màn.

Các mục này giữ lại nội dung các phần "Validation", "Gọi dữ liệu", "Phụ thuộc và rủi ro" và "Chưa chốt" của khuôn 10 mục mình tự đặt ở lần nộp đầu. Template chuẩn không có chỗ tương đương nên mình không bỏ được.

Có một số template của chuẩn mình **không dùng**, kèm lý do:
- `route-list`, `api-map`, `api-contracts`, `behavior-logic`, `job-list`: các template này sinh ra để đảo ngược từ code. Ở đây 26/37 màn chưa có code, nên phần API và batch đã nằm trong technical-spec của từng feature và mục Validation của từng màn.
- `user-stories`: user story giữ mã LAB-1 (`US-01` … `US-20`) và viết thẳng trong technical-spec của từng feature, như template đó cho phép.
- `test-cases`: đây là việc của LAB-5. Mỗi technical-spec đã có mục Verification ghi cách kiểm.

## Bộ tài liệu LAB-4 cũ chuyển sang đâu

Lần nộp đầu có 5 sản phẩm. Lần này mình chuyển toàn bộ sang layout chuẩn, không bỏ nội dung nào:

| Sản phẩm cũ | Nằm ở |
|---|---|
| Wireframe 4 màn + 23 ảnh chụp | `docs/design/` |
| Architecture design | `docs/system/architecture.md` + 3 luồng ở `docs/flows/` |
| 37 screen spec (khuôn 10 mục) | `docs/screens/SCR###/spec.md` (template chuẩn) |
| ADR 7 quyết định | `docs/decisions/ADR-001` … `ADR-007` |
| Database diagram + đối chiếu bảng thật | `docs/generated/entities.md` + `erd.mmd` |

## Quy ước mã dùng chung

| Mã | Ý nghĩa |
|---|---|
| `SCR001` … `SCR037` | Mã màn theo chuẩn. Mỗi spec ghi kèm tên LAB-1 ngay dưới tiêu đề, ví dụ `SCR010` = **SC-10**. `SCR036` = SC-N1, `SCR037` = SC-N2 (2 màn thiết kế mới) |
| `F001` … `F012` | Feature theo chuẩn, ứng với nhóm `F01` … `F11` của LAB-1. `F012` là feature mới ở LAB-4 (quản lý chợ). Tính năng con giữ mã LAB-1 (`F04-02`…) |
| `US-xx`, `FR-…`, `NFR-…`, `BR-…` | Giữ nguyên mã LAB-1, không đánh lại |
| `ADR-001` … `ADR-007` | Quyết định kiến trúc |
| `FL-01` … `FL-03` | Luồng nghiệp vụ |
| `P0001` … | Mã lỗi nghiệp vụ trả từ database. `P0001`–`P0009` đã chạy thật trong prototype, các mã còn lại là đề xuất. Mỗi mã chỉ một nghĩa |
| `B-01` … `B-05` | Lỗi thật tìm thấy trong code/dữ liệu prototype, ghi ở `system/business-rules.md` |
| Chức danh | 4 đã có: `sys_admin` `receiving` `trading` `reconcile`. 5 đề xuất: `party_admin` `rule_admin` `ops_admin` `market_manager` `internal_audit`. Quyền duyệt dùng cờ `can_approve` trên tài khoản |

Ở các spec, phần nào đã chạy trong prototype đều trích file thật, ví dụ `supabase/migrations/0007_rpc_functions.sql`. Phần chưa xây ghi rõ "Chưa có code — màn chỉ đặc tả ở LAB-4", và chỗ nào do mình đề xuất thì đánh dấu `(đề xuất)`. Code không nằm trong bài nộp; các đường dẫn đó trỏ tới repo `thangtn-0016/lab3-product-a` (private).

## Đọc theo thứ tự nào

1. [PREMISE.md](./PREMISE.md): các quyết định làm nền, đọc trước để biết bộ này đã chốt hướng nào.
2. [docs/system/overview.md](./docs/system/overview.md), rồi đến [architecture.md](./docs/system/architecture.md).
3. [docs/generated/feature-list.md](./docs/generated/feature-list.md) và [screen-list.md](./docs/generated/screen-list.md): bản đồ của 12 feature và 37 màn.
4. [docs/generated/entities.md](./docs/generated/entities.md): mô hình dữ liệu và các chỗ thiết kế lệch với bảng thật.
5. Chi tiết theo feature (`docs/features/`) hoặc theo màn (`docs/screens/`).
6. [AI-report.md](./AI-report.md).

Chỗ đáng xem nhất là [SCR010](./docs/screens/SCR010_NegotiatedTradeConfirm/spec.md), màn chốt giao dịch thỏa thuận. Cổng chốt ở màn này kiểm cùng lúc tư cách người mua và số lượng lô còn lại trong một transaction, và đó là trục của cả hệ thống.
