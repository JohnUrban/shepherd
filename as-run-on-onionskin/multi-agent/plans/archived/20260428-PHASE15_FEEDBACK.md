# Phase 15 FEEDBACK — soup-to-brainstorm transfer + brainstorming-stage Q&A

**Stage:** Soup-to-brainstorm transfer (Agent 1 first round, 2026-04-27).
**Theme:** HMM completeness + enrichment.
**Companion files:** `multi-agent/plans/PHASE15_HMM_SOUP.md` (read-only after labeling pass; lives at this path during transfer until verified-complete and archived); `multi-agent/plans/PHASE15_BRAINSTORM.md` (just authored).
**Workflow:** Soup-to-brainstorm transfer per `multi-agent/workflows/phase-development-system_PDS-v2.md`.

---

## How to read this FEEDBACK file

This file is the Q&A surface that runs alongside BRAINSTORM authoring (transfer stage) and later alongside SPEC engineering (brainstorming + SPEC-engineering stage). Entries fall into a few categories:

- **Transfer-status block** — at the top of each Agent 1 round, current state of the soup-to-brainstorm transfer (partial / complete / needs-review-then-iterate).
- **Open questions for the user** — Q1, Q2, ... questions Agent 1 needs answered to proceed or to refine BRAINSTORM entries.
- **Line items for approval** — concrete proposals Agent 1 wants user sign-off on (splits, consolidations, scope-broadening, etc.). User answers in-place by editing this file.
- **Splits and consolidations across SOUP → BRAINSTORM** — interpretive choices Agent 1 made that need explicit user approval per the soup-to-brainstorm transfer convention.
- **User-introduced BRAIN entries (no SOUP source)** — log of BRAIN entries added at user direction during BRAINSTORM authoring rather than carried from SOUP.
- **Auditor findings** (later rounds) — from the auditor agent (Agent 2) when the transfer is reviewed.
- **Agent 1 follow-up notes** (later rounds) — what Agent 1 did to address user answers / auditor findings.

---

## Round 1 — soup-to-brainstorm transfer (Agent 1, 2026-04-27)

### Transfer-status block

**Status:** **Substantively complete; awaiting user review of splits/consolidations/open questions then auditor review.**

**Mechanical coverage:** All 21 SOUP IDs (SOUP15.1 through SOUP15.21) are referenced in at least one BRAIN entry's `Source:` field. See `PHASE15_BRAINSTORM.md` § Coverage check.

**BRAIN entry count:** 24 BRAIN IDs (BRAIN15.1 through BRAIN15.24).

- 21 BRAIN entries covering the 21 SOUP IDs with splits and consolidations as documented below.
- 1 BRAIN entry (BRAIN15.24 — `peak_rcn_stage` audit) added at user direction during BRAINSTORM authoring; has no SOUP source by design. See § "User-introduced BRAIN entries" below for the approval log.

**Items NOT yet in BRAINSTORM (intentionally deferred / cross-referenced rather than transferred):**

- IBM-C6A through IBM-C7B (post-HMM Phase 5/6 deferred items in `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md`) are not directly covered by any BRAIN entry. They are mentioned in BRAIN15.22's framing as "later candidates" but were not in the SOUP and not promoted into BRAINSTORM. Open question Q11 below: should any of these be promoted now or are they correctly out-of-scope for Phase 15?

---

### Splits and consolidations across SOUP → BRAINSTORM (Agent 1 interpretive choices needing user approval)

Per the soup-to-brainstorm transfer convention: splits in BRAINSTORM ARE allowed (multiple BRAIN IDs may cite the same SOUP ID in their `Source:` field) but require a documented FEEDBACK rationale. Consolidations (one BRAIN ID covering multiple SOUP IDs) are also allowed and similarly need a rationale. Both are listed below for explicit user approval.

#### Splits (1 SOUP → multiple BRAIN)

**Split 1: SOUP15.5 (Priority 12.1 — HMM completeness) → BRAIN15.5 + BRAIN15.8 + BRAIN15.13.**

- **Rationale:** SOUP15.5's body has an umbrella + a near-term carry-over queue with concrete distinct items. The umbrella narrative ("HMM completeness matrix across analysis families") is a different kind of idea from the concrete carry-over items ("`peak_rcn_stage` column", "sliding-offset sub-bin refinement", "`--hmm-smooth-halfwidth` APS split"). Each carry-over item is a discrete concrete priority that benefits from its own BRAIN entry to make implementation tracking and SPEC-stage citation straightforward.
- **BRAIN15.5** = umbrella (the matrix + analysis families list + the "implementation plan" steps 1–7).
- **BRAIN15.8** = `peak_rcn_stage` column + sliding-offset sub-bin refinement (gated on BRAIN15.24 audit; per the user directive 2026-04-27).
- **BRAIN15.13** = `--hmm-smooth-halfwidth` APS split, marked RESOLVED in Phase 14 Supplemental cycle 14S.23 — this BRAIN entry exists as a closeout marker; no Phase 15 implementation work.
- **Approval requested.**
    - Approved by John

**Split 2: SOUP15.8 (######OTHER cluster — 3 thin scratch entries) → BRAIN15.12 + BRAIN15.17 + BRAIN15.23.**

- **Rationale:** The three items in the SOUP cluster are unrelated topics that happened to share a "OTHER" header. Treating them as one BRAIN entry would muddle three independent priorities. Each gets its own BRAIN entry, all citing SOUP15.8 in `Source:`.
- **BRAIN15.12** = `--hmm-0-based-statepath` flag (item 2 in the cluster — also cites SOUP15.10 since the 14-S21 audit promoted this same topic into the SOUP under a separate header).
- **BRAIN15.17** = APS plots / fleshing out (item 1; thin scratch — needs user direction for concrete scope, see Q5 below).
- **BRAIN15.23** = `indiv_samples/` path pre-definition cleanup (item 3; folded into BRAIN15.6 execution as a tickler).
- **Approval requested.**
    - Approved by John

**Split 3: SOUP15.18 (CORRECTNESS BUG — `--hmm-mu-scale` + full HMM re-audit + missing PuffStep synonyms) → BRAIN15.9 + BRAIN15.10.**

- **Rationale:** SOUP15.18's body contains two discrete deliverables with different priority profiles: (a) the `--hmm-mu-scale` correctness bug + full HMM translation re-audit (correctness-sensitive, requires Opus Max Effort), and (b) adding missing PuffStep flag synonyms (CLI aliasing, much lighter touch). Splitting them prevents the synonym-adding work from being held up by the heavier re-audit cycle.
- **BRAIN15.9** = mu-scale correctness bug + full HMM translation re-audit.
- **BRAIN15.10** = missing PuffStep flag synonyms (`--hmm-mu`, `--hmm-discreteEmat`, `--hmm-sigma`, `--hmm-path`, `--hmm-constrainEmit`).
- **Approval requested.**
    - Approved by John


#### Consolidations (multiple SOUP → 1 BRAIN)

**Consolidation 1: SOUP15.7 + SOUP15.12 → BRAIN15.7 (amplicon reliability + flat-sample detection).**

- **Rationale:** The SOUP itself notes "These three items merge into a single future-phase priority: pre-APS reliability scoring, flat-sample detection, stage-1 anchoring." (SOUP15.12 body, end of the section.) Treating them as one BRAIN entry honors that explicit merge intent.
- **Approval requested.**
    - Approved by John

**Consolidation 2: SOUP15.17 + SOUP15.18 → BRAIN15.9 (`--hmm-mu-scale` bug + full HMM translation re-audit).**

- **Rationale:** SOUP15.17 ("True state means") is the user's narrative framing of why `--hmm-mu-scale` is wrong; SOUP15.18 ("CORRECTNESS BUG") is the formal bug report + re-audit demand for the same topic. Consolidating preserves both narratives in BRAIN15.9's body — the user-side framing as "User-side framing (from SOUP15.17)" and the formal bug as the body proper. (Note: BRAIN15.10 is also a partial consolidation target since SOUP15.18 includes the missing-synonyms section that's been split out per Split 3.)
- **Approval requested.**
    - Approved by John

**Consolidation 3: SOUP15.10 + SOUP15.8 item 2 → BRAIN15.12 (`--hmm-0-based-statepath`).**

- **Rationale:** SOUP15.10 (the 14-S21 audit) and SOUP15.8 item 2 (the older "stays in BRAINSTORM.md" scratch note) describe the same topic from two angles. The newer 14-S21 audit explicitly promoted the topic into the SOUP, superseding the older "stays in BRAINSTORM.md" note. BRAIN15.12 cites both; documentation tension flagged in Q1 below.
- **Approval requested.**
    - Approved by John


---

### User-introduced BRAIN entries (no SOUP source)

**BRAIN15.24 — `peak_rcn_stage` term audit.**

- **Origin:** User directive in chat (2026-04-27 transfer stage). Verbatim user note: *"`peak_rcn_stage` needs to be thoroughly audited by me at some point. It needs to be explained exactly to me. I say this because I know there have been a few times during development when I scratched my head about it. It is possible that it is an example of an agent implementing an idea that were not fully aligned on. It is also possible that it was implemented perfectly. The name might need to change. All I know is that there should be a note auditing that term even how it is used elsewhere across onionskin - and we should make sure it is correct when we adapt it to the HMM (and everywhere else inherits the correction if its corrected). Make sense? We can audit it even tonight - later on as part of brainstorming."*
- **Follow-up correction (2026-04-27 same session):** The user clarified that the concept might already be implemented in code under a different name (Growth or RMS), and Agent 1 should NOT have framed BRAIN15.24 as "no live code implementation" based on a literal-string grep. Verbatim user note: *"It might be somewhere in the code. I do not know at the moment. If you said 'has no live code implementation' based on something I said, then just know that I don't know that to be true, and we need to audit the code. I think it might be part of Growth or RMS already - or some version of it."* BRAIN15.24's framing was corrected immediately to reflect this — the audit's primary entry point is now "find any existing implementation under another name first" rather than "design from scratch."
- **Status:** Approved by user verbatim above. Logged here for provenance.
- **Why no SOUP source:** Per `AGENT_CONVENTIONS.md § Identifier system across phase-development stages`, a BRAIN entry without a SOUP `Source:` is allowed when the entry originates at BRAINSTORM authoring time at user direction (rather than being a silent agent introduction). This entry is the canonical documented case of that pattern for Phase 15.

---

### Open questions for the user

Each question is numbered. The user answers in-place by editing this file. Agent 1 reads the answers in the next round and updates BRAINSTORM accordingly.

#### Q1 — SOUP15.8 item 2 vs. SOUP15.10 documentation tension

SOUP15.8 item 2 (the older `######OTHER` scratch) says `--hmm-0-based-statepath` "stays in BRAINSTORM.md" — i.e., should NOT be promoted into a phase. SOUP15.10 (the newer 14-S21 audit) explicitly promotes this topic into Phase 15. BRAIN15.12 follows the newer thinking and treats the flag as in-scope for Phase 15.

**Q:** Confirm that the newer 14-S21 audit thinking supersedes the older "stays in BRAINSTORM.md" note? Or is there a middle ground (e.g., land the flag's CLI surface but defer the test additions to a later phase)?

**A:**
- `--hmm-0-based-statepath` is definitely part of Phase 15 and all things that say things like "stays in BRAINSTORM.md" are vestigial artifacts
- note that in the coming brainstorming stage, we will be updating what we plan to do with the default state path to basically be 0-based and the legacy state path to be 1-based
    - I just entered this into KNOWN_ISSUES, but the soup-to-brainstorm stage is not necessarily where we need to worry about that. For now we are just trying to start the BRAINSTORM file from the SOUP contents before updating it.

#### Q2 — `peak_rcn_stage` audit timing

Per BRAIN15.24, the audit can run as a brainstorming-stage activity (possibly tonight, per the user's own note) or formalize during SPEC engineering. BRAIN15.8 (HMM adoption of `peak_rcn_stage`) is gated on BRAIN15.24 clearing.

**Q:** Should BRAIN15.24 happen tonight as a brainstorming-stage activity, or formalize as a SPEC-engineering Priority? The brainstorming-stage option lets us settle the term and find/fix any existing live implementation before SPEC engineering even starts; the SPEC-engineering option folds it into the formal cycle structure.

**A:**
- It is already "tomorrow" but yes let's figure out peak_rcn_stage as part of brainstorming. That will help us brainstorm correctly.

#### Q3 — IBM-C5 (cross-pipeline dynamic origin-detection window) in-scope or design-only?

BRAIN15.22 covers the IBM-C bundle pointer. IBM-C2 is covered by BRAIN15.6+BRAIN15.20 and IBM-C4 by BRAIN15.8 — both implementation work. IBM-C5 is design-mode work per the recon audit; it currently sits inside BRAIN15.22 as "Phase 15 work for IBM-C5: author a design sketch ... decide which (if any) of IBM-C5 lands as concrete code in Phase 15 vs. waiting for a follow-up phase."

**Q:** Is IBM-C5 (a) design-only in Phase 15 (produce a sketch, defer implementation), (b) design + implementation in Phase 15 (full work), or (c) split — design in Phase 15, implementation deferred? Default per the SOUP framing is (a).

**A:**
- design during phase 15 brainstorming, flesh it out
- then given SPEC ID in phase 15 SPEC engineering
- implemented thereafter in phase 15 implementation


#### User-added background info from IBM file to help answer the Q
- **IBM-C2** | HMM parallel child pipeline + SAPS + step-14 APS raw-file fix | Phases 9/12 (scaffold-era "12.2") | Fully designed in BRAINSTORM; blocks biological-correctness bug in HMM step-14; never implemented
- **IBM-C4** | HMM summit accuracy — `peak_rcn_stage` column; sliding-offset sub-bin refinement for HMM | Phase 9 follow-up → scaffold-era "12.1" carry-over | Both fully designed in BRAINSTORM; depend on IBM-C2; never implemented
- **IBM-C5** | Dynamic origin-detection window (cross-pipeline summit refinement) | Phase 13 Priority 13.3 II/9A follow-up; [ISSUE:2026-04-18:3] | Design sketch pending; user explicitly flagged this as needing a design pass before more selector work
- 

#### Q4 — `--hmm-mu-scale` background-region-mean-norm "separate idea" (BRAIN15.9)

In SOUP15.17 the user noted: *"there could be something we do to learn the RCN mean values of what is determined to be background regions on each chromosome, and redoing the chromosome-specific normalization such that the median and/or mean of background regions = 1 as expected."* BRAIN15.9 currently flags this as a **separate idea — do not bundle into the mu-scale bug fix**.

**Q:** Confirm this is correctly treated as a separate (and lower-priority?) idea? If so, should it become its own BRAIN entry now (BRAIN15.25, say), or stay as a flagged sub-note inside BRAIN15.9 to be promoted later if it gets traction?

**A:**
- this should be a **separate idea — do not bundle into the mu-scale bug fix**.
- This idea would basically be
    - do steps 1-8
        - put results in subdirs for each step called pass1
    - define background on each chromosome as intervals that are not above background (defined in step 8 pass1)
    - go back to step 1
        - only use background regions to determine median
        - results can go in top-level of step directory (or in pass2 under the step directory)
    - proceed through steps 2-3
        - results can go in top-level of step directory (or in pass2 under the step directory)
    - in step 4, only use chromosome-specific background regions for renorm
        - results can go in top-level of step directory (or in pass2 under the step directory)
    - proceed from step 5 onward 
        - for steps 5-8, results can go in top-level of step directory (or in pass2 under the step directory)
        - for steps 9+, results can go in top-level of step directory
- Thus the new work boils down to:
    1. determining the background regions in step 8
        - for all things above background, including amplicons and collapsed repeats, use their widest representation found
            - this excludes as much possibly above background stuff as possible
            - this is most conservative in classifying bins as background
                - bins are classified as background if they are never seen to be above background
    2. using the background regions to re-run the pipeline
    3. storing the first pass results in the step directories under pass1/
    4. storing the second pass results in the step directories either at top level or under pass2/
        - under pass2/ makes it more obvious that they are pass2 results
        - but top-level means a lot less has to change in the code
            - for steps 1-8, pass1 results can live at top level until it is time for pass2 such that nothing needs to change about where files from previous steps are found
            - when ready for pass2, current results could be moved to pass1/ before redoing the step, and pass2 results would then live at the top level such that nothing needs to change about where files from previous steps are found
            - it also means the "final" results for each step are always top-level in each step directory. this could be useful on the user's end as well - always in same place for human or post-hoc scripts, etc.
        - agents could weigh in on opinions about this
- For now this can use a flag to turn it on 
    - perhaps `--hmm-bkgrd-renorm on|off (default: off)`
    - in the future we can default to `on`.
    - when it is `off` the results in directories 1-8 could still be stored under


#### Q5 — BRAIN15.17 (APS plots / fleshing out) concrete scope

The SOUP scratch is just "APS needs fleshing out -- plots, for example" — too thin to spec from. BRAIN15.17 currently proposes an interpretive scope: catalog existing APS analysis surface, identify gaps, implement missing plots/tables/notebooks.

**Q:** Is the proposed interpretive scope close to what you intended? Specifically: (a) what concrete plots/tables/notebooks are missing today that you'd want by end-of-Phase-15, and (b) is this BRAIN entry primarily about HMM-APS or also APS for growth/RMS?

**A:**
- Our goal will be to look at the analogous APS steps in Growth and RMS pipelines to ensure there are no gaps regarding what is offered by the HMM pipeline for APS analyses, plots, notebooks, etc
- The HMM pipeline should offer all analyses, outputs, plots, and notebook analyses offered by any other pipeline
- In general, we should make a file where we store all possible APS analyses, outputs, plots, notebooks, etc
    - this file can be used to ensure completeness in all pipelines through future phases
    - this file can and should be updated any time something is added regarding APS analyses, outputs, plots, notebooks, etc
    - it can be an easy look-up table for what is supposed to be present at minimum
        - if an analysis/output is necessarily specific to a pipeline, that can be mentioned in the table
    - we can audit each pipeline against the table from time to time to ensure they are complete
    - we can audit the table itself against the 3 pipelines in the code from time to time to ensure it is complete


#### Q6 — BRAIN15.10 (missing PuffStep synonyms) — flag prefix gaps from SOUP15.18

SOUP15.18 lists 4 PuffStep-flag → onionskin-flag pairs needing synonyms (`--hmm-mu`, `--hmm-discreteEmat`, `--hmm-sigma`, `--hmm-path`, `--hmm-constrainEmit`). It also notes that flags merely `--hmm-`-prefixed (`kmeans`, `iters`, `converge`, `emitpseudo`, `learnpseudo`, `transprobs`, `exp-decay`, `initialprobs`) "don't strictly need synonyms — the `--hmm-` prefix is predictable enough — but could be better matched by updating perhaps." BRAIN15.10 currently treats this as "open question for the brainstorming stage; default is no action."

**Q:** For the merely-prefixed flags, do we add synonyms (matching their PuffStep names exactly: `--hmm-kmeans` already matches `--kmeans` with prefix; `--hmm-iters` matches `--iters` with prefix, etc. — actually these already match)? On second look, the merely-prefixed flags don't actually need synonyms — the user's "could be better matched by updating perhaps" might refer to flag-naming polish (e.g., `--hmm-emitpseudo` → `--hmm-emit-pseudo`), not synonym addition. Confirm intent?

**A:**
- I saw what you wrote for BRAIN15.10 and I agree with it
- the "hmm" prefixes can be thought of as invisible - it is only what is after the prefix that we are comparing to puffstep flag names
- synonym status is assumed true if the flag name was not changed other than adding the hmm prefix
    - this can be explicitly stated in the header section of the HMM argparse group
- synonym notes are only needed for onionskin flags where our default highlighted flag name differs from the puffStep one (aside from the hmm prefix) -- i.e. when we made a decision to change it, we still offer the hmm-prefixed synonym version as well, and with a help string note for it.

#### Q7 — BRAIN15.15 (cross-pipeline dedup) status — what did 14-S20 find?

BRAIN15.15 is conditional: *"If 14-S20's `--dedup-dist` verification reveals that different pipelines implement dedup via different code paths, the unification belongs to Phase 15."*

**Q:** What did 14-S20 find regarding dedup divergence? (Or: where in the dev-system can I find the 14-S20 outcome?) If pipelines already share the dedup code path, BRAIN15.15 closes as N/A in Phase 15.

**A:**
- I will need you to search the code and the CHANGELOG, as well as the PHASE14 and PHASE14_SUPPLEMENTAL files to determine where we ended on this.
- Report the results back in the next FEEDBACK round, and to the chat.
- If it was already done, we can remove this entry, and mark it as "COMPLETE" or "DONE" anywhere in the codebase where it still exists as if it were "OPEN" or "ACTIVE". When doing that, follow AGENT_CONVENTIONS.md for files like KNOWN_ISSUES.md and BRAINSTORM.md (or use their own headers for guidance on how to close or remove an issue)
- If it is not already done, then make a SOUP file for it called DEDUP_SOUP.md and describe the intended work, the audit findings, and anything we would need in the future to pick back up on that thread.

#### Q8 — BRAIN15.19 (clustering defaults) dependency order

BRAIN15.19 (clustering defaults finalization) likely depends on BRAIN15.7 (amplicon reliability) and BRAIN15.14 (--aps-area-excess-floor analytical testing) landing first, since both affect the inputs to clustering.

**Q:** Confirm the dependency order: BRAIN15.7 + BRAIN15.14 → BRAIN15.19? Or is there a way for BRAIN15.19 to start in parallel with the inputs converging later?

**A:**
- Probably it is best for us to treat BRAIN15.19 as dependent on both.
- One test for BRAIN15.19 will be on which `--aps-area-excess-floor` behavior works better
- Amplicon reliability means we will be testing a better set of amplicons
    - testing would not strictly be dependent on this
        - but it would require us testing whether our conclusions changed
        - thus, it makes more sense to just wait until the set of amplicons is refined

#### Q9 — `--hmm-norm-mode` chrom-median consideration ([ISSUE:2026-04-21:1]) — promote to BRAINSTORM?

[ISSUE:2026-04-21:1] notes: *"HMM `--norm-mode` default: consider switching to `chrom-median` in a future phase."* This is HMM-completeness-relevant. The SOUP doesn't explicitly mention it; BRAINSTORM doesn't have a corresponding entry yet.

**Q:** Should this be promoted to a BRAIN entry (BRAIN15.26, say)? It naturally pairs with BRAIN15.21's chrom-median path consuming the pre-pipeline shared gap mask — both are about getting the chrom-median path to a state where it could become the HMM default.

**A:**
- This will be near the end of PHASE15
    - It will depend on putting filters in place to remove things like collapsed repeats
        - We just got done with using the shape filter in the RMS pipeline as part of multistage unification (step 8, 08-multistage-unification)
            - This worked very well, and allows RMS to not depend on ref-stage to eliminate things like collapsed repeats
            - multistage unification involves creating a union set of calls across all candidate amplicons and collapsed repeats, then running the shape filter on each candidate location across all stages, and using the results to give a final multistage classification of each candidate, splitting the union of candidates into a partition of amplicons and collapsed repeats. This is the final authoritative set of calls across all data that the pipeline can move forward with
                - Multistage unification cannot be done when only a single stage is given
                - In that case, there are more false positive classifications for collapsed repeats, which is okay and the expected behavior and result. 
                - The multistage unification process actually allows us to "rescue" amplicons that were falsely labeled as a collapsed repeat in one or more stages because we are using the evolution of the amplicon across multiple stages to update our view on it.
        - The same exact thing needs to be part of the HMM pipeline
            - In addition, the HMM pipeline allows us to monitor state path evolution across the stages for the final multistage unification of amplicon calls vs collapsed repeat calls. Both ideas should be part of the HMM pipeline, and multistage unification can default to one of a few behaviors: (1) use only the triangle filter based results, (2) use only the state path evolution based results, (3) only classify candidates as amplicons if they are classified as amplicons in both methods, (4) include any candidate that is classified as an amplicon in either method. It can default to 4 (least stringent filter). We can change that default if it turns out to be too permissive. Regardless of classification decision, both the multistage triangle analysis and multistage state path evolution analysis should be run and reported on (e.g. results in columns of a table for each amplicon)
                - The state path evolution idea to better classify candidate regions marked as above background in any stage is elaborated on elsewhere, but involves a very similar logic. When multiple stages are present, above background amplicon candidates are not excluded prior to evaluating them with respect to all stages. If there is only one stage, then filters for things like "length" (for example) might be needed to help eliminate false positives (e.g. anything < 50 kb with only one step up) at the expense of false negatives. But with multiple stages, we can more confidently say which regions are truly amplicons. Then we would not want to exclude any state path information from those regions. A very early stage might, for example, have a single step interval that is < 50 kb -- it might only be 10 kb -- but we would know that is really summit (origin) location information. In contrast, a region that is single-step and the same length (whether it is 10 kb or 400 kb) across all stages would more likely be a collapsed repeat. BUT if a single step pops up in the latest stage (and is not present in the stages before it) - we again would not treat that like a collapsed repeat because it was not present across all stages. Anything that pops up late and persists looks real, especially if it also passes the triangle filter. So with the statepath evolution, for each candidate we are looking to see: (1) does it persist after it pops up? (2) if it is present in multiple stages, does it get wider across them? (3) is it is present in multiple stages, does it evolve to have more than one step? (4) if it has more than one step, is it pyramid-shaped (with the summit interval approximately centered between steps on both sides)? For amplicons that start early and grow tall, all checks will pass (and it will survive the triangle multistage process as well). For amplicons that start early (or any stage except the last stage), but do not grow in height, they would need to at least pass checks 1 and 2, and pass the multistage triangle analysis (if triangle results were instructed to be used in the CLI flag discussed above). For amplicons that appear in only one stage, it would have to be the last stage to satisfy check 1, and it would have to pass the multistage triangle analysis (if triangle results were instructed to be used in the CLI flag discussed above).
- So yes, chrom-median should be the default after all the right filters and multistage unification results are in place. Then downstream results would depend only on candidates with "final" classifications as amplicons.

#### Q10 — `tracking/BRAINSTORM.md` HMM-relevant entries that didn't get cited as Source

The SOUP's introduction lists per-amplicon analysis families to cover, but several `tracking/BRAINSTORM.md` HMM entries are not directly cited as `Source:` in any BRAIN entry. Examples:

- `tracking/BRAINSTORM.md [2026-04-16] Long-horizon: unified per-sample smoothed RCN intermediates shared between pipelines` — broadly relevant to BRAIN15.6 (HMM parallel child pipeline) but not currently cited.
- `tracking/BRAINSTORM.md [2026-04-18] Per-chromosome APS clustering bootstrapping experiments` — relevant to BRAIN15.19 (clustering defaults) but not currently cited.
- `tracking/BRAINSTORM.md [2026-04-18] APS_mean_of_means clustering test` — relevant to BRAIN15.19 but not currently cited.

**Q:** Should these be added as `Source:` citations on the relevant BRAIN entries? (Default: yes, since they're informative for SPEC engineering. Confirming.)

**A:**
- RE: `tracking/BRAINSTORM.md [2026-04-16] Long-horizon: unified per-sample smoothed RCN intermediates shared between pipelines`
    - I think we actually concluded in an audit at one point that this would not make sense because the pipelines are doing things differently
    - Nonetheless, if we pursue this it would be in a future phase
    - this idea can be added to its own SOUP file with all appropriate references to other files
    - the SOUP file can be named UNIFIED-RCN_SOUP.md or something like that
- RE: `tracking/BRAINSTORM.md [2026-04-18] Per-chromosome APS clustering bootstrapping experiments`
    - This is a very interesting issue that we should keep in mind when optimizing APS clustering
    - But we should not raise it to a formal objective of this Phase
    - If we happen to make progress on this in this phase or side quests in this phase, then we should update that BRAINSTORM idea with relevant information we learn
    - So we might make some progress here, but it should not be a blocker for phase completion, and it should not be formalized into an objective; just mention it as something to keep in mind at the appropriate time, and to update if progress is made
- RE: `tracking/BRAINSTORM.md [2026-04-18] APS_mean_of_means clustering test`
    - Same general response as above -- 
        - This is a very interesting issue that we should keep in mind when optimizing APS clustering
        - But we should not raise it to a formal objective of this Phase
        - If we happen to make progress on this in this phase or side quests in this phase, then we should update that BRAINSTORM idea with relevant information we learn
        - So we might make some progress here, but it should not be a blocker for phase completion, and it should not be formalized into an objective; just mention it as something to keep in mind at the appropriate time, and to update if progress is made
- Note to agent:
    - It is okay to be bringing all this to light now, but wrapping ideas from BRAINSTORM.md and KNOWN_ISSUES.md and INTENDED-BUT-MISSED-PRIOR-TO-14.md (etc) technically is something we can do after ensuring the SOUP transfer was done successfully. Then the brainstorming stage aggregates from the other files to enrich the PHASE<N>_BRAINSTORM
    - It is okay that we are addressing some of it now, but make sure we keep these answers available to us when I use subsequent prompts that tell you (or another agent) to do a deep dive into BRAINSTORM.md and KNOWN_ISSUES.md and INTENDED-BUT-MISSED-PRIOR-TO-14.md for things to enrich this phase with


#### Q11 — IBM-C6A through IBM-C7B promotion?

`tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` includes IBM-C6A (APS order stability bootstrap), IBM-C6B (better posterior ordering logic), IBM-C6C (stage-activity fold-change using refined summit), IBM-C6D (timing-guided active-stage weights), IBM-C6E (iterative summit↔timing convergence), IBM-C6F (prior/posterior profile similarity), IBM-C7A (APS dimensionality reduction), IBM-C7B (shape-aware APS clustering) — all "post-HMM deferred" items. None are in the SOUP and none have BRAIN entries.

**Q:** Are any of these in scope for Phase 15? Phase 15 is HMM-completeness-focused — some of these (IBM-C6C, IBM-C6D, IBM-C6E, IBM-C7B) feel HMM-adjacent enough that they might fit. Default per "no SOUP source = not in scope without explicit promotion": leave out of Phase 15 unless explicitly promoted.

**A:**
- From the last few questions it seems like we should start making an APS_SOUP.md to start tracking all APS related ideas
- I will go through each item
    - IBM-C6A (APS order stability bootstrap), 
        - Defer
    - IBM-C6B (better posterior ordering logic), 
        - This might be of interest. Can you summarize what we wrote about it in IBM and BRAINSTORM (under tracking/) in the next FEEDBACK?
    - IBM-C6C (stage-activity fold-change using refined summit), 
        - This might be something we have to include. Can you summarize what we wrote about it in IBM and BRAINSTORM (under tracking/) in the next FEEDBACK?
    - IBM-C6D (timing-guided active-stage weights), 
        - Defer
    - IBM-C6E (iterative summit↔timing convergence), 
        - Defer
    - IBM-C6F (prior/posterior profile similarity), 
        - Defer
        - If something like this comes up during our work, it can be a side quest, and we can update its status
    - IBM-C7A (APS dimensionality reduction), 
        - Defer
    - IBM-C7B (shape-aware APS clustering)
        - Is this truly not already in the code? That surprises me. Don't we have a shape option? If so, what does it do? How does what we already do differ from what was written about here?
        - Can you summarize what we wrote about it in IBM and BRAINSTORM (under tracking/) in the next FEEDBACK?
        - shape-aware APS clustering could definitely be something we include if it truly is missing or incomplete

---

### Notes on SOUP body cruft (carried into BRAINSTORM cleanup, not into SOUP)

The following self-references in the SOUP body are **stale `next/`-era cruft** but were left in the SOUP body per the soup-to-brainstorm transfer rule (SOUP body is read-only after the labeling pass). They were stripped during BRAINSTORM transcription per modification rule 1 of the soup-to-brainstorm transfer convention.

- `Priority 12.1`, `Priority 12.2`, `Priority 12.3` headers (cruft from a scaffold-era "Phase 12" framing that was renamed many times during the SOUP's `next/` lifetime). These were captured as section names during BRAINSTORM transcription only when meaningful (e.g., in `BRAIN15.5` heading, `Priority 12.1` is preserved in a "from SOUP" parenthetical for traceability), otherwise stripped.
- `**Stage:** SOUP (pre-BRAINSTORM scratchpad in multi-agent/plans/next/)` framing in the SOUP header. The BRAINSTORM has its own stage-line framing.
- `Lifecycle: Promotes to a per-phase BRAINSTORM in multi-agent/plans/...` — accurate at the time of writing but now superseded; the BRAINSTORM has its own lifecycle-position framing.
- `## Maintenance: SOUP files are intentionally early-stage and disorganized.` — meta-framing for SOUP authors, not relevant to BRAINSTORM readers.
- "this future phase" / "future-phase" constructions throughout — the SOUP was authored before the phase number was known; BRAINSTORM is now Phase 15 and uses "Phase 15" directly.

The SOUP body itself was not edited beyond the labeling pass.

---

### Sourcing notes for the user (informational, not a question)

`tracking/BRAINSTORM.md` HMM-relevant entries used as `Source:` citations across BRAIN15.X:

- `[2026-04-14] HMM summit estimation — use stage of maximum summit RCN, not last stage` → BRAIN15.8.
- `[2026-04-14] HMM summit refinement — sliding-offset sub-bin localization` → BRAIN15.8.
- `[2026-04-18] Cross-pipeline origin detection windows` → BRAIN15.22 (IBM-C5 cross-ref).
- `[2026-04-18] HMM-native amplicon quality criteria` ("ghost levels") → BRAIN15.18.
- `[2026-04-18] SAPS — State-APS from individual-sample HMM decoding` → BRAIN15.20.
- `[2026-04-18] --hmm-0-based-statepath future CLI flag` → BRAIN15.12 (will close when BRAIN15.12 lands).
- `[2026-04-19] HMM parallel child pipeline (per-sample individual decoding) — HIGH PRIORITY` → BRAIN15.6.
- `[2026-04-19] Amplicon reliability scoring + flat-sample detection` → BRAIN15.7.

`tracking/KNOWN_ISSUES.md` HMM-relevant entries used as `Source:` citations:

- `[ISSUE:2026-04-14:1]` Stage-1 pre-amplification isolation → BRAIN15.7.
- `[ISSUE:2026-04-18:1]` APS area-excess floor default and clustering experiment → BRAIN15.14.
- `[ISSUE:2026-04-19:1]` HMM step-14 APS reads raw manifest bedGraphs → BRAIN15.6.
- `[ISSUE:2026-04-19:2]` Amplicon reliability scoring (PROMOTED to PHASE11_SPEC 11.5 but never landed) → BRAIN15.7.
- `[ISSUE:2026-04-19:3]` `--hmm-smooth-halfwidth` APS split — RESOLVED in 14S.23 → BRAIN15.13 (closeout-marker).
- `[ISSUE:2026-04-26:1]` `--hmm-mu-scale` silently scales emission means → BRAIN15.9.
- `[ISSUE:2026-04-21:1]` HMM `--norm-mode` chrom-median consideration → flagged in Q9 above for promotion decision.

`tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` IBM-C entries used:

- IBM-C2 → BRAIN15.6 + BRAIN15.20.
- IBM-C3 → BRAIN15.7.
- IBM-C4 → BRAIN15.8.
- IBM-C5 → BRAIN15.22 (Phase 15 design-sketch work).
- IBM-C6A through IBM-C7B → flagged in Q11 above for promotion decision.

---

### Next steps (after user answers Q1–Q11)

1. Agent 1 reads user answers, updates BRAINSTORM accordingly, appends a "Round 1 follow-up" section to FEEDBACK documenting what changed.
2. Once user answers + any iterative-pass updates are stable, the orchestrator invokes the **soup-to-brainstorm AUDITOR prompt** (Agent 2) per `multi-agent/workflows/phase-development-system_PDS-v2.md`. Agent 2 verifies SOUP ID coverage, BRAIN ID assignment, `Source:` provenance, scope drift, splits/merges, SOUP body integrity (post-labeling), and formatting/convention compliance. Verdict appended below as `## Auditor findings — round 1` (or a follow-up round if needed).
3. When auditor declares CLEAN, the orchestrator invokes the **soup-to-brainstorm CLOSEOUT prompt** to retire the SOUP file to `multi-agent/plans/archived/<YYYYMMDD>-PHASE15_HMM_SOUP.md` and close out the transfer stage. The official brainstorming stage can then begin (BRAIN15.24 audit may run as the first activity per Q2 above).

---

## Round 1 follow-up — Agent 1 (2026-04-28)

### Status

User answered all 11 open questions and approved all 6 splits/consolidations in Round 1 (verbatim "Approved by John" lines + multi-paragraph answers). Agent 1 incorporated each answer into BRAINSTORM and produced the deliverables below. Two new BRAIN entries were added at user direction (BRAIN15.25, BRAIN15.26) and one BRAIN entry was marked RESOLVED based on a code/CHANGELOG investigation (BRAIN15.15).

**New BRAIN entry count:** 26 (BRAIN15.1 through BRAIN15.26).
**SOUP coverage:** unchanged — all 21 SOUP IDs (SOUP15.1 through SOUP15.21) still cited; BRAIN15.25 and BRAIN15.26 are user-directive entries with no SOUP source (logged below).

### Per-question follow-up actions

**Splits/consolidations (all approved):** No BRAINSTORM changes needed — splits/consolidations were already implemented in the Round 1 BRAINSTORM. Approval logged.

**Q1 (SOUP15.8 vs SOUP15.10 documentation tension) → BRAIN15.12 updated.** Removed the "tension flagged in FEEDBACK" framing. Replaced with a note acknowledging the user's resolution: the older "stays in BRAINSTORM.md" scratch note was vestigial; `--hmm-0-based-statepath` is firmly in Phase 15. Also captured the user's note that the brainstorming-stage update (default state path 0-based, legacy 1-based) lives in `tracking/KNOWN_ISSUES.md` and will integrate into BRAIN15.12 during the brainstorming stage, not at this transfer stage.

**Q2 (BRAIN15.24 timing) → BRAIN15.24 updated.** Removed the "tonight or SPEC-engineering" framing. Replaced with the user's direction that the audit runs as part of the Phase 15 brainstorming stage, after the soup-to-brainstorm transfer closes out. User-quoted: *"It is already 'tomorrow' but yes let's figure out peak_rcn_stage as part of brainstorming. That will help us brainstorm correctly."*

**Q3 (IBM-C5 in-scope or design-only) → BRAIN15.22 updated.** Replaced the "decide later" framing with the user's full Phase-15 staged plan: design during brainstorming → SPEC ID in SPEC engineering → implementation in Phase 15 implementation cycle. IBM-C5 lands as concrete code in Phase 15. Background context the user provided (Phase 13 Priority 13.3 II/9A follow-up; [ISSUE:2026-04-18:3]; user-flagged as "needing a design pass before more selector work") was incorporated into BRAIN15.22.

**Q4 (mu-scale separate idea — background-region-mean-norm) → New BRAIN15.25.** Created BRAIN15.25 (HMM two-pass background-region-anchored chromosome-specific renormalization) capturing the user's full design verbatim: pass 1 runs steps 1–8; widest-representation reduction at step 8 defines background regions per chromosome; pass 2 re-runs the pipeline with background-region-only normalization. Includes the candidate flag `--hmm-bkgrd-renorm on|off (default: off)` and the directory-layout open question (option A pass1/pass2 subdirs vs option B top-level + on-the-fly archive). Agent recommendation noted (option B more pragmatic). Final layout decision deferred to SPEC engineering. **BRAIN15.9** was already framed as a separate idea — no edit needed; the user's separation directive was already honored.

**Q5 (BRAIN15.17 concrete scope) → BRAIN15.17 updated.** Replaced the interpretive-scope framing with the user's concrete deliverable scope: HMM should achieve **APS analysis-surface parity** with Growth and RMS (all analyses, outputs, plots, notebooks the other pipelines have). Added a master **APS catalog file** deliverable (location TBD; candidate `multi-agent/full_instructions/APS_CATALOG.md` or a tracking surface) that lists every APS-related analysis/output/plot/notebook across the three pipelines, marks each as Universal vs Pipeline-specific, is maintained going forward, and is auditable in both directions (catalog vs code). The maintenance of this catalog becomes a standing rule.

**Q6 (BRAIN15.10 — hmm prefix invisible rule) → BRAIN15.10 updated.** Captured the user-confirmed rule: the `--hmm-` prefix is invisible for PuffStep-vs-onionskin name comparison. Synonym status is **assumed true** for any flag whose post-prefix name is unchanged from PuffStep — no explicit synonym registration needed for those. The standing rule should be stated in the HMM argparse group's header. Synonym registrations + help-text notes are required ONLY when the post-prefix name was changed.

**Q7 (BRAIN15.15 — 14-S20 dedup outcome) → INVESTIGATED + BRAIN15.15 marked RESOLVED.**

Investigation findings (from `CHANGELOG.md` 14-S20 entry around line 588 onwards + code grep):

- **Growth and RMS already share** `dedup_calls_by_peak_proximity()` in `onionskin_core/rcn_mean_shift_helpers.py`. Growth calls it from `onionskin_core/engines/growth_model_engine.py:1382`; RMS from `onionskin_core/engines/rcn_mean_shift_engine.py:227` and `onionskin.py:4042 / 4142`.
- **HMM uses its own step-8 merging** (`--hmm-merge2`) by design; HMM intentionally does not use `--dedup-dist`. Help text rewritten in 14-S20 to make this explicit (`onionskin.py:1001-1012`).
- The 14-S20 closeout entry **explicitly noted "no PHASE15_BRAINSTORM cross-pipeline dedup note added"** — meaning 14-S20 deliberately tried to AVOID even creating BRAIN15.15. Its existence here is overhead from Phase 15 transfer that we now close.

**Action taken:** BRAIN15.15 marked **RESOLVED** (closeout marker, analogous to BRAIN15.13). No DEDUP_SOUP.md created (would have been if 14-S20 revealed unaddressed divergence; it didn't). Per user direction: *"If it was already done, we can remove this entry, and mark it as 'COMPLETE' or 'DONE' anywhere in the codebase where it still exists as if it were 'OPEN' or 'ACTIVE'."* — applied here.

**Q8 (BRAIN15.19 dependency order) → BRAIN15.19 updated.** Replaced "open question" framing with confirmed dependency on BRAIN15.7 + BRAIN15.14, including the user's rationale: BRAIN15.14 results inform BRAIN15.19's experimental matrix (which `--aps-area-excess-floor` behavior works better is one of BRAIN15.19's tests); BRAIN15.7 produces the reliability-filtered amplicon set that BRAIN15.19's clustering experiments should run on. SPEC-engineering note added: SPEC priority for BRAIN15.19 should explicitly list BRAIN15.7 and BRAIN15.14 as prerequisites in its `Source:` / `Depends-on:` field.

**Q9 (chrom-median default switch + multistage unification) → New BRAIN15.26 + BRAIN15.18 expanded.**

Created BRAIN15.26 (HMM `--norm-mode` chrom-median default switch — late-Phase-15 work gated on prerequisites). Sources: [ISSUE:2026-04-21:1] + user directive. Prerequisites listed: HMM shape-filter / triangle-based multistage unification + state-path-evolution-based multistage unification + combined classification flag (4 modes) + BRAIN15.21 (shared gap mask). Validation criteria captured.

Expanded BRAIN15.18 with the user's rich Q9 content:

- **Sub-priority (d) — combined multistage unification flag:** 4 modes (triangle-only / state-path-only / both-required / either-suffices), default = mode 4. Both methods always run + report in output table regardless of classification mode.
- **Sub-priority (e) — state-path evolution checks:** 4 explicit checks for amplicon-vs-collapsed-repeat classification (persists after pop-up; widens across stages; evolves to >1 step; pyramid-shaped if >1 step). Decision rules per amplicon profile (early-and-grow → all checks pass; early-no-grow → checks 1+2 + triangle pass; single-stage-late-onset → trivial check 1 + triangle pass). Filter philosophy: in multistage runs, do NOT exclude short/single-step regions before multistage analysis runs; in single-stage runs, fall back to length/step-count heuristics.

User explicitly noted (Q10) that further enrichment of (d) and (e) from `tracking/BRAINSTORM.md` etc. is brainstorming-stage activity — captured as a brainstorming-stage note in BRAIN15.18.

**Q10 (tracking/BRAINSTORM.md HMM-relevant entries not yet cited) → Three responses, two SOUP files created.**

- `tracking/BRAINSTORM.md [2026-04-16] Long-horizon: unified per-sample smoothed RCN intermediates`: Audit concluded earlier this doesn't make sense yet (pipelines doing things differently); future-phase. **Created `multi-agent/plans/next/UNIFIED-RCN_SOUP.md`** to preserve the idea with full audit-conclusion context, the three known blockers, cross-references to BRAIN15.6 and BRAIN15.21 (which partially relax some blockers), and open questions for whoever picks it up later.
- `tracking/BRAINSTORM.md [2026-04-18] Per-chromosome APS clustering bootstrapping experiments` and `tracking/BRAINSTORM.md [2026-04-18] APS_mean_of_means clustering test`: Per user direction, NOT formal Phase 15 objectives — keep in mind during APS clustering optimization, fold progress back into the original tracking entries if any occurs. **Captured in `multi-agent/plans/next/APS_SOUP.md`** under the "Side-quest awareness items" section.

User's general note for the agent: *"It is okay to be bringing all this to light now, but wrapping ideas from BRAINSTORM.md and KNOWN_ISSUES.md and INTENDED-BUT-MISSED-PRIOR-TO-14.md (etc) technically is something we can do after ensuring the SOUP transfer was done successfully. Then the brainstorming stage aggregates from the other files to enrich the PHASE<N>_BRAINSTORM. It is okay that we are addressing some of it now, but make sure we keep these answers available to us when I use subsequent prompts that tell you (or another agent) to do a deep dive into BRAINSTORM.md and KNOWN_ISSUES.md and INTENDED-BUT-MISSED-PRIOR-TO-14.md for things to enrich this phase with."*

**Action:** Captured. The brainstorming-stage deep-dive (when invoked) should re-read the relevant Q9, Q10, Q11 answers in this file as input.

**Q11 (IBM-C6A through IBM-C7B promotion) → APS_SOUP.md created + 3 items summarized + 5 items deferred.**

**Created `multi-agent/plans/next/APS_SOUP.md`** as the user requested, capturing:

- **Deferred from Phase 15** (per user disposition): IBM-C6A, IBM-C6D, IBM-C6E, IBM-C6F, IBM-C7A, plus IBM-C9 (related, not in user's Q11 list but cross-referenced).
- **Summarized for further investigation** (user "might be of interest" / "might be required" / "surprised"): IBM-C6B, IBM-C6C, IBM-C7B.
- Full IBM origin + scope quotes for each item.

Per-item summary findings reported below (per user request: *"Can you summarize what we wrote about it in IBM and BRAINSTORM (under tracking/) in the next FEEDBACK?"*):

**IBM-C6B — Better posterior ordering logic.** Origin: ROADMAP Priority 5.4 ✗ DEFERRED (post-HMM); `tracking/BRAINSTORM.md [2026-04-07]` cross-reference (no dedicated detailed entry — only the IBM file's one-line description). Scope (verbatim from IBM): *"Refine scalar / dendrogram / hybrid ordering relationship for posterior grouping."* Confidence: medium. The IBM file's one-line description IS the only substantive content. Deeper design work would be needed to spec.

**IBM-C6C — Stage-activity fold-change using refined summit (`timing.py` refactor).** Origin: ROADMAP Priority 5.0.2 (carried forward from Phase 4) ✗ DEFERRED (post-HMM); `tracking/BRAINSTORM.md [2026-04-07]` cross-reference (no dedicated detailed entry — only the IBM file's two-line description). Scope (verbatim from IBM): *"Non-trivial refactor of `timing.py` to recompute stage-activity fold-change using refined summit positions. Explicitly 'deferred until after Phase 7 HMM integration.'"* The Phase 15 connection: the HMM is gaining sliding-offset summit refinement (BRAIN15.8), so HMM-derived refined summits will exist by end-of-Phase-15. Whether to refactor `timing.py` to consume them in Phase 15 vs later is the decision.

**IBM-C7B — Shape-aware APS clustering.** Origin: ROADMAP Priority 6.3 ✗ DEFERRED (post-HMM); `tracking/BRAINSTORM.md [2026-04-07]` cross-reference. IBM file description (one line): *"Cluster using amplicon morphology, not just integrated magnitude. Post-HMM."*

**Investigation finding (you asked: "Don't we have a shape option?"):** YES. `--aps-feature shape` already exists, added at `CHANGELOG.md` entry "Added — `--aps-feature shape` and `--aps-shape-no-normalize`" (Phase 3.x range, per CHANGELOG line 12960 onwards). The flag clusters APS using per-locus per-bin RCN profile shape (with optional inverse-sqrt locus weighting via `--aps-shape-no-normalize`). This **implements the IBM-C7B brief description ("amplicon morphology, not just integrated magnitude") at face value**. Your surprise is justified — IBM-C7B is essentially **CLOSED by the existing `--aps-feature shape` implementation**.

**Possible nuance:** IBM-C7B's one-line scope is too brief to know whether it intended something MORE than `--aps-feature shape` does — e.g., explicit shape-descriptor matrices, parametric shape fits, multi-feature shape vectors, dimensionality-reduced shape embeddings. The deferred ROADMAP Priority 6.3 may have envisioned a richer shape-clustering surface than the per-bin profile that `--aps-feature shape` actually computes.

**Decision needed:** Either close IBM-C7B as RESOLVED-by-`--aps-feature shape` (no Phase 15 work) OR investigate whether the original 6.3 intent was richer and capture the gap. APS_SOUP.md leaves this open.

### New questions arising from Round 1 follow-up (Q12–Q14)

#### Q12 — Promote IBM-C6B and/or IBM-C6C into Phase 15?

User said both "might be of interest" / "might be something we have to include." Both have only one-line / two-line descriptions in IBM; deeper spec work would be needed before they could land.

**Q:** Now that you've seen the summaries (above), are either (or both) IBM-C6B / IBM-C6C in scope for Phase 15? If yes, they should be promoted to BRAIN entries (BRAIN15.27, BRAIN15.28) and SPEC priorities. If no, they stay in APS_SOUP.md (currently they live there in the "Items requiring more investigation" section). Default per the SOUP framing: stay in APS_SOUP.md unless explicitly promoted.

**A:**
- Ignore IBM-C6B
- IBM-C6C is essentially about a broader idea: use the correct summit for various analyses. Previously, separate analyses were estimating the summit position differently. This is a broader goal to consistently use the same summit. Final summit refinement and selection should happen after multistage unification. That will mean we have a final set of amplicon candidates. The summit refinement should use information from across all stages to get the best estimate. This is why we want to confine summit estimation to only stages there is origin firing activity - the "early" stages relative to a given amplicon's activity / timing. The later stages include only elongation, not replication initiation, so the summit region is expected to be come flatter like a table top rather than pointy like a peak. It is most pointy when the origin is actively firing, and the tip of the point is our best guess of where the origin is. The "timing" analyses get a little screwed up because we are trying to estimate stages of activity using the summit, but we want to estimate the summit using information about timing: what stages the origins are firing in. So it creates a circular problem. The fix is probably to start out with our best summit estimator position (summit refinement step after multistage unification) --> estimate timing ("timing" step)  --> refine summit estimators based on active stages ("active_stage_summit_refinement" step or something like that) --> update timing info (subsequent step "updated_timing"). This is similar to the "summit estimates <-> timing" convergence idea, but here I am not suggesting any further iterations. Best summit -> timing -> update summit -> update timing. Done. The changes to timing results in the timing update are likely to be marginal for most amplicons -- presumably even a coarse estimate of the summit position would work. But the updates to summit positions could be quite meaningful.
    - So wrapping it all up:
        - "Stage-activity fold-change using refined summit"
        - Yes we should ensure that the timing step is using the best summit position up to that point to do its calculations
        - Then we should update summit position based on active stages
        - Then we should update timing based on updated best summit positions

#### Q13 — IBM-C7B closeout decision

Investigation showed `--aps-feature shape` likely implements the IBM-C7B one-line scope as written. The remaining question is whether the original intent was richer than the current implementation.

**Q:** (a) Close IBM-C7B as RESOLVED-by-`--aps-feature shape` (no Phase 15 work; mark closed in IBM file with a closeout note pointing at the implementation), (b) investigate whether richer shape-clustering surfaces (parametric fits, shape-descriptor matrices, embeddings) were the original 6.3 intent — if so, capture the gap as new work, OR (c) defer the decision to brainstorming-stage when you can compare notes against the actual `--aps-feature shape` implementation behavior?

**A:** We investigated this heavily in chat, and have already acted on it. We are updating options for `--aps-feature` to include shape_pca, log2shape_pca, sample_shape_pca_raw, sample_shape_pca_reduced, amp_metrics, sample_metrics, metrics. We are also adding some flags: 
```
--aps-num-amp-pc <int> (Layer 1 PCs per amplicon; default 2). Only consumed by shape_pca, log2shape_pca, and sample_shape_pca_reduced (which uses Layer 1 internally).
--aps-num-sample-pc <int> (Layer 2 PCs; default 2). Only consumed by sample_shape_pca_raw and sample_shape_pca_reduced.
--aps-pca-matrix (Layer 3 toggle; default off). Composable with any --aps-feature choice.
--aps-final-pc <int> (Layer 3 PCs; default 2).
```

#### Q14 — UNIFIED-RCN_SOUP.md and APS_SOUP.md naming

Created two new SOUP files at `multi-agent/plans/next/UNIFIED-RCN_SOUP.md` and `multi-agent/plans/next/APS_SOUP.md`. Both follow the SOUP convention (no SELF-references to phase numbers; cross-references to closed phases allowed; `<THEME>_SOUP.md` naming).

**Q:** Confirm names are acceptable, or suggest alternatives. Particularly: `UNIFIED-RCN_SOUP.md` uses a hyphen in the theme — is that the right shape, or should it be `UNIFIED_RCN_SOUP.md` (underscore) or `UNIFIEDRCN_SOUP.md` (no separator)? AGENT_CONVENTIONS.md does not specify a sub-word separator within the theme name; existing SOUP files (`HMM_SOUP.md`, `SUMMIT_SOUP.md`, `FWD-ARCH_SOUP.md`) use both. So this is a minor naming preference question.

**A:**
- names are fine

---

### Updated transfer-status block

**Status:** **Substantively complete; awaiting user review of Round 1 follow-up actions, answers to Q12–Q14, and any optional iterative-pass tweaks; then auditor review.**

**Mechanical coverage:** All 21 SOUP IDs (SOUP15.1 through SOUP15.21) are referenced in at least one BRAIN entry's `Source:` field. Unchanged from Round 1. See `PHASE15_BRAINSTORM.md` § Coverage check.

**BRAIN entry count:** **26** BRAIN IDs (BRAIN15.1 through BRAIN15.26).

- 21 BRAIN entries covering the 21 SOUP IDs (with splits and consolidations approved in Round 1).
- 3 BRAIN entries with no SOUP source (user-approved): BRAIN15.24 (peak_rcn_stage audit, 2026-04-27), BRAIN15.25 (background-region-mean-norm two-pass, 2026-04-28 Q4), BRAIN15.26 (chrom-median default switch, 2026-04-28 Q9).
- 2 BRAIN entries marked RESOLVED (closeout markers, no Phase 15 implementation): BRAIN15.13 (`--hmm-smooth-halfwidth` APS split — resolved 14S.23), BRAIN15.15 (cross-pipeline dedup — resolved 14-S20).

**New SOUP files created (per Q10 / Q11 user direction):**

- `multi-agent/plans/next/APS_SOUP.md` — captures deferred IBM APS items + side-quest awareness items + IBM-C6B/C6C/C7B summaries pending Q12/Q13 decisions.
- `multi-agent/plans/next/UNIFIED-RCN_SOUP.md` — captures the long-horizon unified-per-sample-smoothed-RCN-intermediates idea preserved with audit-conclusion context for a future phase.

**Items still NOT in BRAINSTORM (deferred per Round 1):**

- IBM-C6A, IBM-C6D, IBM-C6E, IBM-C6F, IBM-C7A — captured in APS_SOUP.md as deferred-from-Phase-15.
- IBM-C9 (`--feature-track NAME:PATH`) — captured in APS_SOUP.md as deferred (related but not in user's Q11 list).
- Per-chromosome APS clustering bootstrap + APS_mean_of_means clustering test — captured in APS_SOUP.md as side-quest awareness items per Q10 user direction.
- IBM-C6B, IBM-C6C, IBM-C7B — captured in APS_SOUP.md "Items requiring more investigation" section pending Q12/Q13 decisions; if promoted, will become BRAIN15.27 / BRAIN15.28 / BRAIN15.29.

### Next steps

1. User reviews this Round 1 follow-up section, answers Q12–Q14, and either approves Round 1 follow-up actions as-is OR requests iterative tweaks.
2. Once stable, orchestrator invokes the **soup-to-brainstorm AUDITOR prompt** (Agent 2) per `multi-agent/workflows/phase-development-system_PDS-v2.md`. Agent 2 verifies SOUP ID coverage, BRAIN ID assignment, `Source:` provenance, scope drift, splits/merges (now including the 3 user-directed BRAIN15.24/25/26 entries with no SOUP source), SOUP body integrity (post-labeling), formatting/convention compliance. Verdict appended below as `## Auditor findings — round 2`.
3. When auditor declares CLEAN, orchestrator invokes the **CLOSEOUT prompt** to retire the SOUP file to `multi-agent/plans/archived/<YYYYMMDD>-PHASE15_HMM_SOUP.md` and close out the transfer stage. Brainstorming stage begins (BRAIN15.24 audit kicks off there per Q2; BRAIN15.18(d)+(e) enrichment happens there per Q9/Q10; BRAIN15.22 IBM-C5 design happens there per Q3).

---

## Round 2 follow-up — Agent 1 (2026-04-28, post Q12-Q14 answers + multi-round APS design discussion)

### Status

User reviewed Round 1 follow-up and answered all three open questions (Q12, Q13, Q14). Q12 was a mixed answer (IBM-C6B ignored, IBM-C6C promoted with substantial new design detail). Q13 transitioned into a multi-round design discussion of richer APS feature modes that produced two new BRAIN entries (BRAIN15.27 + BRAIN15.28). Q14 confirmed names. Round 2 added **3 new BRAIN entries** (BRAIN15.27, BRAIN15.28, BRAIN15.29) plus a new DECISIONS.md entry plus updates to APS_SOUP.md.

**New BRAIN entry count:** **29** BRAIN IDs (BRAIN15.1 through BRAIN15.29). Up by 3 from Round 1.

**Note on the BRAIN-ID slot prediction in Round 1:** The Round 1 transfer-status block predicted that IBM-C6B / C6C / C7B would land at BRAIN15.27 / BRAIN15.28 / BRAIN15.29 if promoted. Actual outcome: Q12 ignored IBM-C6B (no slot needed) and promoted IBM-C6C (landed at BRAIN15.29); Q13 effectively resolved by the BRAIN15.27 + BRAIN15.28 design discussion (slots used for the new APS feature modes + summit_rcn rename, NOT for the original IBM-C7B promotion); IBM-C7B was found to be already RESOLVED by existing `--aps-feature shape` (Priority 6.1) AND further covered by BRAIN15.27. The BRAIN-ID slot allocation differs from the Round 1 prediction but produces the same total count of 3 new entries (28 + 1 = 29 — wait, that gives 29 from 26, which matches — Round 1 was 26 BRAIN entries, Round 2 added 3 to get 29).

### Per-question follow-up actions

**Q12 (IBM-C6B / IBM-C6C promotion) → MIXED ANSWER.**

- **IBM-C6B (better posterior ordering logic):** "Ignore" per user direction. No promotion to Phase 15. Stays in `multi-agent/plans/next/APS_SOUP.md` "Items requiring more investigation" section — available for promotion in a future phase.
- **IBM-C6C (stage-activity fold-change using refined summit; `timing.py` refactor):** **PROMOTED to BRAIN15.29** with substantial new design detail. The user's Q12 answer goes well beyond the IBM file's two-line scope: a four-step one-pass convergence pattern (best initial summit → timing → active-stage-confined refined summit → updated timing). Distinct from EM-iterative IBM-C6E (which stays deferred). User's verbatim design preserved in BRAIN15.29's body with full quoted reasoning. Hard dependencies on BRAIN15.18 (multistage unification), BRAIN15.8 (sliding-offset refinement + `peak_rcn_stage`), BRAIN15.6 (per-sample step-5 outputs), BRAIN15.24 (`peak_rcn_stage` audit clearing).
- APS_SOUP.md IBM-C6C entry updated to "PROMOTED to BRAIN15.29" with closeout pointer.

**Q13 (IBM-C7B closeout) → ANSWERED via the BRAIN15.27 + BRAIN15.28 design discussion.** User's verbatim Q13 answer confirms: *"We investigated this heavily in chat, and have already acted on it. We are updating options for `--aps-feature` to include shape_pca, log2shape_pca, sample_shape_pca_raw, sample_shape_pca_reduced, amp_metrics, sample_metrics, metrics. We are also adding some flags: `--aps-num-amp-pc <int>`, `--aps-num-sample-pc <int>`, `--aps-pca-matrix`, `--aps-final-pc <int>`."* All captured in BRAIN15.27. The summit_rcn rename + RCN-vs-excess migration captured in BRAIN15.28. APS_SOUP.md IBM-C7B updated to RESOLVED-by-existing-and-by-BRAIN15.27.

**Q14 (UNIFIED-RCN_SOUP.md / APS_SOUP.md naming) → ANSWERED "names are fine".** No action; both SOUP files keep their current names.

### New BRAIN entries added in Round 2

**BRAIN15.27 — Composite multi-feature APS clustering modes with three-layer PCA composition.** Per-amplicon features (`summit_rcn`, `width`, `log10area`, `asymmetry_ratio`, `shape_score_raw`); per-sample scalars (sample-level summit-APS, width-APS, log10-area-APS); three composable PCA layers (Layer 1 amplicon-level shape PCA via `--aps-num-amp-pc`; Layer 2 sample-level shape PCA via `--aps-num-sample-pc` with `_raw`/`_reduced` variants; Layer 3 matrix-level PCA via `--aps-pca-matrix` + `--aps-final-pc`); seven new `--aps-feature` modes (`shape_pca`, `log2shape_pca`, `sample_shape_pca_raw`, `sample_shape_pca_reduced`, `amp_metrics`, `sample_metrics`, `metrics`); soft deps on BRAIN15.7 / 15.14 / 15.19 / 15.28.

**BRAIN15.28 — `summit_excess` → `summit_rcn` rename + RCN-as-default migration.** Renames `peak_rcn` → `summit_rcn`; deprecates `summit_excess` and `mean_excess` as per-locus columns; preserves excess+floor as opt-in for sample-level APS aggregation per user's "0-floor makes sense for individual amplicon contributions to sample-level APS" note. Behavior change for `--aps-feature summit` flagged in CHANGELOG when it lands. Area handled separately by BRAIN15.14.

**BRAIN15.29 — Summit-then-timing-then-updated-summit-then-updated-timing convergence (one-pass).** Promotes IBM-C6C with rich design. Four-step pattern: (1) best initial summit post-multistage-unification → (2) estimate timing → (3) refine summit confined to active stages (NOT all stages — addresses elongation-flattening problem) → (4) update timing using refined summits. **NOT EM-iterative** per user direction — single pass, stop after step 4. User's verbatim quote captured. Hard deps on BRAIN15.18 (multistage unification) + BRAIN15.8 (summit refinement infrastructure) + BRAIN15.6 (per-sample outputs) + BRAIN15.24 (`peak_rcn_stage` audit). IBM-C6E (EM-iterative variant) stays deferred per user explicit distinction.

### New DECISIONS.md entry

**[2026-04-28] APS feature mode candidates — accepted/rejected catalog + RCN-vs-excess design decision** appended to `multi-agent/project_context/DECISIONS.md`. Catalogs:
- **Accepted features** folded into BRAIN15.27.
- **Rejected features:** Fourier descriptors (structural mismatch), curvature features (noise + redundancy), multi-family BIC vectors as sample-clustering feature (covered by BRAIN15.18 for amplicon classification), `peak_rcn` as distinct feature (redundant with `summit_rcn` post-rename).
- **RCN-vs-excess design decision:** migrate to raw RCN as default; preserve excess+floor as opt-in. User-verbatim reasoning preserved.

This is the canonical home for the rejection rationales — future agents asking "should we try Fourier / curvature / etc.?" should be pointed there, not buried in this archived FEEDBACK file.

### APS_SOUP.md updates

- **IBM-C6C entry** updated to PROMOTED-to-BRAIN15.29 with full design pointer.
- **IBM-C7B entry** updated to RESOLVED status with closeout pointer to 6.1 + BRAIN15.27.
- **Existing APS-related Phase 15 BRAIN entries cross-reference list** updated to include BRAIN15.27 + BRAIN15.28 + BRAIN15.29.
- **New "Future-phase candidates added during Phase 15 transfer follow-up (2026-04-28)" section** added: amplicon-level clustering as future-phase candidate per user note Q13 follow-up; provenance section for rejected candidates with brief rationales pointing at DECISIONS.md.

### Updated transfer-status block (post-Round-2)

**Status:** **Substantively complete; ready for auditor review (Q12-Q14 all answered, all approved actions executed).**

**Mechanical coverage:** All 21 SOUP IDs (SOUP15.1 through SOUP15.21) cited. Unchanged from Round 1.

**BRAIN entry count:** **29** BRAIN IDs (BRAIN15.1 through BRAIN15.29).

- 21 BRAIN entries covering the 21 SOUP IDs (splits and consolidations approved in Round 1).
- 6 BRAIN entries with no SOUP source (user-approved):
  - BRAIN15.24 — `peak_rcn_stage` audit (2026-04-27).
  - BRAIN15.25 — Background-region two-pass HMM renorm (2026-04-28 Q4).
  - BRAIN15.26 — Chrom-median default switch (2026-04-28 Q9).
  - BRAIN15.27 — Composite APS feature modes + 3-layer PCA (2026-04-28 Q13 multi-round design).
  - BRAIN15.28 — `summit_excess` → `summit_rcn` rename + RCN migration (2026-04-28 same discussion).
  - BRAIN15.29 — Summit ↔ timing one-pass convergence; IBM-C6C promotion (2026-04-28 Q12).
- 2 BRAIN entries marked RESOLVED (closeout markers, no Phase 15 implementation): BRAIN15.13 (`--hmm-smooth-halfwidth` APS split — resolved 14S.23), BRAIN15.15 (cross-pipeline dedup — resolved 14-S20).

**Files touched in Round 2:** `multi-agent/plans/PHASE15_BRAINSTORM.md` (added BRAIN15.27 + BRAIN15.28 + BRAIN15.29; updated Coverage check); `multi-agent/project_context/DECISIONS.md` (new 2026-04-28 entry); `multi-agent/plans/next/APS_SOUP.md` (IBM-C6C → PROMOTED-to-BRAIN15.29; IBM-C7B → RESOLVED; cross-ref list updated; future-phase candidates section added).

### Next steps (post-Round-2)

1. Orchestrator invokes the **soup-to-brainstorm AUDITOR prompt** (Agent 2) per `multi-agent/workflows/phase-development-system_PDS-v2.md`. Auditor verifies all 29 BRAIN entries, the 6 user-introduced no-SOUP-source entries (15.24/25/26/27/28/29), the 2 RESOLVED markers (15.13/15.15), splits/consolidations approval state, SOUP body integrity, formatting/convention compliance.
2. When auditor declares CLEAN, orchestrator invokes the **CLOSEOUT prompt** to retire the SOUP file to `multi-agent/plans/archived/<YYYYMMDD>-PHASE15_HMM_SOUP.md`. Brainstorming stage begins (BRAIN15.24 audit kicks off there per Q2; BRAIN15.18(d)+(e) enrichment happens there per Q9/Q10; BRAIN15.22 IBM-C5 design happens there per Q3).
3. **Optional follow-on cleanup work flagged for the brainstorming stage or a separate cleanup pass:**
   - Update `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` IBM-C7B entry to mark RESOLVED with closeout pointer to 6.1 + BRAIN15.27.
   - Update `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` IBM-C6C entry to mark RESOLVED-via-promotion-to-BRAIN15.29.
   - Update `ROADMAP.md` Priority 6.3 from ✗ DEFERRED to ✓ DONE per the DECISIONS.md 2026-04-28 entry.
   - Update `ROADMAP.md` Priority 5.0.2 from ✗ DEFERRED to ✓ PROMOTED-to-Phase-15 (or mark resolved when BRAIN15.29 lands).
   - All append-friendly cleanup items; not blocking the auditor review.

---

## Auditor findings — round 1

- **Verdict:** `OPEN — Agent 1 follow-up pass needed`

- **Per-criterion findings:**
  1. ✓ **SOUP ID coverage:** All 21 SOUP IDs are covered. `PHASE15_HMM_SOUP.md` contains `SOUP15.1` through `SOUP15.21`; `PHASE15_BRAINSTORM.md` Source lines plus its Coverage check map every ID to at least one BRAIN entry (`PHASE15_BRAINSTORM.md:989-1012`).
  2. ✓ **BRAIN ID assignment:** `PHASE15_BRAINSTORM.md` has 29 BRAIN entries, `BRAIN15.1` through `BRAIN15.29`, sequential and gap-free (`PHASE15_BRAINSTORM.md:31-900`).
  3. ⚠ **Source provenance citations:** Every BRAIN entry has a `Source:` field (29 headings, 29 Source lines). One provenance-text inconsistency remains: BRAIN15.27 and BRAIN15.28 point to the wrong FEEDBACK round label even though their user-approval trail exists.
  4. ✓ **No silent scope narrowing:** I found no unapproved narrowing. The two resolved/closeout markers (BRAIN15.13, BRAIN15.15) are explicitly documented as already completed, not silently deferred.
  5. ✓ **No silent scope broadening / new-idea injection:** The six no-SOUP-source BRAIN entries are logged as user-approved in the Round 2 transfer-status block (`PHASE15_FEEDBACK.md:598-604`) and have Source fields pointing to user directives or prior tracking issues.
  6. ✓ **Splits / merges:** The three SOUP splits and three consolidations are documented with rationales and explicit "Approved by John" lines (`PHASE15_FEEDBACK.md:49-94`). No unapproved split/merge was found.
  7. ✓ **SOUP body integrity:** `git show 762fc63 -- PHASE15_HMM_SOUP.md` shows the labeling commit changed only 21 headings to add `SOUP15.1` through `SOUP15.21`; current `git diff -- PHASE15_HMM_SOUP.md` is empty.
  8. ✓ **Open FEEDBACK questions:** Q1-Q11 and Q12-Q14 are answered. The remaining cleanup list at `PHASE15_FEEDBACK.md:614-617` is explicitly marked optional / not blocking auditor review.
  9. ✓ **Transfer-status block:** FEEDBACK contains a current post-Round-2 transfer-status block stating "Substantively complete; ready for auditor review" (`PHASE15_FEEDBACK.md:591`). Because the only finding is a small provenance-text fix, the substantive status is close but not quite archive-ready until Agent 1 corrects it.
  10. ⚠ **General formatting / convention compliance:** Formatting is consistent overall. The only convention issue is the stale FEEDBACK-round reference in BRAIN15.27 and BRAIN15.28 Source text.

- **Specific issues:**
  1. `multi-agent/plans/PHASE15_BRAINSTORM.md:765` — BRAIN15.27's Source says it is "logged in `PHASE15_FEEDBACK.md` Round 1 follow-up", but BRAIN15.27 is created and logged in Round 2 (`PHASE15_FEEDBACK.md:543-571`, final no-SOUP-source list at `PHASE15_FEEDBACK.md:598-604`). This is a stale provenance label; the user directive itself is documented.
  2. `multi-agent/plans/PHASE15_BRAINSTORM.md:855` — BRAIN15.28's Source also says it is "logged in `PHASE15_FEEDBACK.md` Round 1 follow-up", but BRAIN15.28 is created and logged in Round 2 (`PHASE15_FEEDBACK.md:543-571`, final no-SOUP-source list at `PHASE15_FEEDBACK.md:598-604`). Same provenance-label issue.

- **Recommendations:**
  1. Agent 1 should update only the two stale Source-line phrases in BRAIN15.27 and BRAIN15.28 from "Round 1 follow-up" to "Round 2 follow-up" (or "Round 2 follow-up / post-Round-2 transfer-status block").
  2. After that small BRAINSTORM fix, request a narrow re-audit focused on Source-line provenance for BRAIN15.27/BRAIN15.28 and final archive readiness.
  3. If the re-audit is clean, the orchestrator can run the closeout step and archive the SOUP with:
     `git mv multi-agent/plans/PHASE15_HMM_SOUP.md multi-agent/plans/archived/20260428-PHASE15_HMM_SOUP.md`

---

## Round 2.5 — Agent 1 follow-up to Auditor findings (2026-04-28)

### Status

Auditor (Codex, Round 1 audit) returned verdict **OPEN — Agent 1 follow-up pass needed** with one narrow finding: BRAIN15.27 and BRAIN15.28 had `**Source:**` lines saying *"Round 1 follow-up"* even though those entries were authored in Round 2. Agent 1 confirmed the finding and applied the fix.

### Action taken

**BRAIN15.27 Source: line updated** in `multi-agent/plans/PHASE15_BRAINSTORM.md`. Changed two occurrences of *"Round 1 follow-up"* → *"Round 2 follow-up"* in the Source line (one for the FEEDBACK-section reference, one for the DECISIONS-back-reference framing).

**BRAIN15.28 Source: line updated** in `multi-agent/plans/PHASE15_BRAINSTORM.md`. Changed *"Round 1 follow-up"* → *"Round 2 follow-up"* in the FEEDBACK-section reference.

**BRAIN15.29 Source: line verified** — already correctly labeled *"Round 2 follow-up"*. The auditor flagged 27 and 28 only; 29 was authored after the boilerplate-copy mistake had been made (so 29 didn't inherit the bug). No change to BRAIN15.29 needed.

**No other changes** — per the narrow-fix rule, only the two flagged Source lines were touched. Coverage check, BRAIN entry count, splits/consolidations approval state, SOUP body integrity, all unchanged from the post-Round-2 state.

### Verification

```
$ grep -nE "Round [12] follow-up" multi-agent/plans/PHASE15_BRAINSTORM.md
# All three entries (BRAIN15.27 / BRAIN15.28 / BRAIN15.29) now read "Round 2 follow-up".
```

The narrow auditor finding is fully addressed. Mechanical-coverage state unchanged: 21 SOUP IDs cited; 29 BRAIN IDs sequential; 6 user-introduced no-SOUP-source entries (24/25/26/27/28/29); 2 RESOLVED markers (13/15); SOUP body untouched since the labeling pass.

### Updated transfer-status block (post-Round-2.5)

**Status:** **CLEAN — ready for narrow re-audit on Source-line provenance for BRAIN15.27 / BRAIN15.28.** The auditor's recommended re-audit scope is narrow (just the two flagged Source lines); if the re-audit clears, orchestrator can proceed to the CLOSEOUT prompt and SOUP archival.

**Mechanical coverage:** All 21 SOUP IDs (SOUP15.1 through SOUP15.21) cited in BRAIN entry `Source:` fields. 29 BRAIN IDs sequential 1-29. No gaps.

**BRAIN entry count:** **29** BRAIN IDs — unchanged from post-Round-2.

### Next steps (post-Round-2.5)

1. Orchestrator invokes the **soup-to-brainstorm AUDITOR re-audit prompt** (Agent 2) with narrow scope per the auditor's recommendation: verify BRAIN15.27 / BRAIN15.28 / BRAIN15.29 Source lines all read "Round 2 follow-up"; verify no other unintended changes.
2. When auditor declares CLEAN, orchestrator invokes the **CLOSEOUT prompt** to retire the SOUP file via `git mv multi-agent/plans/PHASE15_HMM_SOUP.md multi-agent/plans/archived/20260428-PHASE15_HMM_SOUP.md`. Brainstorming stage begins.
3. Optional follow-on cleanup work flagged for the brainstorming stage or a separate cleanup pass (unchanged from post-Round-2):
   - Update `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` IBM-C7B entry to mark RESOLVED with closeout pointer to 6.1 + BRAIN15.27.
   - Update `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` IBM-C6C entry to mark RESOLVED-via-promotion-to-BRAIN15.29.
   - Update `ROADMAP.md` Priority 6.3 from ✗ DEFERRED to ✓ DONE per the DECISIONS.md 2026-04-28 entry.
   - Update `ROADMAP.md` Priority 5.0.2 from ✗ DEFERRED to ✓ PROMOTED-to-Phase-15 (or mark resolved when BRAIN15.29 lands).
   - All append-friendly cleanup items; not blocking the auditor re-audit.

---

## Auditor findings — round 2

- **Verdict:** `CLEAN — transfer ready for archive`

- **Per-criterion findings:**
  1. ✓ **SOUP ID coverage:** Unchanged from round 1; all 21 SOUP IDs remain covered by BRAIN Source fields and the BRAINSTORM coverage check.
  2. ✓ **BRAIN ID assignment:** Unchanged from round 1; `BRAIN15.1` through `BRAIN15.29` remain sequential and gap-free.
  3. ✓ **Source provenance citations:** The round 1 provenance-label finding is fixed. `PHASE15_BRAINSTORM.md:765`, `:855`, and `:902` all now cite `Round 2 follow-up`; grep found no remaining `Round 1 follow-up` provenance in BRAINSTORM.
  4. ✓ **No silent scope narrowing:** No new narrowing introduced by the narrow fix.
  5. ✓ **No silent scope broadening / new-idea injection:** No new BRAIN entries introduced by the narrow fix; the six no-SOUP-source entries remain user-approved and logged.
  6. ✓ **Splits / merges:** Unchanged from round 1; documented and approved.
  7. ✓ **SOUP body integrity:** Unchanged from round 1; SOUP remains untouched after the labeling pass.
  8. ✓ **Open FEEDBACK questions:** No open transfer-blocking questions remain. The outstanding cleanup list remains explicitly optional / not blocking closeout.
  9. ✓ **Transfer-status block:** Round 2.5 status correctly requests narrow re-audit; this round 2 audit now clears that request.
  10. ✓ **General formatting / convention compliance:** The stale Source-line references are corrected; no remaining formatting/convention blocker found in the narrow re-audit scope.

- **Specific issues:** None. The two round 1 issues are resolved:
  1. `PHASE15_BRAINSTORM.md:765` now says BRAIN15.27 is logged in `PHASE15_FEEDBACK.md` Round 2 follow-up.
  2. `PHASE15_BRAINSTORM.md:855` now says BRAIN15.28 is logged in `PHASE15_FEEDBACK.md` Round 2 follow-up.

- **Recommendations:**
  1. Orchestrator can proceed with the soup-to-brainstorm CLOSEOUT prompt.
  2. Archive the live SOUP via:
     `git mv multi-agent/plans/PHASE15_HMM_SOUP.md multi-agent/plans/archived/20260428-PHASE15_HMM_SOUP.md`
  3. After closeout, begin the official Phase 15 brainstorming-stage iteration.

---

## Soup-to-brainstorm transfer CLOSED — 20260428

**Auditor verdict:** **CLEAN.** Issued by Codex (Agent 2) at the narrow re-audit pass that followed Round 2.5's source-line fix. The Round 1 audit had returned OPEN with one narrow finding (BRAIN15.27 / BRAIN15.28 Source-line provenance mislabeling); Round 2.5 corrected the labels; the re-audit confirmed the fix and cleared the transfer.

**SOUP archival (proposed `git mv`, orchestrator runs):**

```
git mv multi-agent/plans/PHASE15_HMM_SOUP.md multi-agent/plans/archived/20260428-PHASE15_HMM_SOUP.md
```

The archive filename carries both lineages per the convention: `PHASE15_` for the phase identity, `HMM_` for the SOUP's `next/`-era theme, `20260428` for the archival date.

**Brainstorming stage now begins.** BRAINSTORM is open for expansion per the standard "Brainstorm Stage" rules in `multi-agent/workflows/phase-development-system_PDS-v2.md`:

- Add deeper analysis to existing BRAIN entries.
- Source new ideas from `multi-agent/tracking/BRAINSTORM.md`, `multi-agent/tracking/KNOWN_ISSUES.md`, and `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` (the user's Q10 note explicitly anticipates this brainstorming-stage deep-dive).
- Run BRAIN15.24's `peak_rcn_stage` term audit as a brainstorming-stage activity per Q2 user direction.
- Expand BRAIN15.18 sub-priorities (d) and (e) (multistage unification + state-path evolution) per the user's Q9 note that further enrichment from tracking files is brainstorming-stage work.
- Author IBM-C5 cross-pipeline dynamic origin-detection design sketch under BRAIN15.22 per Q3.
- Continue iterating FEEDBACK Q&A as new questions arise.

**BRAIN ID-count baseline at closeout:**

> **BRAINSTORM contains 29 BRAIN IDs** (BRAIN15.1 through BRAIN15.29) covering all **21 SOUP IDs** (SOUP15.1 through SOUP15.21) — 21 transcribed-from-SOUP entries (with 3 splits and 3 consolidations across SOUP → BRAINSTORM, all approved Round 1) + **6 user-introduced no-SOUP-source entries** (BRAIN15.24 `peak_rcn_stage` audit; BRAIN15.25 HMM 2-pass background renorm; BRAIN15.26 chrom-median default switch; BRAIN15.27 composite APS feature modes + 3-layer PCA; BRAIN15.28 `summit_rcn` rename + RCN migration; BRAIN15.29 summit↔timing one-pass convergence — IBM-C6C promotion) + **2 RESOLVED markers** (BRAIN15.13 `--hmm-smooth-halfwidth` APS split — resolved 14S.23; BRAIN15.15 cross-pipeline dedup — resolved 14-S20).

This is the baseline count for tracking BRAINSTORM evolution during the official brainstorming stage. Brainstorming-stage adds typically increment the count past 29 as new ideas land or existing ones split.

**Cycle artifacts (recap):**

- `multi-agent/plans/PHASE15_BRAINSTORM.md` — the BRAINSTORM with all 29 BRAIN entries.
- `multi-agent/plans/PHASE15_FEEDBACK.md` — this file; full Q&A trail through Round 1 + Round 1 follow-up + Round 2 + Round 2.5 + auditor pass + re-audit + this closeout entry.
- `multi-agent/plans/archived/20260428-PHASE15_HMM_SOUP.md` — the archived SOUP (after orchestrator runs the proposed `git mv`).
- `multi-agent/project_context/DECISIONS.md` 2026-04-28 entry — APS feature mode candidates accepted/rejected catalog + RCN-vs-excess design decision.
- `multi-agent/plans/next/APS_SOUP.md` — new SOUP authored mid-process for deferred APS items + future-phase candidates.
- `multi-agent/plans/next/UNIFIED-RCN_SOUP.md` — new SOUP authored mid-process to preserve the long-horizon unified-RCN-intermediates idea.

**Hand-off to the orchestrator:** the next stage is the official **brainstorming stage**, invoked via the "after soup-to-brainstorm transition" opening brainstorm prompt in `multi-agent/workflows/phase-development-system_PDS-v2.md`. BRAIN15.24's `peak_rcn_stage` audit is a natural first activity per Q2 user direction.

---

## Round 3 — Brainstorming-stage aggregation pass — Agent 1 (2026-04-28)

### Status

First brainstorming-stage pass after the soup-to-brainstorm transfer closed at v0.14.76.2. Per user direction (chat 2026-04-28: *"first finish populating BRAINSTORM with all the ideas, then start working on individual ideas"*), this round did a deep-dive aggregation pass on `tracking/BRAINSTORM.md` + `tracking/KNOWN_ISSUES.md` + `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md` + a code-base audit (`onionskin_core/`, `onionskin.py`) + a `scripts/` + `tests/` audit. Findings below: (a) major code-finding for BRAIN15.24; (b) cross-tier `Source:` enrichment proposals for existing entries; (c) 3 new BRAIN entries proposed for promotion; (d) IBM promotion candidates; (e) side-quest awareness items; (f) new open questions Q15–Q22.

### MAJOR FINDING — BRAIN15.24 audit step 1 partially complete

The aggregation pass found **two existing columns in `onionskin_core/aps.py` that implement closely-related (but not identical) versions of the `peak_rcn_stage` concept**: `max_rcn_stage` (argmax-based) and `regression_stage` (first-post-peak-confirmed-dip). Full code-finding write-up has been added to BRAIN15.24's body under the heading *"Audit step 1 partial-result — found in aggregation pass (2026-04-28, brainstorming-stage Round 3)"*. Summary:

- `max_rcn_stage` ([onionskin_core/aps.py:1016](onionskin_core/aps.py#L1016)): "Post-onset stage at which the summit-window median RCN is highest." Implemented at v0.5.49 from a planning note that proposed the name `peak_rcn_stage` (which became `max_rcn_stage` in code).
- `regression_stage` ([onionskin_core/aps.py:1017](onionskin_core/aps.py#L1017)): "Stage s+1 of the first confirmed dip at or after max_rcn_stage."

User's recalled semantics for `peak_rcn_stage` ("last stage at which summit growth is detected w.r.t. previous stage; treats slight dips as still flat") is a third distinct semantic that doesn't exactly match either existing column. Provisional case classification: **case (b) Partial match — needs correction or refinement OR case (c) need-a-new-column.** The user-walkthrough portion of the audit is what resolves this. See Q15 below.

### Cross-tier `Source:` citation enrichment proposals (existing BRAIN entries)

Per Q10 user note, this is brainstorming-stage work. Proposing citations to add to existing BRAIN entries' `Source:` fields. **None of these are speculative — each citation is a tracking-file or code reference that legitimately informs the BRAIN entry's scope.** I have NOT applied these yet (other than the BRAIN15.24 finding above) — listing them for your review-and-batch-approve.

| BRAIN entry | Proposed citations to add |
|---|---|
| BRAIN15.7 (amplicon reliability + flat-sample) | `tracking/BRAINSTORM.md [2026-04-01] APS locus diagnostics — design discussion and open questions` (extensive discussion of post_support / dip_rate / oscillation annotation / MAD tracks / calibration amplicons); `tracking/BRAINSTORM.md [2026-04-07] Keep/exclude recommendation enhancements` (graduated gap thresholds / shape-score proximity / timing-flag combinations); IBM-C8B (Stage-1 pre-amp principled approach); IBM-C12 (APS locus diagnostics); IBM-C14 (Keep/exclude recommendation enhancements) |
| BRAIN15.8 (HMM peak_rcn_stage + sliding-offset) | `tracking/BRAINSTORM.md [2026-04-01] APS locus diagnostics` (confirms summit-RCN reasoning); `tracking/BRAINSTORM.md [2026-04-07] Post-hoc summit refinement (--refine-summits)` (related summit refinement strategy) |
| BRAIN15.12 (--hmm-0-based-statepath) | `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-28:2]` (user's Q1-follow-up note: 0-based should be DEFAULT, legacy 1-based via `--hmm-1-based-statepath`; equality-of-results regression test design) |
| BRAIN15.17 (APS analysis-surface parity + APS catalog) | `tracking/BRAINSTORM.md [2026-04-09] HMM fork travel and analysis plots — review COMPLETE` (HMM-side plot review with concrete per-plot decisions); `tracking/BRAINSTORM.md [2026-04-09] Fork age tracking — new visualization concept` (HMM-specific plot design); `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-28:1]` (plots and notebooks should be the last directories convention) |
| BRAIN15.18 (HMM shape-score + multistage unification) | `tracking/BRAINSTORM.md [2026-04-07] Multi-engine integration and HMM meta-analysis vision` (broader meta-analysis context); `tracking/BRAINSTORM.md [2026-04-09] HMM right-side asymmetry bias — artifact vs. biology` (HMM-specific Viterbi-direction issue worth flagging in fork-travel completeness work) |
| BRAIN15.24 (peak_rcn_stage audit) | `tracking/BRAINSTORM.md [2026-04-01] APS locus diagnostics` (planning note that proposed the column name; v0.5.49 implementation context); code findings (`onionskin_core/aps.py:976-1017`, `onionskin_core/timing.py:312, 408`, `onionskin_core/summit_plots.py:200`). **Already applied to the BRAIN15.24 body** (see MAJOR FINDING above). |
| BRAIN15.25 (HMM 2-pass background renorm) | `tracking/BRAINSTORM.md [2026-04-11] Shared filtering semantics across pipelines` (chrom-median normalization rationale; no-ratio HMM branch; denominator considerations) |
| BRAIN15.26 (chrom-median default switch) | `tracking/BRAINSTORM.md [2026-04-11] Shared filtering semantics across pipelines` (same as 15.25 — also relevant here) |
| BRAIN15.27 (composite multi-feature APS modes) | `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-22:1] --aps-rank-by future choices` (summit/width/shape ranking — interconnects with composite features); `tracking/BRAINSTORM.md [2026-04-01] APS locus diagnostics` (feeds the feature-space discussion implicitly via summit_rcn / shape_score_raw) |
| BRAIN15.28 (summit_rcn rename) | `tracking/BRAINSTORM.md [2026-04-01] APS locus diagnostics` (extensive summit-RCN-vs-area-excess reasoning that reinforces the BRAIN15.28 + DECISIONS.md framing) |
| BRAIN15.29 (summit↔timing convergence) | `tracking/BRAINSTORM.md [2026-04-01] APS locus diagnostics` (onset-stage detection logic + "stage 1 growth IS detectable" reasoning that's central to BRAIN15.29) |

### 3 NEW BRAIN ENTRIES PROPOSED — needing user approval before authoring

These are concrete deliverables surfaced from the aggregation pass. Each has: (a) source provenance, (b) HMM-completeness rationale, (c) substantive design content already in tracking files, (d) clear scope. They satisfy the substantive-priorities-v2 rule; each justifies a full audit-implement-reaudit cycle.

#### Proposed BRAIN15.30 — HMM sequence filtering CLI flags (`--chromosomes` passthrough + `--min-seq-length` + `--min-bin-count-per-seq`)

- **Source:** `tracking/BRAINSTORM.md [2026-04-09] HMM sequence filtering — CLI flags`; `tracking/BRAINSTORM.md [2026-04-11] Shared filtering semantics across pipelines` (cross-pipeline harmonization framing).
- **Goal:** Honor `--chromosomes` in the HMM pipeline (currently bug — flag exists in CLI but not passed through to HMM engine); add `--min-seq-length INT` (default 50000 bp) and `--min-bin-count-per-seq INT` (default 10) to filter out small associated contigs that clutter the HMM level-emergence heatmap. Plus chromosome-level heatmap variants when `--chromosomes` is specified.
- **HMM-completeness rationale:** Direct HMM-pipeline gap; affects all downstream HMM analyses (heatmaps, fork travel plots, etc.).
- **Cross-pipeline implication:** Per `[2026-04-11] Shared filtering semantics`, these flags should be controller-owned and apply to all pipelines — not HMM-only. If we wire HMM, we should also confirm growth/RMS honor the same flags consistently.
- **Substantive enough:** Yes — bug fix + 2 new flags + heatmap variants + cross-pipeline harmonization audit.

#### Proposed BRAIN15.31 — `--peak-summary` extension to RMS + HMM + max-projection variants (raw / normalized / smoothed)

- **Source:** `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-28:3] Max projection RCN profiles`; current `--peak-summary {quantile,topk,max}` flag in `onionskin.py` whose help-string already says *"Currently applied only by the growth pipeline; planned for RMS and HMM in a future phase."*
- **Goal:** Extend `--peak-summary` to RMS and HMM. Plus add raw/normalized/smoothed variants when `--peak-summary max` is chosen (raw max projection per the original goal; chromosome-specific median-normalized; median-or-trimmed-mean smoothed). Consider `--peak-summary all` option (= quantile + topk + max with all variants). Decide whether operations apply only to amplicon intervals or to the whole genome.
- **HMM-completeness rationale:** The flag's own help-string explicitly schedules HMM/RMS extension as future-phase work — Phase 15 IS that future phase per the HMM-completeness theme.
- **Substantive enough:** Yes — multi-pipeline wiring + new variant computations + design decisions on per-amplicon vs whole-genome scope.

#### Proposed BRAIN15.32 — HMM fork age tracking visualization

- **Source:** `tracking/BRAINSTORM.md [2026-04-09] Fork age tracking — new visualization concept`. Substantial design already exists.
- **Goal:** Implement the fork-age-tracking plot per the [2026-04-09] design: re-index nesting levels by firing-age (oldest = age 0) instead of by rank-within-stage. Renders the SAME biological fork set across stages as a continuous trajectory (with gaps where the fork age isn't yet present in earlier stages). Filename `{trajectory_id}.fork_age_trajectory.png` to coexist with existing `fork_travel_trajectory.png`.
- **HMM-completeness rationale:** HMM-native visualization (only HMM has the per-stage nesting-level data with stable indexing). Falls under the HMM-specific analyses bucket of the Phase 15 theme.
- **Note from the [2026-04-09] design entry:** *"Do NOT assign to Copilot without a more detailed spec (write that spec in a design session with the user first)."* — flagging that BRAIN-stage authoring should expand on the design before SPEC engineering.
- **Substantive enough:** Yes — new visualization with non-trivial level→age re-indexing transform and rendering rules.

### IBM promotion candidates (no FEEDBACK-Q-style "should we promote" verdict from prior round, but surfaced here)

The following IBM items were not promoted in Q11 / Q12, but the aggregation pass found their content overlaps significantly with existing BRAIN entries (mostly BRAIN15.7). Asking whether to formally fold their content into the relevant BRAIN entry (no new BRAIN entry needed) or keep deferred:

- **IBM-C8B Stage-1 pre-amp isolation principled approach** — overlaps BRAIN15.7. The IBM file's design content is more detailed than BRAIN15.7 currently captures. Recommendation: fold the IBM-C8B content into BRAIN15.7's body during a brainstorming-stage expansion pass.
- **IBM-C12 APS locus diagnostics** — overlaps BRAIN15.7. The detailed [2026-04-01] design discussion is the strongest existing source for the per-locus reliability framework. Recommendation: fold into BRAIN15.7.
- **IBM-C14 Keep/exclude recommendation enhancements** — partially overlaps BRAIN15.7 (graduated gap thresholds, timing flags); also touches BRAIN15.18 indirectly. Recommendation: fold the relevant pieces into BRAIN15.7.

These would not require BRAIN15.7 to gain new BRAIN IDs; just expand the existing entry's body to incorporate the deeper design content. See Q16 for explicit user approval.

### Side-quest awareness items (NOT formal BRAIN entries — flagged here so they don't get lost)

These items are tangentially relevant but not Phase 15 priorities:

- `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-28:4] summit_inspector.py ModuleNotFoundError` — broken script; user-flagged as important. Could be fixed as a tooling side-quest within Phase 15 implementation cycles, or as a standalone DEVLOG-routed cleanup. **Not a Phase 15 priority but worth flagging since `summit_inspector.py` is HMM-relevant tooling.**
- `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-18:2] early-parabola-mean` — RMS-side runtime selector tuning; tangentially relates to BRAIN15.8 (HMM summit refinement). Probably stays RMS-specific.
- `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-18:4] RMS chr-II by-eye eval surface` — RMS-evaluation infrastructure. The analogous question for HMM ("does HMM have a full chr-II by-eye eval surface?") is worth raising during BRAIN15.17 catalog work. See Q17.
- `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-24:2] RMS summit parabola pre-smoothing` — RMS-specific; analogous question for HMM falls under BRAIN15.8.
- `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-25:1] --onset-* vs --timing-* prefix harmonization` — CLI naming cleanup; not Phase 15 priority.
- `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md IBM-C1 Internal-code Stage-1/Stage-2 → pass-1/pass-2 terminology` — internal label cleanup deferred from Phase 14. The prompt's "audit scripts/ for filepaths and other things that would need to change" makes this a valid sweep candidate. Not a Phase 15 priority but worth flagging.
- `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md IBM-C10 Robustness dataset expansion` — user-gated; left deferred.
- `tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md IBM-C11 Cross-pipeline synthesis / 04-unified-results/` — planned future phase; not Phase 15.

### HMM-completeness gap audit (vs growth/RMS)

Comparing HMM step inventory ([onionskin_core/output_layout.py:281-301](onionskin_core/output_layout.py#L281-L301)) against growth-model and RMS step inventories. Step gaps where HMM lacks an analog:

| Gap | Growth has | RMS has | HMM has? | Phase 15 BRAIN coverage |
|---|---|---|---|---|
| Dedicated **shape-filter step** | `02-growth-model/03-shape-filter/` | `03-rcn-mean-shift/06-shape-filter/` | NO (shape filter not wired for HMM) | BRAIN15.18 (HMM shape-score wiring) — covered |
| **Signal-tracks / peak-summary step** | `02-growth-model/07-signal-tracks/` | (planned per `--peak-summary` help) | NO | Proposed BRAIN15.31 (this Round 3) — pending approval |
| **Multistage unification step** | (overlap-resolution at step 11) | `03-rcn-mean-shift/08-multistage-unification/` | NO (parts in HMM steps 7-8 collapsedHMM + aboveBackground but no unified step) | BRAIN15.18 — covered |
| **Gap-analysis post-call step** | `02-growth-model/12-gap-analysis/` | `03-rcn-mean-shift/09-gap-analysis/` | NO (HMM has only step-3 removeZeroBins) | BRAIN15.21 — covered |
| **Plots step (dedicated)** | `02-growth-model/14-plots/` (with sub-dirs aps/, growth_curves/, genome_overview/) | `03-rcn-mean-shift/11-plots/` | NO (only `notebooks/` subdir under HMM) | BRAIN15.17 — partially covered; see Q18 |
| **Width-progression step** | `02-growth-model/05-progression/` | NO | NO | Not in any Phase 15 BRAIN entry — see Q19 |

**Implication:** BRAIN15.5 (HMM completeness matrix umbrella) should explicitly enumerate these gaps as the matrix items it tracks. Recommendation: brainstorming-stage expansion pass on BRAIN15.5 to incorporate this gap inventory directly into its body.

### Open questions for the user (Q15–Q22)

#### Q15 — peak_rcn_stage audit step 2 (user-walkthrough): which semantic is desired?

Per the BRAIN15.24 finding above, three candidate semantics for `peak_rcn_stage` exist:

(a) **Argmax across stages (= existing `max_rcn_stage`):** "Stage where summit-window-median RCN is highest, post-onset."
(b) **Per-transition growth-based (user's tentative recall):** "Last stage where RCN_s > RCN_{s-1} (with 'still flat' tolerance for slight dips)."
(c) **First-confirmed-dip-based (= existing `regression_stage`):** "Stage just before the first confirmed-dip transition post-peak."

**Q:** Which semantic is the desired definition? Plus secondary: do we want (i) one column with the chosen semantic, (ii) two columns (keep `max_rcn_stage` AND add a new one), or (iii) three columns (keep `max_rcn_stage` AND `regression_stage` AND add a new `last_growth_stage` for the per-transition-growth semantic)? And: what should the canonical name be — keep `max_rcn_stage` for the argmax variant, or rename to `peak_rcn_stage` to reflect the planning-surface intent?

The audit's user-walkthrough step (BRAIN15.24 step 2) is best done by walking through one or two example amplicons in DS1 chr II using existing `max_rcn_stage` and `regression_stage` outputs alongside the user's recalled-semantic computation. Once you settle the semantics, the audit's case-(b) "correction-and-propagate" plan or case-(c) "implement-from-scratch" plan unblocks; BRAIN15.8 (HMM adoption) is then unblocked too.

**A:**

It is okay to keep max_rcn_stage as argmax, but what is important is where and how it is used. I believe one agent was using max_rcn_stage as if it was the last active stage of growth, which is likely to be the case, but not guaranteed. "regression_stage" is actually also an attempt to figure out the last stage of active growth at the summit. There is a phenomenon where the amplicon height appears to shrink in later stages. This is not necessarily biologically interesting. It can be a technical issue that arises as amplicons get wider and wider affecting various median normalization steps -- it would be interesting if the second pass background estimation fixes this issue. It can also be related to a nucleases that begin breaking down the salivary glands in the later stages. Loci would be attacked by nucleases in proportion to their copy number -- if there are 30 kings in a deck of cards, and only 1 of everything else, the copy number of kings is going to dimish at a fast rate when sampling without replacement; same here. That could be happening toward the latest stages. At that point, there definitely is no high quality summit information. So when regression is stably detected -- meaning when we can see that it went lower and stayed lower or kept going lower, those are stages to ignore. The summit can only be reliably analyzed when the origin is firing, which happens in earlier stages - but has to be defined for each amplicon separately. So the concept was max_rcn_stage should be the stage before the regression starts, but this is not guaranteed, especially if the sames are grouped poorly in the manifest, although that is more of a user problem than ours. And "regression_stage" - first confirmed dip - was another way to estimate when growth stopped. The concept I had was the following: given an approximate (or accurate) summit position +/- X kb, do we dectect growth in Stage S, for S in 1-N (when there are N stages). Stage 1 growth (or stage 2 when 1 is the ref-stage) is detected relative to background since there is no prior stage. Stages 2-N use some statistical test to compare stage S to stage S-1 to see if there was growth relative to the previous stage. At some point we did come up with a bunch of tests for this -- they might be in the timing module. The first active stage, or earliest active stage, or onset_stage -- is the first stage growth / activity is detected at the summit region. The "last active stage" -- we might have had a different name -- is the last stage growth is detected. This is not necessarily the same as the stage before the regression_stage because there could be stages where "no difference" is detected -- i.e. a flat period before regression. It is also not necessarily the same as max_rcn_stage because an outlier can cause it to be too early... or the flat period might screw it up. For comparing stage S to S-1, it does not need to be a ratio between them... we can just look at the series of summit RCN values from stage 1 to N. We already do growth curves - we can look at the input data for the growth curve plots. We could fit a loess line or something so a random oscillation (high outlier) does not necessarily screw up the analysis. Just riffing here.
- To answer this: I believe we have 4 different metrics of interest for defiing the active window: onset_stage, last_active_stage (ot some other name), max_rcn_stage, regression_stage. There is no need to get rid of any. It just depends how and where we are using them. Each could give information we want.
- I believe some of what I said above is desribed as the "origin detection window (ODW)" in KNOWN_ISSUES.md; for example, see [ISSUE:2026-04-18:3] and [ISSUE:2026-04-18:2]

#### Q16 — IBM-C8B / IBM-C12 / IBM-C14 fold into BRAIN15.7?

Three IBM items overlap with BRAIN15.7's amplicon-reliability scope at design-content level. Each has detailed design notes that BRAIN15.7 currently doesn't fully incorporate.

**Q:** Approve folding IBM-C8B + IBM-C12 + IBM-C14 design content into BRAIN15.7's body during a brainstorming-stage expansion pass? (No new BRAIN IDs; just deepen BRAIN15.7's body with the IBM-file content + tracking/BRAINSTORM.md [2026-04-01] details.) Default = yes per the substantive-priorities rule.

**A:**
- yes, absolutely merge the ideas and enrich them with all prior planning notes. try to use verbatim notes when possible to avoid drifting away from the ideas, especially notes written by the human, which are least likely to contain drift. if and when drift is detected, the human can give guidance on how to proceed back toward the target.

#### Q17 — HMM full chr-II by-eye eval surface (analogous to RMS [ISSUE:2026-04-18:4])?

RMS has a known issue about needing a full chr-II by-eye interval evaluation surface (not the lightweight summit-run proxy). The analogous question for HMM: does HMM have a full chr-II by-eye eval surface? If not, should Phase 15 produce one as part of the HMM-completeness work?

**Q:** Should Phase 15 include "build HMM full chr-II by-eye eval surface" as a deliverable (folded into BRAIN15.17 APS analysis-surface parity, or as its own BRAIN entry)? Or defer?

**A:**
- I do not totally understand the quesiton, but give you information I believe will help answer it anyway.
- we can certainly use the "by eye" file `tests/full_chrom_training_data/amplicons.by-eye.bed` for evaluations
- we can and should also use the PuffStep results - various results here: `dev/puffstep/automate/`
- we can and certainly should use our various summit evaluation tests for II/9A as well, codified in various make tests, for example, and all based on this file: `tests/summit_training_data/01-II9A-evidence-based-confinement-boundaries-and-rules.bed`, and to a much lesser extent this file `tests/summit_training_data/02-II2B-evidence-based-confinement-boundaries-and-rules.bed`.
- so yes - we can use the machinery and files already in place to evaluate HMM performance. We have made a ton of tests in that direction. There may or may not need to be some adaptations to HMM run development. I will let you audit the code for that.

#### Q18 — HMM dedicated plots/ step directory

HMM currently has only a `notebooks/` subdir; growth has `14-plots/` with sub-dirs (aps/, growth_curves/, genome_overview/) and RMS has `11-plots/`. Should Phase 15 add a dedicated `<step>-plots/` directory under HMM (e.g., `01-hmm/17-plots/` after the renumbering for SAPS in BRAIN15.6)?

**Q:** Approve adding HMM `<step>-plots/` directory as part of BRAIN15.17's deliverable scope? (Folded into BRAIN15.17 — no new BRAIN entry needed.) Default = yes.

**A:**
- HMM pipeline should absolutely get a `plots` directory for the "completeness" goal, but see KNOWN_ISSUES.md [ISSUE:2026-04-28:1]
- all pipelines should remove the step from the plots and notebooks directories because as development continues, and steps are added or subtracted, the plots and notebooks directories can sit in the wrong place... so might as well put them outside of the steps, and allow plots and notebooks to be written to them at any time. And as new steps or analyses are added, they can just write to those "step-less" directories as needed.

#### Q19 — HMM width-progression step (analogous to growth's `05-progression/`)?

Growth has `02-growth-model/05-progression/` (width progression TSVs). HMM doesn't have an analogous step. The growth-pipeline progression step is tied to the multistage-growth fits; HMM-equivalent would be derivable from steps 7-12 (collapsedHMM + aboveBackground + summitStates + summitBins + amplicon-metrics + fork-travel) but isn't currently emitted as its own progression file.

**Q:** Should Phase 15 produce an HMM-equivalent width-progression output (via folding into BRAIN15.5 HMM completeness matrix or a new BRAIN entry)? Or is this not a real gap because HMM's per-amplicon metrics + fork-travel already capture the same information differently?

**A:**
- Yes, Phase 15 should do this
- I believe we already developed a lot of stuff like this using the state path
- certainly there are plots that track fork progression and fork age
- maybe some of that is in "12-fork-travel" or "15-timing"
- maybe we need new step directories for some of this
- In addition to HMM-specific state path derived metrics, I think we can and should try to also provide the same analogous approach as Growth. The idea would not be to do it only the novel ways afforded by the HMM, but also any other way that is possible for the HMM pipeline to make it self sufficient when run alone.
- So yes, we can do width progression from the state path
    - that is probably most reliable
    - we can track progression of the oldest forks (leading edge of state 1), but with the HMM steps, we can track forks from each doubling round (as we have done with some plots already)
- But also, if the way it is done in Growth would produce meaningfully different results, then that approach could also be included in the HMM pipeline, assuming the HMM pipeline has the analogous input files or can easily produce them (like stage RCNs or individual sample RCNs)

#### Q20 — Approve the 3 proposed new BRAIN entries (15.30, 15.31, 15.32)?

Per the proposals above. Each is substantive enough to justify a full audit-implement-reaudit cycle and has tracking-file design content already.

**Q:** Approve as new Phase 15 BRAIN entries? Yes / No / Partial (specify which) / Modify (specify how).

**A:**

BRAIN15.30 - this is supposed to already be implemented. The --chromosomes flag has always worked on the HMM pipeline, and `--min-seq-length` + `--min-bin-count-per-seq` were originally ONLY in the HMM pipeline and had to be ported out to generalize to all pipelines, if I recall correctly. At minimum, they were developed during HMM pipeline development, and for the HMM pipeline, but may have always applied to all 3 on purpose -- the flags used to be incorrectly grouped with HMM flags though. So yes - BRAIN15.30 has me thoroughly confused. In fact, the claude.ai server went down and severed communication with us, but now I am wondering if you bugged out during this last phase of aggregating ideas and updating the FEEDBACK file. That is how shocking this proposal is to me. It should all be already in the code.

Yes to BRAIN15.31 - I added [ISSUE:2026-04-28:3] to known issues today and intended for it to be part of Phase 15. Yes we should extend this out to all 3 pipelines. Whatever the issue said in KNOWN_ISSUES is fodder for including in Phase 15.

BRAIN15.32 may also already be implemented. I am starting to see the pattern that BRAINSTORM.md may be stale and need an audit and clean up. Nonetheless, BRAIN15.32 is a very important topic - it just might already be done. Certainly we have some version of it already. So it is just a matter of whether this is proposing something new or a fix to it or making it better. Needs a code audit.


Yes to all "IBM promotion candidates" as far as I can tell -- the section below those BRAIN proposals.


#### Q21 — Apply the citation enrichment proposals?

The table above lists `Source:` field additions for 10 existing BRAIN entries. Each citation is a tracking-file or code reference that legitimately informs the BRAIN entry's scope.

**Q:** Approve applying the citation additions in a single batch? Default = yes.

**A:**
- Adding source and context information and merging ideas from various files into BRAINSTORM is what this stage is all about. Absolutely "yes to all" for questions like this. 

#### Q22 — Brainstorming-stage expansion pass on BRAIN15.5 + BRAIN15.7 + BRAIN15.17

Three existing BRAIN entries would benefit from a body-expansion pass during brainstorming stage:

- **BRAIN15.5** (HMM completeness matrix): incorporate the gap-audit table from this Round 3 finding directly into its body. Currently BRAIN15.5's analysis-families list is a flat enumeration; the gap-audit table would make the matrix concrete.
- **BRAIN15.7** (amplicon reliability): incorporate IBM-C8B + IBM-C12 + IBM-C14 design content (subject to Q16 approval).
- **BRAIN15.17** (APS plots/parity): incorporate the [2026-04-09] HMM plot-review decisions + the [2026-04-09] fork-age-tracking design + the dedicated plots/ step decision (subject to Q18 + the proposed BRAIN15.32).

**Q:** Approve the body-expansion pass on these three entries during the next brainstorming-stage round? (Comes after Q15-Q21 answers.)

**A:**
Yes to all -- the brainstorming stage is about aggregating information from across the files and making it non-redundant in this file so we can then make the SPEC with highly enriched non-redundant information for each idea. 

**Notes on KNOWN_ISSUES.md incorporation:**
- This may not be and exhaustive inclusion of everything relevant in KNOWN_ISSUES, but is here to help answer questions and point you in the right direction.
- [ISSUE:2026-04-28:2] is intended to be part of Phase 15
- As are: [ISSUE:2026-04-28:1], [ISSUE:2026-04-28:3], [ISSUE:2026-04-28:4]
- Also: [ISSUE:2026-04-18:3], [ISSUE:2026-04-18:2], [ISSUE:2026-04-14:1], [ISSUE:2026-04-18:1], [ISSUE:2026-04-19:1], [ISSUE:2026-04-19:2], [ISSUE:2026-04-25:1], [ISSUE:2026-04-26:1], [ISSUE:2026-04-21:1], 
- As was [ISSUE:2026-04-19:3], but it says RESOLVED - check the code!
- [ISSUE:2026-04-24:2] is RMS-specific, but in spirit is something we should pay attention to when computing parabola summits in the HMM pipeline. When designing it with the Growth pipeline, smoothing was important. And actually, the type of smoothing it does specifically. I recall we tried changing it to median smoothing, and the summit evaluation results got poorer so we reverted it back to mean smoothing or trimmed mean smoothing that it does -- check the growth code for this, describe it and compare it to how parabola summits are computed for the HMM pipeline.
- [ISSUE:2026-04-22:1] may be important for posterior ordering, but not necessarily posterior groups / posterior manifests. If that is the case, it can go in APS_SOUP.md. If it does affect posterior manifests, then it should be part of Phase 15.
    - We adding it to APS_SOUP.md, it can be removed from KNOWN_ISSUES.md as it found a planning home, OR it can be annotated as "Planned for APS_SOUP.md" and kept until it is closed out. However, that creates a maintenance burden. Perhaps it can be ported to APS_SOUP.md, deleted from KNOWN_ISSUES, but documented in its new home as "Formerly lived in KNOWN_ISSUES.md and was referred to as [ISSUE:2026-04-22:1]"
        - That last bit would allow us to track its history if ever needed, and would improve greppability of the entry in APS_SOUP.md
        - That behavior in general should probably be in AGENT_CONVENTIONS.md
            - move from known issues to new home, annotate new home with "Formerly lived in KNOWN_ISSUES.md and was referred to as <ID>", and deleted from known issues.


**Notes on BRAINSTORM.md incorporation:**
- This may not be and exhaustive inclusion of everything relevant in BRAINSTORM.md, but is here to help answer questions and point you in the right direction.
- The following are intended to be part of Phase 15
    - ## [2026-04-14] HMM summit refinement — sliding-offset sub-bin localization
    - ## [2026-04-18] Cross-pipeline origin detection windows — dynamic onset / last-active-stage summit refinement
        - with the main focus on the HMM pipeline, but where possible, wiring it into the other pipelines, or making it easy to do in the future.
        - this is an important one and has the "last active stage" concept I have been talking about, and that I confused with "max_rcn_stage" -- the ideas here expand upon and replace some of the ideas in:
            - ## [2026-04-14] HMM summit estimation — use stage of maximum summit RCN, not last stage
    - ## [2026-04-19] HMM parallel child pipeline (per-sample individual decoding) — HIGH PRIORITY
    - ## [2026-04-18] SAPS — State-APS from individual-sample HMM decoding
    - ## [2026-04-18] HMM-native amplicon quality criteria
    - ## [2026-04-19] Amplicon reliability scoring and flat-sample detection before APS/clustering
    - ## [2026-04-18] `--hmm-0-based-statepath` future CLI flag
        - but incorporated as described way above 
        - briefly: we should switch to defaulting to 0-based, and have a legacy mode for `--hmm-1-based-statepath`. The legacy mode could be used for all current make tests that depend on legacy mode, but another make test could ensure equality of output (except for different state labels) between 0-based and 1-based modes. There are certain flags that need to have defaults that depend on which mode is running, which is also discussed elsewhere.
    - ## [2026-04-09] HMM fork travel and analysis plots — review COMPLETE
        - It says complete, but make sure it is complete in the spirit of Phase 15 (e.g. compared to other pipelines, and using all its own HMM-specific outputs)
    - ## [2026-04-09] Fork age tracking — new visualization concept
        - As mentioned above, I think we actually did this. But if we did not, then it is included. Audit needed. I am open to improving upon what is present already.
    - ## [2026-04-09] HMM sequence filtering — CLI flags
        - This is almost certainly done -- I see this is where you found your proposal, and I am sorry you were misled. If it is not done, then yes, it belongs here. Audit needed. I suspect it is done though.
- If these ideas are not yet implemented, then we should consider including them in this Phase. Audits needed. We can wire them so other pipelines can easily inherit them. We can discuss HMM-specific variations on these themes too.
    - ## Priority 8.5.1 — Bidirectional fork asymmetry plots
    - ## Priority 8.5.2 — Fork gradient and velocity analysis
    - ## Priority 8.5.3 — Developmental fork trajectory visualization
- To lesser extents, the contents of these are concordant with Phase 15, but perhaps not necessarily part of it. We should note them as possibly needed or desirable at certain steps, like optimizing posterior clustering and posterior amplicon shapes and summits, etc.
    - ## [2026-04-18] Per-chromosome APS clustering bootstrapping experiments
    - ## [2026-04-18] APS_mean_of_means clustering test
    - ## [2026-04-09] HMM right-side asymmetry bias — artifact vs. biology
    - ## [2026-04-01] APS locus diagnostics — design discussion and open questions
    - ## [2026-04-07] Keep/exclude recommendation enhancements (from Priority 4.9)
        - but in the context of the HMM pipeline
- Feedback on some other BRAINSTORM ideas
    - ## [2026-04-14] HMM summit estimation — use stage of maximum summit RCN, not last stage
        - This might be a stale idea. 
            - It is worth implementing this and testing it explicitly in our summit evaluations
            - But overall I think the idea is basically analogous to saying lets model the summit in the "last active stage" which is correlated with "max_rcn_stage"
                - But actually the active stages prior to this are likely to have better summit signal or equally as good.
                - the newer idea would be to identify the "origin detection window (ODW)" which starts at the stage where summit growth is first detected and ends at the last stage summit growth is detected -- this are setting boundaries on stages where the origin is actively firing, and the summit is expected to be more pointy and peak-like. In stages after the ODW, the origin is no longer active or is much less active, and the "summit" region is extended by the last set of replication forks to something more like a flat table top over a broader region -- and where the origin was gets less and less obvious the longer this flat top is. The data may not look totally flat, but it is a matter of how relatively pointy and steep it is above the origin, which is more likely a property of earlier actively firing stages.
        - What it gets right:
            - "NOT the last stage or an average across all stages."
                - Correct: the summit is not likely to be best modeled by assuming it is best in the last stage or an average across all stages. We can use biological expectations on replication origin firing to know that we should be looking in that active window. In some datasets, the last stage (or the average) might be fine. In others, they won't be. 
                - But this actually also extends to the stage of maximum summit RCN
                    - Sometimes this will work if the max captured the pointiness well
                    - In others where the max had replication forks that traveled far, the signal:noise of origin location will be much less. Even if it is at the end of the ODW, other stages in the ODW might have better signal.
            - Interestingly, that issue goes on to explain the problem in a way that sounds correct:
                - "Empirically on ds2 chr II: stages 2-4 have summit bins that directly overlap the known origin region. Stage 4 summit from raw RCN is already shifted laterally. Stage 5 summits are shifted substantially further. This degradation is caused by **elongation interference**: in later stages, forks launched from the origin have traveled outward and their cumulative RCN contribution inflates bins flanking the origin more than the origin bin itself, shifting the parabola center toward flank signal rather than origin signal."
                - It describes stages 2-4 in the same way we would talk about the ODW
                - It shows that stage 5 (last stage) is not the best
                    - This stage is also likely to be highest summit though (from my familiarity with the data), so this observation actually contradicts the premise of the section
                - "Elongation interference" is a cool term. We should use it along with "Origin Detection Window (ODW)". Overall, this part explains the need for ODW well: elongation interference causes unexpected shifts away from the known origin location, and we therefore need to define the ODW and use only that.
            - "area_excess is wrong for stage selection (same logic as best_onset_stage)"
                - This section gets it right too: "Area grows monotonically as fork fronts advance — you cannot use area to identify the best stage. Summit RCN reflects origin-firing intensity and peaks at the stage of maximum initiation before elongation becomes dominant."
            - "A per-stage or best-stage summit strategy is needed."
                - or ODW determination, and from there a per-stage or best-stage strategy, or statistical strategy across the window.
        - For more background on ODW and these ideas, see:
            - ## [2026-04-18] Cross-pipeline origin detection windows — dynamic onset / last-active-stage summit refinement


**Notes on INTENDED-BUT-MISSED-PRIOR-TO-14.md incorporation:**
- Intended for Phase 15. Caveats and nuances I wrote around some of these ideas above still apply here, but I am not going to rewrite them.
    - IBM-C2
        - HMM parallel child pipeline + SAPS + step-14 APS raw-file fix 
        - yes, these were addressed above as well
    - IBM-C3
        - Amplicon reliability scoring + flat-sample detection before APS/clustering
        - yes, these concepts are important
        - Amplicon reliability scoring is the same in spirit to the multistage unification final shape classification in the RMS pipeline, which should be ported to the HMM pipeline, as well as the HMM-specific variant I described elsewhere that follows state path evolution.
    - IBM-C4
        - HMM summit accuracy — `peak_rcn_stage` column; sliding-offset sub-bin refinement for HMM
        - Yes - but with any applicable note or caveat or nuance or modification I noted above regarding these ideas
    - IBM-C5
        - Dynamic origin-detection window (cross-pipeline summit refinement)
        - Yes, this one is very important as described above in various places
    - IBM-C6C
        - Stage-activity fold-change using refined summit (timing.py refactor)
        - I believe I addressed some of this above (including answering some of your questions), so in whatever context I already wrote about it in...
    - IBM-C6D
        - Timing-guided active-stage weights for summit estimation
        - This sounds related to the Origin Detection Window (ODW) concept, so yes we should explore this and integrate good ideas surrounding this theme from this section or any section it arises in.
    - IBM-C6E
        - Iterative summit ↔ timing convergence (EM-like)
        - Sort of, but with the constraints I said above:
            - after multistage unification / amplicon reliability scoring
                - first best summits -> timing / ODW -> update summit -> update timing
                - thats it
    - IBM-C7A
        - APS dimensionality reduction / embedding (PCA, UMAP, diffusion)
        - We discussed what we wanted from this extensively in chat, and above
            - We defined 3 layers PCA can be applied
            - We deferred UMAP and diffusion
    - IBM-C7B
        - Shape-aware APS clustering (cluster on amplicon morphology)
        - We discussed this too 
        - Shape-aware APS clustering is implemented but we are now implementing amplicon-level and sample-level shape PCA clustering as also alluded to in my above comments for IBM-C7A
    - IBM-C8A
        - RCN floor removal experiment + `--aps-area-excess-floor on\|off` flag
        - Yes, this idea is about going back towards regular RCN, not area excess with a 0 floor or RCN with a 1-floor. I discussed it much elsewhere.
    - IBM-C8B
        - Stage-1 pre-amp isolation (principled "flat sample" detection in APS)
        - Yes as already discussed for KNOWN_ISSUES
    - IBM-C12
        - APS locus diagnostics (`best_onset_stage`, `post_support`, `dip_rate`)
        - Yes as far as it is related to ODW stuff and things we are planning, this sounds relevant
    - IBM-C13
        - Per-chromosome APS clustering bootstrapping + APS_mean_of_means clustering test
        - As mentioned above, this may come up as part of development, but we might not need to set it as an objective. We can update the mentions in tracking files if we do cover some or all of it.
    - IBM-C14
        - Keep/exclude recommendation enhancements (post-Priority 4.9)
        - Yes as far as we have discussed it above
        - This type of thing is always involving
            - It is essentially related to amplicon reliability and multistage unification as well as gap filtering and shape filtering in general.
            - all are concepts circling around keeping believable amplicons, and excluding suspicious ones (but documenting them anyway, and documenting why they are suspicious)


**IF THERE ARE ANY QUESTIONS I MISSED:**
- ask me in the chat
- update this file with my response so all agents can gain the context

### Updated transfer-status block (post-Round-3 aggregation pass)

**Status:** **Aggregation pass substantially complete; awaiting user answers to Q15–Q22 before applying citation enrichments + expanding entries + authoring the 3 proposed new BRAIN entries.** Brainstorming stage continues — this is iterative, not closing.

**Mechanical coverage:** Unchanged from post-closeout: 21 SOUP IDs cited, 29 BRAIN IDs sequential. After Q20 approval, BRAIN ID-count goes to 32 (15.30, 15.31, 15.32). After Q22 expansion pass, body content of 15.5/15.7/15.17 deepens but ID-count is unchanged.

**Files touched in Round 3 (pre-user-answer):**

- `multi-agent/plans/PHASE15_BRAINSTORM.md` — BRAIN15.24 body expanded with the audit-step-1-partial-result code findings (`max_rcn_stage` + `regression_stage` + holding-bucket synonymous-concept search).
- `multi-agent/plans/PHASE15_FEEDBACK.md` — this Round 3 section appended.

**Files NOT yet touched (waiting on Q15-Q22):**

- BRAIN15.7 / 15.8 / 15.12 / 15.17 / 15.18 / 15.25-29 Source: line additions (Q21).
- New BRAIN15.30 / 15.31 / 15.32 entries (Q20).
- BRAIN15.5 / 15.7 / 15.17 body-expansion pass (Q22).
- IBM-C8B / C12 / C14 fold-in (Q16).

### Next steps (post-Round-3)

1. User answers Q15–Q22.
2. Agent 1 applies approved actions in a Round 4 follow-up pass: citation enrichments (Q21), new BRAIN entries (Q20), body expansions (Q22), IBM fold-ins (Q16).
3. Iterate as user surfaces additional ideas or corrections.
4. Eventually, brainstorming stage ratchets toward SPEC engineering when the user is satisfied with BRAINSTORM completeness.

---

## Round 4 — Brainstorming-stage Q15–Q22 follow-up actions — Agent 1 (2026-04-28)

### Status

User answered Q15–Q22 (Phase 15 FEEDBACK Round 3 above) AND added substantive content via tracking-file Phase-15 inclusion lists for KNOWN_ISSUES, BRAINSTORM.md, and IBM. Agent 1 also applied a code-base audit that **disproved 2 of the 3 Round 3 proposed BRAIN entries** (BRAIN15.30 + BRAIN15.32 already implemented). User direction: speed > absolute auditing this session; user delegates more, will drift-audit later. User-quoted: *"I am willing to relinquish some human auditing to try to get through this entire phase today."*

User asked questions to be answered live in chat; Agent 1 records answers + chat context here so other agents can inherit context.

### Critical framework — Origin Detection Window (ODW)

User's Q15 answer settled the `peak_rcn_stage` audit at framework level: **keep all four metrics** (`onset_stage`, `last_active_stage`, `max_rcn_stage`, `regression_stage`); ODW = `onset_stage` to `last_active_stage` is the unifying concept; **"elongation interference"** is the term for what happens beyond the ODW. **`max_rcn_stage` was previously conflated with `last_active_stage` by an agent — they are SEPARATE concepts.** ODW is umbrella for BRAIN15.5 (HMM completeness matrix) / BRAIN15.8 (HMM summit refinement) / BRAIN15.18 (HMM shape-score + multistage unification) / BRAIN15.22 (IBM-C5 cross-pipeline ODW design) / BRAIN15.24 (peak_rcn_stage audit) / BRAIN15.29 (summit↔timing one-pass).

### Round 4 actions applied

**1. AGENT_CONVENTIONS additions (`multi-agent/AGENT_CONVENTIONS.md`):**

- **§ KNOWN_ISSUES.md — new migration provenance rule:** when porting a KNOWN_ISSUES entry to a new long-lived planning home, annotate the new-home entry with `**Formerly lived in KNOWN_ISSUES.md and was referred to as [ISSUE:YYYY-MM-DD:N].**` + delete from KNOWN_ISSUES.md (no stub).
- **§ Cross-referencing — new ID-context-in-chat rule:** when referring to project IDs in chat output (or in FEEDBACK / BRAINSTORM cross-references), agents must attach a brief in-situ topic phrase (e.g., "BRAIN15.7 (amplicon reliability)" not bare "BRAIN15.7"). User direction Round 3 chat 2026-04-28.

**2. BRAIN15.24 (`peak_rcn_stage` audit) — major body expansion:** added the four-metric framework, ODW umbrella concept, "elongation interference" framing, audit step 2 user-walkthrough resolution at framework level, audit step 4 naming recommendation (retire `peak_rcn_stage`; introduce `last_active_stage`), audit step 6 gating updates for BRAIN15.8 + BRAIN15.29.

**3. BRAIN15.5 (HMM completeness matrix) — Round 4 expansion:** gap-audit table (analysis families × HMM step inventory); ODW elevated as central organizing concept; Phase 15 deliverable surface as a checklist; cross-pipeline-rewiring scope flagged.

**4. BRAIN15.7 (amplicon reliability) — Round 4 expansion:** folded in IBM-C8B (Stage-1 pre-amp isolation) + IBM-C12 (APS locus diagnostics — `best_onset_stage` / `post_support` / `dip_rate`) + IBM-C14 (Keep/exclude recommendation enhancements) per Q16 user approval. **Verbatim user content** preserved from `tracking/BRAINSTORM.md [2026-04-01]` per user direction *"try to use verbatim notes when possible to avoid drifting."* Captures: founding design principle (must NOT penalize low absolute RCN); 4 calibration amplicons (DS1 chr II) as regression gate; already-implemented machinery (per-stage MAD tracks v0.5.47, oscillation annotation v0.5.48-49) to build on; tight coupling with BRAIN15.18 (multistage unification → final amplicon set → BRAIN15.7 reliability scoring); `--keep-amplicons-bed` user override.

**5. BRAIN15.17 (APS plots/parity catalog) — Round 4 expansion:** added the **step-less plots/notebooks structural change** (cross-pipeline; per Q18 + `[ISSUE:2026-04-28:1]`); HMM plot review verbatim from `tracking/BRAINSTORM.md [2026-04-09]`; fork-age-tracking already-implemented note; by-eye eval surface references per Q17 (`tests/full_chrom_training_data/amplicons.by-eye.bed`, `tests/summit_training_data/01-II9A...`, `dev/puffstep/automate/`); cross-pipeline plot inventory expansion candidates; bidirectional fork priorities 8.5.1/8.5.2/8.5.3 flagged for code audit.

**6. BRAIN15.30 (HMM seq filtering CLI flags) — RESOLVED marker.** Round 4 code audit verified all three flags already wired in `onionskin_core/engines/hmm_engine.py:121-130, 1000-1002`. Closeout marker only.

**7. BRAIN15.32 (HMM fork age tracking) — RESOLVED-with-enhancement-audit marker.** Round 4 code audit confirmed implementation in `onionskin_core/hmm_fork_travel.py:12, 242-243`. Closeout marker only; audit-driven enhancements fold into BRAIN15.17 / BRAIN15.5.

**8. BRAIN15.31 (`--peak-summary` extension to RMS + HMM) — NEW BRAIN authored** per Q20. Source: `tracking/KNOWN_ISSUES.md [ISSUE:2026-04-28:3] Max projection RCN profiles`. Verbatim user content preserved.

**9. BRAIN15.33 (Phase 15 housekeeping bundle) — NEW BRAIN authored** to bundle three thin Phase-15-marked KNOWN_ISSUES (`[ISSUE:2026-04-25:1]` `--onset-*`/`--timing-*` prefix harmonization; `[ISSUE:2026-04-28:4]` `summit_inspector.py` ModuleNotFoundError; `[ISSUE:2026-04-18:2]` `early-parabola-mean` runtime selector follow-up — connects to ODW). Bundled per substantive-priorities-v2 rule.

**10. APS_SOUP.md — `[ISSUE:2026-04-22:1]` migrated** per user direction *"Port it to APS_SOUP."* First application of the new KNOWN_ISSUES → new home with provenance annotation convention. Migrated entry now lives under `multi-agent/plans/next/APS_SOUP.md` with `**Formerly lived in KNOWN_ISSUES.md and was referred to as [ISSUE:2026-04-22:1].**` provenance line. Issue **deleted** from `tracking/KNOWN_ISSUES.md` (no stub).

**11. IBM-C6D (Timing-guided active-stage weights) — folded into BRAIN15.29 deferred-list update** per Q20 IBM bundle answer marking IBM-C6D as ODW-related.

### Round 4 actions APPROVED-but-DEFERRED to a later round

- **Citation enrichments across 10 BRAIN entries (Q21 batch):** approved but token-budget-deferred. Status: APPROVED-DEFERRED. To land in Round 5.
- **BRAINSTORM.md staleness cleanup:** user's Q20 hint *"BRAINSTORM.md may be stale and need an audit and clean up"* — separate housekeeping pass; not blocking.
- **Bidirectional fork priorities 8.5.1 / 8.5.2 / 8.5.3** — code audit needed before promotion.
- **`tracking/KNOWN_ISSUES.md [ISSUE:2026-04-24:2]`** (RMS summit parabola pre-smoothing methodology audit) — fold into BRAIN15.8 (HMM summit refinement) as a methodology-audit deliverable.

### Verbatim user content preserved (chat 2026-04-28; for inheritance by other agents)

#### On four-metric framework + ODW (Q15)

> *"I believe we have 4 different metrics of interest for defining the active window: onset_stage, last_active_stage (or some other name), max_rcn_stage, regression_stage. There is no need to get rid of any. It just depends how and where we are using them."*

> *"Stage 1 growth (or stage 2 when 1 is the ref-stage) is detected relative to background since there is no prior stage. Stages 2-N use some statistical test to compare stage S to stage S-1 to see if there was growth relative to the previous stage."*

> *"For comparing stage S to S-1, it does not need to be a ratio between them... we can just look at the series of summit RCN values from stage 1 to N. We could fit a loess line or something so a random oscillation does not necessarily screw up the analysis."*

#### On ODW superseding earlier framings (BRAINSTORM.md inclusion notes)

> *"the newer idea would be to identify the 'origin detection window (ODW)' which starts at the stage where summit growth is first detected and ends at the last stage summit growth is detected — this are setting boundaries on stages where the origin is actively firing, and the summit is expected to be more pointy and peak-like. In stages after the ODW, the origin is no longer active or is much less active, and the 'summit' region is extended by the last set of replication forks to something more like a flat table top over a broader region."*

> *"'Elongation interference' is a cool term. We should use it along with 'Origin Detection Window (ODW)'."*

#### On step-less plots/notebooks (Q18)

> *"all pipelines should remove the step from the plots and notebooks directories because as development continues, and steps are added or subtracted, the plots and notebooks directories can sit in the wrong place."*

#### On HMM width-progression (Q19)

> *"Yes, we can do width progression from the state path — that is probably most reliable — we can track progression of the oldest forks (leading edge of state 1), but with the HMM steps, we can track forks from each doubling round. But also, if the way it is done in Growth would produce meaningfully different results, then that approach could also be included in the HMM pipeline."*

#### On staleness pattern (Q20)

> *"BRAIN15.30 - this is supposed to already be implemented. [...] BRAIN15.32 may also already be implemented. I am starting to see the pattern that BRAINSTORM.md may be stale and need an audit and clean up."*

#### On verbatim-note preservation (Q16)

> *"yes, absolutely merge the ideas and enrich them with all prior planning notes. try to use verbatim notes when possible to avoid drifting away from the ideas, especially notes written by the human, which are least likely to contain drift."*

#### On the migration-with-provenance convention (Q22 footnote)

> *"port it to APS_SOUP.md, deleted from KNOWN_ISSUES, but documented in its new home as 'Formerly lived in KNOWN_ISSUES.md and was referred to as [ISSUE:2026-04-22:1]'. That last bit would allow us to track its history if ever needed, and would improve greppability. That behavior in general should probably be in AGENT_CONVENTIONS.md."*

#### On ID-context-in-chat (chat-format complaint)

> *"It is very slow for me when you refer to thing like 'IBM-C8B + C12 + C14 design content into BRAIN15.7's body'. While I have all these ideas in my head, the development of files and IDs is actually super fast and hard for humans to keep up with. The IDs are good for agents, and the agents can rip right through all the files. But for humans (me), I need you to refer to them in at least meaningful phrases that makes me think, 'Oh yeah - now we're talking about idea X'."*

#### On user mode for this session

> *"Today I need to prioritize speed of development over absolute human auditor accuracy. I will be leaning hard on you to make sure everything is accurate and lines up with my thinking. [...] I am willing to relinquish some human auditing to try to get through this entire phase today. [...] I reserve the right to continue changing my mind about everything I just said!"*

### Updated transfer-status block (post-Round-4)

**Status:** **Round 4 substantive expansions applied.** Brainstorming stage continues. User mode: speed > absolute auditing.

**BRAIN entry count:** **33** BRAIN IDs (BRAIN15.1–15.33). Up by 4 from post-Round-3 (15.30 + 15.31 + 15.32 + 15.33). Two are RESOLVED markers (15.30 + 15.32); two are new substantive entries (15.31 + 15.33).

- 21 BRAIN entries covering 21 SOUP IDs (splits and consolidations approved Round 1).
- 8 BRAIN entries with no SOUP source (user-approved): 15.24, 15.25, 15.26, 15.27, 15.28, 15.29, 15.31, 15.33.
- 4 BRAIN entries marked RESOLVED (closeout markers, no Phase 15 implementation): 15.13, 15.15, 15.30, 15.32.

**Files touched in Round 4:**

- `multi-agent/AGENT_CONVENTIONS.md` — new ID-context-in-chat rule (§ Cross-referencing); new KNOWN_ISSUES migration provenance rule (§ KNOWN_ISSUES.md).
- `multi-agent/plans/PHASE15_BRAINSTORM.md` — major expansions to BRAIN15.5 / 15.7 / 15.17 / 15.24; new BRAIN15.30 / 15.31 / 15.32 / 15.33; updated Coverage check; updated BRAIN15.29 deferred list (IBM-C6D promotion).
- `multi-agent/plans/next/APS_SOUP.md` — `[ISSUE:2026-04-22:1]` migrated with provenance annotation.
- `multi-agent/tracking/KNOWN_ISSUES.md` — `[ISSUE:2026-04-22:1]` deleted (migrated).
- `multi-agent/plans/PHASE15_FEEDBACK.md` — this Round 4 follow-up section appended.

### Open questions arising from Round 4 (Q23–Q24)

#### Q23 — `tracking/BRAINSTORM.md [2026-04-14] HMM summit estimation — use stage of maximum summit RCN, not last stage` — fully retire as superseded by ODW?

User Round 3 BRAINSTORM-inclusion notes flagged this entry as *"This might be a stale idea. [...] the newer idea would be to identify the 'origin detection window (ODW)'."*

**Q:** Is the [2026-04-14] entry's "use max stage" framing fully superseded by ODW (BRAIN15.24), or is there residual value in keeping it as a fallback/comparison strategy that gets implemented + evaluated alongside ODW-confined methods?

**A:**
- ## [2026-04-14] HMM summit estimation — use stage of maximum summit RCN, not last stage
    - This actually got most things right so can be bundled near the ODW ideas in general
    - even though this idea is superceded, as I mentioned above, we can still implement it
    - That is to say, it would be trivial to test whether summits from the max_rcn_stage (different for each amplicon) produces good results.
    - The ODW and elongation interference concepts are more mature and reasoned out, but what was written here was still close to the target, and not "bad"
    - what is nice about "max_rcn_stage" is that it is a very clear instruction of how to choose a stage for summit modeling for each amplicon
        - And the metric itself is useful to know in general
    - it is worthwhile testing the max_rcn_stage as the input for various summit estimators
    - similarly the other two related metrics make it easy to pick a stage for summit modeling
        - onset_stage - for each amplicon, see if the stage of earliest activity typically produces the best summit results with the various summit estimators (argmax, parabola, etc)
        - last_active_stage - for each amplicon, see if the last stage growth is detected is a good stage
    - But ultimately we can use all stages within or only some stages within the ODW
        - we can apply statistical approaches across the stages for summit estimation
        - we can try to find a stage within that has the best triangle or parabola shape -- that is the most pointy or peak-like, then use that one.
        - for HMM, it could be the stage with narrowest summit state interval too
        - note that we previously developed methods to classify amplicons as things like "pioneers" or something like that, and "constitutive" vs "faculatative"... I forget all the terms, but it was definitely related to or a precursor to the ODW concept, and we already had developed methods to determine whether there was growth between stages or not WITHIN the ODW. That is to say that it is possible to have stages within the ODW that do not show summit height since the last stage, but summit growth continues later in the window. You might want to find this code, and these ideas and outputs to see what is useful, and to make sure we don't redevelop the same ideas again, etc.
    - So I am saying some things here that may involve updating other parts of the Phase 15 Brainstream, bit regarding the question: don't eliminate it, just make it a parenthetical to the ODW stuff in general, and we can actually implement the concept of summit modeling using the max_rcn_stage to test it out.

#### Q24 — Phase 15 BRAINSTORM ratchet to SPEC engineering — what's the trigger?

The brainstorming stage has 33 BRAIN entries with substantive bodies. SPEC engineering will bundle into ~5-10 substantive priorities per the v2 rule. **Trigger options:**

(a) After Round 4 + Round 5 citation enrichment + BRAINSTORM staleness cleanup, ratchet immediately.
(b) After full code audits (8.5.1-3 fork priorities, [ISSUE:2026-04-24:2] parabola smoothing methodology) — more thorough but more delay.
(c) When user explicitly says "we're done; go SPEC."

Default (c) per AGENT_CONVENTIONS scope-authority rule.

**A:**
- "c" for sure - we are almost done, but I will have Codex audit us beforehand.

### Next steps (post-Round-4)

1. User reviews Round 4 actions; flags any drift.
2. Citation enrichments (Q21) can land in a focused Round 5 batch.
3. BRAINSTORM.md staleness cleanup pass (separate housekeeping).
4. Bidirectional fork priorities 8.5.1/2/3 + `[ISSUE:2026-04-24:2]` parabola smoothing — code audits.
5. Eventually ratchet to SPEC engineering per Q24.

---

## Round 5 — Q23+Q24 follow-up actions — Agent 1 (2026-04-28)

### Status

User answered Q23 (`tracking/BRAINSTORM.md [2026-04-14] HMM summit estimation` retire-or-keep) + Q24 (BRAINSTORM → SPEC ratchet trigger). Q23 brought substantive new content beyond just answering: per-amplicon stage-selection strategy menu for testing, the within-ODW non-monotonic-growth note, and the amplicon-class taxonomy code-archaeology audit task. User also clarified live in chat: **retire the term `peak_rcn_stage`; consolidate to `max_rcn_stage`** (existing code column; same concept; planning-surface synonym).

User direction Q24: option (c) — *"we are almost done, but I will have Codex audit us beforehand."* So next step is brainstorming-stage audit (Codex), then closeout, then SPEC engineering kickoff per PDS-v2.

### Round 5 actions applied

**1. BRAIN15.24 (peak_rcn_stage audit) — step-4 naming recommendation re-clarified.** Per user chat clarification 2026-04-28: retire the TERM `peak_rcn_stage` (planning-surface synonym for the existing `max_rcn_stage` code column); keep the IDEA of argmax-based stage selection (as one strategy on BRAIN15.8's menu). The four-metric framework now has a clean naming summary: `onset_stage` / `last_active_stage` / `max_rcn_stage` (formerly aliased as `peak_rcn_stage` in planning surfaces) / `regression_stage`. Cleanup of `peak_rcn_stage` mentions across tracking files is part of BRAIN15.34 (Phase 15 closeout sweep).

**2. BRAIN15.8 (HMM summit refinement) — Round 5 expansion + retitle.** Title reframed from *"`peak_rcn_stage` column + sliding-offset sub-bin refinement"* to *"per-amplicon stage-selection (incl. `max_rcn_stage`) + ODW-confined refinement + sliding-offset sub-bin"* to reflect the broader scope. New section: **menu of per-amplicon stage-selection strategies for testing**, with 6 candidates per Q23 user content:

1. `max_rcn_stage` (argmax — existing column; testable as-is)
2. `onset_stage` (first stage of activity)
3. `last_active_stage` (last stage of growth — new column per BRAIN15.24)
4. All stages within ODW (statistical aggregation)
5. Subset of ODW stages (best triangle/parabola shape)
6. HMM-specific: stage with narrowest summit-state interval

Plus implementation order: stage-selection machinery first → evaluate strategies → wire sliding-offset at chosen stage(s) → report results across strategies.

**3. BRAIN15.7 (amplicon reliability) — Round 5 expansion: amplicon-class taxonomy code-archaeology audit task.** Per Q23 user note about prior pioneer/constitutive/facultative classification methods + within-ODW growth-vs-flat detection. Added as a sub-priority of BRAIN15.7. Audit scope: search code (`onionskin_core/timing.py`, `onionskin_core/aps.py`, `onionskin_core/refinement.py`, `onionskin_core/rcn_mean_shift_helpers.py`) + planning surfaces for existing terms (`pioneer`, `constitutive`, `facultative`, `amplicon_class`, `amplicon_type`, etc.). Per-finding decision: Adopt / Refine / Supersede. Connection to BRAIN15.18 (within-ODW growth-vs-flat detection feeds multistage unification) + BRAIN15.24 (`last_active_stage` may already exist under another name) + BRAIN15.8 (stage-selection strategies should reuse existing classification machinery if available).

**4. BRAIN15.18 (HMM shape-score wiring + multistage unification) — Round 5 update: within-ODW non-monotonic growth handling.** Per Q23 user clarification: the ODW is NOT strictly monotonic; some stages within `[onset_stage, last_active_stage]` may show no growth from the previous stage but growth resumes later. Sub-priorities (d) + (e) (4-mode classification flag + state-path evolution checks) must accommodate this — single non-growth transitions mid-window do NOT close the window; only sustained-flat-or-declining behavior does. Cross-references to BRAIN15.24's `last_active_stage` vs `regression_stage` distinction (different stages because of mid-window plateaus).

**5. `peak_rcn_stage` → `max_rcn_stage` retirement clarification documented.** User chat 2026-04-28: *"Let's differentiate between 'peak_rcn_stage' and 'max_rcn_stage'. I am advocating for 'max_rcn_stage' in my Q23 answer - sorry if I used 'peak_rcn_stage'. Those two terms come up - and we need audit what each is referring to -- if the same thing, then just one. Ultimately I forsee retiring the term 'peak_rcn_stage'."* Audit conclusion (already in BRAIN15.24): same concept; consolidate to `max_rcn_stage`; retire `peak_rcn_stage`. The cleanup of tracking-file mentions of `peak_rcn_stage` is folded into BRAIN15.34's Phase 15 closeout sweep.

### Verbatim user content from Q23 + Q24 chat (preserved here for inheritance)

#### Q23 — "make max_rcn_stage testable; don't eliminate it" framing

> *"This actually got most things right so can be bundled near the ODW ideas in general. Even though this idea is superceded, as I mentioned above, we can still implement it. That is to say, it would be trivial to test whether summits from the max_rcn_stage (different for each amplicon) produces good results. The ODW and elongation interference concepts are more mature and reasoned out, but what was written here was still close to the target, and not 'bad'."*

> *"What is nice about 'max_rcn_stage' is that it is a very clear instruction of how to choose a stage for summit modeling for each amplicon. And the metric itself is useful to know in general. It is worthwhile testing the max_rcn_stage as the input for various summit estimators. Similarly the other two related metrics make it easy to pick a stage for summit modeling: onset_stage - for each amplicon, see if the stage of earliest activity typically produces the best summit results with the various summit estimators (argmax, parabola, etc); last_active_stage - for each amplicon, see if the last stage growth is detected is a good stage."*

> *"But ultimately we can use all stages within or only some stages within the ODW: we can apply statistical approaches across the stages for summit estimation; we can try to find a stage within that has the best triangle or parabola shape — that is the most pointy or peak-like, then use that one; for HMM, it could be the stage with narrowest summit state interval too."*

#### Q23 — pioneer/constitutive/facultative + within-ODW non-monotonic growth

> *"note that we previously developed methods to classify amplicons as things like 'pioneers' or something like that, and 'constitutive' vs 'faculatative'... I forget all the terms, but it was definitely related to or a precursor to the ODW concept, and we already had developed methods to determine whether there was growth between stages or not WITHIN the ODW. That is to say that it is possible to have stages within the ODW that do not show summit height since the last stage, but summit growth continues later in the window. You might want to find this code, and these ideas and outputs to see what is useful, and to make sure we don't redevelop the same ideas again, etc."*

#### Q24 — Codex audit precedes brainstorming-stage closeout

> *"'c' for sure - we are almost done, but I will have Codex audit us beforehand."*

#### Chat (2026-04-28) — peak_rcn_stage retirement

> *"Let's differentiate between 'peak_rcn_stage' and 'max_rcn_stage'. I am advocating for 'max_rcn_stage' in my Q23 answer - sorry if I used 'peak_rcn_stage'. Those two terms come up - and we need audit what each is referring to -- if the same thing, then just one. Ultimately I forsee retiring the term 'peak_rcn_stage'."*

### Updated transfer-status block (post-Round-5)

**Status:** **Brainstorming stage substantively complete.** User direction Q24: ratchet to SPEC engineering after Codex audit + closeout. **Awaiting Codex brainstorming-stage audit.**

**BRAIN entry count:** **34** BRAIN IDs (BRAIN15.1–15.34). Round 5 added no new BRAIN IDs — all changes were body expansions / retitling / clarifications on existing entries (15.7, 15.8, 15.18, 15.24).

- 21 BRAIN entries covering 21 SOUP IDs.
- 9 user-introduced no-SOUP-source entries: 15.24, 15.25, 15.26, 15.27, 15.28, 15.29, 15.31, 15.33, 15.34.
- 4 RESOLVED markers: 15.13, 15.15, 15.30, 15.32.

**Files touched in Round 5:**

- `multi-agent/plans/PHASE15_BRAINSTORM.md` — BRAIN15.7 expansion (amplicon-class taxonomy audit); BRAIN15.8 retitle + Round 5 expansion (stage-selection strategy menu); BRAIN15.18 within-ODW non-monotonic-growth note; BRAIN15.24 step-4 naming recommendation re-clarified (peak_rcn_stage → max_rcn_stage retirement).
- `multi-agent/plans/PHASE15_FEEDBACK.md` — this Round 5 section appended.

### Brainstorming-stage closeout readiness checklist (per PDS-v2 § How the Phase Development System Works)

User flagged Q24 answer: *"we are almost done, but I will have Codex audit us beforehand."* — so the next step is the **brainstorming-stage audit** (Codex acts as Agent 2). After CLEAN, **brainstorming-stage closeout** + ratchet to SPEC engineering. Per PDS-v2:

**Audit scope (suggested for Codex):**

- All 34 BRAIN entries have well-formed bodies with `Source:` provenance.
- All 21 SOUP IDs cited (mechanically: `grep -nE "SOUP15\.[0-9]+" multi-agent/plans/PHASE15_BRAINSTORM.md | sort -u | wc -l` should return 21).
- ODW + four-metric framework consistently applied across BRAIN15.5 / 15.8 / 15.18 / 15.22 / 15.24 / 15.29 (no `peak_rcn_stage` framing surviving outside BRAIN15.24's retirement note).
- User-direction items (Q1-Q24 + chat clarifications) all reflected in BRAIN entries with verbatim quotes preserved where applicable.
- Cross-pipeline-rewiring scope items (BRAIN15.21 gap mask, BRAIN15.31 peak-summary, BRAIN15.17 step-less plots/notebooks) consistent with BRAIN15.5 gap-audit table.
- Tracking-file Phase-15 inclusion items (per Round 3 user lists) covered by at least one BRAIN entry.
- BRAIN15.34 Phase 15 closeout sweep priority is positioned as the LAST priority (not first or middle) so the cleanup happens after all implementation lands.

**Once Codex declares CLEAN:**

1. Brainstorming-stage closeout entry appended to FEEDBACK.
2. (Optional) BRAINSTORM tag-line updated: *"Stage: BRAINSTORM — closed at v0.X.YY.Z; SPEC engineering open"*.
3. SPEC engineering opens per PDS-v2: agent reads BRAINSTORM, writes `multi-agent/plans/PHASE15_SPEC.md` with substantive priorities (5–10 per the v2 rule). Each SPEC priority has a `Source:` / `Covers:` field citing the BRAIN IDs it bundles.
4. SPEC engineering Q&A continues using the same FEEDBACK file (this one).
5. Once SPEC is locked, audit/implement/reaudit cycles begin per `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`.

### Open questions arising from Round 5

None. Round 5 resolved Q23 + Q24; no new questions raised. Ready for Codex audit.

### Next steps (post-Round-5)

1. **Orchestrator invokes Codex (Agent 2) brainstorming-stage audit prompt** per PDS-v2.
2. Agent 1 addresses any Codex findings in a Round 6 follow-up if needed.
3. Once Codex declares CLEAN, brainstorming-stage closeout entry + SPEC engineering kickoff.

## Agent 2 brainstorming-stage code/tracking audit - 2026-04-28

**Auditor:** Codex (Agent 2).

**Scope:** Deep-dive audit of the live Phase 15 BRAINSTORM against code, `scripts/`, `tests/`, `multi-agent/tracking/BRAINSTORM.md`, `multi-agent/tracking/KNOWN_ISSUES.md`, and `multi-agent/tracking/INTENDED-BUT-MISSED-PRIOR-TO-14.md`, focused on missed or under-specified HMM-completeness / parity / HMM-specific-analysis work. I did not edit `PHASE15_BRAINSTORM.md`.

**Verdict:** **OPEN - small Agent 1 follow-up pass recommended before brainstorming-stage closeout.** The current BRAINSTORM is substantively strong and I did not find a missing major pillar. The remaining issues are mostly concrete code-surface inheritance and one semantic split that should be explicit before SPEC engineering so the implementer does not rediscover them mid-cycle.

### High-level assessment

1. **Overall concerns mostly addressed.** The 34-entry BRAINSTORM now captures the core Phase 15 agenda: HMM downstream parity, per-sample HMM child outputs, SAPS, ODW, HMM-specific state-path analyses, APS feature expansion, reliability/flat-sample filtering, summit/timing convergence, and late-stage housekeeping.
2. **No new large BRAIN entry required from this audit.** My findings can be folded into existing entries: BRAIN15.6, BRAIN15.7, BRAIN15.17, BRAIN15.19, BRAIN15.27, BRAIN15.28, BRAIN15.33, and BRAIN15.34.
3. **The main risk is not conceptual scope, but implementation-surface drift.** Several Phase 15 ideas intentionally change HMM step numbering, output schemas, APS feature columns, and notebook/plot layout. The BRAINSTORM acknowledges this generally, but SPEC engineering should inherit a concrete file checklist.

### Specific findings

1. **BRAIN15.6 step-renumbering sweep is directionally right, but should carry exact file targets.**
   - BRAIN15.6 already says to renumber timing to `16-timing/`, clustering to `17-clustering/`, and update helpers/utilities (`PHASE15_BRAINSTORM.md:168`, `PHASE15_BRAINSTORM.md:176-178`).
   - Concrete live surfaces to name in the SPEC: `onionskin_core/output_layout.py:281-300` currently maps HMM step14/15/16 to `14-aps`, `15-timing`, `16-clustering`; `onionskin_core/engines/hmm_engine.py:1236-1268` imports and runs `run_step14_hmm_aps`, `run_step15_hmm_timing`, and `run_step16_hmm_clustering`; `onionskin_core/hmm_notebooks.py:84`, `:111`, and `:122` hardcode `12-fork-travel`, `14-aps`, `15-timing`, and `16-clustering`; `tests/test_pipeline.py:179` asserts the help string includes `[hmm: 14-aps; growth: 13-aps; rms: 12-aps]`; `scripts/summit_inspector.py:153-158` searches `hmm_steps['step15']/hmm_summit_estimates.tsv`.
   - Small typo/gotcha: BRAIN15.6 says `05-medianSmoothedRCN/indiv_samples/` (`PHASE15_BRAINSTORM.md:166`, `:174`), but the actual HMM step directory is `05-medianSmoothed-RCN` (`onionskin_core/output_layout.py:288`). This should be corrected in BRAINSTORM before closeout or handled explicitly in SPEC.

2. **The amplicon-class archaeology concern is real and already has live code.**
   - BRAIN15.7 Round 5 correctly says to search for prior pioneer / constitutive / facultative classification logic (`PHASE15_BRAINSTORM.md:317-334`).
   - Code audit found the live implementation in `onionskin_core/timing.py`: timing output schema includes `latest_activity_stage`, `activity_breadth`, and `amplicon_class` (`timing.py:300-317`); stage-over-stage active-growth detection uses a 1.25 fold threshold after onset (`timing.py:370-393`); output rows write `latest_activity_stage` and `activity_breadth` (`timing.py:466-468`); `_classify_amplicon()` emits `Constitutive_Prolonged`, `Facultative_Prolonged`, `Founder`, `Finisher`, `Intermediary`, and `Excluded` (`timing.py:511-527`).
   - Recommendation: update BRAIN15.7/BRAIN15.24 during Agent 1 follow-up from "find whether this exists" to "known live timing implementation to audit, port, and possibly rename/harmonize." In particular, decide whether `latest_activity_stage` is the predecessor/current name for ODW's `last_active_stage`, or whether Phase 15 should keep both with an explicit distinction.

3. **HMM "clustering" currently names two different concepts; SPEC should split them explicitly.**
   - Current HMM step-14 APS produces sample-level APS matrices and `aps_clusters.tsv` from APS feature columns (`onionskin_core/hmm_ported_analyses.py:149-184`).
   - Current HMM step-16 "clustering" is not posterior sample grouping; it is trajectory clustering from fork-travel feature vectors and emits `trajectory_feature_matrix.tsv` / `trajectory_clusters.tsv` (`onionskin_core/hmm_ported_analyses.py:380-439`).
   - BRAIN15.27 is correctly about APS-driven posterior sample grouping / posterior manifests (`PHASE15_BRAINSTORM.md:1215-1217`), while BRAIN15.19 and BRAIN15.5 also talk about clustering/posterior surfaces. The SPEC should preserve both concepts with distinct names: APS/sample posterior clustering versus HMM trajectory/fork clustering. Otherwise "step 17 clustering" could silently mean the wrong downstream product.

4. **APS experiment harness is a concrete missed script surface.**
   - `scripts/aps_cluster_experiments.py` is still growth-only (`--pipelines growth`) and enumerates only the older `--aps-feature` modes (`scripts/aps_cluster_experiments.py:26-45`).
   - BRAIN15.27 adds multiple new feature modes and BRAIN15.19/BRAIN15.17 need cross-pipeline APS validation. Recommendation: fold this script into BRAIN15.19/BRAIN15.27 as an explicit deliverable: make the harness pipeline-selectable (`growth`, `rms`, `hmm`), include new feature modes, and ensure outputs stay under `dev/runs/<run_name>/` by default/examples.

5. **`summit_excess` to `summit_rcn` migration is captured, but HMM-specific APS code needs an explicit touchpoint.**
   - BRAIN15.28 correctly owns the `summit_excess` -> `summit_rcn` / raw-RCN migration (`PHASE15_BRAINSTORM.md:1301-1324`).
   - The HMM port currently maps `--aps-feature summit` to `summit_excess` in its local feature resolver (`onionskin_core/hmm_ported_analyses.py:149-151`). SPEC should name this HMM-side resolver directly, not just the shared APS module, so HMM does not lag the cross-pipeline rename/default flip.

6. **Header/source-status cleanup is still needed after SOUP archival.**
   - `PHASE15_BRAINSTORM.md:5` still says the source SOUP is `multi-agent/plans/PHASE15_HMM_SOUP.md` and "live-in-`plans/` until ... archived." The SOUP is already archived at `multi-agent/plans/archived/20260428-PHASE15_HMM_SOUP.md`.
   - Recommendation: as part of Agent 1 closeout/follow-up, update the header line to the archived SOUP path or add a closeout note. This is bookkeeping, but it will prevent cold-start confusion.

7. **BRAIN15.17's step-less plot/notebook idea is right; include existing HMM notebook generator as a named target.**
   - BRAIN15.17 already says to remove step numbers from plots/notebooks and sweep scripts/tests (`PHASE15_BRAINSTORM.md:632-654`).
   - Concrete target: `onionskin_core/hmm_notebooks.py` currently writes notebooks that reach into step-numbered HMM directories (`hmm_notebooks.py:84`, `:111`, `:122`) and the function name still reads `write_hmm_phase9_notebooks` at call site (`onionskin_core/engines/hmm_engine.py:1269-1270`). Recommendation: BRAIN15.17 or BRAIN15.33 should explicitly include renaming/modernizing this generator when the HMM notebook surface is expanded.

8. **Tracking-source harmonization is mostly complete.**
   - The high-priority IBM-C items from `INTENDED-BUT-MISSED-PRIOR-TO-14.md` are represented: IBM-C2 by BRAIN15.6/20, IBM-C3/C8B/C12/C14 by BRAIN15.7, IBM-C4 by BRAIN15.8, IBM-C5 by BRAIN15.22/24, IBM-C6C/D by BRAIN15.29, IBM-C7A/B by BRAIN15.27/APS_SOUP, and IBM-C8A by BRAIN15.14.
   - I did not find an unrepresented high-confidence tracking item that belongs in Phase 15. User-gated or broader synthesis ideas, such as external signal-track ingestion and final cross-pipeline unified synthesis, appear properly left outside this HMM-completeness brainstorm.

### Recommendations

1. Agent 1 should do a narrow Round 6 follow-up pass that updates BRAIN15.6, BRAIN15.7/BRAIN15.24, BRAIN15.17/BRAIN15.33, and optionally BRAIN15.19/BRAIN15.27 with the exact code/script/test surfaces above.
2. Correct the `05-medianSmoothedRCN` vs `05-medianSmoothed-RCN` typo in BRAIN15.6.
3. Make `latest_activity_stage` / `amplicon_class` in `timing.py` a first-class input to the ODW and reliability discussion rather than a mere archaeology target.
4. Keep APS/sample clustering and HMM trajectory/fork clustering distinct in SPEC terminology and output paths.
5. After the small follow-up pass, I would consider the brainstorm ready for closeout and SPEC engineering. No additional broad brainstorming round seems necessary from this audit.

**Tests run:** none. This was a planning/code-reading audit only; no production code was changed.

**Signed:** Codex, Agent 2.

---

## Round 6 — Agent 1 triage of Codex audit findings — 2026-04-28

**Triage summary:** Codex's audit verdict was OPEN with a narrow Agent 1 follow-up pass recommended before brainstorming-stage closeout. **All 8 specific findings were ACCEPTED and incorporated** into BRAINSTORM via Round 6 updates on existing entries (no new BRAIN IDs added; per Codex's own assessment, no new major pillar was needed). Round 6 also incorporated **two follow-on user clarifications** that came up during execution: (a) `latest_activity_stage` → `last_active_stage` rename decision (user chose option a — single canonical name; rename code), and (b) Gaussian-overlap probabilistic dip test surfaced as a more sophisticated alternative to the 1.25× hard-threshold test for within-ODW growth detection (user-prompted lookup; cross-references added to BRAIN15.7 + BRAIN15.24).

### Per-finding triage

**Finding 1 — BRAIN15.6 step-renumbering needs concrete file targets + `05-medianSmoothedRCN` typo. → ACCEPTED.**

- **Action:** Added Round 6 update to BRAIN15.6 listing the 5 concrete code-surface targets the SPEC engineering stage must touch: `onionskin_core/output_layout.py:281-300`, `onionskin_core/engines/hmm_engine.py:1236-1268`, `onionskin_core/hmm_notebooks.py:84,111,122`, `tests/test_pipeline.py:179`, `scripts/summit_inspector.py:153-158`. Each target's expected change documented.
- **Typo fix:** Corrected both occurrences of `05-medianSmoothedRCN` → `05-medianSmoothed-RCN` in BRAIN15.6 (lines were 166 + 174).

**Finding 2 — BRAIN15.7/BRAIN15.24 amplicon-class taxonomy: promote from "archaeology target" to "known live timing.py implementation." → ACCEPTED.**

- **Action on BRAIN15.7:** Added Round 6 update to the "Round 5 expansion" section. The amplicon-class machinery is no longer "search for it" — Codex located it in `onionskin_core/timing.py:300-317, :370-393, :466-468, :511-527`. Six classification labels documented (`Constitutive_Prolonged` / `Facultative_Prolonged` / `Founder` / `Finisher` / `Intermediary` / `Excluded`); the 1.25 fold-change threshold and per-classifier inputs documented. Phase 15 deliverable shifts from "find" to "audit + port to HMM + harmonize naming + per-class reliability scoring + cross-pipeline harmonization check + consider unifying within-ODW growth-detection test under the Gaussian-overlap statistical framework" (the 6th deliverable item is the Round 6 user-prompted addition — see below).
- **Action on BRAIN15.24:** Updated the four-metric framework's `last_active_stage` row in the naming-summary table. No longer "NEW column to be added" — explicitly noted as already-existing in code as `latest_activity_stage`. **User decision (chat 2026-04-28): rename `latest_activity_stage` → `last_active_stage` cross-pipeline (option a — single canonical name; no synonyms; minimal-but-contained code churn).** Added a Round 6 connection-to-BRAIN15.7 note. Added a Round 6 connection-to-the-Gaussian-overlap-probabilistic-dip-test note (see Round 6 user-prompted finding 2026-04-28 below).

**Finding 3 — BRAIN15.19/BRAIN15.27 should split sample posterior clustering vs HMM trajectory/fork clustering in SPEC-facing language. → ACCEPTED.**

- **Action on BRAIN15.19:** Added Round 6 update with the explicit two-concept split. The two concepts: (1) **APS sample posterior clustering** (HMM step-14 `aps/`, `aps_clusters.tsv`, `onionskin_core/hmm_ported_analyses.py:149-184`, drives posterior sample groupings / posterior manifests) — what BRAIN15.19 + BRAIN15.27 are about. (2) **HMM trajectory/fork clustering** (HMM step-16 → 17 post-renumbering, `trajectory_feature_matrix.tsv` / `trajectory_clusters.tsv`, `onionskin_core/hmm_ported_analyses.py:380-439`) — HMM-specific; uses fork-trajectory features. SPEC-engineering rule: keep distinctly named in priority titles, output paths, help-string text. Cross-reference to BRAIN15.5 (HMM completeness matrix) for the analysis-families table.
- **Action on BRAIN15.27:** Added Round 6 update echoing the two-concept split (BRAIN15.27 is strictly about APS sample posterior clustering, NOT trajectory clustering). Plus folded in `scripts/aps_cluster_experiments.py` as a Phase 15 BRAIN15.27 + BRAIN15.19 deliverable: make the experiment harness pipeline-selectable (currently growth-only at line 26-45), include all new feature modes, honor BRAIN15.21 + BRAIN15.31 cross-pipeline scope, dev/runs/<run_name>/ outputs.

**Finding 4 — BRAIN15.17/BRAIN15.33 should name `scripts/aps_cluster_experiments.py`, `scripts/summit_inspector.py`, and `onionskin_core/hmm_notebooks.py` as concrete implementation targets. → ACCEPTED.**

- **Action on BRAIN15.17:** Added Round 6 update naming `onionskin_core/hmm_notebooks.py:84, :111, :122` as the existing HMM notebook generator with hardcoded step-numbered paths that Phase 15's step-renumber + step-less plots/notebooks structural change must update. Cross-reference to BRAIN15.33 for the function-rename deliverable.
- **Action on BRAIN15.33:** Added Round 6 update — bundled item 4 (HMM notebook generator function rename + modernization). Renames `write_hmm_phase9_notebooks` → `write_hmm_notebooks` + updates callsite `onionskin_core/engines/hmm_engine.py:1269-1270` + cleans up stale Phase-9-era hardcoded paths inside the function. (`scripts/aps_cluster_experiments.py` is folded into BRAIN15.27 + BRAIN15.19 per Finding 3 action; `scripts/summit_inspector.py` is already in BRAIN15.33 bundled item 2 as the ModuleNotFoundError fix, with the step-renumber update bundled in.)

**Finding 5 — BRAIN15.28 should name HMM-side resolver `onionskin_core/hmm_ported_analyses.py:149-151` explicitly. → ACCEPTED.**

- **Action on BRAIN15.28:** Added Round 6 update naming the HMM-side feature resolver at `hmm_ported_analyses.py:149-151` as a separate touchpoint that the rename must hit alongside the shared APS module resolver. Otherwise HMM continues consuming `summit_excess` after the cross-pipeline rename, producing inconsistent output across pipelines. Cross-reference to BRAIN15.20 (SAPS) for harmonizing SAPS feature naming with the post-rename APS schema.

**Finding 6 — Header cleanup: `PHASE15_BRAINSTORM.md:5` source-SOUP line still says live path. → ACCEPTED.**

- **Action:** Updated `PHASE15_BRAINSTORM.md` line 5 (Source SOUP) and line 6 (Lifecycle position) to reflect: SOUP archived 2026-04-28 at brainstorming-stage cycle closeout v0.14.76.2, archived path `multi-agent/plans/archived/20260428-PHASE15_HMM_SOUP.md`. Lifecycle position updated to: brainstorming stage active; iterated through Round 1–Round 5 + Codex Agent 2 audit at v0.14.77; ratchets to SPEC engineering when user signals Q24 = "we're done."

**Finding 7 — BRAIN15.17 should include existing HMM notebook generator (`onionskin_core/hmm_notebooks.py`) as named target for step-less plots/notebooks structural change. → ACCEPTED.**

- **Action:** Same as Finding 4 — addressed by the BRAIN15.17 Round 6 update + the BRAIN15.33 bundled item 4 cross-reference. Notebook-generator rename (function-name modernization) lives in BRAIN15.33; step-numbered-path-update inside the generator function lives in BRAIN15.17.

**Finding 8 — Tracking-source harmonization is mostly complete; no unrepresented high-confidence tracking item found. → ACCEPTED-AS-CONFIRMATION.**

- **Action:** No BRAINSTORM change needed; Codex's confirmation that user-gated and broader-synthesis ideas are properly outside Phase 15 is logged here. Re-states that high-priority IBM-C items (C2 / C3 / C4 / C5 / C6C / C6D / C7A / C7B / C8A / C8B / C12 / C14) are all represented in BRAIN15.6 / 7 / 8 / 14 / 18 / 22 / 27 / 29 / APS_SOUP. Reassuring confirmation; no further action.

### Round 6 user-prompted findings (chat 2026-04-28; not in Codex's audit but surfaced during Round 6 execution)

**Finding U1 — `latest_activity_stage` → `last_active_stage` rename decision. → DECIDED.**

- **User chat decision:** option (a) — rename in code; single canonical name; no synonyms. Codex's audit had presented three options; user chose the cleanest. Captured in BRAIN15.24 four-metric naming-summary table.

**Finding U2 — Gaussian-overlap probabilistic dip test as more sophisticated alternative to the 1.25× hard-threshold rule. → ADDED to BRAIN15.7 + BRAIN15.24 as a SPEC-engineering-time consideration.**

- **Discovery context:** user prompted Agent 1 to find the statistical test they recalled from the same v0.5.x development era as the timing.py amplicon-class machinery. Agent 1 located it at `onionskin_core/aps.py:748-1041` (probabilistic oscillation test, v0.5.48). Uses `scipy.stats.norm.cdf` at line 993 (verified single occurrence in the codebase via grep for `Phi` / `norm.cdf`). Sigma derived from per-stage MAD bedGraphs (v0.5.47). Output: dip probability per transition; "confirmed dip" thresholding.
- **Why it matters for Phase 15:** the 1.25× test (timing.py:370-393, defines `last_active_stage`) and the Gaussian-overlap test (aps.py:748-1041, defines `regression_stage`, `is_oscillator`, `oscillating_transitions`) are complementary — 1.25× tests upward direction (active growth); Gaussian-overlap tests downward direction (confirmed dip). The Gaussian-overlap test is more sophisticated: variance-aware (uses per-stage MAD), probabilistic (continuous score), adaptive (high-variance stages need bigger fold-change to register). **SPEC-engineering decision:** consider unifying both within-ODW tests under the Gaussian-overlap framework (use it for `last_active_stage` upward direction too, replacing the fixed 1.25× threshold). Architecturally cleaner; biologically more defensible.
- **Action:** Added as deliverable item 6 in BRAIN15.7 Round 6 update + cross-reference note in BRAIN15.24 Round 6 update. Decision deferred to SPEC engineering; brainstorming captures the design space.

### Findings rejected: NONE

All 8 Codex audit findings + 2 user-prompted findings were accepted and incorporated. No reject list. The Codex audit was tightly scoped to genuinely actionable items (no overreach), and Round 6 user-prompted findings naturally extended the scope without any conflict.

### Files touched in Round 6 (Agent 1 follow-up)

- `multi-agent/plans/PHASE15_BRAINSTORM.md` — header line 5/6 updated; BRAIN15.6 Round 6 update + typo fix; BRAIN15.7 Round 6 update (timing.py promotion + Gaussian-overlap unification deliverable); BRAIN15.17 Round 6 update (hmm_notebooks.py named); BRAIN15.19 Round 6 update (clustering split); BRAIN15.24 Round 6 update (last_active_stage canonical + Gaussian-overlap connection); BRAIN15.27 Round 6 update (clustering split + aps_cluster_experiments.py folded in); BRAIN15.28 Round 6 update (hmm_ported_analyses.py:149-151 named); BRAIN15.33 Round 6 update (bundled item 4: hmm_notebooks.py rename).
- `multi-agent/plans/PHASE15_FEEDBACK.md` — this Round 6 triage section appended.

**No new BRAIN IDs added in Round 6.** All findings folded into existing entries per Codex's own assessment that "no new large BRAIN entry required from this audit." BRAIN entry count remains **34** (BRAIN15.1 through BRAIN15.34).

### Updated transfer-status block (post-Round-6)

**Status:** **Brainstorming stage substantively complete; Codex audit findings all addressed via Round 6 follow-up.** Per Q24 user direction (option c), brainstorming-stage closeout + ratchet to SPEC engineering is gated on user signal "we're done; go SPEC."

### Next steps (post-Round-6)

1. **User reviews Round 6 actions** + flags any drift or missed nuance.
2. **Optional second Codex pass** to verify all 8 findings + 2 user-prompted additions landed cleanly. Codex's original audit said *"After the small follow-up pass, I would consider the brainstorm ready for closeout and SPEC engineering. No additional broad brainstorming round seems necessary from this audit."* — so a narrow re-audit on just the Round 6 changes (similar to the Round 2.5 narrow re-audit pattern from the soup-to-brainstorm cycle) is sufficient if user wants a CLEAN verdict before closeout.
3. **Brainstorming-stage closeout** — user signals "go SPEC" when ready; agent appends closeout entry to FEEDBACK; one DEVLOG entry per the new convention (Form B closeout commit, dev-system-routed). Closeout includes archiving BRAINSTORM (or just transitioning state-marker; SPEC engineering will reference BRAINSTORM throughout).
4. **SPEC engineering kickoff** — agent reads BRAINSTORM end-to-end, writes `multi-agent/plans/PHASE15_SPEC.md` with substantive priorities (5–10 per the v2 rule). Each SPEC priority has `Source:` / `Covers:` field citing the BRAIN IDs it bundles. SPEC engineering Q&A continues using the same FEEDBACK file.
5. **Once SPEC is locked,** audit/implement/reaudit cycles begin per `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`.

### Round 6 addendum (post-triage user clarification, 2026-04-28)

After the Round 6 triage section was appended, the user formalized the decision implied by Finding U2 (Gaussian-overlap probabilistic dip test as more sophisticated than the 1.25× hard rule). User-quoted (chat 2026-04-28):

> *"Yes - I want to officially agree that last_active_stage detection mechanics could potentially use the Gaussian-overlap framework and that it would probably be better than just '1.25x'. If we wanted to use '1.25x' then I would want to frame it as a bayesian probability problem where we test the posterior probability that it is >= 1.25x and require the posterior probability to be >0.9 or 0.95 or 0.99 or something like that."*

**Decision recorded:** the 1.25× hard-threshold rule for `last_active_stage` detection is **OFFICIALLY being retired in Phase 15**. Two candidate replacement formulations are both acceptable; SPEC engineering picks A, B, or both-as-flag-options:

- **A. Gaussian-overlap framework** — reuses the existing `aps.py:993` machinery flipped for the upward direction. Tests "any directional growth" without preserving the 1.25× biological-meaningfulness anchor. Zero new statistical machinery.
- **B. Bayesian posterior P(ratio ≥ 1.25)** — tests the biologically-meaningful 1.25× effect-size threshold while adding variance-awareness via the per-stage MAD-derived sigma. Posterior threshold per user: 0.9, 0.95, or 0.99. Computationally similar to A; preserves the historical 1.25× anchor.

**BRAINSTORM updates applied:**

- **BRAIN15.7 deliverable item 6** — rewritten to make the decision official; full side-by-side of formulations A + B; SPEC-engineering decision items deferred (pick A vs B vs both; pick threshold values; cross-pipeline harmonization).
- **BRAIN15.24** — the Gaussian-overlap connection note updated to reflect the official retirement of the 1.25× hard rule + cross-reference to BRAIN15.7 item 6 for the formulation-A-vs-B decision.
- **BRAIN15.18** — within-ODW non-monotonic-growth handling section updated: BRAIN15.18 sub-priorities (d) + (e) consume the **post-rewrite** within-ODW growth-detection test, not the legacy 1.25× rule.

**This means BRAIN15.7 Round 6 deliverable item 6 is now an ACTIVE Phase 15 priority, not just a "consider" item.** SPEC engineering will treat it as a required deliverable when the timing.py rewrite + amplicon-class-classifier audit lands.

### Round 6 addendum 2 (Bayesian ODW System formalization, 2026-04-28)

User extended the framework decision into a coherent system with concrete CLI surface. Verbatim user direction (chat 2026-04-28):

> *"The flag does not have to be as narrow as '--last-active-stage-detector' because it is being used to detect growth betweens stages more generally, and the results might be more useful than just for the last active stage. So it can be more general like '--active-stage-detector' or something directly relating it to the origin detection window ... or something that has both like '--odw-active-stage-detector' ... not sure."*

> *"I prefer the Bayesian too, but I like having options for all 3."*

> *"I would suggest a sibling CLI flags for the probability threshold (default=0.9) and the min effect size cutoff (default=1.25) - then the Bayesian system can be easily toggled and optimized."*

> *"For the purpose of summit detection, we might find that raising the cutoff helps confine the ODW to the most active stages with the best summit signal. And messing around with the probability tolerance level might come in handy for summit optimization as well."*

> *"So this is really cool - we now have plans for a Bayesian Origin Detection Window System."*

**Decisions formalized:**

1. **Flag-name scope generalized.** The detector flag is NOT `--last-active-stage-detector` (too narrow). It controls a system-level test whose outputs feed multiple metrics. Recommended naming uses an `--odw-` prefix to group it with the sibling tunables. Final name TBD by SPEC engineering; user-suggested options include `--active-stage-detector`, `--odw-active-stage-detector`, or similar.

2. **Three formulations all implemented as flag options.** Default `bayesian_posterior`. The other two (`hardthreshold`, `gaussian_overlap`) are available for empirical comparison and edge cases.

3. **Two sibling CLI flags expose the tunable knobs:**

   - `--odw-prob-threshold` (default 0.9) — evidence-strength knob (probability threshold). Used by `gaussian_overlap` and `bayesian_posterior` detectors.
   - `--odw-fold-threshold` (default 1.25) — effect-size meaningfulness knob (min fold-change). Used by `hardthreshold` and `bayesian_posterior` detectors.

   The two knobs are biologically and statistically distinct. The `bayesian_posterior` default uses BOTH (posterior probability ≥ 0.9 that effect size ≥ 1.25 — tightest combo).

4. **Tuning use-case explicit.** Raising either knob narrows the ODW to higher-confidence stages → better summit signal for BRAIN15.8 stage-selection (strategies 4 / 5 / 6). Loosening widens the ODW → better amplicon recall. The Bayesian ODW System's knobs are now the **primary tuning surface for summit optimization in Phase 15.**

5. **System framing recorded.** "Bayesian Origin Detection Window System" is the official name for the framework as a whole. Codifies that the per-transition growth-detection test isn't a single-purpose detector but a parameterized statistical system whose outputs feed `onset_stage`, `last_active_stage`, the per-transition active mask, amplicon-class classification, and (potentially) `regression_stage` if SPEC engineering chooses to unify direction.

**BRAINSTORM updates applied for the system formalization:**

- **BRAIN15.7 Round 6 deliverable item 6** — heading reframed to "Bayesian Origin Detection Window (ODW) System." Concrete CLI surface table added (detector flag + two sibling knobs). Knob-tuning use case for summit optimization made explicit. Cross-reference to BRAIN15.8 stage-selection. Three formulations clearly differentiated (which knobs each consumes).
- **BRAIN15.24 four-metric framework** — Gaussian-overlap connection note rewritten to reference the Bayesian ODW System framework and its tunable knobs. The system's outputs feed `onset_stage` / `last_active_stage` / per-transition active mask / amplicon-class / (optionally) `regression_stage`.
- **BRAIN15.8 stage-selection menu** — added a "Bayesian ODW System tunables as summit-optimization knobs" sub-section. The evaluation matrix should sweep both the per-amplicon strategy menu AND `--odw-prob-threshold` × `--odw-fold-threshold` values. Phase 15 SPEC engineering treats this as the primary summit-optimization tuning surface.

**Phase 15 implication:** the BRAIN15.7 deliverable item 6 has graduated from "consider this" to "implement this as a parameterized system with a CLI surface." SPEC engineering inherits a fully-spec'd system rather than a vague "use a probabilistic test" instruction.

### Round 6 addendum 3 (ODW argparse-group decision, 2026-04-28)

User decision on CLI architecture for the Bayesian ODW System flags. Verbatim user direction (chat 2026-04-28):

> *"Should the CLI flags go in the timing argparse group? Or should they form a new Origin Detection Window argparse group? I think I am in favor of the new ODW group, and the header information for the group can say the results effect timing analyses (and APS analyses or others if it touches them too)."*

**Decision recorded:** the three Bayesian ODW System flags (`--odw-active-stage-detector`, `--odw-prob-threshold`, `--odw-fold-threshold` — final names TBD by SPEC engineering) form a **new "Origin Detection Window" argparse group**, NOT additions to the existing timing group. Group-header text follows the **Universal-in-spirit pattern** (per BRAIN15.16; Phase 14 Supplemental 14-S20) and explicitly enumerates the downstream analyses affected:

- Timing analyses (onset_stage, last_active_stage, amplicon_class — `onionskin_core/timing.py`).
- Summit refinement (BRAIN15.8 stage-selection strategies).
- Amplicon reliability scoring (BRAIN15.7 — credibility evaluation uses ODW).
- Multistage unification (BRAIN15.18 — within-ODW growth/no-growth detection).
- Summit↔timing convergence (BRAIN15.29 — active stages = ODW).
- APS analyses (BRAIN15.27 — if SPEC engineering wires APS feature computation to ODW-confined inputs).

**Rationale:** putting the flags in the timing group would mis-suggest single-domain scope. ODW is genuinely cross-cutting — its boundaries flow through six+ downstream analysis surfaces. The new dedicated group makes the cross-cutting scope discoverable and follows the Universal-in-spirit pattern for cross-pipeline / cross-domain CLI groups.

**BRAINSTORM updates applied:**

- **BRAIN15.7 deliverable item 6** — new "CLI architecture: new ODW argparse group" sub-section added before the CLI surface table. Includes suggested group-header text enumerating downstream surfaces.
- **BRAIN15.16 (APS Universal-in-spirit re-framing forward extension)** — Round 6 update added: the ODW group joins the cross-cutting argparse-group list (alongside APS, Timing, Shape scoring, Overlap, Asymmetric Triangle Model). When SPEC engineering finalizes the ODW group's flags + header text, BRAIN15.16's forward-extension list formally includes the ODW group as a Universal-in-spirit-framed group.

**Phase 15 SPEC engineering inherits a fully-spec'd CLI architecture decision** — group placement, header-text framing pattern, three flags with defaults, three detector formulations with knob-composition rules. SPEC engineering only needs to settle final flag names + final header wording + which detector is the absolute default.

## Agent 2 narrow re-audit after Round 6 — 2026-04-28

**Auditor:** Codex, Agent 2.

**Scope:** Narrow re-audit of Agent 1's Round 6 response to the Codex brainstorming-stage code/tracking audit, plus a light closeout scan for obvious remaining bookkeeping drift. I did not edit `PHASE15_BRAINSTORM.md` or source code while performing this audit pass; after appending this FEEDBACK section, I refreshed `HANDOFF.md` / `TASK.md` bookmarks so future cold-start agents do not chase already-addressed Round 6 work.

**Verdict:** **SUBSTANTIVELY CLEAN — one small bookkeeping correction before closeout.** Agent 1 adequately addressed the prior substantive concerns. I do not see a remaining missing HMM-completeness pillar or a need for another broad brainstorming pass. The only remaining issue I found is a stale/nonexistent version label (`v0.14.77`) that should be removed from current planning text before final brainstorming-stage closeout.

### Findings

1. **Prior Codex audit findings are adequately addressed.** Round 6 incorporated all eight prior findings into existing BRAIN entries without creating unnecessary new BRAIN IDs. The BRAINSTORM now names the concrete HMM step-renumbering/path targets, promotes live `timing.py` `latest_activity_stage` / `amplicon_class` machinery from archaeology target to known implementation input, separates APS sample posterior clustering from HMM trajectory/fork clustering, folds in `scripts/aps_cluster_experiments.py`, names the HMM-side `summit_excess` resolver, fixes the archived-SOUP source path, and names the HMM notebook generator modernization target.

2. **The Round 6 ODW additions are coherent and SPEC-ready.** The Gaussian-overlap / Bayesian posterior discussion has become a clear Bayesian Origin Detection Window System with three detector modes, two tuning knobs, and a dedicated ODW argparse group. That is broader than my original audit item, but the expansion is explicitly user-driven in the Round 6 addenda and harmonizes with BRAIN15.7, BRAIN15.8, BRAIN15.16, BRAIN15.18, BRAIN15.24, BRAIN15.27, and BRAIN15.29.

3. **BRAIN ID / Source formatting remains consistent.** The BRAINSTORM scan still shows one `Source:` line per BRAIN entry and BRAIN IDs remain sequential through BRAIN15.34. Round 6 correctly reports no new BRAIN IDs.

4. **Tracking-file cleanup should remain in BRAIN15.34, not become a pre-SPEC blocker.** The older tracking files still contain entries that Phase 15 will eventually close or migrate, but BRAIN15.34 explicitly owns that final cleanup pass. I do not see a high-confidence tracking idea from this narrow pass that is both Phase-15-relevant and missing from the BRAINSTORM.

5. **Bookkeeping issue: stale `v0.14.77` reference remains.** `multi-agent/plans/PHASE15_BRAINSTORM.md:6` says the BRAINSTORM was iterated through "Codex Agent 2 brainstorming-stage code/tracking audit at v0.14.77". That version label was intentionally removed from the CHANGELOG flow and should not be used for this brainstorming-stage audit. The same stale label appears in Agent 1's Round 6 triage narrative at `multi-agent/plans/PHASE15_FEEDBACK.md:1485` (the action note describing the lifecycle-position update). Suggested replacement wording for the BRAINSTORM lifecycle line: "Codex Agent 2 brainstorming-stage code/tracking audit (2026-04-28)" or "Codex Agent 2 audit recorded in FEEDBACK (2026-04-28)", with no version number.

6. **Project-context bookmarks needed refresh after Round 6.** `multi-agent/project_context/HANDOFF.md` and `multi-agent/project_context/TASK.md` described the pre-Round-6 state as if Agent 1 follow-up were still the next action. This was not a BRAINSTORM content blocker. After appending this FEEDBACK section, I refreshed those bookmarks so the next cold-start agent sees: Round 6 addressed the substantive audit, only the `v0.14.77` bookkeeping cleanup remains, then the stage can close or ratchet to SPEC when the user says "go SPEC."

### Recommendations

1. Agent 1 or the orchestrator should remove the stale `v0.14.77` wording from `PHASE15_BRAINSTORM.md:6`. If editing historical FEEDBACK triage text is allowed, also correct the same stale label in the Round 6 action note; otherwise this re-audit section is enough to document the correction.
2. After the `v0.14.77` cleanup, optionally touch `HANDOFF.md` / `TASK.md` one more time to remove that final bookkeeping-remains note; otherwise the current bookmarks are accurate enough to guide the next cold start.
3. After those tiny bookkeeping updates, I would treat Phase 15 brainstorming as ready for closeout and SPEC engineering. No further broad Agent 2 audit seems necessary.

**Tests run:** none; planning/code-reading audit only.

**Sign-off:** Codex, Agent 2.

---

## SPEC engineering Round 1 — Agent 1 first-pass authoring (2026-04-28)

**Author:** Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High), acting as Agent 1.

**Action:** Authored `multi-agent/plans/PHASE15_SPEC.md` from scratch using the 34-BRAIN-entry brainstorming-stage output as input. Followed the v2 substantive-priorities rule + phase-group SPEC structure per `multi-agent/workflows/phase-development-system_PDS-v2.md`.

**SPEC structure produced:**

- **20 substantive priorities** (SPEC15.1 through SPEC15.20).
- **9 phase-groups** (Phase 1–9; Phase 0 holds 4 RESOLVED markers without SPEC IDs).
- **Each phase-group becomes one batched cycle** in the audit/implement stage (cycles 15.1a through 15.9a).
- **Coverage:** all 34 BRAIN entries cited mechanically auditable via `grep -nE "BRAIN15\.[0-9]+" PHASE15_SPEC.md` (138 references total).
- **5 framing/preamble entries** (BRAIN15.1–5) become SPEC preamble (Phase scope + Core design rules + Active carry-over + Non-goals + Completeness matrix umbrella) rather than standalone priorities.
- **Universal naming + design decisions section** locks 9 cross-cutting decisions inherited from BRAINSTORM (rename target locking, term retirements, framework selection, structural conventions, standing rules) so they are not re-litigated during audit/implement.

**Phase-group breakdown:**

| Phase | Theme | Cycle | Priorities |
|---|---|---|---|
| 0 | RESOLVED markers (no implementation) | (no cycle) | BRAIN15.13 + 15 + 30 + 32 — no SPEC IDs |
| 1 | HMM CLI correctness | 15.1a | SPEC15.1 |
| 2 | Foundational pipeline architecture | 15.2a | SPEC15.2, SPEC15.3 |
| 3 | Bayesian ODW System (cross-cutting framework) | 15.3a | SPEC15.4 |
| 4 | HMM shape-filter + multistage unification + amplicon reliability | 15.4a | SPEC15.5, SPEC15.6 |
| 5 | HMM summit refinement + summit↔timing convergence | 15.5a | SPEC15.7, SPEC15.8 |
| 6 | APS analytical work + clustering defaults finalization | 15.6a | SPEC15.9, SPEC15.10, SPEC15.11, SPEC15.12 |
| 7 | HMM-specific late-phase work | 15.7a | SPEC15.13, SPEC15.14, SPEC15.15, SPEC15.16 |
| 8 | Cross-pipeline analysis-surface parity | 15.8a | SPEC15.17, SPEC15.18 |
| 9 | Phase 15 closeout | 15.9a | SPEC15.19, SPEC15.20 |

**Key consolidations (interpretive choices that may need user/Agent 2 review):**

1. **Bayesian ODW System split out from BRAIN15.7 to SPEC15.4** as its own priority. Rationale: cross-cutting framework consumed by multiple Phase 4+ priorities; lands earlier in implementation order so consumers inherit a stable framework + naming. SPEC15.6 (BRAIN15.7's main amplicon-reliability scope) cross-references SPEC15.4 rather than embedding the framework.
2. **BRAIN15.22 (IBM-C5 cross-pipeline ODW) folded into SPEC15.4** rather than its own priority. Rationale: BRAIN15.22's IBM-C5 implementation IS the Bayesian ODW System (timing.py is shared cross-pipeline); no need for a separate SPEC priority.
3. **BRAIN15.16 Round 6 update (ODW group joins Universal-in-spirit list) folded into SPEC15.4.** Rationale: the ODW group framing decision lives where the ODW group is defined.
4. **BRAIN15.23 (`indiv_samples/` cleanup) folded into SPEC15.3** per the BRAIN entry's own intent.
5. **BRAIN15.5 (HMM completeness matrix) treated as SPEC preamble** rather than a standalone priority. Rationale: umbrella + cross-reference index; standalone deliverables are folded into the priorities that own each gap or into the Universal naming + design decisions section.
6. **BRAIN15.33 item 3 (`early-parabola-mean` follow-up) split between SPEC15.7 (validation work) and SPEC15.19 (issue closeout).** Rationale: ODW-confined input pattern (SPEC15.7) partially supersedes the issue's recommendation; closeout still happens in SPEC15.19's housekeeping bundle.
7. **BRAIN15.33 out-of-scope `[ISSUE:2026-04-24:2]` (RMS summit parabola pre-smoothing) folded into SPEC15.7** as a parabola-smoothing methodology audit deliverable, per the BRAIN entry's own user-quoted note that this is HMM-relevant.

**Open questions surfaced for Agent 2 review (JQ25–JQ33; documented in SPEC body § "Open questions for FEEDBACK"):**

- **JQ25** — Phase-group count and granularity (9 groups / 20 priorities). Alternative: consolidate Phase 6 into 18 priorities total. Reviewer judgment requested.
- **JQ26** — Bayesian ODW System ownership location (consolidated in SPEC15.4 vs split across SPEC15.4 + SPEC15.6).
- **JQ27** — `--hmm-0-based-statepath` default-flip placement (SPEC15.14 with default off + follow-on flip vs all-in-one priority).
- **JQ28** — `regression_stage` detector framework integration (implicit direction-reversal under one flag vs sibling flag).
- **JQ29** — Cross-pipeline summit↔timing convergence (Phase 15 follow-on priority vs explicit punt to future phase).
- **JQ30** — APS catalog file location (`multi-agent/full_instructions/APS_CATALOG.md` recommended vs tracking surface).
- **JQ31** — Class-aware reliability scoring detail level (more concrete spec needed before audit/implement?).
- **JQ32** — Stage-1 growth-detection mechanism (more concrete than "SPEC engineering decides" before audit/implement?).
- **JQ33** — Phase 15 priority count concern (20 priorities is on the high side of "5–10"; comparable to Phase 14 Supplemental's 30 → 6 phase-groups but flagged for reviewer).

**Self-audit findings (Agent 1 SPEC-vs-BRAINSTORM challenge pass):** documented in SPEC body § "Self-audit (Agent 1 SPEC-vs-BRAINSTORM challenge pass)". 13 findings total; 7 addressed in SPEC during authoring (Findings 1–7); 6 raised for FEEDBACK / reviewer judgment (Findings 8–13, mapped to JQ25–JQ32 above).

**Substantive design content NOT re-litigated** (per `multi-agent/project_context/DECISIONS.md` 2026-04-28 entry + brainstorming-stage decisions): APS feature-mode candidates accepted/rejected catalog (Fourier descriptors, curvature features, multi-family-BIC vectors, `peak_rcn` as a distinct feature — all rejected with rationale); RCN-vs-excess design decision; Bayesian ODW System CLI architecture (3 detectors + 2 knobs + new `--odw-*` group); 4-metric naming framework; `peak_rcn_stage` retirement; `summit_excess` → `summit_rcn` rename; `latest_activity_stage` → `last_active_stage` rename; step-less `plots/` and `notebooks/` cross-pipeline structural change; KNOWN_ISSUES migration-with-provenance convention. These are LOCKED at SPEC engineering close.

**Tests run:** none; SPEC-authoring round is planning work only. The audit/implement stage will cover code testing per CLAUDE.md test matrix.

**No CHANGELOG/DEVLOG entry written** per the no-LOG-during-iteration rule (LOG entry comes at SPEC engineering closeout cycle).

**Next action:** Awaiting Agent 2 (Codex or other reviewer) to author the SPEC-engineering Round 1 audit at the bottom of this FEEDBACK file, addressing the open questions above + auditing the SPEC for completeness, ordering, substantive-priorities compliance, and any gaps relative to BRAINSTORM. Per PDS-v2 SPEC engineering Agent 2 prompt, Agent 2 should NOT directly edit the SPEC; findings go in FEEDBACK; Agent 1 then triages and applies in a follow-up SPEC engineering round.

**Sign-off:** Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High), Agent 1.

---

## SPEC engineering Round 1 audit — Agent 2 (Codex) — 2026-04-28

**Auditor:** Codex, Agent 2.

**Scope:** Audited `multi-agent/plans/PHASE15_SPEC.md` against `multi-agent/plans/PHASE15_BRAINSTORM.md` for BRAIN coverage, dependency order, substantive-priority compliance, and SPEC-lock readiness. I did not edit `PHASE15_SPEC.md` or `PHASE15_BRAINSTORM.md`.

**Verdict:** **OPEN — Agent 1 follow-up pass needed before SPEC lock.** The SPEC is strong and mostly well-constructed: all 34 BRAIN IDs are mechanically cited, the phase-group structure is sensible, and most dependency order is correct. I do not recommend another broad brainstorm. However, I found a few real SPEC-construction issues that should be corrected before the BRAINSTORM is archived and the audit/implement stage opens.

### Mechanical checks

- **BRAIN coverage:** PASS. `rg -o "BRAIN15\.[0-9]+" PHASE15_BRAINSTORM.md | sort -uV` and the same command on `PHASE15_SPEC.md` both enumerate BRAIN15.1 through BRAIN15.34.
- **SPEC ID shape:** PASS. SPEC15.1 through SPEC15.20 are sequential and all have `Covers:` fields.
- **Substantive-priority structure:** MOSTLY PASS. 20 priorities is high relative to the 5-10 target, but the 9 phase-groups make the implementation cadence reasonable. I would not collapse the phase further just to hit a lower numeric count; Phase 15 is genuinely wide.
- **Ordering:** MOSTLY PASS, with one important SAPS/clustering ordering concern below.

### Specific findings

1. **Missing or under-owned BRAIN15.5 matrix row: width/fork progression.**

   `PHASE15_BRAINSTORM.md:110` marks **Width progression** as a gap and says Phase 15 should produce HMM-equivalent progression outputs both as HMM-specific state-path/leading-edge/per-doubling-fork outputs and as growth-analogous outputs where meaningful. The SPEC says the BRAIN15.5 matrix is the deliverable checklist (`PHASE15_SPEC.md:72`), but I do not see a SPEC priority that owns this progression-output family. SPEC15.5 covers shape filtering, multistage unification, and state-path credibility; SPEC15.17 covers plots/notebooks; neither explicitly requires a dedicated HMM progression table/track analogous to growth's progression step or the HMM-specific leading-edge/fork-progression form.

   **Recommendation:** Add an explicit deliverable to SPEC15.5 or a small sub-section in SPEC15.17/SPEC15.5 split across outputs + plots:
   - Emit HMM fork/width progression outputs from state-path-derived leading edges and per-doubling-round fork positions.
   - Emit a growth-analogous width-progression output when it produces meaningfully distinct information.
   - Add corresponding plots/notebook surfaces under SPEC15.17 if visualizations are expected.

2. **HMM trajectory/fork clustering is acknowledged but not fully specified as a deliverable.**

   BRAIN15.19 Round 6 says the SPEC must distinguish APS sample posterior clustering from HMM trajectory/fork clustering and also states that both are legitimate Phase 15 deliverables (`PHASE15_BRAINSTORM.md:958-970`). SPEC15.12 correctly distinguishes the concepts (`PHASE15_SPEC.md:673-697`), but its actual inventory/default-optimization deliverables appear to target APS sample posterior clustering only: `onionskin_core/aps.py`, `hmm_ported_analyses.py:149-184`, and APS feature modes (`PHASE15_SPEC.md:681-701`). It does not explicitly audit or tune `trajectory_feature_matrix.tsv` / `trajectory_clusters.tsv` or `hmm_ported_analyses.py:380-439`.

   **Recommendation:** Update SPEC15.12 to include a distinct HMM trajectory/fork-clustering audit deliverable:
   - Inventory existing trajectory feature columns and clustering defaults.
   - Decide whether defaults need tuning separately from APS clustering.
   - Validate trajectory clustering output after SPEC15.3 renumbering and after any fork/width progression additions from finding 1.
   - Keep docs/help text explicitly separate from APS sample posterior clustering.

3. **Ordering concern: SAPS may need to land before final clustering defaults if it can feed clustering.**

   BRAIN15.20 says SAPS depends on BRAIN15.6 only and places SAPS at `15-saps/` before timing/clustering (`PHASE15_BRAINSTORM.md:982-990`). SPEC15.13 implements SAPS in Phase 7, after SPEC15.12 finalizes clustering defaults in Phase 6. But SPEC15.13 deliverable 5 says SAPS can feed the same APS sample posterior clustering machinery, or remain distinct, with the decision deferred to implementation (`PHASE15_SPEC.md:733`). If SAPS may feed clustering, then final clustering defaults should not be finalized before SAPS exists.

   **Recommendation:** Pick one of two clean options:
   - Move SAPS earlier, after SPEC15.10 and before SPEC15.11/SPEC15.12, so clustering-default finalization can include or explicitly exclude SAPS with real outputs available.
   - Or keep SAPS in Phase 7 but state clearly in SPEC15.13 and SPEC15.12 that SAPS is **not** part of Phase 15 clustering-default finalization; any APS+SAPS combined clustering is future work unless the user explicitly broadens Phase 15.

4. **`summit_inspector.py` step-renumber ownership is split ambiguously.**

   SPEC15.3 correctly lists `scripts/summit_inspector.py:153-158` among step-renumbering blast-radius targets (`PHASE15_SPEC.md:232-237`). SPEC15.19 then says to "Bundle the SPEC15.3 step-renumber update for `summit_inspector.py:153-158` here" (`PHASE15_SPEC.md:996`). That risks leaving a script path stale until Phase 9 even though the step renumbering happens in Phase 2.

   **Recommendation:** Make ownership explicit:
   - SPEC15.3 owns all step-number/path updates in `summit_inspector.py`.
   - SPEC15.19 owns only the ModuleNotFoundError/import-path fix and similar script-invocation hardening.
   This keeps the repo coherent immediately after the renumbering cycle.

5. **SPEC15.15 pass1/pass2 layout should be decided before SPEC lock, not left to implementation.**

   SPEC15.15 defers the pass1/pass2 directory layout to implementation (`PHASE15_SPEC.md:807-811`). This is a pipeline-output contract, not a small local implementation detail, and it intersects the user's "Atomize steps" and "No backwards cramming" aims. The SPEC can still allow implementation-time refinement, but it should name the intended output contract before audit/implement opens.

   **Recommendation:** Resolve the layout now in SPEC engineering. I lean toward the existing recommended option (B): final pass-2 outputs live at the normal step paths, pass-1 artifacts are archived under `pass1/`, and a sentinel/metadata file records the two-pass provenance. Also state that no later-stage derived file is silently written into an upstream directory without pass/provenance labeling.

6. **Some open questions are real SPEC-lock questions, not implementation details.**

   I agree with Agent 1 surfacing JQ25-JQ33. My recommendations:

   - **JQ25/JQ33 (count/granularity):** keep 20 priorities / 9 phase-groups. The grouped-cycle cadence is reasonable for this scope.
   - **JQ26 (ODW ownership):** keep ODW centralized in SPEC15.4. Correct dependency direction.
   - **JQ27 (0-based statepath default flip):** do not silently punt the default flip to a future phase. Add a conditional deliverable in SPEC15.14: land the flag, validate, and either flip default in the same cycle if validation passes or record explicit user-approved deferral.
   - **JQ28 (`regression_stage` detector):** prefer one detector family under the ODW group, with direction reversal handled internally. Add a sibling flag only if implementation/audit finds regression needs different defaults.
   - **JQ29 (cross-pipeline summit↔timing):** current HMM-first scope is okay because BRAIN15.29 explicitly framed cross-pipeline extension as deferred follow-on if HMM proves the pattern. Add one sentence saying growth/RMS extension is future-phase unless user opts in now.
   - **JQ30 (APS catalog location):** use `multi-agent/full_instructions/APS_CATALOG.md`. It is a maintained cross-pipeline reference, not a transient tracking note.
   - **JQ31 (class-aware reliability):** acceptable as-is if SPEC15.6 adds acceptance criteria rather than a full formula. I would add a minimum rule: `Founder` / `Constitutive_Prolonged` exclusions require stronger evidence than low-confidence classes, and the 4 DS1 chr II calibration amplicons must remain weight ~= 1.0.
   - **JQ32 (stage-1 growth detection):** make this more concrete before lock. Recommendation: stage 1 uses a background/chrom-median comparison under the same variance-aware primitive and `--odw-prob-threshold`, while stages 2+ use the selected per-transition detector. This avoids leaving a central ODW boundary rule vague.

### Overall assessment

This is not a failed SPEC; it is a good first-pass SPEC with a few gaps exposed by its own thoroughness. After addressing the width/fork progression ownership, trajectory-clustering deliverable, SAPS/clustering order, `summit_inspector.py` ownership split, pass1/pass2 layout, and the key JQ lock decisions, I would expect the SPEC to be ready for a narrow follow-up audit rather than another broad reconstruction.

**Tests run:** none; planning/spec audit only.

**Sign-off:** Codex, Agent 2.

---

## SPEC engineering Round 2 — Agent 1 triage of Codex audit + own-pass additions (2026-04-28)

**Author:** Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High), acting as Agent 1.

**Action:** Triaged Codex's SPEC engineering Round 1 audit (FEEDBACK § "SPEC engineering Round 1 audit — Agent 2 (Codex) — 2026-04-28"). Applied all 6 Codex findings + 8 JQ resolutions. Added 4 own-pass self-audit additions per the prompt's request to also audit cross-codebase impacts. Result: SPEC is structurally complete, dependency-correct, and substantively locked at SPEC engineering Round 2 close.

### Codex findings — all 6 ACCEPTED + applied

| # | Codex finding | Resolution applied | SPEC location |
|---|---|---|---|
| 1 | BRAIN15.5 width/fork progression gap not owned | Added SPEC15.5 sub-priority (f) for HMM fork/width progression outputs (state-path-derived + growth-analogous variants); plot/notebook surfaces in SPEC15.17 deliverable 11 (added) | SPEC15.5 (f); SPEC15.17 (11) |
| 2 | HMM trajectory/fork clustering not fully specified | Restructured SPEC15.12 into Track 1 (APS sample posterior clustering, cross-pipeline) + Track 2 (HMM trajectory/fork clustering, HMM-specific) + Track 3 (SAPS scope lock) | SPEC15.12 deliverables 1–9 |
| 3 | SAPS ordering vs clustering defaults | LOCKED via Codex option (b): SAPS NOT part of Phase 15 clustering-default finalization; combined APS+SAPS clustering = future-phase | SPEC15.12 Track 3 + SPEC15.13 deliverable 5 |
| 4 | `summit_inspector.py` ownership split ambiguous | CLARIFIED: SPEC15.3 owns step-renumber updates (immediate post-renumber coherence); SPEC15.19 owns ONLY ModuleNotFoundError/import-path fix | SPEC15.3 deliverable 9 + SPEC15.19 deliverable 2 |
| 5 | SPEC15.15 pass1/pass2 layout decision | LOCKED option (B): final pass-2 outputs at top-level step paths; pass-1 archived under `pass1/`; sentinel/metadata file (`.pass_metadata.json`) for atomicity + provenance; no silent cramming rule | SPEC15.15 deliverable 4 |
| 6 | JQ lock-time decisions | All 8 JQ resolutions applied per Codex recommendations (see JQ table below) | Universal naming + design decisions rules 10–19 |

### JQ resolutions — all 8 LOCKED at Round 2

| JQ | Codex recommendation | Round 2 resolution | SPEC lock location |
|---|---|---|---|
| JQ25 | Keep 20/9 (reasonable for scope) | ACCEPTED — no SPEC change | n/a (kept as-is) |
| JQ26 | Keep ODW centralized in SPEC15.4 | ACCEPTED — no SPEC change | n/a (kept as-is) |
| JQ27 | Land flag, validate, conditionally flip in SAME cycle OR record explicit deferral; no silent punt | LOCKED conditional structure (7a/7b/7c) | SPEC15.14 deliverable 7; design-decisions rule 12 |
| JQ28 | Same `--odw-active-stage-detector` with internal direction-reversal; sibling flag only if needed | LOCKED single detector family | SPEC15.4 deliverable 4; design-decisions rule 11 |
| JQ29 | Cross-pipeline summit↔timing = future-phase unless user opts in | LOCKED HMM-only Phase 15 scope | SPEC15.8 deliverable 9; design-decisions rule 16 |
| JQ30 | Use `multi-agent/full_instructions/APS_CATALOG.md` | LOCKED location | SPEC15.17 deliverable 4; design-decisions rule 17 |
| JQ31 | Acceptable as-is + add minimum rule (Founder/Constitutive_Prolonged require stronger evidence; 4 calibration amplicons must remain weight ≈ 1.0) | LOCKED minimum rule + regression gate | SPEC15.6 deliverable 8; design-decisions rule 18 |
| JQ32 | Stage 1 uses background/chrom-median comparison under same variance-aware primitive + same `--odw-prob-threshold` | LOCKED stage-1 mechanism | SPEC15.4 implementation notes; design-decisions rule 10 |
| JQ33 | Keep 20 priorities (matches Phase 14 Supplemental's 30/6 phase-groups pattern) | ACCEPTED — no SPEC change | n/a (kept as-is) |

### Codex findings — REJECTED: NONE

All 6 Codex substantive findings + all 8 JQ recommendations were accepted. No findings rejected.

### Own-pass Round 2 additions (Agent 1 self-audit beyond Codex's findings)

Per the prompt's request to also audit cross-codebase impacts:

- **Finding A — Makefile sweep added to SPEC15.3.** Step-renumbering blast radius now includes `Makefile` + `make` targets (previously only `tests/test_pipeline.py:179` was named explicitly). Validation: "verify all `make` targets that exercise HMM still run cleanly post-renumber."
- **Finding B — User-facing doc cross-alignment sweep added to SPEC15.20 (deliverable 9).** README.md + PIPELINE_SPEC.md + ONIONSKIN_FULL_HANDOFF.md + new APS_CATALOG.md alignment audit at Phase 15 closeout. Bundles the long-deferred `ONIONSKIN_FULL_HANDOFF.md` version-header refresh from TASK.md. Eliminates the silent-doc-drift failure mode.
- **Finding C — Stage 1 detection mechanism + regression_stage detector explicit lock language.** Beyond just resolving JQ32 + JQ28, the lock language in design-decisions rules 10 + 11 is structurally explicit so audit/implement can't reintroduce ambiguity.
- **Finding D — `--hmm-0-based-statepath` default-flip protocol structurally codified.** Beyond just resolving JQ27, the SPEC15.14 deliverable 7 (7a/7b/7c structure) eliminates the "land flag and forget" failure mode by requiring conditional flip OR explicit deferral with reason.

### v2 substantive-priorities verification (Round 2)

- 20 priorities / 9 phase-groups — UNCHANGED from Round 1.
- Codex Round 1 audit confirmed structure as "reasonable for this scope" (line 1724–1725 of FEEDBACK).
- No thin priorities standing alone:
  - SPEC15.1 = bundles BRAIN15.9/10/11 (3 entries).
  - SPEC15.5 = 6 sub-priorities (a–f) covering BRAIN15.18 + part of BRAIN15.5.
  - SPEC15.6 = 13 deliverables covering BRAIN15.7 (full priority).
  - SPEC15.12 = 9 deliverables across 3 tracks covering BRAIN15.19.
  - SPEC15.17 = 11 deliverables covering BRAIN15.17 + BRAIN15.5 plot row + BRAIN15.32 enhancement audit.
  - SPEC15.19 = bundles 4 housekeeping items (BRAIN15.33).
- The 4 RESOLVED markers in Phase 0 carry NO SPEC IDs — they are no-implementation closeout markers, not thin priorities.

### Mechanical re-audit post-Round-2

- All 34 BRAIN IDs still cited (BRAIN15.1–34): PASS via `grep -oE "BRAIN15\.[0-9]+" PHASE15_SPEC.md | sort -uV`.
- SPEC IDs still sequential (SPEC15.1–20): PASS.
- All `Covers:` fields still present: PASS.
- Dependency ordering (Round 2 verified):
  - SPEC15.5 sub-priority (f) (Phase 4) → SPEC15.17 deliverable 11 (Phase 8) ✓
  - SPEC15.5 sub-priority (f) (Phase 4) → SPEC15.12 Track 2 hard prereq list (Phase 6) ✓
  - SPEC15.3 (Phase 2) → SPEC15.19 deliverable 2 (Phase 9) clean post-renumber tree ✓
  - All other dependencies unchanged from Round 1 (verified in Round 1 self-audit).

### Files touched in Round 2 (Agent 1 follow-up)

- `multi-agent/plans/PHASE15_SPEC.md` — Round 2 edits across SPEC15.3 / SPEC15.4 / SPEC15.5 / SPEC15.6 / SPEC15.8 / SPEC15.12 / SPEC15.13 / SPEC15.14 / SPEC15.15 / SPEC15.17 / SPEC15.19 / SPEC15.20 + Universal naming + design decisions section (rules 10–19 added) + Open questions section (RESOLVED table) + Self-audit Round 2 section (added).
- `multi-agent/plans/PHASE15_FEEDBACK.md` — this Round 2 entry appended.
- `multi-agent/project_context/HANDOFF.md` — cold-start orientation updated to Round 2 complete + readiness for narrow Agent 2 re-audit.
- `multi-agent/project_context/TASK.md` — immediate-tasks queue updated.

### Round 2 readiness assessment

**SPEC is in good shape for a narrow Agent 2 re-audit** (Round 3) — the targeted kind, not a fresh full audit. Recommend the narrow re-audit verify:
1. The 6 Codex findings are correctly addressed (no regressions, no missed sub-deliverables).
2. The 8 JQ locks are consistent across SPEC body + Universal naming + design decisions section.
3. The 4 own-pass additions don't create new gaps (especially the SPEC15.20 deliverable 9 doc-sweep + the SPEC15.5 sub-priority (f) cross-references).

If the narrow re-audit comes back CLEAN, the SPEC is ready for SPEC-engineering closeout + audit/implement stage opening (per the user's "we are not done with SPEC engineering until I say so" gate + "We are not ready to move on to implementation until I say so" gate).

### No new open questions surfaced in Round 2

Future SPEC engineering rounds may surface JQ34+ as needed.

**Tests run:** none; SPEC-engineering work only.

**No CHANGELOG/DEVLOG entry written** per the no-LOG-during-iteration rule (LOG entry comes at SPEC engineering closeout cycle).

**Sign-off:** Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High), Agent 1.

---

## SPEC engineering Round 3 audit — Agent 2 (Codex) — 2026-04-29

**Auditor:** Codex, Agent 2.

**Scope:** Narrow re-audit after Agent 1's Round 2 follow-up, plus the new user direction immediately preceding this audit: **the user requested that five concepts be generalized to all pipelines now in Phase 15, not deferred**. Those five are ODW / active-stage detection; summit stage-selection / ODW-confined summit refinement; summit ↔ timing convergence; amplicon reliability / flat-sample detection; and APS feature modes / clustering defaults. HMM-specific analyses remain HMM-specific where their inputs are HMM-only (SAPS, state-path credibility / ghost levels, narrowest summit-state interval, HMM trajectory/fork clustering).

**Verdict:** OPEN — targeted Agent 1 follow-up pass needed before SPEC-engineering closeout.

### Round 2 fix verification

- ✓ **BRAIN coverage remains mechanically complete.** `BRAIN15.1` through `BRAIN15.34` are all present in both `PHASE15_BRAINSTORM.md` and `PHASE15_SPEC.md`; no BRAIN IDs dropped.
- ✓ **SPEC IDs remain sequential and substantive.** SPEC15.1 through SPEC15.20 are intact; the 20-priority / 9-phase-group structure still looks reasonable. I do not see a thin-priority problem.
- ✓ **Prior Codex Finding 1 fixed.** Width/fork progression is now owned by SPEC15.5 sub-priority (f) at `multi-agent/plans/PHASE15_SPEC.md:383`, with plot/notebook follow-through in SPEC15.17 deliverable 11 at `multi-agent/plans/PHASE15_SPEC.md:994`.
- ✓ **Prior Codex Finding 2 fixed.** SPEC15.12 now separates APS sample posterior clustering from HMM trajectory/fork clustering, with explicit trajectory feature inventory and validation at `multi-agent/plans/PHASE15_SPEC.md:719-734`.
- ✓ **Prior Codex Finding 3 fixed.** SAPS is explicitly kept out of Phase 15 clustering-default finalization at `multi-agent/plans/PHASE15_SPEC.md:736-740` and `multi-agent/plans/PHASE15_SPEC.md:775-777`. This remains acceptable because the user's new all-pipeline direction was about APS feature modes / clustering defaults, not SAPS-specific clustering.
- ✓ **Prior Codex Finding 4 fixed.** `summit_inspector.py` path renumbering is owned by SPEC15.3 at `multi-agent/plans/PHASE15_SPEC.md:239`; import-path hardening remains in SPEC15.19. Clean split.
- ✓ **Prior Codex Finding 5 fixed.** Two-pass HMM renorm layout is locked in the design-decision section at `multi-agent/plans/PHASE15_SPEC.md:92`; no silent backwards cramming.
- ⚠ **Prior Codex Finding 6 mostly fixed, but superseded by new user scope.** JQ decisions were applied, but JQ29's HMM-only summit ↔ timing lock is now stale because the user explicitly opted into all-pipeline generalization in Phase 15.

### New user-scope findings

1. ✗ **SPEC15.8 still defers cross-pipeline summit ↔ timing, contradicting the new user direction.**

   The SPEC still says `SPEC15.8 implements HMM-only` and that growth/RMS extension is future-phase work at `multi-agent/plans/PHASE15_SPEC.md:95`, `multi-agent/plans/PHASE15_SPEC.md:556`, and `multi-agent/plans/PHASE15_SPEC.md:566-568`. That was correct under Round 2's "unless user opts in" condition, but the user has now opted in. The SPEC should state that the four-step summit → timing → active-stage summit refinement → updated timing pattern is a Phase 15 cross-pipeline deliverable for HMM, growth, and RMS, with pipeline-specific mechanics as needed.

2. ✗ **SPEC15.7 still frames summit stage-selection / ODW-confined summit refinement as HMM-only.**

   SPEC15.7's title and goal are HMM-specific (`multi-agent/plans/PHASE15_SPEC.md:469-473`), and its likely flag is `--hmm-summit-stage-selection` at `multi-agent/plans/PHASE15_SPEC.md:477`. The strategy menu itself contains five strategies that are not HMM-exclusive (`max_rcn_stage`, `onset_stage`, `last_active_stage`, all ODW stages, ODW subset by shape quality) plus one truly HMM-specific strategy (narrowest summit-state interval) at `multi-agent/plans/PHASE15_SPEC.md:481-486`. Because the user requested all-pipeline generalization now, SPEC15.7 should be reframed as a shared summit stage-selection / ODW-confined refinement priority across HMM, growth, and RMS, while keeping strategy 6 HMM-only.

3. ⚠ **SPEC15.6 reliability / flat-sample detection needs an explicit all-pipeline implementation contract.**

   SPEC15.6 says "cross-pipeline harmonization" and notes the live classifier currently runs on growth/RMS and should be ported to HMM (`multi-agent/plans/PHASE15_SPEC.md:403`, `multi-agent/plans/PHASE15_SPEC.md:424-427`). That is close, but under the user's new direction it should be stronger: Phase 15 should implement reliability scoring + flat-sample detection as a shared all-pipeline pre-APS contract, with growth/RMS/HMM all emitting comparable sidecar columns and APS filtering semantics. Current wording could be read as "port to HMM and harmonize naming" rather than "generalize and wire across all three."

4. ✓ **ODW / active-stage detection is already correctly generalized.**

   SPEC15.4 is cross-cutting and says `timing.py` is shared and applies to HMM, growth, and RMS at `multi-agent/plans/PHASE15_SPEC.md:264` and `multi-agent/plans/PHASE15_SPEC.md:304`. No additional scope finding here, except downstream priorities should now consume that cross-pipeline ODW contract rather than re-narrowing it.

5. ✓ / ⚠ **APS feature modes and clustering defaults are mostly generalized, but should be guarded against accidental HMM narrowing during follow-up.**

   SPEC15.11 extends the experiment harness to `growth|rms|hmm` at `multi-agent/plans/PHASE15_SPEC.md:673-678`, and SPEC15.12 says defaults are decided across the three pipelines at `multi-agent/plans/PHASE15_SPEC.md:700-717`. That satisfies the user's intent for APS feature modes / clustering defaults. Agent 1 should preserve this scope while editing the neighboring HMM-centric priorities, and should not use SAPS's HMM-only scope as a reason to narrow APS defaults.

### Specific issues to resolve

1. **Replace stale scope locks.** Update the Universal naming + design decisions section rule 16 (`multi-agent/plans/PHASE15_SPEC.md:95`) and the JQ29 resolved table (`multi-agent/plans/PHASE15_SPEC.md:1261`) so they record the new user decision: cross-pipeline summit ↔ timing convergence is Phase 15 scope because the user requested it.

2. **Reframe Phase 5.** The Phase 5 heading and SPEC15.7/SPEC15.8 wording should become "cross-pipeline summit refinement + summit ↔ timing convergence" or equivalent. The HMM-only title at `multi-agent/plans/PHASE15_SPEC.md:464-469` is now too narrow.

3. **Split shared vs HMM-specific summit strategies.** SPEC15.7 should explicitly define:
   - shared stage-selection strategies for all pipelines: `max_rcn_stage`, `onset_stage`, `last_active_stage`, all-ODW, ODW-best-shape;
   - HMM-only add-on: narrowest summit-state interval;
   - shared flag/default architecture or universal flag plus pipeline-specific overrides, rather than only `--hmm-summit-stage-selection`.

4. **Broaden SPEC15.8 files affected and validation.** SPEC15.8 currently lists HMM engine orchestration only (`multi-agent/plans/PHASE15_SPEC.md:558-560`). It should also anticipate growth and RMS orchestration, output directories, canonical summit/timing selection, docs/tests, and baseline effects for all three pipelines.

5. **Strengthen SPEC15.6 all-pipeline reliability wording.** Make the pre-APS reliability / flat-sample sidecar and APS-filtering semantics explicitly all-pipeline, not just a HMM port from growth/RMS. Keep HMM state-path-aware extensions as optional HMM-specific extras layered on top of the shared contract.

### Readiness assessment

The SPEC is close and substantially stronger after Round 2, but it is **not ready for SPEC-engineering closeout** until the new all-pipeline scope direction is integrated. This is a targeted follow-up, not a full rewrite: the existing priority order can probably remain the same because SPEC15.4 already lands the shared ODW framework before SPEC15.6/15.7/15.8/15.11/15.12 consume it.

### Recommendations

1. Agent 1 should do a narrow Round 3 follow-up pass updating SPEC15.6, SPEC15.7, SPEC15.8, the Phase 5 heading, design-decision rule 16, and the JQ29 resolution text.
2. Agent 1 should add one explicit SPEC note like: **"User requested all-pipeline generalization now in Phase 15 for ODW, summit stage-selection / ODW-confined summit refinement, summit ↔ timing convergence, reliability / flat-sample detection, and APS feature modes / clustering defaults."** This prevents future agents from treating the expansion as speculative.
3. After that pass, a narrow Agent 2 re-audit should be enough. I do not see remaining mechanical BRAIN coverage, ordering, or atomization blockers.

**Tests run:** none; SPEC-engineering audit only.

**No CHANGELOG/DEVLOG entry written** per the no-LOG-during-iteration rule.

**Sign-off:** Codex, Agent 2.

---

## SPEC engineering Round 3 — Agent 1 triage of Codex audit + all-pipeline generalization + Phase 16 initialization (2026-04-29)

**Author:** Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High), acting as Agent 1.

**Action:** Triaged Codex's SPEC engineering Round 3 audit (verdict OPEN with 5 findings explicitly framed around the user's all-pipeline generalization direction). Did a deep code dive across all 5 generalize-candidates to determine inherit-by-default vs needs-explicit-wiring per pipeline. Triaged what fits in Phase 15 vs what defers to Phase 16. Applied substantive edits to PHASE15_SPEC.md + initialized `multi-agent/plans/PHASE16_SPEC.md` per user direction (bypass SOUP/BRAINSTORM).

### Code dive findings (per generalize-candidate, per pipeline)

| Concept | Growth state | RMS state | HMM state | Phase 15 work | Phase 16 work |
|---|---|---|---|---|---|
| **ODW / Bayesian framework** | Inherits via shared `timing.py` (controller calls timing at `onionskin.py:443`) | NOT WIRED — no timing call today | Inherits via inline timing call at `run_step15_hmm_timing` (`hmm_ported_analyses.py:311+`) | Add lightweight RMS-timing wiring (mirror HMM's pattern; ~30 LOC) | None |
| **Reliability + flat-sample** | Pre-APS scorer: NEW for all 3. Classifier (`_classify_amplicon`): already runs (via timing) | Pre-APS scorer: NEW. Classifier: gated on RMS-timing wiring | Pre-APS scorer: NEW. Classifier: already runs | Shared scorer + per-pipeline wiring + classifier auto-extends to RMS post-SPEC15.4 dlv 9 | None — full implementation |
| **Summit stage-selection menu** | Strategies 1-5 cross-pipeline; uses `refine_origin_sliding_offset` already (`growth_model_engine.py:504, 527`) | Strategies 1-5 cross-pipeline; uses `refine_summit_parabola` (different algorithm) | Strategies 1-6 (all of them); uses HMM-specific `hmm_summit_refinement.py` | Strategy menu cross-pipeline + ODW-confined refinement on each pipeline's existing summit-refinement code path + HMM sliding-offset port | RMS sliding-offset port (different algorithm; new-algorithm work) |
| **Summit↔timing 4-step convergence** | Has summit refinement + timing | Has summit refinement; needs timing wiring (SPEC15.4 dlv 9) | Has both; richest active-stage data | Cross-pipeline 4-step orchestration; per-pipeline sub-deliverables | None — full implementation |
| **APS feature modes / clustering defaults** | APS shared in `aps.py` + uses shared resolver | APS shared + uses shared resolver | APS shared + has pipeline-side override at `hmm_ported_analyses.py:150-151` (the only known pipeline-side resolver) | Editorial cleanup; verify no other pipeline-side resolver creep | None |

### Codex findings — all 5 ACCEPTED + applied

| # | Codex finding | Resolution applied | SPEC location |
|---|---|---|---|
| 1 | SPEC15.8 still defers cross-pipeline summit↔timing | REVERSED Round 3: cross-pipeline now Phase 15 in-scope per user direction; per-pipeline sub-deliverables added | SPEC15.8 deliverable 9 (rewritten); Phase 5 heading; design-decisions rule 16 superseded by rule 20 |
| 2 | SPEC15.7 still HMM-only framing | Reframed title + goal cross-pipeline; Universal `--summit-stage-selection` flag + 3 pipeline overrides; strategies 1-5 cross-pipeline; strategy 6 HMM-only; per-pipeline sub-deliverables | SPEC15.7 deliverables 1-4; Phase 5 heading |
| 3 | SPEC15.6 reliability not explicit all-pipeline | Reframed title + goal as shared all-pipeline pre-APS contract; identical sidecar schema; classifier auto-extends to RMS post-SPEC15.4 dlv 9; HMM state-path extras kept OPTIONAL on top of shared | SPEC15.6 goal + deliverables 5/6/7 |
| 4 | ODW already generalized | Confirmed; added SPEC15.4 deliverable 9 (RMS-timing wiring) + 10 (cross-pipeline reach lock) | SPEC15.4 deliverable 9 + 10 |
| 5 | APS feature modes already mostly generalized | Editorial verification only — no SPEC body change needed | n/a |

### Triage decision: Phase 15 vs Phase 16

**Phase 15 (in scope per Round 3):**
- ODW/timing wiring for RMS (lightweight ~30 LOC)
- Shared pre-APS reliability scorer + classifier port across HMM+Growth+RMS
- Summit stage-selection strategy menu (1-5 cross-pipeline; 6 HMM-only) + ODW-confined refinement cross-pipeline
- HMM sliding-offset port (per Phase 15 SPEC15.7 deliverable 3)
- Cross-pipeline summit↔timing 4-step convergence
- APS feature-mode + clustering-defaults editorial guard

**Phase 16 (deferred — focused-completion phase; SPEC initialized at `multi-agent/plans/PHASE16_SPEC.md`):**
- SPEC16.1: RMS sliding-offset sub-bin summit refinement port (RMS uses parabola-style refinement today; sliding-offset is a different algorithm; new-algorithm work)
- SPEC16.X (placeholder): Phase-15-implementation-surfaced refinements that emerge during the audit/implement stage and warrant their own focused cycle (default-flip cycles, cross-pipeline consistency fixes, doc completion, etc.)

**Rationale for Phase 16 split (limited to RMS sliding-offset):**
1. RMS uses `refine_summit_parabola` today — different algorithm from sliding-offset.
2. Adding sliding-offset to RMS requires algorithm-level decision (replace / add-alongside / flag-selectable) + RMS-specific input-contract decisions + RMS-specific validation against by-eye + PuffStep gold-standard. These are SPEC-level decisions that benefit from a focused cycle.
3. Phase 15 SPEC15.7's sliding-offset deliverable for HMM is already substantive work (per-sample data prerequisite, summit-refinement integration, new column emission); adding RMS sliding-offset to the same priority would expand scope beyond what the audit-implement-reaudit cycle can comfortably handle.
4. Phase 16 = focused-completion phase. Bypasses SOUP/BRAINSTORM per user direction. Initialized this round; will be polished after Phase 15 audit/implement work surfaces additional items.

### Own-pass Round 3 additions (Agent 1 self-audit beyond Codex's findings)

- **Finding A — Code dive verified per-pipeline current state.** Concrete code-location citations added to SPEC15.4 deliverable 9 (RMS-timing input contract: produce `rms_stage_summary.tsv` from `per_stage_call_dfs` mirroring HMM's pattern at `hmm_ported_analyses.py:341–354`); SPEC15.6 (per-pipeline classifier state); SPEC15.7 (per-pipeline summit-refinement state); SPEC15.8 (per-pipeline summit + timing prerequisites).
- **Finding B — RMS sliding-offset port deferred to Phase 16 with explicit algorithm-level decision flagged.** SPEC16.1 deliverable 1 names the three options (replace / add-alongside / flag-selectable) for SPEC engineering to pick during Phase 16 implementation; recommendation deferred to that cycle.
- **Finding C — Phase 16 SPEC initialization pattern (bypass SOUP/BRAINSTORM).** Created `multi-agent/plans/PHASE16_SPEC.md` with explicit "lifecycle deviation" framing and inheritance-from-Phase-15 cross-references. Phase 16 cites Phase 15 BRAIN IDs directly via `Source:` fields. Future agents should not be confused by the deviation.
- **Finding D — Design-decisions rules 20 + 21 added.** Rule 20 explicitly captures the user-requested all-pipeline generalization scope + the in-Phase-15-vs-Phase-16 triage. Rule 21 captures HMM-only concepts that legitimately stay HMM-specific (SAPS, trajectory clustering, ghost levels, narrowest summit-state strategy). These rules prevent future agents from treating expansion as out-of-scope OR over-generalizing the HMM-only items.

### v2 substantive-priorities verification (Round 3)

- 20 priorities / 9 phase-groups in Phase 15 — UNCHANGED.
- Affected priority bodies (SPEC15.4, SPEC15.6, SPEC15.7, SPEC15.8) are now significantly heavier with per-pipeline sub-deliverables, but no new SPEC15.X added; no priorities split per-pipeline.
- Phase 16 = 1 substantive priority (SPEC16.1) at minimum + placeholder for emergent items.
- No thin priorities standing alone in either phase.

### Mechanical re-audit post-Round-3

- All 34 BRAIN IDs still cited in PHASE15_SPEC.md (BRAIN15.1–34): PASS.
- SPEC15 IDs still sequential (SPEC15.1–20): PASS.
- All `Covers:` fields still present: PASS.
- PHASE16_SPEC.md initialized cleanly with explicit lifecycle-deviation note + inheritance from Phase 15.
- Dependency ordering (Round 3 verified): cross-pipeline sub-deliverables in SPEC15.4 (dlv 9) → SPEC15.6 (RMS classifier wiring) → SPEC15.7 (RMS strategy menu) → SPEC15.8 (RMS 4-step) all flow correctly via SPEC15.4 dlv 9 as gating prerequisite.

### Files touched in Round 3 (Agent 1 follow-up)

- `multi-agent/plans/PHASE15_SPEC.md` — Round 3 edits across SPEC15.4 (added deliverables 9 + 10) / SPEC15.6 (cross-pipeline reframing of goal + deliverables 5/6/7 + per-pipeline sub-deliverables checklist) / SPEC15.7 (cross-pipeline reframing of title + goal + per-pipeline current state + deliverables 1-4 + per-pipeline sub-deliverables checklist; RMS sliding-offset deferral note) / SPEC15.8 (cross-pipeline reframing of title + goal + per-pipeline current state + deliverable 9 rewrite + per-pipeline sub-deliverables checklist) + Phase 5 heading (HMM → cross-pipeline) + Universal naming + design decisions rules 20 + 21 added + Open questions JQ29 row updated + Self-audit Round 3 section added.
- `multi-agent/plans/PHASE16_SPEC.md` — NEW file initialized this round (lifecycle deviation: bypass SOUP/BRAINSTORM per user direction).
- `multi-agent/plans/PHASE15_FEEDBACK.md` — this Round 3 entry appended.
- `multi-agent/project_context/HANDOFF.md` — cold-start orientation updated to Round 3 complete + Phase 16 initialized.
- `multi-agent/project_context/TASK.md` — immediate-tasks queue updated.

### Round 3 readiness assessment

**SPEC15 is in good shape for a NARROW Agent 2 re-audit (Round 4)** — verify the 3 substantive priority rewrites (SPEC15.6, SPEC15.7, SPEC15.8) + new SPEC15.4 deliverables (9, 10) + design-decisions rules 20 + 21 don't introduce new gaps + per-pipeline sub-deliverables are coherent + the Phase 15 / Phase 16 triage is sound.

**PHASE16_SPEC.md is initialized** — Codex's narrow re-audit may also briefly verify it's structurally clean (per the bypass-SOUP/BRAINSTORM lifecycle deviation note in the SPEC body). Phase 16 SPEC will be polished further after Phase 15 audit/implement work produces lessons.

If the narrow re-audit comes back CLEAN, the SPEC is ready for SPEC-engineering closeout candidate + audit/implement stage opening (per the user's "we are not done with SPEC engineering until I say so" gate).

### No new open questions surfaced in Round 3

Future SPEC engineering rounds may surface JQ34+ as needed. The Phase 16 placeholder for "Phase-15-implementation-surfaced refinements" is the natural home for any work that emerges during the Phase 15 audit/implement cycles.

**Tests run:** none; SPEC-engineering work only.

**No CHANGELOG/DEVLOG entry written** per the no-LOG-during-iteration rule.

**Sign-off:** Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High), Agent 1.

---

## SPEC engineering Round 5 final audit — Agent 2 (Codex) — 2026-04-29

**Auditor:** Codex, Agent 2.

**Scope:** Narrow verification of Agent 1's Round 4 editorial cleanup after Codex Round 4 flagged stale Phase 16 / HMM-only wording in the live SPEC.

**Verdict:** CLEAN — SPEC engineering closeout candidate.

### Findings

- ✓ **Round 4 editorial cleanup landed.** Design-decision rule 16 is now explicitly marked superseded by rule 20; Phase 5 and SPEC15.7 / SPEC15.8 summary rows now use cross-pipeline wording; SPEC15.7's goal no longer says RMS sliding-offset is deferred.
- ✓ **Remaining Phase 16 / SPEC16 / RMS-sliding-offset hits are historical provenance only.** Grep hits in `PHASE15_SPEC.md` are now under explicit `SUPERSEDED` / correction language or in the rule-20 provenance note pointing to `SUMMIT_SOUP.md`. They no longer read as active implementation scope.
- ✓ **Phase 15 vs SUMMIT_SOUP split is coherent.** Phase 15 keeps HMM sliding-offset, cross-pipeline stage-selection / ODW-confined refinement, and cross-pipeline summit↔timing. Future summit-algorithm parity questions (`refine_summit_parabola` to Growth + HMM; HMM `_estimate_parabola_bp` unification; possible generalized multi-stage-aware summit filtering) live in `multi-agent/plans/next/SUMMIT_SOUP.md`.
- ✓ **Mechanical checks remain clean.** All 34 BRAIN IDs are cited in `PHASE15_SPEC.md`; SPEC IDs remain sequential from SPEC15.1 through SPEC15.20; all SPEC priorities still have `Covers:` fields; no atomization issue found.
- ✓ **No substantive blocker found.** I do not see a missing BRAINSTORM item, priority-order problem, unresolved open question, or convention issue that should block closeout.

### Recommendation

The SPEC is ready for the orchestrator/user-gated SPEC-engineering closeout. Next step is the PDS-v2 closeout flow: archive `PHASE15_BRAINSTORM.md` and `PHASE15_FEEDBACK.md`, keep `PHASE15_SPEC.md` live as the implementation contract, create the sibling `PHASE15_AUDIT_LOG.md` skeleton, then begin the audit/implement stage when the user says implementation may start.

**Tests run:** none; SPEC-engineering audit only.

**No CHANGELOG/DEVLOG entry written** per the no-LOG-during-iteration rule.

**Sign-off:** Codex, Agent 2.

---

## SPEC engineering Round 4 audit — Agent 2 (Codex) — 2026-04-29

**Auditor:** Codex, Agent 2.

**Scope:** Narrow final-audit attempt after Agent 1's Round 3 follow-up and the post-Round-3 correction that deleted `PHASE16_SPEC.md` and ported summit-algorithm parity questions into `multi-agent/plans/next/SUMMIT_SOUP.md`. I audited `PHASE15_SPEC.md` against `PHASE15_BRAINSTORM.md`, the Round 3 Codex findings, the SUMMIT_SOUP port, and a light live-code spot check of the summit/timing functions named in the SPEC.

**Verdict:** OPEN — small Agent 1 cleanup pass needed before SPEC-engineering closeout.

This is not a substantive redesign finding. The SPEC is now conceptually close and the Phase 15 vs SUMMIT_SOUP split is basically right. The remaining problem is that the live SPEC still contains stale / contradictory Phase 16 and HMM-only wording after the SUMMIT_SOUP correction. Those lines should be cleaned before the SPEC is locked for implementation, because implementation agents will treat the SPEC as the authoritative contract.

### Checks

- ✓ **BRAIN coverage:** `BRAIN15.1` through `BRAIN15.34` are present in both `PHASE15_BRAINSTORM.md` and `PHASE15_SPEC.md`; no BRAIN coverage gap found.
- ✓ **SPEC ID structure:** `SPEC15.1` through `SPEC15.20` are sequential and all 20 priority headings have `Covers:` fields.
- ✓ **Substantive-priorities rule:** still acceptable. The 20 priorities / 9 phase-groups remain large but substantive; no thin standalone priority introduced.
- ✓ **Round 3 substantive fixes mostly landed:** SPEC15.4 now adds RMS timing wiring; SPEC15.6 is an all-pipeline reliability / flat-sample contract; SPEC15.7 and SPEC15.8 are cross-pipeline in their main bodies; SPEC15.11/15.12 remain cross-pipeline for APS features/defaults; HMM-only concepts are correctly named in rule 21.
- ✓ **SUMMIT_SOUP port is structurally sound:** `PHASE16_SPEC.md` is absent from the working tree; `SUMMIT_SOUP.md` Item 5 now captures the corrected summit-algorithm parity landscape and explicitly says HMM sliding-offset is Phase 15 while `refine_summit_parabola` porting / HMM `_estimate_parabola_bp` unification are future SOUP-stage questions.
- ✓ **Light code spot-check supports the corrected landscape:** `refine_origin_sliding_offset` exists in `refinement.py` and is called by Growth and RMS; `refine_summit_parabola` exists in `refinement.py` and is called by RMS; HMM has local `_estimate_parabola_bp`; Growth calls timing; HMM calls timing; RMS timing wiring is not currently present.
- ⚠ **SPEC cleanup needed:** stale summary and self-audit text remains in the live SPEC and conflicts with the corrected state.

### Specific issues

1. **Stale design-decision rule 16 still looks active.**

   `multi-agent/plans/PHASE15_SPEC.md:95` still says SPEC15.8 implements HMM-only and growth/RMS summit↔timing is future-phase work unless the user opts in. Rule 20 (`PHASE15_SPEC.md:99-103`) supersedes it, but rule 16 itself is still written as a locked rule. Recommendation: replace rule 16 with an explicit "SUPERSEDED by rule 20" note or rewrite it to the Round 3 state. Do not leave two conflicting locked rules in the implementation contract.

2. **Stale HMM-only summary labels remain in the top tables.**

   `PHASE15_SPEC.md:30` still labels phase-group 5 as "HMM summit refinement + summit↔timing convergence", and `PHASE15_SPEC.md:120` still titles SPEC15.7 as "HMM summit refinement..." even though the actual heading at `PHASE15_SPEC.md:500` is now cross-pipeline. Recommendation: update these summary rows to match the body so agents skimming the status table do not narrow the work by accident.

3. **SPEC15.7 goal contains an incorrect post-correction statement about RMS sliding-offset.**

   `PHASE15_SPEC.md:504` still says "RMS sliding-offset port deferred to Phase 16" because RMS uses parabola-style refinement. This directly contradicts the corrected code-dive state at `PHASE15_SPEC.md:508` and `PHASE15_SPEC.md:560`, and `SUMMIT_SOUP.md:204`, which all say RMS already has sliding-offset. Recommendation: replace that sentence with the corrected split: HMM sliding-offset is Phase 15; future SUMMIT_SOUP work is cross-pipeline summit-algorithm parity, especially porting `refine_summit_parabola` to Growth + HMM and deciding HMM `_estimate_parabola_bp` fate.

4. **Remaining Phase 16 references in the SPEC should be converted to SUMMIT_SOUP / future-summit-phase wording.**

   Examples:
   - `PHASE15_SPEC.md:559` says Growth's parabola-smoothing audit informs whether Growth should adopt parabola refinement in Phase 16. Since no active Phase 16 SPEC exists, this should point to SUMMIT_SOUP / future summit phase.
   - `PHASE15_SPEC.md:1316`, `1321`, `1329-1335`, `1340`, and `1354` still describe the now-deleted Phase 16 SPEC / RMS sliding-offset deferral. Since these sections live inside the active SPEC, they should either be updated with "superseded by post-Round-3 SUMMIT_SOUP port" language or rewritten to the corrected state.

5. **HANDOFF has one stale Round 4 target description.**

   `multi-agent/project_context/HANDOFF.md:18` still says Round 4 should verify `PHASE16_SPEC.md` is structurally clean, and `HANDOFF.md:20` still says rule 20 includes "RMS sliding-offset port specifically deferred to Phase 16." The later HANDOFF lines do document the correction, but these stale bullets are in the cold-start orientation. Recommendation: update them after the SPEC cleanup so the next cold-start agent does not chase a deleted file or a corrected-away deferral.

### Readiness assessment

The remaining work is editorial cleanup of the authoritative planning surface, not new design. Once those stale references are corrected, I expect a narrow re-audit can be CLEAN quickly. I do not see a missing BRAIN entry, a broken priority order, an atomization problem, or a substantive Phase 15 vs SUMMIT_SOUP split problem.

### Recommendations

1. Agent 1 should make a small cleanup pass on `PHASE15_SPEC.md` only where stale Round 2 / initial Round 3 wording remains: rule 16, top tables, SPEC15.7 goal, SPEC15.7 per-pipeline checklist wording if needed, Self-audit Round 3, and JQ29 row.
2. Agent 1 should update the HANDOFF cold-start bullets that still mention `PHASE16_SPEC.md` verification / RMS sliding-offset Phase 16 deferral.
3. After that cleanup, run one grep check over `PHASE15_SPEC.md` for `PHASE16`, `SPEC16`, `RMS sliding-offset port`, `implements HMM-only`, and `future-phase unless user opts`; any remaining hits should be historical/provenance only and clearly labeled as superseded.

**Tests run:** none; SPEC-engineering audit only. Code spot-check was via targeted `rg` reads, not execution.

**No CHANGELOG/DEVLOG entry written** per the no-LOG-during-iteration rule.

**Sign-off:** Codex, Agent 2.

---

## SPEC engineering Round 3 follow-up — Phase 16 SPEC ported to SUMMIT_SOUP.md (2026-04-29)

**Author:** Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High), acting as Agent 1.

**Trigger:** User question (chat 2026-04-29 post-Round-3-commit): "I've been trying to get all pipelines to use all summit strategies (except where not possible like HMM-specific stuff). So that would mean RMS is missing stuff Growth/HMM have, AND vice versa. Is that correct to say?"

**Round-3-initial framing error caught:** Round 3 framed Phase 16 as "RMS sliding-offset port" based on incomplete code-grep of `rcn_mean_shift_engine.py` only. A Round-3-post-commit code dive at user request discovered **RMS already uses sliding-offset** via `onionskin_core/rcn_mean_shift_helpers.py:527, 547` (verified 2026-04-29). The actual cross-pipeline summit-algorithm gap landscape is in BOTH directions:
- HMM lacks sliding-offset (Phase 15 SPEC15.7 dlv 3 covers it).
- Growth + HMM lack `refine_summit_parabola` (RMS has it).
- HMM has its own simpler `_estimate_parabola_bp` that could potentially unify with the shared `refine_summit_parabola`.

**User direction (chat 2026-04-29):** SUMMIT_SOUP.md is the proper home for these brainstorm-stage cross-pipeline summit-algorithm parity questions (since they're more brainstorm-y than SPEC-y — multiple open discussion items + scope TBD). Port the corrected content from PHASE16_SPEC.md into SUMMIT_SOUP.md; delete PHASE16_SPEC.md (brief deviation reverted; standard PDS-v2 SOUP → BRAINSTORM → SPEC lifecycle preserved).

**Action taken (2026-04-29 post-Round-3-commit):**

1. **`multi-agent/plans/next/SUMMIT_SOUP.md` updated:**
   - Item 2 (Dynamic origin-detection window) — marked "MIGRATED to Phase 15 SPEC15.4 (2026-04-29)" with full provenance; original content preserved.
   - Item 3 (RMS pre-smoothing investigation) — marked "FOLDED into Phase 15 SPEC15.7 dlv 8 (2026-04-29)" with provenance; original content preserved.
   - Item 4 (HMM peak_rcn_stage + sliding-offset) — marked "MIGRATED to Phase 15 SPEC15.7 (2026-04-29)" with provenance; original content preserved.
   - Item 5 (Cross-pipeline summit estimator audit + unification) — major expansion added with corrected cross-pipeline summit-algorithm landscape table (Round-3-verified) + 4 named cross-pipeline parity gaps + 7 provisional deliverables (a-g) for the SOUP's eventual phase. Provenance note explains the deviation + correction history.
   - Open brainstorm questions section extended with 5 new questions (6-10) covering: `refine_summit_parabola` port to Growth + HMM; HMM `_estimate_parabola_bp` unification; HMM sliding-offset port placement (cross-reference); `hmm_summit_refinement.py` multi-stage-aware filtering generalization; cross-pipeline summit-algorithm parity scope sizing.

2. **`multi-agent/plans/PHASE16_SPEC.md` DELETED** via `git rm`. The brief lifecycle-deviation (bypass SOUP/BRAINSTORM) is reverted; standard PDS-v2 lifecycle preserved. SUMMIT_SOUP graduates to its own phase (with phase number assigned at graduation) via standard SOUP → BRAINSTORM → SPEC lifecycle when the orchestrator decides — at which point the open brainstorm questions get answered at the BRAINSTORM stage.

3. **`multi-agent/plans/PHASE15_SPEC.md` updated:**
   - Design-decisions rule 20 — "Phase 16 (deferred — focused-completion phase; SPEC at PHASE16_SPEC.md)" replaced with "Deferred to a future phase via SUMMIT_SOUP.md (Item 5 expansion + Open brainstorm questions 6-10)"; full provenance + Round-3-post-commit code-dive correction note retained.
   - SPEC15.7 per-pipeline current state + implementation notes cross-references updated to SUMMIT_SOUP.md.
   - SPEC15.7 per-pipeline sub-deliverables checklist corrected to reflect actual cross-pipeline summit-algorithm landscape (RMS already has sliding-offset; the gap is HMM's missing sliding-offset which IS covered by SPEC15.7 dlv 3).

4. **`multi-agent/project_context/HANDOFF.md` + `TASK.md` updated** to reflect the SUMMIT_SOUP port + PHASE16_SPEC.md deletion.

5. **This FEEDBACK entry appended** to document the port. (FEEDBACK is append-only by convention — the historical Round 3 entry above stays intact even though some of its language is now superseded; this follow-up entry is the canonical pointer to the corrected state.)

**Why this approach:**

- The user's instinct that "all pipelines should use all summit strategies except where not possible like HMM-specific stuff" is the correct cross-pipeline-parity principle.
- The brainstorm-stage character of the questions (multiple open discussion items + scope TBD) doesn't fit a SPEC body cleanly. SOUP files are explicitly the right home for that kind of content per PDS-v2.
- SUMMIT_SOUP.md was already structurally perfect for this content (Item 5 = "Cross-pipeline summit estimator audit and unification"; Open brainstorm questions section already established).
- Deleting PHASE16_SPEC.md cleanly reverts a structurally-deviant invention. The earlier commit preserves the file's history in git for anyone who wants to reconstruct the deviation; the current state is clean.

**v2 substantive-priorities verification (post-port):** Phase 15 SPEC15.X structure unchanged (20 priorities / 9 phase-groups / 34 BRAIN IDs cited). Phase 16 no longer exists as an active SPEC — when SUMMIT_SOUP graduates to its own phase via standard lifecycle, the eventual Phase 16 (or whatever number applies at graduation) will go through the standard SOUP → BRAINSTORM → SPEC process.

**Mechanical re-audit post-port:**
- All 34 BRAIN IDs still cited in PHASE15_SPEC.md: PASS.
- SPEC15 IDs still sequential (SPEC15.1–20): PASS.
- All `Covers:` fields still present: PASS.
- PHASE16_SPEC.md no longer exists in working tree (`git rm` performed).
- SUMMIT_SOUP.md has 4 migration markers (Items 2, 3, 4 + Item 5 expansion provenance); Open brainstorm questions extended from 5 to 10.

**Files touched in the port:**
- `multi-agent/plans/next/SUMMIT_SOUP.md` (modified — major expansion)
- `multi-agent/plans/PHASE16_SPEC.md` (DELETED via `git rm`)
- `multi-agent/plans/PHASE15_SPEC.md` (modified — design-decisions rule 20 + SPEC15.7 references updated to point at SUMMIT_SOUP.md)
- `multi-agent/project_context/HANDOFF.md` (modified — cold-start orientation updated)
- `multi-agent/project_context/TASK.md` (modified — immediate-tasks queue updated)
- `multi-agent/plans/PHASE15_FEEDBACK.md` (this entry appended)

**Tests run:** none; SPEC-engineering work only.

**No CHANGELOG/DEVLOG entry written** per the no-LOG-during-iteration rule.

**Sign-off:** Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High), Agent 1.

---

## SPEC engineering Round 4 — Agent 1 editorial cleanup pass (2026-04-29)

**Author:** Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High), acting as Agent 1.

**Action:** Triaged Codex's SPEC engineering Round 4 audit (verdict OPEN-but-small; "I did not find a substantive design gap... The blocker is stale text inside the live SPEC"). All 5 Codex editorial findings ACCEPTED + applied. SPEC structure unchanged (20 priorities / 9 phase-groups / 34 BRAIN IDs cited).

### Codex findings — all 5 ACCEPTED + applied

| # | Codex finding | Resolution applied | SPEC location |
|---|---|---|---|
| 1 | Stale design-decision rule 16 still looks active | Rewrote rule 16 with "SUPERSEDED by rule 20 (Round 3 reversal 2026-04-29)" header + reversal pointer; original Round 2 lock language preserved as historical narrative for traceability | `PHASE15_SPEC.md` rule 16 |
| 2 | Stale HMM-only summary labels remain in top tables | Phase-group 5 label updated to "Cross-pipeline summit refinement + summit↔timing convergence"; SPEC15.7 + SPEC15.8 priority status table titles updated to cross-pipeline wording | `PHASE15_SPEC.md` lines 30, 120, 121 |
| 3 | SPEC15.7 goal contains incorrect post-correction RMS sliding-offset sentence | Goal sentence rewritten: "Implement HMM-side sliding-offset sub-bin localization at parity with Growth + RMS (Growth uses it directly; RMS uses it via `rcn_mean_shift_helpers.py:527, 547`; HMM port = NEW work in this priority)"; added explicit cross-reference to SUMMIT_SOUP.md for cross-pipeline summit-algorithm parity completion | `PHASE15_SPEC.md` SPEC15.7 goal |
| 4 | Remaining Phase 16 references in SPEC should be converted to SUMMIT_SOUP / future-summit-phase wording | (a) SPEC15.7 per-pipeline checklist parabola-Phase-16 reference updated to point at SUMMIT_SOUP.md Open brainstorm question 6; (b) Self-audit Round 3 section gets a HISTORICAL RECORD header with explicit supersession notes; (c) bullets B + C + Phase 16 readiness items get "❌ SUPERSEDED post-Round-3" inline notes; (d) Finding 2 narrative line gets per-bullet "SUPERSEDED" marker | `PHASE15_SPEC.md` SPEC15.7 + Self-audit Round 3 § |
| 5 | HANDOFF has stale Round 4 target description | HANDOFF cold-start bullets rewritten: dropped "verify PHASE16_SPEC.md is structurally clean" target; replaced "RMS sliding-offset port specifically deferred to Phase 16" with corrected SUMMIT_SOUP wording; "Most recent state" updated to Round 4 cleanup state; TASK queue updated to "SPEC engineering closeout decision (awaiting user gate)" | `HANDOFF.md` + `TASK.md` |

### Codex findings — REJECTED: NONE

All 5 editorial findings accepted as-is.

### Codex-recommended grep verification (item 3 of recommendations)

After cleanup, ran the recommended greps to verify all remaining hits are historical/provenance-only and clearly labeled as superseded:

- `grep PHASE16|SPEC16 PHASE15_SPEC.md` — 6 hits, ALL in (a) design-decisions rule 20 provenance narrative, (b) JQ29 row with explicit "REVERSED Round 3" + correction note, or (c) Self-audit Round 3 section under HISTORICAL RECORD header with per-bullet "SUPERSEDED" markers.
- `grep "RMS sliding-offset port" PHASE15_SPEC.md` — 6 hits, all in same historical/provenance-labeled locations.
- `grep "implements HMM-only" PHASE15_SPEC.md` — 1 hit, in superseded rule 16 with explicit reversal language.
- `grep "future-phase unless user opts" PHASE15_SPEC.md` — 2 hits, both in superseded rule 16 + Self-audit Round 3 historical narrative.

All grep hits PASS Codex's "historical/provenance-only and clearly labeled as superseded" bar.

### Mechanical re-audit post-Round-4

- All 34 BRAIN IDs still cited in PHASE15_SPEC.md: PASS.
- SPEC15 IDs still sequential (SPEC15.1–20): PASS.
- All `Covers:` fields still present: PASS.
- No active (non-historical) Phase 16 references in the SPEC body.

### v2 substantive-priorities verification (Round 4)

- 20 priorities / 9 phase-groups in Phase 15 — UNCHANGED. Round 4 was purely editorial — no priority additions, removals, or splits.
- No thin priorities standing alone.

### Files touched in Round 4 (Agent 1 cleanup)

- `multi-agent/plans/PHASE15_SPEC.md` — design-decision rule 16 rewritten as superseded-pointer; phase-group 5 + SPEC15.7 + SPEC15.8 status-table rows updated; SPEC15.7 goal rewritten; SPEC15.7 per-pipeline checklist parabola reference updated; Self-audit Round 3 section gets HISTORICAL RECORD header + per-bullet SUPERSEDED markers; JQ29 row already corrected in Round 3 follow-up.
- `multi-agent/project_context/HANDOFF.md` — "Most recent state" line updated to Round 4 cleanup; cold-start bullets 1-5 rewritten to drop stale PHASE16_SPEC.md verification target + add current state.
- `multi-agent/project_context/TASK.md` — immediate-tasks queue updated to "SPEC engineering closeout decision (awaiting user gate)".
- `multi-agent/plans/PHASE15_FEEDBACK.md` — this Round 4 entry appended.

### Round 4 readiness assessment

**SPEC15 is in good shape for SPEC-engineering closeout.** Codex Round 4 audit verdict was explicitly "not a substantive redesign finding... once those stale references are corrected, I expect a narrow re-audit can be CLEAN quickly." All Round 4 edits are textual / cosmetic; no design content changed.

**Recommended next steps** (per user "we are not done with SPEC engineering until I say so" gate):

1. **Optional narrow Round 5 re-audit** — Codex verifies the Round 4 editorial cleanup. Expected CLEAN. Cheap insurance before SPEC-lock.
2. **OR direct SPEC-engineering closeout** if user signals SPEC-locked: open closeout per PDS-v2 (Form B DEVLOG-fossilizing commit; archive BRAINSTORM + FEEDBACK to `multi-agent/plans/archived/<YYYYMMDD>-PHASE15_BRAINSTORM.md` / `-PHASE15_FEEDBACK.md`; create sibling `PHASE15_AUDIT_LOG.md` skeleton).
3. **THEN audit/implement stage opens** per `multi-agent/workflows/spec_plan_three_role_audit_loop-v2.md`.

### No new open questions surfaced in Round 4

All Round 4 work was editorial cleanup of Round 3's outputs. The substantive scope (Phase 15 cross-pipeline generalization + cross-pipeline summit-algorithm parity completion deferred to SUMMIT_SOUP) is settled.

**Tests run:** none; SPEC-engineering work only.

**No CHANGELOG/DEVLOG entry written** per the no-LOG-during-iteration rule.

**Sign-off:** Claude Code 2.1.112 (claude-opus-4-7 ; Effort: High), Agent 1.
