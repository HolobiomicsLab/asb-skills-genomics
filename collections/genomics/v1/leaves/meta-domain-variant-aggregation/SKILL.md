---
name: meta-domain-variant-aggregation
description: Use when a task asks to reproduce a gene/domain case study (e.g., CDK13
  ENST00000181839.4 with Pkinase PF00069, consensus codon 714), count ClinVar pathogenic
  SNVs inside a domain, or compute gnomAD missense variant counts and mean allele
  frequencies across homologous domain positions.
when_to_use_negative:
- Only transcripts mapped to a Swiss-Prot protein can be used in the analysis — a
  transcript lacking a Swiss-Prot mapping (displayed in gray in the web server) cannot
  support meta-domain aggregation
- Sequences that do not start with a start codon (ATG) or end with a stop codon, or
  whose cDNA translation does not match the GENCODE translation via Biopython's translate
  function, are excluded from the mapping
- 'You need haplotype context or protein-protein interaction information: aggregating
  variation via meta-domains loses such specific context and only encapsulates general
  biological or molecular functions attributed to the domain'
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: MetaDome
  role: Core web server / Python application providing the meta-domain concept, PostgreSQL
    mapping database, and batch_load routine for loading the prebuilt mapping CSV
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL
  role: Relational database (v10) storing the genomic-protein-domain mapping queried
    for transcript positions and domain consensus-position mappings
- name: Docker
  role: Containerizes the MetaDome application stack so it can be run locally for
    database population and prebuilding
  repo: https://github.com/cmbi/metadome
- name: docker-compose
  role: Orchestrates the containerized MetaDome components (web, database, Celery,
    Redis) used by populate_database.sh
  repo: https://github.com/cmbi/metadome
- name: Python
  role: Implementation language of MetaDome (v3.5.1); used for downstream counting/intersection
    scripts producing the case-study CSV
  repo: https://github.com/cmbi/metadome
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:8ea4d89ee8932fd85eea684493e1efe7e116d086a060b0f797d55a8664fbedfb
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
  - Docker
  - MetaDome
  - PostgreSQL
  - Python
  - docker-compose
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# meta-domain-variant-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregate genetic variation (gnomAD population variants and ClinVar pathogenic mutations) across homologous Pfam protein domains into meta-domains, projecting variation onto Pfam domain consensus positions via a PostgreSQL mapping database. This raises per-position variant resolution and supports disease-cohort case studies such as the CDK13 Pkinase (PF00069) meta-domain analysis.

## When to use

Use when a task asks to reproduce a gene/domain case study (e.g., CDK13 ENST00000181839.4 with Pkinase PF00069, consensus codon 714), count ClinVar pathogenic SNVs inside a domain, or compute gnomAD missense variant counts and mean allele frequencies across homologous domain positions.

## When NOT to use

- Only transcripts mapped to a Swiss-Prot protein can be used in the analysis — a transcript lacking a Swiss-Prot mapping (displayed in gray in the web server) cannot support meta-domain aggregation
- Sequences that do not start with a start codon (ATG) or end with a stop codon, or whose cDNA translation does not match the GENCODE translation via Biopython's translate function, are excluded from the mapping
- You need haplotype context or protein-protein interaction information: aggregating variation via meta-domains loses such specific context and only encapsulates general biological or molecular functions attributed to the domain

## Inputs

- Prebuilt mapping database CSV.gz (e.g., MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz from Zenodo record 19376150)
- ClinVar VCF (clinvar_YYYYMMDD.vcf.gz with .tbi, tested version 20251006) for the relevant genome build
- gnomAD sites VCF (gnomad.exomes.V2XXX for GRCh37 or gnomad.joint.v4.X.sites.exomes for GRCh38, with .tbi)
- Pfam domain Stockholm MSA / HMM for the target Pfam family (e.g., PF00069)
- Target transcript identifier (e.g., ENST00000181839.4 for CDK13) and domain consensus position(s)

## Outputs

- Case-study statistics table (e.g., cdk13_enst00000181839_metadomain_analysis.csv) with counts of ClinVar SNVs in the gene/domain, distinct pathogenic positions across homologous domains, number of genomic regions, gnomAD missense variant count, and mean allele frequency
- Per-position list of genomic regions encoding a given domain consensus position across all homologous domain instances
- Meta-domain annotation and tolerance landscape per transcript (available prebuilt or computed on demand)

## How to apply

Download the prebuilt mapping database (e.g., MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz from Zenodo record 19376150) and load it with populate_database.sh into PostgreSQL 10, which runs metadome.batch_load over the CSV. Query the mapping tables for the transcript of interest (e.g., ENST00000181839.4) to obtain protein positions and InterProScan-annotated Pfam domain instances on the mapped Swiss-Prot sequence. Retrieve the domain's Stockholm MSA (e.g., PF00069 meta-domain) and use the mappings table to map a domain consensus position (e.g., 714) back to all corresponding genomic positions across homologous domain instances. Intersect ClinVar pathogenic SNVs with the domain positions to count disease-causing variants inside the domain (6 SNVs for CDK13, at 64 distinct positions across 330 genomic regions), then retrieve gnomAD missense variants (PASS-filtered only) at those regions to compute variant count (65) and mean allele frequency (1.24E-05). Validate every recomputed number against the published case-study values as the acceptance criterion for the output table (cdk13_enst00000181839_metadomain_analysis.csv).

## Related tools

- **MetaDome** (Core web server / Python application providing the meta-domain concept, PostgreSQL mapping database, and batch_load routine for loading the prebuilt mapping CSV) — https://github.com/cmbi/metadome
- **PostgreSQL** (Relational database (v10) storing the genomic-protein-domain mapping queried for transcript positions and domain consensus-position mappings)
- **Docker** (Containerizes the MetaDome application stack so it can be run locally for database population and prebuilding) — https://github.com/cmbi/metadome
- **docker-compose** (Orchestrates the containerized MetaDome components (web, database, Celery, Redis) used by populate_database.sh) — https://github.com/cmbi/metadome
- **Python** (Implementation language of MetaDome (v3.5.1); used for downstream counting/intersection scripts producing the case-study CSV) — https://github.com/cmbi/metadome

## Examples

```
./populate_database.sh --data-dir /path/to/csv_dir --file MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz
```

## Evaluation signals

- Each recomputed count must match the article's CDK13 case-study values: 6 ClinVar SNVs, 64 distinct pathogenic positions, 330 genomic regions, 65 gnomAD missense variants, and mean AF 1.24E-05
- All 6 known disease-causing ClinVar SNVs in CDK13 must fall within the Pkinase (PF00069) protein domain
- Only gnomAD variants meeting the PASS filter are counted in missense variant tallies
- The loaded mapping database scale should be consistent with the release (tens of millions of rows per genome build; the release mappings contain 32,595,355 unique genomic positions linked to 19,226,961 Swiss-Prot residues)
- Consensus position 714 maps back to genomic positions across exactly the homologous PF00069 domain instances recorded in the mappings table

## Limitations

- Aggregating genetic variation via meta-domains loses specific context such as haplotype information or interactions with other proteins
- Aggregation via meta-domains only encapsulates general biological or molecular functions attributed to the domain
- Without prebuilding, the first request for a transcript's tolerance landscape or meta-domain annotation takes minutes
- Results depend on the underlying data snapshot (MetaDome updated data covers 56,319 transcripts, 71,419 protein domains, 12,164,292 gnomAD variants, and 34,076 ClinVar pathogenic mutations), so counts can shift across ClinVar/gnomAD releases

## Evidence

- [results] The resulting mappings contain 32,595,355 unique genomic positions that are linked to 19,226,961 residues in Swiss-Prot protein sequences.: [evidence span withheld — anchor status: not_found]
- [abstract] The aggregation of these variation data across homologous domains into meta-domains allowed us to generate amino acid resolution of genetic tolerance profiles for human protein domains.: [evidence span withheld — anchor status: not_found]
