---
name: vcf-comparison-across-callers
description: Use when benchmarking SNV variant callers (e.g. miniSNV vs Clair3 vs
  Longshot) on ONT Dorado sup-basecalled data by scoring each caller's VCF against
  GIAB truth VCFs within GIAB stratification regions to obtain F1, recall, and HDR.
when_to_use_negative:
- Input data is not ONT long-read nanopore data (e.g. short-read or PacBio HiFi) —
  the benchmark signal and caller selection (miniSNV, Clair3, Longshot with ONT models)
  are specific to ONT R10.4.1 kit v14 sup-basecalled reads
- You need validated production variant calls rather than a comparison/benchmark —
  rerio research models 'have not been scrutinized nor validated' by ONT
- Truth-set stratification files are unavailable — the reported F1/recall and HDR
  comparisons are computed only within GIAB v3.3 stratification regions and are not
  reproducible against whole-genome comparisons
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: miniSNV
  role: Statistical SNV caller under test; leverages read pileup, read-based phasing,
    and consensus generation with a 1000 Genomes known-common-variants index
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Clair3
  role: Deep-learning comparison caller (v1.0.4) run with the latest ONT-trained model
    from the rerio clair3_models repository
  repo: https://github.com/nanoporetech/rerio
- name: Longshot
  role: Diploid SNV comparison caller (v0.4.5) using a pair hidden Markov model, run
    on the same sorted BAMs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:a6bc5e9cec3a851310746a63741f8a06daa639dad58c398bf927203dca0a62fc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CuiMiao-HIT/miniSNV
  repo_url: https://github.com/CuiMiao-HIT/miniSNV
  tools:
  - Clair3
  - Longshot
  - miniSNV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# vcf-comparison-across-callers

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark multiple SNV callers on ONT long-read data by comparing each caller's VCF against GIAB truth VCFs within GIAB v3.3 stratification regions, computing precision, recall, F1, and homozygous discovery rate (HDR) per caller per sample. This is the evaluation framework used to show miniSNV exceeding 99.7% F1/recall against Clair3 and Longshot.

## When to use

Use when benchmarking SNV variant callers (e.g. miniSNV vs Clair3 vs Longshot) on ONT Dorado sup-basecalled data by scoring each caller's VCF against GIAB truth VCFs within GIAB stratification regions to obtain F1, recall, and HDR.

## When NOT to use

- Input data is not ONT long-read nanopore data (e.g. short-read or PacBio HiFi) — the benchmark signal and caller selection (miniSNV, Clair3, Longshot with ONT models) are specific to ONT R10.4.1 kit v14 sup-basecalled reads
- You need validated production variant calls rather than a comparison/benchmark — rerio research models 'have not been scrutinized nor validated' by ONT
- Truth-set stratification files are unavailable — the reported F1/recall and HDR comparisons are computed only within GIAB v3.3 stratification regions and are not reproducible against whole-genome comparisons

## Inputs

- Sorted, samtools-indexed BAM of ONT R10.4.1 kit v14 Dorado sup-basecalled reads per sample (HG002/HG003/HG004)
- GRCh38 (GCA_000001405.15) samtools-indexed reference FASTA
- GIAB truth VCFs for HG002, HG003, HG004
- GIAB v3.3 stratification BED files (difficult-to-map and low-complexity regions)
- miniSNV prebuilt 1000 Genomes known-common-variants index (five files)
- Clair3 ONT-trained model from nanoporetech/rerio clair3_models

## Outputs

- Per-caller SNV VCF files (miniSNV, Clair3, Longshot) per sample
- Per-caller x per-sample precision, recall, and F1 table (benchmark_table_sup.csv)
- Per-caller homozygous discovery rate (HDR) values for HG003

## How to apply

Map ONT reads (kit v14, R10.4.1, Dorado sup-basecalled GIAB Ashkenazi Trio samples HG002/HG003/HG004) to GRCh38 (GCA_000001405.15) and produce sorted, indexed BAMs. Run each caller per sample: miniSNV with its prebuilt 1000 Genomes known-common-variants index (window length L=300 bp, half-length overlaps, >5% allele frequency), Clair3 v1.0.4 with the ONT-trained model from nanoporetech/rerio clair3_models, and Longshot v0.4.5 for diploid SNV calls. Compare each caller's VCF against the GIAB truth VCFs restricted to GIAB v3.3 stratification regions (excluding difficult-to-map and low-complexity regions) to compute precision, recall, and F1 per caller per sample. Compute the homozygous discovery rate (HDR) per caller for the paternal sample HG003, where miniSNV surpasses Clair3 by ~1.8% and Longshot by ~1.6%. Assemble per-caller x per-sample metrics into a benchmark table (benchmark_table_sup.csv) and validate that F1 and recall exceed 99.7% on all three sup-dataset samples.

## Related tools

- **miniSNV** (Statistical SNV caller under test; leverages read pileup, read-based phasing, and consensus generation with a 1000 Genomes known-common-variants index) — https://github.com/CuiMiao-HIT/miniSNV
- **Clair3** (Deep-learning comparison caller (v1.0.4) run with the latest ONT-trained model from the rerio clair3_models repository) — https://github.com/nanoporetech/rerio
- **Longshot** (Diploid SNV comparison caller (v0.4.5) using a pair hidden Markov model, run on the same sorted BAMs)

## Examples

```
python chr_chunk_task.py --fin_ref GRCh38.fa --fin_bam HG003.sorted.bam --workDir work_HG003 --useindex --human --sample HG003
```

## Evaluation signals

- miniSNV F1-score and recall both exceed 99.7% on HG002, HG003, and HG004 sup-datasets against GIAB truth
- Reproduced HDR advantage on HG003 matches reported values: miniSNV surpasses Clair3 by ~1.8% and Longshot by ~1.6%
- All precision/recall/F1 computations are restricted to GIAB v3.3 stratification regions (difficult-to-map and low-complexity regions excluded)
- Each caller's VCF is generated with the paper's versions: Clair3 1.0.4 with the rerio ONT model, Longshot 0.4.5, miniSNV with L=300 bp windows, half-length overlaps, and >5% allele-frequency known-variant background

## Limitations

- HDR advantage was reported specifically for the paternal sample HG003 on the sup-datasets; generalization to other samples or chemistries is not established here
- Clair3's ONT-trained research models from rerio are unvalidated by ONT: 'The results generated by research basecalling models have not been scrutinized nor validated'
- All models in rerio's clair3_models are compatible with Clair3 v1.x but not v2.x, so caller version must be pinned

## Evidence

- [readme] Research model disclaimer for rerio callers: [evidence span withheld — anchor status: external_doc]
