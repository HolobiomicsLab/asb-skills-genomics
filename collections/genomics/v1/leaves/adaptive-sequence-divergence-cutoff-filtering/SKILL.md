---
name: adaptive-sequence-divergence-cutoff-filtering
description: Use when profiling ARG-carrying long reads (ONT/PacBio) where read-cluster
  construction requires quality-controlled overlaps and a single fixed identity cutoff
  would over- or under-filter reads of varying error rates.
when_to_use_negative:
- Input is short-read (second-generation) data — minimap2 '-x ava-ont' overlapping
  and dv-based cutoffs target ONT/PacBio error profiles; short-read AMR pipelines
  (e.g. Lynx) are the appropriate alternative.
- Coverage is insufficient (below ~1× genome copy or sparse overlaps) — inadequate
  read overlaps make clustering unstable, and the article notes Argo defaults to marking
  such species' ARGs as 'unclassified'.
- Goal is de novo reconstruction of ARG genomic context or discovery of unseen hosts
  — assembly/binning-based approaches (metaFlye + Kraken2, SemiBin2 + GTDB-Tk) are
  better suited, and Argo is reference-based.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0196
tools:
- name: minimap2
  role: Produces the read-overlap graph ('-x ava-ont') whose dv tags supply the median
    sequence divergence used for adaptive filtering
- name: Argo
  role: Encapsulates the adaptive dv cutoff, identity matrix construction, and downstream
    MCL clustering and set-cover assignment
  repo: https://github.com/xinehc/argo
- name: MCL
  role: Consumes the identity matrix (x = 1 − dv) from the filtered overlaps to decompose
    the read-overlap graph into clusters
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:d795241db0ad229c060170a73d8c2183e409c7be505fa05ea5f2d24a7218b705
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Adaptive sequence divergence cutoff filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An overlap-filtering step in the Argo long-read ARG pipeline: ONT read overlaps produced by minimap2 are scored with dv (sequence divergence) tags, and both the per-overlap divergence threshold and the alignment identity cutoff are set adaptively from the observed median dv rather than fixed constants.

## When to use

Use when profiling ARG-carrying long reads (ONT/PacBio) where read-cluster construction requires quality-controlled overlaps and a single fixed identity cutoff would over- or under-filter reads of varying error rates.

## When NOT to use

- Input is short-read (second-generation) data — minimap2 '-x ava-ont' overlapping and dv-based cutoffs target ONT/PacBio error profiles; short-read AMR pipelines (e.g. Lynx) are the appropriate alternative.
- Coverage is insufficient (below ~1× genome copy or sparse overlaps) — inadequate read overlaps make clustering unstable, and the article notes Argo defaults to marking such species' ARGs as 'unclassified'.
- Goal is de novo reconstruction of ARG genomic context or discovery of unseen hosts — assembly/binning-based approaches (metaFlye + Kraken2, SemiBin2 + GTDB-Tk) are better suited, and Argo is reference-based.

## Inputs

- Quality-controlled long reads (ONT or PacBio) containing ARG hits, e.g. FASTA/FASTQ such as the 10,000-read Loman Lab R10.3 ONT mock sample (example.fa.gz)
- Argo database indexed with 'minimap2 -x map-ont -d' (GTDB/plasmid FASTAs from database.tar.gz, Zenodo record 20451539)
- minimap2 PAF overlaps carrying dv (divergence) tags

## Outputs

- Filtered overlap set retaining only overlaps with dv <= max(2.5 × median dv, 0.05)
- Identity matrix (x = 1 − dv) over reads, ready for MCL graph clustering
- Read clusters (e.g. 149 clusters on the documented mock example) used for species-level set-cover taxonomic assignment

## How to apply

Overlap ARG-containing reads with minimap2 approximate mapping using '-x ava-ont' and retain the dv tag per overlap. Compute the median sequence divergence across all overlaps to characterize the sample's error profile. Discard overlaps where dv exceeds max(2.5 × median dv, 0.05), and set the adaptive identity cutoff to 90 − 2.5 × 100 × median dv. Build the identity matrix (x = 1 − dv) from the filtered overlaps and pass it to MCL clustering (expansion e = 2, inflation r = 2, pruning z < 1/n per iteration). The rationale, per the article, is that read-overlapping aggregates evidence across local read clusters, reducing misclassifications versus per-read taxonomic classifiers while avoiding assembly.

## Related tools

- **minimap2** (Produces the read-overlap graph ('-x ava-ont') whose dv tags supply the median sequence divergence used for adaptive filtering)
- **Argo** (Encapsulates the adaptive dv cutoff, identity matrix construction, and downstream MCL clustering and set-cover assignment) — https://github.com/xinehc/argo
- **MCL** (Consumes the identity matrix (x = 1 − dv) from the filtered overlaps to decompose the read-overlap graph into clusters)

## Examples

```
minimap2 -x ava-ont example.fa.gz example.fa.gz > overlaps.paf
```

## Evaluation signals

- Verify every retained overlap satisfies dv <= max(2.5 × median dv, 0.05) and that the applied identity cutoff equals 90 − 2.5 × 100 × median dv.
- Check that median dv is recomputed per sample from minimap2 dv tags rather than hard-coded, since cutoffs are sample-adaptive.
- On the documented example (10,000 ONT reads, Loman Lab R10.3 mock), the pipeline should reproduce 149 read clusters downstream of the filter.
- Spurious overlaps (high local divergence) should be removed while homologous same-species overlaps survive, supporting correct species labeling at the cluster level.
- The resulting example.sarg.tsv should match the documented statistics: 8 ARG-carrying species, 14 ARG types, 138 ARG subtypes, and 24.478 total ARG copies per genome.

## Limitations

- Insufficient coverage leads to unstable normalization and hinders taxonomic classification due to inadequate read overlaps and less effective read clustering.
- Read-based profiling cannot reliably distinguish SNPs from sequencing errors, so point-mutation-derived ARGs are excluded from the database.
- Argo is inherently reference-based and may misclassify unseen species or novel plasmid syntenies, so adaptive cutoffs cannot rescue overlaps to absent references.

## Evidence

- [methods] minimap2 approximate mapping with dv-based adaptive overlap filtering: [evidence span withheld — anchor status: not_found]
- [methods] adaptive identity cutoff formula: [evidence span withheld — anchor status: not_found]
- [readme] input read requirements and output contract: "Argo takes *quality-controlled* long reads (either Nanopore or PacBio) as input and returns a table listing predicted ARGs (types and subtypes), their potential hosts, and estimated abundances"
- [discussion] coverage limitation for overlap-based clustering: "insufficient coverage can not only lead to unstable normalization but also hinder accurate taxonomic classification due to inadequate read overlaps and less effective read clustering."
