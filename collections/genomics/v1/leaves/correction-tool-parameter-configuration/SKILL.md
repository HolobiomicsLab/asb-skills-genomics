---
name: correction-tool-parameter-configuration
description: Use when running VeChat (or Racon/CONSENT/Canu/Daccord baselines) on
  PacBio CLR or ONT read sets and you must choose --platform, graph-edge thresholds,
  --split/--scrub, or verify residual error-rate targets.
when_to_use_negative:
- The goal is exploring optimal combinations of preprocessing, error correction and
  core assembly tools for polyploid, cancer or metagenomic settings — the article
  states this requires further efforts beyond its scope.
- Runtime and peak memory are the binding constraints on very large datasets, where
  VeChat has longer runtimes and higher peak memory than the competing correctors.
- The target is polishing assembled contigs (Racon's intended use as a standalone
  consensus module) rather than configuring self-correction of raw long reads as in
  this benchmark.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: VeChat
  role: 'Target corrector being configured: --platform pb/ont, --min-confidence/--min-support
    0.2, --min-identity-cns 0.99, --split for large datasets, --scrub for chimeras'
  repo: https://github.com/HaploKit/vechat
- name: Minimap2
  role: VeChat dependency providing minimizer-based all-vs-all overlaps for graph
    construction; also used to align corrected reads to haplotypes for QV evaluation
  repo: https://github.com/lh3/minimap2
- name: fpa
  role: Pinned dependency (fpa=0.5) filtering all-against-all read mapping by internal
    match, containment, dovetails, and overlap/read length
  repo: https://github.com/natir/fpa
- name: yacrd
  role: Pinned dependency detecting chimeric and not-covered reads from per-read pile-up
    coverage prior to correction
  repo: https://github.com/natir/yacrd
- name: Racon
  role: Baseline consensus corrector run with recommended settings on the same read
    sets for the parameter comparison
  repo: https://github.com/lbcb-sci/racon
- name: Canu
  role: Baseline whose error-correction module is run with recommended settings in
    the benchmark
- name: Daccord
  role: Baseline corrector based on raising local de Bruijn graphs in the benchmark
    comparison
- name: CONSENT
  role: Baseline corrector combining MSA with de Bruijn graphs in the benchmark comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:23816d3f51d78813fb7e09ee7d97b199af94ee5f4877540775a6fb9bfbc30266
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - CONSENT
  - Canu
  - Daccord
  - Minimap2
  - Racon
  - VeChat
  - fpa
  - yacrd
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# correction-tool-parameter-configuration

## Summary

Set up and parameterize long-read error-correction runs: install VeChat with pinned dependencies (Minimap2, yacrd, fpa=0.5), choose --platform and graph-edge thresholds, and run Racon/Canu/Daccord/CONSENT baselines with recommended settings so corrected-read quality is benchmark-comparable.

## When to use

Use when running VeChat (or Racon/CONSENT/Canu/Daccord baselines) on PacBio CLR or ONT read sets and you must choose --platform, graph-edge thresholds, --split/--scrub, or verify residual error-rate targets.

## When NOT to use

- The goal is exploring optimal combinations of preprocessing, error correction and core assembly tools for polyploid, cancer or metagenomic settings — the article states this requires further efforts beyond its scope.
- Runtime and peak memory are the binding constraints on very large datasets, where VeChat has longer runtimes and higher peak memory than the competing correctors.
- The target is polishing assembled contigs (Racon's intended use as a standalone consensus module) rather than configuring self-correction of raw long reads as in this benchmark.

## Inputs

- Raw long-read FASTQ (e.g., reads.fq.gz, gzipped) from PacBio CLR or ONT sequencing
- PBSIM2-simulated read sets over NCBI E. coli references: diploid, triploid (three E. coli strains) and tetraploid mixtures, plus low- and high-complexity metagenomes at 30x coverage per haplotype and 10% simulated error rate
- Small test read file example/reads.fq.gz bundled with the VeChat repository

## Outputs

- Corrected reads in FASTA via -o reads.corrected.fa
- error_rate_table.tsv: residual sequencing error rates (QV) per tool, platform, and ploidy/metagenome condition

## How to apply

Install via `conda install -c bioconda vechat` with dependencies Minimap2, yacrd and fpa=0.5, then run `vechat reads.fq.gz -t 8 --platform pb -o reads.corrected.fa` for PacBio CLR input (or `--platform ont` for ONT), keeping the benchmark defaults --min-confidence 0.2, --min-support 0.2 and --min-identity-cns 0.99, which govern which edges are kept in the variation graph and the sequence identity between overlaps in the consensus round. Add --split on large (e.g., metagenome-scale) datasets to curb peak memory, and --scrub only if chimeric reads are a concern since it defaults to False. The two-cycle algorithm relies on Minimap2 minimizer-based all-vs-all overlaps feeding POA-built variation graphs, so verify the pinned dependency versions (fpa=0.5) before benchmarking. Run Racon, the Canu error-correction module, Daccord and CONSENT on the same read sets with their recommended settings, then align each tool's corrected reads back to the reference haplotypes and tabulate residual error rates (QV) per tool, platform and ploidy/metagenome condition in error_rate_table.tsv. Acceptance: VeChat shows ≈0.014% error vs Racon 0.276% on PacBio diploid E. coli, with improvements inside the 14–30x (diploid), 9–26x (triploid) and 4–11x (tetraploid) ranges.

## Related tools

- **VeChat** (Target corrector being configured: --platform pb/ont, --min-confidence/--min-support 0.2, --min-identity-cns 0.99, --split for large datasets, --scrub for chimeras) — https://github.com/HaploKit/vechat
- **Minimap2** (VeChat dependency providing minimizer-based all-vs-all overlaps for graph construction; also used to align corrected reads to haplotypes for QV evaluation) — https://github.com/lh3/minimap2
- **fpa** (Pinned dependency (fpa=0.5) filtering all-against-all read mapping by internal match, containment, dovetails, and overlap/read length) — https://github.com/natir/fpa
- **yacrd** (Pinned dependency detecting chimeric and not-covered reads from per-read pile-up coverage prior to correction) — https://github.com/natir/yacrd
- **Racon** (Baseline consensus corrector run with recommended settings on the same read sets for the parameter comparison) — https://github.com/lbcb-sci/racon
- **Canu** (Baseline whose error-correction module is run with recommended settings in the benchmark)
- **Daccord** (Baseline corrector based on raising local de Bruijn graphs in the benchmark comparison)
- **CONSENT** (Baseline corrector combining MSA with de Bruijn graphs in the benchmark comparison)

## Examples

```
vechat reads.fq.gz -t 8 --platform pb -o reads.corrected.fa
```

## Evaluation signals

- Run log shows benchmark defaults intact: --min-confidence 0.2, --min-support 0.2, --min-identity-cns 0.99; any drift from these flags changes the configuration under test.
- --platform matches the data source: 'pb' for PacBio CLR input and 'ont' for ONT input; a mismatched platform setting invalidates the run.
- Corrected output reads.corrected.fa, aligned back to reference haplotypes, gives VeChat residual error ≈ 0.014% vs Racon 0.276% on PacBio diploid E. coli.
- Reproduced improvement ratios fall within reported ranges: 14–30x on diploid, 9–26x on triploid, 4–11x on tetraploid genomes relative to the four baselines.
- On large datasets --split is enabled and peak memory stays within budget, since VeChat otherwise exceeds baseline memory usage.

## Limitations

- VeChat does not win on computational resources: longer runtimes and higher peak memory usage, particularly on large datasets; --split mitigates memory at dataset scale.
- VeChat uses off-the-shelf approaches without exploiting all their features, causing unnecessary overheads (e.g., edit-distance-based and sequence-to-graph alignments could be replaced by more efficient routines).
- Optimal combinations of preprocessing, error correction and core assembly tools for polyploid, cancer or metagenomic settings remain unexplored and require further efforts.
- --scrub (chimera removal) defaults to False, so chimeric reads pass through unless explicitly enabled.

## Evidence

- [abstract] Error correction is the canonical first step in long-read sequencing data analysis: "Error correction is the canonical first step in long-read sequencing data analysis"
- [methods] min-confidence default 0.2: "-d MIN_CONFIDENCE, --min-confidence MIN_CONFIDENCE minimum confidence for keeping edges in the graph (default: 0.2)"
- [methods] min-identity-cns default 0.99: "minimum sequence identity between read overlaps in the consensus round (default: 0.99)"
- [results] Minimap2 overlaps in cycle 1: "step 1 computes minimizer based all-vs-all overlaps, for which we employ Minimap229"
- [results] reported improvement ranges: "VeChat achieves approximately 14–30, 9–26 and 4–11 times lower error rates on diploid, triploid and tetraploid genomes, respectively."
- [discussion] runtime/memory caveat: "VeChat does not win the competition in terms of computational resources. In particular on large datasets, VeChat experiences longer runtimes and higher peak memory usage"
- [readme] Racon platform support: [evidence span withheld — anchor status: external_doc]
