## Context

`devops-demo` 仓库当前为空壳（只有初始 commit + README），用于验证 `oh-my-sdd` 工作流能否端到端跑通。change `ARD123456` 引入首个 capability `sdd-demo`，其"实现"本身就是 SDD 5 环流程的产物，而非传统业务代码。

利益相关方：

- **开发者（本 demo 的执行者）**：用最小代价确认 SDD Agent 的五环约束、openspec delta 生成、gh issue/PR 关联机制可用。
- **后续正式 feature 的作者**：把本 change 作为"如何用 SDD 引入新功能"的基线参考。

外部约束：

- `/sdd-spec` 已完成 Ring 1（spec 冻结），proposal + delta spec + .meta.json + gh issue #2 均已就位。
- 分支 `002-ard123456` 已切出，所有后续产物在此分支累积。
- DOP 已记录 change（status: `open`），需在本环结束时推进到 `plan-ready`。

## Goals / Non-Goals

**Goals:**

- 产出一个被 openspec `--strict` 验证通过的 design.md + tasks.md，作为 Ring 2 的冻结产物。
- 在 capability 目录下产出单个 README/示例文档作为"代码产物"，满足 spec scenario 中"code 生成"的字面要求。
- 在 `/sdd-review` 阶段临时构造一次 `openspec validate --strict` 失败，把失败输出贴到 PR body 作为"validate 阻断 push"的现场证据，然后还原 spec。
- PR body 中包含 `Closes #2`，验证 GitHub issue 自动关闭机制。

**Non-Goals:**

- 不引入任何业务功能、可执行脚本、e2e 测试或 CI 钩子。
- 不修改 `proposal.md` / `specs/*.md`（Ring 1 冻结产物，仅作 input）。
- 不演示 oh-my-sdd 之外的 SDD 变体（如双分支、并行副轨）——保持单分支贯穿五环。
- 不覆盖 `openspec/specs/sdd-demo/spec.md`（archive 环的产物，Ring 5 之后才生成）。

## Decisions

### D1：代码产物形态 = capability spec delta 目录下的单个 README

- **选择**：在 `openspec/changes/ard123456/specs/sdd-demo/README.md` 放一个 README/示例文档，说明 sdd-demo capability 的用途与调用方式。该文件随 archive merge 一起进入 `openspec/specs/sdd-demo/README.md`。
- **理由**：proposal 强调"最小可演示、不引入业务功能"，但 spec scenario 又要求"code 生成"。README 是同时满足两端的最低成本形态——它是有形的"产物文件"，但不是业务代码。放在 spec delta 目录里是为了让它跟 spec 一起走 archive merge，避免在仓库根引入独立目录污染空仓库结构。
- **备选（已否决）**：零代码（字面落空）、可执行脚本（超范畴）、TDD 测试占位（测试本身依赖后续产物，环内难以跑通）、仓库根独立目录（污染空仓库）。

### D2：失败路径演示 = `/sdd-review` 环临时构造 + PR body 留痕

- **选择**：review 环执行流程里包含一步"临时把 spec 改坏 → 跑 `openspec validate --strict` → 捕获失败输出 → 贴到 PR body → 还原 spec"。
- **理由**：负面路径与 happy path 天然矛盾，必须在主路径之外"借用"一次。临时构造保证证据真实（不是预写），PR body 留痕保证可追溯，还原则保证主分支最终状态正确。
- **备选（已否决）**：独立 e2e 测试脚本（超范畴、维护成本）、纯文档声明（字面未验证）、docs/failure-demo.md 预写（漂移风险、近似伪造）。

### D3：tasks.md 粒度 = 5 任务（spec / plan / task / apply / review+archive）

- **选择**：tasks.md 列 5 个粗任务，与 oh-my-sdd 五环（spec/plan/task/apply/review）一一对应；archive 作为第 5 个任务（review）的收尾子步骤——因为 archive 在 oh-my-sdd 模型里是 review 通过后的自然延伸（`/sdd-archive` 紧跟 `/sdd-review`），不单独占一环。
- **理由**：本 change 实现量极小（一个 README），细粒度拆分会让任务数远超实际工作量；粗粒度与 oh-my-sdd "一环一任务" 的 mental model 对齐，archive 时 checkbox 追踪的就是环级进度。
- **备选（已否决）**：6 任务（独立 archive 任务，但 archive 没有独立产物只是 merge，强行独立会让任务粒度不均）、中粒度（每环拆 2-3 子任务，对本 demo 是过度工程）、细粒度（每动作一任务，部分会比 commit 还小）。

### D4：完成策略 = 单分支贯穿 + 顺序执行

- **选择**：所有 5 环产物在 `002-ard123456` 单分支上按顺序累积，无并行副轨、无 throwaway 分支。
- **理由**：本 change 的 acceptance criteria 就是"流程能跑通"，引入额外分支会污染验证变量。oh-my-sdd 默认模型也是单分支。
- **备选（已否决）**：双分支隔离失败演示（超出范畴）、主轨+文档副轨（预写近似伪造）。

### D5：gh issue/PR 关联 = PR body 写 `Closes #2`

- **选择**：review 环创建 PR 时，body 中明确包含 `Closes #2`，依赖 GitHub 原生自动关闭机制。
- **理由**：这是 GitHub 标准能力，无需额外脚本或 CI 钩子。issue #2 已存在，PR 创建即可关联。
- **备选（已否决）**：commit message convention（分散、易漏）、gh PR 模板（仓库尚无模板基础设施）。

## Risks / Trade-offs

- **[R1] 临时构造的失败演示不可重演** → PR body 留痕是唯一证据。**缓解**：PR 描述里贴完整 validate 失败输出 + 改坏的 diff hunk，未来可凭文本复现；接受"演示即记录"的权衡。
- **[R2] README 作为"代码产物"可能被严格 reviewer 质疑** → spec 字面要求 "code 生成"。**缓解**：design.md D1 明确论证为何 README 满足字面意义；如 review 阶段被否决，回退到最小可执行脚本（已记为 contingency）。
- **[R3] 粗粒度任务可能让 `/sdd-task` 单次推进过大** → 一个 task 覆盖整环。**缓解**：tasks.md 中每个任务描述里写"完成判定"，让 `/sdd-task` 在环内有明确的 done 标准。
- **[R4] 失败演示发生在 review 环，前面环无法预见** → 如果 spec 设计本身让 validate 无法被改坏（如格式过于宽松），review 环临时构造可能失败。**缓解**：Ring 1 spec 已经写明"validate 失败"的可触发条件（删 scenario / 误用 3 个 `#` / capability 名不一致），review 环挑其一即可。
- **[R5] DOP mock 可能在跨环过程中 reset** → 已观测到 DOP status 当前为 `open` 而非 `spec-in-progress`。**缓解**：本环结束时统一推进到 `plan-ready`，不依赖中间状态保持。
