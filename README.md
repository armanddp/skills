# LiveLabs Marketplace

A Claude Code plugin marketplace containing specialized skills and tools for LiveLabs development.

## Plugins

| Plugin | Description | Skills |
|--------|-------------|--------|
| [devops-plugin](#devops-plugin) | Kubernetes and Google Cloud management | 2 |
| [dimillian-ios](#dimillian-ios) | iOS/SwiftUI development by Dimillian | 9 |
| [media-plugin](#media-plugin) | FFmpeg/FFplay streaming toolkit | 1 |
| [livekit-plugin](#livekit-plugin) | LiveKit WebRTC expertise | 5 |
| [ruby-on-rails-plugin](#ruby-on-rails-plugin) | Rails API with vanilla Rails patterns | 1 |

---

## Installation

### 1. Add the Marketplace

Add to your Claude Code settings (`~/.claude/settings.json` or project `.claude/settings.json`):

```json
{
  "extraKnownMarketplaces": {
    "livelabs-marketplace": {
      "source": {
        "source": "github",
        "repo": "armanddp/skills"
      }
    }
  }
}
```

### 2. Enable Plugins

Enable plugins via Claude Code:

```bash
# Enable all plugins
/plugin enable devops-plugin@livelabs-marketplace
/plugin enable media-plugin@livelabs-marketplace
/plugin enable livekit-plugin@livelabs-marketplace
/plugin enable ruby-on-rails-plugin@livelabs-marketplace
/plugin enable dimillian-ios@livelabs-marketplace
```

Or add to settings:

```json
{
  "enabledPlugins": {
    "devops-plugin@livelabs-marketplace": true,
    "media-plugin@livelabs-marketplace": true,
    "livekit-plugin@livelabs-marketplace": true,
    "ruby-on-rails-plugin@livelabs-marketplace": true,
    "dimillian-ios@livelabs-marketplace": true
  }
}
```

### 3. Local Development

For testing changes locally:

```bash
# Clone the marketplace (with submodules)
git clone --recurse-submodules git@github.com:armanddp/skills.git livelabs-marketplace

# Or if already cloned, initialize submodules
git submodule update --init --recursive

# Add as local marketplace
/plugin marketplace add ./livelabs-marketplace
```

---

## Plugin Details

### devops-plugin

DevOps skills for infrastructure management with per-project configuration persistence.

#### Skills

| Skill | Triggers On | Description |
|-------|-------------|-------------|
| `k8s-manage` | kubectl, pods, deployments, kubernetes | Kubernetes cluster management with context/namespace persistence |
| `gcloud-manage` | gcloud, GCP, Google Cloud | Google Cloud configuration with project/region persistence |

**Features:**
- Per-project config stored in `.devops/k8s.json` and `.devops/gcloud.json`
- Announces active config on first use in session
- Change namespace/project on request

---

### dimillian-ios

iOS and SwiftUI development skills by [Thomas Ricouard (Dimillian)](https://github.com/Dimillian). Included as a git submodule from [Dimillian/Skills](https://github.com/Dimillian/Skills).

#### Skills

| Skill | Triggers On | Description |
|-------|-------------|-------------|
| `app-store-changelog` | changelog, release notes | Generates App Store release notes from git history |
| `gh-issue-fix-flow` | GitHub issue, fix flow | End-to-end GitHub issue resolution workflow |
| `ios-debugger-agent` | iOS debug, simulator | Builds, runs, and debugs iOS apps on simulators |
| `macos-spm-app-packaging` | macOS app, SwiftPM packaging | Scaffolds and packages SwiftPM macOS apps |
| `swift-concurrency-expert` | Swift concurrency, async/await | Reviews and fixes Swift Concurrency compliance |
| `swiftui-liquid-glass` | Liquid Glass, iOS 26, glass effect | iOS 26+ Liquid Glass API implementation |
| `swiftui-performance-audit` | SwiftUI performance, rendering | Diagnoses and improves SwiftUI rendering performance |
| `swiftui-ui-patterns` | SwiftUI patterns, view composition | Best practices for SwiftUI view composition |
| `swiftui-view-refactor` | SwiftUI refactor, view structure | Standardizes view structure and dependencies |

**Updating skills:**
```bash
cd plugins/dimillian-ios/skills
git pull origin main
```

---

### media-plugin

FFmpeg and FFplay toolkit for video streaming and media processing.

#### Skills

| Skill | Triggers On | Description |
|-------|-------------|-------------|
| `ffmpeg-toolkit` | ffmpeg, streaming, video, transcode | Comprehensive FFmpeg knowledge |

#### Commands

| Command | Usage | Description |
|---------|-------|-------------|
| `/stream-test` | `<url> [--source <file>]` | Stream SMPTE bars or file to endpoint |
| `/ffplay` | `<url> [--fullscreen]` | Play stream in background |
| `/transcode` | `<input> <output> [--resolution]` | Convert media files |
| `/ffprobe` | `<input> [--json]` | Inspect media files/streams |
| `/record-stream` | `<url> <output> [--duration]` | Record live stream to file |

**Supported protocols:** SRT, RTMP, RTSP, HLS, UDP multicast

---

### livekit-plugin

Comprehensive LiveKit WebRTC expertise for real-time communication.

#### Skills

| Skill | Triggers On | Description |
|-------|-------------|-------------|
| `livekit-server-ruby` | LiveKit Ruby, AccessToken, VideoGrant | Server-side Ruby SDK |
| `livekit-ios-swift` | LiveKit Swift, iOS room | iOS Swift client SDK |
| `livekit-android` | LiveKit Android, Kotlin | Android Kotlin client SDK |
| `livekit-agents` | LiveKit agents, VoicePipelineAgent | Python agents framework |
| `livekit-egress-ingress` | egress, ingress, RTMP, recording | Recording and streaming |

#### Agents

| Agent | Behavior | Description |
|-------|----------|-------------|
| `livekit-reviewer` | Proactive | Reviews LiveKit code for security and best practices |

**Coverage:**
- Token generation and permissions
- Room management and webhooks
- Track publishing/subscribing
- Voice/video AI agents
- Recording and live streaming

---

### ruby-on-rails-plugin

Rails API development following Basecamp/37signals "vanilla Rails" philosophy.

#### Skills

| Skill | Triggers On | Description |
|-------|-------------|-------------|
| `rails-api` | Rails, Ruby, models, controllers, RSpec | Vanilla Rails patterns and TDD |

**Enforces:**
- Rich domain models, thin controllers
- Concerns over services
- CRUD resources pattern
- Private method indentation
- `_later`/`_now` job pattern
- JSONAPI::Serializer usage
- Test-first development

---

## How Skills Work

Skills auto-trigger based on context. When you mention relevant topics, Claude automatically activates the appropriate skill.

**Example triggers:**

```
"Help me generate a LiveKit token"
→ Activates livekit-server-ruby skill

"Debug why my pods are crashing"
→ Activates k8s-manage skill

"Stream test pattern to SRT endpoint"
→ Activates ffmpeg-toolkit skill

"Create a new Rails model for users"
→ Activates rails-api skill
```

## Directory Structure

```
livelabs-marketplace/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace catalog
├── plugins/
│   ├── devops-plugin/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   └── skills/
│   │       ├── k8s-manage/
│   │       │   └── SKILL.md
│   │       └── gcloud-manage/
│   │           └── SKILL.md
│   ├── dimillian-ios/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   └── skills/              <- git submodule (Dimillian/Skills)
│   │       ├── app-store-changelog/
│   │       ├── swift-concurrency-expert/
│   │       ├── swiftui-liquid-glass/
│   │       └── ...
│   ├── media-plugin/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   ├── skills/
│   │   │   └── ffmpeg-toolkit/
│   │   │       └── SKILL.md
│   │   └── commands/
│   │       ├── stream-test.md
│   │       ├── ffplay.md
│   │       ├── transcode.md
│   │       ├── ffprobe.md
│   │       └── record-stream.md
│   ├── livekit-plugin/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json
│   │   ├── skills/
│   │   │   ├── livekit-server-ruby/
│   │   │   ├── livekit-ios-swift/
│   │   │   ├── livekit-android/
│   │   │   ├── livekit-agents/
│   │   │   └── livekit-egress-ingress/
│   │   └── agents/
│   │       └── livekit-reviewer.md
│   └── ruby-on-rails-plugin/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── skills/
│           └── rails-api/
│               └── SKILL.md
└── README.md
```

## Contributing

1. Clone the repository
2. Create or modify plugins in `plugins/`
3. Update `marketplace.json` if adding new plugins
4. Test locally with `/plugin marketplace add ./`
5. Submit a pull request

## License

MIT
