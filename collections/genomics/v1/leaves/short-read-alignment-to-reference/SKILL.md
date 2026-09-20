---
name: short-read-alignment-to-reference
description: Use when you need to map Illumina paired-end reads to a reference genome
  and recover split-read signal to support structural variation prediction across
  many strains.
when_to_use_negative:
- If the goal is exhaustive (high-sensitivity) SV discovery — this study deliberately
  prioritized high specificity (correctness of inferred SVs), so not all SVs will
  be detected
- If reads are long-read (PacBio/ONT) data — the pipeline relies on paired-end soft-clipping
  and split-read recovery with NextGenMap/YAHA, not long-read alignment
- If SV calls are already produced and only downstream genotyping/curation is needed
  (e.g. re-examining CNV read coverage in clonal populations)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NextGenMap
  role: Aligns Illumina paired-end reads to the ASM294v2.22 S. pombe reference; its
    soft-clipped alignments feed split-read recovery
- name: YAHA
  role: Re-aligns extracted reads with soft-clipped segments of at least 20 bp to
    recover split-read signal across SV breakpoints
- name: Picard-tools
  role: Merges the YAHA split-read alignments with the primary NextGenMap alignments
    into one BAM per strain
- name: Samtools
  role: Down-samples each merged BAM to a uniform 40x mean coverage across strains
- name: Delly
  role: One of four SV inference packages applied to the short-read data with parameters
    optimized on simulated data
- name: Lumpy
  role: One of four SV inference packages applied to the short-read data with parameters
    optimized on simulated data
- name: Pindel
  role: One of four SV inference packages applied to the short-read data with parameters
    optimized on simulated data
- name: cn.MOPs
  role: One of four SV inference packages (copy-number oriented) applied to the short-read
    data
- name: SURVIVOR
  role: Merges and compares per-caller VCFs to generate the consensus/multi-sample
    SV call set and filter calls
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:558236230cb85a67cd47c5b3b7ba95c4628fae82c9cfd87873fd527215d94886
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - Delly
  - Lumpy
  - NextGenMap
  - Picard-tools
  - Pindel
  - SURVIVOR
  - Samtools
  - YAHA
  - cn.MOPs
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# short-read-alignment-to-reference

## Summary

A read-mapping and split-read recovery pipeline for predicting structural variations from Illumina paired-end short reads of 161 S. pombe strains against the ASM294v2.22 reference, feeding a consensus SV-calling workflow (Delly, Lumpy, Pindel, cn.MOPs merged with SURVIVOR).

## When to use

Use when you need to map Illumina paired-end reads to a reference genome and recover split-read signal to support structural variation prediction across many strains.

## When NOT to use

- If the goal is exhaustive (high-sensitivity) SV discovery — this study deliberately prioritized high specificity (correctness of inferred SVs), so not all SVs will be detected
- If reads are long-read (PacBio/ONT) data — the pipeline relies on paired-end soft-clipping and split-read recovery with NextGenMap/YAHA, not long-read alignment
- If SV calls are already produced and only downstream genotyping/curation is needed (e.g. re-examining CNV read coverage in clonal populations)

## Inputs

- Illumina paired-end short-read data for S. pombe strains from ENA projects PRJEB2733 and PRJEB6284
- S. pombe reference genome assembly ASM294v2.22

## Outputs

- Merged BAM per strain at 40x mean coverage (NextGenMap primary alignments + YAHA split-read alignments)
- Per-caller SV call sets (Delly, Lumpy, Pindel, cn.MOPs) in VCF format
- Consensus SV call set generated with SURVIVOR merge, subsequently filtered to 315 variant calls (141 deletions, 112 duplications, 26 inversions, 36 translocations)

## How to apply

Retrieve the Illumina paired-end read data for each S. pombe strain from the ENA projects PRJEB2733 and PRJEB6284, then align the paired-end reads to the ASM294v2.22 S. pombe reference with NextGenMap. Extract reads carrying soft-clipped segments of at least 20 bp from the NextGenMap alignments and re-align these clipped reads with YAHA to recover split-read signal across SV breakpoints. Merge the YAHA split-read alignments with the primary NextGenMap alignments using Picard-tools, and down-sample each merged BAM to a uniform 40x mean coverage with Samtools so caller input is comparable across strains. On the resulting normalized BAMs, predict an initial set of SVs with four inference packages (Delly, Lumpy, Pindel and cn.MOPs) using parameters optimized on simulated data, then merge caller outputs into a consensus with SURVIVOR's merge command before filtering (accepting only SVs detected by at least two callers yields 315 variant calls). The study prioritized high specificity over sensitivity, so the alignment pipeline is tuned to produce reliable calls rather than exhaustive detection.

## Related tools

- **NextGenMap** (Aligns Illumina paired-end reads to the ASM294v2.22 S. pombe reference; its soft-clipped alignments feed split-read recovery)
- **YAHA** (Re-aligns extracted reads with soft-clipped segments of at least 20 bp to recover split-read signal across SV breakpoints)
- **Picard-tools** (Merges the YAHA split-read alignments with the primary NextGenMap alignments into one BAM per strain)
- **Samtools** (Down-samples each merged BAM to a uniform 40x mean coverage across strains)
- **Delly** (One of four SV inference packages applied to the short-read data with parameters optimized on simulated data)
- **Lumpy** (One of four SV inference packages applied to the short-read data with parameters optimized on simulated data)
- **Pindel** (One of four SV inference packages applied to the short-read data with parameters optimized on simulated data)
- **cn.MOPs** (One of four SV inference packages (copy-number oriented) applied to the short-read data)
- **SURVIVOR** (Merges and compares per-caller VCFs to generate the consensus/multi-sample SV call set and filter calls) — https://github.com/fritzsedlazeck/SURVIVOR

## Examples

```
./SURVIVOR merge caller1.vcf caller2.vcf caller3.vcf caller4.vcf consensus.vcf
```

## Evaluation signals

- Each of the 161 strains yields a valid merged BAM at the target 40x mean coverage after Samtools down-sampling
- YAHA re-alignment recovers split-read alignments for all reads with soft-clipped segments of at least 20 bp, and Picard merge produces a single combined BAM per strain
- The consensus SV set from SURVIVOR contains only SVs detected by at least two callers; applying this filter should reproduce the paper's 315 variant calls (141 deletions, 112 duplications, 26 inversions, 36 translocations)
- Caller parameters were optimized on simulated data before applying to real reads, so run SURVIVOR eval against simulated SVs to verify caller settings
- A multi-sample VCF is produced by SURVIVOR merge, allowing per-strain SV genotyping in downstream curation

## Limitations

- The study focused on high specificity (correctness of the inferred SV) rather than high sensitivity, so not all SVs were detected by this pipeline
- Translocations and inversions were more challenging to accurately genotype, so they were excluded from re-examination steps
- SURVIVOR is a tool set for simulating/evaluating SVs and merging/comparing VCFs — users should consult the GitHub wiki for per-command parameters and contact fritz.sedlazeck@gmail.com with problems

## Evidence

- [abstract] To predict an initial set of SVs, we applied four inference software packages (Delly, Lumpy, Pindel and cn.MOPs)24252627 to existing short-read data: "To predict an initial set of SVs, we applied four inference software packages (Delly, Lumpy, Pindel and cn.MOPs)24252627 to existing short-read data"
- [abstract] We then filtered these initial predictions, accepting SVs detected by at least two callers, to obtain 315 variant calls: "We then filtered these initial predictions, accepting SVs detected by at least two callers, to obtain 315 variant calls"
- [discussion] we applied a consensus approach to identify SVs (SURVIVOR), followed by rigorous filtering and manual inspection of all calls: "we applied a consensus approach to identify SVs (SURVIVOR), followed by rigorous filtering and manual inspection of all calls"
- [discussion] We focused on high specificity (the correctness of the inferred SV) rather than high sensitivity (attempting to detect all SVs): "We focused on high specificity (the correctness of the inferred SV) rather than high sensitivity (attempting to detect all SVs)"
- [readme] SURVIVOR is a tool set for simulating/evaluating SVs, merging and comparing SVs within and among samples, and includes various methods to reformat or summarize SVs.: "SURVIVOR is a tool set for simulating/evaluating SVs, merging and comparing SVs within and among samples, and includes various methods to reformat or summarize SVs."
