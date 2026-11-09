---
name: multi-caller-intersection-statistics
description: Use when comparing TE insertion callsets from multiple callers (e.g.
  long-read sTELLeR/TLDR vs short-read RetroSeq/MELT2) and you need overlap counts
  such as N-way intersections and caller-agreement tallies.
when_to_use_negative:
- You only have a single caller's VCF and no second callset to intersect against
- Callsets are in incompatible or non-normalized representations (e.g. non-INS:ME
  naming without TE type and start/end positions in INFO) and cannot first be converted
  to a common VCF form
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: sTELLeR
  role: Long-read TE insertion caller producing an INS:ME VCF mergeable with other
    callers' VCFs
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: TLDR
  role: Independent long-read TE insertion caller on the same PacBio BAM
- name: RetroSeq
  role: Short-read TE insertion caller on the 30x HG004 BAM
- name: MELT2
  role: Short-read TE insertion caller on the 30x HG004 BAM
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:2490349dafea3d6be8da26d428c57dc4b748597413184c556c03e5764f6707ee
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# multi-caller-intersection-statistics

## Summary

Compute pairwise and multi-way (up to four-way) overlaps between transposable element insertion callsets from long-read callers (sTELLeR, TLDR) and short-read callers (RetroSeq, MELT2) on HG004, after normalizing all VCFs to the common INS:ME representation, to quantify caller concordance.

## When to use

Use when comparing TE insertion callsets from multiple callers (e.g. long-read sTELLeR/TLDR vs short-read RetroSeq/MELT2) and you need overlap counts such as N-way intersections and caller-agreement tallies.

## When NOT to use

- You only have a single caller's VCF and no second callset to intersect against
- Callsets are in incompatible or non-normalized representations (e.g. non-INS:ME naming without TE type and start/end positions in INFO) and cannot first be converted to a common VCF form

## Inputs

- sTELLeR VCF of TE insertions from the HG004 PacBio BAM (HG38 reference, Alu/L1 TE fasta, --style pb, --sr 4, -mr 80)
- TLDR VCF of TE insertions from the same HG004 PacBio BAM
- RetroSeq VCF of TE insertions from the 30x short-read HG004 BAM (bwa-mem realigned)
- MELT2 VCF of TE insertions from the 30x short-read HG004 BAM

## Outputs

- Overlap table with pairwise and four-way intersection statistics
- Counts: TEs shared by all four callers (642 in HG004), TE insertions agreed by sTELLeR and TLDR (1302), and TEs undetected by long-read callers (1280)

## How to apply

Run each caller independently on the appropriate input data: sTELLeR (python steller/steller.py with --style pb, --sr 4, -mr 80) and TLDR on the HG004 PacBio BAM, and RetroSeq and MELT2 on the 30x short-read HG004 BAM realigned with bwa-mem. Normalize all four callsets to a common VCF representation so they can be merged, using the INS:ME variant naming with TE type and start/end positions in the INFO field — sTELLeR already emits this format natively, which 'is compatible to merge with VCF files from other variant callers.' Compute pairwise and four-way intersections of the merged callsets, tallying (a) TEs shared by sTELLeR, TLDR, RetroSeq, and MELT2, (b) TE insertions agreed upon by the long-read callers sTELLeR and TLDR, and (c) TEs found only by the short-read callers. Write the intersection statistics to an overlap table.

## Related tools

- **sTELLeR** (Long-read TE insertion caller producing an INS:ME VCF mergeable with other callers' VCFs) — https://github.com/kristinebilgrav/sTELLeR
- **TLDR** (Independent long-read TE insertion caller on the same PacBio BAM)
- **RetroSeq** (Short-read TE insertion caller on the 30x HG004 BAM)
- **MELT2** (Short-read TE insertion caller on the 30x HG004 BAM)

## Examples

```
python steller/steller.py --ref HG38.fa --TE_fasta TEfastasequences.fa --bam HG004_pacbio.bam --sr 4 --style pb -o HG004_steller -mr 80
```

## Evaluation signals

- Four-way overlap of sTELLeR, TLDR, RetroSeq, and MELT2 callsets in HG004 equals 642 TEs
- sTELLeR and TLDR agree on 1302 TE insertions
- 1280 TEs are undetected by the lrGS callers (found only by short-read callers)
- All four callsets are expressed in the common INS:ME VCF representation with TE type and start/end positions in the INFO field before merging
- For sTELLeR itself, the standalone run is correct if the test data yields a total of 7 variants verified against testbamTRUTH.txt

## Limitations

- Statistics are sample-specific (HG004) and depend on the exact caller parameters (--sr 4, -mr 80, --style pb); changing support-read or coverage thresholds will shift the intersection counts
- sTELLeR output haplotag is 0 and genotype reflects zygosity only if the input BAM is not phased
- Merging requires all callers' VCFs to be normalized to the INS:ME convention; callers emitting different representations must be converted first

## Evidence

- [other] In HG004, the overlap of all four callers (sTELLeR, TLDR, RetroSeq, MELT2) was 642 TEs; 1280 TEs were undetected by the lrGS callers, while sTELLeR and TLDR agreed upon 1302 TEI.: [evidence span withheld — anchor status: not_found]
- [other] Normalize all four callsets to a common VCF representation so they can be merged, using the INS:ME variant naming with TE type and start/end positions in the INFO field.: [evidence span withheld — anchor status: not_found]
- [readme] The output is given in a VCF file, which is compatible to merge with VCF files from other variant callers.: "The output is given in a [VCF file](https://samtools.github.io/hts-specs/VCFv4.2.pdf), which is compatible to merge with VCF files from other variant callers."
- [readme] The variant is named INS:ME and in the info section the TE type along with start and end positions are given.: "The variant is named INS:ME and in the info section the TE type along with start and end positions are given."
- [methods] python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80: "python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80"
