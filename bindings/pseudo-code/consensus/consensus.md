# Consensus Module

## Purpose
The Consensus module ensures decentralized agreement on data validity, routing, and blockchain updates in MeshGuardian’s mesh network. It supports Proof-of-Stake (PoS) for low-priority packets, Practical Byzantine Fault Tolerance (PBFT) for high-priority packets, and Asynchronous Delay-Tolerant Consensus (ADTC) for extreme-delay environments (15-30 minute RTTs) in the Interplanetary Profile. The module is modular, allowing new consensus mechanisms via the Pluggable Protocol Engine, with robust error handling and logging for traceability. For interplanetary deployments, ADTC incorporates dynamic timeout adjustments and robust buffering to mitigate disruptions from unpredictable orbital dynamics or prolonged outages.

## Interfaces
- **validate_packet(packet, nodes, consensus_mode)**: Validates a packet using the specified consensus mode (PoS, PBFT, ADTC).
- **propose_route(packet, nodes, consensus_mode)**: Proposes a routing decision for validation.
- **commit_packet(packet, nodes, consensus_mode)**: Commits a validated packet to the blockchain or local ledger.

## Depends On
- **/pseudo-code/helpers.md**: Provides utilities for logical clock management, vote validation, and RTT estimation.
- **/pseudo-code/logging/logger.md**: Logs consensus events and errors.
- **/pseudo-code/constants.md**: Defines consensus mode codes, ADTC parameters, and RTT thresholds.
- **/pseudo-code/security/crypto_utils.md**: Provides cryptographic functions for vote signing.
- **/pseudo-code/audit_trail.md**: Logs consensus outcomes for traceability.

## Called By
- /pseudo-code/networking/routing.md: Validates routing decisions.
- /pseudo-code/audit_trail.md: Logs consensus outcomes.

## Used In
- Use Case 5.2.1: Military Operation: Uses PBFT for secure, high-priority validation.
- Use Case 5.3.1: Hurricane Relief: Employs PoS for efficient coordination.
- Use Case 5.1.1: Mars Rover Data Relay: Uses ADTC for reliable consensus in high-latency networks.

## Pseudocode
```pseudocode
// Structure for consensus context
STRUCT ConsensusContext
    packet: Packet              // Packet to validate
    nodes: LIST[Node]          // Participating nodes
    consensus_mode: INTEGER    // Consensus mode (PoS, PBFT, ADTC)
    capability_flags: BITFIELD // 32-bit Capability Flags
    vector_clock: MAP[NodeID, INTEGER]  // Vector clock for ADTC
    node_weights: MAP[NodeID, FLOAT]    // Weights based on uptime, success rate, solar exposure
END STRUCT

// Function to calculate node weight
FUNCTION calculate_node_weight(node: Node) RETURNS FLOAT
    uptime_score = node.uptime / MAX_UPTIME  // Normalize uptime
    success_score = node.transmission_success_rate  // 0 to 1
    solar_score = node.solar_exposure / MAX_SOLAR  // Normalize solar exposure
    RETURN (0.4 * uptime_score + 0.4 * success_score + 0.2 * solar_score)
END FUNCTION

// Function to adjust ADTC timeout based on RTT estimates
FUNCTION adjust_adtc_timeout(context: ConsensusContext) RETURNS INTEGER
    TRY
        // Estimate real-time RTT using orbital data or historical trends
        current_rtt = CALL helpers.estimate_rtt(context.nodes)
        IF current_rtt > constants.RTT_THRESHOLD THEN
            // Increase timeout for prolonged delays
            new_timeout = constants.A DTC_TIMEOUT * (current_rtt / constants.RTT_THRESHOLD)
        ELSE
            // Use default timeout for stable conditions
            new_timeout = constants.A DTC_TIMEOUT
        END IF
        // Cap timeout to prevent excessive delays
        new_timeout = MIN(new_timeout, constants.A DTC_TIMEOUT * 2)
        CALL log_message("INFO", "ADTC timeout adjusted to " + new_timeout + " seconds, RTT: " + current_rtt, context.capability_flags)
        RETURN new_timeout
    CATCH error
        CALL log_message("ERROR", "Failed to adjust ADTC timeout: " + error, context.capability_flags | BIT_15) // Log as Tier 1
        RETURN constants.A DTC_TIMEOUT // Fallback to default
    END TRY
END FUNCTION

// Function to validate a packet
FUNCTION validate_packet(context: ConsensusContext)
    TRY
        IF context.packet IS NULL THEN
            RAISE ValidationError("Packet is null")
        END IF
        CASE context.consensus_mode
            WHEN constants.POS_CONSENSUS_CODE:
                RETURN validate_pos(context)
            WHEN constants.PBFT_CONSENSUS_CODE:
                RETURN validate_pbft(context)
            WHEN constants.A DTC_CONSENSUS_CODE:
                IF NOT (context.capability_flags & BIT_28) THEN
                    RAISE ConsensusError("ADTC not supported")
                END IF
                RETURN validate_adtc(context)
            DEFAULT:
                RAISE ConsensusError("Unsupported consensus mode: " + context.consensus_mode)
        END CASE
    CATCH error
        CALL log_message("ERROR", "Packet validation failed: " + error, context.capability_flags | BIT_15) // Log as Tier 1
        RAISE error
    END TRY
END FUNCTION

// Function to validate using PoS
FUNCTION validate_pos(context: ConsensusContext)
    TRY
        stakes = CALCULATE_STAKES(context.nodes)
        votes = COLLECT_VOTES(context.packet, context.nodes, stakes)
        IF SUM(votes) >= THRESHOLD(stakes) THEN
            CALL log_message("INFO", "PoS consensus achieved for packet: " + context.packet.id, context.capability_flags)
            RETURN True
        ELSE
            RAISE ConsensusError("PoS consensus failed")
        END IF
    CATCH error
        CALL log_message("ERROR", "PoS validation failed: " + error, context.capability_flags | BIT_15) // Log as Tier 1
        RAISE error
    END TRY
END FUNCTION

// Function to validate using PBFT
FUNCTION validate_pbft(context: ConsensusContext)
    TRY
        pre_prepare = PROPOSE(context.packet, context.nodes)
        prepare_votes = COLLECT_VOTES(pre_prepare, context.nodes)
        IF COUNT(prepare_votes) >= (2 * FAULTY_NODES + 1) THEN
            commit_votes = COMMIT(pre_prepare, context.nodes)
            IF COUNT(commit_votes) >= (2 * FAULTY_NODES + 1) THEN
                CALL log_message("INFO", "PBFT consensus achieved for packet: " + context.packet.id, context.capability_flags)
                RETURN True
            END IF
        END IF
        RAISE ConsensusError("PBFT consensus failed")
    CATCH error
        CALL log_message("ERROR", "PBFT validation failed: " + error, context.capability_flags | BIT_15) // Log as Tier 1
        RAISE error
    END TRY
END FUNCTION

// Function to validate using ADTC
FUNCTION validate_adtc(context: ConsensusContext)
    TRY
        // Update vector clock
        context.vector_clock[CURRENT_NODE_ID] = CALL helpers.increment_lamport_clock(context.vector_clock[CURRENT_NODE_ID])
        vote = {
            "packet_hash": HASH(context.packet),
            "vector_clock": context.vector_clock,
            "node_id": CURRENT_NODE_ID,
            "weight": CALL calculate_node_weight(CURRENT_NODE)
        }
        signature = crypto_utils.sign_vote(vote, PRIVATE_KEY)
        
        // Adjust timeout dynamically based on RTT
        timeout = CALL adjust_adtc_timeout(context)
        
        // Buffer votes with prioritized persistence
        votes = BUFFER_VOTES(vote, signature, context.nodes, timeout=timeout)
        weighted_sum = 0
        valid_votes = []
        FOR each received_vote IN votes
            public_key = GET_PUBLIC_KEY(received_vote.node_id)
            IF CALL helpers.validate_adtc_vote(received_vote, received_vote.signature, public_key) THEN
                context.vector_clock = CALL helpers.merge_vector_clock(context.vector_clock, received_vote.vector_clock)
                weighted_sum += received_vote.weight
                APPEND valid_votes, received_vote
                // Persist vote to local storage for prolonged outages
                CALL store_vote_locally(received_vote, constants.MAX_BUFFER_PERSISTENCE)
            END IF
        END FOR

        // Validate votes (require 2f+1 weighted votes)
        total_nodes = LENGTH(context.nodes)
        max_faulty_nodes = FLOOR((total_nodes - 1) / 3)
        IF total_nodes > 3 THEN
            // Use vector clocks for >3 nodes
            quorum_weight = 0.67 * SUM(context.node_weights.values()) // 2/3 of total weight
            IF weighted_sum >= quorum_weight THEN
                CALL log_message("INFO", "ADTC consensus achieved for packet: " + context.packet.id + 
                                 ", vector_clock: " + context.vector_clock, context.capability_flags | BIT_15)
                // Attempt blockchain anchoring with fallback
                TRY
                    CALL log_event("ADTC_Anchoring", {
                        "packet_id": context.packet.id,
                        "vector_clock": context.vector_clock
                    }, context.capability_flags | BIT_15)
                CATCH blockchain_error
                    // Retry on alternative blockchain (e.g., Avalanche)
                    CALL log_event("ADTC_Anchoring_Retry", {
                        "packet_id": context.packet.id,
                        "vector_clock": context.vector_clock,
                        "fallback_blockchain": "Avalanche"
                    }, context.capability_flags | BIT_15)
                END TRY
                RETURN True
            END IF
        ELSE
            // Use Lamport clocks for <=3 nodes
            IF COUNT(valid_votes) >= (2 * max_faulty_nodes + 1) THEN
                CALL log_message("INFO", "ADTC consensus achieved for packet: " + context.packet.id + 
                                 ", vector_clock: " + context.vector_clock, context.capability_flags | BIT_15)
                // Attempt blockchain anchoring with fallback
                TRY
                    CALL log_event("ADTC_Anchoring", {
                        "packet_id": context.packet.id,
                        "vector_clock": context.vector_clock
                    }, context.capability_flags | BIT_15)
                CATCH blockchain_error
                    // Retry on alternative blockchain (e.g., Avalanche)
                    CALL log_event("ADTC_Anchoring_Retry", {
                        "packet_id": context.packet.id,
                        "vector_clock": context.vector_clock,
                        "fallback_blockchain": "Avalanche"
                    }, context.capability_flags | BIT_15)
                END TRY
                RETURN True
            END IF
        END IF
        RAISE ConsensusError("ADTC consensus failed")
    CATCH error
        CALL log_message("ERROR", "ADTC validation failed: " + error, context.capability_flags | BIT_15) // Log as Tier 1
        RAISE error
    END TRY
END FUNCTION

// Function to store votes locally for persistence
FUNCTION store_vote_locally(vote, max_persistence)
    TRY
        // Store vote in local storage with expiration
        CALL store_local_log(vote, expiration=max_persistence)
        CALL log_message("INFO", "Vote stored locally for persistence: " + vote.node_id, capability_flags)
    CATCH storage_error
        CALL log_message("ERROR", "Failed to store vote locally: " + storage_error, capability_flags | BIT_15) // Log as Tier 1
        RAISE storage_error
    END TRY
END FUNCTION

// Function to propose a route
FUNCTION propose_route(packet, nodes, consensus_mode)
    TRY
        context = NEW ConsensusContext(packet, nodes, consensus_mode)
        RETURN validate_packet(context)
    CATCH error
        CALL log_message("ERROR", "Route proposal failed: " + error, capability_flags | BIT_15) // Log as Tier 1
        RAISE error
    END TRY
END FUNCTION

// Function to commit a packet
FUNCTION commit_packet(packet, nodes, consensus_mode)
    TRY
        context = NEW ConsensusContext(packet, nodes, consensus_mode)
        IF validate_packet(context) THEN
            CALL log_message("INFO", "Packet committed: " + packet.id, capability_flags | BIT_15)
            IF consensus_mode == constants.A DTC_CONSENSUS_CODE THEN
                // Periodic anchoring for long-term missions
                IF MISSION_DURATION > 6_MONTHS AND IS_PERIHELION_WINDOW() THEN
                    TRY
                        CALL log_event("ADTC_Anchoring", {
                            "packet_id": packet.id,
                            "vector_clock": context.vector_clock
                        }, capability_flags | BIT_15)
                    CATCH blockchain_error
                        // Retry on alternative blockchain (e.g., Avalanche)
                        CALL log_event("ADTC_Anchoring_Retry", {
                            "packet_id": packet.id,
                            "vector_clock": context.vector_clock,
                            "fallback_blockchain": "Avalanche"
                        }, capability_flags | BIT_15)
                    END TRY
                END IF
            END IF
            RETURN True
        END IF
        RAISE ConsensusError("Commit failed")
    CATCH error
        CALL log_message("ERROR", "Packet commit failed: " + error, capability_flags | BIT_15) // Log as Tier 1
        RAISE error
    END TRY
END FUNCTION
```

---

## Notes
- Consensus Modes: Supports PoS (low-priority), PBFT (high-priority), and ADTC (interplanetary, high-latency) via consensus_mode.
- ADTC Features: Uses vector clocks for >3 nodes, Lamport clocks for ≤3 nodes, weighted votes (uptime, success rate, solar exposure), and a dynamic timeout (adjusted based on RTT estimates, default 30 minutes). Tolerates f faulty nodes in a 3f+1 network. Buffering persists votes locally during prolonged outages, with periodic checkpointing and opportunistic blockchain anchoring (Solana primary, Avalanche fallback).
- Capability Flags: ADTC requires Bit 28 and Interplanetary Profile (Bits 20-21 = 10). Bits 28–31 reserved for interplanetary features.
- Performance: Minimizes communication overhead by buffering votes, optimized for sparse networks. Dynamic timeout adjustments reduce consensus delays in variable RTT conditions.
- Security: Uses Schnorr or Dilithium signatures for vote authenticity, logged as Tier 1 events.
- Edge Cases: Handles node failures, delayed votes, clock overflows, and connectivity outages, falling back to PoS if ADTC is unsupported. Robust buffering ensures vote persistence during prolonged interplanetary outages.
- On-Chain Anchoring: Anchors cumulative ADTC states to Solana (or Avalanche fallback) during perihelion windows for missions >6 months, with retries for failed transactions.

## TODO
- Optimize ADTC for ultra-low-power devices with minimal buffering.
- Implement predictive RTT modeling for more accurate timeout adjustments.
- Add cross-blockchain retry policies for enhanced anchoring resilience.
- Integrate with MeshGuardian ML Hub for predictive vote prioritization.