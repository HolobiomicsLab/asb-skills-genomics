---
name: sensitivity-precision-benchmark-computation
description: Use when you need to reproduce or extend a per-TE-family sensitivity
  (TP/(TP+FN)) and precision (TP/(TP+FP)) comparison of TE insertion callers run on
  a simulated long-read BAM with a known truth set.
when_to_use_negative:
- The analysis target is a real (non-simulated) genome where no complete truth set
  of insertions exists, so TP/FP/FN cannot be unambiguously counted
- The goal is runtime or CPU-hour benchmarking rather than accuracy — sTELLeR's speed
  claims (5-48x faster, <2% CPU hours) require a separate timing protocol
- Only one caller is being run without any truth set for verification, so sensitivity/precision
  cannot be computed
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: sTELLeR
  role: TE insertion caller evaluated in the benchmark; run with --sr 4, --style pb,
    -mr 80 to produce INS:ME VCF calls
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: TLDR
  role: Competing TE caller benchmarked against sTELLeR on the simulated dataset
- name: xTEA
  role: Competing TE caller benchmarked against sTELLeR; calls no L1 or HERV on the
    simulated data
- name: python3
  role: Scripting environment for parsing VCF outputs and computing per-family sensitivity/precision
- name: pysam
  role: Python library dependency used by sTELLeR and available for VCF/BAM parsing
    in the benchmark scripts
- name: samtools
  role: Dependency for BAM handling in the sTELLeR workflow
- name: minimap2
  role: Dependency for aligning long reads used to build the simulated BAM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:6a50d6df7fbe2915cb9a3ca1e4f793464bfac77f69cc8bc65a6f5413d39716c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kristinebilgrav/sTELLeR
  tools:
  - TLDR
  - minimap2
  - pysam
  - python3
  - sTELLeR
  - samtools
  - xTEA
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# sensitivity-precision-benchmark-computation

## Summary

Benchmark per-TE-family (Alu, L1, HERV, SVA) sensitivity and precision of long-read insertion callers such as sTELLeR, TLDR, and xTEA by comparing called insertions against a simulated truth set, reproducing Table 2 of the sTELLeR paper.

## When to use

Use when you need to reproduce or extend a per-TE-family sensitivity (TP/(TP+FN)) and precision (TP/(TP+FP)) comparison of TE insertion callers run on a simulated long-read BAM with a known truth set.

## When NOT to use

- The analysis target is a real (non-simulated) genome where no complete truth set of insertions exists, so TP/FP/FN cannot be unambiguously counted
- The goal is runtime or CPU-hour benchmarking rather than accuracy — sTELLeR's speed claims (5-48x faster, <2% CPU hours) require a separate timing protocol
- Only one caller is being run without any truth set for verification, so sensitivity/precision cannot be computed

## Inputs

- Simulated long-read BAM/CRAM file containing Alu, L1, HERV, and SVA insertions (e.g. testdata.bam)
- Reference genome FASTA used for alignment (masked chr22 with removed masked parts, e.g. HG38_chr22.fa)
- TE sequence FASTA (e.g. TEfastasequences.fa or TEsequences_SVA_HERV_ALU_L1.fasta)
- Truth set of correct TE insertion positions (testbamTRUTH.txt)

## Outputs

- Per-caller VCF of insertion calls (INS:ME variants with TE type in INFO/FORMAT fields)
- Per-TE-family contingency counts (TP, FP, FN) for each caller
- Benchmark table of sensitivity and precision per caller per TE family (Table 2 equivalent)

## How to apply

Run each caller on the same simulated BAM using its documented parameters — for sTELLeR use steller.py with --sr 4, --style pb, and -mr 80 against the masked chr22 reference and the TE fasta. Parse each caller's output VCF (variants named INS:ME with TE type in the INFO/FORMAT fields) to extract called insertion events. Compare called insertions to the truth set (e.g. testbamTRUTH.txt) to count true positives, false positives, and false negatives per TE family. Compute per-family sensitivity TP/(TP+FN) and precision TP/(TP+FP); correct application reproduces sTELLeR values of 0.97 sensitivity / 1.0 precision for Alu, 0.96/1.0 for L1, 0.98/1.0 for HERV, and 0.97/1.0 for SVA, and xTEA calling no L1 or HERV, matching Table 2.

## Related tools

- **sTELLeR** (TE insertion caller evaluated in the benchmark; run with --sr 4, --style pb, -mr 80 to produce INS:ME VCF calls) — https://github.com/kristinebilgrav/sTELLeR
- **TLDR** (Competing TE caller benchmarked against sTELLeR on the simulated dataset)
- **xTEA** (Competing TE caller benchmarked against sTELLeR; calls no L1 or HERV on the simulated data)
- **python3** (Scripting environment for parsing VCF outputs and computing per-family sensitivity/precision)
- **pysam** (Python library dependency used by sTELLeR and available for VCF/BAM parsing in the benchmark scripts)
- **samtools** (Dependency for BAM handling in the sTELLeR workflow)
- **minimap2** (Dependency for aligning long reads used to build the simulated BAM)

## Examples

```
python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80
```

## Evaluation signals

- sTELLeR reproduces Table 2 sensitivity/precision of 0.97/1.0 (Alu), 0.96/1.0 (L1), 0.98/1.0 (HERV), and 0.97/1.0 (SVA) on the simulated data
- xTEA calls no L1 or HERV insertions, matching Table 2
- Alu sensitivity is 0.97 for sTELLeR, 0.99 for TLDR, and 0.97 for xTEA; precision is 1.0 for sTELLeR and xTEA and 0.97 for TLDR
- sTELLeR identifies a total of 7 variants on the provided test data when compared against testbamTRUTH.txt

## Limitations

- The benchmark is on simulated data with a constructed truth set; real-genome performance may differ since truth sets cannot be exhaustive
- xTEA fails to call L1 and HERV insertions, so family-level comparison is incomplete for that caller
- sTELLeR call parameters (--sr, --mq, --maxreads) affect sensitivity; supporting reads default of 3 (4 used here) depends on sample coverage and desired sensitivity

## Evidence

- [results] On the simulated dataset, Alu detection sensitivity was 0.97 for sTELLeR, 0.99 for TLDR, and 0.97 for xTEA, while precision was 1 for both sTELLeR and xTEA and 0.97 for TLDR.: [evidence span withheld — anchor status: not_found]
- [abstract] Particularly, sTELLeR exhibit higher precision and sensitivity for calling of Alu elements than similar tools.: "Particularly, sTELLeR exhibit higher precision and sensitivity for calling of Alu elements than similar tools."
- [methods] python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80: "python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80"
- [readme] correct results can be verified by comparing to the provided file testbamTRUTH.txt - a total of 7 variants should be identified.: "correct results can be verified by comparing to the provided file testbamTRUTH.txt - a total of 7 variants should be identified."
- [readme] The variant is named INS:ME and in the info section the TE type along with start and end positions are given.: "The variant is named INS:ME and in the info section the TE type along with start and end positions are given."
