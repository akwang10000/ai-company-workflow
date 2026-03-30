# product/implementation-phases.md

## 目标
把当前项目的实施路线图进一步压成 **Codex / 自动化编码代理可直接施工的 phase 清单**，避免阶段文档停留在“方向说明”，而不能转成工程执行。

这份文档主要回答：
- 当前 phase 要做哪些实体、接口、服务、页面
- 这一轮最小交付面应该到哪
- 需要做哪些验证
- 满足什么条件才能进入下一阶段
- 如何在分阶段施工时坚持“系统内部可以复杂，但用户使用必须简单易懂”

它不替代：
- `product/implementation-roadmap.md` 的整体顺序说明职责
- `product/codex-delivery-rules.md` 的交付格式职责
- `product/api-contracts.md` 的接口总表职责

---

## Phase 使用原则
- 先做主链路，再补增强项
- 每一 phase 只收一个明确闭环，不横跳
- 若某个 phase 仍然过大，应继续拆成子阶段，而不是让 Codex 一轮横跨多层
- 进入下一 phase 的前提，不是“感觉差不多”，而是当前 phase 的退出标准已满足
- 若发现领域模型主对象、API 主契约、状态机口径互相冲突，必须先回到文档收口，不应硬写代码
- 系统内部允许复杂，但每个面向用户的交付面都必须保证简单、可理解、低门槛

---

## Phase 0：文档收口与施工基线期

### 本阶段目标
把 Phase 1 MVP 的项目边界、对象模型、状态机口径、API 主链路、画布规格和 Codex 交付规则统一下来，让后续实现有单一真相来源。

### 需要收口的核心文档
- `README.md`
- `docs-map.md`
- `product/product-implementation.md`
- `product/api-contracts.md`
- `product/domain-model.md`
- `product/implementation-phases.md`
- `product/task-transition-api-and-actions.md`

### 交付物
- 统一后的文档口径
- transition 单一真相来源
- Phase 清单
- API 主契约总表
- 明确的对象模型

### Validation
- 7 个核心文档不再互相冲突
- README 与 product 文档边界清楚
- `api-contracts.md` 与 `task-transition-api-and-actions.md` 口径一致
- `domain-model.md` 能覆盖 transition 所需记录对象

### 退出标准
- Codex 可以基于文档开始正式施工
- 当前 MVP 边界和 phase 顺序明确
- 状态机、对象模型、API 主链路不再双轨并存

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

### Stub / 种子数据
至少要有：
- 1 个软件公司模板
- 5 个默认角色模板
- 3 个 workflow 模板
- 3 个任务样例模板

### Validation
- 用户可看到模板列表
- 用户可看到模板详情
- 模板包含角色、workflow、任务样例的最小结构

### 简单易懂约束
- 模板首页必须直接给出推荐模板，不要求用户先理解模板层级
- 模板详情页先解释“启动后你会得到什么”，再展示更细结构

### 退出标准
- 模板不是空概念，而是可查询、可展示、可解释的实际对象

---

## Phase 1B：模板实例化与 Workspace 创建

### 本阶段目标
把“从模板启动”打通，让用户不从空白开始。

### 后端实体 / 仓储
- `Workspace`
- `RoleAssignment`

### 后端服务
- Workspace Instantiation Service

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
- workspace 首页要先回答“现在系统里有什么、下一步点哪里”

### 退出标准
- 用户不再从空白创建项目
- “模板启动”这条链路已真实可跑

---

## Phase 1C：任务 CRUD 主链路

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

### 简单易懂约束
- 新建任务时应优先展示最小必填字段，不要一上来暴露所有高级字段
- 任务详情页默认先显示标题、目标、状态、当前负责人、验收标准、风险
- 不允许让用户通过“内部状态字段编辑”理解任务推进

### 退出标准
- 任务主数据已可用
- 任务详情页可支撑下一阶段接入 transition 与 record 摘要

---

## Phase 1D：任务详情页可执行骨架

### 本阶段目标
在不接完整状态机的前提下，先把任务详情页收成真正可操作的主入口骨架。

### 后端补强
- `GET /api/tasks/{taskId}` 返回 `availableActions` 占位
- `GET /api/tasks/{taskId}/workflow` 返回 workflow 摘要占位

### 前端页面
- 任务详情页补齐：
  - 当前状态卡片
  - 当前负责人
  - 下一步占位动作区
  - workflow 摘要区
  - 记录时间线占位区

### Validation
- 任务详情页已经具备后续接 transition 的承载位
- 前端不需要重构页面骨架就能进入 Phase 2

### 简单易懂约束
- 用户应先看到“当前该做什么”，再看到更细的结构
- 动作区应使用用户动作语言，不先暴露内部术语

### 退出标准
- 任务详情页已经成为后续任务推进的稳定入口

---

# Phase 2：状态机、记录模型与任务推进

## Phase 2A：状态机后端骨架

### 本阶段目标
先把 action-driven transition 的后端骨架做稳，不急着同时铺前端。

### 后端实体 / 仓储
- `WorkflowInstance`

### 后端服务
- Task Transition Service
- Guard Validation Service
- Workflow Runtime Service

### 必做能力
- action enum
- allowed from / target status 映射
- 基础 guard 入口
- 状态推进主流程

### 必做接口
- `POST /api/tasks/{taskId}/transition`（先打通主骨架）

### Validation
- 合法 action 可推进状态
- 非法 action 被拒绝
- 目标状态由后端决定，而不是前端写入

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

### 后端服务
- Record Creation Service

### 必做能力
- transition 成功后创建对应 record
- 写 `ExecutionLog`
- 更新 current owner / next owner

### Validation
- `submit_handoff` 能生成 `HandoffRecord`
- `request_decision` 能生成 `DecisionRecord`
- `reject_to_rework` 能生成 `ReviewRecord`
- `mark_ready_for_delivery` 能生成 `DeliverySummaryRecord`

### 简单易懂约束
- 系统内部可以生成结构化记录
- 但默认展示时应优先有摘要视图，不要求用户直接面对原始结构

### 退出标准
- 状态推进不再只有状态变化，而是伴随可解释记录

---

## Phase 2C：任务详情动作 UI

### 本阶段目标
把 transition API 接到任务详情页，让用户通过动作按钮推进任务。

### 前端页面 / 交互
- action buttons
- handoff 弹层
- decision 弹层
- review 弹层
- delivery summary 弹层

### 必做接口消费
- `POST /api/tasks/{taskId}/transition`
- `GET /api/tasks/{taskId}`（补齐 available actions / record 摘要）

### Validation
- 用户能通过按钮触发 action
- 表单字段缺失时能看到明确错误
- 提交成功后任务详情刷新 current status / next owner / next suggested actions

### 简单易懂约束
- 前端默认显示动作按钮，不显示裸 status 下拉框
- 按钮文案必须用用户动作语言
- disabled action 必须显示原因
- 默认不要求用户理解 payload / guard 等内部概念

### 退出标准
- 用户已经能通过任务详情页完成主链路推进

---

## Phase 2D：三类任务主链路验收

### 本阶段目标
把 feature / bugfix / hotfix 三类任务的主链路走一遍，验证状态机和记录模型是否真的可用。

### 必测链路
- feature 主链路
- bugfix 主链路
- hotfix 主链路

### 必测异常
- 非法 action
- 缺少 payload
- 缺少 required record
- 错误角色执行 action

### Validation
- 三类任务至少各能从 `Draft` 走到 `Ready for Delivery`
- 非法场景返回明确错误码
- 当前 owner、next owner、next suggested actions 可见

### 简单易懂约束
- 即使内部链路复杂，任务详情页默认仍应只突出当前关键动作和关键摘要
- 不允许因为验证复杂度提高而把 UI 做成调试面板

### 退出标准
- Phase 2 主链路已经可以视为“Codex 可稳定继续施工”的基础层

---

# Phase 3：workflow 画布只读主界面

## Phase 3A：canvas projection DTO 与后端组装

### 本阶段目标
先把 workflow runtime 投影成稳定 DTO，让画布前端不直接拼业务对象。

### 后端服务
- Canvas Projection Service
- Node Detail Assembly Service

### 必做接口
- `GET /api/workflows/{workflowInstanceId}/canvas`
- `GET /api/workflows/{workflowInstanceId}/nodes/{nodeId}`

### DTO / 返回结构重点
- `CanvasNode`
- `CanvasEdge`
- `NodeDetailViewModel`
- latest record summaries
- node available actions

### Validation
- DTO 足够支撑画布页
- 画布前端不需要自己拼核心业务对象

### 退出标准
- 画布数据层稳定

---

## Phase 3B：画布只读页

### 本阶段目标
把中间主画布区域做出来，让用户能看懂流程走到哪。

### 前端页面
- Workflow 画布页（三栏布局中的主区）

### Validation
- 节点正确渲染
- 连线正确渲染
- 当前节点 / 当前路径正确高亮

### 简单易懂约束
- 主画布只展示当前理解流程所需信息
- 不把长文本记录、全量日志、内部参数铺在主画布上

### 退出标准
- 用户可一眼看懂主流程大概走向

---

## Phase 3C：节点详情侧栏与页面跳转

### 本阶段目标
补齐右侧详情栏，让画布不仅能看，还能继续进入任务推进。

### 前端页面
- 节点详情侧栏
- 从画布跳任务详情 / 决策页

### Validation
- 右侧详情能显示 owner / inputs / outputs / risks / skills / latest logs / available actions
- 能从画布进入任务详情或决策页

### 简单易懂约束
- 主画布简洁，复杂信息进入右侧详情
- 节点详情里的动作文案仍用用户动作语言

### 退出标准
- 画布已可支撑演示与日常查看

---

# Phase 4：审批、收口与交付摘要

## Phase 4A：Decision Center

### 本阶段目标
把 Waiting Decision 的读取、展示、审批打通。

### 后端服务
- Decision Center Service

### 必做接口
- `GET /api/workspaces/{workspaceId}/decisions`
- `GET /api/decisions/{decisionId}`
- `POST /api/decisions/{decisionId}/approve`
- `POST /api/decisions/{decisionId}/reject`

### 前端页面
- 决策中心页
- 决策详情页

### Validation
- 用户可在系统内完成拍板
- `Waiting Decision -> resume_after_decision` 链路可验证

### 简单易懂约束
- 用户应直接看到为什么需要拍板、推荐选项是什么、下一步影响是什么
- 不要求用户先理解底层 decision record 结构

### 退出标准
- 决策链路已打通

---

## Phase 4B：Ready for Delivery 与收口摘要

### 本阶段目标
把交付前收口信息补齐，让 `Ready for Delivery` 不再只是一个抽象状态。

### 后端服务
- Delivery Readiness Service
- Review / Regression Summary Service

### 前端页面
- 收口摘要组件
- Ready for Delivery 摘要区

### Validation
- bugfix / hotfix 收口摘要可见
- 交付前风险与回归结论可见
- 任务能从 `In Review` 稳定走到 `Done`

### 简单易懂约束
- 用户默认先看到结论摘要、风险摘要、下一步动作
- 不默认展示所有内部记录原文

### 退出标准
- `Ready for Delivery` 的判断标准可见、可解释、可验证

---

# Phase 5：角色与技能挂载可见化

## Phase 5A：角色详情与技能展示

### 本阶段目标
把角色与默认技能从概念设计推进到页面可见。

### 后端实体 / 服务
- `SkillPackageTemplate`
- Role Skill Binding Service

### 必做接口
- `GET /api/workspaces/{workspaceId}/roles`
- `GET /api/roles/{roleAssignmentId}`

### 前端页面
- 角色与技能页

### Validation
- 角色默认技能包可见
- 用户能理解“这个角色默认能做什么”

### 简单易懂约束
- 默认只展示技能名称、说明、是否启用
- schema / binding / retry 等内部参数不默认展开

### 退出标准
- 角色能力不再只是文档概念

---

## Phase 5B：技能启停与节点技能投影

### 本阶段目标
让技能不只可见，还可在合理范围内被启停，并映射到节点详情。

### 后端服务
- Node Skill Projection Service

### 必做接口
- `PATCH /api/roles/{roleAssignmentId}/skills`

### 前端页面
- 节点详情中的技能区

### Validation
- 角色技能启停能生效
- workflow 节点默认技能可见
- 角色、模板、节点之间技能映射口径一致

### 简单易懂约束
- 启停动作保持简单，不做成复杂配置中心
- 主画布不展开全部技能细节

### 退出标准
- 模板默认能力组合已经真正进入产品层

---

# Phase 6：LLM 辅助创建与手机端增强

## Phase 6A：手机端轻操作增强

### 本阶段目标
先补齐手机端核心轻操作，而不是先做复杂生成。

### 后端接口补强
- dashboard / decision / task summary 轻量接口

### 前端页面 / 能力
- 手机端 workspace 概览
- 手机端 decision / approval 流程
- 手机端任务摘要与当前卡点

### Validation
- 手机端可以完成查看进度、查看卡点、轻审批

### 简单易懂约束
- 手机端优先做“看懂 + 拍板 + 轻推进”
- 不要求手机端承载完整复杂编辑

### 退出标准
- 手机端不再只是只读看板

---

## Phase 6B：LLM 辅助草稿生成

### 本阶段目标
让模板和角色创建更顺手，但不破坏模板可控性。

### 后端服务
- Role Draft Generation Service
- Workflow Draft Generation Service
- Skill Recommendation Service

### 必做接口
- 角色草稿生成接口（占位或最小可用）
- workflow 草稿生成接口（占位或最小可用）

### 前端页面 / 能力
- 角色 / workflow 草稿生成入口

### Validation
- 草稿生成结果可被用户继续编辑与确认
- 新用户创建模板不再完全依赖手工填写

### 简单易懂约束
- LLM 负责起草，不直接替用户拍板
- 复杂参数不默认暴露给普通用户
- 生成入口不应打断已有主链路体验

### 退出标准
- 模板创建效率明显提升
- LLM 辅助创建不会破坏模板可控性

---

## 每阶段统一交付格式
每一 phase 至少输出：
- Changed files
- What changed
- Validation
- Current status
- Next recommended step

如果是代码 phase，建议再补：
- Added endpoints
- Added entities / DTOs / services
- Known gaps

---

## 阶段切换规则
只有同时满足以下条件，才允许进入下一 phase：
- 当前 phase 的核心交付物已完成
- 当前 phase 的主链路已验证
- 当前 phase 的剩余问题已列清
- 没有发现新的主对象 / 主契约冲突
- 用户没有明确要求先调整更高优先级事项

---

## 当前推荐实施顺序
1. Phase 0
2. Phase 1A
3. Phase 1B
4. Phase 1C
5. Phase 1D
6. Phase 2A
7. Phase 2B
8. Phase 2C
9. Phase 2D
10. Phase 3A
11. Phase 3B
12. Phase 3C
13. Phase 4A
14. Phase 4B
15. Phase 5A
16. Phase 5B
17. Phase 6A
18. Phase 6B

不要跳过前面的对象模型和 transition 收口，直接去做画布炫技层。

---

## 一句话总结
这份文档的意义不是把 phase 写得更花，而是让 Codex 始终清楚：

**这一轮该做哪些实体、接口、页面与验证；做到什么程度，才算可以交下一棒；并且任何对外可见交付都必须保持简单易懂。**
