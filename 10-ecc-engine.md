# Chapter 10 — ECC engine

Summary: The ECC engine at `0x800`. How error correction works, the four correction methods (one BCH code and three strengths of a Reed-Solomon code) with how each is selected, how many bits or bytes each corrects and how it computes its parity, what is stored on the chip for each, the full `ECC_MODE` map, the status words, the modes without a code, the registers and the byte order.

## 10.1 How error correction works

The correction is done by a hardware engine, the ECC engine at `0x800`: it computes parity and corrects errors in its own logic. There are no ECC tables in memory, and neither the CPU nor the program supplies any. The generator of each code and its field arithmetic are built into the engine. They appear in no firmware image and in no memory table.

The engine works on one 512-byte sector at a time. For each sector it uses two parts of the NAND controller (FMC), named in [chapter 1, section 1.5](01-overview.md#15-named-parts-of-the-data-path):

- **data buffer 0** or **data buffer 1** (512 bytes each) holds the sector's data;
- **parity buffer 0** or **parity buffer 1** (64 bytes each) holds four 16-byte **slots**, one per sector of a 2 KiB unit; slot k holds the parity of sector k (and, for the BCH code, three metadata bytes).

**Metadata** here means the 12 bytes per 2 KiB unit that the CPU passes with a page besides its data (`FMC_META0..2`, [chapter 12](12-cpu-interface.md)). The metadata path stores 3 of them in each of the four slots (bytes 0..2), so they are written to the chip in each sector's chunk ([R6](11-nand-operations.md#1126-r6-meta-layout-word)). Whether a method also protects these 3 bytes against bit errors is a property of the method (table below).

The program moves data between system memory, the data and parity buffers and the chip; the engine only reads and writes a data buffer and a slot (`FMC_DATA_SEL`, [chapter 9, section 9.3](09-nand-controller.md#93-other-nand-controller-registers); the buffer rules are in [chapter 11, section 11.2](11-nand-operations.md#112-ecc-and-data-path-shared-by-read-and-program)).

- **Encode** (before writing): the engine reads the data buffer and writes the parity into the slot named in the `ECC_KICK` word. The program then sends the slot (as the chunk) and the data buffer to the chip.
- **Decode** (after reading): the program has put the stored data into a data buffer and the stored parity into the slot; the engine checks the data against the parity, corrects the data buffer in place if it can, and reports the result in `ECC_RESULT`.

The parity is stored on the chip next to the data, in the chunk in front of each sector ([chapter 10, section 10.9](#109-what-is-stored-on-the-chip)). The engine never touches the chip.

## 10.2 The methods at a glance

`ECC_MODE` (`0x814`) selects the method. It is written before every kick (a write to `ECC_KICK` that starts an operation). All values below were measured on the device. Column meanings:

- "corrects": the largest number of errors per 512-byte sector that a decode corrected in the tests. With one more error, the decode reported the sector uncorrectable.
- "protects metadata": whether the code also covers the 3 metadata bytes of the sector's slot.
- "slot after encode": what the encode leaves in the slot.

| method | `ECC_MODE` | code | parity bytes | corrects | protects metadata | slot after encode | details |
|---|---|---|---|---|---|---|---|
| BCH | `0x180` | binary BCH over GF(2^13) | 13 | **8 bits** anywhere in data, metadata and parity | yes, 3 bytes | bytes 0..2 metadata (left as they were), bytes 3..15 parity | [10.3](#103-bch-8-bits-per-sector) |
| RS-8 | `0x8100` | Reed-Solomon over GF(2^10), 8 parity symbols | 10 | **4 bytes** (any number of bits inside each) | no | slot 0 bytes 0..9 parity, bytes 10..15 not written; the slot field is ignored | [10.5](#105-reed-solomon-8-parity-symbols-4-bytes-per-sector) |
| RS-12 | `0x8104` | the same, 12 parity symbols | 15 | **6 bytes** | no | slot 0 bytes 0..14 parity, byte 15 not written; the slot field is ignored | [10.6](#106-reed-solomon-12-parity-symbols-6-bytes-per-sector) |
| RS-16 | `0x8108` | the same, 16 parity symbols | 20 | **8 bytes** | no | parity-buffer bytes 0..19 (slot 0 and bytes 0..3 of slot 1); the slot field is ignored | [10.7](#107-reed-solomon-16-parity-symbols-8-bytes-per-sector) |
| none | `0`, `0x80`, `0xe00000` (bit 8 and bit 15 clear) | no code | – | nothing (every decode reports uncorrectable) | – | slot not written | [10.12](#1012-modes-with-bit-8-clear-no-code) |

The original programs use only the BCH mode `0x180` ([chapter 11](11-nand-operations.md)). Other values that select the same BCH code and strength: `0x100`, `0x1c0`, `0x181`, `0x380`, `0x780`; the same RS-8 code: `0x8180`. Mode bits that act on every method (the full map is in [chapter 10, section 10.10](#1010-ecc_mode-bit-map)):

- bit 1 with RS-8: the encode writes two more slot bytes, a copy of slot bytes 8 and 9 (one sample);
- bit 4 = 32-bit byte swap: the engine reverses the bytes of each 32-bit word of the data it reads and of the slot it writes (BCH and RS);
- bits 20:16 = data length L (1..31; 0 = 512): only the first L bytes of the data buffer are encoded and checked, and the rest of the sector counts as `0xff` (BCH and RS);
- bits 24:21: no effect on the code; they change `ECC_DONE` bit 5 ([chapter 10, section 10.11](#1011-ecc_done-and-ecc_result)).

### 10.2.1 Running one operation

The same steps for every method (device; the original programs use them for the BCH code, [chapter 11, section 11.2](11-nand-operations.md#112-ecc-and-data-path-shared-by-read-and-program)):

| step | write / wait | value |
|---|---|---|
| 1 | `ECC_DONE` (`0x840`) | `0x1ff`: clear all flags |
| 2 | `ECC_MODE` (`0x814`) | the method's value (table above) |
| 3 | `ECC_KICK` (`0x80c`) | bit 0 = decode or bit 1 = encode; bits 9:8 select the data buffer (`0x100` data buffer 0, `0x200` data buffer 1); bits 13:12 select the parity buffer (`0x1000` parity buffer 0, `0x2000` parity buffer 1); bits 17:16 select slot k (BCH only; a Reed-Solomon operation always uses slot 0) |
| 4 | `wait` event 4 | the operation is done (`ECC_DONE` bit 0) |
| 5 | read `ECC_RESULT` (`0x810`), decode only | bits 19:16 = number corrected (BCH: bits; RS: bytes); `0x11` = uncorrectable, data left as it was |
| 6 | `ECC_DONE` | write 1 to acknowledge |

Examples of `ECC_KICK` values: an encode of sector k from data buffer 0 into slot k of parity buffer 0 is `0x1102 | k << 16`; the matching decode is `0x1101 | k << 16`. The kick words used by the original programs are listed in [chapter 10, section 10.13](#1013-registers).

The encode and the decode of a sector must use the same `ECC_MODE` value.

A decode run in a later program than its encode found nothing to check. The set-up word (`FMCTRL1` = `0x0ff3f8e0`) that starts each program clears the data buffers and parity buffers (inferred). When data is read from the chip, the program first reloads the slot from the stored chunk ([chapter 10, section 10.9](#109-what-is-stored-on-the-chip)).

## 10.3 BCH: 8 bits per sector

Selected by `ECC_MODE` = `0x180` (bit 8 set, bit 15 clear). This is the code of every page the original firmware writes.

### 10.3.1 Strength

- **8 bit errors per sector, in any positions** of the 4224-bit code word (512 data bytes, 3 metadata bytes, 13 parity bytes). With 1 to 8 flipped bits a decode corrects the data and reports `ECC_RESULT` = `n << 16`; with 9 or more it reports `0x11` and leaves the data as it was ([`eccstr2`](14-evidence.md#142108-eccstr2-run-28): 1..8 corrected, 9..12 not).
- The count covers the whole code word: errors in metadata and parity bytes are corrected and counted like data errors (4 data + 4 parity bits = 8 corrected; 5 data + 3 metadata bits = 8 corrected; 9 parity bits uncorrectable, [`eccchunk2`](14-evidence.md#142118-eccchunk2-run-30)).
- The same holds for errors stored on the chip: they decode the same way (8 data bits corrected, 9 not, 2 metadata bits corrected and counted, an all-zero chunk uncorrectable, [`rawwrite`](14-evidence.md#142114-rawwrite-run-29)).
- Eight bits in one byte count as 8 errors: the BCH code counts bits, not bytes.

### 10.3.2 How the code works

The code is a binary BCH code over GF(2^13) that corrects 8 bits. Every parity measured on the device matches this definition:

- field polynomial x^13 + x^4 + x^3 + x + 1 (`0x201b`);
- generator g(x) of degree 104 (13 bytes), roots alpha^1 .. alpha^16: `g = 0x115f914e07b0c138741c5c4fb23` (bit k = coefficient of x^k);
- message: the 512 data bytes followed by the slot's 3 metadata bytes (slot bytes 0..2), most significant bit first (data byte 0 bit 7 is the highest power); so a bit error in a metadata byte on the chip is corrected like one in the data;
- parity = message × x^104 mod g(x), 13 bytes, most significant byte first.

An encode takes the metadata bytes from the slot itself, not from `FMC_META0..2`. The metadata path ([R6](11-nand-operations.md#1126-r6-meta-layout-word)) places them in slot bytes 0..2 before the encode, and the encode writes slot bytes 3..15 only.

The metadata is part of the message (an unwritten slot holds `0xff`). All-zero data therefore gives non-zero parity (zeros: `22 24 98 95 03 a2 a2 f3 10 b3 a3 77 3e` in `0x900..0x90c`).

All twelve encodes measured match this definition (zeros, ones, a ramp, eight single-bit sectors at bits 0, 1, 2, 7, 8, 9, 4088 and 4095, and the strength test's sector; [`eccstr2`](14-evidence.md#142108-eccstr2-run-28), [`eccpar`](14-evidence.md#142102-eccpar-run-27)). Twenty sectors of pages written by the original firmware are code words of this code with their real metadata in slot bytes 0..2 ([`rawpage`](14-evidence.md#142113-rawpage-run-29)).

### 10.3.3 What an encode and a decode leave

- After an encode: the 13 parity bytes in the slot (bytes 3..15) and, in reverse order, in the ECC-area words `0x900..0x90c` (the last word holds one byte). `0x990` reads 0 ([`eccregs`](14-evidence.md#142145-eccregs-run-35)).
- After a decode: the data buffer corrected in place; `0x900..0x90c` cleared; `0x9a4` counts the decodes. `ECC_RESULT` = `n << 16` (n = corrected bits, 0..8) or `0x11`.
- `ECC_DONE` = `0x33` after an encode and `0x35` after a decode ([chapter 10, section 10.11](#1011-ecc_done-and-ecc_result)).

## 10.4 Reed-Solomon: common to all three strengths

`ECC_MODE` bit 15 (with bit 8) selects a Reed-Solomon code instead of the BCH code; bits 3:2 select its strength (0 = 8, 1 = 12, 2 = 16 parity symbols; 3 = as 0). The original programs never use it. The following description matches the device data:

- **Field:** GF(2^10), primitive polynomial x^10 + x^3 + 1 (`0x409`).
- **Symbols:** each data byte is one symbol (values 0..255 in a 10-bit field), and data byte 0 is the coefficient of the highest power. A sector is 512 data symbols; the code word is 520, 524 or 528 symbols, well inside the field's limit of 1023.
- **Generator:** g(y) = (y - 1)(y - a)...(y - a^(2t-1)) with 2t = 8, 12 or 16 parity symbols, where a is a root of x^10 + x^3 + 1 (the field's primitive element). For 8 symbols the coefficients (y^0..y^7, then y^8 = 1) are `0x190 0x299 0x2a5 0x01d 0x3ee 0x1ab 0x30a 0x0ff`.
- **Parity:** m(y) × y^(2t) mod g(y), where m(y) is the polynomial of the 512 data symbols. The result is 2t symbols of 10 bits.
  - The ECC area `0x900..` holds them little-endian, 10 bits each, packed into 10, 15 or 20 bytes.
  - The slot receives the same bytes in reverse order ([chapter 10, section 10.14](#1014-byte-order)).
- **The slot field is ignored.** A Reed-Solomon encode writes its parity at byte 0 of the parity buffer that `ECC_KICK` names (slot 0), whatever slot bits 17:16 name, and a decode reads it from there ([`rschip`](14-evidence.md#142161-rschip-run-37)):
  - an RS-16 encode with slot field 3 wrote its 20 bytes at bytes 0..19 of parity buffer 0, and a decode with slot field 3 then corrected 2 bad bytes;
  - in pages written with one encode per sector (slot fields 0..3, each slot sent to the chip as that sector's chunk) and read back the same way, sector 0 decoded and sectors 1..3 were reported uncorrectable.

  So the parity of only one sector at a time can be in a parity buffer. BCH uses the slot field ([chapter 10, section 10.3](#103-bch-8-bits-per-sector)).
- **Only the 512 data bytes are protected.** The Reed-Solomon code covers the data alone; the 3 metadata bytes of the slot are not part of it (all-zero data gives all-zero parity, whatever the slot holds).
- **What it corrects:** up to t bad bytes (t = 4, 6, 8), whatever the number of flipped bits inside each byte. A decode reports `ECC_RESULT` = `n << 16` with n = the number of corrected bytes, or `0x11` (data left as it was) when more bytes are bad. A burst that crosses a byte boundary counts as two bytes (a 10-bit burst across two bytes: `0x20000`, [`b15ecc`](14-evidence.md#142128-b15ecc-run-32)).
- **Trace:** after an encode with 8 symbols, `0x90a..0x913` hold a trace of the encoder (the encoder computes the parity in a shift register of 8 symbols; the trace is the value of its highest symbol just before each of the last 8 data symbols was fed in; 38 of 38 sectors); with 12 symbols the 5 bytes after the parity hold further bytes.
- **`0x990`:** a 16-bit value, repeated in both halves of the word, that depends on the data and not on the strength (zeros `0000`, all ones `fc36`, the ramp `c68b` with 8 and with 16 symbols, [`eccregs`](14-evidence.md#142145-eccregs-run-35), [`eccmode4`](14-evidence.md#142129-eccmode4-run-33)). Its meaning is open.

How the code was identified:

- Fits of a cyclic code over GF(2), tried for every bit and byte order, failed.
- Within every sampled byte, the parity of bit k+1 is exactly 2 × the parity of bit k in a degree-10 field. This singled out `0x409` among all 60 primitive polynomials of degree 10.
- With bytes as symbols, the parity of every sample then follows from repeated multiplication by y modulo one polynomial of degree 2t.
- Matched: 43 of 43 sampled sectors for 8 symbols (zeros; one bit at bytes 0, 1, 2, 3, 4, 8, 16, 32, 64, 128, 256, 384, 448, 496, 504, 508, 510, 511; bits 15:0 and 4080..4095; two bits together; the ramp `i × 37 + 11`), and 10 of 10 each for 12 and 16 symbols ([`finale`](14-evidence.md#142130-finale-run-34)).

The length field (bits 20:16) applies to this code too: with L = 31 the parity is that of the first 31 bytes with the rest `0xff` ([`eccmode4`](14-evidence.md#142129-eccmode4-run-33)).

## 10.5 Reed-Solomon, 8 parity symbols: 4 bytes per sector

- **Select:** `ECC_MODE` = `0x8100` (`0x8180` selects the same code).
- **Strength:** 4 bad bytes. 1, 2 and 4 bytes in error were corrected (`ECC_RESULT` `0x10000`, `0x20000`, `0x40000`); 8, 9, 10, 12, 16 and 17 bad bytes, and bursts of 40, 80 and 90 bits, were reported uncorrectable ([`b15ecc`](14-evidence.md#142128-b15ecc-run-32)). The same on the chip: a sector stored with 4 bad bytes was corrected (`0x40000`), one with 5 was reported uncorrectable ([`rschip`](14-evidence.md#142161-rschip-run-37)).
- **Parity:** 8 symbols = 80 bits = 10 bytes. Bytes 0..9 of slot 0 hold them; bytes 10..15 are not written (they keep `0xff` from the set-up word). For the ramp data, the slot holds `522ee44c7cc113acebcd ffffffffffff` ([`finale`](14-evidence.md#142130-finale-run-34)).
- **With `ECC_MODE` bit 1** (`0x8102`): slot bytes 10 and 11 receive a copy of bytes 8 and 9 (`…ebcd ebcd ffffffff`, one sample).

## 10.6 Reed-Solomon, 12 parity symbols: 6 bytes per sector

- **Select:** `ECC_MODE` = `0x8104`.
- **Strength:** 6 bad bytes. 4 and 6 bad bytes were corrected (`ECC_RESULT` `0x40000`, `0x60000`); 7 bad bytes were reported uncorrectable (`0x11`, `ECC_DONE` `0x3d`, data left as it was) ([`rsdecode`](14-evidence.md#142133-rsdecode-run-35)). The same on the chip: 6 stored bad bytes corrected (`0x60000`), 7 uncorrectable ([`rschip`](14-evidence.md#142161-rschip-run-37)).
- **Parity:** 12 symbols = 120 bits = 15 bytes. Bytes 0..14 of slot 0 hold them; byte 15 is not written. For the ramp data, the slot holds `186d249847a87441d927d4965ed6e4 ff`. The ECC area holds the same 15 bytes reversed, followed by 5 more bytes (`a8012da801`) ([`rsdecode`](14-evidence.md#142133-rsdecode-run-35), [`finale`](14-evidence.md#142130-finale-run-34)).

## 10.7 Reed-Solomon, 16 parity symbols: 8 bytes per sector

- **Select:** `ECC_MODE` = `0x8108`.
- **Strength:** 8 bad bytes. 4 and 8 bad bytes were corrected (`0x40000`, `0x80000`); 9 and 12 were reported uncorrectable ([`rsdecode`](14-evidence.md#142133-rsdecode-run-35)).
- **Parity:** 16 symbols = 160 bits = 20 bytes, more than a 16-byte slot. The encode writes all 20 bytes at bytes 0..19 of the parity buffer: slot 0 and **bytes 0..3 of slot 1**, also when the kick names slot 3. For the ramp data the parity buffer then holds `4e2923086f4cfc4b7b77718291507950` `ffa89d05` `ff…`, and the ECC area `0x900..0x913` holds the same 20 bytes reversed ([`rsdecode`](14-evidence.md#142133-rsdecode-run-35), [`rschip`](14-evidence.md#142161-rschip-run-37)).
- A decode reads the 20 bytes from the same place: with the slot field 3 it corrected 2 bad bytes from the parity at bytes 0..19 ([`rschip`](14-evidence.md#142161-rschip-run-37)).

## 10.8 What each method corrects

| method | parity bytes | corrects | as bits, scattered (one per byte) | as bits, clustered | evidence |
|---|---|---|---|---|---|
| BCH | 13 | 8 bits | 8 | 8 | [`eccstr2`](14-evidence.md#142108-eccstr2-run-28) |
| RS-8 | 10 | 4 bytes | 4 | up to 32 (4 whole bytes) | [`b15ecc`](14-evidence.md#142128-b15ecc-run-32) |
| RS-12 | 15 | 6 bytes | 6 | up to 48 | [`rsdecode`](14-evidence.md#142133-rsdecode-run-35) |
| RS-16 | 20 | 8 bytes | 8 | up to 64 | [`rsdecode`](14-evidence.md#142133-rsdecode-run-35) |

- No method corrects more than 8 scattered bits per 512-byte sector.
- For errors spread one bit per byte, BCH and RS-16 correct the same number (8); for errors clustered in a few bytes, RS-16 corrects more.
- How the "x bits per y bytes" of a chip datasheet maps onto these numbers depends on the error model of the chip, not on the engine.
- Every case beyond the limits above was reported uncorrectable with the data unchanged. This includes 240 BCH sectors with 9 to 64 random bad bits and 240 RS-8 sectors with 5 to 40 random bad bytes: none was reported corrected or error-free ([`miscorr`](14-evidence.md#142162-miscorr-run-37)).

## 10.9 What is stored on the chip

The engine does not write the chip; the program sends each slot and each data buffer to the chip in the order and with the lengths it chooses (`FMC_DNUM` + 1 bytes per transfer, [chapter 9, section 9.3](09-nand-controller.md#93-other-nand-controller-registers)). What ends up on the chip is therefore the layout the program uses.

**This unit's chip.** Four chip enables answer READ ID `ec d5 14 b6 74` ([`chan1`](14-evidence.md#14257-chan1-run-15)). The restore OS has a chip table row for this ID. The row holds 4096, 128, 8 and 128 in the fields read as blocks per chip enable, pages per block, 512-byte sectors per page and spare bytes per page, and 3872 user blocks ([RE-45](14-evidence.md#14345-re-45-chip-table-row-of-this-units-chip); field meanings inferred). A page therefore holds 8 × 512 = 4096 data bytes plus 128 spare bytes, 4224 bytes in all, the length the original programs transfer per page ([chapter 11, section 11.2](11-nand-operations.md#112-ecc-and-data-path-shared-by-read-and-program)).

### 10.9.1 BCH (the original layout, device)

Every 512-byte sector is stored as its 16-byte chunk followed by the 512 data bytes, all in the page's normal byte stream starting at column 0. A 2 KiB unit is four such sectors (2112 bytes), and a 4 KiB page is two units:

```
| chunk 0 (16) | data 0 (512) | chunk 1 (16) | data 1 (512) | chunk 2 (16) | data 2 (512) | chunk 3 (16) | data 3 (512) |   = 2112 bytes, twice per page
chunk k = [3 metadata bytes][13 BCH parity bytes]
```

- 8 × (16 + 512) = 4224 bytes: the layout fills the page, spare area included, exactly.
- The 3 metadata bytes of chunk k are bytes 3k..3k+2 of the unit's 12 metadata bytes ([R6](11-nand-operations.md#1126-r6-meta-layout-word)); they are protected by the code.
- Read back: the program puts the chunk into slot k and the data into a data buffer, then runs a decode ([chapter 11, section 11.2](11-nand-operations.md#112-ecc-and-data-path-shared-by-read-and-program)).
- Pages written by the original firmware confirm this layout: all twenty sectors read raw are code words with their metadata ([`rawpage`](14-evidence.md#142113-rawpage-run-29)); pages built by the CPU and stored errors behave as the code predicts ([`rawwrite`](14-evidence.md#142114-rawwrite-run-29)).
- An erased (never written) page reads all `0xff`, which is not a code word; `FMC_STATUS` bit 27 tells whether a read carried ECC data at all ([RD7](11-nand-operations.md#1147-rd7-chunk-ecc-flag)).

### 10.9.2 Reed-Solomon

Pages were written with RS-8 and RS-12 by the engine, one encode per sector followed by sending the slot and the data, and read back ([`rschip`](14-evidence.md#142161-rschip-run-37)):

- The chunk stored on the chip is the slot as it was: RS-8 `2a5c0a397c8a9f585350` followed by six `0xff`, RS-12 15 parity bytes followed by one `0xff`.
- Because a Reed-Solomon encode always writes slot 0 ([chapter 10, section 10.4](#104-reed-solomon-common-to-all-three-strengths)), only sector 0, whose chunk was sent from slot 0, carried its parity; sectors 1..3, sent from slots 1..3, were reported uncorrectable. A page with Reed-Solomon parity in every sector needs each sector's chunk sent from slot 0 right after its encode, and loaded into slot 0 before its decode (inferred).
- Stored errors in sector 0 decoded as in the buffers: RS-8 corrected 4 bad bytes and not 5, RS-12 corrected 6 and not 7.

The table is computed from the parity sizes and the page size (inferred). "Bytes per page" is 8 × (512 + parity bytes), plus 3 metadata bytes per sector for BCH. "Free slot bytes" are the slot bytes that the encode leaves available for metadata:

| method | parity bytes | bytes per page | fits 4224 bytes | free slot bytes |
|---|---|---|---|---|
| BCH | 13 (+ 3 metadata) | 4224 | yes, exactly | 3, protected |
| RS-8 | 10 | 4176 | yes (48 bytes left) | 6 (bytes 10..15), not protected |
| RS-12 | 15 | 4216 | yes (8 bytes left) | 1 (byte 15), not protected |
| RS-16 | 20 | 4256 | **no** (32 bytes too many) | none; the parity also overwrites bytes 0..3 of slot 1 |

- The Reed-Solomon parity starts at slot byte 0, the same place where the metadata path puts the metadata. The metadata layout of the BCH chunks therefore does not carry over.
- A read with a Reed-Solomon mode needs the stored parity back at bytes 0.. of the parity buffer (for RS-16 bytes 0..19) before the decode.
- Zero data gives zero Reed-Solomon parity, so an erased page (all `0xff`) is not a code word in these modes either (not tested).

## 10.10 `ECC_MODE` bit map

`ECC_MODE` has 25 writable bits (mask `0x01ffffff`) and reads `0x00e00000` at power-on. The map was measured by encoding a known sector with `0x180` plus one more bit at a time, and then with other bit combinations ([`eccmode`](14-evidence.md#142111-eccmode-run-29), [`eccmode2`](14-evidence.md#142119-eccmode2-run-30), [`eccmode3`](14-evidence.md#142126-eccmode3-run-31), [`finale`](14-evidence.md#142130-finale-run-34), [`done5`](14-evidence.md#142147-done5-run-35)):

| bits | effect |
|---|---|
| bit 0 | no difference seen in parity or slot |
| bit 1 | with RS-8: the slot gets two more bytes, a copy of slot bytes 8 and 9 (one sample); with BCH no effect |
| bits 3:2 | with bit 15: Reed-Solomon strength, 0 = 8 parity symbols (4 bytes), 1 = 12 (6 bytes), 2 = 16 (8 bytes), 3 = as 0; with BCH no effect |
| bit 4 | 32-bit byte swap: the data buffer is read and the slot written with each 32-bit word byte-reversed (both codes; the code itself is unchanged) |
| bits 7:5 | no difference seen in parity or slot (bit 7 tested with bit 8 set) |
| bit 8 | slot path: the encode writes its parity into the named slot and a decode reads it from there; with bits 8 and 15 clear no code is computed ([chapter 10, section 10.12](#1012-modes-with-bit-8-clear-no-code)) |
| bits 14:9 | no difference seen in parity, slot or `ECC_DONE` |
| bit 15 | Reed-Solomon code ([chapter 10, section 10.4](#104-reed-solomon-common-to-all-three-strengths)) instead of BCH |
| bits 20:16 | data length L (5 bits, 1..31; 0 = 512): only the first L bytes of the data buffer are encoded and checked, the rest counts as `0xff` (BCH: L = 1, 2, 3, 4, 5, 8, 12, 16 and 31 matched exactly; a decode with L = 16 ignored a change at byte 100 and corrected 8 bits in byte 5; RS: L = 31 matched) |
| bits 24:21 | no difference seen in parity or slot; they clear `ECC_DONE` bit 5 ([chapter 10, section 10.11](#1011-ecc_done-and-ecc_result)) |
| bits 31:25 | not writable |

`ECC_MODE` = `0x1ffffff` (every writable bit set) acts as RS-8 with the byte swap and L = 31, because bits 3:2 both set select 8 symbols ([`eccmode4`](14-evidence.md#142129-eccmode4-run-33)).

## 10.11 `ECC_DONE` and `ECC_RESULT`

`ECC_RESULT` (`0x810`) after a decode:

| value | meaning |
|---|---|
| `0` | no error |
| `n << 16` (bits 19:16) | n errors corrected: bits (BCH, 1..8) or bytes (Reed-Solomon, 1..8) |
| `0x11` (bits 0 and 4) | uncorrectable; the data buffer is left as it was |

What an encode does to `ECC_RESULT` (device):

- In the modes without a code it becomes `0x11`.
- An encode in the same code as the last decode leaves it unchanged: a BCH encode after a BCH decode (`0x30000`, `0x40000` stayed), a Reed-Solomon encode after a Reed-Solomon decode (`0x20000` stayed) ([`rsresult`](14-evidence.md#142160-rsresult-run-37), [`eccmode2`](14-evidence.md#142119-eccmode2-run-30)).
- An encode in the other code changed it: a Reed-Solomon encode after a BCH decode with 3 corrected bits set it to 0 (from `0x30000`), and a BCH encode after a Reed-Solomon decode set it to `0x30000` (from 0 and from `0x20000`) ([`rsresult`](14-evidence.md#142160-rsresult-run-37)). In `eccmode2` [`eccmode2`](14-evidence.md#142119-eccmode2-run-30), Reed-Solomon encodes after a BCH decode set it to `0x20000`. Where these values come from is open.

`ECC_DONE` (`0x840`) after an operation: bit 0 is the done flag that `wait` event 4 waits for. The register reads `0x33` after an encode and `0x35` after a decode. Two more bits vary:

- **Bit 3 = the last decode was uncorrectable.** A failed decode sets it. It stays set through later encodes, other programs and the `ECC_DONE` = `0x1ff` clear. The next decode that succeeds clears it. An uncorrectable decode reads `0x3d`; an encode after it reads `0x3b`.
- **Bit 5 is clear** when any one of `ECC_MODE` bits 21..24 is set (in `done5`, encode followed by a decode, [`done5`](14-evidence.md#142147-done5-run-35); in `done5b`, an encode alone, an encode followed by a decode, and an encode after a decode, each after a soft reset, [`done5b`](14-evidence.md#142159-done5b-run-37)).
  - In [`done5`](14-evidence.md#142147-done5-run-35) this held on all three base modes (`0x180`, `0x8100`, `0`), after the encode and after the decode (`ECC_DONE` = `0x13`, `0x15`, `0x1b`, `0x1d`).
  - Bits 9..14 and 16..20 alone leave it set.
  - In [`eccmode2`](14-evidence.md#142119-eccmode2-run-30), encodes with bit 21, 22 or 23 alone on `0x180` read `0x33` (bit 5 set); in that series bit 24 alone cleared it. Why `eccmode2` differs from `done5` and `done5b` is open.

## 10.12 Modes with bit 8 clear: no code

The modes `0`, `0x80` and `0xe00000` (device). With bits 8 and 15 clear the engine computes no parity code. An encode writes the next value of a fixed sequence into the ECC area instead:

- The slot is not written; it keeps the `0xff` that the set-up word put there.
- 13 bytes are written into `0x900..0x90c` (the last word holds one byte), and a 16-bit value, repeated in both halves of the word, into `0x990`.
- The written values do not depend on the data in the data buffer or on which of the three mode values is used. The first such encode writes value 1 of the sequence, the second value 2, and so on.
- A decode in these modes reports every sector uncorrectable (`ECC_RESULT` = `0x11`), including a sector whose data and slot are unchanged, because the slot holds no parity. A decode does not advance the sequence and leaves `0x990` unchanged.
- In `seqrestart`, none of the actions tried between encodes restarts the sequence ([`seqrestart`](14-evidence.md#142146-seqrestart-run-35)).
  - The values continued in order (values 7 to 18, [`seqrestart`](14-evidence.md#142146-seqrestart-run-35)) across a soft reset, the sequencer reset (`FMSS_CTRL` = `0x30000`), writes of `ECC_MODE`, `ECC_KICK` (`0`), `ECC_DONE`, `0x940..0x950` and `FMC_CTRL0`, and BCH encodes and decodes.
  - The first such operation gave value 1 in `eccregs`, value 1 in `eccmode4` and value 14 in `finale` ([`eccregs`](14-evidence.md#142145-eccregs-run-35), [`eccmode4`](14-evidence.md#142129-eccmode4-run-33), [`finale`](14-evidence.md#142130-finale-run-34)); in `seqfirst`, started right after a power cycle of the unit, it gave value 2 ([`seqfirst`](14-evidence.md#142150-seqfirst-run-36)).
  - Which event restarts it is open.
- `0x9a0` read `4f888131` in every encode-only run, and `f80fffff`, `ffffffff` or `ffffff80` after a decode in these modes. `0x9a4` (decode count) and `0x9c0` = 1 behave as in the other modes.

The 61 measured values of the sequence, numbered n (the 13 bytes in ECC-area byte order, and the 16-bit `0x990` value):

| n | `0x900..0x90c` | `0x990` |
|---|---|---|
| 1 | `233c853f32c8ce55a1c321a1fc` | `f0f4` |
| 2 | `2e1c79573386228532df6ca2da` | `5bb8` |
| 3 | `7b55880c275b93c33a7204f807` | `4734` |
| 4 | `611570dd25c74b621d4b9efe4b` | `f6ad` |
| 5 | `e87c56af4cfa3be376f15bb3e4` | `3d6d` |
| 6 | `dcfca60c49c28aa039836fbe7c` | `7353` |
| 7 | `edd42e2dda3f79ae9517f0dc37` | `aa32` |
| 8 | `a62f0baf90c8082570138c3f12` | `83d6` |
| 9 | `c47f1becb633ef38283ab3fa84` | `b5b6` |
| 10 | `528950e823dd0c2ee3334b3ccf` | `854a` |
| 11 | `b5d2b4ab2eacd0192881214ff7` | `ce7f` |
| 12 | `993f23a304711734be92d1c260` | `ca68` |
| 13 | `5788eb241e93af5b28f7042410` | `ca68` |
| 14 | `2ca900f00bae330c7f30f0c62a` | `52fa` |
| 15 | `b0c691ff3e6a42d353fb5a0bcb` | `73a8` |
| 16 | `4684475615107a7cfd75b3cebe` | `cb7d` |
| 17 | `5da0a18c3e1f8acedf03f2ac68` | `e61b` |
| 18 | `b1250ddf69ebfa90821e212783` | `7881` |
| 19 | `a49605af7f7209f9bc12b71a3a` | `df16` |
| 20 | `5f6698cd901dfb497dc805f4f8` | `6e53` |
| 21 | `56fb4de8fda80f967a303d769f` | `828f` |
| 22 | `83e1b2e862f0f8fb82654c520f` | `fdda` |
| 23 | `91db19a3b89a11448d953d56c0` | `6aed` |
| 24 | `18152367c7acec9306decbe7f5` | `1f8a` |
| 25 | `1f9ab13532fe2de062de3c167e` | `dc9a` |
| 26 | `0d07c4bdcd92d74f7549d07515` | `1a4b` |
| 27 | `20e225dd66b046a4c6a92a6f17` | `3f63` |
| 28 | `04d8cecd9969b2fbe987f3a8c1` | `a51b` |
| 29 | `5e120d0ccf2c902c8e46069dc5` | `e889` |
| 30 | `359d1fe870186a9fabfaa0eb7d` | `0007` |
| 31 | `8109986bdd922e3164784b8075` | `2039` |
| 32 | `74ec7966e37cc95a54e0129410` | `8af2` |
| 33 | `1cc57661b8694006cbe5c54300` | `38f9` |
| 34 | `f60eb57ac4b58fd1abd5766bca` | `69f6` |
| 35 | `265cab74729f9d6895ded8c4eb` | `f860` |
| 36 | `d130e886cba011cb2f5eaa6c6a` | `de0f` |
| 37 | `7195d49aa7f535b95248f63329` | `7adb` |
| 38 | `bcb796bb950d3ef25ca9079a3f` | `ed88` |
| 39 | `fcfcef834da77616a685bf24b9` | `46c8` |
| 40 | `66b96bc129576180ba475c7794` | `1ca1` |
| 41 | `c5d45d74d885e34434fe38f38c` | `a8f7` |
| 42 | `f15f55f11065cc680d7aff54d6` | `0281` |
| 43 | `b784399bf3c0c9e11009365ce7` | `3552` |
| 44 | `df922891620196b96201b91352` | `dcc1` |
| 45 | `5324f145a44a9dab59e72b0230` | `0727` |
| 46 | `a0f31794c74e3117c61721644f` | `3cd8` |
| 47 | `b89ea43d4ad92733b0db04478b` | `7865` |
| 48 | `5e31699e8dd17f4a8f3a118b75` | `db4a` |
| 49 | `4d10cb08d779410e18424e34e8` | `027b` |
| 50 | `a2b4948a19efe2f01d60715500` | `89e6` |
| 51 | `a70d1462ed388c744871dbd22e` | `0def` |
| 52 | `5abf6fa33192d88538d5b1e9eb` | `f613` |
| 53 | `50cd6e72d83d058d93f7e5e6b6` | `2136` |
| 54 | `89535d3520efbf63095f246929` | `8b06` |
| 55 | `9db75f97f3b004725f1a8c7793` | `68c7` |
| 56 | `0c71fcdc429262a311ab1b91b9` | `3f12` |
| 57 | `07423d5da4aa078cc6c15f55d8` | `a2dd` |
| 58 | `25cf7acac6efcb2e5ba370988d` | `b98a` |
| 59 | `33a9f8c90b9e0170f576f8104e` | `6466` |
| 60 | `77b377e7ce149935ceb3a68ae5` | `6440` |
| 61 | `7884b72515701e84e9f8a36277` | `3c0f` |

Value 13's `0x990` word (`ca68`) was read after a decode, which does not change it; value 12 has the same word. Evidence: [`eccmode2`](14-evidence.md#142119-eccmode2-run-30), [`eccmode4`](14-evidence.md#142129-eccmode4-run-33) (values 1 to 13), [`finale`](14-evidence.md#142130-finale-run-34) (values 14 to 61), [`seqrestart`](14-evidence.md#142146-seqrestart-run-35) (values 7 to 18 again, in order).

What generates the sequence is not known. Checked and ruled out:

- All 61 values are distinct.
- Read as one bit stream (either byte and bit order), the sequence has a linear complexity of about half its length (3171 to 3173 of 6344 bits). Each bit position over the 61 values has a linear complexity of 27 to 34. So no short linear feedback shift register produces the sequence.
- Consecutive values are not one register multiplied by a fixed power of x modulo the BCH generator (checked at the roots a and a^3, both byte orders).
- The `0x990` values are not a 16-bit Galois LFSR stepped 1, 8, 13, 16 or 104 times per operation (all 65535 polynomials).

## 10.13 Registers

| offset | name | meaning |
|---|---|---|
| `0x800` | | reads `0x20060808` and ignores writes (device; an identification word, perhaps a date) |
| `0x80c` | `ECC_KICK` | start an operation: bit 0 decode, bit 1 encode, bits 9:8 select the data buffer (one-hot `0x100` or `0x200`), bits 13:12 select the parity buffer (`0x1000` or `0x2000`), bits 17:16 select the slot 0..3 (BCH; ignored by Reed-Solomon, which uses slot 0) |
| `0x810` | `ECC_RESULT` | after a decode: `n << 16` corrected (BCH bits, RS bytes), `0x11` uncorrectable ([chapter 10, section 10.11](#1011-ecc_done-and-ecc_result)) |
| `0x814` | `ECC_MODE` | the method and its options ([chapter 10, section 10.2](#102-the-methods-at-a-glance), full map [10.10](#1010-ecc_mode-bit-map)); the original programs write `0x180` (BCH) before every kick |
| `0x840` | `ECC_DONE` | bit 0 = done (`wait` event 4); write `0x1ff` to clear all before a kick, `1` to acknowledge; bit 3 = a decode was uncorrectable (stays set), bit 5 ([chapter 10, section 10.11](#1011-ecc_done-and-ecc_result)) |
| `0x844` | | interrupt enable of the ECC sources (6 bits, mask `0x2f`; 0 after power-on): with it non-zero, pending `ECC_DONE` bits raise `CS_IRQ` bit 6 and the CPU's interrupt source 43 (VIC1 line 11; device, [`w2vic`](14-evidence.md#142171-w2vic-run-38)) |
| `0x900..0x913` | ECC area | the parity of the last encode (BCH: 13 bytes, RS: 10 / 15 / 20 bytes) or the sequence value of the modes without a code; cleared by a decode |
| `0x940..0x950` | | five read/write words; values 0, all ones and `0x55555555` changed neither the parity nor a decode ([`eccregs`](14-evidence.md#142145-eccregs-run-35)) |
| `0x990` | | 16-bit value repeated in both halves: 0 after a BCH encode, data-dependent after an RS encode, the sequence value without a code |
| `0x9a4` | | decode count |

A decode corrects the data buffer in place; the DMA of the corrected data follows the decode (observed order in the original programs and in the third-party driver). A BCH encode writes slot bytes 3..15 and leaves bytes 0..2 as the metadata path put them ([`eccchunk`](14-evidence.md#142112-eccchunk-run-29), [`rawwrite`](14-evidence.md#142114-rawwrite-run-29)).

Kick words used by the original programs. Slot `k` is in bits 17:16, and data buffer 1 is selected where bit 9 is set:

- decode: `0x1101`, `0x11201`, `0x21101`, `0x31201`;
- encode: `0x1102`, `0x11202`, `0x21102`, `0x31202`.

The third-party driver also uses decode kicks with parity buffer 1 selected (`0x2000`). This shows that any combination of buffers and slot is accepted.

**Parity buffer by DMA (device).** A parity buffer is 64 bytes (four 16-byte slots). A DMA into it (`FMC_DATA_SEL` = `0x10` or `0x20`, `FMCTRL1` = `0x2e0`) still moves 512 bytes. The data wraps over the 64-byte buffer, so the buffer ends up holding the last 64 bytes of the source (slot 0 = source bytes 448 to 463). A DMA out of it (`FMC_DATA_SEL` = `0x1000` or `0x2000`, `FMCTRL1` = `0x100`) starts at slot 0 ([`eccchunk`](14-evidence.md#142112-eccchunk-run-29)).

**Raw writes (device).** A chunk can also be sent from a data buffer (`FMC_DNUM` = `0xf`, `FMC_DATA_SEL` = `0x100`, `FMCTRL1` = `0xe4`): only `FMC_DNUM` + 1 bytes go to the chip. A page with chunks built by the CPU is therefore written without the ECC engine at all, and reads back as written ([`rawwrite`](14-evidence.md#142114-rawwrite-run-29)).

**Why 512 bytes.** A DMA always moves 512 bytes, whatever `FMC_DNUM` says. With a length of 256, the drain (data buffer to system memory) still wrote 512 bytes. With lengths of 513 to 2048, only 512 bytes arrived ([`bufsize`](14-evidence.md#142103-bufsize-run-27)). The DMA unit of the controller is therefore 512 bytes, and the ECC methods all work on 512-byte sectors.

## 10.14 Byte order

The FMSS engine is little-endian: instruction fields, memory words and the words of the ECC area are little-endian. The parity bytes in the slot and on the chip are stored most significant byte first. Details:

- BCH parity: 13 bytes, most significant byte first in the slot (bytes 3..15) and on the chip; the ECC-area words `0x900..0x90c` hold the same 13 bytes in reverse order.
- Reed-Solomon parity: the ECC area holds the symbols little-endian, 10 bits each; the slot receives the same 10, 15 or 20 bytes in reverse order, starting at slot byte 0.
- `ECC_MODE` bit 4 swaps the bytes of each 32-bit word of the data buffer read and of the slot written, for both codes; the code itself is unchanged.
- The metadata path takes `FMC_META0..2` little-endian and places bytes 3k to 3k+2 in slot k.

The interpretation of why the parity order differs from the rest, and of what `ECC_MODE` bit 4 is for, is given in [chapter 15, section 15.7](15-evidence-interpretation.md#157-ecc-codes-and-modes) (inferred).

References: [chapter 14](14-evidence.md) (outcomes), [chapter 13](13-experiments.md) (what was done and why), [chapter 15](15-evidence-interpretation.md) (what the outcomes mean).

Related chapters: [chapter 5](05-instruction-encoding.md), [chapter 7](07-events-and-waits.md), [chapter 9](09-nand-controller.md), [chapter 11](11-nand-operations.md).
