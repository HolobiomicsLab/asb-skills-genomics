---
name: assembly-quality-index-calculation
description: Use when a draft genome assembly needs a numeric regional/structural
  quality score (R-AQI/S-AQI) based on clipped-read error counts, or when heterozygous
  loci must be separated from true assembly errors.
when_to_use_negative:
- Polyploid genome assemblies — the paper states evaluation for species with higher
  ploidy levels may not be as accurate as the benchmarked diploid cases
- Input is already a finished/reference assembly without raw reads available for re-mapping,
  since CRAQ is reference-free and requires mapping original sequencing reads back
  to the draft
- The goal is completeness rather than accuracy assessment — use BUSCO instead, which
  the paper identifies as the state-of-the-method for completeness evaluation
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: CRAQ
  role: Extracts clipped alignments from filtered BAMs, classifies CRE/CSE errors
    vs CRH/CSH heterozygous loci, and transforms normalized error counts into R-AQI
    and S-AQI scores
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Minimap2
  role: Maps NGS short reads (-ax sr) and SMS long reads (-ax map-hifi/-map-pb/-map-ont)
    back to the assembly to generate clipping signals
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Filters alignments by flag and MAPQ ('-F 1796 -q 20') before CRAQ processing
- name: Integrative Genomics Viewer (IGV)
  role: Visual inspection of filtered alignments at candidate error breakpoints
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:46f7457c082c5c9f7de72c45d26262f3b85fb3029e89100b1d13f20360238695
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
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# assembly-quality-index-calculation

## Summary

Computes CRAQ's R-AQI and S-AQI assembly quality indexes by normalizing CRE and CSE error counts identified from clipped-alignment signals of NGS short-read and SMS long-read mappings, yielding a single-nucleotide-resolution quality score that classifies assemblies as reference (>90), high (80-90), draft (60-80), or low (<60) quality.

## When to use

Use when a draft genome assembly needs a numeric regional/structural quality score (R-AQI/S-AQI) based on clipped-read error counts, or when heterozygous loci must be separated from true assembly errors.

## When NOT to use

- Polyploid genome assemblies — the paper states evaluation for species with higher ploidy levels may not be as accurate as the benchmarked diploid cases
- Input is already a finished/reference assembly without raw reads available for re-mapping, since CRAQ is reference-free and requires mapping original sequencing reads back to the draft
- The goal is completeness rather than accuracy assessment — use BUSCO instead, which the paper identifies as the state-of-the-method for completeness evaluation

## Inputs

- Draft/de novo genome assembly in FASTA format
- Illumina NGS short-read dataset (FASTQ)
- SMS long-read dataset (PacBio HiFi, PacBio CLR, or ONT; FASTQ/BAM)
- Minimap2 SAM/BAM alignments filtered with 'samtools view -F 1796 -q 20'

## Outputs

- out_final.Report with CRE, CSE, CRH, and CSH counts
- Whole-genome R-AQI and S-AQI scores (0-100 scale)
- Coordinates of regional errors (CREs), structural errors (CSEs), and heterozygous variants (CRHs, CSHs)
- Quality class assignment: AQI > 90 reference, 80-90 high, 60-80 draft, < 60 low quality

## How to apply

Map Illumina short reads to the assembly with Minimap2 '-ax sr' and long reads (e.g. PacBio HiFi) with '-ax map-hifi', then sort and index the BAMs with SAMtools. Filter alignments with 'samtools view -F 1796 -q 20' to drop unmapped, secondary, QC-failed, and PCR-duplicate reads with MAPQ < 20, then run CRAQ with default parameters: heterozygous loci are called when the clipping ratio falls in the h cutoff region (0.4-0.6), mapping breakpoints when the ratio exceeds f (0.75), and candidates with coverage below m=2 or above M=5x average coverage are excluded. CRAQ classifies errors as CREs (SMS-spanned NGS breakpoints with irregular coverage) or CSEs (SMS clipping breakpoints near NGS breakpoints/gaps) and reports CRH/CSH heterozygous loci separately. Parse out_final.Report to obtain CRE, CSE, CRH, and CSH counts and the whole-genome R-AQI and S-AQI, which are window-normalized scores (Nw = sum of 1/i over m errors per 0.0001*genome-size window, CSE counts penalized 10x).

## Related tools

- **CRAQ** (Extracts clipped alignments from filtered BAMs, classifies CRE/CSE errors vs CRH/CSH heterozygous loci, and transforms normalized error counts into R-AQI and S-AQI scores) — https://github.com/JiaoLaboratory/CRAQ
- **Minimap2** (Maps NGS short reads (-ax sr) and SMS long reads (-ax map-hifi/-map-pb/-map-ont) back to the assembly to generate clipping signals) — https://github.com/lh3/minimap2
- **SAMtools** (Filters alignments by flag and MAPQ ('-F 1796 -q 20') before CRAQ processing)
- **Integrative Genomics Viewer (IGV)** (Visual inspection of filtered alignments at candidate error breakpoints)

## Examples

```
minimap2 -ax sr assembly.fa illumina.fq.gz | samtools view -F 1796 -q 20 -b > sr.bam; minimap2 -ax map-hifi assembly.fa hifi.fq.gz | samtools view -F 1796 -q 20 -b > sms.bam
```

## Evaluation signals

- Reproduce the D. melanogaster F1 HiCanu benchmark: R-AQI ~93.1 and S-AQI ~91.9 with only 4.2% (127/3060) of clipped loci called true errors (102 CREs + 25 CSEs)
- R-AQI and S-AQI both fall in 0-100 and, for reference-quality assemblies, exceed 90 per the CRAQ classification thresholds
- out_final.Report contains non-negative counts of CREs, CSEs, CRHs, and CSHs summing consistently with the reported total clipped-loci count
- Clipping-ratio-based classification behaves as specified: heterozygous loci in the 0.4-0.6 h region, breakpoints above the 0.75 f cutoff
- CRH/CSH heterozygous counts are reported separately from CRE/CSE error counts, confirming haplotype features were not counted as misassemblies

## Limitations

- Ambiguous regions with coverage above h and below f are reported as 'ambiguous heterozygous or error region' without definitive classification
- CSE/CRE classification depends on the user-tunable coverage-discrepancy parameter (-d), for which no universal value is stated
- Error breakpoint ratios can fall below 100% in practice due to sequencing errors or inaccurate read mapping
- Read mapping is the most resource-intensive step and users must manually split query sequences into fragments to parallelize

## Evidence

- [discussion] The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores, respectively.: "The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores, respectively."
- [abstract] AQI > 90, reference quality; AQI from 80-90, high quality; AQI from 60-80, draft quality; and AQI < 60, low quality: "AQI > 90, reference quality; AQI from 80-90, high quality; AQI from 60-80, draft quality; and AQI < 60, low quality"
- [discussion] filtered out using the ‘-F 1796 -q 20’ parameters in ‘samtools view’: "filtered out using the ‘-F 1796 -q 20’ parameters in ‘samtools view’"
- [discussion] a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6): "a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6)"
- [discussion] excludes candidates with extremely low coverage (m, default = 2) and high coverage (M, default = 5 * average coverage): "excludes candidates with extremely low coverage (m, default = 2) and high coverage (M, default = 5 * average coverage)"
- [readme] It can output coordinates of regional heterozygous variants (CRHs) and coordinates of structural heterozygous variants (CSHs) based on the ratio of clipped: "It can output coordinates of regional heterozygous variants (CRHs) and coordinates of structural heterozygous variants (CSHs) based on the ratio of clipped"
