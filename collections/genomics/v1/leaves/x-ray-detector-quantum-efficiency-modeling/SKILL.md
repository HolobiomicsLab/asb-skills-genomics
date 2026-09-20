---
name: x-ray-detector-quantum-efficiency-modeling
description: Use when a cascaded x-ray detector model must be checked for sensitivity
  to its absorption-coefficient approximation, e.g. verifying that NNPS discrepancies
  shrink relative to quantum-efficiency discrepancies as signal propagates through
  the cascade.
when_to_use_negative:
- The analysis requires per-energy-bin propagation of signal and noise — CASYMIR does
  not propagate signal and noise quantities on a per-energy-bin basis
- The detector is photon-counting or uses processes like adaptive binning — such technologies
  are not yet included in CASYMIR
- Temporal effects such as detector lag and ghosting dominate the noise budget — the
  current implementation does not model them
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: CASYMIR
  role: Provides the 1D cascaded linear-system model, System/Detector objects, and
    the model_output process for computing quantum efficiency and NNPS
  repo: https://github.com/radboud-axti/casymir_public
- name: mucoeff
  role: Default attenuation-coefficient source for the baseline quantum-efficiency
    run (mucoeff==1.0.0)
- name: XrayDB
  role: Implements the legacy mu_source="XRAYDB" approximation path (total attenuation
    minus coherent scatter) being evaluated; optional extra dependency
- name: PyYAML
  role: Loads the example_bct.yaml system configuration and CsI detector material
    YAML from casymir/data/detectors
- name: SpekPy
  role: Handles x-ray spectrum simulation for the source definition used in the cascade
    runs
- name: NumPy
  role: Computes the relative QE discrepancy and normalized RMS difference of NNPS
    curves over the shared frequency axis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:a77e7bdc074095194df727709b2a7bcb8d4739fc8d4125376a3d74fbc2856a9e
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# x-ray-detector-quantum-efficiency-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how CASYMIR's modeled quantum efficiency (QE) and propagated NNPS depend on the attenuation-coefficient source, comparing the default mucoeff tables, the legacy mu_source="XRAYDB" approximation (total attenuation minus coherent scatter), and externally supplied total energy absorption coefficients through the 1D cascaded model.

## When to use

Use when a cascaded x-ray detector model must be checked for sensitivity to its absorption-coefficient approximation, e.g. verifying that NNPS discrepancies shrink relative to quantum-efficiency discrepancies as signal propagates through the cascade.

## When NOT to use

- The analysis requires per-energy-bin propagation of signal and noise — CASYMIR does not propagate signal and noise quantities on a per-energy-bin basis
- The detector is photon-counting or uses processes like adaptive binning — such technologies are not yet included in CASYMIR
- Temporal effects such as detector lag and ghosting dominate the noise budget — the current implementation does not model them

## Inputs

- example_bct.yaml system configuration (Koning bCT: W target, 10 deg anode angle, SID 95 cm, Be/Al filtration)
- CsI detector material YAML from casymir/data/detectors (700 um CsI active layer, 0.1518 mm pixel pitch, ff 0.85)
- mucoeff attenuation-coefficient tables (default source, mucoeff==1.0.0)
- Legacy mu_source="XRAYDB" coefficient path (optional extra: pip install -e ".[xraydb]")
- Externally supplied total energy absorption coefficient tables from the Supplemental Material

## Outputs

- Quantum efficiency curves for each of the three attenuation-coefficient sources
- NNPS curves for each source on the shared frequency grid up to the detector's Nyquist frequency
- Relative discrepancy in quantum efficiency between approximate and reference coefficient sources
- Normalized RMS difference of the corresponding NNPS curves over the shared frequency axis
- Discrepancy table for expert review of the validation criterion

## How to apply

Build the CASYMIR System and Detector objects by loading example_bct.yaml and the CsI material YAML from casymir/data/detectors with PyYAML, then run the 1D cascaded model three times: once with the default mucoeff attenuation-coefficient source (baseline), once with the legacy mu_source="XRAYDB" path implementing the built-in approximation (total attenuation minus coherent scatter), and once with the externally supplied total energy absorption coefficient tables from the Supplemental Material. For each run, record the quantum efficiency and propagate the signal through the process blocks to the model_output stage to obtain the NNPS on the same frequency grid up to the detector's Nyquist frequency. Compute the relative QE discrepancy between the approximate and reference coefficient sources, and the normalized RMS difference of the NNPS curves over the shared frequency axis. Apply the validation criterion that NNPS discrepancies are smaller than QE discrepancies, since the approximation's impact is reduced as the signal propagates through the cascaded model; expect larger discrepancies for systems with lower absorption efficiency, particularly at higher energies.

## Related tools

- **CASYMIR** (Provides the 1D cascaded linear-system model, System/Detector objects, and the model_output process for computing quantum efficiency and NNPS) — https://github.com/radboud-axti/casymir_public
- **mucoeff** (Default attenuation-coefficient source for the baseline quantum-efficiency run (mucoeff==1.0.0))
- **XrayDB** (Implements the legacy mu_source="XRAYDB" approximation path (total attenuation minus coherent scatter) being evaluated; optional extra dependency)
- **PyYAML** (Loads the example_bct.yaml system configuration and CsI detector material YAML from casymir/data/detectors)
- **SpekPy** (Handles x-ray spectrum simulation for the source definition used in the cascade runs)
- **NumPy** (Computes the relative QE discrepancy and normalized RMS difference of NNPS curves over the shared frequency axis)

## Examples

```
python -m pip install -e ".[xraydb]"; python -m examples.bct
```

## Evaluation signals

- All three cascade runs must produce NNPS on the same frequency grid, up to the detector's Nyquist frequency, before discrepancy metrics are computed
- The normalized RMS difference of the NNPS curves must be smaller than the relative quantum-efficiency discrepancy (validation criterion: NNPS impact is reduced as the signal propagates through the model)
- Discrepancies should be moderate for the high-quantum-efficiency CsI bCT system and would be expected to grow for lower absorption-efficiency systems, particularly at higher energies
- The baseline run must use mucoeff (mucoeff==1.0.0); the XRAYDB comparison path requires the optional extra installed via pip install -e ".[xraydb]"
- The system and detector configuration (example_bct.yaml, CsI material YAML) must be identical across all three runs so only the coefficient source varies

## Limitations

- The absorption-coefficient approximation (total attenuation minus coherent scatter) performs well for detectors with high quantum efficiency but introduces larger discrepancies in systems with lower absorption efficiency, particularly at higher energies
- CASYMIR does not propagate signal and noise quantities on a per-energy-bin basis, which limits how coefficient errors are treated across the spectrum
- Sources of gain variability such as Fano noise and photoconductor discharge are not explicitly modeled, so they cannot contribute to the NNPS comparison
- The repository is in transition from CASYMIR v1 to v2; the legacy XRAYDB path is kept only for comparison and xraydb is no longer a required dependency

## Evidence

- [methods] Dependency stack for the cascade model: [evidence span withheld — anchor status: not_found]
