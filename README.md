# Dependency Build Caching Optimization — Demo

## What's here
- `package.json` / `package-lock.json` — a deliberately dependency-heavy Node
  project (express, react, webpack, typescript, eslint, jest, etc. — 465
  resolved packages) so the install step is slow enough to benchmark.
- `.github/workflows/build-cache-demo.yml` — workflow that:
  - Keys `actions/cache` on `hashFiles('**/package-lock.json')`
  - Times the `npm ci` step explicitly (start/end epoch ms)
  - Prints `steps.npm-cache.outputs.cache-hit` so the hit/miss is visible in
    the logs without digging
  - Writes the timing + hit/miss to the run's Job Summary too

## How to run the demo (needs to happen on GitHub itself — Actions can't run
## from this sandbox since it has no push access to your repo)

1. Create a new (or use an existing) GitHub repo and push this folder's
   contents to the `main` branch.
2. Go to the repo's **Actions** tab → select **Dependency Build Caching
   Demo** → **Run workflow** (this is the `workflow_dispatch` trigger, so
   you don't need a new commit each time).
3. **Run #1 (cold cache):**
   - Open the run → `Restore npm dependency cache` step will show no
     matching key found → `Report cache status` prints `Cache hit: false` /
     `CACHE MISS`.
   - Note the `npm ci took N ms` line in `Print install duration`.
4. **Run #2 (same lockfile, no changes):**
   - Trigger `Run workflow` again immediately.
   - `Restore npm dependency cache` step now shows `Cache restored from
     key: ...` → `Report cache status` prints `Cache hit: true` / `CACHE
     HIT`.
   - Compare the `npm ci took N ms` line — this should be noticeably
     lower since npm is installing from the restored `~/.npm` cache
     instead of hitting the registry for every package.
5. For the demo deliverable, screenshot (or copy the text of):
   - The `Report cache status` step output for both runs (miss vs hit)
   - The `Print install duration` line for both runs
   - Optionally the Job Summary panel, which has both in one place

## Why the key is built this way
`hashFiles('**/package-lock.json')` means the cache key changes if and only
if a dependency actually changes. That's the point of the exercise: as long
as you don't touch `package.json`/`package-lock.json` between run #1 and
run #2, you get an exact key match and a full restore. If you want to show
the *invalidation* case too, bump a dependency version, regenerate the
lockfile (`npm install --package-lock-only`), commit, and run again — you'll
see a fresh `CACHE MISS` even though older cache entries exist (the
`restore-keys` prefix fallback will partially help but still trigger new
installs for the changed packages).
