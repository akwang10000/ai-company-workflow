# docs-map.md

## 目标
为本项目现有文档提供一张总览地图，方便人和 Codex 快速定位：
- 这份文档是干什么的
- 什么时候该看它
- 它和别的文档是什么关系

---

## 当前文档地图

### 1. 项目层
#### `README.md`
- 用途：定义项目目标、当前方向、边界
- 什么时候看：第一次进入项目时
- 主要回答：这个项目想干什么、当前先做哪个方向

#### `AGENTS.md`
- 用途：定义项目执行硬规则
- 什么时候看：开始任何正式执行前
- 主要回答：哪些事能自动做，哪些事必须停

#### `docs-map.md`
- 用途：为整套文档做导航总览
- 什么时候看：不知道先读哪份、或想快速定位资料时
- 主要回答：每份文档各自解决什么问题

#### `product/product-principles.md`
- 用途：定义产品设计原则
- 什么时候看：判断产品方向、模板策略、易用性优先级时
- 主要回答：这产品应该做成什么感觉、优先满足谁

#### `product/product-implementation.md`
- 用途：定义第一阶段产品实施约束
- 什么时候看：做原型、定技术选型、规划前后端职责时
- 主要回答：第一阶段应该怎么落地、后端和前端分别怎么定

#### `product/skill-system-design.md`
- 用途：定义技能模块设计，以及技能如何与角色、workflow、界面暴露层协同
- 什么时候看：设计技能层、模板能力组合、界面复杂度控制时
- 主要回答：技能模块该不该有、如何做成“内核复杂、界面克制”

#### `product/domain-model.md`
- 用途：定义当前项目第一阶段的核心领域模型
- 什么时候看：建后端实体、前端状态、模板实例化关系时
- 主要回答：系统里有哪些核心对象，它们之间怎么关联

#### `product/module-breakdown.md`
- 用途：定义当前项目的前后端模块拆分
- 什么时候看：规划实现顺序、决定先做哪个模块时
- 主要回答：系统该拆成哪些模块，优先级怎么排

#### `product/implementation-roadmap.md`
- 用途：定义当前项目的总体实施路线图
- 什么时候看：做阶段规划、判断先后顺序时
- 主要回答：整个项目先做什么、后做什么

#### `product/implementation-phases.md`
- 用途：把路线图细化成 Codex 可分阶段推进的实施阶段
- 什么时候看：准备进入某个 phase 开工时
- 主要回答：当前阶段该交付什么、什么算做完

#### `product/codex-delivery-rules.md`
- 用途：定义 Codex 在当前项目实施中的交付规则
- 什么时候看：准备让 Codex 按文档持续施工时
- 主要回答：每轮改动怎么交付、怎么验证、什么时候能进下一阶段

#### `product/canvas-ui-spec.md`
- 用途：定义 workflow 画布的布局与交互规格
- 什么时候看：实现画布页、讨论布局与操作方式时
- 主要回答：画布怎么摆、怎么点、怎么展示详情

#### `product/screens-and-flows.md`
- 用途：定义第一阶段主要页面与页面流转
- 什么时候看：做前端页面规划、PC/手机分工时
- 主要回答：系统有哪些页面、页面之间怎么跳转

#### `product/api-contracts.md`
- 用途：定义第一阶段主链路的前后端 API 契约
- 什么时候看：开始对接前后端接口、确定返回结构时
- 主要回答：模板、任务、画布、审批这些主链路接口怎么定

#### `product/template-instantiation-flow.md`
- 用途：定义模板如何实例化成可运行 workspace
- 什么时候看：实现“从模板启动”流程时
- 主要回答：用户点下启动后，系统会生成什么、怎么进入第一条任务

#### `product/role-skill-mapping.md`
- 用途：定义角色、技能包、workflow 节点之间的映射
- 什么时候看：实现角色与技能页、节点详情技能展示时
- 主要回答：哪个角色默认带哪些技能、哪个节点展示哪些技能

#### `product/minimum-software-company-template.md`
- 用途：定义第一个软件公司模板的最小启动骨架
- 什么时候看：设计默认模板内容、判断启动后系统应交付什么时
- 主要回答：模板本体到底包含哪些角色、workflow、任务样例、技能包和首页骨架

#### `product/task-status-guards.md`
- 用途：定义统一任务状态机、合法流转和关键留痕规则
- 什么时候看：实现任务状态流转、审核收口、Waiting Decision 与 Ready for Delivery 规则时
- 主要回答：哪些状态能跳、哪些不能跳、什么时候必须生成 decision / handoff / review / delivery summary

#### `product/task-transition-api-and-actions.md`
- 用途：把状态守卫翻译成 transition API、动作按钮和后端状态机实现规则
- 什么时候看：开始设计任务流转接口、前端动作模型、后端 guard validator 时
- 主要回答：动作怎么定义、API 怎么收、前后端分别怎么落状态机

---

### 2. 任务与控制层
#### `governance/task-schema.md`
- 用途：定义统一任务单结构
- 什么时候看：创建、补全、校验任务单时
- 主要回答：一个任务最少要有哪些字段

#### `governance/decision-gates.md`
- 用途：定义审批门和停机点
- 什么时候看：遇到不确定、高风险、需拍板问题时
- 主要回答：什么时候必须停下来等人决策

#### `governance/severity-priority-rules.md`
- 用途：定义 bug / incident / hotfix 的严重度与优先级口径
- 什么时候看：判断问题到底多严重、是否要插队、是否要升级 hotfix 时
- 主要回答：Severity 怎么判、Priority 怎么判、什么时候该走 hotfix

#### `governance/regression-checklist.md`
- 用途：定义变更后的最小回归检查清单
- 什么时候看：bugfix / hotfix / 高风险 feature 准备收口时
- 主要回答：这次改动至少该回归到什么程度、哪些必须记录

#### `governance/ready-for-delivery-checklist.md`
- 用途：定义进入 `Ready for Delivery` 前的最小收口要求
- 什么时候看：准备把任务交给用户 / CEO / Approver 做最终确认时
- 主要回答：什么条件下才算真正具备可交付性

#### `governance/handoff-templates.md`
- 用途：定义角色之间的标准交接格式
- 什么时候看：任何角色切换时
- 主要回答：怎么把任务清楚地交给下一角色

#### `governance/execution-order.md`
- 用途：定义 Codex 执行时的阅读与操作顺序
- 什么时候看：任务刚开始、或不确定下一步先读什么时
- 主要回答：先看哪份文档，再做什么

#### `governance/postmortem-template.md`
- 用途：定义复杂 bug / hotfix 收口后的统一复盘模板
- 什么时候看：问题已止血或修复、需要复盘和 follow-up 时
- 主要回答：这次到底发生了什么、为什么发生、后续怎么补

---

### 3. 角色层
#### `roles/roles.md`
- 用途：定义岗位总览和角色关系
- 什么时候看：理解组织结构时
- 主要回答：项目里有哪些角色，各自负责什么

#### `roles/playbooks/PM-Agent-playbook.md`
- 用途：PM Agent 执行手册
- 什么时候看：当前负责人是 PM Agent 时

#### `roles/playbooks/Tech-Lead-Agent-playbook.md`
- 用途：Tech Lead Agent 执行手册
- 什么时候看：当前负责人是 Tech Lead Agent 时

#### `roles/playbooks/Dev-Agent-playbook.md`
- 用途：Dev Agent 执行手册
- 什么时候看：当前负责人是 Dev Agent 时

#### `roles/playbooks/QA-Review-Agent-playbook.md`
- 用途：QA / Review Agent 执行手册
- 什么时候看：当前负责人是 QA / Review Agent 时

#### `roles/playbooks/Ops-Release-Agent-playbook.md`
- 用途：Ops / Release Agent 执行手册
- 什么时候看：当前负责人是 Ops / Release Agent 时

---

### 4. 工作流层
#### `workflows/workflow.md`
- 用途：研发团队版工作流总体说明
- 什么时候看：理解总体流转时
- 主要回答：研发任务一般怎么流转

#### `workflows/playbooks/feature-workflow.md`
- 用途：标准 feature 任务执行 SOP
- 什么时候看：任务类型为 feature 时
- 主要回答：feature 从提出到归档的具体步骤

#### `workflows/playbooks/bugfix-workflow.md`
- 用途：标准 bug 修复任务执行 SOP
- 什么时候看：任务类型为 bugfix 时
- 主要回答：bug 从登记、定位、修复到回归验证如何流转

#### `workflows/playbooks/hotfix-workflow.md`
- 用途：紧急 hotfix 任务执行 SOP
- 什么时候看：线上问题紧急、需要快速止血时
- 主要回答：在高时效压力下，如何最小化失控风险地完成抢修

---

### 5. 示例层
#### `examples/研发团队-首条闭环示例.md`
- 用途：提供一条完整样板流程
- 什么时候看：想快速理解这套体系如何落地时
- 主要回答：这套流程在真实任务中长什么样

#### `examples/tasks/feature-example.md`
- 用途：提供一条可直接复用的 feature 任务单样例
- 什么时候看：要创建常规功能任务、或想看 feature 任务字段怎么写时
- 主要回答：一条标准 feature 任务单应该长什么样

#### `examples/tasks/bugfix-example.md`
- 用途：提供一条可直接复用的 bugfix 任务单样例
- 什么时候看：要创建常规缺陷任务、或想看 bugfix 任务字段怎么写时
- 主要回答：一条标准 bugfix 任务单应该长什么样

#### `examples/tasks/hotfix-example.md`
- 用途：提供一条可直接复用的 hotfix 任务单样例
- 什么时候看：要创建事故级修复任务、或想看高风险场景如何写任务单时
- 主要回答：一条标准 hotfix 任务单应该如何兼顾速度与风险控制

---

## 推荐阅读路径

### 对人类读者
1. `README.md`
2. `AGENTS.md`
3. `product/product-principles.md`
4. `product/product-implementation.md`
5. `product/domain-model.md`
6. `product/module-breakdown.md`
7. `product/implementation-roadmap.md`
8. `product/implementation-phases.md`
9. `product/api-contracts.md`
10. `product/template-instantiation-flow.md`
11. `product/role-skill-mapping.md`
12. `product/canvas-ui-spec.md`
13. `product/screens-and-flows.md`
14. `product/skill-system-design.md`
15. `roles/roles.md`
16. `workflows/workflow.md`
17. `examples/研发团队-首条闭环示例.md`
18. 按任务类型继续读对应 workflow
19. 需要落任务时参考 `examples/tasks/*.md`
20. 涉及 bug / incident 判断时参考 `governance/severity-priority-rules.md`
21. 准备收口时参考 `governance/regression-checklist.md` 与 `governance/ready-for-delivery-checklist.md`
22. hotfix / 复杂 bug 收口时参考 `governance/postmortem-template.md`

### 对 Codex / 智能体
1. `AGENTS.md`
2. `product/domain-model.md`
3. `product/module-breakdown.md`
4. `product/implementation-roadmap.md`
5. `product/implementation-phases.md`
6. `product/codex-delivery-rules.md`
7. `product/api-contracts.md`
8. `product/template-instantiation-flow.md`
9. `product/role-skill-mapping.md`
10. `product/canvas-ui-spec.md`
11. `product/screens-and-flows.md`
12. `product/product-implementation.md`
13. `governance/task-schema.md`
14. `governance/decision-gates.md`
15. 若为 bug / incident / hotfix，读取 `governance/severity-priority-rules.md`
16. 具体 workflow 文档
17. 当前角色对应的 role playbook
18. `governance/handoff-templates.md`
19. 若需要快速起任务，可参考 `examples/tasks/*.md`
20. 若任务进入收口阶段，读取 `governance/regression-checklist.md` 与 `governance/ready-for-delivery-checklist.md`
21. 若任务需要事故复盘或复杂收口，参考 `governance/postmortem-template.md`

---

## 当前缺口
还建议后续继续补：
- 更多 `examples/tasks/`（接口类 bug、权限类 bug、回滚类 hotfix）
- `task-schema` 的 JSON Schema 版本
- 多场景复用时的字段裁剪策略
- 前端状态模型与接口错误码的更细文档
- 如果后续需要把用户代理化，再考虑 `roles/playbooks/CEO-Agent-playbook.md`

---

## 一句话说明
如果 `governance/execution-order.md` 解决的是“先干什么”，
那 `docs-map.md` 解决的就是“这堆文档分别是干嘛的”；
而新增的 product 实施文档，解决的是“当前项目本身该怎么一步步做出来”。
