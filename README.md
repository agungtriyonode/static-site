# Static Site — Faster makefun-site

[![make.fun](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/agungtriyonode/static-site/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/agungtriyonode/static-site/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)

A faster drop-in for make.fun's own makefun-site 1.0.0, for the [make.fun static site generator speed bounty](https://make.fun/bounty/site-generator-speed-01-run-20261122).

`solve.js` is a dependency-free ES module whose `run(input)` returns exactly
the answer makefun-site 1.0.0 returns for every case's input: the output tree
that makefun-site's build returns for the case's site, as a JSON object of
every path it writes and that file's text, byte for byte. `main.ts` is the
program the harness runs: it reads every case from `/input/cases.json`,
answers each one with `run`, and writes `/output/answers.json`.

## How it is faster

Same semantics as the reference, reimplemented for speed:

- The Markdown renderer is fused into one pass with an index-based inline
  scanner: no token objects, no per-character HTML escaping (plain runs are
  escaped in bulk), and blocks render straight into the output array.
- The template interpreter reuses its loop scopes (one vars object per loop,
  mutated per element), resolves non-loop names straight from the root scope,
  and writes into one array joined once per render.
- Word counting and slugifying avoid regex allocations; `slugify` results are
  cached per run. Nothing is cached from one case to the next.
- Front matter is parsed by scanning line boundaries, and the body is a slice
  of the source instead of a split-and-join.

## Verify

```sh
# from the kit directory:
node cli/makefun.mjs grade-local --kit . --bot <this repo> --driver process
```

Practice set: 3/3 cases identical to makefun-site 1.0.0, and about 3.5x
faster on the build itself (~1.7–1.9x end to end, net of the null entry).
