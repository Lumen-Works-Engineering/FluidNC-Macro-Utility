# Changelog

All notable changes to the FluidNC Probe Utility will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.13.4] - 2024-12-28

### Fixed
- **Height Map exports now use correct coordinates and timestamps**
  - **Problem**: Stored actual WCS probe coordinates instead of target grid positions
  - **Impact**: Heatmap showed all "N/A" because coordinates didn't match (0.01mm tolerance failed)
  - **Example**: User set start at Y+40mm from machine zero, probed at Y=40,90,140... but configured Y=0,50,100...
  - **Fix**: Store TARGET coordinates (x, y from grid) with ACTUAL probe Z value
  - **Result**: Heatmap now correctly maps deviations to grid positions
- **Timestamp format now human-readable**
  - **Old**: `heightmap_1766968597151.csv` (milliseconds since epoch)
  - **New**: `heightmap_2024-12-28T183045.csv` (ISO 8601 format without colons)
  - Applied to both CSV and HTML exports
- **Workflow clarification**: User sets X0 Y0 Z0 at their chosen start point (not machine zero)
  - Start point can be anywhere (e.g., Y+40mm from machine zero to avoid table edges)
  - Probe moves to relative grid positions from that start point
  - Exports show grid coordinates (0, 50, 100...) not absolute WCS coordinates

### Technical
- Changed probe data storage from `{ x: px, y: py, z: pz }` to `{ x: x, y: y, z: pz }`
  - `x, y` = target grid position (HeightMap.slotX[i], HeightMap.yPos[j])
  - `pz` = actual Z from probe response `[PRB:...]`
- Timestamp generation: `new Date().toISOString().replace(/:/g, '').split('.')[0]`
- Heatmap lookup now succeeds with 0.01mm tolerance since coordinates match exactly

## [1.13.3] - 2024-12-28

### Fixed
- **CRITICAL: Height Map probe response not captured** - Fixed `sendAndWait()` returning undefined
  - **Problem**: `sendAndWait()` waited for "ok" but didn't return the probe response
  - **Error**: "Cannot read properties of undefined (reading 'match')" when parsing PRB response
  - **Root cause**: Line 1483 `resolve()` returned undefined instead of the accumulated responses
  - **Impact**: Height Map failed immediately on first probe touch
  - **Fix**: Changed `sendAndWait()` to accumulate all responses and return them on "ok"
  - **Technical**: Added `let responses = ''` and `responses += d` to build complete response string
  - **Sequence**: Receives `[PRB:22.198,51.298,-52.745,0.000:1]` then `ok` → returns full string for parsing

## [1.13.2] - 2024-12-28

### Changed - Height Map Simplified Workflow
- **Removed Z Safe Height setting** - Not needed for Height Map feature
  - **Workflow**: User sets Z=0 a few mm above table surface (safe clearance)
  - **Probing**: All moves happen at Z=0, only probe command moves Z down
  - **Rationale**: No need to lift Z between points for table-level probing
- **Updated probing sequence** - Stay at Z=0 throughout entire operation
  - Move XY to next position (staying at Z=0)
  - Switch to relative mode (G91)
  - Probe down (e.g., Z-10mm from current position)
  - Switch back to absolute mode (G90)
  - Retract to Z=0
  - Repeat for all points
  - **Old sequence**: Lifted to Z=20mm between every point (unnecessary)
  - **New sequence**: Stay at Z=0, only move Z during probe itself
- **Removed Z Safe Height from presets** - No longer saved or loaded
- **Updated probe depth help text** - "How far to probe down from Z=0"

### Technical
- Removed `zSafe` variable and validation from `startHeightmapProbe()`
- Removed `hm-z-safe` input field from Height Map tab
- Removed `zSafe` property from preset save/load functions
- Removed initialization code that set Z safe from default settings
- Simplified move logic - no conditional lifting between slots

## [1.13.1] - 2024-12-28

### Fixed
- **CRITICAL: Height Map probe moving up instead of down** - Fixed G-code mode issue
  - **Problem**: Probe command sent in absolute mode (G90) with negative Z value, causing upward movement
  - **Impact**: Triggered soft limit alarm instead of probing table surface
  - **Root cause**: After moving to Z safe height (e.g., Z=10mm absolute), `G38.2 Z-10` tried to move to Z=-10mm absolute
  - **Fix**: Switch to relative mode (G91) for probe command, then back to absolute (G90)
  - **Sequence**: G0 Z10 (safe) → G0 X0 Y0 → **G91** → G38.2 Z-10 (probe down 10mm) → **G90** → G0 Z10 (retract)
  - **Technical**: Matches probe pattern used in Side Probe and other probe functions
- **Height Map: Added missing `createProbeHelpers()` calls**
  - Both `setHeightmapStartPoint()` and `startHeightmapProbe()` now properly create probe helpers
  - Fixes "sendAndWait is not defined" error

## [1.13.0] - 2024-12-28

### Added - Height Map Feature (Phase 7)
- **Height Map tab for T-slot probing** - Complete implementation for irregular grid probing
  - **Purpose**: Create height maps of CNC work tables with T-slot rails at unequal distances
  - **Use case**: User's modified Genmitsu Hybrid 4040 Pro Max with custom T-slot configuration
- **Preset management for table configurations**
  - Save/load/delete named presets for different table setups
  - Stores: slot X positions, Y positions, Z safe height, probe depth, feedrate, timeout
  - Prevents re-entering complex slot position arrays
- **T-slot configuration**
  - Slot X positions (comma-separated, irregular): e.g., `0,60,140.3,200.3,280.6,340.6`
  - Y positions (comma-separated): e.g., `0,50,100,150,200,250,300,350,380`
  - Supports any number of slots and Y positions
- **Set Start Point button** - Sets current position as WCS origin (G10 L20 P1 X0 Y0)
  - User manually jogs to SW t-slot rail first point before clicking
  - Simplifies workflow for consistent reference point
- **Probing sequence with progress tracking**
  - Sequential probing: for each X slot, probe all Y positions
  - Progress display: "Probing slot 2/6, Y position 5/9 (45.5% complete)"
  - Abort button to safely stop mid-sequence
  - Watchdog timeout per command for safety
- **Results display**
  - Summary statistics: reference Z, min/max/range/average
  - Point count and configuration echo
  - **No inline heatmap** (would be too large for UI)
- **CSV export** - Raw data export
  - Columns: X, Y, Z, Success, Deviation (from reference)
  - Timestamped filename: `heightmap_1703779234567.csv`
  - For analysis in external tools or comparison between runs
- **HTML report export** - Complete visualization
  - Summary statistics table
  - Configuration details (slot positions)
  - **Heatmap table** with color coding (blue=low, white=reference, red=high)
  - Raw data table with deviations
  - Timestamped filename: `heightmap_report_1703779234567.html`
  - For visual comparison after table adjustments

### Added - Settings Tab Enhancements
- **Default Probe Distances section** - Auto-populate probe distance fields
  - Cylinder Probe: 20mm default
  - Hole Probe: 20mm default
  - Corner Probe: 20mm default
  - Side Probe: 5mm default
  - Saves time when switching tabs
- **Default Z Safe Heights section** - Auto-populate Z clearance heights
  - Cylinder/Hole/Corner: 10mm default
  - Side Probe: 10mm default
  - Height Map: 20mm default (larger for table-wide moves)
  - Prevents forgetting to set safe heights
- **Heightmap preset count** - Shows number of saved table configurations
  - Added to Presets Summary section
  - Example: "3 heightmap presets saved"

### Technical
- Added `defaults` object to embedded settings JSON
  - `probeDistances`: { cylinder, hole, corner, side }
  - `zSafeHeights`: { standard, side, heightmap }
- Added `heightmap` array to `presets` object
- Updated `applySettingsToUI()` to populate default value fields
- Updated `collectSettingsFromUI()` to save default values
- Updated `updatePresetCounts()` to include heightmap count
- Height Map uses shared `App.probeInProgress` flag to prevent concurrent probing
- Height Map state object tracks: probeData, slotX, yPos, currentSlot, currentY, totalPoints, completedPoints
- Probe data format: `{x, y, z, success}` array
- Heatmap color algorithm: normalized deviation → blue/white/red gradient
- Parse CSV utility: `parseCSV(str)` converts comma-separated strings to float arrays

### User Experience
- **Height Map workflow**: Position → Set Start Point → Configure slots → Start Probing → Export
- **Preset workflow**: Configure once → Save as preset → Load next time
- **Default values**: Set once in Settings → Auto-populate on all tabs
- **Data freshness**: Timestamp on exports for version tracking
- **Safety**: Abort button, watchdog timeout, Z raise between slots

## [1.12.0] - 2024-12-28

### Fixed
- **Side Probe: Z raise symmetry in "Probe Both Points"** - Now raises Z when moving to Point 1
  - **Problem**: Raised Z moving Point 1→Point 2, but NOT when moving back to Point 1 on second run
  - **Impact**: Could crash into clamps/fixtures when re-running "Probe Both Points"
  - **Fix**: Added Z raise logic before moving to Point 1 (same as Point 2)
  - **Behavior**: If "Raise Z" enabled, BOTH moves (to Point 1 and to Point 2) raise Z for safety

### Added
- **Side Probe: Timestamp on results** - Shows when last probed
  - Format: "Side Probe Results - 2:45:32 PM"
  - Helps verify results are fresh after re-probing
- **Side Probe: Last probe difference tracking** - Shows if adjustment made it better or worse
  - Example: "Last Probe Difference: 0.150 mm (✓ BETTER by 0.120mm)"
  - Shows: "✓ BETTER", "⚠ WORSE", or "= SAME" compared to previous probe
  - Cleared when clicking "Clear All Points"
  - **Impact**: Immediate feedback on whether adjustment worked
- **Side Probe: Point 1 as reference in results** - Clarifies Point 1 should not move
  - Labels: "Point 1 (Reference)" and "Point 2 (Adjust)"
  - Instructions always frame as "Adjust Point 2 to match Point 1"
  - Note at bottom: "(Point 1 = reference, do not move if possible)"

### Changed
- **Side Probe: Jog controls moved to bottom** - Results now visible without scrolling
  - **Old**: Settings → Jog → Point Status → Results (results below fold)
  - **New**: Settings → Point Status → Results → Jog (results always visible)
  - **Impact**: No scrolling needed to see alignment results
- **Side Probe: Button state workflow guidance** - Visual highlighting shows next action
  - **Fresh start**: "Set Point 1 & Probe" highlighted (blue), others disabled/grayed
  - **After Point 1**: "Set Point 2 & Probe" highlighted, Point 1 de-emphasized, Clear enabled
  - **After both points**: "Re-Probe Point 2" and "Probe Both Points" highlighted
  - **Impact**: Always clear what the next step is in the workflow

## [1.11.6] - 2024-12-28

### Fixed
- **CRITICAL: Side Probe not returning to original position** - Fixed WCS corruption
  - **Problem**: After double-probe sequence, probe only retracted 2mm instead of returning to start
  - **Impact**: WCS got corrupted, Point 2 measurements were inaccurate
  - **Sequence**: Fast probe → retract 2mm → slow probe → retract 2mm ❌ (ends 1mm forward from start)
  - **Fixed**: Fast probe → retract → slow probe → **retract to original position** ✓
  - **Technical**: Changed from hardcoded `2mm` to using `retract` setting and proper return distance
  - **Benefit**: Probe returns to exact starting position, preserving WCS and enabling accurate multi-point measurements

### Changed
- **Side Probe: Practical alignment instructions** - Removed degrees, added actionable guidance
  - **Old**: "Rotate clockwise by 0.034° to align" (not useful - can't measure degrees in shop)
  - **New**: "At Point 2 (far end), workpiece is 0.15mm FURTHER from edge. Move Point 2 end CLOSER to edge by 0.15mm."
  - **Shows**: Which end (Point 1/Point 2), which direction (CLOSER/FURTHER), exact distance to move
  - **Impact**: Clear, actionable instructions for workshop alignment

## [1.11.5] - 2024-12-28

### Fixed
- **Side Probe: Undefined variable in probe sequence** - Fixed "axis is not defined" error
  - **Error**: "ReferenceError: axis is not defined" at line 2212 when probing Point 2
  - **Root Cause**: In v1.11.4, renamed `axis` to `probeAxis` in sanity check, but forgot to define `axis` for probe sequence
  - **Fix**: Added `const axis = probeAxis;` before probe sequence to restore variable
  - **Impact**: Point 2 probing now works without crashing

## [1.11.4] - 2024-12-28

### Fixed
- **CRITICAL: Side Probe axis logic inverted** - Fixed sanity check comparing wrong axes
  - **Problem**: Utility rejected Point 2 saying "X must be 5mm apart" when user correctly moved 141mm in Y
  - **Root Cause**: Logic checked that probe axis (X) changed, but should check travel axis (Y) changed
  - **Example**: Probing +X (right), user moves along Y from 86→227mm (correct), but X only changed 0.05mm
  - **Old logic**: "X axis must change by 5mm" ❌ (wrong - X is probe direction, should stay constant)
  - **New logic**: "Y axis must change by 5mm" ✓ (correct - Y is travel direction)
  - **Fix**: Swapped axis/otherAxis to probeAxis/travelAxis with clearer variable names
  - **Impact**: Now correctly validates that user moved along perpendicular axis

### Changed
- **Side Probe: Jog buttons now continuous hold-to-jog** - Matches Control tab behavior
  - **Old**: Click buttons with step size input (not practical for long parts)
  - **New**: Arrow buttons (▲▼◀▶) with click-and-hold continuous jogging
  - **Removed**: Step size input and `jogAxis()` function (now uses shared jog event listeners)
  - **Impact**: Can jog full length of workpiece by holding arrow button

## [1.11.3] - 2024-12-28

### Fixed
- **Side Probe: Jog buttons not working** - Added missing `jogAxis()` function
  - **Error**: "ReferenceError: jogAxis is not defined" when clicking jog buttons
  - **Fix**: Created `jogAxis(axis, distance)` helper function for Side Probe tab
  - **Impact**: XY jog buttons on Side Probe tab now work correctly
- **Side Probe: Point 2 position not updating** - Fixed stale position reading
  - **Problem**: When jogging to Point 2, utility read cached position from Point 1
  - **Root Cause**: Used `sendCommand('?')` which doesn't wait for response
  - **Fix**: Changed to `sendAndWait('?')` with 200ms delay for position update
  - **Example**: User jogged from Y=86mm to Y=227mm, but utility still saw Y=86mm
  - **Impact**: Point 2 now correctly reads current position after manual jogging

## [1.11.2] - 2024-12-28

### Fixed
- **Side Probe: Position reading error** - Fixed "Cannot read properties of undefined" error
  - **Root Cause**: Used `App.currentPos` instead of correct `App.position.mcs`
  - **Fixed in**: `setSidePoint1AndProbe()` and `setSidePoint2AndProbe()` functions
  - **Impact**: Side Probe now correctly reads current machine position before probing

## [1.11.1] - 2024-12-28

### Added
- **Side Probe: Z Safe Height option** - Safety feature for "Probe Both Points" auto-sequence
  - **New Checkbox**: "Raise Z before moving between points"
  - **New Setting**: "Z Safe Distance" (default 10mm, relative movement)
  - **Purpose**: Prevents probe from crashing into clamps, fixtures, or workpiece between points
  - **Behavior**:
    - When checkbox enabled: Raise Z → Move XY to next point → Lower Z
    - When checkbox disabled: Direct XYZ move (faster, but requires clear path)
  - **Only affects auto-sequence**: Manual jogging workflow unchanged
  - **Z movement is relative** (G91): Raises/lowers by specified amount from current position
  - **Confirmation dialog**: Shows "Raise Z: Yes (10mm)" or "Raise Z: No" before running

## [1.11.0] - 2024-12-28

### Added
- **NEW FEATURE: Side Probe (Two-Point Alignment)** - Check if workpiece is parallel to axis
  - **Purpose**: Measure workpiece edge at two points to verify alignment
  - **UI Settings**:
    - Probe Direction: Right (+X), Left (-X), Up (+Y), Down (-Y)
    - Probe Distance: Distance from edge to start position (default 5mm)
    - Feedrate: Fast probe speed (default 100mm/min)
    - Slow Feedrate: Precision probe speed (default 25mm/min)
  - **Jog Controls**: Compact XY jog buttons on Side Probe tab for convenience
  - **Workflow**:
    1. Manually jog to Point 1 (~5mm from edge)
    2. Click "Set Point 1 & Probe" - records position and probes
    3. Manually jog to Point 2 (moving only one axis)
    4. Click "Set Point 2 & Probe" - records position and probes
    5. View results with difference, angle, and correction instructions
  - **Additional Features**:
    - "Re-Probe Point 2" button for adjustment verification
    - "Probe Both Points" auto-sequence using saved positions (G53 moves)
    - Sanity check: warns if Point 2 moved wrong axis or too close to Point 1
    - Results show: measurements, difference, travel distance, angle, correction direction
    - Tolerance check: ✓ if within 0.01mm, ⚠ with rotation/adjustment instructions if not
  - **Point Status Display**: Shows which points are set with coordinates
  - **State Management**: Clear button resets all points for fresh start

## [1.10.3] - 2024-12-27

### Fixed
- **Corner Probe Z behavior** - Removed dangerous Z-down move after moving to corner
  - **Old behavior**: Raised Z, moved to corner, then lowered Z back to probe height
  - **Problem**: Z probe height is at mid-sidewall - lowering would crash into workpiece top
  - **New behavior**: Raises Z to safe height, moves to corner, STOPS at safe height
  - **Workflow**: User removes probe → inserts tool → manually sets Z zero at work surface
  - **If "Move to Corner" unchecked**: Probe stays at position (C), no Z movement
  - **Impact**: Proper tool change workflow, no risk of crashing into workpiece

## [1.10.2] - 2024-12-27

### Fixed
- **CRITICAL: Z Safe Distance G-code mode bug** - Fixed soft limit errors when moving to corner
  - **Root Cause**: After moving to corner XY in G90 (absolute), the Z-down move was also absolute
  - **Old behavior**: `G0 Z-20` (absolute) tried to move to Z=-20mm, causing soft limit errors
  - **New behavior**: Explicitly switch to G91 before Z-down move, making it relative
  - **Z Safe Distance is now RELATIVE**: Raises Z by specified amount from current position
  - **Industry Standard**: Matches touch probe behavior (relative retract distance)
  - **Updated UI label**: Now clearly states "Relative: raise Z this amount from current position"
  - **Impact**: No more soft limit errors, works regardless of where you position Z initially

## [1.10.1] - 2024-12-27

### Fixed
- **Corner Probe Step 8 direction** - Fixed movement to position (B)
  - Was moving in probe direction (toward wall)
  - Now correctly moves AWAY from second wall before probing
  - Example: SW corner now moves -Y (south) before probing +Y (north) to find bottom wall

## [1.10.0] - 2024-12-27

### Changed
- **MAJOR: Complete Corner Probe rewrite** - Implemented per user specification document
  - **New UI Settings**:
    - Setting 1: Corner Location dropdown (SW, SE, NW, NE) - user's perspective
    - Setting 2: Corner Type dropdown (Outside/Inside) - inside disabled for now
    - Setting 3: Start Distance (A) - default 5mm from each wall
    - Setting 4: Travel Distance (2→B) - default 10mm (auto 2× start distance)
    - Setting 5: Move to Corner checkbox - optional Z-safe move at end
    - Setting 6: Z Safe Distance - height to raise before moving to corner
  - **New Probe Sequence** (matches PDF exactly):
    - Step 7: Probe first wall from position (A)
    - Step 8: Move to position (B) - perpendicular to first wall
    - Step 9: Move to position (C) - past first wall by startDist
    - Step 10: Probe second wall
    - Step 11: Optionally raise Z and move to corner (D)
    - Step 12: Display results with Set X/Y Zero buttons
  - **Removed**: Old confusing ±X±Y notation and complicated direction logic
  - **Added**: Sanity check (travel distance ≥ start distance)
  - **Updated**: Setup instructions to match new L-shaped probe pattern
  - **Updated**: Presets to save all new settings
  - **Impact**: Corner probe now follows industry-standard workflow with clear positioning

## [1.9.5] - 2024-12-27

### Fixed
- **CRITICAL: Corner Probe direction logic** - Attempted fix (superseded by v1.10.0 complete rewrite)

## [1.9.4] - 2024-12-27

### Fixed
- **Corner Probe improvements** - Major usability fixes based on user testing
  - **Removed all Z-axis movement** - User manually positions Z height before probing (matches Hole Probe behavior)
  - **Clearer dropdown labels** - Reorganized with "OUTSIDE Corner" and "INSIDE Corner" groups
  - Bottom-Left is now first option (most common use case)
  - Labels now indicate probe direction: "Bottom-Left (probe from lower-left, move +X +Y)"
  - **Added Setup Instructions panel** - Clear guidance for each corner type with positioning details
  - **Important note** highlighted: Z will NOT move during probe sequence
  - Removed "Probe Depth" UI field (no longer used)
  - Updated presets to remove probeDepth parameter
  - **Impact**: Much clearer user experience, no unexpected Z movements that could crash probe

### Changed
- Corner probe default selection changed to "Bottom-Left" (most common scenario)
- Corner probe error handling no longer includes Z retraction

## [1.9.3] - 2024-12-27

### Fixed
- **CRITICAL: Hole Probe retract bug** - Removed dangerous retract move after West wall probe
  - After probing West wall, was retracting WEST (`X-2.000`) instead of staying in place
  - This pushed probe 2mm further into the wall, risking collision with rigid workpieces
  - **Fix**: Removed unnecessary retract before moving to center (absolute positioning handles it)
  - West wall sequence now: probe → calculate → move directly to X center
  - **Impact**: Eliminated collision risk that could break expensive probe

## [1.9.2] - 2024-12-27

### Fixed
- **CRITICAL: Hole Probe logic errors** - Fixed multiple dangerous bugs in hole probing sequence
  - **Bug 1 - Over-travel**: Was using `probeDistance` (search parameter) instead of actual measured hole geometry
    - Now uses absolute positioning: `G53 G0 X{eastX - probeDistance}` to safely move to opposite side
  - **Bug 2 - Wrong probe direction**: After moving to West/South side, was probing in wrong direction
    - West wall: Changed from `+X` to `-X` (probe outward/West, not back toward East)
    - South wall: Changed from `+Y` to `-Y` (probe outward/South, not back toward North)
    - Also fixed retract direction: `+X` after West probe, `+Y` after South probe
  - **Cleanup**: Removed unnecessary Z-axis movements (hole probe stays at one Z level)
  - Fixed for both X-axis (East/West) and Y-axis (North/South) movements
  - **Impact**: Prevented crashes into workpiece/fixture and expensive probe damage

## [1.9.1] - 2024-12-27

### Changed
- Version skipped - incremented to 1.9.2 for micro-versioning clarity

## [1.9] - 2024-12-27

### Added
- **Corner Probe tab (Phase 5)** - Find XY corner of rectangular workpieces
  - Support for both **inside** and **outside** corners
  - 8 corner type options covering all quadrants (NE, SE, NW, SW)
  - Double-probe accuracy for both X and Y edges
  - Automatically accounts for probe tip diameter in corner calculation
  - Preset management for common corner configurations
  - Set X0/Y0/XY0 buttons to set work origin at corner
  - Instructions panel explaining inside vs outside corner probing
- **Corner presets** added to settings structure
- Corner tip diameter display synced with global probe settings

### Technical
- Corner probe logic handles direction multipliers for all 8 corner types
- Inside corners: probe outward from center, add tip radius
- Outside corners: probe inward from edges, subtract tip radius
- Uses shared probe utilities (`createProbeHelpers`, `testProbeConnection`)

## [1.8] - 2024-12-27

### Added
- **Hole Probe tab (Phase 4)** - Find center of holes/bores with double-probe accuracy
  - Probes outward from inside the hole to find walls
  - Calculates hole diameter (subtracts probe tip diameter for internal measurement)
  - Preset management for common hole sizes
  - Repeatability test mode for verification
  - Set X0/Y0/XY0 buttons after probing
- **Shared probe utilities** - Common code extracted for reuse across all probe types
  - `createProbeHelpers()` - Creates `sendAndWait`, `probeAndCapture`, `moveRel` functions
  - `calculateProbeResults()` - For external features (adds tip diameter)
  - `calculateHoleResults()` - For internal features (subtracts tip diameter)
  - `testProbeConnection()` - Shared probe pin test for all tabs
- **VS Code region markers** - `#region` / `#endregion` comments for code folding
- **Shared probe flag** - `App.probeInProgress` prevents concurrent probe operations

### Changed
- Refactored code organization with clear labeled sections
- Probe command ID counter now shared and reset per operation

## [1.7] - 2024-12-27

### Fixed
- **Critical: WCS/MCS coordinate mismatch** - Probe results (`[PRB:...]`) are reported in Machine Coordinates (MCS), but absolute moves were being interpreted as Work Coordinates (WCS). Now uses `G53` prefix for absolute positioning moves to force machine coordinate interpretation.

## [1.6] - 2024-12-27

### Fixed
- **Race condition in command sequencing** - Event listeners now set up BEFORE sending commands to prevent responses from being missed or matched to wrong commands
- Added sequential command IDs for debug logging (`[1]`, `[2]`, etc.)

### Changed
- Refactored probe helpers: `sendAndWait()`, `probeAndCapture()`, `moveRel()`
- Removed redundant G91 prefixes (mode set once at sequence start)
- Cleaner console output with PING message filtering

## [1.5] - 2024-12-27

### Added
- Initial public release
- **Control tab** - Position display (MCS/WCS), click-and-hold jogging, homing, output toggles
- **Cylinder Probe tab** - External cylinder center finding with double-probe accuracy
  - Preset management for common cylinder sizes
  - Repeatability test mode
  - Set X0/Y0/XY0 buttons
- **Settings tab** - All configuration with live JSON editor
- Emergency stop button
- Browser tab shows version number

### Technical
- WebSocket connection to FluidNC (port 81)
- G54 offset query via `$#` command for proper WCS calculation
- Relative probe moves (G91) to avoid soft limit issues
- 4-axis coordinate parsing (X, Y, Z, A)
- Blob message handling for WebSocket responses
- Embedded JSON settings survive browser Save-As operations

## [Unreleased]

### Planned
- Corner Probe tab (Phase 5) - Find XY corner of rectangular workpieces
- Side Align tab (Phase 6) - Two-point alignment to axis
- Height Map tab (Phase 7) - T-slot surface mapping

---

## Version Numbering

- **Major.Minor.Patch** format (e.g., 1.9.2, 1.10.0, 2.0.0)
- **Major**: Breaking changes or major feature additions
- **Minor**: New features (backward compatible)
- **Patch**: Bug fixes and micro-iterations during development/testing
