# 🛡️ MeshGuardian

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)

**Resilient, secure, and modular mesh networking protocol for Delay-Tolerant Networks (DTNs), disaster zones, and interplanetary systems.**

[Full documentation available at https://meshguardian.com](https://meshguardian.com/docs)

MeshGuardian is a next-generation protocol stack designed to keep communication alive—even when everything else fails. Built for the harshest environments, from offline villages and war zones to deep space networks, it brings trust, synchronization, and performance to where traditional internet protocols fall apart.

---

![MeshGuardian vs. Others](/docs/assets/mesh_comparison.webp)

## Key Features

- **Delay-Tolerant Synchronization**  
  Reliable nano-sync, offline buffering, stream ordering, and audit-backed consistency across disconnected nodes.

- **End-to-End Security**  
  AES-256, Schnorr signatures, optional post-quantum crypto (Kyber/Dilithium), blockchain-based audit trails.

- **Adaptive Consensus Engine**  
  Hybrid PoS/PBFT validation based on network size, energy mode, and data priority.

- **Pluggable Protocol Engine**  
  Swap in new transport, encryption, or logging modules at runtime without reconfiguration.

- **Energy-Aware & Edge-Ready**  
  Designed for IoT, drones, smart dust, and low-power embedded nodes.

- **Multi-Cluster & Cross-Region Support**  
  Routes data securely across isolated clusters using relay-aware packet composition.

---

## Comparison with Other Protocols

MeshGuardian stands out among DTN and mesh networking protocols due to its unique blend of security, adaptability, and efficiency. Below are comparisons with other popular protocols:

### Comparison Candidates (Popular DTN/Mesh Protocols)

| **Protocol**            | **Type**            | **Used By / Designed For**         |
|--------------------------|---------------------|------------------------------------|
| Bundle Protocol (BPv7)   | DTN                 | NASA / ESA missions               |
| B.A.T.M.A.N.            | Mesh routing        | OpenWRT / community mesh          |
| OLSR                    | Proactive mesh      | MANETs, mobile ad hoc             |
| Epidemic Routing        | DTN                 | Academic networks / UAVs          |
| SCION                   | Path-aware internet | Future secure routing             |
| LoRa Mesh               | LPWAN               | IoT, remote sensors               |
| MeshGuardian            | DTN + Secure Mesh   | Disaster zones, edge AI, space ops |

### Capability Comparison

| **Capability**              | **MeshGuardian** | **BPv7** | **B.A.T.M.A.N.** | **OLSR** |
|-----------------------------|------------------|----------|------------------|----------|
| Works offline (DTN)         | ✅               | ✅       | ❌               | ❌       |
| Blockchain audit support    | ✅               | ❌       | ❌               | ❌       |
| Pluggable consensus engine  | ✅               | ❌       | ❌               | ❌       |
| AI-enhanced routing/sync    | ✅               | ❌       | ❌               | ❌       |
| Cryptographic agility (PQC) | ✅               | ⚠️ (RSA) | ❌               | ❌       |
| Language-agnostic bindings  | ✅               | ❌       | ❌               | ❌       |
| Synchronization mechanisms  | ✅               | ✅       | ❌               | ❌       |
| Energy-aware optimization   | ✅               | ❌       | ❌               | ❌       |
| Stream ordering support     | ✅               | ⚠️       | ❌               | ❌       |
| Fully decentralized         | ✅               | ⚠️       | ✅               | ⚠️       |
| Cross-cluster awareness     | ✅               | ❌       | ❌               | ❌       |

- ✅ = Supported
- ⚠️ = Partially / with limitations
- ❌ = Not supported

---

## Use Cases

- Emergency communications in disaster response zones
- Interplanetary data relays between ground stations and satellites
- Secure voting and audit logging in off-grid governance systems
- Peer-to-peer coordination in rural or conflict-stricken regions
- Wildlife or environmental sensor swarms across vast terrains

---

## Protocol Architecture
The protocol has been updated to the 64-bit version, which no longer uses the separate "Header", "Payload", and "Trailer" structure from the old 32-bit version. The new architecture integrates these elements for improved efficiency and performance.

Learn more in protocol-specs/


## Multi-Language SDKs

MeshGuardian supports SDKs in many popular languages—each with its own entry point in the `/bindings` directory:

/bindings/  
├── c/  
├── csharp/  
├── python/  
├── java/  
├── go/  
├── javascript/  
├── pseudo-code/  ( All the blue prints of the necessary code )  
├── rust/  
  
  
Each SDK is modular, independently testable, and installable via its native package manager.
NB: The MeshGuardian codebase is currently empty but in the process of being constructed, and with your support as a contributor, it won’t take long to bring it to life.

---


## Contributing

We welcome contributors across languages, platforms, and domains.

> All contributors must sign the [Contributor License Agreement (CLA)](docs/CLA.md) before submitting PRs.

### Repo Setup

```bash
git clone https://github.com/macleen/meshguardian.git
cd meshguardian