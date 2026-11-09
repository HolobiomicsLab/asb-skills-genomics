---
name: mock-community-benchmark-evaluation
description: Use when validating a species-resolved ARG profiling pipeline on real
  long-read mock communities with known supplier reference genomes, where estimated
  ARG abundances must be compared to expected profiles.
when_to_use_negative:
- Short-read metagenomic data — Argo relies on long-read overlapping and is designed
  for quality-controlled Nanopore or PacBio reads
- De novo identification of ARG genomic contexts in new, understudied environments
  — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) approaches
  may be better suited
- Applications requiring ORF-level annotation or detailed sequence typing of highly
  similar ARGs (e.g., distinguishing blaOXA-1 from blaOXA-1042)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3837
- http://edamontology.org/topic_0080
tools:
- name: Argo
  role: Long-read ARG profiler run with '-d database --plasmid' on QC-passed mock
    reads to produce species-resolved ARG calls and cpg abundances
  repo: https://github.com/xinehc/argo
- name: SARG+
  role: Manually curated ARG reference database used both by Argo for detection and
    by DIAMOND blastx to compute expected ARG abundances
  repo: https://github.com/xinehc/sarg-curation
- name: nanoq
  role: Quality filtering of long reads with '--min-qual 10 --min-len 1000'
- name: Porechop
  role: Adapter trimming of raw long reads with '--discard_middle'
- name: DIAMOND
  role: Aligns supplier reference genomes to SARG+ via blastx to compute expected
    ARG occurrence counts; also used to build database indices
- name: minimap2
  role: Indexes GTDB/SARG+ sequences with '-x map-ont -d' for read alignment inside
    Argo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:51edd2aba91ad14d8a4a0e736de79bb70a41f50a3e4cf0776a57860834c99920
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  repo_url: https://github.com/xinehc/argo
  tools:
  - Argo
  - DIAMOND
  - Porechop
  - SARG+
  - minimap2
  - nanoq
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3837
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# mock-community-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a long-read ARG profiler (Argo) against real, sequenced mock communities (D6331, MSA2006, MSA1003, D6300) by comparing estimated versus expected ARG profiles at the type and subtype levels using precision, recall, F1-score, and Pearson correlation of cpg values.

## When to use

Use when validating a species-resolved ARG profiling pipeline on real long-read mock communities with known supplier reference genomes, where estimated ARG abundances must be compared to expected profiles.

## When NOT to use

- Short-read metagenomic data — Argo relies on long-read overlapping and is designed for quality-controlled Nanopore or PacBio reads
- De novo identification of ARG genomic contexts in new, understudied environments — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) approaches may be better suited
- Applications requiring ORF-level annotation or detailed sequence typing of highly similar ARGs (e.g., distinguishing blaOXA-1 from blaOXA-1042)

## Inputs

- Raw long-read (Nanopore/ONT) FASTQ files from four sequenced mock communities (D6331, MSA2006, MSA1003, D6300)
- Supplier reference genomes for the mock community members
- SARG+ ARG database downloaded from Zenodo (database.tar.gz)
- Indexed DIAMOND and minimap2 database files (prot.dmnd, sarg.dmnd, *.mmi)

## Outputs

- Argo-predicted ARG table listing types, subtypes, potential hosts, and abundances in ARG copies per genome (cpg) per mock sample
- Expected ARG abundance profiles derived from DIAMOND blastx alignment of reference genomes to SARG+
- Type-level and subtype-level precision, recall, F1-score, and Pearson correlation coefficients per mock community
- Plasmid-borne ARG calls (e.g., ant(4')-I, qacG, tet(L) for D6300; aph(2")-I for MSA2006)

## How to apply

Quality-control each mock community's long reads with Porechop ('--discard_middle') and nanoq ('--min-qual 10 --min-len 1000'), removing yeast-mapping and unmapped reads for mock samples. Download the SARG+ database from Zenodo and index it with 'diamond makedb' and 'minimap2 -x map-ont -d', then run Argo on each QC-passed sample with '-d database --plasmid', retaining the adaptive identity cutoff of 90 - 2.5 * 100 * median sequence divergence inferred from read overlaps. Compute expected ARG abundances by aligning supplier reference genomes to SARG+ with DIAMOND blastx ('--evalue 1e-15 --id 90 --subject-cover 90 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0'), filtering HSPs, and counting ARG occurrences. Compare estimated versus expected profiles at type and subtype levels, computing precision, recall, F1-score, and Pearson correlation of the cpg (copies per genome) values to judge performance.

## Related tools

- **Argo** (Long-read ARG profiler run with '-d database --plasmid' on QC-passed mock reads to produce species-resolved ARG calls and cpg abundances) — https://github.com/xinehc/argo
- **SARG+** (Manually curated ARG reference database used both by Argo for detection and by DIAMOND blastx to compute expected ARG abundances) — https://github.com/xinehc/sarg-curation
- **nanoq** (Quality filtering of long reads with '--min-qual 10 --min-len 1000')
- **Porechop** (Adapter trimming of raw long reads with '--discard_middle')
- **DIAMOND** (Aligns supplier reference genomes to SARG+ via blastx to compute expected ARG occurrence counts; also used to build database indices)
- **minimap2** (Indexes GTDB/SARG+ sequences with '-x map-ont -d' for read alignment inside Argo)

## Examples

```
argo -d database --plasmid sample.fastq.gz > argo_results.tsv
```

## Evaluation signals

- Type-level Pearson correlations reproduce 0.990 (D6300), 0.976 (MSA2006), 0.981 (MSA1003), and 0.672 (D6331)
- Type and subtype precision above 0.9 for every mock community
- Reported plasmid-borne ARG calls recovered, e.g., ant(4')-I, qacG, tet(L) for D6300 and aph(2")-I at 4.256 cpg in Enterococcus faecalis for MSA2006
- Argo output table lists predicted ARG types/subtypes, potential hosts, and cpg abundances for each QC-passed mock sample
- Reads pass QC thresholds: minimum quality score 10 and minimum length 1000 nt

## Limitations

- Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies
- Point-mutation-derived ARGs are excluded from the database because SNPs cannot be reliably distinguished from sequencing errors in read-based profiling
- Argo considers ARGs of species with less than one genome copy (1×) as 'unclassified' by default, making it less suitable for extremely rare species; insufficient coverage hinders read clustering and taxonomic classification
- Highly similar ARGs are clustered (e.g., blaOXA-1 and blaOXA-1042 grouped as blaOXA), lowering resolution for detailed sequence typing
- D6331 shows notably lower type-level correlation (0.672) than the other mocks, indicating variable performance across read-length/quality regimes

## Evidence
