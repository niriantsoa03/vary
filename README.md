# VARY
### Variable Architecture for Resistive-load pV sYstem

> *Vary* (ᥔᥣᥰ) — Malagasy for **rice**, the staple food this system is designed to cook.

A modular, low-cost solar PV cooking system built around a synchronous buck converter with impedance-matched resistive loads, designed for off-grid households in resource-constrained environments.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Motivation & Problem Statement](#2-motivation--problem-statement)
3. [System Architecture](#3-system-architecture)
4. [Design Rationale](#4-design-rationale)
5. [System Logic & Control](#5-system-logic--control)
6. [Key Design Equations](#6-key-design-equations)
7. [Simulation Results](#7-simulation-results)
8. [Project Phases](#8-project-phases)
9. [Repository Structure](#9-repository-structure)
10. [References](#10-references)

---

## 1. Project Overview

VARY is a master's thesis project targeting the design, simulation, and experimental validation of a **modular solar PV-powered cooking system** using a synchronous buck converter. The system is specifically designed for off-grid contexts where:

- Full PV hardware is not available from day one
- The cooking solution must scale incrementally as budget and resources allow
- Simplicity, repairability, and low cost are non-negotiable constraints

The core technical contribution is the use of **impedance-matched resistive load sizing** to replace active topology switching (i.e. no buck-boost converter needed), reducing component count, cost, and control complexity while maintaining full MPPT capability through a standard algorithm running on a synchronous buck converter.

**Power range:** 50 W – 150 W (scalable)  
**Target application:** Rice cooking, water boiling, general resistive heating  
**Target context:** Off-grid rural households, Madagascar

---

## 2. Motivation & Problem Statement

### The energy access gap

In many off-grid areas of Madagascar and similar developing contexts, households rely on charcoal, firewood, or LPG for cooking. These fuels cause indoor air pollution, deforestation, and represent a recurring cost burden. Solar PV adoption is growing, but most PV cooking solutions either:

- Require expensive inverters and AC induction cookers
- Use overly complex DC-DC converter topologies that are hard to repair locally
- Are not designed for incremental deployment — you need the full system upfront

### The research question

> *Can a low-power solar PV array combined with a low-capacity battery provide sufficient and reliable cooking energy using only a synchronous buck converter and switched resistive loads, in a modular architecture that scales from 50 W to 150 W?*

### Sub-questions

1. Can a synchronous buck converter with standard MPPT replace a more complex buck-boost topology for this application?
2. What is the minimum PV and battery sizing required for a complete cooking session under realistic irradiance conditions?
3. How does resistive load impedance matching through the `R_load < R_mpp` constraint enable reliable MPP tracking without a boost stage?
4. Can the system be validated incrementally — converter first, then thermal load, then full PV integration — without requiring all hardware simultaneously?

---

## 3. System Architecture

```
                    ┌─────────────────────────────────────────────┐
                    │              VARY System                     │
                    │                                              │
  ┌──────────┐      │  ┌──────────┐    ┌─────────────────────┐   │
  │  PV Panel│──────┼─▶│ SW_PV   │───▶│  Synchronous Buck   │   │
  │ 50-150 Wp│      │  │(on/off) │    │  Converter + MPPT   │   │
  └──────────┘      │  └──────────┘    └──────────┬──────────┘   │
                    │                             │               │
                    │                    ┌────────▼────────┐      │
                    │                    │    Battery      │      │
                    │                    │  12V / 20 Ah   │      │
                    │                    └────────┬────────┘      │
                    │                             │               │
                    │              ┌──────────────┼──────────┐    │
                    │              │                         │    │
                    │    ┌─────────▼──────┐    ┌────────────▼──┐ │
                    │    │   SW_Load1     │    │   SW_Load2    │ │
                    │    │  (on/off)      │    │  (on/off)     │ │
                    │    └─────────┬──────┘    └──────┬────────┘ │
                    │              │                   │          │
                    │    ┌─────────▼──────┐    ┌──────▼────────┐ │
                    │    │  CookingLoad1  │    │ CookingLoad2  │ │
                    │    │ Battery-fed    │    │  PV-fed       │ │
                    │    │ Constant draw  │    │ MPPT-tracked  │ │
                    │    │ R = Vbat²/P₁  │    │ R < R_mpp     │ │
                    │    └────────────────┘    └───────────────┘ │
                    │                                              │
                    │         ┌────────────────────┐              │
                    │         │   Microcontroller  │              │
                    │         │  (MPPT + Switch    │              │
                    │         │   Arbitration)     │              │
                    │         └────────────────────┘              │
                    └─────────────────────────────────────────────┘
```

### Component roles

| Component | Role |
|---|---|
| PV Panel (50–150 Wp) | Primary energy source |
| SW_PV | Connects/disconnects PV from the converter |
| Synchronous Buck Converter | Steps down PV voltage to battery/load voltage |
| MPPT Algorithm | Adjusts duty cycle to track maximum power point |
| Battery (12V, 20 Ah) | Energy buffer — stores excess PV, supplies load at night |
| SW_Load1 | Controls battery discharge to CookingLoad1 |
| SW_Load2 | Controls PV power delivery to CookingLoad2 |
| CookingLoad1 | Resistive heating element — battery fed, constant power draw |
| CookingLoad2 | Resistive heating element — PV fed, impedance matched to PV MPP |
| Microcontroller | Runs MPPT algorithm + switch arbitration logic |

### Physical cooking vessel

The cooking vessel is a **modified rice cooker** — selected because:
- It contains a pre-built resistive heating element of known resistance
- It is thermally insulated by design (inner pot, sealed lid, thermal mass)
- The internal thermostat is bypassed and replaced by the microcontroller switch logic
- It is locally available and affordable in Madagascar

---

## 4. Design Rationale

### Why a synchronous buck converter instead of a buck-boost?

The original concept considered a buck-boost converter to handle both stepping up (for cooking from PV) and stepping down (for battery charging). This was abandoned for the following reasons:

| Criterion | Buck-Boost | Synchronous Buck (chosen) |
|---|---|---|
| Switch count | 4 (non-inverting) | 2 |
| Output polarity | Inverted (standard) or complex (non-inverting) | Same as input |
| Control complexity | Mode transition management required | Single control loop |
| Efficiency | 85–92% typical | 93–97% typical |
| Local repairability | Difficult | Straightforward |
| Cost | Higher | Lower |

The key insight is that the buck-boost complexity is unnecessary if the **loads are sized correctly**. By ensuring `R_load2 < R_mpp`, the PV voltage at MPP is always higher than the load voltage — meaning a buck converter can always step down to the required level. No boost stage is ever needed.

### Why resistive loads instead of an induction cooker?

| Criterion | Induction cooker | Resistive element (chosen) |
|---|---|---|
| Input requirement | AC (requires inverter) | DC (direct from converter) |
| Efficiency | ~85% | ~95% (direct heating) |
| Complexity | High (inverter + control) | Low (switch + element) |
| Cost | High | Low |
| Local repair | Difficult | Simple |

### Why modular architecture?

The modular design is both a **practical constraint** (hardware is acquired progressively) and a **deliberate design choice** for the target context. A household can begin with a 50 W system for water heating and expand to 150 W for full meal cooking as budget allows — without replacing any existing components.

---

## 5. System Logic & Control

### Switch arbitration state machine

```
State 1 — CHARGE:
  Condition: Irradiance sufficient AND battery SoC < 100%
  Actions:   SW_PV = ON, SW_Load1 = OFF, SW_Load2 = OFF
  Buck runs MPPT → charges battery

State 2 — COOK_PV (daytime cooking):
  Condition: Irradiance sufficient AND cooking session active
  Actions:   SW_PV = ON, SW_Load1 = OFF, SW_Load2 = ON
  Buck runs MPPT → powers CookingLoad2 directly

State 3 — COOK_BATTERY (evening/night cooking):
  Condition: Irradiance insufficient AND cooking session active AND SoC > SoC_min
  Actions:   SW_PV = OFF, SW_Load1 = ON, SW_Load2 = OFF
  Battery discharges through CookingLoad1

State 4 — COMBINED BOOST (peak cooking):
  Condition: Irradiance sufficient AND cooking session active AND boost enabled
  Actions:   SW_PV = ON, SW_Load1 = ON, SW_Load2 = ON
  PV + Battery discharge simultaneously → maximum cooking power

State 5 — IDLE:
  Condition: No irradiance AND no cooking session AND SoC < SoC_min
  Actions:   All switches OFF
  System protected from deep discharge
```

### MPPT algorithm

The project uses **Perturb & Observe (P&O)** for its simplicity and proven performance in low-cost microcontroller implementations. The algorithm:

1. Measures `V_pv` and `I_pv`
2. Computes `P_pv = V_pv × I_pv`
3. Compares with previous power `P_prev`
4. Perturbs duty cycle `D` by `ΔD` in the direction that increases power
5. Repeats at a fixed sampling interval

The sampling interval is chosen to be significantly longer than the converter's settling time to avoid false perturbations.

### Cooking session definition

Sessions are defined by:
- Start hour and end hour
- Target power (W)
- Boost flag (PV + battery combined or PV only)

The microcontroller checks the current time against the session schedule and activates the appropriate state.

---

## 6. Key Design Equations

### PV model — irradiance curve

Irradiance is modeled as a sine wave:

```
G(h) = G_peak × sin(π × (h − h_sunrise) / (h_sunset − h_sunrise))
       normalized so peak lands at solar noon
```

PV power output:

```
P_pv(h) = G(h) / G_peak × P_wp
```

### Buck converter — voltage conversion

```
V_out = D × V_in

where D = duty cycle (0 < D < 1)
```

### Buck converter — inductor sizing

```
L = (V_in − V_out) × D / (f_sw × ΔI_L)

where:
  f_sw  = switching frequency
  ΔI_L  = acceptable inductor current ripple (typically 20–40% of I_out)
```

### Buck converter — input impedance

```
R_in = R_load / D²
```

This is the impedance the PV source sees. The MPPT algorithm adjusts D to match `R_in = R_mpp`.

### Load sizing — CookingLoad2 (PV-fed, impedance constraint)

For the buck converter to operate without requiring a boost stage:

```
R_load2 < R_mpp = V_mpp / I_mpp = V_mpp² / P_mpp
```

### Load sizing — CookingLoad1 (battery-fed, constant power)

```
R_load1 = V_bat² / P_load1
```

### Water boiling capacity

Energy available per cooking session:

```
E_session = P_cook × t_session   [Wh]
```

Theoretical water mass that can be brought to boiling point from ambient temperature T₀:

```
m = E_session × 3600 / (c_p × (100 − T₀))

where c_p = 4186 J/kg·°C
```

Practical estimate accounting for heat losses:

```
m_practical = m × (1 − η_loss)
```

---

## 7. Simulation Results

The parametric simulation was developed as an interactive tool and covers the full daily energy budget. Key results for the baseline configuration (120 Wp panel, 12V/20Ah battery, 50% initial SoC, 3 cooking sessions at 70 W for 2 hours each) are summarized below.

### Baseline configuration

| Parameter | Value |
|---|---|
| PV peak power | 120 Wp |
| Irradiance peak | 1000 W/m² at 12:00 |
| Sunrise / Sunset | 06:00 / 18:00 |
| Battery | 12V, 20 Ah (240 Wh) |
| Initial SoC | 50% (120 Wh) |
| Converter efficiency | 90% |
| Cooking power | 70 W |
| Sessions | 06:00–08:00, 11:00–13:00, 18:00–20:00 |

### Daily energy budget

| Metric | Value |
|---|---|
| Total PV energy generated | ~432 Wh |
| Total energy delivered to cooking | ~420 Wh |
| Total energy stored in battery | ~85 Wh |
| Final battery SoC | ~62% |

### Session-level results

| Session | Time | PV available | Source | Water (theoretical) | Water (practical, −25% loss) |
|---|---|---|---|---|---|
| Morning | 06:00–08:00 | Low (~10–60 W) | PV + battery | ~1.72 L | ~1.29 L |
| Noon | 11:00–13:00 | High (~100–120 W) | PV dominant | ~1.72 L | ~1.29 L |
| Evening | 18:00–20:00 | None | Battery only | ~1.72 L | ~1.29 L |

### Key observations from simulation

1. **Morning session is the most battery-intensive** — PV contributes less than 40% of cooking power at 06:00, the battery covers the rest. A minimum initial SoC of ~40% is required to complete the session without depletion.

2. **Noon session is the most efficient** — PV alone can cover the full 70 W cooking load near solar noon with a 120 Wp panel. Battery draw is near zero. This is the ideal session for high-power cooking.

3. **Evening session depends entirely on daytime charging** — 2 hours at 70 W requires 140 Wh from the battery. With a 240 Wh battery at 50% initial SoC (120 Wh available), the evening session is marginally feasible. A 100% initial SoC or a larger battery is recommended for reliable evening cooking.

4. **Combined boost mode** (PV + battery simultaneously) at the noon session can deliver up to ~200 W to the cooking load, reducing boiling time by approximately 65% compared to the 70 W baseline for the same energy budget.

5. **The `R_load < R_mpp` constraint** is satisfied for a 70 W resistive load on a 120 Wp panel with Vmp ≈ 18V, Imp ≈ 6.67A, giving R_mpp ≈ 2.7 Ω. A 70 W load at 12V output has R_load = 12²/70 ≈ 2.06 Ω < 2.7 Ω. The constraint holds across the full 50–120 Wp range with appropriate duty cycle adjustment.

---

## 8. Project Phases

### Phase 1 — Synchronous buck converter design and bench validation

**Objective:** Design, build, and validate the synchronous buck converter across the 50–150 W power range using a DC bench supply, before any PV or generator hardware is available.

**Resources required:** DC bench power supply or DC-DC adapter, oscilloscope, multimeter, electronic components

**Tasks:**
- [ ] Compute input voltage range from expected PV Vmp (target: 15–25V input)
- [ ] Size inductor, capacitor, MOSFETs, gate driver for 150W worst-case
- [ ] Select microcontroller and PWM configuration
- [ ] Build converter on perfboard or PCB
- [ ] Open-loop validation: verify `V_out = D × V_in` across duty cycle range
- [ ] Efficiency measurement at 50W, 100W, 150W operating points
- [ ] Switching waveform verification (oscilloscope): check ringing, deadtime, gate signals
- [ ] Thermal test: sustained operation at target power, verify component temperatures
- [ ] Load step response test: verify stability when load switches suddenly
- [ ] Document all measured values vs simulation predictions

**Deliverable:** Validated synchronous buck converter with measured efficiency curve, switching waveforms, and component datasheet compliance report.

---

### Phase 2 — Cooking vessel characterization and modification

**Objective:** Select, modify, and thermally characterize a commercially available rice cooker as the cooking vessel for the VARY system.

**Resources required:** Rice cooker, multimeter, thermometer, timer, water, scale

**Tasks:**
- [ ] Disassemble rice cooker and identify heating element terminals
- [ ] Measure heating element resistance at room temperature and estimate operating resistance
- [ ] Verify `R_element < R_mpp` compatibility with Phase 1 converter
- [ ] Bypass internal thermostat — replace with external switch terminals
- [ ] Reassemble and verify mechanical integrity
- [ ] Thermal characterization experiment:
  - Heat known mass of water to 100°C, measure time and energy
  - Allow to cool, measure cooling rate → derive heat loss coefficient
  - Repeat at different ambient temperatures if possible
- [ ] Compare measured boiling capacity vs simulation prediction
- [ ] Document modified vessel thermal model (loss coefficient, effective insulation)

**Deliverable:** Modified rice cooker with characterized thermal model, measured vs predicted boiling capacity comparison, confirmed `R_load < R_mpp` constraint satisfaction.

---

### Phase 3 — MPPT integration and full system validation

**Objective:** Integrate the MPPT algorithm and switch arbitration logic, connect the full system with PV panel and battery, and validate cooking performance under real operating conditions.

**Resources required:** PV panel (50–120 Wp generator or real panel), 12V battery, Phase 1 converter, Phase 2 vessel, microcontroller

**Tasks:**
- [ ] Implement P&O MPPT algorithm on microcontroller
- [ ] Implement switch arbitration state machine (5 states as defined above)
- [ ] Bench test MPPT with PV simulator or generator: verify tracking accuracy
- [ ] Measure MPPT efficiency (tracked power / theoretical MPP power)
- [ ] Integrate full system: PV → converter → battery → switches → loads
- [ ] Closed-loop cooking experiment 1: morning session (low irradiance, battery assist)
- [ ] Closed-loop cooking experiment 2: noon session (PV dominant)
- [ ] Closed-loop cooking experiment 3: evening session (battery only)
- [ ] Combined boost mode experiment: measure peak power delivery and boiling time
- [ ] Compare all experimental results against simulation predictions
- [ ] Modular scaling test: repeat key experiments at 50W and 120W operating points
- [ ] Document discrepancies and identify sources of error

**Deliverable:** Full experimental validation report with measured vs simulated results across all operating modes, boiling experiments with real water, efficiency and performance characterization across the 50–150W modular range.

---

## 9. Repository Structure

```
VARY/
│
├── README.md                    ← This file
│
├── simulation/
│   ├── simulator.html           ← Interactive parametric simulator (browser-based)
│   └── results/
│       ├── baseline_120Wp.json  ← Simulation output, baseline config
│       └── scenarios/           ← Various sizing scenarios
│
├── design/
│   ├── converter/
│   │   ├── schematic.pdf        ← Buck converter schematic
│   │   ├── bom.csv              ← Bill of materials
│   │   └── calculations.pdf     ← Inductor, capacitor, MOSFET sizing
│   ├── control/
│   │   ├── mppt_po.c            ← P&O MPPT algorithm (microcontroller)
│   │   └── switch_logic.c       ← Switch arbitration state machine
│   └── load/
│       └── load_sizing.pdf      ← R_load1 and R_load2 design calculations
│
├── experiments/
│   ├── phase1/                  ← Bench validation results
│   ├── phase2/                  ← Thermal characterization results
│   └── phase3/                  ← Full system validation results
│
└── thesis/
    ├── chapters/                ← Thesis chapter drafts
    └── figures/                 ← Figures and plots
```

---

## 10. References

1. Mohanty, K.B. et al. (2017). *Selection criteria of DC-DC converter and control variable for MPPT of PV system utilized in heating and cooking applications.* Cogent Engineering.

2. Kabala, C. (2017). *Application of distributed DC/DC electronics in photovoltaic systems.* Colorado State University Master's Thesis.

3. Phipps, M.C. (2025). *Implementation of a three-port solar converter with a battery energy storage system.* University of Arkansas.

4. Atmane, I. et al. (2021). *DC-DC converters for innovative autonomous solar cooker.* ScienceDirect.

5. Sibiya, S. & Venugopal, N. (2017). *Solar powered induction cooking system.* ScienceDirect.

6. MDPI (2020). *Solar home systems for clean cooking: a cost-health benefit analysis.* Sustainability, 12(9), 3909.

---

## License

This project is developed as part of a master's thesis. All simulation code and design documents are open for academic use with attribution.

---

*VARY — because the best technology is the one that puts food on the table.*
