# Claude Code Instructions: FluidNC Probe Utility v1.15.1

## Small patch on top of v1.15.0

**Source file:** `FluidNC_Probe_Utility_v1.15.0.html`
**Output file:** `FluidNC_Probe_Utility_v1.15.1.html`

Make a copy of v1.15.0 named v1.15.1, then apply all edits below.

---

## OVERVIEW

Two changes only:

1. **Zero XY undo** — snapshot G54 WCO before zeroing, show last value, offer one-level undo
2. **Go Z 0 button** — add next to existing Go XY 0 button
3. Version bump to 1.15.1

---

## EDIT 1 — CSS: Add undo row styles

Find:

```
.mcs-row { display: flex; align-items: center; gap: 8px; flex-wrap: wrap; }
```

Insert **before** that line:

```
/* ── Zero XY undo ── */
.zero-undo-row { display: flex; align-items: center; gap: 10px; margin-top: 6px; font-size: 12px; }
.zero-undo-row .undo-info { color: var(--text-muted); font-family: monospace; }
.zero-undo-row .undo-info span { color: var(--warning); }
```

---

## EDIT 2 — Position section HTML: Add Go Z 0 and undo row

Find:

```
      <div style="margin-top:10px;display:flex;gap:8px;flex-wrap:wrap;align-items:center;">
        <button onclick="zeroAxis('XY')" class="btn-primary" title="Set XY job origin to current position (G10 L20 P1 X0 Y0)">Set Job Origin XY Here</button>
        <button onclick="gotoZero('XY')" title="Go to XY job origin (G0 X0 Y0)">Go to Job Origin XY</button>
        <button onclick="refreshPosition()" style="margin-left:auto;" title="Request position update (?)">↻ Refresh</button>
      </div>
```

Replace with:

```
      <div style="margin-top:10px;display:flex;gap:8px;flex-wrap:wrap;align-items:center;">
        <button onclick="zeroAxisXYWithUndo()" class="btn-primary" title="Set XY job origin to current position (G10 L20 P1 X0 Y0)">Set Job Origin XY Here</button>
        <button onclick="gotoZero('XY')" title="Go to XY job origin (G0 X0 Y0)">Go to Job Origin XY</button>
        <button onclick="gotoZero('Z')"  title="Go to Z job origin (G0 Z0)">Go Z 0</button>
        <button onclick="refreshPosition()" style="margin-left:auto;" title="Request position update (?)">↻ Refresh</button>
      </div>
      <div class="zero-undo-row" id="zero-undo-row" style="display:none;">
        <span class="undo-info">Last job origin before Zero XY: <span id="zero-undo-label"></span></span>
        <button onclick="undoZeroXY()" title="Restore previous G54 XY offset">↩ Undo Zero XY</button>
      </div>
```

---

## EDIT 3 — JS: Add zeroAxisXYWithUndo and undoZeroXY functions

Find:

```
//#region ===== MCS MOVE =====
```

Insert **before** that line:

```
//#region ===== ZERO XY UNDO =====
let _zeroXYUndo = null; // { x, y } — previous G54 WCO

function zeroAxisXYWithUndo() {
  if (!App.isConnected) { alert('Not connected'); return; }
  // Snapshot current G54 offset before overwriting it
  _zeroXYUndo = { x: App.wco.x, y: App.wco.y };
  const fx = (_zeroXYUndo.x >= 0 ? '+' : '') + _zeroXYUndo.x.toFixed(3);
  const fy = (_zeroXYUndo.y >= 0 ? '+' : '') + _zeroXYUndo.y.toFixed(3);
  // Show undo row
  document.getElementById('zero-undo-label').textContent = `X=${fx}  Y=${fy}`;
  document.getElementById('zero-undo-row').style.display = '';
  // Perform the zero
  zeroAxis('XY');
}

function undoZeroXY() {
  if (!App.isConnected) { alert('Not connected'); return; }
  if (!_zeroXYUndo) return;
  const fx = (_zeroXYUndo.x >= 0 ? '+' : '') + _zeroXYUndo.x.toFixed(3);
  const fy = (_zeroXYUndo.y >= 0 ? '+' : '') + _zeroXYUndo.y.toFixed(3);
  if (!confirm(
    `Restore previous G54 job origin?\n\n` +
    `Sends: G10 L2 P1 X${_zeroXYUndo.x.toFixed(3)} Y${_zeroXYUndo.y.toFixed(3)}\n\n` +
    `This will undo the last "Set Job Origin XY Here" and restore the previous offset.`
  )) return;
  sendCommand(`G10 L2 P1 X${_zeroXYUndo.x.toFixed(3)} Y${_zeroXYUndo.y.toFixed(3)}`);
  _zeroXYUndo = null;
  document.getElementById('zero-undo-row').style.display = 'none';
  // Refresh position display after a moment
  setTimeout(() => sendCommand('?'), 200);
}
//#endregion
```

---

## EDIT 4 — Version string and title

Find:

```
const VERSION = '1.15.0';
```

Replace with:

```
const VERSION = '1.15.1';
```

Find:

```
<title>FluidNC Probe Utility v1.15.0</title>
```

Replace with:

```
<title>FluidNC Probe Utility v1.15.1</title>
```

---

## NOTES

**Why `App.wco` is the right thing to snapshot:**
`App.wco` holds the current G54 offset (the difference between machine coords and job coords). When `G10 L20 P1 X0 Y0` is sent, the controller recalculates the offset based on current machine position. To undo, `G10 L2 P1 X{old} Y{old}` restores the explicit offset value directly — no motion involved.

**One level of undo only.** Clicking "Set Job Origin XY Here" a second time overwrites `_zeroXYUndo` with the new snapshot (the state just before the second zero). The undo label updates to reflect this. This is intentional — deep undo history would be confusing on a machine control panel.

**Undo row visibility:** Hidden until the first Zero XY is performed. Disappears after undo is used. Reappears (with updated snapshot) if Zero XY is clicked again.

**Go Z 0** calls the existing `gotoZero('Z')` function — no new logic needed, just adding the button. Verify that `gotoZero('Z')` sends `G0 Z0` in the existing code; if it sends `G53 G0 Z0` instead, that is also fine (machine coords).
