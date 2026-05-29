# Test Plan · 测试计划

> 测试金字塔 + 工具 + 命名 + 覆盖率仪表盘。
> 修订记录:v1 (2026-05-28) — 初版

---

## 🔺 测试金字塔

```
        E2E              ← 少而精,每 P0 故事 1 个 happy path
       ─────
      集成测试            ← 每 endpoint 1 happy + 1 error
     ─────────
    单元测试(最厚)        ← domain 100% / services ≥90% / repos ≥80% / util 100%
   ─────────────
```

参考:[Martin Fowler · TestPyramid](https://martinfowler.com/bliki/TestPyramid.html) / Mike Cohn《Succeeding with Agile》。

**底层最厚**:多写单元、少写 E2E。单元跑得快,E2E 慢且易脆。

---

## 📐 三层定义

### 单元测试 Unit
- **覆盖**:`domain/` / `repositories/`(临时 SQLite)/ `services/`(mock repo 或临时 DB)/ `util/`
- **工具**:`pytest`(后端) / `JUnit5` + `kotlinx.coroutines.test`(Android)
- **跑频**:**每写一个函数 → 立刻跑**
- **耗时目标**:全部 < 5 秒
- **覆盖率**:`domain/` 100% / `util/` 100% / `services/` ≥ 90% / `repositories/` ≥ 80%

### 集成测试 Integration
- **覆盖**:HTTP endpoint ↔ services ↔ repositories ↔ SQLite **真实链路**
- **工具**:`pytest` + Flask test client + 临时 SQLite(每测试一个)
- **跑频**:**每写一个 endpoint → 立刻跑**
- **耗时目标**:全部 < 30 秒
- **覆盖率**:每 endpoint 至少 1 happy + 1 error path

### 端到端测试 E2E
- **覆盖**:HTTP 请求 → 完整 stack → 验证 DB 状态 + JSON 响应
- **工具**:`pytest` + `httpx`(请求真实运行的 server);后台 happy path 可选 Playwright;Android UI 用 Compose UI Test + 真实后端
- **跑频**:**每个 C 切片完成 → 跑一次**;**每个里程碑 M_n → 跑 M_1..M_n 全部**
- **耗时目标**:全部 < 2 分钟
- **覆盖率**:每个 P0 故事的核心 happy path 必有 E2E

---

## 🛠 工具与命令

| 工具 | 用途 | 命令 |
|---|---|---|
| `pytest` | 后端单元 + 集成 + E2E | `pytest` |
| `pytest-cov` | 覆盖率 | `pytest --cov=server --cov-report=term-missing` |
| `ruff` | Python lint + 格式化 | `ruff check server/` / `ruff format server/` |
| `mypy` | Python 类型检查(strict) | `mypy server/` |
| `JUnit5` | Android 单元测试 | `cd android && ./gradlew test` |
| `Compose UI Test` | Android UI 测试 | `./gradlew connectedAndroidTest`(需 emulator) |
| `ktlint` | Kotlin lint | `./gradlew ktlintCheck` / `ktlintFormat` |
| `detekt` | Kotlin 静态分析 | `./gradlew detekt` |

---

## 📁 目录结构

```
server/
├── tests/
│   ├── unit/                    # 单元测试
│   │   ├── domain/
│   │   ├── repositories/
│   │   ├── services/
│   │   └── util/
│   ├── integration/             # 集成测试(endpoint ↔ DB)
│   │   ├── test_categories_api.py
│   │   └── test_admin_views.py
│   └── e2e/                     # 端到端
│       └── test_purchase_flow.py
└── conftest.py                  # 共享 fixture(临时 SQLite / Flask app)

android/app/src/
├── test/                        # 单元测试(本地 JVM)
│   └── java/space/hearagain/ridemall/
│       ├── data/
│       └── viewmodel/
└── androidTest/                 # instrumented + UI test
    └── java/space/hearagain/ridemall/
        └── ui/
```

---

## 🔴 红绿循环(Kent Beck TDD)

```
1. RED      写一个失败的测试(empty function → assert fails)
2. GREEN    写最小代码让它通过(no premature optimization)
3. REFACTOR 重构代码与测试(命名 / 抽取 / 简化),保持全绿
```

参考:Kent Beck《Test-Driven Development by Example》(2003)。

**禁止**:写完所有代码再回头补测试。

---

## 📝 命名约定

- 测试文件:`test_<被测模块>.py` / `<BeingTested>Test.kt`
- 测试函数:`test_<场景描述>`,小写下划线;引用 user-stories.md 时:`test_us_admin_03_scenario_3_1_create_category()`
- fixture:`def <用途>_fixture()` 加 `@pytest.fixture`

---

## 🎯 Mock 边界规则

- **`domain/` 不 mock**(纯函数,直接测)
- **`repositories/`**:用**真实 SQLite**(临时数据库),不 mock SQLAlchemy
- **`services/`**:可 mock `repositories/`(对外契约清晰),简单情况优先用临时 SQLite + 真实 repo
- **`views/`**:用 Flask test client + 临时 DB,**不 mock service**(否则测试就是假的)
- **Android `data/repository/`**:可 mock `data/api/`(Retrofit interface)
- **Android `ui/`**:用 Compose UI Test + mock ViewModel

---

## 🔄 回归策略

- **切片末**:跑该切片所有测试,全绿才算交付
- **里程碑末(M_n)**:跑 M_1..M_n 所有测试,确保**无回归**
- **里程碑回归失败** → 立即停,定位是当片引入还是历史漂移,在 `docs/drift-log.md`(惰性创建)登记

---

## ❌ 失败处理

| 场景 | 处理 |
|---|---|
| 单测失败 | 立刻停,修代码或修测试,**不允许**进下一函数 |
| 集成测试失败 | 立刻停,定位是 view / service / repo / domain 哪层 |
| E2E 失败 | 切片**不算完成**,回头修;若涉及共识误解 → 走「共识修改流程」 |
| 测试本身错(误判) | 同时修测试和实现 + `drift-log.md` 登记 + 给用户说明 |

---

## 📊 覆盖率仪表盘

> 每个切片完成后**同提交**更新此节。

### 后端

| 模块 | 单元 | 集成 | 当前覆盖率 |
|---|---|---|---|
| domain/ | 0 / ? | — | 0% |
| repositories/ | 0 / ? | — | 0% |
| services/ | 0 / ? | — | 0% |
| views/api/ | — | 0 / ? | 0% |
| views/admin/ | — | 0 / ? | 0% |

### 车机端

| 模块 | 单元 | UI | 当前覆盖率 |
|---|---|---|---|
| data/repository/ | 0 / ? | — | 0% |
| viewmodel/ | 0 / ? | — | 0% |
| ui/ | — | 0 / ? | 0% |

### 用户故事(Gherkin 场景)

| 状态 | 个数 |
|---|---|
| ⏳ 待细化 | 15 故事(全部) |
| 🟡 进行中 | 0 |
| ✅ 绿 | 0 / 待场景数 |

> 「待场景数」每个故事开工时确定(估 3-5 / 故事,总 ~44)。
