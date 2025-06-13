# MeshGuardian Firmware Integration Guide

**Target**: STM32WL (Cortex-M4) + LoRaWAN + GPS  
This guide outlines how to integrate firmware into your MeshGuardian hardware, including LoRaWAN stack selection, GPS driver implementation, and debugging strategies.

---

## 1. Firmware Stack Options

### A. LoRaWAN Stacks

| Stack         | Pros                                  | Cons                        |
|---------------|---------------------------------------|-----------------------------|
| **LoRaMac-node** | Official Semtech stack, robust        | Steep learning curve        |
| **Zephyr OS**    | Integrated RTOS, modular              | Larger footprint (~100KB+) |
| **RIOT-OS**      | IoT-optimized, lightweight            | Smaller community           |

**Recommendation**: Use **Zephyr OS** for its built-in LoRaWAN support and hardware abstraction.

### B. GPS Library

- **U-blox M10 driver** (via U-blox AT client)
- **NMEA parser** (e.g., `minmea` for lightweight parsing)

---

## 2. Firmware Setup Steps

### A. Project Initialization

```bash
west init zephyrproject
cd zephyrproject
west update
pip install -r zephyr/scripts/requirements.txt
```  

Create a new LoRaWAN project:  
```bash
west build -b stm32wl55_cm4 samples/lorawan/class_a
```  

B. Key Configurations
```ini
LoRaWAN Region Setup (prj.conf):  
CONFIG_LORAMAC_REGION_EU868=y  # or US915
CONFIG_LORAWAN_DEVICE_EUI="PUT_DEVEUI_HERE"
```  

STM32WL RF Switch Setup (overlay.dts):  
```dts
&lora {
  rxtx-gpios = <&gpioc 13 GPIO_ACTIVE_HIGH>;  // Example pin
};
```  

C. GPS Integration (Zephyr)  
```c
#include <drivers/gps.h>
void main() {
    const struct device *gps = DEVICE_DT_GET(DT_NODELABEL(ublox_m10));
    gps_start(gps);  // Begin polling NMEA data
}
```  

## 3. LoRaWAN Activation Modes  
Ensure packet creation aligns with 64-bit capability flags (e.g., Bit 36 for extended compression, Bit 39 for post-quantum cryptography; see protocol-specs/capability_flags.md).  

A. Over-The-Air Activation (OTAA) 
```c
struct lorawan_join_config join_cfg = {
    .mode = LORAWAN_ACT_OTAA,
    .dev_eui = {0x01, 0x02, ...},
    .app_eui = {0x01, 0x02, ...},
    .app_key = {0x01, 0x02, ...},
};
lorawan_join(join_cfg);
```  
B. Activation By Personalization (ABP) 
```c
    struct lorawan_join_config join_cfg = {
        .mode = LORAWAN_ACT_ABP,
        .dev_addr = 0x260BABCD,
        .nwk_skey = {0x01, 0x02, ...},
        .app_skey = {0x01, 0x02, ...},
    };
```  
## 4. Testing & Debugging
A. LoRaWAN Debug Commands  
# Monitor LoRaWAN gateway frames
```bash
    sudo tcpdump -i lo -X -n udp port 1700
```  

B. GPS Data Logging  
```c
    printf("Lat: %f, Lon: %f\n", gps_data.latitude, gps_data.longitude);
```

C. Power Optimization 
```c
    // Enter STOP mode when idle
    pm_power_state_force(PM_STATE_STOP);
```  

## 5. Example Firmware Repository
Clone a working Zephyr-based template or MeshGuardian-specific repository:    
```bash
    git clone https://github.com/macleen/meshguardian  # MeshGuardian-specific
    git clone https://github.com/zephyrproject-rtos/zephyr --branch v3.4.0  # Zephyr template
```  

## 6. Software-Hardware Alignment
Validate firmware support for 64-bit capability flag-driven features, such as:  

- Low-Energy Mode (Bit 23): Optimizes power for off-grid deployments.
- Extended Compression (Bit 36): Requires processing for zlib or Brotli in packets.
- Post-Quantum Cryptography (Bit 39): Demands resources for Kyber/Dilithium.
- Multi-Blockchain Logging (Bit 37): Needs stable LoRaWAN for audit trails.  

See protocol-specs/capability_flags.md for flag definitions and test firmware with varied flag settings.


Troubleshooting Checklist  
Issue	Suggested Solution
LoRa join fails	Check DevEUI/AppKey, antenna connection
GPS no data	Verify UART baud rate (default is 9600)
High current	Enable STM32WL low-power modes

## TODO  
- Implement firmware tests for 64-bit capability flag scenarios (e.g., Bit 36, Bit 39 enabled).