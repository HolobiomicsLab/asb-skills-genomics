---
name: duplication-reclassification-filtering
description: Use when a long-read SV callset (VCF) contains INS records whose inserted
  sequences self-align near their own breakpoints, suggesting they are misclassified
  duplications that need DUP re-labeling.
when_to_use_negative:
- The callset already contains correctly typed DUP calls from a caller with native
  duplication recovery (e.g. VolcanoSV output), so reclassification would double-count
  duplications
- The reference and BAM contig naming differ from the VCF, since VolcanoSV requires
  one consistent naming scheme and a --chrom_list file; inconsistent naming breaks
  alignment-based verification
- Evaluating translocations (TRA) with genotype metrics — the article notes only F1
  and GT_F1 are computed for INVs and DUPs because genotyping reciprocal TRAs is ambiguous
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: minimap2
  role: Aligns extracted inserted sequences from INS ALT alleles back to the hg19
    reference to test whether the insertion self-aligns near its own breakpoint
- name: VolcanoSV
  role: Source of the SV VCF whose INS records are re-labeled; its native duplication
    recovery process motivates the reclassification heuristic
  repo: https://github.com/maiziezhoulab/VolcanoSV
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:9a72c58ac9db9bfe6d2ac089d23ade2d289dd857bacfd80bdf8126be5ce9aecd
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# duplication-reclassification-filtering

## Summary

Re-labels insertion (INS) calls in a long-read SV VCF as duplications (DUP) when the extracted inserted sequence realigns near the record's own breakpoint, reproducing the dedicated duplication recovery step that drives VolcanoSV's elevated F1 and GT_F1 on DUP detection versus SVIM-asm.

## When to use

Use when a long-read SV callset (VCF) contains INS records whose inserted sequences self-align near their own breakpoints, suggesting they are misclassified duplications that need DUP re-labeling.

## When NOT to use

- The callset already contains correctly typed DUP calls from a caller with native duplication recovery (e.g. VolcanoSV output), so reclassification would double-count duplications
- The reference and BAM contig naming differ from the VCF, since VolcanoSV requires one consistent naming scheme and a --chrom_list file; inconsistent naming breaks alignment-based verification
- Evaluating translocations (TRA) with genotype metrics — the article notes only F1 and GT_F1 are computed for INVs and DUPs because genotyping reciprocal TRAs is ambiguous

## Inputs

- SV callset VCF containing INS records (e.g. Hifi_L2_variants.vcf from VolcanoSV output)
- Reference genome FASTA (hg19 genome.fa plus .fai index, from Zenodo record 10520476 or refdata-hg19-2.1.0)

## Outputs

- dup_recovered_variants.vcf — full callset with qualifying INS records re-labeled as SVTYPE=DUP, original header and non-INS records preserved

## How to apply

Load the SV VCF (e.g. the Hifi_L2_variants.vcf example output from the VolcanoSV Zenodo record) and parse every INS record, stripping anchor bases from the ALT allele to isolate the pure inserted sequence. Align each extracted sequence to the hg19 reference (genome.fa, the same reference used for the example BAMs such as Hifi_L2_hg19_minimap2_chr10.bam) using minimap2. For each call, check whether the best alignment of the inserted sequence lands near the record's own breakpoint within a reciprocal-overlap tolerance comparable to the VolcanoSV evaluation settings (p=0.5, P=0.5, r=500, O=0.01). Change SVTYPE to DUP only for qualifying records, leave all other records and VCF fields untouched, and write the full callset, preserving the original header, to dup_recovered_variants.vcf. The rationale is that VolcanoSV's pipeline includes a dedicated duplication recovery process, which explains why its DUP F1 and GT_F1 exceed SVIM-asm's on all simulated datasets.

## Related tools

- **minimap2** (Aligns extracted inserted sequences from INS ALT alleles back to the hg19 reference to test whether the insertion self-aligns near its own breakpoint)
- **VolcanoSV** (Source of the SV VCF whose INS records are re-labeled; its native duplication recovery process motivates the reclassification heuristic) — https://github.com/maiziezhoulab/VolcanoSV

## Examples

```
minimap2 -ax map-hifi genome.fa ins_seqs.fa | samtools sort -@4 > ins_aln.bam
```

## Evaluation signals

- The output VCF parses with a standard VCF reader (well-formed header and records)
- Every original INS record is accounted for in the output (none dropped, none duplicated)
- The number of records re-labeled as DUP exactly equals the count of insertions whose sequence best-aligned near its own breakpoint within the r=500 / p=0.5 tolerance
- All non-INS records and untouched INS records are byte-identical in content (only SVTYPE changed for qualifying calls)
- Re-running with a different reciprocal-overlap threshold changes only the boundary cases, confirming the reclassification is deterministic

## Limitations

- The reciprocal-overlap tolerance mirrors Truvari-style evaluation settings (p=0.5, P=0.5, r=500, O=0.01); other benchmarks may call different subsets of insertions duplications
- Only F1 and GT_F1 can be assessed for the reclassified DUPs; the article notes only these metrics were calculated for INVs and DUPs because genotyping reciprocal complex SV types is ambiguous
- The heuristic is grounded in simulated-data analysis of VolcanoSV vs SVIM-asm; on real data VolcanoSV's recall for complex SVs was relatively lower than on simulated data

## Evidence

- [results] Truvari tolerance settings used by VolcanoSV evaluation: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [results] DUP F1 and GT_F1 are the relevant comparison metrics: "We thus only calculated F1 and GT_F1 for INVs and DUPs"
- [abstract] Motivating finding on complex SV performance: "VolcanoSV outperforms assembly-based tools in the identification of complex SVs, including translocations, duplications, and inversions, in both simulated and real cancer data."
- [readme] Example data reference and file provenance: "The example data is aligned to hg19 reference. You can download the reference files(genome.fa and genome.fa.fai) from zenode(https://zenodo.org/records/10520476)."
