# Pseudocode: Packet Creation

## Background
In MeshGuardian, a **packet** is a data unit sent between nodes (e.g., IoT sensors, crisis radios, 4.1.3, 5.15), like a message in a delivery network. Packets carry a unique ID (tracking number), source (sender), destination (receiver), data (content), and headers (instructions, e.g., urgency). The Core Networking module enables resilient communication in disconnected environments (e.g., war zones, 5.15; rural IoT, 5.14) using Delay-Tolerant Networking (DTN, 4.1.5), Trust Graphs (4.1.2) for secure routing, and protocol-agnostic transports (LoRa, Zigbee, 4.1.1). This file defines packet creation, foundational for blockchain logging (5.11), telehealth (5.12), and emergency messaging (5.16), serving as a blueprint for developers.

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

```python
CLASS PacketHeaders
    """
    Encapsulates packet header attributes for consistency and extensibility.
    Used In: 5.11 (blockchain), 5.12 (telehealth), 5.14 (IoT), 5.15 (aid), 5.16 (chat)
    Single Responsibility: Defines and validates header metadata (TTL, profile, priority).
    """
    // Attributes
    ttl: Integer      // Time-to-live in seconds, e.g., 3600
    profile: String   // Communication profile, e.g., "default", "emergency"
    priority: String  // Priority level, e.g., "normal", "urgent"
    
    METHOD __init__(profile)
        """
        Initializes headers with profile-based settings.
        Args:
            profile: String, e.g., "default", "emergency"
        Raises:
            InvalidProfileError: From /pseudo-code/exceptions/networking_errors.md
        Example:
            headers = NEW PacketHeaders("emergency")
            // headers.ttl is 1800, headers.priority is "urgent"
        OOP Notes:
            - For C++ devs: Like struct Headers { int ttl; string profile; }.
        Python Learning:
            - Try imagining: headers = {"ttl": 3600}; print(headers["ttl"]).
        """
        // Validate profile
        valid_profiles = CALL get_valid_profiles()  // From /pseudo-code/protocol/profiles.md
        IF profile NOT IN valid_profiles
            RAISE InvalidProfileError("Profile '{profile}' not recognized")
        
        // Set attributes
        self.profile = profile
        self.ttl = CALL get_ttl(profile)        // e.g., 3600 for default
        self.priority = CALL get_priority(profile)  // e.g., "urgent"
        
        // Log creation
        CALL log_event("Headers_{profile}", "created")  // To /pseudo-code/audit/audit_trail.md

CLASS PacketCreator
    """
    Creates packet attributes with a unique ID and validated headers.
    Used In: 5.11 (blockchain), 5.12 (telehealth), 5.14 (IoT), 5.15 (aid), 5.16 (chat)
    Single Responsibility: Initializes packet data robustly.
    """
    METHOD create(source, dest, data, profile="default")
        """
        Initializes packet attributes with strict validation.
        Args:
            source: Sender node ID (string, e.g., "Node_A")
            dest: Receiver node ID (string, e.g., "Node_B")
            data: Payload (bytes, e.g., b"hello")
            profile: Communication profile (string, default: "default")
        Returns:
            Dictionary with id, source, dest, data, headers
        Raises:
            MissingSourceError, MissingDestError, EmptyDataError, InvalidProfileError,
            OversizedDataError, DuplicateIdError (from /pseudo-code/exceptions/networking_errors.md)
        Example:
            creator = NEW PacketCreator()
            packet_data = CALL creator.create("Node_A", "Node_B", b"hello", "emergency")
            // packet_data: { "id": "123e4567...", "source": "Node_A", ... }
        Example (Error Handling):
            creator = NEW PacketCreator()
            IF CALL creator.create("", "Node_B", b"data", "invalid")
                // Raises MissingSourceError
        OOP Notes:
            - For embedded devs: Like C++ PacketBuilder::build().
        """
        // Validate inputs
        IF NOT source
            RAISE MissingSourceError("Source ID cannot be empty")
        IF NOT dest
            RAISE MissingDestError("Destination ID cannot be empty")
        IF NOT data
            RAISE EmptyDataError("Payload data cannot be empty")
        
        // Validate data size
        max_size = CALL get_max_data_size(profile)  // e.g., 255 bytes for LoRa (4.1.3)
        IF data_size(data) > max_size
            RAISE OversizedDataError("Payload exceeds {max_size} bytes")
        
        // Generate unique ID
        packet_id = CALL generate_uuid()  // Random UUID
        IF is_duplicate_id(packet_id)
            RAISE DuplicateIdError("ID conflicts with existing packet")
        
        // Initialize headers
        headers = NEW PacketHeaders(profile)  // Sets TTL, priority
        
        // Construct attributes
        packet_data = {
            "id": packet_id,      // Unique ID (5.11 logs)
            "source": source,     // Sender (5.15 routing)
            "dest": dest,         // Receiver (5.15 routing)
            "data": data,         // Payload (5.16 chat)
            "headers": headers    // Headers (5.16 priority)
        }
        
        // Log creation
        CALL log_event(packet_id, "created")  // To /pseudo-code/audit/audit_trail.md
        
        RETURN packet_data

CLASS Packet
    """
    Orchestrates the packet lifecycle using specialized classes.
    Used In: 5.11 (blockchain), 5.12 (telehealth), 5.14 (IoT), 5.15 (aid), 5.16 (chat)
    Purpose: Unified interface for creation, sending, routing, buffering, validation, receiving.
    Dependencies: PacketCreator (this file), PacketReceiver (/pseudo-code/networking/packet_receiving.md),
                PacketSender (/pseudo-code/networking/packet_sending.md),
                PacketRouter (/pseudo-code/networking/packet_routing.md),
                PacketBuffer (/pseudo-code/networking/packet_buffering.md),
                PacketValidator (/pseudo-code/networking/packet_validation.md)
    """
    // Attributes
    creator: PacketCreator    // Creates attributes
    receiver: PacketReceiver  // Receives packets
    sender: PacketSender      // Sends packets
    router: PacketRouter      // Routes packets
    buffer: PacketBuffer      // Buffers packets
    validator: PacketValidator // Validates packets
    id: String               // Unique ID
    source: String           // Sender ID
    dest: String             // Receiver ID
    data: Bytes              // Payload
    headers: PacketHeaders   // Metadata
    
    METHOD __init__(source, dest, data, profile="default")
        """
        Initializes a Packet by creating its attributes.
        Args:
            source: Sender node ID (string)
            dest: Receiver node ID (string)
            data: Payload (bytes)
            profile: Communication profile (string, default: "default")
        Raises:
            Errors from /pseudo-code/exceptions/networking_errors.md
        Example:
            packet = NEW Packet("Node_A", "Node_B", b"hello", "emergency")
            // packet.source is "Node_A", packet.headers.priority is "urgent"
        OOP Notes:
            - For C++ devs: Like Packet::Packet(string source).
        """
        // Initialize mini-classes
        self.creator = NEW PacketCreator()
        self.receiver = NEW PacketReceiver()
        self.sender = NEW PacketSender()
        self.router = NEW PacketRouter()
        self.buffer = NEW PacketBuffer()
        self.validator = NEW PacketValidator()
        
        // Create attributes
        packet_data = CALL self.creator.create(source, dest, data, profile)
        
        // Assign attributes
        self.id = packet_data["id"]
        self.source = packet_data["source"]
        self.dest = packet_data["dest"]
        self.data = packet_data["data"]
        self.headers = packet_data["headers"]
    
    METHOD create()
        """
        Returns packet attributes as a dictionary.
        Returns:
            Dictionary with id, source, dest, data, headers
        Example:
            packet = NEW Packet("Node_A", "Node_B", b"hello")
            data = CALL packet.create()
            // data["headers"].priority is "normal"
        OOP Notes:
            - For embedded devs: Like a C++ getter.
        """
        RETURN {
            "id": self.id,
            "source": self.source,
            "dest": self.dest,
            "data": self.data,
            "headers": self.headers
        }

## Notes
- **Single Responsibility**: `PacketCreator` handles creation, `PacketHeaders` manages headers (4.1.4, 4.1.1).  
- **Extensibility**: `Packet` supports new classes (e.g., /pseudo-code/compression/compression.md) for profiles like satellite (4.2.8). `PacketHeaders` allows fields (e.g., encryption_key, 4.1.4).  
- **Reusability**: `PacketCreator.create` suits CLI tools, daemons, SDKs (5.11–5.16).  
- **Error Handling**: Errors in /pseudo-code/exceptions/networking_errors.md guide debugging.  
- **Edge Cases**: Handles empty inputs, invalid profiles, oversized data, duplicate IDs. Future: data encoding (4.1.2).  
- **Dependencies**:  
  - Headers: /pseudo-code/protocol/profiles.md.  
  - Logging: /pseudo-code/audit/audit_trail.md.  
  - Errors: /pseudo-code/exceptions/networking_errors.md, /pseudo-code/exceptions/base_exception.md.  
- **Future Python**: This folder may become a Python package with `__init__.py`, grouping modules.

## Contributor Guide
This file is a pseudocode blueprint for packet creation. Contribute by refining it to guide future implementations. See /pseudo-code/CONTRIBUTING.md for PR guidelines.

1. **Understand MeshGuardian**: Read Background, Use Cases (5.11–5.16).  
2. **Refine Pseudocode**: Enhance `PacketCreator.create` (e.g., add validation)? Extend `PacketHeaders` (e.g., new fields)?  
3. **Validate Design**: Ensure SRP, check edge cases (4.1.3).  
4. **Document**: Update front-matter, add comments.
5. **Submit**: Fork, commit (e.g., “Add validation”), PR to `/pseudo-code/`.