---
name: sail-readme
description: >-
  The house structure for operational README and runbook documentation on
  Sail/RISC-V test-generation work — the ten-section flow from prerequisites
  through cloning, submodules, settings, generation, simulation, coverage,
  documentation, riscv-arch-test packaging and QEMU. Use when asked to write,
  restructure or review a README, runbook, onboarding guide or "how do I run
  this" document for riscv-test-generation, the Sail Golden Model, isla-testgen,
  or any repository in that family; and whenever a doc needs to tell someone how
  to go from a clean machine to generated tests with measured coverage.
---

# Writing an operational README for Sail test-generation work

The reference implementation is `~/Documents/riscv-test-generation/README.md`.
Read it before writing a new one — it is the worked example of everything below.

## What this structure is for

A reader arriving cold has to get from **nothing installed** to **generated
tests with a coverage number they can defend**. The ten sections are that
journey in order, and the order matters: each section assumes only what the
previous ones established. Do not reorder them to suit a narrative.

## The ten sections

1. **Prerequisites** — a table of tool, what needs it, and notes. Call out the
   one step people miss (for Sail: `eval $(opam env)` in every new shell).
2. **Cloning** — the exact command, including `--recurse-submodules`, and what
   breaks without it.
3. **Submodules** — a table of folder, repository, branch, purpose. Note where
   folder names differ from repository names, and give the recovery command for
   a clone that already went wrong.
4. **Required settings before any command** — environment, builds, path
   resolution, and a verification command that proves the setup works.
5. **Test generation** — the largest section. Preamble settings (PMP, PMA, trap
   handler, PTE), where configuration comes from, which script does what, and
   operand solving.
6. **Simulation** — running simulators together, the comparison methodology, and
   where outputs land.
7. **Coverage** — producing the trace, turning it into a number, the
   denominator, and the feedback loop into generation.
8. **Documentation and packaging** — what to record once numbers settle.
9. **Packaging into riscv-arch-test** — per-extension emission and running under
   ACT4's own pipeline.
10. **Running on QEMU** — and how its requirements differ from the default mode.

Sections 5–7 each end with "hows, whats, and a diagram if it earns its place".

## Rules that make these documents correct

**Every command must have been run.** Do not write a flag you have not confirmed
in the source. Check with `grep -o '"--[a-z-]*"' <script>.py` before documenting
a CLI. This has caught real errors — `--per-elf` existed while the status notes
said per-ELF coverage did not.

**Verify every path you link.** A README full of dead links teaches readers to
stop clicking. Loop over the referenced paths and test each one before
publishing.

**Separate the two model-derived inputs; they are constantly conflated.**

| Input | What it is | What it drives |
|---|---|---|
| Span manifest (`sail_riscv_model.branch_info`) | every instrumented span, emitted once at build time | the testplan and the coverage denominator |
| Sail source declarations (`.sail`) + config JSON | enums, bitfields, match arms | the concrete cases — actual PMP/PTE/trap values |

The branch *trace* (`sail_coverage`, appended at run time) is a third thing
again: it is what the corpus reached, the numerator. Readers — and requests —
routinely say "generate the preamble from the branch trace". Correct it plainly
rather than writing to the misconception.

**State the preamble-not-solver principle wherever generation is described.**
Architectural state is written directly into the preamble; the solver is asked
only for operands. This is the reason the approach works, so it belongs in the
document, not just in the code.

**Never let a coverage number appear without its scope and span kind.** Span
kind moves the figure ~27 points, scope moves it ~35. One corpus supports four
defensible percentages. Put both in the same sentence as the number.

**Say which results are circular.** Expected values come from the Sail model, so
re-running on Sail proves self-consistency only. Only an independent simulator —
Spike — is evidence about the model. Every operational doc must say this.

**Warn about the failure modes that look like something else**, because they
cost the most time:
- Spike's ISA string missing an extension reads exactly like a model divergence.
- `isla-testgen` exits 0 on generation failure, so a runner logs `GEN ok` and
  then runs a file that does not exist.
- Configuration differences (VLEN, ELEN, PMP entry count) look like divergences.
- A passing suite can be verifying nothing — 47.7% of the corpus once did.

**Warn about resource limits where the command appears, not in a footnote.**
Generators must run one at a time; running them in parallel has taken a 15 GB
machine down.

## Diagrams

Use ```mermaid fences — GitHub renders them, and the repository already uses
them in `documentation/`. Draw the mechanism, not its name: a box labelled
"coverage" says less than the prose, while the path from build to manifest to
denominator says what words cannot. Label every arrow.

The diagrams that consistently earn their place:
- the four parts of a generated test (preamble / body / self-check / verdict)
- the two model-derived inputs and where they diverge
- the solver path, including UNSAT becoming a coverage exclusion
- the simulator-agreement verdict
- the coverage feedback loop
- the three mutually-exclusive generation modes (default / `--boot-fixed-entry`
  / `--signature`)

Do not draw a diagram for a linear list of commands.

## Tone

Plain English, short sentences, plain words where one exists ("the solver ran
out of memory", not "resource exhaustion during constraint solving"). Lead with
the number or the answer, then the reason. Tables for counts. Simple wording,
unchanged rigour — keep exact figures, `file:line` references and caveats.

Use a blockquote for anything that will cost the reader an hour if they miss it,
and put it next to the command it concerns.
