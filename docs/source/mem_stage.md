# Memory Stage

Units:

1. LSU(Load Store Unit)
2. MMU(Memory Management Unit)

## LSU (Load Store Unit)

In risc-v core, load store unit need to control load and store instruction and deal with some error, such as address misaligned.

### Design Principle

- LSU Queue

- Writeback Data Calculation

- Address Misaligned

## MMU (Memory Management Unit)

Memory management unit (MMU) is reponsible for translation of virtual address and physical address.

### Design Principle

- Table Lookahead Buffer (TLB)

- Page Table walker (PTW)

- Cache Control
