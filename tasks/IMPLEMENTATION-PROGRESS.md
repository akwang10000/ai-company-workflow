# tasks/IMPLEMENTATION-PROGRESS.md

## 目标

作为当前仓库内 **唯一的动态实施进度真相文件**，用于让 Codex / 智能体在：

- 长时间连续施工后
- 会话中断后
- 新开会话后
- 更换执行代理后

仍能基于仓库内落盘信息恢复当前进度，而不是重新猜测“做到哪了”。

本文件只负责：

- 当前做到哪个 phase / subphase
- 最近完成了什么
- 当前正在做什么
- 当前阻塞与未决问题是什么
- 下一步应该继续做什么
- 新会话恢复时应优先读什么

它不替代：

- `product/implementation-phases.md` 的静态 phase 蓝图职责
- `product/domain-model.md` 等产品主真相职责
- 具体任务单的业务事实职责

---

## 使用规则

- 这份文档是 **动态进度文档**，不是静态设计文档
- 每完成一轮有效实施，必须同步更新本文件
- 若未更新本文件，不算完成本轮交付
- 新会话恢复前，应先读取本文件，再决定是否深入其它文档
- 本文件只记录真实进度，不记录设想中的长期大计划

---

## Current Phase

- Phase: `Phase 0 / 文档收口与施工基线期`
- Subphase: `P0.3 / 包外残留清理与执行手册加硬`
- Status: `verified`

---

## Last Completed

- 最近完成：
  - 统一 README / docs-map / screens / canvas / workflow overview 的当前 Phase 1 口径
  - 统一首个主链路动作名为 `ready_task`
  - 清理补充文档中的旧状态名 `Review / Rework / In Analysis / Queued`
  - 将 workflow / role 文档继续补到 action / payload / record / owner 条件可直接消费的粒度
- 最近一次验证：
  - 主入口文档与补充文档的动作名、状态名、阅读入口已对齐到同一套 Phase 1 口径
- 对应 changed files：
  - `README.md`
  - `docs-map.md`
  - `product/screens-and-flows.md`
  - `product/canvas-ui-spec.md`
  - `workflows/workflow.md`
  - `examples/tasks/*.md`
  - `workflows/playbooks/*.md`
  - `roles/playbooks/Tech-Lead-Agent-playbook.md`
  - `roles/playbooks/Dev-Agent-playbook.md`
  - `roles/playbooks/Ops-Release-Agent-playbook.md`
  - `tasks/IMPLEMENTATION-PROGRESS.md`

---

## In Progress

- 当前正在做：
  - 准备进入真实实现验证前的最终文档对齐
- 当前目标：
  - 让代理在不重新猜系统的前提下，按统一阅读入口直接开工
- 当前涉及文件：
  - 主入口真相层文档
  - workflow / role 执行手册
  - 页面与画布补充说明文档

---

## Blockers / Open Questions

- 当前阻塞：
  - `none`
- 当前未决问题：
  - `none`
- 是否等待人工决策：
  - `no`

---

## Next Step

- 下一步最该做：
  - 进入真实实现验证，优先打通 3 条验收链路
- 为什么是这一步：
  - 当前主真相层已基本冻结，继续扩写总纲收益低于实现验证
- 明确不在本轮做的事：
  - 不扩多行业模板
  - 不扩 `research` / `refactor` 任务类型
  - 不新增高级 owner 改派体系

---

## Current Working Slice

- 当前正在验证的链路：
- 当前实施任务 / 子任务：
- 当前负责角色：
- 当前目标结果：

---

## Files In Flight

- 本轮正在改的文件：
- 已改未验证：

建议优先验证的验收链路：

1. 模板启动链路
2. `submit_handoff -> accept_handoff -> start_review -> mark_ready_for_delivery`
3. `request_decision -> resolve decision -> resume_after_decision`

### 重要说明

上面的 3 条是 **优先验证的验收链路**，不表示可以跳过 `product/implementation-phases.md` 定义的 phase 顺序直接施工。

---

## Resume Instructions

新会话 / 新代理恢复执行前，默认按以下顺序读取：

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
13. 当前任务单 / 当前改动上下文

若 `Status = blocked`：

- 先处理阻塞或等待决策
- 不要直接跳到下一 phase

若 `Status = verified`：

- 优先按 `Next Step` 继续推进
- 不要重新发明实施顺序

---

## Update Template

每轮更新本文件时，至少同步更新：

- `Current Phase`
- `Last Completed`
- `In Progress`
- `Blockers / Open Questions`
- `Next Step`
- `Resume Instructions`（若阅读入口发生变化）
