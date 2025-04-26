# Constants Module

## Purpose
The Constants module provides a centralized location for defining and accessing system-wide constants. It ensures consistency, readability, and maintainability across the system by avoiding hard-coded values and making it easy to update values in one place. This module is essential for systems with configuration settings, limits, or predefined values, including those related to ML-driven features (Bits 13 and 14) and blockchain audit logging (Bit 15).

## Interfaces
- **MAX_PACKET_SIZE**: The maximum allowed size for a data packet in bytes.  
- **DEFAULT_TIMEOUT**: The default timeout value for network operations in seconds.  
- **ENCRYPTION_ALGORITHM**: The default encryption algorithm to use for secure operations.  
- **LOG_LEVEL**: The default logging level for the system (e.g., "INFO", "DEBUG").  
- **ML_PROTOCOL_RSSI_THRESHOLD**: RSSI threshold for static protocol selection (Bit 13).
- **ML_PROTOCOL_DEFAULT**: Default protocol for static selection.
- **ML_FAILURE_BATTERY_THRESHOLD**: Battery threshold for static failure prediction (Bit 14).
- **BLOCKCHAIN_AUDIT_DEFAULT**: Default mode for blockchain audit logging (Bit 15).

## Depends On
- None (foundational module)

## Called By
- **/pseudo-code/networking/data_transmission.md**: Uses `MAX_PACKET_SIZE` to enforce packet size limits.  
- **/pseudo-code/security/encryption.md**:References ENCRYPTION_ALGORITHM for cryptographic operations.
- **/pseudo-code/audit_trail.md**: Uses BLOCKCHAIN_AUDIT_DEFAULT for logging configuration.
- **/pseudo-code/networking/packet_sending.md**: Uses MAX_PACKET_SIZE and DEFAULT_TIMEOUT for packet operations.


## Used In
- **Use Case 5.15: Aid Relays**: Relies on `MAX_PACKET_SIZE` to optimize data transmission in low-bandwidth environments.  
- **Use Case 5.16: Emergency Chat**: Uses `DEFAULT_TIMEOUT` to manage message delivery timeouts in real-time communication.  


## Pseudocode
The constants are defined as immutable values that can be accessed throughout the system. They are grouped here for clarity and ease of maintenance.
```pseudocode

// Define the maximum packet size in bytes
CONSTANT MAX_PACKET_SIZE = 1024

// Define the default timeout for network operations in seconds
CONSTANT DEFAULT_TIMEOUT = 30

// Define the default encryption algorithm
CONSTANT ENCRYPTION_ALGORITHM = "AES-256"

// Define the default logging level
CONSTANT LOG_LEVEL = "INFO"

// Define ML-related thresholds for static protocol selection (Bit 13)
CONSTANT ML_PROTOCOL_RSSI_THRESHOLD = -90  // dBm for Bluetooth vs. LoRa
CONSTANT ML_PROTOCOL_DEFAULT = "LoRa"

// Define ML-related thresholds for static failure prediction (Bit 14)
CONSTANT ML_FAILURE_BATTERY_THRESHOLD = 10  // Percent for node failure

// Define blockchain audit logging default (Bit 15)
CONSTANT BLOCKCHAIN_AUDIT_DEFAULT = 0  // 0 = local logging, 1 = Solana

// Reserved for lightweight blockchain (Bit 16)
CONSTANT LIGHTWEIGHT_BLOCKCHAIN_RESERVED = 0  // Placeholder

```

---

## Notes
- **Immutability**: Constants should be treated as immutable to prevent accidental changes during runtime.  
- **Configuration**: For values like BLOCKCHAIN_AUDIT_DEFAULT that may vary per deployment, consider using a configuration file or environment variables. 
- **Scalability**: New constants (e.g., for Bit 16 lightweight blockchain) can be added as the system evolves.
- **TODO**: Add constants for additional system parameters, such as buffer sizes or retry limits.
Define constants for adaptive batching intervals in the audit trail.