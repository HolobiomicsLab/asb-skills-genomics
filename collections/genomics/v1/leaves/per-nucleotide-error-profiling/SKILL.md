---
name: per-nucleotide-error-profiling
description: Use when nanopore reads (cDNA SQK-PCS111 or direct RNA SQK-RNA002) have
  been aligned to a plasmid or mRNA reference and you need per-position mismatch/insertion/deletion
  counts or per-read mean error rates against the reference.
when_to_use_negative:
- Reads have not yet been aligned to a reference — pysamstats pileup-based statistics
  require a SAM/BAM alignment against a reference FASTA
- The goal is short-read (Illumina) error profiling, which in this workflow is trimmed
  to Q20 with TrimGalore and analysed separately, not with this ONT error profile
- Secondary or non-primary alignments should be included — the workflow explicitly
  filters with 'SAMtools view -F 2048' before profiling
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3569
tools:
- name: pysamstats
  role: Computes the per-nucleotide error profile of mapped reads relative to the
    reference index sequence (matches, mismatches, deletions, insertions per position)
  repo: https://github.com/alimanfoo/pysamstats
- name: Minimap2 (Release 2.20-r1064)
  role: Aligns ONT nanopore reads to the plasmid reference with '-ax map-ont' to produce
    the SAM/BAM input for error profiling
- name: SAMtools v1.15
  role: Converts SAM to sorted and indexed BAM files and extracts primary aligned
    reads via 'SAMtools view -F 2048'
- name: Mana
  role: Runs in --mrna mode on the BAM and reference FASTA to report automated mRNA
    quality statistics (e.g. dRNA error decomposition); depends on Pysam 0.20.0
  repo: https://github.com/scchess/Mana
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:9d283c47bdeeb096039326c8d99eb9248a8acbe32fb42ac3911cfce6db7f1f6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alimanfoo/pysamstats
  tools:
  - Mana
  - Minimap2
  - Minimap2 (Release 2.20-r1064)
  - Pysam
  - SAMtools
  - SAMtools v1.15
  - pysamstats
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3569
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# per-nucleotide-error-profiling

## Summary

Determines the per-nucleotide error profile (mismatches, insertions, deletions) of mapped ONT nanopore reads against a reference sequence using pysamstats, decomposing the observed error rate into error-type classes. Used in the VAX-seq mRNA vaccine QC workflow to quantify sequencing error rates, including elevated deletion rates in the poly(A) tail.

## When to use

Use when nanopore reads (cDNA SQK-PCS111 or direct RNA SQK-RNA002) have been aligned to a plasmid or mRNA reference and you need per-position mismatch/insertion/deletion counts or per-read mean error rates against the reference.

## When NOT to use

- Reads have not yet been aligned to a reference — pysamstats pileup-based statistics require a SAM/BAM alignment against a reference FASTA
- The goal is short-read (Illumina) error profiling, which in this workflow is trimmed to Q20 with TrimGalore and analysed separately, not with this ONT error profile
- Secondary or non-primary alignments should be included — the workflow explicitly filters with 'SAMtools view -F 2048' before profiling

## Inputs

- Sorted and indexed BAM file of Minimap2 ('-ax map-ont') alignments of ONT nanopore reads (cDNA or direct RNA) to a plasmid/mRNA reference
- Reference FASTA sequence (e.g. plasmid reference or reference index sequence)
- Primary-aligned reads extracted with 'SAMtools view -F 2048'
- Optional: pre-built Mana docker image (scchess/mana) with its BAM and FASTA inputs

## Outputs

- Per-nucleotide error profile table of mapped reads relative to the reference (per-position match, mismatch, insertion, deletion counts from pysamstats)
- Per-read mean error rates
- Error-type decomposition totals (e.g. dRNA: 7.78% total = 2.61% mismatch + 5.18% deletion + 1.43% insertion, poly(A) tail excluded)
- Deletion rate estimate within the template-encoded 3' poly(A) tail (e.g. ~13% for cDNA)

## How to apply

After aligning nanopore reads to the plasmid reference with Minimap2 (Release 2.20-r1064) using '-ax map-ont' and generating sorted/indexed BAM files with SAMtools v1.15, restrict the analysis to primary aligned reads (e.g. 'SAMtools view -F 2048' to exclude secondary alignments). Run pysamstats (built on Pysam 0.20.0) to compute the per-nucleotide error profile of mapped reads relative to the reference index sequence, using pileup-based statistics such as 'variation' to obtain matches, mismatches, deletions and insertions per position. For direct RNA sequencing, exclude the poly(A) tail region and decompose errors into mismatch, deletion and insertion classes to total the mean error rate across the mRNA body; optionally run Mana in --mrna mode first for automated QC statistics. For cDNA error profiling, aggregate per-read mean error rates and quantify the deletion rate within the template-encoded 3' poly(A) tail, where deletion rates are elevated. Compare dRNA per-nucleotide error biases against matched plasmid and cDNA profiles to isolate dRNA-specific error modes.

## Related tools

- **pysamstats** (Computes the per-nucleotide error profile of mapped reads relative to the reference index sequence (matches, mismatches, deletions, insertions per position)) — https://github.com/alimanfoo/pysamstats
- **Minimap2 (Release 2.20-r1064)** (Aligns ONT nanopore reads to the plasmid reference with '-ax map-ont' to produce the SAM/BAM input for error profiling)
- **SAMtools v1.15** (Converts SAM to sorted and indexed BAM files and extracts primary aligned reads via 'SAMtools view -F 2048')
- **Mana** (Runs in --mrna mode on the BAM and reference FASTA to report automated mRNA quality statistics (e.g. dRNA error decomposition); depends on Pysam 0.20.0) — https://github.com/scchess/Mana

## Examples

```
pysamstats --type variation --chromosome eGFP_reference alignment.bam > error_profile.tsv
```

## Evaluation signals

- Reproduced cDNA (SQK-PCS111) mean per-read error rate is ~5.0% with a deletion rate of ~13% within the poly(A) tail
- Reproduced dRNA (SQK-RNA002) total error rate matches the reported 7.78%, decomposed as 2.61% mismatch, 5.18% deletion and 1.43% insertion (poly(A) tail excluded)
- Input BAM is sorted and indexed and contains only primary alignments (secondary alignments removed with 'SAMtools view -F 2048')
- Per-position pysamstats records cover the entire mRNA vaccine reference with homogenous coverage, and the poly(A) tail region is either profiled (cDNA) or explicitly excluded (dRNA body analysis)

## Limitations

- The poly(A) tail shows an elevated deletion rate (~13% in cDNA), so error profiles computed over the tail are dominated by deletion artefacts; dRNA analysis excludes the tail region
- Direct RNA sequencing is less reliable than cDNA sequencing; dRNA-specific per-nucleotide error biases (e.g. deletion of low-quality poly(A) tail nucleotides) must be distinguished by comparison against matched plasmid and cDNA profiles
- pysamstats pileup statistics have a default maximum read depth of 8000 unless overridden with --max-depth, which may matter for high-coverage mRNA samples

## Evidence

- [discussion] per-nucleotide error profile with pysamstats: "The per-nucleotide error profile of mapped reads was determined relative to the reference index sequence using pysamstats"
- [discussion] primary alignment extraction: "SAMtools (v1.15) utilising SAMtools view -F 2048 was used to extract read length distributions of the primary aligned reads"
- [results] dRNA error decomposition finding: [evidence span withheld — anchor status: not_found]
- [results] cDNA poly(A) deletion finding: [evidence span withheld — anchor status: not_found]
- [readme] pysamstats variation statistics from README: [evidence span withheld — anchor status: external_doc]
