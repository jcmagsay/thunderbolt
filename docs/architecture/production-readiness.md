# Sync and E2EE Production Readiness

Cross-device synchronization and optional end-to-end encryption are preview features. This guide explains what they do, what they protect, why they are not yet production-proven, and what work would increase confidence.

## The Two-Minute Model

Thunderbolt stores working data in SQLite on each device.

- **Sync** copies supported data between devices through PowerSync and PostgreSQL.
- **End-to-end encryption (E2EE)** encrypts protected fields on the sending device and decrypts them on an approved receiving device.

```text
Without sync:
Laptop SQLite                         Phone SQLite
(separate data)                       (separate data)

Sync without E2EE:
Laptop SQLite ── readable data ──▶ PowerSync/PostgreSQL ──▶ Phone SQLite

Sync with E2EE:
Laptop ── encrypt ──▶ sync infrastructure stores ciphertext ──▶ decrypt ──▶ Phone
```

E2EE is optional and only affects synchronized fields. It is not required for local use.

## Device and Key Flow

Thunderbolt creates one content key for the account. Each approved device has its own key pairs and receives a device-specific wrapped copy of the content key.

When adding a device:

1. The new device creates its key pairs.
2. It requests approval.
3. An existing trusted device wraps the account content key for it.
4. The new device unwraps the key and decrypts synchronized content.

If every approved device is lost, the documented recovery mechanism is a 24-word recovery phrase. Losing all devices and that phrase means losing access to encrypted data.

Revoking a device prevents future sync access and removes its server-side envelope. It cannot remotely erase plaintext or keys already present on a compromised device.

## Protection Boundary

| E2EE helps protect against | E2EE does not automatically protect against |
| --- | --- |
| A stolen PostgreSQL backup revealing protected content | Malware or another user reading an unlocked device |
| A sync operator reading protected fields | A selected cloud model receiving content for inference |
| Network intermediaries seeing synchronized plaintext | Loss of every device and the recovery phrase |
| Server-side disclosure of encrypted chat and skill fields | Visible sync metadata such as IDs, ownership, status, and timestamps |
| Reading ciphertext without the content key | Prompt injection, excessive tool permissions, or incorrect model output |

The inference path is separate from the sync path. When a user sends content to a cloud model, the provider must receive that content to process it. A local or on-prem model keeps inference under the operator's control.

## Appropriate Uses Today

| Good fit | Use caution or wait |
| --- | --- |
| Development and architecture evaluation | Regulated or high-assurance workloads without independent review |
| Internal pilots with recoverable data | Data that cannot tolerate loss, stale state, or conflicts |
| Cross-platform and offline experiments | Assuming revocation erases an already compromised endpoint |
| Contributing tests, recovery behavior, and operational tooling | Deployments without backups, monitoring, restore drills, and upgrade testing |

The implementation uses established primitives: AES-256-GCM for content, P-256 plus ML-KEM-768 for device envelopes, and a BIP-39 recovery phrase. Production trust depends on the complete protocol and operations, not only primitive selection.

## Why Sync Is Still Preview

### Conflict resolution

Concurrent offline edits resolve last-writer-wins at the row level. One device's update can therefore replace another without a semantic merge. Workflows requiring audit history, field-level merging, or transactional invariants need domain-specific conflict handling.

### Coordinated schema deployment

A synced table or column touches frontend and backend schemas, migrations, the shared table registry, and three PowerSync rule configurations. Deploying the frontend first can produce a feature that works locally but silently fails to replicate. Follow the two-PR process in [PowerSync, Account & Device Management](./powersync-account-devices.md).

### Multiple runtime paths

Chrome, Edge, and Firefox transform incoming data in a custom SharedWorker. Safari, iOS, and Tauri use a main-thread transformer path. Security-sensitive changes must be validated across both implementations, including multi-tab, offline, upgrade, and failure behavior.

### PowerSync internal APIs

The custom SharedWorker extends PowerSync's internal `SharedSyncImplementation` through an alias into `@powersync/web/lib/src`. This is not a supported public extension point. Dependency upgrades can change internal behavior or data shape without a useful compile-time failure.

### Operational maturity

Production use requires PostgreSQL backups, restoration drills, PowerSync monitoring, migration discipline, client compatibility policies, and recovery from partially deployed releases.

## Why E2EE Is Still Preview

### No independent cryptography audit

The protocol has not yet undergone an independent cryptography audit. Review must cover key generation and storage, hybrid wrapping, authentication binding, recovery, device approval, replay and substitution attacks, downgrade behavior, malformed ciphertext, and both transform paths.

### Field-level encryption

Only columns in `src/db/encryption/config.ts` are encrypted before sync. Routing and synchronization metadata can remain visible. Local SQLite contains decrypted application data while in use, so device security still matters.

### One account content key

All approved devices share one account content key. This is simple, but compromise has an account-wide blast radius. The current architecture does not document routine content-key rotation, per-record keys, forward secrecy, or post-compromise re-encryption.

### Revocation is not retroactive

Revocation stops future service access; it cannot make a key or plaintext disappear from an actively compromised endpoint. Protecting future writes from a previously copied key would require content-key rotation and re-encryption.

### Recovery risk

Zero-knowledge recovery is deliberately unforgiving. Deployments need clear backup guidance, recovery drills, abuse-resistant support procedures, and an explicit decision about enterprise escrow.

### Mixed-version compatibility

Production confidence requires a test matrix for enablement, rollback, old and new clients, new encrypted columns, interrupted setup, corrupt envelopes, missing keys, and upgrades across all supported runtimes.

## Work That Would Increase Trust

1. Commission and publish an independent protocol and cryptography audit.
2. Publish a threat model covering servers, operators, endpoints, metadata, recovery, and downgrade attacks.
3. Add cross-runtime, multi-tab, and multi-device end-to-end coverage.
4. Add fault injection for interrupted uploads, duplicates, corrupt ciphertext, missing envelopes, expired credentials, conflicts, and partial deployments.
5. Automate parity checks across schemas, migrations, registries, and sync rules.
6. Define supported mixed-version and rollback windows.
7. Document and test backup, restore, disaster recovery, device loss, and incident response.
8. Decide whether key rotation and post-revocation re-encryption are required by the threat model.
9. Publish an inventory of encrypted fields and visible metadata.
10. Replace or continuously validate reliance on private PowerSync APIs.

For a pilot, use non-critical data, keep independent backups, test recovery, restrict infrastructure access, and treat E2EE as defense in depth rather than an audited guarantee.
