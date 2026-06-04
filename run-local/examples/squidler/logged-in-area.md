---
squidlerFormat: 1
name: Logged-in area renders after sign-in
description: |
  Needs a real account, which a fresh CI environment doesn't have. The
  requires-credentials label lets a workflow skip it with
  `exclude-labels: requires-credentials` (see the PR example) while a
  nightly run against staging — where a user exists — still exercises it.
labels:
  - requires-credentials
---

- The user can sign in
  - Navigate to /login
  - [LOGIN]

- The logged-in area is displayed
  - Verify the page shows content that is only available to signed-in users
