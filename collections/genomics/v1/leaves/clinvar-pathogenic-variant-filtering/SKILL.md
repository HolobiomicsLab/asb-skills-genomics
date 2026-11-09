---
name: clinvar-pathogenic-variant-filtering
description: Use when a raw ClinVar VCF (e.g. clinvar_YYYYMMDD.vcf.gz) must be reduced
  to pathogenic SNVs before projecting variants onto Pfam domain consensus positions
  or comparing against gnomAD population variation.
when_to_use_negative:
- The variant set is already filtered/annotated with pathogenicity and domain positions
  (e.g. MetaDome prebuilt visualizations) — refiltering would change counts relative
  to the released dataset
- The goal is population (benign) variation analysis — gnomAD PASS-filtered variants,
  not ClinVar pathogenic variants, are the correct input
- The variants of interest are indels or non-SNV complex variants — the workflow retains
  only single-nucleotide SNVs
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: MetaDome
  role: Consumes the filtered ClinVar pathogenic SNVs, mapping them through its PostgreSQL
    mapping database onto Pfam domain consensus positions for meta-domain analysis
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL
  role: Stores the mapping database ('mappings' table) and InterProScan domain annotations
    used to translate filtered variant positions to Pfam consensus positions
  repo: https://github.com/cmbi/metadome
- name: Python
  role: Scripting the VCF filtering (SNV + Pathogenic status selection) and downstream
    count/tabulation logic
  repo: https://github.com/cmbi/metadome
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:f0b67a2797ed94a44d01524635f2c5bb8be65d60967724e8e8a9513927e3e345
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/laurensvdwiel/metadome
  repo_url: https://github.com/cmbi/metadome
  tools:
  - Docker
  - MetaDome
  - PostgreSQL
  - Python
  - docker-compose
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# clinvar-pathogenic-variant-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduces a raw ClinVar VCF release to disease-causing (Pathogenic) SNVs—exactly the 34,076 mutations MetaDome ingested—so pathogenic missense variants can be mapped onto Pfam domain consensus positions for meta-domain analysis.

## When to use

Use when a raw ClinVar VCF (e.g. clinvar_YYYYMMDD.vcf.gz) must be reduced to pathogenic SNVs before projecting variants onto Pfam domain consensus positions or comparing against gnomAD population variation.

## When NOT to use

- The variant set is already filtered/annotated with pathogenicity and domain positions (e.g. MetaDome prebuilt visualizations) — refiltering would change counts relative to the released dataset
- The goal is population (benign) variation analysis — gnomAD PASS-filtered variants, not ClinVar pathogenic variants, are the correct input
- The variants of interest are indels or non-SNV complex variants — the workflow retains only single-nucleotide SNVs

## Inputs

- ClinVar VCF with tabix index (clinvar_YYYYMMDD.vcf.gz + .tbi), e.g. the 2018-05-03 release used in the article
- MetaDome prebuilt mapping database CSV (e.g. MetaDome_v2.0_GRCh37.p13_GENCODE-v19_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz) loaded into PostgreSQL
- Pfam-A domain annotations from InterProScan (interpro_domains table)

## Outputs

- Filtered set of pathogenic SNVs (34,076 expected for ClinVar 2018-05-03)
- Per-variant Pfam domain assignment with domain consensus position
- Per-dataset counts of pathogenic missense variants inside vs. outside Pfam domains (article baseline: ~71–72% inside)

## How to apply

Load the ClinVar VCF (MetaDome used the 2018-05-03 release; the current codebase was tested with version 20251006 for both GRCh37 and GRCh38) and retain only SNVs with single-nucleotide ref/alt alleles annotated with Pathogenic (disease-causing) status in the CLNSIG field, yielding 34,076 variants for the 2018-05-03 release. These filtered variants are then translated from genomic coordinates to UniProt protein residues via the MetaDome PostgreSQL 'mappings' table (populated from GENCODE/BLAST/Swiss-Prot alignments), and intersected with InterProScan Pfam-A domain annotations (interpro_domains table) to assign Pfam domains and consensus positions. Apply the same PASS-style filter logic used for gnomAD so population and pathogenic datasets are processed consistently—only variants passing filter criteria are considered true variants. Rationale: restricting to pathogenic SNVs matches the paper's coverage baseline (71–72% of disease-causing missense variants fall in Pfam domains) and makes counts comparable to the article's validation totals.

## Related tools

- **MetaDome** (Consumes the filtered ClinVar pathogenic SNVs, mapping them through its PostgreSQL mapping database onto Pfam domain consensus positions for meta-domain analysis) — https://github.com/cmbi/metadome
- **PostgreSQL** (Stores the mapping database ('mappings' table) and InterProScan domain annotations used to translate filtered variant positions to Pfam consensus positions) — https://github.com/cmbi/metadome
- **Python** (Scripting the VCF filtering (SNV + Pathogenic status selection) and downstream count/tabulation logic) — https://github.com/cmbi/metadome

## Examples

```
import pysam; vs = pysam.VariantFile('clinvar_20180503.vcf.gz'); path_snvs = [v for v in vs.fetch() if list(v.filter.keys()) == ['PASS'] and len(v.ref) == len(v.alt) == 1 and 'Pathogenic' in str(v.info.get('CLNSIG',''))]
```

## Evaluation signals

- Variant count for the ClinVar 2018-05-03 release equals the article's 34,076 pathogenic mutations after filtering
- Every retained record is a SNV (single-nucleotide ref and alt) with Pathogenic/disease-causing CLNSIG status
- Intersecting filtered missense variants with Pfam-A domain annotations reproduces the ~71–72% within-domain coverage baseline
- Counts of ClinVar SNVs within a specific domain (e.g. 6 disease-causing SNVs in CDK13 Pkinase/PF00069 for transcript ENST00000181839.4) match the article's case-study values
- Filenames and release conventions match the README expectations: clinvar_YYYYMMDD.vcf.gz plus .tbi under $CLINVAR_DIR/<build>/

## Limitations

- Counts are release-dependent: the article used ClinVar 2018-05-03 (34,076 pathogenic mutations), while the current codebase was tested with version 20251006, so absolute totals will differ between releases
- Only transcripts mapped to a Swiss-Prot protein can be used in the analysis; variants in unmapped transcripts cannot be projected onto Pfam consensus positions
- Only SNVs (synonymous, nonsense, missense for gnomAD; pathogenic SNVs for ClinVar) passing filter criteria are retained — indels and non-PASS records are excluded

## Evidence

- [methods] Load the ClinVar 2018-05-03 VCF and retain SNVs annotated with disease-causing (Pathogenic) status (34,076 variants expected).: [evidence span withheld — anchor status: not_found]
- [abstract] 34,076 pathogenic mutations from ClinVar: "We updated the underlying data of MetaDome to contain information from 56,319 human transcripts, 71,419 protein domains, 12,164,292 genetic variants from gnomAD, and 34,076 pathogenic mutations from"
- [methods] selecting all synonymous, nonsense, and missense variants that meet the PASS filter criteria.: "selecting all synonymous, nonsense, and missense variants that meet the PASS filter criteria. Variants meeting the PASS criteria are considered to be true variants"
- [readme] MetaDome was tested with version 20251006 for GRCh37 and GRCh38: "MetaDome was tested with version 20251006 for GRCh37 and GRCh38, both [available here](ftp://ftp.ncbi.nlm.nih.gov/pub/clinvar/), but later versions may work as well."
- [results] Only the transcripts that are mapped to a Swiss‐Prot protein can be used in the analysis, the others are displayed in gray: "Only the transcripts that are mapped to a Swiss‐Prot protein can be used in the analysis, the others are displayed in gray"
