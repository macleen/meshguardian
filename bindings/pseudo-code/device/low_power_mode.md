# Low-Power Mode Module

## Purpose
The Low-Power Mode module manages the activation and deactivation of Low-E»
nergy Mode (Capability Flag Bit 23) for devices in the MeshGuardian system. Its primary goal is to optimize energy consumption on resource-constrained devices—such as battery-powered sensors, smartphones, or interplanetary relays—by throttling resource-intensive features (e.g., machine learning-driven operations, high-power protocols, and extended compression) when power is low and restoring full functionality when power levels are sufficient. This module is essential for ensuring prolonged operation in energy-scarce environments, such as disaster zones, remote IoT networks, or space missions.

## Interfaces
- **enable_low_power_mode(device_id, battery_level)**: AActivates Low-Energy Mode for a specified device, setting Capability Flag Bit 23 and applying power-saving configurations.
- **disable_low_power_mode(device_id)**: Deactivates Low-Energy Mode, clearing Capability Flag Bit 24 and restoring normal configurations.  
- **check_low_power_mode(device_id)**: Checks if Low-Energy Mode is currently enabled for a device.  
- **monitor_power_status(device_id)**: Monitors battery level or power source to trigger mode transitions.  

## Depends On
- **/pseudo-code/device/hardware_config.md**: Uses `apply_config` to apply power-saving or normal hardware settings during mode transitions.  
- **/pseudo-code/monitoring/node_monitor.md**: Retrieves battery level and power source status for mode decisions.  
- **/pseudo-code/exceptions/hardware_errors.md**: Defines custom exceptions for power-related errors (e.g., `LowPowerError`).  

## Called By
- **/pseudo-code/networking/packet_sending.md**: Checks Low-Energy Mode to adjust protocol selection and transmission settings.  
- **/pseudo-code/networking/packet_receiving.md**: Adjusts reception settings based on Low-Energy Mode status.  
- **/pseudo-code/device/initialization.md**: Initializes devices with appropriate power mode during boot.  

## Used In
- **Use Case 5.15: Aid Relays**: Enables Low-Energy Mode on battery-powered devices in crisis zones to extend operational life for supply tracking.  
- **Use Case 5.16: Emergency Chat**: Manages power modes on smartphones to ensure prolonged real-time communication in disaster scenarios.  
- **Use Case 5.1.1: Mars Rover Data Relay**: Adjusts power settings for interplanetary devices, balancing energy consumption with communication needs.  
-- **Use Case 5.17: Smart City Emergency Response**: Optimizes power usage for real-time data validation in smart city infrastructure.

## Pseudocode
```pseudo-code
// Constants for power thresholds
CONST LOW_BATTERY_THRESHOLD = 20  // Percentage below which Low-Energy Mode is triggered
CONST NORMAL_BATTERY_THRESHOLD = 50  // Percentage above which Low-Energy Mode can be exited

// Function to enable Low-Energy Mode
FUNCTION enable_low_power_mode(device_id, battery_level)
    IF NOT device_exists(device_id)
        RAISE DeviceNotFoundError("Device " + device_id + " not found")
    END IF
    IF battery_level > LOW_BATTERY_THRESHOLD
        RAISE LowPowerError("Battery level " + battery_level + "% is above threshold for Low-Energy Mode")
    END IF
    // Set Capability Flag Bit 23 for Low-Energy Mode
    SET_CAPABILITY_FLAG(device_id, BIT(23), TRUE)
    // Apply power-saving configuration
    config = CALL load_config(device_id)
    config.transmission_power = "low"
    // Disable resource-intensive features
    config.ml_protocol_selection = FALSE  // Disable Bit 22 (ML-based protocol selection)
    config.ml_failure_prediction = FALSE  // Disable Bit 14 (ML failure prediction)
    config.extended_compression = FALSE   // Disable Bit 24 (extended compression)
    config.forward_error_correction = FALSE  // Disable FEC
    CALL apply_config(device_id, config)
    // Log mode activation
    CALL log_event("LowPowerModeEnabled", "Device " + device_id + " entered Low-Energy Mode at " + battery_level + "%", capability_flags)
    RETURN success_status
END FUNCTION

// Function to disable Low-Energy Mode
FUNCTION disable_low_power_mode(device_id)
    IF NOT device_exists(device_id)
        RAISE DeviceNotFoundError("Device " + device_id + " not found")
    END IF
    IF NOT check_low_power_mode(device_id)
        RAISE LowPowerError("Device " + device_id + " is not in Low-Energy Mode")
    END IF
    // Clear Capability Flag Bit 23
    SET_CAPABILITY_FLAG(device_id, BIT(23), FALSE)
    // Restore normal configuration
    config = CALL load_config(device_id)
    config.transmission_power = "normal"
    // Re-enable features if supported by capability flags
    IF capability_flags & BIT(22)
        config.ml_protocol_selection = TRUE
    END IF
    IF capability_flags & BIT(14)
        config.ml_failure_prediction = TRUE
    END IF
    IF capability_flags & BIT(24)
        config.extended_compression = TRUE
    END IF
    config.forward_error_correction = TRUE  // Re-enable FEC if supported
    CALL apply_config(device_id, config)
    // Log mode deactivation
    CALL log_event("LowPowerModeDisabled", "Device " + device_id + " exited Low-Energy Mode", capability_flags)
    RETURN success_status
END FUNCTION

// Function to check Low-Energy Mode status
FUNCTION check_low_power_mode(device_id)
    IF NOT device_exists(device_id)
        RAISE DeviceNotFoundError("Device " + device_id + " not found")
    END IF
    RETURN GET_CAPABILITY_FLAG(device_id, BIT(23))
END FUNCTION

// Function to monitor power status and trigger mode transitions
FUNCTION monitor_power_status(device_id)
    IF NOT device_exists(device_id)
        RAISE DeviceNotFoundError("Device " + device_id + " not found")
    END IF
    battery_level = CALL get_battery_level(device_id)
    IF battery_level <= LOW_BATTERY_THRESHOLD AND NOT check_low_power_mode(device_id)
        CALL enable_low_power_mode(device_id, battery_level)
    ELSE IF battery_level >= NORMAL_BATTERY_THRESHOLD AND check_low_power_mode(device_id)
        CALL disable_low_power_mode(device_id)
    END IF
    RETURN battery_level
END FUNCTION
```

---

## Notes
- 64-Bit Capability Flags: The module now uses 64-bit capability flags, with Low-Energy Mode assigned to Bit 23 (previously Bit 24 in the 32-bit system). Related features like ML-based protocol selection (Bit 22) and extended compression (Bit 24) are toggled based on mode.  
- Feature Throttling: In Low-Energy Mode, resource-intensive features—such as ML-driven operations (Bits 14 and 22), extended compression (Bit 24), and Forward Error Correction (FEC)—are disabled to minimize power usage.  
- Error Handling: Mode transitions and errors are logged via log_event for traceability, with robust checks for invalid device IDs or inappropriate mode states.  
- Interplanetary Support: The module supports interplanetary devices (Bit 40), where efficient power management is critical for extended missions.

## TODO
- Add support for dynamic threshold adjustments based on device type or mission-specific requirements.
- Implement power source detection (e.g., solar, grid) to enhance mode transition logic.
- Map or deprecate legacy 32-bit flags in configuration settings for full 64-bit compatibility.
