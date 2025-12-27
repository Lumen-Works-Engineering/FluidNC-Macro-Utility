# FluidNC Probe Utility

A web-based utility for CNC probe operations with FluidNC controllers. Consolidates multiple probing functions into a single, portable HTML file that connects via WebSocket.

## Features

- **Direct WebSocket Connection** to FluidNC controller
- **Dark Theme UI** - Easy on the eyes in the shop
- **Portable Settings** - Configuration embedded in HTML file, survives Save-As
- **Double-Probe Accuracy** - All probe operations use two-pass probing for precision

### Implemented Tabs

| Tab | Status | Description |
|-----|--------|-------------|
| Control | ✅ Complete | Position display (MCS/WCS), jogging, homing, output toggles |
| Cylinder Probe | ✅ Complete | External cylinder center finding with presets |
| Hole Probe | 🚧 Planned | Internal hole center finding |
| Corner Probe | 🚧 Planned | XY corner detection |
| Side Align | 🚧 Planned | Two-point alignment to axis |
| Height Map | 🚧 Planned | T-slot surface mapping |
| Settings | ✅ Complete | All configuration with JSON editor |

## Quick Start

1. Download `FluidNC_Probe_Utility.html`
2. Open in any modern browser
3. Enter your FluidNC IP address (default: 192.168.73.13)
4. Click **Connect**
5. Use the tabs for different operations

## Requirements

- FluidNC controller with WebSocket enabled (port 81)
- Modern web browser (Chrome, Firefox, Edge)
- Touch probe connected to FluidNC

## Usage

### Control Tab
- **Position Display**: Shows both Machine (MCS) and Work (WCS) coordinates
- **Zero Buttons**: Set current position as work zero
- **Go 0 Buttons**: Rapid to work zero
- **Jog**: Click and hold arrow buttons for continuous jogging
- **Home**: Individual axis or Home All

### Cylinder Probe Tab
1. Position probe above the approximate center of the cylinder
2. Set cylinder diameter and probe parameters
3. Click **Test Probe** to verify probe connection
4. Click **Run Probe** to execute the 4-point probe sequence
5. Use **Set XY0** to set work origin at cylinder center

### Saving Settings
Use browser **File → Save As** (Ctrl+S) to save the HTML file with your current settings embedded. Settings include:
- Connection parameters
- Probe feedrates and timeout
- Jog speeds
- Output commands
- Saved presets

## Technical Details

### Coordinate System
- FluidNC reports Machine Position (MPos) in status reports
- G54 offset queried via `$#` command
- WCS calculated as: `WCS = MCS - G54_offset`

### Probe Commands
- Uses relative moves (G91) for safety
- Double-probe sequence: fast approach + slow precision pass
- Probe results reported in Machine coordinates via `[PRB:x,y,z:1]`

### WebSocket Protocol
- Connects to `ws://<ip>:81/`
- Sends G-code commands with newline terminator
- Handles both text and Blob message responses

## Version History

See [CHANGELOG.md](CHANGELOG.md) for detailed version history.

## License

MIT License - See LICENSE file

## Author

John Sparks / Lumen Works Engineering
