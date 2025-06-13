# Trust Graphs Module

## Purpose
The Trust Graphs module is responsible for managing and analyzing trust relationships between nodes in a network. It exists to enable secure and reliable communication by leveraging trust metrics to assess the credibility of nodes, particularly in decentralized or peer-to-peer systems. This module is critical for applications where trust-based decisions are necessary, such as routing in crisis response networks or secure data sharing in IoT ecosystems. It supports advanced features like quantum-ready signatures (Bit 39), multi-blockchain logging (Bit 37), and interplanetary adjustments (Bit 40) through 64-bit capability flags (see /protocol-specs/capability_flags.md).

## Interfaces
- **build_trust_graph(nodes, relationships)**: Constructs a trust graph from a set of nodes and their trust relationships.  
- **calculate_trust_score(node, target, context)**: Computes the trust score from a given node to a target node based on the trust graph, with context for capability flags.  
- **update_trust_relationship(source, target, trust_value, signature, context)**: Updates or adds a trust relationship between two nodes with a specified trust value and signature, verified using quantum-ready algorithms if enabled.  
- **get_trusted_neighbors(node, context)**: Retrieves a list of nodes that are directly trusted by a given node, considering capability flags.

## Depends On
- **/pseudo-code/data/graph.md**: Provides graph data structures and algorithms for managing nodes and edges.  
- **/pseudo-code/crypto/identity.md**: Supplies cryptographic identities and verification for nodes.  
- **/pseudo-code/exceptions/trust_errors.md**: Defines custom exceptions for trust-related errors.  
- **/pseudo-code/helpers.md**: Provides utility functions for RTT estimation and validation utilities.
- **/pseudo-code/constants.md**: Defines thresholds for trust, timeouts, and blockchain priorities.

## Called By
- **/pseudo-code/routing/trust_based_routing.md**: Uses trust scores to make routing decisions in trust-sensitive networks.  
- **/pseudo-code/security/access_control.md**: Leverages trust relationships to enforce access policies in distributed systems.  

## Used In
- **Use Case 5.15: Aid Relays**: Utilizes trust graphs to ensure secure and reliable routing of supply tracking data in crisis zones.  
- **Use Case 5.16: Emergency Chat**: Employs trust relationships to verify the authenticity of messages in real-time communication systems.  
- **Use Case 5.17: Smart City Emergency Response**: Uses trust graphs to validate real-time sensor data and emergency alerts in smart city infrastructure, ensuring secure and reliable consensus during crises.

## Pseudocode
```pseudo-code
// Structure for Trust Context
STRUCT TrustContext
    capability_flags: BITFIELD64 // 64-bit Capability Flags
    timeout: INTEGER             // Adjusted timeout for operations
END STRUCT

// Function to build a trust graph from nodes and relationships
FUNCTION build_trust_graph(nodes, relationships)
    TRY
        graph = NEW Graph()
        FOR each node IN nodes
            ADD_NODE(graph, node)
        FOR each relationship IN relationships
            ADD_EDGE(graph, relationship.source, relationship.target, relationship.trust_value)
        RETURN graph
    CATCH error
        CALL log_event("ERROR", "Failed to build trust graph: " + error, capability_flags | BIT(37)) // Log as Tier 1
        RAISE GraphError("Trust graph construction failed: " + error)
    END TRY

// Function to calculate trust score from node to target
FUNCTION calculate_trust_score(node, target, context: TrustContext)
    TRY
        IF node == target
            RETURN 1.0  // Full trust in self
        path = FIND_SHORTEST_PATH(graph, node, target)
        IF path IS NULL
            RETURN 0.0  // No trust path exists
        trust_score = 1.0
        FOR each edge IN path
            trust_score = trust_score * edge.trust_value
        RETURN trust_score
    CATCH error
        CALL log_event("ERROR", "Failed to calculate trust score: " + error, context.capability_flags | BIT(37)) // Log as Tier 1
        RAISE TrustError("Trust score calculation failed: " + error)
    END TRY

// Function to update or add a trust relationship
FUNCTION update_trust_relationship(source, target, trust_value, signature, context: TrustContext)
    TRY
        IF trust_value < 0 OR trust_value > 1
            RAISE InvalidTrustValueError("Trust value must be between 0 and 1")
        IF NOT NODE_EXISTS(graph, source) OR NOT NODE_EXISTS(graph, target)
            RAISE NodeNotFoundError("Source or target node not found in graph")
        // Verify signature using quantum-ready algorithms if enabled (Bit 39)
        public_key = GET_PUBLIC_KEY(source)
        IF NOT verify_signature(signature, trust_value, public_key, context.capability_flags)
            RAISE SignatureError("Invalid signature for trust update")
        SET_EDGE(graph, source, target, trust_value)
        CALL log_event("INFO", "Trust relationship updated: " + source + " -> " + target + " = " + trust_value, context.capability_flags | BIT(37))
    CATCH error
        CALL log_event("ERROR", "Failed to update trust relationship: " + error, context.capability_flags | BIT(37)) // Log as Tier 1
        RAISE TrustError("Trust relationship update failed: " + error)
    END TRY

// Function to get trusted neighbors of a node
FUNCTION get_trusted_neighbors(node, context: TrustContext)
    TRY
        neighbors = GET_NEIGHBORS(graph, node)
        trusted_neighbors = []
        FOR each neighbor IN neighbors
            IF GET_EDGE_VALUE(graph, node, neighbor) > TRUST_THRESHOLD
                APPEND trusted_neighbors, neighbor
        RETURN trusted_neighbors
    CATCH error
        CALL log_event("ERROR", "Failed to get trusted neighbors: " + error, context.capability_flags | BIT(37)) // Log as Tier 1
        RAISE TrustError("Trusted neighbors retrieval failed: " + error)
    END TRY

// Function to verify a cryptographic signature
FUNCTION verify_signature(signature, data, public_key, capability_flags)
    // Use quantum-ready algorithms if Bit 39 is set (see /protocol-specs/capability_flags.md)
    IF capability_flags & BIT(39) THEN
        RETURN CRYPTO_VERIFY_QUANTUM(signature, data, public_key)
    ELSE
        RETURN CRYPTO_VERIFY(signature, data, public_key)
    END IF
```

---

## Notes
- **Trust Calculation**: The trust score is computed multiplicatively along the shortest path, assuming trust values are between 0 and 1. Alternative methods (e.g., additive trust) could be explored for different trust models.  
- **Performance**: For large graphs, optimize pathfinding (e.g., using Dijkstra's algorithm with priority queues) to ensure efficiency.  
- **Security**: TTrust values are validated to prevent manipulation, with cryptographic signatures (quantum-ready if Bit 39 is set) ensuring the integrity of trust updates.  
- **Edge Cases**: Handles scenarios like isolated nodes, cycles in the graph, or negative trust values (clamped to 0-1).  
- **Interplanetary Support**: When Bit 40 is set, trust calculations and updates account for high-latency environments by adjusting timeouts and buffering trust updates.
- **Audit Trail**: Trust updates and errors are logged to multiple blockchains or a lightweight ledger (Bit 37) for enhanced auditability and resilience.
- **TODO**: 
- Implement trust decay over time to account for aging relationships.  
- Add support for dynamic trust thresholds based on network conditions.
- Optimize graph algorithms for ultra-low-power devices.
- Integrate with MeshGuardian ML Hub for predictive trust modeling.
- Map legacy 32-bit flags to the 64-bit structure or deprecate them.
