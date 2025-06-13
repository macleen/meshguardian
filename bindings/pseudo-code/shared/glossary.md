# Glossary Module

## Purpose
The Glossary module provides a centralized repository of terms, definitions, and acronyms used throughout the system. It ensures consistent understanding and usage of terminology across documentation, code, and communication, which is crucial for maintaining clarity in complex systems. The module includes terms related to adaptive compression, time-delayed consensus, quantum-ready signatures, multi-blockchain logging, and interplanetary communication, supporting enhanced data efficiency and reliability in diverse environments. It also integrates robust error handling and logging for traceability, aligned with 64-bit capability flags (see /protocol-specs/capability_flags.md)

## Interfaces
- **get_definition(term)**: Retrieve the definition of a specified term from the glossary.
- **add_term(term, definition)**: Add a new term and its definition to the glossary.
- **update_term(term, new_definition)**: Update the definition of an existing term.
- **list_terms()**: Return a list of all terms currently in the glossary.

## Depends On
- **/pseudo-code/shared/helpers.md**: Provides string manipulation and utility functions for term processing.
- **/pseudo-code/logging/logger.md**: Logs errors and events for traceability.

## Called By
- **/pseudo-code/documentation/generator.md**: Uses this module to generate documentation with consistent terminology.
- **/pseudo-code/ui/help_system.md**: Integrates glossary definitions into the help system for user assistance.


## Pseudocode
```pseudocode
// Initialize the glossary as a persistent dictionary with updated terms
PERSISTENT glossary = {
    "Adaptive Compression Selection": "A mechanism to dynamically choose a compression algorithm (e.g., lz4, zstd, TinyML) based on packet size, signal strength, and battery health to optimize bandwidth and energy efficiency.",
    "Compression Type": "A code indicating the compression algorithm used for a packet payload (e.g., 0x0004 for lz4, 0x0002 for zstd, 0x0006 for TinyML, 0x0000 for no compression).",
    "TinyML": "A lightweight machine learning framework used for context-aware compression, optimizing packet payloads based on content patterns in resource-constrained environments.",
    "Asynchronous Delay-Tolerant Consensus": "A consensus mechanism for extreme-delay environments (e.g., 15-30 minute RTTs) that allows asynchronous validation using logical clocks and extended timeout windows, used in the Interplanetary Profile (Bit 40).",
    "Logical Clock": "A mechanism (e.g., Lamport timestamp, vector clock) for ordering events in distributed systems without synchronized real-time clocks, critical for interplanetary consensus.",
    "Time-Delayed Consensus": "A consensus model that extends validation phases to accommodate high-latency environments, ensuring reliability in interplanetary networks.",
    "Quantum-Ready Signature": "A cryptographic signature scheme (e.g., Dilithium, Kyber) designed to resist quantum computer attacks, enabled by Bit 39 in the capability flags.",
    "Multi-Blockchain Logging": "A logging mechanism that submits audit trail events to multiple blockchains (e.g., Solana, Avalanche, Ethereum) for redundancy and resilience, controlled by Bit 37.",
    "Interplanetary Mode": "A system mode that adjusts timeouts and buffering for high-latency environments (e.g., space missions), enabled by Bit 40 in the capability flags."
}

// Function to get the definition of a term
FUNCTION get_definition(term)
    TRY
        IF term IS NULL OR term IS EMPTY THEN
            RAISE ValidationError("Term is null or empty")
        END IF
        normalized_term = CALL helpers.to_lower_case(term)
        IF normalized_term IN glossary THEN
            RETURN glossary[normalized_term]
        ELSE
            RAISE TermNotFoundError("Term '" + term + "' not found in glossary")
        END IF
    CATCH error
        CALL log_message("ERROR", "Failed to get definition for term '" + term + "': " + error, capability_flags | BIT(37))  // Log as Tier 1 with multi-blockchain support
        RAISE error
    END TRY
END FUNCTION

// Function to add a new term to the glossary
FUNCTION add_term(term, definition)
    TRY
        IF term IS NULL OR term IS EMPTY THEN
            RAISE ValidationError("Term is null or empty")
        END IF
        IF definition IS NULL OR definition IS EMPTY THEN
            RAISE ValidationError("Definition is null or empty")
        END IF
        normalized_term = CALL helpers.to_lower_case(term)
        IF normalized_term IN glossary THEN
            RAISE TermAlreadyExistsError("Term '" + term + "' already exists in glossary")
        ELSE
            glossary[normalized_term] = definition
            CALL log_message("INFO", "Added term '" + term + "' to glossary", capability_flags)
        END IF
    CATCH error
        CALL log_message("ERROR", "Failed to add term '" + term + "': " + error, capability_flags | BIT(37))  // Log as Tier 1
        RAISE error
    END TRY
END FUNCTION

// Function to update an existing term's definition
FUNCTION update_term(term, new_definition)
    TRY
        IF term IS NULL OR term IS EMPTY THEN
            RAISE ValidationError("Term is null or empty")
        END IF
        IF new_definition IS NULL OR new_definition IS EMPTY THEN
            RAISE ValidationError("New definition is null or empty")
        END IF
        normalized_term = CALL helpers.to_lower_case(term)
        IF normalized_term NOT IN glossary THEN
            RAISE TermNotFoundError("Term '" + term + "' not found in glossary")
        ELSE
            glossary[normalized_term] = new_definition
            CALL log_message("INFO", "Updated definition for term '" + term + "'", capability_flags)
        END IF
    CATCH error
        CALL log_message("ERROR", "Failed to update term '" + term + "': " + error, capability_flags | BIT(37))  // Log as Tier 1
        RAISE error
    END TRY
END FUNCTION

// Function to list all terms in the glossary
FUNCTION list_terms()
    TRY
        sorted_terms = CALL helpers.sort_list(glossary.keys())
        RETURN sorted_terms
    CATCH sort_error
        CALL log_message("ERROR", "Failed to list glossary terms: " + sort_error, capability_flags | BIT(37))  // Log as Tier 1
        RAISE SortError("Term listing failed: " + sort_error)
    END TRY
END FUNCTION

```

---

## Notes
- Case Sensitivity: Term lookups are case-insensitive using helpers.to_lower_case for flexibility.
- Performance: In-memory storage ensures fast lookups; future persistence requires a storage module.
- Security: Input validation prevents null or empty terms/definitions, with logging to detect misuse.
- Error Handling: Errors are logged as Tier 1 events (Bit 15) via logger.md, ensuring traceability in crisis scenarios.
- New Terms: Added definitions for "Quantum-Ready Signature," "Multi-Blockchain Logging," and "Interplanetary Mode" to reflect expanded features in the 64-bit capability flags.
- Edge Cases: Handles missing terms, duplicates, and invalid inputs with specific errors (ValidationError, TermNotFoundError, TermAlreadyExistsError).

## TODO
- Implement functionality to categorize terms by domain or context (e.g., compression, consensus, networking).
- Add persistent storage once /pseudo-code/storage/data_store.md is available.
- Expand glossary with additional interplanetary terms (e.g., RTT, orbital window) as needed.
- Update terms to reference specific 64-bit capability flags where applicable (e.g., Bit 39 for quantum-ready signatures).
