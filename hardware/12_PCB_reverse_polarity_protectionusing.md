(AO3401 P-MOSFET with gate connected to BAT- via 10kΩ pull-down resistor R)

#### Advantage
- **Very low voltage drop**: ~20 mV vs. ~300 mV with a Schottky diode.
- Ideal for **battery-powered designs** with Low-Energy Mode (Bit 23) where efficiency is critical.

---

## 3. Testing

| Test Type              | Expected Result                      |
|------------------------|--------------------------------------|
| Reverse polarity test  | Current draw < 1 mA (no damage)      |
| Forward voltage drop   | SS34 ≈ 0.3V at 500 mA                |
|                        | AO3401 ≈ 0.02–0.05V at 500 mA        |
| Low-Energy Mode test   | Stable operation with Bit 23 enabled in firmware |

---

## 4. PCB Modifications (Rev 1.7)

- Add **SS34** footprint near `J1` (Battery Connector).
- Update **Bill of Materials** (BOM) with:
  - **Part**: SS34
  - **LCSC Part Number**: `C9457`
- 💲 Estimated Cost: **<$0.15 per unit**
- Optional: Add **AO3401** footprint and 10kΩ resistor for P-MOSFET option.

---

## 5. Software-Hardware Alignment

Ensure reverse polarity protection supports 64-bit capability flag-driven features, such as:
- **Low-Energy Mode (Bit 23)**: Requires minimal voltage drop for efficient power usage.
- **Extended Compression (Bit 36)**: Increases CPU load, demanding stable power delivery.
- **Post-Quantum Cryptography (Bit 39)**: Adds computational overhead, requiring robust power input.
- **Multi-Blockchain Logging (Bit 37)**: Needs consistent power for audit trail sync.

See `protocol-specs/capability_flags.md` for flag definitions and test protection methods under varied flag settings.

---

## 6. Troubleshooting

| Symptom               | Cause/Solution                                       |
|-----------------------|------------------------------------------------------|
| Diode gets hot        | Trace width too small → Increase to ≥1 mm           |
| MOSFET not switching  | Missing gate pull-down → Add 10kΩ resistor to GND   |
| Power instability     | Check firmware capability flags (e.g., Bit 23 for Low-Energy Mode) |

---

## TODO
- Evaluate protection methods under 64-bit capability flag scenarios (e.g., high CPU load with Bit 39 enabled).

---

For most MeshGuardian field use cases (e.g., outdoor deployments, crisis zones), the **SS34** Schottky diode strikes a good balance between protection, cost, and simplicity. Use the **AO3401** P-MOSFET for low-power scenarios with Bit 23 enabled.