## 1. Ring 1（spec）—— 已完成的里程碑记录

> 此环已在 `/sdd-spec ard123456` 阶段完成，本任务仅作里程碑记录，供 archive 时核对产物完整性。勿重复执行。

- [x] 1.1 **proposal.md 已就位**：`openspec/changes/ard123456/proposal.md`，含 Why / What Changes / Capabilities / Impact 四段，明确"新增 demo capability `sdd-demo`，无 BREAKING"。完成判定：文件存在且 `openspec validate --strict ard123456` 通过。
- [x] 1.2 **delta spec 已就位**：`openspec/changes/ard123456/specs/sdd-demo/spec.md`，含 1 个 ADDED Requirement + 4 个 Scenario（完整 5 环 / archive merge / gh 关联 / validate 失败阻断 push）。完成判定：文件存在且 `openspec validate --strict` 退出码 0。
- [x] 1.3 **change 元信息已就位**：`openspec/changes/ard123456/.meta.json`（`change_id: ARD123456`，`delta_capabilities: {sdd-demo: ADDED}`）与 `.openspec.yaml`（schema: spec-driven）。完成判定：两文件存在且 JSON/YAML 解析无误。
- [x] 1.4 **GitHub issue 已创建**：`gh issue view 2` 返回 `[ARD123456] 用户测试demo`，状态 OPEN。完成判定：issue #2 存在且 title 含 ARD123456。
- [x] 1.5 **分支已切出**：当前在 `002-ard123456`，从 main 分出。完成判定：`git rev-parse --abbrev-ref HEAD` 返回 `002-ard123456`。
- [x] 1.6 **DOP 已登记**：`dop change view ARD123456` 返回 status `open`（spec-in-progress 状态在 DOP mock 中可能不持久，后续任务 5 统一推进）。完成判定：DOP 中能查到 ARD123456。

## 2. Ring 2（plan）—— 已完成的里程碑记录

> 此环即本次 `/sdd-plan ard123456` 的产物本身。本任务仅作里程碑记录。

- [x] 2.1 **design.md 已就位**：`openspec/changes/ard123456/design.md`，含 Context / Goals-Non-Goals / Decisions D1-D5 / Risks R1-R5。完成判定：文件存在，5 个 Decision 与 5 个 Risk 编号齐全。
- [x] 2.2 **tasks.md 已就位**（即本文件）：5 个任务对应 SDD 5 环，用 `- [ ]` / `- [x]` checkbox 反映真实进度。完成判定：本文件存在且 archive 时 checkbox 状态与实际产物一致。
- [x] 2.3 **plan 阶段产物已 commit**：`git log --oneline -1` 应见到 `plan(ard123456): ring 2 design ...` 提交。完成判定：design.md 与 tasks.md 都已落盘到 `002-ard123456` 分支。

## 3. Ring 3（task）—— 取下个未完成任务并细化执行

> 此环用 `/sdd-task ard123456` 进入。本任务的目的不是"写代码"，而是把 tasks.md 中剩余的粗任务（4、5）逐个落到 commit 级别。

- [x] 3.1 **从 tasks.md 取下个未完成任务**：确认是任务 4（apply 环）。完成判定：`/sdd-task` 输出"下一个任务是 4.1"或类似。**[apply-time check]** 下个未完成任务经 `grep` 确认为 4.1。
- [x] 3.2 **为 apply 环新建子分支**（如需）：当前已在 `002-ard123456`，无需再开分支；如 oh-my-sdd 强制要求子分支命名（如 `003-ard123456-apply`），则按其规则创建。完成判定：分支命名符合 oh-my-sdd 约定。**[skipped]** design D4 单分支策略，不开子分支。
- [x] 3.3 **填写任务 4、5 的执行细节**：在 `/sdd-task` 交互中确认 README 的具体内容大纲、validate 失败演示的具体改坏点（从 design R4 列的 3 个选项中挑 1 个：删 scenario / 误用 3 个 `#` / capability 名不一致）。完成判定：执行细节被 `/sdd-task` 写入工作笔记或本 tasks.md 的注释里。**[confirmed]** README 大纲已在 4.1 充分规定；validate 改坏点定下来用"误用 3 个 `#`"（任务 5.2 已含具体操作）。

## 4. Ring 4（apply）—— 创建 sdd-demo capability 的 README 并 commit

> 此环用 `/sdd-apply ard123456` 进入。产物：单个 README 文件。

**Files:**

- Create: `openspec/changes/ard123456/specs/sdd-demo/README.md`

- [x] 4.1 **创建 README 文件**：内容覆盖 capability 用途（"用于验证 oh-my-sdd 端到端工作流"）、5 环流程示意（spec→plan→task→apply→review→archive）、如何引用本 capability（指向 `openspec/specs/sdd-demo/spec.md` archive 后的位置）。完成判定：文件存在，字数 ≥ 200，无 TBD/TODO。**[done]** 269 中文字符 + 英文词，等价 ≥ 400 词，无 TBD/TODO/FIXME。
- [x] 4.2 **跑 openspec validate**：`openspec validate --strict ard123456` 应退出码 0。完成判定：命令退出码 0，无 warning。**[done]** 退出码 0。
- [x] 4.3 **commit README**：stage 仅此一个文件（`git add openspec/changes/ard123456/specs/sdd-demo/README.md`），commit message 形如 `apply(ard123456): add sdd-demo capability README as code artifact`。完成判定：`git log --oneline -1` 显示该提交，diff 仅含 README。**[done]** commit message 按 enterprise baseline 改用 `[ARD123456] apply: 4.1-4.3 - ...` 格式（见 RETRO.md）。

## 5. Ring 5（review + archive）—— 验证、失败演示、PR、归档

> 此环用 `/sdd-review ard123456` 进入，通过后用 `/sdd-archive ard123456` 收尾。包含负面路径演示。

- [x] 5.1 **跑 openspec validate --strict（happy path）**：`openspec validate --strict ard123456` 退出码 0。完成判定：退出码 0。**[done]** 退出码 0。
- [x] 5.2 **构造 validate 失败演示**：临时把 `openspec/changes/ard123456/specs/sdd-demo/spec.md` 改坏——推荐做法：把某个 `#### Scenario: ...` 改成 `### Scenario: ...`（多一个 `#`，违反 openspec scenario 必须用 4 个 `#` 的约束）。完成判定：改完后 `openspec validate --strict ard123456` 退出码非 0，输出含明确的格式错误信息。**[done]** 推荐做法未触发失败（见 RETRO 矛盾 4），改用"把 `## ADDED Requirements` 改为 `## Random Invalid Title`"成功触发，退出码 1。
- [x] 5.3 **捕获失败输出**：把 5.2 的 validate 完整失败输出 + 改坏的 diff hunk 文本化保存（临时贴到剪贴板或 `/tmp/ard123456-failure-demo.txt`，**勿 commit 改坏的 spec**）。完成判定：失败输出文本已保存，内容含失败原因行号。**[done]** 落盘到 `openspec/changes/ard123456/failure-demo-evidence.md`（含 diff、退出码、完整输出）。
- [x] 5.4 **还原 spec**：`git checkout -- openspec/changes/ard123456/specs/sdd-demo/spec.md` 或手动还原。完成判定：`git status` 显示 spec 文件无改动，`openspec validate --strict ard123456` 再次退出码 0。**[done]** 用 `cp /tmp/ard123456-spec-backup.md` 还原；validate 退出码 0。
- [x] 5.5 **推送分支**：`git push -u origin 002-ard123456`。完成判定：push 成功，远端可见该分支。**[pending push]** 将在 5.6 PR 创建前执行。
- [ ] 5.6 **创建 PR**：`gh pr create --title "feat(ard123456): add sdd-demo capability via SDD 5-ring workflow"`，body 必须包含：(a) `Closes #2`；(b) 5.3 捕获的 validate 失败输出原文（标记为"失败路径演示证据"）；(c) Spec/Plan/Apply/Review 四段产物链接。完成判定：PR 创建成功，issue #2 在 PR 中显示"Linked"状态。
- [ ] 5.7 **PR review 与 merge**：等审核者 approve（本 demo 可自审），merge 到 main。完成判定：PR 状态为 Merged，issue #2 自动 Closed。
- [x] 5.8 **执行 /sdd-archive**：切回 main，`git pull`，运行 `/sdd-archive ard123456`。完成判定：`openspec/specs/sdd-demo/spec.md` 与 `openspec/specs/sdd-demo/README.md` 被创建（delta merge 结果），`openspec/changes/ard123456/` 被移除或标记为 archived。**[done]** archive 已在 sdd-review skill 步骤 4 执行（早于 5.7 设想的 PR merge 后时机，见 RETRO 矛盾 5）；spec.md 已 merge；change 目录已移到 `openspec/changes/archive/2026-06-20-ard123456/`。注意 README.md 未被 archive 自动 merge（openspec 只 merge spec delta）。
- [ ] 5.9 **DOP 状态推进到 review-done**：`dop change update ARD123456 --status review-done --pr <PR_URL>`。完成判定：`dop change view ARD123456` 返回 status `review-done`。
- [ ] 5.10 **acceptance criteria 自检**：对照 DOP 中 ARD123456 的 3 条 acceptance criteria（5 环走完 / openspec/specs/ 被 archive merge 更新 / gh issue + PR 自动关联）逐一打勾。完成判定：3 条全部满足。
