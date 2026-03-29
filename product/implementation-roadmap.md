# product/implementation-roadmap.md

## 目标
定义 AI Company Workflow 当前项目的实施路线图，让 Codex 和人都知道：
- 先做什么
- 后做什么
- 每一段的目标是什么
- 什么叫“这一段做完了”

---

## 路线图原则
- 先做能支撑模板启动与任务闭环的主骨架
- 先做只读可视化，再做高复杂编辑
- 先做 PC 主流程，再补手机轻操作
- 先把研发团队版跑通，再谈多行业扩展

---

## Phase 0：实施准备
### 目标
把产品原则、对象模型、模块拆分、实施阶段和 Codex 交付规则文档补齐。

### 产出
- product principles / implementation
- domain model
- module breakdown
- implementation phases
- codex delivery rules
- canvas ui spec
- screens and flows

### 完成标准
- Codex 可以根据文档确定实现顺序
- 前后端不会围绕不同对象模型各写各的

---

## Phase 1：模板与任务主骨架
### 目标
做出最小可运行后端骨架和基础前端页面。

### 重点
- CompanyTemplate / RoleTemplate / WorkflowTemplate / TaskTemplate 基础模型
- Workspace 实例化
- TaskInstance 基础 CRUD
- 前端模板页、工作空间页、任务列表页

### 完成标准
- 用户可以看到软件公司模板
- 用户可以基于模板启动一个 workspace
- 用户可以创建和查看基础任务

---

## Phase 2：workflow 主链路跑通
### 目标
让 feature / bugfix / hotfix 的主状态流转真正跑起来。

### 重点
- WorkflowInstance
- 状态推进
- owner 切换
- DecisionRecord / HandoffRecord
- 执行日志

### 完成标准
- 一个任务能从 Draft 走到 Review / Waiting Decision / Ready for Delivery
- 角色切换可追踪
- 决策门可记录

---

## Phase 3：画布只读主界面
### 目标
让用户能在画布上看清任务走到哪了。

### 重点
- CanvasNode / CanvasEdge 投影
- 当前节点高亮
- 节点详情面板
- 任务 / 角色 / 审批节点展示

### 完成标准
- 用户可在 PC 端一眼看懂主流程
- 用户可点击节点查看详情
- 画布可支撑演示与日常查看

---

## Phase 4：审批与交付收口
### 目标
补齐 Waiting Decision、回归、Ready for Delivery 这条线。

### 重点
- 审批页
- 决策建议与结果记录
- 回归结论展示
- Ready for Delivery 收口摘要

### 完成标准
- 用户可在系统里完成拍板
- 交付前状态与风险可见
- hotfix / bugfix 收口链路跑通

---

## Phase 5：角色与技能挂载
### 目标
把技能系统从概念设计推进到默认挂载和可见。

### 重点
- 角色默认技能包
- workflow 节点技能展示
- 启用 / 禁用基础技能包

### 完成标准
- 角色不再只是名字和职责
- 节点详情可显示当前启用技能
- 模板默认能力组合可见

---

## Phase 6：LLM 辅助创建与手机端增强
### 目标
把模板化启动进一步做顺手。

### 重点
- LLM 辅助生成角色 / workflow 草稿
- 手机端查看状态
- 手机端审批拍板
- 手机端查看 hotfix 摘要

### 完成标准
- 新用户可以更快启动模板
- 手机端不只是只读看板

---

## 当前推荐实施顺序
1. Phase 0
2. Phase 1
3. Phase 2
4. Phase 3
5. Phase 4
6. Phase 5
7. Phase 6

不要跳过前面的对象模型和模块拆分，直接去做画布炫技层。

---

## 一句话总结
当前项目的正确路线不是“想到哪做哪”，而是：
**先模板与任务骨架，再 workflow 主链路，再画布，再审批收口，再技能，再 LLM 与手机端。**
