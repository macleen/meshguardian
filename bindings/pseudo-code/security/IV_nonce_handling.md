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
FUNCTION generate_nonce(context)
    sequence = GET_NEXT_SEQUENCE(context)
    random_part = RANDOM_BYTES(8)
    nonce = sequence + random_part
    RETURN nonce

FUNCTION validate_nonce(nonce, context)
    IF NONCE_EXISTS(context, nonce) THEN
        RAISE NonceReuseError("Nonce already used in this context")
    END IF

FUNCTION store_nonce(nonce, context, expiration_time)
    APPEND_TO_NONCE_LOG(context, nonce, expiration_time)

FUNCTION expire_nonces(context)
    DELETE_EXPIRED_NONCES(context)
    LOG("Expired nonces for context: " + context)
```

---

## Notes:
- Nonce Size: Follow cryptographic algorithm requirements (e.g., 96 bits for AES-GCM).
- Uniqueness: Never reuse a nonce with the same key; reuse can completely break security guarantees.
- Replay Protection: Validate nonces to detect repeated or delayed transmissions.
- Expiration Strategy: Expire stored nonces after a session ends or after timeout.
- Storage Strategy: Use in-memory structures for short-lived contexts, persistent DB for long-term ones.

TODO:
- Implement rolling nonce windows for streaming applications.
- Add persistent nonce cache with memory + disk fallback.
*/
