---
name: superspec-implement
description: Execute tasks.md mechanically via OPSX apply with strict TDD contract and scope audit.
category: superspec
tags: [superspec, implement, opsx]
allowed-tools: Bash(opsx:*), Bash(openspec:*), Bash(git:*), Bash(rg:*), Bash(test:*)
argument-hint: [change_id]
---

<!-- SUPERSPEC:IMPLEMENT:START -->

**Guardrails**

- Shell commands MUST be executed by the primary agent.
- All write actions MUST stay within `openspec/changes/<change_id>/tasks.md` declared scope.
- Implement in the smallest verifiable phase; do not batch all tasks at once.
- Verification commands MUST be copied from `tasks.md` verbatim.
- Implement stage MUST NOT invent verification commands.
- Each stage MUST print the standard Exit Report (Active change / Gates / Next).

**Shared Hard Guardrails (Progressive Strictness)**

- Disallowed terms (strict): `TODO`, `TBD`, `decide later`, `we'll implement later`, `it depends`, `as needed`, `we'll decide later`.
- Progressive strictness:
  - research: warn-only
  - plan: zero-decision converge
  - implement: hard-fail gates (strict validate + scope audit)
- Scope rule: any out-of-scope file is a `NEXT-CHANGE` and MUST NOT be implemented in current change.

**Steps (OPSX 1.0 thin wrapper)**

1. **Resolve `change_id`**
   - If missing, run `openspec list --json` and let user pick one.

2. **OPSX Apply Entry**
   - Run: `opsx apply --change <change_id> --phase implement --json`
   - If `opsx` is unavailable, fallback:
     - `openspec status --change <change_id> --json`
     - `openspec instructions apply --change <change_id> --json`

3. **Strict Preflight Gate**
   - Run: `openspec validate <change_id> --strict --no-interactive --json`
   - If validate fails: stop and return to `/superspec-plan <change_id>`.

4. **Execute One Small Phase**
   - Read `tasks.md`, pick the smallest unchecked item set.
   - Implement only files declared in the selected task block `Files:`.
   - Collect verify commands from that same task block only.
   - **TDD contract (must keep):**
     - `TDD: REQUIRED` → run `Verify (RED)` first (expect fail), then implement, then run `Verify` (GREEN, expect pass).
     - `TDD: SKIP` → only allowed with `TDD Skip Reason: DOC|CONFIG|GENERATED`.
   - If verification fails, fix within same scope and re-run the same task-defined verify commands.

5. **Scope Audit (Hard Gate)**
   - Run: `git diff --name-only`
   - Gate rule: every changed non-OpenSpec-managed file MUST exist in `tasks.md` `Files:` whitelist.
   - If any file is outside `Files:`:
     - Stop current implementation.
     - Record those paths under `Deferred Scope` section in `openspec/changes/<change_id>/tasks.md` as `NEXT-CHANGE`.
     - Do not expand current task scope.

6. **Phase Review**
   - Review all changes in this phase against the task block spec and the overall `tasks.md` intent.
   - Classify each finding:
     - **Fixable** (implementation bug, missing logic, style violation): fix immediately within the same scope, re-run verify + scope audit, then re-review.
     - **Spec conflict** (implementation contradicts or is ambiguous against the spec): STOP. Present the conflict clearly to the human:
       ```
       SPEC CONFLICT DETECTED
       Task: <task block name>
       Conflict: <what the implementation does vs what the spec says>
       Options:
         A) <option A>
         B) <option B>
       Awaiting your decision before proceeding.
       ```
     - Wait for human decision. Apply the chosen resolution, then re-run verify + scope audit + review.
   - Review passes only when no fixable issues remain and all spec conflicts are resolved.

7. **Phase Commit**
   - After review passes, commit all staged changes for this phase:
     - `git add` only files in the current task block `Files:` whitelist.
     - Commit message format: `[<change_id>] <task block name>: <one-line summary>`
   - Do NOT squash or amend previous phase commits.

8. **Mark Task Completion**
   - After commit succeeds, check corresponding `- [x]` items in `tasks.md`.

9. **Repeat Until All Checked**
   - Loop steps 4-8 until no unchecked task remains.

10. **Archive**
    - Run: `openspec validate <change_id> --strict --no-interactive --json`
    - Run: `openspec archive <change_id> --yes`

**Exit Criteria**

- Archive succeeds and specs are updated.
- All checks in `tasks.md` are completed.

**Exit Report (Standard)**

- Active change: `<change_id>`
- Gates: `opsx apply --change <change_id> --phase implement --json`, `openspec validate <change_id> --strict --no-interactive --json`, scope audit (`git diff --name-only` vs `tasks.md` `Files:`)
- Next: `openspec archive <change_id> --yes`
<!-- SUPERSPEC:IMPLEMENT:END -->
