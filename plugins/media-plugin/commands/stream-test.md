---
name: stream-test
description: Stream SMPTE test pattern with 1kHz tone to an endpoint
argument-hint: <url> [--source <file>]
allowed-tools: Bash
---

# Stream Test Pattern

Stream SMPTE color bars with 1kHz audio tone to the specified endpoint.

## Arguments

- `<url>` - Required. The destination URL (SRT, RTMP, RTSP, UDP)
- `--source <file>` - Optional. Stream a file instead of test pattern

## Instructions

1. Parse the arguments to get the URL and optional source file
2. Determine the protocol from the URL:
   - `srt://` → Use `-f mpegts`
   - `rtmp://` → Use `-f flv`
   - `rtsp://` → Use `-f rtsp`
   - `udp://` → Use `-f mpegts`
3. Build and run the ffmpeg command

## Test Pattern Command

```bash
ffmpeg -re \
  -f lavfi -i "smptebars=size=1920x1080:rate=30" \
  -f lavfi -i "sine=frequency=1000:sample_rate=48000" \
  -c:v libx264 -preset ultrafast -tune zerolatency -b:v 4M \
  -c:a aac -b:a 128k \
  -f <format> "<url>"
```

## File Streaming Command

```bash
ffmpeg -re -i "<source_file>" \
  -c:v libx264 -preset ultrafast -tune zerolatency -b:v 4M \
  -c:a aac -b:a 128k \
  -f <format> "<url>"
```

## Examples

User: `/stream-test srt://ingest.example.com:9000?mode=caller`
→ Stream SMPTE bars to SRT endpoint

User: `/stream-test rtmp://live.example.com/app/key`
→ Stream SMPTE bars to RTMP endpoint

User: `/stream-test srt://host:9000 --source ~/video.mp4`
→ Stream file to SRT endpoint

## Notes

- Run in background with `&` if user wants to continue working
- Use `-t <seconds>` to limit duration if requested
- The command runs until stopped with Ctrl+C or the stream ends
