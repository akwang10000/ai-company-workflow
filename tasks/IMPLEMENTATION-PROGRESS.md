# tasks/IMPLEMENTATION-PROGRESS.md

## 目标
作为当前仓库内 **唯一的实施进度真相文件**，用于让 Codex / 智能体在：
- 长时间连续施工后
- 会话中断后
- 新开会话后
- 更换执行代理后

仍然能够基于仓库内落盘信息，快速恢复当前实施状态，而不是重新猜测“做到哪了”。

这份文档只负责：
- 当前做到哪个 phase / 子阶段
- 最近完成了什么
- 当前正在做什么
- 当前阻塞与未决问题是什么
- 下一步应该继续做什么
- 新会话恢复时优先读什么

它不替代：
- `product/implementation-phases.md` 的静态 phase 蓝图职责
- `product/codex-delivery-rules.md` 的交付规则职责
- 具体任务单 / API / 状态机文档的真相职责

---

## 使用规则
- 这份文档是 **动态进度文档**，不是静态设计文档
- Codex / 智能体每完成一轮有效实施，必须同步更新本文件
- 若未更新本文件，不算完成本轮交付
- 新会话恢复前，应优先读取本文件，再决定是否继续深入其它文档
- 本文件只记录当前真实进度，不记录设想中的未来大计划

---

## Current Phase
- Phase: `TBD`
- Subphase: `TBD`
- Status: `not_started | in_progress | blocked | verified`

---

## Last Completed
- 最近完成：
  - `TBD`
- 最近一次验证：
  - `TBD`
- 对应 changed files：
  - `TBD`

---

## In Progress
- 当前正在做：
  - `TBD`
- 当前目标：
  - `TBD`
- 当前涉及文件：
  - `TBD`

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
  - `TBD`
- 为什么是这一步：
  - `TBD`
- 明确不在本轮做的事：
  - `TBD`

---

## Resume Instructions
新会话 / 新代理恢复执行前，默认按以下顺序读取：
1. `AGENTS.md`
2. `product/domain-model.md`
3. `product/implementation-phases.md`
4. `product/api-contracts.md`
5. `product/task-transition-api-and-actions.md`
6. `tasks/IMPLEMENTATION-PROGRESS.md`

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
- `Resume Instructions`（若阅读入口已变化）

---

## 当前初始化状态
- Phase: `Phase 0 / 文档收口与施工基线期`
- Subphase: `P0 一致性收口已完成，待进入实现验证`
- Status: `verified`

### 最近完成
- 收窄 `governance/task-schema.md` 的 Phase 1 边界
- 收死 owner 双通道边界
- 收死 `ReviewRecord` append-only 生命周期
- 补充 transition 的 actor / 并发 / 幂等最小规则
- 生成完整可替换文档包

### 当前阻塞
- `none`

### 下一步
- 优先进入真实实现验证，而不是继续扩主文档
- 建议先跑 3 条闭环：
  - 模板启动链路
  - handoff -> review -> delivery 链路
  - decision -> resume 链路
