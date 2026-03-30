# product/task-transition-api-and-actions.md

## 目标

把 `product/task-status-guards.md` 进一步压成**可直接施工的 transition 实现规范**，确保 Codex、后端、前端对“任务状态推进”使用同一套动作模型，而不是各自理解一套状态切换逻辑。

这份文档主要回答：

- 为什么不能让前端直接修改 `status`
- `POST /api/tasks/{taskId}/transition` 应该如何设计
- 每个 action 的允许来源状态、必填 payload、生成记录、owner 变化、目标状态是什么
- 哪些角色允许执行哪些动作
- 哪些动作需要二次确认、哪些属于危险动作
- 前端应暴露哪些动作按钮、弹层和禁用原因
- 后端应如何实现 guard、validator、side effects、错误码
- 决策审批与任务恢复之间的边界是什么
- 最关键的验收 case 应如何定义

它是 **task transition 的单一真相来源**。

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
- `request_decision`
- `start_review`
- `reject_to_rework`
- `mark_ready_for_delivery`
- `complete_delivery`

原因：

- 动作比状态更可解释
- 动作更容易校验必填记录
- 动作更容易沉淀审计日志
- 动作更适合前端做按钮和表单
- 动作更适合后端做 guard 和 side effects

### 2. 系统内部可以复杂，用户动作必须简单易懂

后端可以维护复杂状态机、guard、record、projection；  
但前端对用户暴露的，默认应始终是：

- 当前能做什么动作
- 为什么不能做某个动作
- 做完以后会怎样

而不是要求用户理解内部实现细节。

### 3. actor 身份以服务端认证上下文为准

- 生产契约中，actor 身份应来自服务端认证上下文
- 请求体里的 `actorUserId` / `actorRoleKey` 只能作为调试辅助信息，不能作为最终信任来源
- 若 Phase 1 demo 暂未接入完整认证，可在 dev 模式允许传入，但实现时必须显式标注为临时模式

### 4. owner 变化属于 transition side effect

- 普通 `PATCH /api/tasks/{taskId}` 不负责改 `currentOwner`
- `currentOwner` / `nextOwner` 的业务变化应由 transition 成功后自动更新
- handoff、review、reopen、resume_after_decision 的 owner 变化必须在这份文档中定义清楚

---

## transition API

### 接口

`POST /api/tasks/{taskId}/transition`

### 请求体建议

```json
{
  "action": "submit_handoff",
  "taskVersion": 7,
  "comment": "已完成实现与自测，提交给 QA 回归",
  "payload": {
    "handoff": {
      "fromRole": "dev_agent",
      "toRole": "qa_review_agent",
      "handoffSummary": "完成用户筛选功能与列表分页修复",
      "deliveredArtifacts": [
        "PR #128",
        "self-test summary"
      ],
      "risksOrNotes": "移动端筛选器样式仍建议人工看一眼"
    }
  }
}
```

### 最小并发字段建议

```json
{
  "taskVersion": 7,
  "idempotencyKey": "transition-task_001-submit_handoff-0001"
}
```

说明：
- `taskVersion`：用于最小乐观锁校验，防止旧状态重复提交
- `idempotencyKey`：对高风险动作建议传入；若服务端启用去重，应以它识别重复请求

### 开发期可选调试字段

```json
{
  "debugActorUserId": "user_123",
  "debugActorRoleKey": "dev_agent"
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
    {
      "recordType": "HandoffRecord",
      "recordId": "handoff_record_789"
    },
    {
      "recordType": "ExecutionLog",
      "recordId": "execution_log_456"
    }
  ],
  "currentOwner": {
    "roleKey": "dev_agent"
  },
  "nextOwner": {
    "roleKey": "qa_review_agent"
  },
  "nextSuggestedActions": [
    "accept_handoff",
    "request_decision"
  ],
  "updatedAt": "2026-03-30T10:00:00+09:00"
}
```

---

## owner 变化统一规则

这是 Phase 1 必须收死的实现规则。

### 1. `plannedOwner`

- 仅表示计划负责人
- 可在任务创建或补字段阶段通过普通更新维护
- Phase 1 若需要人工调整 owner，默认只允许调整 `plannedOwner`，不开放普通 `currentOwner` / `nextOwner` 改派通道

### 2. `currentOwner`

- 表示当前业务责任人
- 只能由 transition 成功后更新

### 3. `nextOwner`

- 表示下一位待接手责任人
- 主要在 `submit_handoff`、`request_decision`、`reopen_from_delivery`、`resume_after_decision` 等动作中更新

### 3.5 管理性重新指派边界（Phase 1）

- `PATCH /api/tasks/{taskId}` 只允许维护 `plannedOwner` 与非流转字段
- Phase 1 **不开放**普通 `currentOwner` / `nextOwner` reassign 接口
- `currentOwner` / `nextOwner` 的业务变化，一律视为 transition side effect
- 若后续需要独立“管理性改派当前责任人”能力，应在后续 phase 单独设计，不回写到当前主契约

### 4. handoff 场景

- `submit_handoff` 成功后：
  - `currentOwner` 维持原角色
  - `nextOwner` 变为 `handoff.toRole`
  - 状态进入 `Waiting Handoff`
- `accept_handoff` 成功后：
  - `currentOwner` 切为接手角色
  - `nextOwner` 清空或按业务重算
  - 状态回到 `In Progress`

### 5. review 场景

- `start_review` **不会**隐式改 owner
- 因此若 reviewer 不是当前 owner，必须先走 handoff
- `start_review` 只在当前 owner 已经是 review-capable 角色时允许执行

### 6. decision 场景

- `request_decision` 成功后：
  - 任务状态进入 `Waiting Decision`
  - `currentOwner` 默认保持原角色
  - `nextOwner` 可选，表示决策后预计回到谁继续推进
- `resume_after_decision` 成功后：
  - 根据 payload 中声明的恢复角色更新 `currentOwner`
  - `nextOwner` 清空或重算

### 7. reopen 场景

- `reopen_from_delivery` 成功后：
  - 状态回到 `In Progress`
  - `currentOwner` 切回指定返工角色
  - `nextOwner` 清空或按业务重算

---

## action registry（Phase 1）

| action | allowed from | target status | 必填 payload | 生成记录 | current owner 变化 | next owner 变化 | 典型执行角色 | requires confirmation | danger level |
|---|---|---|---|---|---|---|---|---|---|
| `submit_ready` | `Draft` | `Ready` | 无 | `ExecutionLog` | 不变 | 可清空或保留 planned owner | `pm_agent` / `tech_lead_agent` | 否 | low |
| `start_progress` | `Ready` | `In Progress` | 无 | `ExecutionLog` | 设为当前接手角色 | 清空 | 当前接手角色 | 否 | low |
| `submit_handoff` | `In Progress` | `Waiting Handoff` | `handoff` | `HandoffRecord` + `ExecutionLog` | 不变 | 设为 `handoff.toRole` | 当前执行角色 | 是 | medium |
| `accept_handoff` | `Waiting Handoff` | `In Progress` | 无 | `ExecutionLog` | 切为 `nextOwner` | 清空 | 下一接手角色 | 否 | low |
| `request_decision` | `Ready` / `In Progress` / `Waiting Handoff` / `In Review` / `Rework Required` / `Ready for Delivery` | `Waiting Decision` | `decision` | `DecisionRecord` + `ExecutionLog` | 默认不变 | 可选设置 decision 后预计接手角色 | 当前责任角色 | 是 | medium |
| `resume_after_decision` | `Waiting Decision` | `Ready` / `In Progress` / `Cancelled` | `decisionResume` | `ExecutionLog` | 按 `resumeToRole` 切换 | 清空或重算 | `pm_agent` / `tech_lead_agent` / `ops_release_agent` / `approver` | 是 | medium |
| `start_review` | `In Progress` | `In Review` | `reviewStart` | `ReviewRecord` + `ExecutionLog` | 不变 | 清空 | `qa_review_agent` / `tech_lead_agent` | 否 | low |
| `reject_to_rework` | `In Review` | `Rework Required` | `reviewReject` | `ReviewRecord` + `ExecutionLog` | 默认切回返工角色 | 可选重算 | `qa_review_agent` / `tech_lead_agent` | 是 | medium |
| `restart_rework` | `Rework Required` | `In Progress` | 无 | `ExecutionLog` | 设为当前返工角色 | 清空 | `dev_agent` / `tech_lead_agent` | 否 | low |
| `mark_ready_for_delivery` | `In Review` | `Ready for Delivery` | `reviewPass` + `deliverySummary` | `ReviewRecord` + `DeliverySummaryRecord` + `ExecutionLog` | 默认保持交付前责任角色 | 可选设置交付角色 | `qa_review_agent` / `ops_release_agent` / `tech_lead_agent` | 是 | medium |
| `reopen_from_delivery` | `Ready for Delivery` | `In Progress` | `reopen` | `ExecutionLog` | 切为 `expectedNextOwnerRole` | 清空 | `pm_agent` / `tech_lead_agent` / `qa_review_agent` / `ops_release_agent` | 是 | high |
| `complete_delivery` | `Ready for Delivery` | `Done` | 无 | `ExecutionLog` | 保持或切为交付确认角色 | 清空 | `ops_release_agent` / `pm_agent` | 是 | high |
| `archive_task` | `Done` | `Archived` | 无 | `ExecutionLog` | 不变 | 清空 | `ops_release_agent` / `system` | 是 | low |
| `cancel_task` | `Draft` / `Ready` / `In Progress` / `Waiting Handoff` / `Waiting Decision` | `Cancelled` | 建议 `cancel` | `ExecutionLog` | 终态，不再使用 | 清空 | `pm_agent` / `tech_lead_agent` / `approver` | 是 | high |

---

## 角色权限矩阵（Phase 1）

> 说明：这里给的是默认权限边界。若后续需要更细的权限系统，应在实现层扩展，但 Phase 1 不应突破这张表的主边界。

| action | allowed roles | denied roles | 说明 |
|---|---|---|---|
| `submit_ready` | `pm_agent`, `tech_lead_agent` | `dev_agent`, `qa_review_agent`, `ops_release_agent` | 任务进入可执行态前，应由上游职责角色完成 |
| `start_progress` | `pm_agent`, `tech_lead_agent`, `dev_agent`, `qa_review_agent`, `ops_release_agent` | 无默认拒绝，但需与当前接手角色一致 | 必须是当前接手角色开始执行 |
| `submit_handoff` | `pm_agent`, `tech_lead_agent`, `dev_agent`, `qa_review_agent`, `ops_release_agent` | 无默认拒绝，但需与 `currentOwner` 一致 | 必须由当前执行角色提交 |
| `accept_handoff` | `pm_agent`, `tech_lead_agent`, `dev_agent`, `qa_review_agent`, `ops_release_agent` | 无默认拒绝，但需与 `nextOwner` 一致 | 必须由下一接手角色接收 |
| `request_decision` | `pm_agent`, `tech_lead_agent`, `dev_agent`, `qa_review_agent`, `ops_release_agent` | 无 | 当前责任角色可请求拍板 |
| `resume_after_decision` | `pm_agent`, `tech_lead_agent`, `ops_release_agent`, `approver` | `dev_agent`, `qa_review_agent` | 默认由拍板后负责重新组织推进的角色执行 |
| `start_review` | `tech_lead_agent`, `qa_review_agent` | `dev_agent`, `ops_release_agent` | 默认不由 Dev 自己把自己送入正式 Review |
| `reject_to_rework` | `tech_lead_agent`, `qa_review_agent` | `dev_agent`, `ops_release_agent` | 返工结论应由审核侧给出 |
| `restart_rework` | `dev_agent`, `tech_lead_agent` | `ops_release_agent` | 默认由返工执行侧重新开始 |
| `mark_ready_for_delivery` | `qa_review_agent`, `ops_release_agent`, `tech_lead_agent` | `dev_agent` | 默认不允许 Dev 直接宣布可交付 |
| `reopen_from_delivery` | `pm_agent`, `tech_lead_agent`, `qa_review_agent`, `ops_release_agent` | `dev_agent` | 重新打开应由上游 / 审核 / 交付侧触发 |
| `complete_delivery` | `ops_release_agent`, `pm_agent` | `dev_agent` | 默认由交付 / 确认侧完成 |
| `archive_task` | `ops_release_agent`, `system` | `dev_agent`, `qa_review_agent` | 归档不是开发动作 |
| `cancel_task` | `pm_agent`, `tech_lead_agent`, `approver` | `dev_agent`, `qa_review_agent`, `ops_release_agent` | 取消任务默认属于上游或拍板职责 |

---

## ReviewRecord 生命周期统一规则（Phase 1）

为避免实现分叉，Phase 1 统一采用 **append-only ReviewRecord**：

- `start_review`：新建一条 `ReviewRecord(review_status=started, result=pending)`
- `reject_to_rework`：新建一条 `ReviewRecord(review_status=completed, result=rejected)`
- `mark_ready_for_delivery`：新建一条 `ReviewRecord(review_status=completed, result=passed)`
- Phase 1 不要求在旧 `ReviewRecord` 上原地更新最终审核结果
- `Ready for Delivery` 的审核前置条件，以最新一条 `review_status=completed` 且 `result=passed` 的 `ReviewRecord` 为准

## payload 规则

### 通用规则

- `action`：必填，必须为已注册 action 枚举值
- `comment`：可选，但对 medium / high danger action 建议填写
- `payload`：按 action 决定是否必填
- actor 身份默认来自服务端认证上下文，生产契约不应把请求体里的 actor 字段当作可信输入

### 空值规则

- 必填字符串不可为空字符串
- 必填数组不可为空数组
- 可选数组允许为空数组
- 可选字符串允许为空，但不建议对摘要类字段留空

---

## payload 结构建议与字段强约束

### 1. `handoff` payload

适用 action：

- `submit_handoff`

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
- `handoffSummary` 不可为空字符串
- `deliveredArtifacts` 不可为空数组
- `toRole` 不能与 `fromRole` 相同

```json
{
  "handoff": {
    "fromRole": "dev_agent",
    "toRole": "qa_review_agent",
    "handoffSummary": "完成用户筛选功能与列表分页修复",
    "deliveredArtifacts": [
      "PR #128",
      "self-test summary"
    ],
    "risksOrNotes": "移动端样式建议人工巡检"
  }
}
```

---

### 2. `decision` payload

适用 action：

- `request_decision`

#### required fields

- `decisionTopic`
- `decisionQuestion`
- `options`
- `recommendedOption`
- `riskSummary`

#### optional fields

- `approverRoleKey`
- `expectedResumeRole`
- `expectedResumeStatus`

#### rules

- `options` 不可为空数组
- `recommendedOption` 必须出现在 `options` 中
- 若传 `expectedResumeStatus`，只能是 `Ready` / `In Progress` / `Cancelled`

```json
{
  "decision": {
    "decisionTopic": "scope change",
    "decisionQuestion": "是否砍掉移动端高级筛选",
    "options": ["keep", "cut", "delay"],
    "recommendedOption": "cut",
    "riskSummary": "当前工期不足",
    "approverRoleKey": "approver",
    "expectedResumeRole": "pm_agent",
    "expectedResumeStatus": "Ready"
  }
}
```

---

### 3. `decisionResume` payload

适用 action：

- `resume_after_decision`

#### required fields

- `decisionRecordId`
- `decisionResult`
- `resumeToStatus`
- `resumeToRole`

#### rules

- `decisionRecordId` 必须指向当前任务最新 active 且已 resolve 的 decision
- `decisionResult` 必须与 decision record 上已落地的结果一致
- `resumeToStatus` 只能是 `Ready` / `In Progress` / `Cancelled`
- 若 `resumeToStatus = In Progress`，必须提供明确 `resumeToRole`

```json
{
  "decisionResume": {
    "decisionRecordId": "decision_123",
    "decisionResult": "cut",
    "resumeToStatus": "In Progress",
    "resumeToRole": "dev_agent"
  }
}
```

---

### 4. `reviewStart` payload

适用 action：

- `start_review`

#### required fields

- `reviewType`
- `checklistSummary`

#### optional fields

- `scopeNotes`

#### rules

- `reviewType` 取值：`qa_regression`, `tech_review`, `acceptance_review`
- `checklistSummary` 不可为空字符串
- 执行前必须保证当前 owner 已是审核侧

```json
{
  "reviewStart": {
    "reviewType": "qa_regression",
    "checklistSummary": "开始按核心用例与回归清单执行验证",
    "scopeNotes": "重点看移动端筛选器和分页"
  }
}
```

---

### 5. `reviewReject` payload

适用 action：

- `reject_to_rework`

#### required fields

- `reviewType`
- `checklistSummary`
- `issuesFound`
- `nextAction`
- `returnToRole`

#### rules

- `issuesFound` 不可为空数组
- `returnToRole` 通常为 `dev_agent` 或 `tech_lead_agent`

```json
{
  "reviewReject": {
    "reviewType": "qa_regression",
    "checklistSummary": "核心用例 2 条失败",
    "issuesFound": [
      "筛选条件切换后列表未刷新"
    ],
    "nextAction": "return to dev",
    "returnToRole": "dev_agent"
  }
}
```

---

### 6. `reviewPass` payload

适用 action：

- `mark_ready_for_delivery`

#### required fields

- `reviewType`
- `checklistSummary`

#### optional fields

- `issuesFound`

#### rules

- 允许 `issuesFound` 为空数组
- 该 payload 会生成 `ReviewRecord(result=passed)`

```json
{
  "reviewPass": {
    "reviewType": "qa_regression",
    "checklistSummary": "核心回归通过，满足交付前检查",
    "issuesFound": []
  }
}
```

---

### 7. `deliverySummary` payload

适用 action：

- `mark_ready_for_delivery`

#### required fields

- `changeSummary`
- `affectedScope`
- `validationSummary`
- `remainingRisks`

#### rules

- `affectedScope` 不可为空数组
- `remainingRisks` 不能省略；若无显著剩余风险，应显式写 `none` 或等价口径

```json
{
  "deliverySummary": {
    "changeSummary": "完成筛选功能与分页修复",
    "affectedScope": ["user-list", "mobile-filter-panel"],
    "validationSummary": "feature 自测 + QA 回归通过",
    "remainingRisks": "none"
  }
}
```

---

### 8. `reopen` payload

适用 action：

- `reopen_from_delivery`

#### required fields

- `reason`
- `expectedNextOwnerRole`

#### rules

- `reason` 不可为空字符串
- `expectedNextOwnerRole` 不可为空

```json
{
  "reopen": {
    "reason": "交付前发现关键缺陷",
    "expectedNextOwnerRole": "dev_agent"
  }
}
```

---

### 9. `cancel` payload

适用 action：

- `cancel_task`

#### required fields

- 无（Phase 1 允许只给 comment）

#### recommended fields

- `reason`

#### rules

- 若提供 `cancel.reason`，不可为空字符串
- high danger action 强烈建议填写原因

```json
{
  "cancel": {
    "reason": "需求被明确取消"
  }
}
```

---

## guard 与 validator 规则

### 1. transition validator

后端必须先校验：

- 当前状态是否允许该 action
- actor 是否有权限执行该 action
- 必填 payload 是否齐全
- 前置记录是否满足要求
- 当前任务是否处于可流转状态

### 2. guard checker

后端必须检查：

- 是否存在未决 decision
- 是否缺少 review 结论
- 是否缺少 handoff 记录
- 是否未满足 `Ready for Delivery` 门槛
- 是否试图跳过非法中间状态
- `start_review` 时当前 owner 是否已是审核侧
- `resume_after_decision` 时 decision 是否已经 resolve 且与当前任务匹配

### 3. side effects

transition 成功后，后端应自动：

- 创建对应 record
- 写 `ExecutionLog`
- 更新 task 当前状态
- 更新 `currentOwner` / `nextOwner`
- 刷新 `availableActions`
- 返回 `nextSuggestedActions`

### 4. 并发与幂等最小规则

Phase 1 最少要做到：

- transition 请求必须支持 `taskVersion` 或等价版本号校验
- 当客户端基于旧版本提交 transition 时，后端必须拒绝，并返回 `TRANSITION_VERSION_CONFLICT`
- 同一个任务的 transition 失败时返回明确冲突错误，不允许静默覆盖
- 对 high danger 动作，至少要支持服务端去重或 `idempotencyKey` 二选一；若支持 `idempotencyKey`，重复请求不得重复落记录

---

## 错误码建议

### 业务错误码

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

### 返回结构建议

```json
{
  "success": false,
  "error": {
    "code": "TRANSITION_OWNER_MISMATCH",
    "message": "当前角色不是任务当前责任人，不能执行该动作",
    "details": {
      "currentOwnerRole": "qa_review_agent",
      "actorRole": "dev_agent"
    }
  }
}
```

---

## 前端动作暴露建议

前端不直接展示“状态下拉框”，而展示：

- 可执行动作按钮
- 禁用按钮及原因
- 动作说明弹层
- 必填 payload 表单

### 任务详情页默认先展示

- 当前状态
- 当前负责人
- 下一位接手人（若存在）
- 卡点原因（若有）
- 下一步建议动作
- 最近一条 handoff / decision / review / delivery summary 摘要

---

## 关键验收 case（Phase 1）

### 1. 不能通过 PATCH 直接改状态

- 试图 PATCH `status=Done`
- 后端必须拒绝

### 2. 不能在 reviewer 未接手时直接开始 review

- 当前 `currentOwner = dev_agent`
- actor 为 `qa_review_agent`
- 直接 `start_review`
- 后端必须拒绝，并返回 owner / handoff 相关错误

### 3. 交接链路成立

- `submit_handoff` 成功后进入 `Waiting Handoff`
- `accept_handoff` 成功后回到 `In Progress`
- owner 从开发侧切到审核侧

### 4. 审核开始会留 review started 记录

- `start_review` 成功后进入 `In Review`
- 创建 `ReviewRecord(review_status=started, result=pending)`

### 5. 审核驳回会明确返工责任

- `reject_to_rework` 成功后进入 `Rework Required`
- 创建 `ReviewRecord(result=rejected)`
- owner 切回返工角色

### 6. 审核通过进入待交付

- `mark_ready_for_delivery` 必须同时创建：
  - `ReviewRecord(result=passed)`
  - `DeliverySummaryRecord`

### 7. decision resolve 不会自动恢复任务

- `approve/reject decision record` 完成后
- task 仍停留在 `Waiting Decision`

### 8. 必须显式 resume

- `resume_after_decision` 成功后
- task 才能离开 `Waiting Decision`

---

## 与其他文档的边界

这份文档负责：

- transition action registry
- payload 结构
- owner 变化
- 角色权限矩阵
- side effects
- 错误码
- 关键验收 case

这份文档不重复定义：

- 状态枚举与进入 / 退出条件：见 `product/task-status-guards.md`
- 资源接口总表：见 `product/api-contracts.md`
- 核心对象：见 `product/domain-model.md`

---

## 当前阶段统一结论

Phase 1 的 transition 实现必须坚持 4 件事：

1. **前端发 action，不发裸状态**
2. **owner 变化由 transition 驱动，不由普通 PATCH 驱动**
3. **decision resolve 与 task resume 分开**
4. **review 开始、驳回、通过都要有显式记录**

只要这四件事不破，Codex 就能按一套明确的动作模型推进实现。
