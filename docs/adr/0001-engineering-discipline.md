# ADR-0001 · 工程纪律与交付节奏

- **Status**: Accepted
- **Date**: 2026-05-28
- **Tags**: process, governance, testing

## 背景 Context

车载商店 02 是 demo 项目,用户**不读代码**,但提出了多项高质量要求:
- 测试驱动开发(三层金字塔,反馈推进而非一次写完)
- 显式输入输出边界
- 业务模块清晰,**未来可单一替换**
- 浅层入口禁藏业务
- 边走边测试
- 文档与代码同步
- 共识有约束力,任何越界要走流程

若无明确工作纪律,容易出现:边写边变需求 / 测试缺失 / 文档代码不同步 / AI 或人改动越界 / "上次说过"的口头记忆漂移。

## Decision

采用以下纪律组合,落地到具体文档:

1. **共识为唯一合同**:[需求共识.md](../../需求共识.md)
2. **双向防漂移**:用户主动越界 + 我无意越界,两类都先停问
3. **TDD 红绿循环**(Kent Beck):Red → Green → Refactor,每函数级
4. **业务能力纵切**:C1...C12 共 12 个独立可演示切片
5. **里程碑演示**:M1...M6 共 6 次给用户看
6. **三层文档体系**:elevator pitch ([CLAUDE.md](../../CLAUDE.md)) + living snapshot ([architecture.md](../architecture.md)) + decision history ([docs/adr/](.))
7. **三层测试**:单元 + 集成 + E2E(测试金字塔)
8. **DoD 硬条件**:7 条同时满足才算切片做完(详见 [delivery-process.md](../delivery-process.md))
9. **未落文档不算结论**:任何决策必须以文件形式落到仓库

## Rationale

- **共识** → 防止"边做边变"导致 demo 永远做不完
- **双向防漂移** → AI 容易自己加东西,用户也容易临时改需求,两边都要拦
- **TDD** → 用户明确要求"测试反馈驱动",不允许"先全写完再测"
- **纵切** → 每片可独立演示,集成风险**前置**,降低 demo 失败概率
- **三层文档** → onboarding / 现状理解 / 决策追溯 是 3 种不同认知任务,不能合并(参考 Cyrille Martraire《Living Documentation》)
- **三层测试** → 行业标准测试金字塔(Mike Cohn / Martin Fowler)
- **DoD** → "做完"有明确判定,减少返工和"看起来好像做完了"的灰色地带

## Alternatives Considered

- **A: 不分切片,一口气写完后测试** → 违反用户明文要求,直接放弃
- **B: 横向切**(先全后端再全前端) → 不能独立演示,集成风险后置,demo 失败风险高
- **C: 单一文档**(README 塞所有) → 不同认知任务混合,索引困难,违反 single-responsibility
- **D: 不写 ADR**(靠对话 / 代码注释记) → 违反「未落文档不算结论」第一原则
- **E: 完整敏捷 Scrum** (daily standup / sprint review / burndown) → demo 单人项目,过度工程

## Consequences

- ✅ 演示前每个里程碑都可见、可验证,降低 demo 风险
- ✅ 测试覆盖随时间累积,**不会"突击补测"**
- ✅ 决策可追溯,3 个月后回看不困惑
- ✅ AI agent / 新人 onboard 有明确入口
- ⚠️ 前期文档建设有 1-2 天额外开销(已支付)
- ⚠️ 每个切片增加"对账"步骤,需要纪律执行
- ⚠️ 如果用户多次修改共识,文档同步成本会累积
- 📎 关联:[CLAUDE.md](../../CLAUDE.md) / [需求共识.md](../../需求共识.md) / [delivery-process.md](../delivery-process.md)
