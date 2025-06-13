# Hardware Requirements for MeshGuardian

## Purpose
This document centralizes the hardware requirements for deploying **MeshGuardian nodes** and running its software modules, ensuring compatibility across diverse environments, from low-power IoT devices in rural areas to high-capability servers in interplanetary scenarios. It combines:
- **Node-level hardware components** (e.g., Raspberry Pi, LoRa HAT, power supply) for building a MeshGuardian node in a mesh network.
- **Module-specific computational requirements** (CPU, RAM, power consumption) for running MeshGuardian software modules, supporting features like adaptive compression, zero-knowledge proofs (ZKP), Asynchronous Delay-Tolerant Consensus (ADTC), and multi-blockchain logging.

This consolidated approach facilitates deployment planning for field engineers, developers, and system integrators, addressing both physical hardware and software resource needs.

## Scope
The document covers:
- **Node Hardware**: Physical components for a MeshGuardian node, including Raspberry Pi models, wireless modules, power supplies, and optional accessories, as specified in the original `1_hardware_requirements.md`.
- **Module Requirements**: Computational requirements (CPU, RAM, power) for software modules (Audit Trail, Compression, Consensus, etc.) from `pseudo_code-merged.txt`, mapped in a structured table.

## Node Hardware Requirements

### Required Components
| Component                  | Specification                                                                 | Notes                                                                 |
|---------------------------|------------------------------------------------------------------------------|----------------------------------------------------------------------|
| **Raspberry Pi**          | Recommended: Raspberry Pi 4 (high-performance tasks, multi-hop routing). Alternative: Raspberry Pi Zero 2W (low-power, lightweight setups). | Use Raspberry Pi 4 for complex routing or full nodes; Zero 2W for energy-efficient light nodes. |
| **Power Supply**          | Raspberry Pi 4: USB-C adapter (5V, 3A). Raspberry Pi Zero 2W: Micro-USB adapter (5V, 2.5A). | Use certified adapters to avoid instability or hardware damage.      |
| **MicroSD Card**          | Class 10, 16GB minimum (32GB+ recommended). High-quality brands (e.g., Samsung, SanDisk, Kingston). | Larger capacity for logs, updates, and caching. Prioritize durability. |
| **Wireless Module**       | LoRa HAT or compatible module (e.g., Zigbee, BLE). Ensure Raspberry Pi compatibility. | LoRa for long-range, low-power; Zigbee/BLE for short/medium-range. Verify software stack compatibility. |
| **Case**                  | Protective enclosure for Raspberry Pi and wireless hardware.                  | Protects against dust, shock, moisture in outdoor/field deployments.  |
| **Battery Pack (Optional)** | High-capacity (10,000mAh+), USB battery bank or solar solution. Stable output voltage. | Essential for off-grid, mobile, or emergency deployments. Solar charging ideal for remote areas. |

### Node Hardware Notes
- **Performance vs. Efficiency**: Raspberry Pi 4 (1.5 GHz quad-core, 2-8GB RAM) supports full nodes with intensive tasks (e.g., PBFT, multi-blockchain logging). Raspberry Pi Zero 2W (1 GHz quad-core, 512MB RAM) is suited for light nodes in Low-Energy Mode (Bit 23).
- **Wireless Selection**: Choose based on range (LoRa: up to 10km, Zigbee: 100-300m, BLE: 100m), bandwidth, and power needs. LoRa is preferred for disaster zones or rural deployments (Use Case 5.15).
- **Power Stability**: Ensure stable voltage to prevent brownouts, especially for battery-powered nodes in remote areas (e.g., solar panels for interplanetary relays, Use Case 5.1.1).
- **Field Deployment**: Cases must withstand environmental challenges (e.g., IP65 rating for dust/moisture). Battery packs with solar charging enhance sustainability in off-grid scenarios.

## Module Computational Requirements

### Table of Module Requirements
| Module                     | Minimum CPU       | Minimum RAM | Power Consumption | Notes                                                                 |
|----------------------------|-------------------|-------------|-------------------|----------------------------------------------------------------------|
| **Audit Trail**            | 32 MHz            | 64 KB       | 200 µW            | Supports multi-blockchain logging (Solana, Avalanche, Ethereum) and lightweight ledger (Bit 37). Requires moderate resources for cryptographic hashing (`crypto_utils.sha3_256`) and P2P sync. Scales to 1MB ledger (`LIGHTWEIGHT_LEDGER_MAX_SIZE`). |
| **Compression**            | 16 MHz (lz4)      | 32 KB (lz4) | 100 µW (lz4)      | Supports lz4, zstd, TinyML, zlib, Brotli (Bit 36). lz4 is lightest (16 MHz, 32 KB, 100 µW); zstd needs 32 MHz, 64 KB, 200 µW; TinyML needs 80 MHz, 128 KB, 500 µW with TensorFlow Lite Micro. Low-power devices prioritize lz4 in Low-Energy Mode (Bit 23). |
| **Consensus (ADTC)**       | 32 MHz            | 64 KB       | 200 µW            | Handles PoS, PBFT, ADTC. ADTC requires vector clocks and RTT estimation for interplanetary scenarios (15-30 min RTTs). Moderate resources suffice for PoS; PBFT/ADTC need more for vote buffering (`MAX_BUFFER_PERSISTENCE`). |
| **PBFT Validation**        | 16 MHz (hash-based) | 32 KB (hash-based) | 100 µW (hash-based) | Supports full PBFT (80 MHz, 128 KB, 500 µW) and simplified hash-based validation (16 MHz, 32 KB, 100 µW) for low-power devices. ADTC fallback requires 32 MHz, 64 KB, 200 µW. |
| **PoS Validation**         | 16 MHz (hash-based) | 32 KB (hash-based) | 100 µW (hash-based) | Similar to PBFT, supports hash-based validation for low-power devices. Full PoS with stake-weighted quorums needs 32 MHz, 64 KB, 200 µW. ADTC fallback matches PBFT requirements. |
| **Zero-Knowledge Proofs (ZKP)** | 16 MHz (hash-based) | 32 KB (hash-based) | 100 µW (hash-based) | Supports Bulletproofs (32 MHz, 64 KB, 200 µW), zk-SNARKs (80 MHz, 128 KB, 500 µW), zk-STARKs (120 MHz, 256 KB, 1 mW). Hash-based proofs (SHA3-256) for ultra-low-power devices in Low-Energy Mode (Bit 23). Post-quantum options via Bit 39. |
| **Secure Credential Storage** | 32 MHz            | 64 KB       | 200 µW            | Requires encryption/decryption (`crypto/encryption.md`) and secure vault storage. Moderate resources for AES-256 and key management. Aligns with NIST SP 800-63B, PCI DSS, GDPR standards. |
| **Replay Attack Protection** | 32 MHz            | 64 KB       | 200 µW            | Handles timestamp validation, nonce tracking, and IP-based lockouts. Uses bcrypt hashing (`HASH` function), requiring moderate resources for secure credential verification. |
| **Constants**              | 16 MHz            | 16 KB       | 50 µW             | Foundational module with minimal compute needs for storing immutable constants (e.g., `MAX_PACKET_SIZE`, `ADTC_TIMEOUT`). Low RAM for static data access. |
| **Glossary**               | 16 MHz            | 16 KB       | 50 µW             | Minimal requirements for in-memory term storage and lookup. Depends on string manipulation (`helpers.to_lower_case`), suitable for ultra-low-power devices. |
| **Helpers**                | 32 MHz            | 64 KB       | 200 µW            | Supports utility functions (e.g., `generate_unique_id`, `score_compression_algorithms`, `estimate_rtt`). Moderate resources for cryptographic operations (`crypto_utils.sha3_256`) and TinyML model validation. |

### Module Computational Notes
- **Low-Power Optimizations**: Modules like Compression, PBFT Validation, PoS Validation, and ZKP support Low-Energy Mode (Bit 23), reducing resource needs by using lightweight algorithms (e.g., lz4, hash-based validation, SHA3-256 proofs). Suitable for Raspberry Pi Zero 2W or IoT devices. See `protocol-specs/capability_flags.md` for flag assignments.
- **Interplanetary Scenarios**: Consensus (ADTC) and Audit Trail require robust buffering (`MAX_BUFFER_PERSISTENCE = 30 days`) and storage (1MB ledger, 10MB logs) for 15-30 minute RTTs. Raspberry Pi 4 recommended for full nodes handling ADTC vote processing.
- **High-Capability Devices**: Modules like ZKP (zk-STARKs, Bit 39) and Compression (TinyML, Bit 36) require 80-120 MHz, 128-256 KB RAM, and 500 µW-1 mW, best suited for Raspberry Pi 4 or equivalent.
- **Dependencies**: Cryptographic modules (Audit Trail, Secure Credential Storage, Helpers) rely on SHA3-256, AES-256, or Kyber/Dilithium, needing at least 32 MHz and 64 KB. Compression and ZKP inherit TinyML requirements (80 MHz, 128 KB) when enabled.
- **Scalability**: Full nodes (Raspberry Pi 4) handle all features; light nodes (Raspberry Pi Zero 2W) use simplified validation and local logging to conserve resources.

## Integration with Constants Module (Optional)
To align with the TODO in the Constants module for configuration file integration, the module-specific requirements can be defined as a structured constant. Below is an example pseudocode snippet to integrate into `bindings/pseudo-code/shared/constants.md`:

```pseudocode
// Define hardware requirements for MeshGuardian modules
CONSTANT HARDWARE_REQUIREMENTS = [
    {
        "module": "Audit Trail",
        "min_cpu_mhz": 32,
        "min_ram_kb": 64,
        "power_uw": 200,
        "notes": "Supports multi-blockchain logging (Bit 37) and lightweight ledger. Scales to 1MB ledger."
    },
    {
        "module": "Compression",
        "min_cpu_mhz": 16,
        "min_ram_kb": 32,
        "power_uw": 100,
        "notes": "lz4 baseline; zstd (32 MHz, 64 KB, 200 µW); TinyML, zlib, Brotli (Bit 36, 80 MHz, 128 KB, 500 µW)."
    },
    {
        "module": "Consensus (ADTC)",
        "min_cpu_mhz": 32,
        "min_ram_kb": 64,
        "power_uw": 200,
        "notes": "Handles PoS, PBFT, ADTC. ADTC needs vector clocks and buffering for interplanetary RTTs."
    },
    {
        "module": "PBFT Validation",
        "min_cpu_mhz": 16,
        "min_ram_kb": 32,
        "power_uw": 100,
        "notes": "Hash-based (16 MHz, 32 KB, 100 µW); full PBFT (80 MHz, 128 KB, 500 µW)."
    },
    {
        "module": "PoS Validation",
        "min_cpu_mhz": 16,
        "min_ram_kb": 32,
        "power_uw": 100,
        "notes": "Hash-based (16 MHz, 32 KB, 100 µW); full PoS (32 MHz, 64 KB, 200 µW)."
    },
    {
        "module": "Zero-Knowledge Proofs (ZKP)",
        "min_cpu_mhz": 16,
        "min_ram_kb": 32,
        "power_uw": 100,
        "notes": "Hash-based (16 MHz, 32 KB, 100 µW); Bulletproofs (32 MHz, 64 KB, 200 µW); zk-STARKs (120 MHz, 256 KB, 1 mW, Bit 39)."
    },
    {
        "module": "Secure Credential Storage",
        "min_cpu_mhz": 32,
        "min_ram_kb": 64,
        "power_uw": 200,
        "notes": "Requires AES-256 and secure vault. Aligns with NIST, PCI DSS, GDPR."
    },
    {
        "module": "Replay Attack Protection",
        "min_cpu_mhz": 32,
        "min_ram_kb": 64,
        "power_uw": 200,
        "notes": "Uses bcrypt hashing for credential verification."
    },
    {
        "module": "Constants",
        "min_cpu_mhz": 16,
        "min_ram_kb": 16,
        "power_uw": 50,
        "notes": "Minimal needs for static constant storage."
    },
    {
        "module": "Glossary",
        "min_cpu_mhz": 16,
        "min_ram_kb": 16,
        "power_uw": 50,
        "notes": "Minimal needs for term storage and lookup."
    },
    {
        "module": "Helpers",
        "min_cpu_mhz": 32,
        "min_ram_kb": 64,
        "power_uw": 200,
        "notes": "Supports cryptographic and TinyML utilities."
    }
]
```

## Notes
- Deployment Considerations: Combine node hardware (e.g., Raspberry Pi 4, LoRa HAT) with module requirements to ensure compatibility. For example, a full node running all modules needs Raspberry Pi 4 (1.5 GHz, 2GB RAM) with a high-capacity MicroSD (32GB+) and solar battery pack for interplanetary relays.
- Hardware Compatibility: Node components (LoRa, Zigbee, BLE) align with module requirements for networking (e.g., Compression, Consensus). Verify wireless module support in the MeshGuardian software stack.
- Energy Harvesting: Battery packs with solar charging support Low-Energy Mode (BIT_24) for modules, extending battery life (e.g., 48 hours in Use Case 5.16).
Field Testing: Validate requirements in real-world scenarios (e.g., refugee camp, Use Case 5.15; Mars rover relay, Use Case 5.1.1) to ensure performance under load.

## TODO
- Validation: Test node and module requirements in diverse deployments (e.g., disaster zones, interplanetary simulations) to confirm reliability.
- Configuration File: Integrate module requirements into a JSON/YAML configuration file, parsed by Helpers module, for runtime compatibility checks.
- Compatibility Matrix: Develop a matrix mapping hardware platforms (e.g., Raspberry Pi 4, Zero 2W, Arduino, ESP32) to supported modules and node roles (full/light).
- Future Modules: Add requirements for emerging modules (e.g., Pluggable Protocol Engine plugins, DHT-based P2P sync) as implemented.
Documentation: Enhance with setup guides for specific hardware platforms, linking to
2_step_by_step_process.md
