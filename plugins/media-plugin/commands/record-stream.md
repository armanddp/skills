---
name: record-stream
description: Record a live stream to a file
argument-hint: <url> <output> [--duration <seconds>]
allowed-tools: Bash
---

# Record Stream

Capture a live stream and save it to a file.

## Arguments

- `<url>` - Required. The stream URL to record
- `<output>` - Required. Output file path
- `--duration <seconds>` - Optional. Recording duration (default: until stopped)

## Instructions

1. Parse the stream URL, output path, and optional duration
2. Build ffmpeg command with appropriate settings
3. Run the recording

## Commands

**Record indefinitely (until Ctrl+C):**
```bash
ffmpeg -i "<url>" -c copy "<output>"
```

**Record with duration limit:**
```bash
ffmpeg -i "<url>" -c copy -t <seconds> "<output>"
```

**Record with re-encoding (if source codec not compatible):**
```bash
ffmpeg -i "<url>" \
  -c:v libx264 -preset fast -crf 23 \
  -c:a aac -b:a 128k \
  "<output>"
```

## Examples

User: `/record-stream srt://host:9000?mode=caller recording.mp4`
→ Record SRT stream to MP4 indefinitely

User: `/record-stream rtmp://live.example.com/app/stream clip.mkv --duration 60`
→ Record 60 seconds of RTMP stream

User: `/record-stream udp://239.0.0.1:1234 output.ts --duration 300`
→ Record 5 minutes of UDP multicast

## Output Format Tips

- `.mp4` - Most compatible, requires H.264/AAC
- `.mkv` - Accepts any codec, good for raw capture
- `.ts` - MPEG-TS, good for transport streams
- `.mov` - QuickTime, good for Apple ecosystem

## Notes

- Use `-c copy` to avoid re-encoding (faster, no quality loss)
- MKV container accepts almost any codec without re-encoding
- For MP4, source must be H.264/H.265 + AAC or re-encode
- Run in background with `&` if needed, stop with `pkill ffmpeg`
