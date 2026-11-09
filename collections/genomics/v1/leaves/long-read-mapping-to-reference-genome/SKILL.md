---
name: long-read-mapping-to-reference-genome
description: Use when mapping long noisy PacBio or Oxford Nanopore reads against a
  large reference genome and inversion structural variant calls are desired, e.g.
  NA12878-style datasets in FASTA/FASTQ with a reference genome.fa.
when_to_use_negative:
- Reads are short Illumina reads — invMap is designed for long noisy PacBio/Oxford
  Nanopore reads that span SV breakpoints
- The reference genome is small and memory is limited — invMap requires a minimum
  of 16G of RAM for mapping sequence datasets to large reference genomes
- The goal is detection of structural variants other than inversions (e.g. pure duplications),
  since the anchor-transformation step targets inversion discovery
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: invMap
  role: 'Primary mapper: indexes the reference genome, chains anchors to locate aligned
    regions, detects inversion SVs, and outputs SAM alignments'
  repo: https://github.com/zhang134/invMap.git
- name: KSW2
  role: Gap-filling alignment algorithm used to complete the chained alignment in
    the final pipeline step
- name: PacBio
  role: Source platform of long reads that span SV breakpoints and serve as mapper
    input
- name: Oxford Nanopore
  role: Source platform of long reads that span SV breakpoints and serve as mapper
    input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:d66b1409ee6869d08104d9bc2b9927090c582e26d64d05a78512bc18d636f2f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - KSW2
  - Oxford Nanopore
  - PacBio
  - invMap
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# long-read-mapping-to-reference-genome

## Summary

Map long noisy PacBio/Oxford Nanopore reads to a reference genome with invMap, a chaining-based aligner that transforms non-co-linear anchors to co-linear form so that inversion structural variants are detected and reported alongside standard alignments in SAM format.

## When to use

Use when mapping long noisy PacBio or Oxford Nanopore reads against a large reference genome and inversion structural variant calls are desired, e.g. NA12878-style datasets in FASTA/FASTQ with a reference genome.fa.

## When NOT to use

- Reads are short Illumina reads — invMap is designed for long noisy PacBio/Oxford Nanopore reads that span SV breakpoints
- The reference genome is small and memory is limited — invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes
- The goal is detection of structural variants other than inversions (e.g. pure duplications), since the anchor-transformation step targets inversion discovery

## Inputs

- Reference genome in FASTA format (genome.fa)
- Long noisy sequencing reads in FASTA or FASTQ format (PacBio or Oxford Nanopore)

## Outputs

- invMap_aligned.sam — mapped alignments in SAM format, containing aligned records whose inversions are reported as candidate structural variant (SV) calls

## How to apply

Compile the invMap source with the make command, then run ./invmap to inspect help and options before a real run. invMap first builds a minimizer hash-table index of the reference genome for rapid seed lookup; for each long noisy read it retrieves seed anchors and applies a specifically designed dynamic-programming scoring method for chaining to locate the aligned region. It then examines the remaining anchors inside the located aligned region, transforming nonlinear anchors to linear form (with boundary region extension) to detect potential inversion chains, and fills remaining gaps in the chained alignment with KSW2. Write results with a shell redirect, e.g. ./invMap genome.fa reads.fa > invMap_aligned.sam; ensure at least 16G RAM is available for mapping to large reference genomes.

## Related tools

- **invMap** (Primary mapper: indexes the reference genome, chains anchors to locate aligned regions, detects inversion SVs, and outputs SAM alignments) — https://github.com/zhang134/invMap.git
- **KSW2** (Gap-filling alignment algorithm used to complete the chained alignment in the final pipeline step)
- **PacBio** (Source platform of long reads that span SV breakpoints and serve as mapper input)
- **Oxford Nanopore** (Source platform of long reads that span SV breakpoints and serve as mapper input)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- invMap_aligned.sam exists and is parseable as valid SAM format
- SAM contains aligned records with inversion events reported as candidate SV calls
- Inversion calls are more accurate in locating aligned regions than competing mappers, as claimed on simulated datasets across different genomes and sequencing coverages
- On real data (e.g. NA12878), invMap finds more candidate variant calls for inversions than competing methods
- Running ./invmap with no arguments prints help information and option explanations, confirming a successful build

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes
- Input must be FASTA or FASTQ; other formats are unsupported
- Linux is recommended (Windows 7/10 supported); ~800 M of available disk space is needed
- Detected variants are candidate calls — inversion detection depends on the presence of non-co-linear anchors within the located aligned region

## Evidence

- [abstract] invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [abstract] Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs) than shorter reads: "Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs) than shorter reads"
- [intro] invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes.: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
- [readme] ./invMap genome.fa reads.fa > invMap_aligned.sam: "./invMap genome.fa reads.fa > invMap_aligned.sam"
- [readme] Input file requires fasta or fastq format: "Input file requires fasta or fastq format"
