---
name: pfam-domain-consensus-position-mapping
description: Use when you need to project variants (e.g., ClinVar SNVs or gnomAD missense
  variants) from a gene/transcript like CDK13 ENST00000181839.4 onto Pfam domain consensus
  positions and aggregate them across homologous domain instances of a meta-domain.
when_to_use_negative:
- The transcript of interest is not mapped to a Swiss-Prot protein — MetaDome only
  supports transcripts linked to Swiss-Prot sequences (others are displayed in gray)
- You need variant effect prediction or protein 3D-structure visualization of meta-domain
  information — these are only proposed future directions, not implemented
- You require haplotype context or protein-protein interaction information — aggregation
  via meta-domains loses such specific context
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0160
tools:
- name: MetaDome
  role: Provides the mapping database, meta-domain construction, and batch_load routine
    for loading genomic-to-protein-to-domain mappings into PostgreSQL
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL
  role: Relational database (v10) storing the mapping between genomic positions, protein
    positions, and domain annotations queried for transcript and domain instances
- name: Docker
  role: Containerizes the MetaDome application and its services so the mapping database
    and queries run reproducibly (v17.12.1)
  repo: https://github.com/cmbi/metadome
- name: docker-compose
  role: Orchestrates the containerized MetaDome stack (v1.17.1) used by populate_database.sh
    to load and serve the mapping database
- name: BLAST
  role: Protein-Protein BLAST v2.2.31+ aligns GENCODE Basic translations to Swiss-Prot
    sequences to create the underlying mappings
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:1a59aae5fbed0c9f0194a8e22e9e979f70334153480bba58dd81cac6aafea4a8
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
  - Python
  - docker-compose
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# pfam-domain-consensus-position-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps genomic positions, transcript residues, and Pfam domain consensus positions onto one another via a prebuilt MetaDome mapping database in PostgreSQL, so that variants in different genes sitting on evolutionarily equivalent residues of homologous Pfam domains can be compared as a meta-domain.

## When to use

Use when you need to project variants (e.g., ClinVar SNVs or gnomAD missense variants) from a gene/transcript like CDK13 ENST00000181839.4 onto Pfam domain consensus positions and aggregate them across homologous domain instances of a meta-domain.

## When NOT to use

- The transcript of interest is not mapped to a Swiss-Prot protein — MetaDome only supports transcripts linked to Swiss-Prot sequences (others are displayed in gray)
- You need variant effect prediction or protein 3D-structure visualization of meta-domain information — these are only proposed future directions, not implemented
- You require haplotype context or protein-protein interaction information — aggregation via meta-domains loses such specific context

## Inputs

- Prebuilt MetaDome mapping database CSV (Zenodo record 19376150), per genome build (GRCh37/GRCh38)
- Pfam domain Stockholm alignment for the meta-domain (e.g., PF00069)
- ClinVar VCF (clinvar_YYYYMMDD.vcf.gz + .tbi) for GRCh37/GRCh38
- gnomAD sites VCF (gnomad.exomes.V2XXX.sites.vcf.gz for GRCh37; gnomad.joint.v4.X.sites.exomes.vcf.gz for GRCh38)
- Transcript identifier (e.g., ENST00000181839.4) and target Pfam domain ID (e.g., PF00069)

## Outputs

- Genomic position sets corresponding to a Pfam domain consensus position across homologous domain instances
- Counts of ClinVar pathogenic SNVs within the domain and distinct positions carrying pathogenic variants across homologous domains
- gnomAD missense variant count and mean allele frequency at regions encoding a consensus codon
- Case-study result table (e.g., cdk13_enst00000181839_metadomain_analysis.csv)

## How to apply

Load a prebuilt mapping database CSV (e.g., MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4) into PostgreSQL with populate_database.sh, which runs metadome.batch_load; run it once per genome build since both coexist in one database. Query the mapping tables for the transcript of interest (e.g., ENST00000181839.4) to retrieve its protein positions and InterProScan-annotated domain instances (e.g., Pkinase, PF00069) linked to the mapped Swiss-Prot sequence. Retrieve the domain's Stockholm MSA and use the mappings table to project a domain consensus position (e.g., codon 714) back to all corresponding genomic positions across homologous domain instances. Intersect pathogenic SNVs (ClinVar) and population missense variants (gnomAD, PASS-filtered) with these positions, then write recomputed counts to a CSV; for the CDK13 case this yields 6 ClinVar SNVs, 64 homologous positions, 330 genomic regions, 65 missense variants, and mean AF 1.24E-05.

## Related tools

- **MetaDome** (Provides the mapping database, meta-domain construction, and batch_load routine for loading genomic-to-protein-to-domain mappings into PostgreSQL) — https://github.com/cmbi/metadome
- **PostgreSQL** (Relational database (v10) storing the mapping between genomic positions, protein positions, and domain annotations queried for transcript and domain instances)
- **Docker** (Containerizes the MetaDome application and its services so the mapping database and queries run reproducibly (v17.12.1)) — https://github.com/cmbi/metadome
- **docker-compose** (Orchestrates the containerized MetaDome stack (v1.17.1) used by populate_database.sh to load and serve the mapping database)
- **BLAST** (Protein-Protein BLAST v2.2.31+ aligns GENCODE Basic translations to Swiss-Prot sequences to create the underlying mappings)

## Examples

```
./populate_database.sh --data-dir /path/to/csv-dir --file MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz
```

## Evaluation signals

- Recomputed CDK13 case-study counts must match the published values: 6 ClinVar SNVs, 64 homologous positions, 330 genomic regions, 65 missense variants, mean AF 1.24E-05
- Each queried transcript must resolve to a mapped Swiss-Prot sequence with InterProScan-annotated Pfam domain instances (e.g., PF00069) in the mapping tables
- Number of genomic positions returned for a consensus codon must equal the number of homologous domain instances covering it (e.g., 330 regions for codon 714)
- Only variants meeting the PASS filter criteria are counted in gnomAD-based statistics
- populate_database.sh must load all tens of millions of mapping rows per genome build without loss before queries are run

## Limitations

- Aggregating genetic variation via meta-domains loses specific context such as haplotype information or interactions with other proteins
- Meta-domain aggregation only encapsulates general biological or molecular functions attributed to the domain
- The mapping table holds tens of millions of rows per build, so population is a long load requiring disk headroom for the database plus indexes
- Machine learning variant effect prediction and 3D-structure visualization of meta-domain information are proposed future directions, not implemented

## Evidence
