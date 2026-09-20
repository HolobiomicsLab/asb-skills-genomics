---
name: clingen-svi-criteria-specification-mapping
description: Use when germline SNV/InDel calls (VCF/TSV) in cancer predisposition
  genes must be assigned an automated ACMG/AMP pathogenicity class with per-criterion
  evidence codes.
when_to_use_negative:
- The input is somatic tumor data — use PCGR's oncogenicity/actionability classification
  (VICC/CGC/ClinGen, AMP/ASCO/CAP) instead of germline ACMG/AMP classification
- Variants are already annotated with final ClinVar review-status assertions that
  should be reported verbatim rather than reclassified
- The analysis requires manual, case-by-case expert curation with gene-specific evidence
  specification overrides rather than automated scoring
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_2640
tools:
- name: CPSR
  role: Computational workflow that performs the automated ACMG/AMP germline classification
    against ClinVar, gnomAD and cancerhotspots.org
  repo: https://github.com/sigven/cpsr
- name: pcgrr
  role: R package on which CPSR builds, implementing the classification logic and
    report generation
  repo: https://github.com/sigven/pcgr/tree/master/pcgrr
- name: R
  role: Implementation language of the cpsr/pcgrr packages for classification and
    report generation via Quarto
- name: Python
  role: Implementation language of the CPSR workflow alongside R, distributed via
    Docker
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:73eaeb7b16379bf49c3018faffd2144e6bb4993f68884f8016cf9d6d99714f5d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# clingen-svi-criteria-specification-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An automated ACMG/AMP germline variant classification procedure in which each refined ACMG criterion (over 30 in CPSR, derived from Sher Loc) is weighted with positive or negative point scores coherent with ClinGen SVI criteria specifications, and the weighted sum yields a single per-variant pathogenicity score mapped to the five-level classification (Pathogenic to Benign).

## When to use

Use when germline SNV/InDel calls (VCF/TSV) in cancer predisposition genes must be assigned an automated ACMG/AMP pathogenicity class with per-criterion evidence codes.

## When NOT to use

- The input is somatic tumor data — use PCGR's oncogenicity/actionability classification (VICC/CGC/ClinGen, AMP/ASCO/CAP) instead of germline ACMG/AMP classification
- Variants are already annotated with final ClinVar review-status assertions that should be reported verbatim rather than reclassified
- The analysis requires manual, case-by-case expert curation with gene-specific evidence specification overrides rather than automated scoring

## Inputs

- Germline variant calls (SNVs/InDels) from a single sample in VCF format (PASS-filtered, multi-allelic sites decomposed)
- VEP consequence annotations (CSQ-tagged VCF or TSV)
- Public annotation resources: ClinVar, gnomAD (r2.1.1 non-cancer and r4.1), cancerhotspots.org (v2/v3)
- Cancer predisposition virtual gene panel selection (from >40 expert-established panels)

## Outputs

- Per-variant pathogenicity score (point-based) and five-level classification (Pathogenic to Benign)
- Per-variant '|'-separated ACMG_CODE string of met criteria
- Variant assertion rationale flagging variants at the VUS/Likely Pathogenic border
- TSV file of per-variant classification results, plus interactive HTML report

## How to apply

Load CSQ-tagged germline variant records (VCF or TSV) for the selected gene panel and assign refined ACMG/AMP criteria using ClinVar, gnomAD, and cancerhotspots.org, applying gene-specific allele frequency thresholds for BA1/BS1/PM2/PM1/PM4 with the groupmax approach across gnomAD subpopulations. Evaluate loss-of-function variants through the PVS1 decision tree, excluding exceptions such as GC>GT at splice donor sites or truncations within the last 5% of coding sequence (the LOF_FILTER), and give PM2 (supporting) no weight in the score. Weight and sum the met criteria into a point-based pathogenicity score coherent with ClinGen SVI criteria specifications, then map the score to the five-level classification via calibrated thresholds, flagging variants at the VUS/Likely Pathogenic border. Record met criteria as a '|'-separated ACMG_CODE per variant and write score, classification, ACMG_CODE, and assertion rationale to a TSV file.

## Related tools

- **CPSR** (Computational workflow that performs the automated ACMG/AMP germline classification against ClinVar, gnomAD and cancerhotspots.org) — https://github.com/sigven/cpsr
- **pcgrr** (R package on which CPSR builds, implementing the classification logic and report generation) — https://github.com/sigven/pcgr/tree/master/pcgrr
- **R** (Implementation language of the cpsr/pcgrr packages for classification and report generation via Quarto)
- **Python** (Implementation language of the CPSR workflow alongside R, distributed via Docker)

## Evaluation signals

- Every reported variant carries a '|'-separated ACMG_CODE listing exactly the criteria whose point weights contributed to the score
- The weighted sum maps to one of exactly five levels (Pathogenic, Likely Pathogenic, VUS, Likely Benign, Benign) with VUS/Likely Pathogenic border cases flagged
- PVS1-qualifying LOF variants show the GC>GT splice-donor and last-5%-of-CDS exceptions excluded via LOF_FILTER
- PM2 does not add to the point score despite being met (no-weight supporting criterion)
- Scored classifications are consistent with ClinVar assertions and CPSR demonstrates comparable sensitivity/specificity to other algorithms

## Limitations

- PM2 (supporting) is deliberately given no weight in the point score, limiting score contribution of rare-but-unobserved evidence
- PVS1 evaluation excludes GC>GT splice-donor variants and truncations within the last 5% of coding sequence, so some putative LOF calls will not trigger PVS1
- Classification is sensitive to configuration settings (gnomAD MAF threshold, ClinVar trust level, noncoding filter) and to the PASS-filter requirement on input VCF records

## Evidence
