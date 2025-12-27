# Changelog

All notable changes to the FluidNC Probe Utility will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.5] - 2024-12-27

### Added
- Initial public release
- Control tab with position display, jogging, homing, output toggles
- Cylinder Probe tab with double-probe accuracy
- Settings tab with JSON editor
- Preset management for cylinder probes
- Repeatability test mode for probe verification
- Emergency stop button
- Browser tab shows version number

### Technical
- WebSocket connection to FluidNC
- G54 offset query via `$#` command for proper WCS calculation
- Relative probe moves (G91) to avoid soft limit issues
- 4-axis coordinate parsing (X, Y, Z, A)
- Blob message handling for WebSocket responses
- Embedded JSON settings survive browser Save-As

## [Unreleased]

### Planned
- Hole Probe tab (Phase 4)
- Corner Probe tab (Phase 5)
- Side Align tab (Phase 6)
- Height Map tab (Phase 7)

---

## Version Numbering

- **Major.Minor** format (e.g., 1.5, 1.6, 2.0)
- Major version bump for breaking changes or major feature additions
- Minor version bump for each development iteration
