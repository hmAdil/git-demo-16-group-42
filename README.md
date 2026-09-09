# dep-cache-demo

A minimal Node.js project used to benchmark GitHub Actions dependency
caching (`actions/cache`) against a cold install.

## Stack

- Node.js 20
- express, react, react-dom, axios, lodash, moment, webpack, typescript,
  eslint, jest — chosen purely to give `npm ci` a non-trivial dependency
  tree (~465 resolved packages) to fetch

## Project structure

```
.
├── .github/workflows/build-cache-demo.yml   # CI pipeline with dependency caching
├── package.json
├── package-lock.json
└── README.md
```

## CI pipeline

`.github/workflows/build-cache-demo.yml` runs on push to `main` and can
also be triggered manually from the Actions tab (`workflow_dispatch`).

It:

1. Checks out the repo and sets up Node.js 20
2. Restores/saves `~/.npm` via `actions/cache`, keyed on
   `hashFiles('**/package-lock.json')` — the key only changes when a
   dependency changes
3. Reports whether the cache was hit (`steps.npm-cache.outputs.cache-hit`)
4. Times `npm ci` explicitly and logs the duration
5. Runs `npm run build` and `npm test`

Timing and cache-hit status are printed to the step logs and written to the
run's Job Summary.

## Running locally

```bash
npm ci
npm run build
npm test
```

## Benchmarking the cache

Trigger the workflow twice in a row without changing `package.json` /
`package-lock.json`:

- **First run** — no cache entry exists yet → `cache-hit: false` → `npm ci`
  fetches every package from the registry.
- **Second run** — matching cache key found → `cache-hit: true` → `npm ci`
  installs from the restored `~/.npm` cache, noticeably faster.

To see the cache invalidate on purpose, bump a dependency version,
regenerate the lockfile (`npm install --package-lock-only`), commit, and
run again — you'll get a fresh miss even though older cache entries still
exist.
