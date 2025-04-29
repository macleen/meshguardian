# Data Transmission Module

## Purpose
The Data Transmission module manages the sending and receiving of data packets across the MeshGuardian network, ensuring reliable and efficient communication in Delay-Tolerant Networks (DTNs). It handles packet transmission, retransmission, priority scheduling, and supports ML-driven protocol selection (Bit 13), failure prediction (Bit 14), blockchain audit logging (Bit 15), and Asynchronous Delay-Tolerant Consensus (ADTC) vote transmission for interplanetary environments with 15-30 minute RTTs. The module integrates with the audit trail to log transmission events, aligning with the system’s zero-trust security model and transparency requirements.

## Depends On

- /pseudo-code/networking/packet.md: Provides the Packet class for structuring data packets.
- /pseudo-code/audit/audit_trail.md: Logs transmission and receipt events.
- /pseudo-code/shared/helpers.md: Provides utility functions for node ID retrieval and timeout management.
- /pseudo-code/shared/constants.md: Uses MAX_PACKET_SIZE and DEFAULT_TIMEOUT for transmission constraints.
- /pseudo-code/networking/packet_buffering.md: Buffers packets during high-latency periods.

## Called By
- **/pseudo-code/networking/packet_sending.md**: Initiates packet transmission.
- **/pseudo-code/networking/packet_receiving.md**: Processes received packets.
- **/pseudo-code/consensus.md**: Sends ADTC votes.

## Used In
- **Use Case 5.15: Aid Relays**: Facilitates multi-hop supply tracking in crisis zones.  
- **Use Case 5.16: Emergency Chat**: Enables real-time disaster messaging with priority scheduling.
- **Use Case 5.1.1: Mars Rover Data Relay**: Transmits ADTC votes in high-latency networks.

## Pseudocode
The constants are defined as immutable values that can be accessed throughout the system. They are grouped here for clarity and ease of maintenance.
```pseudocode

/// Function to transmit a packet
FUNCTION transmit_packet(packet, capability_flags)
    // Validate packet size
    IF LENGTH(packet.data) > constants.MAX_PACKET_SIZE THEN
        RAISE PacketSizeError("Packet exceeds MAX_PACKET_SIZE")
    END IF
    // Set priority based on profile
    priority = packet.profile = "emergency" ? "high" : "standard"
    // Handle ADTC votes
    IF packet.profile = "interplanetary" AND packet.data.type = "ADTC_vote" THEN
        timeout = constants.A DTC_TIMEOUT
        details = {
            "id": packet.id,
            "source": packet.source,
            "dest": packet.dest,
            "priority": priority,
            "vector_clock": packet.headers.vector_clock,
            "weight": packet.data.weight
        }
    ELSE
        timeout = constants.DEFAULT_TIMEOUT
        details = {
            "id": packet.id,
            "source": packet.source,
            "dest": packet.dest,
            "priority": priority
        }
    END IF
    // Log transmission event
    CALL log_event("PacketTransmission", details, capability_flags)
    // Attempt transmission
    TRY
        CALL send_packet(packet, timeout)
        RETURN TRUE
    CATCH transmission_error
        // Log retransmission attempt
        CALL log_event("RetransmissionAttempt", {
            "id": packet.id,
            "error": transmission_error
        }, capability_flags)
        // For ADTC votes, buffer instead of immediate retry
        IF packet.data.type = "ADTC_vote" THEN
            CALL packet_buffering.enqueue(packet)
            RETURN FALSE
        END IF
        // Retry up to 3 times for non-ADTC packets
        FOR attempt IN 1 TO constants.DEFAULT_RETRY_LIMIT
            TRY
                CALL send_packet(packet, timeout)
                RETURN TRUE
            CATCH retry_error
                // Log retry failure
                CALL log_event("RetransmissionAttempt", {
                    "id": packet.id,
                    "attempt": attempt,
                    "error": retry_error
                }, capability_flags)
            END TRY
        END FOR
        RETURN FALSE
    END TRY
END FUNCTION

// Function to receive a packet
FUNCTION receive_packet(capability_flags)
    // Wait for incoming packet
    packet = CALL wait_for_packet(constants.DEFAULT_TIMEOUT)
    // Validate packet
    IF packet IS NULL THEN
        RAISE PacketTimeoutError("No packet received within timeout")
    END IF
    // Log receipt event
    details = {
        "id": packet.id,
        "source": packet.source,
        "dest": packet.dest,
        "priority": packet.profile = "emergency" ? "high" : "standard"
    }
    IF packet.profile = "interplanetary" AND packet.data.type = "ADTC_vote" THEN
        details.vector_clock = packet.headers.vector_clock
        details.weight = packet.data.weight
    END IF
    CALL log_event("PacketReceipt", details, capability_flags)
    RETURN packet
END FUNCTION

```

---

## Notes
- Priority Scheduling: High-priority packets (e.g., emergency profile) are transmitted first, logged as Tier 1 events when Bit 15 = 1.
- Reliability: Retransmission attempts (up to DEFAULT_RETRY_LIMIT) for non-ADTC packets; ADTC votes are buffered for opportunistic forwarding.
- Blockchain Audit: Critical transmission/receipt events, including ADTC votes, are logged to Solana (Tier 1) if Bit 15 = 1; standard events use local P2P sync (Tier 2).
- ADTC Support: Handles ADTC vote transmission with extended timeouts (ADTC_TIMEOUT) and buffering, logging vector clock and weight metadata.
- ML Integration: Supports ML-driven protocol selection (Bit 13) and failure prediction (Bit 14) via capability flags.

## TODO
- Implement adaptive retransmission strategies based on network conditions.
- Add support for packet fragmentation for large payloads.
- Optimize ADTC vote buffering for low-bandwidth interplanetary networks.