# Helpers Module

## Purpose
The Helpers module provides a set of utility functions that are commonly used across the system. It exists to promote code reuse, reduce duplication, and simplify complex operations by offering a centralized location for helper functions. This module is essential for maintaining clean, efficient, and readable code, with robust error handling and logging for traceability. New functions support adaptive compression selection, time-delayed consensus, RTT estimation for interplanetary environments, and low-power optimizations for ultra-low-power devices, with references to hardware requirements to guide deployment.

## Interfaces
- **generate_unique_id()**: Generates a unique identifier (e.g., UUID) for use in the system.
- **format_timestamp(timestamp)**: Formats a timestamp into a human-readable string.
- **parse_data(data, format)**: Parses data from a specified format (e.g., JSON, XML) into a usable structure.
- **validate_input(input, rules)**: Validates input data against a set of rules or constraints.
- **score_compression_algorithms(context)**: Scores compression algorithms (`lz4`, `zstd`, `TinyML`) based on packet size, RSSI, and battery health.
- **validate_tinyml_model(model)**: Validates the availability and integrity of a TinyML model for compression.
- **get_current_time_ms()**: Returns the current time in milliseconds for timing operations.
- **increment_lamport_clock(current_clock)**: Increments a Lamport clock for event ordering in time-delayed consensus.
- **merge_vector_clock(local_clock, received_clock)**: Merges a received vector clock with the local clock for causal ordering.
- **validate_adtc_vote(vote, signature, public_key, capability_flags)**: Validates an ADTC vote’s authenticity and integrity, with support for quantum-ready signatures (Bit 39).
- **estimate_rtt(nodes)**: Estimates real-time RTT for interplanetary nodes using orbital data or historical trends.
- **compute_hash_proof(data)**: Generates a SHA3-256 hash-based proof for low-power devices.
- **check_device_capability(context)**: Checks if simplified algorithms should be used based on device capability and energy mode.

## Depends On
- **/pseudo-code/crypto/random.md**: Provides cryptographically secure random bytes for unique IDs.
- **/pseudo-code/logging/logger.md**: Logs errors and events for traceability.
- **/pseudo-code/constants.md**: Defines thresholds for compression scoring, ADTC parameters, RTT estimation, and low-power intervals.
- **/pseudo-code/security/crypto_utils.md**: Provides cryptographic functions for vote validation and hash-based proofs.


## Pseudocode
```pseudocode
// Function to generate a unique identifier
FUNCTION generate_unique_id()
    TRY
        // Use cryptographically secure random bytes for UUID
        unique_id = GENERATE_RANDOM_BYTES(16)  // 128-bit UUID
        RETURN HEX_ENCODE(unique_id)
    CATCH random_error
        CALL log_message("ERROR", "Failed to generate unique ID: " + random_error, capability_flags | BIT(37)) // Log as Tier 1 with multi-blockchain support
        RAISE RandomError("Unique ID generation failed: " + random_error)
    END TRY
END FUNCTION

// Function to format a timestamp into a human-readable string
FUNCTION format_timestamp(timestamp)
    TRY
        // Convert the timestamp to a formatted string (e.g., "YYYY-MM-DD HH:MM:SS")
        formatted_time = TIMESTAMP_TO_STRING(timestamp, "YYYY-MM-DD HH:MM:SS")
        RETURN formatted_time
    CATCH format_error
        CALL log_message("ERROR", "Failed to format timestamp: " + format_error, capability_flags | BIT(37)) // Log as Tier 1
        RAISE FormatError("Timestamp formatting failed: " + format_error)
    END TRY
END FUNCTION

// Function to parse data from a specified format
FUNCTION parse_data(data, format)
    TRY
        // Use basic parsing logic (replace with proper parser if needed)
        IF format == "JSON" THEN
            parsed_data = JSON_PARSE(data)
        ELSE IF format == "XML" THEN
            parsed_data = XML_PARSE(data)
        ELSE
            RAISE FormatError("Unsupported format: " + format)
        END IF
        RETURN parsed_data
    CATCH parse_error
        CALL log_message("ERROR", "Failed to parse data: " + parse_error + " for format: " + format, capability_flags | BIT(37)) // Log as Tier 1
        RAISE ParseError("Data parsing failed: " + parse_error)
    END TRY
END FUNCTION

// Function to validate input data against a set of rules
FUNCTION validate_input(input, rules)
    TRY
        // Apply each validation rule to the input
        FOR each rule IN rules
            IF NOT rule(input) THEN
                description = (rule.description ? rule.description : "No description provided")
                RAISE ValidationError("Input failed validation: " + description)
            END IF
        END FOR
        RETURN True
    CATCH validation_error
        CALL log_message("ERROR", "Input validation failed: " + validation_error, capability_flags | BIT(37)) // Log as Tier 1
        RAISE ValidationError(validation_error)
    END TRY
END FUNCTION

// Function to score compression algorithms
FUNCTION score_compression_algorithms(context)
    TRY
        // Initialize scores
        scores = {
            "lz4": 0,
            "zstd": 0,
            "tinyml": 0
        }

        // Evaluate packet size
        IF context.packet_size > constants.PACKET_SIZE_LARGE THEN
            scores["zstd"] += 40
            scores["tinyml"] += 30
            scores["lz4"] += 10
        ELSE IF context.packet_size > constants.PACKET_SIZE_SMALL THEN
            scores["tinyml"] += 40
            scores["zstd"] += 30
            scores["lz4"] += 20
        ELSE
            scores["lz4"] += 40
            scores["tinyml"] += 20
            scores["zstd"] += 10
        END IF

        // Evaluate RSSI
        IF context.rssi < constants.RSSI_THRESHOLD THEN
            scores["zstd"] += 30
            scores["tinyml"] += 20
            scores["lz4"] += 10
        ELSE
            scores["lz4"] += 30
            scores["tinyml"] += 20
            scores["zstd"] += 10
        END IF

        // Evaluate sender battery health
        IF context.battery_level < constants.BATTERY_LOW THEN
            scores["lz4"] += 30
            scores["tinyml"] += 10
            scores["zstd"] += 5
        ELSE IF context.battery_level < constants.BATTERY_MODERATE THEN
            scores["lz4"] += 20
            scores["zstd"] += 15
            scores["tinyml"] += 10
        ELSE
            scores["tinyml"] += 30
            scores["zstd"] += 20
            scores["lz4"] += 10
        END IF

        // Evaluate receiver battery health (if available)
        IF context.receiver_battery_level IS NOT NULL THEN
            IF context.receiver_battery_level < constants.BATTERY_LOW THEN
                scores["lz4"] += 20
                scores["tinyml"] += 5
                scores["zstd"] += 5
            ELSE
                scores["tinyml"] += 10
                scores["zstd"] += 10
                scores["lz4"] += 5
            END IF
        END IF

        // Adjust based on ML and TinyML support
        IF context.ml_enabled AND context.tinyml_supported THEN
            scores["tinyml"] += 20
        ELSE
            scores["tinyml"] = 0
        END IF

        // Check capability flags for compression support (see /protocol-specs/capability_flags.md)
        IF NOT (context.capability_flags & BIT(21)) THEN  // Bit 21: lz4 compression
            scores["lz4"] = 0
        END IF
        IF NOT (context.capability_flags & BIT(20)) THEN  // Bit 20: zstd compression
            scores["zstd"] = 0
        END IF
        IF NOT (context.capability_flags & BIT(23)) THEN  // Bit 23: TinyML compression
            scores["tinyml"] = 0
        END IF

        RETURN scores
    CATCH error
        CALL log_message("ERROR", "Failed to score compression algorithms: " + error, capability_flags | BIT(37)) // Log as Tier 1 with multi-blockchain support
        RAISE CompressionError("Scoring failed: " + error)
    END TRY
END FUNCTION

// Function to validate TinyML model availability
FUNCTION validate_tinyml_model(model)
    TRY
        IF model IS NULL OR model IS EMPTY THEN
            RAISE ValidationError("TinyML model is null or empty")
        END IF
        // Placeholder for model integrity check (e.g., checksum, version)
        IF NOT MODEL_IS_VALID(model) THEN
            RAISE ValidationError("Invalid TinyML model")
        END IF
        RETURN True
    CATCH validation_error
        CALL log_message("ERROR", "TinyML model validation failed: " + validation_error, capability_flags | BIT(37)) // Log as Tier 1
        RAISE ValidationError(validation_error)
    END TRY
END FUNCTION

// Function to get current time in milliseconds
FUNCTION get_current_time_ms()
    TRY
        current_time = GET_SYSTEM_TIME_MILLISECONDS()
        RETURN current_time
    CATCH time_error
        CALL log_message("ERROR", "Failed to get current time: " + time_error, capability_flags | BIT(37)) // Log as Tier 1
        RAISE TimeError("Time retrieval failed: " + time_error)
    END TRY
END FUNCTION

// Function to increment a Lamport clock
FUNCTION increment_lamport_clock(current_clock)
    TRY
        IF current_clock >= constants.A DTC_MAX_LOGICAL_CLOCK THEN
            current_clock = 0  // Reset to avoid overflow
        END IF
        RETURN current_clock + 1
    CATCH clock_error
        CALL log_message("ERROR", "Failed to increment Lamport clock: " + clock_error, capability_flags | BIT(37)) // Log as Tier 1
        RAISE ClockError("Lamport clock increment failed: " + clock_error)
    END TRY
END FUNCTION

// Function to merge a vector clock
FUNCTION merge_vector_clock(local_clock, received_clock)
    TRY
        merged_clock = {}
        FOR each node_id IN UNION(local_clock.keys(), received_clock.keys())
            local_value = local_clock[node_id] ? local_clock[node_id] : 0
            received_value = received_clock[node_id] ? received_clock[node_id] : 0
            merged_clock[node_id] = MAX(local_value, received_value)
        END FOR
        RETURN merged_clock
    CATCH clock_error
        CALL log_message("ERROR", "Failed to merge vector clock: " + clock_error, capability_flags | BIT(37)) // Log as Tier 1
        RAISE ClockError("Vector clock merge failed: " + clock_error)
    END TRY
END FUNCTION

// Function to validate an ADTC vote with quantum-ready signature support
FUNCTION validate_adtc_vote(vote, signature, public_key, capability_flags)
    TRY
        IF vote IS NULL OR vote IS EMPTY THEN
            RAISE ValidationError("Vote is null or empty")
        END IF
        // Use quantum-ready signature verification if Bit 39 is set
        IF capability_flags & BIT(39) THEN
            IF NOT crypto_utils.verify_quantum_signature(vote, signature, public_key) THEN
                RAISE ValidationError("Invalid quantum-ready vote signature")
            END IF
        ELSE
            IF NOT crypto_utils.verify_signature(vote, signature, public_key) THEN
                RAISE ValidationError("Invalid vote signature")
            END IF
        END IF
        // Validate vote structure (e.g., logical clock, packet hash)
        IF NOT vote.contains("logical_clock") OR NOT vote.contains("packet_hash") THEN
            RAISE ValidationError("Invalid vote structure")
        END IF
        RETURN True
    CATCH validation_error
        CALL log_message("ERROR", "ADTC vote validation failed: " + validation_error, capability_flags | BIT(37)) // Log as Tier 1
        RAISE ValidationError(validation_error)
    END TRY
END FUNCTION

// Function to estimate real-time RTT for interplanetary nodes
FUNCTION estimate_rtt(nodes)
    TRY
        // Placeholder for RTT estimation using orbital data or historical trends
        rtt_estimate = CALCULATE_RTT(nodes)  // Based on node positions, orbital data, or historical RTT
        IF rtt_estimate < 0 OR rtt_estimate IS NULL THEN
            RAISE RTTEstimationError("Invalid RTT estimate")
        END IF
        CALL log_message("INFO", "RTT estimated: " + rtt_estimate + " seconds", capability_flags)
        RETURN rtt_estimate
    CATCH rtt_error
        CALL log_message("ERROR", "Failed to estimate RTT: " + rtt_error, capability_flags | BIT(37)) // Log as Tier 1
        RAISE RTTEstimationError("RTT estimation failed: " + rtt_error)
    END TRY
END FUNCTION

// Function to compute a hash-based proof for low-power devices
FUNCTION compute_hash_proof(data)
    TRY
        IF data IS NULL OR data IS EMPTY THEN
            RAISE HashError("Data is null or empty")
        END IF
        hash_proof = CALL crypto_utils.sha3_256(data)
        CALL log_message("INFO", "Hash-based proof computed for low-power device", capability_flags)
        RETURN hash_proof
    CATCH hash_error
        CALL log_message("ERROR", "Failed to compute hash proof: " + hash_error, capability_flags | BIT(37)) // Log as Tier 1
        RAISE HashError("Hash proof computation failed: " + hash_error)
    END TRY
END FUNCTION

// Function to check device capability for low-power optimizations
FUNCTION check_device_capability(context)
    TRY
        IF context.device_capability_level == 0 OR context.capability_flags & BIT(23) THEN  // Bit 23: Low-Energy Mode
            CALL log_message("INFO", "Simplified algorithms recommended for ultra-low-power device", context.capability_flags)
            RETURN True // Use simplified algorithms
        END IF
        RETURN False // Use full algorithms
    CATCH error
        CALL log_message("ERROR", "Failed to check device capability: " + error, capability_flags | BIT(37)) // Log as Tier 1
        RAISE CapabilityError("Device capability check failed: " + error)
    END TRY
END FUNCTION
```

---

## Notes
- Extensibility: New helper functions (increment_lamport_clock, merge_vector_clock, validate_adtc_vote, estimate_rtt, compute_hash_proof, check_device_capability) support time-delayed consensus, interplanetary RTT estimation, and low-power optimizations, with room for additional utilities.
- Performance: Optimized for high-frequency operations (e.g., generate_unique_id uses 128-bit random bytes, increment_lamport_clock uses simple increments, estimate_rtt and compute_hash_proof use lightweight calculations). check_device_capability adds minimal overhead (<1ms) for low-power checks.
- Security: validate_input, validate_tinyml_model, validate_adtc_vote, compute_hash_proof, and check_device_capability are robust against invalid inputs, logging errors for traceability.
- Error Handling: Errors are logged as Tier 1 events using Bit 37 for multi-blockchain support, ensuring traceability in crisis and interplanetary scenarios.
- Compression Support: score_compression_algorithms enables dynamic algorithm selection, while validate_tinyml_model and compute_hash_proof ensure TinyML reliability and low-power fallbacks.
- Consensus Support: increment_lamport_clock, merge_vector_clock, validate_adtc_vote, estimate_rtt, and compute_hash_proof enable logical clock management, secure vote validation, RTT estimation, and low-power PBFT validation for ADTC.
- Hardware Requirements: Hardware constraints for compression, ZKP generation, and PBFT consensus are detailed in respective modules (compression.md, zkp.md, pbft_validation.md) to guide deployment on ultra-low-power devices.


## TODO
- Implement additional helpers for data serialization or error handling.
- Replace placeholder parsing logic (JSON_PARSE, XML_PARSE) with proper implementations once available.
- Add utility for TinyML model checksum validation in validate_tinyml_model.
- Implement helper for ADTC timeout window validation.
- Develop predictive RTT modeling for estimate_rtt to enhance accuracy in interplanetary scenarios.
- Consolidate hardware requirements into a configuration file or compatibility matrix for deployment documentation.
- Map legacy 32-bit flags to 64-bit or deprecate them in relevant functions.