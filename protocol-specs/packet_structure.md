# MeshGuardian Packet Structure

The MeshGuardian protocol uses a modular, secure, and extensible packet structure designed for Delay-Tolerant Networks (DTNs) in challenging connectivity environments. The 64-bit version integrates routing, synchronization, control, consensus metadata, encryption, and audit elements into a unified structure, replacing the 32-bit version's header-payload-trailer segmentation for improved efficiency. It supports ultra-low-resource devices (<32 KB RAM) with Low-Energy Mode and advanced features like Adaptive Delay-Tolerant Consensus (ADTC) and quantum-safe cryptography. For detailed specifications, refer to the [MeshGuardian documentation](https://meshguardian.org/docs).

---

## Packet Structure Overview

The MeshGuardian packet consists of:
- **Fixed Header** (21 bytes): Core metadata for routing, synchronization, and consensus.
- **Variable CBOR Header**: BPv7-compliant metadata for extended features (e.g., compression, encryption).
- **Variable Payload**: Compressed and encrypted application data, with optional error correction.
- **Dynamic Security/Audit Elements**: Embedded signatures and audit hashes for integrity and traceability.

> Example total packet size: ~491 bytes (21B fixed header + variable CBOR + ~350B payload with Snappy compression, reduced to ~340B in use case 5.17).

---

## Fixed Header

The fixed 21-byte header contains essential metadata, unencrypted for intermediate node processing.

| Field            | Size    | Description |
|------------------|---------|-------------|
| Flags Version    | 1 byte  | Defines Capability Flags structure version |
| Capability Flags | 8 bytes | Protocol version and feature flags (e.g., CBOR, AES-256, Quantum Mode) |
| Consensus Mode   | 1 byte  | Consensus algorithm (e.g., PBFT, ADTC) |
| Hop Count        | 2 bytes | Tracks packet hops |
| TTL              | 4 bytes | Time-to-live (e.g., 3600s for terrestrial) |
| Vector Clock     | 4 bytes | Ensures message consistency |
| Custody Identifier | 1 byte | Supports custody transfer |

---

## Variable CBOR Header

A variable-length, BPv7-compliant CBOR header (minimum 4 bytes) encodes additional metadata, such as source/destination and extended features (e.g., Snappy compression, Solana blockchain logging).

---

## Variable Payload

The payload includes:
- CBOR-encoded, compressed (e.g., Snappy, zstd), and encrypted (e.g., AES-256, Kyber) application data.
- Optional PrivacySafePod for anonymized data.
- Optional Forward Error Correction (FEC) for reliability.

Compression precedes encryption for size optimization.

---

## Dynamic Security and Audit

Security and audit elements are embedded dynamically:
- **Signatures**: Ensure authenticity (e.g., Dilithium for quantum-safe cryptography).
- **Audit Trail Hash**: SHA3-256 for local and blockchain logging (e.g., Solana).

---

## Security & Trust

Packets are validated using:
- Vector Clock for deduplication and ordering
- Zero-knowledge proofs for privacy
- Signatures for zero-trust validation
- Audit trail hashes for traceability

---

## Capability Flags

The 64-bit Capability Flags define supported features:
- **Bits 0–4**: Protocol versioning (32 versions).
- **Bits 5–12**: Core features (e.g., CBOR, LoRa, AES-256).
- **Bits 13–39**: Optional features (e.g., Snappy compression, Quantum Mode), disabled in Low-Energy Mode.
- **Bits 40–63**: Reserved for future use.

For a complete list, see the [MeshGuardian documentation](https://meshguardian.org/docs).

---

## Processing Flow

- **Sender**: Encodes, compresses, encrypts, and signs the packet, adding metadata and audit hashes.
- **Relay**: Validates header, updates hop count and vector clock, and forwards using selected protocol.
- **Receiver**: Verifies signatures, processes FEC, decompresses, decrypts, and handles data (e.g., ADTC votes).

![Processing Flow](/docs/assets/processing-flow.webp)

---

## Considerations

- **Extensibility**: Supports new features via Capability Flags and CBOR sub-flags.
- **Performance**: Optimized for <16 KB RAM with a 21-byte header.
- **Energy Efficiency**: Low-Energy Mode disables advanced features for ultra-low-power devices.
- **Interplanetary Support**: ADTC and extended TTL handle 15–30 minute RTTs.

For detailed field definitions, CBOR sub-flags, and implementation notes, visit the [MeshGuardian documentation](https://meshguardian.org/docs).

---

## See Also

- [consensus_engine.md](./consensus_engine.md)
- [synchronization.md](./synchronization.md)
- [capability_flags.md](./capability_flags.md)
- [hardware_requirements.md](./hardware_requirements.md)
- [CONTRIBUTING.md](./CONTRIBUTING.md)

---

## TODO
- Map legacy 32-bit flags to the 64-bit structure or deprecate them.
- Define additional CBOR sub-flags for future features (Bits 40–63).
