---
name: parallel-cascade-modeling
description: Use when modeling an x-ray detector whose cascade splits into parallel
  pathways — e.g., the Kim architecture where scintillator-transmitted x rays convert
  within the photodiode — and you need combined MTF/NNPS with cross-spectral noise
  terms.
when_to_use_negative:
- The detector cascade is purely serial with no branching — use CASYMIR's standard
  serial process blocks instead of constructing parallel Paths
- The detector is a photon-counting system with adaptive binning, or temporal effects
  such as detector lag and ghosting must be modeled — these are not included in CASYMIR
- Per-energy-bin propagation or input-labeled (input-quantum-limited) formulations
  are required — CASYMIR does not propagate signal and noise quantities on a per-energy-bin
  basis
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: CASYMIR
  role: Provides the Path and Node objects, the quantum-fork routing, apply_parallel_process
    for combining branches with cross-spectral terms, and the model_output process
    for MTF/NNPS
  repo: https://github.com/radboud-axti/casymir_public
- name: Python
  role: Runtime environment (Python 3.10 or newer) in which the parallel cascade is
    defined and executed
- name: NumPy
  role: Frequency-domain arrays for signal, NPS/Wiener spectrum, and cross-spectral
    term computation
- name: SciPy
  role: Core scientific dependency of CASYMIR used in the frequency-domain process
    implementations
- name: PyYAML
  role: Parses the system YAML (detector and source blocks) that seeds the baseline
    cascade
- name: SpekPy
  role: Generates the incident x-ray spectrum (e.g., W target with Be/Al filtration)
    driving the cascade
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:3cd52c36184e5f8ea363252f9bfb939590a9b5371f7367d32313ac728224a553
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
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# parallel-cascade-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs parallel cascades within CASYMIR by defining custom Path/Node branches (e.g., a Kim-style photodiode-conversion pathway alongside the standard scintillation path) and combining them with apply_parallel_process, which accounts for pairwise cross-spectral noise terms at the summation point before computing MTF and NNPS.

## When to use

Use when modeling an x-ray detector whose cascade splits into parallel pathways — e.g., the Kim architecture where scintillator-transmitted x rays convert within the photodiode — and you need combined MTF/NNPS with cross-spectral noise terms.

## When NOT to use

- The detector cascade is purely serial with no branching — use CASYMIR's standard serial process blocks instead of constructing parallel Paths
- The detector is a photon-counting system with adaptive binning, or temporal effects such as detector lag and ghosting must be modeled — these are not included in CASYMIR
- Per-energy-bin propagation or input-labeled (input-quantum-limited) formulations are required — CASYMIR does not propagate signal and noise quantities on a per-energy-bin basis

## Inputs

- System YAML file (e.g., example_bct.yaml) with detector block (type: indirect, active_layer: CsI, px_size: 0.1518, ff: 0.85, thickness: 700, elems: 256, add_noise: 100) and source block (W target, target_angle: 10, SID: 95, Be/Al filtration)
- X-ray spectrum simulated via SpekPy for the tube target, angle, and filtration settings
- Custom parallel process definition: Path and Node objects implementing the alternate conversion pathway (e.g., photodiode conversion of transmitted x rays)
- Frequency vector spanning 0 to the detector's Nyquist frequency with elems frequency samples

## Outputs

- Combined Signal object from apply_parallel_process containing signal and noise propagated through both branches
- MTF and NNPS arrays (up to Nyquist frequency) for baseline and extended configurations via the model_output process
- Comparison CSV (e.g., kim_pathway_mtf_nnps_comparison.csv) with frequency, baseline MTF/NNPS, and extended MTF/NNPS columns
- Optionally .npz outputs in examples/outputs/ with 3D reconstruction-domain arrays, signal transfer, Wiener spectrum, MTF, NNPS, frequency axes, and run metadata

## How to apply

Load the detector system from its YAML description (e.g., example_bct.yaml with CsI 700 um active layer, 0.1518 mm pixel pitch, 0.85 fill factor, W target) and run the baseline 1D cascade to get reference MTF/NNPS via the model_output process. Define the parallel extension as a branch built from CASYMIR Path and Node objects, inserting a quantum-fork node that deterministically routes scintillator-transmitted quanta into the photodiode-conversion path in addition to the scintillation path. Combine the branches with apply_parallel_process, which generates the combined Signal object and computes pairwise cross-spectral terms at the summation point (W_ab = g_a g_b T_a T_b* W_in per Equation 6). Compute MTF = S_out(f)/S_out(0) and NNPS = W_out(f)/S_out(0)^2 — including the aliased Wiener spectrum and additive electronic noise (add_noise field, e.g., 100) — up to the detector's Nyquist frequency using the model_output process. Validate by confirming the baseline reproduces the reported accuracy envelope (RMS percent MTF error 5.4% and NNPS normalized RMS error 5.8% for the CsI bCT detector) and by tabulating baseline vs. extended MTF/NNPS per frequency.

## Related tools

- **CASYMIR** (Provides the Path and Node objects, the quantum-fork routing, apply_parallel_process for combining branches with cross-spectral terms, and the model_output process for MTF/NNPS) — https://github.com/radboud-axti/casymir_public
- **Python** (Runtime environment (Python 3.10 or newer) in which the parallel cascade is defined and executed)
- **NumPy** (Frequency-domain arrays for signal, NPS/Wiener spectrum, and cross-spectral term computation)
- **SciPy** (Core scientific dependency of CASYMIR used in the frequency-domain process implementations)
- **PyYAML** (Parses the system YAML (detector and source blocks) that seeds the baseline cascade)
- **SpekPy** (Generates the incident x-ray spectrum (e.g., W target with Be/Al filtration) driving the cascade)

## Examples

```
python -m examples.bct
```

## Evaluation signals

- apply_parallel_process returns a combined Signal whose summation point includes pairwise cross-spectral terms W_ab = g_a g_b T_a T_b* W_in (Equation 6); omitting these terms is a red flag
- MTF is computed as S_out(f)/S_out(0) and NNPS as W_out(f)/S_out(0)^2, including the aliased Wiener spectrum and additive electronic noise, and terminates at the detector's Nyquist frequency
- Baseline (no parallel pathway) MTF/NNPS reproduce the reported validation envelope: RMS percent MTF error 5.4% and NNPS normalized RMS error 5.8% for the CsI bCT detector
- The comparison CSV exists and contains frequency, baseline, and extended MTF/NNPS columns
- The quantum-fork node deterministically routes scintillator-transmitted quanta into the photodiode path without altering the baseline scintillation branch

## Limitations

- Fano noise, photoconductor discharge, the Lubberts effect, and depth-dependent optical gain variations (optical Swank noise) are not explicitly modeled, so parallel-cascade results omit these variance sources
- The absorption-coefficient approximation (total attenuation minus coherent scatter) introduces larger discrepancies in low-quantum-efficiency systems, particularly at higher energies
- The Kim architecture is not provided out of the box — it must be implemented as a user-defined custom parallel cascade; also, the public repository is in transition from v1 to v2, with the v2 public release expected August 2026 and v1 available under legacy-v1/

## Evidence
