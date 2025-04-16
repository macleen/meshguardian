
# MeshGuardian Consensus Engine

The MeshGuardian Consensus Engine ensures decentralized agreement on packet validation, blockchain-backed audit logging, and trust propagation—tailored for Delay-Tolerant Networks (DTNs), disaster zones, and interplanetary networks.

---

## Purpose

- Establish global trust in decentralized environments
- Validate packets under zero-trust conditions
- Trigger audit trail writes on consensus events
- Adapt consensus rigor to context (e.g., energy or mission-criticality)

---

## Supported Consensus Mechanisms

### Proof-of-Stake (PoS)

- Used for low-priority packets (default)
- Energy-efficient, ideal for IoT and remote nodes
- Leader election based on stake and signal strength

### Practical Byzantine Fault Tolerance (PBFT)

- Used for high-priority packets (`Bit 3 = 1`)
- Ensures strong agreement across a quorum of nodes
- Used in critical infrastructure and safety-of-life deployments

### Hybrid Mode

- Toggles between PoS and PBFT based on:
  - `consensus_mode_flag`
  - Network density or energy flags
  - Packet priority bits

---

## Validation Flow

1. **Packet arrives** → Header is parsed.
2. **Signature checked** → If valid, continue.
3. **Consensus mode triggered**
   - If PoS → validate via local stake/role
   - If PBFT → initiate quorum round
4. **Audit Trail Logging**
   - Tier 1 events → blockchain (e.g., Solana)
   - Tier 2 events → local + P2P

---

## Packet Fields Involved

- `consensus_mode_flag`: indicates PoS, PBFT, or hybrid
- `pbft_leader_id`: used during quorum rounds
- `priority`: triggers escalation to PBFT
- `capability_flags`: determines consensus plugins enabled

---

## Conflict Resolution

- Uses sequence numbers and logical timestamps (Lamport)
- Deterministic fork resolution with lowest hash + quorum sync

---

## Security Features

- Zero-trust validation model
- Quantum-ready signature handling
- Optional post-quantum consensus plugin (future)

---

## Related

- [packet_structure.md](./packet_structure.md)
- [synchronization.md](./synchronization.md)
