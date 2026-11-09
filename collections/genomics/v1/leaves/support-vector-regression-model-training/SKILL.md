---
name: support-vector-regression-model-training
description: Use when predicting ΔΨ caused by genetic variants from a VCF with wild-type
  Ψ available, and when you need to train or reproduce an SVR model on vex-seq-style
  splicing features (TR957/TS1098) and report Pearson correlation coefficients.
when_to_use_negative:
- Wild-type Ψ of the exons is not available — use the PredPSI-SVR-noPSI variant (./main.sh
  without '-p'), which trades about 0.1 PCC for independence from experimental Ψ
- Variants lie in intergenic regions or more than 200 bp from splice sites — the pipeline's
  ANNOVAR-based filter removes them, so an SVR prediction will not be returned for
  those variants
- You need branch-point sequence or effective pre-mRNA secondary-structure features,
  or tissue-specific Ψ prediction — PredPSI-SVR omits these due to the small sample
  size
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: PredPSI-SVR
  role: 'End-to-end pipeline: VCF filtering, feature extraction, SVR training/prediction,
    and ΔΨ output; invoked via main.sh with optional ''-p'' Ψ file'
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: libsvm
  role: ε-SVR implementation with RBF kernel used for training, grid search of C/γ,
    and 10-fold cross-validation; compiled from source (libsvm-3.23) in PredPSI-SVR/tools
  repo: https://www.csie.ntu.edu.tw/~cjlin/libsvm
- name: ANNOVAR
  role: Annotates the input VCF with ensGene (hg19) to filter intergenic/distant variants
    and extract splicing-related features
  repo: http://annovar.openbioinformatics.org/en/latest/user-guide/download/
- name: samtools
  role: Index the hg19 genome FASTA (samtools faidx) for feature extraction from flanking
    sequences; built from source (samtools-1.9)
  repo: http://www.htslib.org/download/
- name: SilVA
  role: Source of ESE-based splicing-enhancer features; PredPSI-SVR/tools/ese3/ese3_mod.py
    is modified from the SilVA package
  repo: https://github.com/buske/silva
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_006
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:9d5498e5d0eff745ecaaacf379721b7ba0a2e54189981b66450c0fca77563a65
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - ANNOVAR
  - Perl
  - PredPSI-SVR
  - Python 3
  - SilVA
  - libsvm
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# support-vector-regression-model-training

## Summary

Train an ε-SVR (LIBSVM, RBF kernel) to predict variant-induced ΔΨ using 42 splicing-related features extracted from the exon and flanking regions of the CAGI-5 vex-seq TR957 training set, with grid-searched C/γ under 10-fold cross-validation and greedy forward feature selection that peaks at 8 features (PCC 0.570).

## When to use

Use when predicting ΔΨ caused by genetic variants from a VCF with wild-type Ψ available, and when you need to train or reproduce an SVR model on vex-seq-style splicing features (TR957/TS1098) and report Pearson correlation coefficients.

## When NOT to use

- Wild-type Ψ of the exons is not available — use the PredPSI-SVR-noPSI variant (./main.sh without '-p'), which trades about 0.1 PCC for independence from experimental Ψ
- Variants lie in intergenic regions or more than 200 bp from splice sites — the pipeline's ANNOVAR-based filter removes them, so an SVR prediction will not be returned for those variants
- You need branch-point sequence or effective pre-mRNA secondary-structure features, or tissue-specific Ψ prediction — PredPSI-SVR omits these due to the small sample size

## Inputs

- Input VCF of genetic variants (hg19 coordinates)
- Wild-type Ψ (percent spliced in) file for the exons, e.g. example/sample.psi (required for full PredPSI-SVR; omitted for noPSI mode)
- vex-seq training set (TR957) and independent test set (TS1098) ΔΨ values
- Precomputed matrix of 42 splicing-related features per variant (splice-site, conservation, sequence-motif, regulatory-element, mRNA-structure features)

## Outputs

- OUTPUT.dpsi — predicted ΔΨ values per variant in the '-o' output directory
- OUTPUT.psi — filtered/annotated wild-type Ψ table (may contain fewer variants than the input VCF due to splice-site proximity filtering)
- Trained LIBSVM model file (ε-SVR, RBF kernel with grid-searched C/γ)
- 10-fold cross-validation predicted-versus-observed ΔΨ table and Pearson correlation coefficients (e.g. pcc_results.csv with CV PCC 0.590 and blind test PCC 0.577 for the 6-feature model)

## How to apply

Set up PredPSI-SVR per src/init.sh: clone the repository, compile libsvm-3.23 and samtools-1.9 with 'make all', download the hg19 ensGene ANNOVAR database and SPIDEX hg19 database into annovar/humandb, and index the hg19 genome with samtools faidx. Extract the 42 splicing-related features (MaxEntScan splice-site, conservation, ESE-based/SilVA-derived, regulatory-element, and mRNA-structure features) for each variant, after the built-in VCF filter removes intergenic variants or variants more than 200 bp from splice sites using ANNOVAR ensGene (hg19). Run ε-SVR with LIBSVM's RBF kernel, grid-searching C and γ under 10-fold cross-validation in which folds are grouped by gene to avoid within-gene leakage. Apply greedy forward feature selection: at each round add the single remaining feature that maximizes the 10-fold CV PCC, stopping when performance peaks at 8 selected features (PCC 0.570); note that further additions decrease performance. For the final server model, drop the CADD-derived priPhyloP and minDistTSS features to get a 6-feature model (CV PCC 0.590, blind test PCC 0.577 on TS1098), demonstrating the greedy 8-feature choice is not globally optimal; if wild-type Ψ is unavailable, run PredPSI-SVR-noPSI (./main.sh without '-p'), which yields CV PCC 0.525 and blind PCC 0.479. Judge correctness by comparing reproduced PCCs to these reported values within numerical tolerance.

## Related tools

- **PredPSI-SVR** (End-to-end pipeline: VCF filtering, feature extraction, SVR training/prediction, and ΔΨ output; invoked via main.sh with optional '-p' Ψ file) — https://github.com/chenkenbio/PredPSI-SVR
- **libsvm** (ε-SVR implementation with RBF kernel used for training, grid search of C/γ, and 10-fold cross-validation; compiled from source (libsvm-3.23) in PredPSI-SVR/tools) — https://www.csie.ntu.edu.tw/~cjlin/libsvm
- **ANNOVAR** (Annotates the input VCF with ensGene (hg19) to filter intergenic/distant variants and extract splicing-related features) — http://annovar.openbioinformatics.org/en/latest/user-guide/download/
- **samtools** (Index the hg19 genome FASTA (samtools faidx) for feature extraction from flanking sequences; built from source (samtools-1.9)) — http://www.htslib.org/download/
- **SilVA** (Source of ESE-based splicing-enhancer features; PredPSI-SVR/tools/ese3/ese3_mod.py is modified from the SilVA package) — https://github.com/buske/silva

## Examples

```
cd ~/PredPSI-SVR/ && ./main.sh example/sample.vcf -p example/sample.psi -o example/outdir
```

## Evaluation signals

- The 8-feature greedy-selected ε-SVR model reproduces the reported 10-fold CV PCC of 0.570 on TR957 within numerical tolerance, with PCC decreasing when further features are added
- The 6-feature final server model (priPhyloP and minDistTSS removed) reproduces CV PCC 0.590 on the combined TR957+TS1098 training data and blind-test PCC 0.577 on TS1098, exceeding the 8-feature model
- PredPSI-SVR-noPSI reproduces 10-fold CV PCC 0.525 on TR957 and independent-test PCC 0.479 on TS1098
- Cross-validation folds are grouped by gene so that variants from the same gene never appear in both train and validation folds (no within-gene leakage)
- OUTPUT.psi contains fewer variants than the input VCF when the 200 bp splice-site/intergenic filter removes entries — an expected, not erroneous, discrepancy

## Limitations

- Requires wild-type Ψ of exons as input; without experimental Ψ, PCCs on the training and test sets drop by about 0.1 (hence the noPSI variant: 0.525/0.479)
- The small number of vex-seq samples prevented using more powerful algorithms like deep learning, and branch point sequence and pre-mRNA secondary structure features are not effectively included
- Tissue-specific Ψ is not incorporated; currently only Human BodyMap Project 2.0 RNA-seq data was used, and many exons cannot be found from MISO analysis due to sequencing depth
- Greedy forward feature selection's 8-feature choice is not globally optimal — removing CADD-derived priPhyloP and minDistTSS yields better CV (0.590) and blind-test (0.577) PCCs

## Evidence
