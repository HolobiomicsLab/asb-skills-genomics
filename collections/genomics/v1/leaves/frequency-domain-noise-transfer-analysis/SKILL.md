---
name: frequency-domain-noise-transfer-analysis
description: Use when you need modeled MTF and NNPS curves for a direct (a-Se) or
  indirect (CsI) conversion x-ray detector under specified tube, filtration, and acquisition
  conditions, to compare against measured image-quality baselines.
when_to_use_negative:
- The analysis requires per-energy-bin propagation of signal and noise, Fano noise,
  photoconductor discharge, or Lubberts/optical Swank depth effects — CASYMIR does
  not model these
- The task involves photon-counting detectors with adaptive binning, or temporal effects
  such as detector lag and ghosting, which are not yet included
- A quantitative image reconstruction (rather than frequency-domain transfer/noise
  modeling) is the goal
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CASYMIR
  role: Python package implementing the cascaded linear-system model that propagates
    the Signal through process blocks and computes MTF and NNPS
  repo: https://github.com/radboud-axti/casymir_public
- name: NumPy
  role: Core numerical dependency for frequency-domain arrays and Wiener spectrum
    computation
- name: SciPy
  role: Core dependency of CASYMIR for scientific computation within the cascade
- name: PyYAML
  role: Parses the system and detector material YAML configuration files
- name: SpekPy
  role: Handles x-ray spectrum simulation for the Tube and Spectrum objects
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:7dcacb44e0a84d2e509a577c648cfaabc798f2834b56c5ea8cc655901b8e9da9
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
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# frequency-domain-noise-transfer-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Model how x-ray detector gain, blur, sampling, and noise processes transfer signal and noise through a cascaded linear-system chain to produce the frequency-domain MTF and NNPS, using the CASYMIR Python package.

## When to use

Use when you need modeled MTF and NNPS curves for a direct (a-Se) or indirect (CsI) conversion x-ray detector under specified tube, filtration, and acquisition conditions, to compare against measured image-quality baselines.

## When NOT to use

- The analysis requires per-energy-bin propagation of signal and noise, Fano noise, photoconductor discharge, or Lubberts/optical Swank depth effects — CASYMIR does not model these
- The task involves photon-counting detectors with adaptive binning, or temporal effects such as detector lag and ghosting, which are not yet included
- A quantitative image reconstruction (rather than frequency-domain transfer/noise modeling) is the goal

## Inputs

- System YAML configuration file (detector block: type, active_layer, px_size, ff, thickness, trapping_depth, elems, add_noise, extra_materials; source block: target_angle, target, SID, filter, external_filter)
- Detector material YAML from casymir/data/detectors (elemental composition, density, K-fluorescence parameters, conversion gain, blur parameters)
- User-defined filtration material YAML from casymir/data/materials
- Acquisition parameters (view count, angular span, reconstruction filter settings, voxel sampling) set in the calling script

## Outputs

- CSV file with frequency vector (up to detector Nyquist frequency), modeled MTF, and NNPS (standalone mode)
- Compressed .npz outputs containing 3D reconstruction-domain signal transfer, Wiener spectrum, MTF, NNPS, frequency axes, projection angles, and run metadata
- Optional fits to the MTF and NNPS

## How to apply

Load a system YAML (e.g., example_dbt.yaml defining the a-Se detector block with px_size, fill factor, thickness, add_noise, and the W-target source with internal Be/Al and external PMMA filtration) plus the matching detector material file from casymir/data/detectors to build System, Detector, Tube, and Spectrum objects. Propagate the Signal through the parallel-cascaded process blocks: quantum selection, K-fluorescence pathways combined via apply_parallel_process, charge-trapping blur, pixel integration, and noise aliasing. Compute the outputs with the model_output process, where MTF = S_out(f)/S_out(0) and NNPS = W_out(f)/S_out(0)^2 using the aliased Wiener spectrum with added electronic noise. For 2D/3D workflows, run the 1D detector stages first, lift to the detector plane with pixel_integration_2d, apply 2D sampling, log normalization, and reconstruction filters, collect views in a SignalStack, then map into a 3D frequency volume with map_dbt_stack_to_volume (choosing nearest-bin splat=False versus kernel splat=True deposition). In standalone mode CASYMIR writes a CSV containing the frequency vector up to the detector Nyquist frequency, the MTF, and the NNPS.

## Related tools

- **CASYMIR** (Python package implementing the cascaded linear-system model that propagates the Signal through process blocks and computes MTF and NNPS) — https://github.com/radboud-axti/casymir_public
- **NumPy** (Core numerical dependency for frequency-domain arrays and Wiener spectrum computation)
- **SciPy** (Core dependency of CASYMIR for scientific computation within the cascade)
- **PyYAML** (Parses the system and detector material YAML configuration files)
- **SpekPy** (Handles x-ray spectrum simulation for the Tube and Spectrum objects)

## Examples

```
python -m examples.dbt
```

## Evaluation signals

- Modeled MTF RMS percent errors below 6% and NNPS normalized RMS error below 3% for the a-Se DBT detector against measured baselines
- For the CsI bCT detector, RMS percent error in MTF of 5.4% and normalized RMS error for NNPS of 5.8%
- CSV frequency vector extends exactly to the detector's Nyquist frequency (determined by px_size)
- NNPS computed as W_out(f)/S_out(0)^2 uses the aliased Wiener spectrum plus the additive electronic noise specified by add_noise in the YAML
- Standalone run produces a CSV with exactly three components: frequency vector, MTF, and NNPS

## Limitations

- Signal and noise are not propagated on a per-energy-bin basis; absorption-coefficient approximation (total attenuation minus coherent scatter) gives moderate discrepancies in quantum efficiency for low-absorption-efficiency systems, especially at higher energies
- Fano noise, photoconductor discharge, Lubberts effect, and optical Swank noise are not explicitly modeled
- Temporal effects (lag, ghosting), photon-counting detectors with adaptive binning, and input-labeled (input-quantum-limited) formulations are not included
- Repository is in transition from v1 to v2; the v2 public release is expected in August 2026, with the legacy v1 projection-domain model in legacy-v1/

## Evidence

- [methods] Dependencies: [evidence span withheld — anchor status: not_found]
