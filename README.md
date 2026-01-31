# Technical Memo: CORE-V-VERIF CV32E40P Quick Start

**Date:** January 31, 2026  
**Author:** Mohammed Omer  
**Subject:** CV32E40P Functional Verification Environment Setup

---

## Tool Versions Installed

| Tool | Version | Installation Path |
|------|---------|-------------------|
| **Verilator** | 5.044 (2026-01-01) | `~/verilator-5.044` |
| **RISC-V Toolchain** | xPack riscv-none-elf-gcc 14.2.0 | `/opt/riscv` |
| **GTKWave** | 3.3.116 | System package |
| **Ubuntu** | 24.04 (ARM64 on UTM) | Running on Mac |

**Environment Variables:**
```bash
CV_SW_TOOLCHAIN="/opt/riscv"
CV_SW_PREFIX="riscv-none-elf-"
CV_SW_MARCH="rv32imc_zicsr"
```

---

## Significant Friction Point Encountered

**Problem:** GCC 12+ requires explicit `zicsr` extension for CSR instructions.

When compiling the BSP, the build failed with:
```
Error: unrecognized opcode `csrw mtvec,a0', extension `zicsr' required
```

**Root Cause:** Newer RISC-V GCC versions follow the updated ISA specification where `zicsr` (CSR instructions) and `zifencei` are no longer implicitly included in base ISAs.

**Resolution:** Added `_zicsr` suffix to the architecture flag:
```bash
export CV_SW_MARCH="rv32imc_zicsr"
```

---

## Why RTL Code is Not in core-v-verif Repository

The RTL code for CV32E40P is **not** part of the core-v-verif repository because CORE-V-VERIF serves as a **unified verification environment** for multiple CORE-V processor cores (CV32E40P, CV32E40X, CV32E40S, CVA6, etc.). Each core has its own independent development lifecycle, release schedule, and maintainers.

**How the testbench gains access to RTL:**
1. The Makefile reads `cv32e40p/sim/ExternalRepos.mk` which specifies the RTL repository URL, branch, and commit hash
2. When `make` is invoked, it automatically clones the RTL to `$CORE_V_VERIF/core-v-cores/cv32e40p/rtl/`
3. This allows pinning specific RTL versions for regression testing

---

## Validation Screenshots

### Hello World Test - PASSED

<img width="2104" height="742" alt="image" src="https://github.com/user-attachments/assets/6cbc1bed-00aa-4aa8-ae13-3a3b04e3aa1f" />


### Instruction Fetch Interface Waveform

<img width="3150" height="1616" alt="image" src="https://github.com/user-attachments/assets/3c50e290-f579-4447-afb9-58a2c6b2559b" />


Captured signals in GTKWave:
- `instr_req_o` - Core requesting instructions
- `instr_gnt_i` - Memory granting requests  
- `instr_addr_o` - Instruction addresses (0x80000xxx range)
- `instr_rdata_i` - 32-bit instruction data returned
- `instr_rvalid_i` - Valid signal indicating data ready

---

## Conclusion

Successfully executed the CORE-V-VERIF Quick Start workflow for CV32E40P using Verilator and open-source tools.

---

## Acknowledgment

This challenge was completed with assistance from an AI coding assistant (Claude/Antigravity) for debugging issues, understanding documentation, and drafting this memo. All concepts were learned and understood through the process.
