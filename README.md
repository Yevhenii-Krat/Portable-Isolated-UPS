# Portable-Isolated-UPS
SYSTEM ARCHITECTURE SPECIFICATION: Portable Isolated UPS Inverter

1. PROJECT OVERVIEW
- System topology: Portable isolated inverter with full input and output galvanic barrier (Transformer-Coupled Isolation).
- Output power: Maximum 100W.
- Main power supply: USB-C PD port (dedicated to powering in normal operation mode).
- Backup power supply: 12V 7Ah lead-acid battery (Backup time: capacity 84 Wh / 100 W = 0.84 h. Considering inverter efficiency of around 90% and the discharge profile, the actual operating time is approx. 45-50 minutes).
- Output voltage: 230V AC (pure sine wave, 50Hz, hardware voltage stabilization).
- Main microcontroller: STM32G4 (Industrial-grade chip, no NUCLEO platform).
- Galvanic barrier: Full digital separation (e.g., ISO7740 series) and physical magnetic isolation at the DC-DC conversion stage.
- Assembly technology: SMD components with precise outlines on the silkscreen layer.

2. ENERGY FLOW ARCHITECTURE (POWER PATH)

A. Low-Voltage Domain (LV DC - Management and Charging)
- Power input: USB-C PD interface with input overvoltage protection (OVP).
- Battery management (BMS): The microcontroller oversees battery charging while maintaining safe capacity limits (50% - 95%) to minimize cell degradation.
- Zero Transition: The use of built-in, ultra-fast hardware comparators in the STM32G4 microcontroller guarantees an uninterrupted switch of the power stage to draw energy from the battery, bypassing software loop delays (software glitch).

B. Processing and Isolation Domain (Galvanic Barrier)
- DC-DC isolation stage: The switching topology steps up the voltage from 12V to a high DC voltage, ensuring full ground separation between the LV circuits and the AC output.
- Main transformer: A switching transformer with a ferrite core (TI-ETD39-DER212) is used, optimized for operation at high switching frequencies (kHz).
- HV DC bus: The secondary voltage is rectified to a stable direct current bus level (approx. 400V DC), galvanically isolated from the battery and the control system.

C. High-Voltage Domain (HV AC - Output Inverter)
- Digital control: The STM32G4 chip generates SPWM signal vectors, transmitted to the HV domain through ultra-fast ISO7740 digital isolators.
- H-Bridge (Floating Output): SPWM signals drive the isolated UCC21520 gate drivers. The inverter circuit intentionally operates with a completely floating ground (Floating Output / Electrical Separation), which prevents electric shock if a single live wire is touched.
- Voltage shaping: The output LC filter transforms the bridge signal into a clean, stable sine wave with parameters of 230V / 50Hz.

3. PHYSICAL CONSTRUCTION, MEASUREMENTS, AND SAFETY

- Mechanics and PCB (Stackup): Modular architecture consisting of 3 or 4 boards of an identical format. Boards are stacked in a cascade ("sandwich" style) and mechanically separated using polyamide standoffs, which guarantees rigidity and safe air gaps.
- Isolation and Standards: Analysis and maintenance of isolation distances (creepage and clearance) in the air and on the PCB surface, in accordance with the requirements of the IEC 62368-1 standard.
- HV Voltage Measurement: Precise, galvanically isolated AMC1200 delta-sigma converters powered by dedicated converters with magnetic isolation, maintaining the full continuity of the galvanic barrier.
- Service Diagnostics: Integration of outlined physical test points (TP) for the 12V, 400V DC, and 230V AC sections.

4. HARDWARE PROTECTIONS AND EMC (FAIL-SAFE DESIGN)

- Emergency Protection (Fail-Safe): The gate drivers (UCC21520) are equipped with hardware pull-down resistors and UVLO circuits. In the event of an MCU freeze or logic power loss, an independent circuit (e.g., watchdog / hardware Enable signal) forces the H-bridge into an immediate safe shutdown state (Latch-off).
- Electromagnetic Compatibility (EMC): Implementation of common-mode chokes on the DC input and AC output in order to suppress conducted interference and meet EMI emission standards.
- Surge and Thermal Protection: The 230V AC output is protected by a varistor (MOV) to absorb flyback pulses from non-linear inductive loads. The system also has NTC thermistors on the power stage heatsinks, allowing for power reduction in case of overheating.
