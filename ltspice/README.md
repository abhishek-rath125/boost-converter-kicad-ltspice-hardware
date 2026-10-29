# Boost Converter LTspice Simulation Models

This directory contains production-grade LTspice XVII / LTspice 24 simulation files modeling the hardware Boost DC-DC converter designed by **Abhishek Rathour** (NIT Goa).

---

## 📁 Files in This Directory

| File | Description | Switching Frequency | Gate Control |
|---|---|---|---|
| [`boost_converter_open_loop.asc`](./boost_converter_open_loop.asc) | Benchmark open-loop boost stage with parameterized duty cycle ($D$) and stepped load ($R_{load}$) for CCM/DCM analysis. | 20 kHz | Parameterized PULSE source ($0-12\,\text{V}$) |
| [`boost_converter_555_pwm.asc`](./boost_converter_555_pwm.asc) | Complete discrete gate drive circuit powered by an NE555 astable PWM generator, driving the IRFZ44N through a $47\,\Omega$ gate resistor and $10\,\text{k}\Omega$ pull-down. | ~20 kHz | NE555 Astable Multivibrator |

---

## ⚡ Circuit Specifications & Component Models

| Parameter | Symbol | Value | Hardware Component / SPICE Model |
|---|---|---|---|
| Input Voltage | $V_{in}$ | 12.0 V | Low-impedance lab DC bench supply |
| Output Voltage | $V_{out}$ | 13.0 V – 24.0 V | Determined by duty cycle $D$ |
| Switching Frequency | $f_{sw}$ | 20 kHz ($T_{sw} = 50\,\mu\text{s}$) | PWM generator |
| Inductor | $L_1$ | $319\,\mu\text{H}$ | Iron-powder toroidal core ($R_{ser} \approx 30\,\text{m}\Omega$) |
| Power Switch | $M_1$ | IRFZ44N | N-MOSFET ($55\,\text{V},\,49\,\text{A},\,R_{DS(on)}=17.5\,\text{m}\Omega$) |
| Freewheeling Diode | $D_1$ | MBRF20100SCT | Schottky barrier ($100\,\text{V},\,20\,\text{A},\,V_F \approx 0.55\,\text{V}$) |
| Output Capacitance | $C_{out}$ | $220\,\mu\text{F} \parallel 3\,\mu\text{F}$ | Low-ESR electrolytic ($45\,\text{m}\Omega$) $\parallel$ film bypass ($5\,\text{m}\Omega$) |
| Gate Resistor | $R_G$ | $47\,\Omega$ | Damping gate ringing & limiting peak drive current |
| Gate Pull-down | $R_{PD}$ | $10\,\text{k}\Omega$ | Assures gate is held LOW during power-up |
| Load Resistance | $R_{load}$ | $100\,\Omega$ / $200\,\Omega$ | High-power ceramic wirewound resistors |

---

## 🔬 Theoretical Analysis & Governing Equations

### 1. Continuous Conduction Mode (CCM)
In steady-state CCM, applying volt-second balance across inductor $L_1$:

$$V_{in} \cdot D \cdot T_{sw} + (V_{in} - V_{out}) \cdot (1 - D) \cdot T_{sw} = 0$$

$$\implies V_{out} = \frac{V_{in}}{1 - D}$$

For $V_{in} = 12\,\text{V}$ and $D = 0.40$:
$$V_{out,ideal} = \frac{12}{1 - 0.40} = 20.0\,\text{V}$$
Accounting for the Schottky diode forward drop ($V_D \approx 0.55\,\text{V}$) and MOSFET conduction loss ($I_{rms}^2 R_{DS(on)}$):
$$V_{out,real} \approx \frac{V_{in} - D \cdot I_L R_{DS(on)}}{1 - D} - V_D \approx 19.3\,\text{V}$$

### 2. Inductor Current Ripple ($\Delta I_L$)
During the ON interval ($0 \le t \le D \cdot T_{sw}$):

$$\Delta I_L = \frac{V_{in} \cdot D}{f_{sw} \cdot L_1} = \frac{12\,\text{V} \times 0.40}{20\times 10^3\,\text{Hz} \times 319\times 10^{-6}\,\text{H}} = \frac{4.8}{6.38} \approx 0.752\,\text{A}$$

Peak inductor current:
$$I_{L,pk} = I_{L,avg} + \frac{\Delta I_L}{2}$$
For $R_{load} = 100\,\Omega$, $I_{out} = \frac{20\,\text{V}}{100\,\Omega} = 0.20\,\text{A}$.
$$I_{L,avg} = \frac{I_{out}}{1 - D} = \frac{0.20}{0.60} \approx 0.333\,\text{A}$$
$$I_{L,min} = 0.333 - \frac{0.752}{2} = -0.043\,\text{A} \implies \text{Near boundary between CCM and DCM!}$$

### 3. Boundary Conduction Mode (BCM) & DCM Operation
The critical inductance $L_{crit}$ for maintaining CCM is given by:

$$L_{crit} = \frac{R_{load} \cdot D (1 - D)^2}{2 \cdot f_{sw}}$$

- **At $R_{load} = 100\,\Omega$:**
  $$L_{crit} = \frac{100 \times 0.4 \times (0.6)^2}{2 \times 20\,000} = \frac{14.4}{40\,000} = 360\,\mu\text{H}$$
  Since $L = 319\,\mu\text{H} \approx L_{crit}$, the converter operates right at the threshold of CCM/DCM, with slight continuous current.
- **At $R_{load} = 200\,\Omega$:**
  $$L_{crit} = 720\,\mu\text{H} > 319\,\mu\text{H}$$
  The inductor current discharges to zero before the next switching period begins, putting the stage firmly in **Discontinuous Conduction Mode (DCM)**.
  In DCM, $V_{out}$ rises above $\frac{V_{in}}{1-D}$ according to:
  $$V_{out,DCM} = V_{in} \left( \frac{1 + \sqrt{1 + \frac{2 D^2 R_{load}}{f_{sw} L}}}{2} \right)$$
  This perfectly explains why the open-loop output voltage depends heavily on load resistance!

---

## 🚀 How to Run the Simulations

1. **Install LTspice**: Download and install [LTspice](https://www.analog.com/en/resources/design-tools-and-calculators/ltspice-simulator.html).
2. **Open Schematic**:
   - Double-click `boost_converter_open_loop.asc` or open it from within LTspice (`File -> Open`).
3. **Run Simulation**:
   - Click the **Run** button (Running Man icon) or press `Alt + S`.
   - The simulation will simulate $10\,\text{ms}$ of converter startup and settle into steady state, saving the last $2\,\text{ms}$ ($8\,\text{ms} \to 10\,\text{ms}$).
4. **Inspect Waveforms**:
   - Probe the `V(vout)` node: Observe the boosted DC output voltage and peak-to-peak ripple.
   - Probe `I(L1)`: Observe the triangular inductor current waveform showing CCM/DCM behavior.
   - Probe `V(sw)`: Observe the drain-source switching node waveform ($0\,\text{V} \leftrightarrow V_{out} + V_{diode}$).
   - Probe `V(gate)`: Verify gate switching transitions and sharp edges shaped by $R_G = 47\,\Omega$.
5. **View Automated Measurements (`.meas`)**:
   - Press `Ctrl + L` to open the **Spice Error Log**.
   - You will see measured values automatically computed for:
     - `Vout_avg`: Average steady-state output voltage.
     - `Vout_ripple`: Peak-to-peak output ripple voltage.
     - `IL_pk`, `IL_min`, `IL_ripple`: Inductor peak, minimum, and ripple currents.
     - `Pin_avg`, `Pout_avg`: Input and output real power.
     - `Efficiency`: Converter power conversion efficiency percentage ($\approx 91\%$).

---

## 👤 Author & Authorizations
- **Author**: Abhishek Rathour
- **Institution**: National Institute of Technology Goa (NIT Goa)
- **Department**: Electrical and Electronics Engineering (EEE)
- **GitHub**: [@abhishek-rath125](https://github.com/abhishek-rath125)
