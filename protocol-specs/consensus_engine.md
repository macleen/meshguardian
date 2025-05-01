# MeshGuardian Consensus Engine

The MeshGuardian Consensus Engine ensures decentralized agreement on packet validation, blockchain-backed audit logging, and trust propagation—tailored for Delay-Tolerant Networks (DTNs), disaster zones, and interplanetary networks.

---

## Purpose

- Establish decentralized trust in zero-trust environments
- Validate packets under constrained network conditions
- Trigger audit trail writes for consensus events
- Adapt consensus rigor to context (e.g., energy constraints, mission-criticality)

---

## Supported Consensus Mechanisms

### Proof-of-Stake (PoS)

- Used for low-priority packets (default)
- Energy-efficient, ideal for IoT and remote nodes
- Leader election based on stake and signal strength (RSSI)

### Practical Byzantine Fault Tolerance (PBFT)

- Used for high-priority packets (`BIT_17 = 1`)
- Ensures strong agreement across a quorum of nodes (2f+1)
- Used in critical infrastructure and safety-of-life deployments

### Asynchronous Delay-Tolerant Consensus (ADTC)

- Used for interplanetary networks (15–30 minute RTTs)
- Handles high-latency DTN with vector clocks and buffering
- Ideal for Mars rover relays (Use Case 5.1.1)

### Hybrid Mode

- Toggles between PoS, PBFT, and ADTC based on:
  - `consensus_mode_flag`
  - Network density, RTT estimates, or energy flags (`BIT_24`)
  - Packet priority (`BIT_17`)

---

## Validation Flow

1. **Packet Arrives** → Header is parsed.
2. **Signature Checked** → If valid, continue.
3. **ZKP Validation** → Verify zk-SNARKs, Bulletproofs, or zk-STARKs (if enabled).
4. **Sequence Number Validation** → Check sequence numbers to prevent replay attacks.
5. **Consensus Mode Triggered**
   - If PoS → Validate via local stake/role and RSSI
   - If PBFT → Initiate PrePrepare, Prepare, Commit phases
   - If ADTC → Buffer votes until quorum
6. **Audit Trail Logging**
   - Tier 1 events (`BIT_15`) → Blockchain (e.g., Solana, Avalanche, Ethereum) with log compression
   - Tier 2 events → Local ledger + P2P sync

---

## Packet Fields Involved

- `consensus_mode_flag`: Indicates mode (`0x01` = PoS, `0x02` = PBFT, `0x03` = ADTC, `0x04` = Hybrid)
- `pbft_leader_id`: 32-byte identifier for PBFT leader node
- `priority_flag`: Triggers PBFT for high-priority packets (`BIT_17 = 1`)
- `capability_flags`: Determines enabled plugins (e.g., `BIT_13` for TinyML, `BIT_24` for Low-Energy Mode)
- `sequence_number`: Prevents replay attacks, validated per sender

---

## Conflict Resolution

- Uses sequence numbers and logical timestamps (Lamport/vector clocks)
- Deterministic fork resolution with lowest hash + quorum sync
- ADTC buffers votes for up to `MAX_BUFFER_PERSISTENCE` (30 days)

---

## Security Features

- Zero-trust validation model
- Quantum-ready signature handling (e.g., Kyber, Dilithium)
- Replay protection via sequence numbers and slashing (`MALICIOUS_VOTE_THRESHOLD`)
- Proposed post-quantum consensus plugin (see `CONTRIBUTING.md`)

---

## Hardware Requirements

- PoS: 16 MHz, 32 KB, 100 µW (Raspberry Pi Zero 2W)
- PBFT: 80 MHz, 128 KB, 500 µW (Raspberry Pi 4)
- ADTC: 32 MHz, 64 KB, 200 µW (Raspberry Pi 4 or Zero 2W)
- Hybrid: Dynamic, up to 80 MHz, 128 KB (see `hardware_requirements.md`)

---

## Related

- [packet_structure.md](./packet_structure.md)
- [synchronization.md](./synchronization.md)
- [hardware_requirements.md](./hardware_requirements.md)
- [CONTRIBUTING.md](./CONTRIBUTING.md)