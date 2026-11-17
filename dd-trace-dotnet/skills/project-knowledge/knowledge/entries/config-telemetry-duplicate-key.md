---
name: config-telemetry-duplicate-key
description: Reading a key another settings class already records, through a tracked `config`, duplicates its telemetry entry and breaks AsmTelemetryTests; read it untracked despite CONV-1.1
type: gotcha
---

CONV-1.1 (`docs/development/for-ai/Coding-Requirements/Codebase-Conventions.md`) flags every
`ConfigurationBuilder` built with `NullConfigurationTelemetry`. There is one case where the untracked builder is
the right choice: a key that **another settings class already reads and records**.

Example: `TracerSettings` needed `DD_APPSEC_ENABLED` (owned and recorded by `SecuritySettings`) to compute a
default. Reading it through `TracerSettings`'s tracked `config` recorded the key a second time.
`AsmTelemetryTests.TestSecurityInitializationRC` (`tracer/test/Datadog.Trace.Security.IntegrationTests`) asserts
exactly **one** `DD_APPSEC_ENABLED` entry, so it failed with "Expected configValues0 to contain 1 item(s), but found 2".

Fix: read it with an untracked builder and add a one-line comment saying who records the key.
`TracerSettings` keeps a shared `untrackedConfig` (also used for `IsRunningInCiVisibility`). If a reviewer cites
CONV-1.1, point to the duplicate record.

**Why it's easy to miss:**
- The duplicate entry has the same value and origin, so it looks harmless in review.
- Unit tests don't catch it. Only the Security integration tests do.
- If the read sits behind a `&&` short-circuit (for example a platform check), the failure only shows on the
  platforms where the read actually runs. For `TracerSettings` that was the Linux ASM jobs only.

Before reading another product's key from a settings class, grep for who else reads it (`git grep "ConfigurationKeys.<Area>.<Key>"`).
