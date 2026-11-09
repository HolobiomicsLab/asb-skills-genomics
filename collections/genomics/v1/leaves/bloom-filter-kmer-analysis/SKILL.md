---
name: bloom-filter-kmer-analysis
description: Use when polishing a draft long-read genome assembly with short reads,
  correcting base errors/small indels and filling gaps, especially when alignment-based
  polishing tools like Pilon or Racon would be too memory-intensive.
when_to_use_negative:
- When only long reads are available and no short reads exist to build the Bloom filters
  from short-read k-mers
- When an alignment-based hybrid polishing tool is required (e.g. alignment-dependent
  workflows around Pilon or Racon), since ntEdit+Sealer is deliberately alignment-free
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: ntEdit
  role: Alignment-free, k-mer-based correction of base errors and small indels; flags
    potentially problematic regions in the draft assembly
  repo: https://github.com/BirolLab/ntEdit
- name: Sealer
  role: Fills assembly gaps and problematic regions flagged by ntEdit using the Bloom
    filter cascade
  repo: https://github.com/BirolLab/abyss
- name: ntHits
  role: Counts and filters short-read k-mers to build the Bloom filters used by the
    protocol
  repo: https://github.com/BirolLab/nthits
- name: ABySS-Bloom
  role: Creates Bloom filters from short read k-mers consumed by ntEdit and Sealer
  repo: https://github.com/BirolLab/abyss
- name: GNU Make
  role: Implements the ntEdit+Sealer pipeline as a Makefile with the 'finish' target
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cpz1.442
    title: 'ntEdit+Sealer: Efficient Targeted Error Resolution and Automated Finishing
      of Long‐Read Genome Assemblies'
  source_content_hash: sha256:52115b3c404b7b2ac5e5bbdb12fe83a4ae27076c8f29fbd71efe178d3d2ba271
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
yes_rate: 0.6
n_claims: 15
---

# bloom-filter-kmer-analysis

## Summary

An alignment-free, k-mer-based genome finishing approach (ntEdit+Sealer) that uses memory-efficient Bloom filters built from short-read k-mers by ntHits and ABySS-Bloom to polish long-read assemblies and fill gaps, invoked as a Makefile pipeline.

## When to use

Use when polishing a draft long-read genome assembly with short reads, correcting base errors/small indels and filling gaps, especially when alignment-based polishing tools like Pilon or Racon would be too memory-intensive.

## When NOT to use

- When only long reads are available and no short reads exist to build the Bloom filters from short-read k-mers
- When an alignment-based hybrid polishing tool is required (e.g. alignment-dependent workflows around Pilon or Racon), since ntEdit+Sealer is deliberately alignment-free

## Inputs

- Draft long-read genome assembly in FASTA format (e.g. ecoli_shasta.fa)
- Paired-end short reads in gzipped FASTQ format (.fq.gz, e.g. SRR15859208_1.fq.gz and SRR15859208_2.fq.gz)
- Descending list of k-mer lengths (e.g. 80 65 50) and Bloom filter size (e.g. b=200M)

## Outputs

- Polished, gap-filled finished assembly FASTA with suffix .ntedit_edited.prepd.sealer_scaffold.fa (e.g. ecoli_shasta.ntedit_edited.prepd.sealer_scaffold.fa)
- Makefile success message 'ntEdit and Sealer polishing steps complete!'

## How to apply

Install dependencies with Conda (conda install -c bioconda nthits ntedit abyss) so ntHits, ntEdit, ABySS-Bloom and Sealer are on the PATH. Place the draft assembly FASTA (e.g. ecoli_shasta.fa) and paired short reads (.fq.gz) in the working directory. Run the Makefile target 'ntedit-sealer finish' with seqs=<draft-assembly.fa>, reads='<reads_1.fq.gz reads_2.fq.gz>', k-mer lengths in descending order (k='80 65 50'), and a Bloom filter size suited to the genome (b=200M for E. coli; b=5G in the generic example). ntHits and ABySS-Bloom build Bloom filters from short-read k-mers; ntEdit then performs alignment-free, k-mer-based correction of base errors and small indels and flags problematic regions, and Sealer fills assembly gaps and those flagged regions using the same Bloom filter cascade. Verify completion by the Makefile success message 'ntEdit and Sealer polishing steps complete!' and the output file suffixed .ntedit_edited.prepd.sealer_scaffold.fa.

## Related tools

- **ntEdit** (Alignment-free, k-mer-based correction of base errors and small indels; flags potentially problematic regions in the draft assembly) — https://github.com/BirolLab/ntEdit
- **Sealer** (Fills assembly gaps and problematic regions flagged by ntEdit using the Bloom filter cascade) — https://github.com/BirolLab/abyss
- **ntHits** (Counts and filters short-read k-mers to build the Bloom filters used by the protocol) — https://github.com/BirolLab/nthits
- **ABySS-Bloom** (Creates Bloom filters from short read k-mers consumed by ntEdit and Sealer) — https://github.com/BirolLab/abyss
- **GNU Make** (Implements the ntEdit+Sealer pipeline as a Makefile with the 'finish' target)

## Examples

```
ntedit-sealer finish seqs=ecoli_shasta.fa reads='SRR15859208_1.fq.gz SRR15859208_2.fq.gz' k='80 65 50' b=200M
```

## Evaluation signals

- The polished output file <assembly>.ntedit_edited.prepd.sealer_scaffold.fa exists in the working directory
- The Makefile prints the success message 'ntEdit and Sealer polishing steps complete!'
- k-mer lengths were supplied in descending order (e.g. k='80 65 50')
- Dependencies (ntHits, ntEdit, ABySS-Bloom, Sealer) resolved on the PATH via the conda install -c bioconda nthits ntedit abyss step
- Bloom filter size parameter (b) was set appropriately for the genome (200M for E. coli scale inputs)

## Limitations

- Selecting optimal values for k-mer lengths (k) and Bloom filter size (b) is a required tuning step; inappropriate values affect results
- Existing hybrid polishing methods rely on read alignments and are memory-intensive, which is why the alignment-free Bloom filter route is used; the protocol itself depends on short-read data
- Reads must be placed (or soft-linked) in the current working directory as .fq.gz for the Makefile pipeline to find them

## Evidence

- [abstract] Here we describe ntEdit+Sealer, an alignment-free, k-mer-based genome finishing protocol that employs memory-efficient Bloom filters: "Here we describe ntEdit+Sealer, an alignment-free, k-mer-based genome finishing protocol that employs memory-efficient Bloom filters"
- [abstract] The protocol includes ntEdit for correcting base errors and small indels, and for marking potentially problematic regions: "The protocol includes ntEdit for correcting base errors and small indels, and for marking potentially problematic regions"
- [abstract] then Sealer for filling both assembly gaps and problematic regions flagged by ntEdit: "then Sealer for filling both assembly gaps and problematic regions flagged by ntEdit"
- [intro] ntHits (Mohamadi, Chu, Coombe, Warren, & Birol, 2020) and ABySS-Bloom (Jackman et al., 2017) are required for creating Bloom filters from short read k-mers: "ntHits (Mohamadi, Chu, Coombe, Warren, & Birol, 2020) and ABySS-Bloom (Jackman et al., 2017) are required for creating Bloom filters from short read k-mers"
- [methods] conda install -c bioconda nthits ntedit abyss: "conda install -c bioconda nthits ntedit abyss"
- [methods] ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G: "ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G"
- [readme] An automated protocol for finishing long-read genome assemblies using short reads.: "An automated protocol for finishing long-read genome assemblies using short reads."
