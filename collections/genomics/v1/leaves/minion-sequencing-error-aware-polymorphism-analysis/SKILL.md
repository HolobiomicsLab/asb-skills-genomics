---
name: minion-sequencing-error-aware-polymorphism-analysis
description: Use when analyzing Oxford Nanopore MinION reads (FASTA/FASTQ) aligned
  against a target sequence to detect and rank SNP candidates, and you need error-aware
  polymorphism probabilities instead of naive majority-rule consensus calls.
when_to_use_negative:
- You need sophisticated SNP statistics or variant calling beyond NanoPipe's current
  scoring — the authors state more sophisticated statistics for SNP detection and
  evaluation are planned future work
- Your task is metagenomics analysis — NanoPipe does not yet support it (implementation
  planned)
- Positions have read coverage below 10 reads — a gap is assigned rather than a nucleotide/consensus
  call, so polymorphism scoring is inappropriate there
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NanoPipe
  role: Web server / local pipeline implementing the coverage filters, transition/transversion-weighted
    probability scoring, and p-error quality assessment to produce the polymorphisms
    table
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST 946
  role: Aligns MinION reads to the target with MinION-adjusted parameters, producing
    the MAF file and reliability scores consumed by the polymorphism analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:dc4f31ea1980203528a0b6dfeb541a5a40793927f482a1b42b744ce4e9f2e2f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST 946
  - NanoPipe
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# minion-sequencing-error-aware-polymorphism-analysis

## Summary

Detects SNP candidates from MinION nanopore reads aligned to a target by combining coverage filters (20% alternate coverage, 30% of maximum contig coverage), transition/transversion-weighted probability scoring, and p-error-based alignment quality assessment, producing a ranked polymorphisms table. This is NanoPipe's core polymorphism-analysis approach, adjusted to the specifics of MinION sequencing technology.

## When to use

Use when analyzing Oxford Nanopore MinION reads (FASTA/FASTQ) aligned against a target sequence to detect and rank SNP candidates, and you need error-aware polymorphism probabilities instead of naive majority-rule consensus calls.

## When NOT to use

- You need sophisticated SNP statistics or variant calling beyond NanoPipe's current scoring — the authors state more sophisticated statistics for SNP detection and evaluation are planned future work
- Your task is metagenomics analysis — NanoPipe does not yet support it (implementation planned)
- Positions have read coverage below 10 reads — a gap is assigned rather than a nucleotide/consensus call, so polymorphism scoring is inappropriate there

## Inputs

- MinION reads in FASTA or FASTQ format (e.g. raw 1D reads)
- Target reference sequence(s) (e.g. a FASTA genome or gene target such as P. falciparum 3D7 or EGFR exons 17-22)
- MAF alignment file produced by LAST/last-split (calc.lastalign.maf)
- LAST reliability scores (ASCII-encoded) for p-error estimation

## Outputs

- Polymorphisms table listing SNP candidates with position, reference base, alternate base, joint probabilities (maximum = 1), and raw counts for each nucleotide at the target position
- Consensus sequence (majority-rule with IUPAC ambiguity codes when counts differ by not more than 20%)
- Alignment statistics and alignment visualization

## How to apply

Parse the MAF alignment file produced by LAST (version 946, e.g. calc.lastalign.maf) and tabulate nucleotide counts at each target position across all mapped reads. Mark a position as a polymorphism candidate only if the alternate nucleotide covers at least 20% of all reads, and drop candidates whose read coverage falls below 30% of the maximum coverage in the respective contig. Score each surviving candidate by multiplying its relative nucleotide frequency by a weight factor (transversions weighted by 1, transitions by 2), then rescale probabilities so the maximum equals 1. Assess alignment quality around each candidate by estimating the p-error over up to 10 nucleotides flanking (but excluding) the SNP position, using LAST reliability scores converted from ASCII values. Finally write the filtered, ranked candidates with position, reference base, alternate base, probability, and raw nucleotide counts to the polymorphisms table file.

## Related tools

- **NanoPipe** (Web server / local pipeline implementing the coverage filters, transition/transversion-weighted probability scoring, and p-error quality assessment to produce the polymorphisms table) — https://github.com/IOB-Muenster/nanopipe2
- **LAST 946** (Aligns MinION reads to the target with MinION-adjusted parameters, producing the MAF file and reliability scores consumed by the polymorphism analysis)

## Examples

```
bash NANOPIPE/check.sh
```

## Evaluation signals

- The polymorphisms table exists and contains one row per candidate passing both the 20% alternate-nucleotide coverage filter and the 30%-of-maximum-contig-coverage filter
- All reported joint probabilities are rescaled so the top candidate has a probability of exactly 1
- Transitions receive twice the weight of transversions in the candidate probabilities
- p-error estimates around each candidate are computed from up to 10 flanking nucleotides (excluding the SNP itself) across all read alignments
- In the P. falciparum validation case, ~99.9% of query reads map and the expected mutations are detected

## Limitations

- SNP detection statistics are not yet sophisticated; more sophisticated statistics for SNP detection and evaluation are planned as future work
- Too-short mapped reads create noise; a read length limit (e.g. 200 nucleotides) can be set to avoid this
- Coverage below 10 reads per position results in a gap instead of a nucleotide call
- Metagenomics analysis is not supported (planned)

## Evidence

- [methods] Alternate-nucleotide coverage filter: "suggests a polymorphism candidate if an alternate nucleotide has coverage of at least 20% of all reads"
- [methods] Maximum-contig-coverage filter: "a SNP candidate should have read coverage of at least 30% from the maximum coverage in the respective contig"
- [methods] Transition/transversion weighting: "transversions are weighted by the factor of 1 and transitions by the factor of 2"
- [methods] p-error quality assessment: "an assessment of the alignment quality around a potential SNP by estimating a p-error"
- [readme] Output scope of NanoPipe: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest, alignment statistics and information about polymorphisms."
- [abstract] Result components: "The results contain alignment statistics, consensus sequence, polymorphisms data, and visualization of the alignment"
