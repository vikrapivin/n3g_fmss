# Chapter 11 — NAND operations

Summary: How the original firmware performs each NAND operation with the engine, with the device measurements that confirm or complement it. The chapter covers the common behaviour (including how a chip and a page are selected, section 11.1.14, with a worked example of a read, an erase and a write of chip CE1, block 3000, page 5), the ECC and data path shared by READ and PROGRAM, RESET and READ ID, page read, page program, block erase, and the fault handler. Each behaviour carries a label (`C1`..`C13`, `R1`..`R7`, `ID1`..`ID5`, `RD1`..`RD11`, `WR1`..`WR9`, `ER1`..`ER4`, `FL1`..`FL3`) defined in this chapter; other chapters use the labels as cross-references. The behaviours are those of the nine original programs ([RE-2](14-evidence.md#1432-re-2-program-table)) and of the original CPU driver.

## 11.1 Common behaviour

### 11.1.1 C1: Placement

Each original program is a flat array of instructions starting at offset 0, 8-byte aligned ([RE-1](14-evidence.md#1431-re-1-nine-sequencer-programs-in-the-ipod-os-image)).

### 11.1.2 C2: Prologue

Before it accesses a chip, every original program writes the following registers (observed in all nine programs, [RE-5](14-evidence.md#1435-re-5-prologue-values)):

- `CS_IRQ = 0xff`.
- `FMSS_C10 = 1`.
- `FMSS_C58 = 4`.
- `FMSS_C4C` = the offset of the program's fault handler ([FL1](#1171-fl1-handler)).
- `FMCTRL1 = 0x0ff3f8e0`.
- `FMC_STATUS = 0x0ff00ffe`.

The third-party driver writes the last two (`FMCTRL1` and `FMC_STATUS`) before every operation.

### 11.1.3 C3: Control word

After the prologue, the programs write `FMC_CTRL0 = 0x801 | timing` (the timing word is in [chapter 12, section 12.3](12-cpu-interface.md#123-timing-word)). Later chip selection changes only bits 8:1 of `FMC_CTRL0`: the program reads the register, modifies those bits and writes it back ([RE-9](14-evidence.md#1439-re-9-control-word-and-chip-enable-changes)). The chip-enable bit is `1 << (CE index + 1)`. At any time exactly one chip-enable bit is set, except when the reset command is sent to all chips at once ([ID2](#1132-id2-reset)).

### 11.1.4 C4: CE index

The chip-enable index CE (0..7) of list entry `i` is `CE = cemap[cel[i]]`: `cel[i]` is the logical chip of entry `i`, and `cemap` is a table that maps logical chips to CE indices ([RE-29](14-evidence.md#14329-re-29-chip-addressing-in-the-programs), [chapter 12, section 12.1](12-cpu-interface.md#121-parameter-words-per-program)).

### 11.1.5 C5: One command cycle

A command cycle has three steps: write the command byte to `FMC_CMD`, `wait` event 0, write `FMC_STATUS` = `0x2` (acknowledging the event).

### 11.1.6 C6: Address cycles

The address cycles of one command are sent as follows:

1. Set `FMC_ANUM`, `FMC_ADDR0`, and `FMC_ADDR1` (only for five cycles).
2. Write `FMCTRL1` = 1.
3. `wait` event 1.
4. Write `FMC_STATUS` = `0x4`.

The formats:

| use | ANUM | ADDR0 | ADDR1 |
|---|---|---|---|
| page read / program, five cycles: column 0 (two bytes), row (three bytes) | 4 | `row << 16` | `row >> 16` |
| block erase, three row cycles | 2 | `row` | not written |
| enhanced status `78h`, three row cycles | 2 | `row` | not written |
| READ ID, one cycle | 0 | 0 | not written |

### 11.1.7 C7: Ready procedure (status poll)

To learn whether a chip has finished its array operation and to fetch its FAIL bit, the original programs do the following:

1. Select its CE and write `FMC_DATA_SEL` = 1.
2. Send command `70h` ([C5](#1115-c5-one-command-cycle)) and write `FMCTRL1` = `0xca`.
3. Read `FMC_NSTAT` and test bit 6 (`0x40`). If the bit is clear, `wait` event 5.
4. Acknowledge `FMC_STATUS` = `0x800000`.
5. Read the FAIL bit as `FMC_NSTAT` & 1 (the `78h` variant tests `FMC_NSTAT` & 3), then write `FMCTRL1` = `0x20`.

The chip is left in status-output mode. The original programs use one `wait` event 5 and do not read `FMC_NSTAT` again after it. This holds for the 13 uses of `wait` event 5 outside ERASE, which share one five-instruction shape. The 6 uses in ERASE arrange the wait differently, with `C58` lowered ([RE-7](14-evidence.md#1437-re-7-wait-event-5-shape), [RE-8](14-evidence.md#1438-re-8-erase-ready-waits)).

The third-party driver does not rely on `wait` event 5. It polls `FMC_NSTAT` repeatedly, up to 10 000 reads per latch and up to 200 ms (3p). How `wait` event 5 is armed and consumed is described in [chapter 7, section 7.1](07-events-and-waits.md#71-wait-events).

The ready wait after a page read or program takes 25 us to 2 ms. A block erase takes about 1.1 ms for a block holding data (device, [`eraseseq`](14-evidence.md#142117-eraseseq-run-29)), the longest wait. In the device checks of the write sequence, the poll ran with `FMC_DATA_SEL` = 1 and latched the status byte repeatedly without `wait` events ([`writeseq2`](14-evidence.md#142125-writeseq2-run-31)).

### 11.1.8 C8: Data transfer

A data transfer has these steps:

1. Set `FMC_DNUM` and `FMC_DATA_SEL`. For a chunk transfer also set `FMC_24` (chunk read) or `FMC_28` (chunk write), the byte offset of the slot in the parity buffer.
2. Write the `FMCTRL1` code.
3. `wait` event 3.
4. Write `FMC_STATUS` = `0x8`.

When a transfer is meant to overlap another operation, the program starts the other operation first and only then waits for the transfer's event ([chapter 11, section 11.2](#112-ecc-and-data-path-shared-by-read-and-program)).

### 11.1.9 C9: Acknowledge hygiene

In the original programs, every event flag in the table of [chapter 7, section 7.1](07-events-and-waits.md#71-wait-events) is acknowledged (cleared) after the program's wait for it. Each program begins with all flags clear (the all-flags clear of [C2](#1112-c2-prologue)). A stale set flag makes the next wait return before the data has moved and corrupts the transfer (observed by the third-party driver: "stale bits let the stage waits pass before the data has moved"). For the ECC engine, the program writes `ECC_DONE` = `0x1ff` before every `ECC_KICK` and `ECC_DONE` = `1` after every `wait` event 4 ([RE-12](14-evidence.md#14312-re-12-ecc-protocol-in-read)).

### 11.1.10 C10: End state

At every normal END of the original programs all CE bits in `FMC_CTRL0` are cleared, bit 24 is cleared, and every flag is acknowledged.

### 11.1.11 C11: Absent chips

READID is the only program that may be given CE indices of chips that are not fitted. It waits only for `wait` events 0, 1 and 3 and never for `wait` event 5 ([RE-24](14-evidence.md#14324-re-24-readid-sequence)). In the other programs, a chip that does not answer gives no status byte. `wait` event 5 then never occurs, and the sequencer faults when the wait times out.

### 11.1.12 C12: One FMCTRL1 operation at a time

`FMCTRL1` holds one pending operation: a second write replaces the first before it has run. Therefore, in the original programs, each `FMCTRL1` code is followed by a wait for that operation's own completion event, and by its acknowledgement, before the next `FMCTRL1` code is written.

Device: a read program wrote `FMCTRL1` = `0x100` (DMA of a data buffer to system memory) and then at once `FMCTRL1` = `0xe2` (NAND read into the other data buffer). The NAND read completed (`wait` event 3), but the DMA-done event (`wait` event 2, `FMC_STATUS` bit 20) never occurred. The sequencer timed out at that `wait` event 2 after the first sector of the page had reached system memory (finding 36).

Two data transfers overlap in time only inside a single `FMCTRL1` code that names both data buffers through `FMC_DATA_SEL` (for example `0x2e4`). ECC operations (`ECC_KICK`) are the exception: the ECC engine is a separate unit, and its operations run while one `FMCTRL1` operation is in flight.

### 11.1.13 C13: Prime after a reset (CPU side)

After `FMSS_CTRL = 0x30000`, the sequencer executes instruction 0 of the first started program only if that instruction is `end` ([`startseq3`](14-evidence.md#14235-startseq3-run-8)). The original CPU driver follows its reset immediately with a start of an idle program of zero bytes ([RE-33](14-evidence.md#14333-re-33-cpu-driver-reset)). The registers `C58`, `C4C` and `C5C` keep their values across the reset. If a previous program left `C58` bit 0 set, every later END runs the fault handler once (device, [`pending`, device run 5](14-evidence.md#14226-pending-run-5), [`pending`, device run 6](14-evidence.md#14230-pending-run-6)).

### 11.1.14 How a chip and a page are selected

This section collects how a run chooses the NAND chip and the page it accesses. The chip is chosen by `FMC_CTRL0`, the page by the row number in the address cycles.

**Which chip.**

- `FMC_CTRL0` bits 8:1 hold one chip-enable bit per chip: bit 1 is CE0, bit 2 is CE1, and so on up to bit 8 for CE7 ([chapter 9, section 9.1](09-nand-controller.md#91-fmc_ctrl0-0x000-nand-controller-configuration-not-a-sequencer-control)).
- Before it talks to a chip, a program writes `FMC_CTRL0` with exactly one of these bits set ([C3](#1113-c3-control-word)). Example control words: `0x43803` selects CE0, `0x43805` CE1 and `0x43809` CE2.
- With several bits set, the same cycles go to all those chips at once. The original programs do this only for the reset command `FFh` ([ID2](#1132-id2-reset)).
- On the unit tested, CE0 to CE3 each answered READ ID.

**Who decides.**

- The original CPU driver builds lists in memory and writes their addresses into parameter RAM ([chapter 12, section 12.1](12-cpu-interface.md#121-parameter-words-per-program)): `P_ROWS` is the row of each page, `P_CELIST` is the logical chip of each page, and `P_CEMAP` points to the table `cemap[8]`.
- The program steps through the lists. For entry `i` it loads the row and the logical chip from system memory with `ldi`, maps the logical chip through the table (`CE = cemap[cel[i]]`; the table is the identity 0..7 in the original CPU driver), and writes the matching `FMC_CTRL0` word by a read-modify-write of bits 8:1 ([C4](#1114-c4-ce-index), [RE-29](14-evidence.md#14329-re-29-chip-addressing-in-the-programs), [RE-9](14-evidence.md#1439-re-9-control-word-and-chip-enable-changes)).
- READID takes the entries of `P_CELIST` directly as CE indices, with no map, and uses `P_ROWS` as the count of chips ([ID4](#1134-id4-per-ce)).
- The erase list holds the first row of each block.

**Which page.**

- The row number is the block number times the pages per block, plus the page within the block. In the erase list the row is that of the first page of the block.
- The device experiments used 128 pages per block on the unit tested, with scratch blocks from block 3000 on ([chapter 13, section 13.1.2](13-experiments.md#1312-nand-scratch-blocks)).

**How the row reaches the chip.**

- The address cycles follow the table of [C6](#1116-c6-address-cycles). `FMC_ANUM` is the number of cycles minus 1. A page read or program sends 5 cycles: column 0 in two bytes, then the row in three bytes, with `FMC_ADDR0` = `row << 16` and `FMC_ADDR1` = `row >> 16`. Erase and the `78h` status send 3 row cycles with `FMC_ADDR0` = row. READ ID sends one cycle with address 0.
- The program then writes `FMCTRL1` = 1, waits for `wait` event 1 and acknowledges `FMC_STATUS` = `0x4`.
- The controller sends the bytes of `FMC_ADDR0` from the least significant byte upward, then the byte of `FMC_ADDR1` (inferred from the formats that work on the device).
- An erase sent with a fourth address cycle (`FMC_ANUM` = 3) still erased the block: the chip ignores the extra cycle (device, [`pageorder`](14-evidence.md#142122-pageorder-run-30)).

**Within the page.** The column starts at 0, and the page is read or written as one stream in order: for each sector its 16-byte chunk, then its 512 data bytes ([R1](#1121-r1-page-stream), [chapter 10, section 10.9](10-ecc-engine.md#109-what-is-stored-on-the-chip)). The programs do not send a new column between sectors.

**Two-plane programs.** The first page of a pair is in block B and the second page in the next block, B+1. Both are addressed in the same way, and the second is sent with `81h` ([WR9](#1159-wr9-variants), [chapter 11, section 11.5.10](#11510-device-checks-of-the-write-sequence)).

### 11.1.15 Worked example: chip CE1, block 3000, page 5

One page, followed through a read, an erase of its block and a write, with every register write and the bytes that go over the NAND bus. The sequences are the ones the device experiments used on the unit tested ([`rawpage`](14-evidence.md#142113-rawpage-run-29), [`rawwrite`](14-evidence.md#142114-rawwrite-run-29), [`eraseseq`](14-evidence.md#142117-eraseseq-run-29), [`writeseq2`](14-evidence.md#142125-writeseq2-run-31)).

**Choosing the chip and the page.**

- Chip: CE1 is bit 2 of `FMC_CTRL0`, so the control word is `0x43805` (`0x43801`, the word with all chip-enable bits clear, with bit 2 set). Setting bit 24 as well gives `0x1043805`, which allows DMA.
- Page: the row number is block x 128 + page = 3000 x 128 + 5 = 384 005 = `0x5DC05`.
- For a read or a write the five address cycles carry column 0 (two bytes) and the row (three bytes): `FMC_ANUM` = 4, `FMC_ADDR0` = row << 16 = `0xDC050000`, `FMC_ADDR1` = row >> 16 = `0x5`. The bytes on the NAND bus are `00 00 05 DC 05` (lowest byte first; this order is inferred from the formats that work on the device).
- For an erase the three address cycles carry the row of the block's first page, 3000 x 128 = 384 000 = `0x5DC00`: `FMC_ANUM` = 2, `FMC_ADDR0` = `0x5DC00`; bus bytes `00 DC 05`.

Every command byte is sent the same way: `FMC_CMD` = the byte, `wait` event 0, `FMC_STATUS` = `0x2`. Every set of address cycles is sent the same way: write `FMC_ANUM`, `FMC_ADDR0`, `FMC_ADDR1`, then `FMCTRL1` = 1, `wait` event 1, `FMC_STATUS` = `0x4`.

**Waiting for the chip (status poll).** The program writes `FMC_DATA_SEL` = 1 and sends command `70h`. Then `FMCTRL1` = `0xca` latches the chip's status byte into `FMC_NSTAT`. While `FMC_NSTAT` bit 6 is clear, the program waits for `wait` event 5, acknowledges with `FMC_STATUS` = `0x800000` and latches again. Finally `FMCTRL1` = `0x20` leaves status mode. The status byte read `0x81` while the chip was busy and `0xc0` when it was ready; bit 0 set would mean the operation failed.

**Reading the page.**

| step | register writes | NAND bus |
|---|---|---|
| select the chip | `FMC_CTRL0` = `0x1043805` | CE1 active |
| start the page load | command `00h`; address cycles (5); command `30h` | `00`, `00 00 05 DC 05`, `30`; the chip copies the page into its page register (busy) |
| wait for the chip | status poll | `70`, status bytes until `0xc0` |
| back to data | `FMCTRL1` = `0x100e0`; command `00h` | `00` |
| per sector k = 0..3: the chunk | `FMC_DNUM` = `0xf`, `FMC_DATA_SEL` = `0x10`, `FMC_24` = 16 x k, `FMCTRL1` = `0x32`, `wait` event 3, `FMC_STATUS` = `0x8` | 16 bytes from the chip into slot k of parity buffer 0 |
| per sector: the data | `FMC_DNUM` = `0x1ff`, `FMC_DATA_SEL` = 1, `FMCTRL1` = `0xe2`, `wait` event 3, `FMC_STATUS` = `0x8` | 512 bytes into data buffer 0 |
| per sector: decode and correct the data in data buffer 0 | `ECC_DONE` = `0x1ff`, `ECC_MODE` = `0x180`, `ECC_KICK` = `0x1101` + (k << 16), `wait` event 4, read `ECC_RESULT` | none |
| per sector: copy to system memory | `FMC_DMA_ADDR` = the system memory address, `FMC_DATA_SEL` = `0x100`, `FMCTRL1` = `0x100`, `wait` event 2, `FMC_STATUS` = `0x100000` | none |
| end | `FMC_CTRL0` = `0x43801` | CE1 inactive |

The chip sends the page as one byte stream. The steps above read its first 2 KiB unit: four times a 16-byte chunk followed by 512 data bytes, 2112 bytes. The second unit of a 4 KiB page (bytes 2112..4223 of the 4224-byte page) follows in the same stream and is read the same way. The controller only splits the stream into the transfers above. No new address or command is needed between sectors.

**Erasing the block.**

| step | register writes | NAND bus |
|---|---|---|
| select the chip | `FMC_CTRL0` = `0x43805` | CE1 active |
| erase | command `60h`; address cycles (3); command `D0h` | `60`, `00 DC 05`, `D0`; the chip erases all 128 pages of block 3000 (about 1.1 ms with data in it) |
| wait for the chip | status poll | `70`, `0x81` until `0xc0` |

**Writing the page.**

| step | register writes | NAND bus |
|---|---|---|
| per sector: data into data buffer 0 | `FMC_CTRL0` = `0x1043805`; `FMC_DNUM` = `0x1ff`, `FMC_DATA_SEL` = 1, `FMC_DMA_ADDR` = the system memory address, `FMCTRL1` = `0x2e0`, `wait` event 2, `FMC_STATUS` = `0x100000` | none (system memory to data buffer 0) |
| metadata (once per page) | `FMC_META0..2` = the 12 metadata bytes, `FMC_METACFG` = `0x3210`, `FMC_METACTL` = 1, poll until bit 0 clears | none |
| start the program | command `80h`; address cycles (5) | `80`, `00 00 05 DC 05` |
| per sector k: parity | `ECC_DONE` = `0x1ff`, `ECC_MODE` = `0x180`, `ECC_KICK` = `0x1102` + (k << 16), `wait` event 4 | none (the 13 parity bytes go into slot k of parity buffer 0) |
| per sector: send the chunk | `FMC_DNUM` = `0xf`, `FMC_DATA_SEL` = `0x1000`, `FMC_28` = 16 x k, `FMCTRL1` = `0x34`, `wait` event 3, `FMC_STATUS` = `0x8` | 16 bytes to the chip |
| per sector: send the data | `FMC_DNUM` = `0x1ff`, `FMC_DATA_SEL` = `0x100`, `FMCTRL1` = `0xe4`, `wait` event 3, `FMC_STATUS` = `0x8` | 512 bytes to the chip |
| confirm | command `10h` | `10`; the chip programs the page (about 0.2 to 0.8 ms) |
| wait for the chip | status poll | `70`, `0x81` until `0xc0`; bit 0 clear = written |

A page can be written only at an erased position of a block. Programming changes bits from 1 to 0 only, so a page written twice without an erase holds the AND of both contents.

## 11.2 ECC and data path (shared by READ and PROGRAM)

### 11.2.1 R1: Page stream

A 2 KiB unit is four sectors, each stored as `[16-byte chunk][512 data bytes]`: 2112 bytes per unit. A 4 KiB page is two consecutive units: 4224 bytes, read or written as one continuous stream starting at column 0 ([RE-15](14-evidence.md#14315-re-15-page-stream)). The 16-byte chunk holds two things:

- the sector's ECC parity (13 bytes; device, [chapter 10, section 10.9](10-ecc-engine.md#109-what-is-stored-on-the-chip));
- one 3-byte fragment of the page's 12-byte spare metadata. The chunk of sector `k` carries fragment `k` (inferred from the original programs, device-confirmed on the flash).

On write, the chunk of a sector precedes its data.

### 11.2.2 R2: Buffers

The controller has two data buffers and two parity buffers. Each parity buffer is divided into four slots. A sector's chunk is read into slot `k` of a parity buffer (`k` = sector index within the unit), and its 512 data bytes are read into a data buffer. The decode kick names the data buffer, the parity buffer and the slot that hold the sector ([R3](#1123-r3-decode-of-one-sector-read)). The original programs use parity buffer 0 for all four slots and alternate between data buffer 0 and data buffer 1. The third-party driver alternates between parity buffer 0 and parity buffer 1 as well.

### 11.2.3 R3: Decode of one sector (READ)

After a sector's chunk (in its slot) and its 512 data bytes have been fully transferred and the waits acknowledged, the original programs do the following ([RE-12](14-evidence.md#14312-re-12-ecc-protocol-in-read)):

1. Write `ECC_DONE` = `0x1ff` and `ECC_MODE` = `0x180`.
2. Write `ECC_KICK` = the decode word (data buffer, parity buffer, slot).
3. `wait` event 4.
4. Write `ECC_DONE` = 1 and read `ECC_RESULT`.

The decode corrects the data in the data buffer in place. The DMA of that data buffer to system memory starts after the decode has completed. A decode overlaps the NAND read of the next sector into the other data buffer (the ECC engine is a separate unit). The DMA of a corrected data buffer and the NAND read into the other data buffer are two `FMCTRL1` operations, so they run one after the other ([C12](#11112-c12-one-fmctrl1-operation-at-a-time)): the program starts one, waits for its event, then starts the other. A data buffer is refilled only after its DMA has completed (its `wait` event 2 has returned). It is not drained or refilled while a decode or encode on it is running.

### 11.2.4 R4: Per-sector verdict

The page word (the result word `res[i]` of the page, [chapter 12, section 12.2](12-cpu-interface.md#122-result-words)) is built from the verdicts of the page's sectors, both units of a 4 KiB page included, combined by OR ([RE-13](14-evidence.md#14313-re-13-page-verdict-in-read)). After each decode:

- if `ECC_RESULT` bit 0 is set, the page word gets bit 30;
- otherwise, with `n = (ECC_RESULT >> 16) & 0xf`, if `n > 0` the page word gets bit `n - 1`.

### 11.2.5 R5: Encode of one sector (PROGRAM)

After the sector's data is in a data buffer (DMA done, acknowledged) and after the page's meta encode ([WR3](#1153-wr3-meta-encode)), the original programs write `ECC_DONE` = `0x1ff`, `ECC_MODE` = `0x180` and `ECC_KICK` = the encode word, then `wait` event 4 and `ECC_DONE` = 1. The chunk in the sector's slot is sent to the chip after the encode has completed. The encode of sector `k+1` overlaps the NAND write of sector `k`'s data.

### 11.2.6 R6: Meta layout word

`FMC_METACFG` selects which slots the 12 metadata bytes are spread over (three bytes per slot). The layout of the original programs uses slots 0..3 of parity buffer 0, one slot per sector of a unit; its word is `0x3210` ([RE-16](14-evidence.md#14316-re-16-meta-engine-use)). The third-party driver places the four chunks of a unit in slot 0 of parity buffer 0, slot 0 of parity buffer 1, slot 1 of parity buffer 0 and slot 1 of parity buffer 1, and uses `0x5140`. The nibbles of `0x5140`, taken from the lowest, are 0, 4, 1, 5: the slot index `buffer * 4 + slot` for fragments 0..3. Both words are inferred to describe the layout that the respective driver uses. The first word is observed in the original programs, the second in the CPU-driven read path.

### 11.2.7 R7: DMA addressing

`FMC_DMA_ADDR` is written with a unit's bus address before the first DMA of that unit. It is written again for the next unit once the previous unit's last DMA has been acknowledged. The address advances by 512 for each sector transfer. `FMC_DMA_CTRL = 7` is written before the last DMA of a run, in the original programs and in the third-party driver ([RE-19](14-evidence.md#14319-re-19-dma-address-and-dma-control)). `FMC_CTRL0` bit 24 is set before the first DMA of the run. It is cleared before the final transfer without DMA (PROGRAM) or at the end of the run (READ). The reason for this placement is unknown.

## 11.3 RESET and READ ID (program READID)

### 11.3.1 ID1: Control word

READID writes `FMC_CTRL0` = `0x33201` with all CE bits clear ([RE-10](14-evidence.md#14310-re-10-readid-control-word)). The third-party driver uses `0x43803`, which has the CE0 bit set. Both words contain slow timing values. The ID read does not depend on the timing (3p: every legal combination reads the ID).

### 11.3.2 ID2: Reset

READID sets all eight CE bits at once and sends command `FFh` ([C5](#1115-c5-one-command-cycle)). It then waits a settle delay (a loop that counts down the value of the parameter word `0xd14`) and only after the delay deselects all CEs. The original CPU driver does not reset the chips itself; READID is the only place the reset happens ([RE-24](14-evidence.md#14324-re-24-readid-sequence), [RE-36](14-evidence.md#14336-re-36-cpu-driver-chip-reset)).

### 11.3.3 ID3: Settle delay

`P_CEMAP` holds a loop count for READID. After the reset command and before the first ID command, the program runs a two-instruction loop for that many iterations (the count must be at least 1). The original CPU driver passes `0x8000`. The third-party driver waits 5 ms, and a caller retries once after a further 5 ms if the first ID is blank. The delay takes the place of a status poll, because the listed CEs may be absent ([C11](#11111-c11-absent-chips)).

### 11.3.4 ID4: Per CE

READID repeats the following for each of the `P_ROWS` list entries. The CE index comes directly from `P_CELIST`, with no `cemap`.

1. Select the CE.
2. Send command `90h`, then one address cycle with address 0.
3. Set `FMC_DNUM` = 7 and `FMC_DATA_SEL` = 1, write `FMCTRL1` = `0x1c2`, `wait` event 3, acknowledge. The 8 ID bytes are now in data buffer 0.
4. Set `FMC_DATA_SEL` = `0x100` and write `FMCTRL1` = `0x340`.
5. Read `FMC_FIFO` twice.

The first read is the word stored into `idbuf[i]` (ID bytes 0..3, byte 0 in bits 7:0; for example `0xa514d3ad` for the ID bytes `AD D3 14 A5`). The second word (ID bytes 4..7) is read and discarded. The third-party driver reads once or twice. An absent CE yields blank data (0 or `0xffffffff`; the original CPU driver rejects both) and does not fault, because only `wait` events 0, 1 and 3 are used.

### 11.3.5 ID5: End

READID deselects all CEs and ends with no failure.

## 11.4 Page read (program READ)

Per page, the protocol as seen by the chip is:

1. Command `00h`.
2. Five address cycles.
3. Command `30h`. The chip is then busy for the page read time tR.
4. Status poll (`70h`, [C7](#1117-c7-ready-procedure-status-poll)).
5. Command `00h`, after writing `FMCTRL1 = 0x100e0`.
6. The stream of [R1](#1121-r1-page-stream): `4 * P_CHUNKS` sectors, with no further commands.

### 11.4.1 RD1: Pipeline across chips

READ first sends the page-load command `30h` for several pages, each on a different CE, before it transfers the data of any of them. It then transfers the pages one after another, in request order, while the chips of the other pages are still busy. A chip does not receive a second read command before all units of its previous page have been transferred. The original program starts every page of a batch. A batch ends at the first page whose CE is already used in the batch ([RE-17](14-evidence.md#14317-re-17-read-batches)).

### 11.4.2 RD2: Before the data

For the chip whose page is to be transferred, READ selects it, runs [C7](#1117-c7-ready-procedure-status-poll), writes `FMCTRL1 = 0x100e0`, and sends command `00h`. Without the status poll the read can start before tR has elapsed; without `00h` the status byte is read instead of data.

### 11.4.3 RD3: Order inside a sector

READ first reads the 16-byte chunk into slot `k` of a parity buffer (`FMC_DNUM = 0xf`, `FMC_DATA_SEL` bits 5:4 selecting the parity buffer, `FMC_24 = 16 * k`, `FMCTRL1 = 0x32`, `wait` event 3). It then reads the 512 data bytes into a data buffer (`FMC_DNUM = 0x1ff`, `FMC_DATA_SEL` bits 1:0 selecting the data buffer, `FMCTRL1 = 0xe2`). The chunk of sector `k` is read before the sector's data and before the decode of sector `k`. The chunk of the next sector can be read while the previous sector decodes.

### 11.4.4 RD4: Decode and DMA

Decode and verdict are as in [R3](#1123-r3-decode-of-one-sector-read) and [R4](#1124-r4-per-sector-verdict). The DMA of each sector to system memory (`FMCTRL1 = 0x100`; `0x1e0` for the last of the run) writes to the address in `FMC_DMA_ADDR`, which advances by 512 per sector, that is by 2 KiB per unit.

### 11.4.5 RD5: Unit boundary (4 KiB pages)

The next unit's first sector is read straight after the previous unit's last sector (one stream, no new command). The DMA address of the next unit comes from the next entry of the DMA list.

### 11.4.6 RD6: Switching chips

READ changes the CE selection only between pages, after the last NAND read of the previous page has completed (its data `wait` event 3 acknowledged). The decode, DMA and meta decode of the previous page's last sector overlap the status poll and the first sector read of the next page (the data and parity buffers are not tied to a CE). The meta decode ([RD9](#1149-rd9-meta)) of a page finishes before the chunk read of the next page's sector 0 overwrites slot 0, if both pages use the same parity buffer.

### 11.4.7 RD7: Chunk-ECC flag

`FMC_STATUS` bit 27 is set by the controller while a chunk is read from the chip. It indicates that the sector being read had been written (programmed). The original firmware treats a page whose bit 27 stayed clear from the page start to the end of its last data read as "no ECC data" (an erased page). A third-party driver measured over a whole 4 GB unit that every programmed page sets the bit and every erased page does not (3p).

READ tests bit 27 once per page, after the last data read of the page has completed. It then acknowledges the bit (writes `0x8000000`) so that the next page starts clear ([RE-18](14-evidence.md#14318-re-18-page-word-and-fmc_status-b27)).

Device: bit 27 was set after reading every written sector (valid code words, sectors with stored errors, and a written sector whose chunk is all `0xff`) and clear after reading every erased sector (all `0xff`). `FMC_STATUS` read `0x0803c000` after a written sector and `0x0003c000` after an erased one ([`rawpage`](14-evidence.md#142113-rawpage-run-29), [`rawwrite`](14-evidence.md#142114-rawwrite-run-29)).

### 11.4.8 RD8: Page result

The page word depends on bit 27:

- If bit 27 stayed clear, the page word is `0x20000000` OR-ed with the verdict of the page's last sector only. Anything accumulated from earlier sectors is discarded.
- Otherwise the page word is the OR of all sector verdicts ([R4](#1124-r4-per-sector-verdict)).

This is the exact behaviour of the original firmware: the test is made before the last sector's decode result is added ([RE-18](14-evidence.md#14318-re-18-page-word-and-fmc_status-b27)). Why the earlier verdicts are discarded is not known; decoding an erased sector gives `ECC_RESULT` = 0 (device, [chapter 16, section 16.1](16-device-findings.md#161-risks-and-their-status), item 4). The word is stored in `res[i]` and ORed into `r6`.

### 11.4.9 RD9: Meta

After the last sector of the page (the last unit) has been decoded, READ writes `FMC_METACFG` ([R6](#1126-r6-meta-layout-word)) and `FMC_METACTL` = 2, polls until bit 1 reads 0, and copies `FMC_META0..2` into `meta[3 * i .. 3 * i + 2]`. The original performs one meta decode per page, using the slots of the last unit ([RE-16](14-evidence.md#14316-re-16-meta-engine-use)). On the chip, both units of a 4 KiB page carry the same 12 metadata bytes: in all 42 pages written by the original firmware that were read whole (pages 0, 1 and 64 of 7 blocks on CE0 and CE1), the 3 metadata bytes of each chunk of the second unit equalled those of the first (device, [`fullpage`](14-evidence.md#142163-fullpage-run-37); [WR3](#1153-wr3-meta-encode)).

### 11.4.10 RD10: Erased and uncorrectable pages

The data is still DMA-ed to system memory (an uncorrectable page delivers whatever the data buffers hold), the metadata is still decoded and stored, and the run continues with the next page. Failure is reported only through the page word.

### 11.4.11 RD11: End

After the last page, READ clears `FMC_CTRL0` bit 24, deselects all CEs, and ends with `r6` = the OR of the page words.

## 11.5 Page program (program PROGRAM)

Protocol as seen by the chip, per page:

- Single plane: `80h`; five address cycles; `4 * P_CHUNKS` times `[16-byte chunk][512 data bytes]`; `10h`; then the chip is busy for the program time tPROG.
- Two plane (`P_FLAGS` bit 0): the two pages of a pair are on one CE, plane 0 first. First page: `80h`, address, data, `11h`. Second page: `81h`, address, data, `10h`.

### 11.5.1 WR1: First sector staging

Before the first command of the run, the program DMAs the first sector's 512 bytes from system memory into data buffer 0 (`FMC_DMA_ADDR` = first list entry, bit 24 set, `FMC_DNUM = 0x1ff`, `FMC_DATA_SEL = 1`, `FMCTRL1 = 0x2e0`, `wait` event 2, acknowledge).

### 11.5.2 WR2: Chip free

Before sending `80h` or `81h` to a CE that already got a program command in this run, the original programs check that the CE is ready and test its FAIL bit ([C7](#1117-c7-ready-procedure-status-poll)). In a two-plane pair this includes the second page's `81h`: after `11h` the chip is busy for a short time, and it is polled before `81h` ([RE-21](14-evidence.md#14321-re-21-two-plane-status-polls)). A FAIL ends the run ([WR7](#1157-wr7-status-and-failure)).

### 11.5.3 WR3: Meta encode

For each page, after the previous page's last chunk has been sent and before the encode of the page's first sector, the program does the following ([RE-16](14-evidence.md#14316-re-16-meta-engine-use)):

1. Write the page's 12 metadata bytes to `FMC_META0..2`.
2. Write `FMC_METACFG` ([R6](#1126-r6-meta-layout-word)).
3. Write `FMC_METACTL` = 1.
4. Poll bit 0 until it reads 0.

The meta encode runs once per page. The same fragments are carried by every unit of a 4 KiB page (inferred).

### 11.5.4 WR4: Sector order

For each sector, the program does the following:

1. Encode ([R5](#1125-r5-encode-of-one-sector-program)).
2. Send the chunk from its slot: `FMC_DNUM = 0xf`, `FMC_DATA_SEL = 0x1000` (parity buffer 0), `FMC_28 = 16 * k`, `FMCTRL1 = 0x34`, `wait` event 3.
3. Send the data: `FMC_DNUM = 0x1ff`, `FMC_DATA_SEL` = the data buffer to empty (and the data buffer to fill, for `0x2e4`), `FMCTRL1 = 0x2e4` (or `0xe4` without DMA), `wait` event 3.

With `0x2e4`, the next sector's data is DMA-ed into the other data buffer during the transfer. The next sector is the next unit's first sector, or the next page's first sector. The `wait` event 2 for that DMA comes before the next encode. Using `0x2e0` and then `0xe4` one after the other is equivalent.

### 11.5.5 WR5: DMA list

The DMA list has one entry per unit, consumed in order. The final page reads no entry beyond the last ([RE-20](14-evidence.md#14320-re-20-write-dma-list)). `FMC_DMA_CTRL = 7` is written before the last DMA. The last sector of the run is written with `FMCTRL1` = `0xe4` (no DMA) after `FMC_CTRL0` bit 24 is cleared.

### 11.5.6 WR6: Confirm

After the last data of a page, the program sends `10h` (or `11h` for the first plane of a pair), then waits for `wait` event 0 and acknowledges. Then it selects the CE of the next page. The next page's first sector is already staged in a data buffer and encoded, because the data and parity buffers are not tied to a CE.

### 11.5.7 WR7: Status and failure

The status poll with FAIL test is done (a) before a CE is reused in the run ([WR2](#1152-wr2-chip-free)) and (b) for every CE used, after the last confirm. The FAIL test is `FMC_NSTAT & 1` for `70h`. On a FAIL, the original program does the following ([RE-22](14-evidence.md#14322-re-22-program-fail-path)):

1. Put the failing CE (the CE currently selected, one-hot) into r6 bits 7:0.
2. Set bit 28.
3. Run the common end ([C10](#11110-c10-end-state)) and end.

The remaining pages are not programmed.

### 11.5.8 WR8: Result

Normal end: `r6` = the mask of the CEs used, bit 28 clear, and the CE bits in `FMC_CTRL0` cleared.

### 11.5.9 WR9: Variants

`P_FLAGS` bit 0 = 1 selects the `80h`/`11h`, `81h`/`10h` form (the original firmware's chip mode 8). The original firmware has further forms:

- two-plane with `80h` for both planes;
- status `78h` with three row cycles (the row of the page about to be programmed on that CE) and fail mask 3;
- cache program (WRITECACHE): it confirms a page with `15h` (cache program) instead of `10h` as long as the remaining page count AND (0 minus the bank count) is non-zero, and ends with a `71h` status sweep. The bank count comes from the parameter word `0xd04`; the program stores 0 minus it in `0xd44` ([RE-46](14-evidence.md#14346-re-46-bank-count-in-writecache)). With a bank count that is a power of two, the AND is non-zero while at least one full group of pages, one page per bank, remains (inferred).

These forms are used for chip modes 2, 3, 4, 9, 12, 13 and are not exercised by the validated chips of the original CPU driver ([RE-27](14-evidence.md#14327-re-27-write-variants), [RE-38](14-evidence.md#14338-re-38-cpu-driver-modes)). The `78h` forms differ in release 1.1.3. There, the final status sweep per CE reads the status with `78h` at four row addresses (a row given in parameter word `0xd00`, that row + `0x80`, rows `0` and `0x80`) with fail mask 3. The other releases use one `70h` with mask 1 ([RE-28](14-evidence.md#14328-re-28-releases-101-to-113)).

### 11.5.10 Device checks of the write sequence

The checks used scratch blocks. Every page was read back raw and compared byte for byte (data, metadata and parity) ([`writeseq`](14-evidence.md#142115-writeseq-run-29), [`writeseq2`](14-evidence.md#142125-writeseq2-run-31), [`pageorder`](14-evidence.md#142122-pageorder-run-30), [`eraseseq`](14-evidence.md#142117-eraseseq-run-29)):

- One page per run, [WR3](#1153-wr3-meta-encode) + [WR4](#1154-wr4-sector-order) with `0xe4` for the data: correct on CE0 and on CE1, also both CEs programmed back to back in one run (`80h` .. `10h` on CE0, then on CE1, then a status poll of each).
- Two-plane pair (`80h` .. `11h`, status poll, `81h` .. `10h`, blocks B and B+1): both pages correct, about 470 us.
- Several pages per run, with the status poll on `FMC_DATA_SEL = 1` ([C7](#1117-c7-ready-procedure-status-poll)) and the status latch repeated without waits: eight pages in order with `0x2e4` overlap and `FMC_DMA_CTRL = 7` before the last DMA, pages 9 and 11, and pages 1 and 3. Every page was correct.
- A run that polled with `FMC_DATA_SEL = 0x100` (left over from the data transfer): the latch returned `0x70` instead of the status ([chapter 7, section 7.1](07-events-and-waits.md#71-wait-events)). The poll ended while the chip was still programming, and the chip ignored the next `80h` (that page read back erased). There is no page-order rule: with a correct poll, multi-page writes are correct.
- Program time (device, one run per page over every page of three blocks, CE0 and CE1, [`progtime`](14-evidence.md#142148-progtime-run-35)): the run times fall into two groups in a fixed pattern.
  - Pages 0 to 3 and then every pair 6-7, 10-11, ..., 122-123 take about 478 us per run.
  - Pages 4-5, 8-9, ..., 124-125 and 126-127 take 728 to 873 us (CE1 50 to 90 us slower than CE0).
  - The pattern is the same in all three blocks (inferred: the chip's lower and upper pages of a cell pair).
  - The run time includes the transfer of the page (2112 bytes, about 0.21 ms) and the ready poll.
- A page programmed a second time without an erase: the flash holds old AND new (bits only go from 1 to 0), so the sectors decode uncorrectable; the status byte reports no FAIL (`0xc0`).
- Status byte: `0xc0` when ready; `0x81` right after `10h` and during an erase, `0x80` during a page read's tR.
- Erase time: about 1.1 ms for a block holding data, about 0.18 ms for a blank block.
- Erase (`60h`, three row cycles, `D0h`): about 1.1 ms; written pages read back erased afterwards; a two-plane erase (`60h` B, `60h` B+1, `D0h`) erased both blocks; an erase on CE1 worked. An erase sent with four address cycles (`FMC_ANUM = 3`) erased the block (about 1.1 ms): the chip ignores the extra cycle.

## 11.6 Block erase (program ERASE)

NAND-visible protocol per entry: `60h`; three row cycles; `D0h`; (chip busy tBERS); later status poll (`70h`, [C7](#1117-c7-ready-procedure-status-poll)).

### 11.6.1 ER1: Entries

For each of the `P_COUNT` entries, ERASE selects `cemap[cel[i]]`, sends command `60h`, sends the address (`FMC_ANUM` = 2, `FMC_ADDR0 = rows[i]`), and sends command `D0h`. Several CEs erase in parallel: the program does not wait for tBERS between entries ([RE-23](14-evidence.md#14323-re-23-erase-sequence)). The original CPU driver lists each CE at most once per run.

### 11.6.2 ER2: Final sweep

After the last `D0h`, the program handles each CE used in turn: it selects the CE, runs the ready procedure [C7](#1117-c7-ready-procedure-status-poll) (the ready wait here is the long one), and takes FAIL = `FMC_NSTAT & 1`.

### 11.6.3 ER3: Result

Normal end: `r6` = the mask of the CEs used, bit 28 clear. On a FAIL, the failing CE is set as a one-hot mask in r6 bits 7:0, bit 28 is set, and the program ends. The original CPU driver maps the mask back to the entry.

### 11.6.4 ER4: End

ERASE deselects all CEs and ends.

## 11.7 Fault handler and timeouts

### 11.7.1 FL1: Handler

Every original program has one handler at the offset written to `FMSS_C4C` ([RE-25](14-evidence.md#14325-re-25-fault-handlers)). The handler works as follows ([RE-25](14-evidence.md#14325-re-25-fault-handlers)):

1. It reads `FMSS_C3C` (the handler leaves the value in r2 for the CPU's diagnostic print).
2. It puts the selected CE bits (`(FMC_CTRL0 >> 1) & 0xff`) into r6 bits 7:0, clearing the old byte.
3. In READ, READID and RAWREAD it writes `FMSS_C50` = 1 and sets r6 bit 27 unconditionally.
4. In ERASE and in the five WRITE programs it does the same only if `FMSS_C50` bit 0 is set.
5. It deselects all CEs. In ERASE and the five WRITE programs it then also sets bit 28, the failure bit. A status FAIL jumps here with `C50` clear, and therefore yields bit 28 without bit 27.
6. It ends.

The handlers differ in length. Counted from the offset in `FMSS_C4C` to the last instruction, a handler has 13 instructions in ERASE and in the five WRITE programs, and 9 in READ, READID and RAWREAD ([RE-1](14-evidence.md#1431-re-1-nine-sequencer-programs-in-the-ipod-os-image)).

### 11.7.2 FL2: Hung hardware

The original programs rely on the `wait` timeout of the hardware and on the 2 s bound of the CPU. A hung run is reported as a timeout and followed by a sequencer reset ([chapter 3, section 3.5](03-cpu-memory-map.md#35-starting-and-finishing-a-run)).

### 11.7.3 FL3: What the CPU does after a fault

The original CPU driver resets the sequencer, discards the run, and reports the bank from r6 bits 7:0 (for a program) or the entry (for an erase). A READ fault has no per-page result: the original CPU driver treats the whole run as failed and ignores `res[]`.

References: [chapter 14](14-evidence.md) (outcomes), [chapter 13](13-experiments.md) (what was done and why), [chapter 15](15-evidence-interpretation.md) (what the outcomes mean).

Related chapters: [chapter 9](09-nand-controller.md), [chapter 10](10-ecc-engine.md), [chapter 12](12-cpu-interface.md), [chapter 7](07-events-and-waits.md).
