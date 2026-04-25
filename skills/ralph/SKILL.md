---
name: ralph
description: Execute an existing implementation plan through a strict Ralph-style iterative loop with repo-local state files instead of growing chat history. Use when the user invokes `$ralph` after planning is already done and wants Codex to implement in small, low-risk, verifiable slices until the work reaches `complete`, `blocked`, `stalled`, or `emergency-cap reached`.
---

# Ralph

## Overview

Use this skill to turn an existing plan into a strict file-backed execution loop. Treat the chosen plan as the source of truth, keep local working memory on disk in `.ralph/` for the active run, and iterate in the smallest appropriate implementable slice rather than carrying the whole thread as memory.

This is an execution skill, not a planning skill.

## Plan Source

Resolve the execution brief in this order:

1. If the `$ralph` invocation includes an explicit plan or replacement execution brief, use that.
2. Otherwise use the latest `<proposed_plan>` block in the current thread.
3. If neither exists, stop and tell the user to create or paste a plan first.

Normalize the chosen plan into `.ralph/source-plan.md`.

Do not reopen settled plan decisions unless current repo truth or validation results directly contradict the plan.

## Memory Model

Initialization may use the current thread to extract the chosen plan and any explicit constraints in the active `$ralph` invocation.

After `.ralph/source-plan.md` and `.ralph/progress.md` exist, do not rely on older thread details except:

- the active `$ralph` invocation
- any explicit plan override in that invocation

Use `.ralph/` files as the only durable working memory layer during the active run. They are temporary local execution state, not permanent repo artifacts.

## Initialization And Git Ignore

Before relying on `.ralph/` state, make a best-effort attempt to keep it out of Git status in the current repo.

1. If the current working directory is inside a Git repo, resolve that repo's Git directory.
2. If `.git/info/exclude` is available and does not already contain `.ralph/`, append exactly one `.ralph/` ignore entry there.
3. Do not edit the repo's tracked `.gitignore` just to hide Ralph working state.
4. If the repo is not Git-backed or `.git/info/exclude` cannot be updated, continue without failing the run and note that auto-ignore could not be applied.

## State Files

Create and maintain exactly these repo-local files:

- `.ralph/source-plan.md`
- `.ralph/progress.md`
- `.ralph/scratchpad.md`

Do not create other persistent `.ralph/` files unless the user explicitly asks for them.

### `.ralph/source-plan.md`

Write a normalized copy of the chosen plan with these sections:

- `Goal`
- `Acceptance Criteria`
- `Constraints`
- `Assumptions And Defaults`

Keep this file stable. Rewrite it only if the user explicitly replaces the plan.

### `.ralph/progress.md`

Use this as the durable compressed state for re-entry. Keep exactly these sections:

- `Goal`
- `Constraints`
- `Acceptance Criteria`
- `Completed`
- `Verified`
- `Open Issues`
- `Next Slice`
- `Loop Status`
- `Last Pass Result`

Under `Last Pass Result`, record:

- `Repo State Change`: `yes` or `no`
- `Acceptance Movement`: `yes` or `no`
- `Validation Result`: `passed`, `failed`, `blocked`, or `not-run`
- `Pass Outcome`: short factual summary

Rewrite this file fully after each pass.

- never append a running history
- preserve only information needed for the next pass, acceptance tracking, or final summary
- preserve verified outcomes and active constraints
- remove stale narration, dead ends, and no-longer-relevant detail

Set `Loop Status` to only one of:

- `active`
- `complete`
- `blocked`
- `stalled`
- `emergency-cap reached`

### `.ralph/scratchpad.md`

Use this for the current pass only. It is ephemeral, not durable memory.

Overwrite it fully each pass with exactly this micro-plan structure:

- `Current Slice`
- `Why This Slice`
- `Files Or Systems`
- `Validation`
- `Stop Condition`

Do not let this file become a running log. Do not carry history forward here.

## Loop Workflow

After initialization, treat `.ralph/source-plan.md` and `.ralph/progress.md` as the authoritative memory, not the accumulated thread.

For each pass:

1. Read `.ralph/source-plan.md` and `.ralph/progress.md`.
2. Read only the repo files relevant to the current incomplete work.
3. Choose the next slice using this rule:
   - pick the smallest independently implementable and verifiable slice that materially advances the source plan
   - prefer slices with a clear validation path
   - prefer slices that touch the fewest files or systems
   - prefer slices that reduce uncertainty or unblock later work
   - avoid speculative refactors or broad cleanup unless directly required
4. Write the micro-plan for that slice into `.ralph/scratchpad.md` before implementation work begins.
5. Implement only that slice plus directly required glue.
6. Run targeted validation for that slice.
7. Rewrite `.ralph/progress.md` as a compact, current summary of repo truth.
8. Overwrite `.ralph/scratchpad.md` before the next pass.

Every pass must explicitly account for:

- whether repo state changed
- whether acceptance criteria moved forward
- whether validation passed, failed, was blocked, or was not run
- what the next smallest appropriate slice is

Continue looping until the work reaches a terminal state.

## Compaction And Reset

Compaction rule:

- if a fact is not needed for the next pass, acceptance tracking, or final summary, remove it from persistent state

Reset rule:

- if `.ralph/progress.md` becomes noisy, contradictory, stale, or too messy to drive the next pass, regenerate it cleanly from:
  - `.ralph/source-plan.md`
  - current repo truth
  - latest validation results

Use reset to recover a clean working state instead of carrying forward distorted summaries.

## Terminal States

The loop can stop only in one of these terminal states:

- `complete`
- `blocked`
- `stalled`
- `emergency-cap reached`

### `complete`

Stop as `complete` only when:

- all acceptance criteria are satisfied
- relevant validation for the completed work passed
- no known required work remains from the source plan

### `blocked`

Stop as `blocked` only when:

- a required user decision is missing
- a credential, service, environment, or artifact is unavailable
- an unresolved contradiction cannot be solved from repo truth

### `stalled`

Stop as `stalled` only when:

- two consecutive passes produce no meaningful repo-state change
- and those same two passes produce no movement on acceptance criteria

### `emergency-cap reached`

Stop as `emergency-cap reached` only when:

- twelve passes have been attempted without reaching another terminal state

## Cleanup

When the loop stops, `.ralph/` should not linger unless the run needs to be resumed.

- `complete`: after deriving the final summary from `.ralph/progress.md`, delete the entire `.ralph/` directory
- `stalled`: after deriving the final summary from `.ralph/progress.md`, delete the entire `.ralph/` directory
- `emergency-cap reached`: after deriving the final summary from `.ralph/progress.md`, delete the entire `.ralph/` directory
- `blocked`: preserve `.ralph/` so the next `$ralph` invocation can resume from the saved state

If cleanup fails for a non-blocked terminal state, report that clearly in the final summary.

## Validation

Prefer repo-native validation in this order:

1. existing targeted tests
2. targeted build or static checks
3. only broader validation if directly necessary

Avoid unrelated repo-wide checks that do not help decide the current slice.

## Reporting

When the loop stops, derive the final summary from current `.ralph/progress.md`, not from freeform recollection.

Return a compact summary with:

- what was completed
- what was verified
- what remains
- why the loop stopped
- whether a hard reset was used
- whether local Ralph state was cleaned up or intentionally preserved

For non-blocked terminal states, derive the summary first, then clean up `.ralph/`, then return the final summary. For `blocked`, preserve `.ralph/` and say that the local Ralph state was preserved because the run is blocked.

Do not produce a per-pass diary unless the user explicitly asks for it.

## Defaults

- No auto-commit
- Explicit invocation only via `$ralph`
- File-backed state is the durable working memory layer
- `.ralph/` should be auto-ignored via `.git/info/exclude` when possible
- `.ralph/` should be cleaned up on final non-blocked terminal states
- After initialization, older thread context is not durable memory

## Example Invocations

- `$ralph`
- `$ralph focus on the riskiest remaining small slice first`
- `$ralph use this revised execution brief instead: ...`
