# Pseudocode: Networking Error Classes

## Background
Errors in MeshGuardian’s Core Networking module signal issues during packet operations (creation, routing, sending, 4.1.4), ensuring robust communication for IoT (4.1.3), crisis response (5.15, 5.16), and blockchain logging (5.11). Each error class identifies a specific failure (e.g., missing sender), like Laravel’s `ValidationException` for fields, and extends BaseException for consistency (/pseudo-code/exceptions/base_exception.md). This file defines networking errors, reusable across modules (e.g., /pseudo-code/protocol/, /pseudo-code/security/), serving as a blueprint.

## Purpose
Defines error classes for packet creation and networking tasks, enabling precise debugging and validation.

## Conventions
- `CLASS Name`: Defines a class, like PHP’s `class` or JS’s `class`.  
- `METHOD name(args)`: Defines a method, like Laravel’s method or JS function.  
- `self`: Refers to the instance, like `$this` or `this`.  
- `// Comment`: Explains logic, like PHP/JS comments.  

## Interfaces
- `MissingSourceError.__init__(message)`
- `MissingDestError.__init__(message)`
- `EmptyDataError.__init__(message)`
- `InvalidProfileError.__init__(message)`
- `OversizedDataError.__init__(message)`
- `DuplicateIdError.__init__(message)`

## Depends On
- `/pseudo-code/exceptions/base_exception.md` (parent BaseException)

## Called By
- /pseudo-code/networking/packet_creation.md (PacketCreator.create, Packet.create)
- /pseudo-code/networking/packet_validation.md (future validation tasks)
- /pseudo-code/networking/packet_sending.md (future sending tasks)
- Other modules (e.g., /pseudo-code/protocol/, /pseudo-code/compression/)

## Used In
- Use Case 5.11: Blockchain logging (validate packets for Solana, 4.1.6)
- Use Case 5.12: Telehealth (secure packet data, 5.12)
- Use Case 5.14: Rural IoT networks (sensor data validation, 4.1.3)
- Use Case 5.15: Aid relays (robust crisis communication, 5.15)
- Use Case 5.16: Emergency chat (reliable messaging, 5.16)

## Pseudocode

```python
CLASS MissingSourceError(BaseException)
    """
    Raised when a packet operation fails due to a missing source ID.
    Used In: 5.11, 5.12, 5.14, 5.15, 5.16
    Purpose: Pinpoints source validation errors for debugging.
    Extends: /pseudo-code/exceptions/base_exception.md
    """
    METHOD __init__(message)
        """
        Initializes the error with a message and timestamp.
        Args:
            message: String explaining the error
        Example:
            RAISE MissingSourceError("Source ID cannot be empty")
            // Handled by caller, e.g., log and retry
        OOP Notes:
            - Like Laravel’s ValidationException for a field.
            - Similar to JavaScript’s Error("message").
            - For C++ devs: Like std::invalid_argument.
        Python Learning:
            - Extends BaseException, like Laravel’s class Exception.
            - `self.message` is like `$this->message` in PHP.
            - Prepares for try/except, like try { ... } catch.
        """
        CALL super.__init__(message)  // Inherits BaseException’s message, timestamp

CLASS MissingDestError(BaseException)
    """
    Raised when a packet operation fails due to a missing destination ID.
    Used In: 5.11, 5.12, 5.14, 5.15, 5.16
    Purpose: Pinpoints destination validation errors.
    Extends: /pseudo-code/exceptions/base_exception.md
    """
    METHOD __init__(message)
        """
        Initializes the error with a message and timestamp.
        Args:
            message: String explaining the error
        Example:
            RAISE MissingDestError("Destination ID cannot be empty")
            // Handled by caller, e.g., log error
        OOP Notes:
            - Like Laravel’s ValidationException for a field.
            - Similar to JavaScript’s Error("message").
            - For C++ devs: Like std::invalid_argument.
        Python Learning:
            - Like PHP’s throw new Exception.
            - Uses BaseException’s timestamp for logging.
        """
        CALL super.__init__(message)

CLASS EmptyDataError(BaseException)
    """
    Raised when a packet operation fails due to empty payload data.
    Used In: 5.11, 5.12, 5.14, 5.15, 5.16
    Purpose: Ensures packets carry meaningful data.
    Extends: /pseudo-code/exceptions/base_exception.md
    """
    METHOD __init__(message)
        """
        Initializes the error with a message and timestamp.
        Args:
            message: String explaining the error
        Example:
            RAISE EmptyDataError("Payload data cannot be empty")
            // Handled by caller, e.g., retry packet
        OOP Notes:
            - Like Laravel’s ValidationException for a field.
            - Similar to JavaScript’s Error("message").
            - For C++ devs: Like std::invalid_argument.
        Python Learning:
            - Like PHP’s throw new Exception.
            - `self.message` stores error details, like Laravel.
        """
        CALL super.__init__(message)

CLASS InvalidProfileError(BaseException)
    """
    Raised when a packet operation fails due to an invalid profile.
    Used In: 5.11, 5.12, 5.14, 5.15, 5.16
    Purpose: Ensures headers use valid profiles.
    Extends: /pseudo-code/exceptions/base_exception.md
    """
    METHOD __init__(message)
        """
        Initializes the error with a message and timestamp.
        Args:
            message: String explaining the error
        Example:
            RAISE InvalidProfileError("Profile 'invalid' not recognized")
            // Handled by caller, e.g., use default profile
        OOP Notes:
            - Like Laravel’s ValidationException for a field.
            - Similar to JavaScript’s Error("message").
            - For C++ devs: Like std::invalid_argument.
        Python Learning:
            - Like PHP’s throw new Exception.
            - Inherits BaseException’s fields.
        """
        CALL super.__init__(message)

CLASS OversizedDataError(BaseException)
    """
    Raised when a packet operation fails due to oversized payload data.
    Used In: 5.11, 5.12, 5.14, 5.15, 5.16
    Purpose: Enforces limits for transports like LoRa (4.1.3).
    Extends: /pseudo-code/exceptions/base_exception.md
    """
    METHOD __init__(message)
        """
        Initializes the error with a message and timestamp.
        Args:
            message: String explaining the error
        Example:
            RAISE OversizedDataError("Payload exceeds 255 bytes")
            // Handled by caller, e.g., compress data
        OOP Notes:
            - Like Laravel’s ValidationException for a field.
            - Similar to JavaScript’s Error("message").
            - For C++ devs: Like std::length_error.
        Python Learning:
            - Like PHP’s throw new Exception.
            - Timestamp aids crisis logging (5.16).
        """
        CALL super.__init__(message)

CLASS DuplicateIdError(BaseException)
    """
    Raised when a packet operation generates a duplicate ID (rare).
    Used In: 5.11, 5.12, 5.14, 5.15, 5.16
    Purpose: Ensures unique packet identifiers.
    Extends: /pseudo-code/exceptions/base_exception.md
    """
    METHOD __init__(message)
        """
        Initializes the error with a message and timestamp.
        Args:
            message: String explaining the error
        Example:
            RAISE DuplicateIdError("ID conflicts with existing packet")
            // Handled by caller, e.g., regenerate ID
        OOP Notes:
            - Like Laravel’s ValidationException for uniqueness.
            - Similar to JavaScript’s Error("message").
            - For C++ devs: Like std::runtime_error.
        Python Learning:
            - Like PHP’s throw new Exception.
            - BaseException’s timestamp tracks errors.
        """
        CALL super.__init__(message)

## Notes
- **Single Responsibility**: Each error handles one failure, ensuring precise debugging (4.1.4).  
- **Extensibility**: Add errors (e.g., InvalidDataFormatError for 4.1.2) for other modules.  
- **Reusability**: Applies to creation, validation, sending, like Laravel’s Exceptions.  
- **Edge Cases**: Covers empty inputs, invalid profiles, oversized data, duplicate IDs (4.1.3, 5.16).  
- **Dependencies**: Extends /pseudo-code/exceptions/base_exception.md.  
- **Python Learning**:  
  - Errors are like Laravel’s throw new Exception, for try/except.  
  - `self.message` is like `$this->message` in PHP.  

## Contributor Guide
Contribute by refining or adding error classes to enhance networking robustness. See /pseudo-code/CONTRIBUTING.md for PR guidelines.

1. **Understand Errors**: Read Background, Used In (5.11–5.16).  
2. **Refine Pseudocode**: Add attributes (e.g., `code`)? New errors (e.g., `InvalidDataFormatError`)?  
3. **Validate Design**: Ensure SRP, cover edge cases (4.1.3).  
4. **Document**: Update front-matter, add comments like Laravel’s.  
5. **Submit**: Fork, commit (e.g., “Add error”), PR to `/pseudo-code/`.