# Logger Module

## Purpose
The Logger module provides a centralized logging mechanism for MeshGuardian, enabling the recording of system events, errors, and debugging information. It supports configurable logging levels (e.g., INFO, DEBUG) and integrates with the audit trail for persistent event storage. The module ensures that logging operations are efficient and secure, supporting the two-tier audit trail system (Solana for Tier 1, local with P2P sync for Tier 2) via Bit 15, and ML-driven features (Bits 13 and 14).

## Depends On

- /pseudo-code/audit_trail.md: Forwards events to the audit trail for persistent logging.
- /pseudo-code/constants.md: Uses LOG_LEVEL to set the initial logging verbosity.
- /pseudo-code/shared/helpers.md: Provides utility functions for formatting log messages.

## Called By
- /pseudo-code/audit_trail.md: Logs events before audit trail processing.
- /pseudo-code/networking/data_transmission.md: Logs transmission-related events.
- /pseudo-code/security/auth.md: Logs authentication attempts and outcomes.

## Used In
- **Use Case 5.15: Aid Relays**: Logs supply tracking events for debugging and compliance.
- **Use Case 5.16: Emergency Chat**: Records message exchange errors for troubleshooting.

## Pseudocode
```pseudocode
// Function to log a message
FUNCTION log_message(level, message, capability_flags)
    // Check if the log level is enabled
    IF level NOT_IN ALLOWED_LEVELS OR level < LOG_LEVEL THEN
        RETURN
    END IF
    // Format log message
    formatted_message = CALL format_log_message(level, message, CALL get_current_timestamp())
    // Forward to audit trail for persistent logging
    event_details = {
        "level": level,
        "message": formatted_message
    }
    CALL log_event("SystemLog", event_details, capability_flags)
    // Output to console or file for debugging (if DEBUG mode)
    IF LOG_LEVEL = "DEBUG" THEN
        CALL output_to_console(formatted_message)
    END IF
END FUNCTION

// Function to set the logging level
FUNCTION set_log_level(new_level)
    IF new_level IN ALLOWED_LEVELS THEN
        LOG_LEVEL = new_level
        CALL log_message("INFO", "Log level set to " + new_level, 0)
    ELSE
        RAISE InvalidLogLevelError("Invalid log level: " + new_level)
    END IF
END FUNCTION
```

---

## Notes
- Configurability: The logging level (LOG_LEVEL) can be adjusted dynamically to control verbosity.
- Integration: All log messages are forwarded to the audit trail, using Solana (Tier 1) for critical logs if Bit 15 = 1, or local P2P sync (Tier 2) otherwise.
- Performance: Logging is optimized to minimize performance impact, with batching for Tier 2 logs.
- Security: Log messages are tamper-proof when stored via the audit trail’s SHA3-256 hashing and ZKP mechanisms.

## TODO
- Implement log filtering by component or module.
- Add support for log output to external systems (e.g., syslog).
