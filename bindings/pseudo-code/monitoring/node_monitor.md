# Node Monitor Module

## Purpose
The Node Monitor module continuously oversees the health and status of nodes in the MeshGuardian network, detecting issues like failures or degradation and triggering alerts. It supports interplanetary environments with Asynchronous Delay-Tolerant Consensus (ADTC) by handling delayed responses (15-30 minute RTTs), ensuring reliability in dynamic scenarios like crisis zones, IoT systems, or Mars rover networks.

## Interfaces
- **start_monitoring()**  
  - **Purpose**: Initiates continuous monitoring of all network nodes.  
  - **Parameters**: None.  
  - **Behavior**: Loops indefinitely, checking node statuses at regular intervals and triggering alerts for problematic nodes.  

- **get_node_status(node_id)**  
  - **Purpose**: Retrieves the current status of a specified node.  
  - **Parameters**:  
    - `node_id`: Identifier of the node to check.  
  - **Returns**:  
    - `"active"` if the node is functioning normally.  
    - `"degraded"` if the node is operational but underperforming.  
    - `"inactive"` if the node is down.  
    - `"unknown"` if the node cannot be found.  

- **alert_on_failure(node_id)**  
  - **Purpose**: Triggers an alert when a node failure or degradation is detected.  
  - **Parameters**:  
    - `node_id`: Identifier of the problematic node.  
  - **Behavior**: Logs the event and notifies administrators; additional recovery actions can be added.  

## Depends On
- `/pseudo-code/networking/node_registry.md`: Provides the list of nodes to monitor.
- `/pseudo-code/networking/health_checks.md`: Includes `is_node_active` and `is_node_degraded` for health assessment.
- `/pseudo-code/audit_trail.md`: Logs monitoring events and alerts.
- `/pseudo-code/constants.md`: Uses `ADTC_TIMEOUT` for interplanetary checks and updated 64-bit constants.

## Called By
- **Network Management System**: Invokes `start_monitoring` to begin the monitoring process.  
- **Monitoring Dashboard**: May call `get_node_status` for real-time status updates.  

## Used By
- **Network Maintenance Tools**: Relies on this module to detect and respond to node issues.  
- **Automated Recovery Systems**: Uses alerts to initiate failover or recovery procedures.  

## Used In
- **Use Case 5.15: Aid Relays**: Ensures node reliability in crisis zones.
- **Use Case 5.16: Emergency Chat**: Maintains consistent messaging.
- **Use Case 5.1.1: Mars Rover Data Relay**: Monitors nodes in high-latency networks. 

## Pseudocode
```pseudo-code
// Function to start monitoring
FUNCTION start_monitoring()
    WHILE True
        FOR each node IN network_nodes
            status = CALL get_node_status(node.id)
            IF status == "inactive" OR status == "degraded" THEN
                CALL alert_on_failure(node.id)
        WAIT_FOR_INTERVAL(monitor_interval)  // e.g., 5 seconds
    END FUNCTION

// Function to get node status
FUNCTION get_node_status(node_id)
    node = CALL find_node_by_id(node_id)
    IF node IS NULL THEN
        RETURN "unknown"
    END IF
    timeout = node.profile == "interplanetary" ? constants.A DTC_TIMEOUT : DEFAULT_TIMEOUT
    IF CALL is_node_active(node, timeout) THEN
        IF node.capability_flags & BIT(22) THEN  // Updated for 64-bit ML feature flag
            failure_predicted = CALL predict_failure_ml(node.battery, node.uptime, node.rssi_trend)
            IF failure_predicted THEN
                RETURN "degraded"
            END IF
        ELSE
            IF node.battery < constants.ML_FAILURE_BATTERY_THRESHOLD THEN
                RETURN "degraded"
            END IF
        END IF
        RETURN "active"
    ELSE IF CALL is_node_degraded(node, timeout) THEN
        RETURN "degraded"
    ELSE
        RETURN "inactive"
    END IF
END FUNCTION

// Function to alert on failure
FUNCTION alert_on_failure(node_id)
    node = CALL find_node_by_id(node_id)
    details = {
        "ml_enabled": node.capability_flags & BIT(22),  // Updated for 64-bit ML feature flag
        "prediction": node.status == "degraded",
        "profile": node.profile
    }
    IF node.profile == "interplanetary" THEN
        details.vector_clock = node.vector_clock
    END IF
    CALL log_event("FailurePrediction", details, node.capability_flags)
    CALL notify_admins("Node " + node_id + " has failed")
END FUNCTION
```

---

## Notes
- Monitoring Interval: Adjustable via monitor_interval (e.g., 5 seconds for terrestrial, longer for interplanetary).  
- ADTC Support: Uses ADTC_TIMEOUT for interplanetary node checks, handling 15-30 minute RTTs.  
- Scalability: Optimizes status checks for large networks, using event-driven checks where possible.  
- Error Handling: Logs failures as Tier 2 events, Tier 1 for critical interplanetary nodes (Bit 37 = 1 in 64-bit system).  
- Customization: Health check logic (is_node_active, is_node_degraded) tailored to system criteria.  
- 64-Bit Capability Flags: Updated to use 64-bit flags (e.g., BIT(22) for ML-based features, BIT(37) for Tier 1 logging). Refer to protocol-specs/capability_flags.md for details.  


## TODO
- Implement event-driven checks for responsiveness.
- Optimize interplanetary monitoring for sparse networks.
- Add support for node weight metrics (uptime, solar exposure) in alerts.
- Verify new 64-bit flag assignments (e.g., interplanetary mode at Bit 40) with the latest system specification.