# workflows/playbooks/hotfix-workflow.md

## 目标

定义紧急 `hotfix` 任务的执行手册，在高时效压力下仍保留最基本的责任边界、风险控制和记录。

---

## 适用范围

适用于：

- 正在影响线上用户的严重问题
- 收入、登录、支付、权限等关键路径异常
- 需要尽快止血的生产问题

---

## 核心原则

- 先止血，再补完备
- 可以压缩流程，不能取消记录
- 高风险生产动作前必须人工拍板
- 必须明确当前是临时止血还是根因修复

---

## 默认角色顺序

- `tech_lead_agent`
- `dev_agent`
- `qa_review_agent`
- `ops_release_agent`
- `approver`

---

## 默认主链路

1. `Draft -> Ready`
2. `Ready -> In Progress`（事故确认 / 止血方案）
3. 需要拍板时：`request_decision -> Waiting Decision -> resume_after_decision`
4. `In Progress -> Waiting Handoff -> In Progress`（Tech Lead -> Dev 或 Dev -> QA）
5. `In Progress -> In Review`
6. `In Review -> Ready for Delivery`
7. `Ready for Delivery -> Done`
8. 若为临时止血，必须补 follow-up 任务

---

## 分阶段执行要求

### 阶段 A：事故确认与止血方案

#### 当前状态与 owner 条件

- `Draft`
- `Ready`
- `In Progress` 且 owner 为 `tech_lead_agent`

#### 允许动作

- `ready_task`
- `start_progress`
- `request_decision`
- `submit_handoff`

#### 最低 payload / record 要求

| 动作 | 最低 payload | 必产出 record |
|---|---|---|
| `ready_task` | 无额外必填 payload | `ExecutionLog` |
| `start_progress` | 无额外必填 payload | `ExecutionLog` |
| `request_decision` | `reason`, `options[]`, `recommendedOption`, `approver` | `DecisionRecord`, `ExecutionLog` |
| `submit_handoff` | `fromRole`, `toRole`, `handoffSummary`, `deliveredArtifacts[]` | `HandoffRecord`, `ExecutionLog` |

#### 必须补齐

- 事故级别
- 影响范围
- 当前止血目标
- 是否需要人工拍板

---

### 阶段 B：实施 hotfix

#### 当前状态与 owner 条件

- `Waiting Handoff` 且 `nextOwner = dev_agent`
- `In Progress` 且 `currentOwner = dev_agent`

#### 允许动作

- `accept_handoff`
- `request_decision`
- `submit_handoff`

#### 最低 payload / record 要求

| 动作 | 最低 payload | 必产出 record |
|---|---|---|
| `accept_handoff` | `acceptanceNote` | `ExecutionLog` |
| `submit_handoff` | `fromRole`, `toRole`, `handoffSummary`, `deliveredArtifacts[]` | `HandoffRecord`, `ExecutionLog` |
| `request_decision` | `reason`, `options[]`, `recommendedOption`, `approver` | `DecisionRecord`, `ExecutionLog` |

#### 必须补齐

- 最小改动路径
- 风险点
- 回滚 / 降级风险说明

---

### 阶段 C：最小关键验证

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
| `reject_to_rework` | `reviewType`, `checklistSummary`, `issuesFound[]`, `nextAction`, `returnToRole` | `ReviewRecord(result=rejected)`, `ExecutionLog` |
| `mark_ready_for_delivery` | `changeSummary`, `affectedScope`, `validationSummary`, `remainingRisks`, `deliveryTo[]` | `ReviewRecord(result=passed)`, `DeliverySummaryRecord`, `ExecutionLog` |

#### 必须补齐

- 最小关键链路验证结果
- 未覆盖项
- 剩余线上风险

---

### 阶段 D：收口与 follow-up

#### 当前状态与 owner 条件

- `Ready for Delivery`
- `Done`

#### 允许动作

当 `status = Ready for Delivery`：

- `complete_delivery`
- `reopen_from_delivery`

当 `status = Done`：

- `archive_task`

#### 必须补齐

- 当前是临时止血还是完整修复
- 是否需要 follow-up bugfix / feature 任务
- 最终交付说明

---

## 必须停机点

以下情况必须进入 `Waiting Decision`：

- 生产数据修复
- 回滚或大范围降级
- 关闭核心功能
- 对外说明或承诺恢复时间

---

## 当前阶段统一结论

`hotfix` workflow 允许压缩流程，但不允许压掉：

- 决策门
- 审核记录
- 交付摘要
- follow-up 标记
