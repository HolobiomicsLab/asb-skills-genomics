---
name: inserted-sequence-realignment-to-reference
description: Use when auditing or reproducing VolcanoSV DUP calls from its SV VCF
  output, or when explaining/comparing VolcanoSV vs SVIM-asm duplication detection
  performance on simulated long-read datasets.
when_to_use_negative:
- When evaluating real complex somatic SVs against the HCC1395 high-confidence callset,
  where only recall is plotted and duplication recovery behavior may differ
- When the callset was produced by a different SV caller (e.g., SVIM-asm or cuteSV)
  whose INS records do not carry the full inserted sequence in the ALT allele
- When analyzing sex chromosomes — the README states VolcanoSV is designed only for
  autosome chromosomes
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: VolcanoSV
  role: Source of the SV VCF callset whose INS records are extracted and re-labeled;
    its duplication recovery process accounts for its elevated DUP F1/GT_F1 vs SVIM-asm
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: minimap2
  role: Aligner used to realign each extracted inserted sequence to the hg19 reference
    FASTA, matching the alignment used for the example BAMs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:67653a73e6fd0f4496b4f0ee6337c4d757f403f70032b2bd3fdee59b5cdd3212
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - VolcanoSV
  - minimap2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# inserted-sequence-realignment-to-reference

## Summary

A duplication-recovery filter that distinguishes true duplications from insertion calls by extracting the pure inserted sequence from each INS record in a VolcanoSV SV VCF, realigning it to the hg19 reference with minimap2, and re-labeling records whose inserted sequence self-aligns near their own breakpoint as DUP. This mechanistically explains VolcanoSV's elevated DUP F1/GT_F1 scores over SVIM-asm on simulated data.

## When to use

Use when auditing or reproducing VolcanoSV DUP calls from its SV VCF output, or when explaining/comparing VolcanoSV vs SVIM-asm duplication detection performance on simulated long-read datasets.

## When NOT to use

- When evaluating real complex somatic SVs against the HCC1395 high-confidence callset, where only recall is plotted and duplication recovery behavior may differ
- When the callset was produced by a different SV caller (e.g., SVIM-asm or cuteSV) whose INS records do not carry the full inserted sequence in the ALT allele
- When analyzing sex chromosomes — the README states VolcanoSV is designed only for autosome chromosomes

## Inputs

- VolcanoSV SV VCF containing INS records (e.g., Hifi_L2_variants.vcf)
- hg19 reference FASTA (genome.fa and genome.fa.fai, downloadable from Zenodo record 10520476)
- minimap2 aligner (same aligner used for the example BAMs)

## Outputs

- Re-labeled callset VCF (dup_recovered_variants.vcf) with qualifying INS records changed to SVTYPE=DUP
- Count of insertions whose inserted sequence self-aligned near their breakpoint (re-labeled DUP count)

## How to apply

Load the VolcanoSV SV VCF (e.g., Hifi_L2_variants.vcf from the Zenodo test data at https://zenodo.org/records/10520476) and parse every INS record, stripping anchor bases from the ALT allele to obtain the pure inserted sequence. Align each extracted sequence to the hg19 reference FASTA (genome.fa) using minimap2, the same aligner used to produce the example BAMs (e.g., Hifi_L2_hg19_minimap2_chr10.bam). For each INS call, check whether the best alignment of its inserted sequence lands near the record's own breakpoint position, using a reciprocal-overlap tolerance comparable to the Truvari-style settings VolcanoSV uses for evaluation (p=0.5, P=0.5, r=500, O=0.01). Re-label qualifying records by changing SVTYPE to DUP while leaving all other VCF fields and records untouched, and write the full callset to a new VCF (dup_recovered_variants.vcf), preserving the original header and non-INS records.

## Related tools

- **VolcanoSV** (Source of the SV VCF callset whose INS records are extracted and re-labeled; its duplication recovery process accounts for its elevated DUP F1/GT_F1 vs SVIM-asm) — https://github.com/maiziezhoulab/VolcanoSV
- **minimap2** (Aligner used to realign each extracted inserted sequence to the hg19 reference FASTA, matching the alignment used for the example BAMs)

## Examples

```
minimap2 -ax map-hifi genome.fa inserted_seqs.fa | samtools sort -o ins_realign.bam; python relabel_ins_to_dup.py --vcf Hifi_L2_variants.vcf --bam ins_realign.bam --p 0.5 --r 500 --out dup_recovered_variants.vcf
```

## Evaluation signals

- Output VCF parses with a standard VCF reader (well-formed header and records)
- Every original INS record is accounted for in the output VCF (none dropped or silently modified beyond SVTYPE)
- The number of records re-labeled as DUP equals the count of insertions whose inserted sequence best-aligned near its own breakpoint within reciprocal-overlap tolerance (p=0.5, P=0.5, r=500, O=0.01)
- Non-INS records and all other VCF fields are byte-identical to the input callset
- Recovered DUP calls reproduce VolcanoSV's higher DUP F1 and GT_F1 relative to SVIM-asm when benchmarked with Truvari against the simulated truth set

## Limitations

- Translocation detection requires a WGS BAM, so the workflow is not applicable to single-chromosome-level analysis
- The Truvari-style tolerance (p=0.5, P=0.5, r=500, O=0.01) defines self-alignment 'near the breakpoint' only approximately; edge cases depend on this reciprocal-overlap choice
- VolcanoSV's recall for real complex SVs was relatively lower than on simulated data, so recovery behavior validated on simulations may not transfer to real tumor data

## Evidence

- [results] dedicated duplication recovery process explanation: [evidence span withheld — anchor status: not_found]
- [results] Truvari benchmarking parameters: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [readme] example data reference and minimap2 alignment: "The example data is aligned to hg19 reference. You can download the reference files(genome.fa and genome.fa.fai) from zenode(https://zenodo.org/records/10520476)."
- [readme] autosome-only scope: "VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different!"
- [results] only F1 and GT_F1 for DUPs: "We thus only calculated F1 and GT_F1 for INVs and DUPs"
