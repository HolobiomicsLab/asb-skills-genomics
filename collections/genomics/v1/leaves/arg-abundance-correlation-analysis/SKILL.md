---
name: arg-abundance-correlation-analysis
description: Use when validating a species-resolved ARG profiler against mock communities
  with known member genomes, or when comparing estimated cpg-based ARG abundances
  to ground-truth expected profiles via Pearson correlation and precision/recall/F1.
when_to_use_negative:
- Samples are real complex metagenomes without known member genomes — no expected
  ARG profile exists to correlate against, so this validation design does not apply
- Input reads are short second-generation reads — Argo relies on long-read overlapping
  of Nanopore/PacBio reads and is not applicable
- Species fall below 1× genome copy coverage — Argo marks such ARGs 'unclassified',
  making abundance correlation uninformative
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Argo
  role: Run on quality-controlled mock-community long reads with '-d database --plasmid'
    to produce estimated ARG type/subtype, host, and cpg abundance profiles
  repo: https://github.com/xinehc/argo
- name: DIAMOND
  role: Align supplier reference genomes to SARG+ (blastx with --evalue 1e-15, --id
    90, --subject-cover 90) to compute expected ARG occurrence counts; also builds
    the sarg protein index
- name: minimap2
  role: Index the database FASTA files with '-x map-ont -d' for ONT read alignment
- name: nanoq
  role: Quality-filter mock long reads with '--min-qual 10 --min-len 1000' before
    profiling
- name: Porechop
  role: Adapter-trim raw long reads with '--discard_middle' during QC
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:3e4d9d620118264df1a39d3f063567867e8246666ec59a62af21c9bb1ec580dc
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
  - minimap2
  - nanoq
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# arg-abundance-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark Argo's species-resolved ARG profiling on sequenced long-read mock communities (D6300, MSA2006, MSA1003, D6331) by comparing estimated versus expected ARG profiles, scoring type/subtype precision, recall, F1-score, and Pearson correlation of ARG copies-per-genome (cpg) abundances. This validates that a long-read overlapping profiler recovers true ARG composition and abundance under varying read lengths and quality scores.

## When to use

Use when validating a species-resolved ARG profiler against mock communities with known member genomes, or when comparing estimated cpg-based ARG abundances to ground-truth expected profiles via Pearson correlation and precision/recall/F1.

## When NOT to use

- Samples are real complex metagenomes without known member genomes — no expected ARG profile exists to correlate against, so this validation design does not apply
- Input reads are short second-generation reads — Argo relies on long-read overlapping of Nanopore/PacBio reads and is not applicable
- Species fall below 1× genome copy coverage — Argo marks such ARGs 'unclassified', making abundance correlation uninformative

## Inputs

- Quality-controlled long reads (Nanopore/PacBio FASTQ) from sequenced mock communities (D6300, MSA2006, MSA1003, D6331)
- SARG+ database downloaded from Zenodo (database.tar.gz with sarg.fa and prot.fa), indexed via DIAMOND makedb and minimap2 '-x map-ont -d'
- Supplier reference genomes of the mock-community members for computing expected ARG abundances

## Outputs

- Argo ARG profile table per sample listing predicted ARG types/subtypes, potential hosts, and estimated cpg abundances
- Expected ARG occurrence counts from DIAMOND blastx alignment of reference genomes to SARG+
- Type- and subtype-level precision, recall, F1-score, and Pearson correlation coefficients of cpg values

## How to apply

Quality-control the raw long reads of the four mock communities with Porechop ('--discard_middle') and nanoq ('--min-qual 10 --min-len 1000'), removing yeast-mapping and unmapped reads from the mock samples. Download the SARG+ database from Zenodo (database.tar.gz) and index it with DIAMOND makedb plus minimap2 '-x map-ont -d' per the Argo repository setup. Run Argo on each QC'd sample with '-d database --plasmid', keeping the adaptive identity cutoff of 90 - 2.5 * 100 * median sequence divergence inferred from read overlaps. Build the expected profile by aligning the supplier reference genomes to SARG+ with DIAMOND blastx ('--evalue 1e-15 --id 90 --subject-cover 90 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0'), filtering HSPs and counting ARG occurrences. Compare estimated versus expected profiles at the type and subtype levels, computing precision, recall, F1-score, and Pearson correlation of the cpg values; a correct run reproduces type-level Pearson correlations of 0.990 (D6300), 0.976 (MSA2006), 0.981 (MSA1003), and 0.672 (D6331) with precision above 0.9 for every mock.

## Related tools

- **Argo** (Run on quality-controlled mock-community long reads with '-d database --plasmid' to produce estimated ARG type/subtype, host, and cpg abundance profiles) — https://github.com/xinehc/argo
- **DIAMOND** (Align supplier reference genomes to SARG+ (blastx with --evalue 1e-15, --id 90, --subject-cover 90) to compute expected ARG occurrence counts; also builds the sarg protein index)
- **minimap2** (Index the database FASTA files with '-x map-ont -d' for ONT read alignment)
- **nanoq** (Quality-filter mock long reads with '--min-qual 10 --min-len 1000' before profiling)
- **Porechop** (Adapter-trim raw long reads with '--discard_middle' during QC)

## Examples

```
argo -d database --plasmid qc_D6300.fastq
```

## Evaluation signals

- Type-level Pearson correlation of cpg values reproduces 0.990 for D6300, 0.976 for MSA2006, 0.981 for MSA1003, and 0.672 for D6331
- Type- and subtype-level precision exceed 0.9 for every mock community
- Plasmid-borne ARG calls are recovered: ant(4')-I, qacG, tet(L) for D6300; aph(2")-I at 4.256 cpg in Enterococcus faecalis for MSA2006
- Argo is invoked with '-d database --plasmid' and the adaptive identity cutoff equals 90 - 2.5 * 100 * median sequence divergence inferred from read overlaps
- Expected counts come from DIAMOND blastx with --evalue 1e-15 --id 90 --subject-cover 90 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0 followed by HSP filtering

## Limitations

- Argo is inherently reference-based and may misclassify unseen species or novel plasmid syntenies, as reflected in D6331's low Pearson correlation of 0.672
- By default, Argo considers ARGs of species with less than one genome copy (1×) as 'unclassified'; insufficient coverage also hinders read overlap clustering and stable normalization
- Highly similar ARGs are clustered (e.g., blaOXA-1 and blaOXA-1042 grouped as blaOXA), lowering subtype-level resolution for detailed sequence typing
- ARGs originating from point mutations are excluded from the database because SNPs cannot be reliably distinguished from sequencing errors in read-based profiling

## Evidence

- [results] with Pearson correlation coefficients of 0.672 for D6331, but 0.976, 0.981, and 0.990 for MSA2006, MSA1003, and D6300, respectively.: [evidence span withheld — anchor status: not_found]
