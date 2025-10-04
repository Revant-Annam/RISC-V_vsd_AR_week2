# Week 2 Lab

The necessary simulation toolchain, including **Icarus Verilog** and **GTKWave**, was successfully installed and configured as part of the Week 0 preparatory tasks.

The primary objective for this week is to build a solid understanding of **System on Chip (SoC) fundamentals**, which is done in the Theory folder in this repository. This theoretical knowledge will be complemented by a practical exercise focused on the functional modeling of the **BabySoC**. The goal is to practice functional modeling of the BabySoC using simulation tools (Icarus Verilog & GTKWave).

## Recap of functional modeling:

To recap, functional modeling acts as a high-level design blueprint, describing the SoC's intended behavior — the "what" of the SoC functions — and allowing for rapid simulation and the correction of logical errors at the earliest and most cost-effective stage of development.

## File structure:

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

<img width="1536" height="864" alt="Screenshot from 2025-09-29 03-26-28" src="https://github.com/user-attachments/assets/3283ca8a-afa1-42d5-bf20-035f3c9ba65e" />


### 2. Converting the .tlv file to .v

If we don’t convert the `rvmyth.tlv` to `rvmyth.v`, we might encounter this error when we try to do the pre_synth_sim.

<img width="1536" height="864" alt="Screenshot from 2025-09-29 21-08-13" src="https://github.com/user-attachments/assets/1ffa23fc-73de-470d-8b9b-0ae4b75ccee4" />


Using the following commands, we can create a Python virtual environment and convert the file with the help of `sandpiper-saas`. The `sandpiper-saas` must be installed in the virtual environment.

```bash
# Step 1: In ../VSDBabySoC directory create a new virtual environment and install sandpiper-saas
python3 -m venv my_env
source my_env/bin/activate
pip install pyyaml click sandpiper-saas

# Step 2: Convert rvmyth.tlv to Verilog
sandpiper-saas -i ./src/module/*.tlv -o rvmyth.v --bestsv --noline -p verilog --outdir ./src/module/
```

<img width="1536" height="864" alt="Screenshot from 2025-09-29 21-16-40" src="https://github.com/user-attachments/assets/6aaa86aa-4916-4524-98e6-83952609e17b" />


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

<img width="1536" height="864" alt="Screenshot from 2025-09-29 21-26-48" src="https://github.com/user-attachments/assets/f3d7e46b-af0e-4dba-b088-5a5fde3a3e5e" />


> **Note:** If you find the Makefile in `../VSDBabySoC`, then you can directly use the commands below:
>
> ```
> cd ~/VLSI/VSDBabySoC
> make pre_synth_sim
> ```

### 4. Analyzing the waveform:

### i. **`avsdpll` 8x PLL:**
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

  <img width="1920" height="1080" alt="Screenshot from 2025-09-30 19-58-21" src="https://github.com/user-attachments/assets/440f01a5-cc03-4862-ae28-1590ad71865d" />


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

### ii. **`avsddac` 10 bit-DAC:**
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
  
  <img width="1920" height="1080" alt="Screenshot from 2025-09-30 21-46-06" src="https://github.com/user-attachments/assets/a7d64527-dc0a-4f64-afd3-b04e52e0aad7" />


* **Analysis of the recieved output** -
  * In the input `D` we can see that as time increases intially the output keeps increasing till it reaches a peak value of `946`. After reaching the peak value the input decreases steadily.
  * The `VREFH` is 1 and `VREFL` is 0. The `EN` signal is always high.
  * Correspondingly we can observe the analog output which is being generated between the values of `0 -> 0 and 946 -> 0.9247311`.

---

### iii. **`rvmyth` core:**

This is a basic architecture of the RISC-V core: <img width="892" height="459" alt="image" src="https://github.com/user-attachments/assets/858c8868-208a-4d83-9eea-c9a9952a65a8" />

* **Input and Output Ports** -

  * **Input Ports**:

    * `CLK`: The main system **clock** signal that drives the processor's pipeline.
    * `reset`: A signal to reset the processor to its initial state (e.g., setting the program counter to zero).

  * **Output Ports**:

    * `OUT[9:0]`: A 10-bit output port. The final line of the code (`OUT = CPU_Xreg_value_a5[17];`) reveals that this port is directly connected to the value of **register x17**.

* **Internal Structure** -

  * **Pipelined Signals**: The signal names follow a pattern. The `_a0` through `_a5` suffix indicates the pipeline stage the signal belongs to, representing a 6-stage pipeline (Fetch, Decode, Register Read, Execute(ALU), Memory(load/store), Writeback).
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
  beq  x0, x0, -32
  ```

* **Basic Functionality** -
  The pipelining is done in an organized manner with `ai` where i is the stage of the instruction.

---

#### **Stage a0 – Fetch Stage**

**Purpose:** Fetch the instruction from instruction memory and prepare the PC for the next stage.

| Variables             | What It Does / Represents                                                               |
| --------------------- | --------------------------------------------------------------------------------------- |
| `CPU_reset_a0`        | Input reset signal; used to initialize PC to 0.                                         |
| `CPU_pc_a0`           | Current program counter (PC). If reset → 0, else computed next PC.                      |
| `CPU_imem_rd_en_a0`   | Enables instruction memory read (1 if fetch active).                                    |
| `CPU_imem_rd_addr_a0` | Address to read instruction from instruction memory (`PC[4:2]` used for word indexing). |

**Flow in a0:**

1. **Reset check:** If `CPU_reset_a0 = 1`, `CPU_pc_a0 = 0` (start of program).
2. **Instruction memory read enable:** `CPU_imem_rd_en_a0 = !CPU_reset_a0` → active unless in reset.
3. **Read address generation:** `CPU_imem_rd_addr_a0 = CPU_pc_a0[4+1:2]` → index for instruction memory which is PC[4:0]/4.
4. **Output:** Provides `CPU_pc_a0` and `CPU_imem_rd_addr_a0` for the next stage (`a1`).

💡 **In short:**

> **a0 stage calculates the PC, enables instruction memory read, and passes the instruction address to the decode stage.**

---

#### **Stage a1 – Decode Stage**

**Purpose:** Decode the fetched instruction, extract operands, immediate values, and identify instruction type.

| Variables                            | What It Does / Represents                                                    |
| ------------------------------------ | ---------------------------------------------------------------------------- |
| `CPU_instr_a1`                       | Holds the 32-bit instruction fetched from memory.                            |
| `CPU_inc_pc_a1`                      | PC incremented by 4 (next sequential instruction address).                   |
| `CPU_is_i_instr_a1`                  | 1 if instruction is I-type (like ADDI, loads).                               |
| `CPU_is_r_instr_a1`                  | 1 if instruction is R-type (like ADD, SUB).                                  |
| `CPU_is_s_instr_a1`                  | 1 if instruction is S-type (store).                                          |
| `CPU_is_b_instr_a1`                  | 1 if instruction is B-type (branch).                                         |
| `CPU_is_u_instr_a1`                  | 1 if instruction is U-type (LUI/AUIPC).                                      |
| `CPU_is_j_instr_a1`                  | 1 if instruction is J-type (Jump).                                            |
| `CPU_imm_a1`                         | Extracted immediate value based on instruction type.                         |
| `w_CPU_rs1_a1`                       | Source register 1 index.                                                     |
| `w_CPU_rs2_a1`                       | Source register 2 index (if needed).                                         |
| `w_CPU_rd_a1`                        | Destination register index.                                                  |
| `CPU_funct3_a1`                      | funct3 field of the instruction (used for operation type).                   |
| `CPU_funct7_a1`                      | funct7 field of the instruction (used for operation type).                   |
| `CPU_dec_bits_a1`                    | Concatenation of funct7[5], funct3, opcode to identify instruction uniquely. |
| `CPU_is_addi_a1`, `CPU_is_add_a1`, … | Signals for specific instructions (decoded from `CPU_dec_bits_a1`).          |
| `CPU_is_jump_a1`                     | 1 if instruction is JAL or JALR.                                             |

**Flow in a1:**

1. **Instruction decode:**

   * Determine the instruction type (I, R, S, B, U, J) from `CPU_instr_a1[6:2]` 
   * Extract immediate (`CPU_imm_a1`) if required.

2. **Register indices:** Determine source (`rs1`, `rs2`) and destination (`rd`) registers from the `CPU_instr_a1`

3. **Operation identification:** Generate flags like `CPU_is_addi_a1`, `CPU_is_sub_a1`, `CPU_is_jump_a1` for control signals in later stages.

4. **PC increment:** Compute `CPU_inc_pc_a1 = CPU_pc_a1 + 4` for the next sequential instruction.

💡 **In short:**

> **a1 stage decodes the instruction, identifies the type, extracts immediate and register indices, and sets up control signals for ALU, branch, or memory operations in later stages.**

---

#### **Stage a2 – Register Read / Target Calculation**

**Purpose:** Read source register values, compute branch/jump targets, and prepare operands for ALU.

| Variables         | What It Does / Represents                                                           |
| -------------------- | ----------------------------------------------------------------------------------- |
| `CPU_src1_value_a2`  | Value read from source register **rs1**.                                            |
| `CPU_src2_value_a2`  | Value read from source register **rs2**.                                            |
| `CPU_br_tgt_pc_a2`   | Branch target PC = current PC + immediate.                                          |
| `CPU_jalr_tgt_pc_a2` | JALR target PC = (rs1 + immediate) with LSB cleared.                                |
| `CPU_pc_a2`          | PC value carried forward for reference (from fetch).                                |

**Flow in a2:**

1. **Register File Read:** `CPU_src1_value_a2` and `CPU_src1_value_a2` ← results in `CPU_result_a3` previous value when `(CPU_rd_a3 == CPU_rs2_a2) && CPU_rf_wr_en_a3` is true else `XReg[CPU_rf_rd_index_1]` or `XReg[CPU_rf_rd_index_2]`.

2. **Branch/Jump Target Calculation:**

   * `CPU_br_tgt_pc_a2 = CPU_pc_a2 + immediate` (for B-type).
   * `CPU_jalr_tgt_pc_a2 = (CPU_src1_value_a2 + immediate) & ~1` (for JALR).

💡 **In short:**

> **a2 stage reads the register file, prepares ALU operands, and computes possible branch/jump targets. It sets up everything needed for execution in `a3`.**

---

#### **Stage a3 – Execute (ALU)**

**Purpose:** At **a3**, the instruction is **executed**. The ALU performs arithmetic/logic/shift/comparison, branch and jump conditions are resolved, and control/data signals are prepared for register file write-back. This stage decides the actual result of the instruction.

| Variables               | What It Does / Represents                                                                      |
| ----------------------- | ---------------------------------------------------------------------------------------------- |
| `CPU_result_a3`         | ALU result (ADD, SUB, AND, OR, XOR, shifts, comparisons, LUI, AUIPC, JAL, load/store address). |
| `CPU_sltu_result_a3`    | Unsigned comparison between `src1` and `src2`.                                                 |
| `CPU_sltiu_result_a3`   | Unsigned comparison between `src1` and immediate.                                              |
| `CPU_rf_wr_en_a3`       | Enables register write if valid instruction with rd ≠ 0 (or load coming from a5).              |
| `CPU_rf_wr_index_a3`    | Destination register index for write-back (rd).                                                |
| `CPU_rf_wr_data_a3`     | Data to write back → ALU result or load result based on validity                               |
| `CPU_taken_br_a3`       | Branch decision flag (BEQ, BNE, BLT, etc.).                                                    |
| `CPU_valid_taken_br_a3` | If branch is valid and taken, flush younger pipeline instructions and updating PC.                             |
| `CPU_valid_load_a3`     | Marks a valid load instruction.                                                                |
| `CPU_valid_jump_a3`     | Marks a valid jump (JAL/JALR).                                                                 |
| `CPU_valid_a3`          | Overall validity of this stage after considering branches/load.                                     |

**Flow in a3**

1. **Operands** (`src1`, `src2`, `imm`) arrive from decode (a2).
2. **ALU computes** `CPU_result_a3` depending on opcode.
3. **Branch condition** is checked → `CPU_taken_br_a3` raised if true.
4. **Jump condition** evaluated → `CPU_valid_jump_a3` raised for JAL/JALR.
5. **Load check** → `CPU_valid_load_a3` asserted if instruction is a load.
6. **Register write-back control** signals prepared:
   * `CPU_rf_wr_en_a3` (enable)
   * `CPU_rf_wr_index_a3` (which register)
   * `CPU_rf_wr_data_a3` (what value)
8. **Validity finalized** (`CPU_valid_a3`) → ensures flushes/jumps/loads are handled cleanly.

💡 **In short:** 
> **a3 executes the instruction, resolves branches/jumps, and sets up the write-back path.**

---

#### **Stage a4 – Memory access**
**Purpose:** For **load instructions**, data is read from memory. For **store instructions**, data is written to memory. This stage computes memory addresses using the ALU result from **a3** and handles memory read/write enable signals.

| Variables      | What It Does / Represents                                                              |
| ------------------------- | --------------------------------------------------------------------------- |
| `CPU_dmem_rd_en_a4`       | Enables memory read for valid load instructions.                            |
| `CPU_dmem_wr_en_a4`       | Enables memory write for valid store instructions.                          |
| `CPU_dmem_addr_a4`        | Address in memory to read from or write to (computed from `CPU_result_a4`). |
| `CPU_dmem_wr_data_a4`     | Data to write to memory (from `CPU_src2_value_a4`).                         |
| `CPU_Dmem_value_a4[dmem]` | Represents the actual memory array element values.                          |
| `w_CPU_dmem_rd_data_a4`   | Data read from memory to be used in the next stage (a5).                    |
| `CPU_valid_load_a4`       | Indicates the instruction is a valid load.                                  |
| `CPU_valid_a4`            | Ensures only valid instructions can write to memory.                        |

**Flow in a4**

1. **Memory Address Setup:** Compute memory address using ALU result: `CPU_dmem_addr_a4 = CPU_result_a4[5:2]`.

2. **Read/Write Control Signals:**

   * `CPU_dmem_rd_en_a4` = 1 if instruction is a valid load.
   * `CPU_dmem_wr_en_a4` = 1 if instruction is a valid store.

3. **Memory Write (Store Instruction):** If `CPU_dmem_wr_en_a4` is high, write `CPU_dmem_wr_data_a4` into the memory location `CPU_dmem_addr_a4`.

4. **Memory Read (Load Instruction):** If `CPU_dmem_rd_en_a4` is high, read `CPU_Dmem_value_a5[CPU_dmem_addr_a4]` into `w_CPU_dmem_rd_data_a4` for a5.

5. **Pipeline Register Update:** Memory array `CPU_Dmem_value_a4` updates using either reset values, written data, or next-stage values.

💡 **In short:**
> **a4 handles memory read/write, computing addresses and preparing data for write-back in a5.**

---

#### **Stage a5 – Write-back (WB)**

**Purpose:** The final stage of the pipeline. Its purpose is to write the instruction's result—either from the ALU or from memory—back into the destination register in the Register File. This step completes the instruction's execution.

| Variables | What It Does / Represents |
| :--- | :--- |
| `CPU_ld_data_a5` | Data read from memory, arriving from the MEM stage (`a4`). |
| `OUT` | The module's 10-bit output, assigned the value of register `x17` in this stage. |

**Flow in a5:**

1.  **Result Selection:** The logic determines which data to write back.
    * If the instruction completing is a **Load** (indicated by `CPU_valid_load_a5`), the data to be written is `CPU_ld_data_a5`.
    * If the instruction is an **ALU operation** (like `ADD`, `ADDI`), the data to be written is the ALU result that has been carried through the pipeline from the Execute stage (`a3`).

2.  **Register File Write:** If the instruction has a valid write enable and a destination register (`rd`), the selected result is written into the Register File. This is the **commit** step that makes the result of the instruction visible to the rest of the processor.

3.  **Module Output Update:** In this specific design, the `OUT` port is continuously updated with the value held in register `x17` as it appears at the end of the pipeline.

💡 **In short:**
> a5 finalizes the instruction by writing the correct result (either from the ALU or from memory) into the destination register.

---

### **Instruction Execution and Waveform Analysis**

<p align = "center">
    <img width="1599" height="826" alt="image" src="https://github.com/user-attachments/assets/12a2a53a-b56b-450e-95d0-b00e1f9644a0" />
</p>

The execution of the hardcoded program can be clearly understood by correlating the program's logic with the signals observed in the GTKWave simulation. The following points describe the key phases of the program's execution and their visual representation in the waveform.

1.  **Processor Start-up (Reset)**
    * **Behavior**: The processor begins execution when the `reset` signal is released (goes from high to low).
    * **Waveform Observation**: You can see the `reset` signal high for the first clock cycle, during which the Program Counter, **`CPU_pc_a0`**, is forced to `0`. On the next clock edge, as `reset` is still high, the processor begins fetching the first instruction from address `0` but the `PC` doesnt get updated. As the instruction at addr `0` is a immediate type instruction the `ADDI x9,x0,1` which is why in the second clock cycle we can observe that the `CPU_is_i_instr_a1` is high.

2.  **Instruction Pipelining**
    * **Behavior**: Each instruction travels through the 6 pipeline stages, with a new instruction entering the pipeline on every clock cycle.
    * **Waveform Observation**: When the reset goes low that is when we can see that the `CPU_pc_a0` stars to increment by 4. This is visualized from the PC signals in each stage: **`CPU_pc_a0`**, **`CPU_pc_a1`**, **`CPU_pc_a2`**, and **`CPU_pc_a3`**. We can see an address appear in `CPU_pc_a0` and then watch it propagate to the subsequent signals, one cycle at a time. As the first 4 instructions are immediate type the `CPU_is_i_instr_a1` is high in the first few clock cycles.
**The cycles stages (explaining the pipeline)**
    * **1st clock cycle**: The first instruction is fetched.
    * **2nd clock cycle**: The second instruction is fetched. The 1st instruction is decoded.
    * **3rd clock cycle**: The third instruction is fetched. The 2nd instruction is decoded. The first instruction reads the required registers.
    * **4th clock cycle**: The fourth instruction is fetched. The 3rd instruction is decoded. The second instruction reads the required registers. The first instruction is executed.
    * **5th clock cycle**: The fifth instruction is fetched. The 4th instruction is decoded. The third instruction reads the required registers. The second instruction is executed. The first instruction enter the memory cycle where it sits idle as the enable is low for reading.
    * **6th clock cycle**: The sixth instruction is fetched. The 5th instruction is decoded. The 4th instruction reads the required registers. The third instruction is executed. The second instruction enter the memory cycle where it sits idle as the enable is low for reading. The first instruction checks for any writeback into the registers `x9`. Here as the fifth instruction is a R-type instruction thus the `CPU_is_r_instr_a1` which goes high as the second instruction is decoded.

3.  **Calculation Loops (Backwards Branching)**
    * **Behavior**: The program contains two "inner" loops that use `bne` instructions to perform repetitive addition and subtraction. These instructions jump backwards to a previous address.
    * **Waveform Observation**: This is seen as a sudden **decrease** in the PC value. For example, you will see **`CPU_pc_a0`** incrementing sequentially (`16`, `20`, `24`) and then, after the `bne` is executed, it will suddenly jump back down to `16`. This jump happens one cycle after the **`CPU_taken_br_a3`** signal goes high. We can also observe that the number of cycles between the decode stage (`CPU_is_b_instr_a1` is high) and the execution stage (`CPU_taken_br_a3` is high) is 2 cycles with the register read cycle in between. 

4.  **Final Result Calculation (Addition loop)**
    * **Behavior**: Before the loops start, the final result of `0` is stored in register `x17`. And the value of the x17 register increases as it enters the first loop which goes till `x11` reaches value of 43.
    * **Waveform Observation**: The **`OUT`** signal, which is connected to `x17` is 0 in the writeback stage of the 5th instruction which is the 5+(6-1) = 10 cycles. Further we can observe in the waveform that the `OUT` signal is incremented every 6 cycles this happens because when the branch instruction is executed in the previous stages the next instructions have already entered the pipeline. So the updated `PC` target enters the instruction fetch `a0` stage so for every increment of the `OUT` variable (write back) it takes 6 cycles. When the bne instruction (at PC=24) is executed in stage a3, the CPU_taken_br_a3 signal is asserted. This immediately instructs the fetch stage to update its PC on the next clock edge. We can observe the CPU_pc_a0 value jumping from its speculative value of 36 down to the branch target of 16, flushing the intermediate instructions.
      
5.  **Program Repetition (The Outer Loop)**
    * **Behavior**: The final instruction in the program is an unconditional branch that forces the processor to restart its main calculation, creating an infinite loop.
    * **Waveform Observation**: This is the largest jump visible. You'll see the **`CPU_pc_a0`** increment to the end of the program (e.g., address `48`) and then, on the next cycle, it will jump all the way back to the start of the calculation at address **`16`** by a subtraction of 32. This shows the entire program repeating itself from the 4th instruction, as intended.

### iv. **`vsdbabysoc` The SoC**
This is the block diagram of `vsdbabysoc` :

<img width="2270" height="1260" alt="image" src="https://github.com/user-attachments/assets/5b4307b0-a488-4a0c-a539-3c3583d26cd7" />


* **Input and Output Ports** -

    * **Input Ports**:
        * `reset`: A signal to reset the processor to its initial state (e.g., setting the program counter to zero).
        * `ENb_VCO`: This is the **VCO Enable** signal. When this signal is high (`1'b1`), the clock generation is enabled. When it's low, the output clock is forced to 0.
        * `VCO_IN`, `ENb_CP`: These ports are included in the module definition but are **not used** in this simplified behavioral model. In a more complex model, they would be used to control the Voltage-Controlled Oscillator and Charge Pump.
        * `REF`: This is the **Reference Clock** input. The model uses the rising edge of this signal to measure its period and adjust the output clock.
        * `VREFH`: A `real` input that represents the **High Reference Voltage**. This defines the maximum output voltage of the DAC.

    * **Output Ports**:
        * `OUT`:  A `real` output that represents the resulting **analog voltage**.

* **Internal Wires and Variables** - 

    * `CLK`: This is the main **Clock Output**. It's the final, high-frequency clock signal generated by the PLL. This is given input to the `rvmyth`.
    * `RV_TO_DAC`: This is the **10-bit digital input value**. This binary number (ranging from 0 to 1023) is what the module converts into an analog voltage. This is the output of the `rvmyth` given to the input of the DAC.

* **Verilog Code** -
    ```verilog
    module vsdbabysoc (
   output wire OUT,
   //
   input  wire reset,
   //
   input  wire VCO_IN,
   input  wire ENb_CP,
   input  wire ENb_VCO,
   input  wire REF,
   //
   // input  wire VREFL,
   input  wire VREFH
    );

   wire CLK;
   wire [9:0] RV_TO_DAC;

   rvmyth core (
      .OUT(RV_TO_DAC),
      .CLK(CLK),
      .reset(reset)
   );

   avsdpll pll (
      .CLK(CLK),
      .VCO_IN(VCO_IN),
      .ENb_CP(ENb_CP),
      .ENb_VCO(ENb_VCO),
      .REF(REF)
   );

   avsddac dac (
      .OUT(OUT),
      .D(RV_TO_DAC),
      // .VREFL(VREFL),
      .VREFH(VREFH)
   );
   
    endmodule
    ```

* **Functionality** -
    This module is a simple System-on-Chip (SoC). Its main function is to act as a **digitally controlled analog voltage generator**.

 1. `avsdpll` (Phase-Locked Loop) ⚙️
This acts as the system's **clock generator**. It takes an external reference clock (`REF`) and other control signals to produce a stable, high-frequency internal clock signal (`CLK`) that drives the rest of the chip.

2. `rvmyth` (RISC-V CPU Core) 🧠
This is the **brain** of the SoC. It's a RISC-V processor that runs on the `CLK` generated by the PLL. It executes a program and, based on its internal logic, produces a 10-bit digital output value (`RV_TO_DAC`).

3. `avsddac` (Digital-to-Analog Converter) 🎛️
This is the **interface to the analog world**. It takes the 10-bit digital value (`RV_TO_DAC`) from the CPU core and converts it into a corresponding analog voltage level on the final `OUT` pin. The conversion is scaled between ground and the high reference voltage `VREFH`.

In summary, the `rvmyth` processor decides what voltage level is needed, outputs a corresponding 10-bit digital code, and the `avsddac` converts that code into an actual analog voltage at the `OUT` pin. The entire system is synchronized by the `avsdpll` clock generator.

* **Expected Output** -

    The expected output of the `vsdbabysoc` module on the `OUT` pin is an **analog voltage**.

    The specific voltage level or waveform of this output is not fixed; it is **dynamically controlled by the software program running on the `rvmyth` RISC-V processor core**.

    1.  The `rvmyth` processor executes its code and produces a 10-bit digital value (`RV_TO_DAC`). This value can range from `0` (binary `0000000000`) to `1023` (binary `1111111111`).
    2.  The `avsddac` (Digital-to-Analog Converter) takes this 10-bit number.
    3.  It converts the number into an analog voltage, where the output voltage `OUT` is proportional to the digital input.

    The relationship can be described by the formula:

    $$V_{OUT} = VREFH \times \frac{D}{2^{10}}$$

    Where:
    * $V_{OUT}$ is the analog voltage on the `OUT` pin.
    * $VREFH$ is the high reference voltage supplied to the chip.
    * $D$ is the 10-bit decimal value of `RV_TO_DAC` (from 0 to 1023).

* **Recieved Output** -

  In the GTKWave viewer, right-click on the `OUT[9:0]` signal. From the context menu that appeared, navigate to `Data Format`, then to the `Analog` sub-menu, and finally select `Step`.

  <img width="1920" height="1080" alt="Screenshot from 2025-10-04 16-19-01" src="https://github.com/user-attachments/assets/6afe0d6f-4b48-411b-bbd6-c3e5869cf05e" />


  This is done to change the visual drawing style of the analog waveform for better analysis.

  <img width="1920" height="1080" alt="Screenshot from 2025-10-04 16-22-44" src="https://github.com/user-attachments/assets/6107a2f2-8ece-43ea-86ce-de3ace828784" />
  

* **Analyzing the recieved output** -
  1. **`reset`**: The simulation begins with `reset` held **high** for a few microseconds. This correctly initializes the system. Once `reset` goes **low**, the processor starts executing its program.
  2. **`CLK`**: This is the main system clock, generated by the PLL. It appears to be a stable, periodic square wave.
  3. **`RV_TO_DAC[9:0]`**: This is the 10-bit digital output from the RISC-V core. A repeating pattern of changing bits. This pattern represents the sequence of numbers the processor is sending to the DAC.
  4. **`VREFH`**: This is the high reference voltage for the DAC. As expected, it is a **constant DC value**, represented by the flat line at the top.
  5. **`OUT`** (Analog Output): This signal is an analog waveform that directly mirrors the digital pattern from `RV_TO_DAC`. It is a periodic periodic signal. 

---

### 4. Synthesis

The synthesis process converts the high-level Register Transfer Level (RTL) description of the SoC into a gate-level netlist, mapping the design to the physical standard cells of the SKY130 technology library. This is performed using the open-source synthesis tool, **Yosys**.

The complete set of commands for this process is defined in the synthesis script located at: `../src/script/yosys.ys`.

The script performs the following key steps:

1.  **Reads the design files**, including the top-level `vsdbabysoc.v`, the CPU core `rvmyth.v`, and other modules.
2.  **Loads the standard cell libraries** (`.lib` files) which contain the definitions and timing information for basic logic gates (like AND, OR, D-flip-flops), the PLL, and the DAC.
   
   <img width="1920" height="1080" alt="Screenshot from 2025-10-03 04-09-01" src="https://github.com/user-attachments/assets/29498512-f05d-47da-9ab2-475917584d76" />


3.  **Synthesizes the design** using the `synth` command, which performs high-level optimizations.

    <img width="1920" height="1080" alt="Screenshot from 2025-10-03 04-09-29" src="https://github.com/user-attachments/assets/35cc54f4-a773-4536-b49b-59e1e567fd8d" />

   
4.  **Maps flip-flops and optimizes** the logic using the `dfflibmap`, `opt`, and `abc` commands.
   
    <img width="1920" height="1080" alt="Screenshot from 2025-10-03 04-09-49" src="https://github.com/user-attachments/assets/cf073699-390b-4c51-be1d-d9406c9317c7" />

    <img width="1920" height="1080" alt="Screenshot from 2025-10-03 04-10-27" src="https://github.com/user-attachments/assets/2a440783-d49d-4804-b021-b4fb612c75f2" />

5.  **Cleans up the netlist** and writes the final synthesized Verilog file to `../output/synth/vsdbabysoc.synth.v`.
6.  **Viewing the elements** using the `stat` command the internal components of the design can be viewed. 

The specific sequence of commands used in the script is as follows:

```tcl
# Read Verilog and Liberty files
read_verilog ./module/vsdbabysoc.v
read_verilog -I./include ../output/compiled_tlv/rvmyth.v
read_verilog -I./include ./module/clk_gate.v
read_liberty -lib ./lib/avsdpll.lib
read_liberty -lib ./lib/avsddac.lib
read_liberty -lib ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib

# Synthesize the top-level module
synth -top vsdbabysoc

# Map flip-flops and perform optimizations
dfflibmap -liberty ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib
opt
abc -liberty ./lib/sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime;{D};strash;dch,-f;map,-M,1,{D}

# Clean up and finalize the netlist
flatten
setundef -zero
clean -purge
rename -enumerate

# Generate statistics and write the output Verilog
stat
write_verilog -noattr ../output/vsdbabysoc.synth.v
```

To view the block diagram of the design we can use the same commands without using the flatten.

**The synthesized output for VSDBabySoC:**

<img width="1280" height="720" alt="Screenshot from 2025-10-03 05-07-02" src="https://github.com/user-attachments/assets/3afd2f50-e79f-4bed-87c3-5bc142f75e29" />




<img width="1920" height="1080" alt="Screenshot from 2025-10-04 17-47-00" src="https://github.com/user-attachments/assets/e11a878c-bfc8-45f2-9a43-e2fb17808a5f" />







