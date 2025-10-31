# tw-desktop (TurboWarp Desktop fork)

Minimal local notes for dbbrandt/tw-desktop.

## Prerequisites
- Node.js 22.x active (zstd required by TurboWarp extensions)
- Git submodules initialized
- GitHub CLI (optional) authenticated as dbbrandt

## Quick start
```bash
# ensure Node 22
nvm use 22  # or your manager (fnm/volta/brew)

# one-time (or after pulling changes)
git submodule update --init --recursive
npm ci

# dev loop (watch + electron)
npm run dev
```
The `dev` script runs:
- `npm run fetch` (downloads libraries, updates packager, prepares extensions)
- `webpack --watch` and `electron src-main/entrypoint.js` concurrently

## One-off manual run
```bash
npm run fetch
npm run webpack:compile
npm run electron:start
```

## Build distributable (dir layout)
```bash
npm run electron:package:dir
```

## Sync fork with upstream
Remotes (expected):
```bash
git remote -v
# origin   https://github.com/dbbrandt/tw-desktop (fetch/push)
# upstream https://github.com/TurboWarp/desktop (fetch/push)
```
Option A (gh):
```bash
gh repo sync --source upstream --branch master
```
Option B (git):
```bash
git fetch upstream
git checkout master
# fast-forward only recommended; switch to merge if needed
git merge --ff-only upstream/master
git push origin master
```

## Notes
- Node 22 is required; Node 20 will fail on `npm run fetch` with `zlib.zstdCompressSync` missing.
- If native modules are rebuilt or Node version changes, reinstall deps:
```bash
rm -rf node_modules && npm ci
```
- If Electron doesn’t start after a big change, re-run `npm run fetch` and `npm run webpack:compile`.
