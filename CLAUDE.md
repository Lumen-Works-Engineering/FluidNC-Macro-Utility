# CLAUDE.md — FluidNC Probe Utility

This file tells Claude Code how to work on this project effectively.

---

## Project Overview

**FluidNC Probe Utility** is a portable, single-file HTML/CSS/JS web application for CNC machine control and probe operations. It communicates with a FluidNC controller via WebSocket. There is no build system, no npm, no framework — just one HTML file that opens in a browser.

**GitHub:** https://github.com/Lumen-Works-Engineering/FluidNC-Macro-Utility
**Current version:** 1.15.1
**Author:** John Sparks / Lumen Works Engineering

---

## File Naming Convention

**Every time you modify the application, save a new versioned HTML file.**

Format: `FluidNC_Probe_Utility_v{MAJOR}.{MINOR}.{PATCH}.html`

When creating a new version:
1. Copy the current version to the new filename (use PowerShell `Copy-Item` — bash cp doesn't work on this Windows system)
2. Apply all edits to the new file
3. Bump the `VERSION` JS constant and the `<title>` tag

```powershell
# Copy syntax that works on this Windows system:
powershell -Command "Copy-Item 'C:\...\FluidNC_Probe_Utility_v1.15.1.html' 'C:\...\FluidNC_Probe_Utility_v1.15.2.html'"
```

**Never edit the previous version file once a new version exists.**

---

## Architecture

### Single-file HTML application
Everything — HTML, CSS, JS, and user settings — lives in one `.html` file. This is intentional: the file is portable, can be opened from a USB drive, and settings are saved via browser Save-As.

### Settings system
User settings are stored in an embedded JSON block inside `<script type="application/json" id="user-settings">`. When settings change in the UI, the JS updates this block in memory. When the user does browser File → Save As (Ctrl+S), the updated settings are saved inside the HTML file itself.

**Settings JSON structure:**
```json
{
  "connection": { "ip": "...", "port": 81 },
  "probe": { "tipDiameter": 3.0, "feedrate": 100, "feedrateSlow": 50, "retractDistance": 2.0, "timeout": 30 },
  "jog": { "xySpeed": 1800, "zSpeed": 400 },
  "outputs": { "mistOn": "M7", "mistOff": "M9", "vacuumOn": "...", "vacuumOff": "...", "iotOn": "...", "iotOff": "..." },
  "defaults": {
    "probeDistances": { "cylinder": 20, "hole": 20, "corner": 20, "side": 5 },
    "zSafeHeights": { "standard": 10, "side": 10, "heightmap": 20 }
  },
  "laser": { "lowPowerPct": 7, "highPowerMaxPct": 50 },
  "toolOffset": { "x": 0, "y": 0 },
  "presets": { "cylinder": [], "hole": [], "corner": [], "heightmap": [] }
}
```

### Global App state
All runtime state lives in the `App` object. Key fields:
- `App.ws` — WebSocket connection
- `App.isConnected` — connection status
- `App.settings` — loaded settings object
- `App.position.mcs` / `App.position.wcs` — current {x,y,z}
- `App.wco` — G54 work coordinate offset
- `App.wcsOffsets[6]` — G54–G59 offsets (index 0=G54)
- `App.activeTool` — 0=spindle, 1=laser
- `App.laserLowActive` — whether low-power laser is firing
- `App.posLog[]` — session position log entries
- `App.probeInProgress` — guards against concurrent probe operations
- `App.offsetMeasurements[]` — tool offset measurement accumulator

### WebSocket communication
- Connects to `ws://<ip>:81/`
- Commands sent with `\n` terminator via `sendCommand(cmd)`
- Responses parsed in `App.ws.onmessage` handler
- Status reports: `<Idle|MPos:x,y,z|...>` → `parseStatusReport()`
- WCS offsets: `[G54:x,y,z]`…`[G59:x,y,z]` → `parseWCSOffsetLine()`
- Probe results: `[PRB:x,y,z,triggered]`
- Alarm messages: `ALARM:n` → `setAlarmState()`

### Probe sequence pattern
All probe operations use async/await with helpers from `createProbeHelpers()`:
- `sendAndWait(cmd)` — sends command, resolves when `ok` received
- `probeAndCapture(axis, dist, feed)` — sends `G38.2`, captures `[PRB:...]` response
- `moveRel(axis, dist)` — relative move in G91

**Listener-before-send pattern** prevents race conditions: event listener is attached before `sendCommand()` is called, so no response can be missed.

---

## Code Organization

The JS uses VS Code-compatible `#region` / `#endregion` markers (use Ctrl+Shift+[ to fold):

```
GLOBAL STATE → SETTINGS MANAGEMENT → WEBSOCKET CONNECTION → MESSAGE HANDLING →
CONTROL TAB → EMERGENCY STOP → ZERO XY UNDO → MCS MOVE → POSITION LOG →
WCS / JOB ORIGINS MANAGER → RETURN POINTS (G28/G30) → TOOL OFFSET →
TOOL & LASER → ALARM CLEAR → SHARED PROBE UTILITIES →
CYLINDER PROBE → HOLE PROBE → CORNER PROBE → SIDE PROBE →
HEIGHT MAP → SETTINGS TAB → INITIALIZATION
```

---

## How to Apply Changes

Instructions are provided as find/replace edit specifications. The pattern is always:

1. Read the target section of the file to verify the find string matches exactly
2. Use the Edit tool with the exact `old_string` and `new_string`
3. Verify the edit landed with a follow-up Grep

When the file is too large to read in one shot (>25,000 tokens), read targeted sections using `offset` and `limit` parameters, or use Grep to locate anchor lines first.

**The file has ~3,500+ lines at v1.15.1.** Always use Grep to locate anchors before editing.

---

## G-code Reference (for this project)

| Command | Meaning |
|---------|---------|
| `$H` | Home all axes |
| `$HX/HY/HZ` | Home individual axis |
| `$#` | Report all WCS offsets (G54–G59, G28, G30, etc.) |
| `$X` | Unlock / clear alarm |
| `?` | Request status report |
| `!` | Feed hold |
| `\x18` (Ctrl+X) | Soft reset |
| `\x85` | Jog cancel |
| `G53` | Machine coordinate system (prefix for absolute moves) |
| `G90/G91` | Absolute / relative mode |
| `G0` | Rapid move |
| `G38.2` | Probe toward, stop on contact |
| `G10 L20 P1` | Set WCS offset (current position = 0) |
| `G10 L2 P1` | Set WCS offset (explicit value) |
| `G28.1` / `G28` | Save / go to Return Point 1 |
| `G30.1` / `G30` | Save / go to Return Point 2 |
| `M3 S{n}` | Spindle/laser on at power S |
| `M5` | Spindle/laser off |
| `M6 T{n}` | Tool change (T0=spindle, T1=laser) |

**Laser fire command:** `G90 M3 S{power} G1 F1` — must be sent as a single command in FluidNC laser mode (`$32=1`). `M5` to stop.

---

## Important Constraints

- **No build step** — never introduce npm, bundlers, or external dependencies
- **Single file** — all code stays in the one HTML file; no separate JS or CSS files
- **Settings must round-trip** — `collectSettingsFromUI()` and `applySettingsToUI()` must stay in sync when new settings keys are added
- **Listener-before-send** — in probe sequences, always attach event listeners before calling `sendCommand()`; never reverse this order
- **G53 for absolute moves** — probe results are in machine coordinates; always use `G53` prefix for absolute moves
- **probeInProgress flag** — set `App.probeInProgress = true` at the start of any probe sequence and `false` in the finally block

---

## Version and Documentation Updates

When bumping version:
1. Update `const VERSION = '...'` in the JS global state region
2. Update `<title>FluidNC Probe Utility v...</title>`
3. Add entry to `CHANGELOG.md` under the new version heading
4. Update version badge in `README.md`
5. Update "Current version" line in `README.md`

When adding new features that users interact with:
- Update the relevant section in `README.md`
- Update `USER_INSTRUCTIONS.md` with step-by-step guidance
- Add to `CHANGELOG.md` under Added/Changed/Fixed

---

## Testing Guidance

This project controls physical CNC machinery — there is no automated test suite. Verification is done by opening the HTML file in a browser and connecting to the controller.

**Browser-only checks (no controller needed):**
- Open the HTML file and verify the page loads without console errors
- Verify title bar shows the correct version
- Check all tabs render without layout breaks
- Verify Settings tab shows new settings fields
- Verify JSON editor shows new keys (`laser`, `toolOffset`)
- Verify Control tab shows all new sections

**With controller connected:**
- Home the machine
- Test position display updates
- Test WCS Refresh (`$#`) populates the G54–G59 table
- Test Return Points save/recall
- Test laser section is disabled until Laser (T1) selected
- Test probe operations on all tabs

---

## Common Pitfalls

- **PowerShell for file ops** — bash `cp` doesn't work on this Windows system; use `powershell -Command "Copy-Item ..."`
- **VERSION mismatch** — the `VERSION` constant was `'1.13.5'` in v1.14.0; it's historical debt. Always set it to match the file version.
- **Read before Edit** — the Edit tool requires the file to have been read first; always Grep/Read to find anchor text before editing
- **Large file reads** — the HTML file is ~3,500+ lines (~50,000 tokens). Read in chunks with `offset`/`limit` or use Grep to find specific sections.
- **Settings sync** — if you add a new settings key, update all three of: embedded JSON, `collectSettingsFromUI()`, and `applySettingsToUI()`
