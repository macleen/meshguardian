# Audit Trail Module

## Purpose
The Audit Trail module ensures transparency and traceability in MeshGuardian by logging critical system events, such as consensus decisions, data transmissions, and errors, to a blockchain or local storage. It supports both Tier 1 (blockchain-based, e.g., Solana, Avalanche, Ethereum) and Tier 2 (local with P2P sync) logging, with dynamic timeout adjustments and robust buffering for interplanetary deployments. To mitigate security risks from reliance on a single blockchain (e.g., Solana), the module implements multi-blockchain support and a custom lightweight ledger for critical events, reducing dependency on external systems. It also integrates with slashing logic for malicious nodes in PBFT and PoS consensus, ensuring accountability in crisis zones and interplanetary missions.

## Interfaces
- **log_event(level, message, flags)**: Logs an event with the specified level, message, and capability flags.
- **submit_to_blockchain(event)**: Submits an event to a blockchain for Tier 1 logging with multi-blockchain support.
- **submit_to_lightweight_ledger(event)**: Logs an event to a custom lightweight ledger for critical events.
- **persist_log_locally(event, expiration)**: Stores an event locally with an expiration time for Tier 2 logging or buffering.
- **sync_p2p_logs(peers)**: Synchronizes locally stored logs with peers for Tier 2 logging.

## Depends On
- **/pseudo-code/security/crypto_utils.md**: Provides cryptographic functions for hashing and signing logs.
- **/pseudo-code/logging/logger.md**: Handles low-level logging operations.
- **/pseudo-code/shared/helpers.md**: Provides utility functions for timestamp formatting and RTT estimation.
- **/pseudo-code/shared/constants.md**: Defines logging levels, blockchain priorities, timeouts, and ledger settings.

## Called By
- **/pseudo-code/consensus/pbft_main.md**: Logs PBFT consensus events.
- **/pseudo-code/consensus/pos_main.md**: Logs PoS consensus events.
- **/pseudo-code/networking/data_transmission.md**: Logs data transmission events.
- **/pseudo-code/security/auth.md**: Logs authentication events.

## Used In
- **Use Case 5.15: Aid Relays**: Logs supply tracking events to ensure transparency in crisis zones.
- **Use Case 5.16: Emergency Chat**: Logs message delivery events for auditability.
- **Use Case 5.1.1: Mars Rover Data Relay**: Logs interplanetary data transfers and ADTC consensus outcomes.

## Pseudocode
```pseudo-code
// Structure for an audit trail event
STRUCT AuditTrailEvent
    level: STRING           // Log level (e.g., "INFO", "ERROR")
    message: STRING         // Event message
    timestamp: INTEGER      // Event timestamp
    capability_flags: BITFIELD // Capability Flags for context
    event_hash: BYTE_ARRAY  // Cryptographic hash of the event
    blockchain: STRING      // Target blockchain (e.g., "Solana", "Avalanche")
END STRUCT

// Function to log an event
FUNCTION log_event(level, message, capability_flags)
    TRY
        // Create audit trail event
        event = NEW AuditTrailEvent
        event.level = level
        event.message = message
        event.timestamp = CALL helpers.get_current_time_ms()
        event.capability_flags = capability_flags
        event.event_hash = CALL crypto_utils.sha3_256(level + message + event.timestamp)
        event.blockchain = "None" // Default, updated if submitted

        // Determine logging tier
        IF level IN ["ERROR", "Consensus", "ADTC_Validation"] OR capability_flags & BIT_15 THEN
            // Tier 1: Blockchain logging with multi-blockchain support
            TRY
                CALL submit_to_blockchain(event)
            CATCH blockchain_error
                CALL log_message("ERROR", "Blockchain submission failed: " + blockchain_error, capability_flags | BIT_15) // Log as Tier 1
                // Fallback to lightweight ledger
                IF constants.LIGHTWEIGHT_LEDGER_ENABLED THEN
                    CALL submit_to_lightweight_ledger(event)
                ELSE
                    // Buffer locally for later submission
                    CALL persist_log_locally(event, constants.MAX_BUFFER_PERSISTENCE)
                END IF
            END TRY
        ELSE
            // Tier 2: Local logging with P2P sync
            CALL persist_log_locally(event, constants.MAX_BUFFER_PERSISTENCE)
            CALL sync_p2p_logs(GET_PEERS())
        END IF
    CATCH error
        CALL log_message("ERROR", "Failed to log event: " + error, capability_flags | BIT_15) // Log as Tier 1
        RAISE AuditTrailError("Event logging failed: " + error)
    END TRY
END FUNCTION

// Function to submit an event to a blockchain
FUNCTION submit_to_blockchain(event)
    TRY
        // Iterate through blockchain priority list
        FOR each blockchain IN constants.BLOCKCHAIN_PRIORITY_LIST
            TRY
                event.blockchain = blockchain
                // Sign event with private key
                signature = CALL crypto_utils.sign_event(event.event_hash, PRIVATE_KEY)
                // Submit to blockchain with dynamic timeout
                timeout = constants.BLOCKCHAIN_SUBMISSION_TIMEOUT
                IF capability_flags & BIT_28 THEN
                    // Adjust timeout for interplanetary scenarios
                    current_rtt = CALL helpers.estimate_rtt()
                    IF current_rtt > constants.RTT_THRESHOLD THEN
                        timeout = timeout * (current_rtt / constants.RTT_THRESHOLD)
                        timeout = MIN(timeout, constants.BLOCKCHAIN_SUBMISSION_TIMEOUT * 2)
                    END IF
                END IF
                tx_id = CALL blockchain.submit(event, signature, timeout)
                CALL log_message("INFO", "Event submitted to " + blockchain + ": tx_id=" + tx_id, capability_flags | BIT_15) // Log as Tier 1
                RETURN tx_id
            CATCH blockchain_error
                CALL log_message("WARNING", "Failed to submit to " + blockchain + ": " + blockchain_error, capability_flags | BIT_15) // Log as Tier 1
                // Continue to next blockchain
            END TRY
        END FOR
        RAISE BlockchainError("Failed to submit to any blockchain")
    CATCH error
        CALL log_message("ERROR", "Blockchain submission failed: " + error, capability_flags | BIT_15) // Log as Tier 1
        RAISE BlockchainError("Blockchain submission failed: " + error)
    END TRY
END FUNCTION

// Function to submit an event to a lightweight ledger
FUNCTION submit_to_lightweight_ledger(event)
    TRY
        // Check ledger size limit
        IF GET_LEDGER_SIZE() >= constants.LIGHTWEIGHT_LEDGER_MAX_SIZE THEN
            RAISE LedgerError("Lightweight ledger full")
        END IF
        // Store event in lightweight ledger
        ledger_entry = {
            "event_hash": event.event_hash,
            "level": event.level,
            "message": event.message,
            "timestamp": event.timestamp,
            "capability_flags": event.capability_flags
        }
        CALL store_ledger_entry(ledger_entry)
        CALL log_message("INFO", "Event submitted to lightweight ledger", capability_flags | BIT_15) // Log as Tier 1
    CATCH ledger_error
        CALL log_message("ERROR", "Lightweight ledger submission failed: " + ledger_error, capability_flags | BIT_15) // Log as Tier 1
        RAISE LedgerError("Lightweight ledger submission failed: " + ledger_error)
    END TRY
END FUNCTION

// Function to persist a log event locally
FUNCTION persist_log_locally(event, expiration)
    TRY
        // Store event in local storage with expiration
        CALL store_local_log(event, expiration)
        CALL log_message("INFO", "Event persisted locally: " + event.event_hash, capability_flags)
    CATCH storage_error
        CALL log_message("ERROR", "Failed to persist log locally: " + storage_error, capability_flags | BIT_15) // Log as Tier 1
        RAISE StorageError("Local log persistence failed: " + storage_error)
    END TRY
END FUNCTION

// Function to synchronize locally stored logs with peers
FUNCTION sync_p2p_logs(peers)
    TRY
        FOR each peer IN peers
            TRY
                local_logs = GET_LOCAL_LOGS()
                peer_logs = CALL peer.request_logs()
                // Merge logs, prioritizing newer timestamps
                merged_logs = MERGE_LOGS(local_logs, peer_logs)
                CALL store_local_log(merged_logs, constants.MAX_BUFFER_PERSISTENCE)
                CALL peer.send_logs(merged_logs)
                CALL log_message("INFO", "Logs synchronized with peer: " + peer.id, capability_flags)
            CATCH peer_error
                CALL log_message("WARNING", "Failed to sync logs with peer " + peer.id + ": " + peer_error, capability_flags)
                // Continue to next peer
            END TRY
        END FOR
    CATCH sync_error
        CALL log_message("ERROR", "P2P log sync failed: " + sync_error, capability_flags | BIT_15) // Log as Tier 1
        RAISE SyncError("P2P log sync failed: " + sync_error)
    END TRY
END FUNCTION
```

---

## Notes

- Transparency: Tier 1 logging uses multi-blockchain support (Solana, Avalanche, Ethereum) for critical events (e.g., errors, consensus, ADTC validation), with a custom lightweight ledger as a fallback to reduce external dependencies. Tier 2 logging uses local storage with P2P sync for non-critical events, ensuring scalability.
- Reliability: Dynamic timeout adjustments (based on RTT estimates) and robust buffering (up to 30 days) ensure reliable logging in high-latency interplanetary scenarios. Multi-blockchain retries and lightweight ledger fallback enhance resilience against blockchain failures.
- Security: Events are cryptographically hashed and signed to prevent tampering. Multi-blockchain support mitigates risks from single-blockchain dependency, while the lightweight ledger provides a secure alternative for critical events. Slashing logic for malicious nodes (in PBFT and PoS) is logged as Tier 1 events for accountability.
- Performance: Blockchain submissions are optimized with dynamic timeouts (capped at 2x default). Local logging and P2P sync are lightweight (<10ms for 1KB logs), with lightweight ledger storage limited to LIGHTWEIGHT_LEDGER_MAX_SIZE to prevent resource exhaustion.
- Scalability: Supports multiple blockchains via BLOCKCHAIN_PRIORITY_LIST, with configurable preferences for latency, cost, or redundancy. Lightweight ledger supports resource-constrained devices with minimal storage overhead.
- Edge Cases: Handles blockchain failures, network disruptions, and storage limits by buffering logs locally, retrying across blockchains, or falling back to the lightweight ledger. Malicious node slashing events are logged with high priority.

## TODO
- Implement compression for locally stored logs to optimize storage.
- Add support for log rotation to manage MAX_BUFFER_SIZE limits.
- Integrate with a distributed hash table (DHT) for more efficient P2P log sync.
- Develop a mechanism for log pruning based on event priority and age.
- Implement audit trail checks for slashed nodes, linking logs to slashing events for traceability.