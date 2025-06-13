# Key Management Module

## Purpose
The Key Management module is responsible for managing the lifecycle of cryptographic keys used in the system. It ensures secure generation, rotation, revocation, and storage of keys, critical for encryption, decryption, and digital signatures, with robust validation to prevent invalid keys.

## Interfaces
- **generate_key(parameters)**: Generates a new cryptographic key based on specified parameters.  
- **rotate_key(key_id, new_parameters, grace_period)**: Replaces an existing key with a new one, with a grace period.  
- **revoke_key(key_id, reason)**: Invalidates a key and propagates revocation.  
- **store_key(key_id, key)**: Securely stores a key in a protected vault.  
- **retrieve_key(key_id)**: Retrieves a stored key with secure memory handling.  

## Depends On
- **/pseudo-code/crypto/encryption.md**: Provides cryptographic functions for key generation and operations.  
- **/pseudo-code/storage/secure_vault.md**: Manages the secure storage of keys.  
- **/pseudo-code/exceptions/security_errors.md**: Defines custom exceptions for security-related errors.  
- **/pseudo-code/audit/audit_trail.md**: Logs key management operations for auditing.  
- **/pseudo-code/logging/logger.md**: Logs key operations and errors for traceability.  

## Called By
- **/pseudo-code/security/encryption.md**: Obtains and manages keys for encryption/decryption.  
- **/pseudo-code/security/auth.md**: Manages keys for authentication processes.  

## Used In
- **Use Case 5.15: Aid Relays**: Manages keys for encrypting supply tracking data in crisis zones.  
- **Use Case 5.16: Emergency Chat**: Handles keys for securing real-time communications.  

## Pseudocode
```pseudocode
// Function to validate key format and integrity
FUNCTION validate_key(key, parameters)
    IF key IS NULL OR key IS EMPTY THEN
        RAISE KeyError("Generated key is null or empty")
    END IF
    expected_length = parameters.key_length
    IF LENGTH(key) != expected_length THEN
        RAISE KeyError("Invalid key length: expected " + expected_length + " bytes")
    END IF
    IF CALC_ENTROPY(key) < MIN_ENTROPY THEN
        RAISE KeyError("Insufficient key entropy")
    END IF
    RETURN TRUE
END FUNCTION

// Function: generate_key
FUNCTION generate_key(parameters, capability_flags)
    IF NOT VALIDATE_KEY_PARAMETERS(parameters) THEN
        RAISE WeakKeyParametersError("Invalid key parameters")
    END IF
    // Check ML feature flags if applicable
    IF parameters.purpose == "ml_protocol_selection" THEN
        IF NOT (capability_flags & ML_PROTOCOL_SELECTION_FLAG) THEN
            RAISE FeatureDisabledError("ML protocol selection is not enabled")
        END IF
        parameters.ml_enabled = TRUE
    ELSE IF parameters.purpose == "ml_failure_prediction" THEN
        IF NOT (capability_flags & ML_FAILURE_PREDICTION_FLAG) THEN
            RAISE FeatureDisabledError("ML failure prediction is not enabled")
        END IF
        parameters.ml_enabled = TRUE
    END IF
    TRY
        key = CRYPTO_GENERATE_KEY(parameters)
        CALL validate_key(key, parameters)
        key_id = GENERATE_UNIQUE_ID()
        CALL store_key(key_id, key)
        CALL log_message("INFO", "Key generated with ID: " + key_id + " for purpose: " + parameters.purpose, capability_flags)
        RETURN key_id
    CATCH crypto_error
        CALL log_message("ERROR", "Key generation failed: " + crypto_error, capability_flags | LOG_TIER_1_FLAG)  // Log as Tier 1
        RAISE KeyError("Key generation failed: " + crypto_error)
    END TRY
END FUNCTION

// Function: rotate_key
FUNCTION rotate_key(key_id, new_parameters, grace_period)
    IF NOT VALIDATE_KEY_PARAMETERS(new_parameters) THEN
        RAISE WeakKeyParametersError("Invalid key parameters for rotation")
    END IF
    TRY
        new_key = CRYPTO_GENERATE_KEY(new_parameters)
        CALL validate_key(new_key, new_parameters)
        old_key = CALL retrieve_key(key_id)
        CALL store_key(key_id + ".old", old_key)
        CALL store_key(key_id, new_key)
        SCHEDULE_DELETION(key_id + ".old", grace_period)
        CALL log_message("INFO", "Key rotated for ID: " + key_id + " by " + CALLER, capability_flags)
        RETURN success_status
    CATCH key_error
        CALL log_message("ERROR", "Key rotation failed: " + key_error, capability_flags | BIT 15)  // Log as Tier 1
        RAISE KeyError("Key rotation failed: " + key_error)
    END TRY
END FUNCTION

// Function: revoke_key
FUNCTION revoke_key(key_id, reason)
    MARK_KEY_AS_REVOKED(key_id)
    CALL delete_key(key_id)
    PUBLISH_REVOCATION(key_id, reason)
    CALL log_message("INFO", "Key revoked for ID: " + key_id + " by " + CALLER + " for reason: " + reason, capability_flags)
END FUNCTION

// Function: store_key
FUNCTION store_key(key_id, key)
    IF HAS_HSM THEN
        HSM_STORE_KEY(key_id, key)
    ELSE
        encrypted_key = ENCRYPT(key)
        STORE_IN_VAULT(key_id, encrypted_key)
    END IF
END FUNCTION

// Function: retrieve_key
FUNCTION retrieve_key(key_id)
    TRY
        IF HAS_HSM THEN
            key = HSM_RETRIEVE_KEY(key_id)
        ELSE
            encrypted_key = RETRIEVE_FROM_VAULT(key_id)
            IF encrypted_key IS NULL THEN
                RAISE KeyNotFoundError("No key found for ID: " + key_id)
            END IF
            key = DECRYPT(encrypted_key)
        END IF
        IF key IS NULL OR key IS EMPTY THEN
            RAISE KeyError("Retrieved key is null or empty")
        END IF
        RETURN key
    CATCH decrypt_error
        CALL log_message("ERROR", "Key retrieval failed: " + decrypt_error, capability_flags | BIT 15)  // Log as Tier 1
        RAISE KeyError("Key retrieval failed: " + decrypt_error)
    FINALLY
        SECURE_WIPE(encrypted_key)
    END TRY
END FUNCTION
```

---

## Notes:
- Key Rotation: Automated rotation includes a grace period to avoid interruptions, with validation for new keys.
- Revocation: Revocations are propagated via PUBLISH_REVOCATION for network-wide consistency.
- Storage: HSM integration provides enhanced protection; fallback to encrypted vault if HSM is unavailable.
- Error Handling: Validates keys for format, length, and entropy, raising KeyError for invalid keys, with logging via logger.md. Secure memory wiping prevents leakage.
- Compliance: Aligns with NIST SP 800-57, FIPS 140-3 Level 2+, and PCI DSS 3.6 for key management. 
- **64-bit Capability Flags Transition**: This module has been updated to support 64-bit capability flags. The `capability_flags` parameter is a 64-bit integer, and hardcoded bit positions (e.g., BIT 15) have been replaced with symbolic constants (e.g., LOG_TIER_1_FLAG) defined in `/pseudo-code/shared/constants.md`. Ensure logging dependencies handle 64-bit integers correctly.

## TODO
- Implement post-quantum crypto readiness and distributed key generation.  
