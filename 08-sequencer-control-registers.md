# Chapter 8 — Sequencer control registers

Summary: The registers of the sequencer's control block (register-window offsets `0xc00` to `0xc5c`): what each register holds, who writes it and how it behaves, including the meaning of each bit of the start word. How the sequencer runs a program is in [chapter 2](02-architecture.md); the timeout length table is in [chapter 7, section 7.5](07-events-and-waits.md#75-timeout-length).

## 8.1 Sequencer control block

The control block sits at register-window offsets `0xc00` to `0xc5c`. The table gives one line per register. The sections after it give the details in the same order. "Device" marks behaviour measured on the device, and "inferred" marks an interpretation.

| offset | name | what it is | written by |
|---|---|---|---|
| `0xc00` | `FMSS_CTRL` | start word, reset, and a "program not finished" status bit | CPU |
| `0xc04` | `FMSS_PROG` | bus address of the program | CPU |
| `0xc08` | `CS_STATUS` | state code: running, waiting, paused, stopped at `pause`, idle | read only |
| `0xc0c` | `CS_IRQ` | flags: end, signal, fault, pause, interrupt sources | set by hardware, cleared by CPU and program |
| `0xc10` | `FMSS_C10` | 8-bit word that the program prologue writes with 1; no effect found | program |
| `0xc14` | `FMSS_C14` | timeout timer, 8-bit down-counter | hardware only |
| `0xc18` to `0xc34` | r0 to r7 | the program's registers; r6 is the result | CPU and program |
| `0xc38` | `FMSS_PC` | program counter (register r8) | program; CPU only at a pause |
| `0xc3c` | `FMSS_C3C` | read by fault handlers; inside a fault handler it holds the offset of the faulting instruction + 8 | hardware |
| `0xc40` | | instruction word of the instruction the sequencer is holding | hardware only |
| `0xc44` | | immediate of the instruction the sequencer is holding | hardware only |
| `0xc48` | | 32-bit word; no effect found | CPU and program |
| `0xc4c` | `FMSS_C4C` | fault vector (byte offset in the program) | program prologue |
| `0xc50` | `FMSS_C50` | which of `wait` events 0 to 5 are pending | hardware |
| `0xc54` | | 12-bit word; no effect found | CPU and program |
| `0xc58` | `FMSS_C58` | fault routing | program prologue |
| `0xc5c` | `FMSS_C5C` | set to 1 by the fault routing | hardware, CPU |

### 8.1.1 Start word and timeout (`FMSS_CTRL`, device)

`FMSS_CTRL` (`0xc00`) is the register that the CPU writes to reset the sequencer and to start, pause, hold and continue a program. The value written is called the start word.

The original firmware writes these two start words:

- `0x30000` resets the sequencer.
- `0xfff5` starts a program. The 3G and the 4G/5G operating systems use it.

The 4G/5G bootloader starts a program with `0xffb5`. This differs from `0xfff5` only in bit 6. The bootloader also polls for the end of the program instead of taking the interrupt.

What bit 6 does is open. No effect of bit 6 has been found in any program tried. These programs included program fetch from SDRAM and SRAM, 2000 `ldi` / `sti` / `pld`, `sig`, a DMA with an ECC encode, a vectored fault, and reads and writes of the reserved window words ([D-startbits-35](14-evidence.md#142132-startbits-run-35), [D-startdiff-35](14-evidence.md#142131-startdiff-run-35)). The reading "bit 6 = interrupt enable" is not confirmed.

| bit | effect when written | read back |
|---|---|---|
| 0 | start | 0 |
| 1 | pause: the program stops before its first instruction, or, when bit 1 is written during a run, exactly where it is. A start word without bit 1 continues the program | as written |
| 2 | timeout enable: without it, a `wait` whose event never comes, or an illegal opcode, waits for ever (no fault; `C14` stays `0xff`) | as written |
| 3 | hold: the sequencer stays in the hold state that follows a reset until a start word without bit 3 (and without bit 16) is written. If bit 3 is written during a run, the program is aborted, and releasing the hold runs it again from instruction 0 | 0 |
| 7:4 | timeout step size n ([chapter 7, section 7.5](07-events-and-waits.md#75-timeout-length)). With n = 0 the timeout comes at once (0 to 2 us, and `C14` then reads `0xfe`). What bit 6 does is open ([chapter 17](17-open-questions.md#171-instruction-set-and-sequencer-behaviour), item 8) | as written |
| 15:8 | timeout length in steps, minus 1 | as written |
| 16 | hold, same as bit 3 | 0 |
| 17 | no effect seen ([D-startbits-35](14-evidence.md#142132-startbits-run-35)) | 0 |
| 18 | not taken from a write | set while a started program has not finished (held, paused or blocked) |
| 19 to 31 | no effect seen in the same set of programs ([D-startbits-35](14-evidence.md#142132-startbits-run-35)) | 0 |

- Bits 15:4 written alone (together with bit 0, without bit 2) changed nothing in a loop, a `wait`, an illegal opcode, `evjmp` or the interrupt line.
- `0xfff5` and `0xffb5` gave the same timeout length, the same CPU interrupt line and the same behaviour in every test tried.
- A start word without bits 1, 3 and 16, written while a program runs, is ignored.
- `0xfff5` sets every bit from 0 to 15 except bits 1 and 3.
- There is no single-step mode: no combination of bits made the sequencer run one instruction per start.

### 8.1.2 `FMSS_PROG` (`0xc04`)

The bus address of the program in system memory, written by the CPU before the start.

### 8.1.3 `CS_STATUS` (`0xc08`)

A state code of the sequencer (device; the CPU read it while a program was in each state). Neither the original programs nor the original CPU driver use it.

| value | state |
|---|---|
| 0 | idle, or held by start-word bit 3 or 16 |
| 1 | running |
| 3 | paused by start-word bit 1 |
| 4 | waiting for a `wait` event, or stalled on an illegal opcode (one state: the timer `C14` counts down in both) |
| 5 | the value that a program reads from `CS_STATUS` itself (with `ld` or `pld`) while it runs. It was 5 in every place tried: right after starting a DMA, an ECC operation or a chip command, after `sig`, and inside a fault handler ([D-c08self-35](14-evidence.md#142140-c08self-run-35)). The CPU reads 1 in the same state |
| `0xa` | stopped at `pause` (opcode `12`) |

- CPU writes are ignored. The reset clears it.
- A start word with bit 1 or bit 3, written while the program is stopped at `pause`, changes the code to 3 or 0 respectively. The program stays stopped at the `pause`.

### 8.1.4 `CS_IRQ` (`0xc0c`)

Flags, cleared by writing 1 to the bit (write-1-to-clear).

| bit | set when (device) |
|---|---|
| 0 | the program ended (`end`) |
| 1 | the program executed `sig` (opcode `10`) |
| 2 | a fault (a timeout or an illegal opcode) |
| 3 | the program is stopped at `pause` (opcode `12`); the 4G/5G CPU driver calls this bit "pause" |
| 4 | an interrupt source of the NAND controller (FMC) that is enabled in its word `0x044`. A program with a command, an address, a data read, a status latch and a DMA was run. In it, bits 0, 1, 2, 11 and 12 of `0x044` each raised `CS_IRQ` bit 4, and the other 27 bits did not (three of three runs, [D-irq044-35](14-evidence.md#142142-irq044-run-35)). Which operation each of those bits belongs to is not identified |
| 5 | bit 2 of the NAND controller words `0x054` to `0x05c` |
| 6 | an `ECC_DONE` bit that is enabled in ECC word `0x844` |

- Bits 4 to 6 follow their sources only once those sources are enabled in `0x044` and `0x844`. Both words are 0 after power-on. The bits were seen after an experiment had left `0x044` = `0xa5a5` and `0x844` = `0x25`.
- The program prologue writes `0xff` to `CS_IRQ`. The CPU writes `0xff` before the start, and writes `5` to acknowledge a finished run.

### 8.1.5 `FMSS_C10` (`0xc10`)

The program prologue writes 1 to it (interpreted as "enable", inferred). The register holds eight bits. No value written to it had a visible effect on a run, the timeout, a `wait`, a DMA or an ECC encode (device, [D-ctrlwords-35](14-evidence.md#142139-ctrlwords-run-35)).

### 8.1.6 `FMSS_C14` (`0xc14`): the timeout timer

An 8-bit down-counter (device). The timeout lasts (start-word bits 15:8) + 1 steps, and the length of one step is set by start-word bits 7:4 ([chapter 8, section 8.1.1](#811-start-word-and-timeout-fmss_ctrl-device)).

- A `wait` that has to wait loads the start word's bits 15:8 into the counter (`0xff` with `0xfff5`) and counts down; `0xf6` was read in the middle of a wait.
- With one of bits 8..15 cleared in the start word, the counter read `0xfe`, `0xfd`, `0xfb`, `0xf7`, `0xef`, `0xdf`, `0xbf` or `0x7f` (one value per cleared bit) after a run ([D-startbits-35](14-evidence.md#142132-startbits-run-35)).
- After a timeout it reads its reload value. The end of a program freezes it at its current value.
- Writes by the CPU or the program do not change it.
- `tmwait` (opcode `15`) waits for it: from a non-zero value it reloads and waits until 0 (about 4.85 ms, leaving 0); from 0 it reloads at once ([chapter 6, opcode `15`](06-opcode-reference.md#15-tmwait)).
- The original erase program writes `0xff` to it after a timeout; that write has no visible effect.

### 8.1.7 r0 to r7 (`0xc18` to `0xc34`)

The program's eight registers; r6 is the result (`SEQ_RESULT`, `0xc30`).

- A CPU write before the start sets the register's starting value.
- A start keeps the values. A reset clears them (device).

### 8.1.8 `FMSS_PC` (`0xc38`): the program counter

The byte offset of the current instruction while a program runs or is stopped; 0 after the end.

- It is register r8 (`0xc18 + 4 * 8`). A program reads it as r8 or with `ld`. A program that writes it (`movi r8`, or `wr` / `st` to `0xc38`) jumps to the written offset.
- The CPU can write it only while the program is stopped at `pause` ([chapter 6, opcode `12`](06-opcode-reference.md#12-pause)); at any other time CPU writes are ignored (device).

### 8.1.9 `FMSS_C3C` (`0xc3c`)

The original fault handlers read this register.

- It reads 0 after every run, whatever the CPU wrote (device).
- Inside a fault handler it holds the offset of the faulting instruction + 8 (device). Faults at `0x28`, `0x40` and `0x70` gave `0x30`, `0x48` and `0x78`. A second fault inside the handler gave the offset of the faulting instruction in the handler + 8 ([D-ctrlwords-35](14-evidence.md#142139-ctrlwords-run-35), [D-c50fault-35](14-evidence.md#142138-c50fault-run-35)).

### 8.1.10 `0xc40` and `0xc44`: the held instruction

The instruction word (`0xc40`) and the immediate (`0xc44`) of the instruction the sequencer is holding (device):

- after the priming start that follows a reset;
- while a program is stopped at `pause`;
- while an instruction waits (`wait`, `wait2`, an illegal opcode).

CPU writes are ignored. The reset clears both.

### 8.1.11 `0xc48`

32 bits, writable by the CPU and by the program; the original programs do not use it.

- A program start clears it, so the program never sees a value that the CPU wrote.
- A value written by the program stays after the end of the program.
- No effect was found on a run, the timeout, a wait, a DMA or an ECC encode (device, [D-ctrlwords-35](14-evidence.md#142139-ctrlwords-run-35)).

### 8.1.12 `FMSS_C4C` (`0xc4c`): the fault vector

The byte offset of the fault handler in the program, written by the program prologue.

### 8.1.13 `FMSS_C50` (`0xc50`): pending events

Bits 0 to 6 show which of `wait` events 0 to 6 are pending (device; bits 0..5 were each seen with only that flag pending, bit 6 after a command sent through the second register set, [D-c50b8-36](14-evidence.md#142152-c50b8-run-36)):

| bit | `wait` event |
|---|---|
| 0 | event 0, command sent |
| 1 | event 1, address sent |
| 2 | event 2, DMA done |
| 3 | event 3, data transfer done |
| 4 | event 4, an `ECC_DONE` bit |
| 5 | event 5, status latch |
| 6 | event 6, command sent by the second register set |

- Each bit follows its flag and clears when that flag is acknowledged.
- The original fault handlers test bit 0 and write 1 to it. Bit 0 is set while a command flag is pending and clear otherwise. A timeout or an illegal opcode does not change it, and the handler's write of 1 does not clear it (device, [D-c50fault-35](14-evidence.md#142138-c50fault-run-35)).
- The value at power-on is `0xffff1800`. Bit 8 was seen set once after a soft reset (`0xffff1900`). In runs 35 and 36 it stayed clear, in 17 reads around soft resets, a sequencer reset, a fault, a second-set command, an ECC decode and ten `end`-only runs ([D-c50b8-36](14-evidence.md#142152-c50b8-run-36)). Its meaning is open.

### 8.1.14 `0xc54`

12 bits, writable by the CPU and by the program.

- A program start clears it; the value stays after the end.
- No effect was found; it is not a timeout setting (device).

### 8.1.15 `FMSS_C58` (`0xc58`): fault routing

The register holds eight bits (writing `0xffff` reads back `0xff`, writing `0x100` reads back 0). The program prologue writes 4. The register keeps its value across a reset. It is not a timeout scale: every value 0..31, and also `0x40`, `0x80`, `0xff` and `0x100`, gave the same timeout time (device).

| bit | effect (device) |
|---|---|
| 0 | at an `end`: sets `C5C`, and every later `end` then runs the fault vector once, until the CPU rewrites `C58`, `C4C` or `C5C`. At a fault: sets `C5C` |
| 1 | with bit 1 alone (`C58` = 2) no `end` ran the fault vector in three runs in a row, whether the CPU wrote `C4C` before or after `C58` or the program wrote it (run 36, with the job's clean-up switched off, [D-c58b2-36](14-evidence.md#142155-c58b2-run-36); run 35). With bits 1 and 2 (`C58` = 6) the first `end` after the CPU wrote `C58` and `C4C` ran the fault vector once, and the next two runs, with `C58` still 6, did not (runs 35 and 36). In run 8, `C58` = 2 with `C4C` written by the program diverted one `end`. The condition is open |
| 2 | a timeout or an illegal-opcode fault continues at the fault vector (with the bit clear, it continues at the next instruction), and sets `C5C` |
| 3 to 7 | no visible effect |

### 8.1.16 `FMSS_C5C` (`0xc5c`)

The fault routing of `C58` (section 8.1.15) sets it to 1. The CPU can write it. It keeps its value across a reset (device).

References: [chapter 14](14-evidence.md) (outcomes), [chapter 13](13-experiments.md) (what was done and why), [chapter 15](15-evidence-interpretation.md) (what the outcomes mean).

Related chapters: [chapter 2](02-architecture.md), [chapter 3](03-cpu-memory-map.md), [chapter 6](06-opcode-reference.md), [chapter 7](07-events-and-waits.md).
