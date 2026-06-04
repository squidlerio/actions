---
squidlerFormat: 1
name: Main navigation works
description: |
  Click through one navigation link and confirm the destination renders.
  The [CONTINUE] marker makes the second goal continue in the same
  browser session instead of starting from a fresh page load.
---

- The site's navigation is visible on the homepage
  - Navigate to /
  - Verify a navigation menu is visible

- Following a navigation link shows the destination page [CONTINUE]
  - Click the first link in the navigation menu
  - Verify the destination page loads with content matching the link's label
