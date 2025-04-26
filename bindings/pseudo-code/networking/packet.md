# Packet Module

## Purpose
The `Packet` class represents a data packet in the networking system, encapsulating essential attributes like a unique identifier, source, destination, data payload, communication profile, and capability flags. It provides methods for initializing packets, converting them to dictionaries, and creating instances from dictionaries. This class is crucial for managing packet structure and operations consistently across the system, supporting features like ML-driven protocol selection (Bit 13), failure prediction (Bit 14), and blockchain audit logging (Bit 15).

## Interfaces
- `__init__(source, dest, data, profile="default")`: Initializes a new packet with the given attributes.
- `to_dict()`: Converts the packet's attributes into a dictionary.
- `from_dict(packet_dict)`: Creates a packet instance from a dictionary.

## Depends On
- `/pseudo-code/shared/utils.md`: Assumes the existence of utility functions like `generate_uuid()` for generating unique identifiers.

## Called By
- `/pseudo-code/networking/packet_sending.md`: Uses `Packet` to create and send packets.
- `/pseudo-code/networking/packet_receiving.md`: Handles incoming packets.
- `/pseudo-code/networking/data_transmission.md`: Manages packet transmission.

## Used In
- **Use Case 5.15**: Multi-hop supply tracking in crisis zones.
- **Use Case 5.16**: Real-time disaster messaging.

## Pseudocode
```pseudocode
CLASS Packet
    """
    Represents a data packet with a unique ID, source, destination, data, profile, and capability flags.
    Single Responsibility: Manages packet structure and basic operations.
    """
    // Attributes
    id: String              // Unique packet identifier
    source: String          // Sender node ID
    dest: String            // Receiver node ID
    data: Bytes             // Payload data
    profile: String         // Communication profile (e.g., "default")
    capability_flags: Integer  // 32-bit capability flags

    METHOD __init__(source, dest, data, profile="default")
        """
        Initializes a Packet with source, destination, data, profile, and capability flags.
        Args:
            source: Sender node ID (string)
            dest: Receiver node ID (string)
            data: Payload (bytes)
            profile: Communication profile (string, default: "default")
        Example:
            packet = NEW Packet("Node_A", "Node_B", b"hello", "default")
        """
        // Generate a unique identifier for the packet
        self.id = CALL generate_uuid()

        // Set attributes
        self.source = source
        self.dest = dest
        self.data = data
        self.profile = profile
        // Initialize capability flags
        self.capability_flags = CALL get_capability_flags()
        IF CALL node_supports_ml_protocol_selection()
            SET self.capability_flags BIT 13 TO 1
        ELSE
            SET self.capability_flags BIT 13 TO 0
        END IF
        IF CALL node_supports_ml_failure_prediction()
            SET self.capability_flags BIT 14 TO 1
        ELSE
            SET self.capability_flags BIT 14 TO 0
        END IF
        IF CALL node_supports_blockchain_audit()
            SET self.capability_flags BIT 15 TO 1  // Enable Solana logging
        ELSE
            SET self.capability_flags BIT 15 TO 0  // Local logging
        END IF
        SET self.capability_flags BIT 16 TO 0  // Reserved for Lightweight Blockchain
        CALL log_event("PacketInitialized", {"id": self.id, "capability_flags": self.capability_flags}, self.capability_flags)
    END METHOD

    METHOD to_dict()
        """
        Converts the packet attributes into a dictionary.
        Returns:
            Dictionary containing id, source, dest, data, profile, and capability_flags
        """
        RETURN {
            "id": self.id,
            "source": self.source,
            "dest": self.dest,
            "data": self.data,
            "profile": self.profile,
            "capability_flags": self.capability_flags
        }
    END METHOD

    CLASS METHOD from_dict(packet_dict)
        """
        Creates a Packet instance from a dictionary.
        Args:
            packet_dict: Dictionary with packet attributes
        Returns:
            Packet instance
        """
        packet = NEW Packet(packet_dict["source"], packet_dict["dest"], packet_dict["data"], packet_dict["profile"])
        packet.id = packet_dict["id"]
        packet.capability_flags = packet_dict["capability_flags"]
        RETURN packet
    END METHOD
END CLASS
```  

---

## Notes
- The Packet class is designed to be simple and focused on managing packet data, with capability flags supporting ML-driven features (Bits 13 and 14) and blockchain audit logging (Bit 15).
- Bit 15 (Blockchain Audit): Enables Solana-based logging for critical events; defaults to local logging for compatibility.
- Bit 16 (Lightweight Blockchain): Reserved for future lightweight blockchain integration.
- Future enhancements could include adding headers or additional metadata.
- Validation for packet attributes is recommended to ensure data integrity.

## TODO
- Implement packet attribute validation.
- Add support for extended headers or metadata fields.
