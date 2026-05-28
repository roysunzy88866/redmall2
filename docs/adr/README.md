# Architecture Decision Records (ADR)

本目录记录每个**重要架构 / 工程决策**,一个决策一份文件。

> 灵感来源:[Michael Nygard "Documenting Architecture Decisions" (2011)](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
> 行业实践:[ThoughtWorks Tech Radar - Lightweight ADRs (Adopt)](https://www.thoughtworks.com/radar/techniques/lightweight-architecture-decision-records)

---

## 📋 索引

| # | 标题 | 状态 | 日期 |
|---|---|---|---|
| [ADR-0001](0001-engineering-discipline.md) | 工程纪律与交付节奏 | Accepted | 2026-05-28 |
| [ADR-0002](0002-lean-domain-layer.md) | 精简领域层(Lean Domain) | Accepted | 2026-05-28 |
| [ADR-0003](0003-flat-layout-and-domain-naming.md) | 后端 flat 布局 + `domain/` 命名 | Accepted | 2026-05-28 |

---

## 🆕 何时新建 ADR

参考 [CLAUDE.md §何时必须写文档](../../CLAUDE.md):
- 技术 / 架构决策被确认
- 引入新依赖(pip / gradle 包)
- 共识灰色地带被拍板
- 推翻过往决策(标记旧 ADR `Superseded by ADR-NNNN`)

**口诀**:如果有人 3 个月后问"我们当初为什么这么做",而代码里看不出答案,那就该有一份 ADR。

---

## 📝 命名 + 编号约定

- 文件名:`NNNN-kebab-case-title.md`(NNNN 四位递增,从 0001 起)
- 标题:简短的决策名,不超过 7 个字
- 编号一旦分配**永不重用**,即使该 ADR 被 Superseded

---

## 📐 模板

复制下方模板到新 ADR 文件:

```markdown
# ADR-NNNN · <决策标题>

- **Status**: Proposed / Accepted / Superseded by ADR-XXXX / Deprecated
- **Date**: YYYY-MM-DD
- **Tags**: architecture / backend / android / deployment / process / ...

## 背景 Context
为什么要做这个决策?遇到了什么问题?当时已知的约束有哪些?

## 决策 Decision
最终选定的方案,一两句话讲清楚。

## 理由 Rationale
为什么选它?关键判断依据是什么?

## 替代方案 Alternatives Considered
对比过哪些方案?各自的优劣?为什么没选?
(这一节防止未来回头看时觉得"当初为什么不直接用 X")

## 后果 Consequences
- ✅ 正面影响:
- ⚠️ 负面影响 / 代价:
- 📎 后续需要做的事 / 关联文档:
```

---

## 🔄 修订 / 推翻一份 ADR

- **不删除**老 ADR。把它的 Status 改为 `Superseded by ADR-NNNN`
- 在新 ADR 顶部加 `Supersedes ADR-NNNN`
- 在本索引上标注老 ADR 的 Status 变化
