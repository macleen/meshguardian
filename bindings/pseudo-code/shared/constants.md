# Constants Module

## Purpose
The Constants module provides a centralized location for defining and accessing system-wide constants. It exists to ensure consistency, readability, and maintainability across the system by avoiding hard-coded values and making it easy to update values in one place. This module is essential for systems with configuration settings, limits, or predefined values that are used in multiple places.

## Interfaces
- **MAX_PACKET_SIZE**: The maximum allowed size for a data packet in bytes.  
- **DEFAULT_TIMEOUT**: The default timeout value for network operations in seconds.  
- **ENCRYPTION_ALGORITHM**: The default encryption algorithm to use for secure operations.  
- **LOG_LEVEL**: The default logging level for the system (e.g., "INFO", "DEBUG").  

## Depends On
- None (foundational module)

## Called By
- **/pseudo-code/networking/data_transmission.md**: Uses `MAX_PACKET_SIZE` to enforce packet size limits.  
- **/pseudo-code/security/encryption.md**: References `ENCRYPTION_ALGORITHM` for cryptographic operations.  
- **/pseudo-code/logging/logger.md**: Uses `LOG_LEVEL` to set the initial logging verbosity.  

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

// Define ML-related thresholds for static protocol selection
CONSTANT ML_PROTOCOL_RSSI_THRESHOLD = -90  // dBm for Bluetooth vs. LoRa
CONSTANT ML_PROTOCOL_DEFAULT = "LoRa"

// Define ML-related thresholds for static failure prediction
CONSTANT ML_FAILURE_BATTERY_THRESHOLD = 10  // Percent for node failure

```

---

## Notes
- **Immutability**: Constants should be treated as immutable to prevent accidental changes during runtime.  
- **Configuration**: For values that may need to be adjusted per deployment, consider using a configuration file or environment variables instead.  
- **Scalability**: As the system grows, new constants can be added here to maintain a single source of truth.  
- **TODO**: Add constants for additional system parameters, such as buffer sizes or retry limits, as needed.  
