# 🖼️ 6. Visual Walkthrough of MeshGuardian

This guide provides a **visual overview** of how data flows through the MeshGuardian architecture—from the moment a message is created, to when it is received, validated, and stored or relayed, leveraging 64-bit capability flags for modularity (see `/protocol-specs/capability_flags.md`).

This is ideal for visual learners or anyone new to how MeshGuardian works.

---

## 🧭 System Overview Diagram

![MeshGuardian Overview Diagram](/assets/system_overview.png)

> 📁 **Note**: Ensure the image is placed in the `/assets/` folder of your GitHub repo and reflects 64-bit capability flag features.

---

## 🧩 Core Flow (Step-by-Step)

1. **User or Sensor Generates Data**  
   - Passed to `PacketCreator.create()`  
   - Profile headers (e.g., Bit 16 for high-priority) and encryption rules applied

2. **Encryption Layer**  
   - Calls `/security/encryption.md`  
   - Uses a derived key from `/security/secure_key_derivation.md`, with post-quantum algorithms (Bit 39)

3. **Routing Decision**  
   - `PacketRouter.route()` chooses next hop  
   - Based on trust graph (if PBFT) or stake (if PoS)

4. **Transmission**  
   - `PacketSender.send()`  
   - Hardware via `/device/hardware_interface.md`, optimized for Low-Energy Mode (Bit 23)

5. **Reception & Validation**  
   - `PacketReceiver.receive()`  
   - Signature and timestamp checked  
   - Calls `PacketValidator.validate()`

6. **Buffering (Optional)**  
   - `PacketBuffer.enqueue()` if node is offline or busy

7. **Decryption + Final Use**  
   - `decrypt_data()` from `/security/encryption.md`, with compressed payloads (Bit 36)  
   - Used by the application or displayed to the user

---

## 🔁 Consensus Path (For Critical Packets)

If the packet is tagged `priority: high` (Bit 16), the following path is activated:

- `ConsensusEngine.run_pbft()`  
- Logs event to `/protocol-specs/audit_trail.md`  
- Writes hash to blockchain (e.g., Solana integration via Bit 37)

See `/protocol-specs/consensus_engine.md` for details.

---

## 📡 Example Use Case: Emergency Chat

```plaintext
Alice types a message →
Encrypted with emergency profile (Bit 39) →
Routed to Bob via 3 mesh hops →
PBFT validates it (Bit 16, quorum of 4) →
Bob receives + decrypts it →
Stored in chat history (encrypted, logged via Bit 37)
```
Module Diagram Reference
Here’s how the pseudo-code folders relate to one another:
```bash
/pseudo-code/
├── networking/      # Packet creation, routing, sending
├── security/        # Encryption, key management
├── consensus/       # PBFT, PoS logic
├── audit/           # Logs and event hashes
├── protocol/        # Profiles, protocol selection
├── shared/          # Helpers and constants
├── device/          # Hardware interface
├── exception/       # System exceptions
├── integration/     # System integration
├── monitoring/      # Activity Monitoring
├── getting_started/ # Starters brief info

```

## Software-Hardware Alignment

Configure 64-bit capability flags in config.py to enable features like high-priority packets (Bit 16), post-quantum cryptography (Bit 39), or audit logging (Bit 37) for optimal operation (see /protocol-specs/capability_flags.md).

## Next Steps

Continue to [7_troubleshooting.md](7_troubleshooting.md) for guidance on debugging common issues, or explore hardware assembly in /hardware/readme.md.

---

Need help? Open a GitHub issue at https://github.com/macleen/meshguardian/issues or join our community on Matrix #meshguardian:matrix.org.