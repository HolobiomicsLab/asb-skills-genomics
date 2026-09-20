---
name: clipped-read-alignment-analysis
description: Use when a draft (non-reference) genome assembly must be assessed or
  corrected for regional and structural errors using raw reads rather than a reference
  genome, especially when heterozygosity may confound error detection.
when_to_use_negative:
- The input is a finished reference-quality genome where AQI > 90 is already expected
  and error discovery would return mostly noise
- The goal is k-mer-based or completeness-only assessment (e.g., Merqury, KAT, BUSCO)
  rather than pinpointing error coordinates — CRAQ requires raw read mapping, which
  is its most resource-intensive step
- Genomes with higher ploidy levels (e.g., polyploids), where accuracy may not match
  the diploid/haploid benchmarked cases
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: CRAQ
  role: 'Core tool: extracts clipped reads (CIGAR S/H) from filtered BAMs and calls
    CRE/CSE breakpoints and CRH/CSH heterozygous loci from clipping ratios and coverage;
    computes AQI scores'
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Minimap2
  role: Maps raw NGS short reads ('-ax sr') and SMS long reads ('-ax map-hifi'/'map-pb'/'map-ont')
    back to the draft assembly to produce clipped alignments
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Filters alignments with 'samtools view -F 1796 -q 20' before clipped-read
    extraction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:5b29331488911e3d86681202461cdc5fd5ff166de404ffbaeb91f98a7f1c168a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - Minimap2
  - SAMtools
  - SyRI
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Clipped-read alignment analysis

## Summary

Detects assembly errors and heterozygous loci in draft genome assemblies at single-nucleotide resolution by extracting soft/hard-clipped alignments (CIGAR 'S'/'H') from NGS short-read and SMS long-read mappings back to the assembly, then classifying loci as CREs, CSEs, CRHs, or CSHs via clipping-ratio and coverage cutoffs (CRAQ logic).

## When to use

Use when a draft (non-reference) genome assembly must be assessed or corrected for regional and structural errors using raw reads rather than a reference genome, especially when heterozygosity may confound error detection.

## When NOT to use

- The input is a finished reference-quality genome where AQI > 90 is already expected and error discovery would return mostly noise
- The goal is k-mer-based or completeness-only assessment (e.g., Merqury, KAT, BUSCO) rather than pinpointing error coordinates — CRAQ requires raw read mapping, which is its most resource-intensive step
- Genomes with higher ploidy levels (e.g., polyploids), where accuracy may not match the diploid/haploid benchmarked cases

## Inputs

- Draft genome assembly FASTA (query; e.g., S. pennellii Canu assembly, A. oxysepala draft contigs)
- NGS short reads (FASTQ) mapped with Minimap2 '-ax sr'
- SMS long reads (FASTQ; ONT/PacBio HiFi/CLR) mapped with Minimap2 '-ax map-ont'/'-ax map-hifi'/'-ax map-pb'
- Filtered BAM alignment files (samtools '-F 1796 -q 20')

## Outputs

- CRE coordinates (small-scale Clip-based Regional Errors)
- CSE coordinates (large-scale Clip-based Structural Errors / misjoin breakpoints)
- CRH and CSH coordinates (regional and structural heterozygous variants)
- R-AQI and S-AQI assembly quality indicator scores and quality classification
- Optionally corrected FASTA with conflicting contigs split at CSE breakpoints (e.g., A.oxysepala_contig_CRAQcorrect.fa)

## How to apply

Map NGS short reads and SMS long reads to the draft assembly with Minimap2 using '-ax sr' for short reads and '-ax map-hifi', '-ax map-pb', or '-ax map-ont' depending on long-read chemistry, then filter with 'samtools view -F 1796 -q 20' to remove MAPQ<20, unmapped, secondary, QC-failed, and PCR-duplicated alignments. CRAQ extracts all clipped reads coded as 'S' or 'H' in the CIGAR string and computes clipping ratios with mapping coverage: a locus is classified as heterozygous (CRH/CSH) when the clipping ratio falls within h (default 0.4–0.6), while a region becomes a mapping breakpoint (CRE or CSE) when the ratio exceeds the stringent cutoff f (default 0.75); candidates with coverage below m (default 2) or above M (default 5× average coverage) are excluded. Multi-mapped reads are filtered by default when identifying CSE breakpoints, and coverage discrepancies between SMS read pairs are tuned with the '-d' parameter. Counts are normalized into R-AQI and S-AQI scores and binned (AQI > 90 reference quality, 80–90 high quality, 60–80 draft quality, < 60 low quality); conflicts can optionally be split at breakpoints for Hi-C or optical-map repair. As validation, on the S. pennellii Canu assembly CRAQ called 8029 breakpoints (7910 CREs, 119 CSEs), of which ~71.4% (5736/8029) overlapped SyRI SVs.

## Related tools

- **CRAQ** (Core tool: extracts clipped reads (CIGAR S/H) from filtered BAMs and calls CRE/CSE breakpoints and CRH/CSH heterozygous loci from clipping ratios and coverage; computes AQI scores) — https://github.com/JiaoLaboratory/CRAQ
- **Minimap2** (Maps raw NGS short reads ('-ax sr') and SMS long reads ('-ax map-hifi'/'map-pb'/'map-ont') back to the draft assembly to produce clipped alignments) — https://github.com/lh3/minimap2
- **SAMtools** (Filters alignments with 'samtools view -F 1796 -q 20' before clipped-read extraction)

## Examples

```
minimap2 -ax map-ont assembly.fa reads.fq.gz | samtools view -F 1796 -q 20 -bS > ont.filtered.bam
```

## Evaluation signals

- Filtered BAM files contain no alignments with MAPQ < 20, unmapped, secondary, QC-failed, or PCR-duplicated flags ('-F 1796 -q 20')
- Clipping-ratio classification obeys the cutoffs: ratio in 0.4–0.6 flagged as heterozygous (CRH/CSH), ratio > 0.75 flagged as error breakpoint (CRE/CSE)
- Coverage filters exclude candidates below m = 2 and above M = 5× average coverage before calling
- Normalized CRE/CSE counts yield R-AQI and S-AQI scores within the published classification bands (AQI > 90 reference quality; 80–90 high; 60–80 draft; < 60 low)
- On the S. pennellii Canu assembly the expected output is 8029 breakpoints (7910 CREs, 119 CSEs) with ~71.4% (5736/8029) overlapping SyRI SVs

## Limitations

- Regions with coverage above the upper bound of h but below f are reported as ambiguous heterozygous-or-error regions without a definitive classification
- The CSE/CRE classification depends on the user-tunable coverage-discrepancy parameter '-d', for which no universal value is stated
- The ratio for assembly errors can fall below 100% in practice due to sequencing errors or inaccurate read mapping
- Read mapping is the most resource-intensive step; users must manually split query sequences into fragments to parallelize

## Evidence

- [discussion] CRAQ first extracts all clipped reads, coded as “S” or “H” in the Compact Idiosyncratic Gapped Alignment Report (CIGAR) string: "CRAQ first extracts all clipped reads, coded as “S” or “H” in the Compact Idiosyncratic Gapped Alignment Report (CIGAR) string"
- [discussion] heterozygous clipping ratio cutoff: "a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6)"
- [discussion] breakpoint clipping ratio cutoff: "A region is classified as a mapping breakpoint when the ratio exceeds a stringent cutoff value f (default = 0.75)"
- [discussion] alignment filtering parameters: "filtered out using the ‘-F 1796 -q 20’ parameters in ‘samtools view’"
- [readme] CRAQ output description: "It can report precise locations of small-scale Clip-based Regional Errors (CREs), large-scale Clip-based Structural Errors (CSEs), as well as overall AQI metrics"
- [discussion] coverage filter thresholds: "excludes candidates with extremely low coverage (m, default = 2) and high coverage (M, default = 5 * average coverage)"
