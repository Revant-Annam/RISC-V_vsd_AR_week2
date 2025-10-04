# VSDBabySoC - Week 2 Research

## About This Week

This repository explores the fundamentals of **System-on-Chip (SoC) design** through the functional modeling and simulation of the **VSDBabySoC**. 

### What is an SoC?

An SoC (System-on-Chip) is a complete computer system integrated onto a single chip, combining components like a CPU, memory, and peripherals to create a compact and power-efficient unit.

### What is VSDBabySoC?

The VSDBabySoC is a simplified, open-source learning platform designed to teach core SoC concepts. It integrates three essential IP blocks:

- 🧠 **RVMYTH Core**: A 32-bit RISC-V CPU that acts as the "brain" of the system
- ⏱️ **8x PLL (Phase-Locked Loop)**: A clock generator that provides the high-frequency "heartbeat" synchronizing the entire chip
- 🔊 **10-bit DAC (Digital-to-Analog Converter)**: An interface that translates the chip's digital output into real-world analog signals

---

## Repository Structure

This repository is organized into the following sections:

### 📚 [`Theory`]
Detailed theoretical concepts covering:
- SoC Integration principles
- Mixed-Signal Design fundamentals
- Multi-Voltage Domain management
- Functional Modeling methodology

**→ [Explore the Theory](./Theory/)**

### 🔬 [`Labs`]
Hands-on practical exercises and simulations:
- Step-by-step tutorials
- Simulation setups and testbenches complilation
- IP block integration 
- Verification of the integration

**→ [Start the Labs](./Labs/)**

---

## Key Learning Concepts

This project demonstrates essential SoC design principles:

**SoC Integration** - Connect and verify different digital and analog IP cores

**Mixed-Signal Design** - Simulate systems combining digital logic (CPU) and analog components (PLL and DAC)

**Multi-Voltage Domains** - Handle different voltage levels (1.8V core, 3.3V peripherals)

**Functional Modeling** - Validate architecture and logic at high-level before implementation, catching issues early and cost-effectively
