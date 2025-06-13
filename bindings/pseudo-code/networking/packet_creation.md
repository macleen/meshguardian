# Pseudocode: Packet Creation

## Background
In MeshGuardian, a **packet** is a data unit sent between nodes (e.g., IoT sensors, crisis radios, 4.1.3, 5.15), like a message in a delivery network. Packets carry a unique ID (tracking number), source (sender), destination (receiver), data (content), and headers (instructions, e.g., urgency). The Core Networking module enables resilient communication in disconnected environments (e.g., war zones, 5.15; rural IoT, 5.14) using Delay-Tolerant Networking (DTN, 4.1.5), Trust Graphs (4.1.2) for secure routing, and protocol-agnostic transports (LoRa, Zigbee, 4.1.1). This file defines packet creation, foundational for blockchain logging (5.11), telehealth (5.12), and emergency messaging (5.16), serving as a blueprint for developers.

**Note**: The MeshGuardian system has transitioned to a 64-bit capability flag architecture, expanding the feature set for future protocol enhancements. Contributors should refer to `protocol-specs/capability_flags.md` for updated bit positions and new flags.

## Purpose
Defines the `PacketCreator` class for initializing packet attributes, the `Packet.create` method for orchestrating creation, and the `PacketHeaders` class for header metadata. Ensures robust packet formation, supporting MeshGuardian’s crisis and IoT goals.

## Conventions
- `CLASS Name`: Defines a class, like PHP’s `class` or JS’s `class`.  
- `METHOD name(args)`: Defines a method.  
- `NEW Class`: Creates an instance, like PHP’s `new` or JS’s `new`.  
- `CALL function`: Invokes a method, like PHP’s `$this->method()` or JS’s `obj.method()`.  
- `IF condition`: Conditional logic, like PHP’s `if` or JS’s `if`.  
- `RAISE Error`: Throws an exception, like PHP’s `throw` or JS’s `throw`.  
- `// Comment`: Explains logic, like PHP/JS comments.  
- `Dictionary`: Key-value structure, like PHP’s array (`['key' => 'value']`) or JS object (`{ key: 'value' }`).

## Interfaces
- `PacketCreator.create(source, dest, data, profile)`
- `PacketHeaders.__init__(profile)`
- `Packet.create(source, dest, data, profile)`

## Depends On
- `/pseudo-code/exceptions/networking_errors.md` (errors like MissingSourceError)  
- `/pseudo-code/exceptions/base_exception.md` (parent class for errors, via networking_errors.md)  
- `/pseudo-code/protocol/profiles.md` (valid profiles)  
- `/pseudo-code/audit/audit_trail.md` (logs creation events)  

## Called By
- `Packet` class (orchestrates lifecycle via `PacketSender`, `PacketRouter`, etc.)
- `NetworkManager` class (manages network tasks, e.g., process_packet)

## Used In
- Use Case 5.11: Blockchain logging (audit data to Solana, 4.1.6)
- Use Case 5.12: Telehealth (secure patient data packets, 5.12)
- Use Case 5.14: Rural IoT networks (sensor data delivery, 4.1.3)
- Use Case 5.15: Aid relays (multi-hop supply tracking in crises, 5.15)
- Use Case 5.16: Emergency chat (real-time disaster messaging, 5.16)

## Pseudocode
```pseudocode
CLASS PacketHeaders
    // Encapsulates packet header attributes for consistency and extensibility.
    // Defines and validates header metadata (TTL, profile, priority, capability_flags).
    METHOD __init__(profile)
        valid_profiles = CALL get_valid_profiles()
        IF profile NOT IN valid_profiles
            RAISE InvalidProfileError("Profile '{profile}' not recognized")

        self.profile = profile
        self.ttl = CALL get_ttl(profile)
        self.priority = CALL get_priority(profile)
        // Initialize capability flags as a 64-bit integer based on node configuration
        self.capability_flags = CALL get_capability_flags()  // Returns a 64-bit integer
        // Set specific bits based on node capabilities (positions updated for 64-bit system)
        IF node_supports_ml_protocol_selection()
            SET self.capability_flags BIT 22 TO 1  // Updated to Bit 22 (example position)
        ELSE
            SET self.capability_flags BIT 22 TO 0
        END IF
        IF node_supports_ml_failure_prediction()
            SET self.capability_flags BIT 14 TO 1  // Bit 14 retained (example position)
        ELSE
            SET self.capability_flags BIT 14 TO 0
        END IF
        CALL log_event("Headers_{profile}", {"profile": profile, "capability_flags": self.capability_flags})

CLASS PacketCreator
    // Creates packet attributes with a unique ID and validated headers.
    METHOD create(source, dest, data, profile="default")
        IF NOT source
            RAISE MissingSourceError("Source ID cannot be empty")
        IF NOT dest
            RAISE MissingDestError("Destination ID cannot be empty")
        IF NOT data
            RAISE EmptyDataError("Payload data cannot be empty")

        max_size = CALL get_max_data_size(profile)
        IF data_size(data) > max_size
            RAISE OversizedDataError("Payload exceeds {max_size} bytes")

        packet_id = CALL generate_uuid()
        IF is_duplicate_id(packet_id)
            RAISE DuplicateIdError("ID conflicts with existing packet")

        headers = NEW PacketHeaders(profile)

        packet_data = {
            "id": packet_id,
            "source": source,
            "dest": dest,
            "data": data,
            "headers": headers
        }

        CALL log_event(packet_id, {"event": "created", "capability_flags": headers.capability_flags})
        RETURN packet_data

CLASS Packet
    METHOD __init__(source, dest, data, profile="default")
        self.creator = NEW PacketCreator()
        self.receiver = NEW PacketReceiver()
        self.sender = NEW PacketSender()
        self.router = NEW PacketRouter()
        self.buffer = NEW PacketBuffer()
        self.validator = NEW PacketValidator()

        packet_data = CALL self.creator.create(source, dest, data, profile)

        self.id = packet_data["id"]
        self.source = packet_data["source"]
        self.dest = packet_data["dest"]
        self.data = packet_data["data"]
        self.headers = packet_data["headers"]

    METHOD create()
        RETURN {
            "id": self.id,
            "source": self.source,
            "dest": self.dest,
            "data": self.data,
            "headers": self.headers
        }
```  

---

## Notes
- Single Responsibility: PacketCreator handles creation, PacketHeaders manages headers (4.1.4, 4.1.1), ensuring focused functionality.
- Extensibility: Packet supports new classes (e.g., /pseudo-code/compression/compression.md) for profiles like satellite (4.2.8). PacketHeaders allows fields (e.g., encryption_key, 4.1.4).
- Reusability: PacketCreator.create suits CLI tools, daemons, SDKs (5.11–5.16), promoting code reuse across contexts.
- Error Handling: Comprehensive errors in /pseudo-code/exceptions/networking_errors.md guide debugging for edge cases like invalid inputs.
- Edge Cases: Handles empty inputs, invalid profiles, oversized data, duplicate IDs. Future enhancements could include data encoding (4.1.2).
- Dependencies:
  - Headers rely on /pseudo-code/protocol/profiles.md for validation.
  - Logging integrates with /pseudo-code/audit/audit_trail.md.
  - Errors link to /pseudo-code/exceptions/networking_errors.md and /pseudo-code/exceptions/base_exception.md.
- Future Considerations: This folder may evolve into a Python package with __init__.py, grouping related modules for better organization.  
- 64-Bit Capability Flags: Updated to use 64-bit integers for capability_flags. Bit positions (e.g., Bit 22 for ML protocol selection) are examples and must be verified with protocol-specs/capability_flags.md.  

## Contributor Guide
- Understand MeshGuardian: Read Background, Use Cases (5.11–5.16).
- Refine Pseudocode: Enhance PacketCreator.create (e.g., add validation)? Extend PacketHeaders (e.g., new fields)?
- Validate Design: Ensure SRP, check edge cases (4.1.3).
- Document: Update front-matter, add comments.
- Submit: Fork, commit (e.g., “Add validation”), PR to /pseudo-code/.
