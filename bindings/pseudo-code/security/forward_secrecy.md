# Forward Secrecy Module

## Purpose:
The Forward Secrecy Module ensures that session keys are generated securely such that the compromise of long-term keys does not affect past communications. This is achieved using ephemeral key exchanges (e.g., ECDH with Curve25519) and robust key derivation (HKDF), tailored for MeshGuardian's decentralized mesh network.

## Interfaces:
- Function: generate_ephemeral_keypair()
Generates a temporary Curve25519 keypair for secure key exchange.

- Function: derive_shared_secret(own_private_key, peer_public_key)
Computes a shared secret using Curve25519 ECDH.

- Function: derive_session_key(shared_secret, context_info)
Derives a session key from the shared secret using HKDF.

- Function: expire_session_key(session_id)
Securely deletes a session key after use.

- Function: check_rekeying_conditions(session)
Triggers rekeying based on session duration or data volume.

- Function: rotate_group_key(group_id)
Rotates group keys for mesh network group communications.

- Function: handle_offline_node(node_id, message)
Queues messages for offline nodes during key transitions.

## Depends On:
- /pseudo-code/crypto/key_derivation_functions.md — Provides HKDF for session key derivation.

- /pseudo-code/crypto/random.md — Generates cryptographically secure random keys.

- /pseudo-code/exceptions/security_errors.md — Manages key exchange/derivation errors.

## Called By:
 -/pseudo-code/security/auth.md — Establishes ephemeral sessions during authentication.

- /pseudo-code/networking/packet_encryption.md — Encrypts packets with session keys.

- /pseudo-code/security/key_management.md — Manages temporary key lifecycles.

## Used In:
- Use Case 5.15: Aid Relays — Ensures old messages remain secure even if node keys are compromised.
- Use Case 5.16: Emergency Chat — Provides per-session encryption for real-time communication.

## Pseudo-code
```pseudocode
/* Function: generate_ephemeral_keypair */
/* Function: generate_ephemeral_keypair */
FUNCTION generate_ephemeral_keypair()
    private_key = X25519_GENERATE_PRIVATE_KEY()
    public_key = X25519_COMPUTE_PUBLIC_KEY(private_key)
    RETURN { "private": private_key, "public": public_key }

/* Function: derive_shared_secret */
FUNCTION derive_shared_secret(own_private_key, peer_public_key)
    shared_secret = X25519_ECDH(own_private_key, peer_public_key)
    RETURN shared_secret

/* Function: derive_session_key */
FUNCTION derive_session_key(shared_secret, context_info)
    session_key = HKDF(shared_secret, context_info || "meshguardian_session_" || session_id)
    STORE_SESSION_KEY(context_info.session_id, session_key)
    RETURN session_key

/* Function: expire_session_key */
FUNCTION expire_session_key(session_id)
    session_key = RETRIEVE_SESSION_KEY(session_id)
    SECURE_ERASE(session_key)
    DELETE_SESSION_KEY(session_id)
    LOG("Session key securely erased for ID: " + session_id)

/* Function: check_rekeying_conditions */
FUNCTION check_rekeying_conditions(session)
    IF session.duration > MAX_SESSION_TIME OR session.bytes_encrypted > REKEY_BYTES_LIMIT THEN
        CALL initiate_rekeying(session)
    END IF

/* Function: rotate_group_key */
FUNCTION rotate_group_key(group_id)
    new_group_key = GENERATE_NEW_GROUP_KEY()
    UPDATE_TREEKEM(group_id, new_group_key)
    BROADCAST_KEY_UPDATE(group_id)
    LOG("Group key rotated for group: " + group_id)

/* Function: handle_offline_node */
FUNCTION handle_offline_node(node_id, message)
    QUEUE_MESSAGE(node_id, message)
    LOG("Message queued for offline node: " + node_id)
```

---

## Notes:
- Ephemeral Key Generation: Uses Curve25519 for high-security, efficient keypairs.
- Key Derivation: HKDF with unique context strings prevents key reuse.
- Session Key Rotation: Rekeying triggered by time (MAX_SESSION_TIME) or data (REKEY_BYTES_LIMIT).
- Secure Deletion: Keys are zeroed out using SECURE_ERASE to prevent recovery.
- **Mesh-Specific Features:**
  - Group key rotation via TreeKEM ensures forward secrecy in group chats.
  - Message queuing supports asynchronous communication for offline nodes.
- Forward Secrecy: Past traffic remains secure even if long-term keys are compromised.

## TODO:
- Add session timeout enforcement.
- Support pre-authenticated key exchange for constrained devices.
