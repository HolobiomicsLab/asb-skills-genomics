---
name: mtf-nnps-computation
description: Use when you need to model detector MTF(f) = S_out(f)/S_out(0) and NNPS(f)
  = W_out(f)/S_out(0)^2 for a direct (a-Se) or indirect (CsI) x-ray detector described
  by a YAML system configuration, and validate against measured MTF/NNPS.
when_to_use_negative:
- You need per-energy-bin propagation of signal and noise quantities — CASYMIR does
  not propagate them per energy bin
- Your detector relies on depth-dependent effects such as the Lubberts effect or optical
  Swank noise, or temporal effects like detector lag and ghosting
- You are modeling photon-counting detectors with adaptive binning, or noise sources
  such as Fano noise or photoconductor discharge
edam_operation: null
tools:
- name: CASYMIR
  role: Python package providing the generalized cascaded process blocks, apply_parallel_process,
    and the model_output process that computes MTF and NNPS
  repo: https://github.com/radboud-axti/casymir_public
- name: SpekPy
  role: Simulates the tungsten-target x-ray spectrum with tube and external filtration
    for the detector cascade input
- name: PyYAML
  role: Parses the system YAML configuration to instantiate System, Detector, Tube,
    and Spectrum objects
- name: NumPy
  role: Frequency-domain arrays for S(f) and W(f) during serial/parallel cascade propagation
- name: SciPy
  role: Core scientific dependency for the frequency-domain signal and noise computations
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_003
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:deab076a941b7ed41a05df91ef0e1592896a316a668df27fc81b01d524bb1997
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# mtf-nnps-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute the modulation transfer function (MTF) and normalized noise power spectrum (NNPS) of an x-ray detector by propagating Signal objects (S̄, S(f), W(f)) through a generalized cascaded linear system model implemented in the CASYMIR Python package.

## When to use

Use when you need to model detector MTF(f) = S_out(f)/S_out(0) and NNPS(f) = W_out(f)/S_out(0)^2 for a direct (a-Se) or indirect (CsI) x-ray detector described by a YAML system configuration, and validate against measured MTF/NNPS.

## When NOT to use

- You need per-energy-bin propagation of signal and noise quantities — CASYMIR does not propagate them per energy bin
- Your detector relies on depth-dependent effects such as the Lubberts effect or optical Swank noise, or temporal effects like detector lag and ghosting
- You are modeling photon-counting detectors with adaptive binning, or noise sources such as Fano noise or photoconductor discharge

## Inputs

- System YAML configuration file (e.g., example_bct.yaml or example_dbt.yaml) specifying detector type, active layer material, px_size, thickness, fill factor, add_noise, and tube/source parameters
- Detector material files from casymir/data/detectors (elemental composition, density, K-fluorescence, conversion gain, blur parameters)
- X-ray spectrum simulated with SpekPy (target, anode angle, tube filtration, external filter, SID)

## Outputs

- CSV file containing the frequency vector up to the detector Nyquist frequency, the MTF, and the NNPS
- Stage-wise Signal objects carrying mean signal (S̄), signal transfer S(f), and Wiener spectrum W(f)
- Optional fits to the MTF and NNPS

## How to apply

Load the system configuration (e.g., example_bct.yaml: W target, 10° anode angle, 95 cm SID, 700 µm CsI, 0.1518 mm pixel pitch, fill factor 0.85, add_noise=100) with PyYAML to instantiate System, Detector, Tube, and Spectrum objects, simulating the tungsten spectrum with SpekPy including Be/Al tube filtration and external Al filtration. Propagate the Signal through the serial cascade of process blocks (quantum selection, K-fluorescence parallel cascade built with Path/Node objects and combined via apply_parallel_process, optical blurring/coupling or charge trapping, pixel integration), computing S̄, S(f), and W(f) at each stage. Run the model_output process to compute MTF(f) = S_out(f)/S_out(0) and NNPS(f) = W_out(f)/S_out(0)^2, including the aliased Wiener spectrum from pixel sampling and the additive electronic noise term. Write the frequency vector up to the detector Nyquist frequency together with the MTF and NNPS to a CSV (standalone mode). Judge correctness by MTF(0) = 1, real-valued Wiener spectra, and RMS percent MTF error and normalized RMS NNPS error below 6% against measured baselines (paper reports 5.4% MTF and 5.8% NNPS for CsI bCT).

## Related tools

- **CASYMIR** (Python package providing the generalized cascaded process blocks, apply_parallel_process, and the model_output process that computes MTF and NNPS) — https://github.com/radboud-axti/casymir_public
- **SpekPy** (Simulates the tungsten-target x-ray spectrum with tube and external filtration for the detector cascade input)
- **PyYAML** (Parses the system YAML configuration to instantiate System, Detector, Tube, and Spectrum objects)
- **NumPy** (Frequency-domain arrays for S(f) and W(f) during serial/parallel cascade propagation)
- **SciPy** (Core scientific dependency for the frequency-domain signal and noise computations)

## Examples

```
python -m examples.bct
```

## Evaluation signals

- MTF at zero frequency equals 1 (MTF(0) = S_out(0)/S_out(0))
- Propagated Wiener spectra are real-valued at every cascade stage
- Modeled MTF and NNPS reproduce measured baselines within the paper's envelope: RMS percent MTF error and normalized RMS NNPS error below 6% (5.4% and 5.8% for CsI bCT)
- CSV output contains a frequency vector up to the detector Nyquist frequency (set by px_size = 0.1518 mm) plus MTF and NNPS columns
- Stage-wise parameters match the values in the source YAML configuration

## Limitations

- Fano noise and photoconductor discharge are not explicitly modeled
- Noise variance from depth-dependent effects (Lubberts effect, optical Swank noise) is not accounted for
- Absorption-coefficient approximation (total attenuation minus coherent scatter) introduces moderate quantum-efficiency discrepancies, larger in low-absorption-efficiency systems at higher energies
- Temporal effects (detector lag, ghosting), photon-counting detectors, input-quantum-limited formulations, and Kim-architecture photodiode conversion are not included in this version

## Evidence
