# MeshGuardian Synchronization Mechanism

MeshGuardian ensures data consistency across Delay-Tolerant Networks (DTNs) through a robust, energy-aware synchronization engine. This system adapts to scenarios where nodes may be offline for extended periods or connected only intermittently.

---

## Purpose

- Prevent data loss and duplication in disconnected networks
- Enable causal and temporal ordering across async nodes
- Maintain audit integrity via hash-consistent event chains
- Operate in both terrestrial and interplanetary DTNs

---

## Key Features

- **Nano-Sync Packets**  
  Minimal overhead sync messages (~10 bytes: 4B sequence, 1B flag, 5B metadata) for extreme low-power operation.

- **Event Hashing + Audit Chain**  
  All events hashed and stored for traceability. Critical entries anchored to Solana, Avalanche, or Ethereum with log compression.

- **Vector Clocks & Lamport Timestamps**  
  Vector clocks extend Lamport timestamps for async systems (e.g., Mars base vs. Earth).

- **Conflict Resolution**  
  Merges conflicting updates via vector clock detection + lowest hash arbitration (AI plugins planned).

- **Offline Buffering**  
  Nodes store unsent packets with TTL (decremented per hop) until reconnected, up to `MAX_BUFFER_PERSISTENCE` (30 days).

---

## Use Cases

- First responders syncing bodycam footage in disaster zones
- Mars rover syncing telemetry to Earth during blackout windows
- Remote wildlife sensors buffering environmental data for weeks
- Refugee camp nodes syncing supply chain data in low-connectivity areas

---

## Relevant Packet Fields

| Field              | Size | Description |
|-------------------|------|-------------|
| `sequence_number` | 4B   | For ordering + deduplication, prevents replay attacks |
| `timestamp`       | 8B   | Logical or real-time clock |
| `vector_clock`    | 8B   | Tracks causal dependencies for async sync |
| `hop_count`       | 1B   | Tracks number of relays |
| `ttl`             | 1B   | Time-to-live; decrements per hop |
| `nano_sync_flag`  | 1B   | Activates lightweight sync mode (`BIT_18`) |
| `audit_trail_hash`| 32B  | Hash of previous event in chain |

---

## Audit + Integrity

- Tier 1: Critical events (`BIT_15`) → Blockchain (Solana, Avalanche, Ethereum) with log compression
- Tier 2: Local log + P2P sync
- Sync checkpoints signed with Schnorr or Dilithium signatures
- ZKP verification for audit events (e.g., zk-SNARKs, Bulletproofs)

---

## Plugin-Enhanced Sync

- AI-powered merge arbitration (proposed, see `CONTRIBUTING.md`)
- Redundancy-aware multi-path re-sync
- Burst-throttle control for congestion collapse

---

## Related Specs

- [packet_structure.md](./packet_structure.md)
- [consensus_engine.md](./consensus_engine.md)
- [hardware_requirements.md](./hardware_requirements.md)
- [CONTRIBUTING.md](./CONTRIBUTING.md)