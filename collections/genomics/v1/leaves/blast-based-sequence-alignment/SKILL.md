---
name: blast-based-sequence-alignment
description: Use when protein-coding transcript translations must be linked to reference
  protein sequences (e.g., GENCODE Basic translations vs. UniProtKB/Swiss-Prot human
  canonical or isoform entries) to build genomic-to-protein position mappings.
when_to_use_negative:
- Non-identical or cross-species homology is needed — the MetaDome pipeline retains
  only identical sequence matches, so approximate homologs cannot be recovered here.
- Input transcripts lack a valid ORF (no ATG start or no stop codon) or their Biopython
  cDNA translation disagrees with the annotated translation — these are excluded before
  BLAST rather than realigned.
- The goal is variant tolerance or meta-domain analysis directly — that requires the
  downstream MetaDome mapping database and Pfam domain projection, not a fresh BLAST
  step.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: BLAST
  role: Protein–Protein BLAST v2.2.31+ (blastp) aligns each GENCODE Basic protein-coding
    translation against human Swiss-Prot canonical/isoform sequences; identical sequence
    matches are retained.
- name: Biopython
  role: Translates each transcript's cDNA to verify it matches the GENCODE translation;
    transcripts with mismatching translations are excluded before BLAST.
- name: ClustalW2
  role: Builds v2.1 alignments for each identical match to derive the nucleotide genomic-to-protein
    position mapping stored in the database.
- name: PostgreSQL
  role: Stores transcript info ('genes'), matched protein info ('proteins'), and per-nucleotide
    genomic-to-protein mappings ('mappings'), with all lookup fields indexed.
- name: Python
  role: Glue code for translation checks, BLAST orchestration, and writing mapping/QC
    outputs (MetaDome is developed in Python v3.5.1).
  repo: https://github.com/cmbi/metadome
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:5b0823ff83bd9e4d5c1831343c8b4a207a755a82f31d0c1d6e998dbe9ce658f9
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
  - ClustalW2
  - MetaDome
  - PostgreSQL
  - Python
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# blast-based-sequence-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align protein-coding transcript translations to a curated reference proteome with Protein–Protein BLAST to establish transcript-to-protein identity links. In MetaDome, this maps GENCODE Basic translations onto Swiss-Prot canonical/isoform sequences as the basis for genomic-to-protein position mappings.

## When to use

Use when protein-coding transcript translations must be linked to reference protein sequences (e.g., GENCODE Basic translations vs. UniProtKB/Swiss-Prot human canonical or isoform entries) to build genomic-to-protein position mappings.

## When NOT to use

- Non-identical or cross-species homology is needed — the MetaDome pipeline retains only identical sequence matches, so approximate homologs cannot be recovered here.
- Input transcripts lack a valid ORF (no ATG start or no stop codon) or their Biopython cDNA translation disagrees with the annotated translation — these are excluded before BLAST rather than realigned.
- The goal is variant tolerance or meta-domain analysis directly — that requires the downstream MetaDome mapping database and Pfam domain projection, not a fresh BLAST step.

## Inputs

- GENCODE Basic protein-coding transcript cDNAs and translations (e.g., GENCODE release 19, GRCh37.p13)
- UniProtKB/Swiss-Prot human canonical and isoform protein sequences (e.g., Release 2016_09)
- Reference BLAST database built from the Swiss-Prot target sequences

## Outputs

- Set of transcript-to-Swiss-Prot identical-sequence match pairs (e.g., 42,116 of 56,319 transcripts linked to 33,492 of 42,130 Swiss-Prot sequences)
- Genomic-position-to-protein-residue mappings stored in a PostgreSQL 'mappings' table (e.g., 32,595,355 unique genomic positions linked to 19,226,961 residues)
- Quality-control report with recomputed mapping statistics

## How to apply

Download the transcript set (GENCODE release 19 GRCh37.p13 Basic protein-coding transcripts) and the reference proteome (UniProtKB/Swiss-Prot Release 2016_09 human canonical and isoform sequences). First apply quality filters: exclude translations that do not start with an ATG start codon (encoding methionine) or end with a stop codon, then verify each remaining transcript by translating its cDNA with Biopython's translate function and exclude any transcript whose cDNA translation is not identical to the GENCODE translation. Run Protein–Protein BLAST v2.2.31+ (blastp) for each protein-coding translation against the human canonical and isoform Swiss-Prot sequences and retain only identical sequence matches — non-identical hits are discarded. For each identical match, build a ClustalW2 v2.1 alignment and store the nucleotide's genomic-to-protein position mapping in a PostgreSQL 'mappings' table (with transcript and protein info in 'genes' and 'proteins' tables, indexed on lookup fields). Recompute mapping totals and compare against published figures as a quality-control gate.

## Related tools

- **BLAST** (Protein–Protein BLAST v2.2.31+ (blastp) aligns each GENCODE Basic protein-coding translation against human Swiss-Prot canonical/isoform sequences; identical sequence matches are retained.)
- **Biopython** (Translates each transcript's cDNA to verify it matches the GENCODE translation; transcripts with mismatching translations are excluded before BLAST.)
- **ClustalW2** (Builds v2.1 alignments for each identical match to derive the nucleotide genomic-to-protein position mapping stored in the database.)
- **PostgreSQL** (Stores transcript info ('genes'), matched protein info ('proteins'), and per-nucleotide genomic-to-protein mappings ('mappings'), with all lookup fields indexed.)
- **Python** (Glue code for translation checks, BLAST orchestration, and writing mapping/QC outputs (MetaDome is developed in Python v3.5.1).) — https://github.com/cmbi/metadome

## Examples

```
blastp -query gencode_basic_translations.fasta -db swissprot_human_canonical_isoform -outfmt '6 qseqid sseqid pident length' -out identical_matches.tsv
```

## Evaluation signals

- Recomputed mapping statistics reproduce the published figures: 42,116 of 56,319 GENCODE Basic transcripts (19,728 genes) linked to 33,492 of 42,130 Swiss-Prot human canonical/isoform sequences.
- All retained transcripts start with an ATG start codon (methionine) and end with a stop codon; any failing this check was excluded.
- Every retained transcript's Biopython cDNA translation is identical to the GENCODE translation.
- Only exact (identical-sequence) BLAST matches are registered in the 'proteins'/'mappings' tables — spot-check that no non-identical hit entered the database.
- Mapping totals are internally consistent: 32,595,355 unique genomic positions linked to 19,226,961 Swiss-Prot residues, with positions and residues traceable through ClustalW2 alignments.

## Limitations

- Only identical sequence matches are kept, so transcripts without a Swiss-Prot identity match cannot be used in downstream analysis (displayed in gray in MetaDome).
- Results are tied to specific dataset versions (GENCODE 19 GRCh37.p13, Swiss-Prot 2016_09, BLAST v2.2.31+); substituting releases changes the mapping counts.
- Only full-length protein-coding GENCODE Basic transcripts are eligible — partial transcripts or non-protein-coding transcripts are out of scope.

## Evidence

- [methods] These mappings are created with Protein–Protein BLAST v2.2.31 + for each protein‐coding translation in the GENCODE Basic set: [evidence span withheld — anchor status: not_found]
- [methods] start/stop codon exclusion: "We exclude sequences that do not start with a start codon (i.e., ATG encoding for methionine), or end with a stop codon"
- [methods] cDNA translation verification: [evidence span withheld — anchor status: not_found]
- [results] published mapping statistics: "42,116 of the 56,319 full‐length protein‐coding GENCODE Basic transcripts for 19,728 human genes are linked to 33,492 of the 42,130 Swiss‐Prot human canonical or isoform sequences"
- [results] unmapped transcripts excluded from analysis: "Only the transcripts that are mapped to a Swiss‐Prot protein can be used in the analysis, the others are displayed in gray"
