<p align="center">
<img src="https://www.vlsisystemdesign.com/wp-content/uploads/2024/05/vsdlogo.png" width="400" height="300">

# *Digital VLSI SoC Design and Planning*
Welcome to the OpenLane workshop! In this workshop, we will delve into the process of designing an Application Specific Integrated Circuit (ASIC) from the Register Transfer Level (RTL) to the Graphical Data System (GDS) file using the OpenLane ASIC flow. The flow is composed of several key steps, starting with an RTL file and culminating in a GDS file.

## Inception of open-source EDA, OpenLANE and sky130 PDK
### Overview of QFN-48 Chip, Pads, Core, Die, and IPs
**VSDSquadron** is a cutting-edge development board based on the RISC-V architecture that is fully open-source.This board presents an exceptional opportunity for individuals to learn about RISC-V and VLSI chip design utilizing only open-source tools, starting from the RTL and extending all the way to the GDSII. The possibilities for learning and advancement with this technology are limitless.RISC-V chips on these boards should be open for VLSI chip design learning, allowing you to explore Place&Route (PNR), standard cells, and layout design. The Caravel IC is a microprocessor chip that interfaces with other components on the board. The design of this chip, from the abstract level down to fabrication, is accomplished through the RTL-to-GDSII flow. The VSDsquadron consists of both a physical programmable circuit board (often referred to as a microcontroller) and software, or an IDE (Integrated Development Environment), that runs on a computer and is used to write and upload code to the physical board.

![vsdsquadron](Day1/vsdsquadron.jpg)

---

### Introduction to IC Design components
![vsdsquadron](Day1/IC_Component1.png)
1. **Die:**
    - The die is the actual silicon wafer where all the components of the IC are fabricated.
    - It serves as the substrate that houses the functional blocks (e.g., processor cores, memory, peripherals) interconnected by metal layers.
2. **Pads:**
    - Pads are metal contact points placed along the perimeter of the die.
    - **Purpose:**
      - Provide connections for power (`vdd3v3`, `vdd1v8`, `vss`) to ensure the IC operates correctly.
      - Enable communication of signals between the IC and external components through input/output pins (`gpio`, `ser_tx`, `ser_rx`, etc.).
    - **Types of Pads:**
      - **Power Pads:** Supply voltage levels (`vdd1v8`, `vdd3v3`) and ground connections (`vss`).
      - **Signal Pads:** Interface for general-purpose inputs/outputs (`gpio`), analog signals (`adc_in`, `analog_out`), and serial communication (`ser_tx`, `ser_rx`).
3. **Core:**
    - The core is the central area of the die, where the main functional logic is implemented.
    - It contains all the key circuits, including the processor core, memory, and other peripherals.
    - Signals from the core are routed to the pads for communication with external devices.

![vsdsquadron](Day1/IC_Component2.png)
1. **Macros:**
    - Macros are reusable, pre-designed blocks of functionality integrated into the chip.
    - These include:
      - **RISC-V SoC:** A RISC-V-based System-on-Chip (SoC) that serves as the primary computational engine of the IC.
      - **GPIO Bank:** A collection of General-Purpose Input/Output pins, used for interfacing with external peripherals like LEDs, buttons, and sensors.
      - **SPI (Serial Peripheral Interface):** A high-speed serial communication protocol used to connect the IC with external devices like flash memory or sensors.
2. **Foundry IPs:**
    - Foundry IPs are pre-designed, verified modules provided by the semiconductor foundry (the manufacturer of the chip).
    - These blocks ensure compatibility with the fabrication process and provide critical functionalities like:
      - **SRAM:** Static Random-Access Memory for storing data or instructions.
      - **ADC (Analog-to-Digital Converter):** Converts analog signals into digital values for processing.
      - **DAC (Digital-to-Analog Converter):** Converts digital data into analog signals for output.

---

### Introduction to RISC-V
RISC-V is an open-standard instruction set architecture (ISA) developed at the University of California, Berkeley, representing the fifth generation of RISC architecture. It is open source, fee-free, and based on established RISC principles. RISC-V supports a variety of hardware platforms and is integrated into many open-source operating systems and popular software toolchains.

The ISA is versatile, with a 32-bit fixed-length base instruction set and variable-length extensions allowing instructions to be in 16-bit increments. It supports 32-bit, 64-bit, and experimental 128-bit address space variants, though the 128-bit variant remains unfrozen due to limited practical use. Chips are connected to packages via bond wires.

![vsdsquadron](Day1/RISC-V_intro.png)
This diagram demonstrates how a simple C program is executed on a RISC-V architecture by translating it into assembly code, mapping the binary representation of the assembly code, and running it on the RISC-V layout.

1 **Writing a C Program:**
- A simple C program is written to perform a specific operation (e.g., swapping two variables in memory).
  
2.**Converting to Assembly Code:**
- The C code is compiled using the RISC-V GCC toolchain to generate assembly and binary machine code.
  
3.**Executing on RISC-V Layout:**
- The binary instructions are sent to the RISC-V processor core, where the processor executes them by decoding the instructions and interacting with the hardware.
    
#### Step-by-Step Explanation
**Step 1: Writing a C Program**
A simple C program is written to demonstrate the operation. For example, the program to swap two variables is as follows:
```
c

void swap(int* myl, int* mys) {
    int temp = myl[0];
    myl[0] = mys[0];
    mys[0] = temp;
}
```
This code uses basic memory operations to swap values between two arrays.

**Step 2: Compiling to RISC-V Assembly Code**
- The C program is compiled using the RISC-V GCC toolchain:
```
bash

riscv64-unknown-elf-gcc -o swap.o swap.c
```
The compiled binary (`swap.o`) is disassembled to generate the RISC-V assembly instructions:
```
bash

riscv64-unknown-elf-objdump -d swap.o
```
The disassembled output shows the equivalent RISC-V instructions:
```
assembly

addi  a5, sp, 48
sw    a5, 0(a0)
ld    a5, 8(a0)
...
```

This step converts the high-level C code into machine-level instructions that the RISC-V processor can understand.

**Step 3: Executing on RISC-V Processor**
- The binary representation of the assembly instructions is sent to the picorv32 RISC-V processor core for execution.
- The processor core decodes and executes the instructions:
    - Registers are initialized, and memory operations (load/store) are performed as per the instructions.
    - The `always` block in the Verilog implementation of `picorv32` handles the instruction decoding and execution cycle:
```
verilog

always @(posedge clk) begin
    is_lui <= instr_lui;
    is_add <= instr_add;
    if (mem_done) begin
        instr_latched <= mem_rdata[6:0] == 7'b0110011;
    end
end
```
**Step 4: Mapping to Physical Layout**
- The Verilog design of the RISC-V core (picorv32) is synthesized into a physical layout using the qflow toolchain.
- The layout includes:
    - **Standard cells:** The building blocks of the processor.
    - **Interconnections:** Signal routing between different cells.
This layout represents the physical implementation of the RISC-V processor, where binary instructions are processed to execute the program.

--

## How Application Software Runs on a Laptop or Computer Package (Chip)

![Application Software to Hardware](Day1/Application_Software_to_Hardware1.png)

This diagram explains the flow of how **application software** interacts with the **hardware** of a computer or laptop, using system software to convert the program into machine-executable instructions. The entire process involves **Operating System (OS), Compiler, Assembler, and Hardware**.

---

### Step-by-Step Execution Flow

#### 1. Application Software
- Application software includes programs such as:
  - **Mozilla Firefox**, **Microsoft Word**, **Acrobat Reader**, etc.
- These are user-level programs designed to perform specific tasks.

#### 2. System Software
System Software acts as a bridge between **Application Software** and the **Hardware**. It comprises three main components:

##### **a) Operating System (OS)**
- Handles core tasks such as:
  - Managing Input/Output (I/O) devices,  
  - Allocating memory, and  
  - Performing low-level system operations.  
- Converts the application software into **high-level code** (e.g., written in **C**, **C++**, **Java**, or **VB**).

##### **b) Compiler**
- The **compiler** translates the high-level programming code into instructions compatible with the **Instruction Set Architecture (ISA)** of the hardware.  
- The resulting output is stored as an **.exe** file.  
- **ISA** acts as an **abstract interface** between:
  - **Software (instructions)**, and  
  - **Hardware (processor/chip)**.  
- In this case, the **RISC-V ISA** is used.

##### **c) Assembler**
- The assembler takes the compiled **.exe file** and converts it into **machine-level binary code**.  
- Binary code consists of **1s and 0s** that the hardware can understand:  
  - Example: `01000110101`, `01010010010`.

#### 3. Hardware
- The binary instructions are sent to the **hardware** for execution.  
- The hardware (chip layout) consists of:
  - Routing paths,  
  - Data output pins (e.g., Dout1, Dout2),  
  - Clock signals (e.g., Clk Out), and  
  - Components like **DECAP1**, **DECAP2**, and more.  
- The hardware executes the instructions to **run the application software**.

---

### Key Concepts

1. **Application Software → System Software → Hardware**
   - Application software is converted into binary form and executed on hardware.

2. **RISC-V Architecture**
   - The **RISC-V ISA** determines how instructions are encoded and executed on the hardware.

3. **System Software Components**:
   - **Operating System**: Manages system-level tasks.  
   - **Compiler**: Translates high-level code to hardware-specific instructions.  
   - **Assembler**: Converts instructions into binary machine code.

4. **Instruction Set Architecture (ISA)**:
   - ISA serves as the bridge between **software instructions** and **hardware execution**.
  
---

![Application Software to Hardware](Day1/Application_Software_to_Hardware2.png)

This diagram shows that After the instruction and assembler stages, there is another interface before the hardware: the Register Transfer Level (RTL) description written in a Hardware Description Language (HDL). This RTL is used to convert instructions into machine code, which is then synthesized into a netlist containing gates and flip-flops.

#### 1. Instruction Set Architecture (ISA)
The ISA provides an abstract interface between software and hardware. It defines operations like arithmetic, logic, and control instructions.
- **Example Instruction:**
 ```
  assembly

  add x6, x10, x6

 ```
This means:
- Add the contents of register `x10` to register `x6` and store the result back in `x6`.

#### 2. Assembler
The Assembler translates ISA instructions into binary machine code. This binary code is the input to the hardware.
- **Machine Code:**
 ```
 mechine code

 01000110101  
 0101001010  
 ```
The binary code corresponds to the operation, source registers, and destination register

#### 3. Register Transfer Level (RTL)
The RTL is written in Hardware Description Language (HDL) like Verilog or VHDL. It describes how the hardware interprets and executes the instructions.
- **Example RTL Snippet (Verilog):**
 ```
 verilog

 module picorv32 #( 
     parameter [0:0] ENABLE_COUNTERS = 1, 
     parameter [0:0] ENABLE_REGS = 1
 );

always @(posedge clk) begin
    if (instr_valid) begin
        if (instr == "ADD") begin
            regfile[instr_rd] <= regfile[instr_rs1] + regfile[instr_rs2];
        end
    end
end
```

- **Key Highlights:**
  - The RTL code performs the ADD operation.
  - Registers instr_rs1 and instr_rs2 are read and summed.
  - The result is stored back in the destination register instr_rd.

#### Netlist Generation
The RTL code is synthesized into a netlist, which describes the hardware implementation using logic gates and flip-flops.
- **Netlist Example (Simplified View):**
 ```

 Copy code
 0000000000110101001000011000011
 ```
The netlist maps the operations to actual digital components like AND gates, flip-flops, and multiplexers.

- **Diagram Representation (from the image):**
  This synthesized design connects combinational logic and sequential logic (flip-flops) to implement the ADD instruction.

----

## Soc design and OpenLANE
### Introduction to all components of open-source digital asic design
1. **Digital Asic Design**
   Designing **Application-Specific Integrated Circuits (ASICs)** in an automated way involves several essential components. These elements work together to ensure a smooth and efficient design process.

   ![asiccomponent](Day1/asiccomponent.png)

   1. **RTL IP's (Register Transfer Level Intellectual Property):** RTL IPs are pre-designed modules or blocks that describe the behavior and structure of digital circuits at the RTL level. They are crucial for reusability and accelerating the design process.

    3. **EDA Tools (Electronic Design Automation Tools)**:
EDA tools provide software solutions for designing, verifying, and implementing ASICs. These tools handle various design stages, including synthesis, simulation, place-and-route, and timing analysis.

    4. **PDK DATA (Process Design Kit Data)**
PDKs contain all the process-specific information, including libraries, models, and layout rules, required to design ASICs for a particular semiconductor technology node. They ensure compatibility with the fabrication process.

3. **Open Source Digital ASIC Design**
   Designing an **Application-Specific Integrated Circuit (ASIC)** has traditionally been a costly and complex process, making it inaccessible for many. However, with advancements in **open-source tools** and resources, the dream of creating a fully functional ASIC has now become a reality.
   
   ![asicelement](Day1/asicelement.png)

    #### 1. RTL Designs:
    - RTL (Register Transfer Level) designs define the digital logic functionality of the ASIC.
    - Open-source platforms like:
      - librecores.org
      - opencores.org
      - github.com
      provide readily available RTL IPs for various applications.

    #### 2. EDA Tools (Electronic Design Automation Tools):
   - Open-source EDA tools help automate the ASIC design process, including synthesis, place-and-route, and timing analysis.
    - Popular tools include:
      - QFlow
      - OpenROAD
      - OpenLANE

    #### 3. PDK Data (Process Design Kit Data):
   - PDKs contain the process-specific libraries, layout rules, and models required for ASIC fabrication.
    - An example is the SkyWater 130nm PDK made available in collaboration with Google.
      - SkyWater PDK GitHub Repository

### Simplified RTL to GDSII Flow

The RTL to GDSII flow is a step-by-step process used in the semiconductor industry to transform a Register Transfer Level (RTL) design into a final GDSII layout, which can be used for chip manufacturing. 

![SimplifiedRTLtoGDSIIFlow](Day1/SimplifiedRTLtoGDSIIFlow.png)
    
#### Flow Steps
1. **Synthesis (Synth)**
The RTL design is converted into a gate-level netlist using a synthesis tool. This step maps the design to a specific technology library, ensuring it meets the desired timing, area, and power constraints.

2. **Floorplanning and Power Planning (FP+PP)**
The chip layout is defined at a high level. Key areas, such as cores and blocks, are positioned, and power distribution is planned to ensure reliable operation.

3. **Placement (Place)**
Standard cells are placed in the predefined floorplan locations. The placement tool optimizes the design for timing and minimizes wirelength.

4. **Clock Tree Synthesis (CTS)**
A clock distribution network is created to deliver a synchronized clock signal across the entire design while minimizing skew and ensuring proper timing.

5. **Routing (Route)**
The interconnections between cells and blocks are routed. This step ensures all signals are correctly connected and meets the timing, power, and design rule constraints.

6. **Sign-Off**
The final design is verified against manufacturing and performance requirements. Checks include timing analysis, design rule checking (DRC), and layout versus schematic (LVS) comparison.

#### Inputs and Outputs
- **Inputs:**
    - **RTL:** The design description in a high-level hardware description language (HDL), such as Verilog or VHDL.
    - **PDK:** The Process Design Kit containing the technology-specific information required for the design.
- **Outputs:**
  - **GDSII:** The final layout format used for fabrication.
 
---
### Introduction to OpenLANE and Strive chipsets

With the release of the open-source PDK at eFabless, it will drive the creation of its own reference open-source ASIC implementation methodology called **OpenLane**. 

#### Open Source ASIC Flow Challenges 

Using open-source EDA tools for ASIC design presents unique challenges compared to commercial solutions. The key issues include **tools qualification**, where open-source tools may lack formal validation for production use, **tools calibration**, which requires manual tuning to match process node specifications, and **missing tools**, as some essential features like advanced DRC, LVS, or power analysis may not be fully available. While OpenLane is a promising step forward, addressing these challenges is crucial to achieving industry-grade reliability and efficiency.

#### OpenLane and striVe: Open-Source ASIC Innovation

**OpenLane** started as an **open-source ASIC design flow** aimed at enabling a **true open-source tape-out experiment**. It provides a complete methodology for RTL-to-GDSII implementation using open-source tools and the SkyWater 130nm process.  

Additionally, **striVe** is a family of **open-everything SoCs**, built entirely on **open PDKs, open EDA tools, and open RTL designs**. This initiative demonstrates the feasibility of a fully open-source semiconductor design ecosystem.  
<p align="left">
    <img src="Day1/striVe_block_diagram.jpg" width="600" />
    <img src="Day1/striVe_GDSII.png" width="350" />
</p>
The image illustrates a **striVe SoC block diagram**, detailing its components such as the **RISC-V core, SRAM, SPI interface, PLL, and GPIOs**, along with a **GDSII layout representation** of the finalized ASIC. This showcases the **practical application of OpenLane in real silicon tape-outs**.

#### striVe SoC Family

The **striVe SoC family** is a set of **open-source System-on-Chip (SoC) designs** created using **OpenLane** and the **SkyWater 130nm PDK**. These designs serve as **reference implementations** for open-source ASIC development.  

1. **Key Highlights**  
- Developed as **open-source SoCs** for research, prototyping, and education.  
- Uses **OpenLane**, an open-source ASIC flow, for RTL-to-GDSII implementation.  
- Built on the **SkyWater 130nm process**, supported by Google and eFabless.  
- Demonstrates **different memory architectures**, including synthesized SRAM and OpenRAM.  
- Aims to **make ASIC design accessible** to the wider hardware community.  

2. **striVe SoC Variants and Features**  

| **SoC Variant** | **Technology Node** | **Memory Architecture** | **Special Features** |  
|---------------|-----------------|---------------------|----------------|  
| **striVe**   | Sky130 SCL       | Synthesized 1 KByte SRAM | Baseline variant |  
| **striVe 2** | Sky130 SCL       | 1 KByte OpenRAM block | Uses OpenRAM instead of synthesized SRAM |  
| **striVe 2a** | Sky130 SCL       | Same as striVe 2 | Single-chip core module |  
| **striVe 3** | OSU SCL          | Synthesized 1 KByte SRAM | Uses OSU Standard Cell Library |  
| **striVe 5** | Sky130 SCL       | 8 × 1 KByte OpenRAM banks | Higher memory capacity |  
| **striVe 6** | Sky130 SCL       | Same as striVe 2 | Includes Design for Testability (DFT) |  

3. **Supported by Open-Source Silicon Community**  
- **SkyWater** – Provides open-source **PDK**.  
- **Google** – Supports open ASIC initiatives.  
- **OpenROAD** – Contributes open EDA tools.  
- **eFabless** – Enables open-source chip manufacturing.

Here's how you can document these images in your GitHub README file:

---

#### OpenLANE ASIC Flow

1. **Goal**

    The main objective of OpenLANE is to **produce a clean GDSII file with no human intervention** (also known as **no-human-in-the-loop** automation). This means that the generated layout should meet all verification requirements without manual fixes.

2. **Clean GDSII Requirements:**
    - **No LVS (Layout vs. Schematic) Violations**
    - **No DRC (Design Rule Check) Violations**
    - **Timing Violations**: Work in progress (WIP)

3. **PDK Support**
OpenLANE is primarily **tuned for the SkyWater 130nm Open PDK** but also supports other process design kits:
    - **SkyWater 130nm Open PDK**
    - **XFAB180**
    - **GF130G**


4. **Containerized Flow**
To ensure a smooth and reproducible experience, OpenLANE is **containerized**, meaning:
    - It works **out of the box** without requiring complex dependencies.
    - Instructions for building and running it natively will be provided.

5. **Features**  

    1. **Tuned for SkyWater 130nm Open PDK**  
        - OpenLANE is optimized for the **SkyWater 130nm** open-source **Process Design Kit (PDK)**.  
        - This means that the toolchain is specifically set up to generate designs that can be **fabricated using SkyWater’s 130nm technology node**.  
        - The 130nm process is widely used in research and open-source silicon projects due to its balance of performance, power, and cost-effectiveness.  

    2. **Support for Multiple PDKs**  
    - In addition to **SkyWater 130nm**, OpenLANE also supports:  
      - **XFAB180** – An **180nm** PDK, often used for mixed-signal and analog ICs.  
      - **GF130G** – A **130nm** PDK from **GlobalFoundries**, providing an alternative fabrication option.  
    - The ability to work with multiple PDKs gives designers more flexibility when choosing a manufacturing process.  

    3. **Containerized Implementation**  
        - OpenLANE is packaged as a **Docker container**, which allows for **quick and easy deployment** across different systems.  
        - Benefits of using a containerized approach:  
          - **No manual installation hassles** – The entire toolchain is bundled and pre-configured.  
          - **Cross-platform support** – Works on **Linux, macOS, and Windows** with minimal setup.  
          - **Consistent Environment** – Ensures the same configurations and dependencies across different users.  
        - In the future, **instructions for native (non-containerized) builds** will be provided for users who prefer to install the tools directly.  

    4. **Can Be Used to Harden Macros and Chips**  
        - **Hardening** refers to converting a **soft macro** (a high-level digital design in Verilog) into a **physical layout** that can be fabricated.  
        - OpenLANE supports hardening **both individual IP blocks (macros) and complete chips**, making it useful for various design scales.  

    5. **Two Modes of Operation**  
        1. **Autonomous Mode** 🛠️  
           - Runs automatically using **default settings** and pre-configured design flows.  
           - Ideal for **quick prototyping** or users who prefer an automated workflow.  
        2. **Interactive Mode**
           - Allows users to **manually adjust** settings at different stages of the ASIC design process.  
           - Useful for **fine-tuning designs** based on specific constraints like power, performance, or area (PPA).  

    6. **Design Space Exploration (DSE)**  
        - **DSE helps find the best set of tool configurations** to optimize the design.  
        - It systematically explores different options to **improve power, performance, and area (PPA)**.  
        - This is especially useful for achieving the best trade-offs when working with different design constraints.  

    7. **Large Number of Design Examples**  
        - OpenLANE includes **43 pre-verified design examples**, each showcasing different capabilities and best practices.  
        - These examples help new users **learn faster** by providing ready-to-use test cases.  
        - **More designs will be added soon**, expanding the available resources for learning and experimentation.  

### Introduction to OpenLANE detailed ASIC design flow

#### Overview  
OpenLane is an open-source toolchain for **ASIC physical design automation** (RTL-to-GDSII) targeting **open PDKs** like **SkyWater 130nm**. It integrates multiple open-source tools for synthesis, placement, routing, and verification.  

The following diagram illustrates the **OpenLane Flow**, which converts an RTL description into a **manufacturable layout**:  

![OpenLane Flow](Day1/openlane-flow.png)  

#### OpenLane Flow Explanation  

A. **RTL Synthesis and Static Timing Analysis (STA)**
1. **RTL Synthesis** (using **Yosys + ABC**) converts the Verilog RTL design into a **gate-level netlist**.  
2. **Static Timing Analysis (STA)** (using **OpenSTA**) checks the timing constraints of the generated netlist.  
3. **Synthesis Exploration** is an optional feature that evaluates different synthesis strategies:  
    - OpenLane provides **four default synthesis strategies** to explore different **area-delay trade-offs**.  
    - Users can also define their **own custom synthesis strategies**.  
4. STA results are **visualized graphically** on an **HTML dashboard** for further analysis.  

B. **Insertion of Design-for-Testability (DFT) Structures**
- The **Fault** tool can optionally **insert scan chains** and **test IO ports** into the synthesized netlist.  
- This step ensures that the fabricated chip can be **tested effectively** after manufacturing.  
- A variant of **strIve chips** includes DFT structures for better OpenLane integration.  

C. **Physical Implementation**
- **Floorplanning & Placement:** The design undergoes **floorplanning** (defining chip layout), **placement**, and **clock tree synthesis (CTS)** using **OpenROAD**.  
- **Optimization:** Post-placement optimization is performed using **OpenPhySyn**.  
- **Diode Insertion:** To prevent **antenna effects**, **custom scripts** insert diodes in the layout.  
- **Logic Equivalence Check (LEC)** (using **Yosys**) verifies that **post-optimization netlists** are functionally equivalent to the original synthesized netlist.  
- **I/O Pin Placement Modes:**  
   - **Default Mode:** Uses **OpenROAD** for automatic pin placement.  
   - **Custom Mode:** Allows manual pin placement for **full control over pin locations**.  
   - **Contextualized Mode:** Automatically optimizes I/O placement **based on SoC integration requirements**.  
- The final output of this stage is a **routed DEF** (Design Exchange Format file), ready for evaluation.  

D. **Post-Routing Evaluation & Verification**
- **Design Rule Check (DRC)** and **Layout-vs-Schematic (LVS)** verification are performed using **magic** and **netgen**.  
- **Antenna Rule Check (ARC)** is conducted using either **OpenROAD’s ARC tool** or **magic**.  
- **RC Extraction:** Extraction of **parasitic resistances and capacitances** using **SPEF EXTRACTOR**.  
- **Final STA Analysis:** Another round of **static timing analysis** is performed to generate **accurate timing reports** for the **physical layout**.  

**Final Output: GDSII and LEF Files**
The final output of the OpenLane flow includes:  
- **GDSII File:** Standard format for sending designs to **fabrication**.  
- **LEF File:** Layout Exchange Format file used for integration into **larger designs**.  

Here's a structured explanation of the terms shown in the images, which you can use in your GitHub README file:  


1. **Synthesis Exploration**  
Synthesis exploration involves analyzing different synthesis strategies to optimize area, power, and timing of digital designs.  
- The scatter plot displays the trade-off between **area** (measured in μm²) and **delay** (measured in ps).  
- Each point represents a synthesis strategy (S1 to S8), where:  
  - **Lower area** is preferred for cost reduction.  
  - **Lower delay** is preferred for better performance.  
  - The ideal synthesis strategy minimizes both area and delay.  

2. **Design Exploration**  
Design exploration is used to compare different placement and routing strategies to find the best configuration for the design.  
- The table presents various design configurations, including:  
  - **Design Name**: The name of the circuit being synthesized (e.g., AES, CORDIC).  
  - **Runtime**: Total execution time for the flow.  
  - **Cell Count**: Number of standard cells in the design.  
  - **TR Vios (Timing Rule Violations)**: Indicates how many timing constraints were violated.  
  - **FP_CORE_UTIL**: Floorplan core utilization percentage.  
  - **Routing Strategy**: The approach used for routing.  
  - **GLB_RT_ADJUSTMENT**: A global routing parameter affecting congestion and optimization.  

3. **OpenLane Regression Testing**  
Regression testing ensures the OpenLane flow remains stable and produces consistent results across multiple designs.  
- The table lists results for ~70 designs, comparing current runs to previously known best runs.  
  - **JPEG Encoder, AES, TEA, CPU, etc.**: These are benchmark designs tested in OpenLane.  
  - **Runtime**: How long each design takes to complete the OpenLane flow.  
  - **Cell Count**: The number of logic cells in the design.  
  - **TR Vios (Timing Violations)**: Ideally, should be zero, indicating no violations.  
  - Green shading indicates successful designs with zero violations.  
