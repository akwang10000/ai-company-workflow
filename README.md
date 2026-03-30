# AI Company Workflow

## 项目一句话
AI Company Workflow 要做的，不是一个普通任务管理工具，而是一个：

**可模板化启动、可角色协作、可按阶段落地执行的 AI 软件公司工作流系统。**

---

## 当前阶段一句话
当前项目 **只聚焦 Phase 1 MVP**，先验证“**软件公司模板 / 软件研发团队版**”能否跑通；
当前不是要一口气做成通用 AI 公司操作系统。

---

## Phase 1 的真实范围
第一阶段只做一个最小模板：

## **第一个软件公司模板**

当前优先要做通的是：
- 模板启动
- 角色分工
- 任务状态推进
- 决策 / 交接 / 评审 / 交付节点记录
- 画布与页面流转
- 后端 API 与前端动作模型
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
- 企业级多租户治理全量能力

---

## 当前要验证的 4 个核心命题

### A. 模板化启动是否成立
能不能用一个“软件公司模板”快速拉起新项目，并自动带出基础角色、流程、任务类型、状态机和页面骨架。

### B. 任务推进是否能被结构化约束
能不能通过 guard、action-driven transition、必填记录、审批 / 评审 / 决策节点，保证推进是可控、可审计、可回溯的。

### C. AI 角色协作是否能被产品化
能不能把产品、开发、测试 / 评审、交付等角色拆清，并支持任务接力、信息沉淀、状态推进和交接。

### D. 文档是否足够支持自动施工
能不能让 Codex / 自动化编码代理不需要大量猜测，也能按 phase 开始前后端实现。

---

## 当前最重要的产品口径

## **不是直接改 status，而是 action-driven task transition**

前端不应暴露一个任意可改状态的下拉框，
而应暴露明确动作，例如：
- 提交交接
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

来决定：
- 能不能执行
- 转到哪个状态
- 需要补什么记录
- 返回什么下一步建议动作

如果这条口径不成立，这个项目就会退化成“带点 AI 的任务看板”，而不是目标中的 AI 软件公司工作流系统。

---

## 当前阶段判断
项目已经从“抽象方法论”推进到“**Codex 可以开始施工的文档化产品设计阶段**”，
但仍需要持续做一致性收口，保证实现层少猜、少返工。

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
3. `product/implementation-phases.md`
4. `product/api-contracts.md`
5. `product/task-transition-api-and-actions.md`

---

## 文档职责分工
- `README.md`：项目目标、当前范围、核心命题、阶段判断
- `docs-map.md`：整套文档导航与阅读路径
- `product/product-implementation.md`：第一阶段产品实施约束
- `product/implementation-phases.md`：分阶段施工清单
- `product/api-contracts.md`：主链路 API 契约总表
- `product/task-transition-api-and-actions.md`：transition 单一真相来源

---

## 当前阶段结论
当前最重要的不是继续扩大概念，而是围绕 Phase 1 MVP 先把以下事情做稳：
- 软件公司模板可启动
- 任务状态机可控推进
- 角色协作可记录、可交接、可审计
- 画布与页面能支撑查看和操作
- 文档足够支持 Codex 分阶段施工

只要这套方法先在软件研发团队场景跑顺，后续才有资格复制到其他行业模板。
