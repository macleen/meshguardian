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
FUNCTION authenticate(user_id, password, timestamp, ip_address)
    LOG("Authentication attempt for user " + user_id + " from IP " + ip_address)

    IF IS_LOCKED_OUT(ip_address)
        LOG("Authentication failed: IP address is locked out")
        RETURN "Authentication failed: IP address is locked out"
    END IF

    IF timestamp < CURRENT_TIME - 5 minutes - 1 minute OR timestamp > CURRENT_TIME + 1 minute
        LOG("Authentication failed: Invalid timestamp")
        RETURN "Authentication failed: Invalid timestamp"
    END IF

    stored_hash = GET_STORED_HASH(user_id)
    provided_hash = HASH(password)

    IF provided_hash != stored_hash
        last_attempt_time = GET_LAST_ATTEMPT_TIME(ip_address)
        IF CURRENT_TIME - last_attempt_time > 1 hour
            RESET_FAILED_ATTEMPTS(ip_address)
        END IF
        INCREMENT_FAILED_ATTEMPTS(ip_address)
        SET_LAST_ATTEMPT_TIME(ip_address, CURRENT_TIME)
        IF GET_FAILED_ATTEMPTS(ip_address) > 5
            LOCKOUT(ip_address, 30 minutes)
            LOG("Authentication failed: Too many attempts")
            RETURN "Authentication failed: Too many attempts"
        ELSE
            LOG("Authentication failed: Invalid credentials")
            RETURN "Authentication failed: Invalid credentials"
        END IF
    ELSE
        RESET_FAILED_ATTEMPTS(ip_address)
        LOG("Authentication successful for user " + user_id)
        RETURN "Authentication successful"
    END IF
```