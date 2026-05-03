# Buck Converter Design and Component Selection

## Introduction

This chapter presents the design and preliminary analysis of a DC-DC buck converter intended for a low-power photovoltaic application. The objective of this phase is to establish a robust component sizing methodology based on analytical models and to determine suitable hardware parameters prior to dynamic simulation and experimental validation.

The design focuses on ensuring stable operation under variable input conditions while maintaining strict constraints on current and voltage ripple.

---

## System Specifications

The system under study consists of a photovoltaic source supplying a regulated DC load through a buck converter. The main design parameters are summarized in Table 1.

**Table 1: System specifications**

| Parameter | Value |
| :--- | :--- |
| Nominal input voltage $V_{in,nom}$ | 18 V |
| Maximum input voltage $V_{in,max}$ | 25 V |
| Output voltage $V_{out}$ | 12 V |
| Maximum output power $P_{max}$ | 50 W |
| Switching frequency $f_{sw}$ | 50 kHz |
| Inductor current ripple | 30% |
| Output voltage ripple | 1% |

These specifications are representative of small-scale standalone photovoltaic systems such as battery charging or low-power DC loads.

---

## Buck Converter Operating Principle

The buck converter is a step-down DC-DC converter that regulates the output voltage by adjusting the duty cycle of a switching element.

Under ideal conditions, the voltage conversion ratio is given by:

$$V_{out} = D \cdot V_{in}$$

where $D$ is the duty cycle.

At nominal input voltage:

$$D = \frac{V_{out}}{V_{in}} = \frac{12}{18} \approx 0.67$$

The duty cycle varies with input voltage, decreasing as $V_{in}$ increases.

---

## Design Methodology

### Output Current

The maximum output current is derived from the power requirement:

$$I_{out,max} = \frac{P_{max}}{V_{out}} = \frac{50}{12} \approx 4.17 \, \text{A}$$

---

### Inductor Design

The inductor is sized to limit the current ripple to a predefined fraction of the output current.

$$\Delta I_L = 0.3 \cdot I_{out,max}$$

The inductance value is determined using:

$$\Delta I_L = \frac{(V_{in} - V_{out}) \cdot D}{L \cdot f_{sw}}$$

This ensures operation in continuous conduction mode and limits stress on the switching devices.

---

### Output Capacitor Design

The output capacitor is selected to satisfy the voltage ripple constraint:

$$\Delta V_{out} = 0.01 \cdot V_{out} = 0.12 \, \text{V}$$

The required capacitance is estimated using:

$$\Delta V_{out} = \frac{\Delta I_L}{8 \cdot f_{sw} \cdot C}$$

---

### MOSFET Selection

The MOSFET must satisfy both voltage and conduction requirements:

* Voltage rating: $V_{DS} > V_{in,max} = 25$ V
* Conduction losses:
    $$P_{cond} = I_{out}^2 \cdot R_{DS(on)}$$

A low $R_{DS(on)}$ device is selected to minimize losses and improve efficiency.

---

## Design Exploration

Two configurations were evaluated to assess the trade-off between component size and electrical performance.

---

### Minimum Configuration

**Table 2: Minimum configuration**

| Parameter | Value |
| :--- | :--- |
| Duty cycle | 66.7% |
| Inductance | 64 $\mu$H |
| Output capacitor | 26 $\mu$F |
| Input capacitor | 21 $\mu$F |
| MOSFET | IRF3205 |

This configuration prioritizes reduced component size while satisfying the design constraints. However, lower capacitance values may result in increased voltage ripple and reduced robustness.

---

### Optimized Configuration

**Table 3: Optimized configuration**

| Parameter | Value |
| :--- | :--- |
| Duty cycle | 66.7% |
| Inductance | 26.7 $\mu$H |
| Output capacitor | 63 $\mu$F |
| Input capacitor | 21--49 $\mu$F |
| MOSFET | IRF3205D |

This configuration reduces inductance and increases capacitance, improving transient response and voltage stability. However, the reduced inductance leads to higher current ripple and increased stress on switching devices.

---

## Final Component Selection

Based on the trade-off between robustness and performance, the final component values are presented in Table 4.

**Table 4: Final selected configuration**

| Parameter | Value |
| :--- | :--- |
| Duty cycle | 66.7% |
| Inductance | 64 $\mu$H |
| Output capacitor | 63 $\mu$F |
| Input capacitor | 49 $\mu$F |
| MOSFET | IRF3205 |

---

## Discussion

The selected design prioritizes robustness, which is essential for photovoltaic systems subject to variable input conditions.

The higher inductance value ensures reduced current ripple and stable continuous conduction mode operation. The increased output capacitance maintains voltage ripple within the specified limit, while the larger input capacitor improves input voltage stability.

Although the optimized configuration offers reduced inductance and potentially faster response, it introduces higher ripple and increased stress on components. Therefore, the selected configuration represents a conservative and reliable design suitable for subsequent simulation and validation.

---

## Conclusion

This chapter presented the analytical design and component selection of a buck converter for a photovoltaic application. A parametric approach was used to evaluate multiple configurations and identify a suitable trade-off between performance and robustness.

The resulting design provides a solid foundation for the next phase, which will focus on dynamic simulation and experimental validation of the system.
