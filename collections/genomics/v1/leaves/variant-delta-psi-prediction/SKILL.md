---
name: variant-delta-psi-prediction
description: Use when you have VCF variant calls near splice sites and want to predict
  delta-PSI for exon skipping events, especially for CAGI vex-seq-style benchmarks
  or prioritizing splicing-disrupting variants.
when_to_use_negative:
- Variants in intergenic regions or more than 200 bp away from splice sites — the
  pipeline filters these out, so predictions will not be produced
- No experimental wild-type PSI is available — PCCs drop by about 0.1 when running
  without the '-p' option
- Branch point sequence or pre-mRNA secondary structure effects are the target — PredPSI-SVR
  does not include these features due to limited sample size
edam_operation: http://edamontology.org/operation_0433
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_3511
tools:
- name: PredPSI-SVR
  role: 'Main pipeline: VCF filtering, feature extraction from exon and flanking sequences,
    and SVR-based delta-PSI prediction'
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: libsvm
  role: Support vector regression engine used to predict delta-PSI from the selected
    splicing features
  repo: https://www.csie.ntu.edu.tw/~cjlin/libsvm
- name: MaxEntScan
  role: Computes splice-site score features, which have the highest correlations among
    features
- name: ANNOVAR
  role: Provides ensGene hg19 annotation and SPIDEX database integration for variant
    annotation
- name: samtools
  role: Indexes the hg19 reference genome FASTA (faidx) for sequence extraction
  repo: http://www.htslib.org/download/
- name: SPIDEX
  role: Third-party database (SPIDEX delta-PSI from SPANR) used as a feature source
    via ANNOVAR
- name: SilVA
  role: PredPSI-SVR's ese3 ESE-motif feature script (ese3_mod.py) is modified from
    a SilVA script
  repo: https://github.com/buske/silva
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:320792dfe78a021e208f9d0133b944f5975e66e0b9522676417694c84f1d10ba
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
  - SilVA
  - libsvm
  - samtools
  edam_operation: http://edamontology.org/operation_0433
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# variant-delta-psi-prediction

## Summary

Predict the change of percent spliced in (delta-PSI) caused by genetic variants using PredPSI-SVR, a support vector regression model trained on the CAGI 5 vex-seq dataset with 42 splicing-related features (8 greedily selected for the final model).

## When to use

Use when you have VCF variant calls near splice sites and want to predict delta-PSI for exon skipping events, especially for CAGI vex-seq-style benchmarks or prioritizing splicing-disrupting variants.

## When NOT to use

- Variants in intergenic regions or more than 200 bp away from splice sites — the pipeline filters these out, so predictions will not be produced
- No experimental wild-type PSI is available — PCCs drop by about 0.1 when running without the '-p' option
- Branch point sequence or pre-mRNA secondary structure effects are the target — PredPSI-SVR does not include these features due to limited sample size

## Inputs

- VCF file with variant calls (single-nucleotide or small variants near splice sites)
- Optional wild-type PSI values per exon (plain text file supplied via '-p' option)
- hg19 reference genome FASTA (chromFa) with samtools faidx index
- ANNOVAR ensGene annotation (hg19) and SPIDEX database in humandb

## Outputs

- OUTPUT.dpsi with predicted delta-PSI per variant
- OUTPUT.psi with filtered per-variant PSI predictions (fewer entries than input VCF due to filtering)

## How to apply

Set up the PredPSI-SVR standalone environment: clone the repo, place ANNOVAR, libsvm (built with make), and samtools in PredPSI-SVR/tools, download the hg19 ensGene annotation and SPIDEX database into humandb, and prepare the hg19 genome (chromFa, faidx-indexed). Configure paths in src/init.sh, ensuring Perl, Python 2, and Python 3 with numpy are available. Run ./main.sh on the input VCF with the optional '-p' flag supplying a wild-type PSI file; the pipeline first filters the VCF to remove intergenic variants and variants more than 200 bp from splice sites, then computes eight greedily-selected splicing features (MaxEntScan splice-site scores, conservation, ESE motifs via ese3, SPIDEX-derived values), and applies the libsvm SVR model to output predicted delta-PSI to OUTPUT.dpsi. Performance is judged by Pearson correlation coefficient between predicted and measured delta-PSI: the trained model achieved PCC 0.570 in 10-fold cross-validation and 0.566 (rank 2) on the vex-seq blind test; later work reproduced a 6-feature server model with CV PCC 0.590 and blind PCC 0.577 by removing priPhyloP and minDistTSS from the 8-feature set.

## Related tools

- **PredPSI-SVR** (Main pipeline: VCF filtering, feature extraction from exon and flanking sequences, and SVR-based delta-PSI prediction) — https://github.com/chenkenbio/PredPSI-SVR
- **libsvm** (Support vector regression engine used to predict delta-PSI from the selected splicing features) — https://www.csie.ntu.edu.tw/~cjlin/libsvm
- **MaxEntScan** (Computes splice-site score features, which have the highest correlations among features)
- **ANNOVAR** (Provides ensGene hg19 annotation and SPIDEX database integration for variant annotation)
- **samtools** (Indexes the hg19 reference genome FASTA (faidx) for sequence extraction) — http://www.htslib.org/download/
- **SPIDEX** (Third-party database (SPIDEX delta-PSI from SPANR) used as a feature source via ANNOVAR)
- **SilVA** (PredPSI-SVR's ese3 ESE-motif feature script (ese3_mod.py) is modified from a SilVA script) — https://github.com/buske/silva

## Examples

```
./main.sh example/sample.vcf -p example/sample.psi -o example/outdir
```

## Evaluation signals

- 10-fold cross-validation PCC on the combined vex-seq training set should match the reported 0.570 (or 0.590 for the 6-feature server model) within reasonable tolerance
- Blind-test PCC on TS1098 (variants on chromosomes 9 to X) should reproduce the reported 0.566 (8-feature model) or 0.577 (6-feature model)
- OUTPUT.psi should contain fewer variants than the input VCF, consistent with removal of intergenic variants and variants >200 bp from splice sites
- Running with '-p' (wild-type PSI supplied) should yield higher PCC than without; the paper reports ~0.1 PCC drop without experimental PSI

## Limitations

- Requires wild-type PSI of exons as input; without experimental PSI, PCCs on training and test sets drop by about 0.1
- Does not include branch point sequence or pre-mRNA secondary structure features due to the small number of samples
- Small sample size prevented use of more powerful algorithms like deep learning
- Only Human BodyMap Project 2.0 RNA-seq data used for MISO analysis; many exons not captured due to sequencing depth; tissue-specific PSI not incorporated

## Evidence

- [methods] Download ANNOVAR, libsvm, and samtools and move them to PredPSI-SVR/tools: [evidence span withheld — anchor status: not_found]
