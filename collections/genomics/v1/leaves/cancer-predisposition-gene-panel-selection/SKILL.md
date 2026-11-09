---
name: cancer-predisposition-gene-panel-selection
description: Use when a germline VCF must be subset to variants within a chosen cancer
  predisposition gene panel before annotation, classification, or reporting, and the
  panel may need coding and noncoding loci or GREEN-only restriction.
when_to_use_negative:
- The variant calls are somatic tumor variants — CPSR's panel restriction targets
  germline SNVs/InDels from a single cancer patient sample; somatic interpretation
  is PCGR's scope
- The user wants whole-genome/exome reporting without panel restriction — no panel
  BED or virtual panel selection is needed
- The input is already panel-restricted and the only remaining step is ACMG classification
  or report generation
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3395
tools:
- name: CPSR
  role: Workflow whose first step filters input VCF variants against the chosen virtual
    gene panel (--custom_panel / --custom_list mechanisms)
  repo: https://github.com/sigven/cpsr
- name: Python
  role: Parses the germline VCF, applies the PASS filter, loads the panel BED/list,
    and performs interval-overlap restriction
- name: PCGR
  role: Companion somatic-variant workflow sharing CPSR's infrastructure; not used
    for germline panel filtering but defines the shared framework
  repo: https://github.com/sigven/pcgr
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:21bdaab5bdef10b82d624f586d30fd6c8d037d3f9c81a8829a39f98b6281ccb8
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3395
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cancer-predisposition-gene-panel-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Restricts an input single-sample germline VCF (SNVs/InDels) to variants overlapping a user-selected cancer predisposition gene panel — one of CPSR's 40+ virtual expert-established panels or a custom four-column BED — with an option to limit analysis to diagnostic-grade (GREEN) PanelApp genes.

## When to use

Use when a germline VCF must be subset to variants within a chosen cancer predisposition gene panel before annotation, classification, or reporting, and the panel may need coding and noncoding loci or GREEN-only restriction.

## When NOT to use

- The variant calls are somatic tumor variants — CPSR's panel restriction targets germline SNVs/InDels from a single cancer patient sample; somatic interpretation is PCGR's scope
- The user wants whole-genome/exome reporting without panel restriction — no panel BED or virtual panel selection is needed
- The input is already panel-restricted and the only remaining step is ACMG classification or report generation

## Inputs

- Single-sample germline VCF (SNVs/InDels, VCF >= v4.2), bgzipped and tabix-indexed
- Custom panel BED file with four columns: chromosome, start, stop, genesymbol (Genomics England PanelApp panel)
- Optional PanelApp confidence-level annotation (to select diagnostic-grade GREEN genes)
- Optional single-column gene text file (--custom_list) as alternative panel definition

## Outputs

- Bgzip-compressed VCF of PASS variants restricted to panel gene intervals (coding and noncoding), with tabix (.tbi) index
- For CNV extension: panel_cnv_table.tsv with locus (chromosome, start, end) and gene columns

## How to apply

First preprocess the VCF CPSR-style: bgzip and tabix-index the single-sample germline VCF, retain only records with 'PASS' in the VCF FILTER column, and decompose multi-allelic sites. Load the virtual panel as a custom BED with exactly four columns (chromosome, start, stop, genesymbol) representing the chosen Genomics England PanelApp panel; if a confidence-level annotation is available, optionally restrict intervals to diagnostic-grade (GREEN) genes. Intersect the PASS variants against the panel BED intervals using an interval-overlap operation (e.g. bedtools intersect or CPSR's --custom_panel mechanism) so that both coding and noncoding variants overlapping panel genes are retained. Write the retained variants to a bgzipped VCF and regenerate the tabix (.tbi) index.

## Related tools

- **CPSR** (Workflow whose first step filters input VCF variants against the chosen virtual gene panel (--custom_panel / --custom_list mechanisms)) — https://github.com/sigven/cpsr
- **Python** (Parses the germline VCF, applies the PASS filter, loads the panel BED/list, and performs interval-overlap restriction)
- **PCGR** (Companion somatic-variant workflow sharing CPSR's infrastructure; not used for germline panel filtering but defines the shared framework) — https://github.com/sigven/pcgr

## Examples

```
cpsr --input_vcf sample.germline.vcf.gz --custom_panel panelapp_panel.bed --custom_list panel_genes.txt --cpsr_config cpsr_config.toml --output_dir cpsr_run --genome_assembly grch38
```

## Evaluation signals

- Every record in the output VCF has FILTER=PASS in the VCF FILTER column
- Each retained variant's coordinates overlap at least one interval of the panel BED (no variants outside the selected panel genes)
- If GREEN-only restriction was requested, all panel intervals map to diagnostic-grade (GREEN) PanelApp genes
- Output VCF is validly bgzipped and its tabix .tbi index is regenerated and consistent with the filtered records
- For the CNV extension, every emitted row in panel_cnv_table.tsv carries PASS status and a gene symbol present in the selected panel, with the required locus and gene fields

## Limitations

- The custom panel BED must have exactly four columns (chromosome, start, stop, genesymbol); other formats are not accepted via --custom_panel
- Only SNVs/InDels from a single germline sample are accepted as primary input; germline DNA copy number variants (CNVs) were explicitly identified as a sanctioned future extension, not a current capability
- Variants not designated 'PASS' in the VCF FILTER column are excluded, so non-PASS variants of interest will be silently dropped
- Diagnostic-grade (GREEN) restriction requires a confidence-level annotation to be available in the panel source

## Evidence
