# Squidler GitHub Actions

A collection of GitHub Actions for running [Squidler](https://qa.squidler.io) website quality and accessibility checks from your CI/CD pipelines.

This repository hosts several actions, each in its own subdirectory and referenced with a path suffix:

| Action | Reference | What it does |
| ------ | --------- | ------------ |
| **Quality Checks** | [`squidlerio/actions/run-remote@v4`](run-remote) | Triggers Squidler checks against a site Squidler's cloud workers can reach on their own (production, staging, a public preview) via the Squidler GitHub integration, and reports results as GitHub check runs. |
| **Local Run** | [`squidlerio/actions/run-local@v4`](run-local) | Runs Squidler standalone test cases against a URL that is only reachable from the runner — a `localhost` dev server brought up earlier in the job, a private preview deploy — and fails the workflow on a failing test. |

## Which one do I want?

- **The site under test is publicly reachable** (production, staging, a public preview URL) and you want checks to show up as native GitHub check runs driven by Squidler's GitHub integration → [`run-remote`](run-remote).
- **The site under test only exists on the runner** (a dev server you `pnpm dev`'d in the same job, a private preview) or you want to run a curated suite of standalone `.md` test cases and gate the PR on the result → [`run-local`](run-local).

## Quick start

### `run-remote`

```yaml
- uses: squidlerio/actions/run-remote@v4
  with:
    api-key: ${{ secrets.SQUIDLER_API_KEY }}
    repository-id: ${{ github.repository_id }}
    sha: ${{ github.sha }}
    ref: ${{ github.ref }}
```

### `run-local`

```yaml
- uses: squidlerio/actions/run-local@v4
  with:
    test-path: squidler/
    base-url: http://localhost:3000
    api-key: ${{ secrets.SQUIDLER_API_KEY }}
```

See each action's README ([`run-remote`](run-remote), [`run-local`](run-local)) for the full input/output reference and worked examples.

## Versioning

Actions are released together under a single set of tags. Pin to a major tag (`@v4`) to follow non-breaking updates, or to an exact tag (`@v4.0.0`) for full reproducibility. Both subactions move with the repository version.

## Support

- 📖 **Documentation**: <https://qa.squidler.io/docs>
- 💬 **Support**: support@squidler.io
- 🐛 **Issues**: [GitHub Issues](https://github.com/squidlerio/actions/issues)

## License

MIT License — see [LICENSE](LICENSE).
