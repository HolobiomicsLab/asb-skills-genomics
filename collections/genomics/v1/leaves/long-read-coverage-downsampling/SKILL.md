---
name: long-read-coverage-downsampling
description: Use when you must quantify how much long-read coverage an SV caller needs
  to match recall/precision, by comparing caller F1 against a GIAB truth set across
  a downsampled HiFi coverage series.
when_to_use_negative:
- Only a single full-coverage long-read callset is available and the goal is one-shot
  SV calling rather than coverage-saturation benchmarking
- The variant classes of interest are outside the 50-10,000 bp medium-size range that
  Blackbird targets
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Blackbird
  role: Hybrid SV caller run on SLR + downsampled long-read data to test how little
    HiFi coverage preserves call quality
  repo: https://github.com/1dayac/Blackbird
- name: Sniffles2
  role: State-of-the-art long-read SV caller used as the coverage-saturation baseline
- name: PBSV
  role: State-of-the-art long-read SV caller used as the coverage-saturation baseline
- name: Truvari
  role: Compares each caller VCF against the GIAB HG002 truth set to derive precision/recall/F1
    per coverage level
- name: BWA
  role: Provides the indexed reference genome required by Blackbird
- name: SPAdes for Blackbird
  role: Local assembly engine invoked by Blackbird via spades.py
  repo: https://github.com/1dayac/spades_for_blackbird
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:02dcd656f782aab93925620cbcb003109a15d49d111acd72b38c182f39215154
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - PBSV
  - SPAdes for Blackbird
  - Sniffles2
  - Truvari
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# long-read-coverage-downsampling

## Summary

Downsample PacBio HiFi long reads to a coverage ladder (5x, 10x, 20x, 34x) and benchmark SV callers (Blackbird hybrid mode vs Sniffles2 and PBSV) to identify the coverage at which F1-scores saturate on the HG002 GIAB benchmark.

## When to use

Use when you must quantify how much long-read coverage an SV caller needs to match state-of-the-art recall/precision, by comparing caller F1 against a GIAB truth set across a downsampled HiFi coverage series.

## When NOT to use

- Only a single full-coverage long-read callset is available and the goal is one-shot SV calling rather than coverage-saturation benchmarking
- The variant classes of interest are outside the 50-10,000 bp medium-size range that Blackbird targets

## Inputs

- Position-sorted, indexed PacBio HiFi BAM downsampled to 5x, 10x, 20x, 34x coverage
- Position-sorted, indexed SLR BAM with BX tags (TELL-Seq 42x or stLFR 35x)
- FASTQ of the same long reads (for Blackbird hybrid mode)
- BWA-indexed GRCh37/hg19 reference genome FASTA
- GIAB HG002_SVs_Tier1_v0.6 (GRCh37) SV benchmark VCF

## Outputs

- out_50.vcf per caller-coverage pair (position-sorted VCF of insertions and deletions > 50 bp)
- Truvari comparison outputs vs the GIAB benchmark callset
- Per-coverage precision, recall, and F1-scores for deletions and insertions (e.g. blackbird_coverage_saturation.csv)
- Saturation curves of F1 vs long-read coverage per caller

## How to apply

Prepare position-sorted, indexed downsampled PacBio HiFi BAMs at 5x, 10x, 20x, and 34x coverage against a BWA-indexed GRCh37 reference. Run Blackbird in hybrid mode on the full SLR datasets (TELL-Seq 42x or stLFR 35x BAM with BX tags) paired with each downsampled long-read BAM (--long-bam) and FASTQ (--long-read) to emit out_50.vcf per coverage level, and run Sniffles2 and PBSV on the same BAMs for comparison. Evaluate each caller-coverage pair with Truvari against GIAB HG002_SVs_Tier1_v0.6, restricting to insertions and deletions longer than 50 bp, and compute precision, recall, and F1 separately for deletions and insertions. Plot F1 vs coverage per caller to identify the saturation point; validation expects Blackbird to reach F1 0.835 (deletions) and 0.808 (insertions) at 5x, comparable to PBSV/Sniffles2 at 10x HiFi coverage.

## Related tools

- **Blackbird** (Hybrid SV caller run on SLR + downsampled long-read data to test how little HiFi coverage preserves call quality) — https://github.com/1dayac/Blackbird
- **Sniffles2** (State-of-the-art long-read SV caller used as the coverage-saturation baseline)
- **PBSV** (State-of-the-art long-read SV caller used as the coverage-saturation baseline)
- **Truvari** (Compares each caller VCF against the GIAB HG002 truth set to derive precision/recall/F1 per coverage level)
- **BWA** (Provides the indexed reference genome required by Blackbird)
- **SPAdes for Blackbird** (Local assembly engine invoked by Blackbird via spades.py) — https://github.com/1dayac/spades_for_blackbird

## Examples

```
blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_hybrid_5x -l pacbio_hifi_5x.bam -m pacbio_hifi_5x.fastq -t 32 -s spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- Each Blackbird run produces a position-sorted out_50.vcf containing only insertion/deletion calls longer than 50 bp
- F1 vs coverage curves show Sniffles2 and PBSV saturating only at 10-20x HiFi coverage, while Blackbird hybrid mode saturates earlier
- At 5x HiFi coverage, Blackbird reaches F1 ~ 0.835 (deletions) and ~ 0.808 (insertions), matching PBSV/Sniffles2 at 10x
- Truvari results are restricted consistently to indels > 50 bp against HG002_SVs_Tier1_v0.6 (GRCh37) for all caller-coverage pairs
- Long-read BAM inputs are position-sorted and indexed and paired correctly with a matching FASTQ for Blackbird's -l/-m options

## Limitations

- out.vcf calls other than out_50.vcf are described as 'not reliable'; only events > 50 bp should be used
- SLR mode requires barcode-containing (BX tag) position-sorted BAMs; results depend on the SLR chemistry (TELL-Seq vs stLFR)
- Findings are established on the HG002 GIAB benchmark (GRCh37); saturation behavior may differ on other genomes or callers' versions
- Blackbird was forked from the SPAdes codebase and requires a separately compiled spades_for_blackbird installation

## Evidence

- [abstract] Blackbird requires only 5× coverage to achieve F1-scores similar to PBSV and Sniffles2 at 10× HiFi.: "Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi long-read coverage."
- [methods] Hybrid-mode Blackbird matches state-of-the-art long-read tools with less long-read data.: "In a hybrid-mode Blackbird demonstrated results similar to state-of-the-art long read tools, but requires less long reads to achieve same results."
- [readme] Blackbird outputs a position-sorted VCF of insertion and deletion calls longer than 50 bp.: "`out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [readme] Hybrid mode pairs a long-read BAM with a FASTQ of the same reads.: "`--long-bam` or `-l` - Position-sorted and indexed long read file (optional, should be paired with long-read option)"
