# MeshGuardian Message Processing Cycle Specification

## Overview
MeshGuardian is a secure, decentralized protocol for message validation, compression, consensus, and auditing in constrained mesh and Delay-Tolerant Networking (DTN) environments. The protocol consists of a modular, auditable message lifecycle spanning 10 core steps and 8 proposed extensions. This document defines each phase, its purpose, and implementation responsibilities.


## Core Steps (1–10)


### Step 1: Node Initialization
- Loads constants and dynamic configuration (JSON/YAML).
- Generates `Node_ID`, `Session_ID`, and utility functions (e.g., RTT estimation, SHA3 hashing).
- Authenticates credentials from Secure Credential Storage.
- Verifies hardware compatibility and entropy sources.

### Step 2: Replay Attack & Downgrade Protection
- Validates timestamp/vector clocks, nonces, and sequence numbers.
- Checks protocol version, encryption/compression strength, and consensus mode.
- Tracks and logs downgrade attempts, applies IP lockouts.

### Step 3: Message Received
- Parses incoming message structure (Header, Payload, Trailer).
- Verifies routing, signature integrity, TTL, and capability flags.
- Buffers chunked fragments and logs anomalies.

### Step 4: Compression
- Decompresses incoming payloads or compresses outgoing ones.
- Selects optimal algorithm (lz4, zstd, Brotli, etc.).
- Validates algorithm security and handles chunked fragments.

### Step 5: Encryption
- Decrypts or encrypts message layers (outer, inner PrivacySafePod).
- Validates cryptographic strength and `QUANTUM_FLAG`.
- Uses AES-256, Kyber, Dilithium, or ChaCha20 based on policy.

### Step 6: ZKP (Zero-Knowledge Proof)
- Verifies or generates zk-SNARKs, Bulletproofs, or zk-STARKs.
- Validates proof scheme and logs events.
- Uses TinyML model for low-energy optimization (optional).

### Step 7: Consensus
- Executes PBFT, PoS, or ADTC protocol depending on `consensus_mode`.
- Validates consensus rules, ZKP, signature, and message eligibility.
- Applies trust decay and slashing on malicious behavior.

### Step 8: PBFT Validation
- Coordinates PrePrepare, Prepare, and Commit phases.
- Produces `consensus_certificate` if quorum is met (2f+1).
- Logs results and slashes malicious validators.

### Step 9: Audit Trail
- Logs message outcome to blockchain, local ledger, or encrypted buffer.
- Supports log compression, `retry_policy`, and P2P sync.
- Logs with `BIT_15` (Tier 1), `BIT_16` (downgrade alert), etc.

### Step 10: End: Message Processed
- Cleans up memory and buffers, rotates session keys.
- Emits ACKs, relays results, and collects metrics.
- Prepares for next message or enters sleep mode (`BIT_24`).

## Proposed Extensions (Steps 11–18)

### Step 11: Network Operation and Maintenance*
- Manages ongoing message processing and network sync.
- Tracks queue, syncs vector clocks, and trust state.

### Step 12: System Monitoring and Optimization*
- Analyzes metrics and anomalies.
- Optimizes parameters and updates trust graph.

### Step 13: Adaptive Network Evolution*
- Updates TinyML models.
- Predicts and mitigates anomalies, refines trust models.

### Step 14: Proactive Threat Mitigation and Recovery*
- Isolates malicious nodes and coordinates threat response.
- Reconfigures network security parameters.

### Step 15: Network Analytics and Reporting*
- Generates network reports and shares insights with stakeholders.
- Validates and signs reports via Consensus.

### Step 16: Network Integration and Scalability*
- Onboards nodes and allocates resources.
- Validates cross-network messages and integrates with other systems.

### Step 17: Cross-Network Federation and Governance*
- Enforces federated governance across trust zones.
- Coordinates consensus and routing with external networks.

### Step 18: Continuous Learning and Protocol Evolution*
- Aggregates feedback and community contributions.
- Applies protocol upgrades and publishes signed versions.

_*Note: Proposed extensions are under review and not yet implemented._

## Capability Flags Summary

| Flag      | Bit       | Meaning                              |
|-----------|-----------|--------------------------------------|
| `BIT_1`   | `0x01`    | Broadcast                            |
| `BIT_2`   | `0x02`    | Unicast                              |
| `BIT_3`   | `0x04`    | Multi-hop Relay                      |
| `BIT_4`   | `0x08`    | Chunked Fragment                     |
| `BIT_5`   | `0x10`    | Secure Negotiation                   |
| `BIT_6–10`| -         | Compression/Encryption Algorithm IDs |
| `BIT_13`  | `0x1000`  | TinyML Optimization                  |
| `BIT_15`  | `0x8000`  | Tier 1 / High-Severity Audit Event   |
| `BIT_16`  | `0x10000` | Downgrade Attack Detected (Alert)    |
| `BIT_24`  | `0x1000000` | Low-Energy Mode                    |

## Testing & CI
- `test_message_cycle.py` covers all 10 core steps.
- Extended steps tested in `test_extensions.py`.
- Run tests with `python -m unittest` and GitHub Actions CI.
- Hardware targets: Raspberry Pi 4 and Zero 2W.

## Future Integration
- Incorporate Steps 11–18 via updated pseudocode and test suites.
- Update `CONTRIBUTING.md` to include community-owned extension modules.
- Maintain protocol compatibility with audit-led upgrade paths.

## Status
- **Core Steps 1–10**: ✅ Fully defined and implemented.
- **Extension Steps 11–18**: ⚠️ Proposed, under review for phased rollout.

## Maintainer Notes
- Contributions should follow audit tagging (`BIT_15`, `BIT_16`).
- Document all new flags, metrics, and hashes in `glossary.md`.
- Ensure backward compatibility when evolving consensus, ZKP, or compression modules.