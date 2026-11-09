---
name: exclusion-zone-bin-filtering
description: Use when benchmarking m6A callers on Nanopore dRNA-seq data at fixed-window
  bin resolution and you want to test whether exclusion zones (e.g., 100 nt at exon
  ends) or related bin-set filters (RRACH motifs, high coverage) change precision/recall.
when_to_use_negative:
- 'When the goal is improving tool performance by pre-filtering calls: the paper found
  the exclusion-zone filter did not significantly boost precision or recall — the
  >100 read-count expression filter was the effective one'
- On the synthetic oligos dataset, which poorly recapitulates in vivo mark location
  and frequency and is not the intended setting for accessible-bin evaluation
- When analysis is at per-site rather than per-bin resolution (fixed --binLength windows),
  since the exclusion-zone definition is bin-based
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NanOlympicsMod
  role: Nextflow pipeline that bins the dRNA-seq data, defines accessible bins (100-nt
    exon-end exclusion zones), and provides the R post-processing and statistical_analysis
    scripts for recomputing precision and recall per filtered bin set
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Execution engine for the containerized NanOlympicsMod workflow that produces
    the per-bin score matrix and filter annotations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:ac2ce6c515d676e0a76b5afd1d0aa4f7b291068e8226574b23e2daf8f8db08bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# exclusion-zone-bin-filtering

## Summary

Restrict the evaluation of Nanopore dRNA-seq m6A detection tool calls to 'accessible' genome bins located more than 100 nt from exon ends, then recompute precision and recall against the m6A gold-standard peaks to test whether removing bins near exon boundaries improves tool performance.

## When to use

Use when benchmarking m6A callers on Nanopore dRNA-seq data at fixed-window bin resolution and you want to test whether exclusion zones (e.g., 100 nt at exon ends) or related bin-set filters (RRACH motifs, high coverage) change precision/recall.

## When NOT to use

- When the goal is improving tool performance by pre-filtering calls: the paper found the exclusion-zone filter did not significantly boost precision or recall — the >100 read-count expression filter was the effective one
- On the synthetic oligos dataset, which poorly recapitulates in vivo mark location and frequency and is not the intended setting for accessible-bin evaluation
- When analysis is at per-site rather than per-bin resolution (fixed --binLength windows), since the exclusion-zone definition is bin-based

## Inputs

- Per-bin m6A score matrix from the NanOlympicsMod pipeline (e.g., task_003 output)
- GENCODE M23 (mouse) / hg38 (human) genome and transcriptome annotation files
- BAM/CRAM alignment files used for coverage computation
- Bed file of m6A gold-standard peaks (--peaksfile)

## Outputs

- Exclusion-zone-free (accessible) bin set with per-bin RRACH, exclusion-zone, and read-count flags
- Per-tool precision and recall values within the filtered bin set
- Precision-recall curves per tool and filter set
- Statistical test of precision/recall change versus the unfiltered baseline

## How to apply

Annotate every genome bin produced with the pipeline's --binLength setting with three binary flags: RRACH motif presence in the bin sequence, distance greater than 100 nt from exon-end exclusion zones, and total read count from the alignments. Construct the exclusion-zone-free bin set (bins farther than 100 nt from exon ends, i.e., the accessible-bins definition in the Methods), alongside the RRACH-containing and highly expressed (>100 read counts) bin sets for comparison. Recompute precision and recall for each tool's calls within the filtered bin set against the m6A gold-standard peaks bed file, using the NanOlympicsMod R post-processing and statistical_analysis scripts, and generate per-tool, per-filter precision-recall curves. Test whether the restriction significantly changes precision or recall relative to the unfiltered baseline. Expected outcome per the paper: the exclusion-zone filter yields no significant gain (unlike the >100-count expression filter, which markedly boosted precision and recall in mouse, confirmed in human).

## Related tools

- **NanOlympicsMod** (Nextflow pipeline that bins the dRNA-seq data, defines accessible bins (100-nt exon-end exclusion zones), and provides the R post-processing and statistical_analysis scripts for recomputing precision and recall per filtered bin set) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Execution engine for the containerized NanOlympicsMod workflow that produces the per-bin score matrix and filter annotations)

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- Every retained bin must be >100 nt from an exon end; verify the exclusion-zone flag against the GENCODE M23/hg38 exon annotations
- Per-tool precision and recall in the exclusion-zone-free set should not differ significantly from the unfiltered baseline (the paper's reported null result)
- As a positive control in the same comparison, the >100 read-count expression filter should show a marked precision and recall improvement in mouse, confirmed in human
- Precision-recall curves per tool and filter set should be reproducible with the pipeline's R post-processing and statistical_analysis scripts
- Bin counts in the filtered set must match the intersection of the --binLength window grid and the exclusion-zone definition

## Limitations

- The paper reports no significant performance gain from the exclusion-zone filter; only the high-expression (>100 counts) filter significantly boosted precision and recall
- Biological datasets lack m6A ground truth; reference sets come from orthogonal short-read methods, so bias differences between orthogonal references and nanopore-based techniques confound performance assessment
- Low m6A stoichiometry in biological datasets only partially explains reduced F1 scores, so filtered-bin PR results inherit this residual uncertainty

## Evidence

- [discussion] tested the exclusion-zone-free bin restriction as a candidate performance-improvement filter: "We tested whether restricting to specific sets of bins, such as those associated with RRACH motifs, high coverage or far from exclusion zones, could improve the performance"
- [discussion] exclusion-zone filtering gave no significant gain, unlike the expression filter: "Only by filtering for highly expressed bins significatively boosted precision and recall."
- [discussion] the exclusion zone is defined as 100 nt at exon ends in the accessible-bins definition: [evidence span withheld — anchor status: not_found]
- [readme] NanOlympicsMod provides the R post-processing and statistical scripts used for the filtered-bin PR analysis: "a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1 reference genome"
