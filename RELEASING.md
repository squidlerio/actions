# Releasing

This repository hosts several actions in subdirectories (`run-remote`,
`run-local`), all released together under one set of tags.

## Tagging scheme

Consumers reference an action by path and tag, e.g. `squidlerio/actions/run-local@v4`.
Two kinds of tags exist:

- **Pinned patch** — `vX.Y.Z` (e.g. `v4.0.0`). Immutable. Points at exactly one commit.
- **Moving major** — `vX` (e.g. `v4`). Re-pointed to the latest `vX.Y.Z` on every release
  within that major. This is what most consumers pin to, so they pick up
  non-breaking fixes automatically.

A new **major** is for breaking changes (renamed/removed inputs, a moved action
path, changed default behaviour). Everything else is a minor or patch.

## Before you tag: regenerate the vendored CLI (`run-local` only)

`run-local` ships a **self-contained bundle** of `@squidlerio/cli` at
`run-local/cli/dist/index.js`. The CLI source is **not in this repo** — it lives in
the [`squidlerio/alles`](https://github.com/squidlerio/alles) monorepo under
`tool/squidler-cli`. The bundle is committed here as a build artifact.

**The bundle does not update itself.** If the CLI changed since the last release,
regenerate it from a checkout of `alles` (esbuild is in the devenv environment),
then copy the artifact and version into this repo:

```bash
# From the alles monorepo:
cd tool/squidler-cli
esbuild src/index.ts \
  --bundle --platform=node --format=esm --target=node20 \
  --outfile=/path/to/actions/run-local/cli/dist/index.js \
  --external:bufferutil --external:utf-8-validate --external:@aws-sdk/client-s3 \
  --banner:js="import { createRequire as __cr } from 'module'; const require = __cr(import.meta.url);"
```

Then sync the version so `--version` and the CDP-proxy compatibility check report
correctly:

```bash
# Copy the `version` from tool/squidler-cli/package.json into
# actions/run-local/cli/package.json
```

Sanity-check the regenerated bundle before committing:

```bash
node run-local/cli/dist/index.js --version    # should print the new version
node run-local/cli/dist/index.js run --help    # should list the run flags
```

See [`run-local/cli/README.md`](run-local/cli/README.md) for the rationale behind
each esbuild flag.

## Cutting a release

1. Land all changes on `main` (including a regenerated `run-local/cli/dist/index.js`
   if the CLI moved — see above).
2. Create the immutable patch tag and push it:

   ```bash
   git tag -a v4.0.0 -m "v4.0.0"
   git push origin v4.0.0
   ```

3. Move the major tag to the same commit and force-push it:

   ```bash
   git tag -f v4 v4.0.0
   git push -f origin v4
   ```

4. (First release of a major only) Create a GitHub Release from the `vX.0.0` tag with
   notes covering breaking changes and the migration path.

## Compatibility notes

- The bare repository path (`squidlerio/actions@vX`) is **not** a usable action —
  the root `action.yml` is a guard that errors with a pointer to the sub-actions.
  Up to and including `v3`, the quality-checks action was published at the root; it
  now lives at `run-remote`. Mention this in the release notes for any major that
  moves an action's path.
- `run-local`'s bundled CLI reports a version that the CDP proxy compares against its
  own expected version. A mismatch only logs a warning (it does not fail the run), but
  keeping the vendored `version` current avoids noisy warnings in consumers' logs.
