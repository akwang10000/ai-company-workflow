# workflows/workflow.md

## 目标

给当前 Phase 1 的研发协作模型提供一个总览，帮助第一次阅读项目的人快速理解：

- 为什么需要 workflow
- 为什么需要 role
- 为什么需要 handoff / review / decision
- 为什么不能退化成“所有人都随便改状态”的任务板

---

## 当前文档定位

本文件是 **总览文档**，不是当前 Phase 1 的执行真相。

它负责：

- 解释协作模型
- 给出各 workflow 的阅读分流
- 概括当前固定主链路

它不负责：

- 重定义状态机
- 重定义 action
- 替代 `workflows/playbooks/*.md`
- 替代 `roles/playbooks/*.md`

真正执行时，以：

- `product/task-status-guards.md`
- `product/task-transition-api-and-actions.md`
- 对应 `workflows/playbooks/*.md`
- 对应 `roles/playbooks/*.md`

为准。

---

## 当前 Phase 1 协作原则

- 任务必须有状态
- 任务必须有当前 owner
- 正式推进必须通过 action-driven transition
- 每次交接都必须落下 handoff 记录
- 审核结论必须落到 `ReviewRecord`
- 交付前必须形成 `DeliverySummaryRecord`
- 高风险动作必须进入 `Waiting Decision`

---

## 当前固定状态集

- `Draft`
- `Ready`
- `In Progress`
- `Waiting Handoff`
- `Waiting Decision`
- `In Review`
- `Rework Required`
- `Ready for Delivery`
- `Done`
- `Archived`
- `Cancelled`

---

## 当前固定动作集

- `ready_task`
- `start_progress`
- `submit_handoff`
- `accept_handoff`
- `request_decision`
- `resume_after_decision`
- `start_review`
- `reject_to_rework`
- `mark_ready_for_delivery`
- `reopen_from_delivery`
- `complete_delivery`
- `archive_task`
- `cancel_task`

---

## 默认协作结构

### 1. feature

典型角色顺序：

`pm_agent -> tech_lead_agent -> dev_agent -> qa_review_agent -> ops_release_agent`

### 2. bugfix

典型角色顺序：

`pm_agent（按需） -> tech_lead_agent -> dev_agent -> qa_review_agent -> ops_release_agent`

### 3. hotfix

典型角色顺序：

`tech_lead_agent -> dev_agent -> qa_review_agent -> ops_release_agent -> approver`

---

## 默认主链路摘要

### 标准 feature / bugfix 主链路

1. `Draft -> Ready`
2. `Ready -> In Progress`
3. `In Progress -> Waiting Handoff -> In Progress`
4. `In Progress -> In Review`
5. `In Review -> Rework Required` 或 `Ready for Delivery`
6. `Ready for Delivery -> Done -> Archived`

### decision 分支

1. `request_decision`
2. task 进入 `Waiting Decision`
3. `DecisionRecord` resolve
4. `resume_after_decision`
5. 回到 `Ready / In Progress / Cancelled`

---

## 什么时候看哪份 workflow 文档

- 任务类型是 `feature`：看 `workflows/playbooks/feature-workflow.md`
- 任务类型是 `bugfix`：看 `workflows/playbooks/bugfix-workflow.md`
- 任务类型是 `hotfix`：看 `workflows/playbooks/hotfix-workflow.md`

---

## 什么时候看哪份 role 文档

- 当前 owner 是 `pm_agent`：看 `roles/playbooks/PM-Agent-playbook.md`
- 当前 owner 是 `tech_lead_agent`：看 `roles/playbooks/Tech-Lead-Agent-playbook.md`
- 当前 owner 是 `dev_agent`：看 `roles/playbooks/Dev-Agent-playbook.md`
- 当前 owner 是 `qa_review_agent`：看 `roles/playbooks/QA-Review-Agent-playbook.md`
- 当前 owner 是 `ops_release_agent`：看 `roles/playbooks/Ops-Release-Agent-playbook.md`

---

## 当前统一结论

workflow 在当前 Phase 1 中的意义，不是“多一份流程图说明”，而是：

- 用主真相约束推进方式
- 用 handoff 承接角色切换
- 用 review / decision / delivery summary 形成闭环
- 让代理知道现在该做什么、做完产出什么、交给谁、什么时候必须停
