---
name: cascaded-linear-system-noise-propagation
description: Use when modeling direct- or indirect-conversion x-ray detectors (a-Se,
  CsI) with a cascaded linear model and you need to trace how an upstream input approximation
  (e.g., attenuation-coefficient source) affects downstream noise metrics like NNPS.
when_to_use_negative:
- The analysis requires per-energy-bin propagation of signal and noise quantities,
  Fano noise, photoconductor discharge, Lubberts effect, or optical Swank noise —
  CASYMIR does not explicitly model these.
- The detector is a photon-counting system or involves temporal effects such as detector
  lag and ghosting, which are not included in this CASYMIR implementation.
- You need input-quantum-limited (input-labeled) DQE formulations, which are not included.
edam_operation: http://edamontology.org/operation_2426
tools:
- name: CASYMIR
  role: Generalized cascaded linear systems model executing the gain/blur/sampling/noise
    process blocks and the model_output stage
  repo: https://github.com/radboud-axti/casymir_public
- name: Python
  role: Runtime environment (Python 3.10 or newer) for the cascade workflow
  repo: https://github.com/radboud-axti/casymir_public
- name: NumPy
  role: Numerical handling of frequency grids and signal arrays
- name: SciPy
  role: Core dependency for signal/frequency computations
- name: PyYAML
  role: Loads system and detector material YAML files
- name: SpekPy
  role: X-ray spectrum simulation used with XrayDB for spectra handling
- name: XrayDB
  role: Optional extra implementing the legacy mu_source="XRAYDB" approximation (total
    attenuation minus coherent scatter)
- name: mucoeff
  role: Default attenuation-coefficient source (mucoeff==1.0.0) for the baseline quantum
    efficiency
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:13a6a640e2ec1861af0aa483abfd35e18e831c5681b492cf29da7e51332a63c0
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
  - XrayDB
  - mucoeff
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cascaded-linear-system-noise-propagation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Propagate x-ray detector signal through serial/parallel gain, blur, sampling, and additive-noise process blocks in CASYMIR to compute quantum efficiency, MTF, and NNPS, and quantify how upstream approximations (e.g., the attenuation-coefficient source) attenuate by the time they reach the NNPS.

## When to use

Use when modeling direct- or indirect-conversion x-ray detectors (a-Se, CsI) with a cascaded linear model and you need to trace how an upstream input approximation (e.g., attenuation-coefficient source) affects downstream noise metrics like NNPS.

## When NOT to use

- The analysis requires per-energy-bin propagation of signal and noise quantities, Fano noise, photoconductor discharge, Lubberts effect, or optical Swank noise — CASYMIR does not explicitly model these.
- The detector is a photon-counting system or involves temporal effects such as detector lag and ghosting, which are not included in this CASYMIR implementation.
- You need input-quantum-limited (input-labeled) DQE formulations, which are not included.

## Inputs

- System configuration YAML (e.g., example_bct.yaml) describing detector and source blocks
- Detector material YAML from casymir/data/detectors (elemental composition, density, K-fluorescence, conversion gain, blur parameters)
- Attenuation-coefficient source: mucoeff (default), XrayDB (legacy mu_source="XRAYDB"), or external total energy absorption coefficient tables

## Outputs

- Quantum efficiency curve per attenuation-coefficient source
- NNPS curves on a shared frequency grid up to the detector's Nyquist frequency
- Relative QE discrepancy and normalized RMS NNPS difference between coefficient sources

## How to apply

Load a system YAML (e.g., example_bct.yaml) and the detector material YAML from casymir/data/detectors with PyYAML to build the CASYMIR System and Detector objects. Run the 1D cascaded model with mucoeff as the default attenuation-coefficient source to obtain baseline quantum efficiency, then propagate the signal through the process blocks to the model_output stage for the baseline NNPS. Re-run the cascade with the legacy mu_source="XRAYDB" path (total attenuation minus coherent scatter) and, optionally, with externally supplied total energy absorption coefficient tables. Compute the relative quantum-efficiency discrepancy and the normalized RMS difference of the NNPS curves over the shared frequency axis (up to Nyquist); the validation criterion is that NNPS discrepancies are reduced relative to QE discrepancies as the signal propagates through the cascade.

## Related tools

- **CASYMIR** (Generalized cascaded linear systems model executing the gain/blur/sampling/noise process blocks and the model_output stage) — https://github.com/radboud-axti/casymir_public
- **Python** (Runtime environment (Python 3.10 or newer) for the cascade workflow) — https://github.com/radboud-axti/casymir_public
- **NumPy** (Numerical handling of frequency grids and signal arrays)
- **SciPy** (Core dependency for signal/frequency computations)
- **PyYAML** (Loads system and detector material YAML files)
- **SpekPy** (X-ray spectrum simulation used with XrayDB for spectra handling)
- **XrayDB** (Optional extra implementing the legacy mu_source="XRAYDB" approximation (total attenuation minus coherent scatter))
- **mucoeff** (Default attenuation-coefficient source (mucoeff==1.0.0) for the baseline quantum efficiency)

## Examples

```
python -m examples.bct
```

## Evaluation signals

- The NNPS discrepancy between coefficient sources is smaller than the quantum-efficiency discrepancy (sensitivity-analysis qualitative criterion).
- All curves are evaluated on the same frequency grid up to the detector's Nyquist frequency.
- Standalone runs produce a CSV with frequency vector, MTF, and NNPS (v1 behavior) or compressed .npz outputs with MTF, NNPS, and frequency axes in examples/outputs/.
- Cross-check modeled MTF/NNPS against measured values: RMS percent errors below 6% (a-Se MTF; CsI MTF 5.4%) and normalized RMS NNPS errors below 3% (a-Se) and 5.8% (CsI).

## Limitations

- The absorption-coefficient approximation (total attenuation minus coherent scatter) performs well only for high quantum efficiency detectors and introduces larger discrepancies in low-absorption-efficiency systems, particularly at higher energies.
- Fano noise, photoconductor discharge, Lubberts effect, optical Swank noise, detector lag/ghosting, and photon-counting/adaptive-binning processes are not explicitly modeled.
- The legacy mu_source="XRAYDB" path requires installing the optional xraydb extra; xraydb is no longer a required dependency.

## Evidence

- [methods] core dependencies: [evidence span withheld — anchor status: not_found]
