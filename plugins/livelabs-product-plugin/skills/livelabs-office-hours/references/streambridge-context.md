# StreamBridge Context

Canonical repo: `/Users/m1/source/livelabs/streambridge-app`

Current product promise: “Describe the event, invite the cameras, and go live.”

Big vision from SPC: Stream is not only a streaming tool. It is a human-camera network with AI curation: authentic human perspectives, intelligently surfaced. The long arc is “8 billion cameras. One AI network. Every event on Earth.”

Current v1 thesis:
- Director is the primary setup/orchestration surface.
- A producer describes an event.
- StreamBridge creates real events, real camera streams, invite links/QRs, a generated event page, and a lightweight producer surface.
- The live moment must bind to real state, not mock previews.

Important StreamBridge assets:
- SRT-first ingest/egress with warm FFmpeg pool members.
- LiveKit/WebRTC preview and event rooms named `event_<event_uuid>`.
- Rails API, PostgreSQL, ActionCable, SolidQueue.
- Director service for stream/design/studio workflows.
- Event page primitives: camera grid, map, leaderboard, timers, overlays, chat, stats.
- Producer surface direction: calm live control room with program output, camera deck, Director chat, and compact controls.

Strategic users:
- independent producers
- endurance race organizers
- school/grassroots sports organizers
- news/field teams
- institutes for Chumo-style streaming operations

Decision filter: does this help a producer go from event intent to live multi-camera output faster, with less coordination, and in a way that compounds toward AI-curated human cameras?
