---
name: snp-candidate-coverage-filtering
description: 'Use when calling SNPs from MinION nanopore reads aligned to a target
  with LAST: coverage thresholds must separate genuine polymorphism candidates from
  mapping noise before consensus/probability reporting.'
when_to_use_negative:
- The alignment stage has not been run — no MAF file or per-position nucleotide counts
  exist yet, so the coverage filters cannot be evaluated
- The goal is metagenomics or multi-species variant analysis; NanoPipe's discussion
  states metagenomics analysis is not yet supported
- Positions are being filtered for consensus calling rather than SNP detection — in
  that case the 10-read minimum and 20% IUPAC ambiguity rules govern, not the polymorphism
  candidate thresholds
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3573
tools:
- name: NanoPipe
  role: Web-driven pipeline that implements the coverage filters and writes the polymorphisms
    table
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST
  role: Produces the version-946 MAF alignment of MinION reads to the target from
    which per-position nucleotide coverage is computed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:0df072c311d2487f5d1c6b244ca9d643d9858f4000db686f92569c929a2f2880
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# snp-candidate-coverage-filtering

## Summary

A coverage-based filtering scheme from NanoPipe that marks positions in a LAST/last-split MAF alignment as SNP candidates only when the alternate nucleotide has ≥20% coverage of all reads and the position has ≥30% of the maximum coverage in its contig, before transition/transversion-weighted probability ranking.

## When to use

Use when calling SNPs from MinION nanopore reads aligned to a target with LAST: coverage thresholds must separate genuine polymorphism candidates from mapping noise before consensus/probability reporting.

## When NOT to use

- The alignment stage has not been run — no MAF file or per-position nucleotide counts exist yet, so the coverage filters cannot be evaluated
- The goal is metagenomics or multi-species variant analysis; NanoPipe's discussion states metagenomics analysis is not yet supported
- Positions are being filtered for consensus calling rather than SNP detection — in that case the 10-read minimum and 20% IUPAC ambiguity rules govern, not the polymorphism candidate thresholds

## Inputs

- MAF alignment file from LAST/last-split (e.g. calc.lastalign.maf) of MinION reads against a target sequence
- MinION read set in FASTA or FASTQ format
- Target reference sequence(s)

## Outputs

- Filtered list of SNP candidate positions passing the 20% alternate-coverage and 30%-of-maximum-contig-coverage filters
- Polymorphisms table with per-candidate position, reference base, alternate base, raw nucleotide counts, and joint probability (maximum = 1)

## How to apply

Parse the MAF alignment (e.g. calc.lastalign.maf) produced by LAST version 946 and tabulate raw nucleotide counts at each target position from all mapped reads. Apply the first NanoPipe filter: accept a position as a polymorphism candidate only if the alternate nucleotide has coverage of at least 20% of all reads. Apply the second filter: drop candidates whose total read coverage is below 30% of the maximum coverage in the respective contig, which removes weakly covered positions prone to sequencing error. For reference-consensus contexts, remember the separate NanoPipe rule that positions with fewer than 10 reads receive a gap, and IUPAC ambiguity codes are emitted when two nucleotide counts differ by ≤20%. Surviving candidates are then ranked by transition/transversion-weighted probabilities (transitions weighted 2, transversions 1), rescaled so the maximum is 1, and written to the polymorphisms table with position, reference base, alternate base, and probability.

## Related tools

- **NanoPipe** (Web-driven pipeline that implements the coverage filters and writes the polymorphisms table) — https://github.com/IOB-Muenster/nanopipe2
- **LAST** (Produces the version-946 MAF alignment of MinION reads to the target from which per-position nucleotide coverage is computed)

## Evaluation signals

- Every reported candidate in the polymorphisms table has an alternate nucleotide with coverage ≥20% of all reads at that position
- No reported candidate has total read coverage below 30% of the maximum coverage in its contig
- Joint probabilities in the polymorphisms table are rescaled so the top candidate equals 1
- Positions with fewer than 10 nucleotide counts are excluded (gap assigned) rather than appearing as candidates
- The polymorphisms table file exists and contains one row per candidate passing both filters, with position, reference base, alternate base, and probability columns

## Limitations

- NanoPipe's SNP detection statistics are not yet sophisticated; more sophisticated statistics for SNP detection and evaluation are planned as future work
- Candidates below 30% of maximum contig coverage are discarded, so true variants in low-coverage regions will be missed
- The minimum of 10 reads per position means positions with less coverage are assigned gaps and cannot yield candidates

## Evidence

- [methods] 20% alternate-coverage filter: "suggests a polymorphism candidate if an alternate nucleotide has coverage of at least 20% of all reads"
- [methods] 30%-of-maximum contig coverage filter: "a SNP candidate should have read coverage of at least 30% from the maximum coverage in the respective contig"
- [methods] minimum 10 reads per position: "The minimum nucleotide count per position should be at least 10 (i.e., at least 10 reads), otherwise a gap will be assigned"
- [methods] transition/transversion weighting applied after filtering: "transversions are weighted by the factor of 1 and transitions by the factor of 2"
- [abstract] polymorphisms data among NanoPipe outputs: "The results contain alignment statistics, consensus sequence, polymorphisms data, and visualization of the alignment"
