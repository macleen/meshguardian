# Key Separation Module

## Purpose:
The Key Separation module ensures that cryptographic keys are used only for their intended purpose. 
This prevents cross-use of keys between different security domains (e.g., encryption, authentication, signing), reducing attack surfaces and preserving cryptographic hygiene.

## Interfaces:
- Function: generate_key(context)
  Generates a key uniquely bound to a specific usage context (e.g., "encryption", "auth").

- Function: use_key(key_id, purpose)
  Validates that a key is used only for its assigned purpose before performing operations.

- Function: tag_key(key_id, purpose)
  Binds a purpose tag to a newly generated or imported key.

- Function: validate_purpose(key_id, requested_purpose)
  Checks if the key is allowed to be used for the given purpose.

## Depends On:
- /pseudo-code/security/key_management.md — Stores and retrieves keys.
- /pseudo-code/security/key_derivation.md — Derives keys with embedded usage context.
- /pseudo-code/exceptions/security_errors.md — Raises exceptions on policy violations.

## Called By:
- /pseudo-code/security/encryption.md — Ensures encryption keys are not reused for MAC or signing.
- /pseudo-code/security/auth.md — Separates keys used for user authentication from those used for data protection.
- /pseudo-code/networking/packet_encryption.md — Uses different keys per protocol layer.

## Used In:
- Use Case 5.15: Aid Relays — Prevents encryption keys from being misused across message types.
- Use Case 5.16: Emergency Chat — Enforces separation between message encryption and session signing.

## Pseudo-code
```pseudocode
/* Function: generate_key */
FUNCTION generate_key(context)
    master_key = GET_MASTER_KEY()
    key = HKDF(master_key, salt=context, info="key_separation")
    key_id = GENERATE_UNIQUE_ID()
    CALL tag_key(key_id, context)
    CALL store_key(key_id, key)
    RETURN key_id

/* Function: tag_key */
FUNCTION tag_key(key_id, purpose)
    IF purpose IN ["ml_protocol_selection", "ml_failure_prediction"]
        // Ensure ML-specific keys are tagged for secure model operations
        SET_KEY_METADATA(key_id, "ml_enabled", TRUE)
    END IF
    IF HAS_HSM THEN
        HSM_SET_USAGE_POLICY(key_id, purpose)
    END IF
    SET_KEY_METADATA(key_id, "purpose", purpose)
    LOG("Key tagged with purpose: " + purpose + " for ID: " + key_id)

/* Function: validate_purpose */
FUNCTION validate_purpose(key_id, requested_purpose)
    stored_purpose = GET_KEY_METADATA(key_id, "purpose")
    IF stored_purpose != requested_purpose THEN
        LOG("Key usage violation attempt for ID: " + key_id)
        RAISE KeyUsageViolationError("Key not valid for purpose: " + requested_purpose)
    END IF

/* Function: use_key */
FUNCTION use_key(key_id, purpose)
    CALL validate_purpose(key_id, purpose)
    key = CALL retrieve_key(key_id)
    LOG("Key used for purpose: " + purpose + " with ID: " + key_id)
    RETURN key

```

---

## Notes:
- Cryptographic Best Practice: Each key should have a singular, tightly-scoped purpose to prevent unintended collisions and leaks.
- Tagging Metadata: Keys are labeled with tags such as "enc", "mac", "sig", or protocol-specific roles.
- Access Control: Usage is always gated through purpose-checking logic.
- Hierarchical Derivation: When using master keys, derive separate children with explicit purpose input into the KDF.
- Hardware Enforcement: If using an HSM, bind usage policies at the hardware level.

## TODO:
- Integrate hierarchical deterministic key derivation (HKDF) with tagged context info.
- Add auditing for key usage by purpose.
- Add purpose constraints to secure vault interface.
