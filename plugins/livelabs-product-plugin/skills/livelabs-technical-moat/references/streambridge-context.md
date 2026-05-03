# StreamBridge Context

Canonical repo: `/Users/m1/source/livelabs/streambridge-app`

Current architecture:
- SRT router accepts publisher/viewer SRT and allocates warm FFmpeg pool members.
- FFmpeg pool fans out SRT through MediaMTX and publishes WHIP to LiveKit when configured.
- LiveKit provides WebRTC preview/live rooms and data channels.
- Rails orchestrates events, streams, routes, ingresses, and auth.
- Director is the AI producer surface.

Moat candidates:
- hard realtime orchestration across unreliable human cameras
- event/camera/operator/viewer data loops
- AI Director actions and training traces
- generated event pages bound to real live media
- fast setup workflow from natural language to live production
- future spatial positioning of camera feeds

Commoditized parts:
- raw WebRTC/SFU infrastructure
- generic AI chat
- generic landing pages
- single-camera streaming
- plain event CRUD
