# 3. Firmware Quickstart: Flashing MeshGuardian to STM32WL

This guide walks you through setting up and flashing the **MeshGuardian firmware** to your **STM32WL** device using **STM32CubeIDE** or **Zephyr RTOS** (depending on your stack choice).

---

## Supported Platforms

| MCU Target Architecture | Platform Option      | Interface         |
|-------------------------|----------------------|-------------------|
| STM32WL55XX             | STM32CubeWL (ST)     | HAL / Low-level   |
| STM32WL55XX             | Zephyr RTOS          | LoRaWAN driver (supports Bit 23 Low-Energy Mode) |
| Raspberry Pi            | Python/C++ Layer     | Via UART to LoRa  |

> 📌 **Note**: For production-ready firmware, STM32WL + STM32CubeWL is recommended.

---

## Option 1: Flash with STM32CubeIDE

### Prerequisites
- [STM32CubeIDE](https://www.st.com/en_US/development-tools/stm32cubeide.html)
- USB ST-Link debugger or J-Link Mini

### Steps
1. **Clone the Repository**:
   ```bash
   git clone https://github.com/macleen/meshguardian.git
```

2. Open STM32CubeIDE:
3. Import the project under /firmware/stm32wl/ into the workspace.
4. Build & Flash
Click the "Build" (hammer) icon to compile.  
Click the "Debug" (bug) icon to flash and run the firmware.  

Note:  
Code relies on CubeWL’s SUBGHZ HAL driver for RF handling.    

Option 2: Build with Zephyr RTOS (Advanced)  
Install Zephyr Toolchain  
```bash
    west init zephyrproject
    cd zephyrproject
    west update
    pip install -r zephyr/scripts/requirements.txt
```
Create Firmware
```bash
    cd zephyrproject
    west build -b stm32wl55jc_dk samples/lorawan/class_a
```

Configure Firmware
Configure LoRaWAN settings and OTAA keys in prj.conf.  
Configure 64-bit capability flags (e.g., Bit 23 for Low-Energy Mode, Bit 39 for post-quantum cryptography) to align with protocol features (see /protocol-specs/capability_flags.md).

Flash Firmware
```bash
    west flash
```

Verify Flash Success
Check for serial output:
```bash
   screen /dev/ttyUSB0 115200
```
Expected logs:
```csharp
    [MeshGuardian] Starting LoRa session...
    [MeshGuardian] Join request sent...
    [MeshGuardian] OTAA Join Success!
```
Run Test Packet
Use the CLI tool:
```bash
   python3 tools/send_packet.py --msg "Hello Mesh!"
```
Expect LoRa transmission log and confirmation of encrypted output (verified with Bit 39 for post-quantum cryptography and Bit 36 for compression).


**Next Steps**
- Secure Join Configuration: Set up secure LoRaWAN authentication.
- Enable GPS + LoRa Join: Integrate GPS module for location-aware nodes.
- Test RF + GPS: Validate performance with /hardware/9_RF_validation_procedure.md.
-  Software-Hardware Alignment: Configure 64-bit capability flags in config.py or prj.conf to match hardware capabilities (e.g., Bit 23 for low-power operation).

---

**Need help** Open a GitHub issue at https://github.com/macleen/meshguardian/issues or join our community on Matrix #meshguardian:matrix.org.