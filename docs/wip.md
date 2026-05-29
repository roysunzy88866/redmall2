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
