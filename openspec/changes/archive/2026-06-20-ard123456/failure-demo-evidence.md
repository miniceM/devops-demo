# openspec validate 失败路径演示证据

> 本文件为 Ring 5 review 阶段执行 spec scenario 4「openspec validate 失败阻断 push」的现场证据。捕获后 spec 已还原，工作树干净。

## 演示操作（对应 tasks.md 5.2）

把 `openspec/changes/ard123456/specs/sdd-demo/spec.md` 第 1 行的 `## ADDED Requirements` 改为 `## Random Invalid Title`（破坏 delta section header）。

```diff
- ## ADDED Requirements
+ ## Random Invalid Title
```

## validate 失败输出（对应 tasks.md 5.3）

退出码：**1**

```
Change 'ard123456' has issues
✗ [ERROR] sdd-demo/spec.md: No delta sections found. Add headers such as "## ADDED Requirements" or move non-delta notes outside specs/.
✗ [ERROR] file: Change must have at least one delta. No deltas found. Ensure your change has a specs/ directory with capability folders (e.g. specs/http-server/spec.md) containing .md files that use delta headers (## ADDED/MODIFIED/REMOVED/RENAMED Requirements) and that each requirement includes at least one "#### Scenario:" block. Tip: run "openspec change show <change-id> --json --deltas-only" to inspect parsed deltas.
Next steps:
  - Ensure change has deltas in specs/: use headers ## ADDED/MODIFIED/REMOVED/RENAMED Requirements
  - Each requirement MUST include at least one #### Scenario: block
  - Debug parsed deltas: openspec change show <id> --json --deltas-only
```

## 还原后状态（对应 tasks.md 5.4）

`git diff openspec/changes/ard123456/specs/sdd-demo/spec.md` 无输出（spec 已 100% 还原）。`openspec validate --strict ard123456` 退出码 0。

## 与 spec scenario 文本的偏差（记 RETRO）

spec scenario 4 列举的失败触发条件——"requirement 缺少 scenario、scenario 误用 3 个 `#`、proposal 中 capability 名与 spec 文件路径不一致"——实测中前两个**未触发** validate 失败（openspec strict 不强制 scenario 必须有 WHEN/THEN，也不强制 4 个 `#`）。实测可靠触发的失败是"破坏 delta section header"（如把 `## ADDED Requirements` 改名）。这构成 spec scenario 与 openspec 实际行为的小漂移，已在 RETRO.md 矛盾 4 记录。
