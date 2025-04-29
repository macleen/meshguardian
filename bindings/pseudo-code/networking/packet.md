# Packet Module

## Purpose
The Packet class represents a data packet in the MeshGuardian network, encapsulating attributes like a unique identifier, source, destination, data payload, communication profile, capability flags, and vector clocks for Asynchronous Delay-Tolerant Consensus (ADTC). It provides methods for initializing packets, converting to dictionaries, and creating instances from dictionaries, supporting ML-driven protocol selection (Bit 13), failure prediction (Bit 14), blockchain audit logging (Bit 15), and ADTC for interplanetary environments.

## Interfaces
- `__init__(source, dest, data, profile="default")`: Initializes a new packet with the given attributes.
- `to_dict()`: Converts the packet's attributes into a dictionary.
- `from_dict(packet_dict)`: Creates a packet instance from a dictionary.

## Depends On
- `/pseudo-code/shared/helpers.md`: Provides vector clock utilities for ADTC.
- `/pseudo-code/audit/audit_trail.md`: Logs packet initialization events.

## Called By
- `/pseudo-code/networking/packet_sending.md`: Creates and sends packets.
- `/pseudo-code/networking/packet_receiving.md`: Handles incoming packets.
- `/pseudo-code/networking/data_transmission.md`: Manages packet transmission.

## Used In
- **Use Case 5.15: Aid Relays**: Multi-hop supply tracking in crisis zones.
- **Use Case 5.16: Emergency Chat**: Real-time disaster messaging.
- **Use Case 5.1.1: Mars Rover Data Relay**: ADTC vote packets in high-latency networks.


## Pseudocode
```pseudocode
CLASS Packet
    // Attributes
    id: String              // Unique packet identifier
    source: String          // Sender node ID
    dest: String            // Receiver node ID
    data: Bytes             // Payload data
    profile: String         // Communication profile (e.g., "default")
    capability_flags: Integer  // 32-bit capability flags
    headers: PacketHeaders     // Header metadata including vector clock

    METHOD __init__(source, dest, data, profile="default")
        // Generate a unique identifier
        self.id = CALL generate_uuid()
        // Set attributes
        self.source = source
        self.dest = dest
        self.data = data
        self.profile = profile
        // Initialize capability flags
        self.capability_flags = CALL get_capability_flags()
        IF CALL node_supports_ml_protocol_selection() THEN
            SET self.capability_flags BIT_13 TO 1
        ELSE
            SET self.capability_flags BIT_13 TO 0
        END IF
        IF CALL node_supports_ml_failure_prediction() THEN
            SET self.capability_flags BIT_14 TO 1
        ELSE
            SET self.capability_flags BIT_14 TO 0
        END IF
        IF CALL node_supports_blockchain_audit() THEN
            SET self.capability_flags BIT_15 TO 1
        ELSE
            SET self.capability_flags BIT_15 TO 0
        END IF
        IF CALL node_supports_adtc() THEN
            SET self.capability_flags BIT_28 TO 1
        ELSE
            SET self.capability_flags BIT_28 TO 0
        END IF
        // Initialize headers with vector clock for ADTC
        self.headers = NEW PacketHeaders(profile)
        IF profile = "interplanetary" AND data.type = "ADTC_vote" THEN
            self.headers.vector_clock[CURRENT_NODE_ID] = CALL helpers.increment_lamport_clock(0)
        END IF
        CALL log_event("PacketInitialized", {
            "id": self.id,
            "capability_flags": self.capability_flags,
            "vector_clock": self.headers.vector_clock
        }, self.capability_flags)
    END METHOD

    METHOD to_dict()
        RETURN {
            "id": self.id,
            "source": self.source,
            "dest": self.dest,
            "data": self.data,
            "profile": self.profile,
            "capability_flags": self.capability_flags,
            "headers": self.headers.to_dict()
        }
    END METHOD

    CLASS METHOD from_dict(packet_dict)
        packet = NEW Packet(packet_dict["source"], packet_dict["dest"], packet_dict["data"], packet_dict["profile"])
        packet.id = packet_dict["id"]
        packet.capability_flags = packet_dict["capability_flags"]
        packet.headers = PacketHeaders.from_dict(packet_dict["headers"])
        RETURN packet
    END METHOD
END CLASS
```  

---

## Notes
- Purpose: Manages packet structure, including vector clocks for ADTC event ordering.
- Capability Flags: Supports ML (Bits 13, 14), blockchain audit (Bit 15), and ADTC (Bit 28). Bits 28–31 reserved for interplanetary features.
- ADTC Support: Includes vector clock in headers for ADTC vote packets, initialized during creation.
- Validation: Recommended for packet attributes to ensure data integrity.

## TODO
- Implement packet attribute validation.
- Add support for extended headers or metadata fields.
- Optimize vector clock storage for low-memory devices.
