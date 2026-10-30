# Chapter 1 — Overview and terms

Summary: What the FMSS engine is, the evidence classes and numbering used in these chapters, the terms, and the three hardware blocks (sequencer, NAND controller (FMC), ECC engine).

These documents describe how the FMSS engine of the iPod nano 3G (S5L8702) works: the sequencer coprocessor inside the NAND controller, the NAND controller (FMC) it drives, and the ECC engine. They cover the engine's behaviour, registers, instruction set, memory maps, ECC, events, timing and interfaces, and how the original firmware uses it. Each statement about the engine's behaviour comes with its evidence.

## 1.1 Evidence classes

The statements in the chapters rest on two sources:

- static analysis of the original programs and of the original CPU driver;
- device tests, which are small programs run on the sequencer of one unit ([chapter 13](13-experiments.md)).

Each statement carries one of the following evidence classes, in the text and in the finding list ([chapter 18](18-ledger.md)):

- **observed**: seen in the original programs or in the original CPU driver; the address is given in the finding list.
- **device**: measured on a unit.
- **inferred**: an interpretation of observed use or of device results.
- **3p**: a driver written by a third party, reported by its authors as tested on a device.
- **open**: not settled.

The outcomes themselves are in [chapter 14](14-evidence.md), under the IDs `D-<experiment>-<device run>` (device) and `RE-<n>` (reverse engineering of the original firmware). [Chapter 15](15-evidence-interpretation.md) states what each outcome is taken to show and which evidence class the statement has.

## 1.2 Numbering and labels

Chapters are numbered 1 to 18. Headings inside chapter N are numbered N.1, N.2, N.2.1 and so on. A cross-reference names the chapter and the section, for example [chapter 16, section 16.2](16-device-findings.md#162-settled-on-a-device). The entries of chapter 6 (opcode reference) are the exception: each is headed by its opcode and mnemonic and carries no number.

Chapter 11 labels the behaviours of the original programs `C1`..`C13` (common), `R1`..`R7` (ECC and data path), `ID1`..`ID5` (reset and ID), `RD1`..`RD11` (page read), `WR1`..`WR9` (page program), `ER1`..`ER4` (erase) and `FL1`..`FL3` (fault handling). Each label is defined in chapter 11 and is used as a cross-reference in other chapters. Finding numbers (`finding 55`) refer to the entries of [chapter 18](18-ledger.md). Names such as `C58`, `C4C` and `C14` are the sequencer control words at offsets `0xc58`, `0xc4c` and `0xc14` ([chapter 8](08-sequencer-control-registers.md)).

Conventions used throughout:

- Bits are written "bit N" or "bits H:L" (high bit first); register offsets and values are hexadecimal in code style (`0x38a00000`).
- Times are given in us and ms. Approximate values are marked with "about".
- "`wait` event N" is the status flag with number N in the list of flags that the `wait` instruction waits for ([chapter 7, section 7.1](07-events-and-waits.md#71-wait-events)). "`wait2` condition N" is the entry with number N in the separate list of conditions that the `wait2` instruction waits for ([chapter 7, section 7.2](07-events-and-waits.md#72-the-conditions-wait2-waits-for)). The two lists are different, so `wait` event N and `wait2` condition N with the same N are unrelated.
- "Original firmware", "original programs" and "original CPU driver" name Apple's firmware, the nine sequencer programs in it, and its NAND driver that runs on the ARM CPU. "Third-party driver" names the third-party driver of the evidence class 3p.

## 1.3 Terms

| term | meaning | source of the name |
|---|---|---|
| **NAND controller (FMC)** | flash memory controller: the SoC's NAND controller at `0x38a00000`. It produces command, address and data cycles on the NAND bus and contains the data buffers and the DMA. It has two channels, whose register sets are at offsets `0x000` and `0x400` ([chapter 9, section 9.5](09-nand-controller.md#95-the-second-nand-controller-register-set-0x400)) | register names of the SoC family (`FMCTRL0`, `FMCTRL1`, ...) as used by third-party headers |
| **FMSS engine** | the whole NAND hardware described here: the sequencer, the NAND controller and the ECC engine ([chapter 1, section 1.4](#14-the-three-blocks)) | the original firmware's own name (its log reads "FMSS sequence timed out", its driver functions are named `...fmss...`); the firmware never spells the name out (an expansion such as "flash memory sequencer subsystem" is a guess) |
| **sequencer** | the small coprocessor inside the NAND controller that runs programs of 8-byte instructions ([chapter 5](05-instruction-encoding.md)) | name used in these documents |
| **M1 engine** | the engine of the nano 3G described in these documents. Whether it is the same hardware version as the M2 engine is pending: the 3G's original programs use more opcodes than the 4G / 5G programs (`pld` and `pst`, opcodes `18` and `19`, [chapter 6](06-opcode-reference.md)) | name used in these documents, after the 4G/5G name M2 |
| **M2 engine** | the same kind of engine in the nano 4G / 5G | the 4G/5G firmware |
| **ECC engine** | error-correcting code (ECC) hardware at offset `0x800` that generates parity for 512-byte sectors and corrects them (how it works and where the parity is kept: [chapter 10, section 10.1](10-ecc-engine.md#101-how-error-correction-works)) | name used in these documents |
| **CE** | chip enable: the signal that selects one NAND chip (`FMC_CTRL0` bits 8:1) | NAND standard |
| **CS_IRQ** | the sequencer's flag word at `0xc0c` (bits for END, fault and others) | the original firmware calls it `FC_CS_IRQ`; what "CS" stands for is not stated |
| **event** | one of the status flags that `wait` can wait for, selected by a number: "`wait` event N" ([chapter 7, section 7.1](07-events-and-waits.md#71-wait-events)) | name used in these documents |
| **condition** | the number N of `wait2`: `wait2` condition N is interrupt source N of the CPU's interrupt controllers (VIC0 lines 0..31, VIC1 lines 32..63), and `wait2 N` waits until that line is raised ([chapter 7, section 7.2](07-events-and-waits.md#72-the-conditions-wait2-waits-for)) | name used in these documents |
| **armed** | a state of the NAND controller that `wait` event 5 depends on: a status latch sets the event's flag only while the event is armed, and each latch uses up the armed state ([chapter 7, section 7.1](07-events-and-waits.md#71-wait-events)) | name used in these documents |
| **VIC0, VIC1** | the CPU's two vectored interrupt controllers; "VIC1 raw bit 22" is bit 22 of VIC1's raw interrupt status (interrupt source 54), the line the original CPU driver watches for the end of a run ([chapter 3, section 3.4](03-cpu-memory-map.md#34-cpu-visible-interrupt-lines)) | SoC register names |
| **parameter RAM** | 32 words at `0xd00` that the CPU and the program both access; they hold a program's inputs and outputs ([chapter 4, section 4.4](04-sequencer-memory-map.md#44-parameter-ram)) | name used in these documents |

## 1.4 The three blocks

The NAND hardware at `0x38a00000` has three parts that share one register window ([chapter 3, section 3.2](03-cpu-memory-map.md#32-memory-map-the-register-window)):

| block | registers | what it does | who drives it |
|---|---|---|---|
| **sequencer** | control block `0xc00..0xc5c` ([chapter 8, section 8.1](08-sequencer-control-registers.md#81-sequencer-control-block)) | runs a program of 8-byte instructions fetched from system memory | the CPU starts and stops it through `FMSS_CTRL` (`0xc00`) |
| **NAND controller** (FMC) | `0x000..0x0fc` (channel 0), `0x400..0x4fc` (channel 1) ([chapter 9](09-nand-controller.md)) | puts commands, addresses and data on the NAND bus and moves data by DMA | whoever writes its registers: the sequencer program, or the CPU directly |
| **ECC engine** | `0x800..0x8fc` ([chapter 10](10-ecc-engine.md)) | generates parity for 512-byte sectors, checks them and corrects them | the same: the sequencer program or the CPU |

The sequencer does no NAND work itself: its program writes the NAND controller's and the ECC engine's registers, exactly as the original CPU driver does, and waits for their completion flags (`wait` events, [chapter 7, section 7.1](07-events-and-waits.md#71-wait-events)). How the sequencer is started and stopped, and how the CPU and the sequencer interact, is in [chapter 2](02-architecture.md).

The original firmware carries nine sequencer programs (ERASE, READ, READID, WRITE, RAWREAD, WRITE2P, WRITE2P_78, WRITE_78, WRITECACHE; [RE-2](14-evidence.md#1432-re-2-program-table)) and an original CPU driver that starts them ([RE-31](14-evidence.md#14331-re-31-cpu-driver-start)); both are described in the chapters that follow, together with the behaviour of the hardware they use.

## 1.5 Named parts of the data path

Data passes through the following parts on its way between system memory and a NAND chip. Each part is defined below by where it is, how big it is, how it is selected, what fills and empties it, and what is known about its contents over time. The other chapters use exactly these names. "Device" marks what was measured and "inferred" marks what follows from it.

**Where the buffers are.** The data buffers and the parity buffers are storage inside the NAND controller block of the SoC (register window at CPU address `0x38a00000`). They have **no address**: neither the CPU memory map nor the register window maps their bytes to any address range.

- No word of the register window is known to give access to a parity buffer: the read / write matrix of all 1024 window words, from the CPU and from the sequencer, found none that does (device, [`access`](14-evidence.md#142105-access-run-27)).
- The word `FMC_FIFO` (`0x080`, the **FIFO port**) is the only window word through which data from the chip has been read. The original READID program uses it in this order (observed, [RE-24](14-evidence.md#14324-re-24-readid-sequence); the third-party driver does the same):
  1. It reads 8 bytes from the chip (`FMC_DATA_SEL` = 1, `FMCTRL1` = `0x1c2`).
  2. It sets `FMC_DATA_SEL` = `0x100` and `FMCTRL1` = `0x340`.
  3. It reads `0x080` twice and obtains the 8 ID bytes in order.

  The FIFO port does not read a data buffer that a DMA filled (device, [`bufport`](14-evidence.md#142154-bufport-run-36)): after data buffer 0 or 1 (and parity buffer 0) was filled by DMA with a known pattern, 130 reads of `0x080` returned 0, with and without `FMCTRL1` = `0x340`. Each read advanced `FMC_28` by 4 (0, 128, 256, 384 after every 32 reads, wrapping at 512), so `FMC_28` is the FIFO port's read position. 128 writes to `0x080` after `FMCTRL1` = `0x340` changed 26 of the 128 words of data buffer 0. Whether the bytes of a READ ID are held in a data buffer or in separate storage is open.
- All other ways into and out of a buffer are transfers. They are the chip transfers and DMAs of the NAND controller (`FMCTRL1`), the ECC operations (`ECC_KICK`) and the metadata path (`FMC_METACTL`).

### 1.5.1 System memory

- **What:** the SoC's SDRAM and internal SRAM on the CPU bus, addressed with 32-bit bus addresses.
- **Used by the FMSS engine for:** the program (fetched starting at the address in `FMSS_PROG`), the per-request lists (read with `ldi`), the results (written with `sti`), and the data of every DMA (bus address in `FMC_DMA_ADDR`).
- **Details:** [chapter 3](03-cpu-memory-map.md), [chapter 4, section 4.2](04-sequencer-memory-map.md).

### 1.5.2 Data buffer 0 and data buffer 1

- **Where:** inside the NAND controller, belonging to channel 0 (`0x000` register set); no address (see "Where the buffers are" above). The CPU and a program reach their bytes only through a DMA, a chip transfer or an ECC operation; reads of the FIFO port returned 0 for them (%s). Channel 1 (`0x400` register set) has its own data buffers 0 and 1: a pattern DMA-ed into each through the `0x400` set came back unchanged ([`ch1buf`](14-evidence.md#142166-ch1buf-run-37)).
- **Size:** 512 bytes each: a DMA always moves exactly 512 bytes into or out of one, whatever `FMC_DNUM` holds (device, [`bufsize`](14-evidence.md#142103-bufsize-run-27)).
- **Selected by** one-hot fields, one for each direction of transfer (device):

  | operation | field | data buffer 0 | data buffer 1 |
  |---|---|---|---|
  | filled by a chip read (`FMCTRL1` = `0xe2`) or a DMA from system memory (`0x2e0`) | `FMC_DATA_SEL` bits 1:0 | `0x1` | `0x2` |
  | emptied by a chip write (`0xe4`) or a DMA to system memory (`0x100`, `0x1e0`) | `FMC_DATA_SEL` bits 9:8 | `0x100` | `0x200` |
  | read and corrected in place by an ECC operation | `ECC_KICK` bits 9:8 | `0x100` | `0x200` |

- **Contents over time:** a data buffer keeps its bytes between operations within one program (device: a DMA into the buffer, an encode and a decode in one program work on the same bytes). The set-up word at the start of a program (`FMCTRL1` = `0x0ff3f8e0`) clears the buffers (inferred: a decode run in a later program than its encode found nothing to check). The buffers are not tied to any one chip enable (observed / 3p).
- **Use:** each holds the 512 data bytes of one sector. The original programs alternate between the two (`FMC_DATA_SEL` `0x102` / `0x201`): one is filled while the other is emptied.

### 1.5.3 Parity buffer 0 and parity buffer 1

- **Where:** inside the NAND controller; no address and no known window port (see "Where the buffers are" above). DMAs through the `0x400` register set (channel 1) into parity buffer 0 and 1 came back unchanged, and after the one into parity buffer 0, channel 0's parity buffer 0 read back channel 1's pattern; so parity buffer 0 is one buffer shared by both channels (inferred, [`ch1buf`](14-evidence.md#142166-ch1buf-run-37)). A set-up word written to channel 1's `FMCTRL1` (`0x404`) left channel 0's parity buffer 0 reading all `0xff` (same evidence).
- **Size:** 64 bytes each, divided into four slots (1.5.4): a 512-byte DMA into a parity buffer wraps around within the 64 bytes, so the buffer ends up holding the last 64 bytes of the source (device, [`eccchunk`](14-evidence.md#142112-eccchunk-run-29)).
- **Selected by** (device):

  | operation | field | parity buffer 0 | parity buffer 1 | which slot |
  |---|---|---|---|---|
  | filled by a chip read of a chunk (`FMCTRL1` = `0x32`) | `FMC_DATA_SEL` bits 5:4 | `0x10` | `0x20` | byte offset in `FMC_24` (`0`, `0x10`, `0x20`, `0x30`) |
  | filled by a DMA from system memory (`0x2e0`) | `FMC_DATA_SEL` bits 5:4 | `0x10` | `0x20` | the whole buffer, wrapping |
  | emptied by a chip write of a chunk (`0x34`) | `FMC_DATA_SEL` bits 13:12 | `0x1000` | `0x2000` | byte offset in `FMC_28` |
  | emptied by a DMA to system memory (`0x100`) | `FMC_DATA_SEL` bits 13:12 | `0x1000` | `0x2000` | from slot 0 |
  | written by an encode, read by a decode | `ECC_KICK` bits 13:12 | `0x1000` | `0x2000` | `ECC_KICK` bits 17:16 = k (BCH); always slot 0 (Reed-Solomon) |
  | written by a meta encode, read by a meta decode | `FMC_METACFG` nibbles (0..3 = buffer 0, 4..7 = buffer 1; inferred) | | | the nibble value modulo 4 (inferred) |

- **Contents over time:** after the set-up word every byte reads `0xff` (device: unwritten slot bytes read `0xff` in every DMA out). Bytes not written by an operation keep their value until the next set-up word (device: a BCH encode leaves slot bytes 0..2 as the meta encode wrote them).
- **Use:** it holds the four chunks of one 2 KiB unit, one chunk per slot. The original programs use parity buffer 0 for all four sectors.

### 1.5.4 Slot

- **What:** slot k (k = 0..3) is bytes 16k to 16k + 15 of a parity buffer.
- **Holds:** the chunk (1.5.5) of one sector while it is moved between the chip and the ECC engine. Its contents per ECC method (which bytes hold metadata, which parity) are in [chapter 10, section 10.2](10-ecc-engine.md#102-the-methods-at-a-glance).
- **Boundaries:** a BCH operation uses the slot that `ECC_KICK` bits 17:16 name. A Reed-Solomon operation ignores that field and always uses slot 0; its 20-byte parity with 16 symbols fills slot 0 and bytes 0..3 of slot 1 (device, [chapter 10, section 10.4](10-ecc-engine.md#104-reed-solomon-common-to-all-three-strengths)).

### 1.5.5 Chunk

- **What:** the bytes that are stored on the chip in front of each 512-byte sector, in the page's byte stream. In the original layout a chunk is 16 bytes: 3 metadata bytes and 13 BCH parity bytes (device).
- **Path:** a chunk is written to the chip from a slot (`0x34`) and read from the chip into a slot (`0x32`); the transfer length is `FMC_DNUM` + 1 bytes (`FMC_DNUM` = `0xf`, giving 16 bytes, in the original programs).
- **Details:** [chapter 10, section 10.9](10-ecc-engine.md#109-what-is-stored-on-the-chip).

### 1.5.6 ECC area

- **What:** the ECC engine's window words `0x900..0x913` (20 bytes), which the CPU and the program can read (the program with `ld` or `pld`).
- **Contents:** after an encode, the parity just computed in reverse byte order (BCH 13 bytes, Reed-Solomon 10, 15 or 20) and, for some modes, further bytes. When no code is used, it holds the next value of a fixed sequence. A decode clears `0x900..0x90c`.
- **Details:** [chapter 10, section 10.13](10-ecc-engine.md#1013-registers).

### 1.5.7 Metadata registers

- **What:** `FMC_META0..2` (`0x060..0x068`), three 32-bit words, 12 bytes in all, holding the metadata of one 2 KiB unit.
- **Path:** a meta encode (`FMC_METACTL` = 1) copies bytes 3k..3k + 2 of the metadata registers into bytes 0..2 of slot k (k = 0..3; the bytes are numbered in the order of the little-endian words); a meta decode (`FMC_METACTL` = 2) copies them back. Which slots are used is set by `FMC_METACFG` ([chapter 11, R6](11-nand-operations.md#1126-r6-meta-layout-word)).

### 1.5.8 FIFO port (`FMC_FIFO`)

- **What:** the word `FMC_FIFO` (`0x080`). After a READ ID transfer (`FMCTRL1` = `0x1c2`, `FMC_DNUM` + 1 bytes, 8 in the original READID), each read of `0x080` with `FMCTRL1` = `0x340` and `FMC_DATA_SEL` = `0x100` returns the next 4 ID bytes. Each read advances `FMC_28` by 4 (its read position, 9 bits). It returned 0 for data buffers filled by DMA, so where the ID bytes are held is open ("Where the buffers are" above). [chapter 9, section 9.3](09-nand-controller.md#93-other-nand-controller-registers).

### 1.5.9 Status register

- **What:** `FMC_NSTAT` (`0x04c`), the most recent NAND status byte, delivered by a status latch (`FMCTRL1` = `0xca`, `wait` event 5). [chapter 7, section 7.1](07-events-and-waits.md#71-wait-events).

### 1.5.10 Chip page

- **What:** one page of a NAND chip: on this unit 4096 data bytes and 128 spare bytes, 4224 bytes in one byte stream ([chapter 10, section 10.9](10-ecc-engine.md#109-what-is-stored-on-the-chip)).
- **Path:** the chip loads a page into its own page register with `00h`, address cycles, `30h`; the controller then reads the page out byte by byte into a slot and a data buffer. A write sends the bytes into the chip's page register and `10h` programs them ([chapter 11](11-nand-operations.md)).

### 1.5.11 What the ECC engine touches

The ECC engine reads and writes only a data buffer and a slot of a parity buffer, and it writes the ECC area. It never reads or writes system memory or a chip. A sector reaches the chip only when the program sends the sector's slot (the chunk) and its data buffer with chip writes.

References: [chapter 13](13-experiments.md) (what was done and why), [chapter 14](14-evidence.md) (outcomes), [chapter 15](15-evidence-interpretation.md) (what the outcomes mean).

Related chapters: [chapter 2](02-architecture.md), [chapter 5](05-instruction-encoding.md), [chapter 9](09-nand-controller.md), [chapter 10](10-ecc-engine.md), [chapter 14](14-evidence.md).
