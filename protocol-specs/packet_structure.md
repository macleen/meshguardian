# MeshGuardian Packet Structure

The MeshGuardian protocol uses a modular, secure, and extensible packet structure designed for Delay-Tolerant Networks (DTNs) and challenging connectivity environments. The structure leverages 64-bit capability flags for protocol modularity (see `capability_flags.md`).

---

## Packet Segments

| Segment   | Size                     | Description |
|-----------|--------------------------|-------------|
| **Header**    | 21 bytes + variable CBOR | Routing, synchronization, control, encryption flags, ZKP, and plugin metadata |
| **Payload**   | Variable                 | Compressed and encrypted application data |
| **Trailer**   | 96 bytes                 | Signature + Audit Trail Hash for integrity and traceability |

> Example total packet size: **384 bytes**  
> (e.g., 21B fixed header + 0B CBOR + 267B payload + 96B trailer, with lz4/zstd compression)

---

## Header (21 bytes + variable CBOR)

Includes:
- `Version` (1B), `Packet Type` (1B), `Priority Flag` (1B, Bit 16 for high-priority)
- `Capability Flags` (8B, e.g., Bit 23 for Low-Energy Mode, Bit 36 for extended compression, Bit 39 for Quantum Mode)
- `Protocol ID` (1B), `Consensus Mode Flag` (1B, `0x01` = PoS, `0x02` = PBFT, `0x03` = ADTC)
- `Source Node ID`, `Destination Node ID`, `Relay ID` (3 × 16B, in CBOR)
- `Sequence Number` (4B), `Timestamp` (8B), `Vector Clock` (8B, in CBOR)
- `Hop Count` (1B), `TTL` (1B), `Compression Type` (1B, e.g., lz4, zstd, zlib, Brotli)
- `Encryption Algorithm ID` (1B, e.g., AES-256, Kyber, Dilithium)
- `Plugin Data` (4B, in CBOR), `ZKP Proof` (16B, zk-SNARKs/Bulletproofs, in CBOR)
- `Energy Mode` (1B), `Congestion Flags` (1B), `FEC Flags` (1B)
- `Checksum` (4B)

---

## Payload (Variable Size)

The payload includes:
- Encrypted JSON or binary message (e.g., AES-256, Kyber, enabled via Bit 39)
- Optional Forward Error Correction (FEC) block (Reed-Solomon)
- Optional PrivacySafePod for field-level encryption (inner layer)

Payload is compressed (e.g., lz4, zstd, zlib, Brotli via Bit 36) **before** encryption.

---

## Trailer (96 bytes)

- **Signature (64B):** Ensures authenticity (e.g., Schnorr, Ed25519, Dilithium via Bit 39)
- **Audit Trail Hash (32B):** Used for local + blockchain logging (e.g., Solana Tier 1 via Bit 37)

---

## Security & Trust

Each packet is validated using:
- Sequence Number + Timestamp + Vector Clock for deduplication and replay protection
- ZKP Proof for zero-knowledge trust verification (e.g., zk-SNARKs, enabled via Bit 39)
- Signature for zero-trust validation
- Blockchain sync via Tier 1 audit events (Bit 37) with log compression
- Audit Trail Hash ensures traceability across local, P2P, and blockchain logs

---

## See Also

- [consensus_engine.md](./consensus_engine.md)
- [synchronization.md](./synchronization.md)
- [capability_flags.md](./capability_flags.md)
- [hardware_requirements.md](./hardware_requirements.md)
- [CONTRIBUTING.md](./CONTRIBUTING.md)

---

## TODO
- Map legacy 32-bit flags (e.g., `BIT_13` for TinyML) to the 64-bit structure or deprecate them.