# UniSerial-Protocol-Virtualized-System-On-Chip
This Project Idea was obtained from an experience Encountered and the initial idea evolved into an complete design of an New SoC  which overcomes many issues or drawbacks possessed by Market Available Micro Controllers  
1. Project Overview
PV-SoC is a custom RISC-V–based System-on-Chip (SoC) that introduces hardware-enforced protocol virtualization over shared GPIO pin pairs.
Unlike conventional microcontrollers that rely on static pin multiplexing, PV-SoC allows the same physical pins to be dynamically reused at runtime for UART, SPI, I2C, or GPIO, with safety guaranteed by a dedicated hardware arbitration engine.The project is implemented end-to-end, from RTL design to GDSII, using industry-relevant open-source ASIC tools.

2. Motivation

Embedded and edge devices are often constrained by:

Limited pin count

Increasing peripheral requirements

Rigid, static pin multiplexing

In existing MCUs, pins are typically configured once at boot. Dynamic reuse across protocols is either unsafe, software-driven, or unsupported.

PV-SoC addresses this gap by introducing:

Runtime protocol switching, Hardware-level conflict prevention, Safe electrical handover between protocols, This reduces pin count while increasing system flexibility.

3. Key Innovation
4. 
Protocol Virtualization over Shared Pin Pairs

Physical pins are grouped into 2-pin protocol-virtualized groups

Each group can dynamically function as:

GPIO supported

UART (TX/RX), I2C (SDA/SCL), SPI (reduced-pin, half-duplex)
Switching is controlled by hardware, not software bit-banging

A custom Universal Serial Engine (USE):

Arbitrates protocol ownership

Enforces exclusive access

Ensures Hi-Z transitions and pull-up control

Detects conflicts and recovers from errors

This is the core contribution of the project.


4. High-Level Architecture

CPU: RISC-V core (SweRV EH1, reused without modification)

Interconnect: AXI for high-speed blocks, APB for peripherals

Custom Blocks:

Programmable PINMUX

Universal Serial Engine (USE FSM)

Peripherals: UART, SPI, I2C, GPIO (reused IP)

Optional Accelerator: AI/NPU via AXI (NVDLA small / Gemmini)

Power-aware design: Clock gating and power domains


5. Pin Architecture

4 protocol-virtualized pin groups

2 pins per group

Same pin pair reused across protocols at runtime

Hardwired (non-muxed) pins:

Power (VDD/VSS)

Clock and reset

JTAG debug

Boot mode select


6. Universal Serial Engine (USE)

The USE is a hardware finite state machine responsible for:

Protocol request handling

Bus quiescing before handover

Ownership enforcement

Timeout-based recovery

Electrical safety guarantees


FSM States (high level)
RESET → IDLE → REQUEST_CHECK → QUIESCE → ASSIGN → ACTIVE → RELEASE → IDLE


Only one protocol can be active per pin group at any time.

7. Reduced-Pin SPI Support

To enable SPI within a 2-pin group:

SPI is implemented in half-duplex reduced-pin mode

Single shared data line + clock

Controlled by USE for safe activation

This is a deliberate engineering trade-off to preserve pin virtualization.

8. Bus & Address Map (Summary)

AXI: CPU, memory, accelerator, DMA

APB: GPIO, UART, SPI, I2C, PINMUX, USE

Example address regions:

Boot ROM: 0x0000_0000

SRAM: 0x2000_0000

PINMUX + USE: 0x6000_0000

APB peripherals: 0x8000_0000


9. Verification Strategy

Directed RTL tests (no UVM)

Protocol switching stress tests

Conflict injection

Timeout and recovery scenarios

Waveform analysis using GTKWave

Focus is on system correctness, not protocol micro-optimizations.


10. Backend Flow

PDK: SkyWater 130 nm

Flow: OpenLane

Stages: Synthesis → Floorplan → P&R → DRC/LVS → GDSII

Layout inspection: KLayout

Backend is demonstration-grade, intended to show RTL-to-GDS capability.


11. Scope Discipline

CPU and accelerator IPs are reused

Custom RTL is limited to:

PINMUX

Universal Serial Engine

SoC integration logic

No modification of third-party IP internals


12. Why This Project Matters

This project demonstrates:

System-level SoC thinking

Hardware/software boundary awareness

Pin-count optimization strategies

Power-aware design

RTL-to-GDSII execution using real flows

