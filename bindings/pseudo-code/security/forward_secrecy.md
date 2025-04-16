# Forward Secrecy Module

## Purpose:
The Forward Secrecy module enables secure session key generation in such a way that compromise of long-term keys does not compromise past communications. 
This is achieved through ephemeral key exchange mechanisms (e.g., Diffie-Hellman or ECDH) that generate a new session key per communication session.

## Interfaces:
- Function: generate_ephemeral_keypair()
  Generates a temporary keypair (private/public) used for secure key exchange.

- Function: derive_shared_secret(own_private_key, peer_public_key)
  Uses Diffie-Hellman key agreement to compute a shared secret.

- Function: derive_session_key(shared_secret, context_info)
  Derives a session key from the shared secret using a KDF.

- Function: expire_session_key(session_id)
  Deletes a session key after use to maintain forward secrecy.

## Depends On:
- /pseudo-code/crypto/key_derivation_functions.md — Supplies KDFs for deriving session keys.
- /pseudo-code/crypto/random.md — Generates cryptographic-quality random ephemeral keys.
- /pseudo-code/exceptions/security_errors.md — Handles errors related to key exchange or derivation.

## Called By:
- /pseudo-code/security/auth.md — To establish ephemeral sessions during login/auth handshake.
- /pseudo-code/networking/packet_encryption.md — For encrypting packets using session keys.
- /pseudo-code/security/key_management.md — For temporary key lifecycle operations.

## Used In:
- Use Case 5.15: Aid Relays — Ensures old messages remain secure even if node keys are compromised.
- Use Case 5.16: Emergency Chat — Provides per-session encryption for real-time communication.

## Pseudo-code
```pseudocode
/* Function: generate_ephemeral_keypair */
FUNCTION generate_ephemeral_keypair()
    private_key = RANDOM_PRIVATE_KEY()
    public_key = COMPUTE_PUBLIC_KEY(private_key)
    RETURN { "private": private_key, "public": public_key }

/* Function: derive_shared_secret */
FUNCTION derive_shared_secret(own_private_key, peer_public_key)
    shared_secret = ECDH(own_private_key, peer_public_key)
    RETURN shared_secret

/* Function: derive_session_key */
FUNCTION derive_session_key(shared_secret, context_info)
    session_key = KDF(shared_secret, context_info)
    STORE_SESSION_KEY(context_info.session_id, session_key)
    RETURN session_key

/* Function: expire_session_key */
FUNCTION expire_session_key(session_id)
    DELETE_SESSION_KEY(session_id)
    LOG("Session key expired for ID: " + session_id)

```

---

## Notes:
- Ephemeral Key Generation: Each session uses a freshly generated keypair to ensure uniqueness.
- Shared Secret Calculation: The shared secret is never stored—only the derived session key.
- Session Key Expiry: Always delete session keys post-session to prevent reuse or leakage.
- Key Derivation: Use HMAC-based KDFs (HKDF) for better resistance against cryptanalysis.
- Forward Secrecy Guarantees: Even if a device is later compromised, past encrypted traffic remains safe.

TODO:
- Add session timeout enforcement.
- Support pre-authenticated key exchange for constrained devices.
