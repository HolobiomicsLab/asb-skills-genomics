---
name: parallel-cascade-cross-spectral-term-computation
description: Use when modeling a detector process whose quanta split or fork into
  parallel paths (e.g., K-fluorescence escape in CsI or charge trapping in a-Se) and
  signal/noise must be recombined with correct cross-spectral terms before computing
  MTF/NNPS.
when_to_use_negative:
- The process is strictly serial with no branching (e.g., pure pixel integration)
  — apply it as a single serial cascade block instead
- Fano noise or photoconductor discharge gain variability must be modeled — these
  are not explicitly handled in CASYMIR's current parallel cascade
- Photon-counting detectors with adaptive binning — such architectures are not yet
  included in CASYMIR
edam_operation: null
tools:
- name: CASYMIR
  role: Provides Path, Node, Signal, and apply_parallel_process for constructing and
    combining parallel cascades with cross-spectral terms
  repo: https://github.com/radboud-axti/casymir_public
- name: Python
  role: Runtime environment; CASYMIR requires Python 3.10 or newer
  repo: https://github.com/radboud-axti/casymir_public
- name: NumPy
  role: Array operations on frequency-domain S(f) and W(f) vectors during parallel
    path combination
- name: PyYAML
  role: Parses system/detector YAML (e.g., example_bct.yaml) to populate detector
    and K-fluorescence parameters for the cascade
- name: SpekPy
  role: X-ray spectrum simulation feeding the cascade's quantum selection stage
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/mp.18079
    title: 'Software Article: A generalized cascaded linear system model implementation
      for x‐ray detectors'
  source_content_hash: sha256:4e8ace60b636ff5c35d7f4d0f34b94097cacc432da79742e824826dbe2d02e26
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
yes_rate: 0.9285714285714286
n_claims: 14
---

# parallel-cascade-cross-spectral-term-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combines the branching stages of a cascaded linear-systems x-ray detector model (e.g., K-fluorescence reabsorption in CsI) using CASYMIR Path/Node objects and apply_parallel_process, computing pairwise cross-spectral terms so that MTF and NPS can be evaluated at any stage of the cascade.

## When to use

Use when modeling a detector process whose quanta split or fork into parallel paths (e.g., K-fluorescence escape in CsI or charge trapping in a-Se) and signal/noise must be recombined with correct cross-spectral terms before computing MTF/NNPS.

## When NOT to use

- The process is strictly serial with no branching (e.g., pure pixel integration) — apply it as a single serial cascade block instead
- Fano noise or photoconductor discharge gain variability must be modeled — these are not explicitly handled in CASYMIR's current parallel cascade
- Photon-counting detectors with adaptive binning — such architectures are not yet included in CASYMIR

## Inputs

- CASYMIR Signal objects carrying mean signal (S̄), signal transfer S(f), and Wiener spectrum W(f) at the stage preceding the parallel split
- CASYMIR Path and Node objects defining each parallel branch (e.g., primary photons vs. K-fluorescence reabsorption paths)
- Detector material YAML from casymir/data/detectors defining K-fluorescence parameters, conversion gain, and blur parameters for CsI or a-Se
- System configuration YAML (e.g., example_bct.yaml: CsI, 700 µm thickness, 0.1518 mm pixel pitch)

## Outputs

- A combined CASYMIR Signal object with summed S̄, S(f), and W(f) including pairwise cross-spectral terms
- Stage-wise MTF(f) and NNPS(f) values available after the parallel block and at model output (via model_output)

## How to apply

Represent each branch of the parallel stage as a CASYMIR Path containing Node objects carrying gain, blur, and noise processes operating on Signal objects (S̄, S(f), W(f)). Call apply_parallel_process to generate the combined Signal object, computing cross-spectral terms pairwise: K_αβ = −ᾱ(1−ᾱ) for Bernoulli-split processes (Equation 5) and K_αβ = 0 for quantum-fork processes (Equation 6), then sum path contributions with Equations 1–3. Place the parallel cascade at the correct point in the serial 1D chain (after quantum selection and energy conversion, before pixel integration), since downstream MTF(f) = S_out(f)/S_out(0) and NNPS(f) = W_out(f)/S_out(0)² inherit the combined statistics. The same block style extends into 2D/3D via pixel_integration_2d and SignalStack collection. The rationale is that ignoring negative cross-spectral correlations from the Bernoulli split would overestimate NPS in scintillator-based detectors.

## Related tools

- **CASYMIR** (Provides Path, Node, Signal, and apply_parallel_process for constructing and combining parallel cascades with cross-spectral terms) — https://github.com/radboud-axti/casymir_public
- **Python** (Runtime environment; CASYMIR requires Python 3.10 or newer) — https://github.com/radboud-axti/casymir_public
- **NumPy** (Array operations on frequency-domain S(f) and W(f) vectors during parallel path combination)
- **PyYAML** (Parses system/detector YAML (e.g., example_bct.yaml) to populate detector and K-fluorescence parameters for the cascade)
- **SpekPy** (X-ray spectrum simulation feeding the cascade's quantum selection stage)

## Examples

```
python -m examples.bct
```

## Evaluation signals

- Propagated Wiener spectra W(f) remain real-valued at every frequency sample
- MTF(0) = 1 at the model output after parallel recombination and pixel integration
- Modeled MTF/NNPS reproduce reported accuracy: RMS percent error in MTF and normalized RMS error in NNPS all below 6% (a-Se: MTF < 6%, NNPS < 3%; CsI: MTF 5.4%, NNPS 5.8%)
- Negative correlation enforced for Bernoulli-split branches (K_αβ = −ᾱ(1−ᾱ)) and zero correlation for quantum forks (K_αβ = 0), per Equations 5–6
- Frequency vector in the standalone CSV output extends exactly to the detector's Nyquist frequency

## Limitations

- Fano noise and photoconductor discharge gain variability are not explicitly modeled
- Depth-dependent effects such as the Lubberts effect and optical Swank noise are not accounted for
- Signal and noise are not propagated on a per-energy-bin basis
- Temporal effects (detector lag, ghosting) and photon-counting architectures with adaptive binning are not included

## Evidence
