---
name: variant-consequence-annotation
description: Use when a single-sample germline VCF (SNVs/InDels, PASS records) needs
  functional consequence annotation (VEP/GENCODE, LOFTEE, dbNSFP, gnomAD, ClinVar,
  CIViC) as a preprocessing step for cancer predisposition variant classification.
when_to_use_negative:
- Input is somatic tumor variant data — CPSR targets germline predisposition; the
  README recommends PCGR for 'interrogation of germline variants' counterpart somatic
  interpretation
- Input VCF contains multi-allelic sites that have not been decomposed, or records
  not designated PASS in the FILTER column
- Analysis concerns mitochondrial (chrM) variants, which CPSR removes before annotation
edam_operation: http://edamontology.org/operation_0331
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: CPSR
  role: Workflow that orchestrates preprocessing, VEP consequence annotation, LOFTEE
    LOF flagging, and vcfanno database attachment
  repo: https://github.com/sigven/cpsr
- name: VEP
  role: Variant Effect Predictor (v115) computes consequence annotations against the
    GENCODE v49/v19 cache with --flag_pick_allele and the NearestExonJB plugin
- name: vcfanno
  role: Attaches ClinVar, gnomAD AFs, dbNSFP v5.3 predictions, cancer hotspot codons,
    CIViC biomarkers and GWAS Catalog hits as INFO tags
- name: pcgrr
  role: R package on which CPSR builds for report generation downstream of annotation
  repo: https://github.com/sigven/pcgr/tree/master/pcgrr
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ijc.33749
    title: 'Cancer Predisposition Sequencing Reporter ( <scp>CPSR</scp> ): A flexible
      variant report engine for high‐throughput germline screening in cancer'
  source_content_hash: sha256:d862be847377705842232cd275f912e347fd6bcf4df9dcbd2788c67bdd19508c
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
  - pcgrr
  - vcfanno
  edam_operation: http://edamontology.org/operation_0331
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# variant-consequence-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate germline SNV/InDel VCFs with variant consequence, loss-of-function, population frequency, deleteriousness, hotspot, and biomarker evidence as implemented in the CPSR workflow, producing a bgzipped/tabix-indexed VCF with CPSR INFO tags.

## When to use

Use when a single-sample germline VCF (SNVs/InDels, PASS records) needs functional consequence annotation (VEP/GENCODE, LOFTEE, dbNSFP, gnomAD, ClinVar, CIViC) as a preprocessing step for cancer predisposition variant classification.

## When NOT to use

- Input is somatic tumor variant data — CPSR targets germline predisposition; the README recommends PCGR for 'interrogation of germline variants' counterpart somatic interpretation
- Input VCF contains multi-allelic sites that have not been decomposed, or records not designated PASS in the FILTER column
- Analysis concerns mitochondrial (chrM) variants, which CPSR removes before annotation

## Inputs

- Single-sample germline variant calls (SNVs/InDels) in VCF format with PASS FILTER entries
- VEP cache: GENCODE v49 (GRCh38) / v19 (GRCh37)
- Annotation databases via vcfanno: ClinVar, gnomAD r4.1 exome + r2.1.1 non-cancer subsets, dbNSFP v5.3, Cancer Hotspots v2/v3, CIViC, NHGRI-EBI GWAS Catalog

## Outputs

- <sample_id>.cpsr.<genome_assembly>.vcf.gz — bgzipped, tabix-indexed annotated VCF
- CPSR INFO tags: CSQ, LOF_FILTER, EXONIC_STATUS, CODING_STATUS, gnomADe_AF fields, MUTATION_HOTSPOT, BIOMARKER_MATCH, gene-level tags

## How to apply

First validate and preprocess the input germline VCF: retain only PASS records, decompose multi-allelic sites, and drop mitochondrial (chrM) variants. Then run VEP v115 against the GENCODE v49 cache (v19 for grch37) with --flag_pick_allele so exactly one main consequence block per variant is selected, attaching HGVSc/HGVSp, canonical/MANE transcript flags and NearestExonJB plugin fields, and apply LOFTEE with filter exceptions: stop_gained/frameshift and canonical splice donor/acceptor variants are LOF unless a GC>GT substitution occurs at a splice donor site or the truncation falls within the last 5% of the coding sequence (LOF_FILTER tag). Finally, use vcfanno to attach ClinVar, gnomAD r4.1 exome subpopulation/global AFs plus r2.1.1 non-cancer AF/AC/AN (supplemented by --af_gnomadg genome AFs), dbNSFP v5.3 in-silico predictions (SIFT, PolyPhen2, CADD, AlphaMissense, dbscSNV ADA/RF), cancerhotspots.org codon matches, CIViC biomarker evidence, and NHGRI-EBI GWAS Catalog hits. Write the annotated VCF with the named CPSR INFO tags, bgzip-compressed and tabix-indexed.

## Related tools

- **CPSR** (Workflow that orchestrates preprocessing, VEP consequence annotation, LOFTEE LOF flagging, and vcfanno database attachment) — https://github.com/sigven/cpsr
- **VEP** (Variant Effect Predictor (v115) computes consequence annotations against the GENCODE v49/v19 cache with --flag_pick_allele and the NearestExonJB plugin)
- **vcfanno** (Attaches ClinVar, gnomAD AFs, dbNSFP v5.3 predictions, cancer hotspot codons, CIViC biomarkers and GWAS Catalog hits as INFO tags)
- **pcgrr** (R package on which CPSR builds for report generation downstream of annotation) — https://github.com/sigven/pcgr/tree/master/pcgrr

## Examples

```
vep -i germline.pass.vcf.gz --cache --assembly GRCh38 --cache_version 115 --flag_pick_allele --hgvs --plugin NearestExonJB -o sample.cpsr.annotated.vcf
```

## Evaluation signals

- Output file <sample_id>.cpsr.<genome_assembly>.vcf.gz exists and is a valid bgzipped/tabix-indexed VCF
- Annotated VCF contains required INFO fields: CSQ, LOF_FILTER, gnomADe_AF, MUTATION_HOTSPOT_MATCH, BIOMARKER_MATCH
- Exactly one picked consequence block per variant (from VEP --flag_pick_allele)
- LOF_FILTER exceptions applied correctly: GC>GT at splice donor sites and truncations within the last 5% of coding sequence are flagged
- No chrM variants and no multi-allelic sites remain in the annotated output

## Limitations

- Only SNVs/InDels from a single sample in VCF format are accepted; CNVs/fusions are out of scope
- Mitochondrial (chrM) variants are removed before annotation, so they cannot be reported
- Non-PASS records are excluded from reporting; multi-allelic sites must be decomposed first

## Evidence

- [methods] VEP consequence annotations: [evidence span withheld — anchor status: not_found]
