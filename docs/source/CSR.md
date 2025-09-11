# Table of Contents

- Overview
- Summary Table: CSR Address, Name, Privilege, Access, Implementation Status
- Machine-level CSRs (field-by-field details)
- User-level CSRs (field-by-field details)
- Custom / Non-standard CSRs
- Behavioral Notes (reset values, exception/interrupt behavior, mip buffer, mtimercmp behavior, etc.)
- Appendix: Notes & TODOs (SATP PPN length, S-mode placeholders, etc.)

---

# Overview

This CSR collection is sourced from the **CSRFile Verilog module**.  
The core currently supports:

- Full M-mode CSR logic implementation  
  (interrupt/exception handling, `mepc`, `mcause`, `mtval`, `mstatus`, `mie`, `mip`, `mtvec`, `mscratch`, `mcounter`, FPU flag interface, etc.)
- Partial U-mode read-only counter CSRs  
  (`cycle`, `time`, `instret` — lower 32-bit half only)
- Several **custom / non-standard CSRs**  
  (e.g., `MTIMECMP`, simulation control, cache flush, etc.)
- Many registers pre-declared for future use  
  (`PMP`, `mhpm`, `mstateen`, `mn*`, `menvcfg`, etc.)

This document lists each CSR with details:

**Address / Privilege / Access / Reset / Implemented? / Description / Notes**

---

# CSR Summary (Key Points)

The table uses **12-bit hex addresses**.  
"Implemented" means the CSR is explicitly handled in read/write/reset/seq logic.

| Addr  | Name       | Priv | Access | Implemented | Reset / Notes |
|-------|-----------|------|--------|-------------|---------------|
| 0x300 | mstatus   | M    | RW     | Yes (R/W)   | Reset = `0x00001800` |
| 0x301 | misa      | M    | RO     | Yes (from `misa_i`) | Impl-defined |
| 0x302 | medeleg   | M    | RW     | Declared, masked | `CSR_MEDELEG_MASK` |
| 0x303 | mideleg   | M    | RW     | Declared, masked | `CSR_MIDELEG_MASK` |
| 0x304 | mie       | M    | RW     | Yes (masked) | `CSR_MIE_MASK` |
| 0x305 | mtvec     | M    | RW     | Yes (masked) | `CSR_MTVEC_MASK`, aligned |
| 0x340 | mscratch  | M    | RW     | Yes          | Reset = 0 |
| 0x341 | mepc      | M    | RW     | Yes          | Reset = 0 |
| 0x342 | mcause    | M    | RW     | Yes (masked) | `CSR_MCAUSE_MASK` |
| 0x343 | mtval     | M    | RW     | Yes          | Reset = 0 |
| 0x344 | mip       | M    | RW     | Yes (buffered update) | `CSR_MIP_MASK` |
| 0xB00 | mcycle    | U/M  | RO(lo) | Yes          | Increments, 32-bit low |
| 0xB80 | mcycleh   | U/M  | RO(hi) | Yes          | Updates on overflow |
| 0xB01 | mtime     | U/M  | RO(lo) | Aliased → `mcycle` | Non-standard |
| 0xC81 | mtimeh    | U/M  | RO(hi) | Aliased → `mcycleh` | |
| 0xF14 | mhartid   | M    | RO     | Yes (from `cpu_id_i`) | Impl-defined |
| 0x7C0 | mtimecmp  | M*   | RW     | Yes (custom) | Sets `csr_mtime_ie_r` |
| 0x180 | satp      | S    | RW     | Yes (stored, masked) | S-mode not implemented |
| 0x001 | fflags    | U    | RW     | Yes (masked) | |
| 0x002 | frm       | U    | RW     | Yes (masked) | |
| 0x003 | fcsr      | U    | RW     | Yes (pack/unpack frm+fflags) | |
| 0x7B2 | dscratch  | Custom | RW   | Declared     | May be simulation-use |
| 0x8B2 | sim_ctrl  | Custom | RW/WO| Declared     | Non-standard: sim exit/putc |
| 0x3A0 | dflush    | Custom | WO   | Declared     | Data-cache control |
| 0x3A1 | dwriteback| Custom | WO   | Declared     | |
| 0x3A2 | dinvalidate| Custom| WO   | Declared     | |

**Planned / Reserved (declared but not mapped):**

`csr_mvendorid_q, csr_marchid_q, csr_mimpid_q, csr_mconfigptr_q, csr_mcounteren_q, csr_mstatush_q, csr_medelegh_q, csr_mtinst_q, csr_mtval2_q, csr_menvcfg_q, csr_menvcfgh_q, csr_mseccfg_q, csr_mseccfgh_q, csr_pmpcfg_q, csr_pmpaddr_q, csr_mstateen_q, csr_mstateenh_q, csr_mn* CSRs, csr_minstret_q, csr_mhpmcounter_q family, csr_mcountinhibit_q, csr_mhpmevent_q family.`

---

# Machine-level CSRs (Details)

## `mstatus` — 0x300
- Priv: M  
- Access: RW  
- Impl: Yes  
- Reset: `0x00001800`  

### Fields:
- UIE(0), SIE(1), MIE(3)  
- UPIE(4), SPIE(5), MPIE(7)  
- SPP(8), MPP(12:11)  
- FS(14:13), XS(16:15)  
- MPRV(17), SUM(18), MXR(19), SD(31)  

**Notes:**
- MPP sanitized to `00` or `11`.  
- Exception/interrupt updates `MPIE`, `MPP`, `MIE`.  
- S-mode fields exist but not functional.  

---

## `misa` — 0x301
- Priv: M  
- Access: RO  
- Impl: Yes (from `misa_i`)  
- Reset: Impl-defined  

---

## `medeleg` — 0x302  
- Priv: M  
- Access: RW  
- Impl: Yes (masked)  
- Reset: 0  

---

## `mideleg` — 0x303  
- Same as medeleg (masked, reset 0).  

---

## `mie` — 0x304  
- Priv: M  
- Access: RW  
- Impl: Yes (masked by `CSR_MIE_MASK`)  
- Reset: 0  

---

## `mtvec` — 0x305  
- Priv: M  
- Access: RW  
- Impl: Yes (masked, aligned)  
- Reset: 0  
- Target for interrupts/exceptions  

---

## `mscratch` — 0x340  
- RW, Reset=0  

## `mepc` — 0x341  
- RW, Reset=0  
- Holds exception PC, used in `mret`.  

## `mcause` — 0x342  
- RW, Reset=0  
- Mask = `CSR_MCAUSE_MASK`  
- Encodes exception/interrupt cause.  

## `mtval` — 0x343  
- RW, Reset=0  
- Updated with faulting PC/address.  

## `mip` — 0x344  
- RW, Reset=0  
- Mask = `CSR_MIP_MASK`  
- Includes buffered update mechanism.  

---

## `mcycle` / `mcycleh` (0xB00 / 0xB80)
- Priv: U/M  
- RO counters  
- Auto-incrementing  
- 64-bit split across lo/hi.  

---

## `mtimecmp` — 0x7C0 (Custom)
- Priv: M*  
- RW, Reset=0  
- When `mcycle == mtimecmp`, sets `MTIP`.  

---

## `mhartid` — 0xF14  
- RO, from `cpu_id_i`  

---

# User-level CSRs

## `fflags` — 0x001  
- RW, Reset=0  
- Also updated by FPU events.  

## `frm` — 0x002  
- RW, Reset=0  

## `fcsr` — 0x003  
- RW  
- Packs `frm + fflags`.  

## `cycle` / `time` / `instret` (0xC00 / 0xC01 / 0xC02)  
- RO, lower 32-bit only  
- `time` aliased to `mcycle` (non-standard)  
- `instret` declared but not mapped.  

---

# Custom / Non-standard CSRs

- `MTIMECMP (0x7C0)` — custom timer compare.  
- `DSCRATCH (0x7B2)`, `SIM_CTRL (0x8B2)` — simulation control.  
- `DFLUSH / DWRITEBACK / DINVALIDATE (0x3A0–0x3A2)` — cache control, declared only.  

---

# Behavioral Notes

- **Reset state:**  
  `csr_priv_q = PRIV_MACHINE`  
  `csr_mstatus_q = 0x00001800`  
  Most CSRs = 0  

- **Interrupt handling:**  
  `irq_pending_r = csr_mip_q & csr_mie_q`  
  Masked by global `MIE`.  
  Interrupt entry updates `mepc`, `mcause`, `MIE`, `MPIE`, `MPP`.  

- **MIP buffering:**  
  Read sets buffer, write clears.  

- **mtime / mtimecmp:**  
  `mtime` aliased to `mcycle`.  
  `mtimecmp` triggers MTIP then disables itself.  

- **Exceptions / mret:**  
  mret restores priv, MIE/MPIE per RISC-V rules.  
  sret placeholder only.  

- **FPU flags:**  
  Written by FPU completion path or CSR writes.  