---
name: gap-filling-draft-genome-assemblies
description: Use when a long-read draft genome assembly (e.g. FASTA with gaps and
  base errors) needs polishing and gap-filling with available paired short reads,
  and alignment-based polishers like Pilon or Racon would be too memory-intensive.
when_to_use_negative:
- Short-read-only assemblies or assemblies with no gaps/base errors to correct — the
  protocol is designed to finish long-read draft assemblies using short reads
- No short-read data is available — ntEdit and Sealer both rely on Bloom filters built
  from short-read k-mers
- Memory is not a constraint and alignment-based hybrid polishers such as Pilon or
  Racon are preferred for their information-rich alignments
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: ntEdit
  role: Performs alignment-free, k-mer-based correction of base errors and small indels
    and flags potentially problematic regions in the draft assembly
  repo: https://github.com/BirolLab/ntEdit
- name: Sealer
  role: Fills assembly gaps and the problematic regions flagged by ntEdit using the
    shared Bloom filter cascade
  repo: https://github.com/BirolLab/abyss
- name: ntHits
  role: Counts and filters short-read k-mers for building the Bloom filters used by
    ntEdit and Sealer
  repo: https://github.com/BirolLab/nthits
- name: ABySS-Bloom
  role: Required (with ntHits) for creating Bloom filters from short-read k-mers
  repo: https://github.com/BirolLab/abyss
- name: GNU Make
  role: Runs the protocol's 'finish' Makefile target that orchestrates the ntEdit
    and Sealer steps
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cpz1.442
    title: 'ntEdit+Sealer: Efficient Targeted Error Resolution and Automated Finishing
      of Long‐Read Genome Assemblies'
  source_content_hash: sha256:aa53f815e9ab7be6349fe751e3ce7c3bacaebe4536730b6b09df5bd47ff51fe9
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
yes_rate: 0.7692307692307693
n_claims: 13
---

# gap-filling-draft-genome-assemblies

## Summary

Runs the ntEdit+Sealer Makefile protocol, which uses alignment-free, k-mer-based Bloom filters built from short reads (via ntHits and ABySS-Bloom) to first polish base errors and small indels with ntEdit and then fill assembly gaps and problematic regions with Sealer in a long-read draft assembly.

## When to use

Use when a long-read draft genome assembly (e.g. FASTA with gaps and base errors) needs polishing and gap-filling with available paired short reads, and alignment-based polishers like Pilon or Racon would be too memory-intensive.

## When NOT to use

- Short-read-only assemblies or assemblies with no gaps/base errors to correct — the protocol is designed to finish long-read draft assemblies using short reads
- No short-read data is available — ntEdit and Sealer both rely on Bloom filters built from short-read k-mers
- Memory is not a constraint and alignment-based hybrid polishers such as Pilon or Racon are preferred for their information-rich alignments

## Inputs

- Draft long-read genome assembly FASTA (e.g. ecoli_shasta.fa)
- Paired-end short-read FASTQ files in gzipped form (e.g. SRR15840208-style reads_1.fq.gz and reads_2.fq.gz; demo uses SRR15859208_1.fq.gz and SRR15859208_2.fq.gz from E. coli strain NDM5, NCBI SRA SAMN21398207)

## Outputs

- Polished and gap-filled assembly FASTA with suffix .ntedit_edited.prepd.sealer_scaffold.fa (e.g. ecoli_shasta.ntedit_edited.prepd.sealer_scaffold.fa)
- Makefile success message 'ntEdit and Sealer polishing steps complete!'

## How to apply

Install the dependencies (conda install -c bioconda nthits ntedit abyss) so ntHits, ntEdit, ABySS-Bloom and Sealer are on the PATH, and clone the ntedit_sealer_protocol Makefile repository. Place the draft assembly (e.g. ecoli_shasta.fa) and paired short reads (SRR15859208_1.fq.gz, SRR15859208_2.fq.gz) in the working directory. Invoke the 'finish' target with the assembly as seqs, both read files as reads, k-mer lengths in descending order (k='80 65 50'), and a Bloom filter size (b=200M for the ~E. coli demo; b=5G is shown for larger genomes in the article). ntHits and ABySS-Bloom build the short-read k-mer Bloom filters; ntEdit then performs alignment-free correction of base errors and small indels and flags problematic regions; Sealer fills assembly gaps and the flagged regions using the same Bloom filter cascade. Confirm successful completion by the presence of the polished assembly named <assembly>.ntedit_edited.prepd.sealer_scaffold.fa and the Makefile success message.

## Related tools

- **ntEdit** (Performs alignment-free, k-mer-based correction of base errors and small indels and flags potentially problematic regions in the draft assembly) — https://github.com/BirolLab/ntEdit
- **Sealer** (Fills assembly gaps and the problematic regions flagged by ntEdit using the shared Bloom filter cascade) — https://github.com/BirolLab/abyss
- **ntHits** (Counts and filters short-read k-mers for building the Bloom filters used by ntEdit and Sealer) — https://github.com/BirolLab/nthits
- **ABySS-Bloom** (Required (with ntHits) for creating Bloom filters from short-read k-mers) — https://github.com/BirolLab/abyss
- **GNU Make** (Runs the protocol's 'finish' Makefile target that orchestrates the ntEdit and Sealer steps)

## Examples

```
ntedit-sealer finish seqs=ecoli_shasta.fa reads='SRR15859208_1.fq.gz SRR15859208_2.fq.gz' k='80 65 50' b=200M
```

## Evaluation signals

- The output file <assembly>.ntedit_edited.prepd.sealer_scaffold.fa (e.g. ecoli_shasta.ntedit_edited.prepd.sealer_scaffold.fa) exists in the working directory
- The Makefile prints the success message 'ntEdit and Sealer polishing steps complete!'
- The k-mer lengths were supplied in descending order (k='80 65 50') as the protocol requires
- The polished assembly is gap-filled relative to the input draft (gap count reduced) and base errors/small indels were corrected
- Intermediate Bloom filters were built at the requested size (e.g. b=200M for the E. coli demo) by ntHits and ABySS-Bloom before ntEdit ran

## Limitations

- k-mer lengths must be given in descending order (e.g. 80 65 50); supplying them otherwise is a misuse of the protocol
- Bloom filter size (b) must be selected appropriately for the genome/read set (200M for the E. coli demo, 5G shown for larger genomes); poor choices affect results
- The protocol is alignment-free by design, so it does not exploit the full information content of read alignments that tools like Pilon or Racon use
-  ntEdit only corrects base errors and small indels; larger problematic regions are flagged rather than corrected, and Sealer fills gaps/flagged regions from short-read k-mers

## Evidence

- [abstract] Protocol invokes ntEdit for polishing and Sealer for gap-filling: "The protocol includes ntEdit for correcting base errors and small indels, and for marking potentially problematic regions"
- [abstract] Sealer fills gaps and ntEdit-flagged regions: "then Sealer for filling both assembly gaps and problematic regions flagged by ntEdit"
- [methods] Concrete finish command with parameters from the article methods: "ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G"
- [methods] Dependencies installed via conda: "conda install -c bioconda nthits ntedit abyss"
- [intro] Bloom filter rationale from intro: "ntHits (Mohamadi, Chu, Coombe, Warren, & Birol, 2020) and ABySS-Bloom (Jackman et al., 2017) are required for creating Bloom filters from short read k-mers"
- [intro] Pipeline output description: "The output of ntEdit+Sealer is a polished and gap-filled long-read genome assembly."
