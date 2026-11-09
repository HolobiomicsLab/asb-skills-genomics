---
name: sv-breakpoint-read-haplotype-assignment
description: Use when long nanopore reads overlapping trio-phased heterozygous SNPs
  must be assigned to a parental haplotype — e.g. to determine the parental origin
  of de novo SV breakpoint junctions or to phase NanoSV/Manta SV calls against a trio-phased
  SNP ground truth.
when_to_use_negative:
- No trio-phased heterozygous SNP anchors (e.g. no trio pedigree or GATK PBT haplotypes)
  are available to define the paternal and maternal haplotypes
- Reads are short Illumina paired-end reads that typically do not span multiple heterozygous
  SNPs, so read-level haplotype likelihoods are uninformative
- The goal is SV discovery/breakpoint detection itself — use NanoSV SV calling instead;
  this skill consumes existing SV calls and phased SNPs
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: LAST
  role: Maps MinION reads to the reference; its alignments supply the split/gapped
    read segments and qualities used for read-based haplotype assignment
  repo: http://last.cbrc.jp/
- name: NanoSV
  role: Produces the heterozygous SV calls (breakpoint junctions and supporting reads)
    whose flanking read haplotypes are compared to phase each SV
  repo: https://github.com/mroosmalen/nanosv
- name: sambamba
  role: Converts, sorts and indexes the LAST-derived SAM/BAM read alignments that
    serve as the phasing input
  repo: https://github.com/biod/sambamba
- name: MinION nanopore sequencer
  role: Source of the long nanopore reads (11–16x coverage) that span multiple trio-phased
    heterozygous SNPs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:fef9f43e27d294f838922bb245b0241452e114273c39e4de4ff8de12c18a419d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LAST
  - MinION nanopore sequencer
  - NanoSV
  - sambamba
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# sv-breakpoint-read-haplotype-assignment

## Summary

Assign individual MinION nanopore reads to a parental (paternal/maternal) haplotype by summing SNP-allele log-likelihoods over trio-phased heterozygous SNPs, then use these haplotype tags to determine the parental origin of de novo chromothripsis breakpoint junctions and to phase heterozygous SV calls directly from long reads.

## When to use

Use when long nanopore reads overlapping trio-phased heterozygous SNPs must be assigned to a parental haplotype — e.g. to determine the parental origin of de novo SV breakpoint junctions or to phase NanoSV/Manta SV calls against a trio-phased SNP ground truth.

## When NOT to use

- No trio-phased heterozygous SNP anchors (e.g. no trio pedigree or GATK PBT haplotypes) are available to define the paternal and maternal haplotypes
- Reads are short Illumina paired-end reads that typically do not span multiple heterozygous SNPs, so read-level haplotype likelihoods are uninformative
- The goal is SV discovery/breakpoint detection itself — use NanoSV SV calling instead; this skill consumes existing SV calls and phased SNPs

## Inputs

- LAST-aligned MinION nanopore read alignment (coordinate-sorted, indexed BAM) including base and mapping qualities
- GATK PBT trio-phased heterozygous SNP list (haplotype anchors / ground truth)
- Heterozygous SV calls from Manta/NanoSV with breakpoint-junction positions and supporting read IDs

## Outputs

- Per-read paternal/maternal haplotype assignments
- Parental-origin assignment for each de novo chromothripsis breakpoint junction
- Per-SV phasing assignments for heterozygous SVs
- sv_phasing_benchmark.tsv with per-SV phasing assignments and concordance metrics

## How to apply

Load the LAST-aligned Patient MinION read BAM (LAST was chosen because these alignments give the most accurate SV calling results with NanoSV) together with the GATK pedigree-based trio (PBT) heterozygous SNP list used as haplotype anchors. For each read overlapping a trio-phased heterozygous SNP, sum the SNP allele log-likelihoods — incorporating base and mapping qualities — and assign the read to the paternal or maternal haplotype by the read-likelihood method. Partition breakpoint-junction supporting reads and reference reads by assigned parental haplotype to determine the parental origin of each de novo junction (all 40 Patient 1 junctions were paternal). For each of the 2389 Manta/NanoSV-overlapping heterozygous SVs, phase the SV by comparing the haplotypes of breakpoint-flanking haplotype-assigned reads. Finally, evaluate phased SV calls against the GATK PBT ground truth, computing per-SV concordance; the rationale is that MinION reads up to hundreds of kilobases span multiple phased SNPs, enabling read-level haplotype separation that short reads cannot achieve.

## Related tools

- **LAST** (Maps MinION reads to the reference; its alignments supply the split/gapped read segments and qualities used for read-based haplotype assignment) — http://last.cbrc.jp/
- **NanoSV** (Produces the heterozygous SV calls (breakpoint junctions and supporting reads) whose flanking read haplotypes are compared to phase each SV) — https://github.com/mroosmalen/nanosv
- **sambamba** (Converts, sorts and indexes the LAST-derived SAM/BAM read alignments that serve as the phasing input) — https://github.com/biod/sambamba
- **MinION nanopore sequencer** (Source of the long nanopore reads (11–16x coverage) that span multiple trio-phased heterozygous SNPs)

## Examples

```
hap = 'paternal' if sum(llh[b]['paternal'] for b in phased_snps_on_read) > sum(llh[b]['maternal'] for b in phased_snps_on_read) else 'maternal'
```

## Evaluation signals

- Phased SV count matches the benchmark: 1909 of 2389 Manta/NanoSV-overlapping heterozygous SVs phased
- Phasing concordance against the GATK PBT trio-phased heterozygous SNP ground truth reaches 98.5%
- All 40 de novo chromothripsis breakpoint junctions in Patient 1 are assigned paternal origin, reproducing Fig. 2
- Per-read haplotype likelihood sums are finite and nonzero, and each read receives exactly one paternal/maternal label
- Input BAM is coordinate-sorted and indexed before read extraction (NanoSV/sambamba requirement)

## Limitations

- Medium coverage (11–16x) and intrinsic nanopore sequencing biases limit how many SVs can be phased and confidence of likelihood assignments
- LAST mapping requires substantial compute resources (lastdb indexing, last-train on >10,000 reads) compared with other long-read mappers
- Simultaneous SNV calling with phasing was not done and is conditional on future MinION/ONT data quality and throughput increases
- SV discovery sensitivity is reduced (~75%) by extremely stringent post-calling filtering, affecting which heterozygous SVs are available to phase

## Evidence

- [discussion] we here devised a computational strategy that allowed accurate phasing of SVs directly from the long nanopore reads using flanking (phased) heterozygous SNPs: [evidence span withheld — anchor status: not_found]
- [discussion] we were able to separate reads by haplotype, which formed the basis for a reconstruction of the long-range structure of chromothripsis rearrangements: "we were able to separate reads by haplotype, which formed the basis for a reconstruction of the long-range structure of chromothripsis rearrangements"
- [intro] The long reads from the MinION allow efficient phasing of genetic variations (SNVs as well as SVs) and enable us to resolve the long-range structure of the chromothripsis: "The long reads from the MinION allow efficient phasing of genetic variations (SNVs as well as SVs) and enable us to resolve the long-range structure of the chromothripsis"
- [readme] We found that LAST alignments give the most accurate results for SV calling with NanoSV.: "We found that LAST alignments give the most accurate results for SV calling with NanoSV. However, mapping with LAST requires more compute resources."
