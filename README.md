# SKY130 RTL Design and Synthesis Workshop

A comprehensive technical journal and lab documentation covering the **5-Day RTL Design, Simulation, and Logic Synthesis Flow using SKY130 Technology**, conducted by **VLSI System Design (VSD)** and mentored by **Kunal Ghosh**.

---

## 📌 Table of Contents
1. [Workshop Overview & Toolset](#1-workshop-overview--toolset)
2. [Environment Setup & Installation](#2-environment-setup--installation)
3. [Cloning Kunal Ghosh Sir's Workshop Repository](#3-cloning-kunal-ghosh-sirs-workshop-repository)
4. [Directory Structure & Collateral Walkthrough](#4-directory-structure--collateral-walkthrough)
   - [A. Standard Cell Timing Library (`lib/`)](#a-standard-cell-timing-library-lib)
   - [B. Standard Cell Verilog Models (`my_lib/verilog_model/`)](#b-standard-cell-verilog-models-my_libverilog_model)
   - [C. RTL Designs and Testbenches (`verilog_files/`)](#c-rtl-designs-and-testbenches-verilog_files)
5. [Day 1: Theory of RTL Design & Simulation](#5-day-1-theory-of-rtl-design--simulation)
   - [What is Design?](#what-is-design)
   - [What is a Simulator?](#what-is-a-simulator)
   - [How Simulator Works (Event-Driven Simulation)](#how-simulator-works-event-driven-simulation)
   - [What is a Testbench?](#what-is-a-testbench)
   - [The 3-Step Simulation Flow (`iverilog` & `GTKWave`)](#the-3-step-simulation-flow-iverilog--gtkwave)
   - [Deep Dive: Working with GTKWave](#deep-dive-working-with-gtkwave)
   - [Working with Text Editors: GVim & Vim Commands](#working-with-text-editors-gvim--vim-commands)
   - [Lab 1: Behavioral Simulation of 2:1 Multiplexer (`good_mux.v`)](#lab-1-behavioral-simulation-of-21-multiplexer-good_muxv)
   - [Lab 2: Logic Synthesis with Yosys (`good_mux.v`)](#lab-2-logic-synthesis-with-yosys-good_muxv)
   - [Synthesized Netlist Generation (`write_verilog -noattr`)](#synthesized-netlist-generation-write_verilog--noattr)
6. [Day 2: Timing Libraries, Hierarchical vs Flat Synthesis & Multipliers](#6-day-2-timing-libraries-hierarchical-vs-flat-synthesis--multipliers)
   - [Introduction to Liberty (`.lib`) Files & PVT Corners](#introduction-to-liberty-lib-files--pvt-corners)
   - [Hierarchical Synthesis (`multiple_modules.v`)](#hierarchical-synthesis-multiple_modulesv)
   - [Flat Synthesis (`flatten`)](#flat-synthesis-flatten)
   - [Sub-Module Level Synthesis (`sub_module1`)](#sub-module-level-synthesis-sub_module1)
   - [Hardware Optimization for Multipliers (`mult_2.v`)](#hardware-optimization-for-multipliers-mult_2v)
7. [Day 3: Combinational and Sequential Logic Optimizations](#7-day-3-combinational-and-sequential-logic-optimizations)
   - [Combinational Logic Optimization (`opt_check.v`, `opt_check3.v`)](#combinational-logic-optimization-opt_checkv-opt_check3v)
   - [Sequential Logic Optimization: Constant Flop Optimization (`dff_const1` to `dff_const4`)](#sequential-logic-optimization-constant-flop-optimization-dff_const1-to-dff_const4)
   - [Sequential Logic Optimization: Unused Flop Elimination (`counter_opt.v` vs `counter_opt2.v`)](#sequential-logic-optimization-unused-flop-elimination-counter_optv-vs-counter_opt2v)
8. [Synthesis Command Quick Reference](#8-synthesis-command-quick-reference)
9. [Author & Acknowledgements](#9-author--acknowledgements)

---

## 1. Workshop Overview & Toolset

This workshop covers frontend digital IC design: from writing behavioral Verilog RTL code, verifying functionality with event-driven simulation, mapping to technology cells via logic synthesis with Yosys, and performing combinational/sequential logic optimizations using the open-source **SkyWater 130nm PDK**.

### Primary EDA Tools
| Tool | Category | Role in Flow |
| :--- | :--- | :--- |
| **Icarus Verilog (`iverilog`)** | Verilog Simulator | Compiles RTL design and testbench into an executable simulation engine (`a.out`). Performs event-driven evaluation and generates Value Change Dump (`.vcd`) waveforms. |
| **GTKWave** | Waveform Viewer | Graphical tool to trace digital signals, clock transitions, and bus transitions across simulation time. |
| **Yosys** | Logic Synthesizer | Translates behavioral Verilog into optimized gate-level netlists mapped to physical standard cells in the target technology. |
| **SkyWater 130nm PDK** | Open Foundry Collaterals | High-Density (`sky130_fd_sc_hd`) standard cell library, liberty (`.lib`) timing definitions, and Verilog simulation primitives. |

---

## 2. Environment Setup & Installation

All tools were configured inside Ubuntu 22.04 LTS running on Windows Subsystem for Linux (WSL2):

```bash
# Update Ubuntu package lists
sudo apt-get update

# Install simulation, synthesis tools, and text editors
sudo apt-get install -y iverilog gtkwave yosys vim-gtk3 git
```

---

## 3. Cloning Kunal Ghosh Sir's Workshop Repository

The official workshop collaterals repository created by **Kunal Ghosh sir** contains all standard cell timing libraries, gate models, and lab designs.

```bash
# Navigate to home directory and create VLSI workspace
cd ~
mkdir -p VLSI
cd VLSI

# Clone Kunal sir's official workshop repository
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git

# Enter the workshop directory
cd sky130RTLDesignAndSynthesisWorkshop
```

---

## 4. Directory Structure & Collateral Walkthrough

```text
~/VLSI/sky130RTLDesignAndSynthesisWorkshop/
├── DC_WORKSHOP/                        # Reference collaterals
├── README.md                           # Course README
├── yosys_run.sh                        # Synthesis automation script
├── lib/                                # Standard cell timing (.lib) libraries
│   └── sky130_fd_sc_hd__tt_025C_1v80.lib
├── my_lib/                             # Gate models for Gate-Level Simulation (GLS)
│   └── verilog_model/
│       ├── primitives.v                # Gate primitives
│       └── sky130_fd_sc_hd.v           # Consolidated behavioral cell models
└── verilog_files/                      # RTL source designs and testbenches
    ├── good_mux.v, tb_good_mux.v
    ├── multiple_modules.v
    ├── mult_2.v, mult_8.v
    ├── opt_check.v, opt_check2.v, opt_check3.v, opt_check4.v
    ├── dff_const1.v - dff_const5.v, tb_dff_const1.v - tb_dff_const5.v
    ├── counter_opt.v, counter_opt2.v
    └── ...
```

### A. Standard Cell Timing Library (`lib/`)
Contains technology timing information, cell delays, and power models at standard PVT (Process, Voltage, Temperature) corners:
- **Path:** `~/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/`
- **File:** `sky130_fd_sc_hd__tt_025C_1v80.lib` (Typical corner, 25°C, 1.8V)
- **Access command:**
  ```bash
  cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib
  ls -lh
  ```

<!-- Add your timing library screenshot below -->
<!-- ![Timing Library](images/timing_lib.png) -->

---

### B. Standard Cell Verilog Models (`my_lib/verilog_model/`)
Contains functional models of standard cells used during post-synthesis Gate-Level Simulation (GLS):
- **Path:** `~/VLSI/sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/`
- **Files:** `primitives.v`, `sky130_fd_sc_hd.v`
- **Access command:**
  ```bash
  cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model
  ls -lh
  ```

<!-- Add your verilog models screenshot below -->
<!-- ![Verilog Models](images/verilog_models.png) -->

---

### C. RTL Designs and Testbenches (`verilog_files/`)
Contains all Verilog RTL designs and testbench files for the workshop:
- **Path:** `~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files/`
- **Access command:**
  ```bash
  cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
  ls -la
  ```

<!-- Add your verilog files directory screenshot below -->
<!-- ![Verilog Files Directory](images/verilog_files.png) -->

---

## 5. Day 1: Theory of RTL Design & Simulation

### What is Design?
> **Design** is the synthesizable Verilog HDL code (or set of codes) written to implement specific architectural and behavioral specifications.


<img width="1366" height="768" alt="What is Design" src="https://github.com/user-attachments/assets/1f4c0b58-7e1a-479e-9e4d-9fb70e1e44a7" />

---

### What is a Simulator?
- **RTL design is checked for adherence to the specifications by simulating the design.**
- **Simulator** is the software tool used for simulating the design before fabrication.

<img width="1366" height="768" alt="What is Simulator" src="https://github.com/user-attachments/assets/8322f603-9cc3-4144-9aad-ec5098274590" />

---

### How Simulator Works (Event-Driven Simulation)
- The simulator continuously **looks for changes on the input signals**.
- **Upon change to the input, the output is evaluated.**
- If there is no change on the inputs, the simulator does not evaluate outputs, saving simulation time and CPU resources.


---

### What is a Testbench?
- A **Testbench (`tb`)** is a Verilog wrapper code that applies input stimulus to the Design Under Test (**DUT**).
- It contains non-synthesizable timing statements (like `#10`, `$dumpfile`, `$dumpvars`, `$finish`) to generate clock, reset, and stimulus patterns.
- The simulator executes the testbench and records signal value transitions into a Value Change Dump (`.vcd`) file.

<img width="1366" height="768" alt="What is Testbench" src="https://github.com/user-attachments/assets/c5cceedc-07f2-407c-9667-8929a385aaab" />

---

### The 3-Step Simulation Flow (`iverilog` & `GTKWave`)

|

<img width="1366" height="768" alt="Simulation Flow" src="https://github.com/user-attachments/assets/8328bffc-baec-4ac6-a46f-73e17b1392c0" />

---

### Deep Dive: Working with GTKWave

#### 1. Why GTKWave needs `.vcd` (Value Change Dump):
GTKWave is strictly a **waveform viewer** that parses digital transitions over time stored inside `.vcd` files. It cannot read Verilog HDL source text files (`.v`).
- Attempting to run `gtkwave design.v` will produce the error:
  `Near byte 17, VCD search table NULL..Is this a VCD file? Malformed!`
- Always compile with `iverilog` and execute `./a.out` first to produce the `.vcd` file before opening GTKWave.

#### 2. How to view signals inside GTKWave:
1. In the top-left **SST (Search Signal Tree)** panel, click on the testbench module (`tb_...`) or DUT instance.
2. In the bottom-left **Signals** box, select the desired signals (`clk`, `reset`, `q`, etc.).
3. Click the **Append** button (or press `Insert`) to add the signals to the waveform viewer.
4. **Zooming & Navigation:**
   - Click **Zoom Fit** (magnifying glass with box icon) to display the entire simulation duration across the screen.
   - Click **Zoom In (`+`)** or **Zoom Out (`-`)** to inspect transition details.
   - Click anywhere along the waveform trace to place a marker and read exact signal values at that timestamp.

---

### Working with Text Editors: GVim & Vim Commands

In digital design, inspecting testbenches and RTL designs side-by-side is done using **GVim**:

```bash
# Open two Verilog files in horizontal split view:
gvim tb_good_mux.v -o good_mux.v &
```

#### Essential Vim Shortcuts:
- **`Esc`**: Exit Insert mode and return to Normal / Command mode.
- **`:syn off`**: Turn off syntax highlighting (essential for smoothly opening huge 170,000+ line `.lib` timing files).
- **`gg`**: Jump directly to line 1 (the very top of the file).
- **`G`**: Jump to the last line of the file.
- **`:q!`**: Quit immediately without saving changes (discards accidental edits safely).
- **`:wq`**: Save changes and quit.

---

### Lab 1: Behavioral Simulation of 2:1 Multiplexer (`good_mux.v`)

#### Design Code: `good_mux.v`


<img width="1366" height="768" alt="Screenshot (224)" src="https://github.com/user-attachments/assets/1d1bef7a-c462-49f0-b163-e94bba4549e4" />


#### Simulation Commands:
```bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files

# 1. Compile design and testbench
iverilog good_mux.v tb_good_mux.v

# 2. Execute simulation (generates tb_good_mux.vcd)
./a.out

# 3. View waveforms in GTKWave
gtkwave tb_good_mux.vcd &
```

<img width="1366" height="768" alt="Screenshot (221)" src="https://github.com/user-attachments/assets/649bd62b-fb8c-41f5-8736-2dbcf9bcec95" />




---
### YOSYS INTRODUCTION

<img width="1366" height="768" alt="Screenshot (226)" src="https://github.com/user-attachments/assets/f9cbf73f-2d40-4169-a1dc-77d19c36248e" />


<img width="1366" height="768" alt="Screenshot (229)" src="https://github.com/user-attachments/assets/976dc888-22e1-4039-8210-266c7a81274f" />

<img width="1366" height="768" alt="Screenshot (246)" src="https://github.com/user-attachments/assets/c23ff691-b624-4eb2-a889-61641db18b31" />

---


### Lab 2: Logic Synthesis with Yosys (`good_mux.v`)

Synthesis transforms the behavioral RTL description into physical gates mapped to the SkyWater 130nm standard cell library (`sky130_fd_sc_hd`).

#### Commands in Yosys:
```text
yosys
yosys> read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> read_verilog good_mux.v
yosys> synth -top good_mux
yosys> abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> show
```

<img width="1366" height="768" alt="Screenshot (249)" src="https://github.com/user-attachments/assets/61f6d1dd-3987-441d-a163-2420062c6022" />



#### Synthesis Results & Optimization:
- In modern Yosys versions, the synthesis engine maps `good_mux` directly to a single dedicated physical cell: **`sky130_fd_sc_hd__mux2_1`**!
- Inputs `i0`, `i1`, and `sel` map directly to pins `A0`, `A1`, and select `S`, while output `y` connects to pin `X`.
- This single-cell mapping achieves minimum silicon area and higher operating speed compared to older 3-gate decomposed equivalents (Inverter + NAND + O21AI).



<img width="1366" height="768" alt="Screenshot (253)" src="https://github.com/user-attachments/assets/2a1ca533-4251-4fbb-adc0-8268aacb0070" />

---

### Synthesized Netlist Generation (`write_verilog -noattr`)
To save the clean synthesized netlist without internal compiler attributes:



<img width="1366" height="768" alt="Screenshot (257)" src="https://github.com/user-attachments/assets/15e727d5-8aa0-415d-8b81-67d979f3451c" />




    


---

## 6. Day 2: Timing Libraries, Hierarchical vs Flat Synthesis & Multipliers

### Introduction to Liberty (`.lib`) Files & PVT Corners
The `.lib` file is an industry-standard ASCII representation of standard cell timing, power, and functionality:
- **PVT Corner:** Process: Typical (TT), Voltage: 1.80V, Temperature: 25°C (`sky130_fd_sc_hd__tt_025C_1v80.lib`).
- Contains lookup tables for cell delay and output transition time as functions of **Input Transition Time (Slew)** and **Output Load Capacitance**.

Viewing the library in GVim:
```bash
gvim ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib &
```

<img width="1366" height="768" alt="Screenshot (261)" src="https://github.com/user-attachments/assets/60e30866-8fb4-4948-8e0f-2462c295c6dd" />


<img width="1366" height="768" alt="Screenshot (264)" src="https://github.com/user-attachments/assets/3e754c67-10fe-4daf-9849-d19ee2dd5e7b" />

---

### Hierarchical Synthesis (`multiple_modules.v`)
`multiple_modules.v` contains a top module instantiating two sub-modules:
- `sub_module1`: implements an AND gate ($y = a \cdot b$).
- `sub_module2`: implements an OR gate ($y = c + d$).

<img width="1366" height="768" alt="Screenshot (275)" src="https://github.com/user-attachments/assets/b2204476-c991-4744-8066-eb2015860221" />

#### Synthesis Commands:
```text
yosys
yosys> read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> read_verilog multiple_modules.v
yosys> synth -top multiple_modules
yosys> abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> show multiple_modules
```

<img width="1366" height="768" alt="Screenshot (266)" src="https://github.com/user-attachments/assets/8a381e4b-a933-4247-b813-dd5a422d19bb" />

#### Observation:
In the hierarchical view, Yosys preserves the sub-module boundaries, showing `sub_module1` and `sub_module2` as modular hierarchical blocks within the top module.

<!-- Add your Hierarchical Schematic screenshot below -->
<!-- ![Hierarchical Synthesis](images/hierarchical_schematic.png) -->

---

<img width="1366" height="768" alt="Screenshot (271)" src="https://github.com/user-attachments/assets/86ed3fa8-8f42-4c0a-8c6c-39dcb8e16efc" />

<img width="1366" height="768" alt="Screenshot (274)" src="https://github.com/user-attachments/assets/c5b1db43-3a9b-4a72-8229-c6ed204451b8" />

### Flat Synthesis (`flatten`)
In flat synthesis, the module boundaries are dissolved to allow Yosys to optimize logic across module borders:

```text
yosys> flatten
yosys> show
```



#### Observation:
The `sub_module1` and `sub_module2` boundary boxes disappear, and physical Sky130 gates (`sky130_fd_sc_hd__and2_0` and `sky130_fd_sc_hd__or2_0`) are directly instantiated at the top level.


### Sub-Module Level Synthesis (`sub_module1`)
If a specific sub-module needs to be synthesized and characterized independently (e.g., for duplicate instances or timing-critical paths):

```text
yosys
yosys> read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> read_verilog multiple_modules.v
yosys> synth -top sub_module1
yosys> abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> show
```

#### Observation:
Yosys synthesizes ONLY `sub_module1`, directly mapping to a Sky130 AND gate (`sky130_fd_sc_hd__and2_0`).

<!-- Add your sub_module1 Schematic screenshot below -->
<!-- ![sub_module1 Schematic](images/sub_module1_schematic.png) -->

---

<img width="1366" height="768" alt="Screenshot (282)" src="https://github.com/user-attachments/assets/7595b95e-a079-4e86-811e-ceee63414f1a" />

### Hardware Optimization for Multipliers (`mult_2.v`)
Multiplication by powers of 2 ($2^n$, such as $A \times 2$ or $A \times 8$) does **not** require physical hardware multiplier cells:
- Multiplying by 2 corresponds to a simple **1-bit left shift** ($A \times 2 = A \ll 1$), with the LSB tied to ground (`1'b0`).

#### Synthesis Commands:
```text
yosys
yosys> read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> read_verilog mult_2.v
yosys> synth -top mul2
yosys> abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> show
```

#### Observation:
Yosys synthesizes **zero multiplier cells**! It maps input bits `2:0` to output bits `3:1`, and ties output bit `0:0` to constant `1'b0` through pure wire routing.

<!-- Add your Multiplier mul2 Schematic screenshot below -->
<!-- ![Multiplier Optimization](images/mult2_schematic.png) -->

---


<img width="458" height="485" alt="sub_module1_window" src="https://github.com/user-attachments/assets/ffb0f0f6-ec76-401e-97ba-52c0edbddfce" />


## 7. Day 3: Combinational and Sequential Logic Optimizations

### Combinational Logic Optimization (`opt_check.v`, `opt_check3.v`)
In combinational optimization, Yosys identifies:
1. **Constant propagation**: Inputs tied to `0` or `1`.
2. **Boolean minimization**: Redundant gates in boolean expressions.
3. **Dead logic elimination**: Gates whose outputs do not drive any primary outputs.

#### Lab: Synthesizing `opt_check3.v`
```text
yosys
yosys> read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> read_verilog opt_check3.v
yosys> synth -top opt_check3
yosys> opt_clean -purge
yosys> abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> show
```

<img width="1366" height="768" alt="Screenshot (305)" src="https://github.com/user-attachments/assets/adcfa888-cc30-4750-b9b5-a7d4f5896a08" />

#### Observation:
Even though the RTL described complex multiplexed logic, the optimizer simplified the entire boolean equation down to **A SINGLE 3-input AND GATE (`sky130_fd_sc_hd__and3_2`)**!

<!-- Add your opt_check3 Schematic screenshot below -->
<!-- ![opt_check3 Schematic](images/opt_check3_schematic.png) -->

---

<img width="1366" height="768" alt="Screenshot (312)" src="https://github.com/user-attachments/assets/788a1997-4865-4fbe-a0e1-9356d6f4ee6f" />

### Sequential Logic Optimization: Constant Flop Optimization (`dff_const1` to `dff_const4`)

Sequential optimization analyzes D-Flip-Flop behavior under constant inputs:

| Design | Description | Reset State | Data ($D$) | Flop Retained? | Explanation |
| :--- | :--- | :--- | :--- | :---: | :--- |
| **`dff_const1`** | Flop with Async Reset | $Q = 0$ | $D = 1$ | **YES** | Output toggles between 0 (on reset) and 1 (on clock). Flop is required (`sky130_fd_sc_hd__dfxtp_1`). |
| **`dff_const2`** | Flop with Async Reset | $Q = 1$ | $D = 1$ | **NO** | Output is always 1 in all conditions. Flop is optimized away. |
| **`dff_const3`** | Dual Flop Architecture | Mixed | Constants | **YES** | Two Flip-Flops retained (`dfstp` and `dfrtp`). |
| **`dff_const4`** | Constant Output Flop | $Q = 1$ | $D = 1$ | **NO** | Output never changes from 1. Both Flip-Flops deleted; outputs tied directly to `1'b1`. |



#### Simulation of `dff_const1` (`iverilog` & `gtkwave`):
```bash
iverilog dff_const1.v tb_dff_const1.v
./a.out
gtkwave tb_dff_const1.vcd &
```

<img width="1366" height="768" alt="Screenshot (314)" src="https://github.com/user-attachments/assets/2d14a5c0-57ee-41a7-85f2-1d51e24edef8" />


<img width="1366" height="768" alt="Screenshot (320)" src="https://github.com/user-attachments/assets/fa4d0bd0-45d3-4eaf-a29a-3cd8630ce839" />

<img width="1366" height="768" alt="Screenshot (327)" src="https://github.com/user-attachments/assets/2b4c48e4-52e6-4be2-99f9-b040cd056d33" />

- At simulation time $0 - 1500\text{ ns}$, `reset = 1`, holding $q = 0$.
- At $1550\text{ ns}$, `reset` drops to $0$, and on the clock edge, $q$ transitions to $1$.

<!-- Add your dff_const1 Waveform screenshot below -->
<!-- ![dff_const1 Waveform](images/dff_const1_waveform.png) -->
### Similarly for `dff_const2` (`iverilog` & `gtkwave`);

<img width="1366" height="768" alt="Screenshot (322)" src="https://github.com/user-attachments/assets/a2935cd6-2ced-41b6-a06f-2422b7e8b764" />
### Similarly for `dff_const3` (`iverilog` & `gtkwave`);
<img width="1366" height="768" alt="Screenshot (329)" src="https://github.com/user-attachments/assets/f5947055-5db2-4026-b1de-4d5994d3a920" />
<img width="1366" height="768" alt="Screenshot (337)" src="https://github.com/user-attachments/assets/7b2f6812-3f94-49b8-bf42-82048bcc9944" />

#### Synthesis of `dff_const4` (Complete Flop Elimination):
```text
yosys
yosys> read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> read_verilog dff_const4.v
yosys> synth -top dff_const4
yosys> dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
yosys> show
```

#### Observation:
In `dff_const4`, because the output is perpetually `1`, Yosys eliminates both Flip-Flops and routes constant `1'b1` through buffers to outputs `q` and `q1`!

<!-- Add your dff_const4 Schematic screenshot below -->
<!-- ![dff_const4 Schematic](images/dff_const4_schematic.png) -->

---
<img width="698" height="687" alt="WhatsApp Image 2026-11-12 at 7 37 53 PM" src="https://github.com/user-attachments/assets/fb11768d-7653-40af-a199-78819562f164" />

### Sequential Logic Optimization: Unused Flop Elimination (`counter_opt.v` vs `counter_opt2.v`)

#### 1. `counter_opt.v` (Unused Flop Pruning):
In `counter_opt.v`, a 3-bit counter (`reg [2:0] count`) is implemented, but only bit 0 is connected to the primary output:
```verilog
assign q = count[0];
```
- Bits `count[1]` and `count[2]` do not drive any outputs.
- During synthesis, Yosys deletes the 2 unused Flip-Flops and creates **ONLY 1 physical Flip-Flop** (`sky130_fd_sc_hd__dfxtp_1`).

<!-- Add your counter_opt 1-flop Schematic screenshot below -->
<!-- ![counter_opt 1-flop Schematic](images/counter_opt_schematic.png) -->
<img width="1366" height="768" alt="Screenshot (344)" src="https://github.com/user-attachments/assets/1bf1634d-80bc-4f92-92e0-52452f9dbcde" />

<img width="1366" height="768" alt="Screenshot (340)" src="https://github.com/user-attachments/assets/579e00e4-c3a3-4b36-a3a1-14458aa338fd" />

#### 2. `counter_opt2.v` (Full Flop Scaling):
In `counter_opt2.v`, the output is modified so that all 3 counter bits are required to compute the output:
```verilog
assign q = (count == 3'b100);
```
- Because evaluating `count == 3'b100` requires all 3 bits of `count`, **none of the Flip-Flops can be deleted**.
- Yosys synthesizes **all 3 physical Flip-Flops** along with the comparator logic gates (NAND, NOR, Inverters)!

<!-- Add your counter_opt2 3-flop Schematic screenshot below -->
<!-- ![counter_opt2 3-flop Schematic](images/counter_opt2_schematic.png) -->

---

<img width="710" height="649" alt="WhatsApp Image 2026-11-12 at 7 42 31 PM" src="https://github.com/user-attachments/assets/4c48c206-89f9-44d5-af0d-128df6768a0e" />

## 8. Synthesis Command Quick Reference

| Task | Command in Yosys |
| :--- | :--- |
| **Load Liberty File** | `read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib` |
| **Read Verilog Design** | `read_verilog <design_name>.v` |
| **Synthesize Top Module** | `synth -top <module_name>` |
| **Map Flip-Flops** | `dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib` |
| **Map Combinational Gates** | `abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib` |
| **Clean Unused Logic** | `opt_clean -purge` |
| **Flatten Hierarchy** | `flatten` |
| **Display Schematic** | `show` or `show <module_name>` |
| **Save Clean Netlist** | `write_verilog -noattr <output_netlist>.v` |

---

## 9. Author & Acknowledgements

- **Author:** Divyashree K
- **Course:** VSD - SKY130 RTL Design and Synthesis Workshop
- **Instructor:** Kunal Ghosh, VLSI System Design (VSD)
- **Technology:** SkyWater 130nm PDK (`sky130_fd_sc_hd`)
- **Tools:** Icarus Verilog (`iverilog`), GTKWave, Yosys Open Synthesis Suite
