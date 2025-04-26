# Packet Receiving Module

## Purpose
The Packet Receiving module processes incoming data packets in the MeshGuardian network, validating and handling them for further processing or forwarding. It integrates with the data transmission module to receive packets and logs events via the audit trail, supporting ML-driven protocol selection (Bit 13), failure prediction (Bit 14), and blockchain audit logging (Bit 15) for transparency and compliance.


## Depends On
- /pseudo-code/packet.md: Provides the Packet class for handling packets.
- /pseudo-code/networking/data_transmission.md: Manages the receipt of packets.
- /pseudo-code/audit_trail.md: Logs packet receipt events.
- /pseudo-code/constants.md: Uses DEFAULT_TIMEOUT for receipt timeouts.
- /pseudo-code/shared/helpers.md: Provides utility functions for validation and capability checks.

## Called By
- /pseudo-code/networking/data_transmission.md: Processes received packets as part of transmission.

## Used In
- **Use Case 5.15: Aid Relays**: Receives supply tracking data packets in crisis zones.
- **Use Case 5.16: Emergency Chat**: Receives real-time disaster messages.

## Pseudocode
```pseudocode
// Function to receive a packet
FUNCTION receive_packet(capability_flags)
    // Receive packet from the network
    packet = CALL receive_packet_from_network(capability_flags)
    // Validate packet
    IF packet IS NULL THEN
        CALL log_event("PacketReceiveFailed", {
            "error": "Timeout or invalid packet"
        }, capability_flags)
        RAISE PacketReceiveError("Failed to receive packet")
    END IF
    // Log receipt event
    CALL log_event("PacketReceived", {
        "id": packet.id,
        "source": packet.source,
        "dest": packet.dest,
        "profile": packet.profile
    }, capability_flags)
    // Validate packet integrity
    IF NOT CALL validate_packet(packet) THEN
        CALL log_event("PacketValidationFailed", {
            "id": packet.id,
            "error": "Invalid packet data"
        }, capability_flags)
        RAISE PacketValidationError("Packet validation failed")
    END IF
    RETURN packet
END FUNCTION
```

---

## Notes
- Validation: Packets are validated for integrity and correctness before processing, with failures logged as Tier 2 events.
- Audit Trail: Receipt events are logged, using Solana (Tier 1) for critical packets if Bit 15 = 1, or local P2P sync (Tier 2).
- ML Support: Capability flags enable ML-driven protocol selection (Bit 13) and failure prediction (Bit 14) for optimized receiving.

## TODO
- Implement packet reassembly for fragmented packets.
- Add support for duplicate packet detection.
