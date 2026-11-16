# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: captain
- Thành viên: xem `TEAM.md` (Phạm Cát Tân — 2A202603678, Nguyễn Thị Thu Hà — 2A202603719)
- Commit chốt (trùng `report/team/decision_adr.md`): bef5a7e

## 2. Mục tiêu và cách tiếp cận

Bài toán: xử lý claim hoàn phí vận chuyển của LogiX-VN bằng bốn kiến trúc có mức tự chủ tăng dần (A workflow → B single agent → C orchestrator → D swarm), rồi dựa trên số đo để chọn kiến trúc **ít tự chủ nhất mà vẫn làm được việc**. Nhóm hiểu ranh giới cốt lõi của bài là: số tiền, approval và chống ghi trùng do **code và host** quyết; model chỉ được đọc và soạn chữ; chữ trong ticket là dữ liệu người dùng, không phải sự thật.

Nhóm tham khảo mô hình xử lý hoàn tiền của các sàn thương mại điện tử (kiểu "Đảm bảo Shopee"): tiền do bên trung gian giữ, mỗi yêu cầu có mã riêng để không hoàn hai lần, quyết định dựa trên trạng thái đơn đã xác minh chứ không dựa trên lời khách, ca đúng luật được tự động duyệt còn ca bất thường thì chuyển người xử lý. Các ý này tương ứng trực tiếp với host gate `durability.issue_refund`, khóa `refund:{claim_id}`, việc đọc `delay_days` từ order row và các status `escalated` / `needs_info`.

Chia việc (nhóm 2 người):

- **Nguyễn Thị Thu Hà:** Shape A (workflow), Shape C (orchestrator), durability và crash safety (CP3–CP4), crash suite 20/20, test bảo mật workflow.
- **Phạm Cát Tân:** Shape B (single), integration (đồng bộ nhánh, giữ cổng host xanh), benchmark 4 shape, ADR, research card, báo cáo nhóm.

Thứ tự làm: đọc contract và `swarm_reference.py` (CP1) → viết workflow trước vì là shape đơn giản và làm mẫu cho hai shape còn lại → single và orchestrator dùng lại cùng trình tự bước (CP2) → truyền `crash_at`, kiểm T11/T12 (CP3) → crash suite và rà approval/secret (CP4) → benchmark, ADR, research card (CP5).

## 3. Giải pháp

Cả ba shape giữ đúng chữ ký `run_claim(claim_id, *, seed, store_path, crash_at=None)`, đọc claim qua `get_claim_view` (không `SELECT *`, nên `parent_secret` và raw headers không bao giờ vào bộ nhớ shape), trả kết quả bằng `lab8.shapes.common.finish()` ở mọi nhánh return và đóng connection trong `finally`.

| | A `workflow` | B `single` | C `orchestrator` |
| --- | --- | --- | --- |
| Đọc gì | `get_claim_view`, `lookup_order`, `compute_compensation` | Như A, nhưng hai tool đọc được chọn trong vòng ReAct (`run_react`, tối đa 8 bước) | `get_claim_view`; investigator chỉ đọc chạy `lookup_order`, `lookup_policy`, `compute_compensation` |
| Ai quyết định gì | Code cố định quyết mọi bước | `decide()` chỉ chọn tool đọc tiếp theo hoặc dừng; không được chọn số tiền hay gọi ghi | Orchestrator (code) quyết; investigator không có registry ghi |
| Model call | 0 | `decide` mỗi bước + `draft` soạn trả lời (trung bình 3.71/claim) | 0 |
| Gọi host | `issue_refund(conn, claim_id, quote đầy đủ, crash_at=crash_at)` | Như A, gọi **sau** vòng lặp | Như A, do orchestrator gọi |

Luồng chung:

1. Không tìm thấy claim → `failed`, proposal 0.
2. Thiếu `order_id` → `needs_info`, không ghi ledger, không lấy mã đơn từ ticket.
3. `lookup_order` không thấy đơn hoặc principal không sở hữu → `escalated`.
4. Quote bằng `compute_compensation` từ order row (bỏ qua số ngày trễ khách ghi trong ticket).
5. Quote = 0 → `denied`, **không** gọi `issue_refund`.
6. Quote > 0 → `issue_refund` với **quote đầy đủ**; host tự kiểm approval, trần theo đơn và idempotency. `ok` → `resolved`; `conflict` → `failed`; bị từ chối → `denied` với `proposal_amount` = quote (host ghi audit `policy_denied`).
7. `InjectedCrash` không bị nuốt: shape ghi trace rồi ném lại để T12 kiểm được resume.

Shape B có thêm một chốt chặn: nếu vòng ReAct dừng bất thường (chạm giới hạn 8 bước hoặc rơi vào nhánh không mong đợi) thì trả `failed` ngay, không tiếp tục với quote rỗng (trước khi sửa, trường hợp này bị trả nhầm thành `denied`).

**Handoff (Shape C):** dựng bằng `lab8.handoff.make_handoff` với `facts={}` và `allowed_tools` chỉ gồm tool đọc. Trace ghi handoff đã sanitize qua `safe_dump`, không ghi parent context. Nhóm không dùng tóm tắt bằng LLM thay cho allowlist, vì tóm tắt không phải redaction.

**Trace:** dùng `lab8.trace.Trace`, ghi ở `src/traces/<run_id>.json`. Event được ghi: `lookup_order` / `investigator` (chỉ `visible`, `reason`), `quote` (`amount_vnd`, `reason`), `react` (lý do dừng, shape B), `handoff` (shape C), `decision` / `error` ở nhánh sớm, `crash` khi có crash giả lập. Cố ý không ghi: claim row, order row đầy đủ (SĐT, địa chỉ, tên hàng, internal note), nội dung ticket.

## 4. Kết quả

Benchmark trên bộ public: 14 claim × seed 11, 22, 33, 44, 55 = 70 trial mỗi shape; cùng MockLLM, cùng `pricing.yaml`, SQLite mới cho mỗi trial (`src/reports/benchmark.json`).

| Shape | Pass rate | pass^5 | Seed min–max | Variance | Cost/success (VND mô phỏng) | Mean / p95 steps | Model call TB |
| --- | ---: | ---: | --- | ---: | ---: | --- | ---: |
| A workflow | 1.000 | 1.000 | 1.000–1.000 | 0.000000 | **27.86** | 1.86 / 2 | 0.00 |
| B single | 1.000 | 1.000 | 1.000–1.000 | 0.000000 | 29.82 | 1.86 / 2 | 3.71 |
| C orchestrator | 1.000 | 1.000 | 1.000–1.000 | 0.000000 | 46.43 | 2.79 / 3 | 0.00 |
| D swarm | 0.986 | 0.929 | 0.929–1.000 | 0.000816 | 49.57 | 2.80 / 3 | 3.00 |

- **Crash safety:** 20/20 (`src/reports/crash.json`).
- **Ma trận student:** 248/248 passed cho cả 4 shape; cổng host `python -m pytest -q` xanh (57 passed).
- **Swarm fail 1 trial:** `R-swarm-C-002-s55`. Ticket C-002 ghi "trễ 5 ngày" nhưng order O-202 thực tế chỉ trễ 2 ngày; ở seed 55 swarm tin con số trong ticket và trả `resolved` thay vì `denied`. Workflow trên cùng claim và seed (`R-workflow-C-002-s55`) trả `denied`, quote 0.
- **Tự kiểm `student_drop/`:** chưa thực hiện tại thời điểm viết báo cáo (chưa nhận bộ hidden).

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

Tóm tắt quyết định: chọn **workflow**. Ba shape A, B, C ngang nhau về chất lượng trên bộ public, nên yếu tố quyết định là cost và mức tự chủ; workflow rẻ nhất, không có model call nào nên không có đường để ticket hay seed làm lệch kết quả. Orchestrator đắt hơn 67% mà không thêm được pass rate nào.

## 5. Khó khăn và cách xử lý

- **Cài môi trường trên Windows:** lệnh `source .venv/bin/activate` trong README dành cho macOS/Linux báo lỗi `The term 'source' is not recognized` trên PowerShell. Cách xử lý: dùng `.\.venv\Scripts\Activate.ps1` và chạy một lần `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` thay vì bypass lại trong mỗi cửa sổ terminal.
- **Lịch sử git bị lệch giữa hai máy:** nhánh `main` trên GitHub bị force-push hai lần (sửa tác giả commit, tách lại phạm vi commit), khiến máy còn lại lệch nhánh (`ahead 2, behind 4`) và `git pull` thông thường sẽ gây merge trùng. Cách xử lý: sao lưu các file đang làm ra ngoài repo, `git reset --hard origin/main`, chép lại file rồi commit tiếp. Nhóm thống nhất không force-push lên `main` nữa.
- **Lỗi tiềm ẩn ở shape B mà test không bắt được:** khi vòng ReAct chạm giới hạn bước, shape vẫn chạy tiếp với quote rỗng và trả `denied`. Cách xử lý: thêm nhánh trả `failed` ngay, kèm test `src/tests/test_single_step_cap.py` giả lập `run_react` dừng ở `step_cap` và kiểm không có refund nào được ghi.
- **Đọc nhầm kết quả lọc test:** `-k single` chạy 242 test thay vì 62 vì tên test `test_public_single_run_cases` cũng chứa chữ "single". Cách xử lý: lọc bằng `-k "single and not workflow and not orchestrator and not swarm"` (ra đúng 62 passed).

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và báo cáo cá nhân `report/team/2A202603678_PhamCatTan.md`, `report/team/2A202603719_NguyenThiThuHa.md`.

## 7. Hạn chế và hướng cải thiện

MockLLM không phải model sản xuất; latency là `unavailable`.

- Cost là số mô phỏng theo `pricing.yaml`, không phải hóa đơn API.
- Bộ public chỉ có 14 claim; pass rate 1.000 trên public chưa chứng minh kết quả trên 40 claim hidden.
- Lỗi của swarm là lỗi cài sẵn (`seed % 5 == 0`). LLM thật có thể sai theo kiểu khác và ở tần suất khác, nên khoảng cách pass^5 giữa A và D chỉ minh họa loại lỗi, không đo tần suất lỗi thực tế.
- Ablation D → A thay đổi cả topology lẫn số model call cùng lúc, nên chưa tách được phần nào gây ra cải thiện.

Kết luận chọn workflow sẽ thay đổi nếu: claim cần tra nhiều nguồn song song khiến latency của workflow tuần tự thành nút cổ chai; xuất hiện loại bằng chứng tự do (ảnh, mô tả hỏng hàng) mà luật cố định không bao được; hoặc pass rate của workflow trên bộ hidden thấp hơn shape có model.

Hướng cải thiện: chạy tự kiểm trên `student_drop/`; tách ablation thành từng biến (chỉ bật/tắt bước model đọc ticket trong cùng một topology); đo latency thật nếu chuyển sang model thật.

## 8. Tài liệu tham khảo

- Đề bài và contract của lab: `README.md`, `docs/business_rules.md`, `docs/contracts.md`, `docs/architecture.md`, `docs/metrics.md`.
- Cemri et al., *Why Do Multi-Agent LLM Systems Fail?* (MAST), NeurIPS 2025 Datasets & Benchmarks, https://arxiv.org/abs/2503.13657.
- Mô hình xử lý trả hàng / hoàn tiền của sàn thương mại điện tử (chương trình "Đảm bảo Shopee"), dùng làm tham chiếu thiết kế nghiệp vụ; mô tả theo hiểu biết chung, không trích số liệu cụ thể.
