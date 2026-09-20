---
name: psi-feature-ablation-modeling
description: Use when no wild-type Ψ values exist for the exons of interest and you
  need to run PredPSI-SVR without the '-p' PSI input, or when quantifying the contribution
  of the Ψ feature to ΔΨ prediction accuracy.
when_to_use_negative:
- Wild-type Ψ values ARE available for the exons of interest — use the full PredPSI-SVR
  with the '-p' option and a PSI file instead, since it achieves higher PCCs (0.570
  CV / 0.566 blind test)
- Variants are in intergenic regions or farther than 200 bp from splice sites — the
  built-in filter removes them, so predictions will not be produced for such inputs
- The goal is de novo ΔΨ prediction with a deep-learning method on large training
  data — the small number of vex-seq samples prevented the authors from using more
  powerful algorithms like deep learning
edam_operation: http://edamontology.org/operation_0426
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PredPSI-SVR
  role: Pipeline being run in noPSI (ablated) mode; invoking ./main.sh without the
    '-p' option omits the wild-type Ψ input feature
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: libsvm
  role: Support vector regression engine (libsvm-3.23, built with 'make all') used
    to train and apply the noPSI ΔΨ model
- name: ANNOVAR
  role: Annotates and filters the input VCF with the hg19 ensGene database, removing
    intergenic variants and variants >200 bp from splice sites
- name: samtools
  role: samtools-1.9, built from source, indexes the hg19 FASTA (samtools faidx) used
    during feature extraction
- name: SPIDEX
  role: Third-party splicing database (hg19_spidex.zip) placed in annovar/humandb
    and used as a feature source
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:a8c3a62241a6b7837cfee020dfa69d4a1ddf7de47eb6ae61c67d458cd5bf815f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - ANNOVAR
  - Perl
  - PredPSI-SVR
  - Python 3
  - SPIDEX
  - libsvm
  - samtools
  edam_operation: http://edamontology.org/operation_0426
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# psi-feature-ablation-modeling

## Summary

Trains and evaluates an ablated variant of PredPSI-SVR (PredPSI-SVR-noPSI) that omits the wild-type Ψ input feature, quantifying how much predictive performance is lost when experimentally measured exon inclusion levels are unavailable.

## When to use

Use when no wild-type Ψ values exist for the exons of interest and you need to run PredPSI-SVR without the '-p' PSI input, or when quantifying the contribution of the Ψ feature to ΔΨ prediction accuracy.

## When NOT to use

- Wild-type Ψ values ARE available for the exons of interest — use the full PredPSI-SVR with the '-p' option and a PSI file instead, since it achieves higher PCCs (0.570 CV / 0.566 blind test)
- Variants are in intergenic regions or farther than 200 bp from splice sites — the built-in filter removes them, so predictions will not be produced for such inputs
- The goal is de novo ΔΨ prediction with a deep-learning method on large training data — the small number of vex-seq samples prevented the authors from using more powerful algorithms like deep learning

## Inputs

- Input VCF of genetic variants (hg19 coordinates)
- ANNOVAR ensGene hg19 annotation database in annovar/humandb
- SPIDEX hg19 database (unzipped hg19_spidex.zip) in annovar/humandb
- hg19 genome FASTA (chromFa) indexed with samtools faidx
- TR957 vex-seq training set (CAGI 5 vex-seq challenge)
- TS1098 independent vex-seq test set

## Outputs

- OUTPUT.dpsi file with predicted ΔΨ values per variant in the '-o' output directory
- 10-fold cross-validation PCC on TR957 (expected ~0.525)
- Independent-test PCC on TS1098 (expected ~0.479)

## How to apply

Set up the PredPSI-SVR environment (compile libsvm-3.23 and samtools-1.9 with 'make all', download the ANNOVAR hg19 ensGene database and the SPIDEX hg19 database into annovar/humandb), then invoke ./main.sh on the input VCF WITHOUT the '-p' option so the pipeline runs in PredPSI-SVR-noPSI mode. The pipeline first filters the VCF to remove variants in intergenic regions or more than 200 bp from splice sites using ANNOVAR ensGene (hg19) annotations, then extracts the splicing-related features (MaxEntScan splice-site, conservation, ESE-based and structural features) and trains the libsvm support vector regression model on the TR957 vex-seq training set with 10-fold cross-validation. Apply the trained noPSI model to the TS1098 independent test set and write predicted ΔΨ values to OUTPUT.dpsi in the '-o' output directory. Compare the resulting PCCs (expected ≈0.525 CV on TR957, ≈0.479 blind on TS1098) against the full PredPSI-SVR model (0.570 CV, 0.566 blind test) to quantify the ~0.1 drop attributed to the missing Ψ feature.

## Related tools

- **PredPSI-SVR** (Pipeline being run in noPSI (ablated) mode; invoking ./main.sh without the '-p' option omits the wild-type Ψ input feature) — https://github.com/chenkenbio/PredPSI-SVR
- **libsvm** (Support vector regression engine (libsvm-3.23, built with 'make all') used to train and apply the noPSI ΔΨ model)
- **ANNOVAR** (Annotates and filters the input VCF with the hg19 ensGene database, removing intergenic variants and variants >200 bp from splice sites)
- **samtools** (samtools-1.9, built from source, indexes the hg19 FASTA (samtools faidx) used during feature extraction)
- **SPIDEX** (Third-party splicing database (hg19_spidex.zip) placed in annovar/humandb and used as a feature source)

## Examples

```
./main.sh example/sample.vcf -o example/outdir
```

## Evaluation signals

- 10-fold CV PCC on TR957 reproduces 0.525 within numerical tolerance
- Independent-test PCC on TS1098 reproduces 0.479 within numerical tolerance
- noPSI PCCs are about 0.1 lower than the full PredPSI-SVR model (0.570 CV, 0.566 blind test), matching the paper's ablation claim
- OUTPUT.dpsi is written to the '-o' directory and contains fewer entries than the input VCF when the splice-site/intergenic filter removes variants
- Environment sanity: libsvm-3.23 and samtools-1.9 compile with 'make all', and ensGene/SPIDEX databases are present in annovar/humandb

## Limitations

- Performance drops without experimental Ψ: 'Without experimental Ψ, PCCs on the training set and test set dropped by about 0.1'
- The method requires wild-type Ψ of exons as an input limitation acknowledged by the authors; the noPSI mode is the workaround but is less accurate
- Variants in intergenic regions or distant to splice sites (more than 200 bp) are silently filtered, so output may cover fewer variants than the input VCF
- Only Human BodyMap Project 2.0 RNA-seq data were used to derive Ψ, and tissue-specific Ψ is not incorporated; small sample size also precluded deep learning

## Evidence

./main.sh example/sample.vcf -o example/outdir"
