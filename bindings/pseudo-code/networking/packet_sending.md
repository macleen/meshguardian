# Packet Sending Module

## Purpose
The Packet Sending module handles the creation and transmission of data packets in the MeshGuardian network, ensuring packets are correctly formatted and sent to their destination. It integrates with the data transmission module to manage the actual sending process and logs events via the audit trail, supporting ML-driven protocol selection (Bit 13), failure prediction (Bit 14), and blockchain audit logging (Bit 15) for transparency and compliance.


## Depends On
- /pseudo-code/packet.md: Provides the Packet class for creating packets.
- /pseudo-code/networking/data_transmission.md: Manages the transmission of packets.
- /pseudo-code/audit_trail.md: Logs packet sending events.
- /pseudo-code/constants.md: Uses MAX_PACKET_SIZE and DEFAULT_TIMEOUT for constraints.
- /pseudo-code/shared/helpers.md: Provides utility functions for node ID and capability checks.

## Called By
- /pseudo-code/networking/data_transmission.md: Initiates packet sending as part of transmission.


## Used In
- **Use Case 5.15: Aid Relays**: Sends supply tracking data packets in crisis zones.
- **Use Case 5.16: Emergency Chat**: Sends real-time disaster messages.

## Pseudocode
```pseudocode
// Function to send a packet
FUNCTION send_packet(source, dest, data, profile, capability_flags)
    // Create a new packet
    packet = NEW Packet(source, dest, data, profile)
    // Log packet creation
    CALL log_event("PacketCreated", {
        "id": packet.id,
        "source": source,
        "dest": dest,
        "profile": profile
    }, capability_flags)
    // Transmit the packet
    success = CALL transmit_packet(packet, capability_flags)
    IF success THEN
        CALL log_event("PacketSent", {
            "id": packet.id,
            "source": source,
            "dest": dest
        }, capability_flags)
        RETURN TRUE
    ELSE
        CALL log_event("PacketSendFailed", {
            "id": packet.id,
            "source": source,
            "dest": dest
        }, capability_flags)
        RETURN FALSE
    END IF
END FUNCTION
```

---

## Notes
- Integration: Packet sending relies on the data transmission module for actual network operations, ensuring compatibility with DTNs.
- Audit Trail: Packet creation and sending events are logged, using Solana (Tier 1) for critical packets if Bit 15 = 1, or local P2P sync (Tier 2).
- ML Support: Capability flags enable ML-driven protocol selection (Bit 13) and failure prediction (Bit 14) for optimized sending.

## TODO
- Add support for packet prioritization based on network congestion.
- Implement retry policies for failed sends with exponential backoff.
