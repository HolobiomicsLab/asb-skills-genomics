---
name: germline-variant-report-generation
description: Use when ACMG/AMP-classified germline variants from a single sample (PASS-filtered
  VCF) must be rendered into CPSR's interactive HTML report plus companion VCF/TSV
  outputs.
when_to_use_negative:
- Input is somatic (tumor) variant data — the companion workflow PCGR handles somatic
  SNVs/InDels, CNAs and RNA fusions, whereas this skill targets germline variants
  from a single cancer patient.
- The goal is variant classification or VEP/variant annotation itself rather than
  the final report-rendering step; this skill starts from already-classified variants.
- Input VCF records are not designated PASS in the FILTER column — non-PASS variants
  are excluded from reporting, so filtering must be redone first.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0625
tools:
- name: CPSR
  role: End-to-end workflow whose final step renders the interactive HTML germline
    variant report and companion VCF/JSON/TSV outputs
  repo: https://github.com/sigven/cpsr
- name: pcgrr
  role: R package CPSR builds on for report generation and visualisation in the Quarto
    report templates
  repo: https://github.com/sigven/pcgr/tree/master/pcgrr
- name: Quarto
  role: Rendering engine that produces the interactive HTML variant report from the
    report template
  repo: https://quarto.org
- name: Docker
  role: Container technology through which the Python/R CPSR workflow (including the
    report step) is delivered
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:d5bf926c5455bba689f27f20f98c58efb7adb4e0f3d6fd4a35c2270dc3140f90
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
  - Docker
  - Python
  - Quarto
  - R
  - pcgrr
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# germline-variant-report-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

CPSR's final workflow step renders ACMG/AMP-classified germline SNVs/InDels for a single sample into an interactive HTML report (Settings, Summary of Findings, Variant Classification, Documentation) via the cpsr/pcgrr R packages and Quarto, with companion annotated VCF, JSON and TSV outputs.

## When to use

Use when ACMG/AMP-classified germline variants from a single sample (PASS-filtered VCF) must be rendered into CPSR's interactive HTML report plus companion VCF/TSV outputs.

## When NOT to use

- Input is somatic (tumor) variant data — the companion workflow PCGR handles somatic SNVs/InDels, CNAs and RNA fusions, whereas this skill targets germline variants from a single cancer patient.
- The goal is variant classification or VEP/variant annotation itself rather than the final report-rendering step; this skill starts from already-classified variants.
- Input VCF records are not designated PASS in the FILTER column — non-PASS variants are excluded from reporting, so filtering must be redone first.

## Inputs

- Germline variant calls (SNVs/InDels) from a single sample (i.e. cancer patient) encoded in VCF format, with reportable variants designated PASS in the FILTER column
- Per-variant ACMG/AMP classifications computed by CPSR against refined ACMG criteria using ClinVar, gnomAD and cancerhotspots.org
- Run configuration: sample ID, genome assembly (GRCh37/GRCh38), and selected virtual cancer predisposition gene panel

## Outputs

- Interactive HTML variant report named <sample_id>.cpsr.<genome_assembly>.html organized into four sections: Settings, Summary of Findings, Variant Classification, Documentation
- Companion annotated VCF (<sample_id>.cpsr.<genome_assembly>.vcf.gz) carrying the CPSR INFO annotation tags
- TSV output files with per-variant classification fields, including the '|'-separated ACMG_CODE column listing which ACMG criteria were met
- JSON output of the annotated/classified variant results

## How to apply

Load the classified germline variant results (per-variant ACMG/AMP classification, derived by CPSR from ClinVar, gnomAD and cancerhotspots.org) together with the run configuration: sample ID, genome assembly, and the selected virtual cancer predisposition gene panel from the >40 expert-established panels. Restrict reporting to records designated PASS in the VCF FILTER column, honoring classification settings such as the gnomAD MAF threshold, ClinVar trust level, noncoding filter, and secondary/pharmacogenomic finding flags. Aggregate variant classifications, key run parameters, and documentation content into the Quarto report template sections (Settings, Summary of Findings, Variant Classification, Documentation) and render with the cpsr R package (report generation and visualisation, built on the pcgrr R package) to produce the interactive HTML report named <sample_id>.cpsr.<genome_assembly>.html. Emit the companion annotated VCF (<sample_id>.cpsr.<genome_assembly>.vcf.gz) carrying the CPSR INFO annotation tags, plus TSV and JSON outputs whose per-variant classification fields include the '|'-separated ACMG_CODE column listing which ACMG criteria were met. Validate that the HTML contains all four required sections and that the companion VCF.gz and TSV files exist with the ACMG_CODE field populated.

## Related tools

- **CPSR** (End-to-end workflow whose final step renders the interactive HTML germline variant report and companion VCF/JSON/TSV outputs) — https://github.com/sigven/cpsr
- **pcgrr** (R package CPSR builds on for report generation and visualisation in the Quarto report templates) — https://github.com/sigven/pcgr/tree/master/pcgrr
- **Quarto** (Rendering engine that produces the interactive HTML variant report from the report template) — https://quarto.org
- **Docker** (Container technology through which the Python/R CPSR workflow (including the report step) is delivered)

## Evaluation signals

- The HTML report file <sample_id>.cpsr.<genome_assembly>.html exists and contains the four required sections: Settings, Summary of Findings, Variant Classification, Documentation.
- The companion <sample_id>.cpsr.<genome_assembly>.vcf.gz exists and carries the CPSR INFO annotation tags.
- The TSV output files exist and include the '|'-separated ACMG_CODE field listing which ACMG criteria were met per variant.
- Every variant appearing in the report was designated PASS in the input VCF FILTER column.
- The report's Settings section reflects the actual run configuration (sample ID, genome assembly, selected virtual gene panel).

## Limitations

- Only variants designated PASS in the VCF FILTER column are used for reporting; other records are excluded.
- Report content is driven by configurable classification settings (gnomAD MAF threshold, ClinVar trust level, secondary findings, pharmacogenomics findings, GWAS hits, noncoding filter), so changing these settings changes what the report presents.
- Reporting is scoped to germline SNVs/InDels within the selected virtual gene panel for a single sample; somatic SNV/InDel, CNA, RNA fusion and expression interpretation belongs to the companion PCGR workflow.

## Evidence
