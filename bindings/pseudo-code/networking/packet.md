# Packet Module

## Purpose
The `Packet` class represents a data packet in the MeshGuardian network, encapsulating attributes like a unique identifier, source, destination, data payload, communication profile, capability flags, and vector clocks for Asynchronous Delay-Tolerant Consensus (ADTC). It provides methods for initializing packets, converting to dictionaries, and creating instances from dictionaries, supporting features controlled by a 64-bit capability flags system, such as ML-driven protocol selection, failure prediction, blockchain audit logging, and ADTC for interplanetary environments. This class has been updated to support the transition to a 64-bit `capability_flags` structure; refer to `protocol-specs/capability_flags.md` for flag definitions.

## Interfaces
- `__init__(source, dest, data, profile="default")`: Initializes a new packet with the given attributes.
- `to_dict()`: Converts the packet's attributes into a dictionary.
- `from_dict(packet_dict)`: Creates a packet instance from a dictionary.

## Depends On
- `/pseudo-code/shared/helpers.md`: Provides vector clock utilities for ADTC.
- `/pseudo-code/audit/audit_trail.md`: Logs packet initialization events.
- `/pseudo-code/shared/constants.md`: Defines constants for capability flag bit positions.

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
    capability_flags: u64   // 64-bit capability flags
    headers: PacketHeaders  // Header metadata including vector clock

    METHOD __init__(source, dest, data, profile="default")
        // Generate a unique identifier
        self.id = CALL generate_uuid()
        // Set attributes
        self.source = source
        self.dest = dest
        self.data = data
        self.profile = profile
        // Initialize capability flags as a 64-bit integer
        self.capability_flags = CALL get_capability_flags()  // Returns a 64-bit integer
        // Set specific bits based on node capabilities (positions per 64-bit spec)
        IF CALL node_supports_ml_protocol_selection()
            SET self.capability_flags constants.ML_PROTOCOL_SELECTION_BIT TO 1
        ELSE
            SET self.capability_flags constants.ML_PROTOCOL_SELECTION_BIT TO 0
        END IF
        IF CALL node_supports_ml_failure_prediction()
            SET self.capability_flags constants.ML_FAILURE_PREDICTION_BIT TO 1
        ELSE
            SET self.capability_flags constants.ML_FAILURE_PREDICTION_BIT TO 0
        END IF
        IF CALL node_supports_blockchain_audit()
            SET self.capability_flags constants.BLOCKCHAIN_AUDIT_BIT TO 1
        ELSE
            SET self.capability_flags constants.BLOCKCHAIN_AUDIT_BIT TO 0
        END IF
        IF CALL node_supports_adtc()
            SET self.capability_flags constants.A DTC_BIT TO 1
        ELSE
            SET self.capability_flags constants.A DTC_BIT TO 0
        END IF
        // Initialize headers with vector clock for ADTC
        self.headers = NEW PacketHeaders(profile)
        IF profile == "interplanetary" AND data.type == "ADTC_vote" THEN
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
- Capability Flags: Supports ML (e.g., constants.ML_PROTOCOL_SELECTION_BIT, constants.ML_FAILURE_PREDICTION_BIT), blockchain audit (e.g., constants.BLOCKCHAIN_AUDIT_BIT), and ADTC (e.g., constants.A DTC_BIT). Bits 40–63 are reserved for future features.
- ADTC Support: Includes vector clock in headers for ADTC vote packets, initialized during creation.
- Validation: Recommended for packet attributes to ensure data integrity.  
- 64-bit Transition: capability_flags is now a 64-bit integer. Bit positions are defined in /pseudo-code/shared/constants.md for flexibility and maintainability. Refer to protocol-specs/capability_flags.md for the full specification  


## TODO
- Implement packet attribute validation.
- Add support for extended headers or metadata fields.
- Optimize vector clock storage for low-memory devices.
- Verify bit positions for capability flags with the latest specification.