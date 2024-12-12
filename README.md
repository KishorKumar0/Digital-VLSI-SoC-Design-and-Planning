<p align="center">
<img src="https://www.vlsisystemdesign.com/wp-content/uploads/2024/05/vsdlogo.png" width="400" height="300">

# *Digital VLSI SoC Design and Planning*
Welcome to the OpenLane workshop! In this workshop, we will delve into the process of designing an Application Specific Integrated Circuit (ASIC) from the Register Transfer Level (RTL) to the Graphical Data System (GDS) file using the OpenLane ASIC flow. The flow is composed of several key steps, starting with an RTL file and culminating in a GDS file.

## Inception of open-source EDA, OpenLANE and sky130 PDK
### Overview of QFN-48 Chip, Pads, Core, Die, and IPs
**VSDSquadron** is a cutting-edge development board based on the RISC-V architecture that is fully open-source.This board presents an exceptional opportunity for individuals to learn about RISC-V and VLSI chip design utilizing only open-source tools, starting from the RTL and extending all the way to the GDSII. The possibilities for learning and advancement with this technology are limitless.RISC-V chips on these boards should be open for VLSI chip design learning, allowing you to explore Place&Route (PNR), standard cells, and layout design. The Caravel IC is a microprocessor chip that interfaces with other components on the board. The design of this chip, from the abstract level down to fabrication, is accomplished through the RTL-to-GDSII flow. The VSDsquadron consists of both a physical programmable circuit board (often referred to as a microcontroller) and software, or an IDE (Integrated Development Environment), that runs on a computer and is used to write and upload code to the physical board.
![vsdsquadron](Day1/vsdsquadron.jpg)
### Introduction to IC Design components
![vsdsquadron](Day1/IC_Component1.png)
1. **Die:**
    - The die is the actual silicon wafer where all the components of the IC are fabricated.
    - It serves as the substrate that houses the functional blocks (e.g., processor cores, memory, peripherals) interconnected by metal layers.
2. **Pads:**
    - Pads are metal contact points placed along the perimeter of the die.
    - Purpose:
      - Provide connections for power (vdd3v3, vdd1v8, vss) to ensure the IC operates correctly.
      - Enable communication of signals between the IC and external components through input/output pins (gpio, ser_tx, ser_rx, etc.).
    - Types of Pads:
      - Power Pads: Supply voltage levels (vdd1v8, vdd3v3) and ground connections (vss).
      - Signal Pads: Interface for general-purpose inputs/outputs (gpio), analog signals (adc_in, analog_out), and serial communication (ser_tx, ser_rx).
3. **Core:**
    - The core is the central area of the die, where the main functional logic is implemented.
    - It contains all the key circuits, including the processor core, memory, and other peripherals.
    - Signals from the core are routed to the pads for communication with external devices.
![vsdsquadron](Day1/IC_Component2.png)
1. **Macros:**
    - Macros are reusable, pre-designed blocks of functionality integrated into the chip.
    - These include:
      - RISC-V SoC: A RISC-V-based System-on-Chip (SoC) that serves as the primary computational engine of the IC.
      - GPIO Bank: A collection of General-Purpose Input/Output pins, used for interfacing with external peripherals like LEDs, buttons, and sensors.
      - SPI (Serial Peripheral Interface): A high-speed serial communication protocol used to connect the IC with external devices like flash memory or sensors.
2. **Foundry IPs:**
    - Foundry IPs are pre-designed, verified modules provided by the semiconductor foundry (the manufacturer of the chip).
    - These blocks ensure compatibility with the fabrication process and provide critical functionalities like:
      - SRAM: Static Random-Access Memory for storing data or instructions.
      - ADC (Analog-to-Digital Converter): Converts analog signals into digital values for processing.
      - DAC (Digital-to-Analog Converter): Converts digital data into analog signals for output.
