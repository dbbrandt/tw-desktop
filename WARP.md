# TurboWarp Remedial Learning Environment — WARP Project Guide

Ticket: TW-1

## Project goals
- Build a remedial development learning environment using TurboWarp/Scratch editor.
- Limit available blocks and features per lesson; provide slow, repetitive lessons tailored for accessibility.
- Target: a highly guided experience for a handicapped learner.

## Constraints and assumptions
- Base: https://github.com/TurboWarp/desktop (local clone present).
- Node.js 20 required.
- macOS development environment.

## Environment notes
- Recommended: nvm (or fnm/asdf) to pin Node 20.
- Git submodules required (repo was cloned with `--recursive`).

## Quick start / resume checklist
1. Node 20 active
   - nvm: `nvm use 20` (or `nvm install 20 && nvm use 20`)
   - Verify: `node -v` → v20.x
2. Ensure submodules are initialized/updated
   - `git submodule update --init --recursive`
   - Check: `git submodule status`
3. Install dependencies (based on lockfile)
   - If `package-lock.json`: `npm ci` (preferred) or `npm install`
   - If `yarn.lock`: `yarn install`
   - If `pnpm-lock.yaml`: `pnpm install`
4. Run the app in development
   - Likely: `npm run start` (Electron dev) or `npm run dev`
   - If start fails, inspect `package.json` scripts for the correct entry.
5. Build distributables (optional)
   - Likely: `npm run build`

## Commands reference (detect and run)
- Detect package manager:
  - `ls -1 | egrep "^(package-lock.json|yarn.lock|pnpm-lock.yaml)$"`
- Submodules:
  - Update: `git submodule update --init --recursive`
  - Status: `git submodule status`
- Lint/tests (to be confirmed from repo):
  - `npm run lint` / `npm test` if available.

## Progress / work log
- [ ] Ensure Node 20 active
- [ ] Submodules fully initialized
- [ ] Dependencies installed
- [ ] App runs locally (electron dev)
- [ ] Build completes successfully
- [ ] Create minimal lesson scaffold (placeholder)
- [x] Add WARP.md to document goals & workflow (this file)

## Next steps
- Confirm package manager and install dependencies.
- Launch desktop app and verify editor loads.
- Document the exact start/build commands from `package.json`.
- Begin “lesson limiter” spike: identify where to hide/disable blocks and UI in TurboWarp desktop.

## Decision log
- Use Node.js v20.
- Base on TurboWarp Desktop repo as upstream.

## Useful links
- TurboWarp Desktop: https://github.com/TurboWarp/desktop
- TurboWarp (general): https://turbowarp.org/
