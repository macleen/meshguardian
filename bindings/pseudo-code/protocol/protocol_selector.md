# Protocol Selector Module

## Purpose
The Protocol Selector module dynamically selects the most appropriate communication protocol based on network conditions, packet characteristics, predefined profiles, and energy constraints. It ensures optimal performance, reliability, and energy efficiency, prioritizing low-power protocols in Low-Energy Mode (Bit 24) and high-latency protocols (e.g., DTN) for Asynchronous Delay-Tolerant Consensus (ADTC) in the Interplanetary Profile (Bit 28).


## Interfaces
- `select_protocol(packet, context)`: Determines the most suitable protocol for a given packet and context, considering Low-Energy Mode.  
- `get_available_protocols(is_low_energy)`: Returns a list of protocols currently available, filtered for low-power protocols in Low-Energy Mode.  
- `set_protocol_preference(preference_list)`: Allows setting a preference order for protocols.  

## Depends On
- `/pseudo-code/protocol/profiles.md`: Provides profile settings that influence protocol selection.  
- `/pseudo-code/networking/network_conditions.md`: Supplies real-time network condition data.  
- `/pseudo-code/shared/constants.md`: Offers constants used in protocol selection logic.  
- `/pseudo-code/audit_trail.md`: Logs protocol selection events.

## Called By
- `/pseudo-code/networking/packet_sending.md`: Uses this module to select the protocol before sending a packet.  
- `/pseudo-code/networking/packet_receiving.md`: Determines the protocol for incoming packets.  
- `/pseudo-code/application_layer/communication_manager.md`: Manages higher-level communication strategies.  

## Used In
- Use Case 5.15: Aid Relays: Selects low-power protocols like LoRa in Low-Energy Mode.
- Use Case 5.16: Emergency Chat: Prioritizes low-latency protocols.
- Use Case 5.1.1: Mars Rover Data Relay: Selects DTN for ADTC packets in high-latency networks. 


## Pseudocode
```pseudocode
// Function to select the most suitable protocol
FUNCTION select_protocol(packet, context)
    // Validate inputs
    IF packet IS NULL OR context IS NULL THEN
        RAISE ProtocolSelectionError("Invalid packet or context")
    END IF
    profile = packet.get_profile()
    network_conditions = context.get_network_conditions()
    is_low_energy = packet.headers.capability_flags HAS BIT_24
    available_protocols = CALL get_available_protocols(is_low_energy)
    
    // Prioritize DTN for ADTC packets in Interplanetary Profile
    IF packet.headers.capability_flags HAS BIT_28 AND profile = "interplanetary" THEN
        IF "DTN" IN available_protocols THEN
            CALL log_event("ProtocolSelection", {
                "protocol": "DTN",
                "ml_enabled": FALSE
            }, packet.headers.capability_flags)
            RETURN "DTN"
        END IF
    END IF
    
    // Prioritize low-power protocol in Low-Energy Mode
    IF is_low_energy THEN
        IF "LoRa" IN available_protocols THEN
            CALL log_event("ProtocolSelection", {
                "protocol": "LoRa",
                "ml_enabled": FALSE
            }, packet.headers.capability_flags)
            RETURN "LoRa"
        ELSE
            CALL log_event("ProtocolSelection", {
                "protocol": "efficient_protocol",
                "ml_enabled": FALSE
            }, packet.headers.capability_flags)
            RETURN "efficient_protocol"
        END IF
    END IF

    // ML-driven protocol selection (skip in Low-Energy Mode)
    IF packet.headers.capability_flags HAS BIT_13 AND NOT is_low_energy THEN
        model = LOAD_TINYML_MODEL("protocol_selection")
        input_data = [network_conditions.rssi, network_conditions.distance, network_conditions.latency]
        output = model.predict(input_data)
        protocol = PARSE_PROTOCOL(output)
        IF protocol IN available_protocols THEN
            CALL log_event("ProtocolSelection", {
                "protocol": protocol,
                "ml_enabled": TRUE
            }, packet.headers.capability_flags)
            RETURN protocol
        END IF
    END IF

    // Static protocol selection
    IF profile.has_preferred_protocol() THEN
        preferred = profile.get_preferred_protocol()
        IF preferred IN available_protocols THEN
            CALL log_event("ProtocolSelection", {
                "protocol": preferred,
                "ml_enabled": FALSE
            }, packet.headers.capability_flags)
            RETURN preferred
        END IF
    END IF
    IF network_conditions.rssi > -90 THEN
        IF "Bluetooth" IN available_protocols THEN
            CALL log_event("ProtocolSelection", {
                "protocol": "Bluetooth",
                "ml_enabled": FALSE
            }, packet.headers.capability_flags)
            RETURN "Bluetooth"
        END IF
    ELSE IF network_conditions.is_high_latency() THEN
        IF "reliable_protocol" IN available_protocols THEN
            CALL log_event("ProtocolSelection", {
                "protocol": "reliable_protocol",
                "ml_enabled": FALSE
            }, packet.headers.capability_flags)
            RETURN "reliable_protocol"
        END IF
    ELSE IF network_conditions.is_low_bandwidth() THEN
        IF "efficient_protocol" IN available_protocols THEN
            CALL log_event("ProtocolSelection", {
                "protocol": "efficient_protocol",
                "ml_enabled": FALSE
            }, packet.headers.capability_flags)
            RETURN "efficient_protocol"
        END IF
    END IF
    // Fallback to default protocol
    IF "default_protocol" IN available_protocols THEN
        CALL log_event("ProtocolSelection", {
            "protocol": "default_protocol",
            "ml_enabled": FALSE
        }, packet.headers.capability_flags)
        RETURN "default_protocol"
    END IF
    RAISE ProtocolSelectionError("No suitable protocol available")
END FUNCTION

// Function to get available protocols
FUNCTION get_available_protocols(is_low_energy)
    full_protocols = ["Bluetooth", "LoRa", "DTN", "reliable_protocol", "efficient_protocol", "default_protocol"]
    IF is_low_energy THEN
        RETURN ["LoRa", "efficient_protocol", "default_protocol"]
    END IF
    RETURN full_protocols
END FUNCTION

// Function to set protocol preference
FUNCTION set_protocol_preference(preference_list)
    IF preference_list IS EMPTY OR preference_list IS NULL THEN
        RAISE ProtocolSelectionError("Invalid protocol preference list")
    END IF
    SET global_protocol_preference TO preference_list
END FUNCTION
```

---

## Notes
- Selection Factors: Considers network conditions, packet priority, profile settings, and energy constraints.
- ADTC Support: Prioritizes DTN for ADTC packets in the Interplanetary Profile (Bit 28), supporting high-latency networks.
- Low-Energy Mode: Prioritizes low-power protocols (e.g., LoRa) and skips ML-driven selection (Bit 13) when Bit 24 = 1.
- Performance: Protocol selection adds <1ms overhead; ML-driven selection skipped in Low-Energy Mode.
- Reliability: Ensures protocol compatibility, with fallbacks to default protocols.

## TODO
- Add protocol fallback for failed selections.
- Implement weighted scoring for static selection.
- Optimize DTN selection for interplanetary low-bandwidth scenarios.
