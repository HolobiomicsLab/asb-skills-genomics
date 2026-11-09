---
name: gnomad-allele-frequency-aggregation
description: Use when you need gnomAD allele-frequency statistics for evolutionarily
  equivalent residues of a Pfam domain (a meta-domain consensus position), such as
  computing the number of missense variants and mean allele frequency at consensus
  codon 714 of the Pkinase (PF00069) domain.
when_to_use_negative:
- The variant set is not PASS-filtered gnomAD data or includes non-missense variants
  when the target statistic is the missense variant count and mean AF — non-PASS calls
  must be excluded
- The question concerns haplotype context, protein-protein interactions, or gene-specific
  functional detail rather than domain-level aggregated tolerance — aggregation over
  meta-domains discards that context
- The transcript of interest cannot be mapped to a Swiss-Prot protein sequence, since
  only such transcripts support domain consensus mapping
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0199
tools:
- name: MetaDome
  role: Provides the mapping database, batch_load loader, and meta-domain projection
    of variation onto Pfam consensus positions used for the frequency aggregation
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL 10
  role: Stores the genomic-to-protein/domain mapping database queried to map consensus
    codons back to genomic positions
- name: Docker
  role: Containers the MetaDome application stack used to run the database and loading
    workflow
- name: docker-compose
  role: Orchestrates the containerized stack (web, database, Celery, Redis) brought
    up by populate_database.sh for the CSV load
- name: Python
  role: MetaDome application language (v3.5.1) in which batch loading and per-codon
    AF computation are scripted
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:5584459ee81ec474d97a48f17ad7b4a052f22c7c1c17793253b9796c06582f7b
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
  - PostgreSQL 10
  - Python
  - docker-compose
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gnomad-allele-frequency-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregate PASS-filtered gnomAD missense variants onto Pfam domain consensus positions via the MetaDome genomic-to-protein mapping database, so population allele frequencies can be computed per meta-domain codon (e.g. mean AF across all genomic regions encoding CDK13 Pkinase consensus codon 714).

## When to use

Use when you need gnomAD allele-frequency statistics for evolutionarily equivalent residues of a Pfam domain (a meta-domain consensus position), such as computing the number of missense variants and mean allele frequency at consensus codon 714 of the Pkinase (PF00069) domain.

## When NOT to use

- The variant set is not PASS-filtered gnomAD data or includes non-missense variants when the target statistic is the missense variant count and mean AF — non-PASS calls must be excluded
- The question concerns haplotype context, protein-protein interactions, or gene-specific functional detail rather than domain-level aggregated tolerance — aggregation over meta-domains discards that context
- The transcript of interest cannot be mapped to a Swiss-Prot protein sequence, since only such transcripts support domain consensus mapping

## Inputs

- Prebuilt MetaDome mapping-database CSV (MetaDome_v2.0_GRCh37.p13_GENCODE-v19_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz or the GRCh38.p14 equivalent) from Zenodo record 19376150
- gnomAD sites VCF with tabix index (gnomad.exomes.V2XXX.sites.vcf.gz for GRCh37; gnomad.joint.v4.X.sites.exomes.vcf.gz for GRCh38), PASS-filtered
- ClinVar VCF (clinvar_YYYYMMDD.vcf.gz) for pathogenic SNV intersection
- Pfam meta-domain Stockholm alignment for PF00069 (Pkinase)
- Transcript identifier (e.g. ENST00000181839.4) and target Pfam consensus position (e.g. 714)

## Outputs

- Per-consensus-position gnomAD missense variant count and mean allele frequency across all genomic regions encoding that codon
- cdk13_enst00000181839_metadomain_analysis.csv containing recomputed counts (6 ClinVar SNVs, 64 positions, 330 genomic regions, 65 missense variants, mean AF 1.24E-05)

## How to apply

Load the prebuilt mapping database (Zenodo record 19376150) into PostgreSQL v10 using populate_database.sh, which loads the CSV through metadome.batch_load. Query the mapping tables for the transcript of interest (e.g. ENST00000181839.4 / CDK13) to retrieve protein positions and the InterProScan-annotated PF00069 domain instances on the mapped Swiss-Prot sequence. Retrieve the Pfam meta-domain Stockholm alignment and use the 'mappings' table to map a domain consensus position (e.g. 714) back to all corresponding genomic positions across homologous domain instances. Intersect the PASS-filtered, missense-only gnomAD variants (from gnomAD exomes v2 for GRCh37 or gnomAD joint v4.1 exomes for GRCh38) with those genomic regions and compute the variant count and mean allele frequency; for CDK13 codon 714 this yields 330 genomic regions, 65 missense variants, and mean AF 1.24E-05. Validation requires these recomputed counts to match the published case-study values (6 ClinVar SNVs / 64 positions / 330 regions / 65 variants, mean AF 1.24E-05).

## Related tools

- **MetaDome** (Provides the mapping database, batch_load loader, and meta-domain projection of variation onto Pfam consensus positions used for the frequency aggregation) — https://github.com/cmbi/metadome
- **PostgreSQL 10** (Stores the genomic-to-protein/domain mapping database queried to map consensus codons back to genomic positions)
- **Docker** (Containers the MetaDome application stack used to run the database and loading workflow)
- **docker-compose** (Orchestrates the containerized stack (web, database, Celery, Redis) brought up by populate_database.sh for the CSV load)
- **Python** (MetaDome application language (v3.5.1) in which batch loading and per-codon AF computation are scripted)

## Examples

```
./populate_database.sh --data-dir /path/to/csv-dir --file MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz
```

## Evaluation signals

- Recomputed counts for the CDK13 ENST00000181839.4 case study must equal the published values 6 ClinVar SNVs / 64 homologous pathogenic positions / 330 genomic regions / 65 missense variants
- Mean allele frequency at consensus codon 714 must reproduce 1.24E-05
- Only variants meeting the gnomAD PASS filter are counted in the aggregation
- Genomic regions retrieved must all map through the 'mappings' table to PF00069 consensus position 714 instances
- Output CSV cdk13_enst00000181839_metadomain_analysis.csv contains all five recomputed statistics

## Limitations

- Only variants passing gnomAD's PASS filter are counted; filtered-out calls are silently excluded
- MetaDome was tested with specific gnomAD releases (v2 for GRCh37, v4.1 joint exomes for GRCh38) and fixed filenames set in metadome/default_settings.py; other releases require configuration changes
- Aggregating genetic variation via meta-domains loses specific context such as haplotype information or interactions with other proteins
- The mapping table holds tens of millions of rows per build, so the database load is long and needs disk headroom for the database plus its indexes

## Evidence
