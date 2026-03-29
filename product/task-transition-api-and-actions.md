# product/task-transition-api-and-actions.md

## 目标
把 `product/task-status-guards.md` 进一步压成可直接施工的接口与交互规则，确保 Codex、后端、前端对“任务状态推进”使用同一套动作模型，而不是各自理解一套状态切换逻辑。

这份文档主要回答：
- 为什么不能直接让前端修改 `status`
- `transition` API 应该怎么设计
- 哪些动作必须带附加记录
- 前端应该暴露哪些动作按钮与弹层
- 后端应如何实现 guard、validator、record side effects

---

## 核心原则

### 1. 不直接开放 status 任意写入
前端不应直接提交：
- `status = In Progress`
- `status = Done`
- `status = Ready for Delivery`

而应提交“动作”：
- start progress
- submit handoff
- request decision
- start review
- reject to rework
- mark ready for delivery
- complete delivery

原因：
- 动作比状态更可解释
- 动作更容易校验必填记录
- 动作更容易沉淀审计日志
- 动作更适合前端做按钮和表单

---

## 动作模型总览
第一阶段建议统一采用 `action` 驱动的 transition 模型。

### 推荐 action 列表
- `submit_ready`
- `start_progress`
- `submit_handoff`
- `accept_handoff`
- `request_decision`
- `resume_after_decision`
- `start_review`
- `reject_to_rework`
- `restart_rework`
- `mark_ready_for_delivery`
- `reopen_from_delivery`
- `complete_delivery`
- `archive_task`
- `cancel_task`

说明：
- action 是接口输入语义
- target status 是后端根据 action 和 guard 计算出来的结果
- 第一阶段不建议让前端显式传 `fromStatus -> toStatus` 作为唯一真相

---

## transition API

### 推荐接口
`POST /tasks/{id}/transition`

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
    "handoff_record_789"
  ],
  "nextSuggestedActions": [
    "accept_handoff",
    "request_decision"
  ],
  "updatedAt": "2026-03-29T17:40:00+08:00"
}
```

---

## action 到状态的映射

| action | expected from | target status | 是否要求附加记录 |
|---|---|---|---|
| submit_ready | Draft | Ready | 否 |
| start_progress | Ready | In Progress | 否 |
| submit_handoff | In Progress | Waiting Handoff | 是，handoff |
| accept_handoff | Waiting Handoff | In Progress | 否 |
| request_decision | Ready / In Progress / Waiting Handoff / In Review / Rework Required / Ready for Delivery | Waiting Decision | 是，decision |
| resume_after_decision | Waiting Decision | Ready 或 In Progress 或 Cancelled | 是，decision result |
| start_review | In Progress | In Review | 是，review start（建议） |
| reject_to_rework | In Review | Rework Required | 是，review |
| restart_rework | Rework Required | In Progress | 否 |
| mark_ready_for_delivery | In Review | Ready for Delivery | 是，delivery summary |
| reopen_from_delivery | Ready for Delivery | In Progress | 建议带 reopening reason |
| complete_delivery | Ready for Delivery | Done | 否，但要求前置记录已齐 |
| archive_task | Done | Archived | 否 |
| cancel_task | Draft / Ready / In Progress / Waiting Handoff / Waiting Decision | Cancelled | 建议带 cancel reason |

---

## 附加记录要求

### 1. handoff payload 必填动作
`submit_handoff`

最低建议结构：
```json
{
  "handoff": {
    "fromRole": "dev_agent",
    "toRole": "qa_review_agent",
    "handoffSummary": "string",
    "deliveredArtifacts": ["string"],
    "risksOrNotes": "string"
  }
}
```

### 2. decision payload 必填动作
- `request_decision`
- `resume_after_decision`

最低建议结构：
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

### 3. review payload 必填动作
- `reject_to_rework`
- `start_review`（建议带）

最低建议结构：
```json
{
  "review": {
    "reviewType": "qa_regression",
    "result": "rejected",
    "checklistSummary": "核心用例 2 条失败",
    "issuesFound": ["筛选条件切换后列表未刷新"],
    "nextAction": "return to dev"
  }
}
```

### 4. deliverySummary payload 必填动作
`mark_ready_for_delivery`

最低建议结构：
```json
{
  "deliverySummary": {
    "changeSummary": "完成筛选功能与分页修复",
    "affectedScope": ["user-list", "mobile-filter-panel"],
    "validationSummary": "feature 自测 + QA 回归通过",
    "remainingRisks": "移动端样式建议再人工巡检一次"
  }
}
```

---

## 后端 guard 规则

### 1. transition validator
后端必须先校验：
- 当前状态是否允许该 action
- actor 是否有权限执行该 action
- 必填 payload 是否齐全
- 前置记录是否满足要求

### 2. guard checker
后端必须检查：
- 是否存在未决 decision
- 是否缺少 review 结论
- 是否缺少 handoff 记录
- 是否未满足 Ready for Delivery 门槛
- 是否试图跳过非法中间状态

### 3. side effects
transition 成功后，后端应自动：
- 创建对应 record
- 写 execution log / audit log
- 更新 task 当前状态
- 更新 current owner / next owner（如适用）
- 返回 next suggested actions

### 4. 错误码建议
- `TASK_TRANSITION_NOT_ALLOWED`
- `TASK_GUARD_CHECK_FAILED`
- `TASK_REQUIRED_PAYLOAD_MISSING`
- `TASK_REQUIRED_RECORD_MISSING`
- `TASK_DECISION_NOT_RESOLVED`
- `TASK_REVIEW_NOT_PASSED`
- `TASK_INVALID_ACTION_FOR_ROLE`

---

## 前端动作模型
前端不要提供裸 `status dropdown`。

### 推荐做法
- 在任务详情页提供当前可执行动作按钮
- 点击动作后弹出对应表单
- 提交成功后刷新 timeline、status、records、next actions

### 不同状态建议动作

#### Draft
- 补齐并提交为 Ready
- 取消任务

#### Ready
- 开始执行
- 请求决策
- 取消任务

#### In Progress
- 提交交接
- 请求决策
- 开始审核（如果当前阶段已完成）
- 取消任务（有限开放）

#### Waiting Handoff
- 接收交接
- 请求决策

#### Waiting Decision
- 按决策结果恢复
- 取消任务

#### In Review
- 驳回返工
- 标记可交付
- 请求决策

#### Rework Required
- 开始返工
- 请求决策

#### Ready for Delivery
- 完成交付
- 重新打开
- 请求决策

#### Done
- 归档

---

## 页面与组件建议

### 任务详情页
至少包含：
- 当前状态卡片
- 当前 owner
- next suggested actions
- handoff / decision / review / delivery summary 时间线
- action buttons

### 动作弹层
建议按动作拆表单：
- 提交交接弹层
- 请求决策弹层
- 审核驳回弹层
- 标记可交付弹层
- 重新打开弹层

### 画布页
节点详情建议展示：
- 当前状态
- 当前可执行动作
- 最近一条 decision / handoff / review 摘要

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

## 与其他文档的关系
- `product/task-status-guards.md`：定义状态机和守卫原则
- `product/api-contracts.md`：定义整体接口风格与返回结构
- `product/screens-and-flows.md`：定义页面流转与端侧场景
- `product/canvas-ui-spec.md`：定义画布与节点详情展示方式

这份文档解决的是：
**如何把状态守卫规则翻译成可实现的 API、按钮动作和后端状态机。**

---

## Codex 实施建议
建议按以下顺序施工：
1. 定义 action 枚举与 action -> target status 映射
2. 实现 `POST /tasks/{id}/transition`
3. 为 handoff / decision / review / delivery summary 建持久化结构
4. 在前端将状态切换改为动作按钮 + 弹层提交
5. 将 next suggested actions 投影到任务详情页和画布节点详情

---

## 一句话总结
`task-transition-api-and-actions.md` 的作用，就是把“任务状态机”从抽象规则，压成 **前端能做按钮、后端能做校验、Codex 能直接施工的动作驱动实现规范**。
