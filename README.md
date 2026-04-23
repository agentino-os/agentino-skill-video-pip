# video-pip

> **Overlay a second video as a picture-in-picture inset on a main video — classic tutorial webcam overlay, reaction video, multi-cam composition. Configurable position, scale, border, and audio routing.**

An Agentino skill that takes two video files and produces a single MP4 with one layered on top of the other. The inset is scaled as a fraction of the main video's width (the most natural way to specify "small webcam in the corner"), placed at one of 9 position presets, optionally bordered, and the audio is routed from main, inset, or a mix of both.

Typical use cases: tutorials (screencast main + webcam inset), reaction videos (content main + reactor inset), multi-cam interviews (wide main + tight inset).

## Install

Requires [Agentino](https://github.com/dagoSte/agentino) ≥ `1.2.0-rc.1` and `ffmpeg`.

```bash
brew install ffmpeg                                                    # macOS
# sudo apt install ffmpeg                                              # Debian / Ubuntu

agentino marketplace install dagoSte/agentino-skill-video-pip
agentino skill show video-pip
```

## Use

### Classic tutorial overlay

```bash
agentino skill exec video-pip \
  -i main_video=/tmp/screencast.mp4 \
  -i inset_video=/tmp/webcam.mp4 \
  -i output_path=/tmp/tutorial.mp4 \
  -i position=bottom-right \
  -i scale=0.25 \
  -i audio_source=main
```

### With a branded border

```bash
agentino skill exec video-pip --input-json '{
  "main_video": "/tmp/screencast.mp4",
  "inset_video": "/tmp/webcam.mp4",
  "output_path": "/tmp/branded.mp4",
  "position": "bottom-right",
  "scale": 0.3,
  "margin": 50,
  "border_width": 4,
  "border_color": "#FFEB3B",
  "audio_source": "mix"
}'
```

### Example output

```json
{
  "output_path": "/tmp/tutorial.mp4",
  "main_duration_s": 613.2,
  "inset_duration_s": 610.8,
  "output_duration_s": 613.2,
  "position": "bottom-right",
  "scale": 0.25,
  "audio_source": "main",
  "border_width": 0
}
```

## Use with agentino run

```bash
agentino run "put /tmp/webcam.mp4 as a bottom-right picture-in-picture on /tmp/screencast.mp4 at 25% size, save to /tmp/tutorial.mp4"
```

The LLM planner recognises "picture-in-picture" + a size hint + a position as the trigger for this skill.

## Inputs

| Input | Type | Default | Description |
|---|---|---|---|
| `main_video` | string | **required** | Background layer. |
| `inset_video` | string | **required** | Picture-in-picture layer. |
| `output_path` | string | `<main>.pip.<ext>` | Destination MP4. |
| `position` | string | `bottom-right` | Inset placement: top/middle/bottom × left/center/right (9 presets). |
| `scale` | number | `0.25` | Inset width as fraction of main width (range 0.05-0.8). |
| `margin` | integer | `40` | Distance in pixels from the nearest edge(s). |
| `audio_source` | string | `main` | `main` \| `inset` \| `mix`. |
| `border_width` | integer | `0` | Border thickness in pixels. `0` disables. |
| `border_color` | string | `white` | Border colour — named or `#RRGGBB`. |

## Outputs

- **`output_path`** — absolute path to the composed MP4.
- **`main_duration_s`** / **`inset_duration_s`** / **`output_duration_s`** — durations; output matches the main video length (inset either loops implicitly or stops early — `overlay=shortest=1` anchors to main).
- **`position`**, **`scale`**, **`audio_source`**, **`border_width`** — resolved settings.

## How it works

1. **Probe** main + inset for durations; detect audio presence on each.
2. **Scale the inset** — `[1:v]scale=iw*<scale>:ih*<scale>:force_original_aspect_ratio=decrease`.
3. **Pad for border** (if `border_width > 0`) — `pad=iw+2bw:ih+2bw:bw:bw:color=<border_color>`.
4. **Overlay** — `[0:v][pip]overlay=x=<expr>:y=<expr>:shortest=1` with x/y expressions derived from the chosen position preset and `margin`.
5. **Audio routing** — `main` → `-map 0:a`, `inset` → `-map 1:a`, `mix` → `[0:a][1:a]amix=inputs=2:duration=shortest[aout]`. Missing audio tracks handled gracefully.
6. **Render** — `libx264` CRF 20 + AAC 192 kbps + `+faststart`.

## Safety

- `sandbox_level: none` (ffmpeg discovery).
- `network_access: false`.
- `file_access: read-write`.
- `agentino skill exec skill-security-check -i path=skill.yaml` → zero findings.

## License

MIT — see [`LICENSE`](./LICENSE).
