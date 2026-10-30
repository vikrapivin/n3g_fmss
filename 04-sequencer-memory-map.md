# Chapter 4 — The register window as the sequencer sees it

Summary: The sequencer addresses the same register window as the CPU (segments and mirrors: [chapter 3, section 3.2](03-cpu-memory-map.md#32-memory-map-the-register-window)), and in addition reaches system memory through `ldi` and `sti` and fetches its own program from memory. This chapter collects what is specific to the sequencer's side: field widths, address decode, the register file, parameter RAM and system memory.

## 4.1 Window offsets and target widths

Field widths (device):

- The register field `R` is decoded in all 8 bits: register numbers 24 and 26 are not treated as 8 and 10 ([chapter 4, section 4.3](#43-register-file-r0r7-the-pc-and-register-numbers-above-7)).
- Window offsets are 12 bits. `wr`, `st` and `ld` with `arg` = `0x1d70`, `0x8d70` or `0xfd70` all reached the word at `0xd70`, so bits 15:12 of `arg` are ignored. `pld` and `pst` wrap their register value at 12 bits in the same way.
- Branch targets and values written to r8 use more bits than window offsets.
  - A branch target is a byte offset from the program start. A target with bit 12, 16 or 24 set did not reach the label it was built from: the bit is not cut off, so the offset points beyond the program. With bit 12 or 16 the run then ended without reaching the label; with bit 24 the sequencer fetched from an address that held no valid instruction and timed out ([D-fwidth-27](14-evidence.md#142100-fwidth-run-27)).
  - A target with bit 31 set reached the label, so bit 31 is ignored.
- `jr` with any of bits 12, 16, 24 and 31 set in the register did not jump at all; execution continued with the next instruction.

## 4.2 Segments and mirrors, sequencer view

The segment table is in [chapter 3, section 3.2](03-cpu-memory-map.md#32-memory-map-the-register-window). Device result (row "address decode (sequencer view)" of [chapter 16, section 16.2](16-device-findings.md#162-settled-on-a-device)), found by reading every word `0x000..0xffc` with `pld`:

- Offset bit 9 is not decoded: `0x200..0x3fc` repeats `0x000..0x1fc`, `0x600` repeats `0x400`, `0xa00` repeats `0x800` and `0xb00` repeats `0x900`.
- `0x400..0x5fc` is a second NAND controller (FMC) register set. The boot ROM also addresses its channels at a spacing of `0x400`.
- `0x100..0x1fc` is a separate block. The same block is visible again at `0x300`, `0x500` and `0x700`. The words `0x1bc..0x1fc` read all ones, except `0x1cc` = `0x8060` and `0x1d0` = `0x3010d0`.
- The word `0x9c0` reads 1. The words `0xe00..0xffc` read 0.

## 4.3 Register file r0..r7, the PC and register numbers above 7

Register file and PC (rows of the control-block table; the primary description is in [chapter 8, section 8.1](08-sequencer-control-registers.md#81-sequencer-control-block)):

| offset | name | meaning |
|---|---|---|
| `0xc18..0xc34` | r0..r7 | the registers; r6 is `SEQ_RESULT`. A value that the CPU writes before the start becomes the register's starting value; a start leaves the registers unchanged; the sequencer reset clears them (device) |
| `0xc38` | `FMSS_PC` | byte offset of the current instruction while a program runs or waits; 0 after the end. It is register r8 (`0xc18 + 4 * 8`). The program reads it as r8 or with `ld`. Writing it (`movi r8`, `wr` or `st` to `[0xc38]`) is a jump. The CPU can write it only while the program is stopped at `pause` ([chapter 6, opcode `12`](06-opcode-reference.md#12-pause)); otherwise CPU writes are ignored (device) |

Register numbers (device, every number from 8 to 255 in every role, [D-regsweep-35](14-evidence.md#142149-regsweep-run-35); further samples [D-regnum-15](14-evidence.md#14259-regnum-run-15)): the register fields hold 8 bits and are decoded in full. Before testing each role, the program loaded distinct values into r0..r7 and into the writable control words `C10`, `C48`, `C54` and `C3C`. This made it possible to tell apart three outcomes: the operand reads r0, the operand reads 0, and the operand reads the control word at `0xc18 + 4 × N`.

| role | 8 | 10 | 9 and 11..255 |
|---|---|---|---|
| destination (`movi`, `mov`, `add`) | the PC: a write is a jump | written into all of r0..r7 | no register and no window word changes |
| argument-field source (`mov`, ALU, immediate form) | the offset of the instruction itself | the destination register's own value | r0 |
| argument-field source (ALU, register form) | the offset of the instruction itself | r0 | r0 |
| register-field source of `st` | see the first bullet below | see the first bullet below | see the first bullet below |
| tested register of `bnz` / `bz` | never branches | never branches | never branches (also `bz` with r0 = 0) |
| target register of `jr` | jumps to itself (the program never ends) | r0 | r0 |
| address register of `ldi` | reads a code-like word (`e90d1c00`) | r0 | r0 |
| address register of `pld` | reads 0 | r0 | r0 |

- `st [REG], rN` with N from 8 to 255 does not store the value of a fixed register. It stores the last value that passed through the register file: the value the instruction before it wrote into a register, or, if that instruction wrote no register, the register value it read (device, [D-stprev-36](14-evidence.md#142156-stprev-run-36); also device runs 15 and 35). With N = 9, 14 and 200 alike, after:
  - `movi r0` / `r3` / `r7`, `add r2, r2, #1`, `mov r4, r1`, `ld r5, [...]`, `pld r6, [r0]`, `ldi r1, [r0]`: the value written to that register;
  - `st [0xd7c], r5`: r5's value (the register `st` read);
  - `wr [0xd7c], #...` and `nop`: r0's value (instructions whose register field is 0);
  - `wait 11`: the value of the instruction before the `wait` (`wait` leaves it unchanged).
- None of the numbers reads or writes a control word: the values preloaded into `C10`, `C48`, `C54` and `C3C` never appeared.
- r8 is the PC: `movi r8, #label` and `mov r8, r1` jump to the label, `mov r1, r8` returns the offset of that instruction ([D-r8-16](14-evidence.md#14273-r8-run-16), [D-r8-17](14-evidence.md#14279-r8-run-17)).

No original program uses any register number above 7.

## 4.4 Parameter RAM

The parameter RAM consists of plain read/write words shared by the CPU and the program. It ends at `0xd7c`; `0xd80..0xdfc` read 0 and drop writes, from the CPU and from the program (device).

- The original CPU driver writes only the words of [chapter 12, section 12.1](12-cpu-interface.md#121-parameter-words-per-program) for the job being started, and rewrites every word it needs before each run.
- The words `0xd2c..0xd3c` and `0xd50..0xd7c` are never written by the original CPU driver. The original programs use them as scratch words, and the value they hold when a program starts is not defined.
- The word `0xd00` is read by the programs of release 1.1.3 that send the command `78h` (WRITE2P_78 and WRITE_78) as a row address ([RE-28](14-evidence.md#14328-re-28-releases-101-to-113)); what writes it is open ([chapter 17](17-open-questions.md#176-original-programs-and-the-cpu-interface)).
- The original READ program uses `pld` and `pst` only on the words `0xd60..0xd7c`, a queue of chip masks ([RE-6](14-evidence.md#1436-re-6-opcodes-18-and-19)).
- The original programs keep working state in registers or in scratch words. They advance the pointer words in place, writing the updated value back to the same word.

## 4.5 System memory and the program

**2. System memory** (the CPU's bus addresses): `ldi` and `sti` take a full 32-bit bus address from a register and access a little-endian 32-bit word. An address that is not a multiple of 4 is rounded down to the word. The DMA of the NAND controller ([chapter 9, section 9.2](09-nand-controller.md#92-fmctrl1-0x004-nand-controller-operation-trigger-not-a-sequencer-control)) also moves data between its buffers and bus addresses (`FMC_DMA_ADDR`). These are bus (physical) addresses, as the original CPU driver passes them (inferred: every address used in the tests is the same in the CPU's view, and a load from address 0 returned code-like words).

**3. The program**: fetched from bus address `FMSS_PROG` (`0xc04`) onwards, 8 bytes per instruction ([chapter 5, section 5.1](05-instruction-encoding.md#51-encoding)). The PC and the branch and jump targets are byte offsets from `FMSS_PROG`, not bus addresses. A program can sit in SDRAM (217 ns per instruction) or in internal SRAM (138 ns) (device, [chapter 16, section 16.2](16-device-findings.md#162-settled-on-a-device)).

## 4.6 What the sequencer sees differently from the CPU

The comparison of CPU and sequencer access ([chapter 3, section 3.2](03-cpu-memory-map.md#32-memory-map-the-register-window)) shows these differences:

- The sequencer can write the PC (a jump); the CPU can write it only while a program is stopped at `pause`.
- A value written by the CPU to `C3C`, `C48` or `C54` before a start is not what the program reads, because the start clears them. A value that the program writes to `C48` or `C54` stays visible to the CPU after the run. `C3C` is cleared at the end of the run.
- After the CPU wrote the words `0x3a8..0x3b4`, the sequencer read back the written values, but the CPU itself read 0 (unexplained).
- The status words (`0x050`, `0x250`, `0x450`, `0x650`, `FMC_28` and their copies) changed by themselves during the test, so the write masks measured on them carry no information.

References: [chapter 14](14-evidence.md) (outcomes), [chapter 13](13-experiments.md) (what was done and why), [chapter 15](15-evidence-interpretation.md) (what the outcomes mean).

Related chapters: [chapter 3](03-cpu-memory-map.md), [chapter 5](05-instruction-encoding.md), [chapter 12](12-cpu-interface.md).
