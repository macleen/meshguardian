# Profiles Module

## Purpose
The Profiles module is responsible for defining and managing communication profiles that dictate how packets are handled within the system. It allows the configuration of settings such as priority, time-to-live (TTL), and other parameters to optimize packet processing for specific scenarios, such as urgent disaster messaging or routine data transfers. This module exists to provide flexibility and control, ensuring the system can adapt to diverse operational needs efficiently.

## Interfaces
- `get_profile_settings(profile_name)`: Retrieves the settings (e.g., TTL, priority) for a given profile.
- `add_profile(profile_name, settings)`: Adds a new profile with specified settings.
- `update_profile(profile_name, new_settings)`: Modifies the settings of an existing profile.
- `remove_profile(profile_name)`: Removes a specified profile from the system.

## Depends On
- `/pseudo-code/shared/constants.md`: Supplies constants like `DEFAULT_TTL` and `DEFAULT_PRIORITY` for default settings.
- `/pseudo-code/audit/audit_trail.md`: Logs profile modifications for auditing purposes.

## Called By
- `/pseudo-code/networking/packet_creation.md`: Fetches profile settings to configure packets.
- `/pseudo-code/networking/packet_headers.md`: Applies profile settings to packet headers.

## Used In
- **5.15**: Multi-hop supply tracking in crisis zones, where profiles prioritize urgent packets.
- **5.16**: Real-time disaster messaging, where profiles ensure low-latency delivery.

## Pseudocode
\`\`\`pseudocode
FUNCTION get_profile_settings(profile_name)
    IF profile_name NOT IN profiles
        RAISE ProfileNotFoundError("Profile '" + profile_name + "' does not exist")
    ELSE
        RETURN profiles[profile_name]

FUNCTION add_profile(profile_name, settings)
    IF profile_name IN profiles
        RAISE ProfileAlreadyExistsError("Profile '" + profile_name + "' already exists")
    ELSE
        // Ensure ML settings are included
        IF NOT "ml_protocol_selection" IN settings
            settings.ml_protocol_selection = FALSE
        END IF
        IF NOT "ml_failure_prediction" IN settings
            settings.ml_failure_prediction = FALSE
        END IF
        SET profiles[profile_name] TO settings
        CALL log_event("Profile added: " + profile_name, {"settings": settings})

FUNCTION update_profile(profile_name, new_settings)
    IF profile_name NOT IN profiles
        RAISE ProfileNotFoundError("Profile '" + profile_name + "' does not exist")
    ELSE
        // Ensure ML settings are included
        IF NOT "ml_protocol_selection" IN new_settings
            new_settings.ml_protocol_selection = profiles[profile_name].ml_protocol_selection
        END IF
        IF NOT "ml_failure_prediction" IN new_settings
            new_settings.ml_failure_prediction = profiles[profile_name].ml_failure_prediction
        END IF
        SET profiles[profile_name] TO new_settings
        CALL log_event("Profile updated: " + profile_name, {"settings": new_settings})

FUNCTION remove_profile(profile_name)
    IF profile_name NOT IN profiles
        RAISE ProfileNotFoundError("Profile '" + profile_name + "' does not exist")
    ELSE
        REMOVE profiles[profile_name]
        CALL log_event("Profile removed: " + profile_name)
\`\`\`

---

## Notes
- Profiles are stored as key-value pairs with the profile name as the key and settings (e.g., `{"ttl": 3600, "priority": "normal"}`) as the value.
- Error handling ensures robustness by checking for missing or duplicate profiles.
- Consider optimizing for scalability as the number of profiles grows.
- Profile settings should be validated to prevent invalid configurations.
- **TODO**: Implement TTL range checks and other validation rules.