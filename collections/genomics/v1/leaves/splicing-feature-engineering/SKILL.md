---
name: splicing-feature-engineering
description: Use when building or refining predictive models of variant-induced splicing
  change (delta-PSI) from genomic sequence and annotation, and when deciding which
  features contribute most to prediction.
when_to_use_negative:
- Input variants are intergenic or more than 200 bp from splice sites — PredPSI-SVR's
  VCF filter will remove them, so feature extraction is pointless
- Sample size is large enough for deep learning approaches (the paper notes the small
  sample count prevented deep learning, but MMSplice-style neural methods are preferable
  at scale)
- Experimental wild-type Psi values are unavailable and even the ~0.1 PCC drop of
  PredPSI-SVR-noPSI is unacceptable for the application
tools:
- name: PredPSI-SVR
  role: Main pipeline that extracts the 42 splicing-related features from exon sequences
    and flanking regions and runs the SVR prediction
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: libsvm
  role: Support vector regression engine used to train and evaluate the feature-selected
    model
  repo: https://www.csie.ntu.edu.tw/~cjlin/libsvm
- name: ANNOVAR
  role: Annotates variants with ensGene (hg19) gene models and the SPIDEX third-party
    database used for feature computation
- name: MaxEntScan
  role: Computes splice site strength features; splicing site features from MaxEntScan
    have the highest correlations with delta-PSI
- name: samtools
  role: Indexes the hg19 FASTA reference (samtools faidx) for sequence retrieval during
    feature extraction
- name: SilVA
  role: Source of the splicing enhancer/suppressor feature script (ese3_mod.py in
    PredPSI-SVR/tools/ese3 is modified from SilVA)
  repo: https://github.com/buske/silva
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:b08bfdb55b3721d31ee0769be18ea0e752e4512f2af9e71144aefd69192a37a7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - ANNOVAR
  - MaxEntScan
  - PredPSI-SVR
  - SilVA
  - libsvm
  - samtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# splicing-feature-engineering

## Summary

Engineer 42 comprehensive splicing-related features (sequence, conservation, splice site, regulatory element, and mRNA secondary structure) from an exon and its flanking regions, then apply greedy feature selection to build SVR models for delta-PSI prediction. This skill captures the feature-extraction and subset-selection workflow behind PredPSI-SVR.

## When to use

Use when building or refining predictive models of variant-induced splicing change (delta-PSI) from genomic sequence and annotation, and when deciding which features contribute most to prediction.

## When NOT to use

- Input variants are intergenic or more than 200 bp from splice sites — PredPSI-SVR's VCF filter will remove them, so feature extraction is pointless
- Sample size is large enough for deep learning approaches (the paper notes the small sample count prevented deep learning, but MMSplice-style neural methods are preferable at scale)
- Experimental wild-type Psi values are unavailable and even the ~0.1 PCC drop of PredPSI-SVR-noPSI is unacceptable for the application

## Inputs

- VCF file of variants near splice sites (PredPSI-SVR filters variants in intergenic regions or more than 200 bp from splice sites)
- Wild-type PSI (percent spliced in) values per exon (optional -p input; performance drops about 0.1 PCC without experimental Psi)
- hg19 reference genome FASTA (chromFa, indexed with samtools faidx)
- ANNOVAR ensGene hg19 annotation database and SPIDEX database (hg19_spidex.zip)
- vex-seq challenge training dataset (5th Critical Assessment of Genome Interpretation) with measured delta-PSI values

## Outputs

- Feature matrix of 42 splicing-related features per variant (or a greedy-selected subset, e.g. 8 or 6 features)
- Trained libsvm support vector regression model for delta-PSI
- Predicted delta-PSI values (OUTPUT.dpsi)
- Pearson correlation coefficients from 10-fold CV and independent test (e.g. pcc_results.csv)

## How to apply

Extract 42 splicing-related features from the sequence of each exon and its flanking regions, including splice site scores (MaxEntScan-based features were found to have the highest correlations), conservation, regulatory elements (e.g. ESE features adapted from SilVA via ese3_mod.py), and mRNA secondary structure. Annotate variants with ANNOVAR using the hg19 ensGene database and the third-party SPIDEX database, with the hg19 genome indexed by samtools faidx. Apply a greedy feature selection algorithm to identify the features contributing most to prediction; in the paper this yielded an 8-feature model. Test alternative subsets — e.g. removing the CADD-derived features priPhyloP (conservation) and minDistTSS (distance to TSS) — and compare 10-fold cross-validation PCC and independent-test PCC on the vex-seq training data (TR957+TS1098) to confirm the subset is not globally optimal (the 6-feature server model reached PCC 0.590 CV / 0.577 blind test). Train support vector regression with libsvm compiled from source and evaluate by Pearson correlation coefficient between predicted and measured delta-PSI.

## Related tools

- **PredPSI-SVR** (Main pipeline that extracts the 42 splicing-related features from exon sequences and flanking regions and runs the SVR prediction) — https://github.com/chenkenbio/PredPSI-SVR
- **libsvm** (Support vector regression engine used to train and evaluate the feature-selected model) — https://www.csie.ntu.edu.tw/~cjlin/libsvm
- **ANNOVAR** (Annotates variants with ensGene (hg19) gene models and the SPIDEX third-party database used for feature computation)
- **MaxEntScan** (Computes splice site strength features; splicing site features from MaxEntScan have the highest correlations with delta-PSI)
- **samtools** (Indexes the hg19 FASTA reference (samtools faidx) for sequence retrieval during feature extraction)
- **SilVA** (Source of the splicing enhancer/suppressor feature script (ese3_mod.py in PredPSI-SVR/tools/ese3 is modified from SilVA)) — https://github.com/buske/silva

## Examples

```
./main.sh example/sample.vcf -p example/sample.psi -o example/outdir
```

## Evaluation signals

- 10-fold cross-validation PCC on the combined TR957+TS1098 vex-seq training set should reproduce the reported values (0.570 for the challenge model; 0.590 for the 6-feature server model) within reasonable tolerance
- Independent blind-test PCC (TS1098) should match the reported 0.566 (challenge) or 0.577 (6-feature server model)
- Feature-removal check: dropping priPhyloP and minDistTSS from the 8-feature CADD-based set should increase PCC on both CV and independent tests, demonstrating greedy selection is not globally optimal
- Feature count sanity check: full extraction yields 42 features; greedy selection yields 8 features; the final server model uses 6 features
- Number of variants in OUTPUT.psi may be smaller than the input VCF because variants in intergenic regions or more than 200 bp from splice sites are filtered out

## Limitations

- PredPSI-SVR does not include branch point sequence features or pre-mRNA secondary structure features effectively due to the limited number of samples
- The small number of samples prevented using more powerful algorithms like deep learning
- The method requires wild-type Psi of exons as input; without experimental Psi, PCCs on the training and test sets dropped by about 0.1
- Only Human BodyMap Project 2.0 RNA-seq data were used, and many exons cannot be found in the MISO analysis due to sequencing depth; tissue-specific Psi is not incorporated
- Greedy feature selection found 8 top features, but the greedy choice is not globally optimal — the 6-feature subset excluding priPhyloP and minDistTSS performed better

## Evidence
