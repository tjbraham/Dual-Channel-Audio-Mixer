# Dual-Channel 3-Band Analog Audio Mixer PCB

A custom single-supply ($9\text{ V}$) dual-channel analog audio mixer featuring a 3-band Linkwitz-Riley ($Q = 0.5$) Sallen-Key active crossover network, $180^\circ$ mid-band phase inversion for flat summation, and an inverting summing architecture designed in **LTspice** and **KiCad** for automated SMT assembly.

---

## Hardware Overview & Key Specifications

| Parameter | Specification |
| :--- | :--- |
| **Channels** | 2 Independent Stereo-to-Mono Input Channels (`J7`, `J8`) + 1 Master Output (`J9`) |
| **Filter Topology** | 2nd-Order Unity-Gain Sallen-Key Active Crossover ($Q = 0.5$, $-6\text{ dB}$ at $f_c$) |
| **Frequency Bands** | **Low:** $< 102.6\text{ Hz}$ \| **Mid:** $102.6\text{ Hz} - 4.98\text{ kHz}$ \| **High:** $> 4.98\text{ kHz}$ |
| **Active ICs** | $4\times$ `TL074CDT` Quad JFET-Input Operational Amplifiers (`SOIC-14`) |
| **Power Supply** | $9\text{ V}$ Single-Supply (`BT1` / `SW1`) with $4.5\text{ V}$ Virtual Ground (`VRef`) |
| **PCB Dimensions** | $87.11\text{ mm} \times 51.33\text{ mm}$ ($3.43\text{ in} \times 2.02\text{ in}$), 2-Layer `FR-4` ($1.6\text{ mm}$, $1\text{ oz}$ Cu) |
| **Component Count** | 89 Total Designators (69 `0805`/`SOIC-14` SMD Parts + 20 Through-Hole/DFT Items) |

---

## Circuit Architecture & Engineering Decisions

### 1. Input Stage & Single-Supply Virtual Reference (`VRef`)
* **Passive Stereo-to-Mono Input Summing:** Each $3.5\text{ mm}$ TRS input jack (`J7` and `J8`) sums Left (Tip) and Right (Ring) channels through matched $1\text{ k}\Omega$ resistors (`R10`/`R11` and `R20`/`R21`), providing a $500\ \Omega$ effective source impedance before AC coupling through $10\ \mu\text{F}$ capacitors (`C2`, `C3`) and biasing to `VRef` via $1\text{ M}\Omega$ resistors (`R9`, `R22`).
* **Virtual Ground Generation (`VRef = 4.5V`):** Because the circuit operates from a single $9\text{ V}$ battery (`BT1`), a $10\text{ k}\Omega / 10\text{ k}\Omega$ voltage divider (`R3`, `R4`) bypassed with a $10\ \mu\text{F}$ capacitor (`C1`, $f_c = 3.18\text{ Hz}$) establishes a low-noise $4.5\text{ V}$ mid-rail reference (`VRef`) for all op-amp non-inverting inputs and filter bias paths.
* **Click-Free Potentiometer Biasing:** All six $10\text{ k}\Omega$ band-level potentiometers (`RV1`–`RV6`) tie their reference terminals directly to `VRef` rather than `GND`. Keeping the wiper and the op-amp summing nodes at an identical $4.5\text{ V}$ DC potential eliminates DC current flow through the potentiometer wipers, preventing scratchy audio noise when adjusting EQ levels.

### 2. 3-Band Sallen-Key Crossover & Impedance Scaling
Each input channel splits into three parallel active filter branches designed around equal-$`R`$, equal-$`C`$ unity-gain Sallen-Key topologies ($Q = 0.5$):
* **Low-Pass Band (`U1A`, `U2C`):** Uses $R = 3.3\text{ k}\Omega$ (`R26`/`R27`, `R12`/`R13`) and $C = 470\text{ nF}$ (`C9`/`C10`, `C21`/`C22`) to set a cutoff frequency of:
  $$f_{c,\text{LP}} = \frac{1}{2\pi (3.3\text{ k}\Omega)(470\text{ nF})} = 102.61\text{ Hz}$$
* **High-Pass Band (`U1B`, `U2D`) & Impedance Scaling:** Uses $C = 4.7\text{ nF}$ (`C13`/`C14`, `C5`/`C7`) and $R = 6.8\text{ k}\Omega$ (`R30`/`R31`, `R14`/`R42`) to set a cutoff frequency of:
  $$f_{c,\text{HP}} = \frac{1}{2\pi (6.8\text{ k}\Omega)(4.7\text{ nF})} = 4979.82\text{ Hz}$$
  Scaling the high-frequency filter impedances up by $100\times$ (from $68\ \Omega / 470\text{ nF}$ to $6.8\text{ k}\Omega / 4.7\text{ nF}$) prevents the high-pass stage from loading down the unbuffered $500\ \Omega$ input node and avoids overloading the `TL074` output stages.
* **Mid Band-Pass Cascade & $180^\circ$ Phase Inversion (`U1C` $\rightarrow$ `U1D` $\rightarrow$ `U2A` & `U3A` $\rightarrow$ `U3B` $\rightarrow$ `U3C`):** Cascades a $102.61\text{ Hz}$ Sallen-Key high-pass filter ($3.3\text{ k}\Omega$, $470\text{ nF}$) into a $4979.82\text{ Hz}$ Sallen-Key low-pass filter ($6.8\text{ k}\Omega$, $4.7\text{ nF}$). Because 2nd-order low-pass and high-pass filters introduce $\pm 90^\circ$ phase shifts at their crossover frequencies, summing them in-phase would create deep $-30\text{ dB}$ cancellation notches at $100\text{ Hz}$ and $5\text{ kHz}$; routing the Mid band through a dedicated unity-gain inverting amplifier (`U2A` with `R1`/`R2` $= 10\text{ k}\Omega$; `U3C` with `R34`/`R46` $= 10\text{ k}\Omega$) flips the Mid band by $180^\circ$ to achieve a flat $0.00\text{ dB}$ acoustic sum across $20\text{ Hz} - 20\text{ kHz}$.

### 3. Channel & Master Summing Stages
* **Channel Summing Amplifiers (`U2B`, `U3D`):** Sum the three EQ potentiometer wipers (`RV1`–`RV3` for Channel 1; `RV4`–`RV6` for Channel 2) through $100\text{ k}\Omega$ input resistors (`R5`–`R7`, `R16`–`R18`) with $100\text{ k}\Omega$ feedback resistors (`R8`, `R19`).
* **Master Summer & Output Coupling (`U4A`):** Sums both channels (`R23`, `R24` $= 100\text{ k}\Omega$) with unity feedback (`R25` $= 100\text{ k}\Omega$), AC-couples the final output through a $10\ \mu\text{F}$ blocking capacitor (`C4`) with a $100\text{ k}\Omega$ ground bleed resistor (`R35`, $f_c = 1.75\text{ Hz}$ into a $10\text{ k}\Omega$ line load), and drives both Tip and Ring of the output jack (`J9`).

---

## PCB Layout, DFM, & Design for Test (DFT)

* **Split Reference Planes:** The 2-layer PCB routes primary signal traces alongside a top-layer `VRef` copper pour (`F.Cu`) and a continuous bottom-layer `GND` return plane (`B.Cu`), providing low-impedance reference paths and shielding sensitive summing nodes from crosstalk.
* **Power Integrity & Op-Amp Termination:** Four local $100\text{ nF}$ (`0805`, X7R) decoupling capacitors (`C17`–`C20`) are placed within $5.8\text{ mm} - 6.9\text{ mm}$ of the `V+` pin (Pin 4) of each `TL074` (`U1`–`U4`) with direct vias to the bottom `GND` plane. Unused op-amp channels on `U4` (`U4B`, `U4C`, `U4D`) are configured as unity-gain followers tied to `VRef` to prevent floating-input oscillation and excess quiescent current draw.
* **Onboard DFT Test Points:** Eight $1.5\text{ mm} \times 1.5\text{ mm}$ bare-copper surface-mount test pads allow safe oscilloscope and multimeter probing during bring-up:
  * **DC Power Rails:** `9V_test1`, `Vref_test1`, `gnd_test1`
  * **AC Signal Chain:** `top_in_test1`, `bot_in_test1`, `top_out_test1`, `bot_out_test1`, `fin_out_test1`
* **Mechanical Integration:** Includes four $3.2\text{ mm}$ `M3` corner mounting holes for enclosure standoffs and custom `Edge.Cuts` slots to recess the barrels of the three horizontal $3.5\text{ mm}$ TRS audio jacks (`J7`–`J9`) flush with the board perimeter.

---

## Bill of Materials (JLCPCB SMT & Hand-Soldered Parts)

### Automated SMT Assembly Components
| Designators | Qty | Value | Package | LCSC Part # | Description / MPN |
| :--- | :---: | :---: | :---: | :---: | :--- |
| `C1, C2, C3, C4` | 4 | $10\ \mu\text{F}$ | `0805` | `C15850` | 25V X5R $\pm 10\%$ MLCC (`CL21A106KAYNNNE`) |
| `C6, C8, C9, C10, C11, C12, C21, C22` | 8 | $470\text{ nF}$ | `0805` | `C13967` | 50V X7R $\pm 10\%$ MLCC (`CL21B474KBFNNNE`) |
| `C5, C7, C13, C14, C15, C16, C23, C24` | 8 | $4.7\text{ nF}$ | `0805` | `C1744` | 50V X7R $\pm 10\%$ MLCC (`0805B472K500NT`) |
| `C17, C18, C19, C20` | 4 | $100\text{ nF}$ | `0805` | `C49678` | 50V X7R $\pm 10\%$ MLCC (`CC0805KRX7R9BB104`) |
| `R10, R11, R20, R21` | 4 | $1\text{ k}\Omega$ | `0805` | `C17513` | $\pm 1\%$ 1/8W Thick Film (`0805W8F1001T5E`) |
| `R12, R13, R15, R26-R29, R36, R43` | 9 | $3.3\text{ k}\Omega$ | `0805` | `C26010` | $\pm 1\%$ 1/8W Thick Film (`0805W8F3301T5E`) |
| `R14, R30-R33, R42, R44, R45` | 8 | $6.8\text{ k}\Omega$ | `0805` | `C17772` | $\pm 1\%$ 1/8W Thick Film (`0805W8F6801T5E`) |
| `R1-R4, R34, R46` | 6 | $10\text{ k}\Omega$ | `0805` | `C17414` | $\pm 1\%$ 1/8W Thick Film (`0805W8F1002T5E`) |
| `R5-R8, R16-R19, R23-R25, R35` | 12 | $100\text{ k}\Omega$ | `0805` | `C149504` | $\pm 1\%$ 1/8W Thick Film (`0805W8F1003T5E`) |
| `R9, R22` | 2 | $1\text{ M}\Omega$ | `0805` | `C17514` | $\pm 1\%$ 1/8W Thick Film (`0805W8F1004T5E`) |
| `U1, U2, U3, U4` | 4 | `TL074` | `SOIC-14` | `C6963` | Quad JFET Op-Amp (`TL074CDT`) |

### Hand-Soldered Through-Hole Components
| Designators | Qty | Footprint | Description |
| :--- | :---: | :--- | :--- |
| `J7, J8, J9` | 3 | `Jack_3.5mm_CUI_SJ1-3525N_Horizontal` | $3.5\text{ mm}$ Stereo TRS Audio Jack (`CUI SJ1-3525N`) |
| `RV1-RV6` | 6 | `PinHeader_1x03_P2.54mm_Vertical` | $1\times 3$ Header ($2.54\text{ mm}$) for $10\text{ k}\Omega$ Linear Potentiometers (`Low`, `Mid`, `High`) |
| `BT1, SW1` | 2 | `PinHeader_1x02_P2.54mm_Vertical` | $1\times 2$ Header ($2.54\text{ mm}$) for $9\text{ V}$ Battery Snap & SPST Power Switch |
| `D1` | 1 | `LED_D5.0mm` | $5\text{ mm}$ Through-Hole Power Indicator LED |

---

## Hardware Bring-Up & Testing Procedure

1. **Visual & Continuity Inspection:** Inspect all `SOIC-14` pins and `0805` pads after SMT delivery; verify with a digital multimeter (DMM) in continuity mode that no short exists between `9V_test1`, `Vref_test1`, and `gnd_test1` before soldering through-hole connectors.
2. **DC Power Rail Verification:** Connect a $9\text{ V}$ source to `BT1`, close `SW1`, confirm `D1` illuminates ($\sim 2.1\text{ mA}$ bias via `R36`), and probe `9V_test1` ($+9.0\text{ V}$) and `Vref_test1` ($+4.50\text{ V} \pm 0.05\text{ V}$) relative to `gnd_test1`.
3. **AC Signal Injection & Crossover Sweep:** Inject a $1\text{ V}_{\text{pp}}$ sine wave from a function generator into `J7` (`top_in_test1`) and `J8` (`bot_in_test1`) across $20\text{ Hz} - 20\text{ kHz}$ ($100\text{ Hz}$, $1\text{ kHz}$, and $10\text{ kHz}$ test points) while monitoring `top_out_test1`, `bot_out_test1`, and `fin_out_test1` on an oscilloscope to verify band isolation, $-6\text{ dB}$ crossover points, and clean summation.

---

## Repository Structure

```text
├── Docs/
│   ├── Audio_Mixer_PCB_schematic.pdf       # Full KiCad 10.0 schematic diagram
│   ├── pcb_3d_render.png                   # 3D board render
│   └── Audio_Mixer_PCB.pdf                 # F.Cu (VRef) and B.Cu (GND) copper views
│   └── Audio_Mixer_LTspice.pdf             # LTSpice diagram PDF
├── Schematic_and_PCB/
│   ├── production/                         # KiCad JLCPCB formatted production files
│       ├── bom.csv                         # JLCPCB-formatted Bill of Materials with LCSC numbers
│       └── positions.csv                   # Component Placement List (CPL) for SMT pick-and-place
│       └── Audio_Mixer_PCB_schematic.zip   # PCB Gerber and Drill files
├── Simulation/
│   └── Two_Channel_Audio_Mixer.asc         # LTspice active filter & summing simulation
├── Audio_Mixer_PCB.kicad_pro               # KiCad project file
├── Audio_Mixer_PCB.kicad_sch               # KiCad schematic file
├── Audio_Mixer_PCB.kicad_pcb               # KiCad 2-layer PCB layout file
└── README.md
```
