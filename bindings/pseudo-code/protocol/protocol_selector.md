# Protocol Selector Module

## Purpose
The Protocol Selector module is responsible for dynamically selecting the most appropriate communication protocol based on the current network conditions, packet characteristics, and predefined profiles. This ensures optimal performance, reliability, and efficiency in various scenarios, such as emergency messaging or routine data transfers.

## Interfaces
- `select_protocol(packet, context)`: Determines the most suitable protocol for a given packet and context.
- `get_available_protocols()`: Returns a list of protocols currently available in the system.
- `set_protocol_preference(preference_list)`: Allows setting a preference order for protocols.

## Depends On
- `/pseudo-code/protocol/profiles.md`: Provides profile settings that influence protocol selection.
- `/pseudo-code/networking/network_conditions.md`: Supplies real-time network condition data.
- `/pseudo-code/shared/constants.md`: Offers constants used in protocol selection logic.

## Called By
- `/pseudo-code/networking/packet_sending.md`: Uses this module to select the protocol before sending a packet.
- `/pseudo-code/networking/packet_receiving.md`: Determines the protocol for incoming packets.
- `/pseudo-code/application_layer/communication_manager.md`: Manages higher-level communication strategies.

## Used In
- **Use Case 5.15**: Multi-hop supply tracking in crisis zones, where different protocols are needed based on network reliability.
- **Use Case 5.16**: Real-time disaster messaging, where low-latency protocols are preferred.

## Pseudocode
```pseudocode
FUNCTION select_protocol(packet, context)
    profile = packet.get_profile()
    network_conditions = context.get_network_conditions()
    available_protocols = get_available_protocols()

    IF packet.headers.capability_flags BIT 13
        // ML-driven protocol selection
        model = LOAD_TINYML_MODEL("protocol_selection")
        input_data = [network_conditions.rssi, network_conditions.distance, network_conditions.latency]
        output = model.predict(input_data)
        protocol = PARSE_PROTOCOL(output)
        IF protocol IN available_protocols
            RETURN protocol
    END IF

    // Static protocol selection
    IF profile.has_preferred_protocol()
        preferred = profile.get_preferred_protocol()
        IF preferred IN available_protocols
            RETURN preferred
    END IF
    IF network_conditions.rssi > -90
        RETURN "Bluetooth"
    ELSE IF network_conditions.is_high_latency()
        RETURN "reliable_protocol"
    ELSE IF network_conditions.is_low_bandwidth()
        RETURN "efficient_protocol"
    ELSE
        RETURN "default_protocol"

FUNCTION get_available_protocols()
    // Return list of supported protocols
    RETURN ["Bluetooth", "LoRa", "reliable_protocol", "efficient_protocol", "default_protocol"]

FUNCTION set_protocol_preference(preference_list)
    // Store preference order for static selection
    SET global_protocol_preference TO preference_list
```

---

## Notes
- Protocol selection can be based on various factors including network conditions, packet priority, and profile settings.
- The current implementation uses a simple if-else logic; consider using a more sophisticated algorithm for better optimization.
- Ensure that the selected protocol is always available and compatible with the current system state.
- **TODO**: Add support for protocol fallback in case the selected protocol fails.
