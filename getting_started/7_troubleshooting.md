# 7. Troubleshooting Guide for MeshGuardian

This guide helps you debug common problems when working with **MeshGuardian**, whether you're setting up your node, configuring firmware, or testing in the field.

---

## Hardware Issues

### 🔌 Node Won’t Power On
- **Check:** Is your battery connected with correct polarity?
- **Fix:** Use reverse polarity protection (see `/hardware/12_PCB_reverse_polarity_protection.md`).

### Voltage Irregularities
- **Check:** Is the 3.3V rail steady (±5%)?
- **Fix:** Inspect the LDO or buck converter. Test with a multimeter.

---

## LoRa Communication Issues

### No LoRa Output Detected
- **Check:** Is `HAL_SUBGHZ_Transmit()` or equivalent being called?
- **Fix:** Ensure SUBGHZ SPI is initialized. Re-check LoRa mode configuration.

### Poor Range
- **Check:** Antenna placement and impedance match.
- **Fix:** Run `Radio_Calibrate()` (CubeWL) or use a VNA. Ensure 50Ω match.

### OTAA Join Fails
- **Check:** Are DevEUI, AppKey, AppEUI correctly set?
- **Fix:** Log join requests. Validate region config (EU868/US915) in `prj.conf`.

---

## GPS Problems

### GPS Not Locking
- **Check:** Is the antenna facing sky with clear view?
- **Fix:** Supply 3.3V to GPS module. Verify UART baud (default: 9600).

### Slow Cold Start
- **Check:** First lock should be <60 seconds.
- **Fix:** Test with open-sky view or use GPS simulator for lab testing.

---

## Security Failures

### Decryption Errors
- **Check:** Are encryption keys mismatched?
- **Fix:** Verify correct key retrieval via `key_management.retrieve_key()`.

### Timestamp Rejected
- **Check:** Is your system clock accurate?
- **Fix:** Sync RTC with GNSS or NTP source. Allow for ±30 sec drift.

---

## Testing Failures

### Tests Don’t Run
- **Check:** Are your dependencies installed?
- **Fix:** Run:
```bash
    pip3 install -r requirements.txt
```
GPS/LoRa Tests Fail
Check: Hardware present and wired correctly?

Fix: Review /hardware/5_testing.md and /hardware/9_RF_validation_procedure.md.

🛡 Consensus/Validation
❌ PBFT Stalls
Check: Is quorum reachable (≥4 nodes)?

Fix: Ensure nodes are synced and all run the same consensus config.

⚠️ High Latency
Check: Are multiple priority packets flooding the network?

Fix: Tune rate-limiting in packet_creation.profile.

🧼 General Cleanup
🧹 Reset Configuration
To reset and start fresh:
```bash
   cd meshguardian
    git clean -fdx
```
Still Stuck?
Check the docs under /pseudo-code/, /hardware/, and /security/.

Open a GitHub Issue with:  
- logs/  
- config/  
- hardware version and firmware branch  

**Next Steps**
Head to:  
➡ 8_contributing.md to learn how to contribute  
➡ /hardware/ to validate hardware integrations  
➡ /networking/packet_validation.md if you’re debugging data flows  