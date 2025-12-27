# Changelog

All notable changes to the FluidNC Probe Utility will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.9.5] - 2024-12-27

### Fixed
- **CRITICAL: Corner Probe direction logic** - Completely rewrote probe movement logic
  - **Root Cause**: Overcomplicated logic with `probeDir` multiplier was moving in wrong directions
  - **Old behavior**: Moved AWAY from workpiece before probing, causing timeouts and wrong direction probes
  - **New behavior**: Simplified - just probe in the xDir/yDir direction (toward workpiece for outside corners)
  - Removed unnecessary "move to search start" logic
  - Both OUTSIDE and INSIDE corners now use same simple logic: probe in the direction indicated by xDir/yDir
  - **Impact**: Corner probe now actually works correctly for all 8 corner types

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
