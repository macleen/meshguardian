# Authentication Module

## Purpose
The Authentication module verifies the identity of nodes in the MeshGuardian network, enforcing a zero-trust security model. It authenticates nodes using biometric hashes and signatures, supporting post-quantum cryptography and integrating with the audit trail to log authentication events. The module leverages a 64-bit capability flags system for features like ML-driven failure prediction and blockchain audit logging to ensure secure and transparent authentication.

## Depends On
- /pseudo-code/audit_trail.md: Logs authentication attempts and outcomes.
- /pseudo-code/shared/helpers.md: Provides utility functions for biometric hash generation and signature verification.
- /pseudo-code/constants.md: Uses ENCRYPTION_ALGORITHM for cryptographic operations.

## Called By
- /pseudo-code/networking/data_transmission.md: Verifies node identities before transmission.
- /pseudo-code/networking/packet_sending.md: Authenticates sender nodes.

## Used In
- **Use Case 5.15: Aid Relays**: Authenticates nodes for secure supply tracking.
- **Use Case 5.16: Emergency Chat**: Verifies nodes for secure message exchanges.

## Pseudocode
```pseudocode
// Function to authenticate a node
// capability_flags: 64-bit integer representing the node's capabilities
FUNCTION authenticate_node(node_id, biometric_hash, signature, capability_flags)
    // Verify biometric hash
    expected_hash = CALL generate_biometric_hash(node_id, CALL get_nonce())
    IF biometric_hash != expected_hash THEN
        CALL log_event("AuthenticationFailed", {
            "node_id": node_id,
            "error": "Invalid biometric hash"
        }, capability_flags)
        RAISE AuthenticationError("Invalid biometric hash")
    END IF
    // Verify signature
    IF NOT CALL verify_signature(biometric_hash, signature, CALL get_public_key(node_id)) THEN
        CALL log_event("AuthenticationFailed", {
            "node_id": node_id,
            "error": "Invalid signature"
        }, capability_flags)
        RAISE AuthenticationError("Invalid signature")
    END IF
    // Log successful authentication
    CALL log_event("AuthenticationSuccess", {
        "node_id": node_id,
        "capability_flags": capability_flags
    }, capability_flags)
    RETURN TRUE
END FUNCTION
```

---

## Notes
- Zero-Trust: Every node is authenticated before communication, using biometric hashes and signatures.
- Audit Trail: Authentication events are logged, using blockchain (Tier 1) for critical events if the corresponding bit (e.g., constants.BLOCKCHAIN_AUDIT_BIT) is set, or local P2P sync (Tier 2).
- Cryptography: Supports post-quantum signatures (e.g., Dilithium) via capability flags, using ENCRYPTION_ALGORITHM as the default.
- ML Integration: Supports ML-driven failure prediction (e.g., constants.ML_FAILURE_PREDICTION_BIT) to exclude unreliable nodes during authentication.
- 64-bit Transition: The capability_flags field is now a 64-bit integer. Bit positions for features are defined in /pseudo-code/shared/constants.md. Refer to protocol-specs/capability_flags.md for the full specification

## TODO
- Implement multi-factor authentication support.
- Add support for blockchain-based identity registries (e.g., Solana).
