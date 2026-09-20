---
name: noise-power-spectrum-error-quantification
description: Use when comparing NNPS and quantum-efficiency curves produced by two
  coefficient sources (approximate vs. reference) over a shared frequency grid up
  to Nyquist, to quantify propagation-mediated error reduction.
when_to_use_negative:
- The goal is absolute experimental validation of MTF/NNPS against measured detector
  data rather than a model-internal sensitivity comparison of coefficient sources
- The detector operates in a regime where CASYMIR is not applicable, e.g., photon-counting
  detectors with adaptive binning, or systems where Fano noise, Lubberts effect, or
  detector lag/ghosting dominate
- Per-energy-bin propagation of signal and noise is required (CASYMIR does not propagate
  quantities on a per-energy-bin basis)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3872
tools:
- name: CASYMIR
  role: Cascaded linear-system model used to build System/Detector objects, run 1D
    detector stages, and propagate signal to the model_output stage for NNPS
  repo: https://github.com/radboud-axti/casymir_public
- name: mucoeff
  role: Default attenuation-coefficient source providing the baseline quantum-efficiency
    run (mucoeff==1.0.0)
- name: XrayDB
  role: Optional legacy mu_source="XRAYDB" path implementing the total-attenuation-minus-coherent-scatter
    approximation being tested
- name: PyYAML
  role: Parses the example_bct.yaml system configuration and CsI detector material
    YAML into CASYMIR objects
- name: NumPy
  role: Computes relative quantum-efficiency discrepancies and normalized RMS differences
    of NNPS curves over the shared frequency axis
- name: SciPy
  role: Standard numerical dependency for the cascaded model computations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:9909907b86a2ef772f7298a1b9b84085a03cb769a08edc64ac1764656fdabdc2
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3872
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# noise-power-spectrum-error-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how an input approximation (e.g., the built-in absorption-coefficient approximation: total attenuation minus coherent scatter) affects modeled quantum efficiency and propagated NNPS in the CASYMIR cascaded linear-system model, testing whether discrepancies shrink as signal propagates through the cascade.

## When to use

Use when comparing NNPS and quantum-efficiency curves produced by two coefficient sources (approximate vs. reference) over a shared frequency grid up to Nyquist, to quantify propagation-mediated error reduction.

## When NOT to use

- The goal is absolute experimental validation of MTF/NNPS against measured detector data rather than a model-internal sensitivity comparison of coefficient sources
- The detector operates in a regime where CASYMIR is not applicable, e.g., photon-counting detectors with adaptive binning, or systems where Fano noise, Lubberts effect, or detector lag/ghosting dominate
- Per-energy-bin propagation of signal and noise is required (CASYMIR does not propagate quantities on a per-energy-bin basis)

## Inputs

- CASYMIR System YAML configuration (e.g., example_bct.yaml with W target, Be/Al and Al/Air filtration, CsI detector, px_size 0.1518 mm)
- Detector material YAML from casymir/data/detectors (CsI: composition, density, K-fluorescence parameters, conversion gain, blur parameters)
- Attenuation-coefficient source specification: mucoeff (default), mu_source="XRAYDB" (legacy optional extra), or externally supplied total energy absorption coefficient tables

## Outputs

- Baseline and approximate quantum-efficiency curves vs. energy
- NNPS curves on a shared frequency vector up to the detector's Nyquist frequency
- Relative discrepancy table for quantum efficiency between coefficient sources
- Normalized RMS difference of NNPS curves over the shared frequency axis

## How to apply

Load the example_bct.yaml system configuration and the CsI detector material YAML from casymir/data/detectors with PyYAML to build the CASYMIR System and Detector objects. Run the 1D cascaded model three times: (1) with the default mucoeff attenuation-coefficient source as baseline, (2) with the legacy mu_source="XRAYDB" path implementing the built-in approximation (total attenuation minus coherent scatter), and (3) with externally supplied total energy absorption coefficient tables. Propagate signal through the process blocks to the model_output stage on the same frequency grid up to Nyquist in each run. Compute the relative discrepancy in quantum efficiency between approximate and reference runs, and the normalized RMS difference of NNPS curves over the shared frequency axis; confirm the qualitative criterion that NNPS discrepancies are smaller than quantum-efficiency discrepancies. Note the article's caveat that the approximation performs well for high-quantum-efficiency detectors but introduces larger discrepancies in low-absorption-efficiency systems, particularly at higher energies.

## Related tools

- **CASYMIR** (Cascaded linear-system model used to build System/Detector objects, run 1D detector stages, and propagate signal to the model_output stage for NNPS) — https://github.com/radboud-axti/casymir_public
- **mucoeff** (Default attenuation-coefficient source providing the baseline quantum-efficiency run (mucoeff==1.0.0))
- **XrayDB** (Optional legacy mu_source="XRAYDB" path implementing the total-attenuation-minus-coherent-scatter approximation being tested)
- **PyYAML** (Parses the example_bct.yaml system configuration and CsI detector material YAML into CASYMIR objects)
- **NumPy** (Computes relative quantum-efficiency discrepancies and normalized RMS differences of NNPS curves over the shared frequency axis)
- **SciPy** (Standard numerical dependency for the cascaded model computations)

## Examples

```
python -m examples.bct  # run baseline (mucoeff); repeat with legacy mu_source="XRAYDB" to compare QE and NNPS up to Nyquist
```

## Evaluation signals

- NNPS discrepancies are smaller than quantum-efficiency discrepancies when both are computed against the reference coefficient source (the sensitivity-analysis qualitative criterion)
- All three runs share an identical frequency vector up to the detector's Nyquist frequency so NNPS differences are computed point-wise on the same grid
- The legacy XRAYDB comparison runs only with the optional extra installed (`pip install -e ".[xraydb]"`); the default run must use mucoeff==1.0.0
- As a scale reference, the article's validation reported normalized RMS errors for the NNPS below 3% (a-Se) and 5.8% (CsI); sensitivity discrepancies should be reported in comparable normalized-RMS terms
- Discrepancies are expected to be larger in low-absorption-efficiency systems and at higher energies, consistent with the article's discussion

## Limitations

- The absorption-coefficient approximation generally performs well for detectors with high quantum efficiency but introduces larger discrepancies in systems with lower absorption efficiency, particularly at higher energies
- CASYMIR does not model Fano noise, photoconductor discharge, Lubberts effect, optical Swank noise, detector lag/ghosting, or photon-counting processes like adaptive binning, all of which limit the fidelity of the propagated NNPS
- Signal and noise quantities are not propagated on a per-energy-bin basis, and input-labeled (input-quantum-limited) formulations are not included
- The public repository is in transition from CASYMIR v1 to v2, with the v2 code, examples, and documentation expected for release in August 2026; v1 lives under legacy-v1/

## Evidence
