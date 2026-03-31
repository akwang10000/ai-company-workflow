# AGENTS.md

## 目的

本文件定义本项目在 **Phase 1：软件公司模板 / 研发团队版** 下的项目级执行硬规则。

它只回答 4 类问题：

- 什么任务可以进入正式执行
- 执行时必须优先相信哪些文档
- 什么情况下必须停止并进入 `Waiting Decision`
- 什么样的推进才算有效推进

本文件不是：

- README
- 产品总纲
- workflow 细节文档
- role 动作手册

---

## 当前固定范围

当前只做 **Phase 1 MVP**，目标是先把研发任务主链路跑通，而不是扩成通用 AI 公司操作系统。

当前主链路至少包括：

- 模板启动
- 任务创建与查看
- 非流转更新
- action-driven task transition
- handoff
- review
- `Ready for Delivery`
- decision / resume

当前固定任务类型只有：

- `feature`
- `bugfix`
- `hotfix`

当前固定场景只有：

- `engineering`

以下内容不属于当前 Phase 1 主执行范围：

- `research`
- `refactor`
- 多行业模板扩展
- 通用高级流程编辑器
- 高自由度 owner 改派系统

---

## 当前实现阶段的唯一主入口

当前仓库已经进入“**主真相冻结、按 phase 实现验证**”阶段。

新会话 / 新代理恢复执行时，默认按以下顺序读取：

1. `tasks/IMPLEMENTATION-PROGRESS.md`
2. `AGENTS.md`
3. `product/domain-model.md`
4. `product/task-status-guards.md`
5. `product/task-transition-api-and-actions.md`
6. `product/api-contracts.md`
7. `product/implementation-phases.md`
8. `governance/task-schema.md`
9. `governance/decision-gates.md`
10. 对应 `workflows/playbooks/*.md`
11. 对应 `roles/playbooks/*.md`
12. `governance/ready-for-delivery-checklist.md`
13. 当前任务单 / 任务文档 / 样例

### 为什么这样排

- `IMPLEMENTATION-PROGRESS.md` 是当前动态进度真相
- product 主文档是当前实现阶段的业务真相入口
- governance 文档负责治理边界，不替代运行时真相
- workflow / role 文档负责把主真相落到执行手册，不再反向定义主模型

---

## 文档冲突处理规则

若文档冲突，按以下优先级处理：

1. **当前真实进度**：`tasks/IMPLEMENTATION-PROGRESS.md`
2. **项目级硬规则与 Phase 1 范围**：`AGENTS.md`
3. **运行时对象、状态、动作、接口真相**：
   - `product/domain-model.md`
   - `product/task-status-guards.md`
   - `product/task-transition-api-and-actions.md`
   - `product/api-contracts.md`
4. **任务单填写与治理字段**：`governance/task-schema.md`
5. **必须停下的情形**：`governance/decision-gates.md`
6. **具体 workflow 执行顺序**：对应 `workflows/playbooks/*.md`
7. **具体角色动作与交接要求**：对应 `roles/playbooks/*.md`

### 关键约束

- 不允许用 workflow / role 文档反向重定义状态机与 transition 主真相
- 不允许把治理字段文档当作运行时 PATCH 契约
- 不允许重新引入第二套阅读入口

---

## 执行准入规则

满足以下条件，任务才允许进入正式执行：

- 有任务单或等价任务草稿
- `title` 已明确
- `goal` 已明确
- `task_type` 已明确且属于 `feature / bugfix / hotfix`
- `status` 已明确
- `current_owner` 已明确
- `acceptance_criteria` 已明确

若缺失以上关键字段：

- 只能停留在补任务单阶段
- 不得假装进入开发 / 修复 / 审核 / 交付阶段

---

## 项目级硬规则

### 1. 不直接改业务状态

前端与代理都不应把“直接改 status”当成正式推进方式。

正式推进必须通过 action-driven transition 完成。

### 2. owner 变化不由普通 PATCH 驱动

`currentOwner` / `nextOwner` 的业务变化必须来自 transition side effect。

### 3. 每次有效推进都必须可追踪

每次有效推进至少要落下这些信息：

- 当前状态
- 当前负责人
- 做了什么
- 产生了什么记录
- 下一步交给谁

### 4. 每次角色切换都必须有 handoff 信息

handoff 至少要写清：

- 已完成什么
- 未完成什么
- 已知风险
- 下一角色先看什么
- 交付了哪些产物

### 5. 每轮有效实施必须更新进度真相文件

每轮有效实施后，必须同步更新：

- `tasks/IMPLEMENTATION-PROGRESS.md`

若未更新，不算完成本轮交付。

### 6. 不得脑补缺失关键信息

文档不完整时：

- 可以标记缺口
- 可以进入 `Waiting Decision`
- 不得把猜测写成确定结论

### 7. 文档编码统一 UTF-8

所有文档、脚本、代码文件统一使用 UTF-8。

---

## 必须停止并进入 `Waiting Decision` 的情况

出现以下任一情况，必须停下：

- 目标不明确
- 范围互相冲突
- 多方案并存且缺少决策标准
- 审核未通过但有人想强推
- 涉及外部影响
- 涉及高风险操作
- 缺少关键上下文或权限仍想继续推进
- hotfix 涉及回滚、降级、生产数据修复、正式对外说明

进入 `Waiting Decision` 时，必须同时输出：

- 当前结论
- 为什么不能继续自动推进
- 可选方案
- 风险差异
- 推荐方案
- 等待谁决策

---

## 当前阶段的成功标准

当前阶段不是追求“全自动代理”，而是追求：

- 能按模板启动 workspace
- 能创建并查看任务
- 能按状态机推进 task transition
- 能正确形成 handoff / decision / review / delivery summary 记录
- 能让执行代理按统一入口少猜测施工

一句话判断标准：

> 如果当前文档和任务状态，还不能让代理明确知道“现在该做什么、做完产出什么、交给谁、什么情况下必须停”，就不能假装已经具备正式执行能力。
