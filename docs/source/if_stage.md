# Instruction Fetch Stage

## Branch Predictor

The primary role of the Branch Predictor is to anticipate whether the instruction at the current PC is a branch, and if so, whether it will be taken and to which target address. In the IF stage, the CPU relies on the prediction to determine the next fetch address, while in the EX stage the actual branch outcome is resolved and compared against the prediction. If a misprediction occurs, the **Branch History Table (BHT)** must be updated and the pipeline (e.g., IF/ID and ID/EX) flushed to remove incorrect instructions, ensuring correct program execution.

### Design Rationale

The **Branch History Table (BHT)** and **Branch Target Buffer (BTB)** complement each other in branch prediction.

- **Branch History Table:** The BHT is a table that records the recent behavior of branch instructions, typically using one or more bits per entry to track whether a branch was taken or not. When the CPU fetches an instruction, the PC is used to index into the BHT, and the stored value provides a prediction about whether the branch will be taken. This prediction guides the Instruction Fetch stage to select the next PC. After the actual outcome of the branch is resolved in the Execute stage, the corresponding BHT entry is updated to reflect the new branch behavior, enabling the predictor to learn from past execution history.
    - **2-bit BHT:** To improve stability, each entry uses a 2-bit saturating counter instead of a single bit. The counter changes state only after two consecutive mispredictions, making the predictor less sensitive to occasional deviations.
    - **GHR:** While 2-bit BHT only consider the behavior of one branch, many programs have correlated branches (e.g., the outcome of one branch depends on a previous one). The GHR stores the outcomes of the most recent branches as a shift register. This global history can then be combined with the PC to form a more informed prediction.
    - **Gshare:** A widely used predictor that combines the PC bits and the GHR by XORing them to form the index into the BHT. This spreads different history patterns across the table, reducing aliasing. In our implementation, we observed that when the BHT size is increased to 2^11 entries (index_bits = 11), using a 4-bit GHR yields the highest prediction accuracy.
- **Branch Target Buffer:** The BTB is a cache-like structure that stores the target addresses of previously executed branch instructions. When the CPU fetches an instruction, the PC is used to index into the BTB. If there is a hit, the BTB provides the predicted target address, allowing the Instruction Fetch stage to redirect execution immediately without waiting for the branch to be resolved in the Execute stage. This significantly reduces the control hazard penalty. The BTB is updated in the Execute stage whenever a branch is resolved, ensuring that future predictions use the most recent target information.

The BHT predicts whether a branch will be taken or not, while the BTB provides the corresponding target address if the branch is taken. Together, they enable the Instruction Fetch stage to quickly decide both whether to jump and where to jump. The BHT ensures prediction accuracy in control flow direction, and the BTB reduces penalty by supplying the next PC without waiting for execution. This synergy minimizes control hazards and improves overall pipeline efficiency.

### Functional Breakdown

- **XOR between PC and GHR:** In a Gshare predictor, selected bits of the Program Counter (PC), typically the lower bits, are XORed with the Global History Register (GHR) to form the index into the Branch History Table (BHT). The primary purpose of this XOR operation is to reduce aliasing—the situation where multiple unrelated branches map to the same BHT entry.
- **Target selection based on BHT state and BTB hit:** After XORing the PC with the GHR and accessing the BHT, a saturating counter state is retrieved to determine whether the branch is predicted as taken.
    - If predicted taken and the BTB entry matches (tag hit), the target address from the BTB is used as the next PC.
    - If predicted not taken, or if the BTB does not hit, the next PC is set to PC+4 for sequential fetching.
- **Misprediction handling in the EX stage:** Only branch and jump instructions are subject to branch prediction checks. In the EX stage:
    - **J-type instructions:** The target address is resolved immediately and does not rely on the predictor, so the PC is always updated to the jump target.
    - **B-type instructions:** A misprediction occurs only when the prediction outcome differs from the actual branch result.
    - When a misprediction is detected, the CPU generates a flush signal to remove incorrect instructions from the pipeline and updates the PC in the next cycle to the correct branch target or PC+4.
- **Target selection on misprediction resolution in the EX stage:** 
    - **J-type instructions:** The PC is updated to the branch target computed in the EX stage (PC plus immediate offset).
    - **B-type instructions:** The PC is updated according to the actual branch outcome determined in the EX stage:
        - If taken, update to the computed branch target.
        - If not taken, update to PC+4 for sequential execution.
