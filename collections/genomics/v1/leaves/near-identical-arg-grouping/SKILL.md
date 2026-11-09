---
name: near-identical-arg-grouping
description: Use when curating an ARG reference database for read-based species-level
  surveillance, where highly similar sequences within a subtype must be collapsed
  to avoid ambiguous subtype assignment from long or short reads.
when_to_use_negative:
- The application requires ORF-level annotation or detailed sequence typing, since
  clustering near-identical ARGs (e.g., blaOXA-1 with blaOXA-1042) lowers resolution
  to the subtype level
- The goal is detecting ARGs arising from point mutations, which are deliberately
  excluded from SARG+ because read-based profiling cannot distinguish SNPs from sequencing
  errors
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
tools:
- name: MMseqs2
  role: Protein sequence clustering (easy-cluster) used to deduplicate the SARG+ reference
    and extension sets with specific sensitivity, coverage, and sequence-identity
    thresholds
  repo: https://github.com/xinehc/sarg-curation
- name: Argo
  role: Downstream long-read ARG profiler that consumes the clustered SARG+ database
    (prot.fa, sarg.fa) as its default reference database
  repo: https://github.com/xinehc/argo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:3a942e08744262888d1b56e3d7850f5a9bfa22d1fc4c928dd98baf3c1b6fb20f
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
  - MMseqs2
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# near-identical-arg-grouping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A database-curation step in the SARG+ workflow that clusters near-identical antibiotic resistance gene protein sequences (e.g., blaOXA-1 and blaOXA-1042, differing by a single R101G change) into single subtypes using MMseqs2, so that read-based profiling can label reads at subtype level without ambiguity.

## When to use

Use when curating an ARG reference database for read-based species-level surveillance, where highly similar sequences within a subtype must be collapsed to avoid ambiguous subtype assignment from long or short reads.

## When NOT to use

- The application requires ORF-level annotation or detailed sequence typing, since clustering near-identical ARGs (e.g., blaOXA-1 with blaOXA-1042) lowers resolution to the subtype level
- The goal is detecting ARGs arising from point mutations, which are deliberately excluded from SARG+ because read-based profiling cannot distinguish SNPs from sequencing errors

## Inputs

- Curated core ARG protein FASTA from CARD v3.2.9, NDARO (2024-05-02.2), and SARG v3.2.1 (39 types, 1039 subtypes, 7041 sequences)
- RefSeq protein extension sequences sharing PGAP annotation evidence (BlastRules or Hidden Markov Models) with experimentally validated ARGs (476,546 sequences)
- Hierarchy table mapping type/subtype/sequence assignments

## Outputs

- Deduplicated protein FASTA (prot.fa) of 104,529 sequences for the SARG+ database
- Hierarchy table recording type/subtype/sequence assignments per cluster
- Component files sarg_ref.fa (experimentally validated sequences) and sarg_ext.fa (computationally derived homologs)

## How to apply

During SARG+ curation, merge protein reference sequences from CARD, NDARO, and SARG into a hierarchy of type (class/family), subtype (subclass/gene), and sequence, labeling multi-subtype sequences with an asterisk (e.g., cml*). After removing regulators, housekeeping genes, and point-mutation-derived ARGs (yielding 39 types, 1039 subtypes, 7041 sequences), group near-identical ARGs differing by few amino acids (e.g., a single R101G change) into single subtypes because read-based profiling cannot reliably distinguish such SNPs from sequencing errors. Deduplicate the reference set with MMseqs2 easy-cluster using '-s 7.5 -c 0.995 --min-seq-id 0.995 --cov-mode 0 --cluster-reassign', and the RefSeq-augmented extension set with '-s 7.5 -c 0.95 --min-seq-id 0.95 --cov-mode 0 --cluster-reassign'. Write the final protein FASTA (prot.fa) and a hierarchy table recording type/subtype/sequence per cluster; the curated SARG+ database must contain exactly 104,529 sequences spanning 39 types and 1053 subtypes, with 81 subtypes classified as Risk Rank I.

## Related tools

- **MMseqs2** (Protein sequence clustering (easy-cluster) used to deduplicate the SARG+ reference and extension sets with specific sensitivity, coverage, and sequence-identity thresholds) — https://github.com/xinehc/sarg-curation
- **Argo** (Downstream long-read ARG profiler that consumes the clustered SARG+ database (prot.fa, sarg.fa) as its default reference database) — https://github.com/xinehc/argo

## Examples

```
mmseqs easy-cluster sarg_ref.fa refCluster tmp -s 7.5 -c 0.995 --min-seq-id 0.995 --cov-mode 0 --cluster-reassign
```

## Evaluation signals

- Final SARG+ database contains exactly 104,529 protein sequences spanning 39 types and 1053 subtypes
- Exactly 81 subtypes in the final database are classified as Risk Rank I
- Near-identical variants such as blaOXA-1 and blaOXA-1042 (single R101G change) resolve to the same subtype label
- Sequences belonging to more than one subtype carry an asterisk suffix (e.g., cml*) in the hierarchy table
- misc/sarg.txt, misc/sarg_ref.txt, and misc/sarg_ext.txt counts are consistent with the summed sarg_ref.fa and sarg_ext.fa sequence counts

## Limitations

- Clustering highly similar ARGs lowers subtype resolution and is a drawback for applications requiring open reading frame (ORF)-based annotation or detailed sequence typing
- ARGs originating from point mutations are consistently excluded from the database because read-based profiling cannot reliably distinguish SNPs from sequencing errors
- The clustering parameters differ for reference (0.995 identity/coverage) versus extension (0.95 identity/coverage) sets, so results are sensitive to which set a sequence belongs to

## Evidence

- [methods] Group highly similar ARGs (e.g., blaOXA-1 and blaOXA-1042, differing by a single R101G amino-acid change) into single subtypes to avoid ambiguity: [evidence span withheld — anchor status: not_found]
- [methods] Deduplicate with MMseqs2 v15.6f45266 easy-cluster using '-s 7.5 -c 0.995 --min-seq-id 0.995 --cov-mode 0 --cluster-reassign' for the reference set: [evidence span withheld — anchor status: not_found]
- [readme] Removes duplicated and cross-mapped sequences by clustering.: [evidence span withheld — anchor status: external_doc]
