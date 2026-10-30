# Chapter 14 — Evidence: outcomes

Summary: The outcomes on which this documentation rests, without interpretation: the printed result of every experiment run on the device, and the observations made in the original firmware by reverse engineering.

## 14.1 Scope

This chapter holds outcomes only, in two groups: the printed outcome of every experiment run on the device ([chapter 14, section 14.2](#142-device-outcomes)), and the observations made in the original firmware by reverse engineering ([chapter 14, section 14.3](#143-reverse-engineering-outcomes)). It contains no interpretation; readings are in [chapter 15](15-evidence-interpretation.md). What was done in each experiment, and why, is in [chapter 13](13-experiments.md). Device runs are numbered 1 to 38 in the order they were made. Entry IDs are `D-<experiment>-<device run>` for device outcomes and `RE-<n>` for reverse-engineering observations. The printed text of each outcome is reproduced word for word, except that names of the test setup were left out.

## 14.2 Device outcomes

171 entries: 142 experiments in 38 device runs. Entries are in the order the runs were made.

### 14.2.1 regmap (run 1)

ID: `D-regmap-1`. Device run 1. Experiment: [regmap](13-experiments.md#1331-regmap).

```
END: r0..r7 at 0xc18..0xc34 match; SEQ_RESULT 11116666 == r6
```

### 14.2.2 clock (run 1)

ID: `D-clock-1`. Device run 1. Experiment: [clock](13-experiments.md#1332-clock).

```
loop of 2 instructions: 873 us for 2000, 8657 us for 20000 iterations -> 216.2 ns per instruction (~5 MHz if 1 per clock)
```

### 14.2.3 alu (run 1)

ID: `D-alu-1`. Device run 1. Experiment: [alu](13-experiments.md#1333-alu).

```
ALU:
  shr 0x80000000 by 1                -> 40000000  logical
  shl 1 by 32 (imm)                  -> 00000000  shift >= 32 gives 0
  shl 1 by 33 (imm)                  -> 00000000  gives 0
  shr 0x80000000 by 32               -> 00000000  gives 0
  add wraps                          -> 00000001  32-bit wrap
  sub below 0                        -> ffffffff  32-bit wrap
  add imm 0 is the 2-operand form    -> 0000000c  r2 = r2 + r1 (2-operand)
  shl by a register >= 32            -> 00000000  gives 0
  ld mask                            -> 00000000  UNEXPECTED
```

### 14.2.4 memory (run 1)

ID: `D-memory-1`. Device run 1. Experiment: [memory](13-experiments.md#1334-memory).

```
memory: END; ldi [base] = 03020100 (expect 03020100 little endian); ldi [base+2] = 03020100 (03020100 if the address is word-aligned down, 05040302 if unaligned access works); sti [base+0x40] -> bytes d4c3b2a1 (expect d4c3b2a1); sti [base+0x81] -> bytes 0x80..0x87 d4c3b2a184858687 (aligned down: d4c3b2a1 at 0x80)
```

### 14.2.5 indirect (run 1)

ID: `D-indirect-1`. Device run 1. Experiment: [indirect](13-experiments.md#1335-indirect).

```
indirect access:
  pld 0xc18 (r0 through the window)    -> r2 = 5a5a0000
  pld 0xd94 (parameter RAM)            -> r2 = 00000000
  pld 0x1d94 (above the window)        -> r2 = 00000000
  pst 0xd98 then ld                    -> r2 = 00000000
```

### 14.2.6 control (run 1)

ID: `D-control-1`. Device run 1. Experiment: [control](13-experiments.md#1336-control).

```
control window after a run: c00:0000fff4 c04:0801af40 c08:00000000 c0c:00000001 c10:00000001 c14:0000007f c18:00000000 c1c:00000000 c20:00000000 c24:0badcafe c28:00000000 c2c:00000000 c30:00000000 c34:00000000 c38:00000000 c3c:00000000 c40:00000000 c44:00000000 c48:00000000 c4c:00000030 c50:ffff1800 c54:00000000 c58:00000000 c5c:00000000
  CPU wrote r3 (0xc24) = 0badcafe before start; the program stored r3 = 00000000 (registers reset at start)
```

### 14.2.7 wait (run 1)

ID: `D-wait-1`. Device run 1. Experiment: [wait](13-experiments.md#1337-wait).

```
WAIT:
  wait 0  -> FAULT   fault vector after   4881 us, CS_IRQ 00000005, C50 ffff1800
  wait 1  -> FAULT   fault vector after   4881 us, CS_IRQ 00000005, C50 ffff1800
  wait 2  -> FAULT   fault vector after   4881 us, CS_IRQ 00000005, C50 ffff1800
  wait 3  -> FAULT   fault vector after   4869 us, CS_IRQ 00000005, C50 ffff1800
  wait 4  -> FAULT   fault vector after   4870 us, CS_IRQ 00000005, C50 ffff1800
  wait 5  -> FAULT   fault vector after   4870 us, CS_IRQ 00000005, C50 ffff1800
  wait 6  -> FAULT   fault vector after   4881 us, CS_IRQ 00000005, C50 ffff1800
  wait 7  -> FAULT   fault vector after   4883 us, CS_IRQ 00000005, C50 ffff1800
  wait 8  -> FAULT   fault vector after   4880 us, CS_IRQ 00000005, C50 ffff1800
  wait 9  -> FAULT   fault vector after   4870 us, CS_IRQ 00000005, C50 ffff1800
  wait 10 -> FAULT   fault vector after   4879 us, CS_IRQ 00000005, C50 ffff1800
  wait 11 -> END     passed       after     15 us, CS_IRQ 00000001, C50 ffff1800
  wait 12 -> FAULT   fault vector after   4863 us, CS_IRQ 00000005, C50 ffff1800
  wait 13 -> FAULT   fault vector after   4880 us, CS_IRQ 00000005, C50 ffff1800
  wait 14 -> FAULT   fault vector after   4881 us, CS_IRQ 00000005, C50 ffff1800
  wait 15 -> FAULT   fault vector after   4872 us, CS_IRQ 00000005, C50 ffff1800
  wait 1 with FMSS_C58 = 0  -> FAULT   after    4882 us, r6 600d600d
  wait 1 with FMSS_C58 = 1  -> FAULT   after    4881 us, r6 600d600d
  wait 1 with FMSS_C58 = 2  -> END     after      15 us, r6 fa17fa17
  wait 1 with FMSS_C58 = 8  -> END     after      15 us, r6 fa17fa17
  wait 1 with FMSS_C58 = 16 -> END     after      15 us, r6 fa17fa17
```

### 14.2.8 unknown (run 1)

ID: `D-unknown-1`. Device run 1. Experiment: [unknown](13-experiments.md#1338-unknown).

```
unknown opcodes (.insn op, r3, arg 1, imm; r1 = 0xd84, r3 = 0x30000003):
  op 08 imm 0      -> END     continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:000000c0->0000013c
  op 08 imm d88    -> END     continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:000001b8->00000034 0x050:00001800->00002000
  op 09 imm 0      -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:000000b0->0000012c
  op 09 imm d88    -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:000001a8->00000024 0x050:00002000->00002800
  op 0f imm 0      -> END     continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:000000a0->0000011c
  op 0f imm d88    -> END     continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000198->00000014 0x050:00002800->00003000
  op 10 imm 0      -> END     continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000090->0000010c
  op 10 imm d88    -> END     continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000188->00000004 0x050:00003000->00003800
  op 12 imm 0      -> TIMEOUT r6=60000006  r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000080->000000fc
  op 12 imm d88    -> TIMEOUT r6=60000006  r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000178->000001f4
  op 15 imm 0      -> END     continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000070->000000ec
  op 15 imm d88    -> END     continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000168->000001e4
  op 16 imm 0      -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000060->000000dc
  op 16 imm d88    -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000158->000001d4
  op 1a imm 0      -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000050->000000cc
  op 1a imm d88    -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000148->000001c4
  op 1b imm 0      -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000040->000000bc
  op 1b imm d88    -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000138->000001b4
  op 1c imm 0      -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000030->000000ac
  op 1c imm d88    -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000128->000001a4
  op 1d imm 0      -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000020->0000009c
  op 1d imm d88    -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000118->00000194
  op 1e imm 0      -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000010->0000008c
  op 1e imm d88    -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000108->00000184
  op 1f imm 0      -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:00000000->0000007c
  op 1f imm d88    -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:000000f8->00000174
  op 20 imm 0      -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:000001f0->0000006c 0x050:00003800->00000000
  op 20 imm d88    -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:000000e8->00000164
  op 40 imm 0      -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:000001e0->0000005c 0x050:00000000->00000800
  op 40 imm d88    -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:000000d8->00000154
  op 80 imm 0      -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:000001d0->0000004c 0x050:00000800->00001000
  op 80 imm d88    -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:000000c8->00000144
  op ff imm 0      -> END     continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:000001c0->0000003c 0x050:00001000->00001800
  op ff imm d88    -> END     continued    r0=00000000 r1=00000000 r2=00000000 r3=00000000; 0x028:000000b8->00000134
```

### 14.2.9 pram (run 2)

ID: `D-pram-2`. Device run 2. Experiment: [pram](13-experiments.md#1339-pram).

```
CPU write then CPU read keeps: 0xd00..0xd7c
  sequencer ld of 0xd04, 0xd20, 0xd40, 0xd5c, 0xd60, 0xd7c, 0xd80, 0xdfc -> 00000000 00000000 00000000 a5000d5c a5000d60 a5000d7c 00000000 00000000
  sequencer wr then CPU read keeps (0xd80..0xdfc): none
```

### 14.2.10 alu (run 2)

ID: `D-alu-2`. Device run 2. Experiment: [alu](13-experiments.md#1333-alu).

```
ALU:
  shr 0x80000000 by 1                -> 00000000  UNEXPECTED
  shl 1 by 32 (imm)                  -> 00000000  shift >= 32 gives 0
  shl 1 by 33 (imm)                  -> 00000000  gives 0
  shr 0x80000000 by 32               -> 00000000  gives 0
  add wraps                          -> 00000000  UNEXPECTED
  sub below 0                        -> 00000000  UNEXPECTED
  add imm 0 is the 2-operand form    -> 00000000  UNEXPECTED
  shl by a register >= 32            -> 00000000  gives 0
  ld mask                            -> 00000000  UNEXPECTED
```

### 14.2.11 indirect (run 2)

ID: `D-indirect-2`. Device run 2. Experiment: [indirect](13-experiments.md#1335-indirect).

```
indirect access:
  pld 0xc18 (r0 through the window)    -> r2 = 00000000
  pld 0xd64 (parameter RAM)            -> r2 = 00000000
  pld 0x1d64 (above the window)        -> r2 = 00000000
  pst 0xd68 then ld                    -> r2 = 00000000
```

### 14.2.12 control (run 2)

ID: `D-control-2`. Device run 2. Experiment: [control](13-experiments.md#1336-control).

```
control window after a run: c00:0000fff4 c04:0801af40 c08:00000000 c0c:00000001 c10:00000001 c14:000000ff c18:00000000 c1c:00000000 c20:00000000 c24:0badcafe c28:00000000 c2c:00000000 c30:00000000 c34:00000000 c38:00000000 c3c:00000000 c40:00000000 c44:00000000 c48:00000000 c4c:00000030 c50:ffff1800 c54:00000000 c58:00000001 c5c:00000001
  CPU wrote r3 (0xc24) = 0badcafe before start; the program stored r3 = 00000000 (not kept (0))
```

### 14.2.13 c58 (run 2)

ID: `D-c58-2`. Device run 2. Experiment: [c58](13-experiments.md#13310-c58).

```
C58 sweep (wait 1, nothing pending):
  C58 = 0x0     -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x1     -> END     vector    after      16 us, CS_IRQ 00000001
  C58 = 0x2     -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x3     -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x4     -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x5     -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x6     -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x7     -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x8     -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x9     -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0xa     -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0xb     -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0xc     -> END     vector    after      16 us, CS_IRQ 00000001
  C58 = 0xd     -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0xe     -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0xf     -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x10    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x11    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x12    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x13    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x14    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x15    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x16    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x17    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x18    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x19    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x1a    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x1b    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x1c    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x1d    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x1e    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x1f    -> END     vector    after      16 us, CS_IRQ 00000001
  C58 = 0x40    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x80    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0xff    -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0x100   -> END     vector    after      15 us, CS_IRQ 00000001
  C58 = 0xffff  -> END     vector    after      15 us, CS_IRQ 00000001
```

### 14.2.14 unknown (run 2)

ID: `D-unknown-2`. Device run 2. Experiment: [unknown](13-experiments.md#1338-unknown).

```
unknown opcodes (.insn op, r3, arg 1, imm; r1 = 0xd70, r3 = 0x30000003; C58 = 0, so a fault flags and continues; op 0b is the control: or r3, r1 [, #imm]):
  op 0b imm 0      -> END     continued    r0=00000000 r1=00000000 r2=00000000; 0x028:00000010->0000008c
  op 0b imm d74    -> END     continued    r0=00000000 r1=00000000 r2=00000000 r3=00000d74; 0x028:00000108->00000184
  op 08 imm 0      -> END     continued    r0=00000000 r1=00000000 r2=00000000; 0x028:00000000->0000007c
  op 08 imm d74    -> END     continued    r0=00000000 r1=00000000 r2=00000000; 0x028:000000f8->00000174
  op 09 imm 0      -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000; 0x028:000001f0->0000006c 0x050:00003000->00003800
  op 09 imm d74    -> FAULT   continued    r0=00000000 r1=00000000 r2=00000000; 0x028:000000e8->00000164
  op 0f imm 0      -> END     r6=60000006  no register changed; 0x028:000001e0->0000005c 0x050:00003800->00000000
  op 0f imm d74    -> END     r6=60000006  no register changed; 0x028:000000d8->00000154
  op 10 imm 0      -> END     continued    no register changed; 0x028:000001d0->0000004c 0x050:00000000->00000800
  op 10 imm d74    -> END     continued    no register changed; 0x028:000000c8->00000144
  op 12 imm 0      -> TIMEOUT r6=60000006  no register changed; 0x028:000001c0->0000003c 0x050:00000800->00001000
  op 12 imm d74    -> TIMEOUT r6=60000006  no register changed; 0x028:000000b8->00000134
  op 15 imm 0      -> END     continued    no register changed; 0x028:000001b0->0000002c 0x050:00001000->00001800
  op 15 imm d74    -> END     continued    no register changed; 0x028:000000a8->00000124
  op 16 imm 0      -> FAULT   continued    no register changed; 0x028:000001a0->0000001c 0x050:00001800->00002000
  op 16 imm d74    -> FAULT   continued    no register changed; 0x028:00000098->00000114
  op 1a imm 0      -> FAULT   continued    no register changed; 0x028:00000190->0000000c 0x050:00002000->00002800
  op 1a imm d74    -> FAULT   continued    no register changed; 0x028:00000088->00000104
  op 1b imm 0      -> FAULT   continued    no register changed; 0x028:00000180->000001fc
  op 1b imm d74    -> FAULT   continued    no register changed; 0x028:00000078->000000f4
  op 1c imm 0      -> FAULT   continued    no register changed; 0x028:00000170->000001ec
  op 1c imm d74    -> FAULT   continued    no register changed; 0x028:00000068->000000e4
  op 1d imm 0      -> FAULT   continued    no register changed; 0x028:00000160->000001dc
  op 1d imm d74    -> FAULT   continued    no register changed; 0x028:00000058->000000d4
  op 1e imm 0      -> FAULT   continued    no register changed; 0x028:00000150->000001cc
  op 1e imm d74    -> FAULT   continued    no register changed; 0x028:00000048->000000c4
  op 1f imm 0      -> FAULT   continued    no register changed; 0x028:00000140->000001bc
  op 1f imm d74    -> FAULT   continued    no register changed; 0x028:00000038->000000b4
  op 20 imm 0      -> FAULT   continued    no register changed; 0x028:00000130->000001ac
  op 20 imm d74    -> FAULT   continued    no register changed; 0x028:00000028->000000a4
  op 40 imm 0      -> FAULT   continued    no register changed; 0x028:00000120->0000019c
  op 40 imm d74    -> FAULT   continued    no register changed; 0x028:00000018->00000094
  op 80 imm 0      -> FAULT   continued    no register changed; 0x028:00000110->0000018c
  op 80 imm d74    -> FAULT   continued    no register changed; 0x028:00000008->00000084
  op ff imm 0      -> END     continued    no register changed; 0x028:00000100->0000017c
  op ff imm d74    -> END     continued    no register changed; 0x028:000001f8->00000074 0x050:00002800->00003000
```

### 14.2.15 pram (run 3)

ID: `D-pram-3`. Device run 3. Experiment: [pram](13-experiments.md#1339-pram).

```
CPU write then CPU read keeps: 0xd00..0xd7c
  sequencer ld of 0xd04, 0xd20, 0xd40, 0xd5c, 0xd60, 0xd7c, 0xd80, 0xdfc -> a5000d04 a5000d20 a5000d40 a5000d5c a5000d60 a5000d7c 00000000 00000000
  sequencer wr then CPU read keeps (0xd80..0xdfc): none
```

### 14.2.16 alu (run 3)

ID: `D-alu-3`. Device run 3. Experiment: [alu](13-experiments.md#1333-alu).

```
ALU:
  shr 0x80000000 by 1                -> 40000000  logical
  shl 1 by 32 (imm)                  -> 00000000  shift >= 32 gives 0
  shl 1 by 33 (imm)                  -> 00000000  gives 0
  shr 0x80000000 by 32               -> 00000000  gives 0
  add wraps                          -> 00000001  32-bit wrap
  sub below 0                        -> ffffffff  32-bit wrap
  add imm 0 is the 2-operand form    -> 0000000c  r2 = r2 + r1 (2-operand)
  shl by a register >= 32            -> 00000000  gives 0
  ld mask                            -> 12005600  AND with the mask
```

### 14.2.17 indirect (run 3)

ID: `D-indirect-3`. Device run 3. Experiment: [indirect](13-experiments.md#1335-indirect).

```
indirect access:
  pld 0xc18 (r0 through the window)    -> r2 = 5a5a0000
  pld 0xd64 (parameter RAM)            -> r2 = 77777777
  pld 0x1d64 (above the window)        -> r2 = 77777777
  pst 0xd68 then ld                    -> r2 = 00c0ffee
```

### 14.2.18 control (run 3)

ID: `D-control-3`. Device run 3. Experiment: [control](13-experiments.md#1336-control).

```
control window after a run: c00:0000fff4 c04:0801af80 c08:00000000 c0c:00000001 c10:00000001 c14:000000ff c18:00000000 c1c:00000000 c20:00000000 c24:0badcafe c28:00000000 c2c:00000000 c30:00000000 c34:00000000 c38:00000000 c3c:00000000 c40:00000000 c44:00000000 c48:00000000 c4c:00000038 c50:ffff1800 c54:00000000 c58:00000004 c5c:00000000
  CPU wrote r3 (0xc24) = 0badcafe before start; the program stored r3 = 0badcafe (preload works)
```

### 14.2.19 c58 (run 3)

ID: `D-c58-3`. Device run 3. Experiment: [c58](13-experiments.md#13310-c58).

```
C58 sweep (wait 1, nothing pending):
  C58 = 0x0     -> FAULT   continued after    4867 us, CS_IRQ 00000005; 0xc4c:00000000->00000030
  C58 = 0x1     -> FAULT   continued after    4881 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000001 0xc5c:00000000->00000001
  C58 = 0x2     -> FAULT   continued after    4868 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000002
  C58 = 0x3     -> FAULT   continued after    4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000003 0xc5c:00000000->00000001
  C58 = 0x4     -> FAULT   vector    after    4881 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000004 0xc5c:00000000->00000001
  C58 = 0x5     -> FAULT   vector    after    4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000005 0xc5c:00000000->00000001
  C58 = 0x6     -> FAULT   vector    after    4866 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000006 0xc5c:00000000->00000001
  C58 = 0x7     -> FAULT   vector    after    4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000007 0xc5c:00000000->00000001
  C58 = 0x8     -> FAULT   continued after    4869 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000008
  C58 = 0x9     -> FAULT   continued after    4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000009 0xc5c:00000000->00000001
  C58 = 0xa     -> FAULT   continued after    4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->0000000a
  C58 = 0xb     -> FAULT   continued after    4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->0000000b 0xc5c:00000000->00000001
  C58 = 0xc     -> FAULT   vector    after    4881 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->0000000c 0xc5c:00000000->00000001
  C58 = 0xd     -> FAULT   vector    after    4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->0000000d 0xc5c:00000000->00000001
  C58 = 0xe     -> FAULT   vector    after    4868 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->0000000e 0xc5c:00000000->00000001
  C58 = 0xf     -> FAULT   vector    after    4868 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->0000000f 0xc5c:00000000->00000001
  C58 = 0x10    -> FAULT   continued after    4869 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000010
  C58 = 0x11    -> FAULT   continued after    4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000011 0xc5c:00000000->00000001
  C58 = 0x12    -> FAULT   continued after    4868 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000012
  C58 = 0x13    -> FAULT   continued after    4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000013 0xc5c:00000000->00000001
  C58 = 0x14    -> FAULT   vector    after    4868 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000014 0xc5c:00000000->00000001
  C58 = 0x15    -> FAULT   vector    after    4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000015 0xc5c:00000000->00000001
  C58 = 0x16    -> FAULT   vector    after    4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000016 0xc5c:00000000->00000001
  C58 = 0x17    -> FAULT   vector    after    4868 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000017 0xc5c:00000000->00000001
  C58 = 0x18    -> FAULT   continued after    4868 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000018
  C58 = 0x19    -> FAULT   continued after    4868 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000019 0xc5c:00000000->00000001
  C58 = 0x1a    -> FAULT   continued after    4869 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->0000001a
  C58 = 0x1b    -> FAULT   continued after    4869 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->0000001b 0xc5c:00000000->00000001
  C58 = 0x1c    -> FAULT   vector    after    4869 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->0000001c 0xc5c:00000000->00000001
  C58 = 0x1d    -> FAULT   vector    after    4882 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->0000001d 0xc5c:00000000->00000001
  C58 = 0x1e    -> FAULT   vector    after    4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->0000001e 0xc5c:00000000->00000001
  C58 = 0x1f    -> FAULT   vector    after    4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->0000001f 0xc5c:00000000->00000001
  C58 = 0x40    -> FAULT   continued after    4881 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000040
  C58 = 0x80    -> FAULT   continued after    4882 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000080
  C58 = 0xff    -> FAULT   vector    after    4869 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->000000ff 0xc5c:00000000->00000001
  C58 = 0x100   -> FAULT   continued after    4869 us, CS_IRQ 00000005; 0xc4c:00000000->00000030
  C58 = 0xffff  -> FAULT   vector    after    4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->000000ff 0xc5c:00000000->00000001
```

### 14.2.20 unknown (run 3)

ID: `D-unknown-3`. Device run 3. Experiment: [unknown](13-experiments.md#1338-unknown).

```
unknown opcodes (.insn op, r3, arg 1, imm; r1 = 0xd70, r3 = 0x30000003; C58 = 0, so a fault flags and continues; op 0b is the control: or r3, r1 [, #imm]):
  op 0b imm 0      -> END     continued    r3=30000d73; 0x028:00000150->000001cc; control 0xc4c:00000000->00000070
  op 0b imm d74    -> END     continued    r3=00000d74; 0x028:00000048->000000c4; control 0xc4c:00000000->00000070
  op 08 imm 0      -> END     continued    no register changed; 0x028:00000140->000001bc; control 0xc4c:00000000->00000070
  op 08 imm d74    -> END     continued    no register changed; 0x028:00000038->000000b4; control 0xc4c:00000000->00000070
  op 09 imm 0      -> FAULT   continued    no register changed; 0x028:00000130->000001ac; control 0xc4c:00000000->00000070
  op 09 imm d74    -> FAULT   continued    no register changed; 0x028:00000028->000000a4; control 0xc4c:00000000->00000070
  op 0f imm 0      -> END     r6=60000006  no register changed; 0x028:00000120->0000019c; control 0xc4c:00000000->00000070
  op 0f imm d74    -> END     r6=60000006  no register changed; 0x028:00000018->00000094; control 0xc4c:00000000->00000070
  op 10 imm 0      -> END     continued    no register changed; 0x028:00000110->0000018c; control 0xc4c:00000000->00000070
  op 10 imm d74    -> END     continued    no register changed; 0x028:00000008->00000084; control 0xc4c:00000000->00000070
  op 12 imm 0      -> TIMEOUT r6=60000006  no register changed; 0x028:00000100->0000017c; control 0xc08:00000000->0000000a 0xc40:00000000->12030001 0xc4c:00000000->00000070
  op 12 imm d74    -> TIMEOUT r6=60000006  no register changed; 0x028:000001f8->00000074 0x050:00002000->00002800; control 0xc08:00000000->0000000a 0xc40:00000000->12030001 0xc44:00000000->00000d74 0xc4c:00000000->00000070
  op 15 imm 0      -> END     continued    no register changed; 0x028:000000f0->0000016c; control 0xc14:000000ff->00000000 0xc4c:00000000->00000070
  op 15 imm d74    -> END     continued    no register changed; 0x028:000001e8->00000064 0x050:00002800->00003000; control 0xc14:00000000->000000ff 0xc4c:00000000->00000070
  op 16 imm 0      -> FAULT   continued    no register changed; 0x028:000000e0->0000015c; control 0xc4c:00000000->00000070
  op 16 imm d74    -> FAULT   continued    no register changed; 0x028:000001d8->00000054 0x050:00003000->00003800; control 0xc4c:00000000->00000070
  op 1a imm 0      -> FAULT   continued    no register changed; 0x028:000000d0->0000014c; control 0xc4c:00000000->00000070
  op 1a imm d74    -> FAULT   continued    no register changed; 0x028:000001c8->00000044 0x050:00003800->00000000; control 0xc4c:00000000->00000070
  op 1b imm 0      -> FAULT   continued    no register changed; 0x028:000000c0->0000013c; control 0xc4c:00000000->00000070
  op 1b imm d74    -> FAULT   continued    no register changed; 0x028:000001b8->00000034 0x050:00000000->00000800; control 0xc4c:00000000->00000070
  op 1c imm 0      -> FAULT   continued    no register changed; 0x028:000000b0->0000012c; control 0xc4c:00000000->00000070
  op 1c imm d74    -> FAULT   continued    no register changed; 0x028:000001a8->00000024 0x050:00000800->00001000; control 0xc4c:00000000->00000070
  op 1d imm 0      -> FAULT   continued    no register changed; 0x028:000000a0->0000011c; control 0xc4c:00000000->00000070
  op 1d imm d74    -> FAULT   continued    no register changed; 0x028:00000198->00000014 0x050:00001000->00001800; control 0xc4c:00000000->00000070
  op 1e imm 0      -> FAULT   continued    no register changed; 0x028:00000090->0000010c; control 0xc4c:00000000->00000070
  op 1e imm d74    -> FAULT   continued    no register changed; 0x028:00000188->00000004 0x050:00001800->00002000; control 0xc4c:00000000->00000070
  op 1f imm 0      -> FAULT   continued    no register changed; 0x028:00000080->000000fc; control 0xc4c:00000000->00000070
  op 1f imm d74    -> FAULT   continued    no register changed; 0x028:00000178->000001f4; control 0xc4c:00000000->00000070
  op 20 imm 0      -> FAULT   continued    no register changed; 0x028:00000070->000000ec; control 0xc4c:00000000->00000070
  op 20 imm d74    -> FAULT   continued    no register changed; 0x028:00000168->000001e4; control 0xc4c:00000000->00000070
  op 40 imm 0      -> FAULT   continued    no register changed; 0x028:00000060->000000dc; control 0xc4c:00000000->00000070
  op 40 imm d74    -> FAULT   continued    no register changed; 0x028:00000158->000001d4; control 0xc4c:00000000->00000070
  op 80 imm 0      -> FAULT   continued    no register changed; 0x028:00000050->000000cc; control 0xc4c:00000000->00000070
  op 80 imm d74    -> FAULT   continued    no register changed; 0x028:00000148->000001c4; control 0xc4c:00000000->00000070
  op ff imm 0      -> END     continued    no register changed; 0x028:00000040->000000bc; control 0xc4c:00000000->00000070
  op ff imm d74    -> END     continued    no register changed; 0x028:00000138->000001b4; control 0xc4c:00000000->00000070
```

### 14.2.21 resume (run 4)

ID: `D-resume-4`. Device run 4. Experiment: [resume](13-experiments.md#13311-resume).

```
start address:
  C58 0, C4C mark          -> END     started at 0; after: C58 0 C5C 0 C4C 20 CS_IRQ 00000001
  C58 1, C4C mark          -> END     started at 0; after: C58 1 C5C 1 C4C 20 CS_IRQ 00000001
  C58 2, C4C mark          -> END     started r0=57a27000 r1=3a2c3a2c; after: C58 2 C5C 0 C4C 20 CS_IRQ 00000001
  C58 4, C4C mark          -> END     started at 0; after: C58 4 C5C 0 C4C 20 CS_IRQ 00000001
  C5C 1, C58 0, C4C mark   -> END     started at 0; after: C58 0 C5C 1 C4C 20 CS_IRQ 00000001
  C5C 1, C58 1, C4C mark   -> END     started at 0; after: C58 1 C5C 1 C4C 20 CS_IRQ 00000001
```

### 14.2.22 wait (run 4)

ID: `D-wait-4`. Device run 4. Experiment: [wait](13-experiments.md#1337-wait).

```
WAIT:
  wait 0  -> FAULT   fault vector after   4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000004 0xc5c:00000000->00000001
  wait 1  -> FAULT   fault vector after   4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000004 0xc5c:00000000->00000001
  wait 2  -> FAULT   fault vector after   4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000004 0xc5c:00000000->00000001
  wait 3  -> FAULT   fault vector after   4881 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000004 0xc5c:00000000->00000001
  wait 4  -> FAULT   fault vector after   4881 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000004 0xc5c:00000000->00000001
  wait 5  -> FAULT   fault vector after   4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000004 0xc5c:00000000->00000001
  wait 6  -> FAULT   fault vector after   4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000004 0xc5c:00000000->00000001
  wait 7  -> FAULT   fault vector after   4867 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000004 0xc5c:00000000->00000001
  wait 8  -> FAULT   fault vector after   4870 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000004 0xc5c:00000000->00000001
  wait 9  -> FAULT   fault vector after   4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000004 0xc5c:00000000->00000001
  wait 10 -> FAULT   fault vector after   4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000004 0xc5c:00000000->00000001
  wait 11 -> END     passed       after     15 us, CS_IRQ 00000001; 0xc4c:00000000->00000030 0xc58:00000000->00000004
  wait 12 -> FAULT   fault vector after   4870 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000004 0xc5c:00000000->00000001
  wait 13 -> FAULT   fault vector after   4868 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000004 0xc5c:00000000->00000001
  wait 14 -> FAULT   fault vector after   4881 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000004 0xc5c:00000000->00000001
  wait 15 -> FAULT   fault vector after   4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000030 0xc58:00000000->00000004 0xc5c:00000000->00000001
```

### 14.2.23 op15 (run 4)

ID: `D-op15-4`. Device run 4. Experiment: [op15](13-experiments.md#13312-op15).

```
op 0x15 (C14 preset 0x55 by the CPU; r5 = a5a5a5a5):
  imm 0          reg 0 arg 0 -> END     C14 00000000  0xc14:000000ff->00000000 0xc4c:00000000->00000030
  imm 0          reg 5 arg 0 -> END     C14 000000ff  0xc14:00000000->000000ff 0xc4c:00000000->00000030
  imm 0          reg 0 arg 5 -> END     C14 00000000  0xc14:000000ff->00000000 0xc4c:00000000->00000030
  imm 1          reg 0 arg 0 -> END     C14 000000ff  0xc14:00000000->000000ff 0xc4c:00000000->00000030
  imm 1          reg 5 arg 0 -> END     C14 00000000  0xc14:000000ff->00000000 0xc4c:00000000->00000030
  imm 1          reg 0 arg 5 -> END     C14 000000ff  0xc14:00000000->000000ff 0xc4c:00000000->00000030
  imm 7f         reg 0 arg 0 -> END     C14 00000000  0xc14:000000ff->00000000 0xc4c:00000000->00000030
  imm 7f         reg 5 arg 0 -> END     C14 000000ff  0xc14:00000000->000000ff 0xc4c:00000000->00000030
  imm 7f         reg 0 arg 5 -> END     C14 00000000  0xc14:000000ff->00000000 0xc4c:00000000->00000030
  imm 80         reg 0 arg 0 -> END     C14 000000ff  0xc14:00000000->000000ff 0xc4c:00000000->00000030
  imm 80         reg 5 arg 0 -> END     C14 00000000  0xc14:000000ff->00000000 0xc4c:00000000->00000030
  imm 80         reg 0 arg 5 -> END     C14 000000ff  0xc14:00000000->000000ff 0xc4c:00000000->00000030
  imm fe         reg 0 arg 0 -> END     C14 00000000  0xc14:000000ff->00000000 0xc4c:00000000->00000030
  imm fe         reg 5 arg 0 -> END     C14 000000ff  0xc14:00000000->000000ff 0xc4c:00000000->00000030
  imm fe         reg 0 arg 5 -> END     C14 00000000  0xc14:000000ff->00000000 0xc4c:00000000->00000030
  imm ff         reg 0 arg 0 -> END     C14 000000ff  0xc14:00000000->000000ff 0xc4c:00000000->00000030
  imm ff         reg 5 arg 0 -> END     C14 00000000  0xc14:000000ff->00000000 0xc4c:00000000->00000030
  imm ff         reg 0 arg 5 -> END     C14 000000ff  0xc14:00000000->000000ff 0xc4c:00000000->00000030
  imm 100        reg 0 arg 0 -> END     C14 00000000  0xc14:000000ff->00000000 0xc4c:00000000->00000030
  imm 100        reg 5 arg 0 -> END     C14 000000ff  0xc14:00000000->000000ff 0xc4c:00000000->00000030
  imm 100        reg 0 arg 5 -> END     C14 00000000  0xc14:000000ff->00000000 0xc4c:00000000->00000030
  imm 12345678   reg 0 arg 0 -> END     C14 000000ff  0xc14:00000000->000000ff 0xc4c:00000000->00000030
  imm 12345678   reg 5 arg 0 -> END     C14 00000000  0xc14:000000ff->00000000 0xc4c:00000000->00000030
  imm 12345678   reg 0 arg 5 -> END     C14 000000ff  0xc14:00000000->000000ff 0xc4c:00000000->00000030
```

### 14.2.24 op12 (run 5)

ID: `D-op12-5`. Device run 5. Experiment: [op12](13-experiments.md#13313-op12).

```
op 0x12 (r2 = expected value, arg = register offset, imm = mask):
  [0xd60] & ffffffff == r2 (1234)    -> BLOCKED  TIMEOUT after  50030 us; 0xc08:00000000->0000000a 0xc40:00000000->12020d60 0xc44:00000000->ffffffff 0xc4c:00000000->00000040
  [0xd60] & ffffffff == r2 (9999)    -> BLOCKED  TIMEOUT after  50026 us; 0xc08:00000000->0000000a 0xc40:00000000->12020d60 0xc44:00000000->ffffffff 0xc4c:00000000->00000040
  [0xd60] & 0 == r2 (0)              -> BLOCKED  TIMEOUT after  50041 us; 0xc08:00000000->0000000a 0xc40:00000000->12020d60 0xc4c:00000000->00000040
  [0xd60] & ff00 == r2 (1200)        -> BLOCKED  TIMEOUT after  50028 us; 0xc08:00000000->0000000a 0xc40:00000000->12020d60 0xc44:00000000->0000ff00 0xc4c:00000000->00000040
  [0xd60] & ff00 == r2 (0034)        -> BLOCKED  TIMEOUT after  50023 us; 0xc08:00000000->0000000a 0xc40:00000000->12020d60 0xc44:00000000->0000ff00 0xc4c:00000000->00000040
  [0xd60] & ffffffff != r2 (0)       -> BLOCKED  TIMEOUT after  50028 us; 0xc08:00000000->0000000a 0xc40:00000000->12020d60 0xc44:00000000->ffffffff 0xc4c:00000000->00000040
  [0xd64] (0) & ffffffff, r2 0       -> BLOCKED  TIMEOUT after  50036 us; 0xc08:00000000->0000000a 0xc40:00000000->12020d64 0xc44:00000000->ffffffff 0xc4c:00000000->00000040
  [0xd64] (0) & ffffffff, r2 1       -> BLOCKED  TIMEOUT after  50026 us; 0xc08:00000000->0000000a 0xc40:00000000->12020d64 0xc44:00000000->ffffffff 0xc4c:00000000->00000040
```

### 14.2.25 c58ill (run 5)

ID: `D-c58ill-5`. Device run 5. Experiment: [c58ill](13-experiments.md#13314-c58ill).

```
illegal opcode 0x09 under each C58:
  C58 = 0 -> FAULT   continued after  4870 us, CS_IRQ 00000005; 0xc4c:00000000->00000038
  C58 = 1 -> FAULT   continued after  4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000038 0xc58:00000000->00000001 0xc5c:00000000->00000001
  C58 = 2 -> FAULT   continued after  4865 us, CS_IRQ 00000005; 0xc4c:00000000->00000038 0xc58:00000000->00000002
  C58 = 4 -> FAULT   vector    after  4880 us, CS_IRQ 00000005; 0xc4c:00000000->00000038 0xc58:00000000->00000004 0xc5c:00000000->00000001
  C58 = 5 -> FAULT   vector    after  4869 us, CS_IRQ 00000005; 0xc4c:00000000->00000038 0xc58:00000000->00000005 0xc5c:00000000->00000001
```

### 14.2.26 pending (run 5)

ID: `D-pending-5`. Device run 5. Experiment: [pending](13-experiments.md#13315-pending).

The experiment changed between runs.

```
start after a program that left state behind:
  first: C58 1, WAIT times out (fault, continue) -> TIMEOUT, C58 0 C5C 0 C4C 0 CS_IRQ 00000000
    next start (no cleanup)  -> neither marker           C58 0 C5C 0 C4C 0 CS_IRQ 00000000
    again (no cleanup)       -> neither marker           C58 0 C5C 0 C4C 0 CS_IRQ 00000000
    after the cleanup        -> started at 0             C58 0 C5C 0 C4C 0 CS_IRQ 00000001
  first: C58 1, no fault (END) -> TIMEOUT, C58 0 C5C 0 C4C 0 CS_IRQ 00000000
    next start (no cleanup)  -> neither marker           C58 0 C5C 0 C4C 0 CS_IRQ 00000000
    again (no cleanup)       -> neither marker           C58 0 C5C 0 C4C 0 CS_IRQ 00000000
    after the cleanup        -> started at 0             C58 0 C5C 0 C4C 0 CS_IRQ 00000001
  first: C58 4, WAIT times out (fault, vector) -> TIMEOUT, C58 0 C5C 0 C4C 0 CS_IRQ 00000000
    next start (no cleanup)  -> neither marker           C58 0 C5C 0 C4C 0 CS_IRQ 00000000
    again (no cleanup)       -> neither marker           C58 0 C5C 0 C4C 0 CS_IRQ 00000000
    after the cleanup        -> started at 0             C58 0 C5C 0 C4C 0 CS_IRQ 00000001
  first: C58 0, WAIT times out (fault, continue) -> TIMEOUT, C58 0 C5C 0 C4C 0 CS_IRQ 00000000
    next start (no cleanup)  -> neither marker           C58 0 C5C 0 C4C 0 CS_IRQ 00000000
    again (no cleanup)       -> neither marker           C58 0 C5C 0 C4C 0 CS_IRQ 00000000
    after the cleanup        -> started at 0             C58 0 C5C 0 C4C 0 CS_IRQ 00000001
```

### 14.2.27 c14 (run 5)

ID: `D-c14-5`. Device run 5. Experiment: [c14](13-experiments.md#13316-c14).

```
FMSS_C14:
  wr C14 #0x12         -> C14 before 000000ff, read by the program 000000ff, after 000000ff
  wr C14 #0x00         -> C14 before 000000ff, read by the program 000000ff, after 000000ff
  wr C14 #0xff         -> C14 before 000000ff, read by the program 000000ff, after 000000ff
  wr C14 #0x0f; op15   -> C14 before 000000ff, read by the program 00000000, after 00000000
  op15                 -> C14 before 00000000, read by the program 000000ff, after 000000ff
  op15; op15           -> C14 before 000000ff, read by the program 000000ff, after 000000ff
  op15 x3              -> C14 before 000000ff, read by the program 00000000, after 00000000
```

### 14.2.28 regmap (run 6)

ID: `D-regmap-6`. Device run 6. Experiment: [regmap](13-experiments.md#1331-regmap).

```
END: r0..r7 at 0xc18..0xc34 match; SEQ_RESULT 11116666 == r6
```

### 14.2.29 startseq (run 6)

ID: `D-startseq-6`. Device run 6. Experiment: [startseq](13-experiments.md#13317-startseq).

```
start after a sequencer reset (marker program):
  reset only               -> TIMEOUT neither marker         FMSS_CTRL after 0004fff4, C08 0, CS_IRQ 00000000
  reset + clean            -> TIMEOUT neither marker         FMSS_CTRL after 0004fff4, C08 0, CS_IRQ 00000000
  reset + 0x400            -> TIMEOUT neither marker         FMSS_CTRL after 0004fff4, C08 0, CS_IRQ 00000000
  reset + preflight        -> END     started at 0           FMSS_CTRL after 0000fff4, C08 0, CS_IRQ 00000001
  reset + clean + 0x400    -> TIMEOUT neither marker         FMSS_CTRL after 0004fff4, C08 0, CS_IRQ 00000000
  reset + all (default)    -> END     started at 0           FMSS_CTRL after 0000fff4, C08 0, CS_IRQ 00000001
  no reset, nothing else   -> END     started at 0           FMSS_CTRL after 0000fff4, C08 0, CS_IRQ 00000001
```

### 14.2.30 pending (run 6)

ID: `D-pending-6`. Device run 6. Experiment: [pending](13-experiments.md#13315-pending).

The experiment changed between runs.

```
start after a program that left state behind:
  first: C58 1, WAIT times out (fault, continue) -> FAULT, C58 1 C5C 1 C4C 20 CS_IRQ 00000005
    next start (old steps)  -> ran 0, then the vector   C58 1 C5C 1 C4C 20 CS_IRQ 00000001
    again (old steps)       -> ran 0, then the vector   C58 1 C5C 1 C4C 20 CS_IRQ 00000001
    current steps (clean)   -> started at 0             C58 0 C5C 0 C4C 0 CS_IRQ 00000001
  first: C58 1, no fault (END) -> END, C58 1 C5C 1 C4C 20 CS_IRQ 00000001
    next start (old steps)  -> ran 0, then the vector   C58 1 C5C 1 C4C 20 CS_IRQ 00000001
    again (old steps)       -> ran 0, then the vector   C58 1 C5C 1 C4C 20 CS_IRQ 00000001
    current steps (clean)   -> started at 0             C58 0 C5C 0 C4C 0 CS_IRQ 00000001
  first: C58 4, WAIT times out (fault, vector) -> FAULT, C58 4 C5C 1 C4C 20 CS_IRQ 00000005
    next start (old steps)  -> started at 0             C58 4 C5C 1 C4C 20 CS_IRQ 00000001
    again (old steps)       -> started at 0             C58 4 C5C 1 C4C 20 CS_IRQ 00000001
    current steps (clean)   -> started at 0             C58 0 C5C 0 C4C 0 CS_IRQ 00000001
  first: C58 0, WAIT times out (fault, continue) -> FAULT, C58 0 C5C 0 C4C 20 CS_IRQ 00000005
    next start (old steps)  -> started at 0             C58 0 C5C 0 C4C 20 CS_IRQ 00000001
    again (old steps)       -> started at 0             C58 0 C5C 0 C4C 20 CS_IRQ 00000001
    current steps (clean)   -> started at 0             C58 0 C5C 0 C4C 0 CS_IRQ 00000001
  first: C58 2, no fault (END) -> END, C58 2 C5C 0 C4C 20 CS_IRQ 00000001
    next start (old steps)  -> started at 0             C58 2 C5C 0 C4C 20 CS_IRQ 00000001
    again (old steps)       -> started at 0             C58 2 C5C 0 C4C 20 CS_IRQ 00000001
    current steps (clean)   -> started at 0             C58 0 C5C 0 C4C 0 CS_IRQ 00000001
```

### 14.2.31 op12ack (run 6)

ID: `D-op12ack-6`. Device run 6. Experiment: [op12ack](13-experiments.md#13318-op12ack).

```
op 0x12 and a CPU write while it blocks:
  none (control)           -> blocked  TIMEOUT after  30015 us (late write at 0 us), CS_IRQ 00000008, C08 a
  CS_IRQ = 0x08 (ack b3)   -> blocked  TIMEOUT after  30016 us (late write at 2010 us), CS_IRQ 00000000, C08 a
  CS_IRQ = 0xff            -> blocked  TIMEOUT after  30013 us (late write at 2010 us), CS_IRQ 00000000, C08 a
  C08 = 0                  -> blocked  TIMEOUT after  30024 us (late write at 2010 us), CS_IRQ 00000008, C08 a
  C08 = 0xb                -> blocked  TIMEOUT after  30018 us (late write at 2010 us), CS_IRQ 00000008, C08 a
  C40 = 0                  -> blocked  TIMEOUT after  30014 us (late write at 2010 us), CS_IRQ 00000008, C08 a
  C14 = 0                  -> blocked  TIMEOUT after  30017 us (late write at 2006 us), CS_IRQ 00000008, C08 a
  C10 = 0                  -> blocked  TIMEOUT after  30024 us (late write at 2010 us), CS_IRQ 00000008, C08 a
```

### 14.2.32 op10 (run 6)

ID: `D-op10-6`. Device run 6. Experiment: [op10](13-experiments.md#13319-op10).

```
op 0x10: END, r6 600d600d (carried on), CS_IRQ read by the program right after it 00000002, CS_IRQ at the end 00000003
```

### 14.2.33 regmap (run 7)

ID: `D-regmap-7`. Device run 7. Experiment: [regmap](13-experiments.md#1331-regmap).

```
END: r0..r7 at 0xc18..0xc34 match; SEQ_RESULT 11116666 == r6
```

### 14.2.34 startseq2 (run 7)

ID: `D-startseq2-7`. Device run 7. Experiment: [startseq2](13-experiments.md#13320-startseq2).

```
reset, then one kind of access, then START:
  nothing in between     -> TIMEOUT no run   FMSS_CTRL after 0004fff4, CS_IRQ 00000000
  FMC_CTRL0 = 0          -> TIMEOUT no run   FMSS_CTRL after 0004fff4, CS_IRQ 00000000
  full snapshot          -> TIMEOUT no run   FMSS_CTRL after 0004fff4, CS_IRQ 00000000
  read window 0x000      -> TIMEOUT no run   FMSS_CTRL after 0004fff4, CS_IRQ 00000000
  read window 0x800      -> TIMEOUT no run   FMSS_CTRL after 0004fff4, CS_IRQ 00000000
  read window 0xc00      -> TIMEOUT no run   FMSS_CTRL after 0004fff4, CS_IRQ 00000000
  read window 0xd00      -> TIMEOUT no run   FMSS_CTRL after 0004fff4, CS_IRQ 00000000
  CPU writes r0          -> TIMEOUT no run   FMSS_CTRL after 0004fff4, CS_IRQ 00000000
  CPU writes 0xd60       -> TIMEOUT no run   FMSS_CTRL after 0004fff4, CS_IRQ 00000000
```

### 14.2.35 startseq3 (run 8)

ID: `D-startseq3-8`. Device run 8. Experiment: [startseq3](13-experiments.md#13321-startseq3).

```
reset, START (no preflight), optionally a CPU write of FMSS_CTRL 2 ms later:
  END-only program         -> END     r0 00000000 r6 00000000  FMSS_CTRL after 0000fff4, C40 00000000 C44 00000000, CS_IRQ 00000001
  no-op (0x08) first       -> TIMEOUT r0 00000000 r6 00000000  FMSS_CTRL after 0004fff4, C40 08000000 C44 00000000, CS_IRQ 00000000
  wr CS_IRQ first          -> TIMEOUT r0 00000000 r6 00000000  FMSS_CTRL after 0004fff4, C40 01000c0c C44 000000ff, CS_IRQ 00000000
  marker (control)         -> TIMEOUT r0 00000000 r6 00000000  FMSS_CTRL after 0004fff4, C40 05000000 C44 57a27000, CS_IRQ 00000000
  marker, then CTRL fff5   -> END     r0 57a27000 r6 600d600d  FMSS_CTRL after 0000fff4, C40 00000000 C44 00000000, CS_IRQ 00000001
  marker, then CTRL 4fff5  -> END     r0 57a27000 r6 600d600d  FMSS_CTRL after 0000fff4, C40 00000000 C44 00000000, CS_IRQ 00000001
  marker, then CTRL 40000  -> TIMEOUT r0 00000000 r6 00000000  FMSS_CTRL after 00040000, C40 05000000 C44 57a27000, CS_IRQ 00000000
  marker, then CTRL 0      -> TIMEOUT r0 00000000 r6 00000000  FMSS_CTRL after 00040000, C40 05000000 C44 57a27000, CS_IRQ 00000000
```

### 14.2.36 oneshot (run 8)

ID: `D-oneshot-8`. Device run 8. Experiment: [oneshot](13-experiments.md#13322-oneshot).

```
C58 bit 1 one-shot:
  CPU sets bit 1, END (control)    -> END     started at 0             C58 2 C5C 0 C4C 18 CS_IRQ 00000001
  program sets bit 1, END          -> END     started at 0             C58 2 C5C 0 C4C 20 CS_IRQ 00000001
  CPU sets bit 1, fault, END       -> FAULT   started at 0             C58 2 C5C 0 C4C 20 CS_IRQ 00000005
  program sets bit 1, fault, END   -> FAULT   started at 0             C58 2 C5C 0 C4C 28 CS_IRQ 00000005
```

### 14.2.37 c50 (run 9)

ID: `D-c50-9`. Device run 9. Experiment: [c50](13-experiments.md#13323-c50).

```
FMSS_C50 seen by the fault handler:
  timeout, handler acks                    -> FAULT   C50 in the handler ffff1800, after the ack ffff1800, after the end ffff1800
  timeout, no ack                          -> FAULT   C50 in the handler ffff1800, after the ack ffff1800, after the end ffff1800
  illegal opcode, handler acks             -> FAULT   C50 in the handler ffff1800, after the ack ffff1800, after the end ffff1800
  program jumps to the handler (no fault)  -> END     C50 in the handler ffff1800, after the ack ffff1800, after the end ffff1800
```

### 14.2.38 holdout (run 9)

ID: `D-holdout-9`. Device run 9. Experiment: [holdout](13-experiments.md#13324-holdout).

```
held-out combinations:
  three timeouts, C58 0          -> FAULT      4877 us  CS_IRQ 00000004  r2 00000000 r3 00000000 r4 00000000 r6 00000000  C14 ff->f6
  shl 1 by 31 / shr by 31        -> END          18 us  CS_IRQ 00000001  r2 80000000 r3 00000001 r4 00000000 r6 600d600d  C14 f0->f0
  pld 0xc30 reads r6             -> END          18 us  CS_IRQ 00000001  r2 6666aaaa r3 00000000 r4 00000000 r6 6666aaaa  C14 f0->f0
  op15 x4                        -> END        9710 us  CS_IRQ 00000001  r2 000000ff r3 00000000 r4 00000000 r6 00000000  C14 f0->ff
  loop of 100                    -> END          53 us  CS_IRQ 00000001  r2 00000000 r3 00000000 r4 00000000 r6 600d600d  C14 ff->ff
  wait 11, timeout, op10         -> FAULT      4876 us  CS_IRQ 00000007  r2 00000000 r3 00000000 r4 00000000 r6 600d600d  C14 ff->ff
  sti then ldi at scratch+0x200  -> END          19 us  CS_IRQ 00000001  r2 13572468 r3 08052f03 r4 13572468 r6 600d600d  C14 ff->ff
```

### 14.2.39 holdout (run 10)

ID: `D-holdout-10`. Device run 10. Experiment: [holdout](13-experiments.md#13324-holdout).

```
held-out combinations (followed to the END after a fault):
  three timeouts, C58 0          -> FAULT     14618 us  CS_IRQ 00000005  r2 00000000 r3 00000000 r4 00000000 r6 600d600d  C14 ff->ff  first fault at 4864 us
  shl 1 by 31 / shr by 31        -> END          18 us  CS_IRQ 00000001  r2 80000000 r3 00000001 r4 00000000 r6 600d600d  C14 ff->ff
  pld 0xc30 reads r6             -> END          18 us  CS_IRQ 00000001  r2 6666aaaa r3 00000000 r4 00000000 r6 6666aaaa  C14 ff->ff
  op15 x4                        -> END        9717 us  CS_IRQ 00000001  r2 000000ff r3 00000000 r4 00000000 r6 00000000  C14 ff->ff
  loop of 100                    -> END          55 us  CS_IRQ 00000001  r2 00000000 r3 00000000 r4 00000000 r6 600d600d  C14 ff->ff
  wait 11, timeout, op10         -> FAULT      4866 us  CS_IRQ 00000007  r2 00000000 r3 00000000 r4 00000000 r6 600d600d  C14 ff->ff  first fault at 4848 us
  sti then ldi at scratch+0x200  -> END          18 us  CS_IRQ 00000001  r2 13572468 r3 08052f43 r4 13572468 r6 600d600d  C14 ff->ff
```

### 14.2.40 op12wait (run 11)

ID: `D-op12wait-11`. Device run 11. Experiment: [op12wait](13-experiments.md#13325-op12wait).

```
opcode 0x12 with the event in the register field:
  0x12 reg 11 (event 11, set at idle)            -> TIMEOUT BLOCKED after  20015 us, CS_IRQ 00000008, C08 a
  0x12 reg 0 after a command byte (event 0 set)  -> TIMEOUT BLOCKED after  20014 us, CS_IRQ 00000008, C08 a
  wait 0 after a command byte (control)          -> END     passed after     17 us, CS_IRQ 00000001, C08 0
  0x12 reg 3 (event 3, not set)                  -> TIMEOUT BLOCKED after  20015 us, CS_IRQ 00000008, C08 a
  0x12 reg 11, arg 5, imm 0x40                   -> TIMEOUT BLOCKED after  20015 us, CS_IRQ 00000008, C08 a
```

### 14.2.41 ev11 (run 11)

ID: `D-ev11-11`. Device run 11. Experiment: [ev11](13-experiments.md#13326-ev11).

```
event 11 around a block erase (tBERS ~1-2 ms):
  idle CE0, wait 11 (control)        -> END         17 us, CS_IRQ 00000001
  erase CE0, end at once (control)   -> END         17 us, CS_IRQ 00000001
  erase CE0, wait 11                 -> END         17 us, CS_IRQ 00000001
  erase CE0, select CE1, wait 11     -> END         17 us, CS_IRQ 00000001
  erase CE0, deselect, wait 11       -> END         16 us, CS_IRQ 00000001
  erase CE0, wait 11, wait 11        -> END         17 us, CS_IRQ 00000001
```

### 14.2.42 ev5 (run 11)

ID: `D-ev5-11`. Device run 11. Experiment: [ev5](13-experiments.md#13327-ev5).

```
event 5 and the status byte (b6 = ready, b0 = fail):
  idle: poll, wait 5                 -> END         17 us, status before wait 5 c0, after c0, last 00, CS_IRQ 00000001
  erase, poll, wait 5                -> END        172 us, status before wait 5 81, after c0, last 00, CS_IRQ 00000001
  erase, poll, wait 5, re-latch x8   -> FAULT    39160 us, status before wait 5 81, after c0, last c0, CS_IRQ 00000005
  erase, wait 11, poll, wait 5       -> END        172 us, status before wait 5 81, after c0, last 00, CS_IRQ 00000001
```

### 14.2.43 opsweep (run 11)

ID: `D-opsweep-11`. Device run 11. Experiment: [opsweep](13-experiments.md#13328-opsweep).

```
opcode classes (first operand form):
  END continued 4.9 ms CS_IRQ 01 window              15
  END continued fast CS_IRQ 01 window                08 ff
  END continued fast CS_IRQ 03 window                10
  FAULT continued 4.9 ms CS_IRQ 05 window            09 16 1a 1b 1c 1d 1e 1f 20 21 22 23 24 25 26 27 28 29 2a 2b 2c 2d 2e 2f 30 31 32 33 34 35 36 37 38 39 3a 3b 3c 3d 3e 3f 40 41 42 43 44 45 46 47 48 49 4a 4b 4c 4d 4e 4f 50 51 52 53 54 55 56 57 58 59 5a 5b 5c 5d 5e 5f 60 61 62 63 64 65 66 67 68 69 6a 6b 6c 6d 6e 6f 70 71 72 73 74 75 76 77 78 79 7a 7b 7c 7d 7e 7f 80 81 82 83 84 85 86 87 88 89 8a 8b 8c 8d 8e 8f 90 91 92 93 94 95 96 97 98 99 9a 9b 9c 9d 9e 9f a0 a1 a2 a3 a4 a5 a6 a7 a8 a9 aa ab ac ad ae af b0 b1 b2 b3 b4 b5 b6 b7 b8 b9 ba bb bc bd be bf c0 c1 c2 c3 c4 c5 c6 c7 c8 c9 ca cb cc cd ce cf d0 d1 d2 d3 d4 d5 d6 d7 d8 d9 da db dc dd de df e0 e1 e2 e3 e4 e5 e6 e7 e8 e9 ea eb ec ed ee ef f0 f1 f2 f3 f4 f5 f6 f7 f8 f9 fa fb fc fd fe
  FAULT continued 5.0 ms CS_IRQ 05 window            
  FAULT continued 5.1 ms CS_IRQ 05 window            
  FAULT continued 5.2 ms CS_IRQ 05 window            
  FAULT stopped 4.9 ms CS_IRQ 04 window              0f
  TIMEOUT stopped 20.0 ms CS_IRQ 08 window           12
all runs:
  op 08 reg 3, arg 1, imm 0          -> END continued fast CS_IRQ 01 window; 0x028:0000007c->000000f8 0xc14:000000f8->000000ff
  op 08 reg 3, arg 0xd70, imm 0xd74  -> END continued fast CS_IRQ 01 window; 0x028:00000174->000001f0
  op 08 reg 0, arg 0, imm 0xffffffff -> END continued fast CS_IRQ 01 window; 0x028:0000006c->000000e8
  op 09 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000164->000001e0
  op 09 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000005c->000000d8
  op 09 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000154->000001d0
  op 0f reg 3, arg 1, imm 0          -> FAULT stopped 4.9 ms CS_IRQ 04 window; 0x028:0000004c->000000c8 0xc08:00000000->00000004 0xc14:000000ff->000000f6 0xc40:00000000->e7922103 0xc44:00000000->0a000076
  op 0f reg 3, arg 0xd70, imm 0xd74  -> FAULT stopped 4.9 ms CS_IRQ 04 window; 0x028:00000144->000001c0 0xc08:00000000->00000004 0xc14:000000f0->000000f6 0xc40:00000000->22833001 0xc44:00000000->e3500b01
  op 0f reg 0, arg 0, imm 0xffffffff -> FAULT stopped 4.9 ms CS_IRQ 04 window; 0x028:0000003c->000000b8 0xc08:00000000->00000004 0xc14:000000f0->000000f6 0xc40:00000000->eaffff86 0xc44:00000000->05982c0c
  op 10 reg 3, arg 1, imm 0          -> END continued fast CS_IRQ 03 window; 0x028:00000134->000001b0
  op 10 reg 3, arg 0xd70, imm 0xd74  -> END continued fast CS_IRQ 03 window; 0x028:0000002c->000000a8
  op 10 reg 0, arg 0, imm 0xffffffff -> END continued fast CS_IRQ 03 window; 0x028:00000124->000001a0
  op 12 reg 3, arg 1, imm 0          -> TIMEOUT stopped 20.0 ms CS_IRQ 08 window; 0x028:0000001c->00000098 0xc08:00000000->0000000a 0xc40:00000000->12030001
  op 12 reg 3, arg 0xd70, imm 0xd74  -> TIMEOUT stopped 20.0 ms CS_IRQ 08 window; 0x028:00000114->00000190 0xc08:00000000->0000000a 0xc40:00000000->12030d70 0xc44:00000000->00000d74
  op 12 reg 0, arg 0, imm 0xffffffff -> TIMEOUT stopped 20.0 ms CS_IRQ 08 window; 0x028:0000000c->00000088 0xc08:00000000->0000000a 0xc40:00000000->12000000 0xc44:00000000->ffffffff
  op 15 reg 3, arg 1, imm 0          -> END continued 4.9 ms CS_IRQ 01 window; 0x028:00000104->00000180 0xc14:000000f0->00000000
  op 15 reg 3, arg 0xd70, imm 0xd74  -> END continued fast CS_IRQ 01 window; 0x028:000001fc->00000078 0x050:00003800->00000000 0xc14:00000000->000000ff
  op 15 reg 0, arg 0, imm 0xffffffff -> END continued 4.9 ms CS_IRQ 01 window; 0x028:000000f4->00000170 0xc14:000000ff->00000000
  op 16 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ec->00000068 0x050:00000000->00000800 0xc14:00000000->000000ff
  op 16 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000e4->00000160
  op 16 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001dc->00000058 0x050:00000800->00001000
  op 1a reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000d4->00000150
  op 1a reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001cc->00000048 0x050:00001000->00001800
  op 1a reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000c4->00000140
  op 1b reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001bc->00000038 0x050:00001800->00002000
  op 1b reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000b4->00000130
  op 1b reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ac->00000028 0x050:00002000->00002800
  op 1c reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000a4->00000120
  op 1c reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000019c->00000018 0x050:00002800->00003000
  op 1c reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000094->00000110
  op 1d reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000018c->00000008 0x050:00003000->00003800
  op 1d reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000084->00000100
  op 1d reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000017c->000001f8
  op 1e reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000074->000000f0
  op 1e reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000016c->000001e8
  op 1e reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000064->000000e0
  op 1f reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000015c->000001d8
  op 1f reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000054->000000d0
  op 1f reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000014c->000001c8
  op 20 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000044->000000c0
  op 20 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000013c->000001b8
  op 20 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000034->000000b0
  op 21 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000012c->000001a8
  op 21 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000024->000000a0
  op 21 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000011c->00000198
  op 22 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000014->00000090
  op 22 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000010c->00000188
  op 22 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000004->00000080
  op 23 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000fc->00000178
  op 23 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001f4->00000070 0x050:00003800->00000000
  op 23 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ec->00000168
  op 24 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001e4->00000060 0x050:00000000->00000800
  op 24 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000dc->00000158
  op 24 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001d4->00000050 0x050:00000800->00001000
  op 25 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000cc->00000148
  op 25 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001c4->00000040 0x050:00001000->00001800
  op 25 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000bc->00000138
  op 26 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001b4->00000030 0x050:00001800->00002000
  op 26 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ac->00000128
  op 26 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001a4->00000020 0x050:00002000->00002800
  op 27 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000009c->00000118
  op 27 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000194->00000010 0x050:00002800->00003000
  op 27 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000008c->00000108
  op 28 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000184->00000000 0x050:00003000->00003800
  op 28 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000007c->000000f8
  op 28 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000174->000001f0
  op 29 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000006c->000000e8
  op 29 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000164->000001e0
  op 29 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000005c->000000d8
  op 2a reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000154->000001d0
  op 2a reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000004c->000000c8
  op 2a reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000144->000001c0
  op 2b reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000003c->000000b8
  op 2b reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000134->000001b0
  op 2b reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000002c->000000a8
  op 2c reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000124->000001a0
  op 2c reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000001c->00000098
  op 2c reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000114->00000190
  op 2d reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000000c->00000088
  op 2d reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000104->00000180
  op 2d reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001fc->00000078 0x050:00003000->00003800
  op 2e reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000f4->00000170
  op 2e reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ec->00000068 0x050:00003800->00000000
  op 2e reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000e4->00000160
  op 2f reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001dc->00000058 0x050:00000000->00000800
  op 2f reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000d4->00000150
  op 2f reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001cc->00000048 0x050:00000800->00001000
  op 30 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000c4->00000140
  op 30 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 5.0 ms CS_IRQ 05 window; 0x028:000001bc->00000038 0x050:00001000->00001800
  op 30 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000b4->00000130
  op 31 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ac->00000028 0x050:00001800->00002000
  op 31 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000a4->00000120
  op 31 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000019c->00000018 0x050:00002000->00002800
  op 32 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000094->00000110
  op 32 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000018c->00000008 0x050:00002800->00003000
  op 32 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000084->00000100
  op 33 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000017c->000001f8
  op 33 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000074->000000f0
  op 33 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000016c->000001e8
  op 34 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000064->000000e0
  op 34 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000015c->000001d8
  op 34 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000054->000000d0
  op 35 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000014c->000001c8
  op 35 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000044->000000c0
  op 35 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000013c->000001b8
  op 36 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000034->000000b0
  op 36 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000012c->000001a8
  op 36 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000024->000000a0
  op 37 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000011c->00000198
  op 37 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000014->00000090
  op 37 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000010c->00000188
  op 38 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000004->00000080
  op 38 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000fc->00000178
  op 38 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001f4->00000070 0x050:00003000->00003800
  op 39 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ec->00000168
  op 39 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001e4->00000060 0x050:00003800->00000000
  op 39 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000dc->00000158
  op 3a reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001d4->00000050 0x050:00000000->00000800
  op 3a reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000cc->00000148
  op 3a reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001c4->00000040 0x050:00000800->00001000
  op 3b reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000bc->00000138
  op 3b reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001b4->00000030 0x050:00001000->00001800
  op 3b reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ac->00000128
  op 3c reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001a4->00000020 0x050:00001800->00002000
  op 3c reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000009c->00000118
  op 3c reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000194->00000010 0x050:00002000->00002800
  op 3d reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000008c->00000108
  op 3d reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000184->00000000 0x050:00002800->00003000
  op 3d reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000007c->000000f8
  op 3e reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000174->000001f0
  op 3e reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000006c->000000e8
  op 3e reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000164->000001e0
  op 3f reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000005c->000000d8
  op 3f reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000154->000001d0
  op 3f reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000004c->000000c8
  op 40 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000144->000001c0
  op 40 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000003c->000000b8
  op 40 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000134->000001b0
  op 41 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000002c->000000a8
  op 41 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000124->000001a0
  op 41 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000001c->00000098
  op 42 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000114->00000190
  op 42 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000000c->00000088
  op 42 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000104->00000180
  op 43 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001fc->00000078 0x050:00002800->00003000
  op 43 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000f4->00000170
  op 43 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ec->00000068 0x050:00003000->00003800
  op 44 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000e4->00000160
  op 44 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001dc->00000058 0x050:00003800->00000000
  op 44 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000d4->00000150
  op 45 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001cc->00000048 0x050:00000000->00000800
  op 45 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000c4->00000140
  op 45 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001bc->00000038 0x050:00000800->00001000
  op 46 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000b4->00000130
  op 46 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ac->00000028 0x050:00001000->00001800
  op 46 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000a4->00000120
  op 47 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000019c->00000018 0x050:00001800->00002000
  op 47 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000094->00000110
  op 47 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000018c->00000008 0x050:00002000->00002800
  op 48 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000084->00000100
  op 48 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000017c->000001f8
  op 48 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000074->000000f0
  op 49 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000016c->000001e8
  op 49 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000064->000000e0
  op 49 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000015c->000001d8
  op 4a reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000054->000000d0
  op 4a reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000014c->000001c8
  op 4a reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000044->000000c0
  op 4b reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000013c->000001b8
  op 4b reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000034->000000b0
  op 4b reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000012c->000001a8
  op 4c reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000024->000000a0
  op 4c reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000011c->00000198
  op 4c reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000014->00000090
  op 4d reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000010c->00000188
  op 4d reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000004->00000080
  op 4d reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000fc->00000178
  op 4e reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001f4->00000070 0x050:00002800->00003000
  op 4e reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ec->00000168
  op 4e reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001e4->00000060 0x050:00003000->00003800
  op 4f reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000dc->00000158
  op 4f reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001d4->00000050 0x050:00003800->00000000
  op 4f reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000cc->00000148
  op 50 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001c4->00000040 0x050:00000000->00000800
  op 50 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000bc->00000138
  op 50 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001b4->00000030 0x050:00000800->00001000
  op 51 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ac->00000128
  op 51 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001a4->00000020 0x050:00001000->00001800
  op 51 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000009c->00000118
  op 52 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000194->00000010 0x050:00001800->00002000
  op 52 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000008c->00000108
  op 52 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000184->00000000 0x050:00002000->00002800
  op 53 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000007c->000000f8
  op 53 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000174->000001f0
  op 53 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000006c->000000e8
  op 54 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000164->000001e0
  op 54 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000005c->000000d8
  op 54 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000154->000001d0
  op 55 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000004c->000000c8
  op 55 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 5.1 ms CS_IRQ 05 window; 0x028:00000144->000001c0
  op 55 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000003c->000000b8
  op 56 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000134->000001b0
  op 56 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000002c->000000a8
  op 56 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000124->000001a0
  op 57 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000001c->00000098
  op 57 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000114->00000190
  op 57 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000000c->00000088
  op 58 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000104->00000180
  op 58 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001fc->00000078 0x050:00002000->00002800
  op 58 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000f4->00000170
  op 59 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ec->00000068 0x050:00002800->00003000
  op 59 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000e4->00000160
  op 59 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001dc->00000058 0x050:00003000->00003800
  op 5a reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000d4->00000150
  op 5a reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001cc->00000048 0x050:00003800->00000000
  op 5a reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000c4->00000140
  op 5b reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001bc->00000038 0x050:00000000->00000800
  op 5b reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000b4->00000130
  op 5b reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ac->00000028 0x050:00000800->00001000
  op 5c reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000a4->00000120
  op 5c reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000019c->00000018 0x050:00001000->00001800
  op 5c reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000094->00000110
  op 5d reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000018c->00000008 0x050:00001800->00002000
  op 5d reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000084->00000100
  op 5d reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000017c->000001f8
  op 5e reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000074->000000f0
  op 5e reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000016c->000001e8
  op 5e reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000064->000000e0
  op 5f reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000015c->000001d8
  op 5f reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000054->000000d0
  op 5f reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000014c->000001c8
  op 60 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000044->000000c0
  op 60 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000013c->000001b8
  op 60 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000034->000000b0
  op 61 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000012c->000001a8
  op 61 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000024->000000a0
  op 61 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000011c->00000198
  op 62 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000014->00000090
  op 62 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000010c->00000188
  op 62 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000004->00000080
  op 63 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000fc->00000178
  op 63 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001f4->00000070 0x050:00002000->00002800
  op 63 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ec->00000168
  op 64 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001e4->00000060 0x050:00002800->00003000
  op 64 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000dc->00000158
  op 64 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001d4->00000050 0x050:00003000->00003800
  op 65 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000cc->00000148
  op 65 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001c4->00000040 0x050:00003800->00000000
  op 65 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000bc->00000138
  op 66 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001b4->00000030 0x050:00000000->00000800
  op 66 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ac->00000128
  op 66 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001a4->00000020 0x050:00000800->00001000
  op 67 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000009c->00000118
  op 67 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000194->00000010 0x050:00001000->00001800
  op 67 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000008c->00000108
  op 68 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000184->00000000 0x050:00001800->00002000
  op 68 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000007c->000000f8
  op 68 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000174->000001f0
  op 69 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000006c->000000e8
  op 69 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000164->000001e0
  op 69 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000005c->000000d8
  op 6a reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000154->000001d0
  op 6a reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000004c->000000c8
  op 6a reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000144->000001c0
  op 6b reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000003c->000000b8
  op 6b reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000134->000001b0
  op 6b reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000002c->000000a8
  op 6c reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000124->000001a0
  op 6c reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000001c->00000098
  op 6c reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000114->00000190
  op 6d reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000000c->00000088
  op 6d reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000104->00000180
  op 6d reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001fc->00000078 0x050:00001800->00002000
  op 6e reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000f4->00000170
  op 6e reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ec->00000068 0x050:00002000->00002800
  op 6e reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000e4->00000160
  op 6f reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001dc->00000058 0x050:00002800->00003000
  op 6f reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000d4->00000150
  op 6f reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001cc->00000048 0x050:00003000->00003800
  op 70 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000c4->00000140
  op 70 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001bc->00000038 0x050:00003800->00000000
  op 70 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000b4->00000130
  op 71 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ac->00000028 0x050:00000000->00000800
  op 71 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000a4->00000120
  op 71 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000019c->00000018 0x050:00000800->00001000
  op 72 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000094->00000110
  op 72 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000018c->00000008 0x050:00001000->00001800
  op 72 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000084->00000100
  op 73 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000017c->000001f8
  op 73 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000074->000000f0
  op 73 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000016c->000001e8
  op 74 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000064->000000e0
  op 74 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000015c->000001d8
  op 74 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000054->000000d0
  op 75 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000014c->000001c8
  op 75 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000044->000000c0
  op 75 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000013c->000001b8
  op 76 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000034->000000b0
  op 76 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000012c->000001a8
  op 76 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000024->000000a0
  op 77 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000011c->00000198
  op 77 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000014->00000090
  op 77 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000010c->00000188
  op 78 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000004->00000080
  op 78 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000fc->00000178
  op 78 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001f4->00000070 0x050:00001800->00002000
  op 79 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ec->00000168
  op 79 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001e4->00000060 0x050:00002000->00002800
  op 79 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000dc->00000158
  op 7a reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001d4->00000050 0x050:00002800->00003000
  op 7a reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000cc->00000148
  op 7a reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001c4->00000040 0x050:00003000->00003800
  op 7b reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000bc->00000138
  op 7b reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001b4->00000030 0x050:00003800->00000000
  op 7b reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ac->00000128
  op 7c reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001a4->00000020 0x050:00000000->00000800
  op 7c reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000009c->00000118
  op 7c reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000194->00000010 0x050:00000800->00001000
  op 7d reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000008c->00000108
  op 7d reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000184->00000000 0x050:00001000->00001800
  op 7d reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000007c->000000f8
  op 7e reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000174->000001f0
  op 7e reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000006c->000000e8
  op 7e reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000164->000001e0
  op 7f reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000005c->000000d8
  op 7f reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000154->000001d0
  op 7f reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000004c->000000c8
  op 80 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000144->000001c0
  op 80 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000003c->000000b8
  op 80 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000134->000001b0
  op 81 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000002c->000000a8
  op 81 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000124->000001a0
  op 81 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000001c->00000098
  op 82 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000114->00000190
  op 82 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000000c->00000088
  op 82 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000104->00000180
  op 83 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001fc->00000078 0x050:00001000->00001800
  op 83 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000f4->00000170
  op 83 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ec->00000068 0x050:00001800->00002000
  op 84 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000e4->00000160
  op 84 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001dc->00000058 0x050:00002000->00002800
  op 84 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000d4->00000150
  op 85 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001cc->00000048 0x050:00002800->00003000
  op 85 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000c4->00000140
  op 85 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001bc->00000038 0x050:00003000->00003800
  op 86 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000b4->00000130
  op 86 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ac->00000028 0x050:00003800->00000000
  op 86 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000a4->00000120
  op 87 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000019c->00000018 0x050:00000000->00000800
  op 87 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000094->00000110
  op 87 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000018c->00000008 0x050:00000800->00001000
  op 88 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000084->00000100
  op 88 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000017c->000001f8
  op 88 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000074->000000f0
  op 89 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000016c->000001e8
  op 89 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000064->000000e0
  op 89 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000015c->000001d8
  op 8a reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000054->000000d0
  op 8a reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000014c->000001c8
  op 8a reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000044->000000c0
  op 8b reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000013c->000001b8
  op 8b reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000034->000000b0
  op 8b reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000012c->000001a8
  op 8c reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000024->000000a0
  op 8c reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000011c->00000198
  op 8c reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000014->00000090
  op 8d reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000010c->00000188
  op 8d reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000004->00000080
  op 8d reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000fc->00000178
  op 8e reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001f4->00000070 0x050:00001000->00001800
  op 8e reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ec->00000168
  op 8e reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001e4->00000060 0x050:00001800->00002000
  op 8f reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000dc->00000158
  op 8f reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001d4->00000050 0x050:00002000->00002800
  op 8f reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000cc->00000148
  op 90 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001c4->00000040 0x050:00002800->00003000
  op 90 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000bc->00000138
  op 90 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001b4->00000030 0x050:00003000->00003800
  op 91 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ac->00000128
  op 91 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001a4->00000020 0x050:00003800->00000000
  op 91 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000009c->00000118
  op 92 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000194->00000010 0x050:00000000->00000800
  op 92 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000008c->00000108
  op 92 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000184->00000000 0x050:00000800->00001000
  op 93 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000007c->000000f8
  op 93 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000174->000001f0
  op 93 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000006c->000000e8
  op 94 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000164->000001e0
  op 94 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000005c->000000d8
  op 94 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000154->000001d0
  op 95 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000004c->000000c8
  op 95 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000144->000001c0
  op 95 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000003c->000000b8
  op 96 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000134->000001b0
  op 96 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000002c->000000a8
  op 96 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000124->000001a0
  op 97 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000001c->00000098
  op 97 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000114->00000190
  op 97 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000000c->00000088
  op 98 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000104->00000180
  op 98 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001fc->00000078 0x050:00000800->00001000
  op 98 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000f4->00000170
  op 99 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ec->00000068 0x050:00001000->00001800
  op 99 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000e4->00000160
  op 99 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001dc->00000058 0x050:00001800->00002000
  op 9a reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000d4->00000150
  op 9a reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001cc->00000048 0x050:00002000->00002800
  op 9a reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000c4->00000140
  op 9b reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001bc->00000038 0x050:00002800->00003000
  op 9b reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000b4->00000130
  op 9b reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ac->00000028 0x050:00003000->00003800
  op 9c reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000a4->00000120
  op 9c reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000019c->00000018 0x050:00003800->00000000
  op 9c reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000094->00000110
  op 9d reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000018c->00000008 0x050:00000000->00000800
  op 9d reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000084->00000100
  op 9d reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000017c->000001f8
  op 9e reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000074->000000f0
  op 9e reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000016c->000001e8
  op 9e reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000064->000000e0
  op 9f reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000015c->000001d8
  op 9f reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000054->000000d0
  op 9f reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000014c->000001c8
  op a0 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000044->000000c0
  op a0 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000013c->000001b8
  op a0 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000034->000000b0
  op a1 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000012c->000001a8
  op a1 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000024->000000a0
  op a1 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000011c->00000198
  op a2 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000014->00000090
  op a2 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000010c->00000188
  op a2 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000004->00000080
  op a3 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000fc->00000178
  op a3 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001f4->00000070 0x050:00000800->00001000
  op a3 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ec->00000168
  op a4 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001e4->00000060 0x050:00001000->00001800
  op a4 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000dc->00000158
  op a4 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001d4->00000050 0x050:00001800->00002000
  op a5 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000cc->00000148
  op a5 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001c4->00000040 0x050:00002000->00002800
  op a5 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000bc->00000138
  op a6 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001b4->00000030 0x050:00002800->00003000
  op a6 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ac->00000128
  op a6 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001a4->00000020 0x050:00003000->00003800
  op a7 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000009c->00000118
  op a7 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000194->00000010 0x050:00003800->00000000
  op a7 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000008c->00000108
  op a8 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000184->00000000 0x050:00000000->00000800
  op a8 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000007c->000000f8
  op a8 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000174->000001f0
  op a9 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000006c->000000e8
  op a9 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000164->000001e0
  op a9 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000005c->000000d8
  op aa reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000154->000001d0
  op aa reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000004c->000000c8
  op aa reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000144->000001c0
  op ab reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000003c->000000b8
  op ab reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000134->000001b0
  op ab reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000002c->000000a8
  op ac reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000124->000001a0
  op ac reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000001c->00000098
  op ac reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000114->00000190
  op ad reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000000c->00000088
  op ad reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000104->00000180
  op ad reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001fc->00000078 0x050:00000000->00000800
  op ae reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000f4->00000170
  op ae reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ec->00000068 0x050:00000800->00001000
  op ae reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000e4->00000160
  op af reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001dc->00000058 0x050:00001000->00001800
  op af reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000d4->00000150
  op af reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001cc->00000048 0x050:00001800->00002000
  op b0 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000c4->00000140
  op b0 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001bc->00000038 0x050:00002000->00002800
  op b0 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000b4->00000130
  op b1 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ac->00000028 0x050:00002800->00003000
  op b1 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000a4->00000120
  op b1 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000019c->00000018 0x050:00003000->00003800
  op b2 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000094->00000110
  op b2 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000018c->00000008 0x050:00003800->00000000
  op b2 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000084->00000100
  op b3 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000017c->000001f8
  op b3 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000074->000000f0
  op b3 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000016c->000001e8
  op b4 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000064->000000e0
  op b4 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000015c->000001d8
  op b4 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000054->000000d0
  op b5 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000014c->000001c8
  op b5 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000044->000000c0
  op b5 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000013c->000001b8
  op b6 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000034->000000b0
  op b6 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000012c->000001a8
  op b6 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000024->000000a0
  op b7 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000011c->00000198
  op b7 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000014->00000090
  op b7 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000010c->00000188
  op b8 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000004->00000080
  op b8 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000fc->00000178
  op b8 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001f4->00000070 0x050:00000000->00000800
  op b9 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ec->00000168
  op b9 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001e4->00000060 0x050:00000800->00001000
  op b9 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000dc->00000158
  op ba reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001d4->00000050 0x050:00001000->00001800
  op ba reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000cc->00000148
  op ba reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001c4->00000040 0x050:00001800->00002000
  op bb reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000bc->00000138
  op bb reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001b4->00000030 0x050:00002000->00002800
  op bb reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ac->00000128
  op bc reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001a4->00000020 0x050:00002800->00003000
  op bc reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000009c->00000118
  op bc reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000194->00000010 0x050:00003000->00003800
  op bd reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000008c->00000108
  op bd reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000184->00000000 0x050:00003800->00000000
  op bd reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000007c->000000f8
  op be reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000174->000001f0
  op be reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000006c->000000e8
  op be reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000164->000001e0
  op bf reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000005c->000000d8
  op bf reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000154->000001d0
  op bf reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000004c->000000c8
  op c0 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000144->000001c0
  op c0 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000003c->000000b8
  op c0 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000134->000001b0
  op c1 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000002c->000000a8
  op c1 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000124->000001a0
  op c1 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000001c->00000098
  op c2 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000114->00000190
  op c2 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000000c->00000088
  op c2 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000104->00000180
  op c3 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001fc->00000078 0x050:00003800->00000000
  op c3 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000f4->00000170
  op c3 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ec->00000068 0x050:00000000->00000800
  op c4 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000e4->00000160
  op c4 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001dc->00000058 0x050:00000800->00001000
  op c4 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000d4->00000150
  op c5 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001cc->00000048 0x050:00001000->00001800
  op c5 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000c4->00000140
  op c5 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001bc->00000038 0x050:00001800->00002000
  op c6 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000b4->00000130
  op c6 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ac->00000028 0x050:00002000->00002800
  op c6 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000a4->00000120
  op c7 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000019c->00000018 0x050:00002800->00003000
  op c7 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000094->00000110
  op c7 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000018c->00000008 0x050:00003000->00003800
  op c8 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000084->00000100
  op c8 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000017c->000001f8
  op c8 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000074->000000f0
  op c9 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000016c->000001e8
  op c9 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000064->000000e0
  op c9 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000015c->000001d8
  op ca reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000054->000000d0
  op ca reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000014c->000001c8
  op ca reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000044->000000c0
  op cb reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000013c->000001b8
  op cb reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000034->000000b0
  op cb reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000012c->000001a8
  op cc reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000024->000000a0
  op cc reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000011c->00000198
  op cc reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000014->00000090
  op cd reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000010c->00000188
  op cd reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000004->00000080
  op cd reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000fc->00000178
  op ce reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001f4->00000070 0x050:00003800->00000000
  op ce reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ec->00000168
  op ce reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001e4->00000060 0x050:00000000->00000800
  op cf reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000dc->00000158
  op cf reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001d4->00000050 0x050:00000800->00001000
  op cf reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000cc->00000148
  op d0 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001c4->00000040 0x050:00001000->00001800
  op d0 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000bc->00000138
  op d0 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001b4->00000030 0x050:00001800->00002000
  op d1 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ac->00000128
  op d1 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001a4->00000020 0x050:00002000->00002800
  op d1 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000009c->00000118
  op d2 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000194->00000010 0x050:00002800->00003000
  op d2 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000008c->00000108
  op d2 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000184->00000000 0x050:00003000->00003800
  op d3 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000007c->000000f8
  op d3 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000174->000001f0
  op d3 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000006c->000000e8
  op d4 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000164->000001e0
  op d4 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000005c->000000d8
  op d4 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000154->000001d0
  op d5 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000004c->000000c8
  op d5 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000144->000001c0
  op d5 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000003c->000000b8
  op d6 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000134->000001b0
  op d6 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000002c->000000a8
  op d6 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000124->000001a0
  op d7 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000001c->00000098
  op d7 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000114->00000190
  op d7 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000000c->00000088
  op d8 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000104->00000180
  op d8 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001fc->00000078 0x050:00003000->00003800
  op d8 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000f4->00000170
  op d9 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ec->00000068 0x050:00003800->00000000
  op d9 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000e4->00000160
  op d9 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001dc->00000058 0x050:00000000->00000800
  op da reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000d4->00000150
  op da reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001cc->00000048 0x050:00000800->00001000
  op da reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000c4->00000140
  op db reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001bc->00000038 0x050:00001000->00001800
  op db reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000b4->00000130
  op db reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ac->00000028 0x050:00001800->00002000
  op dc reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000a4->00000120
  op dc reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000019c->00000018 0x050:00002000->00002800
  op dc reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000094->00000110
  op dd reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000018c->00000008 0x050:00002800->00003000
  op dd reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000084->00000100
  op dd reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000017c->000001f8
  op de reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000074->000000f0
  op de reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000016c->000001e8
  op de reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000064->000000e0
  op df reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000015c->000001d8
  op df reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000054->000000d0
  op df reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000014c->000001c8
  op e0 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000044->000000c0
  op e0 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000013c->000001b8
  op e0 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000034->000000b0
  op e1 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000012c->000001a8
  op e1 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000024->000000a0
  op e1 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000011c->00000198
  op e2 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000014->00000090
  op e2 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000010c->00000188
  op e2 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000004->00000080
  op e3 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000fc->00000178
  op e3 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001f4->00000070 0x050:00003000->00003800
  op e3 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ec->00000168
  op e4 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001e4->00000060 0x050:00003800->00000000
  op e4 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000dc->00000158
  op e4 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001d4->00000050 0x050:00000000->00000800
  op e5 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000cc->00000148
  op e5 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001c4->00000040 0x050:00000800->00001000
  op e5 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000bc->00000138
  op e6 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001b4->00000030 0x050:00001000->00001800
  op e6 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ac->00000128
  op e6 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001a4->00000020 0x050:00001800->00002000
  op e7 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000009c->00000118
  op e7 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000194->00000010 0x050:00002000->00002800
  op e7 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000008c->00000108
  op e8 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000184->00000000 0x050:00002800->00003000
  op e8 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000007c->000000f8
  op e8 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000174->000001f0
  op e9 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000006c->000000e8
  op e9 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000164->000001e0
  op e9 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000005c->000000d8
  op ea reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000154->000001d0
  op ea reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000004c->000000c8
  op ea reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000144->000001c0
  op eb reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000003c->000000b8
  op eb reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 5.2 ms CS_IRQ 05 window; 0x028:00000134->000001b0
  op eb reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000002c->000000a8
  op ec reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000124->000001a0
  op ec reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000001c->00000098
  op ec reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000114->00000190
  op ed reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000000c->00000088
  op ed reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000104->00000180
  op ed reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001fc->00000078 0x050:00002800->00003000
  op ee reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000f4->00000170
  op ee reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ec->00000068 0x050:00003000->00003800
  op ee reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000e4->00000160
  op ef reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001dc->00000058 0x050:00003800->00000000
  op ef reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000d4->00000150
  op ef reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001cc->00000048 0x050:00000000->00000800
  op f0 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000c4->00000140
  op f0 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001bc->00000038 0x050:00000800->00001000
  op f0 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000b4->00000130
  op f1 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001ac->00000028 0x050:00001000->00001800
  op f1 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000a4->00000120
  op f1 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000019c->00000018 0x050:00001800->00002000
  op f2 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000094->00000110
  op f2 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000018c->00000008 0x050:00002000->00002800
  op f2 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000084->00000100
  op f3 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000017c->000001f8
  op f3 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000074->000000f0
  op f3 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000016c->000001e8
  op f4 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000064->000000e0
  op f4 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000015c->000001d8
  op f4 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000054->000000d0
  op f5 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000014c->000001c8
  op f5 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000044->000000c0
  op f5 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000013c->000001b8
  op f6 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000034->000000b0
  op f6 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000012c->000001a8
  op f6 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000024->000000a0
  op f7 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000011c->00000198
  op f7 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000014->00000090
  op f7 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000010c->00000188
  op f8 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000004->00000080
  op f8 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000fc->00000178
  op f8 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001f4->00000070 0x050:00002800->00003000
  op f9 reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ec->00000168
  op f9 reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001e4->00000060 0x050:00003000->00003800
  op f9 reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000dc->00000158
  op fa reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001d4->00000050 0x050:00003800->00000000
  op fa reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000cc->00000148
  op fa reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001c4->00000040 0x050:00000000->00000800
  op fb reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000bc->00000138
  op fb reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001b4->00000030 0x050:00000800->00001000
  op fb reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000000ac->00000128
  op fc reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:000001a4->00000020 0x050:00001000->00001800
  op fc reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000009c->00000118
  op fc reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000194->00000010 0x050:00001800->00002000
  op fd reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000008c->00000108
  op fd reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000184->00000000 0x050:00002000->00002800
  op fd reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000007c->000000f8
  op fe reg 3, arg 1, imm 0          -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000174->000001f0
  op fe reg 3, arg 0xd70, imm 0xd74  -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:0000006c->000000e8
  op fe reg 0, arg 0, imm 0xffffffff -> FAULT continued 4.9 ms CS_IRQ 05 window; 0x028:00000164->000001e0
  op ff reg 3, arg 1, imm 0          -> END continued fast CS_IRQ 01 window; 0x028:0000005c->000000d8
  op ff reg 3, arg 0xd70, imm 0xd74  -> END continued fast CS_IRQ 01 window; 0x028:00000154->000001d0
  op ff reg 0, arg 0, imm 0xffffffff -> END continued fast CS_IRQ 01 window; 0x028:0000004c->000000c8
```

### 14.2.44 optime (run 11)

ID: `D-optime-11`. Device run 11. Experiment: [optime](13-experiments.md#13329-optime).

```
1000 executions each:
  movi (control): 261 us
  op 08 reg 0 arg 0     imm 0        -> END        262 us (+0.00 us per execution vs movi), CS_IRQ 00000001, r6 600d600d
  op 08 reg 7 arg ffff  imm ffffffff -> END        263 us (+0.00 us per execution vs movi), CS_IRQ 00000001, r6 600d600d
  op 08 reg 3 arg d70   imm 1000     -> END        263 us (+0.00 us per execution vs movi), CS_IRQ 00000001, r6 600d600d
  op ff reg 0 arg 0     imm 0        -> END        261 us (+0.00 us per execution vs movi), CS_IRQ 00000001, r6 600d600d
  op ff reg 7 arg ffff  imm ffffffff -> END        262 us (+0.00 us per execution vs movi), CS_IRQ 00000001, r6 600d600d
  op ff reg 3 arg d70   imm 1000     -> END        262 us (+0.00 us per execution vs movi), CS_IRQ 00000001, r6 600d600d
  op 10 reg 0 arg 0     imm 0        -> END        262 us (+0.00 us per execution vs movi), CS_IRQ 00000003, r6 600d600d
  op 10 reg 7 arg ffff  imm ffffffff -> END        262 us (+0.00 us per execution vs movi), CS_IRQ 00000003, r6 600d600d
  op 10 reg 3 arg d70   imm 1000     -> END        262 us (+0.00 us per execution vs movi), CS_IRQ 00000003, r6 600d600d
```

### 14.2.45 ctrlbits (run 11)

ID: `D-ctrlbits-11`. Device run 11. Experiment: [ctrlbits](13-experiments.md#13330-ctrlbits).

```
start words (loop of 1000 x 2 instructions, ~430 us at 0xfff5):
  start 0x0000fff5 -> END        419 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 0000fff4, 36 samples, pcs 0 8 28 30
  start 0x00000001 -> END        408 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000000, 35 samples, pcs 8 28 30
  start 0x00000003 -> TIMEOUT  50030 us, r0    0, r6 00000000, CS_IRQ 00000000, FMSS_CTRL after 00040002, 1 samples, pcs 0
  start 0x00000005 -> END        419 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000004, 36 samples, pcs 0 8 28 30
  start 0x00000009 -> TIMEOUT  50031 us, r0    0, r6 00000000, CS_IRQ 00000000, FMSS_CTRL after 00040000, 1 samples, pcs 0
  start 0x00000011 -> END        419 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000010, 36 samples, pcs 0 8 28 30
  start 0x00000021 -> END        419 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000020, 36 samples, pcs 0 8 28 30
  start 0x00000041 -> END        418 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000040, 36 samples, pcs 0 8 28 30
  start 0x00000081 -> END        418 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000080, 36 samples, pcs 8 28 30
  start 0x00000101 -> END        407 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000100, 35 samples, pcs 8 28 30
  start 0x00000201 -> END        418 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000200, 36 samples, pcs 0 8 28 30
  start 0x00000401 -> END        408 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000400, 35 samples, pcs 8 28 30
  start 0x00000801 -> END        418 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000800, 36 samples, pcs 0 8 28 30
  start 0x00001001 -> END        407 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00001000, 35 samples, pcs 8 28 30
  start 0x00002001 -> END        418 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00002000, 36 samples, pcs 0 8 28 30
  start 0x00004001 -> END        419 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00004000, 36 samples, pcs 8 28 30 38
  start 0x00008001 -> END        419 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00008000, 36 samples, pcs 0 8 28 30
  start 0x00040001 -> END        418 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000000, 36 samples, pcs 8 28 30 40
  start 0x00080001 -> END        419 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000000, 36 samples, pcs 0 8 28 30
  start 0x00100001 -> END        418 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000000, 36 samples, pcs 0 8 28 30
  start 0x00200001 -> END        419 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000000, 36 samples, pcs 0 8 28 30
  start 0x00400001 -> END        419 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000000, 36 samples, pcs 0 8 28 30
  start 0x00800001 -> END        418 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000000, 36 samples, pcs 0 8 28 30
  start 0x01000001 -> END        418 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000000, 36 samples, pcs 0 8 28 30
  start 0x02000001 -> END        418 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000000, 36 samples, pcs 8 28 30 40
  start 0x04000001 -> END        418 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000000, 36 samples, pcs 8 28 30
  start 0x08000001 -> END        408 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000000, 35 samples, pcs 8 28 30
  start 0x10000001 -> END        408 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000000, 35 samples, pcs 8 28 30
  start 0x20000001 -> END        419 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000000, 36 samples, pcs 0 8 28 30
  start 0x40000001 -> END        419 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000000, 36 samples, pcs 0 8 28 30
  start 0x80000001 -> END        419 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000000, 36 samples, pcs 0 8 28 30
  start 0x0000fff7 -> TIMEOUT  50028 us, r0    0, r6 00000000, CS_IRQ 00000000, FMSS_CTRL after 0004fff6, 1 samples, pcs 0
  start 0x0000fffd -> TIMEOUT  50030 us, r0    0, r6 00000000, CS_IRQ 00000000, FMSS_CTRL after 0004fff4, 1 samples, pcs 0
  start 0x0000ffff -> TIMEOUT  50031 us, r0    0, r6 00000000, CS_IRQ 00000000, FMSS_CTRL after 0004fff6, 1 samples, pcs 0
  start 0x0000fff4 -> TIMEOUT  50031 us, r0    0, r6 00000000, CS_IRQ 00000000, FMSS_CTRL after 0000fff4, 1 samples, pcs 0
  start 0x00010001 -> TIMEOUT  50032 us, r0    0, r6 00000000, CS_IRQ 00000000, FMSS_CTRL after 00040000, 1 samples, pcs 0
  start 0x00020001 -> END        418 us, r0    0, r6 600d600d, CS_IRQ 00000001, FMSS_CTRL after 00000000, 36 samples, pcs 0 8 28 30
```

### 14.2.46 fmcmap (run 11)

ID: `D-fmcmap-11`. Device run 11. Experiment: [fmcmap](13-experiments.md#13331-fmcmap).

```
END; non-zero words per 256-byte block:
  0x000..0x0fc: 008:00000070 00c:0005dc00 014:00000001 020:007f167f 028:00000148 02c:00000002 038:00000007 048:000c3000 04c:000000ff 050:00003800
  0x100..0x1fc: 1bc:ffffffff 1c0:ffffffff 1c4:ffffffff 1c8:ffffffff 1cc:00008060 1d0:003010d0 1d4:ffffffff 1d8:ffffffff 1dc:ffffffff 1e0:ffffffff 1e4:ffffffff 1e8:ffffffff 1ec:ffffffff 1f0:ffffffff 1f4:ffffffff 1f8:ffffffff 1fc:ffffffff
  0x200..0x2fc: 208:00000070 20c:0005dc00 214:00000001 220:007f167f 228:000001c8 22c:00000002 238:00000007 248:000c3000 24c:000000ff 250:00003800
  0x300..0x3fc: 3bc:ffffffff 3c0:ffffffff 3c4:ffffffff 3c8:ffffffff 3cc:00008060 3d0:003010d0 3d4:ffffffff 3d8:ffffffff 3dc:ffffffff 3e0:ffffffff 3e4:ffffffff 3e8:ffffffff 3ec:ffffffff 3f0:ffffffff 3f4:ffffffff 3f8:ffffffff 3fc:ffffffff
  0x400..0x4fc: 420:007f167f 438:00000007 448:000c3000 44c:00000001
  0x500..0x5fc: 5bc:ffffffff 5c0:ffffffff 5c4:ffffffff 5c8:ffffffff 5cc:00008060 5d0:003010d0 5d4:ffffffff 5d8:ffffffff 5dc:ffffffff 5e0:ffffffff 5e4:ffffffff 5e8:ffffffff 5ec:ffffffff 5f0:ffffffff 5f4:ffffffff 5f8:ffffffff 5fc:ffffffff
  0x600..0x6fc: 620:007f167f 628:00000080 638:00000007 648:000c3000 64c:00000001
  0x700..0x7fc: 7bc:ffffffff 7c0:ffffffff 7c4:ffffffff 7c8:ffffffff 7cc:00008060 7d0:003010d0 7d4:ffffffff 7d8:ffffffff 7dc:ffffffff 7e0:ffffffff 7e4:ffffffff 7e8:ffffffff 7ec:ffffffff 7f0:ffffffff 7f4:ffffffff 7f8:ffffffff 7fc:ffffffff
  0x800..0x8fc: 800:20060808 814:00e00000 840:00000010
  0x900..0x9fc: 9c0:00000001
  0xa00..0xafc: a00:20060808 a14:00e00000 a40:00000010
  0xb00..0xbfc: bc0:00000001
  0xc00..0xcfc: c00:0004fff4 c04:0801b140 c08:00000005 c14:000000ff c18:08053958 c1c:00000c1c c20:000000f8 c24:000000f8 c38:00000030 c40:18030001 c4c:00000070 c50:ffff1800
  0xd00..0xdfc: d00:a5000d00 d04:a5000d04 d08:a5000d08 d0c:a5000d0c d10:a5000d10 d14:a5000d14 d18:a5000d18 d1c:a5000d1c d20:a5000d20 d24:a5000d24 d28:a5000d28 d2c:a5000d2c d30:a5000d30 d34:a5000d34 d38:a5000d38 d3c:a5000d3c d40:c0de0d40 d44:c0de0d44 d48:c0de0d48 d4c:c0de0d4c d50:c0de0d50 d54:c0de0d54 d58:c0de0d58 d5c:c0de0d5c ... 8 more
  0xe00..0xefc: all zero
  0xf00..0xffc: all zero
```

### 14.2.47 evsweep (run 11)

ID: `D-evsweep-11`. Device run 11. Experiment: [evsweep](13-experiments.md#13332-evsweep).

The experiment changed between runs.

```
wait N after each context (time beyond the same program without the wait; T = timed out):
  idle, CE0 selected           status before 000c3000 | 0:T 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0us 12:T 13:T 14:T 15:T
  after a command byte (70h)   status before 000c3002 | 0:0us 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0us 12:T 13:T 14:T 15:T
  after 70h + status latch     status before 008c3002 | 0:0us 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0us 12:T 13:T 14:T 15:T
  during an erase              status before 000c3000 | 0:T 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0us 12:T 13:T 14:T 15:T
  2 ms after an erase start    status before 000c3000 | 0:T 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0us 12:T 13:T 14:T 15:T
```

### 14.2.48 rwmask (run 12)

ID: `D-rwmask-12`. Device run 12. Experiment: [rwmask](13-experiments.md#13333-rwmask).

```
registers that keep a CPU write or change (others read the same idle value):
  0x000 idle 00000000  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x00c idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x010 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x014 idle 0000a5a5  ones 0000ffff zeros 00000000 a5 0000a5a5 -> writable 0000ffff
  0x018 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x020 idle 00a5a5a5  ones 00ffffff zeros 00000000 a5 00a5a5a5 -> writable 00ffffff
  0x024 idle 000001a5  ones 000001ff zeros 00000000 a5 000001a5 -> writable 000001ff
  0x028 idle 0000009d  ones 000001ff zeros 00000000 a5 000001a5 -> writable 000001ff
  0x02c idle 00000005  ones 00000007 zeros 00000000 a5 00000005 -> writable 00000007
  0x030 idle 000005a5  ones 00001fff zeros 00000000 a5 000005a5 -> writable 00001fff
  0x034 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x044 idle 0000a5a5  ones 0000ffff zeros 00000000 a5 0000a5a5 -> writable 0000ffff
  0x060 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x064 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x068 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x06c idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x070 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x074 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x078 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x804 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x808 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x814 idle 01a5a5a5  ones 01ffffff zeros 00000000 a5 01a5a5a5 -> writable 01ffffff
  0x844 idle 00000025  ones 0000002f zeros 00000000 a5 00000025 -> writable 0000002f
  0xc04 idle 09f92e80  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0xc10 idle 000000a5  ones 000000ff zeros 00000000 a5 000000a5 -> writable 000000ff
  0xc18 idle 00000000  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0xc1c idle 00000000  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0xc20 idle 00000000  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0xc24 idle 00000000  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0xc28 idle 00000000  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0xc2c idle 00000000  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0xc30 idle 00000000  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0xc34 idle 00000000  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0xc3c idle 00000000  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0xc48 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0xc4c idle 00000000  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0xc54 idle 000005a5  ones 00000fff zeros 00000000 a5 000005a5 -> writable 00000fff
  0xc58 idle 00000000  ones 000000ff zeros 00000000 a5 000000a5 -> writable 000000ff
  0xc5c idle 00000000  ones 00000001 zeros 00000000 a5 00000001 -> writable 00000001
  0x400 idle 00000000  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x404 idle 00000105  ones 0000031f zeros 00000306 a5 00000105 -> writable 00000019
  0x408 idle 000000a5  ones 000000ff zeros 00000000 a5 000000a5 -> writable 000000ff
  0x40c idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x410 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x414 idle 0000a5a5  ones 0000ffff zeros 00000000 a5 0000a5a5 -> writable 0000ffff
  0x418 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x420 idle 00a5a5a5  ones 00ffffff zeros 00000000 a5 00a5a5a5 -> writable 00ffffff
  0x424 idle 000001a5  ones 000001ff zeros 00000000 a5 000001a5 -> writable 000001ff
  0x428 idle 000000a5  ones 000001ff zeros 00000000 a5 000001a5 -> writable 000001ff
  0x42c idle 00000005  ones 00000007 zeros 00000000 a5 00000005 -> writable 00000007
  0x430 idle 000005a5  ones 00001fff zeros 00000000 a5 000005a5 -> writable 00001fff
  0x434 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x438 idle 000005a5  ones 00001fff zeros 00000000 a5 000005a5 -> writable 00001fff
  0x444 idle 0000a5a5  ones 0000ffff zeros 00000000 a5 0000a5a5 -> writable 0000ffff
  0x460 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x464 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x468 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x46c idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x470 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x474 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x478 idle a5a5a5a5  ones ffffffff zeros 00000000 a5 a5a5a5a5 -> writable ffffffff
  0x47c idle 000005a4  ones 00000ffc zeros 00000000 a5 000005a4 -> writable 00000ffc
```

### 14.2.49 c50x (run 13)

ID: `D-c50x-13`. Device run 13. Experiment: [c50x](13-experiments.md#13334-c50x).

```
faults and FMSS_C50 b0:
  wait 3, CE0 selected, nothing started    -> END         17 us, C50 in the handler 00000000, C3C 00000000, CS_IRQ 00000001
  ldi from 0x50000000                      -> END         17 us, C50 in the handler 00000000, C3C 00000000, CS_IRQ 00000001
  DMA to 0x50000000                        -> FAULT     4878 us, C50 in the handler ffff1800, C3C 00000070, CS_IRQ 00000015
```

### 14.2.50 evsweep (run 14)

ID: `D-evsweep-14`. Device run 14. Experiment: [evsweep](13-experiments.md#13332-evsweep).

The experiment changed between runs.

```
wait N after each context (time beyond the same program without the wait; T = timed out):
  idle, CE0 selected           status 000c3000 | 0:T 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0us 12:T 13:T 14:T 15:T
  after a command byte (70h)   status 000c3002 | 0:1us 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:1us 12:T 13:T 14:T 15:1us
  after 70h + status latch     status 008c3002 | 0:0us 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0us 12:T 13:T 14:T 15:0us
  during an erase              status 000c3000 | 0:T 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0us 12:T 13:T 14:T 15:T
  2 ms after an erase start    status 000c3000 | 0:T 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:11us 12:T 13:T 14:T 15:T
```

### 14.2.51 op0f (run 14)

ID: `D-op0f-14`. Device run 14. Experiment: [op0f](13-experiments.md#13335-op0f).

```
opcode 0x0f (labels are byte offsets from the program start):
  arg 1 -> r1 = lab1               -> END     lab1         r2..r5,r7 0 0 0 0 0; C3C 00000000 C48 00000000; CS_IRQ 00000001
  reg 1, arg 0 -> r0 = lab2        -> END     lab2         r2..r5,r7 0 0 0 0 0; C3C 00000000 C48 00000000; CS_IRQ 00000001
  arg 9 (r1 if 3 bits)             -> END     lab2         r2..r5,r7 0 0 0 0 0; C3C 00000000 C48 00000000; CS_IRQ 00000001
  imm = lab2, r1 = lab1            -> END     lab1         r2..r5,r7 0 0 0 0 0; C3C 00000000 C48 00000000; CS_IRQ 00000001
  r1 = lab1 + 4 (unaligned)        -> END     nothing      r2..r5,r7 0 0 0 0 0; C3C 00000000 C48 00000000; CS_IRQ 00000001
  r1 = lab1 - 8 (the 0x08 no-op)   -> END     lab1         r2..r5,r7 0 0 0 0 0; C3C 00000000 C48 00000000; CS_IRQ 00000001
```

### 14.2.52 ctrlhold (run 14)

ID: `D-ctrlhold-14`. Device run 14. Experiment: [ctrlhold](13-experiments.md#13336-ctrlhold).

```
start-word bits 1, 3, 16 and 17 (loop of 1000, later writes 2 ms after the start):
  0x3, no later write    -> TIMEOUT  20017 us, r0    0, r6 00000000, PC 0, FMSS_CTRL 00040002, CS_IRQ 00000000, PCs seen 0
  0x3, later 0x1         -> END       2411 us, r0    0, r6 600d600d, PC 0, FMSS_CTRL 00000000, CS_IRQ 00000001, PCs seen 0 0 28 30 30 30 28 30 30 30 30 28
  0x3, later 0xfff5      -> END       2411 us, r0    0, r6 600d600d, PC 0, FMSS_CTRL 0000fff4, CS_IRQ 00000001, PCs seen 0 0 28 28 30 30 28 28 28 28 28 30
  0x9, no later write    -> TIMEOUT  20015 us, r0    0, r6 00000000, PC 0, FMSS_CTRL 00040000, CS_IRQ 00000000, PCs seen 0
  0x9, later 0x9         -> TIMEOUT  20015 us, r0    0, r6 00000000, PC 0, FMSS_CTRL 00040000, CS_IRQ 00000000, PCs seen 0
  0x9, later 0x9 x5      -> TIMEOUT  20015 us, r0    0, r6 00000000, PC 0, FMSS_CTRL 00040000, CS_IRQ 00000000, PCs seen 0
  0x9, later 0x8 x5      -> TIMEOUT  20015 us, r0    0, r6 00000000, PC 0, FMSS_CTRL 00040000, CS_IRQ 00000000, PCs seen 0
  0x9, later 0x1         -> END       2417 us, r0    0, r6 600d600d, PC 0, FMSS_CTRL 00000000, CS_IRQ 00000001, PCs seen 0 0 30 28 28 30 30 30 28 30 30 30
  0x10001, later 0x1     -> END       2411 us, r0    0, r6 600d600d, PC 0, FMSS_CTRL 00000000, CS_IRQ 00000001, PCs seen 0 0 28 28 30 28 28 30 30 30 30 28
  0x3, later 0x2 x5      -> TIMEOUT  20015 us, r0    0, r6 00000000, PC 0, FMSS_CTRL 00040002, CS_IRQ 00000000, PCs seen 0
  END-only, r2..r5 preloaded, start 0xfff5  -> END     r2..r5 22222222 33333333 44444444 55555555
  END-only, r2..r5 preloaded, start 0x20001 -> END     r2..r5 22222222 33333333 44444444 55555555
  END-only, r2..r5 preloaded, start 0x20000 -> TIMEOUT r2..r5 22222222 33333333 44444444 55555555
```

### 14.2.53 evclear (run 14)

ID: `D-evclear-14`. Device run 14. Experiment: [evclear](13-experiments.md#13337-evclear).

```
pending events and FMC_STATUS reads:
  status latch, wait 5                     -> END         17 us (passed), FMC_STATUS read before 00000000, after 008c3000
  status latch, ld FMC_STATUS, wait 5      -> END         17 us (passed), FMC_STATUS read before 008c3000, after 008c3000
  status latch, ld FMC_NSTAT, wait 5       -> END         17 us (passed), FMC_STATUS read before 000000c0, after 008c3000
  status latch, ld FMC_STATUS x2, wait 5   -> END         17 us (passed), FMC_STATUS read before 008c3000, after 008c3000
  idle, wait 3                             -> FAULT     4891 us (timed out), FMC_STATUS read before 00000000, after 000c3000
  idle, ld FMC_STATUS, wait 3              -> FAULT     4870 us (timed out), FMC_STATUS read before 000c3000, after 000c3000
  70h, ld FMC_STATUS, wait 0               -> END         17 us (passed), FMC_STATUS read before 000c3002, after 000c3002
  idle, ld FMC_STATUS, wait 11             -> END         17 us (passed), FMC_STATUS read before 000c3000, after 000c3000
```

### 14.2.54 timeregs (run 14)

ID: `D-timeregs-14`. Device run 14. Experiment: [timeregs](13-experiments.md#13338-timeregs).

```
WAIT 2 (times out) with each word set by the CPU before the start:
  control          -> FAULT     4878 us, first fault at   4862 us, r6 600d600d, after: C10 a5 C3C 00000000 C48 00000000 C54 5a5
  C54 = 0x0        -> FAULT     4864 us, first fault at   4848 us, r6 600d600d, after: C10 a5 C3C 00000000 C48 00000000 C54 000
  C54 = 0x1        -> FAULT     4877 us, first fault at   4862 us, r6 600d600d, after: C10 a5 C3C 00000000 C48 00000000 C54 001
  C54 = 0x10       -> FAULT     4878 us, first fault at   4862 us, r6 600d600d, after: C10 a5 C3C 00000000 C48 00000000 C54 010
  C54 = 0x100      -> FAULT     4870 us, first fault at   4854 us, r6 600d600d, after: C10 a5 C3C 00000000 C48 00000000 C54 100
  C54 = 0x5a5      -> FAULT     4878 us, first fault at   4862 us, r6 600d600d, after: C10 a5 C3C 00000000 C48 00000000 C54 5a5
  C54 = 0xfff      -> FAULT     4862 us, first fault at   4848 us, r6 600d600d, after: C10 a5 C3C 00000000 C48 00000000 C54 fff
  C10 = 0x0        -> FAULT     4878 us, first fault at   4862 us, r6 600d600d, after: C10 00 C3C 00000000 C48 00000000 C54 fff
  C10 = 0x1        -> FAULT     4870 us, first fault at   4854 us, r6 600d600d, after: C10 01 C3C 00000000 C48 00000000 C54 fff
  C10 = 0x80       -> FAULT     4864 us, first fault at   4848 us, r6 600d600d, after: C10 80 C3C 00000000 C48 00000000 C54 fff
  C10 = 0xff       -> FAULT     4891 us, first fault at   4876 us, r6 600d600d, after: C10 ff C3C 00000000 C48 00000000 C54 fff
  C48 = 0x0        -> FAULT     4891 us, first fault at   4876 us, r6 600d600d, after: C10 ff C3C 00000000 C48 00000000 C54 fff
  C48 = 0xffffffff -> FAULT     4884 us, first fault at   4868 us, r6 600d600d, after: C10 ff C3C 00000000 C48 ffffffff C54 fff
  C3C = 0x0        -> FAULT     4877 us, first fault at   4862 us, r6 600d600d, after: C10 ff C3C 00000000 C48 ffffffff C54 fff
  C3C = 0xffffffff -> FAULT     4878 us, first fault at   4862 us, r6 600d600d, after: C10 ff C3C 00000000 C48 ffffffff C54 fff
```

### 14.2.55 ev5rule (run 15)

ID: `D-ev5rule-15`. Device run 15. Experiment: [ev5rule](13-experiments.md#13339-ev5rule).

```
what arms event 5 (the last WAIT 5 of each program is the question):
  70h, wait 0, clear b1, latch (control)   -> last WAIT 5 passed        20 us, FMC_STATUS 008c3000, status byte c0, CS_IRQ 00000031
  70h, wait 0, latch (b1 still set)        -> last WAIT 5 passed        19 us, FMC_STATUS 008c3002, status byte c0, CS_IRQ 00000031
  70h, latch, clear b1                     -> last WAIT 5 passed        19 us, FMC_STATUS 008c3000, status byte c0, CS_IRQ 00000031
  70h, wait 0, latch, clear b1             -> last WAIT 5 passed        20 us, FMC_STATUS 008c3000, status byte c0, CS_IRQ 00000031
  control, then clear b23, latch again     -> last WAIT 5 TIMED OUT   4873 us, FMC_STATUS 000c3000, status byte c0, CS_IRQ 00000035
  control, then latch again (b23 kept)     -> last WAIT 5 passed        20 us, FMC_STATUS 008c3000, status byte c0, CS_IRQ 00000031
  control, then a new 70h cycle            -> last WAIT 5 TIMED OUT   9752 us, FMC_STATUS 040c3000, status byte c0, CS_IRQ 00000035
  control, but 0x90 instead of 70h         -> last WAIT 5 TIMED OUT   4873 us, FMC_STATUS 000c3000, status byte 90, CS_IRQ 00000035
```

### 14.2.56 c08 (run 15)

ID: `D-c08-15`. Device run 15. Experiment: [c08](13-experiments.md#13340-c08).

```
C08 1 ms after the start:
  running (loop)           -> TIMEOUT C08 1, PC 28, FMSS_CTRL 0004fff4, CS_IRQ 00000020, C14 ff
  waiting (wait 2)         -> TIMEOUT C08 4, PC 18, FMSS_CTRL 0004fff4, CS_IRQ 00000020, C14 c0
  stalled (opcode 0x09)    -> TIMEOUT C08 4, PC 18, FMSS_CTRL 0004fff4, CS_IRQ 00000020, C14 c0
  timer (0x15 from 0xff)   -> END     C08 0, PC 0, FMSS_CTRL 0000fff4, CS_IRQ 00000021, C14 ff
  blocked (0x12)           -> TIMEOUT C08 a, PC 18, FMSS_CTRL 0004fff4, CS_IRQ 00000028, C14 ff
  paused (start bit 1)     -> TIMEOUT C08 3, PC 0, FMSS_CTRL 00040002, CS_IRQ 00000020, C14 ff
  held (start bit 3)       -> TIMEOUT C08 0, PC 0, FMSS_CTRL 00040000, CS_IRQ 00000020, C14 ff
```

### 14.2.57 chan1 (run 15)

ID: `D-chan1-15`. Device run 15. Experiment: [chan1](13-experiments.md#13341-chan1).

```
READ ID (90h 00h, 8 bytes) through each FMC register set:
  set 0x000, CE0 (control) -> END        20 us, 0 timeouts, ID b614d5ec 14d5ec74, status 0a041000, CS_IRQ 00000031
  set 0x000, CE1       -> END        19 us, 0 timeouts, ID b614d5ec 14d5ec74, status 08041000, CS_IRQ 00000031
  set 0x000, CE2       -> END        20 us, 0 timeouts, ID b614d5ec 14d5ec74, status 08041000, CS_IRQ 00000031
  set 0x000, CE3       -> END        19 us, 0 timeouts, ID b614d5ec 14d5ec74, status 08041000, CS_IRQ 00000031
  set 0x400, CE0       -> FAULT   14633 us, 3 timeouts, ID 4a01faa1 4a01faa1, status 0e041000, CS_IRQ 00000035
  set 0x400, CE1       -> FAULT   14626 us, 3 timeouts, ID 679a704e 679a704e, status 0e041000, CS_IRQ 00000035
  set 0x400, CE2       -> FAULT   14625 us, 3 timeouts, ID 01010101 01010101, status 0c041000, CS_IRQ 00000035
  set 0x400, CE3       -> FAULT   14624 us, 3 timeouts, ID 01010101 01010101, status 0c041000, CS_IRQ 00000035
  set 0x400, CE4       -> FAULT   14632 us, 3 timeouts, ID 01010101 01010101, status 0c041000, CS_IRQ 00000035
  set 0x400, CE5       -> FAULT   14641 us, 3 timeouts, ID 01010101 01010101, status 0c041000, CS_IRQ 00000035
  set 0x400, CE6       -> FAULT   14625 us, 3 timeouts, ID 01010101 01010101, status 0c041000, CS_IRQ 00000035
  set 0x400, CE7       -> FAULT   14630 us, 3 timeouts, ID 01010101 01010101, status 0c041000, CS_IRQ 00000035
```

### 14.2.58 readev (run 15)

ID: `D-readev-15`. Device run 15. Experiment: [readev](13-experiments.md#13342-readev).

```
page read, wait N after each step (us beyond the program without the wait; T = timed out; the step's own wait is not done yet):
  address sent               20 us, FMC_STATUS 000c3000 ECC_DONE 10 | 0:T 1:0 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:0
  30h sent (tR running)      19 us, FMC_STATUS 000c3002 ECC_DONE 10 | 0:0 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:1
  ready, data mode           55 us, FMC_STATUS 000c3000 ECC_DONE 10 | 0:T 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:0
  parity read started        55 us, FMC_STATUS 00041000 ECC_DONE 10 | 0:T 1:T 2:T 3:0 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:1
  data read started          56 us, FMC_STATUS 00041000 ECC_DONE 10 | 0:T 1:T 2:T 3:49 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:0
  ECC decode started        106 us, FMC_STATUS 0203c000 ECC_DONE 0 | 0:T 1:T 2:T 3:T 4:0 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:0
  DMA started               106 us, FMC_STATUS 0203c000 ECC_DONE 74 | 0:T 1:T 2:T 3:T 4:0 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:0
  all done                 4989 us, FMC_STATUS 0603c000 ECC_DONE 74 | 0:T 1:T 2:T 3:T 4:0 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:0
```

### 14.2.59 regnum (run 15)

ID: `D-regnum-15`. Device run 15. Experiment: [regnum](13-experiments.md#13343-regnum).

```
register numbers above 7 (r0..r7 seeded 0x10101010 * (n + 1)):
  mov r1, r9                     -> END    r6 600d600d; r1=10101010; d70 00000000
  mov r1, r15                    -> END    r6 600d600d; r1=10101010; d70 00000000
  mov r1, r255                   -> END    r6 600d600d; r1=10101010; d70 00000000
  add r1, r9, #1                 -> END    r6 600d600d; r1=10101011; d70 00000000
  st [0xd70], r9                 -> END    r6 600d600d; no register; d70 00000000
  st [0xd70], r14                -> END    r6 600d600d; no register; d70 00000000
  movi r8, #0x5eed0008           -> END    r6 00000000; no register; d70 00000000
  movi r9, #0x5eed0009           -> END    r6 600d600d; no register; d70 00000000
  movi r10, #0x5eed000a          -> END    r6 600d600d; r0=5eed000a r1=5eed000a r2=5eed000a r3=5eed000a r4=5eed000a r5=5eed000a r7=5eed000a; d70 00000000
  movi r11, #0x5eed000b          -> END    r6 600d600d; no register; d70 00000000
  movi r12, #0x5eed000c          -> END    r6 600d600d; no register; d70 00000000
  movi r13, #0x5eed000d          -> END    r6 600d600d; no register; d70 00000000
  movi r14, #0x5eed000e          -> END    r6 600d600d; no register; d70 00000000
  movi r15, #0x5eed000f          -> END    r6 600d600d; no register; d70 00000000
  movi r16, #0x5eed0010          -> END    r6 600d600d; no register; d70 00000000
  movi r24, #0x5eed0018          -> END    r6 600d600d; no register; d70 00000000
  movi r255, #0x5eed00ff         -> END    r6 600d600d; no register; d70 00000000
  ldi r1, [r9] (r0 = scratch)    -> END    r6 600d600d; r1=e59d000c; d70 00000000
  bnz r9 taken? (r0 != 0)        -> END    r6 bad0bad0; no register; d70 00000000
```

### 14.2.60 branch (run 15)

ID: `D-branch-15`. Device run 15. Experiment: [branch](13-experiments.md#13344-branch).

```
branches:
  bnz r1=1 -> lab (control)    -> END    took the branch, CS_IRQ 00000071
  bnz r1=1 -> lab + 4          -> END    neither (r6 0), CS_IRQ 00000071
  bnz r1=1 -> lab - 8          -> END    took the branch, CS_IRQ 00000071
  bnz r1=1, arg 5 -> lab       -> END    took the branch, CS_IRQ 00000071
  bnz r1=0x80000000 -> lab     -> END    took the branch, CS_IRQ 00000071
  bz r1=0, arg 3 -> lab        -> END    took the branch, CS_IRQ 00000071
  bz r1=1 (not taken)          -> END    fell through, CS_IRQ 00000071
```

### 14.2.61 waitf (run 15)

ID: `D-waitf-15`. Device run 15. Experiment: [waitf](13-experiments.md#13345-waitf).

```
WAIT with its other fields set:
  wait 11 (control)                          -> END    passed after    19 us
  wait 3, arg 0x048, imm 0x3000 (bits set)   -> FAULT  TIMED OUT after  4869 us
  wait 3, arg 0x048, imm 0x1 (bit clear)     -> FAULT  TIMED OUT after  4872 us
  wait 3, arg 0xd70, imm ~0 (word set)       -> FAULT  TIMED OUT after  4872 us
  wait 3, imm ~0                             -> FAULT  TIMED OUT after  4888 us
  wait 11, arg 0xd74, imm ~0 (word 0)        -> END    passed after    19 us
  wait 27 (11 + 16)                          -> FAULT  TIMED OUT after  4888 us
  wait 16 (0 + 16)                           -> FAULT  TIMED OUT after  4872 us
  wait 0x8b (11 + 128)                       -> FAULT  TIMED OUT after  4888 us
  wait 255                                   -> FAULT  TIMED OUT after  4888 us
```

### 14.2.62 opcost (run 15)

ID: `D-opcost-15`. Device run 15. Experiment: [opcost](13-experiments.md#13346-opcost).

```
100 iterations of 10 instructions + 2 loop instructions:
  SDRAM movi              -> END      260 us =  217 ns per instruction (  +0 ns vs movi)
  SDRAM mov               -> END      260 us =  217 ns per instruction (  +0 ns vs movi)
  SDRAM add imm           -> END      259 us =  216 ns per instruction (  -1 ns vs movi)
  SDRAM 08 no-op          -> END      264 us =  220 ns per instruction (  +4 ns vs movi)
  SDRAM ldi (RAM)         -> END      430 us =  358 ns per instruction (+170 ns vs movi)
  SDRAM sti (RAM)         -> END      446 us =  372 ns per instruction (+186 ns vs movi)
  SDRAM wr pram           -> END      298 us =  248 ns per instruction ( +38 ns vs movi)
  SDRAM st pram           -> END      299 us =  249 ns per instruction ( +39 ns vs movi)
  SDRAM ld pram           -> END      297 us =  248 ns per instruction ( +37 ns vs movi)
  SDRAM ld FMC_STATUS     -> END      297 us =  248 ns per instruction ( +37 ns vs movi)
  SDRAM wr FMC_ADDR0      -> END      297 us =  248 ns per instruction ( +37 ns vs movi)
  SDRAM pld pram          -> END      297 us =  248 ns per instruction ( +37 ns vs movi)
  SDRAM pst pram          -> END      298 us =  248 ns per instruction ( +38 ns vs movi)
  SDRAM bnz not taken     -> END      259 us =  216 ns per instruction (  -1 ns vs movi)
  SDRAM wait 11 (passes)  -> END      264 us =  220 ns per instruction (  +4 ns vs movi)
  SRAM  movi              -> END      165 us =  138 ns per instruction (  +0 ns vs movi)
  SRAM  mov               -> END      165 us =  138 ns per instruction (  +0 ns vs movi)
  SRAM  add imm           -> END      165 us =  138 ns per instruction (  +0 ns vs movi)
  SRAM  08 no-op          -> END      166 us =  138 ns per instruction (  +1 ns vs movi)
  SRAM  ldi (RAM)         -> END      383 us =  319 ns per instruction (+218 ns vs movi)
  SRAM  sti (RAM)         -> END      285 us =  238 ns per instruction (+120 ns vs movi)
  SRAM  wr pram           -> END      200 us =  167 ns per instruction ( +35 ns vs movi)
  SRAM  st pram           -> END      200 us =  167 ns per instruction ( +35 ns vs movi)
  SRAM  ld pram           -> END      199 us =  166 ns per instruction ( +34 ns vs movi)
  SRAM  ld FMC_STATUS     -> END      200 us =  167 ns per instruction ( +35 ns vs movi)
  SRAM  wr FMC_ADDR0      -> END      199 us =  166 ns per instruction ( +34 ns vs movi)
  SRAM  pld pram          -> END      200 us =  167 ns per instruction ( +35 ns vs movi)
  SRAM  pst pram          -> END      200 us =  167 ns per instruction ( +35 ns vs movi)
  SRAM  bnz not taken     -> END      165 us =  138 ns per instruction (  +0 ns vs movi)
  SRAM  wait 11 (passes)  -> END      166 us =  138 ns per instruction (  +1 ns vs movi)
```

### 14.2.63 op12rel (run 15)

ID: `D-op12rel-15`. Device run 15. Experiment: [op12rel](13-experiments.md#13347-op12rel).

```
op 0x12 and one CPU write while it blocks:
  none (control)             -> blocked   TIMEOUT after  20019 us, CS_IRQ 00000078, C08 a
  C48 = 1                    -> blocked   TIMEOUT after  20026 us, CS_IRQ 00000078, C08 a
  C54 = 1                    -> blocked   TIMEOUT after  20022 us, CS_IRQ 00000078, C08 a
  C3C = 1                    -> blocked   TIMEOUT after  20029 us, CS_IRQ 00000078, C08 a
  C5C = 1                    -> blocked   TIMEOUT after  20029 us, CS_IRQ 00000078, C08 a
  C58 = 0xff                 -> blocked   TIMEOUT after  20027 us, CS_IRQ 00000078, C08 a
  C4C = 0x40                 -> blocked   TIMEOUT after  20021 us, CS_IRQ 00000078, C08 a
  C10 = 0                    -> blocked   TIMEOUT after  20022 us, CS_IRQ 00000078, C08 a
  C10 = 0xff                 -> blocked   TIMEOUT after  20029 us, CS_IRQ 00000078, C08 a
  FMSS_CTRL = 0x1            -> blocked   TIMEOUT after  20028 us, CS_IRQ 00000078, C08 a
  FMSS_CTRL = 0x3            -> blocked   TIMEOUT after  20017 us, CS_IRQ 00000078, C08 3
  FMSS_CTRL = 0x9            -> blocked   TIMEOUT after  20025 us, CS_IRQ 00000078, C08 0
  FMSS_CTRL = 0x4            -> blocked   TIMEOUT after  20029 us, CS_IRQ 00000078, C08 a
  FMSS_CTRL = 0x20001        -> blocked   TIMEOUT after  20030 us, CS_IRQ 00000078, C08 a
  CPU sends 70h (FMC_CMD)    -> blocked   TIMEOUT after  20020 us, CS_IRQ 00000078, C08 a
  FMC_STATUS = ~0            -> blocked   TIMEOUT after  20020 us, CS_IRQ 00000078, C08 a
  pram 0xd00 = 1             -> blocked   TIMEOUT after  20029 us, CS_IRQ 00000068, C08 a
  r0 = 1                     -> blocked   TIMEOUT after  20030 us, CS_IRQ 00000068, C08 a
  C14 = 0                    -> blocked   TIMEOUT after  20020 us, CS_IRQ 00000068, C08 a
```

### 14.2.64 pause (run 15)

ID: `D-pause-15`. Device run 15. Experiment: [pause](13-experiments.md#13348-pause).

```
a 20 000-iteration loop with start-word writes while it runs:
  no writes (control, ~8.6 ms)     -> END       8222 us, r0     0, C08 0, FMSS_CTRL 0000fff4, r0 sampled 19996 17516 15023 12542 10063 7579 4996 2513 36
  0xfff7 at 1 ms                   -> TIMEOUT  30016 us, r0 16539, C08 3, FMSS_CTRL 0004fff6, r0 sampled 19995 19702 19408 19115 18820 18525 18234 17945
  0xfff7 at 1 ms, 0xfff5 at 5 ms   -> END      12207 us, r0     0, C08 0, FMSS_CTRL 0000fff4, r0 sampled 19995 17558 14957 12463 9966 7461 4956 2461
  0xfffd at 1 ms, 0xfff5 at 5 ms   -> END      13199 us, r0     0, C08 0, FMSS_CTRL 0000fff4, r0 sampled 19995 19796 16928 14148 11381 8611 5843 3069 144
  0x10001 at 1 ms, 0x1 at 5 ms     -> END      13207 us, r0     0, C08 0, FMSS_CTRL 00000000, r0 sampled 19994 19762 16863 14051 11248 8443 5642 2837 27
  0xfff5 at 1 ms (restart?)        -> END       8235 us, r0     0, C08 0, FMSS_CTRL 0000fff4, r0 sampled 19995 17553 15041 12541 10049 7549 4950 2455
  0xfff4 at 1 ms                   -> END       8220 us, r0     0, C08 0, FMSS_CTRL 0000fff4, r0 sampled 19995 17588 15108 12642 10178 7709 5144 2678 216
```

### 14.2.65 fmc28 (run 15)

ID: `D-fmc28-15`. Device run 15. Experiment: [fmc28](13-experiments.md#13349-fmc28).

```
FMC_28 across a run (before = after the CPU set-up):
  empty          -> FMC_28 19c -> 018: +124 (+0 vs empty)
  movi           -> FMC_28 094 -> 110: +124 (+0 vs empty)
  pld pram       -> FMC_28 18c -> 008: +124 (+0 vs empty)
  pst pram       -> FMC_28 084 -> 100: +124 (+0 vs empty)
  sti RAM        -> FMC_28 17c -> 1f8: +124 (+0 vs empty)
  ldi RAM        -> FMC_28 074 -> 0f0: +124 (+0 vs empty)
  wr pram        -> FMC_28 16c -> 1e8: +124 (+0 vs empty)
  ld pram        -> FMC_28 064 -> 0e0: +124 (+0 vs empty)
  ld FMC_STATUS  -> FMC_28 15c -> 1d8: +124 (+0 vs empty)
  wr FMC_ADDR0   -> FMC_28 054 -> 0d0: +124 (+0 vs empty)
  ld FMC_28      -> FMC_28 14c -> 1c8: +124 (+0 vs empty)
```

### 14.2.66 blk100 (run 15)

ID: `D-blk100-15`. Device run 15. Experiment: [blk100](13-experiments.md#13350-blk100).

```
END; words that are not 0 in any read:
  0x1b8: value 00000000, after ~0 00000001 (0x3b8: 00000001), after 0 00000000  writable 00000001
  0x1bc: value ffffffff, after ~0 ffffffff (0x3bc: ffffffff), after 0 00000000  writable ffffffff
  0x1c0: value ffffffff, after ~0 ffffffff (0x3c0: ffffffff), after 0 00000000  writable ffffffff
  0x1c4: value ffffffff, after ~0 ffffffff (0x3c4: ffffffff), after 0 00000000  writable ffffffff
  0x1c8: value ffffffff, after ~0 ffffffff (0x3c8: ffffffff), after 0 00000000  writable ffffffff
  0x1cc: value 00008060, after ~0 00ffffff (0x3cc: 00ffffff), after 0 00000000  writable 00ffffff
  0x1d0: value 003010d0, after ~0 00ffffff (0x3d0: 00ffffff), after 0 00000000  writable 00ffffff
  0x1d4: value ffffffff, after ~0 ffffffff (0x3d4: ffffffff), after 0 00000000  writable ffffffff
  0x1d8: value ffffffff, after ~0 ffffffff (0x3d8: ffffffff), after 0 00000000  writable ffffffff
```

### 14.2.67 irqline (run 15)

ID: `D-irqline-15`. Device run 15. Experiment: [irqline](13-experiments.md#13351-irqline).

```
VIC1 raw bit 22 while polling and at the end:
  END, start 0xfff5                    -> END     CS_IRQ 00000061, line seen 1, line at the end 1, preflight line -
  END, start 0xffb5 (no bit 6)         -> END     CS_IRQ 00000061, line seen 1, line at the end 1, preflight line -
  END, start 0x41                      -> END     CS_IRQ 00000061, line seen 1, line at the end 1, preflight line -
  END, start 0x1                       -> END     CS_IRQ 00000061, line seen 1, line at the end 1, preflight line -
  0x10 then block, 0xfff5              -> TIMEOUT CS_IRQ 0000006a, line seen 1, line at the end 1, preflight line -
  0x10 then block, 0xffb5              -> TIMEOUT CS_IRQ 0000006a, line seen 1, line at the end 1, preflight line -
  block only, 0xfff5                   -> TIMEOUT CS_IRQ 00000068, line seen 1, line at the end 1, preflight line -
  timeout (C58 0) then block, 0xfff5   -> FAULT   CS_IRQ 0000006c, line seen 1, line at the end 1, preflight line -
  timeout (C58 0) then block, 0xffb5   -> FAULT   CS_IRQ 0000006c, line seen 1, line at the end 1, preflight line -
```

### 14.2.68 restore (run 16)

ID: `D-restore-16`. Device run 16. Experiment: [restore](13-experiments.md#13352-restore).

The experiment changed between runs.

```
restored 45 words; not as written afterwards: 0x428=00000180
  changed by the restore: 0x00c:00000000->0005dc00 0x010:00000005->00000000 0x014:00000100->00000001 0x018:a5a5a5a5->00000000 0x020:00a5a5a5->007f167f 0x02c:00000004->00000002 0x030:000001ff->00000000 0x034:08053e40->00000000 0x03c:08053e40->00000000 0x040:08053e40->00000000 0x044:0000a5a5->00000000 0x050:00003901->00000901 0x054:00000004->00000000 0x058:00000004->00000000 0x05c:00000004->00000000 0x060:a5a5a5a5->00000000 0x064:a5a5a5a5->00000000 0x068:a5a5a5a5->00000000 0x06c:a5a5a5a5->00000000 0x070:a5a5a5a5->00000000 0x074:a5a5a5a5->00000000 0x078:a5a5a5a5->00000000 0x404:00000300->00000200 0x410:a5a5a5a5->00000000 0x414:00000100->00000000 0x418:a5a5a5a5->00000000 0x420:00a5a5a5->00000000 0x428:00000108->00000180 0x430:00000007->00000000 0x434:a5a5a5a5->00000000 0x438:00000005->00000000 0x43c:a5a5a5a5->00000000 0x440:a5a5a5a5->00000000 0x444:0000a5a5->00000000 0x448:0e041000->0e1c3000 0x450:00000000->00000800 0x454:00000004->00000000 0x458:00000004->00000000 0x45c:00000004->00000000 0x478:a5a5a5a5->00000000 0x480:ffffffff->00000000 0x484:ffffffff->00000000 0x488:ffffffff->00000000 0x48c:ffffffff->00000000 0x490:ffffffff->00000000 0x494:ffffffff->00000000 0x498:ffffffff->00000000 0x49c:ffffffff->00000000 0x4a0:ffffffff->00000000 0x4a4:ffffffff->00000000 0x4a8:ffffffff->00000000 0x4ac:ffffffff->00000000 0x4b0:ffffffff->00000000 0x4b4:ffffffff->00000000 0x4b8:ffffffff->00000000 0x4bc:ffffffff->00000000 0x4c0:ffffffff->00000000 0x4c4:ffffffff->00000000 0x4c8:ffffffff->00000000 0x4cc:ffffffff->00000000 0x4d0:ffffffff->00000000 0x4d4:ffffffff->00000000 0x4d8:ffffffff->00000000 0x4dc:ffffffff->00000000 0x4e0:ffffffff->00000000 0x4e4:ffffffff->00000000 0x4e8:ffffffff->00000000 0x4ec:ffffffff->00000000 0x4f0:ffffffff->00000000 0x4f4:ffffffff->00000000 0x4f8:ffffffff->00000000 0x4fc:ffffffff->00000000 0x804:a5a5a5a5->00000000 0x808:a5a5a5a5->00000000 0x814:00000180->00e00000 0x840:00000074->00000010 0x844:00000025->00000000 0xc0c:00000061->00000001 0xc10:000000ff->00000000 0xc48:00000001->00000000 0xc50:ffff1810->ffff1900 0xc54:00000001->00000000
  CS_IRQ after an END-only run 00000061 -> 00000001, interrupt line at the end 1 -> 0
```

### 14.2.69 readev (run 16)

ID: `D-readev-16`. Device run 16. Experiment: [readev](13-experiments.md#13342-readev).

```
page read, wait N after each step (us beyond the program without the wait; T = timed out; the step's own wait is not done yet):
  address sent               20 us, FMC_STATUS 000c3000 ECC_DONE 10 | 0:T 1:0 2:T 3:T 4:T 5:T 6:T 7:T 8:0 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  30h sent (tR running)      20 us, FMC_STATUS 000c3002 ECC_DONE 10 | 0:0 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:0 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  ready, data mode           56 us, FMC_STATUS 000c3000 ECC_DONE 10 | 0:T 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:0 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  parity read started        55 us, FMC_STATUS 00041000 ECC_DONE 10 | 0:T 1:T 2:T 3:0 4:T 5:T 6:T 7:T 8:0 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  data read started          56 us, FMC_STATUS 00041000 ECC_DONE 10 | 0:T 1:T 2:T 3:49 4:T 5:T 6:T 7:T 8:0 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  ECC decode started        104 us, FMC_STATUS 0003c000 ECC_DONE 0 | 0:T 1:T 2:T 3:T 4:1 5:T 6:T 7:T 8:1 9:T 10:T 11:1 12:T 13:T 14:T 15:T
  DMA started               105 us, FMC_STATUS 0003c000 ECC_DONE 74 | 0:T 1:T 2:T 3:T 4:0 5:T 6:T 7:T 8:0 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  all done                 4963 us, FMC_STATUS 0003c000 ECC_DONE 74 | 0:T 1:T 2:T 3:T 4:15 5:T 6:T 7:T 8:9 9:T 10:T 11:0 12:T 13:T 14:T 15:T
```

### 14.2.70 irqline (run 16)

ID: `D-irqline-16`. Device run 16. Experiment: [irqline](13-experiments.md#13351-irqline).

```
VIC1 raw bit 22 while polling and at the end:
  END, start 0xfff5                    -> END     CS_IRQ 00000001, line seen 0, line at the end 0, preflight line -
  END, start 0xffb5 (no bit 6)         -> END     CS_IRQ 00000001, line seen 0, line at the end 0, preflight line -
  END, start 0x41                      -> END     CS_IRQ 00000001, line seen 0, line at the end 0, preflight line -
  END, start 0x1                       -> END     CS_IRQ 00000001, line seen 0, line at the end 0, preflight line -
  0x10 then block, 0xfff5              -> TIMEOUT CS_IRQ 0000000a, line seen 0, line at the end 0, preflight line -
  0x10 then block, 0xffb5              -> TIMEOUT CS_IRQ 0000000a, line seen 0, line at the end 0, preflight line -
  block only, 0xfff5                   -> TIMEOUT CS_IRQ 00000008, line seen 0, line at the end 0, preflight line -
  timeout (C58 0) then block, 0xfff5   -> FAULT   CS_IRQ 0000000c, line seen 0, line at the end 0, preflight line -
  timeout (C58 0) then block, 0xffb5   -> FAULT   CS_IRQ 0000000c, line seen 0, line at the end 0, preflight line -
```

### 14.2.71 irqen (run 16)

ID: `D-irqen-16`. Device run 16. Experiment: [irqen](13-experiments.md#13353-irqen).

```
one enable bit at a time (then 0), after a 70h command (0x044) or an ECC decode (0x844):
  0x044 = 1      -> END    CS_IRQ in program 00000010, after 00000011, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 2      -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 4      -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 8      -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 10     -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 20     -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 40     -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 80     -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 100    -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 200    -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 400    -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 800    -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 1000   -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 2000   -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 4000   -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 8000   -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 0      -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x844 = 1      -> END    CS_IRQ in program 00000040, after 00000041, line 0/0, FMC_STATUS 000c3000, ECC_DONE 75, 0x054 0
  0x844 = 2      -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3000, ECC_DONE 75, 0x054 0
  0x844 = 4      -> END    CS_IRQ in program 00000040, after 00000041, line 0/0, FMC_STATUS 000c3000, ECC_DONE 75, 0x054 0
  0x844 = 8      -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3000, ECC_DONE 75, 0x054 0
  0x844 = 10     -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3000, ECC_DONE 75, 0x054 0
  0x844 = 20     -> END    CS_IRQ in program 00000040, after 00000041, line 0/0, FMC_STATUS 000c3000, ECC_DONE 75, 0x054 0
  0x844 = 0      -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3000, ECC_DONE 75, 0x054 0
```

### 14.2.72 ev5t (run 16)

ID: `D-ev5t-16`. Device run 16. Experiment: [ev5t](13-experiments.md#13354-ev5t).

```
70h + status latch and the last WAIT 5:
  70h, latch, wait 5             -> 0 timeout(s),     19 us, FMC_STATUS after 008c3002
  70h, latch, clear b1, wait 5   -> 0 timeout(s),     20 us, FMC_STATUS after 008c3000
  70h, latch, wait 0, wait 5     -> 0 timeout(s),     20 us, FMC_STATUS after 008c3002
  70h, latch, 20 movi, wait 5    -> 0 timeout(s),     19 us, FMC_STATUS after 008c3002
  70h, latch, wait 15, wait 5    -> 1 timeout(s),   4880 us, FMC_STATUS after 008c3002
  70h, 20 movi, latch, wait 5    -> 0 timeout(s),     19 us, FMC_STATUS after 008c3002
  70h, wait 0, latch, wait 5     -> 0 timeout(s),     19 us, FMC_STATUS after 008c3002
```

### 14.2.73 r8 (run 16)

ID: `D-r8-16`. Device run 16. Experiment: [r8](13-experiments.md#13355-r8).

```
register numbers 8 / 10 (r0..r7 seeded 0x10101010 * (n + 1)):
  movi r8, #lab                  -> END     LANDED ON lab; r 10101010 20202020 30303030 40404040 50505050 60606060 11111111 80808080; PC 0
  movi r8, #0x5eed0008 (repeat)  -> END     ended without either; r 10101010 20202020 30303030 40404040 50505050 60606060 00000000 80808080; PC 0
  mov r8, r1 (r1 = lab)          -> END     LANDED ON lab; r 10101010 00000078 30303030 40404040 50505050 60606060 11111111 80808080; PC 0
  mov r1, r8 (read r8)           -> END     fell through; r 10101010 00000058 30303030 40404040 50505050 60606060 bad0bad0 80808080; PC 0
  movi r10, #0x12345678 (repeat) -> END     fell through; r 12345678 12345678 12345678 12345678 12345678 12345678 bad0bad0 12345678; PC 0
  mov r10, r2                    -> END     fell through; r 30303030 30303030 30303030 30303030 30303030 30303030 bad0bad0 30303030; PC 0
  add r10, r2, #1                -> END     fell through; r 30303031 30303031 30303031 30303031 30303031 30303031 bad0bad0 30303031; PC 0
  movi r26 (10 + 16)             -> END     fell through; r 10101010 20202020 30303030 40404040 50505050 60606060 bad0bad0 80808080; PC 0
  movi r24 (8 + 16)              -> END     fell through; r 10101010 20202020 30303030 40404040 50505050 60606060 bad0bad0 80808080; PC 0
  ldi r1, [r8]                   -> END     fell through; r 10101010 e321f0d3 30303030 40404040 50505050 60606060 bad0bad0 80808080; PC 0
  ldi r1, [r10]                  -> END     fell through; r 10101010 e3520000 30303030 40404040 50505050 60606060 bad0bad0 80808080; PC 0
```

### 14.2.74 state (run 17)

ID: `D-state-17`. Device run 17. Experiment: [state](13-experiments.md#13356-state).

```
CS_IRQ after an END-only run 00000001, interrupt line 0/0; FMC 0x044 00000000, ECC 0x844 00000000, ECC_DONE 00000010, C10 00, 0x404 00000000
```

### 14.2.75 readev (run 17)

ID: `D-readev-17`. Device run 17. Experiment: [readev](13-experiments.md#13342-readev).

```
page read, wait N after each step (us beyond the program without the wait; T = timed out; the step's own wait is not done yet):
  address sent               20 us, FMC_STATUS 000c3000 ECC_DONE 10 | 0:T 1:0 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  30h sent (tR running)      20 us, FMC_STATUS 000c3002 ECC_DONE 10 | 0:0 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  ready, data mode           55 us, FMC_STATUS 000c3000 ECC_DONE 10 | 0:T 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  parity read started        55 us, FMC_STATUS 00041000 ECC_DONE 10 | 0:T 1:T 2:T 3:0 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  data read started          55 us, FMC_STATUS 00041000 ECC_DONE 10 | 0:T 1:T 2:T 3:49 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  ECC decode started        105 us, FMC_STATUS 0003c000 ECC_DONE 0 | 0:T 1:T 2:T 3:T 4:0 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  DMA started               105 us, FMC_STATUS 0003c000 ECC_DONE 74 | 0:T 1:T 2:T 3:T 4:0 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  all done                 4978 us, FMC_STATUS 0003c000 ECC_DONE 74 | 0:T 1:T 2:T 3:T 4:0 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:T
```

### 14.2.76 irqline (run 17)

ID: `D-irqline-17`. Device run 17. Experiment: [irqline](13-experiments.md#13351-irqline).

```
VIC1 raw bit 22 while polling and at the end:
  END, start 0xfff5                    -> END     CS_IRQ 00000001, line seen 0, line at the end 0, preflight line -
  END, start 0xffb5 (no bit 6)         -> END     CS_IRQ 00000001, line seen 0, line at the end 0, preflight line -
  END, start 0x41                      -> END     CS_IRQ 00000001, line seen 0, line at the end 0, preflight line -
  END, start 0x1                       -> END     CS_IRQ 00000001, line seen 0, line at the end 0, preflight line -
  0x10 then block, 0xfff5              -> TIMEOUT CS_IRQ 0000000a, line seen 0, line at the end 0, preflight line -
  0x10 then block, 0xffb5              -> TIMEOUT CS_IRQ 0000000a, line seen 0, line at the end 0, preflight line -
  block only, 0xfff5                   -> TIMEOUT CS_IRQ 00000008, line seen 0, line at the end 0, preflight line -
  timeout (C58 0) then block, 0xfff5   -> FAULT   CS_IRQ 0000000c, line seen 0, line at the end 0, preflight line -
  timeout (C58 0) then block, 0xffb5   -> FAULT   CS_IRQ 0000000c, line seen 0, line at the end 0, preflight line -
```

### 14.2.77 irqen (run 17)

ID: `D-irqen-17`. Device run 17. Experiment: [irqen](13-experiments.md#13353-irqen).

```
one enable bit at a time (then 0), after a 70h command (0x044) or an ECC decode (0x844):
  0x044 = 1      -> END    CS_IRQ in program 00000010, after 00000011, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 2      -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 4      -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 8      -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 10     -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 20     -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 40     -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 80     -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 100    -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 200    -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 400    -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 800    -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 1000   -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 2000   -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 4000   -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 8000   -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x044 = 0      -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3002, ECC_DONE 74, 0x054 0
  0x844 = 1      -> END    CS_IRQ in program 00000040, after 00000041, line 0/0, FMC_STATUS 000c3000, ECC_DONE 75, 0x054 0
  0x844 = 2      -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3000, ECC_DONE 75, 0x054 0
  0x844 = 4      -> END    CS_IRQ in program 00000040, after 00000041, line 0/0, FMC_STATUS 000c3000, ECC_DONE 75, 0x054 0
  0x844 = 8      -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3000, ECC_DONE 75, 0x054 0
  0x844 = 10     -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3000, ECC_DONE 75, 0x054 0
  0x844 = 20     -> END    CS_IRQ in program 00000040, after 00000041, line 0/0, FMC_STATUS 000c3000, ECC_DONE 75, 0x054 0
  0x844 = 0      -> END    CS_IRQ in program 00000000, after 00000001, line 0/0, FMC_STATUS 000c3000, ECC_DONE 75, 0x054 0
```

### 14.2.78 ev5t (run 17)

ID: `D-ev5t-17`. Device run 17. Experiment: [ev5t](13-experiments.md#13354-ev5t).

```
70h + status latch and the last WAIT 5:
  70h, latch, wait 5             -> 0 timeout(s),     19 us, FMC_STATUS after 008c3002
  70h, latch, clear b1, wait 5   -> 0 timeout(s),     20 us, FMC_STATUS after 008c3000
  70h, latch, wait 0, wait 5     -> 0 timeout(s),     19 us, FMC_STATUS after 008c3002
  70h, latch, 20 movi, wait 5    -> 0 timeout(s),     19 us, FMC_STATUS after 008c3002
  70h, latch, wait 15, wait 5    -> 1 timeout(s),   4880 us, FMC_STATUS after 008c3002
  70h, 20 movi, latch, wait 5    -> 0 timeout(s),     19 us, FMC_STATUS after 008c3002
  70h, wait 0, latch, wait 5     -> 0 timeout(s),     19 us, FMC_STATUS after 008c3002
```

### 14.2.79 r8 (run 17)

ID: `D-r8-17`. Device run 17. Experiment: [r8](13-experiments.md#13355-r8).

```
register numbers 8 / 10 (r0..r7 seeded 0x10101010 * (n + 1)):
  movi r8, #lab                  -> END     LANDED ON lab; r 10101010 20202020 30303030 40404040 50505050 60606060 11111111 80808080; PC 0
  movi r8, #0x5eed0008 (repeat)  -> END     ended without either; r 10101010 20202020 30303030 40404040 50505050 60606060 00000000 80808080; PC 0
  mov r8, r1 (r1 = lab)          -> END     LANDED ON lab; r 10101010 00000078 30303030 40404040 50505050 60606060 11111111 80808080; PC 0
  mov r1, r8 (read r8)           -> END     fell through; r 10101010 00000058 30303030 40404040 50505050 60606060 bad0bad0 80808080; PC 0
  movi r10, #0x12345678 (repeat) -> END     fell through; r 12345678 12345678 12345678 12345678 12345678 12345678 bad0bad0 12345678; PC 0
  mov r10, r2                    -> END     fell through; r 30303030 30303030 30303030 30303030 30303030 30303030 bad0bad0 30303030; PC 0
  add r10, r2, #1                -> END     fell through; r 30303031 30303031 30303031 30303031 30303031 30303031 bad0bad0 30303031; PC 0
  movi r26 (10 + 16)             -> END     fell through; r 10101010 20202020 30303030 40404040 50505050 60606060 bad0bad0 80808080; PC 0
  movi r24 (8 + 16)              -> END     fell through; r 10101010 20202020 30303030 40404040 50505050 60606060 bad0bad0 80808080; PC 0
  ldi r1, [r8]                   -> END     fell through; r 10101010 e321f0d3 30303030 40404040 50505050 60606060 bad0bad0 80808080; PC 0
  ldi r1, [r10]                  -> END     fell through; r 10101010 e59d000c 30303030 40404040 50505050 60606060 bad0bad0 80808080; PC 0
```

### 14.2.80 state (run 18)

ID: `D-state-18`. Device run 18. Experiment: [state](13-experiments.md#13356-state).

```
events passing in an idle program: 0 4 5 11 (power-on: 11); 17 words differ from the recorded power-on values
  0x008 = 00000070
  0x00c = dc000000
  0x010 = 00000005
  0x014 = 00000001
  0x02c = 00000004
  0x030 = 000001ff
  0x034 = 08053e40
  0x03c = 08053e40
  0x040 = 08053e40
  0x048 = 008c3002
  0x050 = 00002800
  0x450 = 00000800
  0x80c = 00001100
  0x814 = 00000180
  0x840 = 00000075
  0x9a4 = 0000003a
  0xc50 = ffff1831
  CS_IRQ after an END-only run 00000001, interrupt line 0/0; FMC 0x044 00000000, ECC 0x844 00000000, ECC_DONE 00000075, C10 00, 0x404 00000000
```

### 14.2.81 ev8 (run 18)

ID: `D-ev8-18`. Device run 18. Experiment: [ev8](13-experiments.md#13357-ev8).

```
events after each step through the second register set (ok = passed, T = timed out):
  set 0x400: 90h             0x448 000c3002 0x450 00000000 0x050 00003000 | 0:ok 1:T 2:T 3:T 4:ok 5:ok 6:ok 7:T 8:T 9:T 10:T 11:ok 12:T 13:T 14:T 15:T
  set 0x400: + address       0x448 000c3006 0x450 00000000 0x050 00003800 | 0:ok 1:T 2:T 3:T 4:ok 5:ok 6:ok 7:ok 8:T 9:T 10:T 11:ok 12:T 13:T 14:T 15:T
  set 0x400: + ID transfer   0x448 08041006 0x450 00000000 0x050 00003800 | 0:ok 1:T 2:T 3:T 4:ok 5:ok 6:ok 7:ok 8:T 9:ok 10:T 11:ok 12:T 13:T 14:T 15:T
  next program, no NAND step 0:ok 1:T 2:T 3:T 4:ok 5:ok 6:T 7:T 8:T 9:T 10:T 11:ok 12:T 13:T 14:T 15:T
```

### 14.2.82 evcons (run 18)

ID: `D-evcons-18`. Device run 18. Experiment: [evcons](13-experiments.md#13358-evcons).

```
which wait of each program passed (ok) or timed out (T):
  70h, latch, wait 15                    -> wait 15:T  (4879 us, FMC_STATUS 008c3002)
  70h, latch, wait 15, wait 5            -> wait 15:T wait 5:ok  (4880 us, FMC_STATUS 008c3002)
  70h, latch, wait 5, wait 15            -> wait 5:ok wait 15:T  (4876 us, FMC_STATUS 008c3002)
  70h, latch, wait 5, wait 5             -> wait 5:ok wait 5:ok  (20 us, FMC_STATUS 008c3002)
  70h, wait 0, wait 0                    -> wait 0:ok wait 0:ok  (19 us, FMC_STATUS 000c3002)
  70h, wait 15, wait 0                   -> wait 15:T wait 0:ok  (4880 us, FMC_STATUS 000c3002)
  70h, wait 0, wait 15                   -> wait 0:ok wait 15:T  (4879 us, FMC_STATUS 000c3002)
  70h, latch, wait 15, wait 15, wait 15  -> wait 15:T wait 15:T wait 15:T  (14625 us, FMC_STATUS 008c3002)
```

### 14.2.83 restore (run 19)

ID: `D-restore-19`. Device run 19. Experiment: [restore](13-experiments.md#13352-restore).

The experiment changed between runs.

```
differ from power-on before: 58 words, after: 7 words
  0x008 = 00000070 (power-on 00000000)
  0x050 = 00001000 (power-on 00000000)
  0x404 = 00000200 (power-on 00000000)
  0x408 = 00000090 (power-on 00000000)
  0x450 = 00000800 (power-on 00000000)
  0x80c = 00001100 (power-on 00000000)
  0x9a4 = 0000003a (power-on 00000000)
  CS_IRQ after an END-only run 00000001 -> 00000001, interrupt line 0 -> 0
  events passing in an idle program afterwards: 11 (power-on: 11)
```

### 14.2.84 dmaloop (run 20)

ID: `D-dmaloop-20`. Device run 20. Experiment: [dmaloop](13-experiments.md#13359-dmaloop).

The experiment changed between runs.

```
RAM -> buffer -> RAM through each register set (ok = the wait passed, T = timed out):
  set 0x000, drain 0x100                     -> fill T, drain T; destination 2/512 bytes match (untouched); status after fill 000c3000, after drain 000c3000; DMA_ADDR 08055e40 / 00000000
  set 0x000, DMA_CTRL 7, drain 0x1e0         -> fill T, drain T; destination 2/512 bytes match (untouched); status after fill 000c3000, after drain 000c3000; DMA_ADDR 08055e40 / 00000000
  set 0x400, drain 0x100, wait 8             -> fill T, drain T; destination 2/512 bytes match (untouched); status after fill 000c3000, after drain 000c3000; DMA_ADDR 08055e40 / 08055e40
  set 0x400, drain 0x100, wait 2             -> fill T, drain T; destination 2/512 bytes match (untouched); status after fill 000c3000, after drain 000c3000; DMA_ADDR 08055e40 / 08055e40
  set 0x400, DMA_CTRL 7, drain 0x1e0, wait 8 -> fill T, drain T; destination 2/512 bytes match (untouched); status after fill 000c3000, after drain 000c3000; DMA_ADDR 08055e40 / 08055e40
```

### 14.2.85 restore (run 21)

ID: `D-restore-21`. Device run 21. Experiment: [restore](13-experiments.md#13352-restore).

The experiment changed between runs.

```
differ from power-on before: 56 words, after: 6 words
  0x008 = 00000070 (power-on 00000000)
  0x050 = 00002000 (power-on 00000000)
  0x408 = 00000090 (power-on 00000000)
  0x450 = 00000800 (power-on 00000000)
  0x80c = 00001100 (power-on 00000000)
  0x9a4 = 0000003a (power-on 00000000)
  CS_IRQ after an END-only run 00000001 -> 00000001, interrupt line 0 -> 0
  events passing in an idle program afterwards: 11 (power-on: 11)
```

### 14.2.86 readev (run 21)

ID: `D-readev-21`. Device run 21. Experiment: [readev](13-experiments.md#13342-readev).

```
page read, wait N after each step (us beyond the program without the wait; T = timed out; the step's own wait is not done yet):
  address sent               20 us, FMC_STATUS 000c3000 ECC_DONE 10 | 0:T 1:0 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  30h sent (tR running)      19 us, FMC_STATUS 000c3002 ECC_DONE 10 | 0:1 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  ready, data mode           55 us, FMC_STATUS 000c3000 ECC_DONE 10 | 0:T 1:T 2:T 3:T 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  parity read started        55 us, FMC_STATUS 00041000 ECC_DONE 10 | 0:T 1:T 2:T 3:0 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:1 12:T 13:T 14:T 15:T
  data read started          55 us, FMC_STATUS 00041000 ECC_DONE 10 | 0:T 1:T 2:T 3:49 4:T 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  ECC decode started        104 us, FMC_STATUS 0003c000 ECC_DONE 0 | 0:T 1:T 2:T 3:T 4:0 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  DMA started               109 us, FMC_STATUS 0003c000 ECC_DONE 74 | 0:T 1:T 2:14 3:T 4:0 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:T
  all done                  123 us, FMC_STATUS 0003c000 ECC_DONE 74 | 0:T 1:T 2:T 3:T 4:0 5:T 6:T 7:T 8:T 9:T 10:T 11:0 12:T 13:T 14:T 15:T
```

### 14.2.87 dmaloop (run 21)

ID: `D-dmaloop-21`. Device run 21. Experiment: [dmaloop](13-experiments.md#13359-dmaloop).

The experiment changed between runs.

```
RAM -> buffer -> RAM through each register set (ok = the wait passed, T = timed out):
  set 0x000, CTRL0 0x43801, drain 0x100      -> fill T, drain T; destination 2/512 bytes match (untouched); status after fill 000c3000, after drain 000c3000; DMA_ADDR 08055e40 / 00000000
  set 0x000, CTRL0 0x43801, DMA_CTRL 7, drain 0x1e0 -> fill T, drain T; destination 2/512 bytes match (untouched); status after fill 000c3000, after drain 000c3000; DMA_ADDR 08055e40 / 00000000
  set 0x400, CTRL0 0x43801, drain 0x100, wait 8 -> fill T, drain T; destination 2/512 bytes match (untouched); status after fill 000c3000, after drain 000c3000; DMA_ADDR 08055e40 / 08055e40
  set 0x400, CTRL0 0x43801, drain 0x100, wait 2 -> fill T, drain T; destination 2/512 bytes match (untouched); status after fill 000c3000, after drain 000c3000; DMA_ADDR 08055e40 / 08055e40
  set 0x000, CTRL0 + b24, drain 0x100        -> fill ok, drain ok; destination 512/512 bytes match; status after fill 021c3000, after drain 021c3000; DMA_ADDR 08055e40 / 08055e40
  set 0x000, CTRL0 + b24, DMA_CTRL 7, drain 0x1e0 -> fill ok, drain ok; destination 512/512 bytes match; status after fill 001c3000, after drain 001c3000; DMA_ADDR 08055e40 / 08055e40
  set 0x400, CTRL0 + b24, drain 0x100, wait 8 -> fill ok, drain ok; destination 512/512 bytes match; status after fill 001c3000, after drain 001c3000; DMA_ADDR 08055e40 / 08055e40
  set 0x400, CTRL0 + b24, drain 0x100, wait 2 -> fill T, drain T; destination 512/512 bytes match; status after fill 001c3000, after drain 001c3000; DMA_ADDR 08055e40 / 08055e40
```

### 14.2.88 op0916 (run 22)

ID: `D-op0916-22`. Device run 22. Experiment: [op0916](13-experiments.md#13360-op0916).

```
each form: T = timed out and faulted, otherwise the run time (a pass, or a delay shorter than the timeout):
  op 09: [0x048] & 0x1000 (bit set) T | [0x048] == 0xc3000 (all set bits) T | [0x048] & 0x2 (bit clear) T | [0xd70] == 0x12345678 T | [0xd70] & 0x8 (bit set) T | [0x800] == 0x20060808 T | r1 = ~0, imm 1 T | r1 = 0, imm 0 (until zero?) T | imm 1 (a short delay?) T | imm 0x100 T | event field 11, imm 0 T
  op 16: [0x048] & 0x1000 (bit set) 20us | [0x048] == 0xc3000 (all set bits) 19us | [0x048] & 0x2 (bit clear) 19us | [0xd70] == 0x12345678 20us | [0xd70] & 0x8 (bit set) 19us | [0x800] == 0x20060808 19us | r1 = ~0, imm 1 19us | r1 = 0, imm 0 (until zero?) 19us | imm 1 (a short delay?) 19us | imm 0x100 19us | event field 11, imm 0 20us
  op 07: [0x048] & 0x1000 (bit set) T | [0x048] == 0xc3000 (all set bits) T | [0x048] & 0x2 (bit clear) T | [0xd70] == 0x12345678 T | [0xd70] & 0x8 (bit set) T | [0x800] == 0x20060808 T | r1 = ~0, imm 1 T | r1 = 0, imm 0 (until zero?) T | imm 1 (a short delay?) T | imm 0x100 T | event field 11, imm 0 19us
  op 1a: [0x048] & 0x1000 (bit set) T | [0x048] == 0xc3000 (all set bits) T | [0x048] & 0x2 (bit clear) T | [0xd70] == 0x12345678 T | [0xd70] & 0x8 (bit set) T | [0x800] == 0x20060808 T | r1 = ~0, imm 1 T | r1 = 0, imm 0 (until zero?) T | imm 1 (a short delay?) T | imm 0x100 T | event field 11, imm 0 T
  op 20: [0x048] & 0x1000 (bit set) T | [0x048] == 0xc3000 (all set bits) T | [0x048] & 0x2 (bit clear) T | [0xd70] == 0x12345678 T | [0xd70] & 0x8 (bit set) T | [0x800] == 0x20060808 T | r1 = ~0, imm 1 T | r1 = 0, imm 0 (until zero?) T | imm 1 (a short delay?) T | imm 0x100 T | event field 11, imm 0 T
```

### 14.2.89 op16 (run 23)

ID: `D-op16-23`. Device run 23. Experiment: [op16](13-experiments.md#13361-op16).

```
opcode 16 (and 09) after one action from a clean state:
  nothing (clean)                      -> 16 T, 09 T; just before: 0x050 00000000, C50 ffff1800, ECC_DONE 10, FMC_STATUS 000c3000
  ECC decode, ECC_DONE left            -> 16 PASS, 09 T; just before: 0x050 00000000, C50 ffff1810, ECC_DONE 3d, FMC_STATUS 000c3000
  ECC decode, ECC_DONE acknowledged    -> 16 T, 09 T; just before: 0x050 00000000, C50 ffff1800, ECC_DONE 10, FMC_STATUS 000c3000
  DMA RAM -> buffer, flag left         -> 16 PASS, 09 T; just before: 0x050 00000001, C50 ffff1804, ECC_DONE 10, FMC_STATUS 001c3000
  DMA, flag acknowledged               -> 16 T, 09 T; just before: 0x050 00000001, C50 ffff1800, ECC_DONE 10, FMC_STATUS 000c3000
  70h command, flag left               -> 16 PASS, 09 T; just before: 0x050 00000000, C50 ffff1801, ECC_DONE 10, FMC_STATUS 000c3002
  70h + status latch                   -> 16 PASS, 09 T; just before: 0x050 00000000, C50 ffff1800, ECC_DONE 10, FMC_STATUS 010c3000
  READ ID through the first set        -> 16 T, 09 T; just before: 0x050 00000000, C50 ffff1800, ECC_DONE 10, FMC_STATUS 08041000
  one sector page read (to RAM)        -> 16 T, 09 T; just before: 0x050 00000000, C50 ffff1810, ECC_DONE 3c, FMC_STATUS 050c3000
```

### 14.2.90 opstate (run 24)

ID: `D-opstate-24`. Device run 24. Experiment: [opstate](13-experiments.md#13362-opstate).

```
after each state: on = continued with the next instruction, vector = went to the fault vector (C4C), T = a timeout fault happened, blocked = no end within 20 ms
  op 08 | nothing:on | command:on | DMA:on | ECC_DONE:on | status:on | all:on | second:on | second:on
  op 09 | nothing:on T | command:on T | DMA:on T | ECC_DONE:on T | status:on T | all:on T | second:on T | second:on T
  op 10 | nothing:on | command:on | DMA:on | ECC_DONE:on | status:on | all:on | second:on | second:on
  op 12 | nothing:blocked | command:blocked | DMA:blocked | ECC_DONE:blocked | status:blocked | all:blocked | second:blocked | second:blocked
  op 15 | nothing:on | command:on | DMA:on | ECC_DONE:on | status:on | all:on | second:on | second:on
  op 16 | nothing:on T | command:vector | DMA:vector | ECC_DONE:vector | status:vector | all:vector | second:vector | second:vector
  op ff | nothing:on | command:on | DMA:on | ECC_DONE:on | status:on | all:on | second:on | second:on
```

### 14.2.91 op16b (run 24)

ID: `D-op16b-24`. Device run 24. Experiment: [op16b](13-experiments.md#13363-op16b).

```
opcode 16:
  flag pending, C4C = alt                            -> went to alt (C4C)      19 us; at the target FMC_STATUS 000c3002 C50 ffff1801; C3C after 00000000
  flag pending, C4C = trap, C58 = 4                  -> went to trap           19 us; at the target FMC_STATUS 00000000 C50 00000000; C3C after 00000000
  command written just before (flag arrives later)   -> went to alt (C4C)      19 us; at the target FMC_STATUS 000c3002 C50 ffff1801; C3C after 00000000
  DMA started just before (~14 us)                   -> went to alt (C4C)      22 us; at the target FMC_STATUS 001c3000 C50 ffff1804; C3C after 00000000
  nothing pending, C4C = alt                         -> continued            4861 us (timeout fault); at the target FMC_STATUS 00000000 C50 00000000; C3C after 00000000
  flag pending, 16 twice                             -> went to alt (C4C)      19 us; at the target FMC_STATUS 000c3002 C50 ffff1801; C3C after 00000000
```

### 14.2.92 op09 (run 24)

ID: `D-op09-24`. Device run 24. Experiment: [op09](13-experiments.md#13364-op09).

```
opcode 09 against 1a, 20 and an unmet wait:
  C14 = 0 (tmwait just before)   09              -> FAULT    9725 us, fault at  9706 us, C14 before 00 after ff, CS_IRQ 04
  C14 = 0 (tmwait just before)   1a              -> FAULT    9726 us, fault at  9708 us, C14 before 00 after ff, CS_IRQ 04
  C14 = 0 (tmwait just before)   20              -> FAULT    9741 us, fault at  9724 us, C14 before 00 after ff, CS_IRQ 04
  C14 = 0 (tmwait just before)   wait 3 (unmet)  -> FAULT    9741 us, fault at  9722 us, C14 before 00 after ff, CS_IRQ 04
  C14 = 0xff (tmwait twice)      09              -> FAULT    9736 us, fault at  9718 us, C14 before ff after ff, CS_IRQ 04
  C14 = 0xff (tmwait twice)      1a              -> FAULT    9726 us, fault at  9708 us, C14 before ff after ff, CS_IRQ 04
  C14 = 0xff (tmwait twice)      20              -> FAULT    9739 us, fault at  9722 us, C14 before ff after ff, CS_IRQ 04
  C14 = 0xff (tmwait twice)      wait 3 (unmet)  -> FAULT    9728 us, fault at  9710 us, C14 before ff after ff, CS_IRQ 04
  state 1 ms in                  09              -> C08 4, C14 bf, C3C 00000000, C40 09000000, C44 00000000, C50 ffff1801, PC 20, CS_IRQ 00
  state 1 ms in                  1a              -> C08 4, C14 c0, C3C 00000000, C40 1a000000, C44 00000000, C50 ffff1801, PC 20, CS_IRQ 00
  state 1 ms in                  20              -> C08 4, C14 c0, C3C 00000000, C40 20000000, C44 00000000, C50 ffff1801, PC 20, CS_IRQ 00
  state 1 ms in                  wait 3 (unmet)  -> C08 4, C14 c0, C3C 00000000, C40 07030000, C44 00000000, C50 ffff1801, PC 20, CS_IRQ 00
  command flag (b1)              09 event field  -> TIMED OUT,   4895 us
  command flag (b1)              09 mask         -> TIMED OUT,   4879 us
  command flag (b1)              09 both         -> TIMED OUT,   4879 us
  DMA flag (b20)                 09 event field  -> TIMED OUT,   4896 us
  DMA flag (b20)                 09 mask         -> TIMED OUT,   4881 us
  DMA flag (b20)                 09 both         -> TIMED OUT,   4881 us
  ECC_DONE b0                    09 event field  -> TIMED OUT,   4892 us
  ECC_DONE b0                    09 mask         -> TIMED OUT,   4878 us
  ECC_DONE b0                    09 both         -> TIMED OUT,   4895 us
  status flag (b23)              09 event field  -> PASSED,     20 us
  status flag (b23)              09 mask         -> TIMED OUT,   4870 us
  status flag (b23)              09 both         -> PASSED,     19 us
```

### 14.2.93 op09b (run 25)

ID: `D-op09b-25`. Device run 25. Experiment: [op09b](13-experiments.md#13365-op09b).

```
opcode 09:
  status pending, 09 ev 5 arg 0    imm 0    -> passed        20 us; NSTAT before 70 after 70
  status pending, 09 ev 5 arg 40   imm 40   -> passed        20 us; NSTAT before 70 after 70
  status pending, 09 ev 5 arg 20   imm 20   -> passed        19 us; NSTAT before 70 after 70
  status pending, 09 ev 5 arg 0    imm 40   -> passed        20 us; NSTAT before 70 after 70
  status pending, 09 ev 5 arg 0    imm 1    -> passed        19 us; NSTAT before 70 after 70
  status pending, 09 ev 5 arg 1    imm 1    -> passed        19 us; NSTAT before 70 after 70
  status pending, 09 ev 5 arg 70   imm ff   -> passed        19 us; NSTAT before 70 after 70
  status pending, 09 ev 5 arg 71   imm ff   -> passed        20 us; NSTAT before 70 after 70
  status pending, 09 ev 5 arg 40   imm 0    -> passed        19 us; NSTAT before 70 after 70
  status pending, 09 ev 5 arg ff   imm ff   -> passed        20 us; NSTAT before 70 after 70
  every flag of both sets pending, 09 event field 0..15: 0:T 1:T 2:T 3:T 4:T 5:ok 6:T 7:T 8:T 9:T 10:T 11:T 12:T 13:T 14:T 15:T
  status latched, b23 acknowledged, 09 ev 5    -> passed, 19 us
  erase running, status latch, 09 ev 5 arg 0x40 imm 0x40  -> passed        19 us; NSTAT right after the latch 81, at the end 81
  erase running, status latch, 09 ev 5 arg 0 imm 0        -> passed        20 us; NSTAT right after the latch 81, at the end 81
  erase running, status latch, reference: wait 5 only     -> passed       180 us; NSTAT right after the latch 81, at the end c0
```

### 14.2.94 op09c (run 26)

ID: `D-op09c-26`. Device run 26. Experiment: [op09c](13-experiments.md#13366-op09c).

```
nothing pending (soft reset first), event numbers that pass:
  no chip selected  09 passes for: 5
  no chip selected  wait passes for: 11
  CE0 selected      09 passes for: 5
  CE0 selected      wait passes for: 11
  CE1 selected      09 passes for: 5
  CE1 selected      wait passes for: 11
```

### 14.2.95 chan1pins (run 27)

ID: `D-chan1pins-27`. Device run 27. Experiment: [chan1pins](13-experiments.md#13367-chan1pins).

```
READ ID (90h 00h, 8 bytes), FMC channel 1 pins switched or not:
  set 0x000, CE0, pins as the test setup leaves them   -> END        18 us, 0 timeouts, ID b614d5ec 14d5ec74, status 08041000
  set 0x000, CE0, channel-1 pins switched      -> END        17 us, 0 timeouts, ID b614d5ec 14d5ec74, status 08041000
  set 0x400, CE0, pins as the test setup leaves them   -> FAULT   14621 us, 3 timeouts, ID 00000000 00000000, status 0a041000
  set 0x400, CE0, channel-1 pins switched      -> FAULT   14641 us, 3 timeouts, ID 00000000 00000000, status 02041000
  set 0x400, CE1, channel-1 pins switched      -> FAULT   14627 us, 3 timeouts, ID ffffffff ffffffff, status 00041000
  set 0x400, CE2, channel-1 pins switched      -> FAULT   14624 us, 3 timeouts, ID ffffffff ffffffff, status 00041000
  set 0x400, CE3, channel-1 pins switched      -> FAULT   14630 us, 3 timeouts, ID ffffffff ffffffff, status 00041000
  set 0x400, CE4, channel-1 pins switched      -> FAULT   14627 us, 3 timeouts, ID ffffffff ffffffff, status 00041000
  set 0x400, CE5, channel-1 pins switched      -> FAULT   14627 us, 3 timeouts, ID ffffffff ffffffff, status 00041000
  set 0x400, CE6, channel-1 pins switched      -> FAULT   14627 us, 3 timeouts, ID ffffffff ffffffff, status 00041000
  set 0x400, CE7, channel-1 pins switched      -> FAULT   14626 us, 3 timeouts, ID ffffffff ffffffff, status 00041000
```

### 14.2.96 pcwrite (run 27)

ID: `D-pcwrite-27`. Device run 27. Experiment: [pcwrite](13-experiments.md#13368-pcwrite).

```
the sequencer and its PC:
  wr [0xc38], #lab             -> END    JUMPED to lab; r2 00000000
  st [0xc38], r1 (r1 = lab)    -> END    JUMPED to lab; r2 00000000
  ld r2, [0xc38] (read it)     -> END    fell through; r2 00000028
```

### 14.2.97 op12cont (run 27)

ID: `D-op12cont-27`. Device run 27. Experiment: [op12cont](13-experiments.md#13369-op12cont).

```
opcode 12 at offset 0x28, CPU writes 2 ms after the start (in this order):
  none (control)                                       -> TIMEOUT still paused                20016 us, PC 28, C08 a, CS_IRQ 00000008
  CS_IRQ = 8, start                                    -> TIMEOUT still paused                20016 us, PC 28, C08 a, CS_IRQ 00000008
  start, CS_IRQ = 8                                    -> TIMEOUT still paused                20015 us, PC 28, C08 a, CS_IRQ 00000000
  PC = pause + 8, CS_IRQ = 8, start (the 4G/5G sequence) -> END     CONTINUED after the pause    2025 us, PC 0, C08 0, CS_IRQ 00000001
  PC = pause + 8, start                                -> END     CONTINUED after the pause    2024 us, PC 0, C08 0, CS_IRQ 00000009
  CS_IRQ = 8, start 0x1                                -> TIMEOUT still paused                20019 us, PC 28, C08 a, CS_IRQ 00000008
  CS_IRQ = 8, start, start                             -> TIMEOUT still paused                20016 us, PC 28, C08 a, CS_IRQ 00000008
```

### 14.2.98 ctrlfx (run 27)

ID: `D-ctrlfx-27`. Device run 27. Experiment: [ctrlfx](13-experiments.md#13370-ctrlfx).

```
on = continued, vector = went to C4C, T<n> = n timeout faults; irq: VIC line seen while polling / at the end:
  start 0x0001 | timeout: TIMEOUT | illegal: TIMEOUT | evjmp: vector | irq: line 0/0 CS_IRQ 11
  start 0xfff5 | timeout: on T1 | illegal: on T1 | evjmp: vector | irq: line 0/0 CS_IRQ 11
  start 0xffb5 | timeout: on T1 | illegal: on T1 | evjmp: vector | irq: line 0/0 CS_IRQ 11
  start 0x0005 | timeout: on T0 | illegal: on | evjmp: vector | irq: line 0/0 CS_IRQ 15
  start 0x0011 | timeout: TIMEOUT | illegal: TIMEOUT | evjmp: vector | irq: line 0/0 CS_IRQ 11
  start 0x0021 | timeout: TIMEOUT | illegal: TIMEOUT | evjmp: vector | irq: line 0/0 CS_IRQ 11
  start 0x0041 | timeout: TIMEOUT | illegal: TIMEOUT | evjmp: vector | irq: line 0/0 CS_IRQ 11
  start 0x0081 | timeout: TIMEOUT | illegal: TIMEOUT | evjmp: vector | irq: line 0/0 CS_IRQ 11
  start 0x0101 | timeout: TIMEOUT | illegal: TIMEOUT | evjmp: vector | irq: line 0/0 CS_IRQ 11
  start 0x0201 | timeout: TIMEOUT | illegal: TIMEOUT | evjmp: vector | irq: line 0/0 CS_IRQ 11
  start 0x0401 | timeout: TIMEOUT | illegal: TIMEOUT | evjmp: vector | irq: line 0/0 CS_IRQ 11
  start 0x0801 | timeout: TIMEOUT | illegal: TIMEOUT | evjmp: vector | irq: line 0/0 CS_IRQ 11
  start 0x1001 | timeout: TIMEOUT | illegal: TIMEOUT | evjmp: vector | irq: line 0/0 CS_IRQ 11
  start 0x2001 | timeout: TIMEOUT | illegal: TIMEOUT | evjmp: vector | irq: line 0/0 CS_IRQ 11
  start 0x4001 | timeout: TIMEOUT | illegal: TIMEOUT | evjmp: vector | irq: line 0/0 CS_IRQ 11
  start 0x8001 | timeout: TIMEOUT | illegal: TIMEOUT | evjmp: vector | irq: line 0/0 CS_IRQ 11
```

### 14.2.99 op09d (run 27)

ID: `D-op09d-27`. Device run 27. Experiment: [op09d](13-experiments.md#13371-op09d).

```
second FMC set (0x400), one flag pending at a time (waits inside the set-up that time out are part of it; the sweep is the last wait):
  nothing pending        0x448 000c3000  wait2 passes for: 5
  nothing pending        0x448 000c3000  wait  passes for: 11
  command flag (b1)      0x448 000c3002  wait2 passes for: 5
  command flag (b1)      0x448 000c3002  wait  passes for: 6 11
  address flag (b2)      0x448 000c3004  wait2 passes for: 5
  address flag (b2)      0x448 000c3004  wait  passes for: 7 11
  transfer flag (b3)     0x448 08041008  wait2 passes for: 5
  transfer flag (b3)     0x448 08041008  wait  passes for: 9 11
  DMA flag (b20)         0x448 001c3000  wait2 passes for: 5
  DMA flag (b20)         0x448 001c3000  wait  passes for: 8 11
  status latch (b23?)    0x448 000c3000  wait2 passes for: 5
  status latch (b23?)    0x448 000c3000  wait  passes for: 11
```

### 14.2.100 fwidth (run 27)

ID: `D-fwidth-27`. Device run 27. Experiment: [fwidth](13-experiments.md#13372-fwidth).

```
field widths:
  wr [0x1d70]          -> END    0xd70 now 11111111
  wr [0x8d70]          -> END    0xd70 now 22222222
  wr [0xfd70]          -> END    0xd70 now 33333333
  st [0x8d70], r1      -> END    0xd70 now 44444444
  ld r2, [0x1d74]      -> END    r2 5a5a5a5a (0xd74 holds 5a5a5a5a)
  ld r2, [0x8d74]      -> END    r2 5a5a5a5a (0xd74 holds 5a5a5a5a)
  bnz      target lab | 0x1000     -> END     went elsewhere (no marker)
  jr       target lab | 0x1000     -> END     fell through
  movi r8  target lab | 0x1000     -> END     went elsewhere (no marker)
  bnz      target lab | 0x10000    -> END     went elsewhere (no marker)
  jr       target lab | 0x10000    -> END     fell through
  movi r8  target lab | 0x10000    -> END     went elsewhere (no marker)
  bnz      target lab | 0x1000000  -> FAULT   went elsewhere (no marker) (timeout fault)
  jr       target lab | 0x1000000  -> END     fell through
  movi r8  target lab | 0x1000000  -> FAULT   went elsewhere (no marker) (timeout fault)
  bnz      target lab | 0x80000000 -> END     landed on lab (upper bits ignored)
  jr       target lab | 0x80000000 -> END     fell through
  movi r8  target lab | 0x80000000 -> END     landed on lab (upper bits ignored)
```

### 14.2.101 irqsrc (run 27)

ID: `D-irqsrc-27`. Device run 27. Experiment: [irqsrc](13-experiments.md#13373-irqsrc).

```
interrupt sources (VIC bits: raw lines seen during the run that were not set in the baseline):
  baseline (nothing pending, no enables): VIC0 seen 00000020, VIC1 seen 00000002
  nothing   none        CS_IRQ 00 C50 ffff1800 FMC_STATUS 000c3000 0x050 00000000  new VIC0 00000000 VIC1 00000000
  nothing   0x044=ffff  CS_IRQ 00 C50 ffff1800 FMC_STATUS 000c3000 0x050 00000000  new VIC0 00000000 VIC1 00000000
  nothing   0x844=2f    CS_IRQ 00 C50 ffff1800 FMC_STATUS 000c3000 0x050 00000000  new VIC0 00000000 VIC1 00000000
  command   none        CS_IRQ 00 C50 ffff1801 FMC_STATUS 000c3002 0x050 00000000  new VIC0 00000000 VIC1 00000000
  command   0x044=ffff  CS_IRQ 10 C50 ffff1801 FMC_STATUS 000c3002 0x050 00000000  new VIC0 00100000 VIC1 00000000
  command   0x844=2f    CS_IRQ 00 C50 ffff1801 FMC_STATUS 000c3002 0x050 00000000  new VIC0 00000000 VIC1 00000000
  address   none        CS_IRQ 00 C50 ffff1802 FMC_STATUS 000c3004 0x050 00000000  new VIC0 00000000 VIC1 00000000
  address   0x044=ffff  CS_IRQ 10 C50 ffff1802 FMC_STATUS 000c3004 0x050 00000000  new VIC0 00000000 VIC1 00000000
  address   0x844=2f    CS_IRQ 00 C50 ffff1802 FMC_STATUS 000c3004 0x050 00000000  new VIC0 00000000 VIC1 00000000
  transfer  none        CS_IRQ 00 C50 ffff1808 FMC_STATUS 08041008 0x050 00000000  new VIC0 00000000 VIC1 00000000
  transfer  0x044=ffff  CS_IRQ 10 C50 ffff1808 FMC_STATUS 08041008 0x050 00000000  new VIC0 00000000 VIC1 00000000
  transfer  0x844=2f    CS_IRQ 00 C50 ffff1808 FMC_STATUS 08041008 0x050 00000000  new VIC0 00000000 VIC1 00000000
  DMA       none        CS_IRQ 00 C50 ffff1804 FMC_STATUS 001c3000 0x050 00000001  new VIC0 00000000 VIC1 00000000
  DMA       0x044=ffff  CS_IRQ 10 C50 ffff1804 FMC_STATUS 001c3000 0x050 00000001  new VIC0 00000000 VIC1 00000000
  DMA       0x844=2f    CS_IRQ 00 C50 ffff1804 FMC_STATUS 001c3000 0x050 00000001  new VIC0 00000000 VIC1 00000000
  ECC       none        CS_IRQ 00 C50 ffff1810 FMC_STATUS 000c3000 0x050 00000000  new VIC0 00000000 VIC1 00000000
  ECC       0x044=ffff  CS_IRQ 00 C50 ffff1810 FMC_STATUS 000c3000 0x050 00000000  new VIC0 00000000 VIC1 00000000
  ECC       0x844=2f    CS_IRQ 40 C50 ffff1810 FMC_STATUS 000c3000 0x050 00000000  new VIC0 00000000 VIC1 00000000
  status    none        CS_IRQ 00 C50 ffff1820 FMC_STATUS 008c3000 0x050 00000000  new VIC0 00000000 VIC1 00000000
  status    0x044=ffff  CS_IRQ 14 C50 ffff1800 FMC_STATUS 010c3000 0x050 00000000  new VIC0 00100000 VIC1 00000000 (FAULT)
  status    0x844=2f    CS_IRQ 04 C50 ffff1800 FMC_STATUS 010c3000 0x050 00000000  new VIC0 00000000 VIC1 00000000 (FAULT)
  command   single 0x044 bits that reach CS_IRQ: b0->CS_IRQ 10
  address   single 0x044 bits that reach CS_IRQ: b0->CS_IRQ 10, b1->CS_IRQ 10
  transfer  single 0x044 bits that reach CS_IRQ: b0->CS_IRQ 10, b1->CS_IRQ 10, b2->CS_IRQ 10, b11->CS_IRQ 10
  DMA       single 0x044 bits that reach CS_IRQ: b11->CS_IRQ 10
  status    single 0x044 bits that reach CS_IRQ: b0->CS_IRQ 10, b13->CS_IRQ 10
```

### 14.2.102 eccpar (run 27)

ID: `D-eccpar-27`. Device run 27. Experiment: [eccpar](13-experiments.md#13374-eccpar).

```
ECC engine with a known sector (no flash access):
  zeros  encode: END; ECC area words that changed: 810:00000011 814:00000180 840:0000003b 900:95982422 904:f3a2a203 908:77a3b310 90c:0000003e; parity buffer DMA done: ffffff3e77a3b310f3a2a20395982422
  zeros  decode, no error         ECC_RESULT 00000011 ECC_DONE 3d; changed vs after the encode: 840:0000003d 900:00000000 904:00000000 908:00000000 90c:00000000 9a4:0000009b
  zeros  decode, 1 bit flipped    ECC_RESULT 00000011 ECC_DONE 3d; changed vs after the encode: 840:0000003d 900:00000000 904:00000000 908:00000000 90c:00000000 9a4:0000009c
  zeros  decode, 2 bits flipped   ECC_RESULT 00000011 ECC_DONE 3d; changed vs after the encode: 840:0000003d 900:00000000 904:00000000 908:00000000 90c:00000000 9a4:0000009d
  ones   encode: END; ECC area words that changed: 840:0000003b 900:b1b427c5 904:aad5f9ff 908:83f9f67b 90c:00000043; parity buffer DMA done: ffffff4383f9f67baad5f9ffb1b427c5
  ones   decode, no error         ECC_RESULT 00000000 ECC_DONE 75; changed vs after the encode: 810:00000000 818:00000000 840:00000075 900:00000000 904:00000000 908:00000000 90c:00000000 9a0:ff0fffff 9a4:0000009e
  ones   decode, 1 bit flipped    ECC_RESULT 00000011 ECC_DONE 3d; changed vs after the encode: 840:0000003d 900:00000000 904:00000000 908:00000000 90c:00000000 9a0:ff0fffff 9a4:0000009f
  ones   decode, 2 bits flipped   ECC_RESULT 00000011 ECC_DONE 3d; changed vs after the encode: 840:0000003d 900:00000000 904:00000000 908:00000000 90c:00000000 9a0:ff0fffff 9a4:000000a0
  ramp   encode: END; ECC area words that changed: 840:0000003b 900:a816dd67 904:3112048d 908:18165999 90c:00000076; parity buffer DMA done: ffffff76181659993112048da816dd67
  ramp   decode, no error         ECC_RESULT 00000011 ECC_DONE 3d; changed vs after the encode: 840:0000003d 900:00000000 904:00000000 908:00000000 90c:00000000 9a4:000000a1
  ramp   decode, 1 bit flipped    ECC_RESULT 00000011 ECC_DONE 3d; changed vs after the encode: 840:0000003d 900:00000000 904:00000000 908:00000000 90c:00000000 9a4:000000a2
  ramp   decode, 2 bits flipped   ECC_RESULT 00000011 ECC_DONE 3d; changed vs after the encode: 840:0000003d 900:00000000 904:00000000 908:00000000 90c:00000000 9a4:000000a3
```

### 14.2.103 bufsize (run 27)

ID: `D-bufsize-27`. Device run 27. Experiment: [bufsize](13-experiments.md#13375-bufsize).

```
data buffer size (RAM -> buffer 0 -> RAM, N bytes):
  N = 256   -> fill ok, drain ok; first 256 bytes intact; bytes written past N: yes; DMA_ADDR after 08055f80 (start 08055f80)
  N = 512   -> fill ok, drain ok; first 512 bytes intact; bytes written past N: none; DMA_ADDR after 08055f80 (start 08055f80)
  N = 513   -> fill ok, drain ok; first 512 bytes intact; bytes written past N: none; DMA_ADDR after 08055f80 (start 08055f80)
  N = 1024  -> fill ok, drain ok; first 512 bytes intact; bytes written past N: none; DMA_ADDR after 08055f80 (start 08055f80)
  N = 2048  -> fill ok, drain ok; first 512 bytes intact; bytes written past N: none; DMA_ADDR after 08055f80 (start 08055f80)
```

### 14.2.104 eccstr (run 27)

ID: `D-eccstr-27`. Device run 27. Experiment: [eccstr](13-experiments.md#13376-eccstr).

```
ECC decode of one known sector with bits flipped (encode in the same mode first):
  ECC_MODE 000180,  0 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data restored
  ECC_MODE 000180,  1 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000180,  2 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000180,  3 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000180,  4 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000180,  5 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000180,  6 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000180,  7 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000180,  8 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000180,  9 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000180, 10 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000180, 11 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000180, 12 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE e00000,  1 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE e00000,  8 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE e00000,  9 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE e00000, 12 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000000,  1 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000000,  8 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000000,  9 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000000, 12 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000080,  1 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000080,  8 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000080,  9 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000080, 12 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000100,  1 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000100,  8 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000100,  9 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000100, 12 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 0001c0,  1 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 0001c0,  8 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 0001c0,  9 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 0001c0, 12 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000200,  1 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000200,  8 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000200,  9 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000200, 12 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000380,  1 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000380,  8 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000380,  9 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
  ECC_MODE 000380, 12 bits flipped -> ECC_RESULT 00000011 (b0 1, corrected  0), data as sent
```

### 14.2.105 access (run 27)

ID: `D-access-27`. Device run 27. Experiment: [access](13-experiments.md#13377-access).

```
bits that follow a write of all ones / zero (writer->reader; - = not tested). Not written: 0x004 0x008 0x038 0x07c 0x080 0x204 0x208 0x238 0x27c 0x280 0x404 0x408 0x438 0x47c 0x480 0x604 0x608 0x638 0x67c 0x680 0x80c 0xa0c 0xc00 0xc04 0xc4c 0xc58 0xc5c; not written by the sequencer: r0..r7, PC
  0x000..0x010  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x014..0x014  cpu->cpu 0000ffff  cpu->seq 0000ffff  seq->seq 0000ffff  seq->cpu 0000ffff
  0x018..0x018  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x01c..0x01c  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x020..0x020  cpu->cpu 00ffffff  cpu->seq 00ffffff  seq->seq 00ffffff  seq->cpu 00ffffff
  0x024..0x024  cpu->cpu 000001ff  cpu->seq 000001ff  seq->seq 000001ff  seq->cpu 000001ff
  0x028..0x028  cpu->cpu 00000003  cpu->seq 00000003  seq->seq 000001ff  seq->cpu 00000003
  0x02c..0x02c  cpu->cpu 00000007  cpu->seq 00000007  seq->seq 00000007  seq->cpu 00000007
  0x030..0x030  cpu->cpu 00001fff  cpu->seq 00001fff  seq->seq 00001fff  seq->cpu 00001fff
  0x034..0x040  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x044..0x044  cpu->cpu 0000ffff  cpu->seq 0000ffff  seq->seq 0000ffff  seq->cpu 0000ffff
  0x048..0x04c  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x050..0x050  cpu->cpu 00002000  cpu->seq 00002000  seq->seq 00000000  seq->cpu 00000000
  0x054..0x05c  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x060..0x078  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x084..0x1b4  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x1b8..0x1b8  cpu->cpu 00000001  cpu->seq 00000001  seq->seq 00000001  seq->cpu 00000001
  0x1bc..0x1c8  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x1cc..0x1d0  cpu->cpu 00ffffff  cpu->seq 00ffffff  seq->seq 00ffffff  seq->cpu 00ffffff
  0x1d4..0x210  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x214..0x214  cpu->cpu 0000ffff  cpu->seq 0000ffff  seq->seq 0000ffff  seq->cpu 0000ffff
  0x218..0x218  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x21c..0x21c  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x220..0x220  cpu->cpu 00ffffff  cpu->seq 00ffffff  seq->seq 00ffffff  seq->cpu 00ffffff
  0x224..0x224  cpu->cpu 000001ff  cpu->seq 000001ff  seq->seq 000001ff  seq->cpu 000001ff
  0x228..0x228  cpu->cpu 00000007  cpu->seq 00000003  seq->seq 000001ff  seq->cpu 00000007
  0x22c..0x22c  cpu->cpu 00000007  cpu->seq 00000007  seq->seq 00000007  seq->cpu 00000007
  0x230..0x230  cpu->cpu 00001fff  cpu->seq 00001fff  seq->seq 00001fff  seq->cpu 00001fff
  0x234..0x240  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x244..0x244  cpu->cpu 0000ffff  cpu->seq 0000ffff  seq->seq 0000ffff  seq->cpu 0000ffff
  0x248..0x24c  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x250..0x250  cpu->cpu 00003000  cpu->seq 00003000  seq->seq 00002800  seq->cpu 00002800
  0x254..0x25c  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x260..0x278  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x284..0x3a4  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x3a8..0x3b4  cpu->cpu 00000000  cpu->seq ffffffff  seq->seq 00000000  seq->cpu 00000000
  0x3b8..0x3b8  cpu->cpu 00000001  cpu->seq 00000001  seq->seq 00000001  seq->cpu 00000001
  0x3bc..0x3c8  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x3cc..0x3d0  cpu->cpu 00ffffff  cpu->seq 00ffffff  seq->seq 00ffffff  seq->cpu 00ffffff
  0x3d4..0x3e4  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x3e8..0x3fc  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x400..0x410  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x414..0x414  cpu->cpu 0000ffff  cpu->seq 0000ffff  seq->seq 0000ffff  seq->cpu 0000ffff
  0x418..0x418  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x41c..0x41c  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x420..0x420  cpu->cpu 00ffffff  cpu->seq 00ffffff  seq->seq 00ffffff  seq->cpu 00ffffff
  0x424..0x424  cpu->cpu 000001ff  cpu->seq 000001ff  seq->seq 000001ff  seq->cpu 000001ff
  0x428..0x428  cpu->cpu 000001ff  cpu->seq 00000007  seq->seq 000001ff  seq->cpu 000001ff
  0x42c..0x42c  cpu->cpu 00000007  cpu->seq 00000007  seq->seq 00000007  seq->cpu 00000007
  0x430..0x430  cpu->cpu 00001fff  cpu->seq 00001fff  seq->seq 00001fff  seq->cpu 00001fff
  0x434..0x440  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x444..0x444  cpu->cpu 0000ffff  cpu->seq 0000ffff  seq->seq 0000ffff  seq->cpu 0000ffff
  0x448..0x44c  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x450..0x450  cpu->cpu 00000000  cpu->seq 00003000  seq->seq 00003000  seq->cpu 00003000
  0x454..0x45c  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x460..0x478  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x484..0x4b0  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x4b4..0x4b8  cpu->cpu 00000001  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x4bc..0x5b4  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x5b8..0x5b8  cpu->cpu 00000001  cpu->seq 00000001  seq->seq 00000001  seq->cpu 00000001
  0x5bc..0x5c8  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x5cc..0x5d0  cpu->cpu 00ffffff  cpu->seq 00ffffff  seq->seq 00ffffff  seq->cpu 00ffffff
  0x5d4..0x5f8  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x5fc..0x5fc  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x600..0x610  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x614..0x614  cpu->cpu 0000ffff  cpu->seq 0000ffff  seq->seq 0000ffff  seq->cpu 0000ffff
  0x618..0x618  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x61c..0x61c  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x620..0x620  cpu->cpu 00ffffff  cpu->seq 00ffffff  seq->seq 00ffffff  seq->cpu 00ffffff
  0x624..0x624  cpu->cpu 000001ff  cpu->seq 000001ff  seq->seq 000001ff  seq->cpu 000001ff
  0x628..0x628  cpu->cpu 00000003  cpu->seq 00000007  seq->seq 000001ff  seq->cpu 00000003
  0x62c..0x62c  cpu->cpu 00000007  cpu->seq 00000007  seq->seq 00000007  seq->cpu 00000007
  0x630..0x630  cpu->cpu 00001fff  cpu->seq 00001fff  seq->seq 00001fff  seq->cpu 00001fff
  0x634..0x640  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x644..0x644  cpu->cpu 0000ffff  cpu->seq 0000ffff  seq->seq 0000ffff  seq->cpu 0000ffff
  0x648..0x64c  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x650..0x650  cpu->cpu 00001000  cpu->seq 00001000  seq->seq 00000000  seq->cpu 00000000
  0x654..0x65c  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x660..0x678  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x684..0x6cc  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x6d0..0x6d0  cpu->cpu 00000001  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x6d4..0x7b4  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x7b8..0x7b8  cpu->cpu 00000001  cpu->seq 00000001  seq->seq 00000001  seq->cpu 00000001
  0x7bc..0x7c8  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x7cc..0x7d0  cpu->cpu 00ffffff  cpu->seq 00ffffff  seq->seq 00ffffff  seq->cpu 00ffffff
  0x7d4..0x7fc  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x800..0x800  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x804..0x808  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x810..0x810  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x814..0x814  cpu->cpu 01ffffff  cpu->seq 01ffffff  seq->seq 01ffffff  seq->cpu 01ffffff
  0x818..0x840  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x844..0x844  cpu->cpu 0000002f  cpu->seq 0000002f  seq->seq 0000002f  seq->cpu 0000002f
  0x848..0x93c  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x940..0x950  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0x954..0x9bc  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0x9c0..0x9c0  cpu->cpu 00000001  cpu->seq 00000001  seq->seq 00000001  seq->cpu 00000001
  0x9c4..0xa00  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0xa04..0xa08  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0xa10..0xa10  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0xa14..0xa14  cpu->cpu 01ffffff  cpu->seq 01ffffff  seq->seq 01ffffff  seq->cpu 01ffffff
  0xa18..0xa40  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0xa44..0xa44  cpu->cpu 0000002f  cpu->seq 0000002f  seq->seq 0000002f  seq->cpu 0000002f
  0xa48..0xb3c  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0xb40..0xb50  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0xb54..0xbbc  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0xbc0..0xbc0  cpu->cpu 00000001  cpu->seq 00000001  seq->seq 00000001  seq->cpu 00000001
  0xbc4..0xc0c  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0xc10..0xc10  cpu->cpu 000000ff  cpu->seq 000000ff  seq->seq 000000ff  seq->cpu 000000ff
  0xc14..0xc14  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0xc18..0xc34  cpu->cpu ffffffff  cpu->seq 00000000  seq->seq -  seq->cpu -
  0xc38..0xc38  cpu->cpu 00000000  cpu->seq 00000000  seq->seq -  seq->cpu -
  0xc3c..0xc3c  cpu->cpu ffffffff  cpu->seq 00000000  seq->seq ffffffff  seq->cpu 00000000
  0xc40..0xc44  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0xc48..0xc48  cpu->cpu ffffffff  cpu->seq 00000000  seq->seq ffffffff  seq->cpu ffffffff
  0xc50..0xc50  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0xc54..0xc54  cpu->cpu 00000fff  cpu->seq 00000000  seq->seq 00000fff  seq->cpu 00000fff
  0xc60..0xcfc  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
  0xd00..0xd7c  cpu->cpu ffffffff  cpu->seq ffffffff  seq->seq ffffffff  seq->cpu ffffffff
  0xd80..0xffc  cpu->cpu 00000000  cpu->seq 00000000  seq->seq 00000000  seq->cpu 00000000
```

### 14.2.106 fields0 (run 28)

ID: `D-fields0-28`. Device run 28. Experiment: [fields0](13-experiments.md#13378-fields0).

```
fields left 0 by the original programs (expected without an effect: r2 = 7777cafe / aaaaaaaa / 11111111, the word written at its plain place):
  st [0xd70], r7, imm 0xff     -> END    r2 00000000  0xd70..0xd78 7777cafe 22222222 33333333  RAM +0..+12 aaaaaaaa bbbbbbbb cccccccc dddddddd
  ldi r2, [r1], imm 0xff       -> END    r2 bbbbbbbb  0xd70..0xd78 11111111 22222222 33333333  RAM +0..+12 aaaaaaaa bbbbbbbb cccccccc dddddddd
  mov r2, r7, imm 0xff         -> END    r2 7777cafe  0xd70..0xd78 11111111 22222222 33333333  RAM +0..+12 aaaaaaaa bbbbbbbb cccccccc dddddddd
  sti [r1], r7, imm 0xff       -> END    r2 00000000  0xd70..0xd78 11111111 22222222 33333333  RAM +0..+12 aaaaaaaa 7777cafe cccccccc dddddddd
  pld r2, [r5], imm 0xff       -> END    r2 11111111  0xd70..0xd78 11111111 22222222 33333333  RAM +0..+12 aaaaaaaa bbbbbbbb cccccccc dddddddd
  pst [r5], r7, imm 0xff       -> END    r2 00000000  0xd70..0xd78 7777cafe 22222222 33333333  RAM +0..+12 aaaaaaaa bbbbbbbb cccccccc dddddddd
  st [0xd70], r7, imm 0x4      -> END    r2 00000000  0xd70..0xd78 7777cafe 22222222 33333333  RAM +0..+12 aaaaaaaa bbbbbbbb cccccccc dddddddd
  ldi r2, [r1], imm 0x4        -> END    r2 bbbbbbbb  0xd70..0xd78 11111111 22222222 33333333  RAM +0..+12 aaaaaaaa bbbbbbbb cccccccc dddddddd
  mov r2, r7, imm 0x4          -> END    r2 7777cafe  0xd70..0xd78 11111111 22222222 33333333  RAM +0..+12 aaaaaaaa bbbbbbbb cccccccc dddddddd
  sti [r1], r7, imm 0x4        -> END    r2 00000000  0xd70..0xd78 11111111 22222222 33333333  RAM +0..+12 aaaaaaaa 7777cafe cccccccc dddddddd
  pld r2, [r5], imm 0x4        -> END    r2 11111111  0xd70..0xd78 11111111 22222222 33333333  RAM +0..+12 aaaaaaaa bbbbbbbb cccccccc dddddddd
  pst [r5], r7, imm 0x4        -> END    r2 00000000  0xd70..0xd78 7777cafe 22222222 33333333  RAM +0..+12 aaaaaaaa bbbbbbbb cccccccc dddddddd
  wr [0xd70], #x, R 3          -> END    r2 00000000  0xd70..0xd78 44444444 22222222 33333333  RAM +0..+12 aaaaaaaa bbbbbbbb cccccccc dddddddd
  wr [0xd70], #x, R 7          -> END    r2 00000000  0xd70..0xd78 44444444 22222222 33333333  RAM +0..+12 aaaaaaaa bbbbbbbb cccccccc dddddddd
  movi r2, #x, arg 0x1234      -> END    r2 55555555  0xd70..0xd78 11111111 22222222 33333333  RAM +0..+12 aaaaaaaa bbbbbbbb cccccccc dddddddd
  movi r2, #x, arg 0xd70       -> END    r2 55555555  0xd70..0xd78 11111111 22222222 33333333  RAM +0..+12 aaaaaaaa bbbbbbbb cccccccc dddddddd
  end                          -> END    SEQ_RESULT 66666666, r6 66666666, CS_IRQ 00000001
  end R 1                      -> FAULT  SEQ_RESULT bad0bad0, r6 bad0bad0, CS_IRQ 00000005
  end arg 1                    -> FAULT  SEQ_RESULT bad0bad0, r6 bad0bad0, CS_IRQ 00000005
  end imm 0x12345678           -> END    SEQ_RESULT 66666666, r6 66666666, CS_IRQ 00000001
  end R 1, arg 7, imm 0xff     -> FAULT  SEQ_RESULT bad0bad0, r6 bad0bad0, CS_IRQ 00000005
```

### 14.2.107 tmofield (run 28)

ID: `D-tmofield-28`. Device run 28. Experiment: [tmofield](13-experiments.md#13379-tmofield).

```
time from the start to the timeout of a wait that is never met:
  start 0x0005 -> FAULT  fault at      2 us, C14 after the fault fe
  start 0x0105 -> FAULT  fault at      0 us, C14 after the fault fe
  start 0x0205 -> FAULT  fault at      2 us, C14 after the fault fe
  start 0x0405 -> FAULT  fault at      0 us, C14 after the fault fe
  start 0x1005 -> FAULT  fault at      2 us, C14 after the fault fe
  start 0x4005 -> FAULT  fault at      0 us, C14 after the fault fe
  start 0x8005 -> FAULT  fault at      0 us, C14 after the fault fe
  start 0xff05 -> FAULT  fault at      2 us, C14 after the fault fe
  start 0x00f5 -> FAULT  fault at     16 us, C14 after the fault 00
  start 0x0ff5 -> FAULT  fault at    310 us, C14 after the fault ff
  start 0x0f05 -> FAULT  fault at      0 us, C14 after the fault fe
  start 0xfff5 -> FAULT  fault at   4868 us, C14 after the fault ff
```

### 14.2.108 eccstr2 (run 28)

ID: `D-eccstr2-28`. Device run 28. Experiment: [eccstr2](13-experiments.md#13380-eccstr2).

```
ECC, encode and decode in one program:
   0 bits flipped -> END    ECC_RESULT 00000000 (uncorrectable 0, corrected  0), ECC_DONE 35, data corrected
   1 bits flipped -> END    ECC_RESULT 00010000 (uncorrectable 0, corrected  1), ECC_DONE 35, data corrected
   2 bits flipped -> END    ECC_RESULT 00020000 (uncorrectable 0, corrected  2), ECC_DONE 35, data corrected
   3 bits flipped -> END    ECC_RESULT 00030000 (uncorrectable 0, corrected  3), ECC_DONE 35, data corrected
   4 bits flipped -> END    ECC_RESULT 00040000 (uncorrectable 0, corrected  4), ECC_DONE 35, data corrected
   5 bits flipped -> END    ECC_RESULT 00050000 (uncorrectable 0, corrected  5), ECC_DONE 35, data corrected
   6 bits flipped -> END    ECC_RESULT 00060000 (uncorrectable 0, corrected  6), ECC_DONE 35, data corrected
   7 bits flipped -> END    ECC_RESULT 00070000 (uncorrectable 0, corrected  7), ECC_DONE 35, data corrected
   8 bits flipped -> END    ECC_RESULT 00080000 (uncorrectable 0, corrected  8), ECC_DONE 35, data corrected
   9 bits flipped -> END    ECC_RESULT 00000011 (uncorrectable 1, corrected  0), ECC_DONE 3d, data as sent
  10 bits flipped -> END    ECC_RESULT 00000011 (uncorrectable 1, corrected  0), ECC_DONE 3d, data as sent
  11 bits flipped -> END    ECC_RESULT 00000011 (uncorrectable 1, corrected  0), ECC_DONE 3d, data as sent
  12 bits flipped -> END    ECC_RESULT 00000011 (uncorrectable 1, corrected  0), ECC_DONE 3d, data as sent
  parity of zeros    : ffffff3e77a3b310f3a2a20395982422
  parity of ones     : ffffff4383f9f67baad5f9ffb1b427c5
  parity of bit 0    : ffffff92b29212590fea32db09c2cda8
  parity of bit 1    : ffffff7204d411f8072004f368e90c15
  parity of bit 2    : ffffffa6914cf6c11aa7efe26f7a744c
  parity of bit 7    : ffffff56c92cce6b0a1464a31e146d94
  parity of bit 8    : fffffff725bfb0d3e57ba1f05a0ba788
  parity of bit 9    : ffffffb92a8f54edd20322a5cf7bd855
  parity of bit 4088 : ffffff691653865aeb56c346eef4b367
  parity of bit 4095 : ffffff8b9c96fade676d55c970afc0f9
```

### 14.2.109 ev5en (run 28)

ID: `D-ev5en-28`. Device run 28. Experiment: [ev5en](13-experiments.md#13381-ev5en).

```
status latch and wait 5 (T = wait 5 timed out), FMC_STATUS right after the latch and after the wait:
  no enable              ok (before wait 008c3000, after 008c3000) | ok (before wait 008c3000, after 008c3000) | ok (before wait 008c3000, after 008c3000)
  0x044 = 1 (CPU)        ok (before wait 008c3000, after 008c3000) | ok (before wait 008c3000, after 008c3000) | ok (before wait 008c3000, after 008c3000)
  0x044 = 0x2000 (CPU)   ok (before wait 008c3000, after 008c3000) | ok (before wait 008c3000, after 008c3000) | ok (before wait 008c3000, after 008c3000)
  0x844 = 1 (CPU)        ok (before wait 008c3000, after 008c3000) | ok (before wait 008c3000, after 008c3000) | ok (before wait 008c3000, after 008c3000)
  0x044 = 1 (program)    ok (before wait 008c3000, after 008c3000) | ok (before wait 008c3000, after 008c3000) | ok (before wait 008c3000, after 008c3000)
```

### 14.2.110 tmobits (run 29)

ID: `D-tmobits-29`. Device run 29. Experiment: [tmobits](13-experiments.md#13382-tmobits).

```
time from the start to the timeout of a wait that is never met:
  start 0xff15 -> FAULT  fault at     18 us, C14 after the fault ff
  start 0xff25 -> FAULT  fault at     72 us, C14 after the fault ff
  start 0xff45 -> FAULT  fault at   1218 us, C14 after the fault ff
  start 0xff85 -> FAULT  fault at   4868 us, C14 after the fault ff
  start 0xff35 -> FAULT  fault at    296 us, C14 after the fault ff
  start 0xffc5 -> FAULT  fault at   4868 us, C14 after the fault ff
  start 0x0f15 -> FAULT  fault at      2 us, C14 after the fault ff
  start 0x0f25 -> FAULT  fault at      0 us, C14 after the fault ff
  start 0x0f45 -> FAULT  fault at     72 us, C14 after the fault ff
  start 0x0f85 -> FAULT  fault at    308 us, C14 after the fault ff
  start 0x01f5 -> FAULT  fault at     46 us, C14 after the fault ff
  start 0x03f5 -> FAULT  fault at     72 us, C14 after the fault ff
  start 0x07f5 -> FAULT  fault at    152 us, C14 after the fault ff
  start 0x3ff5 -> FAULT  fault at   1218 us, C14 after the fault ff
  start 0x7ff5 -> FAULT  fault at   2430 us, C14 after the fault ff
```

### 14.2.111 eccmode (run 29)

ID: `D-eccmode-29`. Device run 29. Experiment: [eccmode](13-experiments.md#13383-eccmode).

```
ECC_MODE sweep (decode: flipped bits: ECC_RESULT/ECC_DONE, data corrected / as sent / other):
  mode 0x0000180: END; encode done 3b; slot 0 ffffff7a82a426c457e3a863a1cb34c8 (the 0x180 code); slots 1..3 ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff; 0x900..0x93c 00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
      decode  0: 00000000/35 corr |  8: 00080000/35 corr |  9: 00000011/3d sent | 12: 00000011/3d sent | 16: 00000011/3d sent
  mode 0x0000000: END; encode done 33; slot 0 ffffffffffffffffffffffffffffffff (DIFFERENT); slots 1..3 ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff; 0x900..0x93c 233c853f32c8ce55a1c321a1fc000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
      decode  0: 00000011/3d corr |  8: 00000011/3d sent |  9: 00000011/3d sent | 12: 00000011/3d sent | 16: 00000011/3d sent
  mode 0x0000080: END; encode done 3b; slot 0 ffffffffffffffffffffffffffffffff (DIFFERENT); slots 1..3 ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff; 0x900..0x93c dcfca60c49c28aa039836fbe7c000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
      decode  0: 00000011/3d corr |  8: 00000011/3d sent |  9: 00000011/3d sent | 12: 00000011/3d sent | 16: 00000011/3d sent
  mode 0x0000100: END; encode done 3b; slot 0 ffffff7a82a426c457e3a863a1cb34c8 (the 0x180 code); slots 1..3 ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff; 0x900..0x93c 00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
      decode  0: 00000000/35 corr |  8: 00080000/35 corr |  9: 00000011/3d sent | 12: 00000011/3d sent | 16: 00000011/3d sent
  mode 0x00001c0: END; encode done 3b; slot 0 ffffff7a82a426c457e3a863a1cb34c8 (the 0x180 code); slots 1..3 ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff; 0x900..0x93c 00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
      decode  0: 00000000/35 corr |  8: 00080000/35 corr |  9: 00000011/3d sent | 12: 00000011/3d sent | 16: 00000011/3d sent
  mode 0x0000181: END; encode done 3b; slot 0 ffffff7a82a426c457e3a863a1cb34c8 (the 0x180 code); slots 1..3 ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff; 0x900..0x93c 00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
      decode  0: 00000000/35 corr |  8: 00080000/35 corr |  9: 00000011/3d sent | 12: 00000011/3d sent | 16: 00000011/3d sent
  mode 0x0000380: END; encode done 3b; slot 0 ffffff7a82a426c457e3a863a1cb34c8 (the 0x180 code); slots 1..3 ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff; 0x900..0x93c 00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
      decode  0: 00000000/35 corr |  8: 00080000/35 corr |  9: 00000011/3d sent | 12: 00000011/3d sent | 16: 00000011/3d sent
  mode 0x0000780: END; encode done 3b; slot 0 ffffff7a82a426c457e3a863a1cb34c8 (the 0x180 code); slots 1..3 ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff; 0x900..0x93c 00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
      decode  0: 00000000/35 corr |  8: 00080000/35 corr |  9: 00000011/3d sent | 12: 00000011/3d sent | 16: 00000011/3d sent
  mode 0x0e00000: END; encode done 1b; slot 0 ffffffffffffffffffffffffffffffff (DIFFERENT); slots 1..3 ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff; 0x900..0x93c b5d2b4ab2eacd0192881214ff7000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
      decode  0: 00000011/1d corr |  8: 00000011/1d sent |  9: 00000011/1d sent | 12: 00000011/1d sent | 16: 00000011/1d sent
  mode 0x1ffffff: END; encode done 13; slot 0 509999fff94faba1ffff8124ffffffff (DIFFERENT); slots 1..3 ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff; 0x900..0x93c 8124f94faba1509999ff31982cb8c5549cad90d80000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
      decode  0: 00000000/15 corr |  8: 00010000/15 other |  9: 00010000/15 other | 12: 00020000/15 other | 16: 00020000/15 other
```

### 14.2.112 eccchunk (run 29)

ID: `D-eccchunk-29`. Device run 29. Experiment: [eccchunk](13-experiments.md#13384-eccchunk).

```
parity buffer DMA, meta encode, decode of host-built chunks:
  DMA memory -> parity buffer (DATA_SEL 0x10) -> memory: END, first 64 bytes back c5d2dfecf90613202d3a4754616e7b8895a2afbcc9d6e3f0fd0a1724313e4b5865727f8c99a6b3c0cddae7f4010e1b2835424f5c697683909daab7c4d1deebf8
  DMA memory -> parity buffer (DATA_SEL 0x20) -> memory: END, first 64 bytes back c5d2dfecf90613202d3a4754616e7b8895a2afbcc9d6e3f0fd0a1724313e4b5865727f8c99a6b3c0cddae7f4010e1b2835424f5c697683909daab7c4d1deebf8
  meta encode (META = a0..ab, 0x3210) + encode slots 0..3: END; slots: a0a1a2aa0cd773401a693eb53a632ccf(code ok) a3a4a5ee0a13318e6b8599c72dadb6c5(code ok) a6a7a86c8947e91865fd95df565803cb(code ok) a9aaab0199264d18b5f03addb57f8e6c(code ok)
  decode, clean                                                -> END   ECC_RESULT 00000011 ECC_DONE 3d, data clean, slot ffffffffffffffffffffffffffffffff
  decode, meta 1 bit                                           -> END   ECC_RESULT 00000011 ECC_DONE 3d, data clean, slot ffffffffffffffffffffffffffffffff
  decode, meta 3 bits                                          -> END   ECC_RESULT 00000011 ECC_DONE 3d, data clean, slot ffffffffffffffffffffffffffffffff
  decode, parity 1 bit                                         -> END   ECC_RESULT 00000011 ECC_DONE 3d, data clean, slot ffffffffffffffffffffffffffffffff
  decode, parity 8 bits                                        -> END   ECC_RESULT 00000011 ECC_DONE 3d, data clean, slot ffffffffffffffffffffffffffffffff
  decode, data 4 + parity 4                                    -> END   ECC_RESULT 00000011 ECC_DONE 3d, data as sent, slot ffffffffffffffffffffffffffffffff
  decode, parity 9 bits                                        -> END   ECC_RESULT 00000011 ECC_DONE 3d, data clean, slot ffffffffffffffffffffffffffffffff
  decode, meta byte order: chunk built with the bytes reversed -> END   ECC_RESULT 00000011 ECC_DONE 3d, data clean, slot ffffffffffffffffffffffffffffffff
```

### 14.2.113 rawpage (run 29)

ID: `D-rawpage-29`. Device run 29. Experiment: [rawpage](13-experiments.md#13385-rawpage).

```
raw page reads, CE0 (chunk as stored, checked against the identified code; res = engine decode, st = FMC_STATUS after the sector's data):
  row 0x000000 END   s0 erased                 res 00000000 st 0003c000 chunk ffffffffffffffffffffffffffffffff
                   s1 erased                 res 00000000 st 0003c000 chunk ffffffffffffffffffffffffffffffff
                   s2 erased                 res 00000000 st 0003c000 chunk ffffffffffffffffffffffffffffffff
                   s3 erased                 res 00000000 st 0003c000 chunk ffffffffffffffffffffffffffffffff
  row 0x000080 END   s0 code ok                res 00000000 st 0803c000 chunk ffffff26426253358a227a1c8f3335a1
                   s1 code ok                res 00000000 st 0803c000 chunk ffffff3e77a3b310f3a2a20395982422
                   s2 code ok                res 00000000 st 0803c000 chunk ffff00620004b809a9fcc776ac46a657
                   s3 code ok                res 00000000 st 0803c000 chunk 80ffffaa9f4f48da3bf3aa151041f41b
  row 0x003201 END   s0 code ok                res 00000000 st 0803c000 chunk 04140258c636bf190b43b15561dbd1d4
                   s1 code ok                res 00000000 st 0803c000 chunk 00ff34dcdefb0beb4fa0ab138f638dea
                   s2 code ok                res 00000000 st 0803c000 chunk 0100ff43691540bf75e726d04dcade29
                   s3 code ok                res 00000000 st 0803c000 chunk 40ffff92bf5756e072be0ee05bb953af
  row 0x01f400 END   s0 code ok                res 00000000 st 0803c000 chunk 00d211c8cfcc6399cf5e09fb0c07f760
                   s1 code ok                res 00000000 st 0803c000 chunk 008a78afd219b4b2b6eeb1b12e15474c
                   s2 code ok                res 00000000 st 0803c000 chunk 0100ff603c9955c82679227a222659c8
                   s3 code ok                res 00000000 st 0803c000 chunk 40fffff6cb3c9bc78c119b5d5149396f
  row 0x03e805 END   s0 code ok                res 00000000 st 0803c000 chunk 14cc18d7420fb10f9e743d3f24917b80
                   s1 code ok                res 00000000 st 0803c000 chunk 00a3103cbdfd4ad044e1a8005add4875
                   s2 code ok                res 00000000 st 0803c000 chunk 0100ff8af264ab05b72499e4a88f3d50
                   s3 code ok                res 00000000 st 0803c000 chunk 40ffff47478af4bc6cb7daef940d7097
  row 0x06d600 END   s0 code ok                res 00000000 st 0803c000 chunk 00ba1a48cc88d9cec52dda6d0ab228fd
                   s1 code ok                res 00000000 st 0803c000 chunk 00f2379e30a95091c82dd4cf484a2496
                   s2 code ok                res 00000000 st 0803c000 chunk 0000ff64a6e3b23c54a70b948491c0f7
                   s3 code ok                res 00000000 st 0803c000 chunk 40ffff47478af4bc6cb7daef940d7097
```

### 14.2.114 rawwrite (run 29)

ID: `D-rawwrite-29`. Device run 29. Experiment: [rawwrite](13-experiments.md#13386-rawwrite).

```
scratch block write paths and raw read-back (res = engine decode, st = FMC_STATUS after the data):
  erase block: END, NSTAT c0
  p0 write (engine: meta encode + encode): END, NSTAT 70
  p1 write (host chunks via parity-buffer DMA): END, NSTAT 70
  p2 write (raw: chunks from a data buffer): END, NSTAT 70
  p3 write (raw, stored errors: 8 / 9 data bits, zero chunk, 2 meta bits): END, NSTAT 70
  p0 s0 code ok                res 00000000 st 0803c000 chunk 4142430648a7fd436a0dee8ab9ada46b data = sent; meta bytes 414243 (sent 414243)
  p0 s1 code ok                res 00000000 st 0803c000 chunk 44454677ea778d147a2fcf612a367686 data = sent; meta bytes 444546 (sent 444546)
  p0 s2 code ok                res 00000000 st 0803c000 chunk 4748492e5699c1d567692e96ccbd6e9c data = sent; meta bytes 474849 (sent 474849)
  p0 s3 code ok                res 00000000 st 0803c000 chunk 4a4b4c0b3164549d25efb6b1c911aaee data = sent; meta bytes 4a4b4c (sent 4a4b4c)
  p1 s0 NOT a code word        res 00000011 st 0803c000 chunk ffffffffffffffffffffffffffffffff stored data =, chunk ffffffffffffffffffffffffffffffff (sent 5152535212d40d397543e85a7e77add2)
  p1 s1 NOT a code word        res 00000011 st 0803c000 chunk ffffffffffffffffffffffffffffffff stored data =, chunk ffffffffffffffffffffffffffffffff (sent 545556dd1fc4a9255e3d81800b1abd7c)
  p1 s2 NOT a code word        res 00000011 st 0803c000 chunk ffffffffffffffffffffffffffffffff stored data =, chunk ffffffffffffffffffffffffffffffff (sent 575859de0ebed657e57ce2bf076fa9ba)
  p1 s3 NOT a code word        res 00000011 st 0803c000 chunk ffffffffffffffffffffffffffffffff stored data =, chunk ffffffffffffffffffffffffffffffff (sent 5a5b5c543088613bc2cb500867a47ec2)
  p2 s0 code ok                res 00000000 st 0803c000 chunk 6162631dd97391e8540db3213763a487 stored = sent
  p2 s1 code ok                res 00000000 st 0803c000 chunk 646566b00b3a1f2b2c63e8dfe437e293 stored = sent
  p2 s2 code ok                res 00000000 st 0803c000 chunk 676869d9a73e1362d1fc3d945b763539 stored = sent
  p2 s3 code ok                res 00000000 st 0803c000 chunk 6a6b6cdfb63a2fa44c0ffd8760a75c0b stored = sent
  p3 s0 NOT a code word        res 00080000 st 0803c000 chunk 717273bde0bf8e08c0d201d5e5f1ed1c stored = sent; after decode clean
  p3 s1 NOT a code word        res 00000011 st 0803c000 chunk 7475760b30bb75534fa12695523e478e stored = sent; after decode as stored
  p3 s2 NOT a code word        res 00000011 st 0803c000 chunk 00000000000000000000000000000000 stored = sent; after decode clean
  p3 s3 NOT a code word        res 00020000 st 0803c000 chunk 7e7b3ca20e7f5dc82ce0e82e8d1d9744 stored = sent; after decode clean
  p4 s0 erased                 res 00000000 st 0003c000 chunk ffffffffffffffffffffffffffffffff 
  p4 s1 erased                 res 00000000 st 0003c000 chunk ffffffffffffffffffffffffffffffff 
  p4 s2 erased                 res 00000000 st 0003c000 chunk ffffffffffffffffffffffffffffffff 
  p4 s3 erased                 res 00000000 st 0003c000 chunk ffffffffffffffffffffffffffffffff
```

### 14.2.115 writeseq (run 29)

ID: `D-writeseq-29`. Device run 29. Experiment: [writeseq](13-experiments.md#13387-writeseq).

```
write sequences (scratch blocks), raw read-back:
  erase CE0 block 3000: FAULT, NSTAT after D0h c0, ready c0, 6003 us
  erase CE0 block 3001: FAULT, NSTAT after D0h c0, ready c0, 6009 us
  erase CE1 block 3000: FAULT, NSTAT after D0h c0, ready c0, 6029 us
  (1) 4 pages, 0x2e4 overlap, DMA_CTRL 7: END, NSTAT per page 70 70 70 70, 941 us
  read (1) CE0 page 10                    all 4 sectors = expected (data + meta + parity); ECC_RESULT 0 0 0 0
  read (1) CE0 page 11                    sectors [0, 1, 2, 3] DIFFER: ffffffffffffffffffffffffffffffff ffffffffffffffffffffffffffffffff ffffffffffffffffffffffffffffffff ffffffffffffffffffffffffffffffff; ECC_RESULT 0 0 0 0
  read (1) CE0 page 12                    all 4 sectors = expected (data + meta + parity); ECC_RESULT 0 0 0 0
  read (1) CE0 page 13                    sectors [0, 1, 2, 3] DIFFER: ffffffffffffffffffffffffffffffff ffffffffffffffffffffffffffffffff ffffffffffffffffffffffffffffffff ffffffffffffffffffffffffffffffff; ECC_RESULT 0 0 0 0
  (2) CE0 + CE1 in one run: END, NSTAT 70 / 70
  read (2) CE0 page 20                    all 4 sectors = expected (data + meta + parity); ECC_RESULT 0 0 0 0
  read (2) CE1 page 20                    all 4 sectors = expected (data + meta + parity); ECC_RESULT 0 0 0 0
  (3) two-plane 80h..11h / 81h..10h: END, NSTAT 70 / 70, 473 us
  read (3) block B page 30                all 4 sectors = expected (data + meta + parity); ECC_RESULT 0 0 0 0
  read (3) block B+1 page 30              all 4 sectors = expected (data + meta + parity); ECC_RESULT 0 0 0 0
  (4) page 10 programmed again without an erase: END, NSTAT 70 (b0 = FAIL)
    s0 data = old AND new, ECC_RESULT 00000011
    s1 data = old AND new, ECC_RESULT 00000011
    s2 data = old AND new, ECC_RESULT 00000011
    s3 data = old AND new, ECC_RESULT 00000011
```

### 14.2.116 op09nand (run 29)

ID: `D-op09nand-29`. Device run 29. Experiment: [op09nand](13-experiments.md#13388-op09nand).

```
wait2 against chip activity (scratch blocks):
  erase running     wait2 passes for: none   [N = 0: fault at 4868 us, NSTAT after c0]
  erase finished    wait2 passes for: none   [N = 0: fault at 5028 us, NSTAT after c0]
  program running   wait2 passes for: none   [N = 0: fault at 4910 us, NSTAT after 70]
  program finished  wait2 passes for: none   [N = 0: fault at 4908 us, NSTAT after 70]
  read tR running   wait2 passes for: none   [N = 0: fault at 4868 us, NSTAT after c0]
  idle              wait2 32..255 passes for: none
  erase running     wait2 32..255 passes for: none
```

### 14.2.117 eraseseq (run 29)

ID: `D-eraseseq-29`. Device run 29. Experiment: [eraseseq](13-experiments.md#13389-eraseseq).

```
erase (scratch blocks; per page: four sectors' state):
    before: CE0 block B page 10                NOT a code word NOT a code word NOT a code word NOT a code word
    before: CE0 block B page 20                code ok code ok code ok code ok
    before: CE0 block B page 30                code ok code ok code ok code ok
  erase CE0 block B (written pages)            FAULT, NSTAT after D0h c0, ready c0,  6002 us
    after: CE0 block B page 10                 erased erased erased erased
    after: CE0 block B page 20                 erased erased erased erased
    after: CE0 block B page 30                 erased erased erased erased
  erase CE0 block B again (already erased)     FAULT, NSTAT after D0h c0, ready c0,  5052 us
    before: CE0 block B+1 page 30              code ok code ok code ok code ok
  two-plane erase B, B+1                       FAULT, NSTAT after D0h c0, ready c0,  6016 us
    after: CE0 block B+1 page 30               erased erased erased erased
    before: CE1 block B page 20                code ok code ok code ok code ok
  erase CE1 block B                            FAULT, NSTAT after D0h c0, ready c0,  6015 us
    after: CE1 block B page 20                 erased erased erased erased
  erase B+2 with FMC_ANUM = 3                  END, ready c0,  1141 us (a 4th address byte: accepted / ignored?)
```

### 14.2.118 eccchunk2 (run 30)

ID: `D-eccchunk2-30`. Device run 30. Experiment: [eccchunk2](13-experiments.md#13390-eccchunk2).

```
decode of host-built chunks in parity slot 0 (slot = after the decode):
  clean                                            -> END   ECC_RESULT 00000000 ECC_DONE 35, data clean, slot clean
  meta 1 bit                                       -> END   ECC_RESULT 00010000 ECC_DONE 35, data clean, slot clean
  meta 3 bits                                      -> END   ECC_RESULT 00030000 ECC_DONE 35, data clean, slot clean
  parity 1 bit                                     -> END   ECC_RESULT 00010000 ECC_DONE 35, data clean, slot clean
  parity 8 bits                                    -> END   ECC_RESULT 00080000 ECC_DONE 35, data clean, slot clean
  data 4 + parity 4                                -> END   ECC_RESULT 00080000 ECC_DONE 35, data clean, slot clean
  data 5 + meta 3                                  -> END   ECC_RESULT 00080000 ECC_DONE 35, data clean, slot clean
  parity 9 bits                                    -> END   ECC_RESULT 00000011 ECC_DONE 3d, data clean, slot as sent
  meta byte order: built with the bytes reversed   -> END   ECC_RESULT 00040000 ECC_DONE 35, data clean, slot 56341245a321cc53d4692df9359caeb7
```

### 14.2.119 eccmode2 (run 30)

ID: `D-eccmode2-30`. Device run 30. Experiment: [eccmode2](13-experiments.md#13391-eccmode2).

```
ECC_MODE map (encode only; bit numbering of the fit sectors: bit 0 = MSB of byte 0):
  0x0000180: END ECC_DONE 33 ECC_RESULT 00040000 slot0 ffffff7a82a426c457e3a863a1cb34c8 area c834cba163a8e357c426a4827a0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
  0x0000380: END ECC_DONE 33 ECC_RESULT 00040000 slot0 ffffff7a82a426c457e3a863a1cb34c8 area c834cba163a8e357c426a4827a0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804  (= 0x180)
  0x0000580: END ECC_DONE 33 ECC_RESULT 00040000 slot0 ffffff7a82a426c457e3a863a1cb34c8 area c834cba163a8e357c426a4827a0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804  (= 0x180)
  0x0000980: END ECC_DONE 33 ECC_RESULT 00040000 slot0 ffffff7a82a426c457e3a863a1cb34c8 area c834cba163a8e357c426a4827a0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804  (= 0x180)
  0x0001180: END ECC_DONE 33 ECC_RESULT 00040000 slot0 ffffff7a82a426c457e3a863a1cb34c8 area c834cba163a8e357c426a4827a0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804  (= 0x180)
  0x0002180: END ECC_DONE 33 ECC_RESULT 00040000 slot0 ffffff7a82a426c457e3a863a1cb34c8 area c834cba163a8e357c426a4827a0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804  (= 0x180)
  0x0004180: END ECC_DONE 33 ECC_RESULT 00040000 slot0 ffffff7a82a426c457e3a863a1cb34c8 area c834cba163a8e357c426a4827a0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804  (= 0x180)
  0x0008180: END ECC_DONE 33 ECC_RESULT 00020000 slot0 522ee44c7cc113acebcdffffffffffff area cdebac13c17c4ce42e52c4ce92066e7b38caf046000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000008bc68bc6a7b1a7b1a7b1a7b1a7b1a7b13181884fc82804
  0x0010180: END ECC_DONE 33 ECC_RESULT 00040000 slot0 ffffff5f3e6308be82eb5ae2251dc2c4 area c4c21d25e25aeb82be08633e5f0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
  0x0020180: END ECC_DONE 33 ECC_RESULT 00040000 slot0 ffffffcb5bda54c8635dfc5aabb10a78 area 780ab1ab5afc5d63c854da5bcb0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
  0x0040180: END ECC_DONE 33 ECC_RESULT 00040000 slot0 ffffff33836b2964d0c95b1d4032d686 area 86d632401d5bc9d064296b83330000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
  0x0080180: END ECC_DONE 33 ECC_RESULT 00040000 slot0 ffffff098b982a1c282601651c2202c8 area c802221c650126281c2a988b090000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
  0x0100180: END ECC_DONE 33 ECC_RESULT 00040000 slot0 ffffff667998630065799e2769fc5125 area 2551fc69279e796500639879660000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
  0x0200180: END ECC_DONE 33 ECC_RESULT 00040000 slot0 ffffff7a82a426c457e3a863a1cb34c8 area c834cba163a8e357c426a4827a0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804  (= 0x180)
  0x0400180: END ECC_DONE 33 ECC_RESULT 00040000 slot0 ffffff7a82a426c457e3a863a1cb34c8 area c834cba163a8e357c426a4827a0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804  (= 0x180)
  0x0800180: END ECC_DONE 33 ECC_RESULT 00040000 slot0 ffffff7a82a426c457e3a863a1cb34c8 area c834cba163a8e357c426a4827a0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804  (= 0x180)
  0x1000180: END ECC_DONE 13 ECC_RESULT 00040000 slot0 ffffff7a82a426c457e3a863a1cb34c8 area c834cba163a8e357c426a4827a0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804  (= 0x180)
  0x0000000: END ECC_DONE 3b ECC_RESULT 00000011 slot0 ffffffffffffffffffffffffffffffff area 233c853f32c8ce55a1c321a1fc0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000f4f0f4f00000000000000000000000003181884fc82804
  0x0000080: END ECC_DONE 3b ECC_RESULT 00000011 slot0 ffffffffffffffffffffffffffffffff area 2e1c79573386228532df6ca2da0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000b85bb85b0000000000000000000000003181884fc82804
  0x0000100: END ECC_DONE 33 ECC_RESULT 00040000 slot0 ffffff7a82a426c457e3a863a1cb34c8 area c834cba163a8e357c426a4827a0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804  (= 0x180)
  0x0e00000: END ECC_DONE 1b ECC_RESULT 00000011 slot0 ffffffffffffffffffffffffffffffff area 7b55880c275b93c33a7204f8070000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000344734470000000000000000000000003181884fc82804
  0x1ffffff: END ECC_DONE 13 ECC_RESULT 00020000 slot0 509999fff94faba1ffff8124ffffffff area 8124f94faba1509999ff31982cb8c5549cad90d800000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000a7b1a7b1a7b1a7b1a7b1a7b1a7b1a7b13181884fc82804
    fit 0x0000000 zeros    slot0 ffffffffffffffffffffffffffffffff area 611570dd25c74b621d4b9efe4b0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000adf6adf60000000000000000000000003181884fc82804
    fit 0x0000000 bit 0    slot0 ffffffffffffffffffffffffffffffff area e87c56af4cfa3be376f15bb3e400000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000006d3d6d3d0000000000000000000000003181884fc82804
    fit 0x0000000 bit 1    slot0 ffffffffffffffffffffffffffffffff area dcfca60c49c28aa039836fbe7c0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000537353730000000000000000000000003181884fc82804
    fit 0x0000000 bit 8    slot0 ffffffffffffffffffffffffffffffff area edd42e2dda3f79ae9517f0dc37000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000032aa32aa0000000000000000000000003181884fc82804
    fit 0x0000000 bit 9    slot0 ffffffffffffffffffffffffffffffff area a62f0baf90c8082570138c3f120000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000d683d6830000000000000000000000003181884fc82804
    fit 0x0000000 bit 4095 slot0 ffffffffffffffffffffffffffffffff area c47f1becb633ef38283ab3fa840000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000b6b5b6b50000000000000000000000003181884fc82804
    fit 0x0000080 zeros    slot0 ffffffffffffffffffffffffffffffff area 528950e823dd0c2ee3334b3ccf00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000004a854a850000000000000000000000003181884fc82804
    fit 0x0000080 bit 0    slot0 ffffffffffffffffffffffffffffffff area b5d2b4ab2eacd0192881214ff700000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007fce7fce0000000000000000000000003181884fc82804
    fit 0x0000080 bit 1    slot0 ffffffffffffffffffffffffffffffff area 993f23a304711734be92d1c260000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000084ac84ac0000000000000000000000003181884fc82804
    fit 0x0000080 bit 8    slot0 ffffffffffffffffffffffffffffffff area 5788eb241e93af5b28f7042410000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000068ca68ca0000000000000000000000003181884fc82804
    fit 0x0000080 bit 9    slot0 ffffffffffffffffffffffffffffffff area 2ca900f00bae330c7f30f0c62a0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000fa52fa520000000000000000000000003181884fc82804
    fit 0x0000080 bit 4095 slot0 ffffffffffffffffffffffffffffffff area b0c691ff3e6a42d353fb5a0bcb0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000a873a8730000000000000000000000003181884fc82804
    fit 0x0e00000 zeros    slot0 ffffffffffffffffffffffffffffffff area 4684475615107a7cfd75b3cebe00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007dcb7dcb0000000000000000000000003181884fc82804
    fit 0x0e00000 bit 0    slot0 ffffffffffffffffffffffffffffffff area 5da0a18c3e1f8acedf03f2ac6800000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001be61be60000000000000000000000003181884fc82804
    fit 0x0e00000 bit 1    slot0 ffffffffffffffffffffffffffffffff area b1250ddf69ebfa90821e2127830000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000817881780000000000000000000000003181884fc82804
    fit 0x0e00000 bit 8    slot0 ffffffffffffffffffffffffffffffff area a49605af7f7209f9bc12b71a3a000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000016df16df0000000000000000000000003181884fc82804
    fit 0x0e00000 bit 9    slot0 ffffffffffffffffffffffffffffffff area 5f6698cd901dfb497dc805f4f80000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000536e536e0000000000000000000000003181884fc82804
    fit 0x0e00000 bit 4095 slot0 ffffffffffffffffffffffffffffffff area 56fb4de8fda80f967a303d769f00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000008f828f820000000000000000000000003181884fc82804
    fit 0x1ffffff zeros    slot0 23113931a4371b24ffffeaeaffffffff area eaeaa4371b242311393165bfd8a29bf63577759100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000fc3bfc3bfc3bfc3bfc3bfc3bfc3bfc3b3181884fc82804
    fit 0x1ffffff bit 0    slot0 2e4c25eee8c1cc9affff2e05ffffffff area 2e05e8c1cc9a2e4c25eed298a1d21422673a3ac000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000feecfeecfeecfeecfeecfeecfeecfeec3181884fc82804
    fit 0x1ffffff bit 1    slot0 b77f975e8aecf1fbffff088fffffffff area 088f8aecf1fbb77f975eba3ca41a5d9c9cd1f238000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000007d507d507d507d507d507d507d507d503181884fc82804
    fit 0x1ffffff bit 8    slot0 5eeff873720d7b17ffff76ceffffffff area 76ce720d7b175eeff873196046465acb673c40ac000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000e3b0e3b0e3b0e3b0e3b0e3b0e3b0e3b3181884fc82804
    fit 0x1ffffff bit 9    slot0 0dae59918f0a2bb9ffffa4eaffffffff area a4ea8f0a2bb90dae5991dbc2d7507aec9ed2ef8f00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000853b853b853b853b853b853b853b853b3181884fc82804
    fit 0x1ffffff bit 4095 slot0 23113931a4371b24ffffeaeaffffffff area eaeaa4371b242311393165bfd8a29bf63577759100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000fc3bfc3bfc3bfc3bfc3bfc3bfc3bfc3b3181884fc82804
    fit 0x0008180 zeros    slot0 00000000000000000000ffffffffffff area 00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000fc3bfc3bfc3bfc3bfc3bfc3b3181884fc82804
    fit 0x0008180 bit 0    slot0 86f49e603532a5c87af7ffffffffffff area f77ac8a53235609ef486282cc4f7ed491edc23b5000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000002f822f82fc3bfc3bfc3bfc3bfc3bfc3b3181884fc82804
    fit 0x0008180 bit 1    slot0 c27a07321e1952e43f7fffffffffffff area 7f3fe452191e32077ac21404eadbf7201faeb15a0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000015811581fc3bfc3bfc3bfc3bfc3bfc3b3181884fc82804
    fit 0x0008180 bit 8    slot0 0a235adfde97782d116fffffffffffff area 6f112d7897dedf5a230a0bf17d7b9207f7482dd50000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000f200f2fc3bfc3bfc3bfc3bfc3bfc3b3181884fc82804
    fit 0x0008180 bit 9    slot0 0531ed7defca9c168ab3ffffffffffff area b38a169ccaef7ded310581faf63dc8876bac966a0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000790079fc3bfc3bfc3bfc3bfc3bfc3b3181884fc82804
    fit 0x0008180 bit 4095 slot0 3ff0a6afee076a5a6590ffffffffffff area 90655a6a07eeafa6f03f000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000005800580fc3bfc3bfc3bfc3bfc3bfc3b3181884fc82804
    fit 0x0010180 zeros    slot0 ffffff9bbcef60c75438704874365886 area 8658367448703854c760efbc9b0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
    fit 0x0010180 bit 0    slot0 fffffff302601dbcad8eb6e8ffba1130 area 3011baffe8b68eadbc1d6002f30000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
    fit 0x0010180 bit 1    slot0 ffffffafe3a8de7aa8e3131831f07c5d area 5d7cf0311813e3a87adea8e3af0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
    fit 0x0010180 bit 8    slot0 ffffff9bbcef60c75438704874365886 area 8658367448703854c760efbc9b0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
    fit 0x0010180 bit 9    slot0 ffffff9bbcef60c75438704874365886 area 8658367448703854c760efbc9b0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
    fit 0x0010180 bit 4095 slot0 ffffff9bbcef60c75438704874365886 area 8658367448703854c760efbc9b0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
    fit 0x0020180 zeros    slot0 ffffffb8ce0bfebef248feaeba8e8166 area 66818ebaaefe48f2befe0bceb80000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
    fit 0x0020180 bit 0    slot0 ffffffd0708483c50bfe380e3102c8d0 area d0c802310e38fe0bc5838470d00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
    fit 0x0020180 bit 1    slot0 ffffff8c914c40030e939dfeff48a5bd area bda548fffe9d930e03404c918c0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
    fit 0x0020180 bit 8    slot0 ffffffeeac932e1a0c6cb6c2f4a3f5a3 area a3f5a3f4c2b66c0c1a2e93acee0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
    fit 0x0020180 bit 9    slot0 ffffff1903cde6d10b5319387f7a4695 area 95467a7f3819530bd1e6cd03190000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
    fit 0x0020180 bit 4095 slot0 ffffffb8ce0bfebef248feaeba8e8166 area 66818ebaaefe48f2befe0bceb80000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003181884fc82804
```

### 14.2.120 tmobits2 (run 30)

ID: `D-tmobits2-30`. Device run 30. Experiment: [tmobits2](13-experiments.md#13392-tmobits2).

```
timeout length, bits 7:4 = 5..7:
  start 0xff55 -> FAULT  fault at   2432 us (n = 5; 2**11 per step would be 4855 us)
  start 0xff65 -> FAULT  fault at   4868 us (n = 6; 2**11 per step would be 4855 us)
  start 0xff75 -> FAULT  fault at   4868 us (n = 7; 2**11 per step would be 4855 us)
  start 0x0f55 -> FAULT  fault at    152 us (n = 5; 2**11 per step would be 303 us)
  start 0x0f75 -> FAULT  fault at    310 us (n = 7; 2**11 per step would be 303 us)
```

### 14.2.121 ev5ready (run 30)

ID: `D-ev5ready-30`. Device run 30. Experiment: [ev5ready](13-experiments.md#13393-ev5ready).

```
status latch event (5) against chip activity (timeouts ~4.87 ms each):
  idle, after a soft reset: 1 latch                END,    17 us, 0 timeout(s)
      latch 0: STATUS 008c3000 NSTAT c0 -> after the wait 008c3000
  idle, no soft reset: 1 latch                     END,    18 us, 0 timeout(s)
      latch 0: STATUS 008c3000 NSTAT c0 -> after the wait 008c3000
  idle, no soft reset: 1 latch (again)             END,    17 us, 0 timeout(s)
      latch 0: STATUS 008c3000 NSTAT c0 -> after the wait 008c3000
  idle: 3 latches                                  FAULT,  9739 us, 2 timeout(s)
      latch 0: STATUS 008c3000 NSTAT c0 -> after the wait 008c3000
      latch 1: STATUS 000c3000 NSTAT c0 -> after the wait 000c3000
      latch 2: STATUS 000c3000 NSTAT c0 -> after the wait 000c3000
  FFh reset, then 2 latches                        FAULT,  4884 us, 1 timeout(s)
      latch 0: STATUS 008c3000 NSTAT c0 -> after the wait 008c3000
      latch 1: STATUS 000c3000 NSTAT c0 -> after the wait 000c3000
  erase, then 3 latches                            FAULT, 10883 us, 2 timeout(s)
      latch 0: STATUS 000c3000 NSTAT 81 -> after the wait 008c3000
      latch 1: STATUS 000c3000 NSTAT c0 -> after the wait 000c3000
      latch 2: STATUS 000c3000 NSTAT c0 -> after the wait 000c3000
  erase, NSTAT polled without waits, then 1 latch  END,   183 us, 0 timeout(s)
      latch 0: STATUS 008c3000 NSTAT c0 -> after the wait 008c3000
  read page (00h .. 30h), then 2 latches           FAULT,  4911 us, 1 timeout(s)
      latch 0: STATUS 000c3000 NSTAT 80 -> after the wait 008c3000
      latch 1: STATUS 000c3000 NSTAT c0 -> after the wait 000c3000
```

### 14.2.122 pageorder (run 30)

ID: `D-pageorder-30`. Device run 30. Experiment: [pageorder](13-experiments.md#13394-pageorder).

```
page program order (scratch blocks), read back raw:
  (a) block 3008 pages [0, 1, 2, 3, 4, 5, 6, 7]: END, 1872 us; page 0 ok (NSTAT 70), page 1 ERASED (NSTAT 70), page 2 ok (NSTAT 70), page 3 ERASED (NSTAT 70), page 4 ok (NSTAT 70), page 5 ERASED (NSTAT 70), page 6 ERASED (NSTAT 70), page 7 ERASED (NSTAT 70)
  (b) block 3009 pages [5]: END, 249 us; page 5 ok (NSTAT 70)
  (c) block 3010 pages [4]: END, 250 us; page 4 ok (NSTAT 70)
  (d) block 3011 pages [4, 5]: END, 473 us; page 4 ok (NSTAT 70), page 5 ERASED (NSTAT 70)
  (e) block 3012 pages [1, 3]: END, 474 us; page 1 ok (NSTAT 70), page 3 ERASED (NSTAT 70)
  (f) erase with 4 address cycles: END, 1141 us; page 0 afterwards: erased erased erased erased
```

### 14.2.123 op09w33 (run 30)

ID: `D-op09w33-30`. Device run 30. Experiment: [op09w33](13-experiments.md#13395-op09w33).

```
wait2, idle chip (no other waits in the program):
  wait2  33 (0b00100001): passes, fault at 0 us
  wait2   5 (0b00000101): passes, fault at 0 us
  wait2   0 (0b00000000): times out, fault at 4868 us
  wait2   1 (0b00000001): times out, fault at 4868 us
  wait2   4 (0b00000100): times out, fault at 4868 us
  wait2  32 (0b00100000): times out, fault at 4868 us
  wait2  37 (0b00100101): times out, fault at 4868 us
  wait2  65 (0b01000001): times out, fault at 4868 us
  wait2  69 (0b01000101): times out, fault at 4868 us
  wait2  97 (0b01100001): times out, fault at 4854 us
  wait2 133 (0b10000101): times out, fault at 4856 us
  wait2 161 (0b10100001): times out, fault at 4868 us
  wait2 165 (0b10100101): times out, fault at 4868 us
```

### 14.2.124 ev5arm (run 31)

ID: `D-ev5arm-31`. Device run 31. Experiment: [ev5arm](13-experiments.md#13396-ev5arm).

```
what re-arms event 5 inside a program (each line: latch, X, latch; a timeout ~4.87 ms):
  latch, nothing,                     latch: second wait timed out ( 4870 us); STATUS after latch 1 000c3000, NSTAT c0
  latch, set-up word,                 latch: second wait PASSED (   17 us); STATUS after latch 1 008c3000, NSTAT c0
  latch, status-clear word,           latch: second wait timed out ( 4896 us); STATUS after latch 1 000c3000, NSTAT c0
  latch, set-up + status-clear,       latch: second wait PASSED (   17 us); STATUS after latch 1 008c3000, NSTAT c0
  latch, CE deselect / reselect,      latch: second wait timed out ( 4896 us); STATUS after latch 1 000c3000, NSTAT c0
  latch, FMCTRL1 0x100e0,             latch: second wait PASSED (   17 us); STATUS after latch 1 008c3000, NSTAT c0
  latch, 16-byte read from the chip,  latch: second wait PASSED (   18 us); STATUS after latch 1 008c3000, NSTAT c0
  latch, ECC kick,                    latch: second wait timed out ( 4905 us); STATUS after latch 1 000c3000, NSTAT c0
  latch, DMA,                         latch: second wait PASSED (   18 us); STATUS after latch 1 008c3000, NSTAT c0
  latch with DATA_SEL 0x100, then a normal latch: 4883 us; latch 0 STATUS 008c3000 NSTAT 70, latch 1 STATUS 000c3000 NSTAT c0
```

### 14.2.125 writeseq2 (run 31)

ID: `D-writeseq2-31`. Device run 31. Experiment: [writeseq2](13-experiments.md#13397-writeseq2).

```
multi-page program runs with the status poll on FMC_DATA_SEL = 1 (ready byte; b0 = FAIL):
  8 pages in order                 END,  5827 us: page 0 ok (ready c0, just after 10h 81), page 1 ok (ready c0, just after 10h 81), page 2 ok (ready c0, just after 10h 81), page 3 ok (ready c0, just after 10h 81), page 4 ok (ready c0, just after 10h 81), page 5 ok (ready c0, just after 10h 81), page 6 ok (ready c0, just after 10h 81), page 7 ok (ready c0, just after 10h 81)
  pages 9, 11                      END,  2019 us: page 9 ok (ready c0), page 11 ok (ready c0)
  page 0 again without an erase    END,   464 us: page 0 DIFFERENT (ready c0, just after 10h 81)
  pages 1, 3 of an empty block     END,   919 us: page 1 ok (ready c0), page 3 ok (ready c0)
```

### 14.2.126 eccmode3 (run 31)

ID: `D-eccmode3-31`. Device run 31. Experiment: [eccmode3](13-experiments.md#13398-eccmode3).

```
ECC_MODE length field and samples:
  b20:16 =  3: encodes the first 3 bytes
  b20:16 =  5: encodes the first 5 bytes
  b20:16 = 12: encodes the first 12 bytes
  b20:16 = 31: encodes the first 31 bytes
  b20:16 = 1 plus 0x0200000: encodes the first 1 bytes
  b20:16 = 1 plus 0x0400000: encodes the first 1 bytes
  b20:16 = 1 plus 0x1000000: encodes the first 1 bytes
  b20:16 = 1 plus 0x03f0000: encodes the first 31 bytes
  decode with b20:16 = 16, byte 100 changed (8 bits): ECC_RESULT 00000000 ECC_DONE 35
  decode with b20:16 = 16, byte 5 changed (8 bits): ECC_RESULT 00080000 ECC_DONE 35
  parity samples for 0x8180: zeros, bits 0..15, 4080..4095
  parity samples for 0x0: zeros, bits 0..15, 4080..4095
  parity samples for 0x80: zeros, bits 0..15, 4080..4095
```

### 14.2.127 ev5two (run 32)

ID: `D-ev5two-32`. Device run 32. Experiment: [ev5two](13-experiments.md#13399-ev5two).

```
event 5 and a second chip (each poll waits on event 5):
  program CE0 + CE1, poll CE0, poll CE1    END,   350 us (0 timeout(s)); poll 1 NSTAT 81 -> c0, poll 2 NSTAT 81 -> c0
  program CE0 + CE1, poll CE1, poll CE0    FAULT,  5224 us (1 timeout(s)); poll 1 NSTAT 81 -> c0, poll 2 NSTAT c0 -> c0
  idle: latch, address cycles, latch       END,    17 us (0 timeout(s)); poll 1 NSTAT c0 -> c0, poll 2 NSTAT c0 -> c0
```

### 14.2.128 b15ecc (run 32)

ID: `D-b15ecc-32`. Device run 32. Experiment: [b15ecc](13-experiments.md#133100-b15ecc).

```
ECC_MODE b15 code, strength (encode and decode in one program) and samples:
  0 bits, one per symbol         -> END   ECC_RESULT 00000000 ECC_DONE 35, data corrected
  1 bits, one per symbol         -> END   ECC_RESULT 00010000 ECC_DONE 35, data corrected
  2 bits, one per symbol         -> END   ECC_RESULT 00020000 ECC_DONE 35, data corrected
  4 bits, one per symbol         -> END   ECC_RESULT 00040000 ECC_DONE 35, data corrected
  8 bits, one per symbol         -> END   ECC_RESULT 00000011 ECC_DONE 3d, data as sent
  9 bits, one per symbol         -> END   ECC_RESULT 00000011 ECC_DONE 3d, data as sent
  10 bits, one per symbol        -> END   ECC_RESULT 00000011 ECC_DONE 3d, data as sent
  12 bits, one per symbol        -> END   ECC_RESULT 00000011 ECC_DONE 3d, data as sent
  16 bits, one per symbol        -> END   ECC_RESULT 00000011 ECC_DONE 3d, data as sent
  17 bits, one per symbol        -> END   ECC_RESULT 00000011 ECC_DONE 3d, data as sent
  burst: 10 bits in 1 symbol     -> END   ECC_RESULT 00020000 ECC_DONE 35, data corrected
  burst: 40 bits in 4 symbols    -> END   ECC_RESULT 00000011 ECC_DONE 3d, data as sent
  burst: 80 bits in 8 symbols    -> END   ECC_RESULT 00000011 ECC_DONE 3d, data as sent
  burst: 90 bits in 9 symbols    -> END   ECC_RESULT 00000011 ECC_DONE 3d, data as sent
  parity samples: one bit at bytes 0..511 (bit 0 and 7), and two bits together
```

### 14.2.129 eccmode4 (run 33)

ID: `D-eccmode4-33`. Device run 33. Experiment: [eccmode4](13-experiments.md#133101-eccmode4).

```
ECC_MODE leftovers (area = 0x900..0x913 after the run):
  0x0 ramp, after a soft reset (1)   ECC_DONE 3b area 233c853f32c8ce55a1c321a1fc00000000000000 0x990 f0f4f0f4
  0x0 ramp, after a soft reset (2)   ECC_DONE 3b area 2e1c79573386228532df6ca2da00000000000000 0x990 5bb85bb8
  0x0 ramp, no reset (3)             ECC_DONE 3b area 7b55880c275b93c33a7204f80700000000000000 0x990 47344734
  0x0 ramp, no reset (4)             ECC_DONE 3b area 611570dd25c74b621d4b9efe4b00000000000000 0x990 f6adf6ad
  0x0 zeros                          ECC_DONE 3b area e87c56af4cfa3be376f15bb3e400000000000000 0x990 3d6d3d6d
  0x0 bit 0                          ECC_DONE 3b area dcfca60c49c28aa039836fbe7c00000000000000 0x990 73537353
  0x0 bit 1                          ECC_DONE 3b area edd42e2dda3f79ae9517f0dc3700000000000000 0x990 aa32aa32
  0x0 bits 0 + 1                     ECC_DONE 3b area a62f0baf90c8082570138c3f1200000000000000 0x990 83d683d6
  0x0 ramp after a BCH decode        ECC_DONE 3b area c47f1becb633ef38283ab3fa8400000000000000 0x990 b5b6b5b6
  0x0 ramp after a BCH encode        ECC_DONE 3b area 528950e823dd0c2ee3334b3ccf00000000000000 0x990 854a854a
  0x80 ramp                          ECC_DONE 3b area b5d2b4ab2eacd0192881214ff700000000000000 0x990 ce7fce7f
  0x80 ramp, then a decode           ECC_DONE 3b area 993f23a304711734be92d1c26000000000000000 0x990 ca68ca68
  0xe00000 ramp                      ECC_DONE 1b area 5788eb241e93af5b28f704241000000000000000 0x990 ca68ca68
  0x8180 ramp, encode only           ECC_DONE 3b area cdebac13c17c4ce42e52c4ce92066e7b38caf046 0x990 c68bc68b
  0x8180 ramp, then a decode         ECC_DONE 3b area cdebac13c17c4ce42e52c4ce92066e7b38caf046 0x990 c68bc68b
  0x8180 zeros                       ECC_DONE 33 area 0000000000000000000000000000000000000000 0x990 00000000
  mode 0x01f8100 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x01f8100 ramp  area 70eb4fd4a29b8a816cfb7318a03a023bd1f0a127
  mode 0x01f8101 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x01f8101 ramp  area 70eb4fd4a29b8a816cfb7318a03a023bd1f0a127
  mode 0x01f8102 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x01f8102 ramp  area 70eb4fd4a29b8a816cfb7318a03a023bd1f0a127
  mode 0x01f8104 zeros area c44e5623898b4ca58d3c9981945cce71fcc564a0
  mode 0x01f8104 ramp  area dbed7a741a9b52038eddd591b20d29ef46914fc1
  mode 0x01f8108 zeros area 1b8814333def7d5c00aec54b302f429dcb9900e1
  mode 0x01f8108 ramp  area 7c4f08935a9d1648df6737804b2631a9f38e5f31
  mode 0x01f8110 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x01f8110 ramp  area 8124f94faba1509999ff31982cb8c5549cad90d8
  mode 0x01f8120 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x01f8120 ramp  area 70eb4fd4a29b8a816cfb7318a03a023bd1f0a127
  mode 0x01f8140 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x01f8140 ramp  area 70eb4fd4a29b8a816cfb7318a03a023bd1f0a127
  mode 0x01f8180 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x01f8180 ramp  area 70eb4fd4a29b8a816cfb7318a03a023bd1f0a127
  mode 0x01f8300 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x01f8300 ramp  area 70eb4fd4a29b8a816cfb7318a03a023bd1f0a127
  mode 0x01f8500 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x01f8500 ramp  area 70eb4fd4a29b8a816cfb7318a03a023bd1f0a127
  mode 0x01f8900 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x01f8900 ramp  area 70eb4fd4a29b8a816cfb7318a03a023bd1f0a127
  mode 0x01f9100 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x01f9100 ramp  area 70eb4fd4a29b8a816cfb7318a03a023bd1f0a127
  mode 0x01fa100 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x01fa100 ramp  area 70eb4fd4a29b8a816cfb7318a03a023bd1f0a127
  mode 0x01fc100 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x01fc100 ramp  area 70eb4fd4a29b8a816cfb7318a03a023bd1f0a127
  mode 0x03f8100 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x03f8100 ramp  area 70eb4fd4a29b8a816cfb7318a03a023bd1f0a127
  mode 0x05f8100 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x05f8100 ramp  area 70eb4fd4a29b8a816cfb7318a03a023bd1f0a127
  mode 0x09f8100 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x09f8100 ramp  area 70eb4fd4a29b8a816cfb7318a03a023bd1f0a127
  mode 0x11f8100 zeros area eaeaa4371b242311393165bfd8a29bf635777591
  mode 0x11f8100 ramp  area 70eb4fd4a29b8a816cfb7318a03a023bd1f0a127
```

### 14.2.130 finale (run 34)

ID: `D-finale-34`. Device run 34. Experiment: [finale](13-experiments.md#133102-finale).

```
final open points:
(1) ECC_MODE b1..b5:
  0x00182: zeros same, ramp same; ECC_DONE 3b; slot (ramp) ffffff7a82a426c457e3a863a1cb34c8
  0x00184: zeros same, ramp same; ECC_DONE 3b; slot (ramp) ffffff7a82a426c457e3a863a1cb34c8
  0x00188: zeros same, ramp same; ECC_DONE 3b; slot (ramp) ffffff7a82a426c457e3a863a1cb34c8
  0x00190: zeros same, ramp CHANGED; ECC_DONE 3b; slot (ramp) 9dffffff58f7b2ed3b029fe7ed47cadd
  0x001a0: zeros same, ramp same; ECC_DONE 3b; slot (ramp) ffffff7a82a426c457e3a863a1cb34c8
  0x08102: zeros same, ramp same; ECC_DONE 33; slot (ramp) 522ee44c7cc113acebcdebcdffffffff
  0x08104: zeros same, ramp CHANGED; ECC_DONE 33; slot (ramp) 186d249847a87441d927d4965ed6e4ff
  0x08108: zeros same, ramp CHANGED; ECC_DONE 33; slot (ramp) 4e2923086f4cfc4b7b77718291507950
  0x08110: zeros same, ramp CHANGED; ECC_DONE 33; slot (ramp) bc3140a699f933afffff79d7ffffffff
  0x08120: zeros same, ramp same; ECC_DONE 33; slot (ramp) 522ee44c7cc113acebcdffffffffffff
(2) END; first values: 2ca900f00bae330c7f30f0c62a b0c691ff3e6a42d353fb5a0bcb 4684475615107a7cfd75b3cebe ... (48 values)
(3) FMC_28 change over 32 repetitions of:
  nothing (32 x movi)            0 (before 000 after 000)
  sti to RAM                     0 (before 000 after 000)
  ldi from RAM                   0 (before 000 after 000)
  wr to the parameter RAM        0 (before 000 after 000)
  DMA RAM -> buffer              0 (before 000 after 000)
  ECC encode                     0 (before 000 after 000)
  16-byte chip read (CE0)        0 (before 000 after 000)
(4) a 2112-byte transfer from the chip with 0x1cc / 0x1d0 changed (r2 = sequencer time stamps):
  default            END,   220 us
  0x1cc = 0          END,   220 us
  0x1cc = 0xffffff   END,   221 us
  0x1d0 = 0          END,   220 us
  0x1d0 = 0xffffff   END,   220 us
(5) VIC0 line 20 / VIC1 line 22 per FMC interrupt source (0x044 enable bit, action):
  command  0x044 bit  0: CS_IRQ 11, VIC0 line 20 SET, VIC1 line 22 clear
  command  0x044 bit  1: CS_IRQ 01, VIC0 line 20 clear, VIC1 line 22 clear
  command  0x044 bit  2: CS_IRQ 01, VIC0 line 20 clear, VIC1 line 22 clear
  address  0x044 bit  0: CS_IRQ 01, VIC0 line 20 clear, VIC1 line 22 clear
  address  0x044 bit  1: CS_IRQ 11, VIC0 line 20 SET, VIC1 line 22 clear
  status   0x044 bit  0: CS_IRQ 11, VIC0 line 20 clear, VIC1 line 22 clear
  status   0x044 bit 13: CS_IRQ 01, VIC0 line 20 clear, VIC1 line 22 clear
  DMA      0x044 bit 11: CS_IRQ 11, VIC0 line 20 clear, VIC1 line 22 clear
(6) wait2 during ECC encode in flight: passes for 5 33
(6) wait2 during DMA in flight: passes for 5 33
```

### 14.2.131 startdiff (run 35)

ID: `D-startdiff-35`. Device run 35. Experiment: [startdiff](13-experiments.md#133103-startdiff).

```
the same programs under start words differing only in bit 6 (differences only; drifting words FMC_28, 0x050, 0x9a4 left out):
  end only                                         0xfff5 vs 0xffb5: window after (a run changed it) 428 000001f8/000000e8, 450 00000000/00001000, 628 00000074/00000164, 650 00000800/00001000
  end only                                         0xffd5 vs 0xff95: window after (a run changed it) 428 000001d8/000000c8, 450 00001800/00002800, 628 00000054/00000144, 650 00002000/00002800
  sig, end                                         0xfff5 vs 0xffb5: window after (a run changed it) 428 000001b8/000000a8, 450 00003000/00000000, 628 00000034/00000124, 650 00003800/00000000
  sig, end                                         0xffd5 vs 0xff95: window after (a run changed it) 428 00000198/00000088, 450 00000800/00001800, 628 00000014/00000104, 650 00001000/00001800
  loop 2000                                        0xfff5 vs 0xffb5: window after (a run changed it) 428 00000178/00000068, 450 00002000/00003000, 628 000001f4/000000e4, 650 00002000/00003000
  loop 2000                                        0xffd5 vs 0xff95: window after (a run changed it) 428 00000158/00000048, 450 00003800/00000800, 628 000001d4/000000c4, 650 00003800/00000800
  wait 10 (never), C58 0                           0xfff5 vs 0xffb5: window after (a run changed it) 428 00000138/00000028, 450 00001000/00002000, 628 000001b4/000000a4, 650 00001000/00002000
  wait 10 (never), C58 0                           0xffd5 vs 0xff95: window after (a run changed it) 428 00000118/00000008, 450 00002800/00003800, 628 00000194/00000084, 650 00002800/00003800
  wait 10 (never), C58 4                           0xfff5 vs 0xffb5: window after (a run changed it) 428 000000f8/000001e8, 628 00000174/00000064, 650 00000000/00001000
  wait 10 (never), C58 4                           0xffd5 vs 0xff95: window after (a run changed it) 428 000000d8/000001c8, 450 00001800/00002000, 628 00000154/00000044, 650 00001800/00002800
  illegal 0x30, C58 4                              0xfff5 vs 0xffb5: window after (a run changed it) 428 000000b8/000001a8, 450 00003000/00003800, 628 00000134/00000024, 650 00003000/00000000
  illegal 0x30, C58 4                              0xffd5 vs 0xff95: window after (a run changed it) 428 00000098/00000188, 450 00000800/00001000, 628 00000114/00000004, 650 00000800/00001800
  evjmp, nothing pending                           0xfff5 vs 0xffb5: window after (a run changed it) 428 00000078/00000168, 450 00002000/00002800, 628 000000f4/000001e4, 650 00002000/00002800
  evjmp, nothing pending                           0xffd5 vs 0xff95: window after (a run changed it) 428 00000058/00000148, 450 00003800/00000000, 628 000000d4/000001c4, 650 00003800/00000000
  tmwait                                           0xfff5 vs 0xffb5: time 4857 / 18 us; window after (a run changed it) 428 00000038/00000128, 450 00001000/00001800, 628 000000b4/000001a4, 650 00001000/00001800, c14 00000000/000000ff
  tmwait                                           0xffd5 vs 0xff95: time 4856 / 17 us; window after (a run changed it) 428 00000018/00000108, 450 00002800/00003000, 628 00000094/00000184, 650 00002800/00003000, c14 00000000/000000ff
  DMA in + ECC encode                              0xfff5 vs 0xffb5: window after (a run changed it) 428 000001f8/000000e8, 450 00003800/00000800, 628 00000074/00000164, 650 00000000/00000800
  DMA in + ECC encode                              0xffd5 vs 0xff95: window after (a run changed it) 428 000001d8/000000c8, 450 00001000/00002000, 628 00000054/00000144, 650 00001800/00002000
  status latch, wait 5                             0xfff5 vs 0xffb5: window after (a run changed it) 428 000001b8/000000a8, 450 00002800/00003800, 628 00000034/00000124, 650 00003000/00003800
  status latch, wait 5                             0xffd5 vs 0xff95: window after (a run changed it) 428 00000198/00000088, 450 00000000/00001000, 628 00000014/00000104, 650 00000800/00001000
  reserved: sequencer reads                        non-zero reserved words read by the sequencer (0xfff5): 3a8=ffffffff, 3ac=ffffffff, 3b0=ffffffff, 3b4=ffffffff, 100=ffffffff, 104=ffffffff, 108=ffffffff, 10c=ffffffff, 110=ffffffff, 114=ffffffff, 118=ffffffff, 11c=ffffffff, 120=ffffffff, 124=ffffffff, 128=ffffffff, 12c=ffffffff
  reserved: sequencer reads                        0xfff5 vs 0xffb5: window after (a run changed it) 428 00000178/00000068, 450 00001800/00002800, 628 000001f4/000000e4, 650 00001800/00002800
  reserved: sequencer reads                        0xffd5 vs 0xff95: window after (a run changed it) 428 00000158/00000048, 450 00003000/00000000, 628 000001d4/000000c4, 650 00003000/00000000
  reserved: sequencer writes all ones, reads back  non-zero reserved words read by the sequencer (0xfff5): 3a8=ffffffff, 3ac=ffffffff, 3b0=ffffffff, 3b4=ffffffff, 100=ffffffff, 104=ffffffff, 108=ffffffff, 10c=ffffffff, 110=ffffffff, 114=ffffffff, 118=ffffffff, 11c=ffffffff, 120=ffffffff, 124=ffffffff, 128=ffffffff, 12c=ffffffff
  reserved: sequencer writes all ones, reads back  0xfff5 vs 0xffb5: window after (a run changed it) 428 00000138/00000028, 450 00000800/00001800, 628 000001b4/000000a4, 650 00000800/00001800
  reserved: sequencer writes all ones, reads back  0xffd5 vs 0xff95: window after (a run changed it) 428 00000118/00000008, 450 00002000/00003000, 628 00000194/00000084, 650 00002000/00003000
```

### 14.2.132 startbits (run 35)

ID: `D-startbits-35`. Device run 35. Experiment: [startbits](13-experiments.md#133104-startbits).

```
start-word bits flipped one at a time from 0xfff5 (differences from the baseline only):
  baseline 0xfff5 x3, run time spread (us): loop, SDRAM 811..828, loop, SRAM 497..498, 2000 ldi 1636..1641, 2000 sti 1511..1514, 2000 pld 1305..1317, sig, end 17..17, DMA + ECC encode 18..19, fault, C58 4 (vector) 4870..4883
  bit  4 (0x0000ffe5): loop, SDRAM: window after (a run changed it) 428 000001f8/00000078, 450 00003800/00001800, 628 00000074/000000f4, 650 00000000/00001800 | loop, SRAM: window after (a run changed it) 428 000000e8/00000168, 450 00000800/00002000, 628 00000164/000001e4 | 2000 ldi: window after (a run changed it) 428 000001d8/00000058, 450 00001000/00003000, 628 00000054/000000d4, 650 00001800/00003000 | 2000 sti: window after (a run changed it) 428 000000c8/00000148, 450 00002000/00003800, 628 00000144/000001c4 | 2000 pld: window after (a run changed it) 428 000001b8/00000038, 450 00002800/00000800, 628 00000034/000000b4, 650 00003000/00000800 | sig, end: window after (a run changed it) 428 000000a8/00000128, 450 00003800/00001000, 628 00000124/000001a4 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000018, 450 00000000/00002000, 628 00000014/00000094, 650 00000800/00002000
  bit  5 (0x0000ffd5): loop, SDRAM: window after (a run changed it) 650 00000000/00003800 | loop, SRAM: window after (a run changed it) 450 00000800/00000000 | 2000 ldi: window after (a run changed it) 650 00001800/00001000 | 2000 sti: window after (a run changed it) 450 00002000/00001800 | 2000 pld: window after (a run changed it) 650 00003000/00002800 | sig, end: window after (a run changed it) 450 00003800/00003000 | DMA + ECC encode: window after (a run changed it) 650 00000800/00000000
  bit  6 (0x0000ffb5): loop, SDRAM: window after (a run changed it) 428 000001f8/00000178, 628 00000074/000001f4, 650 00000000/00001000 | loop, SRAM: window after (a run changed it) 428 000000e8/00000068, 450 00000800/00002000, 628 00000164/000000e4, 650 00000800/00002000 | 2000 ldi: window after (a run changed it) 428 000001d8/00000158, 628 00000054/000001d4, 650 00001800/00002800 | 2000 sti: window after (a run changed it) 428 000000c8/00000048, 450 00002000/00003800, 628 00000144/000000c4, 650 00002000/00003800 | 2000 pld: time 1478 us (base 1305..1317); window after (a run changed it) 428 000001b8/00000138, 628 00000034/000001b4, 650 00003000/00000000 | sig, end: window after (a run changed it) 428 000000a8/00000028, 450 00003800/00001000, 628 00000124/000000a4, 650 00003800/00001000 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000118, 628 00000014/00000194, 650 00000800/00001800
  bit  7 (0x0000ff75): loop, SDRAM: window after (a run changed it) 428 000001f8/000000f8, 628 00000074/00000174, 650 00000000/00003000 | loop, SRAM: window after (a run changed it) 428 000000e8/000001e8, 450 00000800/00003800, 628 00000164/00000064, 650 00000800/00000000 | 2000 ldi: window after (a run changed it) 428 000001d8/000000d8, 450 00001000/00000800, 628 00000054/00000154, 650 00001800/00000800 | 2000 sti: window after (a run changed it) 428 000000c8/000001c8, 450 00002000/00001000, 628 00000144/00000044, 650 00002000/00001800 | 2000 pld: window after (a run changed it) 428 000001b8/000000b8, 450 00002800/00002000, 628 00000034/00000134, 650 00003000/00002000 | sig, end: window after (a run changed it) 428 000000a8/000001a8, 450 00003800/00002800, 628 00000124/00000024, 650 00003800/00003000 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000098, 450 00000000/00003800, 628 00000014/00000114, 650 00000800/00003800
  bit  8 (0x0000fef5): loop, SDRAM: window after (a run changed it) 428 000001f8/00000078, 450 00003800/00001000, 628 00000074/000000f4, 650 00000000/00001000 | loop, SRAM: window after (a run changed it) 428 000000e8/00000168, 450 00000800/00001800, 628 00000164/000001e4 | 2000 ldi: window after (a run changed it) 428 000001d8/00000058, 450 00001000/00002800, 628 00000054/000000d4, 650 00001800/00002800 | 2000 sti: window after (a run changed it) 428 000000c8/00000148, 450 00002000/00003000, 628 00000144/000001c4 | 2000 pld: window after (a run changed it) 428 000001b8/00000038, 450 00002800/00000000, 628 00000034/000000b4, 650 00003000/00000000 | sig, end: window after (a run changed it) 428 000000a8/00000128, 450 00003800/00000800, 628 00000124/000001a4 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000018, 450 00000000/00001800, 628 00000014/00000094, 650 00000800/00001800, c14 000000ff/000000fe
  bit  9 (0x0000fdf5): loop, SDRAM: window after (a run changed it) 650 00000000/00003000 | loop, SRAM: window after (a run changed it) 450 00000800/00003800 | 2000 ldi: window after (a run changed it) 650 00001800/00000800 | 2000 sti: window after (a run changed it) 450 00002000/00001000 | 2000 pld: window after (a run changed it) 650 00003000/00002000 | sig, end: window after (a run changed it) 450 00003800/00002800 | DMA + ECC encode: window after (a run changed it) 650 00000800/00003800, c14 000000ff/000000fd
  bit 10 (0x0000fbf5): loop, SDRAM: window after (a run changed it) 428 000001f8/00000178, 628 00000074/000001f4, 650 00000000/00000800 | loop, SRAM: window after (a run changed it) 428 000000e8/00000068, 450 00000800/00001800, 628 00000164/000000e4, 650 00000800/00001800 | 2000 ldi: window after (a run changed it) 428 000001d8/00000158, 628 00000054/000001d4, 650 00001800/00002000 | 2000 sti: window after (a run changed it) 428 000000c8/00000048, 450 00002000/00003000, 628 00000144/000000c4, 650 00002000/00003000 | 2000 pld: window after (a run changed it) 428 000001b8/00000138, 628 00000034/000001b4, 650 00003000/00003800 | sig, end: window after (a run changed it) 428 000000a8/00000028, 450 00003800/00000800, 628 00000124/000000a4, 650 00003800/00000800 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000118, 628 00000014/00000194, 650 00000800/00001000, c14 000000ff/000000fb | fault, C58 4 (vector): fault at 4776 us (base 4854)
  bit 11 (0x0000f7f5): loop, SDRAM: window after (a run changed it) 428 000001f8/000000f8, 628 00000074/00000174, 650 00000000/00002800 | loop, SRAM: window after (a run changed it) 428 000000e8/000001e8, 450 00000800/00003000, 628 00000164/00000064, 650 00000800/00003800 | 2000 ldi: time 1969 us (base 1636..1641); window after (a run changed it) 428 000001d8/000000d8, 450 00001000/00000000, 628 00000054/00000154, 650 00001800/00000000 | 2000 sti: window after (a run changed it) 428 000000c8/000001c8, 450 00002000/00000800, 628 00000144/00000044, 650 00002000/00001000 | 2000 pld: window after (a run changed it) 428 000001b8/000000b8, 450 00002800/00001800, 628 00000034/00000134, 650 00003000/00001800 | sig, end: window after (a run changed it) 428 000000a8/000001a8, 450 00003800/00002000, 628 00000124/00000024, 650 00003800/00002800 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000098, 450 00000000/00003000, 628 00000014/00000114, 650 00000800/00003000, c14 000000ff/000000f7 | fault, C58 4 (vector): fault at 4710 us (base 4854)
  bit 12 (0x0000eff5): loop, SDRAM: window after (a run changed it) 428 000001f8/00000078, 450 00003800/00000800, 628 00000074/000000f4, 650 00000000/00000800 | loop, SRAM: window after (a run changed it) 428 000000e8/00000168, 450 00000800/00001000, 628 00000164/000001e4 | 2000 ldi: window after (a run changed it) 428 000001d8/00000058, 450 00001000/00002000, 628 00000054/000000d4, 650 00001800/00002000 | 2000 sti: window after (a run changed it) 428 000000c8/00000148, 450 00002000/00002800, 628 00000144/000001c4 | 2000 pld: window after (a run changed it) 428 000001b8/00000038, 450 00002800/00003800, 628 00000034/000000b4, 650 00003000/00003800 | sig, end: window after (a run changed it) 428 000000a8/00000128, 450 00003800/00000000, 628 00000124/000001a4 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000018, 450 00000000/00001000, 628 00000014/00000094, 650 00000800/00001000, c14 000000ff/000000ef | fault, C58 4 (vector): fault at 4552 us (base 4854)
  bit 13 (0x0000dff5): loop, SDRAM: window after (a run changed it) 650 00000000/00002800 | loop, SRAM: window after (a run changed it) 450 00000800/00003000 | 2000 ldi: window after (a run changed it) 650 00001800/00000000 | 2000 sti: window after (a run changed it) 450 00002000/00000800 | 2000 pld: window after (a run changed it) 650 00003000/00001800 | sig, end: window after (a run changed it) 450 00003800/00002000 | DMA + ECC encode: window after (a run changed it) 650 00000800/00003000, c14 000000ff/000000df | fault, C58 4 (vector): fault at 4248 us (base 4854)
  bit 14 (0x0000bff5): loop, SDRAM: window after (a run changed it) 428 000001f8/00000178, 628 00000074/000001f4 | loop, SRAM: window after (a run changed it) 428 000000e8/00000068, 450 00000800/00001000, 628 00000164/000000e4, 650 00000800/00001000 | 2000 ldi: window after (a run changed it) 428 000001d8/00000158, 628 00000054/000001d4 | 2000 sti: window after (a run changed it) 428 000000c8/00000048, 450 00002000/00002800, 628 00000144/000000c4, 650 00002000/00002800 | 2000 pld: window after (a run changed it) 428 000001b8/00000138, 628 00000034/000001b4 | sig, end: window after (a run changed it) 428 000000a8/00000028, 450 00003800/00000000, 628 00000124/000000a4, 650 00003800/00000000 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000118, 628 00000014/00000194, c14 000000ff/000000bf | fault, C58 4 (vector): fault at 3638 us (base 4854)
  bit 15 (0x00007ff5): loop, SDRAM: window after (a run changed it) 428 000001f8/000000f8, 628 00000074/00000174, 650 00000000/00002000 | loop, SRAM: window after (a run changed it) 428 000000e8/000001e8, 450 00000800/00002800, 628 00000164/00000064, 650 00000800/00003000 | 2000 ldi: window after (a run changed it) 428 000001d8/000000d8, 450 00001000/00003800, 628 00000054/00000154, 650 00001800/00003800 | 2000 sti: window after (a run changed it) 428 000000c8/000001c8, 450 00002000/00000000, 628 00000144/00000044, 650 00002000/00000800 | 2000 pld: window after (a run changed it) 428 000001b8/000000b8, 450 00002800/00001000, 628 00000034/00000134, 650 00003000/00001000 | sig, end: window after (a run changed it) 428 000000a8/000001a8, 450 00003800/00001800, 628 00000124/00000024, 650 00003800/00002000 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000098, 450 00000000/00002800, 628 00000014/00000114, 650 00000800/00002800, c14 000000ff/0000007f | fault, C58 4 (vector): fault at 2430 us (base 4854)
  bit 17 (0x0002fff5): loop, SDRAM: window after (a run changed it) 428 000001f8/00000078, 450 00003800/00000000, 628 00000074/000000f4 | loop, SRAM: window after (a run changed it) 428 000000e8/00000168, 628 00000164/000001e4 | 2000 ldi: window after (a run changed it) 428 000001d8/00000058, 450 00001000/00001800, 628 00000054/000000d4 | 2000 sti: window after (a run changed it) 428 000000c8/00000148, 628 00000144/000001c4 | 2000 pld: window after (a run changed it) 428 000001b8/00000038, 450 00002800/00003000, 628 00000034/000000b4 | sig, end: window after (a run changed it) 428 000000a8/00000128, 628 00000124/000001a4 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000018, 450 00000000/00000800, 628 00000014/00000094
  bit 18 (0x0004fff5): loop, SDRAM: window after (a run changed it) 650 00000000/00002000 | loop, SRAM: window after (a run changed it) 450 00000800/00002800 | 2000 ldi: window after (a run changed it) 650 00001800/00003800 | 2000 sti: window after (a run changed it) 450 00002000/00000000 | 2000 pld: window after (a run changed it) 650 00003000/00001000 | sig, end: window after (a run changed it) 450 00003800/00001800 | DMA + ECC encode: window after (a run changed it) 650 00000800/00002800
  bit 19 (0x0008fff5): loop, SDRAM: window after (a run changed it) 428 000001f8/00000178, 628 00000074/000001f4, 650 00000000/00003800 | loop, SRAM: window after (a run changed it) 428 000000e8/00000068, 628 00000164/000000e4 | 2000 ldi: window after (a run changed it) 428 000001d8/00000158, 628 00000054/000001d4, 650 00001800/00001000 | 2000 sti: window after (a run changed it) 428 000000c8/00000048, 628 00000144/000000c4 | 2000 pld: window after (a run changed it) 428 000001b8/00000138, 628 00000034/000001b4, 650 00003000/00002800 | sig, end: window after (a run changed it) 428 000000a8/00000028, 628 00000124/000000a4 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000118, 628 00000014/00000194, 650 00000800/00000000
  bit 20 (0x0010fff5): loop, SDRAM: window after (a run changed it) 428 000001f8/000000f8, 628 00000074/00000174, 650 00000000/00001800 | loop, SRAM: window after (a run changed it) 428 000000e8/000001e8, 450 00000800/00002000, 628 00000164/00000064, 650 00000800/00002800 | 2000 ldi: window after (a run changed it) 428 000001d8/000000d8, 450 00001000/00003000, 628 00000054/00000154, 650 00001800/00003000 | 2000 sti: window after (a run changed it) 428 000000c8/000001c8, 450 00002000/00003800, 628 00000144/00000044, 650 00002000/00000000 | 2000 pld: window after (a run changed it) 428 000001b8/000000b8, 450 00002800/00000800, 628 00000034/00000134, 650 00003000/00000800 | sig, end: window after (a run changed it) 428 000000a8/000001a8, 450 00003800/00001000, 628 00000124/00000024, 650 00003800/00001800 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000098, 450 00000000/00002000, 628 00000014/00000114, 650 00000800/00002000
  bit 21 (0x0020fff5): loop, SDRAM: window after (a run changed it) 428 000001f8/00000078, 628 00000074/000000f4, 650 00000000/00003800 | loop, SRAM: window after (a run changed it) 428 000000e8/00000168, 450 00000800/00000000, 628 00000164/000001e4 | 2000 ldi: window after (a run changed it) 428 000001d8/00000058, 628 00000054/000000d4, 650 00001800/00001000 | 2000 sti: window after (a run changed it) 428 000000c8/00000148, 450 00002000/00001800, 628 00000144/000001c4 | 2000 pld: window after (a run changed it) 428 000001b8/00000038, 628 00000034/000000b4, 650 00003000/00002800 | sig, end: window after (a run changed it) 428 000000a8/00000128, 450 00003800/00003000, 628 00000124/000001a4 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000018, 628 00000014/00000094, 650 00000800/00000000
  bit 22 (0x0040fff5): loop, SDRAM: window after (a run changed it) 650 00000000/00001800 | loop, SRAM: window after (a run changed it) 450 00000800/00002000 | 2000 ldi: window after (a run changed it) 650 00001800/00003000 | 2000 sti: window after (a run changed it) 450 00002000/00003800 | 2000 pld: window after (a run changed it) 650 00003000/00000800 | sig, end: window after (a run changed it) 450 00003800/00001000 | DMA + ECC encode: window after (a run changed it) 650 00000800/00002000
  bit 23 (0x0080fff5): loop, SDRAM: window after (a run changed it) 428 000001f8/00000178, 628 00000074/000001f4, 650 00000000/00003000 | loop, SRAM: window after (a run changed it) 428 000000e8/00000068, 450 00000800/00000000, 628 00000164/000000e4, 650 00000800/00000000 | 2000 ldi: window after (a run changed it) 428 000001d8/00000158, 628 00000054/000001d4, 650 00001800/00000800 | 2000 sti: window after (a run changed it) 428 000000c8/00000048, 450 00002000/00001800, 628 00000144/000000c4, 650 00002000/00001800 | 2000 pld: window after (a run changed it) 428 000001b8/00000138, 628 00000034/000001b4, 650 00003000/00002000 | sig, end: window after (a run changed it) 428 000000a8/00000028, 450 00003800/00003000, 628 00000124/000000a4, 650 00003800/00003000 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000118, 628 00000014/00000194, 650 00000800/00003800
  bit 24 (0x0100fff5): loop, SDRAM: window after (a run changed it) 428 000001f8/000000f8, 628 00000074/00000174, 650 00000000/00001000 | loop, SRAM: window after (a run changed it) 428 000000e8/000001e8, 450 00000800/00001800, 628 00000164/00000064, 650 00000800/00002000 | 2000 ldi: window after (a run changed it) 428 000001d8/000000d8, 450 00001000/00002800, 628 00000054/00000154, 650 00001800/00002800 | 2000 sti: window after (a run changed it) 428 000000c8/000001c8, 450 00002000/00003000, 628 00000144/00000044, 650 00002000/00003800 | 2000 pld: window after (a run changed it) 428 000001b8/000000b8, 450 00002800/00000000, 628 00000034/00000134, 650 00003000/00000000 | sig, end: window after (a run changed it) 428 000000a8/000001a8, 450 00003800/00000800, 628 00000124/00000024, 650 00003800/00001000 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000098, 450 00000000/00001800, 628 00000014/00000114, 650 00000800/00001800
  bit 25 (0x0200fff5): loop, SDRAM: window after (a run changed it) 428 000001f8/00000078, 450 00003800/00003000, 628 00000074/000000f4, 650 00000000/00003000 | loop, SRAM: window after (a run changed it) 428 000000e8/00000168, 450 00000800/00003800, 628 00000164/000001e4 | 2000 ldi: window after (a run changed it) 428 000001d8/00000058, 450 00001000/00000800, 628 00000054/000000d4, 650 00001800/00000800 | 2000 sti: window after (a run changed it) 428 000000c8/00000148, 450 00002000/00001000, 628 00000144/000001c4 | 2000 pld: window after (a run changed it) 428 000001b8/00000038, 450 00002800/00002000, 628 00000034/000000b4, 650 00003000/00002000 | sig, end: window after (a run changed it) 428 000000a8/00000128, 450 00003800/00002800, 628 00000124/000001a4 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000018, 450 00000000/00003800, 628 00000014/00000094, 650 00000800/00003800
  bit 26 (0x0400fff5): loop, SDRAM: window after (a run changed it) 650 00000000/00001000 | loop, SRAM: window after (a run changed it) 450 00000800/00001800 | 2000 ldi: window after (a run changed it) 650 00001800/00002800 | 2000 sti: window after (a run changed it) 450 00002000/00003000 | 2000 pld: window after (a run changed it) 650 00003000/00000000 | sig, end: window after (a run changed it) 450 00003800/00000800 | DMA + ECC encode: window after (a run changed it) 650 00000800/00001800
  bit 27 (0x0800fff5): loop, SDRAM: window after (a run changed it) 428 000001f8/00000178, 628 00000074/000001f4, 650 00000000/00002800 | loop, SRAM: window after (a run changed it) 428 000000e8/00000068, 450 00000800/00003800, 628 00000164/000000e4, 650 00000800/00003800 | 2000 ldi: window after (a run changed it) 428 000001d8/00000158, 628 00000054/000001d4, 650 00001800/00000000 | 2000 sti: window after (a run changed it) 428 000000c8/00000048, 450 00002000/00001000, 628 00000144/000000c4, 650 00002000/00001000 | 2000 pld: window after (a run changed it) 428 000001b8/00000138, 628 00000034/000001b4, 650 00003000/00001800 | sig, end: window after (a run changed it) 428 000000a8/00000028, 450 00003800/00002800, 628 00000124/000000a4, 650 00003800/00002800 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000118, 628 00000014/00000194, 650 00000800/00003000
  bit 28 (0x1000fff5): loop, SDRAM: window after (a run changed it) 428 000001f8/000000f8, 628 00000074/00000174, 650 00000000/00000800 | loop, SRAM: window after (a run changed it) 428 000000e8/000001e8, 450 00000800/00001000, 628 00000164/00000064, 650 00000800/00001800 | 2000 ldi: window after (a run changed it) 428 000001d8/000000d8, 450 00001000/00002000, 628 00000054/00000154, 650 00001800/00002000 | 2000 sti: window after (a run changed it) 428 000000c8/000001c8, 450 00002000/00002800, 628 00000144/00000044, 650 00002000/00003000 | 2000 pld: window after (a run changed it) 428 000001b8/000000b8, 450 00002800/00003800, 628 00000034/00000134, 650 00003000/00003800 | sig, end: window after (a run changed it) 428 000000a8/000001a8, 450 00003800/00000000, 628 00000124/00000024, 650 00003800/00000800 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000098, 450 00000000/00001000, 628 00000014/00000114, 650 00000800/00001000
  bit 29 (0x2000fff5): loop, SDRAM: window after (a run changed it) 428 000001f8/00000078, 450 00003800/00002800, 628 00000074/000000f4, 650 00000000/00002800 | loop, SRAM: window after (a run changed it) 428 000000e8/00000168, 450 00000800/00003000, 628 00000164/000001e4 | 2000 ldi: window after (a run changed it) 428 000001d8/00000058, 450 00001000/00000000, 628 00000054/000000d4, 650 00001800/00000000 | 2000 sti: window after (a run changed it) 428 000000c8/00000148, 450 00002000/00000800, 628 00000144/000001c4 | 2000 pld: window after (a run changed it) 428 000001b8/00000038, 450 00002800/00001800, 628 00000034/000000b4, 650 00003000/00001800 | sig, end: window after (a run changed it) 428 000000a8/00000128, 450 00003800/00002000, 628 00000124/000001a4 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000018, 450 00000000/00003000, 628 00000014/00000094, 650 00000800/00003000
  bit 30 (0x4000fff5): loop, SDRAM: window after (a run changed it) 650 00000000/00000800 | loop, SRAM: window after (a run changed it) 450 00000800/00001000 | 2000 ldi: window after (a run changed it) 650 00001800/00002000 | 2000 sti: window after (a run changed it) 450 00002000/00002800 | 2000 pld: window after (a run changed it) 650 00003000/00003800 | sig, end: window after (a run changed it) 450 00003800/00000000 | DMA + ECC encode: window after (a run changed it) 650 00000800/00001000
  bit 31 (0x8000fff5): loop, SDRAM: window after (a run changed it) 428 000001f8/00000178, 628 00000074/000001f4, 650 00000000/00002000 | loop, SRAM: window after (a run changed it) 428 000000e8/00000068, 450 00000800/00003000, 628 00000164/000000e4, 650 00000800/00003000 | 2000 ldi: window after (a run changed it) 428 000001d8/00000158, 628 00000054/000001d4, 650 00001800/00003800 | 2000 sti: window after (a run changed it) 428 000000c8/00000048, 450 00002000/00000800, 628 00000144/000000c4, 650 00002000/00000800 | 2000 pld: window after (a run changed it) 428 000001b8/00000138, 628 00000034/000001b4, 650 00003000/00001000 | sig, end: window after (a run changed it) 428 000000a8/00000028, 450 00003800/00002000, 628 00000124/000000a4, 650 00003800/00002000 | DMA + ECC encode: window after (a run changed it) 428 00000198/00000118, 628 00000014/00000194, 650 00000800/00002800
```

### 14.2.133 rsdecode (run 35)

ID: `D-rsdecode-35`. Device run 35. Experiment: [rsdecode](13-experiments.md#133105-rsdecode).

```
Reed-Solomon 12 / 16 parity symbols, decode strength and the parity buffer:
  0x08104,  0 bytes changed: ECC_RESULT 00000000 ECC_DONE 35, data corrected
      parity buffer after the encode (slots 0..3): 186d249847a87441d927d4965ed6e4ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
      ECC area 0x900..0x914: e4d65e96d427d94174a84798246d18a8012da80100000000
  0x08104,  4 bytes changed: ECC_RESULT 00040000 ECC_DONE 35, data corrected
  0x08104,  6 bytes changed: ECC_RESULT 00060000 ECC_DONE 35, data corrected
  0x08104,  7 bytes changed: ECC_RESULT 00000011 ECC_DONE 3d, data as sent
  0x08108,  0 bytes changed: ECC_RESULT 00000000 ECC_DONE 35, data corrected
      parity buffer after the encode (slots 0..3): 4e2923086f4cfc4b7b77718291507950ffa89d05ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
      ECC area 0x900..0x914: 059da8ff507950918271777b4bfc4c6f0823294e00000000
  0x08108,  4 bytes changed: ECC_RESULT 00040000 ECC_DONE 35, data corrected
  0x08108,  8 bytes changed: ECC_RESULT 00080000 ECC_DONE 35, data corrected
  0x08108,  9 bytes changed: ECC_RESULT 00000011 ECC_DONE 3d, data as sent
  0x08108, 12 bytes changed: ECC_RESULT 00000011 ECC_DONE 3d, data as sent
```

### 14.2.134 vic20 (run 35)

ID: `D-vic20-35`. Device run 35. Experiment: [vic20](13-experiments.md#133106-vic20).

```
interrupt lines per FMC source, three runs each:
  command  0x044 bit  0: CS_IRQ 11 VIC0.20 1 VIC1.22 0 | CS_IRQ 11 VIC0.20 1 VIC1.22 0 | CS_IRQ 11 VIC0.20 1 VIC1.22 0
  command  0x044 bit  1: CS_IRQ 01 VIC0.20 0 VIC1.22 0 | CS_IRQ 01 VIC0.20 0 VIC1.22 0 | CS_IRQ 01 VIC0.20 0 VIC1.22 0
  command  0x044 bit  2: CS_IRQ 01 VIC0.20 0 VIC1.22 0 | CS_IRQ 01 VIC0.20 0 VIC1.22 0 | CS_IRQ 01 VIC0.20 0 VIC1.22 0
  address  0x044 bit  0: CS_IRQ 01 VIC0.20 0 VIC1.22 0 | CS_IRQ 01 VIC0.20 0 VIC1.22 0 | CS_IRQ 01 VIC0.20 0 VIC1.22 0
  address  0x044 bit  1: CS_IRQ 11 VIC0.20 1 VIC1.22 0 | CS_IRQ 11 VIC0.20 1 VIC1.22 0 | CS_IRQ 11 VIC0.20 1 VIC1.22 0
  status   0x044 bit  0: CS_IRQ 11 VIC0.20 0 VIC1.22 0 | CS_IRQ 11 VIC0.20 0 VIC1.22 0 | CS_IRQ 11 VIC0.20 0 VIC1.22 0
  status   0x044 bit 13: CS_IRQ 01 VIC0.20 0 VIC1.22 0 | CS_IRQ 01 VIC0.20 0 VIC1.22 0 | CS_IRQ 01 VIC0.20 0 VIC1.22 0
  DMA      0x044 bit 11: CS_IRQ 11 VIC0.20 0 VIC1.22 0 | CS_IRQ 11 VIC0.20 0 VIC1.22 0 | CS_IRQ 11 VIC0.20 0 VIC1.22 0
  none     0x044 bit  0: CS_IRQ 01 VIC0.20 0 VIC1.22 0 | CS_IRQ 01 VIC0.20 0 VIC1.22 0 | CS_IRQ 01 VIC0.20 0 VIC1.22 0
```

### 14.2.135 fmc28b (run 35)

ID: `D-fmc28b-35`. Device run 35. Experiment: [fmc28b](13-experiments.md#133107-fmc28b).

```
FMC_28 (9 bits) around runs:
  no set-up nothing            CPU before 232 | program start 480, end 480 | CPU after  92
  no set-up loop 2000          CPU before 464 | program start 200, end 200 | CPU after 324
  no set-up DMA                CPU before 184 | program start 432, end   0 | CPU after 124
  no set-up ECC encode         CPU before 496 | program start 232, end 232 | CPU after 356
  no set-up 16-byte chip read  CPU before 216 | program start 464, end 464 | CPU after  76
  set-up    nothing            CPU before 448 | program start   0, end   0 | CPU after 124
  set-up    loop 2000          CPU before 496 | program start   0, end   0 | CPU after 124
  set-up    DMA                CPU before 496 | program start   0, end   0 | CPU after 124
  set-up    ECC encode         CPU before 496 | program start   0, end   0 | CPU after 124
  set-up    16-byte chip read  CPU before 496 | program start   0, end   0 | CPU after 124
```

### 14.2.136 c58b1 (run 35)

ID: `D-c58b1-35`. Device run 35. Experiment: [c58b1](13-experiments.md#133108-c58b1).

```
C58 bit 1 (run 1 | run 2 without new writes):
  C58=2, C4C                                     END (C5C 0) | then: END (C5C 0)
  C4C, C58=2                                     END (C5C 0) | then: END (C5C 0)
  C58=2 only (C4C left from the previous case)   END (C5C 0)
  C4C only (C58=2 left)                          END (C5C 0) | then: END (C5C 0)
  C58=2, C4C, C4C                                END (C5C 0) | then: END (C5C 0)
  C58=2, C4C, C5C=0                              END (C5C 0) | then: END (C5C 0)
  C58=2, C4C, C58=2                              END (C5C 0) | then: END (C5C 0)
  C4C=0x40 by CPU, C58=2, program writes C4C     END (C5C 0) | then: END (C5C 0)
  C58=3, C4C                                     END (C5C 1) | then: END (C5C 0)
  C58=6, C4C                                     END (C5C 0) | then: END (C5C 0)
```

### 14.2.137 evsweep2 (run 35)

ID: `D-evsweep2-35`. Device run 35. Experiment: [evsweep2](13-experiments.md#133109-evsweep2).

```
wait / wait2 sweep per state (short timeout; pass = no fault):
  idle, CE0 selected                           wait passes [11]; wait2 passes [5]; END/END 1645/1645 us
  erase busy                                   wait passes [11]; wait2 passes [5]; END/END 1659/1660 us
  program busy (80h addr 10h)                  wait passes [11]; wait2 passes [5]; END/END 1660/1658 us
  read busy (00h addr 30h)                     wait passes [11]; wait2 passes [5]; END/END 1645/1658 us
  ECC encode in flight                         wait passes [4, 11]; wait2 passes [5]; END/END 1645/1644 us
      beyond idle: [('wait', 4)]
  DMA in flight                                wait passes [2, 11]; wait2 passes [5]; END/END 1645/1646 us
      beyond idle: [('wait', 2)]
  0x044 / 0x844 all on, command flag pending   wait passes [0, 11]; wait2 passes [5, 20]; END/END 1634/1643 us
      beyond idle: [('wait', 0), ('wait2', 20)]
  second set: command sent                     wait passes [6, 11]; wait2 passes [5]; END/END 1643/1643 us
      beyond idle: [('wait', 6)]
```

### 14.2.138 c50fault (run 35)

ID: `D-c50fault-35`. Device run 35. Experiment: [c50fault](13-experiments.md#133110-c50fault).

```
C50 around faults (bit 0) and bit 8:
  C58 0 timeout (wait 10)  nothing pending  before ffff1800 | after ffff1800 | handler 00000000, after w1 00000000, C3C 00000000 | end C50 ffff1800, FAULT r6 600d600d
  C58 0 timeout (wait 10)  command pending  before ffff1801 | after ffff1801 | handler 00000000, after w1 00000000, C3C 00000000 | end C50 ffff1801, FAULT r6 600d600d
  C58 0 illegal 0x30       nothing pending  before ffff1800 | after ffff1800 | handler 00000000, after w1 00000000, C3C 00000000 | end C50 ffff1800, FAULT r6 600d600d
  C58 0 illegal 0x30       command pending  before ffff1801 | after ffff1801 | handler 00000000, after w1 00000000, C3C 00000000 | end C50 ffff1801, FAULT r6 600d600d
  C58 4 timeout (wait 10)  nothing pending  before ffff1800 | after 00000000 | handler ffff1800, after w1 ffff1800, C3C 00000058 | end C50 ffff1800, FAULT r6 4a4d4a4d
  C58 4 timeout (wait 10)  command pending  before ffff1801 | after 00000000 | handler ffff1801, after w1 ffff1801, C3C 00000068 | end C50 ffff1801, FAULT r6 4a4d4a4d
  C58 4 illegal 0x30       nothing pending  before ffff1800 | after 00000000 | handler ffff1800, after w1 ffff1800, C3C 00000058 | end C50 ffff1800, FAULT r6 4a4d4a4d
  C58 4 illegal 0x30       command pending  before ffff1801 | after 00000000 | handler ffff1801, after w1 ffff1801, C3C 00000068 | end C50 ffff1801, FAULT r6 4a4d4a4d
  C50 bit 8: after a soft reset, END only   before ffff1800 after ffff1800
  C50 bit 8: again                          before ffff1800 after ffff1800
  C50 bit 8: status latch                   before ffff1800 after ffff1820
```

### 14.2.139 ctrlwords (run 35)

ID: `D-ctrlwords-35`. Device run 35. Experiment: [ctrlwords](13-experiments.md#133111-ctrlwords).

```
unused control words (value written: C10 / C48 / C54) and C3C in handlers:
  timeout, C58 4     00000000 FAULT 4883us fault 4868us CS_IRQ 05 r6 fa17fa17 0x900 00000000 | ffffffff FAULT 4883us fault 4868us CS_IRQ 05 r6 fa17fa17 0x900 00000000 | 55555555 FAULT 4870us fault 4854us CS_IRQ 05 r6 fa17fa17 0x900 00000000
  illegal, C58 4     00000000 FAULT 4883us fault 4868us CS_IRQ 05 r6 fa17fa17 0x900 00000000 | ffffffff FAULT 4868us fault 4854us CS_IRQ 05 r6 fa17fa17 0x900 00000000 | 55555555 FAULT 4883us fault 4854us CS_IRQ 05 r6 fa17fa17 0x900 00000000
  wait2 33           00000000 FAULT 4883us fault 4868us CS_IRQ 05 r6 600d600d 0x900 00000000 | ffffffff FAULT 4881us fault 4866us CS_IRQ 05 r6 600d600d 0x900 00000000 | 55555555 FAULT 4865us fault 4850us CS_IRQ 05 r6 600d600d 0x900 00000000
  DMA + ECC encode   00000000 END 18us fault 0us CS_IRQ 01 r6 600d600d 0x900 95982422 | ffffffff END 18us fault 0us CS_IRQ 01 r6 600d600d 0x900 95982422 | 55555555 END 18us fault 0us CS_IRQ 01 r6 600d600d 0x900 95982422
  tmwait twice       00000000 END 4841us fault 0us CS_IRQ 01 r6 600d600d 0x900 95982422 | ffffffff END 18us fault 0us CS_IRQ 01 r6 600d600d 0x900 95982422 | 55555555 END 3250us fault 0us CS_IRQ 01 r6 600d600d 0x900 95982422
  C3C: fault at offset 0x28: first handler entry C3C 00000030 (PC read there 00000058), second entry C3C 00000068; FAULT r6 4a4d4a4d
  C3C: fault at offset 0x40: first handler entry C3C 00000048 (PC read there 00000070), second entry C3C 00000080; FAULT r6 4a4d4a4d
  C3C: fault at offset 0x70: first handler entry C3C 00000078 (PC read there 000000a0), second entry C3C 000000b0; FAULT r6 4a4d4a4d
```

### 14.2.140 c08self (run 35)

ID: `D-c08self-35`. Device run 35. Experiment: [c08self](13-experiments.md#133112-c08self).

```
CS_STATUS read by the program (r1, then r2 after the wait; eeee = not read):
  plain                  C08 5, then eeee; END
  after DMA start        C08 5, then 5; END
  after ECC kick         C08 5, then 5; END
  after a chip command   C08 5, then 5; END
  after sig              C08 5, then eeee; END
  after wait2 33         C08 5, then eeee; FAULT
  pld 0xc08              C08 5, then eeee; END
  in a fault handler     C08 5, then 4; FAULT
```

### 14.2.141 latchrep (run 35)

ID: `D-latchrep-35`. Device run 35. Experiment: [latchrep](13-experiments.md#133113-latchrep).

```
status latch and wait 3, 40 runs each:
  fresh arm                              pass b23 1 b24 0 x40
  no fresh arm                           pass b23 1 b24 0 x40
  fresh arm, 0x044 / 0x844 written       pass b23 1 b24 0 x40
  no fresh arm, 0x044 / 0x844 written    pass b23 1 b24 0 x40
  two latches, no arm between            timeout b23 0 b24 0 x40
  wait 3 on CE0 (16-byte read)           pass b23 0 b24 0 x40
```

### 14.2.142 irq044 (run 35)

ID: `D-irq044-35`. Device run 35. Experiment: [irq044](13-experiments.md#133114-irq044).

```
0x044 one bit at a time (three runs):
  none     CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 0    CS_IRQ 11 V0.20 0 V1.22 0 END | CS_IRQ 11 V0.20 0 V1.22 0 END | CS_IRQ 11 V0.20 0 V1.22 0 END
  bit 1    CS_IRQ 11 V0.20 0 V1.22 0 END | CS_IRQ 11 V0.20 0 V1.22 0 END | CS_IRQ 11 V0.20 0 V1.22 0 END
  bit 2    CS_IRQ 11 V0.20 0 V1.22 0 END | CS_IRQ 11 V0.20 0 V1.22 0 END | CS_IRQ 11 V0.20 0 V1.22 0 END
  bit 3    CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 4    CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 5    CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 6    CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 7    CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 8    CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 9    CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 10   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 11   CS_IRQ 11 V0.20 0 V1.22 0 END | CS_IRQ 11 V0.20 0 V1.22 0 END | CS_IRQ 11 V0.20 0 V1.22 0 END
  bit 12   CS_IRQ 11 V0.20 0 V1.22 0 END | CS_IRQ 11 V0.20 0 V1.22 0 END | CS_IRQ 11 V0.20 0 V1.22 0 END
  bit 13   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 14   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 15   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 16   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 17   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 18   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 19   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 20   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 21   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 22   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 23   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 24   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 25   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 26   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 27   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 28   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 29   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 30   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
  bit 31   CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END | CS_IRQ 01 V0.20 0 V1.22 0 END
```

### 14.2.143 fmcbits (run 35)

ID: `D-fmcbits-35`. Device run 35. Experiment: [fmcbits](13-experiments.md#133115-fmcbits).

```
FMC_CTRL0 bits, FMC_DMA_CTRL and drain selects on a page read:
  erase + program: END, NSTAT c0
  base 0x43803                 END   317us data ==== results 00000000 00000000 00000000 00000000 FMC_DMA_CTRL after 00000007
  bit 9 set                    END   302us data ==== results 00000000 00000000 00000000 00000000 FMC_DMA_CTRL after 00000007
  bit 11 clear                 END   317us data ==== results 00000000 00000000 00000000 00000000 FMC_DMA_CTRL after 00000007
  bit 24 set throughout        END   317us data ==== results 00000000 00000000 00000000 00000000 FMC_DMA_CTRL after 00000007
  FMC_DMA_CTRL 0               END   332us data ==== results 00000000 00000000 00000000 00000000 FMC_DMA_CTRL after 00000000
  FMC_DMA_CTRL 7               END   436us data ==== results 00000000 00000000 00000000 00000000 FMC_DMA_CTRL after 00000007
  FMC_DMA_CTRL 0xff            FAULT  5177us data ===x results 00000000 00000000 00000000 00000000 FMC_DMA_CTRL after 000000ff
  drain 0x102                  END   361us data xxxx results 00000000 00000000 00000000 00000000 FMC_DMA_CTRL after 0000000f
  drain 0x1e0                  END   361us data xxxx results 00000000 00000000 00000000 00000000 FMC_DMA_CTRL after 0000000f
  drain 0x1e0 + DMA_CTRL 7     END   359us data xxx= results 00000000 00000000 00000000 00000000 FMC_DMA_CTRL after 00000007
  READID base                  END ID b614d5ec 14d5ec74 status 08041000
  READID bit 9                 END ID b614d5ec 14d5ec74 status 08041000
  READID no bit 11             END ID b614d5ec 14d5ec74 status 08041000
  READID bit 9, no bit 11      END ID b614d5ec 14d5ec74 status 08041000
```

### 14.2.144 fmcstat (run 35)

ID: `D-fmcstat-35`. Device run 35. Experiment: [fmcstat](13-experiments.md#133116-fmcstat).

```
FMC_STATUS power-on bits:
  after a soft reset: FMC_STATUS 000c3000, 0x448 00000000
  bit 12 (start > after w1c > after the operation): nothing 1>1>1, command 1>1>1, address 1>1>1, data read 1>1>1, status latch 1>1>1, DMA 1>1>1, ECC encode 1>1>1, meta encode 0>1>1, second set command 1>1>1
  bit 13 (start > after w1c > after the operation): nothing 1>1>1, command 1>1>1, address 1>1>1, data read 1>1>0, status latch 1>1>1, DMA 1>1>1, ECC encode 1>1>1, meta encode 0>1>1, second set command 1>1>1
  bit 18 (start > after w1c > after the operation): nothing 1>1>1, command 1>1>1, address 1>1>1, data read 1>1>1, status latch 1>1>1, DMA 1>1>1, ECC encode 1>1>1, meta encode 0>1>1, second set command 1>1>1
  bit 19 (start > after w1c > after the operation): nothing 1>1>1, command 1>1>1, address 1>1>1, data read 1>1>0, status latch 1>1>1, DMA 1>1>1, ECC encode 1>1>1, meta encode 0>1>1, second set command 1>1>1
```

### 14.2.145 eccregs (run 35)

ID: `D-eccregs-35`. Device run 35. Experiment: [eccregs](13-experiments.md#133117-eccregs).

```
ECC words 0x940..0x950 and 0x980..0x99c:
  0x940..0x950 = 00000000: parity ffffff7a82a426c457e3a863a1cb34c8, decode 00080000 / 35, data corrected, 0x940.. after 00000000 00000000 00000000 00000000 00000000
  0x940..0x950 = ffffffff: parity ffffff7a82a426c457e3a863a1cb34c8, decode 00080000 / 35, data corrected, 0x940.. after ffffffff ffffffff ffffffff ffffffff ffffffff
  0x940..0x950 = 55555555: parity ffffff7a82a426c457e3a863a1cb34c8, decode 00080000 / 35, data corrected, 0x940.. after 55555555 55555555 55555555 55555555 55555555
  mode 0x0180 zeros 0x980..0x99c 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
  mode 0x0180 ones  0x980..0x99c 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
  mode 0x0180 ramp  0x980..0x99c 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
  mode 0x8100 zeros 0x980..0x99c 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
  mode 0x8100 ones  0x980..0x99c 00000000 00000000 00000000 00000000 fc36fc36 00000000 00000000 00000000
  mode 0x8100 ramp  0x980..0x99c 00000000 00000000 00000000 00000000 c68bc68b 00000000 00000000 00000000
  mode 0x8108 zeros 0x980..0x99c 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
  mode 0x8108 ones  0x980..0x99c 00000000 00000000 00000000 00000000 fc36fc36 00000000 00000000 00000000
  mode 0x8108 ramp  0x980..0x99c 00000000 00000000 00000000 00000000 c68bc68b 00000000 00000000 00000000
  mode 0x0000 zeros 0x980..0x99c 00000000 00000000 00000000 00000000 f0f4f0f4 00000000 00000000 00000000
  mode 0x0000 ones  0x980..0x99c 00000000 00000000 00000000 00000000 5bb85bb8 00000000 00000000 00000000
  mode 0x0000 ramp  0x980..0x99c 00000000 00000000 00000000 00000000 47344734 00000000 00000000 00000000
  mode 0x0080 zeros 0x980..0x99c 00000000 00000000 00000000 00000000 f6adf6ad 00000000 00000000 00000000
  mode 0x0080 ones  0x980..0x99c 00000000 00000000 00000000 00000000 3d6d3d6d 00000000 00000000 00000000
  mode 0x0080 ramp  0x980..0x99c 00000000 00000000 00000000 00000000 73537353 00000000 00000000 00000000
```

### 14.2.146 seqrestart (run 35)

ID: `D-seqrestart-35`. Device run 35. Experiment: [seqrestart](13-experiments.md#133118-seqrestart).

```
mode 0 encodes, area 0x900..0x913 (three per run):
  between: nothing              run 1 END edd42e2dda3f79ae9517f0dc3700000000000000 a62f0baf90c8082570138c3f1200000000000000 c47f1becb633ef38283ab3fa8400000000000000 | run 2 END 528950e823dd0c2ee3334b3ccf00000000000000 b5d2b4ab2eacd0192881214ff700000000000000 993f23a304711734be92d1c26000000000000000
  between: soft reset           run 1 END 5788eb241e93af5b28f704241000000000000000 2ca900f00bae330c7f30f0c62a00000000000000 b0c691ff3e6a42d353fb5a0bcb00000000000000 | run 2 END 4684475615107a7cfd75b3cebe00000000000000 5da0a18c3e1f8acedf03f2ac6800000000000000 b1250ddf69ebfa90821e21278300000000000000
  between: ECC_MODE write       run 1 END a49605af7f7209f9bc12b71a3a00000000000000 5f6698cd901dfb497dc805f4f800000000000000 56fb4de8fda80f967a303d769f00000000000000 | run 2 END 83e1b2e862f0f8fb82654c520f00000000000000 91db19a3b89a11448d953d56c000000000000000 18152367c7acec9306decbe7f500000000000000
  between: ECC_KICK 0           run 1 END 1f9ab13532fe2de062de3c167e00000000000000 0d07c4bdcd92d74f7549d0751500000000000000 20e225dd66b046a4c6a92a6f1700000000000000 | run 2 END 04d8cecd9969b2fbe987f3a8c100000000000000 5e120d0ccf2c902c8e46069dc500000000000000 359d1fe870186a9fabfaa0eb7d00000000000000
  between: BCH encode           run 1 END 8109986bdd922e3164784b807500000000000000 74ec7966e37cc95a54e012941000000000000000 1cc57661b8694006cbe5c5430000000000000000 | run 2 END f60eb57ac4b58fd1abd5766bca00000000000000 265cab74729f9d6895ded8c4eb00000000000000 d130e886cba011cb2f5eaa6c6a00000000000000
  between: BCH decode           run 1 END 7195d49aa7f535b95248f6332900000000000000 bcb796bb950d3ef25ca9079a3f00000000000000 fcfcef834da77616a685bf24b900000000000000 | run 2 END 66b96bc129576180ba475c779400000000000000 c5d45d74d885e34434fe38f38c00000000000000 f15f55f11065cc680d7aff54d600000000000000
  between: ECC_DONE write       run 1 END b784399bf3c0c9e11009365ce700000000000000 df922891620196b96201b9135200000000000000 5324f145a44a9dab59e72b023000000000000000 | run 2 END a0f31794c74e3117c61721644f00000000000000 b89ea43d4ad92733b0db04478b00000000000000 5e31699e8dd17f4a8f3a118b7500000000000000
  between: 0x940..0x950 write   run 1 END 4d10cb08d779410e18424e34e800000000000000 a2b4948a19efe2f01d6071550000000000000000 a70d1462ed388c744871dbd22e00000000000000 | run 2 END 5abf6fa33192d88538d5b1e9eb00000000000000 50cd6e72d83d058d93f7e5e6b600000000000000 89535d3520efbf63095f24692900000000000000
  between: FMC_CTRL0 write      run 1 END 9db75f97f3b004725f1a8c779300000000000000 0c71fcdc429262a311ab1b91b900000000000000 07423d5da4aa078cc6c15f55d800000000000000 | run 2 END 25cf7acac6efcb2e5ba370988d00000000000000 33a9f8c90b9e0170f576f8104e00000000000000 77b377e7ce149935ceb3a68ae500000000000000
  between: sequencer reset      run 1 END 7884b72515701e84e9f8a3627700000000000000 d34b6dbddee23c03e4920aaf3500000000000000 eede29fd28ac216cd0e414860500000000000000 | run 2 END b8419cccbf896462cb30461d8000000000000000 c26b154c53145ebca3dc7a4fe000000000000000 4daebaea3cd8c7acee94cb80fe00000000000000
```

### 14.2.147 done5 (run 35)

ID: `D-done5-35`. Device run 35. Experiment: [done5](13-experiments.md#133119-done5).

```
ECC_DONE after encode / decode (ee = the wait timed out before it):
  base 0x0180: - 3b/35, b9 33/35, b10 33/35, b11 33/35, b12 33/35, b13 33/35, b14 33/35, b16 33/35, b17 33/35, b18 33/35, b19 33/35, b20 33/35, b21 13/15, b22 13/15, b23 13/15, b24 13/15
  base 0x8100: - 3b/35, b9 33/35, b10 33/35, b11 33/35, b12 33/35, b13 33/35, b14 33/35, b16 33/35, b17 33/35, b18 33/35, b19 33/35, b20 33/35, b21 13/15, b22 13/15, b23 13/15, b24 13/15
  base 0x0000: - 33/3d, b9 3b/3d, b10 3b/3d, b11 3b/3d, b12 3b/3d, b13 3b/3d, b14 3b/3d, b16 3b/3d, b17 3b/3d, b18 3b/3d, b19 3b/3d, b20 3b/3d, b21 1b/1d, b22 1b/1d, b23 1b/1d, b24 1b/1d
```

### 14.2.148 progtime (run 35)

ID: `D-progtime-35`. Device run 35. Experiment: [progtime](13-experiments.md#133120-progtime).

```
page program time per page (one run per page, the same pattern):
  CE0 block +60 (us per page, negative = failed):
      0:   478   478   478   477   728   781   478   478   728   821   478   478   781   821   478   478
     16:   782   822   478   478   729   820   478   478   728   781   478   478   781   781   478   478
     32:   781   821   478   478   781   782   478   478   728   820   478   478   781   821   478   478
     48:   781   820   478   478   780   781   478   478   781   820   478   478   781   782   478   478
     64:   781   821   477   478   781   820   479   478   780   820   478   478   781   781   478   479
     80:   781   821   478   478   781   821   478   478   781   820   479   478   781   820   479   478
     96:   781   781   479   478   729   821   478   478   728   781   478   471   781   781   478   478
    112:   729   781   478   478   781   781   477   478   781   821   479   478   728   781   729   782
  CE0 block +61 (us per page, negative = failed):
      0:   478   477   478   478   728   781   478   478   781   821   478   478   781   820   479   477
     16:   728   821   478   479   781   781   478   478   729   820   478   477   729   781   478   478
     32:   728   821   478   478   781   781   478   477   728   782   478   478   781   781   478   479
     48:   729   820   478   478   781   821   478   478   729   820   478   479   781   781   478   478
     64:   781   820   478   478   782   781   478   479   782   782   478   477   781   821   478   478
     80:   780   821   479   478   781   821   478   478   781   821   479   478   782   782   478   478
     96:   728   817   478   479   782   781   479   478   728   780   478   479   728   777   479   478
    112:   729   782   479   477   729   781   478   478   728   821   478   478   729   781   729   781
  CE1 block +60 (us per page, negative = failed):
      0:   477   478   478   478   835   834   530   532   834   834   477   478   833   834   531   478
     16:   833   833   478   478   834   833   532   478   833   834   472   478   834   834   531   478
     32:   834   835   478   477   833   833   531   478   834   873   531   478   873   834   530   478
     48:   873   833   478   531   834   833   531   478   835   833   478   526   873   834   531   478
     64:   834   873   479   478   873   834   530   478   834   873   478   532   874   835   531   478
     80:   834   835   478   531   833   834   478   478   833   873   478   530   833   834   478   478
     96:   834   873   478   531   835   833   478   478   834   833   478   478   874   834   477   478
    112:   835   833   478   478   834   834   530   478   833   833   478   478   873   834   782   781
```

### 14.2.149 regsweep (run 35)

ID: `D-regsweep-35`. Device run 35. Experiment: [regsweep](13-experiments.md#133121-regsweep).

```
regsweep: A/B done to 31
regsweep: A/B done to 63
regsweep: A/B done to 95
regsweep: A/B done to 127
regsweep: A/B done to 159
regsweep: A/B done to 191
regsweep: A/B done to 223
regsweep: A/B done to 255
register numbers 8..255, every role:
 destinations (movi / mov r1 / add r1 + 1 into N; registers that changed, named by source):
  9 (1 numbers): END; movi: no register; mov: no register; add: no register
  10 (1 numbers): END; movi: r0=imm r1=imm r2=imm r3=imm r4=imm r5=imm r6=imm r7=imm; mov: r0=r1 r2=r1 r3=r1 r4=r1 r5=r1 r6=r1 r7=r1; add: r0=r1+1 r1=r1+1 r2=r1+1 r3=r1+1 r4=r1+1 r5=r1+1 r6=r1+1 r7=r1+1; window 428=000001d8 450=00002000 628=00000054 650=00002800
  11 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001c8 450=00002800 628=00000044 650=00003000
  12 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001b8 450=00003000 628=00000034 650=00003800
  13 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001a8 450=00003800 628=00000024 650=00000000
  14 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000198 450=00000000 628=00000014 650=00000800
  15 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000188 450=00000800 628=00000004 650=00001000
  16 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000178 450=00001000 628=000001f4 650=00001000
  17 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000168 628=000001e4 650=00001800
  18 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000158 450=00002000 628=000001d4
  19 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000148 450=00002800 628=000001c4 650=00002800
  20 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000138 450=00003000 628=000001b4 650=00003000
  21 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000128 450=00003800 628=000001a4 650=00003800
  22 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000118 450=00000000 628=00000194 650=00000000
  23 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000108 450=00000800 628=00000184 650=00000800
  24 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000f8 450=00001000 628=00000174 650=00001000
  25 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000e8 628=00000164 650=00001800
  26 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000d8 450=00002000 628=00000154
  27 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000c8 450=00002800 628=00000144 650=00002800
  28 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000b8 450=00003000 628=00000134 650=00003000
  29 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000a8 450=00003800 628=00000124 650=00003800
  30 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000098 450=00000000 628=00000114 650=00000000
  31 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000088 450=00000800 628=00000104 650=00000800
  32 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000078 450=00001000 628=000000f4 650=00001000
  33 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000068 628=000000e4 650=00001800
  34 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000058 450=00002000 628=000000d4
  35 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000048 450=00002800 628=000000c4 650=00002800
  36 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000038 450=00003000 628=000000b4 650=00003000
  37 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000028 450=00003800 628=000000a4 650=00003800
  38 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000018 450=00000000 628=00000094 650=00000000
  39 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000008 450=00000800 628=00000084 650=00000800
  40 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001f8 450=00000800 628=00000074 650=00001000
  41 (1 numbers): END; movi: no register; mov: no register; add: no register; window 450=00001000 650=00001800
  42 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001d8 628=00000054
  43 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001c8 450=00002000 628=00000044 650=00002800
  44 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001b8 450=00002800 628=00000034 650=00003000
  45 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001a8 450=00003000 628=00000024 650=00003800
  46 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000198 450=00003800 628=00000014 650=00000000
  47 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000188 450=00000000 628=00000004 650=00000800
  48 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000178 450=00000800 628=000001f4 650=00000800
  49 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000168 450=00001000 628=000001e4 650=00001000
  50 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000158 628=000001d4 650=00001800
  51 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000148 450=00002000 628=000001c4
  52 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000138 450=00002800 628=000001b4 650=00002800
  53 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000128 450=00003000 628=000001a4 650=00003000
  54 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000118 450=00003800 628=00000194 650=00003800
  55 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000108 450=00000000 628=00000184 650=00000000
  56 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000f8 450=00000800 628=00000174 650=00000800
  57 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000e8 450=00001000 628=00000164 650=00001000
  58 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000d8 628=00000154 650=00001800
  59 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000c8 450=00002000 628=00000144
  60 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000b8 450=00002800 628=00000134 650=00002800
  61 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000a8 450=00003000 628=00000124 650=00003000
  62 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000098 450=00003800 628=00000114 650=00003800
  63 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000088 450=00000000 628=00000104 650=00000000
  64 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000078 450=00000800 628=000000f4 650=00000800
  65 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000068 450=00001000 628=000000e4 650=00001000
  66 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000058 628=000000d4 650=00001800
  67 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000048 450=00002000 628=000000c4
  68 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000038 450=00002800 628=000000b4 650=00002800
  69 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000028 450=00003000 628=000000a4 650=00003000
  70 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000018 450=00003800 628=00000094 650=00003800
  71 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000008 450=00000000 628=00000084 650=00000000
  72 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001f8 450=00000000 628=00000074 650=00000800
  73 (1 numbers): END; movi: no register; mov: no register; add: no register; window 450=00000800 650=00001000
  74 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001d8 450=00001000 628=00000054 650=00001800
  75 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001c8 628=00000044
  76 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001b8 450=00002000 628=00000034 650=00002800
  77 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001a8 450=00002800 628=00000024 650=00003000
  78 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000198 450=00003000 628=00000014 650=00003800
  79 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000188 450=00003800 628=00000004 650=00000000
  80 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000178 450=00000000 628=000001f4 650=00000000
  81 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000168 450=00000800 628=000001e4 650=00000800
  82 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000158 450=00001000 628=000001d4 650=00001000
  83 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000148 628=000001c4 650=00001800
  84 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000138 450=00002000 628=000001b4
  85 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000128 450=00002800 628=000001a4 650=00002800
  86 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000118 450=00003000 628=00000194 650=00003000
  87 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000108 450=00003800 628=00000184 650=00003800
  88 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000f8 450=00000000 628=00000174 650=00000000
  89 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000e8 450=00000800 628=00000164 650=00000800
  90 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000d8 450=00001000 628=00000154 650=00001000
  91 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000c8 628=00000144 650=00001800
  92 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000b8 450=00002000 628=00000134
  93 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000a8 450=00002800 628=00000124 650=00002800
  94 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000098 450=00003000 628=00000114 650=00003000
  95 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000088 450=00003800 628=00000104 650=00003800
  96 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000078 450=00000000 628=000000f4 650=00000000
  97 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000068 450=00000800 628=000000e4 650=00000800
  98 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000058 450=00001000 628=000000d4 650=00001000
  99 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000048 628=000000c4 650=00001800
  100 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000038 450=00002000 628=000000b4
  101 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000028 450=00002800 628=000000a4 650=00002800
  102 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000018 450=00003000 628=00000094 650=00003000
  103 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000008 450=00003800 628=00000084 650=00003800
  104 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001f8 450=00003800 628=00000074 650=00000000
  105 (1 numbers): END; movi: no register; mov: no register; add: no register; window 450=00000000 650=00000800
  106 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001d8 450=00000800 628=00000054 650=00001000
  107 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001c8 450=00001000 628=00000044 650=00001800
  108 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001b8 628=00000034
  109 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001a8 450=00002000 628=00000024 650=00002800
  110 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000198 450=00002800 628=00000014 650=00003000
  111 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000188 450=00003000 628=00000004 650=00003800
  112 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000178 450=00003800 628=000001f4 650=00003800
  113 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000168 450=00000000 628=000001e4 650=00000000
  114 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000158 450=00000800 628=000001d4 650=00000800
  115 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000148 450=00001000 628=000001c4 650=00001000
  116 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000138 628=000001b4 650=00001800
  117 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000128 450=00002000 628=000001a4
  118 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000118 450=00002800 628=00000194 650=00002800
  119 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000108 450=00003000 628=00000184 650=00003000
  120 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000f8 450=00003800 628=00000174 650=00003800
  121 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000e8 450=00000000 628=00000164 650=00000000
  122 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000d8 450=00000800 628=00000154 650=00000800
  123 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000c8 450=00001000 628=00000144 650=00001000
  124 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000b8 628=00000134 650=00001800
  125 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000a8 450=00002000 628=00000124
  126 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000098 450=00002800 628=00000114 650=00002800
  127 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000088 450=00003000 628=00000104 650=00003000
  128 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000078 450=00003800 628=000000f4 650=00003800
  129 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000068 450=00000000 628=000000e4 650=00000000
  130 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000058 450=00000800 628=000000d4 650=00000800
  131 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000048 450=00001000 628=000000c4 650=00001000
  132 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000038 628=000000b4 650=00001800
  133 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000028 450=00002000 628=000000a4
  134 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000018 450=00002800 628=00000094 650=00002800
  135 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000008 450=00003000 628=00000084 650=00003000
  136 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001f8 450=00003000 628=00000074 650=00003800
  137 (1 numbers): END; movi: no register; mov: no register; add: no register; window 450=00003800 650=00000000
  138 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001d8 450=00000000 628=00000054 650=00000800
  139 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001c8 450=00000800 628=00000044 650=00001000
  140 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001b8 450=00001000 628=00000034 650=00001800
  141 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001a8 628=00000024
  142 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000198 450=00002000 628=00000014 650=00002800
  143 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000188 450=00002800 628=00000004 650=00003000
  144 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000178 450=00003000 628=000001f4 650=00003000
  145 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000168 450=00003800 628=000001e4 650=00003800
  146 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000158 450=00000000 628=000001d4 650=00000000
  147 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000148 450=00000800 628=000001c4 650=00000800
  148 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000138 450=00001000 628=000001b4 650=00001000
  149 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000128 628=000001a4 650=00001800
  150 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000118 450=00002000 628=00000194
  151 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000108 450=00002800 628=00000184 650=00002800
  152 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000f8 450=00003000 628=00000174 650=00003000
  153 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000e8 450=00003800 628=00000164 650=00003800
  154 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000d8 450=00000000 628=00000154 650=00000000
  155 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000c8 450=00000800 628=00000144 650=00000800
  156 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000b8 450=00001000 628=00000134 650=00001000
  157 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000a8 628=00000124 650=00001800
  158 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000098 450=00002000 628=00000114
  159 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000088 450=00002800 628=00000104 650=00002800
  160 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000078 450=00003000 628=000000f4 650=00003000
  161 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000068 450=00003800 628=000000e4 650=00003800
  162 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000058 450=00000000 628=000000d4 650=00000000
  163 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000048 450=00000800 628=000000c4 650=00000800
  164 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000038 450=00001000 628=000000b4 650=00001000
  165 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000028 628=000000a4 650=00001800
  166 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000018 450=00002000 628=00000094
  167 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000008 450=00002800 628=00000084 650=00002800
  168 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001f8 450=00002800 628=00000074 650=00003000
  169 (1 numbers): END; movi: no register; mov: no register; add: no register; window 450=00003000 650=00003800
  170 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001d8 450=00003800 628=00000054 650=00000000
  171 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001c8 450=00000000 628=00000044 650=00000800
  172 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001b8 450=00000800 628=00000034 650=00001000
  173 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001a8 450=00001000 628=00000024 650=00001800
  174 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000198 628=00000014
  175 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000188 450=00002000 628=00000004 650=00002800
  176 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000178 450=00002800 628=000001f4 650=00002800
  177 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000168 450=00003000 628=000001e4 650=00003000
  178 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000158 450=00003800 628=000001d4 650=00003800
  179 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000148 450=00000000 628=000001c4 650=00000000
  180 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000138 450=00000800 628=000001b4 650=00000800
  181 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000128 450=00001000 628=000001a4 650=00001000
  182 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000118 628=00000194 650=00001800
  183 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000108 450=00002000 628=00000184
  184 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000f8 450=00002800 628=00000174 650=00002800
  185 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000e8 450=00003000 628=00000164 650=00003000
  186 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000d8 450=00003800 628=00000154 650=00003800
  187 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000c8 450=00000000 628=00000144 650=00000000
  188 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000b8 450=00000800 628=00000134 650=00000800
  189 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000a8 450=00001000 628=00000124 650=00001000
  190 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000098 628=00000114 650=00001800
  191 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000088 450=00002000 628=00000104
  192 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000078 450=00002800 628=000000f4 650=00002800
  193 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000068 450=00003000 628=000000e4 650=00003000
  194 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000058 450=00003800 628=000000d4 650=00003800
  195 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000048 450=00000000 628=000000c4 650=00000000
  196 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000038 450=00000800 628=000000b4 650=00000800
  197 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000028 450=00001000 628=000000a4 650=00001000
  198 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000018 628=00000094 650=00001800
  199 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000008 450=00002000 628=00000084
  200 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001f8 450=00002000 628=00000074 650=00002800
  201 (1 numbers): END; movi: no register; mov: no register; add: no register; window 450=00002800 650=00003000
  202 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001d8 450=00003000 628=00000054 650=00003800
  203 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001c8 450=00003800 628=00000044 650=00000000
  204 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001b8 450=00000000 628=00000034 650=00000800
  205 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001a8 450=00000800 628=00000024 650=00001000
  206 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000198 450=00001000 628=00000014 650=00001800
  207 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000188 628=00000004
  208 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000178 450=00002000 628=000001f4
  209 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000168 450=00002800 628=000001e4 650=00002800
  210 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000158 450=00003000 628=000001d4 650=00003000
  211 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000148 450=00003800 628=000001c4 650=00003800
  212 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000138 450=00000000 628=000001b4 650=00000000
  213 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000128 450=00000800 628=000001a4 650=00000800
  214 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000118 450=00001000 628=00000194 650=00001000
  215 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000108 628=00000184 650=00001800
  216 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000f8 450=00002000 628=00000174
  217 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000e8 450=00002800 628=00000164 650=00002800
  218 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000d8 450=00003000 628=00000154 650=00003000
  219 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000c8 450=00003800 628=00000144 650=00003800
  220 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000b8 450=00000000 628=00000134 650=00000000
  221 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000a8 450=00000800 628=00000124 650=00000800
  222 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000098 450=00001000 628=00000114 650=00001000
  223 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000088 628=00000104 650=00001800
  224 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000078 450=00002000 628=000000f4
  225 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000068 450=00002800 628=000000e4 650=00002800
  226 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000058 450=00003000 628=000000d4 650=00003000
  227 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000048 450=00003800 628=000000c4 650=00003800
  228 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000038 450=00000000 628=000000b4 650=00000000
  229 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000028 450=00000800 628=000000a4 650=00000800
  230 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000018 450=00001000 628=00000094 650=00001000
  231 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000008 628=00000084 650=00001800
  232 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001f8 628=00000074
  233 (1 numbers): END; movi: no register; mov: no register; add: no register; window 450=00002000 650=00002800
  234 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001d8 450=00002800 628=00000054 650=00003000
  235 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001c8 450=00003000 628=00000044 650=00003800
  236 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001b8 450=00003800 628=00000034 650=00000000
  237 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000001a8 450=00000000 628=00000024 650=00000800
  238 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000198 450=00000800 628=00000014 650=00001000
  239 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000188 450=00001000 628=00000004 650=00001800
  240 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000178 628=000001f4 650=00001800
  241 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000168 450=00002000 628=000001e4
  242 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000158 450=00002800 628=000001d4 650=00002800
  243 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000148 450=00003000 628=000001c4 650=00003000
  244 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000138 450=00003800 628=000001b4 650=00003800
  245 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000128 450=00000000 628=000001a4 650=00000000
  246 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000118 450=00000800 628=00000194 650=00000800
  247 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000108 450=00001000 628=00000184 650=00001000
  248 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000f8 628=00000174 650=00001800
  249 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000e8 450=00002000 628=00000164
  250 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000d8 450=00002800 628=00000154 650=00002800
  251 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000c8 450=00003000 628=00000144 650=00003000
  252 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000b8 450=00003800 628=00000134 650=00003800
  253 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=000000a8 450=00000000 628=00000124 650=00000000
  254 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000098 450=00000800 628=00000114 650=00000800
  255 (1 numbers): END; movi: no register; mov: no register; add: no register; window 428=00000088 450=00001000 628=00000104 650=00001000
 sources (named: rK = seed of rK, CXX = preloaded control word, 0):
  9, 11..255 (246 numbers): END; mov r0; add+1 r0+1; or r1|r0; st r7; bnz not taken/not taken; bz not taken/not taken (r0 != 0 / r0 = 0); jr jumped to r0
  8 (1 numbers): TIMEOUT; mov 00000080; add+1 000000d1; or 5e015f7e; st r7; bnz not taken/not taken; bz not taken/not taken (r0 != 0 / r0 = 0); jr no landing
  10 (1 numbers): END; mov r1; add+1 5e015e5f; or r1|r0; st r7; bnz not taken/not taken; bz not taken/not taken (r0 != 0 / r0 = 0); jr jumped to r0
 addresses (ldi: which scratch marker was read; pld: which parameter-RAM marker; else the word read):
  9..255 (247 numbers): END; ldi addr r0; pld offset r0
  8 (1 numbers): END; ldi e90d1c00; pld 00000000
```

### 14.2.150 seqfirst (run 36)

ID: `D-seqfirst-36`. Device run 36. Experiment: [seqfirst](13-experiments.md#133122-seqfirst).

```
first mode-0 encode of the session: area 2e1c79573386228532df6ca2da 0x990 5bb85bb8 (END)
```

### 14.2.151 w2state (run 36)

ID: `D-w2state-36`. Device run 36. Experiment: [w2state](13-experiments.md#133123-w2state).

```
wait / wait2 passes per controller state (short timeout):
  first program of the session                         wait [4, 11]; wait2 [5]
  again, no reset                                      wait [4, 11]; wait2 [5]
  after a soft reset                                   wait [11]; wait2 [5]
  0x044 = 0xa5a5, 0x844 = 0x25 (CPU, before)           wait [11]; wait2 [5]
  command flag pending, 0x044 bit 0                    wait [0, 11]; wait2 [5, 20]
  command flag pending, 0x044 bit 1                    wait [0, 11]; wait2 [5]
  command flag pending, 0x044 bit 2                    wait [0, 11]; wait2 [5]
  command flag pending, 0x044 bit 3                    wait [0, 11]; wait2 [5]
  command flag pending, 0x044 bit 4                    wait [0, 11]; wait2 [5]
  command flag pending, 0x044 bit 5                    wait [0, 11]; wait2 [5]
  command flag pending, 0x044 bit 6                    wait [0, 11]; wait2 [5]
  command flag pending, 0x044 bit 7                    wait [0, 11]; wait2 [5]
  command flag pending, 0x044 bit 8                    wait [0, 11]; wait2 [5]
  command flag pending, 0x044 bit 9                    wait [0, 11]; wait2 [5]
  command flag pending, 0x044 bit 10                   wait [0, 11]; wait2 [5]
  command flag pending, 0x044 bit 11                   wait [0, 11]; wait2 [5]
  command flag pending, 0x044 bit 12                   wait [0, 11]; wait2 [5]
  command flag pending, 0x044 bit 13                   wait [0, 11]; wait2 [5]
  command flag pending, 0x044 bit 14                   wait [0, 11]; wait2 [5]
  command flag pending, 0x044 bit 15                   wait [0, 11]; wait2 [5]
  command flag pending, 0x844 = 0x2f                   wait [0, 11]; wait2 [5]
  address flag pending                                 wait [1, 11]; wait2 [5]
  DMA flag pending                                     wait [2, 11]; wait2 [5]
  ECC_DONE pending                                     wait [4, 11]; wait2 [5]
  status flag pending                                  wait [5, 11]; wait2 [5]
  after a second-set READ ID                           wait [11]; wait2 [5]
  channel-1 pins switched                              wait [11]; wait2 [5]
  after a page read of CE0                             wait [4, 8, 11]; wait2 [5]
  C10 = 0x0                                            wait [11]; wait2 [5]
  C10 = 0xff                                           wait [11]; wait2 [5]
  no chip selected                                     wait [11]; wait2 [5]
  CE0 selected                                         wait [11]; wait2 [5]
  CE1 selected                                         wait [11]; wait2 [5]
  start word 0x3ff5 (step n = 15)                      wait [11]; wait2 [5]
```

### 14.2.152 c50b8 (run 36)

ID: `D-c50b8-36`. Device run 36. Experiment: [c50b8](13-experiments.md#133124-c50b8).

```
C50 around runs (bit 8 = 0x100):
  END only (1)                   C50 before ffff1800 after ffff1800
  END only (2)                   C50 before ffff1800 after ffff1800
  soft reset, then END           C50 before ffff1800 after ffff1800
  sequencer reset, then END      C50 before ffff1800 after ffff1800
  fault (wait 10, C58 0)         C50 before ffff1800 after ffff1800
  second-set command             C50 before ffff1800 after ffff1840
  ECC decode                     C50 before ffff1840 after ffff1850
  END only, repeat 0             C50 before ffff1850 after ffff1850
  END only, repeat 1             C50 before ffff1850 after ffff1850
  END only, repeat 2             C50 before ffff1850 after ffff1850
  END only, repeat 3             C50 before ffff1850 after ffff1850
  END only, repeat 4             C50 before ffff1850 after ffff1850
  END only, repeat 5             C50 before ffff1850 after ffff1850
  END only, repeat 6             C50 before ffff1850 after ffff1850
  END only, repeat 7             C50 before ffff1850 after ffff1850
  END only, repeat 8             C50 before ffff1850 after ffff1850
  END only, repeat 9             C50 before ffff1850 after ffff1850
```

### 14.2.153 startregion (run 36)

ID: `D-startregion-36`. Device run 36. Experiment: [startregion](13-experiments.md#133125-startregion).

```
start words and NAND regions (page 0 of each block):
  CE0 block    0 (user): same under all four: END 335us data erased res 0 0 0 0 st 3c000 3c000 3c000 3c000
  CE0 block    1 (user): same under all four: END 309us data 1b4a2c73d4 res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE0 block    2 (user): same under all four: END 310us data 1fdd59bb55 res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE0 block  100 (user): same under all four: END 320us data fcbc08c0c1 res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE0 block 2000 (user): same under all four: END 309us data 11fd621527 res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE0 block 3000 (user): same under all four: END 324us data erased res 0 0 0 0 st 3c000 3c000 3c000 3c000
  CE0 block 3871 (user): same under all four: END 319us data 4da47e8d24 res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE0 block 3872 (above user): same under all four: END 310us data 22689fd562 res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE0 block 3873 (above user): same under all four: END 310us data 106b5b49f3 res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE0 block 3900 (above user): same under all four: END 310us data 39d0495b05 res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE0 block 4000 (above user): same under all four: END 335us data erased res 0 0 0 0 st 3c000 3c000 3c000 3c000
  CE0 block 4064 (above user): same under all four: END 324us data erased res 0 0 0 0 st 3c000 3c000 3c000 3c000
  CE0 block 4094 (above user): same under all four: END 309us data b468b9c9a4 res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE0 block 4095 (above user): same under all four: END 310us data b468b9c9a4 res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE1 block    0 (user): same under all four: END 335us data erased res 0 0 0 0 st 3c000 3c000 3c000 3c000
  CE1 block    1 (user): same under all four: END 320us data 29f1115a1f res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE1 block    2 (user): same under all four: END 310us data 794dcfd051 res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE1 block  100 (user): same under all four: END 309us data e68af4508a res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE1 block 2000 (user): same under all four: END 319us data d8fb4f2f78 res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE1 block 3000 (user): same under all four: END 324us data erased res 0 0 0 0 st 3c000 3c000 3c000 3c000
  CE1 block 3871 (user): same under all four: END 320us data fafefb49e7 res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE1 block 3872 (above user): same under all four: END 320us data dcba5b945d res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE1 block 3873 (above user): same under all four: END 319us data 34748cf023 res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE1 block 3900 (above user): same under all four: END 335us data erased res 0 0 0 0 st 3c000 3c000 3c000 3c000
  CE1 block 4000 (above user): same under all four: END 324us data erased res 0 0 0 0 st 3c000 3c000 3c000 3c000
  CE1 block 4064 (above user): same under all four: END 325us data erased res 0 0 0 0 st 3c000 3c000 3c000 3c000
  CE1 block 4094 (above user): same under all four: END 319us data 529cdc3313 res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE1 block 4095 (above user): same under all four: END 319us data 529cdc3313 res 0 0 0 0 st 803c000 803c000 803c000 803c000
  CE0 user block 3002, written under fff5: erase END 179us NSTAT c0, program END 478us NSTAT c0; read fff5 END data ==== res 0 0 0 0 | ffb5 END data ==== res 0 0 0 0
  CE0 user block 3002, written under ffb5: erase END 1141us NSTAT c0, program END 478us NSTAT c0; read fff5 END data ==== res 0 0 0 0 | ffb5 END data ==== res 0 0 0 0
  CE0 block 4000 (above user, read erased), written under fff5: erase END 1142us NSTAT c0, program END 478us NSTAT c0; read fff5 END data ==== res 0 0 0 0 | ffb5 END data ==== res 0 0 0 0
  CE0 block 4000 (above user, read erased), written under ffb5: erase END 1142us NSTAT c0, program END 479us NSTAT c0; read fff5 END data ==== res 0 0 0 0 | ffb5 END data ==== res 0 0 0 0
  CE1 block 3900 (above user, read erased), written under fff5: erase END 1155us NSTAT c0, program END 478us NSTAT c0; read fff5 END data ==== res 0 0 0 0 | ffb5 END data ==== res 0 0 0 0
  CE1 block 3900 (above user, read erased), written under ffb5: erase END 1155us NSTAT c0, program END 478us NSTAT c0; read fff5 END data ==== res 0 0 0 0 | ffb5 END data ==== res 0 0 0 0
```

### 14.2.154 bufport (run 36)

ID: `D-bufport-36`. Device run 36. Experiment: [bufport](13-experiments.md#133126-bufport).

```
FMC_FIFO (0x080) as a port onto the buffers (pattern i*29+7):
  data buffer 0, 0x340       once  END: first words 00000000000000000000000000000000; matches the buffer from byte None for 0 words; FMC_28 [4, 132, 260, 388, 4]
  data buffer 0, 0x340       twice END: first words 00000000000000000000000000000000; matches the buffer from byte None for 0 words; FMC_28 [7, 123, 239, 99, 215]; second read first words 00000000000000000000000000000000 (restarts)
  data buffer 1, 0x340       once  END: first words 00000000000000000000000000000000; matches the buffer from byte None for 0 words; FMC_28 [4, 132, 260, 388, 4]
  data buffer 1, 0x340       twice END: first words 00000000000000000000000000000000; matches the buffer from byte None for 0 words; FMC_28 [7, 123, 239, 99, 215]; second read first words 00000000000000000000000000000000 (restarts)
  data buffer 0, no 0x340    once  END: first words 00000000000000000000000000000000; matches the buffer from byte None for 0 words; FMC_28 [4, 132, 260, 388, 4]
  data buffer 0, no 0x340    twice END: first words 00000000000000000000000000000000; matches the buffer from byte None for 0 words; FMC_28 [4, 132, 260, 388, 4]; second read first words 00000000000000000000000000000000 (restarts)
  parity buffer 0, 0x340     once  END: first words 00000000000000000000000000000000; matches the buffer from byte None for 0 words; FMC_28 [4]
  parity buffer 0, 0x340     twice END: first words 00000000000000000000000000000000; matches the buffer from byte None for 0 words; FMC_28 [199]; second read first words 00000000000000000000000000000000 (restarts)
  write 0x080, FMC_DATA_SEL 1                END: buffer after: first words 0724415e7b98b5d2ef0c294663809dba; 2 of 128 words carry the written marker
  write 0x080, FMC_DATA_SEL 1, FMCTRL1 0x340 END: buffer after: first words eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee; 26 of 128 words carry the written marker
```

### 14.2.155 c58b2 (run 36)

ID: `D-c58b2-36`. Device run 36. Experiment: [c58b2](13-experiments.md#133127-c58b2).

```
C58 bit 1, three runs per case (C58 / C4C / C5C as read before each start):
  CPU C58=2, C4C                                       END (C58 2 C4C 40 C5C 0) | END (C58 2 C4C 40 C5C 0) | END (C58 2 C4C 40 C5C 0)
  CPU C4C, C58=2                                       END (C58 2 C4C 40 C5C 0) | END (C58 2 C4C 40 C5C 0) | END (C58 2 C4C 40 C5C 0)
  CPU C58=2, program C4C (run-8 control, no clean-up)  END (C58 2 C4C 0 C5C 0) | END (C58 2 C4C 18 C5C 0) | END (C58 2 C4C 18 C5C 0)
  CPU C58=2, program C4C (run-8 control, job steps)    END (C58 2 C4C 0 C5C 0) | END (C58 0 C4C 0 C5C 0) | END (C58 0 C4C 0 C5C 0)
  program C58=2 and C4C                                END (C58 0 C4C 0 C5C 0) | END (C58 2 C4C 20 C5C 0) | END (C58 2 C4C 20 C5C 0)
  CPU C58=3, C4C                                       END (C58 3 C4C 40 C5C 0) | END, then the vector (C58 3 C4C 40 C5C 1) | END, then the vector (C58 3 C4C 40 C5C 1)
  CPU C58=6, C4C                                       END, then the vector (C58 6 C4C 40 C5C 0) | END (C58 6 C4C 40 C5C 0) | END (C58 6 C4C 40 C5C 0)
```

### 14.2.156 stprev (run 36)

ID: `D-stprev-36`. Device run 36. Experiment: [stprev](13-experiments.md#133128-stprev).

```
st [0xd70], rN with N above 7, by the instruction before it:
  N=9   after seed only (last write movi r7)               st stored 88008888 (r7 seed)
  N=14  after seed only (last write movi r7)               st stored 88008888 (r7 seed)
  N=200 after seed only (last write movi r7)               st stored 88008888 (r7 seed)
  N=9   after movi r0, #0xa0a0a0a0                         st stored a0a0a0a0 ()
  N=14  after movi r0, #0xa0a0a0a0                         st stored a0a0a0a0 ()
  N=200 after movi r0, #0xa0a0a0a0                         st stored a0a0a0a0 ()
  N=9   after movi r3, #0x33333333                         st stored 33333333 ()
  N=14  after movi r3, #0x33333333                         st stored 33333333 ()
  N=200 after movi r3, #0x33333333                         st stored 33333333 ()
  N=9   after movi r7, #0x77777777                         st stored 77777777 ()
  N=14  after movi r7, #0x77777777                         st stored 77777777 ()
  N=200 after movi r7, #0x77777777                         st stored 77777777 ()
  N=9   after add r2, r2, #1                               st stored 33003334 ()
  N=14  after add r2, r2, #1                               st stored 33003334 ()
  N=200 after add r2, r2, #1                               st stored 33003334 ()
  N=9   after mov r4, r1                                   st stored 22002222 (r1 seed)
  N=14  after mov r4, r1                                   st stored 22002222 (r1 seed)
  N=200 after mov r4, r1                                   st stored 22002222 (r1 seed)
  N=9   after ld r5, [0xd74] (holds 0x5d5d5d5d)            st stored 5d5d5d5d ()
  N=14  after ld r5, [0xd74] (holds 0x5d5d5d5d)            st stored 5d5d5d5d ()
  N=200 after ld r5, [0xd74] (holds 0x5d5d5d5d)            st stored 5d5d5d5d ()
  N=9   after pld r6, [r0] (r0 = 0xd78, holds 0x6e6e6e6e)  st stored 6e6e6e6e ()
  N=14  after pld r6, [r0] (r0 = 0xd78, holds 0x6e6e6e6e)  st stored 6e6e6e6e ()
  N=200 after pld r6, [r0] (r0 = 0xd78, holds 0x6e6e6e6e)  st stored 6e6e6e6e ()
  N=9   after ldi r1, [r0] (scratch 0xcafe0001)            st stored cafe0001 ()
  N=14  after ldi r1, [r0] (scratch 0xcafe0001)            st stored cafe0001 ()
  N=200 after ldi r1, [r0] (scratch 0xcafe0001)            st stored cafe0001 ()
  N=9   after wr [0xd7c], #0x7c7c7c7c                      st stored 11001111 (r0 seed)
  N=14  after wr [0xd7c], #0x7c7c7c7c                      st stored 11001111 (r0 seed)
  N=200 after wr [0xd7c], #0x7c7c7c7c                      st stored 11001111 (r0 seed)
  N=9   after st [0xd7c], r5                               st stored 66006666 (r5 seed)
  N=14  after st [0xd7c], r5                               st stored 66006666 (r5 seed)
  N=200 after st [0xd7c], r5                               st stored 66006666 (r5 seed)
  N=9   after wait 11 (passes)                             st stored 88008888 (r7 seed)
  N=14  after wait 11 (passes)                             st stored 88008888 (r7 seed)
  N=200 after wait 11 (passes)                             st stored 88008888 (r7 seed)
  N=9   after nop                                          st stored 11001111 (r0 seed)
  N=14  after nop                                          st stored 11001111 (r0 seed)
  N=200 after nop                                          st stored 11001111 (r0 seed)
```

### 14.2.157 tmwait2 (run 36)

ID: `D-tmwait2-36`. Device run 36. Experiment: [tmwait2](13-experiments.md#133129-tmwait2).

```
tmwait and C14:
  run 1                      C14 before (CPU) ff | start ff, after each tmwait 00 ff 00 ff | 9702 us
  run 2                      C14 before (CPU) ff | start ff, after each tmwait 00 ff 00 ff | 9704 us
  run 3                      C14 before (CPU) ff | start ff, after each tmwait 00 ff 00 ff | 9704 us
  run after a timeout run    C14 before (CPU) ff | start ff, after each tmwait 00 ff 00 ff | 9713 us
  run after a soft reset     C14 before (CPU) ff | start ff, after each tmwait 00 ff 00 ff | 9721 us
```

### 14.2.158 irqop (run 36)

ID: `D-irqop-36`. Device run 36. Experiment: [irqop](13-experiments.md#133130-irqop).

```
ERROR (tool bug, please report): USBTimeoutError(110, 'Operation timed out')
```

### 14.2.159 done5b (run 37)

ID: `D-done5b-37`. Device run 37. Experiment: [done5b](13-experiments.md#133131-done5b).

```
ECC_DONE (encode / decode) per mode, each case after a soft reset:
  0x0000180: enc 33, enc+dec 33/35, dec(0x180)+enc 33
  0x0200180: enc 13, enc+dec 13/15, dec(0x180)+enc 13
  0x0400180: enc 13, enc+dec 13/15, dec(0x180)+enc 13
  0x0800180: enc 13, enc+dec 13/15, dec(0x180)+enc 13
  0x1000180: enc 13, enc+dec 13/15, dec(0x180)+enc 13
```

### 14.2.160 rsresult (run 37)

ID: `D-rsresult-37`. Device run 37. Experiment: [rsresult](13-experiments.md#133132-rsresult).

```
ECC_RESULT before -> after an encode:
  before: nothing              0x8100: 00000000 -> 00000000 | 0x8104: 00000000 -> 00000000 | 0x8108: 00000000 -> 00000000 | 0x180: 00000000 -> 00000000
  before: BCH decode, 0 bits   0x8100: 00000000 -> 00000000 | 0x8104: 00000000 -> 00000000 | 0x8108: 00000000 -> 00000000 | 0x180: 00000000 -> 00000000
  before: BCH decode, 3 bits   0x8100: 00030000 -> 00000000 | 0x8104: 00030000 -> 00000000 | 0x8108: 00030000 -> 00000000 | 0x180: 00030000 -> 00030000
  before: RS decode, 0 bytes   0x8100: 00000000 -> 00000000 | 0x8104: 00000000 -> 00000000 | 0x8108: 00000000 -> 00000000 | 0x180: 00000000 -> 00030000
  before: RS decode, 2 bytes   0x8100: 00020000 -> 00020000 | 0x8104: 00020000 -> 00020000 | 0x8108: 00020000 -> 00020000 | 0x180: 00020000 -> 00030000
```

### 14.2.161 rschip (run 37)

ID: `D-rschip-37`. Device run 37. Experiment: [rschip](13-experiments.md#133133-rschip).

```
Reed-Solomon on the chip (block 3070):
  erase: END NSTAT c0
  page 0 mode 0x8100: program END NSTAT c0; read END data ==== results 0 11 11 11; chunk 0 as stored 2a5c0a397c8a9f585350ffffffffffff
  page 1 mode 0x8104: program END NSTAT c0; read END data ==== results 0 11 11 11; chunk 0 as stored 905dc73be102afc7ccbe2db808e706ff
  page 2 mode 0x180: program END NSTAT c0; read END data ==== results 0 0 0 0; chunk 0 as stored ffffff1fce51281e06ae8c16922b553a
  page 3 mode 0x8100: program END NSTAT c0; second program (4 zero bytes per sector) END NSTAT c0; read END data =xxx results 40000 11 11 11; chunk 0 as stored 2a5c0a397c8a9f585350ffffffffffff
  page 4 mode 0x8100: program END NSTAT c0; second program (5 zero bytes per sector) END NSTAT c0; read END data xxxx results 11 11 11 11; chunk 0 as stored 2a5c0a397c8a9f585350ffffffffffff
  page 5 mode 0x8104: program END NSTAT c0; second program (6 zero bytes per sector) END NSTAT c0; read END data =xxx results 60000 11 11 11; chunk 0 as stored 905dc73be102afc7ccbe2db808e706ff
  page 6 mode 0x8104: program END NSTAT c0; second program (7 zero bytes per sector) END NSTAT c0; read END data xxxx results 11 11 11 11; chunk 0 as stored 905dc73be102afc7ccbe2db808e706ff
  RS-16 into slot 3: parity buffer 0 4e2923086f4cfc4b7b77718291507950ffa89d05ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff | parity buffer 1 ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff | decode of slot 3 with 2 bad bytes: ECC_RESULT 00020000, data corrected
```

### 14.2.162 miscorr (run 37)

ID: `D-miscorr-37`. Device run 37. Experiment: [miscorr](13-experiments.md#133134-miscorr).

```
decodes far beyond the code's strength:
  BCH   uncorrectable 240, corrected, right data 0, MISCORRECTED 0, UNDETECTED (reported no error) 0
  RS-8  uncorrectable 240, corrected, right data 0, MISCORRECTED 0, UNDETECTED (reported no error) 0
```

### 14.2.163 fullpage (run 37)

ID: `D-fullpage-37`. Device run 37. Experiment: [fullpage](13-experiments.md#133135-fullpage).

```
whole 4 KiB pages, metadata bytes of the two units:
  CE0 block    1 page  0: END metadata unit 0 ffffffffffffffff0080ffff unit 1 ffffffffffffffff0080ffff (SAME)
  CE0 block    1 page  1: END metadata unit 0 ffffffffffffffff0080ffff unit 1 ffffffffffffffff0080ffff (SAME)
  CE0 block    1 page 64: END metadata unit 0 f7ffffffffffffff0080ffff unit 1 f7ffffffffffffff0080ffff (SAME)
  CE0 block    2 page  0: END metadata unit 0 efffffffffffffff0080ffff unit 1 efffffffffffffff0080ffff (SAME)
  CE0 block    2 page  1: END metadata unit 0 efffffffffffffff0080ffff unit 1 efffffffffffffff0080ffff (SAME)
  CE0 block    2 page 64: END metadata unit 0 e7ffffffffffffff0080ffff unit 1 e7ffffffffffffff0080ffff (SAME)
  CE0 block   10 page  0: END metadata unit 0 00d01600ef4e0100ff40ffff unit 1 00d01600ef4e0100ff40ffff (SAME)
  CE0 block   10 page  1: END metadata unit 0 04d01600ef4e0100ff40ffff unit 1 04d01600ef4e0100ff40ffff (SAME)
  CE0 block   10 page 64: END metadata unit 0 00d11600ef4e0100ff40ffff unit 1 00d11600ef4e0100ff40ffff (SAME)
  CE0 block  100 page  0: END metadata unit 0 00140200ff340100ff40ffff unit 1 00140200ff340100ff40ffff (SAME)
  CE0 block  100 page  1: END metadata unit 0 04140200ff340100ff40ffff unit 1 04140200ff340100ff40ffff (SAME)
  CE0 block  100 page 64: END metadata unit 0 00150200ff340100ff40ffff unit 1 00150200ff340100ff40ffff (SAME)
  CE0 block  500 page  0: END metadata unit 0 004e0b0019e90100ff40ffff unit 1 004e0b0019e90100ff40ffff (SAME)
  CE0 block  500 page  1: END metadata unit 0 044e0b0019e90100ff40ffff unit 1 044e0b0019e90100ff40ffff (SAME)
  CE0 block  500 page 64: END metadata unit 0 004f0b0019e90100ff40ffff unit 1 004f0b0019e90100ff40ffff (SAME)
  CE0 block 1000 page  0: END metadata unit 0 00d211008a780100ff40ffff unit 1 00d211008a780100ff40ffff (SAME)
  CE0 block 1000 page  1: END metadata unit 0 04d211008a780100ff40ffff unit 1 04d211008a780100ff40ffff (SAME)
  CE0 block 1000 page 64: END metadata unit 0 00d311008c780100ff40ffff unit 1 00d311008c780100ff40ffff (SAME)
  CE0 block 2000 page  0: END metadata unit 0 00cc1800a3100100ff40ffff unit 1 00cc1800a3100100ff40ffff (SAME)
  CE0 block 2000 page  1: END metadata unit 0 04cc1800a3100100ff40ffff unit 1 04cc1800a3100100ff40ffff (SAME)
  CE0 block 2000 page 64: END metadata unit 0 00cd1800a3100100ff40ffff unit 1 00cd1800a3100100ff40ffff (SAME)
  CE1 block    1 page  0: END metadata unit 0 ffffffffffffffff0080ffff unit 1 ffffffffffffffff0080ffff (SAME)
  CE1 block    1 page  1: END metadata unit 0 ffffffffffffffff0080ffff unit 1 ffffffffffffffff0080ffff (SAME)
  CE1 block    1 page 64: END metadata unit 0 f7ffffffffffffff0080ffff unit 1 f7ffffffffffffff0080ffff (SAME)
  CE1 block    2 page  0: END metadata unit 0 efffffffffffffff0080ffff unit 1 efffffffffffffff0080ffff (SAME)
  CE1 block    2 page  1: END metadata unit 0 efffffffffffffff0080ffff unit 1 efffffffffffffff0080ffff (SAME)
  CE1 block    2 page 64: END metadata unit 0 e7ffffffffffffff0080ffff unit 1 e7ffffffffffffff0080ffff (SAME)
  CE1 block   10 page  0: END metadata unit 0 01d01600ef4e0100ff40ffff unit 1 01d01600ef4e0100ff40ffff (SAME)
  CE1 block   10 page  1: END metadata unit 0 05d01600ef4e0100ff40ffff unit 1 05d01600ef4e0100ff40ffff (SAME)
  CE1 block   10 page 64: END metadata unit 0 01d11600ef4e0100ff40ffff unit 1 01d11600ef4e0100ff40ffff (SAME)
  CE1 block  100 page  0: END metadata unit 0 01140200ff340100ff40ffff unit 1 01140200ff340100ff40ffff (SAME)
  CE1 block  100 page  1: END metadata unit 0 05140200ff340100ff40ffff unit 1 05140200ff340100ff40ffff (SAME)
  CE1 block  100 page 64: END metadata unit 0 01150200ff340100ff40ffff unit 1 01150200ff340100ff40ffff (SAME)
  CE1 block  500 page  0: END metadata unit 0 014e0b0019e90100ff40ffff unit 1 014e0b0019e90100ff40ffff (SAME)
  CE1 block  500 page  1: END metadata unit 0 054e0b0019e90100ff40ffff unit 1 054e0b0019e90100ff40ffff (SAME)
  CE1 block  500 page 64: END metadata unit 0 014f0b0019e90100ff40ffff unit 1 014f0b0019e90100ff40ffff (SAME)
  CE1 block 1000 page  0: END metadata unit 0 01d211008a780100ff40ffff unit 1 01d211008a780100ff40ffff (SAME)
  CE1 block 1000 page  1: END metadata unit 0 05d211008a780100ff40ffff unit 1 05d211008a780100ff40ffff (SAME)
  CE1 block 1000 page 64: END metadata unit 0 01d311008c780100ff40ffff unit 1 01d311008c780100ff40ffff (SAME)
  CE1 block 2000 page  0: END metadata unit 0 01cc1800a3100100ff40ffff unit 1 01cc1800a3100100ff40ffff (SAME)
  CE1 block 2000 page  1: END metadata unit 0 05cc1800a3100100ff40ffff unit 1 05cc1800a3100100ff40ffff (SAME)
  CE1 block 2000 page 64: END metadata unit 0 01cd1800a3100100ff40ffff unit 1 01cd1800a3100100ff40ffff (SAME)
```

### 14.2.164 dmactl (run 37)

ID: `D-dmactl-37`. Device run 37. Experiment: [dmactl](13-experiments.md#133136-dmactl).

```
FMC_DMA_CTRL and data selects on a page read (block 3051):
  program: END NSTAT c0
  drain 0x100 FMC_DMA_CTRL -     END      317us data ====; FMC_DMA_CTRL after 00000007
  drain 0x100 FMC_DMA_CTRL 0x1   END      373us data ====; FMC_DMA_CTRL after 00000001
  drain 0x100 FMC_DMA_CTRL 0x2   FAULT  29495us data =xxx; FMC_DMA_CTRL after 00000042
  drain 0x100 FMC_DMA_CTRL 0x4   FAULT  29477us data =xxx; FMC_DMA_CTRL after 00000084
  drain 0x100 FMC_DMA_CTRL 0x8   END      438us data ====; FMC_DMA_CTRL after 00000008
  drain 0x100 FMC_DMA_CTRL 0x10  FAULT  48965us data xxxx; FMC_DMA_CTRL after 00000010
  drain 0x100 FMC_DMA_CTRL 0x20  FAULT  48974us data xxxx; FMC_DMA_CTRL after 00000020
  drain 0x100 FMC_DMA_CTRL 0x40  FAULT  48971us data xxxx; FMC_DMA_CTRL after 00000040
  drain 0x100 FMC_DMA_CTRL 0x80  FAULT  48972us data xxxx; FMC_DMA_CTRL after 00000080
  drain 0x1e0 FMC_DMA_CTRL -     END      451us data ====; FMC_DMA_CTRL after 00000000
  drain 0x1e0 FMC_DMA_CTRL 0x1   END      383us data ====; FMC_DMA_CTRL after 00000001
  drain 0x1e0 FMC_DMA_CTRL 0x2   FAULT  29486us data =xxx; FMC_DMA_CTRL after 00000042
  drain 0x1e0 FMC_DMA_CTRL 0x4   FAULT  29486us data =xxx; FMC_DMA_CTRL after 00000084
  drain 0x1e0 FMC_DMA_CTRL 0x8   END      437us data ====; FMC_DMA_CTRL after 00000008
  drain 0x1e0 FMC_DMA_CTRL 0x10  FAULT  48966us data xxxx; FMC_DMA_CTRL after 00000010
  drain 0x1e0 FMC_DMA_CTRL 0x20  FAULT  48966us data xxxx; FMC_DMA_CTRL after 00000020
  drain 0x1e0 FMC_DMA_CTRL 0x40  FAULT  48967us data xxxx; FMC_DMA_CTRL after 00000040
  drain 0x1e0 FMC_DMA_CTRL 0x80  FAULT  48970us data xxxx; FMC_DMA_CTRL after 00000080
  combined selects, data buffers alternating (read 0x201 / 0x102, DMA 0x102 / 0x201): END data ==== results 0 0 0 0
```

### 14.2.165 fmcstat2 (run 37)

ID: `D-fmcstat2-37`. Device run 37. Experiment: [fmcstat2](13-experiments.md#133137-fmcstat2).

```
FMC_STATUS bits 12, 13, 18, 19:
  nothing                  bits 12/13/18/19 after the 16-byte read 1010, after the operation 1010 (END)
  DMA into data buffer 0   bits 12/13/18/19 after the 16-byte read 1010, after the operation 1111 (END)
  DMA out of data buffer 0 bits 12/13/18/19 after the 16-byte read 1010, after the operation 1010 (END)
  DMA into parity buffer 0 bits 12/13/18/19 after the 16-byte read 1010, after the operation 1111 (END)
  chip read 512 bytes      bits 12/13/18/19 after the 16-byte read 1010, after the operation 0000 (END)
  ECC encode               bits 12/13/18/19 after the 16-byte read 1010, after the operation 1010 (END)
  status latch             bits 12/13/18/19 after the 16-byte read 1010, after the operation 1111 (END)
  command                  bits 12/13/18/19 after the 16-byte read 1010, after the operation 1010 (END)
  second-set data read     bits 12/13/18/19 after the 16-byte read 1010, after the operation 1010 (END)
  set-up word              bits 12/13/18/19 after the 16-byte read 1010, after the operation 1111 (END)
```

### 14.2.166 ch1buf (run 37)

ID: `D-ch1buf-37`. Device run 37. Experiment: [ch1buf](13-experiments.md#133138-ch1buf).

```
buffers of the second register set (DMA via 0x400):
  second set, parity buffer 0 END: back = pattern; first set's parity buffer 0 CHANGED (c5d2dfecf90613202d3a4754616e7b88)
  second set, parity buffer 1 END: back = pattern; first set's parity buffer 0 CHANGED (ffffffffffffffffffffffffffffffff)
  second set, data buffer 1   END: back = pattern; first set's parity buffer 0 CHANGED (ffffffffffffffffffffffffffffffff)
  second set, data buffer 0   END: back = pattern; first set's parity buffer 0 CHANGED (ffffffffffffffffffffffffffffffff)
```

### 14.2.167 evsweep3 (run 37)

ID: `D-evsweep3-37`. Device run 37. Experiment: [evsweep3](13-experiments.md#133139-evsweep3).

```
wait / wait2 passes during further operations (short timeout):
  second-set DMA in flight       wait [8, 11]; wait2 [5]
  meta encode in flight          wait [11]; wait2 [5]
  meta decode in flight          wait [11]; wait2 [5]
  ECC decode in flight           wait [4, 11]; wait2 [5]
  chip data write in flight      wait [11]; wait2 [5]
  parity-buffer DMA in flight    wait [11]; wait2 [5]
```

### 14.2.168 blk100b (run 37)

ID: `D-blk100b-37`. Device run 37. Experiment: [blk100b](13-experiments.md#133140-blk100b).

```
0x100 block words and 0x3a8..0x3b4:
  default            END  10153us; wait 10 timeout, wait2 33 timeout, wait2 5 pass
  0x1bc = 0          END  10169us; wait 10 timeout, wait2 33 timeout, wait2 5 pass
  0x1c0 = 0          END  10154us; wait 10 timeout, wait2 33 timeout, wait2 5 pass
  0x1c4 = 0          END  10157us; wait 10 timeout, wait2 33 timeout, wait2 5 pass
  0x1c8 = 0          END  10158us; wait 10 timeout, wait2 33 timeout, wait2 5 pass
  0x1cc = 0          END  10145us; wait 10 timeout, wait2 33 timeout, wait2 5 pass
  0x1d0 = 0          END  10154us; wait 10 timeout, wait2 33 timeout, wait2 5 pass
  0x1d4 = 0          END  10143us; wait 10 timeout, wait2 33 timeout, wait2 5 pass
  0x1d8 = 0          END  10156us; wait 10 timeout, wait2 33 timeout, wait2 5 pass
  0x1b8 = 1          END  10158us; wait 10 timeout, wait2 33 timeout, wait2 5 pass
  0x1cc = 0xffffff   END  10157us; wait 10 timeout, wait2 33 timeout, wait2 5 pass
  0x1d0 = 0xffffff   END  10143us; wait 10 timeout, wait2 33 timeout, wait2 5 pass
  0x3a8..0x3b4: CPU wrote c0de000x; sequencer read ffffffff ffffffff ffffffff ffffffff; sequencer wrote 3a83a80x, read back ffffffff ffffffff ffffffff ffffffff; CPU reads after: 00000000 00000000 00000000 00000000
```

### 14.2.169 straps (run 37)

ID: `D-straps-37`. Device run 37. Experiment: [straps](13-experiments.md#133141-straps).

```
GPIO / PHY words: PCON1 11111100 PDAT1 00000003 PCON3 33331222 PDAT3 00000017 PHY+0x28 0000000f; P1.0 1 P1.1 1; P3.6 0 P3.7 0 -> boot device SPI flash, SPI port 0; PHY+0x28 bit 0 = 1
```

### 14.2.170 irqop (run 37)

ID: `D-irqop-37`. Device run 37. Experiment: [irqop](13-experiments.md#133130-irqop).

```
0x044 bit x operation (flag left pending): CS_IRQ bit 4, VIC0 line 20, wait event 15:
  none      command: b4 0 V0.20 0 ev15 timeout | address: b4 0 V0.20 0 ev15 timeout | data read: b4 0 V0.20 0 ev15 timeout | status latch: b4 0 V0.20 0 ev15 timeout | DMA: b4 0 V0.20 0 ev15 timeout
  bit 0     command: b4 1 V0.20 1 ev15 timeout | address: b4 0 V0.20 0 ev15 timeout | data read: b4 0 V0.20 0 ev15 timeout | status latch: b4 1 V0.20 0 ev15 timeout | DMA: b4 0 V0.20 0 ev15 timeout
  bit 1     command: b4 0 V0.20 0 ev15 timeout | address: b4 1 V0.20 1 ev15 timeout | data read: b4 0 V0.20 0 ev15 timeout | status latch: b4 0 V0.20 0 ev15 timeout | DMA: b4 0 V0.20 0 ev15 timeout
  bit 2     command: b4 0 V0.20 0 ev15 timeout | address: b4 0 V0.20 0 ev15 timeout | data read: b4 1 V0.20 1 ev15 timeout | status latch: b4 0 V0.20 0 ev15 timeout | DMA: b4 0 V0.20 0 ev15 timeout
  bit 11    command: b4 0 V0.20 0 ev15 timeout | address: b4 0 V0.20 0 ev15 timeout | data read: b4 0 V0.20 0 ev15 timeout | status latch: b4 0 V0.20 0 ev15 timeout | DMA: b4 1 V0.20 1 ev15 timeout
  bit 12    command: b4 0 V0.20 0 ev15 timeout | address: b4 0 V0.20 0 ev15 timeout | data read: b4 0 V0.20 0 ev15 timeout | status latch: b4 1 V0.20 1 ev15 timeout | DMA: b4 0 V0.20 0 ev15 timeout
```

### 14.2.171 w2vic (run 38)

ID: `D-w2vic-38`. Device run 38. Experiment: [w2vic](13-experiments.md#133142-w2vic).

```
wait2 conditions against the raw interrupt lines (VIC0 0..31, VIC1 32..63):
  idle                                 FAULT: wait2 passed [5]; raw lines set [5]; agree 64 of 64
  command flag pending, 0x044 bit 0    FAULT: wait2 passed [5, 20]; raw lines set [5, 20]; agree 64 of 64
  DMA flag pending, 0x044 bit 11       FAULT: wait2 passed [5, 20]; raw lines set [5, 20]; agree 64 of 64
  ECC_DONE pending, 0x844 bit 0        FAULT: wait2 passed [5, 43]; raw lines set [5, 43]; agree 64 of 64
  wait2 64..255, idle: passed []; VIC0 / VIC1 raw seen during the run 00000020 / 00000000
```

## 14.3 Reverse-engineering outcomes

Observations of the original firmware (`osos` = iPod OS image, `aupd` = restore OS image, `bootrom` = mask ROM). `osos` and `aupd` addresses: IMG1 body at file+0x800, `mem = 0x08000000 + file - 0x800` for `osos` and for the first `aupd` copy of the programs. Each entry states only what the bytes or the code at the cited addresses contain: the address read or written, the constants, masks, shifts and compares applied, the branch that follows a test, the instruction found at a program offset, and counts of bounded searches. Register and parameter names are labels for window offsets ([chapter 4](04-sequencer-memory-map.md), [chapter 9](09-nand-controller.md), [chapter 10](10-ecc-engine.md)); a label says nothing about use. Sequencer instructions are written with the mnemonics of [chapter 5, section 5.3](05-instruction-encoding.md#53-opcodes). `+0xNNN` is a byte offset from a program's first instruction; "the C4C offset" is the immediate of the program's opcode-01 write to `FMSS_C4C` (`0xc4c`). Program starts are listed in RE-3. What the observations are taken to show is in [chapter 15](15-evidence-interpretation.md).

Notation in the entries. A window offset is written `0xNNN`. "`0xNNN` = V" is an opcode-01 instruction (`wr`) that writes the immediate V to window offset `0xNNN`; "`FMC_CMD` = V" is the same for offset `0x008`. "`wait` event N" is an opcode-07 instruction whose `R` field is N. "`ld rR` from `0xNNN` mask M" is an opcode-04 instruction that loads window offset `0xNNN` ANDed with M into register rR; "`st` rR to `0xNNN`" is opcode 02, `ldi` is opcode 03, `sti` opcode 11, `bnz` opcode 0e and `bz` opcode 17. `movi`, `mov`, `and`, `or`, `add`, `sub`, `shl` and `shr` are opcodes 05, 06, 0a, 0b, 0c, 0d, 13 and 14. Registers are r0 to r7. Offsets of the form `+0xNNN` are byte offsets within the named program. Addresses written `[x + 0xNN]` are the word at address x plus 0xNN. ARM code is cited by address and written with ARM mnemonics.

### 14.3.1 RE-1: Nine sequencer programs in the iPod OS image

Citation: `osos file+0x9430,0x9830,0xa488,0xa698,0xaff0,0xb438,0xbdf8,0xc7b8,0xd138`; `aupd` 18 hits.

The 24 bytes `0c0c0001 ff000000 100c0001 01000000 580c0001 04000000` (three instructions: opcode 01 with arg 0x0c0c imm 0xff; opcode 01 with arg 0x0c10 imm 1; opcode 01 with arg 0x0c58 imm 4) occur 9 times in the `osos` body, at file+0x9430, 0x9830, 0xa488, 0xa698, 0xaff0, 0xb438, 0xbdf8, 0xc7b8 and 0xd138, and 18 times in the `aupd` body: at file+0x2ee5c, 0x2f25c, 0x2feb4, 0x300c4, 0x30a1c, 0x30e64, 0x31824, 0x321e4, 0x32b64 and at file+0xe76f0, 0xe7af0, 0xe8748, 0xe8958, 0xe92b0, 0xe96f8, 0xea0b8, 0xeaa78, 0xeb3f8. The byte ranges starting at the nine `osos` hits (lengths in RE-2) are equal to the ranges at the nine hits of each `aupd` set (equal SHA-1 values, position by position). In every one of the nine programs the fourth instruction is opcode 01 with arg 0x0c4c and an immediate that is a multiple of 8 below the program length; the instruction at that offset is opcode 04 with r=2, arg 0x0c3c and imm 0xffffffff. Each program contains two opcode-00 instructions: one at the C4C offset minus 8 and the last instruction of the program, which is also the first opcode 00 at or after the C4C offset. From the C4C offset to the last instruction there are 13 instructions in ERASE and in the five WRITE programs and 9 in READ, READID and RAWREAD. The start of each program is the byte after the last instruction of the previous program (no gap), in `osos` and in each `aupd` set.

### 14.3.2 RE-2: Program table

Citation: `osos file+0x9430..0xd138` (table of the nine programs).

| program | instructions | bytes | C4C offset | `osos` file | `aupd` file (first set / second set) | `FMC_CMD` values |
|---|---|---|---|---|---|---|
| ERASE | 128 | 0x400 | 0x398 | +0x9430 | +0x2ee5c / +0xe76f0 | 70 60 D0 70 |
| READ | 395 | 0xc58 | 0xc10 | +0x9830 | +0x2f25c / +0xe7af0 | 00 30 70 00 70 00 |
| READID | 66 | 0x210 | 0x1c8 | +0xa488 | +0x2feb4 / +0xe8748 | FF 90 |
| WRITE | 299 | 0x958 | 0x8f0 | +0xa698 | +0x300c4 / +0xe8958 | 80 10 70 80 10 70 |
| RAWREAD | 137 | 0x448 | 0x400 | +0xaff0 | +0x30a1c / +0xe92b0 | 00 30 70 00 |
| WRITE2P | 312 | 0x9c0 | 0x958 | +0xb438 | +0x30e64 / +0xe96f8 | 80 11 10 70 80 81 10 70 |
| WRITE2P_78 | 312 | 0x9c0 | 0x958 | +0xbdf8 | +0x31824 / +0xea0b8 | 80 11 10 78 80 10 70 |
| WRITE_78 | 304 | 0x980 | 0x918 | +0xc7b8 | +0x321e4 / +0xeaa78 | 80 10 78 80 10 70 |
| WRITECACHE | 319 | 0x9f8 | 0x990 | +0xd138 | +0x32b64 / +0xeb3f8 | 80 11 10 15 70 80 10 71 |

The last column lists the immediates of the opcode-01 writes to window offset `0x008` (`FMC_CMD`), in the order they appear in the code. The program names are working names.

### 14.3.3 RE-3: Program locations as memory addresses

Citation: `osos mem:0x8008c30` (ERASE), `mem:0x8009030` (READ), `mem:0x8009c88` (READID), `mem:0x8009e98` (WRITE), `mem:0x800a7f0` (RAWREAD), `mem:0x800ac38` (WRITE2P), `mem:0x800b5f8` (WRITE2P_78), `mem:0x800bfb8` (WRITE_78), `mem:0x800c938` (WRITECACHE); `aupd mem:0x0802e65c..0x08032d5c`.

The nine programs start at the `osos` addresses above (IMG1 body at file+0x800, mem = 0x08000000 + file - 0x800). The first `aupd` set starts at `aupd mem:` 0x0802e65c (ERASE), 0x0802ea5c (READ), 0x0802f6b4 (READID), 0x0802f8c4 (WRITE), 0x0803021c (RAWREAD), 0x08030664 (WRITE2P), 0x08031024 (WRITE2P_78), 0x080319e4 (WRITE_78), 0x08032364 (WRITECACHE) and ends at 0x08032d5c; the second set starts at 0x080e6ef0, 0x080e72f0, 0x080e7f48, 0x080e8158, 0x080e8ab0, 0x080e8ef8, 0x080e98b8, 0x080ea278, 0x080eabf8. The bytes of all nine programs in both `aupd` sets are equal to those of the `osos` programs. Bounded search of the 16 bytes `0c0c0001 ff000000 100c0001 01000000` (first two prologue instructions): 9 matches in the `osos` body, 18 in the `aupd` body, 0 in the `wtf` body, 0 in `bootrom`; a search for the 4 bytes `4c0c0001` (opcode 01, arg 0x0c4c) gives the same counts 9, 18, 0, 0.

### 14.3.4 RE-4: Instruction encoding and opcode set used by the programs

Citation: decoded streams of the nine programs.

Each instruction is 8 bytes: bytes 0..1 `arg`, byte 2 `r`, byte 3 `op`, bytes 4..7 `imm`, little endian (the first instruction of ERASE is `0c 0c 00 01 ff 00 00 00`). The op bytes that occur are `00 01 02 03 04 05 06 07 0a 0b 0c 0d 0e 11 13 14 17 18 19`. Op counts over all nine programs: 00:18 01:852 02:223 03:85 04:255 05:68 06:25 07:236 0a:35 0b:88 0c:97 0d:27 0e:92 11:9 13:74 14:34 17:51 18:2 19:1; total 2272 instructions. No program contains op `08`, `09`, `0f`, `10`, `12`, `15`, `16`, `1a..fe` or `ff`. The `imm` of every op `0e` and `17` instruction, and the `imm` written to `0xc4c`, is a multiple of 8 smaller than the length of its program.

### 14.3.5 RE-5: Prologue values

Citation: first 0x30 bytes of every program.

In ERASE, READ, WRITE, RAWREAD, WRITE2P, WRITE2P_78, WRITE_78 and WRITECACHE the first six instructions are opcode 01 writes: `0xc0c` = 0xff (+0x00), `0xc10` = 1 (+0x08), `0xc58` = 4 (+0x10), `0xc4c` = the C4C offset (+0x18), `0x004` = 0x0ff3f8e0 (+0x20), `0x048` = 0x0ff00ffe (+0x28). In READID the first four instructions are the same writes of `0xc0c`, `0xc10`, `0xc58`, `0xc4c`; instructions 5..7 are opcode 04 loads of `0xd08` (r7), `0xd0c` (r6), `0xd10` (r5) with mask 0xffffffff; `0x004` = 0x0ff3f8e0 is written at +0x38 and `0x048` = 0x0ff00ffe at +0x40.

### 14.3.6 RE-6: Opcodes 18 and 19

Citation: `osos mem:0x8009110` (READ+0xe0, op `19`), `mem:0x8009190` (READ+0x160, op `18`), `mem:0x8009710` (READ+0x6e0, op `18`); `osos file+0x9910` (op `19`, r=1, arg=3), `osos file+0x9990` (op `18`, r=7, arg=5), `osos file+0x9f10` (op `18`, r=2, arg=0); `movi r3,#0xd60` at `osos file+0x9890`, `movi r5,#0xd60` at `osos file+0x9988`, `0xd50` initialised to `0xd64` at `osos file+0x9978`.

Op `19` occurs once, at READ+0xe0 (r=1, arg=3, imm 0); op `18` occurs twice, at READ+0x160 (r=7, arg=5, imm 0) and READ+0x6e0 (r=2, arg=0, imm 0); no other program contains either. READ+0x60 is `movi r3,#0xd60`; READ+0xe8 is `add r3,r3,#4`; READ+0x158 is `movi r5,#0xd60`; READ+0x168 is `add r5,r5,#4`; READ+0x148 and READ+0x150 are `movi r0,#0xd64` and a `st` of r0 to `0xd50`; at READ+0x6d8 `ld r0` reads `0xd50`, READ+0x6e8 adds 4 to r0 and READ+0x6f0 stores r0 back to `0xd50`. READ+0x238 is `ld r5` from `0xd60`. The value in r1 at READ+0xe0 is the one-hot word built at READ+0xb0..0xc0 (`movi r0,#1; shl r0,r0,r7; mov r1,r0`).

### 14.3.7 RE-7: Wait event 5 shape

Citation: 19 uses, e.g. `osos file+0x9ac8`; `osos mem:0x80092c8` (READ+0x298), `mem:0x800a578` (WRITE+0x6e0).

`wait` with `R` = 5 occurs 19 times, at: ERASE +0x120, +0x148, +0x178, +0x2f0, +0x318, +0x348; READ +0x298, +0x750; WRITE +0x6e0, +0x8a8; RAWREAD +0x1a8; WRITE2P +0x720, +0x910; WRITE2P_78 +0x748, +0x910; WRITE_78 +0x708, +0x8d0; WRITECACHE +0x780, +0x948. The 13 outside ERASE have the same neighbours: three instructions before the wait are opcode 01 `0x004` = 0xca, opcode 04 `r0` from `0x04c` with mask 0x40, and opcode 0e `bnz r0` with a target equal to the address of the instruction after the wait; the instruction after the wait is opcode 01 `0x048` = 0x800000. The 6 in ERASE are described in RE-8. The `R` values of the `wait` instructions in the nine programs are 0, 1, 2, 3, 4 and 5 only (no `wait` with `R` = 12).

### 14.3.8 RE-8: ERASE ready waits

Citation: `osos mem:0x8008d30` (ERASE+0x100); `osos file+0x9548..0x95a8` and `0x9718..0x9778`.

First group, ERASE+0x100..+0x1a0: +0x100 `0x004` = 0xca; +0x108 `ld r0` from `0x04c` mask 0x40; +0x110 `bnz r0` to +0x180; +0x118 `0xc58` = 0; +0x120 `wait` event 5; +0x128 `0xc14` = 0xff; +0x130 `0xc0c` = 4; +0x138 `ld r0` from `0x04c` mask 0x40; +0x140 `bnz r0` to +0x180; +0x148 `wait` event 5; +0x150 `0xc14` = 0xff; +0x158 `0xc0c` = 4; +0x160 `ld r0` from `0x04c` mask 0x40; +0x168 `bnz r0` to +0x180; +0x170 `0xc58` = 4; +0x178 `wait` event 5; +0x180 `0xc58` = 4; +0x188 `0x048` = 0x800000; +0x190 `ld r2` from `0x04c` mask 1; +0x198 `0x004` = 0x20; +0x1a0 `bnz r2` to the C4C offset 0x398. Second group, ERASE+0x2e8..+0x370: the same instruction sequence at +0x2e8 (`0xc58` = 0) to +0x370, with `bnz` targets +0x350, the `ld` of `0x04c` mask 1 into r4 at +0x360 and `bnz r4` to 0x398 at +0x370.

### 14.3.9 RE-9: Control word and chip-enable changes

Citation: `osos mem:0x8009060` (READ+0x30), `mem:0x8009370` (READ+0x340), `mem:0x8009f28` (WRITE+0x90).

Offset `0x000` (`FMC_CTRL0`) is written at +0x30..+0x48 of ERASE, READ, WRITE, RAWREAD and the other four WRITE programs by `movi r0,#0x801`, `ld r1` from `0xd4c` (from `0xd48` in the five WRITE programs), `or r0,r0,r1`, `st` r0 to `0x000`. Chip changes (ERASE +0xb0, READ +0x178 and +0x248 and +0x700, WRITE +0xb0, RAWREAD +0xc0, and the corresponding positions in the other programs): `ld r1` from `0x000` with mask 0xfffffe01, `or r1,r1,r0`, `st` r1 to `0x000`, where r0 is a value shifted left by 1 just before (READ +0x170 `shl r0,r7,#1`; ERASE +0xa8 `shl r0,r3,#1`, with r3 = `1 << r2` built at +0x98..+0xa0). Mask 0xfffffe01 is also loaded and stored back without an `or` at the end of each program (for example READ +0xbe8). Bit 24: `ld r0` from `0x000`, `or r0,r0,#0x1000000`, `st`, at READ +0x340..+0x350, WRITE +0xf8..+0x108, RAWREAD +0x208..+0x218 (WRITE2P and WRITE2P_78 +0x100, WRITE_78 +0xf8, WRITECACHE +0x120); `ld` with mask 0xfeffffff and `st` back at READ +0xbd8, WRITE +0x780, RAWREAD +0x3d8, WRITE2P and WRITE2P_78 +0x7e8, WRITE_78 +0x7a8, WRITECACHE +0x820. ERASE and READID contain neither.

### 14.3.10 RE-10: READID control word

Citation: `osos mem:0x8009cd0` (READID+0x48).

READID+0x48 is opcode 01 `0x000` = 0x00033201 (binary bits set: 0, 9, 12, 13, 16, 17); bit 9 is set and bit 11 is clear in this value, whereas the other programs build `0x000` from 0x801 (bits 0 and 11) ORed with a parameter word. READID+0x50 writes `0x014` = 0; READID+0x58 is `movi r3,#0xff`, READID+0x60 `shl r0,r3,#1`, followed by the mask-0xfffffe01 read-modify-write of `0x000` (+0x68..+0x78).

### 14.3.11 RE-11: FMCTRL1 codes and FMC_DATA_SEL fields

Citation: counts over all nine programs.

Opcode-01 writes to `0x004` (`FMCTRL1`) over the nine programs, value:count: 0x0ff3f8e0:9, 0x1:16, 0xca:15, 0x20:15, 0x100e0:3, 0x1c2:1, 0x340:1, 0x32:6, 0xe2:11, 0x34:25, 0xe4:5, 0x2e0:5, 0x2e4:20, 0x100:9, 0x1e0:2. Opcode-01 writes to `0x014` (`FMC_DATA_SEL`): 0x0:2, 0x1:23, 0x10:6, 0x100:1, 0x102:16, 0x200:7, 0x201:13, 0x1000:25. Opcode-01 writes to `0x030` (`FMC_DNUM`): 0x7:1, 0xf:31, 0x1ff:43. No program writes `0x004`, `0x014` or `0x030` with opcode 02.

### 14.3.12 RE-12: ECC protocol in READ

Citation: `osos mem:0x80093b0` (READ+0x380..); `osos file+0x9bf0..0x9c30`; 36 writes of `0x80c` (6 with bit 0 set, 30 with bit 1 set).

READ writes `0x80c` six times, with values and offsets 0x1101 (+0x3d8), 0x11201 (+0x4c8), 0x21101 (+0x5b8), 0x31201 (+0x6d0), 0x31201 (+0x998), 0x31201 (+0xa98). Each of the six is preceded by `0x840` = 0x1ff and `0x814` = 0x180 (for the first: +0x3c8 and +0x3d0), followed by `wait` event 4 (at +0x3e0, +0x4d0, +0x5c0, +0x768, +0x9a0, +0xaa0), then `0x840` = 1, then `ld r1` from `0x810` with mask 0xffffffff (at +0x3f0, +0x4e0, +0x5d0, +0x778, +0x9b0, +0xab0). WRITE, WRITE2P, WRITE2P_78, WRITE_78 and WRITECACHE write `0x80c` six times each, 30 in total, with values 0x1102 (three times), 0x11202, 0x21102 and 0x31202 (WRITE: +0x1c0, +0x2d0, +0x360, +0x3f0, +0x500, +0x5c8), each preceded by `0x840` = 0x1ff and `0x814` = 0x180; each of these programs contains six `wait` event 4, each followed by `0x840` = 1. In total 36 writes of `0x80c`. The only window offsets of 0x800 and above that any program accesses are `0x80c`, `0x810` (READ only), `0x814` and `0x840`. ERASE, READID and RAWREAD contain no access to offsets of 0x800 and above.

### 14.3.13 RE-13: Page verdict in READ

Citation: `osos mem:0x8009408` (READ+0x3d8..0x440); `osos file+0x9c20..0x9c60`.

After the `ld r1` from `0x810` at +0x3f0: +0x3f8 `and r0,r1,#1`; +0x400 `bz r0` to +0x418; +0x408 `or r5,r5,#0x40000000`; +0x410 `bnz r0` to +0x448. At +0x418: `shr r1,r1,#0x10`; `and r1,r1,#0xf`; `movi r0,#1`; `shl r0,r0,r1`; `shr r1,r0,#1`; +0x440 `or r5,r5,r1`. The same instruction sequence follows each of the six `ld` of `0x810` (offsets in RE-12), with the branch targets shifted accordingly.

### 14.3.14 RE-14: Decode order

Citation: READ+0x3d8..+0x468; READ+0x4c8..+0xb30.

READ+0x3d8 writes `0x80c` = 0x1101; +0x3e0 `wait` event 4; +0x3e8 `0x840` = 1; +0x3f0 `ld r1` from `0x810`; the sequence of RE-13 follows to +0x440; +0x448 writes `0x004` = 0x100; +0x450 `wait` event 2; +0x458 `0x048` = 0x100000; +0x460 `wait` event 3; +0x468 `0x048` = 8. The order `0x80c` write, `wait` event 4, `0x004` = 0x100, `wait` event 2 also holds for the `0x80c` writes at +0x4c8 (`0x004` = 0x100 at +0x538, `wait` event 2 at +0x540), +0x5b8 (+0x628, +0x630) and +0x998 (+0xa08, +0xa10). For the `0x80c` write at +0x6d0, `wait` event 4 is at +0x768, the `0x07c` sequence of RE-16 follows, and `0x004` = 0x100 is at +0x8e0 with `wait` event 2 at +0x8e8. For the `0x80c` write at +0xa98, `wait` event 4 is at +0xaa0, followed by `0x038` = 7 (+0xb08), `0x014` = 0x200, `0x030` = 0x1ff, `0x004` = 0x1e0 (+0xb20) and `wait` event 2 (+0xb28).

### 14.3.15 RE-15: Page stream

Citation: `FMC_DNUM = 0xf` then `0x1ff` in READ+0x2e8..; `osos file+0x9b30..0x9c00`; `0x34` then `0x2e4` in WRITE+0x268; `osos file+0xa918`.

READ+0x2e8..+0x338: `0x030` = 0xf; `0x014` = 0x10; `0x024` = 0; `0x004` = 0x32; `wait` event 3; `0x048` = 8; `0x030` = 0x1ff; `0x014` = 1; `0x004` = 0xe2; `wait` event 3; `0x048` = 8. The `0x024` writes in READ are 0 (+0x2f8), 0x10 (+0x390), 0x20 (+0x480), 0x30 (+0x570), 0 (+0x878), 0 (+0x950), each followed by `0x004` = 0x32 after `0x030` = 0xf and `0x014` = 0x10, and then by `0x030` = 0x1ff, `0x014` = 0x102 or 0x201 (0x1 in the first block) and `0x004` = 0xe2. WRITE+0x268..: `0x030` = 0xf; `0x014` = 0x1000; `0x028` = 0; `0x004` = 0x34; `wait` event 3; `0x048` = 8; `0x014` = 0x102; `0x030` = 0x1ff; `0x004` = 0x2e4; `wait` event 2; `0x048` = 0x100000. The `0x028` writes in WRITE are 0 (+0x278), 0x10 (+0x308), 0x20 (+0x398), 0x30 (+0x478), 0x30 (+0x7a0).

### 14.3.16 RE-16: Meta engine use

Citation: `osos mem:0x8009ef0..0x800a018` (WRITE+0x58..0x180), `mem:0x8009800` (READ+0x7d0); `osos file+0xa000, 0xa818`.

WRITE+0x128..+0x178: `ld r1` from `0xd1c`; three times (`ldi r0,[r1]`, `add r1,r1,#4`, `st` r0 to `0x060`, `0x064`, `0x068` in turn); `st` r1 to `0xd1c`. +0x180 `0x078` = 0x3210; +0x188 `0x07c` = 1; +0x190 `ld r0` from `0x07c` mask 1; +0x198 `bnz r0` to +0x190. This block occurs again at +0x530..+0x5a0 (WRITE), after the `bnz r7` at +0x528. The `0x07c` = 1 writes are at WRITE +0x188 and +0x590, WRITE2P and WRITE2P_78 +0x190 and +0x598, WRITE_78 +0x188 and +0x590, WRITECACHE +0x1b0 and +0x5b8. In WRITE the first block is before the first `0x80c` write (+0x1c0). READ+0x7d0: `0x078` = 0x3210; +0x7d8 `0x07c` = 2; +0x7e0 `ld r0` from `0x07c` mask 2; +0x7e8 `bnz r0` to +0x7e0; then three times (`ld r0` from `0x060`, `0x064`, `0x068` in turn, `sti [r1],r0`, `add r1,r1,#4`) with r1 loaded from `0xd1c` at +0x7f0 and stored back at +0x840. The block occurs a second time at READ +0xb38..+0xba8. Every `0x078` write in the nine programs has the value 0x3210. ERASE, READID and RAWREAD contain no access to `0x060..0x07c`.

### 14.3.17 RE-17: READ batches

Citation: `osos mem:0x8009088` (READ+0x58..0x158), `mem:0x8009190` (READ+0x160); `osos file+0x9860..0x9b30`.

READ+0x58..+0x100 is a loop: `movi r2,#0` (+0x58) and `movi r3,#0xd60` (+0x60); +0x68 `ld r4` from `0xd18`; +0x70..+0x88 `ld r1` from `0xd10`, `ldi r7,[r1]`, `add r1,r1,#4`, `st` r1 to `0xd10`; +0x90 `shl r7,r7,#2`; +0x98..+0xa8 `ld r0` from `0xd14`, `add r0,r0,r7`, `ldi r7,[r0]`; +0xb0..+0xc0 `movi r0,#1`, `shl r0,r0,r7`, `mov r1,r0`; +0xc8 `and r0,r0,r2`; +0xd0 `bnz r0` to +0x108; +0xd8 `or r2,r2,r1`; +0xe0 op `19`; +0xe8 `add r3,r3,#4`; +0xf0 `sub r4,r4,#1`; +0xf8 `bz r4` to +0x108; +0x100 `bnz r3` to +0x70. +0x108..+0x150: `ld r0` from `0xd10`, `sub r0,r0,#4`, `st` to `0xd10`; `sub r3,r3,#0xd60`; `shr r3,r3,#2`; `st` r3 to `0xd5c`; `st` r2 to `0xd58`; `st` r4 to `0xd18`; `movi r0,#0xd64`, `st` to `0xd50`. +0x158..+0x230: `movi r5,#0xd60`, op `18` (+0x160), `add r5,r5,#4`, the command and address cycles with `FMC_CMD` = 0x00 (+0x190), `0x02c` = 4, `FMC_CMD` = 0x30 (+0x210), `sub r3,r3,#1` (+0x228), `bnz r3` to +0x160 (+0x230). Results: at +0x8b0..+0x8d0, `ld r0` from `0xd24`, `sti [r0],r5`, `add r0,r0,#4`, `st` to `0xd24`, `or r6,r6,r5`.

### 14.3.18 RE-18: Page word and FMC_STATUS b27

Citation: `osos mem:0x80096d0` (READ+0x6a0), `mem:0x8009a98` (READ+0xa68); `osos file+0x9ed0`, `0xa298`.

READ+0x6a0: `ld r0` from `0x048` with mask 0x08000000; +0x6a8 `0x048` = 0x08000000; +0x6b0 `bnz r0` to +0x6c0; +0x6b8 `movi r5,#0x20000000`. The same four instructions are at +0xa68..+0xa80 (targets +0xa88). At +0x8b0..+0x8d8 and +0xbb0..+0xbd0 the word in r5 is stored with `sti` to the address read from `0xd24` (which is then increased by 4 and stored back), r5 is ORed into r6, and (at +0x8d8) r5 is set to 0.

### 14.3.19 RE-19: DMA address and DMA control

Citation: `osos mem:0x80093b0` (READ+0x380), `mem:0x8009b38` (READ+0xb08), `mem:0x800a658` (WRITE+0x7c0).

`st` to `0x034` (`FMC_DMA_ADDR`) occurs at READ +0x378, +0x918, +0xa40; WRITE +0xf0, +0x4a8; RAWREAD +0x240, +0x3a0; WRITE2P and WRITE2P_78 +0xf8, +0x4b0; WRITE_78 +0xf0, +0x4a8; WRITECACHE +0x118, +0x4d0; in each case the stored register was loaded by `ldi` from the address in `0xd20` three instructions earlier (`ld r1` from `0xd20`, `ldi r2,[r1]`, `add r1,r1,#4`, `st` r1 to `0xd20`, `st` r2 to `0x034`). `0x038` (`FMC_DMA_CTRL`) = 7 is written once in each of READ (+0xb08), WRITE (+0x7c0), RAWREAD (+0x3a8), WRITE2P and WRITE2P_78 (+0x828), WRITE_78 (+0x7e8) and WRITECACHE (+0x860); ERASE and READID never write `0x034` or `0x038`.

### 14.3.20 RE-20: WRITE DMA list

Citation: WRITE+0x468...

In WRITE the five-instruction group `ld r1` from `0xd20`, `ldi r2,[r1]`, `add r1,r1,#4`, `st` r1 to `0xd20`, `st` r2 to `0x034` occurs at +0xd0..+0xf0 (before the first `0x80c` write) and at +0x488..+0x4a8 (after `0x028` = 0x30, `0x004` = 0x34 at +0x478..+0x480). The loop word `0xd54` is set from `0xd28` at +0x248..+0x250 and again at +0x438..+0x440, decremented at +0x418..+0x428 (`ld r7`, `sub r7,r7,#1`, `st`), with `bnz r7` to +0x468 at +0x430. `0xd18` is decremented at +0x448..+0x458 (`ld r2`, `sub r2,r2,#1`, `st`), with `bz r2` to +0x780 at +0x460. The same structure is present in the other four WRITE programs at offsets shifted by the differences in RE-2.

### 14.3.21 RE-21: Two-plane status polls

Citation: `osos mem:0x800b220` (WRITE2P+0x5e8), `mem:0x800b3a8` (WRITE2P+0x770).

WRITE2P+0x5e8: `bnz r5` to +0x618; +0x5f0 `FMC_CMD` = 0x11; +0x5f8 `wait` event 0; +0x608 `movi r5,#1`; +0x610 `bnz r5` to +0x638; +0x618 `FMC_CMD` = 0x10; +0x620 `wait` event 0; +0x630 `movi r5,#0`. +0x638..+0x6b8: the list and map reads of RE-29 for the next entry, the read-modify-write of `0x000` with mask 0xfffffe01, `and r3,r3,r6` (+0x6b0), `or r6,r6,r2` (+0x6b8); +0x6e0 `bz r3` to +0x748. When r3 is non-zero: +0x6e8 `0x014` = 1; +0x6f0 `FMC_CMD` = 0x70; +0x708 `0x004` = 0xca; +0x710 `ld r0` from `0x04c` mask 0x40; +0x718 `bnz r0` to +0x728; +0x720 `wait` event 5; +0x728 `0x048` = 0x800000; +0x730 `ld r4` from `0x04c` mask 1; +0x738 `0x004` = 0x20; +0x740 `bnz r4` to 0x958. +0x748 `bnz r5` to +0x770; +0x750 `FMC_CMD` = 0x80; +0x768 `bz r5` to +0x788; +0x770 `FMC_CMD` = 0x81. In WRITE2P_78 `FMC_CMD` = 0x11 is at +0x5f0 and 0x10 at +0x618; in WRITECACHE 0x11 is at +0x610, 0x10 at +0x658 and 0x15 at +0x678.

### 14.3.22 RE-22: Program FAIL path

Citation: `osos mem:0x800a788` (WRITE+0x8f0).

In WRITE, `ld r4` from `0x04c` with mask 1 at +0x6f0 is followed by `0x004` = 0x20 (+0x6f8) and `bnz r4` to +0x8f0 (+0x700); the same triple is at +0x8b8..+0x8c8 (`bnz` target +0x8f0). The instruction at +0x8f0 is the first of the C4C tail described in RE-25; in WRITE its `ld r0` from `0x000` mask 0x1fe is at +0x8f8, `shr r0,r0,#1` at +0x900, `and r6,r6,#0xffffff00` at +0x908, `or r6,r6,r0` at +0x910, and `or r6,r6,#0x10000000` at +0x948.

### 14.3.23 RE-23: ERASE sequence

Citation: `osos mem:0x8008c88`, `mem:0x8008e78` (ERASE+0x58, +0x248).

ERASE+0x50 `movi r6,#0`. +0x58..+0xc0: the list and map reads of RE-29 with the read-modify-write of `0x000` with mask 0xfffffe01. +0xc8 `mov r2,r3`; +0xd0 `and r2,r2,r6`; +0xd8 `bz r2` to +0x1a8. When r2 is non-zero: +0xe0 `0x014` = 1; +0xe8 `FMC_CMD` = 0x70; +0xf0 `wait` event 0; +0xf8 `0x048` = 2; then the first group of RE-8 (+0x100..+0x1a0). +0x1a8 `or r6,r6,r3`; +0x1b0 `FMC_CMD` = 0x60; +0x1b8 `wait` event 0; +0x1c8..+0x1e0 `ld r1` from `0xd0c`, `ldi r2,[r1]`, `add r1,r1,#4`, `st` r1 to `0xd0c`; +0x1e8 `0x02c` = 2; +0x1f0 `st` r2 to `0x00c`; +0x1f8 `0x004` = 1; +0x200 `wait` event 1; +0x208 `0x048` = 4; +0x210 `FMC_CMD` = 0xd0; +0x218 `wait` event 0; +0x228..+0x238 `ld r2` from `0xd18`, `sub r2,r2,#1`, `st` r2 to `0xd18`; +0x240 `bnz r2` to +0x58. +0x248 `movi r2,#1`; +0x250 `movi r3,#8`; +0x260 `sub r3,r3,#1` and +0x268 `bz r3` to +0x380; +0x270 `shl r2,r2,#1`; +0x278..+0x288 `mov r0,r2`, `and r0,r0,r6`, `bz r0` to +0x260; when non-zero +0x290..+0x2a8 `shl r0,r2,#1` and the mask-0xfffffe01 read-modify-write of `0x000`, +0x2b0 `0x014` = 1, +0x2b8 `FMC_CMD` = 0x70, +0x2c0 `wait` event 0, then the second group of RE-8 (+0x2d0..+0x370); +0x378 `bnz r3` to +0x260; +0x380..+0x388 the mask-0xfffffe01 load and store of `0x000`; +0x390 `end` (r6 is the value at that point).

### 14.3.24 RE-24: READID sequence

Citation: `osos mem:0x8009cd0`, `mem:0x8009d20`, `mem:0x8009d48`; `osos mem:0x8009e00` (READID+0x178); `osos file+0xa508, 0xa5d8, 0xa600`; delay loop `osos file+0xa520..0xa538`.

READID+0x58 `movi r3,#0xff`; +0x60 `shl r0,r3,#1`; +0x68..+0x78 the mask-0xfffffe01 read-modify-write of `0x000`; +0x80 `FMC_CMD` = 0xff; +0x88 `wait` event 0; +0x90 `0x048` = 2. +0x98 `ld r1` from `0xd14`; +0xa0 `sub r1,r1,#1`; +0xa8 `bnz r1` to +0xa0. +0xb0 `ld r1` from `0x000` mask 0xfffffe01 and +0xb8 `st` back. Loop head +0xc0 (`0x014` = 0): +0xc8 `ldi r2,[r5]`; +0xd0 `movi r3,#1`; +0xd8 `shl r3,r3,r2`; +0xe0 `shl r0,r3,#1`; +0xe8..+0xf8 the read-modify-write of `0x000`; +0x100 `FMC_CMD` = 0x90; +0x108 `wait` event 0; +0x118 `0x02c` = 0; +0x120 `0x00c` = 0; +0x128 `0x004` = 1; +0x130 `wait` event 1; +0x140 `0x030` = 7; +0x148 `0x014` = 1; +0x150 `0x004` = 0x1c2; +0x158 `wait` event 3; +0x168 `0x014` = 0x100; +0x170 `0x004` = 0x340; +0x178 `ld r0` from `0x080`; +0x180 `sti [r7],r0`; +0x188 `ld r0` from `0x080` (not stored); +0x190..+0x198 the read-modify-write of `0x000` with mask 0xfffffe01 and no `or`; +0x1a0 `add r5,r5,#4`; +0x1a8 `add r7,r7,#4`; +0x1b0 `sub r6,r6,#1`; +0x1b8 `bnz r6` to +0xc0; +0x1c0 `end`. r7, r6 and r5 hold the words loaded from `0xd08`, `0xd0c` and `0xd10` at +0x20..+0x30.

### 14.3.25 RE-25: Fault handlers

Citation: `osos mem:0x8009c40` (READ+0xc10), `mem:0x800a788` (WRITE+0x8f0), `mem:0x8008fc8` (ERASE+0x398).

Every program's C4C tail begins `ld r2` from `0xc3c`; `ld r0` from `0x000` with mask 0x1fe; `shr r0,r0,#1`; `and r6,r6,#0xffffff00`; `or r6,r6,r0`. After that: in READ (+0xc38..+0xc50), READID (+0x1f0..+0x208) and RAWREAD (+0x428..+0x440): `ld r0` from `0xc50` mask 1; `0xc50` = 1; `or r6,r6,#0x8000000`; `end`. In ERASE (+0x3c0..+0x3f8), WRITE (+0x918..+0x950), WRITE2P, WRITE2P_78, WRITE_78 and WRITECACHE: `ld r0` from `0xc50` mask 1; `bz r0` to the instruction after the `or`; `0xc50` = 1; `or r6,r6,#0x8000000`; then the mask-0xfffffe01 read-modify-write of `0x000` without `or`; `or r6,r6,#0x10000000`; `end`. The `bz` is absent in READ, READID and RAWREAD; `or r6,r6,#0x10000000` is absent in those three.

### 14.3.26 RE-26: RAWREAD

Citation: `osos mem:0x800aa50`; `osos file+0xaff0..0xb438`; `osos file+0xb250, 0xb290, 0xb2d0, 0xb310`.

RAWREAD +0x1e0..+0x1f8: `0x030` = 0x1ff, `0x014` = 1, `0x004` = 0xe2, `wait` event 3. +0x250..+0x258 stores `0xd28` into `0xd54`. Loop +0x260..+0x378: four groups of `0x014` = 0x102; `0x030` = 0x1ff; `0x004` = 0xe2; `0x004` = 0x100; `wait` event 3; `0x048` = 8; `wait` event 2; `0x048` = 0x100000 (at +0x260, +0x2a0, +0x2e0, +0x320); +0x360..+0x378 `ld r7` from `0xd54`, `sub r7,r7,#1`, `st`, `bnz r7` to +0x260. +0x3a0 `st` to `0x034`; +0x3a8 `0x038` = 7; +0x3b0 `0x014` = 0x200; +0x3b8 `0x030` = 0x1ff; +0x3c0 `0x004` = 0x1e0; +0x3c8 `wait` event 2. RAWREAD contains no write of `0x030` = 0xf or `0x004` = 0x32, and no access to offsets `0x800` and above. The `0x014` values written in RAWREAD are 0x102 (four times), 0x1 (twice) and 0x200; READ also writes 0x201 (three times) and 0x102 (twice), WRITE writes 0x201 (twice) and 0x102 (twice).

### 14.3.27 RE-27: Write variants

Citation: WRITE_78+0x6b0, WRITECACHE+0x638..+0x678; `osos file+0xba28, 0xbba8, 0xc4e8, 0xce68`; `osos file+0xd198, 0xd1b0, 0xd7b0, 0xda50`.

`FMC_CMD` writes with their offsets: WRITE2P 0x80 (+0x1d0), 0x11 (+0x5f0), 0x10 (+0x618), 0x70 (+0x6f0), 0x80 (+0x750), 0x81 (+0x770), 0x10 (+0x858), 0x70 (+0x8e0). WRITE2P_78: 0x80 (+0x1d0), 0x11 (+0x5f0), 0x10 (+0x618), 0x78 (+0x6f0), 0x80 (+0x770), 0x10 (+0x858), 0x70 (+0x8e0). WRITE_78: 0x80 (+0x1c8), 0x10 (+0x5e0), 0x78 (+0x6b0), 0x80 (+0x730), 0x10 (+0x818), 0x70 (+0x8a0). After each `0x78` write the instructions are `wait` event 0, `0x048` = 2, `0x02c` = 2, `st` r2 to `0x00c`, `0x004` = 1, `wait` event 1, `0x048` = 4, `0x004` = 0xca, `ld r0` from `0x04c` mask 0x40, `bnz`, `wait` event 5, `0x048` = 0x800000, `ld r4` from `0x04c` with mask 3 (WRITE_78 +0x718, WRITE2P_78 +0x758), `0x004` = 0x20, `bnz r4` to the C4C offset; the corresponding `ld r4` after the `0x70` writes (for example WRITE2P +0x730) has mask 1. WRITECACHE: `ld r1` from `0xd18` (+0x638); `ld r0` from `0xd44` (+0x640); `and r0,r0,r1` (+0x648); `bnz r0` to +0x678 (+0x650); `FMC_CMD` = 0x10 (+0x658); `bz r0` to +0x690 (+0x670); `FMC_CMD` = 0x15 (+0x678). `0xd44` is stored at +0x78 from r0, where r0 = 0 (+0x68) minus the word loaded from `0xd04` (+0x60, +0x70). WRITECACHE's last group writes `FMC_CMD` = 0x71 (+0x918) and has `ld r4` from `0x04c` with mask 1 (+0x958).

### 14.3.28 RE-28: Releases 1.0.1 to 1.1.3

Citation: every decrypted release searched; `osos 1.1.3 file+0xd1c8..0xd470` (WRITE_78, final group); WRITE_78 final group at `osos file+0xcfe8..0xd0b0`; WRITE2P_78 at `osos file+0xc668..0xc730` and `osos 1.1.3 file+0xc668..0xc910`; WRITE_78 at `osos 1.1.3 file+0xc998`.

The prologue pattern of RE-1 occurs 9 times in the `osos` image and 18 times in the `aupd` image of each of the releases 1.0.1, 1.0.2, 1.0.3, 1.1, 1.1.2 and 1.1.3; the image `FIRMWARE x1242` contains 18 matches. The nine programs of 1.0.1, 1.0.2, 1.0.3, 1.1 and 1.1.2 have the same lengths as in RE-2 and the same SHA-1 values (at different file offsets). In 1.1.3 (`osos` and both `aupd` sets) and in `FIRMWARE x1242` (both sets) seven programs are equal to those of 1.1.2; WRITE2P_78 has 372 instructions (C4C offset 0xb38) and WRITE_78 has 364 (C4C offset 0xaf8), against 312 (0x958) and 304 (0x918). The final group of instructions in the 1.1.3 WRITE_78 (+0x830..+0xad8; `osos` 1.1.3 file+0xd1c8..0xd470) per set bit of r6 does: `ld r1` from `0xd00`; `0x014` = 1; `FMC_CMD` = 0x78; `wait` event 0; `0x02c` = 2; `st` r1 to `0x00c`; `0x004` = 1; `wait` event 1; `0x048` = 4; `0x004` = 0xca; `ld r0` from `0x04c` mask 0x40; `bnz`; `wait` event 5; `0x048` = 0x800000; `ld r4` from `0x04c` mask 3; `0x004` = 0x20; `bnz r4` to the C4C offset. This is repeated with `add r1,r1,#0x80` (+0x928), with `movi r1,#0` (+0x9b8) and with `movi r1,#0x80` (+0xa48): four groups. In 1.1.2 the corresponding group (`osos` file+0xcfe8..0xd0b0, +0x830..+0x8f8) has one `FMC_CMD` = 0x70 write and one `ld r4` from `0x04c` with mask 1; the same difference exists between WRITE2P_78 of 1.1.2 (`osos` file+0xc668..0xc730) and of 1.1.3 (file+0xc668..0xc910).

### 14.3.29 RE-29: Chip addressing in the programs

Citation: `osos file+0x9488..0x94f0`.

The sequence `ld r1` from `0xd10`; `ldi r2,[r1]`; `add r1,r1,#4`; `st` r1 to `0xd10`; `shl r2,r2,#2`; `ld r0` from `0xd14`; `add r0,r0,r2`; `ldi r2,[r0]` starts at ERASE +0x58, READ +0x70 (the last `ldi` targets r7), WRITE +0x58, RAWREAD +0x68, WRITE2P and WRITE2P_78 +0x60, WRITE_78 +0x58, WRITECACHE +0x80; in each it is followed by `movi rX,#1`, `shl rX,rX,r2` (READ: `shl r0,r0,r7`) and a shift left by 1 into the mask-0xfffffe01 read-modify-write of `0x000`. The WRITE programs repeat it a second time (WRITE and WRITE_78 +0x5f8, WRITE2P and WRITE2P_78 +0x638, WRITECACHE +0x698). READID reads `0xd10` into r5 at +0x30 and executes `ldi r2,[r5]` at +0xc8; it has no `shl r2,r2,#2` / `add` / second `ldi`; its read of `0xd14` is at +0x98 (RE-24).

### 14.3.30 RE-30: Absent programs

Citation: bounded prologue scan of `osos` and `aupd`.

The scan is the search of RE-3 for the 16 bytes `0c0c0001 ff000000 100c0001 01000000` over the `osos`, `aupd`, `wtf` and `bootrom` bodies: 9, 18, 0 and 0 matches, all inside the programs of RE-2. Every program found has one of the nine command lists of RE-2; none has a command list other than those.

### 14.3.31 RE-31: CPU driver: start

Citation: `aupd mem:0x08010c40..0x08010c50`, literal `0x08010c5c`.

The function at `aupd mem:0x08010c40` loads the word at `0x08010c58` (value 0x38a00000) into r0; stores 0xff to `[r0 + 0xc0c]`; loads the word at `0x08010c5c` (value 0x0000fff5) and stores it to `[r0 + 0xc00]`; returns.

### 14.3.32 RE-32: CPU driver: wait and classify

Citation: `aupd mem:0x08010c60..0x08010d0c`.

The function at `aupd mem:0x08010c60` calls `0x08002cc0` (it returns the word at `0x3c7000b4`) and keeps the result as a start stamp; r5 = 0x38e01000 (word at `0x08010d14`); r6 = 0x38a00000 (word at `0x08010d18`). Loop: it reads `[r5 + 8]` and tests bit 22 (mask 0x400000). When the bit is clear it reads `[r5 + 8]` again and calls `0x08002ce8` with the stamp and 0x7d0; that function returns 1 when `now - stamp >= 0x7d0 * 1000` (`now` = the word at `0x3c7000b4`), 0 otherwise, and 0xffffffff when its second argument exceeds the word at `0x080031b8`. A zero result repeats the loop. A non-zero result prints the string at `0x0801bef0` with `[r6 + 0xc0c]` and the value 0x227, calls `0x08010c08` and returns 0. When bit 22 is set it reads `[r6 + 0xc0c]` and tests bit 2 (mask 4). If bit 2 is set: it prints the string at `0x08010d24` with the value read; prints the string at `0x0801bf50` with `[r6 + 0xc18]`, `[r6 + 0xc20]`, `[r6 + 0xc30]` and the value 0x237; calls `0x08010c08`; sets the result to 0; calls `0x080107f0`; returns 0. If bit 2 is clear and bit 0 (mask 1) is set: result 1, calls `0x080107f0`, returns 1. If bits 2 and 0 are both clear: it prints the string at `0x0801bfb8` with the value read, sets the result to 0, calls `0x080107f0`, returns 0. Apart from the register save at entry and the argument store to the stack at `0x08010cd0`, the function contains no store instruction.

### 14.3.33 RE-33: CPU driver: reset

Citation: `aupd mem:0x08010c08`; zero bytes at `aupd mem:0x0802e654`.

The function at `aupd mem:0x08010c08` loads r4 = 0x38a00000 (word at `0x08010c38`); stores 0x30000 to `[r4 + 0xc00]`; calls `0x08002d1c` with 1 (a loop that runs until `now - start >= 1 * 1000`, `now` being the word at `0x3c7000b4`); stores the word at `0x08010c3c` (value 0x0802e654) to `[r4 + 0xc04]`; calls the function at `0x08010c40` (RE-31); and returns through a jump to `0x08002d1c` with 1. The 16 bytes at `aupd mem:0x0802e64c..0x0802e65b` are zero; the first program starts at `0x0802e65c`.

### 14.3.34 RE-34: CPU driver: clear

Citation: `aupd mem:0x080107f0`.

The function at `aupd mem:0x080107f0` stores 5 to `[0x38a00000 + 0xc0c]` (base from the word at `0x0801084c`), calls `0x08002cc0` for a start stamp, and loops: reads `[base + 0xc0c]`, tests mask 5, returns when the result is 0; otherwise adds 1 to a counter and calls `0x08002ccc` with the stamp and 0x64 (returns 1 when `now - stamp >= 0x64`); a non-zero return prints the string at `0x0801be98` with the counter and the value read, then branches to itself without end.

### 14.3.35 RE-35: CPU driver: timing word

Citation: `aupd mem:0x08010734`; callers `aupd mem:0x08014b30`, `mem:0x08014b44`.

The function at `aupd mem:0x08010734` takes three arguments (r0, r1, r2) and does the following in this order:

1. calls `0x08016900` with 2
2. calls `0x08002370` (it returns r0 = quotient and r1 = remainder of r0 / r1) with r0 = 1000 and r1 = the result of `0x08016900`, keeping the quotient as T
3. computes for each of r0, r1 and r2 `q + (remainder != 0)` with q the quotient of the argument by T (c1, c2, c3)
4. when `c3 + c2 < c1` sets c3 = c1 - c2
5. computes hold = c3 - 1 (1 when that is 0), setup = c2 - 1 (1 when that is 0), edo = hold + 1 limited to 3
6. returns `(setup & 7) << 16 | (hold & 7) << 12 | (edo & 3) << 28`

The function at `0x08014978` calls it at `0x08014b30` with the halfwords at offsets 0x10, 0x12, 0x14 of the record at `r5` and stores the result to `[0x0802e624 + 0x1c]`, and at `0x08014b44` with the halfwords at 0x16, 0x18, 0x1a and stores the result to `[0x0802e624 + 0x20]` (`r5` = `0x0801bb70 + 44 * index`, a table of 18 records).

At `aupd mem:0x08014f00`, `0x08014ff4`, `0x080150ec`, `0x08015274`, `0x08015424`, `0x08015758` and `0x08015ad4` a word loaded from `[struct + 0x1c]` (struct = `0x0802e624`) is stored to window offset `0xd4c` and a word loaded from `[struct + 0x20]` is stored to `0xd48` (the store to `0xd48` is two instructions after the store to `0xd4c`); the instruction after each store to `0xd48` is a call to `0x08010c40`.

The programs load `0xd4c` in ERASE, READ and RAWREAD and `0xd48` in the five WRITE programs, each at +0x38, followed by `or` with 0x801 and a store to `0x000` (RE-9).

### 14.3.36 RE-36: CPU driver: chip reset

Citation: `aupd mem:0x0802f6b4`, `aupd mem:0x080155f4`.

The function at `aupd mem:0x080155f4`, in this order:

1. stores the word at `0x08015648` (value 0x0802f6b4) to `[0x38a00000 + 0xc04]`
2. stores `[0x0802e624 + 0x14]` to `0xd08`, the halfword at `[0x08036920 + 0x10]` to `0xd0c`, `[0x0802e624 + 0]` to `0xd10`, and 0x8000 to `0xd14`
3. calls `0x080107f0`, `0x08010c40` and `0x08010c60` in this order
4. returns the word at `0x08015658` (0x80010000) when the call to `0x08010c60` returned 0, and 0 otherwise.

`aupd mem:0x0802f6b4` is the start of the READID program (RE-3), whose first `FMC_CMD` write is 0xff (+0x80, RE-24). Stores with immediate offset 8 in `0x08014978..0x08014d4c` are at `0x08014c1c` (base r8 = `0x0802e624`) and `0x08014ce0` (base r1 = `0x08036b38`); stores with immediate offset 0x400 there are at `0x08014990` (base r0 = `0x38a00000`); `0x080155f4..0x08015648` contains no store with offset 8 or 0x400.

### 14.3.37 RE-37: CPU driver: initialisation

Citation: `aupd mem:0x08014978`.

The function at `aupd mem:0x08014978`, in this order:

1. calls `0x08010c08` (RE-33)
2. stores 0 to `[0x38a00000 + 0x000]` and to `[0x38a00000 + 0x400]` (base from the word at `0x08014d4c`)
3. stores the word at `0x08014d50` (0x0802f8c4) to `[0x0802e624 + 0x24]`
4. when `[0x0802e624 + 0x0]` is 0 it calls `0x080060c8` with 0x20 and stores the result there, and does the same with 0x20 for `[0x0802e624 + 0x14]`
5. stores to `[[0x0802e624 + 0x0] + 4 * i]` the value i, for i = 0 to 7
6. stores 8 as a halfword at `[0x08036920 + 0x10]`
7. calls `0x080155f4` (RE-36)
8. when its result is non-zero branches to `0x08014cc8`
9. stores 0 to the halfword `[0x08036920 + 0x10]`
10. then for r4 = 0..7 loads `[[0x0802e624 + 0x14] + 4 * r4]`, skips it when it is 0 or 0xffffffff, otherwise prints a string with r4, compares it with the first word of each of the 18 records at `0x0801bb70` (stride 44 bytes) and adds 1 to the halfword `[0x08036920 + 0x10]` when it matches.

### 14.3.38 RE-38: CPU driver: modes

Citation: `aupd mem:0x08010854`, jump table 0x08010890.

The function at `aupd mem:0x08010854` takes a value r4. If r4 > 13 it returns 0; otherwise `add pc,pc,r4,lsl #2` at `0x08010888` selects the branch at `0x08010890 + 4 * r4`. The branch targets are `0x08010988` (return 0) for r4 = 0, 5, 6, 7, 10, 11; and for the others the following code, each storing a halfword to `[0x08036920 + 0x3a]` and words to `[0x0802e624 + 0x24]`, `+0x28`, `+0x2c`, then calling `0x08003f40` and returning r4:

- r4 = 1 (`0x080108c8`): halfword 1; +0x24 = 0x0802f8c4; +0x28 = +0x2c = 0x08010bfc.
- r4 = 2 (`0x080108f4`): 2; +0x24 = 0x080319e4; +0x28 = 0x08010b70; +0x2c = 0x08010a70.
- r4 = 3 (`0x0801090c`): 2; +0x24 = 0x08031024; +0x28 = 0x08010ba0; +0x2c = 0x08010aa0.
- r4 = 4 (`0x0801091c`): 4; +0x24 = 0x08031024; +0x28 = 0x08010bc4; +0x2c = 0x08010ac4.
- r4 = 8 (`0x080108e0`): 2; +0x24 = 0x08030664; +0x28 = 0x08010ba0; +0x2c = 0x08010aa0.
- r4 = 9 (`0x0801093c`): 2; +0x24 = 0x0802f8c4; +0x28 = 0x08010b70; +0x2c = 0x08010a70.
- r4 = 12 (`0x08010950`): 2; +0x24 = 0x08032364; +0x28 = 0x08010afc; +0x2c = 0x080109fc.
- r4 = 13 (`0x08010964`): 2; +0x24 = 0x08032364; +0x28 = 0x08010b2c; +0x2c = 0x08010a2c.

The values 0x0802f8c4, 0x080319e4, 0x08031024, 0x08030664 and 0x08032364 are the start addresses of WRITE, WRITE_78, WRITE2P_78, WRITE2P and WRITECACHE in the first `aupd` set (RE-3).

### 14.3.39 RE-39: CPU driver: result classification

Citation: `aupd mem:0x0802ea5c` offsets 0x06a0 and 0x0a68; evaluator `aupd mem:0x0800ff00`.

Program side: READ+0x6a0 and +0xa68 are described in RE-18; the instruction sequences at the six `ld` of `0x810` in READ are described in RE-13.

Function side: the function at `aupd mem:0x0800ff00` takes r0 (a word), r1 (a pointer or 0) and r2. When r1 is non-zero it stores a zero byte to `[r1]`.

It then proceeds in this order:

1. if bit 30 (mask 0x40000000) of r0 is set it adds 1 to the 64-bit counter at `[0x08036a38 + 0x80]` and returns the word at `0x080100d0` (value 0x80020000)
2. if r2 equals 1 and bit 29 (mask 0x20000000) of r0 is set and the low byte `r0 & 0xff` is 0, it adds 1 to the 64-bit counter at `[0x08036a38 + 0x78]` and returns 1
3. otherwise, if the low byte is 0, it adds 1 to the counter at `[0x08036a38 + 0xb0]` and returns 0
4. otherwise it tests the bits of the low byte from bit 7 down to bit 0: for the highest set bit it adds 1 to the counter at `[0x08036a38 + 0xf0]` (bit 7), 0xe8 (bit 6), 0xe0 (bit 5), 0xd8 (bit 4), 0xd0 (bit 3), 0xc8 (bit 2), 0xc0 (bit 1), 0xb8 (bit 0), stores 8, 7, 6, 5, 4, 3, 2 or 1 respectively to `[r1]` when r1 is non-zero, and returns 0.

It is called from `0x080152bc` (r0 = `[r10 + 0xc30]`, r2 = 1 when r4 equals 1 and 0 otherwise, r1 = `[sp + 0x3c]`) and branched to from `0x08015450` when the result of `0x08010c60` is non-zero (r0 = `[r4 + 0xc30]`, r1 = `[sp + 0x38]`, r2 = 0).

Four places load `[rN + 0xc30]` and test bit 28 (mask 0x10000000): at `0x08014f30` (r9) and `0x08015024` (r4) the result is the word 0x80030000 (words at `0x08014f5c`, `0x08015050`) when the bit is set and 0 when it is clear; at `0x0801578c` (r10) and `0x08015b0c` (r4) the result is 0 when it is clear and, when it is set, a string is printed (with the values 0x526 and 0x4dd, words at `0x080157cc`, `0x08015b50`) and the result is 0x80030000 (words at `0x08015808`, `0x08015b8c`).

### 14.3.40 RE-40: CPU driver: other sequencer registers

Citation: functions of the original CPU driver.

A scan of `aupd mem:0x08000000..0x0802e653` and `0x08032d70..0x080e6edf` for ARM word load or store instructions (condition always, immediate offset added) with an immediate offset of 0xc08, 0xc10, 0xc14, 0xc1c, 0xc24, 0xc28, 0xc2c, 0xc34, 0xc38, 0xc3c, 0xc4c, 0xc50, 0xc5c, 0xc60, 0xc64, 0xc6c, 0x814 or 0x840 finds no match. The same scan finds, for offset 0xc00, stores at `0x08010c14` and `0x08010c50`; for 0xc04, stores at `0x08010c24`, `0x08014e7c`, `0x08014f98`, `0x08015094`, `0x080151bc`, `0x0801531c`, `0x08015600`, `0x080156a0` and `0x08015890`; for 0xc0c, stores at `0x08010800` and `0x08010c48` and loads at `0x08010828`, `0x0801083c`, `0x08010c94` and `0x08010cb8`; for 0xc18 a load at `0x08010cdc`; for 0xc20 a load at `0x08010cd8`; for 0xc30 loads at `0x08010cd4`, `0x08014f30`, `0x08015024`, `0x080152ac`, `0x0801578c` and `0x08015b0c` (and the conditional `ldrne` at `0x0801543c`). The scan does not cover accesses with a register offset or a computed address.

### 14.3.41 RE-41: Second microcode copy

Citation: `aupd mem:0x080e6ef0..0x080eb3f8+0x9f8`, tag at `aupd mem:0x08032d6c`; `aupd mem:0x08000020`.

The second set of nine programs starts at `aupd mem:0x080e6ef0` (RE-3) and its bytes are equal to those of the first set and of the `osos` programs. In the `aupd` body the 32-bit words equal to the start address of a program of the first set are at: ERASE `0x08014f48`, `0x08015038`; READ `0x080152c8`, `0x0801546c`; READID `0x08015648`; WRITE `0x080109a8`, `0x08014d50`, `0x080157b4`, `0x08015b40`; RAWREAD `0x08015124`; WRITE2P `0x080109b8`; WRITE2P_78 `0x08010994`; WRITE_78 `0x080109bc`; WRITECACHE `0x08010990`. No 32-bit word of the `aupd` body equals the start address of a program of the second set, or has a value in `0x080e6ee0..0x080ebfff`. The words at `aupd mem:0x08032d64` and `0x08032d68` are 0x000000d8 and 0x00000002; the 4 bytes at `0x08032d6c` are `64 6c 79 50` (ASCII `dlyP`), at `0x08032d7c` `70 55 77 46` (`pUwF`) and at `0x08032d84` `68 73 6c 66` (`hslf`); the 16 bytes before `0x080e6ef0` are zero. The word at `aupd mem:0x08000020` is 0xeb000d11 (an ARM `bl`); no program bytes are there.

### 14.3.42 RE-42: Driver generation

Citation: strings from `aupd mem:0x0801c000`.

A string at `aupd mem:0x0801c000` ends with `Whimory2_1\FIL\M1A\M1Aflash.c`. Strings that follow it begin `[FIL:ERR] m1fmssResetAndVerifyIds`, `[FIL:ERR] m1fmssWriteScatteredPages`, `[FIL:ERR] m1fmssReadScatteredPages`, `[FIL:ERR] m1fmssEraseSequentialBlocks`, `[FIL:ERR] m1fmssEraseSingleBlock`, `[FIL:ERR] m1fmssWriteSequentialPages` and `[FIL:ERR] m1fmssReadSequentialPages`. The string at `aupd mem:0x08010d24` reads `[FIL:ERR] FMSS sequence timed out (FC_CS_IRQ := 0x%x)!`; the string at `0x0801bef0` begins `[FIL:ERR] _waitForCSInterrupt() timed out after 2 seconds`.

### 14.3.43 RE-43: Boot ROM NAND access

Citation: `bootrom mem:0x20009910` (page reader), `bootrom mem:0x20002e0c` (`channel < 2`), `bootrom mem:0x20001830` (pin set-up).

Page reader, `bootrom mem:0x20009910` with arguments (c, ce, row, dest, out); b = 0x38a00000 + c * 0x400 (word at `0x20009cdc`); `wait(a, m)` below is a call of `0x20000e50(a, m, 0x186a0)`, a loop that returns when all bits of m are set in `[a]` or after 0x186a0 iterations.

Sequence, in order:

1. `[b + 4]` = 0x0001f0e0
2. `[b + 0]` = `0x43801 | 1 << (ce + 1)`
3. `[b + 8]` = 0
4. `wait(b + 0x48, 2)`
5. `[b + 0x48]` = 2
6. `[b + 0x2c]` = 4
7. `[b + 0xc]` = `row << 16`
8. `[b + 0x10]` = `row >> 16`
9. `[b + 4]` = 1
10. `wait(b + 0x48, 4)`
11. `[b + 0x48]` = 4
12. `[b + 8]` = 0x30
13. `wait(b + 0x48, 2)`
14. `[b + 0x48]` = 2
15. call `0x20009614(c, ce)`
16. `[b + 8]` = 0
17. `wait(b + 0x48, 2)`
18. `[b + 0x48]` = 2
19. `[0x38a00840]` = 0x7f

Then, for i = 0..3, in order:

1. `[b + 0x48]` = 0x8000000
2. `[b + 0x30]` = 0xf
3. `[b + 0x14]` = `1 << ((i & 1) + 4)`
4. `[b + 0x24]` = 0 (i < 2) or 0x10
5. `[b + 4]` = 0x32
6. `wait(b + 0x48, 8)`
7. `[b + 0x48]` = 8
8. `[b + 0x30]` = 0x1ff
9. `[b + 0x14]` = `1 << (i & 1)`
10. `[b + 0x24]` = 0
11. `[b + 4]` = 0x22
12. `wait(b + 0x48, 8)`
13. `[b + 0x48]` = 8
14. the word `[b + 0x48]` is read and, when its bit 27 (0x8000000) is set, `0x2000ac1c(i & 1, (i & 1) + 4, (i & 3) >> 1)` is called and `[0x38a00810]`, that bit and `[0x38a00840]` are ORed into three accumulators
15. `[b + 0x34]` = `dest + i * 0x200`
16. `[b + 0x38]` = 7
17. `[b + 0]` = `([b + 0] & 0xfffffbff) | 0x1000001`
18. `[b + 0x14]` = `1 << ((i & 1) + 8)`
19. `[b + 4]` = 0x1a0
20. `wait(b + 0x48, 0x100000)`
21. `[b + 0x48]` = 0x100000

After the loop, in order:

1. `[b + 0x78]` = 0x5140
2. `[b + 0x7c]` = 2
3. the loop runs while `[b + 0x7c] & 2`
4. `out[0..2]` = `[b + 0x60]`, `[b + 0x64]`, `[b + 0x68]`
5. `[b + 0]` = 0x43802
6. the result is 0 when no iteration saw bit 27, else 3 when bit 0 of the ORed `0x810` words is set, else 6 when bit 5 of the ORed `0x840` words is set, else 1
7. `[b + 0x48]` = 0xffffffff
8. `[0x38a00840]` = 0x7f

Channel routine, `bootrom mem:0x20002e0c(ch)`: does nothing when ch >= 2; otherwise calls `0x2000147c(8)`, `0x2000147c(0xc)`, `0x20001830(ch)`, `0x200096e4(ch, 0)`, `0x20009910(ch, 0, 0, 0x22020000, buf)`, and `0x20009910(ch, 0, k + 1, k * 0x800 + 0x22000000, buf)` for k from 0 below min(ceil(w / 0x800), 0x3f), where w is the word at offset 0xc (or 0x10) of the buffer at `0x22020000`.

Pin routine, `bootrom mem:0x20001830(ch)`, base 0x3cf00000 (word at `0x200019ec`): ch = 0 stores `([base + 0x140] & 0xffff0000) | 0x2222` to `[base + 0x140]`, `([base + 0x120] & 0xfff0fff0) | 0x20002` to `[base + 0x120]` and 0x22222222 to `[base + 0x100]`; ch = 1 stores `([base + 0x180] & 0xffff0000) | 0x2222` to `[base + 0x180]`, `([base + 0xe0] & 0xfff0ffff) | 0x20000` to `[base + 0xe0]`, `([base + 0x140] & 0xffff) | 0x22220000` to `[base + 0x140]` and `([base + 0x160] & 0xfff00000) | 0x22222` to `[base + 0x160]`.

### 14.3.44 RE-44: No cache maintenance in the CPU-side driver

Citation: `aupd`, the CPU-side NAND driver functions (function table at `aupd mem:0x08036b38`).

The function at `aupd mem:0x08014978` stores these twelve function addresses to the table at `0x08036b38` (base from the word at `0x08014de8`), at the offsets given, with stores between `0x08014cd0` and `0x08014d30`: 0x00 `0x08015560`, 0x04 `0x08015484`, 0x08 `0x08015054`, 0x0c `0x080152e0`, 0x10 `0x08015178`, 0x14 `0x080154f0`, 0x18 `0x0801565c`, 0x1c `0x0801580c`, 0x20 `0x08015b90`, 0x24 `0x08014f60`, 0x28 `0x08014e40`, 0x2c `0x08015bf0`. The ARM words in `aupd mem:0x08000000..0x0802e653` that are coprocessor-15 register transfers (MCR or MRC, coprocessor number 15, condition always) number 176; they lie in `0x080033ec..0x080036f0` and at `0x080213a8` and above; none lies in `0x08004000..0x080213a7`, which contains the twelve functions above, `0x08010734`, `0x08010854`, `0x08010c08`, `0x08010c40`, `0x08010c60` and `0x080107f0`. The transfers with CRn = 7 are at `0x080034b0`, `0x080034b8`, `0x080034c4`, `0x080034cc`, `0x08021f3c` and `0x080228fc`. No branch word (B or BL, any condition) in `0x08010000..0x08015fff` has a target in `0x080033ec..0x080036f0` or `0x080213a8..0x080229ff`.

### 14.3.45 RE-45: Chip table row of this unit's chip

Citation: `aupd mem:0x0801bb70` (`file+0x1c370`), `aupd mem:0x0801bb9c` (`file+0x1c39c`).

Two consecutive 0x2c-byte rows begin with the bytes `ec d5 14 b6`. The halfwords at row offsets +4, +6, +8, +0xa, +0xc and +0xe are 2, 4096, 128, 8, 128, 1 in the first row and 4, 4096, 128, 8, 128, 1 in the second; the halfword at +0x20 is 3872 in both; the word at +0x24 is 8 in the first row and 1 in the second.

### 14.3.46 RE-46: Bank count in WRITECACHE

Citation: `osos file+0xd138` (WRITECACHE) +0x58..+0x78 and +0x630..+0x690.

- +0x58 `movi r5,#0`; +0x60 `ld r1` from `0xd04` mask `0xffffffff`; +0x68 `movi r0,#0`; +0x70 `sub` with `R` = 0, `arg` = 1, `imm` = 0 (register form: r0 = r0 - r1); +0x78 `st` r0 to `0xd44`.
- +0x630 `bnz r5` to +0x698; +0x638 `ld r1` from `0xd18`; +0x640 `ld r0` from `0xd44`; +0x648 `and` with `R` = 0, `arg` = 1, `imm` = 0 (register form: r0 = r0 AND r1); +0x650 `bnz r0` to +0x678.
- +0x658 `FMC_CMD` (`0x008`) = `0x10`; +0x660 `wait` event 0; +0x668 `0x048` = 2; +0x670 `bz r0` to +0x690.
- +0x678 `FMC_CMD` = `0x15`; +0x680 `wait` event 0; +0x688 `0x048` = 2; +0x690 `movi r5,#0`.
- No other instruction of the nine programs reads or writes `0xd04` or `0xd44` (all opcode `01`, `02` and `04` instructions with these offsets searched).

### 14.3.47 RE-47: Boot ROM boot-device selection

Citation: `bootrom mem:0x20003230`, `mem:0x200019d4`, `mem:0x200018fc`.

- `0x200019d4(p, n)` returns bit n of the word at `0x3cf00004 + 0x20 * p` (the word address is the constant at `bootrom mem:0x200019f8`, `0x3cf00004`).
- The function at `0x20003230` calls `0x200018fc(1, 0, 0)` and `0x200018fc(1, 1, 0)`, then a = `0x200019d4(1, 0)` and b = `0x200019d4(1, 1)`.
- If bit 0 of the word at `0x3c400028` is 0, or a != 1 and b != 0, it calls `0x200018fc(3, 6, 0)` and `0x200018fc(3, 7, 0)`, then c = `0x200019d4(3, 6)` and d = `0x200019d4(3, 7)`, and calls: `0x20002fa8(0)` for c = 0, d = 0; `0x20002fa8(2)` for c = 0, d = 1; `0x20002e0c(0)` for c = 1, d = 0; `0x20002e0c(1)` for c = 1, d = 1.
- Otherwise, and after any of these calls returns, execution continues at `0x200033b8`.
- `0x20002e0c` is the NAND boot routine of RE-43 (channel 0 or 1); `0x20002fa8(x)` reads from an SPI controller x (calls `0x20008f90(x, ...)` and `0x2000906c(x, 0x800, ...)`).

References: [chapter 14](14-evidence.md) (outcomes), [chapter 13](13-experiments.md) (what was done and why), [chapter 15](15-evidence-interpretation.md) (what the outcomes mean).

Related chapters: [chapter 13](13-experiments.md), [chapter 15](15-evidence-interpretation.md), [chapter 18](18-ledger.md).
