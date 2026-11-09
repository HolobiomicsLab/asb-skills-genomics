---
name: cascaded-linear-system-modeling
description: Use when the task requires modeling MTF and NNPS of an x-ray detector
  (e.g., a-Se direct conversion for DM/DBT or CsI indirect conversion for bCT/CBCT)
  under defined acquisition conditions from a system YAML configuration.
when_to_use_negative:
- 'Detectors or phenomena outside the model''s scope: photon-counting detectors with
  adaptive binning, temporal effects such as detector lag and ghosting, Fano noise,
  photoconductor discharge, the Lubberts effect, and optical Swank noise are not explicitly
  modeled'
- Analyses requiring per-energy-bin propagation of signal and noise quantities, or
  input-quantum-limited (input-labeled) formulations
- Detectors with low absorption efficiency at higher energies, where the absorption-coefficient
  approximation (total attenuation minus coherent scatter) introduces larger discrepancies
edam_operation: http://edamontology.org/operation_0244
edam_topics: []
tools:
- name: CASYMIR
  role: Core Python package implementing the generalized cascaded linear system model
    for direct and indirect x-ray detectors
  repo: https://github.com/radboud-axti/casymir_public
- name: Python
  role: Runtime language (Python 3.10 or newer required) for executing the cascade
    and model_output computations
  repo: https://github.com/radboud-axti/casymir_public
- name: SpekPy
  role: Simulates the x-ray spectrum (e.g., W target with Be/Al tube filtration and
    external filtration) feeding the cascade
- name: NumPy
  role: Frequency-domain array computations for S(f), W(f), and Wiener-spectrum aliasing
- name: SciPy
  role: Core scientific dependency of CASYMIR for the frequency-domain process blocks
- name: PyYAML
  role: Parses the system YAML files to populate the System, Detector, Tube, and Spectrum
    objects
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:1281370739688ef2a2915bf8e089ff243d759a7a33fdcdf0dcfeca08db85c769
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cascaded-linear-system-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Model direct- or indirect-conversion x-ray detectors as a cascaded linear system of gain, blur, sampling, and noise process blocks using CASYMIR, propagating signal S̄, S(f), and W(f) through serial/parallel frequency-domain stages to compute MTF and NNPS.

## When to use

Use when the task requires modeling MTF and NNPS of an x-ray detector (e.g., a-Se direct conversion for DM/DBT or CsI indirect conversion for bCT/CBCT) under defined acquisition conditions from a system YAML configuration.

## When NOT to use

- Detectors or phenomena outside the model's scope: photon-counting detectors with adaptive binning, temporal effects such as detector lag and ghosting, Fano noise, photoconductor discharge, the Lubberts effect, and optical Swank noise are not explicitly modeled
- Analyses requiring per-energy-bin propagation of signal and noise quantities, or input-quantum-limited (input-labeled) formulations
- Detectors with low absorption efficiency at higher energies, where the absorption-coefficient approximation (total attenuation minus coherent scatter) introduces larger discrepancies

## Inputs

- System configuration YAML (e.g., example_dbt.yaml, example_bct.yaml, example_cbct.yaml, example_novation_prototype.yaml) with detector block (type, active_layer, px_size, ff, thickness, trapping_depth, elems, add_noise, extra_materials) and source block (target_angle, target, SID, filter, external_filter)
- Detector material definition YAML from casymir/data/detectors (composition, density, K-fluorescence parameters, conversion gain, blur parameters)
- User-defined filtration material YAMLs from casymir/data/materials (SpekPy materials by chemical formula or fractional composition)
- Acquisition parameters set in the calling script (view count, angular span, reconstruction filter settings, voxel sampling)

## Outputs

- CSV file (standalone mode) containing the frequency vector up to the detector Nyquist frequency, the modeled MTF, and the NNPS
- Compressed .npz outputs (examples/) with 3D reconstruction-domain signal transfer, Wiener spectrum, MTF, NNPS, frequency axes, projection angles, and run metadata
- Intermediary Signal objects carrying S̄, S(f), and W(f) at any cascade stage

## How to apply

Install CASYMIR (Python 3.10+, pip install -e .; core deps numpy, scipy, spekpy, pyyaml, tqdm, mucoeff==1.0.0) and load a system YAML (e.g., example_dbt.yaml or example_bct.yaml with a Koning bCT: W target, 10° anode angle, 95 cm SID, 700 µm CsI, 0.1518 mm pixel pitch, add_noise=100) to instantiate System, Detector, Tube, and Spectrum objects, using SpekPy for the tungsten spectrum with internal Be/Al filtration and PMMA or Al external filtration. Assemble the serial 1D detector cascade of process blocks (quantum selection, energy conversion, charge trapping for a-Se or optical blurring/coupling for CsI, pixel integration) applied to Signal objects, and construct the K-fluorescence parallel cascade with Path and Node objects combined via apply_parallel_process, with cross-spectral terms K_αβ = −ᾱ(1−ᾱ) for Bernoulli splits and 0 for quantum forks. For 2D/3D workflows, lift the 1D result with pixel_integration_2d, apply 2D sampling/log normalization/reconstruction filters, collect views in a SignalStack, and map to a 3D frequency volume (for DBT use map_dbt_stack_to_volume with splat=False for nearest-bin deposition or splat=True with gaussian/triangular/sinc kernels). Compute outputs with the model_output process: MTF(f) = S_out(f)/S_out(0) and NNPS(f) = W_out(f)/S_out(0)², including the aliased Wiener spectrum and additive electronic noise; standalone mode writes a CSV with the frequency vector up to the detector Nyquist frequency, the MTF, and the NNPS.

## Related tools

- **CASYMIR** (Core Python package implementing the generalized cascaded linear system model for direct and indirect x-ray detectors) — https://github.com/radboud-axti/casymir_public
- **Python** (Runtime language (Python 3.10 or newer required) for executing the cascade and model_output computations) — https://github.com/radboud-axti/casymir_public
- **SpekPy** (Simulates the x-ray spectrum (e.g., W target with Be/Al tube filtration and external filtration) feeding the cascade)
- **NumPy** (Frequency-domain array computations for S(f), W(f), and Wiener-spectrum aliasing)
- **SciPy** (Core scientific dependency of CASYMIR for the frequency-domain process blocks)
- **PyYAML** (Parses the system YAML files to populate the System, Detector, Tube, and Spectrum objects)

## Examples

```
python -m pip install -e . ; python -m examples.dbt
```

## Evaluation signals

- MTF(0) = 1 and propagated Wiener spectra are real-valued at every cascade stage
- Against experimentally measured a-Se DBT baselines: RMS percent error in MTF below 6% and normalized RMS error in NNPS below 3%
- Against experimentally measured CsI bCT baselines: 5.4% RMS percent error in MTF and 5.8% normalized RMS error in NNPS (all errors below 6% for evaluated DBT and bCT spectra)
- Frequency vector in the standalone CSV extends exactly up to the detector Nyquist frequency defined by px_size
- Stage-wise parameters match the values in the system YAML configuration (e.g., pixel pitch, thickness, add_noise=100, fill factor)

## Limitations

- Fano noise and photoconductor discharge gain variability are not explicitly modeled
- Depth-dependent effects such as the Lubberts effect and optical Swank noise are not accounted for
- Signal and noise are not propagated on a per-energy-bin basis; absorption-coefficient approximation causes moderate quantum-efficiency discrepancies, especially in low-absorption-efficiency systems at higher energies
- Temporal effects (detector lag, ghosting), photon-counting detectors with adaptive binning, and Kim's detector architecture are not included, though custom parallel cascades are supported

## Evidence

- [abstract] CASYMIR is a flexible and open-source Python package for modeling direct/indirect detectors: "We introduce CASYMIR, a flexible and open-source Python package capable of modeling direct and indirect-conversion x-ray imaging detectors under various acquisition conditions."
- [abstract] Validation accuracy of the modeled MTF and NNPS: "The modeled MTF had root-mean-square (RMS) percent errors below 6% for the a-Se detector, while the normalized RMS error for the NNPS was below 3%."
- [methods] Parallel cascade combination function: "the apply_parallel_process function is used to generate the combined Signal object"
- [results] Standalone-mode output format: "When run in standalone mode, CASYMIR generates a CSV file containing the frequency vector (up to the detector's Nyquist frequency), the MTF, and the NNPS."
- [readme] Installation command and dependency set: "CASYMIR requires Python 3.10 or newer. Install it from the repository root with:

```bash
python -m pip install -e .```"
- [readme] 3D model block ordering from 1D cascade to frequency volume: "1. Run the 1D detector stages.
2. Lift the result to a 2D detector plane with `pixel_integration_2d`.
3. Apply 2D sampling, log normalization, and reconstruction filters."
