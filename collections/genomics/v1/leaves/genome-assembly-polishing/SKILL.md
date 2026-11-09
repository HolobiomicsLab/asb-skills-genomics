---
name: genome-assembly-polishing
description: Use when a draft long-read genome assembly (e.g. FASTA such as ecoli_shasta.fa)
  needs base-error correction, small-indel repair, and gap filling using paired short
  reads (.fq.gz), and alignment-based polishers are too memory-intensive.
when_to_use_negative:
- The input is already a polished, finished assembly with no gaps or base errors to
  correct — the protocol's ntEdit and Sealer stages would have nothing to resolve.
- Only long reads are available with no short-read data — the Bloom filters are built
  from short-read k-mers via ntHits and ABySS-Bloom, so the protocol cannot run without
  them.
- A read-alignment-based polisher is preferred and the genome is small enough that
  alignment memory requirements are not a constraint (the protocol targets memory
  scalability).
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: ntEdit
  role: Performs alignment-free, k-mer-based correction of base errors and small indels
    in the draft assembly and marks potentially problematic regions
  repo: https://github.com/BirolLab/ntEdit
- name: Sealer
  role: Fills assembly gaps and problematic regions flagged by ntEdit using the Bloom
    filter cascade
  repo: https://github.com/BirolLab/abyss
- name: ntHits
  role: Counts and filters short-read k-mer frequencies for building Bloom filters
  repo: https://github.com/BirolLab/nthits
- name: ABySS-Bloom
  role: Creates Bloom filters from short-read k-mers used by ntEdit and Sealer
  repo: https://github.com/BirolLab/abyss
- name: GNU Make
  role: Drives the ntedit-sealer Makefile pipeline invoked with the 'finish' target
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cpz1.442
    title: 'ntEdit+Sealer: Efficient Targeted Error Resolution and Automated Finishing
      of Long‐Read Genome Assemblies'
  source_content_hash: sha256:407112f155d42cec5affa323008577bfe7e9bd826a397b51b5ada7596ae2b543
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
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# genome-assembly-polishing

## Summary

An alignment-free, k-mer-based genome finishing protocol (ntEdit+Sealer) that polishes a draft long-read assembly with short reads using memory-efficient Bloom filters built by ntHits and ABySS-Bloom, then gap-fills with Sealer.

## When to use

Use when a draft long-read genome assembly (e.g. FASTA such as ecoli_shasta.fa) needs base-error correction, small-indel repair, and gap filling using paired short reads (.fq.gz), and alignment-based polishers are too memory-intensive.

## When NOT to use

- The input is already a polished, finished assembly with no gaps or base errors to correct — the protocol's ntEdit and Sealer stages would have nothing to resolve.
- Only long reads are available with no short-read data — the Bloom filters are built from short-read k-mers via ntHits and ABySS-Bloom, so the protocol cannot run without them.
- A read-alignment-based polisher is preferred and the genome is small enough that alignment memory requirements are not a constraint (the protocol targets memory scalability).

## Inputs

- Draft long-read genome assembly in FASTA format (e.g. ecoli_shasta.fa)
- Paired short reads in gzipped FASTQ (e.g. SRR15859208_1.fq.gz and SRR15859208_2.fq.gz, E. coli strain NDM5, SRA accession SAMN21398207)
- k-mer lengths provided in descending order (e.g. k='80 65 50')
- Bloom filter size parameter b (e.g. b=200M)

## Outputs

- Polished and gap-filled assembly FASTA with suffix .ntedit_edited.prepd.sealer_scaffold.fa (e.g. ecoli_shasta.ntedit_edited.prepd.sealer_scaffold.fa)
- Makefile success message 'ntEdit and Sealer polishing steps complete!'

## How to apply

Install dependencies with Conda (`conda install -c bioconda nthits ntedit abyss`) so ntHits, ntEdit, ABySS-Bloom and Sealer are on the PATH, and clone the ntedit_sealer_protocol Makefile repository. Place the draft assembly (e.g. ecoli_shasta.fa) and paired short reads (e.g. SRR15859208_1.fq.gz SRR15859208_2.fq.gz) in the working directory. Run the Makefile 'finish' target, e.g. `ntedit-sealer finish seqs=ecoli_shasta.fa reads='SRR15859208_1.fq.gz SRR15859208_2.fq.gz' k='80 65 50' b=200M`, supplying k-mer lengths in descending order and choosing the Bloom filter size (b) appropriate for the genome. ntHits and ABySS-Bloom build the short-read k-mer Bloom filter cascade; ntEdit then performs alignment-free correction of base errors and small indels and flags problematic regions; Sealer fills assembly gaps and the flagged regions using the same Bloom filters. Confirm successful completion by the Makefile message 'ntEdit and Sealer polishing steps complete!' and the presence of the output assembly with suffix .ntedit_edited.prepd.sealer_scaffold.fa.

## Related tools

- **ntEdit** (Performs alignment-free, k-mer-based correction of base errors and small indels in the draft assembly and marks potentially problematic regions) — https://github.com/BirolLab/ntEdit
- **Sealer** (Fills assembly gaps and problematic regions flagged by ntEdit using the Bloom filter cascade) — https://github.com/BirolLab/abyss
- **ntHits** (Counts and filters short-read k-mer frequencies for building Bloom filters) — https://github.com/BirolLab/nthits
- **ABySS-Bloom** (Creates Bloom filters from short-read k-mers used by ntEdit and Sealer) — https://github.com/BirolLab/abyss
- **GNU Make** (Drives the ntedit-sealer Makefile pipeline invoked with the 'finish' target)

## Examples

```
ntedit-sealer finish seqs=ecoli_shasta.fa reads='SRR15859208_1.fq.gz SRR15859208_2.fq.gz' k='80 65 50' b=200M
```

## Evaluation signals

- The Makefile prints the success message 'ntEdit and Sealer polishing steps complete!' at the end of the run.
- The output file ecoli_shasta.ntedit_edited.prepd.sealer_scaffold.fa exists in the working directory with the expected .ntedit_edited.prepd.sealer_scaffold.fa suffix.
- K-mer lengths are supplied in descending order (e.g. '80 65 50'); the run fails or behaves incorrectly if this ordering is violated.
- Assembly quality is assessed after polishing (e.g. with QUAST): the finished assembly should show corrected base errors, fewer small indels, and filled gaps relative to the draft.
- Bloom filter size b (e.g. 200M) is set appropriately for the genome so that k-mer lookups by ntEdit and Sealer are reliable.

## Limitations

- Optimal k-mer lengths and Bloom filter size (b) must be selected by the user; poor choices reduce polishing and gap-filling performance.
- The protocol is alignment-free by design, so per-base alignment evidence of corrections is not produced; accuracy depends on the Bloom filter representation of short-read k-mers.
- Dependencies (ntHits, ntEdit, ABySS with Sealer and ABySS-Bloom, GNU Make) must be installed and on the PATH, e.g. via conda, before the Makefile can run.

## Evidence

- [abstract] Here we describe ntEdit+Sealer, an alignment-free, k-mer-based genome finishing protocol that employs memory-efficient Bloom filters: "Here we describe ntEdit+Sealer, an alignment-free, k-mer-based genome finishing protocol that employs memory-efficient Bloom filters"
- [abstract] The protocol includes ntEdit for correcting base errors and small indels, and for marking potentially problematic regions: "The protocol includes ntEdit for correcting base errors and small indels, and for marking potentially problematic regions"
- [intro] ntHits and ABySS-Bloom are required for creating Bloom filters from short read k-mers: "Additionally, ntHits (Mohamadi, Chu, Coombe, Warren, & Birol, 2020) and ABySS-Bloom (Jackman et al., 2017) are required for creating Bloom filters from short read k-mers"
- [methods] ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G: "ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G"
- [methods] conda install -c bioconda nthits ntedit abyss: "conda install -c bioconda nthits ntedit abyss"
- [readme] ntEdit polishes the draft assembly and flags erroneous regions, then Sealer fills assembly gaps and erroneous sequence regions flagged by ntEdit.: [evidence span withheld — anchor status: not_found]
