---
name: livekit-egress-ingress
description: LiveKit Egress and Ingress expertise for recording rooms, streaming to RTMP, and importing external streams. Use when working with recording, egress, ingress, RTMP streaming, WHIP, HLS, or exporting LiveKit sessions.
---

# LiveKit Egress & Ingress

Expert knowledge for recording, streaming, and importing media with LiveKit.

## Egress Overview

Egress exports media from LiveKit rooms. Types include:
- **Room Composite** - Record entire room with layout
- **Participant Egress** - Record specific participant
- **Track Composite** - Record specific tracks
- **Track Egress** - Export raw tracks
- **Web Egress** - Record any web page

## Ingress Overview

Ingress imports external streams into LiveKit rooms:
- **RTMP/RTMPS** - From OBS, encoders
- **WHIP** - WebRTC HTTP Ingestion
- **HTTP Media** - HLS, MP4, audio files
- **SRT** - Secure Reliable Transport

---

## Room Composite Egress

Record the entire room with a web-based layout.

### Ruby Example

```ruby
require 'livekit'

egress_client = LiveKit::EgressServiceClient.new(
  ENV['LIVEKIT_URL'],
  api_key: ENV['LIVEKIT_API_KEY'],
  api_secret: ENV['LIVEKIT_API_SECRET']
)

# Record to file
egress = egress_client.start_room_composite_egress(
  room_name: 'my-room',
  file_outputs: [
    LiveKit::EncodedFileOutput.new(
      file_type: LiveKit::EncodedFileType::MP4,
      filepath: 's3://bucket/recordings/{room_name}-{time}.mp4',
      s3: LiveKit::S3Upload.new(
        access_key: ENV['AWS_ACCESS_KEY'],
        secret: ENV['AWS_SECRET_KEY'],
        bucket: 'my-bucket',
        region: 'us-east-1'
      )
    )
  ],
  preset: LiveKit::EncodingOptionsPreset::H264_720P_30
)

puts "Egress started: #{egress.egress_id}"
```

### Stream to RTMP

```ruby
egress = egress_client.start_room_composite_egress(
  room_name: 'my-room',
  stream_outputs: [
    LiveKit::StreamOutput.new(
      protocol: LiveKit::StreamProtocol::RTMP,
      urls: [
        'rtmp://live.youtube.com/your-stream-key',
        'rtmp://live.twitch.tv/your-stream-key'
      ]
    )
  ],
  preset: LiveKit::EncodingOptionsPreset::H264_1080P_30
)
```

### Record and Stream Simultaneously

```ruby
egress = egress_client.start_room_composite_egress(
  room_name: 'my-room',
  file_outputs: [file_output],
  stream_outputs: [stream_output],
  preset: LiveKit::EncodingOptionsPreset::H264_1080P_30
)
```

## Participant Egress

Record a specific participant's audio and video.

```ruby
egress = egress_client.start_participant_egress(
  room_name: 'my-room',
  identity: 'speaker-1',
  file_outputs: [
    LiveKit::EncodedFileOutput.new(
      file_type: LiveKit::EncodedFileType::MP4,
      filepath: 's3://bucket/participants/{participant_identity}.mp4',
      s3: s3_config
    )
  ]
)
```

## Track Egress

Export individual tracks without transcoding (raw format).

```ruby
# Export audio track to file
egress = egress_client.start_track_egress(
  room_name: 'my-room',
  track_id: 'TR_audio_xxx',
  file: LiveKit::DirectFileOutput.new(
    filepath: 's3://bucket/audio/{track_id}.ogg',
    s3: s3_config
  )
)

# Stream audio to WebSocket (for transcription)
egress = egress_client.start_track_egress(
  room_name: 'my-room',
  track_id: 'TR_audio_xxx',
  websocket_url: 'wss://transcription-service.com/stream'
)
```

## Auto Egress

Automatically record rooms on creation.

```ruby
room = room_service.create_room(
  name: 'auto-record-room',
  egress: LiveKit::RoomEgress.new(
    room: LiveKit::RoomCompositeEgressRequest.new(
      file_outputs: [file_output],
      preset: LiveKit::EncodingOptionsPreset::H264_720P_30
    )
  )
)
```

## Managing Egress

### List Active Egress

```ruby
egresses = egress_client.list_egress(room_name: 'my-room')
egresses.each do |egress|
  puts "#{egress.egress_id}: #{egress.status}"
end
```

### Stop Egress

```ruby
egress_client.stop_egress(egress_id: 'EG_xxx')
```

### Update Stream URLs

```ruby
egress_client.update_stream(
  egress_id: 'EG_xxx',
  add_output_urls: ['rtmp://new-destination.com/key'],
  remove_output_urls: ['rtmp://old-destination.com/key']
)
```

## Encoding Presets

| Preset | Resolution | FPS | Video Bitrate |
|--------|------------|-----|---------------|
| H264_720P_30 | 1280x720 | 30 | 3 Mbps |
| H264_720P_60 | 1280x720 | 60 | 4.5 Mbps |
| H264_1080P_30 | 1920x1080 | 30 | 4.5 Mbps |
| H264_1080P_60 | 1920x1080 | 60 | 6 Mbps |
| PORTRAIT_H264_720P_30 | 720x1280 | 30 | 3 Mbps |
| PORTRAIT_H264_1080P_30 | 1080x1920 | 30 | 4.5 Mbps |

## Custom Encoding

```ruby
egress = egress_client.start_room_composite_egress(
  room_name: 'my-room',
  file_outputs: [file_output],
  options: LiveKit::EncodingOptions.new(
    width: 1920,
    height: 1080,
    video_bitrate: 6_000_000,  # 6 Mbps
    framerate: 60,
    video_codec: LiveKit::VideoCodec::H264_HIGH,
    audio_codec: LiveKit::AudioCodec::AAC,
    audio_bitrate: 192_000,
    audio_frequency: 48000
  )
)
```

---

## RTMP Ingress

Import RTMP streams (from OBS, hardware encoders).

### Create RTMP Ingress

```ruby
ingress_client = LiveKit::IngressServiceClient.new(
  ENV['LIVEKIT_URL'],
  api_key: ENV['LIVEKIT_API_KEY'],
  api_secret: ENV['LIVEKIT_API_SECRET']
)

ingress = ingress_client.create_ingress(
  input_type: LiveKit::IngressInput::RTMP_INPUT,
  name: 'obs-stream',
  room_name: 'broadcast-room',
  participant_identity: 'broadcaster',
  participant_name: 'Live Broadcaster'
)

puts "RTMP URL: #{ingress.url}"
puts "Stream Key: #{ingress.stream_key}"
```

### OBS Configuration

```
Server: rtmp://your-project.livekit.cloud/x
Stream Key: (from ingress.stream_key)
```

## WHIP Ingress

Import WebRTC streams via WHIP protocol.

```ruby
ingress = ingress_client.create_ingress(
  input_type: LiveKit::IngressInput::WHIP_INPUT,
  name: 'webrtc-stream',
  room_name: 'broadcast-room',
  participant_identity: 'webcam-feed'
)

puts "WHIP URL: #{ingress.url}"
```

## URL Ingress (Pull)

Import HTTP media files or streams.

```ruby
# Import HLS stream
ingress = ingress_client.create_ingress(
  input_type: LiveKit::IngressInput::URL_INPUT,
  url: 'https://example.com/stream.m3u8',
  name: 'hls-restream',
  room_name: 'restream-room',
  participant_identity: 'external-feed'
)

# Import MP4 file
ingress = ingress_client.create_ingress(
  input_type: LiveKit::IngressInput::URL_INPUT,
  url: 'https://example.com/video.mp4',
  name: 'video-playback',
  room_name: 'watch-party',
  participant_identity: 'video-player'
)
```

## Managing Ingress

### List Ingress

```ruby
ingresses = ingress_client.list_ingress(room_name: 'my-room')
```

### Update Ingress

```ruby
ingress_client.update_ingress(
  ingress_id: 'IN_xxx',
  name: 'new-name',
  room_name: 'new-room'
)
```

### Delete Ingress

```ruby
ingress_client.delete_ingress(ingress_id: 'IN_xxx')
```

## Video/Audio Configuration

### Video Encoding

```ruby
ingress = ingress_client.create_ingress(
  input_type: LiveKit::IngressInput::RTMP_INPUT,
  room_name: 'my-room',
  participant_identity: 'streamer',
  video: LiveKit::IngressVideoOptions.new(
    source: LiveKit::TrackSource::CAMERA,
    preset: LiveKit::IngressVideoEncodingPreset::H264_1080P_30,
    # Or custom encoding:
    # encoding_options: LiveKit::VideoEncodingOptions.new(...)
  )
)
```

### Audio Encoding

```ruby
ingress = ingress_client.create_ingress(
  input_type: LiveKit::IngressInput::RTMP_INPUT,
  room_name: 'my-room',
  participant_identity: 'streamer',
  audio: LiveKit::IngressAudioOptions.new(
    source: LiveKit::TrackSource::MICROPHONE,
    preset: LiveKit::IngressAudioEncodingPreset::OPUS_STEREO_96KBPS
  )
)
```

## Storage Configuration

### S3-Compatible Storage

```ruby
s3_config = LiveKit::S3Upload.new(
  access_key: ENV['AWS_ACCESS_KEY'],
  secret: ENV['AWS_SECRET_KEY'],
  bucket: 'recordings-bucket',
  region: 'us-east-1',
  # Optional: custom endpoint for S3-compatible services
  endpoint: 'https://s3.custom-provider.com'
)
```

### GCP Storage

```ruby
gcp_config = LiveKit::GCPUpload.new(
  credentials: File.read('service-account.json'),
  bucket: 'recordings-bucket'
)
```

### Azure Blob Storage

```ruby
azure_config = LiveKit::AzureBlobUpload.new(
  account_name: 'myaccount',
  account_key: ENV['AZURE_KEY'],
  container_name: 'recordings'
)
```

## Webhooks for Egress/Ingress

Monitor egress and ingress status via webhooks:

```ruby
# Webhook events
# egress_started - Recording/stream started
# egress_updated - Status changed
# egress_ended - Recording/stream ended
# ingress_started - Input stream started
# ingress_ended - Input stream ended

def handle_webhook(event)
  case event.event
  when 'egress_ended'
    egress = event.egress_info
    if egress.status == 'EGRESS_COMPLETE'
      # Get download URL
      file = egress.file_results.first
      puts "Recording available at: #{file.download_url}"
    end
  end
end
```

## Best Practices

### Recording
- Use appropriate presets for your use case
- Enable auto-egress for rooms that always need recording
- Store recordings in cloud storage, not local disk
- Monitor egress webhooks for failures

### Streaming
- Test RTMP endpoints before going live
- Use multiple output URLs for redundancy
- Monitor stream health via webhooks

### Ingress
- Validate stream before creating ingress
- Set appropriate video/audio quality for bandwidth
- Use WHIP for lowest latency input

## References

- [Egress Docs](https://docs.livekit.io/home/egress/overview/)
- [Ingress Docs](https://docs.livekit.io/home/ingress/overview/)
- [Storage Configuration](https://docs.livekit.io/home/egress/storage/)
