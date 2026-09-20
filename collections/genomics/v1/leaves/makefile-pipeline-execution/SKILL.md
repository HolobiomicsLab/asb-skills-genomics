---
name: makefile-pipeline-execution
description: Use when a draft long-read genome assembly (e.g. a Shasta assembly FASTA)
  must be polished and gap-filled with paired short reads (fq.gz) via the ntEdit+Sealer
  Makefile protocol.
when_to_use_negative:
- The goal is hybrid polishing that requires read alignments (e.g. Pilon or Racon
  workflows) rather than alignment-free, k-mer-based Bloom-filter polishing
- The draft assembly is already polished and gap-filled, so no base-error correction
  or gap resolution is needed
- Only long reads are available and no short-read data exist to build the k-mer Bloom
  filters
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: ntEdit
  role: Performs alignment-free, k-mer-based correction of base errors and small indels
    in the draft assembly and flags potentially problematic regions
  repo: https://github.com/BirolLab/ntEdit
- name: Sealer
  role: Fills assembly gaps and erroneous sequence regions flagged by ntEdit using
    the short-read Bloom filter cascade
  repo: https://github.com/BirolLab/abyss
- name: ntHits
  role: Counts and filters short-read k-mers for building the Bloom filters used by
    ntEdit and Sealer
  repo: https://github.com/BirolLab/nthits
- name: ABySS-Bloom
  role: Creates Bloom filters from short-read k-mers for the protocol's memory-efficient
    k-mer lookups
  repo: https://github.com/BirolLab/abyss
- name: GNU Make
  role: Orchestrates the protocol as a Makefile pipeline with the 'finish' target
    and variable overrides (seqs, reads, k, b)
- name: ntEdit+Sealer
  role: The combined Makefile-implemented genome finishing protocol being executed
  repo: https://github.com/BirolLab/ntedit_sealer_protocol
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cpz1.442
    title: 'ntEdit+Sealer: Efficient Targeted Error Resolution and Automated Finishing
      of Long‐Read Genome Assemblies'
  source_content_hash: sha256:c3beedd8a45fe313796666a17c791efedf643e04ffee61902db040032c5e796d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BirolLab/ntEdit
  tools:
  - ABySS
  - ABySS-Bloom
  - GNU Make
  - Sealer
  - ntEdit
  - ntEdit+Sealer
  - ntHits
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# makefile-pipeline-execution

## Summary

Executes the ntEdit+Sealer assembly finishing protocol as a GNU Make pipeline that polishes a draft long-read assembly and gap-fills it using short-read Bloom filters, invoked with a single 'finish' target and variable overrides.

## When to use

Use when a draft long-read genome assembly (e.g. a Shasta assembly FASTA) must be polished and gap-filled with paired short reads (fq.gz) via the ntEdit+Sealer Makefile protocol.

## When NOT to use

- The goal is hybrid polishing that requires read alignments (e.g. Pilon or Racon workflows) rather than alignment-free, k-mer-based Bloom-filter polishing
- The draft assembly is already polished and gap-filled, so no base-error correction or gap resolution is needed
- Only long reads are available and no short-read data exist to build the k-mer Bloom filters

## Inputs

- Draft long-read genome assembly FASTA (e.g. ecoli_shasta.fa from a Shasta assembly of E. coli strain NDM5)
- Paired-end short-read FASTQ files in .fq.gz format (e.g. SRR15859208_1.fq.gz, SRR15859208_2.fq.gz from SRA accession SAMN21398207)
- K-mer lengths provided in descending order (e.g. '80 65 50')
- Bloom filter size parameter (e.g. b=200M for the E. coli demo, b=5G generically)

## Outputs

- Polished and gap-filled assembly FASTA with suffix .ntedit_edited.prepd.sealer_scaffold.fa (e.g. ecoli_shasta.ntedit_edited.prepd.sealer_scaffold.fa)
- Makefile success message 'ntEdit and Sealer polishing steps complete!'

## How to apply

Clone the BirolLab/ntedit_sealer_protocol repository and install dependencies with 'conda install -c bioconda nthits ntedit abyss' so ntHits, ntEdit, ABySS-Bloom and Sealer are on the PATH. Place the draft assembly (e.g. ecoli_shasta.fa) and the paired short reads (e.g. SRR15859208_1.fq.gz SRR15859208_2.fq.gz) in the working directory, soft-linking if needed. Invoke the Makefile's 'finish' target with variable overrides, supplying k-mer lengths in descending order (k='80 65 50') and a Bloom filter size (b=200M for the ~5 Mb E. coli demo; b=5G is the generic example). GNU Make orchestrates ntHits and ABySS-Bloom building short-read k-mer Bloom filters, ntEdit's alignment-free correction of base errors and small indels, and Sealer's gap filling of regions flagged by ntEdit. Correct application is judged by the Makefile success message 'ntEdit and Sealer polishing steps complete!' and the presence of the finished assembly.

## Related tools

- **ntEdit** (Performs alignment-free, k-mer-based correction of base errors and small indels in the draft assembly and flags potentially problematic regions) — https://github.com/BirolLab/ntEdit
- **Sealer** (Fills assembly gaps and erroneous sequence regions flagged by ntEdit using the short-read Bloom filter cascade) — https://github.com/BirolLab/abyss
- **ntHits** (Counts and filters short-read k-mers for building the Bloom filters used by ntEdit and Sealer) — https://github.com/BirolLab/nthits
- **ABySS-Bloom** (Creates Bloom filters from short-read k-mers for the protocol's memory-efficient k-mer lookups) — https://github.com/BirolLab/abyss
- **GNU Make** (Orchestrates the protocol as a Makefile pipeline with the 'finish' target and variable overrides (seqs, reads, k, b))
- **ntEdit+Sealer** (The combined Makefile-implemented genome finishing protocol being executed) — https://github.com/BirolLab/ntedit_sealer_protocol

## Examples

```
ntedit-sealer finish seqs=ecoli_shasta.fa reads='SRR15859208_1.fq.gz SRR15859208_2.fq.gz' k='80 65 50' b=200M
```

## Evaluation signals

- The Makefile completes with the success message 'ntEdit and Sealer polishing steps complete!'
- An output file named <assembly>.ntedit_edited.prepd.sealer_scaffold.fa (e.g. ecoli_shasta.ntedit_edited.prepd.sealer_scaffold.fa) exists in the working directory
- K-mer lengths were passed in descending order (k='80 65 50') as the pipeline expects
- The Bloom filter size parameter (b) was set appropriately for genome size (e.g. b=200M for E. coli, b=5G generically)
- Dependencies (ntHits, ntEdit, ABySS including Sealer and ABySS-Bloom) are on the PATH, e.g. installed via conda from bioconda

## Limitations

- Bloom filter size (b) and k-mer lengths (k) must be selected appropriately; suboptimal values affect correction and gap-filling performance
- ntEdit+Sealer is alignment-free and k-mer-based, so it differs from alignment-based polishers like Pilon and Racon and may not suit all polishing needs
- The protocol requires both a draft assembly and short-read data (as .fq.gz) in the working directory; files may need to be soft-linked

## Evidence

- [abstract] Here we describe ntEdit+Sealer, an alignment-free, k-mer-based genome finishing protocol that employs memory-efficient Bloom filters: "Here we describe ntEdit+Sealer, an alignment-free, k-mer-based genome finishing protocol that employs memory-efficient Bloom filters"
- [methods] install via conda: "conda install -c bioconda nthits ntedit abyss"
- [methods] generic finish invocation: "ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G"
- [readme] README pipeline description: "An automated protocol for finishing long-read genome assemblies using short reads."
- [abstract] protocol is a Makefile: "is available as a Makefile pipeline from https://github.com/bcgsc/ntedit_sealer_protocol"
