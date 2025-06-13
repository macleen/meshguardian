# Logger Module

## Purpose
The Logger module provides a centralized logging mechanism for MeshGuardian, enabling the recording of system events, errors, and debugging information. It supports configurable logging levels (e.g., INFO, DEBUG) and integrates with the audit trail for persistent event storage. The module ensures that logging operations are efficient, secure, and robust, supporting the two-tier audit trail system (blockchain for Tier 1, local with P2P sync for Tier 2) via Bit 37 in the 64-bit system, and ML-driven features, with buffer overflow handling to prevent log loss.

## Depends On
- **/pseudo-code/audit_trail.md**: Forwards events to the audit trail for persistent logging.
- **/pseudo-code/constants.md**: Uses `LOG_LEVEL` to set the initial logging verbosity.
- **/pseudo-code/shared/helpers.md**: Provides utility functions for formatting log messages.

## Called By
- **/pseudo-code/networking/data_transmission.md**: Logs transmission-related events.
- **/pseudo-code/security/auth.md**: Logs authentication attempts and outcomes.

## Used In
- **Use Case 5.15: Aid Relays**: Logs supply tracking events for debugging and compliance.
- **Use Case 5.16: Emergency Chat**: Records message exchange errors for troubleshooting.

## Pseudocode
```pseudocode
// Constants for buffer management
CONST MAX_BUFFER_SIZE = 10485760  // 10MB default buffer size
CONST CRITICAL_LOG_THRESHOLD = 0.8  // 80% buffer capacity triggers overflow handling

// Function to check buffer capacity and handle overflow
FUNCTION check_buffer_capacity(log_level, log_size)
    available_space = CALL get_available_buffer_space()
    IF available_space < log_size THEN
        // Buffer overflow imminent
        IF log_level IN ["ERROR", "CRITICAL"] THEN
            // Prioritize high-priority logs: Remove oldest low-priority logs
            oldest_low_priority_logs = CALL get_oldest_logs_by_level(["DEBUG", "INFO"], log_size)
            CALL remove_logs(oldest_low_priority_logs)
            // Log overflow event as high-priority
            overflow_message = "Removed low-priority logs to accommodate " + log_level + " log"
            CALL log_message("CRITICAL", overflow_message, capability_flags | BIT(37))  // Force Tier 1 if Bit 37 enabled
        ELSE
            // Drop low-priority log and raise warning
            RAISE BufferOverflowWarning("Buffer full, " + log_level + " log dropped")
            RETURN FALSE
        END IF
    END IF
    RETURN TRUE
END FUNCTION

// Function to log a message
FUNCTION log_message(level, message, capability_flags)
    // Check if the log level is enabled
    IF level NOT_IN ALLOWED_LEVELS OR level < LOG_LEVEL THEN
        RETURN
    END IF
    // Estimate log size
    log_size = CALL estimate_log_size(level, message)
    // Check buffer capacity before logging
    IF NOT CALL check_buffer_capacity(level, log_size) THEN
        RETURN  // Exit if low-priority log cannot be stored
    END IF
    // Format log message
    formatted_message = CALL format_log_message(level, message, CALL get_current_timestamp())
    // Forward to audit trail for persistent logging
    event_details = {
        "level": level,
        "message": formatted_message,
        "priority": (level IN ["ERROR", "CRITICAL"] ? "high" : "low")
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
- Configurability: The logging level (LOG_LEVEL) can be adjusted dynamically to control verbosity, supporting levels like DEBUG, INFO, ERROR, and CRITICAL.
- Integration: All log messages are forwarded to the audit trail, using blockchain (Tier 1) for critical logs if Bit 37 = 1, or local P2P sync (Tier 2) otherwise, with buffer overflow handling to ensure no log loss.
- Performance: Logging is optimized with batching for Tier 2 logs (every 10 seconds via audit trail) and minimal buffer checks (<1ms), ensuring low overhead. In Low-Energy Mode (Bit 23), logging frequency may be reduced to conserve power.
- Security: Log messages are tamper-proof when stored via the audit trail’s SHA3-256 hashing and ZKP mechanisms. Buffer overflow events are logged as Tier 1 for traceability.
- Buffer Management: A 10MB buffer is used with a prioritization strategy to preserve high-priority logs (ERROR, CRITICAL) by removing low-priority logs (DEBUG, INFO) if needed, aligning with Section 7.2.
- 64-Bit Capability Flags: Updated to use Bit 37 for Tier 1 logging (previously Bit 15 in the 32-bit system). Refer to protocol-specs/capability_flags.md for full 64-bit flag definitions.

## TODO
- Implement log filtering by component or module.
- Add support for log output to external systems (e.g., syslog).
- Update references to other 64-bit capability flags as needed (e.g., ML-driven features).
