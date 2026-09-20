---
name: consensus-sequence-generation
description: Use when a FASTA/FASTQ file of Oxford Nanopore MinION reads has been
  aligned to a target database and a majority-rule consensus, per-position nucleotide
  counts, and candidate SNP calls are needed.
when_to_use_negative:
- The analysis goal is metagenomics classification of mixed-species MinION data —
  NanoPipe does not yet support metagenomics analysis.
- The input is already assembled contigs or a finished genome rather than per-read
  MinION alignments — no per-position read counts exist to drive the majority-rule
  consensus.
- Positions covered by fewer than 10 reads are required to be called definitively
  — NanoPipe assigns a gap instead of a nucleotide there.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: NanoPipe
  role: Web/local pipeline whose nanopipe_calc_polymorphism.py computes the per-position
    nucleotide counts, majority-rule consensus, and filtered polymorphism table
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST 946
  role: Aligns the MinION reads to the target with lastal/last-split using MinION-optimized,
    last-train parameters to produce calc.lastalign.maf that feeds consensus calling
- name: samtools
  role: Converts maf-convert SAM output into sorted, indexed calc.bam/calc.bam.bai
    used as the alignment basis for polymorphism computation and IGV browsing
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:6c34cf8fca8394e35015f7eed992e4a0957aa3323c8aef682a8b461db0fe5931
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST 946
  - NanoPipe
  - samtools
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# consensus-sequence-generation

## Summary

Generates a majority-rule consensus sequence and per-position nucleotide counts from MinION nanopore reads aligned to a target with NanoPipe (LAST + samtools + nanopipe_calc_polymorphism.py), flagging ambiguous positions with IUPAC codes and candidate SNPs for downstream evaluation.

## When to use

Use when a FASTA/FASTQ file of Oxford Nanopore MinION reads has been aligned to a target database and a majority-rule consensus, per-position nucleotide counts, and candidate SNP calls are needed.

## When NOT to use

- The analysis goal is metagenomics classification of mixed-species MinION data — NanoPipe does not yet support metagenomics analysis.
- The input is already assembled contigs or a finished genome rather than per-read MinION alignments — no per-position read counts exist to drive the majority-rule consensus.
- Positions covered by fewer than 10 reads are required to be called definitively — NanoPipe assigns a gap instead of a nucleotide there.

## Inputs

- MinION nanopore read query file in FASTA or FASTQ format (e.g. input.query)
- LAST target database built with lastdb from a target FASTA (e.g. NANOPIPE/targets/target)
- Local SNP annotation database for the organism (e.g. PlasmoDB P. falciparum SNP data via -s plasf, or dbSNP for human)

## Outputs

- calc.nuccounts: per-position nucleotide count table over the alignment
- calc.nuccounts.1.poly: filtered polymorphism table with candidate SNPs and dbSNP/PlasmoDB annotations
- majority-rule consensus sequence with IUPAC ambiguity codes at mixed positions
- calc.bam and calc.bam.bai (sorted, indexed alignments for IGV browsing)

## How to apply

Align the MinION query reads against the LAST-built target database with lastal/last-split using MinION-trained parameters (input.substmatrix), convert calc.lastalign.maf to SAM via maf-convert, and sort/index to calc.bam with samtools. Then run nanopipe_calc_polymorphism.py to compute per-position nucleotide counts (calc.nuccounts) and apply the consensus rules: at least 10 reads per position (otherwise a gap is assigned), the majority-rule nucleotide with the higher count is chosen, and when the top two nucleotide counts differ by no more than 20% both are included as an IUPAC ambiguity code. In parallel, polymorphism candidates are those with alternate coverage of at least 20% of all reads and read coverage of at least 30% of the maximum contig coverage, with transitions weighted 2x versus transversions and a p-error assessed over ±10 nt around each candidate SNP. Validate by confirming expected mutations (e.g. PfCRT/K-propeller SNPs with PlasmoDB matches, or H1975 EGFR transcript-region mutations) appear as true SNPs with joint probability 1 in calc.nuccounts.1.poly.

## Related tools

- **NanoPipe** (Web/local pipeline whose nanopipe_calc_polymorphism.py computes the per-position nucleotide counts, majority-rule consensus, and filtered polymorphism table) — https://github.com/IOB-Muenster/nanopipe2
- **LAST 946** (Aligns the MinION reads to the target with lastal/last-split using MinION-optimized, last-train parameters to produce calc.lastalign.maf that feeds consensus calling)
- **samtools** (Converts maf-convert SAM output into sorted, indexed calc.bam/calc.bam.bai used as the alignment basis for polymorphism computation and IGV browsing)

## Examples

```
nanopipe_calc.pl -q input.query -t input.target && nanopipe_calc_polymorphism.py calc -s plasf
```

## Evaluation signals

- Mapped-read fraction sanity check: the P. falciparum test case mapped 99.9% of query reads; substantially lower rates suggest a target/preprocessing problem.
- Consensus rule check: every position with fewer than 10 reads is output as a gap, and IUPAC codes appear only where the top two nucleotide counts differ by ≤20%.
- Polymorphism table check: reported candidate SNPs must have alternate coverage ≥20% of all reads and coverage ≥30% of the maximum contig coverage.
- Biological validation: expected known mutations (e.g. H1975 EGFR exon 17-22 mutations or PfCRT/K-propeller SNPs) appear as true SNPs with joint probability 1 in calc.nuccounts.1.poly.
- Annotation check: PlasmoDB lookup returns the expected pattern, e.g. 'PlasmoDB local: 3 match(es); 0 artifact(s)'.

## Limitations

- SNP detection statistics are not yet sophisticated; the authors state the next step includes more sophisticated statistics for SNP detection and evaluation.
- A minimum of 10 reads per position is required; lower coverage positions are assigned a gap, so low-depth amplicons yield incomplete consensus.
- Read length limits (e.g. 200 nt) should be set to avoid 'noise' from too-short reads mapping spuriously.

## Evidence

- [methods] The consensus sequence is calculated based on the majority rule, i.e., the nucleotide with the higher count: "The consensus sequence is calculated based on the majority rule, i.e., the nucleotide with the higher count"
- [methods] minimum 10 reads rule: "The minimum nucleotide count per position should be at least 10 (i.e., at least 10 reads), otherwise a gap will be assigned"
- [methods] IUPAC ambiguity rule: "If the counts for any two nucleotides differ from each other by not more than by 20%, both nucleotides are included"
- [abstract] outputs of NanoPipe: "The results contain alignment statistics, consensus sequence, polymorphisms data, and visualization of the alignment"
- [readme] NanoPipe purpose (README): "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest, alignment statistics and information about polymorphisms."
