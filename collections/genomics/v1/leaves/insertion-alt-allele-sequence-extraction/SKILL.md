---
name: insertion-alt-allele-sequence-extraction
description: Use when a VolcanoSV (or similar assembly-based) SV VCF contains INS
  records whose inserted sequences may self-align near their own breakpoints, and
  you need to recover misclassified duplication calls for DUP benchmarking.
when_to_use_negative:
- The VCF contains no INS records or ALT alleles carry only symbolic tags (<INS>)
  without inserted sequence, so no sequence can be extracted for realignment
- The goal is benchmarking INV or TRA calls, where genotyping reciprocal translocations
  is ambiguous and only F1/GT_F1 are used without this recovery step
- The callset was produced by a tool whose INS records already carry correct SVTYPE
  labels (e.g., SVIM-asm), making self-alignment-based re-labeling redundant
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: VolcanoSV
  role: Produces the SV VCF whose INS records' ALT alleles are parsed and whose duplication
    recovery behavior this skill reproduces
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: minimap2
  role: Aligns each extracted inserted sequence to the hg19 reference genome, as used
    for the example BAMs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:b255f11840515bc4b66b462a7c70226c3e6c01878510a63971ddc60c41de4456
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - VolcanoSV
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# insertion-alt-allele-sequence-extraction

## Summary

Extracts the pure inserted sequence from ALT alleles of INS records in a VolcanoSV SV VCF and realigns each sequence to the hg19 reference with minimap2 to determine which insertions are actually duplications, supporting re-labeling of INS calls as DUP.

## When to use

Use when a VolcanoSV (or similar assembly-based) SV VCF contains INS records whose inserted sequences may self-align near their own breakpoints, and you need to recover misclassified duplication calls for DUP benchmarking.

## When NOT to use

- The VCF contains no INS records or ALT alleles carry only symbolic tags (<INS>) without inserted sequence, so no sequence can be extracted for realignment
- The goal is benchmarking INV or TRA calls, where genotyping reciprocal translocations is ambiguous and only F1/GT_F1 are used without this recovery step
- The callset was produced by a tool whose INS records already carry correct SVTYPE labels (e.g., SVIM-asm), making self-alignment-based re-labeling redundant

## Inputs

- VolcanoSV SV VCF containing INS records with sequence-carrying ALT alleles (e.g., Hifi_L2_variants.vcf)
- Reference genome FASTA used for alignment, e.g., hg19 genome.fa with genome.fa.fai (Zenodo record 10520476)

## Outputs

- Re-labeled callset VCF (dup_recovered_variants.vcf) in which qualifying INS records have SVTYPE=DUP and all other records/header are preserved
- Per-INS alignment results mapping each inserted sequence to its best reference location

## How to apply

Load the VolcanoSV SV VCF (e.g., Hifi_L2_variants.vcf from Zenodo test data) and strip anchor bases from each INS record's ALT allele to obtain the pure inserted sequence. Align each extracted sequence to the hg19 reference (genome.fa used for the example BAMs, e.g. Hifi_L2_hg19_minimap2_chr10.bam) with minimap2. An INS record qualifies if the best alignment of its inserted sequence lands near the record's own breakpoint within reciprocal-overlap tolerance comparable to the Truvari settings VolcanoSV uses for evaluation (p=0.5, P=0.5, r=500, O=0.01). Re-label qualifying records by changing SVTYPE to DUP, write the full callset preserving headers and non-INS records to dup_recovered_variants.vcf. The rationale is that VolcanoSV's dedicated duplication recovery step explains its elevated F1 and GT_F1 on DUPs versus SVIM-asm, so recovering these calls reproduces that behavior before benchmarking.

## Related tools

- **VolcanoSV** (Produces the SV VCF whose INS records' ALT alleles are parsed and whose duplication recovery behavior this skill reproduces) — https://github.com/maiziezhoulab/VolcanoSV
- **minimap2** (Aligns each extracted inserted sequence to the hg19 reference genome, as used for the example BAMs)

## Examples

```
minimap2 -ax map-hifi genome.fa ins_seqs.fa > ins_aln.sam
```

## Evaluation signals

- Output VCF parses with a standard VCF reader (well-formed)
- Every original INS record is accounted for in the output VCF
- The number of records re-labeled as DUP equals the count of insertions whose sequence self-aligned near its breakpoint
- Non-INS records and the original VCF header are unchanged relative to the input VCF
- Re-labeled DUP calls, when benchmarked with Truvari (p=0.5, P=0.5, r=500, O=0.01), improve DUP F1/GT_F1 consistent with VolcanoSV's reported advantage over SVIM-asm

## Limitations

- Anchor-base stripping and self-alignment only recover duplications already present as INS calls; truly missed duplications cannot be recovered this way
- Re-labeling depends on an overlap tolerance; insertions whose sequence aligns ambiguously (e.g., in segmental duplications) may be mis-re-labeled
- The comparison assumes evaluation settings matching the paper; only F1 and GT_F1 are meaningful metrics for DUPs since genotyping reciprocal translocations is ambiguous

## Evidence

- [results] Truvari reciprocal-overlap settings used by VolcanoSV: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [readme] Example data provenance (hg19 reference, chr10 VCF, Zenodo): "The example data is aligned to hg19 reference. You can download the reference files(genome.fa and genome.fa.fai) from zenode(https://zenodo.org/records/10520476)."
- [readme] Test data includes VCF files for Hifi, CLR and ONT: "we provided the chr10 BAM file, contigs file and VCF file for Hifi, CLR and ONT data. You can download them from [zenodo](https://zenodo.org/records/10520476)."
- [abstract] Motivation: VolcanoSV outperforms on complex SVs including duplications: "VolcanoSV outperforms assembly-based tools in the identification of complex SVs, including translocations, duplications, and inversions, in both simulated and real cancer data."
