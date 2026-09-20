---
name: variant-splicing-effect-prediction
description: Use when you have a VCF of variants near exon skipping events and need
  predicted ΔΨ values or prioritization of splicing-affecting (e.g. synonymous) mutations
  on hg19.
when_to_use_negative:
- 'Variants are far from splice sites: the tool automatically drops intergenic variants
  and those more than 200 bp from splice sites, so distant variants yield no predictions.'
- 'Exons lack experimental wild-type Ψ and high accuracy is required: PredPSI-SVR-noPSI
  loses about 0.1 PCC on both training and test sets.'
- 'Branch point or pre-mRNA secondary-structure effects are the target: PredPSI-SVR
  does not include these features effectively due to limited sample numbers.'
edam_operation: http://edamontology.org/operation_0331
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: PredPSI-SVR
  role: Core SVR model that filters the VCF, extracts 42 splicing-related features,
    and predicts ΔΨ; main.sh is the entry point.
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: libsvm
  role: Support vector regression engine used to train and apply the ΔΨ prediction
    model.
- name: ANNOVAR
  role: Annotates variants with hg19 ensGene gene models used for splice-site proximity
    filtering.
- name: samtools
  role: Indexes the hg19 genome FASTA (samtools faidx) for sequence feature extraction.
- name: SPANR
  role: Competing baseline; its SPIDEX ΔΨ scores are compared against PredPSI-SVR
    predictions.
- name: MaxEntScan
  role: Computes splice-site strength features (MES5/MES3) used in feature extraction
    and as a comparison method.
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:54ec5be091638b436b58801e683b280fa540accb6777224abfb35269a1013352
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - ANNOVAR
  - MaxEntScan
  - Perl
  - PredPSI-SVR
  - Python 3
  - SPANR
  - libsvm
  - samtools
  edam_operation: http://edamontology.org/operation_0331
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# variant-splicing-effect-prediction

## Summary

Predict the change of percent spliced in (ΔΨ) caused by genetic variants using the PredPSI-SVR support vector regression model trained on 42 splicing-related features (eight selected by greedy feature selection) from the CAGI 5 vex-seq challenge data.

## When to use

Use when you have a VCF of variants near exon skipping events and need predicted ΔΨ values or prioritization of splicing-affecting (e.g. synonymous) mutations on hg19.

## When NOT to use

- Variants are far from splice sites: the tool automatically drops intergenic variants and those more than 200 bp from splice sites, so distant variants yield no predictions.
- Exons lack experimental wild-type Ψ and high accuracy is required: PredPSI-SVR-noPSI loses about 0.1 PCC on both training and test sets.
- Branch point or pre-mRNA secondary-structure effects are the target: PredPSI-SVR does not include these features effectively due to limited sample numbers.

## Inputs

- Input VCF file of genetic variants (hg19 coordinates)
- Optional wild-type Ψ file (tabular PSI values per exon, supplied via -p)
- hg19 genome FASTA indexed with samtools faidx
- ANNOVAR hg19 ensGene annotation and SPIDEX databases in annovar/humandb

## Outputs

- OUTPUT.dpsi file of predicted ΔΨ values per variant (in the -o output directory)
- OUTPUT.psi file (filtered annotated variants, may be fewer than input VCF)

## How to apply

Set up the environment per src/init.sh: clone PredPSI-SVR, install ANNOVAR, libsvm-3.23 and samtools-1.9 (built with 'make all'), download the hg19 ensGene ANNOVAR database and SPIDEX into annovar/humandb, and index the hg19 chromFa genome with samtools faidx. The tool first filters the input VCF to remove variants in intergenic regions or farther than 200 bp from splice sites, then extracts splicing-related features (MaxEntScan splice-site, conservation, ESE-based, and mRNA structural features) and scores them with the libsvm SVR to output predicted ΔΨ. Run './main.sh input.vcf -p wildtype.psi -o outdir' when experimental wild-type Ψ is available; omit the '-p' option (PredPSI-SVR-noPSI) for exons without available Ψ, accepting a ~0.1 PCC drop. Judge success by Pearson correlation of predicted versus observed ΔΨ (reference: PCC 0.570 in 10-fold CV and 0.566 on the blind vex-seq test) or, for synonymous-mutation prioritization, by ranking variants by |ΔΨ| and computing AUC against pathogenic labels.

## Related tools

- **PredPSI-SVR** (Core SVR model that filters the VCF, extracts 42 splicing-related features, and predicts ΔΨ; main.sh is the entry point.) — https://github.com/chenkenbio/PredPSI-SVR
- **libsvm** (Support vector regression engine used to train and apply the ΔΨ prediction model.)
- **ANNOVAR** (Annotates variants with hg19 ensGene gene models used for splice-site proximity filtering.)
- **samtools** (Indexes the hg19 genome FASTA (samtools faidx) for sequence feature extraction.)
- **SPANR** (Competing baseline; its SPIDEX ΔΨ scores are compared against PredPSI-SVR predictions.)
- **MaxEntScan** (Computes splice-site strength features (MES5/MES3) used in feature extraction and as a comparison method.)

## Examples

```
cd ~/PredPSI-SVR/ && ./main.sh example/sample.vcf -p example/sample.psi -o example/outdir
```

## Evaluation signals

- 10-fold cross-validation PCC on the TR957 vex-seq training set should reach ~0.570 for the full model (0.525 for the noPSI variant).
- Independent blind-test PCC on TS1098 should reach ~0.566 for the full model (0.479 for the noPSI variant).
- OUTPUT.dpsi should contain predictions only for variants within 200 bp of splice sites and not in intergenic regions; OUTPUT.psi may contain fewer variants than the input VCF.
- For synonymous-mutation prioritization, |ΔΨ| ranking should yield AUC ≈ 0.579 on SynonMut-psi (0.508 noPSI; 0.575 on SynonMut-complete), significantly better than SPANR.
- Running with the -p option versus without should be the only difference when reproducing PredPSI-SVR vs PredPSI-SVR-noPSI results.

## Limitations

- The method requires wild-type Ψ of exons; without experimental Ψ, PCCs on the training set and test set dropped by about 0.1.
- PredPSI-SVR does not include branch point sequence features or pre-mRNA secondary structure effectively due to the relatively small number of samples.
- The small sample size also prevented use of more powerful algorithms like deep learning.
- Only Human BodyMap Project 2.0 RNA-seq data were used; many exons could not be measured due to sequencing depth, and tissue-specific Ψ is not incorporated.

## Evidence
