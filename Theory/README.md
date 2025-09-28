# Fundamentals of SoC Design and the BabySoC Learning Model

## What is a System-on-Chip (SoC)?

A **System-on-Chip (SoC)** is a single integrated circuit (IC) that combines **all the essential components of a computer system** into one chip. Instead of using multiple chips for CPU, memory, input/output controllers, and communication interfaces, an SoC integrates them into a **compact, power-efficient unit**.

It is essentially a **complete electronic system built into a single silicon die**, optimized for a specific application such as mobile computing, IoT, automotive electronics, or high-performance servers.

**SoC basic architecture:**

<p align="center">
  <img width="355" height="322" alt="image" src="https://github.com/user-attachments/assets/bba65c83-9243-4167-894b-1795e9cfd94c" />
</p>

---

## **Key Characteristics of an SoC**

1. **Integration**

   * All major system components are present on one chip.
   * Reduces size, cost, and power consumption compared to multi-chip systems.

2. **Application-Specific Design**

   * Many SoCs are tailored to specific use cases (e.g., smartphone SoCs with GPUs, camera processors, AI accelerators).

3. **Heterogeneous Computing**

   * Modern SoCs integrate not only CPUs but also GPUs, DSPs (Digital Signal Processors), and AI/ML accelerators for parallel tasks.

4. **Power Efficiency**

   * Essential for battery-powered devices like smartphones, wearables, and IoT nodes.

---

## **Main Components of a System-on-Chip (SoC)**

### 1. **Central Processing Unit (CPU)**

* **Role:** The CPU is the **brain of the SoC**. It fetches instructions, decodes them, and executes operations.
* **Features:**

  * Usually based on **RISC architectures** (ARM, RISC-V, MIPS).
  * Can have **single-core or multi-core** CPUs. A single-core CPU has one processing unit that executes tasks sequentially, while a multi-core CPU has multiple cores, allowing it to perform tasks in parallel, leading to significantly improved performance and multitasking capabilities.
  * Includes **cache memory (L1/L2/L3)** for high-speed data access. Cache memory is a small, high-speed, and volatile type of computer memory that stores frequently used data and instructions.

---

### 2. **Memory**

* **Role:** Stores instructions and data for the CPU and other processing units.
* **Types:**

1. **RAM** Integrated within the SoC for quick access. RAM is for temorary storage and consume a large area.
2. **ROM** It is used to store permanent data. The data is not erased even when the system is powered off.

---

### 3. **Interconnect / Bus System**

* **Role:** Acts as the **nervous system** of the SoC, connecting CPU, memory, and peripherals.
* **Types of interconnects:**

  * **Bus-based (AMBA):**

    * **AHB (Advanced High-performance Bus):** For high-speed cores.
    * **APB (Advanced Peripheral Bus):** For low-speed peripherals.
    * **AXI (Advanced eXtensible Interface):** For high-bandwidth communication.
  * **Network-on-Chip (NoC):** Used in complex, multi-core SoCs for parallel data flow.

---

### 4. **Peripherals and I/O Interfaces**

* **Role:** Allow the SoC to communicate with the **outside world**.
* **Types:**

1. **Communication Interfaces:**

    * UART (serial communication)
    * SPI & I2C (sensor and chip-to-chip communication)
    * USB, Ethernet, PCIe (high-speed external connectivity)
2. **Timers & Counters:** For real-time task scheduling.
3. **Interrupt Controllers:** Manage hardware interrupts efficiently.
4. **GPIOs (General Purpose I/O):** Simple on/off pins to control devices like LEDs, motors, or switches.

---

### 5. **Specialized Processing Units**

Modern SoCs often include **dedicated accelerators** for performance and efficiency:

* **GPU (Graphics Processing Unit):**

  * Handles rendering of images, video playback, and gaming.
  * Also used for parallel computing tasks.

* **DSP (Digital Signal Processor):**

  * Optimized for **real-time signal processing** like audio, video, and communication signals.
  * Performs operations like FFTs, filtering, and compression.

* **AI/ML Accelerators:**

  * Perform matrix multiplications for deep learning tasks.
  * Found in modern SoCs for voice recognition, image processing, and autonomous driving.

---

### 6. **Power Management Unit (PMU)**

* **Role:** Controls power delivery and consumption within the SoC.
* **Techniques:**

  * **DVFS (Dynamic Voltage and Frequency Scaling):** Adjusts CPU speed based on workload.
  * **Power gating:** Shuts down unused blocks.
  * **Clock gating:** Disables clock for idle circuits.

---

## **Advantages of SoC**

1. **Compact Size:** Combines CPU, memory, peripherals, and accelerators on a single chip, saving space.
2. **Low Power Consumption:** On-chip integration and optimized communication reduce energy usage.
3. **High Performance:** Fast interconnects and specialized units (GPU, DSP, NPU) improve computation speed.
4. **Cost-Effective:** Fewer external components reduce manufacturing and assembly costs.
5. **Customizable:** Can be tailored for specific applications like smartphones, IoT devices, or automotive systems.
6. **Reliability:** Fewer external connections reduce signal integrity issues and improve robustness.

---

## **Disadvantages of SoC**

1. **Complex Design:** Integrating multiple components on a single chip is challenging and requires advanced design tools.
2. **Limited Upgradability:** Hard to upgrade individual components like CPU or memory once fabricated.
3. **Thermal Management Issues:** High integration can lead to hotspots that are difficult to cool.
4. **Higher Initial Cost:** NRE (Non-Recurring Engineering) and mask costs for custom SoCs are high.
5. **Debugging Difficulty:** Fault isolation is harder because everything is on a single chip.

---

## SoC Design flow:

<p align ="center">
  <img width="433" height="652" alt="image" src="https://github.com/user-attachments/assets/69ceb5c2-4823-44e3-b0aa-bb46cac5325d" />
</p>

### Hardware path:
1. **Specification** – Define SoC's features, performance, power, and functionality. 
2. **Architecture Design (Hardware/Software Partitioning)** – Division of tasks will be handled by hardware (for speed and efficiency) and software(for flexibility).
3. **High level modelling** - Model of the hardware created using a high-level language (like SystemC) to simulate and validate the architecture before RTL design.
4. **RTL Design** – Write functionality in HDL (Verilog/VHDL).
5. **Functional Verification** – Design is tested to ensure the hardware logic is free of bugs. At this stage, HW/SW Co-simulation takes place, where the hardware simulation is run together with the system software. This allows for early integration testing.
6. **Logic Synthesis and DFT** – Convert RTL into a gate-level netlist and design is made easier for testing.
7. **Placement and routing** – Arrange IP blocks and place logic gates on silicon and attach them using metal wires.
8. **Timing verification and sign off** – Build balanced clock distribution ensuring no violation.
10. **Physical Verification** – Check timing closure, DRC, LVS for correctness.
11. **Tape-out** – Final design (GDSII) sent to fab.
12. **Fabrication & Packaging** – Manufacture, cut, package, and test chips.

### Software Path:
This path runs in parallel with the hardware design.

1. **Software Development** - Low-level firmware, device drivers, operating system, and applications that will run on the SoC's processor cores.
2. **Software Testing and Refinement** - The software is tested, often against the hardware simulation (in co-simulation), and refined to fix bugs and improve performance.
3. **Final Code** - The fully verified software is ready to be loaded onto the physical chip once it's available.

* **Manufacturing** - The chip is physically fabricated on a silicon wafer at a foundry.
* **Post-Silicon Validation and Integration** - The Final Code is loaded on the chips and the hardware and software integration is tested.
* **Mass Production** - Once the chip is fully validated and proven to work, it's approved for manufacturing in high volumes.

---

## VSDBabySoC

It's a small, complete **System-on-Chip (SoC)** built around the popular and open-source **RISC-V** architecture. The purpose of the BabySoC is to test 3 open source IP cores together and analyze the analog part of it.

**Block diagram:**

<img width="2270" height="1260" alt="image" src="https://github.com/user-attachments/assets/0834cd6a-d241-4808-a123-db239a0ee82e" />


### Key Components

1.  **CPU:** The **RVMYTH** microprocessor. This is the central processing unit that executes instructions and runs programs.
2.  **Clock generator and controller:** An **8x-PLL** (Phase-Locked Loop). This generates a stable, high-frequency clock signal, which synchronizes all operations on the chip.
3.  **Output:** A **10-bit DAC** (Digital-to-Analog Converter). This helps the chip to communicate with other analog devices. A DAC converts digital signal (input) into real-world analog signals.

### RVMYTH:
RVMYTH is an open-source, 32-bit microprocessor core developed by VLSI System Design (VSD) Corp. It is based on the popular RISC-V instruction set architecture and is specifically designed to be simple, clear, and easy to understand, making it an ideal processor for educational purposes. It implements the RV32I instruction set. The pipeline for the execution of the instructions is IF (instruction fetch) → ID (Instruction Decode) → EX (Execute) → MEM (Memory Access) → WB (Write Back). RVMYTH has a clock input which initiates the instruction execution. The clock is given by the PLL. It uses the SPI interface to communicate with an external device. The output of the RVMYTH is fed to a DAC. 

### PLL:
A PLL (Phased locked loop) is a control system that generates a stable, high-frequency output clock from a slower, stable input reference clock. PLL works using a feedback loop that constantly corrects itself. It has three main parts:

1. Phase Detector: This component compares the output clock with the input reference clock. It generates an error voltage that represents the difference in their timing (phase).
2. Loop filter: This is a low pass filter which ensures that there is no noise passes to the next stages from the input signal.
3. Voltage-Controlled Oscillator (VCO): This generates the final output clock. The frequency of the clock it produces is controlled by the voltage it receives.
4. Feedback Loop: A feedback loop generally consists of a frequency divider. The loop generates a high-frequency signal with a Voltage-Controlled Oscillator (VCO) and then immediately uses a divider to slow that signal down by a set amount, for example, by 8. This slowed-down signal is then compared to the original slow reference clock. To make the two signals match perfectly, the feedback loop forces the VCO to run exactly 8 times faster than the reference. Thus creating a high frequency clock from a slower reference signal.

Here **8x-PLL** means it uses a frequency divider of 8, thus generating a clock 8 times faster. 

<p align="center">
  <img width="327" height="146" alt="image" src="https://github.com/user-attachments/assets/5c038b8f-2774-4fa8-9aec-3761e83df356" />
</p>


#### Why PLL:
* **Clock Speed Multiplication:** SoCs need very high-speed clocks, but the stable external crystal oscillators that provide a reference signal are slower. The PLL multiplies this slow reference to the high frequency the chip actually needs.

* **On-Chip Clock Distribution:** A single external crystal oscillator isn't powerful enough to send a clean clock signal to millions of components across a large chip. The on-chip PLL acts as a powerful hub, generating a strong internal clock that can be effectively distributed throughout the entire SoC with minimal errors.

* **Jitter and Noise Reduction:** The PLL's feedback loop actively filters and cleans the clock signal. This reduces jitter (tiny variations in clock timing), resulting in a more stable clock for the system, which is critical for high-speed operations.

### DAC:
A DAC (Digital-to-Analog Converter) is a circuit that converts digital data into a real-world, continuous analog signal. It acts as a bridge between the digital world of processors and the analog world we live in. A DAC receives a binary number as its input. Each bit in this number has a specific weight. The DAC creates an output voltage that is the sum of the weights of all the bits that are '1'. The Most Significant Bit (MSB) has the highest weight, and the Least Significant Bit (LSB) has the smallest. 

<p align="center">
  <img width="293" height="325" alt="image" src="https://github.com/user-attachments/assets/64e1b925-eefd-4601-9065-82b473c38d0c" />
</p>


#### Characteristics of a DAC are:

* **Resolution (in bits):** It is the number of distinct steps the output can have. More bits mean smaller steps and a more accurate, smoother analog signal. A 10-bit DAC has 2<sup>10 </sup> or 1,024 steps.

* **Speed (Sample Rate):** How quick the DAC can perform a new conversion. Higher speeds are needed to accurately reproduce high-frequency signals.

### Overall Architecture of VSDBabySoC:

It is a mixed signal SoC as it has contains both digital logic (the `rvmyth` processor) and analog components (the PLL and DAC). It is operating at multiple voltages (1.8V and 3.3V) as shown in the block diagram. It consists of Level shifters (LS) which take a signal that represents a logical '1' at a low voltage and convert it to a signal that represents a logical '1' at a higher voltage, or vice versa. These are required as the peripherals (SPI and DAC) are taking in 3.3V at their ports. The processor `rvmyth` is working at 1.8V to reduce the power losses. While the peripherals need to have a stable signal which is the reason they operate at different voltages. 

## Why BabySoC is a simplified model for learning SoC concepts

VSDBabySoC simplifies the system to just three essential IP cores: the `rvmyth` CPU, the `avsdpll_1v8` PLL clock generator, and the `avsddac_3v3` DAC interface. This avoids the complexity of a commercial chip which can have dozens of components but also ensures that it helps understand the basics of SoC. This design helps to learn the most important concepts. It is not just a CPU in isolation; we can solving the real-world problem of integrating separate digital and analog blocks that operate at different voltages (1.8V and 3.3V).

Everything is fully open source. Unlike in commercial design where many IPs are black boxes, we can study the actual hardware source code for every single part of the BabySoC, which gives a transparent and deep understanding of how it truly works. 

## Role of Functional Modelling

Before moving into detailed RTL coding and physical implementation, SoC design starts with **functional modelling**:

### Definition
Functional modeling is a crucial early-stage process in SoC design where a high-level, executable model of the chip is created to define and verify what the system should do, long before deciding how it will be built with actual hardware.

### Purpose
* **Early Validation of Concept:** Functional modelling creates a high-level simulation of the SoC before writing RTL code. It checks whether the intended architecture and data flow work logically.

* **Bug Detection at Low Cost:** Finding design flaws early in modelling is much cheaper and faster than fixing them after RTL or physical design. Saves huge effort before moving to complex stages like synthesis and layout. As it is writtern in high level language the abstraction is less which makes the design understandable. 

* **Performance Estimation:** Functional models help estimate latency, throughput, and power trends. Designers can see if the architecture will meet the specification goals.

* **Guidance for RTL Development:** Once the high-level behaviour is validated, RTL coders have a clear reference model to follow. Reduces ambiguity and accelerates RTL design.

## Summary

- A **System-on-Chip** integrates CPU, memory, peripherals, and interconnect into one silicon die
- The **BabySoC** offers a simplified platform to practice SoC fundamentals, reducing complexity while maintaining the essential components of a SoC.
- **Functional modelling** acts as the first verification step in SoC design, ensuring correctness before diving into detailed RTL design and physical layout

By working with BabySoC using simulation tools, learners gain practical experience in **SoC design methodology**, laying the groundwork for advanced topics like RTL design, synthesis, and physical implementation.

---
