---
name: livekit-android
description: LiveKit Android Kotlin SDK expertise for building real-time video/audio apps. Use when working with LiveKit Android SDK, Room, LocalParticipant, RemoteParticipant, tracks, camera, microphone, or Android WebRTC integration.
---

# LiveKit Android Kotlin SDK

Expert knowledge for building Android apps with LiveKit real-time communication.

## Installation

### Gradle (build.gradle.kts)

```kotlin
dependencies {
    implementation("io.livekit:livekit-android:2.5.0")

    // For Compose UI components (optional)
    implementation("io.livekit:livekit-android-compose-components:2.5.0")
}
```

### Permissions (AndroidManifest.xml)

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

<!-- For Bluetooth audio -->
<uses-permission android:name="android.permission.BLUETOOTH" android:maxSdkVersion="30" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
```

## Connecting to a Room

### Basic Connection

```kotlin
import io.livekit.android.LiveKit
import io.livekit.android.room.Room
import io.livekit.android.room.RoomListener

class CallActivity : AppCompatActivity() {

    private lateinit var room: Room

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        room = LiveKit.create(applicationContext)

        lifecycleScope.launch {
            room.connect(
                url = "wss://your-project.livekit.cloud",
                token = accessToken
            )
        }
    }
}
```

### Connection with Options

```kotlin
val connectOptions = ConnectOptions(
    autoSubscribe = true,
    protocolVersion = ProtocolVersion.v9
)

val roomOptions = RoomOptions(
    adaptiveStream = true,
    dynacast = true,
    e2eeOptions = null
)

room.connect(
    url = serverUrl,
    token = token,
    options = connectOptions,
    roomOptions = roomOptions
)
```

## Room Events

### Using RoomListener

```kotlin
room.listener = object : RoomListener {

    override fun onConnectionStateChanged(
        room: Room,
        state: ConnectionState,
        error: Exception?
    ) {
        when (state) {
            ConnectionState.DISCONNECTED -> Log.d(TAG, "Disconnected")
            ConnectionState.CONNECTING -> Log.d(TAG, "Connecting...")
            ConnectionState.RECONNECTING -> Log.d(TAG, "Reconnecting...")
            ConnectionState.CONNECTED -> Log.d(TAG, "Connected!")
        }
    }

    override fun onParticipantConnected(
        room: Room,
        participant: RemoteParticipant
    ) {
        Log.d(TAG, "Participant joined: ${participant.identity}")
    }

    override fun onParticipantDisconnected(
        room: Room,
        participant: RemoteParticipant
    ) {
        Log.d(TAG, "Participant left: ${participant.identity}")
    }

    override fun onTrackSubscribed(
        track: Track,
        publication: RemoteTrackPublication,
        participant: RemoteParticipant,
        room: Room
    ) {
        if (track is VideoTrack) {
            track.addRenderer(videoView)
        }
    }

    override fun onTrackUnsubscribed(
        track: Track,
        publication: RemoteTrackPublication,
        participant: RemoteParticipant,
        room: Room
    ) {
        if (track is VideoTrack) {
            track.removeRenderer(videoView)
        }
    }
}
```

### Using Flow (Recommended)

```kotlin
lifecycleScope.launch {
    room.events.collect { event ->
        when (event) {
            is RoomEvent.ParticipantConnected -> {
                Log.d(TAG, "Joined: ${event.participant.identity}")
            }
            is RoomEvent.TrackSubscribed -> {
                handleTrackSubscribed(event.track, event.participant)
            }
            is RoomEvent.Disconnected -> {
                handleDisconnect(event.error)
            }
        }
    }
}
```

## Publishing Tracks

### Camera and Microphone

```kotlin
// Enable camera
room.localParticipant.setCameraEnabled(true)

// Enable microphone
room.localParticipant.setMicrophoneEnabled(true)

// With specific options
val cameraCaptureOptions = LocalVideoTrackOptions(
    position = CameraPosition.FRONT,
    captureParams = VideoPreset.H720.capture
)

room.localParticipant.setCameraEnabled(
    enabled = true,
    options = cameraCaptureOptions
)
```

### Publish Options

```kotlin
val videoPublishOptions = VideoPublishOptions(
    videoEncoding = VideoEncoding(
        maxBitrate = 1_500_000,  // 1.5 Mbps
        maxFps = 30
    ),
    simulcast = true,
    scalabilityMode = ScalabilityMode.L3T3
)

room.localParticipant.setCameraEnabled(
    enabled = true,
    publishOptions = videoPublishOptions
)
```

### Screen Share

```kotlin
// Create MediaProjection intent
val mediaProjectionManager = getSystemService(Context.MEDIA_PROJECTION_SERVICE)
    as MediaProjectionManager
val screenCaptureIntent = mediaProjectionManager.createScreenCaptureIntent()

// Launch screen capture picker
screenCaptureLauncher.launch(screenCaptureIntent)

// In result callback
private val screenCaptureLauncher = registerForActivityResult(
    ActivityResultContracts.StartActivityForResult()
) { result ->
    if (result.resultCode == Activity.RESULT_OK) {
        result.data?.let { intent ->
            room.localParticipant.setScreenShareEnabled(true, intent)
        }
    }
}
```

## Rendering Video

### Using SurfaceViewRenderer

```kotlin
// In layout XML
<org.webrtc.SurfaceViewRenderer
    android:id="@+id/videoView"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />

// In code
val videoView = findViewById<SurfaceViewRenderer>(R.id.videoView)
videoView.init(room.eglBase.eglBaseContext, null)

// Attach track
videoTrack.addRenderer(videoView)

// Cleanup
override fun onDestroy() {
    videoTrack.removeRenderer(videoView)
    videoView.release()
    super.onDestroy()
}
```

### Using Jetpack Compose

```kotlin
import io.livekit.android.compose.VideoTrackView

@Composable
fun ParticipantView(participant: Participant) {
    val videoTrack = participant.videoTrackPublications
        .firstOrNull()?.track as? VideoTrack

    videoTrack?.let { track ->
        VideoTrackView(
            track = track,
            modifier = Modifier
                .fillMaxWidth()
                .aspectRatio(16f / 9f)
        )
    }
}
```

## Data Messages

### Send Data

```kotlin
// Send to all participants
room.localParticipant.publishData(
    data = "Hello".toByteArray(),
    reliability = DataPublishReliability.RELIABLE,
    topic = "chat"
)

// Send to specific participants
room.localParticipant.publishData(
    data = messageData,
    reliability = DataPublishReliability.RELIABLE,
    topic = "private",
    destinationIdentities = listOf("user-123")
)
```

### Receive Data

```kotlin
override fun onDataReceived(
    data: ByteArray,
    participant: RemoteParticipant?,
    room: Room,
    topic: String?
) {
    if (topic == "chat") {
        val message = String(data, Charsets.UTF_8)
        Log.d(TAG, "Chat message: $message")
    }
}

// Or with Flow
room.events.filterIsInstance<RoomEvent.DataReceived>()
    .collect { event ->
        handleDataReceived(event.data, event.topic)
    }
```

## Track Quality Control

### Set Preferred Quality

```kotlin
// Request specific quality
publication.setVideoQuality(VideoQuality.HIGH)  // LOW, MEDIUM, HIGH
```

### Disable Track Temporarily

```kotlin
// Stop receiving video to save bandwidth
publication.setSubscribed(false)

// Resume later
publication.setSubscribed(true)
```

## Disconnection

```kotlin
// Graceful disconnect
room.disconnect()

// Handle in lifecycle
override fun onDestroy() {
    room.disconnect()
    room.release()  // Release all resources
    super.onDestroy()
}
```

## Best Practices

### Permission Handling

```kotlin
private val permissionLauncher = registerForActivityResult(
    ActivityResultContracts.RequestMultiplePermissions()
) { permissions ->
    val allGranted = permissions.all { it.value }
    if (allGranted) {
        connectToRoom()
    } else {
        showPermissionError()
    }
}

fun requestPermissions() {
    permissionLauncher.launch(
        arrayOf(
            Manifest.permission.CAMERA,
            Manifest.permission.RECORD_AUDIO
        )
    )
}
```

### Audio Focus

```kotlin
// Configure audio for voice/video chat
LiveKit.create(
    appContext = applicationContext,
    audioType = AudioType.CommunicationMode()  // Optimized for voice
)
```

### Background Handling

```kotlin
// In your Room
override fun onStop() {
    super.onStop()
    // Disable camera when backgrounded
    room.localParticipant.setCameraEnabled(false)
}

override fun onStart() {
    super.onStart()
    // Re-enable camera when foregrounded
    room.localParticipant.setCameraEnabled(true)
}
```

### Error Handling

```kotlin
try {
    room.connect(url, token)
} catch (e: RoomException) {
    when (e) {
        is RoomException.ConnectException -> {
            Log.e(TAG, "Connection failed: ${e.message}")
        }
        is RoomException.TimeoutException -> {
            Log.e(TAG, "Connection timed out")
        }
        else -> {
            Log.e(TAG, "Room error: ${e.message}")
        }
    }
}
```

### Memory Management

```kotlin
class CallViewModel : ViewModel() {
    private val room = LiveKit.create(appContext)

    override fun onCleared() {
        room.disconnect()
        room.release()
        super.onCleared()
    }
}
```

## References

- [LiveKit Android SDK](https://github.com/livekit/client-sdk-android)
- [Android Quickstart](https://docs.livekit.io/realtime/quickstarts/android/)
- [Compose Components](https://github.com/livekit/client-sdk-android/tree/main/livekit-android-compose-components)
