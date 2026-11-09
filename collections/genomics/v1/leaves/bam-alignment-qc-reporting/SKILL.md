---
name: bam-alignment-qc-reporting
description: Use when you have sorted, indexed alignment BAM files (e.g. ONT cDNA
  or Illumina TruSeq alignments against a plasmid/mRNA reference FASTA) and need automated
  QC reports covering coverage homogeneity, per-nucleotide error rates, and consensus
  accuracy.
when_to_use_negative:
- Input is raw FASTQ reads rather than sorted, indexed BAM alignments — first preprocess
  (Porechop barcode removal, Minimap2 alignment, SAMtools indexing)
- You need direct detection of nucleoside modifications (e.g. N1-methylpseudouridine)
  — that requires direct RNA sequencing analysis, not BAM QC reporting
- A single summary read-length or run-quality statistic suffices and NanoPlot alone
  answers the question — Mana's ~10 minute analysis is aimed at full mRNA QC reporting
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3170
tools:
- name: Mana
  role: Primary command-line toolkit that analyses ONT/Illumina alignment BAMs and
    automates mRNA QC reports, internally invoking SAMtools, BCFtools, Pysam, and
    bedtools
  repo: https://github.com/scchess/Mana
- name: SAMtools
  role: Generates sorted and indexed BAM files and supports read-level filtering (e.g.
    samtools view -F 2048 for primary alignments)
- name: BCFtools
  role: Performs variant calling and consensus sequence generation within the Mana
    QC workflow
- name: Pysam
  role: Provides pysam/pysamstats-based per-nucleotide error profiles of mapped reads
    relative to the reference sequence
  repo: https://github.com/alimanfoo/pysamstats
- name: Minimap2 (Release 2.20-r1064)
  role: Upstream aligner producing the input BAMs (with -ax map-ont for Nanopore reads)
- name: NanoPlot v1.38.1
  role: Complementary run and sample read-quality statistics alongside the alignment
    QC reports
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:14eb142c41ff26dcd7a61c4aafb40e713594f57e24b864b3b23868cbee7ac712
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scchess/Mana
  tools:
  - BCFtools
  - Illumina
  - Mana
  - Minimap2 (Release 2.20-r1064)
  - NanoPlot v1.38.1
  - Pysam
  - SAMtools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# bam-alignment-qc-reporting

## Summary

Automated BAM alignment quality-control reporting for mRNA sequencing using the Mana toolkit, which internally orchestrates SAMtools, BCFtools, Pysam, pysamstats, and bedtools to produce per-nucleotide coverage, per-nucleotide error profiles, and consensus statistics from alignment BAMs.

## When to use

Use when you have sorted, indexed alignment BAM files (e.g. ONT cDNA or Illumina TruSeq alignments against a plasmid/mRNA reference FASTA) and need automated QC reports covering coverage homogeneity, per-nucleotide error rates, and consensus accuracy.

## When NOT to use

- Input is raw FASTQ reads rather than sorted, indexed BAM alignments — first preprocess (Porechop barcode removal, Minimap2 alignment, SAMtools indexing)
- You need direct detection of nucleoside modifications (e.g. N1-methylpseudouridine) — that requires direct RNA sequencing analysis, not BAM QC reporting
- A single summary read-length or run-quality statistic suffices and NanoPlot alone answers the question — Mana's ~10 minute analysis is aimed at full mRNA QC reporting

## Inputs

- Sorted, indexed alignment BAM file (e.g. ONT cDNA alignment produced via Minimap2 -ax map-ont and SAMtools v1.15)
- Short-read alignment BAM (e.g. Illumina TruSeq alignment)
- Reference FASTA (plasmid or mRNA reference sequence)

## Outputs

- Per-nucleotide coverage report over the mRNA
- Per-nucleotide error profile of mapped reads relative to the reference
- Consensus sequence statistics from BCFtools variant calling
- QC report of mRNA quality/coverage homogeneity

## How to apply

Mount the working directory containing the alignment BAM(s) and reference FASTA into the Mana docker image with -v (e.g. -v ${PWD}:/src) and run 'python3 mana.py --mrna' with -b pointing to the BAM, -f to the reference FASTA, and -o to an output directory; Mana internally invokes SAMtools, BCFtools, Pysam, and bedtools. Run Mana once per alignment (e.g. once for the Illumina TruSeq BAM, once for the ONT cDNA BAM) to generate comparable outputs. From the outputs, extract per-nucleotide coverage and per-nucleotide error profiles (pysamstats relative to the reference sequence) and consensus statistics (BCFtools variant calling and consensus generation). For low-complexity regions such as the template-encoded 3' poly(A) tail, quantify short-read misalignment and the drop in consensus accuracy relative to the reference-encoded poly(A) sequence, and compare coverage homogeneity: ONT cDNA reads give full homogeneous coverage of the entire mRNA length while Illumina coverage is heterogeneous and disrupted at the poly(A) tail.

## Related tools

- **Mana** (Primary command-line toolkit that analyses ONT/Illumina alignment BAMs and automates mRNA QC reports, internally invoking SAMtools, BCFtools, Pysam, and bedtools) — https://github.com/scchess/Mana
- **SAMtools** (Generates sorted and indexed BAM files and supports read-level filtering (e.g. samtools view -F 2048 for primary alignments))
- **BCFtools** (Performs variant calling and consensus sequence generation within the Mana QC workflow)
- **Pysam** (Provides pysam/pysamstats-based per-nucleotide error profiles of mapped reads relative to the reference sequence) — https://github.com/alimanfoo/pysamstats
- **Minimap2 (Release 2.20-r1064)** (Upstream aligner producing the input BAMs (with -ax map-ont for Nanopore reads))
- **NanoPlot v1.38.1** (Complementary run and sample read-quality statistics alongside the alignment QC reports)

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out
```

## Evaluation signals

- Mana completes the '--mrna' run without error for each BAM and writes the -o output directory (~10 minutes analysis time expected per run)
- ONT cDNA alignment shows full and homogeneous per-nucleotide coverage across the entire mRNA vaccine length
- Per-nucleotide error profile confirms sequence identity between the called consensus and the reference; Illumina coverage is heterogeneous with misalignment concentrated at the 3' poly(A) tail
- Coverage profiles of sequencing replicates are reproducible, with squared correlation of heterogeneous Illumina coverage reaching approximately R2 = 0.99
- Input BAMs are sorted and indexed (SAMtools v1.15) before running Mana; primary-aligned reads only (excluding secondary alignments, samtools view -F 2048) are used for read-length distributions

## Limitations

- Short-read Illumina sequencing misaligns at the low-complexity poly(A) tail, causing many errors and poor consensus accuracy — do not trust Illumina-based consensus over that region
- Mana is designed for analysis of ONT alignments (.BAM); the docker build takes >= 20 minutes if built locally, and the pre-built image is recommended
- Paper data are proof of concept with n = 1 to n = 4, no predetermined sample size, and experiments were not randomised or blinded

## Evidence

- [readme] Mana analyses ONT alignments (.BAM) to report quality control statistics: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."
- [full_text] Mana internally invokes SAMtools, BCFtools, Pysam, and bedtools: "The per-nucleotide error profile of mapped reads was determined relative to the reference index sequence using pysamstats"
- [full_text] Variant calling and consensus generation with BCFtools: "Variant calling and consensus sequence generation was completed using bcftools (v1.15)"
- [full_text] ONT cDNA gives full homogeneous coverage across the entire mRNA length: "Nanopore cDNA sequencing was sufficient to confirm sequence identity with full and homogenous coverage of the entire mRNA vaccine length"
- [readme] Docker mounting of files is required to run Mana: "To run an analysis with the file paths, one would do: docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out"
