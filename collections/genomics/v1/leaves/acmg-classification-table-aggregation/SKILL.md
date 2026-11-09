---
name: acmg-classification-table-aggregation
description: Use when rendering classified germline SNVs/InDels from a CPSR run into
  the per-sample report tables, or when producing/validating the CPSR HTML, VCF.gz
  and TSV outputs with ACMG_CODE fields.
when_to_use_negative:
- Input is somatic tumor data (SNVs/InDels, CNAs, RNA fusions or RNA-seq expression)
  — the article recommends the companion PCGR workflow for individual tumor genomes,
  not CPSR
- Variants have not yet passed through CPSR's ACMG/AMP classification and annotation
  steps, so no per-variant classifications exist to aggregate
- Input VCF records lack a 'PASS' FILTER designation or contain un-decomposed multi-allelic
  sites, which the reporting step does not accept
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3379
tools:
- name: CPSR
  role: Computational workflow whose final step aggregates classified germline variants
    and renders the structured report
  repo: https://github.com/sigven/cpsr
- name: pcgrr
  role: R package that CPSR builds on for report generation infrastructure
  repo: https://github.com/sigven/pcgr/tree/master/pcgrr
- name: R
  role: Language of the cpsr package used for report generation and visualisation
    via Quarto
- name: Quarto
  role: Templating/rendering engine for the interactive HTML variant report
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:dacaf002abdab73b2314d4b437a9d3b09e625109e1eebf5be771e01b7525ba20
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
  - Quarto
  - R
  - pcgrr
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# ACMG classification table aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Final CPSR workflow step that aggregates per-variant ACMG/AMP germline variant classifications (including the '|'-separated ACMG_CODE criteria) together with run settings into an interactive HTML report and companion annotated VCF, JSON and TSV outputs.

## When to use

Use when rendering classified germline SNVs/InDels from a CPSR run into the per-sample report tables, or when producing/validating the CPSR HTML, VCF.gz and TSV outputs with ACMG_CODE fields.

## When NOT to use

- Input is somatic tumor data (SNVs/InDels, CNAs, RNA fusions or RNA-seq expression) — the article recommends the companion PCGR workflow for individual tumor genomes, not CPSR
- Variants have not yet passed through CPSR's ACMG/AMP classification and annotation steps, so no per-variant classifications exist to aggregate
- Input VCF records lack a 'PASS' FILTER designation or contain un-decomposed multi-allelic sites, which the reporting step does not accept

## Inputs

- Germline variant calls (SNVs/InDels) for a single sample in VCF format, PASS-filtered and multi-allelic-decomposed
- Per-variant ACMG/AMP classification results from the CPSR annotation/classification workflow
- Run configuration (sample ID, genome assembly GRCh37/GRCh38, virtual gene panel selection, gnomAD MAF threshold, ClinVar trust level, secondary findings and pharmacogenomics options)

## Outputs

- Interactive HTML variant report named <sample_id>.cpsr.<genome_assembly>.html with sections: Settings, Summary of Findings, Variant Classification, Documentation
- Companion annotated VCF (<sample_id>.cpsr.<genome_assembly>.vcf.gz) with CPSR INFO annotation tags
- TSV output files with per-variant classification fields, including the '|'-separated ACMG_CODE column
- JSON output of annotated variant findings

## How to apply

Load the classified germline variant results (ACMG/AMP classification per variant, Pathogenic to Benign, derived from more than 30 refined ACMG criteria using ClinVar, gnomAD, dbNSFP and cancerhotspots.org) together with the run configuration (sample ID, genome assembly GRCh37/GRCh38, selected virtual gene panel). Aggregate the variant classifications, key run parameters and documentation content into the Quarto report template sections: Settings, Summary of Findings, Variant Classification and Documentation. Render with the cpsr R package (report generation and visualisation via Quarto, built on the pcgrr R package) to produce the interactive HTML report named <sample_id>.cpsr.<genome_assembly>.html. Emit the companion annotated VCF (<sample_id>.cpsr.<genome_assembly>.vcf.gz) carrying CPSR INFO annotation tags, and TSV files whose per-variant fields include the '|'-separated ACMG_CODE column listing which ACMG criteria were met. Only variants designated 'PASS' in the VCF FILTER column are used for reporting, and multi-allelic sites must be decomposed before classification.

## Related tools

- **CPSR** (Computational workflow whose final step aggregates classified germline variants and renders the structured report) — https://github.com/sigven/cpsr
- **pcgrr** (R package that CPSR builds on for report generation infrastructure) — https://github.com/sigven/pcgr/tree/master/pcgrr
- **R** (Language of the cpsr package used for report generation and visualisation via Quarto)
- **Quarto** (Templating/rendering engine for the interactive HTML variant report)

## Evaluation signals

- HTML report file <sample_id>.cpsr.<genome_assembly>.html exists and contains the four required sections: Settings, Summary of Findings, Variant Classification and Documentation
- Companion files <sample_id>.cpsr.<genome_assembly>.vcf.gz and the TSV outputs exist alongside the HTML report
- TSV files include the ACMG_CODE field with '|'-separated ACMG criteria codes per variant
- Every variant used in the report tables has a 'PASS' value in the VCF FILTER column of the input
- Variant classifications span the ACMG/AMP categories Pathogenic to Benign, each backed by the refined ACMG criteria recorded in ACMG_CODE

## Limitations

- Only variants designated 'PASS' in the VCF FILTER column are used for reporting; skipped processing occurs when no PASS variants are detected in the input VCF
- Multi-allelic sites are decomposed upstream, so the report step assumes single-allelic records
- Report content depends on configurable variant classification settings (gnomAD MAF threshold, ClinVar trust level, secondary findings, pharmacogenomics findings, GWAS hits, noncoding filter), so outputs vary per configuration

## Evidence

- [methods] Report rendered via R/Quarto packages: "**R** (the `cpsr` package) — report generation and visualisation via Quarto"
- [methods] ACMG/AMP classification against public resources: "CPSR classifies germline variants against ACMG/AMP
criteria using public resources (ClinVar, gnomAD, cancerhotspots.org)"
- [methods] Only PASS variants reported: "Variants used for reporting should be designated as 'PASS' in the VCF FILTER column"
