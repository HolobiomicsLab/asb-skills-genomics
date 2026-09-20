---
name: nucleotide-count-tabulation-from-maf-alignments
description: Use when a LAST/last-split MAF alignment (e.g. calc.lastalign.maf) of
  Oxford Nanopore MinION reads against a target sequence is available and per-position
  nucleotide counts are needed for consensus calling or polymorphism/SNP candidate
  detection.
when_to_use_negative:
- Variant data already exists as a finished polymorphisms/variant table — recounting
  from the MAF would duplicate the per-position counts already reported
- The goal is metagenomic or mixed-community analysis — the authors state metagenomics
  analysis is not yet supported and implementation is planned
- Reads are not Oxford Nanopore MinION reads — NanoPipe's alignment parameters and
  count/coverage thresholds are tuned to MinION technology specifics
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3268
tools:
- name: NanoPipe
  role: Pipeline whose polymorphism-analysis stage parses the MAF alignment, tabulates
    nucleotide counts at each target position, and writes the filtered, probability-ranked
    polymorphisms table
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST
  role: Sequence aligner (version 946) whose last-split MAF output (e.g. calc.lastalign.maf)
    supplies the read-to-target alignments from which nucleotide counts are tabulated;
    its reliability (ASCII) scores feed the p-error flank assessment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:d54b5e841bbbb3cea12a851ba7d41761ecd740d92005632eef0697c433a462e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST
  - NanoPipe
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3268
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# nucleotide-count-tabulation-from-maf-alignments

## Summary

Tabulates raw nucleotide counts at every position of a target sequence from a LAST/last-split MAF alignment of Oxford Nanopore MinION reads; these counts underpin NanoPipe's majority-rule consensus, IUPAC ambiguity calls, and transition/transversion-weighted SNP candidate scoring.

## When to use

Use when a LAST/last-split MAF alignment (e.g. calc.lastalign.maf) of Oxford Nanopore MinION reads against a target sequence is available and per-position nucleotide counts are needed for consensus calling or polymorphism/SNP candidate detection.

## When NOT to use

- Variant data already exists as a finished polymorphisms/variant table — recounting from the MAF would duplicate the per-position counts already reported
- The goal is metagenomic or mixed-community analysis — the authors state metagenomics analysis is not yet supported and implementation is planned
- Reads are not Oxford Nanopore MinION reads — NanoPipe's alignment parameters and count/coverage thresholds are tuned to MinION technology specifics

## Inputs

- MAF alignment file produced by LAST/last-split (e.g. calc.lastalign.maf) mapping MinION reads to the target
- Target reference sequence(s) in FASTA
- MinION read set in FASTA or FASTQ format (upstream input that produces the MAF)

## Outputs

- Per-position nucleotide count table for the target (raw counts of each nucleotide across all mapped reads)
- Majority-rule consensus sequence with IUPAC ambiguity codes where top-two nucleotide counts differ by ≤20% (gap where counts < 10)
- Polymorphisms table rows: SNP candidates with position, reference base, alternate base, rescaled joint probability (maximum = 1), and raw nucleotide counts at the position

## How to apply

Parse the MAF file written by LAST (version 946) after last-split mapping of MinION reads to the target (e.g. calc.lastalign.maf) and accumulate raw counts of each nucleotide at every target position across all mapped reads. Apply the paper's 10-read floor: a position with fewer than 10 nucleotide counts is assigned a gap, and otherwise the consensus is taken by majority rule (the nucleotide with the higher count). Where the counts for any two nucleotides differ by not more than 20%, include both in the consensus via the IUPAC ambiguity code. Reuse the same counts for SNP calling: mark a position as a polymorphism candidate only if the alternate nucleotide has coverage of at least 20% of all reads and the position has read coverage of at least 30% of the maximum coverage in the respective contig. Score candidates by relative nucleotide frequency times the weight (transversions = 1, transitions = 2), rescale all probabilities so the maximum joint probability is 1, and support the p-error assessment over up to 10 nucleotides flanking the SNP (excluding the SNP itself) using LAST reliability scores converted from ASCII values. Rationale: the count floor and the 20%/30% coverage filters prevent low-support positions from surfacing as candidates, while the transition/transversion weighting is the authors' scheme for ranking candidates by relative frequency.

## Related tools

- **NanoPipe** (Pipeline whose polymorphism-analysis stage parses the MAF alignment, tabulates nucleotide counts at each target position, and writes the filtered, probability-ranked polymorphisms table) — https://github.com/IOB-Muenster/nanopipe2
- **LAST** (Sequence aligner (version 946) whose last-split MAF output (e.g. calc.lastalign.maf) supplies the read-to-target alignments from which nucleotide counts are tabulated; its reliability (ASCII) scores feed the p-error flank assessment)

## Examples

```
bash NANOPIPE/check.sh
```

## Evaluation signals

- The polymorphisms table exists and contains one row per candidate passing both the ≥20% alternate-nucleotide coverage and ≥30%-of-maximum-contig-coverage filters
- Raw counts for each nucleotide at the target position are reported for every candidate and match a manual recount of the MAF alignment columns
- Joint probabilities are rescaled so the top candidate has a joint probability of exactly 1 (maximum = 1)
- Positions with fewer than 10 reads receive a gap in the consensus rather than a nucleotide call
- Where counts for two nucleotides differ by not more than 20%, the consensus position encodes both nucleotides via the IUPAC ambiguity code

## Limitations

- Positions with a nucleotide count below 10 are assigned a gap, so low-coverage targets yield incomplete count and consensus output
- Mapping of too-short reads adds 'noise'; the user must set a read length limit (e.g., 200 nucleotides) to suppress it
- SNP detection statistics are deliberately simple (weighted frequencies plus p-error flanks); the authors note more sophisticated statistics for SNP detection and evaluation are planned future work

## Evidence

- [methods] mapping of sequencing reads to the target using version 946 of the LAST sequence aligner: "mapping of sequencing reads to the target using version 946 of the LAST sequence aligner"
- [methods] minimum nucleotide count per position: "The minimum nucleotide count per position should be at least 10 (i.e., at least 10 reads), otherwise a gap will be assigned"
- [methods] majority rule consensus: "The consensus sequence is calculated based on the majority rule, i.e., the nucleotide with the higher count"
- [methods] IUPAC ambiguity inclusion: "If the counts for any two nucleotides differ from each other by not more than by 20%, both nucleotides are included"
- [methods] 20% alternate coverage filter: "suggests a polymorphism candidate if an alternate nucleotide has coverage of at least 20% of all reads"
- [methods] 30% of maximum contig coverage filter: "a SNP candidate should have read coverage of at least 30% from the maximum coverage in the respective contig"
- [methods] transition/transversion weighting: "transversions are weighted by the factor of 1 and transitions by the factor of 2"
- [abstract] outputs include consensus and polymorphisms data: "The results contain alignment statistics, consensus sequence, polymorphisms data, and visualization of the alignment"
- [readme] README scope of outputs: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest, alignment statistics and information about polymorphisms."
- [results] read length limit to avoid short-read noise: "To avoid “noise,” i.e., mapping of too short reads, the user can set the read length limit, e.g., to 200 nucleotides"
- [discussion] SNP statistics are simple, future work planned: "The next step for the NanoPipe project will include more sophisticated statistics for SNP detection and evaluation."
