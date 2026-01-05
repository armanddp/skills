---
name: ffmpeg-toolkit
description: FFmpeg and FFplay toolkit for video streaming, playback, transcoding, recording, and media inspection. Use when working with video/audio streaming, SRT/RTMP/RTSP/HLS/UDP protocols, test patterns, ffplay, ffprobe, media conversion, or capturing streams.
---

# FFmpeg Toolkit

You are a media streaming expert helping with ffmpeg, ffplay, and ffprobe operations.

## Prerequisites

Verify ffmpeg is installed before running commands:
```bash
ffmpeg -version
```

## Streaming Test Patterns

### SMPTE Bars with 1kHz Tone

Generate standard SMPTE color bars with a 1kHz audio tone:

```bash
ffmpeg -re \
  -f lavfi -i "smptebars=size=1920x1080:rate=30" \
  -f lavfi -i "sine=frequency=1000:sample_rate=48000" \
  -c:v libx264 -preset ultrafast -tune zerolatency -b:v 4M \
  -c:a aac -b:a 128k \
  -f mpegts "<OUTPUT_URL>"
```

### Protocol-Specific Examples

**SRT (Secure Reliable Transport):**
```bash
# Caller mode (connecting to listener)
ffmpeg -re \
  -f lavfi -i "smptebars=size=1920x1080:rate=30" \
  -f lavfi -i "sine=frequency=1000:sample_rate=48000" \
  -c:v libx264 -preset ultrafast -tune zerolatency -b:v 4M \
  -c:a aac -b:a 128k \
  -f mpegts "srt://hostname:port?mode=caller"

# Listener mode (waiting for connections)
ffmpeg -re \
  -f lavfi -i "smptebars=size=1920x1080:rate=30" \
  -f lavfi -i "sine=frequency=1000:sample_rate=48000" \
  -c:v libx264 -preset ultrafast -tune zerolatency -b:v 4M \
  -c:a aac -b:a 128k \
  -f mpegts "srt://0.0.0.0:port?mode=listener"
```

**RTMP:**
```bash
ffmpeg -re \
  -f lavfi -i "smptebars=size=1920x1080:rate=30" \
  -f lavfi -i "sine=frequency=1000:sample_rate=48000" \
  -c:v libx264 -preset ultrafast -tune zerolatency -b:v 4M \
  -c:a aac -b:a 128k \
  -f flv "rtmp://hostname/app/stream_key"
```

**RTSP:**
```bash
ffmpeg -re \
  -f lavfi -i "smptebars=size=1920x1080:rate=30" \
  -f lavfi -i "sine=frequency=1000:sample_rate=48000" \
  -c:v libx264 -preset ultrafast -tune zerolatency -b:v 4M \
  -c:a aac -b:a 128k \
  -f rtsp "rtsp://hostname:554/stream"
```

**UDP Multicast:**
```bash
ffmpeg -re \
  -f lavfi -i "smptebars=size=1920x1080:rate=30" \
  -f lavfi -i "sine=frequency=1000:sample_rate=48000" \
  -c:v libx264 -preset ultrafast -tune zerolatency -b:v 4M \
  -c:a aac -b:a 128k \
  -f mpegts "udp://239.0.0.1:1234?pkt_size=1316"
```

## Streaming Files to Endpoints

Stream an existing file to any protocol endpoint:

```bash
ffmpeg -re -i "/path/to/file.mp4" \
  -c:v libx264 -preset ultrafast -tune zerolatency -b:v 4M \
  -c:a aac -b:a 128k \
  -f mpegts "srt://hostname:port?mode=caller"
```

**Copy codecs (no re-encoding, faster):**
```bash
ffmpeg -re -i "/path/to/file.mp4" \
  -c copy \
  -f mpegts "srt://hostname:port?mode=caller"
```

## Playback with FFplay

**Run ffplay in background** so it doesn't block Claude:

```bash
ffplay -i "srt://hostname:port?mode=caller" &
```

**With options:**
```bash
# Low latency playback
ffplay -fflags nobuffer -flags low_delay -i "srt://..." &

# Specific window size
ffplay -x 1280 -y 720 -i "srt://..." &

# Fullscreen
ffplay -fs -i "srt://..." &

# With stats overlay
ffplay -stats -i "srt://..." &
```

**Protocol examples:**
```bash
# SRT
ffplay -i "srt://hostname:port?mode=caller" &

# RTMP
ffplay -i "rtmp://hostname/app/stream" &

# RTSP
ffplay -i "rtsp://hostname:554/stream" &

# UDP
ffplay -i "udp://239.0.0.1:1234" &

# HLS
ffplay -i "https://hostname/stream.m3u8" &
```

## Recording Streams

Capture a live stream to file:

```bash
# Record SRT stream to MP4
ffmpeg -i "srt://hostname:port?mode=caller" \
  -c copy \
  -t 60 \
  output.mp4

# Record with duration limit (seconds)
ffmpeg -i "srt://..." -c copy -t 300 recording.mp4

# Record RTMP to MKV
ffmpeg -i "rtmp://hostname/app/stream" \
  -c copy \
  recording.mkv
```

## Transcoding

### Format Conversion

```bash
# MP4 to MKV
ffmpeg -i input.mp4 -c copy output.mkv

# Any format to MP4 (re-encode)
ffmpeg -i input.avi \
  -c:v libx264 -preset medium -crf 23 \
  -c:a aac -b:a 128k \
  output.mp4
```

### Resolution/Quality Changes

```bash
# Scale to 720p
ffmpeg -i input.mp4 \
  -vf "scale=1280:720" \
  -c:v libx264 -preset medium -crf 23 \
  -c:a copy \
  output_720p.mp4

# Scale to 1080p
ffmpeg -i input.mp4 \
  -vf "scale=1920:1080" \
  -c:v libx264 -preset medium -crf 23 \
  -c:a copy \
  output_1080p.mp4
```

### Codec Changes

```bash
# Convert to H.265/HEVC
ffmpeg -i input.mp4 \
  -c:v libx265 -preset medium -crf 28 \
  -c:a aac -b:a 128k \
  output_hevc.mp4

# Extract audio only
ffmpeg -i input.mp4 -vn -c:a copy output.aac
ffmpeg -i input.mp4 -vn -c:a libmp3lame -b:a 192k output.mp3
```

## Media Inspection (FFprobe)

### Basic Info

```bash
ffprobe -v quiet -print_format json -show_format -show_streams input.mp4
```

### Quick Summary

```bash
ffprobe -v error -show_entries format=duration,size,bit_rate -of default=noprint_wrappers=1 input.mp4
```

### Stream Details

```bash
# Video stream info
ffprobe -v error -select_streams v:0 -show_entries stream=codec_name,width,height,r_frame_rate,bit_rate -of csv=p=0 input.mp4

# Audio stream info
ffprobe -v error -select_streams a:0 -show_entries stream=codec_name,sample_rate,channels,bit_rate -of csv=p=0 input.mp4
```

### Probe Live Streams

```bash
ffprobe -v error -show_format -show_streams "srt://hostname:port?mode=caller"
```

## Common Options Reference

### Encoding Presets
- `ultrafast` - Fastest, largest file (good for streaming)
- `superfast`, `veryfast`, `faster`, `fast` - Balance options
- `medium` - Default, good balance
- `slow`, `slower`, `veryslow` - Best quality, slowest

### CRF Quality (H.264/H.265)
- `0` - Lossless
- `18` - Visually lossless
- `23` - Default, good quality
- `28` - Smaller file, acceptable quality
- `51` - Worst quality

### Tune Options
- `zerolatency` - Best for streaming
- `film` - Film content
- `animation` - Animated content
- `grain` - Preserve film grain

## Troubleshooting

### SRT Connection Issues
```bash
# Test with verbose output
ffmpeg -v verbose -i "srt://..." -f null -

# Check if port is open
nc -vz hostname port
```

### Stream Not Playing
```bash
# Try with buffer disabled
ffplay -fflags nobuffer -i "srt://..."

# Increase probe size for slow starts
ffplay -probesize 32M -analyzeduration 10M -i "srt://..."
```

### High Latency
Use these options for lower latency:
```bash
-fflags nobuffer -flags low_delay -strict experimental
```
