---
name: torchscript-model-export
description: Use when a trained PyTorch MLP from the k-mer size sweep (k=7-25) has
  been selected (k=21) and must be packaged as a deployable artifact consumed by z-calling-base
  with -k 21.
when_to_use_negative:
- Exporting the single-nucleotide A/Z calling model — that artifact is model/k11-mixed-AZ/scripted_m11.pth
  and must be invoked with -k 11, not -k 21
- No GPU-accelerated inference is needed — the tool is explicitly designed for CPU-only
  Linux environments
- The read-level dZ/canonical decision — that step uses the pre-trained SVM (model/svm/k21ReadClassifier),
  not the exported TorchScript module
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: Z-Calling
  role: Source repository containing the PyTorch MLP training code and the C++ z-calling-base
    executable that loads the exported TorchScript model with -k 21
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PyTorch
  role: Framework used to train the k-mer feature MLP and to script/serialize it as
    model/k21-full-ZA/scripted_m21.pth (torch 2.2.2+cpu)
  repo: https://github.com/xiaochuanle/Z-Calling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:87ed7a54df1ec6830d5f0e92c9af535d44ac35aaca7e4622427063c958e630ac
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xiaochuanle/Z-Calling
  repo_url: https://github.com/xiaochuanle/Z-Calling
  tools:
  - PyTorch
  - Z-Calling
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# torchscript-model-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Serializes the selected k-mer-specific PyTorch MLP (e.g., the k=21 full-substitution model trained on E. coli full-Z vs full-dA kinetic features) as a TorchScript artifact (scripted_m21.pth) that the C++ z-calling-base executable can load for CPU inference.

## When to use

Use when a trained PyTorch MLP from the k-mer size sweep (k=7-25) has been selected (k=21) and must be packaged as a deployable artifact consumed by z-calling-base with -k 21.

## When NOT to use

- Exporting the single-nucleotide A/Z calling model — that artifact is model/k11-mixed-AZ/scripted_m11.pth and must be invoked with -k 11, not -k 21
- No GPU-accelerated inference is needed — the tool is explicitly designed for CPU-only Linux environments
- The read-level dZ/canonical decision — that step uses the pre-trained SVM (model/svm/k21ReadClassifier), not the exported TorchScript module

## Inputs

- Trained PyTorch MLP classifier on PW/IPD k-mer kinetic features (k=21, full-Z vs full-dA E. coli training reads)
- Conda environment with torch 2.2.2+cpu (environment.yml from Z-Calling repo)

## Outputs

- TorchScript artifact model/k21-full-ZA/scripted_m21.pth for use with z-calling-base at -k 21
- Output BAM with per-base Z probabilities in MM/ML (and ZP) tags when the exported model is exercised

## How to apply

After sweeping odd k-mer sizes 7-25 and confirming k=21 balances AUC (0.9646-0.9796) and CPU training/calling cost, export the winning MLP — trained on PW/IPD kinetic signals from E. coli full-Z vs full-dA reads — as a TorchScript scripted module. Save it at model/k21-full-ZA/scripted_m21.pth so z-calling-base can load it with the matching -k 21 argument; note the companion base-calling model k11-mixed-AZ/scripted_m11.pth uses -k 11 and must not be interchanged. The export must be done from the project's Conda environment (Z-Calling) where torch 2.2.2+cpu is installed, since the tool is designed to run on standard Linux CPUs without GPU acceleration. Verify the export by running build/z-calling-base -k 21 with the scripted model on held-out BAMs and confirming per-base Z probabilities are written into the MM/ML (and ZP) tags of the output BAM.

## Related tools

- **Z-Calling** (Source repository containing the PyTorch MLP training code and the C++ z-calling-base executable that loads the exported TorchScript model with -k 21) — https://github.com/xiaochuanle/Z-Calling
- **PyTorch** (Framework used to train the k-mer feature MLP and to script/serialize it as model/k21-full-ZA/scripted_m21.pth (torch 2.2.2+cpu)) — https://github.com/xiaochuanle/Z-Calling

## Examples

```
build/z-calling-base -k 21 -t 16 <Input_BAM> model/k21-full-ZA/scripted_m21.pth <Output_BAM>
```

## Evaluation signals

- model/k21-full-ZA/scripted_m21.pth exists in the model/ directory and loads successfully in the Z-Calling conda environment (torch 2.2.2+cpu)
- build/z-calling-base -k 21 runs to completion on a filtered test BAM using the exported model, producing an output BAM with MM/ML and ZP tags
- Read-level AUC on held-out fruit fly (Revio, Sequel II) and E. coli (Revio) test sets falls within the reported 0.9646-0.9796 range and F1 within 0.9045-0.9384
- Base-calling AUC/F1 on the Z base calling module fall within the README's stated 0.94-0.96 and 0.85-0.92 ranges
- k=21 was chosen over other odd k in the 7-25 sweep by plotting AUC and CPU training/calling time versus k

## Limitations

- The k21-full-ZA model is trained on full-dA/full-dZ datasets and is intended for classifying entire reads as dZ-modified or native, not site-specific Z/A discrimination
- Model selection depended on CPU training/calling time benchmarks (AMD EPYC 7402); a different CPU profile could shift the optimal k
- Base calling peak memory is ~31 GB, so the export/evaluation workflow needs ≥32 GB RAM

## Evidence
