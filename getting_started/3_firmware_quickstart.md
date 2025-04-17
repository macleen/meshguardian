# 3. Firmware Quickstart: Flashing MeshGuardian to STM32WL

This guide walks you through setting up and flashing the **MeshGuardian firmware** to your **STM32WL** device using **STM32CubeIDE** or **Zephyr RTOS** (depending on your stack choice).

---

## Supported Platforms

| MCU Target       | Stack Option      | Interface         |
|------------------|-------------------|-------------------|
| STM32WL55JC      | STM32CubeWL (ST)  | HAL / Low-level   |
| STM32WL55JC      | Zephyr RTOS       | LoRaWAN driver    |
| Raspberry Pi     | Python/C++ Layer  | Via UART to LoRa  |

> 📌 For production-ready firmware, STM32WL + STM32CubeWL is recommended.

---

## Option 1: Flash with STM32CubeIDE

### Prerequisites

- [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html)
- USB ST-Link debugger or J-Link Mini

### Steps

1. **Clone the Repo:**

```bash
    git clone https://github.com/macleen/meshguardian.git
```

2. Open STM32CubeIDE:
3. Import the project under /firmware/stm32wl/ into the workspace.
4. Build & Flash
Click the "hammer" (build), then "bug" (debug) to flash and run.

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
Flash Firmware
```bash
    west flash
```
Configure LoRa OTAA keys in prj.conf
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
Expect LoRa transmission log and encrypted output confirmation.

**Next Steps**
- Secure Join Configuration
- Enable GPS + LoRa Join
- Test RF + GPS

**Need help** compiling? Join #firmware-help on Discord or open an issue on GitHub.