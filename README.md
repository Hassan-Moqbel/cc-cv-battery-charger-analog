# CC/CV Battery Charger

![Power Electronics](https://img.shields.io/badge/Domain-Power_Electronics-FF6F00?style=for-the-badge)
![Battery Management Systems](https://img.shields.io/badge/Topic-BMS-009999?style=for-the-badge)
![CC-CV Charging Profile](https://img.shields.io/badge/Topology-CC_CV_Charging-4B0082?style=for-the-badge)
![Analog Regulation](https://img.shields.io/badge/Circuit-Analog_Regulation-00599C?style=for-the-badge)
![Hardware Verified](https://img.shields.io/badge/Status-Breadboard_Verified-28A745?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)

## Executive Overview
Safely restoring energy to secondary battery cells (such as Lithium-Ion or Sealed Lead-Acid) requires strict adherence to specific charging algorithms. This project details the theoretical design and physical breadboard implementation of an analog **Constant-Current / Constant-Voltage (CC/CV) Battery Charger**. Utilizing linear analog regulation, the hardware autonomously manages the two critical phases of battery charging: delivering a high continuous bulk current (CC) to quickly restore capacity, followed by a clamped absorption voltage (CV) that gently tapers the current to zero as the cell reaches peak saturation.

> [!WARNING]
> **Battery Safety & Thermal Runaway Callout**
> Charging sensitive battery chemistries like Lithium-Ion or Lithium-Polymer without proper voltage limits will directly cause catastrophic thermal runaway, venting, and combustion. Overcharging severely degrades cell chemistry. Furthermore, this CC/CV power supply is purely a charging regulator and does not replace a dedicated Battery Management System (BMS). A secondary BMS board must always be installed on the battery pack to provide absolute over-voltage, under-voltage, and short-circuit hard-cutoff protection.

## System Highlights
- **Dual-Stage Analog Charging Profile**: Safely manages both the high-power bulk charging phase and the low-power absorption phase.
- **Autonomous Mode Transition**: Seamlessly crosses the boundary between constant current and constant voltage regulation based purely on the rising internal voltage of the battery.
- **Reverse Discharge Blocking**: A robust series rectifier diode prevents the battery from back-feeding into the regulator circuitry when the primary DC power source is disconnected.
- **Short-Circuit Protection**: The constant-current stage inherently limits the absolute maximum current output, effectively providing indefinite short-circuit tolerance.

## System Architecture Diagram

mermaid
flowchart LR
    DC["DC Power Source"] --> CC["Current Limiting Stage \nCC Mode"]
    CC -->|I_charge| CV["Voltage Clamping Stage \nCV Mode"]
    CV -->|V_cv| DIODE["Reverse Polarity & \nBack-feed Blocking Diode"]
    DIODE --> BATT["Battery Terminals"]


## Theoretical & Mathematical Models

### 1. Constant Current Phase Regulation (Bulk Charging)
During the CC phase, the battery's voltage is low. The current regulator dominates the circuit, forcing a constant bulk current defined by a reference voltage and a precision sense resistor:
$$I_{"charge"} = \frac{"V_{ref"}}{R_{"sense"}}$$

### 2. Constant Voltage Clamping Regulation (Absorption/Float)
As the battery charges, its terminal voltage rises. Once it hits the target voltage, the secondary regulator takes over to clamp the voltage, defined by its resistor divider network:
$$V_{"cv"} = V_{"ref"} \left(1 + \frac{"R_2"}{R_1}\right)$$

### 3. Autonomous Transition Boundary Condition
The exact moment the circuit switches from the Bulk (CC) phase to the Absorption (CV) phase occurs when the battery's terminal voltage matches the clamped voltage threshold:
$$\text{"Mode Switch at "} V_{"cell"}(t) = V_{"cv"} \implies I_{"charge"} \text{" begins exponential decay"}$$

### 4. Exponential Current Decay (CV Stage)
During the CV stage, the battery voltage is held constant while its internal charge approaches $100\%$. The current tapers off exponentially:
$$i_b(t) = I_{"charge"} \cdot e^{-\frac{"t - t_0"}{\tau}}$$
*(Charging is typically terminated or switched to a trickle float when $I_{"cutoff"} \approx 0.1 \times I_{"charge"}$).*

### 5. Linear Pass Element Thermal Dissipation
Because this relies on analog linear regulators, the maximum heat is generated when the battery is completely dead, producing the highest voltage drop across the regulator:
$$P_{"D(max)"} = (V_{"in"} - V_{"batt,min"}) \cdot I_{"charge"}$$
*(Heatsinks must be sized to continuously dissipate $P_{"D(max)"}$ to prevent thermal shutdown).*

## Hardware Bill of Materials (BOM)
| Component | Function |
| :--- | :--- |
| **Voltage/Current Regulators** | e.g., LM317T or LM338K for analog linear power regulation |
| **Current Sense Resistors** | High-wattage wirewound/ceramic resistors ($R_{"sense"}$) |
| **Shunt Potentiometers** | Precision multi-turn trimpots to set $V_{"cv"}$ thresholds |
| **Power Rectifier Diodes** | High-current blocking diode (e.g., 1N5408 or Schottky for low $V_f$) |
| **Heat Sinks** | TO-220/TO-3 aluminum extrusion heatsinks for thermal stability |

## Calibration & Multi-stage Tuning Guide
1. **Setting the Output Voltage ($V_{"cv"}$)**: Disconnect the battery. Measure the output terminals with a multimeter. Adjust the voltage tuning potentiometer until the output exactly matches the battery manufacturer's specified charge voltage (e.g., $14.4\text{"V"}$ for SLA or $4.2\text{"V"}$ for a single Li-ion cell), adding $+0.7\text{"V"}$ to compensate for the blocking diode drop.
2. **Setting the Current Limit ($I_{"charge"}$)**: Place a high-current ammeter directly across the output terminals (creating a short circuit). The CC stage will prevent failure. Adjust the current-sense resistor (or trimpot) until the meter reads the desired bulk charging current (e.g., $0.5\text{"C"}$ or $1.0\text{"C"}$ of battery capacity).

## Authentic Artifacts Catalog
- **Engineering Report**: [`docs/CC and CV battery charging_حسن مقبل.pdf`](docs/)
- **Original Schematics & Physical Hardware**: Located in ["`docs/images/`"](docs/images/) as **[ORIGINAL SCHEMATIC & PROTOTYPE ARTIFACTS]**.

## Engineering Audit & Tradeoffs
- **Linear Analog Regulation vs. Switch-Mode (Buck) Charging**: This analog CC/CV topology is exceptionally simple, inexpensive, and introduces zero Electromagnetic Interference (EMI) to nearby RF/Audio circuits. However, it is highly thermally inefficient. Dropping $24\text{"V"}_{"in"}$ down to charge a $12\text{"V"}$ battery at $2\text{"A"}$ burns $24\text{"Watts"}$ of pure heat. In contrast, a modern Switch-Mode Power Supply (SMPS) buck converter chip (like the XL4015) can perform the exact same CC/CV profile at $>90\%$ efficiency without massive heatsinks, at the cost of high-frequency inductor noise.

---

**Hassan Moqbel Morshed Ghaleb**
Mechatronics Engineer | Mechanical Design & CAD (SolidWorks & AutoCAD) | Preventive Maintenance & Electromechanical Systems | Industrial Automation, Control Systems, Robotics & Intelligent Machines | CAD/FEA, Embedded Systems, Python & C++
[GitHub](https://github.com/Hassan-Moqbel) · [Facebook](https://www.facebook.com/share/1BqxAgVjHi/) · [LinkedIn](https://www.linkedin.com/in/hassan-moqbel)

## License
This project is licensed under the ["MIT License"](LICENSE).
