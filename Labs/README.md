# Week 2 Lab

The necessary simulation toolchain, including **Icarus Verilog** and **GTKWave**, was successfully installed and configured as part of the Week 0 preparatory tasks.

The primary objective for this week is to build a solid understanding of **System on Chip (SoC) fundamentals**, which is done in the Theory folder in this repository. This theoretical knowledge will be complemented by a practical exercise focused on the functional modeling of the **BabySoC**. The goal is to practice functional modeling of the BabySoC using simulation tools (Icarus Verilog & GTKWave).

## Recap of functional modeling:

To recap, functional modeling acts as a high-level design blueprint, describing the SoC's intended behavior — the "what" of the SoC functions — and allowing for rapid simulation and the correction of logical errors at the earliest and most cost-effective stage of development.

## File structure:

Here is the project structure with brief descriptions formatted as comments.

```
VSDBabySoC/           # Project root directory
├── src/              # Main directory for all source code
│   ├── include/      # Holds reusable Verilog header files (.vh)
│   │   └── sandpiper.vh  # Header with macros & definitions, for TL-Verilog
│   ├── module/       # Contains all Verilog hardware module files (.v)
│   │   ├── vsdbabysoc.v  # Top-level module; integrates all components into the SoC
│   │   ├── rvmyth.v      # The RISC-V CPU core, the "brain" of the SoC
│   │   ├── avsdpll.v     # PLL (Phase-Locked Loop) module for clock generation
│   │   ├── avsddac.v     # DAC (Digital-to-Analog Converter) module
│   │   └── testbench.v   # Simulation testbench to verify the SoC's functionality
└── output/           # Directory for all generated files (e.g., waveforms, logs)
    └── compiled_tlv/ # Stores intermediate files from the TL-Verilog compiler
```

## Starting with the project:

### 1. Clone the Git repository

In the home directory, we can use this command to clone and enter the directory. Further commands to verify the file structure are shown in the image below.

```bash
# Cloning the git repository
git clone https://github.com/manili/VSDBabySoC.git
cd VSDBabySoC

# Installing required packages
sudo apt update
sudo apt install python3-venv python3-pip
```

...the image for the cd VSDBabySoC

### 2. Converting the .tlv file to .v

If we don’t convert the `rvmyth.tlv` to `rvmyth.v`, we might encounter this error when we try to do the pre_synth_sim.
...paste the image of the error.

Using the following commands, we can create a Python virtual environment and convert the file with the help of `sandpiper-saas`. The `sandpiper-saas` must be installed in the virtual environment.

```bash
# Step 1: In ../VSDBabySoC directory create a new virtual environment and install sandpiper-saas
python3 -m venv sp_env
source my_env/bin/activate
pip install pyyaml click sandpiper-saas

# Step 2: Convert rvmyth.tlv to Verilog
sandpiper-saas -i ./src/module/*.tlv -o rvmyth.v --bestsv --noline -p verilog --outdir ./src/module/
```

...screenshot for the conversion.

### 3. Pre-synthesis simulation

If the `pre_synth_sim` directory is not present in `../VSDBabySoC/output/`, then use the command:

```
mkdir -p output/pre_synth_sim
```

Then we can compile the Verilog files in the `~/VLSI/VSDBabySoC/src/module` directory using the command:

```bash
iverilog -o ~/VLSI/VSDBabySoC/output/pre_synth_sim/pre_synth_sim.out -DPRE_SYNTH_SIM -I ~/VLSI/VSDBabySoC/src/include -I ~/VLSI/VSDBabySoC/src/module ~/VLSI/VSDBabySoC/src/module/testbench.v
```

Explanation of the command:

* **`iverilog`** - Compiles Verilog source code and creates an executable file that can be run to simulate the design.

* **`-o ~/VLSI/VSDBabySoC/output/pre_synth_sim/pre_synth_sim.out`**

  * The **`-o`** flag specifies the **output file**.
  * The path tells the compiler to name the compiled output `pre_synth_sim.out` and save it inside the `~/VLSI/VSDBabySoC/output/pre_synth_sim/` directory. This `.out` file is the simulation executable.

* **`-DPRE_SYNTH_SIM`**

  * The **`-D`** flag is used to **define a macro**.
  * This specific command defines the macro `PRE_SYNTH_SIM`. This is done to conditionally include or exclude blocks of code during compilation.

* **`-I ~/VLSI/VSDBabySoC/src/include`** and **`-I ~/VLSI/VSDBabySoC/src/module`**

  * The **`-I`** flag specifies an **include directory**.
  * This tells the compiler where to search for files that are referenced with an `` `include `` directive in our Verilog code. Here it is used for including the header files (../include directory) and Verilog files (../module directory). By providing these paths, we don’t have to specify the full path for every included file.

* **`~/VLSI/VSDBabySoC/src/module/testbench.v`**

  * This is the **top-level Verilog file** for the compilation. The compiler starts with this file and automatically finds and compiles all the other modules that are instantiated within it (like `vsdbabysoc`, etc.).

> **Note:** If you find the Makefile in `../VSDBabySoC`, then you can directly use the commands below:
>
> ```
> cd ~/VLSI/VSDBabySoC
> make pre_synth_sim
> ```

### 4. Analyzing the waveform:

---
