---
name: kmer-emission-gaussian-mixture-training
description: Use when you need to train a custom nanoHiMe emission model from nanopore
  event-level alignments of methylated-control DNA (e.g. PCR-amplified E. coli K12
  MG1655 treated with M.SssI and/or Hia5) before running mCG or 6mA calling.
when_to_use_negative:
- You only need methylation calling with the pre-trained nanoHiMe model — no custom
  training eventalign data is available.
- 'Your training DNA has partially methylated adenines at heterogeneous positions
  and you need base-resolution 6mA prediction: the k-bar collapsing gives 6mA-containing
  vs non-6mA k-mer recognition but not per-position 6mA calls.'
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0654
tools:
- name: nanoHiMe
  role: Runs Model_training to fit per-6-mer Gaussian mixture emission parameters
    via EM and event re-alignment
  repo: github.com/YinLabTJ/nanoHiMe
- name: nanopolish
  role: Produces the event-level alignment (eventalign -n --scale-events) used as
    training input
- name: minimap2
  role: Aligns basecalled reads to the reference with -ax map-ont prior to event alignment
- name: samtools
  role: Indexes the reference (faidx) and filters reads to mapped, MAPQ >=20 (-b -q
    20 -F 4)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:b2242d6320f6d0cdc99491839fc90437b6096598bf403d4f49993aa6d63d76aa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - minimap2
  - nanoHiMe
  - nanopolish
  - samtools
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# kmer-emission-gaussian-mixture-training

## Summary

Trains nanoHiMe's per-6-mer emission-distribution parameters (Gaussian mixture weights omega, means mu, sigmas sigma) via expectation-maximization on nanopolish eventalign data, enabling the HMM to distinguish methylated CpG and 6mA contexts from nanopore signals.

## When to use

Use when you need to train a custom nanoHiMe emission model from nanopore event-level alignments of methylated-control DNA (e.g. PCR-amplified E. coli K12 MG1655 treated with M.SssI and/or Hia5) before running mCG or 6mA calling.

## When NOT to use

- You only need methylation calling with the pre-trained nanoHiMe model — no custom training eventalign data is available.
- Your training DNA has partially methylated adenines at heterogeneous positions and you need base-resolution 6mA prediction: the k-bar collapsing gives 6mA-containing vs non-6mA k-mer recognition but not per-position 6mA calls.

## Inputs

- nanopolish eventalign file (test.eventalign.txt or .gz, produced with -n and --scale-events)
- Reference FASTA (e.g. ref_upper.fa or E.coli_K12_MG1655.fasta) with samtools faidx index
- Initial EM parameter file (EM_shift.txt)
- Mapping-quality >=20 filtered BAM (test.sorted.q20.mapped.bam)

## Outputs

- EM.out: per-6-mer table of omega1/mu1/sigma1, omega2/mu2/sigma2, omega3/mu3/sigma3 Gaussian mixture parameters
- Trained model used by nanoHiMe mCG and 6mA calling modes

## How to apply

First prepare the training eventalign file: uppercase the reference (nanoHiMe upper.pl), samtools faidx index it, align basecalled reads with minimap2 -ax map-ont, filter to mapped reads with mapping quality >=20 (samtools view -b -q 20 -F 4), and run nanopolish eventalign with -n and --scale-events. Then run Model_training with mode -m mcg (or ma/ma_mcg), 5 re-alignment cycles (-c 5), 3 Gaussians per 6-mer (-g 3), 500 EM iterations (-i 500), initializing from the EM_shift.txt parameter file. EM fits omega_j, mu_j, sigma_j per 6-mer, re-assigning events to k-mers via the Viterbi algorithm for 5 iterations and keeping the parameters from the final iteration. For 6mA training, k-mers with adenine methylation at different positions cannot be assigned precisely, so they are grouped into a k-bar k-mer and assigned the learned parameters.

## Related tools

- **nanoHiMe** (Runs Model_training to fit per-6-mer Gaussian mixture emission parameters via EM and event re-alignment) — github.com/YinLabTJ/nanoHiMe
- **nanopolish** (Produces the event-level alignment (eventalign -n --scale-events) used as training input)
- **minimap2** (Aligns basecalled reads to the reference with -ax map-ont prior to event alignment)
- **samtools** (Indexes the reference (faidx) and filters reads to mapped, MAPQ >=20 (-b -q 20 -F 4))

## Examples

```
Model_training -m mcg -c 5 -r /.../ref_upper.fa -e /.../training.eventalign.txt.gz -o /.../output_dir -g 3 -p /.../EM_shift.txt -i 500
```

## Evaluation signals

- EM.out exists and contains one row for each 6-mer with omega, mu, and sigma for all three Gaussian components
- Sanity-check the AAAAAA row: omega1 ~0.9255, mu1 ~86.45, sigma1 ~1.21 as reported in the README example
- Each 6-mer's omega values sum to approximately 1
- Training eventalign input was generated from reads with mapping quality >=20 and nanopolish eventalign was run with --scale-events

## Limitations

- Partial adenine methylation in training DNA makes it impossible to precisely assign learned parameters to 6-mers with 6mA at different positions; such k-mers are collapsed to an averaged k-bar k-mer, so 6mA recognition is k-mer-level, not base-resolution
- EM defaults: 5 re-alignment cycles, 3 Gaussians per 6-mer, 1000 max iterations (paper used 500); initialization depends on the EM_shift.txt file
- A unified methylation-calling model covering all combinations of introduced modifications (5-hmC, 5-fC, 4-mC, GpC methylation) does not yet exist and requires more-extensive training models

## Evidence

- [other] Parameter training used four PCR-amplified E. coli K12 MG1655 datasets (untreated, M.SssI-, Hia5-, or both-treated).: [evidence span withheld — anchor status: not_found]
- [discussion] Viterbi re-alignment iterations quote: "We iterated the process for 5 times and assigned the parameters from the final iteration"
- [discussion] EM fitting of omega, mu, sigma: "To use the expectation-maximization algorithm to fit ωj, μj, σj"
- [discussion] k-bar collapsing limitation: "As the partial adenine methylation made it impossible to precisely assign the learned parameters to k-mers"
- [readme] README EM.out example values: "AAAAAA  0.925549559849263       86.4533687505569        1.21282717235525"
