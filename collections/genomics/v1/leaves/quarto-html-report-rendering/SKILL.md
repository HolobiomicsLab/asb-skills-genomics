---
name: quarto-html-report-rendering
description: Use when classified germline variant results (ACMG/AMP calls per variant)
  plus run configuration (sample ID, genome assembly, virtual gene panel) must be
  rendered into an interactive HTML variant report with companion VCF/TSV outputs.
when_to_use_negative:
- Input is a somatic tumor variant callset requiring oncogenicity/actionability interpretation
  — use the companion PCGR tool instead
- The VCF has not yet been annotated and ACMG/AMP-classified (e.g. raw variant calls);
  report rendering expects classified results
- Multi-allelic sites are present and have not been decomposed, or records are not
  designated 'PASS' in the VCF FILTER column
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: cpsr
  role: R package providing report generation and visualisation via Quarto for the
    germline variant report
  repo: https://github.com/sigven/cpsr
- name: pcgrr
  role: R package CPSR builds on, underlying report generation and report templates
  repo: https://github.com/sigven/pcgr/tree/master/pcgrr
- name: PCGR
  role: Somatic counterpart sharing the same framework and infrastructure as CPSR
  repo: https://github.com/sigven/pcgr
- name: Quarto
  role: Rendering engine used by the pcgrr/cpsr-based report templates to produce
    the interactive HTML output
  repo: https://quarto.org
- name: rmarkdown
  role: R markdown rendering infrastructure supporting dynamic report output
  repo: https://pkgs.rstudio.com/rmarkdown/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:b4401a5425507a580ea61a3299f34f601532bba00de62673620dd926616e6c56
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
  - PCGR
  - Python
  - Quarto
  - R
  - cpsr
  - pcgrr
  - rmarkdown
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# quarto-html-report-rendering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Renders CPSR's final classified germline variant results into an interactive HTML report via the Quarto engine (using the cpsr R package, built on pcgrr), emitting <sample_id>.cpsr.<genome_assembly>.html alongside companion annotated VCF.gz, JSON and TSV files.

## When to use

Use when classified germline variant results (ACMG/AMP calls per variant) plus run configuration (sample ID, genome assembly, virtual gene panel) must be rendered into an interactive HTML variant report with companion VCF/TSV outputs.

## When NOT to use

- Input is a somatic tumor variant callset requiring oncogenicity/actionability interpretation — use the companion PCGR tool instead
- The VCF has not yet been annotated and ACMG/AMP-classified (e.g. raw variant calls); report rendering expects classified results
- Multi-allelic sites are present and have not been decomposed, or records are not designated 'PASS' in the VCF FILTER column

## Inputs

- Classified germline variant results (ACMG/AMP classification per variant) for a single sample
- Run configuration: sample ID, genome assembly (GRCh37/GRCh38), selected virtual cancer predisposition gene panel
- Single-sample germline SNV/InDel VCF (variants designated 'PASS' in the FILTER column, multi-allelic sites decomposed)

## Outputs

- Interactive HTML report: <sample_id>.cpsr.<genome_assembly>.html with Settings, Summary of Findings, Variant Classification and Documentation sections
- Annotated VCF with CPSR INFO annotation tags: <sample_id>.cpsr.<genome_assembly>.vcf.gz
- TSV output files with per-variant classification fields, including the '|'-separated ACMG_CODE column
- JSON output file

## How to apply

Load the classified germline variant results — ACMG/AMP classification per variant, derived from more than 30 refined ACMG criteria against public resources (ClinVar, gnomAD, cancerhotspots.org) — together with the run configuration (sample ID, genome assembly, virtual gene panel). Aggregate the classifications, key run parameters, and documentation content into the Quarto report template sections (Settings, Summary of Findings, Variant Classification, Documentation), rendering through the cpsr R package whose report generation and visualisation run via Quarto on top of the pcgrr R package. Report rendering operates on variants that are designated 'PASS' in the VCF FILTER column; multi-allelic input sites are decomposed beforehand. Emit the companion annotated VCF (<sample_id>.cpsr.<genome_assembly>.vcf.gz) carrying CPSR INFO annotation tags, plus TSV output files with per-variant classification fields including the '|'-separated ACMG_CODE column listing which ACMG criteria were met.

## Related tools

- **cpsr** (R package providing report generation and visualisation via Quarto for the germline variant report) — https://github.com/sigven/cpsr
- **pcgrr** (R package CPSR builds on, underlying report generation and report templates) — https://github.com/sigven/pcgr/tree/master/pcgrr
- **PCGR** (Somatic counterpart sharing the same framework and infrastructure as CPSR) — https://github.com/sigven/pcgr
- **Quarto** (Rendering engine used by the pcgrr/cpsr-based report templates to produce the interactive HTML output) — https://quarto.org
- **rmarkdown** (R markdown rendering infrastructure supporting dynamic report output) — https://pkgs.rstudio.com/rmarkdown/

## Evaluation signals

- The HTML report file <sample_id>.cpsr.<genome_assembly>.html exists and contains the four required sections: Settings, Summary of Findings, Variant Classification, and Documentation
- The companion annotated VCF <sample_id>.cpsr.<genome_assembly>.vcf.gz exists and carries CPSR INFO annotation tags
- TSV output files exist and include the ACMG_CODE field with '|'-separated ACMG criteria codes
- All reported variants were designated 'PASS' in the VCF FILTER column of the input
- Report content reflects the run configuration (correct sample ID, genome assembly, and selected virtual gene panel)

## Limitations

- Only variants designated 'PASS' in the VCF FILTER column are used for reporting; non-PASS records are dropped
- Multi-allelic input sites must be decomposed before reporting
- Designed for germline SNVs/InDels from a single sample; somatic, CNA, RNA fusion and RNA-seq interpretation is PCGR's scope, not CPSR's

## Evidence

- [readme] Quarto-based report engine in PCGR framework: [evidence span withheld — anchor status: external_doc]
