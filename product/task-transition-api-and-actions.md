# product/task-transition-api-and-actions.md

## 目标

定义 Phase 1 的 task transition 动作模型、请求体结构、owner side effects、角色权限边界、guard / validator、并发 / 幂等等实现约束。

这是当前 transition 的单一真相文档。

---

## 核心原则

### 1. 不直接开放业务状态任意写入

前端不应直接提交：

- `status = In Progress`
- `status = In Review`
- `status = Ready for Delivery`
- `status = Done`

而应提交明确动作，例如：

- `start_progress`
- `submit_handoff`
- `accept_handoff`
- `request_decision`
- `resume_after_decision`
- `start_review`
- `reject_to_rework`
- `mark_ready_for_delivery`
- `complete_delivery`
- `reopen_from_delivery`
- `cancel_task`
- `archive_task`

### 2. 系统内部可以复杂，用户动作必须简单易懂

前端默认暴露：

- 当前能做什么动作
- 为什么不能做某个动作
- 做完以后会怎样

### 3. actor 身份以服务端认证上下文为准

生产契约中，actor 身份应来自服务端认证上下文。

### 4. owner 变化属于 transition side effect

- 普通 `PATCH /api/tasks/{taskId}` 不负责改 `currentOwner`
- `currentOwner / nextOwner` 的业务变化应由 transition 成功后自动更新

---

## Transition API

### 接口

`POST /api/tasks/{taskId}/transition`

### 请求体建议

```json
{
  "action": "submit_handoff",
  "taskVersion": 7,
  "idempotencyKey": "transition-task_001-submit_handoff-0001",
  "comment": "已完成实现与自测，提交给 QA 回归",
  "payload": {
    "handoff": {
      "fromRole": "dev_agent",
      "toRole": "qa_review_agent",
      "handoffSummary": "完成用户筛选功能与列表分页修复",
      "deliveredArtifacts": ["PR #128", "self-test summary"],
      "risksOrNotes": "移动端筛选器样式仍建议人工看一眼"
    }
  }
}
```

### 返回体建议

```json
{
  "taskId": "task_001",
  "previousStatus": "In Progress",
  "currentStatus": "Waiting Handoff",
  "acceptedAction": "submit_handoff",
  "createdRecords": [
    { "recordType": "HandoffRecord", "recordId": "handoff_record_789" },
    { "recordType": "ExecutionLog", "recordId": "execution_log_456" }
  ],
  "currentOwner": { "roleKey": "dev_agent" },
  "nextOwner": { "roleKey": "qa_review_agent" },
  "nextSuggestedActions": ["accept_handoff", "request_decision"],
  "updatedAt": "2026-03-31T10:00:00+09:00"
}
```

---

## 通用请求字段规则

所有 transition 请求共享以下 envelope 字段：

- `action`
- `taskVersion`
- `idempotencyKey`
- `comment`
- `payload`

### `comment` 规则

- `comment` 是所有 transition 通用的自然语言补充说明字段
- `comment` 默认可选
- `comment` 会写入 `ExecutionLog.comment`
- `comment` 不能替代 action-specific payload 的 required fields
- 若 action 已要求 `reason`、`acceptanceNote`、`issuesFound`、`changeSummary` 等结构化字段，仍必须照 payload 规范提交

---

## Action Registry（Phase 1）

| action | from status | to status | 必要记录 | owner side effect |
|---|---|---|---|---|
| `ready_task` | `Draft` | `Ready` | `ExecutionLog` | 不改 `currentOwner` |
| `start_progress` | `Ready`, `Rework Required` | `In Progress` | `ExecutionLog` | `currentOwner` 保持执行角色 |
| `submit_handoff` | `In Progress` | `Waiting Handoff` | `HandoffRecord`, `ExecutionLog` | `nextOwner = toRole` |
| `accept_handoff` | `Waiting Handoff` | `In Progress` | `ExecutionLog` | `currentOwner = nextOwner`, 然后重算 / 清空 `nextOwner` |
| `request_decision` | `Ready`, `In Progress`, `Waiting Handoff`, `In Review`, `Rework Required`, `Ready for Delivery` | `Waiting Decision` | `DecisionRecord`, `ExecutionLog` | `currentOwner` 默认保持，`nextOwner` 可选 |
| `resume_after_decision` | `Waiting Decision` | `Ready`, `In Progress`, `Cancelled` | `ExecutionLog` | 若恢复到 `In Progress`，按 payload 设置 `currentOwner` |
| `start_review` | `In Progress` | `In Review` | `ReviewRecord(result=pending)`, `ExecutionLog` | `currentOwner` 必须已是审核侧 |
| `reject_to_rework` | `In Review` | `Rework Required` | `ReviewRecord(result=rejected)`, `ExecutionLog` | `currentOwner = returnToRole` |
| `mark_ready_for_delivery` | `In Review` | `Ready for Delivery` | `ReviewRecord(result=passed)`, `DeliverySummaryRecord`, `ExecutionLog` | `currentOwner` 可保持审核 / 交付侧，`nextOwner` 指向交付确认方 |
| `complete_delivery` | `Ready for Delivery` | `Done` | `ExecutionLog` | `currentOwner` 保持收口角色 |
| `reopen_from_delivery` | `Ready for Delivery` | `In Progress` | `ExecutionLog` | `currentOwner = reopenToRole` |
| `cancel_task` | `Draft`, `Ready`, `In Progress`, `Waiting Handoff`, `Waiting Decision` | `Cancelled` | `ExecutionLog` | 清空 `nextOwner` |
| `archive_task` | `Done` | `Archived` | `ExecutionLog` | 清空 `nextOwner` |

---

## owner 变化统一规则

### `currentOwner`

- 表示当前实际责任人
- 只能由 transition 成功后更新

### `nextOwner`

- 表示下一位待接手责任人
- 主要在 `submit_handoff`、`request_decision`、`reopen_from_delivery`、`resume_after_decision` 中更新

### 管理性重新指派边界（Phase 1）

- `PATCH /api/tasks/{taskId}` 只允许维护 `plannedOwner` 与非流转字段
- Phase 1 **不开放**普通 `currentOwner` / `nextOwner` reassign 接口
- 若后续需要独立“管理性改派当前责任人”能力，应在后续 phase 单独设计

---

## Payload 结构

### 1. `handoff` payload

适用 action：`submit_handoff`

#### required fields

- `fromRole`
- `toRole`
- `handoffSummary`
- `deliveredArtifacts`

#### optional fields

- `risksOrNotes`

#### rules

- `fromRole` 必须等于当前 actor role
- `toRole` 不可为空
- `deliveredArtifacts` 不可为空数组
- `toRole` 不能与 `fromRole` 相同

### 2. `handoffAcceptance` payload

适用 action：`accept_handoff`

#### required fields

- `acceptanceNote`

#### rules

- `acceptanceNote` 用于说明接手确认、已知前置物已收到、下一步将如何继续
- 不得用 `comment` 替代 `acceptanceNote`
- `accept_handoff` 成功后，必须将 `currentOwner` 更新为 `nextOwner`

### 3. `decisionRequest` payload

适用 action：`request_decision`

#### required fields

- `reason`
- `options`
- `recommendedOption`
- `approver`

### 4. `decisionResume` payload

适用 action：`resume_after_decision`

#### required fields

- `decisionRecordId`
- `decisionResult`
- `resumeToStatus`
- `resumeToRole`（当 `resumeToStatus = In Progress` 时必填）

#### rules

- `resumeToStatus` 只能是 `Ready` / `In Progress` / `Cancelled`
- `decisionRecordId` 必须指向当前任务最新 active 且已 resolve 的 decision

### 5. `reviewStart` payload

适用 action：`start_review`

#### required fields

- `reviewType`
- `checklistSummary`

### 6. `reviewReject` payload

适用 action：`reject_to_rework`

#### required fields

- `reviewType`
- `checklistSummary`
- `issuesFound`
- `nextAction`
- `returnToRole`

### 7. `deliverySummary` payload

适用 action：`mark_ready_for_delivery`

#### required fields

- `changeSummary`
- `affectedScope`
- `validationSummary`
- `remainingRisks`
- `deliveryTo[]`

#### rules

- `deliveryTo` 为数组，不可为空数组
- `deliveryTo` 表示当前任务进入交付确认时的明确交付对象 / 最终确认方
- `mark_ready_for_delivery` 成功时，服务端基于该 payload 创建 `DeliverySummaryRecord`

### 8. `reopen` payload

适用 action：`reopen_from_delivery`

#### required fields

- `reason`
- `reopenToRole`

### 9. `cancel` payload

适用 action：`cancel_task`

#### recommended fields

- `reason`

---

## Guard 与 Validator 规则

后端必须先校验：

- 当前状态是否允许该 action
- actor 是否有权限执行该 action
- 必填 payload 是否齐全
- 前置记录是否满足要求
- 当前任务是否处于可流转状态

### 重点 guard

- `start_review` 前，当前 owner 必须是审核侧
- `mark_ready_for_delivery` 前，必须已完成审核通过所需检查
- `resume_after_decision` 前，相关 `DecisionRecord` 必须已 resolve

---

## 角色权限最小矩阵（建议）

| role | 可执行动作 |
|---|---|
| `pm_agent` | `ready_task`, `start_progress`, `submit_handoff`, `request_decision`, `cancel_task` |
| `tech_lead_agent` | `start_progress`, `submit_handoff`, `accept_handoff`, `start_review`, `request_decision`, `cancel_task` |
| `dev_agent` | `start_progress`, `submit_handoff`, `accept_handoff`, `request_decision` |
| `qa_review_agent` | `accept_handoff`, `start_review`, `reject_to_rework`, `mark_ready_for_delivery`, `request_decision` |
| `ops_release_agent` | `complete_delivery`, `archive_task`, `request_decision`, `reopen_from_delivery` |
| `approver` | decision approve / reject（通过 Decision API，不走普通 task transition） |

---

## 并发与幂等最小规则

Phase 1 最少要做到：

- transition 请求必须支持 `taskVersion` 或等价版本号校验
- 基于旧版本提交 transition 时，后端必须拒绝，并返回 `TRANSITION_VERSION_CONFLICT`
- 同一任务 transition 失败时返回明确冲突错误，不允许静默覆盖
- 对 high danger 动作，至少要支持服务端去重或 `idempotencyKey`

---

## 错误码建议

- `TRANSITION_ACTION_NOT_ALLOWED`
- `TRANSITION_ROLE_NOT_ALLOWED`
- `TRANSITION_PAYLOAD_INVALID`
- `TRANSITION_GUARD_FAILED`
- `TRANSITION_OWNER_MISMATCH`
- `TRANSITION_DECISION_NOT_RESOLVED`
- `TRANSITION_DECISION_RESULT_MISMATCH`
- `TRANSITION_REVIEW_PREREQUISITE_MISSING`
- `TRANSITION_DELIVERY_SUMMARY_MISSING`
- `TRANSITION_VERSION_CONFLICT`

---

## 关键验收 Case（Phase 1）

1. `submit_handoff` 后必须生成 `HandoffRecord`
2. `accept_handoff` 后 owner 必须转为接手角色
3. `start_review` 不得隐式完成 owner 迁移
4. `reject_to_rework` 后必须进入 `Rework Required`
5. `mark_ready_for_delivery` 必须同时创建 `ReviewRecord(result=passed)` 与 `DeliverySummaryRecord`
6. decision resolve 不会自动恢复任务
7. 必须显式 `resume_after_decision`

---

## 当前阶段统一结论

Phase 1 的 transition 实现必须坚持 4 件事：

1. 前端发 action，不发裸状态
2. owner 变化由 transition 驱动，不由普通 PATCH 驱动
3. decision resolve 与 task resume 分开
4. review 开始、驳回、通过都要有显式记录
