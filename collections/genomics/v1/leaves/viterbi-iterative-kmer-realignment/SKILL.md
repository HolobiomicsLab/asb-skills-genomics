---
name: viterbi-iterative-kmer-realignment
description: Use when training nanoHiMe emission parameters from a nanopolish eventalign
  file and you must iteratively reassign events to 6-mers via Viterbi decoding before
  EM fitting of Gaussian components.
when_to_use_negative:
- You need base-resolution 6mA prediction — the k-bar collapse of k-mers with 6mA
  at different positions means the model recognizes 6mA-containing vs non-6mA sites
  but cannot assign methylation to individual adenine positions
- You want methylation calls directly from raw signal without event alignment — this
  skill requires a nanopolish eventalign input, not FAST5 alone
- A pre-trained model covering your modification combination already suffices; retraining
  is only needed for new modification contexts
edam_operation: http://edamontology.org/operation_0478
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: nanoHiMe
  role: Runs Model_training (EM with 5 Viterbi re-alignment cycles, 3 Gaussians per
    6-mer, 500 iterations) to learn per-6-mer emission parameters
  repo: github.com/YinLabTJ/nanoHiMe
- name: nanopolish
  role: Produces the event-level alignment to reference k-mers (eventalign -n --scale-events)
    used as the training input
- name: minimap2
  role: Aligns basecalled ONT reads to the reference with -ax map-ont prior to event
    alignment
- name: samtools
  role: Indexes the reference and filters training reads to mapped, mapping-quality
    >=20 (view -b -q 20 -F 4)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:415093da455327d719d1368384b418d4b3de905e93c5aeec3b6da4bcb0025a42
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - minimap2
  - nanoHiMe
  - nanopolish
  - samtools
  edam_operation: http://edamontology.org/operation_0478
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Viterbi iterative k-mer realignment for emission-parameter training

## Summary

nanoHiMe trains per-6-mer emission distributions (Gaussian mixture omega/mu/sigma) by iteratively re-aligning nanopore event-level alignments to k-mers with the Viterbi algorithm and refitting the mixtures with expectation-maximization; k-mers with partially methylated adenines are collapsed to an averaged k-bar k-mer because parameters cannot be assigned position-precisely.

## When to use

Use when training nanoHiMe emission parameters from a nanopolish eventalign file and you must iteratively reassign events to 6-mers via Viterbi decoding before EM fitting of Gaussian components.

## When NOT to use

- You need base-resolution 6mA prediction — the k-bar collapse of k-mers with 6mA at different positions means the model recognizes 6mA-containing vs non-6mA sites but cannot assign methylation to individual adenine positions
- You want methylation calls directly from raw signal without event alignment — this skill requires a nanopolish eventalign input, not FAST5 alone
- A pre-trained model covering your modification combination already suffices; retraining is only needed for new modification contexts

## Inputs

- nanopolish eventalign file (test.eventalign.txt or .gz) from mapped, mapping-quality >=20 reads
- Uppercase reference FASTA with samtools faidx index (e.g. ref_upper.fa, hg19.fa, E. coli K12 MG1655)
- Initial EM parameter file (EM_shift.txt)
- Training datasets, e.g. four PCR-amplified E. coli K12 MG1655 sets (untreated, M.SssI-, Hia5-, or both-treated)

## Outputs

- EM.out table of per-6-mer Gaussian mixture parameters (omega1/mu1/sigma1, omega2/mu2/sigma2, omega3/mu3/sigma3)
- Trained nanoHiMe model usable for mCG calling and 6mA-containing-region calling
- k-bar (collapsed) k-mer parameters for partially methylated adenine k-mers

## How to apply

Prepare the event-level alignment by indexing the reference, aligning basecalled reads with minimap2 -ax map-ont, filtering to mapped reads with mapping quality >=20 via samtools view -b -q 20 -F 4, and running nanopolish eventalign -n --scale-events. Run Model_training with -m (ma, mcg, or ma_mcg), -c 5 cycles of event re-alignment, -g 3 Gaussians per 6-mer, -i 500 EM iterations (default 1000), and initialization from the EM_shift.txt parameter file, using absolute paths for -r, -e, -o, and -p. Each cycle reassigns events to k-mers with the Viterbi algorithm, refits omega/mu/sigma by EM, and the process is iterated 5 times, keeping the parameters from the final iteration. For k-mers containing 6mA at different positions, partial adenine methylation prevents precise parameter assignment, so such k-mers are grouped into a new k-bar k-mer that receives the learned parameters. Trained models are then used for CpG methylation calling and 6mA-containing-region calling with log-likelihood-ratio thresholds (1.5/-1.5 for CpG; 8 or 32 for 50 bp 6mA windows).

## Related tools

- **nanoHiMe** (Runs Model_training (EM with 5 Viterbi re-alignment cycles, 3 Gaussians per 6-mer, 500 iterations) to learn per-6-mer emission parameters) — github.com/YinLabTJ/nanoHiMe
- **nanopolish** (Produces the event-level alignment to reference k-mers (eventalign -n --scale-events) used as the training input)
- **minimap2** (Aligns basecalled ONT reads to the reference with -ax map-ont prior to event alignment)
- **samtools** (Indexes the reference and filters training reads to mapped, mapping-quality >=20 (view -b -q 20 -F 4))

## Examples

```
Model_training -m mcg -c 5 -r /path/ref_upper.fa -e /path/training.eventalign.txt.gz -o /path/output_dir -g 3 -p /path/EM_shift.txt -i 500
```

## Evaluation signals

- EM.out exists and contains a row for each 6-mer with omega, mu, and sigma for all three Gaussian components
- The AAAAAA row matches expected training output (omega1 ~0.926, mu1 ~86.45, sigma1 ~1.21)
- Re-alignment completed the specified number of cycles (-c 5) and EM reached the iteration cap (-i 500)
- Model output reproduces published classification behavior, e.g. nanoHiMe outperforming Megalodon for 6mA-containing-site prediction on dual-modification DNA
- All input paths passed to Model_training (-r, -e, -o, -p) are absolute and the reference has a samtools faidx index

## Limitations

- Partial adenine methylation makes it impossible to precisely assign learned parameters to k-mers with 6mA at different positions, so such k-mers are collapsed to an averaged k-bar k-mer without base-resolution 6mA prediction
- More-extensive training models are required for all combinations of introduced modifications (5-hmC, 5-fC, 4-mC, GpC methylation); a unified calling model does not yet exist
- Parameters are taken from the final iteration only, so convergence quality depends on the EM_shift.txt initialization and the -c/-i settings

## Evidence

- [discussion] Iterative Viterbi realignment with EM parameter refitting: "We iterated the process for 5 times and assigned the parameters from the final iteration"
- [discussion] EM fitting of Gaussian mixture parameters: "To use the expectation-maximization algorithm to fit ωj, μj, σj"
- [discussion] k-bar collapse rationale: "As the partial adenine methylation made it impossible to precisely assign the learned parameters to k-mers"
- [methods] Training read filter: "For the reads with mapping quality ≥20, we performed initial alignment between events"
- [readme] Model_training invocation and defaults: "Model_training -m mcg -c 5 -r /.../ref_upper.fa -e /.../training.eventalign.txt.gz -o /.../output_dir -g 3 -p /.../EM_shift.txt -i 500"
- [readme] EM.out output schema: "The output file EM.out will contain the parameters of each gaussian distribution for each 6-mer"
