---
name: allele-based-read-scoring-and-haplotype-reweighting
description: Use when raw long reads (Nanopore R9/R10 or PacBio CLR) from a diploid
  genome must be error-corrected where the sequencing error rate exceeds haplotype
  divergence and heterozygote alleles would otherwise be collapsed into the consensus.
when_to_use_negative:
- Input is PacBio HiFi or Nanopore duplex data — the paper reports PECAT achieves
  mediocre performance on these, and accurate SNP calling from corrected reads alone
  is not possible (unlike Hifiasm on HiFi reads).
- The goal is consensus/polishing of an already assembled contig (use a reference-based
  polisher such as Medaka or Racon instead of haplotype-aware read correction).
- Regions of very low heterozygosity rate — the method may not distinguish small sequencing
  errors from heterozygote alleles even with highly accurate reads.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: PECAT
  role: 'Runs the haplotype-aware POA error-correction module: allele-based +1/-1
    supporting-read scoring, high-scorer selection, re-weighting and iterative consensus
    generation.'
  repo: https://github.com/lemene/PECAT
- name: minimap2
  role: Produces the raw read-to-read overlaps (e.g. -x ava-pb preset via corr_rd2rd_options)
    used as candidate supporting-read alignments for POA graph construction.
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:3706469207d17d9acd923acb169f882a6b075814cc86eb80661169c635c4a8e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lemene/PECAT
  tools:
  - PECAT
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# allele-based-read-scoring-and-haplotype-reweighting

## Summary

PECAT's haplotype-aware POA-based error correction: supporting reads are scored +1/-1 against the template read at dominant heterozygous branches, high-scoring (same-haplotype) reads are selected and re-weighted as consensus evidence, so heterozygote alleles survive while sequencing errors are corrected.

## When to use

Use when raw long reads (Nanopore R9/R10 or PacBio CLR) from a diploid genome must be error-corrected where the sequencing error rate exceeds haplotype divergence and heterozygote alleles would otherwise be collapsed into the consensus.

## When NOT to use

- Input is PacBio HiFi or Nanopore duplex data — the paper reports PECAT achieves mediocre performance on these, and accurate SNP calling from corrected reads alone is not possible (unlike Hifiasm on HiFi reads).
- The goal is consensus/polishing of an already assembled contig (use a reference-based polisher such as Medaka or Racon instead of haplotype-aware read correction).
- Regions of very low heterozygosity rate — the method may not distinguish small sequencing errors from heterozygote alleles even with highly accurate reads.

## Inputs

- Raw long noisy reads in FASTA/FASTQ (gzipped accepted), e.g. ./demo/reads.fasta.gz
- Filled PECAT config file (genome_size, prep_output_coverage, corr_rd2rd_options, corr_iterate_number)
- Read-to-read overlap alignments (PAF) from minimap2 between the extracted raw reads

## Outputs

- Corrected reads FASTA at the correction output location (e.g. S1/1-correct/corrected_reads.fasta)
- Per-supporting-read allele scores and the selected high-scoring (same-haplotype) supporting-read set with re-weighted consensus evidence
- Heterozygote-retaining, error-corrected consensus reads ready for the diploid assembly step

## How to apply

Extract the prep_output_coverage longest raw reads from the input FASTA/FASTQ (e.g. ./demo/reads.fasta.gz with genome_size set in the config), then compute candidate supporting-read overlaps with minimap2 using corr_rd2rd_options (e.g. '-x ava-pb' for PacBio CLR). Build the partial-order alignment (POA) graph and detect dominant parallel branches as heterozygous locations; score each supporting read +1 when it and the template read pass through the same dominant branch, -1 otherwise. Histogram the read scores, select the high-scoring supporting reads (very likely same haplotype as the template), and re-weight them as supporting evidence for the consensus. Generate the consensus path by dynamic programming, penalizing each mismatch of a supporting read by P_c = max(0.4^r, 0.3)*W_c at round r, and iterate corr_iterate_number rounds; validation of the step is the corrected-reads FASTA at S1/1-correct/corrected_reads.fasta produced via the 'pecat.pl correct' entry point.

## Related tools

- **PECAT** (Runs the haplotype-aware POA error-correction module: allele-based +1/-1 supporting-read scoring, high-scorer selection, re-weighting and iterative consensus generation.) — https://github.com/lemene/PECAT
- **minimap2** (Produces the raw read-to-read overlaps (e.g. -x ava-pb preset via corr_rd2rd_options) used as candidate supporting-read alignments for POA graph construction.) — https://github.com/lh3/minimap2

## Examples

```
pecat.pl correct config.txt  # config sets genome_size=1500000, prep_output_coverage, corr_rd2rd_options, corr_iterate_number
```

## Evaluation signals

- The corrected-reads FASTA exists at the 1-correct output location (S1/1-correct/corrected_reads.fasta) and the run completed through the 'pecat.pl correct' entry point.
- Known heterozygous variants are retained as alternative alleles across the corrected read set instead of being merged into the consensus.
- Per-read base error rate of corrected reads is substantially lower than the raw Nanopore R9/R10 or PacBio CLR input while haplotype divergence is preserved.
- The mismatch penalty schedule in consensus generation matches P_c = max(0.4^r, 0.3)*W_c for round r and correction runs corr_iterate_number rounds.
- Read score histograms show bimodal separation, with only high-scoring (same-haplotype) supporting reads contributing weighted evidence to each consensus.

## Limitations

- In very low heterozygosity rate regions the error correction may not effectively distinguish small errors from heterozygote alleles, and SNP calling accuracy there remains low.
- PECAT needs higher data coverage to obtain a high-quality phased assembly.
- Assembly/correction parameters used in the study are only detailed in Supplementary Notes 4 and 5, not the main text.

## Evidence

- [abstract] We design a haplotype-aware error correction method that can retain heterozygote alleles while correcting sequencing errors.: "We design a haplotype-aware error correction method that can retain heterozygote alleles while correcting sequencing errors."
- [discussion] The PECAT error correction method is based on the partial-order alignment (POA) graph method4,47.: "The PECAT error correction method is based on the partial-order alignment (POA) graph method4,47."
- [discussion] The method scores each supporting read against the template read in the POA graph (+1 when both pass through the same dominant branch at heterozygous positions, -1 otherwise): [evidence span withheld — anchor status: not_found]
- [intro] One challenge for correct-then-assemble pipelines in assembling a diploid genome using long noisy reads is how to retain heterozygotes during error correction.: "One challenge for correct-then-assemble pipelines in assembling a diploid genome using long noisy reads is how to retain heterozygotes during error correction."
- [readme] PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method.: "PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method."
