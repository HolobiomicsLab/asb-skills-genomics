---
name: isoform-depth-threshold-verification
description: Use when reproducing or validating IsoCon sensitivity curves on simulated
  PacBio Iso-Seq read sets (e.g., Supplementary Fig. 1) to determine capture depth
  thresholds per isoform or gene family.
when_to_use_negative:
- When reads are not pre-processed with the Iso-Seq bioinformatic pipeline to remove
  primers, barcodes, and chimeras — IsoCon assumes this pre-processing
- When the goal is per-base accuracy benchmarking against ICE on real (not simulated)
  data rather than isoform capture-depth thresholding
- When a ground-truth isoform list and per-isoform read counts are unavailable, since
  fate classification requires mapping output candidates back to simulated isoforms
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: IsoCon
  role: Runs the pipeline (error correction + statistical filtering) on each simulated
    read set to produce final_candidates.fa, not_converged.fa, and cluster_info.tsv
    for isoform fate classification
  repo: https://github.com/ksahlin/IsoCon
- name: parasail
  role: Used by IsoCon internally to create pairwise alignments from each read to
    the cluster center during iterative error correction
  repo: https://github.com/jeffdaily/parasail-python
- name: IsoCon_Eval
  role: Snakemake pipeline supplying the simulated read sets, read counts, abundances,
    and replicates used for the depth experiments
  repo: https://github.com/ksahlin/IsoCon_Eval
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:8c2ea66cad7a49d22e3ab653ec7324447ec0dd22880fcd77505eb9c8fa7652ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - IsoCon
  - IsoCon_Eval
  - parasail
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# isoform-depth-threshold-verification

## Summary

Verifies the minimum per-isoform sequencing depth at which IsoCon recovers simulated transcripts from highly similar multigene families (e.g., TSPY, HSFY), by classifying each isoform as captured, filtered in the statistical step, or lost during error correction.

## When to use

Use when reproducing or validating IsoCon sensitivity curves on simulated PacBio Iso-Seq read sets (e.g., Supplementary Fig. 1) to determine capture depth thresholds per isoform or gene family.

## When NOT to use

- When reads are not pre-processed with the Iso-Seq bioinformatic pipeline to remove primers, barcodes, and chimeras — IsoCon assumes this pre-processing
- When the goal is per-base accuracy benchmarking against ICE on real (not simulated) data rather than isoform capture-depth thresholding
- When a ground-truth isoform list and per-isoform read counts are unavailable, since fate classification requires mapping output candidates back to simulated isoforms

## Inputs

- Simulated PacBio Iso-Seq read sets (10 replicates per condition) from the IsoCon_Eval repository
- Per-isoform read counts for each simulated isoform
- simulated_experiments. config file (GENE_MEMBERS, READ_COUNTS, RUNS, ABUNDANCE, TOOLS fields)
- IsoCon outputs: final_candidates.fa, not_converged.fa, cluster_info.tsv

## Outputs

- Per-isoform fate classifications (captured / filtered in statistical step / lost in error correction) aggregated across replicates
- Stripplot of isoform fate versus experiment depth and per-isoform depth (fig_s1_isoform_capture_stripplot.png)
- Verified per-isoform capture depth thresholds (~3 reads for TSPY, ~10 reads for HSFY)

## How to apply

Obtain the simulated PacBio Iso-Seq read sets from the ksahlin/IsoCon_Eval repository, which runs 10 replicates per condition across read counts (e.g., 20, 100, 500, 2500, 12500) via a snakemake pipeline configured in simulated_experiments.. Run 'IsoCon pipeline -fl_reads <reads> -outfolder <dir>' on each replicate to produce final_candidates.fa, not_converged.fa, and cluster_info.tsv. Using cluster_info.tsv together with per-isoform read counts, classify each simulated isoform as captured (present in final_candidates.fa), filtered in the statistical step, or lost during error correction (present only in not_converged.fa). Aggregate fate classifications across replicates as a function of experiment depth and per-isoform depth, then plot isoform fate versus depth (e.g., a seaborn stripplot saved as fig_s1_isoform_capture_stripplot.png). Validate that the minimum per-isoform depth at which isoforms are captured matches the reported thresholds: approximately 3 reads for TSPY and approximately 10 reads for HSFY; note that mutation rate plays only a minor role compared to transcript depth, with low-depth candidates mostly lost in the error-correction step.

## Related tools

- **IsoCon** (Runs the pipeline (error correction + statistical filtering) on each simulated read set to produce final_candidates.fa, not_converged.fa, and cluster_info.tsv for isoform fate classification) — https://github.com/ksahlin/IsoCon
- **parasail** (Used by IsoCon internally to create pairwise alignments from each read to the cluster center during iterative error correction) — https://github.com/jeffdaily/parasail-python
- **IsoCon_Eval** (Snakemake pipeline supplying the simulated read sets, read counts, abundances, and replicates used for the depth experiments) — https://github.com/ksahlin/IsoCon_Eval

## Examples

```
IsoCon pipeline -fl_reads simulated_reads.fastq -outfolder sim_out && snakemake --configfile simulated_experiments. --latency-wait 100 --verbose -n
```

## Evaluation signals

- Minimum per-isoform depth at which isoforms appear in final_candidates.fa is approximately 3 reads for TSPY and approximately 10 reads for HSFY
- Each simulated isoform in every replicate maps to exactly one of three fates: captured, filtered in the statistical step, or lost in error correction
- Isoforms present only in not_converged.fa (and absent from final_candidates.fa) are consistently classified as lost during error correction
- Aggregated fate counts across the 10 replicates per condition show lower-depth candidates predominantly lost in error correction, with mutation rate having only a minor effect
- Snakemake dry run ('snakemake --configfile simulated_experiments. --latency-wait 100 --verbose -n') resolves the expected READ_COUNTS x RUNS x ABUNDANCE job DAG before execution

## Limitations

- Thresholds were calibrated for TSPY and HSFY gene-family simulations; other gene families or abundance models (constant vs. exponential) may shift the capture depth
- Reads must be pre-processed by the Iso-Seq bioinformatic pipeline (primer/barcode removal, chimera filtering) before IsoCon
- IsoCon does not distinguish alleles from distinct gene copies — downstream analysis is needed for gene-copy assignment, so capture does not imply copy resolution
- Results in the paper were obtained with a specific IsoCon commit (79589f3 on GitHub); later versions may alter filtering behavior (e.g., --min_test_ratio default 5, alpha 0.01, variant-position threshold 10)

## Evidence

- [results] IsoCon captures most TSPY transcripts at a per-isoform depth of >3 reads, while for HSFY this threshold is approximately 10 reads; the mutation rate plays only a minor role compared to transcript depth, with lower-depth candidates mostly lost in the error correction step.: [evidence span withheld — anchor status: not_found]
- [readme] IsoCon pipeline CLI invocation: "IsoCon pipeline -fl_reads <reads.fastq> -outfolder </path/to/output>"
- [readme] Simulation experiment configuration in IsoCon_Eval: [evidence span withheld — anchor status: external_doc]
- [discussion] Scripts for the depth experiments live in IsoCon_Eval: "Scripts for all analyses are available at https://github.com/ksahlin/IsoCon_Eval. This repository also include a snakemake63 workflow"
- [discussion] Iterative removal of statistically non-significant candidates defines the filtering fate category: "iterative removal of statistically non-significant candidates."
