# Packet Sending Module

## Purpose
The Packet Sending module handles the creation and transmission of data packets in the MeshGuardian network, ensuring packets are correctly formatted and sent to their destination. It integrates with the data transmission module to manage sending, logs events via the audit trail, and supports ML-driven protocol selection (Bit 13), failure prediction (Bit 14), blockchain audit logging (Bit 15), and Asynchronous Delay-Tolerant Consensus (ADTC) vote sending for interplanetary environments with 15-30 minute RTTs.

## Depends On
- /pseudo-code/networking/packet.md: Provides the Packet class for creating packets.
- /pseudo-code/networking/data_transmission.md: Manages the transmission of packets.
- /pseudo-code/audit/audit_trail.md: Logs packet sending events.
- /pseudo-code/constants.md: Uses MAX_PACKET_SIZE and DEFAULT_TIMEOUT for constraints.
- /pseudo-code/shared/helpers.md: Provides utility functions for node ID and capability checks.
- /pseudo-code/networking/packet_buffering.md: Buffers ADTC votes.capability checks.

## Called By
- /pseudo-code/networking/data_transmission.md: Initiates packet sending as part of transmission.
- /pseudo-code/consensus/consensus.md: Sends ADTC votes.

## Used In
- **Use Case 5.15: Aid Relays**: Sends supply tracking data packets in crisis zones.
- **Use Case 5.16: Emergency Chat**: Sends real-time disaster messages.
- **Use Case 5.1.1**: Mars Rover Data Relay: Sends ADTC votes in high-latency networks.

## Pseudocode
```pseudocode
// Function to send a packet
FUNCTION send_packet(source, dest, data, profile, capability_flags)
    // Create a new packet
    packet = NEW Packet(source, dest, data, profile)
    // Log packet creation
    details = {
        "id": packet.id,
        "source": source,
        "dest": dest,
        "profile": profile
    }
    IF profile = "interplanetary" AND data.type = "ADTC_vote" THEN
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
- Integration: Relies on data_transmission.md for network operations, supporting DTNs.
- Audit Trail: Logs packet creation and sending, using Solana (Tier 1) for critical packets (Bit 15 = 1) or local P2P sync (Tier 2).
- ADTC Support: Handles ADTC vote packets with vector clock and weight metadata, buffering via packet_buffering.md.
- ML Support: Supports ML-driven protocol selection (Bit 13) and failure prediction (Bit 14).

## TODO
- Add support for packet prioritization based on network congestion.
- Implement retry policies with exponential backoff.
- Optimize ADTC vote sending for low-bandwidth networks.
