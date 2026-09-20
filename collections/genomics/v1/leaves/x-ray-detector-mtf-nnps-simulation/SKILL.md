---
name: x-ray-detector-mtf-nnps-simulation
description: Use when a task requires modeled MTF and NNPS curves for a direct- (a-Se)
  or indirect-conversion (CsI) x-ray detector under defined tube, filtration, and
  acquisition settings, e.g. reproducing DBT or bCT image-quality baselines.
when_to_use_negative:
- The detector or task involves photon-counting technology with adaptive binning,
  which is not included in CASYMIR
- Modeling requires temporal effects such as detector lag and ghosting, or depth-dependent
  effects like the Lubberts effect or optical Swank noise
- The analysis requires per-energy-bin propagation of signal and noise, Fano noise,
  or photoconductor discharge modeling
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CASYMIR
  role: Core Python package that implements the cascaded linear-systems model and
    generates MTF/NNPS outputs
  repo: https://github.com/radboud-axti/casymir_public
- name: SpekPy
  role: Generates the x-ray tube spectrum for the defined target, anode angle, and
    filtration
- name: NumPy
  role: Numerical computation of the frequency-domain signal, MTF, and NNPS arrays
- name: SciPy
  role: Standard scientific-library dependency for the cascade computations
- name: PyYAML
  role: Parses the system YAML configuration (detector and source blocks)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:d8178162c2fef4b3f2107b6538979fe058a26f778b40cc553a62fb7831005ec2
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
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# x-ray-detector-mtf-nnps-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Models x-ray detector resolution and noise with CASYMIR, an open-source Python cascaded linear-systems package that propagates signal through gain, blur, sampling, and noise process blocks to produce MTF and NNPS curves, validated against commercial a-Se (DM/DBT) and CsI (bCT) detectors.

## When to use

Use when a task requires modeled MTF and NNPS curves for a direct- (a-Se) or indirect-conversion (CsI) x-ray detector under defined tube, filtration, and acquisition settings, e.g. reproducing DBT or bCT image-quality baselines.

## When NOT to use

- The detector or task involves photon-counting technology with adaptive binning, which is not included in CASYMIR
- Modeling requires temporal effects such as detector lag and ghosting, or depth-dependent effects like the Lubberts effect or optical Swank noise
- The analysis requires per-energy-bin propagation of signal and noise, Fano noise, or photoconductor discharge modeling

## Inputs

- System YAML configuration file (e.g. example_dbt.yaml) with detector block (type, active_layer, px_size, ff, thickness, trapping_depth, elems, add_noise, extra_materials) and source block (target, target_angle, SID, filter, external_filter)
- Detector material YAML from casymir/data/detectors (elemental composition, density, K-fluorescence parameters, conversion gain, blur parameters)
- User-defined filtration material YAMLs from casymir/data/materials (SpekPy materials by formula or fractional composition)
- Acquisition parameters set in the calling script (view count, angular span, reconstruction filter settings, voxel sampling)

## Outputs

- CSV file with the frequency vector (up to the detector's Nyquist frequency), modeled MTF, and NNPS (standalone mode)
- Optional fits to the MTF and NNPS curves
- Compressed .npz example outputs containing 3D reconstruction-domain arrays: signal transfer, Wiener spectrum, MTF, NNPS, frequency axes, projection angles, and run metadata

## How to apply

Install CASYMIR (Python 3.10+) from the repository root with its core dependencies (NumPy, SciPy, PyYAML, SpekPy), then load a system YAML such as example_dbt.yaml, which defines the detector block (active layer, px_size, fill factor, thickness, add_noise electronic noise) and the source block (W target, target_angle, SID, internal and external filtration). Load the matching detector material definition from casymir/data/detectors and build the System, Detector, Tube, and Spectrum objects. Propagate the Signal through the cascaded blocks in order — quantum selection, K-fluorescence pathways combined via apply_parallel_process, charge-trapping blur, pixel integration, and noise aliasing. Compute outputs with the model_output process, using MTF = S_out(f)/S_out(0) and NNPS = W_out(f)/S_out(0)^2 with the aliased Wiener spectrum plus electronic noise. Run in standalone mode to write a CSV containing the frequency vector up to the detector's Nyquist frequency, the MTF, and the NNPS; optionally generate fits to both curves.

## Related tools

- **CASYMIR** (Core Python package that implements the cascaded linear-systems model and generates MTF/NNPS outputs) — https://github.com/radboud-axti/casymir_public
- **SpekPy** (Generates the x-ray tube spectrum for the defined target, anode angle, and filtration)
- **NumPy** (Numerical computation of the frequency-domain signal, MTF, and NNPS arrays)
- **SciPy** (Standard scientific-library dependency for the cascade computations)
- **PyYAML** (Parses the system YAML configuration (detector and source blocks))

## Examples

```
python -m pip install -e . && python -m examples.dbt
```

## Evaluation signals

- Modeled MTF RMS percent error below 6% for the a-Se detector when compared against experimentally measured DBT baselines
- Normalized RMS error for the NNPS below 3% for the a-Se detector (below 6% across evaluated DBT and bCT spectra)
- Output frequency vector extends exactly up to the detector's Nyquist frequency
- Internal consistency check: MTF = S_out(f)/S_out(0) (normalized to 1 at f=0) and NNPS = W_out(f)/S_out(0)^2 with aliased Wiener spectrum plus add_noise electronic noise
- CsI cross-check: RMS percent error in MTF 5.4% and normalized RMS NNPS error 5.8% for the breast-CT case

## Limitations

- Fano noise and photoconductor discharge are not explicitly modeled
- Signal and noise quantities are not propagated on a per-energy-bin basis; input-quantum-limited (input-labeled) formulations are not included
- Depth-dependent effects such as the Lubberts effect and optical Swank noise are not accounted for; the absorption-coefficient approximation (total attenuation minus coherent scatter) introduces larger discrepancies in low-absorption-efficiency systems, particularly at higher energies
- Kim's detector architecture, temporal effects (lag, ghosting), and photon-counting detectors with adaptive binning are not included in this version
- Repository is in transition from v1 to v2; public v2 code and documentation are expected in August 2026, with the v1 projection-domain implementation in legacy-v1/

## Evidence

- [methods] CASYMIR relies on standard libraries such as NumPy, SciPy, and PyYAML, as well as SpekPy: [evidence span withheld — anchor status: not_found]
- [discussion] However, other sources of gain variability, such as Fano noise or photoconductor discharge, are not explicitly modeled.: [evidence span withheld — anchor status: not_found]
