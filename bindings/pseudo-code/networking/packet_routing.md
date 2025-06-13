# Packet Routing Module

## Purpose
The Routing module calculates the optimal path for data packets from a source node to a destination node in a network. It ensures efficient and reliable packet delivery by selecting routes based on factors like shortest distance, fault tolerance, and network metrics, crucial for dynamic network systems like crisis response or IoT applications. The module prevents routing loops using destination checks, Hop Count, and TTL validation, ensuring robust operation in unstable networks.

## NB: 
Capability flags controlling routing features are defined as a 64-bit integer, with specific bit positions managed via a constants file (e.g., /pseudo-code/shared/constants.md) for flexibility across system updates  

## Interfaces
- **select_route(source, destination, packet)**  
  - Purpose: Finds the best route from source to destination, preventing loops with destination and TTL checks.  
  - Parameters:  
    - source: Starting node.  
    - destination: Target node.  
    - packet: Packet object containing headers (e.g., capability_flags as a 64-bit integer, hop_count, ttl)  
  - Returns:  
    - "Local delivery" if source and destination are the same.  
    - List of nodes representing the route if one exists.  
  - Raises:  
    - RoutingError: If no route is available or TTL is exceeded.  
  - Note: capability_flags is a 64-bit integer, with bit positions defined in /pseudo-code/shared/constants.md.  

  
- **find_shortest_path(source, destination)**  
  - Purpose: Computes the shortest path using Dijkstra's algorithm.  
  - Parameters:  
    - source: Starting node.  
    - destination: Target node.  
  - Returns:  
    - List of nodes forming the shortest path.  
    - NULL if no path exists.  

- **find_ml_route(source, destination, network_metrics)**  
  - Purpose: Computes a route using a TinyML model, ensuring loop-free paths.  
  - Parameters:  
    - source: Starting node.  
    - destination: Target node.  
    - network_metrics: Metrics like latency, bandwidth, node health.  
  - Returns:  
    - List of nodes forming the ML-driven route.  
    - NULL if no valid route exists.  

- **update_route_table(new_routes)**  
  - Purpose: Updates the routing table with new route information.  
  - Parameters:  
    - new_routes: New route data.  

- **handle_routing_error(error_type)**  
  - Purpose: Processes routing-related errors.  
  - Parameters:  
    - error_type: Type of error encountered (e.g., "NoRoute", "TTLExceeded", "CyclicRoute").  

## Depends On
- Network Topology: Provides current node and connection data (e.g., /pseudo-code/networking/topology.md).  
- Routing Exceptions: Defines errors like RoutingError (e.g., /pseudo-code/exceptions/routing_errors.md).  
- Utility Functions: Helper functions like distance(node1, node2) (e.g., /pseudo-code/shared/utils.md).  

## Called By
- Packet Sending Module: Uses select_route to determine paths (e.g., /pseudo-code/networking/packet_sending.md).  
- Network Management Module: Calls update_route_table to refresh routes (e.g., /pseudo-code/networking/network_management.md).  
- Communication Manager: Relies on handle_routing_error for error handling (e.g., /pseudo-code/application_layer/communication_manager.md).  

## Used By
- Packet Sending Module: Depends on this for routing decisions.  
- Network Management Module: Uses it to maintain route consistency.  
- Communication Manager: Integrates it for robust error management.  

## Used In
- Crisis Zone Supply Tracking: Ensures multi-hop routing in unstable networks, preventing loops for reliable aid delivery.  
- Real-Time Disaster Messaging: Guarantees fast, loop-free message delivery in dynamic environments.  

## Pseudocode
```pseudo-code
// Function to detect cycles in a route
FUNCTION detect_cycle(route)
    seen_nodes = SET()
    FOR node IN route
        IF node IN seen_nodes
            RETURN TRUE  // Cycle detected
        ADD node TO seen_nodes
    END FOR
    RETURN FALSE  // No cycle

// Function to select the best route
FUNCTION select_route(source, destination, packet)
    // Check if source is the destination
    IF source == destination
        RETURN "Local delivery"
    END IF
    // Check Hop Count against TTL to prevent loops
    IF packet.headers.hop_count >= packet.headers.ttl
        RAISE RoutingError("TTL exceeded, potential loop detected")
    END IF
    // Select route based on Capability Flags
    IF packet.headers.capability_flags BIT 13
        // ML-driven routing
        route = CALL find_ml_route(source, destination, packet.network_metrics)
    ELSE
        // Static routing (shortest path)
        route = CALL find_shortest_path(source, destination)
    END IF
    // Validate route
    IF route IS NOT NULL
        // Filter out nodes predicted to fail if Bit 14 is enabled
        IF packet.headers.capability_flags BIT 14
            filtered_route = []
            FOR each node IN route
                IF NOT CALL predict_failure_ml(node.battery, node.uptime, node.rssi_trend)
                    APPEND filtered_route, node
                END IF
            IF filtered_route IS NOT EMPTY
                // Increment Hop Count for next hop
                packet.headers.hop_count = packet.headers.hop_count + 1
                RETURN filtered_route
            END IF
        ELSE
            // Increment Hop Count for next hop
            packet.headers.hop_count = packet.headers.hop_count + 1
            RETURN route
        END IF
    RAISE RoutingError("No route found")
END FUNCTION

// Function to find the shortest path using Dijkstra's algorithm
FUNCTION find_shortest_path(source, destination)
    SET distances TO dictionary with all nodes set to infinity
    SET distances[source] TO 0
    SET previous TO dictionary with all nodes set to null
    SET unvisited TO set of all nodes

    WHILE unvisited IS NOT EMPTY
        current = node in unvisited with smallest distance
        IF current == destination
            BREAK
        REMOVE current from unvisited
        FOR each neighbor of current
            IF neighbor in unvisited
                tentative_distance = distances[current] + distance(current, neighbor)
                IF tentative_distance < distances[neighbor]
                    SET distances[neighbor] TO tentative_distance
                    SET previous[neighbor] TO current
                END IF
    END WHILE

    path = []
    current = destination
    WHILE current IS NOT NULL
        INSERT current at beginning of path
        current = previous[current]

    IF path[0] == source
        RETURN path
    ELSE
        RETURN NULL
END FUNCTION

// Function to find an ML-driven route
FUNCTION find_ml_route(source, destination, network_metrics)
    // Load TinyML model for routing
    model = LOAD_TINYML_MODEL("routing")
    input_data = [network_metrics.latency, network_metrics.bandwidth, network_metrics.node_health]
    output = model.predict(input_data)
    route = PARSE_ROUTE(output)
    // Validate route to prevent cycles
    IF route IS NOT NULL AND NOT CALL detect_cycle(route)
        RETURN route
    RAISE RoutingError("Cyclic or invalid ML-driven route detected")
END FUNCTION

// Function to update the routing table
FUNCTION update_route_table(new_routes)
    // Logic to update routing table with new_routes
    PASS
END FUNCTION

// Function to handle routing errors
FUNCTION handle_routing_error(error_type)
    // Log error and trigger recovery actions
    IF error_type == "TTLExceeded"
        CALL log_event("RoutingError", "TTL exceeded, packet dropped", capability_flags)
    ELSE IF error_type == "CyclicRoute"
        CALL log_event("RoutingError", "Cyclic route detected, recomputing", capability_flags)
    ELSE
        CALL log_event("RoutingError", "No route found: " + error_type, capability_flags)
    END IF
    // Trigger route table update or fallback routing
    PASS
END FUNCTION

```

---

## Notes
- Algorithm: find_shortest_path uses Dijkstra's algorithm with O(V^2) time complexity (V = number of nodes). A priority queue could improve this to O(E + V log V), where E is the number of edges.
- Loop Prevention: Routing loops are prevented by:
    - Checking if the source is the destination in select_route, returning "Local delivery" to stop forwarding.
    - Validating Hop Count against TTL in select_route, dropping packets if exceeded to avoid infinite loops.
    - Detecting cycles in ML-driven routes via detect_cycle in find_ml_route, ensuring loop-free paths.

- Error Handling: select_route raises RoutingError for invalid routes, TTL exceedance, or cyclic routes. handle_routing_error logs errors and triggers recovery (e.g., route recomputation).
- Placeholders: update_route_table and parts of handle_routing_error are stubs and need system-specific implementations. 
- Customization: Adjust distance(node1, node2) based on network metrics (e.g., latency, bandwidth). ML-driven routing (Bit 13) uses TinyML models for dynamic route selection.
- Performance: Destination and TTL checks add minimal overhead (<1ms per packet). Cycle detection in find_ml_route is O(n) for n nodes in the route.
- Reliability: Each node calls select_route to re-evaluate the route, ensuring destination checks at every hop, critical for dynamic networks like crisis zones.
