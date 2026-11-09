---
name: structural-variant-calling-from-bam
description: Use when you have long-read nanopore BAM alignments against GRCh37 and
  need a filtered SV VCF as input for downstream gene-fusion annotation or breakpoint
  refinement.
when_to_use_negative:
- Input reads are short paired-end or already a curated VCF of final fusion calls
  rather than long-read BAM alignments
- DNA is fragmented (e.g. FFPE material), where short read lengths prevent comprehensive
  SV/fusion identification
- The goal is small-variant (SNV/indel) calling rather than gross structural variants
  and gene fusions
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: NanoSV
  role: Primary SV caller run on the minimap2-sorted BAM with default parameters min_mapq=12,
    depth_support=False, mapq_flag=48; run a second pass with cluster_count=2 on LAST-remapped
    reads to refine breakpoints
  repo: https://github.com/mroosmalen/nanosv
- name: Sniffles
  role: Alternative SV caller to NanoSV, run with defaults -s 2 -n -1 --genotype
- name: minimap2
  role: Maps ONT nanopore reads to GRCh37 to produce the input BAM (v.2.6)
  repo: https://github.com/lh3/minimap2
- name: Samtools
  role: Indexes and sorts the minimap2 alignment BAM prior to SV calling (v.1.7)
- name: LAST
  role: Remaps extracted fusion-spanning reads for increased alignment accuracy before
    the breakpoint-refinement NanoSV pass (v.921)
- name: NanoFG
  role: Pipeline wrapper that invokes the SV-calling step; accepts a BAM and VCF via
    -b/-v flags
  repo: https://github.com/SdeBlank/NanoFG
merged_aliases:
- structural-variant-calling
merged_alias_records:
- alias: structural-variant-calling-from-bam-alignments
  slug: structural-variant-calling-from-bam-alignments
  jaccard_score: 0.8
  method: token-set-jaccard
  decision: needs_review
- alias: structural-variant-calling-from-long-reads
  slug: structural-variant-calling-from-long-reads
  cosine_score: 0.8404
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_003
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:eed1946e68d1b92f165cd4b4073b6983fa883448810d1b99612f03dc07bafc10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - Delly
  - FUDGE
  - LAST
  - Lumpy
  - Manta
  - Minimap2
  - NanoFG
  - NanoSV
  - NanoSim
  - Samtools
  - Sniffles
  - Wtdbg2
  - bedtools
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# structural-variant-calling-from-bam

## Summary

Call structural variants from long-read nanopore BAM alignments to the GRCh37 human reference using NanoSV (v.1.2.4) or Sniffles (v.1.0.9), then filter the resulting VCF as the first stage of the NanoFG fusion-gene pipeline.

## When to use

Use when you have long-read nanopore BAM alignments against GRCh37 and need a filtered SV VCF as input for downstream gene-fusion annotation or breakpoint refinement.

## When NOT to use

- Input reads are short paired-end or already a curated VCF of final fusion calls rather than long-read BAM alignments
- DNA is fragmented (e.g. FFPE material), where short read lengths prevent comprehensive SV/fusion identification
- The goal is small-variant (SNV/indel) calling rather than gross structural variants and gene fusions

## Inputs

- Sorted, indexed BAM file of nanopore long reads aligned to GRCh37 (produced with minimap2 v.2.6)
- Nanopore FASTQ read directory (alternative entry point via NanoFG.sh -f)
- Pre-computed SV VCF (optional, supplied via the -v flag)

## Outputs

- Filtered VCF of structural variants (built-in caller filters applied, all insertions removed)
- Candidate fusion-gene annotations (SV ends in different ENSEMBL genes, same-strand, ≥2 fusion-spanning reads)

## How to apply

Map nanopore FASTQ reads to the GRCh37 human reference genome with minimap2 (v.2.6) and index/sort the BAM with Samtools (v.1.7). Call SVs from the sorted BAM with either NanoSV (v.1.2.4, defaults: min_mapq=12, depth_support=False, mapq_flag=48) or Sniffles (v.1.0.9, defaults -s 2 -n -1 --genotype). Remove every SV failing the caller's built-in filters and remove all insertions from the VCF, since only deletions/inversions/translocations carry fusion-relevant gene-pair breakpoints. The resulting filtered VCF is annotated with ENSEMBL genes; SVs with both ends in different genes are flagged as possible fusions. For nucleotide-resolution breakpoint refinement, supporting reads can later be re-extracted and remapped with LAST followed by a second NanoSV pass with cluster_count=2.

## Related tools

- **NanoSV** (Primary SV caller run on the minimap2-sorted BAM with default parameters min_mapq=12, depth_support=False, mapq_flag=48; run a second pass with cluster_count=2 on LAST-remapped reads to refine breakpoints) — https://github.com/mroosmalen/nanosv
- **Sniffles** (Alternative SV caller to NanoSV, run with defaults -s 2 -n -1 --genotype)
- **minimap2** (Maps ONT nanopore reads to GRCh37 to produce the input BAM (v.2.6)) — https://github.com/lh3/minimap2
- **Samtools** (Indexes and sorts the minimap2 alignment BAM prior to SV calling (v.1.7))
- **LAST** (Remaps extracted fusion-spanning reads for increased alignment accuracy before the breakpoint-refinement NanoSV pass (v.921))
- **NanoFG** (Pipeline wrapper that invokes the SV-calling step; accepts a BAM and VCF via -b/-v flags) — https://github.com/SdeBlank/NanoFG

## Examples

```
bash NanoFG.sh -b <path/to/bam> -v <path/to/vcf> -n SAMPLE_NAME
```

## Evaluation signals

- Output VCF contains no insertion calls — all insertions must have been removed after calling
- Every retained SV passes the built-in NanoSV or Sniffles filters
- Reported fusions are each supported by at least two fusion-spanning reads (the paper's reliability cut-off)
- Fusion breakpoints from the second NanoSV pass (cluster_count=2 on LAST-remapped reads) match the expected test-set breakpoints
- A priori: BAM is sorted/indexed Samtools v.1.7 output from minimap2 v.2.6 against GRCh37

## Limitations

- Two fusions detected with only one supporting read fall below the two-read cut-off and required manual breakpoint PCR validation
- Intratumoral heterogeneity and tumor purity are likely to influence the lower detection limits of the assay
- SVs must have both ends within annotated ENSEMBL genes on the same strand to be retained as continuous-transcript fusion candidates

## Evidence

- [discussion] By default, all SVs that do not pass the built-in NanoSV or Sniffles filters are removed. In addition, all insertions are also removed from the VCF.: "By default, all SVs that do not pass the built-in NanoSV or Sniffles filters are removed. In addition, all insertions are also removed from the VCF."
- [discussion] NanoSV v1.2.4 default parameters: "The user can choose either NanoSV (v. 1.2.4)10 with default parameters: ‘min_mapq=12, depth_support=False, mapq_flag=48’"
- [discussion] Sniffles default parameters: "or Sniffles (v.1.0.9)28 with default parameters: ‘-s 2 -n -1 --genotype’ to detect SVs."
- [discussion] Reads mapped to GRCh37 with minimap2: "Reads were mapped to the human reference genome version GRCHh37 by using minimap2 (v. 2.6)"
- [readme] NanoFG accepts BAM/VCF inputs: "bash NanoFG.sh -b </path/to/bam> [-v </path/to/vcf>] [-n SAMPLE_NAME ]"
