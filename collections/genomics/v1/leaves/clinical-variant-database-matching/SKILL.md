---
name: clinical-variant-database-matching
description: Use when a germline SNV/InDel VCF (PASS-filtered, multi-allelic sites
  decomposed, chrM removed) must be enriched with clinical significance, biomarker,
  population allele frequency and in-silico deleteriousness annotations before ACMG/AMP
  classification.
when_to_use_negative:
- Input is somatic tumor variant data — CPSR targets germline variant interpretation,
  and the accompanying PCGR tool is the intended choice for somatic SNVs/InDels, CNAs
  and RNA fusions.
- Variants are not in VCF format or are multi-allelic without prior decomposition,
  since the annotation step expects decomposed PASS records with one picked consequence
  block each.
- The goal is de novo variant discovery or calling rather than annotation of existing
  germline calls against clinical databases.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: VEP
  role: Consequence annotation against the GENCODE v49 cache (--flag_pick_allele),
    attaching HGVSc/HGVSp, canonical/MANE flags and the LOFTEE loss-of-function plugin
- name: vcfanno
  role: Attaches ClinVar, gnomAD AFs, dbNSFP predictions, cancerhotspots.org codon
    matches, CIViC biomarker evidence and GWAS Catalog hits as INFO fields
- name: ClinVar
  role: Database of clinically related variants consulted for clinical significance
    classification of germline variants
- name: CIViC
  role: Source of clinical interpretations of variants in cancer used for germline
    biomarker detection (prognosis, diagnosis, drug sensitivity/resistance)
- name: gnomAD
  role: Population allele frequencies (r4.1 exome subpopulation and global AFs, r2.1.1
    non-cancer subset, genome AFs via --af_gnomadg) used for ACMG BA1/PM2-type evidence
- name: dbNSFP
  role: In-silico deleteriousness predictions (SIFT, PolyPhen2, CADD, AlphaMissense,
    dbscSNV ADA/RF) attached per variant
- name: CPSR
  role: The enclosing workflow that validates the VCF, orchestrates VEP/vcfanno annotation
    and writes the tagged annotated VCF
  repo: https://github.com/sigven/cpsr
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:cb7e48d28534ccd7d6614452b740651446cd534e194e955e0dd43ac6b14cf917
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
  - ClinVar
  - Docker
  - NHGRI-EBI GWAS Catalog
  - PCGR
  - VEP
  - bgzip/tabix
  - dbNSFP
  - gnomAD
  - vcfanno
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# clinical-variant-database-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate germline variant calls against public clinical and population databases — ClinVar, CIViC, gnomAD, dbNSFP, cancerhotspots.org and the GWAS Catalog — as CPSR does in its functional variant annotation step using VEP and vcfanno.

## When to use

Use when a germline SNV/InDel VCF (PASS-filtered, multi-allelic sites decomposed, chrM removed) must be enriched with clinical significance, biomarker, population allele frequency and in-silico deleteriousness annotations before ACMG/AMP classification.

## When NOT to use

- Input is somatic tumor variant data — CPSR targets germline variant interpretation, and the accompanying PCGR tool is the intended choice for somatic SNVs/InDels, CNAs and RNA fusions.
- Variants are not in VCF format or are multi-allelic without prior decomposition, since the annotation step expects decomposed PASS records with one picked consequence block each.
- The goal is de novo variant discovery or calling rather than annotation of existing germline calls against clinical databases.

## Inputs

- Germline SNV/InDel VCF from a single sample (PASS-filtered, multi-allelic sites decomposed, chrM variants removed)
- VEP GENCODE cache (v49 for GRCh38, v19 for GRCh37)
- Reference annotation databases: ClinVar, gnomAD r4.1/r2.1.1, dbNSFP v5.3, CIViC, Cancer Hotspots v2/v3, NHGRI-EBI GWAS Catalog, dbSNP build 156

## Outputs

- Bgzip-compressed, tabix-indexed annotated VCF (<sample_id>.cpsr.<genome_assembly>.vcf.gz) with CPSR INFO tags: CSQ, LOF_FILTER, EXONIC_STATUS, CODING_STATUS, gnomADe_AF, MUTATION_HOTSPOT_MATCH, BIOMARKER_MATCH

## How to apply

First validate and preprocess the germline VCF: retain only PASS records, decompose multi-allelic sites and drop mitochondrial variants, since downstream classification and reporting assume one clean record per variant. Run Variant Effect Predictor (v115 against the GENCODE v49 cache, v19 for GRCh37) with --flag_pick_allele so exactly one main consequence block (CSQ) per variant carries HGVSc/HGVSp, canonical/MANE transcript flags and NearestExonJB fields, and apply the LOFTEE plugin with its filter exceptions — GC>GT at splice donor sites or truncations within the last 5% of the coding sequence are tagged LOF_FILTER rather than true LOF. Then use vcfanno to attach ClinVar, gnomAD r4.1 exome subpopulation/global AFs plus r2.1.1 non-cancer subset AF/AC/AN (supplemented by --af_gnomadg genome AFs), dbNSFP v5.3 predictions (SIFT, PolyPhen2, CADD, AlphaMissense, dbscSNV ADA/RF), cancerhotspots.org codon matches, CIViC biomarker evidence and NHGRI-EBI GWAS Catalog hits. Write the result as a bgzipped, tabix-indexed VCF with named CPSR INFO tags (CSQ, LOF_FILTER, EXONIC_STATUS, CODING_STATUS, gnomAD AF fields, MUTATION_HOTSPOT, BIOMARKER_MATCH). These annotations then feed the automated classification procedure built on more than 30 refined ACMG criteria and the PVS1 decision tree for LOF variants.

## Related tools

- **VEP** (Consequence annotation against the GENCODE v49 cache (--flag_pick_allele), attaching HGVSc/HGVSp, canonical/MANE flags and the LOFTEE loss-of-function plugin)
- **vcfanno** (Attaches ClinVar, gnomAD AFs, dbNSFP predictions, cancerhotspots.org codon matches, CIViC biomarker evidence and GWAS Catalog hits as INFO fields)
- **ClinVar** (Database of clinically related variants consulted for clinical significance classification of germline variants)
- **CIViC** (Source of clinical interpretations of variants in cancer used for germline biomarker detection (prognosis, diagnosis, drug sensitivity/resistance))
- **gnomAD** (Population allele frequencies (r4.1 exome subpopulation and global AFs, r2.1.1 non-cancer subset, genome AFs via --af_gnomadg) used for ACMG BA1/PM2-type evidence)
- **dbNSFP** (In-silico deleteriousness predictions (SIFT, PolyPhen2, CADD, AlphaMissense, dbscSNV ADA/RF) attached per variant)
- **CPSR** (The enclosing workflow that validates the VCF, orchestrates VEP/vcfanno annotation and writes the tagged annotated VCF) — https://github.com/sigven/cpsr

## Evaluation signals

- Output file <sample_id>.cpsr.<genome_assembly>.vcf.gz exists and is a valid bgzipped/tabix-indexed VCF.
- Required INFO fields are present in the annotated VCF: CSQ, LOF_FILTER, gnomADe_AF, MUTATION_HOTSPOT_MATCH, BIOMARKER_MATCH.
- Exactly one picked consequence block (CSQ) exists per variant, a consequence of --flag_pick_allele on a decomposed VCF.
- LOF calls for stop_gained/frameshift and canonical splice donor/acceptor variants carry LOF_FILTER tags only for the defined exceptions (GC>GT at splice donors, last-5% CDS truncations).
- Input preprocessing invariants hold: only PASS records retained, multi-allelic sites decomposed, no chrM variants in the annotated output.

## Limitations

- LOFTEE-style LOF calls have defined exceptions: GC to GT at splice donor sites or truncations within the last 5% of coding sequence are flagged via LOF_FILTER rather than counted as putative LOF.
- Variant classification settings (gnomAD MAF threshold, ClinVar trust level, secondary findings, pharmacogenomics findings, GWAS hits, noncoding filter) must be configured; default behavior may not suit every panel.
- Only PASS-designated VCF records are used for reporting, so unfiltered or non-PASS calls are silently excluded.
- Annotation versions drift with releases (e.g. ClinVar 2026-06, CIViC 2026-06-20, VEP v115/GENCODE v49); results are version-dependent.

## Evidence
