---
name: yaml-system-configuration-loading
description: Use when a CASYMIR modeling run must be initialized from a system YAML
  file describing detector and source hardware (e.g., a-Se direct conversion with
  W target and Be/Al filtration, or CsI indirect conversion) and the corresponding
  material definitions must be resolved.
when_to_use_negative:
- Acquisition choices such as view count, angular span, reconstruction filter settings,
  and voxel sampling are not set in the YAML — configure those in the calling script/API
  workflow, not by editing the system file
- If the legacy mu_source='XRAYDB' attenuation path is needed, xraydb must be installed
  as an optional extra (pip install -e ".[xraydb]"); mucoeff==1.0.0 is the default
  coefficient source
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CASYMIR
  role: Package whose System/Detector/Tube/Spectrum objects are constructed from the
    parsed YAML configuration
  repo: https://github.com/radboud-axti/casymir_public
- name: PyYAML
  role: Parses the system and detector material YAML files
- name: SpekPy
  role: Underlies x-ray spectrum simulation for the Tube and Spectrum objects; filtration
    materials are defined as SpekPy materials
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:1648065daf432d6bb9d4bef9cc352f12924e2e964adf09f61e5a72eb142ffdcf
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
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# yaml-system-configuration-loading

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load a CASYMIR system YAML file (e.g., example_dbt.yaml or example_bct.yaml) with PyYAML together with the matching detector material definition from casymir/data/detectors to construct the System, Detector, Tube, and Spectrum objects that drive the cascaded linear-system model.

## When to use

Use when a CASYMIR modeling run must be initialized from a system YAML file describing detector and source hardware (e.g., a-Se direct conversion with W target and Be/Al filtration, or CsI indirect conversion) and the corresponding material definitions must be resolved.

## When NOT to use

- Acquisition choices such as view count, angular span, reconstruction filter settings, and voxel sampling are not set in the YAML — configure those in the calling script/API workflow, not by editing the system file
- If the legacy mu_source='XRAYDB' attenuation path is needed, xraydb must be installed as an optional extra (pip install -e ".[xraydb]"); mucoeff==1.0.0 is the default coefficient source

## Inputs

- CASYMIR system YAML file (e.g., example_dbt.yaml, example_bct.yaml, example_cbct.yaml, example_novation_prototype.yaml)
- Detector material YAML from casymir/data/detectors (e.g., a-Se, CsI)
- Filtration material YAML from casymir/data/materials (SpekPy material definitions by formula or fractional elemental composition)

## Outputs

- CASYMIR System object
- CASYMIR Detector object
- CASYMIR Tube object
- CASYMIR Spectrum object

## How to apply

Parse the system YAML (e.g., example_dbt.yaml for the Siemens Mammomat Revelation DBT, or example_bct.yaml for the Koning bCT) using PyYAML; the file contains a system_id, description, a detector block, and a source block. From the detector block read type (direct/indirect), active_layer, px_size, ff (fill factor), thickness in micrometers, trapping_depth, elems (number of frequency samples), add_noise (electronic noise), and extra_materials (cover material/thickness pairs such as Carbon Fiber and Silicon Dioxide). From the source block read target (W/Mo/Rh), target_angle, SID in cm, filter (internal tube filtration, e.g., [(Be, 1.4), (Al, 1.514)]), and external_filter (e.g., [(Al, 10), (Air, 950)]). Resolve the active_layer name to its detector material YAML in casymir/data/detectors — these define elemental composition, density, K-fluorescence parameters, conversion gain, and blur parameters — and resolve any user-defined filtration materials from casymir/data/materials (SpekPy materials by formula or fractional elemental composition). Build the System, Detector, Tube, and Spectrum objects from these parsed blocks so subsequent cascade processes and SpekPy spectrum generation can proceed.

## Related tools

- **CASYMIR** (Package whose System/Detector/Tube/Spectrum objects are constructed from the parsed YAML configuration) — https://github.com/radboud-axti/casymir_public
- **PyYAML** (Parses the system and detector material YAML files)
- **SpekPy** (Underlies x-ray spectrum simulation for the Tube and Spectrum objects; filtration materials are defined as SpekPy materials)

## Examples

```
import yaml; cfg = yaml.safe_load(open('examples/example_dbt.yaml')); det, src = cfg['detector'], cfg['source']  # build System, Detector, Tube, Spectrum
```

## Evaluation signals

- All detector fields (type, active_layer, px_size, ff, thickness, trapping_depth, elems, add_noise, extra_materials) and source fields (target, target_angle, SID, filter, external_filter) are populated without KeyErrors
- active_layer name resolves to an existing YAML in casymir/data/detectors containing elemental composition, density, K-fluorescence parameters, conversion gain, and blur parameters
- Constructed Spectrum matches the specified target/anode angle/filtration (e.g., W target at 10 degrees with Be/Al internal and Al/Air external filtration)
- Frequency grid spans up to the detector Nyquist frequency implied by px_size with the requested number of samples (elems)
- Downstream model_output stage produces MTF = S_out(f)/S_out(0) and NNPS = W_out(f)/S_out(0)^2 consistent with the configured detector

## Limitations

- Repository is in transition from CASYMIR v1 to v2; v2 public code, examples, and documentation expected August 2026 — v1 projection-domain model lives in legacy-v1/
- The YAML describes only source and detector hardware; acquisition choices must be set in the calling script
- The absorption-coefficient approximation (mucoeff default) performs well for high-quantum-efficiency detectors but introduces larger discrepancies in systems with lower absorption efficiency, particularly at higher energies

## Evidence

- [methods] build objects workflow: [evidence span withheld — anchor status: not_found]
- [methods] PyYAML dependency: [evidence span withheld — anchor status: not_found]
