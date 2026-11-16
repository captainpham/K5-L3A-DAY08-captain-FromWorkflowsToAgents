# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R2 · Failure taxonomy |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Cemri et al., *Why Do Multi-Agent LLM Systems Fail?* (MAST), NeurIPS 2025 Datasets & Benchmarks, https://arxiv.org/abs/2503.13657 — benchmark/phân tích trace có gán nhãn |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Lỗi của hệ multi-agent LLM gom được thành 14 failure mode trong 3 nhóm: lỗi đặc tả/thiết kế hệ thống, lệch pha giữa các agent, và lỗi kiểm chứng/kết thúc task; nhiều lỗi đến từ thiết kế hệ thống chứ không chỉ do model yếu |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Trace từ các framework MAS mã nguồn mở chạy LLM thật trên task lập trình và toán; nhãn do người và LLM-as-judge gán. Không phải task claims/hoàn tiền, không có host chặn ghi tiền như lab này |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Lỗi của swarm ở seed 55 thuộc nhóm "kiểm chứng": agent nhận số ngày trễ trong ticket mà không đối chiếu với order row. Đổi D (swarm) → A (workflow), tức bỏ bước để model đọc ticket vào quyết định, thì pass^5 tăng từ < 1.0 lên 1.0, seed variance về 0, cost/success giảm khoảng 40% vì không còn model call |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Cố định: MockLLM, tool registry manifest 1.1, seed 11/22/33/44/55, `data/public_claims.jsonl` (14 claim), `config/pricing.yaml`, evaluator `lab8.benchmark` với SQLite mới mỗi trial. Biến thay đổi: shape D → A (có/không có bước model đọc ticket để quyết định) |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Lỗi dự kiến: refund `resolved` cho claim mà ticket khai khống số ngày trễ (đáp án `denied`). Đo bằng pass^5 và seed variance theo shape, cộng số trial có reason `status resolved != denied` |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json`
- Run id / file report: `src/reports/benchmark.json`; trial fail `R-swarm-C-002-s55`, đối chứng `R-workflow-C-002-s55` (pass, `denied`, quote 0)
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: **Đúng.** Swarm: pass 0.986, pass^5 0.929, seed min–max 0.929–1.000, variance 0.000816, cost/success 49.57 VND, 1 trial fail (C-002, seed 55: ticket "trễ 5 ngày", order row không đủ 3 ngày, swarm trả `resolved`). Workflow: pass 1.000, pass^5 1.000, variance 0, cost/success 27.86 VND (giảm 44%), 0 model call. Lỗi biến mất khi bỏ bước model đọc ticket, khớp nhóm lỗi kiểm chứng của MAST.
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`. Đổi D → A thay đổi cả topology lẫn số model call, nên chưa tách riêng được phần nào gây ra cải thiện. Lỗi của swarm là lỗi cài sẵn (`seed % 5 == 0`), nên tần suất 1/70 không phản ánh tần suất lỗi của LLM thật. Corpus chỉ có 14 claim.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
