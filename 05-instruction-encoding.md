# Chapter 5 — Instruction encoding

Summary: The 8-byte instruction word, what each field means for each opcode, the opcode summary table, the ALU operand rule and the byte order. Details per opcode are in [chapter 6](06-opcode-reference.md); field widths are in [chapter 4, section 4.1](04-sequencer-memory-map.md#41-window-offsets-and-target-widths).

## 5.1 Encoding

```
word0 = opcode << 24 | R << 16 | arg        (arg: u16)
word1 = imm                                  (u32)
```

The instruction is stored little-endian; `word0` is bytes +0..+3 and `word1` is bytes +4..+7. The fields mean:

- `R` names a register (0..7) or an event number.
- `arg` is a window offset (in register-access instructions) or a register number (in register-operand instructions).
- `imm` is an immediate value, a mask or a branch target.

Every instruction is 8 bytes, whatever the opcode. A program is an array of instructions, and branch and jump targets are byte offsets into it. The bytes lie in memory as follows (bus order); the sequencer fetches from `FMSS_PROG + pc`:

```
byte:   +0      +1      +2      +3      +4      +5      +6      +7
        arg[7:0] arg[15:8] R    opcode  imm[7:0] imm[15:8] imm[23:16] imm[31:24]
```

For example `movi r6, #0x600d600d` is the bytes `00 00 06 05 0d 60 0d 60`, and `wr [0xc58], #4` is `58 0c 00 01 04 00 00 00`.

## 5.2 Operand map

What each field means for each opcode. **used**: the field carries the operand named. **ignored**: the field was set to different values on a device (such as `0xff` and offset-like values, for fields the original programs leave 0) and nothing changed. **0**: the field must be 0, because other values change the behaviour. Every field of every valid opcode has been tested; none is left unknown. Register numbers 8..255 in a register field are described in [chapter 4, section 4.3](04-sequencer-memory-map.md#43-register-file-r0r7-the-pc-and-register-numbers-above-7).

| op | mnemonic | `R` (byte +2) | `arg` (bytes +0..1) | `imm` (bytes +4..7) |
|---|---|---|---|---|
| `00` | `end` | 0 (a non-zero value faults, as an illegal opcode) | 0 (a non-zero value faults) | ignored |
| `01` | `wr` | ignored | used: window offset | used: value |
| `02` | `st` | used: source register | used: window offset | ignored |
| `03` | `ldi` | used: destination register | used: register holding the bus address | ignored |
| `04` | `ld` | used: destination register | used: window offset | used: AND mask |
| `05` | `movi` | used: destination register | ignored | used: value |
| `06` | `mov` | used: destination register | used: source register | ignored |
| `07` | `wait` | used: event number = which hardware condition to wait for ([chapter 7, section 7.1](07-events-and-waits.md#71-wait-events); 8 bits) | ignored | ignored |
| `08` | `nop` | ignored | ignored | ignored |
| `09` | `wait2` | used: the number N of the interrupt source to wait for (VIC0 line N for N = 0..31, VIC1 line N - 32 for N = 32..63; [chapter 7, section 7.2](07-events-and-waits.md#72-the-conditions-wait2-waits-for)) | ignored | ignored |
| `0a`..`0d`, `13`, `14` | ALU | used: destination (and first source in the register form) | used: source register | used: immediate; 0 selects the register form |
| `0e` | `bnz` | used: tested register | ignored | used: branch target (byte offset) |
| `0f` | `jr` | ignored | used: register holding the target | ignored |
| `10` | `sig` | ignored | ignored | ignored |
| `11` | `sti` | used: source register | used: register holding the bus address | ignored |
| `12` | `pause` | ignored | ignored | ignored |
| `15` | `tmwait` | ignored | ignored | ignored |
| `16` | `evjmp` | ignored | ignored | ignored |
| `17` | `bz` | used: tested register | ignored | used: branch target (byte offset) |
| `18` | `pld` | used: destination register | used: register holding the window offset | ignored |
| `19` | `pst` | used: source register | used: register holding the window offset | ignored |
| `1a`..`fe` | (illegal) | ignored | ignored | ignored |
| `ff` | `nop2` | ignored | ignored | ignored |

Field widths (device): `R` is decoded in all 8 bits, window offsets are 12 bits, and branch targets use more bits; the details are in [chapter 4, section 4.1](04-sequencer-memory-map.md#41-window-offsets-and-target-widths).

## 5.3 Opcodes

The mnemonics of the opcodes that no original program uses (`nop`, `wait2`, `jr`, `sig`, `pause`, `tmwait`, `evjmp`, `nop2`) are names given in these documents after their measured behaviour; the original programs give no names for them.

`wait2` is named for behaving exactly like `wait` (same waiting state, timer and fault). Its register field selects an interrupt source of the CPU instead of one of `wait`'s events: `wait2 N` continues once interrupt source N (VIC0 line N for N = 0..31, VIC1 line N - 32 for N = 32..63) is raised, and numbers 64..255 never pass. See [chapter 7, section 7.2](07-events-and-waits.md#72-the-conditions-wait2-waits-for).

All 256 opcode values were run on a device, each in three operand forms ([chapter 16, section 16.2](16-device-findings.md#162-settled-on-a-device)). The **source** column says where the description of an opcode comes from:

- **original + device**: the original programs use it (encoding and use observed there), and device tests confirm its behaviour.
- **device only**: no original program uses it; everything known comes from device tests.
- **device (illegal)**: it faults in every form tried.

An opcode that stalls for the full timeout (about 4.87 ms) and then faults looks the same as a wait whose condition is never met. "Illegal" therefore means "no condition was found that makes the opcode do anything else".

| op | mnemonic | semantics | source | notes |
|---|---|---|---|---|
| `00` | `end` | stop, raise `CS_IRQ` bit 0; r6 is the result | original + device | `R` and `arg` 0 (a non-zero value faults), `imm` ignored; `SEQ_RESULT` = r6 |
| `01` | `wr [REG], #imm` | `REG = imm` | original + device | write to the register window (adds 37 ns to the instruction time) |
| `02` | `st [REG], rR` | `REG = rR` | original + device | |
| `03` | `ldi rR, [rA]` | `rR = mem32[rA]` | original + device | system memory, bus address in rA; little-endian; an unaligned address is rounded down to the word |
| `04` | `ld rR, [REG], mask` | `rR = REG & mask` | original + device | `mask` in `imm`; full word = `0xffffffff` |
| `05` | `movi rR, #imm` | `rR = imm` | original + device | |
| `06` | `mov rR, rA` | `rR = rA` | original + device | |
| `07` | `wait N` | block until `wait` event N is set ([chapter 7, section 7.1](07-events-and-waits.md#71-wait-events)), else time out after about 4.87 ms | original + device | `R` = event number; all 8 bits count (16, 27, 139 and 255 do not act as events 0, 11 or 15); argument and immediate ignored; events are levels (a flag stays set while its condition holds) and a wait does not clear them |
| `08` | `nop` | no operation | device only | no visible effect in any operand form; costs one instruction time |
| `09` | `wait2 N` | like `wait`, but N selects an interrupt source of the CPU ([chapter 7, section 7.2](07-events-and-waits.md#72-the-conditions-wait2-waits-for)): it continues once source N is raised in the raw status of VIC0 (N = 0..31) or VIC1 (N = 32..63); otherwise it waits for the timeout and faults like an unmet `wait` | device only | `arg` and `imm` ignored; not a wait for chip ready and not a mask test. Details: [chapter 6, opcode `09`](06-opcode-reference.md#09-wait2) |
| `0a` | `and` | ALU, see [chapter 5, section 5.5](#55-alu-operand-rule) | original + device | |
| `0b` | `or` | ALU | original + device | |
| `0c` | `add` | ALU | original + device | wraps mod 2^32 |
| `0d` | `sub` | ALU | original + device | wraps mod 2^32 |
| `0e` | `bnz rR, target` | `if rR != 0: pc = imm` | original + device | tests all 32 bits; argument ignored; the target is used as given (not rounded to a multiple of 8) |
| `0f` | `jr rA` | jump through a register: `pc = r(arg)` | device only | byte offset from the program start, not rounded; immediate ignored; no return address is kept, so together with `movi rX, #return` it provides calls and returns, and it also provides computed jumps |
| `10` | `sig` | signal the CPU: set `CS_IRQ` bit 1, continue | device only | a flag the CPU can see while the program runs |
| `11` | `sti [rA], rR` | `mem32[rA] = rR` | original + device | little-endian; unaligned rounded down |
| `12` | `pause` | stop and hand control to the CPU (`CS_IRQ` bit 3); the CPU continues the program by writing the PC and a start word | device only | fields ignored; details: [chapter 6, opcode `12`](06-opcode-reference.md#12-pause) |
| `13` | `shl` | ALU, logical left | original + device | a count of 32 or more gives 0 |
| `14` | `shr` | ALU, logical right | original + device | a count of 32 or more gives 0 |
| `15` | `tmwait` | wait for the timeout timer `C14` | device only | if the timer is non-zero: reload it with `0xff` and wait about 4.85 ms until it reaches 0; if the timer is 0: reload it with `0xff` at once ([chapter 8, section 8.1.6](08-sequencer-control-registers.md#816-fmss_c14-0xc14-the-timeout-timer)) |
| `16` | `evjmp` | event jump: wait until any event flag is set (in either NAND controller (FMC) register set, in the ECC engine or in the status flags), then jump to the fault vector in `C4C`; if no flag is set within the timeout, a fault is raised | device only | fields ignored; no fault is raised when it jumps; the flag stays set. Details: [chapter 6, opcode `16`](06-opcode-reference.md#16-evjmp) |
| `17` | `bz rR, target` | `if rR == 0: pc = imm` | original + device | as `bnz` |
| `18` | `pld rR, [rA]` | `rR = window word at byte offset rA` | original + device | 3G only; the originals use parameter RAM offsets only; the offset wraps at 12 bits and reaches every word of the window, r0..r7 included |
| `19` | `pst [rA], rR` | `window word at byte offset rA = rR` | original + device | 3G only; as `pld` |
| `1a..fe` | | illegal | device (illegal) | every value in the range stalls for the timeout and then faults; the opcode byte is decoded in all 8 bits (see "Opcode decoding" below) |
| `ff` | `nop2` | no operation | device only | as `08`: an all-ones word (erased or `0xff`-filled memory) is a no-op, not an end |

**Opcode decoding.** The opcode byte counts in all 8 bits. `20`, `40` and `80` are `00` (`end`) with one high bit set. A decoder that looked only at the low bits would treat them as `end`. On the device they stall and fault like every other illegal value, and all three behave alike. `21` (`01` with bit 5 set) likewise does not act as `wr`. No instruction has a second encoding ([chapter 6, the opcode values without a meaning](06-opcode-reference.md#1afe-and-other-values-without-a-meaning)).

Instruction time (device): the time is set by fetching the 8-byte instruction, 217 ns from SDRAM and 138 ns from internal SRAM. A register-window access adds about 35 ns, and `ldi` and `sti` add 120 to 220 ns ([chapter 16, section 16.2](16-device-findings.md#162-settled-on-a-device)).

## 5.4 Illegal opcodes

The values `1a..fe` are the illegal opcodes: they fault in every form tried (table above and [chapter 6](06-opcode-reference.md#1afe-and-other-values-without-a-meaning)). Opcodes `09` (`wait2`) and `16` (`evjmp`) fault in most tests and so resemble illegal opcodes, but each has a condition under which it passes. They are valid opcodes ([chapter 6](06-opcode-reference.md#09-wait2), [chapter 7](07-events-and-waits.md)). Register numbers above 7 in a register field are described in [chapter 4, section 4.3](04-sequencer-memory-map.md#43-register-file-r0r7-the-pc-and-register-numbers-above-7).

## 5.5 ALU operand rule

- Immediate form (`imm != 0`): `rR = r(arg) OP imm`. The source register is in `arg`, the destination in `R`.
- Register form (`imm == 0`): `rR = rR OP r(arg)`.
- An immediate of 0 is indistinguishable from the register form, so no instruction encodes "`rD = rA op #0`"; `mov` performs that copy.
- The original programs have no unconditional jump. They branch on a register known to be non-zero, or they pair `bz` and `bnz` on one register. The hardware does have a jump through a register, opcode `0f` ([chapter 5, section 5.3](#53-opcodes)), which also gives computed jumps and subroutine returns.
- There is no compare instruction. The original programs test equality by subtracting and branching on the difference.

## 5.6 Byte order

The FMSS engine is little-endian throughout, with one exception in the ECC engine's parity:

- Instruction fields and words: the instruction is stored little-endian ([chapter 5, section 5.1](#51-encoding)); `ldi` and `sti` access little-endian 32-bit words ([chapter 4, section 4.5](04-sequencer-memory-map.md#45-system-memory-and-the-program)); the parameter and result words are little-endian words ([chapter 12](12-cpu-interface.md)).
- ECC area: the words `0x900..0x90c` hold the parity bytes in little-endian word order, which is the reverse of the order in the parity slot ([chapter 10, section 10.14](10-ecc-engine.md#1014-byte-order)).
- Parity in the slot and on the flash: the parity bytes are stored most significant byte first ([chapter 10, section 10.14](10-ecc-engine.md#1014-byte-order)).

The reading of why the parity order differs (inferred) is in [chapter 15, section 15.7](15-evidence-interpretation.md#157-ecc-codes-and-modes).

References: [chapter 14](14-evidence.md) (outcomes), [chapter 13](13-experiments.md) (what was done and why), [chapter 15](15-evidence-interpretation.md) (what the outcomes mean).

Related chapters: [chapter 4](04-sequencer-memory-map.md), [chapter 6](06-opcode-reference.md), [chapter 7](07-events-and-waits.md), [chapter 10](10-ecc-engine.md).
