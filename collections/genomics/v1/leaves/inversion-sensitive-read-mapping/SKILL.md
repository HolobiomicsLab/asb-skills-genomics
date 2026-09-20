---
name: inversion-sensitive-read-mapping
description: Use when mapping PacBio or ONT long reads to a reference genome where
  inversion structural variants are expected or are the target of detection, or when
  benchmarking mapper accuracy on SURVIVOR-implanted inversion truth sets.
when_to_use_negative:
- 'Reads are short (Illumina-style): invMap targets long noisy PacBio/ONT reads that
  can span inversion breakpoints'
- The analysis goal is small variant or SNV calling rather than inversion/structural-variant
  detection
- The machine lacks the minimum 16G of RAM required for mapping to large reference
  genomes
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: invMap
  role: Primary mapper that chains anchors and flags potential inversions, outputting
    SAM alignments
  repo: https://github.com/zhang134/invMap.git
- name: SURVIVOR
  role: Implants 1000 inversions into the GRCh37 reference to generate the altered
    reference and truth VCF for benchmarking
- name: minimap2
  role: Baseline mapper for accuracy comparison on implanted inversions
- name: NGMLR
  role: Baseline long-read mapper in the inversion-detection accuracy comparison
- name: Winnowmap2
  role: Baseline long-read mapper in the inversion-detection accuracy comparison
- name: lra
  role: Baseline long-read mapper in the inversion-detection accuracy comparison
- name: SVision
  role: Calls structural variants from each mapper's alignments for truth-set comparison
- name: samtools
  role: Converts and sorts each mapper's SAM output to BAM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:a81a513caba9b29007607c6750587a35083b5869031264afdf26f41656583980
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - NGMLR
  - SURVIVOR
  - SVision
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# inversion-sensitive-read-mapping

## Summary

Map long noisy reads (PacBio/Oxford Nanopore) to a reference genome with invMap, a mapper specifically designed to chain anchors and detect inversion structural variants, then call SVs and benchmark accuracy against minimap2, NGMLR, Winnowmap2, and lra.

## When to use

Use when mapping PacBio or ONT long reads to a reference genome where inversion structural variants are expected or are the target of detection, or when benchmarking mapper accuracy on SURVIVOR-implanted inversion truth sets.

## When NOT to use

- Reads are short (Illumina-style): invMap targets long noisy PacBio/ONT reads that can span inversion breakpoints
- The analysis goal is small variant or SNV calling rather than inversion/structural-variant detection
- The machine lacks the minimum 16G of RAM required for mapping to large reference genomes

## Inputs

- Reference genome in FASTA format (e.g. GRCh37 with SURVIVOR-implanted inversions)
- Long noisy reads in FASTA or FASTQ format (PacBio or Oxford Nanopore)
- Truth VCF of SURVIVOR-implanted inversions (for benchmarking)

## Outputs

- SAM-format alignment file (invMap_aligned.sam) flagging candidate inversions
- Sorted BAM per mapper (via samtools)
- SV calls from SVision compared against the inversion truth set, summarized as per-mapper accuracy (e.g. mapper_accuracy_benchmark.csv)

## How to apply

Compile invMap from source with 'make' and index/run it on a FASTA genome and FASTA/FASTQ reads, redirecting SAM output (e.g. './invMap genome.fa reads.fa > invMap_aligned.sam'). invMap first locates each read's aligned region with a specifically designed scoring method for chaining, then checks remaining anchors in the aligned region to discover potential inversions. For benchmarking, implant 1000 inversions into GRCh37 with SURVIVOR to create an altered reference plus a truth VCF, map the HG002 PacBio reads with each mapper, convert/sort SAM to BAM with samtools, call SVs with SVision, and compare calls to the truth set. Published real-read accuracies to expect: invMap 76.55%, minimap2 75.15%, NGMLR 73.4%, Winnowmap2 67.10%, lra 63.25%.

## Related tools

- **invMap** (Primary mapper that chains anchors and flags potential inversions, outputting SAM alignments) — https://github.com/zhang134/invMap.git
- **SURVIVOR** (Implants 1000 inversions into the GRCh37 reference to generate the altered reference and truth VCF for benchmarking)
- **minimap2** (Baseline mapper for accuracy comparison on implanted inversions)
- **NGMLR** (Baseline long-read mapper in the inversion-detection accuracy comparison)
- **Winnowmap2** (Baseline long-read mapper in the inversion-detection accuracy comparison)
- **lra** (Baseline long-read mapper in the inversion-detection accuracy comparison)
- **SVision** (Calls structural variants from each mapper's alignments for truth-set comparison)
- **samtools** (Converts and sorts each mapper's SAM output to BAM)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- Output is valid SAM (invMap_aligned.sam) and converts/sorts cleanly to BAM with samtools without errors
- On the HG002 PacBio benchmark with 1000 SURVIVOR-implanted inversions, invMap accuracy should approximate 76.55% (vs minimap2 75.15%, NGMLR 73.4%, Winnowmap2 67.10%, lra 63.25%)
- SV calls from SVision align to the implanted-inversion truth VCF at the expected rate
- On the NA12878 real dataset, invMap should surface more candidate inversion variant calls than competing mappers

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes
- Accuracy figures are specific to the HG002 PacBio / 1000-implanted-inversion benchmark and may vary with genome, coverage, and read chemistry
- Requires local compilation with make; input must be FASTA or FASTQ format

## Evidence

- [abstract] invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [results] invMap achieves 76.55% accuracy, higher than minimap2 (75.15%), NGMLR (73.4%), and Winnowmap2 (67.10%): [evidence span withheld — anchor status: not_found]
- [readme] ./invMap genome.fa reads.fa > invMap_aligned.sam: "./invMap genome.fa reads.fa > invMap_aligned.sam"
- [intro] invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes.: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
- [readme] Input file requires fasta or fastq format: "Input file requires fasta or fastq format"
