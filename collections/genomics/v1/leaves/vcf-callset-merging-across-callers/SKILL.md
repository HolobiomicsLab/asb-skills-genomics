---
name: vcf-callset-merging-across-callers
description: Use when multiple TE-insertion caller VCFs (e.g. sTELLeR, TLDR from PacBio/ONT
  lrGS and RetroSeq, MELT2 from short-read BAMs) must be merged into a common representation
  to compute caller agreement and overlap statistics.
when_to_use_negative:
- The callsets are already in a common normalized VCF schema — merging directly without
  re-normalization risks duplicate/incorrect overlaps
- Comparing a single caller's output against a reference truth set rather than computing
  inter-caller overlap statistics
- Input BAMs are not the expected sequencing type for the caller (e.g. running sTELLeR
  with --style pb on short-read data instead of PacBio CLR or ONT)
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: sTELLeR
  role: Long-read TE insertion caller producing the haplotype-aware INS:ME VCF that
    anchors the merge
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: TLDR
  role: Independent long-read TE insertion caller whose callset is intersected with
    sTELLeR's
- name: RetroSeq
  role: Short-read TE insertion caller run on the 30x HG004 BAM
- name: MELT2
  role: Short-read TE insertion caller run on the 30x HG004 BAM
- name: python3
  role: Environment used to run sTELLeR and compute callset intersections
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:76ea082c6250374e44480ae914a925cd568cc67fad98306bda2cb329e43b4de6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kristinebilgrav/sTELLeR
  tools:
  - MELT2
  - RetroSeq
  - TLDR
  - python3
  - sTELLeR
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# vcf-callset-merging-across-callers

## Summary

Normalizes and merges transposable element insertion (TEI) callsets from multiple callers — long-read (sTELLeR, TLDR) and short-read (RetroSeq, MELT2) — into a common INS:ME VCF representation so pairwise and four-way overlaps can be tallied, as done to reproduce the HG004 Fig 2b overlap analysis (642 TEs shared by all four callers).

## When to use

Use when multiple TE-insertion caller VCFs (e.g. sTELLeR, TLDR from PacBio/ONT lrGS and RetroSeq, MELT2 from short-read BAMs) must be merged into a common representation to compute caller agreement and overlap statistics.

## When NOT to use

- The callsets are already in a common normalized VCF schema — merging directly without re-normalization risks duplicate/incorrect overlaps
- Comparing a single caller's output against a reference truth set rather than computing inter-caller overlap statistics
- Input BAMs are not the expected sequencing type for the caller (e.g. running sTELLeR with --style pb on short-read data instead of PacBio CLR or ONT)

## Inputs

- sTELLeR VCF of TE insertions from HG004 PacBio BAM (HG38 ref, Alu/L1 TE fasta, --sr 4, -mr 80)
- TLDR VCF of TE insertions from the same HG004 PacBio BAM
- RetroSeq VCF of TE insertions from 30x short-read HG004 BAM
- MELT2 VCF of TE insertions from 30x short-read HG004 BAM
- TE fasta sequences (e.g. TEfastasequences.fa with Alu and L1 subfamilies)

## Outputs

- Normalized VCF callsets in common INS:ME representation (TE type, start/end in INFO)
- Merged multi-caller TEI VCF
- Overlap table reporting four-caller overlap (642), sTELLeR/TLDR agreement (1302), and TEs missed by lrGS callers (1280)

## How to apply

Run each caller first: sTELLeR on the HG004 PacBio BAM with --style pb, --sr 4 and -mr 80 against the HG38 reference plus an Alu/L1 TE fasta, TLDR on the same long-read BAM, and RetroSeq/MELT2 on the 30x short-read BAM realigned with bwa-mem. Then normalize all four callsets to a common VCF representation so they can be merged, using the INS:ME variant naming with TE type and start/end positions in the INFO field — sTELLeR already emits this convention ('The variant is named INS:ME and in the info section the TE type along with start and end positions are given'). Compute pairwise and four-way intersections, tallying (a) TEs shared by all four callers, (b) TEI agreed by sTELLeR and TLDR, and (c) TEs found only by short-read callers, and write an overlap table. Validation against the published HG004 statistics (642 four-caller TEs, 1302 sTELLeR/TLDR agreements, 1280 TEs missed by lrGS callers) confirms correct normalization and merging.

## Related tools

- **sTELLeR** (Long-read TE insertion caller producing the haplotype-aware INS:ME VCF that anchors the merge) — https://github.com/kristinebilgrav/sTELLeR
- **TLDR** (Independent long-read TE insertion caller whose callset is intersected with sTELLeR's)
- **RetroSeq** (Short-read TE insertion caller run on the 30x HG004 BAM)
- **MELT2** (Short-read TE insertion caller run on the 30x HG004 BAM)
- **python3** (Environment used to run sTELLeR and compute callset intersections)

## Examples

```
python steller/steller.py --ref HG38.fa --TE_fasta fasta/TEfastasequences.fa --bam HG004.pb.bam --sr 4 --style pb -o HG004_steller -mr 80
```

## Evaluation signals

- Four-way intersection of sTELLeR, TLDR, RetroSeq and MELT2 callsets in HG004 reproduces 642 TEs
- sTELLeR and TLDR agree on 1302 TE insertions
- 1280 TEs are undetected by the lrGS callers (short-read-only calls)
- Merged VCF retains INS:ME variant naming with TE type and start/end positions in the INFO field
- All four callsets were normalized to the same VCF representation before merging (no position/format mismatch artifacts)

## Limitations

- Overlaps depend on caller-specific parameters; the HG004 figures were obtained with sTELLeR --sr 4 and -mr 80 and may shift with different thresholds
- Normalization must be applied before merging because sTELLeR, TLDR, RetroSeq and MELT2 otherwise emit differing VCF representations
- sTELLeR's default --sr is 3 and --maxreads 100; deviating from the paper's settings (4 and 80) changes callset sensitivity and thus the overlap statistics

## Evidence

- [readme] The output is given in a VCF file, which is compatible to merge with VCF files from other variant callers.: "The output is given in a [VCF file](https://samtools.github.io/hts-specs/VCFv4.2.pdf), which is compatible to merge with VCF files from other variant callers."
- [readme] The variant is named INS:ME and in the info section the TE type along with start and end positions are given.: "The variant is named INS:ME and in the info section the TE type along with start and end positions are given."
- [methods] sTELLeR example run parameters with pb style, sr 4 and maxreads 80: "python steller/steller.py --ref < genome ref_file > --TE_fasta < TE sequence fasta file > --bam < bamfile > --sr 4 --style pb -o < output prefix > -mr 80"
- [readme] Supporting reads depend on coverage and desired sensitivity (default=3): "Supporting reads (--sr) of a insertion depends on the coverage of the sample and the desired sensitivity (dafault=3)."
- [abstract] sTELLeR is haplotype aware and outputs results in VCF format: "The caller is haplotype aware and output results in a variant call format (VCF) file, enabling compatibility with other variant callers and downstream analysis."
