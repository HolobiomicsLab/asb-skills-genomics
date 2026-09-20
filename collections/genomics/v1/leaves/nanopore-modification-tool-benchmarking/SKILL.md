---
name: nanopore-modification-tool-benchmarking
description: Use when evaluating how accurately nanopore RNA modification callers
  (m6A/m5C) estimate site-level stoichiometry or sensitivity/FDR on known ground-truth
  mixtures, e.g. IVT sets with 20-100% modification levels.
when_to_use_negative:
- When no ground-truth or synthetic modified/unmodified read sets exist — differential
  or exploratory modification detection is a different task than stoichiometry benchmarking
- When the comparison tools require matched WT/KO pairs but only a single-sample signal-based
  method is applicable (e.g. using CHEUI-solo alone without a KO control)
- When evaluating modifications other than m6A/m5C, since benchmark evaluation is
  restricted to k-mers with one centered modified base and no additional bases of
  the same type
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CHEUI
  role: Single-sample two-stage deep learning model (Model 1 read-level, Model 2 site/stoichiometry)
    benchmarked on WT-only IVT mixtures; Model 2 probability > 0.9999 defines called
    sites
  repo: https://github.com/comprna/CHEUI
- name: Epinano
  role: Wet/dry comparison tool run in EpiNano-Error sumErr mode on Guppy v3.0.3 basecalls
    over WT/KO pairs
- name: Nanocompore
  role: Sample-comparison tool run on WT/KO pairs; stoichiometry estimated from WT
    modified-cluster counts over total WT reads
- name: Xpore
  role: Sample-comparison tool run on WT/KO pairs; stoichiometry taken from mod_rate_WT-rep1;
    output filtered with xpore postprocessing
- name: NanoRMS
  role: Comparison tool providing k-NN and k-means WT modified fractions for stoichiometry
    estimation
- name: Tombo
  role: Sample-comparison mode tool providing modified-read fractions per site
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:2cda3c5b3221713853ebf2d8f4abb03961c5a7df6da7ce25342b5f6293ce7386
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - Epinano
  - NanoRMS
  - Nanocompore
  - Tombo
  - Xpore
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 24
---

# nanopore-modification-tool-benchmarking

## Summary

A benchmarking protocol for comparing nanopore direct RNA modification detection tools (CHEUI, Xpore, Nanocompore, Epinano, NanoRMS, Tombo) on IVT stoichiometry mixtures, measuring site-wise true/false positives and Pearson correlation of predicted vs expected modification stoichiometry.

## When to use

Use when evaluating how accurately nanopore RNA modification callers (m6A/m5C) estimate site-level stoichiometry or sensitivity/FDR on known ground-truth mixtures, e.g. IVT sets with 20-100% modification levels.

## When NOT to use

- When no ground-truth or synthetic modified/unmodified read sets exist — differential or exploratory modification detection is a different task than stoichiometry benchmarking
- When the comparison tools require matched WT/KO pairs but only a single-sample signal-based method is applicable (e.g. using CHEUI-solo alone without a KO control)
- When evaluating modifications other than m6A/m5C, since benchmark evaluation is restricted to k-mers with one centered modified base and no additional bases of the same type

## Inputs

- IVT direct RNA nanopore reads, modified and unmodified, per modification type (m6A, m5C) from Jenjaroenpun et al.
- FAST5 raw signal files and basecalled FASTQ (Guppy; e.g. v3.0.3 for Epinano, v4.0.11+ for CHEUI)
- BAM alignments to a reference transcriptome (minimap2, primary positive-strand, samtools -F 232457 filtered)
- Nanopolish eventalign output with --scale-events --signal-index --samples --print-read-names
- Reference k-mer model file (model_kmer.csv) for CHEUI preprocessing

## Outputs

- benchmark_results.csv with per-tool TP counts, FP counts, and Pearson correlation coefficients at each stoichiometry level
- Per-site stoichiometry estimates per tool (CHEUI Model 2 probabilities, Xpore mod_rate, Nanocompore modified-cluster fractions, NanoRMS modified fractions, Tombo modified-read fractions)
- TP calls restricted to clean centered-modification 9-mers and FP rates from unmodified-only controls

## How to apply

Construct mock WT/KO sample pairs by randomly sampling reads from modified and unmodified Jenjaroenpunpun et al. IVT read sets to build stoichiometry mixtures of 20, 40, 60, 80, and 100% (WT) plus unmodified-only reads (KO), then run sample-comparison tools (Epinano in sumErr mode on Guppy v3.0.3 basecalls, Nanocompore, Xpore) on WT/KO pairs and single-sample tools (CHEUI-solo) on WT alone with default parameters at alpha = 0.05/0.01/0.001 with multiple-testing correction. Restrict true-positive evaluation to k-mers (k = 9) containing one centered m6A with no additional As (or one centered m5C with no additional Cs), and estimate false-positive rates by running each tool on unmodified-only read sets. Compute per-site stoichiometry with tool-specific readouts: CHEUI Model 2 probability > 0.9999 sites with per-read Model 1 probability > 0.7 / < 0.3 calls, Xpore mod_rate_WT-rep1 values, Nanocompore WT modified-cluster counts over total WT reads, NanoRMS (k-NN and k-means) modified fractions, and Tombo modified-read fractions. Finally compute Pearson correlations between predicted and expected stoichiometry per tool per modification type; the paper's reference value is r = 0.839 for both m6A and m5C for CHEUI-solo.

## Related tools

- **CHEUI** (Single-sample two-stage deep learning model (Model 1 read-level, Model 2 site/stoichiometry) benchmarked on WT-only IVT mixtures; Model 2 probability > 0.9999 defines called sites) — https://github.com/comprna/CHEUI
- **Epinano** (Wet/dry comparison tool run in EpiNano-Error sumErr mode on Guppy v3.0.3 basecalls over WT/KO pairs)
- **Nanocompore** (Sample-comparison tool run on WT/KO pairs; stoichiometry estimated from WT modified-cluster counts over total WT reads)
- **Xpore** (Sample-comparison tool run on WT/KO pairs; stoichiometry taken from mod_rate_WT-rep1; output filtered with xpore postprocessing)
- **NanoRMS** (Comparison tool providing k-NN and k-means WT modified fractions for stoichiometry estimation)
- **Tombo** (Sample-comparison mode tool providing modified-read fractions per site)

## Examples

```
nanopolish eventalign -t 48 --reads <read fastq> --bam <sorted-bam-file> --genome <transcriptome fasta> --scale-events --signal-index --samples --print-read-names > nanopolish_out.txt
```

## Evaluation signals

- CHEUI-solo stoichiometry Pearson correlation on the IVT set 2 benchmark should reproduce r = 0.839 for m6A and r = 0.839 for m5C
- Per-tool TP counts should be restricted to 9-mers with exactly one centered modified base (m6A without additional As; m5C without additional Cs)
- FP rates must be computed against unmodified-only read sets (single unmodified set for CHEUI) and should be near zero for the top-performing configuration
- TP/FP trends across the 20/40/60/80/100% mixtures should match the patterns reported in Fig. 2f-2h
- Multiple-testing-corrected significance thresholds at alpha = 0.05, 0.01, and 0.001 should be applied consistently across comparison tools, and CHEUI-solo Model 2 positivity should use probability > 0.99

## Limitations

- Very strict cut-offs used on cell transcriptome sites to maintain low expected FDR reduce CHEUI sensitivity; relaxing thresholds would recover more sites but at the cost of potential false positives
- Nanopore signal variability limits single-molecule discrimination, since differences between modified and non-modified read signals are often comparable to variation among non-modified reads
- Biases and differences in detection rates exist between technologies, so per-tool TP/FP results should not be extrapolated as absolute modification abundance

## Evidence

- [results] finding: CHEUI-solo outperformed all other tools in site-wise stoichiometry prediction on the independent IVT benchmark, achieving a higher correlation with ground truth: [evidence span withheld — anchor status: not_found]
- [methods] Build mock WT and KO samples by randomly sampling reads from the modified and unmodified Jenjaroenpunpun et al. IVT sets: [evidence span withheld — anchor status: not_found]
- [methods] Evaluate true positives only on k-mers (k = 9) containing one centered m6A with no additional As: [evidence span withheld — anchor status: not_found]
- [readme] Nanopolish eventalign invocation requirements from the CHEUI README: [evidence span withheld — anchor status: external_doc]
- [discussion] Strict cutoffs trade sensitivity for FDR control in the transcriptome-wide application: "we used very strict cut-offs to maintain a low expected false discovery rate (FDR). While these strict cutoffs resulted in a reduction of CHEUI sensitivity"
