---
name: stoichiometry-correlation-analysis
description: Use when benchmarking site-wise modification stoichiometry estimation
  from nanopore direct RNA reads against ground-truth IVT stoichiometry mixtures (20-100%),
  or when reproducing the CHEUI vs competing-tools accuracy comparison.
when_to_use_negative:
- Goal is differential modification detection between two biological conditions rather
  than absolute stoichiometry (CHEUI-diff / sample-comparison mode is the appropriate
  path)
- Data are not Oxford Nanopore direct RNA sequencing reads with ionic-current signals
  (e.g., short-read or amplicon data have no signal level for CHEUI/signal-based tools)
- Target modification is not m6A or m5C; the CHEUI trained models are specific to
  these modifications (pseudouridine/other mods require alternative models, e.g. SWARM)
tools:
- name: CHEUI
  role: Two-stage neural network predicting m6A/m5C per read (Model 1) and site stoichiometry
    (Model 2) from a single sample; its Pearson correlation with ground truth (r =
    0.839 for both m6A and m5C) is the benchmark target.
  repo: github.com/comprna/CHEUI-public
- name: Epinano
  role: Base-calling error based WT/KO comparison tool (EpiNano-Error sumErr mode,
    Guppy v3.0.3 basecalls) benchmarked for stoichiometry accuracy.
- name: Nanocompore
  role: Sample-comparison tool; stoichiometry estimated from WT-modified-cluster counts
    over total WT reads.
- name: Xpore
  role: Sample-comparison tool; stoichiometry taken from xpore postprocessing-filtered
    mod_rate_WT-rep1 values.
- name: NanoRMS
  role: Provides alternative stoichiometry estimates via k-NN and k-means WT modified
    fractions.
- name: Tombo
  role: Sample-comparison mode tool; stoichiometry from modified-read fractions.
- name: Nanopolish
  role: Resquiggles and eventaligns read signals to the transcript reference (--scale-events
    --signal-index --samples --print-read-names) as CHEUI preprocessing.
- name: Minimap2
  role: Aligns basecalled reads to the reference transcriptome before signal-to-transcript
    alignment.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:4c9d45f9949ac0cd55b5650ac8f1b95a1ae36a38224f6495349505924da88987
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHEUI
  - Epinano
  - Minimap2
  - NanoRMS
  - Nanocompore
  - Nanopolish
  - Tombo
  - Xpore
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# stoichiometry-correlation-analysis

## Summary

Benchmark of site-wise RNA modification stoichiometry prediction (m6A, m5C) from nanopore direct RNA data, comparing CHEUI-solo against Xpore, Nanocompore, Epinano, NanoRMS and Tombo on controlled IVT mixtures by Pearson correlation with expected stoichiometry.

## When to use

Use when benchmarking site-wise modification stoichiometry estimation from nanopore direct RNA reads against ground-truth IVT stoichiometry mixtures (20-100%), or when reproducing the CHEUI vs competing-tools accuracy comparison.

## When NOT to use

- Goal is differential modification detection between two biological conditions rather than absolute stoichiometry (CHEUI-diff / sample-comparison mode is the appropriate path)
- Data are not Oxford Nanopore direct RNA sequencing reads with ionic-current signals (e.g., short-read or amplicon data have no signal level for CHEUI/signal-based tools)
- Target modification is not m6A or m5C; the CHEUI trained models are specific to these modifications (pseudouridine/other mods require alternative models, e.g. SWARM)

## Inputs

- Nanopore FAST5 raw signal files from Jenjaroenpun et al. IVT m6A and m5C sequencing experiments (modified and unmodified)
- FASTQ reads basecalled with Guppy (v3.0.3 for the comparison tools; Guppy 4.0.11+ template_rna_r9.4.1_70bps model for CHEUI)
- Sorted BAM of primary, positive-strand transcriptome alignments (minimap2 -ax map-ont -k14, samtools best-match filtering)
- Nanopolish eventalign output (with --scale-events --signal-index --samples --print-read-names)
- CHEUI k-mer expected-signal means file (model_kmer.csv) and pretrained CHEUI_m6A/m5C model1.h5 and model2.h5
- Reference transcriptome FASTA

## Outputs

- benchmark_results.csv with per-tool true-positive counts, false-positive counts, and Pearson correlation coefficients
- Per-site stoichiometry estimates per tool (e.g., CHEUI Model 2 site probabilities, Xpore mod_rate_WT-rep1, Nanocompore WT modified-cluster fractions, NanoRMS k-NN/k-means modified fractions, Tombo modified-read fractions)
- Site-level modification calls at alpha = 0.05, 0.01, 0.001 (multiple-testing corrected) and CHEUI-solo Model 2 positivity at probability > 0.99

## How to apply

Build mock WT/KO samples by randomly sampling reads from modified and unmodified Jenjaroenpun et al. IVT sets to create stoichiometry mixtures of 20, 40, 60, 80, and 100 percent (WT), with unmodified-only reads as KO; run Epinano (Guppy v3.0.3 basecalling, EpiNano-Error sumErr mode), Nanocompore and Xpore on WT/KO pairs, and CHEUI on the WT sample alone, at alpha = 0.05, 0.01, and 0.001 with multiple-testing correction and CHEUI-solo Model 2 positivity at probability > 0.99. Restrict true positives to k-mers (k = 9) containing one centered m6A with no additional As (or one centered m5C with no additional Cs); estimate false-positive rates by calling sites modified on unmodified-only read sets. Compute per-site stoichiometry from tool-specific outputs: CHEUI Model 2 probability > 0.9999 sites with per-read Model 1 probability > 0.7 / < 0.3 calls, Xpore mod_rate_WT-rep1, Nanocompore WT-modified-cluster counts over total WT reads, NanoRMS (k-NN and k-means) WT modified fractions, and Tombo modified-read fractions, then compute Pearson correlations between predicted and expected stoichiometry.

## Related tools

- **CHEUI** (Two-stage neural network predicting m6A/m5C per read (Model 1) and site stoichiometry (Model 2) from a single sample; its Pearson correlation with ground truth (r = 0.839 for both m6A and m5C) is the benchmark target.) — github.com/comprna/CHEUI-public
- **Epinano** (Base-calling error based WT/KO comparison tool (EpiNano-Error sumErr mode, Guppy v3.0.3 basecalls) benchmarked for stoichiometry accuracy.)
- **Nanocompore** (Sample-comparison tool; stoichiometry estimated from WT-modified-cluster counts over total WT reads.)
- **Xpore** (Sample-comparison tool; stoichiometry taken from xpore postprocessing-filtered mod_rate_WT-rep1 values.)
- **NanoRMS** (Provides alternative stoichiometry estimates via k-NN and k-means WT modified fractions.)
- **Tombo** (Sample-comparison mode tool; stoichiometry from modified-read fractions.)
- **Nanopolish** (Resquiggles and eventaligns read signals to the transcript reference (--scale-events --signal-index --samples --print-read-names) as CHEUI preprocessing.)
- **Minimap2** (Aligns basecalled reads to the reference transcriptome before signal-to-transcript alignment.)

## Examples

```
minimap2 -ax map-ont -k14 <transcriptome fasta> <read fastq> | samtools view -F 2324 -b | samtools sort > <sorted-bam-file>
```

## Evaluation signals

- CHEUI-solo stoichiometry Pearson correlation should reproduce the reported r = 0.839 for both m6A and m5C on the independent IVT benchmark
- Per-tool TP counts, FP counts and correlation coefficients written to benchmark_results.csv should match the trends in Fig. 2f-2h
- True positives are evaluated only on k = 9 mers with one centered m6A and no additional As (or one centered m5C and no additional Cs) at each stoichiometry level (20, 40, 60, 80, 100 percent)
- CHEUI Model 2 stoichiometry uses sites with probability > 0.9999 and per-read Model 1 probabilities > 0.7 / < 0.3
- False-positive rate estimated on unmodified-only read sets should remain low for the top-performing tool

## Limitations

- Strict cut-offs on cell transcriptome sites maintain a low expected FDR but reduce CHEUI sensitivity; relaxing thresholds recovers more orthogonal-method sites at the cost of potential false positives
- Nanopore signal variability means differences between modified and non-modified reads are often comparable to variation among non-modified reads, limiting per-read discrimination
- Modified sites detected only from nanopore reads require independent validation experiments; biases and differences in detection rates exist between technologies

## Evidence

- [intro] CHEUI is based on a two-stage neural network and was trained using read signals generated from in-vitro transcripts (IVTs): "CHEUI is based on a two-stage neural network and was trained using read signals generated from in-vitro transcripts (IVTs)"
- [discussion] CHEUI accomplishes high sensitivity, precision, and modification stoichiometry accuracy compared to other nanopore-based tools.: "These analyses showed that CHEUI accomplishes high sensitivity, precision, and modification stoichiometry accuracy compared to other nanopore-based tools."
- [methods] Reads were then filtered to select the best match for each read using samtools -F 232457.: "Reads were then filtered to select the best match for each read using samtools -F 232457."
- [methods] Nanocompore WT-modified-cluster counts over total WT reads, NanoRMS (k-NN and k-means) WT modified fractions, and Tombo modified-read fractions: "For Xpore, we used the module xpore postprocessing to filter the output."
- [readme] CHEUI model 1 (read level predictions) and model 2 (site level predictions) use different predictive models for m6A and m5C: [evidence span withheld — anchor status: external_doc]
