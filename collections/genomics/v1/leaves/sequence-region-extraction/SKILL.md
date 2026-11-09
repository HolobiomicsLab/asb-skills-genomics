---
name: sequence-region-extraction
description: Use when constructing splice-site training/testing datasets where each
  example must be a fixed 400-nt sequence window around a true or false acceptor or
  donor site before one-hot encoding.
when_to_use_negative:
- The downstream model or task requires a sequence window length other than 400 nt,
  since the CNNSplice constructed dataset permits a Sequence Length of 400
- The input is already a fixed-length, one-hot-encoded sequence matrix ready for CNN
  training rather than raw splice-site annotations on a genome
- The goal is splice-site model training/testing rather than dataset construction
  (use train.py/test.py directly instead)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: CNNSplice
  role: Defines the fixed 400-nt sequence-length constraint and consumes the extracted,
    one-hot-encoded 400-nt regions as training/testing input for splice-site prediction
  repo: https://github.com/OluwadareLab/CNNSplice.git
- name: Splice2Deep
  role: Source of the cross-organism splice-site datasets (Albaradei et al.) from
    which the 400-nt regions are extracted
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.031
    title: 'CNNSplice: Robust models for splice site prediction using convolutional
      neural networks'
  source_content_hash: sha256:7330915f8aae4e2dde379ed12c674726554da9df85adca0aa485d36d6b1dd6de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OluwadareLab/CNNSplice.git
  tools:
  - CNNSplice
  - Splice2Deep
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# sequence-region-extraction

## Summary

Extracts fixed-length 400-nt genomic sequence regions centered on splice sites (acceptor/donor) so that every example conforms to the Sequence Length of 400 constraint imposed by CNNSplice's CNN models.

## When to use

Use when constructing splice-site training/testing datasets where each example must be a fixed 400-nt sequence window around a true or false acceptor or donor site before one-hot encoding.

## When NOT to use

- The downstream model or task requires a sequence window length other than 400 nt, since the CNNSplice constructed dataset permits a Sequence Length of 400
- The input is already a fixed-length, one-hot-encoded sequence matrix ready for CNN training rather than raw splice-site annotations on a genome
- The goal is splice-site model training/testing rather than dataset construction (use train.py/test.py directly instead)

## Inputs

- Splice-site datasets for Homo sapiens, Oryza sativa japonica, Arabidopsis thaliana, Drosophila melanogaster, and Caenorhabditis elegans (as used by Albaradei et al. 2020)
- True and false acceptor and donor splice-site annotations (genomic coordinates)
- Genome/reference sequences for each organism

## Outputs

- Balanced acceptor/donor sets of 5000 true and 5000 false instances per organism
- Imbalanced acceptor/donor sets of 7500 true and 2500 false instances per organism (3:1 ratio)
- 400-nt sequence regions (positions 0-399) centered on each splice site
- Summary table mirroring Table 1 (dataset_table1.csv) of per-organism balanced and imbalanced counts

## How to apply

For each organism dataset (Homo sapiens, Oryza sativa japonica, Arabidopsis thaliana, Drosophila melanogaster, Caenorhabditis elegans), sample true/false splice-site examples to form balanced sets (5000 true and 5000 false acceptor and donor entries) and imbalanced sets (7500 true and 2500 false, a 3:1 ratio). For every example, extract the 400-nt region spanning positions 0-399 centered on the splice site, respecting the fixed sequence-length constraint that CNNSplice's constructed dataset permits (Sequence Length of 400). Assemble the per-organism counts into a summary table mirroring Table 1 (e.g. dataset_table1.csv) and apply one-hot encoding to the genomic sequences before model input, as required by the CNNSplice pipeline.

## Related tools

- **CNNSplice** (Defines the fixed 400-nt sequence-length constraint and consumes the extracted, one-hot-encoded 400-nt regions as training/testing input for splice-site prediction) — https://github.com/OluwadareLab/CNNSplice.git
- **Splice2Deep** (Source of the cross-organism splice-site datasets (Albaradei et al.) from which the 400-nt regions are extracted)

## Examples

```
python train.py -n "output_name" -m "balanced"
```

## Evaluation signals

- Each organism's balanced set contains exactly 5000 true and 5000 false acceptor and donor entries (1:1 ratio)
- Each organism's imbalanced set contains exactly 7500 true and 2500 false acceptor and donor entries (3:1 ratio)
- All extracted sequence regions are exactly 400 nt in length, satisfying the dataset's Sequence Length of 400 constraint
- The assembled summary table reproduces the composition reported in Table 1 across all five organisms
- Extracted sequences can be transformed via one-hot encoding without dimension errors before feeding train.py

## Limitations

- The dataset construction is bound to the fixed Sequence Length of 400; other window sizes are not supported by the CNNSplice pipeline
- Only the five organism datasets used by Albaradei et al. (2020) are covered; new organisms would require equivalent sampling and windowing
- Genomic sequence input data must be transformed using one-hot encoding before model use; skipping this step breaks the workflow

## Evidence

- [results] we constructed 5000 true and 5000 false acceptor and donor datasets for the balanced experiments and 7500 true and 2500 false acceptor and donor datasets for the imbalanced: "we constructed 5000 true and 5000 false acceptor and donor datasets for the balanced experiments and 7500 true and 2500 false acceptor and donor datasets for the imbalanced"
- [readme] Our constructed dataset permits a **Sequence Length of 400**: "Our constructed dataset permits a **Sequence Length of 400**"
- [readme] Genomic sequence input data should be transfomed using one-hot encoding.: "Genomic sequence input data should be transfomed using one-hot encoding."
- [results] the five organisms' datasets: Homo sapiens, Oryza sativa japonica, Arabidopsis thaliana, Drosophila melanogaster, and Caenorhabditis elegans organisms: [evidence span withheld — anchor status: not_found]
