---
name: assembly-based-tr-allele-confirmation
description: Use when long-read TR caller output (e.g., TRiCoLOR BCF calls) for a
  sample with an available assembly-based truth set must be independently confirmed,
  by checking whether non-reference alleles appear in the truth set.
when_to_use_negative:
- The sample has no available assembly-based truth set — in that case rely on Illumina
  FM-index validation with <= 2 bp discrepancy tolerance instead
- The calls are reference-genotype TRs rather than non-reference alleles, since the
  confirmation ratio is defined only over non-reference alleles
- Repeats lie in unassembled genomic regions (centromeres, telomeres), which neither
  TRiCoLOR nor assembly-based truth sets can cover
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: TRiCoLOR
  role: Produces the de novo tandem repeat calls (SENSoR low-entropy detection + REFER
    motif profiling) whose non-reference alleles are confirmed against the assembly
    truth set
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: minimap2
  role: Aligns ONT FASTQ reads to GRCh38 to generate the haplotype-resolved BAM input
    for TRiCoLOR
- name: SPOA
  role: Builds haplotype-specific low-error consensus sequences within TRiCoLOR REFER
    (~77% error reduction for ONT, ~88% for PB) prior to call generation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:e4270da39a0774a2df8cde1b22189cfb1d8ca4ad589325eba3b2c6b2cd920409
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - Docker
  - SPOA
  - TRiCoLOR
  - TRiCoLOR REFER
  - TRiCoLOR SENSoR
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# assembly-based-tr-allele-confirmation

## Summary

Confirm non-reference tandem repeat alleles called from long-read data against an assembly-based truth set, as done for HG00733 where 42/58 (~75%) of TRiCoLOR-called non-reference alleles were confirmed. This provides an orthogonal, reference-free validation layer complementing Illumina FM-index validation.

## When to use

Use when long-read TR caller output (e.g., TRiCoLOR BCF calls) for a sample with an available assembly-based truth set must be independently confirmed, by checking whether non-reference alleles appear in the truth set.

## When NOT to use

- The sample has no available assembly-based truth set — in that case rely on Illumina FM-index validation with <= 2 bp discrepancy tolerance instead
- The calls are reference-genotype TRs rather than non-reference alleles, since the confirmation ratio is defined only over non-reference alleles
- Repeats lie in unassembled genomic regions (centromeres, telomeres), which neither TRiCoLOR nor assembly-based truth sets can cover

## Inputs

- TRiCoLOR REFER output calls in BCF-compliant format (TRs >= 50 bp) from haplotype-resolved BAM files
- Assembly-based truth set of TR alleles for the same sample (e.g., HG00733)
- Haplotype-resolved long-read alignments (ONT or PB BAMs aligned to GRCh38 with minimap2)

## Outputs

- Count of non-reference TR alleles confirmed against the assembly truth set (e.g., 42/58)
- Per-sample confirmation ratio (percentage of non-reference alleles present in the truth set)
- List of confirmed vs unconfirmed non-reference TR alleles

## How to apply

After running TRiCoLOR SENSoR and REFER on haplotype-resolved long-read alignments (e.g., HG00733 ONT/PB data aligned to GRCh38 with minimap2), extract the subset of calls representing non-reference TR alleles. Compare each non-reference allele against the assembly-based truth set for the same sample, counting an allele as confirmed when it is present in the truth set. Compute the confirmation ratio as confirmed alleles over total non-reference alleles tested (the paper reports 42/58, ~75%, for HG00733). Use this ratio alongside Illumina FM-index validation ratios (~82% HG00733, ~85% HG00514, ~86% NA19240) to judge whether the de novo TR profiling reproduced the paper's real-data performance.

## Related tools

- **TRiCoLOR** (Produces the de novo tandem repeat calls (SENSoR low-entropy detection + REFER motif profiling) whose non-reference alleles are confirmed against the assembly truth set) — https://github.com/davidebolo1993/TRiCoLOR
- **minimap2** (Aligns ONT FASTQ reads to GRCh38 to generate the haplotype-resolved BAM input for TRiCoLOR)
- **SPOA** (Builds haplotype-specific low-error consensus sequences within TRiCoLOR REFER (~77% error reduction for ONT, ~88% for PB) prior to call generation)

## Examples

```
docker run -v $PWD:/data davidebolo1993/tricolor tricolor SENSoR --bam hg00733.bam --output sensor.bed && tricolor REFER --regions sensor.bed --coverage 8 --output calls.bcf  # then intersect non-reference calls.bcf TRs with the HG00733 assembly truth set (42/58 expected)
```

## Evaluation signals

- Confirmation ratio of non-reference TR alleles against the assembly truth set matches the reported 42/58 (~75%) for HG00733
- Only non-reference alleles (those differing from GRCh38) are included in the denominator of the confirmation ratio
- Per-sample validation ratios computed alongside (Illumina FM-index, <= 2 bp discrepancy) are consistent with ~82% (HG00733), ~85% (HG00514), and ~86% (NA19240)
- All confirmed alleles derive from TRiCoLOR calls meeting the >= 50 bp minimum TR length stored in BCF format

## Limitations

- Confirmation is partial: only ~75% (42/58) of HG00733 non-reference alleles were confirmed by the assembly-based truth set, so some validated calls remain unconfirmed
- TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres)
- Truth-set concordance depends on upstream filtering (e.g., mean coverage >8 for ONT and >10x for PB regions) and consensus accuracy limits (~77%/~88% error reduction with SPOA)

## Evidence

- [abstract] On real human whole-genome sequencing data, TRiCoLOR achieves high validation rates.: "For real human whole-genome sequencing data, TRiCoLOR achieves high validation rates, suggesting its suitability to identify tandem repeat variation in personal genomes."
- [methods] TRs >= 50 bp stored in BCF format: "TRs (those ≥50 bp, by default) varying between the haplotypes or the reference are eventually stored in BCF-compliant format"
- [results] Validation against Illumina FM indexes with <= 2 bp discrepancies: "we counted as valid calls also variants that are found in the Illumina FM indexes with ≤2 bp discrepancies"
- [results] Coverage filtering of low-entropy regions: [evidence span withheld — anchor status: not_found]
- [discussion] Unassembled regions cannot be profiled: "TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres)"
- [readme] TRiCoLOR repository and documentation (README): "TRiCoLOR: tandem repeats profiler for long-read sequencing data"
