---
name: tsv-variant-results-export
description: Use when a germline variant classification workflow needs flat, machine-readable
  per-variant tables (e.g. for downstream scripts or LIMS import) after CPSR ACMG/AMP
  classification.
when_to_use_negative:
- Input is somatic tumor variant data — CPSR is designed for germline variant calls
  from a single sample; use PCGR instead
- Input VCF records are not designated 'PASS' in the FILTER column or contain unresolved
  multi-allelic sites — these are decomposed/filtered before reporting
- A human-readable interactive report alone is required and no downstream machine
  parsing of variant classifications is planned
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: CPSR
  role: Workflow that classifies germline variants and generates the TSV, VCF, and
    HTML outputs
  repo: https://github.com/sigven/cpsr
- name: R
  role: cpsr package performs report generation and visualisation via Quarto, producing
    the TSV/HTML outputs
  repo: https://github.com/sigven/cpsr
- name: pcgrr
  role: R package CPSR builds on for report generation and visualisation
  repo: https://github.com/sigven/pcgr/tree/master/pcgrr
- name: Python
  role: CPSR implementation language handling variant processing and output emission
  repo: https://github.com/sigven/cpsr
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:2db4aa1e144ca3f005e41386216f95b447a488b8f5f8bc00aadd4b9ffd9f78cb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/sigven/cpsr
  repo_url: https://github.com/sigven/cpsr
  tools:
  - CPSR
  - Python
  - R
  - pcgrr
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# tsv-variant-results-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Export CPSR-classified germline variants as TSV tables with per-variant classification fields, including the '|'-separated ACMG_CODE column that lists which ACMG/AMP criteria were met, alongside the annotated VCF and interactive HTML report.

## When to use

Use when a germline variant classification workflow needs flat, machine-readable per-variant tables (e.g. for downstream scripts or LIMS import) after CPSR ACMG/AMP classification.

## When NOT to use

- Input is somatic tumor variant data — CPSR is designed for germline variant calls from a single sample; use PCGR instead
- Input VCF records are not designated 'PASS' in the FILTER column or contain unresolved multi-allelic sites — these are decomposed/filtered before reporting
- A human-readable interactive report alone is required and no downstream machine parsing of variant classifications is planned

## Inputs

- Classified germline variant results (SNVs/InDels) with per-variant ACMG/AMP classifications from a CPSR run
- Run configuration (sample ID, genome assembly GRCh37/GRCh38, selected virtual gene panel)
- Annotated VCF with CPSR INFO annotation tags

## Outputs

- TSV output files with per-variant classification fields
- ACMG_CODE column with '|'-separated list of met ACMG criteria per variant
- Companion annotated VCF: <sample_id>.cpsr.<genome_assembly>.vcf.gz
- Interactive HTML report: <sample_id>.cpsr.<genome_assembly>.html (Settings, Summary of Findings, Variant Classification, Documentation sections)

## How to apply

After CPSR has classified germline variants against ACMG/AMP criteria using public resources (ClinVar, gnomAD, cancerhotspots.org), aggregate the per-variant classification fields into TSV output files that accompany the interactive HTML report. Each TSV row carries variant-level annotation and classification columns; the ACMG_CODE column encodes the exact criteria met as a '|'-separated string (e.g. individual refined ACMG criteria such as PVS1 from the >30-criteria implementation), which lets consumers reconstruct why each variant was called Pathogenic through Benign. Emit these TSVs alongside the companion annotated VCF (<sample_id>.cpsr.<genome_assembly>.vcf.gz carrying CPSR INFO tags) and the HTML report named <sample_id>.cpsr.<genome_assembly>.html, all rendered from the same run configuration (sample ID, genome assembly, virtual gene panel). Validation hinges on the TSV containing the ACMG_CODE field and matching the variants present in the annotated VCF.

## Related tools

- **CPSR** (Workflow that classifies germline variants and generates the TSV, VCF, and HTML outputs) — https://github.com/sigven/cpsr
- **R** (cpsr package performs report generation and visualisation via Quarto, producing the TSV/HTML outputs) — https://github.com/sigven/cpsr
- **pcgrr** (R package CPSR builds on for report generation and visualisation) — https://github.com/sigven/pcgr/tree/master/pcgrr
- **Python** (CPSR implementation language handling variant processing and output emission) — https://github.com/sigven/cpsr

## Evaluation signals

- TSV files exist alongside <sample_id>.cpsr.<genome_assembly>.html and <sample_id>.cpsr.<genome_assembly>.vcf.gz for the run
- Every TSV includes the ACMG_CODE field, with '|'-separated ACMG criteria codes consistent with the variant's overall classification (Pathogenic to Benign)
- Variant count and genomic coordinates in the TSV match the PASS records in the annotated VCF.gz
- Only variants designated 'PASS' in the VCF FILTER column appear in the TSV, and multi-allelic sites have been decomposed before export
- HTML report contains the four required sections (Settings, Summary of Findings, Variant Classification, Documentation) consistent with the TSV content

## Limitations

- Variants must pass the PASS FILTER designation; non-PASS records are excluded from reporting
- Multi-allelic input sites require decomposition before classification/export
- Some classification behavior depends on configurable settings (gnomAD MAF threshold, ClinVar trust level, noncoding filter), so TSV content varies with run configuration

## Evidence

- [methods] TSV output with per-variant classification fields: [evidence span withheld — anchor status: not_found]
