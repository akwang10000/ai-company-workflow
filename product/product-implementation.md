# product/product-implementation.md

## 目标
定义 AI Company Workflow 在 **Phase 1 MVP** 的产品实施约束，确保后续原型、后端、前端、交互设计和 Codex 施工沿同一条路径推进，而不是各写各的。

这份文档主要回答：
- Phase 1 到底做什么
- Phase 1 明确不做什么
- 前后端分别承担什么职责
- 端侧形态、模板优先、状态推进、画布交互应遵守哪些约束
- 如何把“系统内部可以复杂、用户使用必须简单”落实到实现层

它不替代：
- `README.md` 的项目总览职责
- `product/api-contracts.md` 的接口明细职责
- `product/task-transition-api-and-actions.md` 的 transition 单一真相职责
- `product/implementation-phases.md` 的工程化 phase 清单职责

---

## 当前实施范围
当前项目 **只聚焦 Phase 1 MVP**。

Phase 1 不是做“通用 AI 公司操作系统”，而是先把：

## **第一个软件公司模板**

做成一个：
- 可启动
- 可演示
- 可复用
- 可按阶段继续施工

的最小产品骨架。

### Phase 1 要覆盖的核心能力
- 模板启动
- 角色分工
- 任务单管理
- action-driven 状态推进
- 决策门 / 审批点 / 交接记录 / 审核收口
- workflow 画布查看与轻操作
- 角色与技能的默认挂载和展示
- 当前项目自身的前后端实施蓝图文档

### Phase 1 优先覆盖的任务类型
- `feature`
- `bugfix`
- `hotfix`

### Phase 1 明确不强求
- 通用模板市场
- 多行业同时覆盖
- 高复杂度权限系统
- 大规模自动化编排引擎
- 深度 BI / 报表系统
- 多租户企业级治理全量能力
- 自由度极高的高级流程编排器

---

## 当前产品实施判断框架
Phase 1 的产品实施，默认围绕以下 4 个命题展开：

### A. 模板化启动是否成立
用户能否不从空白开始，而是通过“软件公司模板”直接拉起一个 workspace，并获得默认角色、默认 workflow、默认任务样例、默认技能骨架。

### B. 任务推进是否能被结构化约束
系统是否真正做到：
- 不是随便改 status
- 而是由 action 驱动 transition
- 配合 guard、必填记录、审批 / 评审 / 决策门完成受控推进

### C. AI 角色协作是否能被产品化
系统是否支持按明确角色拆分协作，而不是只提供一个大而化之的 AI 助手。

### D. 文档是否足够支持自动施工
文档是否足以让 Codex / 自动化编码代理按 phase 开工，而不是实现时大量猜测。

---

## Phase 1 的目标体验
Phase 1 不是要让用户感受到“这个系统很强大”，而是要让用户感受到：
- 一进来就能开始，而不是先配置半天
- 一看就知道卡在哪，而不是先理解内部模型
- 一点就能执行关键动作，而不是先研究状态机术语
- 默认界面足够简单，复杂性藏在系统内部

如果 Phase 1 做对了，用户应该觉得：

**这个系统虽然底层严谨，但用起来不费脑。**

---

## Phase 1 的核心产品口径

## **前端不直接改 status，而是通过 action 驱动任务流转**

这条规则是当前 Phase 1 最重要的产品 / 架构约束。

前端不应暴露一个任意可写的状态下拉框，
而应暴露明确动作，例如：
- 开始执行
- 提交交接
- 请求决策
- 开始审核
- 驳回返工
- 标记可交付
- 完成交付

后端根据：
- 当前状态
- guard 校验
- payload 完整性
- 必填记录
- actor 角色权限

来决定：
- 是否允许执行
- 目标状态是什么
- 要生成哪些记录
- 下一步建议动作是什么

如果 Phase 1 没有把这条口径做稳，产品就会退化成“带流程名词的任务系统”，而不是可审计、可回溯的 AI 软件公司工作流系统。

---

## 复杂度暴露约束

### 原则一：内核可以复杂，默认界面必须克制
以下复杂度允许存在于工程内部：
- guard 校验
- transition registry
- decision / handoff / review / delivery summary 记录模型
- skill schema / tool binding / fallback policy
- workflow runtime 与 canvas projection

但默认界面不应直接暴露这些内部结构。

### 原则二：默认只展示“当前完成任务所需信息”
默认界面优先展示：
- 当前状态 / 当前节点
- 当前负责人
- 当前卡点
- 当前可执行动作
- 最近一条必要记录摘要

默认不优先展示：
- 内部 payload 结构
- schema / timeout / retry
- 全量日志
- 所有技能参数
- 全部内部状态字段

### 原则三：复杂配置折叠进高级层
允许存在高级配置，但必须满足：
- 默认收起
- 不阻断新用户启动
- 不要求普通用户理解才能完成主链路
- 不让主界面变成专家后台

### 原则四：用户看到的是动作，不是内部术语
用户界面应优先使用：
- 提交交接
- 请求拍板
- 驳回返工
- 标记可交付
- 完成交付

而不是直接把这些内部词扔给用户：
- transition
- payload
- guard failed
- workflow instance
- role assignment id

系统内部可以保留这些术语，UI 默认不应把它们当主语言。

---

## 后端实施约束

### 建议技术选型
第一阶段后端优先采用：
- **Java + Spring Boot**

### 选择原因
- 适合承载任务流转、状态机、审批流、模板管理等强业务建模需求
- 生态成熟，便于后续扩展审计、消息通知、持久化和集成能力
- 对流程系统 / 企业应用 / 工作台类产品的长期演进更稳

### 后端 Phase 1 应优先承担
- `CompanyTemplate` / `RoleTemplate` / `WorkflowTemplate` / `TaskTemplate` 管理
- `Workspace` 实例化
- `TaskInstance` CRUD
- action-driven transition
- `DecisionRecord` / `HandoffRecord` / `ReviewRecord` / `DeliverySummaryRecord` / `ExecutionLog`
- 角色与默认技能绑定关系管理
- 画布节点 / 边投影数据输出
- LLM 辅助创建能力的接口预留

### 后端 Phase 1 不应过早承担
- 高自由度流程编排器
- 复杂运行时技能编排引擎
- 大量跨 workspace 的平台治理能力
- 为未来多行业场景过度抽象的统一大模型

---

## 前端实施约束

### 前端目标
前端第一阶段必须优先支持：
- **PC + 手机双端可用**

### 原则
不是“PC 做完整、手机凑合能看”，而是从产品设计阶段就考虑：
- PC：搭建、查看全局、看画布、处理复杂信息
- 手机：看进度、看卡点、审批拍板、做轻量操作

### PC 端优先场景
- 模板列表与模板详情
- workspace 首页
- 任务列表与任务详情
- workflow 画布页
- 决策中心 / 审批页
- 角色与技能页

### 手机端优先场景
- 查看当前任务状态
- 查看当前卡点
- 接收审批提醒 / 决策提醒
- 快速批准 / 驳回 / 备注
- 查看 bugfix / hotfix 进展摘要
- 查看当前节点摘要与风险说明

### 前端设计要求
- 不允许关键流程只能在单一端完成
- PC 与手机应共享同一套核心业务模型
- 手机端可以弱化复杂编辑，但不能缺失关键查看和拍板能力
- 默认界面应少参数暴露，复杂配置折叠到高级层
- 新用户应能在不理解内部模型的情况下完成第一条主链路操作

---

## 模板驱动实施要求
Phase 1 必须坚持：
- 模板优先，而不是空白优先
- 新用户优先从“第一个软件公司模板”启动
- 角色、workflow、task 都要有预制模板
- 技能优先以默认技能包形式随角色与模板一起交付

### 最小启动模板至少包含
- PM Agent
- Tech Lead Agent
- Dev Agent
- QA / Review Agent
- Ops / Release Agent
- feature workflow
- bugfix workflow
- hotfix workflow
- 3 类任务样例
- 一组基础技能包

### 模板实例化后的最低体验要求
用户点击“从模板启动”后，应默认得到：
- 1 个 workspace
- 1 组默认角色实例
- 3 套 workflow 骨架
- 3 类任务样例入口
- 1 套基础 dashboard 数据结构
- 1 组默认技能挂载结果

用户此时应该可以：
- 不自己先定义角色
- 不自己先手工画流程
- 不自己先发明任务字段
- 直接开始创建第一条任务

---

## 工作流画布实施要求

### 目标
workflow 画布不是装饰层，而是产品核心操作界面之一。

### Phase 1 画布至少要支持
- 节点展示：角色节点、任务节点、审批节点、状态节点
- 连线展示：标准流转、返工流转、决策流转
- 当前状态高亮
- 当前负责人可见
- 点击节点查看详情
- 从画布进入任务详情或审批页

### 体验要求
- 用户一眼看懂“任务现在走到哪了”
- 优先考虑易懂和易操作，不做成 BPMN 专家工具
- 默认模板应能直接落到画布里，而不是只存在文档说明中
- 技能信息优先放在节点详情层展示，不在主画布堆太多信息
- 用户不应为了完成主链路操作而先理解内部状态机实现

### 进一步规格参考
- `product/canvas-ui-spec.md`
- `product/screens-and-flows.md`
- `product/template-instantiation-flow.md`
- `product/role-skill-mapping.md`

---

## 状态推进与记录留痕实施要求
Phase 1 的任务推进至少要具备：
- 当前状态可解释
- 合法流转可验证
- 非法流转可拒绝
- 关键动作有记录
- 关键问题可进入 `Waiting Decision`
- 收口前有 `Ready for Delivery` 门槛

### 至少要能沉淀的记录类型
- `DecisionRecord`
- `HandoffRecord`
- `ReviewRecord`
- `DeliverySummaryRecord`
- `ExecutionLog`

### 至少要能回答的问题
- 任务为什么到了当前状态
- 是谁推进的
- 基于什么 payload 推进的
- 缺了什么所以不能推进
- 下一步建议动作是什么

### 面向用户的表达要求
这些记录在系统内部可以结构化、严格化；
但默认面向用户展示时，应优先显示摘要和结论，而不是直接倾倒原始结构。

---

## LLM 辅助创建要求
Phase 1 应预留并逐步支持：
- 用户通过自然语言描述团队或业务场景
- LLM 生成角色模板初稿
- LLM 生成 workflow 草稿
- LLM 推荐技能组合与模板骨架
- 用户微调后保存为正式模板

### 原则
- LLM 负责起草，不直接替用户拍板
- 用户可以微调角色职责、输入输出、审批边界
- 模板一旦确认，应可固化并复用
- 复杂参数默认不暴露给普通用户

---

## 与其他文档的配套关系
如果目标是让 Codex 依据文档逐步实现当前项目前后端功能，建议配套阅读：
- `product/domain-model.md`：对象怎么建模
- `product/module-breakdown.md`：模块怎么拆
- `product/implementation-roadmap.md`：整体先后顺序
- `product/implementation-phases.md`：每个 phase 做什么
- `product/codex-delivery-rules.md`：每轮怎么交付
- `product/api-contracts.md`：主链路 API 怎么对齐
- `product/task-transition-api-and-actions.md`：任务 transition 怎么实现
- `product/template-instantiation-flow.md`：模板怎么启动
- `product/minimum-software-company-template.md`：模板本体包含什么
- `product/canvas-ui-spec.md`：画布怎么做
- `product/screens-and-flows.md`：页面怎么串起来

---

## Phase 1 完成后的理想效果
如果第一阶段实现到位，用户应能：
- 直接选择一个软件公司最小启动模板
- 启动一个 workspace，而不是从空白开始
- 创建并推进 feature / bugfix / hotfix 任务
- 在 PC 上查看全局、看画布、处理复杂信息
- 在手机上查看进度并完成轻量审批动作
- 通过动作按钮推进任务，而不是直接改 status
- 在关键节点留下 decision / handoff / review / delivery summary 记录
- 让 Codex 能按 phase 接着施工，而不是大量猜测

---

## 当前实施优先级建议
1. 固化后端模型：模板、workspace、task、workflow、records
2. 固化 action-driven transition：先把状态机做稳
3. 固化前端信息架构：PC + 手机共享同一核心模型
4. 做最小画布原型：先能看、能点、能追踪
5. 做模板启动流程：先让用户不从空白开始
6. 再逐步补角色与技能挂载
7. 最后补 LLM 辅助创建能力

更细的施工拆分继续参考：
- `product/implementation-roadmap.md`
- `product/implementation-phases.md`

---

## 一句话原则
Phase 1 不是做“最强配置系统”，而是做一套：

**模板可直接启动、角色协作可追踪、任务推进可受控、画布可理解、双端可用，并且“系统内部可以复杂、用户使用一定要简单”的 AI 软件公司工作流产品。**
