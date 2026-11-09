---
name: transition-transversion-probability-weighting
description: Decision support for detecting and ranking SNP candidates from MinION
  reads aligned to a target with LAST, when you need transition/transversion-weighted
  polymorphism probabilities from a MAF alignment.
when_to_use_negative:
- Input reads are not from Oxford Nanopore MinION sequencing or have not been aligned
  to a target with LAST, since the weighting and p-error estimation depend on LAST
  reliability scores
- A more sophisticated SNP-calling statistic is required — the article notes current
  detection statistics are not yet sophisticated and more advanced evaluation is planned
  future work
- 'The task is consensus base calling at low coverage: positions with fewer than 10
  reads get a gap assigned, so below this depth the polymorphism table is unreliable'
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3573
tools:
- name: NanoPipe
  role: Web-driven pipeline implementing the polymorphism analysis, probability weighting,
    and polymorphisms table generation
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST
  role: Sequence aligner (version 946) mapping MinION reads to the target; produces
    the MAF alignment and ASCII reliability scores used for p-error estimation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:7c6c099d3a5bb83dc9f414e1da663b277be48b6839ebd2f8023aaf29b4ccd48d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST
  - NanoPipe
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3573
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# transition-transversion-probability-weighting

## Summary

NanoPipe's SNP-candidate scoring method: alternate nucleotides are flagged when they cover ≥20% of reads and ≥30% of maximum contig coverage, then weighted probabilities are computed by multiplying relative nucleotide frequencies by transition (factor 2) versus transversion (factor 1) weights and rescaled so the top candidate has a joint probability of 1.

## When to use

Decision support for detecting and ranking SNP candidates from MinION reads aligned to a target with LAST, when you need transition/transversion-weighted polymorphism probabilities from a MAF alignment.

## When NOT to use

- Input reads are not from Oxford Nanopore MinION sequencing or have not been aligned to a target with LAST, since the weighting and p-error estimation depend on LAST reliability scores
- A more sophisticated SNP-calling statistic is required — the article notes current detection statistics are not yet sophisticated and more advanced evaluation is planned future work
- The task is consensus base calling at low coverage: positions with fewer than 10 reads get a gap assigned, so below this depth the polymorphism table is unreliable

## Inputs

- MAF alignment file from LAST/last-split (e.g. calc.lastalign.maf) of MinION reads mapped to a target sequence
- LAST per-column reliability scores (ASCII-encoded)
- Target/contig sequence with reference bases

## Outputs

- Polymorphisms table listing each SNP candidate with position, reference base, alternate base, transition/transversion-weighted joint probability (maximum = 1), and raw nucleotide counts at the target position

## How to apply

Parse the MAF alignment file (e.g. calc.lastalign.maf) produced by LAST/last-split and tabulate nucleotide counts at each target position across all mapped reads. Flag a position as a polymorphism candidate only if the alternate nucleotide has coverage of at least 20% of all reads, and drop candidates with read coverage below 30% of the maximum coverage in the respective contig. Compute a weighted probability per candidate by multiplying the relative nucleotide frequency by a weight factor (transversions = 1, transitions = 2), then rescale so the maximum joint probability is 1. Assess local alignment quality by estimating p-error over up to 10 nucleotides flanking the SNP (excluding the SNP position) using LAST reliability scores converted from ASCII values. Write filtered, ranked candidates with position, reference base, alternate base, and probability to the polymorphisms table, alongside raw nucleotide counts per position.

## Related tools

- **NanoPipe** (Web-driven pipeline implementing the polymorphism analysis, probability weighting, and polymorphisms table generation) — https://github.com/IOB-Muenster/nanopipe2
- **LAST** (Sequence aligner (version 946) mapping MinION reads to the target; produces the MAF alignment and ASCII reliability scores used for p-error estimation)

## Examples

```
bash NANOPIPE/check.sh && python calc_polymorphisms.py --maf calc.lastalign.maf --min-count 10 --alt-cov 0.20 --max-cov 0.30 --trans-weight 2 --transv-weight 1 --out polymorphisms_table.txt
```

## Evaluation signals

- Polymorphisms table file exists and contains one row per candidate passing both the 20% alternate-coverage and the 30%-of-maximum-contig-coverage filters
- All joint probabilities in the table are rescaled so the top-ranked candidate has a joint probability of exactly 1
- Transitions are weighted twice as heavily (factor 2) as transversions (factor 1), so weighted probabilities reflect this asymmetry
- Raw nucleotide counts are reported for each nucleotide at each candidate target position
- Positions with total read coverage below 10 reads are excluded (gap assigned) and do not appear as SNP candidates

## Limitations

- SNP detection statistics are not yet sophisticated; the authors state 'more sophisticated statistics for SNP detection and evaluation are planned future work'
- Candidates with alternate coverage below 20% of all reads or below 30% of maximum contig coverage are dropped, so low-frequency variants are missed
- Positions with fewer than 10 reads are assigned a gap rather than scored, limiting sensitivity on very low-coverage targets
- Users should set a read-length limit (e.g. 200 nucleotides) to avoid 'noise' from mapping of too-short reads

## Evidence

- [methods] Alternate-coverage threshold: "suggests a polymorphism candidate if an alternate nucleotide has coverage of at least 20% of all reads"
- [methods] Maximum-coverage threshold: "a SNP candidate should have read coverage of at least 30% from the maximum coverage in the respective contig"
- [methods] Transition/transversion weights: "transversions are weighted by the factor of 1 and transitions by the factor of 2"
- [methods] p-error quality assessment: "an assessment of the alignment quality around a potential SNP by estimating a p-error"
- [methods] Minimum count per position: "The minimum nucleotide count per position should be at least 10 (i.e., at least 10 reads), otherwise a gap will be assigned"
- [abstract] Polymorphism output contents: "The results contain alignment statistics, consensus sequence, polymorphisms data, and visualization of the alignment"
- [readme] Tool scope: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest, alignment statistics and information about polymorphisms."
- [discussion] Future-work caveat: "The next step for the NanoPipe project will include more sophisticated statistics for SNP detection and evaluation."
