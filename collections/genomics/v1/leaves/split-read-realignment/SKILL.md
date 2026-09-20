---
name: split-read-realignment
description: Use when short-read alignments to a reference genome show soft-clipped
  reads (breakpoint-spanning signal) that must be recovered for SV discovery, e.g.
  before running callers like Delly/Lumpy/Pindel.
when_to_use_negative:
- When reads are long (PacBio/ONT) — SURVIVOR's simreads simulates long reads, but
  this clipping/re-alignment workflow is designed for short Illumina paired-end data
- When the aim is high sensitivity rather than high specificity — the authors note
  this pipeline prioritizes correctness of inferred SVs over detecting all SVs, so
  previously unknown or low-support breakpoints may be missed
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0102
tools:
- name: NextGenMap
  role: Primary paired-end read alignment to the S. pombe reference; its soft-clipped
    reads are extracted as split-read candidates
- name: YAHA
  role: Re-alignment of extracted ≥20 bp soft-clipped reads to recover split-read
    signal across SV breakpoints
- name: Picard-tools
  role: Merging YAHA split-read alignments with primary NextGenMap alignments into
    one BAM
- name: Samtools
  role: Down-sampling merged BAMs to a uniform 40x mean coverage per strain
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:6a5200cc83325a5a919649977724a6c132e36290590b4084699f19bdf93a54f2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Delly
  - NextGenMap
  - Picard-tools
  - Samtools
  - YAHA
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# split-read-realignment

## Summary

Recover split-read signal across structural variant breakpoints by re-aligning soft-clipped reads with a dedicated aligner and merging them back into the primary alignments, producing uniform-coverage BAMs ready for multi-caller SV prediction.

## When to use

Use when short-read alignments to a reference genome show soft-clipped reads (breakpoint-spanning signal) that must be recovered for SV discovery, e.g. before running callers like Delly/Lumpy/Pindel.

## When NOT to use

- When reads are long (PacBio/ONT) — SURVIVOR's simreads simulates long reads, but this clipping/re-alignment workflow is designed for short Illumina paired-end data
- When the aim is high sensitivity rather than high specificity — the authors note this pipeline prioritizes correctness of inferred SVs over detecting all SVs, so previously unknown or low-support breakpoints may be missed

## Inputs

- Illumina paired-end read data (FASTQ) per strain, e.g. from ENA projects PRJEB2733 and PRJEB6284
- Reference genome assembly (S. pombe ASM294v2.22)

## Outputs

- Merged BAM per strain containing primary NextGenMap alignments plus YAHA split-read alignments
- Coverage-normalized BAM per strain down-sampled to 40x mean coverage

## How to apply

Align Illumina paired-end reads to the reference (here ASM294v2.22 for S. pombe) with NextGenMap, then extract reads carrying soft-clipped segments of at least 20 bp from the alignments — these clips indicate reads spanning SV breakpoints that the primary alignment could not place. Re-align the extracted clipped reads with YAHA, a split-read aligner, to anchor both halves of each read across candidate breakpoints. Merge the YAHA split-read alignments with the primary NextGenMap alignments using Picard-tools so callers see a single combined BAM per strain. Down-sample each merged BAM to a uniform 40x mean coverage with Samtools so that coverage-based SV callers (e.g. cn.MOPS) are comparable across strains. Verify each strain yields a valid merged BAM at the target 40x before feeding the set to SV callers.

## Related tools

- **NextGenMap** (Primary paired-end read alignment to the S. pombe reference; its soft-clipped reads are extracted as split-read candidates)
- **YAHA** (Re-alignment of extracted ≥20 bp soft-clipped reads to recover split-read signal across SV breakpoints)
- **Picard-tools** (Merging YAHA split-read alignments with primary NextGenMap alignments into one BAM)
- **Samtools** (Down-sampling merged BAMs to a uniform 40x mean coverage per strain)

## Examples

```
NextGenMap -paired -t 8 -f ASM294v2.22.fa -s reads_R1.fq -s2 reads_R2.fq -o sam && samtools view -Sb merged.sam | samtools downsample - 40x > strain_40x.bam
```

## Evaluation signals

- Each strain produces a valid merged BAM at the target 40x mean coverage after down-sampling
- The fraction of reads with soft-clipped segments ≥20 bp is non-trivial before YAHA re-alignment (clipped reads were successfully extracted)
- Merged BAM is well-formed (valid header, single read group, Picard merge completed without errors)
- Downstream SV callers (Delly, Lumpy, Pindel, cn.MOPS) produce calls on the merged BAMs; requiring agreement of at least two callers yields 315 variant calls in this study

## Limitations

- The pipeline prioritizes high specificity over high sensitivity, so not all SVs present in the data are detected
- Some SV classes (translocations and inversions) remain difficult to genotype accurately even after this alignment stage, and were excluded from genotyping re-examination
- Low complexity regions and annotated LTRs at breakpoints confound downstream filtering, so calls overlapping them are removed after calling

## Evidence

- [other] soft-clip extraction threshold: [evidence span withheld — anchor status: not_found]
- [other] YAHA re-alignment purpose: [evidence span withheld — anchor status: not_found]
- [other] merge and downsample: [evidence span withheld — anchor status: not_found]
- [abstract] downstream consensus filtering: "We then filtered these initial predictions, accepting SVs detected by at least two callers, to obtain 315 variant calls"
- [discussion] specificity over sensitivity: "We focused on high specificity (the correctness of the inferred SV) rather than high sensitivity (attempting to detect all SVs)"
