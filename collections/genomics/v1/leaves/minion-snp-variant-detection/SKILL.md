---
name: minion-snp-variant-detection
description: Use when MinION nanopore amplicon or targeted reads (FASTA/FASTQ) must
  be aligned to a reference target and expected mutations/SNPs identified, e.g. EGFR
  exons 17-22 from cell line H1975 against hg38.
when_to_use_negative:
- Metagenomics analysis — NanoPipe does not yet support it; the authors state implementation
  of a metagenomics option is planned
- Analysis requiring sophisticated SNP statistics beyond the current pipeline — the
  authors note 'more sophisticated statistics for SNP detection and evaluation' are
  planned future work
- Reads too short for reliable mapping — set a read length limit (e.g. 200 nucleotides)
  or the result is 'noise' from too-short read alignments
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: NanoPipe
  role: Web-driven/local pipeline performing data upload, read-to-target alignment,
    polymorphism calling, and results display for MinION data
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST 946
  role: Sequence aligner used to map MinION reads to the target via lastal/last-split
    with MinION-adjusted parameters trained by last-train
- name: samtools
  role: Converts and sorts LAST SAM output into indexed BAM (calc.bam with calc.bam.bai)
    for downstream polymorphism analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:04aa75bb7ced2c63047c2f387ac84a2907cb2472b6101be9761894a922ab7bf5
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# minion-snp-variant-detection

## Summary

Detect SNP variants from Oxford Nanopore MinION amplicon reads using NanoPipe: align FASTA/FASTQ reads against a target database with LAST (version 946, MinION-trained parameters), then call polymorphisms with nanopipe_calc_polymorphism.py using coverage- and probability-based filters.

## When to use

Use when MinION nanopore amplicon or targeted reads (FASTA/FASTQ) must be aligned to a reference target and expected mutations/SNPs identified, e.g. EGFR exons 17-22 from cell line H1975 against hg38.

## When NOT to use

- Metagenomics analysis — NanoPipe does not yet support it; the authors state implementation of a metagenomics option is planned
- Analysis requiring sophisticated SNP statistics beyond the current pipeline — the authors note 'more sophisticated statistics for SNP detection and evaluation' are planned future work
- Reads too short for reliable mapping — set a read length limit (e.g. 200 nucleotides) or the result is 'noise' from too-short read alignments

## Inputs

- MinION reads in FASTA or FASTQ format (query, e.g. H1975 EGFR exons 17-22 amplicon reads)
- Target reference sequence(s) in FASTA (e.g. hg38/RefSeq GCF_000001405.38 EGFR)
- dbSNP for SNP annotation

## Outputs

- calc.lastalign.maf (MAF alignments)
- calc.sam / calc.bam / calc.bam.bai (converted, sorted, indexed alignments)
- Consensus sequence with per-position nucleotide counts (calc.nuccounts.1.poly)
- Polymorphisms table of candidate SNPs with joint probability and dbSNP annotation
- Alignment statistics and alignment visualization

## How to apply

Build a target database by placing the reference FASTA (e.g. hg38/RefSeq GCF_000001405.38 EGFR) in a targets subfolder and indexing with NanoPipe's bundled lastdb (NANOPIPE/tools/bin/lastdb target target.fasta). Align the query reads with lastal/last-split (LAST version 946) using MinION-optimized parameters obtained via last-train, then convert calc.lastalign.maf to SAM/BAM with maf-convert and samtools. Run nanopipe_calc_polymorphism.py to compute per-position nucleotide counts and a majority-rule consensus (minimum 10 reads per position; IUPAC codes when two nucleotides differ by ≤20%). Candidate SNPs are those where the alternate nucleotide has ≥20% of reads, read coverage ≥30% of the maximum contig coverage, transitions weighted 2× transversions, and p-error assessed over ±10 nt around the SNP; annotate candidates against dbSNP. Confirm expected mutations appear as true SNPs (joint probability 1) in the calc.nuccounts.1.poly polymorphisms table.

## Related tools

- **NanoPipe** (Web-driven/local pipeline performing data upload, read-to-target alignment, polymorphism calling, and results display for MinION data) — https://github.com/IOB-Muenster/nanopipe2
- **LAST 946** (Sequence aligner used to map MinION reads to the target via lastal/last-split with MinION-adjusted parameters trained by last-train)
- **samtools** (Converts and sorts LAST SAM output into indexed BAM (calc.bam with calc.bam.bai) for downstream polymorphism analysis)

## Examples

```
NANOPIPE/tools/bin/lastdb target target.fasta; nanopipe_calc_polymorphism.py calc.maf
```

## Evaluation signals

- All expected region-specific mutations are listed as true SNPs with joint probability 1 in the calc.nuccounts.1.poly polymorphisms table (e.g. the four expected EGFR mutations in H1975 exons 17-22)
- A high fraction of query reads map: NanoPipe mapped 99.9% of reads in the Plasmodium case study
- Consensus positions with fewer than 10 reads are assigned gaps — verify the consensus respects this minimum nucleotide count
- Every reported SNP candidate satisfies the filters: alternate coverage ≥20% of reads and read coverage ≥30% of the maximum contig coverage
- BAM is properly sorted and indexed (calc.bam with companion calc.bam.bai)

## Limitations

- Current SNP detection statistics are 'not yet sophisticated'; more sophisticated statistics for SNP detection and evaluation are planned future work
- Metagenomics analysis is not yet supported
- Consensus positions with fewer than 10 reads are assigned a gap rather than a nucleotide
- Too-short reads can cause mapping 'noise'; a read length limit (e.g. 200 nucleotides) should be set

## Evidence

- [abstract] NanoPipe provides alignment parameters adjusted for MinION sequencing technology specifics: "NanoPipe was developed in consideration of the specifics of the MinION sequencing technologies, providing accordingly adjusted alignment parameters"
- [methods] minimum nucleotide count per position at least 10: "The minimum nucleotide count per position should be at least 10 (i.e., at least 10 reads), otherwise a gap will be assigned"
- [methods] IUPAC ambiguity inclusion at 20% count difference: "If the counts for any two nucleotides differ from each other by not more than by 20%, both nucleotides are included"
- [methods] SNP candidate coverage filters: "suggests a polymorphism candidate if an alternate nucleotide has coverage of at least 20% of all reads"
- [methods] LAST alignment of MinION reads: "mapping of sequencing reads to the target using version 946 of the LAST sequence aligner"
- [results] EGFR case validation: [evidence span withheld — anchor status: not_found]
- [readme] NanoPipe purpose and outputs: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest, alignment statistics and information about polymorphisms"
