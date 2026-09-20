---
name: haplotype-resolved-assembly-based-sv-calling
description: Use when calling haplotype-resolved SVs (including complex translocations,
  duplications, and inversions) from PacBio HiFi/CLR or ONT long-read WGS BAMs, especially
  when alignment-based callers underperform or low-coverage (10x) data must be handled.
when_to_use_negative:
- Calling SVs on sex (X/Y) chromosomes — VolcanoSV is designed only for autosome chromosomes
  since the phasing procedure for sex chromosomes is very different
- Using the VolcanoSV-asm component on homozygous samples such as CHM13, which derives
  from a single homozygous complete hydatidiform mole and is unsuitable for haplotype
  phasing
- Detecting small indels from ONT data via assembly — assembly-based small-indel detection
  proved ineffective for ONT data with low precision for all tools
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: VolcanoSV
  role: 'Hybrid SV detection pipeline: reference + local de novo assembly to generate
    a phased diploid assembly and call DEL, INS, DUP, INV, TRA, small indels, and
    SNPs'
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: hifiasm
  role: One of the integrated assemblers selectable in VolcanoSV-asm for local chromosome-level
    de novo assembly
  repo: https://github.com/chhylp123/hifiasm
- name: Flye
  role: One of the integrated assemblers selectable in VolcanoSV-asm for long-read
    assembly
  repo: https://github.com/fenderglass/Flye
- name: wtdbg2
  role: One of the integrated assemblers selectable in VolcanoSV-asm
  repo: https://github.com/ruanjue/wtdbg2
- name: SVIM-asm
  role: Assembly-based comparison caller for benchmarking complex SV detection performance
  repo: https://github.com/eldariont/svim-asm
- name: Truvari
  role: Benchmarks SV calls against the GIAB gold standard with parameters p=0.5,
    P=0.5, r=500, O=0.01
- name: NanoVar
  role: Alignment-based comparison caller benchmarked against VolcanoSV
- name: cuteSV
  role: Alignment-based comparison caller benchmarked against VolcanoSV
- name: pbsv
  role: Alignment-based comparison caller benchmarked against VolcanoSV
- name: Sniffles2
  role: Alignment-based comparison caller benchmarked against VolcanoSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:bb45dfbf5d1a443d3528c9175fd25e2edb632775b17f87e950c12728e1e2ccb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - DeBreak
  - Flye
  - MAMnet
  - NanoVar
  - PBSIM
  - SURVIVOR
  - SVDSS
  - SVIM-asm
  - Sniffles2
  - Truvari
  - VISOR
  - VolcanoSV
  - cuteSV
  - hifiasm
  - pbsv
  - wtdbg2
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# haplotype-resolved-assembly-based-sv-calling

## Summary

Reproduces VolcanoSV's hybrid SV detection workflow, which combines reference-based read alignment with local de novo assembly to build a phased diploid assembly and call haplotype-resolved structural variants (DEL, INS, DUP, INV, TRA) from long-read sequencing data.

## When to use

Use when calling haplotype-resolved SVs (including complex translocations, duplications, and inversions) from PacBio HiFi/CLR or ONT long-read WGS BAMs, especially when alignment-based callers underperform or low-coverage (10x) data must be handled.

## When NOT to use

- Calling SVs on sex (X/Y) chromosomes — VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different
- Using the VolcanoSV-asm component on homozygous samples such as CHM13, which derives from a single homozygous complete hydatidiform mole and is unsuitable for haplotype phasing
- Detecting small indels from ONT data via assembly — assembly-based small-indel detection proved ineffective for ONT data with low precision for all tools

## Inputs

- Aligned long-read WGS BAM file (PacBio HiFi, PacBio CLR, or ONT)
- Reference genome FASTA with .fai index (e.g., hg19 genome.fa)
- chrom_list.txt text file listing autosome names in WGS mode
- Test data (chr10 BAM, contigs, VCF) from zenodo.org/records/10520476
- Truth/high-confidence SV callset (e.g., GIAB SV gold standard) for benchmarking

## Outputs

- Phased diploid assembly contigs per chromosome
- Large-indel SV VCF (deletions and insertions)
- Complex SV VCF (translocations, duplications, inversions) from volcanosv-vc-complex-sv.py
- SNP and small-indel VCFs from the VolcanoSV-vc components
- Truvari benchmarking metrics (recall, precision, F1, GT_F1)

## How to apply

Run the hybrid pipeline per chromosome (autosomes only): first volcanosv-asm.py to generate a phased diploid assembly from a reference genome plus local de novo assembly using one of the integrated assemblers (hifiasm, Flye, wtdbg2, miniasm, Shasta, NextDenovo, HiCanu/Canu), leveraging phased SNPs and unique k-mer similarity analysis for haplotype-resolved discovery; then volcanosv-vc-large-indel.py with read-based signature filtering and stringent supporting-signature-depth filtering (e.g., Hifi DEL bounds lb=0.2, rb=2.6), followed by volcanosv-vc-complex-sv.py whose large-indel VCF feeds the complex SV caller. In WGS mode, supply a -cl chrom_list.txt (one autosome per line) and a -px prefix; translocation detection requires a WGS BAM and is only supported in WGS mode. Benchmark the resulting VCF against a truth callset (e.g., GIAB SV gold standard) with Truvari (v4.0.0) using p=0.5, P=0.5, r=500, O=0.01, computing F1 and GT_F1 for INVs/DUPs and recall for translocations.

## Related tools

- **VolcanoSV** (Hybrid SV detection pipeline: reference + local de novo assembly to generate a phased diploid assembly and call DEL, INS, DUP, INV, TRA, small indels, and SNPs) — https://github.com/maiziezhoulab/VolcanoSV
- **hifiasm** (One of the integrated assemblers selectable in VolcanoSV-asm for local chromosome-level de novo assembly) — https://github.com/chhylp123/hifiasm
- **Flye** (One of the integrated assemblers selectable in VolcanoSV-asm for long-read assembly) — https://github.com/fenderglass/Flye
- **wtdbg2** (One of the integrated assemblers selectable in VolcanoSV-asm) — https://github.com/ruanjue/wtdbg2
- **SVIM-asm** (Assembly-based comparison caller for benchmarking complex SV detection performance) — https://github.com/eldariont/svim-asm
- **Truvari** (Benchmarks SV calls against the GIAB gold standard with parameters p=0.5, P=0.5, r=500, O=0.01)
- **NanoVar** (Alignment-based comparison caller benchmarked against VolcanoSV)
- **cuteSV** (Alignment-based comparison caller benchmarked against VolcanoSV)
- **pbsv** (Alignment-based comparison caller benchmarked against VolcanoSV)
- **Sniffles2** (Alignment-based comparison caller benchmarked against VolcanoSV)

## Examples

```
conda activate volcanosv; python ${path_to_volcanosv}/bin/VolcanoSV-asm/volcanosv-asm.py -bam wgs.bam -ref genome.fa -t 16 -chr chr10 -asm hifiasm -o out_dir
```

## Evaluation signals

- Across Hifi datasets VolcanoSV achieved the best average F1 of 91.03% (insertions) and 94.19% (deletions) with genotyping accuracy of 98.32% and 99.01%; across CLR datasets F1 of 89.72% and 93.70%
- For somatic complex SVs on HCC1395, VolcanoSV recall exceeds SVIM-asm across all libraries and complex SV types, with the largest advantage on duplications (12% higher on PacBio CLR, 17% higher on ONT)
- Truvari run completed with the specified parameter set (p=0.5, P=0.5, r=500, O=0.01; S=30 for complex SV benchmarking)
- Pipeline reproduces performance on the provided chr10 test data from Zenodo before WGS runs
- Each raw SV call retains at least one supporting signature; DEL calls pass the technology-specific empirical depth boundaries

## Limitations

- VolcanoSV is designed only for autosome chromosomes; the phasing procedure for sex chromosomes is very different
- VolcanoSV's recall for real complex somatic SVs was relatively lower compared to its performance on simulated data or indel SVs, and it failed to detect a sufficient number of unique somatic complex SVs in the HCC1395 tumor sample beyond germline SVs
- The HCC1395 high-confidence benchmark callset included SV calls only from alignment-based tools, potentially biasing evaluation against VolcanoSV
- Assembly algorithms that build contigs by following the simplest paths through overlapping reads may miss complex SVs
- Translocation detection requires a WGS BAM file as support and cannot be run at the single-chromosome level
