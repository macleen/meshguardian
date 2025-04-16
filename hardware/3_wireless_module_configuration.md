# Wireless Module Configuration for MeshGuardian Node

This document provides detailed instructions for configuring the **wireless communication module** (e.g., LoRa HAT) on your Raspberry Pi-based MeshGuardian node. Proper configuration is essential for enabling **mesh networking capabilities**, especially in environments with limited or no connectivity.

![Wireless Module Configuration Diagram](/docs/assets/wireless_module_configuration.png)

---

## 1. Prerequisites

Before proceeding, make sure the following are completed:

- Raspberry Pi is set up with Raspberry Pi OS.
- Required interfaces (e.g., GPIO) are enabled via `raspi-config`.
- Wireless module (e.g., LoRa) is **physically attached** to the Raspberry Pi’s GPIO pins.

---

## 2. Install Required Drivers and Libraries

Most wireless modules need specific libraries or kernel drivers to function correctly.

For example, with a **LoRa module (SX127x-based)**:

### Required Libraries

- `RPi.GPIO`: For controlling GPIO pins.
- `sx127x`: Python library for Semtech SX127x LoRa modules.

### Install Using:

```bash
    sudo apt update
    sudo apt install python3-pip
    pip3 install RPi.GPIO
    pip3 install sx127x  # Replace with the correct library for your module
```
📘 Refer to your module’s datasheet or vendor docs for exact dependencies.  

## 3. Configure Module Parameters
You’ll need to configure settings such as frequency band, transmission power, and spreading factor to align with your network environment.

3.1 Frequency Band
Choose a band suitable for your region:  
Europe: 868 MHz  
North America: 915 MHz  
```python
    from sx127x import SX127x
    lora = SX127x()
    lora.set_frequency(868000000)  # Set to 868 MHz
```

3.2 Transmission Power  
Adjust the power level to balance range and energy use:  
```python
    lora.set_tx_power(14)  # 14 dBm
```  
3.3 Spreading Factor  
Spreading factor affects range vs. data rate:  
```python
    lora.set_spreading_factor(7)  # SF7 = shorter range, faster data
```
## 4. Integrate with MeshGuardian Software
Ensure your MeshGuardian software stack is aligned with your wireless module.  

Configuration Options:  
Device Path: Ensure correct SPI/I2C device path (e.g., /dev/spidev0.0).  

Protocol Settings: Match software parameters with module settings.  

Example Configuration Snippet:  
```json
{
  "wireless_module": {
    "type": "LoRa",
    "frequency": 868000000,
    "tx_power": 14,
    "spreading_factor": 7
  }
}
```
## 5. Test the Configuration
After setup, validate functionality through test scripts or diagnostic tools.  

Suggested Steps:  
Send Test Packet: Use a script to transmit data.  

Monitor Signal Strength: Confirm reception, analyze RSSI, and adjust   settings.  

Example Command:  
```bash
    python3 test_lora.py
```
🧪 Monitor for transmission success and debug using serial logs or terminal output.  

---

You're now ready to deploy a MeshGuardian node with working wireless communication. If you encounter issues, revisit the frequency, power, and spreading factor settings to ensure regional and device compatibility.  


