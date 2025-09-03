# LSU (Load Store Unit)

In RISC-V core, the Load Store Unit (LSU) is a critical pipeline component responsible for controlling load and store instructions and handling various error conditions, particularly address misalignment. The LSU acts as an interface between the processor's execution stage and the Memory Management Unit (MMU), ensuring proper data transfer, address calculation, and exception handling.

## Design Principle

- **LSU Queue:** The `LSU` implement internal queue to store all necessary dtat for memory operactions. 

- **Address Misaligned:** In this part, we want LSU detect misaligned error and solve it. Therefore, we need to record every condition, including memory addresses, data values, instruction types, and control signals.

- **Writeback Data Calculation:** This part is for load instruction. In our design, lsu will sent read word signal to mmu even if instruction is `WB` or `WH`. Therefore, it needs caculate word data after lsu get terget data.

## Implement Detail

### Module Parameters

- LENGTH: Queue length parameter.
- DEPTH: Queue depth parameter.
- DATASIZE: Internal data size for queue operations.

### Address and Data Procressing

- **Address:** `LSU` will control address which will be sent into `MMU` be mutipler of 4.
- **Data:** `LSU` use `mask` to control which byte of write data are needed to be writen into memory.

### LSU Queue

Input Data (Total 77 bits):

|Signal     |Width  |Description            |
|-----------|-------|-----------------------|
|mem_addr   |32     |Target memory address  |
|mem_data   |32     |value of write data    |
|lb_inst, lh_inst, lw_inst   |1     |Load instruction type|
|signed_inst   |1     |load instruction is signed |
|mem_rd, mem_wr |1     |memory read and write signal|
|mem_mask   |4     |store or load mask|
|u_type     |1|additional instruction for misaligned problem|

Action:

- Push: input instruction is valid, and misaligned condition.
- Pop: when memory return finished signal.

### Address Misaligned

The `LSU` includes sophisticated misalignment detection and correction logic. When misaligned memory accesses are detected, the `LSU` automatically:

- Splits the misaligned access into multiple aligned memory transactions
- Maintains state information across multiple memory cycles
- Reconstructs the final result from partial memory responses
- Ensures atomicity of the original memory operation from an architectural perspective

The LSU handles unaligned memory accesses by splitting them into multiple aligned accesses:

- **Unaligned Detection**
    - Half-word unaligned: Address bits [1:0] = 2'b11
    - Word unaligned: Address bits [1:0] ≠ 2'b00

- **Unaligned Access Handling:** Maintains state machine (u_state) for multi-cycle unaligned operations. Then, automatically generates second memory access for unaligned transfers.Finally, reconstructs data from multiple memory responses in writeback calculation.

### Writeback Value Calculation

For load instructions, the LSU performs intelligent data processing since it always requests full 32-bit words from the MMU regardless of the actual load instruction type (LB, LH, LW). The writeback calculation unit:

Extracts the relevant bytes from the loaded word based on address and instruction type
Performs sign extension or zero extension as required
Handles data reconstruction for misaligned accesses
Manages the timing of writeback operations to the register file

### Exception Handling

In this part, `LSU` will receive signal like `load_fault` which is sended from `MMU`. Then, generates exceptions:

- Load Page Fault (EXCEPTION_PAGE_FAULT_LOAD): When mmu_load_fault is asserted during read
- Store Page Fault (EXCEPTION_PAGE_FAULT_STORE): When mmu_store_fault is asserted during write
