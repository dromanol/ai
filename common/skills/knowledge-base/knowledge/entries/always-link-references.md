---
name: always-link-references
description: Never mention a PR or a ticket without a link — make each mention itself a link, every time, in every kind of text
type: working-style
---

Never write a reference to a PR, a ticket or a tracked issue (`#65`, `APPSEC-123`, an Error Tracking
issue ID, a Workflow ID) without a link. The reference text itself is the link:
`[#65](https://github.com/<org>/<repo>/pull/65)`, `[APPSEC-123](https://datadoghq.atlassian.net/browse/APPSEC-123)`.

**Why:** the user said "Nunca añadas una referencia a una PR o un ticket sin un enlace". A Jira status
comment had to be edited after it was posted. Each PR line had its URL at the end, but "Merge after
#65" and the merge-order list had bare `#NN`.

**How to apply:**
- Link **each** mention, also repeats in the same text ("Merge after #65", "With #67", ordered lists).
  A URL elsewhere on the line does not count.
- Applies everywhere: Jira/Confluence/Slack text, PR descriptions, commit messages that leave GitHub,
  in-session replies and tables, memory and knowledge files.
- GitHub autolinks `#NN` only inside the same repository. Jira, Slack and Confluence never do. Write
  the full URL there.
- Before you post or show text, scan it for `#\d+`, `[A-Z]+-\d+` and bare issue IDs.
