---
name: structural-variant-breakpoint-mapping-evaluation
description: Use when comparing the accuracy of long-read mappers for locating aligned
  regions or detecting inversion SVs on simulated PacBio CCS/CLR or ONT reads across
  reference genomes of different sizes.
when_to_use_negative:
- The goal is real-data variant discovery rather than mapper benchmarking — e.g. finding
  candidate inversion variant calls on a real dataset such as NA12878 (a separate
  downstream analysis).
- Input reads are short reads or already accurately aligned with trusted ground-truth
  coordinates, where location-accuracy benchmarking adds no information.
- The reference genome or read set is much larger than chromosome-scale assemblies
  used here — invMap requires a minimum of 16 GB RAM for mapping to large reference
  genomes.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: invMap
  role: Long-read mapper under evaluation; maps reads to the reference genome and
    emits SAM alignments used to compute location accuracy and detect inversions
  repo: https://github.com/zhang134/invMap.git
- name: PBSIM
  role: Simulates PacBio CCS and CLR reads from the reference genomes
- name: PBSIM2
  role: Simulates Oxford Nanopore (ONT) reads from the reference genomes
- name: minimap2
  role: Competing long-read mapper, run with its long-read preset as baseline
- name: Winnowmap2
  role: Competing long-read mapper, run with its long-read preset as baseline
- name: lra
  role: Competing long-read mapper, run with its long-read preset as baseline
- name: NGMLR
  role: Competing long-read mapper, run with its long-read preset as baseline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:41afb703098b4ae030aa84a1eeb6b62f6df88a76e7345ab7b4acc34da4da60bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - NGMLR
  - PBSIM
  - PBSIM2
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# structural-variant-breakpoint-mapping-evaluation

## Summary

Benchmark long-read mapping accuracy and inversion structural-variant (SV) calling for invMap against minimap2, Winnowmap2, lra, and NGMLR using PBSIM/PBSIM2-simulated PacBio CCS, CLR, and ONT reads, computing location accuracy as the fraction of reads whose aligned region matches the true simulated origin.

## When to use

Use when comparing the accuracy of long-read mappers for locating aligned regions or detecting inversion SVs on simulated PacBio CCS/CLR or ONT reads across reference genomes of different sizes.

## When NOT to use

- The goal is real-data variant discovery rather than mapper benchmarking — e.g. finding candidate inversion variant calls on a real dataset such as NA12878 (a separate downstream analysis).
- Input reads are short reads or already accurately aligned with trusted ground-truth coordinates, where location-accuracy benchmarking adds no information.
- The reference genome or read set is much larger than chromosome-scale assemblies used here — invMap requires a minimum of 16 GB RAM for mapping to large reference genomes.

## Inputs

- Reference genome FASTA files (Neisseria meningitidis, Abortiporus biennis, GRCh37 chromosome 1)
- Simulated long reads: PacBio CCS and CLR from PBSIM, ONT from PBSIM2 (FASTA/FASTQ)
- SAM alignment files from invMap, minimap2, Winnowmap2, lra, and NGMLR

## Outputs

- Per-mapper, per-genome, per-platform location-accuracy table (location_accuracy_s4.csv)
- invMap SAM output (invMap_aligned.sam)
- Comparison against Supplementary Table S4 / Fig. 4 expectations

## How to apply

Download the three reference genome FASTA files (Neisseria meningitidis, Abortiporus biennis, GRCh37 chromosome 1) and simulate PacBio CCS and CLR reads with PBSIM and ONT reads with PBSIM2 at the coverages used in the paper. Map each simulated read set back to its reference with invMap ('./invMap genome.fa reads.fa > invMap_aligned.sam') and with minimap2, Winnowmap2, lra, and NGMLR using their long-read presets. Parse each mapper's SAM alignments and compute location accuracy: the fraction of reads whose reported aligned region matches the true simulated origin region. Aggregate per-mapper, per-genome, per-platform accuracy values into location_accuracy_s4.csv; the reproduced result is judged correct if invMap exceeds 99% on PacBio CCS/CLR and 98% on ONT read sets while outperforming the four competing mappers. Rationale: invMap's chaining-based scoring locates the aligned region first, then inspects remaining anchors within that region to discover potential inversions, which drives its higher location accuracy and inversion SV calls.

## Related tools

- **invMap** (Long-read mapper under evaluation; maps reads to the reference genome and emits SAM alignments used to compute location accuracy and detect inversions) — https://github.com/zhang134/invMap.git
- **PBSIM** (Simulates PacBio CCS and CLR reads from the reference genomes)
- **PBSIM2** (Simulates Oxford Nanopore (ONT) reads from the reference genomes)
- **minimap2** (Competing long-read mapper, run with its long-read preset as baseline)
- **Winnowmap2** (Competing long-read mapper, run with its long-read preset as baseline)
- **lra** (Competing long-read mapper, run with its long-read preset as baseline)
- **NGMLR** (Competing long-read mapper, run with its long-read preset as baseline)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- invMap attains >99% location accuracy on simulated PacBio CCS read sets
- invMap attains >99% location accuracy on simulated PacBio CLR read sets
- invMap attains >98% location accuracy on simulated ONT read sets
- invMap's accuracy values match Supplementary Table S4 / Fig. 4 and exceed those of minimap2, Winnowmap2, lra, and NGMLR
- All mapper outputs parse as valid SAM files with aligned-region coordinates comparable to the true simulated origin regions

## Limitations

- invMap requires a minimum of 16 GB of RAM for mapping sequence datasets to large reference genomes.
- Benchmark is based on simulated reads; validation on real data is limited to the NA12878 human sequencing dataset where invMap is evaluated by the number of candidate inversion variant calls found.
- Approximately 800 MB of available disk space is required for the invMap installation.

## Evidence

- [abstract] invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [abstract] invMap is more accurate to locate aligned regions and call SVs for inversions than competing methods on simulated datasets: "experimental results demonstrate that invMap is more accurate to locate aligned regions and call SVs for inversions than the competing methods"
- [intro] run invMap on a genome and reads file to produce SAM output: "'./invMap genome.fa reads.fa > invMap_aligned.sam'"
- [intro] invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes.: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
- [readme] Input file requires fasta or fastq format: "Input file requires fasta or fastq format"
