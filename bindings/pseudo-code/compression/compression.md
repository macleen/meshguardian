# Compression Module

## Purpose
The Compression module optimizes data transmission in MeshGuardian by reducing packet size, improving bandwidth usage, and minimizing energy consumption. It supports adaptive compression algorithm selection (lz4, zstd, TinyML) based on packet size, signal strength (RSSI), battery health, and 64-bit capability flags, ensuring efficiency in diverse environments, including crisis zones and interplanetary missions. The module is optimized for ultra-low-power devices (e.g., IoT nodes in crisis zones) by prioritizing lightweight algorithms and limiting TinyML inference frequency, with clear hardware requirements to guide deployment. It integrates with the audit trail for logging and supports Low-Energy Mode (Bit 23) for resource-constrained scenarios.

## Interfaces
- **compress_packet(packet, context)**: Compresses a packet based on the provided context, returning the compressed packet.
- **decompress_packet(compressed_packet, algorithm)**: Decompresses a packet using the specified algorithm, returning the original packet.
- **select_compression_algorithm(context)**: Selects the optimal compression algorithm (`lz4`, `zstd`, `TinyML`) based on context parameters.

## Depends On
- **/pseudo-code/helpers.md**: Provides scoring and TinyML model validation utilities.
- **/pseudo-code/logging/logger.md**: Logs compression events and errors.
- **/pseudo-code/constants.md**: Defines compression type codes, thresholds, and TinyML inference intervals.
- **/pseudo-code/audit_trail.md**: Logs compression decisions for traceability.

## Called By
- /pseudo-code/networking/packet_sending.md: Compresses packets before transmission.
- /pseudo-code/networking/packet_receiving.md: Decompresses received packets.

## Used In
- Use Case 5.15: Aid Relays: Compresses supply tracking data in low-bandwidth crisis zones.
- Use Case 5.16: Emergency Chat: Optimizes message transmission for efficiency.
- Use Case 5.1.1: Mars Rover Data Relay: Reduces data size for interplanetary transmission.

## Pseudocode
```pseudocode
// Structure for compression context
STRUCT CompressionContext
    packet_size: INTEGER        // Size of the packet in bytes
    rssi: INTEGER              // Signal strength in dBm
    battery_level: INTEGER     // Battery level in percent
    capability_flags: BITFIELD64 // 64-bit Capability Flags
    device_capability_level: INTEGER // 0 = minimal, 1 = moderate, 2 = high
    packet_count: INTEGER      // Counter for tracking packet frequency
    tinyml_model: MODEL        // TinyML model for compression (if applicable)
END STRUCT

// Function to select compression algorithm
FUNCTION select_compression_algorithm(context: CompressionContext) RETURNS INTEGER
    TRY
        // Check if compression is supported
        IF NOT (context.capability_flags & (BIT(6) | BIT(7) | BIT(9))) THEN
            RETURN constants.NO_COMPRESSION
        END IF

        // Prioritize lightweight algorithm for ultra-low-power devices
        IF context.device_capability_level == 0 OR context.capability_flags & BIT(23) THEN
            IF context.capability_flags & BIT(9) THEN
                CALL log_message("INFO", "Selected lz4 for ultra-low-power device", context.capability_flags)
                RETURN constants.LZ4_CODE
            END IF
            RETURN constants.NO_COMPRESSION
        END IF

        // Limit TinyML inference frequency for low-power devices
        IF context.device_capability_level == 1 AND context.packet_count % constants.TINYML_INFERENCE_INTERVAL != 0 THEN
            IF context.capability_flags & BIT(7) THEN
                CALL log_message("INFO", "Skipped TinyML inference, using zstd level 1", context.capability_flags)
                RETURN constants.ZSTD_CODE
            END IF
            RETURN constants.LZ4_CODE
        END IF

        // Increment packet counter for inference frequency
        context.packet_count = context.packet_count + 1

        // Score compression algorithms
        scores = CALL helpers.score_compression_algorithms(context)
        
        // Validate TinyML model if selected
        IF scores["tinyml"] > scores["lz4"] AND scores["tinyml"] > scores["zstd"] THEN
            start_time = CALL helpers.get_current_time_ms()
            TRY
                CALL helpers.validate_tinyml_model(context.tinyml_model)
                end_time = CALL helpers.get_current_time_ms()
                inference_time = end_time - start_time
                IF inference_time > constants.TINYML_MAX_INFERENCE_TIME THEN
                    CALL log_message("WARNING", "TinyML inference too slow: " + inference_time + "ms, falling back to zstd", context.capability_flags)
                    IF context.capability_flags & BIT(7) THEN
                        RETURN constants.ZSTD_CODE
                    END IF
                    RETURN constants.LZ4_CODE
                END IF
                CALL log_message("INFO", "Selected TinyML compression, inference time: " + inference_time + "ms", context.capability_flags)
                RETURN constants.TINYML_CODE
            CATCH validation_error
                CALL log_message("ERROR", "TinyML model validation failed: " + validation_error, context.capability_flags | BIT(37)) // Log as Tier 1
                IF context.capability_flags & BIT(7) THEN
                    RETURN constants.ZSTD_CODE
                END IF
                RETURN constants.LZ4_CODE
            END TRY
        END IF

        // Select highest-scoring algorithm
        IF scores["lz4"] > scores["zstd"] AND context.capability_flags & BIT(9) THEN
            CALL log_message("INFO", "Selected lz4 compression", context.capability_flags)
            RETURN constants.LZ4_CODE
        END IF
        IF context.capability_flags & BIT(7) THEN
            CALL log_message("INFO", "Selected zstd compression", context.capability_flags)
            RETURN constants.ZSTD_CODE
        END IF
        RETURN constants.NO_COMPRESSION
    CATCH error
        CALL log_message("ERROR", "Compression algorithm selection failed: " + error, context.capability_flags | BIT(37)) // Log as Tier 1
        RETURN constants.NO_COMPRESSION
    END TRY
END FUNCTION

// Function to compress a packet
FUNCTION compress_packet(packet, context)
    TRY
        IF packet IS NULL THEN
            RAISE CompressionError("Packet is null")
        END IF
        algorithm = CALL select_compression_algorithm(context)
        compressed_packet = NULL
        CASE algorithm
            WHEN constants.LZ4_CODE:
                compressed_packet = LZ4_COMPRESS(packet)
            WHEN constants.ZSTD_CODE:
                // Adjust zstd level based on battery health
                IF context.battery_level < constants.BATTERY_LOW THEN
                    zstd_level = 1
                ELSE IF context.battery_level < constants.BATTERY_MODERATE THEN
                    zstd_level = 3
                ELSE
                    zstd_level = 5
                END IF
                compressed_packet = ZSTD_COMPRESS(packet, zstd_level)
            WHEN constants.TINYML_CODE:
                compressed_packet = TINYML_COMPRESS(packet, context.tinyml_model)
            DEFAULT:
                RETURN packet // No compression
        END CASE
        IF compressed_packet IS NULL THEN
            RAISE CompressionError("Compression failed for algorithm: " + algorithm)
        END IF
        CALL log_message("INFO", "Packet compressed with algorithm: " + algorithm + ", original_size: " + packet.size + ", compressed_size: " + compressed_packet.size, context.capability_flags)
        RETURN compressed_packet
    CATCH error
        CALL log_message("ERROR", "Packet compression failed: " + error, context.capability_flags | BIT(37)) // Log as Tier 1
        RAISE error
    END TRY
END FUNCTION

// Function to decompress a packet
FUNCTION decompress_packet(compressed_packet, algorithm)
    TRY
        IF compressed_packet IS NULL THEN
            RAISE CompressionError("Compressed packet is null")
        END IF
        packet = NULL
        CASE algorithm
            WHEN constants.LZ4_CODE:
                packet = LZ4_DECOMPRESS(compressed_packet)
            WHEN constants.ZSTD_CODE:
                packet = ZSTD_DECOMPRESS(compressed_packet)
            WHEN constants.TINYML_CODE:
                packet = TINYML_DECOMPRESS(compressed_packet)
            DEFAULT:
                RAISE CompressionError("Unsupported decompression algorithm: " + algorithm)
        END CASE
        IF packet IS NULL THEN
            RAISE CompressionError("Decompression failed for algorithm: " + algorithm)
        END IF
        CALL log_message("INFO", "Packet decompressed with algorithm: " + algorithm)
        RETURN packet
    CATCH error
        CALL log_message("ERROR", "Packet decompression failed: " + error)
        RAISE error
    END TRY
END FUNCTION

```

---

## Notes
- Adaptive Compression: The module dynamically selects lz4 (fast, low overhead), zstd (high compression ratio), or TinyML (context-aware) based on packet size, RSSI, battery health, and 64-bit capability flags (e.g., Bit 23 for Low-Energy Mode, Bit 36 for extended compression).
- Algorithm Support: Supports compression algorithms as indicated by the 64-bit capability flags (see /protocol-specs/capability_flags.md), with specific bits for lz4 (Bit 9), zstd (Bit 7), TinyML (Bit 6), and extended compression (Bit 36).
- TinyML Optimization: Limits TinyML inference to every 10 packets (TINYML_INFERENCE_INTERVAL) on low-power devices (Device Capability Level 1) to reduce energy consumption. Falls back to zstd or lz4 if inference exceeds 200ms or TinyML is unsupported.
- Energy Efficiency: Supports Low-Energy Mode (Bit 23) by reducing zstd compression levels (1, 3, 5) based on battery health and disabling TinyML for minimal-capability devices.


Hardware Requirements:  

- lz4: Minimum 16 MHz CPU, 32 KB RAM, 100 µW power consumption; suitable for ultra-low-power IoT nodes.
- zstd: Minimum 32 MHz CPU, 64 KB RAM, 200 µW power consumption; suitable for moderate-capability devices.
- TinyML: Minimum 80 MHz CPU, 128 KB RAM, 500 µW power consumption, TensorFlow Lite Micro support; recommended for high-capability devices only.

Performance: Achieves high compression ratios (up to 70% for zstd, 50% for TinyML) with low latency (e.g., lz4 < 1ms for 1KB packets). Optimized for crisis zones with weak signals (RSSI < -90 dBm).

Audit Trail: Compression decisions logged as Tier 2 events; errors as Tier 1 (Bit 37) for traceability.

Edge Cases: Handles unsupported algorithms, low battery, and weak signals by falling back to no compression or lightweight algorithms.

## TODO 
- Implement dynamic adjustment of compression thresholds based on network congestion.

- Add support for new compression algorithms via the Pluggable Protocol Engine.

- Integrate hardware requirements into a configuration file or compatibility matrix for deployment documentation.

- Implement support for extended compression algorithms (e.g., zlib, Brotli) when Bit 36 is set.