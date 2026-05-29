# 架构 · 车机端(android/)

> [← 返回总览](../architecture.md)
> **修订记录**:v1 (2026-05-28) — 从 architecture.md 拆出
> **硬上限 150 行**。

## 模块清单 `android/app/src/main/java/space/hearagain/ridemall/`

| 模块 | 职责 | 允许依赖 |
|---|---|---|
| `data/dto/` | 网络层 DTO(Retrofit 反序列化目标) | `moshi` |
| `data/model/` | UI domain model(车机端的"domain") | 无 |
| `data/api/` | Retrofit 接口定义 | `retrofit`, `dto` |
| `data/repository/` | 数据聚合 + DTO ↔ Model 映射 | `api`, `dto`, `model` |
| `viewmodel/` | UI 状态机(StateFlow) | `repository`, `model` |
| `ui/` | Compose 渲染 | `viewmodel`, `model` |
| `util/` | QR 生成等纯函数 | 无 |

> `res/`(values / drawable / mipmap / layout / xml 等)是 **Gradle 框架目录,不计入模块清单**,pre-commit 路径检查特例放行。
> 新增 java 包目录必须同步 [docs/.path-whitelist](../.path-whitelist)。

## 依赖边界(谁能调谁)

```
ui → viewmodel → repository → api → (HTTPS)
model ←── ui / viewmodel / repository 都用,它不依赖任何层
```

**反方向禁止**:
- `ui`(Composable) ❌ 直接调 Retrofit / 网络
- `data/model` ❌ import `retrofit` / `dto`

## 关键映射点(易漂移,无机械保护 — 见 tech-debt)

- `data/dto/XxxDto` ↔ `data/model/Xxx`:必须经 `data/repository/` 显式 `.toModel()`
- 后端 endpoint 路径 ↔ `data/api/ApiService.kt` ↔ `docs/api.md`:**三处契约必须一致**,目前纯人工同步(DEBT-2026-05-003)

## 断网行为(共识 F5 / §1)

车机端**不存任何本地数据**;断网 / 超时 → 全屏「网络异常,点击重试」,不做离线缓存。
