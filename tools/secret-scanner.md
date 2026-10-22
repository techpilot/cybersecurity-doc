# Secret Scanner

Python tool for catching hardcoded secrets before they leave a machine or a pipeline — runs as a local pre-commit hook and in CI/CD.

**Repo:** <link once pushed>

## Why I built it

<one line — what problem this closes, e.g. catching an API key before it hits a commit history>

## How it works

- Detection: <regex pattern matching for known secret formats / entropy-based detection for high-randomness strings — confirm which you used>
- Integration: pre-commit hook (`.pre-commit-hooks.yaml`) + CI/CD step (<GitHub Actions / other — confirm>)
- Scope: <scans staged diffs only? full repo? confirm>
