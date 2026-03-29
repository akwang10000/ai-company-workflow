# product/implementation-phases.md

## 目标
把当前项目的实施路线图进一步细化成可执行阶段，让 Codex 可以按阶段施工，而不是一次性铺太大。

---

## Phase 0：文档收口期
### 要做什么
- 补齐产品实施文档
- 固化领域模型
- 固化模块拆分
- 固化画布规格
- 固化 Codex 交付规则

### 本阶段产出
- `product/domain-model.md`
- `product/module-breakdown.md`
- `product/implementation-roadmap.md`
- `product/implementation-phases.md`
- `product/codex-delivery-rules.md`
- `product/canvas-ui-spec.md`
- `product/screens-and-flows.md`

### 完成标准
- 项目实现顺序明确
- 画布和前后端边界明确
- Codex 可以基于文档开始正式施工

---

## Phase 1：后端核心模型期
### 后端目标
- 建立核心实体与仓储
- 跑通模板与任务主骨架

### 建议交付
- CompanyTemplate
- RoleTemplate
- WorkflowTemplate
- TaskTemplate
- Workspace
- TaskInstance
- 基础 CRUD API

### 前端目标
- 先搭应用壳和基础页面框架

### 完成标准
- 模板可查
- workspace 可建
- task 可建可查

---

## Phase 2：workflow 运行期
### 后端目标
- 建 WorkflowInstance
- 建状态推进逻辑
- 建 DecisionRecord / HandoffRecord / ExecutionLog

### 前端目标
- 接任务详情页
- 展示状态流转和交接记录

### 完成标准
- feature / bugfix / hotfix 至少各能跑主链路
- 当前 owner、当前状态、下一步接收方可见

---

## Phase 3：画布展示期
### 后端目标
- 输出画布投影数据

### 前端目标
- 做 Workflow 画布页
- 节点高亮
- 连线展示
- 右侧详情面板

### 完成标准
- 用户能在画布页看懂流程走到哪
- 节点详情可查看输入、输出、风险、技能、日志摘要

---

## Phase 4：审批与收口期
### 后端目标
- Waiting Decision API
- Ready for Delivery 摘要接口
- 回归与审核结果展示接口

### 前端目标
- 审批页
- 收口页 / 交付摘要组件

### 完成标准
- 用户可在系统中拍板
- bugfix / hotfix 收口可见
- Ready for Delivery 流程跑通

---

## Phase 5：角色与技能期
### 后端目标
- SkillPackageTemplate
- RoleTemplate 与 SkillPackage 绑定
- 节点技能映射

### 前端目标
- 角色与技能页
- 节点详情显示技能

### 完成标准
- 角色默认技能包可见
- workflow 节点默认技能可见

---

## Phase 6：LLM 与手机端期
### 后端目标
- LLM 辅助草稿生成接口

### 前端目标
- 手机端查看进度
- 手机端审批
- 角色 / workflow 草稿生成入口

### 完成标准
- 手机端能承担轻操作
- 模板创建不再完全依赖手工填写

---

## Codex 每阶段交付格式建议
每阶段至少输出：
- Changed files
- What changed
- Validation
- Current status
- Next recommended step

---

## 一句话总结
Phase 文档的意义不是分得很花，而是让 Codex 始终知道：
**这一轮该做哪一层，做到什么程度才算能交下一棒。**
