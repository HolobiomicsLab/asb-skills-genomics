---
name: dz-dna-read-classification
description: Use when you have PacBio CCS BAMs with kinetic tags (fi, ri, fp, rp)
  and need per-read '+'/'-' calls distinguishing dZ-DNA from canonical DNA, e.g. in
  metagenomes or mixed dZ/canonical samples.
when_to_use_negative:
- Site-specific single-nucleotide Z/A discrimination is the goal — use the k11-mixed-AZ
  model (-k 11) base-calling module instead, which records per-base Z probability
- Quantifying Z fractions at individual loci in hybrid (partially substituted) samples
  — the SVM is optimized for fully substituted dZ-genomes and misses hybrid reads
  with diluted kinetic signatures
- Input reads lack PacBio kinetic signals (e.g. ONT data or CCS generated without
  --keep-kinetics)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3168
tools:
- name: Z-Calling
  role: Machine-learning toolkit providing filter_bam.py, z-calling-base (k21 MLP
    with ZP tags), and z-calling-read predict (k21ReadClassifier SVM) for per-read
    dZ-DNA classification
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PacBio Circular Consensus Sequencing (CCS)
  role: Read source; CCS BAMs must be generated with --keep-kinetics to preserve PW
    and IPD signals consumed by the classifier
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:61c63b07f36083c19d569d17982bc3844e27e35b46c63eb5af623b9f978a9e2d
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
  - PacBio Circular Consensus Sequencing (CCS)
  - Z-Calling
  - minimap2
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# dz-dna-read-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies individual PacBio HiFi (CCS) reads as dZ-DNA-positive ('+') or canonical ('-') using Z-Calling's k21 MLP base-caller followed by the k21ReadClassifier SVM, enabling detection of dZ-DNA molecules in mixed samples.

## When to use

Use when you have PacBio CCS BAMs with kinetic tags (fi, ri, fp, rp) and need per-read '+'/'-' calls distinguishing dZ-DNA from canonical DNA, e.g. in metagenomes or mixed dZ/canonical samples.

## When NOT to use

- Site-specific single-nucleotide Z/A discrimination is the goal — use the k11-mixed-AZ model (-k 11) base-calling module instead, which records per-base Z probability
- Quantifying Z fractions at individual loci in hybrid (partially substituted) samples — the SVM is optimized for fully substituted dZ-genomes and misses hybrid reads with diluted kinetic signatures
- Input reads lack PacBio kinetic signals (e.g. ONT data or CCS generated without --keep-kinetics)

## Inputs

- PacBio CCS BAM with kinetic tags (fi, ri, fp, rp, fn, rn, Median_PW, Median_IPD)
- Z-Calling pre-trained model model/k21-full-ZA/scripted_m21.pth (use with -k 21)
- Z-Calling pre-trained SVM model model/svm/k21ReadClassifier

## Outputs

- BAM annotated with ZP tags from z-calling-base
- Two-column TSV: read name and classification '+' (dZ-DNA) / '-' (canonical) / unclassified

## How to apply

First filter the raw BAM with py/filter_bam.py, requiring kinetic tags (fi, ri, fp, rp), discarding reads with Median_PW/Median_IPD < 0.3 or above the 99th percentile, and requiring fn >= 3 and rn >= 3 passes — this preserves the PW kinetics that Z incorporation perturbs. Optionally align the filtered reads with pbmm2 (--preset CCS --sort). Then run build/z-calling-base with the k21-full-ZA model (-k 21) to annotate reads with ZP probability tags (stored in the output BAM), followed by build/z-calling-read predict with model/svm/k21ReadClassifier, min_len 500 (hard requirement) and intervals 100 (exact requirement), producing a two-column TSV of read name and '+' (dZ-DNA), '-' (canonical), or unclassified. Z incorporation primarily shifts pulse width (PW) in a sequence-dependent manner, which is why the MLP+SVM stack trained on full-dA/dZ datasets works at the read level. The tool detects dZ-DNA down to ~1% dZ content with zero false positives reported in benchmarking.

## Related tools

- **Z-Calling** (Machine-learning toolkit providing filter_bam.py, z-calling-base (k21 MLP with ZP tags), and z-calling-read predict (k21ReadClassifier SVM) for per-read dZ-DNA classification) — https://github.com/xiaochuanle/Z-Calling
- **PacBio Circular Consensus Sequencing (CCS)** (Read source; CCS BAMs must be generated with --keep-kinetics to preserve PW and IPD signals consumed by the classifier)

## Examples

```
build/z-calling-base -k 21 -t 16 Filtered.bam model/k21-full-ZA/scripted_m21.pth ZP.bam && build/z-calling-read predict ZP.bam read_calls.tsv model/svm/k21ReadClassifier 500 100
```

## Evaluation signals

- Output TSV contains exactly two columns: read name and a '+'/'-' classification (any other value means the read failed the min_len criterion)
- All reads in the TSV are >= 500 bp (min_len requirement) and features were extracted with intervals=100
- Upstream filter_bam.py output retains only reads with fi/ri/fp/rp tags, Median_PW/Median_IPD in [0.3, 99th percentile], and fn >= 3, rn >= 3
- In artificial metagenome mixtures, all samples with >= 25 dZ-positive yeast hybrid reads call '+', with metagenome 7 (10 yeast reads) as the only acceptable false negative
- No false positives among canonical-DNA controls (e.g. native E. coli example reads), consistent with the reported zero false positive discovery in benchmarking

## Limitations

- The SVM read classifier is optimized for fully substituted dZ-genomes and misses hybrid reads with diluted kinetic signatures (the sole false negative, metagenome 7, contained only 10 yeast hybrid reads)
- The inherent noise of PacBio kinetic measurements means the subtle Z-induced PW shift (0.5–1.5-fold) overlaps substantially with canonical Adenine variance, limiting hybrid sample quantification
- min_len must be >= 500 and intervals must be exactly 100 for the current model; reads failing these criteria are returned unclassified

## Evidence

- [methods] z-calling-read predict with the k21ReadClassifier model using min_len 500 and intervals 100 to produce the per-read '+' (dZ) / '-' (canonical) classification TSV: [evidence span withheld — anchor status: not_found]
