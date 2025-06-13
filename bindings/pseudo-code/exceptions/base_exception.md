# Pseudocode: Base Exception Class

## Background
Errors in MeshGuardian signal issues across modules (networking, security, 4.1.4, 4.1.1), ensuring robust communication for IoT (4.1.3), crisis response (5.15, 5.16), and blockchain logging (5.11). The `BaseException` class is the parent for all errors, providing common attributes (e.g., message, timestamp). This file defines the foundation for errors, reusable across modules (e.g., /pseudo-code/networking/, /pseudo-code/protocol/), serving as a blueprint for developers.

**Note**: This class is independent of specific protocol configurations like capability flags. However, errors extending this class may be influenced by flags such as Low-Energy Mode (Bit 23) or interplanetary communication (Bit 40) in the 64-bit system. See `protocol-specs/capability_flags.md` for details.

## Purpose
Defines the `BaseException` class as the parent for all MeshGuardian errors, ensuring consistent error handling.

## Conventions
- `CLASS Name`: Defines a class.  
- `METHOD name(args)`: Defines a method.  
- `self`: Refers to the instance, like `this` in other languages.  
- `CALL function`: Invokes a method.  
- `// Comment`: Explains logic.  

## Interfaces
- `BaseException.__init__(message)`

## Depends On
- None (foundational class)

## Called By
- /pseudo-code/exceptions/networking_errors.md (MissingSourceError, etc.)
- Future files (e.g., /pseudo-code/exceptions/protocol_errors.md, /pseudo-code/exceptions/security_errors.md)

## Used In
- Use Case 5.11: Blockchain logging (error tracking for Solana, 4.1.6)
- Use Case 5.12: Telehealth (secure packet errors, 5.12)
- Use Case 5.14: Rural IoT networks (sensor error handling, 4.1.3)
- Use Case 5.15: Aid relays (crisis error debugging, 5.15)
- Use Case 5.16: Emergency chat (reliable error logging, 5.16)

## Pseudocode
```pseudocode
CLASS BaseException
    // Base class for all MeshGuardian errors, providing common attributes.
    // Used In: 5.11, 5.12, 5.14, 5.15, 5.16
    // Purpose: Ensures consistent error handling across modules.

    // Attributes
    message: String    // Error description, e.g., "Source ID cannot be empty"
    timestamp: Integer // Creation time, e.g., Unix epoch

    METHOD __init__(message)
        // Initializes the error with a message and timestamp.
        // Args:
        //     message: String explaining the error
        // Example:
        //     error = NEW BaseException("Generic error")
        //     error.message is "Generic error", error.timestamp is e.g., 1739467200
        // OOP Notes:
        //     - For C++ devs: Like std::exception with custom fields.
        self.message = message
        self.timestamp = CALL get_current_timestamp()  // e.g., Unix epoch