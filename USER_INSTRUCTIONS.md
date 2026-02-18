# FluidNC Probe Utility — User Instructions

**Version 1.15.1**

This guide covers everything you need to use the utility from first-time setup through daily workflows.

---

## Table of Contents

1. [First-Time Setup](#1-first-time-setup)
2. [Session Startup Checklist](#2-session-startup-checklist)
3. [Saving Your Settings](#3-saving-your-settings)
4. [Control Tab](#4-control-tab)
   - Position Display
   - Jogging
   - Move to Machine Position
   - Position Log
   - Saved Job Origins (G54–G59)
   - Return Points (G28 / G30)
   - Tool Offset — Spindle to Laser
   - Active Tool Selector
   - Laser Control
5. [Cylinder Probe](#5-cylinder-probe)
6. [Hole Probe](#6-hole-probe)
7. [Corner Probe](#7-corner-probe)
8. [Side Align](#8-side-align)
9. [Height Map](#9-height-map)
10. [Settings Tab](#10-settings-tab)
11. [Coordinate System Concepts](#11-coordinate-system-concepts)
12. [Troubleshooting](#12-troubleshooting)

---

## 1. First-Time Setup

### Step 1 — Open the utility
Double-click `FluidNC_Probe_Utility_v1.15.1.html` (or whichever is the latest version). It opens in your default browser. If it opens in Internet Explorer, right-click the file and choose Open With → Chrome, Firefox, or Edge.

### Step 2 — Enter connection details
In the header at the top of the page:
- **IP Address** — enter your FluidNC controller's IP (e.g., `192.168.1.50`). You can find this in your router's DHCP table or in the FluidNC WebUI.
- **Port** — leave as `81` (FluidNC WebSocket default)

### Step 3 — Connect
Click **Connect**. The status indicator turns green and shows "Connected" when successful.

If connection fails, check:
- The IP address is correct
- Your computer is on the same network as the controller
- FluidNC is powered on and running
- No firewall is blocking port 81

### Step 4 — Set your probe parameters (Settings tab)
Go to the **Settings** tab and configure:
- **Tip Diameter** — diameter of your probe ball (e.g., `3.0` mm for a 3mm Ruby ball)
- **Fast Feedrate** — probing speed for first pass (e.g., `100` mm/min)
- **Slow Feedrate** — precision speed for second pass (e.g., `50` mm/min)
- **Retract Distance** — how far to pull back between fast and slow probe (e.g., `2.0` mm)
- **Jog XY Speed** / **Z Speed** — how fast the jog buttons move the machine

### Step 5 — Save your settings
Press **Ctrl+S** in the browser and save the HTML file back to the same location. Your settings are now embedded in the file and will be there next time you open it.

---

## 2. Session Startup Checklist

Every time you start a session:

1. **Open the HTML file** and click **Connect**
2. **Click Home All ($H)** on the Control tab — *always home before doing anything else*
3. If this is a new job: jog to your work origin and **Set Job Origin XY Here**
4. If returning to an existing job: click **Go There** on Return Point 1 (if you saved it)

---

## 3. Saving Your Settings

Settings live inside the HTML file itself. To save permanently:

1. Make changes in the Settings tab (IP address, feedrates, presets, tool offset, etc.)
2. Press **Ctrl+S** in the browser
3. Choose "Save As" and overwrite the same HTML file

**Everything is saved:** IP address, probe feedrates, jog speeds, laser settings, tool offset, output M-codes, all presets, default probe distances.

> Note: The position log and tool offset measurement accumulator are **session only** — they clear when you close or refresh the page. The stored tool offset in Settings is permanent.

---

## 4. Control Tab

The Control tab is organized as a 4-section grid at the top, followed by full-width sections below.

---

### Position Display

Shows two coordinate readings for each axis:

| Column | What it means |
|--------|--------------|
| **Machine Pos** | Absolute position from homing switches. 0,0,0 is always at the switches. Never changes unless you re-home or move a switch. |
| **Job Pos (G54)** | Position relative to your job origin. 0,0,0 is wherever you set your job origin (Set Job Origin XY Here). |

**Buttons:**
- **Zero X / Zero Y / Zero Z** — Sets that axis's job origin to the current machine position. Equivalent to `G10 L20 P1 X0` etc.
- **Set Job Origin XY Here** — Sets XY job origin at the current position. After clicking, an **↩ Undo Zero XY** row appears — click it if you made a mistake, and it restores the previous offset.
- **Go to Job Origin XY** — Rapids to X0 Y0 in job coordinates.
- **Go Z 0** — Rapids to Z0 in job coordinates.
- **↻ Refresh** — Requests a fresh position report from the controller.

---

### Home Section

- **Home All ($H)** — Homes all axes. Do this at the start of every session.
- **Home X / Home Y / Home Z** — Home individual axes.

> Always home before using Return Points, G53 moves, or the WCS Manager. Machine position is meaningless without a home cycle.

---

### Jog Section

- Click and **hold** the arrow buttons for continuous jogging.
- XY buttons move X and Y; Z buttons move Z up and down.
- Jog speeds are set in the Settings tab.

---

### Outputs Section

Toggle outputs on/off:
- **Mist** — Turns mist coolant on/off (M7/M9 or custom M-code from Settings)
- **Vacuum** — Turns vacuum on/off
- **IoT PDU** — Controls a smart power strip or similar device

M-codes for each output are configured in the Settings tab.

---

### Move to Machine Position

Use this to move to an exact machine coordinate, bypassing job origins. Useful for returning to a known fixture position or loading position.

1. Check the checkboxes for the axes you want to move
2. Enter the machine coordinates (negative numbers are normal — machine origin is at the switches)
3. Click **Go** — sends `G53 G0 X... Y... Z...`
4. **← From Current** — populates all three fields with your current machine position (useful for recording and re-using positions)

> ⚠ Make sure Z is clear before moving XY to avoid crashing into fixtures or clamps.

---

### Position Log

Records machine and job positions during your session. Useful for:
- Measuring the offset between two features
- Remembering where you were before moving elsewhere
- Calculating differences between probe positions

**How to use:**
1. Move to a position you want to record
2. Click **● Record Current Position**
3. Each entry shows: editable label, timestamp, Machine X/Y/Z, Job X/Y/Z
4. **Go** button on each entry returns to that exact machine position
5. To measure the difference between two positions:
   - Check the checkboxes on exactly two entries
   - Click **Diff 2 Selected**
   - ΔX/Y/Z shown for both machine and job coordinates

The log clears when you close or refresh the page.

---

### Saved Job Origins (G54–G59)

The controller has six coordinate system slots (G54 through G59). Each one stores a machine position that becomes "XYZ 0,0,0" for jobs using that slot.

**G54 is the default** — almost all G-code files use G54. The other slots (G55–G59) are for special purposes like a secondary fixture, a laser offset, or a different tool datum.

**To load current values from the controller:**
Click **↻ Refresh from Controller** — sends `$#` and populates the table with all six slots.

**Per-slot buttons:**
- **Go** — Moves the machine to the position stored in that slot (the machine position that is job origin 0,0,0 for that slot)
- **Set Here** — Makes your current machine position the job origin for that slot (`G10 L20 P{n} X0 Y0 Z0`). Shows a confirm dialog before writing to EEPROM.
- **Set Values** — Opens an edit panel where you can type explicit machine coordinates for that slot's origin (`G10 L2 P{n} X... Y... Z...`). Use this to set a known fixture position by number.

**Reset Job Origin G54 to Zero:**
Sends `G10 L2 P1 X0 Y0 Z0`. After this, job position equals machine position (both are zero at the homing switches). Use this before running LightBurn in Absolute Coords mode.

> Changes to WCS offsets are written to controller EEPROM immediately — they survive power-off and controller restarts.

---

### Return Points (G28 / G30)

Two parking spots stored in the controller (survive power-off and controller restarts). Use these to return to a work position accurately after any home cycle.

**Typical PCB milling workflow:**
1. Home the machine
2. Jog to the corner of your PCB
3. Click **Set Job Origin XY Here** to set your job origin
4. Click **Save Here** on Return Point 1
5. Run your job
6. Next session: Home → click **Go There** on Return Point 1 → job coordinates are automatically correct, no re-zeroing needed

**Return Point 1 (G28)** — e.g. your primary work origin or PCB corner
**Return Point 2 (G30)** — e.g. a tool change position or safe park

> Save Here shows a confirm dialog with your current machine position before writing to EEPROM.

---

### Tool Offset — Spindle to Laser

If your machine has both a spindle (router/mill) and a laser, they are physically offset from each other. This section measures and stores that offset so you can use the same job origin for both tools.

**The stored offset** is laser position minus spindle position. If positive X, the laser is to the right of the spindle.

**Move Laser to Spindle Zero:**
After setting your job origin with the spindle, click this button to move the laser beam to where the spindle was. The machine moves by the stored offset (`G90 G0 X{offsetX} Y{offsetY}`). Then set the job origin here as normal for laser work.

**Measuring the offset (one-time procedure):**

1. Click **▶ Measure / Update Offset (expand)** to open the measurement panel
2. **Step 1** — Position the spindle tip precisely over a reference mark (e.g., a punch mark or cross-hair on the workpiece). Click **Record Spindle Position**.
3. Switch to laser tool (change physically or via Active Tool selector)
4. **Step 2** — Align the laser dot precisely over the same reference mark (use Low Power laser for this). Click **Record Laser Position**.
5. The calculated ΔX/Y appears. Click **Add to Measurements** to add it to the list.
6. Repeat steps 1–5 a few times for accuracy.
7. When satisfied, click **Save Average as Offset** — this writes the average to Settings and saves it in the file.

> Multiple measurements average out positioning errors. Two or three measurements are usually sufficient.

---

### Active Tool Selector

Tells the controller which tool is active:
- **🔩 Spindle (T0)** — sends `M6 T0`
- **⚡ Laser (T1)** — sends `M6 T1`

The header badge at the top of the page shows SPINDLE or LASER at all times.

> ⚠ The laser will not fire (M3 will not work) until Laser (T1) is selected. Always switch here before attempting to fire the laser.

When switching back from Laser to Spindle, if the low-power laser is on it is automatically turned off.

---

### Laser Control

> ⚠ **Safety first.** Wear appropriate laser safety glasses (450nm OD4+). Ensure the beam path is clear. Never leave the machine unattended while the laser is firing.

The Laser Control panel is **disabled** (dimmed) until you select Laser (T1) in the Active Tool section above.

**Low Power (alignment) — toggle:**
- Set percentage (1–10%, capped for safety)
- Click **Fire Low Power** to toggle the beam on; click again to turn it off
- Use this for aligning the laser dot to your work origin
- The PWM value (e.g., `= S18`) is shown next to the percentage so you know exactly what's being sent

**High Power (marking) — hold to fire:**
- Set percentage (1–50%, max set in Settings as a safety cap)
- Click and **hold** the button — laser fires only while held
- Releasing the button, moving the mouse off it, or any touch-end event stops the laser immediately
- Use this for test marks or burn-in operations

**Emergency stop** (⚠ STOP button in header) sends `M5` to kill the laser in addition to the feed hold and soft reset.

---

## 5. Cylinder Probe

Find the XY center of external cylindrical features: bosses, round stock, pins, dowels.

**Setup:**
1. Position the probe tip approximately 2mm above the cylinder, roughly centered
2. Z height is your choice — the probe stays at this Z for the entire sequence
3. **Approximate Diameter** — enter your best estimate (±5mm is fine, just needs to be close enough for the probe to reach the wall)

**Running:**
1. Click **Test Probe** to verify the probe pin is working (machine touches pin momentarily)
2. Click **Run Probe** — the sequence probes East, West, North, South (two passes each direction)
3. Results show: Center X, Center Y, Diameter X, Diameter Y
4. Click **Set XY0** to move to center and set job origin there

**Tip diameter** is added to the measurement to account for the ball's radius.

**Presets** — save and load combinations of diameter and distance settings for parts you probe repeatedly.

**Repeatability test** — click the repeat button to run the probe multiple times and see statistical data (mean, standard deviation, runout). Export as CSV for quality records.

---

## 6. Hole Probe

Find the XY center of internal cylindrical features: holes, bores, pockets.

**Setup:**
1. Position the probe tip inside the hole, roughly centered
2. Z height stays fixed — position manually before starting
3. **Approximate Diameter** — enter the nominal hole diameter

**Running:**
Same as Cylinder Probe: Test Probe → Run Probe → Set XY0.

**Key difference from Cylinder:** The probe starts inside and moves outward to the walls. Tip diameter is **subtracted** (not added) because the probe tip is inside the feature.

---

## 7. Corner Probe

Find the XY corner of a rectangular workpiece or fixture.

**Important: Z does not move.** Position your probe at the desired height (mid-sidewall) before clicking Run Probe.

**Setup:**
1. Jog Z to the desired probe height (typically halfway up the workpiece sidewall)
2. **Corner Location** — select SW, SE, NW, or NE
   - This is from your perspective standing at the machine front (south side)
   - SW = bottom-left corner as you look at the machine
   - NE = top-right corner
3. **Start Distance (A)** — how far the probe tip is from each wall when you click Run (typically 5mm)
4. **Travel Distance** — how far to move perpendicular to set up for the second wall probe (must be ≥ Start Distance)

**Positioning before running:**
- For SW corner: position probe to the LEFT of and BELOW the corner (in the SW quadrant, outside the workpiece), at Start Distance from each wall

**Running:**
Click **Test Probe**, then **Run Probe**. The L-shaped sequence probes one wall, repositions, probes the second wall, and calculates the corner intersection.

**Results:** Corner X, Corner Y. Click **Set XY0** to set job origin at the corner.

---

## 8. Side Align

Verify a workpiece edge is parallel to a machine axis by measuring it at two points.

**Use case:** You've clamped a board and want to know if it's square to the machine before milling.

**Setup:**
1. **Probe Direction** — which way the probe moves to find the edge (Right/Left/Up/Down)
2. **Probe Distance** — how far from the edge to position the probe (typically 5mm)
3. Check "Raise Z before moving" if there are clamps or fixtures in the path

**Running:**

1. Jog to Point 1 (~5mm from the edge)
2. Click **Set Point 1 & Probe** — records position and performs double-probe
3. Jog to Point 2 (move only along the edge — do not change the probe axis)
4. Click **Set Point 2 & Probe** — probes the second location

**Results:**
- Point 1 and Point 2 measurements (distance from edge)
- **Difference** — how much the edge has moved between the two points
- **Correction** — e.g., "At Point 2 end, workpiece is 0.15mm FURTHER. Move Point 2 end CLOSER by 0.15mm"
- ✓ if within 0.01mm

**Iterating:**
1. Adjust the workpiece slightly based on the correction
2. Click **Re-Probe Point 2** to re-measure without moving Point 1
3. Repeat until ✓

Or click **Probe Both Points** to automatically sequence back to Point 1, probe, then move to Point 2 and probe.

---

## 9. Height Map

Map the Z height across a T-slot work table, typically to understand the flatness of the surface.

**Use case:** Your CNC table has T-slot rails at irregular X positions. This maps the height at each rail at multiple Y positions to show where to shim or how flat the surface is.

**Setup:**
1. Set Z=0 a few mm above the table surface at your desired starting corner (this is your clearance height during XY moves — do NOT set Z=0 at the table surface)
2. Click **Set Start Point** — this becomes XY 0,0 for the map
3. **Slot X Positions** — comma-separated X coordinates for each T-slot rail (mm from start), e.g.: `0,60,140.3,200.3,280.6,340.6`
4. **Y Positions** — comma-separated Y coordinates to probe at each slot, e.g.: `0,50,100,150,200,250,300,350,380`
5. **Probe Depth** — how far to probe down from Z=0 (e.g., 10mm if table surface is 8mm below)
6. **Double-touch** (default on) — each point does fast probe + retract + slow probe for accuracy

**Running:**
Click **Start Probing**. Progress shows slot and Y position counts. An **Abort** button stops safely mid-sequence.

**Exporting:**
- **Export CSV** — Raw data with X, Y, Z, deviation from reference. Open in Excel or LibreOffice for analysis.
- **Export HTML** — Full visual report with color heatmap (blue=low, white=reference, red=high) and raw data table.

---

## 10. Settings Tab

All configuration is in the Settings tab. Changes auto-save as you type.

### Connection
- **IP Address** — FluidNC controller IP
- **Port** — WebSocket port (default: 81)

### Probe Settings
| Setting | Description | Typical Value |
|---------|-------------|---------------|
| Tip Diameter | Probe ball diameter | 3.0 mm (3mm Ruby ball) |
| Fast Feedrate | First probe pass speed | 100 mm/min |
| Slow Feedrate | Second probe pass speed | 50 mm/min |
| Retract Distance | Pullback between passes | 2.0 mm |
| Timeout | Max time per probe move | 30 seconds |

### Jog Settings
- **XY Speed** — XY jog speed (mm/min), e.g., 1800
- **Z Speed** — Z jog speed (mm/min), e.g., 400

### Laser Settings
- **Low Power Default** — Default % for alignment beam (1–10, capped at 10 in UI)
- **High Power Max** — Safety cap on marking power (%). You cannot exceed this in the Laser Control panel.

### Stored Tool Offset
- **Offset X / Y** — The stored spindle-to-laser offset in mm. Set automatically by the Tool Offset measurement workflow. You can also edit manually here.

### Outputs
Set the M-code commands sent when toggling Mist, Vacuum, and IoT outputs. Common values:
- Mist On: `M7`, Mist Off: `M9`
- Vacuum On: `M8`, Vacuum Off: `M9`
- IoT: custom M-codes for your relay

### Default Probe Distances & Z Safe Heights
These auto-populate the distance and height fields when you switch probe tabs. Set them once and stop re-entering the same values every session.

### Settings JSON Editor
Direct access to the raw settings JSON. Click **Full Screen** for a larger editor. Press **Esc** to exit full screen. Click **Apply** to parse and apply changes. Click **Format** to pretty-print.

### Preset Summary
Shows how many saved presets exist for each probe type.

---

## 11. Coordinate System Concepts

### Machine Position (MCS)
The absolute truth. 0,0,0 is at your homing switches. After homing, this never changes regardless of what job origins you set. All machine moves are referenced here.

### Job Origin (G54)
An offset from machine position. When you "Set Job Origin XY Here," you are telling the controller: "right now, wherever I am, call this XYZ 0,0,0." Your G-code files use these coordinates. G54 is the default slot — almost all G-code files use it.

### Six WCS Slots (G54–G59)
You have six job origin slots. G54 is the default. You might use G55 for a second fixture, G56 for laser work if you prefer not to change G54, etc. Each stores independently in the controller's EEPROM.

### Return Points (G28, G30)
Two machine coordinate positions stored in EEPROM. Use them to return to the same spot after power cycling. Unlike job origins, they are in machine coordinates — they point to an absolute location, not a relative offset.

### The Golden Rule
**Home the machine first.** Every session, every time. Machine position is meaningless without a home cycle. Job origins, return points, and tool offsets all depend on accurate machine position.

---

## 12. Troubleshooting

### Cannot connect
- Verify the IP address in the header
- Verify FluidNC is powered on and connected to the network
- Try pinging the IP from your PC: `ping 192.168.x.x` in Command Prompt
- Verify no firewall is blocking port 81

### Position display not updating
- Click **↻ Refresh** to request a status report manually
- The display updates whenever the controller sends a status report — check the browser console (F12) for error messages

### Probe doesn't trigger
- Click **Test Probe** first — if this fails, the probe connection is the problem
- Check the probe wire and FluidNC probe input configuration
- Verify the probe circuit is closed (probe not triggered) before starting

### Probe triggered immediately (wrong direction of travel)
- The probe was already making contact before moving; retract it and reposition

### "Not connected" alert on probe operations
- The WebSocket connection dropped; click **Connect** again

### Alarm state after E-stop or limit switch
- Scroll down on the Control tab to the **Alarm** section
- Verify the machine is in a safe state (no crashes, limits not stuck)
- Click **Unlock / Clear Alarm ($X)**

### Job origin is wrong after returning to work
- Did you home the machine this session? Return Points and job origins are only valid after homing.
- Check the WCS Manager (Saved Job Origins) — click Refresh and verify G54 shows the expected values

### Laser not firing
- Is Laser (T1) selected in the Active Tool section? The Laser Control panel is disabled until you select it.
- Is `$32=1` set in your FluidNC configuration? Laser mode must be enabled in the controller.

### Settings disappeared
- The HTML file was closed without saving. Open the file and re-enter settings, then press Ctrl+S.
- If you opened the file from a browser download (without saving to disk first), the browser may not allow writing back to the file.

### I accidentally zeroed the wrong origin
- Use **↩ Undo Zero XY** (appears immediately after clicking "Set Job Origin XY Here") to restore the previous G54 offset.
- Or: go to the WCS Manager, click **Set Values** on G54, and enter the correct machine coordinates manually.
