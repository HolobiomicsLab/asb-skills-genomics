---
name: bam-read-pileup-filtering
description: Use when preparing long-read BAM alignments for Longshot candidate SNV
  discovery and you must enforce Longshot's read-level (min_mapq) and position-level
  (min_cov, max_cov) filters before allelotyping or genotyping.
when_to_use_negative:
- Calling structural variants (Longshot only calls SNVs; SVs > 50 bp are unsupported
  — use Delly or an assembly-based method instead)
- Short-read variant calling where GATK HaplotypeCaller or FreeBayes are the appropriate
  tools and Longshot's long-read error model does not apply
- The input is not an aligned, sorted and indexed BAM/CRAM with a matching indexed
  FASTA reference
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Longshot
  role: Implements the pileup-based candidate SNV identification with its min_mapq,
    min_cov, max_cov and auto_max_cov filters applied to reads from the input BAM/CRAM
  repo: https://github.com/pjedge/longshot
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:e7c9386c146dc9f7276904c7eb57a21460598fc31a3d2a6c3093381ade2ae1e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - Longshot
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# BAM read pileup filtering for candidate SNV discovery

## Summary

Filters reads and positions from a sorted, indexed BAM/CRAM of long error-prone reads (PacBio SMRT / Oxford Nanopore) so that only high-confidence read observations feed Longshot's pileup-based candidate SNV identification and downstream allelotyping.

## When to use

Use when preparing long-read BAM alignments for Longshot candidate SNV discovery and you must enforce Longshot's read-level (min_mapq) and position-level (min_cov, max_cov) filters before allelotyping or genotyping.

## When NOT to use

- Calling structural variants (Longshot only calls SNVs; SVs > 50 bp are unsupported — use Delly or an assembly-based method instead)
- Short-read variant calling where GATK HaplotypeCaller or FreeBayes are the appropriate tools and Longshot's long-read error model does not apply
- The input is not an aligned, sorted and indexed BAM/CRAM with a matching indexed FASTA reference

## Inputs

- Sorted, indexed BAM or CRAM file of long error-prone reads (e.g., PacBio SMRT, Oxford Nanopore)
- Indexed FASTA reference genome that the BAM is aligned to

## Outputs

- Set of candidate SNV positions passing min_cov/max_cov and depth filters
- Filtered set of reads (MAPQ >= 20) available for per-position allelotyping and pileup-based genotype likelihood calculation

## How to apply

Load the sorted, indexed BAM/CRAM (aligned to an indexed FASTA reference) and discard any read whose mapping quality falls below min_mapq (default 20); only reads passing this filter count toward coverage. For each genomic position, consider it a potential SNV only if filtered-read coverage lies between min_cov (default 6) and max_cov (default 8000, or auto_max_cov = mean_coverage + 5*sqrt(mean_coverage)); the maximum-coverage cutoff exists to remove false SNVs caused by copy number amplification. Additionally, variants with read depth > d + 5*sqrt(d), where d is the median read depth across the entire dataset, are filtered out before haplotype-informed genotyping. Positions that survive these filters proceed to candidate SNV identification via genotype likelihood calculation from the pileup, and their reads pass forward to pair-HMM allelotyping, where further per-read filters apply (max CIGAR indel length 20, minimum allele quality 7.0).

## Related tools

- **Longshot** (Implements the pileup-based candidate SNV identification with its min_mapq, min_cov, max_cov and auto_max_cov filters applied to reads from the input BAM/CRAM) — https://github.com/pjedge/longshot

## Examples

```
longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf -q 20 -c 6 -C 8000
```

## Evaluation signals

- No read with mapping quality below 20 (min_mapq default) contributes coverage at any candidate position
- No candidate SNV position has filtered-read coverage below 6 (min_cov) or above 8000 (max_cov) unless auto_max_cov (mean + 5*sqrt(mean)) is enabled
- Positions with depth > d + 5*sqrt(d), where d is the median dataset read depth, are absent from downstream genotyping input
- Retained allelotype observations have phred-scaled allele quality >= 7.0 after filtering
- Candidate pileup SNV calls are reproducible from the same BAM using Longshot's documented defaults

## Limitations

- Allele observations with phred-scaled quality below 7.0 are discarded, which reduces the effective read depth for SMS reads
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge, so read-level filtering alone cannot rescue misalignments
- The max read depth filter (d + 5*sqrt(d)) is required to remove copy number amplification false SNVs; omitting it inflates false positives in amplified regions

## Evidence

- [methods] min_mapq (minimum mapping quality filter): "Minimum mapping quality to use a read. [default: 20]"
- [methods] min_cov and max_cov coverage filters: "Minimum coverage (of reads passing filters) to consider position as a potential SNV. [default: 6]"
- [discussion] max read depth filter for copy number amplification: [evidence span withheld — anchor status: not_found]
- [discussion] allele quality threshold reduces effective depth: [evidence span withheld — anchor status: not_found]
- [discussion] candidate SNV identification from pileup: "Potential SNVs are identified from the pileup of aligned reads by performing a genotype likelihood calculation similar to Samtools"
