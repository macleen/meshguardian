# 🔗 4. Protocol Demo: MeshGuardian Packet Lifecycle

This guide walks you through a **minimal working demonstration** of MeshGuardian’s protocol, showcasing how a packet is created, validated, routed, and received between nodes.

You'll see how the **pseudo-code** maps to real-world flow using simplified examples and mock values.

---

## 🧱 Prerequisites

Before running the protocol demo, make sure you’ve done the following:

- ✅ Set up your dev environment (see `1_environment_setup.md`)
- ✅ Assembled a test node or simulated one (see `2_simulated_node.md`)
- ✅ Installed any firmware or software from `3_firmware_quickstart.md`

---

## 📦 Step 1: Create a Packet

Module: `/pseudo-code/networking/packet_creation.md`

```python
creator = PacketCreator()
packet = creator.create("Node_A", "Node_B", b"Emergency message", profile="emergency")
```
This will auto-generate:  
- id: Unique UUID  
- headers: TTL, priority, and profile
- data: Raw bytes payload  

Profile-based attributes like ttl=1800s or priority=urgent are pulled from /protocol/profiles.md

## Step 2: Validate the Packet
Module: /pseudo-code/networking/packet_validation.md
```python
if validator.validate(packet):
    print("Packet validated")
else:
    print("Packet rejected")
```

## Step 3: Route the Packet
Module: /pseudo-code/networking/packet_routing.md  
```python
    route = router.find_shortest_path("Node_A", "Node_B")
    router.forward(packet, route)
```
MeshGuardian uses trust-weighted paths (see /consensus/trust_graphs.md)

If node health is degraded, it finds alternative paths  
In DTN mode, packets may be stored until the next hop is online.  

## Step 4: Receive the Packet
Module: /pseudo-code/networking/packet_receiving.md  
```python
    receiver.listen()
    message = receiver.read_packet()
    print(message.data.decode("utf-8"))
```
Verifies headers again  
Optionally logs metadata (audit trail)  

Recap: Full Lifecycle  
Phase	Module	Outcome
Creation	packet_creation.md	Packet initialized with profile
Validation	packet_validation.md	Checked against security rules
Routing	packet_routing.md	Path determined via trust graph
Receiving	packet_receiving.md	Packet delivered + logged

Optional: Add Compression or Encryption
Enhance the protocol demo by:

🔒 Using /security/encryption.md to encrypt packet.data
📦 Using /compression/compression.md to reduce packet size

**Next Step**: 5_data_flow.md
Continue to the next file to understand how the entire data pipeline is structured—across modules, devices, and environments.
