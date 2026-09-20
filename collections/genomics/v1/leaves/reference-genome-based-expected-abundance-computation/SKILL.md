---
name: reference-genome-based-expected-abundance-computation
description: 'Use when validating a long-read ARG profiler on sequenced mock communities:
  supplier reference genomes with known ARG content are available and a per-ARG expected
  abundance (cpg) profile is needed for precision/recall/Pearson evaluation.'
when_to_use_negative:
- Profiling unknown environmental or clinical metagenomes where no curated reference
  genomes exist — use the profiler (e.g., Argo) directly instead of building an expected
  profile
- When ARG content must be discovered de novo (novel genes, point-mutation-derived
  ARGs), since this workflow only counts occurrences of known SARG+ reference entries
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_3837
- http://edamontology.org/topic_3673
tools:
- name: DIAMOND
  role: Aligns supplier reference genomes (blastx) against the indexed SARG+ protein
    database to identify ARG homologs and their occurrence counts
  repo: https://github.com/xinehc/sarg-curation
- name: Argo
  role: Profiler whose estimated ARG cpg values are compared against the expected
    abundances computed here
  repo: https://github.com/xinehc/argo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:64569cc5fa18dbc99d5a42bc6d141bfe3c6531517b160b27769d89f5290b8f67
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  repo_url: https://github.com/xinehc/sarg-curation
  tools:
  - Argo
  - DIAMOND
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_3837
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# reference-genome-based-expected-abundance-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the expected (ground-truth) ARG abundance profile of a mock community by aligning supplier reference genomes against the SARG+ database with DIAMOND blastx, then filtering HSPs and counting ARG occurrences. Provides the reference profile against which Argo's estimated ARG abundances (cpg values) are validated.

## When to use

Use when validating a long-read ARG profiler on sequenced mock communities: supplier reference genomes with known ARG content are available and a per-ARG expected abundance (cpg) profile is needed for precision/recall/Pearson evaluation.

## When NOT to use

- Profiling unknown environmental or clinical metagenomes where no curated reference genomes exist — use the profiler (e.g., Argo) directly instead of building an expected profile
- When ARG content must be discovered de novo (novel genes, point-mutation-derived ARGs), since this workflow only counts occurrences of known SARG+ reference entries

## Inputs

- Supplier reference genome assemblies (Fasta) of the mock community strains (e.g., D6300, D6331, MSA1003, MSA2006)
- SARG+ protein database (sarg.fa from xinehc/sarg-curation, indexed with DIAMOND makedb)
- Known strain abundances/composition of each mock community (for scaling counts to cpg)

## Outputs

- Filtered DIAMOND blastx HSP table mapping each ARG subtype to its source reference genome(s)
- Per-ARG expected occurrence counts and expected cpg values per mock community
- Expected ARG type- and subtype-level abundance profiles for evaluation

## How to apply

Align each supplier reference genome of the mock community to the SARG+ database using DIAMOND blastx with stringent parameters: --evalue 1e-15 --id 90 --subject-cover 90 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsp 0. Apply range culling and range coverage so partial, redundant, or frameshifted hits to paralogous SARG+ sequences are suppressed, mirroring the curated-database logic that pairs experimentally validated reference sequences with computationally derived homologs. Filter the resulting HSPs and count ARG occurrences per genome; combined with the known organism abundance of each mock community strain, this yields expected ARG copies-per-genome (cpg) values. Compare these expected profiles at ARG type and subtype level against profiler estimates using precision, recall, F1-score, and Pearson correlation of the cpg values.

## Related tools

- **DIAMOND** (Aligns supplier reference genomes (blastx) against the indexed SARG+ protein database to identify ARG homologs and their occurrence counts) — https://github.com/xinehc/sarg-curation
- **Argo** (Profiler whose estimated ARG cpg values are compared against the expected abundances computed here) — https://github.com/xinehc/argo

## Examples

```
diamond blastx --db database/sarg --query reference_genome.fna --evalue 1e-15 --id 90 --subject-cover 90 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0 -o expected_hsps.tsv
```

## Evaluation signals

- Filtered HSPs should only retain alignments with >=90% identity and >=90% subject coverage (--id 90 --subject-cover 90)
- Expected profiles must reproduce the reported validation targets: type-level Pearson correlations of 0.990 (D6300), 0.976 (MSA2006), 0.981 (MSA1003), 0.672 (D6331) versus Argo estimates
- Precision and recall above 0.9 at both type and subtype level for every mock community
- Expected cpg values must include known plasmid-borne ARGs such as ant(4')-I, qacG, tet(L) for D6300 and aph(2")-I at 4.256 cpg in Enterococcus faecalis for MSA2006
- Occurrence counts must be normalized to copies-per-genome (cpg) rather than raw read or hit counts before comparison

## Limitations

- Relies on SARG+ annotations; ARGs arising from point mutations are excluded from the database because SNPs cannot be reliably distinguished from sequencing errors in read-based profiling
- Highly similar ARGs are clustered in SARG+ (e.g., blaOXA-1 and blaOXA-1042 grouped as blaOXA), so expected profiles carry type-level rather than exact-sequence resolution
- Only valid for well-characterized mock communities with supplier reference genomes; not applicable to novel or understudied environments requiring de novo ARG identification

## Evidence

- [methods] Compute expected ARG abundances by aligning the supplier reference genomes to SARG+ with DIAMOND blastx (--evalue 1e-15 --id 90 --subject-cover 90 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0), filtering HSPs, and counting ARG occurrences.: [evidence span withheld — anchor status: not_found]
- [methods] Compare estimated versus expected ARG profiles at the type and subtype levels, computing precision, recall, F1-score, and Pearson correlation of the cpg values.: [evidence span withheld — anchor status: not_found]
