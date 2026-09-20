---
name: genomic-coordinate-overlap-analysis
description: Use when a single-sample germline VCF must be filtered to variants overlapping
  a gene panel BED (coding and noncoding regions), optionally restricted to diagnostic-grade
  (GREEN) PanelApp genes.
when_to_use_negative:
- The input VCF is a somatic tumor callset — CPSR is designed for germline variant
  calls from a single sample, and PCGR is the somatic counterpart
- The goal is gene-level enrichment or interval statistics rather than restricting
  variant records — an overlap counting workflow is not the same as panel-based VCF
  subsetting
- The panel regions are not representable as a four-column BED (chromosome, start,
  stop, genesymbol), e.g. only a gene-symbol list without coordinates
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3395
tools:
- name: CPSR
  role: Provides the --custom_panel mechanism for restricting input VCF variants to
    the chosen virtual gene panel via coordinate overlap
  repo: https://github.com/sigven/cpsr
- name: Python
  role: Implementation language for the panel-restriction/intersection step of the
    workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:156fc6692f85224da1e0e53c260734ad0b8bfd55c9b4330d931f11c0faee6ea2
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3395
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# genomic-coordinate-overlap-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Restricts a germline VCF to variants falling within user-selected genomic intervals (e.g. a cancer predisposition gene panel BED with chromosome, start, stop, genesymbol) using interval-overlap operations, as implemented in CPSR's panel-restriction step.

## When to use

Use when a single-sample germline VCF must be filtered to variants overlapping a gene panel BED (coding and noncoding regions), optionally restricted to diagnostic-grade (GREEN) PanelApp genes.

## When NOT to use

- The input VCF is a somatic tumor callset — CPSR is designed for germline variant calls from a single sample, and PCGR is the somatic counterpart
- The goal is gene-level enrichment or interval statistics rather than restricting variant records — an overlap counting workflow is not the same as panel-based VCF subsetting
- The panel regions are not representable as a four-column BED (chromosome, start, stop, genesymbol), e.g. only a gene-symbol list without coordinates

## Inputs

- Single-sample germline VCF of SNVs/InDels (bgzipped and tabix-indexed, FILTER=PASS records, multi-allelic sites decomposed)
- Custom gene panel BED file with exactly four columns: chromosome, start, stop, genesymbol (Genomics England PanelApp panel)
- Optional PanelApp confidence-level annotation to select diagnostic-grade (GREEN) genes

## Outputs

- Panel-restricted VCF containing only variants whose coordinates overlap panel gene intervals
- Tabix (.tbi) index for the filtered bgzipped VCF

## How to apply

First preprocess the VCF: bgzip-compress, tabix-index, retain only records designated 'PASS' in the VCF FILTER column, and decompose any multi-allelic sites (CPSR-style preprocessing). Load the custom panel BED file, which must contain exactly four columns (chromosome, start, stop, genesymbol) representing the chosen Genomics England PanelApp panel; optionally restrict intervals to diagnostic-grade (GREEN) genes when a confidence-level annotation is available. Intersect the PASS variants against the panel BED intervals using an interval-overlap operation (e.g. bedtools intersect or the CPSR --custom_panel mechanism) so that both coding and noncoding variants overlapping panel genes are retained. Write the retained variants as a bgzipped VCF with a tabix (.tbi) index. The rationale is that CPSR profiles only genes in the selected virtual panel (from more than 40 expert-established panels), so coordinate overlap against the BED is the mechanism that scopes all downstream annotation and ACMG classification.

## Related tools

- **CPSR** (Provides the --custom_panel mechanism for restricting input VCF variants to the chosen virtual gene panel via coordinate overlap) — https://github.com/sigven/cpsr
- **Python** (Implementation language for the panel-restriction/intersection step of the workflow)

## Examples

```
cpsr --input germline.vcf.gz --custom_panel panelapp_panel.bed --genome_assembly grch38 --output_dir . --sample_id tumor --cpsr_report.html
```

## Evaluation signals

- Every record in the output VCF has FILTER=PASS in the VCF FILTER column
- Each retained variant's coordinates overlap one of the panel BED gene intervals, with no variants outside the selected panel
- Multi-allelic input sites are decomposed into single-allelic records before intersection
- Output VCF is validly bgzip-compressed and its tabix (.tbi) index is generated and usable
- When GREEN-only restriction is enabled, every interval in the panel BED corresponds to a diagnostic-grade (GREEN) PanelApp gene

## Limitations

- The custom panel BED must have exactly four columns (chromosome, start, stop, genesymbol); other formats are not accepted
- Restricting to GREEN PanelApp genes is only possible when a confidence-level annotation is available in the panel data
- Only 'PASS' variants are used for reporting — filtered-out VCF records are silently excluded from downstream analysis

## Evidence
