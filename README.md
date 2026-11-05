# Design of an S-Band Low Noise Amplifier Using the BFP405 RF Bipolar Transistor

Design, simulation and PCB layout of a single-stage low noise amplifier (LNA) centered at **2.4 GHz** (lower S-band), using the Infineon **BFP405** silicon NPN RF transistor and **microstrip matching networks**.

> **Status:** Simulation complete. Board layout and fabrication and VNA / noise-figure measurements are pending.

**Research Project (EC4095), 2026-2027**
Department of Electronics and Communication Engineering
**Supervisor:** Prof. Prasun Chongder

---

## Table of Contents

1. [Overview](#overview)
2. [Key Results](#key-results)
3. [Design Specifications](#design-specifications)
4. [Design Flow](#design-flow)
5. [Repository Structure](#repository-structure)
6. [Tools Used](#tools-used)
7. [How to Reproduce](#how-to-reproduce)
8. [Known Limitations](#known-limitations)
9. [Future Work](#future-work)
10. [References](#references)
11. [Authors](#authors)
12. [License](#license)

---

## Overview

The LNA is the first active block of an RF receiver front end, so its noise figure and gain largely set the sensitivity of the whole receiver (Friis' cascade formula). This project targets:

| Target | Goal |
| --- | --- |
| Noise figure | < 2 dB |
| Gain (S21) | > 15 dB |
| Input return loss (S11) | better than 10 dB |
| Stability | unconditionally stable (K > 1) |

Matching is implemented entirely with microstrip lines, steps, tees and stubs, so the amplifier is planar, compact and suitable for low-cost PCB fabrication.

![Complete LNA schematic](images/schematic.png)

---

## Key Results

Simulated in Keysight ADS:

| Parameter | Target | Simulated |
| --- | --- | --- |
| Noise figure | < 2 dB | 1.46 dB (device NFmin) |
| Gain S21 | > 15 dB | 15.57 dB at 2.46 GHz |
| Input return loss S11 | < -10 dB | -33.4 dB at 2.46 GHz |
| Stability factor K | > 1 | 1.025 at 2.4 GHz |
| Maximum available gain (MAG) | n/a | 19.84 dB at 2.4 GHz |

- Gain stays above 15 dB over roughly 2.2-2.5 GHz.
- S11 is better than -10 dB from about 2.4 to 2.7 GHz.
- The response is narrowband, as expected from a single-stage, stub-matched design.

![Simulated S21 and S11](images/s21_s11_plot.png)

---

## Design Specifications

### Bias point

| Parameter | Value |
| --- | --- |
| VCE | 2 V |
| IC | 4.92 mA |
| IB | 56 uA |
| DC power | about 10 mW |

### Substrate

| Parameter | Value |
| --- | --- |
| Relative permittivity (er) | 4.4 |
| Substrate height (h) | 0.508 mm |
| Conductor thickness (t) | 0.035 mm |
| Loss tangent (tan d) | 0.01 |

### Bias network and components

| Component | Value |
| --- | --- |
| Supply voltage | 3.0 V |
| Bias resistor R1 | 30 kOhm |
| Inductors L1, L2 | 15 nH |
| DC-blocking capacitors C1, C2 | 10 pF |

### Matching network (test bench)

| Element | Width x Length |
| --- | --- |
| Input line TL4 | 1.5 mm x 16.57 mm |
| Output series line TL2 | 0.9 mm x 4.58 mm |
| Shunt short-circuited stub TL1 | 0.9 mm x 3.46 mm |
| Output line TL3 | 3 mm length |

### Device noise and match data at 2.4 GHz

| Quantity | Value |
| --- | --- |
| NFmin | 1.463 dB |
| Gamma_opt | 0.202 angle 49.5 deg |
| Z_opt | 61.6 + j19.8 Ohm |
| Z_source (simultaneous match) | 5.05 + j4.56 Ohm |
| Z_load (simultaneous match) | 20.7 + j121.6 Ohm |
| Power gain, noise-matched source | 16.85 dB |

---

## Design Flow

1. **Bias selection:** the BFP405 datasheet NFmin vs. IC curve was used to choose VCE = 2 V, IC of about 5 mA as a compromise between noise figure and gain. A DC sweep in ADS confirmed IC = 4.92 mA at IB = 56 uA.
2. **Device characterization:** S-parameters, noise parameters and simultaneous-match impedances at 2.4 GHz with noise calculation enabled. K = 1.025, so the device is unconditionally stable.
3. **Matching network design:** the input network presents Gamma_opt for minimum noise figure; the output network gives a conjugate match to 50 Ohm for high gain. Designed on the Smith chart and refined with tunable line variables (sweep 2-3 GHz, 10 MHz steps).
4. **Microstrip realization:** impedances and electrical lengths converted to physical dimensions through the W/h ratio on the chosen substrate. A 1 mm line gives about 49 Ohm (50 Ohm feed line).
5. **Bias network and DC blocks:** 3 V supply, R1, L1/L2, and 10 pF series capacitors C1/C2.
6. **PCB layout:** planar layout drawn in Ansys HFSS, with emitter pads grounded through vias placed next to the pads and a short input line to limit noise-figure degradation.

![PCB layout](layout/hfss/layout_final.png)

---

## Repository Structure

```
.
├── README.md
├── LICENSE
├── .gitignore
├── report/                 IEEE-format project report (PDF / DOCX)
├── presentation/           Project presentation (PPTX)
├── simulation/
│   └── ads/                Keysight ADS workspace / netlists and exported results
├── layout/
│   └── hfss/               Ansys HFSS layout project, exports and layout image
├── images/                 Schematic, S-parameter plots, bias-point plots
└── docs/                   Design notes and calculations
```

> Adjust this tree to match the files you actually uploaded.

---

## Tools Used

| Tool | Purpose | Version |
| --- | --- | --- |
| Keysight ADS | Schematic, DC / S-parameter / noise simulation | _add version_ |
| Ansys HFSS | PCB layout drawing | _add version_ |
| Microsoft Word | IEEE-format report | n/a |

---

## How to Reproduce

1. Clone the repository:
   ```bash
   git clone https://github.com/<your-username>/<repo-name>.git
   ```
2. Open the ADS workspace in `simulation/ads/`.
3. **Device characterization:** run the DC sweep (VCE 0-3 V, IB 36-76 uA) and the S-parameter / noise simulation at 2.4 GHz.
4. **Matching test bench:** run the S-parameter sweep from 2 to 3 GHz in 10 MHz steps and evaluate the power gain with the `pwr_gain` function.
5. **Full LNA:** simulate the complete schematic (with bias network and DC blocks) over 2.0-3.0 GHz to obtain S11 and S21.
6. Open the layout project in `layout/hfss/` to view the PCB layout.

---

## Known Limitations

- **Design is simulation-only.** The board has not been fabricated or measured yet.
- **Small gain margin:** S21 = 15.57 dB is only 0.57 dB above the 15 dB target, so component and substrate tolerances could push measured gain below target.
- **Small stability margin:** K = 1.025 at 2.4 GHz; stability should be rechecked over the whole band.
- **Noise figure not yet verified on the full circuit:** 1.46 dB is the device NFmin. A full-circuit noise-figure sweep is still needed to confirm < 2 dB over the band.
- **Offset between gain peak and best input match:** gain peaks near 2.34 GHz while the best S11 is near 2.46 GHz (about 0.1 GHz apart).
- **Narrowband response** (single-stage, stub-based matching).

---

## Future Work

- Fabricate the PCB and measure with a VNA and noise-figure meter; compare with simulation.
- Align the gain peak with the input match.
- Increase the gain and stability margins.
- Widen the bandwidth toward the full 2-4 GHz S-band (e.g., balanced or broadband matching).
- Explore differential LNA configurations.

---

## References

1. Infineon Technologies AG, "BFP405: Surface mount wideband silicon NPN RF bipolar transistor," Datasheet, Rev. 3.0, Jul. 2024.
2. K.-W. Yeom, *Microwave Circuit Design: A Practical Approach Using ADS*. Upper Saddle River, NJ, USA: Prentice Hall, 2015.
3. G. Gonzalez, *Microwave Transistor Amplifiers: Analysis and Design*, 2nd ed. Upper Saddle River, NJ, USA: Prentice Hall, 1997.

> The BFP405 datasheet is not included in this repository for copyright reasons. Download it from the Infineon website.

---

## Authors

- **Dinesh Behera**




---



## Acknowledgment

The authors thank Prof. Prasun Chongder for guidance throughout the Research Project.
