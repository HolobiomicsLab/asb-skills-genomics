---
name: false-positive-rate-estimation
description: 'Use when setting or validating a CHEUI-solo site-level significance
  cutoff: run the full CHEUI pipeline on unmodified IVT control DRS replicates and
  count called sites to measure FPR per modification.'
when_to_use_negative:
- The sample contains genuine modifications (e.g. WT cell-line RNA rather than an
  IVT negative control) — counting called sites there does not measure FPR; use an
  unmodified IVT control instead.
- Assessing differential-modification error, where significance is controlled with
  Benjamini-Hochberg FDR correction in CHEUI-diff (alpha=0.05) rather than an absolute
  site probability cutoff.
- Benchmarking stoichiometry accuracy on mixtures, where the relevant metric is Pearson
  correlation with ground truth rather than FPR on unmodified-only sites.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: CHEUI
  role: CHEUI-solo Models 1 and 2 are run on the IVT negative control; sites with
    Model 2 probability >0.9999 are counted as false positives
  repo: https://github.com/comprna/CHEUI
- name: nanopore direct RNA sequencing
  role: Source of the unmodified IVT control signal data on which the FPR is measured
- name: R2Dtool
  role: Downstream annotation/liftover of significant calls whose cutoff FDR was established
    by this estimation (cheui_to_bed.sh, R2_annotate, R2_lift)
  repo: https://github.com/comprna/R2Dtool
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:939d41e472212d360eac3f48be16c7adee731a05fdfaf71728ee1cba04ef4571
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - Epinano
  - NanoRMS
  - Nanocompore
  - R2Dtool
  - Tombo
  - Xpore
  - nanopore direct RNA sequencing
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# false-positive-rate-estimation

## Summary

Empirically quantifies the per-site false-positive rate of CHEUI-solo m6A/m5C calls by running CHEUI Models 1 and 2 on fully unmodified in-vitro-transcribed (IVT) control direct RNA reads, and establishes probability cutoffs (e.g. P>0.9999) whose FDR is estimated by shuffling read signals across transcriptomic sites.

## When to use

Use when setting or validating a CHEUI-solo site-level significance cutoff: run the full CHEUI pipeline on unmodified IVT control DRS replicates and count called sites to measure FPR per modification.

## When NOT to use

- The sample contains genuine modifications (e.g. WT cell-line RNA rather than an IVT negative control) — counting called sites there does not measure FPR; use an unmodified IVT control instead.
- Assessing differential-modification error, where significance is controlled with Benjamini-Hochberg FDR correction in CHEUI-diff (alpha=0.05) rather than an absolute site probability cutoff.
- Benchmarking stoichiometry accuracy on mixtures, where the relevant metric is Pearson correlation with ground truth rather than FPR on unmodified-only sites.

## Inputs

- Unmodified in-vitro-transcribed (IVT) HeLa WT direct RNA sequencing FAST5/FASTQ files (3 replicates, negative control)
- Reference transcriptome (GRCh38 / Gencode v38 FASTA) for minimap2 and nanopolish
- Nanopolish eventalign output (event-rescaled, signal-indexed)
- CHEUI trained models (CHEUI_m6A_model1.h5/model2.h5, CHEUI_m5C_model1.h5/model2.h5)
- k-mer model file (CHEUI/kmer_models/model_kmer.csv) for CHEUI_preprocess

## Outputs

- Per-modification FPR estimate on IVT control (reference: m6A 0.0003, m5C 0.0006 at P>0.9999; per-site FPR ~0.00074 m6A / ~0.00034 m5C at P>0.99 on IVT set 1)
- Shuffled-signal-based FDR estimate for the chosen probability cutoff (FDR = 0 for m6A; 0.000384 for m5C)
- Summary metric tables (e.g. m6A_site_metrics.tsv with FPR alongside site counts and DRACH proportion)

## How to apply

Run the standard CHEUI preprocessing chain (Guppy basecalling, minimap2 alignment, samtools -F 232457 filtering, nanopolish eventalign with --scale-events --signal-index --samples --print-read-names) on the three replicates of DRS of in-vitro-transcribed WT HeLa cells, then run CHEUI-solo Model 1 per-read prediction and Model 2 site-level prediction only on transcriptomic sites with coverage >20 reads. Count sites with Model 2 probability >0.9999 as false positives: the reference FPR is 0.0003 for m6A and 0.0006 for m5C on this IVT control. On the controlled IVT set 1 mixtures (Jenjaroenpun et al.), the per-site FPR at the looser Model 2 threshold P>0.99 is approximately 0.00074 (m6A) and 0.00034 (m5C). For cell-line call sets, establish the significance cutoff by shuffling all read signals across all transcriptomic sites while maintaining the same number of sites, giving estimated FDR = 0 for m6A and FDR = 0.000384 for m5C at P>=0.9999. Use these measured rates to justify the very strict cutoffs applied to the large number of transcriptomic sites tested.

## Related tools

- **CHEUI** (CHEUI-solo Models 1 and 2 are run on the IVT negative control; sites with Model 2 probability >0.9999 are counted as false positives) — https://github.com/comprna/CHEUI
- **nanopore direct RNA sequencing** (Source of the unmodified IVT control signal data on which the FPR is measured)
- **R2Dtool** (Downstream annotation/liftover of significant calls whose cutoff FDR was established by this estimation (cheui_to_bed.sh, R2_annotate, R2_lift)) — https://github.com/comprna/R2Dtool

## Examples

```
nanopolish eventalign -t 48 --reads <read fastq> --bam <sorted-bam-file> --genome <transcriptome fasta> --scale-events --signal-index --samples --print-read-names > nanopolish_out.txt
```

## Evaluation signals

- IVT HeLa control yields m6A FPR ≈ 0.0003 (sites with Model 2 probability >0.9999 counted as false positives)
- IVT HeLa control yields m5C FPR ≈ 0.0006 at the same 0.9999 cutoff
- On IVT set 1 fully unmodified sites at P>0.99, per-site FPR ≈ 0.00074 (m6A) and ≈ 0.00034 (m5C)
- Shuffling read signals across all transcriptomic sites (same site count) reproduces estimated FDR = 0 for m6A and 0.000384 for m5C at P>=0.9999
- FPR summary metrics accompany site calls in the output tables (e.g. m6A_site_metrics.tsv, nsun2_m5c_analysis_results.tsv)

## Limitations

- Very strict cutoffs applied to the large number of transcriptomic sites reduce CHEUI sensitivity; relaxing thresholds would recover more orthogonal-supported sites but at the cost of potential false positives
- Nanopore signal variability limits detection: differences between modified and non-modified reads are often comparable to differences among non-modified reads
- Modified sites detected only from nanopore reads still require independent validation experiments

## Evidence

- [methods] Apply CHEUI-solo Models 1 and 2 with the same 0.9999 cutoff to the three replicates of DRS of in-vitro transcribed WT HeLa cells and classify the predicted m6A sites as false positives to estimate the m6A FPR (reference value 0.0003).: [evidence span withheld — anchor status: not_found]
- [methods] FDR estimation by shuffling read signals across transcriptomic sites: "we shuffled all read signals across all transcriptomic sites, maintaining the same number of transcriptomic sites"
- [discussion] Strict cutoffs trade sensitivity for low expected FDR: "given the large number of sites tested, we used very strict cut-offs to maintain a low expected false discovery rate (FDR)."
- [readme] CHEUI preprocessing consumes nanopolish eventalign output with rescaling and signal indexing: [evidence span withheld — anchor status: external_doc]
