---
name: panelapp-gene-panel-bed-encoding
description: Use when a germline VCF must be filtered to variants in a user-selected
  cancer predisposition gene panel, or when a PanelApp panel needs to be encoded as
  a BED for CPSR --custom_panel-style interval restriction.
when_to_use_negative:
- The input is a somatic/tumor VCF or multi-sample VCF — CPSR-style panel restriction
  here applies to single-sample germline variant calls
- The goal is variant classification (ACMG/AMP) or annotation rather than restricting
  the variant set to a gene panel
- The VCF has already been subset to the target genes, so an interval-overlap step
  would be redundant
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: CPSR
  role: Workflow whose first step restricts the input germline VCF to the chosen virtual
    gene panel via its --custom_panel mechanism
  repo: https://github.com/sigven/cpsr
- name: Python
  role: Implementation language for preprocessing (bgzip/tabix handling, PASS filtering,
    multi-allelic decomposition, BED interval intersection)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:70d794dc3b49bdb61410744a88c9d7a13582083aee3f27ea19f62ba1bcdb536e
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# panelapp-gene-panel-bed-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes a chosen Genomics England PanelApp cancer predisposition gene panel as a four-column BED file (chromosome, start, stop, genesymbol) and uses it to restrict a single-sample germline VCF to variants within panel genes, as in the CPSR workflow's first step. Optionally limits intervals to diagnostic-grade (GREEN) PanelApp genes.

## When to use

Use when a germline VCF must be filtered to variants in a user-selected cancer predisposition gene panel, or when a PanelApp panel needs to be encoded as a BED for CPSR --custom_panel-style interval restriction.

## When NOT to use

- The input is a somatic/tumor VCF or multi-sample VCF — CPSR-style panel restriction here applies to single-sample germline variant calls
- The goal is variant classification (ACMG/AMP) or annotation rather than restricting the variant set to a gene panel
- The VCF has already been subset to the target genes, so an interval-overlap step would be redundant

## Inputs

- Single-sample germline VCF of SNVs/InDels (bgzip-compressed and tabix-indexed, FILTER=PASS records only)
- Genomics England PanelApp panel gene list (optionally with diagnostic-grade confidence levels), encoded as a four-column BED file (chromosome, start, stop, genesymbol)

## Outputs

- Bgzipped VCF restricted to variants overlapping panel gene intervals
- Tabix index (.tbi) for the filtered VCF
- Optionally, a panel BED restricted to diagnostic-grade (GREEN) PanelApp genes

## How to apply

First compress and index the single-sample germline VCF with bgzip/tabix, retain only records with FILTER=PASS, and decompose multi-allelic sites (CPSR-style preprocessing). Load the PanelApp panel and write it as a BED file with exactly four columns (chromosome, start, stop, genesymbol) covering both coding and noncoding regions of the genes; if a confidence-level annotation is available, optionally restrict intervals to diagnostic-grade (GREEN) genes. Intersect the PASS variants against the panel BED intervals with an interval-overlap operation (bedtools intersect, or the CPSR --custom_panel mechanism) so only variants overlapping panel genes are retained. Write the retained variants to a bgzipped VCF and regenerate its tabix (.tbi) index so downstream CPSR annotation/classification operates only on the selected panel.

## Related tools

- **CPSR** (Workflow whose first step restricts the input germline VCF to the chosen virtual gene panel via its --custom_panel mechanism) — https://github.com/sigven/cpsr
- **Python** (Implementation language for preprocessing (bgzip/tabix handling, PASS filtering, multi-allelic decomposition, BED interval intersection))

## Examples

```
bedtools intersect -a sample.pass.vcf.gz -b panelapp_panel.bed > sample.panel.vcf
```

## Evaluation signals

- Every record in the filtered output VCF has FILTER=PASS
- Each retained variant's coordinates overlap one of the panel BED gene intervals — no variants outside the selected panel remain
- Multi-allelic sites from the input have been decomposed to bi-allelic records
- The output VCF is bgzip-compressed and a corresponding tabix (.tbi) index exists
- The panel BED parses with exactly four columns (chromosome, start, stop, genesymbol) per interval

## Limitations

- The custom panel BED must have exactly four columns (chromosome, start, stop, genesymbol); other formats are rejected
- Restriction to diagnostic-grade (GREEN) genes requires that a confidence-level annotation is available on the panel intervals
- Filtering is by interval overlap over coding and noncoding regions of panel genes; it does not itself assess variant consequence or pathogenicity

## Evidence
