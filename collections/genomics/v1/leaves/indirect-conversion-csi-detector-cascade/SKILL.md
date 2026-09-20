---
name: indirect-conversion-csi-detector-cascade
description: Use when validating or reproducing a modeled MTF/NNPS curve for an indirect-conversion
  CsI flat-panel detector (e.g., Koning bCT), given a system YAML configuration and
  expected experimental transfer curves.
when_to_use_negative:
- The detector is a direct-conversion a-Se device (DM/DBT) — use the direct-conversion
  cascade variant instead of the CsI indirect pipeline
- The system is a photon-counting detector or involves adaptive binning, detector
  lag/ghosting, or Fano noise — CASYMIR does not model these processes
- Per-energy-bin propagation of signal and noise quantities is required — CASYMIR
  does not propagate on a per-energy-bin basis
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: CASYMIR
  role: Python package providing the cascaded linear-system model blocks (quantum
    selection, K-fluorescence, optical blur, pixel integration, noise aliasing) and
    the model_output process for MTF/NNPS
  repo: https://github.com/radboud-axti/casymir_public
- name: SpekPy
  role: Simulates the tungsten-target x-ray spectrum with the Be/Al tube filtration
    and Al external filtration defined in example_bct.yaml
- name: PyYAML
  role: Parses the example_bct.yaml system configuration into System, Detector, Tube,
    and Spectrum objects
- name: NumPy
  role: Frequency-domain computation of S(f), W(f), MTF, and NNPS in the cascade
- name: SciPy
  role: Core dependency used for the frequency-domain transfer and interpolation calculations
    in the cascade
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:0401829f11e9e41861535f0d723b63370a247e8e7f4eaa60e291b4dc3ee57520
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
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# indirect-conversion-csi-detector-cascade

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce the CASYMIR cascaded linear-systems model of a CsI flat-panel dedicated breast CT (bCT) detector by propagating signal and noise through quantum selection, K-fluorescence, optical blur/coupling, pixel integration, and noise-aliasing blocks, then computing MTF and NNPS against experimental baselines (5.4% RMS MTF and 5.8% normalized RMS NNPS error).

## When to use

Use when validating or reproducing a modeled MTF/NNPS curve for an indirect-conversion CsI flat-panel detector (e.g., Koning bCT), given a system YAML configuration and expected experimental transfer curves.

## When NOT to use

- The detector is a direct-conversion a-Se device (DM/DBT) — use the direct-conversion cascade variant instead of the CsI indirect pipeline
- The system is a photon-counting detector or involves adaptive binning, detector lag/ghosting, or Fano noise — CASYMIR does not model these processes
- Per-energy-bin propagation of signal and noise quantities is required — CASYMIR does not propagate on a per-energy-bin basis

## Inputs

- example_bct.yaml system configuration (Koning bCT: CsI indirect detector, px_size 0.1518 mm, ff 0.85, thickness 700 um, add_noise 100)
- W-target x-ray tube spectrum from SpekPy with target_angle 10 deg, SID 95 cm, internal Be/Al filter and external Al/Air filter
- Experimentally measured CsI bCT MTF and NNPS curves for validation comparison

## Outputs

- CSV file with the frequency vector (up to the detector Nyquist frequency), the MTF, and the NNPS
- Per-stage Signal quantities S-bar, S(f), and W(f) through the CsI cascade
- Optional fits to the MTF and NNPS

## How to apply

Load the example_bct.yaml configuration with PyYAML to instantiate the CASYMIR System, Detector, Tube, and Spectrum objects, using SpekPy to simulate the W-target spectrum with Be/Al tube filtration (Be 1.4 mm, Al 1.514 mm) and external Al filtration (Al 10 mm, Air 950 mm). Propagate the Signal through the serial and parallel cascade for the CsI indirect detector — quantum selection, K-fluorescence pathways, optical blurring and coupling, pixel integration, and noise aliasing — computing S-bar, S(f), and W(f) at each stage, combining parallel cascades via apply_parallel_process. Apply the model_output process to compute MTF(f) = Sout(f)/Sout(0) and NNPS(f) = Wout(f)/Sout(0)^2, including the aliased Wiener spectrum and additive electronic noise (add_noise = 100, pixel pitch 0.1518 mm, 700 um CsI thickness). In standalone mode, write a CSV with the frequency vector up to the detector Nyquist frequency, the MTF, and the NNPS. Judge correctness by expert comparison of modeled curves against the experimentally measured CsI bCT baseline; the paper reports 5.4% RMS MTF error and 5.8% normalized RMS NNPS error, with all errors below 6%.

## Related tools

- **CASYMIR** (Python package providing the cascaded linear-system model blocks (quantum selection, K-fluorescence, optical blur, pixel integration, noise aliasing) and the model_output process for MTF/NNPS) — https://github.com/radboud-axti/casymir_public
- **SpekPy** (Simulates the tungsten-target x-ray spectrum with the Be/Al tube filtration and Al external filtration defined in example_bct.yaml)
- **PyYAML** (Parses the example_bct.yaml system configuration into System, Detector, Tube, and Spectrum objects)
- **NumPy** (Frequency-domain computation of S(f), W(f), MTF, and NNPS in the cascade)
- **SciPy** (Core dependency used for the frequency-domain transfer and interpolation calculations in the cascade)

## Examples

```
python -m examples.bct
```

## Evaluation signals

- RMS percent error of modeled MTF vs. measured CsI bCT baseline should be near 5.4% (target < 6%)
- Normalized RMS error of modeled NNPS vs. measured baseline should be near 5.8% (target < 6%)
- Standalone CSV output contains a frequency vector extending to the detector Nyquist frequency (set by px_size 0.1518 mm), plus MTF and NNPS columns
- NNPS includes the aliased Wiener spectrum and additive electronic noise term add_noise = 100
- MTF at f = 0 must equal 1 (MTF(f) = Sout(f)/Sout(0))

## Limitations

- Does not account for depth-dependent effects such as the Lubberts effect or optical Swank noise in the CsI layer
- Fano noise and photoconductor discharge are not explicitly modeled
- Signal and noise are not propagated on a per-energy-bin basis; absorption-coefficient approximation introduces larger discrepancies at higher energies and lower absorption efficiency
- Temporal effects such as detector lag and ghosting are not modeled
- Repository is in transition from v1 to v2; the v2 public code and examples are expected for release in August 2026, with the legacy v1 projection-domain model under legacy-v1/

## Evidence
