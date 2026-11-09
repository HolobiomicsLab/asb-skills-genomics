---
name: dna-methylation-motif-typing-modeling
description: Use when building or reproducing a model that classifies methylation
  type (4mC/5mC/6mA) and methylated base positions for bacterial motifs from nanopore
  signal features, e.g. LOOCV across REBASE-annotated motifs.
when_to_use_negative:
- Data has already been converted to bedMethyl summary tables via modkit pileup —
  Nanoident needs the five raw-signal/basecall features per motif, not per-position
  modification counts
- Input is standard nanodisco output limited to mean features only — the original
  nanodisco pipeline cannot supply the four additional features (std, time, proportion
  of reference bases, average quality score) without running update.sh
- Goal is metagenomic binning by methylation pattern rather than motif typing/fine-mapping
  — nanodisco's methylation-profile clustering covers that, not this model
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0749
tools:
- name: Nanoident
  role: Multi-column CNN for typing and fine mapping of bacterial methylation motifs;
    model classes wired into ./model/train_test.py and trained on pkl-format feature
    data
  repo: https://github.com/cz-csu/Nanoident
- name: nanodisco
  role: Baseline method retrained under the same LOOCV protocol, and source of the
    preprocessing container (Singularity v3.2.1+) for nanopore signal feature extraction
  repo: https://github.com/fanglab/nanodisco
- name: bam-readcount
  role: Installed alongside nanodisco to obtain basecall features for motif sequences
- name: conda
  role: Builds the Python training environment directly from Nanoident/env/freeze.yml
- name: Singularity
  role: Builds the R/nanodisco virtual environment (v3.2.1 and above) for preprocessing
    and baseline comparison
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:68fb83723a7b00d38cea184d74528f0b6f4f3bf005c29bef012290a48237eacb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cz-csu/Nanoident
  tools:
  - Nanoident
  - R
  - Rscript
  - Singularity
  - bam-readcount
  - conda
  - nanodisco
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# dna-methylation-motif-typing-modeling

## Summary

Model and evaluate bacterial DNA methylation motif typing/fine-mapping from nanopore sequencing using Nanoident, a multi-scale multi-column CNN that consumes five signal- and basecall-derived features per motif sequence.

## When to use

Use when building or reproducing a model that classifies methylation type (4mC/5mC/6mA) and methylated base positions for bacterial motifs from nanopore signal features, e.g. LOOCV across REBASE-annotated motifs.

## When NOT to use

- Data has already been converted to bedMethyl summary tables via modkit pileup — Nanoident needs the five raw-signal/basecall features per motif, not per-position modification counts
- Input is standard nanodisco output limited to mean features only — the original nanodisco pipeline cannot supply the four additional features (std, time, proportion of reference bases, average quality score) without running update.sh
- Goal is metagenomic binning by methylation pattern rather than motif typing/fine-mapping — nanodisco's methylation-profile clustering covers that, not this model

## Inputs

- pkl-format training features converted from R (data_from_R_to_python.ipynb) covering 7 bacteria and 46 methylation motifs (7 4mC, 11 5mC, 28 6mA)
- Nanopore raw signal preprocessed files (WGA and NAT samples, chunk-merged) from nanodisco-style preprocessing
- Reference genome FASTA files (NCBI CP041693, CP041696, NC_008261.1, CP014225.1, CP023448.1, NC_007796.1, NC_002946.2, CP041695, CP003732)
- REBASE methylation motif annotations
- best_model/Nanoident.pt pretrained model

## Outputs

- Trained multi-scale CNN model (~14,319 trainable parameters) wired into ./model/train_test.py
- Per-motif and average LOOCV accuracy tables for Nanoident vs retrained nanodisco
- Methylation type (4mC/5mC/6mA) predictions and methylated base positions per motif
- Independent-test-set accuracies over 12 motifs from two held-out bacteria

## How to apply

Extract five features per motif sequence from nanopore raw signals and basecalls (mean, std, time, proportion of reference bases, average quality score, plus base feature for positional correction) using the nanodisco preprocessing extended by update.sh, bam-readcount, and the Nanoident_preprocess/enrich/readcount shell scripts. Convert R outputs to pkl via data_from_R_to_python.ipynb, then train the multi-column CNN (Large column: single DSC conv kernel 4; Medium: DSC convs kernels 3 and 2; Small: three stacked kernel-2 DSC blocks) by concatenating column outputs, applying a learned weight block that adaptively weights the multi-scale feature channels, fusing with a 1x1 convolution, and adding residual connections around the conv blocks. Evaluate typing accuracy by LOOCV: hold out one of the 46 training motifs (7 4mC, 11 5mC, 28 6mA from 7 bacteria), train on the remaining 45 with train_loocv_model.py, test on the held-out motif with train_loocv_test.py, and compare against a retrained nanodisco baseline run under the same protocol in the Singularity-based R environment. The architecture totals 14,319 trainable parameters and the expected result is 83.68% average LOOCV accuracy versus nanodisco's 73.93%, with ~13% improvement on the 12-motif independent test set (6 4mC, 1 5mC, 5 6mA).

## Related tools

- **Nanoident** (Multi-column CNN for typing and fine mapping of bacterial methylation motifs; model classes wired into ./model/train_test.py and trained on pkl-format feature data) — https://github.com/cz-csu/Nanoident
- **nanodisco** (Baseline method retrained under the same LOOCV protocol, and source of the preprocessing container (Singularity v3.2.1+) for nanopore signal feature extraction) — https://github.com/fanglab/nanodisco
- **bam-readcount** (Installed alongside nanodisco to obtain basecall features for motif sequences)
- **conda** (Builds the Python training environment directly from Nanoident/env/freeze.yml)
- **Singularity** (Builds the R/nanodisco virtual environment (v3.2.1 and above) for preprocessing and baseline comparison)

## Examples

```
conda env create -f freeze.yml && python ./model/train_test.py
```

## Evaluation signals

- Model instantiation succeeds and total trainable parameter count equals 14,319
- Average LOOCV accuracy over the 46 motifs reproduces 83.68% and exceeds retrained nanodisco's 73.93%
- Independent 12-motif test set shows ~13% accuracy improvement over the previous method
- pkl conversion via data_from_R_to_python.ipynb loads without error in the Python environment created from Nanoident/env/freeze.yml
- Each of the 46 LOOCV folds (train_loocv_model.py / train_loocv_test.py) runs without error and yields a fold-level typing accuracy

## Limitations

- nanodisco preprocessing originally handled only mean features; update.sh must be run to add std, time, proportion-of-reference-bases, and average-quality-score features plus the base feature for positional correction
- Because of memory constraints, preprocessing must be done in chunks first and then merged
- Two virtual environments must be maintained (R/nanodisco under Singularity, Python under conda), with data exchanged via pkl conversion

## Evidence

- [methods] LOOCV accuracy and architecture: "Use Nanoident/env/freeze.yml to generate the environment directly under conda."
- [readme] Feature extension: "The original nanodisco could only handle mean features, but Nanoident can handle four additional features"
- [abstract] LOOCV result: "The Leave-One-Out Cross-Validation (LOOCV) on the dataset containing 7 bacteria samples with 46 methylation motifs shows that Nanoident achieves ∼10% improvement in accuracy than the previous method."
- [supplementary] Motif composition: "There are 46 different motifs in these seven bacteria, including 7 4mC motifs, 11 5mC motifs, and 28 6mA motifs."
- [readme] Conversion step: "convert it to a pkl file that can be used in a python environment using data_from_R_to_python.ipynb"
- [supplementary] Data source: "All data used in this work are available at the Sequence Read Archive under the BioProjects PRJNA559199"
