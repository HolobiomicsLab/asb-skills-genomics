---
name: yaml-system-configuration
description: 'Use when configuring a CASYMIR simulation of a direct (a-Se) or indirect
  (CsI) x-ray detector: you need to translate hardware specifications (pixel pitch,
  fill factor, active-layer thickness, tube target/anode angle, filtration) into the
  package''s YAML schema before running any cascade or MTF/NNPS.'
when_to_use_negative:
- Acquisition-level settings such as view count, angular span, reconstruction filter
  settings, or voxel sampling are being configured — these belong in the calling script
  or API workflow, not the system YAML
- The detector material is not represented by a YAML name in casymir/data/detectors
  and no casymir/data/materials definition can be written for it
- The task requires modeling phenomena not captured by the schema (e.g. photon-counting
  adaptive binning, detector lag/ghosting, Lubberts effect, per-energy-bin propagation)
edam_operation: http://edamontology.org/operation_0227
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CASYMIR
  role: Package whose YAML schema (detector and source blocks) and data directories
    (casymir/data/detectors, casymir/data/materials) this skill populates
  repo: https://github.com/radboud-axti/casymir_public
- name: PyYAML
  role: Parses the system YAML file to populate the System, Detector, Tube, and Spectrum
    objects
- name: SpekPy
  role: Underlying x-ray spectrum engine referenced by the source block filtration
    materials (filter, external_filter entries defined as SpekPy materials)
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:4f5c7e0a739e925bba89d9af21bdd59fd1e02edb50fc1324019c34178dd3eac3
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
  edam_operation: http://edamontology.org/operation_0227
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# yaml-system-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Author and parse CASYMIR system YAML files (e.g., example_bct.yaml) that define detector hardware (material, pixel pitch, fill factor, thickness, additive noise) and x-ray source parameters (target, anode angle, SID, filtration) to populate the System, Detector, Tube, and Spectrum objects used by the cascaded linear-system model.

## When to use

Use when configuring a CASYMIR simulation of a direct (a-Se) or indirect (CsI) x-ray detector: you need to translate hardware specifications (pixel pitch, fill factor, active-layer thickness, tube target/anode angle, filtration) into the package's YAML schema before running any cascade or MTF/NNPS computation.

## When NOT to use

- Acquisition-level settings such as view count, angular span, reconstruction filter settings, or voxel sampling are being configured — these belong in the calling script or API workflow, not the system YAML
- The detector material is not represented by a YAML name in casymir/data/detectors and no casymir/data/materials definition can be written for it
- The task requires modeling phenomena not captured by the schema (e.g. photon-counting adaptive binning, detector lag/ghosting, Lubberts effect, per-energy-bin propagation)

## Inputs

- System YAML file with system_id, description, detector, and source blocks (e.g. example_bct.yaml: CsI 700 um, px_size 0.1518 mm, ff 0.85, W target, 10 deg anode angle, SID 95 cm)
- Detector material YAML files from casymir/data/detectors (elemental composition, density, K-fluorescence parameters, conversion gain, blur parameters for a-Se and CsI)
- User-defined filtration material YAML files from casymir/data/materials (SpekPy materials by chemical formula or fractional elemental composition)

## Outputs

- Populated CASYMIR System, Detector, Tube, and Spectrum objects parsed from the YAML via PyYAML
- Validated stage-wise parameter values matching the YAML configuration (verified against reported MTF/NNPS accuracy, e.g. 5.4% RMS MTF error and 5.8% NNPS normalized RMS error for the CsI detector)

## How to apply

Write (or adapt an example such as example_bct.yaml or example_dbt.yaml) with four blocks: system_id, description, detector, and source. In the detector block set type (direct or indirect), active_layer (a material YAML name in casymir/data/detectors), px_size in mm, ff (fill factor), thickness in micrometers, trapping_depth for direct converters, elems (number of frequency samples), add_noise (additive electronic noise), and extra_materials (cover materials as material/thickness pairs such as (Carbon Fiber, 2.5)). In the source block set target_angle (anode angle in degrees), target (W, Mo, or Rh), SID in cm, filter (internal tube filtration, e.g. [(Be, 1.4), (Al, 1.514)]), and external_filter (filtration between tube and detector, e.g. [(Al, 10), (Air, 950)]). Load the file with PyYAML to populate the System, Detector, Tube, and Spectrum objects, and validate the parsed stage-wise parameters against the intended hardware specification. Acquisition choices (view count, angular span, reconstruction filters, voxel sampling) are deliberately not in the YAML and must be set in the calling script or API workflow.

## Related tools

- **CASYMIR** (Package whose YAML schema (detector and source blocks) and data directories (casymir/data/detectors, casymir/data/materials) this skill populates) — https://github.com/radboud-axti/casymir_public
- **PyYAML** (Parses the system YAML file to populate the System, Detector, Tube, and Spectrum objects)
- **SpekPy** (Underlying x-ray spectrum engine referenced by the source block filtration materials (filter, external_filter entries defined as SpekPy materials))

## Examples

```
python -m pip install -e . ; python -m examples.bct  # example_bct.yaml defines the Koning bCT detector/source for the baseline cascade
```

## Evaluation signals

- YAML file parses cleanly with PyYAML and contains the four required blocks: system_id, description, detector, source
- Detector fields satisfy the schema: type in {direct, indirect}, px_size in mm, thickness in micrometers, trapping_depth set for direct converters (0 for indirect), elems and add_noise present
- Source fields are physically consistent: target in {W, Mo, Rh}, target_angle in degrees, SID in cm, filter/external_filter given as material/thickness pairs
- Parsed parameter values match the intended hardware spec (e.g. example_bct.yaml reproduces Koning bCT: CsI 700 um, 0.1518 mm pitch, 0.85 fill factor, W target, 10 deg angle, 95 cm SID)
- Downstream baseline run reproduces published MTF/NNPS within the reported accuracy envelope (below 6% RMS MTF error; 5.4%/5.8% for the CsI detector, below 3% NNPS normalized RMS error for a-Se)

## Limitations

- Acquisition choices such as view count, angular span, reconstruction filter settings, and voxel sampling are not set in the YAML and must be set in the calling script or API workflow
- The repository is transitioning from CASYMIR v1 to v2, with the public v2 code, examples, and documentation expected after August 2026; the v1 projection-domain implementation is under legacy-v1/
- xraydb is no longer a required dependency — the legacy mu_source="XRAYDB" comparison path needs the optional [xraydb] extra; mucoeff==1.0.0 is the default attenuation-coefficient source
- The YAML schema cannot express temporal effects (lag, ghosting), photon-counting processes, or depth-dependent effects such as the Lubberts effect and optical Swank noise, which CASYMIR does not model

## Evidence
