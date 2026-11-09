---
name: coverage-downsampling-to-target-depth
description: Use when a collection of short-read BAM alignments has heterogeneous
  sequencing depth (e.g. 161 S. pombe strains re-sequenced to different depths) and
  SV calling or copy-number inference requires uniform coverage before variant discovery.
when_to_use_negative:
- A strain's merged BAM is already below the target coverage (e.g. under 40x) — downsampling
  cannot raise depth, so the sample must be left as-is or excluded
- The downstream analysis depends on absolute read depth, such as re-examining read
  coverage of the 49 CNVs to genotype copy number within clonal populations — downsampling
  would distort the coverage signal used for copy-number estimation
- Input is unaligned FASTQ or a raw base-level coverage table rather than a merged
  alignment BAM
tools:
- name: Samtools
  role: Performs the downsampling of each merged BAM to a uniform 40x mean coverage
    (samtools view -s downsampling fraction)
- name: Picard-tools
  role: Upstream step producing the merged BAM (merges YAHA split-read alignments
    with primary NextGenMap alignments) that is then downsampled
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:f175d86e0aa037035e228e9e28a7d77b9db9159b4d96b5b4651f1ba08b41f37b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Delly
  - NextGenMap
  - Picard-tools
  - Samtools
  - YAHA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# coverage-downsampling-to-target-depth

## Summary

Randomly downsamples each strain's merged BAM alignments to a uniform mean coverage (40x in the Jeffares et al. 2017 S. pombe SV study) using Samtools, so that SV callers (Delly, Lumpy, Pindel, cn.MOPs) see comparable depth across all samples and depth differences between strains do not bias SV calling.

## When to use

Use when a collection of short-read BAM alignments has heterogeneous sequencing depth (e.g. 161 S. pombe strains re-sequenced to different depths) and SV calling or copy-number inference requires uniform coverage before variant discovery.

## When NOT to use

- A strain's merged BAM is already below the target coverage (e.g. under 40x) — downsampling cannot raise depth, so the sample must be left as-is or excluded
- The downstream analysis depends on absolute read depth, such as re-examining read coverage of the 49 CNVs to genotype copy number within clonal populations — downsampling would distort the coverage signal used for copy-number estimation
- Input is unaligned FASTQ or a raw base-level coverage table rather than a merged alignment BAM

## Inputs

- Merged BAM per strain containing primary NextGenMap paired-end alignments plus YAHA split-read alignments (merged with Picard-tools)
- Reference genome ASM294v2.22 (used to compute mean coverage)
- Target mean coverage value (40x)

## Outputs

- Downsampled BAM per strain at approximately uniform 40x mean coverage
- Validation record that each strain's downsampled BAM is valid and at target coverage

## How to apply

After building the per-strain merged BAM (NextGenMap alignments of paired-end reads to ASM294v2.22, supplemented with YAHA realignment of reads carrying soft-clipped segments of at least 20 bp, merged with Picard-tools), compute the mean coverage of each BAM and choose a sampling fraction so the mean depth equals 40x. Run Samtools view with the -s downsampling fraction on the merged BAM to produce the downsampled file, one per strain. Verify that each of the 161 strains yields a valid merged BAM at the target 40x mean coverage before proceeding to SV prediction. The rationale is to harmonize depth across strains so the four inference packages are run on comparable data; the study explicitly favored high specificity (correctness of inferred SVs) over sensitivity, and uniform 40x depth supports that goal.

## Related tools

- **Samtools** (Performs the downsampling of each merged BAM to a uniform 40x mean coverage (samtools view -s downsampling fraction))
- **Picard-tools** (Upstream step producing the merged BAM (merges YAHA split-read alignments with primary NextGenMap alignments) that is then downsampled)

## Examples

```
samtools view -s <fraction> -b -o strain_40x.bam merged.bam  # fraction chosen so mean depth ≈ 40x
```

## Evaluation signals

- Mean coverage of each downsampled BAM, computed against the ASM294v2.22 reference, is approximately 40x
- All 161 S. pombe strains yield a valid, non-empty merged BAM at the target 40x coverage with no truncation errors
- Downsampled BAMs still retain the split-read signal: YAHA-aligned clipped reads (soft-clipped segments of at least 20 bp) are present after merging and downsampling
- Downsampled depth variance across strains is substantially reduced relative to pre-downsampling depth variance
- Downstream SV calling (Delly, Lumpy, Pindel, cn.MOPs) on the uniform-depth BAMs yields candidate calls of the expected scale (315 variant calls after the two-caller consensus filter in the reference study)

## Limitations

- Downsampling discards reads, so sensitivity is reduced; the study prioritized high specificity (correctness of the inferred SV) rather than high sensitivity, so not all SVs were detected
- Uniform mean coverage does not guarantee uniform local coverage — uneven regions can still cause depth artifacts affecting SV calls
- Coverage-based copy-number genotyping on downsampled data is unreliable; the authors re-examined read coverage of CNVs in the original data, excluding translocations and inversions because they were hard to genotype

## Evidence

- [methods] Down-sample each merged BAM to a uniform 40x mean coverage with Samtools.: [evidence span withheld — anchor status: not_found]
- [methods] Verify that each strain yields a valid merged BAM at the target 40x coverage.: [evidence span withheld — anchor status: not_found]
- [methods] Extract reads carrying soft-clipped segments of at least 20 bp from the NextGenMap alignments.: [evidence span withheld — anchor status: not_found]
- [abstract] we applied four inference software packages (Delly, Lumpy, Pindel and cn.MOPs) to existing short-read data: "we applied four inference software packages (Delly, Lumpy, Pindel and cn.MOPs)24252627 to existing short-read data17"
- [discussion] We focused on high specificity (the correctness of the inferred SV) rather than high sensitivity (attempting to detect all SVs): "We focused on high specificity (the correctness of the inferred SV) rather than high sensitivity (attempting to detect all SVs)"
