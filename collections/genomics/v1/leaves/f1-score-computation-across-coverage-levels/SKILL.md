---
name: f1-score-computation-across-coverage-levels
description: Use when benchmarking SV callers on downsampled PacBio HiFi BAMs (e.g.
  5x, 10x, 20x, 34x) to quantify how precision/recall/F1 for deletions and insertions
  >50 bp varies with coverage against a truth set like GIAB HG002.
when_to_use_negative:
- Evaluating variant types other than insertions and deletions longer than 50 bp —
  Blackbird's out_50.vcf and the GIAB Tier1 comparison are restricted to this event
  class
- Benchmarking on genomes without an available GIAB-style high-confidence truth set,
  where Truvari-based precision/recall computation cannot be trusted
- Comparing callers that do not produce VCF callsets compatible with Truvari input
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Blackbird
  role: Hybrid-mode SV caller run on SLR BAM plus downsampled long-read BAM to produce
    out_50.vcf at each coverage level
  repo: https://github.com/1dayac/Blackbird
- name: Sniffles2
  role: Long-read SV caller baseline run on the same downsampled PacBio HiFi BAMs
    to produce comparable VCF callsets
- name: PBSV
  role: Long-read SV caller baseline run on the same downsampled PacBio HiFi BAMs
    to produce comparable VCF callsets
- name: Truvari
  role: Benchmarks each caller-coverage callset against the GIAB HG002_SVs_Tier1_v0.6
    truth set to compute precision, recall, and F1
- name: BWA
  role: Provides the BWA-indexed reference genome required by Blackbird (--rerefence/-r)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:e67ee0c5ea9d05c81b01940d8c0f3e15df73536c14793d77723caf802cebab66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - PBSV
  - Sniffles2
  - Truvari
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# f1-score-computation-across-coverage-levels

## Summary

Benchmark protocol for computing precision, recall, and F1-scores for structural variant (SV) calling across multiple long-read coverage levels, comparing Blackbird hybrid mode (SLR + PacBio HiFi) against Sniffles2 and PBSV to determine the minimum coverage at which each caller saturates.

## When to use

Use when benchmarking SV callers on downsampled PacBio HiFi BAMs (e.g. 5x, 10x, 20x, 34x) to quantify how precision/recall/F1 for deletions and insertions >50 bp varies with coverage against a truth set like GIAB HG002.

## When NOT to use

- Evaluating variant types other than insertions and deletions longer than 50 bp — Blackbird's out_50.vcf and the GIAB Tier1 comparison are restricted to this event class
- Benchmarking on genomes without an available GIAB-style high-confidence truth set, where Truvari-based precision/recall computation cannot be trusted
- Comparing callers that do not produce VCF callsets compatible with Truvari input

## Inputs

- Position-sorted and indexed SLR BAM file with BX tags (TELL-Seq 42x or stLFR 35x)
- Downsampled PacBio HiFi BAM files at 5x, 10x, 20x, and 34x coverage
- BWA-indexed GRCh37 reference genome FASTA
- GIAB HG002_SVs_Tier1_v0.6 benchmark VCF callset (GRCh37)

## Outputs

- out_50.vcf per caller-coverage pair (position-sorted VCF of insertions and deletions longer than 50 bp)
- Truvari comparison outputs per caller-coverage pair
- blackbird_coverage_saturation.csv with F1 vs coverage per caller
- Saturation curves plotting F1-score against long-read coverage for each caller

## How to apply

Combine SLR data (TELL-Seq 42x or stLFR 35x BAM with BX tags) with each downsampled PacBio HiFi BAM and run Blackbird in hybrid mode to emit out_50.vcf per coverage level; run Sniffles2 and PBSV on the same downsampled BAMs for comparison. For each caller-coverage pair, run Truvari against the GIAB HG002_SVs_Tier1_v0.6 benchmark callset (GRCh37), restricting to insertions and deletions longer than 50 bp. From Truvari outputs compute precision, recall, and F1 for deletions and insertions separately at each coverage, tabulate F1 vs coverage per caller, and plot saturation curves. Validation compares reproduced saturation against the reported finding that Blackbird reaches F1 of 0.835 (deletions) and 0.808 (insertions) at only 5x coverage, similar to PBSV and Sniffles2 at 10x PacBio Hi-Fi.

## Related tools

- **Blackbird** (Hybrid-mode SV caller run on SLR BAM plus downsampled long-read BAM to produce out_50.vcf at each coverage level) — https://github.com/1dayac/Blackbird
- **Sniffles2** (Long-read SV caller baseline run on the same downsampled PacBio HiFi BAMs to produce comparable VCF callsets)
- **PBSV** (Long-read SV caller baseline run on the same downsampled PacBio HiFi BAMs to produce comparable VCF callsets)
- **Truvari** (Benchmarks each caller-coverage callset against the GIAB HG002_SVs_Tier1_v0.6 truth set to compute precision, recall, and F1)
- **BWA** (Provides the BWA-indexed reference genome required by Blackbird (--rerefence/-r))

## Examples

```
blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_5x -l hifi_5x.bam -m hifi_5x.fastq -t 32 -s spades.py; truvari bench -b HG002_SVs_Tier1_v0.6.vcf -c blackbird_5x/out_50.vcf
```

## Evaluation signals

- Truvari reports F1 for deletions and insertions separately at each coverage level; Blackbird with TELL-Seq should reach F1 ≈ 0.835 (deletions) and ≈ 0.808 (insertions) at only 5x PacBio HiFi coverage
- Reproduced Sniffles2 and PBSV F1 curves should saturate only at 10–20x long-read coverage, matching the reported finding
- out_50.vcf files must be position-sorted and contain only insertion/deletion calls longer than 50 bp
- blackbird_coverage_saturation.csv should contain one F1 value per caller-coverage-type combination with monotonic or saturating trends as coverage increases
- Coverage downsampling points (5x, 10x, 20x, 34x) must be consistently applied to the same PacBio HiFi BAM for all three callers

## Limitations

- SVs are restricted to the 50–10,000 bp range; shorter or larger events and other variant classes are not addressed by this benchmark
- Results are specific to the HG002 GIAB benchmark on GRCh37 and may not generalize to other genomes or truth sets
- Blackbird's out.vcf contains additional calls that are 'not reliable', so only out_50.vcf should feed the F1 computation

## Evidence

- [abstract] Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi long-read coverage.: "Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi long-read coverage."
- [readme] out_50.vcf output format: "`out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [methods] hybrid mode finding: "In a hybrid-mode Blackbird demonstrated results similar to state-of-the-art long read tools, but requires less long reads to achieve same results."
- [readme] README CLI options: "`--long-bam` or `-l` - Position-sorted and indexed long read file (optional, should be paired with long-read option)"
- [abstract] HG002 GIAB benchmark scope: "On the HG002 Genome in a Bottle (GIAB) benchmark, Blackbird in hybrid mode demonstrated results comparable to state-of-the-art long-read tools, while using less long-read coverage."
