---
name: bam-merge-with-picard
description: Use when you have primary short-read alignments from NextGenMap plus
  re-aligned soft-clipped reads from YAHA and must combine them into one BAM before
  SV calling or coverage normalization.
when_to_use_negative:
- The YAHA split-read alignments are already contained in the NextGenMap primary BAM
  (no separate merge step is needed).
- The goal is merging SV call sets (VCF consensus) rather than read alignments — that
  is SURVIVOR merge territory, not Picard BAM merging.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Picard-tools
  role: Merges the YAHA split-read alignments with the primary NextGenMap alignments
    into a single BAM per strain
- name: NextGenMap
  role: Produces the primary paired-end alignments and the soft-clipped reads (>=
    20 bp clip) that YAHA re-aligns
- name: YAHA
  role: Re-aligns extracted clipped reads to recover split-read signal across SV breakpoints
    prior to merging
- name: Samtools
  role: Down-samples each merged BAM to a uniform 40x mean coverage after Picard merging
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:a833667ebd95ecedab7e0c30e83cada0f13371509366c747827fa6152597c3d2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Delly
  - NextGenMap
  - Picard-tools
  - Samtools
  - YAHA
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bam-merge-with-picard

## Summary

Merges YAHA split-read (clipped-read) alignments with primary NextGenMap alignments into a single merged BAM per S. pombe strain using Picard-tools, so downstream SV callers (Delly, Lumpy, Pindel, cn.MOPs) see both discordant-pair and split-read evidence in one file.

## When to use

Use when you have primary short-read alignments from NextGenMap plus re-aligned soft-clipped reads from YAHA and must combine them into one BAM before SV calling or coverage normalization.

## When NOT to use

- The YAHA split-read alignments are already contained in the NextGenMap primary BAM (no separate merge step is needed).
- The goal is merging SV call sets (VCF consensus) rather than read alignments — that is SURVIVOR merge territory, not Picard BAM merging.

## Inputs

- Primary BAM from NextGenMap alignment of Illumina paired-end reads to the ASM294v2.22 S. pombe reference
- YAHA BAM/SAM containing re-aligned reads with soft-clipped segments of >= 20 bp

## Outputs

- One merged BAM per strain combining primary NextGenMap alignments and YAHA split-read alignments
- Merged BAMs subsequently down-sampled to 40x mean coverage with Samtools

## How to apply

First align Illumina paired-end reads of each strain to the ASM294v2.22 S. pombe reference with NextGenMap, then extract reads carrying soft-clipped segments of at least 20 bp and re-align those clipped reads with YAHA to recover split-read signal across SV breakpoints. Merge the YAHA supplementary alignments with the primary NextGenMap alignments using Picard-tools (MergeSamFiles-style BAM merging), keeping one merged BAM per strain. After merging, down-sample each BAM to a uniform 40x mean coverage with Samtools so SV inference is not biased by coverage differences among the 161 strains. Verify every strain yields a valid, complete merged BAM at the target 40x coverage before proceeding to the four-caller SV prediction stage.

## Related tools

- **Picard-tools** (Merges the YAHA split-read alignments with the primary NextGenMap alignments into a single BAM per strain)
- **NextGenMap** (Produces the primary paired-end alignments and the soft-clipped reads (>= 20 bp clip) that YAHA re-aligns)
- **YAHA** (Re-aligns extracted clipped reads to recover split-read signal across SV breakpoints prior to merging)
- **Samtools** (Down-samples each merged BAM to a uniform 40x mean coverage after Picard merging)

## Examples

```
java -jar picard.jar MergeSamFiles I=ngm_primary.bam I=yaha_splits.bam O=merged.bam ASSUME_SORTED=true VALIDATION_STRINGENCY=LENIENT && samtools view -s 0.4 -b merged.bam > merged_40x.bam
```

## Evaluation signals

- Each of the 161 strains yields a valid merged BAM that loads and passes Picard/Samtools validation without errors.
- Post-merge, down-sampling with Samtools brings each BAM to a uniform 40x mean coverage.
- Read counts in the merged BAM equal the sum of primary (NextGenMap) plus clipped-read (YAHA) alignments, with no duplicated read records at SV breakpoints.
- Merged BAMs support SV calling that feeds the two-caller consensus filter, which yields 315 variant calls (141 deletions, 112 duplications, 26 inversions, 36 translocations).

## Limitations

- The study prioritized high specificity (correctness of inferred SVs) over high sensitivity, so merging for maximal call sensitivity was not the design goal.
- Translocations and inversions were more challenging to accurately genotype even after merging, and were excluded from re-examination.
- Strain coverage differences must be normalized (40x down-sampling) after merging, otherwise callers are biased by uneven input coverage.

## Evidence

- [methods] Merge the YAHA split-read alignments with the primary NextGenMap alignments using Picard-tools.: [evidence span withheld — anchor status: not_found]
- [methods] Extract reads carrying soft-clipped segments of at least 20 bp from the NextGenMap alignments.: [evidence span withheld — anchor status: not_found]
- [abstract] To predict an initial set of SVs, we applied four inference software packages (Delly, Lumpy, Pindel and cn.MOPs) to existing short-read data, using parameters optimized on simulated data: [evidence span withheld — anchor status: not_found]
- [abstract] We then filtered these initial predictions, accepting SVs detected by at least two callers, to obtain 315 variant calls: "We then filtered these initial predictions, accepting SVs detected by at least two callers, to obtain 315 variant calls"
