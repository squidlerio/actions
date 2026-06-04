# Squidler Local Run (`run-local`)

Run Squidler test cases against any URL reachable from the workflow runner — your dev server, a preview deploy, or staging — and fail the workflow if a test fails.

Test cases are markdown files in your repository. Squidler executes them against `base-url` and reports back via GitHub Actions annotations, JUnit XML, TAP, or plain stdout.

> Targeting a publicly-reachable site (production, staging, a public preview) and want results as native GitHub check runs via the Squidler GitHub integration instead? Use the [`run-remote`](../run-remote) action.

## Usage

```yaml
- uses: squidlerio/actions/run-local@v4
  with:
    test-path: squidler/
    base-url: https://staging.example.com
    api-key: ${{ secrets.SQUIDLER_API_KEY }}
```

The minimal version above runs every `*.md` test case under `squidler/` against the supplied URL. Defaults: `concurrency: 4`, `format: github` (so failures appear as inline PR annotations).

See the [examples/](examples) directory for complete, copy-pasteable workflows: [test the PR's code on a runner-local dev server](examples/pr-test-dev-server.yml), [a nightly integration test against staging](examples/scheduled-integration-test.yml), and [testing a preview deploy](examples/preview-deploy.yml). [examples/squidler/](examples/squidler) holds three minimal test cases to seed your own `squidler/` directory from: a [homepage smoke test](examples/squidler/homepage-loads.md), a [navigation click-through](examples/squidler/navigation-works.md), and a [login-gated test](examples/squidler/logged-in-area.md) demonstrating the `requires-credentials` label.

## Inputs

| Name              | Required | Default                   | Description                                                                                                                                                  |
| ----------------- | -------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `test-path`       | yes      |                           | A `.md` file or a directory of them. Directories are walked recursively for `*.md`.                                                                          |
| `base-url`        | yes      |                           | The URL the run targets. Anything the runner can reach: a localhost dev server brought up earlier in the job, a preview deploy URL, staging, or production. |
| `api-key`         | yes      |                           | Squidler organisation API key (`sqo…`). Use a GitHub secret — `secrets.SQUIDLER_API_KEY`. Get one at <https://qa.squidler.io/integrations/api-keys>.            |
| `concurrency`     | no       | `4`                       | Max parallel runs across the batch. Standalone runs don't queue server-side, so the CLI owns parallelism. Drop to `1` for serial.                            |
| `format`          | no       | `github`                  | Report format: `github` (workflow annotations), `junit` (JUnit XML, pair with `output:`), `tap` (Test Anything Protocol), or `human` (plain stdout).         |
| `output`          | no       | _stdout_                  | Write the formatted report to this path instead of stdout. Typical pairing: `format: junit` + `output: junit.xml`.                                            |
| `exclude-labels`  | no       |                           | Comma-separated frontmatter labels to skip. Useful when some tests need state your CI environment can't provide (e.g. `requires-credentials`).                |
| `no-local-chrome` | no       | `false`                   | Skip the local Chrome the action launches by default. Only valid when `base-url` is publicly reachable from Squidler's cloud workers.                        |
| `poll-timeout-ms` | no       | `600000`                  | Per-test poll timeout in milliseconds. Override for unusually long-running tests.                                                                            |
| `api-url`         | no       | `https://api.squidler.io` | Squidler API base URL. Override only for self-hosted or staging Squidler instances.                                                                          |
| `node-version`    | no       | `24`                      | Node.js version used to run the bundled CLI. The bundle targets Node 20+.                                                                                    |

## Outputs

| Name        | Description                                                                                                                          |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `exit-code` | `0` if every test passed, `1` if any test failed or errored, `2` if any test exceeded its poll timeout, `3` if Chrome/daemon setup failed. |

## Scenarios

### Test your dev server in PR CI

Spin up your dev server in the same job and point the action at it. Squidler's cloud worker drives a local Chrome on the runner over the CDP proxy, so `http://localhost:*` is reachable without tunnels or port-forwarding.

```yaml
name: PR smoke
on: pull_request

jobs:
  squidler:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6

      - name: Bring up the dev server
        run: |
          pnpm install
          pnpm dev &
          npx wait-on http://localhost:3000

      - uses: squidlerio/actions/run-local@v4
        with:
          test-path: squidler/
          base-url: http://localhost:3000
          api-key: ${{ secrets.SQUIDLER_API_KEY }}
```

### Run against a preview deploy

If you already publish a preview URL on each PR (Vercel, Netlify, Render), skip the local Chrome:

```yaml
- uses: squidlerio/actions/run-local@v4
  with:
    test-path: squidler/critical-paths/
    base-url: ${{ steps.preview.outputs.url }}
    api-key: ${{ secrets.SQUIDLER_API_KEY }}
    no-local-chrome: 'true'
```

### Produce a JUnit report

Pair `format: junit` with `output:` so a downstream test-reporter step can pick the file up:

```yaml
- uses: squidlerio/actions/run-local@v4
  with:
    test-path: squidler/
    base-url: https://staging.example.com
    api-key: ${{ secrets.SQUIDLER_API_KEY }}
    format: junit
    output: junit.xml

- uses: actions/upload-artifact@v7
  if: always()
  with:
    name: squidler-junit
    path: junit.xml
```

### Skip tests that need credentials

Some tests assert post-login content and can't run against an empty environment. Tag their frontmatter and skip them via `exclude-labels`:

```yaml
---
squidlerFormat: 1
name: Dashboard renders after login
labels:
  - requires-credentials
---
```

```yaml
- uses: squidlerio/actions/run-local@v4
  with:
    test-path: squidler/
    base-url: http://localhost:3000
    api-key: ${{ secrets.SQUIDLER_API_KEY }}
    exclude-labels: requires-credentials
```

## Requirements

- A Squidler organisation API key. Get one at <https://qa.squidler.io/integrations/api-keys> — store it as a GitHub secret.
- Test cases as `.md` files in your repository. The MCP server's `test_case_create_standalone` tool generates these via a guided conversation; you can also write them by hand — start from the minimal ones in [examples/squidler/](examples/squidler). See the [test-case authoring guide](https://qa.squidler.io/docs/llm/standalone-runs) for the markdown format.
- A reachable `base-url`. For local dev servers this is whatever port your `pnpm dev` / `npm start` / etc. binds — the action handles the cloud-to-localhost bridging via a local Chrome.

## How it works

The action sets up Node and runs a self-contained Squidler CLI bundle that ships with it (under [`cli/`](cli)) — there's no install or build step at action time. The CLI launches a local Chrome on the runner. When Squidler's cloud worker executes your test case, its browser commands are tunnelled through that local Chrome via Squidler's CDP proxy — that's what makes `http://localhost:*` URLs work from a cloud worker. Each test gets a fresh Chrome state (no leaked cookies/localStorage between runs).

`base-url` is the test's host. Test case markdown stays path-relative (`Navigate to /signup`) so the same `.md` file can target localhost in CI and a staging URL in nightly smoke without edits.

Step Optimization and Fixed Actions kick in automatically on the second-and-subsequent run of an unchanged test case — Squidler matches by content hash within your organisation, so re-runs are faster than first runs without your workflow tracking anything.

## License

MIT.
