# product/api-contracts.md

## 目标

定义 AI Company Workflow 在 Phase 1 MVP 的主链路 API 契约，确保：

- 后端知道需要提供哪些资源接口
- 前端知道能拿到什么结构化数据
- Codex 在实现时不会前后端各写各的
- 模板启动、任务流转、画布展示、审批拍板这些主链路能够对齐

这份文档负责：

- 主链路 REST 资源与路径
- 请求 / 响应结构约定
- 错误码分层
- 各资源接口的最小字段要求
- 哪些字段允许普通更新，哪些必须走 transition

这份文档不重复定义：

- 任务状态机原则：见 `product/task-status-guards.md`
- task transition 动作真相：见 `product/task-transition-api-and-actions.md`

---

## API 统一原则

- 资源接口归资源接口
- task transition 归 action-driven transition
- 不允许旧的 `fromStatus -> toStatus` 范式继续作为主契约存在
- `PATCH /api/tasks/{taskId}` 只允许更新**非流转字段**
- `currentOwner` / `nextOwner` / `status` 的业务变化必须走 transition
- decision 审批与 task 恢复分开：审批接口只解决 decision record，不直接推进 task

---

## 通用响应建议

### 成功响应

```json
{
  "success": true,
  "data": {}
}
```

### 失败响应

```json
{
  "success": false,
  "error": {
    "code": "TRANSITION_ACTION_NOT_ALLOWED",
    "message": "当前状态不允许执行该动作",
    "details": {}
  }
}
```

---

## 1. Template APIs

### `GET /api/templates`

#### 目标

获取可启动模板列表。

#### 返回重点

- `companyTemplateId`
- `name`
- `summary`
- `includedRoleCount`
- `includedWorkflowCount`
- `recommendedStartFlow`
- `status`

---

### `GET /api/templates/{companyTemplateId}`

#### 目标

获取单个模板详情。

#### 返回重点

- template 基本信息
- 默认角色
- 默认 workflow
- 默认任务类型
- 推荐启动流程

---

### `POST /api/templates/{companyTemplateId}/instantiate`

#### 目标

基于模板创建 workspace。

#### 请求体最小建议

```json
{
  "workspaceName": "后台研发协作空间"
}
```

#### 返回重点

- `workspaceId`
- `workspaceName`
- `companyTemplateId`
- `defaultRoles`
- `dashboardEntry`

---

## 2. Workspace APIs

### `GET /api/workspaces/{workspaceId}`

#### 目标

获取 workspace 基本信息。

#### 返回重点

- `workspaceId`
- `name`
- `status`
- `companyTemplate`
- `members`
- `defaultRoles`

---

### `GET /api/workspaces/{workspaceId}/dashboard`

#### 目标

获取工作空间首页聚合数据。

#### 返回重点

- task summary
- waiting decisions
- waiting handoffs
- ready for delivery
- recent activity
- recommended next entry

---

## 3. Task APIs

### `GET /api/workspaces/{workspaceId}/tasks`

#### 目标

获取任务列表。

#### 返回重点

- `taskId`
- `title`
- `taskType`
- `status`
- `priority`
- `currentOwner`
- `nextOwner`
- `availableActions`
- `updatedAt`

---

### `POST /api/workspaces/{workspaceId}/tasks`

#### 目标

创建任务。

#### 请求体最小建议

```json
{
  "title": "新增后台封禁记录查询页",
  "taskType": "feature",
  "goal": "支持运营按用户和时间筛选封禁记录",
  "acceptanceCriteria": [
    "支持按用户筛选",
    "支持分页",
    "支持时间范围筛选"
  ]
}
```

#### 返回重点

- `taskId`
- `status`
- `plannedOwner`
- `currentOwner`
- `availableActions`

---

### `GET /api/tasks/{taskId}`

#### 目标

获取任务详情。

#### 返回重点

- `taskId`
- `title`
- `goal`
- `taskType`
- `status`
- `plannedOwner`
- `currentOwner`
- `nextOwner`
- `acceptanceCriteria`
- `definitionOfDone`
- `knownRisks`
- `availableActions`
- `latestRecordSummaries`

#### `data` 示例

```json
{
  "taskId": "task_001",
  "title": "新增后台封禁记录查询页",
  "taskType": "feature",
  "status": "In Progress",
  "plannedOwner": {
    "roleKey": "dev_agent",
    "displayName": "Dev Agent"
  },
  "currentOwner": {
    "roleKey": "dev_agent",
    "displayName": "Dev Agent"
  },
  "nextOwner": {
    "roleKey": "qa_review_agent",
    "displayName": "QA / Review Agent"
  },
  "availableActions": [
    "submit_handoff",
    "request_decision"
  ]
}
```

---

### `PATCH /api/tasks/{taskId}`

#### 目标

更新任务字段。

#### 适用范围

- 补字段
- 更新风险
- 更新背景
- 更新验收标准
- 更新截止时间
- 更新 `plannedOwner`
- 更新非流转型业务字段

#### 明确禁止

- 不允许通过本接口直接修改业务状态
- 不允许通过本接口直接修改 `currentOwner`
- 不允许通过本接口直接修改 `nextOwner`
- 不允许用本接口替代 transition API

#### 请求体建议

```json
{
  "title": "新增后台封禁记录查询页",
  "knownRisks": [
    "移动端筛选器样式需要人工巡检"
  ],
  "plannedOwnerRoleAssignmentId": "role_assign_dev_001"
}
```

---

### `POST /api/tasks/{taskId}/transition`

#### 目标

按 action 推进任务状态。

#### 说明

- 该接口采用 action-driven 模型
- 前端不传 `fromStatus -> toStatus` 作为唯一真相
- target status 由后端根据 action、当前状态、guard、payload、角色权限推导
- 具体 action 约束、payload 结构、side effects 以 `product/task-transition-api-and-actions.md` 为准

#### 请求体建议

```json
{
  "action": "submit_handoff",
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

#### 返回体建议

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
  "updatedAt": "2026-03-30T10:00:00+09:00"
}
```

---

## 4. Workflow APIs

### `GET /api/tasks/{taskId}/workflow`

#### 目标

获取任务对应的 workflow 实例摘要。

#### 返回重点

- workflow instance
- current node
- current status projection
- current owner projection
- highlighted path summary

---

### `GET /api/workflows/{workflowInstanceId}/canvas`

#### 目标

获取画布数据。

#### 返回重点

- nodes
- edges
- current highlighted path
- default viewport

---

### `GET /api/workflows/{workflowInstanceId}/nodes/{nodeId}`

#### 目标

获取节点详情面板数据。

#### 返回重点

- title
- summary
- owner
- inputs
- outputs
- enabled skills
- risks
- latest logs
- available actions
- latest decision / handoff / review summary

---

## 5. Decision APIs

### `GET /api/workspaces/{workspaceId}/decisions`

#### 目标

获取待审批列表。

#### 返回重点

- `decisionRecordId`
- `taskId`
- `decisionTopic`
- `decisionQuestion`
- `recommendedOption`
- `status`
- `requestedBy`
- `createdAt`

---

### `GET /api/decisions/{decisionRecordId}`

#### 目标

获取单个决策详情。

#### 返回重点

- decision 基本信息
- options
- risk summary
- current status
- related task summary

---

### `POST /api/decisions/{decisionRecordId}/approve`

### `POST /api/decisions/{decisionRecordId}/reject`

#### 目标

完成 decision record 的审批动作。

#### 重要说明

- 这两个接口只负责更新 `DecisionRecord.status`
- 它们**不会**直接修改任务状态
- task 仍停留在 `Waiting Decision`
- 恢复 task 必须通过 `POST /api/tasks/{taskId}/transition` + `resume_after_decision`

#### 请求体建议

```json
{
  "comment": "同意砍掉移动端高级筛选，优先保主路径"
}
```

#### 返回重点

- `decisionRecordId`
- `status`
- `decisionResult`
- `resolvedBy`
- `resolvedAt`
- `relatedTaskId`

---

## 6. Role & Skill APIs

### `GET /api/workspaces/{workspaceId}/roles`

#### 目标

获取当前 workspace 的角色实例列表。

### `GET /api/roles/{roleAssignmentId}`

#### 目标

获取单个角色详情。

### `PATCH /api/roles/{roleAssignmentId}/skills`

#### 目标

更新角色启用的技能包。

#### 说明

- 这属于角色配置，不属于任务 transition
- 不应借该接口隐式修改任务状态或任务 owner

---

## 7. Dashboard APIs

### `GET /api/workspaces/{workspaceId}/dashboard`

#### 目标

获取工作空间首页聚合数据。

#### 返回重点

- 全局任务数
- 各状态任务数
- waiting decision 数
- waiting handoff 数
- ready for delivery 数
- 最近活跃任务
- 推荐入口动作

---

## 错误码分层建议

### 通用层

- `VALIDATION_ERROR`
- `NOT_FOUND`
- `FORBIDDEN`
- `VERSION_CONFLICT`

### task transition 层

- `TRANSITION_ACTION_NOT_ALLOWED`
- `TRANSITION_ROLE_NOT_ALLOWED`
- `TRANSITION_PAYLOAD_INVALID`
- `TRANSITION_GUARD_FAILED`
- `TRANSITION_OWNER_MISMATCH`
- `TRANSITION_DECISION_NOT_RESOLVED`
- `TRANSITION_DECISION_RESULT_MISMATCH`

### decision 审批层

- `DECISION_ALREADY_RESOLVED`
- `DECISION_NOT_WAITING_APPROVAL`
- `DECISION_FORBIDDEN`

---

## API 与文档的关系

- `product/api-contracts.md`：定义资源接口总表与错误码场景映射
- `product/task-status-guards.md`：定义状态机与 guard 原则
- `product/task-transition-api-and-actions.md`：定义 transition 的动作真相、payload、side effects、权限矩阵、关键验收 case
- `product/screens-and-flows.md`：定义页面如何消费这些接口
- `product/canvas-ui-spec.md`：定义画布页与节点详情如何消费 workflow API

---

## 当前阶段统一结论

Phase 1 的接口口径必须坚持：

- 资源接口归资源接口
- task transition 归 action-driven transition
- 不允许旧的 `fromStatus -> toStatus` 范式继续作为主契约存在
- 不允许 `PATCH /tasks/{id}` 偷改 `status` 或 `currentOwner`
- decision 审批与 task 恢复必须分开

只要这几条不再双轨并存，Codex 就能按一套明确 API 契约往前施工。
