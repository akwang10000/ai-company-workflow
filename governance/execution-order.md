# governance/execution-order.md

## 目标
定义 Codex / 智能体在本项目中执行任务时，**先读什么、后读什么、在什么条件下切换到哪份文档**。

这份文档解决的问题不是“文档有哪些”，而是：
> 当一个任务真的来了，Codex 应该按什么顺序理解规则、进入角色、执行流程、完成交接。

---

## 核心原则
- 先读全局规则，再读任务定义
- 先判断任务类型，再进入对应 workflow
- 先明确当前角色，再读对应 role playbook
- 先确认是否有决策门，再继续执行
- 文档阅读顺序必须服务执行，不服务“看起来完整”

---

## 默认执行顺序（总览）
当 Codex 接到一个任务时，默认按以下顺序处理：
1. `AGENTS.md`
2. `governance/task-schema.md`
3. `governance/decision-gates.md`
4. 判断任务类型与场景
5. 进入对应 `workflows/playbooks/*.md`
6. 根据当前角色进入对应 `roles/playbooks/*.md`
7. 发生角色切换时参考 `governance/handoff-templates.md`
8. 若任务进入完成态，参考 Ops / Release 收口规则

---

## 第 0 步：先确认当前任务是否存在
### 规则
如果没有任务单，或者至少没有任务草稿，就不要假装自己已经进入执行阶段。

### 检查点
- 是否有 `title`
- 是否有 `goal`
- 是否有 `status`
- 是否知道当前负责人是谁

若缺失过多：
- 先补任务单
- 不进入 workflow

---

## 第 1 步：读取全局规则
### 必读文档
- `AGENTS.md`

### 目的
确认项目级硬规则，例如：
- 什么情况下必须停机
- 什么动作不能自动越权执行
- 文档要写到什么粒度才算可执行

---

## 第 2 步：读取任务结构规则
### 必读文档
- `governance/task-schema.md`

### 目的
确认任务单应包含哪些字段，当前还缺什么。

### 要做什么
- 校验当前任务信息是否足够
- 如果字段不足，优先补齐
- 如果验收标准缺失，不进入开发执行

---

## 第 3 步：读取停机与审批规则
### 必读文档
- `governance/decision-gates.md`

### 目的
防止任务在不该继续时硬往下冲。

### 要做什么
检查当前任务是否触发以下情况：
- 目标不明确
- 多方案冲突
- 涉及高风险动作
- 涉及外部影响
- 审核未通过但还想强推

若命中：
- 进入 `Waiting Decision`

---

## 第 4 步：识别任务所属 workflow
### 要做什么
根据 `task_type` / `scenario` 判断要进入哪条工作流。

### 当前推荐映射
- `scenario = engineering` + `task_type = feature`
  - 进入 `workflows/playbooks/feature-workflow.md`
- `scenario = engineering` + `task_type = bugfix`
  - 进入 `workflows/playbooks/bugfix-workflow.md`
- `scenario = engineering` + `task_type = refactor`
  - 暂时参考 `workflows/playbooks/feature-workflow.md`
- `scenario = engineering` + 问题紧急且有线上影响
  - 改走 `workflows/playbooks/hotfix-workflow.md`

### 注意
如果还没有对应 workflow：
- 先选最接近的 workflow 作为临时骨架
- 同时标记“workflow 尚未专门化”

---

## 第 5 步：识别当前角色并读取角色手册
### 要做什么
根据 `current_owner` 判断当前是谁在执行。

### 当前角色与文档映射
- `PM Agent`
  - `roles/playbooks/PM-Agent-playbook.md`
- `Tech Lead Agent`
  - `roles/playbooks/Tech-Lead-Agent-playbook.md`
- `Dev Agent`
  - `roles/playbooks/Dev-Agent-playbook.md`
- `QA / Review Agent`
  - `roles/playbooks/QA-Review-Agent-playbook.md`
- `Ops / Release Agent`
  - `roles/playbooks/Ops-Release-Agent-playbook.md`

### 规则
如果不知道当前角色是谁，就不要继续执行具体动作。

---

## 第 6 步：按角色执行当前阶段动作
### 要做什么
在 workflow 的阶段约束下，按 role playbook 执行当前职责。

### 关键要求
- 输出要对齐任务单字段
- 状态变更必须明确
- 不能跳步骤声称完成

---

## 第 7 步：发生交接时读取交接模板
### 必读文档
- `governance/handoff-templates.md`

### 适用时机
- 从 PM 交给 Tech Lead
- 从 Tech Lead 交给 Dev
- 从 Dev 交给 QA
- 从 QA 打回 Dev
- 从 QA 交给 CEO / 用户
- 从 CEO / 用户交给 Ops / Release

### 规则
每次角色切换，都必须带交接说明。

---

## 第 8 步：进入审核 / 决策 / 收口分支
### 审核阶段
若状态为 `Review`：
- 优先参考对应 workflow
- 再读 `QA-Review-Agent-playbook.md`

### 决策阶段
若状态为 `Waiting Decision`：
- 参考 `governance/decision-gates.md`
- 输出可选方案、风险差异、推荐意见

### 收口阶段
若状态为 `Ready for Delivery` 或 `Done`：
- 参考 `Ops-Release-Agent-playbook.md`
- 完成交付摘要、风险记录、归档动作

---

## 一条简化执行链
### 当前研发 feature 任务
1. 读 `AGENTS.md`
2. 读 `governance/task-schema.md`
3. 读 `governance/decision-gates.md`
4. 读 `workflows/playbooks/feature-workflow.md`
5. 根据当前负责人读对应 role playbook
6. 做当前阶段动作
7. 用 `governance/handoff-templates.md` 交给下一角色
8. 反复推进，直到 `Done / Archived`

### 当前研发 bugfix 任务
1. 读 `AGENTS.md`
2. 读 `governance/task-schema.md`
3. 读 `governance/decision-gates.md`
4. 读 `workflows/playbooks/bugfix-workflow.md`
5. 根据当前负责人读对应 role playbook
6. 做当前阶段动作
7. 用 `governance/handoff-templates.md` 交给下一角色
8. 完成回归验证后收口

### 当前研发 hotfix 任务
1. 读 `AGENTS.md`
2. 读 `governance/decision-gates.md`
3. 读 `workflows/playbooks/hotfix-workflow.md`
4. 先确认是否需要人工拍板
5. 根据当前负责人读对应 role playbook
6. 压缩流程但保留记录
7. 完成止血后创建 follow-up 任务或归档

---

## Codex 禁止事项
在阅读顺序上，禁止以下行为：
- 不读全局规则就直接干活
- 不看任务 schema 就自己脑补字段
- 不看 workflow 就跳到某一个角色手册
- 不看 decision gate 就越权推进
- 不写交接说明就切角色
