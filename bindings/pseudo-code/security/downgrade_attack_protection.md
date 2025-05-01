# Authentication Logic with Downgrade Attack Protection

This document outlines the downgrade attack protection logic implemented in MeshGuardian, specifically focusing on protocol version validation, encryption and compression algorithm enforcement, consensus mode verification, and secure negotiation.

---

## Function: `validate_protocol_security(user_id, ip_address, session_state)`

Performs the downgrade attack protection procedure including:
    - Protocol version validation
    - Encryption algorithm and strength enforcement
    - Compression algorithm validation for chunked payloads
    - Consensus mode verification
    - Secure negotiation flag checking

### Logic

```pseudocode
FUNCTION validate_protocol_security(user_id, ip_address, session_state)
    cached_config = CALL cache_security_config()
    
    # Check protocol version
    IF session_state.protocol_version < cached_config.protocol_version THEN
        CALL session_state.downgrade_attempts[user_id] += 1
        CALL LOG_SECURITY_EVENT("Downgrade attempt: protocol version " + session_state.protocol_version, BIT_15 | BIT_16)
        CALL broadcast_downgrade_alert(user_id, "Invalid protocol version")
        IF session_state.downgrade_attempts[user_id] >= DOWNGRADE_ATTEMPT_THRESHOLD THEN
            CALL LOCKOUT(ip_address, 1440)  # 24-hour lockout
            CALL slash_malicious_node(user_id, "Downgrade attack detected")
        END IF
        RETURN False
    END IF

    # Check encryption algorithm
    IF session_state.encryption_algorithm NOT IN cached_config.encryption_algorithms OR session_state.encryption_strength < cached_config.min_encryption_strength THEN
        CALL session_state.downgrade_attempts[user_id] += 1
        CALL LOG_SECURITY_EVENT("Downgrade attempt: weak encryption " + session_state.encryption_algorithm, BIT_15 | BIT_16)
        CALL broadcast_downgrade_alert(user_id, "Weak encryption")
        IF session_state.downgrade_attempts[user_id] >= DOWNGRADE_ATTEMPT_THRESHOLD THEN
            CALL LOCKOUT(ip_address, 1440)
            CALL slash_malicious_node(user_id, "Downgrade attack detected")
        END IF
        RETURN False
    END IF

    # Check compression algorithm (if applicable)
    IF session_state.capability_flags & BIT_4 THEN  # Chunked fragment
        IF session_state.compression_algorithm NOT IN cached_config.compression_algorithms THEN
            CALL session_state.downgrade_attempts[user_id] += 1
            CALL LOG_SECURITY_EVENT("Downgrade attempt: weak compression " + session_state.compression_algorithm, BIT_15 | BIT_16)
            CALL broadcast_downgrade_alert(user_id, "Weak compression")
            RETURN False
        END IF
    END IF

    # Check consensus mode
    IF session_state.consensus_mode NOT IN cached_config.allowed_consensus_modes THEN
        CALL session_state.downgrade_attempts[user_id] += 1
        CALL LOG_SECURITY_EVENT("Downgrade attempt: invalid consensus mode " + session_state.consensus_mode, BIT_15 | BIT_16)
        CALL broadcast_downgrade_alert(user_id, "Invalid consensus mode")
        RETURN False
    END IF

    # Ensure secure negotiation flag
    IF NOT (session_state.capability_flags & BIT_5) THEN
        CALL LOG_SECURITY_EVENT("Missing secure negotiation flag", BIT_15)
        RETURN False
    END IF

    # Reset downgrade attempts on success
    session_state.downgrade_attempts[user_id] = 0
    RETURN True
END FUNCTION

FUNCTION broadcast_downgrade_alert(user_id, reason)
    alert_message = {
        "type": "DowngradeAlert",
        "sender": user_id,
        "reason": reason,
        "capability_flags": BIT_1 | BIT_16,  # Broadcast with downgrade alert flag
        "timestamp": get_current_timestamp(),
        "nonce": generate_unique_id()
    }
    CALL route_message(alert_message, network_nodes)
END FUNCTION
```

## Helper Functions

- `cache_security_config()`: Caches security configurations (encryption algorithms, compression algorithms, protocol version, consensus modes) for low-power optimization

- `LOG_SECURITY_EVENT(message, flags)`: Logs security events like downgrade attempts for further analysis, using capability flags (e.g., BIT_15, BIT_16).

- `LOCKOUT(ip_address, duration)`:  Locks out the IP address for a specified duration.
- `slash_malicious_node(user_id, reason)`: Slashes a malicious node, decaying its trust score and logging the event.

- `broadcast_downgrade_alert(user_id, reason)`: Broadcasts a downgrade attempt alert to the network with BIT_16.

- `get_current_timestamp()`:  Retrieves the current system timestamp.

- `generate_unique_id()`: Generates a unique nonce for messages

- `route_message(message, nodes)`: Routes messages to specified nodes (e.g., for downgrade alerts).

- `get_trust_score(user_id)`: Retrieves the trust score for a node (placeholder for trust graph).

- `update_trust_score(user_id, score)`: Updates the trust score for a node (placeholder for trust graph).

---

## Comparison to Standards

- **TLS 1.3**: Uses strict protocol version validation to prevent downgrade attacks. MeshGuardian aligns by enforcing PROTOCOL_VERSION and rejecting outdated versions, with BIT_16 alerts for network awareness.
- **WireGuard**: Employs secure algorithm negotiation and session-based counters. MeshGuardian’s capability flags (BIT_5, BIT_16) and encryption strength checks (MIN_ENCRYPTION_STRENGTH) provide similar protections.
- **Signal Protocol**: Ensures secure key negotiation and resists protocol weakening. MeshGuardian’s enforcement of encryption algorithms (e.g., AES-256, Kyber) and compression whitelisting (e.g., zstd, lz4) mirrors this resilience.

---

## Notes

- Downgrade Attack Protection: Validates protocol version, encryption algorithms (e.g., AES-256, Kyber), compression algorithms (e.g., zstd, lz4), consensus modes (PoS, PBFT, ADTC), and secure negotiation flag (BIT_5) to prevent fallback to insecure configurations. Broadcasts alerts with BIT_16.
- IP Lockout Mechanism: Locks out IPs after 3 downgrade attempts (DOWNGRADE_ATTEMPT_THRESHOLD), with a 24-hour duration, mitigating persistent attacks.
- Secure Credential Storage: Enforces minimum encryption strength (MIN_ENCRYPTION_STRENGTH = 256 bits) during validation.
- Optimization: Caches security configurations (cache_security_config) for low-power devices (e.g., Raspberry Pi Zero 2W, LoRa HATs, hardware_requirements.md), reducing lookup overhead.
- Trust Graph Integration: Slashes malicious nodes with trust score decay (TRUST_DECAY_FACTOR = 0.1), preparing for adaptive trust models.
- Glossary Terms:
    -- Downgrade Attack: A threat where a node is tricked into using a less secure protocol, cipher, or consensus method, exposing vulnerabilities.
    -- BIT_5: Capability flag (0x10) indicating secure negotiation mode is enabled. Must be present in all protocol-compliant messages.
    -- BIT_15: Capability flag (0x8000) indicating high-severity security events, used for Tier 1 audit logging (e.g., downgrade attempts).
    -- BIT_16: Capability flag (0x10000) indicating a downgrade attempt broadcast to alert the network of potential attacks.
- Future Improvements: Add geo-based exceptions for trusted locations, integrate quantum-resistant signatures, and implement full trust graph scoring (get_trust_score, update_trust_score).