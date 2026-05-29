# CLAUDE.md · 车载商店 02 工作纪律

> **修订记录**
> - v1 (2026-05-28) — 初版
> - v1.1 — 代码硬规矩 7 → 15(错误分层 / 副作用 / 事务 / 输入校验 / 依赖门槛 / 修改范围 / 运行环境 / 命名)
> - v1.2 — 合并第一原则+锁定结论;+ 新会话 SOP / 切片完整性 / commit 格式 / 两锚校验
> - v1.3 — + 自动化检查节(pre-commit hooks + Claude Code Stop hook)
> - v1.4 — + 顶层骨架(server/android/deploy 各一份 _README);代码硬规矩 16(架构遵循)+ 配套 hook
> - v1.5 — 审计修复:SOP 加 wip + Gherkin 步;Rule 16 改路径白名单;architecture 拆分册;+ 故事 ID / 架构 bump hook
> - v1.6 — + 真相优先级 + 跨文档一致性核对(防文档腐烂)
>
> 每次进入本项目自动加载。这是**我必须遵守的铁律**。详细规则下沉到对应专题文档。本文件硬上限 200 行。

---

## 📜 不变铁律

**未落文档不算结论** — 任何讨论 / 对齐 / 决策必须以文件形式落到仓库;下一轮会话不接受"上次说过",**只认仓库**。判断口诀:**3 天后还要再讲一次的,就写下来**。

**锁定结论**:[需求共识.md §3 关键决策](需求共识.md) 已定型,**不要再问、不要再议**;要改走「共识修改流程」(用户明文 + 改共识 + 同提交记 ADR)。

**真相优先级 + 防腐**:文档打架谁赢 —— 范围→`需求共识.md`、为什么→`docs/adr/`、行为→测试、带锚点硬事实→`需求共识.md §3` 锚点表那一行(别处只写「见 Fn/An」**不重抄数值**);**改了任一锚点事实,完工前必做跨文档一致性核对**(grep 锚点值 + 通读找语义矛盾,清净才算 `done`)。

---

## 🔄 新会话自启动 SOP

用户开会话只需一个字 "嗯" 或直接动作。我**自动**:
1. 跑 `git log --oneline -20` 看上次到哪
2. `cat docs/wip.md` 看有无跨会话半成品遗留
3. 读 `docs/story-map.md` 校验切片 / 故事状态
4. 两锚一致 → 提议「上次完成 C_n,下一片 C_{n+1}(US-XXX-NN),开始?」;不一致 → 停下问用户,登记 `drift-log.md`
5. 用户拍板后:**先在 `docs/user-stories.md` 补本片 Gherkin → 用户二次确认 → 才进红绿循环**(禁止跳过直接写代码)

---

## ⚖️ 两锚校验铁律

两个独立事实之锚:`git log` commit 状态(开工/wip/done) + `docs/story-map.md` 故事状态(⏳/🟡/✅)。
**两锚必须一致**;不一致 → 停 + 问用户 + 落 `drift-log.md`。

---

## 💾 commit message 强制格式

`[C_n <状态>] <描述> (<故事 ID>)`,状态 ∈ {`开工`, `wip`, `done`};里程碑用 `[M_n <状态>]`。
例:`[C3 done] categories CRUD (US-ADMIN-03)` / `[M0 done] initial docs scaffold`。
**`.pre-commit-config.yaml` 自动校验格式**;不合规 commit 物理无法落地。

---

## ✋ 切片完整性铁律

- **禁止半成品过夜**:切片未达 [DoD 7 条](docs/delivery-process.md) 不允许 `commit -m "[C_n done]"`
- 会话末要么 `done` 要么 `wip` 要么 `git stash`
- **不允许**未 commit 的本地改动跨会话遗留

---

## 🪝 自动化检查

人为纪律 + 机械补强,**两层**:
- **`.pre-commit-config.yaml`**:commit 前自动跑 — 基础卫生 + 硬编码扫描(Rule 14)+ 路径白名单(Rule 16,`docs/.path-whitelist` 整行匹配)+ architecture 改动强制 bump + commit 格式 + **C_n 必带故事 ID** + **代码⟹文档闸门**(改 `server/`·`android/` 的 `.py`/`.kt` 必须同提交带 `.md`,否则 message 写 `[doc-skip: 原因]`)+ **两锚同步闸门**(`[C_n 开工|done]` 必带 `docs/story-map.md`;`[C_n done]` 还须带 `docs/wip.md`);失败 **block commit**
- **`.claude/settings.json`**:会话结束自动 `git status --short && git log --oneline -5`,作为两锚校验的**输入**(打印,非阻断)
- 安装见 [README.md §运行](README.md);**待 C1 起补 ruff / mypy / pytest 钩子**(配置已写好,注释保留)

---

## 📋 文档地图(完整索引见 [README.md](README.md))

| 我要做什么 | 该读哪个 |
|---|---|
| 知道**做什么、不做什么** | [需求共识.md](需求共识.md) |
| 知道**系统现在长什么样** | [docs/architecture.md](docs/architecture.md) |
| 知道**还有哪些用户故事 + 优先级** | [docs/story-map.md](docs/story-map.md) |
| 知道**测试要覆盖的 Gherkin 场景** | [docs/user-stories.md](docs/user-stories.md) |
| 知道**测试怎么写、覆盖到哪** | [docs/test-plan.md](docs/test-plan.md) |
| 知道**什么算"做完"、怎么给用户反馈** | [docs/delivery-process.md](docs/delivery-process.md) |
| 知道**怎么运行 / 测试 / lint** | [README.md](README.md) §运行 |
| 知道**为什么这样设计** | [docs/adr/](docs/adr/) |

---

## 🏗 架构一句话

车机 Android(Kotlin + Compose + Retrofit)→ HTTPS → 后端 Flask + SQLite。
后端 5 层:`views`(HTTP)→ `services`(编排)→ `repositories`(数据)→ `models`(ORM)+ `domain`(纯业务,**任何层可依赖它,它不依赖任何层**)。
车机端 5 层:`ui`(Compose)→ `viewmodel` → `data/repository` → `data/api` + `data/model`,DTO 与 Model 显式分离。
部署:Mac mini + Cloudflared tunnel @ `ridemall.hearagain.space`。
详细模块图见 [docs/architecture.md](docs/architecture.md)。

---

## 🚧 共识防漂移(3 条铁律)

1. **用户主动让我越界** → 先停 + 回放「这超出共识 §X」+ 三选(更新共识 / 临时补丁 / 放弃)→ 用户拍板再动
2. **我无意越界** → 自停 + 落 `docs/drift-log.md`(惰性创建)+ 等用户确认
3. **共识灰色地带**(共识 §7)→ 一律停,列 2-3 个做法 + 后果 + 推荐 → 用户拍板 → 记一份 ADR

---

## 💻 代码硬规矩(15 条)

### 架构与边界
1. **后端 5 层依赖单向**:`views → services → repositories → models`;`domain` 谁都能 import,**它不能 import 任何其他层**
2. **车机端**:Composable **不能**调网络,只能调 ViewModel;`DTO ≠ Model`,必须经 Repository 显式映射
3. **入参出参用 dataclass**(Python)/ **data class**(Kotlin),**禁止 dict 渗入 `services` / `domain`**

### 浅层入口反模式
4. **`views` / Composable 写业务超过 3 行** → 停,先抽到 `services` + 单测,再回 view 调用
5. **发现旧代码业务藏在浅层入口** → 先抽离 + 用户确认,**再**叠新功能

### 模块声明
6. **每个 module 顶部** `__all__` + docstring 声明「依赖什么 / 暴露什么」

### 测试覆盖
7. **`domain/` 100% 单测覆盖**(纯函数,无理由不覆盖)

### 错误处理 / 副作用 / 事务
8. **错误处理分层**:`domain/` 抛 domain exception(`ProductNotFound`、`BannerLimitExceeded` 等);`services/` 不抛 `HTTPException`;**只有 `views/`** 把 domain exception 翻译为 HTTP 状态码 + JSON
9. **副作用边界**:`domain/` 禁止 print / log / IO;时间 / 随机数 / env 必须通过参数注入(不允许 `datetime.now()`、`os.environ.get(...)` 直接出现在 domain)
10. **事务边界**:写库操作必须在 `services/` 用显式 transaction 包裹;`repositories/` 不自行 commit / rollback;**一次业务操作 = 一个事务**

### 输入与依赖
11. **用户输入校验**:外部输入(HTTP 参数、表单、上传)在 `views/` 层用 dataclass + validator 验证;失败统一返 400 + `{"error":"..."}`;**domain / services 不接收未经校验的数据**
12. **新依赖添加门槛**:加 pip / gradle 新包前**必须先开 ADR**(原因 / 替代方案 / 影响);「未落文档不算结论」的推论

### 修改范围与运行环境
13. **修改范围最小化**:非本任务文件不顺手改 / 重命名 / 优化;未经确认不跨模块重构;发现更大问题 → 先报告 + 落 `drift-log.md`,**不借机大改**;每提交最小可验证改动
14. **不假设运行环境**:任何 hard-code 路径 / 端口 / URL / 用户名 / 密码 → **必须走配置**(env / `config.py` / `BuildConfig`);代码里不允许出现 `localhost:18767`、`/Users/Admin/...`、`Ridemall@Demo2026` 等具体值

### 命名约定
15. **命名约定**:类型后缀固定 `Repository` / `Service` / `ViewModel` / `Dto` / `Dao`;布尔用 `is_*` / `has_*` / `can_*`(Python)、`is*` / `has*` / `can*`(Kotlin);基础风格(snake_case / PascalCase / camelCase)由 `ruff` / `ktlint` 强制

### 架构遵循
16. **新文件目录必须在 `docs/.path-whitelist` 声明**(机器可读)+ 在 `docs/architecture/` 对应分册有人话说明:写 `server/` `android/` `deploy/` 下新文件前先确认目录已登记;未登记 → 先加白名单 + 改分册再 commit。`path-must-be-declared` hook **整行精确匹配**机械阻止(android `res/` 框架目录特例放行)

---

## 🧪 测试纪律(3 条 · 详见 [test-plan.md](docs/test-plan.md))

1. **红绿循环**(Kent Beck TDD):先写**失败的测试** → 写代码让它绿 → 重构。**禁止"一口气写完再测"**
2. **每写一个 service / repository / domain / util 函数 → 立刻单测**;每写一个 endpoint → 立刻集成测试
3. **切片测试全绿** → 才算切片完成;否则不允许交付

---

## ✅ 交付铁律

切片"做完" = 单元 + 集成 + E2E 全绿 + 文档同步 + drift-log 无未闭合 + 中文交付报告 + 共识对账完成。
详见 [delivery-process.md §DoD](docs/delivery-process.md)。

---

## 📝 何时必须写文档

| 触发场景 | 落到 |
|---|---|
| 用户讨论需求 / 范围被确认 | `需求共识.md`(用户明文授权)+ 同提交记 ADR |
| 技术 / 架构决策被确认 | `docs/adr/0NNN-*.md` |
| API 改动 | `docs/api.md`(同提交) |
| 新模块 / 边界变化 / 数据流变化 | `docs/architecture/` 对应分册 + `docs/.path-whitelist`(同提交) |
| 用户故事 / 验收场景新增 | `docs/story-map.md` + `docs/user-stories.md` |
| 越界事件(主动或无意) | `docs/drift-log.md`(惰性创建) |
| 临时方案 / 妥协 | `docs/tech-debt.md`(惰性创建) |
| **删除 / 重命名任何文档前** | **必须 grep 所有引用并修复**;裸删禁止 |

---

## 🛑 越界即停

- 共识漂移 / 灰色地带 → 先停问用户
- 写"临时方案"前 → 先登记 `docs/tech-debt.md`;**无登记不允许写**
- 代码 `TODO` / `FIXME` **必须带债务编号**(`# DEBT-2026-MM-NNN`);裸 TODO 禁止
- 安全类债务 🔴 不允许 `accepted`,演示完必须闭合

---

## 💬 给用户反馈(2 条核心 · 详见 [delivery-process.md](docs/delivery-process.md))

1. 用户**不读代码**。所有反馈用中文人话,**不贴代码**
2. 测试报告格式:`✅ 23/23 通过` 或 `❌ 22/23(失败:Banner 超 5 应报错但没报)`,失败用人话讲业务影响

---

## 📅 实施节奏

业务能力纵切 C1...C12,演示节奏 M1...M6(共 6 次)。详见 [delivery-process.md §实施节奏](docs/delivery-process.md)。

---

## ⚠️ 防膨胀触发(3 核心 · 详见 [architecture.md](docs/architecture.md))

任一触发 → 强制拆:
1. 本文件 > 200 行 / `architecture.md` 总览 > 80 行 / 任一架构分册 > 150 行
2. 新人 onboard > 15 分钟还没看懂
3. 频繁需要横向引用 3+ ADR 才能解释一个问题
