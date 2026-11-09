---
name: kmer-coverage-fraction-quantification
description: Use when evaluating whether nanopore/BS-seq derived methylation site
  lists cover all possible k-mer motifs around a cytosine context (CpG/CHG/CHH), or
  when reproducing a reported motif-coverage statistic before training a methylation-calling
  model.
when_to_use_negative:
- When the goal is single-molecule or per-read methylation accuracy (use DeepPlant/Dorado
  benchmarking metrics like F1 or BS-seq correlation instead)
- When contexts are not centered on a canonical cytosine motif (e.g., non-CHH or degenerate
  motifs) without first redefining the candidate motif space enumeration
- When site lists are low-confidence or lack matched positive/negative labels, since
  the >=50-support-per-class threshold requires both classes
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: DeepPlant
  role: Produces the high-confidence methylation site calls whose 9-mer CHH context
    coverage is being quantified; the coverage statistic validates the training dataset
    DeepPlant was trained on
  repo: https://github.com/xiaochuanle/DeepPlant
- name: DeepSignal-Plant
  role: Prior deep learning tool for genome-wide 5mC detection in CpG, CHG, and CHH
    contexts; contextualizes the need for diverse 9-mer motif coverage in plant methylation
    models
- name: extract_hc_sites
  role: DeepPlant subcommand that extracts high-confidence positive and negative sites
    with a configurable kmer_size, supplying the labeled site lists used for k-mer
    tallying
  repo: https://github.com/xiaochuanle/DeepPlant
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:46b713ec3e5c807defcd1d24d6fa3e92e4a253407448a4d6a1ed1ded40b009df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaochuanle/DeepPlant
  tools:
  - DeepPlant
  - DeepSignal-Plant
  - extract_hc_sites
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# kmer-coverage-fraction-quantification

## Summary

Quantifies what fraction of all possible sequence contexts (e.g., the 36,864 9-mers centered on a CHH motif) are represented in a methylation-site training dataset, using minimum positive/negative support thresholds. This verifies that a training corpus generalizes across sequence space, as in the DeepPlant CHH training set (97.2% coverage, ~9225 samples/context).

## When to use

Use when evaluating whether nanopore/BS-seq derived methylation site lists cover all possible k-mer motifs around a cytosine context (CpG/CHG/CHH), or when reproducing a reported motif-coverage statistic before training a methylation-calling model.

## When NOT to use

- When the goal is single-molecule or per-read methylation accuracy (use DeepPlant/Dorado benchmarking metrics like F1 or BS-seq correlation instead)
- When contexts are not centered on a canonical cytosine motif (e.g., non-CHH or degenerate motifs) without first redefining the candidate motif space enumeration
- When site lists are low-confidence or lack matched positive/negative labels, since the >=50-support-per-class threshold requires both classes

## Inputs

- High-confidence CHH positive (methylated) site lists from nanopore-derived methylation calls, per species
- High-confidence CHH negative (unmethylated) site lists, per species
- Reference genome sequences for extracting flanking-base context around each cytosine
- k-mer size parameter (k=9 for CHH contexts) and minimum support threshold (>=50 per class)

## Outputs

- Coverage fraction over the full candidate k-mer space (e.g., covered/36864, expected 0.972 for combined species)
- Per-9-mer support counts (positive and negative occurrences per motif)
- Summary table written to CSV (e.g., chh_9mer_coverage_report.csv) with covered count, coverage fraction, and per-context counts

## How to apply

Load and merge the high-confidence methylated (positive) and unmethylated (negative) cytosine site lists from each species' dataset (e.g., S. miltiorrhiza, S. tuberosum, R. communis nanopore data), extracting the k-mer sequence context centered on each cytosine with pandas/numpy (k=9 for CHH). Enumerate the full candidate motif space programmatically — for CHH 9-mers this is 9 central trinucleotide positions x 4^6 flanking-base combinations = 36,864 contexts via itertools.product over A/C/G/T. Tally per-motif occurrence counts separately in the positive and negative sets using the extract_hc_sites-style high-confidence site extraction, then flag a motif as covered only when it reaches the minimum-support threshold (>=50 occurrences in BOTH the positive and negative sets). Divide covered motifs by the total candidate count (36,864) to obtain the coverage fraction, and compare against the published benchmark (97.2% combined three-species coverage, averaging over 9225 samples per context). Rationale: DeepPlant's cross-species generalization depends on diverse 9-mer motif representation in training data, so this coverage metric directly validates training-set sufficiency.

## Related tools

- **DeepPlant** (Produces the high-confidence methylation site calls whose 9-mer CHH context coverage is being quantified; the coverage statistic validates the training dataset DeepPlant was trained on) — https://github.com/xiaochuanle/DeepPlant
- **DeepSignal-Plant** (Prior deep learning tool for genome-wide 5mC detection in CpG, CHG, and CHH contexts; contextualizes the need for diverse 9-mer motif coverage in plant methylation models)
- **extract_hc_sites** (DeepPlant subcommand that extracts high-confidence positive and negative sites with a configurable kmer_size, supplying the labeled site lists used for k-mer tallying) — https://github.com/xiaochuanle/DeepPlant

## Examples

```
python compute_chh_coverage.py --sites pos_sites.txt neg_sites.txt --kmer_size 9 --min_support 50 --output chh_9mer_coverage_report.csv
```

## Evaluation signals

- Covered-count invariant: the number of enumerated candidate contexts must equal 36,864 for 9-mer CHH contexts (9 x 4^6); any other total indicates an enumeration bug
- Coverage fraction for the combined S. miltiorrhiza + S. tuberosum + R. communis dataset should reproduce the published 97.2% (35,831 of 36,864 contexts covered)
- Average samples per covered context should be approximately 9225, as reported for the DeepPlant training dataset
- A motif is counted as covered only if BOTH positive and negative sets contain >=50 occurrences; verify no motif meeting only one threshold is counted
- Per-species coverage computed independently should be lower than the combined three-species coverage, confirming that merging datasets increases motif diversity

## Limitations

- Reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species
- Scarcity of high-methylation CHH samples, although partially mitigated in this study, continues to pose challenges for model training and generalization
- In vitro enzyme-catalyzed DNA methylation is ineffective for generating CHH positive samples because CHH methylation depends on RdDM (DRM1/DRM2), so coverage cannot easily be boosted artificially

## Evidence

- [intro] Generate new Nanopore and BS-seq data from selected species to increase the number of CHH-positive training samples, covering 97.2% of all possible 9-mers centered with CHH motifs: "The new dataset covers 97.2% of all possible 9-mers centered with CHH"
- [discussion] DeepPlant training dataset covers 97.2% of all possible 9-mer CHH contexts, averaging over 9225 samples per context: "Our comprehensive training dataset now covers 97.2% of all possible 9-mer CHH contexts, averaging over 9225 samples per context"
- [abstract] Scarcity of methylation-positive CHH training samples was addressed by screening species with abundant high-methylation CHH sites using bisulfite-sequencing and generating datasets covering diverse 9-mer motifs: "We address the scarcity of methylation-positive CHH training samples through screening species with abundant high-methylation CHH sites using bisulfite-sequencing and generate datasets that cover"
- [methods] extract_hc_sites subcommand with kmer_size argument used to extract high-confidence positive and negative sites: "Usage: extract_hc_sites [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir pos neg kmer_size num_workers"
- [discussion] Reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species: "Our reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species"
