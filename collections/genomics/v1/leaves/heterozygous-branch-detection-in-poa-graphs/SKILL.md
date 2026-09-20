---
name: heterozygous-branch-detection-in-poa-graphs
description: Use when correcting long noisy Nanopore R9/R10 or PacBio CLR reads for
  diploid assembly and sequencing errors must be corrected without collapsing heterozygote
  alleles, especially when error rate exceeds haplotype divergence.
when_to_use_negative:
- Input reads are already high-accuracy (Nanopore duplex or PacBio HiFi) where PECAT
  achieves only mediocre performance
- Regions with very low heterozygosity rate, where the method cannot reliably distinguish
  small sequencing errors from genuine heterozygote alleles
- Goal is consensus/variant calling on an existing assembly rather than read-level
  error correction for de novo diploid assembly
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3693
tools:
- name: PECAT
  role: Implements the haplotype-aware POA error correction, including same-haplotype
    supporting-read scoring and re-weighting; run via 'pecat.pl correct'
  repo: https://github.com/lemene/PECAT
- name: minimap2
  role: Produces candidate supporting-read overlaps for POA graph construction (e.g.,
    '-x ava-pb' for PacBio CLR reads)
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:70e24c5ede35c1f51caf4d6533108459847c7901b453edeaab36e951173c0edb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lemene/PECAT
  tools:
  - PECAT
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3693
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# heterozygous-branch-detection-in-poa-graphs

## Summary

Detects heterozygous positions in a partial-order alignment (POA) graph as dominant parallel branches, scores each supporting read +1/-1 per branch, and selects high-scoring same-haplotype reads to correct the template read while retaining heterozygote alleles. This is PECAT's haplotype-aware error correction core for long noisy reads.

## When to use

Use when correcting long noisy Nanopore R9/R10 or PacBio CLR reads for diploid assembly and sequencing errors must be corrected without collapsing heterozygote alleles, especially when error rate exceeds haplotype divergence.

## When NOT to use

- Input reads are already high-accuracy (Nanopore duplex or PacBio HiFi) where PECAT achieves only mediocre performance
- Regions with very low heterozygosity rate, where the method cannot reliably distinguish small sequencing errors from genuine heterozygote alleles
- Goal is consensus/variant calling on an existing assembly rather than read-level error correction for de novo diploid assembly

## Inputs

- Raw long noisy reads in FASTA/FASTQ format (e.g., ./demo/reads.fasta.gz; Nanopore R9, Nanopore R10, or PacBio CLR)
- Filled PECAT config file (e.g., genome_size=1500000, prep_output_coverage, corr_iterate_number)
- Supporting-read overlaps from minimap2 read-to-read alignment (corr_rd2rd_options)

## Outputs

- Corrected reads FASTA file at the correction output location (S1/1-correct/corrected_reads.fasta)
- Supporting-read score assignments to allelic branches at heterozygous positions

## How to apply

Extract the longest raw reads (e.g., the prep_output_coverage longest reads, 80X by default, or all reads if under 80X) and align them to each other with minimap2 (e.g., '-x ava-ont'/'-x ava-pb' depending on chemistry) to obtain candidate supporting-read overlaps. Build the POA graph for each template read and detect heterozygous locations as dominant parallel branches; score each supporting read +1 when it passes through the same dominant branch at a heterozygous position and -1 otherwise. Histogram the read scores, select only the high-scoring supporting reads (very likely same haplotype as the template read), and re-weight them as supporting evidence for the consensus. Generate the consensus path by dynamic programming, penalizing each mismatch of a supporting read with P_c = max(0.4^r, 0.3)*W_c, and iterate corr_iterate_number rounds of correction; write the corrected reads as a FASTA file (S1/1-correct/corrected_reads.fasta via 'pecat.pl correct').

## Related tools

- **PECAT** (Implements the haplotype-aware POA error correction, including same-haplotype supporting-read scoring and re-weighting; run via 'pecat.pl correct') — https://github.com/lemene/PECAT
- **minimap2** (Produces candidate supporting-read overlaps for POA graph construction (e.g., '-x ava-pb' for PacBio CLR reads)) — https://github.com/lh3/minimap2

## Examples

```
perl pecat.pl correct config.ini
```

## Evaluation signals

- Corrected-reads FASTA file exists at the project's 1-correct output location (S1/1-correct/corrected_reads.fasta)
- Pipeline completes via the 'pecat.pl correct' entry point without failure
- Heterozygous SNPs are retained in the corrected reads (checkable downstream by aligning corrected reads to the haplotype-collapsed genome for SNP calling)
- Consensus mismatch penalty applied per supporting read equals P_c = max(0.4^r, 0.3)*W_c, iterated corr_iterate_number times
- Downstream assembly yields more contiguous haplotype-specific contigs (e.g., phase block NG50 gains such as 59.4/58.0 Mb on HG002 Nanopore R10)

## Limitations

- Error correction cannot distinguish small errors from heterozygote alleles in very low heterozygosity rate regions, even with highly accurate reads
- Requires higher data coverage to obtain high-quality phased assembly
- Corrected reads alone are insufficient for accurate SNP calling (PECAT additionally calls SNPs on raw Nanopore reads with Clair3)

## Evidence

- [discussion] POA branch scoring for haplotype assignment: [evidence span withheld — anchor status: not_found]
- [abstract] Haplotype-aware correction rationale: "We design a haplotype-aware error correction method that can retain heterozygote alleles while correcting sequencing errors."
- [discussion] POA-based consensus generation: "The PECAT error correction method is based on the partial-order alignment (POA) graph method"
- [discussion] minimap2 read-overlap detection: "we find candidate overlaps between raw reads using minimap2 with parameters “-x ava-pb” for PacBio CLR reads"
- [readme] PECAT dependencies including minimap2: "PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method."
- [intro] Challenge motivating the skill: "One challenge for correct-then-assemble pipelines in assembling a diploid genome using long noisy reads is how to retain heterozygotes during error correction."
