---
name: centromere-telomere-repeat-annotation-of-sv-false-discoveries
description: Use when benchmarking assembly-based SV callers (e.g., VolcanoSV-vc,
  Dipcall, SVIM-asm, PAV) for reference-bias artifacts in which SV calls against a
  reference like GRCh38 become false discoveries against T2T-CHM13, especially in
  centromeric or telomeric regions.
when_to_use_negative:
- The goal is benchmarking SNVs or small variants against a truth set such as the
  GIAB HG002 SV gold standard — that analysis uses Truvari, not the CHM13-vs-GRCh38
  FDR design
- The sample is not CHM13 (e.g., a heterozygous sample like HG002) — VolcanoSV-asm
  haplotype phasing applies, and the CHM13-vs-GRCh38 excess-call design is not directly
  transferable
- Comparing read-based (alignment-based) SV callers rather than assembly-based callers,
  whose artifacts against reference choice this FDR design targets
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: VolcanoSV
  role: Assembly-based SV caller (VolcanoSV-vc, otherasm scripts) whose CHM13-vs-GRCh38
    FDR is the primary measurement target
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: Dipcall
  role: Comparison assembly-based caller run on the CHM13 diploid assembly against
    both GRCh38 and T2T-CHM13
  repo: https://github.com/lh3/dipcall
- name: SVIM-asm
  role: Comparison assembly-based caller run on the CHM13 diploid assembly against
    both GRCh38 and T2T-CHM13
  repo: https://github.com/eldariont/svim-asm
- name: PAV
  role: Comparison assembly-based caller run on the CHM13 diploid assembly against
    both GRCh38 and T2T-CHM13
  repo: https://github.com/EichlerLab/pav
- name: Flagger
  role: Read-mapping-based assembly evaluation tool referenced by the article for
    assembly quality assessment (HMM-Flagger recommended)
  repo: https://github.com/mobinasri/flagger
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:0a9a084d12fdf424f4bf059ff394c8ee02cd6b4b160fe4de75046e55d5e938c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - Dipcall
  - Flagger
  - PAV
  - SVIM-asm
  - VolcanoSV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Centromere/telomere repeat annotation of SV false discoveries

## Summary

A benchmarking procedure that computes per-tool false discovery rates (FDR) for assembly-based SV callers by comparing SV calls made against T2T-CHM13 versus GRCh38, then intersects the excess calls with T2T centromere satellite and telomere BED tracks to attribute false discoveries to repetitive sequence.

## When to use

Use when benchmarking assembly-based SV callers (e.g., VolcanoSV-vc, Dipcall, SVIM-asm, PAV) for reference-bias artifacts in which SV calls against a reference like GRCh38 become false discoveries against T2T-CHM13, especially in centromeric or telomeric regions.

## When NOT to use

- The goal is benchmarking SNVs or small variants against a truth set such as the GIAB HG002 SV gold standard — that analysis uses Truvari, not the CHM13-vs-GRCh38 FDR design
- The sample is not CHM13 (e.g., a heterozygous sample like HG002) — VolcanoSV-asm haplotype phasing applies, and the CHM13-vs-GRCh38 excess-call design is not directly transferable
- Comparing read-based (alignment-based) SV callers rather than assembly-based callers, whose artifacts against reference choice this FDR design targets

## Inputs

- CHM13 diploid assembly FASTA (contig names tagged hp1/hp2)
- --chrom_list text file listing one contig name per line
- CHM13 PacBio HiFi BAM: Hifi_L6 (SRR1129212, 33x) and Hifi_L7 (SRX5633451, 23.7x)
- T2T-CHM13 reference genome FASTA
- GRCh38 reference genome FASTA
- chm13v2.0_censat_v2.0.bed (centromere satellite annotation)
- chm13v2.0_telomere.bed (telomere annotation)
- SV call sets (VCF) from each assembly-based tool against both references

## Outputs

- Per-tool false discovery rate values (SVs vs CHM13 / SVs vs GRCh38)
- Centromere and telomere overlap counts per tool's false-discovery SV calls
- Combined comparison table: false_discovery_rate_table.tsv

## How to apply

Reformat the CHM13 diploid assembly FASTA so contig names carry 'hp1'/'hp2' haplotype tags and build a --chrom_list file listing one contig name per line, as required by VolcanoSV-vc in 'otherasm' mode. Run VolcanoSV-vc (volcanosv-vc-large-indel-otherasm.py and volcanosv-vc-complex-sv-otherasm.py) against T2T-CHM13 using the CHM13 PacBio HiFi BAMs (Hifi_L6, SRR1129212, 33x; Hifi_L7, SRX5633451, 23.7x), and run the comparison assembly-based callers Dipcall, SVIM-asm, and PAV on the same diploid assembly against both GRCh38 and T2T-CHM13. For each tool, compute the FDR as the ratio of the number of detected SVs against CHM13 to the number of SVs against GRCh38. Intersect each tool's false-discovery SV calls with the T2T annotation BED files chm13v2.0_censat_v2.0.bed (centromere satellites) and chm13v2.0_telomere.bed (telomeres) to count how many false discoveries fall in centromeric or telomeric sequence. Compile per-tool FDR values with centromere/telomere overlap counts into a single comparison table (false_discovery_rate_table.tsv); the expected result is that VolcanoSV-vc exhibits the lowest FDR among the four assembly-based callers.

## Related tools

- **VolcanoSV** (Assembly-based SV caller (VolcanoSV-vc, otherasm scripts) whose CHM13-vs-GRCh38 FDR is the primary measurement target) — https://github.com/maiziezhoulab/VolcanoSV
- **Dipcall** (Comparison assembly-based caller run on the CHM13 diploid assembly against both GRCh38 and T2T-CHM13) — https://github.com/lh3/dipcall
- **SVIM-asm** (Comparison assembly-based caller run on the CHM13 diploid assembly against both GRCh38 and T2T-CHM13) — https://github.com/eldariont/svim-asm
- **PAV** (Comparison assembly-based caller run on the CHM13 diploid assembly against both GRCh38 and T2T-CHM13) — https://github.com/EichlerLab/pav
- **Flagger** (Read-mapping-based assembly evaluation tool referenced by the article for assembly quality assessment (HMM-Flagger recommended)) — https://github.com/mobinasri/flagger

## Examples

```
python volcanosv-vc-large-indel-otherasm.py --bam_file Hifi_L6.bam --reference chm13.fa -cl chrom_list.txt; bedtools intersect -a volcano_sv_fdr.vcf -b chm13v2.0_censat_v2.0.bed chm13v2.0_telomere.bed > fdr_repeat_overlaps.tsv
```

## Evaluation signals

- VolcanoSV-vc shows the lowest FDR among the four assembly-based callers (VolcanoSV, PAV, SVIM-asm, Dipcall), consistent with the paper's reported finding
- Each tool's FDR equals the number of SVs detected against T2T-CHM13 divided by the number of SVs detected against GRCh38
- The output table false_discovery_rate_table.tsv contains per-tool FDR values plus centromere and telomere overlap counts
- Contig names in the reformatted CHM13 assembly FASTA, BAM headers, and --chrom_list file are consistent (one name per line, exact matches)
- False-discovery intersection counts are non-empty only when calls against CHM13 exceed calls against GRCh38 for that tool

## Limitations

- The CHM13 sample derives from a single homozygous complete hydatidiform mole, so the VolcanoSV-asm haplotype-phasing component is not used; SV calling relies on externally supplied diploid assemblies
- VolcanoSV is designed only for autosome chromosomes; phasing of sex chromosomes is not supported
- Reference-choice artifacts may not be the sole source of excess calls; assembly algorithms may also miss complex SVs because they prefer the simplest paths through overlapping reads

## Evidence

- [results] FDR definition: "The false discovery rate was computed as the ratio of the number of detected SVs against CHM13 to the number of SVs against GRCh38"
- [intro] Low FDR claim: "VolcanoSV-vc, the assembly-based SV calling component, has a low false discovery rate"
- [discussion] BED track sources: "we downloaded two BED files from https://s3-us-west-2.amazonaws.com/human-pangenomics/T2T/CHM13/assemblies/annotation/chm13v2.0_censat_v2.0.bed"
- [results] CHM13 assembly mode: "We did not use the VolcanoSV-asm component for the diploid assembly since the CHM13 sample derives from a single homozygous complete hydatidiform mole"
- [readme] chrom_list requirement: "you must supply a `--chrom_list`/`-cl` (or `-chl`) text file listing one chromosome/contig name per line, exactly as they appear in your reference"
