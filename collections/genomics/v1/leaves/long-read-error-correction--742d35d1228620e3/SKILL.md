---
name: long-read-error-correction
description: Use when correcting raw Nanopore R9/R10 or PacBio CLR diploid long reads
  prior to assembly and heterozygous alleles (rather than only consensus accuracy)
  must be preserved.
when_to_use_negative:
- Input is Nanopore duplex or PacBio HiFi reads, where the article reports PECAT achieves
  only mediocre correction/assembly performance
- The goal is maximum single-haplotype consensus accuracy regardless of heterozygote
  retention (e.g. a haploid reference polish), where standard POA consensus tools
  suffice
- Very low heterozygosity-rate regions are the focus, since the method cannot reliably
  distinguish small sequencing errors from heterozygote alleles there
edam_operation: http://edamontology.org/operation_3185
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: PECAT
  role: 'Main pipeline: implements the haplotype-aware POA-based error correction
    (''pecat.pl correct cfgfile'')'
  repo: https://github.com/lemene/PECAT
- name: minimap2
  role: Computes read-to-read overlaps (e.g. '-x ava-pb' for PacBio CLR) that feed
    POA graph construction
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:8f768ccaccef7d59c96faf43a80749085098cd9d051d133dd50e0d1204147f1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lemene/PECAT
  tools:
  - FALCON-Unzip
  - NECAT
  - PECAT
  - minimap2
  edam_operation: http://edamontology.org/operation_3185
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# long-read-error-correction

## Summary

Haplotype-aware error correction of long noisy reads using a POA-graph method that scores supporting reads for same-haplotype membership so heterozygote alleles are retained while sequencing errors (typical of Nanopore R9/R10 and PacBio CLR data) are corrected.

## When to use

Use when correcting raw Nanopore R9/R10 or PacBio CLR diploid long reads prior to assembly and heterozygous alleles (rather than only consensus accuracy) must be preserved.

## When NOT to use

- Input is Nanopore duplex or PacBio HiFi reads, where the article reports PECAT achieves only mediocre correction/assembly performance
- The goal is maximum single-haplotype consensus accuracy regardless of heterozygote retention (e.g. a haploid reference polish), where standard POA consensus tools suffice
- Very low heterozygosity-rate regions are the focus, since the method cannot reliably distinguish small sequencing errors from heterozygote alleles there

## Inputs

- Raw long reads in FASTA/FASTQ (e.g. ./demo/reads.fasta.gz) from Nanopore R9, Nanopore R10, or PacBio CLR sequencing of a diploid genome
- PECAT config file with genome_size, prep_output_coverage, corr_rd2rd_options, and corr_iterate_number set

## Outputs

- Corrected reads FASTA at the project's 1-correct output location (S1/1-correct/corrected_reads.fasta)
- Corrected reads retaining heterozygote alleles, suitable for downstream SNP calling and diploid string-graph assembly

## How to apply

Extract the longest raw reads at the configured coverage (prep_output_coverage, e.g. the longest 80X) and align reads to each other with minimap2 using corr_rd2rd_options (e.g. '-x ava-pb' for PacBio CLR) to get candidate supporting-read overlaps. Build a partial-order alignment (POA) graph per template read and detect heterozygous positions as dominant parallel branches; score each supporting read +1 when it passes through the same branch as the template and -1 otherwise, so high-scoring reads are likely same-haplotype. Histogram the scores, select the high-scoring reads, and re-weight them (W_c) as supporting evidence in the dynamic-programming consensus, penalizing each mismatch by P_c = max(0.4^r, 0.3)*W_c; iterate corr_iterate_number rounds. The pipeline entry point is 'pecat.pl correct cfgfile' with genome_size set in the config, producing corrected reads at 1-correct/corrected_reads.fasta.

## Related tools

- **PECAT** (Main pipeline: implements the haplotype-aware POA-based error correction ('pecat.pl correct cfgfile')) — https://github.com/lemene/PECAT
- **minimap2** (Computes read-to-read overlaps (e.g. '-x ava-pb' for PacBio CLR) that feed POA graph construction) — https://github.com/lh3/minimap2

## Examples

```
pecat.pl correct cfgfile  # config sets genome_size, prep_output_coverage, corr_iterate_number; outputs 1-correct/corrected_reads.fasta
```

## Evaluation signals

- Corrected-reads FASTA exists at the expected output path (S1/1-correct/corrected_reads.fasta) after running 'pecat.pl correct'
- Haplotype-specific k-mer consistency of corrected reads reaches >= 99.4% on trio-classified diploid datasets (computed as sum of max(kp,km) over sum of (kp+km) on parent-specific k-mer pairs)
- Haplotype-specific k-mer completeness: percentage of parent-specific k-mers (occurring >= 4 times in the 40X longest reads) retained in the 40X longest corrected reads
- Corrector comparison: PECAT consistency exceeds Canu, FALCON, MECAT2, and NECAT, all of which are <= 94.8%
- Corrected reads support subsequent heterozygous SNP calling against the haplotype-collapsed first-round assembly

## Limitations

- Error correction may not distinguish small sequencing errors from heterozygote alleles in very low heterozygosity-rate regions
- Corrected reads alone are insufficient for accurate SNP calling, unlike Hifiasm with HiFi reads; PECAT must combine corrected- and raw-read SNP callers
- PECAT needs higher data coverage to obtain a high-quality phased assembly
- Mediocre performance on Nanopore duplex and PacBio HiFi reads

## Evidence

- [abstract] haplotype-aware error correction that retains heterozygote alleles while correcting sequencing errors: "We design a haplotype-aware error correction method that can retain heterozygote alleles while correcting sequencing errors."
- [discussion] POA graph scoring of supporting reads: "The PECAT error correction method is based on the partial-order alignment (POA) graph method4,47."
- [discussion] minimap2 overlap parameters: "we find candidate overlaps between raw reads using minimap2 with parameters “-x ava-pb” for PacBio CLR reads"
- [readme] PECAT is a phased error correction and assembly tool: "PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method."
- [discussion] low-heterozygosity limitation: "our current error correction method may not be effective enough to distinguish small errors from heterozygote alleles in very low heterozygosity rate regions"
