# Packet Module

## Purpose
The `Packet` class represents a data packet in the networking system, encapsulating essential attributes like a unique identifier, source, destination, data payload, and communication profile. It provides methods for initializing packets, converting them to dictionaries, and creating instances from dictionaries. This class is crucial for managing packet structure and operations consistently across the system.

## Interfaces
- `__init__(source, dest, data, profile="default")`: Initializes a new packet with the given attributes.
- `to_dict()`: Converts the packet's attributes into a dictionary.
- `from_dict(packet_dict)`: Creates a packet instance from a dictionary.

## Depends On
- `/pseudo-code/shared/utils.md`: Assumes the existence of utility functions like `generate_uuid()` for generating unique identifiers.

## Called By
- `/pseudo-code/networking/packet_sending.md`: Uses `Packet` to create and send packets.
- `/pseudo-code/networking/packet_receiving.md`: Handles incoming packets.
- `/pseudo-code/networking/packet_routing.md`: Manages packet routing decisions.

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
        IF node_supports_ml_protocol_selection()
            SET self.capability_flags BIT 13 TO 1
        ELSE
            SET self.capability_flags BIT 13 TO 0
        END IF
        IF node_supports_ml_failure_prediction()
            SET self.capability_flags BIT 14 TO 1
        ELSE
            SET self.capability_flags BIT 14 TO 0
        END IF
        CALL log_event("PacketInitialized", {"id": self.id, "capability_flags": self.capability_flags})

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
```  

---

## Notes
- The Packet class is designed to be simple and focused on managing packet data.
- Future enhancements could include adding headers or additional metadata.
- Consider implementing validation for packet attributes to ensure data integrity.
