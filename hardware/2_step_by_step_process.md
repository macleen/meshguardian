# Step-by-Step Process for Setting Up a MeshGuardian Node

This guide provides a structured process for assembling and configuring a MeshGuardian node using a Raspberry Pi. It covers everything from preparing the hardware to installing the software, ensuring your node is ready for testing and deployment.

![Alt Text](/docs/assets/node_setup.png)

---

## 1. Prepare the Raspberry Pi

### Flash the Operating System
- Download **Raspberry Pi OS** (formerly Raspbian).
- Use [Raspberry Pi Imager](https://www.raspberrypi.com/software/) to flash the image onto a **microSD card**.

> 💡 **Tip**: Use the "Lite" version for a minimal setup if you’re comfortable with command-line interfaces.

### Insert the MicroSD Card
- Place the flashed microSD card into the Raspberry Pi.

### Connect Peripherals (Optional)
- Attach a monitor, keyboard, and mouse for the initial setup.
- Alternatively, enable **SSH** for headless remote access.

### Power On
- Connect the power supply to boot the Raspberry Pi.

---

## 2. Initial Configuration

### Update the System
Open a terminal and run:
```bash
    sudo apt update && sudo apt upgrade
```
Enable Necessary Interfaces
Use the configuration tool:
```bash
    sudo raspi-config
```
Navigate to Interfacing Options and enable interfaces like GPIO, I2C, or SPI, as required by your wireless module.

## 3. Assemble the Hardware
Attach the Wireless Module
Connect your LoRa HAT or other wireless module to the Raspberry Pi’s GPIO pins.

Refer to the module’s documentation for exact pin layout.

Optional – Add Battery Pack  
For off-grid operation, connect a battery pack to the Raspberry Pi’s power input.  

Place in Case  
Enclose the setup in a protective case to shield against dust, moisture, or impact.

## 4. Configure the Wireless Module
Install Drivers  
For LoRa or similar modules, run:
```bash
    sudo apt install python3-pip
    pip3 install RPi.GPIO
```
Follow any additional setup steps in the module’s documentation.

Set Parameters  
Configure frequency bands, power levels, and regional compliance settings.  
This may involve editing configuration files or running setup scripts.  

## 5. Install MeshGuardian Software
Clone the Repository  
```bash
    git clone https://github.com/your-repo/meshguardian.git
```    
Replace the URL with the actual MeshGuardian repository.  
Install Dependencies  
```bash
    cd meshguardian
    pip3 install -r requirements.txt
```
Configure the Software  
Edit the configuration files to match your hardware setup and network   environment.  
  
Follow guidance in the /pseudo-code/ folder for protocol, security, and device logic.

## 6. Set Up Auto-Start (Optional)
Enable on Boot  
Edit rc.local:
```bash
    sudo nano /etc/rc.local
```
Add your software start command before exit 0. 
Alternatively, create a custom systemd service for better control.

## 7. Reboot and Verify
Reboot the System  
```bash
    sudo reboot
```
Check Services  
Confirm that the MeshGuardian software runs at startup.  

Ensure the wireless module is active and transmitting as expected.  
Use logs or diagnostics tools for troubleshooting.  

---

This process will help you assemble and configure your MeshGuardian node efficiently. For advanced topics (e.g., testing, deployment, optimization), continue to the next files in this /hardware folder.