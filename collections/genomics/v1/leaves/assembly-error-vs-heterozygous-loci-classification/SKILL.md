---
name: assembly-error-vs-heterozygous-loci-classification
description: Use when evaluating a de novo genome assembly (e.g. HiCanu contigs of
  a heterozygous F1 hybrid) where clipped alignment signals must be split into true
  misassemblies versus haplotype differences before computing assembly quality indexes.
when_to_use_negative:
- Genome assemblies of species with higher ploidy levels than diploid, where CRAQ
  evaluation 'may not be as accurate as the benchmarked cases presented here'
- Regions where clipping ratio falls between the h upper bound and f cutoff (0.6–0.75),
  which CRAQ reports only as ambiguous heterozygous-or-error regions without a definitive
  classification
- When the goal is assembly completeness (BUSCO-style gene completeness) rather than
  base-level error detection — CRAQ targets regional and structural errors from clipping
  signals
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: CRAQ
  role: Core tool that extracts clipped alignments (CIGAR S/H), classifies loci as
    CREs/CSEs vs CRHs/CSHs via clipping-ratio and coverage cutoffs, and computes R-AQI/S-AQI
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Minimap2
  role: Maps NGS short reads ('-ax sr') and PacBio HiFi long reads ('-ax map-hifi')
    back to the assembly to generate the clipping signals CRAQ consumes
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Filters BAM alignments with '-F 1796 -q 20' to remove low-MAPQ, unmapped,
    secondary, QC-failed, and duplicate reads before CRAQ analysis
- name: Integrative Genomics Viewer (IGV)
  role: Visual inspection of filtered alignments at classified breakpoints to confirm
    error vs heterozygous calls
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:b3b20d994b63f8c7ebacac552d042985b546699f1dedd030918f3c07c4a07cb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - Integrative Genomics Viewer (IGV)
  - Minimap2
  - Perl
  - SAMtools
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# assembly-error-vs-heterozygous-loci-classification

## Summary

Classifies clipped-read loci from NGS/SMS read mappings back to a draft assembly into true assembly errors (CREs, CSEs) versus heterozygous variants (CRHs, CSHs) using clipping ratios and coverage, and converts error counts into R-AQI and S-AQI quality scores.

## When to use

Use when evaluating a de novo genome assembly (e.g. HiCanu contigs of a heterozygous F1 hybrid) where clipped alignment signals must be split into true misassemblies versus haplotype differences before computing assembly quality indexes.

## When NOT to use

- Genome assemblies of species with higher ploidy levels than diploid, where CRAQ evaluation 'may not be as accurate as the benchmarked cases presented here'
- Regions where clipping ratio falls between the h upper bound and f cutoff (0.6–0.75), which CRAQ reports only as ambiguous heterozygous-or-error regions without a definitive classification
- When the goal is assembly completeness (BUSCO-style gene completeness) rather than base-level error detection — CRAQ targets regional and structural errors from clipping signals

## Inputs

- Draft genome assembly FASTA (e.g. HiCanu D. melanogaster F1 hybrid assembly)
- Illumina short-read sequencing data (FASTQ)
- SMS long-read data (e.g. PacBio HiFi FASTQ)
- Minimap2-sorted BAM alignments for NGS and SMS reads, filtered with 'samtools view -F 1796 -q 20'

## Outputs

- out_final.Report with counts of CREs, CSEs, CRHs, CSHs and whole-genome R-AQI and S-AQI values
- Coordinates of regional errors (CREs), structural errors (CSEs), regional heterozygous variants (CRHs), and structural heterozygous variants (CSHs)
- Percentage of clipped loci classified as true errors vs heterozygous loci (e.g. 4.2%, 127/3060, for HiCanu D. melanogaster F1)

## How to apply

Map Illumina short reads and PacBio HiFi long reads to the assembly with Minimap2 ('-ax sr' and '-ax map-hifi' presets), then sort/index with SAMtools and filter with 'samtools view -F 1796 -q 20' to drop unmapped, secondary, QC-failed, and PCR-duplicate alignments with MAPQ <20. Run CRAQ on the assembly with the two filtered BAMs under default parameters: heterozygous loci are called when the clipping ratio falls in the cutoff region h (default 0.4–0.6), mapping breakpoints when the ratio exceeds f (default 0.75), and candidates with coverage below m (2) or above M (5x average coverage) are excluded. CRAQ then classifies errors as CREs (SMS-spanned NGS breakpoints with irregular coverage) or CSEs (SMS clipping breakpoints near NGS breakpoints/gaps), computes R-AQI and S-AQI from window-normalized error counts (CSE counts penalized 10x), and reports the counts in out_final.Report. The rationale is that haplotype divergence in F1 assemblies produces balanced clipping ratios and normal coverage, whereas true errors produce coverage irregularities and conflicting NGS/SMS breakpoints, allowing the 0.4–0.6 versus >0.75 ratio windows to separate the two classes.

## Related tools

- **CRAQ** (Core tool that extracts clipped alignments (CIGAR S/H), classifies loci as CREs/CSEs vs CRHs/CSHs via clipping-ratio and coverage cutoffs, and computes R-AQI/S-AQI) — https://github.com/JiaoLaboratory/CRAQ
- **Minimap2** (Maps NGS short reads ('-ax sr') and PacBio HiFi long reads ('-ax map-hifi') back to the assembly to generate the clipping signals CRAQ consumes) — https://github.com/lh3/minimap2
- **SAMtools** (Filters BAM alignments with '-F 1796 -q 20' to remove low-MAPQ, unmapped, secondary, QC-failed, and duplicate reads before CRAQ analysis)
- **Integrative Genomics Viewer (IGV)** (Visual inspection of filtered alignments at classified breakpoints to confirm error vs heterozygous calls)

## Examples

```
minimap2 -ax sr assembly.fa illumina.fq.gz | samtools sort -o ngs.bam; minimap2 -ax map-hifi assembly.fa hifi.fq.gz | samtools sort -o sms.bam; samtools view -F 1796 -q 20 -b ngs.bam > ngs.filtered.bam; samtools view -F 1796 -q 20 -b sms.bam > sms.filtered.bam; perl CRAQ.pl -a assembly.fa -n ngs.filtered.bam -s sms.filtered.bam
```

## Evaluation signals

- For the HiCanu D. melanogaster F1 assembly, the report should show ~4.2% (127/3060) of clipped loci called true errors: 102 CREs and 25 CSEs
- Whole-genome R-AQI should reproduce the Table 2 HiCanu value (~93.1) and S-AQI (~91.9), placing the assembly in the AQI > 90 'reference quality' band
- CRH/CSH counts should dominate over CRE/CSE counts in a heterozygous F1 assembly, confirming that most clipped loci are haplotype differences rather than errors
- Sanity-check that filtered BAMs contain no alignments with MAPQ <20, secondary, QC-fail, or duplicate flags after the '-F 1796 -q 20' filter
- Spot-check a sample of CRE/CSE and CRH/CSH coordinates in IGV: error breakpoints should show irregular coverage, heterozygous loci balanced clipping ratios (0.4–0.6)

## Limitations

- Evaluation for polyploid species 'may not be as accurate as the benchmarked cases presented here'
- Regions with clipping ratio between the h upper level and f (0.6–0.75) are reported as ambiguous heterozygous-or-error without definitive classification
- CSE/CRE classification depends on the user-tunable coverage-discrepancy parameter '-d', for which no universal value is stated
- Error breakpoint ratios can fall below 100% in practice due to sequencing errors or inaccurate read mapping
- Coverage filters exclude candidates below m (2) or above M (5x average coverage), so loci with extreme coverage are not classified

## Evidence

- [discussion] CRAQ distinguishes error breakpoints and heterozygous loci based on read-mapping coverage data and effective clipping ratios.: "CRAQ distinguishes error breakpoints and heterozygous loci based on read-mapping coverage data and effective clipping ratios."
- [discussion] heterozygous cutoff region h default 0.4-0.6: "a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6)"
- [discussion] filtered out using the '-F 1796 -q 20' parameters in 'samtools view': "filtered out using the ‘-F 1796 -q 20’ parameters in ‘samtools view’35"
- [abstract] AQI classification thresholds from abstract: "AQI > 90, reference quality; AQI from 80-90, high quality; AQI from 60-80, draft quality; and AQI < 60, low quality"
- [abstract] CRAQ outputs CRH and CSH coordinates based on clipped alignment ratio and coverage: "It can output coordinates of regional heterozygous variants (CRHs) and coordinates of structural heterozygous variants (CSHs) based on the ratio of clipped alignments and mapping coverage"
- [discussion] excludes candidates with extremely low coverage and high coverage: "excludes candidates with extremely low coverage (m, default = 2) and high coverage (M, default = 5 * average coverage)"
