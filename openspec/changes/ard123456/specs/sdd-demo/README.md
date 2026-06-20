# sdd-demo Capability

## 用途

`sdd-demo` 是一个**演示型 capability**，用于验证 [oh-my-sdd](https://github.com/) 工作流能否在当前仓库端到端跑通。它本身不提供业务功能，存在的唯一目的是把 SDD（Spec-Driven Development）五环流程在真实 git 仓库里完整走一遍——spec → plan → task → apply → review → archive——确认每一环的产物、约束、钩子都能按设计工作。

引入这个 capability 后，后续真正业务功能的作者可以把它当作"如何用 SDD 引入新 capability"的基线参考。

## 五环流程

本 capability 通过以下五环顺序生成并归档：

| 环 | 命令 | 产物 |
|---|---|---|
| 1. Spec | `/sdd-spec ard123456` | `proposal.md` + `specs/sdd-demo/spec.md`（delta）+ `.meta.json` |
| 2. Plan | `/sdd-plan ard123456` | `design.md`（D1-D5 决策 + R1-R5 风险）+ `tasks.md`（5 环 = 5 任务） |
| 3. Task | `/sdd-task ard123456` | 在 `.meta.json` 记 `task_phase: skipped`（粒度已足） |
| 4. Apply | `/sdd-apply ard123456` | 本 README（capability 的"代码产物"） |
| 5. Review+Archive | `/sdd-review ard123456` + `/sdd-archive ard123456` | 失败路径演示证据贴 PR body；archive 后 spec/README 进入 `openspec/specs/sdd-demo/` |

## 如何引用本 capability

**Archive 后**（PR merge + `/sdd-archive` 完成后）：

- 需求规格：[`openspec/specs/sdd-demo/spec.md`](../../../../specs/sdd-demo/spec.md)
- 本 README：`openspec/specs/sdd-demo/README.md`

**Archive 前**（当前状态）：

- 需求规格 delta：`openspec/changes/ard123456/specs/sdd-demo/spec.md`
- 设计与任务：`openspec/changes/ard123456/design.md` / `tasks.md`

## 关键约束（来自 spec）

本 capability 的 spec 含 4 个 scenario，其中负面路径 scenario——"openspec validate 失败阻断 push"——通过 `/sdd-review` 阶段临时改坏 spec、捕获 validate 失败输出、贴入 PR body 作为证据、再还原 spec 的方式演示。这种"即走即记"的策略保证证据真实且不污染主分支，详见 `design.md` 决策 D2。
