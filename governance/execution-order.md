# governance/execution-order.md

## 目标

定义 Codex / 智能体在本项目中执行任务时，**先读什么、后读什么、在什么条件下切换到哪份文档**。

这份文档解决的问题不是“文档有哪些”，而是：

> 当一个任务真的来了，代理应该按什么顺序理解规则、进入 workflow、进入角色、完成交接与收口。

---

## 核心原则

- 先恢复真实进度，再读取静态设计
- 先读 product 主真相，再读执行层手册
- 先判断 task 是否可执行，再进入 workflow
- 先判断是否命中 decision gate，再继续执行
- workflow 与 role 文档负责消费主真相，不再反向定义第二套规则

---

## 统一执行入口顺序

收到任务后，默认按以下顺序处理：

1. `tasks/IMPLEMENTATION-PROGRESS.md`
2. `AGENTS.md`
3. `product/domain-model.md`
4. `product/task-status-guards.md`
5. `product/task-transition-api-and-actions.md`
6. `product/api-contracts.md`
7. `product/implementation-phases.md`
8. `governance/task-schema.md`
9. `governance/decision-gates.md`
10. 对应 `workflows/playbooks/*.md`
11. 对应 `roles/playbooks/*.md`
12. `governance/ready-for-delivery-checklist.md`
13. 当前任务单 / 实际改动上下文

---

## 标准执行分流

### A. 先判断任务是否可执行

至少确认：

- `title`
- `goal`
- `task_type`
- `status`
- `current_owner`
- `acceptance_criteria`

若不完整：

- 先补任务单
- 不进入正式执行

### B. 再判断任务类型

当前 Phase 1 只允许：

- `feature`
- `bugfix`
- `hotfix`

若不是以上类型：

- 不纳入当前主线实现
- 不要回写旧口径到当前文档主真相

### C. 再进入对应 workflow

- `feature` -> `workflows/playbooks/feature-workflow.md`
- `bugfix` -> `workflows/playbooks/bugfix-workflow.md`
- `hotfix` -> `workflows/playbooks/hotfix-workflow.md`

### D. 再进入当前 owner 对应 role 手册

- `pm_agent` -> `roles/playbooks/PM-Agent-playbook.md`
- `tech_lead_agent` -> `roles/playbooks/Tech-Lead-Agent-playbook.md`
- `dev_agent` -> `roles/playbooks/Dev-Agent-playbook.md`
- `qa_review_agent` -> `roles/playbooks/QA-Review-Agent-playbook.md`
- `ops_release_agent` -> `roles/playbooks/Ops-Release-Agent-playbook.md`

---

## 进入收口前必须检查的文档

进入 `Ready for Delivery` 前，必须读取：

- `governance/ready-for-delivery-checklist.md`

若发现：

- 无有效 `ReviewRecord(result=passed)`
- 无 `DeliverySummaryRecord`
- 缺少最小验证摘要
- 缺少明确交付对象

则不得进入 `Ready for Delivery`。

---

## 统一术语要求

当前 Phase 1 只允许使用以下固定口径：

- 状态名：`In Review`，不得写成 `Review`
- 任务类型：`feature / bugfix / hotfix`，不得把 `refactor` 当作当前主线任务类型
- 恢复链路：`request_decision -> resolve decision -> resume_after_decision`
- 审核链路：`submit_handoff -> accept_handoff -> start_review`

---

## 当前阶段统一结论

本文件的作用是：

- 消灭第二套阅读入口
- 让所有执行入口文档共享同一套顺序
- 明确当前实现阶段的主入口是冻结的 product 主真相文档
