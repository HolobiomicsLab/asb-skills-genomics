---
name: drach-motif-site-coverage-filtering
description: Use when read-level RedNano feature files (chrom, pos, alignstrand, loc_in_ref,
  readname, ...) must be converted into site-level candidate m6A sites before training
  or running predict.py, and low-coverage sites need to be excluded.
when_to_use_negative:
- Input is already the aggregated site-level file (e.g. test.features.sort.aggre.c20.tsv)
  — re-running aggregation/filtering would be redundant
- The analysis requires per-read (not per-site) predictions, since aggregation collapses
  read-level instances into single sites
- The DRS data were not resquigged and feature-extracted with RedNano's extract_features.py
  (k=5, s=65), since the site-level table schema (column 4 = coverage) would not exist
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: RedNano
  role: Provides scripts/aggre_features_to_site_level.py that aggregates sorted read-level
    features into site-level rows prior to coverage filtering and model training/prediction
  repo: https://github.com/Derryxu/RedNano
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:4df93d243cbd685e22d65bd12de4b7755502b9a2d502ef32cbf69f6fc458b020
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - CUDA
  - Python
  - RedNano
  - h5py
  - jvarkit
  - minimap2
  - ont-tombo
  - samtools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# DRACH-motif site coverage filtering

## Summary

Aggregate RedNano read-level feature rows to candidate m6A sites and retain only sites with sufficient read depth (coverage >= 20 in column 4), optionally restricted to DRACH/RRACH motif contexts, so that downstream site-level m6A probability predictions are statistically supported.

## When to use

Use when read-level RedNano feature files (chrom, pos, alignstrand, loc_in_ref, readname, ...) must be converted into site-level candidate m6A sites before training or running predict.py, and low-coverage sites need to be excluded.

## When NOT to use

- Input is already the aggregated site-level file (e.g. test.features.sort.aggre.c20.tsv) — re-running aggregation/filtering would be redundant
- The analysis requires per-read (not per-site) predictions, since aggregation collapses read-level instances into single sites
- The DRS data were not resquigged and feature-extracted with RedNano's extract_features.py (k=5, s=65), since the site-level table schema (column 4 = coverage) would not exist

## Inputs

- test.features.sort.tsv — sorted read-level RedNano feature table (columns: chrom, pos, alignstrand, loc_in_ref, readname, strand, k_mer, k_signals_rect, qual, mis, ins, dele)
- Labeled modified/unmodified feature rows for training (labels 0/1 appended via awk)

## Outputs

- test.features.sort.aggre.tsv — site-level aggregated feature table from aggre_features_to_site_level.py
- test.features.sort.aggre.c20.tsv — coverage-filtered site table retaining only rows with column-4 coverage >= 20

## How to apply

First sort the extracted per-read feature files by transcript (column 1), position (column 2, numeric), and read name (column 5) so all reads of a site are contiguous. Run scripts/aggre_features_to_site_level.py on the sorted file to collapse read-level rows into site-level rows, then apply the awk coverage filter `awk 'BEGIN{ FS=OFS="\t" } $4>=20'` to produce a file such as test.features.sort.aggre.c20.tsv. The rationale is that RedNano uses a multiple-instance learning approach in which each site aggregates its reads, and sites with fewer than 20 reads lack the evidence needed for reliable site-level probability estimates. When benchmarking on human HEK293T data, sites are evaluated in RRACH and DRACH motif contexts, and the resulting coverage-filtered site table (with 0/1 labels appended for training) feeds train.py or predict.py with --model_type comb_basecall_raw.

## Related tools

- **RedNano** (Provides scripts/aggre_features_to_site_level.py that aggregates sorted read-level features into site-level rows prior to coverage filtering and model training/prediction) — https://github.com/Derryxu/RedNano

## Examples

```
sort -k1,1 -k2,2n -k5,5 -T./ --parallel=30 -S30G test/features/* > test.features.sort.tsv && python RedNano/scripts/aggre_features_to_site_level.py --input test.features.sort.tsv --output test.features.sort.aggre.tsv && awk 'BEGIN{ FS=OFS="\t" } $4>=20 {print $0}' test.features.sort.aggre.tsv > test.features.sort.aggre.c20.tsv
```

## Evaluation signals

- Every retained row in test.features.sort.aggre.c20.tsv has column 4 (coverage) >= 20
- Output has the expected site-level schema where column 4 is the read count used by the awk $4>=20 filter
- Each output site aggregates only reads sharing transcript, position, and read grouping as produced by sort -k1,1 -k2,2n -k5,5
- On the human HEK293T benchmark (PRJEB40872), site-level AUC/AUPR computed on the filtered table should exceed the m6Anet baseline (RedNano AUC 0.8511 vs 0.8230 for RRACH; 0.8513 vs 0.8187 for DRACH)

## Limitations

- Sites below the coverage >= 20 threshold are discarded entirely, so lowly expressed modified sites cannot be detected
- The aggregation/filter step assumes RedNano's 12-column feature format and k-mer length 5 / signal length 65 settings; deviations change the column layout
- The README's tombo resquiggle reference must be a transcript/gene fasta, not a genome file; otherwise feature extraction and hence site aggregation fail

## Evidence

- [methods] awk 'BEGIN{ FS=OFS="\t" } $4>=20 {print $0}' test.features.sort.aggre.tsv > test.features.sort.aggre.c20.tsv: "awk 'BEGIN{ FS=OFS="\t" } $4>=20 {print $0}' test.features.sort.aggre.tsv > test.features.sort.aggre.c20.tsv"
- [readme] python /path/to/RedNano/scripts/aggre_features_to_site_level.py --input test.features.sort.tsv --output test.features.sort.aggre.tsv: "python /path/to/RedNano/scripts/aggre_features_to_site_level.py --input test.features.sort.tsv --output test.features.sort.aggre.tsv"
- [readme] To use the multiple instance learning approach, the read-level features should be aggregated: "To use the multiple instance learning approach, the read-level features should be aggregated as follows:"
- [abstract] RedNano surpasses existing methods by achieving higher area under the ROC curve (AUC) and area under the precision-recall curve (AUPRs) in all three datasets.: "The results demonstrate that RedNano surpasses existing methods by achieving higher area under the ROC curve (AUC) and area under the precision-recall curve (AUPRs) in all three datasets."
