---
name: pacbio-metagenome-read-correction
description: Use when raw PacBio CLR (or ONT) FASTQ/FASTA reads from mixed samples
  such as metagenomes or polyploid genomes must be error-corrected without masking
  true low-frequency haplotype variants.
when_to_use_negative:
- When consensus-sequence-based correction is sufficient, e.g. clonal haploid samples
  where masking of low-frequency variants is not a concern
- When runtime and peak memory are the binding constraints — VeChat experiences longer
  runtimes and higher peak memory usage than competing tools, particularly on large
  datasets
- When reads are short or already high-accuracy (e.g. PacBio HiFi/CCS or Illumina),
  since VeChat targets noisy long TGS reads
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: VeChat
  role: Primary two-cycle variation-graph-based error correction tool applied to the
    raw PacBio reads
  repo: https://github.com/HaploKit/vechat
- name: Minimap2
  role: Computes minimizer-based all-vs-all read overlaps that seed the variation-graph
    construction in cycle 1
  repo: https://github.com/lh3/minimap2
- name: racon
  role: Modified racon is built automatically as a dependency used within the VeChat
    correction pipeline
  repo: https://github.com/lbcb-sci/racon
- name: fpa
  role: Dependency (pinned at version 0.5) used to filter pairwise alignments of read
    overlaps
  repo: https://github.com/natir/fpa
- name: yacrd
  role: Dependency used with all-against-all read mappings for chimeric-read handling
    in the pipeline
  repo: https://github.com/natir/yacrd
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:319510e3138148628a07af37a5de2ee18669cde5ec3c44911bb43bdb96b0718e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - CONSENT
  - Canu
  - Daccord
  - Minimap2
  - PacBio
  - Racon
  - VeChat
  - fpa
  - racon
  - yacrd
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# pacbio-metagenome-read-correction

## Summary

Haplotype-aware self-correction of noisy PacBio CLR metagenomic reads using VeChat's two-cycle variation-graph workflow, which preserves low-frequency haplotype variants that consensus-sequence templates mistakenly mask as errors.

## When to use

Use when raw PacBio CLR (or ONT) FASTQ/FASTA reads from mixed samples such as metagenomes or polyploid genomes must be error-corrected without masking true low-frequency haplotype variants.

## When NOT to use

- When consensus-sequence-based correction is sufficient, e.g. clonal haploid samples where masking of low-frequency variants is not a concern
- When runtime and peak memory are the binding constraints — VeChat experiences longer runtimes and higher peak memory usage than competing tools, particularly on large datasets
- When reads are short or already high-accuracy (e.g. PacBio HiFi/CCS or Illumina), since VeChat targets noisy long TGS reads

## Inputs

- Raw PacBio CLR sequencing reads in FASTQ or FASTA format (gzip compression supported), e.g. NWC metagenome, Microbial 10-plex metagenome, or yeast pseudo-diploid read sets from ENA PRJEB7245
- ONT reads (via --platform ont) as an alternative sequencing platform

## Outputs

- Corrected full-length reads (reads.corrected.fa) preserving haplotype-specific true variants
- Residual sequencing error rates (read accuracy/QV) quantified by aligning corrected reads back to references, e.g. pacbio_error_rates.tsv

## How to apply

Install VeChat from bioconda with its dependencies (minimap2, yacrd, fpa=0.5, and a modified racon built automatically), then run vechat reads.fq.gz -t 8 --platform pb -o reads.corrected.fa keeping default graph-pruning parameters: --min-confidence 0.2, --min-support 0.2, and --min-identity-cns 0.99. The method runs two correction cycles: cycle 1 computes minimizer-based all-vs-all overlaps with Minimap2, builds read alignment piles, divides them into windows, and constructs variation graphs via partial order alignment (POA), iteratively pruning spurious nodes/edges using a frequent itemset model until convergence; cycle 2 recomputes overlaps with base-level alignments for haplotype-aware overlap filtration and derives haplotype-aware consensus sequences via dynamic programming. For large metagenome FASTQ files (>20G) or FASTA files (>10G), apply the --split option, optionally using scripts/vechat_hpc.fast.sh for HPC parallelization. Validate by aligning corrected reads back and computing residual error rate (QV), expecting reproduction of reported values such as 0.101% (NWC metagenome) and 0.089% (Microbial 10-plex).

## Related tools

- **VeChat** (Primary two-cycle variation-graph-based error correction tool applied to the raw PacBio reads) — https://github.com/HaploKit/vechat
- **Minimap2** (Computes minimizer-based all-vs-all read overlaps that seed the variation-graph construction in cycle 1) — https://github.com/lh3/minimap2
- **racon** (Modified racon is built automatically as a dependency used within the VeChat correction pipeline) — https://github.com/lbcb-sci/racon
- **fpa** (Dependency (pinned at version 0.5) used to filter pairwise alignments of read overlaps) — https://github.com/natir/fpa
- **yacrd** (Dependency used with all-against-all read mappings for chimeric-read handling in the pipeline) — https://github.com/natir/yacrd

## Examples

```
vechat reads.fq.gz -t 8 --platform pb -o reads.corrected.fa
```

## Evaluation signals

- reads.corrected.fa exists for each dataset after the run
- Computed residual error rates reproduce reported values: 0.089% (Microbial 10-plex), 0.101% (NWC metagenome), 0.236% (yeast pseudo-diploid)
- VeChat error rates are the lowest of all tools tested on the real PacBio datasets (approximately 2–4x, 1.4–7.8x, and 3.3–5.6x lower than competitors)
- Default parameters were used: --min-confidence 0.2, --min-support 0.2, --min-identity-cns 0.99, with --platform pb
- A quick smoke test with example/reads.fq.gz completes successfully before running large datasets

## Limitations

- VeChat does not win in terms of computational resources: on large datasets it experiences longer runtimes and higher peak memory usage than competing correction tools
- VeChat uses off-the-shelf approaches without exploiting all their features, causing unnecessary overheads; edit-distance and sequence-to-graph alignments could be replaced by more efficient routines
- Consensus-induced bias handling matters only for mixed samples; the exploration of optimal preprocessing/error-correction/assembly pipelines for particular polyploid, cancer, or metagenomic settings remains future work

## Evidence

- [results] VeChat attains the lowest error rates of all tools tested: [evidence span withheld — anchor status: not_found]
- [abstract] VeChat is based on variation graphs: "We present VeChat, as an approach to implement this idea: VeChat is based on variation graphs, as a popular type of data structure for pangenome reference systems."
- [results] graph pruning via frequent itemset model: "we make use of a frequent itemset model that involves read coverage, sequencing errors and co-occurrence of characters in reads"
- [other] VeChat command-line usage with platform setting: [evidence span withheld — anchor status: not_found]
- [discussion] longer runtimes and higher memory on large datasets: "In particular on large datasets, VeChat experiences longer runtimes and higher peak memory usage"
