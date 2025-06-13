# Encryption Module

## Purpose
The Encryption module is designed to secure data by providing encryption and decryption capabilities. It exists to protect sensitive information from unauthorized access, tampering, or interception, ensuring confidentiality, integrity, and authenticity. This is essential for applications like secure messaging, data storage, or communication in untrusted environments, with robust error handling for invalid keys to support post-quantum and conventional cryptography. The module has been updated to support a 64-bit `capability_flags` structure; refer to `protocol-specs/capability_flags.md` for flag definitions.

## Interfaces
- **`encrypt_data(data, key, context, packet)`**: Encrypts the input data using the provided key, with protocol-specific algorithms (e.g., AES-GCM, Kyber).  
- **`decrypt_data(encrypted_data, key, context, packet)`**: Decrypts encrypted data using the specified key, handling invalid key errors.  
- **`generate_key()`**: Creates a new encryption key.  
- **`load_key(key_name)`**: Retrieves an existing key by its name or identifier.  

## Depends On
- **`/pseudo-code/security/key_management.md`**: Handles the generation, storage, and retrieval of encryption keys.  
- **`/pseudo-code/shared/crypto_library.md`**: Supplies cryptographic algorithms and utilities for encryption and decryption.  
- **`/pseudo-code/logging/logger.md`**: Logs encryption errors and events for traceability.  
- **`/pseudo-code/shared/constants.md`**: Defines constants for capability flag bit positions.

## Called By
- **`/pseudo-code/networking/data_transfer.md`**: Secures data before it is transmitted over the network.  
- **`/pseudo-code/storage/data_storage.md`**: Encrypts data prior to storage to safeguard it from unauthorized access.  

## Used In
- **Use Case 5.15**: Secure communication in crisis zones, protecting data during multi-hop transfers with robust key error handling.  
- **Use Case 5.16**: Safeguarding sensitive configuration files from unauthorized access or modification, ensuring decryption reliability.  

## Pseudocode
```pseudocode
// Function to validate key format and integrity
FUNCTION validate_key(key, protocol)
    IF key IS NULL OR key IS EMPTY THEN
        RAISE KeyError("Key is null or empty")
    END IF
    IF protocol == "LoRa" THEN
        // Kyber key requirements (post-quantum)
        IF NOT IS_VALID_KYBER_KEY(key) THEN
            RAISE KeyError("Invalid Kyber key format for LoRa protocol")
        END IF
    ELSE
        // AES-GCM key requirements (e.g., 256-bit)
        IF LENGTH(key) != 32 THEN
            RAISE KeyError("Invalid AES-GCM key length: expected 32 bytes")
        END IF
    END IF
    RETURN TRUE
END FUNCTION

// Function to encrypt data
FUNCTION encrypt_data(data, master_key, context, packet)
    IF data IS NULL OR data IS EMPTY THEN
        RAISE EncryptionError("Data is null or empty")
    END IF
    // Select protocol based on capability flags (64-bit integer)
    IF packet.headers.capability_flags & constants.ML_PROTOCOL_SELECTION_BIT THEN
        protocol = CALL select_protocol_ml(context.network_metrics)
    ELSE
        protocol = CALL select_protocol_static(context.network_metrics)
    END IF
    // Derive and validate session key
    session_key = CALL derive_key(master_key, context)
    CALL validate_key(session_key, protocol)
    nonce = CALL generate_nonce()
    TRY
        IF protocol == "Bluetooth" THEN
            encrypted_data, tag = AES_GCM_ENCRYPT(session_key, nonce, data)
        ELSE IF protocol == "LoRa" THEN
            encrypted_data, tag = KYBER_ENCRYPT(session_key, nonce, data)
        ELSE
            encrypted_data, tag = AES_GCM_ENCRYPT(session_key, nonce, data)  // Default
        END IF
        RETURN nonce + encrypted_data + tag
    CATCH crypto_error
        CALL log_message("ERROR", "Encryption failed: " + crypto_error + " for protocol: " + protocol, packet.headers.capability_flags | constants.BLOCKCHAIN_AUDIT_BIT)  // Log as Tier 1
        RAISE EncryptionError("Encryption failed: " + crypto_error)
    END TRY
END FUNCTION

// Function to decrypt data
FUNCTION decrypt_data(encrypted_data, master_key, context, packet)
    IF encrypted_data IS NULL OR encrypted_data IS EMPTY THEN
        RAISE EncryptionError("Encrypted data is null or empty")
    END IF
    // Select protocol based on capability flags (64-bit integer)
    IF packet.headers.capability_flags & constants.ML_PROTOCOL_SELECTION_BIT THEN
        protocol = CALL select_protocol_ml(context.network_metrics)
    ELSE
        protocol = CALL select_protocol_static(context.network_metrics)
    END IF
    // Derive and validate session key
    session_key = CALL derive_key(master_key, context)
    CALL validate_key(session_key, protocol)
    nonce = EXTRACT_NONCE(encrypted_data)
    ciphertext = EXTRACT_CIPHERTEXT(encrypted_data)
    tag = EXTRACT_TAG(encrypted_data)
    TRY
        IF protocol == "Bluetooth" THEN
            data = AES_GCM_DECRYPT(session_key, nonce, ciphertext, tag)
        ELSE IF protocol == "LoRa" THEN
            data = KYBER_DECRYPT(session_key, nonce, ciphertext, tag)
        ELSE
            data = AES_GCM_DECRYPT(session_key, nonce, ciphertext, tag)  // Default
        END IF
        RETURN data
    CATCH crypto_error
        CALL log_message("ERROR", "Decryption failed: " + crypto_error + " for protocol: " + protocol, packet.headers.capability_flags | constants.BLOCKCHAIN_AUDIT_BIT)  // Log as Tier 1
        RAISE EncryptionError("Decryption failed: " + crypto_error)
    END TRY
END FUNCTION

// Function to generate a key
FUNCTION generate_key()
    RETURN CALL key_management.generate_key()
END FUNCTION

// Function to load a key
FUNCTION load_key(key_name)
    key = CALL key_management.load_key(key_name)
    IF key IS NULL THEN
        RAISE KeyError("No key found for name: " + key_name)
    END IF
    RETURN key
END FUNCTION

// Function to derive a key
FUNCTION derive_key(master_key, context)
    IF master_key IS NULL OR master_key IS EMPTY THEN
        RAISE KeyError("Master key is null or empty")
    END IF
    session_key = HKDF(master_key, context)
    IF session_key IS NULL OR session_key IS EMPTY THEN
        RAISE KeyError("Failed to derive session key")
    END IF
    RETURN session_key
END FUNCTION

// Function to generate a nonce
FUNCTION generate_nonce()
    RETURN SECURE_RANDOM(12)
END FUNCTION

```

---

## Notes
- Key Security: Encryption keys are stored securely via /pseudo-code/security/key_management.md, ideally using a hardware security module (HSM). Invalid key errors are logged as Tier 1 events for traceability.  
- Performance Considerations: Encryption/decryption latency is optimized for AES-GCM (~1µs/KB) and Kyber (~10ms for key encapsulation). Key validation adds minimal overhead (<1ms).  
- Algorithm Support: Supports AES-GCM for Bluetooth and default protocols, and Kyber for LoRa (post-quantum). Future enhancements could add ChaCha20 or Dilithium signatures.  
- Error Handling: Validates keys for format and length, raising KeyError for invalid keys and EncryptionError for crypto failures, with audit trail logging via logger.md.  
- 64-bit Capability Flags: The capability_flags field is now a 64-bit integer. Bit positions are defined in /pseudo-code/shared/constants.md for flexibility and maintainability. Refer to protocol-specs/capability_flags.md for the full specification.  
- Future Improvements: Implement key rotation via /pseudo-code/security/key_management.md to refresh keys periodically. Add support for hardware-accelerated cryptography (e.g., via HSMs).  

