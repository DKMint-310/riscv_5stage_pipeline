# 5-Stage Pipelined RISC-V Processor

A SystemVerilog implementation of a 32-bit 5-stage pipelined RISC-V core (RV32I subset) featuring dynamic hazard detection, pipeline stalling, and data forwarding mechanisms for high throughput execution.

---

## DUT Overview

The core implements a classic 5-stage RISC pipeline executing integer instructions from the RV32I ISA:

- **Fetch (`IF`):** Updates the Program Counter (PC) and fetches instructions from Instruction Memory.
- **Decode (`ID`):** Reads register operands, decodes control signals, and generates immediate values.
- **Execute (`EX`):** Performs ALU operations, evaluates branches, and calculates effective target addresses.
- **Memory (`MEM`):** Executes byte/word memory read and write operations to Data Memory.
- **Writeback (`WB`):** Writes ALU or memory execution results back into the 32x32-bit Register File.
- **Hazard Units:** Integrated **Forwarding Unit** (bypasses RAW hazards) and **Hazard Detection Unit** (stalls on Load-Use and flushes on branch misprediction).

---

## Interface 

| Signal | Direction | Width | Description |
|:---|:---:|:---:|:---|
| `clk` | Input | 1 | System clock |
| `rstn` | Input | 1 | Active-low asynchronous reset |
| `imem_addr` | Output | 32 | Instruction Memory address driven by IF stage (PC) |
| `imem_rdata` | Input | 32 | Fetched instruction word from Instruction Memory |
| `dmem_addr` | Output | 32 | Data Memory address driven by EX/MEM pipeline stage |
| `dmem_wdata` | Output | 32 | Write data bus to Data Memory |
| `dmem_rdata` | Input | 32 | Read data bus from Data Memory |
| `dmem_we` | Output | 1 | Write enable pulse to Data Memory |
| `dmem_re` | Output | 1 | Read enable signal to Data Memory |

---

## Architecture

### Pipeline & Control Flow
```text
[ IF Stage ] ──> [ ID Stage ] ──> [ EX Stage ] ──> [ MEM Stage ] ──> [ WB Stage ]
     │                │                 │                  │                 │
     └─────── Hazard Detection ◄────────┴────── Data Forwarding ◄────────────┘
```

### Module Components

| Module | File Location | Role & Functionality |
|:---|:---|:---|
| `reg_file` | `rtl/reg_file.sv` | 32x32-bit dual-read single-write register file (`x0` hardwired to 0). |
| `alu` | `rtl/alu.sv` | Core ALU supporting arithmetic, logic, shifts, and comparisons (`SLT`). |
| `forwarding_unit` | `rtl/forwarding_unit.sv` | Bypasses data from `EX/MEM` and `MEM/WB` stages directly to `EX` stage operands. |
| `hazard_unit` | `rtl/hazard_unit.sv` | Generates pipeline stalls for Load-Use dependencies and flushes on branches. |
| `pipeline_regs` | `rtl/pipeline_regs.sv` | Inter-stage pipeline registers (`IF/ID`, `ID/EX`, `EX/MEM`, `MEM/WB`). |
| `riscv_top` | `rtl/riscv_top.sv` | Top-level processor core integrating all 5 pipeline stages and hazard units. |
| `riscv_if` | `tb/riscv_if.sv` | SystemVerilog interface bridging core top with memory and verification components. |
| `riscv_mem_model` | `tb/riscv_mem_model.sv` | Unified virtual associative RAM model acting as Instruction and Data Memory. |
| `riscv_tb_top` | `tb/riscv_tb_top.sv` | Top-level testbench module generating clock, reset, and launching simulation. |

---

## Test Scenarios

| Test Case / Feature | Description |
|:---|:---|
| **EX-to-EX Forwarding** | Direct data bypass from `EX/MEM` register to ALU input for consecutive arithmetic instructions. |
| **MEM-to-EX Forwarding** | Data bypass from `MEM/WB` register to ALU input when instructions are separated by one cycle. |
| **Load-Use Stall** | Single-cycle pipeline freeze (stalls `IF`/`ID`, inserts `NOP` in `EX`) on `LW` followed by dependent R-type instruction. |
| **Branch Control Flush** | Clears mispredicted instructions in pipeline when a branch/jump is taken in `EX` stage. |

---

## How to Run Simulation

Execute the following commands from the `sim/` directory using AMD Vivado (XSim):

### 1. Compile & Elaborate

```bash
xvlog -sv -f filelist.f --default_timeunit 1ns/1ps
xelab riscv_tb_top -debug typical -s tb_top -override_timeunit -timescale 1ns/1ps
```
### 2. Run Simulation

```bash
xsim tb_top -runall
```
