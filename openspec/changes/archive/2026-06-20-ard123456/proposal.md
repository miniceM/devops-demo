## Why

当前仓库需要验证 oh-my-sdd 工作流能否端到端跑通（spec → plan → task → apply → review → archive）。在引入正式业务功能前，先用一个最小可演示的 capability 走完整流程，确认 SDD Agent 的五环约束、openspec delta 生成、gh issue/PR 关联等机制可正常工作，作为后续正式功能引入 SDD 流程的基线。

## What Changes

- 新增 demo capability `sdd-demo`：用于验证 SDD 5 环流程端到端可演示的最小能力
- 不修改任何现有 capability（`openspec/specs/` 当前为空）
- 无 BREAKING 变更

## Capabilities

### New Capabilities

- `sdd-demo`: 演示 oh-my-sdd 端到端工作流（spec → plan → task → apply → review → archive），作为正式业务功能引入前的流程验证基线

### Modified Capabilities

（无——`openspec/specs/` 当前为空，本次为首次创建 capability）

## Impact

- 新增 `openspec/changes/ard123456/specs/sdd-demo/spec.md`（delta，仅含 ADDED Requirements）
- archive 后将创建 `openspec/specs/sdd-demo/spec.md`（项目首个正式 spec）
- 涉及 gh issue 创建与 PR 关联
- 不影响现有代码（main 分支当前只有初始 commit）
