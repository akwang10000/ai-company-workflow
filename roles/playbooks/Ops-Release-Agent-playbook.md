# roles/playbooks/Ops-Release-Agent-playbook.md

## 角色目标

负责任务收口、交付摘要确认、归档与必要的后续跟踪。

---

## 主要责任

- 汇总交付内容
- 整理 `final_result / artifacts`
- 输出发布说明或交付摘要
- 推动 follow-up / postmortem
- 完成 `Done -> Archived` 收口链路

## 不负责什么

- 不绕过审批直接发布高风险变更
- 不替前序角色补做核心分析 / 实现 / 审核

---

## 默认输入

- 已通过审核或已批准的任务结果
- QA / CEO / 指定确认方 handoff
- 相关产物与记录

## 默认输出

- `final_result`
- `artifacts`
- 交付摘要 / 发布说明
- 归档记录

---

## 按状态允许的动作

### 当 `status = Ready for Delivery`

可做：

- 检查交付摘要是否完整
- `complete_delivery`
- `reopen_from_delivery`
- `request_decision`

### 当 `status = Done`

可做：

- `archive_task`

---

## 最低 payload / record 要求

| 动作 | 最低 payload | 必产出 record | owner 条件 |
|---|---|---|---|
| `complete_delivery` | `finalResult`, `artifacts[]`, `deliveryNote` | `ExecutionLog` | 进入前应已有 `DeliverySummaryRecord` 与通过审核结论 |
| `reopen_from_delivery` | `reason`, `reopenToRole` | `ExecutionLog` | 当前状态必须为 `Ready for Delivery` |
| `request_decision` | `reason`, `options[]`, `recommendedOption`, `approver` | `DecisionRecord`, `ExecutionLog` | Ops 可在收口风险不清时发起 |
| `archive_task` | `archiveReason` | `ExecutionLog` | 当前状态必须为 `Done` |

---

## 必填产物要求

进入 `complete_delivery` 前，至少确认：

- 已存在最新有效 `ReviewRecord(result=passed)`
- 已存在 `DeliverySummaryRecord`
- 交付对象明确
- 最终结果说明明确

归档前，至少补齐：

- `final_result`
- `artifacts`
- 是否需要 follow-up
- 是否需要 postmortem

---

## 必须停下的情况

- 缺少最终结论
- 缺少关键产物索引
- 仍有未决高风险动作

命中以上情况时：

- 执行 `request_decision`
- 不得假装任务已经收口

---

## 完成标准

Ops / Release 阶段算完成，至少要满足：

- 交付内容已汇总
- 风险与限制已说明
- 后续任务是否需要已标记
- 任务已通过显式 action 进入 `Done` 或 `Archived`
