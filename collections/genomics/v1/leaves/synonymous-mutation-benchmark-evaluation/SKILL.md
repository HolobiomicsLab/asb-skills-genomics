---
name: synonymous-mutation-benchmark-evaluation
description: Use when you have a set of pathogenic vs. benign synonymous mutations
  (VCF) and need to rank splicing-prediction tools by their ability to separate the
  classes via AUC on the SynonMut benchmark.
when_to_use_negative:
- The task is predicting continuous ΔΨ values for the vex-seq challenge (regression/PCC
  evaluation) rather than classifying synonymous mutations as splicing-affecting vs.
  not
- Variants are non-synonymous, intergenic, or lie more than 200 bp from splice sites
  — the PredPSI-SVR VCF filter will remove them before scoring
- The benchmarking question concerns deep-learning ΔΨ predictors like MMSplice or
  SpliceAI rather than the SVR/SPANR/MaxEntScan trio evaluated here
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0199
tools:
- name: PredPSI-SVR
  role: SVR-based scorer of |ΔΨ| for synonymous variants; run with -p (with wild-type
    Ψ) and without (-noPSI)
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: SPANR
  role: Competing baseline providing SPIDEX ΔΨ scores for AUC comparison
- name: MaxEntScan
  role: Competing baseline scored by summed absolute change |ΔMES5|+|ΔMES3|; limited
    scanning scope defines the out-of-scope subset
- name: ANNOVAR
  role: Annotates the filtered VCF with hg19 ensGene splice-site proximity used for
    VCF filtering
- name: samtools
  role: Indexes the hg19 genome (faidx) used for sequence retrieval during scoring
- name: libsvm
  role: Provides the support vector regression backend for PredPSI-SVR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:294f061c0f1d1769ffb5716d1ce0ff35b406803804e80a36838f144cf46a89c6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - ANNOVAR
  - MaxEntScan
  - Perl
  - PredPSI-SVR
  - SPANR
  - libsvm
  - samtools
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# synonymous-mutation-benchmark-evaluation

## Summary

Benchmarks how well splicing-impact scores (PredPSI-SVR |ΔΨ|, SPANR SPIDEX ΔΨ, MaxEntScan |ΔMES5|+|ΔMES3|) prioritize pathogenic synonymous mutations on the SynonMut datasets by computing and statistically comparing AUCs.

## When to use

Use when you have a set of pathogenic vs. benign synonymous mutations (VCF) and need to rank splicing-prediction tools by their ability to separate the classes via AUC on the SynonMut benchmark.

## When NOT to use

- The task is predicting continuous ΔΨ values for the vex-seq challenge (regression/PCC evaluation) rather than classifying synonymous mutations as splicing-affecting vs. not
- Variants are non-synonymous, intergenic, or lie more than 200 bp from splice sites — the PredPSI-SVR VCF filter will remove them before scoring
- The benchmarking question concerns deep-learning ΔΨ predictors like MMSplice or SpliceAI rather than the SVR/SPANR/MaxEntScan trio evaluated here

## Inputs

- VCF of synonymous mutations (SynonMut datasets)
- Experimental wild-type Ψ file for exons (optional, for the -p option)
- PredPSI-SVR environment: ANNOVAR humandb with hg19 ensGene, SPIDEX database, samtools-indexed hg19 genome

## Outputs

- Per-variant splicing scores: PredPSI-SVR predicted |ΔΨ| (OUTPUT.dpsi), SPANR SPIDEX ΔΨ, MaxEntScan |ΔMES5|+|ΔMES3|
- AUC per method per dataset subset (SynonMut-psi, SynonMut-complete, MaxEntScan out-of-scope subset)
- P-values from pairwise AUC significance comparisons

## How to apply

Prepare the PredPSI-SVR environment (Perl, Python 3 + numpy, ANNOVAR hg19 ensGene in humandb, SPIDEX, samtools-indexed hg19) and check variables in src/init.sh. Filter the input synonymous-mutation VCF to drop intergenic variants and variants farther than 200 bp from splice sites, then score with PredPSI-SVR twice: with the -p option supplying experimental wild-type Ψ (SynonMut-psi setting) and without it (PredPSI-SVR-noPSI) on both Ψ-matched and complete variant sets. Score the same variants with SPANR (SPIDEX ΔΨ) and MaxEntScan using the summed absolute change |ΔMES5|+|ΔMES3|, compute AUCs for each score, and compare with significance tests (e.g. PredPSI-SVR 0.579 vs SPANR P=0.036; noPSI 0.575 on SynonMut-complete, P=0.004 vs SPANR and P=0.049 vs MaxEntScan). Also evaluate the 11024 mutations outside MaxEntScan's scanning scope, where PredPSI-SVR-noPSI reaches 0.536 versus SPANR's 0.501 (P=0.018).

## Related tools

- **PredPSI-SVR** (SVR-based scorer of |ΔΨ| for synonymous variants; run with -p (with wild-type Ψ) and without (-noPSI)) — https://github.com/chenkenbio/PredPSI-SVR
- **SPANR** (Competing baseline providing SPIDEX ΔΨ scores for AUC comparison)
- **MaxEntScan** (Competing baseline scored by summed absolute change |ΔMES5|+|ΔMES3|; limited scanning scope defines the out-of-scope subset)
- **ANNOVAR** (Annotates the filtered VCF with hg19 ensGene splice-site proximity used for VCF filtering)
- **samtools** (Indexes the hg19 genome (faidx) used for sequence retrieval during scoring)
- **libsvm** (Provides the support vector regression backend for PredPSI-SVR)

## Examples

```
./main.sh example/sample.vcf -p example/sample.psi -o example/outdir
```

## Evaluation signals

- Reproduced PredPSI-SVR AUC on SynonMut-psi equals 0.579 (significantly better than SPANR, P=0.036)
- PredPSI-SVR-noPSI AUC equals 0.508 on SynonMut-psi and 0.575 on SynonMut-complete (P=0.004 vs SPANR, P=0.049 vs MaxEntScan)
- On the 11024 mutations outside MaxEntScan's scanning scope, PredPSI-SVR-noPSI AUC is 0.536 versus 0.501 for SPANR (P=0.018)
- OUTPUT.psi contains fewer variants than the input VCF when the >200 bp splice-site filter removes distant/intergenic variants — verify the filtered subset is intentional
- Each method's AUC exceeds the 0.5 random baseline on its scored subsets, except noPSI's near-chance 0.508 on SynonMut-psi

## Limitations

- Without experimental wild-type Ψ (no -p option), performance drops: AUC falls from 0.579 to 0.508 on SynonMut-psi, and the paper reports PCCs dropping by about 0.1 without experimental Ψ
- MaxEntScan cannot score all variants — 11024 mutations fall outside its scanning scope and require a separate subset analysis
- PredPSI-SVR lacks branch point sequence and pre-mRNA secondary structure features due to limited samples, and small training data prevented deep learning approaches
- Only Human BodyMap Project 2.0 RNA-seq data was used; many exons cannot be analyzed due to sequencing depth, and tissue-specific Ψ is not incorporated

## Evidence

- [results] PredPSI-SVR achieved an AUC of 0.579 on SynonMut-psi, significantly better than SPANR (P-value=0.036) and 6.6% higher than MaxEntScan: [evidence span withheld — anchor status: not_found]
- [methods] Score the filtered variants with PredPSI-SVR using the -p option to supply the experimental wild-type Ψ file (SynonMut-psi setting).: [evidence span withheld — anchor status: not_found]
