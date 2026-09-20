---
name: spidex-dpsi-score-integration
description: Use when variant sets (e.g. dbDSM pathogenic or 1000 Genomes normal synonymous
  mutations) need SPANR/SPIDEX predicted dPsi scores joined on, with unscored variants
  removed before benchmarking or model evaluation.
when_to_use_negative:
- When variants already carry experimental or predicted dPsi values, or when the analysis
  needs experimentally measured Psi rather than SPANR-predicted dPsi (PredPSI-SVR
  itself notes performance drops without experimental Psi)
- When variants are intergenic or distant from splice sites (>200 bp per the PredPSI-SVR
  VCF filter; >300 bp per the SynonMut benchmark window) — they will be removed anyway
- When working directly in hg38 without performing CrossMap liftover, since hg19_spidex
  coordinates only apply to hg19
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: SPIDEX
  role: Third-party database of SPANR-predicted dPsi scores joined to each variant
    via the hg19_spidex ANNOVAR database
  repo: http://tools.genes.toronto.edu/
- name: SPANR
  role: Source method behind the SPIDEX dPsi predictions used as the comparable-score
    column for benchmark variants
- name: ANNOVAR
  role: Annotation framework whose humandb hosts both the ensGene hg19 gene model
    and the hg19_spidex database used to attach dPsi scores
  repo: http://annovar.openbioinformatics.org/en/latest/user-guide/download/
- name: CrossMap
  role: Converts hg38 variant coordinates to hg19 so they are compatible with the
    hg19_spidex database
- name: samtools
  role: Indexes the hg19 reference FASTA (samtools faidx hg19.fasta) supporting the
    coordinate conversion and annotation steps
  repo: http://www.htslib.org/download/
- name: PredPSI-SVR
  role: Pipeline whose splice-site VCF filter and SPIDEX-based evaluation the scored
    variants feed into
  repo: https://github.com/chenkenbio/PredPSI-SVR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:f9c3eec04cab3876a5ba11f051c4afa181ec04b8511a873729f39f219874c3bc
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# spidex-dpsi-score-integration

## Summary

Attaches SPANR/SPIDEX predicted delta-PSI (SPIDEX dPsi) scores to genetic variants via the hg19_spidex ANNOVAR database, and drops variants that lack a SPANR score. This score join underpins benchmark construction (e.g. the SynonMut-complete and SynonMut-psi synonymous mutation sets used to test PredPSI-SVR's ability to prioritize deleterious synonymous mutations).

## When to use

Use when variant sets (e.g. dbDSM pathogenic or 1000 Genomes normal synonymous mutations) need SPANR/SPIDEX predicted dPsi scores joined on, with unscored variants removed before benchmarking or model evaluation.

## When NOT to use

- When variants already carry experimental or predicted dPsi values, or when the analysis needs experimentally measured Psi rather than SPANR-predicted dPsi (PredPSI-SVR itself notes performance drops without experimental Psi)
- When variants are intergenic or distant from splice sites (>200 bp per the PredPSI-SVR VCF filter; >300 bp per the SynonMut benchmark window) — they will be removed anyway
- When working directly in hg38 without performing CrossMap liftover, since hg19_spidex coordinates only apply to hg19

## Inputs

- Variant lists in VCF-like form: dbDSM pathogenic synonymous mutations and 1000 Genomes normal synonymous variants (allele frequency 0.1-0.9), initially in hg38 coordinates
- hg19_spidex.zip SPIDEX database (from http://www.openbioinformatics.org/annovar/spidex_download_form.php)
- ANNOVAR humandb with ensGene hg19 annotation
- hg19 reference genome FASTA (UCSC goldenPath chromFa.tar.gz) indexed with samtools faidx

## Outputs

- Variant tables in which each retained mutation is annotated with its SPANR/SPIDEX predicted dPsi score
- Labeled benchmark CSV tables (SynonMut-complete: 757 pathogenic / 10822 normal; SynonMut-psi: 87 pathogenic / 826 normal) containing only splice-proximal mutations with SPANR scores

## How to apply

Download the SPIDEX third-party database from the ANNOVAR spidex download form, place hg19_spidex.zip in PredPSI-SVR/tools/annovar/humandb, and decompress it with unzip so it sits alongside the ANNOVAR ensGene hg19 database (downloaded with annotate_variation.pl -buildver hg19 -downdb -webfrom annovar ensGene). Because the hg19_spidex database is hg19-based, first convert input variants from hg38 to hg19 with CrossMap against the UCSC hg19 chromFa reference (indexed with samtools faidx), dropping variants that fail to lift over. Annotate the lifted variants against ensGene hg19, keep only mutations within the splice-site window (the article's benchmark uses 300 bp; PredPSI-SVR itself filters intergenic variants and variants more than 200 bp from splice sites), then join the SPIDEX/SPANR predicted dPsi score to each variant. Finally remove any mutation lacking a SPANR score, since downstream PredPSI-SVR evaluation against SPANR requires a comparable predicted dPsi for every variant.

## Related tools

- **SPIDEX** (Third-party database of SPANR-predicted dPsi scores joined to each variant via the hg19_spidex ANNOVAR database) — http://tools.genes.toronto.edu/
- **SPANR** (Source method behind the SPIDEX dPsi predictions used as the comparable-score column for benchmark variants)
- **ANNOVAR** (Annotation framework whose humandb hosts both the ensGene hg19 gene model and the hg19_spidex database used to attach dPsi scores) — http://annovar.openbioinformatics.org/en/latest/user-guide/download/
- **CrossMap** (Converts hg38 variant coordinates to hg19 so they are compatible with the hg19_spidex database)
- **samtools** (Indexes the hg19 reference FASTA (samtools faidx hg19.fasta) supporting the coordinate conversion and annotation steps) — http://www.htslib.org/download/
- **PredPSI-SVR** (Pipeline whose splice-site VCF filter and SPIDEX-based evaluation the scored variants feed into) — https://github.com/chenkenbio/PredPSI-SVR

## Examples

```
cd ~/PredPSI-SVR/tools/annovar/humandb && unzip hg19_spidex.zip
```

## Evaluation signals

- Every retained variant row carries a non-null SPANR/SPIDEX dPsi score; mutations lacking SPANR scores are absent from the output table
- Decompressed hg19_spidex database is present in PredPSI-SVR/tools/annovar/humandb alongside ensGene hg19 annotation, and ANNOVAR queries against it succeed on hg19 coordinates
- All output variants lie within the splice-site window (300 bp for the benchmark; PredPSI-SVR's own filter removes intergenic variants and those >200 bp from splice sites)
- Assembled benchmark counts match expectations: 757 pathogenic / 10822 normal (SynonMut-complete) and 87 pathogenic / 826 normal (SynonMut-psi)
- Normal-variant subset is restricted to 1000 Genomes alleles with AF between 0.1 and 0.9, and all coordinates are hg19 after CrossMap liftover

## Limitations

- SPIDEX provides SPANR's predicted dPsi indirectly via a deep Bayes network, and the article notes indirect dPsi prediction is usually less accurate than methods designed for direct prediction
- hg19_spidex is hg19-specific: variants that fail CrossMap hg38-to-hg19 liftover are dropped, and hg38-native analyses need conversion first
- SPIDEX coverage is incomplete — some variants simply lack SPANR scores and must be removed, shrinking the benchmark
- PredPSI-SVR's VCF filtering (intergenic or >200 bp from splice sites) means output variant counts can be lower than the input VCF count

## Evidence
