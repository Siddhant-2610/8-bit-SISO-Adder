# 8-Bit SISO Adder
**SCL 180nm CMOS | Transistor-Level Full Custom Physical Design**

## Overview
This repository contains the full-custom, transistor-level physical implementation and post-silicon characterization of a 8-Bit SISO Adder in the SCL 180nm CMOS process. 

To maintain control over device geometry, active-area sharing, and parasitic routing penalties, automated standard-cell synthesis methodologies were intentionally bypassed. The project focuses on the complete Backend-of-Line (BEOL) workflow: schematic capture of fundamental logic primitives, custom standard cell layout utilizing Euler path optimization, top-level datapath integration, and Assura sign-off (DRC, LVS, PEX).

## Key Architectural Features
* **Full-Custom Transistor-Level Design:** Built entirely at the transistor level in 180nm technology, avoiding standard-cell abstraction for tighter control over timing and area.
* **Dual SISO Input Registers:** Two independent 8-bit Serial-In-Serial-Out registers (IN1, IN2) shift in operands LSB-first over the first 8 clock cycles, avoiding the need for parallel-load infrastructure.
* **Single-Bit Full Adder + Carry D Flip-Flop Datapath:** A minimal combinational core — one Full Adder and one carry-holding DFF — reused serially across all 8 bit positions instead of replicating adder logic per bit.
* **Asynchronous Active-Low Reset:** Hardware-level `RST` pin instantaneously clears internal state independent of the clock, ensuring deterministic power-up/re-initialization behavior.
* **Serial 25-Cycle Operation:** A single 25-cycle sequence spans Input Loading (cycles 1–8), Addition/SUM Generation (cycles 8–16), and Output Shifting (cycles 17–25) — all through the same reused datapath.

## Performance Characteristics
This adder macro is optimized as a compact, low-power datapath primitive suited for area- and energy-constrained serial arithmetic in edge applications.
* **High-Speed Operation:** The physical datapath achieves a max frequency of 3.85 GHz, driven by the minimal single-bit critical path (Full Adder + carry DFF) rather than a wide parallel adder chain.
* **Low-Power Footprint:** Averages 78 µW under random input switching, with 24.375 pJ total energy per 8-bit addition — a direct result of reusing one bit-slice of logic across 25 cycles instead of parallelizing.
* **Compact Layout:** Occupies only 200 × 47.5 µm (9,500 µm²), reflecting the area savings of a bit-serial architecture over a fully parallel adder.


## Post-Layout Performance (PEX Extracted)
| Parameter | Description | Value |
| :--- | :--- | :--- |
| **Technology Node** | SCL CMOS | 180nm |
| **Supply Voltage (VDD)** | Nominal | 1.8 V |
| **Maximum Frequency (Fmax)** | CLK to Datapath | 3.85 GHz |
| **Dynamic Power** | Active switching | 78 µW |
| **Total Area** | Silicon Bounding Box | 9500 µm² |
*Sign-off: 100% DRC and LVS clean in Cadence Assura*

## Tools Used
* **Schematic & Layout:** Cadence Virtuoso
* **Analog/Mixed-Signal Simulation:** Cadence ADE Explorer (Spectre RF)
* **Physical Verification:** Assura (DRC, LVS, PEX)

## Pin Configuration
| Pin Name | Direction | Description |
| :--- | :--- | :--- |
| **VDD** | Input | 1.8V Core Supply |
| **GND** | Input | 0.0V Reference |
| **IN1** | Input | 8-bit serial input |
| **IN2** | Input | other 8-bit serial input |
| **CLK** | Input | System Clock (Rising-edge active) |
| **RST** | Input | Asynchronous active-low initialization |
| **OUT** | Output | upto 9 bit serial output |


## Module Hierarchy
The top module (`serialadder`) is built from the ground up using fundamental logic gates:
```text
serialadder
|-- reg (x2 instances)
|   |-- dff (x8 instances)
|       |-- inv (x3 instances)
|       |-- nand (x6 instances)
|       |-- nand_3in (x2 instances)
|
|-- fulladder 
|   |-- xor (x2 instances)
|       |-- nand (x4 instances)
|   |-- and (x3 instances)
|       |-- nand 
|       |-- inv 
|   |-- or (x2 instances)
|       |-- nand (x3 instances)
|
|-- dff 
|       |-- inv (x3 instances)
|       |-- nand (x6 instances)
|       |-- nand_3in (x2 instances)
|
|-- outreg 
|   |-- dff (x9 instances)
|       |-- inv (x3 instances)
|       |-- nand (x6 instances)
|       |-- nand_3in (x2 instances)
