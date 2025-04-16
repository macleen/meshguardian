# IV/Nonce Handling Module

## Purpose:
This module manages the generation, storage, and validation of Initialization Vectors (IVs) or nonces used in cryptographic operations such as encryption and message authentication. 
Proper IV/nonce handling is essential to prevent vulnerabilities such as replay attacks, ciphertext duplication, or nonce reuse in AEAD (Authenticated Encryption with Associated Data) schemes.

## Interfaces:
- Function: generate_nonce()
  Generates a cryptographically secure and unique nonce.

- Function: validate_nonce(nonce, context)
  Ensures the nonce hasn't been reused in the same context/session.

- Function: store_nonce(nonce, context)
  Stores the used nonce to prevent reuse.

- Function: expire_nonces(context)
  Clears expired or obsolete nonces based on context (e.g., session end).

## Depends On:
- /pseudo-code/crypto/random.md — Generates cryptographically secure random bytes.
- /pseudo-code/exceptions/crypto_errors.md — For handling nonce reuse, invalid IVs.
- /pseudo-code/security/session_management.md — To associate nonces with session IDs or contexts.

## Called By:
- /pseudo-code/security/encryption.md — For encrypting data using schemes like AES-GCM, ChaCha20-Poly1305.
- /pseudo-code/security/packet_protection.md — For securing packets in MeshGuardian.
- /pseudo-code/security/forward_secrecy.md — Where nonce uniqueness ensures session-level integrity.

## Used In:
- Use Case 5.15: Aid Relays — Prevents replay in packet-level encryption.
- Use Case 5.16: Emergency Chat — Ensures nonce uniqueness per message/session.


## Pseudo-code
```pseudocode

/* Function: generate_nonce */
FUNCTION generate_nonce(context, algorithm)
    sequence = GET_NEXT_SEQUENCE(context)
    random_part = RANDOM_BYTES(8)
    nonce = sequence + random_part
    // Enforce algorithm-specific sizing
    IF algorithm == "AES-GCM" THEN
        nonce = nonce[0:12]  // 12 bytes for AES-GCM
    ELSE IF algorithm == "ChaCha20-Poly1305" THEN
        nonce = nonce[0:12]  // 12 bytes for ChaCha20-Poly1305
    END IF
    RETURN nonce

/* Function: validate_nonce */
FUNCTION validate_nonce(nonce, context, timestamp, algorithm)
    IF timestamp < CURRENT_TIME - MAX_CLOCK_SKEW THEN
        LOG_SECURITY_EVENT("Stale nonce detected", context, nonce)
        RAISE StaleNonceError("Nonce is too old")
    END IF
    IF NONCE_EXISTS(context, nonce) THEN
        LOG_SECURITY_EVENT("Nonce reuse attempt", context, nonce)
        RAISE NonceReuseError("Nonce already used in this context")
    END IF
    // Validate nonce size for the algorithm
    IF algorithm == "AES-GCM" AND LENGTH(nonce) != 12 THEN
        RAISE InvalidNonceError("Invalid nonce size for AES-GCM")
    END IF

/* Function: store_nonce */
FUNCTION store_nonce(nonce, context, expiration_time)
    IF IS_PERSISTENT_CONTEXT(context) THEN
        encrypted_nonce = ENCRYPT(nonce)
        APPEND_TO_NONCE_LOG(context, encrypted_nonce, expiration_time)
    ELSE
        APPEND_TO_NONCE_LOG(context, nonce, expiration_time)
    END IF

/* Function: expire_nonces */
FUNCTION expire_nonces(context)
    DELETE_EXPIRED_NONCES(context)
    LOG("Expired nonces for context: " + context)
```

---

## Notes:
- Nonce Size: Enforced based on the cryptographic algorithm (e.g., 12 bytes for AES-GCM).
- Uniqueness: Hybrid generation (sequence + random) ensures uniqueness and unpredictability.
- Replay Protection: Timestamp validation with clock skew tolerance prevents replay attacks.
- Storage: Encrypted storage for persistent contexts enhances security.
Compliance: Aligns with NIST SP 800-38D for AES-GCM nonce requirements, FIPS 140-3 for RNG, and GDPR Art. 32 for pseudonymization via context binding.
- Key Rotation: Nonces are reset after key rotation to maintain security; ensure nonce logs are cleared or re-indexed post-rotation.

TODO:
- Implement rolling nonce windows for streaming applications.
- Add persistent nonce cache with memory + disk fallback.

