---
name: system-tests-local-run-gotchas
description: Running system-tests locally from Windows/WSL — force missing_feature tests with -F, WSL setup traps, runner pip timeout, and the LF rewrite the Linux package build leaves in the working tree.
type: gotcha
---

Lessons from running system-tests against a local build (the `run-system-tests` skill) on Windows.

**Run a test the manifest skips.** Tests marked `missing_feature` for dotnet in
`system-tests/manifests/dotnet.yml` are xfail'ed; force them with `-F <nodeid>` (repeatable; or
`SYSTEM_TESTS_FORCE_EXECUTE=a,b`):

```bash
TEST_LIBRARY=dotnet ./run.sh APPSEC_CORRUPTED_RULES tests/appsec/test_customconf.py::Test_CorruptedRules_Telemetry \
  -F tests/appsec/test_customconf.py::Test_CorruptedRules_Telemetry
```

A forced test that works reports `1 passed`, not `xpassed`. The scenario name comes from the test
class decorator (`@scenarios.appsec_corrupted_rules` → `APPSEC_CORRUPTED_RULES`).

**WSL traps.**
- Calling `wsl wslpath -a ...` from PowerShell returns UTF-16 with NULs, so `cd "$st"` fails with
  `cd: null directory`. Use the literal `/mnt/c/...` path instead.
- Ubuntu 26.04 ships no `python3.12`: install from `ppa:deadsnakes/ppa` together with
  `python3.12-venv python3.12-dev build-essential` (needs interactive sudo, so the user runs it).
- `docker` inside WSL needs Docker Desktop → Resources → WSL Integration for the distro. Applying it
  restarts Docker and kills any in-flight build (`rpc error ... EOF`): just rerun.

**Runner venv timeout.** `./build.sh dotnet` caps each step at 600 s; on `/mnt/c` the runner's
`pip install -e .` can exceed it after the weblog image is already built. Rerun only the runner:
`./build.sh dotnet -i runner` (wheels are cached by then).

**The Linux package build rewrites the repo to LF.** It bind-mounts the clone into Linux containers,
leaving ~500 tracked files modified with line-ending-only changes. Before committing, stage only real
content changes plus new files, then discard the rest:

```bash
git add $(git diff --ignore-cr-at-eol --name-only) <new files>
git commit ...
git checkout -- .
```

Related: [[linux-package-targets]], [[glibc-tar-bundles-musl]], [[build-native-wrapper]]
