# Changelog

All notable changes to the FluidNC Probe Utility will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
