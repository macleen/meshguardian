# Reverse Polarity Protection for MeshGuardian

**Objective:**  
Prevent damage to the MeshGuardian node if the battery connections (V+ and GND) are accidentally reversed.

---

## 1. Recommended Solutions

| Method         | Component  | Pros                               | Cons                         |
|----------------|------------|------------------------------------|------------------------------|
| Schottky Diode | SS34       | ✅ Low cost (~$0.10), simple        | ⚠️ ~0.3V voltage drop         |
| P-MOSFET       | AO3401     | ✅ Near-zero voltage drop           | ⚠️ Slightly more complex      |
| Diode Bridge   | 4x 1N5819  | ✅ Works in any polarity            | ❌ High voltage loss (~1.4V)  |

**Recommendation:**  
Use a **Schottky diode (SS34)** for simplicity, unless ultra-low dropout is required (in which case consider a P-MOSFET).

---

## 2. Implementation – Schottky Diode

### Schematic

BAT+ ──┤S
P-MOS (AO3401)
BAT- ──┤G───── PCB_GND
│D
└────── PCB_V+  

### Advantage
- **Very low voltage drop**: ~20 mV vs. ~300 mV with a Schottky diode.
- Suitable for **battery-powered designs** where efficiency is critical.

---

## 4. Testing

| Test Type              | Expected Result                      |
|------------------------|--------------------------------------|
| Reverse polarity test  | Current draw < 1 mA (no damage)      |
| Forward voltage drop   | SS34 ≈ 0.3V at 500 mA                |
|                        | AO3401 ≈ 0.02–0.05V at 500 mA        |

---

## 5. PCB Modifications (Rev 1.7)

- Add **SS34** footprint near `J1` (Battery Connector).
- Update **Bill of Materials** (BOM) with:
  - **Part**: SS34
  - **LCSC Part Number**: `C9457`
- 💲 Estimated Cost: **<$0.15 per unit**

---

## Troubleshooting

| Symptom               | Cause/Solution                                       |
|-----------------------|------------------------------------------------------|
| Diode gets hot        | Trace width too small → Increase to ≥1 mm           |
| MOSFET not switching  | Missing gate pull-down → Add 10kΩ resistor to GND   |

---

For most MeshGuardian field use cases (e.g., outdoor deployments, crisis zones), the **SS34** Schottky diode strikes a good balance between protection, cost, and simplicity.
