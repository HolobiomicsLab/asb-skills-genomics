---
name: reported-fraction-verification
description: Use when validating reported percentages of undetected known variants
  located in homopolymer runs or STRs, where undetected sets are defined by combinations
  of Illumina, PacBio, and ONT caller failures against a truth VCF.
when_to_use_negative:
- Input is raw caller VCF output that has not yet been intersected with the GIAB/1KGp3
  truth set to define the undetected-variant classes
- The reference genome build does not match the truth-set build (e.g., GRCh37 truth
  variants queried against GRCh38), since flank extraction would be misaligned
- The goal is de novo variant discovery rather than characterizing why known truth
  variants were missed
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0102
tools:
- name: samtools faidx
  role: Extracts downstream reference flanking sequence at each variant position from
    the indexed GRCh37/GRCh38 FASTA
- name: pysam
  role: Python FASTA reader used programmatically to fetch flanking sequence per variant
    coordinate
- name: Clairvoyante
  role: Multi-task CNN variant caller whose misses on Illumina, PacBio, and ONT data
    define the undetected-variant sets being characterized
  repo: https://github.com/aquaskyline/Clairvoyante
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:e2c34cc790aa3c65d6a58723a193c99af635373481622fb428a2b70cf59748d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - Illumina
  - Oxford Nanopore
  - PacBio
  - pysam
  - samtools faidx
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# reported-fraction-verification

## Summary

Recomputes the fractions of known (GIAB/1KGp3 truth) variants undetected by Illumina, PacBio, and/or Oxford Nanopore variant callers that lie in homopolymer runs or short tandem repeats, and checks the recomputed percentages against the reported values (26.67% homopolymer / 29.45% STR of 12,331 variants missed by all three technologies; 57.67% / 18.54% of 178,331 missed by PacBio+ONT).

## When to use

Use when validating reported percentages of undetected known variants located in homopolymer runs or STRs, where undetected sets are defined by combinations of Illumina, PacBio, and ONT caller failures against a truth VCF.

## When NOT to use

- Input is raw caller VCF output that has not yet been intersected with the GIAB/1KGp3 truth set to define the undetected-variant classes
- The reference genome build does not match the truth-set build (e.g., GRCh37 truth variants queried against GRCh38), since flank extraction would be misaligned
- The goal is de novo variant discovery rather than characterizing why known truth variants were missed

## Inputs

- VCF/set of 12,331 known variants undetected by Illumina, PacBio, and ONT callers simultaneously (from GIAB 3.3.2 HG001 / 1KGp3 truth)
- VCF/set of 178,331 known variants undetected by both PacBio and ONT callers
- Reference genome FASTA (GRCh37, or GRCh38 for build-matched truth sets) with faidx index
- pysam (Python) or samtools faidx for sequence extraction

## Outputs

- Per-variant annotation table with homopolymer/STR context labels (homopolymer_str_classification.csv)
- Fraction summary: homopolymer and STR percentages per undetected-variant set (expected ~26.67%/29.45% for 12,331; ~57.67%/18.54% for 178,331)

## How to apply

Load the undetected-variant sets — the 12,331 variants missed simultaneously by Illumina, PacBio, and Oxford Nanopore callers, and the 178,331 variants missed by both PacBio and ONT — derived from GIAB/1KGp3 truth variants. For each variant, extract the downstream reference flank from GRCh37 (or GRCh38 when the truth set is build-matched) using pysam/samtools faidx. Scan each downstream flank for homopolymer runs (>=4 identical consecutive bases) and short tandem repeats (>=3 repeat units of a 1-6 bp motif) using regex/tandem-repeat scanning logic. Tabulate per-variant context labels, compute the two fractions per set, and compare against the reported values; write per-variant annotations plus the fraction summary to homopolymer_str_classification.csv. The rationale is that elevated SMS background errors in these repetitive contexts explain why callers miss truth variants there.

## Related tools

- **samtools faidx** (Extracts downstream reference flanking sequence at each variant position from the indexed GRCh37/GRCh38 FASTA)
- **pysam** (Python FASTA reader used programmatically to fetch flanking sequence per variant coordinate)
- **Clairvoyante** (Multi-task CNN variant caller whose misses on Illumina, PacBio, and ONT data define the undetected-variant sets being characterized) — https://github.com/aquaskyline/Clairvoyante

## Examples

```
python -c "import pysam,re; fa=pysam.FastaFile('GRCh37.fa'); s=fa.fetch(c,p,p+50); print(bool(re.search(r'(.)\1{3,}',s)), bool(re.search(r'(.{1,6})\1{2,}',s)))"
```

## Evaluation signals

- Recomputed homopolymer fraction of the 12,331-variant set matches the reported 26.67% (approximately 3,287 variants labeled homopolymer-positive)
- Recomputed STR fraction of the 12,331-variant set matches the reported 29.45%
- The 178,331-variant PacBio+ONT-missed set reproduces ~57.67% homopolymer and ~18.54% STR
- homopolymer_str_classification.csv contains exactly one annotated row per input variant with both context labels populated
- Fraction totals are stable under a re-run with a different FASTA reader (pysam vs. samtools faidx) on the same build-matched reference

## Limitations

- Results depend on the chosen motif definitions — homopolymer as >=4 identical consecutive bases and STR as >=3 repeat units of a 1-6 bp motif — so alternative definitions will shift the fractions
- Undetected-variant sets are contingent on the specific callers, truth set (GIAB 3.3.2 HG001), and sequencing runs used; Clairvoyante itself misses multi-allelic variants and provides exact indel alleles only for indels <=4 bp, which shapes the missed sets
- Truth set and reference build must be matched (GRCh37 vs GRCh38) or flank extraction yields wrong contexts

## Evidence

- [results] Among the 12,331 known variants undetected by all three sequencing technologies, 26.67% are located in homopolymer runs and 29.45% are located in short tandem repeats: [evidence span withheld — anchor status: not_found]
- [methods] Extract the downstream reference sequence flanking each variant position from GRCh37 (or GRCh38 where the truth set is build-matched) using a genome FASTA reader in pysam/samtools faidx.: [evidence span withheld — anchor status: not_found]
- [methods] Scan the extracted downstream flanks for homopolymer runs (>=4 identical consecutive bases) and short tandem repeats (>=3 repeat units of a 1-6 bp motif) using regex/tandem-repeat scanning logic.: [evidence span withheld — anchor status: not_found]
- [abstract] using Illumina, PacBio, and Oxford Nanopore data, respectively: "using Illumina, PacBio, and Oxford Nanopore data, respectively"
- [discussion] we used as our truth dataset the latest dataset version 3.3.2 for HG001: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
