# Custom GUI Plan — Lesson‑Limited Toolbox

Ticket: TW-2

## Goals
- Provide a lesson-specific, restricted toolbox (only selected built-in blocks) for a highly guided learning experience.
- Toggleable at runtime; persists across app restarts; easily reset to defaults.
- Keep Desktop/Electron fork thin; concentrate logic in scratch‑gui.
- Avoid patching VM/runtime; do not rely on hideFromPalette for core blocks.

## Architecture (GUI vs Desktop)
- scratch-gui: Owns the editor UI and toolbox generation. This is where we inject a gate to substitute the default toolbox with a JSON-defined toolbox when enabled.
- Desktop (Electron wrapper): Loads the GUI in a browser window and provides app menus/settings. Optional place to surface a GUI toggle, but not required for core functionality.

## Thin Desktop strategy
- Maintain a “thin” Desktop fork that:
  - Pins the GUI submodule to our custom GUI commit(s).
  - Optionally exposes a minimal Desktop Settings control (checkbox + file picker) that forwards a JSON toolbox to the renderer via IPC.
  - Optionally enforces kiosk/lockdown/devtools policy if needed for classrooms.
- Keep all toolbox logic in GUI, so the Desktop fork remains low-maintenance.

## GUI changes (initial scope)
1) Lesson toolbox helper (new)
- Location: `src/lib/lesson-toolbox.(ts|js)`
- Responsibilities:
  - Persistence via `localStorage` keys:
    - `lessonToolbox.enabled` → `"true"|"false"`
    - `lessonToolbox.json` → raw JSON text
  - Validation/parsing of the JSON; return XML (or a structure GUI already accepts) or `null` on failure.
  - Basic schema checking: ensure `kind: "categoryToolbox"` and `contents: []` are present.

2) Gate toolbox builder
- Location: `src/lib/make-toolbox-xml.(ts|js)` (or equivalent in TurboWarp’s fork)
- Change: Before returning the default toolbox, check `lessonToolbox.enabled`; if enabled and JSON parses, return the lesson toolbox instead; else fall back to default.

3) Menu entries for toggle/load/reset
- Location: `src/components/menu-bar/menu-bar.(tsx|jsx)` (File menu)
- Add submenu "Lesson Toolbox":
  - "Enable Lesson Toolbox" (checkbox)
  - "Load Lesson Toolbox JSON…" (file picker → read file text → save → enable)
  - "Reset Lesson Toolbox" (disable + clear JSON)
- Wire actions to trigger a toolbox rebuild (existing GUI action used when language/theme changes or when rebuilding toolbox after settings changes).

4) Rebuild trigger
- Location: GUI container that can rebuild the toolbox (e.g., `src/containers/gui.(tsx|jsx)`)
- Provide a handler like `onRequestToolboxRebuild()` to force the palette to refresh using the current builder.

## Toggle UX
- Entry point: File → Lesson Toolbox → [✓ Enable], [Load JSON…], [Reset].
- Flows:
  - Load JSON… → choose file → enable automatically → rebuild toolbox.
  - Enable/disable → rebuild toolbox; keep JSON content unless Reset is chosen.
  - Reset → disable + clear JSON; rebuild to defaults.
- Error handling:
  - On invalid JSON, show a non-blocking toast/modal: “Invalid toolbox JSON; using default toolbox.” Do not enable until a valid JSON is saved.

## JSON schema/sample
- Expected shape is the standard Scratch/Blockly category toolbox in JSON form.
- Minimal example (only Go to x:y, Move, Wait, Repeat):

```json
{
  "kind": "categoryToolbox",
  "contents": [
    {
      "kind": "category",
      "name": "Motion",
      "categorystyle": "motion_category",
      "contents": [
        { "kind": "block", "type": "motion_gotoxy" },
        { "kind": "block", "type": "motion_movesteps" }
      ]
    },
    {
      "kind": "category",
      "name": "Control",
      "categorystyle": "control_category",
      "contents": [
        {
          "kind": "block",
          "type": "control_wait",
          "inputs": {
            "DURATION": {
              "shadow": { "type": "math_positive_number", "fields": { "NUM": 1 } }
            }
          }
        },
        {
          "kind": "block",
          "type": "control_repeat",
          "inputs": {
            "TIMES": {
              "shadow": { "type": "math_whole_number", "fields": { "NUM": 10 } }
            }
          }
        }
      ]
    }
  ]
}
```

Notes:
- `type` is the block opcode (e.g., `motion_movesteps`).
- `categorystyle` should match existing styles to keep them themed.
- Shadows are optional; defaults can be omitted and the editor will supply standard shadows.

## Rebuild flow
- Source of truth: `make-toolbox-xml()`.
- When menu state changes or JSON updates, dispatch the GUI’s existing toolbox-refresh action. Typical flows that already rebuild include language changes; reuse the same mechanism.
- On rebuild, the gate selects the lesson toolbox if enabled and valid; otherwise default.

## Persistence
- Primary: `localStorage` in the renderer (per profile). Keys: `lessonToolbox.enabled`, `lessonToolbox.json`.
- Optional Desktop integration:
  - Desktop Settings can send JSON to the renderer via IPC; GUI saves it to `localStorage`.
  - Desktop can provide a policy to lock the lesson toolbox (kiosk mode), preventing edits.

## Risks
- Upstream changes to toolbox generation may move or rename `make-toolbox-xml`.
- Invalid JSON causing empty toolbox; mitigated by validation + safe fallback.
- Extensions/addons that assume full categories exist might behave unexpectedly.
- Divergence from upstream if GUI files are refactored; keep patches small and well-isolated.

## Next steps
1) Implement `src/lib/lesson-toolbox.(ts|js)` with persistence + validator.
2) Gate `src/lib/make-toolbox-xml.(ts|js)` to consult the lesson toolbox.
3) Add File → Lesson Toolbox menu items; wire to rebuild.
4) Smoke-test with sample JSON and ensure Reset restores default.
5) Create a thin Desktop fork to pin GUI commit; optionally add a Desktop Settings mirror of the toggle using IPC.
6) Document how to package, how to provide per-lesson JSON files, and how to lock settings for classroom mode.
