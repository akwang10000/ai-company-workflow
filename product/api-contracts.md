# product/api-contracts.md

## 目标
定义 AI Company Workflow 在 **Phase 1 MVP** 的主链路 API 契约，确保：
- 后端知道需要提供哪些资源接口
- 前端知道能拿到什么结构化数据
- Codex 在实现时不会前后端各写各的
- 模板启动、任务流转、画布展示、审批拍板这些主链路能够对齐

这份文档负责：
- 主链路 REST 资源与路径
- 请求 / 响应结构约定
- 错误码分层
- 各资源接口的最小字段要求

这份文档不重复定义：
- 任务状态机原则：见 `product/task-status-guards.md`
- task transition 动作真相：见 `product/task-transition-api-and-actions.md`

---

## 设计原则
- 先保主链路，再补高级能力
- 先定义资源、路径、返回结构，再谈优化
- 前端优先拿到“能支撑页面”的稳定结构
- task transition 必须走 **action-driven** 模型，不允许前端任意写业务状态
- 一个接口只解决一类资源问题，避免混杂过多职责

---

## API 分组
Phase 1 建议按以下资源分组：
- Template APIs
- Workspace APIs
- Task APIs
- Workflow APIs
- Decision APIs
- Role & Skill APIs
- Dashboard APIs

---

## 通用约定

### Base Path
统一采用：
- `/api/...`

### 成功返回
```json
{
  "success": true,
  "data": {},
  "message": "ok"
}
```

### 失败返回
```json
{
  "success": false,
  "error": {
    "code": "TASK_REQUIRED_PAYLOAD_MISSING",
    "message": "handoff payload is required",
    "details": {
      "field": "payload.handoff"
    }
  }
}
```

### 通用错误码分层
#### 通用校验类
- `VALIDATION_FAILED`
- `RESOURCE_NOT_FOUND`
- `UNAUTHORIZED`
- `FORBIDDEN`
- `CONFLICT`

#### 任务流转类
- `TASK_TRANSITION_NOT_ALLOWED`
- `TASK_GUARD_CHECK_FAILED`
- `TASK_REQUIRED_PAYLOAD_MISSING`
- `TASK_REQUIRED_RECORD_MISSING`
- `TASK_DECISION_NOT_RESOLVED`
- `TASK_REVIEW_NOT_PASSED`
- `TASK_INVALID_ACTION_FOR_ROLE`

#### 模板 / 实例化类
- `TEMPLATE_NOT_AVAILABLE`
- `WORKSPACE_INSTANTIATION_FAILED`

---

## 错误码场景映射（Phase 1 最小集）

| 场景 | 推荐错误码 | 前端提示方向 |
|---|---|---|
| 请求体缺少最小必填字段 | `VALIDATION_FAILED` | 提示补齐字段 |
| 资源不存在 | `RESOURCE_NOT_FOUND` | 提示对象已不存在或链接失效 |
| 未登录 / 身份失效 | `UNAUTHORIZED` | 提示重新登录 |
| 当前用户无权访问该资源 | `FORBIDDEN` | 提示无访问权限 |
| 资源状态冲突（非 transition 语义） | `CONFLICT` | 提示当前数据已变化，请刷新 |
| 当前状态不允许执行该 action | `TASK_TRANSITION_NOT_ALLOWED` | 提示当前阶段不能这样推进 |
| guard 未通过 | `TASK_GUARD_CHECK_FAILED` | 提示还有前置条件未满足 |
| 缺少 action 所需 payload | `TASK_REQUIRED_PAYLOAD_MISSING` | 提示补齐表单字段 |
| 缺少 action 所需记录 | `TASK_REQUIRED_RECORD_MISSING` | 提示先完成上一条必要记录 |
| decision 尚未解决 | `TASK_DECISION_NOT_RESOLVED` | 提示先完成拍板 |
| review 未通过 | `TASK_REVIEW_NOT_PASSED` | 提示先完成返工或通过审核 |
| 当前角色无权执行该 action | `TASK_INVALID_ACTION_FOR_ROLE` | 提示当前角色不能执行该动作 |
| 模板不可用 | `TEMPLATE_NOT_AVAILABLE` | 提示模板暂不可启动 |
| workspace 实例化失败 | `WORKSPACE_INSTANTIATION_FAILED` | 提示启动失败并建议重试 |

---

## 资源概览

| 资源 | 作用 |
|---|---|
| Template | 查询与实例化软件公司模板 |
| Workspace | 查看工作空间总览与 dashboard |
| Task | 任务 CRUD、详情、transition、记录摘要 |
| Workflow | workflow instance 摘要、画布、节点详情 |
| Decision | 待拍板列表、决策详情、审批动作 |
| Role & Skill | 角色实例、技能包展示与启停 |

---

## 1. Template APIs

### `GET /api/templates`
#### 目标
获取模板列表。

#### 返回重点
- 软件公司模板卡片
- 模板摘要
- 是否推荐

#### `data` 示例
```json
{
  "items": [
    {
      "companyTemplateId": "company_tpl_sw_001",
      "name": "第一个软件公司模板",
      "summary": "研发团队版最小启动模板",
      "recommended": true,
      "version": "v1",
      "status": "active"
    }
  ]
}
```

---

### `GET /api/templates/{companyTemplateId}`
#### 目标
获取模板详情。

#### 返回重点
- 包含的角色模板
- workflow 模板
- 任务样例
- 默认技能包
- 推荐启动路径

---

### `POST /api/templates/{companyTemplateId}/instantiate`
#### 目标
从模板创建 workspace。

#### 请求体建议
```json
{
  "workspaceName": "AI 公司实验室",
  "ownerUserId": "user_001"
}
```

#### 返回重点
- 新建 workspace id
- 默认角色实例摘要
- 默认 workflow 骨架摘要
- 首页跳转信息

#### `data` 示例
```json
{
  "workspaceId": "ws_001",
  "workspaceName": "AI 公司实验室",
  "roleAssignments": [
    {
      "roleAssignmentId": "ra_pm_001",
      "roleKey": "pm_agent",
      "displayName": "PM Agent"
    }
  ],
  "workflowSkeletons": [
    {
      "workflowTemplateId": "wf_feature_v1",
      "name": "feature workflow"
    }
  ],
  "redirect": {
    "path": "/workspaces/ws_001"
  }
}
```

---

## 2. Workspace APIs

### `GET /api/workspaces/{workspaceId}`
#### 目标
获取工作空间概览。

#### 返回重点
- 当前任务数
- Waiting Decision 数
- Rework 数
- 最近动态
- 推荐下一步

---

### `GET /api/workspaces/{workspaceId}/dashboard`
#### 目标
获取工作空间首页卡片数据。

#### 返回重点
- summary cards
- pending decisions
- active tasks
- recent logs
- quick actions

---

## 3. Task APIs

### `GET /api/workspaces/{workspaceId}/tasks`
#### 目标
获取任务列表。

#### 支持筛选
- `taskType`
- `status`
- `owner`
- `keyword`

---

### `POST /api/workspaces/{workspaceId}/tasks`
#### 目标
创建任务。

#### 请求体最小字段
```json
{
  "title": "新增后台封禁记录查询页",
  "taskType": "feature",
  "scenario": "engineering",
  "goal": "让运营自助查询封禁记录",
  "priority": "medium",
  "acceptanceCriteria": [
    "支持按用户 ID 查询",
    "无权限用户不可访问"
  ]
}
```

#### 返回重点
- task 基础信息
- 初始状态
- 可执行动作

---

### `GET /api/tasks/{taskId}`
#### 目标
获取任务详情。

#### 返回重点
- 基础字段
- acceptance criteria
- known risks
- current owner
- next owner
- workflow 摘要
- handoff / decision / review / delivery summary 摘要
- current available actions

#### `data` 示例
```json
{
  "taskId": "task_001",
  "title": "新增后台封禁记录查询页",
  "taskType": "feature",
  "status": "In Progress",
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
    "request_decision",
    "start_review"
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
- 更新 owner
- 更新截止时间
- 更新非流转型业务字段

#### 明确禁止
- 不允许通过本接口直接修改业务状态
- 不允许用本接口替代 transition API

---

### `POST /api/tasks/{taskId}/transition`
#### 目标
按 action 推进任务状态。

#### 说明
- 该接口采用 **action-driven** 模型
- 前端不传 `fromStatus -> toStatus` 作为唯一真相
- target status 由后端根据 action、当前状态、guard、payload、角色权限推导
- 具体 action 约束、payload 结构、side effects 以 `product/task-transition-api-and-actions.md` 为准

#### 请求体建议
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
  "updatedAt": "2026-03-29T17:40:00+08:00"
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
- current status
- current owner
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

### `GET /api/decisions/{decisionId}`
#### 目标
获取单个决策详情。

### `POST /api/decisions/{decisionId}/approve`
### `POST /api/decisions/{decisionId}/reject`
#### 目标
完成审批动作。

---

## 6. Role & Skill APIs

### `GET /api/workspaces/{workspaceId}/roles`
### `GET /api/roles/{roleAssignmentId}`
### `PATCH /api/roles/{roleAssignmentId}/skills`

---

## 7. Dashboard APIs

### `GET /api/workspaces/{workspaceId}/dashboard`
#### 目标
获取工作空间首页聚合数据。

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

---

## 一句话总结
这份文档的作用，是把 Phase 1 的主链路 API 收成一套：

**前后端都能对齐、Codex 能按它开工、错误处理有清晰归属、且与 action-driven 状态机一致的 REST 契约总表。**
