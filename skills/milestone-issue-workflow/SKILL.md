---
invoke: user
---

# Milestone/Task → GitHub Issue Workflow

Turns a Milestone→Task execution plan (the output of the `project-planning`
skill, or any equivalent plan document already broken into milestones and
tangible-output tasks) into a **live, reviewable set of GitHub Issues and
PRs**, with a specific hierarchy and a review/merge discipline that keeps a
human reviewer in full control of what lands. Project-agnostic — apply the
same structure on any repo, not just the one it was first used on.

## Core structure

- **One Issue per Milestone.** Title it after the milestone name. Body is a
  short description of what the milestone achieves (its "definition of
  done"), plus a **checklist of its task issues**, referenced by number:
  ```markdown
  ## Tasks
  - [ ] #124 Model the registers/fields for X
  - [ ] #125 Build SV covergroups for X
  - [ ] #126 Build Python stimulus + pytest for X
  ```
  GitHub auto-renders `#N` as a linked issue and shows the checklist
  progress on the milestone issue itself — this is the whole point of using
  a checklist instead of prose: the milestone issue's completion percentage
  updates automatically as task issues close.
- **One Issue per Task**, each with a single tangible output (per
  `project-planning`'s own "every task shows a tangible output" rule — this
  is exactly what makes a task issue small enough to review quickly). Body:
  what the task produces, and a back-link to its parent milestone issue
  (`Part of #123`).
- **Ordering constraint**: task issues must exist *before* the milestone
  issue's checklist can reference their numbers. Create all of a milestone's
  task issues first, capture the returned issue numbers, then create (or
  edit) the milestone issue with the checklist filled in.

## Native sub-issue linking (real indentation, not just a checklist)

The checklist above (`- [ ] #124`) is text — GitHub also has a **native
sub-issue** relationship (`addSubIssue` GraphQL mutation) that makes task
issues render as an actual indented/collapsible tree under their milestone
issue, both in the Issues list and in the Projects v2 table view. Do both:
keep the checklist (readable in the issue body itself, shows a progress
percentage) *and* add the native link (gives real tab-indentation without
needing a separate reordering step). They're additive, not a replacement for
each other, and removing a sub-issue link later (`removeSubIssue`) is
non-destructive if it doesn't look right.

Old `gh` CLI versions (e.g. 2.4.0) have no subcommand for this — drive the
GraphQL API directly:

```bash
# one query, aliased, to fetch every milestone issue's node ID at once
gh api graphql -f query='
query {
  repository(owner: "<owner>", name: "<repo>") {
    m1: issue(number: 21) { id }
    m2: issue(number: 24) { id }
  }
}'

# then one mutation per task issue — pass the task by URL (subIssueUrl) so
# you never need to look up the CHILD's node ID, only the parent's
gh api graphql -f query='
mutation {
  addSubIssue(input: {issueId: "<parent_node_id>", subIssueUrl: "https://github.com/<owner>/<repo>/issues/<task_number>"}) {
    issue { number }
    subIssue { number }
  }
}'
```

For dozens of links, write a small script keyed by `milestone_number ->
[task_numbers]` (reuse the same map you built for the checklist step) and
loop the mutation — same milestone-by-milestone batching discipline as bulk
issue creation. Spot-check afterward with a `subIssues(first: N) {
totalCount nodes { number } }` query on a couple of milestones rather than
trusting the loop's exit code alone. A milestone issue that is deliberately
task-less (e.g. a pure "already-done, informational" milestone with no
distinct task issues of its own) should show `totalCount: 0` — don't treat
that as a bug, just confirm it was intentional in the plan document.

## PR discipline

- **One or more PRs per task issue** — a task can take more than one PR if
  the work naturally splits, but never bundle multiple *different* task
  issues into one PR. Small, single-purpose PRs are the entire reason for
  this granularity; don't undo it by batching.
- Every PR references its task issue explicitly (`Fixes #N` if the PR alone
  completes the task, `Refs #N` if it's partial). Never leave a PR
  unlinked to an issue.
- **If one PR closes multiple issues, use a separate `Fixes #N` / `Closes
  #N` line per issue — never comma-separate them on one line.** Confirmed
  twice on real PRs in this project: `Closes #62, #63.` only auto-closed
  `#62` — GitHub's keyword parser silently drops every issue after the
  first when they're comma-joined (same failure mode as trailing text
  after the number, e.g. `Fixes #117 (DV side)`, which also only parses
  the first token). After merging a multi-issue PR, verify with `gh api
  graphql` querying `pullRequest(number: N) { closingIssuesReferences {
  nodes { number } } }` — don't trust that every issue named in the body
  actually closed just because the wording looks unambiguous.
- **Merge authority stays with the human reviewer, always.** Open the PR,
  then stop — never merge your own PR on this workflow unless explicitly
  told to for that specific PR. This is the load-bearing rule: the entire
  point of the issue/PR structure is that someone else reviews and approves
  every change before it lands.

## When review requests changes

Do **not** silently amend the same PR/commit to satisfy review feedback
without a paper trail. Instead:
1. Open a **new task issue** describing the specific change requested,
   cross-linked to the original task issue and PR (`Follow-up to #N`).
2. Do the follow-up work as a **new PR** referencing that new issue.

This mirrors the git discipline of "new commit per fix, never amend a
pushed commit" — same idea, one level up at the issue/PR granularity. It
keeps the history of *what was asked for* vs. *what was actually delivered*
auditable, rather than collapsing review iterations into an untraceable
edit.

## Bulk-creating issues from an existing plan document

When a plan document already has a Milestone → Task table structure (e.g.
`project-planning`'s template, or a project's own roadmap file):

1. Extract each milestone's task list from the plan document.
2. For each milestone, create its task issues first (`gh issue create --repo
   <owner>/<repo> --title "..." --body "..."`), capturing each returned
   issue number.
3. Create the milestone issue (or edit it if it already exists as a plain
   markdown section) with a checklist built from the task issue numbers
   captured in step 2.
4. Do this milestone-by-milestone rather than creating all task issues for
   every milestone up front — keeps the running issue-number bookkeeping
   simple and lets you sanity-check one milestone's shape before repeating
   the pattern ~N more times.
5. Report back which issues were created (numbers + titles), not just "done"
   — the user needs the actual issue list to start assigning/reviewing.

For large plans (dozens of tasks), write a small script that does steps 2-3
programmatically rather than issuing one `gh` call per task by hand — but
still create issues in milestone-sized batches so a mistake in one
milestone's shape doesn't propagate silently across all of them before
anyone notices.

## Target dates

Add dates only where they were actually discussed/confirmed — never invent a
backward-planned schedule silently, per `project-planning`'s "never fabricate
business-critical unknowns" rule. Where a milestone/task has no real date
yet, write `TBD` (or `No date` for an explicitly separate no-date track)
rather than leaving the field blank/absent — an explicit `TBD` shows the gap
was considered, not overlooked.

Two places to record a date once confirmed:
1. **The plan document itself** — add a `Target date` column to each
   milestone's task table (or a one-line note under the milestone heading).
   This is the source of truth.
2. **GitHub issues** — a Project v2 board may show a `Target date`/`Start
   date` column, but these belong to a separate, newer **"Issue Fields"**
   surface (`Repository.issueFields` / `IssueFieldDate`), not plain Project
   custom fields, even though they render inside the Project view. Two traps
   here, both worth avoiding on the first try:
   - The ID shown for these fields in a Project's own `fields { ... }` query
     (`ProjectV2Field`) looks valid but is the **wrong ID** for this purpose —
     passing it to `updateIssueFieldValue` fails with "Issue field with id N
     not found", and passing the same field to
     `updateProjectV2ItemFieldValue` fails with "Issue field values cannot be
     updated using the updateProjectV2ItemFieldValue mutation."
   - Get the **real** field ID from `repository.issueFields` instead:
     ```graphql
     query {
       repository(owner: "<owner>", name: "<repo>") {
         issueFields(first: 20) {
           nodes { ... on IssueFieldDate { id name dataType } }
         }
       }
     }
     ```
     Then set it per issue (needs the issue's node `id`, not its number):
     ```graphql
     mutation {
       updateIssueFieldValue(input: {
         issueId: "<issue_node_id>",
         issueField: { fieldId: "<real_IssueFieldDate_id>", dateValue: "2026-08-17" }
       }) { clientMutationId }
     }
     ```
   This is the real, board-visible date column — worth doing over the
   body-text fallback below whenever the target repo's Project already shows
   `Target date`/`Start date` as a field. If for some reason `issueFields`
   comes back empty (the fields don't exist yet on that repo), fall back to
   prepending a plain `**Target date:** YYYY-MM-DD` line to the issue's body
   (same fetch-body/edit-body pattern as the bulk body-text fixes above).

If new dates surface later (a fabricated one gets confirmed, or a TBD gets
a real date), treat it as a normal edit to both places — this isn't a
one-time pass.

## Native GitHub Milestones (progress bar + due date, separate from the issue checklist)

Once "also use native Milestones" is confirmed (see the non-decision below),
create one real `Milestone` object per plan-Milestone and put every one of
its issues (the milestone issue itself plus all its task issues) into it —
this is what makes GitHub's own Milestones page show a percent-complete bar
and open/closed counts per milestone, same as the checklist does inside a
single issue, but as a first-class sidebar view.

1. Create the milestone object (title should match the milestone issue's
   title, for a single consistent name across both systems):
   ```bash
   gh api repos/<owner>/<repo>/milestones -f title="Milestone 8 -- Component Review" \
     -f description="..." -f state=open [-f due_on="2026-08-17T12:00:00Z"]
   ```
2. Put every issue belonging to that milestone into it:
   ```bash
   gh issue edit <n> --repo <owner>/<repo> --milestone "Milestone 8 -- Component Review"
   ```
   `--milestone` takes the title string, not the milestone's own number — no
   need to track the returned milestone number for this step.
3. **`due_on` timestamp gotcha, confirmed live**: passing midnight UTC
   (`"2026-08-17T00:00:00Z"`) gets silently stored/returned as the *previous*
   day (`2026-08-16`) — a real, reproducible off-by-one, not a display
   artifact (checked via the milestones list endpoint immediately after
   creation). Fix: use a mid-day UTC time instead
   (`"2026-08-17T12:00:00Z"`) — round-trips correctly. If a due date already
   landed one day early, `gh api repos/<owner>/<repo>/milestones/<num> -X
   PATCH -f due_on="...T12:00:00Z"` fixes it in place.
4. Only set `due_on` where a real date exists (same "don't fabricate" rule
   as the Target date section above) — leave it unset for genuinely
   undated milestones rather than inventing one just because a reference
   example (another project's Milestones page) shows every row dated.

## When the user renumbers milestones by editing GitHub directly

A user may reorder milestones themselves (e.g. dragging bars in a Projects
Timeline view, or editing issue titles by hand) rather than asking for it
in chat. Don't assume your last-known numbering is still current — **live
re-fetch before acting**:

```graphql
query {
  organization(login: "<org>") {
    projectV2(number: <N>) {
      items(first: 100) {
        nodes { content { ... on Issue { number title milestone { title } } } }
      }
    }
  }
}
```

A common resulting inconsistency: the user edits an issue's **title** (e.g.
renaming "Milestone 17" to "Milestone 12" to move it earlier in the
sequence) but the underlying **native Milestone object's title** is
untouched, so the issue's title and its milestone assignment now disagree
(`gh issue view N --json title,milestone` shows two different numbers).
The fix is almost always a **rename**, not a reassignment: since the issue
is already linked to the right native Milestone object, retitle that
object to match (`gh api repos/.../milestones/<native_num> -X PATCH -f
title="..."`) rather than moving issues between milestones. Confirm this by
checking whether the issues the user is asking about are already under the
milestone in question before assuming anything needs re-linking.

Renumbering one milestone up/down cascades — re-grep the **entire** plan
document and **every issue body** for `Milestone N` mentions afterward, not
just the moved milestone's own text. Real categories of stale reference
found in practice: cross-task mentions ("incorporate Milestone 11/12
results" — one of the two numbers shifted, the other didn't), gating/range
statements ("reached once Milestones 8-13 close" — a non-contiguous list
after a swap, can't stay a clean range), and even genuinely pre-existing
bugs unrelated to the current renumbering (a title that referenced the
wrong milestone from an earlier pass) — fix those too while already
auditing the same text.

## Native Milestones have no custom order — a real, permanent limitation

Unlike a Projects v2 board (which supports arbitrary drag-reorder via
`updateProjectV2ItemPosition`), GitHub's native Milestones list has **no
position field at all**. A milestone's creation order is fixed forever once
created and cannot be changed by any PATCH. This matters because:

- The Milestones page's default sort is **"Newest first" (creation order),
  not due date or title.** Renaming a milestone's title to reorder it
  logically (e.g. moving "Milestone 17" to become "Milestone 12") does
  **not** change where it appears under this default sort — it still
  displays according to when the underlying object was actually created.
- The reliable, non-destructive fix: make sure every milestone's `due_on`
  is set in the same order as its logical number, then have the user
  select **"Due date"** in the page's own Sort dropdown — verify this
  produces the right order via the API first (`?sort=due_on&direction=asc`)
  before telling the user to switch it.
- The only way to fix the *default* ("Newest first") ordering itself is to
  **delete and recreate every milestone from scratch** in the exact right
  sequence — recreating only the out-of-place ones doesn't work, because
  new milestones get the *next* sequential number after the current
  maximum, not a number that fills the gap. This means re-linking every
  associated issue and re-setting every due date/description. That's a
  large, disruptive operation for what is fundamentally a cosmetic
  sort-order preference — lay out the cost plainly and let the user decide,
  don't do it unilaterally just because they mentioned the order looks off.

## Multiple date surfaces drift independently — always live-diff, don't assume

Once a plan has (1) the plan document's own target-date column, (2) the
Project board's `Target date` Issue Field, and (3) native Milestones'
`due_on`, these are **three separate stores** with no automatic sync
between them. A user dragging bars in a Projects Timeline view changes (2)
directly without touching (1) or (3) at all. After any user-driven UI
change, pull all relevant surfaces fresh and diff them explicitly (e.g. one
query listing every native Milestone's `due_on`, one query listing every
milestone-descriptor issue's `Target date` field value, compared side by
side) rather than trusting whichever surface you personally wrote last.
Whichever surface reflects the user's most recent direct edit is the one to
treat as current truth, and the others should be reconciled to match it —
not overwritten with your own older values.

A related trap already hit once: a milestone-descriptor issue's own
`Target date` can drift from its *child tasks'* dates (e.g. the parent
shows one date, all its children show a different, mutually-consistent
one) — almost always an accidental side effect of dragging the parent's
own bar rather than deliberate intent. Flag it explicitly and offer to
resync the parent to match its children rather than silently picking one.

## Assignees

Set `--add-assignee <login>` (or `--add-assignee "@me"` for the
currently-authenticated account) on every issue, same milestone-by-milestone
batch as everything else. Don't guess who owns what if the project has more
than one real contributor — ask for the actual name-to-milestone/task
mapping rather than defaulting everyone to one person. Defaulting to the
authenticated user is reasonable *only* when the plan document's own
staffing note says the project is genuinely solo (e.g. "Staffing: solo,
directing AI agents") — state that assumption explicitly rather than
applying it silently.

## What this skill does not decide for you

- **Labels/taxonomy** (e.g. a `milestone` label vs. a `task` label,
  priority labels, area labels) — apply whatever labeling convention the
  target repo already uses, or ask if none exists yet. Don't invent a new
  taxonomy silently.
- **Native GitHub Milestones feature** — this workflow's "milestone issue"
  is a plain Issue with a checklist, not GitHub's built-in Milestones
  object. If the target repo also wants native Milestones (for the
  due-date/percent-complete UI GitHub provides separately from issues),
  that's an additional, explicit decision — don't conflate the two without
  confirming which (or both) the user wants. Once confirmed, see the section
  above for how to actually wire it up.
- **Whether a task issue needs one PR or several** — genuinely task-
  dependent; don't force a 1:1 assumption if a task's own tangible output
  naturally splits into sequential PRs.
