---
name: k-fluorescence-parallel-pathway-modeling
description: Use when modeling an indirect-conversion detector with an active scintillator
  layer (e.g., 700 µm CsI in example_bct.yaml) whose K-fluorescence escape/reabsorption
  requires a parallel, not serial, cascade of gain and blur processes.
when_to_use_negative:
- The detector is direct-conversion a-Se with no scintillator, so charge trapping
  is handled by serial cascade blocks instead of a parallel K-fluorescence branch
- The architecture requires per-energy-bin signal/noise propagation, which CASYMIR
  does not support
- Depth-dependent scintillator effects such as the Lubberts effect or optical Swank
  noise dominate, since these are not accounted for by the parallel pathway model
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: CASYMIR
  role: Provides the Path, Node, and apply_parallel_process machinery to assemble
    and combine the K-fluorescence parallel cascade in the frequency domain
  repo: https://github.com/radboud-axti/casymir_public
- name: PyYAML
  role: Loads the system/detector YAML configuration that specifies the CsI active
    layer and its K-fluorescence-related detector fields
  repo: https://github.com/radboud-axti/casymir_public
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:a66c8bbb65d1d94589e71c38945da82a906894f92333265e71e33d743824c73b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/radboud-axti/casymir_public
  repo_url: https://github.com/radboud-axti/casymir_public
  tools:
  - CASYMIR
  - NumPy
  - PyYAML
  - Python
  - SciPy
  - SpekPy
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# k-fluorescence-parallel-pathway-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Model the K-fluorescence escape process in scintillator-based x-ray detectors as a parallel cascade within CASYMIR, combining reabsorbed and escaped pathways with cross-spectral density terms so the MTF and NPS remain computable at every stage of the frequency-domain cascade.

## When to use

Use when modeling an indirect-conversion detector with an active scintillator layer (e.g., 700 µm CsI in example_bct.yaml) whose K-fluorescence escape/reabsorption requires a parallel, not serial, cascade of gain and blur processes.

## When NOT to use

- The detector is direct-conversion a-Se with no scintillator, so charge trapping is handled by serial cascade blocks instead of a parallel K-fluorescence branch
- The architecture requires per-energy-bin signal/noise propagation, which CASYMIR does not support
- Depth-dependent scintillator effects such as the Lubberts effect or optical Swank noise dominate, since these are not accounted for by the parallel pathway model

## Inputs

- CASYMIR Path and Node objects defining each K-fluorescence branch (gain/blur per pathway)
- Signal object from the preceding serial cascade stage carrying S̄, S(f), and W(f)
- Detector material YAML for the scintillator (e.g., CsI) with K-fluorescence parameters from casymir/data/detectors

## Outputs

- Combined Signal object produced by apply_parallel_process, with summed NPS including pairwise cross-spectral terms K_αβ
- Frequency-domain W(f) and S(f) ready for downstream pixel integration and model_output computation of MTF/NNPS

## How to apply

Construct the K-fluorescence parallel cascade using CASYMIR Path and Node objects, where each path represents a fluorescence fate (reabsorption vs. escape) with its own gain and blur transfer functions. Combine the path contributions with apply_parallel_process, computing cross-spectral terms pairwise per Equation 5 (Bernoulli split: K_αβ = −ᾱ(1−ᾱ)) and Equation 6 (quantum fork: K_αβ = 0), and sum the paths with Equations 1–3 to yield a single combined Signal object carrying S̄, S(f), and W(f). The scintillator's K-fluorescence parameters (composition, density, conversion gain, blur) come from the detector material YAML in casymir/data/detectors for the active layer such as CsI. This parallel block is appended after the serial 1D stages (quantum selection, energy conversion, pixel integration) so downstream processes like model_output see a single propagated signal.

## Related tools

- **CASYMIR** (Provides the Path, Node, and apply_parallel_process machinery to assemble and combine the K-fluorescence parallel cascade in the frequency domain) — https://github.com/radboud-axti/casymir_public
- **PyYAML** (Loads the system/detector YAML configuration that specifies the CsI active layer and its K-fluorescence-related detector fields) — https://github.com/radboud-axti/casymir_public

## Examples

```
python -m examples.bct
```

## Evaluation signals

- Propagated Wiener spectra W(f) remain real-valued after combining all parallel paths
- Cross-spectral terms match the paper's relations: Bernoulli split paths give K_αβ = −ᾱ(1−ᾱ); quantum fork paths give K_αβ = 0
- Downstream model_output still satisfies MTF(0) = 1 after the parallel cascade is summed
- Modeled MTF/NNPS for the CsI bCT system stay within the paper's reported accuracy (RMS percent error in MTF 5.4%, normalized RMS error in NNPS 5.8%)
- Summation of parallel paths follows Equations 1–3, i.e., path NPS contributions and cross-terms combine rather than a single-path NPS being returned

## Limitations

- Other gain-variability sources such as Fano noise and photoconductor discharge are not explicitly modeled
- Depth-dependent effects (Lubberts effect, optical Swank noise from depth-dependent optical gain) are not accounted for in the parallel pathway noise variance
- Signal and noise quantities are not propagated on a per-energy-bin basis
- Kim's detector architecture (x-ray conversion within the photodiode from scintillator-transmitted x rays) is not provided out of the box, though the framework allows users to define custom parallel cascades

## Evidence
