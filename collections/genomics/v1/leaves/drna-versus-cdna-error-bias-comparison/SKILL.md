---
name: drna-versus-cdna-error-bias-comparison
description: Use when comparing error modes of SQK-RNA002 direct RNA sequencing against
  matched cDNA or plasmid sequencing of the same mRNA vaccine construct, to determine
  which per-nucleotide biases are dRNA-specific.
when_to_use_negative:
- When only cDNA or Illumina data are available and no matched direct RNA sequencing
  exists to isolate dRNA-specific biases
- When the analysis target includes the poly(A) tail itself — the article restricts
  the mean error-rate calculation to the mRNA body, excluding the poly(A) tail
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Minimap2 (Release 2.20-r1064)
  role: Aligns ONT reads to the plasmid/reference using the '-ax map-ont' preset prior
    to error profiling
- name: SAMtools v1.15
  role: Converts SAM to sorted and indexed BAM; used to extract primary aligned reads
    (view -F 2048)
- name: Mana
  role: Analyses ONT alignments (.BAM) in --mrna mode to report automated mRNA quality
    statistics
  repo: https://github.com/scchess/Mana
- name: pysamstats
  role: 'Computes the per-nucleotide error profile (variation statistics: matches,
    mismatches, deletions, insertions) against the reference'
  repo: https://github.com/alimanfoo/pysamstats
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:d3b7f3b0e2918cc0f3daed4cb09528c45ea09519c53010eae89036c79c22b7f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scchess/Mana
  tools:
  - Mana
  - Minimap2
  - Minimap2 (Release 2.20-r1064)
  - Pysam
  - SAMtools
  - SAMtools v1.15
  - pysamstats
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# drna-versus-cdna-error-bias-comparison

## Summary

Quantifies per-nucleotide sequencing error profiles of ONT direct RNA (dRNA) reads and decomposes them into mismatch, deletion and insertion classes, then compares against matched plasmid and cDNA sequencing to isolate dRNA-specific error biases (e.g., deletion of low-quality poly(A) tail nucleotides).

## When to use

Use when comparing error modes of SQK-RNA002 direct RNA sequencing against matched cDNA or plasmid sequencing of the same mRNA vaccine construct, to determine which per-nucleotide biases are dRNA-specific.

## When NOT to use

- When only cDNA or Illumina data are available and no matched direct RNA sequencing exists to isolate dRNA-specific biases
- When the analysis target includes the poly(A) tail itself — the article restricts the mean error-rate calculation to the mRNA body, excluding the poly(A) tail

## Inputs

- ONI/ONT Nanopore direct RNA (SQK-RNA002) reads and matched cDNA reads in FASTQ format
- Plasmid or mRNA reference sequence (FASTA)
- Sorted and indexed BAM alignment produced by Minimap2 -ax map-ont and SAMtools v1.15
- Reference index sequence used for per-nucleotide comparison

## Outputs

- Mean sequencing error rate for the mRNA body (excluding poly(A) tail), e.g., 7.78%
- Error-type decomposition: mismatch %, deletion %, insertion % (e.g., 2.61%, 5.18%, 1.43%)
- Per-nucleotide error profile from pysamstats variation statistics
- dRNA-specific error bias calls versus matched plasmid and cDNA profiles

## How to apply

Align ONT reads to the reference with Minimap2 (Release 2.20-r1064) using the '-ax map-ont' preset, then convert and sort the SAM files into indexed BAMs with SAMtools v1.15. Run Mana (scchess/mana docker image) in --mrna mode with -b, -f and -o arguments to produce alignment-based QC statistics. Compute the per-nucleotide error profile of mapped reads relative to the reference index sequence using pysamstats (variation statistics), restricted to primary aligned reads (e.g., excluding secondary alignments via SAMtools view -F 2048 logic) and excluding the poly(A) tail region. Decompose errors into mismatch, deletion and insertion classes and total the mean error rate across the mRNA body. Finally, compare dRNA per-nucleotide biases against matched plasmid and cDNA error profiles to distinguish dRNA-specific error modes from shared sequencing noise.

## Related tools

- **Minimap2 (Release 2.20-r1064)** (Aligns ONT reads to the plasmid/reference using the '-ax map-ont' preset prior to error profiling)
- **SAMtools v1.15** (Converts SAM to sorted and indexed BAM; used to extract primary aligned reads (view -F 2048))
- **Mana** (Analyses ONT alignments (.BAM) in --mrna mode to report automated mRNA quality statistics) — https://github.com/scchess/Mana
- **pysamstats** (Computes the per-nucleotide error profile (variation statistics: matches, mismatches, deletions, insertions) against the reference) — https://github.com/alimanfoo/pysamstats

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out && pysamstats --type variation --fasta /src/file.fasta /src/alignment.bam
```

## Evaluation signals

- Reproduced total mean error rate over the mRNA body (poly(A) excluded) matches the reported 7.78%
- Error decomposition matches the reported 2.61% mismatch, 5.18% deletion, 1.43% insertion
- Per-nucleotide profile restricted to primary aligned reads (secondary alignments excluded) is reproducible via SAMtools view -F 2048 filtering
- dRNA error biases show deletion enrichment at low-quality poly(A) tail nucleotides relative to matched plasmid and cDNA profiles
- Mana --mrna run on the sorted/indexed BAM completes (~10 min) and produces expected QC output files in the -o directory

## Limitations

- Deletion of low-quality poly(A) tail nucleotides is a dRNA-specific bias, so error rates computed over the tail are not comparable to cDNA
- The authors found cDNA sequencing more reliable than direct RNA sequencing, so dRNA error profiles should be treated as chemistry-informative but noisier
- Mana depends on specific versions (SAMtools 1.5.1/1.15, BCFtools 1.16, Pysam 0.20.0, bedtools 2.30.0); manual installation is slow and the README recommends the pre-built docker image

## Evidence

- [discussion] Per-nucleotide error profile determination with pysamstats: "The per-nucleotide error profile of mapped reads was determined relative to the reference index sequence using pysamstats"
- [discussion] Alignment preset for ONT reads: "aligned to the plasmid reference via Minimap2 (Release 2.20-r1064) with -ax map-ont for Nanopore"
- [discussion] BAM generation with SAMtools: "processed via SAMtools v1.15 to generate sorted and indexed BAM files"
- [readme] Mana usage from README: "docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out"
- [readme] pysamstats variation statistics type: [evidence span withheld — anchor status: external_doc]
- [discussion] cDNA reliability comparison: "While we found cDNA sequencing more reliable than direct RNA sequencing, we nevertheless anticipate direct RNA sequencing will be a useful research tool"
