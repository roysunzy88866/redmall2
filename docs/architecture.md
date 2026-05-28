# 架构现状(Architecture Snapshot)

> **状态**:蓝图(C1 实施前)。每完成一个 C 切片,此文档同提交更新到反映现实。
> **修订记录**:v1 (2026-05-28) — 初版蓝图
>
> 这份是「**现在系统长什么样**」的快照。
> 「**为什么这样设计**」请看 [docs/adr/](adr/)。
> 「**我们要做什么**」请看 [需求共识.md](../需求共识.md)。
> 「**铁律和工作纪律**」请看 [CLAUDE.md](../CLAUDE.md)。
>
> **硬上限 150 行,超了拆分**(拆分触发条件见末尾)。

---

## 一图流

```
[车机 Android]                            [浏览器]
       │                                       │
       │  HTTPS                                │  HTTPS
       ▼                                       ▼
       └────────► ridemall.hearagain.space ◄───┘
                          │
                          ▼ Cloudflared (panqian-tunnel)
                  [Mac mini @ 127.0.0.1:18767]
                          │
                  ┌───────┴─────────┐
                  │ Flask app       │
                  │  views/         │ ←─ HTTP 层
                  │  services/      │ ←─ 业务编排
                  │  repositories/  │ ←─ 数据访问
                  │  models/        │ ←─ ORM
                  │  domain/        │ ←─ 业务规则(无依赖)
                  └────────┬────────┘
                           │
                           ▼
                   [SQLite ridemall.db]
                   + uploads/(商品图)
```

---

## 模块清单

### 后端 `server/`

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

### 车机端 `android/app/src/main/java/space/hearagain/ridemall/`

| 模块 | 职责 | 允许依赖 |
|---|---|---|
| `data/dto/` | 网络层 DTO(Retrofit 反序列化目标) | `moshi` |
| `data/model/` | UI domain model(车机端的"domain") | 无 |
| `data/api/` | Retrofit 接口定义 | `retrofit`, `dto` |
| `data/repository/` | 数据聚合 + DTO ↔ Model 映射 | `api`, `dto`, `model` |
| `viewmodel/` | UI 状态机(StateFlow) | `repository`, `model` |
| `ui/` | Compose 渲染 | `viewmodel`, `model` |
| `util/` | QR 生成等纯函数 | 无 |

---

## 关键边界(谁能调谁)

```
后端:
  views   →  services  →  repositories  →  models
                                              ↓
                                            (SQLite)
  domain   ←── 谁都能 import,它不 import 任何层

车机端:
  ui  →  viewmodel  →  repository  →  api  →  (HTTPS)
  model  ←── ui / viewmodel / repository 都用,它不依赖任何层
```

**反方向禁止**:
- `domain` ❌ import `services` / `repositories` / `models` / `flask`
- `repositories` ❌ import `services` / `views`
- `services` ❌ import `views` / `flask.request`
- `ui` (Composable) ❌ 直接调 Retrofit / 网络
- `data/model` ❌ import `retrofit` / `dto`

---

## 数据流(典型路径)

**下单流程**:
```
Android ProductDetailScreen 点「下单」
  → ViewModel.placeOrder(productId)
  → Repository.placeOrder(productId, deviceId)
  → POST /api/orders  {product_id, qty:1, device_id}
  → Flask views/api/orders.py 接收
  → 校验 dataclass(PlaceOrderRequest)
  → services.place_order(request)
        ├── repositories.products.get_active(id)  → Product (domain)
        ├── domain.rules.validate_order(product, qty=1)
        ├── domain.entities.Order.snapshot_from(product)
        └── repositories.orders.create(order)  → 事务 commit
  → 返回 OrderCreatedDto JSON 201
  → Android Repository 映射 DTO → OrderConfirmation (model)
  → ViewModel 切到 success 状态
  → UI 显示「支付成功」
```

---

## 怎么加一个新功能(快速指引)

1. **看共识**:[需求共识.md](../需求共识.md) 这个功能在不在范围内?不在 → 走「共识修改流程」
2. **找/加故事**:[docs/story-map.md](story-map.md) 找对应 `US-*` ID;若无则新增
3. **写场景**:[docs/user-stories.md](user-stories.md) 补 Gherkin Given/When/Then(JIT)
4. **建测试空壳**:对应 `server/tests/` 或 `android/.../test/` 写失败测试(red)
5. **自底向上实现**:`domain` → `repository` → `service` → `view`,每层配单测让其变绿(green)
6. **同提交同步文档**:动了 API → `docs/api.md`;动了模块/边界 → 本文件;出了决策 → 新建 `docs/adr/0NNN-*.md`
7. **跑全测**:`pytest` + Android instrumented test,全绿才能交付

---

## 部署拓扑

Mac mini → launchd 拉起 Flask(`127.0.0.1:18767`)→ Cloudflared `panqian-tunnel` ingress → `https://ridemall.hearagain.space`。
Android `debug` 走 `http://10.0.2.2:18767/`,`release` 走公网。演示完 `launchctl unload` 下线。
详见 [需求共识.md](../需求共识.md) F6 + [ADR-0001](adr/0001-engineering-discipline.md)。

---

## 防膨胀拆分(本文件 > 150 行触发)

按以下方向拆,**不**按时间/特性纵切:

| 触发 | 拆出去 |
|---|---|
| 后端模块 > 10 个 或 数据流 > 5 条主路径 | `docs/architecture/backend.md` |
| 车机端屏幕 > 8 个 或 状态机复杂化 | `docs/architecture/android.md` |
| 部署组件增加(Redis / 队列 / CDN) | `docs/architecture/deployment.md` |
| 引入 AI agent / 多服务边界 | `docs/architecture/agents.md` |

拆完后本文件保留:一图流 + 模块清单顶层 + 跨域数据流 + 跳转索引。
