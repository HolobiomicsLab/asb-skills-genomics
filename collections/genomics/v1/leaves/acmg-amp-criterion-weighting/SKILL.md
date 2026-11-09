---
name: acmg-amp-criterion-weighting
description: Use when germline SNV/InDel calls in cancer predisposition genes must
  be automatically classified against ACMG/AMP criteria with a reproducible point-based
  score rather than manual expert curation.
when_to_use_negative:
- The input is somatic tumor data requiring oncogenicity/actionability classification
  (VICC/CGC/ClinGen and AMP/ASCO/CAP guidelines, as in PCGR) rather than germline
  ACMG/AMP pathogenicity classification
- Variants fall outside the selected cancer predisposition gene panel or are non-PASS/noncoding
  records excluded by the noncoding filter
- A manually curated ClinVar assertion already exists and the goal is to report it,
  not to re-derive an automated score
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CPSR
  role: Computational workflow that performs the automated ACMG/AMP classification
    of germline variants
  repo: https://github.com/sigven/cpsr
- name: pcgrr
  role: R package on which CPSR builds to implement the ACMG/AMP classification logic
  repo: https://github.com/sigven/pcgr/tree/master/pcgrr
- name: R
  role: Language implementing the classification (pcgrr/cpsr packages) and downstream
    report generation
- name: PCGR
  role: Somatic counterpart sharing CPSR's framework and infrastructure
  repo: https://github.com/sigven/pcgr
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:b863d140f6f8d3e1f2e1f3f742e28372031bf2f880392c818468ecec77fa0c83
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
  - R
  - pcgrr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# acmg-amp-criterion-weighting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

CPSR's automated germline variant classification assigns refined ACMG/AMP criteria (30+), weights each met criterion with positive/negative point scores reflecting relative strength, and sums them into a single per-variant pathogenicity score mapped to the five-level Pathogenic-to-Benign classification.

## When to use

Use when germline SNV/InDel calls in cancer predisposition genes must be automatically classified against ACMG/AMP criteria with a reproducible point-based score rather than manual expert curation.

## When NOT to use

- The input is somatic tumor data requiring oncogenicity/actionability classification (VICC/CGC/ClinGen and AMP/ASCO/CAP guidelines, as in PCGR) rather than germline ACMG/AMP pathogenicity classification
- Variants fall outside the selected cancer predisposition gene panel or are non-PASS/noncoding records excluded by the noncoding filter
- A manually curated ClinVar assertion already exists and the goal is to report it, not to re-derive an automated score

## Inputs

- Annotated germline variant calls (SNVs/InDels) from a single sample in VCF format (PASS-filtered, multi-allelic sites decomposed, VEP consequence annotations)
- ClinVar clinical significance annotations
- gnomAD r2.1.1/r4.1 non-cancer allele frequencies (including subpopulation frequencies)
- Cancer hotspot annotations (cancerhotspots.org v2/v3)

## Outputs

- Per-variant pathogenicity point score
- Five-level classification (Pathogenic to Benign) per variant
- '|'-separated ACMG_CODE string of met criteria per variant
- Assertion rationale per variant, written to a TSV file

## How to apply

Assign refined ACMG/AMP criteria to each annotated germline variant using public resources (ClinVar, gnomAD, cancerhotspots.org), applying gene-specific allele frequency thresholds for BA1/BS1/PM2/PM1/PM4 with the groupmax approach across gnomAD subpopulations. Evaluate loss-of-function variants with the PVS1 decision tree, excluding GC>GT splice-donor and last-5%-of-CDS truncation exceptions (LOF_FILTER), and give PM2 (supporting) no weight in the score. Weight and sum the met criteria into a point-based pathogenicity score coherent with ClinGen SVI specifications, then map the score to the five-level classification via calibrated thresholds, flagging variants at the VUS/Likely Pathogenic border. Record the met criteria as a '|'-separated ACMG_CODE per variant alongside the score and assertion rationale.

## Related tools

- **CPSR** (Computational workflow that performs the automated ACMG/AMP classification of germline variants) — https://github.com/sigven/cpsr
- **pcgrr** (R package on which CPSR builds to implement the ACMG/AMP classification logic) — https://github.com/sigven/pcgr/tree/master/pcgrr
- **R** (Language implementing the classification (pcgrr/cpsr packages) and downstream report generation)
- **PCGR** (Somatic counterpart sharing CPSR's framework and infrastructure) — https://github.com/sigven/pcgr

## Evaluation signals

- Each variant carries a '|'-separated ACMG_CODE listing all met criteria, which can be cross-checked against the reported point score
- PM2 contributes no weight to the final score even when met (supporting-evidence down-weighting invariant)
- PVS1 calls must exclude GC>GT splice-donor substitutions and truncations within the last 5% of the CDS (LOF_FILTER exceptions respected)
- BA1/BS1/PM2/PM1/PM4 frequency criteria use gene-specific gnomAD MAF thresholds with the groupmax approach across subpopulations
- Final score maps to exactly one of the five classification levels, with VUS/Likely Pathogenic border cases flagged

## Limitations

- Classification quality depends on configured settings (gnomAD MAF threshold, ClinVar trust level, noncoding filter) and on the completeness of the bundled public resources (ClinVar, gnomAD, cancerhotspots.org)
- Fully automated point-based scoring can place variants at the VUS/Likely Pathogenic border, requiring manual review
- LOF evidence is restricted by exception rules (GC>GT splice donor, last-5% CDS truncations), so some premature-truncation variants do not receive full PVS1 weight

## Evidence

- [abstract] A fully automated variant classification procedure based on more than 30 refined American College of Medical Genetics and Genomics (ACMG) criteria represents an integral part of the workflow: "A fully automated variant classification procedure based on more than 30 refined American College of Medical Genetics and Genomics (ACMG) criteria represents an integral part of the workflow"
- [methods] CPSR classifies germline variants against ACMG/AMP criteria using public resources (ClinVar, gnomAD, cancerhotspots.org): "CPSR classifies germline variants against ACMG/AMP criteria using public resources (ClinVar, gnomAD, cancerhotspots.org)"
- [methods] LOF_FILTER | Loss-of-function filter - exceptions to putative LOF variants - GC to GT at splice donor sites or truncations within the last 5% of coding sequence: "`LOF_FILTER` | Loss-of-function filter - exceptions to putative LOF variants - GC to GT at splice donor sites or truncations within the last 5% of coding sequence"
- [methods] refined handling of PVS1 criterion for loss-of-function (LOF) variants according to PVS1 decision tree: "refined handling of PVS1 criterion for loss-of-function (LOF) variants according to
        PVS1 decision tree"
- [readme] Variant classification (Pathogenic to Benign) through a dedicated implementation of ACMG/AMP guidelines: "**Variant classification** (*Pathogenic* to *Benign*) through a dedicated implementation of [ACMG/AMP guidelines]"
