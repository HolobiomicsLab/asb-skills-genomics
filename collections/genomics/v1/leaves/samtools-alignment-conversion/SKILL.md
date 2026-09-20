---
name: samtools-alignment-conversion
description: Use when LAST (lastal/last-split) has produced a .maf alignment of nanopore
  reads and you need a sorted, indexed BAM (calc.sam, calc.bam, calc.bam.bai) as input
  for downstream SNP/consensus computation.
when_to_use_negative:
- Input is already a sorted, indexed BAM — re-running maf-convert/samtools conversion
  is unnecessary
- Reads have not yet been aligned to a target with LAST (no MAF exists); run lastal/last-split
  first
- Metagenomics analysis is intended — the article states NanoPipe does not yet support
  metagenomics
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: NanoPipe
  role: Pipeline that chains maf-convert and samtools conversion steps and consumes
    the BAM for polymorphism calculation via nanopipe_calc_polymorphism.py
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST 946
  role: Produces the calc.lastalign.maf input; its maf-convert utility performs the
    MAF-to-SAM conversion
- name: samtools
  role: Sorts calc.sam and indexes the BAM (calc.bam, calc.bam.bai); listed as a required
    installation step in NanoPipe
- name: MinION
  role: Source instrument generating the nanopore query reads being converted into
    alignment formats
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:38f4b2c7f45a36aef65becfb2c9d061240759aa746dbd0a241d1a26fa6cce5d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST 946
  - MinION
  - NanoPipe
  - samtools
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# samtools-alignment-conversion

## Summary

Convert LAST aligner MAF output for MinION nanopore reads into sorted, indexed BAM with LAST's maf-convert and samtools, so that downstream polymorphism analysis in NanoPipe can operate on standard alignment formats.

## When to use

Use when LAST (lastal/last-split) has produced a .maf alignment of nanopore reads and you need a sorted, indexed BAM (calc.sam, calc.bam, calc.bam.bai) as input for downstream SNP/consensus computation.

## When NOT to use

- Input is already a sorted, indexed BAM — re-running maf-convert/samtools conversion is unnecessary
- Reads have not yet been aligned to a target with LAST (no MAF exists); run lastal/last-split first
- Metagenomics analysis is intended — the article states NanoPipe does not yet support metagenomics

## Inputs

- calc.lastalign.maf — MAF-format read alignments from LAST version 946 (lastal/last-split with last-train parameters)
- MinION query reads in FASTA or FASTQ format
- Target database built with lastdb from a target FASTA (e.g., EGFR reference from hg38/RefSeq GCF_000001405.38)

## Outputs

- calc.sam — SAM-format alignment
- calc.bam — sorted BAM alignment
- calc.bam.bai — BAM index
- Downstream: calc.nuccounts.1.poly with per-position nucleotide counts, consensus, and polymorphism table

## How to apply

After running lastal/last-split with MinION-optimized parameters obtained via last-train to produce calc.lastalign.maf, convert the MAF to SAM using LAST's maf-convert tool. Sort the resulting calc.sam with samtools and index it to obtain calc.bam and calc.bam.bai; these steps are part of NanoPipe's local installation prerequisites (install last; install samtools). The sorted BAM feeds nanopipe_calc_polymorphism.py, which computes per-position nucleotide counts and majority-rule consensus (minimum 10 reads per position, IUPAC codes when two nucleotides differ by ≤20%). Correct conversion is confirmed when calc.bam is indexable and the polymorphism pipeline produces calc.nuccounts.1.poly with expected SNPs listed as true polymorphisms.

## Related tools

- **NanoPipe** (Pipeline that chains maf-convert and samtools conversion steps and consumes the BAM for polymorphism calculation via nanopipe_calc_polymorphism.py) — https://github.com/IOB-Muenster/nanopipe2
- **LAST 946** (Produces the calc.lastalign.maf input; its maf-convert utility performs the MAF-to-SAM conversion)
- **samtools** (Sorts calc.sam and indexes the BAM (calc.bam, calc.bam.bai); listed as a required installation step in NanoPipe)
- **MinION** (Source instrument generating the nanopore query reads being converted into alignment formats)

## Examples

```
maf-convert sam calc.lastalign.maf > calc.sam; samtools sort calc.sam -o calc.bam; samtools index calc.bam
```

## Evaluation signals

- calc.sam is produced from calc.lastalign.maf without error by maf-convert
- samtools successfully sorts and indexes the output, yielding calc.bam and calc.bam.bai
- nanopipe_calc_polymorphism.py runs on the BAM and emits calc.nuccounts.1.poly
- Expected SNPs appear as true polymorphisms with joint probability 1 in the polymorphisms table (e.g., the four expected EGFR mutations for H1975)
- Alignment statistics show high mapping rates (NanoPipe mapped 99.9% of query reads in the Plasmodium study case)

## Limitations

- SAM/BAM conversion assumes reads were pre-filtered appropriately; too-short mapped reads cause 'noise', so a read length limit (e.g., 200 nucleotides) may need to be set upstream
- Current SNP detection statistics are not yet sophisticated; more sophisticated statistics for SNP detection and evaluation are planned future work
- SAM/BAM conversion assumes the LAST alignment was produced with MinION-specific parameters obtained via last-train; generic alignment parameters may reduce accuracy

## Evidence

- [methods] Convert the MAF alignment to SAM and then to sorted, indexed BAM using LAST's maf-convert and samtools: [evidence span withheld — anchor status: not_found]
- [methods] Align reads with LAST using last-train MinION parameters: [evidence span withheld — anchor status: not_found]
- [methods] Minimum nucleotide count per position at least 10: "The minimum nucleotide count per position should be at least 10 (i.e., at least 10 reads), otherwise a gap will be assigned"
- [abstract] NanoPipe provides alignment parameters adjusted for MinION sequencing technology specifics: "NanoPipe was developed in consideration of the specifics of the MinION sequencing technologies, providing accordingly adjusted alignment parameters"
