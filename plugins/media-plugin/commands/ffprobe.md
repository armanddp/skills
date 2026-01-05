---
name: ffprobe
description: Inspect media files or streams for format, codec, and stream information
argument-hint: <input> [--json] [--streams] [--format]
allowed-tools: Bash
---

# FFprobe Media Inspection

Analyze media files or streams to get detailed format and stream information.

## Arguments

- `<input>` - Required. File path or stream URL
- `--json` - Optional. Output in JSON format
- `--streams` - Optional. Show detailed stream info
- `--format` - Optional. Show format/container info
- `--summary` - Optional. Show brief summary only

## Instructions

1. Parse the input path/URL and options
2. Build appropriate ffprobe command
3. Run and display results

## Commands

**Full JSON output (default for detailed analysis):**
```bash
ffprobe -v quiet -print_format json -show_format -show_streams "<input>"
```

**Quick summary:**
```bash
ffprobe -v error -show_entries format=filename,duration,size,bit_rate,format_name -of default=noprint_wrappers=1 "<input>"
```

**Video stream details:**
```bash
ffprobe -v error -select_streams v:0 -show_entries stream=codec_name,codec_long_name,width,height,r_frame_rate,bit_rate,pix_fmt -of default=noprint_wrappers=1 "<input>"
```

**Audio stream details:**
```bash
ffprobe -v error -select_streams a:0 -show_entries stream=codec_name,codec_long_name,sample_rate,channels,bit_rate -of default=noprint_wrappers=1 "<input>"
```

## Examples

User: `/ffprobe video.mp4`
→ Show full JSON analysis

User: `/ffprobe video.mp4 --summary`
→ Show brief format summary

User: `/ffprobe srt://host:9000?mode=caller`
→ Probe live SRT stream

User: `/ffprobe video.mp4 --streams`
→ Show detailed stream information

## Output Interpretation

**Common video codecs:**
- `h264` - H.264/AVC (most common)
- `hevc` - H.265/HEVC (newer, smaller files)
- `vp9` - VP9 (WebM)
- `av1` - AV1 (newest, best compression)

**Common audio codecs:**
- `aac` - Advanced Audio Coding
- `mp3` - MPEG Layer 3
- `opus` - Opus (low latency)
- `ac3` - Dolby Digital

**Frame rate format:**
- `30/1` = 30 fps
- `30000/1001` = 29.97 fps (NTSC)
- `25/1` = 25 fps (PAL)
- `24000/1001` = 23.976 fps (film)

## Notes

- For streams, may take a moment to probe
- Use `-probesize` and `-analyzeduration` for slow streams
- JSON output is best for programmatic analysis
