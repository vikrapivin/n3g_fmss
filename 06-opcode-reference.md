# Chapter 6 — Opcode reference

Summary: One entry per valid opcode, in opcode order, followed by one entry for the opcode values that have no meaning. The entries are headed by the opcode and its mnemonic and are not numbered. The summary table is in [chapter 5, section 5.3](05-instruction-encoding.md#53-opcodes).

## 6.1 Conventions and measurement notes

The fields are those of [chapter 5, section 5.1](05-instruction-encoding.md#51-encoding): `R` (8 bits), `arg` (16 bits) and `imm` (32 bits). Statements marked "device" give behaviour measured on a unit; times are per instruction with the program in SDRAM unless stated. Each entry says whether the original programs use the opcode ([RE-4](14-evidence.md#1434-re-4-instruction-encoding-and-opcode-set-used-by-the-programs) lists the opcodes that the nine original programs contain). The mnemonics of the opcodes that no original program uses (`08`, `09`, `0f`, `10`, `12`, `15`, `16`, `ff`) are names given in these documents after the measured behaviour. The source of a description is **original + device** (used by the original programs and confirmed on a device) or **device only** (no original program uses it).

## 6.2 Opcodes

### `00` `end`

- Stop the program.
- Encoding: `R` and `arg` are 0 in the original programs. The original programs encode `end` as two zero words.
- If `R` = 1, if `arg` = 1, or if both are set, the instruction does not end the program. It faults like an illegal opcode instead (it runs the fault vector and sets `CS_IRQ` bit 2) (device).
- `imm` is ignored: `0x12345678` ended the program normally. The two words of the `end` instruction, including `imm`, are left in `0xc40` and `0xc44` (device).
- Effect: the program stops. `CS_IRQ` bit 0 is set, and `SEQ_RESULT` (`0xc30`) shows the value of r6. Afterwards `FMSS_PC` reads 0, `C08` returns to 0 and the timer `C14` stops counting.
- If `C58` bit 0 is set, an `end` also sets `C5C` and leaves a latch. With the latch set, every later `end` runs the fault vector once ([chapter 8, section 8.1.15](08-sequencer-control-registers.md#8115-fmss_c58-0xc58-fault-routing)).
- If the one-shot is armed by `C58` bit 1, one `end` is diverted to the fault vector (device).
- Original programs: used by every original program.
- Evidence: [D-fields0-28](14-evidence.md#142106-fields0-run-28), [D-oneshot-8](14-evidence.md#14236-oneshot-run-8), [D-pending-5](14-evidence.md#14226-pending-run-5), [D-pending-6](14-evidence.md#14230-pending-run-6), [D-regmap-1](14-evidence.md#1421-regmap-run-1), [D-regmap-6](14-evidence.md#14228-regmap-run-6), [D-regmap-7](14-evidence.md#14233-regmap-run-7), [RE-4](14-evidence.md#1434-re-4-instruction-encoding-and-opcode-set-used-by-the-programs).

### `01` `wr`

- Write an immediate value to a register of the register window.
- Syntax: `wr [REG], #imm`.
- Encoding: `arg` = window offset, `imm` = value; `R` is ignored. The window offset selects a register of the NAND controller (FMC), of the ECC engine, of the sequencer block or in the parameter RAM. The offset is 12 bits: bits 15:12 of `arg` are ignored (device).
- Effect: writes `imm` to the register at that offset. For offsets below `0xc00`, bit 9 of the offset is ignored ([chapter 3, section 3.2](03-cpu-memory-map.md#32-memory-map-the-register-window)).
- The write has the register's own side effects. A command byte written to `FMC_CMD` is sent to the chip. A code written to `FMCTRL1` starts an operation.
- Timing: about 37 ns more than `movi` (device).
- Original programs: used for every fixed register value.
- Evidence: [D-opcost-15](14-evidence.md#14262-opcost-run-15), [D-fwidth-27](14-evidence.md#142100-fwidth-run-27), [D-fields0-28](14-evidence.md#142106-fields0-run-28), [RE-4](14-evidence.md#1434-re-4-instruction-encoding-and-opcode-set-used-by-the-programs).

### `02` `st`

- Store a sequencer register to a register of the register window.
- Syntax: `st [REG], rR`.
- Encoding: `arg` = window offset, `R` = source register; `imm` is ignored.
- Effect: writes the value of register rR to the window register.
- With `R` from 8 to 255, `st` stores the last value that passed through the register file: the value the previous instruction wrote into a register, or the register value the previous instruction read (r5 after `st [..], r5`; r0 after `nop` or `wr`, whose register field is 0); `wait` leaves it unchanged (device, [D-stprev-36](14-evidence.md#142156-stprev-run-36); details in [chapter 4, section 4.3](04-sequencer-memory-map.md#43-register-file-r0r7-the-pc-and-register-numbers-above-7)).
- Timing: as `wr`.
- Original programs: used (control words built in a register, list pointers into parameter RAM).
- Evidence: [D-regnum-15](14-evidence.md#14259-regnum-run-15), [D-regsweep-35](14-evidence.md#142149-regsweep-run-35), [D-fwidth-27](14-evidence.md#142100-fwidth-run-27), [D-fields0-28](14-evidence.md#142106-fields0-run-28), [D-opcost-15](14-evidence.md#14262-opcost-run-15).

### `03` `ldi`

- Load a word from system memory.
- Syntax: `ldi rR, [rA]`.
- Encoding: `arg` = register holding a bus address, `R` = destination; `imm` is ignored.
- Effect: a 32-bit little-endian load from system memory. An unaligned address is rounded down to the word boundary (device).
- If `arg` is a register number from 9 to 255, r0 supplies the address. If `arg` is 8, the load returns the word `e90d1c00`, which looks like code, as a load from a low address does (device, [D-regsweep-35](14-evidence.md#142149-regsweep-run-35)).
- Timing: about 170 ns more than `movi` from SDRAM, 218 ns more from internal SRAM (device).
- Original programs: used to read the lists of values supplied for each request (chip-enable lists, DMA address lists).
- Evidence: [D-memory-1](14-evidence.md#1424-memory-run-1), [D-regnum-15](14-evidence.md#14259-regnum-run-15), [D-regsweep-35](14-evidence.md#142149-regsweep-run-35), [D-opcost-15](14-evidence.md#14262-opcost-run-15), [D-fields0-28](14-evidence.md#142106-fields0-run-28).

### `04` `ld`

- Read a window register and keep only the bits selected by a mask.
- Syntax: `ld rR, [REG], mask`.
- Encoding: `arg` = window offset, `imm` = mask (`0xffffffff` keeps the whole word), `R` = destination register.
- Effect: rR = (window register) AND mask. Reading a status register does not clear any flag (device).
- Timing: as `wr`.
- Original programs: used for status tests and read-modify-write of `FMC_CTRL0`.
- Evidence: [D-alu-1](14-evidence.md#1423-alu-run-1), [D-alu-2](14-evidence.md#14210-alu-run-2), [D-alu-3](14-evidence.md#14216-alu-run-3), [D-evclear-14](14-evidence.md#14253-evclear-run-14), [D-fwidth-27](14-evidence.md#142100-fwidth-run-27), [RE-9](14-evidence.md#1439-re-9-control-word-and-chip-enable-changes).

### `05` `movi`

- Load an immediate.
- Syntax: `movi rR, #imm`.
- Encoding: `R` = destination, `imm` = value; `arg` is ignored.
- Effect: rR = imm. Register numbers above 7 behave differently (device, [D-regsweep-35](14-evidence.md#142149-regsweep-run-35)):
  - `R` = 8 jumps to `imm`, because r8 is the program counter.
  - `R` = 10 loads all eight registers r0..r7.
  - `R` = 9 and `R` = 11..255 change no register and no window word.
- Timing: the base instruction time, 217 ns (SDRAM) / 138 ns (SRAM) (device).
- Original programs: used (constants, markers, loop counters).
- Evidence: [D-r8-16](14-evidence.md#14273-r8-run-16), [D-r8-17](14-evidence.md#14279-r8-run-17), [D-regnum-15](14-evidence.md#14259-regnum-run-15), [D-opcost-15](14-evidence.md#14262-opcost-run-15), [D-fields0-28](14-evidence.md#142106-fields0-run-28).

### `06` `mov`

- Copy a register.
- Syntax: `mov rR, rA`.
- Encoding: `R` = destination, `arg` = source register; `imm` is ignored.
- Effect: rR = r(arg). Register numbers above 7 behave differently (device, [D-regsweep-35](14-evidence.md#142149-regsweep-run-35)):
  - As a source, `arg` = 8 reads the byte offset of the current instruction.
  - As a source, `arg` = 10 reads the destination register.
  - As a source, `arg` = 9 and `arg` = 11..255 read r0.
  - As a destination, `R` = 8 jumps and `R` = 10 loads all eight registers, as for `movi`.
- Original programs: used.
- Evidence: [D-regnum-15](14-evidence.md#14259-regnum-run-15), [D-r8-16](14-evidence.md#14273-r8-run-16), [D-r8-17](14-evidence.md#14279-r8-run-17), [D-fields0-28](14-evidence.md#142106-fields0-run-28).

### `07` `wait`

- Wait for an event.
- Syntax: `wait N`.
- Encoding: `R` = event number (8 bits); `arg` and `imm` are ignored (device; the 4G/5G form with a register and mask does not exist here).
- Effect: if `wait` event N is set, the program continues at once. Otherwise the sequencer waits.
- The wait ends with a timeout fault if the event is still not set after the timeout. The timeout is set by the start word: about 4.87 ms with the usual start word `0xfff5`, and no timeout at all without start-word bit 2 ([chapter 8, section 8.1.1](08-sequencer-control-registers.md#811-start-word-and-timeout-fmss_ctrl-device)).
- The timeout fault sets `CS_IRQ` bit 2. The program then continues at the fault vector if `C58` bit 2 is set, and at the next instruction otherwise.
- While waiting, `C08` = 4 and the timer `C14` counts down from `0xff`.
- Events are levels: an event stays set while its condition holds, and a wait that passes does not clear it ([chapter 7, section 7.1](07-events-and-waits.md#71-wait-events)). A `wait` for an event number above 15 never passes.
- Timing: a wait that passes costs the base instruction time.
- Original programs: used after every command, address, transfer, DMA, ECC operation and status poll.
- Evidence: [D-wait-1](14-evidence.md#1427-wait-run-1), [D-wait-4](14-evidence.md#14222-wait-run-4), [D-waitf-15](14-evidence.md#14261-waitf-run-15), [D-c58-2](14-evidence.md#14213-c58-run-2), [D-c58-3](14-evidence.md#14219-c58-run-3), [D-ctrlfx-27](14-evidence.md#14298-ctrlfx-run-27), [D-evcons-18](14-evidence.md#14282-evcons-run-18), [RE-7](14-evidence.md#1437-re-7-wait-event-5-shape).

### `08` `nop`

- No operation.
- Encoding: all fields ignored.
- Effect: no visible effect in any operand form. A pending flag of either NAND controller register set or of the ECC engine does not change this (device). An all-ones word (erased or `0xff`-filled memory) is `ff`, not `08`; see `ff`.
- Timing: the same as `movi`.
- Original programs: not used.
- Evidence: [D-unknown-1](14-evidence.md#1428-unknown-run-1), [D-unknown-2](14-evidence.md#14214-unknown-run-2), [D-unknown-3](14-evidence.md#14220-unknown-run-3), [D-optime-11](14-evidence.md#14244-optime-run-11), [D-opstate-24](14-evidence.md#14290-opstate-run-24), [D-opsweep-11](14-evidence.md#14243-opsweep-run-11).
- Note: two nop opcodes exist, `08` `nop` and `ff` `nop2`; in every test they behave the same way (no visible effect, same time). Why there are two is open.

### `09` `wait2`

- Wait for an interrupt line of the CPU: `wait2 N` continues once interrupt source N is raised in the raw status of the CPU's interrupt controllers (VIC0 line N for N = 0..31, VIC1 line N - 32 for N = 32..63); numbers 64..255 never pass (device, [D-w2vic-38](14-evidence.md#142171-w2vic-run-38); [chapter 7, section 7.2](07-events-and-waits.md#72-the-conditions-wait2-waits-for)).
- Syntax: `wait2 N`.
- Encoding: `R` = the number N of the `wait2` condition to wait for ([chapter 7, section 7.2](07-events-and-waits.md#72-the-conditions-wait2-waits-for)). `arg` and `imm` are ignored: ten operand forms, some matching the status byte and some not, made no difference.
- Effect (device): which conditions were true:
  - `wait2` condition 5 (VIC0 line 5, a source outside the FMSS engine) was true in every state and run, so the program continues at once.
  - `wait2` condition 20 (VIC0 line 20, the NAND controller's line) is true while a NAND controller flag is pending and enabled in `0x044` ([D-w2state-36](14-evidence.md#142151-w2state-run-36), [D-w2vic-38](14-evidence.md#142171-w2vic-run-38)).
  - `wait2` condition 43 (VIC1 line 11, the ECC engine's line) is true while an `ECC_DONE` flag is pending and enabled in `0x844` ([D-w2vic-38](14-evidence.md#142171-w2vic-run-38)).
  - `wait2` condition 33 (VIC1 line 1, a source outside the FMSS engine) was true in every state of runs 29 to 34 and in no state of runs 35 to 38.
  - For every other N (0..255), the sequencer waits about 4.87 ms in the waiting state (`C08` = 4) and then raises the timeout fault, as for a `wait` whose event is never set. The timer `C14` is handled as for `wait`.
- `wait2` condition 5 continued at once in every state tried: status flag pending, status flag acknowledged, and during a block erase while the chip reported busy (status `0x81`). In the same block-erase state, `wait` event 5 waited 180 us for ready.
- From a clean state with nothing pending, N = 0..31 with no chip, CE0 or CE1 selected: only condition 5 passed. In the same state `wait` passed only for `wait` event 11.
- Against chip activity ([`op09nand`](14-evidence.md#142116-op09nand-run-29)), N = 0..31 was tried during each of these states, and only condition 5 passed in all of them:
  - while a block erase runs, and right after it;
  - while a page program runs, and right after it;
  - during the busy time of a page read.
- N = 32..255 while idle and during an erase: only condition 33 passed (run 29). N = 0..63 while an ECC operation and a DMA run: only conditions 5 and 33 passed (run 34).
- Pending flags of the second register set made no difference.
- Run 35 ([`evsweep2`](14-evidence.md#142137-evsweep2-run-35)) tried N = 0..255 with a short timeout, in forward and in reverse order:
  - In each of these states only condition 5 passed: idle, erase busy, program busy, read busy, ECC operation in flight, DMA in flight, and after a command to the second register set.
  - With every enable on and a command flag pending, conditions 5 and 20 passed.
  - In the same run, `wait2` condition 33 also timed out in three other programs ([D-ctrlwords-35](14-evidence.md#142139-ctrlwords-run-35), [D-c08self-35](14-evidence.md#142140-c08self-run-35)).
- Original programs: not used.
- Evidence: [D-op09-24](14-evidence.md#14292-op09-run-24), [D-op09b-25](14-evidence.md#14293-op09b-run-25), [D-op09c-26](14-evidence.md#14294-op09c-run-26), [D-op09d-27](14-evidence.md#14299-op09d-run-27), [D-op09nand-29](14-evidence.md#142116-op09nand-run-29), [D-op09w33-30](14-evidence.md#142123-op09w33-run-30), [D-op0916-22](14-evidence.md#14288-op0916-run-22), [D-finale-34](14-evidence.md#142130-finale-run-34), [D-evsweep2-35](14-evidence.md#142137-evsweep2-run-35).

### `0a` `and`

- Bitwise AND.
- Encoding: `R` = destination register (and first source register in the register form), `arg` = source register, `imm` = immediate. `imm` = 0 selects the register form, in which the second operand comes from a register instead of from `imm`.
- Effect: bitwise AND. With a non-zero immediate, `rR = r(arg) AND imm` (source register in `arg`, destination in `R`); with an immediate of 0 the register form `rR = rR AND r(arg)` applies ([chapter 5, section 5.5](05-instruction-encoding.md#55-alu-operand-rule)). Register numbers of 8 and above behave as for `mov` ([chapter 4, section 4.3](04-sequencer-memory-map.md#43-register-file-r0r7-the-pc-and-register-numbers-above-7)).
- Timing: the base instruction time, as `movi` (217 ns from SDRAM, 138 ns from internal SRAM).
- Original programs: used (counts in [RE-4](14-evidence.md#1434-re-4-instruction-encoding-and-opcode-set-used-by-the-programs)).
- Evidence: [D-alu-1](14-evidence.md#1423-alu-run-1), [D-alu-2](14-evidence.md#14210-alu-run-2), [D-alu-3](14-evidence.md#14216-alu-run-3), [D-regnum-15](14-evidence.md#14259-regnum-run-15).

### `0b` `or`

- Bitwise OR.
- Encoding: `R` = destination register (and first source register in the register form), `arg` = source register, `imm` = immediate. `imm` = 0 selects the register form, in which the second operand comes from a register instead of from `imm`.
- Effect: bitwise OR. With a non-zero immediate, `rR = r(arg) OR imm` (source register in `arg`, destination in `R`); with an immediate of 0 the register form `rR = rR OR r(arg)` applies ([chapter 5, section 5.5](05-instruction-encoding.md#55-alu-operand-rule)). Register numbers of 8 and above behave as for `mov` ([chapter 4, section 4.3](04-sequencer-memory-map.md#43-register-file-r0r7-the-pc-and-register-numbers-above-7)).
- Timing: the base instruction time, as `movi` (217 ns from SDRAM, 138 ns from internal SRAM).
- Original programs: used (counts in [RE-4](14-evidence.md#1434-re-4-instruction-encoding-and-opcode-set-used-by-the-programs)).
- Evidence: [D-alu-1](14-evidence.md#1423-alu-run-1), [D-alu-2](14-evidence.md#14210-alu-run-2), [D-alu-3](14-evidence.md#14216-alu-run-3), [D-regnum-15](14-evidence.md#14259-regnum-run-15).

### `0c` `add`

- Addition.
- Encoding: `R` = destination register (and first source register in the register form), `arg` = source register, `imm` = immediate. `imm` = 0 selects the register form, in which the second operand comes from a register instead of from `imm`.
- Effect: addition, wrapping at 32 bits (device). With a non-zero immediate, `rR = r(arg) + imm` (source register in `arg`, destination in `R`); with an immediate of 0 the register form `rR = rR + r(arg)` applies ([chapter 5, section 5.5](05-instruction-encoding.md#55-alu-operand-rule)). Register numbers of 8 and above behave as for `mov` ([chapter 4, section 4.3](04-sequencer-memory-map.md#43-register-file-r0r7-the-pc-and-register-numbers-above-7)).
- Timing: the base instruction time, as `movi` (217 ns from SDRAM, 138 ns from internal SRAM).
- Original programs: used (counts in [RE-4](14-evidence.md#1434-re-4-instruction-encoding-and-opcode-set-used-by-the-programs)).
- Evidence: [D-alu-1](14-evidence.md#1423-alu-run-1), [D-alu-2](14-evidence.md#14210-alu-run-2), [D-alu-3](14-evidence.md#14216-alu-run-3), [D-regnum-15](14-evidence.md#14259-regnum-run-15), [D-opcost-15](14-evidence.md#14262-opcost-run-15).

### `0d` `sub`

- Subtraction.
- Encoding: `R` = destination register (and first source register in the register form), `arg` = source register, `imm` = immediate. `imm` = 0 selects the register form, in which the second operand comes from a register instead of from `imm`.
- Effect: subtraction, wrapping at 32 bits (device). With a non-zero immediate, `rR = r(arg) - imm` (source register in `arg`, destination in `R`); with an immediate of 0 the register form `rR = rR - r(arg)` applies ([chapter 5, section 5.5](05-instruction-encoding.md#55-alu-operand-rule)). Register numbers of 8 and above behave as for `mov` ([chapter 4, section 4.3](04-sequencer-memory-map.md#43-register-file-r0r7-the-pc-and-register-numbers-above-7)).
- Timing: the base instruction time, as `movi` (217 ns from SDRAM, 138 ns from internal SRAM).
- Original programs: used (counts in [RE-4](14-evidence.md#1434-re-4-instruction-encoding-and-opcode-set-used-by-the-programs)).
- Evidence: [D-alu-1](14-evidence.md#1423-alu-run-1), [D-alu-2](14-evidence.md#14210-alu-run-2), [D-alu-3](14-evidence.md#14216-alu-run-3), [D-regnum-15](14-evidence.md#14259-regnum-run-15).

### `0e` `bnz`

- Branch if non-zero.
- Syntax: `bnz rR, target`.
- Encoding: `R` = tested register, `imm` = target (byte offset from the program start); `arg` is ignored (device).
- Effect: branch if rR is non-zero. All 32 bits are tested, so `0x80000000` counts as non-zero.
- The target is used as given. An offset that is not a multiple of 8 starts decoding in the middle of an instruction: a target 4 bytes past an instruction boundary lands mid-instruction, and a target 8 bytes before the intended label runs the instruction before the label.
- With any `R` from 8 to 255 (8 included), the branch was never taken, whether r0 was zero or non-zero (device, [D-regsweep-35](14-evidence.md#142149-regsweep-run-35)).
- The target is not cut to 12 bits: with bit 12, 16 or 24 set it points beyond the program, and the branch does not reach the label the low bits name. Target bit 31 is ignored ([chapter 4, section 4.1](04-sequencer-memory-map.md#41-window-offsets-and-target-widths)).
- Timing: not taken, as `movi`.
- Original programs: used for every loop and condition; they have no unconditional branch.
- Evidence: [D-branch-15](14-evidence.md#14260-branch-run-15), [D-regnum-15](14-evidence.md#14259-regnum-run-15), [D-regsweep-35](14-evidence.md#142149-regsweep-run-35), [D-fwidth-27](14-evidence.md#142100-fwidth-run-27).

### `0f` `jr`

- Jump through a register.
- Syntax: `jr rA`.
- Encoding: `arg` = register holding the target (byte offset from the program start); `R` and `imm` are ignored.
- Effect (device): the program counter is set to the value of r(arg), without rounding. No return address is saved.
- `arg` = 9..255 jumped through r0, and so did `arg` = `0xd70` (a value above 255 in the 16-bit field). `arg` = 8 jumps to the `jr` instruction itself, so the program never ends ([D-regsweep-35](14-evidence.md#142149-regsweep-run-35)).
- If a `movi rX, #return` runs before the jump, a `jr` through rX continues at `return`. The opcode therefore provides computed jumps and subroutine returns.
- If the register value has bit 12, 16, 24 or 31 set, no jump happens.
- Original programs: not used.
- Evidence: [D-op0f-14](14-evidence.md#14251-op0f-run-14), [D-indirect-1](14-evidence.md#1425-indirect-run-1), [D-indirect-2](14-evidence.md#14211-indirect-run-2), [D-indirect-3](14-evidence.md#14217-indirect-run-3), [D-fwidth-27](14-evidence.md#142100-fwidth-run-27), [D-regnum-15](14-evidence.md#14259-regnum-run-15).

### `10` `sig`

- Signal the CPU (`CS_IRQ` bit 1).
- Encoding: all fields ignored.
- Effect (device): sets `CS_IRQ` bit 1 and continues with the next instruction, also when a flag is pending. The CPU can read the flag while the program runs. The CPU interrupt line was not raised.
- Original programs: not used.
- Evidence: [D-op10-6](14-evidence.md#14232-op10-run-6), [D-opstate-24](14-evidence.md#14290-opstate-run-24), [D-irqline-15](14-evidence.md#14267-irqline-run-15), [D-irqline-16](14-evidence.md#14270-irqline-run-16), [D-irqline-17](14-evidence.md#14276-irqline-run-17).

### `11` `sti`

- Store a word to system memory.
- Syntax: `sti [rA], rR`.
- Encoding: `arg` = register holding a bus address, `R` = source; `imm` is ignored.
- Effect: a 32-bit little-endian store of rR to system memory. An unaligned address is rounded down to the word boundary (device).
- Timing: about 186 ns more than `movi` from SDRAM, 120 ns more from internal SRAM (device).
- Original programs: used to store the result and the metadata of each request.
- Evidence: [D-memory-1](14-evidence.md#1424-memory-run-1), [D-opcost-15](14-evidence.md#14262-opcost-run-15), [D-fields0-28](14-evidence.md#142106-fields0-run-28).

### `12` `pause`

- Hand control to the CPU.
- Encoding: all fields ignored; pending flags make no difference.
- Effect (device): `pause` stops the program at this instruction and raises `CS_IRQ` bit 3. Its fields are ignored. It acts as a breakpoint: the CPU continues the program by writing the PC (`0xc38`) and a start word.
- While stopped: `CS_IRQ` bit 3 is set, `C08` = `0xa`, the instruction is latched in `0xc40` and `0xc44`, and `FMSS_CTRL` bit 18 stays set. There is no timeout.
- Continuing: the CPU writes the PC (`0xc38`) with the byte offset at which to continue (normally the offset of the `pause` itself + 8) and then writes a start word (`0xfff5`). The program runs on from that offset. In the measurement the pause lasted 2 ms, and the `end` ran 25 us after the CPU's writes.
- The CPU does not need to acknowledge `CS_IRQ` bit 3 to continue. If it does not, bit 3 stays set.
- If the CPU writes a start word without writing the PC, the sequencer continues at the `pause` instruction itself, which pauses again.
- The CPU can write the PC only while the sequencer is stopped at a `pause`. The write is ignored when the sequencer is idle.
- Resetting the sequencer also ends the pause. Eighteen other kinds of CPU write were tried, and none of them released it.
- The 4G/5G driver uses the same sequence for its "pause" (`FMSS_PC += 8; CS_IRQ = 8; FMSS_CTRL |= 1`).
- Original programs: the 3G original programs do not use it.
- Evidence: [D-op12-5](14-evidence.md#14224-op12-run-5), [D-op12wait-11](14-evidence.md#14240-op12wait-run-11), [D-op12ack-6](14-evidence.md#14231-op12ack-run-6), [D-op12rel-15](14-evidence.md#14263-op12rel-run-15), [D-op12cont-27](14-evidence.md#14297-op12cont-run-27), [D-pause-15](14-evidence.md#14264-pause-run-15), [D-ctrlhold-14](14-evidence.md#14252-ctrlhold-run-14), [D-c08-15](14-evidence.md#14256-c08-run-15).

### `13` `shl`

- Logical shift left.
- Encoding: `R` = destination register (and first source register in the register form), `arg` = source register, `imm` = immediate. `imm` = 0 selects the register form, in which the second operand comes from a register instead of from `imm`.
- Effect: logical shift left. A shift count of 32 or more gives 0, whether the count is the immediate or comes from a register (device). With a non-zero immediate, `rR = r(arg) << imm` (source register in `arg`, destination in `R`); with an immediate of 0 the register form `rR = rR << r(arg)` applies ([chapter 5, section 5.5](05-instruction-encoding.md#55-alu-operand-rule)). Register numbers of 8 and above behave as for `mov` ([chapter 4, section 4.3](04-sequencer-memory-map.md#43-register-file-r0r7-the-pc-and-register-numbers-above-7)).
- Timing: the base instruction time, as `movi` (217 ns from SDRAM, 138 ns from internal SRAM).
- Original programs: used (counts in [RE-4](14-evidence.md#1434-re-4-instruction-encoding-and-opcode-set-used-by-the-programs)).
- Evidence: [D-alu-1](14-evidence.md#1423-alu-run-1), [D-alu-2](14-evidence.md#14210-alu-run-2), [D-alu-3](14-evidence.md#14216-alu-run-3), [D-holdout-9](14-evidence.md#14238-holdout-run-9), [D-holdout-10](14-evidence.md#14239-holdout-run-10).

### `14` `shr`

- Logical shift right.
- Encoding: `R` = destination register (and first source register in the register form), `arg` = source register, `imm` = immediate. `imm` = 0 selects the register form, in which the second operand comes from a register instead of from `imm`.
- Effect: logical shift right (not arithmetic, so the top bits are filled with 0). A shift count of 32 or more gives 0, whether the count is the immediate or comes from a register (device). With a non-zero immediate, `rR = r(arg) >> imm` (source register in `arg`, destination in `R`); with an immediate of 0 the register form `rR = rR >> r(arg)` applies ([chapter 5, section 5.5](05-instruction-encoding.md#55-alu-operand-rule)). Register numbers of 8 and above behave as for `mov` ([chapter 4, section 4.3](04-sequencer-memory-map.md#43-register-file-r0r7-the-pc-and-register-numbers-above-7)).
- Timing: the base instruction time, as `movi` (217 ns from SDRAM, 138 ns from internal SRAM).
- Original programs: used (counts in [RE-4](14-evidence.md#1434-re-4-instruction-encoding-and-opcode-set-used-by-the-programs)).
- Evidence: [D-alu-1](14-evidence.md#1423-alu-run-1), [D-alu-2](14-evidence.md#14210-alu-run-2), [D-alu-3](14-evidence.md#14216-alu-run-3).

### `15` `tmwait`

- Wait until the timeout timer reaches 0.
- Encoding: all fields ignored; pending flags make no difference.
- Effect (device): if the timer `C14` is non-zero, `tmwait` reloads the timer from bits 15:8 of the start word (`0xff` with `0xfff5`). It then waits until the timer reaches 0 (about 4.85 ms) and leaves it at 0. If the timer is 0, `tmwait` reloads it and continues at once. Two `tmwait` in one program made the whole program run 4841, 18 and 3250 us in three runs ([D-ctrlwords-35](14-evidence.md#142139-ctrlwords-run-35); not explained). Four `tmwait` in one program alternated regularly in five runs: `C14` read 0, `0xff`, 0, `0xff` after them, and each run took about 9.7 ms ([D-tmwait2-36](14-evidence.md#142157-tmwait2-run-36)).
- Original programs: not used.
- Evidence: [D-op15-4](14-evidence.md#14223-op15-run-4), [D-c14-5](14-evidence.md#14227-c14-run-5), [D-opstate-24](14-evidence.md#14290-opstate-run-24).

### `16` `evjmp`

- Event jump: wait for any event flag, then jump to the fault vector.
- Encoding: no field is used.
- Effect (device): `evjmp` waits until any event flag of either NAND controller register set or of the ECC engine is set. The events are 0..9: command sent, address sent, DMA done, transfer done, ECC done, status latched, and the second register set's flags. It then continues at the fault vector `C4C`.
- No fault is raised: after the `end` at the vector, `CS_IRQ` holds only bit 0.
- If no flag is pending, `evjmp` waits about 4.87 ms in the waiting state and raises the timeout fault like an unmet `wait`. It then continues as for any fault: at the fault vector if `C58` bit 2 is set, at the next instruction if it is clear.
- Each of the following sent the program to the vector within about 20 us: a command flag, a DMA flag, an `ECC_DONE` bit 0, a status flag, `ECC_DONE` bits left by a page read, and the command and DMA flags of the second register set (events 6 and 8). With every flag acknowledged, `evjmp` timed out.
- `evjmp` also reacts to a flag that is set during the wait: a command written just before took 19 us, and a DMA started just before took 22 us.
- The target is whatever `C4C` holds. `C58` makes no difference to the target. The flag is still set at the target, and `C3C` is not written.
- In the opcode sweep it always timed out because no flag was pending there.
- Original programs: not used.
- Evidence: [D-op16-23](14-evidence.md#14289-op16-run-23), [D-op16b-24](14-evidence.md#14291-op16b-run-24), [D-opstate-24](14-evidence.md#14290-opstate-run-24), [D-op0916-22](14-evidence.md#14288-op0916-run-22), [D-opsweep-11](14-evidence.md#14243-opsweep-run-11).

### `17` `bz`

- Branch if zero.
- Syntax: `bz rR, target`.
- Encoding: `R` = tested register, `imm` = target (byte offset from the program start); `arg` is ignored (device).
- Effect: branch if rR is zero; all 32 bits are tested. Target behaviour as for `bnz` (`0e`); with any `R` from 8 to 255 it never branched, also with r0 = 0 (device, [D-regsweep-35](14-evidence.md#142149-regsweep-run-35)).
- Timing: not taken, as `movi`.
- Original programs: used (51 uses, [RE-4](14-evidence.md#1434-re-4-instruction-encoding-and-opcode-set-used-by-the-programs)).
- Evidence: [D-branch-15](14-evidence.md#14260-branch-run-15), [D-regnum-15](14-evidence.md#14259-regnum-run-15).

### `18` `pld`

- Load a window word at a computed offset.
- Syntax: `pld rR, [rA]`.
- Encoding: `arg` = register holding a window byte offset, `R` = destination; `imm` is ignored.
- Effect (device): rR = the word of the register window at byte offset rA, where rA is the value of the register named by `arg`.
- The offset wraps at 12 bits (`0x1d64` reads `0xd64`). Every word of the window can be read this way, including the registers r0..r7: a `pld` of offset `0xc18` returns r0.
- If `arg` is a register number from 9 to 255, r0 supplies the offset ([D-regsweep-35](14-evidence.md#142149-regsweep-run-35)).
- This opcode is present on the 3G only.
- Timing: as `wr`.
- Original programs: used twice, in READ, on parameter RAM `0xd60..0xd7c` (a queue of chip masks) ([RE-6](14-evidence.md#1436-re-6-opcodes-18-and-19)).
- Evidence: [D-indirect-1](14-evidence.md#1425-indirect-run-1), [D-indirect-2](14-evidence.md#14211-indirect-run-2), [D-indirect-3](14-evidence.md#14217-indirect-run-3), [D-fmcmap-11](14-evidence.md#14246-fmcmap-run-11), [D-fields0-28](14-evidence.md#142106-fields0-run-28), [D-opcost-15](14-evidence.md#14262-opcost-run-15), [RE-6](14-evidence.md#1436-re-6-opcodes-18-and-19).

### `19` `pst`

- Store a window word at a computed offset.
- Syntax: `pst [rA], rR`.
- Encoding: `arg` = register holding a window byte offset, `R` = source; `imm` is ignored.
- Effect: stores rR into the word of the register window at byte offset rA, where rA is the value of the register named by `arg` (device). The offset is as for `pld`: it wraps at 12 bits, and every word of the window can be reached.
- This opcode is present on the 3G only.
- Timing: as `wr`.
- Original programs: used once, in READ, on parameter RAM `0xd60..0xd7c` ([RE-6](14-evidence.md#1436-re-6-opcodes-18-and-19)).
- Evidence: [D-indirect-1](14-evidence.md#1425-indirect-run-1), [D-indirect-2](14-evidence.md#14211-indirect-run-2), [D-indirect-3](14-evidence.md#14217-indirect-run-3), [D-fmcmap-11](14-evidence.md#14246-fmcmap-run-11), [D-fields0-28](14-evidence.md#142106-fields0-run-28), [D-opcost-15](14-evidence.md#14262-opcost-run-15), [RE-6](14-evidence.md#1436-re-6-opcodes-18-and-19).

### `ff` `nop2`

- No operation.
- Encoding: all fields ignored.
- Effect: as `08`: no visible effect in any operand form, the same time as `movi`, no reaction to a pending flag (device). An all-ones word (erased or `0xff`-filled memory) therefore runs as a no-operation, not as an end.
- Original programs: not used.
- Evidence: [D-unknown-1](14-evidence.md#1428-unknown-run-1), [D-unknown-2](14-evidence.md#14214-unknown-run-2), [D-unknown-3](14-evidence.md#14220-unknown-run-3), [D-optime-11](14-evidence.md#14244-optime-run-11), [D-opstate-24](14-evidence.md#14290-opstate-run-24), [D-opsweep-11](14-evidence.md#14243-opsweep-run-11).

### `1a`..`fe` and other values without a meaning

- Illegal opcodes.
- Effect (device): all 256 opcode values were run, each in three operand forms. The values `1a..fe` fault in every form tried: the instruction stalls for the timeout (about 4.87 ms in the waiting state, `C08` = 4) and then raises the timeout fault. None of these values changed a register, RAM, parameter RAM or any window word.
- The opcode byte is decoded in all 8 bits, so no opcode has a second encoding. `20`, `40` and `80` are `00` (`end`) with one extra high bit set (`0x20` = `0010 0000`, `0x40` = `0100 0000`, `0x80` = `1000 0000`). If the hardware decoded only the low bits of the opcode byte, these three would act as further copies of `end`. They do not: each behaves like every other value in this range and like each other, stalling for the timeout (about 4.87 ms) and then faulting.
- The same holds for `21`, which is `01` (`wr`) with bit 5 set: it does not act as `wr`. The event number of `wait` is decoded in all 8 bits in the same way: `wait` events 16, 27, 139 and 255 do not act as `wait` events 0, 11 or 15.
- A fault that comes after the full timeout of about 4.87 ms in the waiting state cannot by itself distinguish an illegal opcode from a wait whose condition was never met. "Illegal" here means that no condition was found that makes the value do anything else.
- The valid opcodes are the 19 used by the original programs plus `08`, `09`, `0f`, `10`, `12`, `15`, `16` and `ff`.
- Original programs: not used.
- Evidence: [D-opsweep-11](14-evidence.md#14243-opsweep-run-11), [D-unknown-1](14-evidence.md#1428-unknown-run-1), [D-unknown-2](14-evidence.md#14214-unknown-run-2), [D-unknown-3](14-evidence.md#14220-unknown-run-3), [D-op0916-22](14-evidence.md#14288-op0916-run-22), [D-c58ill-5](14-evidence.md#14225-c58ill-run-5).

References: [chapter 14](14-evidence.md) (outcomes), [chapter 13](13-experiments.md) (what was done and why), [chapter 15](15-evidence-interpretation.md) (what the outcomes mean).

Related chapters: [chapter 5](05-instruction-encoding.md), [chapter 7](07-events-and-waits.md), [chapter 8](08-sequencer-control-registers.md).
