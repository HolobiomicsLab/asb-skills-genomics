---
name: tolerance-landscape-visualization
description: Use when you need a per-residue genetic tolerance profile for a protein-coding
  transcript from population variation, or to interpret whether a variant sits in
  a tolerated or intolerant region.
when_to_use_negative:
- The transcript of interest is not mapped to a Swiss-Prot protein — in MetaDome such
  transcripts are displayed in gray and cannot be used in the analysis
- You need positional intolerance aggregated across homologous domains (meta-domain
  analysis) rather than a single-transcript gene-wide dN/dS profile
- Variants are indels or fail the PASS filter — only synonymous, nonsense, and missense
  SNVs meeting PASS criteria are counted
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3673
tools:
- name: MetaDome
  role: Web server and Python package that computes the sliding-window dN/dS tolerance
    landscape from gnomAD/ClinVar variation and the mapping database
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL 10
  role: Stores the mapping database used to translate each variant's genomic position
    into the corresponding protein residue
- name: Python 3.5.1
  role: Implementation language for the tolerance landscape computation (pandas/PyVCF
    loading, codon-table background correction)
  repo: https://github.com/cmbi/metadome
- name: Docker
  role: Containerizes the MetaDome server stack (Flask web layer, PostgreSQL, Celery,
    Redis) that serves the tolerance landscape
  repo: https://www.docker.com/get-docker
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:792a504bd040d4378f8b0b9091518aee58ad1b4e221d899da16eda497fdd5d43
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
  - Python 3.5.1
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# tolerance-landscape-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute and visualize a gene-wide tolerance landscape as a missense/synonymous (dN/dS) ratio over a sliding window of 21 residues, using gnomAD and ClinVar SNVs mapped onto a transcript's Swiss-Prot protein via MetaDome's PostgreSQL mapping database.

## When to use

Use when you need a per-residue genetic tolerance profile for a protein-coding transcript from population variation, or to interpret whether a variant sits in a tolerated or intolerant region.

## When NOT to use

- The transcript of interest is not mapped to a Swiss-Prot protein — in MetaDome such transcripts are displayed in gray and cannot be used in the analysis
- You need positional intolerance aggregated across homologous domains (meta-domain analysis) rather than a single-transcript gene-wide dN/dS profile
- Variants are indels or fail the PASS filter — only synonymous, nonsense, and missense SNVs meeting PASS criteria are counted

## Inputs

- gnomAD sites VCF (e.g. gnomad.exomes.V2XXX.sites.vcf.gz for GRCh37, gnomad.joint.v4.X.sites.exomes.vcf.gz for GRCh38) with .tbi index
- ClinVar release VCF (clinvar_YYYYMMDD.vcf.gz) with .tbi index
- MetaDome prebuild mapping database (CSV.gz linking genomic positions to transcripts, UniProt residues, and Pfam domain consensus positions) loaded into PostgreSQL 10
- Transcript-to-Swiss-Prot protein mappings built from GENCODE Basic translations aligned to Swiss-Prot with Protein-Protein BLAST

## Outputs

- Per-position dN/dS tolerance landscape track (tolerance_landscape.csv, one row per protein position, window edges flagged as noisy)
- Gene-wide tolerance landscape visualization served by the MetaDome web server (optionally prebuilt via metadome.prebuild_all)

## How to apply

Load the gnomAD VCF (r2.0.2 for GRCh37; v4.1 joint exomes for GRCh38) and the ClinVar release VCF, retaining only synonymous, nonsense, and missense SNVs that meet the PASS filter criteria plus pathogenic ClinVar SNVs. Query the PostgreSQL mapping database to translate each variant's genomic position into the corresponding Swiss-Prot protein residue of the transcript. Per residue, count observed missense and synonymous variants and normalize each by the number of possible substitutions from the codon table (background correction). Compute dN/dS as (missense_obs/missense_bg)/(synonymous_obs/synonymous_bg) over a sliding window of size 21 (10 residues before and 10 after each residue of interest) across the entirety of the protein, emitting one row per position and marking window edges as noisy since they are not averaged over a full-length window.

## Related tools

- **MetaDome** (Web server and Python package that computes the sliding-window dN/dS tolerance landscape from gnomAD/ClinVar variation and the mapping database) — https://github.com/cmbi/metadome
- **PostgreSQL 10** (Stores the mapping database used to translate each variant's genomic position into the corresponding protein residue)
- **Python 3.5.1** (Implementation language for the tolerance landscape computation (pandas/PyVCF loading, codon-table background correction)) — https://github.com/cmbi/metadome
- **Docker** (Containerizes the MetaDome server stack (Flask web layer, PostgreSQL, Celery, Redis) that serves the tolerance landscape) — https://www.docker.com/get-docker

## Examples

```
./prebuild_all_after_populate_database.sh --workers 8 --genome-build GRCh38.p14
```

## Evaluation signals

- The computed tolerance track must reproduce codon for codon against the released prebuild-visualizations quality-control report (quality_control_report_GRCh37.p13.md / GRCh38.p14.md) for the corresponding genome build
- Only variants meeting the gnomAD PASS filter contribute counts — re-filtering with a different criterion should change the profile, confirming the filter is active
- Window positions near the start/end of the protein should be flagged as noisy because they are not averaged over a full-length window of 21
- dN/dS values are positive ratios of normalized missense to synonymous counts; positions with zero synonymous observations should be inspected for division artifacts
- Running ./prebuild_all_after_populate_database.sh --report should list pending items and, after a full build, report none remaining

## Limitations

- Window edges (start/end of the protein) are noisy because they are not averaged over a full-length window of size 21
- First on-demand request for a transcript without prebuilding takes minutes; prebuilding is optional but affects latency only
- Only SNVs (synonymous, nonsense, missense) meeting PASS criteria from gnomAD and pathogenic SNVs from ClinVar are counted; other variant classes are ignored

## Evidence
