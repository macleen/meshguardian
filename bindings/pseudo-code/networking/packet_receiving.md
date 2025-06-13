# Packet Receiving Module

## Purpose
The Packet Receiving module processes incoming data packets in the MeshGuardian network, validating and handling them for further processing or forwarding. It integrates with the data transmission module to receive packets, logs events via the audit trail, and leverages a 64-bit capability flags system to support features such as ML-driven protocol selection (e.g., Bit 22), failure prediction (e.g., Bit 23), blockchain audit logging (e.g., Bit 24), and Asynchronous Delay-Tolerant Consensus (ADTC) vote processing for interplanetary environments (e.g., use of available Bit 40) with 15-30 minute RTTs

## Depends On
- /pseudo-code/networking/packet.md: Provides the Packet class for handling packets.
- /pseudo-code/networking/data_transmission.md: Manages the receipt of packets.
- /pseudo-code/audit/audit_trail.md: Logs packet receipt events.
- /pseudo-code/shared/constants.md: Uses DEFAULT_TIMEOUT for receipt timeouts.
- /pseudo-code/shared/helpers.md: Provides utility functions for validation and capability checks.
- /pseudo-code/security/crypto_utils.md: Validates ADTC vote signatures.

## Called By
- /pseudo-code/networking/data_transmission.md: Processes received packets as part of transmission.
- /pseudo-code/consensus.md: Receives ADTC votes.

## Used In
- **Use Case 5.15: Aid Relays**: Receives supply tracking data packets in crisis zones.
- **Use Case 5.16: Emergency Chat**: Receives real-time disaster messages.
- **Use Case 5.1.1: Mars Rover Data Relay**: Receives ADTC votes in high-latency networks.

## Pseudocode
```pseudocode
// Function to receive a packet
FUNCTION receive_packet(capability_flags: u64)
    // Determine timeout based on interplanetary mode (e.g., Bit 40)
    timeout = (capability_flags & BIT(40)) ? constants.A DTC_TIMEOUT : constants.DEFAULT_TIMEOUT
    packet = CALL receive_packet_from_network(capability_flags, timeout)
    // Validate packet
    IF packet IS NULL THEN
        CALL log_event("PacketReceiveFailed", {
            "error": "Timeout or invalid packet"
        }, capability_flags)
        RAISE PacketReceiveError("Failed to receive packet")
    END IF
    // Log receipt event
    details = {
        "id": packet.id,
        "source": packet.source,
        "dest": packet.dest,
        "profile": packet.profile
    }
    IF packet.profile == "interplanetary" AND packet.data.type == "ADTC_vote" THEN
        details.vector_clock = packet.headers.vector_clock
        details.weight = packet.data.weight
        public_key = GET_PUBLIC_KEY(packet.source)
        IF NOT crypto_utils.verify_signature(packet.data, packet.signature, public_key) THEN
            CALL log_event("PacketValidationFailed", {
                "id": packet.id,
                "error": "Invalid ADTC vote signature"
            }, capability_flags)
            RAISE PacketValidationError("Invalid ADTC vote signature")
        END IF
    END IF
    CALL log_event("PacketReceived", details, capability_flags)
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
- Validation: Validates packet integrity and ADTC vote signatures, with failures logged as Tier 2 events.  
- Audit Trail: Logs receipt events, using Solana (Tier 1) for critical packets (e.g., Bit 24 = 1) or local P2P sync (Tier 2), per the 64-bit capability flags specification.  
- ADTC Support: Processes ADTC votes with vector clock and weight metadata, using extended timeouts (ADTC_TIMEOUT) when interplanetary mode is enabled (e.g., Bit 40).  
- ML Support: Supports ML-driven protocol selection (e.g., Bit 22) and failure prediction (e.g., Bit 23) as defined in the 64-bit flags system.  
- 64-bit Transition: Bit positions have been updated from the 32-bit system; refer to the MeshGuardian 64-bit capability flags specification for exact mappings.  

## TODO
- Implement packet reassembly for fragmented packets.
- Add support for duplicate packet detection.
- Optimize ADTC vote validation for low-bandwidth networks.
- Verify compatibility of new 64-bit flags with existing ML and ADTC features.
