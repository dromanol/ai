# global knowledge base (cross-project)

Reusable, **non-obvious** knowledge that applies regardless of the repository you're working in. One
entry per file under `entries/`. This index is auto-loaded into context at session start (every
session, everywhere) and grows via `/knowledge-base save`. Keep it curated and concise — prune stale
entries. Repo-specific knowledge belongs in that repo's own `/project-knowledge` store, not here.

## Entries
- [Never be complacent](entries/never-complacent.md) — no flattery or blind compliance: be critical with reasons, ask what's in doubt, verify instead of assuming
- [Keep all output concise](entries/concise-generated-text.md) — PRs, commits, comments, **and session replies/reports**: lead with the conclusion, trim words not facts
- [Minimal code comments](entries/minimal-code-comments.md) — default to zero comments; only a *why* the code can't show, one line
- [GitHub token via ddtool](entries/github-token-ddtool.md) — `ddtool auth github token` (defaults to DataDog org); first use needs `ddtool auth github login --org DataDog`
- [SSH rejected by SAML SSO; two GitHub accounts](entries/github-ssh-saml-sso.md) — re-authorize the key for the org; `github.com` = dromanol/DataDog, `ddoghq.github.com` = _ddog/ddoghq; dd-go lives in ddoghq now; `gh` per account via `GH_TOKEN=$(gh auth token --user <acct>)`
- [Benchmark regression triage](entries/benchmark-regression-triage.md) — cross-PR history + %→ns/op arithmetic tell a real CI regression from a miscalibrated baseline
- [Bash tool is not PowerShell](entries/bash-tool-vs-powershell-syntax.md) — PowerShell here-strings `@'…'@` corrupt the argument silently in Bash; use a heredoc and verify
- [Signed commit troubleshooting](entries/signed-commit-troubleshooting.md) — 1Password failures are transient: retry, never bypass signing; "No signature" is a verification gap
- [Editing repo files from scripts on Windows](entries/windows-script-file-edits.md) — Bash `/tmp` != Windows-Python `/tmp`; preserve the original BOM and CRLFs
- [PR review threads via `gh api`](entries/github-pr-review-via-cli.md) — three endpoints, anchor on `diff_hunk`, reply with `in_reply_to`, never rename the head branch
- [Always link references](entries/always-link-references.md) — never mention a PR or ticket without a link; link each mention, in every kind of text
- [Worktrees, not branch checkouts](entries/git-worktrees-not-checkout.md) — GitHub repos: each branch in `git worktree add` under `<repo>.worktrees\<leaf>`; main clone stays on default branch
- [1Password CLI secrets](entries/1password-cli-secrets.md) — inject keys with one `op run` + `op://` refs, never print; prompt repeats per process; org 2 keys in `Employee`
