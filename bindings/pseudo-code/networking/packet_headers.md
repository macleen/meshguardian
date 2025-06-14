# Pseudo-code: Packet Headers

## Purpose
The PacketHeaders class encapsulates packet header attributes for consistency and extensibility in the MeshGuardian network. It defines and validates metadata such as TTL, profile, priority, capability flags, vector clocks, and consensus mode. This supports features like ML-driven protocol selection, failure prediction, blockchain audit logging, and Asynchronous Delay-Tolerant Consensus (ADTC) for interplanetary environments. The class has been updated to use 64-bit capability flags to accommodate an expanded feature set.


## Interfaces
- init(profile): Initializes headers with profile-based settings.
- to_dict(): Converts headers to a dictionary.
- from_dict(header_dict): Creates headers from a dictionary.

## Depends On
- /pseudo-code/protocol/profiles.md: Validates profiles and retrieves settings.
- /pseudo-code/audit_trail.md: Logs header initialization events.
- /pseudo-code/exceptions/networking_errors.md: Defines InvalidProfileError.
- /pseudo-code/helpers.md: Provides vector clock utilities.
- /pseudo-code/constants.md: Defines ADTC_CONSENSUS_CODE.

## Called By
- /pseudo-code/packet.md: Initializes headers for packets.
- /pseudo-code/networking/packet_creation.md: Uses headers during packet creation.

## Used In
- Use Case 5.11: Blockchain Logging: Supports Solana audit trails.
- Use Case 5.12: Telehealth: Ensures secure patient data packets.
- Use Case 5.14: Rural IoT: Manages sensor data delivery.
- Use Case 5.15: Aid Relays: Tracks supply in crisis zones.
- Use Case 5.16: Emergency Chat: Enables real-time messaging.
- Use Case 5.1.1: Mars Rover Data Relay: Supports ADTC vote packets.

## Pseudocode
```pseudocode
CLASS PacketHeaders
    // Attributes
    ttl: Integer      // Time-to-live in seconds
    profile: String   // Communication profile
    priority: String  // Priority level
    capability_flags: u64  // 64-bit capability flags
    vector_clock: MAP[NodeID, INTEGER]  // Vector clock for ADTC
    consensus_mode: Integer  // Consensus mode (e.g., ADTC_CONSENSUS_CODE)

    METHOD __init__(profile)
        valid_profiles = CALL get_valid_profiles()
        IF profile NOT IN valid_profiles THEN
            RAISE InvalidProfileError("Profile '{profile}' not recognized")
        END IF
        self.profile = profile
        self.ttl = CALL get_ttl(profile)
        self.priority = CALL get_priority(profile)
        // Initialize capability flags as a 64-bit integer
        self.capability_flags = CALL get_capability_flags()  // Returns a 64-bit integer
        // Set specific bits based on node capabilities (updated for 64-bit system)
        IF CALL node_supports_ml_protocol_selection() THEN
            SET self.capability_flags BIT 22 TO 1  // Updated position
        ELSE
            SET self.capability_flags BIT 22 TO 0
        END IF
        IF CALL node_supports_ml_failure_prediction() THEN
            SET self.capability_flags BIT 14 TO 1  // Retained position (example)
        ELSE
            SET self.capability_flags BIT 14 TO 0
        END IF
        IF CALL node_supports_adtc() THEN
            SET self.capability_flags BIT 40 TO 1  // Updated position for interplanetary mode
        ELSE
            SET self.capability_flags BIT 40 TO 0
        END IF
        self.vector_clock = {}
        IF profile == "interplanetary" THEN
            self.vector_clock[CURRENT_NODE_ID] = CALL helpers.increment_lamport_clock(0)
            self.consensus_mode = constants.A DTC_CONSENSUS_CODE
        ELSE
            self.consensus_mode = 0
        END IF
        CALL log_event("Headers_{profile}", {
            "profile": profile,
            "capability_flags": self.capability_flags,
            "vector_clock": self.vector_clock
        }, self.capability_flags)
    END METHOD

    METHOD to_dict()
        RETURN {
            "ttl": self.ttl,
            "profile": self.profile,
            "priority": self.priority,
            "capability_flags": self.capability_flags,
            "vector_clock": self.vector_clock,
            "consensus_mode": self.consensus_mode
        }
    END METHOD

    CLASS METHOD from_dict(header_dict)
        headers = NEW PacketHeaders(header_dict["profile"])
        headers.ttl = header_dict["ttl"]
        headers.priority = header_dict["priority"]
        headers.capability_flags = header_dict["capability_flags"]
        headers.vector_clock = header_dict["vector_clock"]
        headers.consensus_mode = header_dict["consensus_mode"]
        RETURN headers
    END METHOD
END CLASS
```

---

## Notes
- Role: Ensures consistent packet metadata, including vector clocks and consensus mode for ADTC.  
- ADTC Support: Initializes vector clock and sets ADTC_CONSENSUS_CODE for interplanetary profiles.  
- Capability Flags: Supports ML (e.g., Bits 22, 14) and interplanetary mode (e.g., Bit 40). Bits 40–63 are reserved for future features.  
- Logging: Logs header initialization, using Tier 1 (e.g., blockchain) if the corresponding flag (e.g., Bit 37) is set.  
- 64-Bit Transition: Updated to use 64-bit capability flags. Refer to protocol-specs/capability_flags.md for exact bit assignments.  


## TODO
- Add validation for header fields.
- Optimize vector clock storage for low-memory devices.
- Support dynamic updates to consensus mode.
- Confirm exact bit positions with the latest MeshGuardian specification.