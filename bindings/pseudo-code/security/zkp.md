# Zero-Knowledge Proof (ZKP) Module

## Purpose
The Zero-Knowledge Proof (ZKP) module enables privacy-preserving operations in MeshGuardian by generating and verifying ZKPs for secure authentication, data validation, and audit trail logging. It supports multiple ZKP schemes (zk-SNARKs, zk-STARKs, Bulletproofs) to ensure flexibility across use cases, including crisis zones and interplanetary missions. The module is optimized for ultra-low-power devices (e.g., IoT nodes in crisis zones) by prioritizing lightweight ZKP schemes and limiting generation frequency, with clear hardware requirements to guide deployment. It integrates with the audit trail for traceability and supports Low-Energy Mode (Bit 24) for resource-constrained scenarios.

## Interfaces
- **generate_zkp(data, scheme, context)**: Generates a ZKP for the given data using the specified scheme, returning the proof.
- **verify_zkp(proof, data, scheme)**: Verifies a ZKP against the provided data and scheme, returning a boolean result.

## Depends On
- **/pseudo-code/crypto/crypto_utils.md**: Provides cryptographic primitives for ZKP generation and verification.
- **/pseudo-code/logging/logger.md**: Logs ZKP events and errors.
- **/pseudo-code/constants.md**: Defines ZKP scheme codes, computation time thresholds, and generation intervals.
- **/pseudo-code/audit_trail.md**: Logs ZKP outcomes for traceability.

## Called By
- /pseudo-code/audit_trail.md: Generates ZKPs for Tier 1 blockchain logging.
- /pseudo-code/security/auth.md: Uses ZKPs for privacy-preserving authentication.
- /pseudo-code/consensus/consensus.md: Applies ZKPs for secure vote validation in ADTC.

## Used In
- Use Case 5.15: Aid Relays: Ensures privacy in supply tracking data in crisis zones.
- Use Case 5.16: Emergency Chat: Protects user identities in message exchanges.
- Use Case 5.1.1: Mars Rover Data Relay: Secures interplanetary data validation.

## Pseudocode
```pseudocode
// Structure for ZKP context
STRUCT ZKPContext
    data: BYTE_ARRAY           // Data to prove
    capability_flags: BITFIELD // 32-bit Capability Flags
    device_capability_level: INTEGER // 0 = minimal, 1 = moderate, 2 = high
    event_count: INTEGER       // Counter for tracking ZKP generation frequency
END STRUCT

// Function to generate a ZKP
FUNCTION generate_zkp(data, scheme, context: ZKPContext) RETURNS BYTE_ARRAY
    TRY
        IF data IS NULL THEN
            RAISE ZKPError("Data is null")
        END IF

        // Prioritize lightweight scheme for ultra-low-power devices
        IF context.device_capability_level == 0 OR context.capability_flags & BIT_24 THEN
            IF scheme == constants.BULLETPROOF_CODE THEN
                CALL log_message("INFO", "Selected Bulletproofs for ultra-low-power device", context.capability_flags)
            ELSE
                CALL log_message("INFO", "Falling back to hash-based proof for ultra-low-power device", context.capability_flags)
                hash_proof = CALL crypto_utils.sha3_256(data)
                RETURN hash_proof // Fallback to SHA3-256 hash
            END IF
        END IF

        // Limit ZKP generation frequency for low-power devices
        IF context.device_capability_level == 1 AND context.event_count % constants.ZKP_GENERATION_INTERVAL != 0 THEN
            CALL log_message("INFO", "Skipped ZKP generation, using hash-based proof", context.capability_flags)
            hash_proof = CALL crypto_utils.sha3_256(data)
            RETURN hash_proof // Fallback to SHA3-256 hash
        END IF

        // Increment event counter for generation frequency
        context.event_count = context.event_count + 1

        // Generate ZKP based on scheme
        proof = NULL
        start_time = CALL get_current_time_ms()
        CASE scheme
            WHEN constants.ZK_SNARK_CODE:
                proof = ZK_SNARK_GENERATE(data)
            WHEN constants.ZK_STARK_CODE:
                proof = ZK_STARK_GENERATE(data)
            WHEN constants.BULLETPROOF_CODE:
                proof = BULLETPROOF_GENERATE(data)
            DEFAULT:
                RAISE ZKPError("Unsupported ZKP scheme: " + scheme)
        END CASE
        end_time = CALL get_current_time_ms()
        computation_time = end_time - start_time

        // Check computation time
        IF computation_time > constants.ZKP_MAX_COMPUTATION_TIME THEN
            CALL log_message("WARNING", "ZKP generation too slow: " + computation_time + "ms, falling back to hash-based proof", context.capability_flags)
            hash_proof = CALL crypto_utils.sha3_256(data)
            RETURN hash_proof // Fallback to SHA3-256 hash
        END IF

        IF proof IS NULL THEN
            RAISE ZKPError("ZKP generation failed for scheme: " + scheme)
        END IF
        CALL log_message("INFO", "ZKP generated with scheme: " + scheme + ", computation_time: " + computation_time + "ms", context.capability_flags)
        CALL log_event("ZKP_Generated", {
            "scheme": scheme,
            "data_hash": CALL crypto_utils.sha3_256(data),
            "computation_time": computation_time
        }, context.capability_flags | BIT_15) // Log as Tier 1
        RETURN proof
    CATCH error
        CALL log_message("ERROR", "ZKP generation failed: " + error, context.capability_flags | BIT_15) // Log as Tier 1
        RAISE error
    END TRY
END FUNCTION

// Function to verify a ZKP
FUNCTION verify_zkp(proof, data, scheme)
    TRY
        IF proof IS NULL OR data IS NULL THEN
            RAISE ZKPError("Proof or data is null")
        END IF
        result = FALSE
        IF scheme == constants.HASH_PROOF_CODE THEN
            // Verify hash-based proof for low-power devices
            expected_hash = CALL crypto_utils.sha3_256(data)
            result = (proof == expected_hash)
        ELSE
            // Verify ZKP based on scheme
            CASE scheme
                WHEN constants.ZK_SNARK_CODE:
                    result = ZK_SNARK_VERIFY(proof, data)
                WHEN constants.ZK_STARK_CODE:
                    result = ZK_STARK_VERIFY(proof, data)
                WHEN constants.BULLETPROOF_CODE:
                    result = BULLETPROOF_VERIFY(proof, data)
                DEFAULT:
                    RAISE ZKPError("Unsupported ZKP scheme: " + scheme)
            END CASE
        END IF
        IF NOT result THEN
            CALL log_message("ERROR", "ZKP verification failed for scheme: " + scheme, capability_flags | BIT_15) // Log as Tier 1
            RETURN FALSE
        END IF
        CALL log_message("INFO", "ZKP verified with scheme: " + scheme, capability_flags)
        CALL log_event("ZKP_Verified", {
            "scheme": scheme,
            "data_hash": CALL crypto_utils.sha3_256(data)
        }, capability_flags | BIT_15) // Log as Tier 1
        RETURN TRUE
    CATCH error
        CALL log_message("ERROR", "ZKP verification failed: " + error, capability_flags | BIT_15) // Log as Tier 1
        RAISE error
    END TRY
END FUNCTION

```

---


## Notes
- ZKP Schemes: Supports zk-SNARKs (high compactness, trusted setup), zk-STARKs (quantum-resistant, no trusted setup), and Bulletproofs (lightweight, no trusted setup). For ultra-low-power devices (Device Capability Level 0 or Bit 24 = 1), prioritizes Bulletproofs or falls back to SHA3-256 hash-based proofs to minimize computational overhead.
- Energy Optimization: Limits ZKP generation to every 10 events (ZKP_GENERATION_INTERVAL) on low-power devices (Device Capability Level 1), using hash-based proofs to reduce energy consumption. Falls back to hash-based proofs if generation exceeds 500ms (ZKP_MAX_COMPUTATION_TIME).
- Hardware Requirements:
    -- Bulletproofs: Minimum 32 MHz CPU, 64 KB RAM, 200 µW power consumption; suitable for low-power IoT nodes.
    -- zk-SNARKs: Minimum 80 MHz CPU, 128 KB RAM, 500 µW power consumption; recommended for moderate-capability devices.
    -- zk-STARKs: Minimum 120 MHz CPU, 256 KB RAM, 1 mW power consumption; suitable for high-capability devices only.
    -- Hash-based Proof (SHA3-256): Minimum 16 MHz CPU, 32 KB RAM, 100 µW power consumption; fallback for ultra-low-power devices.
- Performance: Bulletproofs offer lower computational overhead (e.g., <100ms for 1KB data) compared to zk-SNARKs (~200ms) or zk-STARKs (~500ms). Hash-based proofs are fastest (<10ms) but less privacy-preserving.
- Security: Ensures privacy and integrity via ZKPs, with post-quantum signatures (Bit 4) for future-proofing. Fallback hash-based proofs maintain integrity but sacrifice zero-knowledge properties.
- Audit Trail: ZKP generation and verification logged as Tier 1 events for traceability in crisis and interplanetary scenarios.
- Edge Cases: Handles unsupported schemes, low-power constraints, and slow computations by falling back to hash-based proofs or skipping ZKP generation.

## TODO
- Optimize ZKP generation for additional lightweight schemes (e.g., PLONK).
- Add support for hardware-accelerated ZKP computation via the Pluggable Protocol Engine.
- Integrate hardware requirements into a configuration file or compatibility matrix for deployment documentation.