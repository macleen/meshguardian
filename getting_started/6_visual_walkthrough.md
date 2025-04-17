# 🖼️ 6. Visual Walkthrough of MeshGuardian

This guide provides a **visual overview** of how data flows through the MeshGuardian architecture—from the moment a message is created, to when it is received, validated, and stored or relayed.

This is ideal for visual learners or anyone new to how MeshGuardian works.

---

## 🧭 System Overview Diagram

![MeshGuardian Overview Diagram](/assets/system_overview.png)

> 📁 Make sure the image is placed in the `/assets/` folder of your GitHub repo.

---

## 🧩 Core Flow (Step-by-Step)

1. **User or Sensor Generates Data**  
   → Passed to `PacketCreator.create()`  
   → Profile headers and encryption rules applied

2. **Encryption Layer**  
   → Calls `/security/encryption.md`  
   → Uses a derived key from `/security/secure_key_derivation.md`

3. **Routing Decision**  
   → `PacketRouter.route()` chooses next hop  
   → Based on trust graph (if PBFT) or stake (if PoS)

4. **Transmission**  
   → `PacketSender.send()`  
   → Hardware via `/device/hardware_interface.md`

5. **Reception & Validation**  
   → `PacketReceiver.receive()`  
   → Signature and timestamp checked  
   → Calls `PacketValidator.validate()`

6. **Buffering (Optional)**  
   → `PacketBuffer.enqueue()` if node is offline or busy

7. **Decryption + Final Use**  
   → `decrypt_data()` from `/security/encryption.md`  
   → Used by the application or displayed to the user

---

## 🔁 Consensus Path (For Critical Packets)

If the packet is tagged `priority: high`, the following path is activated:

- `ConsensusEngine.run_pbft()`  
- Logs event to `/audit/audit_trail.md`  
- Writes hash to blockchain (e.g., Solana integration)

See `/consensus/pbft_validation.md` for details.

---

## 📡 Example Use Case: Emergency Chat

```plaintext
Alice types a message →
Encrypted with emergency profile →
Routed to Bob via 3 mesh hops →
PBFT validates it (quorum of 4) →
Bob receives + decrypts it →
Stored in chat history (encrypted)
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
