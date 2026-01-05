---
name: livekit-server-ruby
description: LiveKit server-side Ruby SDK expertise for token generation, room management, webhooks, and API integration. Use when working with LiveKit Ruby gem, AccessToken, VideoGrant, RoomServiceClient, webhooks, or backend LiveKit integration.
---

# LiveKit Server Ruby SDK

Expert knowledge for LiveKit server-side integration using the Ruby SDK.

## Installation

```ruby
# Gemfile
gem 'livekit-server-sdk'
```

## Token Generation

### Basic Token Creation

```ruby
require 'livekit'

token = LiveKit::AccessToken.new(
  api_key: ENV['LIVEKIT_API_KEY'],
  api_secret: ENV['LIVEKIT_API_SECRET']
)

token.identity = 'user-123'
token.name = 'John Doe'
token.ttl = 3600 # 1 hour

token.video_grant = LiveKit::VideoGrant.from_hash(
  roomJoin: true,
  room: 'my-room'
)

jwt = token.to_jwt
```

### VideoGrant Permissions

All available VideoGrant options:

```ruby
token.video_grant = LiveKit::VideoGrant.from_hash(
  # Room permissions
  roomJoin: true,           # Required to join a room
  room: 'room-name',        # Required: room name
  roomCreate: false,        # Create/delete rooms
  roomList: false,          # List available rooms
  roomAdmin: false,         # Moderate room (kick, mute)
  roomRecord: false,        # Use Egress service

  # Publishing permissions
  canPublish: true,         # Publish audio/video tracks
  canPublishData: true,     # Publish data messages
  canPublishSources: ['camera', 'microphone', 'screen_share', 'screen_share_audio'],

  # Subscribing permissions
  canSubscribe: true,       # Subscribe to other tracks

  # Metadata
  canUpdateOwnMetadata: false,  # Update own participant metadata

  # Visibility
  hidden: false,            # Hide from other participants

  # Participant type
  kind: 'standard',         # standard, ingress, egress, sip, agent

  # Advanced
  ingressAdmin: false,      # Use Ingress service
  destinationRoom: nil      # Room for forwarding
)
```

### Common Token Patterns

**Viewer-only (subscribe only):**
```ruby
token.video_grant = LiveKit::VideoGrant.from_hash(
  roomJoin: true,
  room: 'broadcast-room',
  canSubscribe: true,
  canPublish: false,
  canPublishData: false
)
```

**Broadcaster (publish only):**
```ruby
token.video_grant = LiveKit::VideoGrant.from_hash(
  roomJoin: true,
  room: 'broadcast-room',
  canSubscribe: false,
  canPublish: true,
  canPublishSources: ['camera', 'microphone']
)
```

**Admin token:**
```ruby
token.video_grant = LiveKit::VideoGrant.from_hash(
  roomJoin: true,
  room: 'any-room',
  roomAdmin: true,
  roomRecord: true,
  canPublish: true,
  canSubscribe: true
)
```

## Room Management

### RoomServiceClient Setup

```ruby
require 'livekit'

room_service = LiveKit::RoomServiceClient.new(
  ENV['LIVEKIT_URL'],
  api_key: ENV['LIVEKIT_API_KEY'],
  api_secret: ENV['LIVEKIT_API_SECRET']
)
```

### Room Operations

**Create a room:**
```ruby
room = room_service.create_room(
  name: 'my-room',
  empty_timeout: 600,       # Close after 10 min empty
  max_participants: 50
)
```

**List rooms:**
```ruby
rooms = room_service.list_rooms
rooms.each do |room|
  puts "#{room.name}: #{room.num_participants} participants"
end
```

**Delete a room:**
```ruby
# This disconnects all participants immediately
room_service.delete_room(name: 'my-room')
```

**Get room info:**
```ruby
room = room_service.get_room(name: 'my-room')
```

### Participant Management

**List participants:**
```ruby
participants = room_service.list_participants(room: 'my-room')
```

**Remove participant:**
```ruby
room_service.remove_participant(
  room: 'my-room',
  identity: 'user-123'
)
```

**Update participant metadata:**
```ruby
room_service.update_participant(
  room: 'my-room',
  identity: 'user-123',
  metadata: '{"role": "moderator"}'
)
```

**Mute participant track:**
```ruby
room_service.mute_published_track(
  room: 'my-room',
  identity: 'user-123',
  track_sid: 'TR_xxxxx',
  muted: true
)
```

## Webhooks

### Webhook Events

| Event | Description |
|-------|-------------|
| `room_started` | Room created/first participant joined |
| `room_finished` | Room closed |
| `participant_joined` | Participant connected |
| `participant_left` | Participant disconnected |
| `track_published` | Track published to room |
| `track_unpublished` | Track removed |
| `egress_started` | Recording/stream started |
| `egress_updated` | Egress status changed |
| `egress_ended` | Recording/stream ended |
| `ingress_started` | Ingress stream started |
| `ingress_ended` | Ingress stream ended |

### Webhook Verification (Rails Example)

```ruby
class WebhooksController < ApplicationController
  skip_before_action :verify_authenticity_token

  def livekit
    # Verify webhook signature
    receiver = LiveKit::WebhookReceiver.new(
      api_key: ENV['LIVEKIT_API_KEY'],
      api_secret: ENV['LIVEKIT_API_SECRET']
    )

    auth_header = request.headers['Authorization']
    body = request.raw_post

    begin
      event = receiver.receive(body, auth_header)

      case event.event
      when 'room_started'
        handle_room_started(event.room)
      when 'participant_joined'
        handle_participant_joined(event.participant)
      when 'track_published'
        handle_track_published(event.track)
      end

      head :ok
    rescue => e
      Rails.logger.error("Webhook verification failed: #{e.message}")
      head :unauthorized
    end
  end
end
```

## Best Practices

### Token Security
- **Never expose API secret** in client-side code
- **Set appropriate TTL** - tokens should expire (1-24 hours typical)
- **Use minimal permissions** - grant only what's needed
- **Validate identity** - ensure token identity matches authenticated user

### Room Management
- **Use empty_timeout** - prevent orphaned rooms
- **Set max_participants** - prevent overloading
- **Use metadata** - store room state without external DB calls

### Error Handling
```ruby
begin
  room_service.create_room(name: 'my-room')
rescue LiveKit::TwirpError => e
  case e.code
  when :already_exists
    # Room already exists
  when :permission_denied
    # Invalid credentials
  when :resource_exhausted
    # Rate limited
  else
    raise
  end
end
```

## Environment Variables

Required configuration:
```bash
LIVEKIT_URL=wss://your-project.livekit.cloud
LIVEKIT_API_KEY=your-api-key
LIVEKIT_API_SECRET=your-api-secret
```

## References

- [LiveKit Ruby SDK](https://github.com/livekit/server-sdk-ruby)
- [Token Generation Docs](https://docs.livekit.io/home/server/generating-tokens/)
- [Room Management Docs](https://docs.livekit.io/home/server/managing-rooms/)
- [Webhooks Docs](https://docs.livekit.io/home/server/webhooks/)
