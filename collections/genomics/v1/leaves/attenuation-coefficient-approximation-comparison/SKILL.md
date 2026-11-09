---
name: attenuation-coefficient-approximation-comparison
description: Use when auditing or configuring the attenuation-coefficient source of
  a CASYMIR detector cascade, e.
when_to_use_negative:
- The goal is per-energy-bin propagation of signal and noise quantities — CASYMIR
  does not propagate signal and noise on a per-energy-bin basis, so this skill cannot
  refine the approximation that way
- The detector is a photon-counting system with adaptive binning or has temporal effects
  (lag, ghosting) that dominate the noise budget — these are outside the current CASYMIR
  implementation this comparison is built on
- The approximation's effect on quantum efficiency itself is not of interest and the
  system's absorption behavior is already known to be high-efficiency across the spectrum
  (the approximation 'generally performs well for detectors with high quantum efficiency')
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: CASYMIR
  role: Python package whose cascaded linear-system model is run under three attenuation-coefficient
    sources to compare quantum efficiency and NNPS; System and Detector objects built
    from YAML configurations in casymir/data/detectors
  repo: https://github.com/radboud-axti/casymir_public
- name: mucoeff
  role: Default attenuation-coefficient source (mucoeff==1.0.0) providing the reference/baseline
    quantum-efficiency run
- name: XrayDB
  role: Optional dependency backing the legacy mu_source="XRAYDB" path that implements
    the built-in approximation (total attenuation minus coherent scatter); must be
    installed via the [xraydb] extra
- name: PyYAML
  role: Loads the example_bct.yaml system configuration and CsI detector material
    YAML to construct the System and Detector objects
- name: SpekPy
  role: Handles x-ray spectrum simulation for the W/Be/Al acquisition conditions under
    which the coefficient sources are compared
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:fa13a6c5cb465794679e1fdb26c085804fb34865a753f32595bc7057c80f1513
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# attenuation-coefficient-approximation-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A sensitivity analysis that quantifies how CASYMIR's modeled quantum efficiency and propagated NNPS depend on the attenuation-coefficient source — the built-in approximation (total attenuation minus coherent scatter via XrayDB) versus externally supplied energy absorption coefficients — and tests whether NNPS discrepancies shrink relative to quantum-efficiency discrepancies.

## When to use

Use when auditing or configuring the attenuation-coefficient source of a CASYMIR detector cascade, e.g. deciding between the default mucoeff path, the legacy mu_source="XRAYDB" approximation, or external absorption-coefficient tables, or when reporting the uncertainty this approximation introduces in quantum efficiency and NNPS.

## When NOT to use

- The goal is per-energy-bin propagation of signal and noise quantities — CASYMIR does not propagate signal and noise on a per-energy-bin basis, so this skill cannot refine the approximation that way
- The detector is a photon-counting system with adaptive binning or has temporal effects (lag, ghosting) that dominate the noise budget — these are outside the current CASYMIR implementation this comparison is built on
- The approximation's effect on quantum efficiency itself is not of interest and the system's absorption behavior is already known to be high-efficiency across the spectrum (the approximation 'generally performs well for detectors with high quantum efficiency')

## Inputs

- System configuration YAML (e.g. casymir example_bct.yaml: W target, 10 deg anode angle, SID 95 cm, Be/Al internal filter, Al/Air external filter, CsI 700 um indirect detector, px_size 0.1518 mm, add_noise 100)
- Detector material YAML from casymir/data/detectors (CsI: elemental composition, density, K-fluorescence parameters, conversion gain, blur parameters)
- Attenuation-coefficient source option: default mucoeff==1.0.0, legacy XrayDB path, or external total energy absorption coefficient tables from the Supplemental Material
- X-ray spectrum parameters (target, target_angle, filtration) handled via SpekPy

## Outputs

- Baseline quantum efficiency from the 1D cascaded model (mucoeff source)
- Approximate quantum efficiency and NNPS from the mu_source="XRAYDB" run, sampled on the same frequency grid up to the detector Nyquist frequency
- Relative quantum-efficiency discrepancy table comparing approximate vs reference coefficient sources
- Normalized RMS difference of NNPS curves over the shared frequency axis
- Discrepancy table used for expert review of the sensitivity criterion

## How to apply

Load a system configuration such as example_bct.yaml and the CsI detector material YAML from casymir/data/detectors with PyYAML to build the CASYMIR System and Detector objects. Run the 1D cascaded model three times: first with the default mucoeff attenuation-coefficient source to obtain baseline quantum efficiency and baseline NNPS via the model_output stage; then with the legacy mu_source="XRAYDB" path implementing the built-in approximation (total attenuation minus coherent scatter), recording the approximate quantum efficiency and NNPS on the same frequency grid up to Nyquist; and finally with externally supplied total energy absorption coefficient tables from the Supplemental Material. Compute the relative discrepancy in quantum efficiency between the approximate and reference coefficient sources and the normalized RMS difference of the NNPS curves over the shared frequency axis. The validation criterion is qualitative and comparative: confirm that the NNPS discrepancies are reduced relative to the quantum-efficiency discrepancies as the signal propagates through the cascaded model.

## Related tools

- **CASYMIR** (Python package whose cascaded linear-system model is run under three attenuation-coefficient sources to compare quantum efficiency and NNPS; System and Detector objects built from YAML configurations in casymir/data/detectors) — https://github.com/radboud-axti/casymir_public
- **mucoeff** (Default attenuation-coefficient source (mucoeff==1.0.0) providing the reference/baseline quantum-efficiency run)
- **XrayDB** (Optional dependency backing the legacy mu_source="XRAYDB" path that implements the built-in approximation (total attenuation minus coherent scatter); must be installed via the [xraydb] extra)
- **PyYAML** (Loads the example_bct.yaml system configuration and CsI detector material YAML to construct the System and Detector objects)
- **SpekPy** (Handles x-ray spectrum simulation for the W/Be/Al acquisition conditions under which the coefficient sources are compared)

## Examples

```
python -m pip install -e ".[xraydb]" && python -m examples.bct
```

## Evaluation signals

- The normalized RMS difference of NNPS curves over the shared frequency axis (up to Nyquist) is smaller than the relative quantum-efficiency discrepancy — the qualitative sensitivity criterion must hold
- All three cascade runs (mucoeff, XRAYDB, external tables) execute to the model_output stage and produce NNPS on the same frequency grid, allowing direct comparison
- Discrepancies are larger at higher energies and in systems with lower absorption efficiency; the comparison should reproduce that energy-dependent pattern rather than a uniform offset
- The expert-reviewed discrepancy table documents that the approximation 'performs well for detectors with high quantum efficiency' — for the CsI bCT example, deviations should remain moderate (article-wide MTF/NNPS validation errors stay below ~6%)
- The optional xraydb extra is installed; running mu_source="XRAYDB" without it should fail, confirming the legacy path is isolated from the mucoeff default

## Limitations

- The approximation (total attenuation minus coherent scatter) generally performs well for detectors with high quantum efficiency but introduces larger discrepancies in systems with lower absorption efficiency, particularly at higher energies
- CASYMIR does not propagate signal and noise quantities on a per-energy-bin basis, so the comparison cannot resolve energy-resolved error structure
- The comparison is restricted to the frequency grid up to the detector Nyquist frequency
- CASYMIR omits Fano noise, photoconductor discharge, Lubberts effect, optical Swank noise, detector lag/ghosting, and photon-counting processes — these unmodeled terms are outside the scope of this sensitivity analysis

## Evidence

- [discussion] Sensitivity analysis showed the absorption-coefficient approximation has moderate impact on quantum efficiency but reduced impact on NNPS: "The results show that, while the approximation introduces moderate discrepancies in quantum efficiency, its impact on the NNPS is reduced as it propagates through the model."
- [discussion] Energy and absorption-efficiency dependence of the approximation: "This approach generally performs well for detectors with high quantum efficiency, but it introduces larger discrepancies in systems with lower absorption efficiency, particularly at higher energies."
- [readme] Legacy XRAYDB path is an optional extra separate from the default mucoeff source: "install the optional extra only if you need the legacy `mu_source="XRAYDB"` comparison path."
- [methods] Library stack used to run the comparison: [evidence span withheld — anchor status: not_found]
