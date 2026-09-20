---
name: genomic-to-protein-position-variant-mapping
description: Use when SNVs from gnomAD r2.0.2 or ClinVar VCFs must be projected onto
  UniProt/Swiss-Prot residues and Pfam consensus positions for gene-level or meta-domain
  tolerance analyses.
when_to_use_negative:
- The transcript of interest is not mapped to a Swiss-Prot protein — MetaDome only
  analyzes transcripts linked to a Swiss-Prot entry (others are displayed in gray)
- The variants of interest are indels or non-PASS variants — the pipeline retains
  only SNVs meeting the PASS filter criteria
- You need haplotype context or protein-protein interaction information — aggregating
  variation via meta-domains loses this specific context
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: MetaDome
  role: Provides the mapping database and batch_load/prebuild code that link genomic
    positions to protein residues and Pfam domain consensus positions
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL 10
  role: Relational database storing the complete mapping between genomic positions,
    protein positions, and domain annotations; queried via the 'mappings' table
- name: BLAST
  role: Protein-Protein BLAST v2.2.31+ creates the GENCODE Basic translation to Swiss-Prot
    alignments underlying the mappings
- name: Python 3.5.1
  role: Implementation language for MetaDome, including metadome.batch_load used to
    load the mapping CSV
  repo: https://github.com/cmbi/metadome
provenance:
  source_task_ids:
  - task_005
  - task_003
  - task_006
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:96ff5015311bb8d30f546edb9ff83acc918af10821e3a659be4937eab71c37ca
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
  - Docker
  - MetaDome
  - PostgreSQL
  - PostgreSQL 10
  - Python
  - Python 3.5.1
  - docker-compose
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# genomic-to-protein-position-variant-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Translate SNVs from gnomAD and ClinVar VCFs from genomic coordinates to Swiss-Prot protein residues and Pfam domain consensus positions using MetaDome's PostgreSQL mapping database, enabling variant comparison across homologous protein domains.

## When to use

Use when SNVs from gnomAD r2.0.2 or ClinVar VCFs must be projected onto UniProt/Swiss-Prot residues and Pfam consensus positions for gene-level or meta-domain tolerance analyses.

## When NOT to use

- The transcript of interest is not mapped to a Swiss-Prot protein — MetaDome only analyzes transcripts linked to a Swiss-Prot entry (others are displayed in gray)
- The variants of interest are indels or non-PASS variants — the pipeline retains only SNVs meeting the PASS filter criteria
- You need haplotype context or protein-protein interaction information — aggregating variation via meta-domains loses this specific context

## Inputs

- gnomAD r2.0.2 sites VCF (synonymous, nonsense, missense SNVs, PASS filter; 12,164,292 variants expected)
- ClinVar release VCF (pathogenic/disease-causing SNVs; 34,076 variants expected)
- MetaDome prebuild mapping database CSV (e.g. MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz)
- Pfam-A domain annotations (interpro_domains table from InterProScan, e.g. v5.20-59.0)
- Pfam HMM/Stockholm files for meta-domain consensus position mapping

## Outputs

- Per-variant mappings from genomic position to UniProt residue and Pfam consensus position
- Populated PostgreSQL mapping database ('mappings' and 'interpro_domains' tables)
- Tabulated counts of pathogenic missense variants inside vs outside Pfam domains (~71-72% coverage)
- Case-study count tables, e.g. cdk13_enst00000181839_metadomain_analysis.csv

## How to apply

Obtain a prebuilt mapping database CSV from Zenodo record 19376150 (MetaDome_v2.0 for GRCh37 or GRCh38) or build one via the metadome_prebuild pipelines, then load it with populate_database.sh, which populates a PostgreSQL v10 database through metadome.batch_load. Load the gnomAD r2.0.2 VCF (retaining synonymous, nonsense, and missense SNVs meeting the PASS filter; 12,164,292 variants expected) and the ClinVar pathogenic SNVs (34,076 expected). Query the PostgreSQL 'mappings' table to translate each SNV's genomic position into the corresponding UniProt protein residue, and intersect mapped residues with Pfam-A domain annotations (from InterProScan) to assign Pfam domain and consensus positions. The mapping database links genomic, protein, and domain positions because alignments were created with Protein-Protein BLAST v2.2.31+ between GENCODE Basic translations and Swiss-Prot sequences.

## Related tools

- **MetaDome** (Provides the mapping database and batch_load/prebuild code that link genomic positions to protein residues and Pfam domain consensus positions) — https://github.com/cmbi/metadome
- **PostgreSQL 10** (Relational database storing the complete mapping between genomic positions, protein positions, and domain annotations; queried via the 'mappings' table)
- **BLAST** (Protein-Protein BLAST v2.2.31+ creates the GENCODE Basic translation to Swiss-Prot alignments underlying the mappings)
- **Python 3.5.1** (Implementation language for MetaDome, including metadome.batch_load used to load the mapping CSV) — https://github.com/cmbi/metadome

## Examples

```
./populate_database.sh --data-dir /path/to/csv/dir --file MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz
```

## Evaluation signals

- The gnomAD PASS-filtered variant count equals 12,164,292 and the ClinVar pathogenic count equals 34,076
- The recomputed mapping database reproduces 32,595,355 unique genomic positions linked to 19,226,961 Swiss-Prot residues
- Pfam-domain coverage of pathogenic missense variants is consistent with the reported ~71-72%
- CDK13 ENST00000181839.4 case study counts match: 6 ClinVar SNVs, 64 homologous pathogenic positions, 330 genomic regions, 65 gnomAD missense variants, mean AF 1.24E-05
- The tolerance track reproduces codon for codon against the released prebuild-visualizations quality-control report for the corresponding genome build

## Limitations

- Only transcripts mapped to a Swiss-Prot protein can be used in the analysis; in the reported build, 42,116 of 56,319 GENCODE Basic transcripts were linked to Swiss-Prot sequences
- The mapping load is heavy: the mapping table holds tens of millions of rows per genome build, requiring disk headroom and a long load time
- Aggregating variation via meta-domains loses specific context such as haplotype information or protein-protein interactions, and only encapsulates general biological or molecular functions

## Evidence

- [results] Mapping scale: [evidence span withheld — anchor status: not_found]
- [methods] GENCODE to Swiss-Prot alignment: [evidence span withheld — anchor status: not_found]
