---
name: github-ssh-saml-sso
description: Git over SSH fails with "The '<org>' organization has enabled or enforced SAML SSO" → re-authorize that key for the org; two GitHub accounts (dromanol for DataDog, _ddog for ddoghq) each need their own SSH host alias. dd-go now lives in ddoghq.
type: gotcha
---

Symptom: a push or clone fails with

```
ERROR: The '<org>' organization has enabled or enforced SAML SSO.
To access this repository, you must use ... SSH with an SSH key ... that has been authorized for this organization.
```

Misleading signals, because authentication itself works:
- `ssh -T git@github.com` succeeds ("Hi <user>!").
- `git ls-remote` / fetch on **public** repos (e.g. dd-trace-dotnet) still work; only writes are rejected.
  Private repos fail outright, even on clone.

Cause: the SSH key isn't SSO-authorized for that org (the authorization or session expired, or it was
never granted).

Fix (the user does this in the browser, **logged in with the account that owns the key**):
1. https://github.com/settings/keys
2. Find the key the remote uses (match fingerprint: `ssh-keygen -lf <IdentityFile>` from `~/.ssh/config`).
3. **Configure SSO** → **Authorize** for the org. If it already shows authorized, deauthorize and re-authorize.

`git push --dry-run` reproduces the error without pushing.

**Two accounts, two host aliases.** This machine has two GitHub identities, selected by SSH host:

| Host alias | Account | Key | Orgs |
|---|---|---|---|
| `github.com` | `dromanol` | `datadog.pub` | DataDog (dd-trace-dotnet, system-tests...) |
| `ddoghq.github.com` | `daniel-romanolorente_ddog` (managed) | `ddoghq.pub` | ddoghq |

The URL GitHub's **Code** button shows (`git@github.com:ddoghq/...`) uses the wrong account, so rewrite
the host: `git clone git@ddoghq.github.com:ddoghq/dd-go.git`. A browser 404 on a private repo usually
means the browser session is on the other account.

**`gh` has both accounts too, and the active one may be wrong.** `gh auth status` lists
`daniel-romanolorente_ddog` and `dromanol`. With the managed account active, personal repos
(`dromanol/ai`) fail with `GraphQL: Unauthorized: As an Enterprise Managed User, you cannot access this
content`. Pick the account per command instead of switching globally:
`GH_TOKEN=$(gh auth token --user dromanol) gh pr create ...` (or `--user daniel-romanolorente_ddog` for
ddoghq).

**dd-go moved to ddoghq.** `DataDog/dd-go` still exists, but it is frozen read-only ("🔴 FROZEN"). The
live repo is `ddoghq/dd-go` (default branch `prod`), and PRs go there. On Windows a full checkout fails
on paths containing `*`. Use a sparse cone checkout and `git config core.protectNTFS false` in that
clone (only the sparse paths get written).

On Windows, test with the same ssh git uses (`git config core.sshCommand`, e.g.
`C:/Windows/System32/OpenSSH/ssh.exe` talking to the 1Password agent). Git Bash's own `ssh` can't load
the 1Password `.pub`-only IdentityFile ("invalid format") and gives a false "Permission denied".

Related: [[github-token-ddtool]] for `gh` / API auth (it authenticates as `dromanol`, so it can't see
ddoghq repos).
