# WIP · 跨会话半成品记忆

> 用途:`git stash` 或会话中断时,在此写一行人话,让**下次新会话冷启动**能接上。
> 规则:
> - stash / 中断前:加一行 `时间 — 状态(TDD 红/绿/重构哪步)— 下一步`
> - 切片 `done` 时:**清空本文件**(只留本说明 + 下方分隔线);pre-commit 在 `[C_n done]` 时校验本文件无遗留条目
> - 新会话自启动 SOP 第一步之一:`cat docs/wip.md`

---

<!-- 在此线下方写 WIP 条目;无条目 = 干净,可正常开新切片 -->

<!-- 干净:2026-05-29 车机端 UI/IA 原型已定稿并全部落盘
     (prototype-wireframes.md + ADR-0004 + 需求共识.md + user-stories.md US-CAR-01~07 + story-map.md)。
     无半成品遗留,可正常开新切片。 -->

<!-- 阻塞:2026-05-29 用户暂缓 C1 开工,等 Claude Design 出 hi-fi 设计稿。
     交付形式:用户会发链接(待定)。
     我"确认设计稿完毕" = 对照 US-CAR-01~07 七个故事,每个都有对应设计页;
       核对完结果落 docs/drift-log.md(若有冲突)或 ADR-0004 追加"设计稿对照表"。
     在用户明确说"设计稿好了"之前,不动 C1、不动任何代码。 -->

<!-- 已解除:2026-05-29 设计稿核对完成 + 3 处不一致已裁决 + 落档完毕:
     - design_handoff_ridemall_carstore/ 已交付,7 故事 100% 覆盖
     - D-001/D-002/D-003 已 closed(见 drift-log.md)
     - ADR-0004 追加「设计稿对照表」 amendment(7 故事映射 + 视觉决策锁定)
     - user-stories.md US-CAR-05 倒计时 03:00 → 02:59
     当前状态:干净,可开 C1(US-SYS-01 健康检查 endpoint)。等用户说"开 C1"。 -->
