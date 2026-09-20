---
name: paired-end-fastq-read-handling
description: Use when input data are gzip-compressed paired-end FASTQ short reads
  (e.g., the SRR15859208 E. coli NDM5 dataset) and a k-mer distribution or Bloom filter
  input is needed before k/b selection for ntEdit+Sealer.
when_to_use_negative:
- Input reads are single-end or unpaired FASTQ — the skill assumes forward and reverse
  mate files supplied jointly
- Reads are uncompressed or in a non-FASTQ container (e.g., BAM) — the pipeline expects
  .fq.gz files in the working directory
- The goal is assembly polishing or gap filling itself rather than k-mer distribution
  profiling — use the downstream ntEdit/Sealer steps instead
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: ntCard
  role: k-mer counting tool run once on both FASTQ read files with -k80,65,50 to generate
    the k-mer frequency histograms
  repo: https://github.com/BirolLab/ntCard
- name: ntedit_sealer_protocol
  role: Makefile pipeline that consumes the paired-end reads (reads='reads_1.fq.gz
    reads_2.fq.gz') for Bloom filter building and finishing
  repo: https://github.com/BirolLab/ntedit_sealer_protocol
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cpz1.442
    title: 'ntEdit+Sealer: Efficient Targeted Error Resolution and Automated Finishing
      of Long‐Read Genome Assemblies'
  source_content_hash: sha256:7d24ce83f369d075083c0cbb4a165cbba80f2ca208a121ece1495666d5be3b15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BirolLab/ntCard
  tools:
  - ntCard
  - ntEdit
  - ntedit_sealer_protocol
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# paired-end-fastq-read-handling

## Summary

Handles gzip-compressed paired-end short-read FASTQ files (.fq.gz) by passing the forward and reverse read files jointly to k-mer counting tools such as ntCard, so k-mer statistics are computed in a single pass over both mates of the library.

## When to use

Use when input data are gzip-compressed paired-end FASTQ short reads (e.g., the SRR15859208 E. coli NDM5 dataset) and a k-mer distribution or Bloom filter input is needed before k/b selection for ntEdit+Sealer.

## When NOT to use

- Input reads are single-end or unpaired FASTQ — the skill assumes forward and reverse mate files supplied jointly
- Reads are uncompressed or in a non-FASTQ container (e.g., BAM) — the pipeline expects .fq.gz files in the working directory
- The goal is assembly polishing or gap filling itself rather than k-mer distribution profiling — use the downstream ntEdit/Sealer steps instead

## Inputs

- gzip-compressed paired-end short-read FASTQ files (reads_1.fq.gz forward, reads_2.fq.gz reverse)
- comma-separated list of k-mer sizes (e.g., 80,65,50)

## Outputs

- two-column tab-separated k-mer frequency histograms per k value (freq_k80.hist, freq_k65.hist, freq_k50.hist)
- Fk summary metric lines on the first two rows of each histogram (F0: distinct k-mers; F1: total k-mers)

## How to apply

Obtain the paired-end short-read FASTQ files (forward and reverse), keeping them gzip-compressed with .fq.gz extensions in the working directory. Run the k-mer counting tool once, supplying both read files on the command line together with a comma-separated k list (e.g., -k80,65,50), so counting is performed jointly for all k-mer sizes in a single pass. ntCard then emits one two-column, tab-separated k-mer frequency histogram per k value using the prefix freq (freq_k80.hist, freq_k65.hist, freq_k50.hist). These histograms feed the subsequent selection of k-mer lengths (k) and Bloom filter size (b) in the ntEdit+Sealer Makefile pipeline, where the same paired reads are passed as reads='reads_1.fq.gz reads_2.fq.gz'.

## Related tools

- **ntCard** (k-mer counting tool run once on both FASTQ read files with -k80,65,50 to generate the k-mer frequency histograms) — https://github.com/BirolLab/ntCard
- **ntedit_sealer_protocol** (Makefile pipeline that consumes the paired-end reads (reads='reads_1.fq.gz reads_2.fq.gz') for Bloom filter building and finishing) — https://github.com/BirolLab/ntedit_sealer_protocol

## Examples

```
ntcard -k80,65,50 reads_1.fq.gz reads_2.fq.gz --prefix=freq
```

## Evaluation signals

- All three histogram files (freq_k80.hist, freq_k65.hist, freq_k50.hist) exist in the working directory after a single ntCard run
- Each histogram file is a two-column, tab-separated file
- The first two rows of each histogram contain the Fk summary metric lines: F0 (distinct k-mers) and F1 (total k-mers)
- F1 should be substantially larger than F0, consistent with total k-mers counting every occurrence while F0 counts distinct k-mers only
- Both mate files (forward and reverse) were supplied in the same ntCard invocation so counts reflect the full paired-end library

## Limitations

- Hist- generation depends on the exact prefix convention (freq); other prefixes require adapting downstream validation checks
- All planned k values must be passed in a single -k argument (comma-separated) to benefit from the single-pass joint counting; separate runs per k would duplicate work
- The protocol expects reads to remain gzip-compressed with .fq.gz extensions; decompressed or renamed inputs may break the Makefile pipeline

## Evidence

- [full_text] Running ntcard with -k80,65,50 on both FASTQ read files generates a k-mer distribution histogram for each k, printed to a two-column, tab-separated file: [evidence span withheld — anchor status: not_found]
- [full_text] Obtain the paired-end short-read FASTQ files (forward and reverse) for the SRR15859208 E. coli NDM5 dataset, ensuring they are gzip-compressed with .fq.gz extensions: [evidence span withheld — anchor status: not_found]
- [methods] ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G: "ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G"
- [readme] An automated protocol for finishing long-read genome assemblies using short reads.: "An automated protocol for finishing long-read genome assemblies using short reads."
