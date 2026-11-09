---
name: reference-alignment-error-rate-quantification
description: Use when benchmarking long-read error correctors (e.g. VeChat, Racon,
  CONSENT, Daccord, Canu) and you need ground-truth error rates from reference alignment,
  or when auditing whether reference-free Merqury QV estimates are method-dependent
  and biased.
when_to_use_negative:
- When no reference assembly/ground truth exists for the dataset and only reference-free
  (Merqury k-mer) estimates are possible
- When reads are short Illumina reads — minimap2 long-read presets (map-ont, map-pb)
  and this correction-benchmarking context target long noisy reads
- When the goal is a computational-resources benchmark; the article reports this skill
  addresses error-rate bias, not runtime/memory comparison
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: VeChat
  role: Produces the corrected read sets (reads.corrected.fa) whose error rates are
    quantified; run with --min-confidence 0.2, --min-support 0.2 and --platform pb/ont
  repo: https://github.com/HaploKit/vechat
- name: Merqury
  role: Provides reference-free k-mer QV and error-rate estimates from a meryl k-mer
    database of the raw reads
- name: minimap2
  role: Aligns the corrected long reads to the reference assembly (e.g. map-ont preset)
    to feed QUAST alignment-based error quantification
  repo: https://github.com/lh3/minimap2
- name: QUAST
  role: Computes alignment-based substitution and indel error counts of the corrected
    reads against the reference ground truth
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:1a669f6068c5d6ea0868030ff97dc30e9d96c75c906125b3e3fc0191810aca94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - Merqury
  - QUAST
  - VeChat
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# reference-alignment-error-rate-quantification

## Summary

Quantifies true substitution and indel error rates of corrected long reads by aligning them to a reference assembly with minimap2 and running QUAST, then compares these alignment-based rates against Merqury QV-derived error rates to expose reference-free estimation bias per correction method.

## When to use

Use when benchmarking long-read error correctors (e.g. VeChat, Racon, CONSENT, Daccord, Canu) and you need ground-truth error rates from reference alignment, or when auditing whether reference-free Merqury QV estimates are method-dependent and biased.

## When NOT to use

- When no reference assembly/ground truth exists for the dataset and only reference-free (Merqury k-mer) estimates are possible
- When reads are short Illumina reads — minimap2 long-read presets (map-ont, map-pb) and this correction-benchmarking context target long noisy reads
- When the goal is a computational-resources benchmark; the article reports this skill addresses error-rate bias, not runtime/memory comparison

## Inputs

- Corrected long-read FASTA files (e.g. reads.corrected.fa) per dataset and correction method (VeChat, Racon, CONSENT, Daccord, Canu)
- Raw uncorrected read files (FASTA/FASTQ) for meryl k-mer counting
- Reference assembly/genome per dataset (e.g. HG002 reference, human gut microbiome reference)
- minimap2 PAF/SAM alignments of corrected reads against the reference

## Outputs

- Merqury QV and QV-derived error-rate estimates per dataset/corrector
- QUAST alignment-based substitution and indel error counts per dataset/corrector
- Underestimation fold-difference per dataset/corrector (QUAST error rate / Merqury QV-derived error rate)
- error_rate_comparison.csv with rows per dataset/corrector and columns for Merqury QV, QUAST error rate, and fold-difference

## How to apply

Take the corrected read files (e.g. reads.corrected.fa from VeChat runs with --min-confidence 0.2, --min-support 0.2 and the platform-appropriate setting) for each dataset such as HG002 ONT diploid and human gut microbiome data. Build a k-mer count database from the raw uncorrected reads with meryl and run Merqury to obtain reference-free QV and error-rate estimates; in parallel, map the corrected reads to the corresponding reference assembly using minimap2 (e.g. -x map-ont for ONT data) and run QUAST on the alignments to get substitution and indel error counts against ground truth. For each dataset/corrector, compute the underestimation factor as the QUAST alignment-based error rate divided by the Merqury QV-derived error rate, and tabulate Merqury QV, QUAST error rate, and fold-difference per dataset/corrector (e.g. in error_rate_comparison.csv). The rationale is that Merqury underestimates true error on ONT diploid genomes in a method-dependent manner, so the QUAST-vs-QV fold-difference is the bias diagnostic.

## Related tools

- **VeChat** (Produces the corrected read sets (reads.corrected.fa) whose error rates are quantified; run with --min-confidence 0.2, --min-support 0.2 and --platform pb/ont) — https://github.com/HaploKit/vechat
- **Merqury** (Provides reference-free k-mer QV and error-rate estimates from a meryl k-mer database of the raw reads)
- **minimap2** (Aligns the corrected long reads to the reference assembly (e.g. map-ont preset) to feed QUAST alignment-based error quantification) — https://github.com/lh3/minimap2
- **QUAST** (Computes alignment-based substitution and indel error counts of the corrected reads against the reference ground truth)

## Examples

```
minimap2 -ax map-ont ref.fa reads.corrected.fa > corrected.sam; quast.py -r ref.fa corrected.sam -o quast_out
```

## Evaluation signals

- error_rate_comparison.csv contains both Merqury QV and QUAST alignment-based error rates for every dataset/corrector pair (e.g. both HG002 and human gut microbiome)
- All derived fold-differences (QUAST error rate / Merqury QV-derived error rate) are finite positive values
- Fold-differences are consistent in magnitude with paper-reported QV-based error-rate results on HG002 and the human gut microbiome dataset (e.g. Merqury underestimation factors on ONT diploid genomes ranging ~1.75 to 11.84 across correctors)
- minimap2 alignments were produced with the platform-appropriate preset (map-ont for ONT data) and QUAST reports both substitution and indel error categories
- Merqury was run against a meryl k-mer database built from the raw uncorrected reads, not the corrected reads

## Limitations

- Merqury-based estimates are biased: on ONT-sequenced diploid genomes it underestimates QUAST/ground-truth error rates by factors of 4.33 (CONSENT), 11.84 (Daccord), but only 2.54 (Racon), 2.05 (Canu), 1.75 (VeChat) — so Merqury QV alone cannot rank correctors fairly
- VeChat experiences longer runtimes and higher peak memory usage on large datasets, so resource-constrained settings may limit generating corrected reads for this analysis
- Exploring optimal preprocessing/error-correction/assembly pipeline combinations for polyploid, cancer, or metagenomic settings is beyond the scope of the study and requires further efforts

## Evidence

- [methods] Align the corrected reads to the corresponding reference assemblies using minimap2 and run QUAST to obtain alignment-based substitution and indel error counts per dataset.: [evidence span withheld — anchor status: not_found]
- [results] On ONT-sequenced diploid genomes, Merqury underestimates the true error rates (as measured by QUAST against ground truth) by factors of 4.33 for CONSENT and even 11.84 for Daccord.: [evidence span withheld — anchor status: not_found]
- [readme] minimap2 supports Oxford Nanopore genomic read mapping via the map-ont preset as documented in its README.: [evidence span withheld — anchor status: external_doc]
- [results] The article recommends minimap2 for minimizer-based all-vs-all overlaps in its correction workflow.: "step 1 computes minimizer based all-vs-all overlaps, for which we employ Minimap229"
