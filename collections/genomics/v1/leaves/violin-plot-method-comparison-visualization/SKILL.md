---
name: violin-plot-method-comparison-visualization
description: Use when benchmarking two or more reconstruction/clustering methods on
  simulated read sets with per-method, per-condition precision and recall scores and
  the goal is a depth-stratified distribution comparison figure.
when_to_use_negative:
- The goal is a real-data (experimental) comparison rather than the simulated-data
  benchmark with ground-truth isoforms — the violin figure is defined only where ground
  truth exists
- Each method-condition combination has only a single score (no replicate runs), making
  distributional violin plots meaningless; use a line/scatter plot instead
- Input scores are already aggregated summaries (mean/CI per depth) rather than per-run
  replicates — violins require the underlying score distributions
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: IsoCon
  role: Method under evaluation whose precision/recall distributions are plotted;
    run as 'IsoCon pipeline -fl_reads <reads> -outfolder <output>' on simulated read
    sets
  repo: https://github.com/ksahlin/IsoCon
- name: ICE
  role: Baseline comparator (PacBio's Iso-Seq clustering/correction tool) run on identical
    simulated read sets; its near-0 precision at depth >= 500 is the headline violin-plot
    signal
- name: IsoCon_Eval
  role: Snakemake workflow driving all simulations, method runs, and evaluation/figure
    generation; configured via simulated_experiments.json
  repo: https://github.com/ksahlin/IsoCon_Eval
- name: Snakemake
  role: Pipeline engine that orchestrates simulation, reconstruction, scoring, and
    plot rendering steps; requires python3
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:8734bf0ab5541f2918ca14f072158991c1b861a807caf64bf59992cf4e318ff8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - ICE
  - Iso-Seq
  - IsoCon
  - IsoCon_Eval
  - Snakemake
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Violin-plot method comparison visualization

## Summary

Render grouped violin plots of nucleotide-level precision and recall as a function of read depth to compare transcript-reconstruction methods (IsoCon vs ICE) across simulated multigene-family experiments (TSPY, HSFY, DAZ), reproducing the benchmark figure of the IsoCon paper.

## When to use

Use when benchmarking two or more reconstruction/clustering methods on simulated read sets with per-method, per-condition precision and recall scores and the goal is a depth-stratified distribution comparison figure.

## When NOT to use

- The goal is a real-data (experimental) comparison rather than the simulated-data benchmark with ground-truth isoforms — the violin figure is defined only where ground truth exists
- Each method-condition combination has only a single score (no replicate runs), making distributional violin plots meaningless; use a line/scatter plot instead
- Input scores are already aggregated summaries (mean/CI per depth) rather than per-run replicates — violins require the underlying score distributions

## Inputs

- Per-method, per-condition precision and recall scores computed against simulated ground-truth isoforms (one row per gene family x mutation rate x read count x run)
- Simulated PacBio Iso-Seq read sets for TSPY, HSFY, DAZ with 30 isoforms, unequal abundances, and varying mutation rates
- Reconstructed transcript FASTA files: IsoCon final_candidates.fa and ICE output for identical read sets
- IsoCon_Eval config file simulated_experiments.json (TOOLS, READ_COUNTS, RUNS, ABUNDANCE, GENE_MEMBERS fields)

## Outputs

- Grouped violin plots of precision as a function of read depth, one series per method (IsoCon vs ICE)
- Grouped violin plots of recall as a function of read depth, across the three gene families
- Reproduction of the paper's Fig. 1 benchmark figure

## How to apply

Score each method (IsoCon from commit 79589f3 and ICE) against the simulated ground-truth isoform set, computing nucleotide-level precision and recall for each gene family (TSPY, HSFY, DAZ), mutation rate, and read depth (e.g. READ_COUNTS 20, 100, 500, 2500, 12500 in IsoCon_Eval's simulated_experiments.json). Pool the per-experiment replicate scores (multiple RUNS and ABUNDANCE settings such as constant vs exponential) so each method x depth combination yields a distribution rather than a point estimate. Render grouped violins with read depth on the x-axis, precision or recall on the y-axis, and one violin per method, faceted or grouped across the three gene families. The rationale is that precision differences (ICE collapsing toward near-0 precision while IsoCon stays above 80% at depth >= 500) are distributional — over-correction of rare variants — so violins reveal both central tendency and spread that bar charts would hide. Drive the whole simulation-to-figure loop with the IsoCon_Eval Snakemake workflow configured via simulated_experiments.json, dry-running with `snakemake -n` before launching on a cluster.

## Related tools

- **IsoCon** (Method under evaluation whose precision/recall distributions are plotted; run as 'IsoCon pipeline -fl_reads <reads> -outfolder <output>' on simulated read sets) — https://github.com/ksahlin/IsoCon
- **ICE** (Baseline comparator (PacBio's Iso-Seq clustering/correction tool) run on identical simulated read sets; its near-0 precision at depth >= 500 is the headline violin-plot signal)
- **IsoCon_Eval** (Snakemake workflow driving all simulations, method runs, and evaluation/figure generation; configured via simulated_experiments.json) — https://github.com/ksahlin/IsoCon_Eval
- **Snakemake** (Pipeline engine that orchestrates simulation, reconstruction, scoring, and plot rendering steps; requires python3)

## Examples

```
snakemake --configfile simulated_experiments.json --latency-wait 100 --verbose -n
```

## Evaluation signals

- ICE precision distributions collapse toward ~0 at read depths >= 500 while IsoCon precision remains above 80% at those depths in the regenerated plots
- IsoCon transcripts in final_candidates.fa match the published reconstruction output for the same simulated read sets
- Each violin contains scores from the configured replicate space (e.g. 10 RUNS, constant and exponential ABUNDANCE, three mutation rates) rather than single points
- Snakemake dry run (`snakemake -n --configfile simulated_experiments.json`) resolves all jobs before launching, and TOOLS/READ_COUNTS/GENE_MEMBERS fields match the intended experiment subset
- Precision and recall are computed at the nucleotide level against the simulated ground-truth isoform set, per family, mutation rate, and read depth

## Limitations

- The benchmark is meaningful only for simulated data where the ground-truth isoform set is known; on experimental data no such violin comparison can be scored
- Removing one tool or experiment changes the figure: TOOLS and EXPERIMENTS fields in simulated_experiments.json must include both ISOCON and ICE for a valid comparison
- The pipeline is compute-heavy and intended for cluster execution (sbatch cluster profile with --keep-going and many concurrent jobs)
- IsoCon results are pinned to GitHub commit 79589f3; other versions may not reproduce the precision/recall distributions

## Evidence

- [results] On simulated data, ICE's precision is close to 0 at read depths of 500 or higher, whereas IsoCon's precision always remains above 80% at those depths: [evidence span withheld — anchor status: not_found]
- [readme] README simulated experiment config: [evidence span withheld — anchor status: external_doc]
- [readme] README dry-run instruction: "try a dry run as

```
snakemake  --configfile simulated_experiments.json --latency-wait 100 --verbose  -n
"
- [discussion] IsoCon commit pinning: "The results of IsoCon in this paper were obtained with commit 79589f3 on GitHub."
- [discussion] Snakemake workflow in eval repo: "Scripts for all analyses are available at https://github.com/ksahlin/IsoCon_Eval. This repository also include a snakemake63 workflow"
