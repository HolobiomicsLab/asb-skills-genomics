---
name: per-organism-likelihood-ratio-source-assignment
description: Use when dZ-positive reads in a mixed metagenome must be traced to their
  organism of origin, i.e. a per-read Z-Calling SVM TSV exists and reads are aligned
  to a multi-organism reference and you need per-species LR+ evidence of dZ-DNA presence.
when_to_use_negative:
- The goal is single-nucleotide Z/A site calling or Z-frequency profiling (z-bam2txt/zfreq)
  rather than assigning reads to source organisms
- Reads cannot be mapped to a multi-organism reference with high fidelity (dv:f >=
  0.05), so the LR+ denominator would be unreliable
- The sample consists of fully substituted dZ-genome only (no mixed metagenome), where
  the simpler read classifier alone suffices
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0637
tools:
- name: Z-Calling
  role: Provides the SVM read classifier (k21ReadClassifier) that labels reads '+'
    (dZ) or '-' (canonical) via ZP tags from the k21-full-ZA base-calling model
  repo: https://github.com/xiaochuanle/Z-Calling
- name: minimap2
  role: Maps filtered reads (-x map-hifi preset) to the multi-organism reference;
    primary pairs with dv:f < 0.05 are kept for LR+ analysis
- name: samtools
  role: Converts the filtered BAM to FASTA (samtools fasta -@8) as minimap2 input
- name: pbmm2
  role: Aligns filtered CCS reads to the reference (--preset CCS --sort) when the
    input BAM was unmapped
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:ad3e1df2ace2d0e33118327eafa36edfdade06f0e9b6457552f2f306ed963984
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xiaochuanle/Z-Calling
  repo_url: https://github.com/xiaochuanle/Z-Calling
  tools:
  - Z-Calling
  - minimap2
  - pbmm2
  - samtools
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# per-organism-likelihood-ratio-source-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A downstream Z-Calling pipeline that identifies the taxonomic sources of dZ-DNA reads in mixed metagenomes by joining per-read SVM '+'/'-' classifications with primary read-to-reference-contig alignments and computing per-organism positive likelihood ratios (LR+). It enables dZ-positive organisms to be pinpointed while the SVM read classifier enforces zero false positive discovery.

## When to use

Use when dZ-positive reads in a mixed metagenome must be traced to their organism of origin, i.e. a per-read Z-Calling SVM TSV exists and reads are aligned to a multi-organism reference and you need per-species LR+ evidence of dZ-DNA presence.

## When NOT to use

- The goal is single-nucleotide Z/A site calling or Z-frequency profiling (z-bam2txt/zfreq) rather than assigning reads to source organisms
- Reads cannot be mapped to a multi-organism reference with high fidelity (dv:f >= 0.05), so the LR+ denominator would be unreliable
- The sample consists of fully substituted dZ-genome only (no mixed metagenome), where the simpler read classifier alone suffices

## Inputs

- Z-Calling per-read classification TSV from z-calling-read predict (columns: read name; '+' dZ-DNA / '-' canonical)
- Filtered BAM from filter_bam.py (kinetic tags fi, ri, fp, rp present; fn >= 3 and rn >= 3; Median_PW/Median_IPD >= 0.3 and <= 99th percentile)
- read_RefContig.tsv from minimap2 (-x map-hifi) restricted to primary pairs with dv:f < 0.05
- Contig-to-species table (ref_species TSV) mapping multi-organism reference contigs to organism names

## Outputs

- SVM.LR.tsv: per-organism positive likelihood ratios (LR+) quantifying dZ-DNA evidence for each source organism

## How to apply

First produce the per-read classification TSV by running the Z-Calling base module with the k21-full-ZA model (-k 21) to add ZP tags, then build/z-calling-read predict with model/svm/k21ReadClassifier using min_len 500 and intervals exactly 100; reads are labeled '+' (dZ-DNA) or '-' (canonical). Second, convert the filtered BAM to FASTA (samtools fasta -@8) and map with minimap2 -t 8 -x map-hifi, retaining only primary read-to-contig pairs with alignment divergence dv:f < 0.05 into read_RefContig.tsv. Finally run py/SVM_LR_analysis.py with --SVM (the classification TSV), --read_ref (read_RefContig.tsv) and --ref_species (contig-to-species table) to compute per-organism LR+ in SVM.LR.tsv. The rationale is that only high-fidelity primary alignments (dv:f < 0.05) should anchor the LR+ computation, since the SVM is deliberately tuned for high confidence to avoid false positive organism calls.

## Related tools

- **Z-Calling** (Provides the SVM read classifier (k21ReadClassifier) that labels reads '+' (dZ) or '-' (canonical) via ZP tags from the k21-full-ZA base-calling model) — https://github.com/xiaochuanle/Z-Calling
- **minimap2** (Maps filtered reads (-x map-hifi preset) to the multi-organism reference; primary pairs with dv:f < 0.05 are kept for LR+ analysis)
- **samtools** (Converts the filtered BAM to FASTA (samtools fasta -@8) as minimap2 input)
- **pbmm2** (Aligns filtered CCS reads to the reference (--preset CCS --sort) when the input BAM was unmapped)

## Examples

```
python py/SVM_LR_analysis.py --SVM z-calling-read-Result.tsv --read_ref read_RefContig.tsv --ref_species ref_species.tsv --output SVM.LR.tsv
```

## Evaluation signals

- All metagenomes containing at least 25 yeast hybrid reads must be called dZ-positive; metagenome 7 (only 10 yeast hybrid reads) is the single expected false negative
- Zero false positive organism discovery should be reproduced, consistent with the benchmarking claim in the README
- Only primary read-to-contig pairs with dv:f < 0.05 should enter read_RefContig.tsv
- Output TSV schema check: read classification file has exactly two columns (read name; '+', '-', or unclassified), and SVM.LR.tsv reports one LR+ per organism
- Unclassified entries in the classification TSV should correspond to reads failing the min_len 500 length criterion

## Limitations

- The SVM read classifier is optimized for fully substituted dZ-genomes and can miss hybrid reads with diluted kinetic signatures; metagenome 7 with only 10 yeast hybrid reads was the sole false negative
- Subtle Z-induced kinetic shifts (0.5-1.5-fold) overlap with canonical Adenine variance, limiting quantification of hybrid samples
- The SVM enforces high confidence to avoid false positives in organism identification, trading off sensitivity for very low-dZ-content organisms

## Evidence

- [results] Validation threshold from the 12-metagenome benchmark: [evidence span withheld — anchor status: not_found]
