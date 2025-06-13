# MeshGuardian RF Validation Procedure

This document outlines the procedure for validating the RF performance of the MeshGuardian node, focusing on both LoRa and GPS subsystems. The validation ensures compliance with design targets and functional readiness for field deployment.

---

## Tools Required

- Spectrum Analyzer / VNA (Vector Network Analyzer)  
- LoRa Tester (e.g., LoRaWAN gateway or dedicated tester)  
- GPS Simulator or access to open-sky environment  
- Power Supply (3.3V ±5%)  
- Firmware configured with relevant 64-bit capability flags (e.g., Bit 23 for Low-Energy Mode; see `protocol-specs/capability_flags.md`)

---

## 1. LoRa RF Validation

### A. Conducted Power Test (Direct Cable Connection)

- Connect the PCB’s RF output to a spectrum analyzer using an SMA cable.
- Power on the PCB and enable Continuous Wave (CW) mode transmission at:
  - **868 MHz (EU)** or **915 MHz (US)**
  - Target output power: **+14 dBm**
- **Measure**:
  - Output Power: Should be within **±1 dB** of the target.
  - Frequency Error: Should be **< ±10 kHz** from the center frequency.

### B. Modulation Quality

- Transmit LoRa packets using:
  - **Spreading Factor (SF7)** and **125 kHz** bandwidth
  - Firmware configured with relevant 64-bit capability flags (e.g., Bit 36 for extended compression, Bit 39 for post-quantum cryptography; see `protocol-specs/capability_flags.md`) to reflect deployment settings.
- **Verify**:
  - Packet Error Rate (PER): Should be **<1%** at max range.
  - RSSI/SNR: Compare against reference design expectations.

### C. Radiated Test (Antenna Connected)

- Place the PCB in an anechoic chamber or open outdoor space (min. 3m from obstacles).
- **Measure**:
  - EIRP (Effective Isotropic Radiated Power) using a calibrated antenna.
  - Communication range: Should reach **≥1 km line-of-sight** with a gateway.

---

## 2. GPS Performance Validation

### A. Cold Start Time

- Power on the PCB with a clear sky view.
- Measure time to first GPS lock:
  - Target: **<60 seconds**

### B. Sensitivity Test

- Simulate weak signals by attenuating GPS signal strength from **-130 dBm** to **-150 dBm**.
- **Expected Behavior**:
  - **-130 dBm**: GPS should **maintain lock**
  - **-150 dBm**: GPS may **lose lock** (typical limit for consumer modules)

### C. Antenna Performance

- Measure VSWR at **1575.42 MHz**:
  - Target: **<2:1**
- Verify radiation pattern:
  - Ensure no nulls or dropouts in the **upward direction** (important for satellite view)

---

## 3. Interference Checks

- Monitor the **LoRa band (868/915 MHz)** for spurious emissions.
- Ensure **GPS signal is not desensitized** during LoRa transmission.

---

## 4. Software-Hardware Alignment

Validate RF performance under different 64-bit capability flag configurations to ensure hardware supports protocol features, such as:
- **Low-Energy Mode (Bit 23)**: Optimizes power during transmission.
- **Extended Compression (Bit 36)**: Increases packet size, potentially affecting PER or range.
- **Post-Quantum Cryptography (Bit 39)**: Adds computational overhead, possibly impacting timing.
- **Multi-Blockchain Logging (Bit 37)**: Requires stable RF for audit trail sync.

See `protocol-specs/capability_flags.md` for flag definitions and test with varied flag settings to confirm compatibility.

---

## Pass/Fail Criteria

| Test                      | Target Value             |
|---------------------------|--------------------------|
| LoRa Output Power         | +14 dBm ±1 dB            |
| LoRa Frequency Error      | < ±10 kHz                |
| GPS Cold Start            | < 60 seconds             |
| GPS Sensitivity           | Track at -130 dBm        |
| VSWR (GPS Antenna)        | < 2:1                    |

---

## Troubleshooting Tips

**Low LoRa Power?**  
- Check the **RF matching network** (LC values).  
- Verify **STM32WL RFIO settings** in firmware.  
- Ensure correct capability flag settings (e.g., Bit 23 for Low-Energy Mode) in firmware.

**GPS Not Locking?**  
- Confirm proper **antenna connection**.  
- Ensure stable **3.3V power supply** to the GPS module.

**Unexpected Packet Errors?**  
- Verify 64-bit capability flags (e.g., Bit 36 for compression) in firmware align with test conditions.

---

## TODO
- Develop test cases for RF performance under various 64-bit capability flag scenarios (e.g., Bit 36, Bit 39 enabled).

---

For accurate results, perform all tests in a controlled RF environment and log the results for traceability and certification purposes.