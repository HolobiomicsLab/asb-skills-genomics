---
name: polymorphism-artifact-filtering
description: Use when separating true SNPs from sequencing/mapping artifacts in MinION
  read alignments — e.g. per-position nucleotide counts from NanoPipe must be filtered
  with the ≥10-read minimum, ≥20% alternate coverage, and p-error quality checks before
  calling polymorphisms.
when_to_use_negative:
- The goal is metagenomics analysis — the article states NanoPipe does not yet support
  metagenomics and only plans this option
- Positions failing the thresholds (alternate coverage <20% of reads, position coverage
  <10 reads, or <30% of maximum contig coverage) — these are artifacts/gaps by definition,
  not reportable SNPs
- High-confidence variant calling from mature, orchestrated variant-caller stacks
  is needed — the authors note current NanoPipe SNP detection statistics 'are not
  yet sophisticated' and plan more sophisticated statistics as future work
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0102
tools:
- name: NanoPipe
  role: runs nanopipe_calc_polymorphism.py to compute per-position nucleotide counts,
    majority-rule consensus, IUPAC coding, and filtered candidate SNPs from the BAM
    alignment
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST 946
  role: produces the MAF alignment (calc.lastalign.maf) whose mapped reads feed the
    polymorphism filtering; read length limits avoid mapping of too-short reads
- name: samtools
  role: converts the MAF alignment to sorted, indexed BAM (calc.bam, calc.bam.bai)
    required as input for polymorphism calculation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:b882b9a9eae4ebc6517d8e291e7b032e7d9038a23b11794590748802045c3439
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST 946
  - NanoPipe
  - samtools
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# polymorphism-artifact-filtering

## Summary

Applies NanoPipe's majority-rule consensus and SNP-candidate filtering criteria to per-position nucleotide counts from aligned Oxford Nanopore MinION reads, suppressing sequencing/mapping noise (e.g. short-read mismapping) before calling polymorphisms with dbSNP annotation.

## When to use

Use when separating true SNPs from sequencing/mapping artifacts in MinION read alignments — e.g. per-position nucleotide counts from NanoPipe must be filtered with the ≥10-read minimum, ≥20% alternate coverage, and p-error quality checks before calling polymorphisms.

## When NOT to use

- The goal is metagenomics analysis — the article states NanoPipe does not yet support metagenomics and only plans this option
- Positions failing the thresholds (alternate coverage <20% of reads, position coverage <10 reads, or <30% of maximum contig coverage) — these are artifacts/gaps by definition, not reportable SNPs
- High-confidence variant calling from mature, orchestrated variant-caller stacks is needed — the authors note current NanoPipe SNP detection statistics 'are not yet sophisticated' and plan more sophisticated statistics as future work

## Inputs

- sorted, indexed BAM alignment of MinION reads (calc.bam / calc.bam.bai) from LAST lastal/last-split with MinION-optimized last-train parameters
- MAF alignment file (calc.lastalign.maf)
- reference/target database built from a target FASTA (e.g. hg38/RefSeq GCF_000001405.38 EGFR exons) with lastdb
- query MinION reads in FASTA or FASTQ format

## Outputs

- per-position nucleotide count and polymorphism file (calc.nuccounts.1.poly)
- majority-rule consensus sequence with IUPAC ambiguity codes where top-two nucleotide counts differ by ≤20%
- candidate SNP table with transition/transversion-weighted probability, p-error quality score, and dbSNP annotation (true SNPs at joint probability 1)

## How to apply

Run nanopipe_calc_polymorphism.py on the sorted BAM (calc.bam, produced from calc.lastalign.maf via maf-convert and samtools) to obtain per-position nucleotide counts and a majority-rule consensus; assign a gap wherever a position has fewer than 10 reads. Include both nucleotides as IUPAC ambiguity codes only when the top two nucleotide counts differ by ≤20%. Promote a position to a SNP candidate only if the alternate nucleotide covers ≥20% of all reads and the position's read coverage is ≥30% of the maximum coverage in the respective contig, then score candidates with transitions weighted 2 versus transversions 1 and a p-error assessment of alignment quality over ±10 nt around the SNP, annotating against dbSNP. To suppress 'noise' from too-short reads, set a read length limit (e.g. 200 nucleotides) before alignment. Validate by checking that expected true SNPs appear in the polymorphisms table with joint probability 1 (as with the four expected EGFR mutations in the H1975 exon 17-22 case).

## Related tools

- **NanoPipe** (runs nanopipe_calc_polymorphism.py to compute per-position nucleotide counts, majority-rule consensus, IUPAC coding, and filtered candidate SNPs from the BAM alignment) — https://github.com/IOB-Muenster/nanopipe2
- **LAST 946** (produces the MAF alignment (calc.lastalign.maf) whose mapped reads feed the polymorphism filtering; read length limits avoid mapping of too-short reads)
- **samtools** (converts the MAF alignment to sorted, indexed BAM (calc.bam, calc.bam.bai) required as input for polymorphism calculation)

## Examples

```
python NANOPIPE/nanopipe_calc_polymorphism.py  # emits calc.nuccounts.1.poly; inspect .poly for true SNPs at joint probability 1
```

## Evaluation signals

- In the H1975 EGFR exon 17-22 benchmark, the four expected EGFR transcript-region mutations are listed as true SNPs with joint probability 1 in the polymorphisms table
- Every position with fewer than 10 reads is output as a gap rather than a called nucleotide
- Positions where the top two nucleotide counts differ by ≤20% carry an IUPAC ambiguity code in the consensus instead of the majority nucleotide alone
- Every reported SNP candidate satisfies alternate-nucleotide coverage ≥20% of all reads AND position coverage ≥30% of the maximum contig coverage
- Candidate scoring shows transitions weighted by factor 2 and transversions by factor 1, with p-error estimated over ±10 nt around each SNP

## Limitations

- Current SNP detection statistics are not yet sophisticated; more sophisticated statistics for SNP detection and evaluation are planned as future work
- Without setting a read length limit (e.g. 200 nucleotides), too-short reads map as 'noise' and can introduce spurious low-level counts
- Positions with <10 reads are assigned gaps, so low-coverage regions yield no consensus call rather than a tentative SNP

## Evidence

- [methods] minimum nucleotide count threshold: "The minimum nucleotide count per position should be at least 10 (i.e., at least 10 reads), otherwise a gap will be assigned"
- [methods] alternate coverage candidate rule: "suggests a polymorphism candidate if an alternate nucleotide has coverage of at least 20% of all reads"
- [methods] contig coverage rule: "a SNP candidate should have read coverage of at least 30% from the maximum coverage in the respective contig"
- [methods] transition/transversion weighting: "transversions are weighted by the factor of 1 and transitions by the factor of 2"
- [results] short-read noise control: "To avoid “noise,” i.e., mapping of too short reads, the user can set the read length limit, e.g., to 200 nucleotides"
- [discussion] future statistics caveat: "The next step for the NanoPipe project will include more sophisticated statistics for SNP detection and evaluation."
