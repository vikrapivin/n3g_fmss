# Chapter 9 — NAND controller (FMC)

Summary: The registers of the NAND controller (FMC): the configuration word, the operation codes, the other registers, the status flags, the data-buffer routing and the second register set (channel 1).

## 9.1 `FMC_CTRL0` (`0x000`): NAND controller configuration (not a sequencer control)

| bits | meaning | evidence |
|---|---|---|
| 0 | controller enable; always 1 | observed |
| 8:1 | chip enables: which NAND chips the controller talks to (details below the table) | observed / 3p / device |
| 9 | set in the control word of the original READID program (`0x33201`) and clear in the other original programs. Setting it during a whole page read or a READ ID changed neither the data, the decode results nor the ID ([`fmcbits`](14-evidence.md#142143-fmcbits-run-35)) | observed / device; meaning unknown |
| 10 | named "enable DMA" in third-party headers. It is clear in every original program: there, DMA is started through bit 24 and `FMC_DMA_ADDR`. The third-party driver clears it for single-chunk reads | observed / 3p |
| 11 | set in every original program except READID (the control word `0x801` consists of bit 11 and bit 0). Clearing it during a whole page read or a READ ID changed nothing ([`fmcbits`](14-evidence.md#142143-fmcbits-run-35)) | observed / device; meaning unknown |
| 14:12 | timing: hold ("write pulse") clocks | observed formula ([chapter 12, section 12.3](12-cpu-interface.md#123-timing-word)) |
| 18:16 | timing: setup clocks | observed formula |
| 24 | DMA enable: set while the data phase of a run uses DMA, cleared at the end (details below the table) | observed / 3p / device |
| 29:28 | EDO (read-capture delay) | observed formula |

Chip enables (bits 8:1):

- Each bit selects one chip-enable line: bit 1 is chip 0 (CE0), bit 2 is CE1, and so on up to bit 8 for CE7.
- Normally exactly one of these bits is set, so commands, addresses and data go to that chip only. With several bits set, the same cycles go to all those chips at once; the original programs do that only for the RESET command (`FFh`).
- Examples with the test programs' control word: `0x43801` selects no chip, `0x43803` CE0, `0x43805` CE1 and `0x43809` CE2.
- On the unit tested, CE0 to CE3 each answered READ ID ([chapter 16, section 16.2](16-device-findings.md#162-settled-on-a-device)).
- How a run chooses the chip and the page is described in [chapter 11, section 11.1.14](11-nand-operations.md#11114-how-a-chip-and-a-page-are-selected).

DMA enable (bit 24, device):

- With bit 24 clear, no DMA runs at all. This holds in both directions (`0x2e0` system memory to data buffer, `0x100` or `0x1e0` data buffer to system memory) and in both register sets. `FMC_STATUS` bit 20 never sets, so the `wait` times out.
- With bit 24 set, the same transfers complete (512 bytes, data intact).
- Leaving bit 24 set through a whole page read (chip transfers included) changed nothing ([`fmcbits`](14-evidence.md#142143-fmcbits-run-35)).

The programs load `0x801 | timing` once and then change the CE bits with a read-modify-write that preserves every other bit. A control word with bit 0 clear is not usable.

## 9.2 `FMCTRL1` (`0x004`): NAND controller operation trigger (not a sequencer control)

Writing a value (a "code" in the table below) to `FMCTRL1` starts the operation that the code names. Bit names (third-party driver): bit 0 address cycles, bit 1 NAND-to-controller data, bit 2 controller-to-NAND data, bit 6 clear write FIFO, bit 7 clear read FIFO. The rest of each code is known from observed use only.

| code | operation | data buffer routing from `FMC_DATA_SEL` | evidence |
|---|---|---|---|
| `0x1` | send `FMC_ANUM` + 1 address cycles from `FMC_ADDR0` and `FMC_ADDR1` | none | observed / 3p |
| `0x20` | end of a status phase | none | observed / 3p |
| `0x32` | read `FMC_DNUM` + 1 bytes from the NAND into a 16-byte slot of a parity buffer | parity buffer: `FMC_DATA_SEL` bits 5:4; slot offset: `FMC_24` | observed / 3p |
| `0x34` | write a 16-byte slot of a parity buffer to the NAND | parity buffer: `FMC_DATA_SEL` bits 13:12; slot offset: `FMC_28` | observed / 3p |
| `0xca` | start the NAND status poll; the status byte appears in `FMC_NSTAT` and `FMC_STATUS` bit 23 is set | none | observed / 3p |
| `0xe2` | read `FMC_DNUM` + 1 bytes from the NAND into a data buffer | data buffer: `FMC_DATA_SEL` bits 1:0 | observed / 3p |
| `0xe4` | write a data buffer to the NAND | data buffer: `FMC_DATA_SEL` bits 9:8 | observed / 3p |
| `0x100` | DMA a data buffer to system memory. A second `FMCTRL1` write before this DMA's `wait` event 2 has returned replaces the DMA ([C12](11-nand-operations.md#11112-c12-one-fmctrl1-operation-at-a-time)) | drain: `FMC_DATA_SEL` bits 9:8 | observed / 3p / device |
| `0x1c2` | read `FMC_DNUM` + 1 bytes from the chip (READ ID), to be read through the FIFO port `FMC_FIFO` | `FMC_DATA_SEL` = 1 | observed / 3p |
| `0x1e0` | as `0x100`, used for the last DMA of a run, which is preceded by a write of `FMC_DMA_CTRL` = 7 | drain: `FMC_DATA_SEL` bits 9:8 | observed / 3p |
| `0x2e0` | DMA system memory to a data buffer, no NAND transfer | data buffer: `FMC_DATA_SEL` bits 1:0 | observed / 3p |
| `0x2e4` | write one data buffer to the NAND while a DMA from system memory fills the other | write: `FMC_DATA_SEL` bits 9:8; fill: `FMC_DATA_SEL` bits 1:0 | observed / 3p |
| `0x340` | FIFO read mode, written before reading `FMC_FIFO` | `FMC_DATA_SEL` = `0x100` | observed / 3p |
| `0x100e0` | prefix written before the `00h` command that returns a chip from status output to data output | none | observed / 3p |
| `0x0ff3f8e0` | set-up word, written once at the start of every run, followed by the all-flags clear (`FMC_STATUS` = `0x0ff00ffe`) | none | observed / 3p |

Codes that are not in this table (for example `0x12`, `0x42`, `0x180` and `0x1a0`) occur in the 4G / 5G firmware or in the third-party driver's single-chunk path. The original 3G programs do not use them.

## 9.3 Other NAND controller registers

| offset | name | meaning |
|---|---|---|
| `0x008` | `FMC_CMD` | NAND command byte; writing sends one command cycle; flag bit 1 |
| `0x00c` / `0x010` | `FMC_ADDR0` / `FMC_ADDR1` | address bytes, least significant first: ADDR0 bytes 0..3, ADDR1 byte 4 |
| `0x014` | `FMC_DATA_SEL` | selects the data buffer and parity buffer of an operation (table below) |
| `0x024` | `FMC_24` | byte offset of the slot in the parity buffer (`0, 0x10, 0x20, 0x30`) that a parity read (`0x32`) fills |
| `0x028` | `FMC_28` | byte offset of the slot in the parity buffer that a parity write (`0x34`) sends; it also acts as a counter (notes below the table) |
| `0x02c` | `FMC_ANUM` | address cycles minus 1 |
| `0x030` | `FMC_DNUM` | transfer length minus 1 of NAND transfers (`0xf` = 16 bytes, `0x1ff` = 512, `7` = 8); 13 bits writable (device). It does not set the DMA length (below the table) |
| `0x034` | `FMC_DMA_ADDR` | bus address of the next DMA transfer. The original programs write it once per 2 KiB unit, so the controller must advance the address by 512 internally after each data buffer DMA. After a single 512-byte DMA the register still reads the value written (device) |
| `0x038` | `FMC_DMA_CTRL` | the original programs and the third-party driver write 7 before the last DMA of a run (observed). In a page read test (device, [`fmcbits`](14-evidence.md#142143-fmcbits-run-35)), the values 0 and 7 deliver correct data, and 7 adds about 100 us. The value `0xff` hangs the DMA: the wait times out and that sector is wrong. Each bit alone, written before every data DMA of a page read (device, [`dmactl`](14-evidence.md#142164-dmactl-run-37)): bit 0 and bit 3 deliver correct data (bit 0 adds about 60 us, bit 3 about 120 us per page); bit 1 and bit 2 deliver the first sector correctly, then every later DMA times out, and the register then reads `0x42` and `0x84` (bit 6 or bit 7 set as well); each of bits 4..7 makes every DMA time out. The value persists between runs. Meaning unknown |
| `0x048` | `FMC_STATUS` | event flags, write-1-to-clear ([chapter 9, section 9.4](#94-fmc_status-bits)) |
| `0x04c` | `FMC_NSTAT` | last NAND status byte latched by `FMCTRL1` = `0xca` (NAND status bits: bit 0 and bit 1 are fail flags, bit 5 is array ready, bit 6 is ready) |
| `0x060..0x068` | `FMC_META0..2` | 12-byte spare metadata: output of a meta decode, input of a meta encode |
| `0x078` | `FMC_METACFG` | metadata layout: four 4-bit nibbles, low nibble first. Nibble j names the slot that meta fragment j (3 bytes) is copied from or to. The nibble value is parity buffer number x 4 + slot number (0..7) (inferred, [R6](11-nand-operations.md#1126-r6-meta-layout-word)) |
| `0x07c` | `FMC_METACTL` | write 1 = meta encode (poll until bit 0 reads 0); write 2 = meta decode (poll until bit 1 reads 0) |
| `0x080` | `FMC_FIFO` | FIFO port: with `FMCTRL1` = `0x340` and `FMC_DATA_SEL` = `0x100` each read returns the next 4 bytes of a READ ID transfer; reads returned 0 for data buffers filled by DMA, and each read advances `FMC_28` by 4 ([`bufport`](14-evidence.md#142154-bufport-run-36); [chapter 1, section 1.5](01-overview.md#15-named-parts-of-the-data-path)) |

Notes on the registers (device unless stated; [chapter 16, section 16.2](16-device-findings.md#162-settled-on-a-device)):

- `FMC_28` is also the read position of the FIFO port: each read of `FMC_FIFO` (`0x080`) advances it by 4, and it wraps at 512 (it read 0, 128, 256, 384 and 0 after each 32 reads, [`bufport`](14-evidence.md#142154-bufport-run-36)). While a program runs without reading `0x080` it does not change, except that a DMA and the set-up word set it to 0.
- Between the CPU's dump of the register window before a run and the start of the run, it advanced by 248. Between the end of the run and the CPU's dump after the run, it advanced by 124 ([`fmc28b`](14-evidence.md#142135-fmc28b-run-35)).
- It advanced by exactly 128 over 128 iterations of a `pld`/`sti` loop, and by `0x7c` per probe run.
- In programs that write the set-up word first, it stayed 0 over 32 repetitions each of plain instructions, `sti`, `ldi`, parameter-RAM writes, DMAs, ECC encodes and 16-byte chip reads.
- The CPU's dumps of the whole window read `0x080` and its mirrors, which fits the advances between runs (inferred).
- `FMC_DNUM` does not set the DMA length: a DMA always moves 512 bytes (256 to 2048 tried). The 512-byte sector size is known from two sources. In the original programs, every data transfer uses `0x1ff` with one ECC operation and one 512-byte DMA per sector. The page format agrees: 4 x (512 + 16) = 2112 bytes, a 2 KiB page plus its 64-byte spare.
- For offsets below `0xc00`, the register window ignores address bit 9, so `0x200` reads as `0x000`, `0x600` as `0x400` and `0xa00` as `0x800`.
- `0x400..0x5fc` is a second register set ([chapter 9, section 9.5](#95-the-second-nand-controller-register-set-0x400)) with the same layout and write masks. The boot ROM addresses the NAND controller of each channel at a stride of `0x400`. The CPU set-up writes the word `0x400` as 0.
- `0x100..0x1fc` is a further block of words that the sequencer can only read. The same block appears again at `0x300`, `0x500` and `0x700`. Its meaning is open.

`FMC_DATA_SEL` (`0x014`) holds one-hot fields:

| bits | meaning |
|---|---|
| 1:0 | data buffer being filled (`1` = data buffer 0, `2` = data buffer 1) |
| 5:4 | parity buffer being filled by a read (`0x10` or `0x20`) |
| 9:8 | data buffer being drained (`0x100` or `0x200`) |
| 13:12 | parity buffer feeding a NAND write (`0x1000` or `0x2000`) |

Two combined values occur, which alternate the two data buffers: `0x102` (fill data buffer 1, drain data buffer 0) and `0x201` (fill data buffer 0, drain data buffer 1).

Drain selects on a page read (device):

- With `FMC_DMA_CTRL` = 0, 1 or 8, the drain select `0x1e0` delivered correct data in every sector, and so did the combined values used as the original READ uses them: each sector read with `0x201` / `0x102` into data buffer 0 / 1, decoded there and copied out with `0x102` / `0x201` ([`dmactl`](14-evidence.md#142164-dmactl-run-37)).
- In `fmcbits`, `0x102` and `0x1e0` delivered wrong data in every sector and left `FMC_DMA_CTRL` reading `0xf` ([`fmcbits`](14-evidence.md#142143-fmcbits-run-35)). That read followed the case that wrote `FMC_DMA_CTRL` = `0xff`, a value that persists and makes every DMA fail (inferred cause).

The controller has two 512-byte data buffers and two parity buffers of four 16-byte slots each. The buffers belong to no chip enable (observed / 3p).

## 9.4 `FMC_STATUS` bits

| bit | mask | meaning | acknowledged by writing |
|---|---|---|---|
| 1 | `0x2` | command sent (`wait` event 0) | the same mask |
| 2 | `0x4` | address sent (`wait` event 1) | the same mask |
| 3 | `0x8` | data transfer done (`wait` event 3) | the same mask |
| 20 | `0x100000` | DMA done (`wait` event 2) | the same mask |
| 23 | `0x800000` | status byte latched (`wait` event 5) | the same mask |
| 27 | `0x8000000` | set after a sector is read from the chip if that sector had been programmed; clear if it was erased, [RD7](11-nand-operations.md#1147-rd7-chunk-ecc-flag) (not a `wait` event) | the same mask |
| all | `0x0ff00ffe` | bits 1..11 and 20..27: the flags above and the other bits in those two ranges, cleared at the start of every run | the same mask |

All flags are levels: a `wait` passes while its flag is set and does not clear it (device).

- After power-on, `FMC_STATUS` reads `0x000c3000` (bits 12, 13, 18 and 19; meaning open). These four bits are also set after a soft reset, and writing 1 to them does not clear them.
- A 16-byte data read from the chip (`FMCTRL1` = `0x32`) clears bits 13 and 19; a 512-byte data read (`0xe2`) clears all four. After the 16-byte read, a DMA into a data buffer or a parity buffer, a status latch and the set-up word set bits 13 and 19 again; a DMA out of a data buffer, an ECC encode, a command and a second-set data read do not ([`fmcstat2`](14-evidence.md#142165-fmcstat2-run-37)).
- With all four bits set, a command, an address, a status latch, a DMA, an ECC encode, a meta encode and a command in the second register set leave them set ([`fmcstat`](14-evidence.md#142144-fmcstat-run-35)).
- The second register set ([chapter 9, section 9.5](#95-the-second-nand-controller-register-set-0x400)) has the same flags in its own status word `0x448`. These flags drive `wait` events 6 (bit 1), 7 (bit 2), 8 (bit 20) and 9 (bit 3) (device, [chapter 7, section 7.1](07-events-and-waits.md#71-wait-events)).
- In some runs a status latch failed to raise `wait` event 5, and `FMC_STATUS` showed bit 24 (`0x01000000`) instead of bit 23.
  - In [`irqsrc`](14-evidence.md#142101-irqsrc-run-27) it happened in both status-latch programs that had an interrupt enable written (`0x044` = `0xffff` or `0x844` = `0x2f`), and not in the one without.
  - In [`op16`](14-evidence.md#14289-op16-run-23) it occurred in a status-latch program with no enable written.
  - In the fifteen programs of [`ev5en`](14-evidence.md#142109-ev5en-run-28) (five enable settings, three times each) it did not occur.
  - The cause is open ([chapter 17](17-open-questions.md#172-events-and-the-status-latch), item 12). One possible explanation, not tested, is a status latch made without a fresh arm of `wait` event 5 ([chapter 7, section 7.1](07-events-and-waits.md#71-wait-events)).
  - In `latchrep`, 160 latches (with and without `0x044` / `0x844` written), all set bit 23 and none set bit 24 ([`latchrep`](14-evidence.md#142141-latchrep-run-35)).

## 9.5 The second NAND controller register set (`0x400`)

Where it sits in the window: [chapter 3, section 3.2](03-cpu-memory-map.md#32-memory-map-the-register-window).

The second register set (`0x400..0x4fc`, CPU address `0x38a00400`) is a second NAND channel controller, laid out word for word like the first set, with the same write masks:

- `0x400` is its `FMC_CTRL0`, `0x404` its `FMCTRL1`, `0x408` its command byte, `0x40c` and `0x410` its address words, `0x414` its data select, `0x42c` its address count, `0x430` its length and `0x434` its DMA address;
- `0x448` is its status, `0x44c` its status byte, and `0x424`, `0x438`, `0x444`, `0x460..0x478` and `0x47c` are the remaining words.

Unlike `0x200..0x3fc`, it is not a mirror of the first set. Device results show this:

- The two sets held different values at the same time (command byte `0x008` = `0x70` while `0x408` = `0x90`; `0x048` = `0x008c3002` while `0x448` = `0x000c3002`).
- A READ ID through the second set changed only its own words.
- A DMA in the second set raised `wait` event 8 and not `wait` event 2.
- The two DMA address registers held different values.

Further device results:

- Its state machine runs: a READ ID through it sends the command (`0x448` bit 1) and the address cycles (bit 2) and moves the 8 bytes (bit 3; `0x424` counts them).
- Its flags drive `wait` events 6 (command), 7 (address), 8 (DMA) and 9 (transfer), as `wait` events 0, 1, 2 and 3 do for the first set. The `evjmp` instruction reacts to them.
- It has its own DMA engine and its own data buffers 0 and 1: system memory to buffer to system memory works through it with its own `FMC_CTRL0` bit 24 set. Its parity buffers 0 and 1 return a pattern DMA-ed into them; parity buffer 0 is shared with the first register set (inferred: after the second set's DMA into its parity buffer 0, the first set's parity buffer 0 read the second set's pattern, [`ch1buf`](14-evidence.md#142166-ch1buf-run-37)).
- No chip answered on its bus on any chip enable when only the channel-0 pins were configured; its FIFO port returned junk.

The second register set is channel 1 of the NAND controller, on its own pins. The boot ROM's NAND boot routine takes a channel number, 0 or 1 (`bootrom mem:0x20002e0c`: `channel < 2`). The routine reads pages from CE0 of that channel into SRAM with a page reader that addresses the registers at `channel * 0x400` ([RE-43](14-evidence.md#14343-re-43-boot-rom-nand-access)). The routine's pin set-up (`bootrom mem:0x20001830`; GPIO configuration words at `0x3cf00000 + port * 0x20`, function 2 per 4-bit field) differs per channel:

Which device the boot ROM boots from is set by two strap pins, GPIO port 3 pins 6 and 7 ([RE-47](14-evidence.md#14347-re-47-boot-rom-boot-device-selection)): 0 / 0 SPI flash on SPI port 0, 0 / 1 SPI flash on SPI port 2, 1 / 0 NAND channel 0, 1 / 1 NAND channel 1. On this unit both pins read 0, so it boots from the SPI flash ([`straps`](14-evidence.md#142169-straps-run-37)).

| channel | GPIO ports and pins switched to function 2 |
|---|---|
| 0 (`0x000` set) | port 8 all eight pins; port 9 two pins (fields 0 and 4); port 10 pins 0..3 |
| 1 (`0x400` set) | port 10 pins 4..7; port 11 pins 0..4; port 12 pins 0..3; port 7 pin 4 |

In the first device tests only the channel-0 pins were configured (ports 8, 9 and the lower half of port 10), so the channel-1 controller drove no pins and the FIFO port returned junk ([`ev8`](14-evidence.md#14281-ev8-run-18), [`chan1`](14-evidence.md#14257-chan1-run-15)). With the channel-1 pins switched as the boot ROM does, a READ ID through the `0x400` set read `0xff` bytes on CE1 to CE7 (an undriven bus) and nothing on CE0 (device). No chip answers on channel 1 of this board, so the nano 3G uses only channel 0 (inferred from the one board).

The original programs and the original CPU driver never use the second register set, apart from the original CPU driver writing its `FMC_CTRL0` (`0x400`) as 0 at start-up and power-off ([RE-37](14-evidence.md#14337-re-37-cpu-driver-initialisation)).

## 9.6 Status latch needs `FMC_DATA_SEL = 1`

After a write, `FMC_DATA_SEL` was still `0x100` from the data transfer. In that state the status latch returned `0x70` (the last command byte) instead of the chip's status. Bit 6 of `0x70` reads as "ready", so a poll ended while the chip was still programming. The next page's program command was then ignored, and those pages read back erased. With `FMC_DATA_SEL` = 1 ([C7](11-nand-operations.md#1117-c7-ready-procedure-status-poll)) the status byte is the chip's: `0xc0` when ready and `0x81` when busy.

The arming rule of `wait` event 5 and the other status-latch rules are in [chapter 7, section 7.1](07-events-and-waits.md#71-wait-events).

References: [chapter 14](14-evidence.md) (outcomes), [chapter 13](13-experiments.md) (what was done and why), [chapter 15](15-evidence-interpretation.md) (what the outcomes mean).

Related chapters: [chapter 3](03-cpu-memory-map.md), [chapter 7](07-events-and-waits.md), [chapter 10](10-ecc-engine.md), [chapter 11](11-nand-operations.md).
