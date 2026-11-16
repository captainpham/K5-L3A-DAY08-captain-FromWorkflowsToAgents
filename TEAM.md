# Team

## Thông tin nhóm

- Tên nhóm: captain
- Lớp: L3A
- Repo nhóm: `K5-L3A-DAY08-captain-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt: https://github.com/captainpham/K5-L3A-DAY08-captain-FromWorkflowsToAgents, nhánh `main`, commit chốt `bef5a7e` (trùng `report/team/decision_adr.md`)

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Phạm Cát Tân | 2A202603678 | tanpham090921@gmail.com | captainpham | Shape B (single), integration, benchmark, ADR, research card, báo cáo nhóm |
|   2 | Nguyễn Thị Thu Hà | 2A202603719 | nha253158@gmail.com | nha253158-hub | Shape A (workflow), Shape C (orchestrator), crash & idempotency (CP3–CP4) |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Phạm Cát Tân | Shape B (single) + chặn vòng ReAct dừng giữa chừng, integration, benchmark 4 shape, ADR, research card | 222d40d, bef5a7e, 5a06275 | `src/tests/test_single_step_cap.py` |
| Nguyễn Thị Thu Hà | Shape A (workflow), Shape C (orchestrator), crash & idempotency, crash suite 20/20 | e390e23, 83aaacb, 073e2e9 | `src/tests/test_workflow_security.py` |

## Cá nhân

Mỗi thành viên có một báo cáo riêng trong `report/team/`:

- Phạm Cát Tân: `report/team/2A202603678_PhamCatTan.md`
- Nguyễn Thị Thu Hà: `report/team/2A202603719_NguyenThiThuHa.md`
