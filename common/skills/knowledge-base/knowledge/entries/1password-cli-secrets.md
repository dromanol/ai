---
name: 1password-cli-secrets
description: Inject secrets from 1Password into a command with `op run` and `op://` references, without printing them; why the approval prompt repeats; how to fix "cannot connect to 1Password app"
type: howto
---

Use the 1Password CLI (`op`) to give a script an API key without the key passing through the conversation,
a file or the shell history.

**Inject, never print:**

```bash
DD_API_KEY="op://<vault>/<item>/<field>" DD_APP_KEY="op://<vault>/<item>/<field>" \
  op run --account datadog.1password.com -- bash script.sh
```

`op run` resolves the references only in the environment of that process and masks the values if they
appear in its output. Never run `op item get ... --fields credential` or `op read` from an agent: it prints
the secret. To inspect an item, print only the field names and lengths:
`op item get <id> --format json | python -c '... print(f["id"], f["type"], len(f.get("value") or ""))'`.
API key = 32 chars, App key = 40 chars.

Known items: the org 2 (Datadog HQ) keys are in vault `Employee`, items `DD_API_KEY` and `DD_APP_KEY`, field
`credential`. Before you use them, check the org with read-only calls in the same `op run`:
`GET /api/v1/validate` (API key only) and `GET /api/v2/current_user` (included `orgs` = "Datadog HQ").

**The approval prompt repeats.** With the desktop app integration, 1Password authorizes one terminal session
(a process and its children) for ~10 min idle / 12 h max. Each Claude Code Bash tool call is a new process,
so each `op` call can prompt again. Put all the work in **one** `op run`. `op signin` exits 0, but a later
`op whoami` still says "not signed in", because it is a different process. Test the session with a real
command (`op vault list`) in the same call. A service account token removes the prompts, but an admin must
create it, and the token is another long-lived secret.

**Errors:**
- `cannot connect to 1Password app, make sure it is running`: unlock the app, and turn on
  Settings → Developer → "Integrate with 1Password CLI". Install on Windows with
  `winget install AgileBits.1Password.CLI`.
- `account is not signed in`: the app is reachable, but this process has no session. Run the real command,
  and the user approves the prompt.
- `authorization prompt dismissed`: the user closed the prompt or it timed out. Retry once and ask the user
  to approve.

Related: [[signed-commit-troubleshooting]] (the 1Password SSH agent for git signing).
