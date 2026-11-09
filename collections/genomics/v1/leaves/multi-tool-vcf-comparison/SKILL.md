---
name: multi-tool-vcf-comparison
description: Use when comparing TE insertion callers on the same long-read (PacBio/ONT)
  BAMs — e.g. HG002/HG01071 — and you need per-caller true-positive counts scored
  against an assembly-based callset.
when_to_use_negative:
- Input is short-read data — sTELLeR is designed and tested for long-read data (PacBio
  or ONT)
- The comparison target is a reference-presence callset rather than non-reference
  TE insertions — sTELLeR detects non-reference TE insertions
- You only have a single caller's VCF with no truth callset — true-positive scoring
  against the assembly-based callset is not possible
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: sTELLeR
  role: Primary TE insertion caller producing haplotype-aware INS:ME VCF records from
    PacBio BAMs for TP counting
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: pysam
  role: Dependency used to parse BAM/VCF data during calling and VCF-truth-set intersection
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: samtools
  role: Dependency for BAM handling when preparing caller inputs
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: minimap2
  role: Dependency listed for sTELLeR installation (alignment backend)
  repo: https://github.com/kristinebilgrav/sTELLeR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:b0adb62dfebdda571fcd1cbbf6f6746cb247f9cf605ee329bebc348d5a82844f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kristinebilgrav/sTELLeR
  tools:
  - minimap2
  - pysam
  - python3
  - sTELLeR
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# multi-tool-vcf-comparison

## Summary

Benchmark transposable element insertion (TEI) callers (sTELLeR, TLDR, xTEA) by intersecting each caller's INS:ME VCF records against an assembly-based truth callset, producing per-caller true-positive counts for reproducibility (e.g. Table 3).

## When to use

Use when comparing TE insertion callers on the same long-read (PacBio/ONT) BAMs — e.g. HG002/HG01071 — and you need per-caller true-positive counts scored against an assembly-based callset.

## When NOT to use

- Input is short-read data — sTELLeR is designed and tested for long-read data (PacBio or ONT)
- The comparison target is a reference-presence callset rather than non-reference TE insertions — sTELLeR detects non-reference TE insertions
- You only have a single caller's VCF with no truth callset — true-positive scoring against the assembly-based callset is not possible

## Inputs

- GRCh38-realigned PacBio BAM files for HG002 and HG01071
- Reference genome FASTA (e.g. HG38)
- TE sequence FASTA (--TE_fasta, e.g. TEfastasequences.fa with Alu, L1, SVA, HERV elements)
- Assembly-based truth callset of TE insertions
- TEI caller VCF outputs (INS:ME records) from sTELLeR, TLDR, and xTEA

## Outputs

- Per-sample TE insertion VCF per caller (testdata_res_repeats.vcf-style output)
- Per-caller true-positive count table (e.g. caller_tp_counts.tsv with sTELLeR 1885/1914, TLDR 1741/1808, xTEA 1543/1645)

## How to apply

Run sTELLeR (steller.py) on each GRCh38-realigned PacBio BAM with --style pb, --sr 4, --mq 20 (default) and --mr 80, and run TLDR and xTEA on the same BAM files to produce per-sample TE insertion VCFs. Extract the INS:ME records from each VCF — sTELLeR names its variants 'INS:ME' with TE type, transposon start/end in the INFO field. Intersect each caller's records with the assembly-based truth callset to count true positives per caller per sample. Aggregate TP counts into a per-caller table (e.g. sTELLeR 1885/1914, TLDR 1741/1808, xTEA 1543/1645 for HG002/HG01071) and write it as a TSV. Judgement of correctness relies on truth-set comparison, mirroring sTELLeR's own verification workflow where reproduced counts must match reported values within tolerance.

## Related tools

- **sTELLeR** (Primary TE insertion caller producing haplotype-aware INS:ME VCF records from PacBio BAMs for TP counting) — https://github.com/kristinebilgrav/sTELLeR
- **pysam** (Dependency used to parse BAM/VCF data during calling and VCF-truth-set intersection) — https://github.com/kristinebilgrav/sTELLeR
- **samtools** (Dependency for BAM handling when preparing caller inputs) — https://github.com/kristinebilgrav/sTELLeR
- **minimap2** (Dependency listed for sTELLeR installation (alignment backend)) — https://github.com/kristinebilgrav/sTELLeR

## Examples

```
python steller/steller.py --ref HG38.fa --TE_fasta fasta/TEfastasequences.fa --bam HG002.pb.bam --sr 4 --style pb -o HG002_res -mr 80
```

## Evaluation signals

- Reproduced sTELLeR TP count matches the reported 1885 true positives in HG002 within tolerance
- Output TSV contains one TP-count row per caller (sTELLeR, TLDR, xTEA)
- Relative ordering of TP counts reproduces Table 3: sTELLeR > TLDR > xTEA in both HG002 and HG01071
- All VCF records counted are INS:ME variants with TE type and transposon start/end positions in the INFO field
- On the sTELLeR_supplementary test data, a total of 7 variants should be identified when validated against testbamTRUTH.txt

## Limitations

- sTELLeR minimum supporting split reads (--sr) default is 3; the benchmark uses --sr 4 and maxreads --mr 80, so default settings will not reproduce the reported counts
- Intermediate files (ID_chr_reads.fasta, ID_chr_repeats.sam) are removed by default unless -k/--keep_intermediates is set
- sTELLeR output VCF haplotype is 0 and genotype 1/0 or 1/1 if the input BAM is not phased, which may complicate haplotype-aware comparisons

## Evidence

- [results] Against the assembly-based callset, sTELLeR called 1885 true positive TEI in HG002 and 1914 in HG01071, exceeding TLDR (1741/1808) and xTEA (1543/1645).: [evidence span withheld — anchor status: not_found]
- [readme] python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80: "python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80"
- [readme] The variant is named INS:ME and in the info section the TE type along with start and end positions are given.: "The variant is named INS:ME and in the info section the TE type along with start and end positions are given."
- [methods] correct results can be verified by comparing to the provided file testbamTRUTH.txt: "correct results can be verified by comparing to the provided file testbamTRUTH.txt"
- [intro] Output detection results in VCF format with haplotype information if input is phased: [evidence span withheld — anchor status: external_doc]
