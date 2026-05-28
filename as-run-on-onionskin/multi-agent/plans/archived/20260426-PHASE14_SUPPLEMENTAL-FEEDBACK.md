# PHASE 14 SUPPLEMENTAL FEEDBACK

**Authors:** John M. Urban, Claude Code 2.1.104 (claude-sonnet-4-6)

**Purpose:** Open naming and design questions for the supplemental Phase 14 priorities
defined in `PHASE14_SUPPLEMENTAL-SPEC.md`. Resolve each question before handing the spec
to a Role 2 implementer.

---

## OPEN QUESTIONS ROUND 1

### Q1 — Final names for `--method` and `--ensemble-methods`

**Current:** `--method`, `--ensemble-methods`
**Spec recommendation:** `--growth-method`, `--growth-ensemble-methods`
**User alternatives:** `--growth-detection-method`, `--growth-model` (for `--method`);
`--growth-ensemble`, `--growth-ensemble-models` (for `--ensemble-methods`)

**Context:**
`--method` controls the trend-fitting function used to build the growth evidence track
(linear/isotonic/step/unimodal/ensemble). It is the most algorithm-facing flag in the growth
group. Naming options and their tradeoffs:

| Candidate | Pros | Cons |
|---|---|---|
| `--growth-method` | Simple, consistent with priority | "method" is generic |
| `--growth-detection-method` | Very explicit | Long; "detection" usually means the z-thresh step |
| `--growth-model` | Short | Ambiguous (could mean the whole pipeline) |
| `--growth-fit-method` | Specific to curve fitting | Adds a new word ("fit") |

For `--ensemble-methods`:

| Candidate | Pros | Cons |
|---|---|---|
| `--growth-ensemble-methods` | Consistent; "methods" signals it's a list | Slightly long |
| `--growth-ensemble` | Short | Ambiguous (could be a single method name) |
| `--growth-ensemble-models` | — | Incorrect — these are fitting methods, not ML models |

**Question for user:** Which names do you prefer?

**Answer from John M. Urban**:
- `--growth-fit-method`
- `--growth-ensemble-methods`

---

### Q2 — Final name for `--stage-weight-mode`

**Current:** `--stage-weight-mode`
**Spec recommendation:** `--growth-stage-weight-mode`
**User alternatives:** `--growth-stage-weight`, `--growth-stage-weighting`,
`--growth-stage-weight-rule`

**Context:**
This flag controls how replicate counts weight stages during origin refinement and triangle
model fitting. It has three choices: `replicate`, `stage`, `equal`. The `mode` suffix is
consistent with other choice-based flags (e.g., `--norm-mode`). Dropping `mode` makes the
name shorter but loses the signal that it's a mode selection. Alternatives:

| Candidate | Pros | Cons |
|---|---|---|
| `--growth-stage-weight-mode` | Consistent with `--norm-mode` pattern | Long |
| `--growth-stage-weight` | Short | Could be confused with a numeric weight value |
| `--growth-stage-weighting` | Reads naturally | Non-standard suffix |
| `--growth-stage-weight-rule` | Clear it's a policy | Unusual word choice |

**Question for user:** Which do you prefer?

**Answer from John M. Urban**:
- `--growth-stage-weight-mode` is fine

---

### Q3 — Final name for `--w-grid-kb`

**Current:** `--w-grid-kb`
**Spec recommendation:** `--growth-model-width-grid`
**User alternatives:** `--growth-w-grid`, `--growth-window-grid`

**Context:**
`w` stands for `wL`/`wR` — the left and right halfwidths of the asymmetric triangle model.
The flag provides a comma-separated grid of values (in kb) that are searched by grid-search
optimization. The existing `--growth-halfwidths` flag already uses "halfwidths" for the
mean-shift detection halfwidths. Using "halfwidth" again here could create confusion.
"Window" in `--growth-window-grid` could be confused with `--growth-window` (the mean-shift
scan window).

| Candidate | Pros | Cons |
|---|---|---|
| `--growth-model-width-grid` | Clear: model's width grid, not detection | Somewhat long |
| `--growth-w-grid` | Keeps familiar abbreviation | Opaque to new users |
| `--growth-window-grid` | Shorter | Conflicts conceptually with `--growth-window` |
| `--growth-tri-grid` | Short; "tri" = triangle | Unusual abbreviation |
| `--growth-halfwidth-grid` | Explicit | Conflicts with `--growth-halfwidths` naming |

**Question for user:** Which do you prefer? Note that `w` = `wL`/`wR` = asymmetric triangle
halfwidths. This context should inform the help string regardless of the flag name chosen.

**Answer from John M. Urban**:

See "More Feedback from John M. Urban" section below for more context.

--> universal = --asym-tri-model-halfwidth-grid 

--> growth pipeline =  --growth-asym-tri-model-halfwidth-grid

---

### Q4 — Should `--refine-smooth-kb` become `--growth-refine-smooth` (dropping -kb)?

**Spec recommendation:** Yes — drop `-kb` per Phase 14 convention (other renamed flags like
`--growth-halfwidths`, `--growth-trend`, `--growth-smooth` all dropped the unit suffix).
**Alternative:** Keep `-kb` for explicitness since the type is `int` and the unit matters.

**Context:** Phase 14 renamed `--halfwidths-kb-multi` → `--growth-halfwidths` (dropped `-kb`),
`--trend-kb-multi` → `--growth-trend`, etc. The convention is now to put units in the help
string, not the flag name. Consistency suggests dropping `-kb` from all the refine/model flags.
The user also listed `--refine-smooth-kb` → `--growth-refine-smooth` without the -kb, suggesting
they agree.

**Question for user:** Confirm dropping `-kb` from all 5 flags (`--refine-window-kb`,
`--refine-smooth-kb`, `--model-window-kb`, `--model-smooth-kb`)? The spec assumes yes.

**Answer from John M. Urban**:
- Yes dropping "kb" is a convention we should follow in general. "kb" is mentioned in the help string (or should be).
- Some of those flags may have additional renaming considerations spelled out below in "More Feedback from John M. Urban"

---

### Q5 — Should `--peak-summary`, `--peak-quantile`, `--peak-topk` become Universal flags?

**Current:** Growth-only flags with no equivalent in RMS or HMM.
**Spec recommendation:** Rename to `--growth-peak-*` (growth-prefixed only) for now.
**Open question:** Should these be promoted to Universal, with `--growth-peak-*` overrides?

**Context:**
`--peak-summary` controls how per-call peak signal height is summarized across replicate
stage medians for the "peak profile" output. Currently only the growth pipeline emits this
kind of profile. But the question of "how do you characterize peak height across replicates"
is conceptually general to any multi-stage pipeline.

**Does RMS use something similar?**
The RMS pipeline runs per-stage mean-shift detection and then unifies calls. It does not
currently compute a "peak-summary profile" in the same sense. The concept could apply if
RMS were extended to emit similar per-call signal summaries.

**Does HMM use something similar?**
The HMM pipeline computes stage-median tracks (Step 2) and emits per-call statistics, but
it does not have a peak-summary profile concept currently.

**Tradeoffs:**
- Universal promotion now: consistent with the Universal/override pattern; anticipates
  future RMS/HMM summary profiles; but adds Universal flags for a feature only growth uses.
- Growth-prefixed only: simpler; correct for current scope; can promote later when RMS/HMM
  actually need it; avoids premature abstraction.

**Recommendation:** Keep growth-prefixed for now. If/when RMS or HMM adopts similar summary
profile outputs, promote at that time.

**Question for user:** Agree with growth-prefixed-only approach? Or promote to Universal now?

**Answer from John M. Urban**:
- This should be made universal now with note in help string that it only applies to growth for now.
- From my understanding, this is computed on per-stage RCN profiles and is trying to give a single summary RCN profile. If that is true, then this is something universal that all pipelines might want to  be doing. It is certainly outside the scope of just modeling the growth track. 
- Anything in the growth pipeline that is concerned with RCN profiles (as opposed to growth track profiles) should be considered universal. We should look for this now so we can see what we want to pull out of growth later to apply to RMS and/or HMM pipelines.

---

### Q6 — Should `--stage-median-resolution` become a Universal flag?

**Current:** Growth-only.
**Spec recommendation:** Rename to `--growth-stage-median-resolution` (growth-prefixed only).
**Open question:** HMM also computes stage medians (Step 2). Should this be Universal?

**Context:**
`--stage-median-resolution` controls which resolution of bedGraph is used for stage-median
within-calls tracks (`best`, `base`, `hires`). The growth pipeline uses these for Step 8
(stage-median within-calls) output.

The HMM Step 2 computes combined stage-median tracks as part of its preprocessing, but the
concept of "resolution selection" for HMM inputs is handled differently (the HMM ingests
whatever manifest is provided). It is not clear that exposing a `--stage-median-resolution`
knob would be meaningful to HMM users right now.

**Recommendation:** Keep growth-prefixed for now.

**Question for user:** Agree?

**Answer from John M. Urban**:
- I agree that this flag seems tailored to the growth pipeline, at least for now.

---

### Q7 — `--growth-model-window` vs `--growth-model-window-kb` (similar for smooth)

The `--model-window-kb` flag takes an `int` in kb. If renamed to `--growth-model-window`, a
user might not know the unit is kb. The Phase 14 convention dropped units from flag names and
put them in help strings — but those flags (halfwidths, trend, smooth) had `kb` in the old
name as a legacy artefact. Here the unit is genuinely important context.

**Question for user:** Accept `--growth-model-window` (unit in help only), or prefer
`--growth-model-window-kb`? This applies equally to `--growth-model-smooth`, 
`--growth-refine-window`, `--growth-refine-smooth`.

The spec assumes the unit-dropping convention (consistent with the rest of Phase 14), but
this question is worth an explicit decision.

**Answer from John M. Urban**:
- we should drop "kb" in general
- it should be spelled out in the help text that it is expecting kb, although whether it makes sense to enforce an integer is debatable, but probably acceptable if we are dealing in tens of kb or hundreds of kb, less so if only "ones" of kb.
- as a rule for the repo, if the width being asked for is a halfwidth, we should use "halfwidth" instead of "window". We have not been consistent with naming CLI flags in this regard, and could benefit by identifying flags that might need name updates based on this.
- Some of those flags may have additional renaming considerations spelled out below in "More Feedback from John M. Urban"

---

### Summary — decisions resolved (Round 1)

| # | Question | Final Decision |
|---|---|---|
| Q1 | `--method` → ? | `--growth-fit-method` |
| Q1 | `--ensemble-methods` → ? | `--growth-ensemble-methods` |
| Q2 | `--stage-weight-mode` → ? | `--growth-stage-weight-mode` |
| Q3 | `--w-grid-kb` → Universal name? | Universal `--asym-tri-model-halfwidth-grid`; growth `--growth-asym-tri-model-halfwidth-grid` |
| Q4 | Drop `-kb` from all flags? | Yes; mention kb in help strings |
| Q5 | `--peak-*` Universal or growth-only? | **Universal now**; + growth overrides; help notes growth-only for now |
| Q6 | `--stage-median-resolution` Universal or growth-only? | Growth-only → `--growth-stage-median-resolution` |
| Q7 | Drop units; use "halfwidth" for halfwidths? | Yes; use "halfwidth" in flag names where the value IS a halfwidth |



## More Feedback from John M. Urban

**asymmetric triangle fitting and timing analyses should be universal, and part of all pipelines**
- Let's make that an immediate future direction. It should definitely be in RMS. And it should be also in HMM, although we have other methods for analyzing similar things using the HMM state path. Analyses like timing, progression, growth asymmetry, APS and so on should all be considered goals of all pipelines. We need to think of each pipeline as a means to identify amplicons with possibly unique ways of learning about their summits as well. But once we have amplicons, then all downstream aims are shared by all pipelines: summit refinement, timing, elongation asymmetry, and so on. 
- I would say we should prepare for the eventuality of the triangle options being universal right now. Add CLI flags to a universal section -- perhaps the timing section or a fork asymmetry section (?), and update the names in growth. The help strings for the universal flags can say they only affect the growth pipeline for now, but are in future directions for the other pipelines. 

**helpstring**
What was proposed:
--growth-stage-weight-mode 
  - How replicate counts weight stages in origin refinement and triangle model fitting. replicate: weight by count (default). stage: equal weight per stage. equal: all replicates equal.

As a rule, whenever "triangle model fitting" is used, please give it a little more context of what that means. And call it "asymmetric triangle model fitting" to better distinguish it from the triangle filter. Perhaps a short but full description would be "per-stage asymmetric triangle model fitting for growth and timing analyses" - or something like that.

This minimally affects
--growth-stage-weight-mode
--growth-model-window
--growth-model-smooth
--growth-model-width-grid


**Triangle-specific flags should mention triangle***
I would also add that the CLI flags that are specific to "per-stage asymmetric triangle model fitting for growth and timing analyses" should mention something about the triangle.

It seems like these flags are for the triangle:


--growth-model-window 
--> universal = --asym-tri-model-window 
--> growth pipeline =  --growth-asym-tri-model-window


--growth-model-smooth
--> universal = --asym-tri-model-smooth 
--> growth pipeline =  --growth-asym-tri-model-smooth


--growth-model-width-grid
--> universal = --asym-tri-model-halfwidth-grid 
--> growth pipeline =  --growth-asym-tri-model-halfwidth-grid


The CLI flags need to be more intuitive, and the help strings need to do a little more explaining. That is a general trend I am noticing.


**Order of CLI flags should follow order in which they are used across the pipeline**
- Let's make sure they are being presented in this logical ordering
- It makes it confusing if there are later steps referenced before earlier steps, etc.


**Step mentions in help strings**
- Each help string should make an attempt to say which steps are affected by the CLI flag
- ...where steps are defined by the output directory names
- The full output directory step names should be used so they are easier to catch later if the architecture changes and the help strings need updating
- Step mentions should be true for all flags, not just growth pipeline flags.
  - HMM and RMS should do this too
  - Flags in universal sections would be pipeline step mentions. An example for an APS flag might be: "hmm: 14-aps; growth: 13-aps; rms: 12-aps." Or that might be for the section header lines if it applies to all flags in the section.

---

## OPEN QUESTIONS ROUND 2

### Q8 — "window" vs "halfwidth" for `--refine-window-kb` and `--asym-tri-model-window`

**Q7 rule:** "If the width being asked for is a halfwidth, we should use 'halfwidth' instead
of 'window'."

**`--refine-window-kb`:** Spec description: "Half-window in kb around the initial peak estimate
within which the origin is refined." This IS a halfwidth (search radius, not full width).
By Q7 rule → `--growth-refine-halfwidth`. The spec uses this name pending your confirmation.

**`--model-window-kb` → `--asym-tri-model-window`:** Spec description: "Half-window in kb around
the refined origin within which the asymmetric triangle model is fit per stage." Also a
halfwidth. By Q7 rule → `--asym-tri-model-halfwidth`. However, you explicitly wrote
`--asym-tri-model-window` in the More Feedback section naming table.

**Question:** Confirm `--growth-refine-halfwidth` for `--refine-window-kb`? And should
`--asym-tri-model-window` become `--asym-tri-model-halfwidth` for Q7 consistency, or keep
`--asym-tri-model-window` as written?

**Answer from John M. Urban:**
- Yes to `--growth-refine-halfwidth`
  - Also, if it applies: `--refine-smooth-kb` -> `--growth-refine-smooth-halfwidth`
- Yes to `--asym-tri-model-halfwidth`
  - And to `--growth-asym-tri-model-halfwidth` if it applies.
  - This is a great application of the rule. Good catch. 

---

## COPILOT Repo-wide end-user CLI audit — naming and help findings

**Audit scope used for this pass:** live end-user argparse surfaces only.

- Main CLI: `onionskin.py:build_parser()`
- Standalone engines: `onionskin_core/engines/growth_model_engine.py:main()`, `onionskin_core/engines/rcn_mean_shift_engine.py:main()`
- User-facing scripts: `scripts/summit_inspector.py:main()`, `scripts/aps_cluster_experiments.py:main()`, `scripts/aps_cluster_report.py:main()`
- Deliberately excluded from the main report: internal/specialized diagnostics such as `scripts/compare_puffstep_outputs.py` and `scripts/rcn_summit_diagnostics.py`

### Overall judgment

- Phase 14 proper substantially improved the parser structure. The main CLI is now organized well, the `Universal`/pipeline-specific override split is coherent, and the strongest surfaces already feel much more intentional than they did before the 14.x rename work.
- The remaining naming debt is real, but concentrated. Most of it is either:
  - already inside the current supplemental Growth rename wave, or
  - a smaller HMM-specific terminology/abbreviation debt that should probably be handled in a follow-up rather than mixed into 14-S1/14-S2/14-S3.
- The bigger cross-cutting problem is now help-string quality rather than raw flag naming. Many flags have adequate names but still do not tell the user what pipeline step they affect, when they matter, or how they interact with universal vs pipeline-specific overrides.

### Naming findings

#### 1. High-priority naming work already justified by the supplemental spec

The current supplemental direction is correct. The live Growth parser still has a block of unprefixed or under-described flags that do not match the Phase 14 naming contract:

- `--method`
- `--ensemble-methods`
- `--stage-weight-mode`
- `--refine-window-kb`
- `--refine-smooth-kb`
- `--model-window-kb`
- `--model-smooth-kb`
- `--w-grid-kb`
- `--stage-median-resolution`

The user feedback in this file resolves the direction well:

- prefer `--growth-fit-method`
- prefer `--growth-ensemble-methods`
- keep `--growth-stage-weight-mode`
- use `halfwidth` where the value is a radius rather than a full window
- use explicit `asym-tri-*` naming for triangle-model surfaces
- drop `-kb` from flag names and carry units in help

#### 2. Q8 resolves more than the current spec text currently reflects

The Q8 answer narrows two additional naming points that should be synced into the spec before Role 2 work starts:

- `--refine-window-kb` should become `--growth-refine-halfwidth`
- `--refine-smooth-kb` likely also wants the halfwidth framing if the live parameter is in fact a smoothing halfwidth rather than a generic smoothing span: `--growth-refine-smooth-halfwidth`
- `--model-window-kb` should follow the same rule and prefer `--asym-tri-model-halfwidth` / `--growth-asym-tri-model-halfwidth`

This means the feedback file is now ahead of `PHASE14_SUPPLEMENTAL-SPEC.md` on at least one naming point, and possibly two.

#### 3. Additional rename candidates outside the current supplemental scope

These are real naming debts, but I would not fold them into the current Growth cleanup unless the user explicitly wants the supplemental widened:

- HMM `special` terminology is weaker than `background` terminology.
  - Current canonical mixed surfaces include synonyms such as `--hmm-expected-background-length` / `--hmm-expected-special-length`, `--hmm-background-idx` / `--hmm-special-idx`, `--hmm-init-background` / `--hmm-init-special`, `--hmm-leave-background-state` / `--hmm-leave-special-state`, and `--hmm-leave-amp-step` / `--hmm-leave-other`.
  - Judgment: keep compatibility, but de-emphasize the `special` / `other` spellings and treat `background` / `amp-step` as the clearly preferred public vocabulary.
- `--hmm-emodel` is too cryptic for a top-level CLI.
  - Recommended future rename candidate: `--hmm-emission-model`.
  - The current name reads like an internal shorthand, not a user-facing control.
- `--aps-rank-by` currently exposes only one allowed choice (`area`).
  - Judgment: the name itself is fine, but the flag currently behaves more like a reserved surface than a meaningful choice point. Consider hiding it until multiple real choices exist, or keep it only if an imminent expansion is planned.
- `--growth-window` and `--rms-window` deserve a later consistency review.
  - Their help currently says `fit window half-size`, which suggests another possible `halfwidth` naming cleanup in a future pass. I would not expand the current supplemental to include them unless the user wants a broader rename wave.

#### 4. Naming surfaces that are basically fine and should not be churned right now

These look adequately intuitive and aligned with the current Phase 14 contract:

- `--pipelines`
- `--norm-mode`
- `--growth-z-thresh`, `--rms-z-thresh`
- `--growth-halfwidths`, `--rms-halfwidths`
- `--dedup-dist`
- `--shape-score-threshold`, `--shape-score-strict-bic`
- `--rms-summit-policy`, `--rms-early-summit-stages`

Some of these still need help-string improvement, but not renaming.

### Help-string findings

#### 1. The main remaining repo-wide deficit is step/context clarity

John's note above is correct: help strings should consistently tell the user what output step or downstream analysis they affect. Right now that context is uneven.

The strongest existing pattern is the richer Growth supplemental wording that ties parameters to concrete outputs like `09-summit-refinement`, `10-timing`, or `07-signal-tracks`. That pattern should be treated as the model for the next pass across all parser groups, not just Growth.

High-value groups for step/context improvement:

- Timing flags in `onionskin.py`
- Overlap/dedup flags in `onionskin.py`
- HMM modeling and decoding flags in `onionskin.py`
- Standalone engine CLIs, especially the Growth engine

#### 2. Placeholder flags are currently the weakest user-facing help surfaces

These flags are structurally exposed but not meaningfully active yet:

- `--hmm-shape-score-threshold`
- `--hmm-shape-score-strict-bic`
- `--rms-shape-score-threshold`
- `--rms-shape-score-strict-bic`
- `--rms-bootstrap-origins`
- `--growth-bootstrap-origins`

Judgment:

- the current wording is honest, but still not very user-friendly
- either hide these from normal help until they are wired, or make the help more explicit that they are reserved parser surfaces with no current behavioral effect in the present release
- if they stay visible, the help should explain whether they are true no-ops, partial wiring hooks, or inheritance placeholders

#### 3. HMM help is information-rich but still too jargon-heavy in spots

The HMM section is detailed, which is good, but a few help surfaces still read more like internal notes than user-facing guidance:

- `--hmm-emodel` uses an opaque name and packs too much model jargon into a single short alias
- `--hmm-decode-path` explains `viterbi` vs `posterior`, but could do a better job saying when a user would actually choose one over the other
- `--hmm-training` similarly explains the algorithms but not the practical decision rule for this repo's data
- several per-flag notes still mention PuffStep inline; a section-level compatibility note would likely be cleaner than repeating `Port of PuffStep ...` inside many individual help strings

Judgment: the issue is not lack of detail, but weak prioritization of user intent over implementation provenance.

#### 4. Growth standalone engine help is noticeably behind the top-level CLI

`onionskin_core/engines/growth_model_engine.py` still has several flags with missing or extremely terse help compared with the top-level CLI standards. Examples include:

- `--trend-kb`
- `--smooth-kb`
- `--z-thresh`
- `--halfwidths-kb`
- `--peak-search-kb`
- `--window-kb`
- `--rng-seed`
- `--out-prefix`

Judgment: the standalone Growth engine is now correctly cross-linked to the top-level CLI, but its local help still feels like an internal engine entry point rather than a polished public parser.

#### 5. RMS standalone engine help is acceptable but still terse

`onionskin_core/engines/rcn_mean_shift_engine.py` is better than the Growth engine in raw concision, but still light on context:

- `Stage1 robust z threshold`
- `Stage1 halfwidths`
- `Stage2 peak search radius`
- `Stage2 fit window half-size`

This is understandable to someone who already knows the pipeline, but not especially intuitive to a new user. A small wording pass would help.

#### 6. Some main-CLI help strings are already strong and should be used as style anchors

These are good examples worth preserving as models for future rewrites:

- `--pipelines`
- `--timing-onset-quantile`
- `--aps-cluster-k`
- `--aps-feature`
- `--shape-score-strict-bic`

They are strong because they explain defaults, intended use, and tradeoffs rather than just restating the parameter name.

### Script-level audit notes

The user-facing script CLIs are mostly acceptable and do not need the same rename urgency as the main parser.

- `scripts/summit_inspector.py` is generally clear. Its main remaining improvement is more explicit step/output-context language, not renaming.
- `scripts/aps_cluster_experiments.py` is simple and readable. The flags are appropriately utilitarian for a wrapper script.
- `scripts/aps_cluster_report.py` is also structurally fine. Help is brief but adequate for its narrow purpose.

Judgment: these script CLIs should not drive the current supplemental scope.

### Recommended routing from this audit

#### Keep inside the current supplemental implementation wave

- the Growth rename set in 14-S1
- the Universal/growth peak-summary move in 14-S2
- the asymmetric triangle universalization in 14-S3
- sync the spec to the resolved Q8 answer before Role 2 begins

#### Do not silently widen 14-S1/14-S2/14-S3 yet

The broader audit found additional debt, but I recommend keeping it separate unless the user explicitly asks to widen the supplemental. The main extra work items are:

- HMM naming cleanup (`special`/`other` de-emphasis; possible `--hmm-emission-model` rename)
- placeholder-flag help cleanup or help-surface hiding
- repo-wide step-mention/help-context harmonization across Universal, HMM, RMS, and standalone engine parsers
- possible later `window` vs `halfwidth` consistency pass for other flags such as `--growth-window` / `--rms-window`

#### Strong recommendation for the next audit round

Role 1 should update `PHASE14_SUPPLEMENTAL-SPEC.md` before handing anything to Role 2, because the feedback file now contains two classes of information that the spec does not fully reflect yet:

1. resolved Q8 naming answers
2. the broader repo-wide CLI audit, which confirms that the current Growth supplemental is justified but should not silently absorb unrelated HMM/help cleanup without an explicit scoping decision

**Signed:** GitHub Copilot (GPT-5.4) — 2026-04-22 15:04 EDT


## John M. Urban Feedback on Copilot review
- This is not exhaustive. Just some comments.
- I do not agree that we should retire "special" terminology for HMM. It is a remnant synonym from PuffStep, which I believe we mention in the help text already. If not, mention it.
- We can add `--hmm-emission-model` but keep `--hmm-emodel` as a remnant PuffStep synonym.
- Do not hide `--aps-rank-by`. Expand its help text. Put a note in KNOWN_ISSUES that this likely was intended to have more options (perhaps: summit, width, shape).
- Yes, if half-size is synonymous with halfwidth, then (1) --growth-window and --rms-window should minimally be changed to --growth-halfwidth and --rms-halfwidth, and (2) the use of "half-size" should change to halfwidth to use consistent terminology. In addition to updating to "--growth-halfwidth and --rms-halfwidth", I also would not mind adding 1-2 more words to make those CLI flags more intuitive. I also noticed the help string uses the "Stage-2" terminology -- I believe we agreed elsewhere that "Stage-1" and "Stage-2" in this context should change to "First pass" and "Second pass" - or maybe "Pass-1" and "Pass-2" - please check and confirm. If so, it is time to change the terms in the help string. We should consider changing it internally in the code as well to be consistent.
- RE: "The main remaining repo-wide deficit is step/context clarity". I fully agree. Help strings should consistently tell the user what output step or downstream analysis they affect. The pattern Copilot described should be treated as the model for the next pass across all parser groups, not just Growth.
- RE: "Placeholder flags are currently the weakest user-facing help surfaces". Do not remove or hide these. It is okay to "make the help more explicit that they are reserved parser surfaces with no current behavioral effect in the present release" and to "explain whether they are true no-ops, partial wiring hooks, or inheritance placeholders".
- RE: "HMM help is information-rich but still too jargon-heavy in spots".
  - I disagree with --hmm-emodel being too full of stuff - I want it to be.
  - It would be okay for "--hmm-decode-path" help text to "do a better job saying when a user would actually choose one over the other". If I recall, the viterbi is almost always preferred at the scale the HMM is working on since it is much faster and gives accurate results.
  -  I somewhat agree that a section-level compatibility note would likely be cleaner than repeating "Port of PuffStep ..." inside many individual help strings, except where it is specific to the helpstring.
  - It is okay that --hmm-training overexplains algorithms. Do not simplify. It would be fine to add more user-facing decision context though, only if we have something useful to say there.
  - RE: "Growth standalone engine help is noticeably behind the top-level CLI" 
    - `onionskin_core/engines/growth_model_engine.py` is never expected to be used by anyone directly, only by onionskin.py. 
    - We can update it if we want, but maybe the real question is why do we still have user-facing CLI flags in the engines themselves if they are never expected to be directly used by us or a user? Do we have a need to use them directly in development? Or is it sufficient to access them through the onionskin CLI?
    - Am I making sense? Did I misunderstand the point made here?
  - RE: "RMS standalone engine help is acceptable but still terse"
    - Same here -- is this about help messages output by `onionskin_core/engines/rcn_mean_shift_engine.py` directly?
    - Again - we never expect it to be used directly anymore. This is vestigial stuff left over from when the engines were different scripts. Now, onionskin has consumed them, and we don't need them as separate scripts that can be directly accessed anymore -- unless someone who disagrees can persuade me of their point of view. 
  - RE: "Some main-CLI help strings are already strong and should be used as style anchors"
    - Let's take this into consideration.
  
  That is about all my feedback on Copilot's report. Feel free to ask questions.



---

## OPEN QUESTIONS ROUND 3

### Q9 — Final names for `--growth-window` and `--rms-window` (Priority 14-S4)

**Context:** Both flags have help text "Stage-2 fit window half-size in kb." The value IS a
halfwidth (search radius, not full span) — the Q7 halfwidth-naming rule applies. The user also
requested "1-2 more words to make those CLI flags more intuitive."

**Proposed candidates:**

| Old name | Candidate A | Candidate B | Candidate C |
|---|---|---|---|
| `--growth-window` | `--growth-scan-halfwidth` | `--growth-ms-halfwidth` | `--growth-fit-halfwidth` |
| `--rms-window` | `--rms-scan-halfwidth` | `--rms-ms-halfwidth` | `--rms-fit-halfwidth` |

Recommendation: `--growth-scan-halfwidth` / `--rms-scan-halfwidth` — "scan" captures the
pass-2 window role without tying the name to a specific algorithm ("ms" = mean-shift is
implementation-internal; "fit" conflicts with `--growth-fit-method`).

**Question for user:** Which names do you prefer? Or propose your own.

**Answer from John M. Urban:**
- Yes, I love the "scan" terminology here: `--growth-scan-halfwidth` / `--rms-scan-halfwidth`
  - Should the CLI flag mention "pass-2"? is there a corresponding flag associated with "pass-1"?
- Note that "ms" is somewhat overloaded as we have used it in the past as shorthand for "multi stage" as well. This does not affect this decision, but perhaps should be noted somewhere.


---

### Q10 — Stage-2 → Pass-2 terminology (Priority 14-S4)

**Context:** The help strings for `--growth-window` and `--rms-window` currently say
"Stage-2 fit window half-size." The user believes "Stage-2 → Pass-2 (or Second pass)" was
agreed somewhere but no written record was found. This affects help strings for `--growth-window`,
`--rms-window`, and potentially any other help text that uses "Stage-1"/"Stage-2" to mean
the first/second pass of the detection algorithm (not biological amplification stages).

**Options:**
- `Pass 2` / `Pass 1` (space, not hyphen — clearest parallel structure)
- `Second pass` / `First pass` (more verbose but unambiguous)
- Keep `Stage-2` / `Stage-1` (if the agreement to change was not actually made)

Recommendation: `Pass 2` / `Pass 1`.

**Question for user:** What should "Stage-2" become in these help strings? And was this
actually agreed previously?

**Answer from John M. Urban:**
- For natural language, we should use "First pass (pass 1)" and "Second pass (pass 2)" the first time the terms are encountered in a help string. Subsequent times can just use what is in the parentheses: "pass 1" and "pass 2".

---

### Q11 — Standalone engine CLI: keep, remove, or deprecate? (Priority 14-S10)

**Context:** `onionskin_core/engines/growth_model_engine.py` and
`onionskin_core/engines/rcn_mean_shift_engine.py` each have their own full `argparse` parser
and `-h` output (standalone CLI entry points). These were created when the engines were
separate scripts. Since `onionskin.py` has consumed them, direct engine invocation is no longer
expected by users or in normal development workflows.

**Options:**

| Option | What it means |
|---|---|
| **Keep + help-string parity pass** | Maintain both engine CLIs; include them in the 14-S8 step-mention pass; ongoing sync burden |
| **Remove** | Delete the `argparse` parsers and `main()` entry points from the engine files; engines become pure Python APIs; `onionskin.py` is the sole CLI entry point; simplifies long-term maintenance |
| **Deprecate** | Add a deprecation warning to engine `main()` output; plan removal in a future phase |

Recommendation: **Remove.** Vestigial; never expected to be used directly; removing eliminates
an ongoing maintenance burden with no user-facing cost.

**Question for user:** Keep, remove, or deprecate the standalone engine CLIs?

**Answer from John M. Urban:**
- Remove
- Let's move forward with simplifying onionskin here since all pipelines are accessed from onionskin.py. As stated, I do not foresee directly using the engines, especially if they are not maintained in parallel, which has proven burdensome.


---


## RECON AUDIT — Intended-but-missed work for Phase 14 + Phase 14 Supplemental

**Author:** Claude Code (claude-opus-4-7) — 2026-04-22 ~20:00 EDT
**Scope of this audit:** Read all Phase 14 planning artifacts (archived BRAINSTORM, FEEDBACK,
SPEC) and Phase 14 Supplemental artifacts (current FEEDBACK, SPEC) against `KNOWN_ISSUES.md`,
`BRAINSTORM.md`, and the live parser in `onionskin.py`. Identify items that were intended
for Phase 14 or its supplemental phase (by the letter or spirit of either), but that were
either never promoted into a SPEC priority, were silently narrowed out of scope, or were
partially covered and need expansion.

**Method:** Cross-referenced every Phase 14 BRAINSTORM priority, every FEEDBACK instruction,
every Phase 14 SPEC priority, and every Phase 14 Supplemental priority. Audited the
`## More Feedback from John M. Urban` and `## John M. Urban Feedback on Copilot review`
sections line-by-line against the current `PHASE14_SUPPLEMENTAL-SPEC.md` to find instructions
that were only partially captured. Spot-checked the live parser in `onionskin.py` for terms
("Stage-1", "Stage-2", Timing group structure, PuffStep notes) referenced in the feedback.

---

### Finding 1 — Phase 14 proper: coverage is excellent

Phase 14 proper (v0.14.00 → v0.14.46) closed with all 13 brainstorm priorities reflected in
the archived `PHASE14_SPEC.md` (11 implementation priorities). Cross-referenced against the
"self-audit against brainstorm" table in the archived SPEC: no brainstorm priority was dropped
outright. The scope-narrowing problem the user flagged is concentrated in **Phase 14
Supplemental**, not in Phase 14 proper.

**One small leftover from Phase 14 proper** (already flagged elsewhere, worth re-surfacing):
- v0.14.42 closeout noted that `CLAUDE.md` line 28 still references `PHASE11_SPEC.md` as the
  active phase plan. This stale-pointer defect was logged as a "supplemental finding (does not
  block closeout)" and has been carried in TASK.md as a parked task rather than a priority.
  Per `AGENT_CONVENTIONS.md`, all four agent files must stay in sync (CLAUDE.md, AGENTS.md,
  GEMINI.md, .github/copilot-instructions.md).

---

### Finding 2 — Phase 14 Supplemental: intended items NOT captured as priorities

Reading `## More Feedback from John M. Urban` and `## John M. Urban Feedback on Copilot
review` against the current `PHASE14_SUPPLEMENTAL-SPEC.md`, several user instructions were
only partially promoted. Each is listed with its source quote and the gap.

#### 2.1 — Downstream-analyses Universal direction beyond triangle model flags

**Source (user):** "Analyses like timing, progression, growth asymmetry, APS and so on should
all be considered goals of all pipelines. We need to think of each pipeline as a means to
identify amplicons with possibly unique ways of learning about their summits as well. But once
we have amplicons, then all downstream aims are shared by all pipelines: summit refinement,
timing, elongation asymmetry, and so on."

**Current coverage:**
- 14-S3 universalized asymmetric triangle model flags. ✓
- 14-S12 surveys remaining *growth-group* flags for RCN-profile Universal candidates. ✓

**Gap:** The user's direction is broader than growth-flag RCN-profile candidates. It
explicitly extends to the **Timing parser group**, which is currently Universal-looking in
structure but bare (no `--growth-*` overrides, no multi-pipeline step mentions). Timing flags
like `--onset-rcn-threshold`, `--onset-span-rcn-threshold`, `--onset-method`,
`--onset-z-threshold`, `--timing-onset-quantile`, `--timing-exclude-loci`, `--near-gap-bp` all
currently apply to the growth pipeline only but are not prefixed. They should be audited for
either:
- explicit Universal framing (help text and step mentions spanning all pipelines), or
- growth-prefix rename if they truly are growth-only, or
- Universal + `--growth-*` override pattern if they are a shared concept.

**Adjacent:** The `--onset-*` vs `--timing-*` mix-prefix within the Timing group is itself a
Phase 14 naming-consistency concern that was not raised earlier.

**Recommended priority:** **14-S13 — Timing parser group audit and Universal promotion
decision.**

#### 2.2 — Internal-code "Stage-1"/"Stage-2" terminology consistency

**Source (user on Q10/Copilot review):** "In addition to updating to `--growth-halfwidth and
--rms-halfwidth`, I also would not mind adding 1-2 more words to make those CLI flags more
intuitive. I also noticed the help string uses the 'Stage-2' terminology — I believe we agreed
elsewhere that 'Stage-1' and 'Stage-2' in this context should change to 'First pass' and
'Second pass' — or maybe 'Pass-1' and 'Pass-2' — please check and confirm. **If so, it is time
to change the terms in the help string. We should consider changing it internally in the code
as well to be consistent.**"

**Current coverage:**
- 14-S4 addresses the help-string terminology change ("Second pass (pass 2)" on first use,
  "pass 2" subsequently) for the scan-halfwidth flags. ✓

**Gap:** 14-S4 explicitly scopes to help strings only. The user asked to "consider changing it
internally in the code as well." An internal-code audit for "Stage-1"/"Stage-2"/`stage_1`/
`stage_2` terminology that refers to **detection passes** (as distinct from biological
stage 1 / stage 2 of amplification — which IS a real biological concept that must NOT be
renamed) has not been planned.

**Live evidence:** `onionskin.py` lines 1772, 1779, 1864, 1871 use "Stage-2" in help strings
for `--rms-peak-search`, `--rms-window`, `--growth-peak-search`, `--growth-window` — those
match the 14-S4 scope. But the growth/RMS engine files and `onionskin_core/` may have internal
docstrings, variable names, or comments using "Stage-1"/"Stage-2" for the detection passes.
Those are NOT in scope of 14-S4 as currently written.

**Critical distinction:** `onionskin.py:2895`, 3035, 3058 use "stage-1" in the context of
*biological posterior stage 1 samples* — a completely different concept. An audit MUST
distinguish "detection pass 1/2" from "biological stage 1/2"; only the former gets renamed.

**Recommended priority:** **14-S14 — Internal-code audit for "Stage-1"/"Stage-2" detection-pass
terminology.**

#### 2.3 — "Asymmetric Triangle Model" group name — consider alternative

**Source (user):** "Add CLI flags to a universal section — perhaps the timing section or a
fork asymmetry section (?)"

**Current coverage:**
- 14-S3 created a new Universal "Asymmetric Triangle Model" parser group with 3 flags.

**Gap:** The user raised (as a question) whether a "fork asymmetry" section would be
appropriate, or whether the triangle flags should live in the Timing group instead. No
explicit scoping decision was recorded. The current 14-S3 name is correct for the specific
flags it contains, but it may not scale: if more "fork asymmetry" flags are promoted in future
(e.g., elongation asymmetry metrics the user mentioned), there will be a question of whether
to add them to this same group or create another.

**Recommended action:** Not a priority by itself, but **open question for user** — confirm
the Asymmetric Triangle Model group name is the intended final name, or consider renaming it
to something like "Fork Asymmetry" or merging into Timing. User decision recorded here.

#### 2.4 — PuffStep compatibility note coverage in HMM help strings

**Source (user on Copilot review):** "I do not agree that we should retire 'special'
terminology for HMM. It is a remnant synonym from PuffStep, **which I believe we mention in
the help text already. If not, mention it.**"

**Current coverage:**
- 14-S9 includes a section-level PuffStep compatibility note in the HMM group description.
- The feedback says "remove redundant 'Port of PuffStep' text from individual flag help
  strings where the per-flag note adds nothing beyond the group-level note; keep it where the
  flag is PuffStep-specific."

**Gap:** Spot-checked `onionskin.py:1286–1581`. Most HMM PuffStep synonyms DO already mention
PuffStep in the help text (e.g., `--hmm-special-idx` says "Also accepted as `--hmm-special-idx`
(PuffStep synonym)."). However, a **systematic audit** to confirm EVERY synonym carries this
note is not explicitly required by 14-S9. 14-S9's focus is group-level consolidation and
`--hmm-decode-path`/`--hmm-training`. The per-flag preservation-audit is implicit.

**Recommended action:** Fold into 14-S9 explicitly, OR create **14-S15 — HMM PuffStep-synonym
help-string audit.** Keep small; the finding is one line per flag.

#### 2.5 — Summit refinement as future Universal candidate

**Source (user):** "once we have amplicons, then all downstream aims are shared by all
pipelines: **summit refinement**, timing, elongation asymmetry, and so on."

**Current coverage:**
- 14-S1 has a note: "Summit refinement is a shared downstream goal and may be promoted to
  Universal in a future phase."
- 14-S12 surveys growth flags for RCN-profile Universal candidates — lists
  `--growth-refine-halfwidth` and `--growth-refine-smooth` as survey targets.

**Gap:** "Future phase" framing is narrowing. The user did not scope summit refinement out
— he explicitly included it. 14-S12's Universal survey covers this implicitly. If 14-S12
surfaces "refine" flags as strong Universal candidates, the implementation pass should
include them now rather than deferring. **This is a scope-authority risk**: "may be promoted
in a future phase" is agent language that can become an unapproved defer.

**Recommended action:** Update 14-S12's deliverable framing: its output table must produce
concrete promote-now / promote-later / growth-only recommendations, and the user decides
— not the agent's default-to-defer instinct.

#### 2.6 — Universal flag help must list pipeline-specific step mentions

**Source (user):** "Step mentions should be true for all flags, not just growth pipeline
flags. HMM and RMS should do this too. **Flags in universal sections would be pipeline step
mentions.** An example for an APS flag might be: 'hmm: 14-aps; growth: 13-aps; rms: 12-aps.'
Or that might be for the section header lines if it applies to all flags in the section."

**Current coverage:**
- 14-S8 covers this as a convention: "For Universal flags: list all pipelines, e.g.,
  `[hmm: 14-aps; growth: 13-aps; rms: 12-aps]`." ✓

**Gap:** Verify application. The existing Shape scoring universal flags
(`--shape-score-threshold`, `--shape-score-strict-bic`) already in the Universal/Shape-scoring
groups — their help strings should carry multi-pipeline step mentions. 14-S8 says the quality
bar applies to ALL groups, which covers this, but the specific step-mention mapping for
Shape-scoring, Universal-general, and Overlap/Dedup groups should be explicit in the 14-S8
implementation plan since these flags affect steps across all three pipelines.

**Recommended action:** Expand 14-S8's target-groups list to call out the **multi-pipeline
step-mention requirement** explicitly for Universal, Shape scoring, Overlap/dedup, APS, and
Asymmetric Triangle Model groups. (The growth-pipeline-only groups are easier — they carry
only a growth step.)

#### 2.7 — README.md + PIPELINE_SPEC.md + ONIONSKIN_FULL_HANDOFF.md doc sweep after supplemental

**Source (PDS-v1.md rule):** "Identify all code that touches or is touched by changes we are
proposing, and that will need to be updated in parallel.… Include updates needed for README.md
and similar documentation files."

**Current coverage:**
- Each of 14-S1, 14-S2, 14-S3, 14-S4, 14-S10 mentions PIPELINE_SPEC.md or
  ONIONSKIN_FULL_HANDOFF.md in its "documentation" sub-section.
- No unified post-supplemental doc audit is scoped.

**Gap:** Phase 14 proper had Priority 14.10 as a unified doc migration pass; Phase 14
Supplemental has no equivalent. After 14-S1 through 14-S12 are implemented, a Role 1
documentation audit should sweep:
- `multi-agent/full_instructions/PIPELINE_SPEC.md`
- `multi-agent/full_instructions/ONIONSKIN_FULL_HANDOFF.md`
- `README.md` — specifically any user-facing flag examples and any "Universal section"
  overview text (the Asymmetric Triangle Model group is new; peak-summary flags moved to
  Universal).
- `multi-agent/project_context/DECISIONS.md` — the "ms" ambiguity note from 14-S4 should be
  added here.

**Recommended priority:** **14-S16 — Phase 14 Supplemental documentation sweep.**

#### 2.8 — AGENT_CONVENTIONS.md CLI conventions section verification

**Source:** AGENT_CONVENTIONS.md section `## CLI Flag Naming and Structure Conventions` was
added in Phase 14.11 based on the live parser at that time.

**Gap:** After Phase 14 Supplemental lands (new Universal group, new halfwidth naming rule,
peak-summary Universal promotion pattern), that conventions section may drift from the live
parser. The user's `halfwidth`-vs-`window` rule (Q7) and the `--<group>-<concept>` Universal
pattern for non-pipeline-universal groups (Asymmetric Triangle Model) are not yet codified.

**Recommended action:** Fold into 14-S16 or create **14-S17 — Update AGENT_CONVENTIONS.md CLI
conventions section to reflect live parser after supplemental.** Small scope.

#### 2.9 — `make test` runtime (KNOWN_ISSUES 2026-04-21:2) and supplemental test additions

**Source:** KNOWN_ISSUES.md [ISSUE:2026-04-21:2] — `make test` runtime (~3:20 at time of
entry; currently ~60s since Phase 14 added pytest-xdist; but new help-regression tests and
deprecated-flag parameterization from 14-S4/S7 will grow the suite).

**Gap:** Phase 14 Supplemental is adding test surface area (help regressions for new parser
groups, new deprecated-flag entries). Not a blocker, but the open KNOWN_ISSUES item becomes
more actionable as the test suite grows. Not strictly in Phase 14 scope.

**Recommended action:** Leave in KNOWN_ISSUES. Not a supplemental priority.

#### 2.10 — Stale-attribute grep after all supplemental work

**Source (PDS-v1.md):** The "identify all code that touches or is touched by" requirement plus
the Phase 14.2 pattern of grep-based verification.

**Gap:** 14-S1 has a grep command for its own attribute renames. 14-S2 and 14-S3 have grep
commands. 14-S4 has a grep command. But no unified "after all supplemental priorities close,
sweep the entire repo for any lingering stale attribute names and flag strings" audit is
scoped.

**Recommended action:** Include as part of 14-S16 closeout validation.

---

### Finding 3 — Phase 14 proper items that did NOT appear to be narrowed

Items the user mentioned in Phase 14 brainstorm or feedback that ARE adequately covered and
do NOT need reopening:

- CLI restructuring (Universal, Shape scoring, Advanced dissolution): ✓ 14.1
- Flag renames (single/multi): ✓ 14.2
- Deprecation catches: ✓ 14.4
- `--norm-mode` behavior: ✓ 14.3
- Section header renames: ✓ 14.1
- Test updates: ✓ 14.7
- Scripts cleanup: ✓ 14.8
- `--pipelines rms` synonym: ✓ 14.5
- Developer-helper naming (Makefile target, agent files): ✓ 14.9
- Live documentation migration (PIPELINE_SPEC, HANDOFF, README, DECISIONS): ✓ 14.10
- AGENT_CONVENTIONS.md CLI rule: ✓ 14.11
- Engine CLI crosswalk notes: ✓ 14.6 (though 14-S10 now removes the engine CLIs entirely)

---

### Finding 4 — Priority consolidation recommendations

I propose adding the following priorities to `PHASE14_SUPPLEMENTAL-SPEC.md`:

| New priority | Source | Effort |
|---|---|---|
| **14-S13 — Timing parser group audit + Universal promotion decision** | Finding 2.1 | Medium (survey + decisions + implementation if promotions approved) |
| **14-S14 — Internal-code "Stage-1"/"Stage-2" detection-pass terminology audit** | Finding 2.2 | Small–medium (audit scope; implementation size depends on findings) |
| **14-S15 — HMM PuffStep-synonym help-string preservation audit** | Finding 2.4 | Small |
| **14-S16 — Phase 14 Supplemental closeout documentation sweep** | Findings 2.7, 2.8, 2.10 | Medium |

I propose **expanding** these existing priorities:
- **14-S8** — add explicit Universal-flag multi-pipeline step-mention requirement (Finding 2.6).
- **14-S9** — incorporate per-flag PuffStep-synonym preservation (Finding 2.4) if not split as 14-S15.
- **14-S12** — reframe the deliverable to produce concrete promote-now recommendations, not defer-by-default (Finding 2.5).

I propose leaving these items outside the supplemental scope:
- KNOWN_ISSUES [ISSUE:2026-04-21:2] `make test` runtime (Finding 2.9) — not a Phase 14 concern.
- Asymmetric Triangle Model group rename question (Finding 2.3) — surface as open question; user decides.

---

## OPEN QUESTIONS ROUND 4 — Recon-audit follow-ups

### Feedback from John while reading
- Finding 1:
        - Yes update CLAUDE.md and all agent files
- Finding 2.1:
        -Flags in timing could have better help strings, including mentioning pipelines and steps.
        - These are intended to be universal with no prefix
        - At the moment, I do not have a use-case in mind where different pipelines would benefit from their own --<prefix>-parameters
        - For now - just keep all these in timimg ; if they apply only to growth right now, then explicitly say that in the string. It calls it out to us as much as users.
                - So:
                - yes: explicit Universal framing (help text and step mentions spanning all pipelines)
                - no: not meant to be growth only
                - no: not meant to have --growth* versions at the moment
- Finding 2.2:
        - Move this audit idea to multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md
                - We will have to rename that to simply: multi-agent/tracking/INTENDED-BUT-MISSED.md
- Finding 2.3
        - Asymmetric Triangle Model is fine for now
        - We can evolve on this
        - We don't have to track this concern
- Finding 2.4
        - We should keep notes like "Also accepted as --hmm-special-idx (PuffStep synonym)."
        - I consider that unique enough to the flag to keep in place -- not moved to the top of the section.
        - Let's just leave the help text in HMM alone for now. If anything, a note can be added to the top. But let's not remove anything.
- Finding 2.5
        - I need to come back to this after reading 14-S1 and 14-S12
        - the table idea is fine
- Finding 2.6
        - It would be okay to strengthen the wording here so all flags are given attention.
        - Yes, overall, we are trying to add step and pipeline step mentions to all flags.
- Finding 2.7
        - Yes, Phase 14 Supplemental documentation sweep is good.
- Finding 2.8
        - This seems fine
- Finding 2.9
        - Okay to leave in KNOWN_ISSUES
- Finding 2.10
        - Okay to Include as part of 14-S16 closeout validation.
- Other
  - --near-gap-bp is not about replication timing; it is universal (or intended to be) so, since it does not have a home, it should move to regular universal for now
  - --posterior should also be added above to the generic universal section
      - even though it seems natural to put it in APS, it is actually a more generic question to answer: does the user want a posterior portion of the run (regardless of how we do it)


### Q12 — Timing parser group: audit for Universal promotion?

**Context:** Timing flags (`--onset-rcn-threshold`, `--onset-span-rcn-threshold`,
`--onset-method`, `--onset-z-threshold`, `--timing-onset-quantile`, `--timing-exclude-loci`,
`--near-gap-bp`) currently live in the Timing parser group and apply only to the growth
pipeline. User feedback established timing as a shared downstream goal across all pipelines.

**Options:**
1. **Leave as-is** — Timing is Universal-shaped but growth-only-active; help strings
   already advertise the intent.
2. **Add `--growth-*` prefixes** to the Timing flags since they are currently growth-only,
   and rely on a future phase to re-universalize when RMS/HMM timing lands.
3. **Full Universal-promotion pass** — rewrite Timing help strings with multi-pipeline step
   mentions (`[growth: 10-timing; rms: planned; hmm: planned]`); add `--growth-onset-*`
   override flags per the 14-S2/14-S3 pattern; codify the plan.
4. **Mixed** — promote the shared concepts (onset thresholds, exclusion regions) to Universal
   with growth overrides; leave narrowly-growth concepts (`--near-gap-bp`?) as growth-only.

**Question:** Which approach do you prefer for Priority **14-S13**? Or should this be
deferred to a post-supplemental phase?

**Secondary question:** Also resolve the `--onset-*` vs `--timing-*` prefix inconsistency
within the Timing group (some flags are `--onset-*` bare, some are `--timing-*`).

**Answer from John M. Urban:**
- Leave as-is
- Do not add `--growth-*` stuff at this time since I do not currently suspect a need for different values for different pipelines
- Yes write step mentions in 'pipeline step' format discussed elsewhere
- Defer adding `--timing-*` prefixes for now. Can be a KNOWN_ISSUE.

---

### Q13 — Internal-code "Stage-1"/"Stage-2" detection-pass terminology audit scope

**Context:** 14-S4 addresses help strings only. The user asked to "consider changing it
internally in the code as well to be consistent." The concept is the detection pass
(first pass = coarse scan; second pass = refined fit), NOT the biological amplification
stages (stage 1 of early S-phase, stage 2 of mid-S, etc. — these MUST stay as-is).

**Options:**
1. **Full audit + rename** — audit `onionskin.py`, `onionskin_core/engines/*.py`,
   `onionskin_core/*.py`, and tests for "Stage-1"/"Stage-2"/`stage_1`/`stage_2`/
   "stage-1"/"stage-2" terminology referring to detection passes. Rename all to
   "pass 1"/"pass 2"/`pass_1`/`pass_2`. Preserve biological stage-1/stage-2 usages.
2. **Audit-only deliverable** — produce a table showing where each "Stage-{1,2}" token lives
   and classify each as "detection pass" vs "biological stage". User decides which
   subset to rename.
3. **Leave internal code as-is** — help strings are the user-visible surface; internal
   naming drift is low-cost.
4. **Targeted rename** — only rename in the immediate surface of the scan-halfwidth flags
   (e.g., any docstring for `_build_detect_extra`, `_refine_kwargs`). Leave deeper internal
   code alone.

**Question:** Which approach do you prefer for Priority **14-S14**?

**Answer from John M. Urban:**
- Let's leave this at audit-only for now.
- Put the audit findings and output in KNOWN_ISSUES for later, or in its own file under `multi-agent/tracking/`

---

### Q14 — Parser group name: "Asymmetric Triangle Model" final or change?

**Context:** 14-S3 created a new Universal parser group titled "Asymmetric Triangle Model"
to hold `--asym-tri-model-halfwidth`, `--asym-tri-model-smooth`, and
`--asym-tri-model-halfwidth-grid`. User previously mused: "perhaps the timing section or a
fork asymmetry section (?)". No final decision was recorded.

**Options:**
1. **Keep "Asymmetric Triangle Model"** — accurate; scales awkwardly if more fork-asymmetry
   flags arrive later.
2. **Rename to "Fork Asymmetry"** — broader; room for future flags (e.g., elongation
   asymmetry metrics); less literal about the underlying math.
3. **Merge into Timing group** — triangle model is "used for fork travel rate and direction",
   which is timing-adjacent. Avoids a new group; may cause group to sprawl.
4. **Rename to "Growth/Fork Asymmetry"** or similar — explicit about current scope and planned
   future expansion.

**Question:** Which do you prefer? (This decides whether 14-S3 needs a group-rename pass.)

**Answer from John M. Urban:**
- Keep as "Asymmetric Triangle Model" for now.
- I recognize there may be a future need to change this, but we can do that when it arises.

---

### Q15 — HMM PuffStep-synonym help-string audit: separate priority or fold into 14-S9?

**Context:** 14-S9 focuses on `--hmm-decode-path` practical guidance, `--hmm-training`
decision context, and group-level PuffStep consolidation. A systematic audit confirming every
per-flag PuffStep synonym preserves its PuffStep mention in help text is NOT explicitly
scoped as a 14-S9 deliverable.

**Options:**
1. **Fold into 14-S9** — add a sub-deliverable: "audit each HMM flag with a PuffStep synonym;
   ensure the synonym name and its 'PuffStep synonym' label are both present in the help
   text."
2. **Separate priority 14-S15** — keep 14-S9 narrow; let 14-S15 be a dedicated HMM help audit.
3. **Skip** — spot-check shows most synonyms already have the mention; rely on normal 14-S8
   help-quality pass to catch gaps.

**Question:** Fold in, separate, or skip?

**Answer from John M. Urban:**
- Let's leave the help text as is with regard to PuffStep mentions for now.
- They can be audited to see if changes are needed.
- The audit findings and output can be added to `multi-agent/plans/next/PHASE15_BRAINSTORM.md` 
  - This will be for updating the HMM pipeline

---

### Q16 — 14-S12 Universal survey: promote-now bias or defer-bias?

**Context:** 14-S12 currently produces a survey table of growth flags that might be Universal
candidates. The spec uses hedged language ("promotion decision"). Scope-authority risk:
defer-by-default could narrow the supplemental again.

**Options:**
1. **Promote-now bias** — survey must classify each candidate as "promote now" (add to
   supplemental scope immediately), "promote next supplemental phase", or "stay growth-only".
   User decides which; any "promote now" gets added as its own mini-priority.
2. **Produce survey only; defer all decisions to user** — current behavior, low scope risk.
3. **Survey + pre-implement the clear wins** — agent implements any survey items that are
   obvious Universal candidates (e.g., summit refinement flags) and flags borderline cases.

**Question:** Which approach? This directly tests the scope-authority rule: option 1 or 3
requires user decisions; option 2 keeps agent-defer-bias.

**Answer from John M. Urban:**
- Produce the survey results - append to the end of this file, and tell me where it is for final decisions

---

### Q17 — Phase 14 Supplemental closeout doc sweep (14-S16) scope

**Context:** Phase 14 proper had Priority 14.10 as a unified documentation migration pass.
Phase 14 Supplemental currently has no equivalent; each supplemental priority handles its own
docs, but no final unified sweep is scoped.

**Proposed 14-S16 scope:**
- PIPELINE_SPEC.md — audit for supplemental-era stale flag names; update narrative/tables.
- ONIONSKIN_FULL_HANDOFF.md — audit for supplemental-era stale references.
- README.md — update if user-facing (peak-summary Universal promotion, Asymmetric Triangle
  Model group).
- AGENT_CONVENTIONS.md CLI conventions section — verify still accurate after supplemental;
  add halfwidth-vs-window rule; codify Universal groups beyond the pipeline-specific pattern
  (Asymmetric Triangle Model; Shape scoring).
- DECISIONS.md — add the "ms" ambiguity note from 14-S4; add any other supplemental design
  decisions worth preserving.
- Final stale-attribute grep sweep across the entire repo.

**Question:** Accept proposed scope for **14-S16**? Anything to add or remove? Should the
AGENT_CONVENTIONS.md update be split into its own priority (**14-S17**)?

**Answer from John M. Urban:**
- Accept

---

### Q18 — CLAUDE.md stale `PHASE11_SPEC.md` pointer: promote to priority or keep as task?

**Context:** Currently tracked as a task-level item in TASK.md and HANDOFF.md. Per
`AGENT_CONVENTIONS.md`, all four agent files (CLAUDE.md, AGENTS.md, GEMINI.md,
.github/copilot-instructions.md) must be kept in sync. The fix is small — 4 one-line edits —
but the cross-file-sync requirement means it is audit-worthy.

**Options:**
1. **Keep as task** — small enough that it does not need a priority wrapper; handle at
   archive time.
2. **Promote to 14-S18** — explicit priority with an audit+implement+re-audit loop (same
   pattern as other supplemental priorities).
3. **Fold into 14-S16** — handle as part of the closeout doc sweep.

**Question:** Which? If promoted, 14-S18 would also include a broader audit: "are any other
agent files drifting from each other?"

**Answer from John M. Urban:**
- This does not need a priority. It is just something that can be done.
- A broader agent file audit for drift can be also be done without making it a priority.
- Just report the results to chat screen about drift, ask user if they want it fixed.

---

**Signed:** Claude Code (claude-opus-4-7) — 2026-04-22 ~20:00 EDT


---


## RECON AUDIT — SECOND PASS (after user pushback)

**Author:** Claude Code (claude-opus-4-7) — 2026-04-22 ~20:40 EDT

**User pushback:** "Ignore the SPEC. Think of the SPECs as the products we know are too
narrow. Use PHASE14 BRAINSTORM and FEEDBACK as where missing material might be. Also use
KNOWN_ISSUES.md and BRAINSTORM.md as where missing issues might be."

My first pass leaned too heavily on the archived SPEC's own self-audit table, which is
exactly the narrowing mechanism the user flagged. This second pass re-reads
`archived/20260421-PHASE14_BRAINSTORM.md` and `archived/20260421-PHASE14_FEEDBACK.md`
line-by-line for user-voice content, plus `KNOWN_ISSUES.md` and `BRAINSTORM.md` for
Phase 14-relevant items, and compares directly against the live code (not the SPEC).

---

### Finding 5 — User-authored --pipelines help text was drastically shortened

**Source (user, PHASE14_FEEDBACK line 97–101):**

> "Update to say: Available: hmm, growth, rms, all. Any one or more can be used with a
> comma-separated list. **all = hmm,growth,rms.** hmm = hidden markov model. growth =
> multiple stage growth modeling. rms = RCN mean shift. RCN = relative copy number. HMM
> and RMS can handle a single file, single stage with multiple replicates, a single stage
> versus a reference stage (each with 1 or more replicates), and multiple stages with or
> without a reference stage (--norm-mode set to chrom-median or ref-stage, respectively).
> The growth modeling pipeline is designed for 3 or more stages without a reference stage
> (--norm-mode chrom-median) or 4 or more stages when one is a reference stage (--norm-mode
> ref-stage). To use different normalization modes across pipelines, see their
> pipeline-specific --norm-mode flags."

**Live code (`onionskin.py:782–786`):**

```
"Comma-separated list of pipelines to run, or 'all'. "
"Available: growth, rms (or rcn-mean-shift), hmm. "
"Examples: --pipelines growth,rms  --pipelines hmm  --pipelines all. "
"Default: all (growth + rcn-mean-shift). "
"HMM is opt-in: include 'hmm' explicitly to run the HMM engine."
```

**Gaps vs user text (three separate defects):**

1. **Verbose content removed.** The user wrote ~110 words of educational help content
   explaining pipeline applicability, replicate/stage requirements, and normalization
   interactions. The implementation reduced this to ~40 words. The user's text contained
   per-pipeline descriptions (hmm/growth/rms definitions, RCN = relative copy number) and
   normalization-contract guidance. All removed.
2. **Factual contradiction with live runtime.** The user wrote "all = hmm,growth,rms." The
   help text says "Default: all (growth + rcn-mean-shift). HMM is opt-in." But
   `onionskin.py:_resolve_requested_effective_pipelines()` returns `{"growth",
   "rcn-mean-shift", "hmm"}` when `"all"` is requested. So `--pipelines all` DOES run HMM
   at runtime. The help text is wrong.
3. **User's intent: `all` means `all`.** The user explicitly wrote "all = hmm,growth,rms"
   and the spec/code runtime matches that intent. Only the help text and the "Default: all
   (growth + rcn-mean-shift). HMM is opt-in" sentence are inconsistent. Either the help
   text must be corrected to match the runtime, or the runtime must be corrected to match
   the help text.

**This is a genuine narrowing.** The user wrote a verbose spec; the agent shipped a short
contradictory version.

**Recommended priority:** **14-S19 — Restore user's verbose `--pipelines` help text; resolve
the `all` semantics contradiction.**

---

### Finding 6 — "Same goes for timing and APS sections" partially addressed

**Source (user, PHASE14_FEEDBACK line 62):**

> "Anything in the overlap section should be considered as potential to be universal even
> if it applies to only one pipeline right now. **Same goes for timing and APS sections.**"

**Coverage:**
- Overlap: Universal-positioned via new structure in 14.1. ✓
- Timing: NOT audited in Phase 14 proper; partially captured in my first-pass 14-S13.
- APS: NOT audited for Universal promotion either in Phase 14 proper or in my first pass.

**APS audit gap:** APS flags (`--aps-rank-by`, `--aps-cluster-k`, `--aps-feature`,
`--aps-singleton-guard`, `--aps-area-excess-floor` future, etc.) are conceptually
pipeline-agnostic (APS is a pipeline-agnostic metric). But the APS parser group stands
apart from Universal. The user asked for APS to be "considered as potential to be
universal." This was not done.

**Recommended priority:** **14-S20 — APS parser group Universal-promotion audit** (parallel
to 14-S13 for Timing).

---

### Finding 7 — User's literal question on --dedup-dist never answered

**Source (user, PHASE14_FEEDBACK line 62):**

> "I imagine this applies to both RMS and Growth pipelines, each of which uses mean shift
> for calls, correct?"

**Gap:** This is a literal user question about whether `--dedup-dist` should apply
equally to RMS and Growth. No agent response directly confirmed/denied. The flag was moved
to the Overlap group, which is positioned as Universal-like, but the semantic question —
does `--dedup-dist` actually dedup both RMS and Growth calls at runtime? — was never
verified against the code.

**Recommended action:** Small verification task. Confirm `--dedup-dist` semantics in the
live code path; if it's RMS-only today, surface that fact in help text and decide whether
Growth should also respect it. Fold into 14-S20 or 14-S8.

---

### Finding 8 — BRAINSTORM.md `--hmm-0-based-statepath` flag

**Source (`BRAINSTORM.md` line 1918, 2026-04-18):**

> "**--hmm-0-based-statepath future CLI flag** … Status: Future option. Breaking change if
> made the default; safest as an opt-in flag. … With 0-based states, the expected
> amplification level at a bin is directly 2^state (e.g., state 0 = 2^0 = 1×, state 2 = 4×
> amplification). This makes the mathematical relationship between state values and copy
> number explicit."

**Gap:** A fully-designed opt-in CLI flag sitting in BRAINSTORM. It fits the Phase 14
spirit (new user-facing CLI flag, HMM group, pipeline-specific `--hmm-*` prefix, explicit
opt-in default, clear convention). It was never formally promoted or deferred by the user.

**Recommended priority:** **14-S21 — Add `--hmm-0-based-statepath` opt-in flag** (or
user decision to defer).

---

### Finding 9 — KNOWN_ISSUES [ISSUE:2026-04-18:1] proposes a new CLI flag

**Source (`KNOWN_ISSUES.md:283–306`, `[ISSUE:2026-04-18:1]`):**

> "`_locus_metrics()` in `onionskin_core/aps.py` currently floors per-bin excess at 0 …
> Likely landing zone: One-line change … **CLI flag to toggle the floor on and off
> (`--aps-area-excess-floor on|off`; default on unless off does better, then default off)**
> plus a report comparing cluster quality before/after."

**Gap:** A concrete CLI flag proposal (`--aps-area-excess-floor on|off`) sitting in
KNOWN_ISSUES. It fits Phase 14 spirit (new APS-group flag, explicit on/off convention
matching `--shape-score-strict-bic`). However, adding the flag is gated on an analytical
experiment determining the default.

**Recommended action:** Surface as **14-S22 — `--aps-area-excess-floor on|off` flag
(gated on experiment)**. Either add the flag now with `default=on` (matching current
behavior) and let the experiment decide the default later, or defer until experiment is
done.

---

### Finding 10 — KNOWN_ISSUES [ISSUE:2026-04-19:3] proposes --hmm-smooth-halfwidth split

**Source (`KNOWN_ISSUES.md:371–378`, `[ISSUE:2026-04-19:3]`):**

> "`--hmm-smooth-halfwidth` controls both step-5 whole-genome bedGraph smoothing and
> step-14 APS per-locus smoothing. These may want different window sizes but are currently
> locked together. Potential future split: `--hmm-smooth-halfwidth` (step-5) and
> `--hmm-aps-smooth-halfwidth` (step-14), both defaulting to the same value. Low priority."

**Gap:** A CLI-split proposal sitting in KNOWN_ISSUES. Directly Phase 14-style CLI
cleanup (split an overloaded flag into two with clear step mentions — exactly the
step-mention convention 14-S8 adopts). User marked it "Low priority" but did not defer
formally.

**Recommended action:** Surface as **14-S23 — Split `--hmm-smooth-halfwidth` into
step-5 and step-14 APS flags** (or user-confirmed defer).

---

### Finding 11 — BRAINSTORM.md `--validate-flags` was agent-deferred, not user-deferred

**Source (`BRAINSTORM.md` line 1992, 2026-04-21):**

> "Status: Speculative. Deferred from Phase 14 (rejected as out-of-scope; pre-parse
> deprecation errors already serve this purpose for Phase 14's needs)."

**Archived PHASE14_FEEDBACK Agent 1 Triage Round 4:**
> "`--validate-flags` developer mode — **REJECTED for Phase 14 scope.** … Deferring to
> `multi-agent/BRAINSTORM.md` as a speculative developer-tooling idea."

**Gap:** The rejection was decided by an agent, not by the user. Per the scope-authority
rule in AGENT_CONVENTIONS.md, deferral to BRAINSTORM without explicit user approval is the
exact pattern the user objected to. The user never weighed in on `--validate-flags`.

**Recommended action:** Surface as an explicit user decision. If the user approves the
deferral, leave in BRAINSTORM. If not, promote as **14-S24 — `--validate-flags` developer
scanner**.

---

### Finding 12 — BRAINSTORM: unresolved --pipelines multistage alias question

**Source (`BRAINSTORM.md:473–474`, `[2026-04-11]`):**

> "Naming/alias expectations should be clarified, especially whether the user-facing term
> should remain `growth` or whether **`multistage` should become an accepted alias** if
> that is more natural."

**Gap:** The entry is marked "Promoted to ROADMAP Phase 10 and formalized in the Phase 10
spec." But within Phase 14 Priority 14.5 (and the supplemental), we addressed `rms` vs
`rcn-mean-shift` aliasing but NOT `multistage` vs `growth`. The live code uses
"multistage" internally (in function names, engine references) but the user-facing
`--pipelines` token is `growth`. Is `--pipelines multistage` accepted as an alias?

**Verified:** `onionskin.py:_normalize_pipeline_token()` handles only `rms` and
`per-stage`, not `multistage`. So `--pipelines multistage` does NOT work.

**Recommended action:** Small: decide whether `multistage` should be accepted as a
`--pipelines growth` alias. Fold into 14-S19 (`--pipelines` help/contract clarification)
or surface as its own decision.

---

### Finding 13 — HMM placeholder shape-score flags: wiring plan question

**Source (user, PHASE14_FEEDBACK line 42):**

> "There **absolutely will be** an HMM flag for this as well so we can add it now as a
> placeholder: `--hmm-shape-score-threshold`"

**Current state (per 14-S7):** `--hmm-shape-score-threshold` and
`--hmm-shape-score-strict-bic` are "Inheritance placeholders — Parser-registered; no HMM
shape-score sink."

**Gap:** The user said "absolutely will be" — i.e., eventual wiring was the explicit
intent. 14-S7 clarifies placeholder wording but does not add any plan to wire the HMM
shape-score filter. This is arguably out-of-scope for a CLI supplemental, but the user's
"absolutely will be" statement suggests a roadmap item is appropriate.

**Recommended action:** Not a Phase 14 priority. Add to ROADMAP or BRAINSTORM as a
genuine future-phase item (analytical implementation, not CLI cleanup).

---

### Finding 14 — User's shape-scoring "valve" per pipeline

**Source (user, PHASE14_FEEDBACK lines 54–58):**

> "Each pipeline can have its own valve to turn this on or off regardless of the universal
> - e.g.:
>  `--hmm-shape-score-strict-bic [on|off]`
>  `--growth-shape-score-strict-bic [on|off]`
>  `--rms-shape-score-strict-bic [on|off]`
> These all default to 'None' which defaults to `--shape-score-strict-bic [on|off]`"

**Current state:**
- `--growth-shape-score-strict-bic` is wired (14.2-C loop closed).
- `--rms-shape-score-strict-bic` is a "parser-only placeholder" per 14-S7.
- `--hmm-shape-score-strict-bic` is a "parser-only placeholder" per 14-S7.

**Gap:** User's explicit intent was all three valves should work. Only one works. This
overlaps with Finding 13 — user wrote the intent, agents delivered partial scope.

**Recommended action:** Document the partial state in DECISIONS.md so future agents know
the user-stated intent vs delivered state. Fold into 14-S7 (placeholder help clarification
explicitly names what's wired vs not).

---

### Finding 15 — --rcn- prefix convention for RCN-profile-operating flags

**Source (user, PHASE14_FEEDBACK lines 73–76):**

> "`-rcn-` should be kept to differentiate it from the growth track, if this is for
> smoothing RCN files and not the growth track file."

**Generalization opportunity:** The user articulated a naming rule: flags that operate on
RCN profiles (not the growth track) should carry `-rcn-` in the name. `--growth-rcn-*`
follows this. But peak-summary flags (moved to Universal in 14-S2) operate on RCN
profiles — should they be `--rcn-peak-summary`? Asymmetric triangle model flags operate on
stage-median RCN profiles — should they be `--rcn-asym-tri-*`?

**Analysis:** The user-voice rule is narrow ("if this is for smoothing RCN files"). The
current supplemental does NOT apply this rule as a general naming convention. 14-S12
mentions RCN-profile-operating flags but as a Universal-promotion axis, not a
naming-prefix axis.

**Recommended action:** Open question — should the `-rcn-` prefix become a general
convention for RCN-profile-operating flags? If yes, some of the 14-S2/S3 Universal flags
need renaming.

---

### Finding 16 — Obsolete BRAINSTORM.md entries to be cleaned up

Several BRAINSTORM.md entries are now stale because Phase 14 addressed them:
- `[2026-04-15] CLI flag terminology harmonization — deferred from Phase 11 wrap-up
  triage` — Phase 14 did this. Entry is obsolete.
- `[2026-04-09] HMM sequence filtering — CLI flags` — `--chromosomes`,
  `--min-seq-length`, `--min-bin-count-per-seq` were added. Entry is obsolete.
- `[2026-04-11] Pipeline execution contracts and --pipelines completeness` — marked
  "Promoted to Phase 10" but Phase 14 work touches `--pipelines` further. At minimum,
  confirm the promotion still holds; otherwise update.

**Recommended action:** Housekeeping sweep as part of 14-S16 (or equivalent closeout doc
pass). Not substantive.

---

### Summary of second-pass findings

Beyond the six priorities I proposed in the first pass, the following user-intent items
were not adequately captured:

| Finding | Proposed priority | Source |
|---|---|---|
| Finding 5: `--pipelines` help verbose text lost + `all` semantics inconsistency | **14-S19** | PHASE14_FEEDBACK Q7 user answer |
| Finding 6: APS parser group Universal-promotion audit | **14-S20** | PHASE14_FEEDBACK "Same goes for timing and APS sections" |
| Finding 7: `--dedup-dist` semantics verification | Fold into 14-S20 | PHASE14_FEEDBACK Q4 user question |
| Finding 8: `--hmm-0-based-statepath` opt-in flag | **14-S21** | BRAINSTORM.md |
| Finding 9: `--aps-area-excess-floor on\|off` flag | **14-S22** | KNOWN_ISSUES [2026-04-18:1] |
| Finding 10: Split `--hmm-smooth-halfwidth` → + `--hmm-aps-smooth-halfwidth` | **14-S23** | KNOWN_ISSUES [2026-04-19:3] |
| Finding 11: `--validate-flags` was agent-deferred, not user-deferred | **14-S24** (or confirmed defer) | BRAINSTORM.md |
| Finding 12: `--pipelines multistage` alias decision | Fold into 14-S19 | BRAINSTORM.md |
| Finding 13: HMM shape-score wiring plan | ROADMAP, not Phase 14 | PHASE14_FEEDBACK user intent |
| Finding 14: `--rms-shape-score-strict-bic` and `--hmm-shape-score-strict-bic` placeholder state | Fold into 14-S7 | PHASE14_FEEDBACK user voice |
| Finding 15: `-rcn-` prefix convention generalization | Open question | PHASE14_FEEDBACK Q5 user voice |
| Finding 16: Obsolete BRAINSTORM.md entries cleanup | Fold into 14-S16 | Housekeeping |

---

## OPEN QUESTIONS ROUND 5 — Second-pass recon-audit follow-ups

### Feedback from John while reading
- Finding 5
	- RE: verbose content: Add my help text below what is there. What is there might need some fixing for accuracy, but serves as a nice "Quick start" for the flag. What I wrote could come after to give more context.
	- RE: Factual contradiction: fix help text
	- RE: all means all: fix help text 
	- I agree with : Restore user's verbose --pipelines help text; resolve the all semantics contradiction.
- Finding 6
	- "promotion to universal" needs to right scope here.
		- the term "universal" might be overloaded since there is a "universal" section and a "universal" concept in the overlap, timing, and aps sections
		- for APS, it is the concept of being universal, not to be moved to the section called "universal"
- Finding 7
	- verify, but keep where it is either way
	- this is an example flag that is forward-looking to universal use even if it is not right now
	- there is not an example in my mind where dedup dist logic would change across pipelines
	- definitley find out what pipelines and which steps it affects
	- and if pipelines are doing the same thing differently or with different functions / code, that should be discovered as well
		- we have an ongoing attempt to unify code and make it non-redundant ; pipelines should be sharing code as much as possible
- Finding 8
	- I believe we should add the --hmm-0-based-statepath flag
	- However, I believe when --hmm-0-based-statepath is used, it should change the defaults of  --hmm-thresh-state and --hmm-max-state-thres to 0 and -1 (None) or something
		- So there needs to be an argument catch that sees this and updates the defaults if those flags are not specifically set by the user
		- if those flags are specifically set by the user, then just use what they give
		- And their docstrings should be updated to reflect their adaptability to using --hmm-0-based-statepath
- Finding 9 
	- We should add --aps-area-excess-floor AND wire it into all three pipeline APS modules
	- Default should be to whatever the current behavior is to (default=on seems to be correct)
	- We likely need to handle two separate issues
		- 1. Area excess for APS scores : 
			- subtracting 1 and setting negative bins to 0 was a somewhat clever way to force each amplicon to contribute >= 0 to APS
			- however, since RCN values oscillate around 1, when 1 is subtracted and floored at 0, it systematically makes the APS higher than 0 across all bins.
			- it should only do the 0 operation at the end after the area is summed, if it is less than 0, set it to 0.
			- This allows the negative bins to keep the APS of an amplicon closer to 0 in general, and adds a slight correction after seeing the sum, if needed.
		- 2. Area excess for shape-based clustering:
			- Setting the floor to 0 probably does the most damage here, which is what the original hypothesis was.
			- we were trying to find the best way(s) to cluster to find the posterior groupings.
			- we realized setting the 0 floor (or RCN=1 floor) is possibly resulting in information loss with respect to shape similarities when many bins are set to 0 or 1.
	- My instinct is that the default will eventually be to not use the 0-floor or 1-floor methods
		- This was something introduced by an agent that I had never done in the past
		- I accepted it because it seemed harmless, and in some ways clever
		- But it might be over-correcting data in ways that make some downstream analyses worse, not better
	- For this phase, add it and wire it so it obeys the current behavior as default
		- In a future Phase, probably the end of Phase 15, we can add a Priority to finish testing this and finding the best default cluster conditions for posterior groups in general
			- Part of that will also be identifying truly "flat" non-amplification samples for "stage 1", then clustering the rest into automated groups
			- Part of the testing for that was things like ensuring no M-shaped posterior amplicons, seeing summit evaluations improve or at least stay the same, no "regressions"
			- We have notes on this elsewhere - this can be a late priority in phase 15
- Finding 10 
	- Do this split now and wire it: --hmm-smooth-halfwidth and --hmm-aps-smooth-halfwidth
	- Both can have the same defaults, but the flexibility is worth having for later testing
	- Make sure the help strings explain what each will apply to. Help strings should recognize the existence of the other similar flag, and note that defaults are currently the same.
- Finding 11
	- Regarding --validate-flags, I would need an explanation of what this would do and how it would benefit us.
	- Then I could decide to accept or reject for this phase.
- Finding 12 
	- "multistage" has been a retired term in reference to the "growth pipeline" for a long time, and should not be resurfaced as a term to use for the growth pipeline
	- It is a remnant from when there was only RMS for single stage analyses and Growth for multistage analyses.
	- Now all 3 pipelines are capable of "multistage" analyses.
	- So now "multistage" is treated as a property of the structure of the input data, not of pipeline capability
	- I thought we wrote this out somewhere, but if needed, it can be added to AGENT_CONVENTIONS for a more permanent reminder -- it might currently be in DECISIONS.md, but that is almost never read by agents
	- "multistage" should not be used by the growth pipeline internally either
		- it would be acceptable to use "multistage" only in the context of what type of data is being used, and for any pipeline.
		- such a scenario may not exist, but that would be acceptable
		- using that terminology to refer to growth pipeline stuff should be avoided, and we could fix this
	- `multistage` should NOT be accepted as a `--pipelines growth` alias. 
- Finding 13
	- HMM development will take place in Phase 15
	- This idea can be appended to the end of `multi-agent/plans/next/PHASE15_BRAINSTORM.md` for now
		- Explain it there in as much or more detail as it is explained elsewhere 
- Finding 14
	- If RMS pipeline is already set up to use this, which it is, then we should wire it now. That was the intention.
	- Same for HMM pipeline. If we have shape filtering there already, then wire it. 
	- For HMM, either way, I assume we will also develop this aspect more in Phase15, and discussing further development can be added to `multi-agent/plans/next/PHASE15_BRAINSTORM.md`
		- specifically, add mention that we should add to HMM what we do for RMS with the meta-analysis of amplicon shapes across stages to get a final set of amplicons and collapsed repeats
		- that idea will work with the HMM pipeline too; as will some other HMM-specific concepts such as detecting state path growth
- Finding 15 
	- This would need a full audit with a table of each flag, what group it currently sits in, what the suggested rename would be, and what the help text currently says (so 4 columns).
	- I can then make decisions on it
- Finding 16
	- "[2026-04-15] CLI flag terminology harmonization" can be cleaned up 
	- "[2026-04-09] HMM sequence filtering — CLI flags" can be cleaned up, assuming all is done - (is heatmap part done?)
	- "[2026-04-11] Pipeline execution contracts and --pipelines completeness" also seems like it can be cleaned up
	- Yes, do house-keeping sweep


**OTHER**:
- it seems the "universal" term might be tricking you or other agents up
  - you or other agents may conflate the concept of being universal with the category we named "Universal".
- technically the following argparse categories should be thought of as "universal"
  - Input / Output
  - Universal
  - Shape scoring
  - Overlap resolution 
  - Timing
  - Asymmetric Triangle Model
  - APS 
- So "universal" is a term meaning it is above or intended to be above >1 pipeline
  - and intended to be useful for >1 pipeline
  - but that can often be overriden by pipeline-specific versions of the same parameter
    - this is not always something we need to implement: some parameters may truly never need a pipeline-specific override whereas others will
- To avoid this terminology/concept confusion, perhaps
  - the Universal argparse category can be renamed to "General" or another better term
  - All categories that operate at the "universal" level can mention that in the top of their section.
- The only things that should not be "above" all pipelines are the pipeline-specific categories themselves (and their contents)
  - These are used for pipeline-specific flags and to fine-tune otherwise universal parameters
- Universal level flags can have different behaviors
  - be used in 1 pipeline for now with plans to expand to 1 or both other pipelines
  - be used in 2 pipelines with no further plans to expans
  - be used in all 3 pipelines
  - they can have also behave differently in terms of how pipelines inherit from them (I am sure you can see some differences)
    - pipelines might inherit the same value from a universal flag when it is not set
    - other universal flags may lead to different pipeline-specific behaviors when not set (e.g. norm-mode)
- These categories are an attempt to aggregate CLI flags by biological motivation
  - especially when for the most part, all pipelines can probably use the same parameter(s)
- Pipeline-specific flags are not always needed, but sometimes they are
  - for things that are becoming "universal" we can leave a pipeline-specific flag where it came from
  - for things that start out as "universal" we can add pipeline-specific flags when we see a need for them


### Q19 — `--pipelines` help text: restore verbose user version? Fix `all` semantics?

**Context:** User wrote ~110 words of educational help in PHASE14_FEEDBACK Q7. Live code
has ~40 words. User wrote "all = hmm,growth,rms." but help says "Default: all (growth +
rcn-mean-shift). HMM is opt-in." Runtime actually includes HMM when `all` is requested.

**Options:**
1. **Full restore.** Replace current help with the verbose user-written text verbatim, and
   fix the `all` semantics sentence to match user intent and runtime (remove "HMM is
   opt-in"). This respects the user-written spec.
2. **Fact-fix only.** Keep the short help but correct the `all` semantics wording. Do not
   restore the verbose educational content.
3. **Partial restore.** Restore the pipeline-definition sentences (what each pipeline is)
   but not the replicate/stage/normalization applicability prose.

**Secondary question (Finding 12):** Accept `--pipelines multistage` as an alias for
`--pipelines growth`? User once asked whether this should happen; never resolved.

**Answer from John M. Urban:**
- From my notes above on Finding 5:
	- RE: verbose content: Add my help text below what is there. What is there might need some fixing for accuracy, but serves as a nice "Quick start" for the flag. What I wrote could come after to give more context.
	- RE: Factual contradiction: fix help text
	- RE: all means all: fix help text 
	- I agree with : Restore user's verbose --pipelines help text; resolve the all semantics contradiction.
    - Restore but do it as I suggested above.
- Secondary Q: Reject. `--pipelines multistage` should NOT be an alias.
  - See reasoning in my above feedback section.

---

### Q20 — APS parser group Universal-promotion audit (14-S20)?

**Context:** User explicitly said "Anything in the overlap section should be considered as
potential to be universal even if it applies to only one pipeline right now. Same goes for
timing and APS sections." My first pass captured Timing as 14-S13 but not APS.

**Scope of 14-S20 (proposed):** Survey APS flags (`--aps-rank-by`, `--aps-cluster-k`,
`--aps-feature`, `--aps-singleton-guard`, etc. — there are ~8–12 APS flags). Determine:
- Which are already pipeline-agnostic in runtime behavior?
- Which have help strings that claim pipeline-specific scope?
- Should the APS section be re-framed as Universal (with per-pipeline overrides where
  needed)?

**Secondary (Finding 7):** Verify `--dedup-dist` semantics — does it apply to RMS AND
Growth in the live code? User asked; never confirmed.

**Question:** Accept 14-S20 as a priority? Any scope adjustments?

**Answer from John M. Urban:**
- From above:
- Finding 6
	- "promotion to universal" needs to right scope here.
		- the term "universal" might be overloaded since there is a "universal" section and a "universal" concept in the overlap, timing, and aps sections
		- for APS, it is the concept of being universal, not to be moved to the section called "universal"
- Finding 7
	- verify, but keep where it is either way
	- this is an example flag that is forward-looking to universal use even if it is not right now
	- there is not an example in my mind where dedup dist logic would change across pipelines
	- definitley find out what pipelines and which steps it affects
	- and if pipelines are doing the same thing differently or with different functions / code, that should be discovered as well
		- we have an ongoing attempt to unify code and make it non-redundant ; pipelines should be sharing code as much as possible

- Does that answer your questions?
  - APS is universal as in, it will be in all pipelines
    - if this means the section needs to be re-framed, then yes 
    - do not merge it with the "universal" section though
  - I am not sure at the moment what else needs to be done, but yes, you can determine and report:
    - which are already pipeline-agnostic in runtime behavior
    - which have help strings that claim pipeline-specific scope


---

### Q21 — `--hmm-0-based-statepath` opt-in flag (14-S21)

**Context:** Fully-designed opt-in flag in BRAINSTORM.md (line 1918). The design is
complete: default False, subtracts 1 from state-path outputs, makes `2^state` the direct
copy-number mapping.

**Options:**
1. **Add now (14-S21).** Promote from BRAINSTORM into the supplemental scope. Small
   implementation.
2. **Keep in BRAINSTORM, defer explicitly.** User-confirmed defer (not agent-decided).
3. **Add but gate on HMM work.** The flag depends on HMM output conventions; if HMM work
   is expected in a near-term phase, bundle it there.

**Answer from John M. Urban:**
- From my feedback above:
- Finding 8
	- I believe we should add the --hmm-0-based-statepath flag
	- However, I believe when --hmm-0-based-statepath is used, it should change the defaults of  --hmm-thresh-state and --hmm-max-state-thres to 0 and -1 (None) or something
		- So there needs to be an argument catch that sees this and updates the defaults if those flags are not specifically set by the user
		- if those flags are specifically set by the user, then just use what they give
		- And their docstrings should be updated to reflect their adaptability to using --hmm-0-based-statepath


- This is to be done, and can be done in this phase or the entire idea including my feedback can be appended to the end of `multi-agent/plans/next/PHASE15_BRAINSTORM.md` where we are collecting ideas for HMM development
  - So it will be either done right now or very soon
  - if you defer it to multi-agent/plans/next/PHASE15_BRAINSTORM.md, which is fine, then also
    - audit what would need to be done, and report audit findings there as well to save us work in the future


---

### Q22 — `--aps-area-excess-floor on|off` flag (14-S22)

**Context:** KNOWN_ISSUES [2026-04-18:1] proposes this flag. Gated on an analytical
experiment to determine the default.

**Options:**
1. **Add flag now with `default=on`** (current behavior), let experiment decide the
   default later. Pure CLI addition; no behavior change today.
2. **Keep in KNOWN_ISSUES until experiment runs.** Agent-defer rather than user-defer; but
   user did write "Likely landing zone" in the entry, implying they want it to land
   eventually.
3. **Add flag AND schedule the experiment** as a follow-up analytical task. This widens
   scope but aligns with user intent.

**Answer from John M. Urban:**
From my feedback above:
- Finding 9 
	- We should add --aps-area-excess-floor AND wire it into all three pipeline APS modules
	- Default should be to whatever the current behavior is to (default=on seems to be correct)
	- We likely need to handle two separate issues
		- 1. Area excess for APS scores : 
			- subtracting 1 and setting negative bins to 0 was a somewhat clever way to force each amplicon to contribute >= 0 to APS
			- however, since RCN values oscillate around 1, when 1 is subtracted and floored at 0, it systematically makes the APS higher than 0 across all bins.
			- it should only do the 0 operation at the end after the area is summed, if it is less than 0, set it to 0.
			- This allows the negative bins to keep the APS of an amplicon closer to 0 in general, and adds a slight correction after seeing the sum, if needed.
		- 2. Area excess for shape-based clustering:
			- Setting the floor to 0 probably does the most damage here, which is what the original hypothesis was.
			- we were trying to find the best way(s) to cluster to find the posterior groupings.
			- we realized setting the 0 floor (or RCN=1 floor) is possibly resulting in information loss with respect to shape similarities when many bins are set to 0 or 1.
	- My instinct is that the default will eventually be to not use the 0-floor or 1-floor methods
		- This was something introduced by an agent that I had never done in the past
		- I accepted it because it seemed harmless, and in some ways clever
		- But it might be over-correcting data in ways that make some downstream analyses worse, not better
	- For this phase, add it and wire it so it obeys the current behavior as default
		- In a future Phase, probably the end of Phase 15, we can add a Priority to finish testing this and finding the best default cluster conditions for posterior groups in general
			- Part of that will also be identifying truly "flat" non-amplification samples for "stage 1", then clustering the rest into automated groups
			- Part of the testing for that was things like ensuring no M-shaped posterior amplicons, seeing summit evaluations improve or at least stay the same, no "regressions"
			- We have notes on this elsewhere - this can be a late priority in phase 15



---

### Q23 — Split `--hmm-smooth-halfwidth` into step-5 and step-14 APS (14-S23)

**Context:** KNOWN_ISSUES [2026-04-19:3] — user marked "Low priority" but not formally
deferred. The split is pure Phase 14-style CLI cleanup: one overloaded flag → two flags
with clear step mentions.

**Options:**
1. **Do the split now.** New flag `--hmm-aps-smooth-halfwidth` (step-14 APS) defaulting to
   `--hmm-smooth-halfwidth` value. The latter stays (step-5). Small implementation.
2. **Keep in KNOWN_ISSUES.** User-confirmed defer.
3. **Different split semantics.** Rename both flags with explicit step tags
   (`--hmm-smooth-halfwidth-step5`, `--hmm-smooth-halfwidth-step14` or similar).

**Answer from John M. Urban:**
From my feedback section above:
- Finding 10 
	- Do this split now and wire it: --hmm-smooth-halfwidth and --hmm-aps-smooth-halfwidth
	- Both can have the same defaults, but the flexibility is worth having for later testing
	- Make sure the help strings explain what each will apply to. Help strings should recognize the existence of the other similar flag, and note that defaults are currently the same.

---

### Q24 — `--validate-flags` developer scanner: user-decide defer vs promote

**Context:** Agent 1 rejected `--validate-flags` for Phase 14 scope. User never
weighed in. Per scope-authority rule, agent-deferral without user input is the pattern to
avoid.

**Options:**
1. **User-confirm defer.** Leave in BRAINSTORM; user agrees it's not Phase 14.
2. **Promote to 14-S24.** Implement as a low-cost developer-tool helper.
3. **Promote but reframe.** Make it a sub-command rather than a flag (e.g.,
   `onionskin validate-flags path/to/script.sh`).

**Answer from John M. Urban:**
From above:
- Finding 11
	- Regarding --validate-flags, I would need an explanation of what this would do and how it would benefit us.
	- Then I could decide to accept or reject for this phase.

---

### Q25 — `-rcn-` prefix convention as a general naming rule? (Finding 15)

**Context:** User said `--growth-rcn-smooth-bins` should use `-rcn-` to distinguish from
growth-track smoothing. Should this become a general naming convention? Some current
Universal flags operate on RCN profiles (e.g., `--peak-summary`, `--asym-tri-model-*`).

**Options:**
1. **No generalization.** The user's rule is narrow to `--growth-rcn-*`. Leave
   Universal RCN-profile flags unprefixed.
2. **Codify as convention.** Any flag operating on RCN profiles carries `-rcn-` somewhere
   in the name. Rename `--peak-summary` → `--rcn-peak-summary`, etc. (substantial
   churn).
3. **Codify for growth-group overrides only.** Growth-group `--growth-*` overrides for
   RCN-profile flags use `--growth-rcn-*`; Universal flags stay prefix-free.

**Answer from John M. Urban:**
From above:
- Finding 15 
	- This would need a full audit with a table of each flag, what group it currently sits in, what the suggested rename would be, and what the help text currently says (so 4 columns).
	- I can then make decisions on it

---

### Q26 — HMM shape-score-strict-bic and rms-shape-score-strict-bic wiring (Finding 14)

**Context:** User's explicit intent (PHASE14_FEEDBACK): three per-pipeline valves all
wired. Current state: only `--growth-shape-score-strict-bic` is wired. RMS and HMM are
placeholders.

**Options:**
1. **Wire RMS now, keep HMM placeholder.** RMS shape-filter sink exists
   (`shape_filter_calls()`); wiring the RMS valve is arguably small. HMM has no
   shape-filter sink yet.
2. **Keep both placeholders; document user intent in DECISIONS.md.** Current 14-S7
   behavior with added forward plan.
3. **Full wiring push.** Wire RMS; build a shape-filter sink for HMM and wire it too.
   This is bigger and may extend into analytical work.

**Answer from John M. Urban:**
From above:
- Finding 14
	- If RMS pipeline is already set up to use this, which it is, then we should wire it now. That was the intention.
	- Same for HMM pipeline. If we have shape filtering there already, then wire it. 
	- For HMM, either way, I assume we will also develop this aspect more in Phase15, and discussing further development can be added to `multi-agent/plans/next/PHASE15_BRAINSTORM.md`
		- specifically, add mention that we should add to HMM what we do for RMS with the meta-analysis of amplicon shapes across stages to get a final set of amplicons and collapsed repeats
		- that idea will work with the HMM pipeline too; as will some other HMM-specific concepts such as detecting state path growth

- shorter answer based on your options
  - **Wire RMS now, keep HMM placeholder (and append this to PHASE15 file where we are collecting HMM development ideas).**

---

**Signed:** Claude Code (claude-opus-4-7) — 2026-04-22 ~20:40 EDT


---


## OPEN QUESTIONS ROUND 6 — follow-ups after user's Round 4/5 answers

**Author:** Claude Code 2.1.104 (claude-opus-4-7) — 2026-04-23

### Q27 — `--validate-flags` developer scanner: what it would do and how it would benefit us

Q24 asked you to defer or promote `--validate-flags`; you asked for an explanation first.
Here is the explanation so you can decide.

**What it would do:**

A developer-facing scanner that reads either a script file (e.g., a `.sh` or `.py`) or a
captured argv list, and reports every occurrence of a deprecated / retired / renamed
onionskin CLI flag. The scanner uses the same `_DEPRECATED_FLAGS` redirect table that
already powers the pre-parse gate in `onionskin.py:_check_deprecated_flags()`. Output would
list each hit with line/position + the current replacement.

Two possible implementation shapes:

1. **Top-level flag:** `onionskin --validate-flags path/to/script.sh` or
   `onionskin --validate-flags --argv "-- --z-thresh-single 4.0 --strict-bic"`.
2. **Sub-command (cleaner):** `onionskin validate-flags path/to/script.sh`.

**How it would benefit us:**

The pre-parse `_check_deprecated_flags()` gate already hard-fails any onionskin invocation
that uses a removed flag. But it only fires when onionskin is actually invoked. `tests/`
and `scripts/` sometimes contain code paths that are rarely exercised — a batch job that
runs once a quarter, a shell script invoked only by an evaluator, etc. Those scripts can
silently carry stale flag names for a long time before the pre-parse gate catches them.

A validate-flags scanner lets us walk `tests/`, `scripts/`, and any other ad-hoc caller and
report drift before the next invocation breaks. It is primarily a developer-ergonomics tool,
not a user-ergonomics tool.

**Counter-argument for deferring:**

The existing pre-parse gate already provides the safety net at runtime. The scanner's
marginal benefit is catching drift *earlier* (without running the script). In a small
project with tight agent control over the `tests/` and `scripts/` surfaces, that marginal
benefit may be low — a grep for old flag names would catch the same drift. The scanner is
more useful as onionskin grows external users.

**Decision options:**

1. **Accept for Phase 14 Supplemental** — small implementation (~50–100 lines of Python
   plus tests). Use the existing `_DEPRECATED_FLAGS` table. Preferred form: sub-command
   (`onionskin validate-flags path`).
2. **Defer to a future phase** — user-approved defer (legitimate per scope-authority rule
   since you now have the full context). Leave in `BRAINSTORM.md`.
3. **Replace with a Makefile target** — even lighter weight:
   `make validate-flags` that runs `grep -rnE "<regex>"` over `tests/` and `scripts/` and
   reports hits. No new CLI surface. Would satisfy ~90% of the drift-detection value
   without expanding onionskin's public API.

**Question:** Which option? If #1, confirm sub-command form. If #3, confirm target name.

**Answer from John M. Urban:**
- We can implement a version of this described below.
- After talking with Claude (opus), the design we liked was not a flag but a separate script:
1. scripts/validate_onionskin_flags.py — standalone script with its own argparse, imports _DEPRECATED_FLAGS from onionskin (or from a small shared module if you want to decouple).
2. make validate-flags Makefile target → one-word developer entry point; wraps python scripts/validate_onionskin_flags.py "$@". This solves the discoverability concern without adding Python dispatch complexity.
3. Mention the script in README.md and ONIONSKIN_FULL_HANDOFF.md under developer tooling.
4. No dispatch wrapper in onionskin.py.

---

### Q28 — 14-S12 growth-group flag survey triage

The 14-S12 survey (below, under `## 14-S12 Survey — Remaining growth-group flags`) lists
every remaining growth-group flag (post 14-S1/S2/S3) with an operates-on classification
and a recommendation. User approval is needed for any "promote now" or "promote next
supplemental" recommendation before Role 2 proceeds.

**Question:** For each flag in the survey below that has a "promote now" or
"promote next supplemental" recommendation, approve / modify / reject. Items without a
recommendation (e.g., "keep growth-only") do not need an answer unless you disagree.

**Answer from John M. Urban:**
Only responding to flags with "promote now" or "promote next supplemental" and flags I disagree with, if any.

First I want to focus on: `--growth-stage-weight-mode`
- This brought to my attention that stage-weight-mode had a role that could be involved in the HMM and RMS pipelines as well.
- I previously thought it was just being used for the growth modeling step, but that was incorrect based on your description.
- If it is truly something that can and will be expanded out to "Stage weighting in summit refinement + timing" steps in all pipelines, then perhaps it should be universal. It it hard to see why we would want different parameters here for different pipelines, so we can add pipeline-specific sub-versions of this if a need arises.
- For now it should be promoted to a universal flag with help text that says the pipelines and steps it applies to -- explicitly pointing out that it is currently only part of the growth pipeline.
- Is RMS not already in needed of this? Are we doing summit refinement differently in each pipeline?
- This also raised my awareness to "Summit Refinement" potentially deserving its own category in the argument parser. Should this be in a "Summit Refinement" section? Should it be in the "Timing" section? If it is in one and not the other, then the header text for both categories should point out that this flag affects both steps. Is a flag split needed? Would that make sense to do here?
- This seems to have raised more questions than answers.

Next: `--growth-refine-halfwidth` and `--growth-refine-smooth`
- These also seem like a candidates for the "summit refinement" category proposed above
- It says promote next supplemental, but there is not going to be one. Next is HMM dev. I would like a lot of this CLI organizational stuff and decision-making out of the way before starting the HMM
- So, this one also seems like it should have a universal flag `--refine-summit-halfwidth` or `--refine-origin-halfwidth`.
- The description of this flag raised to my awareness that I think we use "origin" and "summit" to refer to the same concept in different parts of the pipeline. Can you confirm? Certainly, it is true that we are using the summit as the technical thing to identify the replication origin, the biological thing. So the question is, do we use the technical term, which is more precisely what we are doing, or the biological term, which is more about what we are trying to do?
  - I would propose that changing the "origin" term to the "summit" term when they are interchangeable would have the smallest blast radius in terms of work. I like the "origin" term better because it is what I want to find, but it is a less precise, more hopeful term. "putative origin" would be more accurate, but that is lengthy. So we should probably refer to it as "summit" and explain in summit-relevant help strings that the summit is used as a proxy for the location of the re-replication origin and/or re-replication initiation zone.
  - So this brings up a new priority that we can attack now, at least for the public facing stuff.
    - when "origin" is referring to "summit", use "summit" in the flag name, and explain in the help text that it is an attempt to map the re-replication initiation zone or origin at the top of the amplicon. And "summit refinement" is an attempt to improve the analysis of the top region of the amplicon to better localize the origin zone.


Next: `--growth-trend` and `--growth-smooth` and `--growth-peak-search`
- Are these "windows", and if so, are these "halfwidths"?
- If they are halfwidths, 
  - then it seems to me we are trying to standardize CLI flags to contain that word
    - as well as trying to make sure help text is more clear as to whether it is a window size or a halfwidth size.
  - then the CLI flags should probably be:
    - `--growth-trend-halfwidth`
    - `--growth-smooth-halfwidth` 
    - `--growth-peak-search-halfwidth`
- Also, we are renaming `--growth-window` -> `--growth-scan-halfwidth`
  - It also appears that `--growth-peak-search` (potentially renamed `--growth-peak-search-halfwidth`) is a "stage-2" (second pass) parameter. So should it also get the "scan" terminology: `--growth-scan-peak-search` or `--growth-scan-peak-search-halfwidth`? Feel free to pushback here and suggest alternatives.


Next: `--growth-shape-filter`
- This is defaulted to false, but not clear why
- one question would be: is this applied to the growth track signal or RCN signal?
- another question would be: is there a need to apply it to both growth and RCN in the growth pipeline?
- another question would be: even though it is off by default, is there ever a time we actually don't want to apply this filter? I think it has been working great in the RMS pipeline, and feel comfortable defaulting to on.
- Perhaps we could keep this flag, but change it to the `--growth-shape-filter on|off` style some of our other flags have with a default to `on`.
- It can stay in growth alone because this flag is just saying whether to turn it on in growth or not.
- Seems like there is more discussion to be had here.

**Note**: It seems like all "promote next supplemental" were things we should do in "Phase 14 Supplemental". Nonetheless, I recognize you may not have the answers you need yet because more questions were raised.


---

### Q29 — 14-S25 `-rcn-` prefix audit: which renames do you want?

The 14-S25 audit (below, under `## 14-S25 Audit — -rcn- prefix convention candidates`)
lists flags whose value operates on RCN profiles but which do not currently carry `-rcn-`
in the name. For each, a suggested rename is given.

**Question:** For each flag in the audit below, approve / modify / reject the suggested
rename. Approved renames get implemented by Role 2 as a mini-priority inside 14-S25. If you
want the `-rcn-` convention formalized in `AGENT_CONVENTIONS.md`, say so and it joins the
14-S16 closeout doc sweep.

**Answer from John M. Urban:**
- your decisions are fine, but I noticed in all or almost all cases that the help string can be updated to more specifically say it is an operation on RCN signal.
- for example, 
  - Old: "Method to summarize per-call peak signal height across replicate stage medians for the peak profile output."
  - New: "Method to summarize RCN signal height for each peak call across stages for the peak summary profile output in <step-directory-location>. Each stage is represented as the per-bin median across its replicates."
    - Note - I am try to make this read in simpler easier-to-undertand language.
- While reading this, I also wondered if we should be using the term "peak" here or "amplicon".
  - Similar to the "summit" vs "origin" argument, this is a technical vs biological term.
  - However, here I wonder if the biological term wins out.
    - while we are unsure about origin location, we are pretty darn sure about what is and is not an amplicon.
    - moreover, peak is an overloaded term:
      - some people instantly think of ChIP-seq peaks, which are tiny compared to these
      - some people comflate "peak" and "summit"
  - Easiest fix is to stick with "peak" but update help text to explicitly state that in this context "peak" is referring to the entire breadth and shape of the mountain-shaped amplicons. So "peak call" really means the interval that defines the amplicon boundaries.
---

### Q30 — 14-S16 / 14-S17 RMS-APS wiring sub-scope: update PIPELINE_SPEC / HANDOFF / README?

14-S26 wires `--rms-shape-score-strict-bic` (and `--rms-shape-score-threshold` if needed)
into the RMS shape-filter sink. This is a non-trivial behavior change: RMS shape-filter
behavior now responds to a per-pipeline override. Previously the override was parser-only
and RMS used the universal value (or `off`, depending on wiring).

**Question:** Do you want the RMS-APS wiring to update user-facing docs (PIPELINE_SPEC.md,
ONIONSKIN_FULL_HANDOFF.md, README.md) as part of 14-S26, or should that ride the 14-S16
closeout doc sweep? Default assumption: 14-S16 closeout doc sweep. But if you want 14-S26
to carry its own docs update, the implementation scope doubles.

**Answer from John M. Urban:**
- Go with your recommendation for documentation. If the same thing is accomplished by the end of the phase, it does not matter to me when it is done.

---

## 14-S12 Survey — Remaining growth-group flags (Q28 drives triage)

**Produced:** 2026-04-23 by Claude Code 2.1.104 (claude-opus-4-7) per Q16 direction.

**Survey method:** Read `onionskin.py:build_parser()` Growth model group directly. For each
flag, classified by what it operates on and whether the concept is shared across pipelines.

**Already promoted or in scope elsewhere:**

| Flag | Status |
|---|---|
| `--peak-summary`, `--peak-quantile`, `--peak-topk` | Already Universal (14-S2) |
| `--asym-tri-model-*` | Already Universal (14-S3) |
| `--shape-score-threshold`, `--shape-score-strict-bic` | Already Universal (Phase 14.1) |
| `--bootstrap-origins` | Already Universal (Phase 14.1) |
| `--growth-norm-mode` | Per-pipeline override pattern, already in place (Phase 14.1) |

**Remaining growth-group flags (after 14-S1/S2/S3) — survey table:**

| Flag | Operates on | Shared conceptually across pipelines? | Recommendation | Justification |
|---|---|---|---|---|
| `--growth-fit-method` | Growth evidence track (trend-fitting across stages) | No — growth-model-specific math | **Keep growth-only** | Pure growth-track fitting; no RMS/HMM analog |
| `--growth-ensemble-methods` | Growth evidence track (ensemble over fit methods) | No | **Keep growth-only** | Tied to `--growth-fit-method` |
| `--growth-stage-weight-mode` | Stage weighting in summit refinement + timing | Yes (all pipelines refine summits across stages) | **Promote next supplemental** | Genuinely shared concept; promotion needs coordination with 14-S20 APS re-framing. Could become Universal with per-pipeline overrides in a follow-on wave. |
| `--growth-refine-halfwidth` | RCN profile (origin refinement search radius) | Yes (all pipelines refine summits on RCN) | **Promote next supplemental** | Operates on RCN; all three pipelines have or will have summit refinement. Candidate for Universal + `--growth/rms/hmm` overrides pattern. |
| `--growth-refine-smooth` | RCN profile (smoothing before refinement) | Yes | **Promote next supplemental** | Same category as `--growth-refine-halfwidth` |
| `--growth-stage-median-resolution` | Which RCN resolution feeds stage-median profiles | Conceptually yes but user said growth-only for now (Q6) | **Keep growth-only** (per Q6) | User-confirmed decision stands |
| `--growth-z-thresh` | Mean-shift detection threshold on growth evidence track | Concept is shared (RMS has `--rms-z-thresh`); but per-pipeline values likely legitimately differ | **Keep growth-specific** (parallel-naming pattern) | Growth and RMS already have parallel `--growth-z-thresh` / `--rms-z-thresh` flags. They can share a Universal `--z-thresh` that they inherit from, but the per-pipeline defaults legitimately differ (growth=4.5, RMS=see RMS defaults). A Universal promotion is defensible but churn-heavy; recommend staying parallel unless user asks otherwise. |
| `--growth-halfwidths` | Mean-shift detection halfwidths | Same as above (parallel with `--rms-halfwidths`) | **Keep growth-specific** | Same rationale |
| `--growth-trend` | Rolling-median baseline on growth evidence track | Growth-track specific ≠ RMS-track specific (different signals) | **Keep growth-only** | Growth operates on growth track; RMS operates on RCN tracks. Same word "trend" but different math. |
| `--growth-smooth` | Smoothing on growth evidence track | Same — growth-track specific | **Keep growth-only** | Same rationale |
| `--growth-peak-search` | Pass-2 peak search radius on growth evidence track | Shared concept with `--rms-peak-search` | **Keep growth-specific** (parallel pattern) | Same rationale as `--growth-z-thresh` |
| `--growth-window` → (14-S4 rename: `--growth-scan-halfwidth`) | Pass-2 scan halfwidth on growth evidence track | Shared concept with `--rms-scan-halfwidth` | **Keep growth-specific** (parallel pattern after 14-S4) | Same rationale |
| `--growth-rcn-smooth-halfwidth` | RCN profile (per-sample smoothing before summit + APS) | Yes — user explicit ("Not used by the HMM pipeline" but could be extended) | **Promote next supplemental** | Operates on RCN profiles (already has `-rcn-` prefix per Finding 15). Clear Universal candidate for smoothing-before-summit-refinement pattern. Consider 14-S25 `-rcn-` audit for naming. |
| `--growth-shape-filter` | Growth call-set shape-filter toggle | Parallel RMS flag already exists (`--shape-score-threshold` + sink) | **Keep growth-specific** | Already matches the universal/override pattern via `--shape-score-threshold` |
| `--growth-shape-score-threshold` | Growth shape-score threshold | Universal already; this is the override | **Already correct** | No-op |
| `--growth-shape-score-strict-bic` | Growth shape-score strict-BIC | Universal already; this is the override | **Already correct** | Wired in 14.2-C |
| `--growth-bootstrap-origins` | Growth bootstrap origins (placeholder) | Universal already | **Already correct** | Stays placeholder until bootstrap is implemented |

**Survey summary:**
- 3 "promote next supplemental" candidates: `--growth-stage-weight-mode`,
  `--growth-refine-halfwidth`, `--growth-refine-smooth`, `--growth-rcn-smooth-halfwidth`.
  (4 items — miscount; see table.)
- 4 "keep growth-specific (parallel pattern)" items: the detection flags
  (`--growth-z-thresh`, `--growth-halfwidths`, `--growth-peak-search`,
  `--growth-scan-halfwidth`) that already have `--rms-*` siblings. User may want to unify
  via Universal + override pattern in a future wave; not recommended now.
- All others: keep growth-only.

**Decision gate:** Q28 — user confirms which "promote next supplemental" items become a
mini-priority inside this supplemental (or deferred to Phase 15 / 16).

---

## 14-S25 Audit — `-rcn-` prefix convention candidates (Q29 drives decisions)

**Produced:** 2026-04-23 by Claude Code 2.1.104 (claude-opus-4-7) per Q25 direction.

**Scope:** All live `onionskin.py:build_parser()` flags whose value operates on RCN
profiles (not growth evidence track, not pure metadata).

**Already has `-rcn-` in name:**

| Flag | Group | Current help (excerpt) |
|---|---|---|
| `--growth-rcn-smooth-halfwidth` | Growth model | "Half-width in bins for the running median smooth applied to per-sample RCN before summit parabola fitting and APS computation in the growth pipeline." |

**Audit — flags operating on RCN profiles but without `-rcn-` in name:**

| Flag | Current parser group | Current help (excerpt) | Suggested rename |
|---|---|---|---|
| `--peak-summary` | Universal | "Method to summarize per-call peak signal height across replicate stage medians for the peak profile output." | **No rename** — "peak" reads cleaner than "rcn-peak". The phrase "across replicate stage medians" tells the user what data it operates on. Narrow `-rcn-` applies only when there is ambiguity between RCN profiles and growth evidence track; not ambiguous here. |
| `--peak-quantile` | Universal | "Fractile (0–1) used when --peak-summary quantile (default 0.9)." | **No rename** — same reasoning. |
| `--peak-topk` | Universal | "Number of top stage medians averaged when --peak-summary topk (default 2)." | **No rename** — same reasoning. |
| `--asym-tri-model-halfwidth` | Asymmetric Triangle Model | "Search halfwidth in kb around the refined amplicon origin used to fit the per-stage asymmetric triangle model." | **No rename** — model name "asymmetric triangle" uniquely identifies what is being fit. `-rcn-` would add noise without disambiguation. |
| `--asym-tri-model-smooth` | Asymmetric Triangle Model | "Smoothing span in kb applied to the stage-median profile before curvature estimation." | **No rename** — same. |
| `--asym-tri-model-halfwidth-grid` | Asymmetric Triangle Model | "Comma-separated grid of half-width values in kb…" | **No rename** — same. |
| `--growth-refine-halfwidth` | Growth model | "Search halfwidth in kb around the initial amplicon peak estimate within which the origin is refined by finding per-stage argmax positions." | **`--growth-rcn-refine-halfwidth`** — operates on RCN profiles (stage-median RCN argmax search). Parallel to `--growth-rcn-smooth-halfwidth`. Disambiguates from hypothetical growth-track refinement. |
| `--growth-refine-smooth` | Growth model | "Smoothing span in kb applied to stage-median profiles during origin refinement peak finding." | **`--growth-rcn-refine-smooth`** — same reasoning. |
| `--hmm-smooth-halfwidth` | HMM engine | "Smoothing half-width in bins for HMM step-5 whole-genome bedGraph smoothing…" | **No rename** — bedGraph IS the RCN surface in HMM; the flag name is unambiguous in its home group. Cross-ref with `--hmm-aps-smooth-halfwidth` (new in 14-S23). |
| `--hmm-trim-halfwidth` | HMM engine | "Trim half-width in bins applied during HMM step-5 median smoothing…" | **No rename** — same rationale. |
| (future) `--hmm-aps-smooth-halfwidth` | HMM engine (14-S23) | Step-14 APS per-locus smoothing on RCN profiles | **No rename** — same rationale |
| `--rms-trend` | RCN Mean Shift (RMS) | "Rolling-median baseline window in kb" | **No rename** — operates on RCN, but RMS group context makes this unambiguous; parallel to `--growth-trend` (on growth track). The parallel-naming decision for detection flags argues against divergence here. |
| `--rms-smooth` | RCN Mean Shift (RMS) | "Smoothing window for residual signal in kb" | **No rename** — same. |

**Summary:**
- 2 concrete rename candidates: `--growth-refine-halfwidth` → `--growth-rcn-refine-halfwidth`
  and `--growth-refine-smooth` → `--growth-rcn-refine-smooth`. These disambiguate from
  "growth-track refinement" (hypothetical; currently none) and parallel
  `--growth-rcn-smooth-halfwidth`.
- All other candidates: the auditor recommends **no rename** because the flag's group
  context or helper wording already disambiguates the data surface, and adding `-rcn-` would
  be noise.
- If you want a stricter convention ("every flag operating on RCN must have `-rcn-`"), the
  list of renames expands substantially — see the "Universal" and "Asymmetric Triangle
  Model" rows; they would all need `-rcn-` and `_DEPRECATED_FLAGS` entries.

**Decision gate:** Q29 — which suggested renames do you want, if any? If no, the audit
closes as "no action" and we codify the narrow convention (only use `-rcn-` when the flag's
name would otherwise be ambiguous with a non-RCN surface in the same group or at the same
prefix).

---

## 14-S20 — Preliminary APS flag inventory (full audit deferred to Role 2)

**Produced:** 2026-04-23 by Claude Code 2.1.104 (claude-opus-4-7) — preliminary inventory
only. Role 2 during 14-S20 implementation completes the runtime verification per Q20.

**APS group flags in `onionskin.py:build_parser()`:**

- `--aps-scale` (`choices=["none", "zscore", "both"]`)
- `--aps-cluster-method` (`choices=["hierarchical"]`)
- `--aps-cluster-k`
- `--aps-singleton-guard`
- `--aps-width-threshold`
- `--aps-feature`
- `--aps-shape-no-normalize`
- `--aps-focus-loci`
- `--aps-rank-by`
- `--aps-weight-loci`

**Expected action per Q20 (for Role 2):**
1. Re-frame APS group header to say it is Universal-in-spirit (applies to all three
   pipelines' APS computations) but keep it as its own parser group.
2. Add `[hmm: 14-aps; growth: 13-aps; rms: 12-aps]` step mentions.
3. For each flag, audit runtime reach — does it touch HMM step-14 APS as well as
   growth/rms APS? Fix help strings where they falsely claim pipeline-specific scope.
4. Verify `--dedup-dist` semantics (Finding 7): does it apply to RMS AND Growth? Update
   help accordingly. Surface code-unification opportunity to PHASE15_BRAINSTORM if
   different pipelines implement dedup differently.

---

**Signed:** Claude Code 2.1.104 (claude-opus-4-7) — 2026-04-23


---


## OPEN QUESTIONS ROUND 7 — tight follow-ups after Q28 raised more questions

**Author:** Claude Code 2.1.104 (claude-opus-4-7) — 2026-04-23

---

### Q31 — Where should the newly-Universal summit-refinement flags live?

`--stage-weight-mode`, `--refine-summit-halfwidth`, `--refine-summit-smooth`,
`--rcn-smooth-halfwidth` (promoted from growth per Q28).

Pick one:

1. **New "Summit Refinement" parser group** (between Timing and Asymmetric Triangle Model).
2. **Expand Asymmetric Triangle Model group** to cover summit-refinement too.
3. **Fold into Timing group.**
4. **Add directly to Universal section, no subgroup.**

My recommendation: **option 1** (new group).

**Answer from John M. Urban:**
- option 1 - new "Summit Refinement" group
  - can you confirm each option affects only or primarily summit refinement steps?

---

### Q32 — Confirm the Universal flag names.

Proposed:
- `--growth-stage-weight-mode` → Universal `--stage-weight-mode`
- `--growth-refine-halfwidth` → Universal `--refine-summit-halfwidth`
- `--growth-refine-smooth` → Universal `--refine-summit-smooth`
- `--growth-rcn-smooth-halfwidth` → Universal `--rcn-smooth-halfwidth`

Growth overrides keep the `--growth-*` prefix with the new stem.

Approve, or propose changes?

**Answer from John M. Urban:**
- Yes to:
- `--growth-stage-weight-mode` → Universal `--stage-weight-mode`
- `--growth-refine-halfwidth` → Universal `--refine-summit-halfwidth`
- `--growth-refine-smooth` → Universal `--refine-summit-smooth`

Update: 
- subject in question: `--growth-rcn-smooth-halfwidth` → Universal `--rcn-smooth-halfwidth`
- --growth-rcn-smooth-halfwidth INT
  - the help text says "Half-width in bins for the running median smooth applied to per-sample RCN before summit parabola fitting and APS computation in the growth pipeline."
- that it is for "summit parabola fitting" tells me this flag name is too generic.
  - should we try `--summit-parabola-smooth-halfwidth` 
- but that it is used for both parabola and APS tells me that would over-fit to the parabola application
- that it affects two categories makes it hard to assign to one or the other
- which category is it primarily for? one, the other, or both?
- I don't think it would make sense to put in "general" though
- is this also what is used in RMS and HMM pipelines for summit parabola fitting?
  - do the other pipelines use this operation that would require this flag?
  - do they use their own smoothed RCN bins computed earlier in the pipeline?
    - if not, why not?
    - if so, does this parameter affect that?
    - if so but we tried to force-implement HMM and RMS pipelines to use this parameter, would that force a second round of smoothing?
  - is it possible that this flag truly is growth pipeline specific since the growth pipeline might need further direction on how to treat RCN profiles?

Feedback would be appreciated here.

---

### Q33 — Detection-flag halfwidth audit: which renames?

Pushback on your earlier proposal after code check:

- `--growth-trend`, `--growth-smooth`: likely **windows, not halfwidths**. Recommend
  **no rename**.
- `--growth-peak-search`: help says "radius" → IS a halfwidth. Recommend rename to
  `--growth-peak-search-halfwidth`. **Do NOT** add a "scan" prefix (too long).
- Apply the same decision to parallel RMS flags.

Approve, or override pushback?

**Answer from John M. Urban:**
- Your way is fine.

---

### Q34 — `--growth-shape-filter` default flip to `on|off`?

Proposal: flip from `action="store_true"` (off default) to `choices=["on","off"], default="on"`.

Approve? Any semantic concern (growth-track vs RCN signal applicability) Role 2 should
verify and report in the implementation round?

**Answer from John M. Urban:**
- I approve this change. I am not sure what to look out for. If you want, we can use default off for now until it is further tested. That might make most sense. Then put a KNOWN_ISSUE that we need to test keeping it on. Does it change any results? I believe we ran this once and found out it does not. So it could be harmless to have on.
- Did you answer whether the shape filter in growth applies to growth signal or RCN?

---

### Q35 — `--bootstrap-origins` rename to `--bootstrap-summits` as part of 14-S28?

Summit/origin terminology harmonization (14-S28) wants "summit" everywhere it is
interchangeable with "origin." `--bootstrap-origins` is one candidate. But "origin" here
could specifically mean bootstrap-of-origin-coordinates, which is a narrower technical
meaning than "summit."

Rename, or keep as-is?

**Answer from John M. Urban:**
- We really need a better idea of what this bootstrap is doing.
- Is it refining the summit postion?
- "bootstrap-of-origin-coordinates" is not super meaningful here because we have no idea where the origin is. So how is "origin" being defined here? Is it starting out defined as a "summit"? if so, how is that being defined here? RCN max in local area? is this producing good results at all? I know parabola is one of our best summit (origin position) estimators. How does the results from this bootstrapping compare? What do these results affect? How are they used downstream?
- The help string says: "Bootstrap replicates for origin confidence interval (0 disables)."
  - what does it think the origin is?
  - this part of the pipeline is so old it likely refers to a lower quality origin classifier than the parabola, for example
  - we need to figure out what it is doing
  - we need to figure out if it would make more sense to not even do it, or to do it for the parabola instead
- Can you explain how bootstrap-of-origin-coordinates is a narrower technical
meaning than "summit"?


---

**Code-investigation findings I want to call out (not a question, just context):**

- `stage_weight_mode` currently used only in `growth_model_engine.py:refine_origin_for_call()`.
  RMS (`refine_summit_parabola`) and HMM do not use it today. The concept is applicable to
  them but wiring is Phase 15 work.
- "Origin" and "summit" are used interchangeably in the codebase. Public-facing harmonization
  to "summit" is 14-S28 scope. Internal function names (`refine_origin_*`) stay for now.
- The parser already has some "summit" flags (`--rms-summit-policy`,
  `--rms-early-summit-stages`) and some "origin" flags/help strings. 14-S28 normalizes this.

**Signed:** Claude Code 2.1.104 (claude-opus-4-7) — 2026-04-23 (Round 7 close)


---


## OPEN QUESTIONS ROUND 8 — three tight follow-ups from Round 7 investigation

**Author:** Claude Code 2.1.104 (claude-opus-4-7) — 2026-04-23

---

### Q36 — Confirm `--growth-rcn-smooth-halfwidth` stays growth-specific (not promoted to Universal)?

After code check: the flag's dual-use (summit parabola + APS) is growth-pipeline-specific
architecture. HMM has its own RCN smoothing via `--hmm-smooth-halfwidth` (step-5) and
`--hmm-aps-smooth-halfwidth` (step-14, from 14-S23). RMS doesn't pre-smooth RCN before
its summit parabola. Forcing a universal promotion would cause double-smoothing in HMM.

Recommend keeping as `--growth-rcn-smooth-halfwidth` in the Growth model group. Summit
Refinement group holds 3 flags (not 4).

Confirm? Or override and promote anyway?

**Answer from John M. Urban:**
- it is concerning that RMS does not pre-smooth RCN before its summit parabola. I believe we found that helped in an earlier testing round.
- just put that in KNOWN_ISSUES I suppose and get this as `--growth-rcn-smooth-halfwidth` in the Growth model group for now.

---

### Q37 — Rename `--bootstrap-origins` → `--bootstrap-summits`?

After code check: `_bootstrap_peak_ci` (RMS) and `refine_origin_for_call` (growth) both
bootstrap over replicates to produce CIs on the detected peak position. "Origin" here is
the same summit concept — legacy naming, not a narrower technical term. Output columns
are `origin_ci_low_bp`, `origin_ci_high_bp`, `origin_ci_width_bp`, `origin_boot_sd_bp`.

Recommend rename `--bootstrap-origins` → `--bootstrap-summits` per 14-S28. Growth and
RMS overrides (`--growth-bootstrap-origins`, `--rms-bootstrap-origins`) get the matching
rename.

Approve?

**Answer from John M. Urban:**
- `--bootstrap-summits` and same changes in RMS and Growth

---

### Q38 — Rename output columns `final_origin_bp`, `origin_ci_*_bp`, `origin_boot_sd_bp` too?

Q37 just touches CLI flag names. The `_origins.tsv` output columns currently include
`final_origin_bp`, `origin_ci_low_bp`, `origin_ci_high_bp`, `origin_ci_width_bp`,
`origin_boot_sd_bp`. Consumer call sites exist in `notebooks.py`, `summit_plots.py`,
test fixtures, and downstream analyses.

Options:

1. **Rename columns too** (consistent with 14-S28, larger blast radius — must update
   every read site).
2. **Rename CLI flags only, leave columns** (internal column names stay "origin"; small
   visible inconsistency: flags say "summit" but output columns say "origin"). Simpler
   now; could rename in a future phase if the inconsistency bothers you.

Recommend **option 2** for Phase 14 Supplemental. Rename columns is a bigger pass with
real blast radius; can be a Phase 15+ item bundled with any schema revisions.

Approve option 2, or want option 1?

**Answe from John M. Urbanr:**
- Option 1. Sorry but we need to be much more consistent. The columns are user-facing as far as I am concerned. We can't say origin here and summit there. We need to say summit. Moreover, we have more than one summit estimation method now, so it should actually have an additional tag that distinguishes it. For example, I believe parabola is usually tagged on when talking about the parabola estimator. What would be the equivalent here? What is a good tag to distribguish this summit estimate from others?

---

**Signed:** Claude Code 2.1.104 (claude-opus-4-7) — 2026-04-23 (Round 8 close)


---


## OPEN QUESTIONS ROUND 9 — final follow-up on column tag scheme

**Author:** Claude Code 2.1.104 (claude-opus-4-7) — 2026-04-23

---

### Q39 — Column rename tag scheme: use "final" as the method tag?

You asked in Q38: "we have more than one summit estimation method now, so it should
actually have an additional tag that distinguishes it. What is a good tag to distinguish
this summit estimate from others?"

My answer: **"final" IS already the method tag.** The column `final_origin_bp` is the
canonical/winner summit estimate per ROADMAP Priority 5.9.7 (parabola-mean with argmax
fallback). It is distinguished from the per-method columns that already carry their own
method tags:

- `argmax_mean_bp`, `argmax_median_bp` — argmax method
- `parabola_mean_bp`, `parabola_median_bp`, `parabola_vertex_bp` — parabola method
- `sliding_offset_bp` — sliding-offset method
- `final_origin_bp` — **final = "the winner"** (method-agnostic; the one that downstream
  code reads by default)

Proposed renames preserving "final" as tag:

| Old | New |
|---|---|
| `final_origin_bp` | `final_summit_bp` |
| `origin_ci_low_bp` | `final_summit_ci_low_bp` |
| `origin_ci_high_bp` | `final_summit_ci_high_bp` |
| `origin_ci_width_bp` | `final_summit_ci_width_bp` |
| `origin_boot_sd_bp` | `final_summit_boot_sd_bp` |

Alternative if you want bootstrap explicitly tagged on the CI columns (since the CIs
come from bootstrap, not from the winner estimator itself):
- `final_summit_ci_*_bp` → `final_summit_bootstrap_ci_*_bp`
- `final_summit_boot_sd_bp` → `final_summit_bootstrap_sd_bp`

More explicit but longer.

**Question:** Approve `final_summit_*` scheme, or want the `bootstrap` tag added to the
CI/SD columns?

**Answer from John M. Urban:**

This could be the answer BUT that depends on the investigation defined below.
| Old | New |
|---|---|
| `final_origin_bp` | `final_summit_bp` |
| `origin_ci_low_bp` | `final_summit_bootstrap_ci_low_bp` |
| `origin_ci_high_bp` | `final_summit_bootstrap_ci_high_bp` |
| `origin_ci_width_bp` | `final_summit_bootstrap_ci_width_bp` |
| `origin_boot_sd_bp` | `final_summit_bootstrap_sd_bp` |


JQ1: Does documentation anywhere tell the user that "final_summit_bp" corresponds to the parabola summit estimator?


JQ2: Is it correct to assume that there is a parabola column identical to the final_summit_bp column as?

JQ3: If final_summit_bp is the parabola estimator, do the bootstrap intervals have anything to do with it?
- Do we bootstrap the parabola now?
- Or is it still bootstrap information from a different estimator?
  - if it is from a different estimator, then it is not "final".
  - perhaps we should only do bootstrapping on what is considered final.
    - this would require extra coding that we could save until phase 15
    - the fix for now would be to not use "final" there, and tag it with the actual estimator that was bootstrapped... 
    - moreover, all things that inherit the CI intervals are called into question if we have determined they are from a poorer estimator than the parabola estimator. This likely affects the summit_inspector script, summit evaluations, and where ever it is used in our analyses.
    - when bootstrapping is done on the "best" summit estimator, it would be more meaningful
- Please investigate and get back to me.

---

## KNOWN_ISSUES items to create during Role 2 implementation

These items came up across Rounds 4–8. Role 2 creates the `multi-agent/KNOWN_ISSUES.md`
(or `multi-agent/tracking/KNOWN_ISSUES.md` after the 14-S16 closeout move) entries during
the relevant implementation round. Recording here so they are not forgotten.

1. **`--onset-*` vs `--timing-*` prefix harmonization** (from Q12 → 14-S13).
   Timing group currently has both prefixes; Q12 decided to defer unification.
2. **Test `--growth-shape-filter on` on live datasets** (from Q34 → 14-S30).
   Schedule a comparison round; if no regressions, flip default from `off` to `on` in a
   future phase.

## Summit Methodology items parked in `multi-agent/plans/next/PHASE16_BRAINSTORM.md`

Per Round 9 decision: the following summit-estimator / summit-refinement items are NOT
added to KNOWN_ISSUES. They are parked in the new Summit Methodology brainstorm file for
a dedicated future phase (likely Phase 16; filename currently `PHASE-TBD` because the
existing `PHASE16_BRAINSTORM.md` scaffold appears to have been consumed by Phase 13 and
can be archived).

- **Investigate RMS summit parabola pre-smoothing** (from Q36): user noted prior testing
  showed pre-smoothing helped.
- **Bootstrap the parabola estimator directly** (from Q39 / JQ3): currently the
  `final_summit_low/high_bp` columns mix semantics — range across per-stage parabola
  vertices (when parabola wins) vs bootstrap CI on argmax (when argmax fallback wins).
  `argmax_mean_bootstrap_sd_bp` is honestly named but is not the SD of `final_summit_bp`
  when parabola wins. Fixing this needs parabola-specific bootstrap.
- **Dynamic origin-detection window** (IBM-C5, also in
  `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md`): per-amplicon active-stage
  window for summit refinement.
- **HMM `peak_rcn_stage` + sliding-offset sub-bin** (IBM-C4): HMM summit accuracy work.
- **Cross-pipeline summit harmonization**: five estimator families exist across pipelines
  (argmax, parabola, sliding-offset, RMS selector policies, HMM state-path); decide which
  remain and which are unified.

**Signed:** Claude Code 2.1.104 (claude-opus-4-7) — 2026-04-23 (Round 9)

---

## 14-S20 APS audit — universal reach confirmed

Role 2 implementation confirmed that the APS parser group is Universal-in-spirit:
the APS flags reach all three pipeline APS computations through the shared
`compute_and_write_aps()` path and the HMM `run_step14_hmm_aps()` wrapper.

The earlier possibility that one or more APS flags were pipeline-specific is incorrect for
the current surface. APS is computed on the per-pipeline summit/amplicon set, but the
user-facing APS options control feature construction, clustering, rank computation, and
posterior staging consistently across Growth, RMS, and HMM.

`--aps-area-excess-floor` first received the explicit step mention in the 14-S22 /
v0.14.70 work. The 14-S20 pass extends the same
`[hmm: 14-aps; growth: 13-aps; rms: 12-aps]` framing to the rest of the APS group.

No PHASE15_BRAINSTORM cross-pipeline dedup note was added: Growth and RMS already share
`dedup_calls_by_peak_proximity()` in `onionskin_core/rcn_mean_shift_helpers.py`, while
HMM uses its own step-8 merging chain by design.
