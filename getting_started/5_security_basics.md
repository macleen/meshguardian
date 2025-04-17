# 5. Security Basics in MeshGuardian

MeshGuardian is built from the ground up to support **secure communication** in hostile or disconnected environments. This document introduces the foundational security principles and modules in the system.

Whether you’re deploying in a rural IoT network or a disaster zone, these basics ensure **confidentiality, integrity, and trust**.

---

## Core Security Modules

MeshGuardian separates security into dedicated submodules:

| Module                          | Role                                  |
|----------------------------------|----------------------------------------|
| `/security/encryption.md`       | Handles encryption/decryption of packets |
| `/security/key_management.md`   | Manages cryptographic keys lifecycle    |
| `/security/privacy_safe_pod.md` | Protects identity and location metadata |
| `/security/zkp.md`              | Verifies data ownership without revealing content |

> Security in MeshGuardian is not optional—it’s deeply integrated with networking, device, and protocol layers.

---

## Encryption Workflow (at a Glance)

Before sending:
1. The payload (`packet.data`) is **encrypted** with a key retrieved from the key management module.
2. Headers may also be encrypted in "confidential" profiles (e.g., `"profile": "telehealth"`).

On receive:
1. The system checks **packet authenticity**.
2. If valid, it **decrypts the data** using the correct key.

Module references:
- `/security/encryption.md`
- `/security/key_management.md`

---

## Key Generation and Storage

- Keys are generated using `generate_key()` with secure parameters.
- Keys are stored via `store_key()` in secure vaults or Hardware Security Modules (HSMs).
- You can rotate or revoke keys via:
  ```python
  rotate_key(key_id, new_parameters)
  revoke_key(key_id)
  ```

  🔄 Regular key rotation is encouraged for forward secrecy and trust management.

Authentication & Nonce Protection  
To prevent replay attacks, MeshGuardian uses:
- Timestamp validation
- Optional nonce
- Rate-limiting + IP lockout on repeated failures
- See /security/auth.md for details.
- Privacy Safe Pods (Optional)
- For high-privacy deployments:
    - Nodes are placed into privacy-safe enclaves
    - Identity, geolocation, and timing metadata are obfuscated
    - Helpful in civil society or protest use cases
    - See /security/privacy_safe_pod.md

**Tips for Developers**
- Use derive_key() from /security/secure_key_derivation.md instead of storing passphrases
- Validate all inputs and hash passwords with bcrypt or argon2
- Never store private keys in plaintext—even during debugging
- Enable encryption_required=true in packet headers for sensitive profiles
- Summary  


**Concept Implemented Through**
- Encryption	/security/encryption.md  
- Key Management	/security/key_management.md  
- Replay Protection	/security/auth.md  
- Privacy Zones	/security/privacy_safe_pod.md  
- Advanced Crypto	/security/zkp.md, /security/key_derivation.md  
- Next: → 6_hardware_basics.md (or /hardware/readme.md if you’re diving into device assembly)

