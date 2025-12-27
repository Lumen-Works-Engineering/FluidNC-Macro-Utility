# FluidNC Probe Utility

A web-based utility for CNC probe operations with FluidNC controllers. Consolidates multiple probing functions into a single, portable HTML file that connects via WebSocket.

![Version](https://img.shields.io/badge/version-1.9.2-blue)
![License](https://img.shields.io/badge/license-MIT-green)

## Features

- **Direct WebSocket Connection** to FluidNC controller
- **Dark Theme UI** - Easy on the eyes in the shop
- **Portable Settings** - Configuration embedded in HTML file, survives Save-As
- **Double-Probe Accuracy** - All probe operations use two-pass probing for precision
- **Preset Management** - Save and recall settings for common workpieces

### Tab Overview

| Tab | Status | Description |
|-----|--------|-------------|
| Control | ✅ Complete | Position display (MCS/WCS), jogging, homing, output toggles |
| Cylinder Probe | ✅ Complete | External cylinder/boss center finding |
| Hole Probe | ✅ Complete | Internal hole/bore center finding |
| Corner Probe | ✅ Complete | Inside/outside corner detection (8 quadrants) |
| Side Align | 🚧 Planned | Two-point alignment to axis |
| Height Map | 🚧 Planned | T-slot surface mapping |
| Settings | ✅ Complete | All configuration with JSON editor |

## Quick Start

1. Download `FluidNC_Probe_Utility.html`
2. Open in any modern browser (Chrome, Firefox, Edge)
3. Enter your FluidNC IP address (default: 192.168.73.13)
4. Click **Connect**
5. Use the tabs for different operations

## Requirements

- FluidNC controller with WebSocket enabled (port 81)
- Modern web browser
- Touch probe connected to FluidNC

## Usage

### Control Tab
- **Position Display**: Shows both Machine (MCS) and Work (WCS) coordinates
- **Zero Buttons**: Set current position as work zero (G10 L20)
- **Go 0 Buttons**: Rapid to work zero
- **Jog**: Click and hold arrow buttons for continuous jogging
- **Home**: Individual axis or Home All
- **Outputs**: Toggle Mist, Vacuum, IoT relay

### Cylinder Probe Tab
Use for finding the center of external cylindrical features (bosses, dowels, round stock).

1. Position probe tip approximately 2mm above the cylinder center
2. Set approximate cylinder diameter
3. Click **Test Probe** to verify probe connection
4. Click **Run Probe** to execute 4-point double-probe sequence
5. Use **Set XY0** to set work origin at center

### Hole Probe Tab
Use for finding the center of internal cylindrical features (holes, bores, pockets).

1. Position probe tip inside the hole, near center
2. Set approximate hole diameter
3. Click **Test Probe** to verify probe connection
4. Click **Run Probe** to execute 4-point double-probe sequence
5. Use **Set XY0** to set work origin at center

**Key Difference**: Cylinder probe moves outward then probes inward. Hole probe starts inside and probes outward to walls.

### Corner Probe Tab
Use for finding the XY corner of rectangular workpieces (both inside and outside corners).

1. Select **Corner Type** from dropdown (8 options covering all quadrants)
   - **Inside corners**: Position probe inside the corner (e.g., inside corner of a pocket)
   - **Outside corners**: Position probe outside the corner (e.g., corner of workpiece edge)
2. Set search distance (how far to move while searching for edges)
3. Set probe depth (how far below surface to probe)
4. Click **Test Probe** to verify probe connection
5. Click **Run Probe** to execute 2-edge double-probe sequence
6. Use **Set XY0** to set work origin at corner

**Corner Types**: NE (+X+Y), SE (+X-Y), NW (-X+Y), SW (-X-Y) for both inside and outside corners. The tool automatically accounts for probe tip diameter based on corner type.

### Saving Settings
Use browser **File → Save As** (Ctrl+S) to save the HTML file with your current settings embedded. Settings include:
- Connection parameters
- Probe feedrates and timeout
- Jog speeds
- Output commands (M-codes)
- All saved presets

## Technical Details

### Coordinate Systems
- FluidNC reports Machine Position (MPos) in status reports
- G54 offset queried via `$#` command
- WCS calculated as: `WCS = MCS - G54_offset`
- Probe results are in MCS; moves use `G53` for machine coordinates

### Probe Sequence
1. Set G91 (relative mode)
2. Move to first probe position
3. Fast probe (G38.2) to find surface
4. Retract slightly
5. Slow probe for precision measurement
6. Repeat for all 4 directions
7. Calculate center from averaged results
8. Move to center using G53 (machine coordinates)

### WebSocket Protocol
- Connects to `ws://<ip>:81/`
- Commands sent with newline terminator
- Handles both text and Blob message responses
- Sequential command/response with listener-before-send pattern

## Code Organization

The JavaScript is organized with VS Code-compatible region markers:

```
#region ===== GLOBAL STATE =====
#region ===== SETTINGS MANAGEMENT =====
#region ===== WEBSOCKET CONNECTION =====
#region ===== MESSAGE HANDLING =====
#region ===== CONTROL TAB =====
#region ===== SHARED PROBE UTILITIES =====
#region ===== CYLINDER PROBE =====
#region ===== HOLE PROBE =====
#region ===== INITIALIZATION =====
```

Use `Ctrl+Shift+[` to fold regions in VS Code.

## Version History

See [CHANGELOG.md](CHANGELOG.md) for detailed version history.

Current version: **1.9.2** (2024-12-27)

## Contributing

1. Fork the repository
2. Create a feature branch
3. Test thoroughly with actual hardware
4. Submit a pull request

**Important**: This software controls CNC machinery. Test all changes carefully to avoid crashes or damage.

## License

MIT License - See [LICENSE](LICENSE) file

## Author

John Sparks / Lumen Works Engineering

## Acknowledgments

- FluidNC project for the excellent CNC controller firmware
- gSender macros used as reference for probe sequences
