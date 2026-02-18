# Claude Code Instructions: FluidNC Probe Utility v1.15.0

## SUPERSEDES previous v1.15.0 instruction document — use this one only

**Source file:** `FluidNC_Probe_Utility_v1.14.0.html`
**Output file:** `FluidNC_Probe_Utility_v1.15.0.html`

Make a copy of v1.14.0 named v1.15.0, then apply all edits below in order.

---

## OVERVIEW

All changes are to the Control tab and its supporting JS/CSS/Settings.
No other tabs are modified. Changes:

1. CSS — new styles for all new components
2. Header — tool indicator badge
3. Control tab HTML — 9 sections (some new, some replacing existing):
   - Move to Machine Position (MCS Move)
   - Position Log
   - WCS / Job Origins Manager
   - Return Points (G28/G30)
   - Tool Offset (new — measure, accumulate, average, store)
   - Active Tool selector
   - Laser Control (with corrected fire command)
   - Clear Alarm
   - Coordinate System Reference (educational, bottom of tab)
4. Settings tab — Laser and Tool Offset settings
5. Embedded JSON — add `laser` and `toolOffset` keys
6. Global App state — new fields
7. Message handler — parse G54–G59 from `$#`
8. Replace `parseG54Offset` with `parseWCSOffsetLine`
9. Emergency stop — add M5
10. New JS regions — all new functions
11. Settings collect/apply — laser and toolOffset
12. Init — wire up new components
13. Version string and title

---

## EDIT 1 — CSS

Find:

```
.jog-label { text-align: center; font-size: 10px; color: var(--text-muted); margin-top: 3px; }
.home-buttons { display: flex; flex-wrap: wrap; gap: 6px; }
```

Replace with:

```
.jog-label { text-align: center; font-size: 10px; color: var(--text-muted); margin-top: 3px; }
.home-buttons { display: flex; flex-wrap: wrap; gap: 6px; }

/* ── Tool indicator badge ── */
.tool-badge {
  font-size: 11px; font-weight: bold; padding: 3px 10px;
  border-radius: 3px; letter-spacing: 0.05em;
}
.tool-badge-spindle { background: #1a3a1a; color: var(--success); border: 1px solid var(--success); }
.tool-badge-laser   { background: #3a1500; color: var(--warning); border: 1px solid var(--warning); }

/* ── Tool selector buttons ── */
.tool-select-row { display: flex; align-items: center; gap: 10px; flex-wrap: wrap; }
.btn-tool-spindle-active { background: #1a3a1a !important; border-color: var(--success) !important; color: var(--success) !important; font-weight: bold; }
.btn-tool-laser-active   { background: #3a1500 !important; border-color: var(--warning) !important; color: var(--warning) !important; font-weight: bold; }

/* ── Laser control ── */
.laser-section-disabled { opacity: 0.4; pointer-events: none; }
.laser-firing-low  { color: var(--warning); font-weight: bold; font-size: 12px; }
.laser-firing-high { color: var(--danger);  font-weight: bold; font-size: 12px; }
#laser-high-btn { user-select: none; -webkit-user-select: none; }
#laser-high-btn.firing { background: var(--danger) !important; border-color: var(--danger) !important; color: #fff !important; }
.laser-warn { color: var(--warning); font-size: 11px; font-style: italic; }
.pwm-hint { color: var(--text-muted); font-size: 11px; margin-left: 4px; }

/* ── WCS / Job Origins table ── */
.wcs-table { width: 100%; border-collapse: collapse; font-size: 12px; }
.wcs-table th, .wcs-table td { padding: 5px 8px; border-bottom: 1px solid var(--border); text-align: right; }
.wcs-table th { text-align: left; color: var(--text-secondary); font-weight: normal; font-size: 11px; background: var(--bg-tertiary); }
.wcs-table .wcs-slot  { text-align: left; font-weight: bold; font-family: monospace; min-width: 60px; }
.wcs-table .wcs-note  { text-align: left; color: var(--text-muted); font-size: 11px; }
.wcs-table .wcs-acts  { text-align: center; white-space: nowrap; }
.wcs-table .wcs-acts button { padding: 2px 7px; font-size: 11px; margin: 0 1px; }
.wcs-edit-panel { background: var(--bg-tertiary); border: 1px solid var(--accent); border-radius: 4px; padding: 10px; margin-top: 8px; display: none; }
.wcs-edit-panel.visible { display: block; }

/* ── Position log table ── */
.poslog-table { width: 100%; border-collapse: collapse; font-size: 12px; }
.poslog-table th, .poslog-table td { padding: 4px 8px; border-bottom: 1px solid var(--border); text-align: right; }
.poslog-table th { text-align: left; color: var(--text-secondary); font-weight: normal; font-size: 11px; background: var(--bg-tertiary); }
.poslog-table .pl-check { text-align: center; width: 28px; }
.poslog-table .pl-label { text-align: left; }
.poslog-table .pl-label input { background: transparent; border: none; color: var(--text-primary); font-size: 12px; width: 100px; }
.poslog-table .pl-label input:focus { background: var(--bg-input); border: 1px solid var(--accent); border-radius: 2px; }
.poslog-table .pl-acts { text-align: center; white-space: nowrap; }
.poslog-table .pl-acts button { padding: 2px 6px; font-size: 11px; margin: 0 1px; }
.diff-out { font-family: monospace; font-size: 12px; background: var(--bg-primary); padding: 6px 10px;
  border: 1px solid var(--border); border-radius: 3px; color: var(--accent); white-space: pre; margin-top: 8px; }

/* ── MCS Move inputs ── */
.mcs-row { display: flex; align-items: center; gap: 8px; flex-wrap: wrap; }
.mcs-row label { display: flex; align-items: center; gap: 4px; font-size: 12px; cursor: pointer; }
.mcs-row input[type="number"] { width: 88px; }

/* ── Return Points ── */
.return-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
.return-group { border: 1px solid var(--border); border-radius: 4px; padding: 8px; }
.return-group-title { font-size: 11px; color: var(--text-secondary); margin-bottom: 6px; }
.return-group .btn-row { display: flex; gap: 6px; }

/* ── Tool Offset ── */
.offset-stored { font-family: monospace; font-size: 13px; color: var(--accent); padding: 4px 0; }
.offset-meas-table { width: 100%; border-collapse: collapse; font-size: 12px; margin-top: 6px; }
.offset-meas-table th, .offset-meas-table td { padding: 4px 8px; border-bottom: 1px solid var(--border); text-align: right; }
.offset-meas-table th { text-align: left; color: var(--text-secondary); font-weight: normal; background: var(--bg-tertiary); }
.offset-meas-table .om-acts button { padding: 2px 6px; font-size: 11px; }
.offset-step { background: var(--bg-tertiary); border-left: 3px solid var(--accent); padding: 8px 10px; margin: 6px 0; border-radius: 0 4px 4px 0; font-size: 12px; }
.offset-step-done { border-left-color: var(--success); }
.offset-step-title { font-weight: bold; margin-bottom: 3px; }
.offset-calc-result { font-family: monospace; color: var(--warning); font-size: 13px; padding: 4px 0; }

/* ── Coordinate Reference ── */
.coord-ref { background: var(--bg-primary); border: 1px solid var(--border); border-radius: 4px; padding: 12px 14px; font-size: 12px; line-height: 1.7; }
.coord-ref h4 { font-size: 12px; color: var(--accent); margin: 10px 0 4px 0; }
.coord-ref h4:first-child { margin-top: 0; }
.coord-ref .gcmd { font-family: monospace; color: var(--warning); font-size: 11px; }
.coord-ref .rule { background: var(--bg-secondary); border-left: 3px solid var(--success); padding: 6px 10px; margin-top: 8px; border-radius: 0 3px 3px 0; font-size: 12px; }
```

---

## EDIT 2 — Header: Add tool badge

Find:

```
  <div style="flex: 1;"></div>
  <button id="stop-btn" class="btn-danger">⚠ STOP</button>
```

Replace with:

```
  <div style="flex: 1;"></div>
  <span id="header-tool-badge" class="tool-badge tool-badge-spindle" title="Active tool — use Tool Selector in Control tab to change">SPINDLE</span>
  <button id="stop-btn" class="btn-danger">⚠ STOP</button>
```

---

## EDIT 3 — Control tab HTML: Replace contents of `panel-control`

Find this entire block:

```
<!-- CONTROL TAB -->
<div class="tab-panel active" id="panel-control">
  <div class="control-grid">
    <div class="section">
      <div class="section-title">Position</div>
      <table class="position-table">
        <thead><tr><th></th><th>MCS</th><th>WCS</th><th>Actions</th></tr></thead>
        <tbody>
          <tr>
            <td class="axis-label">X</td>
            <td class="coord-value" id="pos-mcs-x">0.000</td>
            <td class="coord-value" id="pos-wcs-x">0.000</td>
            <td class="actions">
              <button onclick="zeroAxis('X')">Zero</button>
              <button onclick="gotoZero('X')">Go 0</button>
            </td>
          </tr>
          <tr>
            <td class="axis-label">Y</td>
            <td class="coord-value" id="pos-mcs-y">0.000</td>
            <td class="coord-value" id="pos-wcs-y">0.000</td>
            <td class="actions">
              <button onclick="zeroAxis('Y')">Zero</button>
              <button onclick="gotoZero('Y')">Go 0</button>
            </td>
          </tr>
          <tr>
            <td class="axis-label">Z</td>
            <td class="coord-value" id="pos-mcs-z">0.000</td>
            <td class="coord-value" id="pos-wcs-z">0.000</td>
            <td class="actions">
              <button onclick="zeroAxis('Z')">Zero</button>
              <button onclick="gotoZero('Z')">Go 0</button>
            </td>
          </tr>
        </tbody>
      </table>
      <div style="margin-top: 12px; display: flex; gap: 8px; flex-wrap: wrap;">
        <button onclick="zeroAxis('XY')" class="btn-primary">Zero XY</button>
        <button onclick="gotoZero('XY')">Go XY 0</button>
        <button onclick="refreshPosition()" style="margin-left: auto;">↻ Refresh</button>
      </div>
    </div>
    <div class="section">
      <div class="section-title">Home</div>
      <div class="home-buttons">
        <button onclick="homeAxis('all')" class="btn-primary">Home All</button>
        <button onclick="homeAxis('X')">Home X</button>
        <button onclick="homeAxis('Y')">Home Y</button>
        <button onclick="homeAxis('Z')">Home Z</button>
      </div>
    </div>
    <div class="section">
      <div class="section-title">Jog</div>
      <div class="jog-container">
        <div>
          <div class="jog-xy">
            <div class="jog-placeholder"></div>
            <button class="jog-btn" data-axis="Y" data-dir="+">▲</button>
            <div class="jog-placeholder"></div>
            <button class="jog-btn" data-axis="X" data-dir="-">◀</button>
            <div class="jog-placeholder"></div>
            <button class="jog-btn" data-axis="X" data-dir="+">▶</button>
            <div class="jog-placeholder"></div>
            <button class="jog-btn" data-axis="Y" data-dir="-">▼</button>
            <div class="jog-placeholder"></div>
          </div>
          <div class="jog-label">XY</div>
        </div>
        <div>
          <div class="jog-z">
            <button class="jog-btn" data-axis="Z" data-dir="+">▲</button>
            <button class="jog-btn" data-axis="Z" data-dir="-">▼</button>
          </div>
          <div class="jog-label">Z</div>
        </div>
      </div>
      <p class="text-muted mt-1" style="text-align: center; font-size: 11px;">Click and hold to jog</p>
    </div>
    <div class="section">
      <div class="section-title">Outputs</div>
      <div class="output-row">
        <span class="output-name">Mist</span>
        <button id="mist-btn" onclick="toggleOutput('mist')">OFF</button>
      </div>
      <div class="output-row">
        <span class="output-name">Vacuum</span>
        <button id="vacuum-btn" onclick="toggleOutput('vacuum')">OFF</button>
      </div>
      <div class="output-row">
        <span class="output-name">IoT PDU</span>
        <button id="iot-btn" onclick="toggleOutput('iot')">OFF</button>
      </div>
    </div>
  </div>
</div>
```

Replace with:

```
<!-- CONTROL TAB -->
<div class="tab-panel active" id="panel-control">
  <div class="control-grid">

    <!-- ── Position ── -->
    <div class="section">
      <div class="section-title">
        Position
        <span class="text-muted" style="font-weight:normal;font-size:11px;margin-left:6px;">Machine Pos (absolute) / Job Origin offset</span>
      </div>
      <table class="position-table">
        <thead><tr><th></th><th>Machine Pos</th><th>Job Pos (G54)</th><th>Actions</th></tr></thead>
        <tbody>
          <tr>
            <td class="axis-label">X</td>
            <td class="coord-value" id="pos-mcs-x">0.000</td>
            <td class="coord-value" id="pos-wcs-x">0.000</td>
            <td class="actions">
              <button onclick="zeroAxis('X')" title="Set X job origin here (G10 L20 P1 X0)">Zero X</button>
              <button onclick="gotoZero('X')" title="Go to X job origin (G0 X0)">Go 0</button>
            </td>
          </tr>
          <tr>
            <td class="axis-label">Y</td>
            <td class="coord-value" id="pos-mcs-y">0.000</td>
            <td class="coord-value" id="pos-wcs-y">0.000</td>
            <td class="actions">
              <button onclick="zeroAxis('Y')" title="Set Y job origin here (G10 L20 P1 Y0)">Zero Y</button>
              <button onclick="gotoZero('Y')" title="Go to Y job origin (G0 Y0)">Go 0</button>
            </td>
          </tr>
          <tr>
            <td class="axis-label">Z</td>
            <td class="coord-value" id="pos-mcs-z">0.000</td>
            <td class="coord-value" id="pos-wcs-z">0.000</td>
            <td class="actions">
              <button onclick="zeroAxis('Z')" title="Set Z job origin here (G10 L20 P1 Z0)">Zero Z</button>
              <button onclick="gotoZero('Z')" title="Go to Z job origin (G0 Z0)">Go 0</button>
            </td>
          </tr>
        </tbody>
      </table>
      <div style="margin-top:10px;display:flex;gap:8px;flex-wrap:wrap;align-items:center;">
        <button onclick="zeroAxis('XY')" class="btn-primary" title="Set XY job origin to current position (G10 L20 P1 X0 Y0)">Set Job Origin XY Here</button>
        <button onclick="gotoZero('XY')" title="Go to XY job origin (G0 X0 Y0)">Go to Job Origin XY</button>
        <button onclick="refreshPosition()" style="margin-left:auto;" title="Request position update (?)">↻ Refresh</button>
      </div>
    </div>

    <!-- ── Home ── -->
    <div class="section">
      <div class="section-title">Home <span class="text-muted" style="font-weight:normal;font-size:11px;">— always home before starting work</span></div>
      <div class="home-buttons">
        <button onclick="homeAxis('all')" class="btn-primary">Home All ($H)</button>
        <button onclick="homeAxis('X')">Home X</button>
        <button onclick="homeAxis('Y')">Home Y</button>
        <button onclick="homeAxis('Z')">Home Z</button>
      </div>
    </div>

    <!-- ── Jog ── -->
    <div class="section">
      <div class="section-title">Jog</div>
      <div class="jog-container">
        <div>
          <div class="jog-xy">
            <div class="jog-placeholder"></div>
            <button class="jog-btn" data-axis="Y" data-dir="+">▲</button>
            <div class="jog-placeholder"></div>
            <button class="jog-btn" data-axis="X" data-dir="-">◀</button>
            <div class="jog-placeholder"></div>
            <button class="jog-btn" data-axis="X" data-dir="+">▶</button>
            <div class="jog-placeholder"></div>
            <button class="jog-btn" data-axis="Y" data-dir="-">▼</button>
            <div class="jog-placeholder"></div>
          </div>
          <div class="jog-label">XY</div>
        </div>
        <div>
          <div class="jog-z">
            <button class="jog-btn" data-axis="Z" data-dir="+">▲</button>
            <button class="jog-btn" data-axis="Z" data-dir="-">▼</button>
          </div>
          <div class="jog-label">Z</div>
        </div>
      </div>
      <p class="text-muted mt-1" style="text-align:center;font-size:11px;">Click and hold to jog</p>
    </div>

    <!-- ── Outputs ── -->
    <div class="section">
      <div class="section-title">Outputs</div>
      <div class="output-row">
        <span class="output-name">Mist</span>
        <button id="mist-btn" onclick="toggleOutput('mist')">OFF</button>
      </div>
      <div class="output-row">
        <span class="output-name">Vacuum</span>
        <button id="vacuum-btn" onclick="toggleOutput('vacuum')">OFF</button>
      </div>
      <div class="output-row">
        <span class="output-name">IoT PDU</span>
        <button id="iot-btn" onclick="toggleOutput('iot')">OFF</button>
      </div>
    </div>

  </div><!-- end control-grid -->

  <!-- ════════════════════════════════════════════════ -->
  <!-- ── Move to Machine Position ── -->
  <!-- ════════════════════════════════════════════════ -->
  <div class="section" style="margin-top:12px;">
    <div class="section-title">Move to Machine Position
      <span class="text-muted" style="font-weight:normal;font-size:11px;margin-left:6px;">(G53 — absolute machine coordinates, requires homing first)</span>
    </div>
    <div class="mcs-row">
      <label><input type="checkbox" id="mcs-x-en"> X:</label>
      <input type="number" id="mcs-x-val" step="0.001" placeholder="0.000">
      <label><input type="checkbox" id="mcs-y-en"> Y:</label>
      <input type="number" id="mcs-y-val" step="0.001" placeholder="0.000">
      <label><input type="checkbox" id="mcs-z-en"> Z:</label>
      <input type="number" id="mcs-z-val" step="0.001" placeholder="0.000">
      <button onclick="moveToMCS()" class="btn-primary">Go</button>
      <button onclick="fillMCSFromCurrent()" title="Populate fields with current machine position">← From Current</button>
    </div>
    <p class="text-muted mt-1" style="font-size:11px;">Check the axes you want to move. ⚠ Make sure Z is clear before moving XY to a new position.</p>
  </div>

  <!-- ════════════════════════════════════════════════ -->
  <!-- ── Position Log ── -->
  <!-- ════════════════════════════════════════════════ -->
  <div class="section" style="margin-top:12px;">
    <div class="section-title">Position Log
      <span class="text-muted" style="font-weight:normal;font-size:11px;margin-left:8px;">Session only — clears on page refresh. Useful for measuring tool offsets.</span>
    </div>
    <div style="display:flex;gap:8px;margin-bottom:8px;flex-wrap:wrap;align-items:center;">
      <button onclick="recordPosition()" class="btn-primary">● Record Current Position</button>
      <button onclick="diffSelectedPositions()" id="pos-diff-btn">Diff 2 Selected</button>
      <button onclick="clearPositionLog()" style="margin-left:auto;">Clear Log</button>
    </div>
    <div id="pos-log-wrap">
      <p class="text-muted" style="font-size:12px;">No positions recorded yet.</p>
    </div>
    <div id="pos-log-diff" class="diff-out" style="display:none;"></div>
  </div>

  <!-- ════════════════════════════════════════════════ -->
  <!-- ── Job Origins Manager (G54–G59) ── -->
  <!-- ════════════════════════════════════════════════ -->
  <div class="section" style="margin-top:12px;">
    <div class="section-title">Saved Job Origins (G54–G59)
      <button onclick="refreshWCSTable()" style="float:right;padding:2px 8px;font-size:11px;" title="Send $# to read all offsets from controller">↻ Refresh from Controller</button>
    </div>
    <p class="text-muted" style="font-size:11px;margin-bottom:8px;">
      Each slot stores a machine position that becomes XYZ 0,0,0 for jobs using that slot.
      <strong>G54 is the default</strong> used by almost all G-code files.
      Stored in controller EEPROM — survive power off.
    </p>
    <table class="wcs-table">
      <thead>
        <tr>
          <th style="text-align:left;">Slot</th>
          <th style="text-align:left;">Purpose / Note</th>
          <th>Mach X</th><th>Mach Y</th><th>Mach Z</th>
          <th style="text-align:center;">Actions</th>
        </tr>
      </thead>
      <tbody id="wcs-tbody">
        <tr><td colspan="6" class="text-muted" style="text-align:center;padding:10px;">Connect and click ↻ Refresh to load offsets.</td></tr>
      </tbody>
    </table>
    <div class="wcs-edit-panel" id="wcs-edit-panel">
      <div style="font-size:12px;margin-bottom:8px;">
        Set <strong id="wcs-edit-label">G54</strong> job origin to this machine position
        <span class="text-muted" style="font-size:11px;">(the machine coordinates that map to job XYZ 0,0,0)</span>
      </div>
      <div class="mcs-row">
        <label style="min-width:20px;">X:</label>
        <input type="number" id="wcs-edit-x" step="0.001" style="width:90px;">
        <label style="min-width:20px;">Y:</label>
        <input type="number" id="wcs-edit-y" step="0.001" style="width:90px;">
        <label style="min-width:20px;">Z:</label>
        <input type="number" id="wcs-edit-z" step="0.001" style="width:90px;">
        <button onclick="applyWCSEdit()" class="btn-primary">Apply</button>
        <button onclick="cancelWCSEdit()">Cancel</button>
      </div>
      <p class="text-muted" style="font-size:11px;margin-top:6px;">Sends: G10 L2 P{n} X{val} Y{val} Z{val} — writes directly to controller EEPROM.</p>
    </div>
    <div style="display:flex;gap:8px;margin-top:8px;flex-wrap:wrap;align-items:center;">
      <button onclick="clearG54()" title="Reset G54 to zero. Required before using LightBurn in Absolute Coords mode.">
        Reset Job Origin G54 to Zero
      </button>
      <span class="text-muted" style="font-size:11px;">Sends G10 L2 P1 X0 Y0 Z0 — after homing, job position will equal machine position. Use before LightBurn.</span>
    </div>
  </div>

  <!-- ════════════════════════════════════════════════ -->
  <!-- ── Return Points (G28 / G30) ── -->
  <!-- ════════════════════════════════════════════════ -->
  <div class="section" style="margin-top:12px;">
    <div class="section-title">Return Points
      <span class="text-muted" style="font-weight:normal;font-size:11px;margin-left:6px;">Stored in machine coordinates — survive power off, recall with one button after homing</span>
    </div>
    <div class="return-grid">
      <div class="return-group">
        <div class="return-group-title">Return Point 1 (G28) — e.g. PCB job origin</div>
        <div class="btn-row">
          <button onclick="saveReturnPoint(1)" class="btn-primary" title="Save current machine position as Return Point 1 (G28.1)">Save Here</button>
          <button onclick="gotoReturnPoint(1)" title="Go to Return Point 1 (G28)">Go There</button>
        </div>
      </div>
      <div class="return-group">
        <div class="return-group-title">Return Point 2 (G30) — e.g. tool change position</div>
        <div class="btn-row">
          <button onclick="saveReturnPoint(2)" class="btn-primary" title="Save current machine position as Return Point 2 (G30.1)">Save Here</button>
          <button onclick="gotoReturnPoint(2)" title="Go to Return Point 2 (G30)">Go There</button>
        </div>
      </div>
    </div>
    <p class="text-muted mt-1" style="font-size:11px;">
      <strong>Workflow:</strong> Jog to your PCB corner, zero job XY, then click Save Here (Return Point 1).
      Next session: Home → Go There → job coordinates are already correct, no re-zeroing needed.
    </p>
  </div>

  <!-- ════════════════════════════════════════════════ -->
  <!-- ── Tool Offset (Spindle → Laser) ── -->
  <!-- ════════════════════════════════════════════════ -->
  <div class="section" style="margin-top:12px;">
    <div class="section-title">Tool Offset — Spindle to Laser</div>

    <!-- Stored offset display -->
    <div style="display:flex;align-items:center;gap:16px;margin-bottom:10px;flex-wrap:wrap;">
      <div>
        <span class="text-muted" style="font-size:11px;">Stored offset (laser relative to spindle):</span><br>
        <span class="offset-stored" id="offset-display">Not measured yet</span>
      </div>
      <button onclick="moveToLaserZero()" id="laser-zero-btn" class="btn-primary"
        title="Move laser to where spindle was zeroed. Then set job origin here manually.">
        Move Laser to Spindle Zero
      </button>
      <span class="text-muted" style="font-size:11px;">Sends: G0 X{offsetX} Y{offsetY} — then zero job origin manually as usual.</span>
    </div>

    <!-- Measure workflow -->
    <details>
      <summary style="cursor:pointer;font-size:12px;color:var(--accent);margin-bottom:8px;">▶ Measure / Update Offset (expand)</summary>

      <div class="offset-step" id="offset-step1">
        <div class="offset-step-title">Step 1 — Position spindle tip at reference mark on workpiece</div>
        <button onclick="recordSpindlePos()" class="btn-primary">Record Spindle Position</button>
        <span id="offset-spindle-recorded" style="margin-left:10px;font-size:12px;"></span>
      </div>

      <div class="offset-step" id="offset-step2">
        <div class="offset-step-title">Step 2 — Switch to laser, align beam to same mark, then record</div>
        <button onclick="recordLaserPos()">Record Laser Position</button>
        <span id="offset-laser-recorded" style="margin-left:10px;font-size:12px;"></span>
      </div>

      <div id="offset-calc-row" style="display:none;margin:8px 0;">
        <span class="text-muted" style="font-size:12px;">Calculated offset: </span>
        <span class="offset-calc-result" id="offset-calc-display"></span>
        <button onclick="addOffsetMeasurement()" class="btn-primary" style="margin-left:12px;">Add to Measurements</button>
      </div>

      <!-- Measurements list -->
      <div id="offset-meas-wrap" style="margin-top:8px;display:none;">
        <div style="display:flex;align-items:center;gap:12px;margin-bottom:4px;">
          <span style="font-size:12px;">Measurements:</span>
          <span class="offset-stored" id="offset-avg-display" style="font-size:12px;"></span>
          <button onclick="saveAverageOffset()" class="btn-success" style="margin-left:auto;" title="Save the average of all measurements as the stored offset">Save Average as Offset</button>
          <button onclick="clearOffsetMeasurements()">Clear All</button>
        </div>
        <table class="offset-meas-table" id="offset-meas-table">
          <thead><tr><th>#</th><th>ΔX mm</th><th>ΔY mm</th><th></th></tr></thead>
          <tbody id="offset-meas-tbody"></tbody>
        </table>
      </div>
    </details>
  </div>

  <!-- ════════════════════════════════════════════════ -->
  <!-- ── Active Tool ── -->
  <!-- ════════════════════════════════════════════════ -->
  <div class="section" style="margin-top:12px;">
    <div class="section-title">Active Tool</div>
    <div class="tool-select-row">
      <button id="tool-spindle-btn" onclick="selectTool(0)" class="btn-tool-spindle-active">🔩 Spindle (T0)</button>
      <button id="tool-laser-btn"   onclick="selectTool(1)">⚡ Laser (T1)</button>
      <span style="font-size:12px;color:var(--text-secondary);">
        Active: <span id="tool-active-label" style="font-weight:bold;color:var(--success);">Spindle (T0)</span>
      </span>
      <span class="text-muted" style="font-size:11px;margin-left:auto;">Sends M6 T0 or M6 T1</span>
    </div>
    <p class="text-muted mt-1" style="font-size:11px;">
      ⚠ M3 (laser fire) will not work until the laser is selected with M6 T1. Always switch tools here before firing.
    </p>
  </div>

  <!-- ════════════════════════════════════════════════ -->
  <!-- ── Laser Control ── -->
  <!-- ════════════════════════════════════════════════ -->
  <div class="section" id="laser-section" style="margin-top:12px;">
    <div class="section-title">
      Laser Control
      <span id="laser-disabled-notice" class="text-muted" style="font-weight:normal;font-size:11px;margin-left:8px;">⚠ Switch to Laser (T1) above to enable</span>
    </div>
    <div id="laser-inner">
      <!-- Low power -->
      <div class="form-row" style="margin-bottom:10px;align-items:center;">
        <span class="form-label">Low Power (align):</span>
        <input type="number" id="laser-low-pct" min="1" max="10" value="7" step="1" style="width:52px;" title="1–10% max">
        <span class="unit-suffix">%</span>
        <span class="pwm-hint" id="laser-low-pwm-hint">= S18</span>
        <button id="laser-low-btn" onclick="toggleLaserLow()" style="margin-left:8px;">Fire Low Power (toggle)</button>
        <span id="laser-low-ind" class="laser-firing-low" style="display:none;margin-left:8px;">● FIRING</span>
      </div>
      <!-- High power -->
      <div class="form-row" style="margin-bottom:8px;align-items:center;">
        <span class="form-label">High Power (mark):</span>
        <input type="number" id="laser-high-pct" min="1" max="50" value="30" step="1" style="width:52px;" title="Max set in Settings">
        <span class="unit-suffix">%</span>
        <span class="pwm-hint" id="laser-high-pwm-hint">= S77</span>
        <button id="laser-high-btn" style="margin-left:8px;">Hold to Fire ▶</button>
        <span id="laser-high-ind" class="laser-firing-high" style="display:none;margin-left:8px;">⚠ HIGH POWER FIRING</span>
      </div>
      <p class="laser-warn">⚠ Wear laser safety glasses (450nm OD4+). Ensure beam path is clear. High power fires ONLY while button is held down.</p>
      <p class="text-muted mt-1" style="font-size:11px;">Fire command: G90 M3 S{n} G1 F1  |  Stop: M5  |  Power scale 0–255 PWM</p>
    </div>
  </div>

  <!-- ════════════════════════════════════════════════ -->
  <!-- ── Clear Alarm ── -->
  <!-- ════════════════════════════════════════════════ -->
  <div class="section" style="margin-top:12px;">
    <div class="section-title">Alarm</div>
    <div style="display:flex;align-items:center;gap:12px;">
      <button onclick="clearAlarm()" class="btn-danger">Unlock / Clear Alarm ($X)</button>
      <span class="text-muted" style="font-size:12px;">Use after an E-stop, limit switch trip, or soft reset. Verify the machine is safe before unlocking.</span>
    </div>
  </div>

  <!-- ════════════════════════════════════════════════ -->
  <!-- ── Coordinate System Reference ── -->
  <!-- ════════════════════════════════════════════════ -->
  <div class="section" style="margin-top:12px;">
    <div class="section-title">Coordinate System Reference</div>
    <div class="coord-ref">

      <h4>Machine Position <span class="gcmd">(G53 / MCS)</span></h4>
      The absolute truth. 0,0,0 is always at your homing switches. Never changes unless you move a switch or change machine geometry.
      Every command that uses <span class="gcmd">G53</span> moves to these absolute coordinates regardless of any job origin settings.
      The machine position display at the top of this tab always shows this.

      <h4>Job Origin — G54 <span class="gcmd">(default)</span> through G59</h4>
      When you jog to the corner of your workpiece and click "Set Job Origin XY Here", you are telling the controller
      "wherever the machine is right now is XYZ 0,0,0 for this job." That offset is stored in <span class="gcmd">G54</span> and survives power-off.
      Your G-code files use this origin. Six slots are available (G54–G59) for different fixtures or tools.
      <strong>Almost all G-code files expect G54</strong> — leave the others for special purposes like a secondary fixture or laser offset.

      <h4>Return Points <span class="gcmd">(G28 / G30)</span></h4>
      Two parking spots stored in machine coordinates. Save a position with "Save Here" and recall it with "Go There" after any
      homing cycle. Perfect for returning to your PCB work origin across sessions without re-zeroing.
      Stored in controller EEPROM — survive power-off and controller restarts.

      <h4>Tool Offset</h4>
      Because the laser is physically mounted offset from the spindle, the machine must move by the measured offset amount
      to put the laser beam where the spindle was. This utility stores that offset and the "Move Laser to Spindle Zero" button
      applies it automatically.

      <h4>G10 — Setting offsets by command</h4>
      <span class="gcmd">G10 L20 P1 X0 Y0</span> — "Make my current position become job origin 0,0 for G54."
      This is what "Set Job Origin XY Here" sends. <br>
      <span class="gcmd">G10 L2 P1 X-150 Y-80</span> — "Set the G54 origin to machine position -150,-80 explicitly."
      This is what "Set Values" in the Job Origins table sends.

      <div class="rule">
        <strong>The golden rule:</strong> Home the machine first. After homing, machine position is always accurate.
        Everything else — job origins, return points, tool offsets — depends on a successful home cycle.
      </div>
    </div>
  </div>

</div>
```

---

## EDIT 4 — Settings tab: Add Laser and Tool Offset sections

Find:

```
    <div class="section">
      <div class="section-title">Default Z Safe Heights</div>
```

Replace with:

```
    <div class="section">
      <div class="section-title">Laser Settings</div>
      <div class="form-row">
        <span class="form-label">Low Power Default:</span>
        <div class="input-with-unit">
          <input type="number" id="setting-laser-low" value="7" min="1" max="10" step="1">
          <span class="unit-suffix">% (max 10)</span>
        </div>
      </div>
      <div class="form-row">
        <span class="form-label">High Power Max:</span>
        <div class="input-with-unit">
          <input type="number" id="setting-laser-high-max" value="50" min="1" max="100" step="1">
          <span class="unit-suffix">% (safety cap)</span>
        </div>
      </div>
      <p class="text-muted mt-1" style="font-size:11px;">Power scale: 0–255 PWM. Low power capped at 10% in UI regardless of this setting.</p>
    </div>
    <div class="section">
      <div class="section-title">Stored Tool Offset (Spindle → Laser)</div>
      <div class="form-row">
        <span class="form-label">Offset X:</span>
        <div class="input-with-unit">
          <input type="number" id="setting-offset-x" value="0" step="0.001">
          <span class="unit-suffix">mm</span>
        </div>
      </div>
      <div class="form-row">
        <span class="form-label">Offset Y:</span>
        <div class="input-with-unit">
          <input type="number" id="setting-offset-y" value="0" step="0.001">
          <span class="unit-suffix">mm</span>
        </div>
      </div>
      <p class="text-muted mt-1" style="font-size:11px;">
        These values are set automatically by the "Save Average as Offset" button in the Tool Offset section.
        You can also edit them manually here. Positive X = laser is to the right of spindle.
      </p>
    </div>
    <div class="section">
      <div class="section-title">Default Z Safe Heights</div>
```

---

## EDIT 5 — Embedded JSON: Add laser and toolOffset keys

Find:

```
  "presets": { "cylinder": [], "hole": [], "corner": [], "heightmap": [] }
```

Replace with:

```
  "laser": { "lowPowerPct": 7, "highPowerMaxPct": 50 },
  "toolOffset": { "x": 0, "y": 0 },
  "presets": { "cylinder": [], "hole": [], "corner": [], "heightmap": [] }
```

---

## EDIT 6 — Global App state

Find:

```
const App = {
  ws: null,
  isConnected: false,
  settings: null,
  alarmState: 'None',
  position: {
    mcs: { x: 0, y: 0, z: 0 },
    wcs: { x: 0, y: 0, z: 0 }
  },
  wco: { x: 0, y: 0, z: 0 },
  isJogging: false,
  probeInProgress: false  // Shared flag for any probe operation
};
```

Replace with:

```
const App = {
  ws: null,
  isConnected: false,
  settings: null,
  alarmState: 'None',
  position: {
    mcs: { x: 0, y: 0, z: 0 },
    wcs: { x: 0, y: 0, z: 0 }
  },
  wco: { x: 0, y: 0, z: 0 },
  isJogging: false,
  probeInProgress: false,
  // Tool state
  activeTool: 0,            // 0 = spindle, 1 = laser
  laserLowActive: false,
  // WCS offsets [0]=G54 … [5]=G59
  wcsOffsets: [
    { slot: 'G54', pNum: 1, note: 'Default (most G-code files use this)', x: 0, y: 0, z: 0 },
    { slot: 'G55', pNum: 2, note: '', x: 0, y: 0, z: 0 },
    { slot: 'G56', pNum: 3, note: '', x: 0, y: 0, z: 0 },
    { slot: 'G57', pNum: 4, note: '', x: 0, y: 0, z: 0 },
    { slot: 'G58', pNum: 5, note: '', x: 0, y: 0, z: 0 },
    { slot: 'G59', pNum: 6, note: '', x: 0, y: 0, z: 0 }
  ],
  wcsEditIdx: null,
  // Position log (session only)
  posLog: [],
  posLogIdSeq: 0,
  // Tool offset measurement
  offsetSpindlePos: null,   // { x, y } recorded spindle reference
  offsetLaserPos: null,     // { x, y } recorded laser reference
  offsetMeasurements: []    // array of { x, y }
};
```

---

## EDIT 7 — Message handler: parse all WCS offsets

Find:

```
    if (trimmed.startsWith('<') && trimmed.endsWith('>')) {
      parseStatusReport(trimmed);
    } else if (trimmed.startsWith('[G54:')) {
      parseG54Offset(trimmed);
    } else if (trimmed.toLowerCase().startsWith('alarm:') || trimmed.includes('ALARM:')) {
      setAlarmState(trimmed);
    }
```

Replace with:

```
    if (trimmed.startsWith('<') && trimmed.endsWith('>')) {
      parseStatusReport(trimmed);
    } else if (/^\[G5[4-9]:/.test(trimmed)) {
      parseWCSOffsetLine(trimmed);
    } else if (trimmed.toLowerCase().startsWith('alarm:') || trimmed.includes('ALARM:')) {
      setAlarmState(trimmed);
    }
```

---

## EDIT 8 — Replace parseG54Offset with parseWCSOffsetLine

Find:

```
function parseG54Offset(line) {
  const match = line.match(/\[G54:([-\d.]+),([-\d.]+),([-\d.]+)/);
  if (match) {
    App.wco.x = parseFloat(match[1]) || 0;
    App.wco.y = parseFloat(match[2]) || 0;
    App.wco.z = parseFloat(match[3]) || 0;
    console.log('G54 offset:', App.wco);
    App.position.wcs.x = App.position.mcs.x - App.wco.x;
    App.position.wcs.y = App.position.mcs.y - App.wco.y;
    App.position.wcs.z = App.position.mcs.z - App.wco.z;
    updatePositionDisplay();
  }
}
```

Replace with:

```
function parseWCSOffsetLine(line) {
  const m = line.match(/\[G(5[4-9]):([-\d.]+),([-\d.]+),([-\d.]+)/);
  if (!m) return;
  const x = parseFloat(m[2]) || 0;
  const y = parseFloat(m[3]) || 0;
  const z = parseFloat(m[4]) || 0;
  const slotName = 'G' + m[1];

  if (slotName === 'G54') {
    App.wco.x = x; App.wco.y = y; App.wco.z = z;
    App.position.wcs.x = App.position.mcs.x - App.wco.x;
    App.position.wcs.y = App.position.mcs.y - App.wco.y;
    App.position.wcs.z = App.position.mcs.z - App.wco.z;
    updatePositionDisplay();
  }

  const idx = parseInt(m[1]) - 54; // 54→0 … 59→5
  if (idx >= 0 && idx < 6) {
    App.wcsOffsets[idx].x = x;
    App.wcsOffsets[idx].y = y;
    App.wcsOffsets[idx].z = z;
  }
  renderWCSTable(); // re-render after each line; fine since all 6 arrive quickly
}
```

---

## EDIT 9 — Emergency stop: ensure laser off + reset laser state

Find:

```
function emergencyStop() {
  console.log('*** EMERGENCY STOP ***');
  if (App.ws && App.isConnected) {
    App.ws.send('!');
    setTimeout(() => { if (App.ws && App.isConnected) App.ws.send('\x18'); }, 100);
  }
  stopContinuousJog();
  App.probeInProgress = false;
}
```

Replace with:

```
function emergencyStop() {
  console.log('*** EMERGENCY STOP ***');
  if (App.ws && App.isConnected) {
    App.ws.send('!');
    setTimeout(() => { if (App.ws && App.isConnected) App.ws.send('\x18'); }, 100);
    setTimeout(() => { if (App.ws && App.isConnected) App.ws.send('M5\n'); }, 300);
  }
  stopContinuousJog();
  App.probeInProgress = false;
  // Reset laser UI state
  if (App.laserLowActive) {
    App.laserLowActive = false;
    const btn = document.getElementById('laser-low-btn');
    const ind = document.getElementById('laser-low-ind');
    if (btn) { btn.textContent = 'Fire Low Power (toggle)'; btn.style.cssText = ''; }
    if (ind) ind.style.display = 'none';
  }
  const highBtn = document.getElementById('laser-high-btn');
  const highInd = document.getElementById('laser-high-ind');
  if (highBtn) highBtn.classList.remove('firing');
  if (highInd) highInd.style.display = 'none';
}
```

---

## EDIT 10 — New JS regions: Insert after stopContinuousJog / end of CONTROL TAB region

Find:

```
function stopContinuousJog() {
  if (App.isJogging) {
    App.isJogging = false;
    if (App.ws && App.isConnected) App.ws.send(String.fromCharCode(0x85));
  }
}
//#endregion
```

Replace with:

```
function stopContinuousJog() {
  if (App.isJogging) {
    App.isJogging = false;
    if (App.ws && App.isConnected) App.ws.send(String.fromCharCode(0x85));
  }
}
//#endregion

//#region ===== MCS MOVE =====
function moveToMCS() {
  if (!App.isConnected) { alert('Not connected'); return; }
  const parts = [];
  if (document.getElementById('mcs-x-en').checked) {
    const v = parseFloat(document.getElementById('mcs-x-val').value);
    if (isNaN(v)) { alert('Invalid X value'); return; }
    parts.push('X' + v.toFixed(3));
  }
  if (document.getElementById('mcs-y-en').checked) {
    const v = parseFloat(document.getElementById('mcs-y-val').value);
    if (isNaN(v)) { alert('Invalid Y value'); return; }
    parts.push('Y' + v.toFixed(3));
  }
  if (document.getElementById('mcs-z-en').checked) {
    const v = parseFloat(document.getElementById('mcs-z-val').value);
    if (isNaN(v)) { alert('Invalid Z value'); return; }
    parts.push('Z' + v.toFixed(3));
  }
  if (!parts.length) { alert('Check at least one axis to move.'); return; }
  sendCommand('G53 G0 ' + parts.join(' '));
}

function fillMCSFromCurrent() {
  document.getElementById('mcs-x-val').value = App.position.mcs.x.toFixed(3);
  document.getElementById('mcs-y-val').value = App.position.mcs.y.toFixed(3);
  document.getElementById('mcs-z-val').value = App.position.mcs.z.toFixed(3);
  document.getElementById('mcs-x-en').checked = true;
  document.getElementById('mcs-y-en').checked = true;
  document.getElementById('mcs-z-en').checked = true;
}
//#endregion

//#region ===== POSITION LOG =====
function recordPosition() {
  const id = ++App.posLogIdSeq;
  App.posLog.push({
    id,
    label: 'Pos ' + id,
    mcs: { ...App.position.mcs },
    wcs: { ...App.position.wcs },
    time: new Date().toLocaleTimeString(),
    selected: false
  });
  renderPositionLog();
}

function renderPositionLog() {
  const wrap = document.getElementById('pos-log-wrap');
  if (!App.posLog.length) {
    wrap.innerHTML = '<p class="text-muted" style="font-size:12px;">No positions recorded yet.</p>';
    return;
  }
  let h = '<table class="poslog-table"><thead><tr>' +
    '<th class="pl-check">✓</th><th class="pl-label">Label</th><th>Time</th>' +
    '<th>Mach X</th><th>Mach Y</th><th>Mach Z</th>' +
    '<th>Job X</th><th>Job Y</th><th>Job Z</th><th></th>' +
    '</tr></thead><tbody>';
  for (const e of App.posLog) {
    const bg = e.selected ? 'background:var(--bg-tertiary);' : '';
    h += `<tr style="${bg}">
      <td class="pl-check"><input type="checkbox" ${e.selected ? 'checked' : ''}
        onchange="togglePosLogSel(${e.id},this.checked)"></td>
      <td class="pl-label"><input type="text" value="${e.label}"
        onchange="renamePosLog(${e.id},this.value)"></td>
      <td>${e.time}</td>
      <td>${e.mcs.x.toFixed(3)}</td><td>${e.mcs.y.toFixed(3)}</td><td>${e.mcs.z.toFixed(3)}</td>
      <td>${e.wcs.x.toFixed(3)}</td><td>${e.wcs.y.toFixed(3)}</td><td>${e.wcs.z.toFixed(3)}</td>
      <td class="pl-acts">
        <button onclick="gotoPosLog(${e.id})" title="Move to this machine position">Go</button>
        <button onclick="deletePosLog(${e.id})">✕</button>
      </td>
    </tr>`;
  }
  wrap.innerHTML = h + '</tbody></table>';
}

function togglePosLogSel(id, checked) {
  const e = App.posLog.find(p => p.id === id);
  if (e) e.selected = checked;
}

function renamePosLog(id, label) {
  const e = App.posLog.find(p => p.id === id);
  if (e) e.label = label;
}

function gotoPosLog(id) {
  if (!App.isConnected) { alert('Not connected'); return; }
  const e = App.posLog.find(p => p.id === id);
  if (!e) return;
  sendCommand(`G53 G0 X${e.mcs.x.toFixed(3)} Y${e.mcs.y.toFixed(3)} Z${e.mcs.z.toFixed(3)}`);
}

function deletePosLog(id) {
  App.posLog = App.posLog.filter(p => p.id !== id);
  renderPositionLog();
}

function diffSelectedPositions() {
  const sel = App.posLog.filter(p => p.selected);
  if (sel.length !== 2) { alert('Check exactly 2 entries to diff them.'); return; }
  const [a, b] = sel;
  const fmt = (n) => (n >= 0 ? '+' : '') + n.toFixed(3);
  const out =
    `Diff: "${a.label}"  →  "${b.label}"\n` +
    `Machine: ΔX=${fmt(b.mcs.x - a.mcs.x)}  ΔY=${fmt(b.mcs.y - a.mcs.y)}  ΔZ=${fmt(b.mcs.z - a.mcs.z)}\n` +
    `Job:     ΔX=${fmt(b.wcs.x - a.wcs.x)}  ΔY=${fmt(b.wcs.y - a.wcs.y)}  ΔZ=${fmt(b.wcs.z - a.wcs.z)}`;
  const el = document.getElementById('pos-log-diff');
  el.textContent = out;
  el.style.display = 'block';
}

function clearPositionLog() {
  if (App.posLog.length && !confirm('Clear all recorded positions?')) return;
  App.posLog = [];
  App.posLogIdSeq = 0;
  renderPositionLog();
  const el = document.getElementById('pos-log-diff');
  el.style.display = 'none';
}
//#endregion

//#region ===== WCS / JOB ORIGINS MANAGER =====
function refreshWCSTable() {
  if (!App.isConnected) { alert('Not connected'); return; }
  sendCommand('$#');
}

function renderWCSTable() {
  const tbody = document.getElementById('wcs-tbody');
  if (!tbody) return;
  let h = '';
  for (let i = 0; i < App.wcsOffsets.length; i++) {
    const w = App.wcsOffsets[i];
    h += `<tr>
      <td class="wcs-slot">${w.slot}</td>
      <td class="wcs-note"><span style="font-size:11px;">${w.note}</span></td>
      <td>${w.x.toFixed(3)}</td><td>${w.y.toFixed(3)}</td><td>${w.z.toFixed(3)}</td>
      <td class="wcs-acts">
        <button onclick="gotoWCS(${i})" title="Move to this job origin position (G53 move to offset coords)">Go</button>
        <button onclick="setWCSHere(${i})" title="Make current position this slot's job origin (G10 L20)"
          style="color:var(--warning);">Set Here</button>
        <button onclick="openWCSEdit(${i})" title="Enter explicit machine coordinates for this origin">Set Values</button>
      </td>
    </tr>`;
  }
  tbody.innerHTML = h;
}

function gotoWCS(idx) {
  if (!App.isConnected) { alert('Not connected'); return; }
  const w = App.wcsOffsets[idx];
  // The stored values are the offset (machine coords = job 0,0,0)
  sendCommand(`G53 G0 X${w.x.toFixed(3)} Y${w.y.toFixed(3)} Z${w.z.toFixed(3)}`);
}

function setWCSHere(idx) {
  if (!App.isConnected) { alert('Not connected'); return; }
  const w = App.wcsOffsets[idx];
  if (!confirm(
    `Set ${w.slot} job origin to current position?\n\n` +
    `Sends: G10 L20 P${w.pNum} X0 Y0 Z0\n\n` +
    `This makes wherever you are right now become XYZ 0,0,0 for ${w.slot}.\n` +
    `Writes to controller EEPROM.`
  )) return;
  sendCommand(`G10 L20 P${w.pNum} X0 Y0 Z0`);
  setTimeout(() => sendCommand('$#'), 300);
}

function openWCSEdit(idx) {
  const w = App.wcsOffsets[idx];
  App.wcsEditIdx = idx;
  document.getElementById('wcs-edit-label').textContent = w.slot;
  document.getElementById('wcs-edit-x').value = w.x.toFixed(3);
  document.getElementById('wcs-edit-y').value = w.y.toFixed(3);
  document.getElementById('wcs-edit-z').value = w.z.toFixed(3);
  const panel = document.getElementById('wcs-edit-panel');
  panel.classList.add('visible');
  panel.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
}

function applyWCSEdit() {
  if (!App.isConnected) { alert('Not connected'); return; }
  if (App.wcsEditIdx === null) return;
  const w = App.wcsOffsets[App.wcsEditIdx];
  const x = parseFloat(document.getElementById('wcs-edit-x').value);
  const y = parseFloat(document.getElementById('wcs-edit-y').value);
  const z = parseFloat(document.getElementById('wcs-edit-z').value);
  if ([x, y, z].some(isNaN)) { alert('All three values must be valid numbers.'); return; }
  if (!confirm(
    `Set ${w.slot} job origin to machine position X${x.toFixed(3)} Y${y.toFixed(3)} Z${z.toFixed(3)}?\n\n` +
    `Sends: G10 L2 P${w.pNum} X${x.toFixed(3)} Y${y.toFixed(3)} Z${z.toFixed(3)}\n` +
    `Writes to controller EEPROM.`
  )) return;
  sendCommand(`G10 L2 P${w.pNum} X${x.toFixed(3)} Y${y.toFixed(3)} Z${z.toFixed(3)}`);
  cancelWCSEdit();
  setTimeout(() => sendCommand('$#'), 300);
}

function cancelWCSEdit() {
  App.wcsEditIdx = null;
  document.getElementById('wcs-edit-panel').classList.remove('visible');
}

function clearG54() {
  if (!App.isConnected) { alert('Not connected'); return; }
  if (!confirm(
    'Reset G54 job origin to zero?\n\n' +
    'Sends: G10 L2 P1 X0 Y0 Z0\n\n' +
    'After homing, job position will equal machine position.\n' +
    'Use this before starting LightBurn in Absolute Coords mode.'
  )) return;
  sendCommand('G10 L2 P1 X0 Y0 Z0');
  setTimeout(() => { sendCommand('$#'); setTimeout(() => sendCommand('?'), 100); }, 200);
}
//#endregion

//#region ===== RETURN POINTS (G28 / G30) =====
function saveReturnPoint(num) {
  if (!App.isConnected) { alert('Not connected'); return; }
  const cmd = num === 1 ? 'G28.1' : 'G30.1';
  if (!confirm(
    `Save current machine position as Return Point ${num}?\n\n` +
    `Sends: ${cmd}\n\n` +
    `Current position: X${App.position.mcs.x.toFixed(3)} Y${App.position.mcs.y.toFixed(3)} Z${App.position.mcs.z.toFixed(3)}\n\n` +
    `This overwrites any previously saved Return Point ${num}. Stored in controller EEPROM.`
  )) return;
  sendCommand(cmd);
}

function gotoReturnPoint(num) {
  if (!App.isConnected) { alert('Not connected'); return; }
  const cmd = num === 1 ? 'G28' : 'G30';
  sendCommand(cmd);
}
//#endregion

//#region ===== TOOL OFFSET =====
function updateOffsetDisplay() {
  const el = document.getElementById('offset-display');
  const to = App.settings && App.settings.toolOffset;
  if (!to || (to.x === 0 && to.y === 0)) {
    el.textContent = 'Not measured yet';
    el.style.color = 'var(--text-muted)';
  } else {
    const fx = (to.x >= 0 ? '+' : '') + to.x.toFixed(3);
    const fy = (to.y >= 0 ? '+' : '') + to.y.toFixed(3);
    el.textContent = `X: ${fx} mm   Y: ${fy} mm`;
    el.style.color = 'var(--accent)';
  }
}

function moveToLaserZero() {
  if (!App.isConnected) { alert('Not connected'); return; }
  const to = App.settings && App.settings.toolOffset;
  if (!to || (to.x === 0 && to.y === 0)) {
    if (!confirm('Tool offset is zero or not measured yet. Move to job origin XY anyway?')) return;
  }
  const x = (to ? to.x : 0).toFixed(3);
  const y = (to ? to.y : 0).toFixed(3);
  sendCommand(`G90 G0 X${x} Y${y}`);
}

function recordSpindlePos() {
  App.offsetSpindlePos = { x: App.position.wcs.x, y: App.position.wcs.y };
  document.getElementById('offset-spindle-recorded').textContent =
    `✓ Recorded: X${App.offsetSpindlePos.x.toFixed(3)} Y${App.offsetSpindlePos.y.toFixed(3)}`;
  document.getElementById('offset-spindle-recorded').style.color = 'var(--success)';
  document.getElementById('offset-step1').classList.add('offset-step-done');
  checkOffsetCalc();
}

function recordLaserPos() {
  App.offsetLaserPos = { x: App.position.wcs.x, y: App.position.wcs.y };
  document.getElementById('offset-laser-recorded').textContent =
    `✓ Recorded: X${App.offsetLaserPos.x.toFixed(3)} Y${App.offsetLaserPos.y.toFixed(3)}`;
  document.getElementById('offset-laser-recorded').style.color = 'var(--success)';
  document.getElementById('offset-step2').classList.add('offset-step-done');
  checkOffsetCalc();
}

function checkOffsetCalc() {
  if (!App.offsetSpindlePos || !App.offsetLaserPos) return;
  const dx = App.offsetLaserPos.x - App.offsetSpindlePos.x;
  const dy = App.offsetLaserPos.y - App.offsetSpindlePos.y;
  const fx = (dx >= 0 ? '+' : '') + dx.toFixed(3);
  const fy = (dy >= 0 ? '+' : '') + dy.toFixed(3);
  document.getElementById('offset-calc-display').textContent = `ΔX=${fx} mm   ΔY=${fy} mm`;
  document.getElementById('offset-calc-row').style.display = 'flex';
  document.getElementById('offset-calc-row').style.alignItems = 'center';
  // Store for add button
  App._pendingOffset = { x: dx, y: dy };
}

function addOffsetMeasurement() {
  if (!App._pendingOffset) return;
  App.offsetMeasurements.push({ ...App._pendingOffset });
  App._pendingOffset = null;
  // Reset steps
  App.offsetSpindlePos = null;
  App.offsetLaserPos = null;
  document.getElementById('offset-spindle-recorded').textContent = '';
  document.getElementById('offset-laser-recorded').textContent = '';
  document.getElementById('offset-step1').classList.remove('offset-step-done');
  document.getElementById('offset-step2').classList.remove('offset-step-done');
  document.getElementById('offset-calc-row').style.display = 'none';
  renderOffsetMeasurements();
}

function renderOffsetMeasurements() {
  const wrap = document.getElementById('offset-meas-wrap');
  const tbody = document.getElementById('offset-meas-tbody');
  const avgEl = document.getElementById('offset-avg-display');
  if (!App.offsetMeasurements.length) {
    wrap.style.display = 'none';
    return;
  }
  wrap.style.display = 'block';
  const avgX = App.offsetMeasurements.reduce((s, m) => s + m.x, 0) / App.offsetMeasurements.length;
  const avgY = App.offsetMeasurements.reduce((s, m) => s + m.y, 0) / App.offsetMeasurements.length;
  const fmt = (n) => (n >= 0 ? '+' : '') + n.toFixed(3);
  avgEl.textContent = `Average: ΔX=${fmt(avgX)}  ΔY=${fmt(avgY)}`;
  let rows = '';
  App.offsetMeasurements.forEach((m, i) => {
    rows += `<tr>
      <td>${i + 1}</td>
      <td>${fmt(m.x)}</td>
      <td>${fmt(m.y)}</td>
      <td class="om-acts"><button onclick="deleteOffsetMeas(${i})">✕</button></td>
    </tr>`;
  });
  tbody.innerHTML = rows;
}

function deleteOffsetMeas(idx) {
  App.offsetMeasurements.splice(idx, 1);
  renderOffsetMeasurements();
}

function clearOffsetMeasurements() {
  if (!confirm('Clear all offset measurements?')) return;
  App.offsetMeasurements = [];
  renderOffsetMeasurements();
}

function saveAverageOffset() {
  if (!App.offsetMeasurements.length) { alert('No measurements to average.'); return; }
  const avgX = App.offsetMeasurements.reduce((s, m) => s + m.x, 0) / App.offsetMeasurements.length;
  const avgY = App.offsetMeasurements.reduce((s, m) => s + m.y, 0) / App.offsetMeasurements.length;
  const fmt = (n) => (n >= 0 ? '+' : '') + n.toFixed(3);
  if (!confirm(
    `Save average offset as stored tool offset?\n\n` +
    `ΔX=${fmt(avgX)} mm   ΔY=${fmt(avgY)} mm\n` +
    `(based on ${App.offsetMeasurements.length} measurement${App.offsetMeasurements.length > 1 ? 's' : ''})\n\n` +
    `This will update the settings and the Settings tab.`
  )) return;
  if (!App.settings.toolOffset) App.settings.toolOffset = {};
  App.settings.toolOffset.x = parseFloat(avgX.toFixed(3));
  App.settings.toolOffset.y = parseFloat(avgY.toFixed(3));
  document.getElementById('setting-offset-x').value = App.settings.toolOffset.x;
  document.getElementById('setting-offset-y').value = App.settings.toolOffset.y;
  saveSettingsToEmbedded();
  updateSettingsJSON();
  updateOffsetDisplay();
  alert(`Tool offset saved: ΔX=${fmt(avgX)}  ΔY=${fmt(avgY)}\n\nUse "Move Laser to Spindle Zero" to apply it.`);
}
//#endregion

//#region ===== TOOL & LASER =====
function selectTool(toolNum) {
  if (!App.isConnected) { alert('Not connected'); return; }
  if (App.activeTool === 1 && toolNum === 0 && App.laserLowActive) {
    sendCommand('M5');
    App.laserLowActive = false;
  }
  sendCommand('M6 T' + toolNum);
  App.activeTool = toolNum;
  updateToolUI();
}

function updateToolUI() {
  const isLaser = App.activeTool === 1;
  // Header badge
  const badge = document.getElementById('header-tool-badge');
  badge.textContent = isLaser ? 'LASER' : 'SPINDLE';
  badge.className = 'tool-badge ' + (isLaser ? 'tool-badge-laser' : 'tool-badge-spindle');
  // Active label
  const lbl = document.getElementById('tool-active-label');
  lbl.textContent = isLaser ? 'Laser (T1)' : 'Spindle (T0)';
  lbl.style.color = isLaser ? 'var(--warning)' : 'var(--success)';
  // Tool buttons
  document.getElementById('tool-spindle-btn').className =
    isLaser ? '' : 'btn-tool-spindle-active';
  document.getElementById('tool-laser-btn').className =
    isLaser ? 'btn-tool-laser-active' : '';
  // Laser section enabled/disabled
  const inner = document.getElementById('laser-inner');
  const notice = document.getElementById('laser-disabled-notice');
  inner.classList.toggle('laser-section-disabled', !isLaser);
  notice.style.display = isLaser ? 'none' : '';
  // Sync settings into laser inputs
  if (App.settings) {
    const ls = App.settings.laser || {};
    const lowPct = ls.lowPowerPct || 7;
    const highMax = ls.highPowerMaxPct || 50;
    document.getElementById('laser-low-pct').value = lowPct;
    document.getElementById('laser-high-pct').setAttribute('max', highMax);
    const curHigh = parseInt(document.getElementById('laser-high-pct').value) || 30;
    if (curHigh > highMax) document.getElementById('laser-high-pct').value = highMax;
  }
  updatePWMHints();
}

function updatePWMHints() {
  const lowPct  = parseInt(document.getElementById('laser-low-pct').value)  || 7;
  const highPct = parseInt(document.getElementById('laser-high-pct').value) || 30;
  const maxPct  = parseInt(document.getElementById('laser-high-pct').getAttribute('max')) || 50;
  const clampedHigh = Math.min(maxPct, highPct);
  document.getElementById('laser-low-pwm-hint').textContent =
    '= S' + Math.round(Math.min(10, lowPct) / 100 * 255);
  document.getElementById('laser-high-pwm-hint').textContent =
    '= S' + Math.round(clampedHigh / 100 * 255);
}

function toggleLaserLow() {
  if (!App.isConnected) { alert('Not connected'); return; }
  if (App.activeTool !== 1) { alert('Switch to Laser (T1) first using the Active Tool buttons.'); return; }
  App.laserLowActive = !App.laserLowActive;
  const pct   = Math.min(10, Math.max(1, parseInt(document.getElementById('laser-low-pct').value) || 7));
  const power = Math.round(pct / 100 * 255);
  const btn   = document.getElementById('laser-low-btn');
  const ind   = document.getElementById('laser-low-ind');
  if (App.laserLowActive) {
    sendCommand('G90 M3 S' + power + ' G1 F1');
    btn.textContent = '■ Stop Low Power';
    btn.style.background = 'var(--warning)';
    btn.style.color = '#000';
    ind.style.display = '';
  } else {
    sendCommand('M5');
    btn.textContent = 'Fire Low Power (toggle)';
    btn.style.background = '';
    btn.style.color = '';
    ind.style.display = 'none';
  }
}

function setupHighPowerLaser() {
  const btn = document.getElementById('laser-high-btn');
  const fire = (e) => {
    e.preventDefault();
    if (!App.isConnected) return;
    if (App.activeTool !== 1) { alert('Switch to Laser (T1) first.'); return; }
    if (App.laserLowActive) { alert('Turn off low power laser before using high power.'); return; }
    if (btn.classList.contains('firing')) return; // already firing
    const maxPct  = parseInt(btn.getAttribute('max') ||
      (App.settings && App.settings.laser ? App.settings.laser.highPowerMaxPct : 50)) || 50;
    const inputPct = parseInt(document.getElementById('laser-high-pct').value) || 30;
    const pct   = Math.min(maxPct, Math.max(1, inputPct));
    const power = Math.round(pct / 100 * 255);
    sendCommand('G90 M3 S' + power + ' G1 F1');
    btn.classList.add('firing');
    document.getElementById('laser-high-ind').style.display = '';
  };
  const stop = () => {
    if (btn.classList.contains('firing')) {
      sendCommand('M5');
      btn.classList.remove('firing');
      document.getElementById('laser-high-ind').style.display = 'none';
    }
  };
  btn.addEventListener('mousedown',   fire);
  btn.addEventListener('mouseup',     stop);
  btn.addEventListener('mouseleave',  stop);
  btn.addEventListener('touchstart',  fire, { passive: false });
  btn.addEventListener('touchend',    stop);
  btn.addEventListener('touchcancel', stop);
  btn.addEventListener('contextmenu', (e) => { e.preventDefault(); stop(); });
}
//#endregion

//#region ===== ALARM CLEAR =====
function clearAlarm() {
  if (!App.isConnected) { alert('Not connected'); return; }
  sendCommand('$X');
  setTimeout(() => sendCommand('?'), 200);
}
//#endregion
```

---

## EDIT 11 — Settings collect: Add laser and toolOffset

Find:

```
    defaults: {
      probeDistances: {
        cylinder: parseFloat(document.getElementById('setting-default-cyl-distance').value) || 20,
        hole: parseFloat(document.getElementById('setting-default-hole-distance').value) || 20,
        corner: parseFloat(document.getElementById('setting-default-corner-distance').value) || 20,
        side: parseFloat(document.getElementById('setting-default-side-distance').value) || 5
      },
      zSafeHeights: {
        standard: parseFloat(document.getElementById('setting-default-z-safe').value) || 10,
        side: parseFloat(document.getElementById('setting-default-side-z-safe').value) || 10,
        heightmap: parseFloat(document.getElementById('setting-default-heightmap-z-safe').value) || 20
      }
    },
    presets: App.settings.presets
```

Replace with:

```
    defaults: {
      probeDistances: {
        cylinder: parseFloat(document.getElementById('setting-default-cyl-distance').value) || 20,
        hole: parseFloat(document.getElementById('setting-default-hole-distance').value) || 20,
        corner: parseFloat(document.getElementById('setting-default-corner-distance').value) || 20,
        side: parseFloat(document.getElementById('setting-default-side-distance').value) || 5
      },
      zSafeHeights: {
        standard: parseFloat(document.getElementById('setting-default-z-safe').value) || 10,
        side: parseFloat(document.getElementById('setting-default-side-z-safe').value) || 10,
        heightmap: parseFloat(document.getElementById('setting-default-heightmap-z-safe').value) || 20
      }
    },
    laser: {
      lowPowerPct:    Math.min(10, parseInt(document.getElementById('setting-laser-low').value) || 7),
      highPowerMaxPct: parseInt(document.getElementById('setting-laser-high-max').value) || 50
    },
    toolOffset: {
      x: parseFloat(document.getElementById('setting-offset-x').value) || 0,
      y: parseFloat(document.getElementById('setting-offset-y').value) || 0
    },
    presets: App.settings.presets
```

---

## EDIT 12 — Settings apply: Add laser and toolOffset

Find:

```
  document.getElementById('setting-jog-xy').value = App.settings.jog.xySpeed;
  document.getElementById('setting-jog-z').value = App.settings.jog.zSpeed;
  document.getElementById('setting-mist-on').value = App.settings.outputs.mistOn;
```

Replace with:

```
  document.getElementById('setting-jog-xy').value = App.settings.jog.xySpeed;
  document.getElementById('setting-jog-z').value = App.settings.jog.zSpeed;
  if (App.settings.laser) {
    document.getElementById('setting-laser-low').value     = App.settings.laser.lowPowerPct     || 7;
    document.getElementById('setting-laser-high-max').value = App.settings.laser.highPowerMaxPct || 50;
  }
  if (App.settings.toolOffset) {
    document.getElementById('setting-offset-x').value = App.settings.toolOffset.x || 0;
    document.getElementById('setting-offset-y').value = App.settings.toolOffset.y || 0;
  }
  document.getElementById('setting-mist-on').value = App.settings.outputs.mistOn;
```

---

## EDIT 13 — Init: Wire up new components

Find:

```
  loadCylinderPresets();
  loadHolePresets();
  loadCornerPresets();
  loadHeightmapPresets();
```

Replace with:

```
  loadCylinderPresets();
  loadHolePresets();
  loadCornerPresets();
  loadHeightmapPresets();

  // New control tab features
  setupHighPowerLaser();
  updateToolUI();
  updateOffsetDisplay();
  renderPositionLog();

  // Live PWM hints when power inputs change
  document.getElementById('laser-low-pct').addEventListener('input', updatePWMHints);
  document.getElementById('laser-high-pct').addEventListener('input', updatePWMHints);
```

---

## EDIT 14 — Version string and title

Find:

```
const VERSION = '1.13.5';
```

Replace with:

```
const VERSION = '1.15.0';
```

Find:

```
<title>FluidNC Probe Utility v1.14.0</title>
```

Replace with:

```
<title>FluidNC Probe Utility v1.15.0</title>
```

---

## POST-EDIT NOTES

**Laser firing** — confirmed working sequence: `G90 M3 S{n} G1 F1` as a single sendCommand call. `M5` to stop. Required because `$32=1` (laser mode) gates PWM output on motion; a zero-distance G1 is discarded by the planner, but sending it on the same line as M3 works.

**Tool offset "Move Laser to Spindle Zero"** sends `G90 G0 X{offsetX} Y{offsetY}` — this moves in WCS (job coordinates), so the laser ends up at the physical location where the spindle was zeroed. After arriving, user zeros job XY manually as normal.

**WCS table notes column** — the `note` field in each `wcsOffsets` entry is display-only in the rendered table. Future: make it editable inline.

**G28/G30 confirm dialogs** show the current machine position so user can verify before saving — important guard against accidentally overwriting a good return point.

**The `btn` attribute `max` lookup** in `setupHighPowerLaser` has a fallback chain: button attribute → settings → hardcoded 50. The button doesn't actually have a `max` attribute set in HTML, so it will always fall through to settings. This is correct behavior — simplify if desired by removing the attribute lookup.
