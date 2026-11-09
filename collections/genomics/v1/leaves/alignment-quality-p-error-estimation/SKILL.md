---
name: alignment-quality-p-error-estimation
description: Decision support for prioritizing SNP candidates detected in MinION read
  alignments (LAST/last-split MAF) when variant calls must be weighted by local alignment
  quality rather than raw coverage alone.
when_to_use_negative:
- The goal is consensus sequence calling only — for the majority-rule consensus NanoPipe
  uses a different rule (minimum 10 reads per position; IUPAC ambiguity codes when
  two nucleotides differ by ≤20%), not the SNP p-error filter
- The input is metagenomic data — the article states metagenomics analysis is not
  yet supported in NanoPipe
- Reads are very short/noisy fragments; NanoPipe lets users set a read length limit
  (e.g. 200 nucleotides) to avoid noise from too-short reads, and candidates below
  30% of maximum contig coverage are dropped regardless
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_2885
tools:
- name: NanoPipe
  role: Web server / local pipeline that implements the polymorphism-analysis module,
    including the p-error alignment-quality assessment and weighted SNP scoring
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST
  role: Sequence aligner (version 946) producing the MAF alignments and reliability
    scores whose ASCII-converted values feed the p-error estimation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:239c8d80cc13c6a5b76dc85ab0b47f9739fd4396438bd8e32a8ed5aa8bb350f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST
  - NanoPipe
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_2885
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# alignment-quality-p-error-estimation

## Summary

A NanoPipe filter step that assesses alignment quality around each SNP candidate by estimating a p-error from LAST reliability scores over a window of up to 10 nucleotides flanking the variant (excluding the SNP position itself). It ranks transition/transversion-weighted polymorphism candidates so that confidence reflects local alignment reliability of noisy MinION reads.

## When to use

Decision support for prioritizing SNP candidates detected in MinION read alignments (LAST/last-split MAF) when variant calls must be weighted by local alignment quality rather than raw coverage alone.

## When NOT to use

- The goal is consensus sequence calling only — for the majority-rule consensus NanoPipe uses a different rule (minimum 10 reads per position; IUPAC ambiguity codes when two nucleotides differ by ≤20%), not the SNP p-error filter
- The input is metagenomic data — the article states metagenomics analysis is not yet supported in NanoPipe
- Reads are very short/noisy fragments; NanoPipe lets users set a read length limit (e.g. 200 nucleotides) to avoid noise from too-short reads, and candidates below 30% of maximum contig coverage are dropped regardless

## Inputs

- MAF alignment file from LAST/last-split (e.g. calc.lastalign.maf) of MinION reads against the target sequence
- LAST per-character reliability scores (ASCII-encoded) from the read alignments
- Per-position nucleotide counts across all mapped reads

## Outputs

- Polymorphisms table listing each filtered SNP candidate with position, reference base, alternate base, joint probability (maximum = 1), and raw nucleotide counts

## How to apply

After mapping Oxford Nanopore MinION reads to the target with LAST version 946 (producing calc.lastalign.maf), mark positions as polymorphism candidates when the alternate nucleotide has coverage of at least 20% of all reads and read coverage of at least 30% of the maximum coverage in the respective contig. For each surviving candidate, compute a weighted probability by multiplying the relative nucleotide frequency by a custom factor (transversions weighted by 1, transitions by 2), then rescale all probabilities so the maximum is 1. To account for alignment quality, estimate a p-error over a region of up to 10 nucleotides before and after the SNP position (excluding the SNP itself) across all read alignments, converting LAST reliability scores from their ASCII values. Write position, reference base, alternate base, joint probability, and raw per-nucleotide counts to the polymorphisms table; the top-ranked candidate must carry a joint probability of exactly 1.

## Related tools

- **NanoPipe** (Web server / local pipeline that implements the polymorphism-analysis module, including the p-error alignment-quality assessment and weighted SNP scoring) — https://github.com/IOB-Muenster/nanopipe2
- **LAST** (Sequence aligner (version 946) producing the MAF alignments and reliability scores whose ASCII-converted values feed the p-error estimation)

## Evaluation signals

- Every row in the polymorphisms table corresponds to a candidate whose alternate nucleotide had ≥20% of all reads and whose coverage was ≥30% of the maximum coverage in its contig
- All joint probabilities are rescaled to a [0,1] range with the top candidate equal to exactly 1
- Positions with fewer than 10 supporting reads are assigned a gap rather than reported as candidates
- Transitions receive twice the weight of transversions in the computed probabilities
- The p-error is computed over up to 10 nucleotides before and after each SNP position, excluding the SNP position itself

## Limitations

- The article notes current SNP detection statistics are not yet sophisticated; more sophisticated statistics for SNP detection and evaluation are planned future work
- The p-error window covers only up to 10 nucleotides on each side of the SNP, so quality signals beyond that window are not considered
- Candidates below 30% of maximum contig coverage are dropped, so true variants in low-coverage contigs will be missed

## Evidence

- [methods] p-error alignment quality assessment: "an assessment of the alignment quality around a potential SNP by estimating a p-error"
- [methods] transition/transversion weighting: "transversions are weighted by the factor of 1 and transitions by the factor of 2"
- [methods] coverage filters on candidates: "a SNP candidate should have read coverage of at least 30% from the maximum coverage in the respective contig"
- [discussion] future statistical sophistication: "The next step for the NanoPipe project will include more sophisticated statistics for SNP detection and evaluation."
- [readme] polymorphisms in NanoPipe output: "it provides alignments to any target of interest, alignment statistics and information about polymorphisms"
