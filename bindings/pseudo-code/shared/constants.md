# Constants Module

## Purpose
The Constants module centralizes system-wide constants for MeshGuardian, ensuring consistency, readability, and maintainability by avoiding hard-coded values. It supports features like consensus modes, 64-bit capability flags, adaptive compression, time-delayed consensus for interplanetary environments, low-power optimizations, zero-knowledge proofs (ZKP), slashing logic, and multi-blockchain/lightweight ledger logging. This module is immutable, with plans to integrate some constants into a configuration file for deployment flexibility, reducing single-blockchain risks and ensuring node accountability.

## Interfaces
- Consensus Mode Codes: Specifies consensus algorithms (e.g., PoS, PBFT, ADTC) in packet headers.  
- Capability Flags: Defines 64-bit bit masks for MeshGuardian features (see /protocol-specs/capability_flags.md).  
- Packet and Network Constants: Sets limits and timeouts for packet handling and network operations.  
- Encryption and Security Constants: Lists supported encryption algorithms and security thresholds.  
- Logging and Audit Constants: Configures logging levels, buffer sizes, and blockchain audit settings.  
-Compression Constants: Defines codes and thresholds for adaptive compression selection.  
- Consensus and ZKP Constants: Sets timeouts, intervals, and thresholds for consensus and ZKP operations.  
- Slashing and Accountability Constants: Defines thresholds and codes for penalizing malicious nodes.  
- Blockchain and Ledger Constants: Configures priorities and settings for multi-blockchain and lightweight ledger logging.  

## Depends On
- None (foundational module)

## Called By
- **Anywhere where constants are needed**


## Pseudocode
```pseudocode
// Consensus Mode Codes (1-byte field)
// Used in packet headers to specify the consensus algorithm
const CONSENSUS_MODE_POS    = 0x01    // Requires BIT(33)
const CONSENSUS_MODE_PBFT   = 0x02    // Requires BIT(32)
const CONSENSUS_MODE_ADTC   = 0x03    // Requires BIT(34)
const CONSENSUS_MODE_HYBRID = 0x04    // Requires BIT(32), BIT(33), or BIT(34)
const CONSENSUS_MODE_NONE   = 0x00    // No consensus (default)

// Capability Flags Constants
// 64-bit bit masks for MeshGuardian features (see /protocol-specs/capability_flags.md)
// Parsed using bitwise operations (<1 KB RAM)

// Versioning (Bits 0-4)
// Reserved for protocol versioning (0-31)

// Core Features (Bits 5-12)
const BIT_5_CBOR_ENCODING         = 1 << 5  // BPv7 bundle formatting
const BIT_6_BASIC_ROUTING         = 1 << 6  // Store-and-forward DTN routing
const BIT_7_LORA_SUPPORT          = 1 << 7  // Low-bandwidth convergence layer
const BIT_8_NANO_SYNC             = 1 << 8  // 4-byte synchronization packet
const BIT_9_VECTOR_CLOCK          = 1 << 9  // Simplified 4-byte causal ordering
const BIT_10_CUSTODY_TRANSFER     = 1 << 10 // Basic custody acknowledgment
const BIT_11_AES_ENCRYPTION       = 1 << 11 // 0 = AES-128, 1 = AES-256
const BIT_12_HMAC_SHA1            = 1 << 12 // Baseline 160-bit authentication

// Low-Energy Mode (Bit 13)
const BIT_13_LOW_ENERGY_MODE      = 1 << 13 // Disables bits 14-63 for <32 KB RAM

// Optional Security Features (Bits 14-19)
const BIT_14_MINIMAL_LOGGING      = 1 << 14 // Basic event logging (<16 bytes)
const BIT_15_PSK                  = 1 << 15 // Pre-shared key exchange
const BIT_16_SCHNORR              = 1 << 16 // Advanced authentication
const BIT_17_DILITHIUM2           = 1 << 17 // Post-quantum authentication
const BIT_18_KYBER_512            = 1 << 18 // Post-quantum key exchange
const BIT_19_ZKPS                 = 1 << 19 // PrivacySafePods authentication

// Compression (Bits 20-23)
const BIT_20_ZSTD_COMPRESSION     = 1 << 20 // Standard zstd compression
const BIT_21_LZ4_COMPRESSION      = 1 << 21 // Lightweight lz4 compression
const BIT_22_ML_PROTOCOL_SELECTION = 1 << 22 // ML-based protocol choice
const BIT_23_TINYML_COMPRESSION   = 1 << 23 // AI-driven compression

// Extended Compression (Bit 24)
const BIT_24_EXTENDED_COMPRESSION = 1 << 24 // Extended compression (zlib, Brotli) via CBOR sub-flags

// FEC Features (Bit 25)
const BIT_25_FEC_FEATURES         = 1 << 25 // FEC (Reed-Solomon, LDPC) via CBOR sub-flags

// Routing and Convergence (Bits 26-28)
const BIT_26_TRUST_GRAPHS         = 1 << 26 // Advanced routing with trust scores
const BIT_27_TCPCLV4              = 1 << 27 // BPv7 TCP convergence layer
const BIT_28_BLE                  = 1 << 28 // Bluetooth Low Energy convergence

// Synchronization and Custody (Bits 29-31)
const BIT_29_EXTENDED_NANO_SYNC   = 1 << 29 // 7B/10B sync packet variants
const BIT_30_FULL_VECTOR_CLOCK    = 1 << 30 // Extended causal ordering
const BIT_31_EXTENDED_CUSTODY     = 1 << 31 // >10 custody records

// Consensus Mechanisms (Bits 32-34)
const BIT_32_PBFT                 = 1 << 32 // Practical Byzantine Fault Tolerance
const BIT_33_POS                  = 1 << 33 // Proof-of-Stake
const BIT_34_ADTC                 = 1 << 34 // Adaptive DTN Consensus

// Plugins (Bits 35-37)
const BIT_35_ADMIN_RECORDS        = 1 << 35 // BPv7 status reports
const BIT_36_BPSEC                = 1 << 36 // BPv7 security blocks
const BIT_37_BLOCKCHAIN_AUDIT     = 1 << 37 // Blockchain logging for audit integrity

// New Features (Bits 38-39)
const BIT_38_MULTI_BLOCKCHAIN_LOGGING = 1 << 38 // Optional secondary blockchain logging via CBOR sub-flags
const BIT_39_TRUST_DECAY          = 1 << 39 // Passive trust degradation via CBOR sub-flags

// Reserved (Bits 40-63)
// Reserved for future expansions

// CBOR Sub-Flags for BIT_24 (Extended Compression)
const EXT_COMP_ZLIB               = 0x01    // zlib compression
const EXT_COMP_BROTLI             = 0x02    // Brotli compression

// CBOR Sub-Flags for BIT_25 (FEC Features)
const EXT_FEC_RS                  = 0x01    // Reed-Solomon FEC
const EXT_FEC_LDPC                = 0x02    // LDPC FEC

// CBOR Sub-Flags for BIT_38 (Multi-Blockchain Logging)
const EXT_AUDIT_CHAIN_AVALANCHE   = 0x01    // Avalanche blockchain
const EXT_AUDIT_CHAIN_ETHEREUM    = 0x02    // Ethereum blockchain

// CBOR Sub-Flags for BIT_39 (Trust Decay)
const EXT_TRUST_DECAY_PASSIVE     = 0x01    // Passive trust degradation

// Packet and Network Constants
const MAX_PACKET_SIZE = 1024         // Maximum packet size in bytes
const DEFAULT_TIMEOUT = 30           // Default timeout for network operations in seconds
const DEFAULT_RETRY_LIMIT = 3        // Default number of retries for network operations

// Encryption and Security Constants
const ENCRYPTION_ALGORITHMS = {
    "default": "AES-256",
    "LoRa": "Kyber"
}

// Logging and Audit Constants
const LOG_LEVEL = "INFO"             // Default logging level (e.g., "INFO", "DEBUG")
const MAX_BUFFER_SIZE = 10485760     // 10MB max buffer size for logging and audit trails
const BLOCKCHAIN_AUDIT_DEFAULT = 0   // 0 = local logging, 1 = blockchain logging (e.g., Solana)
const BLOCKCHAIN_SUBMISSION_TIMEOUT = 5  // Timeout for blockchain submissions in seconds
const MAX_BUFFER_PERSISTENCE = 2592000  // 30 days in seconds for buffered logs or votes
const BLOCKCHAIN_PRIORITY_LIST = ["Solana", "Avalanche", "Ethereum"]  // Blockchain preferences
const LIGHTWEIGHT_LEDGER_ENABLED = True  // Toggle for lightweight ledger
const LIGHTWEIGHT_LEDGER_MAX_SIZE = 1048576  // 1MB max storage for lightweight ledger

// Compression Constants
const LZ4_CODE = 0x0004              // Compression type code for lz4
const ZSTD_CODE = 0x0002             // Compression type code for zstd
const TINYML_CODE = 0x0006           // Compression type code for TinyML
const NO_COMPRESSION = 0x0000        // No compression
const RSSI_THRESHOLD = -90           // dBm, weak signal threshold
const BATTERY_LOW = 20               // %, low battery threshold
const BATTERY_MODERATE = 50          // %, moderate battery threshold
const PACKET_SIZE_LARGE = 1024       // Bytes, large packet threshold
const PACKET_SIZE_MEDIUM = 768       // Bytes, medium packet threshold
const PACKET_SIZE_SMALL = 256        // Bytes, small packet threshold
const TINYML_MAX_INFERENCE_TIME = 200  // ms, max TinyML inference time
const TINYML_INFERENCE_INTERVAL = 10   // Packets, interval for TinyML inference

// Consensus and ZKP Constants
const ADTC_TIMEOUT = 1800            // 30 minutes in seconds for validation window
const ADTC_MAX_LOGICAL_CLOCK = 4294967295  // 32-bit max for Lamport clock
const ADTC_CONSENSUS_CODE = 0x03     // Consensus mode code for ADTC
const RTT_THRESHOLD = 900            // 15 minutes in seconds for RTT timeout adjustments
const ZKP_GENERATION_INTERVAL = 10   // Events, interval for ZKP generation
const PBFT_VALIDATION_INTERVAL = 10  // Messages, interval for PBFT validation
const ZKP_MAX_COMPUTATION_TIME = 500 // ms, max ZKP computation time before fallback

// Slashing and Accountability Constants
const MALICIOUS_VOTE_THRESHOLD = 3   // Max malicious vote attempts before slashing
const SLASHING_EVENT_CODE = 0xFF     // Code for slashing events in audit trail

// Bit positions for capability flags (64-bit)
SET ML_ROUTING_BIT = 1 << 22          // Example: Bit 22 for ML-driven routing
SET ML_FAILURE_PREDICTION_BIT = 1 << 23 // Example: Bit 23 for failure prediction
// Add other bits as per the MeshGuardian specification

// Debug Mode
const DEBUG_MODE = False             // Toggle for debug logging


```

---

## Notes
- 64-Bit Flags: Transitioned to a 64-bit capability flag structure, adding new features like extended compression (Bit 24), multi-blockchain logging (Bit 38), and trust decay (Bit 39).
- Immutability: Constants are immutable to prevent runtime changes, ensuring consistency across modules.
- Configuration: Values like BLOCKCHAIN_AUDIT_DEFAULT, ADTC_TIMEOUT, and TINYML_INFERENCE_INTERVAL are candidates for a future configuration file.
- Scalability: Supports additional constants for compression, consensus, ZKP, slashing, and blockchain logging.
- Security: MALICIOUS_VOTE_THRESHOLD and SLASHING_EVENT_CODE ensure node accountability, with BLOCKCHAIN_PRIORITY_LIST reducing single-blockchain risks.
- Compression: Aligns with capability flags (Bits 20-24) for dynamic algorithm selection.
- Consensus:Supports ADTC, PBFT, and PoS with relevant timeouts and thresholds.
- Interplanetary: RTT_THRESHOLD and ADTC_TIMEOUT cater to high-latency environments..
- Low-Power: TINYML_INFERENCE_INTERVAL optimizes for resource-constrained devices.

## TODO
- Map legacy 32-bit flags to 64-bit or deprecate them.
- Integrate configurable constants into a configuration file.
- Add constants for advanced slashing policies (e.g., reputation scoring).
- Define adaptive batching intervals for audit trails.