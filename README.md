# AI Company Workflow

## 项目一句话
打造一个由创业者定义目标、由多智能体按岗位协作执行的 AI 组织工作流系统；当前第一阶段先用**研发团队版**验证这套方法是否真能跑通。

## 当前定位
这不是“只做研发团队”的固定产品，而是一个可扩展的 AI 公司工作流框架。
当前先拿**研发团队版**做第一阶段验证，后续还会继续尝试：
- AI 短剧自动化生成
- 自媒体运营
- 内容工作室
- 其他可流程化的业务团队

## 当前目标
先验证这件事在研发团队场景下是否具备真实实用价值，并沉淀第一版可运行的：
- 角色体系
- 任务流转方案
- 审批与停机规则
- 交接模板
- 文档导航与执行顺序
- 产品实施约束与模板优先原则
- 可驱动 Codex 分阶段落地当前项目前后端功能的实施文档

## 研发团队版要解决什么问题
- 创业者/技术负责人精力有限，需求分析、任务拆解、开发推进、测试验证都靠自己盯，成本高
- 单个 AI 只能完成点状编码或问答，缺少完整研发协作链路
- 研发工作不是“写一段代码”这么简单，而是需求、设计、开发、测试、验收、发布说明的一整条流程
- 现有自动化工具可以连接系统，但不能天然承担岗位职责、交接责任和审核机制

## 第一阶段边界
本项目第一阶段不追求“完全自治研发公司”，而是先做：
- 角色化研发智能体设计
- 研发任务状态流转
- 岗位协作与交付机制
- 审批与人工兜底
- 模板化启动体验
- 文档化沉淀，方便后续产品化与多行业扩展
- 当前项目自身的前后端实施蓝图

## 第一阶段当前已具备的骨架
目前文档已经覆盖：
- 项目边界与执行硬规则：`README.md`、`AGENTS.md`
- 任务结构：`governance/task-schema.md`
- 决策门：`governance/decision-gates.md`
- 缺陷严重度 / 优先级规则：`governance/severity-priority-rules.md`
- 交接模板：`governance/handoff-templates.md`
- 总体工作流：`workflows/workflow.md`
- 执行顺序：`governance/execution-order.md`
- 回归检查：`governance/regression-checklist.md`
- 交付前收口检查：`governance/ready-for-delivery-checklist.md`
- 角色执行手册：`roles/playbooks/*.md`
- 任务 SOP：
  - `workflows/playbooks/feature-workflow.md`
  - `workflows/playbooks/bugfix-workflow.md`
  - `workflows/playbooks/hotfix-workflow.md`
- 样板案例：`examples/研发团队-首条闭环示例.md`
- 任务样例：
  - `examples/tasks/feature-example.md`
  - `examples/tasks/bugfix-example.md`
  - `examples/tasks/hotfix-example.md`
- 复盘模板：
  - `governance/postmortem-template.md`
- 产品原则与实现文档：
  - `product/product-implementation.md`
  - `product/product-principles.md`
  - `product/skill-system-design.md`
- 当前项目实施文档：
  - `product/domain-model.md`
  - `product/module-breakdown.md`
  - `product/implementation-roadmap.md`
  - `product/implementation-phases.md`
  - `product/codex-delivery-rules.md`
  - `product/canvas-ui-spec.md`
  - `product/screens-and-flows.md`
  - `product/api-contracts.md`
  - `product/template-instantiation-flow.md`
  - `product/role-skill-mapping.md`
  - `product/minimum-software-company-template.md`
  - `product/task-status-guards.md`
- 导航地图：`docs-map.md`

## 推荐阅读顺序
### 如果你是人，第一次看这套文档
1. `README.md`
2. `AGENTS.md`
3. `docs-map.md`
4. `product/product-principles.md`
5. `product/product-implementation.md`
6. `product/domain-model.md`
7. `product/module-breakdown.md`
8. `product/implementation-roadmap.md`
9. `product/implementation-phases.md`
10. `product/api-contracts.md`
11. `product/template-instantiation-flow.md`
12. `product/role-skill-mapping.md`
13. `product/minimum-software-company-template.md`
14. `product/task-status-guards.md`
15. `product/canvas-ui-spec.md`
16. `product/screens-and-flows.md`
17. `roles/roles.md`
18. `workflows/workflow.md`
19. `examples/研发团队-首条闭环示例.md`
20. `examples/tasks/*.md`
21. `governance/severity-priority-rules.md`
22. `governance/regression-checklist.md`
23. `governance/ready-for-delivery-checklist.md`
24. `governance/postmortem-template.md`

### 如果你是 Codex / 智能体，准备开始执行当前项目实施
1. `AGENTS.md`
2. `product/domain-model.md`
3. `product/module-breakdown.md`
4. `product/implementation-roadmap.md`
5. `product/implementation-phases.md`
6. `product/codex-delivery-rules.md`
7. `product/api-contracts.md`
8. `product/template-instantiation-flow.md`
9. `product/role-skill-mapping.md`
10. `product/minimum-software-company-template.md`
11. `product/task-status-guards.md`
12. `product/canvas-ui-spec.md`
13. `product/screens-and-flows.md`
14. `product/product-implementation.md`
15. `governance/task-schema.md`
16. `governance/decision-gates.md`
17. 进入当前阶段具体实现

## 当前阶段结论
- **研发团队版只是第一个验证方向，不是最终唯一方向**
- 当前重点不是做出“很聪明的代理”，而是做出**能交接、能停机、能闭环**的执行体系
- 同时开始明确产品实现边界：模板优先、最小启动优先、双端可用、画布可操作
- 当前还进一步补齐了“如何让 Codex 按阶段实现当前项目前后端功能”的实施文档层
- 模板启动、API 对齐、角色-技能挂载也已经补到可施工层
- 一旦这套方法论在研发场景跑顺，再复制到内容、自媒体、AI 短剧等场景

## 仍然存在的缺口
虽然骨架已齐，但离“真正产品化”还差几块：
- 更多 bugfix / hotfix 变体任务样例
- `task-schema` 的 JSON Schema 化
- 多场景复用时的字段裁剪策略
- 更完整的角色模板 / workflow 模板商品化组织方式
- 前端状态模型与接口错误码的进一步细化

## 下一步建议
按优先级建议继续补：
1. 增补更多 `examples/tasks/` 变体样例
2. 把任务单进一步结构化成可直接被系统消费的 schema
3. 细化前端状态模型与接口错误码
4. 梳理多场景复用时的字段裁剪与模板层级
5. 开始把“第一个软件公司模板”包装成最小启动模板
