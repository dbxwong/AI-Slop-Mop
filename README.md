# AI-Slop-Mop

A writing skill for LLM agents. Three agents handle two jobs: producing clean prose, and checking that the argument inside it actually holds. Most de-slop tools stop at word choice. This one adds a check for whether the reasoning is sound, because a paragraph can be free of every AI tell and still not follow from its own premises.

## The three agents

- **Editor** rewrites the draft against the ruleset below: leads with the point, uses active voice, keeps concrete detail, cuts dead weight.
- **Slop Checker** is a companion eval, not a rewriter. It scans the edited draft against 62 rules in 8 mutually exclusive, collectively exhaustive (MECE) categories, and gates release on zero fabrication, no em-dash clusters, and answer-first structure.
- **Argument Auditor** reconstructs each substantive claim as premises leading to a conclusion (Toulmin-style: grounds, warrant, qualifier, rebuttal), checks it against a 17-type logical-leap taxonomy across 5 families, grades each leap load-bearing or decorative, and gates release on zero unresolved load-bearing leaps.

Applies by default: it runs on prose output without being invoked. It doesn't touch code, locked reference data, or structured or tabular content, since format follows content.

## Why eight categories, not a flat list

Each rule belongs to exactly one category, at the level where the fix actually happens (mutually exclusive). The categories run from word choice to substance, so every tell has a home (collectively exhaustive):

| Category | Covers |
|---|---|
| A. Lexical tells | Banned verbs and adjectives (delve, leverage, robust, transformative) |
| B. Phrase tells | Filler qualifiers, zeitgeist openers, dead business metaphors |
| C. Sentence-structure tells | Binary contrasts, colon reveals, fake-strong verbs, nominalization |
| D. Paragraph and discourse tells | Throat-clearing, triplet addiction, summary-recap endings, fake-profound kickers |
| E. Rhetorical-stance tells | Weasel attribution, importance puffery, audience flattery, false agency |
| F. Punctuation and format tells | Em-dash overuse, decorative bullets, emoji, header inflation |
| G. Substance and accuracy | Answer-first, sourced claims, no fabrication, Claim-Mechanism-Reality |
| H. Leftover chat and letter artifacts | "I hope this helps," "as an AI," letter phrasing in non-letters |

A rule that could sit in two categories goes to the deeper one. "Serves as a centralized hub" is a verb-choice fix (category C), not a vocabulary fix, because that's where the edit happens.

## Why the Argument Auditor is separate from the Slop Checker

Clean prose and sound reasoning are different properties. A sentence can pass every lexical, structural, and formatting check and still smuggle in a conclusion its premises don't support. The Auditor catches:

- **Structural leaps**: non sequitur, missing premise, circularity, suppressed conclusion
- **Generalization leaps**: hasty generalization, composition, division, quantifier shift
- **Causal leaps**: correlation treated as causation, post hoc, ignored confounders, reversed causation
- **Semantic leaps**: equivocation, undefined load-bearing terms
- **Modal, normative, and evidential leaps**: authority-as-proof, "could" sliding into "will," is-ought slides, false dilemmas, base-rate neglect

It can't verify that a premise is factually true, only that it's sourced or flagged [VERIFY]. Truth-checking is the Slop Checker's job (rule 46); the Auditor checks the inference between premises and conclusion.

## Usage

**Edit mode (default).** Hand it a draft. It runs Editor, then Slop Checker, then Argument Auditor, loops on any failed gate, and returns the edited draft plus a short "What changed" note.

**Detect mode.** Ask it to audit, scan, or flag without rewriting. It names each rule or leap type by number, quotes the exact line, and gives the fix in a few words. No rewrite, no guessing whether AI wrote the original.

## Release gates

**Slop gate:** zero fabrication; zero Category H artifacts; em dashes within budget with no clusters; answer-first satisfied; tell density under threshold; point and edge intact.

**Argument gate:** every major claim reconstructs into premises and a conclusion; zero unresolved load-bearing leaps; every load-bearing premise sourced or flagged [VERIFY]; qualifiers match the strength of the grounds; rebuttal conditions acknowledged where the claim is contestable.

Both gates must pass before output ships.

## Known limitations

This is a pattern-and-inference checker, not a fact-checker or a taste arbiter. It can certify that an argument is validly structured and still not know whether a premise is true or whether the underlying idea is good; that judgment stays with the user.

It was tuned initially in a Singapore public-sector register: no em dashes as a rhythm crutch, no absolute uniqueness claims, quantitative anchoring. The 62 rules and 17 leap types are register-agnostic; a domain-specific overlay for tone and banned clichés is meant to sit on top of them, not replace them.

Detection checklists catch surface and structural tells. They don't confirm a piece says anything worth reading; that's a separate, harder problem this skill doesn't try to solve.

---

## The operative prompt (v5)

*The text below is what runs when the skill executes.*

### Part 1: Slop rules (Editor + Slop Checker)

**How the rules are organized (MECE).** Eight categories, word to substance. Each rule has one home, placed at the level where the fix happens, so no rule is double-counted. The categories span every level where slop appears: word choice, stock phrases, sentence shape, paragraph flow, rhetorical stance, punctuation and format, substance and accuracy, and leftover chat artifacts. A pattern that fits two places goes to the deepest one.

#### A. Lexical tells (word choice)

- **Buzz-verbs:** delve, leverage, utilize, foster, harness, streamline, unlock, elevate, embark, supercharge, spearhead, facilitate, empower. Use the plain verb.
- **Puffery adjectives:** robust, seamless, cutting-edge, transformative, pivotal, multifaceted, meticulous, intricate, paramount, vibrant, rich, comprehensive, groundbreaking, innovative, ever-evolving.
- **Hedging and filler adverbs:** really, just, actually, genuinely, honestly, simply, truly, fundamentally, crucially, importantly, inherently, inevitably, arguably.
- **AI transition words:** moreover, furthermore, additionally, notably, indeed, ultimately, overall, used to glue rather than connect.
- **Cliché nouns:** landscape, ecosystem, realm, tapestry, hub, portal, journey, paradigm, synergy, beacon.

#### B. Phrase tells

- **Filler qualifiers:** it's worth noting, it's important to note, at the end of the day, when it comes to, at its core, in terms of, with regard to, in order to, going forward.
- **Zeitgeist openers:** in today's world, in the age of, in the fast-paced world of, in the digital era.
- **Meta-article phrases:** in this article, let's dive in, let's explore, in this section, without further ado.
- **Dead business metaphors:** deep dive, move the needle, level up, game changer, low-hanging fruit, at the intersection of, drive impact.

#### C. Sentence-structure tells

- **Binary contrast / negative parallelism:** "Not X. It's Y." State Y directly.
- **Negative listing:** "Not a X. Not a Y. A Z." Say Z.
- **Colon reveal:** noun phrase, colon, dramatic lowercase clause. Rewrite as a plain sentence.
- **Choppy sentence syndrome:** connected ideas split into stubs. Join them.
- **Dramatic fragmentation:** "That's it. That's the whole thing." Use complete sentences.
- **Staccato triads:** "We simplify. We focus. We execute." Rewrite or vary.
- **Superficial -ing analysis:** highlighting, underscoring, reflecting, showcasing, illustrating. Replace with the real consequence or cut.
- **Fake-strong verb phrases:** "serves as a centralized hub for." Prefer is/has or a direct verb.
- **Nominalization:** "made a decision" for decided, "has the ability to" for can.
- **False ranges:** "ranges from X to Y," specific-sounding, content-free.
- **Rhetorical setups:** "What if I told you," "Think about it:," self-answered pairs.

#### D. Paragraph and discourse tells

- **Buried lede / throat-clearing:** "Here's the thing," "Let me be clear." Front-load the point.
- **Rule of three / triplet addiction:** cut arbitrary threes to the strongest item.
- **Robotic rhythm:** uniform sentence length, repeated shapes, stacked fragments.
- **Summary-recap endings:** "In conclusion," "Overall." End on the last concrete point.
- **Fake-profound kicker:** delete the final aphorism; end on a concrete sentence.
- **Synonym cycling:** repeat the clear word instead of rotating terms.
- **Mixed idea/topic units:** split multi-idea sentences and multi-topic paragraphs.
- **Hedging seesaw / both-sides:** take a side or state the specific tradeoff.

#### E. Rhetorical-stance tells

- **Faux-insight setup:** "What nobody tells you." Cut; let the claim stand.
- **Importance puffery / promotional drift:** "marks a pivotal moment," "stands as a testament," "enduring legacy." State the fact.
- **Editorializing asides:** "It's important to note," "no discussion would be complete without."
- **Audience flattery:** "Whether you're a solo founder or a Fortune 500 exec."
- **Weasel attribution:** "experts agree," "studies show." Name the source with a date or cut. (Reasoning-side twin: L15.)
- **False agency / narrator-from-a-distance:** inanimate subjects doing human verbs. Give the real actor.
- **Vague declaratives:** "this changes everything." Replace with the specific.

#### F. Punctuation and formatting tells

- **Em dashes:** none in short copy; one or two in long drafts only when they beat commas, periods, or parentheses. Remove clusters.
- **Exclamation spam:** zero in analytical prose.
- **Ellipsis abuse:** cut suspense or trailing dots.
- **Emoji in headings or bullets:** remove.
- **Mid-sentence bold for emphasis:** remove.
- **Decorative bullets:** convert to prose where prose reads better.
- **Header inflation:** cut headers the content doesn't need; fix inconsistent case.
- **Formatting leaks:** strip markdown or backticks in plain-text destinations.
- **Hashtag stacks:** remove trailing tag piles.

#### G. Substance and accuracy

- **Answer-first:** core answer in the first 150-200 words; each section opens with its point.
- **Sourced claims:** every statistic, finding, or claim gets a named source and date. Flag unverifiable ones as [VERIFY: needs source].
- **No fabrication:** never invent statistics, quotes, examples, citations, DOIs, or links. Hard fail.
- **Concrete over generic:** names, numbers, dates, mechanisms, examples beat abstractions. Protect the specific fact; don't smooth a useful detail into generic importance.
- **Firsthand specificity:** at least one example only someone who did the work could write.
- **Claim-Mechanism-Reality:** state the claim, how it works, what actually happens. Cut claims that can't carry all three.
- **Preserve edge:** sharpen a strong opinion; don't sand it down to sound balanced.

#### H. Leftover chat and letter artifacts

- **Letter phrasing in non-letters:** "I hope this message finds you well."
- **Collaborative sign-offs:** "I hope this helps!" "Let me know if you need anything else."
- **AI self-reference / meta-commentary:** "As an AI," "Certainly! Here's," "I'd be happy to."

**Positive frame.** Lead with the point, in active voice, and make every sentence earn its place. Open the writing up rather than dumbing it down, and write for the actual reader and the actual job. Keep the meaning; ask if it's unclear. Keep the existing structure unless it hurts the piece, and say why if you reorganize.

### Part 2: Slop Checker (eval)

Enforces cleanliness, not quality. Runs on every edit.

**Evidence rule.** Every flag quotes the exact line and cites the rule number. A flag without a quote is not a flag.

**Six passes**, in order:

1. Lexical (A, B)
2. Structure (C, D)
3. Stance (E)
4. Punctuation and format (F)
5. Substance (G)
6. Artifacts (H)

Record each hit with its line.

**Scoring.** Tell density equals hits per 100 words; target below 1.0 for analytical prose, 0 for short copy. Category pass means A-F and H at zero for short copy; for long drafts, no category above 2 hits and no rule fired more than once.

**Release gate**, all must hold: zero fabrication (rule 47); zero Category H; em dashes within the rule-36 budget, no clusters; answer-first satisfied (rule 45); tell density under threshold; point and edge intact.

**MECE self-audit.** If one line trips two rules, assign it to the deeper category (substance, then stance, then structure, then phrase, then lexical) and report it once. If a real tell fits no rule 1-55, log it as a candidate rule with a quoted example.

### Part 3: Argument Auditor (soundness agent)

A distinct agent. Runs after the Slop Checker on any prose that argues, recommends, evaluates, or persuades: memos, recommendations, screening rationales, deck narratives, investment or policy cases. Skipped for casual replies and pure description.

#### What it checks, and what it can't

- **Validity:** does the conclusion follow from the premises if the premises are granted?
- **Premise acceptability:** is each premise sourced (rule 46) or self-evident? Unsourced load-bearing premises are flagged, not assumed.
- **Soundness / cogency:** valid or strongly supported structure plus acceptable premises.

The auditor can't verify the empirical truth of a premise. It can certify that an inference is airtight and still not know the premise is factually correct. Truth-checking is rule 46's job; the auditor's job is the link between premises and conclusion.

#### Step 1: Reconstruct the argument (Toulmin-lite)

For each major claim, extract:

- **Conclusion:** the point being argued.
- **Grounds:** the stated premises and evidence.
- **Warrant:** the often-unstated principle that licenses moving from grounds to conclusion. This is where most leaps hide.
- **Qualifier:** how strong the claim is (certainly, probably, in these cases).
- **Rebuttal:** the conditions under which it would fail.

Write the argument as premises leading to a conclusion in plain lines. If it can't be reconstructed, the prose is too vague to be sound; flag that first.

#### Step 2: Logical-leap layer (17 types, 5 families)

A leap is any point where the conclusion outruns the grounds. Name the type, quote the line, and state the missing premise or the fix. Grouped MECE by defect location.

**Structural (the inference form fails)**
- **L1 Non sequitur:** conclusion doesn't follow from the grounds at all. State the actual gap.
- **L2 Enthymeme / missing premise:** a hidden premise is needed and isn't obvious or granted. Surface it; ask whether it holds.
- **L3 Circularity:** the conclusion is smuggled into a premise. Find independent support.
- **L4 Suppressed conclusion:** the real claim is implied but never stated, so it escapes scrutiny. State it.

**Generalization (scope outruns evidence)**
- **L5 Hasty generalization:** few or unrepresentative cases generalized to a broad rule. Bound the claim to the evidence.
- **L6 Composition:** what's true of the parts asserted of the whole.
- **L7 Division:** what's true of the whole asserted of each part.
- **L8 Scope / quantifier shift:** "some" or "many" slides to "all" or "most." Restore the true quantifier.

**Causal (mechanism unproven)**
- **L9 Correlation treated as causation:** name the mechanism or downgrade to association.
- **L10 Post hoc:** sequence treated as cause.
- **L11 Single-cause / ignored confounder:** one driver asserted where several act. Name the alternatives.
- **L12 Reversed causation:** direction assumed, not shown.

**Semantic (a term moves)**
- **L13 Equivocation:** a key term changes meaning mid-argument. Fix one definition.
- **L14 Ambiguity / undefined term:** a load-bearing term is never pinned down. Define it.

**Modal, normative, and evidential (strength or basis overstated)**
- **L15 Appeal to authority as proof:** "experts agree" used as the reason, not evidence. Give the actual grounds. (Style twin: rule 33.)
- **L16 Modal leap:** possible slides to probable slides to certain, or could slides to will. Restore the qualifier.
- **L17 Is-ought / value leap:** a prescription drawn from description alone, or a value premise smuggled as fact. Surface the value premise. Also covers false dilemma (excluded options) and base-rate neglect: name the missing option or the base rate.

#### Step 3: Grade each leap

- **Load-bearing:** the conclusion collapses without it. Must be fixed before release.
- **Decorative:** overstatement that doesn't carry the argument. Downgrade the wording.

For each leap: quoted line, leap type, load-bearing or decorative, and the fix (missing premise, restored qualifier, or named alternative).

#### Step 4: Steelman before you cut

State the strongest version of the argument the author could have meant. Audit that version, not a weak paraphrase. If the steelman survives, say so; a sound argument passes clean.

#### Argument release gate

All must hold:

- Every major claim reconstructs into premises leading to a conclusion.
- Zero load-bearing leaps unresolved.
- Every load-bearing premise is sourced (rule 46) or self-evident; unsourced ones carry [VERIFY].
- Qualifiers match the strength of the grounds (no rule-L16 overreach).
- The rebuttal condition is acknowledged where the claim is contestable.

Fail any gate: revise and re-run. A piece must clear both the Slop Checker gate and the Argument gate to ship.

### Workflow

1. Read the full draft. State the core point in one sentence; if that's not possible, ask.
2. **Detect request** (audit only, no rewrite): run the relevant checker, name each rule (1-62) or leap (L1-L17) by number, quote the line, give the fix. Stop; offer to edit.
3. **Edit request** (default):
   - Editor rewrites top to bottom against Part 1.
   - Slop Checker runs Part 2; loop until its gate passes.
   - Argument Auditor runs Part 3 on argumentative prose; loop until its gate passes.
4. Output the edited draft, a short "What changed" section, and an "Argument audit" section listing any leaps found and how they were resolved, or flagged if the fix needs a fact that isn't available.

Slop rules synthesized and reworded from the header sources; base skill petergyang/no-ai-slop under MIT, Wikipedia signs under CC BY-SA. The argument layer is built on the Toulmin model and standard informal-logic fallacy categories, which are not copyrightable. No source file is reproduced verbatim.

---

## Attribution

Slop rules synthesized, from:

- petergyang/no-ai-slop (MIT)
- xr0zv/no-ai-slop
- jalaalrd/anti-ai-slop-writing
- hardikpandya/stop-slop
- stephenturner/skill-deslop
- Byk3y/no-slop
- realrossmanngroup/no_ai_slop_writing_rules
- Wikipedia: Signs of AI writing (CC BY-SA, WikiProject AI Cleanup)
- H.E.A.R.T. framework (sourced evidence, answer-first, real voice, trust signals)

The argument layer is built on the Toulmin model of argumentation and standard informal-logic fallacy categories, both public-domain analytical frameworks, not sourced from any single repo. No source file is reproduced verbatim; all rules are restated and reorganized into the MECE structure above.

## License

MIT, consistent with the primary upstream source (petergyang/no-ai-slop). Preserve attribution to the sources above in derivatives.
