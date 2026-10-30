# Chapter 13 — Experiments: what was done and why

Summary: This chapter lists every experiment run on the device, in the order of its first run, with the procedure and the question it was run to answer. It records no outcomes and no interpretation; outcomes are in [chapter 14](14-evidence.md), their reading is in [chapter 15](15-evidence-interpretation.md).

## 13.1 How an experiment runs on the device

An experiment is one or more small sequencer programs (8-byte instructions, the format of [chapter 5](05-instruction-encoding.md)) that the CPU loads into memory and starts. For each run of a program the CPU first presets what the experiment needs (registers r0..r7, parameter words, other register words), writes the program address to `FMSS_PROG`, clears `CS_IRQ` and writes a start word to `FMSS_CTRL` (`0xfff5` unless the experiment sets another). During the run the CPU can write further words at set times (for example a second start word 1 ms or 2 ms after the start) and samples chosen registers (the PC, r0, `FMSS_CTRL`, `C08`, `C14` and others) at intervals. After the run it records the time to END, to a fault or to the time bound it gave the run, the register file, the PC, the control, NAND controller (FMC), ECC and parameter windows before and after, and the samples. Chip enables are deselected during the experiments unless the experiment says otherwise. Experiments that need a clean state write `C58`, `C4C` and `C5C` as 0 before each start.

The experiments ran in 38 device runs; a device run is one batch of experiments executed in sequence on one unit. The outcome of an experiment is the summary text its code printed; chapter 14 reproduces it word for word.

### 13.1.1 Soft reset

A soft reset, as used by the `restore` experiment and by experiments that start from a defined state, is: every writable NAND controller, ECC and sequencer word is written to its power-on value, the status words are acknowledged, the `0x100` block is rewritten by the sequencer, and parameter RAM is zeroed. `restore` then compares every word with the power-on values. A power cycle is the other way to reach the power-on state; `state` records the state as it is after a power cycle.

### 13.1.2 NAND scratch blocks

Experiments that write or erase NAND do so only in scratch blocks starting at block 3000 (128 pages per block), on CE0 and, in four experiments, on CE1, and in `startregion` also blocks above the user area. Blocks used (erased, programmed or, for `readev`, read): 3000 (`ev11`, `ev5`, `evsweep`, `op09b`, `rawwrite`, `writeseq`, `eraseseq`, `readev`), 3001 and 3002 (`writeseq`, `eraseseq`), 3004 to 3006 (`op09nand`), 3007 (`ev5ready`), 3008 to 3013 (`pageorder`), 3014 and 3015 (`writeseq2`), 3016 on CE0 and CE1 (`ev5two`). `rawpage` and `readev` only read. All other experiments do not access the NAND array; the experiments `chan1` and `chan1pins` read the chip ID. In device run 35: block 3000 (`evsweep2`, `irq044`), 3050 (`fmcbits`), 3060 and 3061 of CE0 and 3060 of CE1 (`progtime`). In device runs 36 and 37: blocks 3002 and 4000 of CE0 and block 3900 of CE1 (`startregion`), 3051 (`dmactl`), 3070 (`rschip`), 3071 (`evsweep3`, a program cancelled before its confirm).

### 13.1.3 Loading and reading the buffers

The buffers have no address ([chapter 1, section 1.5](01-overview.md#15-named-parts-of-the-data-path)), so every experiment reached them through the transfers of the NAND controller and the ECC engine. Known data was placed in a **scratch area** of system memory: the CPU wrote it there before the start, and read the area back after the run.

| step | program writes | effect |
|---|---|---|
| data into data buffer 0 (1) | `FMC_CTRL0` = `0x1043801` (bit 24, DMA enable), `FMC_DNUM` = `0x1ff`, `FMC_DATA_SEL` = `0x1` (`0x2`), `FMC_DMA_ADDR` = bus address in the scratch area, `FMCTRL1` = `0x2e0`, `wait` event 2, `FMC_STATUS` = `0x100000` | 512 bytes from system memory into the data buffer |
| data buffer 0 (1) back to memory | `FMC_DATA_SEL` = `0x100` (`0x200`), `FMC_DMA_ADDR`, `FMCTRL1` = `0x100`, `wait` event 2 | 512 bytes from the data buffer into the scratch area |
| chunks into parity buffer 0 (1) | as the first row with `FMC_DATA_SEL` = `0x10` (`0x20`) | the DMA moves 512 bytes and wraps over the 64-byte buffer: slot k receives source bytes 448 + 16k to 463 + 16k, so host-built chunks were placed in the last 64 bytes of the 512-byte source |
| parity buffer 0 (1) back to memory | `FMC_DATA_SEL` = `0x1000` (`0x2000`), `FMCTRL1` = `0x100`, `wait` event 2 | the buffer from slot 0 onwards into the scratch area |
| parity computed into slot k | data buffer filled as above, then `ECC_DONE` = `0x1ff`, `ECC_MODE`, `ECC_KICK` = `0x1102` &#124; k << 16, `wait` event 4 | the engine writes the parity into slot k of parity buffer 0 (and a copy, reversed, into the ECC area `0x900..0x913`, read with `ld` or by the CPU) |
| decode | stored data in a data buffer and the chunk in slot k (by DMA as above, or from the chip), then `ECC_KICK` = `0x1101` &#124; k << 16, `wait` event 4, `ld` of `ECC_RESULT` | the data buffer corrected in place; the result word read by the program |
| from or to the chip | chip reads `0x32` (chunk into a slot, `FMC_24` = 16k) and `0xe2` (data); chip writes `0x34` (slot, `FMC_28` = 16k) and `0xe4` (data) | as in the original programs ([chapter 11, section 11.2](11-nand-operations.md#112-ecc-and-data-path-shared-by-read-and-program)) |

Encode and decode of one sector were run in the same program, because the set-up word at the start of each program clears the buffers (inferred, [chapter 10, section 10.2.1](10-ecc-engine.md#1021-running-one-operation)). No experiment read or wrote a buffer through the FIFO port `FMC_FIFO` (`0x080`), the only window word known to reach a data buffer ([chapter 1, section 1.5](01-overview.md#15-named-parts-of-the-data-path)).

### 13.1.4 Steps around each program run

Each program of an experiment runs inside a fixed sequence of CPU steps. An experiment can skip single steps; its description then names them ("no preflight", "no clean-up", "no reset").

Before the start, in this order:

1. **Reset:** `FMSS_CTRL` = `0x30000`, then 1 ms.
2. **Clean-up:** the CPU writes 0 to `C58`, `C4C` and `C5C`.
3. `CS_IRQ` is cleared.
4. **+0x400 = 0:** the CPU writes 0 to `FMC_CTRL0` of the second register set (`0x400`).
5. **Preflight:** the CPU starts a program that consists of `end` only, with `0xfff5`, and waits up to 10 ms for it to end. Its result is recorded ("preflight ok"); a failed preflight means the sequencer does not run at all.
6. `FMC_CTRL0` (`0x000`) = 0, unless the experiment keeps it.
7. The experiment's own register writes (registers r0..r7, parameter words, other words such as `C58` or `C4C`).
8. **Before-snapshot:** the CPU reads the windows `0x000`, `0x400`, `0x800`, `0xc00` and `0xd00` (64 words each; `0x080` is not read). If the experiment asks for it, the CPU also reads all 1024 words of the window, `0x080` and its mirrors included.
9. **Start:** `FMSS_PROG` = the program address, `CS_IRQ` cleared, `FMSS_CTRL` = the start word.

After the end (or the time bound): the after-snapshot (and, if asked for, the full read of the window), then the **park**: `FMSS_CTRL` = `0x30000`, 1 ms, the clean-up of step 2, the `end`-only program run once, `CS_IRQ` cleared, `FMC_CTRL0` = 0. In the first device runs the park had no clean-up step; experiments that recreate those steps say so in their description.

## 13.2 Device runs

A device run is one session on the unit: emCORE is started on the iPod, and one command runs a list of experiments in order. Device runs are numbered in the order they were made. An evidence entry names the experiment and the device run it comes from (for example `D-done5b-37`: experiment `done5b`, device run 37); when an experiment ran in more than one device run, the run number tells the entries apart. Device run 37 repeated the experiments that device run 36 could not complete after the unit stopped answering.

| device run | experiments run in it |
|---|---|
| 1 | `regmap`, `clock`, `alu`, `memory`, `indirect`, `control`, `wait`, `unknown` |
| 2 | `pram`, `alu`, `indirect`, `control`, `c58`, `unknown` |
| 3 | `pram`, `alu`, `indirect`, `control`, `c58`, `unknown` |
| 4 | `resume`, `wait`, `op15` |
| 5 | `op12`, `c58ill`, `pending`, `c14` |
| 6 | `regmap`, `startseq`, `pending`, `op12ack`, `op10` |
| 7 | `regmap`, `startseq2` |
| 8 | `startseq3`, `oneshot` |
| 9 | `c50`, `holdout` |
| 10 | `holdout` |
| 11 | `op12wait`, `ev11`, `ev5`, `opsweep`, `optime`, `ctrlbits`, `fmcmap`, `evsweep` |
| 12 | `rwmask` |
| 13 | `c50x` |
| 14 | `evsweep`, `op0f`, `ctrlhold`, `evclear`, `timeregs` |
| 15 | `ev5rule`, `c08`, `chan1`, `readev`, `regnum`, `branch`, `waitf`, `opcost`, `op12rel`, `pause`, `fmc28`, `blk100`, `irqline` |
| 16 | `restore`, `readev`, `irqline`, `irqen`, `ev5t`, `r8` |
| 17 | `state`, `readev`, `irqline`, `irqen`, `ev5t`, `r8` |
| 18 | `state`, `ev8`, `evcons` |
| 19 | `restore` |
| 20 | `dmaloop` |
| 21 | `restore`, `readev`, `dmaloop` |
| 22 | `op0916` |
| 23 | `op16` |
| 24 | `opstate`, `op16b`, `op09` |
| 25 | `op09b` |
| 26 | `op09c` |
| 27 | `chan1pins`, `pcwrite`, `op12cont`, `ctrlfx`, `op09d`, `fwidth`, `irqsrc`, `eccpar`, `bufsize`, `eccstr`, `access` |
| 28 | `fields0`, `tmofield`, `eccstr2`, `ev5en` |
| 29 | `tmobits`, `eccmode`, `eccchunk`, `rawpage`, `rawwrite`, `writeseq`, `op09nand`, `eraseseq` |
| 30 | `eccchunk2`, `eccmode2`, `tmobits2`, `ev5ready`, `pageorder`, `op09w33` |
| 31 | `ev5arm`, `writeseq2`, `eccmode3` |
| 32 | `ev5two`, `b15ecc` |
| 33 | `eccmode4` |
| 34 | `finale` |
| 35 | `startdiff`, `startbits`, `rsdecode`, `vic20`, `fmc28b`, `c58b1`, `evsweep2`, `c50fault`, `ctrlwords`, `c08self`, `latchrep`, `irq044`, `fmcbits`, `fmcstat`, `eccregs`, `seqrestart`, `done5`, `progtime`, `regsweep` |
| 36 | `seqfirst`, `w2state`, `c50b8`, `startregion`, `bufport`, `c58b2`, `stprev`, `tmwait2`, `irqop` |
| 37 | `done5b`, `rsresult`, `rschip`, `miscorr`, `fullpage`, `dmactl`, `fmcstat2`, `ch1buf`, `evsweep3`, `blk100b`, `straps`, `irqop` |
| 38 | `w2vic` |

## 13.3 Experiments

### 13.3.1 regmap

Device runs: 1, 6, 7.

Procedure and question: The experiment checks whether the registers r0..r7 are the CPU-visible words at 0xc18 + 4n, and whether SEQ_RESULT (0xc30) is r6.

### 13.3.2 clock

Device run: 1.

Procedure and question: The experiment measures how fast the sequencer executes instructions, using a loop of two instructions run at two different lengths.

### 13.3.3 alu

Device runs: 1, 2, 3.

Procedure and question: The experiment tests the edge cases of the ALU instructions: the kind of shift and the range of shift counts, wrap-around, the form selected by an immediate (imm) of 0, and the mask of ld.

### 13.3.4 memory

Device run: 1.

Procedure and question: The experiment tests ldi and sti on system memory (the scratch area): word access, byte order, and an unaligned address.

### 13.3.5 indirect

Device runs: 1, 2, 3.

Procedure and question: The experiment checks which offsets pld and pst reach: the register file, parameter RAM, and offsets beyond 0xfff.

### 13.3.6 control

Device runs: 1, 2, 3.

Procedure and question: The experiment records what the sequencer's control words (0xc00..0xc5c) read back after a run, and checks whether the CPU can preload r0..r7.

### 13.3.7 wait

Device runs: 1, 4.

Procedure and question: The experiment runs `wait` on event N while nothing is pending. It finds which events are already set, which time out, and how long the timeout is.

### 13.3.8 unknown

Device runs: 1, 2, 3.

Procedure and question: The experiment runs opcodes without a known meaning and records what they change. The registers are seeded with values beforehand and the fault vector is set.

### 13.3.9 pram

Device runs: 2, 3.

Procedure and question: The experiment checks which words of parameter RAM (0xd00..0xdfc) keep a value written by the CPU, and which words the sequencer can read and write.

### 13.3.10 c58

Device runs: 2, 3.

Procedure and question: The experiment checks what each value of FMSS_C58 does to a `wait` that times out: the time taken, the fault flag, and whether the program continues at the fault vector or at the next instruction.

### 13.3.11 resume

Device run: 4.

Procedure and question: The experiment checks where a start begins execution. The CPU sets FMSS_C58 and FMSS_C5C before the start, and a marker is placed at the fault vector.

### 13.3.12 op15

Device run: 4.

Procedure and question: Opcode 0x15 had changed FMSS_C14: to 0 with imm 0, and to 0xff with imm 0xd88. The experiment checks what value the opcode writes and from which instruction field the value comes.

### 13.3.13 op12

Device run: 5.

Procedure and question: Opcode 0x12 stops the program without an event timeout and latches itself in 0xc40/0xc44. The experiment checks whether it is a conditional wait on a register. The CPU presets 0xd60 to 0x00001234. In each case the instruction either passes (the program continues) or stops (the time bound of the test expires).

### 13.3.14 c58ill

Device run: 5.

Procedure and question: The experiment checks whether the bits of FMSS_C58 also govern the fault raised by an illegal opcode (0x09), as they govern the timeout of a `wait`.

### 13.3.15 pending

Device runs: 5, 6.

The experiment changed between runs. Procedure and question per version:

- device run 5: The state found in the second device run. Does a program that leaves C58 bit 0 set (after a fault, or not) make the next start begin at the fault vector? The programs run back to back with no cleanup in between, so the state is kept.
- device run 6: The state found in the second device run. A program leaves C58 = 1 (after a fault, or not); where do the next starts begin? This version recreates the steps used then ([chapter 13, section 13.1.4](#1314-steps-around-each-program-run)): a park without the clean-up after the run, and reset, the `0x400` write and the preflight before it.

### 13.3.16 c14

Device run: 5.

Procedure and question: The experiment checks whether a write by the sequencer sets FMSS_C14, and whether opcode 0x15 inverts it (applied twice, the value is unchanged).

### 13.3.17 startseq

Device run: 6.

Procedure and question: After a sequencer reset, a START is accepted but nothing executes unless a further step is taken. The experiment checks which step lets a program run.

### 13.3.18 op12ack

Device run: 6.

Procedure and question: Opcode 0x12 stops the program with CS_IRQ bit 3 set. The experiment checks which CPU write, made 2 ms after the start, lets the program continue.

### 13.3.19 op10

Device run: 6.

Procedure and question: A run of opcode 0x10 had ended with CS_IRQ = 3. The experiment checks whether the opcode sets CS_IRQ bit 1 and lets the program carry on.

### 13.3.20 startseq2

Device run: 7.

Procedure and question: A START right after a reset runs the program, but with register accesses in between it does not. The experiment looks for the access responsible. Each case is a reset with no preflight and no cleanup; an END flag in CS_IRQ means that the program ran.

### 13.3.21 startseq3

Device run: 8.

Procedure and question: After a reset, a START fetches instruction 0 (into C40/C44) but does not execute it unless it is END. The experiment confirms this and looks for what releases the instruction, trying a CPU write of FMSS_CTRL 2 ms after the start. Each case is a reset with no preflight and nothing else in between.

### 13.3.22 oneshot

Device run: 8.

Procedure and question: FMSS_C58 bit 1 makes the next END run the fault vector once. The experiment checks whether the bit is also armed by a write from the sequencer, and whether a fault disarms it.

### 13.3.23 c50

Device run: 9.

Procedure and question: The original fault handlers test FMSS_C50 bit 0 while they run and then write 1 to it. The experiment checks whether a fault sets the bit, and whether the acknowledge or the END clears it.

### 13.3.24 holdout

Device runs: 9, 10.

Procedure and question: The experiment tests rules found in earlier experiments on combinations of conditions that none of those experiments used. For each combination the outcome that the rules predict was written down before the run, and the run shows whether the device agrees.

### 13.3.25 op12wait

Device run: 11.

Procedure and question: The experiment checks whether opcode 0x12 is a `wait` without a timeout, with the event number in the register field as in `wait`. It makes no NAND access.

### 13.3.26 ev11

Device run: 11.

Procedure and question: The experiment checks whether `wait` event 11 is the chips' ready line. It runs `wait` event 11 right after starting a block erase on CE0 (destructive, in a scratch block).

NAND access: Erases block 3000 of CE0 (the erase is started and left running while the event is tested).

### 13.3.27 ev5

Device run: 11.

Procedure and question: The experiment checks whether `wait` event 5 depends on a refreshed status byte or on the chip being ready. It takes a status poll (70h, FMCTRL1 = 0xca) during an erase and when the chip is idle.

NAND access: Erases block 3000 of CE0 (status polls during the erase).

### 13.3.28 opsweep

Device run: 11.

Procedure and question: The experiment runs every opcode 0x00..0xff without a known meaning, in three operand forms. The registers are seeded with harmless pointers, C58 is 0 (a fault sets its flag and the program continues) and the timeout is 20 ms. It records which opcodes end the program, fault, block, stall or change something.

### 13.3.29 optime

Device run: 11.

Procedure and question: The experiment checks whether opcodes with no visible effect take time. Each is executed 1000 times in a loop and compared with `movi`, in several operand forms. The same battery includes each opcode that opsweep found to continue fast with no change.

### 13.3.30 ctrlbits

Device run: 11.

Procedure and question: The driver writes 0xfff5 as the start word. The experiment sets each other bit alone (together with bit 0) on a loop of 1000 iterations, and checks whether any bit changes the speed, single-stepping, interrupts or the start itself.

### 13.3.31 fmcmap

Device run: 11.

Procedure and question: The experiment reads the whole 4 KB register window as the sequencer sees it (pld of every word into RAM). This includes registers outside the windows that the CPU snapshot covers.

### 13.3.32 evsweep

Device runs: 11, 14.

The experiment changed between runs. Procedure and question per version:

- device run 11: All 16 `wait` events are tried around NAND activity (destructive: erases scratch block 3000 of CE0). The experiment records which pass at once, which pass later and which time out, and reads FMC_STATUS just before and after each wait.
- device run 14: All 16 `wait` events are tried around NAND activity (destructive: erases scratch block 3000 of CE0). The experiment records which pass at once, which pass later and which time out, and reads FMC_STATUS after each wait. A sequencer read of FMC_STATUS before the wait had consumed events on the device, so this version does not read it first.

NAND access: Erases block 3000 of CE0.

### 13.3.33 rwmask

Device run: 12.

Procedure and question: For every word of the 0x000, 0x800, 0xc00 and 0x400 windows (registers that trigger an action are left out), the experiment writes all ones and then zero from the CPU before the start and reads the word back. This shows which bits are writable, read-only or write-1-to-clear.

### 13.3.34 c50x

Device run: 13.

Procedure and question: The experiment checks which fault sets FMSS_C50 bit 0. It runs on its own and last, because the bus cases may hang the SoC.

### 13.3.35 op0f

Device run: 14.

Procedure and question: A device run had shown that opcode 0x0f acts as an indirect jump (the PC ended at the seeded register's value + 8). The experiment checks which register field names the target, whether the target is a byte offset from the program start, and whether a return address is kept anywhere.

### 13.3.36 ctrlhold

Device run: 14.

Procedure and question: FMSS_CTRL bits 1, 3 and 16 kept a program from running (device). The experiment checks whether they pause it (a later write without them releases it), whether they single-step it (each later write runs one instruction), and whether bit 17 clears r0..r7.

### 13.3.37 evclear

Device run: 14.

Procedure and question: The experiment checks whether a sequencer read of FMC_STATUS (or FMC_NSTAT) consumes a pending event before a `wait` sees it.

### 13.3.38 timeregs

Device run: 14.

Procedure and question: The experiment checks whether any of the writable sequencer words with no known use (C10, C3C, C48, C54) changes the `wait` timeout.

### 13.3.39 ev5rule

Device run: 15.

Procedure and question: The experiment checks what arms `wait` event 5. A status latch (FMCTRL1 = 0xca) after 70h had passed `wait` event 5 when the command-done flag (FMC_STATUS bit 1) had been cleared first, never when the flag was still set, and never for a second latch.

### 13.3.40 c08

Device run: 15.

Procedure and question: The experiment checks whether FMSS C08 is a state code. The CPU reads it while a program runs, waits, stalls, blocks, is paused or is held.

### 13.3.41 chan1

Device run: 15.

Procedure and question: The experiment checks whether the register set at +0x400 is a second NAND channel. It sends READ ID through that set for each chip enable and compares the result with the same READ ID sent through the first set. Waits that time out continue (C58 = 0).

NAND access: Reads the chip ID through the second register set and through the first; no array access.

### 13.3.42 readev

Device runs: 15, 16, 17, 21.

Procedure and question: The experiment tries all 16 `wait` events at each step of a page read (CE0, scratch block 3000, page 0). The steps are 00h, address, 30h, status poll, parity and data transfer, ECC decode, and DMA. It records which event belongs to which step and which events pass together.

NAND access: Reads page 0 of block 3000 on CE0; no erase or program.

### 13.3.43 regnum

Device run: 15.

Procedure and question: The experiment checks which register is read and written when a register number from 8 to 255 is used in the register field and in the argument field.

### 13.3.44 branch

Device run: 15.

Procedure and question: The experiment tests the edge cases of bz and bnz: a misaligned target, the argument field, and a sign bit.

### 13.3.45 waitf

Device run: 15.

Procedure and question: The experiment checks the other fields of `wait`. The 4G/5G engine waits on a register under a mask. The experiment checks whether the 3G uses arg and imm, and whether event numbers above 15 act as their low bits.

### 13.3.46 opcost

Device run: 15.

Procedure and question: The experiment measures the time per instruction type, with each type executed 100 x 10 times in a loop and compared against movi. The program runs from SDRAM and from internal SRAM.

### 13.3.47 op12rel

Device run: 15.

Procedure and question: Opcode 0x12 stops the program. The experiment checks whether any CPU write other than those tried so far releases it. The write is made 2 ms after the start.

### 13.3.48 pause

Device run: 15.

Procedure and question: The experiment checks whether start-word bit 1, written while a program runs, pauses it midway, and whether a later start without that bit resumes it.

### 13.3.49 fmc28

Device run: 15.

Procedure and question: The experiment checks what FMC_28 counts, by running 64 instructions of one kind and comparing the result with an empty program.

### 13.3.50 blk100

Device run: 15.

Procedure and question: The experiment reads the block at 0x100..0x1fc (repeated at 0x300, 0x500 and 0x700). It then writes all ones and zero from the sequencer and reads back what is left, at 0x100 and at the 0x300 copy. Finally it writes the old value back.

### 13.3.51 irqline

Device runs: 15, 16, 17.

Procedure and question: The experiment checks which sequencer states raise the CPU interrupt line (VIC1 raw bit 22), and whether start-word bit 6 is its enable.

### 13.3.52 restore

Device runs: 16, 19, 21.

The experiment changed between runs. Procedure and question per version:

- device run 16: A soft reset is tried instead of a power cycle. The state is recorded before; then the writable NAND controller, ECC and sequencer words are written back to the values they had before any experiment touched them, and pending ECC_DONE bits are acknowledged; then the state is recorded after, for comparison with `state` taken right after a power cycle.
- device runs 19, 21: A soft reset is tried instead of a power cycle. Every writable NAND controller, ECC and sequencer word is written to its power-on value, status words are acknowledged, the 0x100 block is rewritten by the sequencer, and parameter RAM is zeroed. Then every word is compared with the power-on values. What differs afterwards can be cleared only by a power cycle or by a reset not found yet.

### 13.3.53 irqen

Device runs: 16, 17.

Procedure and question: The experiment uses NAND controller word 0x044 and ECC word 0x844 as interrupt enables. One enable bit is set at a time, and then a command byte (FMC_STATUS bit 1) or an ECC decode (ECC_DONE) is triggered. It records which CS_IRQ bit is set and whether the CPU interrupt line follows.

### 13.3.54 ev5t

Device runs: 16, 17.

Procedure and question: In evsweep, `wait` event 5 straight after 70h and a status latch timed out, but in ev5rule it passed when a `wait` event 0 was placed in between. The experiment checks whether the cause is timing, the pending command flag, or an ordering rule.

### 13.3.55 r8

Device runs: 16, 17.

Procedure and question: Register numbers 8 and 10 had effects: `movi r8` ended the program early and `movi r10` wrote every register. The experiment checks whether r8 is the PC and what r10 is.

### 13.3.56 state

Device runs: 17, 18.

Procedure and question: The experiment records the state as it is. It runs first after a power cycle, so that it records the power-on values.

### 13.3.57 ev8

Device run: 18.

Procedure and question: `wait` event 8 passed at every step after a soft reset (the register set at +0x400 still held a READ ID sent through it) and never after a power cycle. The experiment checks whether events 6..15 belong to the second register set. It runs the READ ID of the second set step by step and tries every event after each step.

### 13.3.58 evcons

Device run: 18.

Procedure and question: The experiment checks whether a `wait` consumes its event. The sequence '70h, latch, `wait` event 15, `wait` event 5' had timed out once while every other order passed.

### 13.3.59 dmaloop

Device runs: 20, 21.

The experiment changed between runs. Procedure and question per version:

- device run 20: DMA without a chip, through each register set: from RAM into the controller's data buffer (FMCTRL1 = 0x2e0) and back to RAM (FMCTRL1 = 0x100, or 0x1e0 after FMC_DMA_CTRL = 7). Does the 0x400 set have its own buffers and DMA, which event says DMA done, and why did the first set's DMA in the page read never finish?
- device run 21: DMA without a chip, through each register set: from RAM into the controller's data buffer (FMCTRL1 = 0x2e0) and back to RAM (FMCTRL1 = 0x100, or 0x1e0 after FMC_DMA_CTRL = 7). Does the 0x400 set have its own buffers and DMA, which event says DMA done, and does a DMA need FMC_CTRL0 bit 24 (the read programs set it around their DMA; the probes did not)?

### 13.3.60 op0916

Device run: 22.

Procedure and question: Opcodes 09 and 16 had been classed as illegal. The experiment checks whether they are really illegal or are waits whose condition the opcode sweep never met. Each is run with operands that would satisfy a register-and-mask wait (the 4G/5G `wait` has that form; the 3G `wait` ignores arg and imm), with a register-field condition, or with a short delay. Opcode 07 (`wait`) and opcodes 1a and 20 (illegal) serve as references.

### 13.3.61 op16

Device run: 23.

Procedure and question: Opcode 16 timed out in the opcode sweep but passed at once in every form after DMA, ECC and page-read runs. The experiment checks which condition lets it pass. Each case is a soft reset, one action, and then opcode 16 (with opcode 09 as a reference).

### 13.3.62 opstate

Device run: 24.

Procedure and question: The experiment covers the opcodes before 1a that showed nothing or only timed out (08, 09, 10, 12, 15, 16, ff). It checks whether any of them reacts to pending completion flags of either register set. Each case is a soft reset, one state, and then the opcode. C4C points to a handler, so a jump to the fault vector shows up as 'vector'.

### 13.3.63 op16b

Device run: 24.

Procedure and question: Opcode 16 jumped to the fault vector at once while a completion flag was pending, and otherwise timed out. The experiment checks whether it goes to C4C whatever C4C holds, whether it waits for a flag that arrives later, whether C58 matters, and whether the flag is cleared.

### 13.3.64 op09

Device run: 24.

Procedure and question: The experiment checks whether opcode 09 means 'wait until the timeout' (a real instruction) or is just illegal. It is compared with opcodes 1a and 20 and with an unmet `wait`. It is started once with the timer `C14` at 0 (a `tmwait` instruction, opcode `15`, run before it leaves `C14` at 0) and once with `C14` at `0xff`. Timing and state while it waits are recorded, and cases with a pending flag named in its condition number or its mask are included.

### 13.3.65 op09b

Device run: 25.

Procedure and question: Opcode 09 (`wait2`) with condition number 5 continued at once while the status flag (`wait` event 5) was pending, and timed out for the other flags. The experiment checks whether it is a status wait with a mask test on FMC_NSTAT (the 4G/5G 'wait for ready' form), which event numbers it takes, whether it needs the flag, and whether it waits for a busy chip to become ready.

NAND access: Erases block 3000 of CE0 in some cases.

### 13.3.66 op09c

Device run: 26.

Procedure and question: Opcode 09 (`wait2`) passed with condition number 5 in every state tried and timed out with condition numbers 0..15 otherwise. From a clean state with nothing pending, the experiment checks which condition numbers 0..31 let it pass, with no chip selected and with CE0 / CE1 selected. As a reference, `wait` events 0..15 are tried in the same states.

### 13.3.67 chan1pins

Device run: 27.

Procedure and question: Channel 1 of the NAND controller (the 0x400 register set) has its own pins, which the test setup leaves unswitched (the boot ROM switches them for a channel-1 NAND boot). The experiment sends READ ID through channel 1 on CE0..7 with those pins switched for the run. As a control, READ ID through the first set is run with and without the pin switch.

NAND access: Reads the chip ID through the second register set and through the first; no array access.

### 13.3.68 pcwrite

Device run: 27.

Procedure and question: The CPU cannot write the PC (0xc38). The experiment checks whether the sequencer can, and whether such a write makes a jump (as a write to r8 does).

### 13.3.69 op12cont

Device run: 27.

Procedure and question: Opcode 12 stops the program with CS_IRQ bit 3. The 4G/5G driver treats CS_IRQ bit 3 as 'pause' and continues the program with three steps: FMSS_PC += 8, CS_IRQ = 8, FMSS_CTRL |= 1. The experiment checks whether this sequence, or a part of it, continues the program on the 3G.

### 13.3.70 ctrlfx

Device run: 27.

Procedure and question: The experiment checks what start-word bits 2 and 4..15 enable. Each start word is run on four behaviours that an enable could gate: a `wait` that times out, an illegal opcode, evjmp with a flag pending, and the CPU interrupt line with the NAND controller command-flag source enabled (0x044 bit 0).

### 13.3.71 op09d

Device run: 27.

Procedure and question: The experiment checks whether `wait2` (opcode 09) waits on the second register set's flags. Each flag of the second set (0x400) is made pending on its own (command, address, transfer, DMA, status latch). In that state `wait2` conditions 0..15 and `wait` events 0..15 are tried.

### 13.3.72 fwidth

Device run: 27.

Procedure and question: The experiment checks field widths: whether wr, st and ld use the upper bits of their 16-bit offset, and whether branches, jr and writes to r8 use the upper bits of their 32-bit target. (pld and pst wrap at 12 bits, which is already known.)

### 13.3.73 irqsrc

Device run: 27.

Procedure and question: The experiment tests interrupt sources. Each flag of the first register set (command, address, transfer, DMA, ECC, status) is made pending on its own, with no enables, with all NAND controller enables (0x044 = 0xffff), or with all ECC enables (0x844 = 0x2f). CS_IRQ, C50 and every VIC0 / VIC1 raw line are recorded. It then finds which single 0x044 bit routes each flag that reached CS_IRQ.

### 13.3.74 eccpar

Device run: 27.

Procedure and question: The experiment checks where the ECC parity bytes and the decoder's results become visible. It uses no flash. A known 512-byte sector is moved by DMA into data buffer 0 and encoded into parity buffer 0 slot 0, then decoded unchanged and with one and two bits flipped. The ECC engine window words 0x800..0x9fc are dumped after each step.

### 13.3.75 bufsize

Device run: 27.

Procedure and question: The experiment checks how big a data buffer is. N bytes of a known pattern are moved by DMA into data buffer 0 and N bytes are moved back out (FMC_DNUM = N - 1), for N = 256, 512, 513, 1024 and 2048. It checks whether a longer transfer wraps, stops or times out at 512.

### 13.3.76 eccstr

Device run: 27.

Procedure and question: The experiment measures the ECC correction strength. A known sector is encoded and then decoded with 1..12 bits flipped, spread over the sector, to find from how many flipped bits it becomes uncorrectable. It also checks whether ECC_MODE (always 0x180 in the original programs) changes that, with other modes tried at 8, 9 and 12 flipped bits.

### 13.3.77 access

Device run: 27.

Procedure and question: For every word 0x000..0xffc, the experiment checks whether a CPU write reads back to the CPU and to the sequencer, and whether a sequencer write reads back to the sequencer and to the CPU. It writes all ones and zero and notes which bits follow the write. The original values are put back after each group of 16 words, and a soft reset is done at the end.

### 13.3.78 fields0

Device run: 28.

Procedure and question: The experiment covers the fields that the original programs always leave 0: end's R / arg / imm, wr's R, movi's arg, and the imm of st, ldi, mov, sti, pld and pst. Each is tried with a mask-like value (0xff) and an offset-like value (4) to see whether anything changes.

### 13.3.79 tmofield

Device run: 28.

Procedure and question: The experiment checks the timeout length. Start-word bit 2 enables the timeout; the experiment checks which bits set its length, and in what unit.

### 13.3.80 eccstr2

Device run: 28.

Procedure and question: The experiment measures ECC strength and identifies the code, with encode and decode in one program (the set-up word at the start of a program clears the buffers). 0..12 bits are flipped before the decode. The parity of single-bit sectors is recorded so that the code can be identified by comparing it with computed parity afterwards.

### 13.3.81 ev5en

Device run: 28.

Procedure and question: In every run with an interrupt enable written, the status latch failed (FMC_STATUS bit 24 was set instead of bit 23, and there was no `wait` event 5). The experiment repeats the runs three times each with no enable, with single enable bits, and with the ECC enable. It also runs the case in which the program, instead of the CPU, writes the enable.

### 13.3.82 tmobits

Device run: 29.

Procedure and question: This is a second sweep of the timeout length. It checks which of start-word bits 7:4 enable or scale the count (bits 7:4 = 0 gave an immediate timeout, 0xf the long one), and how the length depends on bits 15:8 with bits 7:4 = 0xf.

### 13.3.83 eccmode

Device run: 29.

Procedure and question: The experiment tries ECC_MODE values other than 0x180 and checks whether the parity (in the slot and in the 0x900 area, with the whole ECC engine dumped) or the strength changes. For each mode a sector is encoded and then decoded in the same program with 0, 8, 9, 12 and 16 flipped data bits.

### 13.3.84 eccchunk

Device run: 29.

Procedure and question: (1) Can a parity slot be filled by DMA from memory? This is needed for raw writes and for chunk tests. (2) The meta encode: FMC_META0..2, FMC_METACFG = 0x3210 and FMC_METACTL = 1 are written, then an ECC encode is run. Where do the 12 metadata bytes land, and are they part of the parity? (3) Decodes of chunks built by the CPU (to the identified code) with errors in the metadata or parity bytes.

### 13.3.85 rawpage

Device run: 29.

Procedure and question: Pages written by the original firmware are read raw (chunk and data as stored, before the correction). They are checked against the identified code by a separate computation, and the engine's own decode of the same pages is also run.

NAND access: Reads six rows of CE0 (rows 0, 128, 12801, 128000, 256005, 448000) raw; no erase or program.

### 13.3.86 rawwrite

Device run: 29.

Procedure and question: DESTRUCTIVE (scratch block of CE0): erase, then program pages by every path and read them back raw. Case p0 is the engine's own path (meta encode and encode, parity slot taken from the parity buffer) and shows the on-flash layout and the metadata order. Case p1 uses chunks built by the CPU and moved by DMA into the parity buffer. Case p2 uses chunks built by the CPU and sent from a data buffer (no ECC engine at all: a raw write). Case p3 stores errors on the flash (8 or 9 data bits, a zero chunk, 2 metadata bits). Case p4 is left erased.

NAND access: Erases block 3000 of CE0, programs pages in it by four paths, leaves one page erased, and reads the pages back raw.

### 13.3.87 writeseq

Device run: 29.

Procedure and question: DESTRUCTIVE (scratch blocks): the write sequences that the original firmware uses, checked by raw read-back. (1) Four pages in one run with the overlapped transfer 0x2e4 (one sector is sent while the next is moved by DMA into the other data buffer), FMC_DMA_CTRL = 7 before the last page, the last sector sent with 0xe4, a meta encode per page, and a ready poll before the next 80h. (2) One run that programs a page on CE0 and a page on CE1 back to back and then polls both. (3) A two-plane pair (80h .. 11h, poll, 81h .. 10h) on blocks B and B+1. (4) A page programmed a second time without an erase: does the status register (FMC_NSTAT) report FAIL, and what is stored?

NAND access: Erases blocks 3000 and 3001 of CE0 and block 3000 of CE1; programs pages in them (including page 30 of block 3001 and a second program of one page without an erase) and reads them back raw.

### 13.3.88 op09nand

Device run: 29.

Procedure and question: DESTRUCTIVE (scratch blocks): does `wait2` (opcode 09) track write, erase or read activity of the chip? `wait2` condition N is tried for N = 0..31 while a block erase runs, right after it finished, while a page program runs, right after it finished, and during a page read's tR (the time the chip needs to load the page). For N = 32..255 it is tried idle and during an erase (not tried before). Per case the experiment records whether the wait passed or timed out, how long it waited, and the chip's status right after it (bit 6 = ready).

NAND access: Erases block 3006 of CE0 and programs and reads pages in blocks 3004 and 3005 of CE0.

### 13.3.89 eraseseq

Device run: 29.

Procedure and question: DESTRUCTIVE (scratch blocks): the erase. The experiment measures the time of an erase (of a block with written pages and of an already erased block) and the status register (FMC_NSTAT) right after D0h (busy?) and at ready. It checks whether written pages read back erased. It also runs a two-plane erase (60h B, 60h B+1, D0h, with B and B+1 being adjacent blocks), the erase on CE1, and an erase sent with 4 address cycles instead of 3 (FMC_ANUM 3: accepted or refused?).

NAND access: Reads pages 10, 20 and 30 of CE0 block 3000, page 30 of CE0 block 3001 and page 20 of CE1 block 3000 before and after erases of CE0 block 3000 (twice), a two-plane erase of CE0 blocks 3000 and 3001, an erase of CE1 block 3000, and an erase of CE0 block 3002 sent with FMC_ANUM = 3.

### 13.3.90 eccchunk2

Device run: 30.

Procedure and question: The experiment decodes chunks built by the CPU (to the identified code) and loaded into parity slot 0 by DMA (the chunk is in the last 64 bytes of the 512-byte DMA, which is what the 64-byte buffer keeps). The chunks carry errors in the metadata or parity bytes, errors mixed with data errors, and the metadata byte order is checked.

### 13.3.91 eccmode2

Device run: 30.

Procedure and question: ECC_MODE map, encode only (no decode afterwards, so that the ECC area shows what the encode left): 0x180 plus each single bit 9..24, and the values 0, 0x80, 0x100, 0xe00000 and 0x1ffffff. For the modes that differ, the parity of zeros and of single-bit sectors is recorded, to fit their code in the way the code of 0x180 was fitted.

### 13.3.92 tmobits2

Device run: 30.

Procedure and question: Timeout: where does the growth with start-word bits 7:4 stop? The values n = 5, 6 and 7 of bits 7:4 were untested; n of 8 and above = 2**11.

### 13.3.93 ev5ready

Device run: 30.

Procedure and question: DESTRUCTIVE (erase of a scratch block). An erase program's second status latch timed out every time, while the first (taken while the erase ran) waited for the end of the erase. The experiment checks whether `wait` event 5 (status latch done) means the chip becoming ready rather than 'status byte latched'. The cases are: latches when idle (after a soft reset and not), twice in one program, after a chip reset (FFh), and after an erase. FMC_STATUS and FMC_NSTAT are read before each wait. The original firmware's order (read FMC_NSTAT first, wait only while the chip is busy) is also run.

NAND access: Erases block 3007 of CE0.

### 13.3.94 pageorder

Device run: 30.

Procedure and question: DESTRUCTIVE (scratch blocks). In writeseq, pages 11 and 13 (programmed after pages 10 and 12, block otherwise empty) read back erased although each program took its busy time. Is this an MLC page-order rule? The cases are: (a) pages 0..7 in order in one run (the original firmware's overlapped transfers); (b) page 5 alone in an erased block; (c) page 4 alone; (d) pages 4 and 5; (e) pages 1 and 3 (odd only); (f) an erase sent with 4 address cycles (FMC_ANUM 3) of a block holding a page: is it erased?

NAND access: Programs and reads pages in blocks 3008 to 3013 of CE0 (one scenario per block) and erases a block that holds a page.

### 13.3.95 op09w33

Device run: 30.

Procedure and question: `wait2` condition 33 passed when idle and during an erase (condition 5 is the other true one). The experiment tries condition 33 in the other chip states. It also tries the condition numbers that share bits with 5 or 33 (1, 4, 32, 37, 65, 69, 97, 133, 161, 165) when idle, to look for a pattern.

### 13.3.96 ev5arm

Device run: 31.

Procedure and question: `wait` event 5 is set by the first status latch of a program only. What re-arms it inside a program? Each case is: latch, X, latch, with X being nothing, the set-up word, the status-clear word, a CE deselect and reselect, FMCTRL1 = 0x100e0, a 16-byte read from the chip (00h and a 0x32 transfer), an ECC kick, or a DMA. A further case is a latch with FMC_DATA_SEL = 0x100 (what does it set?).

### 13.3.97 writeseq2

Device run: 31.

Procedure and question: DESTRUCTIVE (scratch blocks). Multi-page runs with the status poll done right (FMC_DATA_SEL = 1): eight pages in order with overlapped transfers and FMC_DMA_CTRL = 7 before the last DMA; pages 1 and 3; a page programmed again without an erase (FAIL bit?); and the status byte while a program runs.

NAND access: Programs pages in blocks 3014 and 3015 of CE0 and reads them back.

### 13.3.98 eccmode3

Device run: 31.

Procedure and question: ECC_MODE: (1) the length field, bits 20:16, with the values 3, 5, 12 and 31, and with bits 24:21 added (is the field wider?); (2) a decode with the length field set (does it check only those bytes?); (3) parity samples for the codes not yet identified (bit 15 and the modes without bit 8): sectors with a single bit set, at bit positions 0..15 and 4080..4095.

### 13.3.99 ev5two

Device run: 32.

Procedure and question: DESTRUCTIVE (scratch block on CE0 and CE1). `wait` event 5 is re-armed by FMCTRL1 operations. Does a chip becoming ready re-arm it too? A page is programmed on CE0 and on CE1; CE0 is polled (its latch waits for ready), and then CE1 is polled with no FMCTRL1 operation in between (the original WRITE does this). A further question: do address cycles (FMCTRL1 = 1) re-arm it?

NAND access: Programs a page in block 3016 on CE0 and on CE1 (after erasing it).

### 13.3.100 b15ecc

Device run: 32.

Procedure and question: ECC_MODE bit 15 (0x8180) selects the GF(2^10) code with 20 parity bytes. The experiment measures its correction strength (encode and decode in one program, with k flipped bits spread one per 10-bit symbol, and with bursts inside a few symbols). It also takes parity samples across the sector to identify the generator.

### 13.3.101 eccmode4

Device run: 33.

Procedure and question: ECC_MODE leftovers. For the modes without bit 8: is the result reproducible (same sector three times, after a soft reset or not), is it linear over GF(2) (data with bit 0, bit 1 and both set), and does an earlier operation change it (after a BCH decode or encode)? For 0x1ffffff: which bit beyond bit 8, bit 15 and length 31 changes the code? For bit 15: the upper 10 bytes of the ECC area after an encode only and after a decode.

### 13.3.102 finale

Device run: 34.

Procedure and question: The last open points. (1) ECC_MODE bits 5:1 with the BCH (0x180) and the Reed-Solomon (0x8100) code, with single-byte samples for the bits that change something. (2) The sequence of the modes without bit 8: 48 values in one program. (3) What FMC_28 counts (sti, ldi, DMA, chip transfers, ECC operations, plain instructions). (4) Do the words 0x1cc and 0x1d0 of the 0x100 block change the time of a NAND transfer? (5) Which NAND controller interrupt sources raise VIC0 line 20? (6) `wait2` conditions 0..63 while an ECC operation and a DMA are still running.

### 13.3.103 startdiff

Device run: 35.

Procedure and question: Start-word bit 6: the original operating systems start with 0xfff5 and the 4G/5G bootloader with 0xffb5, and no difference between them has been found. Bit 6 is part of the timeout step n (bits 7:4). The values n = 15 and 11 (and 13 and 9) give the same step, so each such pair differs in bit 6 alone. The same set of programs is run under 0xfff5 / 0xffb5 and under 0xffd5 / 0xff95, each after a soft reset, and every observable is compared. The observables are the end reason, the time, CS_IRQ, r0..r7, the interrupt lines, the whole window after the run, and the sequencer's reads and writes of the reserved parts of the window (0xc60..0xcfc, 0xd80..0xffc, 0x100..0x1fc, 0x3a8..0x3b4).

### 13.3.104 startbits

Device run: 35.

Procedure and question: The experiment checks what the start-word bits do beyond the known ones (bit 0 start, bit 1 pause, bit 2 timeout enable, bits 3 and 16 hold, bits 7:4 timeout step, bits 15:8 timeout length, bit 18 read-back status). Each bit 4..31 is flipped alone in 0xfff5 (bit 16 is left out because it holds the sequencer). For each variant a set of programs is run that looks at things not measured before: instruction fetch from SDRAM and from internal SRAM, system-memory reads (ldi) and writes (sti), window reads (pld), sig and the interrupt lines, a DMA plus an ECC encode, and a fault with vectoring. Every variant is compared with the 0xfff5 baseline, which is run three times to measure the run-to-run spread. Only differences beyond that spread are reported. Timeout lengths change with bits 4..15 by design and are shown for the fault program only.

### 13.3.105 rsdecode

Device run: 35.

Procedure and question: The experiment covers Reed-Solomon with 12 and 16 parity symbols (ECC_MODE bit 15 together with bit 2 or bit 3). It checks the decode strength (encode and decode in one program, with k bytes changed) and where the 20 parity bytes of the 16-symbol code go. After the encode, the whole parity buffer (all four slots) is copied out; a slot holds 16 bytes.

### 13.3.106 vic20

Device run: 35.

Procedure and question: VIC0 line 20 was seen for different FMC interrupt sources in two runs. Each source is tried with its enable bit, three times, each time after a soft reset. Is the line stable per source, and which CS_IRQ bits come with it?

### 13.3.107 fmc28b

Device run: 35.

Procedure and question: FMC_28 stays 0 through every operation after the set-up word, but rose by about 124 per probe run without it. A series of runs reads FMC_28 at its start and end, with and without the set-up word, and with nothing, a long loop, a chip read, a DMA or an ECC encode in between. The CPU's window dumps give the value between runs.

### 13.3.108 c58b1

Device run: 35.

Procedure and question: C58 bit 1: an END ran the fault vector once after the CPU wrote C58 = 2 and then C4C, and never when the program wrote C4C. The CPU writes in different orders and repeats them (no clean-up writes before them). Which write arms the one-shot, and is it used up?

### 13.3.109 evsweep2

Device run: 35.

Procedure and question: `wait` events 0..15 and `wait2` conditions 0..255 are tried in states not tried before: during an erase, a page program and a page read (chip busy); with an ECC encode in flight; with a DMA in flight; with every 0x044 / 0x844 interrupt source enabled and a command flag pending; after a second-set command; and idle. The timeout is short (start word 0x3f15: 64 x 8 cycles) and C58 is 0, so a timeout continues; a pass means no fault. Each state is swept in forward and in reverse order, because the chip state changes while the sweep runs.

### 13.3.110 c50fault

Device run: 35.

Procedure and question: C50 bit 0 (the original fault handlers test it and write 1 to it) and bit 8. A fault (the timeout of a `wait` on event 10, and an illegal opcode) is raised while a command flag is pending, with C58 = 0 and with C58 = 4. C50 is read before the fault, right after it (C58 = 0) or in the handler (C58 = 4), and after the handler's write of 1. For bit 8, C50 is read after a soft reset, after an END-only run, and after each kind of operation.

### 13.3.111 ctrlwords

Device run: 35.

Procedure and question: C10, C48 and C54 hold values but have no known effect, and the original fault handlers read C3C. The same operations (a timeout fault with C58 = 4, an illegal opcode, `wait2` condition 33, a DMA plus ECC encode, and two `tmwait` instructions (opcode `15`)) are run with the three words C10, C48 and C54 written all ones by the program and with them written zero. C3C is also read in handlers entered from three different faulting offsets, and in a second fault inside the handler.

### 13.3.112 c08self

Device run: 35.

Procedure and question: CS_STATUS as the program itself reads it (ld 0xc08 and pld), in places where the CPU cannot sample it: right after a DMA, an ECC kick and a chip command are started, inside a fault handler, after sig, after a `wait2` condition 33, and after a CPU pause and continue (C08 read after the pause point).

### 13.3.113 latchrep

Device run: 35.

Procedure and question: The experiment covers the intermittent results: `wait` event 5 straight after the 70h latch, the status latch's `wait` event 5 (once timed out in two identical runs), FMC_STATUS bit 24 instead of bit 23, and `wait` event 3 on CE0. Each is run 40 times, with and without a fresh arm (FMCTRL1 = 0x20 before the latch), and with and without 0x044 / 0x844 written. The tallies are of passes and timeouts and of the FMC_STATUS bits after the latch.

### 13.3.114 irq044

Device run: 35.

Procedure and question: Word 0x044 is set bit by bit (bits 0..31), with three runs per bit, each after a soft reset, using one program that does a command, an address, a 16-byte data read, a status latch and a DMA. Which bits raise CS_IRQ bit 4 or bit 5, VIC0 line 20 and VIC1 line 22, and is the result stable?

### 13.3.115 fmcbits

Device run: 35.

Procedure and question: DESTRUCTIVE (block 3050 of CE0). A page written by the engine is read back with FMC_CTRL0 bit 9 set, bit 11 clear and bit 24 (DMA enable) set for the whole read; with FMC_DMA_CTRL = 0, 7 or 0xff before the last DMA; and with the drain select 0x102 or 0x1e0 instead of 0x100. READID is also run with bit 9 set and bit 11 clear. The data, the decode results and the time are recorded.

### 13.3.116 fmcstat

Device run: 35.

Procedure and question: FMC_STATUS has power-on bits 12, 13, 18 and 19. Each is acknowledged alone (write 1), and the experiment then finds which operation sets it again (a command, an address, a data read, a status latch, a DMA, an ECC encode, a meta encode, a second-set command). The value right after a soft reset is also recorded.

### 13.3.117 eccregs

Device run: 35.

Procedure and question: ECC engine words without a known effect. Words 0x940..0x950 are written with 0, all ones or 0x55555555 before an encode and a decode with 8 bit errors (parity and decode result are compared). Words 0x990..0x99c are read after encodes of zeros, ones and a ramp in the BCH, Reed-Solomon and no-bit-8 modes.

### 13.3.118 seqrestart

Device run: 35.

Procedure and question: The sequence of the modes without bit 8 restarted once and continued once between runs. Each run does three encodes of the same sector in mode 0, with one action between the runs: nothing, a soft reset, an ECC_MODE write, an write of 0 to ECC_KICK, a BCH encode, a BCH decode, an ECC_DONE write, a write of 0x940..0x950, an FMC_CTRL0 write, or the sequencer reset (0x30000). The area words 0x900..0x913 of every encode are recorded.

### 13.3.119 done5

Device run: 35.

Procedure and question: ECC_DONE after an encode and after a decode, for ECC_MODE bits 9..14 and 16..24 set one at a time, on the BCH mode 0x180, the Reed-Solomon mode 0x8100 and the no-bit-8 mode 0x0. The rule for ECC_DONE bit 5 has so far been fitted to only a few modes.

### 13.3.120 progtime

Device run: 35.

Procedure and question: DESTRUCTIVE (blocks 3060 and 3061 of CE0, block 3060 of CE1). Every page of a block is programmed in order with four sectors of a pattern (engine encode), one run per page. The run time (ready poll included) is recorded per page. Are pages 4..11 slower, and what is the pattern over pages 0..127?

### 13.3.121 regsweep

Device run: 35.

Procedure and question: Register numbers 8..255 in every operand role. Only samples were tested before: 9, 11..16, 24 and 255 as a movi destination, 9 / 15 / 255 as an argument-field source, 9 / 14 as a register-field source, and 9 as an ldi address; r10 was never tried as a source. Before each role, r0..r7 are seeded with distinct values and the program first writes distinct values to the writable control words C10, C48, C54 and C3C. This makes 'reads r0', 'reads 0' and 'reads a control word (0xc18 + 4 N)' distinguishable. Program A tests destinations (number 8 is left out because a write to it is a jump): movi, mov and add to N, with r0..r7 stored to parameter RAM after each and the whole window compared. Program B tests sources: mov, add and or with N in the argument field, st, bnz and bz with N in the register field, and jr N. Program C tests addresses and is run for all numbers after A and B: ldi with N as the address register (r0..r7 and C48 / C3C hold scratch addresses with markers), and pld with N as the address register (r0..r7 hold parameter-RAM offsets with markers). Numbers are grouped by identical behaviour.

### 13.3.122 seqfirst

Device run: 36.

Procedure and question: The fixed sequence of the ECC modes without a code started at value 1 in some device runs and continued in others. One encode in mode 0 is the first ECC operation of the run, after a power cycle of the unit: which value of the sequence comes out?

### 13.3.123 w2state

Device run: 36.

Procedure and question: `wait2` condition 33 was true in every state of device runs 29 to 34 and in none of device run 35; condition 20 was true once, with every interrupt enable on and a command flag pending. All `wait` events 0..15 and `wait2` conditions 0..255 are tried with a short timeout (start word `0x3f15`), first in the first program of the run, then after one change of state at a time: a soft reset; `0x044` = `0xa5a5` and `0x844` = `0x25` left set; each `0x044` bit 0..15 alone with a command flag pending; `0x844` = `0x2f` with a command flag pending; each `FMC_STATUS` / `ECC_DONE` flag left pending; a READ ID through the second register set; the channel-1 pins switched; a page read of CE0; `C10` = 0 and `0xff`; no chip, CE0 or CE1 selected; the start word `0x3ff5`.

### 13.3.124 c50b8

Device run: 36.

Procedure and question: `C50` bit 8 was seen once after a soft reset (`0xffff1900`). The CPU reads `C50` before and after each of these runs: the first two runs, a run after a soft reset, a run after a sequencer reset alone (`FMSS_CTRL` = `0x30000`), a run that faults, a run with a second-set command, a run with an ECC decode, and ten `end`-only runs.

### 13.3.125 startregion

Device run: 36.

Procedure and question: On the 4G / 5G the bootloader starts its programs with `0xffb5` and reads only its own blocks, while the operating system starts with `0xfff5` and works the user area; every earlier comparison of the two start words ran in the user area. Page 0 of blocks in the user area (blocks 0..3871 of this chip) and of blocks above it, on CE0 and CE1, is read under `0xfff5`, `0xffb5`, `0xffd5` and `0xff95` (the second pair also differs in bit 6 alone); per read the end of the run, its time, the four decode results, the `FMC_STATUS` words and a hash of the data and chunks are compared. Then (destructive) a user block (3002) and the first block above 3871 whose page 0 read erased are erased and programmed under each of the two start words in turn, and each is read back under both.

### 13.3.126 bufport

Device run: 36.

Procedure and question: The original READID reads the 8 ID bytes of data buffer 0 through `FMC_FIFO` (`0x080`). Here a buffer is filled by DMA with a known pattern (`i * 29 + 7`) and `0x080` is read 130 times (one word more than 512 bytes), with `FMC_28` read every 32 reads: data buffer 0 (drain select `0x100`) and data buffer 1 (`0x200`) after `FMCTRL1` = `0x340`, data buffer 0 without `0x340`, parity buffer 0 (`0x1000`), each once and twice in a row. Then `0x080` is written 128 times with `FMC_DATA_SEL` = 1 (without and after `FMCTRL1` = `0x340`) and data buffer 0 is copied out by DMA: can a program write a data buffer through the port?

### 13.3.127 c58b2

Device run: 36.

Procedure and question: `C58` bit 1: an `end` ran the fault vector once in device run 8, and never in device run 35, where the job's park wrote 0 to `C58`, `C4C` and `C5C` after every run. Here the job's clean-up is switched off before and after the run, so `C58` keeps its value for three runs in a row. Cases: the CPU writes `C58` = 2 and then `C4C`; `C4C` and then `C58` = 2; `C58` = 2 with the program writing `C4C` (the control case of device run 8, also with the job's normal steps); the program writing `C58` = 2 and `C4C`; the CPU writing `C58` = 3 or 6 and `C4C`.

### 13.3.128 stprev

Device run: 36.

Procedure and question: `st` with a register number above 7 stored 0 after `movi r6, #0` (device run 15) and r7's value after `movi r7` (device run 35). The instruction just before `st [0xd70], rN` (N = 9, 14, 200) is varied, with r0..r7 seeded with distinct values first: `movi` to r0, r3 or r7; `add`; `mov`; `ld` of a window word; `pld`; `ldi`; `wr` to the window; `st` of r5; a `wait` that passes; `nop`; and nothing after the seeding.

### 13.3.129 tmwait2

Device run: 36.

Procedure and question: Two `tmwait` in one program made the program run 4841, 18 and 3250 us in three runs. `C14` is read at the start and after each of four `tmwait`, and the program's run time is recorded: the same program three times in a row, then after a run that timed out, then after a soft reset.

### 13.3.130 irqop

Device runs: 36, 37.

Procedure and question: Which `FMC_STATUS` flag each `0x044` bit routes into `CS_IRQ` bit 4, and what `wait` event 15 waits for. For `0x044` bits 0, 1, 2, 11 and 12 (and no bit), one operation per run whose flag is left pending (a command, an address, a 16-byte data read, a status latch, a DMA): `CS_IRQ` bit 4, VIC0 line 20, and whether `wait` event 15 passes (`C58` = 0; the fault flag is read after it). In device run 37 each program also acknowledges every flag and writes 0 to `0x044` and `0x844` before its `end`, and the experiment runs last.

### 13.3.131 done5b

Device run: 37.

Procedure and question: `ECC_DONE` bit 5 with `ECC_MODE` bit 21, 22 or 23 alone on `0x180` read set in device run 30 (encode only) and clear in device run 35 (encode, then decode, in one program). For each of bits 21..24: an encode only; an encode and then a decode; a decode first and then the encode; each case after a soft reset.

### 13.3.132 rsresult

Device run: 37.

Procedure and question: A Reed-Solomon encode (`0x8180`) left `ECC_RESULT` = `0x20000` where BCH encodes kept the earlier value. `ECC_RESULT` is read before and after an encode with 8, 12 and 16 Reed-Solomon symbols and with BCH, that follows: nothing (after a soft reset), a BCH decode with 0 and with 3 corrected bits, a Reed-Solomon decode with 0 and with 2 corrected bytes.

### 13.3.133 rschip

Device run: 37.

Procedure and question: Destructive (block 3070 of CE0). Reed-Solomon parity on the chip: pages written by the engine with RS-8 (`0x8100`), RS-12 (`0x8104`) and BCH, read back as stored and decoded in the same mode. Stored byte errors: a page is programmed a second time without an erase, with `0x00` at k data bytes of each sector and `0xff` elsewhere (programming only clears bits), and decoded: RS-8 with 4 and 5 bad bytes, RS-12 with 6 and 7. In the buffers: an RS-16 encode with the kick's slot field set to 3, both parity buffers copied out, and a decode with the slot field 3 of data with 2 bad bytes.

### 13.3.134 miscorr

Device run: 37.

Procedure and question: Can a decode report success with wrong data? BCH decodes with 9..64 random flipped bits and RS-8 decodes with 5..40 random bad bytes, 240 sectors each (12 per program, each encoded and decoded in the same program): how many are reported uncorrectable, how many corrected with the original data back, how many reported corrected with other data, and how many reported without error with the data unchanged.

### 13.3.135 fullpage

Device run: 37.

Procedure and question: Read-only. Do both 2 KiB units of a 4 KiB page written by the original firmware carry the same metadata? Pages 0, 1 and 64 of blocks 1, 2, 10, 100, 500, 1000 and 2000 of CE0 and CE1 are read whole (8 sectors, as stored): per page whether it is written, and the 3 metadata bytes of each of the 8 chunks.

### 13.3.136 dmactl

Device run: 37.

Procedure and question: Destructive (block 3051 of CE0). `FMC_DMA_CTRL` = 1 << b (b = 0..7) written before every data DMA of a page read, with the drain selects `0x100` and `0x1e0`; and the combined selects of the original READ (filling one data buffer while selecting the other for draining: `0x201` / `0x102`), alternating the data buffers per sector.

### 13.3.137 fmcstat2

Device run: 37.

Procedure and question: `FMC_STATUS` bits 12, 13, 18 and 19 are set after a soft reset, are not cleared by writing 1, and a 16-byte data read clears bits 13 and 19. After that data read, which operation sets bits 13 and 19 again: a DMA into a data buffer, a DMA out of one, a DMA into a parity buffer, a 512-byte chip data read, an ECC encode, a status latch, a command, a second-set data read, the set-up word?

### 13.3.138 ch1buf

Device run: 37.

Procedure and question: Does the second register set (channel 1) have parity buffers and two data buffers? Through the `0x400` set a pattern is DMA-ed into parity buffer 0 or 1 (`0x414` = `0x10` / `0x20`) and back (`0x1000` / `0x2000`), and into data buffer 0 or 1 and back; before that the first set's parity buffer 0 is filled with another pattern, and it is copied out afterwards.

### 13.3.139 evsweep3

Device run: 37.

Procedure and question: All `wait` events 0..15 and `wait2` conditions 0..255 (short timeout) during operations not swept before: a second-set DMA, a meta encode, a meta decode, an ECC decode, a chip data write (`80h`, address, 512 bytes; cancelled with `FFh` afterwards) and a parity-buffer DMA, each still running.

### 13.3.140 blk100b

Device run: 37.

Procedure and question: The `0x100` block and the words `0x3a8..0x3b4`. Each writable word of the `0x100` block is set by the sequencer (to 0; `0x1b8` to 1; `0x1cc` and `0x1d0` also to `0xffffff`), and the same program then runs 8 DMAs, 8 ECC encodes and decodes, a 512-byte chip read, a `wait` that times out, and `wait2` conditions 33 and 5; the word is restored at the end. `0x3a8..0x3b4` are written by the CPU before the start and by the program (`pst`), and read by both.

### 13.3.141 straps

Device run: 37.

Procedure and question: Read-only, CPU only. The boot ROM chooses its boot device from GPIO port 3 pins 6 and 7, and checks port 1 pins 0 and 1 together with bit 0 of the USB PHY word `0x3c400028` for a USB boot. The CPU reads the configuration and data words of ports 1 and 3 and that PHY word.

### 13.3.142 w2vic

Device run: 38.

Procedure and question: Are the `wait2` conditions the CPU's interrupt lines? `wait2` condition 20 is true exactly when VIC0 line 20, the NAND controller's interrupt line, is raised. For N = 0..63, just before each `wait2 N` (short timeout, start word `0x3f15`), the program reads the raw interrupt status words of VIC0 (`0x38e00008`) and VIC1 (`0x38e01008`) with `ldi`. For each N the pass or timeout is compared with raw line N (VIC0 bit N for N < 32, VIC1 bit N - 32 for N = 32..63). States: idle; a command flag pending with `0x044` bit 0; a DMA flag pending with `0x044` bit 11; an `ECC_DONE` flag pending with `0x844` bit 0. Then N = 64..255 once, idle. Each program acknowledges every flag and writes 0 to `0x044` and `0x844` before its `end`.

References: [chapter 14](14-evidence.md) (outcomes), [chapter 13](13-experiments.md) (what was done and why), [chapter 15](15-evidence-interpretation.md) (what the outcomes mean).

Related chapters: [chapter 14](14-evidence.md), [chapter 15](15-evidence-interpretation.md).
