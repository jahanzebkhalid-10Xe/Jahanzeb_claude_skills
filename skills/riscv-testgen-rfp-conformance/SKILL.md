---
name: riscv-testgen-rfp-conformance
description: >-
  The target the riscv-test-generation project is converging on: RISC-V
  International's "Automatic Test Generation using the Sail RISC-V Golden
  Model" RFP v1.1, mapped requirement by requirement to what is built, what
  is partial, and what is missing. Use whenever working in
  ~/Documents/riscv-test-generation or on the Sail test-generation framework,
  and specifically when asked what is left to do, whether something is in
  scope, how the work maps to the RFP, what to prioritise, what to put in the
  proposal, or whether a claim about coverage/completeness is defensible.
---

# What riscv-test-generation is converging towards

The single source of truth for scope on this project is **RFP v1.1**,
*"Automatic Test Generation using the Sail RISC-V Golden Model"*, from RISC-V
International. Everything below is drawn from that document.

**Read this before deciding what to work on, before saying something is
"done", and before quoting a number in anything that leaves the repo.**

## Where this sits now: the proposal window has closed

**The proposal deadline was 31 August 2026, and it has passed.** The proposal
itself is finished and built:
`PROPOSAL/Proposal_automatic_test_generation_using_sail.pdf` (20 pages — cover,
contents, 11 numbered sections) plus a cost-free variant,
`Proposal_no_cost_no_titlepage.pdf` (19 pages). Both render from
`PROPOSAL/proposal-2026.html`; see [[riscv-testgen-proposal-deliverable]] for how
to rebuild them. Whether it was actually emailed to `tech-proposals@riscv.org` is
the user's action and nothing in this repo records it — ask, do not assume.

**That date was never the deadline for the framework.** Deliverables 1–5 are
post-award and run over months. A missing deliverable is not a crisis; it is
something the proposal has now committed to in writing. The questions that matter
from here are whether the plan survives review and whether the numbers in it hold
when someone else re-runs them.

Do not re-derive urgency from any date on this page. Do not treat "deliverable
not built yet" as "behind schedule" without checking which side of the award it
falls on.

## Requirements, and where each one stands

Two columns of status, because they age differently. **Requirement** text is
stable — it changes only if a new RFP version appears. **Status** drifts, and
must be re-derived rather than trusted (see "Keeping this honest" below).

### Goals

| # | Requirement | Status |
|---|---|---|
| G1 | Use the current sail-riscv model | **Met** — tests are generated from the model's own instruction definitions, so they track it |
| G2 | Generate valid RISC-V ELFs | **Met** — real toolchain, the Golden Model's own `crt0.S`/`link.ld`/runtime, HTIF termination |
| G3 | Organise tests by ISA extension | **Met** — `out/<config>/<extension>/` plus a manifest; the sweep is driven per-extension |
| G4 | Support the model's configurations, **including the ones in its CI: RV32+RV64, F/D, various VLEN/ELEN** | **Partial — the largest gap.** RV32 and RV64 both work (verified end-to-end, oracle and isla). But the model's CI builds **16** configs (`rv{32,64}d` × VLEN {64,128,256,512} × ELEN {32,64}) and we run **2** — `v128_e64` at both XLENs |
| G5 | Focus on privileged; **M-mode PMP/PMA, trap and interrupt handling required**; S-mode VM/translation highly desirable | **Measured on one clean run, 2026-08-31.** 1,144 privileged ELFs from six generators reach **77.7% of reachable privileged branch spans (539/694)** and **61.1% of all privileged spans (1,522/2,492)**. PMP, traps, interrupts and Sv translation all have derived cases passing on Sail and Spike. PMA is no longer zero, but at **18.8%** (`sys/pma`, 16/85) it is the weakest *required* area; the page-table walker is weaker still at **15.0%** (`sys/vmem_ptw`, 9/60). See [[riscv-testgen-clean-run-baseline]] and [[riscv-testgen-derived-cases]] |
| G6 | Coverage measure for an **individual ELF** and for **all ELFs** in a suite | **Met** — suite-level is the default; per-ELF is `coverage_report.py --per-elf`, which writes `spans<TAB>path` sorted by contribution. Its own help text names it "the other half of RFP Goal 6". *(Corrected 2026-09-01 — this row read "per-ELF does not exist" until the flag was found in the source.)* |
| G7 | Open-source licence compatible with the Golden Model | **Met** — Apache-2.0, which the RFP names as preferred |

### Deliverables

| # | Requirement | Status |
|---|---|---|
| D1 | Repository + user-facing docs + **developer-facing docs for long-term community maintenance** | **Strong.** The submitted proposal (PDF, 20 pages), a 20-slide deck, feature walkthroughs (PMP, virtual memory), the pipeline/provenance map, a worked end-to-end demonstration and the reproducible privileged run under `demo/privileged/` all live in the repo; see [[riscv-testgen-doc-pages]] and [[riscv-testgen-proposal-deliverable]]. Traps, interrupts, PMA and CSRs still have no feature page, and **none of it has been read by anyone outside the project** — see [[riscv-testgen-review-gap]] |
| D2 | Example scripts to generate a suite from a config | **Met** |
| D3 | Example scripts to run a simulator, collect results, show a summary | **Met, and beyond it** — Sail, Spike, QEMU and CVA6 RTL, with a summary report |
| D4 | **Approved and merged PRs integrating the framework into the Golden Model's CI** | **Not started.** The reporting machinery exists (`regression.py` exits 1 on regression); the wiring and the PR do not. Gated on maintainer cadence — start the conversation early |
| D5 | **PRs filed for any needed Golden Model fixes** | **Started.** A4 (shadow-stack PTE) is fixed and open as PR #2 **on the fork**, verified with `ctest` 664/664. Note the RFP wants these *approved and merged upstream*, so the fork PR is evidence, not completion — upstreaming is a separate decision the user controls |

### Considerations

| # | Requirement | Status |
|---|---|---|
| A | ACT4-compatible ELFs (macros, signature format) | **Met** — steps 1–6 implemented and proven through ACT4's own pipeline; only coverpoint tagging deferred |
| B | Consistent formatting; pointers to originating Sail code *encouraged, not required* | **Partial** — formatting yes, Sail pointers no. Lowest priority on this page |
| C | Unprivileged ISA (no FP/vector) should be straightforward | **Exceeded** — 1280 instructions across 23 extensions |
| D | Framework must be **extendable** to hypervisor, FP, vector — those are future iterations | **Met with evidence, not assertion** — FP and vector generators exist and pass today |
| E | Tests and coverage may change as the model evolves | **Met by design** — the instruction set is parsed from the model |
| F | Coverage expected to be **in terms of the Sail code** | **Met** — branch coverage against the real `.sail` sources |
| G | Self-checking or trace-based pass/fail | **Met for the mechanism, unfinished for the corpus.** Tests are self-checking and have been audited for false passes. But on the 2026-08-31 clean run only **665 of 1,144** ELFs reached SUCCESS; the other 479 have not been split into model faults and framework faults. Do not describe the corpus as passing |

## Scope: what is deliberately excluded, and why that is legitimate

- **Hypervisor** — excluded twice over: the Goals say "can be considered
  out-of-scope for this RFQ", and Consideration D lists it as a future
  iteration. Note there is an orphaned upstream draft (sail-riscv PR #612)
  whose maintainer invited pickup; check it before proposing anything
  from scratch.
- **Floating point and vector** — Consideration D. They are still generated and
  measured, because *extendability to them is itself a requirement* and a
  working generator is the only convincing evidence of it. They just do not
  belong in the current deliverable's denominator.

**Always report both coverage numbers, and check which way the exclusion cuts.**
It is not a fixed direction — it depends on what the corpus actually generated.
On a full sweep, FP and V are the *better*-covered half, so excluding them raises
the figure and flatters the work. On the privileged-only run of 2026-08-31 they
were not generated at all, so they sit at 0% and *depress* the full-scope figure
by construction (24.0% full RFP scope against 61.1% privileged). Quoting either
number alone misleads, in opposite directions, on the same repository. The scope
files (`python-isla/rfp-scope.txt`, `rfp-scope-current.txt`,
`rfp-scope-privileged.txt`) encode the splits with written justifications; keep
exclusions justified there rather than implied in code.

## Priority order

Ranked by what moves the proposal, not by effort. Re-rank when facts change,
but state why.

1. ~~File the A4 PR.~~ **Done 2026-08-10** — on the **fork only**
   (`10x-Engineers/sail-riscv-testgen` PR #2). Two hunks in `vmem_pte.sail`:
   the assertion, and step 8's `Atomic(_) => pte_W & pte_R` re-denying what the
   shadow-stack branch just permitted. Verified abort -> page fault -> SUCCESS,
   `ctest` 664/664. **Standing constraint: nothing goes on official repos** —
   in `~/Documents/sail-riscv` the remote `origin` *is* `riscv/sail-riscv`; the
   fork is `org`, and `gh pr create` needs `--repo` pinned.
   Next candidate for D5/D4: **A1** (PMP default-deny divergence).
2. ~~Produce a privileged-only coverage number.~~ ~~Re-run and re-baseline.~~
   **Both done 2026-08-31**, from scratch on a wiped corpus.
   `demo/privileged/run.sh` is now the single entry point: it generates all six
   privileged generators strictly serially, derives the exclusion list, replays
   once, and reports every scope off that one trace into
   `demo/privileged/results/SUMMARY.md`, each figure with the command that
   produces it. `demo/privileged/status.sh` shows progress mid-run. Use these
   rather than assembling a number by hand.
   See [[riscv-testgen-clean-run-baseline]].
3. **Split the 479 ELFs that did not reach SUCCESS into model fault vs framework
   fault.** This is the biggest remaining hole in the numbers and the user's
   single most-repeated ask ([[riscv-testgen-status-reporting]]). The failures
   are concentrated, not random: on RV64, 87 CSRs had no passing instruction,
   almost all `pmpcfg1`-`15`, `pmpaddr16`-`63`, the `stateen` family and the
   trigger CSRs — WARL or configuration-dependent registers, where a legalised
   read-back not matching a static expectation is far likelier than a model
   defect. Expect most of these to be ours.
4. **Get the work reviewed by someone outside it.** No code review, no external
   reader, repository still private, D4 and D5 both defined as *merged* PRs and
   neither merged. This is now the honest limiter on any completion claim —
   see [[riscv-testgen-review-gap]].
5. **The two weakest required areas, both now measured**: `sys/vmem_ptw` at
   **15.0%** (the page-table walker — depth, not feasibility) and `sys/pma` at
   **18.8%**. Neither needs new generator capability. Every PMA attribute is a
   per-region field in the model's own config JSON, so a PMA test is "run this
   access under a config whose region forbids it"; a `pma_cases.py` alongside
   the other three extractors is the obvious shape.
6. **Close the riscv-arch-test gap** ([[riscv-testgen-act4-comparison]]). Their
   141 hand-written privileged tests reach 524 spans against our 441. Two named
   items recover most of it and need no new capability: `sfence.vma` variants
   (one opcode) and misaligned accesses *through translation* (an address
   offset on an existing VM case).
7. **Widen the oracle across the 16-config matrix** (G4). isla cannot follow —
   its `B129` bitvector cannot represent VLEN 256 or 512 — but the oracle runs
   the real model and has no such limit. Frame this as the hybrid earning its
   keep, which is what it is.

Deliberately *not* rushed: **D4's merged CI PR** (gated on maintainers; the
proposal can commit to it credibly) and **Consideration B's Sail pointers**
(explicitly optional).

## How the work maps to the evaluation criteria

The proposal is scored on these six, in this order. Worth checking any planned
work against them.

1. **Coverage of the privileged architecture** — strongest single lever. This
   is why a privileged-only number matters more than raising the overall one.
2. **Long-term maintainability and extensibility** — the model-sourced parser
   is the argument here: an instruction added to Sail is picked up without
   editing a list. Say that explicitly.
3. **Inclusion/integration of existing open-source communities** — ACT4
   compatibility, isla-testgen, upstream PRs.
4. **Demonstrated technical skills in those communities** — merged PRs are the
   only hard evidence. See priority 1.
5. **Cost**
6. **Date of delivery**

## Keeping this honest

**Re-derive status; do not trust the table above as current.** G5, D1 and
Consideration G were re-derived **2026-08-31** from one clean privileged run; the
remaining rows are older snapshots (2026-08-27 and 2026-08-10). To refresh:

```
# the privileged number, from scratch -- generates, excludes, replays, reports
~/Documents/riscv-test-generation/demo/privileged/run.sh --clean   # ~1h, serial
~/Documents/riscv-test-generation/demo/privileged/run.sh --report  # measure only
~/Documents/riscv-test-generation/demo/privileged/status.sh        # progress mid-run

# the whole-corpus view
cd ~/Documents/riscv-test-generation/python-isla
python3 regression.py        # what got worse since the baseline (exit 1 if anything did)
python3 status_report.py     # per-extension pass/fail, split MODEL vs FRAMEWORK vs ROUTED
```

**Snapshot as of 2026-08-31**, privileged only, 1,144 ELFs, one clean run:
functions **81.4%** (184/226), branches **77.7%** (539/694), expressions **50.8%**
(799/1,572), all spans **61.1%** (1,522/2,492). Denominator: 2,868 instrumented
privileged spans minus 376 structurally unreachable, each with a written reason in
`demo/privileged/results/exclusions.json`. 665 of the 1,144 ELFs reached SUCCESS.

**Snapshot as of 2026-08-10** (whole current scope, superseded but not
contradicted — different denominator): branch coverage **71.8%** current scope /
**79.6%** combined; **0 framework failures** in scope; **2** model failures (A2,
PMP entry 0).

Rules learned on this project, each the hard way:

- **A passing suite is not evidence that anything was checked.** 47.7% of the
  corpus once passed while comparing empty expected-state tables. Before
  calling a requirement met, confirm the test *asserts* something — and prefer
  a negative control (sabotage it; it must fail) over an argument.
- **Never merge "the model is wrong" with "our generator is wrong".** That
  split is the most useful distinction on this project and the ratio is easy to
  get backwards. Promote a failure to MODEL only with a written-up reproducer
  in `findings.md`; "Sail passes, Spike fails" alone is more often an ISA-string
  or VLEN difference.
- **A finding earns only the failures its reproducer actually covers.** The 12
  `ssamoswap` failures were filed under a real, reproduced model defect found
  nearby, because the story fit. They were unrelated — the spec forbids Zicfiss
  in M-mode, the model implements that correctly, and the generator was emitting
  M-mode tests. The defect was genuine; the attribution was invented. Check that
  the fix actually changes the failing tests before crediting it with them.
- **Say which results are circular.** Oracle expected values come from
  `sail_riscv_sim`, so re-running there proves only self-consistency. Only the
  Spike run is independent evidence about the model.
- **Check a "we can't do X" claim before repeating it.** "The oracle backends
  are RV64-only" was stated confidently here and was simply false — an RV32
  suite generates and passes on both simulators. Verify, then assert.
- **A percentage is only as good as the run behind it.** The working
  `sail_coverage` is routinely left holding a *partial* replay — it read 13.9%
  overall on 2026-08-26 against a real ~60%. Span totals from `branch_info` are
  stable; percentages are not. Re-run the full corpus before quoting one, and
  date it wherever it lands.
- **Before quoting a diff total, check whether it is a move.** `sail-riscv` shows
  `-456` against upstream, which reads like a fork of the model. It is not:
  `csr_end.sail` gains +408 while every other file loses -446 — about 205 CSR
  clauses relocated, not deleted. Sum the destination against the sources before
  putting a number in the proposal. See [[riscv-testgen-fork-provenance]].
- **"Strong" needs a denominator.** PMP was recorded here as strong for months
  while exercising 1 of 18 access kinds. Nine passing scenarios against a
  44-case space is not coverage; it is nine cases. Ask what the deciding
  function can tell apart before calling any feature done.
- **Spike's ISA string must name every extension a test relies on.** This has
  now bitten three times in one day: `_svadu` for a `menvcfg.ADUE` case,
  `_sscofpmf` for LCOFI, `_zicbom`/`_zicboz`/`_zicbop` for cache-block ops.
  Each time the symptom was identical — Sail SUCCESS, Spike FAILURE — which
  reads exactly like a Golden Model divergence and is an argument to the
  simulator. Derive the ISA string from what the case needs; never hardcode one.
- **Check whether a second function legalises the input.** `pmpCheckRWX` says a
  store needs W; `pmpWriteCfg` reserves W-without-R and clears all three on
  write. A case derived from the deciding function alone gets silently rewritten
  by the model and then fails looking like a model bug. The rule that constrains
  an input often lives somewhere other than the rule that consumes it.
- **A number without its scope and its span kind is not a number.** One corpus,
  four defensible percentages: privileged 61.1%, current deliverable 49.6%, full
  RFP scope 24.0%, whole model 26.7%. Changing the span kind moves it another 27
  points (branches 77.7% against expressions 50.8%). Always say which, in the
  same sentence as the figure.

- **Measure the whole corpus, not one generator's output directory.**
  `coverage_report.py`'s `DEFAULT_ELF_DIRS` names six directories. I once
  measured the opcode sweep alone and reported 47.6%/39.9% as the project's
  coverage, then had to withdraw both. The file's own comments warn about
  exactly this.

- **Count deduplicated spans, not lines of `branch_info`.** The instrumented
  model has **15,157** spans. I reported 41,407 — raw line count, with each span
  counted once per occurrence. Off by 2.7x in the flattering direction.

- **Never run the generators in parallel.** Doing so took a 15 GB machine down
  hard and lost `/tmp` with it. `run.sh` exists to prevent a repeat: strictly one
  generator at a time, `nice -n 10`, `ISLA_MEM_LIMIT_GIB=2`. Do not "speed it up"
  by backgrounding the steps.

- **`pgrep -f foo.py` matches the script that is checking for `foo.py`.** This
  cost three separate mistakes in one session, including killing my own watcher
  and telling the user a sweep was running when it had been dead 34 minutes.
  Match the *process*: `ps -eo args | grep -q "^[^ ]*python[0-9.]* .*csr_sweep\.py"`,
  and cross-check against the age of the newest output file.

- **`isla-testgen` exits 0 when generation fails.** It prints "Generation attempt
  failed: Unable to continue" and writes no ELF. A runner checking only
  `if r.returncode:` therefore logs `GEN ok`, then runs a file that does not
  exist — which surfaces as a Sail/Spike *disagreement* and reads like a model
  divergence. Check the ELF exists before believing any verdict about it.

- **"REGRESSION" compares against an expectation table, not against history.**
  The PMP scenario tests were labelled a regression on RV32; the previous
  corpus's RV32 build fails identically on today's model, and every RV64 build
  passes. They have never worked on RV32. Check the old artefact before calling
  something a regression.

- **Coverage counts what executed, not what was verified — but measure the gap
  before caveating it.** The obvious worry is that the 479 non-passing ELFs
  inflate the figure. Replaying only the 665 self-verifying ones gives
  byte-identical coverage (1,522/2,492; 539/694): those 479 contribute zero
  unique spans. A caveat you can dissolve with one command beats a caveat you
  repeat in every report.

- **Nothing here has been reviewed by anyone outside the project.** Private repo,
  no code review, D4 and D5 both defined as *merged* PRs and neither merged. Any
  percentage-complete figure has to carry that, or it is not honest — the user
  asked for exactly this correction. See [[riscv-testgen-review-gap]].

- **When status changes, update this file and say what changed.** A stale north
  star is worse than none, because it is trusted.
