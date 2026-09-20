---
name: splicing-feature-computation-from-exon-sequence
description: Use when you have variant calls (VCF) near splice sites with hg19 exon
  annotations and need per-variant splicing feature vectors to feed a delta-PSI regression
  model such as PredPSI-SVR.
when_to_use_negative:
- Variants lie in intergenic regions or more than 200 bp from splice sites — the PredPSI-SVR
  pipeline filters these out before feature computation
- The exon of interest requires branch-point or pre-mRNA secondary-structure features
  — PredPSI-SVR does not model these effectively due to small sample size
edam_operation: http://edamontology.org/operation_0278
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0203
tools:
- name: PredPSI-SVR
  role: Standalone pipeline that orchestrates feature computation from exon/flanking
    sequence and applies the libsvm SVR model to the eight selected features
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: MaxEntScan
  role: Computes splice donor/acceptor site scores; these splice-site features showed
    the highest correlations in the study
- name: ANNOVAR
  role: Annotates variants against hg19 ensGene to identify exon/splice-site proximity
    and supply SPIDEX values
- name: SPIDEX
  role: Third-party database supplying SPANR delta-PSI values used as a feature
- name: SilVA (ese3)
  role: ese3_mod.py in PredPSI-SVR/tools is modified from SilVA to compute ESE motif
    features
  repo: https://github.com/buske/silva
- name: samtools
  role: Indexes hg19.fasta (faidx) so exon and flanking sequences can be retrieved
    for feature extraction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:076b4bd398464bb263260794fa0006383556a84a437bba1515378390104f5e99
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - ANNOVAR
  - MaxEntScan
  - Perl
  - PredPSI-SVR
  - SPIDEX
  - SilVA (ese3)
  - samtools
  edam_operation: http://edamontology.org/operation_0278
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# splicing-feature-computation-from-exon-sequence

## Summary

Computes splicing-related features from an exon and its flanking sequence for delta-PSI prediction, reducing 42 candidate features (MaxEntScan splice-site scores, conservation, ESE motifs via ese3, SPIDEX delta-PSI, etc.) to the eight features selected by greedy feature selection for the PredPSI-SVR support vector regression model.

## When to use

Use when you have variant calls (VCF) near splice sites with hg19 exon annotations and need per-variant splicing feature vectors to feed a delta-PSI regression model such as PredPSI-SVR.

## When NOT to use

- Variants lie in intergenic regions or more than 200 bp from splice sites — the PredPSI-SVR pipeline filters these out before feature computation
- The exon of interest requires branch-point or pre-mRNA secondary-structure features — PredPSI-SVR does not model these effectively due to small sample size

## Inputs

- VCF file of genetic variants near splice sites (hg19 coordinates)
- hg19 genome FASTA (UCSC goldenPath chromFa) with samtools .fai index
- ANNOVAR hg19 ensGene annotation database
- SPIDEX database (hg19_spidex) for SPANR-derived delta-PSI features
- Optional: wild-type Psi values of affected exons (for the '-p' PredPSI-SVR mode)

## Outputs

- Per-variant splicing feature matrix (eight greedily-selected features per variant)
- Predicted delta-PSI per variant (OUTPUT.dpsi)
- Per-variant Psi predictions for filtered variants (OUTPUT.psi)

## How to apply

Annotate the VCF with ANNOVAR (hg19 ensGene) to retain only variants within 200 bp of splice sites and drop intergenic ones, using samtools-faidx'd hg19.fasta to retrieve exon and flanking sequence. For each retained variant, compute the feature set: MaxEntScan splice donor/acceptor scores (reported as having the highest correlations), conservation scores, ESE motif features from the ese3 module (modified from the SilVA package), and SPIDEX SPIDEXdelta-PSI values. Apply the greedy feature selection outcome so only the eight most contributing features are carried forward, since the 8-feature model raised PCC from 0.516 to 0.566 on the blind test. Write the resulting feature matrix for input to the libsvm support vector regression, which outputs predicted delta-PSI to OUTPUT.dpsi and per-variant Psi predictions to OUTPUT.psi.

## Related tools

- **PredPSI-SVR** (Standalone pipeline that orchestrates feature computation from exon/flanking sequence and applies the libsvm SVR model to the eight selected features) — https://github.com/chenkenbio/PredPSI-SVR
- **MaxEntScan** (Computes splice donor/acceptor site scores; these splice-site features showed the highest correlations in the study)
- **ANNOVAR** (Annotates variants against hg19 ensGene to identify exon/splice-site proximity and supply SPIDEX values)
- **SPIDEX** (Third-party database supplying SPANR delta-PSI values used as a feature)
- **SilVA (ese3)** (ese3_mod.py in PredPSI-SVR/tools is modified from SilVA to compute ESE motif features) — https://github.com/buske/silva
- **samtools** (Indexes hg19.fasta (faidx) so exon and flanking sequences can be retrieved for feature extraction)

## Examples

```
cd ~/PredPSI-SVR/ && ./main.sh example/sample.vcf -p example/sample.psi -o example/outdir
```

## Evaluation signals

- Reproducing the CAGI vex-seq blind test (TS1098, 1098 variants on chromosomes 9 to X) yields Pearson correlation of 0.566 between predicted and experimental delta-PSI (rank-2 result)
- 10-fold cross-validation on the vex-seq training set yields PCC of 0.570
- OUTPUT.psi contains fewer variants than the input VCF, consistent with removal of intergenic variants and variants more than 200 bp from splice sites
- The feature vector for each variant contains exactly the eight greedily-selected features; the 42-feature full set should not be fed to the final SVR
- The 8-feature model improves PCC from 0.516 (fewer features) to 0.566 on the independent test set

## Limitations

- Without experimental wild-type Psi input, PCC drops by about 0.1 on both training and test sets
- Branch point sequence and pre-mRNA secondary structure features are not effectively included due to limited sample numbers
- The small number of training samples precluded more powerful algorithms such as deep learning
- Tissue-specific Psi is not incorporated, limiting discovery of pathogenic variants in specific diseases
- Only Human BodyMap Project 2.0 RNA-seq data were used; many exons cannot be found in the MISO analysis due to sequencing depth

## Evidence
