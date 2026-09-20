---
name: read-mapping-quality-filtering
description: Use when raw NGS short reads (Illumina) or SMS long reads (PacBio HiFi/CLR,
  ONT) are being mapped to a draft genome assembly as input for clipped-read-based
  error calling (CRE/CSE) and AQI computation.
when_to_use_negative:
- The alignments are already the filtered BAMs consumed by CRAQ — re-filtering could
  discard legitimate clipped reads and bias clipping-ratio cutoffs.
- The quality-assessment goal is completeness- or k-mer-based (e.g. BUSCO, Merqury,
  LAI) and involves no read mapping to the draft assembly.
- Only a single read type is available and clipping-based error classification (CRE/CSE),
  which integrates both NGS and SMS mapping, cannot be performed.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Minimap2
  role: Maps raw NGS short reads and SMS long reads to the draft assembly using '-ax
    sr' and '-ax map-hifi/map-pb/map-ont' presets before filtering
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Executes the '-F 1796 -q 20' alignment filter (removes unmapped, secondary,
    QC-failed, PCR-duplicate, and MAPQ < 20 records) on the Minimap2 BAM output
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:40ac0762256eda3147432b50948a5d1cd806b9173846f9c0a9a31a0170d72646
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - CRAQ
  - Integrative Genomics Viewer (IGV)
  - Minimap2
  - Perl
  - SAMtools
  - SyRI
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# read-mapping-quality-filtering

## Summary

A cleanup step in the CRAQ assembly-assessment pipeline that maps raw sequencing reads back to a draft assembly with Minimap2 and then removes low-confidence alignments with 'samtools view -F 1796 -q 20' so that only high-quality clipped signals feed error and heterozygosity classification.

## When to use

Use when raw NGS short reads (Illumina) or SMS long reads (PacBio HiFi/CLR, ONT) are being mapped to a draft genome assembly as input for clipped-read-based error calling (CRE/CSE) and AQI computation.

## When NOT to use

- The alignments are already the filtered BAMs consumed by CRAQ — re-filtering could discard legitimate clipped reads and bias clipping-ratio cutoffs.
- The quality-assessment goal is completeness- or k-mer-based (e.g. BUSCO, Merqury, LAI) and involves no read mapping to the draft assembly.
- Only a single read type is available and clipping-based error classification (CRE/CSE), which integrates both NGS and SMS mapping, cannot be performed.

## Inputs

- Raw Illumina short-read data (FASTQ)
- Raw SMS long-read data — PacBio HiFi, PacBio CLR, or ONT (FASTQ)
- Draft genome assembly (FASTA), e.g. HiCanu or Canu contigs
- SAM/BAM alignment files produced by Minimap2

## Outputs

- Sorted, indexed BAM files per read type mapped to the assembly
- Filtered BAM files (unmapped, secondary, QC-failed, PCR-duplicated, and MAPQ < 20 alignments removed) ready for CRAQ input
- Downstream: out_final.Report with CRE, CSE, CRH, CSH counts and whole-genome R-AQI / S-AQI values

## How to apply

Map genomic short reads to the draft assembly with Minimap2 using the '-ax sr' preset and long reads with the '-ax map-hifi' (PacBio HiFi), '-ax map-pb' (PacBio CLR), or '-ax map-ont' (ONT) preset, then sort and index the SAM output into BAM. Filter each alignment file with 'samtools view -F 1796 -q 20', which simultaneously removes unmapped, secondary, QC-failed, and PCR-duplicate alignments (flag 1796) and discards alignments with MAPQ below 20. The rationale is that CRAQ distinguishes true error breakpoints (clipping ratio > f = 0.75) from heterozygous loci (clipping ratio in h = 0.4–0.6) based on effective clipped-alignment ratios and read coverage (candidates below m = 2 or above 5x average coverage are excluded), so low-quality or duplicated alignments would corrupt both the clipping ratios and the coverage statistics. Filtered BAMs (both short-read and long-read) are then passed directly to CRAQ, whose results are inspected in IGV if needed.

## Related tools

- **Minimap2** (Maps raw NGS short reads and SMS long reads to the draft assembly using '-ax sr' and '-ax map-hifi/map-pb/map-ont' presets before filtering) — https://github.com/lh3/minimap2
- **SAMtools** (Executes the '-F 1796 -q 20' alignment filter (removes unmapped, secondary, QC-failed, PCR-duplicate, and MAPQ < 20 records) on the Minimap2 BAM output)

## Examples

```
samtools view -b -F 1796 -q 20 aln.sorted.bam > aln.filtered.bam && samtools index aln.filtered.bam
```

## Evaluation signals

- Filtered BAM contains no records with FLAG bits matching 1796 (unmapped, secondary, QC-fail, or duplicate) and no MAPQ < 20 alignments.
- Coverage distribution of filtered alignments stays within CRAQ's usable band (candidates with coverage below m = 2 or above M = 5x average are excluded downstream, not by this filter).
- Running CRAQ on the filtered BAMs reproduces published benchmarks: e.g. D. melanogaster F1 HiCanu assembly gives ~4.2% (127/3060) true errors (102 CREs, 25 CSEs) with R-AQI ~93.1 and S-AQI ~91.9.
- Filtered alignments retain clipping events ('S'/'H' in the CIGAR string), which CRAQ needs for error and heterozygous-loci calling.
- AQI values from the resulting report should fall in the expected classification band (AQI > 90 reference quality for validated reference assemblies).

## Limitations

- Filtering depends on MAPQ and flag conventions of the mapper; the '-F 1796 -q 20' parameters were validated with Minimap2 2.17+ and SAMtools 1.3.1+.
- Read mapping is the most resource-intensive step of CRAQ; users must manually split query sequences into multiple fragments to parallelize.
- Error-breakpoint ratios can fall below 100% in practice due to sequencing errors or inaccurate read mapping, so filtering cannot eliminate all misclassified loci.
- CRAQ accuracy for polyploid genomes is not established, so the filtered BAMs may not support reliable classification in higher-ploidy settings.

## Evidence

- [discussion] Any read alignments with low mapping quality (MAPQ <20) or that were unmapped, secondary, QC-failed, or PCR-duplicated were filtered out: [evidence span withheld — anchor status: not_found]
- [discussion] filtered out using the '-F 1796 -q 20' parameters in 'samtools view': [evidence span withheld — anchor status: not_found]
- [abstract] By default, Minimap2 '-ax sr' and '-ax map-hifi' options were used: "By default, Minimap2 ‘–ax sr’ and  ‘–ax map-hifi’(‘map-hifi’ for PacBio HiFi,‘map-pb’ for PacBio CLR, ‘map-ont’ for ONT library) options were used"
- [readme] minimap2 PacBio CLR genomic reads example command: [evidence span withheld — anchor status: external_doc]
