---
name: project-planning
description: >-
  Builds evidence-based, stakeholder-ready execution plans with day-by-day or
  milestone-level task breakdowns, staffing models, and honest risk/tradeoff
  disclosure. Use when the user asks to create a project plan, execution
  timeline, delivery schedule, sprint/milestone breakdown, resourcing plan, or
  wants to estimate "how long will this take" / "how many people do we need"
  for a real engineering or business deliverable. Also use when a user wants
  to compress, extend, restructure, or re-cost an existing plan.
---

# Project Planning

Builds project plans the way a credible internal planning document should
read: grounded in verified facts, organized around trackable units of
completion, honest about what's compressed and what breaks if it is, and
pitched at the right abstraction level for who's reading it.

## Core principles

1. **Verify before estimating.** Read the actual codebase, prior art, or
   reference material before putting a number on anything. A plan built from
   "read the real `pmp_control.sail` file and confirmed the `Backend`
   interface's exact method signature" is defensible; a plan built from "test
   frameworks usually take a few months" is not — and falls apart the moment
   someone checks it. If you can't verify something (team availability, other
   project commitments, actual billing rates), say so explicitly instead of
   estimating it.

2. **Organize around trackable units, not just calendar time.** Pick the
   thing stakeholders actually care about completing — a feature, an
   extension, a module, a component — and structure milestones around
   "X is done" rather than "we did stuff for two weeks." Include a completion
   tracker table mapping each unit to the milestone/day it lands.

3. **Every task shows a tangible output. Research is never a standalone
   task.** "Study the API" is not a task; "API client built, handles auth and
   retries" is. If research is genuinely needed, fold it into the task that
   produces the first real artifact — don't give it its own line.

4. **Model staffing realistically.** Account for onboarding/ramp-up time,
   staggered start dates if not everyone starts day one, and whether
   workstreams can genuinely run in parallel. Two people on the *same*
   workstream hits diminishing returns fast (shared files, coordination
   overhead); two people on *independent* workstreams parallelize well. Don't
   assume linear speedup from headcount.

5. **When compressing an estimate, show the math and name what broke.**
   State the compression method plainly (e.g., "applying the same ~30%
   reduction already used elsewhere in this engagement" — reuse an existing,
   defensible factor rather than inventing a new one each time). Say exactly
   which phase absorbed the cut and what slack disappeared as a result. Never
   silently shrink a number to hit a requested target without disclosing the
   tradeoff.

6. **State known risks plainly, in their own section.** Not "risks may
   exist" — the specific thing that could slip, why, and what the fallback
   is if it does (absorb into a later phase, cut scope, extend deadline).

7. **Decide what unused buffer time does.** If a contingency block goes
   unneeded, define upfront whether it's reinvested (more depth/coverage) or
   just returns the schedule early — don't leave it undefined.

8. **Map every required deliverable to a specific point in the schedule.**
   A table: deliverable → where/when it lands. This is what lets someone
   verify nothing was dropped.

9. **Give resourcing scenarios when asked, computed consistently.** "Faster
   or slower with different staffing" should use the same estimation method
   across every row of the table, not a different guess per scenario.

10. **Never fabricate business-critical unknowns.** Actual staff
    availability, what else a team is committed to, real billing rates or
    margins — if you don't have real data, say exactly that and what's needed
    to fill the gap, rather than inventing a plausible-sounding number.

11. **Match abstraction level to the audience, and ask if unsure.**
    Engineers need file names and function signatures. Stakeholders need
    plain language and tangible outputs, no implementation detail. If a
    request to "modify the plan" is ambiguous, ask what specifically before
    reworking it — schedule/staffing changes and abstraction-level changes
    are different edits.

12. **Treat the plan as one living document.** Edit it incrementally as
    scope, staffing, or deadlines change. After every edit, re-verify the
    day/week math is internally consistent — totals per phase should sum to
    the stated grand total every time, not just on the first draft.

## Workflow

1. **Gather the real inputs.** What's the actual deliverable? Is there
   existing code/prior art to ground estimates in? What's already been
   committed to (an existing draft quote, a prior estimate) that this plan
   needs to reconcile with, not silently contradict?
2. **Verify, don't guess.** Read the relevant code/docs/specs directly.
   Note exact facts (file paths, function names, existing config state) you
   can cite later instead of hand-waving.
3. **Break the deliverable into trackable units** (features, extensions,
   modules) and rough-size each based on what you actually found in step 2.
4. **Design the staffing model.** Solo start vs. parallel from day one?
   Genuine independent workstreams, or one team stacked on one thing? Include
   ramp-up time honestly.
5. **Draft phases/milestones**, each with tasks that show tangible output,
   tagged to the trackable unit they belong to.
6. **Add the supporting tables**: completion tracker, deliverable mapping,
   known risks, (if asked) resourcing scenarios.
7. **State the math plainly** in a Context section: total time, how it was
   derived, and what it's grounded in.
8. **Iterate**: when the user asks for a change, identify exactly what class
   of change it is (scope, staffing, timeline compression, abstraction level,
   pricing) before touching the document, and re-verify all totals after.

## Plan document template

```markdown
# [Project Name] Execution Plan

## Context
[What this plan is for, why now, what it must reconcile with if anything
already exists. State the total time/effort and how it was derived —
verified from real inputs, not top-down guessed.]

## High-Level Approach
[2-4 bullets: phases, staffing shape, what happens when.]

## Phase 1 — [Name] (Dates/Days X-Y)
**Staffing:** [who]

| Day(s) | Unit | Task (tangible output) |
|---|---|---|
| ... | ... | ... |

**Milestone: [Unit] — complete.**

## Phase 2 — [Name] ...
[repeat, parallel workstreams as separate sub-tables if applicable]

## Completion Tracker
| Milestone | Units covered | Target completion |
|---|---|---|

## Deliverables → Where They Land
| # | Deliverable | Delivered |
|---|---|---|

## Known Risk, Stated Plainly
[What could slip, why, and the fallback if it does. What's explicitly NOT
covered by this estimate.]
```

## Anti-patterns (things this skill has been burned by before)

- **A "study X" or "research Y" task with no other output.** Always fold
  into the task that produces something.
- **Compressing a total without saying what got cut.** If Phase 1 shrank
  from 5 weeks to 4, say which specific piece of work absorbed the cut.
- **Re-deriving a new, different compression factor every time asked to
  adjust the timeline.** Reuse the same stated method for consistency, or
  explain why this time is different.
- **Answering "how much should we price this" or "how many people are
  available" with a confident invented number.** Give the calculation
  framework and known reference points, flag what's genuinely outside your
  visibility, and let the human supply the missing input.
- **Silently keeping a stale number** (old week count, old risk-table
  entry) after a restructure. Re-read the whole document after a major edit
  and check every cross-reference, not just the section you touched.
- **Treating the plan as finished once the work starts.** A forward plan is
  a hypothesis about ordering and sizing, and it is routinely wrong in ways
  worth capturing. Once a phase is genuinely done, rewrite that section
  *backwards from what happened* — what the real dependency was, what the
  real cost was, and which assumption failed. Keep the original framing
  visible where it still holds, so the correction reads as evidence rather
  than revisionism.

  Two failure shapes are worth looking for specifically, because they
  generalise:
  - **A foundation that never appeared in the plan at all.** If several
    phases each hit the same underlying blocker, that blocker was the real
    first milestone and nobody wrote it down.
  - **Sizing inverted.** The phase labelled "large, novel" arriving in an
    afternoon on top of infrastructure a "medium" phase had already built
    is a signal the plan measured *unfamiliarity* rather than *work*.

- **A completion criterion that a broken deliverable would also satisfy.**
  "All tests pass", "the pipeline runs clean", "no errors in the log" are
  each satisfied by a thing that does nothing at all. This is not
  hypothetical: on one project 47.7% of a test corpus passed while checking
  nothing, and the milestone had been marked done off the green run. Write
  the criterion so that it *cannot* be met by an empty implementation —
  name the output that must exist, or pair the check with a negative
  control ("and sabotaging X must make it fail"). Where a phase's evidence
  is a number, state how the number could be wrong before quoting it.

- **A completion tracker that mixes what you owe with what someone else
  owes.** Items blocked on an upstream fix, a vendor, or a review queue are
  not the same class as items you can finish today, and a single
  "outstanding" count invites the wrong conclusion about whose problem the
  project has. Split them, and label the blocked ones with who owns them.
  The split usually flatters the team — which is exactly why it needs to be
  computed honestly rather than reached for.
