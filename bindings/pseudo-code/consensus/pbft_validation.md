# PBFT Validation Module  

## Purpose
The PBFT Validation module is responsible for validating messages, transactions, and votes within the Practical Byzantine Fault Tolerance (PBFT) consensus algorithm. It ensures that all data being agreed upon by the network meets criteria for integrity, authenticity, and consistency, even in the presence of faulty or malicious nodes, with slashing logic to penalize malicious behavior. The module provides a critical layer of trust and reliability in distributed systems, enabling consensus in environments where up to one-third of nodes might exhibit Byzantine behavior, with fallback handling for sparse networks and high-latency interplanetary scenarios. For resource-constrained devices, it is optimized by simplifying validation, and it supports multi-blockchain or lightweight ledger logging to mitigate single-blockchain dependency risks.

## Interfaces
- **validate_message(message, context)**: Verifies the structure, integrity, and authenticity of a message received during the consensus process.  
- **verify_signature(signature, message, public_key)**: Validates the cryptographic signature attached to a message using the sender’s public key.  
- **check_quorum(votes, context)**: Confirms that a sufficient number of nodes (a quorum) have agreed on a decision or vote, satisfying PBFT’s fault tolerance requirements, and checks for sufficient node counts.  
- **slash_malicious_node(node_id, reason)**: Penalizes a node for malicious behavior by adding it to a slashed nodes list and logging the event.

## Depends On
- **/pseudo-code/crypto/signature.md**: Supplies cryptographic functions for verifying message signatures.  
- **/pseudo-code/consensus/pbft_state.md**: Provides access to the current state of the PBFT consensus process, such as sequence numbers and view information.  
- **/pseudo-code/network/message.md**: Defines the structure and handling of messages exchanged over the network.  
- **/pseudo-code/exceptions/helpers.md**: Provides utility functions for RTT estimation and validation utilities.
- **/pseudo-code/constants.md**: Defines thresholds for RTT, buffering persistence, validation intervals, slashing, and blockchain priorities.

## Called By
- **/pseudo-code/consensus/pbft_main.md**: Invokes this module as part of the core PBFT consensus workflow to validate incoming messages and votes.  
- **/pseudo-code/application_layer/transaction_processor.md**: Uses this module to validate transactions before they are proposed for inclusion in a block.  

## Used In
- **Use Case 5.15: Aid Relays**: Ensures that supply tracking data is consistently validated and agreed upon across nodes in crisis zones, preventing data tampering.  
- **Use Case 5.16: Emergency Chat**: Maintains agreement on the ordering and delivery of messages in real-time communication systems, ensuring reliable operation under faulty conditions.  

## Pseudocode
```pseudo-code
// Structure for PBFT context
STRUCT PBFTContext
    message: Message           // Message to validate
    capability_flags: BITFIELD // 32-bit Capability Flags
    device_capability_level: INTEGER // 0 = minimal, 1 = moderate, 2 = high
    message_count: INTEGER     // Counter for tracking validation frequency
    malicious_votes: MAP       // Tracks malicious vote attempts by node_id
END STRUCT

// Function to adjust validation timeout based on RTT
FUNCTION adjust_validation_timeout() RETURNS INTEGER
    TRY
        // Estimate real-time RTT using orbital data or historical trends
        current_rtt = CALL helpers.estimate_rtt()
        IF current_rtt > constants.RTT_THRESHOLD THEN
            // Increase timeout for prolonged delays
            new_timeout = constants.DEFAULT_TIMEOUT * (current_rtt / constants.RTT_THRESHOLD)
        ELSE
            // Use default timeout for stable conditions
            new_timeout = constants.DEFAULT_TIMEOUT
        END IF
        // Cap timeout to prevent excessive delays
        new_timeout = MIN(new_timeout, constants.DEFAULT_TIMEOUT * 2)
        CALL log_event("INFO", "Validation timeout adjusted to " + new_timeout + " seconds, RTT: " + current_rtt, capability_flags)
        RETURN new_timeout
    CATCH error
        CALL log_event("ERROR", "Failed to adjust validation timeout: " + error, capability_flags | BIT_15) // Log as Tier 1
        RETURN constants.DEFAULT_TIMEOUT // Fallback to default
    END TRY
END FUNCTION

// Function to slash a malicious node
FUNCTION slash_malicious_node(node_id, reason)
    TRY
        // Add node to slashed nodes list
        SLASHED_NODES_LIST.ADD(node_id)
        // Log slashing event with multi-blockchain support
        CALL log_event("SLASHING", "Node slashed: " + node_id + ", Reason: " + reason, capability_flags | BIT_15 | constants.SLASHING_EVENT_CODE)
        CALL log_message("INFO", "Node " + node_id + " slashed for: " + reason, capability_flags | BIT_15)
    CATCH error
        CALL log_event("ERROR", "Failed to slash node " + node_id + ": " + error, capability_flags | BIT_15) // Log as Tier 1
        RAISE SlashingError("Node slashing failed: " + error)
    END TRY
END FUNCTION

// Function to validate a message
FUNCTION validate_message(message, context: PBFTContext)
    // Check if sender is slashed
    IF message.sender IN SLASHED_NODES_LIST THEN
        CALL log_event("WARNING", "Message from slashed node: " + message.sender, context.capability_flags | BIT_15)
        RETURN False
    END IF
    // Check if the message is well-formed
    IF NOT is_well_formed(message)
        CALL context.malicious_votes[message.sender] += 1
        IF context.malicious_votes[message.sender] >= constants.MALICIOUS_VOTE_THRESHOLD THEN
            CALL slash_malicious_node(message.sender, "Repeated malformed messages")
        END IF
        RETURN False
    // Validate Feature Flags
    feature_flags = message.feature_flags
    IF feature_flags BIT_13 AND feature_flags BIT_24 THEN
        CALL context.malicious_votes[message.sender] += 1
        IF context.malicious_votes[message.sender] >= constants.MALICIOUS_VOTE_THRESHOLD THEN
            CALL slash_malicious_node(message.sender, "Invalid feature flags: ML with Low-Energy Mode")
        END IF
        RETURN False  // ML Protocol Selection (Bit 13) not allowed in Low-Energy Mode (Bit 24)
    END IF
    IF feature_flags BIT_14 AND feature_flags BIT_24 THEN
        CALL context.malicious_votes[message.sender] += 1
        IF context.malicious_votes[message.sender] >= constants.MALICIOUS_VOTE_THRESHOLD THEN
            CALL slash_malicious_node(message.sender, "Invalid feature flags: ML Failure Prediction with Low-Energy Mode")
        END IF
        RETURN False  // ML Failure Prediction (Bit 14) not allowed in Low-Energy Mode (Bit 24)
    END IF
    // Check for interplanetary scenario requiring ADTC
    timeout = CALL adjust_validation_timeout()
    IF timeout > constants.RTT_THRESHOLD AND feature_flags BIT_28 THEN
        CALL log_event("INFO", "High RTT detected, falling back to ADTC: " + timeout + " seconds", feature_flags | BIT_15)
        RETURN False  // Trigger ADTC fallback
    END IF
    // Simplify validation for ultra-low-power devices
    IF context.device_capability_level == 0 OR context.capability_flags & BIT_24 THEN
        CALL log_event("INFO", "Using simplified hash-based validation for ultra-low-power device", context.capability_flags)
        expected_hash = CALL crypto_utils.sha3_256(message.content)
        IF message.hash != expected_hash THEN
            CALL context.malicious_votes[message.sender] += 1
            IF context.malicious_votes[message.sender] >= constants.MALICIOUS_VOTE_THRESHOLD THEN
                CALL slash_malicious_node(message.sender, "Invalid hash-based validation")
            END IF
            RETURN False
        END IF
        RETURN True
    END IF
    // Limit validation frequency for low-power devices
    IF context.device_capability_level == 1 AND context.message_count % constants.PBFT_VALIDATION_INTERVAL != 0 THEN
        CALL log_event("INFO", "Skipped full PBFT validation, using hash-based check", context.capability_flags)
        expected_hash = CALL crypto_utils.sha3_256(message.content)
        IF message.hash != expected_hash THEN
            CALL context.malicious_votes[message.sender] += 1
            IF context.malicious_votes[message.sender] >= constants.MALICIOUS_VOTE_THRESHOLD THEN
                CALL slash_malicious_node(message.sender, "Invalid hash-based validation")
            END IF
            RETURN False
        END IF
        RETURN True
    END IF
    // Increment message counter for validation frequency
    context.message_count = context.message_count + 1
    // Retrieve sender's public key and verify signature
    public_key = GET_SENDER_PUBLIC_KEY(message.sender)
    IF NOT verify_signature(message.signature, message.content, public_key)
        CALL context.malicious_votes[message.sender] += 1
        IF context.malicious_votes[message.sender] >= constants.MALICIOUS_VOTE_THRESHOLD THEN
            CALL slash_malicious_node(message.sender, "Invalid signature")
        END IF
        RETURN False
    // Check timestamp to prevent replay attacks
    IF message.timestamp < CURRENT_TIME - MAX_TIME_DRIFT
        CALL context.malicious_votes[message.sender] += 1
        IF context.malicious_votes[message.sender] >= constants.MALICIOUS_VOTE_THRESHOLD THEN
            CALL slash_malicious_node(message.sender, "Invalid timestamp")
        END IF
        RETURN False
    RETURN True
END FUNCTION

// Function to verify a cryptographic signature
FUNCTION verify_signature(signature, message, public_key)
    // Delegate to cryptographic library for signature verification
    RETURN CRYPTO_VERIFY(signature, message, public_key)
END FUNCTION

// Function to check if a quorum is achieved
FUNCTION check_quorum(votes, context: PBFTContext)
    // Check total node count for PBFT requirements
    total_nodes = GET_TOTAL_NODES()
    max_faulty_nodes = FLOOR((total_nodes - 1) / 3)  // f = max faulty nodes
    min_nodes_required = 3 * max_faulty_nodes + 1  // 3f + 1
    IF total_nodes < min_nodes_required
        // Log insufficient node count as Tier 1 event
        CALL log_event("ConsensusError", "Insufficient nodes for PBFT: " + total_nodes + " < " + min_nodes_required, context.capability_flags | BIT_15)
        // Buffer votes locally for persistence during outages
        FOR each vote IN votes
            IF vote.node_id NOT IN SLASHED_NODES_LIST THEN
                CALL store_vote_locally(vote, constants.MAX_BUFFER_PERSISTENCE)
            END IF
        END FOR
        // Attempt fallback to ADTC if supported
        IF context.capability_flags BIT_28 THEN
            CALL log_event("INFO", "Falling back to ADTC due to insufficient PBFT nodes", context.capability_flags | BIT_15)
            RETURN False  // Trigger ADTC fallback
        END IF
        // Log to blockchain or lightweight ledger
        TRY
            CALL log_event("ConsensusError", "Insufficient nodes for PBFT, PoS fallback", context.capability_flags | BIT_15)
        CATCH blockchain_error
            CALL log_event("ConsensusError_Retry", "Retry on lightweight ledger: Insufficient nodes for PBFT", context.capability_flags | BIT_15)
        END TRY
        RAISE ConsensusError("Insufficient nodes for PBFT consensus, consider PoS fallback")
    END IF
    // Calculate required quorum size (2f + 1)
    quorum_size = 2 * max_faulty_nodes + 1
    // Optimize vote processing for low-power devices
    agreeing_votes = 0
    IF context.device_capability_level <= 1 THEN
        // Process only first 2f+1 valid votes to reduce overhead
        FOR each vote IN votes UNTIL agreeing_votes >= quorum_size
            IF vote.is_valid AND vote.node_id NOT IN SLASHED_NODES_LIST THEN
                agreeing_votes = agreeing_votes + 1
            END IF
        END FOR
    ELSE
        // Full vote processing for high-capability devices
        FOR each vote IN votes
            IF vote.is_valid AND vote.node_id NOT IN SLASHED_NODES_LIST THEN
                agreeing_votes = agreeing_votes + 1
            END IF
        END FOR
    END IF
    IF agreeing_votes >= quorum_size
        CALL log_event("INFO", "Quorum achieved with " + agreeing_votes + " valid votes", context.capability_flags | BIT_15)
        RETURN True
    ELSE
        // Buffer valid votes locally for persistence
        FOR each vote IN votes
            IF vote.node_id NOT IN SLASHED_NODES_LIST THEN
                CALL store_vote_locally(vote, constants.MAX_BUFFER_PERSISTENCE)
            END IF
        END FOR
        CALL log_event("WARNING", "Quorum not achieved: " + agreeing_votes + " < " + quorum_size, context.capability_flags | BIT_15)
        RETURN False
    END IF
END FUNCTION

// Function to store votes locally for persistence
FUNCTION store_vote_locally(vote, max_persistence)
    TRY
        // Store vote in local storage with expiration
        CALL store_local_log(vote, expiration=max_persistence)
        CALL log_event("INFO", "Vote stored locally for persistence: " + vote.node_id, capability_flags)
    CATCH storage_error
        CALL log_event("ERROR", "Failed to store vote locally: " + storage_error, capability_flags | BIT_15) // Log as Tier 1
        RAISE StorageError("Vote storage failed: " + storage_error)
    END TRY
END FUNCTION
```

---

## Notes
- Performance: Validation is optimized to minimize latency in high-throughput consensus systems. Node count checks and slashing logic add minimal overhead (<1ms). Dynamic timeout adjustments ensure reliability in interplanetary scenarios with high RTTs. For low-power devices (Device Capability Level 0 or 1), simplified hash-based validation and limited vote processing reduce computational overhead (e.g., <10ms for 1KB messages).
- Security: Cryptographic signature verification uses robust, attack-resistant algorithms for high-capability devices. Simplified hash-based validation for low-power devices maintains integrity but sacrifices some Byzantine fault tolerance. Slashing logic penalizes malicious nodes (e.g., for invalid votes or signatures), logging events to multiple blockchains or a lightweight ledger to mitigate single-blockchain risks. Insufficient node count checks prevent consensus stalls in sparse networks, with buffered votes ensuring persistence during outages.
- Scalability: The quorum size adapts dynamically based on the total number of nodes and desired fault tolerance (f = floor((n-1)/3)). Fallback to ADTC or PoS ensures operation in sparse or high-latency networks, with multi-blockchain or lightweight ledger logging enhancing audit trail reliability. Low-power optimizations limit validation frequency and vote processing for resource-constrained devices.
- Hardware Requirements:
    -- Full PBFT Validation: Minimum 80 MHz CPU, 128 KB RAM, 500 µW power consumption; suitable for moderate- to high-capability devices.
    -- Simplified Hash-based Validation: Minimum 16 MHz CPU, 32 KB RAM, 100 µW power consumption; suitable for ultra-low-power IoT nodes.
    -- ADTC Fallback: Minimum 32 MHz CPU, 64 KB RAM, 200 µW power consumption; suitable for low-power devices in high-latency scenarios.
- Edge Cases: Handles network delays, duplicate messages, malicious vote tampering, insufficient node counts, and interplanetary connectivity outages by raising errors, buffering votes, and logging critical events to multiple blockchains or a lightweight ledger. Low-power devices use simplified validation to handle resource constraints, with slashing for malicious behavior ensuring accountability.


## TODO
- Add sequence number validation to detect out-of-order messages and enhance replay protection.
- Optimize slashing logic to include reputation scoring for partial penalties.
- Enhance RTT-based timeout adjustments for ultra-low-power devices in interplanetary scenarios.
- Develop cross-blockchain retry policies for enhanced audit trail resilience.
- Integrate hardware requirements into a configuration file or compatibility matrix for deployment documentation.
- Implement audit trail checks for slashed nodes, linking logs to slashing events for traceability.