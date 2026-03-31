# workflows/playbooks/feature-workflow.md

## 目标

定义标准 `feature` 任务从提出到交付的 **执行手册**，而不是高层 SOP 摘要。

---

## 适用范围

适用于：

- 新增功能
- 明确需求的小型迭代
- 可拆解为标准研发流程的 `feature`

不直接覆盖：

- 正在扩散的线上事故
- 需要分钟级响应的 hotfix
- 超出当前 Phase 1 范围的大型重构

---

## 默认角色顺序

- `pm_agent`
- `tech_lead_agent`
- `dev_agent`
- `qa_review_agent`
- `ops_release_agent`

---

## 默认主链路

1. `Draft -> Ready`
2. `Ready -> In Progress`（PM 澄清）
3. `In Progress -> Waiting Handoff -> In Progress`（PM -> Tech Lead）
4. `In Progress -> Waiting Handoff -> In Progress`（Tech Lead -> Dev）
5. `In Progress -> Waiting Handoff -> In Progress`（Dev -> QA）
6. `In Progress -> In Review`（QA 开始审核）
7. `In Review -> Ready for Delivery`
8. `Ready for Delivery -> Done -> Archived`

---

## 分阶段执行要求

### 阶段 A：任务成形（PM）

#### 当前状态与 owner 条件

- `Draft`
- `Ready`
- `In Progress` 且 `currentOwner = pm_agent`

#### 允许动作

- `ready_task`
- `start_progress`
- `request_decision`
- `submit_handoff`

#### 最低 payload / record 要求

| 动作 | 最低 payload | 必产出 record | owner 约束 |
|---|---|---|---|
| `ready_task` | `readinessSummary` | `ExecutionLog` | 当前 owner 应为创建侧 / PM 侧 |
| `start_progress` | `workSummary` | `ExecutionLog` | 当前 owner 保持 `pm_agent` |
| `request_decision` | `decisionReason`, `options[]`, `recommendedOption` | `DecisionRecord`, `ExecutionLog` | PM 可发起 |
| `submit_handoff` | `fromRole`, `toRole`, `handoffSummary`, `deliveredArtifacts[]` | `HandoffRecord`, `ExecutionLog` | `toRole = tech_lead_agent` |

#### 必填产物

- 清晰 `goal`
- `non_goals`
- `acceptance_criteria`
- `definition_of_done`
- handoff 给 Tech Lead 的摘要

---

### 阶段 B：技术方案（Tech Lead）

#### 当前状态与 owner 条件

- `Waiting Handoff` 且 `nextOwner = tech_lead_agent`
- `In Progress` 且 `currentOwner = tech_lead_agent`

#### 允许动作

- `accept_handoff`
- `request_decision`
- `submit_handoff`

#### 最低 payload / record 要求

| 动作 | 最低 payload | 必产出 record | owner 约束 |
|---|---|---|---|
| `accept_handoff` | `acceptanceNote` | `ExecutionLog` | `nextOwner` 必须是 `tech_lead_agent` |
| `request_decision` | `decisionReason`, `options[]`, `recommendedOption` | `DecisionRecord`, `ExecutionLog` | Tech Lead 可发起 |
| `submit_handoff` | `fromRole`, `toRole`, `handoffSummary`, `deliveredArtifacts[]` | `HandoffRecord`, `ExecutionLog` | `toRole = dev_agent` |

#### 必填产物

- 实现路径摘要
- 风险清单
- 依赖与约束
- handoff 给 Dev 的方案摘要

---

### 阶段 C：实现与自测（Dev）

#### 当前状态与 owner 条件

- `Waiting Handoff` 且 `nextOwner = dev_agent`
- `In Progress` 且 `currentOwner = dev_agent`
- `Rework Required` 且返工 owner 已回到 `dev_agent`

#### 允许动作

- `accept_handoff`
- `request_decision`
- `submit_handoff`
- `restart_rework`

#### 最低 payload / record 要求

| 动作 | 最低 payload | 必产出 record | owner 约束 |
|---|---|---|---|
| `accept_handoff` | `acceptanceNote` | `ExecutionLog` | `nextOwner = dev_agent` |
| `restart_rework` | `reworkPlan` | `ExecutionLog` | 当前 owner 应为返工执行人 |
| `submit_handoff` | `fromRole`, `toRole`, `handoffSummary`, `deliveredArtifacts[]` | `HandoffRecord`, `ExecutionLog` | `toRole = qa_review_agent` |
| `request_decision` | `decisionReason`, `options[]`, `recommendedOption` | `DecisionRecord`, `ExecutionLog` | Dev 可发起 |

#### 必填产物

- 修改点清单
- 自测摘要
- 未测项
- 已知风险
- handoff 给 QA 的交接摘要与产物索引

---

### 阶段 D：审核（QA）

#### 当前状态与 owner 条件

- `Waiting Handoff` 且 `nextOwner = qa_review_agent`
- `In Progress` 且 `currentOwner = qa_review_agent`
- `In Review` 且 `currentOwner = qa_review_agent`

#### 允许动作

- `accept_handoff`
- `start_review`
- `reject_to_rework`
- `mark_ready_for_delivery`
- `request_decision`

#### 最低 payload / record 要求

| 动作 | 最低 payload | 必产出 record | owner 约束 |
|---|---|---|---|
| `accept_handoff` | `acceptanceNote` | `ExecutionLog` | `nextOwner = qa_review_agent` |
| `start_review` | `reviewType`, `checklistSummary` | `ReviewRecord(result=pending)`, `ExecutionLog` | 当前 owner 必须已是审核侧 |
| `reject_to_rework` | `issuesFound[]`, `nextAction`, `returnToRole` | `ReviewRecord(result=rejected)`, `ExecutionLog` | 当前状态必须为 `In Review` |
| `mark_ready_for_delivery` | `reviewSummary`, `changeSummary`, `validationSummary`, `remainingRisks`, `deliveryTo[]` | `ReviewRecord(result=passed)`, `DeliverySummaryRecord`, `ExecutionLog` | 当前状态必须为 `In Review` |

#### 必填产物

- `ReviewRecord(result=pending / rejected / passed)`
- 审核摘要
- 问题点或放行条件
- 通过时的 `DeliverySummaryRecord`

---

### 阶段 E：交付与归档（Ops）

#### 当前状态与 owner 条件

- `Ready for Delivery`
- `Done`

#### 允许动作

- `complete_delivery`
- `archive_task`
- `reopen_from_delivery`
- `request_decision`

#### 最低 payload / record 要求

| 动作 | 最低 payload | 必产出 record | owner 约束 |
|---|---|---|---|
| `complete_delivery` | `finalResult`, `artifacts[]`, `deliveryNote` | `ExecutionLog` | 进入前应已有 `DeliverySummaryRecord` |
| `archive_task` | `archiveReason` | `ExecutionLog` | 当前状态必须为 `Done` |
| `reopen_from_delivery` | `reopenReason`, `returnToRole` | `ExecutionLog` | 当前状态应为 `Ready for Delivery` 或 `Done` |

#### 必填产物

- 交付说明 / 发布说明
- 产物索引
- 必要的 follow-up 标记

---

## 关键 decision gate

以下情况必须执行 `request_decision`：

- PM 发现需求边界无法确认
- Tech Lead 给出多个差异显著方案
- QA 认为风险明显但业务想强推
- 交付前是否真正上线需要业务拍板

---

## Ready for Delivery 前必须满足

- 最新有效 `ReviewRecord(result=passed)` 已存在
- `DeliverySummaryRecord` 已存在
- 交付对象明确
- 不存在未决 `Waiting Decision`

---

## 当前阶段统一结论

`feature` workflow 在当前阶段不是“谁先做后做”的说明书，而是：

- 以状态为边界
- 以 action 为推进方式
- 以 record 为事实沉淀
- 以 handoff 为责任转移
