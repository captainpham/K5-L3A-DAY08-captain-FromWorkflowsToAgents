# Báo cáo cá nhân — 2A202603719_NguyenThiThuHa

## 1. Thông tin thành viên

- Họ tên: Nguyễn Thị Thu Hà
- MSSV: 2A202603719
- Email: nha253158@gmail.com
- GitHub: nha253158-hub

## 2. Phần việc phụ trách

- **Shape A (`workflow.py`):** Xây dựng quy trình tuần tự xác định (Deterministic Workflow) xử lý yêu cầu bồi thường phí vận chuyển, không để model quyết định số tiền.
- **Shape C (`orchestrator.py`):** Xây dựng kiến trúc Orchestrator phối hợp với read-only Investigators thông qua cơ chế handoff allowlist bảo mật.
- **Durability & Crash Safety (CP3, CP4):** Đảm bảo tính Idempotency khi retry (T11) và khả năng chịu lỗi phục hồi khi crash giữa chừng (T12).
- **Thực thi và sinh bằng chứng Crash Safety Suite 20/20:** Tạo file bằng chứng `src/reports/crash.json`.
- **Viết test kiểm thử riêng:** Xây dựng file test bảo mật `src/tests/test_workflow_security.py` (đáp ứng tiêu chí Rubric C8).

## 3. Đóng góp cụ thể

- **Commit `e390e23` (`feat(shapes): implement workflow and orchestrator shapes (CP2)`):**
  - Cài đặt hoàn chỉnh `run_claim` cho Shape A (`workflow.py`): đọc claim view an toàn (`get_claim_view`), tra cứu đơn hàng (`lookup_order`), tính toán số tiền hoàn dựa trên dòng order (`compute_compensation`), gọi `durability.issue_refund` với quote đầy đủ và trả về đúng hợp đồng `RunResult`.
  - Cài đặt Shape C (`orchestrator.py`): đóng gói thông tin qua `make_handoff` an toàn, chạy các investigator chỉ đọc qua `run_readonly_investigators`, ngăn chặn hoàn toàn việc rò rỉ secret hoặc PII qua ranh giới giữa các agent.
  - Cả hai shape đều đạt kết quả tuyệt đối **62/62 passed** trong bộ test student của môn học.
- **Commit `83aaacb` (`test(workflow): add security and edge-case tests for workflow shape`):**
  - Viết file test `src/tests/test_workflow_security.py` kiểm tra xác nhận trace của workflow không chứa bất kỳ chuỗi `CANARY_SECRET_*` hoặc token bí mật nào.
  - Kiểm tra xử lý trường hợp mã claim không tồn tại (`unknown_claim`) trả về đúng status `failed` và proposal = 0 mà không văng exception unhandled.
- **Commit `073e2e9` (`chore(evidence): add 20/20 crash safety report (CP4)`):**
  - Chạy công cụ `lab8.crash_test` mô phỏng 20 điểm ngắt crash nguy hiểm (trước effect, sau effect trước checkpoint, sau checkpoint, conflict retry, retry không có approval, swarm crash), đạt kết quả an toàn tuyệt đối 20/20 và commit file bằng chứng `src/reports/crash.json`.
- **Quyết định kỹ thuật đưa ra:**
  - Luôn truyền **quote đầy đủ** vào `durability.issue_refund` để Host tự áp dụng trần bồi thường theo đơn (`per-order cap`), tránh lỗi trừ 2 lần ở các ca bồi thường bổ sung (case T14).
  - Khi gặp `InjectedCrash`, luôn để exception bung ra ngoài sau khi ghi log trace để phục vụ kiểm tra khả năng phục hồi khi crash (T12).
  - Áp dụng cơ chế Allowlist sanitization nghiêm ngặt cho handoff trong Shape C để investigator không bao giờ tiếp cận được `parent_secret` hay raw headers.

## 4. Bằng chứng

- **Lịch sử commit kỹ thuật trên nhánh `main`:**
  - Commit `e390e23`: cài đặt `workflow.py` và `orchestrator.py`.
  - Commit `83aaacb`: thêm test bảo mật `src/tests/test_workflow_security.py`.
  - Commit `073e2e9`: bổ sung báo cáo `src/reports/crash.json`.
- **Bằng chứng test:**
  - `python -m pytest -q tests/test_workflow_security.py` $\rightarrow$ `2 passed`.
  - `python -m pytest -q -m student -o addopts= -k "(test_t11 or test_t12) and (workflow or orchestrator)"` $\rightarrow$ `4 passed` (đạt cả T11 và T12).
  - `python -m lab8.crash_test --count 20 --out reports/crash.json` $\rightarrow$ `crash_safety 20/20`.

## 5. Công cụ AI đã dùng

- Sử dụng trợ lý AI Antigravity / Gemini CLI để hỗ trợ phân tích contract, kiểm tra các kịch bản test student T01–T14 và rà soát các ranh giới bảo mật trace.
- Tự kiểm tra: Toàn bộ code sinh ra được kiểm thử độc lập qua bộ test suite của môn học (`pytest -q`, `pytest -m student`), cổng Host luôn giữ xanh 54/54 passed.

## 6. Tự đánh giá

- Hoàn thành đầy đủ và vượt mức các mục tiêu kỹ thuật của CP1, CP2, CP3, CP4 cho 2 shape được phân công.
- Code đáp ứng tuyệt đối 3 tiêu chí Must-Pass (không lộ secret, không bypass approval, crash safety 20/20).
- Đánh giá mức độ đóng góp: Hoàn thành xuất sắc nhiệm vụ (10/10).
