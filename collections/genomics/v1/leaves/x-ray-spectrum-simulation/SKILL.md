---
name: x-ray-spectrum-simulation
description: Use when a detector model requires the incident x-ray spectrum as input
  — e.g., simulating a W-target tube with Be/Al internal filtration and Al external
  filtration for breast CT or DBT detector cascades.
when_to_use_negative:
- The task requires per-energy-bin propagation of signal and noise quantities — CASYMIR
  explicitly does not support this
- Photon-counting detectors or adaptive-binning processes, which are not included
  in CASYMIR's scope
- Spectra requiring effects outside SpekPy's tube model scope (e.g., Fano noise or
  photoconductor discharge effects tied to energy deposition)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SpekPy
  role: Simulates the tungsten-target x-ray spectrum with Be/Al tube filtration and
    Al external filtration as input to the CASYMIR model
- name: CASYMIR
  role: Consumes the simulated spectrum via the System/Tube/Spectrum objects to propagate
    signal through the detector cascade
  repo: https://github.com/radboud-axti/casymir_public
- name: PyYAML
  role: Parses the example_bct.yaml source block that parameterizes the spectrum simulation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:b34108c08d42bb76c4e3e65562c79de47ed7c43b50b2380de045a4a4cb7f2369
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# x-ray-spectrum-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate the polychromatic x-ray spectrum incident on the detector (e.g., tungsten-target with tube and external filtration) using SpekPy as the spectral source feeding the CASYMIR cascaded linear system model for MTF/NNPS computation.

## When to use

Use when a detector model requires the incident x-ray spectrum as input — e.g., simulating a W-target tube with Be/Al internal filtration and Al external filtration for breast CT or DBT detector cascades.

## When NOT to use

- The task requires per-energy-bin propagation of signal and noise quantities — CASYMIR explicitly does not support this
- Photon-counting detectors or adaptive-binning processes, which are not included in CASYMIR's scope
- Spectra requiring effects outside SpekPy's tube model scope (e.g., Fano noise or photoconductor discharge effects tied to energy deposition)

## Inputs

- System YAML source block (target, target_angle, SID, filter, external_filter)
- Filtration material definitions from casymir/data/materials (SpekPy material formulas or fractional elemental compositions)
- Detector cover materials (extra_materials pairs) for attenuation through the detector entrance

## Outputs

- SpekPy-simulated polychromatic x-ray spectrum (energy-fluence vs. energy) attached to the CASYMIR Spectrum object
- Spectrum-attenuated quantum efficiency driving the CsI or a-Se detector cascade S-bar, S(f), and W(f) computations

## How to apply

Define the source parameters in the system YAML: target material (W, Mo, or Rh), anode target_angle (e.g., 10 degrees for the Koning bCT example), SID in cm (e.g., 95), internal tube filter as material/thickness pairs [(Be, 1.4), (Al, 1.514)], and external_filter between tube and detector [(Al, 10), (Air, 950)]. Use SpekPy — CASYMIR's designated spectrum-simulation dependency alongside XrayDB for attenuation handling — to generate the tungsten spectrum with the specified filtration, then pass it into the CASYMIR System/Tube/Spectrum objects instantiated from the YAML so the Signal can be propagated through the detector cascade. The spectrum's attenuation through detector cover materials listed in extra_materials (e.g., Carbon Fiber 2.5, Silicon Dioxide 1) and quantum efficiency in the active layer (CsI 700 micrometers) depends directly on this simulated spectrum. Note that CASYMIR does not propagate signal and noise per energy bin, so a single spectrum-averaged propagation is the intended use.

## Related tools

- **SpekPy** (Simulates the tungsten-target x-ray spectrum with Be/Al tube filtration and Al external filtration as input to the CASYMIR model)
- **CASYMIR** (Consumes the simulated spectrum via the System/Tube/Spectrum objects to propagate signal through the detector cascade) — https://github.com/radboud-axti/casymir_public
- **PyYAML** (Parses the example_bct.yaml source block that parameterizes the spectrum simulation)

## Examples

```
python -m examples.bct
```

## Evaluation signals

- The spectrum yields modeled MTF within ~5.4% RMS percent error and NNPS within ~5.8% normalized RMS error for the CsI bCT configuration against experimental measurements
- All modeled MTF RMS errors and NNPS normalized RMS errors stay below 6% for the evaluated DBT and bCT spectra
- Sensitivity analysis: the absorption-coefficient approximation shows moderate quantum-efficiency discrepancies but reduced NNPS impact, so spectrum-driven quantum efficiency changes should not drastically shift NNPS
- Simulated spectrum respects the specified filtration stack: internal [(Be, 1.4), (Al, 1.514)] and external [(Al, 10), (Air, 950)] for the bCT example

## Limitations

- CASYMIR does not propagate signal and noise quantities on a per-energy-bin basis; the spectrum is integrated into averaged quantities
- The absorption-coefficient approximation (total attenuation minus coherent scatter) performs well for high quantum-efficiency detectors but introduces larger discrepancies in low-absorption-efficiency systems, particularly at higher energies
- Gain variability sources such as Fano noise are not explicitly modeled, so spectrum-dependent effects of that kind are out of scope

## Evidence

- [methods] SpekPy as spectrum simulator: [evidence span withheld — anchor status: not_found]
