# Core-Chromosome Parity Diagnosis

Date: 2026-04-11

## Scope

Investigate the remaining HMM parity differences after parity was refocused to
chromosomes X, II, III, and IV.

Questions addressed:
- Which differences reflect known legacy PuffStep step-9/10 summit artifacts?
- Which differences reflect current native HMM behavioral drift?
- Did the drift come from changed HMM math or from the newer parity workflow?

## Runs and references used

- Original full reference: `dev/puffstep/automate/5kb/`
- Original core reference: `dev/puffstep/automate/5kb-corechrom/`
- Current full-genome native run: `dev/runs/puffstep-py-fullgenome/03-hmm/`
- Current core-only parity run: `dev/runs/puffstep-py/03-hmm/`
- Corrected v2 full reference: `dev/puffstep/automate/5kb-v2/`
- Corrected v2 core reference: `dev/puffstep/automate/5kb-corechrom-v2/`

## Main findings

### 1. Full-genome native HMM still matches when compared like-for-like

Validation command:

```bash
python scripts/compare_puffstep_outputs.py \
	--reference dev/puffstep/automate/5kb-v2 \
	--run dev/runs/puffstep-py-fullgenome/03-hmm
```

Result:

- `06-HMM`: 4 pass
- `07-collapsedHMM`: 4 pass
- `08-aboveBackground`: 8 pass
- `09-summitStates`: 4 pass
- `10-summitBins`: 8 pass
- `OVERALL: PASS (28 files match)`

Interpretation:

- There is no evidence here that the current native HMM implementation drifted in
	steps 6-8 relative to the earlier full-genome parity path.
- The native pipeline still reproduces the corrected full-genome reference exactly.

### 2. The old full reference contains legacy step-9/10 artifacts

The corrected `5kb-v2` reference replaces step 9 and step 10 files for stages 2-5
with outputs from the validated current full-genome native run.

This corrects:
- the confirmed stage-4 chromosome-X legacy zero-width summit artifact
- associated-contig step-9/10 legacy point artifacts inherited from PuffStep's
	zero-width single-interval collapse behavior

### 3. The remaining core-only diffs are introduced by the core-only rerun workflow

Validation command:

```bash
python scripts/compare_puffstep_outputs.py \
	--reference dev/puffstep/automate/5kb-corechrom-v2 \
	--run dev/runs/puffstep-py/03-hmm
```

Result:

- `06-HMM`: 1 pass / 3 fail
- `07-collapsedHMM`: 1 pass / 3 fail
- `08-aboveBackground`: 3 pass / 5 fail
- `09-summitStates`: 2 pass / 2 fail
- `10-summitBins`: 0 pass / 8 fail
- `OVERALL: FAIL (21 files failed or missing, 7 passed)`

Interpretation:

- These remaining differences are not caused by the old step-9/10 legacy summit bug.
- They arise because the current parity run is recomputed on `--chromosomes X,II,III,IV`
	before step 1, while the reference was originally generated on the full genome and
	then filtered down to those chromosomes.

## Why the core-only rerun drifts

In the current HMM workflow, chromosome filtering happens before any HMM step runs.
This changes the actual inputs to:

- step 1 median normalization
- step 2 group median / union statistics
- step 4 chromosomal median-ratio normalization
- step 5 smoothed RCN values
- step 6 HMM state calls

Once step 6 state calls shift by 1-3 bins, steps 7-10 inherit those differences.

This explains the observed pattern:
- small left/right boundary shifts in state 1 or higher-state intervals
- matching locus identity but slightly different start/end coordinates
- modest smoothed/raw summit-bin RCN differences in step 10

## Classification of remaining diffs

### Class A: corrected legacy-reference bugs

- Step 9 zero-width summit-state intervals caused by PuffStep/CovBed collapse behavior
- Step 10 one-base point-expansion summit bins or truncated point-like summit outputs

Resolution:
- move these corrected outputs into `5kb-v2` and `5kb-corechrom-v2`

### Class B: core-only rerun methodology drift

- Step 6 state-path boundary shifts on core chromosomes
- Step 7 collapsed-state boundary shifts inherited from step 6
- Step 8 above-background interval shifts inherited from step 7
- Step 9 summit-state interval shifts inherited from step 8
- Step 10 RCN differences caused by using a different smoothed/raw input surface after
	the earlier steps drifted

Resolution options:
- define parity as full-genome rerun compared against full-genome or v2 reference, then
	restrict interpretation to core chromosomes afterward
- or create a new core-only gold standard by rerunning the legacy pipeline in true
	core-only mode, if that behavior is desired as its own separate baseline

## Provenance of the workflow change

The workflow-level cause is visible in git history:

- commit `1434022`: introduced pre-HMM sequence/chromosome filtering into the native HMM path
- commit `6b923bb`: changed `make puffstep-py` to run with `--chromosomes X,II,III,IV`
	and compare against `5kb-corechrom`

Together, those changes made the parity run a core-only recomputation rather than a
full-genome recomputation filtered at comparison time.

## Reproducibility

The script `scripts/generate_hmm_v2_references.py` rebuilds:

- `dev/puffstep/automate/5kb-v2/`
- `dev/puffstep/automate/5kb-corechrom-v2/`

from the validated current full-genome native run:

- `dev/runs/puffstep-py-fullgenome/03-hmm/`

