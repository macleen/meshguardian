# Authentication Logic with Replay Attack Protection

This document outlines the authentication logic implemented in MeshGuardian, specifically focusing on replay attack protection, timestamp validation, IP-based lockout, and secure hashing.

---

## Function: `authenticate(user_id, password, timestamp, ip_address)`

Performs the full authentication procedure including:

- Credential validation
- Timestamp freshness verification
- IP-based brute-force protection

### Logic

```pseudocode
FUNCTION verify_message(message, nonce_cache, session_state)
    # Check timestamp freshness (±30s tolerance)
    IF abs(message.timestamp - current_time()) > 30
        RETURN False
    END IF
    
    # Verify timestamp signature
    IF NOT verify_timestamp_signature(message.timestamp, message.signature)
        RETURN False
    END IF
    
    # Reject duplicate nonces
    IF nonce_cache.contains(message.nonce)
        RETURN False
    END IF
    
    # Check sequence number
    IF message.sequence_number <= session_state.last_sequence
        RETURN False
    END IF
    
    # Verify HMAC with session key
    IF NOT verify_hmac(message, session_state.key)
        RETURN False
    END IF
    
    # Update state
    nonce_cache.add(message.nonce, ttl=300)  # 5-minute expiration
    session_state.last_sequence = message.sequence_number
    RETURN True
```

## Helper Functions

- `GET_STORED_HASH(user_id)`: Retrieves the stored hash for the user from secure storage.

- `HASH(password)`: Hashes the password using a secure algorithm (e.g., bcrypt).

- `IS_LOCKED_OUT(ip_address)`: Checks if the IP address is currently locked out.

- `LOCKOUT(ip_address, duration)`: Locks out the IP address for a specified duration.

- `INCREMENT_FAILED_ATTEMPTS(ip_address)`: Increments the failed attempt counter for the IP address.

- `GET_FAILED_ATTEMPTS(ip_address)`: Retrieves the number of failed attempts for the IP address.

- `RESET_FAILED_ATTEMPTS(ip_address)`: Resets the failed attempt counter for the IP address.

- `GET_LAST_ATTEMPT_TIME(ip_address)`: Retrieves the timestamp of the last failed attempt for the IP address.

- `SET_LAST_ATTEMPT_TIME(ip_address, time)`: Sets the timestamp of the last failed attempt for the IP address.

- `LOG(message)`: Logs the message for auditing and monitoring.

---

## Comparison to Standards

- **TLS 1.3**: Uses nonces and timestamps with tight validation. The enhanced MeshGuardian aligns by adding nonces and a ±30-second window.

- **WireGuard**: Relies on session-based counters. Adding sequence numbers brings MeshGuardian closer to this model.

- **Signal Protocol**: Employs one-time prekeys. While **MeshGuardian**  doesn’t use prekeys, one-time tokens and HMACs achieve similar security.

---

## Notes

- **Replay Attack Protection**: The inclusion of a timestamp ensures that authentication requests are time-sensitive, preventing replay attacks by rejecting requests outside a defined window.

- **IP Lockout Mechanism**: Protects against brute-force attacks by locking out IP addresses after a certain number of failed attempts.

- **Secure Credential Storage**: Passwords are stored as hashes using a secure algorithm, and the provided password is hashed and compared to the stored hash, avoiding plaintext storage.

- **Rate Limiting**: Failed attempts are tracked by IP address, with a lockout triggered after 5 failures within a 1-hour window, lasting 30 minutes. The counter resets after an hour of inactivity or a successful login.

- **Future Improvements**: Add nonces or session tokens for enhanced security.
