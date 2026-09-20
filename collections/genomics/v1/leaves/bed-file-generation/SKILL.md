---
name: bed-file-generation
description: Use when you have haplotype-resolved BAM alignments from error-prone
  long reads (ONT or PacBio) and must generate a genome-wide BED of de novo candidate
  repetitive regions before tandem-repeat profiling.
when_to_use_negative:
- Target regions lie in unassembled parts of the genome (e.g., human centromeres and
  telomeres), which TRiCoLOR cannot identify repeats in
- Input is short-read or non-haplotype-resolved data, since the module assumes haplotype-resolved
  long-read alignments
- You need profiling of annotated tandem repeats rather than de novo discovery of
  candidate repetitive regions
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: TRiCoLOR SENSoR
  role: Shannon-entropy scanner whose output is the candidate repetitive regions BED
    file
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR
  role: Parent tool requiring haplotype-resolved long-read alignments as input; SENSoR
    is its repeat-identification front end
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: pysam
  role: Used to load BAM files and iterate over each sequencing read's aligned genomic
    coordinates
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:8e1e767959985b7bc300da326ad172558005a4dd997c0b934e2973b4ebe5d9da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - TRiCoLOR
  - TRiCoLOR SENSoR
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# bed-file-generation

## Summary

Reconstructs TRiCoLOR's SENSoR module, which scans haplotype-resolved ONT long-read alignments for low Shannon-entropy windows (20 bp windows, entropy <= 1.23, >= 5 read support) and emits a sorted, 3-column candidate_repetitive_regions.bed of merged intervals (~160,000-~260,000 regions, mean ~900 bp per sample).

## When to use

Use when you have haplotype-resolved BAM alignments from error-prone long reads (ONT or PacBio) and must generate a genome-wide BED of de novo candidate repetitive regions before tandem-repeat profiling.

## When NOT to use

- Target regions lie in unassembled parts of the genome (e.g., human centromeres and telomeres), which TRiCoLOR cannot identify repeats in
- Input is short-read or non-haplotype-resolved data, since the module assumes haplotype-resolved long-read alignments
- You need profiling of annotated tandem repeats rather than de novo discovery of candidate repetitive regions

## Inputs

- Haplotype-resolved long-read BAM alignment files (e.g., ONT reads aligned to GRCh38 with minimap2)
- Reference genome coordinates implicit in the BAM (GRCh38)

## Outputs

- Sorted, 3-column BED file (chromosome, start, end) named candidate_repetitive_regions.bed
- Approximately 160,000-260,000 candidate repetitive regions per sample, mean region length ~900 bp (reduced to ~150,000 after filtering for mean coverage >10x on PB data)

## How to apply

Load the haplotype-resolved BAM files with pysam and, for each sequencing read, compute the Shannon entropy of each non-overlapping sliding window of 20 bp along the aligned genomic coordinates. Retain window coordinates whose entropy is <= 1.23, the default threshold that discriminates repetitive from non-repetitive DNA on synthetic ONT and PB reads, keeping only windows supported by >= 5 independent reads. Merge retained window coordinates that fall within 100 bp of one another into contiguous candidate repetitive regions, then write them to a sorted, 3-column BED file (chromosome, start, end) named candidate_repetitive_regions.bed. The rationale is that repetitive sequence manifests as low information content, so entropy scanning recovers repeat loci without prior knowledge of motifs or locations, feeding downstream TRiCoLOR REFER profiling.

## Related tools

- **TRiCoLOR SENSoR** (Shannon-entropy scanner whose output is the candidate repetitive regions BED file) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR** (Parent tool requiring haplotype-resolved long-read alignments as input; SENSoR is its repeat-identification front end) — https://github.com/davidebolo1993/TRiCoLOR
- **pysam** (Used to load BAM files and iterate over each sequencing read's aligned genomic coordinates)

## Examples

```
docker pull davidebolo1993/tricolor  # then run the SENSoR module on haplotype-resolved BAMs per the docs at https://davidebolo1993.github.io/tricolordoc/
```

## Evaluation signals

- Output is a valid BED file with exactly 3 columns (chromosome, start, end) and is coordinate-sorted
- All emitted intervals are non-overlapping (merge step resolved any retained windows within 100 bp of each other)
- Every constituent window satisfies both criteria: entropy <= 1.23 and support from >= 5 independent reads
- Region counts fall in the reported range of ~160,000-~260,000 candidate regions per sample with mean length ~900 bp
- Downstream REFER profiling accepts the BED without schema errors, consistent with the 1.23 threshold's reported discrimination of repetitive from non-repetitive sequence

## Limitations

- TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres)
- The default entropy threshold of 1.23 was empirically estimated on synthetic ONT and PB reads; recalibration may be needed for other data types
- The tool does not yet support polyploid species or haploid chromosomes (e.g., the human Y chromosome)

## Evidence

- [methods] sensor_20bp_windows: "computes, for each sequencing read, its Shannon entropy content in non-overlapping, sliding windows of a pre-trained size (20 bp, by default)."
- [methods] entropy_threshold_123: "The default entropy treshold of 1.23 efficiently discriminates between repetitive and non-repetitive DNA sequences using syntethic ONT and PB reads"
- [methods] sensor_module_role: "This is achieved using the SENSoR module, which uses the Shannon entropy of DNA sequences to identify candidate repetitive segments in genomic sequences"
- [discussion] unassembled_regions: "TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres)"
