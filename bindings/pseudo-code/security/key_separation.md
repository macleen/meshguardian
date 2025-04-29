# Key Separation Module

## Purpose
The Key Separation module ensures that cryptographic keys are used only for their intended purpose, preventing cross-use between security domains (e.g., encryption, authentication, signing). It reduces attack surfaces and preserves cryptographic hygiene, with validation to ensure key integrity.

## Interfaces
- **generate_key(context)**: Generates a key uniquely bound to a specific usage context.  
- **use_key(key_id, purpose)**: Validates that a key is used only for its assigned purpose.  
- **tag_key(key_id, purpose)**: Binds a purpose tag to a newly generated or imported key.  
- **validate_purpose(key_id, requested_purpose)**: Checks if the key is allowed for the given purpose.  

## Depends On
- **/pseudo-code/security/key_management.md**: Stores and retrieves keys.  
- **/pseudo-code/security/secure_key_derivation.md**: Derives keys with embedded usage context.  
- **/pseudo-code/exceptions/security_errors.md**: Raises exceptions on policy violations.  
- **/pseudo-code/logging/logger.md**: Logs key operations and errors for traceability.  

## Called By
- **/pseudo-code/security/encryption.md**: Ensures encryption keys are not reused for MAC or signing.  
- **/pseudo-code/security/auth.md**: Separates keys for user authentication from data protection.  
- **/pseudo-code/networking/packet_encryption.md**: Uses different keys per protocol layer.  

## Used In
- **Use Case 5.15: Aid Relays**: Prevents encryption keys from being misused across message types.  
- **Use Case 5.16: Emergency Chat**: Enforces separation between message encryption and session signing.  

## Pseudocode
```pseudocode
// Function to validate derived key
FUNCTION validate_key(key)
    IF key IS NULL OR key IS EMPTY THEN
        RAISE KeyError("Derived key is null or empty")
    END IF
    IF LENGTH(key) != 32 THEN
        RAISE KeyError("Invalid key length: expected 32 bytes")
    END IF
    IF CALC_ENTROPY(key) < MIN_ENTROPY THEN
        RAISE KeyError("Insufficient key entropy")
    END IF
    RETURN TRUE
END FUNCTION

// Function: generate_key
FUNCTION generate_key(context)
    master_key = GET_MASTER_KEY()
    IF master_key IS NULL OR master_key IS EMPTY THEN
        RAISE KeyError("Master key is null or empty")
    END IF
    TRY
        key = HKDF(master_key, salt=context, info="key_separation")
        CALL validate_key(key)
        key_id = GENERATE_UNIQUE_ID()
        CALL tag_key(key_id, context)
        CALL store_key(key_id, key)
        CALL log_message("INFO", "Key generated for context: " + context + " with ID: " + key_id, capability_flags)
        RETURN key_id
    CATCH hkdf_error
        CALL log_message("ERROR", "Key generation failed: " + hkdf_error, capability_flags | BIT 15)  // Log as Tier 1
        RAISE KeyError("Key generation failed: " + hkdf_error)
    END TRY
END FUNCTION

// Function: tag_key
FUNCTION tag_key(key_id, purpose)
    IF purpose IN ["ml_protocol_selection", "ml_failure_prediction"] THEN
        SET_KEY_METADATA(key_id, "ml_enabled", TRUE)
    END IF
    IF HAS_HSM THEN
        HSM_SET_USAGE_POLICY(key_id, purpose)
    END IF
    SET_KEY_METADATA(key_id, "purpose", purpose)
    CALL log_message("INFO", "Key tagged with purpose: " + purpose + " for ID: " + key_id, capability_flags)
END FUNCTION

// Function: validate_purpose
FUNCTION validate_purpose(key_id, requested_purpose)
    stored_purpose = GET_KEY_METADATA(key_id, "purpose")
    IF stored_purpose != requested_purpose THEN
        CALL log_message("ERROR", "Key usage violation attempt for ID: " + key_id + " with purpose: " + requested_purpose, capability_flags | BIT 15)  // Log as Tier 1
        RAISE KeyUsageViolationError("Key not valid for purpose: " + requested_purpose)
    END IF
END FUNCTION

// Function: use_key
FUNCTION use_key(key_id, purpose)
    CALL validate_purpose(key_id, purpose)
    key = CALL retrieve_key(key_id)
    CALL log_message("INFO", "Key used for purpose: " + purpose + " with ID: " + key_id, capability_flags)
    RETURN key
END FUNCTION

```

---

## Notes:
- Cryptographic Best Practice: Keys have singular, tightly-scoped purposes to prevent collisions and leaks, with validation for integrity.
- Tagging Metadata: Keys are labeled with tags (e.g., "enc", "mac", "sig") or protocol-specific roles.
- Access Control: Usage is gated through purpose-checking logic, with error handling for violations.
- Hierarchical Derivation: Derives separate keys from master keys with explicit purpose input into HKDF, validated for format and entropy.
- Error Handling: Validates keys and purposes, raising KeyError or KeyUsageViolationError, with logging via logger.md.
-Hardware Enforcement: Binds usage policies at the hardware level if using an HSM.

## TODO
- Integrate hierarchical deterministic key derivation (HKDF) with tagged context info.
- Add auditing for key usage by purpose. 
- Add purpose constraints to secure vault interface.
