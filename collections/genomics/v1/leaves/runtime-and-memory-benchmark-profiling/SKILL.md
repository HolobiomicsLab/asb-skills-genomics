---
name: runtime-and-memory-benchmark-profiling
description: Use when comparing the computational efficiency (wall-clock time and
  peak resident set size) of nanopore SNV variant callers on a real ONT dataset such
  as HG002 sup basecalled with Dorado.
when_to_use_negative:
- The goal is variant-calling accuracy evaluation (F1-score, precision, recall against
  GIAB truth) rather than computational efficiency comparison — use an accuracy benchmarking
  workflow instead.
- Input data is short-read or PacBio sequencing — the skill targets ONT long-read
  sup-datasets (R10.4.1, kit v14) as benchmarked in the article.
- Only a single variant caller needs to be run for production calling; the skill is
  specifically a comparative multi-caller efficiency benchmark.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: miniSNV
  role: Primary SNV caller being benchmarked; expected to have lowest peak memory
    and fastest runtime; requires index built from 1000 Genomes population VCF
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Clair3
  role: Comparator deep-learning caller (pileup + full-alignment); run with the ONT
    rerio-trained model on the HG002 sup BAM
  repo: https://github.com/nanoporetech/rerio
- name: Longshot
  role: Comparator diploid SNV caller using a pair hidden Markov model; observed at
    88.13 GB peak memory
- name: DeepVariant
  role: Comparator deep-learning variant caller included in the efficiency benchmark
- name: PEPPER-Margin-DeepVariant
  role: Comparator full-alignment-based ensemble caller included in the efficiency
    benchmark
- name: Nanocaller
  role: Comparator pile-based CNN caller included in the efficiency benchmark
- name: Oxford Nanopore Technologies (ONT) / Dorado
  role: Source of the sup basecalled HG002 BAM on which the benchmark is performed
  repo: https://github.com/nanoporetech/rerio
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:c0d2ec1ba7dcf82f7debee1b3d667909dcf884e1983fb4deeab43bdff6e03591
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CuiMiao-HIT/miniSNV
  repo_url: https://github.com/CuiMiao-HIT/miniSNV
  tools:
  - Clair3
  - DeepVariant
  - Longshot
  - Nanocaller
  - Oxford Nanopore Technologies (ONT) / Dorado
  - PEPPER-Margin-DeepVariant
  - miniSNV
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# runtime-and-memory-benchmark-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the wall-clock runtime and peak memory footprint (peak RSS) of ONT SNV variant callers — miniSNV, Clair3, Longshot, DeepVariant, PEPPER-Margin-DeepVariant, Nanocaller — on the HG002 sup-dataset to quantify miniSNV's memory and speed advantages.

## When to use

Use when comparing the computational efficiency (wall-clock time and peak resident set size) of nanopore SNV variant callers on a real ONT dataset such as HG002 sup basecalled with Dorado.

## When NOT to use

- The goal is variant-calling accuracy evaluation (F1-score, precision, recall against GIAB truth) rather than computational efficiency comparison — use an accuracy benchmarking workflow instead.
- Input data is short-read or PacBio sequencing — the skill targets ONT long-read sup-datasets (R10.4.1, kit v14) as benchmarked in the article.
- Only a single variant caller needs to be run for production calling; the skill is specifically a comparative multi-caller efficiency benchmark.

## Inputs

- HG002 ONT sup BAM (GIAB Ashkenazi Trio, kit v14, R10.4.1 flowcells, Dorado basecalled, samtools-indexed)
- GRCh38 reference FASTA (samtools-indexed)
- miniSNV index folder built from 1000 Genomes population VCF of known common variants
- ONT rerio-trained Clair3 model from github.com/nanoporetech/rerio/tree/master/clair3_models

## Outputs

- performance_table.csv with per-caller wall-clock runtime (seconds) and peak RSS (GB) under single-thread and default threading configurations
- Variant calls (VCF) from miniSNV, Clair3, Longshot, DeepVariant, PEPPER-Margin-DeepVariant, Nanocaller for downstream accuracy checks

## How to apply

Obtain the HG002 sup ONT BAM from the GIAB Ashkenazi Trio deposit (kit v14 chemistry, R10.4.1 flowcells, ~200 Gbases/sample, Dorado basecalled), the GRCh38 reference, and prepare the miniSNV index from the 1000 Genomes population VCF used as variation backgrounds. Run each caller under both single-thread and default threading configurations; for Clair3 use the ONT rerio-trained model from nanoporetech/rerio clair3_models. Wrap every invocation with a resource monitor such as /usr/bin/time -v to record wall-clock time in seconds and peak resident set size in GB. Tabulate per-caller runtime and peak RSS into performance_table.csv. Validate that miniSNV shows the lowest peak memory (4.76 GB vs Clair3 10.84 GB and Longshot 88.13 GB, i.e. ~2x and ~20x smaller) and the fastest single-thread runtime, while retaining F1-score and recall >99.7% on the HG002 sup-dataset.

## Related tools

- **miniSNV** (Primary SNV caller being benchmarked; expected to have lowest peak memory and fastest runtime; requires index built from 1000 Genomes population VCF) — https://github.com/CuiMiao-HIT/miniSNV
- **Clair3** (Comparator deep-learning caller (pileup + full-alignment); run with the ONT rerio-trained model on the HG002 sup BAM) — https://github.com/nanoporetech/rerio
- **Longshot** (Comparator diploid SNV caller using a pair hidden Markov model; observed at 88.13 GB peak memory)
- **DeepVariant** (Comparator deep-learning variant caller included in the efficiency benchmark)
- **PEPPER-Margin-DeepVariant** (Comparator full-alignment-based ensemble caller included in the efficiency benchmark)
- **Nanocaller** (Comparator pile-based CNN caller included in the efficiency benchmark)
- **Oxford Nanopore Technologies (ONT) / Dorado** (Source of the sup basecalled HG002 BAM on which the benchmark is performed) — https://github.com/nanoporetech/rerio

## Examples

```
/usr/bin/time -v python chr_chunk_task.py --fin_ref GRCh38.fa --fin_bam HG002_sup.bam --workDir workdir --useindex -i miniSNV_index --human --threads 1
```

## Evaluation signals

- miniSNV peak memory is 4.76 GB — lowest among all benchmarked callers on the HG002 sup-dataset.
- Clair3 peak memory is ~10.84 GB (~2x miniSNV) and Longshot peak memory is ~88.13 GB (~20x miniSNV).
- miniSNV records the shortest single-thread wall-clock runtime among all callers.
- Accuracy must not degrade while profiling: miniSNV F1-score and recall on the HG002 sup-dataset exceed 99.7%.
- Every caller invocation has both a wall-clock seconds value and a peak RSS GB value recorded under both single-thread and default threading configurations.

## Limitations

- Memory/runtime results are dataset- and hardware-dependent; figures (4.76 GB, 10.84 GB, 88.13 GB) were measured on the HG002 ONT sup-dataset under the paper's threading configurations.
- Clair3 rerio models are research releases not validated by Oxford Nanopore, and rerio notes 'The results generated by research basecalling models have not been scrutinized nor validated'.
- The document's discussion section contains only a changelog placeholder, so no author-stated caveats beyond benchmark context are available.

## Evidence

- [results] Memory footprint: [evidence span withheld — anchor status: not_found]
- [readme] Rerio research model caveat: [evidence span withheld — anchor status: external_doc]
