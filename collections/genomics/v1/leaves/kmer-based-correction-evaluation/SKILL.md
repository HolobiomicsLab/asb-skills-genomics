---
name: kmer-based-correction-evaluation
description: Use when comparing hybrid error-correction outputs (e.g., HECIL, LSC,
  PacBioToCA, LoRDEC, proovread, CoLoRMap) on PacBio long reads and you need a k-mer-based
  quality metric or a stopping rule for iterative correction.
when_to_use_negative:
- When the primary evaluation target is assembly-based quality (N50, dnadiff accuracy)
  rather than per-read correction — QUAST/MUMmer on Canu assemblies is the appropriate
  path
- When the input is uncorrected reads alone with no correction to evaluate — the metric
  requires a corrected read set to be meaningful
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Jellyfish
  role: k-mer counting tool used to compute the number of unique k-mers after each
    correction algorithm and at the end of each HECIL iteration
- name: HECIL
  role: hybrid error correction framework whose corrected long-read FASTA outputs
    are evaluated and whose iterative loop is terminated by the Jellyfish-based relative-improvement
    criterion
  repo: https://github.com/NDBL/HECIL
- name: BWA
  role: aligns short reads to updated long reads during realignment within the loop
    whose termination is governed by the k-mer criterion
  repo: https://github.com/NDBL/HECIL
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:30e9aa20a2e34519534be98af12ae8a349028b5efec9e57af754df5f7d60d396
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BWA
  - BWA-MEM
  - Canu
  - CoLoRMap
  - HECIL
  - Jellyfish
  - LSC
  - LoRDEC
  - PacBioToCA
  - QUAST
  - proovread
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# kmer-based-correction-evaluation

## Summary

Evaluate long-read error correction quality by counting unique k-mers of corrected reads with Jellyfish, using the number of unique k-mers as a proxy metric and as the relative-improvement termination signal in HECIL's iterative learning loop.

## When to use

Use when comparing hybrid error-correction outputs (e.g., HECIL, LSC, PacBioToCA, LoRDEC, proovread, CoLoRMap) on PacBio long reads and you need a k-mer-based quality metric or a stopping rule for iterative correction.

## When NOT to use

- When the primary evaluation target is assembly-based quality (N50, dnadiff accuracy) rather than per-read correction — QUAST/MUMmer on Canu assemblies is the appropriate path
- When the input is uncorrected reads alone with no correction to evaluate — the metric requires a corrected read set to be meaningful

## Inputs

- Corrected long-read FASTA (e.g., LongRead_Corrected.fasta from HECIL, or outputs of LoRDEC, proovread, PacBioToCA, LSC, CoLoRMap)
- Uncorrected PacBio long-read FASTA (e.g., E. coli K-12 MG1655 filtered set of 33,360 reads)
- Jellyfish-compatible k-mer length setting for counting unique k-mers

## Outputs

- Per-correction-run unique k-mer counts (one value per tool or per HECIL iteration)
- Per-iteration relative-improvement values used as the iterative-learning termination criterion
- hecil_iterative_corrected_reads.fasta with final corrected long reads and per-iteration unique k-mer counts

## How to apply

After each correction round, run Jellyfish on the corrected long-read FASTA to compute the number of unique k-mers, treating fewer unique k-mers as better correction (lower error burden). In the iterative setting, run Jellyfish at the end of every HECIL iteration and compute the relative improvement (#unique k-mers(ℓ−1) − #unique k-mers(ℓ)) / #unique k-mers(ℓ−1); terminate when this falls below ε ∈ (0,1) or when the maximum iteration count n is reached. Use the same k-mer metric to compare across baselines, as was done for the E. coli dataset (33,360 PacBio reads corrected with 22,720,100 Illumina short reads), where HECIL iteration 1 produced the lowest unique k-mer counts. Combine with alignment-based metrics (BLASR percent identity, percent matched bases) and assembly metrics (Canu/QUAST N50, MUMmer dnadiff) for a full benchmark, but the k-mer count alone is sufficient for the within-loop termination decision.

## Related tools

- **Jellyfish** (k-mer counting tool used to compute the number of unique k-mers after each correction algorithm and at the end of each HECIL iteration)
- **HECIL** (hybrid error correction framework whose corrected long-read FASTA outputs are evaluated and whose iterative loop is terminated by the Jellyfish-based relative-improvement criterion) — https://github.com/NDBL/HECIL
- **BWA** (aligns short reads to updated long reads during realignment within the loop whose termination is governed by the k-mer criterion) — https://github.com/NDBL/HECIL

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out; jellyfish count -m 21 -s 100M -t 16 LongRead_Corrected.fasta; jellyfish stats mer_counts.jf
```

## Evaluation signals

- Unique k-mer counts should decrease monotonically (or nearly so) across successive correction methods/iterations, with HECIL expected to produce the lowest counts
- The iterative loop must terminate when the relative improvement (#unique k-mers(ℓ−1) − #unique k-mers(ℓ)) / #unique k-mers(ℓ−1) falls below ε or the maximum iteration count n is reached
- The correction log should record monotonically fixed high-confidence corrections (α > 95 percentile of optimal costs J_k*) across iterations
- Cross-check the k-mer ranking against BLASR percent identity: the method with the fewest unique k-mers (e.g., HECIL iteration 1) should also rank at or near the top on PI and percent matched bases

## Limitations

- Iterative improvement in unique k-mers until saturation is only a conjecture, with no mathematical proof; proving it remains an open problem
- Fewer unique k-mers is a proxy: it must be paired with alignment-based metrics (BLASR) and assembly metrics (QUAST/dnadiff) to rule out over-correction
- Each additional HECIL iteration adds total execution time; the termination threshold ε and iteration cap n must be chosen to bound cost

## Evidence

- [results] Jellyfish unique k-mers as evaluation metric: "We employ the widely-used k-mer counting tool Jellyfish19 to compute the number of unique k-mers obtained after each correction algorithm"
- [methods] Termination criterion for iterative learning: "we also recommend selecting a secondary termination criterion: the maximal number n of allowable iterations for iterative learning."
- [results] HECIL iteration 1 achieves lowest unique k-mers: "long reads corrected by the core algorithm of HECIL (iteration 1) generate the lowest number of k-mers for each of these datasets"
- [readme] HECIL output file from README: "The output containing all (corrected and uncorrected) long reads will be saved in LongRead_Corrected.fasta"
- [discussion] Saturation conjecture is unproven: "We speculate that the iterative procedure will improve the performance metrics that we are concerned with (for example, the number of unique k-mers) until saturation."
