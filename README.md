# AI Company Workflow

## 项目一句话

AI Company Workflow 要做的，不是一个普通任务管理工具，而是一个：

**可模板化启动、可角色协作、可按阶段落地执行的 AI 软件公司工作流系统。**

---

## 当前阶段一句话

当前项目只聚焦 **Phase 1 MVP**，先验证“**第一个软件公司模板 / 软件研发团队版**”是否能跑通。  
当前阶段不是要一口气做成通用 AI 公司操作系统。

---

## Phase 1 的真实范围

第一阶段只做一个最小模板：

## 第一个软件公司模板

当前优先要做通的是：

- 模板启动
- 角色分工
- 任务单管理
- action-driven 状态推进
- handoff / decision / review / delivery summary 等记录
- workflow 画布查看与轻操作
- API 与前端动作模型对齐
- 文档是否足够支撑 Codex / 自动化编码代理按 phase 施工

第一阶段优先覆盖的任务类型：

- `feature`
- `bugfix`
- `hotfix`

第一阶段不追求：

- 通用模板市场
- 多行业一次性覆盖
- 高复杂权限体系
- 大规模自动化编排引擎
- 深度 BI / 报表系统
- 企业级多租户全量治理
- 高自由度高级流程编辑器

---

## 当前要验证的 4 个核心命题

### A. 模板化启动是否成立

能不能用一个“软件公司模板”快速拉起新项目，并自动带出：

- 基础角色
- 基础流程
- 基础任务类型
- 基础状态机
- 基础页面与操作骨架

### B. 任务推进是否能被结构化约束

能不能通过：

- guard / 守卫规则
- action-driven transition
- 必填记录
- 审批 / 评审 / 决策节点

保证推进过程是：

- 可控
- 可审计
- 可回溯

### C. AI 角色协作是否能被产品化

能不能把产品、技术、开发、审核、交付等角色拆清，并支持：

- 任务接力
- 信息沉淀
- 交接记录
- 审核与返工
- 收口与交付

### D. 文档是否足够支持自动施工

能不能让 Codex / 自动化编码代理不需要大量猜系统长什么样，也能按 phase 开始前后端实现。

---

## 当前最重要的产品口径

## 不是直接改 status，而是 action-driven task transition

前端不应暴露一个任意可改状态的下拉框，  
而应暴露明确动作，例如：

- 开始执行
- 提交交接
- 开始审核
- 请求拍板
- 驳回返工
- 标记可交付
- 完成交付

后端根据：

- 当前状态
- 守卫规则
- payload 完整性
- 必填记录
- 角色权限
- 决策与前置记录状态

来决定：

- 能不能执行
- 转到哪个状态
- 生成哪些记录
- owner 如何变化
- 返回什么下一步建议动作

如果这条口径不成立，这个项目就会退化成“带点 AI 的任务看板”，而不是目标中的 AI 软件公司工作流系统。

---

## 当前阶段判断

项目已经从“抽象方法论”推进到“**Codex 可以开始施工的文档化产品设计阶段**”。

当前阶段不再适合继续扩大文档面。  
主骨架进入 **冻结前后的最终一致性修补阶段**：

- 主模型不再大改
- 主 API 不再双轨并存
- 状态机不再反复加分支
- 后续以小修、边界澄清、实现验证为主

---

## 当前冻结策略

### 现在应冻结为主、只允许补歧义的文档

- `product/domain-model.md`
- `product/task-status-guards.md`
- `product/task-transition-api-and-actions.md`
- `product/api-contracts.md`
- `product/implementation-phases.md`
- `product/minimum-software-company-template.md`

### 现在只允许跟随性小修的文档

- `README.md`
- `docs-map.md`
- `product/product-implementation.md`
- `product/screens-and-flows.md`
- `examples/*`

### 当前阶段不允许的动作

- 新开一份新的“总纲型”主文档
- 再扩一轮新的 Phase 1 概念面
- 在未验证闭环前继续扩权限、多租户、市场、复杂编排

---

## 最短阅读入口

### 第一次看项目

1. `README.md`
2. `AGENTS.md`
3. `docs-map.md`
4. `product/product-implementation.md`
5. `product/implementation-phases.md`

### 准备开始实现

1. `AGENTS.md`
2. `product/domain-model.md`
3. `product/task-status-guards.md`
4. `product/task-transition-api-and-actions.md`
5. `product/api-contracts.md`
6. `product/implementation-phases.md`

---

## 文档职责分工

- `README.md`：项目目标、当前范围、核心命题、阶段判断
- `docs-map.md`：整套文档导航与阅读路径
- `product/product-implementation.md`：第一阶段产品实施约束
- `product/domain-model.md`：核心领域对象与 source of truth
- `product/task-status-guards.md`：状态机、进入/退出条件、前置记录要求
- `product/task-transition-api-and-actions.md`：transition 的动作真相、payload、owner 变化、错误码
- `product/api-contracts.md`：资源接口总表与请求/响应契约
- `product/implementation-phases.md`：分阶段施工与退出标准

---

## 当前最优下一步

当前最重要的不是继续扩大概念，而是围绕 Phase 1 MVP 先把以下闭环做稳：

1. 模板启动闭环  
   `GET templates -> instantiate -> workspace dashboard`

2. 任务主链路闭环  
   `create task -> submit_ready -> start_progress`

3. 交接 / 审核 / 收口闭环  
   `submit_handoff -> accept_handoff -> start_review -> reject_to_rework / mark_ready_for_delivery -> complete_delivery`

4. 决策闭环  
   `request_decision -> approve/reject decision record -> resume_after_decision`

---

## 当前阶段结论

当前项目已经具备进入“**冻结主骨架 + 实现验证**”的条件。

判断标准不是“文档看起来很多”，  
而是：

- 软件公司模板可启动
- 任务状态机可控推进
- 角色协作可记录、可交接、可审计
- 画布与页面能支撑查看和轻操作
- 文档足够支持 Codex 分阶段施工

只要这套方法先在软件研发团队场景跑顺，后续才有资格复制到其他行业模板。
