AI-Slop-Mop

A writing skill for LLM agents with two separate jobs done by three agents: write clean prose, and check that the argument inside it actually holds. Most "de-slop" tools stop at word choice. This one adds a companion agent that checks whether the reasoning is sound, since a paragraph can be free of every AI tell and still not follow from its own premises.

The three agents
Editor — rewrites a draft against the ruleset below. Leads with the point, uses active voice, keeps concrete detail, cuts dead weight.
Slop Checker — a companion eval, not a rewriter. Scans the edited draft for 62 rules across 8 mutually exclusive, collectively exhaustive categories and gates release on zero fabrication, no em-dash clusters, and answer-first structure.
Argument Auditor — reconstructs each substantive claim as premises leading to a conclusion (Toulmin-style: grounds, warrant, qualifier, rebuttal), then checks it against a 17-type logical-leap taxonomy across 5 families. Grades each leap load-bearing or decorative and gates release on zero unresolved load-bearing leaps.

Default-on: applies to prose output without needing to be invoked. Does not touch code, locked reference data, or structured/tabular content, since format follows content.

Why 8 categories and not a flat list

Every rule is assigned to exactly one category, at the level where the fix actually happens (mutually exclusive), and the categories run word to substance so nothing observed across the source material falls outside all eight (collectively exhaustive):

Category	Covers
A. Lexical tells	banned verbs and adjectives (delve, leverage, robust, transformative)
B. Phrase tells	filler qualifiers, zeitgeist openers, dead business metaphors
C. Sentence-structure tells	binary contrasts, colon reveals, fake-strong verbs, nominalization
D. Paragraph/discourse tells	throat-clearing, triplet addiction, summary-recap endings, fake-profound kickers
E. Rhetorical-stance tells	weasel attribution, importance puffery, audience flattery, false agency
F. Punctuation/format tells	em-dash overuse, decorative bullets, emoji, header inflation
G. Substance and accuracy	answer-first, sourced claims, no fabrication, Claim-Mechanism-Reality
H. Leftover chat/letter artifacts	"I hope this helps," "as an AI," letter phrasing in non-letters

A rule that could plausibly sit in two categories goes to the deeper one: "serves as a centralized hub" is a verb-choice fix (category C), not a vocabulary fix, because that's where the edit happens.

Why the Argument Auditor is separate from the Slop Checker

Clean prose and sound reasoning are different properties. A sentence can pass every lexical, structural, and formatting check and still smuggle in a conclusion its premises don't support. The Auditor catches:

Structural leaps — non sequitur, missing premise, circularity, suppressed conclusion
Generalization leaps — hasty generalization, composition, division, quantifier shift
Causal leaps — correlation treated as causation, post hoc, ignored confounders, reversed causation
Semantic leaps — equivocation, undefined load-bearing terms
Modal/normative/evidential leaps — authority-as-proof, "could" sliding into "will," is-ought slides, false dilemmas, base-rate neglect

It cannot verify that a premise is factually true, only that it's sourced or flagged [VERIFY]. Truth-checking is the Slop Checker's job (rule 46); the Auditor's job is the inference between premises and conclusion.

Usage
Edit mode (default): hand it a draft. It runs Editor → Slop Checker → Argument Auditor in sequence, loops on any failed gate, and returns the edited draft plus a short "What changed" note.
Detect mode: ask it to audit, scan, or flag without rewriting. It names each rule or leap type by number, quotes the exact line, and gives the fix in a few words. No rewrite, no guessing whether AI wrote it.
Release gates

Slop gate: zero fabrication · zero Category H artifacts · em dashes within budget, no clusters · answer-first satisfied · tell density under threshold · point and edge intact.

Argument gate: every major claim reconstructs into premises → conclusion · zero unresolved load-bearing leaps · every load-bearing premise sourced or flagged [VERIFY] · qualifiers match the strength of the grounds · rebuttal conditions acknowledged where the claim is contestable.

Both gates must pass before output ships.

Known limitations
This is a pattern-and-inference checker, not a fact-checker or a taste arbiter. It can certify an argument is validly structured and still not know whether a premise is true, or whether the underlying idea is good. That judgment stays with the person using it.
Tuned initially in a Singapore public-sector register (no em dashes as rhythm crutch, no absolute uniqueness claims, quantitative anchoring). The core 62 rules and 17 leap types are register-agnostic; a domain-specific overlay (tone, banned clichés) is meant to sit on top, not replace them.
Detection checklists catch surface and structural tells. They don't confirm a piece says anything worth reading. That's a separate, harder problem this skill doesn't claim to solve.
Attribution

Slop rules synthesized and reworded, not copied, from:

petergyang/no-ai-slop (MIT)
xr0zv/no-ai-slop
jalaalrd/anti-ai-slop-writing
hardikpandya/stop-slop
stephenturner/skill-deslop
Byk3y/no-slop
realrossmanngroup/no_ai_slop_writing_rules
Wikipedia:Signs of AI writing (CC BY-SA, WikiProject AI Cleanup)
H.E.A.R.T. framework (addition-side checklist: sourced evidence, answer-first, real voice, trust signals)

Argument layer built on the Toulmin model of argumentation and standard informal-logic fallacy categories, both public-domain analytical frameworks, not sourced from any single repo.

No source file is reproduced verbatim; all rules are restated and reorganized into the MECE structure above.

License

MIT, consistent with the primary upstream source (petergyang/no-ai-slop). Attribution to the sources above should be preserved in derivatives.
