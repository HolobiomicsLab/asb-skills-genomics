---
name: replicate-read-count-aggregation
description: Use when isoCirc isocirc.out tables from multiple technical and biological
  replicates must be pooled, filtered at a minimum readCount threshold (>=2), and
  split into one-replicate vs. two-replicate BSJ sets for reproducibility analysis.
when_to_use_negative:
- When only a single-technical-replicate isoCirc run exists and no replicate structure
  is available for aggregation or support stratification
- When the goal is full-length circRNA isoform characterization rather than BSJ-level
  reproducibility filtering — use the isoCirc pipeline's isoform identification instead
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3512
tools:
- name: isoCirc
  role: Source of the isocirc.out BSJ tables (readCount, coordinates, isKnownBSJ)
    that are aggregated across technical replicates
  repo: https://github.com/Xinglab/isoCirc
- name: Python
  role: pandas-based aggregation of readCount over technical replicates and stratification
    by biological-replicate support
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:3c49c987330c1b787249f807215d66f7ae57be715e9995bbec324e660076df54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - Python
  - bedtools
  - isoCirc
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# replicate-read-count-aggregation

## Summary

Aggregates isoCirc BSJ read counts across technical replicates and stratifies BSJs by biological-replicate support (detected in one vs. both biological replicates) to define reproducible high-confidence back-splice junction sets before downstream annotation.

## When to use

Use when isoCirc isocirc.out tables from multiple technical and biological replicates must be pooled, filtered at a minimum readCount threshold (>=2), and split into one-replicate vs. two-replicate BSJ sets for reproducibility analysis.

## When NOT to use

- When only a single-technical-replicate isoCirc run exists and no replicate structure is available for aggregation or support stratification
- When the goal is full-length circRNA isoform characterization rather than BSJ-level reproducibility filtering — use the isoCirc pipeline's isoform identification instead

## Inputs

- isoCirc isocirc.out tables (35-column format with startCoor0based/endCoor/geneStrand, readCount, isKnownBSJ) per technical replicate
- Mapping of technical replicates to biological replicates (sample sheet or naming convention)

## Outputs

- Aggregated high-confidence BSJ table with readCount summed over technical replicates (>=2 reads)
- Per-BSJ replicate-support labels (one biological replicate vs. both)
- Summary statistics CSV (e.g. known_novel_bsj_summary.csv) with totals and fractions per stratum

## How to apply

Parse each 35-column isocirc.out table (BSJ genomic coordinates, readCount, isKnownBSJ fields) and sum readCount across technical replicates within each biological replicate, retaining only BSJs supported by >= 2 reads as 'high-confidence'. Then classify each filtered BSJ by biological-replicate support: detected in one biological replicate vs. detected in both. Compute summary statistics (total counts and known/novel fractions) separately for each stratum, because replicate-consistent BSJs behave differently — e.g. HEK293 BSJs found in both biological replicates are 85.1% known in circBase/MiOncoCirc, versus only 46.9% known for BSJs found in one replicate out of 49,993 total high-confidence BSJs. Write per-BSJ labels plus summary statistics to a CSV for validation against the expected numbers.

## Related tools

- **isoCirc** (Source of the isocirc.out BSJ tables (readCount, coordinates, isKnownBSJ) that are aggregated across technical replicates) — https://github.com/Xinglab/isoCirc
- **Python** (pandas-based aggregation of readCount over technical replicates and stratification by biological-replicate support)

## Examples

```
python aggregate_bsj_reads.py --isocirc-out rep1a.isocirc.out,rep1b.isocirc.out,rep2a.isocirc.out,rep2b.isocirc.out --min-reads 2 --out known_novel_bsj_summary.csv
```

## Evaluation signals

- Aggregated HEK293 totals must reproduce 49,993 high-confidence BSJs (readCount >= 2) before database annotation
- BSJs detected in both biological replicates must number 16,934
- Known/novel split for both-replicate BSJs must be 85.1% known / 14.9% novel against circBase and MiOncoCirc
- Known/novel split for one-replicate BSJs must be 46.9% known / 53.1% novel
- No BSJ should appear in both the one-replicate and both-replicate strata (strata partition the filtered set)

## Limitations

- isoCirc is far from saturation at current sequencing depth, so read-count-based filtering thresholds will miss lowly expressed BSJs
- isoCirc applies more stringent filtering to BSJs involving novel splice sites than to known splice sites, so replicate-support statistics differ by BSJ class
- The >= 2 read threshold defines 'high-confidence' here; other confidence definitions will yield different totals and known/novel fractions

## Evidence

- [methods] Known vs novel stratification by replicate support: "Number of candidate BSJs reported in existing circRNA BSJ database (circBase / MiOncoCirc)"
- [discussion] Post-consensus stringency for reproducible BSJ calls: "After calling consensus sequences from isoCirc raw reads, we use stringent criteria to remove potential false-positive detection"
- [readme] Dependency on the isoCirc output pipeline: "isoCirc is a long-read sequencing strategy coupled with an integrated 
computational pipeline to characterize full-length circRNA isoforms"
