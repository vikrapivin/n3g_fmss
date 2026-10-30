# nano 3G FMSS ("M1"): sequencer, NAND controller and ECC engine

These documents describe how the FMSS engine of the iPod nano 3G (S5L8702) works: the sequencer coprocessor inside the NAND controller, the NAND controller (FMC) it drives, and the ECC engine. They cover the engine's behaviour, registers, instruction set, memory maps, ECC, events, timing and interfaces, and how the original firmware uses it. Each statement about the engine's behaviour comes with its evidence.

The documents form a book of numbered chapters. Chapter 1 defines the evidence classes, the numbering and the terms. Chapters 2 to 12 describe the engine. Chapters 13 to 15 hold the experiments, their outcomes and the interpretation of the outcomes. Chapters 16 to 18 collect the device findings, the open questions and the finding list.

## Chapters

| chapter | title | content |
|---|---|---|
| 1 | [Overview and terms](01-overview.md) | evidence classes, numbering, terms, the three blocks |
| 2 | [Architecture: execution model, start, stop and fault handling](02-architecture.md) | execution model, start, pause, hold, stop, END and fault handling, diagrams of the CPU and sequencer interface |
| 3 | [The register window as the ARM CPU sees it](03-cpu-memory-map.md) | segments, mirrors, access matrix, CPU-only rules, interrupt lines, how the original CPU driver runs a program |
| 4 | [The register window as the sequencer sees it](04-sequencer-memory-map.md) | 12-bit offsets, register file, parameter RAM, system memory, program fetch |
| 5 | [Instruction encoding](05-instruction-encoding.md) | instruction word layout, operand map, opcode summary table, ALU operand rule, byte order |
| 6 | [Opcode reference](06-opcode-reference.md) | one entry per valid opcode, and one for the opcode values without a meaning |
| 7 | [Events, waits and the timeout](07-events-and-waits.md) | `wait` events, the rules for `wait` event 5, `wait2` conditions, `tmwait` and `evjmp`, the timer `C14`, the timeout length table |
| 8 | [Sequencer control registers](08-sequencer-control-registers.md) | the register table of the control block `0xc00..0xc5c` and the start-word bits |
| 9 | [NAND controller (FMC)](09-nand-controller.md) | `FMC_CTRL0`, the `FMCTRL1` codes, the other registers, `FMC_STATUS`, `FMC_DATA_SEL`, the second register set |
| 10 | [ECC engine](10-ecc-engine.md) | on-flash layout, BCH and Reed-Solomon codes, the `ECC_MODE` map, modes without bit 8, strength, registers, byte order |
| 11 | [NAND operations](11-nand-operations.md) | how the original firmware performs reset, ID read, page read, page program, block erase and fault handling, and the device checks |
| 12 | [CPU interface: parameter words, result words and timing word](12-cpu-interface.md) | parameter words, result words and the timing word |
| 13 | [Experiments: what was done and why](13-experiments.md) | every experiment run on the device |
| 14 | [Evidence: outcomes](14-evidence.md) | the outcomes of the device experiments and of the reverse engineering of the original firmware |
| 15 | [Evidence interpretation](15-evidence-interpretation.md) | what the outcomes are taken to show, by topic |
| 16 | [Device findings: risks and what only a device can settle](16-device-findings.md) | the risks and their status, and the results settled on a device |
| 17 | [Open questions](17-open-questions.md) | every point marked open, not measured or unexplained |
| 18 | [Finding list](18-ledger.md) | findings 1 to 175 with evidence and class |

## Sources

- lemonjesus, "S5L8702-FMISS-Tools", https://github.com/lemonjesus/S5L8702-FMISS-Tools/tree/main
