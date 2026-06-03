# Vendored Squidler CLI

`dist/index.js` is a **generated, self-contained bundle** of `@squidlerio/cli`. The
[`run-local`](../) action runs it directly with `node` — there is no `npm install`
or build step at action runtime, so the action works in any consuming repository
without assuming the Squidler monorepo is checked out.

**Do not edit `dist/index.js` by hand.** It is a build artifact.

## Provenance

- **Source:** `tool/squidler-cli` in the `squidlerio/alles` monorepo.
- **Bundler:** [esbuild](https://esbuild.github.io/) (single-file ESM bundle, all runtime
  dependencies inlined).

`package.json` carries the CLI `version` (read at runtime by the CLI to report
`--version`) and `"type": "module"` (the bundle is ESM).

## Regenerating

From a checkout of the `alles` monorepo, with esbuild available (it is in the
devenv environment):

```bash
cd tool/squidler-cli
esbuild src/index.ts \
  --bundle --platform=node --format=esm --target=node20 \
  --outfile=/path/to/actions/run-local/cli/dist/index.js \
  --external:bufferutil --external:utf-8-validate --external:@aws-sdk/client-s3 \
  --banner:js="import { createRequire as __cr } from 'module'; const require = __cr(import.meta.url);"
```

Then copy the CLI's `version` from `tool/squidler-cli/package.json` into this
directory's `package.json`.

Notes on the flags:

- `--format=esm` + the `createRequire` banner: the CLI is ESM and a couple of code
  paths use `require()` for Node built-ins; the banner shims `require` under ESM.
- `--external:bufferutil --external:utf-8-validate`: optional native speedups for
  `ws`, loaded behind `try/catch` — excluded so the bundle stays pure-JS and
  portable.
- `--external:@aws-sdk/client-s3`: an optional dependency of `unzipper`'s S3 path,
  which the CLI never exercises (it only unzips local Chrome downloads).
