---
name: transcode
description: Convert media files between formats, codecs, or resolutions
argument-hint: <input> <output> [--resolution <res>] [--codec <codec>]
allowed-tools: Bash
---

# Transcode Media

Convert media files with optional resolution and codec changes.

## Arguments

- `<input>` - Required. Source file path
- `<output>` - Required. Destination file path
- `--resolution <res>` - Optional. Target resolution (720p, 1080p, 4k, or WxH)
- `--codec <codec>` - Optional. Video codec (h264, h265, copy)

## Instructions

1. Parse input/output paths and options
2. Determine output format from extension
3. Build ffmpeg command with appropriate settings
4. Run the transcode

## Resolution Mappings

- `720p` → `-vf "scale=1280:720"`
- `1080p` → `-vf "scale=1920:1080"`
- `4k` → `-vf "scale=3840:2160"`
- `WxH` → `-vf "scale=W:H"`

## Codec Mappings

- `h264` → `-c:v libx264 -preset medium -crf 23`
- `h265` → `-c:v libx265 -preset medium -crf 28`
- `copy` → `-c:v copy` (no re-encoding)

## Base Commands

**Simple format conversion (copy streams):**
```bash
ffmpeg -i "<input>" -c copy "<output>"
```

**With re-encoding:**
```bash
ffmpeg -i "<input>" \
  -c:v libx264 -preset medium -crf 23 \
  -c:a aac -b:a 128k \
  "<output>"
```

**With resolution change:**
```bash
ffmpeg -i "<input>" \
  -vf "scale=1920:1080" \
  -c:v libx264 -preset medium -crf 23 \
  -c:a aac -b:a 128k \
  "<output>"
```

## Examples

User: `/transcode input.avi output.mp4`
→ Convert AVI to MP4 with H.264

User: `/transcode video.mp4 video_720p.mp4 --resolution 720p`
→ Downscale to 720p

User: `/transcode input.mp4 output.mkv --codec h265`
→ Convert to HEVC/H.265

User: `/transcode input.mov output.mp4 --codec copy`
→ Remux without re-encoding (fast)

## Notes

- Use `--codec copy` when possible for speed (no quality loss)
- Re-encoding required for resolution changes
- Audio is converted to AAC 128k unless copying
- Add `-y` to overwrite existing output files
