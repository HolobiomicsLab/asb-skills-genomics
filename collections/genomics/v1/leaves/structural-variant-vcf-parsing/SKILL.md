---
name: structural-variant-vcf-parsing
description: Use when you have a long-read SV caller's VCF (e.g., VolcanoSV VCF from
  Zenodo test data) and need to inspect, filter, or re-type records — specifically
  to recover DUP calls hidden among INS records.
when_to_use_negative:
- The input callset already contains DUP calls that match the ground truth — no INS-to-DUP
  recovery is needed
- 'Sex chromosomes (chrX/chrY) are in scope: VolcanoSV is designed only for autosomes
  because its phasing differs for sex chromosomes'
- The analysis is a plain Truvari benchmark of an existing callset rather than recovery
  of mislabeled insertion calls
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: VolcanoSV
  role: Produces the SV VCF with INS records targeted for duplication recovery; its
    DUP F1/GT_F1 advantage over SVIM-asm motivates the re-typing procedure
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: minimap2
  role: Aligns extracted INS inserted sequences back to the hg19 reference (genome.fa)
    to determine whether each insertion self-aligns near its breakpoint
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:98a220bac1a4cb7e2d498b1b8556e3ceeaad9c9c628486147d985a0edc74f6ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - VolcanoSV
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# structural-variant-vcf-parsing

## Summary

A method-level procedure for parsing long-read SV callset VCFs (e.g., VolcanoSV output on hg19), extracting INS inserted sequences, realigning them to the reference, and post-hoc relabeling self-aligning insertions as DUP calls.

## When to use

Use when you have a long-read SV caller's VCF (e.g., VolcanoSV VCF from Zenodo test data) and need to inspect, filter, or re-type records — specifically to recover DUP calls hidden among INS records.

## When NOT to use

- The input callset already contains DUP calls that match the ground truth — no INS-to-DUP recovery is needed
- Sex chromosomes (chrX/chrY) are in scope: VolcanoSV is designed only for autosomes because its phasing differs for sex chromosomes
- The analysis is a plain Truvari benchmark of an existing callset rather than recovery of mislabeled insertion calls

## Inputs

- VolcanoSV SV VCF output (e.g., Hifi_L2_variants.vcf) containing INS records
- hg19 reference FASTA (genome.fa and genome.fa.fai, from Zenodo record 10520476)
- minimap2 (aligner used for the example BAMs, e.g., Hifi_L2_hg19_minimap2_chr10.bam)

## Outputs

- dup_recovered_variants.vcf: full re-labeled callset with qualifying INS records re-typed as DUP, original header and non-INS records preserved
- count of INS records re-labeled as DUP (records whose inserted sequence self-aligned near their breakpoint)

## How to apply

Load the VolcanoSV SV VCF (e.g., Hifi_L2_variants.vcf from the Zenodo chr10 example, aligned to hg19 genome.fa) with a standard VCF reader such as pysam.VariantFile. For every INS record, strip the anchor base from the ALT allele to obtain the pure inserted sequence. Align each inserted sequence to the hg19 reference FASTA using minimap2 (the same aligner used to produce the example BAMs such as Hifi_L2_hg19_minimap2_chr10.bam). Re-label an INS record as DUP (SVTYPE changed to DUP) when its best alignment lands near its own breakpoint, using reciprocal-overlap tolerances comparable to VolcanoSV's evaluation settings (p=0.5, P=0.5, r=500, O=0.01). Write the full re-labeled callset preserving the original header and all non-INS records. The rationale comes from the paper's attribution of VolcanoSV's elevated DUP F1 and GT_F1 over SVIM-asm to its dedicated duplication recovery component during SV detection.

## Related tools

- **VolcanoSV** (Produces the SV VCF with INS records targeted for duplication recovery; its DUP F1/GT_F1 advantage over SVIM-asm motivates the re-typing procedure) — https://github.com/maiziezhoulab/VolcanoSV
- **minimap2** (Aligns extracted INS inserted sequences back to the hg19 reference (genome.fa) to determine whether each insertion self-aligns near its breakpoint)

## Examples

```
import pysam; vcf = pysam.VariantFile('Hifi_L2_variants.vcf'); [r for r in vcf.fetch('chr10') if 'INS' in r.info['SVTYPE']]
```

## Evaluation signals

- Output VCF parses cleanly with a standard VCF reader (e.g., pysam.VariantFile)
- Every original INS record is accounted for in the output (no records dropped or duplicated)
- The number of records re-labeled as DUP equals the count of insertions whose sequence best-aligns within the overlap tolerance (p=0.5, P=0.5, r=500, O=0.01 style settings) of its own breakpoint
- Re-labeled records differ from originals only in SVTYPE; all other VCF fields and the header are byte-identical
- After re-labeling, DUP F1/GT_F1 against a benchmark (Truvari-style) increases, consistent with VolcanoSV's reported DUP advantage

## Limitations

- Genotyping reciprocal translocations is ambiguous — the paper only calculated F1 and GT_F1 for INVs and DUPs, so this recovery logic is restricted to DUP typing
- VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different
- Translocation detection requires a WGS BAM and does not apply at single-chromosome level, so the recovery workflow cannot be extended to TRA records on single-chromosome test data

## Evidence

- [results] Only F1 and GT_F1 calculated for INVs and DUPs since genotyping reciprocal TRAs is ambiguous: "We thus only calculated F1 and GT_F1 for INVs and DUPs"
- [results] Truvari overlap parameters define the re-labeling tolerance: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [readme] Example data reference and BAMs are hg19/minimap2 based: "The example data is aligned to hg19 reference. You can download the reference files(genome.fa and genome.fa.fai) from zenode(https://zenodo.org/records/10520476)."
- [readme] Autosome-only scope of the VolcanoSV pipeline: "VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different!"
