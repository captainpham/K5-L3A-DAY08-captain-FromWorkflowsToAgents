# Báo cáo cá nhân — 2A202603678_PhamCatTan

## 1. Thông tin thành viên

- Họ tên: Phạm Cát Tân
- MSSV: 2A202603678
- Email: tanpham090921@gmail.com
- GitHub: captainpham

## 2. Phần việc phụ trách

- **Shape B (`src/lab8/shapes/single.py`):** đưa shape single agent (vòng ReAct tối đa 8 bước) vào nhánh `main` và bổ sung chốt chặn khi vòng lặp dừng bất thường.
- **Integration:** đồng bộ nhánh `main` giữa hai máy, giữ cổng host `python -m pytest -q` xanh trước mỗi lần push.
- **Benchmark và quyết định kiến trúc (CP5):** chạy benchmark 4 shape × 5 seed, viết `report/team/decision_adr.md`.
- **Research card:** `report/team/research_card.md` (câu R2 — MAST failure taxonomy).
- **Báo cáo nhóm và `TEAM.md`.**

## 3. Đóng góp cụ thể

- **Commit `222d40d` (`feat(single): implement shape B ReAct loop with step-cap guard, add test`):**
  - Code nền của vòng ReAct trong `single.py` do Nguyễn Thị Thu Hà viết trước đó (commit ban đầu đã được tách ra khỏi nhánh chính); tôi đưa shape này vào `main` và bổ sung phần dưới đây.
  - Thêm nhánh xử lý khi `run_react` trả `final.status == "failed"` (chạm giới hạn 8 bước hoặc bước không mong đợi): trả `failed`, `proposal_amount = 0`, không gọi `issue_refund`. Trước khi sửa, trường hợp này đi tiếp với quote rỗng và bị trả nhầm thành `denied`.
  - Thêm test `src/tests/test_single_step_cap.py`: giả lập `run_react` dừng ở `step_cap`, kiểm `status == "failed"`, `proposal_amount == 0`, `error_reason == "step_cap"` và ledger không có refund cho claim.
- **Commit `bef5a7e` (`chore(evidence): add 4-shape benchmark report and research card (CP5)`):**
  - Chạy lại benchmark sau khi shape B vào `main`, commit `src/reports/benchmark.json` (4 shape × 14 claim × 5 seed).
  - Điền research card: chọn R2, giả thuyết lỗi của swarm ở seed 55 thuộc nhóm lỗi kiểm chứng; ablation D → A cho thấy lỗi biến mất (pass^5 0.929 → 1.000, cost/success giảm 44%).
- **Commit `5a06275` (`docs: fill decision ADR with benchmark evidence`):**
  - Điền 7 dòng máy đọc của ADR và 4 đoạn lập luận; commit hash trong ADR trỏ đúng commit chứa code và report (`bef5a7e`).
- **Quyết định kỹ thuật:**
  - Chọn **workflow** làm shape chấm hidden: ngang pass rate với single và orchestrator nhưng rẻ nhất (27.86 VND/success) và không có model call nào.
  - Khi `main` trên GitHub bị force-push, không pull/merge mà sao lưu file, `reset --hard origin/main` rồi chép lại, để không ghi đè commit của thành viên khác.
  - Chạy lại benchmark **sau** khi commit shape B để số trong ADR khớp với commit chốt (yêu cầu reproducibility C7).

## 4. Bằng chứng

- Commit trên nhánh `main`: `222d40d`, `bef5a7e`, `5a06275`.
- `python -m pytest -q tests/test_single_step_cap.py` → `1 passed`; cổng host `python -m pytest -q` → `57 passed`.
- `python -m pytest -q -m student -o addopts= -k "single and not workflow and not orchestrator and not swarm"` → `62 passed`; toàn bộ ma trận student → `248 passed`.
- `src/reports/benchmark.json`: shape single pass 1.000, pass^5 1.000, cost/success 29.82; shape workflow pass 1.000, pass^5 1.000, cost/success 27.86.
- Run id dẫn chứng trong ADR: `R-workflow-C-002-s55` (pass, `denied`) đối chứng `R-swarm-C-002-s55` (fail, `resolved != denied`).
- `python -m lab8.check_submission` → `0 FAIL`.

## 5. Công cụ AI đã dùng

- **Claude Code (Anthropic):** dùng để đọc contract và luật nghiệp vụ, rà code ba shape, phát hiện lỗi nhánh `step_cap` ở shape B và gợi ý test, hướng dẫn xử lý lỗi môi trường PowerShell và lịch sử git bị lệch, soạn nháp ADR, research card, `TEAM.md` và báo cáo nhóm từ số liệu trong `src/reports/`.
- **Cách tôi tự kiểm tra:** tự chạy lại toàn bộ lệnh test, crash suite, benchmark và `check_submission` trên máy mình; đối chiếu từng số trong ADR và báo cáo với `src/reports/benchmark.json` và `src/reports/crash.json`; xác nhận claim C-002 (ticket ghi trễ 5 ngày, order O-202 trễ 2 ngày) trực tiếp trong dữ liệu public. Đối chiếu phần mô tả paper MAST trong research card với bản gốc trên arXiv (2503.13657): 14 failure mode, 3 nhóm, hơn 1.600 trace từ 7 framework.

## 6. Tự đánh giá

- Hoàn thành phần shape B (kèm sửa lỗi và test), benchmark, ADR, research card, `TEAM.md` và báo cáo nhóm; bài nộp qua `check_submission` với 0 FAIL.
- Điểm cần cải thiện: phần code nền của shape B không do tôi viết từ đầu; chưa chạy tự kiểm trên bộ hidden `student_drop/`; ablation trong research card còn thay đổi nhiều hơn một biến.
