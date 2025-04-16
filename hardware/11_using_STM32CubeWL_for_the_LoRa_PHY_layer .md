# STM32CubeWL for LoRa PHY Integration – MeshGuardian Guide

This document explains how and why to use **STM32CubeWL** for implementing the **LoRa PHY layer** in your MeshGuardian project, including setup instructions, firmware integration, tuning, and troubleshooting.

---

## Why STM32CubeWL?

### ✅ Official ST Support
- Fully optimized for the **STM32WL series** with integrated **LoRa (SUBGHZ) radio**
- Handles **low-level RF timing**, modulation, and hardware abstraction
- Simplifies **certification compliance** and **radio configuration**

### ✅ Pre-Validated LoRaWAN Compliance
- Certified **LoRaWAN v1.0.4 stack**
- Supports **EU868**, **US915**, and other regional bands
- Reduces development and debugging time for radio-specific code

### ✅ Built-in Hardware Abstraction
Manages:
- RF switch control  
- TCXO handling  
- Power sequencing

---

## Implementation Guide

### 1. Install STM32CubeWL
- Download from the [STMicroelectronics website](https://www.st.com/en/embedded-software/stm32cubewl.html)
- Recommended IDE: **STM32CubeIDE** or **STM32CubeMX**

---

### 2. Key Configurations

#### A. LoRa PHY Initialization Example

```c
#include "stm32wlxx_hal_subghz.h"

SUBGHZ_HandleTypeDef hsubghz;

void LoRa_PHY_Init(void) {
  hsubghz.Init.BaudratePrescaler = SUBGHZSPI_BAUDRATEPRESCALER_4;
  HAL_SUBGHZ_Init(&hsubghz);

  // Set LoRa parameters (EU868 example)
  SUBGHZ_LoRa_InitTypeDef LoRaConfig = {
    .Freq = 868000000,
    .Spread = LORA_SF7,
    .Bandwidth = LORA_BW_125,
    .Coderate = LORA_CR_4_5,
    .PreambleLen = 8
  };

  HAL_SUBGHZ_LoRa_SetConfig(&hsubghz, &LoRaConfig);
}
```

**B. RF Output Tuning**
Use: SUBGHZ_RADIO_Calibrate()  
Verify with VNA:  
Target VSWR: < 1.5:1 at 868/915 MHz

## 3. Integration with Firmware
### Compatible With:
- **LoRaMac-node** (official Semtech stack included in CubeWL)
- **Zephyr RTOS** via STM32 HAL drivers  

Example: OTAA Join Request 
```c
    MlmeReqJoin_t joinParams = {
    .DevEui = {0x01, 0x02, ...},
    .AppEui = {0x01, 0x02, ...},
    .AppKey = {0x01, 0x02, ...},
    };

    LoRaMacMlmeRequest(&joinParams);
```

## Advantages Over Custom PHY

| Feature                | STM32CubeWL               | Custom PHY Implementation            |
|------------------------|---------------------------|---------------------------------------|
| Development Time       | ✅ Fast (ST-certified)     | ❌ Slow, higher risk                  |
| RF Performance         | ✅ Optimized               | ❌ Manual tuning needed              |
| Certification Readiness| ✅ Pre-tested (ETSI)       | ❌ Requires full re-test             |


**Critical Implementation Checks**
Clock Configuration  
Ensure TCXO is enabled (if used).  
CubeWL expects a 32 MHz reference clock in SystemClock_Config().  
Power Management  
Enter low-power mode with:  
```c
    HAL_SUBGHZ_SetLowPowerMode();
```

**Antenna Switch Control**
Common RXTX control pin: PC13  
Validate configuration in subghz_conf.h  

## Troubleshooting Tips

| Symptom         | Fix Suggestion                                                                 |
|------------------|--------------------------------------------------------------------------------|
| ❌ No LoRa Output | Check `SUBGHZ_SPI` initialization, verify `HAL_SUBGHZ_Transmit()` is called   |
| ⚠️ Poor Range     | Use `Radio_Calibrate()` and match antenna to 50Ω                              |
| ❌ GPS No Lock    | Check antenna wiring, ensure 3.3V rail is stable                              |


**Next Steps**
Import into Project
```c
    cp -r STM32CubeWL/Projects/NUCLEO-WL55JC/Examples/SUBGHZ /your_project
```

**Adapt to MeshGuardian**
Update pin mappings in subghz_conf.h  
Integrate with RTOS or Zephyr if used  
Match settings with /pseudo-code/protocol/ and /hardware/  

---

With STM32CubeWL, you can accelerate LoRa PHY development, reduce debugging complexity, and ensure a production-grade RF stack for MeshGuardian’s mission-critical deployments.




