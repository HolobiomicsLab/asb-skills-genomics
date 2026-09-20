---
name: yaml-configuration-driven-detector-modeling
description: Use when reproducing or configuring a modeled x-ray detector system (e.g.,
  CsI bCT, a-Se DBT) in CASYMIR where detector and source hardware parameters must
  be loaded from a YAML file before running the cascade.
when_to_use_negative:
- When acquisition choices (view count, angular span, reconstruction filter settings,
  voxel sampling) must be varied — these are set in the calling script or API workflow,
  not in the system YAML
- When the detector architecture requires processes not covered by the YAML schema,
  such as photon-counting detectors with adaptive binning, or Kim's scintillator-transmitted
  architecture (custom parallel cascades are needed instead)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CASYMIR
  role: Python package whose System, Detector, Tube, and Spectrum objects are instantiated
    from the YAML system configuration and whose cascade computes MTF/NNPS
  repo: https://github.com/radboud-axti/casymir_public
- name: PyYAML
  role: Parses the example_bct.yaml system configuration into the CASYMIR object model
- name: SpekPy
  role: Simulates the tungsten-target x-ray spectrum with the Be/Al tube filtration
    and Al external filtration defined in the source block
- name: Python
  role: Runtime environment (Python 3.10 or newer required) in which the YAML-driven
    workflow executes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:2d1aaabf724d50a461ab6eb84abf1514573e1f3a8f5c87fa481eda99c0e4df35
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# yaml-configuration-driven-detector-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Instantiate a CASYMIR x-ray detector system model from a declarative YAML configuration (e.g., example_bct.yaml) that specifies detector hardware (CsI, pixel pitch, fill factor, additive noise) and source parameters (W target, tube filtration), enabling reproducible MTF/NNPS modeling without hard-coded parameters.

## When to use

Use when reproducing or configuring a modeled x-ray detector system (e.g., CsI bCT, a-Se DBT) in CASYMIR where detector and source hardware parameters must be loaded from a YAML file before running the cascade.

## When NOT to use

- When acquisition choices (view count, angular span, reconstruction filter settings, voxel sampling) must be varied — these are set in the calling script or API workflow, not in the system YAML
- When the detector architecture requires processes not covered by the YAML schema, such as photon-counting detectors with adaptive binning, or Kim's scintillator-transmitted architecture (custom parallel cascades are needed instead)

## Inputs

- System configuration YAML file (e.g., example_bct.yaml, example_dbt.yaml, example_cbct.yaml, example_novation_prototype.yaml)
- Detector material YAML definitions from casymir/data/detectors (elemental composition, density, K-fluorescence parameters, conversion gain, blur parameters)
- User-defined filtration material YAML files from casymir/data/materials (SpekPy materials by formula or fractional elemental composition)

## Outputs

- Instantiated CASYMIR System, Detector, Tube, and Spectrum objects
- Standalone-mode CSV output with the frequency vector up to the detector Nyquist frequency, the MTF, and the NNPS
- For example scripts: compressed .npz outputs in examples/outputs/ with signal transfer, Wiener spectrum, MTF, NNPS, frequency axes, projection angles, and run metadata

## How to apply

Write or load a system YAML file containing a system_id, description, a detector block, and a source block, and parse it with PyYAML to instantiate the CASYMIR System, Detector, Tube, and Spectrum objects. The detector block specifies type (direct/indirect), active_layer (a material YAML in casymir/data/detectors), px_size (mm), ff (fill factor), thickness (µm), trapping_depth, elems (number of frequency samples), add_noise (electronic noise), and extra_materials (cover material/thickness pairs). The source block specifies target_angle, target (W/Mo/Rh), SID (cm), filter (internal tube filtration), and external_filter; the tungsten spectrum with Be/Al filtration is simulated with SpekPy. For the CsI bCT example this is: type=indirect, active_layer=CsI, px_size=0.1518, ff=0.85, thickness=700, elems=256, add_noise=100, extra_materials=[(Carbon Fiber, 2.5), (Silicon Dioxide, 1)], with source target_angle=10, target=W, SID=95, filter=[(Be, 1.4), (Al, 1.514)], external_filter=[(Al, 10), (Air, 950)]. After configuration, propagate the Signal through the serial and parallel cascade process blocks and apply the model_output process to compute MTF and NNPS.

## Related tools

- **CASYMIR** (Python package whose System, Detector, Tube, and Spectrum objects are instantiated from the YAML system configuration and whose cascade computes MTF/NNPS) — https://github.com/radboud-axti/casymir_public
- **PyYAML** (Parses the example_bct.yaml system configuration into the CASYMIR object model)
- **SpekPy** (Simulates the tungsten-target x-ray spectrum with the Be/Al tube filtration and Al external filtration defined in the source block)
- **Python** (Runtime environment (Python 3.10 or newer required) in which the YAML-driven workflow executes)

## Examples

```
python -m examples.bct  # loads example_bct.yaml (CsI, px_size=0.1518, add_noise=100; W target, Be/Al filtration) and writes outputs to examples/outputs/
```

## Evaluation signals

- For the CsI bCT configuration, modeled MTF should agree with experimental measurements within a 5.4% RMS percent error and NNPS within a 5.8% normalized RMS error
- The generated CSV must contain three aligned columns: frequency vector extending to the detector Nyquist frequency, MTF, and NNPS
- All required YAML fields must parse: detector block fields (type, active_layer, px_size, ff, thickness, elems, add_noise) and source block fields (target_angle, target, SID, filter, external_filter)
- MTF at zero frequency should normalize to 1 (MTF(f) = Sout(f)/Sout(0)), and NNPS should equal Wout(f)/Sout(0)^2 including the aliased Wiener spectrum and additive electronic noise

## Limitations

- The public repository is in transition from CASYMIR v1 to v2, with v2 code, examples, and documentation expected for public release in August 2026; the reported v1 projection-domain model is under legacy-v1/
- CASYMIR does not propagate signal and noise on a per-energy-bin basis and does not explicitly model Fano noise, photoconductor discharge, Lubberts effect, optical Swank noise, detector lag/ghosting, or photon-counting processes
- The absorption-coefficient approximation (total attenuation minus coherent scatter) introduces larger discrepancies in systems with lower absorption efficiency, particularly at higher energies

## Evidence

- [methods] CASYMIR relies on standard libraries such as NumPy, SciPy, and PyYAML, as well as SpekPy and XrayDB for handling x-ray spectrum simulations: [evidence span withheld — anchor status: not_found]
