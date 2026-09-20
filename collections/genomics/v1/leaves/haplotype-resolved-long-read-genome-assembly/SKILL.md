---
name: haplotype-resolved-long-read-genome-assembly
description: Use when you have a long-read BAM (PacBio HiFi/CLR or ONT) from a diploid
  sample and need chromosome-wise phased maternal/paternal contigs to enable haplotype-resolved
  SV discovery or downstream assembly-based variant benchmarking.
when_to_use_negative:
- Input reads are from sex chromosomes — VolcanoSV is designed only for autosomes
  because its phasing procedure for sex chromosomes differs fundamentally
- The sample is haploid/homozygous (e.g. CHM13 derived from a complete hydatidiform
  mole) — haplotype phasing is inapplicable, as the authors noted for CHM13
- You only have short-read (Illumina) data or already possess a finished phased assembly
  and only need SV calling from it — use VolcanoSV-vc directly or another caller instead
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: VolcanoSV
  role: Hybrid pipeline whose VolcanoSV-asm component performs reference-guided, phased-SNP-based
    diploid assembly per chromosome from long-read BAMs
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: hifiasm
  role: Optional integrated de novo assembler selectable via -asm for HiFi-based per-chromosome
    assembly
  repo: https://github.com/chhylp123/hifiasm
- name: Flye
  role: Optional integrated long-read assembler selectable via -asm (version 2.9.6)
    for single-molecule reads
  repo: https://github.com/fenderglass/Flye
- name: wtdbg2
  role: Optional integrated fuzzy-bruijn-graph assembler selectable via -asm for noisy
    long reads
  repo: https://github.com/ruanjue/wtdbg2
- name: HMM-Flagger
  role: Read-mapping-based HMM tool to QC the resulting assemblies for mis-assemblies
    via anomalous read-coverage partitioning
  repo: https://github.com/mobinasri/flagger
- name: Truvari
  role: Benchmarking tool to evaluate SV calls derived from the phased assembly against
    the GIAB HG002 gold standard (v4.0.0)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:46542c26ad9af7710106a19240d6272bb279a34b83cf555e5d4b440f65d98c87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - Dipcall
  - Flye
  - HMM-Flagger
  - PAV
  - SVIM-asm
  - Truvari
  - VolcanoSV
  - hifiasm
  - rasusa
  - wtdbg2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# haplotype-resolved-long-read-genome-assembly

## Summary

Generates a phased diploid assembly from single-molecule long reads (HiFi, CLR, or ONT) by combining reference-guided phasing of SNPs with local de novo assembly, per autosome, as implemented in VolcanoSV-asm. The phased assembly is the substrate for haplotype-resolved SV calling rather than a standalone assembly product.

## When to use

Use when you have a long-read BAM (PacBio HiFi/CLR or ONT) from a diploid sample and need chromosome-wise phased maternal/paternal contigs to enable haplotype-resolved SV discovery or downstream assembly-based variant benchmarking.

## When NOT to use

- Input reads are from sex chromosomes — VolcanoSV is designed only for autosomes because its phasing procedure for sex chromosomes differs fundamentally
- The sample is haploid/homozygous (e.g. CHM13 derived from a complete hydatidiform mole) — haplotype phasing is inapplicable, as the authors noted for CHM13
- You only have short-read (Illumina) data or already possess a finished phased assembly and only need SV calling from it — use VolcanoSV-vc directly or another caller instead

## Inputs

- Long-read BAM file aligned to a reference genome (PacBio HiFi, PacBio CLR, or ONT; e.g. HG002 Hifi_L1 56x, CLR_L1 89x, ONT_L1 48x)
- Reference genome FASTA plus .fai index (e.g. hg19 genome.fa from the Zenodo test data or GRCh38)
- chrom_list.txt listing one autosome/contig name per line, matching reference and BAM naming (WGS mode)
- Choice of assembler via -asm (wtdbg2, canu, miniasm, shasta, nextdenovo, hifiasm, hicanu, flye)

## Outputs

- Phased diploid assembly contigs per chromosome (maternal and paternal haplotypes)
- Intermediate assembly files (e.g. contigs/GFA outputs from the selected assembler) under the specified output directory
- Downstream phased, haplotype-resolved SV VCFs (DEL, INS, INV, TRA, DUP, small indel, SNP) when passed to the VolcanoSV-vc components

## How to apply

Run VolcanoSV-asm per chromosome (it is designed to be executed chromosome by chromosome, e.g. chr10 first as a test) using `${path_to_volcanosv}/bin/VolcanoSV-asm/volcanosv-asm.py` with the BAM, reference FASTA, thread count, chromosome name (-chr), and the platform-appropriate -dtype (Hifi/CLR/ONT). VolcanoSV integrates multiple assemblers selectable via -asm (wtdbg2, canu/hicanu, miniasm, shasta, nextdenovo, hifiasm, flye), so pick the assembler suited to the read technology and depth. Internally the pipeline uses phased SNPs and unique k-mer similarity analysis to assign contigs to haplotypes and produce a phased diploid assembly. Process autosomes only, and supply a chrom_list.txt (one contig name per line matching the reference) when switching to WGS mode; the phased assembly then feeds volcanosv-vc-large-indel.py, volcanosv-vc-complex-sv.py, and volcanosv-vc-small-indel.py. For regions enriched in segmental duplications, apply the optional assembly-improvement step described in the README before variant calling.

## Related tools

- **VolcanoSV** (Hybrid pipeline whose VolcanoSV-asm component performs reference-guided, phased-SNP-based diploid assembly per chromosome from long-read BAMs) — https://github.com/maiziezhoulab/VolcanoSV
- **hifiasm** (Optional integrated de novo assembler selectable via -asm for HiFi-based per-chromosome assembly) — https://github.com/chhylp123/hifiasm
- **Flye** (Optional integrated long-read assembler selectable via -asm (version 2.9.6) for single-molecule reads) — https://github.com/fenderglass/Flye
- **wtdbg2** (Optional integrated fuzzy-bruijn-graph assembler selectable via -asm for noisy long reads) — https://github.com/ruanjue/wtdbg2
- **HMM-Flagger** (Read-mapping-based HMM tool to QC the resulting assemblies for mis-assemblies via anomalous read-coverage partitioning) — https://github.com/mobinasri/flagger
- **Truvari** (Benchmarking tool to evaluate SV calls derived from the phased assembly against the GIAB HG002 gold standard (v4.0.0))

## Examples

```
python ${path_to_volcanosv}/bin/VolcanoSV-asm/volcanosv-asm.py -bam chr10.Hifi.bam -ref genome.fa -o chr10_out -t 16 -chr chr10 -dtype Hifi
```

## Evaluation signals

- SVs called from the phased assembly match the GIAB HG002 gold standard at high Truvari scores (VolcanoSV reported average F1 of 91.03% insertions / 94.19% deletions and genotyping accuracy 98.32% / 99.01% across HiFi datasets)
- Low false discovery rate confirmed by comparing detected SVs against T2T-CHM13 versus GRCh38 (FDR = ratio of SVs detected against CHM13 to SVs against GRCh38)
- Assembly succeeds only for autosomes — any attempt on chrX/chrY should be rejected by the pipeline or excluded by chrom_list.txt
- SV calls retain high F1 after subsampling to 40x, 30x, 20x, and even 10x coverage with rasusa v0.6.0, indicating the phased assembly did not collapse at reduced depth
- Haplotype phasing is coherent: unique k-mer similarity analysis assigns each contig to one parental haplotype, and downstream VCFs carry phased GT fields

## Limitations

- VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different
- Assembly algorithms prefer the simplest paths through overlapping reads, which may miss complex SVs
- Assembly-based small-indel detection was ineffective for ONT data (low precision for all three tools evaluated)
- At very low coverage (5x subsampling) performance degrades relative to 10x and above
- Translocation (complex SV) detection requires WGS BAM support and cannot be run in single-chromosome mode

## Evidence

- [abstract] hybrid pipeline with local de novo assembly: "VolcanoSV, an innovative hybrid SV detection pipeline that utilizes both a reference genome and local de novo assembly to generate a phased diploid assembly."
- [abstract] phased SNPs and k-mer similarity for haplotype resolution: "VolcanoSV uses phased SNPs and unique k-mer similarity analysis, enabling precise haplotype-resolved SV discovery"
- [readme] autosome-only design: "VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different!"
- [readme] integrated assemblers: "We integrated multiple state-of-the-art assemblers into the pipeline, including"
- [results] rasusa subsampling for robustness testing: "All three datasets were subsampled to 40x, 30x, 20x, 10x, and 5x coverage using rasusa (v0.6.0)37"
- [results] FDR evaluation strategy: "The false discovery rate was computed as the ratio of the number of detected SVs against CHM13 to the number of SVs against GRCh38"
