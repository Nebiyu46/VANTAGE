# VANTAGE

**V**ibration **AN**alysis **T**ool for **A**ppliance dia**G**nostic **E**valuation

A portable, battery-powered diagnostic instrument for predictive maintenance of domestic washing machines.

---

**Author:** Nebiyu Hailu  
**Course:** Design of Electronic Devices and Systems

## Overview

VANTAGE implements predictive maintenance by analyzing the acoustic and vibrational "fingerprint" of washing machines. Unlike reactive repair strategies, it enables proactive monitoring to detect faults before catastrophic failure occurs.

The system uses a piezoelectric contact sensor to capture mechanical vibrations, processed through a custom analog front-end with programmable gain amplification and split-path filtering. An onboard STM32 microcontroller performs real-time FFT analysis and compares signatures against learned baselines to detect incipient faults.

## Features

- **Vibration Analysis** — Real-time FFT for detecting unbalance and motor faults
- **Envelope Analysis** — High-frequency demodulation for early-stage bearing pitting detection
- **Adaptive Monitoring** — "Learn Mode" to establish statistical baselines for specific machine cycles
- **Health Metrics** — Quantitative "Health Percentage" and specific fault alerts via LCD
- **Portable Design** — Rechargeable LiPo battery with USB-C charging
- **Robust Construction** — IP54 rated enclosure for laundry room environments

## Project Structure

```
VANTAGE/
├── hardware/
│   ├── kicad/              # PCB schematic and layout (KiCad 7+)
│   │   ├── Vibration_Analyer.kicad_pro
│   │   ├── Vibration_Analyer.kicad_sch    # Main schematic
│   │   ├── Vibration_Analyer.kicad_pcb    # PCB layout
│   │   ├── mcu_sec.kicad_sch              # MCU section
│   │   └── power_section.kicad_sch        # Power supply section
│   └── enclosure/          # 3D printable enclosure (STL)
├── firmware/
│   ├── Core/               # Application source code
│   │   ├── Inc/            # Header files
│   │   └── Src/            # Source files
│   ├── Vibration_ana.ioc   # STM32CubeMX configuration
│   ├── CMakeLists.txt      # CMake build configuration
│   └── startup_stm32f401xc.s
└── docs/                   # Design documentation
    ├── Vibration Analyzer Design Report.pdf
    ├── Enclosure Drawing Part 1.pdf
    └── Enclosure Drawing Part 2.pdf
```

## Hardware Specifications

| Component | Specification |
|-----------|---------------|
| **MCU** | STM32F401RCT6 (ARM Cortex-M4 with FPU) |
| **Logic Voltage** | 3.3V |
| **ADC** | 12-bit, >2 Msps capable |
| **Display** | 128x64 Graphic LCD or 16x2 Character LCD (I2C) |
| **Battery** | Single-cell LiPo 3.7V, 1000mAh |
| **Charging** | USB-C 5V with TP4056 management |
| **Regulator** | MCP1700-3302E LDO |
| **Op-Amps** | MCP6022 (Rail-to-Rail) |
| **Sensor** | Piezoelectric contact transducer |

## Analog Front-End Architecture

```
                                            ┌──────────────┐
                                       ┌───▶│  LPF ~8kHz   │────▶ ADC (PC0)
                                       │    │ (Anti-Alias) │       Path A (FFT)
              ┌───────┐   ┌─────────┐  │    └──────────────┘
 Piezo ──▶   │ Buffer │──▶│   PGA   │──┤
              └───────┘   │ 1×-128× │  │    ┌──────────┐   ┌──────────────┐
                          └─────────┘  └───▶│ HPF >2kHz│──▶│  Envelope    │──▶ ADC (PB1)
                                            └──────────┘   │  Detector    │    Path B (Envelope)
                                                           └──────────────┘
```

- **Path A (Spectral Analysis):** 2nd-order Sallen-Key LPF (~8 kHz cutoff) for FFT
- **Path B (Feature Extraction):** HPF (~2.1 kHz) → Envelope Detector for bearing faults
- **PGA:** Digitally controlled gain (1× to 128×) via CD4051B multiplexer
- **Virtual Ground:** 1.65V reference for single-supply operation

## Fault Detection

| Fault Type | Detection Method | Frequency Range |
|------------|------------------|-----------------|
| Load Unbalance | Peak at 1× RPM | 10-20 Hz |
| Bearing Wear | Envelope analysis | >2 kHz |
| Belt Degradation | Harmonic analysis | Motor freq. |
| Electrical Fault | 100 Hz signature | 100 Hz |
| Suspension Failure | Low-freq response | <50 Hz |

## Building the Firmware

### Prerequisites
- [STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html)
- [ARM GCC Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm)
- CMake 3.20+

### Build Steps

1. **Generate HAL Drivers** (first time only):
   ```bash
   # Open firmware/Vibration_ana.ioc in STM32CubeMX
   # Click "Generate Code" to create Drivers/ folder
   ```

2. **Build**:
   ```bash
   cd firmware
   mkdir build && cd build
   cmake ..
   make
   ```

## Environmental Design

| Specification | Value |
|---------------|-------|
| **IP Rating** | IP54 (Dust Protected / Splash Proof) |
| **Environment** | Laundry room (humidity, vibration) |
| **Enclosure** | ABS/PETG, 3D printed |
| **PCB Protection** | Acrylic conformal coating |
| **Sealing** | Silicone gaskets |
| **Operating Temp** | Up to 60°C |
