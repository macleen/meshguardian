
# MeshGuardian Packet Structure

The MeshGuardian protocol uses a modular, secure, and extensible packet structure designed for Delay-Tolerant Networks (DTNs) and challenging connectivity environments.

---

## Packet Segments

| Segment   | Size         | Description |
|-----------|--------------|-------------|
| **Header**    | 128 bytes    | Routing, synchronization, control, encryption flags, and plugin metadata |
| **Payload**   | Variable     | Compressed and encrypted application data |
| **Trailer**   | 96 bytes     | Signature + Audit Trail Hash for integrity and traceability |

> Example total packet size: **491 bytes**  
> Payload: 267B (based on compression)

---

## Header (128 bytes)

Includes:
- `Version` (1B), `Packet Type` (1B), `Priority` (1B)
- `Capability Flags` (4B), `Protocol ID` (1B)
- `Source Node ID`, `Destination Node ID`, `Relay ID` (3 × 16B)
- `Sequence Number` (4B), `Timestamp` (8B)
- `Hop Count`, `TTL`, `Compression Type`, `Encryption Algorithm ID`
- `Plugin Data` (4B), `Biometric Hash` (32B)
- `Energy Mode`, `Congestion Flags`, `FEC Flags` + `Checksum` (4B)
- `Reserved` (2B padding for alignment)

---

## Payload (Variable Size)

The payload includes:
- Encrypted JSON or binary message
- Optional Forward Error Correction (FEC) block
- Optional redacted fields (PrivacySafePod) for field-level encryption

Payload is compressed (e.g., zlib, Brotli) **before** encryption.

---

## Trailer (96 bytes)

- **Signature (64B):** Ensures authenticity (e.g., Schnorr)
- **Audit Trail Hash (32B):** Used for local + blockchain logging (e.g., Solana Tier 1)

---

## Security & Trust

Each packet is validated using:
- Sequence Number + Timestamp for deduplication
- Signature for zero-trust validation
- Optional blockchain sync via Tier 1 audit events

---

## See Also

- [consensus_engine.md](./consensus_engine.md)
- [synchronization.md](./synchronization.md)
