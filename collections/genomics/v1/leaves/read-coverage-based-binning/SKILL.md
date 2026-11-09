---
name: read-coverage-based-binning
description: Use when benchmarking m6A detection on dRNA-seq requires restricting
  per-bin calls to motif-containing, exclusion-zone-free, or high-coverage bins to
  test whether such filters improve precision-recall performance.
when_to_use_negative:
- Input is a synthetic oligos dataset, where in vivo bin restriction (motifs, exclusion
  zones, coverage) is not meaningful
- The analysis goal is raw tool benchmarking on the unfiltered bin baseline rather
  than testing the effect of bin-set restrictions
- Bins lack read-count or sequence information, so the three flags cannot be computed
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0102
tools:
- name: NanOlympicsMod
  role: Nextflow pipeline that produces the per-bin m6A score matrix and provides
    the R post-processing and statistical_analysis scripts used to recompute precision/recall
    within each filtered bin set
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine executing the containerized binning and filtering steps of
    NanOlympicsMod
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:6a9ab2d08da00dd4d5c701e5550e0963623b9ac51781e00fae01221bb1fb1738
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# read-coverage-based-binning

## Summary

Annotates fixed genome bins from dRNA-seq alignments with coverage counts and binary flags (RRACH motif presence, distance from 100-nt exon-end exclusion zones) so that m6A tool calls restricted to RRACH-containing, exclusion-zone-free, or highly expressed (>100 read counts) bins can be re-scored for precision and recall.

## When to use

Use when benchmarking m6A detection on dRNA-seq requires restricting per-bin calls to motif-containing, exclusion-zone-free, or high-coverage bins to test whether such filters improve precision-recall performance.

## When NOT to use

- Input is a synthetic oligos dataset, where in vivo bin restriction (motifs, exclusion zones, coverage) is not meaningful
- The analysis goal is raw tool benchmarking on the unfiltered bin baseline rather than testing the effect of bin-set restrictions
- Bins lack read-count or sequence information, so the three flags cannot be computed

## Inputs

- per-bin m6A score matrix from the NanOlympicsMod pipeline (fixed --binLength window)
- GENCODE M23 (mouse) / hg38 (human) genome and transcriptome annotation files
- alignment (BAM) files used for coverage computation
- m6A gold-standard peaks bed file (--peaksfile)

## Outputs

- bin annotation table with RRACH flag, exclusion-zone flag, and total read count per bin
- three filtered bin sets (RRACH-containing, exclusion-zone-free, high-coverage >100 counts)
- per-tool precision and recall values per filtered bin set
- precision-recall curves per tool and filter set

## How to apply

Load the per-bin m6A score matrix (produced at the pipeline's --binLength fixed window size) together with the genome annotation (GENCODE M23 for mouse, hg38 for human) and the alignment files used for coverage computation. Annotate every bin with three binary flags: presence of an RRACH motif in the bin sequence, distance greater than the 100-nt exon-end exclusion zone, and total read count from the alignments. Construct the three filtered bin sets — RRACH-containing bins, bins outside exclusion zones, and highly expressed bins (>100 read counts) — then recompute precision and recall for each tool's calls within each set against the m6A gold-standard peaks bed file using the pipeline's R post-processing and statistical_analysis scripts. The rationale, grounded in the article, is that coverage is a major driver of tool performance, so bin-level read-count filtering is expected to boost precision and recall, whereas the RRACH and exclusion-zone filters are expected to yield only marginal or no gains.

## Related tools

- **NanOlympicsMod** (Nextflow pipeline that produces the per-bin m6A score matrix and provides the R post-processing and statistical_analysis scripts used to recompute precision/recall within each filtered bin set) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine executing the containerized binning and filtering steps of NanOlympicsMod)

## Examples

```
git clone https://github.com/mfurla/NanOlympicsMod.git && nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- Every bin carries all three annotations: an RRACH flag, an exclusion-zone flag (>100 nt from exon ends), and a total read count
- High-coverage bin set is defined by the >100 read-count threshold; RRACH bins are the subset with a detectable RRACH motif in the bin sequence
- Recomputed PR values must reproduce the reported pattern: marginal precision gain for RRACH, no significant gain for exclusion zone, marked precision and recall improvement for high-expression bins (confirmed in mouse and human)
- Each filtered-set PR curve should be compared to the unfiltered baseline PR curve to confirm whether the restriction changes precision or recall significantly
- Filtered bin sets must be a strict subset of the bins present in the per-bin score matrix

## Limitations

- Biological datasets (yeast, mouse, human) lack ground truth for m6A positions; reference sets come from orthogonal short-read methods, so differences in bias between orthogonal references and nanopore techniques confound performance assessment
- Precision and recall are markedly influenced by sequencing depth, so bin coverage reflects depth as much as expression
- Several tools could not complete benchmark runs or produced empty outputs, restricting which tools can be re-scored within filtered bin sets

## Evidence

- [discussion] We tested whether restricting to specific sets of bins, such as those associated with RRACH motifs, high coverage or far from exclusion zones, could improve the performance: "We tested whether restricting to specific sets of bins, such as those associated with RRACH motifs, high coverage or far from exclusion zones, could improve the performance"
- [discussion] Only by filtering for highly expressed bins significatively boosted precision and recall.: "Only by filtering for highly expressed bins significatively boosted precision and recall."
- [methods] --peaksfile Path to bed file with set of m6A gold-standard peaks: "--peaksfile Path to bed file with set of m6A gold-standard peaks"
- [other] --binLength: [evidence span withheld — anchor status: not_found]
- [intro] The results were benchmarked against reference m6A sets obtained from established orthogonal techniques.: "The results were benchmarked against reference m6A sets obtained from established orthogonal techniques."
