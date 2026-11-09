---
name: x-ray-detector-noise-transfer-analysis
description: Use when you need to model the MTF and NNPS of a direct (a-Se) or indirect
  (CsI) x-ray imaging detector — or extend the cascade with custom parallel pathways
  — from system YAML descriptions, and validate against measured MTF/NNPS.
when_to_use_negative:
- The task requires per-energy-bin propagation of signal and noise quantities — CASYMIR
  does not propagate these on a per-energy-bin basis
- The detector physics depends on effects not modeled by CASYMIR, such as Fano noise,
  Lubberts effect, optical Swank noise, or detector lag/ghosting
- The target is a photon-counting detector involving processes like adaptive binning,
  which are not yet included
tools:
- name: CASYMIR
  role: 'Core cascaded linear-systems framework: process blocks, Path/Node parallel
    cascades, apply_parallel_process, pixel_integration_2d, model_output'
  repo: https://github.com/radboud-axti/casymir_public
- name: NumPy
  role: Core numerical dependency for frequency-domain signal and Wiener-spectrum
    computation
- name: SciPy
  role: Core dependency used by CASYMIR for the cascade calculations
- name: PyYAML
  role: Loads the system YAML (e.g. example_bct.yaml) describing detector and source
    hardware
- name: SpekPy
  role: X-ray spectrum simulation for the source block (target, filtration)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:a6f644a1b69d8e2a7ca5c0dc191f94028e81798e03116eb371f62aae76de0187
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
yes_rate: 0.9444444444444444
n_claims: 18
---

# x-ray-detector-noise-transfer-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Model x-ray detector resolution and noise propagation with CASYMIR's generalized cascaded linear-systems framework, propagating signal through serial/parallel gain, blur, sampling, and noise process blocks to compute MTF and NNPS up to the detector's Nyquist frequency.

## When to use

Use when you need to model the MTF and NNPS of a direct (a-Se) or indirect (CsI) x-ray imaging detector — or extend the cascade with custom parallel pathways — from system YAML descriptions, and validate against measured MTF/NNPS.

## When NOT to use

- The task requires per-energy-bin propagation of signal and noise quantities — CASYMIR does not propagate these on a per-energy-bin basis
- The detector physics depends on effects not modeled by CASYMIR, such as Fano noise, Lubberts effect, optical Swank noise, or detector lag/ghosting
- The target is a photon-counting detector involving processes like adaptive binning, which are not yet included

## Inputs

- System YAML file (detector block: type, active_layer, px_size, ff, thickness, trapping_depth, elems, add_noise, extra_materials; source block: target_angle, target, SID, filter, external_filter)
- Detector material YAML from casymir/data/detectors (composition, density, K-fluorescence parameters, conversion gain, blur parameters)
- Filtration material YAML from casymir/data/materials
- Measured MTF and NNPS projections from a commercial x-ray system for validation

## Outputs

- CSV file with frequency vector (up to detector Nyquist frequency), MTF, and NNPS
- Optional fits to the MTF and NNPS
- Compressed .npz files with 3D reconstruction-domain arrays: signal transfer, Wiener spectrum, MTF, NNPS, frequency axes, projection angles, run metadata

## How to apply

Describe the system in a YAML file (e.g. example_bct.yaml: indirect CsI detector, 700 um active layer, 0.1518 mm pixel pitch, 0.85 fill factor, W target, Be/Al tube filtration) and load it into CASYMIR, which requires Python 3.10+. Run the 1D detector cascade process blocks, lift to the 2D detector plane with pixel_integration_2d, apply 2D sampling, log normalization, and reconstruction filters, collect views in a SignalStack, and map into a 3D frequency volume with map_dbt_stack_to_volume. For custom architectures (e.g. a Kim-style photodiode-conversion pathway), define parallel branches with Path and Node objects and combine them via apply_parallel_process, which computes pairwise cross-spectral terms at the summation point. Compute MTF = S_out(f)/S_out(0) and NNPS = W_out(f)/S_out(0)^2 (including aliased Wiener spectrum and additive electronic noise) with the model_output process. In standalone mode CASYMIR emits a CSV with the frequency vector (up to Nyquist), MTF, and NNPS; judge correctness by reproducing baseline measurements within the reported envelope (RMS MTF percent error below 6%; for CsI, 5.4% MTF and 5.8% NNPS normalized RMS error).

## Related tools

- **CASYMIR** (Core cascaded linear-systems framework: process blocks, Path/Node parallel cascades, apply_parallel_process, pixel_integration_2d, model_output) — https://github.com/radboud-axti/casymir_public
- **NumPy** (Core numerical dependency for frequency-domain signal and Wiener-spectrum computation)
- **SciPy** (Core dependency used by CASYMIR for the cascade calculations)
- **PyYAML** (Loads the system YAML (e.g. example_bct.yaml) describing detector and source hardware)
- **SpekPy** (X-ray spectrum simulation for the source block (target, filtration))

## Examples

```
python -m examples.bct
```

## Evaluation signals

- Reproduce baseline MTF/NNPS within the reported accuracy envelope: a-Se MTF RMS percent error below 6% with NNPS normalized RMS error below 3%; CsI MTF 5.4% and NNPS 5.8%
- Standalone run produces a CSV whose columns are frequency, MTF, and NNPS, with the frequency vector extending to the detector's Nyquist frequency
- The model_output process yields MTF = S_out(f)/S_out(0) and NNPS = W_out(f)/S_out(0)^2, with NNPS including aliased Wiener spectrum and additive electronic noise (add_noise field, e.g. 100)
- Example scripts save .npz outputs containing 3D reconstruction-domain arrays including signal transfer, Wiener spectrum, MTF, NNPS, frequency axes, projection angles, and run metadata

## Limitations

- Fano noise and photoconductor discharge are not explicitly modeled
- Depth-dependent effects such as the Lubberts effect and optical Swank noise are not accounted for
- The absorption-coefficient approximation performs well for high-quantum-efficiency detectors but introduces larger discrepancies in low-absorption-efficiency systems, particularly at higher energies
- Temporal effects such as detector lag and ghosting are not modeled
- Kim's detector architecture is not provided in this version, though the framework supports it via custom parallel cascades

## Evidence
