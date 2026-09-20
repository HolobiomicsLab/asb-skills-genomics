---
name: transcript-proteome-mapping-verification
description: Use when you need to verify or rebuild the mapping between genome-build
  transcript translations (e.g. GENCODE Basic) and reference proteome sequences (Swiss-Prot/UniProt),
  or to reproduce the linked Pfam domain inventory and its reported counts.
when_to_use_negative:
- Your goal is downstream variant-tolerance or meta-domain analysis rather than validating
  the transcript-to-protein mapping and domain inventory itself
- Input is RNA-seq expression quantification or variant VCFs — this skill operates
  on transcript translations and reference protein sequences
- You already have a released prebuilt mapping database (e.g. from Zenodo record 19376150)
  and only need to load it via populate_database.sh rather than rebuild the mapping
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0085
tools:
- name: BLAST
  role: Protein–Protein BLAST v2.2.31+ matches each GENCODE Basic translation to its
    identical Swiss-Prot canonical/isoform sequence
- name: InterProScan
  role: InterProScan v5.20-59.0 with Pfam-A v30.0 annotates Pfam domain instances
    on the matched Swiss-Prot sequences
- name: Pfam
  role: Pfam-A library supplies the domain definitions projected onto Swiss-Prot sequences
    and domain consensus positions
- name: PostgreSQL
  role: PostgreSQL v10 relational database stores the mapping (proteins table) and
    domain annotations (interpro_domains table)
- name: Biopython
  role: Biopython's translate function verifies that transcript cDNA matches the GENCODE
    translation; mismatches are excluded
- name: MetaDome
  role: The mapping database being verified; its repository provides the prebuild
    pipelines and populate_database.sh loader
  repo: https://github.com/cmbi/metadome
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:f86d457a52795392728f28d6e9224a30edd8ada743969f47d9f48551bdf16b12
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
  - InterProScan
  - MetaDome
  - Pfam
  - PostgreSQL
  - Python
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# transcript-proteome-mapping-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce and verify the GENCODE-transcript to Swiss-Prot protein mapping and its Pfam domain inventory — the backbone of MetaDome's mapping database — by BLAST-matching GENCODE Basic translations to UniProt sequences, applying the paper's translation-quality filters, and recomputing domain counts with InterProScan/Pfam-A.

## When to use

Use when you need to verify or rebuild the mapping between genome-build transcript translations (e.g. GENCODE Basic) and reference proteome sequences (Swiss-Prot/UniProt), or to reproduce the linked Pfam domain inventory and its reported counts.

## When NOT to use

- Your goal is downstream variant-tolerance or meta-domain analysis rather than validating the transcript-to-protein mapping and domain inventory itself
- Input is RNA-seq expression quantification or variant VCFs — this skill operates on transcript translations and reference protein sequences
- You already have a released prebuilt mapping database (e.g. from Zenodo record 19376150) and only need to load it via populate_database.sh rather than rebuild the mapping

## Inputs

- GENCODE Basic full-length protein-coding transcript translations (FASTA)
- Swiss-Prot human canonical and isoform sequences (42,130 FASTA entries in the paper)
- Transcript cDNA sequences for translation-consistency checks
- Pfam-A domain library (v30.0 in the paper)
- InterProScan output (TSV) of Pfam domain instances

## Outputs

- GENCODE-transcript to Swiss-Prot sequence mapping (proteins table, PostgreSQL)
- Pfam domain instance inventory in the interpro_domains table (71,419 domains over 30,406 Swiss-Prot sequences in the paper)
- Mapping coverage statistics: 32,595,355 unique genomic positions linked to 19,226,961 Swiss-Prot residues

## How to apply

Run Protein–Protein BLAST (v2.2.31+ in the paper) for each full-length protein-coding GENCODE Basic translation against the 42,130 Swiss-Prot human canonical/isoform sequences, keeping identical-sequence matches. Apply the paper's exclusion filters: drop sequences that do not start with an ATG start codon or do not end with a stop codon, and check each transcript's cDNA against its GENCODE translation via Biopython's translate function, excluding non-identical cases. Store accepted links in the 'proteins' table of a PostgreSQL v10 mapping database, then annotate the matched Swiss-Prot sequences with InterProScan v5.20-59.0 using the Pfam-A v30.0 library and load each domain instance into the 'interpro_domains' table. Finally tally the inventory — distinct Swiss-Prot sequences carrying at least one domain and total domain instances — and compare against the reported counts.

## Related tools

- **BLAST** (Protein–Protein BLAST v2.2.31+ matches each GENCODE Basic translation to its identical Swiss-Prot canonical/isoform sequence)
- **InterProScan** (InterProScan v5.20-59.0 with Pfam-A v30.0 annotates Pfam domain instances on the matched Swiss-Prot sequences)
- **Pfam** (Pfam-A library supplies the domain definitions projected onto Swiss-Prot sequences and domain consensus positions)
- **PostgreSQL** (PostgreSQL v10 relational database stores the mapping (proteins table) and domain annotations (interpro_domains table))
- **Biopython** (Biopython's translate function verifies that transcript cDNA matches the GENCODE translation; mismatches are excluded)
- **MetaDome** (The mapping database being verified; its repository provides the prebuild pipelines and populate_database.sh loader) — https://github.com/cmbi/metadome

## Examples

```
./populate_database.sh --data-dir /path/to/data --file MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz
```

## Evaluation signals

- Recomputed Pfam inventory matches the reported 71,419 domain instances linked to 30,406 Swiss-Prot sequences
- Mapping coverage matches the reported 42,116 of 56,319 GENCODE Basic transcripts (19,728 genes) linked to 33,492 of 42,130 Swiss-Prot sequences
- Position-level totals match the reported 32,595,355 unique genomic positions linked to 19,226,961 Swiss-Prot residues
- The InterProScan parse yields exactly one interpro_domains record per domain instance, keyed to its Swiss-Prot sequence entry
- All retained transcript-protein pairs are identical-sequence matches whose cDNA translation (Biopython) is identical to the GENCODE translation, and sequences start with ATG and end with a stop codon

## Limitations

- Only GENCODE Basic full-length protein-coding translations passing the start/stop-codon and cDNA-translation checks are mapped; transcripts not matching a Swiss-Prot protein cannot be used in the analysis
- Reported counts are specific to the paper's versions (GENCODE Basic, InterProScan v5.20-59.0, Pfam-A v30.0); current releases use different versions (e.g. UniProt-2025-01, Pfam-37.4) so counts will differ
- The mapping table holds tens of millions of rows per genome build, so loading requires a long run and substantial disk headroom for the database plus indexes

## Evidence

- [methods] These mappings are created with Protein–Protein BLAST v2.2.31 + for each protein‐coding translation in the GENCODE Basic set: [evidence span withheld — anchor status: not_found]
- [methods] We checked if the complementary DNA (cDNA) sequence of the transcripts match the GENCODE translation via Biopython's translate function: [evidence span withheld — anchor status: not_found]
