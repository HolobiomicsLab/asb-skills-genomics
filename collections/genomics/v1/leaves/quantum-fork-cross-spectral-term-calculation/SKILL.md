---
name: quantum-fork-cross-spectral-term-calculation
description: Use when a detector cascade forks one quantum path into two parallel
  branches (gain/blur stages on each) that are summed downstream, and the combined
  NPS must include cross-terms W_ab between branches per Equation 6.
when_to_use_negative:
- The cascade is purely serial (no fork and recombination), so there are no branch-pair
  cross-spectral terms to compute — a single chain of process blocks suffices
- Signal and noise must be propagated on a per-energy-bin basis, or the detector is
  photon-counting with adaptive binning — CASYMIR does not propagate per energy bin
  and does not include photon-counting processes
- The target physics is dominated by depth-dependent effects such as the Lubberts
  effect or optical Swank noise, which CASYMIR does not account for
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CASYMIR
  role: Provides the Path/Node parallel-cascade structure and the apply_parallel_process
    function that generates the combined Signal object with cross-spectral terms
  repo: https://github.com/radboud-axti/casymir_public
- name: Python
  role: Runtime for the CASYMIR package (Python 3.10 or newer) in which the parallel
    branch combination and model_output evaluation are scripted
  repo: https://github.com/radboud-axti/casymir_public
- name: NumPy
  role: Frequency-domain array arithmetic for the branch transfer functions and cross-spectral
    term evaluation
- name: PyYAML
  role: Loads the system YAML (e.g., example_bct.yaml) describing the CsI detector
    and W source used as baseline for the parallel model
- name: SpekPy
  role: X-ray spectrum simulation for the W target source feeding the forked cascade
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:33e44adc49e788f3031e57099c172b6ce6ea0487f02c0ea9f9156777cc1ea3a8
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# quantum-fork-cross-spectral-term-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the pairwise cross-spectral noise terms that arise when CASYMIR parallel cascade branches (e.g., a quantum fork splitting scintillator-transmitted quanta into a Kim photodiode-conversion path alongside the standard scintillation path) are recombined at a summation point, so that the NNPS of the combined Signal is physically correct.

## When to use

Use when a detector cascade forks one quantum path into two parallel branches (gain/blur stages on each) that are summed downstream, and the combined NPS must include cross-terms W_ab between branches per Equation 6.

## When NOT to use

- The cascade is purely serial (no fork and recombination), so there are no branch-pair cross-spectral terms to compute — a single chain of process blocks suffices
- Signal and noise must be propagated on a per-energy-bin basis, or the detector is photon-counting with adaptive binning — CASYMIR does not propagate per energy bin and does not include photon-counting processes
- The target physics is dominated by depth-dependent effects such as the Lubberts effect or optical Swank noise, which CASYMIR does not account for

## Inputs

- Two or more CASYMIR Signal objects, one per parallel branch (e.g., scintillation path and Kim photodiode-conversion path) sharing a common input Wiener spectrum W_in at the quantum-fork node
- Per-branch gain factors (g_a, g_b) and frequency-domain transfer functions (T_a, T_b) defined on the shared frequency vector up to Nyquist
- System definition YAML (e.g., example_bct.yaml: CsI indirect detector, px_size 0.1518 mm, thickness 700 um, ff 0.85, add_noise 100, W target, SID 95)

## Outputs

- A single combined CASYMIR Signal object containing the summed branch signals and the cross-spectral terms W_ab = g_a g_b T_a T_b* W_in
- MTF and NNPS (with aliased Wiener spectrum and additive electronic noise) from model_output for the extended model
- Comparison table kim_pathway_mtf_nnps_comparison.csv with frequency, baseline and extended MTF/NNPS columns

## How to apply

Define the two branches as CASYMIR Path/Node objects, with a quantum-fork node deterministically routing scintillator-transmitted quanta into the photodiode-conversion path in addition to the standard scintillation path (for the Kim architecture: CsI 700 um active layer, 0.1518 mm pixel, 0.85 fill factor, W source as in example_bct.yaml). Each branch carries its own gain and transfer function; the input Wiener spectrum W_in is shared at the fork. Recombine the branches with apply_parallel_process, which generates the combined Signal object and evaluates the pairwise cross-spectral term W_ab = g_a * g_b * T_a * T_b* * W_in at the summation point instead of summing branch NPSs independently. Then run model_output to obtain MTF = S_out(f)/S_out(0) and NNPS = W_out(f)/S_out(0)^2 (including aliased Wiener spectrum and additive electronic noise) up to Nyquist, and compare against the baseline (no Kim pathway) configuration. Validate by confirming the baseline reproduces the published CsI accuracy envelope: RMS percent MTF error 5.4% and NNPS normalized RMS error 5.8%.

## Related tools

- **CASYMIR** (Provides the Path/Node parallel-cascade structure and the apply_parallel_process function that generates the combined Signal object with cross-spectral terms) — https://github.com/radboud-axti/casymir_public
- **Python** (Runtime for the CASYMIR package (Python 3.10 or newer) in which the parallel branch combination and model_output evaluation are scripted) — https://github.com/radboud-axti/casymir_public
- **NumPy** (Frequency-domain array arithmetic for the branch transfer functions and cross-spectral term evaluation)
- **PyYAML** (Loads the system YAML (e.g., example_bct.yaml) describing the CsI detector and W source used as baseline for the parallel model)
- **SpekPy** (X-ray spectrum simulation for the W target source feeding the forked cascade)

## Examples

```
combined = casymir.processes.apply_parallel_process(signal_a, signal_b)  # quantum-fork recombination with cross-spectral terms
```

## Evaluation signals

- apply_parallel_process returns a single combined Signal object (not separate branch signals), and its NPS equals the sum of branch NPSs plus the cross term g_a g_b T_a T_b* W_in
- Baseline (no Kim pathway) MTF and NNPS reproduce the published CsI accuracy envelope: RMS percent MTF error 5.4% and NNPS normalized RMS error 5.8%
- model_output CSV contains the frequency vector up to the detector's Nyquist frequency together with MTF and NNPS columns
- kim_pathway_mtf_nnps_comparison.csv exists and contains frequency, baseline, and extended MTF/NNPS columns
- Extended-model NNPS differs from the baseline only in frequency regions where the branch transfer functions T_a, T_b are not identical, confirming the cross term is frequency-dependent and properly conjugated

## Limitations

- The Kim detector architecture (x-ray conversion within the photodiode from scintillator-transmitted x rays) is not provided in this CASYMIR version; it must be implemented as a user-defined parallel cascade
- Gain-variability sources such as Fano noise and photoconductor discharge are not explicitly modeled, so the cross-spectral terms exclude these contributions
- CASYMIR does not propagate signal and noise on a per-energy-bin basis, so cross terms are computed for spectrally integrated quantities
- Depth-dependent effects (Lubberts effect, optical Swank noise) are not accounted for in the branch transfer functions

## Evidence
