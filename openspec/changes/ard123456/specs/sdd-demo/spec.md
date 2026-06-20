## ADDED Requirements

### Requirement: SDD 流程端到端可演示

系统 SHALL 提供一个最小 demo capability `sdd-demo`，使得开发者可以走完 oh-my-sdd 的五环流程（spec/plan/task/apply/review）并通过 archive merge 将 spec 合并到 `openspec/specs/`。该能力 SHALL 在任一环节被跳过或产物缺失时阻断后续流程，以强制 SDD 五环约束。

#### Scenario: 完整走完 5 环

- **WHEN** 开发者依次执行 `/sdd-spec`、`/sdd-plan`、`/sdd-task`、`/sdd-apply`、`/sdd-review`
- **THEN** 所有阶段产物（proposal/specs/tasks/code/review）均生成且 `openspec validate --strict` 通过

#### Scenario: archive merge 更新 specs

- **WHEN** `/sdd-archive` 在 review 通过后执行
- **THEN** `openspec/specs/sdd-demo/spec.md` 被创建，内容包含本变更 ADDED 的 requirement 与 scenario

#### Scenario: gh issue 与 PR 自动关联

- **WHEN** gh issue 创建后，后续 PR 在 body 中引用 issue 编号（如 `Closes #NNN`）
- **THEN** PR 合并时 issue 自动关闭

#### Scenario: openspec validate 失败阻断 push

- **WHEN** `/sdd-review` 执行时 `openspec validate --strict <change>` 返回非零退出码（如 requirement 缺少 scenario、scenario 误用 3 个 `#`、proposal 中 capability 名与 spec 文件路径不一致等）
- **THEN** `/sdd-review` SHALL 拒绝执行 `git push`，输出 validate 失败详情，并要求开发者修复 spec 后重新执行 `/sdd-review`
