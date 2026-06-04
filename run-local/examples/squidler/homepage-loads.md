---
squidlerFormat: 1
name: Homepage loads
description: |
  The smallest useful test case: open the front page and check that real
  content rendered. Paths are relative — `base-url` comes from the
  workflow — so the same file tests localhost in PR CI and staging in the
  nightly without edits.
---

- The homepage loads with its main content visible
  - Navigate to /
  - Verify the page's main heading is visible
  - Verify the page shows no error message
