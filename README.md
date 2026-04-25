# Ralph for Codex

`$ralph` is a Codex-native, Ralph-style execution skill for turning an existing implementation plan into a strict file-backed loop.

It is inspired by the Ralph Wiggum approach, but it is not the external Open Ralph Wiggum CLI. This package is a Codex skill that uses repo-local `.ralph/` state files as temporary working memory so execution stays focused and stopping behavior stays predictable.

## When To Use It

Use `$ralph` after a plan already exists in the thread.

Good fits:

- you already have a `<proposed_plan>` and want Codex to implement it
- you want small, low-risk, verifiable slices
- you want progress tracked in `.ralph/` instead of relying on a long chat history

Not a fit:

- planning from scratch
- open-ended brainstorming
- replacing the external Ralph CLI supervisor

## Install

Install with Codex's built-in GitHub skill installer:

```bash
python ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo sankim/ralph-for-codex \
  --path skills/ralph
```

URL form also works:

```bash
python ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --url https://github.com/sankim/ralph-for-codex/tree/main/skills/ralph
```

If your Python environment hits HTTPS certificate issues during the default download path, force git mode:

```bash
python ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo sankim/ralph-for-codex \
  --path skills/ralph \
  --method git
```

Restart Codex after installing so the new skill is discovered.

## Usage

Use `$ralph` after an implementation plan already exists in the thread.

Examples:

- `$ralph`
- `$ralph focus on the riskiest remaining small slice first`
- `$ralph use this revised execution brief instead: ...`

## State Files

During execution, `$ralph` uses repo-local files under `.ralph/`:

- `.ralph/source-plan.md`
- `.ralph/progress.md`
- `.ralph/scratchpad.md`

These files act as the working memory for the active run.

When possible, `$ralph` adds `.ralph/` to the current repo's local `.git/info/exclude` so the working files stay out of `git status` without editing the repo's tracked `.gitignore`.

When the run ends:

- `complete`: `.ralph/` is deleted
- `stalled`: `.ralph/` is deleted
- `emergency-cap reached`: `.ralph/` is deleted
- `blocked`: `.ralph/` is preserved so the next run can resume

## Terminal States

The loop stops only in one of these states:

- `complete`
- `blocked`
- `stalled`
- `emergency-cap reached`

## Limitations

- This package is a Codex skill, not an external process supervisor
- It expects an existing implementation plan
- It does not auto-commit by default
- Auto-ignore is best-effort and depends on Git metadata being available

## License

MIT
