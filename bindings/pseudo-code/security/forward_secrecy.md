# Forward Secrecy Module

## Purpose
The Forward Secrecy Module ensures that session keys are generated securely such that the compromise of long-term keys does not affect past communications. This is achieved using ephemeral key exchanges (e.g., ECDH with Curve25519) and robust key derivation (HKDF), tailored for MeshGuardian's decentralized mesh network, with error handling for invalid keys.

## Interfaces
- **generate_ephemeral_keypair()**: Generates a temporary Curve25519 keypair for secure key exchange.  
- **derive_shared_secret(own_private_key, peer_public_key)**: Computes a shared secret using Curve25519 ECDH.  
- **derive_session_key(shared_secret, context_info, packet)**: Derives a session key from the shared secret using HKDF.  
- **expire_session_key(session_id)**: Securely deletes a session key after use.  
- **check_rekeying_conditions(session, packet)**: Triggers rekeying based on session duration or data volume.  
- **rotate_group_key(group_id)**: Rotates group keys for mesh network group communications.  
- **handle_offline_node(node_id, message)**: Queues messages for offline nodes during key transitions.  

## Depends On
- **/pseudo-code/crypto/key_derivation_functions.md**: Provides HKDF for session key derivation.  
- **/pseudo-code/crypto/random.md**: Generates cryptographically secure random keys.  
- **/pseudo-code/exceptions/security_errors.md**: Manages key exchange/derivation errors.  
- **/pseudo-code/logging/logger.md**: Logs key operations and errors for traceability.  
- **/pseudo-code/shared/constants.md**: Defines constants for capability flag bit positions.

## Called By
- **/pseudo-code/security/auth.md**: Establishes ephemeral sessions during authentication.  
- **/pseudo-code/networking/packet_encryption.md**: Encrypts packets with session keys.  
- **/pseudo-code/security/key_management.md**: Manages temporary key lifecycles.  

## Used In
- **Use Case 5.15: Aid Relays**: Ensures old messages remain secure even if node keys are compromised.  
- **Use Case 5.16: Emergency Chat**: Provides per-session encryption for real-time communication.  

## Pseudocode
```pseudocode
// Assuming capability_flags is available in the current context or passed as needed

// Function to validate Curve25519 key
FUNCTION validate_curve25519_key(key, is_private)
    IF key IS NULL OR key IS EMPTY THEN
        RAISE KeyError("Key is null or empty")
    END IF
    IF LENGTH(key) != 32 THEN
        RAISE KeyError("Invalid Curve25519 key length: expected 32 bytes")
    END IF
    IF is_private AND NOT IS_VALID_X25519_PRIVATE_KEY(key) THEN
        RAISE KeyError("Invalid Curve25519 private key format")
    END IF
    IF NOT is_private AND NOT IS_VALID_X25519_PUBLIC_KEY(key) THEN
        RAISE KeyError("Invalid Curve25519 public key format")
    END IF
    RETURN TRUE
END FUNCTION

// Function: generate_ephemeral_keypair
FUNCTION generate_ephemeral_keypair()
    TRY
        private_key = X25519_GENERATE_PRIVATE_KEY()
        CALL validate_curve25519_key(private_key, TRUE)
        public_key = X25519_COMPUTE_PUBLIC_KEY(private_key)
        CALL validate_curve25519_key(public_key, FALSE)
        RETURN { "private": private_key, "public": public_key }
    CATCH key_error
        // Assuming capability_flags is available in the context
        CALL log_message("ERROR", "Failed to generate keypair: " + key_error, capability_flags | LOG_TIER_1_FLAG)  // Log as Tier 1
        RAISE KeyError("Keypair generation failed: " + key_error)
    END TRY
END FUNCTION

// Function: derive_shared_secret
FUNCTION derive_shared_secret(own_private_key, peer_public_key)
    CALL validate_curve25519_key(own_private_key, TRUE)
    CALL validate_curve25519_key(peer_public_key, FALSE)
    TRY
        shared_secret = X25519_ECDH(own_private_key, peer_public_key)
        IF shared_secret IS NULL OR shared_secret IS EMPTY THEN
            RAISE KeyError("Failed to compute shared secret")
        END IF
        RETURN shared_secret
    CATCH ecdh_error
        // Assuming capability_flags is available in the context
        CALL log_message("ERROR", "ECDH failed: " + ecdh_error, capability_flags | LOG_TIER_1_FLAG)  // Log as Tier 1
        RAISE KeyError("Shared secret derivation failed: " + ecdh_error)
    END TRY
END FUNCTION

// Function: derive_session_key
FUNCTION derive_session_key(shared_secret, context_info, packet)
    IF shared_secret IS NULL OR shared_secret IS EMPTY THEN
        RAISE KeyError("Shared secret is null or empty")
    END IF
    // Select protocol based on capability flags (64-bit integer)
    IF packet.capability_flags & ML_PROTOCOL_SELECTION_BIT THEN
        protocol = CALL select_protocol_ml(context_info.network_metrics)
    ELSE
        protocol = CALL select_protocol_static(context_info.network_metrics)
    END IF
    TRY
        // Assuming session_id is part of context_info
        session_key = HKDF(shared_secret, context_info || "meshguardian_session_" || context_info.session_id || protocol)
        IF session_key IS NULL OR session_key IS EMPTY THEN
            RAISE KeyError("Failed to derive session key")
        END IF
        IF LENGTH(session_key) != 32 THEN
            RAISE KeyError("Invalid session key length: expected 32 bytes")
        END IF
        STORE_SESSION_KEY(context_info.session_id, session_key)
        CALL log_message("INFO", "Session key derived for session: " + context_info.session_id + " with protocol: " + protocol, packet.capability_flags)
        RETURN session_key
    CATCH hkdf_error
        CALL log_message("ERROR", "Session key derivation failed: " + hkdf_error, packet.capability_flags | LOG_TIER_1_FLAG)  // Log as Tier 1
        RAISE KeyError("Session key derivation failed: " + hkdf_error)
    END TRY
END FUNCTION

// Function: expire_session_key
FUNCTION expire_session_key(session_id)
    session_key = RETRIEVE_SESSION_KEY(session_id)
    IF session_key IS NULL THEN
        CALL log_message("WARNING", "No session key found for ID: " + session_id, capability_flags)
        RETURN
    END IF
    SECURE_ERASE(session_key)
    DELETE_SESSION_KEY(session_id)
    CALL log_message("INFO", "Session key securely erased for ID: " + session_id, capability_flags)
END FUNCTION

// Function: check_rekeying_conditions
FUNCTION check_rekeying_conditions(session, packet)
    IF session.duration > MAX_SESSION_TIME OR session.bytes_encrypted > REKEY_BYTES_LIMIT THEN
        CALL initiate_rekeying(session)
    END IF
    IF packet.capability_flags & ML_FAILURE_PREDICTION_BIT THEN
        failure_predicted = CALL predict_failure_ml(session.node.battery, session.node.uptime, session.node.rssi_trend)
        IF failure_predicted THEN
            CALL initiate_rekeying(session)
            CALL log_message("INFO", "Rekeying triggered due to ML-predicted failure for session: " + session.id, packet.capability_flags)
        END IF
    END IF
END FUNCTION

// Function: rotate_group_key
FUNCTION rotate_group_key(group_id)
    TRY
        new_group_key = GENERATE_NEW_GROUP_KEY()
        IF new_group_key IS NULL OR LENGTH(new_group_key) != 32 THEN
            RAISE KeyError("Invalid group key generated")
        END IF
        UPDATE_TREEKEM(group_id, new_group_key)
        BROADCAST_KEY_UPDATE(group_id)
        CALL log_message("INFO", "Group key rotated for group: " + group_id, capability_flags)
    CATCH key_error
        CALL log_message("ERROR", "Group key rotation failed: " + key_error, capability_flags | LOG_TIER_1_FLAG)  // Log as Tier 1
        RAISE KeyError("Group key rotation failed: " + key_error)
    END TRY
END FUNCTION

// Function: handle_offline_node
FUNCTION handle_offline_node(node_id, message)
    QUEUE_MESSAGE(node_id, message)
    CALL log_message("INFO", "Message queued for offline node: " + node_id, capability_flags)
END FUNCTION
```

---

## Notes:
- Ephemeral Key Generation: Uses Curve25519 for high-security, efficient keypairs, with validation to ensure key integrity.
- Key Derivation: HKDF with unique context strings prevents key reuse, with error handling for derivation failures.
- Session Key Rotation: Rekeying triggered by time (MAX_SESSION_TIME) or data (REKEY_BYTES_LIMIT), with ML-driven triggers (e.g., ML_FAILURE_PREDICTION_BIT).
- Secure Deletion: Keys are zeroed out using SECURE_ERASE, with checks for key existence to prevent errors.
- **Mesh-Specific Features**:
    -- Group key rotation via TreeKEM ensures forward secrecy in group chats.
    -- Message queuing supports asynchronous communication for offline nodes.
- Error Handling: Validates keys for format and integrity, raising KeyError for invalid keys, with logging via logger.md for traceability.
- Forward Secrecy: Past traffic remains secure even if long-term keys are compromised.
- 64-bit Capability Flags: The capability_flags field is now a 64-bit integer. Bit positions are defined in /pseudo-code/shared/constants.md for flexibility and maintainability. Refer to protocol-specs/capability_flags.md for the full specification.  

## TODO:
- Add session timeout enforcement.
- Support pre-authenticated key exchange for constrained devices.
