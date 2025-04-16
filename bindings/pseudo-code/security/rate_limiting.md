# Rate Limiting Module

## Purpose
The Rate Limiting module is designed to prevent abuse and protect against brute-force attacks by limiting the number of authentication attempts from a single source within a specified time frame. It exists to enhance the security of the authentication process, ensuring that attackers cannot make unlimited login attempts, which could compromise system integrity.

## Interfaces
- **Function: `check_rate_limit(source)`**  
  Checks if the rate limit for the given source has been exceeded.  
- **Function: `record_attempt(source)`**  
  Records an authentication attempt for the given source.  
- **Function: `reset_attempts(source)`**  
  Resets the attempt counter for the given source.  
- **Function: `configure_rate_limit(max_attempts, time_window)`**  
  Configures the rate limiting parameters, such as the maximum number of attempts and the time window.  

## Depends On
- **/pseudo-code/storage/rate_limit_storage.md**: Manages the storage and retrieval of attempt counts and timestamps.  
- **/pseudo-code/config/config_manager.md**: Loads configuration settings for rate limiting parameters.  

## Called By
- **/pseudo-code/security/auth.md**: Invokes `check_rate_limit` before processing a login attempt and `record_attempt` after an attempt.  

## Used In
- **Use Case 5.15: Aid Relays**: Protects authentication for accessing supply tracking systems in crisis zones.  
- **Use Case 5.16: Emergency Chat**: Secures real-time messaging authentication in disaster scenarios.  

## Pseudocode
The following pseudo-code outlines the core logic for tracking and limiting authentication attempts.
```pseudocode
// Function: check_rate_limit
FUNCTION check_rate_limit(source)
    attempts, last_attempt_time = GET_ATTEMPT_DATA(source)
    IF CURRENT_TIME - last_attempt_time > time_window THEN
        RESET_ATTEMPTS(source)
        RETURN False  // Limit not exceeded
    END IF
    IF attempts >= max_attempts * WARNING_THRESHOLD THEN
        LOG_POTENTIAL_ATTACK(source)
    END IF
    IF attempts >= max_attempts THEN
        RETURN True  // Limit exceeded
    ELSE
        RETURN False  // Limit not exceeded
    END IF

// Function: record_attempt
FUNCTION record_attempt(source, success)
    attempts, last_attempt_time = GET_ATTEMPT_DATA(source)
    IF CURRENT_TIME - last_attempt_time > time_window THEN
        SET_ATTEMPTS(source, 1)
    ELSE
        IF NOT success THEN
            SET_ATTEMPTS(source, attempts + 1)
        ELSE
            RESET_ATTEMPTS(source)  // Reward successful auth
        END IF
    END IF
    SET_LAST_ATTEMPT_TIME(source, CURRENT_TIME)
    ENCRYPT_AND_STORE(source, {attempts, last_attempt_time})

// Function: reset_attempts
FUNCTION reset_attempts(source)
    SET_ATTEMPTS(source, 0)
    SET_LAST_ATTEMPT_TIME(source, NULL)
    ENCRYPT_AND_STORE(source, {0, NULL})

// Function: configure_rate_limit(max_attempts, time_window)
FUNCTION configure_rate_limit(max_attempts, time_window)
    IF is_new_device(source) THEN
        SET_MAX_ATTEMPTS(LOWER_LIMIT)
    ELSE
        SET_MAX_ATTEMPTS(max_attempts)
    END IF
    SET_TIME_WINDOW(time_window)
    IF max_attempts IS NULL THEN
        SET_MAX_ATTEMPTS(5)  // Fail-secure default
    END IF

// Helper functions
FUNCTION GET_ATTEMPT_DATA(source)
    IF CACHE_HAS(source) THEN
        RETURN CACHE_GET(source)
    ELSE
        data = DECRYPT_STORAGE(STORAGE_GET(source))
        CACHE_SET(source, data)
        RETURN data
    END IF

FUNCTION SET_ATTEMPTS(source, attempts)
    ENCRYPT_AND_STORE(source, attempts)

FUNCTION SET_LAST_ATTEMPT_TIME(source, time)
    ENCRYPT_AND_STORE(source, {GET_ATTEMPTS(source), time})

FUNCTION ENCRYPT_AND_STORE(source, data)
    encrypted_data = ENCRYPT(data)
    STORE_IN_RATE_LIMIT_STORAGE(source, encrypted_data)

FUNCTION is_new_device(source)
    // Check if source is a new device based on historical data or risk signals
    RETURN IS_NEW_DEVICE(source)
```

---

## Notes
- **Granularity**: The `source` can be a user ID, IP address, or a combination, depending on the desired rate limiting strategy.  
- **Storage**: Uses encrypted storage to protect attempt data from tampering, with a cache layer for performance. 
- **Performance**: Caching optimizes frequent reads, minimizing latency in high-volume systems.
- **Security**: Protects against time manipulation with monotonic clocks (implied) and storage tampering with encryption.
- **Adaptive Limiting**: Adjusts max_attempts for new devices or based on risk signals from /pseudo-code/security/risk_assessment.md.



**TODO**: Integrate machine learning for anomaly detection and WAF/CDN integration for distributed protection.
