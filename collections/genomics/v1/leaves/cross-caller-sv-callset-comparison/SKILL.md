---
name: cross-caller-sv-callset-comparison
description: Use when comparing SV callsets from multiple assembly-based callers on
  the same assembly to quantify reference-induced false discoveries, e.g., CHM13 assemblies
  called against T2T-CHM13 vs GRCh38.
when_to_use_negative:
- Evaluating alignment-based read-mapping SV callers against a truth set (e.g., GIAB
  HG002 gold standard with Truvari) — use a recall/precision benchmark instead
- The sample is haplotype-diverse and haplotype phasing is needed; CHM13 derives from
  a single homozygous complete hydatidiform mole, so VolcanoSV-asm phasing is unsuitable
  here and the pipeline is run only for variant calling
- Dipcall is being considered for complex SV (TRA/DUP/INV) detection — it was excluded
  because it was not designed to detect complex SVs
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: VolcanoSV
  role: Assembly-based SV caller under test; volcanosv-vc-large-indel-otherasm.py
    and volcanosv-vc-complex-sv-otherasm.py call SVs from other assemblies against
    T2T-CHM13
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: Dipcall
  role: Comparison assembly-based caller run on the same CHM13 assembly against both
    references
  repo: https://github.com/lh3/dipcall
- name: SVIM-asm
  role: Comparison assembly-based caller run on the same CHM13 assembly against both
    references
  repo: https://github.com/eldariont/svim-asm
- name: PAV
  role: Comparison assembly-based caller run on the same CHM13 assembly against both
    references
  repo: https://github.com/EichlerLab/pav
- name: Flagger
  role: Recommended companion for evaluating dual/diploid assemblies via read-mapping-based
    mis-assembly detection
  repo: https://github.com/mobinasri/flagger
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:455b4f5afebbe1c738d36a90412145d1d7798cf1577f3b44a6f96c157da0f966
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
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-caller-sv-callset-comparison

## Summary

Benchmark false discovery rates of assembly-based SV callers (VolcanoSV-vc, PAV, SVIM-asm, Dipcall) by calling SVs from the same CHM13 diploid assembly against T2T-CHM13 versus GRCh38 references and computing the ratio of call counts, annotating false discoveries against centromere/telomere BED tracks.

## When to use

Use when comparing SV callsets from multiple assembly-based callers on the same assembly to quantify reference-induced false discoveries, e.g., CHM13 assemblies called against T2T-CHM13 vs GRCh38.

## When NOT to use

- Evaluating alignment-based read-mapping SV callers against a truth set (e.g., GIAB HG002 gold standard with Truvari) — use a recall/precision benchmark instead
- The sample is haplotype-diverse and haplotype phasing is needed; CHM13 derives from a single homozygous complete hydatidiform mole, so VolcanoSV-asm phasing is unsuitable here and the pipeline is run only for variant calling
- Dipcall is being considered for complex SV (TRA/DUP/INV) detection — it was excluded because it was not designed to detect complex SVs

## Inputs

- CHM13 diploid assembly FASTA (contigs tagged hp1/hp2)
- CHM13 PacBio HiFi BAM (Hifi_L6, SRR1129212, 33x; Hifi_L7, SRX5633451, 23.7x)
- T2T-CHM13 reference genome (FASTA + index)
- GRCh38 reference genome (FASTA + index)
- --chrom_list text file with one contig name per line
- Annotation BED tracks: chm13v2.0_censat_v2.0.bed and chm13v2.0_telomere.bed

## Outputs

- Per-tool SV callsets (VCF) against both T2T-CHM13 and GRCh38 references
- Per-tool false discovery rates (ratio of SVs detected against CHM13 to SVs against GRCh38)
- Centromere/telomere overlap counts for false-discovery SV calls
- Comparison table false_discovery_rate_table.tsv

## How to apply

Reformat the CHM13 diploid assembly FASTA so contig names carry hp1/hp2 haplotype tags, then run VolcanoSV-vc (volcanosv-vc-large-indel-otherasm.py and volcanosv-vc-complex-sv-otherasm.py) against the T2T-CHM13 reference with the CHM13 PacBio HiFi BAMs (Hifi_L6, SRR1129212, 33x; Hifi_L7, SRX5633451, 23.7x), supplying a --chrom_list file listing one contig name per line. Run Dipcall, SVIM-asm, and PAV on the same diploid assembly against both GRCh38 and T2T-CHM13. Compute each tool's false discovery rate as the ratio of SVs detected against CHM13 (T2T) to SVs detected against GRCh38, then intersect the false-discovery calls with chm13v2.0_censat_v2.0.bed (centromere satellites) and chm13v2.0_telomere.bed to localize artifacts. Compile per-tool FDR and centromere/telomere overlap counts into a single comparison table (false_discovery_rate_table.tsv); correct application yields VolcanoSV-vc with the lowest FDR.

## Related tools

- **VolcanoSV** (Assembly-based SV caller under test; volcanosv-vc-large-indel-otherasm.py and volcanosv-vc-complex-sv-otherasm.py call SVs from other assemblies against T2T-CHM13) — https://github.com/maiziezhoulab/VolcanoSV
- **Dipcall** (Comparison assembly-based caller run on the same CHM13 assembly against both references) — https://github.com/lh3/dipcall
- **SVIM-asm** (Comparison assembly-based caller run on the same CHM13 assembly against both references) — https://github.com/eldariont/svim-asm
- **PAV** (Comparison assembly-based caller run on the same CHM13 assembly against both references) — https://github.com/EichlerLab/pav
- **Flagger** (Recommended companion for evaluating dual/diploid assemblies via read-mapping-based mis-assembly detection) — https://github.com/mobinasri/flagger

## Examples

```
python volcanosv-vc-large-indel-otherasm.py --bam_file Hifi_L6.bam --assembly chm13_diploid.fa --reference chm13.fa --chrom_list chrom_list.txt -o fdr_volcanosv/
```

## Evaluation signals

- VolcanoSV-vc should exhibit the lowest false discovery rate among VolcanoSV, PAV, SVIM-asm, and Dipcall
- Each tool's FDR equals (number of SVs detected against T2T-CHM13) / (number of SVs detected against GRCh38) — verify the arithmetic per tool
- All diploid assembly contig names carry hp1/hp2 haplotype tags and match the BAM and --chrom_list naming exactly
- Centromere/telomere overlap counts should be non-negative integers summing to at most the number of false-discovery calls per tool
- false_discovery_rate_table.tsv contains one row per tool with FDR plus censat/telomere overlap columns

## Limitations

- VolcanoSV-asm is not used for the CHM13 diploid assembly because the sample derives from a single homozygous complete hydatidiform mole and is unsuitable for haplotype phasing
- VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different
- Reference choice materially changes call counts: SVs detected against T2T-CHM13 include genuine sequence absent from GRCh38, so raw counts are not comparable without the dual-reference ratio design
- Dipcall must be excluded from complex SV analyses since it was not designed to detect complex SVs

## Evidence

- [intro] VolcanoSV-vc, the assembly-based SV calling component, has a low false discovery rate: "VolcanoSV-vc, the assembly-based SV calling component, has a low false discovery rate"
- [results] FDR definition: "The false discovery rate was computed as the ratio of the number of detected SVs against CHM13 to the number of SVs against GRCh38"
- [results] CHM13 datasets: "CHM13 PacBio CCS SRR1129212Hifi_L633×"
- [results] homozygous caveat: "We did not use the VolcanoSV-asm component for the diploid assembly since the CHM13 sample derives from a single homozygous complete hydatidiform mole"
- [discussion] censat BED download: "we downloaded two BED files from https://s3-us-west-2.amazonaws.com/human-pangenomics/T2T/CHM13/assemblies/annotation/chm13v2.0_censat_v2.0.bed"
- [readme] chrom_list requirement: "you must supply a `--chrom_list`/`-cl` (or `-chl`) text file listing one chromosome/contig name per line, exactly as they appear in your reference"
- [results] Dipcall exclusion: "Dipcall was not involved in this analysis since it was not designed to detect complex SVs"
