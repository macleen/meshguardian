# Low-Power Mode Module

## Purpose
The Low-Power Mode module manages the activation and deactivation of Low-Energy Mode (Capability Flag Bit 24) for devices in the MeshGuardian system. It exists to optimize energy consumption on resource-constrained devices, such as battery-powered sensors or smartphones, by throttling resource-intensive features (e.g., ML-driven operations, high-power protocols) when power is low and restoring full functionality when power is sufficient. This module is critical for ensuring prolonged operation in energy-scarce environments like disaster zones or remote IoT networks.

## Interfaces
- **enable_low_power_mode(device_id, battery_level)**: Activates Low-Energy Mode for a device, setting Capability Flag Bit 24 and applying power-saving configurations.  
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
    // Set Capability Flag Bit 24
    SET_CAPABILITY_FLAG(device_id, BIT 24, TRUE)
    // Apply power-saving configuration
    config = CALL load_config(device_id)
    config.transmission_power = "low"
    config.ml_protocol_selection = FALSE  // Disable Bit 13
    config.ml_failure_prediction = FALSE  // Disable Bit 14
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
    // Clear Capability Flag Bit 24
    SET_CAPABILITY_FLAG(device_id, BIT 24, FALSE)
    // Restore normal configuration
    config = CALL load_config(device_id)
    config.transmission_power = "normal"
    config.ml_protocol_selection = TRUE  // Re-enable Bit 13 if supported
    config.ml_failure_prediction = TRUE  // Re-enable Bit 14 if supported
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
    RETURN GET_CAPABILITY_FLAG(device_id, BIT 24)
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
- Power Thresholds: Low-Energy Mode is triggered below 20% battery (LOW_BATTERY_THRESHOLD) and exited above 50% (NORMAL_BATTERY_THRESHOLD) to prevent frequent mode switching and ensure stable operation.
- Feature Throttling: In Low-Energy Mode, disables ML-driven protocol selection (Bit 13), ML failure prediction (Bit 14), and Forward Error Correction (FEC) to reduce power consumption, as specified in Section 2.1.6.
- Performance: Mode transition checks add minimal overhead (<1ms), and configuration changes are applied efficiently via hardware_config.md.
- Reliability: Logs mode transitions as Tier 2 events (via Audit Trail module) for traceability, with error handling for invalid device IDs or mode states.
- Security: Ensures capability flag updates are protected by node authentication (Section 3.2: Capability Negotiation) to prevent unauthorized mode changes.

## TODO
- Add support for dynamic threshold adjustment based on device type or mission requirements.
- Implement power source detection (e.g., solar, grid) to refine mode transitions.
