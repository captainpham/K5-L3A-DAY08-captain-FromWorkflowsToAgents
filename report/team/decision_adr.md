# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: bef5a7e
- Run id dẫn chứng: R-workflow-C-002-s55 (đối chứng R-swarm-C-002-s55 fail) — src/reports/benchmark.json
- Pass rate / pass^5 / seed min-max / variance: 1.000 / 1.000 / 1.000–1.000 / 0.000 (14 claim public × seed 11,22,33,44,55 = 70 trial)
- Cost per success (mô phỏng, VND): 27.86 (single 29.82, orchestrator 46.43, swarm 49.57)
- Crash safety: 20/20 (src/reports/crash.json); T11, T12 của workflow xanh trong ma trận student
- Secret leak / approval bypass: 0 / 0 trên 70 trial public (quét src/traces/R-workflow* không có CANARY_SECRET_; không có refund nào ngoài kỳ vọng)

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Viết 1 đoạn. So cùng corpus và cùng seed, không so với một lần chạy lẻ.

Trên cùng 14 claim public, cùng 5 seed, cùng `pricing.yaml` và cùng evaluator (`lab8.benchmark`, SQLite mới cho mỗi trial), ba shape workflow, single và orchestrator đều đạt pass rate 1.000 và pass^5 1.000. Swarm chỉ đạt 0.986 / 0.929 vì fail claim C-002 ở seed 55 (`status resolved != denied`): swarm tin số ngày trễ viết trong ticket thay vì dòng order. Khi chất lượng ngang nhau thì yếu tố quyết định là cost và mức tự chủ. Workflow rẻ nhất (27.86 VND/success, 0 model call, p95 2 bước), không có bước nào để model quyết định, nên không có đường nào để ticket hay seed làm lệch kết quả. Đây là kiến trúc ít tự chủ nhất mà vẫn làm đủ việc.

## Có đáng đổi lấy multi-agent không

Không, trên dữ liệu hiện tại. Orchestrator cùng pass rate nhưng cost/success cao hơn 67% (46.43 so với 27.86) và nhiều bước hơn (2.79 so với 1.86); swarm vừa đắt hơn 78% vừa có variance > 0. Multi-agent không mua thêm được pass rate nào. Quyết định sẽ đảo lại nếu: (1) claim có nhiều nguồn cần tra song song (nhiều đơn, nhiều kho, nhiều đối tác vận chuyển) khiến investigator song song giảm latency đáng kể; (2) xuất hiện loại claim mà luật cố định không bao được, phải đọc hiểu bằng chứng tự do (ảnh, mô tả hỏng hàng) và pass rate của workflow trên bộ hidden rơi dưới mức của shape có model; (3) có số đo latency thật cho thấy workflow tuần tự là nút cổ chai.

## Trace và bằng chứng

Trace workflow ghi 3 loại event: `lookup_order` (chỉ `visible`, `reason`), `quote` (chỉ `amount_vnd`, `reason`), và `crash` (`at`) khi có crash giả lập; nhánh sớm ghi `error` hoặc `decision`. Nhóm cố ý **không** ghi: claim row (chứa `parent_secret`, `raw_headers_json`), order row đầy đủ (SĐT, địa chỉ, tên hàng, internal note) và nội dung ticket (dữ liệu người dùng, có thể chứa PII). Claim được đọc qua `get_claim_view` thay vì `SELECT *`, nên secret không bao giờ vào bộ nhớ của shape. Trace nằm ở `src/traces/<run_id>.json`, đúng chỗ grader quét; shape không ghi file nào khác và không in ra stdout. Nếu trace phải nằm ngoài chỗ scanner quét được, đội security vẫn chứng minh được bằng cách chỉ ra trong code rằng mọi lệnh `trace.add` chỉ nhận field từ allowlist, kèm test `test_workflow_security.py` khẳng định không có `CANARY_SECRET_` trong trace.

## Giới hạn

MockLLM không phải model sản xuất. Latency để `unavailable`. Không sửa JSON cho đủ 0.75.

- Cost là số mô phỏng theo `pricing.yaml`, không phải hóa đơn API.
- Corpus public chỉ có 14 claim; pass rate 1.000 trên public chưa chứng minh được kết quả trên 40 claim hidden.
- Lỗi của swarm là lỗi cài sẵn (`seed % 5 == 0`); LLM thật có thể sai theo kiểu khác và ở tần suất khác.
