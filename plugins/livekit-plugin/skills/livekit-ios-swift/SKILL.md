---
name: livekit-ios-swift
description: LiveKit iOS Swift SDK expertise for building real-time video/audio apps. Use when working with LiveKit Swift SDK, Room, LocalParticipant, RemoteParticipant, tracks, camera, microphone, or iOS WebRTC integration.
---

# LiveKit iOS Swift SDK

Expert knowledge for building iOS apps with LiveKit real-time communication.

## Installation

### Swift Package Manager

```swift
// Package.swift or Xcode > File > Add Package Dependencies
dependencies: [
    .package(url: "https://github.com/livekit/client-sdk-swift.git", from: "2.0.0")
]
```

### Import

```swift
import LiveKit
```

## Connecting to a Room

### Basic Connection

```swift
let room = Room()
room.add(delegate: self) // start receiving callbacks

try await room.connect(
    url: "wss://your-project.livekit.cloud",
    token: accessToken
)
```

### Connection with Options

```swift
let connectOptions = ConnectOptions(
    autoSubscribe: true,  // Auto-subscribe to all tracks
    protocolVersion: .v9
)

let roomOptions = RoomOptions(
    adaptiveStream: true,     // Optimize video quality
    dynacast: true,           // Stop sending video when not visible
    e2eeOptions: nil          // End-to-end encryption
)

try await room.connect(
    url: serverUrl,
    token: token,
    connectOptions: connectOptions,
    roomOptions: roomOptions
)
```

## Room Delegate Events

```swift
extension MyViewController: RoomDelegate {

    func room(_ room: Room,
              didUpdateConnectionState state: ConnectionState,
              from oldState: ConnectionState) {
        switch state {
        case .disconnected:
            print("Disconnected from room (was \(oldState))")
        case .connecting:
            print("Connecting…")
        case .reconnecting:
            print("Reconnecting…")
        case .connected:
            print("Connected!")
        }
    }

    // Participant connected
    func room(_ room: Room, participantDidConnect participant: RemoteParticipant) {
        print("Participant connected: \(participant.identity)")
    }

    // Participant disconnected
    func room(_ room: Room, participantDidDisconnect participant: RemoteParticipant) {
        print("Participant disconnected: \(participant.identity)")
    }

    // Track published
    func room(_ room: Room, participant: RemoteParticipant,
              didPublishTrack publication: RemoteTrackPublication) {
        print("Track published: \(publication.kind)")
    }

    // Track subscribed (ready to render)
    func room(_ room: Room, participant: RemoteParticipant,
              didSubscribeTrack publication: RemoteTrackPublication) {
        if let videoTrack = publication.track as? VideoTrack {
            // Attach to view
            videoTrack.add(renderer: videoView)
        }
    }

    // Track unsubscribed
    func room(_ room: Room, participant: RemoteParticipant,
              didUnsubscribeTrack publication: RemoteTrackPublication) {
        if let videoTrack = publication.track as? VideoTrack {
            videoTrack.remove(renderer: videoView)
        }
    }
}
```

## Publishing Tracks

### Camera and Microphone

```swift
// Enable camera
try await room.localParticipant.setCamera(enabled: true)

// Enable microphone
try await room.localParticipant.setMicrophone(enabled: true)

// With specific options
let cameraCaptureOptions = CameraCaptureOptions(
    position: .front,
    preferredFormat: nil,  // Let SDK choose
    fps: 30
)

try await room.localParticipant.setCamera(
    enabled: true,
    captureOptions: cameraCaptureOptions
)
```

### Publish Options

```swift
let videoPublishOptions = VideoPublishOptions(
    encoding: VideoEncoding(
        maxBitrate: 1_500_000,  // 1.5 Mbps
        maxFps: 30
    ),
    simulcast: true,  // Enable simulcast layers
    scalabilityMode: .L3T3
)

try await room.localParticipant.setCamera(
    enabled: true,
    publishOptions: videoPublishOptions
)
```

### Screen Share

```swift
// iOS screen share requires Broadcast Upload Extension
try await room.localParticipant.setScreenShare(enabled: true)
```

## Subscribing to Tracks

### Auto-Subscribe (Default)

With `autoSubscribe: true` (default), tracks are subscribed automatically:

```swift
func room(_ room: Room, participant: RemoteParticipant,
          didSubscribeTrack publication: RemoteTrackPublication) {

    switch publication.track {
    case let videoTrack as VideoTrack:
        videoTrack.add(renderer: remoteVideoView)

    case let audioTrack as AudioTrack:
        // Audio plays automatically
        break

    default:
        break
    }
}
```

### Manual Subscribe

```swift
let connectOptions = ConnectOptions(autoSubscribe: false)

// Later, subscribe to specific tracks
for participant in room.remoteParticipants.values {
    for publication in participant.trackPublications.values {
        if publication.kind == .video {
            try await publication.setSubscribed(true)
        }
    }
}
```

## Rendering Video

### Using VideoView

```swift
import LiveKit

class VideoCell: UICollectionViewCell {
    let videoView = VideoView()

    func configure(with track: VideoTrack) {
        track.add(renderer: videoView)
    }

    override func prepareForReuse() {
        super.prepareForReuse()
        videoView.track = nil
    }
}
```

### SwiftUI Integration

```swift
import LiveKit
import SwiftUI

struct ParticipantView: View {
    @ObservedObject var participant: Participant

    var body: some View {
        if let publication = participant.mainVideoPublication,
           let track = publication.track as? VideoTrack {
            SwiftUIVideoView(track: track)
                .aspectRatio(16/9, contentMode: .fit)
        }
    }
}
```

## Data Messages

### Send Data

```swift
// Send to all participants
try await room.localParticipant.publish(
    data: "Hello".data(using: .utf8)!,
    options: DataPublishOptions(
        topic: "chat",
        reliable: true
    )
)

// Send to specific participants
try await room.localParticipant.publish(
    data: messageData,
    options: DataPublishOptions(
        topic: "private",
        reliable: true,
        destinationIdentities: ["user-123"]
    )
)
```

### Receive Data

```swift
func room(_ room: Room, participant: RemoteParticipant?,
          didReceiveData data: Data, forTopic topic: String, encryptionType: EncryptionType) {

    if topic == "chat" {
        let message = String(data: data, encoding: .utf8)
        print("Chat message: \(message ?? "")")
    }
}
```

## Track Quality Control

### Set Preferred Quality

```swift
// Request specific quality for a track
if let publication = participant.trackPublications["TR_xxx"] as? RemoteTrackPublication {
    try await publication.setVideoQuality(.high)  // .low, .medium, .high
}
```

### Adaptive Streaming

With `adaptiveStream: true`, the SDK automatically optimizes video quality based on:
- View size
- Network conditions
- CPU usage

## Disconnection

```swift
// Graceful disconnect
await room.disconnect()

// Handle unexpected disconnection
func room(_ room: Room, didDisconnectWithError error: LiveKitError?) {
    if let error = error {
        print("Disconnected with error: \(error)")
        // Attempt reconnect or show error UI
    } else {
        print("Disconnected gracefully")
    }
}
```

## Best Practices

### Permission Handling

```swift
import AVFoundation

func requestPermissions() async -> Bool {
    let videoStatus = await AVCaptureDevice.requestAccess(for: .video)
    let audioStatus = await AVCaptureDevice.requestAccess(for: .audio)
    return videoStatus && audioStatus
}
```

### Background Audio

```swift
// In AppDelegate or SceneDelegate
import AVFAudio

func configureAudioSession() {
    let session = AVAudioSession.sharedInstance()
    try? session.setCategory(
        .playAndRecord,
        mode: .videoChat,
        options: [.defaultToSpeaker, .allowBluetooth]
    )
    try? session.setActive(true)
}
```

### Memory Management

```swift
// Remove video renderers when view disappears
override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)

    for participant in room.remoteParticipants.values {
        for publication in participant.trackPublications.values {
            if let videoTrack = publication.track as? VideoTrack {
                videoTrack.remove(renderer: videoView)
            }
        }
    }
}
```

### Error Handling

```swift
do {
    try await room.connect(url: url, token: token)
} catch let error as LiveKitError {
    switch error {
    case .invalidURL:
        print("Invalid server URL")
    case .timeout:
        print("Connection timed out")
    case .serverUnreachable:
        print("Cannot reach server")
    default:
        print("Connection error: \(error)")
    }
}
```

## References

- [LiveKit iOS SDK](https://github.com/livekit/client-sdk-swift)
- [iOS Quickstart](https://docs.livekit.io/realtime/quickstarts/ios/)
- [Room Connection Docs](https://docs.livekit.io/home/client/connect/)
- [Track Management Docs](https://docs.livekit.io/home/client/tracks/)
