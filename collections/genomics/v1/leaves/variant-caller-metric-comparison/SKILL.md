---
name: variant-caller-metric-comparison
description: Use when comparing the sensitivity/specificity of TR or variant callers
  on simulated long-read BAMs with a known truth set, or when reproducing per-class
  precision/recall/F1 benchmark tables at defined discrepancy thresholds.
when_to_use_negative:
- Input is real (non-simulated) human whole-genome sequencing data without a truth
  set — validation then requires orthogonal Illumina FM indexes rather than per-class
  recall/precision/F1 against simulated truth.
- The genome contains unassembled regions such as human centromeres and telomeres,
  which TRiCoLOR cannot identify repeats in.
- The aim is TR profiling in polyploid species or haploid chromosomes (e.g., the human
  Y chromosome), which TRiCoLOR does not support.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3574
tools:
- name: TRiCoLOR
  role: Primary TR caller under benchmark; run as SENSoR (Shannon-entropy repetitive-region
    detection) followed by REFER (SPOA consensus + RegEx motif resolution) on simulated
    ONT and PB BAMs
  repo: https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- name: TRiCoLOR SENSoR
  role: Detects candidate low-entropy repetitive regions per read using non-overlapping
    20 bp sliding windows with entropy drop <= 1.23, outputting candidate regions
    in BED format
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR REFER
  role: Fetches and trims spanning reads, builds SPOA haplotype-specific consensus
    sequences, realigns with minimap2, and resolves TR motifs/multiplicity into BCF
    calls
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: SPOA
  role: Partial order alignment used inside REFER to compute haplotype-specific low-error
    consensus sequences (~77% ONT and ~88% PB error reduction)
- name: minimap2
  role: Realigns consensus sequences for TR detection; favored over alternative aligners
    for speed and mapping accuracy on synthetic data
- name: NCRF
  role: Alignment-based comparator long-read TR caller run on the same simulated BAMs
    to obtain benchmark recall/precision/F1
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:09b31011ea5a5d4ea786ea95007e6b65eef84d0d567399fc6d6a65458b6a9547
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NCRF
  - SPOA
  - TRiCoLOR
  - TRiCoLOR REFER
  - TRiCoLOR SENSoR
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3574
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# variant-caller-metric-comparison

## Summary

Benchmark two long-read tandem repeat callers (TRiCoLOR and NCRF) on VISOR-simulated haplotype-resolved ONT and PacBio BAMs by computing per-class recall, precision, and F1 against a simulated truth set at 1-motif and 2-motif discrepancy thresholds. This reproduces the TRiCoLOR paper's head-to-head comparison showing TRiCoLOR's superior recall on contractions, small expansions, and long expansions.

## When to use

Use when comparing the sensitivity/specificity of TR or variant callers on simulated long-read BAMs with a known truth set, or when reproducing per-class precision/recall/F1 benchmark tables at defined discrepancy thresholds.

## When NOT to use

- Input is real (non-simulated) human whole-genome sequencing data without a truth set — validation then requires orthogonal Illumina FM indexes rather than per-class recall/precision/F1 against simulated truth.
- The genome contains unassembled regions such as human centromeres and telomeres, which TRiCoLOR cannot identify repeats in.
- The aim is TR profiling in polyploid species or haploid chromosomes (e.g., the human Y chromosome), which TRiCoLOR does not support.

## Inputs

- VISOR-simulated haplotype-resolved ONT long-read BAM files with embedded TR contractions, small expansions (~7 motifs), and long expansions (+200 motifs)
- VISOR-simulated haplotype-resolved PacBio long-read BAM files with the same embedded TR variant classes
- Simulated truth set of TR variant calls with known motifs and multiplicities
- TRiCoLOR SENSoR candidate repetitive-region BED files
- TRiCoLOR REFER TR calls in BCF format and NCRF comparator TR calls

## Outputs

- benchmark_results.csv containing per-variant-class recall, precision, and F1 for TRiCoLOR and NCRF at 1-motif and 2-motif discrepancy thresholds
- Per-class recall values: TRiCoLOR 0.97/0.86/0.73 and NCRF 0.87/0.74/0.53 for contractions/small expansions/long expansions
- Motif correctness verification for each simulated TR

## How to apply

Generate VISOR-simulated haplotype-resolved ONT and PacBio BAMs (sequencing accuracy ~0.85–0.95, haplotype coverage 5–20x) embedding TR contractions, small expansions (~7 motifs), and exceptionally long expansions (+200 motifs). Run TRiCoLOR SENSoR with default Shannon-entropy settings (non-overlapping 20 bp sliding windows, entropy drop <= 1.23, >= 5 supporting reads, merging windows within 100 bp) to produce candidate repetitive regions in BED format, then TRiCoLOR REFER to build SPOA haplotype-specific consensus sequences (error reduction ~77% ONT, ~88% PB), realign with minimap2, and call TRs (motifs <= 6 bp, >= 5 perfect repeats, edit distance <= 1, TR length >= 50 bp) into BCF format. Run NCRF on the identical simulated BAMs as the comparator. Evaluate both tools against the simulated truth set, computing recall, precision, and F1 per variant class (contractions, small expansions, long expansions) at 1-motif and 2-motif discrepancy thresholds and verifying motif correctness. Aggregate per-class metrics — expected TRiCoLOR recall 0.97 (contractions), 0.86 (expansions), 0.73 (long expansions) vs NCRF 0.87/0.74/0.53, with F1 close to 1 at 1-motif discrepancy and P ~ 1 and R ~ 1 at 2-motif discrepancy — and write the comparison table to benchmark_results.csv.

## Related tools

- **TRiCoLOR** (Primary TR caller under benchmark; run as SENSoR (Shannon-entropy repetitive-region detection) followed by REFER (SPOA consensus + RegEx motif resolution) on simulated ONT and PB BAMs) — https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- **TRiCoLOR SENSoR** (Detects candidate low-entropy repetitive regions per read using non-overlapping 20 bp sliding windows with entropy drop <= 1.23, outputting candidate regions in BED format) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR REFER** (Fetches and trims spanning reads, builds SPOA haplotype-specific consensus sequences, realigns with minimap2, and resolves TR motifs/multiplicity into BCF calls) — https://github.com/davidebolo1993/TRiCoLOR
- **SPOA** (Partial order alignment used inside REFER to compute haplotype-specific low-error consensus sequences (~77% ONT and ~88% PB error reduction))
- **minimap2** (Realigns consensus sequences for TR detection; favored over alternative aligners for speed and mapping accuracy on synthetic data)
- **NCRF** (Alignment-based comparator long-read TR caller run on the same simulated BAMs to obtain benchmark recall/precision/F1)

## Examples

```
docker run -v $(pwd):/data davidebolo1993/tricolor TRiCoLOR -f reference.fa -b simulated_ont.bam --ngs /data --regions 1; NCRF reference.fa simulated_ont.bam > ncrf_calls.txt
```

## Evaluation signals

- TRiCoLOR per-class recall matches reported values within small numerical tolerance: R = 0.97 (contractions), 0.86 (expansions), 0.73 (long expansions).
- NCRF per-class recall matches reported values: R = 0.87 (contractions), 0.74 (expansions), 0.53 (long expansions).
- F1 is close to 1 for both callers at 1-motif discrepancy; precision ~ 1 and recall ~ 1 at 2-motif discrepancy.
- Every simulated TR has its motif correctly identified by the caller.
- TRiCoLOR recall exceeds NCRF recall in every variant class (contractions, small expansions, long expansions), consistent with the paper's finding of improved sensitivity.

## Limitations

- TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres).
- No support for polyploid species or haploid chromosomes (e.g., the human Y chromosome); extending support is listed as future work.
- Parameter values for profiling mini-satellites are not validated — the tool has been extensively applied only to micro-satellites.
- Recall on exceptionally long expansions (+200 motifs) is markedly lower (~0.73) than on contractions and small expansions, for both callers.

## Evidence

- [abstract] TRiCoLOR demonstrates excellent performance and improved sensitivity and specificity compared with alternative tools on synthetic data.: "TRiCoLOR demonstrates excellent performance and improved sensitivity and specificity compared with alternative tools on synthetic data."
- [methods] This is achieved using the SENSoR module, which uses the Shannon entropy of DNA sequences to identify candidate repetitive segments: "This is achieved using the SENSoR module, which uses the Shannon entropy of DNA sequences to identify candidate repetitive segments in genomic sequences"
- [methods] The default entropy treshold of 1.23 efficiently discriminates between repetitive and non-repetitive DNA sequences using syntethic ONT and PB reads: "The default entropy treshold of 1.23 efficiently discriminates between repetitive and non-repetitive DNA sequences using syntethic ONT and PB reads"
- [methods] to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [discussion] TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres): "TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres)"
- [readme] TRiCoLOR is available to be installed from source or as a Docker container.: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
