---
name: sv-breakpoint-gene-annotation
description: Use when nanopore reads from targeted fusion-gene enrichment need fusion
  partner and breakpoint identification, i.e. mapping long reads to GRCh37, calling
  SVs, and annotating SV breakpoints with genes to nominate fusions.
when_to_use_negative:
- Input DNA is fragmented (e.g. FFPE material), which will most likely fail to comprehensively
  identify fusion genes due to short read lengths
- Only one fusion-spanning read supports the candidate fusion and no breakpoint PCR
  validation is possible — the two-read cut-off exists so fusions can be called without
  further validation
- Whole-genome amplification was used on low-input DNA without prior knowledge of
  both fusion gene partners
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: NanoFG
  role: Pipeline integrating mapping, SV calling, gene annotation, fusion filtering,
    and primer design for fusion gene detection
  repo: https://github.com/SdeBlank/NanoFG
- name: Minimap2
  role: Maps long nanopore reads to the GRCh37 human reference genome (v2.6)
  repo: https://github.com/lh3/minimap2
- name: NanoSV
  role: Structural variant caller for long-read data (v1.2.4, default parameters min_mapq=12,
    depth_support=False, mapq_flag=48)
  repo: https://github.com/mroosmalen/nanosv
- name: Samtools
  role: SAM/BAM manipulation within the NanoFG pipeline (v1.7)
- name: Primer3
  role: Designs breakpoint-spanning validation primers from the breakpoint sequence
  repo: https://github.com/primer3-org/primer3
- name: Sniffles
  role: Alternative SV caller selectable in NanoFG (v1.0.9, parameters '-s 2 -n -1
    --genotype')
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:a3af02e1a412ad191f682224743508b2d2c45e7aed9bdbab406b25142b6d892c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SdeBlank/NanoFG
  tools:
  - FUDGE
  - LAST
  - Minimap2
  - NanoFG
  - NanoSV
  - Primer3
  - Samtools
  - Sniffles
  - Wtdbg2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# sv-breakpoint-gene-annotation

## Summary

Annotates both ends of structural variants from long-read nanopore data with ENSEMBL genes to flag inter-gene SVs as candidate fusion genes, then filters and reports fusion configuration, exact breakpoint locations, breakpoint sequences, and Primer3 validation primers. Implemented as the NanoFG pipeline following FUDGE CRISPR-Cas9 enrichment sequencing.

## When to use

Use when nanopore reads from targeted fusion-gene enrichment need fusion partner and breakpoint identification, i.e. mapping long reads to GRCh37, calling SVs, and annotating SV breakpoints with genes to nominate fusions.

## When NOT to use

- Input DNA is fragmented (e.g. FFPE material), which will most likely fail to comprehensively identify fusion genes due to short read lengths
- Only one fusion-spanning read supports the candidate fusion and no breakpoint PCR validation is possible — the two-read cut-off exists so fusions can be called without further validation
- Whole-genome amplification was used on low-input DNA without prior knowledge of both fusion gene partners

## Inputs

- FASTQ directory of nanopore reads (or BAM plus VCF via -b/-v flags)
- GRCh37 human reference genome
- ENSEMBL gene annotation database

## Outputs

- Filtered and gene-annotated SV VCF
- Fusion gene report listing fusion partners, exact breakpoint locations, breakpoint sequences, and fusion-spanning primer sequences
- Primer3 breakpoint-spanning validation primers

## How to apply

Map long nanopore reads to the GRCh37 human reference genome with minimap2 (v2.6), then call structural variants with NanoSV (v1.2.4) using default parameters (min_mapq=12, depth_support=False, mapq_flag=48); alternatively Sniffles (v1.0.9) with '-s 2 -n -1 --genotype' can be selected. Filter the resulting VCF by removing all SVs that fail the caller's built-in filters and removing all insertions. Annotate both ends of each SV with ENSEMBL genes: SVs with both ends in different genes are flagged as possible fusions, and only same-strand fusions capable of producing a continuous transcript are retained. Apply a cut-off of at least two fusion-spanning reads per fusion to reliably detect a fusion gene without further validation, then design breakpoint-spanning validation primers with Primer3. The final NanoFG report gives the fusion gene configuration, breakpoint-location, breakpoint-sequence, and fusion-spanning primer sequences; runtime on ~25000 nanopore reads is approximately 20 min on a single thread.

## Related tools

- **NanoFG** (Pipeline integrating mapping, SV calling, gene annotation, fusion filtering, and primer design for fusion gene detection) — https://github.com/SdeBlank/NanoFG
- **Minimap2** (Maps long nanopore reads to the GRCh37 human reference genome (v2.6)) — https://github.com/lh3/minimap2
- **NanoSV** (Structural variant caller for long-read data (v1.2.4, default parameters min_mapq=12, depth_support=False, mapq_flag=48)) — https://github.com/mroosmalen/nanosv
- **Samtools** (SAM/BAM manipulation within the NanoFG pipeline (v1.7))
- **Primer3** (Designs breakpoint-spanning validation primers from the breakpoint sequence) — https://github.com/primer3-org/primer3
- **Sniffles** (Alternative SV caller selectable in NanoFG (v1.0.9, parameters '-s 2 -n -1 --genotype'))

## Examples

```
bash NanoFG.sh -f </path/to/fastqdir> -n SAMPLE_NAME
```

## Evaluation signals

- VCF contains no insertions and no SVs failing the built-in NanoSV/Sniffles filters
- Every reported fusion has both SV ends annotated in different ENSEMBL genes on the same strand with a continuous transcript possible
- Each reported fusion is supported by at least two fusion-spanning reads
- Final report includes fusion partners, exact breakpoint locations, breakpoint sequences, and fusion-spanning primer sequences
- NanoFG completes on ~25000 nanopore reads in roughly 20 min on a single thread

## Limitations

- The assay requires non-fragmented DNA; FFPE material will most likely fail to comprehensively identify fusion genes
- Two fusions were detected with only one supporting read, requiring manual validation by breakpoint PCR
- Intratumoral heterogeneity and tumor purity likely influence the lower detection limits
- Decreased on-target coverage observed for low-throughput sequencing runs and/or more distal breakpoint events
- Assay demonstrated on ten recurrent fusion genes; expansion to any gene of interest is possible but not yet demonstrated

## Evidence

- [other] NanoFG extracts fusion-spanning reads, detects fusion genes, and reports the exact fusion gene configuration, breakpoint-location, breakpoint-sequence, and fusion-spanning primer sequences.: [evidence span withheld — anchor status: not_found]
- [discussion] Reads were mapped to the human reference genome version GRCHh37 by using minimap2 (v. 2.6): "Reads were mapped to the human reference genome version GRCHh37 by using minimap2 (v. 2.6)"
- [discussion] By default, all SVs that do not pass the built-in NanoSV or Sniffles filters are removed. In addition, all insertions are also removed from the VCF.: "By default, all SVs that do not pass the built-in NanoSV or Sniffles filters are removed. In addition, all insertions are also removed from the VCF."
- [discussion] If both ends of the SV are positioned in different genes it was flagged as a possible fusion.: "If both ends of the SV are positioned in different genes it was flagged as a possible fusion."
- [discussion] We set a cut-off of at least two fusion-spanning reads to reliably detect a fusion gene without further validation.: "We set a cut-off of at least two fusion-spanning reads to reliably detect a fusion gene without further validation."
- [discussion] The run time of NanoFG on +−25000 nanopore reads is approximately 20 min using a single thread.: "The run time of NanoFG on +−25000 nanopore reads is approximately 20 min using a single thread."
- [readme] bash NanoFG.sh -f </path/to/fastqdir>  [-n SAMPLE_NAME ] [-s SELECTION] [-cf] [-cc] [-df] [-dc]: "bash NanoFG.sh -f </path/to/fastqdir>  [-n SAMPLE_NAME ] [-s SELECTION] [-cf] [-cc] [-df] [-dc]"
- [discussion] A limitation of this approach is the requirement of non-fragmented DNA. Applying the FUDGE crRNA protocol to FFPE material: "A limitation of this approach is the requirement of non-fragmented DNA. Applying the FUDGE crRNA protocol to FFPE material"
