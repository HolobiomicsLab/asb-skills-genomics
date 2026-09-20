---
name: differential-pr-significance-testing
description: Use when per-bin m6A scores from Nanopore dRNA-seq tools are already
  computed and you need to quantify whether restricting the precision-recall analysis
  to motif-containing, exclusion-zone-free, or high-coverage bins significantly improves
  tool performance.
when_to_use_negative:
- The per-bin m6A score matrix and tool calls do not yet exist — run the NanOlympicsMod
  pipeline and bin-level scoring first (task_003).
- Working on the synthetic oligos dataset, which lacks in vivo mark location/frequency
  and the confounding effect of proximal RNA modifications; the motif/coverage filters
  are intended for biological (yeast, mouse, human) datasets.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: NanOlympicsMod
  role: Containerized Nextflow pipeline whose R post-processing and statistical_analysis
    scripts recompute precision/recall per filtered bin set and generate PR curves
    against the --peaksfile gold standard
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Execution framework for the containerized NanOlympicsMod pipeline that produced
    the per-bin scores being re-evaluated
  repo: https://github.com/mfurla/NanOlympicsMod
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:a40749ee4cd76883e04c0f1e9f7df50e959f5806df5c9527042b669cb9b46dbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# differential-pr-significance-testing

## Summary

Recompute precision and recall for dRNA-seq m6A detection tool calls under alternative bin-set restrictions (RRACH motif bins, bins outside 100-nt exon-end exclusion zones, highly expressed bins >100 counts) against m6A gold-standard peaks, and test whether each restriction significantly changes performance relative to the unfiltered baseline.

## When to use

Use when per-bin m6A scores from Nanopore dRNA-seq tools are already computed and you need to quantify whether restricting the precision-recall analysis to motif-containing, exclusion-zone-free, or high-coverage bins significantly improves tool performance.

## When NOT to use

- The per-bin m6A score matrix and tool calls do not yet exist — run the NanOlympicsMod pipeline and bin-level scoring first (task_003).
- Working on the synthetic oligos dataset, which lacks in vivo mark location/frequency and the confounding effect of proximal RNA modifications; the motif/coverage filters are intended for biological (yeast, mouse, human) datasets.

## Inputs

- Per-bin m6A score matrix from task_003 (fixed --binLength windows over genome bins)
- GENCODE M23 (mouse) / hg38 (human) genome and transcriptome annotation files
- Alignment files (BAM) used for per-bin coverage computation
- m6A gold-standard peaks bed file (--peaksfile)
- Per-tool m6A call sets from NanOlympicsMod runs (with 'default' or 'relaxed' --threshold filtering)

## Outputs

- Annotated bin table with RRACH flag, exclusion-zone flag, and read-count flag per bin
- Filtered bin sets: RRACH-containing, exclusion-zone-free, highly expressed (>100 read counts)
- Per-tool precision and recall values within each filtered bin set
- Precision-recall curves per tool and filter set, with significance tests vs. the unfiltered baseline

## How to apply

Load the per-bin m6A score matrix (produced with the NanOlympicsMod pipeline's --binLength setting) together with GENCODE M23 (mouse) / hg38 (human) annotations and the alignment files used for coverage. Annotate each genome bin with three binary flags: RRACH motif presence in the bin sequence, distance greater than the 100-nt exon-end exclusion zone, and total read count from the alignments. Construct the three filtered bin sets, then recompute precision and recall per tool against the m6A gold-standard peaks bed file (passed via the pipeline's --peaksfile argument) using NanOlympicsMod's R post-processing and statistical_analysis scripts. Generate precision-recall curves per tool and filter set (as in the pipeline's PR curve plotting thresholds) and test whether each restriction significantly changes precision or recall relative to the unfiltered baseline. Validate against the reported pattern: marginal precision gain for the RRACH filter, no significant gain for the exclusion-zone filter, and a marked precision AND recall improvement for the >100-count high-expression filter.

## Related tools

- **NanOlympicsMod** (Containerized Nextflow pipeline whose R post-processing and statistical_analysis scripts recompute precision/recall per filtered bin set and generate PR curves against the --peaksfile gold standard) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Execution framework for the containerized NanOlympicsMod pipeline that produced the per-bin scores being re-evaluated) — https://github.com/mfurla/NanOlympicsMod

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- The high-expression (>100 read counts) filter shows a marked improvement in BOTH precision and recall on the mouse PR analysis, confirmed in human.
- The RRACH filter shows only a marginal precision gain.
- The exclusion-zone (100 nt at exon ends) filter shows no significant gain in precision or recall.
- Each filtered-bin-set PR curve and AUPRC can be traced back to calls benchmarked against the --peaksfile gold-standard bed.
- Bin annotation flags are internally consistent: each bin carries exactly one RRACH, exclusion-zone-distance, and read-count value derived from the same --binLength windows and alignments.

## Limitations

- Biological datasets (yeast, mouse, human) lack a true ground truth; the gold-standard reference m6A sets come from orthogonal short-read methods whose biases may confound nanopore-based performance assessment.
- Precision/recall estimates are markedly influenced by sequencing depth; the >100-count filter trades bin count for reliability and may leave few evaluable bins in low-coverage datasets.
- Several m6A detection tools could not complete NanOlympicsMod benchmark runs (abandoned due to lack of progress or empty output files), so per-tool comparisons may not cover all 14 tools.

## Evidence

- [discussion] Only by filtering for highly expressed bins significatively boosted precision and recall.: "Only by filtering for highly expressed bins significatively boosted precision and recall."
- [discussion] Restricting calls to specific bin sets to test performance improvement: "We tested whether restricting to specific sets of bins, such as those associated with RRACH motifs, high coverage or far from exclusion zones, could improve the performance"
- [methods] Gold-standard peaks bed file input parameter: "--peaksfile Path to bed file with set of m6A gold-standard peaks"
- [methods] Threshold filtering of m6A sites in the pipeline: "--threshold Set of thresholds to use for the filtering of m6A sites (choose between 'default' and 'relaxed')"
- [readme] Pipeline run command from README: "nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir""
