---
name: metagenome-and-diploid-genome-evaluation
description: Use when corrected long reads from multiple correctors (FASTA/FASTQ,
  ONT or PacBio) must be quality-evaluated on diploid (HG002) or metagenomic datasets,
  and reference-free QV must be validated against alignment-based ground truth.
when_to_use_negative:
- The evaluation target is haplotype-aware assembly quality (switch error) rather
  than per-read error-rate bias between k-mer and alignment metrics
- Only a single correction method is being assessed and no cross-method underestimation-factor
  comparison is intended
- The dataset is polyploid, cancer, or metagenomic and the goal is optimal preprocessing/correction/assembly
  pipeline selection — the article states this exploration is beyond its scope and
  requires further efforts
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: VeChat
  role: Generates one arm of the corrected read sets (variation-graph-based haplotype-aware
    correction) with --min-confidence 0.2 and --min-support 0.2 defaults
  repo: https://github.com/HaploKit/vechat
- name: Merqury
  role: Reference-free k-mer-based QV and error-rate estimation from meryl k-mer counts
    of the raw reads
- name: QUAST
  role: Alignment-based substitution and indel error counting against the reference
    ground truth
- name: minimap2
  role: Aligns corrected reads to the reference assembly (map-ont/map-pb presets)
    prior to QUAST evaluation
  repo: https://github.com/lh3/minimap2
- name: Racon
  role: Consensus-based correction comparison method whose corrected output feeds
    the same evaluation pipeline
  repo: https://github.com/lbcb-sci/racon
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:5f4d1b1b6706fe7aa45c41029320d8c58d26e3a83417317c81069ca539953f06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - Merqury
  - QUAST
  - Racon
  - VeChat
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metagenome-and-diploid-genome-evaluation

## Summary

Benchmark error rates of corrected long reads (e.g., VeChat, CONSENT, Racon, Daccord, Canu) on diploid (HG002) and human gut microbiome datasets by contrasting reference-free Merqury k-mer QV estimates with QUAST alignment-based error rates computed against reference ground truth, exposing method-dependent underestimation biases.

## When to use

Use when corrected long reads from multiple correctors (FASTA/FASTQ, ONT or PacBio) must be quality-evaluated on diploid (HG002) or metagenomic datasets, and reference-free QV must be validated against alignment-based ground truth.

## When NOT to use

- The evaluation target is haplotype-aware assembly quality (switch error) rather than per-read error-rate bias between k-mer and alignment metrics
- Only a single correction method is being assessed and no cross-method underestimation-factor comparison is intended
- The dataset is polyploid, cancer, or metagenomic and the goal is optimal preprocessing/correction/assembly pipeline selection — the article states this exploration is beyond its scope and requires further efforts

## Inputs

- Corrected long-read FASTA files (e.g., reads.corrected.fa) from each correction method (VeChat, CONSENT, Racon, Canu, Daccord, VeChat)
- Raw uncorrected long-read FASTQ/FASTA files (ONT or PacBio) for meryl k-mer counting
- Reference assembly/genome ground-truth FASTA per dataset (HG002, human gut microbiome)

## Outputs

- error_rate_comparison.csv with rows per dataset/corrector and columns for Merqury QV, QUAST alignment-based error rate, and fold-difference (underestimation factor)
- Per-dataset Merqury QV / error-rate reports
- Per-dataset QUAST substitution and indel error counts

## How to apply

First, correct raw reads with each method using its default settings — for VeChat use default graph-pruning thresholds (--min-confidence 0.2, --min-support 0.2) and the matching --platform pb/ont flag. Build a k-mer count database from the raw uncorrected reads with meryl and run Merqury on each corrected read set to obtain reference-free QV/error-rate estimates per dataset. Independently align the corrected reads to the corresponding reference assemblies with minimap2 (e.g., the map-ont preset for Oxford Nanopore data) and run QUAST to extract alignment-based substitution and indel error counts. For each dataset/corrector pair, compute the underestimation factor as the QUAST alignment-based error rate divided by the Merqury QV-derived error rate; the article's motivation is that on ONT-sequenced diploid genomes Merqury underestimates true error rates by factors as large as 11.84 (Daccord) versus 1.75 (VeChat). Collate rows per dataset/corrector into error_rate_comparison.csv with columns for Merqury QV, QUAST alignment error rate, and the fold-difference.

## Related tools

- **VeChat** (Generates one arm of the corrected read sets (variation-graph-based haplotype-aware correction) with --min-confidence 0.2 and --min-support 0.2 defaults) — https://github.com/HaploKit/vechat
- **Merqury** (Reference-free k-mer-based QV and error-rate estimation from meryl k-mer counts of the raw reads)
- **QUAST** (Alignment-based substitution and indel error counting against the reference ground truth)
- **minimap2** (Aligns corrected reads to the reference assembly (map-ont/map-pb presets) prior to QUAST evaluation) — https://github.com/lh3/minimap2
- **Racon** (Consensus-based correction comparison method whose corrected output feeds the same evaluation pipeline) — https://github.com/lbcb-sci/racon

## Examples

```
./minimap2 -ax map-ont ref.fa reads.corrected.fa > corrected_vs_ref.sam && quast.py corrected_vs_ref.sam -o quast_out
```

## Evaluation signals

- error_rate_comparison.csv contains both error metrics (Merqury QV and QUAST alignment-based rate) for both datasets (HG002 and human gut microbiome)
- All derived fold-differences are finite positive values
- Fold-difference magnitudes are consistent with the paper-reported QV-based error-rate results on HG002 and the human gut microbiome (e.g., VeChat ~1.5 times lower error rate QV than CONSENT on HG002)
- Merqury QV error rates are computed from a meryl k-mer database built on the raw uncorrected reads, not on the corrected reads
- Cross-method sanity check: on ONT diploid data, correctors whose QV-derived errors are much lower than QUAST alignment errors (e.g., Daccord factor 11.84) should stand out against low-bias correctors (e.g., VeChat factor 1.75)

## Limitations

- Merqury-based estimates systematically underestimate true (QUAST-vs-ground-truth) error rates on ONT-sequenced diploid genomes, and the size of the bias is method-dependent (4.33 for CONSENT, 11.84 for Daccord, 2.54 for Racon, 2.05 for Canu, 1.75 for VeChat), so QV alone cannot rank correctors
- VeChat does not win the competition in terms of computational resources; on large datasets it has longer runtimes and higher peak memory usage
- VeChat uses off-the-shelf approaches without exploiting all their features, causing unnecessary overheads

## Evidence

- [results] VeChat achieves approximately 1.5 times lower error rate (QV) and 1.2 times lower switch error on HG002 compared with CONSENT: "VeChat achieves approximately 1.5 times lower error rate (QV) and 1.2 times lower switch error on HG002 compared with CONSENT"
- [results] Daccord lowest QV on metagenome: "Whereas on the human gut microbiome dataset, Daccord achieves the lowest error rate (QV) while VeChat obtains comparable read accuracy."
- [methods] VeChat default graph-pruning parameters: "-d MIN_CONFIDENCE, --min-confidence MIN_CONFIDENCE minimum confidence for keeping edges in the graph (default: 0.2)"
- [readme] minimap2 ONT mapping invocation: [evidence span withheld — anchor status: external_doc]
- [abstract] VeChat graph-based consensus rationale: "Unlike consensus sequence templates, graph-based reference systems are not affected by such biases, so do not mistakenly mask true variants as errors"
