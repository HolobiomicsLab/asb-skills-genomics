---
name: synonymous-mutation-dataset-curation
description: Use when building a pathogenic-vs-normal synonymous mutation benchmark
  to evaluate splicing-impact predictors (e.g. PredPSI-SVR, SPANR/SPIDEX), especially
  when exonic variants near splice sites with predicted ΔΨ scores are needed.
when_to_use_negative:
- Variants are already annotated and carry SPIDEX/SPANR scores — use PredPSI-SVR prediction
  directly instead of re-curating a benchmark
- Goal is predicting ΔΨ for new variants rather than building an evaluation benchmark
  — run PredPSI-SVR's main.sh pipeline on a VCF instead
- Variants are non-synonymous or fall more than 300 bp from splice sites — they are
  excluded by the dataset's splice-proximity and mutation-class filters
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: CrossMap
  role: Lifts pathogenic and normal synonymous variants from hg38 to hg19 coordinates
- name: ANNOVAR
  role: Annotates lifted variants against ensGene hg19 to determine splice-site distance
    and filter within 300 bp
  repo: http://annovar.openbioinformatics.org/en/latest/user-guide/download/
- name: SPIDEX
  role: Third-party hg19_spidex database providing SPANR-predicted ΔΨ scores joined
    to each curated variant
  repo: http://www.openbioinformatics.org/annovar/spidex_download_form.php
- name: samtools
  role: Indexes the hg19 chromFa reference genome (samtools faidx) used during liftover
  repo: http://www.htslib.org/download/
- name: SPANR
  role: Source of SPIDEX ΔΨ predictions used to require and label each benchmark mutation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:5a425042cbf52c1cb76c88ef21b2c516a31f1b9fc3c7fe727fe0fd0fe0d15707
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ANNOVAR
  - CrossMap
  - PredPSI-SVR
  - SPANR
  - SPIDEX
  - samtools
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# synonymous-mutation-dataset-curation

## Summary

Curates labeled synonymous-mutation benchmark datasets (SynonMut-complete and SynonMut-psi) by combining dbDSM pathogenic synonymous mutations with 1000 Genomes normal variants (AF 0.1-0.9), lifting hg38 coordinates to hg19, and filtering to splice-proximal variants carrying SPANR/SPIDEX scores for evaluating PredPSI-SVR.

## When to use

Use when building a pathogenic-vs-normal synonymous mutation benchmark to evaluate splicing-impact predictors (e.g. PredPSI-SVR, SPANR/SPIDEX), especially when exonic variants near splice sites with predicted ΔΨ scores are needed.

## When NOT to use

- Variants are already annotated and carry SPIDEX/SPANR scores — use PredPSI-SVR prediction directly instead of re-curating a benchmark
- Goal is predicting ΔΨ for new variants rather than building an evaluation benchmark — run PredPSI-SVR's main.sh pipeline on a VCF instead
- Variants are non-synonymous or fall more than 300 bp from splice sites — they are excluded by the dataset's splice-proximity and mutation-class filters

## Inputs

- dbDSM pathogenic synonymous mutation catalog
- 1000 Genomes Project normal synonymous variants (VCF), allele frequency 0.1-0.9
- hg38-to-hg19 liftover chain file
- hg19 reference genome (UCSC goldenPath chromFa, samtools-faidx indexed FASTA)
- ANNOVAR ensGene hg19 annotation database
- hg19 SPIDEX database (hg19_spidex.zip) with SPANR ΔΨ scores
- Exon-level experimental Ψ values for the SynonMut-psi subset

## Outputs

- SynonMut-complete CSV table: 757 pathogenic / 10822 normal synonymous mutations, each within 300 bp of a splice site and carrying a SPANR score
- SynonMut-psi CSV table: 87 pathogenic / 826 normal synonymous mutations mapped to exons with experimental Ψ

## How to apply

Download the pathogenic synonymous mutation catalog from dbDSM and normal synonymous variants from the 1000 Genomes Project, keeping only normal variants with allele frequency between 0.1 and 0.9. Convert all variants from hg38 to hg19 with CrossMap against the UCSC hg19 chromFa reference indexed by samtools faidx, dropping variants that fail to lift over. Annotate the lifted variants against the ANNOVAR ensGene hg19 database (annotate_variation.pl -buildver hg19 -downdb -webfrom annovar ensGene) and keep only mutations within 300 bp of a splice site, discarding intergenic and distant variants. Join SPANR/SPIDEX predicted ΔΨ scores from the hg19_spidex database to each variant and remove mutations lacking SPANR scores. Assemble SynonMut-complete (757 pathogenic / 10822 normal) and extract the Ψ-bearing subset SynonMut-psi (87 pathogenic / 826 normal) as CSV tables.

## Related tools

- **CrossMap** (Lifts pathogenic and normal synonymous variants from hg38 to hg19 coordinates)
- **ANNOVAR** (Annotates lifted variants against ensGene hg19 to determine splice-site distance and filter within 300 bp) — http://annovar.openbioinformatics.org/en/latest/user-guide/download/
- **SPIDEX** (Third-party hg19_spidex database providing SPANR-predicted ΔΨ scores joined to each curated variant) — http://www.openbioinformatics.org/annovar/spidex_download_form.php
- **samtools** (Indexes the hg19 chromFa reference genome (samtools faidx) used during liftover) — http://www.htslib.org/download/
- **SPANR** (Source of SPIDEX ΔΨ predictions used to require and label each benchmark mutation)

## Examples

```
./annotate_variation.pl -buildver hg19 -downdb -webfrom annovar ensGene ./humandb/
```

## Evaluation signals

- SynonMut-complete contains exactly 757 pathogenic and 10822 normal synonymous mutations
- SynonMut-psi contains exactly 87 pathogenic and 826 normal mutations with experimental Ψ
- Every retained variant lies within the 300 bp splice-site window per ANNOVAR ensGene hg19 annotation
- Every retained variant carries a SPANR/SPIDEX score; mutations lacking SPANR scores were removed
- All normal variants fall within allele frequency 0.1-0.9 in the 1000 Genomes set, and variants failing hg38→hg19 liftOver were dropped

## Limitations

- Variants that fail CrossMap hg38→hg19 liftOver are silently dropped from the benchmark
- Requiring a SPANR score and a 300 bp splice-site window discards many mutations, reducing benchmark coverage
- The Ψ-bearing subset is small (87 pathogenic / 826 normal); the article notes small sample numbers prevented use of more powerful algorithms like deep learning
- PredPSI-SVR additionally filters variants more than 200 bp from splice sites at prediction time, so benchmark variants beyond 200 bp may be excluded at inference

## Evidence

- [other] The synonymous mutation benchmark was built from dbDSM pathogenic mutations and 1000 Genomes normal mutations (AF 0.1-0.9), converted hg38 to hg19 with CrossMap: [evidence span withheld — anchor status: not_found]
- [other] yielding SynonMut-complete (757 pathogenic / 10822 normal); mapping to exons with experimental Ψ gave SynonMut-psi (87 pathogenic / 826 normal): [evidence span withheld — anchor status: not_found]
