# 5-Stage Pipelined RISC-V Processor

A SystemVerilog implementation of a 32-bit 5-stage pipelined RISC-V core (RV32I subset) featuring dynamic hazard detection, pipeline stalling, and data forwarding mechanisms for high throughput execution.

---

## 📌 DUT Overview

The core implements a classic 5-stage RISC pipeline executing integer instructions from the RV32I ISA:

- **Fetch (`IF`):** Updates the Program Counter (PC) and fetches instructions from Instruction Memory.
- **Decode (`ID`):** Reads register operands, decodes control signals, and generates immediate values.
- **Execute (`EX`):** Performs ALU operations, evaluates branches, and calculates effective target addresses.
- **Memory (`MEM`):** Executes byte/word memory read and write operations to Data Memory.
- **Writeback (`WB`):** Writes ALU or memory execution results back into the 32x32-bit Register File.
- **Hazard Units:** Integrated **Forwarding Unit** (bypasses RAW hazards) and **Hazard Detection Unit** (stalls on Load-Use and flushes on branch misprediction).

---

## 🔌 Core Interface Signals

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

## 🏗️ Core Architecture

### Pipeline & Control Flow
```text
[ IF Stage ] ──> [ ID Stage ] ──> [ EX Stage ] ──> [ MEM Stage ] ──> [ WB Stage ]
     │                │                 │                  │                 │
     └─────── Hazard Detection ◄────────┴────── Data Forwarding ◄────────────┘
│   └── filelist.f             # Simulation source file manifest
└── README.md
