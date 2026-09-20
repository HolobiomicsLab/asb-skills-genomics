---
name: swissprot-sequence-set-preparation
description: Use when building a genome-to-protein mapping database (e.g. MetaDome
  prebuild) and you need the filtered Swiss-Prot FASTA subset of GENCODE-matched canonical/isoform
  sequences as input to domain annotation.
when_to_use_negative:
- The mapping database is only being consumed (e.g. loading a released prebuild CSV
  via populate_database.sh) rather than rebuilt from raw GENCODE/UniProt sources.
- Your protein reference is not Swiss-Prot/UniProt-based or your transcripts come
  from a non-GENCODE annotation, since the identical-sequence matching logic assumes
  GENCODE Basic translations.
- You need domain annotations for sequences without a GENCODE transcript match; only
  transcripts mapped to a Swiss-Prot protein are usable in this workflow.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0623
- http://edamontology.org/topic_0080
tools:
- name: BLAST
  role: Protein-Protein BLAST v2.2.31+ aligns each GENCODE Basic protein-coding translation
    to Swiss-Prot sequences to find identical-sequence matches
- name: Biopython
  role: Its translate function verifies that transcript cDNA sequences match the GENCODE
    translations before a sequence is accepted
- name: MetaDome
  role: The prepared Swiss-Prot sequence set populates the 'proteins' table of the
    MetaDome PostgreSQL mapping database built by the prebuild pipelines
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL
  role: Stores the mapping database including the matched Swiss-Prot protein entries
    used downstream for domain annotation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:5a94102fccd835a70871e21c39c95f7d9bb6ff13d6339e5ab47284fdf0d4d560
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/laurensvdwiel/metadome
  repo_url: https://github.com/cmbi/metadome
  tools:
  - BLAST
  - Biopython
  - MetaDome
  - PostgreSQL
  - Python
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0623
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# swissprot-sequence-set-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepares the Swiss-Prot human canonical and isoform sequence set that will serve as the protein reference for downstream Pfam/InterProScan domain annotation in a MetaDome-style mapping database, by selecting the Swiss-Prot entries whose sequences are identical matches to GENCODE Basic translations and applying sequence-integrity filters.

## When to use

Use when building a genome-to-protein mapping database (e.g. MetaDome prebuild) and you need the filtered Swiss-Prot FASTA subset of GENCODE-matched canonical/isoform sequences as input to domain annotation.

## When NOT to use

- The mapping database is only being consumed (e.g. loading a released prebuild CSV via populate_database.sh) rather than rebuilt from raw GENCODE/UniProt sources.
- Your protein reference is not Swiss-Prot/UniProt-based or your transcripts come from a non-GENCODE annotation, since the identical-sequence matching logic assumes GENCODE Basic translations.
- You need domain annotations for sequences without a GENCODE transcript match; only transcripts mapped to a Swiss-Prot protein are usable in this workflow.

## Inputs

- Human Swiss-Prot canonical and isoform sequences (FASTA, from UniProt)
- Full-length protein-coding GENCODE Basic transcript translations (FASTA)
- GENCODE transcript cDNA sequences (for translation verification)

## Outputs

- Filtered FASTA of matched Swiss-Prot human sequences (input to InterProScan/Pfam annotation)
- GENCODE transcript-to-Swiss-Prot protein mapping records for the 'proteins' table of the PostgreSQL mapping database

## How to apply

Retrieve the human Swiss-Prot canonical and isoform sequences from UniProt and the full-length protein-coding GENCODE Basic translations. Align each GENCODE protein-coding translation against Swiss-Prot with Protein-Protein BLAST (v2.2.31+ in the article) to identify identical-sequence matches, linking 42,116 of 56,319 GENCODE Basic transcripts to 33,492 of the 42,130 Swiss-Prot human sequences. Apply integrity filters: exclude sequences that do not start with a start codon (ATG, methionine) or do not end with a stop codon, and verify each transcript's cDNA translates exactly to the GENCODE translation using Biopython's translate function, excluding mismatches. Emit the filtered, matched Swiss-Prot sequences as FASTA (the records stored in the 'proteins' table of the PostgreSQL mapping database) so a downstream InterProScan/Pfam step can annotate domain instances over exactly this set.

## Related tools

- **BLAST** (Protein-Protein BLAST v2.2.31+ aligns each GENCODE Basic protein-coding translation to Swiss-Prot sequences to find identical-sequence matches)
- **Biopython** (Its translate function verifies that transcript cDNA sequences match the GENCODE translations before a sequence is accepted)
- **MetaDome** (The prepared Swiss-Prot sequence set populates the 'proteins' table of the MetaDome PostgreSQL mapping database built by the prebuild pipelines) — https://github.com/cmbi/metadome
- **PostgreSQL** (Stores the mapping database including the matched Swiss-Prot protein entries used downstream for domain annotation)

## Examples

```
blastp -query gencode_basic_translations.fasta -db swissprot_human -outfmt 6 -out gencode_swissprot_matches.tsv
```

## Evaluation signals

- The matched set should cover 42,116 of 56,319 full-length protein-coding GENCODE Basic transcripts for 19,728 human genes, linked to 33,492 of the 42,130 Swiss-Prot human canonical or isoform sequences.
- Every retained Swiss-Prot sequence must correspond to an identical-sequence BLAST match to a GENCODE Basic translation — no partial or mismatched alignments accepted.
- Sequences failing the start-codon (ATG) or stop-codon check, or whose cDNA translation mismatches the GENCODE translation via Biopython's translate, must be excluded from the FASTA.
- The resulting mappings should contain 32,595,355 unique genomic positions linked to 19,226,961 residues in Swiss-Prot protein sequences.
- One record per matched sequence should exist in the 'proteins' table, keyed by the corresponding Swiss-Prot entry.

## Limitations

- Only GENCODE Basic full-length protein-coding transcripts that map to a Swiss-Prot protein can be used in downstream analysis; unmapped transcripts are excluded.
- cDNA sequences whose Biopython translation does not exactly match the GENCODE translation are excluded, so transcripts with annotation discrepancies are dropped.
- Version skew matters: the article used GENCODE with Swiss-Prot 42,130-sequence release and BLAST v2.2.31+; the current MetaDome v2.0 release pairs GRCh37/GENCODE-v19 and GRCh38/GENCODE-v45 with UniProt-2025-01 and Pfam-37.4, so reproducing the original counts (71,419 domains / 30,406 sequences) requires the original versions.

## Evidence

- [methods] These mappings are created with Protein–Protein BLAST v2.2.31 + for each protein‐coding translation in the GENCODE Basic set: [evidence span withheld — anchor status: not_found]
- [methods] We checked if the complementary DNA (cDNA) sequence of the transcripts match the GENCODE translation via Biopython's translate function, if they are not identical then these are excluded: [evidence span withheld — anchor status: not_found]
