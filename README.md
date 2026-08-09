# 5-Stage Pipelined RISC-V Processor

A SystemVerilog implementation of a 32-bit 5-stage pipelined RISC-V core (RV32I subset) featuring dynamic hazard detection, pipeline stalling, and data forwarding mechanisms to achieve high throughput and low latency execution.

---

## 📌 Features & Architecture

* **5-Stage Execution Pipeline:**
  * **IF (Instruction Fetch):** Increments PC and fetches instructions from Instruction Memory.
  * **ID (Instruction Decode):** Decodes instruction fields, reads register operands, and generates immediates.
  * **EX (Execute):** Executes ALU arithmetic/logic operations and calculates branch addresses.
  * **MEM (Memory Access):** Performs byte/word read and write accesses to Data Memory.
  * **WB (Writeback):** Writes memory or ALU results back to the Register File.
* **Data Forwarding Unit:** Resolves RAW (Read-After-Write) hazards by bypassing data directly from `EX/MEM` or `MEM/WB` stages to the ALU inputs in `EX` stage without stalling.
* **Hazard Detection Unit:** Detects Load-Use dependencies and inserts single-cycle stalls (pipeline freezes & NOP insertions) along with control hazard flushing on branches.
* **Modular Testbench:** Includes virtual RAM memory model and interface-based stimulus drive for self-checking verification.

---

## 📁 Repository Structure

```text
riscv_5stage_pipeline/
├── rtl/
│   ├── alu.sv                 # Arithmetic Logic Unit
│   ├── reg_file.sv            # 32x32-bit Register File
│   ├── forwarding_unit.sv     # Data Forwarding logic
│   ├── hazard_unit.sv         # Stalling & Flushing hazard logic
│   ├── pipeline_regs.sv       # IF/ID, ID/EX, EX/MEM, MEM/WB registers
│   └── riscv_top.sv           # Core Top Module
├── tb/
│   ├── riscv_if.sv            # SystemVerilog Interface
│   ├── riscv_mem_model.sv     # Virtual RAM & Instruction Loader
│   └── riscv_tb_top.sv        # Top-level Testbench Module
├── sim/
│   └── filelist.f             # Simulation source file manifest
└── README.md
