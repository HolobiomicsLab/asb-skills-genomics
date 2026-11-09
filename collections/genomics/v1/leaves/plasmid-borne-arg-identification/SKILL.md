---
name: plasmid-borne-arg-identification
description: Use when profiling long-read metagenomic samples with Argo and you need
  to distinguish plasmid-borne ARGs from chromosomal ones, or when validating ARG–host–plasmid
  calls against mock-community reference genomes.
when_to_use_negative:
- The sample is extremely low coverage (<1x genome copy per species) or reads are
  too short for reliable read-overlap clustering, since plasmid synteny evidence and
  taxonomic classification become unstable
- The goal is de novo identification of novel plasmid syntenies or unseen species,
  since Argo is inherently reference-based
- Input reads are short reads, since Argo requires long reads for the read-overlapping
  strategy that underpins plasmid-context assignment
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3796
- http://edamontology.org/topic_3174
tools:
- name: Argo
  role: Long-read ARG profiler run with '--plasmid' to detect plasmid-borne ARGs and
    assign host species via read-cluster set cover
  repo: https://github.com/xinehc/argo
- name: DIAMOND
  role: Aligns supplier reference genomes to SARG+ (blastx with --evalue 1e-15 --id
    90 --subject-cover 90) to compute expected ARG occurrences for validation
- name: minimap2
  role: Indexes the Argo database for ONT base-level alignment ('-x map-ont -d')
- name: SARG+
  role: Manually curated ARG database (reference + extension) used by Argo as the
    default detection database
  repo: https://github.com/xinehc/sarg-curation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:8928e4fdc6f535e73092c1a6917349fc2f7f584ba63b8c4c956555f1c89fe21c
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
  - SARG+
  - minimap2
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3796
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# plasmid-borne-arg-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify plasmid-borne antibiotic resistance genes (ARGs) and their host species using Argo's plasmid-detection mode on quality-controlled long reads, then validate the calls against supplier reference genomes of sequenced mock communities (e.g., ant(4')-I, qacG, tet(L) in D6300; aph(2")-I at 4.256 cpg in Enterococcus faecalis for MSA2006).

## When to use

Use when profiling long-read metagenomic samples with Argo and you need to distinguish plasmid-borne ARGs from chromosomal ones, or when validating ARG–host–plasmid calls against mock-community reference genomes.

## When NOT to use

- The sample is extremely low coverage (<1x genome copy per species) or reads are too short for reliable read-overlap clustering, since plasmid synteny evidence and taxonomic classification become unstable
- The goal is de novo identification of novel plasmid syntenies or unseen species, since Argo is inherently reference-based
- Input reads are short reads, since Argo requires long reads for the read-overlapping strategy that underpins plasmid-context assignment

## Inputs

- Quality-controlled long reads (Nanopore or PacBio FASTQ) from mock communities D6300, MSA2006, MSA1003, D6331
- SARG+ database downloaded from Zenodo and indexed with DIAMOND makedb and minimap2 ('-x map-ont -d')
- Supplier reference genomes of the mock communities (for computing expected ARG occurrences)

## Outputs

- Table of predicted plasmid-borne ARGs (types and subtypes), their host species, and abundances in ARG copies per genome (cpg)
- Precision, recall, F1-score, and Pearson correlation of cpg values comparing estimated versus expected plasmid-borne ARG profiles

## How to apply

Run Argo on quality-controlled long reads (Porechop '--discard_middle', nanoq '--min-qual 10 --min-len 1000', yeast-mapping and unmapped reads removed for mock samples) with '-d database --plasmid', using the SARG+ database indexed with DIAMOND makedb and minimap2 ('-x map-ont -d'). Argo leverages plasmid synteny in its set-cover-based species labeling to flag ARGs carried on plasmids and reports them as types/subtypes with hosts and abundances in ARG copies per genome (cpg). To compute expected plasmid-borne ARG profiles, align supplier reference genomes to SARG+ with DIAMOND blastx ('--evalue 1e-15 --id 90 --subject-cover 90 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0'), filter HSPs, and count ARG occurrences, then compare estimated versus expected calls at type and subtype levels (precision, recall, F1, Pearson correlation of cpg). Correct application is confirmed when reported plasmid-borne calls are reproduced, e.g., ant(4')-I, qacG, and tet(L) for D6300 and aph(2")-I at 4.256 cpg in Enterococcus faecalis for MSA2006.

## Related tools

- **Argo** (Long-read ARG profiler run with '--plasmid' to detect plasmid-borne ARGs and assign host species via read-cluster set cover) — https://github.com/xinehc/argo
- **DIAMOND** (Aligns supplier reference genomes to SARG+ (blastx with --evalue 1e-15 --id 90 --subject-cover 90) to compute expected ARG occurrences for validation)
- **minimap2** (Indexes the Argo database for ONT base-level alignment ('-x map-ont -d'))
- **SARG+** (Manually curated ARG database (reference + extension) used by Argo as the default detection database) — https://github.com/xinehc/sarg-curation

## Examples

```
argo --input sample.fastq -d database --plasmid --output argo_results
```

## Evaluation signals

- Reproduce type-level Pearson correlations of 0.990 (D6300), 0.976 (MSA2006), 0.981 (MSA1003), and 0.672 (D6331) on the mock communities
- Type/subtype precision above 0.9 for every mock community
- Reported plasmid-borne ARG calls are recovered: ant(4')-I, qacG, tet(L) for D6300; aph(2")-I at 4.256 cpg in Enterococcus faecalis for MSA2006
- Abundances are expressed as ARG copies per genome (cpg), with values comparable to expected counts from DIAMOND blastx against supplier reference genomes
- Output table contains predicted ARG types/subtypes, host species, and cpg abundances without missing-host entries for validated plasmid calls

## Limitations

- Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies
- D6331 shows degraded performance (Pearson correlation 0.672), likely reflecting read-length/quality differences among mock communities
- Insufficient coverage leads to unstable normalization and hinders accurate taxonomic classification due to inadequate read overlaps and less effective read clustering

## Evidence

- [methods] Run Argo on each quality-controlled mock sample with '-d database --plasmid': [evidence span withheld — anchor status: not_found]
- [results] Validation calls for plasmid-borne ARGs in mock communities: [evidence span withheld — anchor status: not_found]
