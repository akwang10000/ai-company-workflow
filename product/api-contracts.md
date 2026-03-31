# product/api-contracts.md

## 目标

定义 Phase 1 对外暴露的最小 API 契约分层，明确哪些是模板接口、哪些是 workspace 接口、哪些是 task 资源接口、哪些是 transition / decision 接口，以及普通 PATCH 与 transition 的边界。

---

## 设计原则

- 资源接口负责 CRUD 与查询
- transition 接口负责状态推进与 owner 变化
- decision resolve 与 task resume 分开
- workflow 视图接口服务展示，不反向写业务主状态

---

## 1. Template APIs

### `GET /api/templates`

获取可启动模板列表。

### `GET /api/templates/{companyTemplateId}`

获取模板详情、默认角色、默认 workflow、默认任务类型。

### `POST /api/templates/{companyTemplateId}/instantiate`

基于模板创建 workspace。

#### 请求体建议

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

获取 workspace 基本信息。

### `GET /api/workspaces/{workspaceId}/dashboard`

获取首页聚合数据。

#### 返回重点

- task summary
- waiting decisions
- waiting handoffs
- ready for delivery
- recent activity
- recommended next entry

---

## 3. Task Resource APIs

### `POST /api/workspaces/{workspaceId}/tasks`

创建任务。

### `GET /api/tasks/{taskId}`

获取任务详情。

### `GET /api/tasks/{taskId}/records`

获取任务相关 records。

### `PATCH /api/tasks/{taskId}`

更新 **非流转字段**。

#### Phase 1 明确允许

- `title`
- `goal`
- `acceptanceCriteria`
- `definitionOfDone`
- `priority`
- `plannedOwner`
- 其他非流转说明字段

#### Phase 1 明确禁止

- 直接改 `status`
- 直接改 `currentOwner`
- 直接改 `nextOwner`
- 直接伪造审核通过 / 决策完成 / 交付完成

---

## 4. Task Transition API

### `POST /api/tasks/{taskId}/transition`

所有正式流转动作都通过这个接口进入。

#### 请求体最小建议

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
      "deliveredArtifacts": ["PR #128", "self-test summary"],
      "risksOrNotes": "移动端筛选器样式仍建议人工看一眼"
    }
  }
}
```

#### 关键返回

- `previousStatus`
- `currentStatus`
- `acceptedAction`
- `createdRecords`
- `currentOwner`
- `nextOwner`
- `nextSuggestedActions`

---

## 5. Decision APIs

### `GET /api/tasks/{taskId}/decisions`

获取当前任务所有 decision records。

### `POST /api/decisions/{decisionRecordId}/approve`

批准某个 decision。

### `POST /api/decisions/{decisionRecordId}/reject`

拒绝某个 decision。

### 关键边界

- approve / reject 只更新 `DecisionRecord`
- 不直接让 task 离开 `Waiting Decision`
- task 恢复必须通过 `resume_after_decision`

---

## 6. Workflow / View APIs

### `GET /api/tasks/{taskId}/workflow-view`

获取任务在 workflow / 画布上的投影。

### `GET /api/tasks/{taskId}/available-actions`

获取当前可执行动作、禁用原因和下一建议动作。

### 关键边界

- 这些接口服务前端展示与轻操作
- 不反向定义 task 主状态

---

## 当前阶段统一结论

Phase 1 的 API 契约必须坚持：

1. 普通 PATCH 不负责业务流转
2. 所有正式推进都走 `POST /api/tasks/{taskId}/transition`
3. decision resolve 与 task resume 分开
4. workflow 视图接口只做投影，不做主真相
