We buillt a controller circuit that controls a single-phase full bridge inverter circuit with LC filter. The LC filter is used to get a purely sinusoidal wave. Our main motive here is to set up a controller circuit that controls the rms value of the output voltage of an inverter circuit. The project is done in Matlab Simulink. 

# Single-Phase H-Bridge Inverter with PI Voltage Control

> A MATLAB/Simulink implementation of a closed-loop single-phase inverter that converts DC to a regulated AC sine wave using Sinusoidal PWM (SPWM) and PI-based feedback control.

## 📐 System Overview

```
DC Source → H-Bridge (BJTs) → LC Filter → AC Load
               ↑
         PWM Gate Signals
               ↑
     SPWM Comparators (×2)
               ↑
     PI(s) × sin(ωt)  ←  Error
                             ↑
                  Vref − Vout_RMS
```

---

## 🔧 Three Main Stages

### 1. PWM Generation (Left Zone)
- Two **`>`** relational comparator blocks perform **Sinusoidal PWM (SPWM)**
- Each comparator compares a **sine wave** (modulating signal) against a **triangular carrier wave**
- Output logic:
  ```
  sine > carrier  →  Output = 1 (switch ON)
  sine < carrier  →  Output = 0 (switch OFF)
  ```
- **NOT gates** generate complementary signals with a small **dead-time delay** to prevent shoot-through
- Gate signal mapping:

  | Comparator | Direct Output | Complemented Output |
  |---|---|---|
  | Top | S1 | S4 |
  | Bottom | S3 | S2 |

---

### 2. Power Stage — H-Bridge (Middle Zone)

- Four **BJT switches** form a full H-bridge
- Two comparators are needed because the H-bridge has **two independent legs**
- AC output is produced by alternating diagonal switch pairs:

  | Half Cycle | Active Switches | Current Direction |
  |---|---|---|
  | Positive | S1 + S2 | Left → Load → Right |
  | Negative | S3 + S4 | Right → Load → Left |

- **LC Low-Pass Filter** converts the PWM output into a clean sine wave:
  ```
  H-bridge PWM  →  [L]  →  [C]  →  Sine Wave Output
  ```
  - **Inductor**: blocks high-frequency switching harmonics
  - **Capacitor**: smooths residual ripple to ground
  - Cutoff frequency: `f_cutoff = 1 / (2π√LC)  ≪  f_switching`

---

### 3. Closed-Loop PI Feedback (Bottom-Right Zone)

- Measures output **RMS voltage** and compares it to a **reference voltage (Vref)**
- Error signal: `e = Vref_RMS − Vout_RMS`
- PI Controller formula:

$$PI(s) = K_p + \frac{K_i}{s}$$

| Term | Action |
|---|---|
| **Proportional (Kp · e)** | Reacts to current error instantly |
| **Integral (Ki · ∫e dt)** | Eliminates steady-state error over time |

- PI output is multiplied by sine waves to **scale the modulating signal amplitude**:
  ```
  PI_output × +sin(ωt)  →  Top comparator    (drives S1/S4)
  PI_output × −sin(ωt)  →  Bottom comparator (drives S3/S4)
  ```
- Two multipliers with **opposing sine waves (+sin and −sin)** are required for bipolar SPWM

---

## 🛠 Supporting Blocks

| Block | Role |
|---|---|
| **MUX** | Bundles S1–S4 into one scope (`Gate Pulse`) for visualization only — no effect on control |
| **NOT Gates** | Generate complementary switch signals with dead-time to prevent DC bus short |
| **RMS Block** | Measures true RMS of output voltage for feedback |
| **Two Sine Sources** | 180° phase-shifted to drive opposing H-bridge legs |

---

## 📊 How PWM Becomes a Sine Wave

The LC filter then removes the high-frequency carrier, leaving only the 50/60 Hz sine.

