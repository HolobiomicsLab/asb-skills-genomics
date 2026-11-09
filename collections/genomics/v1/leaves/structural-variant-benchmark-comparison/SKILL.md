---
name: structural-variant-benchmark-comparison
description: Use when comparing insertion/deletion (>50 bp) SV callsets against GIAB
  HG002 or simulated ground truth across coverage levels to quantify caller recall,
  precision, F1, and long-read coverage saturation.
when_to_use_negative:
- Calling variants in a single sample without any benchmark/ground-truth SV set or
  competing caller — the skill is about comparison and evaluation, not detection itself
- Detecting SVs from short-read data alone — the article notes most SVs in the 50–10,000
  bp range cannot be resolved with short-read sequencing
- Variant sizes below 50 bp — Blackbird's reliable output (out_50.vcf) and the benchmark
  scope are restricted to events longer than 50 bp
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3175
tools:
- name: Blackbird
  role: SLR and hybrid (SLR + long-read) SV caller whose out_50.vcf callsets are benchmarked
    against competitors across coverage levels
  repo: https://github.com/1dayac/Blackbird
- name: PBSV
  role: State-of-the-art long-read SV caller run on the same PacBio HiFi BAMs as a
    competitor callset
- name: Sniffles2
  role: State-of-the-art long-read SV caller run on the same PacBio HiFi BAMs as a
    competitor callset
- name: Truvari
  role: Compares each caller's VCF to the ground-truth SV set to compute recall, precision,
    and F1 per SV type and coverage
- name: SPAdes for blackbird
  role: Forked assembler codebase required for Blackbird's local segment assembly,
    supplied via --spades
  repo: https://github.com/1dayac/spades_for_blackbird
- name: BWA
  role: Indexes the reference genome that Blackbird requires via --reference
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:d2e894a6a7faf03c6f60298636f34255a25c18f30317d41f34945878a8607786
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - Manta
  - PBSV
  - SPAdes for blackbird
  - Sniffles2
  - SvABA
  - Truvari
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# structural-variant-benchmark-comparison

## Summary

Benchmark structural variant callers (Blackbird, PBSV, Sniffles2, SvABA, Manta) by comparing their VCF callsets to a ground-truth SV set with Truvari, computing per-SV-type recall, precision, and F1 across long-read coverage levels. Used to show Blackbird hybrid mode needs only 5× PacBio HiFi coverage to match PBSV/Sniffles2 at 10×.

## When to use

Use when comparing insertion/deletion (>50 bp) SV callsets against GIAB HG002 or simulated ground truth across coverage levels to quantify caller recall, precision, F1, and long-read coverage saturation.

## When NOT to use

- Calling variants in a single sample without any benchmark/ground-truth SV set or competing caller — the skill is about comparison and evaluation, not detection itself
- Detecting SVs from short-read data alone — the article notes most SVs in the 50–10,000 bp range cannot be resolved with short-read sequencing
- Variant sizes below 50 bp — Blackbird's reliable output (out_50.vcf) and the benchmark scope are restricted to events longer than 50 bp

## Inputs

- Position-sorted, indexed SLR BAM with BX barcode tags (e.g. TELL-Seq 42× or stLFR 35× HG002)
- BWA-indexed reference genome (GRCh37/hg19 or GRCh38 FASTA)
- Downsampled PacBio HiFi long-read BAM plus matching FASTQ (5×, 10×, 20×, 34× coverage)
- Ground-truth SV callset (GIAB HG002_SVs_Tier1_v0.6 VCF or simulated 1437-insertion list)

## Outputs

- out_50.vcf — position-sorted VCF of insertion/deletion calls >50 bp per caller per coverage level
- Truvari comparison outputs with recall, precision, and F1 per SV type per caller per coverage
- Benchmark tables (benchmark_results.csv, blackbird_coverage_saturation.csv) and saturation curves

## How to apply

Run each caller on identical inputs — Blackbird in hybrid mode with the position-sorted, BX-tagged SLR BAM (-b), BWA-indexed GRCh37/GRCh38 reference (-r), output folder (-o), and paired long-read BAM (-l) plus FASTQ (-m) at each downsampled PacBio HiFi coverage (5×, 10×, 20×, 34×), and run PBSV and Sniffles2 on the same long-read BAMs alone. Restrict comparison to insertions and deletions longer than 50 bp (Blackbird emits these in out_50.vcf) and evaluate each VCF against ground truth (GIAB HG002_SVs_Tier1_v0.6 on GRCh37, or the 1437 ground-truth insertions from Huddleston & Eichler 2016) using Truvari. Compute recall, precision, and F1 separately per SV type at each coverage point, then tabulate F1 vs coverage to identify the saturation point per caller. The key decision point is the long-read coverage at which each caller saturates: Blackbird hybrid mode reaches F1 of 0.835 (deletions) and 0.808 (insertions) at 5×, similar to PBSV/Sniffles2 at 10×, and its simulated-37× SLR recall was 85–87% (deletions), 75–77% (insertions), with 90–93% precision.

## Related tools

- **Blackbird** (SLR and hybrid (SLR + long-read) SV caller whose out_50.vcf callsets are benchmarked against competitors across coverage levels) — https://github.com/1dayac/Blackbird
- **PBSV** (State-of-the-art long-read SV caller run on the same PacBio HiFi BAMs as a competitor callset)
- **Sniffles2** (State-of-the-art long-read SV caller run on the same PacBio HiFi BAMs as a competitor callset)
- **Truvari** (Compares each caller's VCF to the ground-truth SV set to compute recall, precision, and F1 per SV type and coverage)
- **SPAdes for blackbird** (Forked assembler codebase required for Blackbird's local segment assembly, supplied via --spades) — https://github.com/1dayac/spades_for_blackbird
- **BWA** (Indexes the reference genome that Blackbird requires via --reference)

## Examples

```
/home/dmm2017/Blackbird/assembler/bin/blackbird -b /local/storage/data/10X/HG002/NA24385.GRCh37.phased_possorted_bam.bam -r /local/workdir/dmm2017/hg37/refdata-hg19-2.1.0/fasta/genome.fa -o blackbird_10X_chr1 -t 32 -s /home/dmm2017/spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- out_50.vcf is produced, position-sorted, and contains only insertion/deletion calls longer than 50 bp
- Blackbird hybrid-mode F1 at 5× long-read coverage approximates 0.835 (deletions) and 0.808 (insertions), comparable to PBSV/Sniffles2 at 10×
- On simulated 37× SLR data, Blackbird hybrid recall falls in 85–87% (deletions) and 75–77% (insertions) with 90–93% precision across 5×–30× long-read coverage
- Truvari comparisons are restricted to insertions/deletions >50 bp against the correct reference build (GRCh37 GIAB Tier1 v0.6 or the Huddleston & Eichler 2016 truth set)
- Runtime/memory sanity check: Sniffles2 completes in minutes with <1 Gb, PBSV ~1h19m/27 Gb, Blackbird hybrid ~11h/117 Gb — large deviations indicate a mis-specified run

## Limitations

- Blackbird hybrid mode is slow and memory-hungry on HG002 (11 h, 117 Gb peak RSS) versus Sniffles2 (3 min, 0.6 Gb), so runtime-tolerant settings or many threads (--threads, default 1) are needed
- Calls in out.vcf (events ≤50 bp) are explicitly 'not reliable'; only out_50.vcf should enter benchmarks
- The SPAdes-for-Blackbird dependency must be compiled separately and passed via --spades/--long-bam/--long-read pairing; the hybrid options must be given together

## Evidence

- [abstract] Blackbird requires only 5× long-read coverage to achieve F1-scores of 0.835 (deletions) and 0.808 (insertions), similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi coverage.: "Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi"
- [methods] In hybrid mode, Blackbird matches state-of-the-art long-read callers but needs less coverage.: "In a hybrid-mode Blackbird demonstrated results similar to state-of-the-art long read tools, but requires less long reads to achieve same results."
- [readme] The benchmark callset is out_50.vcf with indels >50 bp.: "out_50.vcf is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [abstract] Long-read callers outperform short reads for 50–10,000 bp SVs, motivating the benchmark scope.: "most structural variations, especially within 50–10,000 bp range cannot be resolved with short-read sequencing, but long-read structural variant callers perform better"
