---
name: soft-clipped-read-extraction
description: Use when paired-end short reads aligned to a reference (e.g. ASM294v2.22)
  need split-read evidence recovered at SV breakpoints, and the primary aligner soft-clips
  breakpoint-spanning reads instead of aligning them contiguously.
when_to_use_negative:
- Long-read data are available and direct split-read or read-pair evidence can be
  obtained without clip-and-realign rescue
- Only read-pair (discordant-pair) SV evidence is targeted and soft-clip signal is
  not needed
- Input alignments were produced by an aligner that already reports split/chimeric
  alignments (e.g. no soft-clipped breakpoint reads to rescue)
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: NextGenMap
  role: Primary aligner producing the BAM from which soft-clipped reads (≥20 bp clipped
    segments) are extracted
- name: YAHA
  role: Re-aligns the extracted clipped reads to recover split-read signal across
    SV breakpoints
- name: Picard-tools
  role: Merges YAHA split-read alignments with the primary NextGenMap alignments
- name: Samtools
  role: Down-samples each merged BAM to a uniform 40x mean coverage
- name: SURVIVOR
  role: Downstream consensus merging of SV caller outputs on the merged BAMs; also
    provides 'filter' and 'stats' utilities for VCF-based SV comparison
  repo: https://github.com/fritzsedlazeck/SURVIVOR
- name: Delly
  role: One of the SV inference packages applied to the 40x-normalized merged BAMs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:3fb60cfa4fd6bd1bc0adf1124c8a77f2f87ab1028d8b0b968575496203c87b46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - Delly
  - NextGenMap
  - Picard-tools
  - SURVIVOR
  - Samtools
  - YAHA
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Soft-clipped read extraction and split-read re-alignment for SV breakpoint recovery

## Summary

Extracts reads with soft-clipped segments (≥20 bp) from NextGenMap alignments of Illumina paired-end S. pombe reads and re-aligns them with YAHA to recover split-read signal across structural-variant breakpoints, merging results back into a 40x-normalized BAM.

## When to use

Use when paired-end short reads aligned to a reference (e.g. ASM294v2.22) need split-read evidence recovered at SV breakpoints, and the primary aligner soft-clips breakpoint-spanning reads instead of aligning them contiguously.

## When NOT to use

- Long-read data are available and direct split-read or read-pair evidence can be obtained without clip-and-realign rescue
- Only read-pair (discordant-pair) SV evidence is targeted and soft-clip signal is not needed
- Input alignments were produced by an aligner that already reports split/chimeric alignments (e.g. no soft-clipped breakpoint reads to rescue)

## Inputs

- Illumina paired-end FASTQ reads per S. pombe strain (ENA projects PRJEB2733 and PRJEB6284)
- NextGenMap BAM alignments against the ASM294v2.22 S. pombe reference

## Outputs

- Set of extracted reads with soft-clipped segments ≥20 bp
- YAHA split-read alignments across SV breakpoints
- Merged BAM (NextGenMap primary alignments + YAHA split-read alignments) per strain
- Coverage-normalized (40x mean) merged BAM per strain ready for SV calling

## How to apply

Align Illumina paired-end reads from each strain to the ASM294v2.22 S. pombe reference with NextGenMap. Scan the resulting BAM for reads carrying soft-clipped segments of at least 20 bp and extract those reads, since the primary aligner left breakpoint-spanning portions unaligned. Re-align the extracted clipped reads with YAHA to recover split-read alignments across SV breakpoints. Merge the YAHA split-read alignments with the primary NextGenMap alignments using Picard-tools, then down-sample each merged BAM to a uniform 40x mean coverage with Samtools so SV callers are not biased by coverage differences among the 161 strains. This merged BAM feeds downstream SV calling with Delly, Lumpy, Pindel and cn.MOPs, whose consensus (SVs detected by at least two callers via SURVIVOR) yielded 315 initial variant calls.

## Related tools

- **NextGenMap** (Primary aligner producing the BAM from which soft-clipped reads (≥20 bp clipped segments) are extracted)
- **YAHA** (Re-aligns the extracted clipped reads to recover split-read signal across SV breakpoints)
- **Picard-tools** (Merges YAHA split-read alignments with the primary NextGenMap alignments)
- **Samtools** (Down-samples each merged BAM to a uniform 40x mean coverage)
- **SURVIVOR** (Downstream consensus merging of SV caller outputs on the merged BAMs; also provides 'filter' and 'stats' utilities for VCF-based SV comparison) — https://github.com/fritzsedlazeck/SURVIVOR
- **Delly** (One of the SV inference packages applied to the 40x-normalized merged BAMs)

## Examples

```
samtools view -h ngm.bam | awk '$6 ~ /S/ {print}' > clipped_reads.sam; YAHA align -query clipped_reads.fa -target ref.fa; picard MergeSamFiles; samtools view -s 0.4 -b merged.bam > merged_40x.bam
```

## Evaluation signals

- Each of the 161 strains yields a valid merged BAM at the target 40x mean coverage after Samtools down-sampling
- Extracted read set contains only reads with soft-clipped segments of at least 20 bp in the NextGenMap alignment
- YAHA split-read alignments map to expected SV breakpoint regions and are successfully merged with the primary alignments via Picard-tools
- Downstream SV calling on merged BAMs produces calls amenable to the two-caller consensus rule (315 calls retained before further filtering)
- Coverage histograms per strain are comparable (no strain dominates due to unequal library depth) before consensus SV merging

## Limitations

- The study prioritized high specificity (correctness of the inferred SV) rather than high sensitivity, so not all SVs were detected
- Translocations and inversions were more challenging to accurately genotype from this data, so CNVs alone were re-examined during genotyping
- SV breakpoints overlapping low-complexity regions or annotated long terminal repeats were removed, limiting recovery of SVs in those regions

## Evidence

- [other] Extract reads carrying soft-clipped segments of at least 20 bp from the NextGenMap alignments.: [evidence span withheld — anchor status: not_found]
- [other] Re-align the extracted clipped reads with YAHA to recover split-read signal across SV breakpoints.: [evidence span withheld — anchor status: not_found]
- [other] Merge the YAHA split-read alignments with the primary NextGenMap alignments using Picard-tools.: [evidence span withheld — anchor status: not_found]
- [discussion] We focused on high specificity (the correctness of the inferred SV) rather than high sensitivity (attempting to detect all SVs): "We focused on high specificity (the correctness of the inferred SV) rather than high sensitivity (attempting to detect all SVs)"
- [readme] SURVIVOR is a tool set for simulating/evaluating SVs, merging and comparing SVs within and among samples: "SURVIVOR is a tool set for simulating/evaluating SVs, merging and comparing SVs within and among samples"
