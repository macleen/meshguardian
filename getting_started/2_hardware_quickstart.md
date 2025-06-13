# 2. Hardware Quickstart: Build Your First MeshGuardian Node

This guide walks you through assembling a basic MeshGuardian node using a **Raspberry Pi** (or compatible MCU like STM32WL), suitable for field use in disconnected environments.

---

## What You’ll Need

| Component                | Recommended Model                    |
|--------------------------|--------------------------------------|
| Single Board Computer    | Raspberry Pi 3/4 or STM32WL55        |
| Wireless Module (LoRa)   | Dragino LoRa HAT / EByte E22 / SX1262 |
| GPS Module               | u-blox M10 or M8                    |
| Power Supply             | 5V 2.5A adapter / Battery Pack (supports Bit 23 Low-Energy Mode) |
| SD Card (min. 8GB)       | Class 10 or better                   |
| Antenna (LoRa + GPS)     | 868/915 MHz + passive GPS patch     |
| Optional Case            | Weatherproof enclosure               |

> 💡 Tip: For outdoor testing, use solar + battery combo with a charge controller to support Low-Energy Mode (Bit 23).

---

## Step 1: Flash the Operating System

### Raspberry Pi

1. Download [Raspberry Pi OS Lite](https://www.raspberrypi.com/software/).
2. Flash using [Raspberry Pi Imager](https://www.raspberrypi.com/software/).
3. Enable **SSH** via `raspi-config` (or add `ssh` file to boot partition).

### STM32WL

- Use STM32CubeProgrammer to flash the MCU with precompiled firmware from `/hardware/`.

---

## Step 2: Connect Modules

### LoRa

- Connect LoRa HAT to GPIO headers (Raspberry Pi) or pin headers (STM32WL).
- Ensure TX/RX pins match your board configuration.

### GPS

- Wire GPS TX → MCU RX, and RX → TX.
- Antenna must have a clear sky view.

---

## Step 3: Power Up and Boot

- Insert the SD card (Pi) or connect debugger (MCU).
- Power on your board.
- SSH into Raspberry Pi or connect via USB serial to STM32.

---

## Step 4: Install MeshGuardian Software

Clone and set up:

```bash
    git clone https://github.com/macleen/meshguardian.git
    cd meshguardian
    sudo ./install.sh
```

Configure 64-bit capability flags in config.py to enable features like Low-Energy Mode (Bit 23) or post-quantum cryptography (Bit 39; see /protocol-specs/capability_flags.md).

🧠 Refer to /hardware/4_software_implementation.md for detailed firmware setup.


## Step 5: Verify Node Operation
Run diagnostics:
```bash
    python3 tools/ping_mesh.py
```
Expected Output:
```output  
    ✔ LoRa module detected  
    ✔ GPS lock acquired  
    ✔ Encrypted packet test: OK  
    ✔ Low-Energy Mode (Bit 23): OK
```
## Next Steps
- Secure Your Node: Configure encryption and authentication settings.
- Test RF Performance: Validate LoRa and GPS with /hardware/9_RF_validation_procedure.md.
- Contribute to Hardware Layer: See /CONTRIBUTING.md.
- Software-Hardware Alignment: Ensure 64-bit capability flags in config.py match hardware capabilities (e.g., Bit 23 for battery-powered setups).  

--

**Need help?** Open a GitHub issue at https://github.com/macleen/meshguardian/issues or check our community chat.