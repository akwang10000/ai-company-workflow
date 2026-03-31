# governance/ready-for-delivery-checklist.md

## 目标

定义任务进入 `Ready for Delivery` 前必须满足的最小收口条件，避免“东西改完了就算可交付”。

---

## 核心原则

### 1. `Ready for Delivery` 不等于 `Done`

`Ready for Delivery` 表示：

- 审核已通过
- 交付摘要已形成
- 当前已经具备交付确认条件

它不等于：

- 已完成最终业务确认
- 已完成归档

### 2. 没有收口说明，不算 Ready

不能只改完东西就把状态切到 `Ready for Delivery`。

至少要能回答：

- 做了什么
- 怎么验证的
- 还有什么没做
- 有什么风险
- 交给谁确认

### 3. 风险可带着走，但不能藏着走

允许带着已知风险进入 `Ready for Delivery`，但必须满足：

- 风险已被记录
- 风险未超出允许范围
- 最终确认人知道并接受

### 4. 缺少关键验证时，不能硬进 Ready

若缺少以下任一项，则不得进入 `Ready for Delivery`：

- 基本验收结果
- 基本回归记录
- 当前版本产物说明
- 明确的交付对象

---

## 进入 `Ready for Delivery` 的最小检查清单

### A. 任务定义完整性

- [ ] 任务单存在且可读
- [ ] `title / goal / status / current_owner / acceptance_criteria` 已补齐
- [ ] 当前任务范围没有明显漂移或未决冲突
- [ ] 当前任务不处于 `Waiting Decision`

### B. 审核前置条件

- [ ] 已存在最新有效 `ReviewRecord(result=passed)`
- [ ] 审核结论与验收标准可对应
- [ ] 若存在已知限制，已显式写出

### C. 交付摘要前置条件

- [ ] 已存在 `DeliverySummaryRecord`
- [ ] `changeSummary` 已填写
- [ ] `affectedScope` 已填写
- [ ] `validationSummary` 已填写
- [ ] `remainingRisks` 已填写；若无风险，也已显式写 `none`

### D. 交接与确认前置条件

- [ ] 已明确交付对象 / 最终确认方
- [ ] 已明确当前任务是“待确认交付”，而不是“已完成归档”

---

## 按任务类型补充检查

### feature

至少补充：

- [ ] 关键功能路径已验证
- [ ] 非范围内容已说明未做

### bugfix

至少补充：

- [ ] 已说明复现条件是否消失
- [ ] 已说明是否做了相关回归

### hotfix

至少补充：

- [ ] 已明确当前是临时止血还是完整修复
- [ ] 已说明是否需要 follow-up
- [ ] 已说明剩余线上风险

---

## Delivery Summary 推荐模板

```yaml
delivery_summary:
  changeSummary: |
    完成了什么改动。
  affectedScope:
    - 页面 / 模块 / 接口 / 配置项
  validationSummary: |
    做了哪些验证，哪些未验证。
  remainingRisks: |
    若无显著风险，显式写 none。
  deliveryTo:
    - CEO / 用户 / 指定确认方
```

---

## 当前阶段统一结论

进入 `Ready for Delivery` 的最小标准不是“做完了”，而是：

- 审核通过
- 交付摘要齐全
- 风险显式可见
- 已具备明确交付确认对象
