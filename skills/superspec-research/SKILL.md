---
name: superspec-research
description: Transform requirements into constraint sets and create an OpenSpec change via OPSX 1.0 flow.
category: superspec
tags: [superspec, research, openspec, constraints, opsx]
allowed-tools: Bash(openspec:*)
argument-hint: [requirements_text]
---

<!-- SUPERSPEC:RESEARCH:START -->

**Core Philosophy**

- Research produces constraint sets, not information dumps.
- Constraints eliminate decision points later; implementation should be mechanical.
- Do not make architectural decisions; capture constraints, dependencies, risks, and explicit open questions.
- OPSX 1.0 handles change scaffolding; SuperSpec adds exploration, synthesis, and quality guardrails.

**Guardrails**

- Shell commands MUST be executed by the primary agent (or delegated via `task(category=quick)` only when execution delegation is explicitly needed).
- If requirements text is missing, STOP and ask the user for the full requirements.
- If requirements are vague: produce open questions and use `AskUserQuestion` to resolve them before creating a change.
- Keep all generated artifacts under `openspec/changes/<change_id>/...` (created by OpenSpec command flow, not manual scaffold).
- Change id format: `kebab-case` (ASCII `a-z0-9-`, e.g. `add-auth`).
- Change id MUST be stable and human-chosen (no timestamps/hashes).
- Use efficient codebase retrieval/search where possible; avoid noisy full-repo scans.
- Subagent division MUST be by context boundary (module/area), NOT by role.
- Each boundary must be self-contained; subagent outputs must not depend on other boundaries.
- DO NOT run Interview Gate in research. Interview Gate belongs to plan stage only.

**Shared Hard Guardrails (Progressive Strictness)**

- Disallowed terms (lint targets): `TODO`, `TBD`, `decide later`, `we'll implement later`, `it depends`, `as needed`, `we'll decide later`.
- Interpretation:
  - research: these terms are allowed as structured placeholders but treated as WARN-level debt
  - plan/implement: these terms are NOT allowed and must hard-fail
- Progressive strictness:
  - research: warn-only (never blocks for disallowed terms / open items)
  - plan: converge to zero-decision before handoff
  - implement: hard-fail gates (zero-decision + scope audit)
- When working on a change, verification commands MUST come from `tasks.md` (do not invent new verify steps during implement).
- Scope rule: any change outside `tasks.md` declared `Files:` MUST stop and be logged as `type: NEXT-CHANGE` (not implemented in current change).
- Each stage MUST print the standard Exit Report (Active change / Gates / Next).

**Steps (OPSX 1.0 Thin Wrapper)**

1. **Validate Inputs**
   - If `$ARGUMENTS` is empty:
     - Ask the user for the full requirements text.
     - Stop here.

2. **Preflight: OpenSpec Initialized**
   - If `openspec/` is missing: instruct the user to run `/superspec-init` first, then retry.
   - Run: `openspec --version` (must succeed).

3. **Initial Codebase Assessment**
   - Locate likely entrypoints for the requirement, relevant modules/directories, and existing conventions.
   - Decide whether the codebase is single-context or multi-context.

4. **Define Exploration Boundaries (Context-Based)**
   - Identify 2-4 context boundaries (examples: user domain, auth, infra/config).
   - For each boundary, define scope and expected findings.

5. **Parallel Exploration (Required for Multi-Context)**
   - If multiple boundaries are identified: you MUST spawn `@explore` subagents in parallel, one per boundary.
   - Recommended dispatch form: `task(category=deep, subagent_type=explore, ...)` per boundary.
   - Each subagent MUST output JSON following this schema:
     ```json
     {
       "module_name": "string",
       "existing_structures": ["..."],
       "existing_conventions": ["..."],
       "constraints_discovered": ["..."],
       "open_questions": ["..."],
       "dependencies": ["..."],
       "risks": ["..."],
       "success_criteria_hints": ["..."]
     }
     ```

6. **Validate Subagent Outputs (Schema Compliance)**
   - Verify each subagent output conforms to the JSON schema.
   - If any output is missing fields or not parseable: re-run that boundary exploration.

7. **Aggregate and Synthesize Constraint Sets**
   - Merge findings into:
     - Hard constraints (MUST / MUST NOT)
     - Soft constraints (conventions/preferences)
     - Dependencies (ordering/coupling)
     - Risks
     - Open questions
     - Verifiable success criteria
   - Ensure proposal-compatible format:
     - Constraints use MUST/MUST NOT bullets
     - Success criteria are commandable/verifiable checklists
   - Preserve SuperSpec TDD contract expectations for downstream planning:
     - `TDD: REQUIRED` needs RED/GREEN verify commands in plan stage
     - `TDD: SKIP` only for DOC/CONFIG/GENERATED with explicit reason

8. **Resolve Ambiguities with User (Tool-Driven)**
   - Use `AskUserQuestion` to resolve open questions before change creation whenever possible.
   - Convert user answers into explicit constraints.
   - Any still-open item must be explicitly owned by user.

9. **Generate and Confirm `<change_id>`**
   - Derive a short kebab-case candidate from requirement text.
   - Ask user to confirm.
   - If not accepted, ask for a different short kebab-case id and reconfirm.

10. **Create Change via OpenSpec (No Manual Scaffold)**
   - Run:
     ```bash
     openspec new change <change_id>
     ```
   - This command is the only allowed scaffold mechanism in research.
   - Do NOT use manual `mkdir` / direct file bootstrap flow.

11. **Generate Artifacts via OPSX Commands (Sequential)**
   - Use `/opsx-continue` or equivalent `openspec instructions` invocation to generate/update artifacts in order:
     1) `proposal.md`
     2) `specs/<capability>/spec.md`
     3) `design.md`
     4) `tasks.md`
   - Apply synthesized SuperSpec constraints into each artifact.
   - Keep research-stage placeholders structured and minimal where unresolved details remain.

12. **Initialize Reviews Artifacts**
   - Ensure `reviews/` contains at least:
     - `backend.md`
     - `frontend.md`
   - If conflict ledger is required by downstream workflow, initialize `reviews/conflicts.md` with the standard template.

13. **Validate Change**
   - Run:
     ```bash
     openspec validate <change_id> --no-interactive --json
     ```
   - If invalid, surface errors and fix via OPSX artifact update flow, then re-validate.

**Reference**

- Use `openspec --help` to confirm command syntax in current CLI version.
- OPSX 1.0 preferred flow: create change with `openspec new change`, then generate artifacts through `openspec instructions` / `/opsx-continue`.

**Exit Criteria**

- Change is created via `openspec new change <change_id>`.
- Artifacts are generated via OPSX flow (not manual scaffold).
- Constraint sets and success criteria are captured.
- `openspec validate <change_id> --no-interactive --json` passes.
- Return `<change_id>` to the user.

**Exit Report (Standard)**

- Active change: `<change_id>`
- Gates: `openspec validate <change_id> --no-interactive --json`
- Next: `/superspec-plan <change_id>`
<!-- SUPERSPEC:RESEARCH:END -->
