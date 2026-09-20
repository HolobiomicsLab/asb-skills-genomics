---
name: expectation-maximization-parameter-estimation
description: Use when nanopore eventalign files are available and you need to learn
  per-6-mer emission parameters (Gaussian mixture weights, means, standard deviations)
  for a custom nanoHiMe methylation-calling model.
when_to_use_negative:
- You only need to run modification calling with an existing trained model — use nanoHiMe
  call-6mA-modification or call-CpG-methylation instead of training
- Input events come from unfiltered or low-mapping-quality alignments; the article
  restricts training to reads with mapping quality >=20
- You lack the nanopolish eventalign training file — EM training operates on event-level
  alignments, not raw FAST5 or FASTQ
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0654
tools:
- name: nanoHiMe
  role: Model_training module runs the EM (with Viterbi re-alignment) to fit per-6-mer
    Gaussian emission parameters
  repo: github.com/YinLabTJ/nanoHiMe
- name: nanopolish
  role: eventalign (-n, --scale-events) produces the event-to-k-mer alignment consumed
    by EM training
- name: minimap2
  role: aligns basecalled reads (-ax map-ont) to the reference genome before event
    alignment
- name: samtools
  role: indexes the reference (faidx) and filters training reads to mapped, MQ>=20
    with view -b -q 20 -F 4
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:f8e50ab7bb835451170e4bf199b9c39e3e1b2086d414777d1c48b2e93b7a57ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - minimap2
  - nanoHiMe
  - nanopolish
  - samtools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# expectation-maximization-parameter-estimation

## Summary

Fit Gaussian mixture emission-distribution parameters (omega, mu, sigma) for each 6-mer from nanopore eventalign data using nanoHiMe's EM-based Model_training module, enabling subsequent 6mA and CpG methylation calling with a hidden Markov model.

## When to use

Use when nanopore eventalign files are available and you need to learn per-6-mer emission parameters (Gaussian mixture weights, means, standard deviations) for a custom nanoHiMe methylation-calling model.

## When NOT to use

- You only need to run modification calling with an existing trained model — use nanoHiMe call-6mA-modification or call-CpG-methylation instead of training
- Input events come from unfiltered or low-mapping-quality alignments; the article restricts training to reads with mapping quality >=20
- You lack the nanopolish eventalign training file — EM training operates on event-level alignments, not raw FAST5 or FASTQ

## Inputs

- nanopolish eventalign output file (test.eventalign.txt, optionally .gz) from reads aligned with mapping quality >=20
- Uppercase reference FASTA with samtools faidx index (e.g., hg19.fa or E.coli_K12_MG1655.fasta)
- EM initial parameter file (EM_shift.txt, default nanoHiMe/perl_script/EM_shift.txt)

## Outputs

- EM.out file with one row per 6-mer listing omega1-3, mu1-3, and sigma1-3 for the three Gaussian components (e.g., AAAAAA: omega1 0.9255, mu1 86.4534, sigma1 1.2128)

## How to apply

Prepare training data by converting the reference FASTA to uppercase (nanoHiMe upper.pl), indexing it with samtools faidx, aligning basecalled reads with minimap2 -ax map-ont, filtering to mapped reads with mapping quality >=20 via samtools view -b -q 20 -F 4, and producing an event-level alignment with nanopolish eventalign (-n, --scale-events). Then run Model_training with -m mcg (or ma/ma_mcg), -c 5 re-alignment cycles in which events are iteratively re-assigned to k-mers via the Viterbi algorithm, -g 3 Gaussians per 6-mer, -p EM_shift.txt as the EM initial-value file, and -i 500 maximum EM iterations. The EM algorithm fits the mixture weights (omega_j), means (mu_j), and standard deviations (sigma_j) of the current-level signal for each 6-mer. Because partial adenine methylation makes it impossible to precisely assign learned parameters to 6-mers with 6mA at different positions, such k-mers are collapsed into an averaged k-mer (k-bar) to which the learned parameters are assigned. The output EM.out contains one row per 6-mer with omega1-3, mu1-3, sigma1-3 and can be validated against reference values (e.g., AAAAAA: omega1 ~0.926, mu1 ~86.45, sigma1 ~1.21).

## Related tools

- **nanoHiMe** (Model_training module runs the EM (with Viterbi re-alignment) to fit per-6-mer Gaussian emission parameters) — github.com/YinLabTJ/nanoHiMe
- **nanopolish** (eventalign (-n, --scale-events) produces the event-to-k-mer alignment consumed by EM training)
- **minimap2** (aligns basecalled reads (-ax map-ont) to the reference genome before event alignment)
- **samtools** (indexes the reference (faidx) and filters training reads to mapped, MQ>=20 with view -b -q 20 -F 4)

## Examples

```
Model_training -m mcg -c 5 -r /.../ref_upper.fa -e /.../training.eventalign.txt.gz -o /.../output_dir -g 3 -p /.../EM_shift.txt -i 500
```

## Evaluation signals

- EM.out exists and contains a row for every expected 6-mer with omega, mu, and sigma values for all three Gaussian components
- AAAAAA row matches reference training output (omega1 ~0.926, mu1 ~86.45, sigma1 ~1.21)
- Mixture weights within each 6-mer row sum to approximately 1 (e.g., AAAAAA: 0.9255 + 0.0372 + 0.0372)
- Re-alignment completed for the requested -c 5 cycles and EM ran up to the -i 500 iteration cap without errors
- Input eventalign training file was derived only from reads with mapping quality >=20 (samtools view -q 20 -F 4)

## Limitations

- Partial adenine methylation prevents precise assignment of learned parameters to k-mers with 6mA at different positions; such k-mers are collapsed to an averaged k-mer (k-bar), so the model recognizes 6mA-containing vs non-6mA sites but cannot predict 6mA at base resolution
- Training used PCR-amplified E. coli K12 MG1655 datasets (untreated, M.SssI-, Hia5-, or both-treated); models for other modification combinations (5-hmC, 5-fC, 4-mC, GpC methylation) do not yet exist
- 6mA calling on the whole genome without an input.peak.bed restriction may be quite slow

## Evidence

- [discussion] To use the expectation-maximization algorithm to fit ωj, μj, σj: "To use the expectation-maximization algorithm to fit ωj, μj, σj"
- [discussion] We iterated the process for 5 times and assigned the parameters from the final iteration: "We iterated the process for 5 times and assigned the parameters from the final iteration"
- [discussion] As the partial adenine methylation made it impossible to precisely assign the learned parameters to k-mers: "As the partial adenine methylation made it impossible to precisely assign the learned parameters to k-mers"
- [readme] The output file EM.out will contain the parameters of each gaussian distribution for each 6-mer: "The output file EM.out will contain the parameters of each gaussian distribution for each 6-mer"
- [discussion] For the reads with mapping quality ≥20, we performed initial alignment between events: "For the reads with mapping quality ≥20, we performed initial alignment between events"
