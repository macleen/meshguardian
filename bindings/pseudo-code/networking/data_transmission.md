# Data Transmission Module

## Purpose
The Data Transmission module manages the sending and receiving of data packets across the MeshGuardian network, ensuring reliable and efficient communication in Delay-Tolerant Networks (DTNs). It handles packet transmission, retransmission, and priority scheduling, supporting ML-driven protocol selection (Bit 13), failure prediction (Bit 14), and blockchain audit logging (Bit 15) for critical events. The module integrates with the audit trail to log transmission events, aligning with the system’s zero-trust security model and transparency requirements.


## Depends On

- /pseudo-code/packet.md: Provides the Packet class for structuring data packets.
- /pseudo-code/audit_trail.md: Logs transmission and receipt events.
- /pseudo-code/shared/helpers.md: Provides utility functions for node ID retrieval and timeout management.
- /pseudo-code/constants.md: Uses MAX_PACKET_SIZE and DEFAULT_TIMEOUT for transmission constraints.

## Called By
- **/pseudo-code/networking/packet_sending.md**: Initiates packet transmission.
- **/pseudo-code/networking/packet_receiving.md**: Processes received packets.


## Used In
- **Use Case 5.15: Aid Relays**: Facilitates multi-hop supply tracking in crisis zones.  
- **Use Case 5.16: Emergency Chat**: Enables real-time disaster messaging with priority scheduling.

## Pseudocode
The constants are defined as immutable values that can be accessed throughout the system. They are grouped here for clarity and ease of maintenance.
```pseudocode

/// Function to transmit a packet
FUNCTION transmit_packet(packet, capability_flags)
    // Validate packet size
    IF LENGTH(packet.data) > MAX_PACKET_SIZE THEN
        RAISE PacketSizeError("Packet exceeds MAX_PACKET_SIZE")
    END IF
    // Set priority based on profile
    priority = packet.profile = "emergency" ? "high" : "standard"
    // Log transmission event
    CALL log_event("PacketTransmission", {
        "id": packet.id,
        "source": packet.source,
        "dest": packet.dest,
        "priority": priority
    }, capability_flags)
    // Attempt transmission
    timeout = DEFAULT_TIMEOUT
    TRY
        CALL send_packet(packet, timeout)
        RETURN TRUE
    CATCH transmission_error
        // Log retransmission attempt
        CALL log_event("RetransmissionAttempt", {
            "id": packet.id,
            "error": transmission_error
        }, capability_flags)
        // Retry up to 3 times
        FOR attempt IN 1 TO 3
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
    packet = CALL wait_for_packet(DEFAULT_TIMEOUT)
    // Validate packet
    IF packet IS NULL THEN
        RAISE PacketTimeoutError("No packet received within timeout")
    END IF
    // Log receipt event
    CALL log_event("PacketReceipt", {
        "id": packet.id,
        "source": packet.source,
        "dest": packet.dest,
        "priority": packet.profile = "emergency" ? "high" : "standard"
    }, capability_flags)
    RETURN packet
END FUNCTION

```

---

## Notes
- **Priority Scheduling**: High-priority packets (e.g., emergency profile) are transmitted first, logged as Tier 1 events when Bit 15 = 1.
- **Reliability**: Retransmission attempts (up to 3) ensure reliability in DTNs, with events logged as Tier 2.
- **Blockchain Audit**: Critical transmission/receipt events are logged to Solana (Tier 1) if Bit 15 is enabled; standard events use local P2P sync (Tier 2).
- **ML Integration**: Supports ML-driven protocol selection (Bit 13) and failure prediction (Bit 14) via capability flags.

## TODO
- Implement adaptive retransmission strategies based on network conditions.
- Add support for packet fragmentation for large payloads.