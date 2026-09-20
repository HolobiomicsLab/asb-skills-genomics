---
name: pgap-annotation-evidence-matching
description: Use when building or extending a protein reference database for ARG surveillance
  and you need computationally derived homologs of validated genes, matching RefSeq
  records by their PGAP annotation evidence rather than by sequence similarity alone.
when_to_use_negative:
- The application requires ORF-level annotation or detailed sequence typing that distinguishes
  near-identical variants such as blaOXA-1 versus blaOXA-1042 — SARG+ clusters highly
  similar ARGs into single subtypes.
- You need to detect ARGs arising from point mutations — read-based profiling cannot
  reliably distinguish SNPs from sequencing errors, so point-mutation-derived ARGs
  are excluded from SARG+.
- Starting from raw metagenomic reads rather than reference curation — use Argo or
  Lynx profiling pipelines instead of rebuilding the database.
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3697
tools:
- name: MMseqs2
  role: Deduplicates reference (99.5% identity/cov) and extension (95% identity/cov)
    sequence sets via easy-cluster
- name: diamond
  role: Builds a BLAST database from refseq_protein .faa.gz to index annotation lookups
- name: blast
  role: blastdbcmd extracts sequences from downloaded nr and env_nr BLAST databases
- name: Argo
  role: Downstream long-read ARG profiler that consumes SARG+ as its default database
  repo: https://github.com/xinehc/argo
- name: NCBI PGAP
  role: Annotation pipeline whose BlastRules/HMM evidence accessions define which
    RefSeq homologs enter the extension set
  repo: https://github.com/ncbi/pgap
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:70c7c4fc599fcc16b7937dbf89933352100062986460f730a91c3293d6ca6352
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
  - MMseqs2
  - NCBI PGAP
  - blast
  - diamond
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pgap-annotation-evidence-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Augments a curated ARG reference database (SARG+) with RefSeq protein sequences that are annotated by the same NCBI PGAP evidence (BlastRules or Hidden Markov Models) as experimentally validated ARGs, expanding coverage beyond the one-or-few representative sequences per gene found in CARD, NDARO, and SARG.

## When to use

Use when building or extending a protein reference database for ARG surveillance and you need computationally derived homologs of validated genes, matching RefSeq records by their PGAP annotation evidence rather than by sequence similarity alone.

## When NOT to use

- The application requires ORF-level annotation or detailed sequence typing that distinguishes near-identical variants such as blaOXA-1 versus blaOXA-1042 — SARG+ clusters highly similar ARGs into single subtypes.
- You need to detect ARGs arising from point mutations — read-based profiling cannot reliably distinguish SNPs from sequencing errors, so point-mutation-derived ARGs are excluded from SARG+.
- Starting from raw metagenomic reads rather than reference curation — use Argo or Lynx profiling pipelines instead of rebuilding the database.

## Inputs

- RefSeq refseq_protein wp_protein .faa.gz and .gpff.gz records (plasmid, archaea, bacteria)
- NDARO ReferenceGeneCatalog.txt and AMRProt.fa
- CARD v3.2.9 protein reference sequences
- SARG v3.2.1 protein sequences
- NCBI nr and env_nr protein databases
- KEGG KOfam ko_list and profiles
- sarg. curation spec and reference/reference.fasta

## Outputs

- sarg.fa — combined SARG+ reference and extension protein FASTA
- sarg_ref.fa — experimentally validated reference component
- sarg_ext.fa — computationally derived extension component
- misc/summary.tsv — types, subtypes, and sources of reference sequences
- misc/evidence.tsv — PGAP evidence used for extension construction
- misc/sarg.txt, misc/sarg_ref.txt, misc/sarg_ext.txt — per-ARG counts

## How to apply

First curate the SARG+ reference from CARD v3.2.9, NDARO (2024-05-02.2) and SARG v3.2.1, removing regulators, housekeeping genes, and point-mutation ARGs to yield a core of 39 types, 1039 subtypes, and 7041 sequences. Then parse RefSeq 'wp_protein' records (plasmid, archaea, bacteria kingdoms) with their .gpff GenBank annotations, extracting the 'Evidence-For-Name-Assignment' structured comment to obtain each protein's evidence accession and source identifier. Match extension candidates whose PGAP evidence (BlastRules or Hidden Markov Models, cross-checked via KOfam profiles) is identical to that used for experimentally validated SARG+ reference sequences, via the n3-parse-evidence notebook. Deduplicate the resulting reference and extension sets separately with MMseqs2 easy-cluster (reference: '-s 7.5 -c 0.995 --min-seq-id 0.995 --cov-mode 0 --cluster-reassign'; extension: '-s 7.5 -c 0.95 --min-seq-id 0.95 --cov-mode 0 --cluster-reassign'), producing a final database of 104,529 sequences across 39 types and 1053 subtypes.

## Related tools

- **MMseqs2** (Deduplicates reference (99.5% identity/cov) and extension (95% identity/cov) sequence sets via easy-cluster)
- **diamond** (Builds a BLAST database from refseq_protein .faa.gz to index annotation lookups)
- **blast** (blastdbcmd extracts sequences from downloaded nr and env_nr BLAST databases)
- **Argo** (Downstream long-read ARG profiler that consumes SARG+ as its default database) — https://github.com/xinehc/argo
- **NCBI PGAP** (Annotation pipeline whose BlastRules/HMM evidence accessions define which RefSeq homologs enter the extension set) — https://github.com/ncbi/pgap

## Examples

```
diamond makedb --quiet --in tmp/refseq_protein.faa.gz --db tmp/refseq_protein
```

## Evaluation signals

- Final curated database contains exactly 104,529 protein sequences spanning 39 types and 1053 subtypes
- Core pre-extension reference contains 39 types, 1039 subtypes, and 7041 sequences after removing regulators, housekeeping genes, and point-mutation ARGs
- 81 of the 1053 subtypes are classified as Risk Rank I
- misc/evidence.tsv accounts for every PGAP evidence accession (BlastRules or HMM) used to derive extension sequences
- Sequences belonging to more than one subtype are consistently labeled with an asterisk (e.g., cml*) in the hierarchy table

## Limitations

- The extension is limited to RefSeq proteins sharing PGAP annotation evidence with validated ARGs; ARGs present in RefSeq only under other evidence types are missed.
- Highly similar ARGs are collapsed into single subtypes (e.g., blaOXA-1 and blaOXA-1042 grouped as blaOXA), lowering resolution for ORF-based annotation or detailed sequence typing.
- Extracting sequences and annotation evidence from nr, env_nr, and refseq_protein is time-consuming and requires parallelized blastdbcmd and diamond makedb runs.

## Evidence

- [readme] RefSeq PGAP evidence extension: [evidence span withheld — anchor status: not_found]
- [other] MMseqs2 deduplication parameters: [evidence span withheld — anchor status: not_found]
