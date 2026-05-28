# ADR-0002 · 精简领域层(Lean Domain)

- **Status**: Accepted
- **Date**: 2026-05-28
- **Tags**: architecture, backend, android

## 背景 Context

用户要求:
- 业务模块清晰,**未来可单一替换**
- 边界清晰,**显式输入输出**
- 浅层入口禁藏业务,优先抽到深层模块

但项目是 demo 体量:14 个用户故事、~44 个场景,业务规则简单(Banner 限 5、qty=1、price > 0、商品下架不可下单等)。

需要在「业务规则集中」与「不过度工程化」之间找平衡。

## Decision

后端引入 **`server/domain/`** 层,采用 **lean** 版本(非完整 DDD):

- `domain/entities.py` — 纯 dataclass:Category / Product / Order / BannerSlot
- `domain/rules.py` — 校验函数:`validate_banner_slots`、`validate_order_request` 等
- `domain/errors.py` — 业务异常:`ProductNotFound` / `BannerLimitExceeded` / `OrderQtyInvalid` 等
- **100% 纯函数 + 数据**,无副作用,可零基础设施单测
- 不引入:aggregates / value objects / domain events / repositories interface in domain

车机端对应:`android/.../data/model/` 充当车机端的 domain 层,与 `data/dto/` **显式分离**。

## Rationale

- 业务规则集中于 `domain/`,**替换 ORM 或 Web 框架时不动业务**(满足"可单一替换")
- 100% 单测覆盖容易达成(纯函数 + 无 IO,可并发跑)
- 单向依赖图**无环**:任何层可 import domain,domain 不 import 任何层
- 跟 [CLAUDE.md §代码硬规矩 1-3](../../CLAUDE.md) 的层级铁律对齐
- "lean" 而非"完整 DDD",避免对简单业务过度抽象

## Alternatives Considered

- **A: 不要 domain 层,业务规则散在 services 里** → 违反"单一模块可替换";换 ORM 要改业务;违反"浅层入口不藏业务"原则在 service 层的对应版本
- **B: 完整 DDD**(aggregates / value objects / domain events) → demo 业务规则太简单,这些抽象是过度工程;新人成本陡增
- **C: 用 SQLAlchemy ORM model 当 domain**(model 上加业务方法) → domain 被迫依赖 SQLAlchemy,失去可替换性;DDD 称此为 anemic domain 反模式
- **D: 业务规则只在 service 函数里,不抽校验** → 同一规则被多次复制(创建 banner 时 + 编辑时都要校验上限),漂移风险高

## Consequences

- ✅ 业务规则集中,可单一替换 ORM / Web 框架
- ✅ 单测易写、易快(无 DB 依赖,可并发跑)
- ✅ 业务异常类型显式,view 层翻译 HTTP 状态码清晰
- ⚠️ DTO ↔ Domain ↔ ORM **三层映射**,小步骤稍繁
- ⚠️ 需要在 `repositories/` 显式写 ORM ↔ domain 映射代码
- ⚠️ 新增字段时,DTO / domain entity / ORM model 三处都要改
- 📎 关联:[CLAUDE.md §代码硬规矩 1](../../CLAUDE.md) / [ADR-0003](0003-flat-layout-and-domain-naming.md) / [architecture.md §模块清单](../architecture.md)
