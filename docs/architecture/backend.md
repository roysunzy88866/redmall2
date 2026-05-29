# 架构 · 后端(server/)

> [← 返回总览](../architecture.md)
> **修订记录**:v1 (2026-05-28) — 从 architecture.md 拆出
> **硬上限 150 行**。

## 模块清单 `server/`

| 模块 | 职责 | 允许依赖 |
|---|---|---|
| `domain/` | 业务实体(dataclass) + 校验规则 + 业务异常 | 无 |
| `models/` | SQLAlchemy ORM 表结构 | `sqlalchemy` |
| `repositories/` | ORM ↔ domain 映射 + 查询 + 写入 | `domain`, `models` |
| `services/` | 业务编排(校验 + 事务 + 调 repo) | `domain`, `repositories` |
| `views/api/` | 消费端 HTTP(车机调用) | `services`, `flask` |
| `views/admin/` | 后台 HTTP + Jinja 渲染 | `services`, `flask` |
| `config.py` | 配置(密钥 / 数据库 / 上传目录) | 无 |
| `scripts/` | 一次性脚本(init_db / create_admin / seed_demo) | 全部 |

> 新增目录必须同步 [docs/.path-whitelist](../.path-whitelist)(否则 pre-commit block)。

## 依赖边界(谁能调谁)

```
views → services → repositories → models → (SQLite)
domain ←── 谁都能 import,它不 import 任何层
```

**反方向禁止**:
- `domain` ❌ import `services` / `repositories` / `models` / `flask`
- `repositories` ❌ import `services` / `views`
- `services` ❌ import `views` / `flask.request`

## 数据流 · 下单

```
POST /api/orders {product_id, qty:1, device_id}
  → views/api/orders.py 校验 dataclass(PlaceOrderRequest)
  → services.place_order(request)
        ├── repositories.products.get_active(id) → Product(domain)
        ├── domain.rules.validate_order(product, qty=1)
        ├── domain.entities.Order.snapshot_from(product)
        └── repositories.orders.create(order) → 事务 commit
  → 返回 OrderCreatedDto JSON 201
```

## HTTP 错误码映射(单一真相,各 view 照此翻译)

| domain 异常 | HTTP | JSON |
|---|---|---|
| (输入校验失败) | 400 | `{"error": "..."}` |
| (未登录,后台) | 401 | 重定向登录页 |
| `ProductNotFound` / 资源不存在 | 404 | `{"error": "..."}` |
| `BannerLimitExceeded` / 业务冲突 | 409 | `{"error": "..."}` |
| (未捕获) | 500 | `{"error": "internal"}` |

> 新增 domain 异常时在此表加一行(Rule 8 要求只在 views 翻译)。
