# docs-map.md

## 目标
这份文档只负责一件事：

**为当前仓库提供清晰、可执行的文档导航地图。**

它不替代 README，也不替代产品实施文档，更不重新定义 API 或状态机。

它主要回答：
- 这份文档是干什么的
- 什么时候该看它
- 它和其他文档的关系是什么
- 如果我是人 / Codex / 实现者，应该按什么顺序读

---

## 当前文档治理口径
为避免文档重复、职责混乱，当前仓库默认采用以下分工：

- `README.md`：负责**项目目标、当前范围、核心命题、阶段判断**
- `docs-map.md`：负责**导航、阅读顺序、职责关系**
- `product/product-implementation.md`：负责**第一阶段产品实施约束**
- `product/api-contracts.md`：负责**主链路 REST API 契约总表**
- `product/task-transition-api-and-actions.md`：负责**任务 transition 的单一真相来源**
- `product/domain-model.md`：负责**对象建模与对象关系**
- `product/implementation-phases.md`：负责**按 phase 施工的工程化拆分**

如果一份文档开始同时承担“项目介绍 + 导航 + 实施细则 + 接口定义”，就说明职责已经跑偏，需要回收。

---

## 第一层：项目级入口文档

### `README.md`
- 用途：定义项目目标、当前 Phase 1 范围、四个核心命题、阶段判断
- 什么时候看：第一次进入仓库时
- 主要回答：这个项目当前到底在做什么、边界在哪里、为什么先做软件公司模板
- 不负责：完整导航、接口细节、状态机实现细节

### `AGENTS.md`
- 用途：定义项目级执行硬规则
- 什么时候看：开始任何正式执行、改写、实现前
- 主要回答：什么情况下能执行、什么情况下必须停、执行型文档应该长什么样
- 优先级：若与局部文档冲突，安全 / 停机 / 审批规则以它为准

### `docs-map.md`
- 用途：为整套文档做导航总览
- 什么时候看：不知道先读哪份文档，或需要定位资料时
- 主要回答：每份文档各自解决什么问题

---

## 第二层：Phase 1 产品实施主文档

### `product/product-principles.md`
- 用途：定义产品设计原则
- 什么时候看：判断产品方向、模板策略、界面克制原则时
- 主要回答：第一阶段产品应该优先服务谁、应该做成什么感觉

### `product/product-implementation.md`
- 用途：定义第一阶段产品实施约束
- 什么时候看：做原型、定技术选型、划前后端边界、讨论端侧职责时
- 主要回答：第一阶段到底做什么、不做什么、前后端分别承担什么职责
- 不负责：逐条 API、逐条状态机、逐 phase endpoint 清单

### `product/domain-model.md`
- 用途：定义第一阶段核心领域模型
- 什么时候看：建后端实体、前端核心状态模型、模板实例化关系时
- 主要回答：系统有哪些核心对象、模板与实例怎么区分、记录模型怎么归类

### `product/module-breakdown.md`
- 用途：定义前后端模块拆分
- 什么时候看：规划实现顺序、判断模块边界时
- 主要回答：系统该拆成哪些模块、哪些是 P0 / P1 / P2

### `product/implementation-roadmap.md`
- 用途：定义总体实施路线图
- 什么时候看：讨论项目顺序、判断先做什么后做什么时
- 主要回答：为什么是这个实施顺序

### `product/implementation-phases.md`
- 用途：把路线图压成可施工的 phase 清单
- 什么时候看：准备进入某个 phase 开工时
- 主要回答：当前 phase 的实体、接口、页面、验证和退出标准是什么

### `product/codex-delivery-rules.md`
- 用途：定义 Codex / 自动化编码代理在当前项目中的交付规则
- 什么时候看：准备让 Codex 按文档持续施工时
- 主要回答：每轮交付应如何汇报、如何验证、何时允许进入下一阶段

---

## 第三层：核心产品专题文档

### `product/api-contracts.md`
- 用途：定义第一阶段主链路 API 契约总表
- 什么时候看：开始对接前后端接口、列 endpoint 与返回结构时
- 主要回答：模板、workspace、task、workflow、decision、role/skill 这些资源接口怎么定
- 注意：task transition 的动作模型以 `product/task-transition-api-and-actions.md` 为准

### `product/task-status-guards.md`
- 用途：定义统一任务状态机、合法流转、关键留痕规则
- 什么时候看：实现任务状态推进、审核收口、Waiting Decision / Ready for Delivery 规则时
- 主要回答：哪些状态允许跳、什么时候必须生成 decision / handoff / review / delivery summary 记录

### `product/task-transition-api-and-actions.md`
- 用途：把状态守卫翻译成 transition API、动作按钮、后端校验规则
- 什么时候看：设计任务流转接口、前端动作模型、后端 guard validator 时
- 主要回答：为什么不能直接改 status、action 如何定义、transition 成功后要产生哪些 side effects
- 优先级：它是 task transition 的实现单一真相来源

### `product/template-instantiation-flow.md`
- 用途：定义模板如何实例化成可运行 workspace
- 什么时候看：实现“从模板启动”流程时
- 主要回答：用户点击启动后生成哪些对象、先看到什么、哪些是自动完成

### `product/minimum-software-company-template.md`
- 用途：定义“第一个软件公司模板”的最小边界
- 什么时候看：设计默认模板内容、判断启动后系统应交付什么时
- 主要回答：模板本体到底包含哪些角色、workflow、任务样例、技能包和首页骨架

### `product/role-skill-mapping.md`
- 用途：定义角色、技能包、workflow 节点之间的映射
- 什么时候看：实现角色与技能页、节点详情技能展示时
- 主要回答：哪个角色默认带哪些技能、哪个节点展示哪些技能

### `product/skill-system-design.md`
- 用途：定义技能模块设计，以及技能如何与角色、workflow、界面暴露层协同
- 什么时候看：设计技能层、模板能力组合、界面复杂度控制时
- 主要回答：技能模块该不该有、如何做成“内核复杂、界面克制”

### `product/canvas-ui-spec.md`
- 用途：定义 workflow 画布布局与交互规格
- 什么时候看：实现画布页、讨论布局与操作方式时
- 主要回答：画布怎么摆、主画布上显示什么、详情侧栏展示什么

### `product/screens-and-flows.md`
- 用途：定义第一阶段主要页面与页面流转
- 什么时候看：做页面规划、确定 PC / 手机分工时
- 主要回答：系统有哪些页面、页面之间如何跳转

---

## 第四层：治理与执行控制文档

### `governance/task-schema.md`
- 用途：定义统一任务单结构
- 什么时候看：创建、补全、校验任务单时
- 主要回答：一个任务最少要有哪些字段

### `governance/decision-gates.md`
- 用途：定义审批门和停机点
- 什么时候看：遇到不确定、高风险、需拍板问题时
- 主要回答：什么时候必须停下来等人决策

### `governance/severity-priority-rules.md`
- 用途：定义 bug / incident / hotfix 的严重度与优先级口径
- 什么时候看：判断问题是否要升级 hotfix、是否应插队时
- 主要回答：Severity 怎么判、Priority 怎么判、什么时候该走 hotfix

### `governance/regression-checklist.md`
- 用途：定义变更后的最小回归检查清单
- 什么时候看：bugfix / hotfix / 高风险 feature 准备收口时
- 主要回答：这次改动至少该回归到什么程度、哪些结果必须记录

### `governance/ready-for-delivery-checklist.md`
- 用途：定义进入 `Ready for Delivery` 前的最小收口要求
- 什么时候看：准备把任务交给用户 / approver 做最终确认时
- 主要回答：什么条件下才算真正具备可交付性

### `governance/handoff-templates.md`
- 用途：定义角色之间的标准交接格式
- 什么时候看：任何角色切换时
- 主要回答：怎么把任务清楚交给下一角色

### `governance/execution-order.md`
- 用途：定义 Codex 执行时的阅读与操作顺序
- 什么时候看：任务刚开始或不确定下一步先读什么时
- 主要回答：先看哪份文档，再做什么

### `governance/postmortem-template.md`
- 用途：定义复杂 bug / hotfix 收口后的统一复盘模板
- 什么时候看：问题已止血或修复，需要复盘和 follow-up 时
- 主要回答：这次到底发生了什么、为什么发生、后续怎么补

---

## 第五层：角色、工作流与样例文档

### `roles/roles.md`
- 用途：定义岗位总览和角色关系
- 什么时候看：理解组织结构时
- 主要回答：系统里有哪些角色、各自负责什么

### `roles/playbooks/*.md`
- 用途：定义具体角色的执行手册
- 什么时候看：当前负责人切换到对应角色时
- 主要回答：这个角色该看什么、做什么、向谁交接

### `workflows/workflow.md`
- 用途：定义研发团队版工作流总体说明
- 什么时候看：理解总体流转时
- 主要回答：研发任务一般怎么流转

### `workflows/playbooks/*.md`
- 用途：定义 feature / bugfix / hotfix 的 SOP
- 什么时候看：任务类型已经确定时
- 主要回答：该类型任务从创建到收口如何流转

### `examples/研发团队-首条闭环示例.md`
- 用途：给出一条完整样板流程
- 什么时候看：想快速理解整套体系如何落地时
- 主要回答：这套流程在真实任务里长什么样

### `examples/tasks/*.md`
- 用途：提供可复用的任务样例
- 什么时候看：准备创建任务、或想看任务字段如何填写时
- 主要回答：一条标准 feature / bugfix / hotfix 任务单应该长什么样

---

## 推荐阅读路径

### 路径 A：第一次看项目的人
1. `README.md`
2. `AGENTS.md`
3. `docs-map.md`
4. `product/product-implementation.md`
5. `product/domain-model.md`
6. `product/implementation-phases.md`
7. `product/api-contracts.md`
8. `product/task-transition-api-and-actions.md`
9. `product/canvas-ui-spec.md`
10. `product/screens-and-flows.md`
11. `roles/roles.md`
12. `workflows/workflow.md`
13. `examples/研发团队-首条闭环示例.md`

### 路径 B：准备按文档施工的 Codex / 开发者
1. `AGENTS.md`
2. `product/domain-model.md`
3. `product/module-breakdown.md`
4. `product/implementation-roadmap.md`
5. `product/implementation-phases.md`
6. `product/codex-delivery-rules.md`
7. `product/api-contracts.md`
8. `product/task-transition-api-and-actions.md`
9. `product/template-instantiation-flow.md`
10. `product/minimum-software-company-template.md`
11. `product/canvas-ui-spec.md`
12. `product/screens-and-flows.md`
13. `governance/task-schema.md`
14. `governance/decision-gates.md`
15. 进入当前 phase 的具体实现

### 路径 C：只想理解任务状态机的人
1. `product/task-status-guards.md`
2. `product/task-transition-api-and-actions.md`
3. `product/api-contracts.md`
4. `governance/decision-gates.md`
5. `governance/ready-for-delivery-checklist.md`

---

## 当前最重要的 7 个治理文件
如果当前目标是做 Phase 1 MVP 的项目级文档治理，优先关注：
1. `README.md`
2. `docs-map.md`
3. `product/product-implementation.md`
4. `product/api-contracts.md`
5. `product/domain-model.md`
6. `product/implementation-phases.md`
7. `product/task-transition-api-and-actions.md`

这 7 份文档需要保持口径一致，否则 Codex 很容易进入“局部看起来都对，但合起来互相打架”的状态。

---

## 一句话说明
如果 `README.md` 解决的是“这个项目现在要做什么”，
那 `docs-map.md` 解决的就是：

**“这套文档分别解决什么问题，应该按什么路径去读。”**
