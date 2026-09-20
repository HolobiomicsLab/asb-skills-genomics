---
name: variant-classification-tsv-reporting
description: Use when germline SNV/InDel VCF calls from a single sample need automated
  ACMG/AMP classification (Pathogenic to Benign) against cancer predisposition gene
  panels, with per-variant criteria codes and scores reported.
when_to_use_negative:
- The query variants are somatic tumor aberrations — use PCGR's oncogenicity/actionability
  classification (VICC/CGC/ClinGen and AMP/ASCO/CAP guidelines) instead
- Variants are not designated 'PASS' in the VCF FILTER column, as only PASS variants
  are used for CPSR reporting
- The goal is interactive HTML reporting of prognosis/diagnosis/drug-response biomarkers
  rather than the ACMG classification scoring step itself
edam_operation: http://edamontology.org/operation_3225
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0622
tools:
- name: CPSR
  role: Computational workflow that performs the automated ACMG/AMP germline variant
    classification and produces the per-variant report
  repo: https://github.com/sigven/cpsr
- name: pcgrr
  role: R package that CPSR builds on to implement the classification logic and report
    generation
  repo: https://github.com/sigven/pcgr/tree/master/pcgrr
- name: R
  role: Implementation language of the cpsr package for report generation and visualisation
    via Quarto
- name: Python
  role: Implementation language of the CPSR workflow
- name: Docker
  role: Container technology through which CPSR is freely distributed and run
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:d1c133b78ecb7c89145041d88d33e141fe787be17c951dbef4ac7ee398fab609
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
  - R
  - pcgrr
  edam_operation: http://edamontology.org/operation_3225
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# variant-classification-tsv-reporting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated ACMG/AMP pathogenicity classification of germline variants using CPSR's Sher Loc-derived point-based scoring of 30+ refined criteria, with per-variant results (score, classification, '|'-separated ACMG_CODE) written to a TSV file.

## When to use

Use when germline SNV/InDel VCF calls from a single sample need automated ACMG/AMP classification (Pathogenic to Benign) against cancer predisposition gene panels, with per-variant criteria codes and scores reported.

## When NOT to use

- The query variants are somatic tumor aberrations — use PCGR's oncogenicity/actionability classification (VICC/CGC/ClinGen and AMP/ASCO/CAP guidelines) instead
- Variants are not designated 'PASS' in the VCF FILTER column, as only PASS variants are used for CPSR reporting
- The goal is interactive HTML reporting of prognosis/diagnosis/drug-response biomarkers rather than the ACMG classification scoring step itself

## Inputs

- Germline variant calls (SNVs/InDels) from a single sample in VCF format (PASS-filtered, multi-allelic sites decomposed)
- Annotated variant records with VEP CSQ consequence annotations (CSQ-tagged VCF or TSV)
- Virtual cancer predisposition gene panel selection (from >40 expert-established panels)

## Outputs

- Per-variant TSV file with pathogenicity score, five-level classification, '|'-separated ACMG_CODE, and assertion rationale
- Five-level variant classification (Pathogenic to Benign) per germline variant

## How to apply

Load the annotated germline variant records (CSQ-tagged VCF or TSV) restricted to the selected virtual cancer predisposition gene panel, keeping only variants designated 'PASS' in the VCF FILTER column. Assign refined ACMG/AMP criteria using public resources (ClinVar, gnomAD, cancerhotspots.org), applying gene-specific allele frequency thresholds for BA1/BS1/PM2/PM1/PM4 with the groupmax approach across gnomAD subpopulations. Evaluate loss-of-function variants with the PVS1 decision tree, applying the LOF_FILTER exceptions (GC>GT at splice donor sites or truncations within the last 5% of the coding sequence), and give PM2 (supporting) no weight in the score. Weight and sum met criteria into a point-based pathogenicity score coherent with ClinGen SVI specifications, map it to the five-level classification via calibrated thresholds, and flag variants at the VUS/Likely Pathogenic border. Record met criteria as a '|'-separated ACMG_CODE per variant and write score, classification, ACMG_CODE, and assertion rationale to a TSV file.

## Related tools

- **CPSR** (Computational workflow that performs the automated ACMG/AMP germline variant classification and produces the per-variant report) — https://github.com/sigven/cpsr
- **pcgrr** (R package that CPSR builds on to implement the classification logic and report generation) — https://github.com/sigven/pcgr/tree/master/pcgrr
- **R** (Implementation language of the cpsr package for report generation and visualisation via Quarto)
- **Python** (Implementation language of the CPSR workflow)
- **Docker** (Container technology through which CPSR is freely distributed and run)

## Examples

```
cpsr --query_vcf sample.germline.vcf --cpsr_config_file cpsr.yaml --genome_assembly grch38 --panel_of_cancers predisposition_cancer --output_dir results --output_id patient1
```

## Evaluation signals

- Every reported variant carries a '|'-separated ACMG_CODE listing the met refined criteria, populated from ClinVar, gnomAD, and cancerhotspots.org evidence
- The point-based pathogenicity score maps coherently to one of the five ACMG classification levels, with VUS/Likely Pathogenic border cases explicitly flagged
- PVS1-decision-tree LOF variants exclude the LOF_FILTER exceptions (GC to GT at splice donor sites, truncations within the last 5% of coding sequence)
- Allele frequency criteria (BA1/BS1/PM2/PM1/PM4) respect gene-specific gnomAD MAF thresholds evaluated via the groupmax approach across subpopulations
- Output TSV only contains variants from the selected virtual gene panel whose VCF FILTER column is 'PASS'

## Limitations

- PM2 (supporting) is given no weight in the point-based score, which may affect borderline classifications
- LOF_FILTER exceptions mean putative loss-of-function variants at GC>GT splice donor sites or within the last 5% of the CDS are not treated as PVS1-qualifying
- Classification depends on configurable settings (gnomAD MAF threshold, ClinVar trust level, noncoding filter), so results vary with configuration choices

## Evidence

- [results] In the Sher Loc-derived procedure used by CPSR's automated classification, each ACMG criterion is weighted with positive or negative point scores: [evidence span withheld — anchor status: not_found]
