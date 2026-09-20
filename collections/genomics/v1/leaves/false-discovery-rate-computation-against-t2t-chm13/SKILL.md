---
name: false-discovery-rate-computation-against-t2t-chm13
description: Use when benchmarking assembly-based SV callers on CHM13 long reads against
  the T2T-CHM13 reference and quantifying how many calls are reference-mismatch false
  discoveries rather than true variants.
when_to_use_negative:
- Benchmarking against a labeled truth set such as the GIAB HG002 SV gold standard
  — use Truvari-based benchmarking (p=0.5, P=0.5, r=500, O=0.01) instead
- Calling complex SVs with Dipcall, which was excluded from complex SV analysis because
  it was not designed to detect them
- Running the VolcanoSV-asm phasing component on the CHM13 sample, since it derives
  from a single homozygous complete hydatidiform mole and is unsuitable for haplotype
  phasing
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: VolcanoSV
  role: SV caller under evaluation; its assembly-based component VolcanoSV-vc (volcanosv-vc-*-otherasm.py
    scripts) calls SVs from other assemblies against T2T-CHM13
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: Dipcall
  role: Comparison assembly-based SV caller run on the CHM13 diploid assembly against
    both GRCh38 and T2T-CHM13
  repo: https://github.com/lh3/dipcall
- name: SVIM-asm
  role: Comparison assembly-based SV caller run on the CHM13 diploid assembly against
    both GRCh38 and T2T-CHM13
  repo: https://github.com/eldariont/svim-asm
- name: PAV
  role: Comparison assembly-based SV caller run on the CHM13 diploid assembly against
    both GRCh38 and T2T-CHM13
  repo: https://github.com/EichlerLab/pav
- name: HMM-Flagger/Flagger
  role: Recommended for evaluating dual/diploid assemblies via read-mapping-based
    mis-assembly detection
  repo: https://github.com/mobinasri/flagger
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:9c3f6b8433706902b2e031491d3f1cbd1956740ae75986b60b6610be5788ad11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - Dipcall
  - HMM-Flagger/Flagger
  - PAV
  - SVIM-asm
  - VolcanoSV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# false-discovery-rate-computation-against-t2t-chm13

## Summary

Benchmark assembly-based SV callers (VolcanoSV-vc, Dipcall, SVIM-asm, PAV) by computing the false discovery rate as the ratio of SVs called by CHM13 long reads against the T2T-CHM13 reference versus against GRCh38, attributing excess calls to reference-mismatch artifacts in centromeric/telomeric sequence.

## When to use

Use when benchmarking assembly-based SV callers on CHM13 long reads against the T2T-CHM13 reference and quantifying how many calls are reference-mismatch false discoveries rather than true variants.

## When NOT to use

- Benchmarking against a labeled truth set such as the GIAB HG002 SV gold standard — use Truvari-based benchmarking (p=0.5, P=0.5, r=500, O=0.01) instead
- Calling complex SVs with Dipcall, which was excluded from complex SV analysis because it was not designed to detect them
- Running the VolcanoSV-asm phasing component on the CHM13 sample, since it derives from a single homozygous complete hydatidiform mole and is unsuitable for haplotype phasing

## Inputs

- CHM13 diploid assembly FASTA with hp1/hp2 haplotype-tagged contig names
- CHM13 PacBio HiFi BAM (Hifi_L6, SRR1129212, 33x; Hifi_L7, SRX5633451, 23.7x)
- T2T-CHM13 reference FASTA and GRCh38 reference FASTA
- --chrom_list text file listing one contig name per line
- chm13v2.0_censat_v2.0.bed (centromere satellite annotation)
- chm13v2.0_telomere.bed (telomere annotation)

## Outputs

- Per-tool false discovery rate values (ratio of SVs detected against CHM13 to SVs against GRCh38)
- Per-tool counts of false-discovery SVs overlapping centromeric and telomeric regions
- false_discovery_rate_table.tsv comparison table

## How to apply

Reformat the CHM13 diploid assembly FASTA so contig names carry 'hp1'/'hp2' haplotype tags, then run VolcanoSV-vc (volcanosv-vc-large-indel-otherasm.py and volcanosv-vc-complex-sv-otherasm.py) against T2T-CHM13 with the CHM13 PacBio HiFi BAMs (Hifi_L6, SRR1129212, 33x; Hifi_L7, SRX5633451, 23.7x), supplying a --chrom_list file listing one contig name per line exactly as in the reference and BAM. Run Dipcall, SVIM-asm, and PAV on the same CHM13 diploid assembly against both GRCh38 and T2T-CHM13. For each tool compute FDR as the ratio of the number of SVs detected against CHM13 to the number detected against GRCh38, and intersect the false-discovery calls with chm13v2.0_censat_v2.0.bed (centromere satellites) and chm13v2.0_telomere.bed to annotate where excess calls fall. Compile per-tool FDR plus centromere/telomere overlap counts into false_discovery_rate_table.tsv and confirm VolcanoSV-vc exhibits the lowest FDR among the compared tools.

## Related tools

- **VolcanoSV** (SV caller under evaluation; its assembly-based component VolcanoSV-vc (volcanosv-vc-*-otherasm.py scripts) calls SVs from other assemblies against T2T-CHM13) — https://github.com/maiziezhoulab/VolcanoSV
- **Dipcall** (Comparison assembly-based SV caller run on the CHM13 diploid assembly against both GRCh38 and T2T-CHM13) — https://github.com/lh3/dipcall
- **SVIM-asm** (Comparison assembly-based SV caller run on the CHM13 diploid assembly against both GRCh38 and T2T-CHM13) — https://github.com/eldariont/svim-asm
- **PAV** (Comparison assembly-based SV caller run on the CHM13 diploid assembly against both GRCh38 and T2T-CHM13) — https://github.com/EichlerLab/pav
- **HMM-Flagger/Flagger** (Recommended for evaluating dual/diploid assemblies via read-mapping-based mis-assembly detection) — https://github.com/mobinasri/flagger

## Examples

```
python volcanosv-vc-large-indel-otherasm.py -bam Hifi_L6_SRR1129212.bam -ref t2t-chm13.fa -o out_fdr -t 16 -cl chrom_list.txt
```

## Evaluation signals

- VolcanoSV-vc shows the lowest FDR among VolcanoSV, PAV, SVIM-asm, and Dipcall, consistent with the paper's reported finding
- Each FDR equals the count of SVs called against CHM13 divided by the count called against GRCh38 for the same tool and assembly input
- Excess (false-discovery) SVs predominantly intersect chm13v2.0_censat_v2.0.bed or chm13v2.0_telomere.bed intervals, explaining the reference-mismatch effect
- All contig names in --chrom_list, the reference FASTA, and the BAM headers match exactly (e.g. hp1/hp2-tagged CHM13 contigs), as required by VolcanoSV
- false_discovery_rate_table.tsv contains one FDR value per tool plus centromere/telomere overlap counts for all four callers

## Limitations

- FDR is a proxy metric: the ratio of CHM13-referenced to GRCh38-referenced calls assumes reference-mismatch artifacts concentrate in centromeric/telomeric sequence rather than being directly validated against a truth set
- VolcanoSV-asm cannot be used to build the CHM13 diploid assembly because the sample derives from a single homozygous complete hydatidiform mole
- VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different

## Evidence

- [results] The false discovery rate was computed as the ratio of the number of detected SVs against CHM13 to the number of SVs against GRCh38: "The false discovery rate was computed as the ratio of the number of detected SVs against CHM13 to the number of SVs against GRCh38"
- [intro] VolcanoSV-vc, the assembly-based SV calling component, has a low false discovery rate: "VolcanoSV-vc, the assembly-based SV calling component, has a low false discovery rate"
- [discussion] we downloaded two BED files from https://s3-us-west-2.amazonaws.com/human-pangenomics/T2T/CHM13/assemblies/annotation/chm13v2.0_censat_v2.0.bed: "we downloaded two BED files from https://s3-us-west-2.amazonaws.com/human-pangenomics/T2T/CHM13/assemblies/annotation/chm13v2.0_censat_v2.0.bed"
- [readme] you must supply a --chrom_list/-cl text file listing one chromosome/contig name per line, exactly as they appear in your reference: "you must supply a `--chrom_list`/`-cl` (or `-chl`) text file listing one chromosome/contig name per line, exactly as they appear in your reference"
- [results] We did not use the VolcanoSV-asm component for the diploid assembly since the CHM13 sample derives from a single homozygous complete hydatidiform mole: "We did not use the VolcanoSV-asm component for the diploid assembly since the CHM13 sample derives from a single homozygous complete hydatidiform mole"
