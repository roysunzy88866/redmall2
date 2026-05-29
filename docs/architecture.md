# 架构现状(Architecture Snapshot)

> **状态**:蓝图(C1 实施前)。每完成一个 C 切片,对应分册同提交更新。
> **修订记录**:v2 (2026-05-28) — 拆分为总览 + backend/android/deployment 三分册(原 v1 单文件 149 行贴顶)
>
> 本文件 = **总览 + 跳转索引**。详细模块清单见分册。
> 「为什么这样设计」→ [docs/adr/](adr/);「做什么」→ [需求共识.md](../需求共识.md);「铁律」→ [CLAUDE.md](../CLAUDE.md)。
> **本文件硬上限 80 行;各分册硬上限 150 行**。

---

## 一图流

```
[车机 Android]                            [浏览器]
       │  HTTPS                                │  HTTPS
       ▼                                       ▼
       └────────► ridemall.hearagain.space ◄───┘
                          │ Cloudflared (panqian-tunnel)
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
                           ▼
                   [SQLite ridemall.db] + uploads/(商品图)
```

---

## 分册索引

| 分册 | 内容 |
|---|---|
| [architecture/backend.md](architecture/backend.md) | 后端模块清单 + 依赖边界 + 下单数据流 |
| [architecture/android.md](architecture/android.md) | 车机端模块清单 + 依赖边界 + 映射点 |
| [architecture/deployment.md](architecture/deployment.md) | 部署拓扑(Mac mini + Cloudflared)|

**机器可读路径白名单**:[docs/.path-whitelist](.path-whitelist) —— pre-commit hook 用它判定新文件路径合法性。**新增模块时:白名单 + 对应分册同步**。

---

## 怎么加一个新功能(快速指引)

1. **看共识**:[需求共识.md](../需求共识.md) 在不在范围内?不在 → 走「共识修改流程」
2. **找/加故事**:[story-map.md](story-map.md) 找 `US-*` ID;无则新增
3. **写场景**:[user-stories.md](user-stories.md) 补 Gherkin(JIT)
4. **建测试空壳**:写失败测试(red)
5. **自底向上实现**:`domain → repository → service → view`,每层配单测变绿(green)
6. **同步文档 + 白名单**:动 API → `docs/api.md`;动模块 → 对应 architecture 分册 + `.path-whitelist`;出决策 → 新 ADR
7. **跑全测**:`pytest` + Android test,全绿才交付

---

## 防膨胀拆分

任一分册 > 150 行 → 继续拆(如 backend.md 拆出 `backend-data-flow.md`)。
引入 AI agent / 多服务边界 → 新增 `architecture/agents.md`。
