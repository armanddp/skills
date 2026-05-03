# StreamBridge Context

Canonical repo: `/Users/m1/source/livelabs/streambridge-app`

Product promise: “Describe the event, invite the cameras, and go live.”

Core v1 objects:
- `Event`
- `Stream`
- `EventPage`
- client-composed `ProducerState`
- event-scoped `DirectorSession`

Core v1 surfaces:
- Director: setup and iteration.
- Event page: viewer-facing live experience from primitives.
- Producer surface: live control page at `/events/:id/produce`.

Truth rules:
- If Director creates a camera, a real backend stream must exist.
- If an event page references a camera, it must bind to a real stream identity.
- Producer output must use real live route/page state.
- Featured camera, overlays, and published state must be server-backed and broadcastable.

Out of scope for v1 unless explicitly pulled in:
- autonomous directing
- full broadcast automation
- replay/slow motion
- audio mixing
- complex editorial workflows
