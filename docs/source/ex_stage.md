# Execute Stage

## Multiplier

The 32-bit Multiplier is a dedicated functional unit designed to perform high-speed multiplication for the RISC-V instruction set architecture. Its primary purpose is to provide the required `MUL`, `MULH`, `MULHSU`, and `MULHU` operations.

### Design Rationale

This multiplier employs a combination of the **Radix-4 Booth algorithm** and a **Wallace Tree structure** to achieve high performance and efficient hardware utilization.

- **Radix-4 Booth Decoding:** The Booth algorithm is a multiplication technique that reduces the number of partial products that need to be summed. For a 32-bit multiplier, using Radix-4 effectively halves the number of partial products compared to a simple bit-by-bit multiplication. This simplification is crucial as it directly translates to less hardware in the subsequent summation stages.
- **Wallace Tree:** A Wallace Tree is a highly optimized carry-save adder (CSA) tree that efficiently reduces a large number of partial product rows to just two rows in a logarithmic number of steps. Its advantage lies in its ability to avoid full carry propagation in its intermediate stages, thus keeping the combinational delay low per level of the tree.

By combining these two techniques, the multiplier minimizes the number of terms to sum, then efficiently sums them in a parallel, non-carry-propagating manner.

### Functional Breakdown

- **Radix-4 Booth Encoding & Partial Product Generation:** This initial stage prepares the multiplicand (rs1) and multiplier (rs2) based on the specific RISC-V instruction. The 32-bit multiplier is fed into the **Radix-4 Booth Decoder**. This decoder examines overlapping groups of 3 bits of the multiplier (including an implicit trailing zero) to generate 34 bits data for each partial product (1 sign bit + 32 data bits).
Based on the 34 bits data, the actual partial product rows are generated. Since Radix-4 processes two bits at a time, 17 partial products are generated. These partial products are already appropriately shifted according to their bit position.
- **Wallace Tree Reduction:** The numerous partial product rows are then fed into the Wallace Tree. This tree is composed of a 3-to-2 compressor and 3 layers of 4-to-2 compressors. In each layer, three input rows are reduced to two output rows without immediate carry propagation across the full width. This process continues until only two rows remain. The Wallace tree is pipelined itself, meaning it may span multiple clock cycles, with pipeline registers inserted between layers of compressors.
- **Final Summation:** The two remaining 64-bit vectors represent the complete 64-bit product in carry-save form. These two vectors are then fed into an adder. This is the only stage where full carry propagation occurs across the entire width of the 64-bit product.

## Divider

The Radix-4 SRT Divider is a dedicated functional unit designed to perform high-speed integer division for the RISC-V instruction set. Its primary purpose is to efficiently compute the quotient and remainder for all `DIV`, `DIVU`, `REM`, and `REMU` operations.

### Design Rationale

This divider is built around the **SRT division algorithm** to achieve high performance. The SRT algorithm is a family of digit-recurrence algorithms that determine multiple bits of the quotient in each clock cycle, thereby accelerating the division process.

- **Radix-4:** The divider generates two quotient bits at a time. It does this by using a base-4 representation internally, where each quotient digit can take on values from a redundant set, `{-2, -1, 0, 1, 2}`. Processing two bits at a time drastically reduces the number of iterations needed to complete a 32-bit division.
- **Redundancy:** The use of a redundant quotient digit set is the key to the SRT algorithm's speed. It allows the divider to select the next quotient digit based on only a few of the most significant bits of the partial remainder, without waiting for the slow process of full carry propagation. This simplifies the decision-making logic and allows the core iterative loop to run at a higher clock speed.

### Functional Breakdown

- **Pre-processing and Normalization:** Before the iterative process begins, the dividend and divisor are prepared and represented as 64-bit numbers. A key step is normalization, where the divisor is left-shifted until its most significant `1` bit is at a predetermined position. The dividend is shifted by the same amount, and the number of shifts is recorded. This ensures the operands are properly aligned for the SRT iterations and handles cases with leading zeros efficiently.
- **Core Iterative Loop:** Main part of the SRT divider, implemented as a multi-cycle iterative loop. For each iteration 2 bits of quotient in redundant form are generated, and the remainder is updated. Crucially, the additions and subtractions are performed using a Carry-Save Adder (CSA), avoiding the long carry-propagation delay of a standard adder. The partial remainder is stored in a carry-save format to facilitate this fast addition.
- **Quotient Digit Selection:** Within the recurrence loop, quotient bits are determined by the Quotient Digit Selection logic. The QDS logic examines 6 of the most significant bits of the partial remainder and the divisor to select the next 2 digits. This eliminates the need to compute the full partial remainder in each cycle, dramatically speeding up the process.
- **Finalization:** After the required number of iterations, the divider has a quotient in a redundant Radix-4 form and a final partial remainder still in carry-save form. The final 2 clock cycles performs the necessary steps:
    - Restore remainder and quotient if needed.
    - The redundant quotient digits are converted into a standard two's complement binary number.
    - The final remainder (in its carry-save form) is converted into a standard binary representation.
