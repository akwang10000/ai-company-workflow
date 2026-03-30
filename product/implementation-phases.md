# product/implementation-phases.md

## 目标

把当前项目的实施路线图进一步压成 Codex / 自动化编码代理可直接施工的 phase 清单，避免阶段文档停留在“方向说明”，而不能转成工程执行。

这份文档主要回答：

- 当前 phase 要做哪些实体、接口、服务、页面
- 这一轮最小交付面应该到哪
- 需要做哪些验证
- 满足什么条件才能进入下一阶段
- 如何在分阶段施工时坚持“系统内部可以复杂，但用户使用必须简单易懂”
- 哪些文档当前已经冻结，施工时不应擅自改主真相

它不替代：

- `product/codex-delivery-rules.md` 的交付格式职责
- `product/api-contracts.md` 的接口总表职责
- `product/task-transition-api-and-actions.md` 的 transition 单一真相来源职责

---

## Phase 使用原则

- 先做主链路，再补增强项
- 每一 phase 只收一个明确闭环，不横跳
- 若某个 phase 仍然过大，应继续拆成子阶段，而不是让 Codex 一轮横跨多层
- 进入下一 phase 的前提，不是“感觉差不多”，而是当前 phase 的退出标准已满足
- 若发现领域模型主对象、API 主契约、状态机口径互相冲突，必须先回到文档收口，不应硬写代码
- 系统内部允许复杂，但每个面向用户的交付面都必须保证简单、可理解、低门槛
- Phase 0 之后，不再新增新的“总纲型”主文档

---

## 当前冻结边界

### 冻结为主、只允许补歧义

- `product/domain-model.md`
- `product/task-status-guards.md`
- `product/task-transition-api-and-actions.md`
- `product/api-contracts.md`
- `product/implementation-phases.md`
- `product/minimum-software-company-template.md`

### 只允许跟随性小修

- `README.md`
- `docs-map.md`
- `product/product-implementation.md`
- `product/screens-and-flows.md`
- `examples/*`

### 施工期发现问题时的处理顺序

1. 先判断是不是文档歧义
2. 若是歧义，只允许补一处真相，不允许再开新总纲
3. 若不是歧义，而是范围过大，优先缩小当前 phase 交付面
4. 不允许靠代码先实现一套“临时逻辑”绕过主文档

---

## Phase 0：文档收口与施工基线期

### 本阶段目标

把 Phase 1 MVP 的项目边界、对象模型、状态机口径、API 主链路、画布规格和 Codex 交付规则统一下来，让后续实现有单一真相来源。

### 本阶段必做事项

- 明确 Phase 1 只做“软件公司模板 / 研发团队版”
- 明确 `TaskInstance` 是任务业务状态主真相
- 明确 `PATCH /tasks/{id}` 不能直接改 `status/currentOwner/nextOwner`
- 明确 `governance/task-schema.md` 只服务 Phase 1 研发团队版，不再保留多行业漂移口
- 明确 handoff 与 review 的默认顺序
- 明确 `ReviewRecord` 采用 append-only 生命周期
- 明确 decision approval 与 task resume 的边界
- 明确 actor 身份来源与最小版本冲突 / 幂等规则
- 冻结主骨架文档

### Validation

- 文档之间不再双轨并存
- README、domain model、status guards、transition、API 契约、task schema 口径一致
- Codex 能基于文档开始正式施工

### 退出标准

- 当前 MVP 边界明确
- 状态机、对象模型、API 主链路不再互相打架
- 不再需要继续扩大文档面

---

# Phase 1：模板启动与任务主骨架

## Phase 1A：模板查询与种子数据

### 本阶段目标

先把“默认模板存在且可被读取”做出来，让系统有明确起点。

### 后端实体 / 仓储

- `CompanyTemplate`
- `RoleTemplate`
- `WorkflowTemplate`
- `TaskTemplate`

### 后端服务

- Template Query Service
- Seed Data Loader

### 必做接口

- `GET /api/templates`
- `GET /api/templates/{companyTemplateId}`

### 前端页面

- 模板首页（只读）
- 模板详情页（只读）

### Validation

- 模板列表可读
- 模板详情可读
- 默认软件公司模板的角色、workflow、任务类型能展示出来

### 简单易懂约束

- 第一次看到系统时，不要求用户先理解内部 workflow schema
- 模板页默认回答“这是什么模板、会带出什么、我点哪里开始”

### 退出标准

- 系统已有真实模板起点，不再从空白想象启动

---

## Phase 1B：模板实例化与 workspace 骨架

### 本阶段目标

把“从模板创建 workspace”做成可跑闭环。

### 后端实体 / 仓储

- `Workspace`
- `RoleAssignment`

### 后端服务

- Workspace Instantiate Service

### 必做接口

- `POST /api/templates/{companyTemplateId}/instantiate`
- `GET /api/workspaces/{workspaceId}`
- `GET /api/workspaces/{workspaceId}/dashboard`

### 前端页面

- 模板启动弹窗
- workspace 首页骨架

### Validation

- 用户能从模板创建 workspace
- 能自动得到默认角色实例
- workspace 首页能展示基础概览

### 简单易懂约束

- 启动时只要求最少输入，不要求用户一开始配置所有角色和技能
- 启动成功后默认进入 workspace 首页，不把用户直接丢到复杂配置页
- workspace 首页先回答“现在系统里有什么、下一步点哪里”

### 退出标准

- “模板启动”这条链路已真实可跑

---

## Phase 1C：任务 CRUD 主链路（非流转）

### 本阶段目标

把任务创建、查看、更新打通，为后续状态机和 workflow 运行准备骨架。

### 后端实体 / 仓储

- `TaskInstance`

### 后端服务

- Task CRUD Service

### 必做接口

- `GET /api/workspaces/{workspaceId}/tasks`
- `POST /api/workspaces/{workspaceId}/tasks`
- `GET /api/tasks/{taskId}`
- `PATCH /api/tasks/{taskId}`（仅非流转字段）

### 前端页面

- 任务列表页
- 新建任务页
- 任务详情页骨架

### Validation

- 能创建任务
- 能查看任务列表
- 能查看任务详情
- 能更新非流转字段
- 不能通过 PATCH 直接修改 `status/currentOwner/nextOwner`

### 简单易懂约束

- 新建任务页默认只收最小必要字段
- 任务详情页默认展示当前状态、当前负责人、下一步建议动作
- 不要求普通用户先理解 record schema

### 退出标准

- 任务主资源已稳定
- 后续 transition 可以挂在这条主资源链路上

---

# Phase 2：transition 核心引擎

## Phase 2A：action-driven transition 骨架

### 本阶段目标

先把任务状态推进的主引擎做出来，禁止“手工改 status”成为主路径。

### 必做能力

- action enum
- `allowed from -> target status` 映射
- 基础 guard 入口
- transition 主流程
- owner 变化 side effect
- 版本冲突 / 幂等基础处理

### 必做接口

- `POST /api/tasks/{taskId}/transition`

### Validation

- 合法 action 可推进状态
- 非法 action 被拒绝
- 目标状态由后端决定，而不是前端写入
- owner 变化由 transition side effect 驱动

### 简单易懂约束

- 即便内部状态机实现复杂，对前端暴露的仍应是“动作 -> 结果”模型
- 不允许把内部状态切换细节原样甩给用户操作

### 退出标准

- transition 后端骨架已可用
- 状态推进逻辑不再依赖手工改 status

---

## Phase 2B：记录模型与 transition side effects

### 本阶段目标

把 transition 所需记录模型补齐，并让状态推进真正生成对应留痕。

### 后端实体 / 仓储

- `DecisionRecord`
- `HandoffRecord`
- `ReviewRecord`
- `DeliverySummaryRecord`
- `ExecutionLog`

### 必做能力

- transition 成功后自动创建对应记录
- 最近记录摘要可回填到 task detail
- handoff / review / delivery summary / decision 的 guard 可被校验

### Validation

- `submit_handoff` 创建 `HandoffRecord`
- `start_review` 创建 `ReviewRecord(started/pending)`
- `reject_to_rework` 创建 `ReviewRecord(rejected)`
- `mark_ready_for_delivery` 创建 `ReviewRecord(passed)` + `DeliverySummaryRecord`
- 所有 transition 写 `ExecutionLog`

### 退出标准

- task transition 不再只是改一个 status 字段
- 审计链路与关键记录模型可用

---

## Phase 2C：交接 / 审核 / 决策闭环验证

### 本阶段目标

打通最关键的 3 条业务闭环。

### 必测闭环

#### 闭环 1：交接 + 审核

`submit_handoff -> accept_handoff -> start_review`

#### 闭环 2：审核驳回 + 返工

`start_review -> reject_to_rework -> restart_rework`

#### 闭环 3：拍板

`request_decision -> approve/reject decision record -> resume_after_decision`

### Validation

- reviewer 未接手前不能直接正式开始 review
- decision resolve 后任务不会自动离开 `Waiting Decision`
- 必须显式 `resume_after_decision`
- Ready for Delivery 前必须有 passed review + delivery summary

### 退出标准

- handoff / review / decision 三套主规则已真实可跑
- 最危险的歧义点已经通过实现被验证

---

# Phase 3：workflow 与画布查看

## Phase 3A：workflow 实例与画布投影

### 本阶段目标

让任务不只是“列表项”，而能映射到 workflow 视图。

### 后端实体 / 仓储

- `WorkflowInstance`
- `CanvasNode`
- `CanvasEdge`

### 必做接口

- `GET /api/tasks/{taskId}/workflow`
- `GET /api/workflows/{workflowInstanceId}/canvas`
- `GET /api/workflows/{workflowInstanceId}/nodes/{nodeId}`

### Validation

- Task 能关联到 workflow instance
- 画布能展示当前节点与高亮路径
- 节点详情能显示最近记录摘要与 available actions

### 简单易懂约束

- 画布第一页先强调“当前卡在哪里、谁在负责、下一步做什么”
- 不把 workflow 内部 schema 原样暴露给普通用户

### 退出标准

- workflow 不再只是文档概念，而有真实运行投影

---

## Phase 3B：任务详情动作面与 dashboard 收口

### 本阶段目标

把用户日常最常用的操作面做出来。

### 必做页面

- workspace dashboard
- task detail action panel
- waiting decisions list
- waiting handoffs summary

### Validation

- 用户能在 dashboard 看见卡点
- 用户能在 task detail 完成主动作
- 禁用动作能给出明确原因

### 退出标准

- 用户不需要理解内部 schema 也能完成主链路操作

---

# Phase 4：角色配置与最小技能系统

## Phase 4A：角色实例与技能包最小配置

### 本阶段目标

补齐角色查看与最小技能启用能力。

### 必做接口

- `GET /api/workspaces/{workspaceId}/roles`
- `GET /api/roles/{roleAssignmentId}`
- `PATCH /api/roles/{roleAssignmentId}/skills`

### Validation

- 角色实例列表可见
- 单角色详情可见
- 技能包开关可保存

### 退出标准

- 软件公司模板的“角色协作”不再只是概念名词，而有最小配置能力

---

# Phase 5：实现验证期与文档反向修补

## Phase 5A：以实现结果回打文档

### 本阶段目标

不是继续扩主文档，而是把实现中暴露的歧义收回单一真相来源。

### 允许改动

- 明确歧义
- 修正不一致
- 收缩过大的交付面

### 不允许改动

- 新增新的总纲文档
- 在未验证闭环前扩张 Phase 1 边界
- 把本该由 transition 管的逻辑再散回普通接口

### Validation

- 实现反馈的问题能在主真相文档中找到唯一归口
- 不再出现“README 这样写、API 那样写、实现又是第三套”的情况

### 退出标准

- Phase 1 MVP 达到“可跑、可演示、可继续迭代”的状态

---

## 当前阶段统一结论

当前最优动作不是继续写更大的 Phase 说明，  
而是：

1. 冻结主骨架
2. 先做模板启动闭环
3. 再做任务主资源 + transition 主链路
4. 用 handoff / review / decision 三条闭环反向验证文档

只要这条顺序不乱，Codex 就能按步骤施工，而不是一边写一边猜。
