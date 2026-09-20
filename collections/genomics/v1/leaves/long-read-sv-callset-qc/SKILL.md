---
name: long-read-sv-callset-qc
description: Use when a VolcanoSV VCF from simulated long-read data (VISOR/PBSIM hg19)
  must be QC'd for INS-vs-DUP mislabeling, or when explaining/benchmarking elevated
  DUP F1 and GT_F1 against SVIM-asm.
when_to_use_negative:
- The input VCF contains no INS records or the analysis targets only real (non-simulated)
  data where INS/DUP labels are already trusted (e.g., GIAB gold-standard benchmarking).
- The goal is a full SV caller benchmark rather than INS/DUP label recovery — Truvari
  comparison against the gold standard should be run separately, and genotyping of
  reciprocal TRAs is ambiguous (only F1 and GT_F1 are meaningful for INVs and DUPs).
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: VolcanoSV
  role: Produces the hybrid-reference/de novo-assembly SV callset (VCF) whose INS
    records are examined and re-labeled; its dedicated duplication recovery process
    motivates this QC step
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: minimap2
  role: Aligns each extracted inserted sequence back to the hg19 reference genome.fa,
    the same mapper used for the example BAMs
- name: Truvari
  role: Defines the reciprocal-overlap tolerance (p=0.5, P=0.5, r=500, O=0.01) used
    to judge whether an inserted sequence self-aligns near its own breakpoint
- name: SVIM-asm
  role: Assembly-based comparison baseline against which VolcanoSV's DUP F1 and GT_F1
    improvements are assessed
  repo: https://github.com/eldariont/svim-asm
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:d32aee792262d81fe2f33caaff2d1e500a5733000188a8571147283af7f070b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - SVIM-asm
  - Truvari
  - VolcanoSV
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# long-read-sv-callset-qc

## Summary

Quality-control and re-labeling procedure for long-read SV callsets (VCF) in which insertion (INS) records whose inserted sequence self-aligns near their own breakpoint are recovered as duplication (DUP) calls, the mechanism VolcanoSV uses to explain its elevated DUP F1 and GT_F1 versus SVIM-asm on simulated data.

## When to use

Use when a VolcanoSV VCF from simulated long-read data (VISOR/PBSIM hg19) must be QC'd for INS-vs-DUP mislabeling, or when explaining/benchmarking elevated DUP F1 and GT_F1 against SVIM-asm.

## When NOT to use

- The input VCF contains no INS records or the analysis targets only real (non-simulated) data where INS/DUP labels are already trusted (e.g., GIAB gold-standard benchmarking).
- The goal is a full SV caller benchmark rather than INS/DUP label recovery — Truvari comparison against the gold standard should be run separately, and genotyping of reciprocal TRAs is ambiguous (only F1 and GT_F1 are meaningful for INVs and DUPs).

## Inputs

- VolcanoSV SV VCF containing INS records (e.g., Hifi_L2_variants.vcf from Zenodo record 10520476)
- hg19 reference FASTA (genome.fa and genome.fa.fai from Zenodo, or refdata-hg19-2.1.0)
- Long-read BAM aligned with minimap2 to hg19 (e.g., Hifi_L2_hg19_minimap2_chr10.bam), for provenance/coverage checks

## Outputs

- dup_recovered_variants.vcf: full callset with qualifying INS records re-labeled SVTYPE=DUP, original header and non-INS records preserved
- Count of INS records re-labeled as DUP (number of insertions whose sequence self-aligned near their breakpoint)

## How to apply

Parse the VolcanoSV SV VCF (e.g., Hifi_L2_variants.vcf from Zenodo record 10520476) and strip anchor bases from each INS record's ALT allele to obtain the pure inserted sequence. Align each inserted sequence to the hg19 reference FASTA (genome.fa, the reference used for the example BAMs such as Hifi_L2_hg19_minimap2_chr10.bam) using minimap2, matching the mapper used for the source data. Re-label an INS record as DUP (change SVTYPE only, leave all other VCF fields untouched) when the best alignment of its inserted sequence lands within reciprocal-overlap tolerance of the record's own breakpoint, comparable to the Truvari-style settings VolcanoSV uses for evaluation (p=0.5, P=0.5, r=500, O=0.01). Write the full re-labeled callset to dup_recovered_variants.vcf, preserving the original header and all non-INS records; the rationale is that VolcanoSV's dedicated duplication recovery process is the primary reason its F1 and GT_F1 for DUPs exceed SVIM-asm across all simulated datasets.

## Related tools

- **VolcanoSV** (Produces the hybrid-reference/de novo-assembly SV callset (VCF) whose INS records are examined and re-labeled; its dedicated duplication recovery process motivates this QC step) — https://github.com/maiziezhoulab/VolcanoSV
- **minimap2** (Aligns each extracted inserted sequence back to the hg19 reference genome.fa, the same mapper used for the example BAMs)
- **Truvari** (Defines the reciprocal-overlap tolerance (p=0.5, P=0.5, r=500, O=0.01) used to judge whether an inserted sequence self-aligns near its own breakpoint)
- **SVIM-asm** (Assembly-based comparison baseline against which VolcanoSV's DUP F1 and GT_F1 improvements are assessed) — https://github.com/eldariont/svim-asm

## Examples

```
minimap2 -ax map-hifi genome.fa inserted_seqs.fa | samtools sort -o ins_aln.bam
```

## Evaluation signals

- The output dup_recovered_variants.vcf parses with a standard VCF reader and is well-formed.
- Every original INS record is accounted for in the output; no records are dropped and non-INS records are byte-identical to the input.
- The number of records re-labeled as DUP exactly matches the count of insertions whose extracted sequence self-aligned near its own breakpoint under the p=0.5, P=0.5, r=500, O=0.01 reciprocal-overlap tolerance.
- Re-run Truvari benchmarking with p=0.5, P=0.5, r=500, O=0.01: DUP F1 and GT_F1 for the re-labeled callset should improve relative to the unrecovered callset.
- Simulated truth provenance holds: the underlying data were created by inserting SVs with VISOR (v1.1.2) into hg19 and simulating reads with PBSIM (v3.0.0), so recovered DUPs should correspond to VISOR-inserted duplications.

## Limitations

- Only F1 and GT_F1 (not genotype-aware precision metrics) are meaningful for DUPs/INVs; genotyping of reciprocal TRAs is ambiguous.
- Simulated-data gains may not transfer to real complex somatic SVs: VolcanoSV's recall on HCC1395 somatic SVs was relatively lower than on simulated data, and the HCC1395 high-confidence callset included only alignment-based tools, potentially biasing evaluation.
- Assembly algorithms prefer simplest paths through overlapping reads, which may cause some complex SVs to be missed regardless of label recovery.
- VolcanoSV is designed only for autosome chromosomes; sex chromosomes require a different phasing procedure.

## Evidence

- [results] Truvari tolerance parameters: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [results] Genotyping scope for complex SVs: "We thus only calculated F1 and GT_F1 for INVs and DUPs"
- [results] Simulated data provenance: "We used VISOR (v1.1.2)33 to insert SNPs and complex SVs into the hg19 human reference, and then used PBSIM (v3.0.0)34 to simulate reads"
- [readme] Reference and example data location: "The example data is aligned to hg19 reference. You can download the reference files(genome.fa and genome.fa.fai) from zenode(https://zenodo.org/records/10520476)."
- [abstract] Superior complex SV performance claim: "VolcanoSV outperforms assembly-based tools in the identification of complex SVs, including translocations, duplications, and inversions, in both simulated and real cancer data."
