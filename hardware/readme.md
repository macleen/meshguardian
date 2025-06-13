# Building a MeshGuardian Node Using Raspberry Pi

MeshGuardian is designed to provide secure, resilient communication in environments with limited or no traditional connectivity—such as rural areas, disaster zones, or crisis response deployments. One of the key ways to deploy MeshGuardian is by using a Raspberry Pi as a physical node in a mesh network. The MeshGuardian software relies on 64-bit capability flags to enable features like low-energy operation and advanced security, which influence hardware configurations (see `protocol-specs/capability_flags.md`).

This guide walks you through everything you need to build, configure, and deploy your own MeshGuardian node using readily available hardware components and open-source software.

---

## 📁 What's Inside This Folder

The `/hardware` subfolder contains a collection of step-by-step Markdown guides to help you build and launch a MeshGuardian node:

### 1. [`1_hardware_requirements.md`](./1_hardware_requirements.md)
A complete list of hardware components you'll need, including the Raspberry Pi, wireless modules (e.g., LoRa, Wi-Fi), antennas, and power supplies.

### 2. [`2_step_by_step_process.md`](./2_step_by_step_process.md)
Instructions for physically assembling the node and setting up the base operating system on the Raspberry Pi.

### 3. [`3_wireless_module_configuration.md`](./3_wireless_module_configuration.md)
How to configure the wireless communication modules for mesh networking, including frequency, range, and interface settings, with support for protocol features via 64-bit capability flags.

### 4. [`4_software_implementation.md`](./4_software_implementation.md)
Steps for installing and configuring the MeshGuardian software stack, including dependencies, system services, and 64-bit capability flag settings for protocol functionality.

### 5. [`5_testing.md`](./5_testing.md)
Tools and procedures for testing your MeshGuardian node, verifying packet transmission with 64-bit capability flags, and troubleshooting common issues.

### 6. [`6_deployment.md`](./6_deployment.md)
Guidelines for real-world deployment in various environments, including considerations for power, placement, and connectivity, with flag-driven configurations.

### 7. [`7_additional_considerations.md`](./7_additional_considerations.md)
Advanced tips and optional features such as GPS integration, heat dissipation, solar power, and rugged enclosures, optimized for flag-enabled modes.

### 8. [`8_feasability_review.md`](./8_feasability_review.md)
Technical feasibility review of hardware design choices including MCU, GPS module, power circuitry, and PCB layout considerations.

### 9. [`9_RF_validation_procedure.md`](./9_RF_validation_procedure.md)
Complete RF validation steps for LoRa and GPS modules including test setups, performance metrics, and compliance checks, with flag-influenced packet testing.

### 10. [`10_firmware_integration.md`](./10_firmware_integration.md)
Guide for integrating firmware with STM32WL, LoRaWAN stack options, GPS data acquisition, and software structure, supporting 64-bit capability flags.

### 11. [`11_using_STM32CubeWL_for_the_LoRa_PHY_layer.md`](./11_using_STM32CubeWL_for_the_LoRa_PHY_layer.md)
Details for leveraging STMicroelectronics’ CubeWL platform for optimized LoRa PHY-layer configuration on STM32WL, aligned with flag-driven protocol features.

### 12. [`12_PCB_reverse_polarity_protection.md`](./12_PCB_reverse_polarity_protection.md)
Methods to implement reverse polarity protection (diode, MOSFET), schematics, BOM updates, and PCB layout tips, supporting low-power flag settings.

---

## ⚙️ Getting Started

To begin, open [`1_hardware_requirements.md`](./1_hardware_requirements.md) and start gathering your components. Then, follow the guides in order for a complete walkthrough from assembly to deployment.

### Software-Hardware Integration
Configure 64-bit capability flags in `config.py` to enable MeshGuardian protocol features (e.g., Low-Energy Mode via Bit 23, extended compression via Bit 36). Refer to `protocol-specs/capability_flags.md` for flag definitions and ensure hardware settings align with software requirements.

---

> 💡 Tip: You don’t need to be a hardware expert. Each step is written with clarity in mind, making it accessible to makers, developers, and field engineers. Understanding 64-bit capability flags can help optimize your node’s performance.