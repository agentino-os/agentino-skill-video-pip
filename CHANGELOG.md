# Changelog

All notable changes to the `video-pip` Agentino skill are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and this skill adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] — 2026-04-23

First public release for the Agentino marketplace.

### Added

- `agentino skill exec video-pip` composes two videos as main + inset picture-in-picture:
  - Inset scaled as a fraction of the main video width (`scale`, range 0.05-0.8).
  - 9 position presets: top/middle/bottom × left/center/right.
  - Configurable `margin` from the nearest edge(s).
  - Optional coloured border (`border_width` > 0 + `border_color` named or `#RRGGBB`).
  - Audio routing: `main` | `inset` | `mix` (amix both).
  - Graceful handling when one (or both) inputs have no audio track.
  - Single `filter_complex` pass with `libx264` CRF 20 + AAC 192 kbps + `+faststart`.

### Tested

- 13.57 s main + 1.51 s inset at `bottom-right scale=0.3 border_width=4 border_color=#FFEB3B` → 13.56 s composed MP4, inset cleanly visible through full duration.
- Zero findings from `agentino skill exec skill-security-check -i path=skill.yaml` (fail-on = high).

### Requires

- Agentino ≥ 1.2.0-rc.1
- `ffmpeg` (and `ffprobe`) on `PATH`
