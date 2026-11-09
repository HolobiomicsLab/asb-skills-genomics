---
name: giab-truth-set-benchmark-evaluation
description: Use when benchmarking SNV callers on ONT nanopore reads against GIAB
  truth sets — e.g., reproducing miniSNV vs Clair3/Longshot precision, recall, F1,
  and HDR comparisons on HG002/HG003/HG004 sup-datasets.
when_to_use_negative:
- The data are simulated ONT-like reads from non-human species (Mouse, Drosophila,
  Arabidopsis) or bacterial GC-content genomes with no GIAB truth VCF available —
  this benchmark depends on GIAB trio truth sets and stratifications
- The variant class of interest is indels or structural variants — miniSNV and this
  protocol are specific to SNV calling
- You already have per-caller metrics in benchmark_table_sup.csv and only need to
  interpret or compare HDR/F1 values rather than regenerate them
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_0091
tools:
- name: miniSNV
  role: Primary SNV caller under benchmark; run per sorted BAM with the 1000 Genomes
    index to produce SNV calls and HDR for HG003
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Clair3
  role: Comparison DL-based caller (v1.0.4) run with the ONT-trained model from rerio
    clair3_models to produce per-sample SNV calls
  repo: https://github.com/nanoporetech/rerio/tree/master/clair3_models
- name: Longshot
  role: Diploid SNV caller (v0.4.5) used as the second statistical comparison baseline
    per BAM
- name: Dorado
  role: Sup-basecaller producing the kit v14 ONT reads that underlie the benchmark
    BAMs
  repo: https://github.com/nanoporetech/dorado
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:95abb4c4f45bc300b3a6b617681afac5e7e091dac7bb8b49a690c1574dc413d6
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
  - Dorado
  - Longshot
  - miniSNV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# giab-truth-set-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking protocol that evaluates SNV variant callers (miniSNV, Clair3 1.0.4, Longshot 0.4.5) on GIAB Ashkenazi trio ONT data (kit v14, R10.4.1, Dorado sup-basecalled) by comparing caller VCFs against GIAB truth VCFs within GIAB v3.3 stratification regions, reporting precision, recall, F1, and homozygous discovery rate (HDR).

## When to use

Use when benchmarking SNV callers on ONT nanopore reads against GIAB truth sets — e.g., reproducing miniSNV vs Clair3/Longshot precision, recall, F1, and HDR comparisons on HG002/HG003/HG004 sup-datasets.

## When NOT to use

- The data are simulated ONT-like reads from non-human species (Mouse, Drosophila, Arabidopsis) or bacterial GC-content genomes with no GIAB truth VCF available — this benchmark depends on GIAB trio truth sets and stratifications
- The variant class of interest is indels or structural variants — miniSNV and this protocol are specific to SNV calling
- You already have per-caller metrics in benchmark_table_sup.csv and only need to interpret or compare HDR/F1 values rather than regenerate them

## Inputs

- samtools-indexed sorted BAM of ONT reads mapped to GRCh38 (GCA_000001405.15) per sample (HG002/HG003/HG004, kit v14 Dorado sup)
- samtools-indexed GRCh38 FASTA reference
- GIAB truth VCFs for HG002, HG003, HG004
- GIAB v3.3 stratification BED files (difficult-to-map and low-complexity regions)
- miniSNV prebuilt 1000 Genomes index folder (five files, used with --useindex)
- Clair3 ONT-trained model from nanoporetech/rerio clair3_models (e.g., r1041_e82_400bps_sup variant)

## Outputs

- Per-caller, per-sample SNV VCFs (miniSNV, Clair3 1.0.4, Longshot 0.4.5)
- benchmark_table_sup.csv with precision, recall, and F1 per caller per sample
- Homozygous discovery rate (HDR) per caller for HG003

## How to apply

Download the GIAB AshkenazimTrio ONT deposits (kit v14 chemistry, R10.4.1 flowcells, Dorado sup-basecalled, ~200 Gbases per sample) and map reads to GRCh38 (GCA_000001405.15) into samtools-indexed sorted BAMs. Run miniSNV on each BAM with its prebuilt 1000 Genomes known-common-variants index (window length L=300 bp with half-length overlaps, >5% allele frequency), run Clair3 1.0.4 with the ONT-trained model from nanoporetech/rerio clair3_models matching the sup basecaller, and run Longshot 0.4.5 for diploid SNV calls. Compare each caller's VCF against the GIAB truth VCFs within GIAB v3.3 stratification regions (accounting for difficult-to-map and low-complexity regions) and compute precision, recall, and F1 per caller per sample. Finally compute the homozygous discovery rate (HDR) for the paternal sample HG003, where the reference result shows miniSNV surpasses Clair3 by ~1.8% and Longshot by ~1.6%, and assemble metrics into benchmark_table_sup.csv.

## Related tools

- **miniSNV** (Primary SNV caller under benchmark; run per sorted BAM with the 1000 Genomes index to produce SNV calls and HDR for HG003) — https://github.com/CuiMiao-HIT/miniSNV
- **Clair3** (Comparison DL-based caller (v1.0.4) run with the ONT-trained model from rerio clair3_models to produce per-sample SNV calls) — https://github.com/nanoporetech/rerio/tree/master/clair3_models
- **Longshot** (Diploid SNV caller (v0.4.5) used as the second statistical comparison baseline per BAM)
- **Dorado** (Sup-basecaller producing the kit v14 ONT reads that underlie the benchmark BAMs) — https://github.com/nanoporetech/dorado

## Examples

```
python chr_chunk_task.py --fin_ref GRCh38.fa --fin_bam HG003.sorted.bam --workDir work_HG003 --useindex -i index/ --human --sample HG003
```

## Evaluation signals

- miniSNV's F1-score and recall each exceed 99.7% on HG002, HG003, and HG004 sup-datasets with precision close behind
- Reproduced HG003 HDR shows miniSNV ahead of Clair3 by ~1.8% and Longshot by ~1.6% on the sup-datasets
- Precision/recall/F1 are computed only within GIAB v3.3 stratification regions, with difficult-to-map and low-complexity regions handled per stratification annotations
- Clair3 was run with the latest ONT-trained model from the rerio clair3_models repository, matched to the R10.4.1 sup basecalling chemistry
- All per-caller x per-sample metrics assemble cleanly into benchmark_table_sup.csv without missing samples

## Limitations

- miniSNV is SNV-specific — the benchmark does not cover indel or SV calling
- HDR advantage was reported for the paternal sample HG003 on sup-datasets; do not generalize the 1.8%/1.6% margins to other samples or basecalling modes without recomputation
- rerio research models (Clair3 and Dorado configs) carry ONT's disclaimer that results have not been scrutinized nor validated by Oxford Nanopore
- No discussion/changelog section text was available for the article, limiting documented caveats to results and methods sections

## Evidence

- [results] sup-dataset F1/recall gate: "miniSNV simultaneously achieved the highest F1-score and recall, all of which were greater than 99.7%, and closer precision on HG002, HG003, and HG004 sup-datasets"
- [results] benchmark dataset provenance: [evidence span withheld — anchor status: not_found]
- [results] stratification regions used: "difficult-to-map regions and low-complexity regions of the genome which defined by GIAB v3.3 stratifications"
