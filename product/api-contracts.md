# product/api-contracts.md

## 目标
定义 AI Company Workflow 第一阶段的前后端 API 契约，确保：
- 后端知道要提供哪些接口
- 前端知道能拿到什么数据
- Codex 在实现时不至于前后端各写各的
- 画布、任务、审批、模板启动这些主链路能对齐

---

## 设计原则
- 先保主链路，再补高级能力
- 先定义资源与返回结构，再谈优化
- 前端优先拿到“能支撑页面”的稳定结构
- 第一阶段不追求接口面面俱到，先保证模板启动、任务流转、画布展示、审批拍板可跑通

---

## API 分组
建议第一阶段按以下资源分组：
- Template APIs
- Workspace APIs
- Task APIs
- Workflow APIs
- Decision APIs
- Role & Skill APIs
- Dashboard APIs

---

## 通用返回约定

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
    "code": "TASK_VALIDATION_FAILED",
    "message": "acceptance_criteria is required"
  }
}
```

---

## 1. Template APIs

### `GET /api/templates`
#### 目标
获取模板列表。

#### 返回重点
- 软件公司模板卡片
- 模板摘要
- 是否推荐

#### data 示例
```json
{
  "items": [
    {
      "companyTemplateId": "company_tpl_sw_001",
      "name": "第一个软件公司模板",
      "summary": "研发团队版最小启动模板",
      "recommended": true,
      "version": "v1"
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
- 默认创建的角色实例摘要
- 默认 workflow 摘要
- 首页跳转信息

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

---

### `GET /api/workspaces/{workspaceId}/dashboard`
#### 目标
获取工作空间首页卡片数据。

#### 返回重点
- summary cards
- pending decisions
- active tasks
- recent logs

---

## 3. Task APIs

### `GET /api/workspaces/{workspaceId}/tasks`
#### 目标
获取任务列表。

#### 支持筛选
- taskType
- status
- owner
- keyword

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
- handoff / review / rework 摘要

---

### `PATCH /api/tasks/{taskId}`
#### 目标
更新任务字段。

#### 适用
- 补字段
- 更新风险
- 更新 owner
- 更新截止时间

---

### `POST /api/tasks/{taskId}/transition`
#### 目标
推进任务状态。

#### 请求体建议
```json
{
  "fromStatus": "In Progress",
  "toStatus": "Review",
  "reason": "开发完成并已自测"
}
```

#### 说明
后端必须做状态守卫校验，不能盲切。

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

---

### `GET /api/workflows/{workflowInstanceId}/canvas`
#### 目标
获取画布数据。

#### 返回重点
- nodes
- edges
- current highlighted path
- default viewport

#### data 示例
```json
{
  "nodes": [
    {
      "nodeId": "node_pm",
      "nodeType": "role",
      "title": "PM Agent",
      "status": "done",
      "ownerLabel": "PM Agent"
    }
  ],
  "edges": [
    {
      "edgeId": "edge_pm_tl",
      "sourceNodeId": "node_pm",
      "targetNodeId": "node_tl",
      "edgeType": "normal"
    }
  ],
  "currentNodeId": "node_dev"
}
```

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

---

## 5. Decision APIs

### `GET /api/workspaces/{workspaceId}/decisions`
#### 目标
获取待审批列表。

#### 返回重点
- task title
- decision type
- reason
- recommended option
- approver
- created at

---

### `GET /api/decisions/{decisionId}`
#### 目标
获取单个决策详情。

#### 返回重点
- 当前结论
- 可选方案
- 风险差异
- 推荐意见

---

### `POST /api/decisions/{decisionId}/approve`
### `POST /api/decisions/{decisionId}/reject`
#### 目标
完成审批动作。

#### 请求体建议
```json
{
  "comment": "同意按方案 A 推进"
}
```

---

## 6. Role & Skill APIs

### `GET /api/workspaces/{workspaceId}/roles`
#### 目标
获取角色实例与角色模板信息。

---

### `GET /api/roles/{roleAssignmentId}`
#### 目标
获取角色详情。

#### 返回重点
- 角色职责
- 非职责
- 默认技能包
- 当前启用技能

---

### `PATCH /api/roles/{roleAssignmentId}/skills`
#### 目标
启用 / 禁用角色默认技能包。

#### 请求体建议
```json
{
  "enabledSkillPackageIds": [
    "skill_pkg_pm_basic",
    "skill_pkg_task_breakdown"
  ]
}
```

---

## 7. Dashboard APIs

### `GET /api/workspaces/{workspaceId}/overview`
#### 目标
给首页和手机端提供统一概览。

#### 返回重点
- active task count
- waiting decision count
- hot tasks
- hotfix summary
- current blockers

---

## 画布接口与页面的关系
- 模板首页 / 模板详情页：依赖 Template APIs
- 工作空间首页：依赖 Workspace / Dashboard APIs
- 任务列表 / 任务详情页：依赖 Task APIs
- Workflow 画布页：依赖 Workflow APIs
- 决策中心页：依赖 Decision APIs
- 角色与技能页：依赖 Role & Skill APIs

---

## 第一阶段暂不强求的 API
- 复杂模板版本 diff
- 多人协作冲突解决
- 实时协同编辑画布
- 高级权限矩阵接口
- 深度 BI 统计接口

---

## Codex 实施建议
如果 Codex 开始按 API 契约实现，建议顺序：
1. Template APIs
2. Workspace APIs
3. Task APIs
4. Workflow APIs
5. Decision APIs
6. Role & Skill APIs
7. Dashboard APIs

---

## 一句话总结
API 契约文档的意义不是把接口写满，而是先把：
**模板启动、任务流转、画布展示、审批拍板** 这几条主链路的前后端接口收死。
