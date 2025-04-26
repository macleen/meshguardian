# Audit Trail Module

## Purpose
The Audit Trail module logs significant events and actions within the MeshGuardian system to ensure transparency, accountability, and traceability for auditing, compliance, or debugging. It supports critical systems like crisis response tools and secure communication platforms. The module implements a simplified two-tier logging system: Tier 1 logs critical events to the Solana blockchain with Zero-Knowledge Proofs (ZKPs) when Capability Flags Bit 15 (Blockchain Audit) is enabled, and Tier 2 logs standard events locally with peer-to-peer (P2P) synchronization, batched every 10 seconds using 64-byte Merkle proofs. This design supports ML-driven events (Bits 13 and 14), aligns with DTNWG’s simplicity preference, and avoids IOTA due to security risks and overhead.

## Interfaces
- **log_event(event_type, details, capability_flags)**: Records an event with a specified type, additional details, and capability flags to determine logging tier (Solana or local).
- **get_logs(filter_criteria)**: Retrieves logs based on criteria such as time range or event type.  
- **archive_logs()**: Moves older logs to an archive for long-term storage.  

## Depends On
- **/pseudo-code/storage/log_storage.md**: Handles the storage and retrieval of log data.  
- **/pseudo-code/exceptions/log_errors.md**: Defines custom exceptions for logging-related 
errors.  
- **/pseudo-code/exceptions/helpers.md**:  Provides utility functions for timestamp generation and JSON serialization.

## Called By
- **/pseudo-code/security/auth.md**: Logs authentication events.  
- **/pseudo-code/networking/data_transmission.md**: Records data transmission events.  

## Used In
- **Use Case 5.15: Aid Relays**: Logs supply tracking actions for audit trails in crisis zones.  
- **Use Case 5.16: Emergency Chat**: Tracks message exchanges for compliance and debugging.  

## Pseudocode
```pseudo-code
// Function to log an event
FUNCTION log_event(event_type, details, capability_flags)
    // Create a log entry with timestamp, event type, and details
    log_entry = {
        "timestamp": CALL get_current_timestamp(),
        "event_type": event_type,
        "details": details,
        "source_node": CALL get_node_id(),
        "tier": 2  // Default to Tier 2 (local)
    }
    // Handle ML-related events
    IF event_type = "ProtocolSelection" THEN
        log_entry.ml_enabled = details.ml_enabled  // True if Bit 13 = 1
        log_entry.protocol = details.protocol      // e.g., "Bluetooth", "LoRa"
    END IF
    IF event_type = "FailurePrediction" THEN
        log_entry.ml_enabled = details.ml_enabled  // True if Bit 14 = 1
        log_entry.prediction = details.prediction  // True/False for failure
    END IF
    // Determine logging tier based on event type and Bit 15
    IF (event_type IN ["PacketTransmission", "PacketReceipt", "ValidationOutcome", "FaultHandling", "AuthenticationSuccess"]
        AND details.priority = "high"
        AND capability_flags HAS BIT 15) THEN
        log_entry.tier = 1  // Tier 1 (Solana)
    END IF
    // Compute audit trail hash
    log_entry.hash = CALL sha3_256(CALL serialize_json(log_entry))
    // Handle logging based on tier
    IF log_entry.tier = 1 THEN
        // Generate ZKP for Tier 1
        zkp_proof = CALL generate_zk_snark(log_entry.hash)
        TRY
            CALL submit_to_solana(log_entry.hash, zkp_proof, CALL get_node_id())
        CATCH connection_error
            // Buffer for interplanetary scenarios
            CALL buffer_log(log_entry, zkp_proof)
        END TRY
    ELSE
        // Store locally for Tier 2
        CALL store_local_log(log_entry)
        // Batch and sync every 10 seconds
        IF CALL time_since_last_sync() >= 10 THEN
            logs_to_sync = CALL get_local_logs()
            merkle_tree = CALL create_merkle_tree([log.hash FOR log IN logs_to_sync])
            merkle_root = merkle_tree.root
            merkle_proofs = merkle_tree.proofs  // 64-byte proofs
            FOR peer IN CALL get_peers()
                CALL send_to_peer(peer, merkle_root, merkle_proofs, logs_to_sync)
            END FOR
            CALL clear_local_logs()
        END IF
    END IF
END FUNCTION

// Function to retrieve logs based on filter criteria
FUNCTION get_logs(filter_criteria)
    // Retrieve logs that match the filter criteria
    logs = CALL retrieve_logs(filter_criteria)
    RETURN logs
END FUNCTION

// Function to archive older logs
FUNCTION archive_logs()
    // Define criteria for logs to archive (e.g., older than 30 days)
    archive_criteria = CALL define_archive_criteria()
    // Retrieve logs to archive
    logs_to_archive = CALL retrieve_logs(archive_criteria)
    // Move logs to archive storage
    CALL archive_logs(logs_to_archive)
    // Remove archived logs from active storage
    CALL remove_archived_logs(logs_to_archive)
END FUNCTION
```

---

## Notes
- **Two-Tier System**: Tier 1 uses Solana with ZKPs for critical events (Bit 15 = 1); Tier 2 uses local logging with 10-second batching and Merkle proofs for standard events, including ML-driven protocol selection (Bit 13) and failure prediction (Bit 14). 
- **Retention Policies**: Define how long logs are kept before archiving or deletion.  
- **Performance**: Batching Tier 2 logs every 10 seconds minimizes overhead; Tier 1 Solana submissions add 1–2s latency. 
- **Security**: SHA3-256 hashes, ZKPs, and Solana’s immutability protect logs from tampering. Access is restricted via Biometric Hash and Signature.
- **Interplanetary Support**: Tier 1 submissions are buffered locally until connectivity is available; Tier 2 logs sync with local peers.
- **TODO**: Implement log rotation to manage storage efficiently.  
