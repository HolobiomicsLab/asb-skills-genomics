---
name: parallel-cascade-signal-propagation
description: Use when modeling an x-ray imaging detector (e.g., a-Se direct conversion
  or CsI indirect conversion) whose resolution/noise behavior must be predicted from
  system YAML configurations and acquisition parameters.
when_to_use_negative:
- Modeling photon-counting detectors or adaptive-binning processes, which are not
  yet included in CASYMIR
- Systems where temporal effects such as detector lag and ghosting, or depth-dependent
  effects (Lubberts effect, optical Swank noise, Fano noise, photoconductor discharge),
  dominate noise propagation—these are not explicitly modeled
- Analyses requiring per-energy-bin propagation of signal and noise quantities or
  input-quantum-limited (input-labeled) formulations
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3399
tools:
- name: CASYMIR
  role: Python package providing the generalized cascaded linear-system model, process
    blocks, apply_parallel_process, and model_output computations
  repo: https://github.com/radboud-axti/casymir_public
- name: Python 3.10
  role: Runtime environment required by CASYMIR
- name: NumPy
  role: Core dependency for frequency-domain arrays and signal computations
- name: SciPy
  role: Core dependency for scientific computation in the cascade model
- name: PyYAML
  role: Parsing of the system and detector material YAML configuration files
- name: SpekPy
  role: X-ray spectrum simulation for the Tube/Spectrum objects and filtration materials
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:04528d0c3f87ce4a31861f3644ea792b86354b050047928ddcaa6863112ecc01
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
  - Python 3.10
  - SciPy
  - SpekPy
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# parallel-cascade-signal-propagation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Propagate x-ray signal and noise through a detector modeled as cascaded gain, blur, sampling, and aliasing process blocks—serial or in parallel—using CASYMIR to compute the MTF and NNPS of direct- or indirect-conversion detectors.

## When to use

Use when modeling an x-ray imaging detector (e.g., a-Se direct conversion or CsI indirect conversion) whose resolution/noise behavior must be predicted from system YAML configurations and acquisition parameters.

## When NOT to use

- Modeling photon-counting detectors or adaptive-binning processes, which are not yet included in CASYMIR
- Systems where temporal effects such as detector lag and ghosting, or depth-dependent effects (Lubberts effect, optical Swank noise, Fano noise, photoconductor discharge), dominate noise propagation—these are not explicitly modeled
- Analyses requiring per-energy-bin propagation of signal and noise quantities or input-quantum-limited (input-labeled) formulations

## Inputs

- System YAML configuration file (e.g., example_dbt.yaml) with detector block (type, active_layer, px_size, ff, thickness, trapping_depth, elems, add_noise) and source block (target, target_angle, SID, filter, external_filter)
- Detector material YAML from casymir/data/detectors (composition, density, K-fluorescence parameters, conversion gain, blur parameters)
- Acquisition parameters (view count, angular span, filter settings, voxel sampling) set in the calling script

## Outputs

- CSV file with frequency vector up to the detector Nyquist frequency, the modeled MTF, and the NNPS
- Optional fits to the MTF and NNPS
- Signal objects at intermediate cascade stages for MTF/NPS inspection

## How to apply

Install CASYMIR (Python 3.10+, dependencies NumPy, SciPy, PyYAML, SpekPy) and load a system YAML such as example_dbt.yaml, which defines the detector block (active layer, pixel pitch, fill factor, thickness, electronic noise) and source block (W target, internal Be/Al filtration, external filtration). Load the detector material definition from casymir/data/detectors and construct the System, Detector, Tube, and Spectrum objects. Propagate the Signal through the a-Se cascade: quantum selection, K-fluorescence pathways combined with apply_parallel_process to build a combined Signal object, charge-trapping blur, pixel integration, and noise aliasing. Compute outputs with the model_output process using MTF = S_out(f)/S_out(0) and NNPS = W_out(f)/S_out(0)^2, with the aliased Wiener spectrum plus additive electronic noise (add_noise). Validate by comparing modeled curves against measured baselines; the paper reports MTF RMS percent errors below 6% and NNPS normalized RMS error below 3% for the a-Se detector.

## Related tools

- **CASYMIR** (Python package providing the generalized cascaded linear-system model, process blocks, apply_parallel_process, and model_output computations) — https://github.com/radboud-axti/casymir_public
- **Python 3.10** (Runtime environment required by CASYMIR)
- **NumPy** (Core dependency for frequency-domain arrays and signal computations)
- **SciPy** (Core dependency for scientific computation in the cascade model)
- **PyYAML** (Parsing of the system and detector material YAML configuration files)
- **SpekPy** (X-ray spectrum simulation for the Tube/Spectrum objects and filtration materials)

## Examples

```
python -m examples.dbt
```

## Evaluation signals

- CSV output contains exactly three columns: frequency vector extending up to the detector Nyquist frequency, MTF, and NNPS
- MTF at f=0 equals 1 by construction (MTF = S_out(f)/S_out(0))
- Validation against measured DBT baselines: MTF RMS percent error below 6% and NNPS normalized RMS error below 3% for the a-Se detector
- Parallel cascade branches (e.g., K-fluorescence pathways) produce a single combined Signal object after apply_parallel_process
- NNPS includes the aliased Wiener spectrum plus the additive electronic noise term (add_noise from the YAML)

## Limitations

- Fano noise and photoconductor discharge are not explicitly modeled
- Signal and noise quantities are not propagated on a per-energy-bin basis; input-quantum-limited (input-labeled) formulations are not included
- Depth-dependent effects (Lubberts effect, optical Swank noise) are not accounted for; detector lag and ghosting are not modeled
- The absorption-coefficient approximation (total attenuation minus coherent scatter) introduces larger discrepancies in low-quantum-efficiency systems, particularly at higher energies
- Repository is in transition from v1 to v2; the v2 public code, examples, and documentation are expected for release in August 2026

## Evidence
