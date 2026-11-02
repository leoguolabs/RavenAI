## Context

ConfigManager currently generates `${hostname}-${appVersion}-${random}` and saves it only in electron-store. The service indexes persistent device snapshots by that ID. Existing users can edit IDs through IPC. A reconnect closes the old socket, whose disconnect callback can then mark the replacement offline.

## Goals / Non-Goals

Goals: retain identity through application reinstall, version/hostname changes and userData relocation; preserve an existing ID during upgrade; isolate OS users and machines; retain custom identities; verify the full link locally.

Non-goals: recover an already-lost legacy random ID, merge historical duplicates by hostname, authenticate devices, survive OS reinstall/hardware identity replacement, or deploy to production.

## Decisions

1. Derive `raven-<sha256>` from a versioned Raven namespace, platform, OS machine identifier and OS user. Read macOS IOPlatformUUID, Windows MachineGuid (64-bit registry), or Linux machine-id using bounded OS calls. Exclude hostname, app version, installation path and MAC address. Hash locally and never send raw system identifiers. This avoids a new dependency and unstable network-interface identifiers.
2. Save `{version, scope, deviceId, deviceName}` atomically under the OS user's `appData/RavenAI/device-identity/identity-v1.json`, separate from Raven's `userData` and portable data. Prefer a matching durable record, otherwise adopt unbound legacy settings only on initial migration; settings already bound to another scope are ignored. Persist scope back into electron-store. Keep manual identity edits in both stores. New display names default to the hostname.
3. If system identity is unavailable, reuse the durable record; otherwise create a random identity and persist it. If system identity becomes available later, bind that fallback record without changing its ID. Warn on persistent storage errors; a deterministic identity remains recoverable without the record, but preserving legacy/custom/fallback IDs requires that record or the original settings.
4. Keep wire messages unchanged. Registering the same ID on the same socket is idempotent. Replacing a socket publishes the replacement before closing the old one and fails prompts belonging to the superseded connection. WebSocket-originated updates and disconnects carry their socket reference and cannot affect a newer connection.

## Risks / Trade-offs

- A machine/OS-user identity represents one logical client, so concurrent installations share the same identity and the newest connection wins.
- Cloned OS machine IDs or manually assigned duplicate IDs can collide; hostname-only merging would make this worse and is excluded.
- Deleting all settings before the first upgrade loses the old random ID permanently. No automatic historical cleanup is safe without stronger ownership evidence.
- Removing the durable identity record can lose legacy/custom/fallback identity; new deterministic IDs survive if the OS identity is unchanged.

## Migration Plan

Client upgrade writes the existing identity to the durable record on first read. No server data migration or protocol negotiation is needed. Server connection ownership fixes are backward compatible. Reverting to an older client preserves its mirrored settings but removes reinstall recovery. Test against an isolated local API and real Electron main process; leave production untouched.

## Open Questions

None required for this change. Native Windows/Linux machine-identifier execution remains platform-specific verification outside this macOS host.
