# Tech Debt · 技术债

> 共识内但代码质量 / 机制妥协的登记。格式见 [CLAUDE.md](../CLAUDE.md)。
> 区别:范围越界记 [drift-log.md](drift-log.md);这里记"范围内但打折"。
> 🔴 安全类不允许 `accepted`,演示完必须闭合。
> 每个里程碑末盘点;open 超 10 条 → 强制停手还债。

---

## DEBT-2026-05-001 · 后台固定明文密码 🔴
- 描述:后台密码固定 `Ridemall@Demo2026`,公网弱口令
- 触发原因:用户决定演示简化(共识 §3 A1/A2)
- 影响:安全
- 偿还计划:演示完 `launchctl unload` 下线服务(共识 §5 验收 6)
- 状态:open

## DEBT-2026-05-002 · ORM ↔ domain ↔ DTO 三层映射无机械保护
- 描述:加/改一个字段需同步 ~8 处(ORM / domain entity / 请求 dataclass / repository mapper / api.md / Android DTO / model / mapper),无 hook 抓漏
- 触发原因:lean domain 架构(ADR-0002)的固有代价
- 影响:可维护性;**预测 C4(商品 + 上传)首次出现字段漂移**
- 偿还计划:**C4 开工前重新决断** —— 撤回 ADR-0002 改"更 lean"(ORM 直接当 entity),或加映射一致性测试
- 状态:open

## DEBT-2026-05-003 · 前后端 API 契约三向同步纯人工
- 描述:后端 endpoint 路径 ↔ Android `ApiService.kt` ↔ `docs/api.md`,三处人工保持一致
- 触发原因:demo 不引入 OpenAPI codegen
- 影响:正确性;**预测 C8(Android 接通后端)起出现**
- 偿还计划:若 M4 痛,引入最小 OpenAPI schema 或契约测试
- 状态:open

## DEBT-2026-05-004 · 覆盖率仪表盘人工填写
- 描述:`test-plan.md` 覆盖率表手填,无自动提取
- 触发原因:C1 前 pytest-cov 未接
- 偿还计划:C1 接 `pytest-cov` 后改为"命令实时查",表格只留说明
- 状态:open

## DEBT-2026-05-005 · 文档修订记录无全仓快照语义 🟢
- 描述:各文档 v 号独立,无统一"仓库快照版本";精确还原某时刻全套一致版本要靠 git
- 偿还计划:每个里程碑末打 git tag(如 `m1-done`)
- 状态:open

## DEBT-2026-05-006 · 口头偏好缺登记位(低于 ADR 门槛)
- 描述:用户实施级口头偏好(如"这个 endpoint 返回 201 不要 200")不够格开 ADR,但 git log 不接、共识不接 → 易丢
- 触发原因:决策分级有缝隙
- 偿还计划:暂以 commit message body + `docs/wip.md` 临时承载;若反复丢失再设专门登记
- 状态:open
