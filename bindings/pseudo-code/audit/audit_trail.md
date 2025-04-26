# Audit Trail Module

## Purpose
The Audit Trail module is responsible for logging significant events and actions within the system to maintain a record for auditing, compliance, or debugging. It ensures transparency, accountability, and traceability in critical systems like crisis response tools or secure communication platforms. The module implements a simplified two-tier logging system: Tier 1 logs critical events to the Solana blockchain with Zero-Knowledge Proofs (ZKPs) when Capability Flags Bit 15 (Blockchain Audit) is enabled, and Tier 2 logs standard events locally with peer-to-peer (P2P) synchronization, batched every 10 seconds using 64-byte Merkle proofs. This design supports ML-driven events (Bits 13 and 14) and aligns with DTNWG’s simplicity preference, avoiding IOTA due to security risks and overhead.

## Interfaces
- **log_event(event_type, details, capability_flags)**: Records an event with a specified type, additional details, and capability flags to determine logging tier (Solana or local).
- **get_logs(filter_criteria)**: Retrieves logs based on criteria such as time range or event type.  
- **archive_logs()**: Moves older logs to an archive for long-term storage.  

## Depends On
- **/pseudo-code/storage/log_storage.md**: Handles the storage and retrieval of log data.  
- **/pseudo-code/exceptions/log_errors.md**: Defines custom exceptions for logging-related errors.  

## Called By
- **/pseudo-code/security/auth.md**: Logs authentication events.  
- **/pseudo-code/networking/data_transmission.md**: Records data transmission events.  

## Used In
- **Use Case 5.15: Aid Relays**: Logs supply tracking actions for audit trails in crisis zones.  
- **Use Case 5.16: Emergency Chat**: Tracks message exchanges for compliance and debugging.  

## Pseudocode
```pseudo-code
/// Function to log an event
FUNCTION log_event(event_type, details, capability_flags)
    // Create a log entry with timestamp, event type, and details
    log_entry = {
        "timestamp": GET_CURRENT_TIMESTAMP(),
        "event_type": event_type,
        "details": details,
        "source_node": GET_NODE_ID(),
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
    IF (event_type IN ["PacketTransmission", "PacketReceipt", "ValidationOutcome", "FaultHandling"]
        AND details.priority = "high"
        AND capability_flags HAS BIT 15) THEN
        log_entry.tier = 1  // Tier 1 (Solana)
    END IF
    // Compute audit trail hash
    log_entry.hash = SHA3_256(SERIALIZE_JSON(log_entry))
    // Handle logging based on tier
    IF log_entry.tier = 1 THEN
        // Generate ZKP for Tier 1
        zkp_proof = GENERATE_ZK_SNARK(log_entry.hash)
        TRY
            SUBMIT_TO_SOLANA(log_entry.hash, zkp_proof, GET_NODE_ID())
        CATCH connection_error
            // Buffer for interplanetary scenarios
            BUFFER_LOG(log_entry, zkp_proof)
        END TRY
    ELSE
        // Store locally for Tier 2
        STORE_LOCAL_LOG(log_entry)
        // Batch and sync every 10 seconds
        IF TIME_SINCE_LAST_SYNC() >= 10 THEN
            logs_to_sync = GET_LOCAL_LOGS()
            merkle_tree = CREATE_MERKLE_TREE([log.hash FOR log IN logs_to_sync])
            merkle_root = merkle_tree.root
            merkle_proofs = merkle_tree.proofs  // 64-byte proofs
            FOR peer IN GET_PEERS()
                SEND_TO_PEER(peer, merkle_root, merkle_proofs, logs_to_sync)
            END FOR
            CLEAR_LOCAL_LOGS()
        END IF
    END IF
END FUNCTION

// Function to retrieve logs based on filter criteria
FUNCTION get_logs(filter_criteria)
    // Retrieve logs that match the filter criteria
    logs = RETRIEVE_LOGS(filter_criteria)
    RETURN logs
END FUNCTION

// Function to archive older logs
FUNCTION archive_logs()
    // Define criteria for logs to archive (e.g., older than 30 days)
    archive_criteria = DEFINE_ARCHIVE_CRITERIA()
    // Retrieve logs to archive
    logs_to_archive = RETRIEVE_LOGS(archive_criteria)
    // Move logs to archive storage
    ARCHIVE_LOGS(logs_to_archive)
    // Remove archived logs from active storage
    REMOVE_ARCHIVED_LOGS(logs_to_archive)
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
