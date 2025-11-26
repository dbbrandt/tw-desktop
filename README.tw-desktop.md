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

---

## Develop with a sibling tw-gui (linked scratch-gui)
This repo supports live development against a local tw-gui checkout using npm link.

1) In your tw-gui checkout (sibling directory)
```bash
# from ../tw-gui
npm ci
npm link  # exposes scratch-gui globally for linking
```

2) In this turbowarp desktop repo
```bash
# ensure deps
npm ci

# link to the local scratch-gui from tw-gui
npm run gui:link  # runs: npm link scratch-gui

# start desktop with the linked GUI
npm run dev:gui
```

Notes about the webpack setup here:
- scratch-blocks media is resolved dynamically (findScratchBlocksMedia) so media copies even when scratch-gui/scratch-blocks are linked.
- React, React-DOM, React-Redux, and Redux are aliased to this repo’s node_modules and resolve.symlinks=false avoids duplicate React trees when using links.

### Unlink / restore registry packages
If you want to go back to registry scratch-gui:
```bash
# in this repo
npm unlink scratch-gui && npm install

# optionally in ../tw-gui, remove global link
npm unlink -g scratch-gui || true
```

### Troubleshooting
- Runtime error "Could not find store in the context of Connect(...)": ensure a single copy of React/React-Redux is bundled. Run:
  ```bash
  npm ls react react-dom react-redux redux | sed -n '1,120p'
  ```
  Aliases in webpack.config.cjs plus symlinks=false should prevent duplicates.
- Build error "unable to locate ... node_modules/scratch-blocks/media glob": the config now resolves the real path; re-run `npm run webpack:compile` or `npm run dev:gui` after linking.

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

## Build macOS DMG for distribution (with local tw-gui)

1) Ensure `../tw-gui` is up to date and linked
```bash
# from ../tw-gui
npm ci
npm link  # exposes scratch-gui globally for linking

# back in this repo
npm run gui:link  # runs: npm link scratch-gui
```

2) Build production assets
```bash
npm run fetch
npm run webpack:prod
```

3) Build a macOS DMG (universal, no publish)
```bash
npx electron-builder --mac dmg --universal --publish=never
```

Artifacts:
- DMG: `dist/TurboWarp-Setup-<version>.dmg` (send to users)
- App bundle: under `dist/mac` (or similar) for local install/testing

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
