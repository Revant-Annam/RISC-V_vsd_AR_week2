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

#### i. **`avsdpll` 8x PLL:**
This is the block diagram of `avsdpll` :
<img width="800" height="450" alt="image" src="https://github.com/user-attachments/assets/e5c5b873-9f5f-44e8-9a44-98c3110b2ef3" />

* **Input and Output Ports** -

    * **Input Ports**:
        * `REF`: This is the **Reference Clock** input. The model uses the rising edge of this signal to measure its period and adjust the output clock.
        * `ENb_VCO`: This is the **VCO Enable** signal. When this signal is high (`1'b1`), the clock generation is enabled. When it's low, the output clock is forced to 0.
        * `VCO_IN`, `ENb_CP`: These ports are included in the module definition but are **not used** in this simplified behavioral model. In a more complex model, they would be used to control the Voltage-Controlled Oscillator and Charge Pump.

    * **Output Ports**:
        * `CLK`: This is the main **Clock Output**. It's the final, high-frequency clock signal generated by the PLL.

* **Internal Wires and Variables** - 

    * `period`: A `real` number that stores the current period of the output `CLK` in nanoseconds. This value is dynamically updated by the locking logic.
    * `lastedge`: A `real` number that stores the simulation time of the previous rising edge of the `REF` clock.
    * `refpd`: A `real` number that stores the measured period of the `REF` clock.

* **Verilog Code** -
    ```verilog
    module avsdpll (
   output reg  CLK,
   input  wire VCO_IN,
   input  wire ENb_CP,
   input  wire ENb_VCO,
   input  wire REF
    );
       real period, lastedge, refpd;

       initial begin
          lastedge = 0.0;
          period = 25.0; // 25ns period = 40MHz
          CLK <= 0;
       end

      // Toggle clock at rate determined by period
       always @(CLK or ENb_VCO) begin
          if (ENb_VCO == 1'b1) begin
             #(period / 2.0);
             CLK <= (CLK === 1'b0);
          end
          else if (ENb_VCO == 1'b0) begin
             CLK <= 1'b0;
          end 
          else begin
             CLK <= 1'bx;
          end
       end
   
       // Update period on every reference rising edge
       always @(posedge REF) begin
          if (lastedge > 0.0) begin
             refpd = $realtime - lastedge;
             // Adjust period towards 1/8 the reference period
             //period = (0.99 * period) + (0.01 * (refpd / 8.0));
             period =  (refpd / 8.0) ;
          end
          lastedge = $realtime;
       end
    endmodule
    ```

* **Functionality** -

    The module operates with two main processes running in parallel:

  * **Clock Generation**: An `always` block continuously generates the `CLK` output. It waits for a duration of `period / 2.0`, then inverts the `CLK` signal. This creates a clock wave with a 50% duty cycle whose frequency is determined by the `period` variable. This process only runs when the `ENb_VCO` signal is high.

  * **Frequency Locking**: On every rising edge of the `REF` input, it measures the time that has passed since the *last* rising edge. This gives it the period of the reference clock (`refpd`). It then immediately sets the output clock's `period` to be **exactly 1/8th of the measured reference period** (`period = refpd / 8.0`).

    This behavior perfectly models the primary function of an **8x frequency multiplier**.

* **Expected Output** -

    Based on the code, the expected output is to see the following behavior in a waveform viewer:

  * **Initial State**: The simulation will start, and the `CLK` output will begin toggling with its initial period of **25 ns** (a frequency of 40 MHz).

  * **Locking Event**: The model waits for the **second** rising edge of the `REF` clock (it needs two edges to measure one period). At that exact moment, it will calculate the `REF` period. The `CLK` period will instantly change to be 1/8th of the `REF` period.

  * **Final State**: The `CLK` output will continue to run at a time period that is exactly **by 8** the frequency of the `REF` input. This means that the `CLK` will be running with a frequency of **8 times** that of the `REF` signal.

* **Recieved Output** -
    ...image of the waveform of the PLL output

* **Analyzing the recieved output** -
  * The initial output `CLK` is having a period of **25ns** then it instantly changes to **35.416ns** which is `REF/8.0`.
  * The internal variable `lastedge` tells the occurance of the rising edge in the `REF` signal.
  * The internal variable `refpd` tells the period of the `REF` signal.
  * The internal variable `period` tells the period of the output `CLK` signal.
  * The `ENb_VCO` signal is high from the start of the simulation thus the PLL gives the output from the very start of the simulation.
  * The `ENb_CP` signal is `x` throughout the input as it is not used in the design.
  * The `VCO_IN` signal is the signal which is recieved after the frequency division of the output. As it is a feedback signal we can observe a slight delay when compared to the `REF` input signal.
  * By verifying the waveform we can tell that the PLL is working as a **8x frequency multiplier**.

---

#### ii. **`avsddac` 10 bit-DAC:**
This is the block diagram of `avsddac`:
<img width="803" height="451" alt="image" src="https://github.com/user-attachments/assets/3b3ef41b-eb67-4a00-a5ff-be1e95f0214d" />

* **Input and Output Ports** -

  * **Input Ports**:

      * `D[9:0]`: This is the main **10-bit digital input value**. This binary number (ranging from 0 to 1023) is what the module converts into an analog voltage.
      * `VREFH`: A `real` input that represents the **High Reference Voltage**. This defines the maximum output voltage of the DAC.
      * `VREFL`: A `real` input that represents the **Low Reference Voltage**. This defines the minimum output voltage of the DAC.

  * **Output Ports**:

      * `OUT`: A `real` output that represents the resulting **analog voltage**.

* **Internal Wires and Variables** -

  * `EN`: An internal wire that acts as an **Enable** signal.
  * `Dext[10:0]`: An 11-bit extended version of the 10-bit input `D`. This is used to ensure the `$itor` (integer to real) conversion function works correctly.
  * `NaN`: A `real` variable used to represent an undefined state ("Not a Number").

* **Verilog Code** -

    ```verilog
    module avsddac (
   OUT,
   D,
   VREFH,
   VREFL
    );

   output      OUT;
   input [9:0] D;
   input       VREFH;
   input       VREFL;
   

   reg  real OUT;
   wire real VREFL;
   wire real VREFH;

   real NaN;
   wire EN;

   wire [10:0] Dext;	// unsigned extended

   assign Dext = {1'b0, D};
   assign EN = 1;

   initial begin
      NaN = 0.0 / 0.0;
      if (EN == 1'b0) begin
         OUT <= 0.0;
      end
      else if (VREFH == NaN) begin
         OUT <= NaN;
      end
      else if (VREFL == NaN) begin
         OUT <= NaN;
      end
      else if (EN == 1'b1) begin
         OUT <= VREFL + ($itor(Dext) / 1023.0) * (VREFH - VREFL);
      end
      else begin
         OUT <= NaN;
      end
   end

   always @(D or EN or VREFH or VREFL) begin
      if (EN == 1'b0) begin
         OUT <= 0.0;
      end
      else if (VREFH == NaN) begin
         OUT <= NaN;
      end
      else if (VREFL == NaN) begin
         OUT <= NaN;
      end
      else if (EN == 1'b1) begin
         OUT <= VREFL + ($itor(Dext) / 1023.0) * (VREFH - VREFL);
      end
      else begin
         OUT <= NaN;
          end
       end
    endmodule
    ```
* **Basic Functionality** -

    The core of this module is a single line of code that executes whenever any input changes:

    ```verilog
    OUT <= VREFL + ($itor(Dext) / 1023.0) * (VREFH - VREFL);
    ```

 This is the mathematical formula for a linear DAC. It works by:

 * Calculating the total voltage range (`VREFH - VREFL`).
 * Scaling the digital input `D` to a fraction between 0.0 and 1.0 (by dividing it by 1023.0, the maximum value for a 10-bit number).
 * Multiplying the voltage range by this fraction.
 * Adding the result to the base low voltage (`VREFL`).

 Overall it maps the digital input value (0-1023) linearly onto the analog voltage range defined by `VREFL` and `VREFH`.

* **Expected Output** -

 The output `OUT` will be a `real` voltage value that changes instantly in response to changes in the digital input `D`.

  * When the input `D` is **0** (`10'b0000000000`), the output `OUT` will be exactly equal to **`VREFL`**.
  * When the input `D` is **1023** (`10'b1111111111`), the output `OUT` will be exactly equal to **`VREFH`**.
  * When the input `D` is halfway, for example **512** (`10'b1000000000`), the output `OUT` will be almost exactly in the middle of `VREFL` and `VREFH`.

* **Recieved Output** -
  ...recieved output for dac image

* **Analysis of the recieved output** -
  * In the input `D` we can see that as time increases intially the output keeps increasing till it reaches a peak value of `946`. After reaching the peak value the input decreases steadily.
  * The `VREFH` is 1 and `VREFL` is 0. The `EN` signal is always high.
  * Correspondingly we can observe the analog output which is being generated between the values of `0 -> 0 and 946 -> 0.9247311`.

---

#### iii. **`rvmyth` core:** 
This is a basic architecture of the RISC-V core:
<img width="892" height="459" alt="image" src="https://github.com/user-attachments/assets/858c8868-208a-4d83-9eea-c9a9952a65a8" />


* **Input and Output Ports** -

    * **Input Ports**:
        * `CLK`: The main system **clock** signal that drives the processor's pipeline.
        * `reset`: A signal to reset the processor to its initial state (e.g., setting the program counter to zero).

    * **Output Ports**:
        * `OUT[9:0]`: A 10-bit output port. The final line of the code (`OUT = CPU_Xreg_value_a5[17];`) reveals that this port is directly connected to the value of **register x17**.

* **Internal Structure** -

    * **Pipelined Signals**: The signal names follow a pattern. The `_a0` through `_a5` suffix indicates the pipeline stage the signal belongs to, representing a 5-stage pipeline (Fetch, Decode, Execute(ALU), Memory(load/store), Writeback). 
    * **Instruction Memory (`instrs` array)**: This is a **hardcoded ROM** inside the processor. It contains a fixed 13-instruction program that the CPU will execute. This means the CPU isn't fetching code from the outside world; it runs this specific program every time the system is turned on.
    * **Register File (`CPU_Xreg_value_a*`)**: This represents the 32 general-purpose 32-bit registers (x0-x31) of the RISC-V architecture.
    * **Data Memory (`CPU_Dmem_value_a*`)**: A small internal RAM for the processor to use for load and store operations.
      
* **Instructions which are hard-coded in the ROM** -
    ```assembly
    addi x9, x0, 1
    addi x10, x0, 43
    addi x11, x0, 0
    addi x17, x0, 0
    add  x17, x17, x11
    addi x11, x11, 1
    bne  x11, x10, -8
    add  x17, x17, x11
    sub  x17, x17, x11
    sub  x11, x11, x9
    bne  x11, x9, -8
    sub  x17, x17, x11
    beq  x0, x0, 0
    ```
    
* **Basic Functionality** -
    The pipelining is done in an organized manner with `ai` where i is the stage of the instruction.

    * **Stage a0 – Fetch Stage** - Fetch the instruction from instruction memory and prepare the PC for the next stage.

        | Variables               | What It Does / Represents                                                               |
        | -------------------------- | --------------------------------------------------------------------------------------- |
        | `CPU_reset_a0`             | Input reset signal; used to initialize PC to 0.                                         |
        | `CPU_pc_a0`                | Current program counter (PC). If reset → 0, else computed next PC.                      |
        | `CPU_imem_rd_en_a0`        | Enables instruction memory read (1 if fetch active).                                    |
        | `CPU_imem_rd_addr_a0`      | Address to read instruction from instruction memory (`PC[4:2]` used for word indexing). |

        **Flow in a0:**

        1. **Reset check:** If `CPU_reset_a0 = 1`, `CPU_pc_a0 = 0` (start of program).
        2. **Instruction memory read enable:** `CPU_imem_rd_en_a0 = !CPU_reset_a0` → active unless in reset.
        3. **Read address generation:** `CPU_imem_rd_addr_a0 = CPU_pc_a0[4+1:2]` → index for instruction memory which is PC/4.
        4. **Output:** Provides `CPU_pc_a0` and `CPU_imem_rd_addr_a0` for the next stage (`a1`).

        **In short:**
        > **a0 stage calculates the PC, enables instruction memory read, and passes the instruction address to the decode stage.**

    With the help of the first instruction lets understand the basic functionality of the `rvmyth`

    **Instruction:**

    ```
    ADDI r9, r0, 1
    ```

    * Adds immediate `1` to `r0` (which is always 0) and writes result to `r9`.
    **Stage-by-stage Execution of `ADDI r9, r0, 1`**

| Stage                                      | Suffix | What happens                                                                               | Relevant Variables                                                                                                                                  |
| ------------------------------------------ | ------ | ------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| **0 – Fetch**                              | `a0`   | PC is used to fetch instruction from instruction memory.                                   | `CPU_pc_a0` → PC (0 at reset)<br>`CPU_imem_rd_en_a0` → 1<br>`CPU_imem_rd_addr_a0` → 0 (instruction index)                                           |
| **1 – Decode**                             | `a1`   | Instruction decoded; identify it as `I-type` and immediate extracted.                      | `CPU_instr_a1` → `instrs[0]`<br>`CPU_imm_a1` → `1`<br>`CPU_is_i_instr_a1` → 1<br>`CPU_rs1_a1` → r0<br>`CPU_rd_a1` → r9<br>`CPU_opcode_a1` → 0010011 |
| **2 – Register Read / Target Calculation** | `a2`   | Read source register `r0` (always 0). Compute jump/branch targets (not relevant for ADDI). | `CPU_src1_value_a2` → `CPU_Xreg_value_a4[r0]` → 0<br>`CPU_src2_value_a2` → N/A<br>`CPU_br_tgt_pc_a2`, `CPU_jalr_tgt_pc_a2` → computed but unused    |
| **3 – Execute / ALU**                      | `a3`   | ALU performs addition with immediate.                                                      | `CPU_result_a3` → `CPU_src1_value_a3 + CPU_imm_a3` → `0 + 1 = 1`<br>`CPU_rf_wr_en_a3` → 1 (because rd=r9 valid)<br>`CPU_valid_a3` → 1               |
| **4 – Memory Access**                      | `a4`   | No memory operation needed for ADDI. Signals just pass through.                            | `CPU_dmem_rd_en_a4` → 0<br>`CPU_dmem_wr_en_a4` → 0<br>`CPU_dmem_addr_a4` → N/A                                                                      |
| **5 – Writeback**                          | `a5`   | ALU result written to destination register `r9`.                                           | `CPU_Xreg_value_a5[r9]` → 1<br>`OUT` → 1 (for your waveform)                                                                                        |


* **Expected Output**

    * The processor will take a number of clock cycles to execute the program. During this time, the `OUT` port (reflecting register `r17`) will change as intermediate values are calculated.
    * After the program completes its main calculations (after instruction #11) and enters the infinite loop, register `r17` will hold the final value of **0**.
    * Therefore, the expected final output is that the `OUT[9:0]` port will settle to a stable value of **`10'b0000000000`** and remain there for the rest of the simulation.

* **Recieved Output** -
  ...recieved output for rvmyth or core image

* **Analysis of the recieved output** -
  * In the input `D` we can see that as time increases intially the output keeps increasing till it reaches a peak value of `946`. After reaching the peak value the input decreases steadily.
  * The `VREFH` is 1 and `VREFL` is 0.
  * Corresponsingly we can observe the analog output which is being generated between the values of ` -> 0 and -> 1`.

---
