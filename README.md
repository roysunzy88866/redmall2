# 车载商店 02(Ridemall Demo)

> 车载横屏 1920×720 上的购物 demo + 后端商品管理后台。**内部演示性质**,不接真实支付,不做物流。
> 完整范围与边界:[需求共识.md](需求共识.md)

---

## 📚 文档地图

### 必读
| 文档 | 作用 | 谁该读 |
|---|---|---|
| [需求共识.md](需求共识.md) | 产品合同(范围 / 边界 / 验收 / 锁定决策) | 你 / 新开发 / AI |
| [CLAUDE.md](CLAUDE.md) | 工作纪律 + 代码硬规矩(15 条铁律) | AI / 任何动代码的人 |
| [docs/architecture.md](docs/architecture.md) | 系统现状快照(模块图 / 边界 / 数据流) | 新开发 / AI / onboarding |

### 测试驱动核心
| 文档 | 作用 |
|---|---|
| [docs/story-map.md](docs/story-map.md) | 14 个用户故事 ID + 优先级 + 状态 |
| [docs/user-stories.md](docs/user-stories.md) | 每故事的 Gherkin 验收场景(JIT 增长) |
| [docs/test-plan.md](docs/test-plan.md) | 测试金字塔 + 工具 + 命名 + 覆盖率仪表盘 |
| [docs/delivery-process.md](docs/delivery-process.md) | DoD + 给用户反馈姿势 + 实施节奏 |

### 决策与演进
| 文档 | 作用 |
|---|---|
| [docs/adr/](docs/adr/) | 架构决策记录(每决策一文件,Status 字段) |

### 惰性创建(出现时才有)
- `docs/api.md` — C3 上线第一个 API 时创建
- `docs/drift-log.md` — 第一次越界事件时创建
- `docs/tech-debt.md` — 第一次登记技术债时创建

---

## 🚪 不同角色的入口

| 你是 | 从这开始读 |
|---|---|
| 产品 owner(你) | [需求共识.md](需求共识.md) → [docs/story-map.md](docs/story-map.md) |
| 新加入的开发 | 本文 → [docs/architecture.md](docs/architecture.md) → [CLAUDE.md](CLAUDE.md) |
| AI agent(Claude / Cursor / etc.) | [CLAUDE.md](CLAUDE.md)(自动加载)→ 按本文索引按需打开 |
| 维护者(找 bug / 加功能) | [docs/architecture.md](docs/architecture.md) "怎么加新功能" 节 |

---

## ▶️ 快速运行

> ⚠️ 待 C1 切片完成后填充。当前项目尚无代码。

### 后端(本地开发)
```bash
# 进入后端目录
cd server

# 创建虚拟环境 + 安装依赖
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# 初始化数据库 + 创建管理员(用户名 admin / 密码 Ridemall@Demo2026)
python scripts/init_db.py
python scripts/create_admin.py

# 启动服务(默认端口 18767)
python -m server.app --port 18767

# 验证(另开终端)
curl -s http://127.0.0.1:18767/api/categories | jq .
```

后台地址:`http://127.0.0.1:18767/admin/login`

### 车机端(Android Studio)
> 待 C4 切片完成后填充。

---

## 🧪 测试

详见 [docs/test-plan.md](docs/test-plan.md)。简表:

```bash
# 后端单测 + 集成测试
cd server && pytest

# 后端覆盖率
pytest --cov=server --cov-report=term-missing

# 后端端到端(需服务在跑)
pytest tests/e2e/

# Android 单测
cd android && ./gradlew test

# Android instrumented test(需 emulator/设备)
./gradlew connectedAndroidTest
```

---

## 🧹 Lint / 格式化

```bash
# 后端
ruff check server/        # 检查
ruff format server/       # 自动格式化
mypy server/              # 类型检查(strict)

# Android
cd android && ./gradlew ktlintCheck     # 检查
./gradlew ktlintFormat                  # 自动格式化
./gradlew detekt                        # 静态分析
```

---

## 🪝 自动化检查(pre-commit hooks)

一次性安装(每台开发机):
```bash
pip install pre-commit
pre-commit install --hook-type pre-commit --hook-type commit-msg
```

之后每次 `git commit` 自动检查:
- 基础卫生(尾空格 / 文件末换行 / 大文件 / 合并冲突 / yaml 合法)
- **硬编码扫描**(`localhost:` / 用户路径 / 密码字面量) → block(违反 CLAUDE.md Rule 14)
- **commit message 格式**(`[C_n 状态]` / `[M_n 状态]`)→ block

详见 `.pre-commit-config.yaml` 和 [CLAUDE.md §自动化检查](CLAUDE.md)。

---

## 🚀 部署

> 待 C11 切片完成后填充。

简述:Mac mini + Cloudflared tunnel(`panqian-tunnel`)+ launchd。
公网地址:`https://ridemall.hearagain.space`
详见 [docs/architecture.md](docs/architecture.md) §部署拓扑 + [docs/adr/](docs/adr/)。

---

## 📜 修订记录

- v0.1 (2026-05-28) — 初版骨架,代码未实施
