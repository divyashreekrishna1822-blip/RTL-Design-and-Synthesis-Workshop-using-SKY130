# RTL-Design-and-Synthesis-Workshop-using-SKY130

# Introduction to Verilog RTL Design and Simulation

Documentation based on the VSD SKY130 RTL Design and Synthesis Workshop (`SKY130RTL D1SK1`).

---

## 1. Design
- **Design** is the actual Verilog code or set of Verilog codes which has the intended functionality to meet the required specifications.
- It defines the digital logic, ports (inputs and outputs), and behavioral or structural implementation of the hardware block.

---

## 2. Simulator
- **RTL design is checked for adherence to the specifications by simulating the design.**
- **Simulator** is the software tool used for simulating the design.
- It verifies whether the written Verilog code matches the expected behavior before synthesis and manufacturing.

---

## 3. How Simulator Works
- The simulator continuously **looks for changes on the input signals**.
- **Upon change to the input, the output is evaluated.**
- If there is no change in the input signals, no evaluation is performed (event-driven simulation).

---

## 4. Testbench
- A **Testbench** is a Verilog module used to apply test stimulus (inputs) to the Design Under Test (**DUT**).
- It observes and monitors the outputs to verify correct functionality.
- Unlike the design, a testbench does not represent hardware; it contains timing constructs (such as `#10`, `$dumpfile`, `$dumpvars`, and `$finish`) to generate waveforms.

---

## 5. Icarus Verilog (`iverilog`)
- **Icarus Verilog** is an open-source Verilog simulation and synthesis tool.
- It takes the RTL design file and the testbench file as inputs, compiles them, and produces an executable simulation output (e.g., `a.out`).
- Running the executable evaluates the design and dumps the simulation results into a Value Change Dump (`.vcd`) file.

