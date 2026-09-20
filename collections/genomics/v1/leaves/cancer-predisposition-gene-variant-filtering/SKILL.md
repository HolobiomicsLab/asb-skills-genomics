---
name: cancer-predisposition-gene-variant-filtering
description: Use when a single-sample germline VCF (SNVs/InDels) must be filtered,
  annotated, and classified Pathogenic-to-Benign in the context of cancer predisposition
  gene panels, e.g. for clinical germline screening reports.
when_to_use_negative:
- Input is somatic tumor data (SNVs/InDels, CNAs, RNA fusions, RNA-seq) — use PCGR,
  which handles tumor genomes, rather than CPSR
- Variants fall outside the selected cancer predisposition gene panels or are not
  designated PASS in the VCF FILTER column
- The task is per-patient somatic actionability (AMP/ASCO/CAP therapeutic/diagnostic/prognostic
  mapping) rather than germline predisposition classification
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: CPSR
  role: Core workflow that filters, annotates, and classifies germline variants against
    ACMG/AMP criteria for cancer predisposition reporting
  repo: https://github.com/sigven/cpsr
- name: pcgrr
  role: R package underlying CPSR's classification and report-generation logic
  repo: https://github.com/sigven/pcgr/tree/master/pcgrr
- name: R
  role: Implementation language of the cpsr package used for report generation and
    visualisation via Quarto
  repo: https://github.com/sigven/cpsr
- name: Docker
  role: Container technology through which the Python/R CPSR tool is distributed and
    run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:948312e6bfefe5b38f02f91aee73c1180c0c55aa9732f120c980fa45a07e83e4
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cancer-predisposition-gene-variant-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filter and classify germline SNV/InDel variant calls from a single cancer patient sample against curated cancer predisposition gene panels, using the CPSR workflow's ACMG/AMP-based point classification with refined filters (PASS, LOF_FILTER, noncoding filter) and public resources (ClinVar, gnomAD, cancerhotspots.org).

## When to use

Use when a single-sample germline VCF (SNVs/InDels) must be filtered, annotated, and classified Pathogenic-to-Benign in the context of cancer predisposition gene panels, e.g. for clinical germline screening reports.

## When NOT to use

- Input is somatic tumor data (SNVs/InDels, CNAs, RNA fusions, RNA-seq) — use PCGR, which handles tumor genomes, rather than CPSR
- Variants fall outside the selected cancer predisposition gene panels or are not designated PASS in the VCF FILTER column
- The task is per-patient somatic actionability (AMP/ASCO/CAP therapeutic/diagnostic/prognostic mapping) rather than germline predisposition classification

## Inputs

- Single-sample germline variant calls (SNVs/InDels) in VCF format, restricted to PASS FILTER records
- Configuration of variant classification settings (gnomAD MAF threshold, ClinVar trust level, secondary findings, pharmacogenomics findings, GWAS hits, noncoding filter)
- Selected virtual cancer predisposition gene panel (from more than 40 expert-established panels)

## Outputs

- Per-variant ACMG/AMP classification (Pathogenic, Likely pathogenic, VUS, Likely benign, Benign)
- Interactive HTML report with gene/variant annotations and database links
- Optional list of actionable secondary findings as recommended by ACMG

## How to apply

Accept a query VCF with germline variant calls (SNVs/InDels) from a single sample and require variants to be designated 'PASS' in the VCF FILTER column before reporting; decompose multi-allelic sites first. Annotate consequences with Ensembl VEP and score variants with CPSR's implementation of more than 30 refined ACMG/AMP criteria, including PVS1 decision-tree evaluation of loss-of-function variants. Apply the LOF_FILTER exceptions (GC to GT at splice donor sites or truncations within the last 5% of the coding sequence) and configure classification settings such as gnomAD MAF threshold, ClinVar trust level, secondary findings, pharmacogenomics findings, GWAS hits, and the noncoding filter. Cross-reference ClinVar, gnomAD (r2.1.1 non-cancer allele frequencies / r4.1), dbNSFP, and Cancer Hotspots so classification thresholds were calibrated against ClinVar variants with a minimum review status of two stars (55,841 variants across 335 cancer predisposition genes), yielding 92.8% concordance for P/LP, 95.7% for VUS, and 96.3% for B/LB. Render results as an interactive HTML report with optional ACMG secondary findings and chemotherapy adverse-event variants.

## Related tools

- **CPSR** (Core workflow that filters, annotates, and classifies germline variants against ACMG/AMP criteria for cancer predisposition reporting) — https://github.com/sigven/cpsr
- **pcgrr** (R package underlying CPSR's classification and report-generation logic) — https://github.com/sigven/pcgr/tree/master/pcgrr
- **R** (Implementation language of the cpsr package used for report generation and visualisation via Quarto) — https://github.com/sigven/cpsr
- **Docker** (Container technology through which the Python/R CPSR tool is distributed and run)

## Evaluation signals

- Output HTML report lists only variants designated 'PASS' in the VCF FILTER column, after multi-allelic decomposition
- Every classified variant carries an ACMG/AMP five-level assertion (P, LP, VUS, LB, B) derived from the >30 refined criteria, with PVS1 decision-tree outcomes traceable for LOF variants
- Per-class concordance against ClinVar two-star assertions in cancer predisposition genes reproduces 92.8% (P/LP), 95.7% (VUS), 96.3% (B/LB) within tolerance
- LOF_FILTER exceptions behave as specified: GC to GT at splice donor sites and truncations within the last 5% of the coding sequence are treated as exceptions to putative LOF
- Report honors configured classification settings (gnomAD MAF threshold, ClinVar trust level, secondary findings, pharmacogenomics findings, GWAS hits, noncoding filter)

## Limitations

- Non-PASS VCF records are excluded from reporting, so unfiltered or failing variants will be silently absent from output
- Classification depends on bundled public resource versions (ClinVar, gnomAD, dbNSFP, Cancer Hotspots, CIViC, CancerMine, UniProt, Pfam); classification calls shift as these bundles update
- LOF_FILTER exceptions mean not all truncating or splice-donor variants are treated as loss-of-function, which can lower pathogenicity calls near the last 5% of the CDS

## Evidence
