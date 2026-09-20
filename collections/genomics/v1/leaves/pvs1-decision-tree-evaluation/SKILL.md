---
name: pvs1-decision-tree-evaluation
description: Use when classifying germline SNVs/InDels in cancer predisposition genes
  where a variant is predicted to cause loss of function (frameshift, stop-gained,
  canonical splice-site) and the PVS1 criterion must be scored automatically.
when_to_use_negative:
- The variant is already interpreted with a ClinVar assertion at Pathogenic/Likely
  Pathogenic with sufficient trust level, making de novo PVS1 scoring redundant
- The input is somatic tumor data — this skill is for germline cancer-predisposition
  classification; somatic variants should be handled with PCGR's oncogenicity workflow
- The variant is a missense/inframe change with no predicted loss of function, where
  PVS1 does not apply (PP3/PM1-type evidence is used instead)
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: CPSR
  role: Workflow that implements the PVS1 decision tree as part of its automated ACMG/AMP
    germline variant classification against ClinVar, gnomAD and cancerhotspots.org
  repo: https://github.com/sigven/cpsr
- name: pcgrr
  role: R package providing the classification logic on which CPSR builds for ACMG/AMP
    criteria evaluation
  repo: https://github.com/sigven/pcgr/tree/master/pcgrr
- name: VEP
  role: Supplies the consequence annotations (splice-site, frameshift, stop-gained)
    that drive the PVS1 decision tree branches
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:d858be59ce678d74ef9537b9cbd45bd15baacdd6f52714568dbbf390427ce244
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# PVS1 decision-tree evaluation of loss-of-function variants

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An automated evaluation of loss-of-function (LOF) germline variants against the PVS1 (very strong pathogenic) ACMG/AMP criterion, implemented in CPSR as a decision tree that grades the predicted impact of a truncating variant before assigning pathogenicity point scores.

## When to use

Use when classifying germline SNVs/InDels in cancer predisposition genes where a variant is predicted to cause loss of function (frameshift, stop-gained, canonical splice-site) and the PVS1 criterion must be scored automatically.

## When NOT to use

- The variant is already interpreted with a ClinVar assertion at Pathogenic/Likely Pathogenic with sufficient trust level, making de novo PVS1 scoring redundant
- The input is somatic tumor data — this skill is for germline cancer-predisposition classification; somatic variants should be handled with PCGR's oncogenicity workflow
- The variant is a missense/inframe change with no predicted loss of function, where PVS1 does not apply (PP3/PM1-type evidence is used instead)

## Inputs

- Annotated germline variant calls (SNVs/InDels) from a single sample in VCF format, PASS-filtered, with VEP consequence annotations (CSQ tags)
- Gene-level annotations of coding sequence structure (exon positions, CDS extent) from Ensembl/VEP
- Public resources for ACMG classification: ClinVar, gnomAD (r2.1.1/r4.1), cancerhotspots.org

## Outputs

- Per-variant PVS1 evidence status feeding the ACMG point-based pathogenicity score
- Five-level variant classification (Pathogenic to Benign) with assertion rationale
- '|'-separated ACMG_CODE string of met criteria per variant, written to a TSV output file

## How to apply

In CPSR's automated ACMG classification, each LOF variant is run through the PVS1 decision tree, which evaluates whether the variant consequence type (e.g. frameshift, splice-affecting, stop-gained) meets the criteria for PVS1 and at what evidence strength. Variants flagged as putative LOF are first passed through the LOF_FILTER, which removes known exception classes: GC to GT substitutions at splice donor sites and truncations within the last 5% of the coding sequence, since these are not presumed to trigger nonsense-mediated decay. The resulting PVS1 status is combined with other met ACMG criteria (weighted positive/negative points per ClinGen SVI specifications, with PM2 given no supporting weight) and summed into a single per-variant pathogenicity score that is mapped to the five-level classification. Met criteria are recorded per variant in a '|'-separated ACMG_CODE field alongside the score and classification.

## Related tools

- **CPSR** (Workflow that implements the PVS1 decision tree as part of its automated ACMG/AMP germline variant classification against ClinVar, gnomAD and cancerhotspots.org) — https://github.com/sigven/cpsr
- **pcgrr** (R package providing the classification logic on which CPSR builds for ACMG/AMP criteria evaluation) — https://github.com/sigven/pcgr/tree/master/pcgrr
- **VEP** (Supplies the consequence annotations (splice-site, frameshift, stop-gained) that drive the PVS1 decision tree branches)

## Evaluation signals

- Known LOF exception variants (GC>GT at splice donor sites, truncations within the last 5% of coding sequence) are excluded by LOF_FILTER and do not receive PVS1 points
- PM2 (supporting) contributes no weight to the summed pathogenicity score even when allele-frequency criteria are met
- Each variant's ACMG_CODE field lists the met criteria '|'-separated and the point sum maps to one of the five ClinGen-coherent classification levels
- Variants at the VUS/Likely Pathogenic border are explicitly flagged rather than silently upgraded

## Limitations

- GC to GT substitutions at splice donor sites are treated as exceptions to putative LOF and thus escape PVS1
- Truncations within the last 5% of coding sequence are excluded from PVS1 as presumptively non-LOF
- Automated PVS1 scoring is refined but gene-specific LOH (loss-of-function) evidence is not independently curated per variant; allele-frequency thresholds (BA1/BS1/PM2/PM1/PM4) and the groupmax gnomAD subpopulation approach govern the neighboring criteria

## Evidence
