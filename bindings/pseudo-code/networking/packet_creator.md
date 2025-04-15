# Pseudo-code: Packet Creator

## Class: PacketCreator

### Method: create(source, dest, data, profile="default")

**Purpose**:  
Constructs a packet with a unique identifier, source, destination, data, profile, and timestamp.

**Parameters**:  
- `source`: The sender’s identifier.  
- `dest`: The recipient’s identifier.  
- `data`: The payload of the packet.  
- `profile`: The packet’s profile (defaults to "default").

**Returns**:  
A structured packet with the following fields:  
- `"id"`: Unique identifier for the packet.  
- `"source"`: Sender’s identifier.  
- `"dest"`: Recipient’s identifier.  
- `"data"`: Payload.  
- `"profile"`: Profile setting.  
- `"timestamp"`: Creation time.

**Pseudo-code**:  
```pseudo-code
CLASS PacketCreator
    METHOD create(source, dest, data, profile="default")
        packet_id = GENERATE_UNIQUE_ID()
        timestamp = GET_CURRENT_TIMESTAMP()
        packet = STRUCTURE {
            "id": packet_id,
            "source": source,
            "dest": dest,
            "data": data,
            "profile": profile,
            "timestamp": timestamp
        }
        RETURN packet