# governance/postmortem-template.md

## 目标
定义研发团队版中，复杂 bug、线上事故、hotfix 收口后的统一复盘模板，避免问题处理完就散，导致经验无法沉淀、follow-up 无法落地。

这份模板的重点不是写长篇总结，而是让 Codex / 智能体和人都能快速回答：
- 发生了什么
- 为什么发生
- 是怎么止血或修复的
- 还有哪些债没还
- 下次怎么减少再发生概率

---

## 适用范围
适用于：
- hotfix 结束后的事故复盘
- 复杂 bug 修复后的过程复盘
- 影响较大、返工较多、暴露系统性问题的任务

通常在以下情况建议补写：
- `postmortem_needed = true`
- 问题影响了生产用户或核心业务路径
- 问题暴露出流程缺口、监控缺口或设计缺口
- 本次只做了临时止血，后续还要治理

---

## 使用原则
- 先讲事实，再讲判断
- 区分“止血完成”和“根因解决”
- 不写情绪化归因
- 不把复盘写成甩锅材料
- 每条 follow-up 都尽量落到明确责任和动作

---

## 标准复盘模板

```yaml
postmortem_id: PM-001
task_id: HOTFIX-001
title: 支付回调重复入账事故复盘
severity: P1
status: Draft
owner: Ops / Release Agent
participants:
  - CEO / 用户
  - Tech Lead Agent
  - Dev Agent
  - QA / Review Agent

summary:
  incident_type: hotfix
  happened_at: 2026-03-28T09:15:00+08:00
  detected_at: 2026-03-28T09:18:00+08:00
  mitigated_at: 2026-03-28T10:02:00+08:00
  resolved_at: TBD
  impact_scope: 部分支付成功订单出现重复入账和状态异常
  user_impact: 影响财务对账和部分用户履约判断
  current_state: 已完成止血，根因治理待后续任务处理

what_happened:
  - 09:15 监控发现支付回调重复处理数量异常上升
  - 09:18 财务确认部分订单出现重复入账
  - 09:26 Tech Lead 判断问题与幂等保护缺失有关
  - 09:35 CEO 批准最小止血方案
  - 10:02 hotfix 上线并确认新增重复处理被阻断

root_cause:
  direct_cause:
    - 支付回调处理链路缺少足够严格的幂等校验
  contributing_factors:
    - 第三方回调重试行为未被完整纳入设计假设
    - 监控对重复处理异常缺乏更早预警阈值
    - 历史代码中订单状态机约束不够集中
  unknowns:
    - 是否还存在其他重复处理入口待继续排查

mitigation_and_fix:
  immediate_actions:
    - 增加支付回调幂等校验
    - 阻断同一支付单号的重复入账路径
  temporary_workarounds:
    - 暂未处理历史异常订单，仅先止血
  permanent_fix_needed: true

what_went_well:
  - 监控较快发现异常趋势
  - 决策链较短，拍板速度可接受
  - hotfix 后新增异常被及时阻断

what_went_wrong:
  - 回调幂等保护设计不足
  - 事故前缺少针对重复处理的专项预警
  - hotfix 前对历史脏数据处理策略没有统一模板

follow_up_actions:
  - title: 补历史异常订单修复脚本与核对流程
    owner: Dev Agent
    due_at: TBD
    priority: high
  - title: 建立支付重复处理告警与监控看板
    owner: Tech Lead Agent
    due_at: TBD
    priority: high
  - title: 梳理支付状态机并补设计说明
    owner: Tech Lead Agent
    due_at: TBD
    priority: medium
  - title: 形成支付类 hotfix 对外同步模板
    owner: Ops / Release Agent
    due_at: TBD
    priority: medium

final_assessment:
  recurrence_risk: medium
  process_gap: true
  monitoring_gap: true
  documentation_gap: true
  close_condition: 新增异常已阻断，follow-up 已建单并明确责任人
```

---

## 必答复盘问题
如果不想一上来写完整 YAML，至少要回答这 8 个问题：
1. 发生了什么？
2. 影响了谁、影响多大？
3. 最早怎么发现的？
4. 直接原因是什么？
5. 这次做的是止血还是根因修复？
6. 还剩哪些未解决问题？
7. 后续任务分别归谁？
8. 下次怎样更早发现 / 更少再犯？

---

## Codex 使用要求
当 Codex 负责输出 postmortem 时，必须做到：
- 不把猜测写成事实
- 区分 `direct_cause` 和 `contributing_factors`
- follow-up 不能只写“后续优化一下”
- 如果根因仍未完全确认，必须明确写出 `unknowns`
- 如果只是 hotfix 止血，不能包装成“问题已彻底解决”
