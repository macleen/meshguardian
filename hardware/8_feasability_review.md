# Feasibility Review for MeshGuardian Node

![Alt Text](/docs/assets/PCB_assembly_drawing.png)

This document provides a feasibility review of the hardware components and design choices for building a MeshGuardian node. It focuses on key areas including **MCU & LoRa integration**, **GPS module**, **power management**, **sensors**, **PCB layout analysis**, and **software-hardware alignment**.

---

## 1. MCU & LoRa Integration (STM32WL)

The **STM32WL55CC** is a dual-core Cortex-M4/M0+ microcontroller with an integrated LoRa radio, making it an excellent choice for low-power mesh networking applications.

### Pros
- Integrated LoRa modem (saves space, reduces BOM).
- Supports Sub-GHz bands (868/915 MHz for EU/US regions).
- Sufficient processing power for 64-bit capability flag features (e.g., extended compression via Bit 36, post-quantum cryptography via Bit 39).

### Potential Issues
- STM32WL availability can be problematic — verify supply chain status.
- RF matching network (in schematic) must be tuned for optimal range and performance.

---

## 2. GPS Module (U-blox MAX-M10S?)

The node likely uses a **U-blox M10 series** GPS module known for its low power consumption and high sensitivity.

### Pros
- Good for outdoor tracking.
- Low power operation (~20mA active).

### Concerns
- Antenna placement is critical — must ensure a clear sky view.
- Passive components (e.g., SAW filter, LNA) must be correctly selected and placed.

---

## 3. Power Management

Includes battery charging (e.g., LiPo) and DC-DC buck conversion (e.g., **TPS62743**).

### Pros
- Supports solar charging — ideal for remote or crisis deployments with Low-Energy Mode (Bit 23; see `protocol-specs/capability_flags.md`).
- High-efficiency conversion (~90%+).

### Concerns
- No visible load switch — risk of battery drain during idle state?
- Reverse polarity protection appears to be missing — critical for field use.

---

## 4. Sensors (If Present)

Assumed use of I2C or SPI-based sensors (e.g., temperature, humidity).

### Suggestion
- Add **TVS diodes** on I2C lines for **ESD protection** and field durability.

---

## 5. PCB Layout Analysis

### 4-Layer Stackup (Assumed)
- **Top Layer**: Components and RF traces.
- **Inner Layers**: Ground plane (critical for clean RF operation).
- **Bottom Layer**: Signal routing and shielding.

### RF Section (LoRa & GPS)

#### Good Practices Observed:
- 50Ω impedance-controlled traces used for antenna lines.
- Ground plane cutouts around RF sections to reduce high-frequency noise.

#### Potential Issues:
- LoRa antenna placement near GPS antenna — may cause RF interference.
- No π-matching network tuning shown — needed for optimal SWR and range.

### Decoupling & Noise Reduction
- STM32WL requires proper decoupling: **100nF + 4.7µF** close to each VDD pin.
- GPS LNA power should be filtered using a **ferrite bead and capacitor combo** to reduce noise.

---

## 6. Software-Hardware Alignment

Ensure hardware capabilities support software features enabled by 64-bit capability flags, such as:
- **Low-Energy Mode (Bit 23)**: Optimizes power usage for off-grid deployments.
- **Extended Compression (Bit 36)**: Requires sufficient MCU processing for zlib or Brotli.
- **Post-Quantum Cryptography (Bit 39)**: Demands additional computational resources for Kyber/Dilithium.
- **Multi-Blockchain Logging (Bit 37)**: Needs adequate storage and processing for audit trails.

Validate hardware performance against these features during testing; see `protocol-specs/capability_flags.md` for flag definitions.

---

> This feasibility analysis is a living document. Future revisions may include test results, schematic snippets, or real-world validation insights.

## TODO
- Assess hardware performance under 64-bit capability flag-enabled features (e.g., CPU load with Bit 39 enabled).