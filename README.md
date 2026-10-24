# SpaceRad-HIL: COTS Radiation Tolerance & Hardware-in-the-Loop Simulator

## Overview
SpaceRad-HIL is a Hardware-in-the-Loop (HIL) framework designed to simulate, analyze, and mitigate radiation-induced faults in Commercial Off-The-Shelf (COTS) microcontrollers for space and high-energy physics applications. 

The project bridges Monte Carlo physical simulations with real-time embedded fault-tolerance mechanisms, specifically targeting the **STM32F401** (ARM Cortex-M4) architecture. By avoiding costly radiation-hardened components, this framework demonstrates how Software-Implemented Fault Tolerance (SIFT) can reliably mitigate Single Event Effects (SEE) in harsh environments. I started this project to deepen my knowledge about ionizing radiations - which is one of my favourite fields - and to demonstrate I can seriously understand a problem and work on solutions using the most advanced tools. 

## System Architecture
The framework operates across three integrated layers:

1. **Geant4 Physics Engine:** Simulates the interaction of protons and gamma rays with a macroscopic $10 \times 10 \times 0.3 \text{ mm}$ pure silicon target. It models direct ionization and secondary nuclear reactions, exporting the deposited energy ($\Delta E$) event-by-event.
2. **Python HIL Injector:** Acts as the stochastic bridge. It reads the Monte Carlo dataset, converts energy releases into 12-bit ADC equivalents, and streams them to the STM32 via UART, emulating the Poisson temporal distribution of real particle fluxes.
3. **STM32F401 Embedded Mitigation (SIFT):** The bare-metal C/C++ firmware operates as a Multi-Channel Analyzer (MCA) and implements active mitigation against Single Event Upsets (SEUs) and Single Event Functional Interrupts (SEFIs) using Triple Modular Redundancy (TMR), internal software watchdogs, and memory scrubbing.

## Scientific Background & State of the Art
The usage of COTS microcontrollers in radiation environments (such as Low Earth Orbit or CERN accelerator tunnels) offers significant advantages in compactness, power efficiency, and cost compared to traditional radiation-hardened components. However, commercial MCUs lack hardware-based Triple Modular Redundancy (TMR) and are highly susceptible to both cumulative damage and stochastic effects:

* **Single Event Effects (SEE):** High-energy particles induce defects in the crystal lattice by atom displacement. In modern CMOS SRAMs, low-energy protons primarily cause SEUs through direct ionization, while high-energy protons trigger nuclear reactions resulting in high-LET (Linear Energy Transfer) recoil ions.
* **Soft and Hard Errors:** These stochastic events manifest as non-destructive soft errors (e.g., bit-flips in SRAM or configuration registers) or hard SEFIs that disrupt the Program Counter or cause peripheral crashes. 
* **Total Ionizing Dose (TID):** Long-term exposure generates electron-hole pairs trapped in the silicon oxide, leading to increased leakage currents, analog performance degradation, and eventual loss of Flash writability.

### Mitigation Strategy
Since hardware-level error correction is often unavailable on COTS devices, Software-Implemented Hardware Fault Tolerance (SIHFT) techniques are mandatory to ensure system reliability. Relying solely on external hardware watchdogs is insufficient to recover from all SEFI states, as a microcontroller can fail while still feeding the external watchdog pins. 

Therefore, this framework implements comprehensive Software Mitigation Schemes (SMS):
* **Memory Scrubbing & EDAC:** Periodic software-driven error detection and correction routines to prevent the accumulation of Multiple-Bit Upsets (MBUs) in the SRAM.
* **Internal Software Watchdogs:** Real-Time Clock (RTC) based software watchdogs to detect synchronization failures, ensuring the MCU recovers from anomalous sleep states or infinite loops.
* **Instruction & Data Redundancy:** Implementation of Software TMR for critical counters and data structures, along with control-flow checking signatures to validate execution paths.

Empirical validation in accelerated test facilities has demonstrated that implementing these software mitigation schemes drastically improves operational availability, reducing system downtime in COTS components from 48.82% to 2.58% under heavy irradiation.

## Getting Started
### Prerequisites
* Linux/WSL2 with Geant4 (v11.x) and CMake
* Python 3.x (`pyserial`, `numpy`)
* STM32F401RE Nucleo-64 board & STM32CubeIDE

### Usage
1. **Geant4 Simulation:** Run the modified `exampleB1` macro to generate the `sensor_hits.csv` dataset.
2. **STM32 Bring-up:** Flash the SIFT-enabled firmware onto the STM32F401.
3. **HIL Injection:** Execute `hil_injector.py` to stream the radiation events to the MCU and monitor the serial telemetry for automated fault recovery logs.
