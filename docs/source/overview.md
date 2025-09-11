# Overview

![](images/riscv32-core-arch.png)

## Description

4-stage, single issue, inorder pipelined CPU  
simply stall the pipeline via `PipelineCtrl` for the execution or memory access.

- Fetch
- Decode
- Execution
- Writeback

## Stages

### Fetch stage

Determine the next program counter,
and fetch instruction from instruction memory.

Next PC is determined according to `branch predictor`, `branch unit`, and `CSR`.

#### Modules and Logic Blocks

- PC
- branch predictor
- EX next pc logic
- branch flush logic
- next pc logic

### Decode stage

Decode the instruction and get data from `Register` or `FRegister`.

#### Modules and Logic Blocks

- ID PipelineRegister
- ImmGen
- Control

### Execution stage

Execute the instruction or perform Memory access.

#### Modules and Logic Blocks

- EX PipelineRegister
- ForwardUnit
- Input Selecting Mux
- EX start logic
- EX done logic
- Functional Units
    - Bypass
    - LSU
    - ALU
    - MUL_DIV
    - FPU
    - BranchUnit
- Exception Handler
- CSR

### Writeback stage

Write the value to Registers.

#### Modules and Logic Blocks

- WB PipelineRegister
- Writeback Mux
- Write enable logic
