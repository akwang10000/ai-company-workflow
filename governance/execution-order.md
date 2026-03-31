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
- 涉及 bug / incident / hotfix 时，先补严重度与优先级判断
- 进入收口前，必须补回归与交付检查
- 文档阅读顺序必须服务执行，不服务“看起来完整”

---

## 默认执行顺序（总览）
当 Codex 接到一个任务时，默认按以下顺序处理：
1. `AGENTS.md`
2. `governance/task-schema.md`
3. `governance/decision-gates.md`
4. 若为 bug / incident / hotfix，读取 `governance/severity-priority-rules.md`
5. 判断任务类型与场景
6. 进入对应 `workflows/playbooks/*.md`
7. 根据当前角色进入对应 `roles/playbooks/*.md`
8. 发生角色切换时参考 `governance/handoff-templates.md`
9. 进入收口前读取 `governance/regression-checklist.md`
10. 准备交付前读取 `governance/ready-for-delivery-checklist.md`
11. 若任务为复杂 bug / hotfix，收尾时再看 `governance/postmortem-template.md`

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

## 第 1.5 步：先恢复当前实施进度
### 必读文档
- `tasks/IMPLEMENTATION-PROGRESS.md`

### 目的
确认当前真实进度已经做到哪，避免新会话 / 新代理重新猜测：
- 当前 phase / subphase
- 最近完成项
- 当前正在做什么
- 当前阻塞 / 未决问题
- 下一步最该做什么

### 规则
- 若进度文件未更新到最新状态，应先补齐，再继续执行
- 若进度文件显示 `blocked`，先处理阻塞或等待决策，不得假装继续推进
- 若进度文件与实际代码 / 文档状态冲突，必须先收口进度真相

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

## 第 4 步：若为 bug / incident / hotfix，先判断严重度与优先级
### 必读文档
- `governance/severity-priority-rules.md`

### 目的
先统一口径：
- 问题有多严重
- 现在有多急
- 应走 `bugfix` 还是 `hotfix`

### 要做什么
- 先判断 `severity`
- 再判断 `priority`
- 再决定是否升级为 `hotfix`
- 若涉及回滚、生产修复、对外影响，则同步检查是否进入 `Waiting Decision`

### 注意
不要把：
- 老板催得急
- 今天想上线
- 看起来麻烦

直接等价成高严重度。

---

## 第 5 步：识别任务所属 workflow
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

## 第 6 步：识别当前角色并读取角色手册
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

## 第 7 步：按角色执行当前阶段动作
### 要做什么
在 workflow 的阶段约束下，按 role playbook 执行当前职责。

### 关键要求
- 输出要对齐任务单字段
- 状态变更必须明确
- 不能跳步骤声称完成

---

## 第 8 步：发生交接时读取交接模板
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

## 第 9 步：进入收口前先做回归检查
### 必读文档
- `governance/regression-checklist.md`

### 适用时机
- bugfix 准备提审
- hotfix 准备发布
- feature 影响已有链路时
- 任何高风险改动准备收口时

### 要做什么
- 明确修复点是否验证
- 明确上下游是否抽查
- 明确已验证 / 未验证范围
- 明确剩余风险

### 规则
如果连最小回归记录都没有：
- 不得假装进入 `Ready for Delivery`

---

## 第 10 步：准备交付前做 Ready 检查
### 必读文档
- `governance/ready-for-delivery-checklist.md`

### 适用时机
- 准备把状态切到 `Ready for Delivery`
- 准备交给 CEO / 用户 / Approver 做最终确认

### 要做什么
- 核对交付物是否齐
- 核对验收与回归摘要是否齐
- 核对风险与未完成项是否说明白
- 核对下一接收方是否明确

### 规则
不能只因为“开发做完了”就切到 `Ready for Delivery`。

---

## 第 11 步：进入审核 / 决策 / 收口分支
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

### 复盘阶段
若为复杂 bug / hotfix，或事故级问题：
- 参考 `governance/postmortem-template.md`
- 补 postmortem 与 follow-up

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
8. 若影响旧链路，读 `governance/regression-checklist.md`
9. 进入交付前，读 `governance/ready-for-delivery-checklist.md`
10. 反复推进，直到 `Done / Archived`

### 当前研发 bugfix 任务
1. 读 `AGENTS.md`
2. 读 `governance/task-schema.md`
3. 读 `governance/decision-gates.md`
4. 读 `governance/severity-priority-rules.md`
5. 读 `workflows/playbooks/bugfix-workflow.md`
6. 根据当前负责人读对应 role playbook
7. 做当前阶段动作
8. 用 `governance/handoff-templates.md` 交给下一角色
9. 用 `governance/regression-checklist.md` 完成回归记录
10. 用 `governance/ready-for-delivery-checklist.md` 做交付前检查
11. 完成收口

### 当前研发 hotfix 任务
1. 读 `AGENTS.md`
2. 读 `governance/decision-gates.md`
3. 读 `governance/severity-priority-rules.md`
4. 读 `workflows/playbooks/hotfix-workflow.md`
5. 先确认是否需要人工拍板
6. 根据当前负责人读对应 role playbook
7. 压缩流程但保留记录
8. 用 `governance/regression-checklist.md` 留下最小回归记录
9. 用 `governance/ready-for-delivery-checklist.md` 做发布前收口
10. 完成止血后创建 follow-up 任务或归档

---

## Codex 禁止事项
在阅读顺序上，禁止以下行为：
- 不读全局规则就直接干活
- 不看任务 schema 就自己脑补字段
- 不看 severity / priority 规则就草率判断 hotfix
- 不看 workflow 就跳到某一个角色手册
- 不看 decision gate 就越权推进
- 不做回归记录就宣称可交付
- 不写交接说明就切角色
