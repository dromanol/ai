---
name: github-ssh-saml-sso
description: `git push` fails with "The 'DataDog' organization has enabled or enforced SAML SSO" → the SSH key's SSO authorization lapsed; re-authorize it in GitHub settings.
type: gotcha
---

Symptom: `git push` to a DataDog-org repo fails with

```
ERROR: The 'DataDog' organization has enabled or enforced SAML SSO.
To access this repository, you must use ... SSH with an SSH key ... that has been authorized for this organization.
```

Misleading signals — auth itself is fine:
- `ssh -T git@github.com` succeeds ("Hi <user>!").
- `git ls-remote` / fetch on **public** repos (e.g. dd-trace-dotnet) still work; only writes are rejected.

Cause: the SSH key is no longer SSO-authorized for the DataDog org (authorization/session expired).

Fix (user action, browser):
1. https://github.com/settings/keys
2. Find the key the remote uses (match fingerprint: `ssh-keygen -lf <IdentityFile>` from `~/.ssh/config`).
3. **Configure SSO** → **Authorize** for DataDog. If it already shows authorized, deauthorize and re-authorize.

Diagnose quickly with `git push --dry-run`, which reproduces the error without pushing.

On Windows, test with the same ssh git uses (`git config core.sshCommand`, e.g.
`C:/Windows/System32/OpenSSH/ssh.exe` talking to the 1Password agent); Git Bash's own `ssh` can't load
the 1Password `.pub`-only IdentityFile ("invalid format") and gives a false "Permission denied".

Related: [[github-token-ddtool]] for `gh` / API auth.
