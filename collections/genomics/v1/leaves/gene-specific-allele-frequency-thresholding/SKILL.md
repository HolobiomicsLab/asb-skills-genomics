---
name: gene-specific-allele-frequency-thresholding
description: Use when classifying germline SNVs/InDels against ACMG/AMP criteria (BA1/BS1/PM2/PM1/PM4)
  and gene-specific gnomAD allele-frequency thresholds, not a single global MAF cutoff,
  are required.
when_to_use_negative:
- Input is somatic tumor sequencing data — use PCGR's oncogenicity classification
  (VICC/CGC/ClinGen) instead of germline ACMG classification
- Variants lack population frequency annotation (no gnomAD coverage) — absence/rarity
  criteria cannot be assessed and frequencies cannot drive BA1/BS1/PM2
- The analysis requires a single uniform MAF cutoff across all genes — this skill
  deliberately applies per-gene thresholds via the groupmax subpopulation approach
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: CPSR
  role: Workflow that implements the automated ACMG/AMP classification with gene-specific
    gnomAD allele-frequency thresholds (BA1/BS1/PM2/PM1/PM4) and the groupmax subpopulation
    approach
  repo: https://github.com/sigven/cpsr
- name: pcgrr
  role: R package on which CPSR builds to perform the germline variant classification
    logic
  repo: https://github.com/sigven/pcgr/tree/master/pcgrr
- name: R
  role: Implementation language for the cpsr/pcgrr classification and report generation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:fd057d0749c78540c4a9c96f2b56feb4a0fbf8af0fedcd65a8540bc45e0eae01
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Gene-specific allele frequency thresholding for ACMG germline variant classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies gene-specific gnomAD population allele-frequency cutoffs when assigning the ACMG/AMP rarity and frequency criteria (BA1, BS1, PM2, PM1, PM4) in CPSR's automated germline classification, using the groupmax approach across gnomAD subpopulations.

## When to use

Use when classifying germline SNVs/InDels against ACMG/AMP criteria (BA1/BS1/PM2/PM1/PM4) and gene-specific gnomAD allele-frequency thresholds, not a single global MAF cutoff, are required.

## When NOT to use

- Input is somatic tumor sequencing data — use PCGR's oncogenicity classification (VICC/CGC/ClinGen) instead of germline ACMG classification
- Variants lack population frequency annotation (no gnomAD coverage) — absence/rarity criteria cannot be assessed and frequencies cannot drive BA1/BS1/PM2
- The analysis requires a single uniform MAF cutoff across all genes — this skill deliberately applies per-gene thresholds via the groupmax subpopulation approach

## Inputs

- Germline variant calls (SNVs/InDels) from a single sample, encoded in VCF format (PASS-filtered, multi-allelic sites decomposed)
- CSQ/VEP consequence-annotated variant records (VCF or TSV) for variants in the selected cancer predisposition gene panel
- gnomAD allele-frequency annotations (r2.1.1 non-cancer and r4.1, per subpopulation)
- Gene panel configuration specifying the cancer predisposition genes under analysis

## Outputs

- Per-variant ACMG criterion assignments including BA1/BS1/PM2/PM1/PM4 with gene-specific frequency-threshold rationale
- Per-variant point-based pathogenicity score mapped to the five-level ACMG classification (recorded with score, classification, and '|'-separated ACMG_CODE)

## How to apply

Annotate the input CSQ-tagged VCF or TSV with gnomAD population frequencies (r2.1.1 non-cancer and r4.1 in CPSR) and, for each variant in the selected cancer predisposition gene panel, compare the observed allele frequency against thresholds that are set per gene rather than globally. Use the groupmax approach, taking the maximum allele frequency across gnomAD subpopulations, as the value tested against the BA1/BS1/PM2 thresholds, so that no single subpopulation's frequency is missed. Assign BA1 or BS1 when frequencies exceed the gene-specific stand-alone or strong-evidence cutoffs, and apply PM2 (absence/rarity) as a supporting criterion that carries no weight in the final point-based score. Combine these frequency-based calls with consequence-aware criteria such as PM1 (hotspot/domain) and PM4 (in-frame length change) evaluated via public resources (ClinVar, gnomAD, cancerhotspots.org), keeping the threshold configuration coherent with ClinGen SVI specifications.

## Related tools

- **CPSR** (Workflow that implements the automated ACMG/AMP classification with gene-specific gnomAD allele-frequency thresholds (BA1/BS1/PM2/PM1/PM4) and the groupmax subpopulation approach) — https://github.com/sigven/cpsr
- **pcgrr** (R package on which CPSR builds to perform the germline variant classification logic) — https://github.com/sigven/pcgr/tree/master/pcgrr
- **R** (Implementation language for the cpsr/pcgrr classification and report generation)

## Evaluation signals

- Each variant's ACMG_CODE field records the met criteria '|'-separated, and BA1/BS1/PM2 calls trace back to a gene-specific gnomAD threshold rather than a global one
- Variants flagged at the VUS/Likely Pathogenic border are explicitly flagged, indicating the calibrated score thresholds were applied
- PM2 is recorded as a supporting criterion carrying no weight in the summed pathogenicity score
- LOF variants evaluated alongside frequency criteria respect PVS1 exceptions (GC>GT splice donor, last 5% of CDS truncations) via the LOF_FILTER
- Resulting classifications are comparable in sensitivity/specificity to other algorithms for pathogenic variant detection

## Limitations

- Thresholds depend on gnomAD coverage; variants absent from gnomAD or in poorly covered genes may receive misleading PM2 support
- Automated classification cannot capture all gene-specific clinical context; variants at the VUS/Likely Pathogenic border need manual review
- Only PASS-designated VCF records are used for reporting; variants failing FILTER are excluded from classification

## Evidence
