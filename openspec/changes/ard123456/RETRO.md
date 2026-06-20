## RETRO — apply 阶段发现的 task 假设矛盾

### 矛盾 1：tasks.md 4.3 commit message 格式 vs enterprise baseline

**发现时间**：apply 阶段开始前的 plan review。

**矛盾描述**：
- `tasks.md` 4.3 写的 commit message 是 `apply(ard123456): add sdd-demo capability README as code artifact`（conventional commits 风格）。
- 但 enterprise baseline（`~/.claude/CLAUDE.md`）明确："所有 git commit message 必须以 `[<change-id>]` 开头——阻断性强制要求。无 change-id 的 commit 禁止产生。"
- 同时 `oh-my-sdd:sdd-apply` SKILL.md 强制规则也写："commit message 格式：`[<change-id>] apply: <task-id> - <subject>`"。

**历史影响**：本次 change 之前的 commit（`6515542 plan(...)` / `55dba82 plan(...)` / `f4890d8 spec(...)` / `3b0190c task(...)` / `cfe2635 4.0: ...`）均未以 `[ARD123456]` 开头，已经违反 baseline。这些是 Ring 1-3 阶段的 commit，本 RETRO 不追溯修正（破坏已推送历史的风险 > 修复收益），但作为流程漏洞记录。

**本次处理**：
- 在本次 apply 阶段，所有 commit 严格使用 `[ARD123456] apply: <task-id> - <subject>` 格式。
- tasks.md 4.3 的 commit message 视为 writing-plans 在 plan 阶段的 input bug——后续 oh-my-sdd 维护者应改 writing-plans 输出格式，让 tasks.md 自带的 commit message 模板与 baseline 一致。

**根本原因**：sdd-plan 阶段委托的 `superpowers:writing-plans` 是通用 skill，不知道 enterprise baseline 的 `[<change-id>]` 约定，产出了 conventional commits 风格的 commit message 模板。

**建议修复**（给 oh-my-sdd 维护者）：
1. 在 sdd-plan skill 里，给 brainstorming/writing-plans 显式传"commit message 必须以 `[<change-id>]` 开头"的约束。
2. 或在 sdd-apply skill 里，把"用 tasks.md 里写的格式"改为"强制覆盖为 `[<change-id>] apply: <task-id> - <subject>` 格式"。
