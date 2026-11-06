# SWARM — Shared Planner / Doer / Checker framework
## For dbxwong prompt libraries — v1, 2026-11-06

This file is the shared contract for all four libraries in this folder.
Each library imports it by reference and then adds only its own domain
rules. Do not copy-paste a rival version of these rules into a library:
if the swarm changes, change it here, and note the version in each
library header.

> **Spelling note — confirm with the owner before pushing.**
> The user wrote "Helmeiers catechism". Research found no match for
> "Helmeier" / "Helmeyer". The clear match is the **Heilmeier
> Catechism** (with an "i"), George H. Heilmeier, DARPA director
> 1975–1977. DARPA: "a set of eight questions that researchers and
> engineers answer to fully understand the context, costs, and effort
> required to execute a given project."
> Source: https://www.darpa.mil/about/heilmeier-catechism
> All drafts below use "Heilmeier". If the owner meant something
> else, rename before pushing. See CHANGES.md, Open items.

---

## 0. Governing thought (Minto)

**One sentence:** Every task runs as Planner plans, Doer does, Checker checks, Metrics defines success, and a Red Team Council of domain experts tries to break it — in a bounded loop of
max 3 iterations, and nothing ships unless the Checker and the Red Team both pass it
against primary sources.

Everything below exists to support that sentence. If a rule below
conflicts with it, the sentence wins.

---

## 1. The three roles — separate contexts on purpose

Collapsing any two roles into one context reintroduces the
self-review blind spot the swarm exists to avoid. The same
reasoning that produced a gap will not reliably catch it two
sentences later.

| Role | Inputs | Outputs | Forbidden |
|---|---|---|---|
| **Planner** | Goal, constraints, sources, definition of done | Ordered steps, each with specific, checkable acceptance criteria; loop budget (max 3); precedence rules; conflict rules up front | Never executes a step. Never edits the artifact. Never judges a result. Never rewrites criteria after work starts, except by a formal re-plan (see §4). |
| **Doer** | One step + its acceptance criteria + prior Checker feedback only — not the full plan, not other Doers' drafts | Artifact + assumptions made + sources / evidence for each claim | Never grades its own output. Never writes its own criteria. Never fixes other steps. Never claims DONE. |
| **Checker** | Planner's criteria + Doer's artifact + primary sources / tool output, in a **clean context** (no Doer reasoning, false starts, or justification) | Per-criterion PASS / FAIL, specific failures with quoted evidence, severity (CRITICAL / HIGH / LOW); overall verdict | Never rewrites or edits. Never adds new criteria. Never approves on "looks good". Never passes if any CRITICAL or HIGH fails. `DONE_WITH_CONCERNS` = FAIL. |

### Handoff contract

```
Planner → step {id, description, acceptance_criteria[]}
Doer    → result {step_id, artifact, notes, sources[]}
Checker → verdict {step_id, approved: bool, feedback[]}
          → back to Doer on FAIL
          → back to Planner for re-plan on HIGH / CRITICAL
          → to Orchestrator to stop on PASS / cap / stall
```

### Swarm orchestration (writing / research)

Independent-first, then synthesis.

1. Each researcher / Doer works in parallel from the same brief,
   returns structured output + sources + confidence, without
   seeing peers.
2. A single Synthesizer (the Planner, or a separate role) merges,
   resolves conflicts, and owns the final text.
3. Front-load conflict rules in the brief so the swarm gathers
   instead of arbitrating.
4. Verification uses a stronger / slower model as refuter, not a worker.

**Conflict precedence** (front-load this in every brief;
inference — no single source stated this exact ordering):
1. Primary source / tool output
2. Planner's original spec / criteria
3. Newer / more specific evidence
4. Majority only as a tie-breaker, never as truth

---

## 2. Loop engineering — 7 rules

1. **Code owns the loop, model owns the step.** The orchestrator
   is a plain fixed sequence, not another agent. It owns budget,
   halting, and routing. The model decides only the next edit
   inside a step. This stops the model redefining its own stop rule.
2. **Progress is verified external state, not self-report.**
   Progress = checks passed, issues closed, facts verified —
   counted, not claimed. Never let the Doer declare victory.
3. **Hard cap + no-progress + stall, checked before every
   iteration.** See §3.
4. **Re-run the FULL gate every time, on the FULL original spec.**
   Each pass re-checks all criteria against the original brief,
   not just the changed section. One final full run after cleanup.
   Narrow re-checks let iteration-1 regressions slip through.
5. **No-regression rule.** If a gate passes fewer checks than the
   previous iteration, revert the last change immediately. Do not
   "fix forward" from a regressed state.
6. **Deterministic checks before LLM critique, and critic never
   rewrites.** Schema / tests / lint / source lookup first; model
   critic only for what a script cannot check. Checker returns
   PASS / FAIL + specific failures with evidence. It does not edit.
7. **Every exit is a named terminal state, reported with evidence.**
   See §3. A precise stuck-report is a good outcome; a fake
   "done" is the worst outcome.

### Loop budget

- **Outer loop: max 3 iterations. Per-step: max 2 retries
  (3 attempts total). Never raise it autonomously.**
  Only the user may raise the cap.
- Iteration counter starts at 1 and is stated at the start of
  every iteration.

---

## 3. Termination — stop only for a named reason

Copy this block into every library. Check the conditions in
this order, before every iteration.

```
LOOP: max 3 iterations. Iteration counter starts at 1 and is stated
at the start of every iteration. Only the user may raise the cap —
never raise it yourself.

STOP ONLY FOR A NAMED REASON:
- CONVERGED: Checker returns PASS on every acceptance criterion,
  after a full re-run of all gates on the full original brief.
- CAPPED: iteration = 3 and Checker has not passed. Stop, return
  best-so-far + remaining failures. Do not do a 4th pass.
- STALLED: same failure survives 3 attempts, OR output is
  near-identical to the previous iteration, OR re-plan produces
  the same scope as the previous plan, OR you alternate between
  two states (A → B → A). Stop, report the stuck failure +
  2 hypotheses.
- NO-PROGRESS: progress count (checks passed / issues closed)
  does not move for 2 consecutive iterations. Stop.
- REGRESSED: a gate passes fewer checks than the previous
  iteration. Revert the last change and stop if it recurs.
- ESCALATED: HIGH / CRITICAL Checker failure, merge conflict,
  or destructive action. Stop and hand to the human with the trace.

On any non-CONVERGED stop, do NOT claim success. Report:
terminal state, iteration count, best-so-far, remaining failures.
```

Report the iteration count and the terminal state in one line
in the final output, e.g. `Loop: 2 iterations — CONVERGED.`

---

## 4. Re-plan — the only way criteria change

Criteria belong to the Planner. If a Checker failure shows the
plan itself is wrong:

1. Checker routes to Planner, with the quoted failure.
2. Planner issues a formal re-plan: new step list, new
   acceptance criteria, and one line saying what changed and why.
3. The iteration counter does **not** reset.
4. If the re-plan produces the same scope as the previous plan,
   that is STALLED. Stop.

Never silently rewrite criteria mid-loop to make a failing
draft pass.

---

## 5. Minto Pyramid Principle — structure every output

Source: Barbara Minto, *The Pyramid Principle* (1987, 3rd ed. 2009).
McKinsey, 1960s–70s.

1. **Answer first — one governing thought.** Every document has
   a single top sentence that answers the question. Everything
   below supports it. 2–5 supports per level; recurse until atomic.
2. **SCQA intro; A = governing thought.** Situation (agreed
   context) → Complication (what changed) → Question (raised in
   the reader's mind) → Answer (the governing thought).
   The A *is* the top of the pyramid.
3. **The three pyramid rules — non-negotiable at every level:**
   - Parent summarises children (pull-back rule).
   - Siblings are the same kind of idea — all reasons, all steps,
     all findings, never mixed.
   - Siblings are logically ordered — deductive, chronological,
     structural, or by degree of importance.
4. **Action titles, not topic titles.** Headings are complete
   thoughts / insights, not labels like "Background" / "Update".
   For action groups, summarise by the *effect* of doing the
   actions; for situation groups, summarise by the *inference*
   from their similarity.
5. **Prefer inductive at the top; deductive only short and low.**
   Inductive = set of like reasons → inference. Deductive =
   major premise → minor premise → conclusion. Max 4 points in
   a deductive chain, max 5 in an inductive group; if more,
   regroup. Present action before argument — what to do, then why.
6. **Question / answer dialogue + "So What?" test.** Every point
   must raise a question the next line answers. If a datum does
   not directly support the point above it, move it to appendix
   or delete it.

---

## 6. MECE — a 4-step test (Checker runs this, not a vibe check)

Source: Minto coined MECE at McKinsey, late 1960s. Mutually
Exclusive = no overlap; Collectively Exhaustive = no gaps.
"Every item belongs in exactly one bucket, and every item has
a bucket." MECE is a *test*, not a framework.

1. **Place test (overlap).** Pick 3 real items and place each.
   If any fits in 2 buckets, fail ME. Watch hidden overlaps:
   subset in two places, e.g. "Online, Retail, Mobile"
   (mobile ⊂ online).
2. **Gap test (exhaustive).** Ask "what else could it be?" If you
   can name something with no home, fail CE. Add a bucket or a
   small "Other" — if "Other" is the biggest bucket, the split
   is wrong.
3. **Sum test.** If buckets are numbers, they must add back to
   the total. If not, something is double-counted or missing.
   Prefer splits MECE by construction: an equation, a process,
   one dimension at a time, or "X vs. not X".
4. **Same-kind + relevance test.** Buckets at one level must be
   the same kind and one dimension only — e.g. "Northeast, South,
   West, online" mixes geography with channel. Each bucket must
   matter to the decision: "Would the answer change if this
   bucket turned out big?" If not, fold it into "Other".
   MECE but useless is still a fail.

Common failures: mixing dimensions; hidden overlap; forgotten
branch; a flat list of 12 bullets instead of a tree; false
parallels ("discovery, planning, exiting").

**MECE self-audit (Slop-Mop rule):** if one line trips two rules,
assign it to the deeper category (substance, then stance, then
structure, then phrase, then lexical) and report it once.

---

## 7. Heilmeier Catechism — Checker asks this of every plan

Source: George H. Heilmeier, DARPA director 1975–1977.
DARPA, https://www.darpa.mil/about/heilmeier-catechism

Canonical 8 questions (verbatim from DARPA):

1. What are you trying to do? Articulate your objectives using
   absolutely no jargon.
2. How is it done today, and what are the limits of current practice?
3. What is new in your approach and why do you think it will be
   successful?
4. Who cares? If you are successful, what difference will it make?
5. What are the risks?
6. How much will it cost?
7. How long will it take?
8. What are the mid-term and final "exams" to check for success?

**Checker condensed form (7 questions, inference — adaptation,
not canonical).** Use this when the full 8 would bloat the check:

1. What are we trying to do, in plain words with no jargon?
2. How is it done today, and where exactly does that break?
3. What is actually new here — and why would it work *now*?
4. Who cares, and what changes if we succeed?
5. What are the risks, the payoffs, and the cheapest way to be wrong?
6. What will it cost, and how long will it take — honestly?
7. What is the mid-term exam and the final exam? How will we
   know, in advance, that we failed?

If any Heilmeier question has no answer, that is a Checker
failure (HIGH if it is load-bearing, LOW if decorative).
Do not invent an answer to close it. Flag it [VERIFY] or
return it to the Planner.

---

## 8. Veteran test — "Imagine I'm a veteran, what would they say?"

No named published test with this exact wording was found.
Treat it as an *operational* Checker persona, not a citable
framework. The Checker adopts the stance of a 20-year
practitioner who has seen this exact pitch fail before.

Run all six, in order. Any FAIL is a Checker failure.

1. **Swap test.** Could this paragraph be pasted into another
   company / domain / topic unchanged and still sound plausible?
   If yes, FAIL — generic.
2. **Density test.** Underline 3–5 concrete facts, names,
   numbers, dates, or trade-offs in the first 200 words.
   Fewer = FAIL.
3. **Primary-source test.** Every load-bearing claim traces to a
   primary source (not a secondary summary). Unverifiable
   statistic or invented citation = FAIL — remove or verify.
4. **Objection test.** Name the 3 strongest objections a 20-year
   practitioner would raise, the edge case where this fails, and
   what they would do differently on Monday morning. If the draft
   has no counter-argument, FAIL.
5. **Stance test.** A veteran has an opinion. What is *rejected*
   here, and why? Purely balanced hedging with no choice = FAIL.
6. **Deeper-not-larger test.** What should be *cut*? If the fix
   is always "add more", FAIL. Require a cut list, with a reason
   each cut does not lose meaning.

### Deeper, not larger — the cut rule

Every Checker verdict must include a **Cut list** (may be empty
only if the Checker states why nothing can be cut):

- Item to cut, and
- Why cutting it does not lose meaning.

A revision that only adds length is not an improvement.
If the Doer adds words to fix a Checker failure, the Checker
must name what was removed to pay for them, or fail the
revision on density.

---

## 9. Checker — the full gate (all libraries)

The Checker runs, in this order, in a clean context:

1. **Deterministic checks first.** Format, counts, word limits,
   bracket format, sums, dates, links resolve. Script / lookup
   what can be scripted / looked up.
2. **Primary-source accuracy.** Every load-bearing claim traces
   to a primary source named with a date. Secondary summaries do
   not count. Unverifiable = [VERIFY] or FAIL, never smoothed over.
   Never invent statistics, quotes, examples, citations, DOIs,
   or links. Hard fail.
3. **Slop-Mop (AI-Slop-Mop v6).** The 62 rules in 8 MECE
   categories (A–H) + the 17-leap Argument Auditor (L1–L17).
   See `AI-Slop-Mop/SKILL.md`. Slop gate + Argument gate must
   both pass.
4. **Minto + MECE.** §5 and §6 above. Governing thought present?
   Action titles? Siblings same-kind and ordered? MECE 4-step
   test passed?
5. **Heilmeier.** §7 above. All 8 (or condensed 7) answered,
   or flagged.
6. **Veteran test.** §8 above, including the cut list.
7. **Red Team Council.** §11 below. Metrics Agent scores the draft
   against the success metrics set at planning time, then the
   domain Council tries to break it. A CRITICAL Red Team finding
   fails the gate even if steps 1 to 6 pass.
8. **Deeper not larger and Blind Spot Audit.** §12 below. The draft
   must get denser, not longer, and the Sociotechnical Analyst
   must answer what the draft is not considering. A revision that
   only adds length, or a blind spot that is load-bearing, fails
   the gate.

**Severity:** CRITICAL = fabrication, wrong primary fact, or a
load-bearing leap. HIGH = unverifiable load-bearing claim,
MECE failure, missing Heilmeier answer, veteran-test FAIL.
LOW = style / density issue that does not change the conclusion.

**Verdict:** PASS only if zero CRITICAL and zero HIGH, and the
Red Team Council returns no CRITICAL finding.
`DONE_WITH_CONCERNS` = FAIL.

---

## 10. Metrics Agent — define success before work starts

A separate Metrics Agent runs with the Planner, before the Doer
starts. It does not write the artifact. It defines how success
will be measured, so the Checker and the Red Team have a fixed
target and the Doer cannot move it later.

1. **Identify 3 to 5 success metrics.** Each metric names what
   good looks like, how it is measured, and the pass threshold.
   Example metrics, adapted per domain: primary-source coverage,
   factual accuracy, decision usefulness, completeness against
   the brief, and clarity for the named reader.
2. **Set the measure of success in one line per metric.** Format:
   metric, measurement, threshold. Example: primary-source
   coverage, share of load-bearing claims with a named primary
   source and date, threshold 100 percent.
3. **Freeze the metrics.** The Planner records them in the plan.
   The Doer, the Checker, and the Red Team use the same metrics.
   No role changes a threshold mid-loop to make a failing draft
   pass. Only a formal re-plan can change them, and the iteration
   counter does not reset.
4. **Score, do not describe.** At check time, the Metrics Agent
   returns a score per metric, with evidence, not a general
   impression. A metric below threshold is a HIGH failure, unless
   it is load-bearing, in which case it is CRITICAL.

The Metrics Agent is a module, not a fourth author. It can run
as a separate agent in a clean context, or as a named step inside
the Planner. Either way, its output is a fixed metric sheet that
the rest of the swarm must use.

## 11. Red Team Council — mixture of experts, by domain

After the Checker passes steps 1 to 6 in §9, the Red Team Council
runs as the final adversarial gate. Its job is to break the draft,
not to polish it.

### How the Council is chosen

The Planner names 3 to 4 domain experts for the task, based on
the domain, not a fixed list. Each expert has a distinct lens,
and no two experts share the same lens.

For work prompts, one seat is always the Sociotechnical Analyst
from §12. That analyst covers people, process, tools, incentives,
power, and adoption. The other seats change by domain.

Domain examples, to adapt per task:
- Executive Bios: financial analyst, corporate governance expert,
  Sociotechnical Analyst, and a reader who will use the brief
  in a meeting.
- Strategy Paper Review: economist, operator, Sociotechnical
  Analyst, and risk expert.
- Alphasense Research: market analyst, customer and buyer expert,
  Sociotechnical Analyst, and channel or ecosystem expert.
- General writing: subject expert, sceptical reader, Sociotechnical
  Analyst, and a veteran practitioner from §8.

If the domain has no clear experts, the Planner states that gap
in the plan and uses the four general lenses above.

### How the Council runs

1. **Independent first.** Each expert works alone, in a clean
   context, with the draft, the metric sheet from §10, and the
   primary sources only. No expert sees another expert draft.
2. **Attack, do not edit.** Each expert lists attacks: the
   strongest counter-argument, the missing evidence, the edge
   case where the conclusion fails, the alternative interpretation
   the draft ignores, and the claim a domain expert would
   challenge first. Each attack quotes the line and names the
   metric it breaks.
3. **Council synthesis.** A Council Chair, who is not a Doer,
   merges the attacks, removes duplicates, and ranks them by
   severity: CRITICAL, HIGH, or LOW. The Chair does not soften
   an attack to reach agreement. A single CRITICAL attack is
   enough to fail the gate.
4. **Verdict.** The Council returns PASS only if no CRITICAL
   attack survives and no HIGH attack is load-bearing. Otherwise
   it returns FAIL with the ranked attacks and the metrics they
   break. The verdict goes back to the Planner for re-plan, or
   to the Doer for a fix, within the same loop budget of max 3.
5. **No new work in the verdict.** The Council never rewrites
   the draft and never adds new acceptance criteria. It flags
   and grades only.

### Red Team rules

- The Red Team runs once per iteration, after the Checker, not
  in parallel with the Doer. A draft that fails the Checker does
  not reach the Red Team.
- The Red Team uses the same loop budget. It does not reset the
  iteration counter and it does not raise the cap.
- A Red Team FAIL is a loop iteration, not a separate loop.
  The next iteration re-runs the full gate in §9, including the
  Red Team, on the full brief.
- The Red Team must name what would change its mind. For each
  CRITICAL attack, state the evidence or the change that would
  close it. An attack with no close condition is a LOW finding,
  not a CRITICAL one.

## 12. Deeper not larger — Blind Spot Audit

This layer produces the deeper version, not the bigger version.
Length is not depth. Depth is more truth per word, fewer
assumptions left untested, and a clearer statement of what was
cut and why.

### Deeper not larger rule

1. Every revision must increase density, not length. Count facts
   per 200 words. If density falls, the revision fails, even if
   it answers a Checker point.
2. A revision that adds words must name what was removed to pay
   for them. No net growth unless the Checker required new
   evidence, and then only the evidence plus one line of
   explanation.
3. Every Checker verdict includes a Cut list with a reason for
   each cut. An empty Cut list must state why nothing can be cut.
4. The final output includes a one-line Deeper version note:
   what was cut, what was tightened, and what question was
   answered more directly.

### Blind Spot Audit — what am I not considering

Run this audit once per iteration, after the Red Team, in a clean
context. Ask these 10 questions and answer each with evidence or
with a named gap:

1. What assumption about people is load-bearing here?
2. Who loses if this succeeds, and what will they do?
3. What breaks at scale, under time pressure, or with low skill?
4. What happens when incentives change or the reward moves?
5. What is the second-order effect, and the third?
6. What would make this advice wrong in 12 months?
7. Whose voice is missing, and what would they say?
8. What does the data not show, and what was not measured?
9. What would a well-informed rival do differently?
10. What have we already tried, and why did it fail?

If a question has no answer and the point is load-bearing, that
is a HIGH failure. Return it to the Planner with the question
named.

### Frameworks not yet in the swarm — check them here

The swarm already uses Minto, MECE, Heilmeier, and the Veteran
test. The Audit adds these lenses when they fit the task. Do not
force all of them. Name the ones you used and the ones you ruled
out, with one line each.

- Sociotechnical Systems, Trist and Emery: social and technical
  systems must be designed together. A tool that fits the tech
  but breaks the workflow fails.
- Systems Thinking, Meadows: leverage points, feedback loops,
  stocks and flows. Look for the loop, not only the event.
- Second-order effects and Pre-mortem: imagine the plan has
  failed, then work back to why.
- Chesterton Fence and Inversion: do not remove a rule or a step
  until you know why it exists, and ask what would guarantee
  failure, then avoid it.
- Cynefin: separate clear, complicated, complex, and chaotic
  problems. Do not use a clear-problem method on a complex one.
- Diffusion of Innovations and Technology Acceptance: who
  adopts first, who resists, and what makes adoption stick.
- Normal Accidents and Human Factors: tight coupling plus
  complexity creates failure. Design for the tired operator,
  not the ideal one.
- Jobs to be Done and Wardley Mapping: what job is the reader
  hiring this output to do, and which parts are custom and
  which are commodity.

### Sociotechnical Analyst — a required Red Team seat for work prompts

The Sociotechnical Analyst is not a general critic. It checks
the fit between people, process, tools, and organisation.

Run these 6 checks and report each as PASS or FAIL with evidence:

1. Roles and skills: who does the work, with what skill, and
   what happens when that person is absent?
2. Workflow and handoffs: where does work cross teams, and
   where does it wait, stall, or get reworked?
3. Tools and fit: does the tool fit the real workflow, or does
   the workflow bend to fit the tool?
4. Incentives and power: who is rewarded for this, who is
   penalised, and who can block it without owning it?
5. Adoption: what changes on Monday morning, who must change
   first, and what support makes that change possible?
6. Unintended consequences: what extra work, what new risk,
   and what informal workaround will this create?

A FAIL on a load-bearing check is a CRITICAL Red Team finding.
The Analyst must name the person or the team who would feel the
failure first, not only the abstract risk.

## 13. What the swarm does NOT do

- The Checker never rewrites. It flags, quotes, and grades.
- The Planner never executes. It plans, sets criteria, and re-plans.
- The Doer never grades itself. It produces, cites, and lists
  assumptions.
- The Metrics Agent never writes the artifact. It defines and
  scores success only.
- The Red Team never rewrites. It attacks, ranks, and returns
  a verdict only.
- No role raises the loop cap. Only the user does.
- No role claims CONVERGED on a non-CONVERGED stop. Report the
  real terminal state.

---

## Sources for this framework

- Loop engineering: Chaitanya Giri, "Loop Engineering: Designing
  Agent Loops That Converge", 3 Jul 2026,
  https://munderdiffl.in/blog/loop-engineering-for-ai-agents/
- Anthropic, "Building Effective Agents", Schluntz & Zhang,
  19 Dec 2024
- Evaluator–Optimizer pattern,
  https://github.com/ksksingh022/ai_system_design/blob/HEAD/p05-evaluator-optimizer.md
- Planner, Worker, and Reviewer, Cloud Engineer Lab,
  https://www.cloudengineerlab.com/blog/build-a-multi-agent-system-planner-worker-reviewer
- ywc-agentic SKILL.md,
  https://github.com/yongwoon/ywc-agent-toolkit/blob/HEAD/claude-code/skills/ywc-agentic/SKILL.md
- Verification Loop, MaaawKit,
  https://github.com/mathias7799/maaawkit/blob/HEAD/plugins/maaaw-kit/skills/verification-loop/SKILL.md
- Minto, Barbara, *The Pyramid Principle* (1987, 3rd ed. 2009)
- MECE: https://www.mece.in/us/learn/what-is-mece (updated 27 Sep 2026)
- Heilmeier Catechism, DARPA,
  https://www.darpa.mil/about/heilmeier-catechism
- Slop grounding: petergyang/no-ai-slop (MIT), Wikipedia: Signs
  of AI writing (CC BY-SA), H.E.A.R.T. framework
- Toulmin model of argumentation; standard informal-logic
  fallacy categories (public-domain analytical frameworks)
