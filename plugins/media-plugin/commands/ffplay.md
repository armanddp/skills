---
name: ffplay
description: Play a video stream or file with ffplay
argument-hint: <url> [--fullscreen] [--low-latency]
allowed-tools: Bash
---

# FFplay Playback

Launch ffplay to view a stream or file. Runs in background by default.

## Arguments

- `<url>` - Required. The stream URL or file path
- `--fullscreen` or `-fs` - Optional. Launch in fullscreen mode
- `--low-latency` or `-ll` - Optional. Minimize playback latency

## Instructions

1. Parse the URL/path and options
2. Build the ffplay command with appropriate flags
3. Run in background with `&` so it doesn't block

## Base Command

```bash
ffplay -i "<url>" &
```

## With Options

```bash
# Low latency
ffplay -fflags nobuffer -flags low_delay -i "<url>" &

# Fullscreen
ffplay -fs -i "<url>" &

# Both
ffplay -fflags nobuffer -flags low_delay -fs -i "<url>" &
```

## Examples

User: `/ffplay srt://ingest.example.com:9000?mode=caller`
→ Play SRT stream in window

User: `/ffplay rtmp://live.example.com/app/stream --fullscreen`
→ Play RTMP stream fullscreen

User: `/ffplay udp://239.0.0.1:1234 --low-latency`
→ Play UDP multicast with minimal latency

User: `/ffplay ~/Downloads/video.mp4`
→ Play local file

## Supported Protocols

- `srt://` - Secure Reliable Transport
- `rtmp://` - Real-Time Messaging Protocol
- `rtsp://` - Real-Time Streaming Protocol
- `udp://` - UDP (including multicast)
- `http://` / `https://` - HLS streams (.m3u8)
- Local file paths

## Notes

- Always run with `&` to background the process
- User can close the ffplay window manually
- For SRT, specify `?mode=caller` or `?mode=listener` in URL
