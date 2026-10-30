# Chapter 3 — The register window as the ARM CPU sees it

Summary: The NAND hardware occupies a 4 KB register window at CPU address `0x38a00000`. This chapter describes it from the CPU side: the segments of the window, the mirrors (offsets that reach the same register), the words that always read 0, the access rules, the interrupt lines and the way the original CPU driver runs a program. The sequencer's own view is in [chapter 4](04-sequencer-memory-map.md).

## 3.1 Offsets and names

All offsets are from the base of the NAND controller (FMC) register window, `0x38a00000`. Register names are descriptive labels for window offsets. Registers that are not listed are not touched by the original programs.

## 3.2 Memory map: the register window

The sequencer works with three address spaces; the first is the register window described here, the other two are system memory and the program itself ([chapter 4, section 4.5](04-sequencer-memory-map.md#45-system-memory-and-the-program)).

**1. The register window** is 4 KB at CPU address `0x38a00000`. Instructions name a word in it by its offset:

- `wr`, `st` and `ld` carry the offset in `arg`.
- `pld` and `pst` take the offset from a register and wrap it to 12 bits, so they reach any word of the window, r0..r7 included.

The CPU sees the same words at `0x38a00000 + offset`. Below `0xc00`, offset bit 9 is not decoded. Some ranges are therefore **mirrors**: the same registers under a second address, not separate ones (device, [chapter 16, section 16.2](16-device-findings.md#162-settled-on-a-device)):

| offsets | what is there | kind |
|---|---|---|
| `0x000..0x0fc` | first register set: the NAND controller the programs use ([chapter 9](09-nand-controller.md)) | registers |
| `0x100..0x1fc` | a block whose purpose is unknown; the words at `0x1b8..0x1d8` are writable (at power-on `0x1cc` = `0x8060`, `0x1d0` = `0x3010d0`, and the other writable words are all ones); the rest of the block is read-only | registers |
| `0x200..0x3fc` | `0x000..0x1fc` again: every word reads the same and every write mask is the same (CPU and sequencer writes, [chapter 16, section 16.2](16-device-findings.md#162-settled-on-a-device)) | **mirror** of `0x000..0x1fc` |
| `0x400..0x4fc` | **second register set**: a second, independent NAND controller with the same register layout ([chapter 9, section 9.5](09-nand-controller.md#95-the-second-nand-controller-register-set-0x400)). Not a mirror: it holds its own values | registers |
| `0x500..0x5fc` | read from the sequencer, every word equals the word `0x400` lower in the `0x100` block (`0x100..0x1fc`) | mirror of the `0x100` block (inferred from the equal contents) |
| `0x600..0x7fc` | `0x400..0x5fc` again (same reads, same write masks) | **mirror** of `0x400..0x5fc` |
| `0x800..0x8fc` | ECC engine ([chapter 10, section 10.1](10-ecc-engine.md#101-how-error-correction-works)) | registers |
| `0x900..0x9fc` | further ECC registers: `0x900..0x90c` hold the 13 parity bytes after an encode (read-only), `0x940..0x950` are five 32-bit read/write words (no effect known), `0x9a0` changes during a decode, `0x9a4` is a decode counter, and `0x9c0` has a read/write bit 0 (1 at power-on) | registers |
| `0xa00..0xbfc` | `0x800..0x9fc` again (same reads, same write masks) | **mirror** of `0x800..0x9fc` |
| `0xc00..0xc5c` | sequencer control block ([chapter 8, section 8.1](08-sequencer-control-registers.md#81-sequencer-control-block)): start word, program address, `CS_IRQ`, r0..r7 at `0xc18..0xc34`, PC, fault vector and others | registers |
| `0xc60..0xcfc` | nothing: reads 0, writes are lost | empty |
| `0xd00..0xd7c` | parameter RAM, 32 words shared by the CPU and the program ([chapter 4, section 4.4](04-sequencer-memory-map.md#44-parameter-ram)) | RAM |
| `0xd80..0xdfc`, `0xe00..0xffc` | nothing: reads 0, writes are lost | empty |

Who can read and write what (device, [`access`](14-evidence.md#142105-access-run-27)): every word was written with all ones and with zero, once from the CPU and once from the sequencer. Almost every register behaves the same for both. The differences are:

- The PC (`0xc38`): the sequencer can write it (a jump); the CPU can write it only while a program is stopped at `pause`.
- `C3C` (`0xc3c`), `C48` (`0xc48`) and `C54` (`0xc54`): a value written by the CPU before a start is not what the program reads, because the start clears these registers. A value that the program writes to `C48` or `C54` stays visible to the CPU after the run. `C3C` is cleared at the end of the run.
- After the CPU wrote the words `0x3a8..0x3b4`, the sequencer read back the written values, but the CPU itself read 0 (unexplained).
- The status words (`0x050`, `0x250`, `0x450`, `0x650`, `FMC_28` and their copies) changed by themselves during the test, so the write masks measured on them carry no information.

## 3.3 CPU-only rules

Rules that hold for CPU accesses to the window, in order of offset (device unless stated). The full register table is in [chapter 8](08-sequencer-control-registers.md); see also findings 55 and 103.

- `0xc08` (`CS_STATUS`): CPU writes are ignored.
- `0xc14` (`FMSS_C14`): neither CPU nor program writes change it.
- `0xc18..0xc34` (r0..r7): a value that the CPU writes before the start becomes the register's starting value; a start leaves the registers unchanged; the sequencer reset clears them.
- `0xc38` (`FMSS_PC`): the CPU can write it only while the program is stopped at `pause` ([chapter 8, section 8.1](08-sequencer-control-registers.md#81-sequencer-control-block), [chapter 6, opcode `12`](06-opcode-reference.md#12-pause)); otherwise CPU writes are ignored.
- `0xc3c` (`FMSS_C3C`): reads 0 after every run, whatever the CPU wrote.
- `0xc40`, `0xc44`: CPU writes are ignored; the sequencer reset clears them.
- `0xc48`: 32 bits, writable by the CPU and by the program, not used by the original programs. A program start clears it, so the program does not see a value written by the CPU. A value written by the program stays after the end. No effect found.
- `0xc50` (`FMSS_C50`): CPU writes are ignored.
- `0xc54`: 12 bits, writable by the CPU and by the program. A program start clears it; a value written by the program stays after the end. No effect found; it is not a timeout setting.
- `0xc60..0xcfc` and `0xd80..0xdfc`: read 0 and drop writes, from the CPU and from the program.

CPU write masks (row "CPU register masks" of [chapter 16, section 16.2](16-device-findings.md#162-settled-on-a-device), device). Each mask is the set of bits that kept their value, found by reading each word back after a CPU write of all ones, of 0 and of `0xa5a5a5a5`. The table gives the number of writable low bits, or an explicit mask.

| block | writable bits per word | notes |
|---|---|---|
| sequencer | `0xc04`, r0..r7, `0xc3c`, `0xc48`, `0xc4c`: 32 bits; `0xc10`, `0xc58`: 8 bits; `0xc54`: 12 bits; `0xc5c`: 1 bit | read-only: `0xc08`, `0xc14`, `0xc38`, `0xc40`, `0xc44`, `0xc50`; the rest of `0xc00..0xcfc` reads 0 |
| NAND controller (`0x000..0x0fc`) | `0x000`, `0x00c`, `0x010`, `0x018`, `0x034`, `0x060..0x078`: 32 bits; `0x014`: 16 bits; `0x020`: 24 bits; `0x024`, `0x028`: 9 bits; `0x02c`: 3 bits; `0x030`: 13 bits; `0x044`: 16 bits | |
| ECC engine | `0x804`, `0x808`: 32 bits; `0x814`: 25 bits; `0x844`: mask `0x2f` | `0x800` reads `0x20060808` and ignores writes |
| second register set (`0x400`) | the layout of the first set again: `0x400..0x478` match `0x000..0x078` bit for bit; `0x438`: 13 bits; `0x47c`: mask `0xffc` | `0x404` has write mask `0x19`; its bits 1, 2, 8 and 9 behave like status bits |

## 3.4 CPU-visible interrupt lines

The original CPU driver polls the VIC1 raw status bit 22 (source 54) for the end of a run ([RE-32](14-evidence.md#14332-re-32-cpu-driver-wait-and-classify)). In the device tests that bit stayed clear after END, after a fault and with any start word. With a NAND controller or ECC interrupt enable set, only `CS_IRQ` bits 4 and 6 responded. Whether a further enable outside the sequencer block is needed is open ([chapter 2, section 2.3](02-architecture.md#23-end-and-fault)).

Device results (row "interrupt line" of [chapter 16, section 16.2](16-device-findings.md#162-settled-on-a-device)):

- **`0x044` bit per source** (device, one operation per program with its flag left pending, [`irqop`, device run 37](14-evidence.md#142170-irqop-run-37)):

  | `0x044` bit | source (`FMC_STATUS` flag) | `CS_IRQ` bit 4 | VIC0 line 20 |
  |---|---|---|---|
  | 0 | command sent (bit 1) | set | set |
  | 1 | address sent (bit 2) | set | set |
  | 2 | data transfer done (bit 3) | set | set |
  | 11 | DMA done (bit 20) | set | set |
  | 12 | status latched (bit 23) | set | set |

  Each bit routes exactly one flag; with no bit set nothing was raised. All NAND controller sources share VIC0 line 20 (interrupt source 20). The ECC engine's sources, enabled in `0x844`, raise VIC1 line 11 (interrupt source 43): with `0x844` bit 0 set and an `ECC_DONE` flag pending, the raw status showed line 43 and no other new line ([`w2vic`](14-evidence.md#142171-w2vic-run-38)). In the status-latch program the `70h` command flag, acknowledged before the latch, still left `CS_IRQ` bit 4 set with `0x044` bit 0, while VIC0 line 20 stayed clear: `CS_IRQ` bit 4 keeps a routed flag after the flag is acknowledged, and VIC0 line 20 follows a routed flag only while it is pending (inferred from this program).

- After a power cycle and after a soft reset alike, VIC1 raw bit 22 stayed clear for an END, a fault, opcode `10` and a `12` stop, with and without bit 6 in the start word.
- With every VIC0 and VIC1 raw line recorded, no line followed END, faults or start-word bits.
- With `0x044` = `0xffff`, VIC0 line 20 was seen in the command and status cases. It was not seen in the address, transfer and DMA cases, although `CS_IRQ` bit 4 was set in those cases too. VIC1 line 22 was never seen.
- Single `0x044` bits that set a pending flag in `CS_IRQ` bit 4, by the kind of operation:
  - command: bit 0;
  - address: bits 0 and 1;
  - transfer: bits 0, 1, 2 and 11;
  - DMA: bit 11;
  - status latch: bits 0 and 13.

  These programs raised several flags each (for example, the address case also sent a command), which accounts for the extra bits (inferred from the one-flag table above).
- `0x044` bit 0 routes `FMC_STATUS` bit 1 into `CS_IRQ` bit 4; with a command byte sent, the other fifteen bits routed nothing.
- `0x844` bits 0, 2 and 5 route `ECC_DONE` into `CS_IRQ` bit 6; bits 1 and 3 route nothing, and bit 4 is not writable.
- Even with these enables set, the line stayed clear. Whether a further enable outside the sequencer block is needed is open.
- One source at a time after a soft reset ([`finale`](14-evidence.md#142130-finale-run-34)):
  - VIC0 line 20 appeared with a command and `0x044` bit 0, and with address cycles and bit 1.
  - It did not appear with a command and bit 1 or 2, with an address and bit 0, with a status latch and bit 0 or 13, or with a DMA and bit 11.
  - `CS_IRQ` bit 4 was set for a command with bit 0, an address with bit 1, a status latch with bit 0 and a DMA with bit 11.
- In the [`irqsrc`](14-evidence.md#142101-irqsrc-run-27) tests, line 20 appeared with a command and bit 0 and with a status latch and bit 13, in place of an address and bit 1. The `CS_IRQ` bit 4 results also differed for an address with bit 0 and a status latch with bit 13. Line 20 is therefore not the same as `CS_IRQ` bit 4.

- Each case three times after a soft reset ([`vic20`](14-evidence.md#142134-vic20-run-35), [`irq044`](14-evidence.md#142142-irq044-run-35)):
  - VIC0 line 20 appeared with a command and `0x044` bit 0 and with an address and bit 1, the same in all three repetitions of each case. It did not appear with a status latch (bit 0 or 13) or a DMA (bit 11).
  - In a program with a command, an address, a data read, a status latch and a DMA, with the flags acknowledged, `0x044` bits 0, 1, 2, 11 and 12 each set `CS_IRQ` bit 4. None of them raised VIC0 line 20.
  - VIC1 line 22 was never seen.

Related findings: 70, 82, 108, 140, 146, 150, 167. The `CS_IRQ` flags themselves are in [chapter 8](08-sequencer-control-registers.md).

## 3.5 Starting and finishing a run

How the original CPU driver runs a program ([RE-31](14-evidence.md#14331-re-31-cpu-driver-start), [RE-32](14-evidence.md#14332-re-32-cpu-driver-wait-and-classify), [RE-33](14-evidence.md#14333-re-33-cpu-driver-reset), [RE-34](14-evidence.md#14334-re-34-cpu-driver-clear)):

1. It writes the parameter words ([chapter 12, section 12.1](12-cpu-interface.md#121-parameter-words-per-program)) and fills the lists in memory. The sequencer fetches the program and reads the lists from system memory. Its accesses are not coherent with the CPU data cache.
2. It writes the program's bus address to `FMSS_PROG` and `0xff` to `CS_IRQ`, then writes `0xfff5` to `FMSS_CTRL`.
3. It waits for completion by polling the raw VIC1 status bit 22 and reading `CS_IRQ`: bit 0 is END, bit 2 is a fault. It allows a run at most 2 s. On a fault or a timeout it resets the sequencer in this order: it writes `0x30000` to `FMSS_CTRL`, waits 1 ms, runs an idle program, clears `CS_IRQ` and writes 0 to `FMC_CTRL0`.
4. After a clean END the result is `SEQ_RESULT`. After a fault the result is the value that the fault handler left in r6.

A run that does not end within 2 s is reported as a timeout and followed by a sequencer reset.

References: [chapter 14](14-evidence.md) (outcomes), [chapter 13](13-experiments.md) (what was done and why), [chapter 15](15-evidence-interpretation.md) (what the outcomes mean).

Related chapters: [chapter 2](02-architecture.md), [chapter 4](04-sequencer-memory-map.md), [chapter 8](08-sequencer-control-registers.md), [chapter 9](09-nand-controller.md), [chapter 12](12-cpu-interface.md).
