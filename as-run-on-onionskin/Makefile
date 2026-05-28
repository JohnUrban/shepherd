.PHONY: help fast test toy toy-single single twin test-single full full-no-split full-twin full-collapsed-repeat summit summit-smoke summit-baseline summit-hmm rcn-summit-diagnostic optimize-single validate-flags dev clean shape-score-bic puffstep-II puffstep-py puffstep-py-core puffstep-py-filtered puff-compare puff-compare-core puff-parity hmm-fork-age-smoke posterior-layout-smoke clust-ds1-II clust-ds2-II clust-report-ds1-II clust-report-ds2-II clust-ds2-4chr clust-report-ds2-4chr hmm-trim-eval hmm-trim-eval-hw3tw0 hmm-trim-eval-hw3tw1 hmm-trim-eval-hw3tw2 hmm-trim-eval-hw4tw1 hmm-trim-eval-hw4tw2 hmm-trim-report

HMM_PARITY_REFERENCE=dev/puffstep/automate/5kb-v2
HMM_PARITY_CORE_CHROMS=X,II,III,IV
HMM_PARITY_CORE_REFERENCE=dev/puffstep/automate/5kb-corechrom-v2

help:
	@echo "Available targets:"
	@echo "  make fast                  - run test + toy + single + twin in parallel (-j4)"
	@echo "  make test                  - run CLI + pytest parallel (~1 min)"
	@echo "  make toy                   - multistage toy dataset smoke test (fast)"
	@echo "  make toy-single            - single-file + single-stage smoke test (fast)"
	@echo "  make single                - alias for toy-single"
	@echo "  make twin                  - twin-peak resolution validation, chr II toy data (fast)"
	@echo "  make test-single           - full single-mode regression T1-T12 (slow)"
	@echo "  make full                  - full chr II test: sensitivity + no-split + twin peaks (medium)"
	@echo "  make full-no-split         - chr II spurious-split regression only (medium)"
	@echo "  make full-twin             - chr II twin-peak resolution test only (medium)"
	@echo "  make full-collapsed-repeat - single-file shape-filter / collapsed-repeat FP test (fast)"
	@echo "  make summit                - canonical summit regression surface for growth + HMM + rcn-mean-shift, all runs, regression check (slow)"
	@echo "  make summit-smoke          - canonical quick summit smoke for growth + HMM + rcn-mean-shift, DS2 hires only (fast)"
	@echo "  make summit-baseline       - store the canonical growth + HMM + rcn-mean-shift summit regression baseline"
	@echo "  make summit-hmm            - HMM-only by-eye summit evaluation on the DS2 training surface"
	@echo "  make rcn-summit-diagnostic - run the dedicated chr-II rcn summit diagnostic artifact generator (slow)"
	@echo "  make optimize-single       - exploratory single-sample parameter grid search (manual tuning utility)"
	@echo "  make validate-flags FILE   - scan files for deprecated onionskin CLI flags and redirects"
	@echo "  make shape-score-bic       - compare default vs --shape-score-strict-bic on shape filter on toy + DS2 data (fast)"
	@echo "  make puffstep-II           - run PuffStep bash pipeline on full-genome DS2 5kb data (slow)"
	@echo "  make puffstep-py           - run Python HMM engine on full-genome DS2 5kb data with authoritative parity settings (slow)"
	@echo "  make puffstep-py-core      - run Python HMM engine on exploratory core parity chromosomes X,II,III,IV (slow)"
	@echo "  make puffstep-py-filtered  - run Python HMM engine on full-genome DS2 5kb data with filtered development settings (slow)"
	@echo "  make hmm-fork-age-smoke    - generate a tiny HMM-only step-12 fork-age smoke run in dev/runs (fast)"
	@echo "  make puff-compare          - HMM-only parity compare against the authoritative full-genome v2 reference"
	@echo "  make puff-compare-core     - compare puffstep-py-core (or RUN=<dir>/01-prior/01-hmm) vs exploratory core-chromosome v2 reference"
	@echo "  make puff-parity           - run authoritative full-genome parity and compare against 5kb-v2"
	@echo "  make posterior-layout-smoke - run toy prior+posterior grouped-layout smoke, including HMM, in dev/runs"
	@echo "  make clust-ds1-II          - run all 11 APS clustering experiments on ds1+chr-II, then report"
	@echo "  make clust-ds2-II          - run all 11 APS clustering experiments on ds2+chr-II, then report"
	@echo "  make clust-ds2-4chr        - run all 11 APS clustering experiments on ds2+X,II,III,IV (full 4-chr), then report"
	@echo "  make clust-report-ds1-II   - re-print the ds1+chr-II comparison table (experiments must already exist)"
	@echo "  make clust-report-ds2-II   - re-print the ds2+chr-II comparison table (experiments must already exist)"
	@echo "  make hmm-trim-eval         - run all 5 --hmm-trim-halfwidth evaluation variants on ds2+chr-II (slow)"
	@echo "  make hmm-trim-eval-hw3tw0  - run baseline (hw=3 tw=0 median) on ds2+chr-II"
	@echo "  make hmm-trim-eval-hw3tw1  - run hw=3 tw=1 trimmed-mean on ds2+chr-II"
	@echo "  make hmm-trim-eval-hw3tw2  - run hw=3 tw=2 trimmed-mean on ds2+chr-II"
	@echo "  make hmm-trim-eval-hw4tw1  - run hw=4 tw=1 trimmed-mean on ds2+chr-II"
	@echo "  make hmm-trim-eval-hw4tw2  - run hw=4 tw=2 trimmed-mean on ds2+chr-II"
	@echo "  make hmm-trim-report       - compare all 5 trim variants (runs must exist)"
	@echo "  make dev                   - example sandbox run"
	@echo "  make clean                 - remove outputs"

fast:
	$(MAKE) -j4 test toy single twin

test:
	@echo "Running CLI check + pytest..."
	python onionskin.py -h
	pytest -n auto -q

toy:
	bash tests/run_toy.sh

toy-single:
	bash tests/run_toy_single.sh

single: toy-single

twin:
	bash tests/run_twin_peak_test.sh

test-single:
	bash tests/run_single_tests.sh

full:
	bash tests/run_full_chrII_test.sh full

full-no-split:
	bash tests/run_full_chrII_test.sh no-split

full-twin:
	bash tests/run_full_chrII_test.sh twin

full-collapsed-repeat:
	bash tests/run_full_chrII_test.sh collapsed-repeat

summit:
	@echo "[summit] canonical growth + HMM summit regression surface"
	bash tests/run_summit_precision_test.sh full

summit-smoke:
	@echo "[summit-smoke] canonical quick growth + HMM summit smoke"
	bash tests/run_summit_precision_test.sh smoke

summit-baseline:
	@echo "[summit-baseline] storing canonical growth + HMM summit regression baseline"
	bash tests/run_summit_precision_test.sh baseline

summit-hmm:
	@echo "[summit-hmm] HMM-only by-eye summit evaluation on the DS2 full-chromosome training surface"
	rm -rf dev/runs/summit-hmm
	python onionskin.py \
		--manifest tests/full_chrom_training_data/manifest.dataset2.5000bp.tsv \
		--hires-manifest tests/full_chrom_training_data/manifest.dataset2.1000bp.tsv,tests/full_chrom_training_data/manifest.dataset2.500bp.tsv \
		--out-dir dev/runs/summit-hmm \
		--out-prefix summit_hmm \
		--pipelines hmm \
		--skip-aps \
		--norm-mode chrom-median \
		--chromosomes II
	python scripts/summit_inspector.py \
		--run-dir dev/runs/summit-hmm \
		--pipelines hmm \
		--region "$$(awk 'NR==1 {print $$1\":\"$$2\"-\"$$3}' tests/summit_training_data/II9A.HMM.bed)" \
		--annot-bed tests/summit_training_data/II9A.HMM.bed \
		--output dev/runs/summit-hmm/II9A_hmm
	python scripts/summit_inspector.py \
		--run-dir dev/runs/summit-hmm \
		--pipelines hmm \
		--region "$$(awk 'NR==1 {print $$1\":\"$$2\"-\"$$3}' tests/summit_training_data/II2B.HMM.bed)" \
		--annot-bed tests/summit_training_data/II2B.HMM.bed \
		--output dev/runs/summit-hmm/II2B_hmm

rcn-summit-diagnostic:
	@echo "[rcn-summit-diagnostic] refreshing summit outputs and writing a dedicated rcn diagnostic artifact"
	SUMMIT_CHROMOSOMES=II bash tests/run_rcn_summit_diagnostic.sh refresh

optimize-single:
	python tests/optimize_single_params.py

validate-flags:
	python scripts/validate_onionskin_flags.py $(filter-out $@,$(MAKECMDGOALS))

shape-score-bic:
	bash tests/test_strict_bic.sh

CONDA_BIN=/Users/johnurban/software/conda/anaconda3/bin

puffstep-II:
	@mkdir -p dev/runs/puffstep-II
	cd dev/runs/puffstep-II && PATH=$(CONDA_BIN):$$PATH \
		bash $(CURDIR)/PuffStep/utils/puffStepPipeline.parallel.sh \
		$(CURDIR)/dev/datasets/full_genome/batch/manifest.batch-5kb.fofn

puffstep-py:
	@mkdir -p dev/runs/puffstep-py
	python onionskin.py \
		--manifest $(CURDIR)/dev/datasets/full_genome/batch/manifest.batch-5kb.fofn \
		--out-dir dev/runs/puffstep-py \
		--pipelines hmm \
		--skip-aps \
		--hmm-statepath-base 1 \
		--hmm-norm-mode ref-stage \
		--chromosomes all \
		--min-seq-length 0 \
		--min-bin-count-per-seq 1 \
		--verbose

puffstep-py-core:
	@mkdir -p dev/runs/puffstep-py-core
	python onionskin.py \
		--manifest $(CURDIR)/dev/datasets/full_genome/batch/manifest.batch-5kb.fofn \
		--out-dir dev/runs/puffstep-py-core \
		--pipelines hmm \
		--skip-aps \
		--hmm-statepath-base 1 \
		--hmm-norm-mode ref-stage \
		--chromosomes $(HMM_PARITY_CORE_CHROMS) \
		--min-seq-length 0 \
		--min-bin-count-per-seq 1 \
		--verbose

puffstep-py-filtered:
	@mkdir -p dev/runs/puffstep-py-filtered
	python onionskin.py \
		--manifest $(CURDIR)/dev/datasets/full_genome/batch/manifest.batch-5kb.fofn \
		--out-dir dev/runs/puffstep-py-filtered \
		--pipelines hmm \
		--skip-aps \
		--hmm-statepath-base 1 \
		--hmm-norm-mode ref-stage \
		--min-seq-length 50000 \
		--min-bin-count-per-seq 10 \
		--verbose

hmm-fork-age-smoke:
	python tests/run_hmm_fork_age_smoke.py

# ── hmm-trim-halfwidth evaluation (ds2 + chr II, 5 smoothing variants) ───────

HMM_TRIM_MANIFEST=$(CURDIR)/tests/full_chrom_training_data/manifest.dataset2.5000bp.tsv
HMM_TRIM_EVAL_DIR=dev/runs/hmm_trim_eval

hmm-trim-eval: hmm-trim-eval-hw3tw0 hmm-trim-eval-hw3tw1 hmm-trim-eval-hw3tw2 hmm-trim-eval-hw4tw1 hmm-trim-eval-hw4tw2
	@echo "[hmm-trim-eval] All 5 trim variants complete."
	@echo "  $(HMM_TRIM_EVAL_DIR)/hw3tw0  (hw=3 tw=0, pure median, baseline)"
	@echo "  $(HMM_TRIM_EVAL_DIR)/hw3tw1  (hw=3 tw=1, trimmed-mean)"
	@echo "  $(HMM_TRIM_EVAL_DIR)/hw3tw2  (hw=3 tw=2, trimmed-mean)"
	@echo "  $(HMM_TRIM_EVAL_DIR)/hw4tw1  (hw=4 tw=1, wider window + trimmed-mean)"
	@echo "  $(HMM_TRIM_EVAL_DIR)/hw4tw2  (hw=4 tw=2, wider window + trimmed-mean)"
	@echo "Run 'make hmm-trim-report' to compare."

hmm-trim-eval-hw3tw0:
	@rm -rf $(HMM_TRIM_EVAL_DIR)/hw3tw0
	python onionskin.py \
		--manifest $(HMM_TRIM_MANIFEST) \
		--chromosomes II \
		--out-dir $(HMM_TRIM_EVAL_DIR)/hw3tw0 \
		--pipelines hmm \
		--skip-aps \
		--hmm-smooth-halfwidth 3 \
		--hmm-trim-halfwidth 0 \
		--verbose

hmm-trim-eval-hw3tw1:
	@rm -rf $(HMM_TRIM_EVAL_DIR)/hw3tw1
	python onionskin.py \
		--manifest $(HMM_TRIM_MANIFEST) \
		--chromosomes II \
		--out-dir $(HMM_TRIM_EVAL_DIR)/hw3tw1 \
		--pipelines hmm \
		--skip-aps \
		--hmm-smooth-halfwidth 3 \
		--hmm-trim-halfwidth 1 \
		--verbose

hmm-trim-eval-hw3tw2:
	@rm -rf $(HMM_TRIM_EVAL_DIR)/hw3tw2
	python onionskin.py \
		--manifest $(HMM_TRIM_MANIFEST) \
		--chromosomes II \
		--out-dir $(HMM_TRIM_EVAL_DIR)/hw3tw2 \
		--pipelines hmm \
		--skip-aps \
		--hmm-smooth-halfwidth 3 \
		--hmm-trim-halfwidth 2 \
		--verbose

hmm-trim-eval-hw4tw1:
	@rm -rf $(HMM_TRIM_EVAL_DIR)/hw4tw1
	python onionskin.py \
		--manifest $(HMM_TRIM_MANIFEST) \
		--chromosomes II \
		--out-dir $(HMM_TRIM_EVAL_DIR)/hw4tw1 \
		--pipelines hmm \
		--skip-aps \
		--hmm-smooth-halfwidth 4 \
		--hmm-trim-halfwidth 1 \
		--verbose

hmm-trim-eval-hw4tw2:
	@rm -rf $(HMM_TRIM_EVAL_DIR)/hw4tw2
	python onionskin.py \
		--manifest $(HMM_TRIM_MANIFEST) \
		--chromosomes II \
		--out-dir $(HMM_TRIM_EVAL_DIR)/hw4tw2 \
		--pipelines hmm \
		--skip-aps \
		--hmm-smooth-halfwidth 4 \
		--hmm-trim-halfwidth 2 \
		--verbose

hmm-trim-report:
	python tests/eval_hmm_trim_halfwidth.py \
		--runs \
		  hw3tw0:$(HMM_TRIM_EVAL_DIR)/hw3tw0 \
		  hw3tw1:$(HMM_TRIM_EVAL_DIR)/hw3tw1 \
		  hw3tw2:$(HMM_TRIM_EVAL_DIR)/hw3tw2 \
		  hw4tw1:$(HMM_TRIM_EVAL_DIR)/hw4tw1 \
		  hw4tw2:$(HMM_TRIM_EVAL_DIR)/hw4tw2 \
		--by-eye tests/full_chrom_training_data/amplicons.by-eye.bed \
		--out dev/runs/hmm_trim_eval/report.md

puff-compare:
	python scripts/compare_puffstep_outputs.py \
		--reference $(or $(REF),$(HMM_PARITY_REFERENCE)) \
		--run $(or $(RUN),dev/runs/puffstep-py/01-prior/01-hmm)

puff-compare-core:
	python scripts/compare_puffstep_outputs.py \
		--reference $(or $(REF),$(HMM_PARITY_CORE_REFERENCE)) \
		--run $(or $(RUN),dev/runs/puffstep-py-core/01-prior/01-hmm)

puff-parity: puffstep-py puff-compare

posterior-layout-smoke:
	@rm -rf dev/runs/posterior-layout-smoke
	python onionskin.py \
		--manifest tests/toy_dataset/manifest_5kb.tsv \
		--chromosomes II \
		--out-dir dev/runs/posterior-layout-smoke \
		--out-prefix posterior_layout_smoke \
		--resolve-overlaps \
		--pipelines all \
		--growth-fit-method linear \
		--norm-mode chrom-median \
		--min-seq-length 0 \
		--min-bin-count-per-seq 1 \
		--verbose

clust-ds1-II:
	python scripts/aps_cluster_experiments.py \
		--manifest tests/full_chrom_training_data/manifest.dataset1.5000bp.tsv \
		--chromosomes II \
		--out-dir dev/runs/aps_clust_ds1_II \
		--prefix ds1
	python scripts/aps_cluster_report.py \
		--exp-dir dev/runs/aps_clust_ds1_II \
		--prefix ds1 \
		--chromosome II \
		--ref-locus 32000000-35000000

clust-ds2-II:
	python scripts/aps_cluster_experiments.py \
		--manifest tests/full_chrom_training_data/manifest.dataset2.5000bp.tsv \
		--chromosomes II \
		--out-dir dev/runs/aps_clust_ds2_II \
		--prefix ds2
	python scripts/aps_cluster_report.py \
		--exp-dir dev/runs/aps_clust_ds2_II \
		--prefix ds2 \
		--chromosome II \
		--ref-locus 32000000-35000000

clust-report-ds1-II:
	python scripts/aps_cluster_report.py \
		--exp-dir dev/runs/aps_clust_ds1_II \
		--prefix ds1 \
		--chromosome II \
		--ref-locus 32000000-35000000

clust-report-ds2-II:
	python scripts/aps_cluster_report.py \
		--exp-dir dev/runs/aps_clust_ds2_II \
		--prefix ds2 \
		--chromosome II \
		--ref-locus 32000000-35000000

clust-ds2-4chr:
	python scripts/aps_cluster_experiments.py \
		--manifest dev/datasets/full_genome/batch/manifest.batch-5kb.fofn \
		--chromosomes X,II,III,IV \
		--out-dir dev/runs/aps_clust_ds2_4chr \
		--prefix ds2
	@echo "=== chr II ===" && python scripts/aps_cluster_report.py \
		--exp-dir dev/runs/aps_clust_ds2_4chr \
		--prefix ds2 \
		--chromosome II \
		--ref-locus 32000000-35000000
	@echo "=== chr III ===" && python scripts/aps_cluster_report.py \
		--exp-dir dev/runs/aps_clust_ds2_4chr \
		--prefix ds2 \
		--chromosome III
	@echo "=== chr X ===" && python scripts/aps_cluster_report.py \
		--exp-dir dev/runs/aps_clust_ds2_4chr \
		--prefix ds2 \
		--chromosome X
	@echo "=== chr IV ===" && python scripts/aps_cluster_report.py \
		--exp-dir dev/runs/aps_clust_ds2_4chr \
		--prefix ds2 \
		--chromosome IV \
		--ref-locus 32000000-35000000

clust-report-ds2-4chr:
	@echo "=== chr II ===" && python scripts/aps_cluster_report.py \
		--exp-dir dev/runs/aps_clust_ds2_4chr \
		--prefix ds2 \
		--chromosome II \
		--ref-locus 32000000-35000000
	@echo "=== chr III ===" && python scripts/aps_cluster_report.py \
		--exp-dir dev/runs/aps_clust_ds2_4chr \
		--prefix ds2 \
		--chromosome III
	@echo "=== chr X ===" && python scripts/aps_cluster_report.py \
		--exp-dir dev/runs/aps_clust_ds2_4chr \
		--prefix ds2 \
		--chromosome X
	@echo "=== chr IV ===" && python scripts/aps_cluster_report.py \
		--exp-dir dev/runs/aps_clust_ds2_4chr \
		--prefix ds2 \
		--chromosome IV \
		--ref-locus 32000000-35000000

dev:
	bash scripts/dev_run.sh test_run --manifest tests/toy_dataset/manifest_5kb.tsv --chromosomes II

clean:
	rm -rf dev/runs/*
	rm -rf onionskin_results
	rm -rf toy_out
