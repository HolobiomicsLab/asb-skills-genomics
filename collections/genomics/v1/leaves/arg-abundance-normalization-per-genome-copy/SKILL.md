---
name: arg-abundance-normalization-per-genome-copy
description: Use when long-read metagenomic ARG profiling has produced species-resolved
  ARG detections and read clusters, and absolute abundances must be expressed as ARG
  copies per genome (cpg) rather than raw coverage.
when_to_use_negative:
- Input reads are short (second-generation) reads — Argo's overlap-graph clustering
  and Melon genome-copy estimation require long reads
- Coverage is extremely low or species fall below 1× genome copy, where Argo reports
  'unclassified' and cpg normalization is unstable
- 'Goal is ORF-level or sequence-typing resolution: highly similar ARGs are clustered
  (e.g. blaOXA-1 and blaOXA-1042 as blaOXA), so cpg cannot resolve sub-ARG-level abundance'
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3837
- http://edamontology.org/topic_3697
tools:
- name: Argo
  role: End-to-end profiler that performs the clustering, set-cover species assignment,
    and final Melon-based cpg normalization
  repo: https://github.com/xinehc/argo
- name: Melon
  role: Estimates species genome copies from read depth, providing the denominator
    for normalizing summed ARG copy coverage to cpg
- name: minimap2
  role: Produces the read overlaps (-x ava-ont / -x map-ont) from which coverage supporting
    ARG copy and genome-copy estimates derive
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:2d5a4eb2d07d803ca1d7487053a4e4b881c57625dd58cf2d53fbc0ad0708a94e
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
  - MCL
  - Melon
  - minimap2
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3837
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# arg-abundance-normalization-per-genome-copy

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalize summed ARG copy coverage by species genome-copy estimates computed with Melon, yielding ARG copies per genome (cpg) — equivalent to ARG copies per cell under a single-genome-per-cell assumption — written as a tab-delimited abundance table.

## When to use

Use when long-read metagenomic ARG profiling has produced species-resolved ARG detections and read clusters, and absolute abundances must be expressed as ARG copies per genome (cpg) rather than raw coverage.

## When NOT to use

- Input reads are short (second-generation) reads — Argo's overlap-graph clustering and Melon genome-copy estimation require long reads
- Coverage is extremely low or species fall below 1× genome copy, where Argo reports 'unclassified' and cpg normalization is unstable
- Goal is ORF-level or sequence-typing resolution: highly similar ARGs are clustered (e.g. blaOXA-1 and blaOXA-1042 as blaOXA), so cpg cannot resolve sub-ARG-level abundance

## Inputs

- Quality-controlled ONT or PacBio long reads (e.g. FASTA/FASTQ such as example.fa.gz, 10,000 QC'd R10.3 mock reads)
- Argo database with SARG+ protein sequences (database/prot.fa, database/sarg.fa) and ARG-tailored GTDB/plasmid reference FASTAs
- Read clusters with species labels produced by Argo's MCL clustering and set-cover assignment
- Arg copy coverage summed per detected ARG subtype

## Outputs

- Tab-delimited abundance table of ARG copies per genome (cpg) per species and ARG type/subtype, e.g. example.sarg.tsv
- Per-species genome-copy estimates from Melon used as the normalization denominator

## How to apply

After Argo has clustered ARG-containing reads (MCL on the minimap2 overlap identity matrix) and assigned species labels via greedy weighted set cover, compute summed ARG copy coverage for each detected ARG and divide it by species genome copies estimated with Melon, a read-depth-based tool. This converts per-species ARG signal into copies per genome (cpg), which is directly interpretable as copies per cell assuming each cell contains a single genome. Write the resulting cpg values to a tab-delimited table (e.g. example.sarg.tsv). Note that the default detection threshold marks ARGs of species below 1× genome copy as 'unclassified', so cpg is only reported for species meeting that coverage floor. Insufficient sequencing coverage destabilizes this normalization, so verify adequate coverage before trusting the abundance estimates.

## Related tools

- **Argo** (End-to-end profiler that performs the clustering, set-cover species assignment, and final Melon-based cpg normalization) — https://github.com/xinehc/argo
- **Melon** (Estimates species genome copies from read depth, providing the denominator for normalizing summed ARG copy coverage to cpg)
- **minimap2** (Produces the read overlaps (-x ava-ont / -x map-ont) from which coverage supporting ARG copy and genome-copy estimates derive)

## Examples

```
argo example.fa.gz -d database -o . --plasmid
```

## Evaluation signals

- Output table is tab-delimited and contains finite, non-negative cpg values for every reported ARG-species pair
- Reproducing the documented example yields 8 ARG-carrying species, 14 ARG types, 138 ARG subtypes, and a total of 24.478 ARG copies per genome in example.sarg.tsv
- Species whose estimated genome copies fall below 1× are reported as 'unclassified' rather than assigned a cpg value
- cpg denominators match Melon's per-species genome-copy estimates recomputed from the same read-depth input

## Limitations

- Insufficient coverage leads to unstable normalization and hinders accurate taxonomic classification due to inadequate read overlaps and less effective read clustering
- By default Argo considers ARGs of species with less than one genome copy (1×) as 'unclassified', making it less suitable for extremely rare species
- cpg assumes each cell contains a single genome, which may not hold for polyploid organisms or extrachromosomal copy-number variation

## Evidence

- [methods] Workflow: cpg written to example.sarg.tsv: [evidence span withheld — anchor status: not_found]
