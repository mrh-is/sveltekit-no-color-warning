# SvelteKit `NO_COLOR` warning reproduction

An unmodified `sv create --template minimal` project. It shows that `vite build` prints a Node warning when the shell sets `NO_COLOR`.

## Steps

```sh
pnpm install
NO_COLOR=1 pnpm build
```

The build succeeds, but Node prints this twice (once during route analysis, once after the client build):

```text
(node:89716) Warning: The 'NO_COLOR' env is ignored due to the 'FORCE_COLOR' env being set.
```

Running `pnpm build` without `NO_COLOR` prints no warning.

## Cause

`forked()` in `packages/kit/src/utils/fork.js` starts each worker with `{ ...process.env, SVELTEKIT_FORK: 'true', FORCE_COLOR: '1' }`. That always sets `FORCE_COLOR`, so a worker inherits both variables whenever the parent has `NO_COLOR`, and Node warns about the conflict.
