# Changelog

All notable changes to the FluidNC Probe Utility will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.9.1] - 2024-12-27

### Fixed
- **CRITICAL: Hole Probe movement bug** - Fixed dangerous over-travel when moving from East to West probe
  - Was moving `-(probeDistance - retract)` then `-(probeDistance)` which caused double movement
  - Now correctly moves `-(probeDistance * 2)` in single move to opposite side
  - Fixed for both X (East/West) and Y (North/South) axes
  - **Impact**: Prevented potential crashes into workpiece/fixture
- Hole probe now correctly retracts and probes inward from opposite side

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

- **Major.Minor** format (e.g., 1.5, 1.6, 2.0)
- Major version bump for breaking changes or major feature additions
- Minor version bump for each development iteration
