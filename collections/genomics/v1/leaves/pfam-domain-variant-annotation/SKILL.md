---
name: pfam-domain-variant-annotation
description: Use when missense/pathogenic SNVs from VCFs (gnomAD, ClinVar) must be
  annotated with Pfam domain assignments and consensus positions to compute domain
  coverage or build meta-domain tolerance profiles.
when_to_use_negative:
- The variant of interest lies in a transcript not mapped to a Swiss-Prot protein
  — those transcripts are excluded from the analysis (displayed gray in MetaDome).
- You need haplotype context or protein-protein interaction information — meta-domain
  aggregation explicitly loses such specific context.
- Variants are indels or non-SNV events; the pipeline retains only synonymous, nonsense,
  and missense SNVs.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_0622
tools:
- name: MetaDome
  role: Provides the PostgreSQL mapping database, domain model, and prebuild pipelines
    (metadome_prebuild_GRCh37/GRCh38) that link genomic positions to UniProt residues
    and Pfam consensus positions; supplies populate_database.sh and metadome.batch_load
    for loading.
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL
  role: Stores the complete mapping between genomic positions, protein positions,
    and all domain annotations; queried via the 'mappings' and 'interpro_domains'
    tables to annotate each SNV.
- name: BLAST
  role: Protein-Protein BLAST v2.2.31+ aligns each GENCODE Basic protein-coding translation
    to Swiss-Prot sequences to create the genomic-to-protein mappings.
- name: Docker
  role: Containerizes the MetaDome application and its PostgreSQL/Celery/Redis services
    (Docker v17.12.1, docker-compose v1.17.1) so the mapping database can be populated
    and queried reproducibly.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:2017d08ab477f9dcdd985bf9ea7c5c8829663c96aec1204d435a36ef8c3463f0
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# pfam-domain-variant-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Projects SNVs (e.g. from gnomAD and ClinVar VCFs) onto Pfam domain consensus positions via a PostgreSQL mapping database that links genomic positions to UniProt residues and Pfam domains, enabling cross-gene comparison of variants at evolutionarily equivalent residues.

## When to use

Use when missense/pathogenic SNVs from VCFs (gnomAD, ClinVar) must be annotated with Pfam domain assignments and consensus positions to compute domain coverage or build meta-domain tolerance profiles.

## When NOT to use

- The variant of interest lies in a transcript not mapped to a Swiss-Prot protein — those transcripts are excluded from the analysis (displayed gray in MetaDome).
- You need haplotype context or protein-protein interaction information — meta-domain aggregation explicitly loses such specific context.
- Variants are indels or non-SNV events; the pipeline retains only synonymous, nonsense, and missense SNVs.

## Inputs

- gnomAD sites VCF (e.g. gnomad.exomes.V2XXX.sites.vcf.gz with .tbi), PASS-filtered synonymous/nonsense/missense SNVs
- ClinVar VCF (e.g. clinvar_YYYYMMDD.vcf.gz with .tbi) filtered to Pathogenic SNVs
- Prebuilt MetaDome mapping database CSV (e.g. MetaDome_v2.0_GRCh37.p13_GENCODE-v19_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz)
- Pfam-A / InterProScan v5.20-59.0 domain annotations

## Outputs

- Per-variant annotations of Pfam domain and domain consensus position
- Fraction of pathogenic missense variants located inside vs outside a Pfam domain (~71-72% for HGMD/ClinVar)
- Per-domain-family variant count tables for downstream meta-domain construction

## How to apply

First filter the variant VCFs: retain only synonymous, nonsense, and missense SNVs meeting the PASS filter criteria (12,164,292 gnomAD variants expected) and only Pathogenic/disease-causing ClinVar SNVs (34,076 expected). Then query the PostgreSQL mapping database — populated from GENCODE Basic translations aligned to Swiss-Prot canonical/isoform sequences with Protein-Protein BLAST v2.2.31+ — using its 'mappings' table to translate each SNV's genomic position to the corresponding UniProt protein residue. Intersect the mapped residues with Pfam-A domain annotations (interpro_domains table, from InterProScan v5.20-59.0) to assign each variant a Pfam domain and consensus position. Finally compute, per dataset, the fraction of pathogenic missense variants inside versus outside a Pfam domain and tabulate counts per domain family. Only transcripts mapped to a Swiss-Prot protein are usable in the analysis.

## Related tools

- **MetaDome** (Provides the PostgreSQL mapping database, domain model, and prebuild pipelines (metadome_prebuild_GRCh37/GRCh38) that link genomic positions to UniProt residues and Pfam consensus positions; supplies populate_database.sh and metadome.batch_load for loading.) — https://github.com/cmbi/metadome
- **PostgreSQL** (Stores the complete mapping between genomic positions, protein positions, and all domain annotations; queried via the 'mappings' and 'interpro_domains' tables to annotate each SNV.)
- **BLAST** (Protein-Protein BLAST v2.2.31+ aligns each GENCODE Basic protein-coding translation to Swiss-Prot sequences to create the genomic-to-protein mappings.)
- **Docker** (Containerizes the MetaDome application and its PostgreSQL/Celery/Redis services (Docker v17.12.1, docker-compose v1.17.1) so the mapping database can be populated and queried reproducibly.)

## Examples

```
./populate_database.sh --data-dir /path/to/csv-dir --file MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz
```

## Evaluation signals

- gnomAD filtering yields 12,164,292 variants; ClinVar filtering yields 34,076 pathogenic mutations.
- Mapping database totals match 71,419 Pfam domains linked to 30,406 Swiss-Prot sequences (and 32,595,355 unique genomic positions to 19,226,961 Swiss-Prot residues).
- Recomputed Pfam-domain coverage of pathogenic missense variants is consistent with the reported ~71-72%.
- Every annotated variant's consensus position falls within the HMM-derived boundaries of its assigned Pfam domain instance.
- Quality-control reports committed beside the release confirm aggregation statistics recomputed from the released dataset.

## Limitations

- Only transcripts mapped to a Swiss-Prot protein can be used in the analysis; others are excluded.
- Aggregating variation via meta-domains loses specific context such as haplotype information or protein-protein interactions.
- Aggregation via meta-domains only encapsulates general biological or molecular functions attributed to the domain.
- The mapping table holds tens of millions of rows per build, so population requires long load times and substantial disk headroom; METADOME_POSTGRES_DB_DIR must be empty on first start.

## Evidence
