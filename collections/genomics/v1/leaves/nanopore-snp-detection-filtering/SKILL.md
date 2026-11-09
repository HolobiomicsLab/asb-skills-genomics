---
name: nanopore-snp-detection-filtering
description: Use when MinION amplicon or whole-genome reads (FASTA/FASTQ) must be
  mapped to a reference and candidate SNPs called with explicit coverage thresholds,
  e.g., targeted Plasmodium falciparum or EGFR amplicon resequencing.
when_to_use_negative:
- The goal is metagenomics analysis — the article states metagenomics is not yet supported
  and only planned as future work.
- Input reads are too short and would map as 'noise' without setting a read length
  limit (e.g., 200 nucleotides).
- A coverage-of-reads count table already exists and only downstream consensus/statistics
  are needed — rerunning the LAST alignment stage would be redundant.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: NanoPipe
  role: Web/local pipeline that aligns MinION reads and computes nucleotide counts,
    consensus, and the filtered polymorphism table
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST 946
  role: Sequence aligner used for read-to-target mapping (lastal with MinION-adjusted
    parameters, piped into last-split)
- name: samtools
  role: Converts and processes the maf-convert SAM output (view -bS, sort, index)
    into calc.bam/calc.bam.bai
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:f5e1002ce3c50c39042b2f917b0cc201f40088f40da259a6f59ded0553f329c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST 946
  - NanoPipe
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# nanopore-snp-detection-filtering

## Summary

Detect and filter SNPs from Oxford Nanopore MinION read alignments using NanoPipe's polymorphism module, which builds a per-position nucleotide count table, a majority-rule consensus, and a weighted polymorphism table with coverage- and count-based cutoffs.

## When to use

Use when MinION amplicon or whole-genome reads (FASTA/FASTQ) must be mapped to a reference and candidate SNPs called with explicit coverage thresholds, e.g., targeted Plasmodium falciparum or EGFR amplicon resequencing.

## When NOT to use

- The goal is metagenomics analysis — the article states metagenomics is not yet supported and only planned as future work.
- Input reads are too short and would map as 'noise' without setting a read length limit (e.g., 200 nucleotides).
- A coverage-of-reads count table already exists and only downstream consensus/statistics are needed — rerunning the LAST alignment stage would be redundant.

## Inputs

- MinION query reads in FASTA or FASTQ format (e.g., nanopipe2 testcase input.query from ENA study PRJEB30433)
- Target reference FASTA with a LAST database built via lastdb (e.g., plasmodium.tgz / target.fasta, Plasmodium falciparum 3D7)
- Trained LAST substitution matrix (input.substmatrix)
- Local PlasmoDB SNP database for P. falciparum 3D7 (all 14 chromosomes)

## Outputs

- calc.lastalign.maf (LAST alignment MAF)
- calc.bam and calc.bam.bai (sorted, indexed alignments for IGV)
- calc.nuccounts.1 (per-position nucleotide count table)
- calc.nuccounts.1.poly (filtered polymorphism table with consensus, SNP candidates, and PlasmoDB match/artifact counts)

## How to apply

Map reads to the prepared target with LAST (lastal with parameters -A 15 -B 4 -P 2 -Q 0 -S 1 -a 15 -b 3 -k 2 and a trained input.substmatrix, piped into last-split), convert the MAF to SAM/BAM (maf-convert, samtools view -bS, sort, index) for IGV browsing. Run nanopipe_calc_polymorphism.py with the local PlasmoDB SNP database (e.g., -s plasf) to compute per-position nucleotide counts and a majority-rule consensus: positions with fewer than 10 reads get a gap, and if the top two nucleotides differ by ≤20% both are encoded with IUPAC codes. Filter polymorphism candidates requiring an alternate nucleotide coverage ≥20% of all reads and read coverage ≥30% of the maximum contig coverage, weighting transitions 2x versus transversions (factor 1) and estimating p-error around each potential SNP to assess alignment quality. Judge success by mapping rate (~99.9% in the P. falciparum test case) and recovery of the expected PfCRT/K-propeller mutations with PlasmoDB matches reported in calc.nuccounts.1.poly.

## Related tools

- **NanoPipe** (Web/local pipeline that aligns MinION reads and computes nucleotide counts, consensus, and the filtered polymorphism table) — https://github.com/IOB-Muenster/nanopipe2
- **LAST 946** (Sequence aligner used for read-to-target mapping (lastal with MinION-adjusted parameters, piped into last-split))
- **samtools** (Converts and processes the maf-convert SAM output (view -bS, sort, index) into calc.bam/calc.bam.bai)

## Examples

```
nanopipe_calc.pl -q input.query -t input.target && nanopipe_calc_polymorphism.py -s plasf
```

## Evaluation signals

- Mapping rate of query reads approaches ~99.9% as reported for the Plasmodium test case.
- calc.nuccounts.1.poly reports the expected PfCRT/K-propeller mutations with PlasmoDB matches (e.g., 'PlasmoDB local: 3 match(es); 0 artifact(s)').
- Every consensus position carries at least 10 reads; otherwise a gap is assigned — verify no position with <10 reads is called.
- Alternate-allele coverage is ≥20% of all reads and candidate sites have coverage ≥30% of the maximum contig coverage.
- Full analysis of the 21 MB FASTA query against the P. falciparum genome completes in under ~4 minutes on a typical run.

## Limitations

- Current SNP detection statistics are not yet sophisticated; more sophisticated statistics for SNP detection and evaluation are planned as future work.
- Metagenomics analysis is not yet supported.
- Positions with fewer than 10 reads are assigned gaps, so low-coverage regions yield no calls.
- Too-short reads can create mapping 'noise'; a read length limit (e.g., 200 nucleotides) must be set to avoid this.

## Evidence

- [results] NanoPipe succeeded in mapping 99.9% of all query reads and detecting the expected mutations: "NanoPipe succeeded in mapping 99.9% of all query reads and detecting the expected mutations"
- [methods] minimum nucleotide count: "The minimum nucleotide count per position should be at least 10 (i.e., at least 10 reads), otherwise a gap will be assigned"
- [methods] alternate coverage filter: "suggests a polymorphism candidate if an alternate nucleotide has coverage of at least 20% of all reads"
- [methods] transition/transversion weighting: "transversions are weighted by the factor of 1 and transitions by the factor of 2"
- [results] read length limit: "To avoid “noise,” i.e., mapping of too short reads, the user can set the read length limit, e.g., to 200 nucleotides"
- [readme] README scope: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest, alignment statistics and information about polymorphisms."
