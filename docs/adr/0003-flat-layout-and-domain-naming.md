# ADR-0003 · 后端 flat 布局 + `domain/` 命名

- **Status**: Accepted
- **Date**: 2026-05-28
- **Tags**: architecture, backend, naming

## 背景 Context

启动 [ADR-0002](0002-lean-domain-layer.md) 引入 domain 层时,出现两个具体决策点:

1. **Python 项目布局**
   - flat 布局:`server/...` 在仓库根
   - src 布局:`src/ridemall/...`,需 `pip install -e .` 才能跑

2. **业务规则层命名**
   - `domain/`(DDD 术语)
   - `core/`(更通用)
   - `business/` / `logic/` / `model/`(其他可能)

需要明确选哪个,避免后续每次创建文件都纠结。

## Decision

- **后端用 flat 布局**:`server/...` 在仓库根,**不用** `src/` 前缀
- **业务规则层命名 `domain/`**,**不用** `core/`
- 启动方式:`python -m server.app`(无需 pip install)
- 车机端被 gradle 强制 src 布局(`android/app/src/main/...`),**不在本决策范围**

## Rationale

**布局**:
- demo 12 切片,不需要 install discipline 来防 import 污染
- flat 布局启动门槛低,README 一行命令即可上手
- `python -m server.app` 自然支持,无需额外配置

**命名**:
- `domain` 是 DDD 精确术语(业务实体 + 规则),含义明确
- `core` 含义模糊 —— 业内有人放业务规则,有人放共用基础设施(日志/配置/异常基类),**易污染**
- 命名精确性 → 未来不会有人问"为什么 `domain/` 里有 `logger.py`"
- 行业可移植:其他 Python / Java / Kotlin 项目也用 `domain` 这个词

## Alternatives Considered

- **A: src 布局**(`src/ridemall/...`) → demo 体量过度工程,启动步骤增加;主要价值(防 import 污染)在 demo 体量下用不上
- **B: 业务层叫 `core/`** → 含义模糊,易被其他东西(日志、配置、工具)塞进来污染;违反"边界清晰"原则
- **C: 业务层叫 `business/` / `logic/`** → 不如 `domain` 行业通用;DDD 文献都用 domain
- **D: 业务层叫 `model/`** → 跟 ORM `models/` 命名冲突,造成混淆
- **E: 业务规则不抽独立目录,直接放 `server/` 根** → 5 层架构需要清晰目录边界,根目录扁平化反而失序

## Consequences

- ✅ `python -m server.app` 一行启动,README 简单
- ✅ `domain/` 命名精确,不会被误用做杂物层
- ✅ 跨项目可移植性高(其他 Python 项目可复用此结构)
- ⚠️ 如果未来项目扩张到 monorepo(多个 Python 包),可能需要迁移到 src 布局
- ⚠️ 该迁移成本:加 `pyproject.toml` + `setup.py`,改 import 路径;约半天工作量
- 📎 关联:[ADR-0002](0002-lean-domain-layer.md) / [architecture.md §模块清单](../architecture.md)
