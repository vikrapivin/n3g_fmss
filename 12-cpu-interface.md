# Chapter 12 — CPU interface: parameter words, result words and timing word

Summary: How the original CPU driver and the original programs exchange data through parameter RAM: the parameter words that carry a program's inputs and outputs, the result words that report each run, and the timing word that sets the NAND bus timing. The sequence the driver follows to start a run is in [chapter 3, section 3.5](03-cpu-memory-map.md#35-starting-and-finishing-a-run).

## 12.1 Parameter words per program

All pointers are bus addresses of arrays of 32-bit little-endian words. A program derives the CE index as `cemap[cel[i]]`, where `cel[i]` is the logical chip of list entry `i`. The original CPU driver fills `cemap` with the identity map 0..7, and the original programs read the table ([RE-29](14-evidence.md#14329-re-29-chip-addressing-in-the-programs)). The words are listed in order of offset. How the lists select the chip and the page of each access is described in [chapter 11, section 11.1.14](11-nand-operations.md#11114-how-a-chip-and-a-page-are-selected).

| word | name | READID | READ | PROGRAM | ERASE |
|---|---|---|---|---|---|
| `0xd04` | banks | unused | in: the bank count; READ does not read it | in: the bank count; only WRITECACHE reads it ([chapter 11, WR9](11-nand-operations.md#1159-wr9-variants)) | in: the bank count; ERASE does not read it |
| `0xd08` | `P_IDBUF` | out: ID words | | | |
| `0xd0c` | `P_ROWS` | **count of CEs** | in: row (page number) per page | in: row per page | in: first row of the block per entry (`block * pages per block`) |
| `0xd10` | `P_CELIST` | in: CE indices to probe | in: logical chip per page | in: logical chip per page | in: logical chip per entry |
| `0xd14` | `P_CEMAP` | **reset settle loop count** | in: pointer to `cemap[8]` | in: pointer to `cemap[8]` | in: pointer to `cemap[8]` |
| `0xd18` | `P_COUNT` | | in: pages (1..32) | in: pages (1..32; even for two-plane) | in: entries (1..8; each CE at most once) |
| `0xd1c` | `P_META` | | out: 3 words per page | in: 3 words per page | |
| `0xd20` | `P_DMALIST` | | in: one bus address per 2 KiB unit | in: one bus address per 2 KiB unit | |
| `0xd24` | `P_RESULTS` | | out: one result word per page | | |
| `0xd28` | `P_CHUNKS` | | in: 2 KiB units per page (1 or 2) | in: 1 or 2 | |
| `0xd44` | `P_FLAGS` | | | in: bit 0 = two-plane pairs | |
| `0xd48` | `P_WTIMING` | | | in: timing word | |
| `0xd4c` | `P_TIMING` | | in: timing word | | in: timing word |

Layout of the lists (the original CPU driver builds them; `i` is the page position and `u` the unit index within a page):

- `rows[i]`, `cel[i]`: one word per page (the row and the logical chip of page `i`).
- `dma[i * P_CHUNKS + u]` = the bus address of unit `u` of page `i`. The units of a page lie one after the other in system memory, in order. The program reads exactly `P_COUNT * P_CHUNKS` entries, in this order. It reads no further entries, so the array need not be longer.
- `res[i]` (READ out): the result word of page `i`. The words are in request order, whatever order the pages are processed in.
- `meta[i * 3 .. i * 3 + 2]` (READ out / PROGRAM in): the 12 spare metadata bytes of page `i` as three words, `FMC_META0..2` in that order.
- `idbuf[i]` (READID out): the ID word read for list position `i`.

Limits per run: at most 32 pages, at most 2 units per page and at most 8 CEs. The program size is at most `0x1000` bytes for READ and PROGRAM, `0x800` bytes for ERASE and `0x400` bytes for READID.

## 12.2 Result words

**READ**: each page has a result word `res[i]`, and the OR of all page words is returned in `r6` (`SEQ_RESULT`). The bits of a page word:

| bit | meaning |
|---|---|
| 30 | at least one sector of the page was uncorrectable (`ECC_RESULT` bit 0) |
| 29 | marker: the chunk-ECC flag (`FMC_STATUS` bit 27) stayed clear for the whole page ([RD8](11-nand-operations.md#1148-rd8-page-result)) |
| `n - 1` (n = 1..15) | at least one sector needed `n` corrected bits (`n` = `ECC_RESULT[19:16]`) |

The original CPU driver classifies a page as failed if bit 30 is set, as corrected if any of bits 7:0 is set, and as clean otherwise. It does not interpret bit 29. After a fault, `r6` holds the mask of the selected CEs in bits 7:0 together with bit 27 (`0x08000000`).

**PROGRAM and ERASE**, r6:

- Normal end: the bit mask of the CEs used (`1 << CE`), with bit 28 clear.
- Failure: bit 28 set and bits 7:0 = the failing CE or CEs as a one-hot mask. The original CPU driver takes the lowest set bit for a program failure, and the first list entry whose CE is in the mask for an erase failure.
- Fault ([FL1](11-nand-operations.md#1171-fl1-handler)): bit 27 is set in addition.

The original CPU driver tests only bit 28 and the CE bits.

**READID**: the original CPU driver does not read `r6` after a clean END. After a fault, `r6` is as for READ above.

## 12.3 Timing word

The program ORs a timing word into `FMC_CTRL0`. The word has the fields `setup << 16 | hold << 12 | edo << 28`. `P_TIMING` is used by READ, RAWREAD and ERASE, and `P_WTIMING` by the five WRITE programs. READID uses a fixed word ([ID1](11-nand-operations.md#1131-id1-control-word)).

The original CPU driver builds the word from three times in nanoseconds taken from the chip table record of the chip found: the cycle time, the low time and the high time of the bus signals ([RE-35](14-evidence.md#14335-re-35-cpu-driver-timing-word)). T is the bus clock period in nanoseconds: 1000 divided by the value that the driver's clock function returns for argument 2, read as the bus clock frequency in MHz (inferred from that function). `c1`, `c2` and `c3` are the three times in whole clock periods, rounded up:

1. `c1` = `ceil(cycle / T)`, `c2` = `ceil(low / T)`, `c3` = `ceil(high / T)`.
2. If `c3 + c2 < c1`, then `c3 = c1 - c2`.
3. `hold = max(c3 - 1, 1)` and `setup = max(c2 - 1, 1)`.
4. `edo = min(hold + 1, 3)`; `edo` is the read-capture delay field (`FMC_CTRL0` bits 29:28, [chapter 9, section 9.1](09-nand-controller.md#91-fmc_ctrl0-0x000-nand-controller-configuration-not-a-sequencer-control)).
5. The word is `(setup & 7) << 16 | (hold & 7) << 12 | (edo & 3) << 28`.

The driver keeps the read word and the write word in a structure, at offsets `0x1c` and `0x20`. Before it starts a program, it copies them to the parameter words `0xd4c` (`P_TIMING`) and `0xd48` (`P_WTIMING`). The figures come from a table of 18 records of 44 bytes, one record per chip type. The bus clock frequency is known only as an inferred value of about 108 MHz ([chapter 7, section 7.5](07-events-and-waits.md#75-timeout-length)). The third-party driver runs the default word `(4 << 16) | (3 << 12)` (no EDO) on hardware.

References: [chapter 14](14-evidence.md) (outcomes), [chapter 13](13-experiments.md) (what was done and why), [chapter 15](15-evidence-interpretation.md) (what the outcomes mean).

Related chapters: [chapter 11](11-nand-operations.md), [chapter 3](03-cpu-memory-map.md).
