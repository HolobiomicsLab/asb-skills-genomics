---
name: match-accuracy-metric-computation
description: Use when you need to quantify and compare barcode-matching accuracy of
  scTagger against FLAMES and an exact dynamic-programming baseline on datasets where
  match vs ambiguous vs mismatch vs skip fractions are computed per method.
when_to_use_negative:
- You need barcode assignments for downstream cell-level analysis on real data rather
  than a controlled accuracy comparison — this skill is for evaluating matching methods
  against a baseline on simulated datasets.
- Your barcode-calling step must be judged by its own criteria instead of a fixed
  classification (match/ambiguous/mismatch/skip) — adapt the category definitions
  first.
- The task involves matching UMI sequences between SR and LR data, which scTagger
  does not yet support (future work per the article).
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scTagger
  role: Produces the barcode-match assignments being evaluated; run as extract_lr_bc,
    extract_sr_bc, and match_trie (default e=2) per dataset.
  repo: https://github.com/vpc-ccg/scTagger/
- name: edlib
  role: Brute-force exact edit-distance matching baseline against which scTagger's
    match rate is compared (v1.3.9).
  repo: https://github.com/Martinsos/edlib
- name: FLAMES
  role: 'State-of-the-art comparator whose barcode assignments are classified and
    benchmarked (commit #18fb83c).'
  repo: https://github.com/LuyiTian/FLAMES
- name: 10x Genomics Cell Ranger
  role: Supplies the short-read BAM (mapping, filtering, barcode and UMI counting)
    from which extract_sr_bc selects the top SR barcodes.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.104530
    title: Fast and accurate matching of cellular barcodes across short-reads and
      long-reads of single-cell RNA-seq experiments
  source_content_hash: sha256:a6adf50ff4d589fa9354bca6a4ca2961310b0cf4e9c644501c5f9d65e3c48844
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/vpc-ccg/scTagger
  repo_url: https://github.com/vpc-ccg/scTagger
  tools:
  - 10x Genomics Cell Ranger
  - FLAMES
  - edlib
  - scTagger
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# match-accuracy-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify per-long-read barcode assignments from scTagger, a brute-force edlib baseline, and FLAMES into match/ambiguous/mismatch/skip categories and compute the percentage of each category, reproducing the accuracy table (Table 3) on the simulated LR-SR datasets.

## When to use

Use when you need to quantify and compare barcode-matching accuracy of scTagger against FLAMES and an exact dynamic-programming baseline on datasets where match vs ambiguous vs mismatch vs skip fractions are computed per method.

## When NOT to use

- You need barcode assignments for downstream cell-level analysis on real data rather than a controlled accuracy comparison — this skill is for evaluating matching methods against a baseline on simulated datasets.
- Your barcode-calling step must be judged by its own criteria instead of a fixed classification (match/ambiguous/mismatch/skip) — adapt the category definitions first.
- The task involves matching UMI sequences between SR and LR data, which scTagger does not yet support (future work per the article).

## Inputs

- Simulated LR-SR datasets (Small, Medium, Large) from Figshare DOI 10.6084/m9.figshare.19740475.v1 (long-read FASTQs and Cell Ranger BAM)
- scTagger extract_lr_bc output TSV (read-id, best adapter edit distance, adapter alignment start, LR segment)
- scTagger extract_sr_bc output TSV (barcode, count) of top selected short-read barcodes
- Match outputs from scTagger match_trie, edlib (v1.3.9) brute-force matching, and FLAMES (commit #18fb83c)

## Outputs

- Per-read classification (match / ambiguous / mismatch / skip) for each method
- CSV table of match, ambiguous, mismatch, and skip percentages per method (scTagger, brute-force, FLAMES) per simulated dataset, comparable to Table 3

## How to apply

Run the three-stage scTagger pipeline on each simulated dataset: extract_lr_bc on the long-read FASTQs to pull the 20bp LR segment after the short-read adapter alignment, extract_sr_bc on the Cell Ranger BAM to select the top short-read barcodes (stopping when a 1,000-barcode increment adds less than t=0.5% of total SRs), then match_trie with the default maximum edit distance e=2 and multiple threads. Run a brute-force exact edit-distance matching of the same LR segments against the selected SR barcodes using edlib (v1.3.9) as the ground-truth-quality baseline, and run FLAMES (commit #18fb83c) on the same data. For each method, classify every long read as match (a unique barcode at minimum edit distance), ambiguous (multiple barcodes tie at the minimum edit distance), mismatch, or skip, and compute each category's percentage. Compile percentages per method per dataset (Small, Medium, Large) into a single CSV and validate against the article's Table 3, where scTagger (e.g., Large: 53.26% matches) outperforms FLAMES (30.05%) and approaches the brute-force baseline.

## Related tools

- **scTagger** (Produces the barcode-match assignments being evaluated; run as extract_lr_bc, extract_sr_bc, and match_trie (default e=2) per dataset.) — https://github.com/vpc-ccg/scTagger/
- **edlib** (Brute-force exact edit-distance matching baseline against which scTagger's match rate is compared (v1.3.9).) — https://github.com/Martinsos/edlib
- **FLAMES** (State-of-the-art comparator whose barcode assignments are classified and benchmarked (commit #18fb83c).) — https://github.com/LuyiTian/FLAMES
- **10x Genomics Cell Ranger** (Supplies the short-read BAM (mapping, filtering, barcode and UMI counting) from which extract_sr_bc selects the top SR barcodes.)

## Examples

```
./scTagger.py match_trie -lr sim_large_lr_segments.tsv -sr sim_large_top_sr_barcodes.tsv -o sim_large_matches.tsv -t 8
```

## Evaluation signals

- For each method and dataset, match + ambiguous + mismatch + skip percentages sum to 100% of long reads.
- Reproduced percentages agree with the article's Table 3 values for the simulated datasets (e.g., Large: scTagger 53.26% matches vs FLAMES 30.05%).
- scTagger's match rate at the default e=2 is very similar to the brute-force edlib baseline, since the vast majority of LRs with >2 edits have no unique barcode match.
- Ambiguous calls correspond exactly to long reads with multiple barcodes tied at the minimum edit distance, not merely low edit distance.
- Counts of classified reads equal the number of long reads with an extracted LR segment (extract_lr_bc output rows).

## Limitations

- Accuracy conclusions depend on the simulated data generation (minnow/Badread simulators); behavior on real datasets may differ.
- scTagger is limited to e=2 by default, so reads with >2 edits are inherently unmatched (the article notes the vast majority of such LRs have no unique match anyway).
- The evaluation pipeline is 10x Genomics-specific; scTagger in its current form is specific for 10x scRNA-seq.

## Evidence
