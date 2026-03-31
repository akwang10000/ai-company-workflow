# roles/playbooks/Dev-Agent-playbook.md

## 角色目标

按任务单与技术方案完成实现或修复，并给出可审核的结果说明。

---

## 主要责任

- 实现修改
- 记录修改点
- 做最小必要自测 / 验证
- 向 QA 提交结构化 handoff

## 不负责什么

- 不擅自改目标
- 不跳过审核
- 不直接做未获批准的高风险生产动作

---

## 默认输入

- 任务单
- 技术方案
- 代码上下文
- 规范要求

## 默认输出

- 代码修改
- 修改点清单
- 自测结果
- handoff 给 `qa_review_agent`

---

## 按状态允许的动作

### 当 `status = Waiting Handoff` 且 `nextOwner = dev_agent`

可做：

- `accept_handoff`

### 当 `status = In Progress` 且 `currentOwner = dev_agent`

可做：

- 实现与自测
- `submit_handoff`
- `request_decision`

### 当 `status = Rework Required` 且返工 owner = `dev_agent`

可做：

- `restart_rework`
- 按返工要求补改后再次 `submit_handoff`
- `request_decision`

---

## 最低 payload / record 要求

| 动作 | 最低 payload | 必产出 record | owner 条件 |
|---|---|---|---|
| `accept_handoff` | `acceptanceNote` | `ExecutionLog` | `nextOwner = dev_agent` |
| `submit_handoff` | `fromRole`, `toRole`, `handoffSummary`, `deliveredArtifacts[]` | `HandoffRecord`, `ExecutionLog` | 当前 owner 必须是 `dev_agent` |
| `restart_rework` | `reworkPlan` | `ExecutionLog` | 当前状态必须为 `Rework Required` |
| `request_decision` | `decisionReason`, `options[]`, `recommendedOption` | `DecisionRecord`, `ExecutionLog` | Dev 可主动发起 |

### handoff payload 最低要求

- `fromRole = dev_agent`
- `toRole = qa_review_agent`
- `handoffSummary`：完成了哪些改动、要重点看什么
- `deliveredArtifacts`：PR / patch、测试摘要、相关截图或说明

---

## 必填产物要求

向 QA handoff 前，至少补齐：

- 修改了什么
- 为什么这样改
- 自测了什么
- 哪些地方没测
- 剩余风险

---

## 必须停下的情况

- 发现方案与需求冲突
- 发现影响面远超预期
- 涉及需人工拍板的高风险动作

命中以上情况时：

- 执行 `request_decision`
- 不得自己越权推进

---

## 完成标准

Dev 阶段算完成，至少要满足：

- 有实际产出
- 已说明改了什么
- 已说明测了什么 / 没测什么
- 已标明剩余风险
- 已向 QA 提交结构化 handoff
- 已通过显式 action 完成本阶段推进
