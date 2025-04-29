# Constants Module

## Purpose
The Constants module provides a centralized location for defining and accessing system-wide constants. It ensures consistency, readability, and maintainability by avoiding hard-coded values, supporting ML-driven features (Bits 13 and 14), blockchain audit logging (Bit 15), adaptive compression selection, time-delayed consensus for interplanetary environments, low-power optimizations, and security enhancements. Constants are defined for packet handling, encryption, logging, synchronization, compression, consensus, ZKP generation, slashing logic, and multi-blockchain/lightweight ledger logging, with plans for configuration file integration to mitigate single-blockchain risks and ensure node accountability.

## Interfaces
- **MAX_PACKET_SIZE**: Maximum allowed packet size in bytes.
- **DEFAULT_TIMEOUT**: Default timeout for network operations in seconds.
- **ENCRYPTION_ALGORITHMS**: Supported encryption algorithms for secure operations.
- **LOG_LEVEL**: Default logging level (e.g., "INFO", "DEBUG").
- **ML_PROTOCOL_RSSI_THRESHOLD**: RSSI threshold for static protocol selection (Bit 13).
- **ML_PROTOCOL_DEFAULT**: Default protocol for static selection.
- **ML_FAILURE_BATTERY_THRESHOLD**: Battery threshold for static failure prediction (Bit 14).
- **BLOCKCHAIN_AUDIT_DEFAULT**: Default mode for blockchain audit logging (Bit 15).
- **MAX_BUFFER_SIZE**: Maximum buffer size for logging and audit trails.
- **DEFAULT_RETRY_LIMIT**: Default number of retries for network operations.
- **LZ4_CODE**: Compression type code for lz4.
- **ZSTD_CODE**: Compression type code for zstd.
- **TINYML_CODE**: Compression type code for TinyML.
- **NO_COMPRESSION**: Compression type code for no compression.
- **RSSI_THRESHOLD**: RSSI threshold for compression selection.
- **BATTERY_LOW**: Low battery threshold for compression selection.
- **BATTERY_MODERATE**: Moderate battery threshold for compression selection.
- **PACKET_SIZE_LARGE**: Large packet size threshold for compression selection.
- **PACKET_SIZE_MEDIUM**: Medium packet size threshold for compression selection.
- **PACKET_SIZE_SMALL**: Small packet size threshold for compression selection.
- **TINYML_MAX_INFERENCE_TIME**: Maximum TinyML inference time before fallback.
- **TINYML_INFERENCE_INTERVAL**: Interval for TinyML inference on low-power devices.
- **ZKP_GENERATION_INTERVAL**: Interval for ZKP generation on low-power devices.
- **PBFT_VALIDATION_INTERVAL**: Interval for PBFT validation on low-power devices.
- **ZKP_MAX_COMPUTATION_TIME**: Maximum ZKP computation time before fallback.
- **DEBUG_MODE**: Toggle for debug logging.
- **ADTC_TIMEOUT**: Timeout window for Asynchronous Delay-Tolerant Consensus in seconds.
- **ADTC_MAX_LOGICAL_CLOCK**: Maximum logical clock value before reset.
- **ADTC_CONSENSUS_CODE**: Consensus mode code for ADTC.
- **RTT_THRESHOLD**: RTT threshold for dynamic timeout adjustments in interplanetary scenarios.
- **BLOCKCHAIN_SUBMISSION_TIMEOUT**: Default timeout for blockchain submissions.
- **MAX_BUFFER_PERSISTENCE**: Maximum persistence duration for buffered logs or votes.
- **MALICIOUS_VOTE_THRESHOLD**: Maximum malicious vote attempts before slashing.
- **SLASHING_EVENT_CODE**: Code for slashing events in audit trail logging.
- **BLOCKCHAIN_PRIORITY_LIST**: Ordered list of blockchains for Tier 1 logging.
- **LIGHTWEIGHT_LEDGER_ENABLED**: Toggles lightweight ledger for critical events.
- **LIGHTWEIGHT_LEDGER_MAX_SIZE**: Maximum storage size for lightweight ledger.

## Depends On
- None (foundational module)

## Called By
- **/pseudo-code/networking/data_transmission.md**: Uses `MAX_PACKET_SIZE` to enforce packet size limits.
- **/pseudo-code/security/encryption.md**: References `ENCRYPTION_ALGORITHMS` for cryptographic operations.
- **/pseudo-code/audit_trail.md**: Uses `BLOCKCHAIN_AUDIT_DEFAULT`, `MAX_BUFFER_SIZE`, `RTT_THRESHOLD`, `BLOCKCHAIN_SUBMISSION_TIMEOUT`, `MAX_BUFFER_PERSISTENCE`, `BLOCKCHAIN_PRIORITY_LIST`, `LIGHTWEIGHT_LEDGER_ENABLED`, `LIGHTWEIGHT_LEDGER_MAX_SIZE`, and `SLASHING_EVENT_CODE` for logging configuration.
- **/pseudo-code/networking/packet_sending.md**: Uses `MAX_PACKET_SIZE` and `DEFAULT_TIMEOUT` for packet operations.
- **/pseudo-code/integration/external_sync.md**: Uses `DEFAULT_RETRY_LIMIT` for synchronization retries.
- **/pseudo-code/compression.md**: Uses compression-related constants and `TINYML_INFERENCE_INTERVAL` for adaptive selection.
- **/pseudo-code/consensus.md**: Uses `ADTC_TIMEOUT`, `ADTC_MAX_LOGICAL_CLOCK`, `ADTC_CONSENSUS_CODE`, `RTT_THRESHOLD`, and `MAX_BUFFER_PERSISTENCE` for time-delayed consensus.
- **/pseudo-code/security/zkp.md**: Uses `ZKP_GENERATION_INTERVAL` and `ZKP_MAX_COMPUTATION_TIME` for ZKP generation.
- **/pseudo-code/consensus/pbft_validation.md**: Uses `PBFT_VALIDATION_INTERVAL`, `RTT_THRESHOLD`, `MAX_BUFFER_PERSISTENCE`, `MALICIOUS_VOTE_THRESHOLD`, and `SLASHING_EVENT_CODE` for PBFT validation and slashing.
- **/pseudo-code/consensus/pos_validation.md**: Uses `RTT_THRESHOLD`, `MAX_BUFFER_PERSISTENCE`, `MALICIOUS_VOTE_THRESHOLD`, and `SLASHING_EVENT_CODE` for PoS validation and slashing.

## Used In
- **Use Case 5.15: Aid Relays**: Relies on `MAX_PACKET_SIZE`, compression constants, `TINYML_INFERENCE_INTERVAL`, and `MALICIOUS_VOTE_THRESHOLD` to optimize data transmission and ensure node accountability in low-bandwidth environments.
- **Use Case 5.16: Emergency Chat**: Uses `DEFAULT_TIMEOUT`, `DEFAULT_RETRY_LIMIT`, compression constants, `ZKP_GENERATION_INTERVAL`, and `SLASHING_EVENT_CODE` to manage message delivery, efficiency, and malicious node detection.
- **Use Case 5.1.1: Mars Rover Data Relay**: Uses `ADTC_TIMEOUT`, `ADTC_CONSENSUS_CODE`, `RTT_THRESHOLD`, `MAX_BUFFER_PERSISTENCE`, `PBFT_VALIDATION_INTERVAL`, `BLOCKCHAIN_PRIORITY_LIST`, and `LIGHTWEIGHT_LEDGER_ENABLED` for reliable consensus and logging in high-latency interplanetary networks.

## Pseudocode
```pseudocode
// Define the maximum packet size in bytes
CONSTANT MAX_PACKET_SIZE = 1024

// Define the default timeout for network operations in seconds
CONSTANT DEFAULT_TIMEOUT = 30

// Define supported encryption algorithms
CONSTANT ENCRYPTION_ALGORITHMS = {
    "default": "AES-256",
    "LoRa": "Kyber"
}

// Define the default logging level
CONSTANT LOG_LEVEL = "INFO"

// Define ML-related thresholds for static protocol selection (Bit 13)
CONSTANT ML_PROTOCOL_RSSI_THRESHOLD = -90  // dBm for Bluetooth vs. LoRa
CONSTANT ML_PROTOCOL_DEFAULT = "LoRa"

// Define ML-related thresholds for static failure prediction (Bit 14)
CONSTANT ML_FAILURE_BATTERY_THRESHOLD = 10  // Percent for node failure

// Define blockchain audit logging default (Bit 15)
CONSTANT BLOCKCHAIN_AUDIT_DEFAULT = 0  // 0 = local logging, 1 = Solana

// Define reserved constant for lightweight blockchain (Bit 16)
CONSTANT LIGHTWEIGHT_BLOCKCHAIN_RESERVED = 0  // Placeholder

// Define maximum buffer size for logging and audit trails
CONSTANT MAX_BUFFER_SIZE = 10485760  // 10MB

// Define default retry limit for network operations
CONSTANT DEFAULT_RETRY_LIMIT = 3  // Number of retries

// Define compression type codes
CONSTANT LZ4_CODE = 0x0004          // Compression type code for lz4
CONSTANT ZSTD_CODE = 0x0002         // Compression type code for zstd
CONSTANT TINYML_CODE = 0x0006       // Compression type code for TinyML
CONSTANT NO_COMPRESSION = 0x0000    // No compression

// Define thresholds for adaptive compression selection
CONSTANT RSSI_THRESHOLD = -90       // dBm, weak signal threshold
CONSTANT BATTERY_LOW = 20           // %, low battery threshold
CONSTANT BATTERY_MODERATE = 50      // %, moderate battery threshold
CONSTANT PACKET_SIZE_LARGE = 1024   // Bytes, large packet threshold
CONSTANT PACKET_SIZE_MEDIUM = 768   // Bytes, medium packet threshold
CONSTANT PACKET_SIZE_SMALL = 256    // Bytes, small packet threshold
CONSTANT TINYML_MAX_INFERENCE_TIME = 200  // ms, max TinyML inference time
CONSTANT TINYML_INFERENCE_INTERVAL = 10   // Packets, interval for TinyML inference on low-power devices
CONSTANT ZKP_GENERATION_INTERVAL = 10     // Events, interval for ZKP generation on low-power devices
CONSTANT PBFT_VALIDATION_INTERVAL = 10    // Messages, interval for PBFT validation on low-power devices
CONSTANT ZKP_MAX_COMPUTATION_TIME = 500   // ms, max ZKP computation time before fallback
CONSTANT DEBUG_MODE = FALSE         // Toggle debug logging

// Define constants for Asynchronous Delay-Tolerant Consensus (ADTC)
CONSTANT ADTC_TIMEOUT = 1800        // 30 minutes in seconds for validation window
CONSTANT ADTC_MAX_LOGICAL_CLOCK = 4294967295  // 32-bit max for Lamport clock
CONSTANT ADTC_CONSENSUS_CODE = 0x03  // Consensus mode code for ADTC

// Define constants for interplanetary deployments
CONSTANT RTT_THRESHOLD = 900        // 15 minutes in seconds for RTT timeout adjustments
CONSTANT BLOCKCHAIN_SUBMISSION_TIMEOUT = 5  // Default timeout for blockchain submissions in seconds
CONSTANT MAX_BUFFER_PERSISTENCE = 2592000  // 30 days in seconds for buffered logs or votes

// Define constants for slashing logic
CONSTANT MALICIOUS_VOTE_THRESHOLD = 3  // Max malicious vote attempts before slashing
CONSTANT SLASHING_EVENT_CODE = 0xFF    // Code for slashing events in audit trail

// Define constants for multi-blockchain and lightweight ledger logging
CONSTANT BLOCKCHAIN_PRIORITY_LIST = ["Solana", "Avalanche", "Ethereum"]  // Order of blockchain preferences
CONSTANT LIGHTWEIGHT_LEDGER_ENABLED = TRUE  // Toggle for lightweight ledger
CONSTANT LIGHTWEIGHT_LEDGER_MAX_SIZE = 1048576  // 1MB max storage for lightweight ledger

```

---

## Notes
- Immutability: Constants are immutable to prevent runtime changes, ensuring consistency across modules.
- Configuration: Values like BLOCKCHAIN_AUDIT_DEFAULT, compression thresholds, ADTC_TIMEOUT, RTT_THRESHOLD, BLOCKCHAIN_SUBMISSION_TIMEOUT, MAX_BUFFER_PERSISTENCE, low-power intervals (TINYML_INFERENCE_INTERVAL, etc.), MALICIOUS_VOTE_THRESHOLD, SLASHING_EVENT_CODE, BLOCKCHAIN_PRIORITY_LIST, and LIGHTWEIGHT_LEDGER_ENABLED should move to a configuration file for deployment flexibility, planned as a future enhancement.
- Scalability: Supports additional constants for compression, consensus, ZKP generation, interplanetary operations, slashing logic (MALICIOUS_VOTE_THRESHOLD, SLASHING_EVENT_CODE), and multi-blockchain/lightweight ledger logging (BLOCKCHAIN_PRIORITY_LIST, LIGHTWEIGHT_LEDGER_ENABLED, LIGHTWEIGHT_LEDGER_MAX_SIZE) to enable adaptive features for security, low-power, and high-latency environments.
- Security: MALICIOUS_VOTE_THRESHOLD and SLASHING_EVENT_CODE ensure accountability by enabling slashing for malicious nodes, with BLOCKCHAIN_PRIORITY_LIST and LIGHTWEIGHT_LEDGER_ENABLED reducing single-blockchain dependency risks through diversified logging.
- Compression Flexibility: Compression constants (LZ4_CODE, PACKET_SIZE_MEDIUM, TINYML_INFERENCE_INTERVAL) align with capability flags (Bits 7, 9, 6) for dynamic algorithm selection and low-power optimization.
- Consensus Support: ADTC_TIMEOUT, ADTC_MAX_LOGICAL_CLOCK, ADTC_CONSENSUS_CODE, RTT_THRESHOLD, MAX_BUFFER_PERSISTENCE, PBFT_VALIDATION_INTERVAL, MALICIOUS_VOTE_THRESHOLD, and SLASHING_EVENT_CODE enable time-delayed consensus, reliable buffering, and node accountability in PBFT and PoS.
- ZKP Support: ZKP_GENERATION_INTERVAL and ZKP_MAX_COMPUTATION_TIME support energy-efficient ZKP generation on resource-constrained devices.
- Debug Support: DEBUG_MODE enables optional logging for compression, consensus, ZKP, and slashing decisions, aiding field debugging without runtime overhead when disabled.

## TODO
- Add constants for additional system parameters, such as buffer sizes or retry limits.
- Define constants for adaptive batching intervals in the audit trail.
- Integrate compression, consensus, ZKP, interplanetary, and security thresholds with a configuration file for runtime adjustments.
- Consolidate hardware requirements into a configuration file or compatibility matrix for deployment documentation.
- Add constants for advanced slashing policies, such as reputation scoring or partial penalties.