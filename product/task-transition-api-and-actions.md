# product/task-transition-api-and-actions.md

## 目标
把 `product/task-status-guards.md` 进一步压成 **可直接施工的 transition 实现规范**，确保 Codex、后端、前端对“任务状态推进”使用同一套动作模型，而不是各自理解一套状态切换逻辑。

这份文档主要回答：
- 为什么不能让前端直接修改 `status`
- `POST /api/tasks/{taskId}/transition` 应该如何设计
- 每个 action 的允许来源状态、必填 payload、生成记录、目标状态是什么
- 哪些角色允许执行哪些动作
- 哪些动作需要二次确认、哪些属于危险动作
- 前端应暴露哪些动作按钮、弹层和禁用原因
- 后端应如何实现 guard、validator、side effects、错误码
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

---

## transition API

### 接口
`POST /api/tasks/{taskId}/transition`

### 请求体建议
```json
{
  "action": "submit_handoff",
  "actorUserId": "user_123",
  "actorRoleKey": "dev_agent",
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
  "updatedAt": "2026-03-29T17:40:00+08:00"
}
```

---

## action registry（Phase 1）

| action | allowed from | target status | 必填 payload | 生成记录 | 典型执行角色 | requires confirmation | danger level |
|---|---|---|---|---|---|---|---|
| `submit_ready` | `Draft` | `Ready` | 无 | `ExecutionLog` | PM / Tech Lead | 否 | low |
| `start_progress` | `Ready` | `In Progress` | 无 | `ExecutionLog` | 当前接手角色 | 否 | low |
| `submit_handoff` | `In Progress` | `Waiting Handoff` | `handoff` | `HandoffRecord` + `ExecutionLog` | 当前执行角色 | 是 | medium |
| `accept_handoff` | `Waiting Handoff` | `In Progress` | 无 | `ExecutionLog` | 下一接手角色 | 否 | low |
| `request_decision` | `Ready` / `In Progress` / `Waiting Handoff` / `In Review` / `Rework Required` / `Ready for Delivery` | `Waiting Decision` | `decision` | `DecisionRecord` + `ExecutionLog` | 当前责任角色 | 是 | medium |
| `resume_after_decision` | `Waiting Decision` | `Ready` / `In Progress` / `Cancelled` | `decision` | `ExecutionLog` | approver 之后的接手角色 | 是 | medium |
| `start_review` | `In Progress` | `In Review` | 建议 `review` | `ReviewRecord`（可选）+ `ExecutionLog` | QA / Review / Tech Lead | 否 | low |
| `reject_to_rework` | `In Review` | `Rework Required` | `review` | `ReviewRecord` + `ExecutionLog` | QA / Review / Tech Lead | 是 | medium |
| `restart_rework` | `Rework Required` | `In Progress` | 无 | `ExecutionLog` | Dev / 当前返工角色 | 否 | low |
| `mark_ready_for_delivery` | `In Review` | `Ready for Delivery` | `deliverySummary` | `DeliverySummaryRecord` + `ExecutionLog` | QA / Review / Ops / Tech Lead | 是 | medium |
| `reopen_from_delivery` | `Ready for Delivery` | `In Progress` | 建议 `reopen` | `ExecutionLog` | approver / 当前责任角色 | 是 | high |
| `complete_delivery` | `Ready for Delivery` | `Done` | 无 | `ExecutionLog` | Ops / approver | 是 | high |
| `archive_task` | `Done` | `Archived` | 无 | `ExecutionLog` | Ops / system | 是 | low |
| `cancel_task` | `Draft` / `Ready` / `In Progress` / `Waiting Handoff` / `Waiting Decision` | `Cancelled` | 建议 `cancel` | `ExecutionLog` | PM / approver | 是 | high |

---

## 角色权限矩阵（Phase 1）

> 说明：这里给的是默认权限边界。若后续需要更细的权限系统，应在实现层扩展，但 Phase 1 不应突破这张表的主边界。

| action | allowed roles | denied roles | 说明 |
|---|---|---|---|
| `submit_ready` | `pm_agent`, `tech_lead_agent` | `dev_agent`, `qa_review_agent`, `ops_release_agent` | 任务从 Draft 进入可执行态前，应由上游职责角色完成 |
| `start_progress` | `pm_agent`, `tech_lead_agent`, `dev_agent`, `qa_review_agent`, `ops_release_agent` | 无默认拒绝，但需与当前接手角色一致 | 必须是当前接手角色开始执行 |
| `submit_handoff` | `pm_agent`, `tech_lead_agent`, `dev_agent`, `qa_review_agent`, `ops_release_agent` | 无默认拒绝，但需与当前 owner 一致 | 必须由当前执行角色提交 |
| `accept_handoff` | `pm_agent`, `tech_lead_agent`, `dev_agent`, `qa_review_agent`, `ops_release_agent` | 无默认拒绝，但需与 `handoff.toRole` 一致 | 必须由下一接手角色接收 |
| `request_decision` | `pm_agent`, `tech_lead_agent`, `dev_agent`, `qa_review_agent`, `ops_release_agent` | 无 | 当前责任角色可请求拍板 |
| `resume_after_decision` | `pm_agent`, `tech_lead_agent`, `ops_release_agent` | `dev_agent`, `qa_review_agent` | 默认由拍板后负责重新组织推进的角色执行 |
| `start_review` | `tech_lead_agent`, `qa_review_agent` | `dev_agent`, `ops_release_agent` | 默认不由 Dev 自己把自己送入正式 Review |
| `reject_to_rework` | `tech_lead_agent`, `qa_review_agent` | `dev_agent`, `ops_release_agent` | 返工结论应由审核侧给出 |
| `restart_rework` | `dev_agent`, `tech_lead_agent` | `ops_release_agent` | 默认由返工执行侧重新开始 |
| `mark_ready_for_delivery` | `qa_review_agent`, `ops_release_agent`, `tech_lead_agent` | `dev_agent` | 默认不允许 Dev 直接宣布可交付 |
| `reopen_from_delivery` | `pm_agent`, `tech_lead_agent`, `qa_review_agent`, `ops_release_agent` | `dev_agent` | 重新打开应由上游/审核/交付侧触发 |
| `complete_delivery` | `ops_release_agent`, `pm_agent` | `dev_agent` | 默认由交付/确认侧完成 |
| `archive_task` | `ops_release_agent`, `system` | `dev_agent`, `qa_review_agent` | 归档不是开发动作 |
| `cancel_task` | `pm_agent`, `tech_lead_agent`, `approver` | `dev_agent`, `qa_review_agent`, `ops_release_agent` | 取消任务默认属于上游或拍板职责 |

---

## payload 规则

### 通用规则
- `action`：必填，必须为已注册 action 枚举值
- `actorUserId`：必填，不能为空字符串
- `actorRoleKey`：必填，必须为当前系统合法角色键
- `comment`：可选，但对 medium / high danger action 建议填写
- `payload`：按 action 决定是否必填

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
- `resume_after_decision`

#### `request_decision` required fields
- `decisionTopic`
- `decisionQuestion`
- `options`
- `recommendedOption`
- `riskSummary`

#### `resume_after_decision` required fields
- `decisionTopic`
- `decisionResult`

#### optional fields
- `decisionQuestion`
- `options`
- `recommendedOption`
- `riskSummary`

#### rules
- `options` 对 `request_decision` 不可为空数组
- `decisionResult` 对 `request_decision` 可为空
- `decisionResult` 对 `resume_after_decision` 必填

```json
{
  "decision": {
    "decisionTopic": "scope change",
    "decisionQuestion": "是否砍掉移动端高级筛选",
    "options": ["keep", "cut", "delay"],
    "recommendedOption": "cut",
    "riskSummary": "当前工期不足",
    "decisionResult": "cut"
  }
}
```

---

### 3. `review` payload
适用 action：
- `start_review`（建议）
- `reject_to_rework`（必填）

#### required fields（`reject_to_rework`）
- `reviewType`
- `result`
- `checklistSummary`
- `issuesFound`
- `nextAction`

#### optional fields（`start_review`）
- `reviewType`
- `checklistSummary`

#### enum values
- `reviewType`: `qa_regression`, `tech_review`, `acceptance_review`
- `result`: `passed`, `rejected`

#### rules
- `reject_to_rework` 时 `result` 必须为 `rejected`
- `issuesFound` 对 `reject_to_rework` 不可为空数组

```json
{
  "review": {
    "reviewType": "qa_regression",
    "result": "rejected",
    "checklistSummary": "核心用例 2 条失败",
    "issuesFound": [
      "筛选条件切换后列表未刷新"
    ],
    "nextAction": "return to dev"
  }
}
```

---

### 4. `deliverySummary` payload
适用 action：
- `mark_ready_for_delivery`

#### required fields
- `changeSummary`
- `affectedScope`
- `validationSummary`
- `remainingRisks`

#### optional fields
- 无

#### rules
- `affectedScope` 不可为空数组
- `remainingRisks` 不能省略；若无显著剩余风险，应显式写 `none` 或等价口径

```json
{
  "deliverySummary": {
    "changeSummary": "完成筛选功能与分页修复",
    "affectedScope": [
      "user-list",
      "mobile-filter-panel"
    ],
    "validationSummary": "feature 自测 + QA 回归通过",
    "remainingRisks": "移动端样式建议再人工巡检一次"
  }
}
```

---

### 5. `reopen` payload
适用 action：
- `reopen_from_delivery`

#### required fields
- `reason`

#### optional fields
- `expectedNextOwnerRole`

#### rules
- `reason` 不可为空字符串

```json
{
  "reopen": {
    "reason": "交付前发现关键缺陷",
    "expectedNextOwnerRole": "dev_agent"
  }
}
```

---

### 6. `cancel` payload
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

### 3. side effects
transition 成功后，后端应自动：
- 创建对应 record
- 写 `ExecutionLog`
- 更新 task 当前状态
- 更新 current owner / next owner（如适用）
- 返回 next suggested actions

### 4. 推荐错误码
- `TASK_TRANSITION_NOT_ALLOWED`
- `TASK_GUARD_CHECK_FAILED`
- `TASK_REQUIRED_PAYLOAD_MISSING`
- `TASK_REQUIRED_RECORD_MISSING`
- `TASK_DECISION_NOT_RESOLVED`
- `TASK_REVIEW_NOT_PASSED`
- `TASK_INVALID_ACTION_FOR_ROLE`

---

## 关键 guard 细化

### `submit_handoff`
必须满足：
- 当前状态是 `In Progress`
- 有明确当前 owner
- payload 中存在 `handoff`
- `handoff.fromRole` 与当前 actor 角色一致

### `request_decision`
必须满足：
- 当前状态允许进入 `Waiting Decision`
- payload 中存在 `decision`
- 有明确 decision topic / question / options

### `resume_after_decision`
必须满足：
- 当前状态是 `Waiting Decision`
- 最新 `DecisionRecord` 已有明确结果
- payload 中存在 `decision.decisionResult`

### `reject_to_rework`
必须满足：
- 当前状态是 `In Review`
- payload 中存在 `review`
- `review.result = rejected`

### `mark_ready_for_delivery`
必须满足：
- 当前状态是 `In Review`
- 已有审核通过前提
- payload 中存在 `deliverySummary`
- 适用的回归 / 审核条件已满足

### `complete_delivery`
必须满足：
- 当前状态是 `Ready for Delivery`
- 必要记录已齐全
- 没有未解决的关键 guard failure

---

## 前端动作模型
前端不要提供裸 `status dropdown`。

### 推荐做法
- 在任务详情页提供当前可执行动作按钮
- 点击动作后弹出对应表单
- 若 action 不可用，应展示 disabled reason
- 对 requires confirmation = `是` 的动作，必须有确认层
- 对 danger level = `high` 的动作，默认使用危险动作样式
- 提交成功后刷新 timeline、status、records、next actions

---

## 不同状态下建议暴露的动作

### `Draft`
- `submit_ready`
- `cancel_task`

### `Ready`
- `start_progress`
- `request_decision`
- `cancel_task`

### `In Progress`
- `submit_handoff`
- `request_decision`
- `start_review`
- `cancel_task`（有限开放）

### `Waiting Handoff`
- `accept_handoff`
- `request_decision`

### `Waiting Decision`
- `resume_after_decision`
- `cancel_task`

### `In Review`
- `reject_to_rework`
- `mark_ready_for_delivery`
- `request_decision`

### `Rework Required`
- `restart_rework`
- `request_decision`

### `Ready for Delivery`
- `complete_delivery`
- `reopen_from_delivery`
- `request_decision`

### `Done`
- `archive_task`

---

## 动作按钮与弹层建议

### 任务详情页至少包含
- 当前状态卡片
- 当前 owner
- next suggested actions
- handoff / decision / review / delivery summary 时间线
- action buttons

### 动作弹层建议按 action 拆分
- 提交交接弹层
- 请求决策弹层
- 恢复执行弹层
- 审核驳回弹层
- 标记可交付弹层
- 重新打开弹层
- 取消任务弹层

### 禁用态必须说明原因
例如：
- 缺少必填 payload
- 当前角色无权限
- 上一条 decision 未解决
- 尚未满足交付前置条件

---

## UI 动作文案建议
| action | 默认按钮文案 |
|---|---|
| `submit_ready` | 提交为可执行 |
| `start_progress` | 开始执行 |
| `submit_handoff` | 提交交接 |
| `accept_handoff` | 接收交接 |
| `request_decision` | 请求拍板 |
| `resume_after_decision` | 按拍板结果继续 |
| `start_review` | 开始审核 |
| `reject_to_rework` | 驳回返工 |
| `restart_rework` | 开始返工 |
| `mark_ready_for_delivery` | 标记可交付 |
| `reopen_from_delivery` | 重新打开任务 |
| `complete_delivery` | 完成交付 |
| `archive_task` | 归档任务 |
| `cancel_task` | 取消任务 |

---

## feature / bugfix / hotfix 的动作差异

### feature
更强调：
- `request_decision` 用于范围裁剪
- `mark_ready_for_delivery` 前必须有清晰验收摘要

### bugfix
更强调：
- `reject_to_rework` 时写清复现与回归失败点
- `mark_ready_for_delivery` 前写清回归范围

### hotfix
更强调：
- `request_decision` 用于是否直接上线 / 是否回滚
- `mark_ready_for_delivery` 前必须显式写剩余风险

---

## 关键验收 case（Phase 1 最小集）

### Case 1：合法 handoff
- given: task status = `In Progress`，actor = `dev_agent`
- when: action = `submit_handoff`，payload.handoff 完整
- then: status -> `Waiting Handoff`
- and: 创建 `HandoffRecord`
- and: 返回 `nextSuggestedActions`

### Case 2：非法跨状态完工
- given: task status = `In Review`
- when: action = `complete_delivery`
- then: 返回 `TASK_TRANSITION_NOT_ALLOWED`

### Case 3：缺少 handoff payload
- given: task status = `In Progress`
- when: action = `submit_handoff`，payload 缺失
- then: 返回 `TASK_REQUIRED_PAYLOAD_MISSING`

### Case 4：错误角色驳回返工
- given: task status = `In Review`，actor = `dev_agent`
- when: action = `reject_to_rework`
- then: 返回 `TASK_INVALID_ACTION_FOR_ROLE`

### Case 5：request_decision 正常进入等待拍板
- given: task status = `In Progress`
- when: action = `request_decision`，decision payload 完整
- then: status -> `Waiting Decision`
- and: 创建 `DecisionRecord`

### Case 6：resume_after_decision 缺少结果
- given: task status = `Waiting Decision`
- when: action = `resume_after_decision`，decisionResult 缺失
- then: 返回 `TASK_REQUIRED_PAYLOAD_MISSING`

### Case 7：审核驳回返工
- given: task status = `In Review`，actor = `qa_review_agent`
- when: action = `reject_to_rework`，review.result = `rejected`
- then: status -> `Rework Required`
- and: 创建 `ReviewRecord`

### Case 8：标记可交付
- given: task status = `In Review`，回归已通过
- when: action = `mark_ready_for_delivery`，deliverySummary 完整
- then: status -> `Ready for Delivery`
- and: 创建 `DeliverySummaryRecord`

### Case 9：交付前重新打开
- given: task status = `Ready for Delivery`
- when: action = `reopen_from_delivery`，reason 完整
- then: status -> `In Progress`
- and: 返回新的 nextSuggestedActions

### Case 10：取消任务
- given: task status = `Ready`，actor = `pm_agent`
- when: action = `cancel_task`
- then: status -> `Cancelled`

### Case 11：hotfix 请求拍板
- given: task type = `hotfix`，status = `In Progress`
- when: action = `request_decision`
- then: 返回 `Waiting Decision`
- and: 决策中应能说明是否直接上线 / 回滚

### Case 12：Done 后归档
- given: task status = `Done`
- when: action = `archive_task`
- then: status -> `Archived`

---

## 与其他文档的关系
- `product/task-status-guards.md`：定义状态机和守卫原则
- `product/api-contracts.md`：定义整体接口风格与资源接口总表
- `product/domain-model.md`：定义 `DecisionRecord` / `HandoffRecord` / `ReviewRecord` / `DeliverySummaryRecord` / `ExecutionLog` 等对象
- `product/screens-and-flows.md`：定义页面流转与端侧场景
- `product/canvas-ui-spec.md`：定义画布与节点详情展示方式

---

## Codex 实施建议
建议按以下顺序施工：
1. 定义 action 枚举与 action registry
2. 实现 `POST /api/tasks/{taskId}/transition`
3. 为 handoff / decision / review / delivery summary 建持久化结构
4. 在前端把状态切换改为动作按钮 + 弹层提交
5. 将 `nextSuggestedActions` 与 `disabledReason` 投影到任务详情页和画布节点详情
6. 用本文件中的关键验收 case 做最小回归验证

---

## 一句话总结
这份文档的作用，就是把“任务状态机”从抽象规则压成一套：

**前端能做按钮、后端能做校验、权限边界清楚、记录模型能对齐、异常场景能验证、Codex 能直接施工的 action-driven transition 规范。**
