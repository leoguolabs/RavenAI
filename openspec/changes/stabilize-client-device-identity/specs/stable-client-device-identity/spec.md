## ADDED Requirements

### Requirement: Stable machine and user identity
RavenClient SHALL use a stable opaque identity scoped to the OS machine and OS user, independent of application version, hostname, userData and installation directory. Raw machine identifiers MUST remain local.

#### Scenario: Reinstall with cleared application data
- **WHEN** an upgraded or new client is reinstalled on the same machine and OS user with empty userData
- **THEN** it registers the same device ID and the service retains one device entry

#### Scenario: Different machine or OS user
- **WHEN** a bound client profile is copied to another machine or OS user
- **THEN** its automatic identity differs from the source profile

### Requirement: Preserve migrated and custom identities
The client SHALL migrate existing IDs and names into an independent durable record and SHALL persist explicit identity edits there. Matching durable identity SHALL take precedence over stale profile settings.

#### Scenario: Existing installation upgrade and reinstall
- **WHEN** an installation with a legacy ID upgrades, then reinstalls with empty userData
- **THEN** its original ID and display name remain usable

#### Scenario: Unavailable system identifier
- **WHEN** system machine identification is unavailable
- **THEN** the client reuses its durable identity or persists a random fallback, retaining it on subsequent launches and when system identification becomes available

### Requirement: Latest connection owns device state
The service SHALL route state updates and prompts only through the currently registered connection. It SHALL treat repeat registration on the same socket as idempotent.

#### Scenario: Reconnect replaces old socket
- **WHEN** a second socket registers an existing ID and the first socket disconnects or sends a late update
- **THEN** the replacement remains online, receives prompts and owns its capabilities and heartbeat state

#### Scenario: Service restart
- **WHEN** the service restores cached devices and the client reconnects with its stable ID
- **THEN** the existing device entry becomes online without adding a duplicate
