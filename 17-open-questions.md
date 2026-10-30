# Chapter 17 — Open questions

Summary: Every point that these documents marks as open, not measured, unexplained, unknown or only inferred where the text says it is not settled. Each entry gives the status as recorded, the chapter with the details and the finding numbers of [chapter 18](18-ledger.md). Nothing here is a new finding; the entries only collect what the other chapters state, and each open point is listed once.

## 17.1 Instruction set and sequencer behaviour

| # | open point | status as recorded | details | findings |
|---|---|---|---|---|
| 1 | Which devices raise interrupt lines 5 and 33, the two `wait2` conditions seen true without FMSS activity (`wait2` condition N is interrupt source N, [chapter 7, section 7.2](07-events-and-waits.md#72-the-conditions-wait2-waits-for)) | not determined | ([chapter 7, section 7.2](07-events-and-waits.md#72-the-conditions-wait2-waits-for)) | 120, 174 |
| 2 | Why there are two nop instructions (`08` nop, `ff` nop2) that behave the same way | device: both have no visible effect in every operand form and take the same time | [chapter 6](06-opcode-reference.md) | 47, 89 |
| 3 | `C58` bit 1: with bit 1 alone no `end` was diverted in [`c58b1`](14-evidence.md#142136-c58b1-run-35) and in [`c58b2`](14-evidence.md#142155-c58b2-run-36), which also ran three programs in a row with the device job's clean-up switched off; with bits 1 and 2 (`C58` = 6) the first `end` was diverted once; in [`oneshot`](14-evidence.md#14236-oneshot-run-8), `C58` = 2 with `C4C` written by the program diverted one `end` | condition open | [chapter 8, section 8.1.15](08-sequencer-control-registers.md#8115-fmss_c58-0xc58-fault-routing) | 46, 162 |
| 4 | `wait` events 10 and 12 to 15 never passed in any state tried (`wait` event 15 also not with an enabled interrupt source pending); `wait` event 11 passes in every state (what it is remains open; it is not the chips' ready line, finding 57); `wait` event 8 passed after a page read through the first register set | open | [chapter 7](07-events-and-waits.md), [chapter 16](16-device-findings.md) | 43, 57, 62, 75, 152, 163 |
| 5 | `C08`: the full set of state codes | "full code set open" | [chapter 8, section 8.1.3](08-sequencer-control-registers.md#813-cs_status-0xc08) | 67, 157 |
| 6 | `FMSS_C50` bit 8, seen set once, after a soft reset (`C50` read `0xffff1900`); not seen after a soft reset in [`c50fault`](14-evidence.md#142138-c50fault-run-35) and [`c50b8`](14-evidence.md#142152-c50b8-run-36) | meaning open | [chapter 8, section 8.1.13](08-sequencer-control-registers.md#8113-fmss_c50-0xc50-pending-events) | 85 |
| 7 | `FMSS_C10`, `C48`, `C54` can be written by the CPU, but no effect of the written value was found | no effect found | [chapter 8](08-sequencer-control-registers.md) | 55, 64 |
| 8 | What start-word bit 6 does: `0xfff5` (the original operating systems) and `0xffb5` (the 4G/5G bootloader) differ only in bit 6; no difference was found in any program tried (timeout, interrupt line, loops, waits, illegal opcodes, `evjmp`, program fetch, `ldi` / `sti` / `pld`, DMA, ECC, reserved-word access); access to other NAND regions under the two words is not measured; reads, erases and programs in the user area and in the blocks above it, on both chips, showed no difference either | open | [chapter 8, section 8.1.1](08-sequencer-control-registers.md#811-start-word-and-timeout-fmss_ctrl-device) | 54, 76, 153 |
| 9 | Timeout length rule: the steps were measured, not derived (n, the value of start-word bits 7:4: n = 5 breaks the pattern of n = 1 to 4); the bus clock (about 108 MHz), which the timeout length and the timing word of the original CPU driver depend on, is inferred and otherwise not established | device (steps); clock inferred | [chapter 7, section 7.5](07-events-and-waits.md#75-timeout-length), [chapter 12, section 12.3](12-cpu-interface.md#123-timing-word) | 31, 107, 111 |
| 10 | Two `tmwait` in one program made the whole program run 4841, 18 and 3250 us with three different values written to `C10`, `C48` and `C54` before it ([`ctrlwords`](14-evidence.md#142139-ctrlwords-run-35)), while four `tmwait` in one program alternated regularly between a full wait and no wait in five repetitions ([`tmwait2`](14-evidence.md#142157-tmwait2-run-36)) | not explained | [chapter 7, section 7.3](07-events-and-waits.md#73-tmwait-and-evjmp) | 47, 151 |

## 17.2 Events and the status latch

| # | open point | status as recorded | details | findings |
|---|---|---|---|---|
| 11 | A `wait` event 5 placed straight after a `70h` command and a status latch (`FMCTRL1` = `0xca`) timed out once although `FMC_STATUS` bit 23 read set afterwards; the status latch's `wait` event 5 timed out in one of two identical runs; in [`latchrep`](14-evidence.md#142141-latchrep-run-35), 160 of 160 latches passed | intermittent, cause open | [chapter 7](07-events-and-waits.md), [chapter 16](16-device-findings.md) | 63, 66, 87, 156 |
| 12 | `FMC_STATUS` bit 24 set instead of bit 23 after a status latch (so `wait` event 5 never came): seen in [`irqsrc`](14-evidence.md#142101-irqsrc-run-27) in both status-latch programs with `0x044` or `0x844` written and in one program of [`op16`](14-evidence.md#14289-op16-run-23) with no enable, not in the fifteen programs of [`ev5en`](14-evidence.md#142109-ev5en-run-28) and not in the 160 latches of [`latchrep`](14-evidence.md#142141-latchrep-run-35) | cause open | ([chapter 9, section 9.4](09-nand-controller.md#94-fmc_status-bits)), [chapter 16](16-device-findings.md) | 104, 110, 121, 156 |
| 13 | One `wait` event 3 with CE0 selected passed in one program and timed out in three other programs; it passed 40 of 40 times in [`latchrep`](14-evidence.md#142141-latchrep-run-35) | unresolved | [chapter 16](16-device-findings.md) | 63, 156 |

## 17.3 Interrupts

| # | open point | status as recorded | details | findings |
|---|---|---|---|---|
| 14 | The enable outside the sequencer block that the original firmware sets, so that VIC1 raw bit 22 (a CPU interrupt line) follows END (it never set in the device tests) | open | ([chapter 2, section 2.3](02-architecture.md#23-end-and-fault)), [chapter 3](03-cpu-memory-map.md) | 76, 102, 140, 146 |
| 15 | The source of `CS_IRQ` bit 5 | not identified | [chapter 8](08-sequencer-control-registers.md), [chapter 16](16-device-findings.md) | 65, 76, 102 |

## 17.4 NAND controller registers

| # | open point | status as recorded | details | findings |
|---|---|---|---|---|
| 16 | `FMC_CTRL0` bit 9 (set in the original READID's control word) and bit 11 (set in every program except READID): no effect found on a page read or a READ ID | meaning unknown | ([chapter 9, section 9.1](09-nand-controller.md#91-fmc_ctrl0-0x000-nand-controller-configuration-not-a-sequencer-control)) | 11, 158 |
| 17 | `FMC_DMA_CTRL` (written 7 before the last DMA of a run): bits 0 and 3 alone deliver correct data, bits 1 and 2 break every DMA after the first and read back with bit 6 or 7 added, bits 4..7 break every DMA; what the bits mean | meaning open | ([chapter 9, section 9.3](09-nand-controller.md#93-other-nand-controller-registers)), [chapter 11](11-nand-operations.md) ([R7](11-nand-operations.md#1127-r7-dma-addressing)) | 24, 158 |
| 18 | `FMC_STATUS` bits 12, 13, 18, 19: set at power-on and after a soft reset, not cleared by writing 1; a 16-byte data read clears 13 and 19, a 512-byte data read clears all four, a DMA into a buffer, a status latch and the set-up word set 13 and 19 again | meaning open | ([chapter 9, section 9.4](09-nand-controller.md#94-fmc_status-bits)) | 78, 159 |
| 19 | What the FIFO port `FMC_FIFO` (`0x080`) reads: it returns the bytes of a READ ID, returned 0 for data buffers filled by DMA, and 128 writes to it changed 26 words of data buffer 0; `FMC_28` is its read position | open | [chapter 1, section 1.5](01-overview.md#15-named-parts-of-the-data-path), [chapter 9, section 9.3](09-nand-controller.md#93-other-nand-controller-registers) | 60, 145, 155, 164 |
| 20 | The block at `0x100..0x1fc` (writable words `0x1b8..0x1d8`): setting any writable word changed neither NAND bus timing nor the time of DMAs, ECC operations, a chip read or a timeout | meaning open | [chapter 3](03-cpu-memory-map.md), [chapter 9](09-nand-controller.md), [chapter 16](16-device-findings.md) | 74, 145 |
| 21 | The words `0x3a8..0x3b4`: in [`access`](14-evidence.md#142105-access-run-27) the sequencer's reads returned the values the CPU had written; in [`blk100b`](14-evidence.md#142168-blk100b-run-37) the sequencer read `0xffffffff` (also after its own writes) and the CPU read 0 | unexplained | ([chapter 3, section 3.2](03-cpu-memory-map.md#32-memory-map-the-register-window)) | 103 |
| 22 | ECC engine words `0x990..0x99c` (after an encode: a 16-bit value that depends on the data with the Reed-Solomon code, 0 with BCH), which is no CRC-16 of the 512 data bytes (every polynomial, both bit orders and both byte orders of the result checked against the measured values) and `0x800` (an identification word, read `0x20060808`, "perhaps a date") | meaning open / perhaps | [chapter 10, section 10.13](10-ecc-engine.md#1013-registers) | 56, 103, 149 |
| 23 | No chip answers on channel 1 of this board; "channel 0 only" is inferred from the one board | inferred from one board | ([chapter 9, section 9.5](09-nand-controller.md#95-the-second-nand-controller-register-set-0x400)) | 73, 93, 100 |

## 17.5 ECC engine

| # | open point | status as recorded | details | findings |
|---|---|---|---|---|
| 24 | What generates the fixed sequence of the modes without a code (61 distinct values, no short linear-feedback shift register; not a multiple of the BCH generator, not a 16-bit Galois LFSR), and which event restarts it (none of the ten actions tried between encodes in [`seqrestart`](14-evidence.md#142146-seqrestart-run-35) restarts it; the first such encode of a device run gave value 1 in [`eccmode4`](14-evidence.md#142129-eccmode4-run-33) and in [`eccregs`](14-evidence.md#142145-eccregs-run-35), value 14 in [`finale`](14-evidence.md#142130-finale-run-34), and at value 2 in [`seqfirst`](14-evidence.md#142150-seqfirst-run-36) after a power cycle | open (bounded negative searches) | ([chapter 10, section 10.12](10-ecc-engine.md#1012-modes-with-bit-8-clear-no-code)) | 137, 144 |
| 25 | The 2 extra slot bytes with `ECC_MODE` bit 1 and RS-8 (one sample) | one sample | ([chapter 10, section 10.5](10-ecc-engine.md#105-reed-solomon-8-parity-symbols-4-bytes-per-sector)) | 136 |
| 26 | `ECC_DONE` bit 5 with `ECC_MODE` bits 21, 22, 23 alone: clear in [`done5`](14-evidence.md#142147-done5-run-35) and [`done5b`](14-evidence.md#142159-done5b-run-37), set in [`eccmode2`](14-evidence.md#142119-eccmode2-run-30) | why `eccmode2` differs is open | ([chapter 10, section 10.11](10-ecc-engine.md#1011-ecc_done-and-ecc_result)) | 139 |
| 27 | What an encode writes to `ECC_RESULT` when its code differs from the last decode's: a Reed-Solomon encode after a BCH decode set 0 in [`rsresult`](14-evidence.md#142160-rsresult-run-37) or `0x20000` in [`eccmode2`](14-evidence.md#142119-eccmode2-run-30); a BCH encode after a Reed-Solomon decode set `0x30000` | open | ([chapter 10, section 10.11](10-ecc-engine.md#1011-ecc_done-and-ecc_result)) | 139, 166 |
| 28 | The reading of the 8-symbol Reed-Solomon form as an older controller's "4-bit ECC per 512 bytes" | "Interpretation (not measured)" | [chapter 15](15-evidence-interpretation.md) | 134 |

## 17.6 Original programs and the CPU interface

| # | open point | status as recorded | details | findings |
|---|---|---|---|---|
| 29 | Why the page word discards the verdicts of earlier sectors when `FMC_STATUS` bit 27 stayed clear ([RD8](11-nand-operations.md#1148-rd8-page-result)); a plausible reason is that a blank sector may decode as uncorrectable (settled for erased sectors by item 4 of [chapter 16, section 16.1](16-device-findings.md#161-risks-and-their-status)) | "is not known" | [chapter 11](11-nand-operations.md) ([RD8](11-nand-operations.md#1148-rd8-page-result)), [chapter 16](16-device-findings.md) | 22, 117 |
| 30 | Why `FMC_CTRL0` bit 24 is set and cleared where it is ([R7](11-nand-operations.md#1127-r7-dma-addressing)); set for a whole page read it changed nothing | reason unknown | [chapter 11](11-nand-operations.md) | 24, 81, 158 |
| 31 | RAWREAD loads `FMC_DATA_SEL` = `0x102` in all steps; used as the drain select of a page read it delivered wrong data | observed; meaning open | [chapter 18](18-ledger.md) | 34, 158 |
| 32 | The reading of the two program times (about 478 and 728..873 us per run) as the chip's lower and upper pages of a cell pair | inferred | [chapter 11](11-nand-operations.md) | 131 |
| 33 | Which row address the original CPU driver writes to the parameter word `0xd00`, which the `78h` programs of release 1.1.3 read | open | [chapter 4, section 4.4](04-sequencer-memory-map.md#44-parameter-ram), [chapter 11](11-nand-operations.md) ([WR9](11-nand-operations.md#1159-wr9-variants)) | 37 |

## 17.7 Soft reset versus power cycle

| # | open point | status as recorded | details | findings |
|---|---|---|---|---|
| 34 | After a soft reset of every writable word, status words (`FMC_STATUS` bits 17:14, `0x050`, `0x448` / `0x450`, ECC-area words `0x980`, `0x984`, `0x9a4`, `C50` bit 8) and the timer `C14` still differ from a power cycle | recorded as differing; behaviour matched | [chapter 16](16-device-findings.md) | 78, 80 |
| 35 | Why the device stopped answering during the first version of `irqop` ([`irqop`, device run 36](14-evidence.md#142158-irqop-run-36); programs that left a NAND controller flag pending with its `0x044` enable set when they ended); the version that acknowledged every flag and cleared the enables ran without fault ([`irqop`, device run 37](14-evidence.md#142170-irqop-run-37)) | not explained | [chapter 13](13-experiments.md) | 167 |

References: [chapter 14](14-evidence.md) (outcomes), [chapter 13](13-experiments.md) (what was done and why), [chapter 15](15-evidence-interpretation.md) (what the outcomes mean).

Related chapters: [chapter 16](16-device-findings.md), [chapter 18](18-ledger.md), [chapter 15](15-evidence-interpretation.md).
