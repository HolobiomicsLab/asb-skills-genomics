---
name: protein-sequence-clustering-deduplication
description: Use when merging heterogeneous protein reference databases (e.g., CARD,
  NDARO, SARG, RefSeq homologs) and redundant near-identical sequences must be collapsed
  into a consistent type/subtype hierarchy before read mapping.
when_to_use_negative:
- Applications requiring ORF-based annotation or detailed sequence typing, since clustering
  highly similar ARGs (e.g., blaOXA-1 with blaOXA-1042) intentionally lowers resolution
- Analyses targeting point-mutation-derived ARG variants, which are excluded from
  the database because read-based profiling cannot reliably distinguish SNPs from
  sequencing errors
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
tools:
- name: MMseqs2
  role: Sequence clustering/deduplication via easy-cluster with tuned sensitivity,
    coverage, and min-seq-id thresholds
  repo: https://github.com/xinehc/sarg-curation
- name: Argo
  role: Downstream consumer of the deduplicated SARG+ database for long-read ARG profiling
  repo: https://github.com/xinehc/argo
- name: diamond
  role: Builds BLAST databases (diamond makedb) for RefSeq protein screening and database
    indexing
  repo: https://github.com/xinehc/sarg-curation
- name: seqkit
  role: FASTA sequence utilities during curation
  repo: https://github.com/xinehc/sarg-curation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:9b09c00c3984081bbdbf8560dcfff0078a77ad9ba8b2f4a489033712a52d55d4
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
  - diamond
  - seqkit
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# protein-sequence-clustering-deduplication

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Deduplicate and organize large protein reference sets into non-redundant clusters with strict sequence-identity thresholds, as done to curate the SARG+ ARG database (104,529 sequences) for species-level read-based antibiotic resistance surveillance.

## When to use

Use when merging heterogeneous protein reference databases (e.g., CARD, NDARO, SARG, RefSeq homologs) and redundant near-identical sequences must be collapsed into a consistent type/subtype hierarchy before read mapping.

## When NOT to use

- Applications requiring ORF-based annotation or detailed sequence typing, since clustering highly similar ARGs (e.g., blaOXA-1 with blaOXA-1042) intentionally lowers resolution
- Analyses targeting point-mutation-derived ARG variants, which are excluded from the database because read-based profiling cannot reliably distinguish SNPs from sequencing errors

## Inputs

- Protein FASTA reference sequences from CARD v3.2.9, NDARO (2024-05-02.2), and SARG v3.2.1
- RefSeq protein sequences annotated with PGAP evidence (BlastRules or Hidden Markov Models) matching experimentally validated ARGs (~476,546 extension sequences)
- Hierarchy/metadata table mapping sequences to ARG type and subtype (e.g., sarg., summary.tsv)

## Outputs

- Curated non-redundant protein FASTA (prot.fa / sarg.fa; 104,529 sequences across 39 types and 1053 subtypes)
- Reference component FASTA (sarg_ref.fa) of experimentally validated sequences
- Extension component FASTA (sarg_ext.fa) of computationally derived homologs
- Hierarchy table recording type/subtype/sequence assignments per cluster

## How to apply

Merge source protein FASTA files (CARD v3.2.9, NDARO 2024-05-02.2, SARG v3.2.1, and RefSeq PGAP-annotated homologs) into a consistent hierarchy of type (class/family), subtype (subclass/gene), and sequence, first removing regulators, housekeeping genes, and point-mutation-derived ARGs. Group highly similar ARGs (e.g., blaOXA-1 and blaOXA-1042, differing by a single R101G change) into single subtypes to avoid ambiguous classification. Then deduplicate with MMseqs2 easy-cluster using '-s 7.5 -c 0.995 --min-seq-id 0.995 --cov-mode 0 --cluster-reassign' for the reference set (experimentally validated sequences) and '-s 7.5 -c 0.95 --min-seq-id 0.95 --cov-mode 0 --cluster-reassign' for the extension set (computationally derived homologs). Rationale: the higher-identity threshold preserves validated reference granularity while the lower threshold compresses the large homolog extension. Label sequences belonging to more than one subtype with an asterisk (e.g., cml*) and record type/subtype/sequence assignments per cluster in a hierarchy table.

## Related tools

- **MMseqs2** (Sequence clustering/deduplication via easy-cluster with tuned sensitivity, coverage, and min-seq-id thresholds) — https://github.com/xinehc/sarg-curation
- **Argo** (Downstream consumer of the deduplicated SARG+ database for long-read ARG profiling) — https://github.com/xinehc/argo
- **diamond** (Builds BLAST databases (diamond makedb) for RefSeq protein screening and database indexing) — https://github.com/xinehc/sarg-curation
- **seqkit** (FASTA sequence utilities during curation) — https://github.com/xinehc/sarg-curation

## Examples

```
mmseqs easy-cluster ref.fa ref_cluster tmp --min-seq-id 0.995 -c 0.995 --cov-mode 0 --cluster-reassign -s 7.5
```

## Evaluation signals

- Final curated database contains exactly 104,529 protein sequences
- Database spans 39 types and 1053 subtypes, of which 81 subtypes are classified as Risk Rank I
- Reference set clustered with --min-seq-id 0.995 / -c 0.995 and extension set with --min-seq-id 0.95 / -c 0.95 (MMseqs2 -s 7.5, --cov-mode 0, --cluster-reassign)
- Core (pre-extension) database contains 39 types, 1039 subtypes and 7041 sequences after removing regulators, housekeeping genes, and point-mutation ARGs
- Every cluster has a recorded type/subtype assignment in the hierarchy table; multi-subtype sequences carry an asterisk label (e.g., cml*)

## Limitations

- Clustering highly similar ARGs into single subtypes lowers resolution and could be a drawback for applications requiring ORF-based annotation or detailed sequence typing
- ARGs originating from point mutations are consistently excluded from the database due to challenges distinguishing SNPs from sequencing errors in read-based profiling
- Extracting RefSeq sequences and annotation evidence can be time-consuming; blastdbcmd and diamond makedb should be run in parallel on multiple cores

## Evidence

- [other] task_004 workflow step 4: [evidence span withheld — anchor status: not_found]
- [other] task_004 finding: [evidence span withheld — anchor status: not_found]
- [readme] sarg-curation README: [evidence span withheld — anchor status: not_found]
