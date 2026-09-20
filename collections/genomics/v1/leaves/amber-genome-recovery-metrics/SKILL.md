---
name: amber-genome-recovery-metrics
description: Use when comparing binning tool outputs (e.g., SemiBin2 vs. an ablation
  variant) on datasets with known reference genomes, to quantify high-quality bin
  recovery at strain/species/genus resolution.
when_to_use_negative:
- Datasets lack ground-truth reference genomes, since AMBER evaluation requires known
  genome assignments to score bin precision and recall
- You need de novo bin quality assessment without references — use single-copy marker
  gene or coverage/Completeness-Contamination heuristics instead
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: AMBER
  role: Evaluates all bins from both binning configurations against ground truth,
    computing high-quality bin counts at strain/species/genus levels
- name: SemiBin2
  role: Produces the primary bins (self-supervised mode via single_easy_bin) that
    are evaluated against the NoSemi ablation
  repo: https://github.com/BigDataBiology/SemiBin/
- name: samtools
  role: Sorts and indexes BAM files from read-to-contig mapping before binning, a
    prerequisite for generating the evaluated bins
  repo: https://github.com/samtools/samtools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:5287ef4db78c48ea2bf6cd0d0620c261245f0da4bfaaa21a8e335f00129295fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BigDataBiology/SemiBin
  tools:
  - AMBER
  - SemiBin1
  - SemiBin2
  - samtools
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# amber-genome-recovery-metrics

## Summary

Evaluate metagenomic binning results (MAG recovery) with AMBER by comparing predicted bins against ground-truth genomes, counting high-quality bins at strain, species, and genus levels. This is the benchmark metric used in the SemiBin2 paper to quantify binning performance on CAMI II simulated datasets.

## When to use

Use when comparing binning tool outputs (e.g., SemiBin2 vs. an ablation variant) on datasets with known reference genomes, to quantify high-quality bin recovery at strain/species/genus resolution.

## When NOT to use

- Datasets lack ground-truth reference genomes, since AMBER evaluation requires known genome assignments to score bin precision and recall
- You need de novo bin quality assessment without references — use single-copy marker gene or coverage/Completeness-Contamination heuristics instead

## Inputs

- Predicted bins (output_bins directories) from each binning configuration and sequencing type
- Ground-truth genome assignments / gold-standard mappings for the CAMI II simulated datasets
- Per-environment bin sets from five CAMI II environments, for both short-read and long-read arms

## Outputs

- High-quality bin counts at strain, species, and genus levels (short reads) and total high-quality bins (long reads) per configuration
- Percentage increase in high-quality bins of the full method over the ablation per environment and sequencing type
- Aggregate comparison table (self_supervised_ablation_results.tsv)

## How to apply

After generating bins for each configuration (e.g., full SemiBin2 self-supervised mode vs. the NoSemi ablation that skips train_self and clusters on the original data.csv/data_split.csv feature tables), run AMBER on all bins from both configurations using the ground-truth genome assignments. For CAMI II short-read data, compute the number of distinct high-quality bins at strain, species, and genus levels; for long-read data, compute total high-quality bins. Then compute the percentage increase of the full method over the ablation per environment and per sequencing type (the paper reports SemiBin2 recovering 21.6%/18.4%/17.1% more high-quality strains/species/genera on short reads and 31.5% more high-quality bins on long reads). Write per-environment and aggregate comparisons into a TSV table and verify it confirms the full method outperforms the ablation.

## Related tools

- **AMBER** (Evaluates all bins from both binning configurations against ground truth, computing high-quality bin counts at strain/species/genus levels)
- **SemiBin2** (Produces the primary bins (self-supervised mode via single_easy_bin) that are evaluated against the NoSemi ablation) — https://github.com/BigDataBiology/SemiBin/
- **samtools** (Sorts and indexes BAM files from read-to-contig mapping before binning, a prerequisite for generating the evaluated bins) — https://github.com/samtools/samtools

## Examples

```
SemiBin single_easy_bin -i contigs.fasta -b *.bam -o output_bins --sequencing-type=long_read
```

## Evaluation signals

- self_supervised_ablation_results.tsv exists and contains per-environment and aggregate comparisons for both short-read and long-read arms
- Full SemiBin2 with self-supervised learning shows strictly more high-quality bins than the NoSemi ablation in each environment, consistent with 21.6%/18.4%/17.1% more high-quality strains/species/genera (short reads) and 31.5% more high-quality bins (long reads)
- High-quality bin counts are reported separately at strain, species, and genus levels for short reads, and as total high-quality bins for long reads
- Percentage increases recomputed per environment match the direction and approximate magnitude of the paper's reported aggregates

## Limitations

- Evaluation depends on ground-truth annotations, so it is only applicable to simulated/benchmark datasets (e.g., CAMI II) where true genome compositions are known
- Many SemiBin operations are probabilistic — running the pipeline twice can return different bins and hence different AMBER scores (noted in the SemiBin tutorial README)
- The comparison here is an ablation (NoSemi), not a full cross-binner benchmark; results may not generalize to comparisons with other binning tools

## Evidence

- [abstract] SemiBin2 uses self-supervised learning to learn feature embeddings from contigs, and self-supervised learning achieves better results than the semi-supervised learning used in SemiBin1 in simulated and real datasets: "We propose SemiBin2, which uses self-supervised learning to learn feature embeddings from the contigs. In simulated and real datasets, we show that self-supervised learning achieves better results"
- [methods] Ablation removes train_self and clusters on original feature tables; evaluation counts high-quality bins at strain/species/genus levels.: "train_semi (replacing train_self from the self-supervised mode)"
- [readme] Benchmark code for evaluating binners is provided in the SemiBin2_benchmark repository.: [evidence span withheld — anchor status: external_doc]
- [readme] SemiBin operations are probabilistic and repeated runs can yield different results, affecting reproducibility of evaluation scores.: [evidence span withheld — anchor status: external_doc]
- [methods] Binning pipeline prerequisites: assemble reads into contigs, map reads back, sort/index BAM files with samtools.: "**Step 2**: Map reads to the FASTA file generated in _Step 1_."
