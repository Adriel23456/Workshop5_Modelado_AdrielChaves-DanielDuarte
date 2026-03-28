# Interactive System Proposal

## a. System Name

**ARM5 Pipeline Sim** — A 5-stage ARM pipeline processor simulator with real-time stage visualization, built in Python.

---

## b. General Description

**ARM5 Pipeline Sim** is a software-based simulator written in Python that 
models the behavior of a classic 5-stage pipeline processor following the 
ARM architecture. The simulator executes a reduced but representative subset 
of the ARM instruction set, visualizing in real time how instructions flow 
through each stage of the pipeline, including hazard detection, stalling, 
and forwarding mechanisms.

---

### Pipeline Architecture Overview

The simulated pipeline follows the classic 5-stage RISC model:
```
  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
  │  FETCH  │──▶│ DECODE  │──▶│EXECUTE  │──▶│ MEMORY  │──▶│ WRITE   │
  │  (IF)   │   │  (ID)   │   │  (EX)   │   │  (MEM)  │   │  BACK   │
  │         │   │         │   │         │   │         │   │  (WB)   │
  └─────────┘   └─────────┘   └─────────┘   └─────────┘   └─────────┘
       │               │             │              │              │
  Fetch instr.    Decode &      ALU operation   Load/Store    Write result
  from memory     read regs     or branch eval  data memory   to register
```

---

### Stage Descriptions

| Stage | Name | Description |
|---|---|---|
| IF | Instruction Fetch | Reads the next instruction from memory using the Program Counter (PC). |
| ID | Instruction Decode | Decodes the opcode, reads source registers, and detects hazards. |
| EX | Execute | Performs the ALU operation (ADD, SUB, AND, ORR, CMP, etc.) or evaluates a branch condition. |
| MEM | Memory Access | Reads from or writes to data memory (LDR / STR instructions). |
| WB | Write Back | Writes the result of the operation back to the destination register. |

---

### Supported ARM Instruction Subset

The simulator supports a representative subset of the ARMv4T / ARMv5 
instruction set, covering the most common data processing, memory, and 
control flow operations:

| Category | Instructions |
|---|---|
| Data Processing | `ADD`, `SUB`, `MUL`, `AND`, `ORR`, `EOR`, `MOV`, `CMP` |
| Memory | `LDR`, `STR` |
| Control Flow | `B`, `BEQ`, `BNE`, `BL` |
| No-Operation | `NOP` |

---

### Hazard Handling

One of the core features of the simulator is its accurate modeling of 
pipeline hazards — situations where the pipeline cannot proceed normally 
due to data or control dependencies between instructions.

#### Data Hazards

Occur when an instruction depends on the result of a previous instruction 
that has not yet completed.
```
  Instruction 1:  ADD R1, R2, R3      ← produces R1 in EX stage
  Instruction 2:  SUB R4, R1, R5      ← needs R1, but it's not ready yet!
```

**Resolution strategies modeled:**

| Strategy | Description |
|---|---|
| Stalling (bubble insertion) | Pipeline is paused by inserting NOP bubbles until the dependency is resolved. |
| Data Forwarding / Bypassing | Result is forwarded directly from EX or MEM stage to the EX input of a later instruction, avoiding stalls when possible. |

#### Control Hazards

Occur on branch instructions, where the next PC value is not known until 
the branch is evaluated in the EX stage.
```
  Branch taken:    PC jumps to target → instructions already fetched are flushed
  Branch not taken: Pipeline continues normally
```

**Resolution strategy modeled:**

| Strategy | Description |
|---|---|
| Flush on branch | Instructions fetched after a branch are flushed if the branch is taken. |

---

### Register File

The simulator models the ARM 16-register general-purpose register file:

| Register | ARM Name | Common Role |
|---|---|---|
| R0 – R3 | a1 – a4 | Function arguments / return values |
| R4 – R11 | v1 – v8 | General purpose (callee-saved) |
| R12 | IP | Intra-procedure call scratch register |
| R13 | SP | Stack Pointer |
| R14 | LR | Link Register (return address) |
| R15 | PC | Program Counter |

---

### System Architecture (Software Modules)

The simulator is structured as a set of Python modules, each representing 
a logical component of the processor:
```
  ┌──────────────────────────────────────────────────────────┐
  │                     ARM5 Pipeline Sim                    │
  │                                                          │
  │  ┌──────────┐  ┌──────────┐  ┌────────────────────────┐ │
  │  │  Parser  │  │  Memory  │  │    Register File        │ │
  │  │  Module  │  │  Module  │  │    (R0 – R15)           │ │
  │  └────┬─────┘  └────┬─────┘  └───────────┬────────────┘ │
  │       │             │                    │               │
  │  ┌────▼─────────────▼────────────────────▼────────────┐  │
  │  │               Pipeline Controller                   │  │
  │  │   IF  │  ID  │  EX  │  MEM  │  WB                  │  │
  │  └────────────────────────┬───────────────────────────┘  │
  │                           │                              │
  │              ┌────────────▼────────────┐                 │
  │              │   Hazard Detection &    │                 │
  │              │   Forwarding Unit       │                 │
  │              └────────────┬────────────┘                 │
  │                           │                              │
  │              ┌────────────▼────────────┐                 │
  │              │   Visualization Layer   │                 │
  │              │   (CLI / GUI output)    │                 │
  │              └─────────────────────────┘                 │
  └──────────────────────────────────────────────────────────┘
```

---

### Output and Visualization

At each clock cycle, the simulator outputs a snapshot of the pipeline state, 
showing which instruction occupies each stage, the current register file 
values, and any stalls or flushes that occurred:
```
  Cycle 5:
  ┌─────────┬──────────────┬──────────────┬──────────────┬──────────────┐
  │   IF    │     ID       │     EX       │    MEM       │     WB       │
  ├─────────┼──────────────┼──────────────┼──────────────┼──────────────┤
  │ LDR R5  │  SUB R4,R1  │  ** STALL ** │  ADD R1,R2  │  MOV R3,#10  │
  └─────────┴──────────────┴──────────────┴──────────────┴──────────────┘
  Hazard detected: RAW dependency on R1 → stall inserted at EX
```

---

### Technology Stack

| Component | Technology |
|---|---|
| Core simulator | Python 3.x |
| Instruction parser | Custom lexer/parser (regex-based) |
| Pipeline state management | Python dataclasses / dictionaries |
| Visualization | Rich (CLI tables) or Tkinter (GUI, optional) |
| Testing | pytest |
| Version control | Git + GitHub |

---

## c. Target User

Computer Engineering students and professionals interested in understanding CPU execution mechanics at the instruction level.

---

## d. Inputs

| Sensor / Source | Signal Type |
|---|---|
| | |
| | |

> *To be completed.*

---

## e. Outputs

| Actuator / Output | Signal Type |
|---|---|
| | |
| | |

> *To be completed.*

---

## Notes

> *Any additional observations, constraints, or design decisions go here.*