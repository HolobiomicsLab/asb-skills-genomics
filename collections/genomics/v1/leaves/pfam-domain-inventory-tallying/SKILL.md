---
name: pfam-domain-inventory-tallying
description: Use when building or auditing the Pfam domain inventory of a protein
  mapping database — i.e., when you must reproduce counts like 71,419 Pfam domains
  on 30,406 Swiss-Prot sequences, or populate domain-annotation tables from InterProScan
  output.
when_to_use_negative:
- You only need gnomAD/ClinVar variant loading into a live MetaDome instance — use
  the prebuilt mapping database CSV and populate_database.sh path instead of re-running
  domain annotation
- Your transcripts are not mapped to a Swiss-Prot protein (unmapped transcripts are
  excluded and displayed in gray, not annotated)
- You need domain annotation of non-human or non-GENCODE-matched proteomes; the inventory
  is defined only over the GENCODE-matched Swiss-Prot human sequences
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0623
tools:
- name: InterProScan
  role: Annotates the matched Swiss-Prot FASTA with Pfam-A domain instances whose
    records populate the interpro_domains table
- name: Pfam
  role: Pfam-A domain library (v30.0 in the published build; Pfam 37.4 in the v2 prebuild
    pipelines) supplying the domain definitions and consensus positions
- name: PostgreSQL
  role: Stores the mapping database, including domain annotations, in the interpro_domains
    table keyed by Swiss-Prot entries
- name: MetaDome
  role: The application whose mapping database is enriched with the domain inventory;
    loading uses metadome.batch_load via populate_database.sh
  repo: https://github.com/cmbi/metadome
- name: Python
  role: Parsing of InterProScan output and batch loading into PostgreSQL (MetaDome
    v3.5.1 codebase)
  repo: https://github.com/cmbi/metadome
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:4d830fb974dcd13ae9cb2cd3a4ec2183a36e9799edb3861c3807291e3fc0b5b3
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
  - InterProScan
  - MetaDome
  - Pfam
  - PostgreSQL
  - Python
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# pfam-domain-inventory-tallying

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate the Swiss-Prot sequences stored in a MetaDome mapping database with Pfam protein domain instances (via InterProScan) and tally the resulting domain inventory — total domain instances and distinct annotated sequences. This establishes the domain inventory underlying MetaDome's meta-domain aggregation.

## When to use

Use when building or auditing the Pfam domain inventory of a protein mapping database — i.e., when you must reproduce counts like 71,419 Pfam domains on 30,406 Swiss-Prot sequences, or populate domain-annotation tables from InterProScan output.

## When NOT to use

- You only need gnomAD/ClinVar variant loading into a live MetaDome instance — use the prebuilt mapping database CSV and populate_database.sh path instead of re-running domain annotation
- Your transcripts are not mapped to a Swiss-Prot protein (unmapped transcripts are excluded and displayed in gray, not annotated)
- You need domain annotation of non-human or non-GENCODE-matched proteomes; the inventory is defined only over the GENCODE-matched Swiss-Prot human sequences

## Inputs

- FASTA of matched human Swiss-Prot canonical and isoform sequences (GENCODE Basic translation matches from the 'proteins' table)
- InterProScan v5.20-59.0 with the Pfam-A v30.0 domain library
- PostgreSQL mapping database ('proteins' and 'interpro_domains' tables)

## Outputs

- Populated 'interpro_domains' table with one record per Pfam domain instance, keyed by Swiss-Prot sequence entry
- Domain inventory counts: total Pfam domain instances and number of distinct Swiss-Prot sequences with at least one domain (expected: 71,419 domains over 30,406 sequences)

## How to apply

Retrieve the matched human Swiss-Prot canonical and isoform sequences (the identical-sequence matches to GENCODE Basic translations recorded in the 'proteins' table) as FASTA, excluding sequences failing the start/stop codon and cDNA-translation-match checks. Run InterProScan (v5.20-59.0 in the published build) over these sequences with the Pfam-A v30.0 domain library to produce one record per domain instance. Parse the InterProScan output and load each annotation into the 'interpro_domains' table of the PostgreSQL mapping database, keyed by Swiss-Prot sequence entry (in the v2 codebase, bulk loading is done through metadome.batch_load via populate_database.sh). Tally the inventory: count total Pfam domain instances and the number of distinct Swiss-Prot sequences carrying at least one domain, and compare against the reported 71,419 domains over 30,406 sequences. The rationale is that projecting variation onto Pfam domain consensus positions is what later lets variants in different genes be compared at evolutionarily equivalent residues.

## Related tools

- **InterProScan** (Annotates the matched Swiss-Prot FASTA with Pfam-A domain instances whose records populate the interpro_domains table)
- **Pfam** (Pfam-A domain library (v30.0 in the published build; Pfam 37.4 in the v2 prebuild pipelines) supplying the domain definitions and consensus positions)
- **PostgreSQL** (Stores the mapping database, including domain annotations, in the interpro_domains table keyed by Swiss-Prot entries)
- **MetaDome** (The application whose mapping database is enriched with the domain inventory; loading uses metadome.batch_load via populate_database.sh) — https://github.com/cmbi/metadome
- **Python** (Parsing of InterProScan output and batch loading into PostgreSQL (MetaDome v3.5.1 codebase)) — https://github.com/cmbi/metadome

## Examples

```
./populate_database.sh --data-dir /path/to/csv_dir --file MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz
```

## Evaluation signals

- Total Pfam domain instance count equals 71,419 and distinct annotated Swiss-Prot sequences equals 30,406 (within expected tolerance)
- The InterProScan output contains exactly one record per domain instance, each keyed to an existing Swiss-Prot sequence entry in the 'proteins' table
- Every annotated Swiss-Prot sequence in 'interpro_domains' traces back to an identical-sequence GENCODE Basic translation match
- In v2 releases, recomputed aggregation statistics in the quality control report match the released dataset and the tolerance track reproduces codon for codon

## Limitations

- Domain inventory size depends on the reference versions: the published 71,419/30,406 figures correspond to Pfam-A v30.0 and GENCODE-matched Swiss-Prot, while the current v2 prebuilds use Pfam 37.4, UniProt 2025-01, and GENCODE v19/v45, so counts will differ across releases
- Only variants/transcripts mapped to a Swiss-Prot protein can be used downstream; sequences failing the ATG start codon, terminal stop codon, or Biopython cDNA-translation-match checks are excluded
- Aggregation via meta-domains only encapsulates general biological or molecular functions attributed to the domain and loses context such as haplotype information or protein-protein interactions

## Evidence
