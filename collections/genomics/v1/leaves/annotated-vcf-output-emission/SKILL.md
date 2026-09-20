---
name: annotated-vcf-output-emission
description: Use when a germline variant reporting workflow must emit classified variants
  as a bgzipped annotated VCF companion to its interactive HTML and TSV report outputs.
when_to_use_negative:
- The query data are somatic tumor variants (SNVs/InDels, CNAs, RNA fusions) rather
  than germline calls from a single cancer-patient sample — that is PCGR's remit,
  not CPSR's VCF emission
- Input VCF records are not designated 'PASS' in the FILTER column, or contain multi-allelic
  sites that have not been decomposed — the reporting pipeline excludes the former
  and requires decomposition of the latter first
- Only the interactive HTML report is the deliverable and no downstream consumer needs
  a machine-readable annotated variant track
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3396
tools:
- name: CPSR
  role: Workflow that annotates, classifies and emits the companion bgzipped annotated
    VCF with CPSR INFO tags as part of report generation
  repo: https://github.com/sigven/cpsr
- name: pcgrr
  role: R package CPSR builds on; provides the report-generation and variant-annotation
    infrastructure behind the emitted VCF
  repo: https://github.com/sigven/pcgr
- name: VEP
  role: Source of the complete consequence annotations carried into the emitted VCF
    records
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:35cc73fa265079496430d02d74b86e63b0c9a239833c094bc6c34677d0b8c3ce
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
  - VEP
  - pcgrr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3396
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# annotated-vcf-output-emission

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

CPSR's output-emission step: alongside the interactive HTML variant report, the workflow writes a bgzipped annotated VCF (<sample_id>.cpsr.<genome_assembly>.vcf.gz) carrying CPSR INFO annotation tags, keeping ACMG/AMP-classified germline variants machine-readable together with companion JSON and TSV outputs.

## When to use

Use when a germline variant reporting workflow must emit classified variants as a bgzipped annotated VCF companion to its interactive HTML and TSV report outputs.

## When NOT to use

- The query data are somatic tumor variants (SNVs/InDels, CNAs, RNA fusions) rather than germline calls from a single cancer-patient sample — that is PCGR's remit, not CPSR's VCF emission
- Input VCF records are not designated 'PASS' in the FILTER column, or contain multi-allelic sites that have not been decomposed — the reporting pipeline excludes the former and requires decomposition of the latter first
- Only the interactive HTML report is the deliverable and no downstream consumer needs a machine-readable annotated variant track

## Inputs

- Germline SNV/InDel variant calls from a single sample in VCF format (PASS-filtered, multi-allelic sites decomposed)
- Classified variant results with per-variant ACMG/AMP classifications and CPSR annotation tags (ClinVar, gnomAD, VEP consequences)
- Run configuration: sample ID, genome assembly (GRCh37/GRCh38), virtual gene panel selection

## Outputs

- <sample_id>.cpsr.<genome_assembly>.vcf.gz — bgzipped annotated VCF carrying CPSR INFO annotation tags, emitted as companion to the interactive HTML report and TSV outputs

## How to apply

Run this as part of CPSR's report generation step, after the input germline SNV/InDel VCF from a single sample has been validated, multi-allelic sites decomposed, and variants classified against ACMG/AMP criteria using ClinVar, gnomAD, cancerhotspots.org and complete VEP consequence annotations. Restrict emission to records designated 'PASS' in the VCF FILTER column, since the reporting pipeline excludes anything else. Serialize the classified variants to a bgzipped VCF named <sample_id>.cpsr.<genome_assembly>.vcf.gz, encoding the sample ID and genome assembly (GRCh37/GRCh38) from the run configuration in the filename. Populate each record's INFO column with CPSR annotation tags that mirror the per-variant fields in the companion TSV outputs, including the '|'-separated ACMG_CODE column listing which ACMG criteria were met, so the VCF stays consistent with the interactive HTML report sections (Settings, Summary of Findings, Variant Classification, Documentation). This emission runs inside the cpsr/pcgrr R pipeline alongside Quarto report rendering, not as a standalone post-processing script.

## Related tools

- **CPSR** (Workflow that annotates, classifies and emits the companion bgzipped annotated VCF with CPSR INFO tags as part of report generation) — https://github.com/sigven/cpsr
- **pcgrr** (R package CPSR builds on; provides the report-generation and variant-annotation infrastructure behind the emitted VCF) — https://github.com/sigven/pcgr
- **VEP** (Source of the complete consequence annotations carried into the emitted VCF records)

## Evaluation signals

- <sample_id>.cpsr.<genome_assembly>.vcf.gz exists next to the .html report and decompresses as a valid bgzip-compressed VCF
- INFO tags on each emitted record carry the same per-variant annotation fields found in the companion TSV, including a '|'-separated ACMG_CODE listing met ACMG criteria
- All emitted records show 'PASS' in the FILTER column, matching the reporting requirement
- Former multi-allelic input sites appear as decomposed single-ALT records rather than combined ALT alleles
- The filename's sample_id and genome_assembly components match the values shown in the report's Settings section

## Limitations

- Scope is germline SNVs/InDels from a single sample; other variant classes are outside the CPSR workflow
- Non-PASS variants are excluded from the emitted outputs, so a small VCF may reflect FILTER-based exclusion rather than a genuinely variant-poor sample
- INFO tag names and TSV field layout (e.g., the ACMG_CODE column) should be verified against the specific CPSR release used, as the article documents the behavior without enumerating every tag

## Evidence

- [other] Emit the companion annotated VCF (<sample_id>.cpsr.<genome_assembly>.vcf.gz) carrying the CPSR INFO annotation tags.: [evidence span withheld — anchor status: not_found]
- [other] verify the HTML report file exists and contains the required sections, the companion VCF.gz and TSV files exist, and the TSV includes the ACMG_CODE field.: [evidence span withheld — anchor status: not_found]
