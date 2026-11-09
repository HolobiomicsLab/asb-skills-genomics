---
name: loss-of-function-variant-filtering
description: Use when annotating germline SNVs/InDels from a VCF and you need to flag
  stop_gained/frameshift or canonical splice donor/acceptor variants as LOF, or need
  to resolve which LOF calls should be downgraded by filter exceptions.
when_to_use_negative:
- Input is a somatic (tumor) VCF — this workflow is designed for germline cancer-predisposition
  screening; use PCGR instead
- Input VCF still contains non-PASS records, multi-allelic sites, or mitochondrial
  (chrM) variants — preprocess first or LOF calls will not be reliable
- You need raw consequence annotation only, without LOF filtering or ACMG PVS1 evaluation
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CPSR
  role: workflow that applies LOF_FILTER tagging and PVS1 decision-tree evaluation
    of LOF variants during germline annotation
  repo: https://github.com/sigven/cpsr
- name: VEP
  role: Variant Effect Predictor run against the GENCODE cache with --flag_pick_allele
    and the LOFTEE plugin to produce LOF consequence calls
- name: vcfanno
  role: attaches supporting annotation datasets (ClinVar, gnomAD, dbNSFP, CIViC) alongside
    the LOF tags in the output VCF
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:b73cdefb0c0ae87c0ebc13226f49afcd87bdb0c1c56739fc79e69549f4f6ea64
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
  - CIViC
  - CPSR
  - Docker
  - NHGRI-EBI GWAS Catalog
  - PCGR
  - VEP
  - bgzip/tabix
  - dbNSFP
  - gnomAD
  - vcfanno
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# loss-of-function-variant-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates and filters putative loss-of-function (LOF) germline variants using VEP's LOFTEE plugin and the CPSR LOF_FILTER tag, applying exceptions (GC>GT at splice donor sites, truncations within the last 5% of coding sequence) and PVS1 decision-tree logic before ACMG classification.

## When to use

Use when annotating germline SNVs/InDels from a VCF and you need to flag stop_gained/frameshift or canonical splice donor/acceptor variants as LOF, or need to resolve which LOF calls should be downgraded by filter exceptions.

## When NOT to use

- Input is a somatic (tumor) VCF — this workflow is designed for germline cancer-predisposition screening; use PCGR instead
- Input VCF still contains non-PASS records, multi-allelic sites, or mitochondrial (chrM) variants — preprocess first or LOF calls will not be reliable
- You need raw consequence annotation only, without LOF filtering or ACMG PVS1 evaluation

## Inputs

- Germline variant calls (SNVs/InDels) for a single sample in VCF format, PASS-filtered and multi-allelic-decomposed
- GENCODE v49 (v19 for GRCh37) VEP cache with LOFTEE plugin data

## Outputs

- Annotated bgzipped/tabix-indexed VCF with INFO fields CSQ, LOF_FILTER, EXONIC_STATUS, CODING_STATUS
- Per-variant putative-LOF flags with filter-exception annotations feeding the PVS1/ACMG classification

## How to apply

Run Variant Effect Predictor (VEP v115, GENCODE v49 cache; v19 for GRCh37) with --flag_pick_allele so each variant carries exactly one main consequence block, and enable the LOFTEE plugin to call loss-of-function predictions. Retain stop_gained/frameshift variants and canonical splice donor/acceptor variants as putative LOF, then apply the LOF_FILTER exceptions: a GC>GT substitution at a splice donor site or a truncation within the last 5% of the coding sequence disqualifies the LOF call. Feed the LOF/LOF_FILTER status into the PVS1 decision tree as part of CPSR's refined ACMG/AMP handling of LOF variants. Write the result as named CPSR INFO tags (CSQ, LOF_FILTER, EXONIC_STATUS, CODING_STATUS) in a bgzipped, tabix-indexed VCF, and verify the tag content per variant before downstream classification.

## Related tools

- **CPSR** (workflow that applies LOF_FILTER tagging and PVS1 decision-tree evaluation of LOF variants during germline annotation) — https://github.com/sigven/cpsr
- **VEP** (Variant Effect Predictor run against the GENCODE cache with --flag_pick_allele and the LOFTEE plugin to produce LOF consequence calls)
- **vcfanno** (attaches supporting annotation datasets (ClinVar, gnomAD, dbNSFP, CIViC) alongside the LOF tags in the output VCF)

## Evaluation signals

- Output VCF is a valid bgzipped/tabix-indexed file with the required INFO fields (CSQ, LOF_FILTER, EXONIC_STATUS, CODING_STATUS)
- Exactly one picked consequence block per variant (VEP --flag_pick_allele behavior)
- Known exceptions are honored: GC>GT substitutions at splice donor sites and truncations in the last 5% of coding sequence carry a LOF_FILTER tag
- Each putative LOF variant has a consistent PVS1 evaluation traceable to the LOF/LOF_FILTER status
- No mitochondrial (chrM) or non-PASS variants present in the annotated output

## Limitations

- LOF calls are heuristic: splice-region exceptions (GC>GT at donor sites) and C-terminal truncations in the last 5% of coding sequence are deliberately filtered, so a plain LOFTEE call is not sufficient without the CPSR exceptions
- Interpretation depends on VEP/GENCODE cache versions (VEP v115, GENCODE v49, or v19 for GRCh37); results may shift across annotation releases

## Evidence

- [methods] VEP consequence annotations underpin the LOF filtering: [evidence span withheld — anchor status: not_found]
