# Packet Sending Module

## Purpose
The Packet Sending module handles the creation and transmission of data packets in the MeshGuardian network, ensuring packets are correctly formatted and sent to their destination. It integrates with the data transmission module to manage sending, logs events via the audit trail, and supports features controlled by a 64-bit capability flags system, such as ML-driven protocol selection, failure prediction, blockchain audit logging, and Asynchronous Delay-Tolerant Consensus (ADTC) vote sending for interplanetary environments with 15-30 minute RTTs. This module has been updated to support the transition to a 64-bit `capability_flags` structure; refer to `protocol-specs/capability_flags.md` for flag definitions.

## Depends On
- `/pseudo-code/networking/packet.md`: Provides the Packet class for creating packets.
- `/pseudo-code/networking/data_transmission.md`: Manages the transmission of packets.
- `/pseudo-code/audit/audit_trail.md`: Logs packet sending events.
- `/pseudo-code/constants.md`: Uses `MAX_PACKET_SIZE` and `DEFAULT_TIMEOUT` for constraints.
- `/pseudo-code/shared/helpers.md`: Provides utility functions for node ID and capability checks.
- `/pseudo-code/networking/packet_buffering.md`: Buffers ADTC votes.

## Called By
- `/pseudo-code/networking/data_transmission.md`: Initiates packet sending as part of transmission.
- `/pseudo-code/consensus/consensus.md`: Sends ADTC votes.

## Used In
- **Use Case 5.15: Aid Relays**: Sends supply tracking data packets in crisis zones.
- **Use Case 5.16: Emergency Chat**: Sends real-time disaster messages.
- **Use Case 5.1.1: Mars Rover Data Relay**: Sends ADTC votes in high-latency networks.

## Pseudocode
```pseudocode
// Function to send a packet
FUNCTION send_packet(source, dest, data, profile, capability_flags: u64)
    // Create a new packet
    packet = NEW Packet(source, dest, data, profile)
    // Log packet creation
    details = {
        "id": packet.id,
        "source": source,
        "dest": dest,
        "profile": profile
    }
    IF profile == "interplanetary" AND data.type == "ADTC_vote" THEN
        details.vector_clock = packet.headers.vector_clock
        details.weight = data.weight
    END IF
    CALL log_event("PacketCreated", details, capability_flags)
    // Transmit the packet
    success = CALL transmit_packet(packet, capability_flags)
    IF success THEN
        CALL log_event("PacketSent", details, capability_flags)
        RETURN TRUE
    ELSE
        CALL log_event("PacketSendFailed", details, capability_flags)
        RETURN FALSE
    END IF
END FUNCTION
```

---

## Notes
- Integration: Relies on data_transmission.md for network operations, supporting Delay-Tolerant Networks (DTNs).  
- Audit Trail: Logs packet creation and sending, using blockchain (Tier 1) for critical packets (e.g., when constants.BLOCKCHAIN_AUDIT_BIT is set) or local P2P sync (Tier 2).  
- ADTC Support: Handles ADTC vote packets with vector clock and weight metadata, buffering via packet_buffering.md.  
- ML Support: Supports ML-driven protocol selection (e.g., constants.- - ML_PROTOCOL_SELECTION_BIT) and failure prediction (e.g., constants.ML_FAILURE_PREDICTION_BIT).  
- 64-bit Transition: capability_flags is now a 64-bit integer. Bit positions are defined in /pseudo-code/shared/constants.md to avoid hardcoding and ensure maintainability. Refer to protocol-specs/capability_flags.md for the full specification.  

## TODO
- Add support for packet prioritization based on network congestion.
- Implement retry policies with exponential backoff.
- Optimize ADTC vote sending for low-bandwidth networks.  
- Verify bit positions for capability flags with the latest specification.
