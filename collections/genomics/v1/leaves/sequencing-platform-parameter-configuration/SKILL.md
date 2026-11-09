---
name: sequencing-platform-parameter-configuration
description: Use when running VeChat (or its Minimap2 overlap step) on raw long reads
  and the input platform (PacBio PB or ONT) must be declared so platform-appropriate
  alignment presets and error models are applied.
when_to_use_negative:
- Input reads are already corrected or are short Illumina reads — VeChat is a self-correction
  method for noisy long TGS reads (PacBio/ONT)
- The analysis goal is polishing an assembly rather than correcting raw long reads
  — Racon is the consensus/polishing module in this toolchain
- A fixed reference genome exists for correction — VeChat performs haplotype-aware
  self-correction against ad-hoc variation graphs built from the reads themselves
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: VeChat
  role: Error-correction pipeline whose --platform flag (pb or ont, default pb) selects
    the sequencing platform configuration
  repo: https://github.com/HaploKit/vechat
- name: minimap2
  role: Computes minimizer-based all-vs-all read overlaps in Cycle 1 and base-level
    overlaps in Cycle 2, using platform presets (e.g. ava-pb / ava-ont)
  repo: https://github.com/lh3/minimap2
- name: Racon
  role: Prior-art consensus module (consensus template approach) that VeChat replaces
    with variation-graph-based correction
  repo: https://github.com/lbcb-sci/racon
- name: fpa
  role: Filter of pairwise (all-against-all read mapping) overlaps used in the VeChat
    pipeline
  repo: https://github.com/natir/fpa
- name: yacrd
  role: Chimeric-read detection over all-against-all read mapping pile-up coverage
  repo: https://github.com/natir/yacrd
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:855afed240a3df7b82bae85663d944e20d0ad0f13d7deb06c65fabd8059e4bdf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - Racon
  - VeChat
  - fpa
  - minimap2
  - yacrd
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# sequencing-platform-parameter-configuration

## Summary

Configure sequencing-platform-specific parameters (PacBio vs Oxford Nanopore) when running VeChat's two-cycle long-read error-correction workflow and its Minimap2 overlap stage, since read chemistry and error profiles determine preset choices such as --platform and ava-pb/ava-ont overlap modes.

## When to use

Use when running VeChat (or its Minimap2 overlap step) on raw long reads and the input platform (PacBio PB or ONT) must be declared so platform-appropriate alignment presets and error models are applied.

## When NOT to use

- Input reads are already corrected or are short Illumina reads — VeChat is a self-correction method for noisy long TGS reads (PacBio/ONT)
- The analysis goal is polishing an assembly rather than correcting raw long reads — Racon is the consensus/polishing module in this toolchain
- A fixed reference genome exists for correction — VeChat performs haplotype-aware self-correction against ad-hoc variation graphs built from the reads themselves

## Inputs

- Raw long-read sequencing file in FASTA or FASTQ (gzip-compressed allowed), e.g. example/reads.fq.gz
- Sequencing platform identifier: pb (Pacific Biosciences) or ont (Oxford Nanopore Technologies)

## Outputs

- Corrected reads file reads.corrected.fa (valid FASTA, one corrected sequence per input read)
- Pre-corrected full-length reads after Cycle 1 concatenation (intermediate)

## How to apply

Before running the VeChat two-cycle workflow, set the platform flag explicitly: --platform pb (default) for Pacific Biosciences CLR reads or --platform ont for Oxford Nanopore reads, matching the platform to the chemistry of the input FASTA/FASTQ (gzip allowed, e.g. example/reads.fq.gz). The platform choice drives Cycle 1's minimizer-based all-vs-all overlaps computed with Minimap2, which exposes matching platform presets (ava-pb for PacBio read overlap, ava-ont for Nanopore read overlap). Keep the platform consistent across both cycles: Cycle 1 uses minimizer overlaps while Cycle 2 recomputes all-vs-all overlaps with base-level alignments for haplotype-aware filtration at minimum overlap sequence identity 0.99 (--min-identity-cns). Other graph-filtration thresholds (--min-confidence 0.2, --min-support 0.2) are independent of platform but tuned per dataset; use --split for FASTQ > 20G or FASTA > 10G and --scrub for chimeric reads. Correct platform declaration is essential because VeChat's accuracy gains (4–15x fewer errors for PacBio, 1–10x for ONT vs state of the art) were benchmarked separately per chemistry.

## Related tools

- **VeChat** (Error-correction pipeline whose --platform flag (pb or ont, default pb) selects the sequencing platform configuration) — https://github.com/HaploKit/vechat
- **minimap2** (Computes minimizer-based all-vs-all read overlaps in Cycle 1 and base-level overlaps in Cycle 2, using platform presets (e.g. ava-pb / ava-ont)) — https://github.com/lh3/minimap2
- **Racon** (Prior-art consensus module (consensus template approach) that VeChat replaces with variation-graph-based correction) — https://github.com/lbcb-sci/racon
- **fpa** (Filter of pairwise (all-against-all read mapping) overlaps used in the VeChat pipeline) — https://github.com/natir/fpa
- **yacrd** (Chimeric-read detection over all-against-all read mapping pile-up coverage) — https://github.com/natir/yacrd

## Examples

```
minimap2 -x ava-pb reads.fa reads.fa > overlaps.paf
```

## Evaluation signals

- Output file reads.corrected.fa exists and is a valid FASTA containing one corrected sequence per input read
- The --platform value matches the actual chemistry of the input reads (pb for PacBio CLR, ont for ONT); mismatches invalidate overlap presets
- Cycle 2 haplotype-aware overlap filtration applies the minimum sequence identity between read overlaps in the consensus round (default 0.99 via --min-identity-cns)
- Graph-pruning thresholds remain at defaults unless deliberately changed: --min-confidence 0.2 and --min-support 0.2
- Benchmarked error-rate improvement should fall in the reported range: 4–15x fewer errors (PacBio) and 1–10x fewer errors (ONT) vs state-of-the-art correctors

## Limitations

- VeChat has higher runtimes and peak memory usage on large datasets and does not win in terms of computational resources
- VeChat uses off-the-shelf approaches (e.g. Minimap2, edit-distance and sequence-to-graph alignments) without exploiting all their features, causing avoidable overheads
- Optimal combinations of preprocessing, error correction and assembly for polyploid, cancer or metagenomic settings are beyond the scope of the study and need further exploration

## Evidence

- [methods] set the sequencing platform via --platform (pb or ont, default pb): [evidence span withheld — anchor status: not_found]
- [results] step 1 computes minimizer based all-vs-all overlaps, for which we employ Minimap2: "step 1 computes minimizer based all-vs-all overlaps, for which we employ Minimap2"
- [readme] Minimap2 seamlessly works with gzip'd FASTA and FASTQ formats as input.: [evidence span withheld — anchor status: external_doc]
- [abstract] 4 to 15 (Pacific Biosciences) and 1 to 10 times (Oxford Nanopore Technologies) less errors: "long reads corrected by VeChat contain 4 to 15 (Pacific Biosciences) and 1 to 10 times (Oxford Nanopore Technologies) less errors than when being corrected by state of the art approaches"
- [methods] minimum sequence identity between read overlaps in the consensus round (default: 0.99): "minimum sequence identity between read overlaps in the consensus round (default: 0.99)"
