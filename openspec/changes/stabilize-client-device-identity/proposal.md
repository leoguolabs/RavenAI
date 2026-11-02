## Why

RavenClient creates a hostname/version/random device ID in its disposable application settings. Reinstalling with cleared settings creates duplicate RavenAIService devices and breaks references to the original ID.

## What Changes

- Resolve a version-independent, machine-and-OS-user-scoped identity, with a durable identity record outside the application data directory.
- Preserve existing and explicitly edited IDs across upgrades and subsequent reinstalls; use a deterministic hashed system identity for new installations and a persisted random fallback when system identity is unavailable.
- Bind local settings to their identity scope so copying settings to another computer does not copy its device identity.
- Keep the latest WebSocket connection authoritative when a stable device ID reconnects.
- Verify reinstall, migration, scope isolation, reconnect, heartbeat, capabilities and prompt routing against a real local service.

## Capabilities

### New Capabilities

- `stable-client-device-identity`: Reinstall-safe client identity and connection ownership.

### Modified Capabilities

None.

## Impact

RavenClient ConfigManager and a new main-process identity resolver; RavenAIService device-link manager and WebSocket endpoint; focused regression and local end-to-end tests. The wire protocol remains compatible. No production deployment, historical-device deletion, or account-authentication changes are included.
