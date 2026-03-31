# workflows/playbooks/bugfix-workflow.md

## 目标

定义常规 `bugfix` 任务从发现、定位、修复到验证、收口的执行手册。

---

## 适用范围

适用于：

- 已知问题修复
- 可复现的功能异常
- 明确范围的回归问题

不直接覆盖：

- 正在扩散的生产事故
- 需要分钟级响应的 hotfix

---

## 默认角色顺序

- `pm_agent`（按需）
- `tech_lead_agent`
- `dev_agent`
- `qa_review_agent`
- `ops_release_agent`

---

## 默认主链路

1. `Draft -> Ready`
2. `Ready -> In Progress`（补复现与验收）
3. `In Progress -> Waiting Handoff -> In Progress`（Tech Lead / PM -> Dev）
4. `In Progress -> Waiting Handoff -> In Progress`（Dev -> QA）
5. `In Progress -> In Review`
6. `In Review -> Rework Required` 或 `Ready for Delivery`
7. `Ready for Delivery -> Done -> Archived`

---

## 分阶段执行要求

### 阶段 A：问题成形

#### 当前状态与 owner 条件

- `Draft`
- `Ready`
- `In Progress` 且 owner 为 `pm_agent` 或 `tech_lead_agent`

#### 允许动作

- `ready_task`
- `start_progress`
- `request_decision`
- `submit_handoff`

#### 最低 payload / record 要求

| 动作 | 最低 payload | 必产出 record |
|---|---|---|
| `ready_task` | `readinessSummary` | `ExecutionLog` |
| `start_progress` | `workSummary` | `ExecutionLog` |
| `request_decision` | `decisionReason`, `options[]`, `recommendedOption` | `DecisionRecord`, `ExecutionLog` |
| `submit_handoff` | `fromRole`, `toRole`, `handoffSummary`, `deliveredArtifacts[]` | `HandoffRecord`, `ExecutionLog` |

#### 必须补齐

- 复现条件
- 影响范围
- 验收标准
- 是否属于 hotfix 而不是普通 bugfix

---

### 阶段 B：定位与修复路径

#### 当前状态与 owner 条件

- `Waiting Handoff` 且 `nextOwner = tech_lead_agent` 或 `dev_agent`
- `In Progress` 且 `currentOwner = tech_lead_agent` 或 `dev_agent`

#### 允许动作

- `accept_handoff`
- `request_decision`
- `submit_handoff`

#### 最低 payload / record 要求

| 动作 | 最低 payload | 必产出 record |
|---|---|---|
| `accept_handoff` | `acceptanceNote` | `ExecutionLog` |
| `request_decision` | `decisionReason`, `options[]`, `recommendedOption` | `DecisionRecord`, `ExecutionLog` |
| `submit_handoff` | `fromRole`, `toRole`, `handoffSummary`, `deliveredArtifacts[]` | `HandoffRecord`, `ExecutionLog` |

#### 必须补齐

- 根因初判
- 修复范围
- 回归风险

---

### 阶段 C：修复与自测

#### 当前状态与 owner 条件

- `In Progress` 且 `currentOwner = dev_agent`
- `Rework Required` 且返工 owner 已回到 `dev_agent`

#### 允许动作

- `restart_rework`
- `submit_handoff`
- `request_decision`

#### 最低 payload / record 要求

| 动作 | 最低 payload | 必产出 record |
|---|---|---|
| `restart_rework` | `reworkPlan` | `ExecutionLog` |
| `submit_handoff` | `fromRole`, `toRole`, `handoffSummary`, `deliveredArtifacts[]` | `HandoffRecord`, `ExecutionLog` |
| `request_decision` | `decisionReason`, `options[]`, `recommendedOption` | `DecisionRecord`, `ExecutionLog` |

#### 必须补齐

- 实际改动点
- 复现是否消失
- 最小回归说明
- 剩余风险

---

### 阶段 D：审核与回归

#### 当前状态与 owner 条件

- `Waiting Handoff` 且 `nextOwner = qa_review_agent`
- `In Progress` 且 `currentOwner = qa_review_agent`
- `In Review`

#### 允许动作

- `accept_handoff`
- `start_review`
- `reject_to_rework`
- `mark_ready_for_delivery`
- `request_decision`

#### 最低 payload / record 要求

| 动作 | 最低 payload | 必产出 record |
|---|---|---|
| `start_review` | `reviewType`, `checklistSummary` | `ReviewRecord(result=pending)`, `ExecutionLog` |
| `reject_to_rework` | `issuesFound[]`, `nextAction`, `returnToRole` | `ReviewRecord(result=rejected)`, `ExecutionLog` |
| `mark_ready_for_delivery` | `reviewSummary`, `changeSummary`, `validationSummary`, `remainingRisks`, `deliveryTo[]` | `ReviewRecord(result=passed)`, `DeliverySummaryRecord`, `ExecutionLog` |

#### 必须补齐

- `ReviewRecord`
- 是否通过验收
- 是否通过相关回归

---

### 阶段 E：交付与归档

#### 当前状态与 owner 条件

- `Ready for Delivery`
- `Done`

#### 允许动作

- `complete_delivery`
- `archive_task`
- `reopen_from_delivery`

#### 必须补齐

- 是否接受当前修复范围
- 是否需要 follow-up
- 最终交付说明

---

## 必须停机的情况

- 根因不明但影响面过大
- 修复方案可能引入高风险兼容性问题
- 需要直接改生产数据

---

## 当前阶段统一结论

`bugfix` workflow 的重点不是“按部就班”，而是：

- 先把复现与范围说清
- 再把修复与回归写清
- 最后通过审核与交付摘要收口
