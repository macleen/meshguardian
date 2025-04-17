# MeshGuardian Overview

Welcome to **MeshGuardian**, a resilient communication system built for unstable, offline, or decentralized environments — from disaster zones to rural IoT to digital legacies.

This document gives you a high-level introduction to the purpose, vision, and modular design of the MeshGuardian system. Whether you're a contributor, tester, researcher, or humanitarian technologist — you're in the right place.

---

## What Is MeshGuardian?

**MeshGuardian** is an open framework for building secure, delay-tolerant, and protocol-agnostic communication networks that operate in hostile or disconnected environments.

It combines:
- 📦 Custom packet architecture with secure headers
- 🌐 Delay-Tolerant Networking (DTN) logic
- 🔐 Post-quantum-ready encryption and authentication
- 🧠 AI-guided journaling and memory preservation (future-facing)
- ⚙️ Hardware integration for edge devices (e.g., STM32WL, Raspberry Pi)

---

## Why It Exists

Traditional internet-based communication fails in:

- ⚠️ Natural disasters (earthquakes, floods, etc.)
- 🔇 War zones and censored regions
- 🌌 Space and planetary communication
- 📡 Remote communities without internet

MeshGuardian aims to **bridge that gap** by enabling secure, store-and-forward communication in such conditions — with or without centralized infrastructure.

---

## Core Components

| Folder | Purpose |
|--------|---------|
| `/pseudo-code/` | Language-agnostic specification of all modules |
| `/hardware/` | Build and deploy your own MeshGuardian node |
| `/security/` | Cryptographic foundation: keys, encryption, authentication |
| `/networking/` | Core packet creation, sending, buffering, routing |
| `/monitoring/` | Node health checks and diagnostics |
| `/protocol/` | Profiles, selectors, and multi-protocol support |
| `/exceptions/` | Custom error classes for modular robustness |

---

## What Makes MeshGuardian Unique?

✅ Delay-Tolerant by Design  
✅ Built-In Key Management + Forward Secrecy  
✅ Modular & Extensible (LoRa, BLE, Zigbee, TCP/IP)  
✅ Inspired by real-world humanitarian and edge-tech needs  
✅ Openly documented, testable, and auditable

---

## What’s Next?

Choose your path:

- 🔧 **[Hardware Setup](2_hardware_quickstart.md)** → Build a node with Raspberry Pi or STM32WL
- 💻 **[Firmware Setup](3_firmware_quickstart.md)** → Flash the core MeshGuardian logic
- 📡 **[Protocol Simulation](4_protocol_demo.md)** → Run a minimal demo of packet routing
- 🔐 **[Security Introduction](5_security_basics.md)** → Understand our approach to cryptographic safety

---

## Questions?

See [faq.md](faq.md) or [open an issue](https://github.com/macleen/meshguardian/issues) on GitHub.

> “Technology that fails in a crisis is just privilege disguised as progress. MeshGuardian is built for those who can't afford failure.”  
> — The Team

---
