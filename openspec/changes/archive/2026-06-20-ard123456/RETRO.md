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

### 矛盾 2：proposal.md Impact 漏报 openspec/config.yaml

**发现时间**：sdd-review 阶段 code review（reviewer Important issue #1）。

**矛盾描述**：
- `proposal.md` Impact 段写"不影响现有代码（main 分支当前只有初始 commit）"。
- 但 `f4890d8`（Ring 1 freeze commit）实际新增了 `openspec/config.yaml`，未被 proposal 列入变更足迹。

**根本原因**：Ring 1 freeze 时把 `openspec/config.yaml` 视为 `openspec init` 自动生成的脚手架（类似 `.gitignore`），不算"业务代码"或"变更内容"。这个判断在技术上合理，但 proposal 的 Impact 段没显式说明，让 reviewer 困惑。

**本次处理**：不修 proposal（Ring 1 frozen input）。在 review 阶段补此 RETRO 记录，让 reviewer 看到"已识别 + 已说明"。

**建议修复**（给后续 spec 模板）：proposal Impact 段增加一个子项"openspec 脚手架文件（config.yaml 等）"，让"自动生成 vs 业务变更"的边界显式化。

### 矛盾 3：cfe2635 baseline 违反（已修复）

**发现时间**：sdd-review 阶段 code review（reviewer Important issue #2）。

**矛盾描述**：commit `cfe2635` 由用户（miniceM）在 /sdd-task 阶段创建，message 为 `4.0: mark ring-3 meta-tasks as skipped (refs ard123456)`，未以 `[ARD123456]` 开头，违反 enterprise baseline。

**本次处理**：用户选 A 方案，用 `git commit-tree + rebase --onto` 非交互式改写（保留原 tree 与 author），cfe2635 被替换为 `b52bb13 [ARD123456] task: 4.0 - mark ring-3 meta-tasks as skipped`。后续 3 个 apply commit 的 SHA 同步更新（diff 内容不变）。

**残留**：4 个更早的 baseline 违反 commit（`6515542 plan(...)` / `55dba82 plan(...)` / `f4890d8 spec(...)` / `3b0190c task(...)`）按 RETRO 矛盾 1 的"不追溯已推送历史"立场保留不改。

### 矛盾 4：spec scenario 4 的失败触发条件与 openspec 实际行为漂移

**发现时间**：sdd-review 阶段执行 tasks.md 5.2（失败路径演示）。

**矛盾描述**：
- spec scenario 4 列举的 validate 失败触发条件："requirement 缺少 scenario、scenario 误用 3 个 `#`、proposal 中 capability 名与 spec 文件路径不一致"。
- 实测前两个**未触发**：
  - 删除 scenario 的 WHEN/THEN 内容 → `openspec validate --strict` 仍通过
  - 把 `#### Scenario:` 改为 `### Scenario:`（少一个 `#`）→ validate 仍通过
- 实测可靠触发的方式：破坏 delta section header（如把 `## ADDED Requirements` 改成 `## Random Invalid Title`）。

**根本原因**：spec scenario 在 Ring 1 写作时基于对 openspec strict 行为的推测，没有实测验证。openspec strict 实际只校验"delta section 结构"，对 scenario 内部格式（`#` 数量、WHEN/THEN 必填）较宽松。

**本次处理**：tasks.md 5.2 改用"破坏 delta header"方式触发失败，failure-demo-evidence.md 记录完整证据。spec scenario 文本不修（Ring 1 frozen input）。

**建议修复**（给 oh-my-sdd 维护者 / 后续 spec 模板）：scenario 描述失败条件时应基于实测，避免"想当然"的触发条件列表。


