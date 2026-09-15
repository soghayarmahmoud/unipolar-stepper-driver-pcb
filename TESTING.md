# Hardware Testing and Validation Protocol

**Revision:** 1.0  
**Scope:** Unipolar Stepper Motor Driver PCB (Prototype v1.0)  
**Last Updated:** 2026-09-16

---

## Table of Contents

1. [Purpose and Safety](#1-purpose-and-safety)
2. [Required Test Equipment](#2-required-test-equipment)
3. [Stage 1 — Visual and Continuity Inspection](#3-stage-1--visual-and-continuity-inspection)
4. [Stage 2 — Power Rail Validation (Multimeter)](#4-stage-2--power-rail-validation-multimeter)
5. [Stage 3 — NE555 Clock Generation (Oscilloscope)](#5-stage-3--ne555-clock-generation-oscilloscope)
6. [Stage 4 — 74LS194 Shift-Register Logic](#6-stage-4--74ls194-shift-register-logic)
7. [Stage 5 — 7407 Buffer Stage](#7-stage-5--7407-buffer-stage)
8. [Stage 6 — TIP120 Power Stage (No Motor Load)](#8-stage-6--tip120-power-stage-no-motor-load)
9. [Stage 7 — Motor Connection and Full Functional Test](#9-stage-7--motor-connection-and-full-functional-test)
10. [Troubleshooting Reference](#10-troubleshooting-reference)
11. [Test Record Template](#11-test-record-template)

---

## 1. Purpose and Safety

This document defines a staged validation procedure for the manufactured Unipolar Stepper Motor Driver PCB. The protocol is designed to isolate faults in each functional block before connecting an expensive motor, preventing damage to the motor winding or the power transistors from an undetected logic or power-stage failure.

### Safety Precautions

- **Never connect the motor winding until Stages 1 through 6 have passed.** A shorted output or stuck-high logic line can destroy a motor winding or a **TIP120** transistor.
- Use a current-limited bench power supply for both `VCC` and `VMOT` rails during testing. Set the current limit to 200 mA for `VCC` and 500 mA for `VMOT` during no-load validation.
- Verify that all power is removed before changing jumper configurations, probing with meter leads in resistance mode, or connecting/disconnecting the motor.
- The **TIP120** devices dissipate significant power when switching inductive loads. Ensure heatsinks are installed before Stage 7.

---

## 2. Required Test Equipment

| Equipment | Minimum Specification | Used In Stage |
|---|---|---|
| Digital multimeter (DMM) | DC voltage accuracy ±0.5 %, continuity buzzer, diode test | 1, 2, 4, 6 |
| Oscilloscope | 2-channel, ≥20 MHz bandwidth, ≥100 MS/s | 3, 4, 5, 6 |
| Bench power supply (dual) | 0–15 V, current-limited, isolated outputs | 2–7 |
| Logic probe or LED logic tester | TTL-compatible, high/low/pulse indication | 4, 5 |
| Load resistor (dummy) | 10 Ω, 5 W (per channel) — substitutes for motor winding | 6 |
| Small flat-head screwdriver | For NE555 timing adjustment (if trimmer installed) | 3 |

---

## 3. Stage 1 — Visual and Continuity Inspection

**Power: OFF. Both supply rails disconnected.**

### 3.1 Visual Inspection

Examine the board under magnification for the following defects:

| Check | Acceptance Criterion |
|---|---|
| Solder joints | All through-hole pads have concave, shiny fillets; no cold joints, no solder bridges |
| Component polarity | **D1**–**D4** (1N4001) cathode bands face the `VMOT` side; electrolytic capacitors (if any) positive leads match silkscreen |
| IC orientation | Pin 1 of **U1** (NE555), **U2** (74LS194), **U3** (7407) aligns with the silkscreen notch or dot |
| **TIP120** placement | **Q1**–**Q4** metal tabs face the designated heatsink area; no pins bent under the package |
| Foreign debris | No solder balls, clipped leads, or flux residue bridging adjacent pads |
| Board outline | No cracks or delamination near the board edge or mounting holes |

### 3.2 Continuity Checks (DMM in Resistance / Continuity Mode)

With all ICs removed from their sockets (if socketed) or before first power-up, verify the following:

| Test Point 1 | Test Point 2 | Expected Resistance |
|---|---|---|
| `VCC` net (**U1** pin 14, **U2** pin 16, **U3** pin 14) | `GND` | > 1 MΩ (no short) |
| `VMOT` net (motor connector pin, **Q1**–**Q4** collectors) | `GND` | > 1 MΩ (no short) |
| `VCC` | `VMOT` | > 1 MΩ (rails isolated) |
| Each motor winding output (**Q1**–**Q4** emitters to connector) | `GND` | > 1 MΩ (no output short) |
| `GND` pour (bottom layer) | Any mounting hole | < 1 Ω (ground continuity) |

> If any resistance measurement reads below 100 Ω where an open circuit is expected, stop and locate the short before applying power. Common causes: solder bridge between power and ground pins, misplaced component, or internal PCB defect.

---

## 4. Stage 2 — Power Rail Validation (Multimeter)

**Power: `VCC` only. `VMOT` disconnected. Motor disconnected.**

### 4.1 VCC Rail Bring-Up

1. Set the bench supply to 5.0 V with a 200 mA current limit.
2. Connect the positive lead to the `VCC` input pin and the negative lead to `GND`.
3. Enable the supply. Observe the current reading.

| Parameter | Acceptance Criterion |
|---|---|
| Supply current | < 50 mA (quiescent current of three ICs + pull-up resistors) |
| `VCC` voltage at **U1** pin 14 | 4.75 V – 5.25 V (TTL Vcc minimum) |
| `VCC` voltage at **U2** pin 16 | 4.75 V – 5.25 V |
| `VCC` voltage at **U3** pin 14 | 4.75 V – 5.25 V |
| Decoupling capacitor voltage (100 nF at each IC) | Matches `VCC` within 50 mV |

### 4.2 VMOT Rail Check (No Load)

1. Keep `VCC` powered.
2. Set a second bench supply to the intended motor voltage (start at 5 V for initial validation; rated maximum 12 V).
3. Set current limit to 100 mA.
4. Connect to the `VMOT` input pin.

| Parameter | Acceptance Criterion |
|---|---|
| `VMOT` supply current | < 10 mA (no load connected; only leakage through flyback diodes) |
| `VMOT` voltage at motor connector | Matches supply setting within 2 % |
| `VMOT` to `VCC` isolation | No voltage coupling (measure `VCC` remains stable) |

> If the current limit trips immediately on either rail, power off and return to Stage 1 to locate the short.

---

## 5. Stage 3 — NE555 Clock Generation (Oscilloscope)

**Power: `VCC` + `VMOT` (VMOT at 5 V for safety). Motor disconnected.**

### 5.1 Clock Output Measurement

1. Connect oscilloscope Channel 1 probe (1× or 10×) to **U1** pin 3 (OUTPUT).
2. Connect the probe ground lead to a nearby `GND` point.
3. Set the scope to 2 V/div vertical, 1 ms/div horizontal (adjust as needed).

| Measurement | Expected Value | Tolerance |
|---|---|---|
| Waveform shape | Square wave (near 50 % duty cycle) | — |
| Logic HIGH level | ≥ 2.4 V (TTL V_OH) | — |
| Logic LOW level | ≤ 0.4 V (TTL V_OL) | — |
| Rise time | < 200 ns | — |
| Fall time | < 200 ns | — |
| Frequency | f = 1 / (0.693 × C1 × (R1 + 2×R2)) | ±10 % (component tolerance) |
| Duty cycle | > 50 % (astable configuration inherent) | — |

### 5.2 Timing Capacitor Waveform (Optional)

Connect Channel 2 to **U1** pin 6/2 (THRESHOLD/TRIGGER, tied together in astable mode).

| Measurement | Expected |
|---|---|
| Waveform | Exponential ramp between ~1/3 Vcc and ~2/3 Vcc |
| Upper threshold | ~3.33 V (at Vcc = 5 V) |
| Lower threshold | ~1.67 V (at Vcc = 5 V) |

> If pin 3 is stuck HIGH or LOW, check **U1** pin orientation, the R1/R2/C1 soldering, and the RESET pin (pin 4) connection to `VCC`. A floating RESET pin will cause unpredictable output.

---

## 6. Stage 4 — 74LS194 Shift-Register Logic

**Power: `VCC` + `VMOT` at 5 V. Motor disconnected. TIP120 base resistors may remain connected.**

### 6.1 Power-Up Default State

1. With the NE555 clock running (verified in Stage 3), observe the four parallel outputs of **U2** (pins 15 = QA, 1 = QB, 10 = QC, 9 = QD) using a logic probe or oscilloscope.

| Condition | Expected Behavior |
|---|---|
| Power-up (S0 = HIGH, S1 = LOW for right-shift mode) | One output HIGH, cycling through QA → QB → QC → QD → QA |
| Clock present at pin 11 | Outputs advance on each rising clock edge |
| DIR jumper set to reverse (S0 = LOW, S1 = HIGH) | Sequence reverses: QA → QD → QC → QB → QA |
| Both S0 and S1 HIGH (parallel load mode) | Outputs load the values on pins 3–6 (A–D) on next clock |

### 6.2 Four-Phase Sequence Verification

Use a two-channel oscilloscope or logic analyzer to capture at least two consecutive output pairs:

| Test | Expected Pattern (one-hot) |
|---|---|
| QA (pin 15) vs QB (pin 1) | HIGH pulses are non-overlapping, separated by one clock period |
| QB (pin 1) vs QC (pin 10) | Non-overlapping, sequential |
| QC (pin 10) vs QD (pin 9) | Non-overlapping, sequential |
| Full cycle (QA → QB → QC → QD) | Exactly one output HIGH at any instant; period = 4 × clock period |

> The 74LS194 requires a valid logic-level clock. The NE555 output directly drives pin 11. If the outputs do not advance, verify the clock reaches pin 11 with a scope (it may be shorted to ground or disconnected by a cold solder joint).

### 6.3 Direction Control Test

1. Locate the DIR jumper or switch controlling the S0/S1 mode pins of **U2**.
2. Toggle the direction while observing the output sequence.

| DIR Setting | S0 | S1 | Mode | Expected Sequence |
|---|---|---|---|---|
| Forward | HIGH | LOW | Shift right | QA → QB → QC → QD |
| Reverse | LOW | HIGH | Shift left | QA → QD → QC → QB |
| Hold (if implemented) | LOW | LOW | Hold | Outputs frozen |

> If direction reversal does not work, check the S0/S1 pin connections and the DIR jumper soldering. Both pins must never be left floating — tie unused mode pins to defined logic levels.

---

## 7. Stage 5 — 7407 Buffer Stage

**Power: `VCC` + `VMOT` at 5 V. Motor disconnected.**

The **7407** (U3) is a hex open-collector non-inverting buffer. Each 74LS194 output drives one 7407 input, and each 7407 output pulls down the corresponding **TIP120** base resistor network.

### 7.1 Input-Output Correspondence

| 74LS194 Output | 7407 Input Pin | 7407 Output Pin | Drives |
|---|---|---|---|
| QA (pin 15) | **U3** pin 1 | **U3** pin 2 | **Q1** base |
| QB (pin 1) | **U3** pin 3 | **U3** pin 4 | **Q2** base |
| QC (pin 10) | **U3** pin 5 | **U3** pin 6 | **Q3** base |
| QD (pin 9) | **U3** pin 9 | **U3** pin 8 | **Q4** base |

### 7.2 Open-Collector Behavior

1. Connect an oscilloscope probe to a 7407 output pin (e.g., **U3** pin 2).
2. Connect the probe ground to `GND`.

| Input (74LS194 output) | Expected 7407 Output |
|---|---|
| HIGH (≥ 2.4 V) | LOW (≤ 0.4 V) — output transistor ON, pulling to ground |
| LOW (≤ 0.4 V) | High-impedance (floating, pulled up by the base resistor network to `VCC` or `VMOT` side) |

> Because the 7407 is open-collector, the HIGH state is not actively driven. Measure the voltage at the 7407 output with respect to ground: when the corresponding 74LS194 output is LOW, the 7407 output should read near the pull-up voltage (typically `VCC` through the base resistor). If it reads 0 V, the output transistor may be shorted or the pull-up resistor may be missing.

---

## 8. Stage 6 — TIP120 Power Stage (No Motor Load)

**Power: `VCC` + `VMOT` at 5 V. Motor disconnected. Connect a 10 Ω / 5 W dummy load resistor between each motor winding output and `VMOT` sequentially, or all four simultaneously if current limit permits.**

> This stage validates that each **TIP120** switches correctly without an inductive motor load. Use resistive dummy loads to avoid flyback transients during validation.

### 8.1 Single-Channel Switching Test

For each channel (Q1 through Q4):

1. Connect the dummy load between the corresponding motor output pin and `VMOT`.
2. Set the oscilloscope probe to the **TIP120** emitter (motor output side).
3. Allow the clock to run.

| Measurement | Expected |
|---|---|
| Output voltage when corresponding 74LS194 output = HIGH | Near `GND` (TIP120 saturated, Vce(sat) ≈ 2 V) |
| Output voltage when corresponding 74LS194 output = LOW | Near `VMOT` (TIP120 OFF, dummy load pulls to VMOT) |
| Switching transition | Clean rise/fall, no excessive ringing (> 2× VMOT peak) |
| Dummy load current | I = VMOT / 10 Ω ≈ 0.5 A at VMOT = 5 V (within 5 W rating) |

### 8.2 Cross-Channel Isolation

With all four dummy loads connected:

| Check | Expected |
|---|---|
| Only one output LOW at any time | Matches the 74LS194 one-hot sequence |
| No two outputs simultaneously LOW | No shoot-through (would short VMOT through two dummy loads) |
| Each output cycles independently | Q1 → Q2 → Q3 → Q4 → Q1 (or reverse per DIR setting) |

### 8.3 Flyback Diode Polarity (Resistive Load)

Even without an inductive load, verify the flyback diode orientation:

| Test | Expected |
|---|---|
| DMM diode test: anode at motor output, cathode at VMOT | Forward voltage ~0.5–0.7 V (1N4001 conducting) |
| DMM diode test: anode at VMOT, cathode at motor output | Open circuit (reverse biased) |

> If a flyback diode is reversed, it will conduct continuously when the corresponding TIP120 is OFF, shorting VMOT to the motor output. This will cause excessive current draw and possible diode failure.

---

## 9. Stage 7 — Motor Connection and Full Functional Test

**Prerequisite: Stages 1–6 all pass. Heatsinks installed on Q1–Q4.**

### 9.1 Motor Connection

1. Power off both rails.
2. Identify the unipolar stepper motor winding leads:
   - Common center tap (CT) → connect to `VMOT`
   - Four winding ends → connect to Q1, Q2, Q3, Q4 outputs (order per motor datasheet)
3. Verify the motor winding resistance with a DMM before connecting:
   - CT to each winding end: typically 5–50 Ω (varies by motor)
   - Winding end to winding end (same phase): 2× CT-to-end resistance
   - Winding end to winding end (different phase): open circuit

### 9.2 Low-Voltage Functional Test

1. Set `VMOT` to the motor's rated minimum voltage (or 5 V for initial spin test).
2. Set `VCC` to 5 V.
3. Set current limit on `VMOT` supply to 1.5× the motor's rated per-phase current.
4. Power on.

| Observation | Expected |
|---|---|
| Motor shaft | Rotates smoothly in the direction set by DIR jumper |
| Motor sound | Uniform stepping noise, no grinding or stalling |
| `VMOT` supply current | Within 1.5× rated per-phase current (one winding energized at a time) |
| `VCC` supply current | Remains < 100 mA (logic stage unaffected by motor load) |
| **TIP120** case temperature | Warm but not too hot to touch (< 60 °C) after 1 minute |

### 9.3 Direction and Speed Test

| Test | Expected |
|---|---|
| Toggle DIR jumper while running | Motor reverses direction smoothly, no lost steps at low speed |
| Adjust NE555 frequency (if trimmer) | Motor speed changes proportionally; no stalling within rated speed range |
| Increase `VMOT` to rated voltage (incrementally) | Torque increases; current remains within limits |

### 9.4 Full-Voltage Burn-In (Optional)

1. Run the motor at rated voltage and mid-range speed for 10 minutes.
2. Monitor **TIP120** case temperatures with a thermocouple or infrared thermometer.
3. Acceptance: no transistor exceeds 80 °C case temperature (with heatsink); no motor stalling; no unusual odor or smoke.

---

## 10. Troubleshooting Reference

| Symptom | Likely Cause | Diagnostic Step |
|---|---|---|
| `VCC` current limit trips on power-up | Short between `VCC` and `GND`; reversed IC | Return to Stage 1.3; inspect **U1**–**U3** pin  orientation |
| NE555 pin 3 stuck HIGH | RESET pin (pin 4) floating or tied LOW; C1 shorted | Measure pin 4 voltage (must be ≥ 0.7 Vcc); check C1 for short |
| NE555 pin 3 stuck LOW | C1 open; R1/R2 open; OUTPUT pin shorted to GND | Check C1 continuity; measure R1/R2; inspect pin 3 for solder bridge |
| 74LS194 outputs do not advance | Clock not reaching pin 11; S0/S1 both LOW (hold mode); **U2** defective | Scope pin 11; verify S0/S1 voltages; swap **U2** if socketed |
| All 74LS194 outputs HIGH simultaneously | S0 = S1 = HIGH (parallel load) with all data inputs HIGH | Check DIR jumper; verify S0/S1 are not both tied HIGH |
| 7407 output always LOW | Output transistor shorted; pull-up resistor missing | DMM diode test between output and GND; check base resistor network |
| 7407 output always floating | Input not driven; **U3** defective; Vcc missing on **U3** | Measure **U3** pin 14 voltage; check input from 74LS194 |
| One motor channel does not fire | Corresponding **TIP120** defective; base resistor open; flyback diode shorted | DMM transistor test on **TIP120**; check base resistor; verify diode orientation |
| Motor vibrates but does not rotate | Winding order incorrect; two windings swapped; insufficient VMOT | Recheck motor lead wiring; verify one-hot sequence reaches outputs; increase VMOT |
| Motor runs hot / stalling | VMOT too high; current limit too high; missing flyback diode | Reduce VMOT; verify current limit; check D1–D4 presence and orientation |
| **TIP120** runs very hot | Insufficient heatsinking; base drive current too low (not fully saturated); motor overcurrent | Install larger heatsinks; verify base resistor value; check motor current rating |

---

## 11. Test Record Template

Print or copy this section for each board tested.

```
Board Serial Number: ____________________
Date Tested: ____________________
Tester: ____________________

Stage 1 — Visual/Continuity:   [ ] PASS  [ ] FAIL  Notes: ____________
Stage 2 — Power Rails:         [ ] PASS  [ ] FAIL  Notes: ____________
Stage 3 — NE555 Clock:         [ ] PASS  [ ] FAIL  Notes: ____________
  Measured frequency: ________ Hz   Duty cycle: ________ %
Stage 4 — 74LS194 Logic:       [ ] PASS  [ ] FAIL  Notes: ____________
  Forward sequence verified: [ ]   Reverse sequence verified: [ ]
Stage 5 — 7407 Buffer:         [ ] PASS  [ ] FAIL  Notes: ____________
Stage 6 — TIP120 Power (dummy):[ ] PASS  [ ] FAIL  Notes: ____________
  Channels verified: Q1[ ] Q2[ ] Q3[ ] Q4[ ]
Stage 7 — Motor Full Test:     [ ] PASS  [ ] FAIL  Notes: ____________
  Motor model: ____________  VMOT: ________ V  Direction: Fwd[ ] Rev[ ]

Overall Result: [ ] PASS  [ ] FAIL  [ ] CONDITIONAL (see notes)
Signature: ____________________
```

---

*This testing protocol is intended for use by qualified electronics technicians. Always verify component values and connections against the schematic (`Hardware/Stepper_Motor_Driver.kicad_sch`) before applying power.*
