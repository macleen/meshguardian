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
FUNCTION authenticate(user_id, password, timestamp, nonce, ip_address, session_state)
    LOG("Authentication attempt for user " + user_id + " from IP " + ip_address)

    # Check if IP address is locked out
    IF IS_LOCKED_OUT(ip_address)
        LOG("Authentication failed: IP address is locked out")
        RETURN "Authentication failed: IP address is locked out"
    END IF

    # Check timestamp freshness (±30s tolerance with jitter)
    time_diff = abs(timestamp - current_time())
    IF time_diff > 30 + random_jitter(5)
        LOG("Authentication failed: Invalid timestamp")
        RETURN "Authentication failed: Invalid timestamp"
    END IF

    # Reject duplicate nonces
    IF nonce_cache.contains(nonce)
        LOG_SECURITY_EVENT("Replay attempt detected for nonce: " + nonce)
        RETURN "Authentication failed: Nonce reused"
    END IF

    # Verify credentials
    stored_hash = GET_STORED_HASH(user_id)
    provided_hash = HASH(password)

    IF provided_hash != stored_hash
        last_attempt_time = GET_LAST_ATTEMPT_TIME(ip_address)
        IF CURRENT_TIME - last_attempt_time > 1 hour
            RESET_FAILED_ATTEMPTS(ip_address)
        END IF
        INCREMENT_FAILED_ATTEMPTS(ip_address)
        SET_LAST_ATTEMPT_TIME(ip_address, CURRENT_TIME)
        attempts = GET_FAILED_ATTEMPTS(ip_address)
        IF attempts > 5
            # Progressive lockout: exponential backoff up to 24 hours
            lockout_duration = MIN(30 * 2^(attempts - 5), 1440)  # In minutes
            LOCKOUT(ip_address, lockout_duration)
            LOG("Authentication failed: Too many attempts, locked out for " + lockout_duration + " minutes")
            RETURN "Authentication failed: Too many attempts"
        ELSE
            LOG("Authentication failed: Invalid credentials")
            RETURN "Authentication failed: Invalid credentials"
        END IF
    ELSE
        # Successful authentication
        RESET_FAILED_ATTEMPTS(ip_address)
        nonce_cache.add(nonce, ttl=300)  # 5-minute expiration
        LOG("Authentication successful for user " + user_id)
        RETURN "Authentication successful"
    END IF
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

- `LOG_SECURITY_EVENT(message)`: Logs security events like replay attempts for further analysis.

- `random_jitter(max_seconds)`: Generates a random jitter (0 to max_seconds) to handle minor clock skew.

---

## Comparison to Standards

- **TLS 1.3**: Uses nonces and timestamps with tight validation. The enhanced MeshGuardian aligns by adding nonces and a ±30-second window.

- **WireGuard**: Relies on session-based counters. Adding sequence numbers brings MeshGuardian closer to this model.

- **Signal Protocol**: Employs one-time prekeys. While **MeshGuardian**  doesn’t use prekeys, one-time tokens and HMACs achieve similar security.

---

## Notes

- **Replay Attack Protection**: Timestamps with a ±30-second window (plus jitter) and nonce tracking prevent replay attacks by rejecting stale or duplicate messages.

- **IP Lockout Mechanism**: Progressive lockout durations (exponential backoff up to 24 hours) mitigate brute-force attacks effectively.

- **Secure Credential Storage**: Passwords are stored as hashes using a secure algorithm (e.g., bcrypt), avoiding plaintext storage.

- **Rate Limiting**: Tracks failed attempts by IP address, with lockout after 5 failures within a 1-hour window. Resets occur after an hour of inactivity or successful login.

- **HMAC Key Rotation Policy**: HMAC keys are rotated every 24 hours or after 1 million requests to limit exposure and ensure security.

- **Future Improvements**: Will add geo-based exceptions for trusted locations and integrating quantum-resistant signatures.