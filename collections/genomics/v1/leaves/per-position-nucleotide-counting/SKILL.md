---
name: per-position-nucleotide-counting
description: Use when MinION reads have been mapped to target sequences (e.g., via
  LAST) and a per-target consensus in FASTA format is needed, with explicit coverage
  and ambiguity handling per position.
when_to_use_negative:
- The goal is metagenomics analysis — the article states NanoPipe does not yet support
  it (implementation is planned), so multi-species mixtures are out of scope.
- Positions need statistically evaluated SNP calls with sophisticated scoring — the
  article notes current SNP detection statistics are not yet sophisticated; use the
  consensus module only for majority-rule bases, not formal SNP calling.
- Input is already a consensus or variant table — this skill operates on per-read
  alignments and would add nothing.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NanoPipe
  role: web-driven pipeline whose alignment-analysis and results-display stages perform
    the per-position counting and emit the consensus FASTA
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST
  role: produces the read-to-target MAF alignments (version 946) that serve as the
    input for nucleotide aggregation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:ff340174839a367397273356c884b5409fc0638805ae20e0faa3b7d27d725bf8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST
  - NanoPipe
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# per-position nucleotide counting

## Summary

Builds a majority-rule consensus sequence for each target chromosome/region by aggregating nucleotide counts from MinION read-to-target alignments (MAF output of LAST 946), emitting gaps where support is insufficient and IUPAC ambiguity codes where two nucleotides are nearly tied.

## When to use

Use when MinION reads have been mapped to target sequences (e.g., via LAST) and a per-target consensus in FASTA format is needed, with explicit coverage and ambiguity handling per position.

## When NOT to use

- The goal is metagenomics analysis — the article states NanoPipe does not yet support it (implementation is planned), so multi-species mixtures are out of scope.
- Positions need statistically evaluated SNP calls with sophisticated scoring — the article notes current SNP detection statistics are not yet sophisticated; use the consensus module only for majority-rule bases, not formal SNP calling.
- Input is already a consensus or variant table — this skill operates on per-read alignments and would add nothing.

## Inputs

- read-to-target alignments in MAF format from LAST version 946
- MinION sequencing reads in FASTA or FASTQ format (aligned prior to this step)
- target sequences (user-specified target database, e.g., a genome or PCR-amplicon regions)

## Outputs

- individual consensus sequence per target chromosome/region in FASTA format

## How to apply

Parse the read-to-target MAF alignments produced by LAST version 946 during the NanoPipe alignment-analysis stage, and for each target chromosome/region count nucleotides at every position across all mapped reads. Apply the majority rule: assign the nucleotide with the higher count to each consensus position. Assign a gap wherever the minimum nucleotide count per position falls below 10 reads, and when counts for any two nucleotides differ by not more than 20%, encode both nucleotides using IUPAC nomenclature. Rationale: the ≥10-read floor guards against low-coverage artifacts, while the 20% tie threshold preserves genuine within-sample polymorphism rather than arbitrarily resolving near-ties. Write the resulting per-target consensus to a FASTA output file viewable and downloadable from the results display stage.

## Related tools

- **NanoPipe** (web-driven pipeline whose alignment-analysis and results-display stages perform the per-position counting and emit the consensus FASTA) — https://github.com/IOB-Muenster/nanopipe2
- **LAST** (produces the read-to-target MAF alignments (version 946) that serve as the input for nucleotide aggregation)

## Examples

```
bash NANOPIPE/check.sh
```

## Evaluation signals

- Consensus FASTA contains gaps at positions supported by fewer than 10 reads.
- Positions where the top two nucleotide counts differ by at most 20% carry IUPAC ambiguity codes rather than a single base.
- Exactly one consensus sequence is emitted per target chromosome/region.
- Every consensus position matches the majority nucleotide count aggregated over all mapped reads.
- Installation sanity check passes via the provided check.sh script before running.

## Limitations

- Positions with fewer than 10 supporting reads become gaps, so very shallow coverage yields gappy consensus sequences.
- The read length limit (e.g., 200 nucleotides) should be set to avoid noise from too-short reads mapping spuriously.
- Consensus is majority-rule only; users wanting more sophisticated SNP statistics are advised to wait for planned future work.
- Originally a web-based tool; local use requires manual installation and code modification.

## Evidence

- [methods] majority rule with gap threshold: "The consensus sequence is calculated based on the majority rule, i.e., the nucleotide with the higher count"
- [methods] minimum count of 10: "The minimum nucleotide count per position should be at least 10 (i.e., at least 10 reads), otherwise a gap will be assigned"
- [methods] IUPAC tie handling: "If the counts for any two nucleotides differ from each other by not more than by 20%, both nucleotides are included"
- [methods] alignment source: "mapping of sequencing reads to the target using version 946 of the LAST sequence aligner"
- [readme] tool purpose per README: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest"
