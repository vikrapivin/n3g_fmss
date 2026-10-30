# Chapter 7 — Events, waits and the timeout

Summary: The hardware flags that `wait` waits for, the conditions that `wait2` waits for, the rules of `wait` event 5, the length of the timeout and the timeout timer `C14`. Each event was identified from the status bit that the original programs pair with the corresponding `wait`, and was then mapped on a device. No data sheet was used.

**Two separate number lists.** `wait` and `wait2` both take a number, but the numbers name different things.

- `wait` event 5 is the status-latch event ([chapter 7, section 7.1](#71-wait-events)). It passes when a status latch completes.
- `wait2` condition 5 is an entry of the separate list of `wait2` ([chapter 7, section 7.2](#72-the-conditions-wait2-waits-for)). It is always true and has nothing to do with the status latch.

These documents always write "`wait` event N" and "`wait2` condition N" in full, so that the two lists are not confused.

## 7.1 Wait events

A `wait` blocks the program until a flag of the NAND controller (FMC) or of the ECC engine is set. The flags are sticky: a flag stays set until the program clears it by writing 1 to its bit (write-1-to-clear). A `wait` on a flag that is already set returns immediately.

The original programs clear each flag after the `wait` that used it, and begin each run with all flags clear ([C9](11-nand-operations.md#1119-c9-acknowledge-hygiene)). A flag left set by an earlier operation lets the next `wait` return before the data has moved (observed by the third-party driver).

| event | flag | set by the hardware when | clear with |
|---|---|---|---|
| 0 | `FMC_STATUS` bit 1 (`0x2`) | a command byte written to `FMC_CMD` has been sent to the NAND | `FMC_STATUS = 0x2` |
| 1 | `FMC_STATUS` bit 2 (`0x4`) | the address cycles started with `FMCTRL1 = 1` have been sent | `FMC_STATUS = 0x4` |
| 2 | `FMC_STATUS` bit 20 (`0x100000`) | a DMA transfer between a controller data buffer and memory has finished | `FMC_STATUS = 0x100000` |
| 3 | `FMC_STATUS` bit 3 (`0x8`) | a NAND data transfer (read into a buffer, write from a buffer, parity slice, ID) has finished | `FMC_STATUS = 0x8` |
| 4 | `ECC_DONE` bit 0 | the ECC engine has finished the operation started with `ECC_KICK` | `ECC_DONE = 1` |
| 5 | `FMC_STATUS` bit 23 (`0x800000`) | the NAND status poll started with `FMCTRL1 = 0xca` has delivered a status byte into `FMC_NSTAT` | `FMC_STATUS = 0x800000` |

Events are levels, not one-shot signals.

- A `wait` does not consume its event: `wait` events 0 and 5 each passed twice in a row.
- A `wait` event whose flag a previous program left set passes at once.
- `wait` event 4 passes on any `ECC_DONE` bit other than bit 4. It still passed after bit 0 was acknowledged and the value `0x74` was left over.

Further event numbers (device, [chapter 16, section 16.2](16-device-findings.md#162-settled-on-a-device)):

- `wait` events 6, 7, 8 and 9 are the command, address, DMA and transfer flags of the second register set ([chapter 9, section 9.5](09-nand-controller.md#95-the-second-nand-controller-register-set-0x400)). They are bits 1, 2, 20 and 3 of word `0x448`, the same bit numbers as `wait` events 0, 1, 2 and 3 use in the first set. `wait` event 8 is the DMA-done flag of the second set: a loopback from system memory into a data buffer and back to system memory through the second set passed `wait` event 8 and did not pass `wait` event 2. `wait` event 8 also passed while a second-set DMA was running ([`evsweep3`](14-evidence.md#142167-evsweep3-run-37)), and after a page read of CE0 through the first register set, whose DMA flags had been acknowledged ([`w2state`](14-evidence.md#142151-w2state-run-36); why is open).
- `wait` event 11 passes in every state tried, including while a chip erases, so it is not the chips' ready line.
- `wait` event 15 passed only in early tests where word `0x044` had been left non-zero. It did not pass with every `0x044` / `0x844` enable written on and a command flag pending ([`evsweep2`](14-evidence.md#142137-evsweep2-run-35)), nor with each `0x044` bit 0, 1, 2, 11 and 12 set and its own flag pending ([`irqop`, device run 37](14-evidence.md#142170-irqop-run-37)). So it is not "an enabled source is pending"; what it waits for is open.
- `wait` events 10 and 12 to 14 never passed, and neither did event numbers above 15. This held also during an erase, a page program, a page read, an ECC encode and decode, a DMA, a parity-buffer DMA, a second-set DMA, a meta encode and decode and a chip data write ([`evsweep2`](14-evidence.md#142137-evsweep2-run-35), [`evsweep3`](14-evidence.md#142167-evsweep3-run-37)).

### 7.1.1 Rules for `wait` event 5

`wait` event 5 is the flag `FMC_STATUS` bit 23, read as a level. A status latch (`FMCTRL1 = 0xca`) sets it under these rules (device: [`ev5ready`](14-evidence.md#142121-ev5ready-run-30), [`ev5arm`](14-evidence.md#142124-ev5arm-run-31), [`ev5two`](14-evidence.md#142127-ev5two-run-32)):

- **Arm.** A status latch sets bit 23 only while `wait` event 5 is armed, and each latch uses up the arm.
  - With an idle chip, the latch sets the bit at once.
  - With a busy chip, the bit is set when the chip becomes ready, and the `wait` lasts until then. The busy status byte in `FMC_NSTAT` is `0x81` during an erase or a program and `0x80` during the busy time of a page read.
  - A latch made with the wrong select (`FMC_DATA_SEL = 0x100`) also uses up the arm.
  - Every latch updates `FMC_NSTAT`.
- **What arms it.** These re-arm it: an `FMCTRL1` operation (the set-up word, `0x100e0`, a transfer from the chip with `0x32`, a DMA with `0x2e0`), address cycles (`FMCTRL1 = 1`), and the polled chip becoming ready. These do not re-arm it: `FMCTRL1 = 0x20`, the status-clear word (`FMC_STATUS` = `0x0ff00ffe`), a CE deselect followed by a reselect, and an ECC operation.
- **Consequence for repeated latches.** Suppose two latches have only `FMCTRL1 = 0x20` and a fresh `70h` command (or a chip reset `FFh`) between them. Then one `wait` event 5 passes and the other times out. With only a fresh `70h` between two latches, the second `wait` event 5 timed out 40 of 40 times ([`latchrep`](14-evidence.md#142141-latchrep-run-35)).
- **Two chips.** Two chips were programmed and then polled.
  - Polling the first chip (which waits for ready) and then the second passed both `wait` event 5.
  - Polling the second chip first (the one that finishes later) and then the first timed out on the first chip. The first chip's ready transition had already happened before the earlier latch.
- **Level.** If bit 23 is cleared after a `wait` event 5, a second `wait` event 5 times out. If bit 23 is left set, the second `wait` event 5 passes on the old flag. A second latch with no controller operation in between does not set bit 23 again.
- **After a latch.** The controller is in status mode and refuses the next command byte until `FMCTRL1 = 0x20`: after the command write, `FMC_STATUS` bit 1 (command sent) stays clear and bit 26 is set instead; the original programs write `0x20` before `00h`. A latch after `90h` instead of `70h` sets no bit 23.
- **Timing.** A status poll during an erase passed `wait` event 5 once the chip was ready (about 155 us in), with the status byte going from `0x81` to `0xc0`; an idle chip passes at once.
- **Reads.** A sequencer read of `FMC_STATUS` or `FMC_NSTAT` does not consume an event.
- **Repeatability.** 160 single latches (with and without a fresh arm before them, with and without `0x044` / `0x844` written) all passed, with bit 23 set and bit 24 clear ([`latchrep`](14-evidence.md#142141-latchrep-run-35)). A `wait` event 5 placed straight after the pair "`70h`, latch" timed out once, although bit 23 read set afterwards. Any instruction or wait between the latch and the `wait` event 5 made it pass (open).

**The status latch needs `FMC_DATA_SEL = 1` (device).** After a write, with `FMC_DATA_SEL = 0x100` left from the data transfer, the latch returned `0x70` (the last command byte) instead of the chip's status. Bit 6 of `0x70` reads as "ready". The poll therefore ended while the chip was still programming, and the program command for the next page was ignored (those pages read back erased). With `FMC_DATA_SEL = 1` ([C7](11-nand-operations.md#1117-c7-ready-procedure-status-poll)) the status byte is the chip's. The status bytes are `0xc0` when ready and `0x81` when busy.

How the original programs use `wait` event 5:

- They test `FMC_NSTAT` bit 6 first and wait on `wait` event 5 only while the chip is busy. Of the 19 uses, 13 (all outside ERASE) have this five-instruction shape; the 6 uses in ERASE are staged differently ([RE-7](14-evidence.md#1437-re-7-wait-event-5-shape), [RE-8](14-evidence.md#1438-re-8-erase-ready-waits)).
- The READ and WRITE programs do not test `FMC_NSTAT` again after the wait. The ready procedure with a re-test and a bounded re-latch is described in [C7](11-nand-operations.md#1117-c7-ready-procedure-status-poll).
- The third-party driver measured bit 23 set after every one of its status polls, whether the chip was ready or not (128 of 128, 3p).

### 7.1.2 Timeouts

Every original program writes `FMSS_C4C` (fault vector) and `FMSS_C58` (value 4) in its prologue. A `wait` whose flag is not set times out after about 4.87 ms with the start word `0xfff5` that the original CPU drivers use, whatever `C58` holds, and raises `CS_IRQ` bit 2 (device, [chapter 16, section 16.2](16-device-findings.md#162-settled-on-a-device)). Without start-word bit 2 there is no timeout ([chapter 8, section 8.1.1](08-sequencer-control-registers.md#811-start-word-and-timeout-fmss_ctrl-device)).

`C58` bit 2 decides where the program goes on: if set (the prologue's 4), at the fault vector; if clear, at the instruction after the `wait`. The original erase program relies on this when it lowers `C58` to 0 around its waits, acknowledges the timeout itself (`CS_IRQ = 4`) and tests again. The other bits of `C58` are in [chapter 8, section 8.1.15](08-sequencer-control-registers.md#8115-fmss_c58-0xc58-fault-routing). `C10`, `C3C`, `C48` and `C54` do not change the timeout.

## 7.2 The conditions `wait2` waits for

`wait` and `wait2` (opcode `09`) both take a number N in their register field. Each waits until the hardware condition with that number is true, or faults after about 4.87 ms.

- For `wait`, the numbers are the events of [chapter 7, section 7.1](#71-wait-events): `wait` event 0 is "command sent", `wait` event 1 is "address sent", and so on.
- For `wait2`, the numbers are the CPU's interrupt sources. **`wait2` condition N is true while interrupt source N is raised in the raw status of the CPU's interrupt controllers**: VIC0 line N for N = 0..31 and VIC1 line N - 32 for N = 32..63 (device, [`w2vic`](14-evidence.md#142171-w2vic-run-38)). Numbers 64..255 never pass.

How this was established: just before each `wait2 N` (N = 0..63) the program read both raw status words (`0x38e00008` for VIC0, `0x38e01008` for VIC1). In four states, idle and with three different interrupt sources raised, the pass or timeout of all 64 conditions matched the raw line, 64 of 64 each time ([`w2vic`](14-evidence.md#142171-w2vic-run-38)).

Conditions seen true:

| `wait2` condition N | interrupt line | when it was raised | seen |
|---|---|---|---|
| 5 | VIC0 line 5 | in every state tried (a source outside the FMSS engine) | device |
| 20 | VIC0 line 20, the NAND controller's line | while a NAND controller flag is pending and enabled in `0x044` (command flag with bit 0, DMA flag with bit 11, [chapter 3, section 3.4](03-cpu-memory-map.md#34-cpu-visible-interrupt-lines)) | device |
| 33 | VIC1 line 1 | raised while the chip was idle in [`op09w33`](14-evidence.md#142123-op09w33-run-30) and during an ECC operation and a DMA in [`finale`](14-evidence.md#142130-finale-run-34); not raised in [`op09nand`](14-evidence.md#142116-op09nand-run-29), [`evsweep2`](14-evidence.md#142137-evsweep2-run-35), [`w2state`](14-evidence.md#142151-w2state-run-36) or [`w2vic`](14-evidence.md#142171-w2vic-run-38) (a source outside the FMSS engine) | device |
| 43 | VIC1 line 11, the ECC engine's line | while an `ECC_DONE` flag is pending and enabled in `0x844` (bit 0) | device |

All other numbers never passed in any state tried, also while a chip erased, programmed or read, during ECC operations and DMAs, and with the second register set's flags pending ([`w2state`](14-evidence.md#142151-w2state-run-36), [`evsweep2`](14-evidence.md#142137-evsweep2-run-35)).

So `wait2` condition 5 is unrelated to `wait` event 5 (the status latch), despite the same number. Lines 5 and 33 belong to devices outside the FMSS engine. This explains why `wait2` condition 33 passed while the chip was idle in [`op09w33`](14-evidence.md#142123-op09w33-run-30) and during an ECC operation and a DMA in [`finale`](14-evidence.md#142130-finale-run-34); it did not pass in [`op09nand`](14-evidence.md#142116-op09nand-run-29) (idle and during an erase), [`evsweep2`](14-evidence.md#142137-evsweep2-run-35), [`ctrlwords`](14-evidence.md#142139-ctrlwords-run-35), [`w2state`](14-evidence.md#142151-w2state-run-36), [`blk100b`](14-evidence.md#142168-blk100b-run-37) or [`w2vic`](14-evidence.md#142171-w2vic-run-38). Which devices raise lines 5 and 33 is not determined here.

## 7.3 tmwait and evjmp

The descriptions of both opcodes are in [chapter 6, opcode `15`](06-opcode-reference.md#15-tmwait) (`tmwait`) and [chapter 6, opcode `16`](06-opcode-reference.md#16-evjmp) (`evjmp`). `tmwait` waits for the timer `C14` to reach 0 (about 4.85 ms). `evjmp` waits for any event flag and then continues at the fault vector `C4C`.

## 7.4 Timer `C14`

`FMSS_C14` (`0xc14`) is the timeout timer, an 8-bit down-counter ([chapter 8, section 8.1.6](08-sequencer-control-registers.md#816-fmss_c14-0xc14-the-timeout-timer)). The timeout lasts (start-word bits 15:8) + 1 steps, and the length of one step is set by start-word bits 7:4 ([chapter 8, section 8.1.1](08-sequencer-control-registers.md#811-start-word-and-timeout-fmss_ctrl-device)). Device behaviour:

- A `wait` that has to wait loads the start word's bits 15:8 (`0xff` with `0xfff5`) and counts down (`0xf6` was read in the middle of a wait). With one of bits 8..15 cleared in the start word, `C14` read `0xfe`, `0xfd`, `0xfb`, `0xf7`, `0xef`, `0xdf`, `0xbf`, `0x7f` after a run with waits ([`startbits`](14-evidence.md#142132-startbits-run-35)).
- After a timeout the timer reads its reload value; the end of a program freezes it; CPU and program writes do not change it.
- Opcode `15` waits for it: from a non-zero value it reloads and waits until 0 (about 4.85 ms, leaving 0); from 0 it reloads at once. Two `tmwait` in one program made the whole program run 4841, 18 and 3250 us in three runs with different values written to `C10`, `C48` and `C54` before it ([`ctrlwords`](14-evidence.md#142139-ctrlwords-run-35); not explained).
- The write of `0xff` that the erase program makes after a timeout therefore has no visible effect.

Findings 48 and 51 ([chapter 18](18-ledger.md)). The device rows "timer `C14`" and "`C08`" are in [chapter 16, section 16.2](16-device-findings.md#162-settled-on-a-device).

## 7.5 Timeout length

The timeout length is (bits 15:8 + 1) x 2^s bus cycles, where s depends on n = bits 7:4 of the start word. The table was measured with 24 start words, all within the measurement step ([`tmofield`](14-evidence.md#142107-tmofield-run-28), [`tmobits`](14-evidence.md#142110-tmobits-run-29), [`tmobits2`](14-evidence.md#142120-tmobits2-run-30)):

| n (bits 7:4) | s | example (start word: time to the fault) |
|---|---|---|
| 0 | at once | `0xff05`: 0 to 2 us |
| 1 | 3 | `0xff15`: 18 us |
| 2 | 5 | `0xff25`: 72 us |
| 3 | 7 | `0xff35`: 296 us |
| 4 | 9 | `0xff45`: 1218 us |
| 5 | 10 | `0xff55`: 2432 us |
| 6 to 15 | 11 | `0xfff5`: 4868 us |

More examples:

- n = 0: `0x0005`, `0x0105` to `0xff05` and `0x0f05` all fault after 0 to 2 us.
- n = 1: `0x0f15` after 2 us. n = 2: `0x0f25` after 0 us. n = 4: `0x0f45` after 72 us. n = 5: `0x0f55` after 152 us.
- n = 6 to 15: `0xff65`, `0xff75`, `0xff85`, `0xffc5` and `0xfff5` after 4868 us; `0xffb5` after 4.85 ms.
- n = 7 and 8 with bits 15:8 = `0x0f`: `0x0f75` after 310 us and `0x0f85` after 308 us.
- n = 15 with other bits 15:8: `0x00f5` after 16 us, `0x01f5` 46 us, `0x03f5` 72 us, `0x07f5` 152 us, `0x0ff5` 310 us, `0x3ff5` 1218 us and `0x7ff5` 2430 us.

The steps are measured, not derived from a rule: n = 5 breaks the factor-4 pattern of n = 1 to 4. The bus clock of about 108 MHz is inferred.

References: [chapter 14](14-evidence.md) (outcomes), [chapter 13](13-experiments.md) (what was done and why), [chapter 15](15-evidence-interpretation.md) (what the outcomes mean).

Related chapters: [chapter 6](06-opcode-reference.md), [chapter 9](09-nand-controller.md), [chapter 10](10-ecc-engine.md), [chapter 16](16-device-findings.md), [chapter 17](17-open-questions.md).
