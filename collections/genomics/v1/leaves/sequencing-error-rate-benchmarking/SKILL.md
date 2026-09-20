---
name: sequencing-error-rate-benchmarking
description: Use when raw PacBio CLR read sets (e.g., yeast pseudo-diploid, NWC metagenome,
  Microbial 10-plex) must be corrected and their residual sequencing error rate computed
  to reproduce or compare against published correction benchmarks.
when_to_use_negative:
- Input reads are Oxford Nanopore rather than PacBio CLR — the reported validation
  values (0.089%/0.101%/0.236%) are specific to PacBio datasets and will not reproduce
- Reads are already corrected or consensus-polished, so residual error-rate benchmarking
  against raw-read correction results is meaningless
- Small exploratory datasets where VeChat's higher runtimes and peak memory usage
  on large files (needing --split/HPC splitting) are not warranted
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3693
tools:
- name: VeChat
  role: Two-cycle variation-graph-based error correction of PacBio reads with default
    graph-pruning parameters (min-confidence 0.2, min-support 0.2, min-identity-cns
    0.99)
  repo: https://github.com/HaploKit/vechat
- name: minimap2
  role: Computes minimizer-based all-vs-all read overlaps that feed VeChat's correction
    cycles and back-alignment for error quantification
  repo: https://github.com/lh3/minimap2
- name: racon
  role: Modified racon built into the VeChat conda environment, used as consensus/alignment
    module in the correction pipeline
  repo: https://github.com/lbcb-sci/racon
- name: fpa
  role: Filters all-against-all read mapping output (pinned at version 0.5 in the
    conda setup)
  repo: https://github.com/natir/fpa
- name: yacrd
  role: Chimeric read detection over all-against-all mappings in the VeChat environment
  repo: https://github.com/natir/yacrd
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:444615ca239c8f2ca3236bd6704580cbe833b4389143b7849a2aa5fb5f49a527
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - VeChat
  - fpa
  - minimap2
  - racon
  - yacrd
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3693
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# sequencing-error-rate-benchmarking

## Summary

Benchmarks long-read error-correction performance by running correction tools (VeChat and competitors) on real PacBio CLR datasets and quantifying residual error rates (read accuracy/QV) against published values from the VeChat paper.

## When to use

Use when raw PacBio CLR read sets (e.g., yeast pseudo-diploid, NWC metagenome, Microbial 10-plex) must be corrected and their residual sequencing error rate computed to reproduce or compare against published correction benchmarks.

## When NOT to use

- Input reads are Oxford Nanopore rather than PacBio CLR — the reported validation values (0.089%/0.101%/0.236%) are specific to PacBio datasets and will not reproduce
- Reads are already corrected or consensus-polished, so residual error-rate benchmarking against raw-read correction results is meaningless
- Small exploratory datasets where VeChat's higher runtimes and peak memory usage on large files (needing --split/HPC splitting) are not warranted

## Inputs

- Raw PacBio CLR read sets in FASTA/FASTQ format (gzip supported): Microbial 10-plex metagenome, NWC metagenome, yeast pseudo-diploid genome (ENA PRJEB7245)
- Corrected read outputs from competing state-of-the-art tools for comparison (e.g., Racon, Canu, Daccord, LoRMA, CONSENT, FLAS)

## Outputs

- reads.corrected.fa per dataset
- Residual sequencing error rate (read accuracy/QV) per dataset
- pacbio_error_rates.tsv tabulating the three error rates (0.089%, 0.101%, 0.236%)

## How to apply

Download the real PacBio CLR read sets (Microbial 10-plex metagenome, NWC metagenome, yeast pseudo-diploid from ENA PRJEB7245) in FASTA/FASTQ format. Set up the environment via conda (bioconda vechat with minimap2, yacrd, fpa=0.5, and the modified racon built automatically). For each dataset, run VeChat with the PacBio platform setting and default graph-pruning parameters (min-confidence 0.2, min-support 0.2, min-identity-cns 0.99) so the two-cycle variation-graph correction runs as reported; use --split for FASTQ > 20G or FASTA > 10G and optionally the scripts/vechat_hpc.fast.sh HPC splitter. Align corrected reads back and quantify residual error rate (read accuracy/QV), then tabulate in pacbio_error_rates.tsv. Validation: error rates should reproduce 0.089% (Microbial 10-plex), 0.101% (NWC metagenome), and 0.236% (yeast pseudo-diploid), and reads.corrected.fa must exist for each dataset.

## Related tools

- **VeChat** (Two-cycle variation-graph-based error correction of PacBio reads with default graph-pruning parameters (min-confidence 0.2, min-support 0.2, min-identity-cns 0.99)) — https://github.com/HaploKit/vechat
- **minimap2** (Computes minimizer-based all-vs-all read overlaps that feed VeChat's correction cycles and back-alignment for error quantification) — https://github.com/lh3/minimap2
- **racon** (Modified racon built into the VeChat conda environment, used as consensus/alignment module in the correction pipeline) — https://github.com/lbcb-sci/racon
- **fpa** (Filters all-against-all read mapping output (pinned at version 0.5 in the conda setup)) — https://github.com/natir/fpa
- **yacrd** (Chimeric read detection over all-against-all mappings in the VeChat environment) — https://github.com/natir/yacrd

## Examples

```
vechat reads.fq.gz -t 8 --platform pb -o reads.corrected.fa
```

## Evaluation signals

- Computed error rates reproduce the published values: 0.089% (Microbial 10-plex), 0.101% (NWC metagenome), 0.236% (yeast pseudo-diploid)
- reads.corrected.fa exists for each dataset after the VeChat run
- VeChat error rates are the lowest of all tools tested: ~2–4x (yeast), 1.4–7.8x (NWC), and 3.3–5.6x (Microbial 10-plex) lower than competing correction tools
- Default parameters were used (min-confidence 0.2, min-support 0.2, min-identity-cns 0.99) and --platform pb was set
- pacbio_error_rates.tsv contains all three tabulated error rates with matching dataset labels

## Limitations

- VeChat does not win in terms of computational resources; on large datasets it experiences longer runtimes and higher peak memory usage
- VeChat uses off-the-shelf approaches without exploiting all their features, causing unnecessary overheads that could be avoided
- Computations such as edit-distance-based alignments and sequence-to-graph alignments could be replaced by more efficient routines in the future

## Evidence

- [results] VeChat attains the lowest error rates of all tools tested — 0.236% (Yeast pseudo-diploid), 0.101% (NWC metagenome) and 0.089% (Microbial 10-plex, per Table 4): [evidence span withheld — anchor status: not_found]
- [abstract] long reads corrected by VeChat contain 4 to 15 (Pacific Biosciences) and 1 to 10 times (Oxford Nanopore Technologies) less errors than when being corrected by state of the art approaches: "long reads corrected by VeChat contain 4 to 15 (Pacific Biosciences) and 1 to 10 times (Oxford Nanopore Technologies) less errors than when being corrected by state of the art approaches"
- [other] vechat reads.fq.gz -t 8 --platform pb -o reads.corrected.fa with default graph-pruning parameters min-confidence 0.2, min-support 0.2, min-identity-cns 0.99: [evidence span withheld — anchor status: not_found]
- [discussion] VeChat does not win the competition in terms of computational resources. In particular on large datasets, VeChat experiences longer runtimes and higher peak memory usage: "VeChat does not win the competition in terms of computational resources. In particular on large datasets, VeChat experiences longer runtimes and higher peak memory usage"
- [readme] One can test the program using the small sequencing read file `example/reads.fq.gz`: "One can test the program using the small sequencing read file `example/reads.fq.gz`"
