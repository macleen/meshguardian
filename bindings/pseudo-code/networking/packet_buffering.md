# Pseudocode: Packet Buffering

## Purpose
The Packet Buffering module temporarily stores data packets in the MeshGuardian network to manage flow, prevent loss during congestion, and ensure reliable delivery in Delay-Tolerant Networks (DTNs). It supports buffering of Asynchronous Delay-Tolerant Consensus (ADTC) votes in high-latency interplanetary environments (15-30 minute RTTs), using extended timeouts and priority queuing for critical packets, ensuring stability in fluctuating network conditions like crisis zones or interplanetary relays.

## Interfaces
- **enqueue(packet, timeout)**: Adds a packet to the buffer with a specified timeout.
- **dequeue()**: Removes and returns the next packet from the buffer, prioritizing high-priority packets.
- **is_full()**: Returns True if the buffer is at capacity, False otherwise.
- **is_empty()**: Returns True if the buffer is empty, False otherwise.
- **size()**: Returns the current number of packets in the buffer.
- **flush()**: Removes all packets from the buffer.

## Depends On
This module relies on the following components:
- **`/pseudo-code/networking/packet_creation.md`**: Uses the `PacketCreator` class to generate structured packets for buffering.
- **`/pseudo-code/exceptions/networking_errors.md`**: Depends on error classes like `BufferOverflowError` to handle cases where the buffer cannot accept more packets.
- **`/pseudo-code/shared/constants.md`**: Uses predefined constants, such as `MAX_BUFFER_SIZE`, to set the buffer’s capacity limits.
- **`/pseudo-code/audit/audit_trail.md`**: Logs buffering events.

## Called By
The packet buffering module is utilized by:
- **`/pseudo-code/networking/packet_sending.md`**: Buffers outgoing packets before they are transmitted over the network.
- **`/pseudo-code/networking/packet_receiving.md`**: Stores incoming packets before they are processed or validated.
- **`/pseudo-code/networking/packet_routing.md`**: Holds packets temporarily during routing decisions in complex network paths.
- **`/pseudo-code/data_transmission.md`**: Buffers ADTC votes during high-latency periods.

## Used In
This module is referenced in the following use cases:
- **Use Case 5.15**: Aid relays – Buffers packets to prevent loss during unreliable network conditions in emergency zones.
- **Use Case 5.16**: Emergency chat – Manages packet flow to ensure real-time communication during high-traffic disaster scenarios.
- **Use Case 5.1.1: Mars Rover Data Relay**: Buffers ADTC votes for reliable consensus.

## Pseudocode
```pseudocode
CLASS PacketBuffer
    METHOD initialize(max_size)
        buffer = NEW PriorityQueue()  // Prioritize by packet priority
        self.max_size = max_size
        self.timeouts = NEW Dictionary()  // Map packet ID to timeout

    METHOD enqueue(packet, timeout=constants.DEFAULT_TIMEOUT)
        IF size() >= self.max_size THEN
            RAISE BufferOverflowError("Cannot add packet: buffer is full")
        END IF
        priority = packet.profile == "emergency" ? "high" : (packet.data.type == "ADTC_vote" ? "medium" : "low")
        ADD packet TO buffer WITH priority
        self.timeouts[packet.id] = timeout
        CALL log_event("PacketBuffered", {
            "id": packet.id,
            "priority": priority,
            "timeout": timeout
        }, packet.capability_flags)
    END METHOD

    METHOD dequeue()
        IF is_empty() THEN
            RETURN NULL
        END IF
        packet = REMOVE highest priority packet FROM buffer
        IF packet.timeouts[packet.id] < CURRENT_TIME THEN
            CALL log_event("PacketDiscarded", {
                "id": packet.id,
                "reason": "Timeout expired"
            }, packet.capability_flags)
            RETURN NULL
        END IF
        REMOVE packet.id FROM self.timeouts
        CALL log_event("PacketDequeued", {
            "id": packet.id
        }, packet.capability_flags)
        RETURN packet
    END METHOD

    METHOD is_full()
        RETURN size() >= self.max_size
    END METHOD

    METHOD is_empty()
        RETURN size() == 0
    END METHOD

    METHOD size()
        RETURN length of buffer
    END METHOD

    METHOD flush()
        CLEAR buffer
        CLEAR self.timeouts
        CALL log_event("BufferFlushed", {}, capability_flags)
    END METHOD
```

---

## Notes
- Edge Cases: Handles buffer overflow with BufferOverflowError, returns NULL for dequeue on empty or expired packets.  
- Performance: Uses a priority queue for O(log n) enqueue/dequeue, optimized for high-priority (emergency) and medium-priority (ADTC votes) packets.  
- ADTC Support: Buffers ADTC votes with ADTC_TIMEOUT (30 minutes), ensuring delivery during communication windows.  
- Thread Safety: Requires locks for concurrent access in multi-threaded environments.  
- Audit Logging: Logs buffering, dequeuing, and discarding events, using Tier 1 for ADTC votes if Bit 37 = 1.  
- 64-Bit Capability Flags: Updated to use 64-bit flags (e.g., BIT(37) for Tier 1 logging). Refer to protocol-specs/capability_flags.md for details.

## TODO
- Add priority-based queuing for critical packets.
- Implement timeout mechanisms to discard stale packets.
- Optimize buffer for low-bandwidth interplanetary networks.