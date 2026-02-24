---
name: superspec-plan
description: Produce an OPSX 1.0 thin-wrapper zero-decision plan with one-question-at-a-time Interview Gate, Oracle convergence loops, and strict OpenSpec validation.
category: superspec
tags: [ superspec, plan, opsx, pbt ]
allowed-tools: Bash(openspec:*), Bash(rg:*), Bash(test:*), Bash(ls:*)
argument-hint: [ change_id ]
---

<!-- SUPERSPEC:PLAN:START -->

# SuperSpec Plan（OPSX 1.0 薄包装）

目标：将计划阶段收敛为 **零决策**，并把实现阶段所需约束固化到 artifacts；禁止把决策延后到 implement。

## Guardrails

- Shell 命令由主代理执行。
- 产物编辑由 writer 角色执行。
- 评审只使用 `@oracle`（只读、参考输出；不得写文件）。
- 目标是消灭所有决策点，implement 必须可机械执行。
- 任何无法确定的约束必须回问用户，禁止猜测。
- 禁止把旧冲突台账文件作为真值来源。
- 禁止依赖旧 markdown linter 脚本。
- 每个任务必须使用 `test-driven-development` skill；`TDD:` 字段不得为空或 SKIP。

## Required Files（Plan 阶段）

- `openspec/changes/<change_id>/proposal.md`
- `openspec/changes/<change_id>/tasks.md`
- `openspec/changes/<change_id>/design.md`

## 新真值机制（替代旧冲突台账）

### A) `design.md` 必须包含 `## Open Questions Ledger`

使用 OPEN/RESOLVED 维护问题台账，作为 plan 阶段唯一开放问题真值来源。

模板：

```md
## Open Questions Ledger

- id: Q-001
  status: OPEN|RESOLVED
  owner: user|primary-agent
  question: <待决策问题>
  decision: <RESOLVED 时填写明确选择+参数>
  resolved_in: <时间戳或回合标记，如 Interview-3 / Oracle-2>
  verify: <复核命令>
```

### B) `tasks.md` 每个任务必须新增字段

每个任务块必须含有：

- `Decision Source: Interview Gate|@oracle|User Direct`
- `Resolved In: <Interview-x|Oracle-x|User-y>`

并继续保留：`Files:` / `Acceptance Criteria:` / `TDD:` / `Verify:`（及 `Verify (RED):` 若 TDD REQUIRED）。
> **硬性约束**：`TDD:` 字段必须填写，且必须声明使用 `test-driven-development` skill（格式：`REQUIRED — skill: test-driven-development`）。空值或 `SKIP`
> 视为校验失败。

## Steps

0. **Resolve `change_id`**

- 若未提供 `change_id`：
    1) 运行 `openspec list --json`
    2) 让用户选择
    3) 使用选中的 `<change_id>` 继续

1. **前置校验（严格）**

- 运行：`openspec validate <change_id> --strict --no-interactive --json`
- 若 `--strict` 不支持：运行 `openspec --help` 后降级为 `openspec validate <change_id> --no-interactive --json`，并在输出中显式说明降级。
- 若校验失败且为结构性缺失：先回到 research 修复。

2. **读取 artifacts 状态（OPSX 指令源）**

- 运行：`openspec instructions apply --change <change_id> --json`
- 解析当前 proposal/tasks/design 的缺口，形成 Interview Gate 问题候选。

3. **Interview Gate（硬约束）**

- 必须一次一问（每条消息只允许一个问题）。
- 必须至少 5 问（至少 5 个已回答问题）。
- 推荐多选（2-4 选项 + Other）。
- 只在 `design.md` 的 `Open Questions Ledger` 中 `owner=user` 的 OPEN 项为 0 时结束。
- 若用户拒绝回答必需问题：立即停止，不得继续 plan。

每次回答后必须立即固化：

- 更新 `design.md`：
    - Decision Log（明确选择与参数）
    - `## Open Questions Ledger` 对应项改为 RESOLVED
- 更新 `tasks.md`：
    - 补齐/修正 `Files`、`Acceptance Criteria`、`TDD`、`Verify`
    - `TDD:` 字段必须填写为 `REQUIRED — skill: test-driven-development`（不得为空或 SKIP）
    - 填写 `Decision Source`、`Resolved In`
- 运行严格校验：`openspec validate <change_id> --strict --no-interactive --json`

4. **@oracle 多轮收敛（>=5 轮）**

- 至少执行 5 轮 `@oracle` 约束收敛审查（可超过 5 轮直到收敛）。
- 调用要求固定：`Return Unified Diff Patch only; do not modify files`。
- 每轮流程：
    1) `@oracle` 输出约束补丁建议（只读）
    2) 主代理归一化为可执行约束
    3) writer 更新 `design.md` 与 `tasks.md`
    4) 将新发现问题写入 `Open Questions Ledger`（OPEN/RESOLVED）
    5) 运行 `openspec validate <change_id> --strict --no-interactive --json`

收敛判定（全部满足）：

- 已完成 >=5 轮 @oracle 收敛
- `Open Questions Ledger` 中 OPEN=0
- 严格校验通过

5. **PBT 属性提取（必须保留）**

- 对每条核心需求补齐 PBT 属性：
    - `INVARIANT`
    - `FALSIFICATION STRATEGY`
- 建议覆盖：幂等性、交换/结合、round-trip、不变量保持、单调性、边界约束。
- PBT 结论记录进 `design.md`，并将对应任务的 `Decision Source` / `Resolved In` 回填到 `tasks.md`。

6. **最终严格校验（交接前硬门槛）**

- 运行：`openspec validate <change_id> --strict --no-interactive --json`
- 且人工检查：
    - `design.md` 存在 `## Open Questions Ledger`
    - Ledger 中 OPEN=0
    - `tasks.md` 每个任务都有 `Decision Source`、`Resolved In`

## Anti-Patterns（Reject）

- 批量提问替代 Interview Gate（违例）。
- 少于 5 轮 @oracle 就声称收敛（违例）。
- 继续使用旧冲突台账文件作为开放问题真值（违例）。
- 依赖旧 markdown linter 门禁（违例）。
- `tasks.md` 中任意任务的 `TDD:` 字段为空、缺失或含 `SKIP`（违例）。

## Exit Criteria

- `openspec validate <change_id> --strict --no-interactive --json` 通过。
- Interview Gate 满足：一次一问，且至少 5 问已回答。
- `@oracle` 收敛满足：>=5 轮并完成收敛。
- `design.md` 的 `Open Questions Ledger`：OPEN=0。
- `tasks.md` 任务字段完整（含 `Decision Source` / `Resolved In`）。
- `tasks.md` 所有任务的 `TDD:` 字段均为 `REQUIRED — skill: test-driven-development`。

## Exit Report（Standard）

- Active change: `<change_id>`
- Gates: `openspec validate <change_id> --strict --no-interactive --json`, Interview Gate(一次一问, 至少5问), @oracle收敛(>=5轮), Open Questions
  Ledger OPEN=0
- Next: `/superspec-implement <change_id>`

<!-- SUPERSPEC:PLAN:END -->
