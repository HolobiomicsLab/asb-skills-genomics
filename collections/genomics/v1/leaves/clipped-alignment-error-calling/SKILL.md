---
name: clipped-alignment-error-calling
description: Use when a draft genome assembly must be assessed and its errors pinpointed
  reference-free, using raw NGS short reads and SMS long reads (HiFi/PacBio/ONT) mapped
  back to the assembly to expose clipping signals.
when_to_use_negative:
- Polyploid genome assemblies — the article warns 'evaluation of genome assemblies
  for species with higher ploidy levels may not be as accurate as the benchmarked
  cases presented here'.
- Cases requiring a definitive call for ambiguous loci — regions with clipping ratio
  above h but below f are reported as 'ambiguous heterozygous or error region' without
  classification.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: CRAQ
  role: Extracts clipped alignments and calls CRE/CSE errors, heterozygous CRH/CSH
    loci, and R-AQI/S-AQI scores from the filtered BAM files
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Minimap2
  role: Maps NGS short reads ('-ax sr') and SMS long reads ('-ax map-hifi/map-pb/map-ont')
    back to the draft assembly to generate clipping signals
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Filters alignments with 'samtools view -F 1796 -q 20' before clipped-read
    extraction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:95e361e43d4e71ff45742db4b85987982cb87e0662289c3bb37c4a000f1dd7b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - Inspector
  - Merqury
  - Minimap2
  - PBSIM
  - QUAST
  - SAMtools
  - Wgsim
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# clipped-alignment-error-calling

## Summary

CRAQ-based detection of regional (CRE) and structural (CSE) assembly errors at single-nucleotide resolution by mining clipped alignments (CIGAR S/H) plus coverage signals after mapping NGS short reads and SMS long reads back to a draft assembly, reference-free.

## When to use

Use when a draft genome assembly must be assessed and its errors pinpointed reference-free, using raw NGS short reads and SMS long reads (HiFi/PacBio/ONT) mapped back to the assembly to expose clipping signals.

## When NOT to use

- Polyploid genome assemblies — the article warns 'evaluation of genome assemblies for species with higher ploidy levels may not be as accurate as the benchmarked cases presented here'.
- Cases requiring a definitive call for ambiguous loci — regions with clipping ratio above h but below f are reported as 'ambiguous heterozygous or error region' without classification.

## Inputs

- Draft genome assembly FASTA (contigs) as the mapping query/reference
- Raw NGS short-read FASTQ (e.g. Illumina paired-end)
- Raw SMS long-read FASTQ (PacBio HiFi, PacBio CLR, or ONT)
- Minimap2 SAM/BAM alignments of both read sets against the draft assembly

## Outputs

- CRE coordinate list (small-scale Clip-based Regional Errors)
- CSE coordinate list (large-scale Clip-based Structural Errors / misjoin breakpoints)
- CRH and CSH heterozygous-variant coordinates
- R-AQI and S-AQI assembly quality indicator scores (AQI >90 reference quality; 80–90 high; 60–80 draft; <60 low)
- Optional CRAQ-corrected contig FASTA split at CSE positions

## How to apply

Map raw short reads to the draft contigs with Minimap2 '–ax sr' and long reads with '–ax map-hifi' (or 'map-pb'/'map-ont'), then filter alignments with 'samtools view -F 1796 -q 20' to drop MAPQ<20, unmapped, secondary, QC-failed, and PCR-duplicated records. CRAQ extracts all clipped reads (CIGAR 'S' or 'H') and classifies loci by effective clipping ratio: a clipping ratio within h (default 0.4–0.6) marks heterozygous variants (CRHs/CSHs), while a ratio exceeding f (default 0.75) marks a mapping breakpoint. Candidates are excluded when coverage is below m (default 2) or above M (default 5× average coverage), and multi-mapped reads are filtered by default when calling CSE breakpoints; the coverage-discrepancy cutoff between SMS reads is user-tunable via '-d'. Output coordinates are classified as CREs (small-scale Clip-based Regional Errors) or CSEs (large-scale Clip-based Structural Errors) and normalized into R-AQI and S-AQI scores, with CSE breakpoints optionally used to split misjoined contigs.

## Related tools

- **CRAQ** (Extracts clipped alignments and calls CRE/CSE errors, heterozygous CRH/CSH loci, and R-AQI/S-AQI scores from the filtered BAM files) — https://github.com/JiaoLaboratory/CRAQ
- **Minimap2** (Maps NGS short reads ('-ax sr') and SMS long reads ('-ax map-hifi/map-pb/map-ont') back to the draft assembly to generate clipping signals) — https://github.com/lh3/minimap2
- **SAMtools** (Filters alignments with 'samtools view -F 1796 -q 20' before clipped-read extraction)

## Examples

```
minimap2 -ax sr draft_contigs.fa reads_1.fq reads_2.fq | samtools view -F 1796 -q 20 -b > filtered.bam && CRAQ -n 10 -s 10000 -q 20 filtered_short.bam filtered_long.bam -o craq_out
```

## Evaluation signals

- On the hg38 simulation benchmark, CRAQ should reach F1 ≈ 97.463% for CREs and ≈ 97.067% for CSEs (reported F1 >97%), beating Inspector and Merqury (F1 87.7%).
- Heterozygous calls (CRH/CSH) must coincide with clipping ratios in the h band (0.4–0.6); error breakpoints must exceed f = 0.75.
- Filtered BAM must contain no alignments with MAPQ <20 or flags matching 1796 (unmapped, secondary, QC-failed, PCR-duplicate).
- Candidates with coverage below m = 2 or above M = 5× average coverage should be absent from the call set.
- AQI output should fall in a documented band: >90 reference quality, 80–90 high quality, 60–80 draft, <60 low quality.

## Limitations

- Ambiguous regions (coverage above h but below f) are reported as 'ambiguous heterozygous or error region' without a definitive classification.
- The CSE/CRE coverage-discrepancy cutoff '-d' is user-tunable with no stated universal value.
- Error breakpoint ratios can fall below 100% in practice due to sequencing errors or inaccurate read mapping.
- Read mapping is the most resource-intensive step; users must manually split query sequences into fragments to parallelize.
- Multi-mapped reads are filtered out by default when identifying CSE breakpoints, and accuracy for higher-ploidy genomes is not established.

## Evidence

- [abstract] CRAQ integrates the reads-mapping status (reads coverage & clipping signals) of NGS short-reads and SMS long-reads to detect types of assembly errors and heterozygous variants: "CRAQ integrates the reads-mapping status (reads coverage & clipping signals) of NGS short-reads and SMS long-reads to detect types of assembly errors"
- [discussion] CRAQ extracts clipped reads and classifies loci: "CRAQ first extracts all clipped reads, coded as “S” or “H” in the Compact Idiosyncratic Gapped Alignment Report (CIGAR) string"
- [discussion] Filter thresholds for alignments: [evidence span withheld — anchor status: not_found]
- [discussion] Heterozygous cutoff region: "a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6)"
- [discussion] Breakpoint cutoff: "A region is classified as a mapping breakpoint when the ratio exceeds a stringent cutoff value f (default = 0.75)"
- [discussion] Coverage exclusion bounds: "excludes candidates with extremely low coverage (m, default = 2) and high coverage (M, default = 5 * average coverage)"
- [readme] CRAQ output classes: "It can report precise locations of small-scale Clip-based Regional Errors (CREs), large-scale Clip-based Structural Errors (CSEs)"
