---
name: superspec-init
description: Initialize OpenSpec for this repo and validate toolchain + reviewer availability.
category: superspec
tags: [superspec, init, openspec, setup, opencode]
allowed-tools: Bash(openspec:*)
---

<!-- SUPERSPEC:INIT:START -->

**Guardrails**

- Shell commands MUST be executed by the primary agent, not by `task(category=quick|deep)`.
- All code/doc edits MUST be executed by `task(category=quick|deep)`.
- `@oracle` is reference-only; it MUST NOT edit files; it MAY run shell commands if needed for review.
- Do not proceed to the next step until the current step completes successfully.
- Do not overwrite existing `openspec/` without explicit confirmation.
- Do not attempt `sudo` unless the user explicitly asks.
- If OpenSpec is not available: attempt automatic installation using the appropriate method for the detected operating system.

**Shared Hard Guardrails (Progressive Strictness)**

- Disallowed terms (lint targets): `TODO`, `TBD`, `decide later`, `we'll implement later`, `it depends`, `as needed`, `we'll decide later`.
- Progressive strictness:
  - research: warn-only (never blocks for disallowed terms / open items)
  - plan: converge to zero-decision before handoff
  - implement: hard-fail gates (zero-decision + scope audit)
- When working on a change, verification commands MUST come from `tasks.md` (do not invent new verify steps during implement).
- Scope rule: any change outside `tasks.md` declared `Files:` MUST stop and be logged as `type: NEXT-CHANGE` (not implemented in current change).
- Each stage MUST print the standard Exit Report (Active change / Gates / Next).

**Steps**

1. **Detect Operating System**
   - Run: `uname -s`
   - Report the detected OS and any required command adaptations.

2. **Ensure OpenSpec CLI Is Available**
   - Run: `openspec --version`
   - If the command succeeds: treat OpenSpec as already installed and continue.
   - If `openspec` is missing:
     - First, check if Node.js is available and meets version requirements:
       - Run: `node --version`
       - If Node.js is missing or version is below 20.19.0: stop and ask the user to install Node.js 20.19.0 or higher.
     - Attempt automatic installation:
       - Run: `npm install -g @fission-ai/openspec@latest`
     - If automatic installation fails: stop and ask the user to install OpenSpec manually.

3. **Initialize OpenSpec in This Repo (OpenCode Integration Required)**
   - If `openspec/` already exists: ask the user before re-running init.
   - Prefer agent-compatible, non-interactive init:
     ```bash
     openspec init --tools opencode
     ```
   - Verify `openspec/` directory exists.
   - Verify the CLI can return agent-readable JSON:
     ```bash
     openspec list --json
     ```

4. **Update OpenSpec Tool Bindings**
   - Run: `openspec update`
   - Verify schema discovery is agent-readable:
     ```bash
     openspec schemas --json
     ```

5. **Summary Report**
   - Print a checklist-style report:
     - OpenSpec installation: ✓/✗ (`openspec --version`)
     - Project initialization: ✓/✗ (`openspec/` exists)
     - Config file: ✓/✗ (`openspec/config.yaml` exists)
     - Agent JSON: ✓/✗ (`openspec list --json`, `openspec schemas --json`)

**Reference**

- Use `openspec --help` if command forms differ across versions.

**Exit Criteria**

- `openspec/` exists.
- `openspec --version` succeeds.
- `openspec update` succeeds (or the user explicitly chose to skip).

**Exit Report (Standard)**

- Active change: N/A
- Gates: `openspec --version`, `openspec update`
- Next: `/superspec-research "<full requirements text>"`
<!-- SUPERSPEC:INIT:END -->
