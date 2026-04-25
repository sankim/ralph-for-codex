# Changelog

## Unreleased

- Added best-effort local Git ignore via `.git/info/exclude` for `.ralph/` working state
- Added terminal-state cleanup rules so `.ralph/` is deleted for `complete`, `stalled`, and `emergency-cap reached`
- Preserved `.ralph/` on `blocked` so interrupted runs can resume cleanly

## v0.1.0

- Initial public release of `Ralph for Codex`
- Added explicit `$ralph` Codex skill
- Added strict `.ralph/` file-backed execution protocol
- Added guardrailed terminal states: `complete`, `blocked`, `stalled`, and `emergency-cap reached`
