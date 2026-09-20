---
name: truvari-callset-comparison-against-giab
description: Use when you have SV callsets (VCF) from one or more callers and a truth
  benchmark (GIAB HG002 Tier1 VCF) and need precision/recall/F1 stratified by SV type
  (deletions vs insertions >50 bp) or by sequencing coverage.
when_to_use_negative:
- Callsets contain variant classes other than insertions and deletions longer than
  50 bp, since Blackbird's out_50.vcf and this benchmarking protocol target only indels
  >50 bp
- No curated truth set exists for the genome/sample being evaluated (the protocol
  depends on the GIAB HG002 Tier1 benchmark on GRCh37)
- You need to benchmark SNVs or short variants rather than structural variations in
  the 50–10,000 bp range
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: Truvari
  role: Compares caller SV VCFs against the GIAB HG002_SVs_Tier1_v0.6 benchmark to
    yield precision, recall, and F1 per SV type and coverage level
- name: Blackbird
  role: Produces the out_50.vcf SV callset (hybrid SLR + low-coverage long-read mode)
    that is benchmarked
  repo: https://github.com/1dayac/Blackbird
- name: Sniffles2
  role: Long-read SV caller whose downsampled-coverage callsets are compared as a
    state-of-the-art baseline
- name: PBSV
  role: Long-read SV caller whose downsampled-coverage callsets are compared as a
    state-of-the-art baseline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:e6c1e19225c0c564afd9acdb372181a7ade94169b11fce10fed180c1bf9cc51c
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
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# truvari-callset-comparison-against-giab

## Summary

Benchmark structural variant caller outputs (e.g. Blackbird out_50.vcf, Sniffles2, PBSV VCFs) against the GIAB HG002_SVs_Tier1_v0.6 benchmark callset on GRCh37 using Truvari, computing per-SV-type precision, recall, and F1 to assess caller performance across long-read coverage levels.

## When to use

Use when you have SV callsets (VCF) from one or more callers and a truth benchmark (GIAB HG002 Tier1 VCF) and need precision/recall/F1 stratified by SV type (deletions vs insertions >50 bp) or by sequencing coverage.

## When NOT to use

- Callsets contain variant classes other than insertions and deletions longer than 50 bp, since Blackbird's out_50.vcf and this benchmarking protocol target only indels >50 bp
- No curated truth set exists for the genome/sample being evaluated (the protocol depends on the GIAB HG002 Tier1 benchmark on GRCh37)
- You need to benchmark SNVs or short variants rather than structural variations in the 50–10,000 bp range

## Inputs

- Caller SV VCF (e.g. Blackbird out_50.vcf, position-sorted, containing insertion and deletion calls longer than 50 bp)
- GIAB HG002_SVs_Tier1_v0.6 benchmark callset (GRCh37)
- Downsampled PacBio HiFi BAMs at 5x, 10x, 20x, and 34x coverage (indirectly, via caller VCFs)
- Caller-coverage configuration (Blackbird hybrid with TELL-Seq 42x or stLFR 35x SLR BAM; Sniffles2; PBSV)

## Outputs

- Truvari benchmark output per caller-coverage pair
- Precision, recall, and F1-scores computed separately for deletions and insertions at each coverage level
- F1-vs-coverage table (blackbird_coverage_saturation.csv)
- Saturation curves plotting F1 against long-read coverage per caller

## How to apply

For each caller-coverage pair, run Truvari in benchmark mode comparing the caller VCF (e.g. Blackbird's out_50.vcf, which holds insertion and deletion calls longer than 50 bp) against the GIAB HG002_SVs_Tier1_v0.6 benchmark callset on GRCh37, restricting analysis to insertions and deletions longer than 50 bp. In the Blackbird coverage-saturation workflow this is done for Blackbird hybrid runs on TELL-Seq (42x) or stLFR (35x) SLR data combined with downsampled PacBio HiFi BAMs (5x, 10x, 20x, 34x), and for Sniffles2 and PBSV run on the same downsampled BAMs. Parse the Truvari outputs to compute precision, recall, and F1 separately for deletions and insertions at each coverage level. Tabulate F1 versus coverage per caller (e.g. blackbird_coverage_saturation.csv) and plot saturation curves to identify the coverage at which each caller saturates.

## Related tools

- **Truvari** (Compares caller SV VCFs against the GIAB HG002_SVs_Tier1_v0.6 benchmark to yield precision, recall, and F1 per SV type and coverage level)
- **Blackbird** (Produces the out_50.vcf SV callset (hybrid SLR + low-coverage long-read mode) that is benchmarked) — https://github.com/1dayac/Blackbird
- **Sniffles2** (Long-read SV caller whose downsampled-coverage callsets are compared as a state-of-the-art baseline)
- **PBSV** (Long-read SV caller whose downsampled-coverage callsets are compared as a state-of-the-art baseline)

## Examples

```
truvari bench -b HG002_SVs_Tier1_v0.6.GRCh37.vcf.gz -c blackbird_5x/out_50.vcf -o truvari_blackbird_5x --no-ref
```

## Evaluation signals

- Each Truvari run completes for every caller-coverage pair (Blackbird TELL-Seq/stLFR x 5x/10x/20x/34x HiFi, Sniffles2, PBSV) on the GRCh37 HG002_SVs_Tier1_v0.6 benchmark
- Precision, recall, and F1 are recoverable separately for deletions and insertions at each coverage level from the Truvari outputs
- Reproduced saturation matches the reported behavior: Blackbird reaches F1 of 0.835 (deletions) and 0.808 (insertions) at only 5x long-read coverage, similar to PBSV and Sniffles2 at 10x
- blackbird_coverage_saturation.csv contains one F1-per-caller-per-coverage row set and the plotted saturation curves show Sniffles2/PBSV saturating only at 10–20x while Blackbird saturates earlier
- All VCFs compared are position-sorted and restricted to insertions/deletions longer than 50 bp

## Limitations

- Sniffles2 and PBSV saturate only at 10–20x long-read coverage on the HG002 dataset, so conclusions below that coverage for these callers are unreliable
- Only insertions and deletions longer than 50 bp are assessed; Blackbird's out.vcf contains additional calls that are 'not reliable' and are excluded
- Benchmarking depends on the GIAB HG002 Tier1 v0.6 truth set on GRCh37 and cannot be transferred to genomes without a curated SV truth set

## Evidence

- [methods] out_50.vcf is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp.: "`out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [abstract] Blackbird requires only 5× coverage to achieve F1-scores of 0.835 (deletions) and 0.808 (insertions), matching PBSV and Sniffles2 at 10× PacBio Hi-Fi coverage.: "Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi long-read coverage."
- [results] Results saturate only at 10–20× long-read coverage for Sniffles2 and PBSV, whereas Blackbird achieves saturation earlier with comparable or better recall at lower coverage.: [evidence span withheld — anchor status: not_found]
- [readme] out.vcf contains all other calls, though they are not reliable.: "`out.vcf` contains all other calls, though they are not reliable."
