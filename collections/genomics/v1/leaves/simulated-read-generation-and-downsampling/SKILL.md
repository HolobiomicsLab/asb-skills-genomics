---
name: simulated-read-generation-and-downsampling
description: 'Use when benchmarking SV callers on controlled data: simulated SLR reads
  with known ground-truth SVs are needed, or long-read coverage must be systematically
  varied (5×–30×) to test coverage sensitivity.'
when_to_use_negative:
- The goal is to assess caller performance on real patient or clinical samples where
  simulation artifacts (e.g. idealized read geometry) may inflate accuracy
- The caller under test does not consume SLR barcode (BX tag) information or low-coverage
  long reads, making the SLR simulation irrelevant
- Ground-truth SVs are unavailable for the region of interest, since Truvari recall/precision
  cannot be computed
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Blackbird
  role: SV caller under test, run in SLR and hybrid (SLR + downsampled long-read)
    modes
  repo: https://github.com/1dayac/Blackbird
- name: PBSV
  role: Long-read-only comparison caller run on the same downsampled BAMs
- name: Sniffles2
  role: Long-read-only comparison caller run on the same downsampled BAMs
- name: BWA
  role: Aligns simulated SLR reads; reference must be BWA-indexed for Blackbird
- name: Truvari
  role: Computes recall and precision of each callset against the ground-truth SV
    list
- name: SPAdes
  role: Local assembly engine underlying Blackbird (spades_for_blackbird fork)
  repo: https://github.com/1dayac/spades_for_blackbird
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:f3c4126f639970b8cb79d292a15b0edbf47aa541f5849cf4fba56c0a05bc3375
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - PBSV
  - SPAdes
  - Sniffles2
  - Truvari
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Simulated read generation and downsampling

## Summary

Generates a simulated 37× synthetic long read (SLR) chromosome-1 dataset with LRSIM carrying BX barcode tags plus ground-truth SVs, then downsamples PacBio HiFi long reads from 30× to 5× to benchmark SV callers under controlled coverage conditions.

## When to use

Use when benchmarking SV callers on controlled data: simulated SLR reads with known ground-truth SVs are needed, or long-read coverage must be systematically varied (5×–30×) to test coverage sensitivity.

## When NOT to use

- The goal is to assess caller performance on real patient or clinical samples where simulation artifacts (e.g. idealized read geometry) may inflate accuracy
- The caller under test does not consume SLR barcode (BX tag) information or low-coverage long reads, making the SLR simulation irrelevant
- Ground-truth SVs are unavailable for the region of interest, since Truvari recall/precision cannot be computed

## Inputs

- GRCh38 reference genome (BWA-indexed)
- Ground-truth SV list: 1437 insertions from Huddleston & Eichler 2016
- PacBio HiFi long-read BAM/FASTQ to downsample
- LRSIM simulation configuration for 37× SLR chromosome-1 reads with BX barcode tags

## Outputs

- Simulated 37× SLR chromosome-1 BAM with BX tags, position-sorted and indexed
- Downsampled PacBio HiFi BAM + FASTQ pairs at 5×, 10×, 15×, 20×, 25×, 30× coverage
- Per-coverage VCF callsets from each SV caller
- benchmark_results.csv with recall, precision, and F1 per caller per SV type per coverage level

## How to apply

Simulate a 37× SLR chromosome-1 read set with LRSIM carrying BX barcode tags aligned against a BWA-indexed GRCh38 reference, using the 1437 inserted SVs from Huddleston & Eichler 2016 as ground truth. Align the simulated SLR reads with BWA and index the position-sorted BAM. Downsample PacBio HiFi long reads at coverage levels spanning 5× to 30×, producing one indexed BAM and matching FASTQ per coverage point. Run the SV caller(s) under test at each coverage point and compare output VCFs (calls longer than 50 bp) to the ground-truth SV list using Truvari to compute per-type (deletion, insertion) recall and precision. The key decision point is whether the caller retains acceptable F1 at low coverage — e.g. Blackbird hybrid mode reaches F1 0.835 (deletions) and 0.808 (insertions) at 5×, comparable to PBSV/Sniffles2 at 10×.

## Related tools

- **Blackbird** (SV caller under test, run in SLR and hybrid (SLR + downsampled long-read) modes) — https://github.com/1dayac/Blackbird
- **PBSV** (Long-read-only comparison caller run on the same downsampled BAMs)
- **Sniffles2** (Long-read-only comparison caller run on the same downsampled BAMs)
- **BWA** (Aligns simulated SLR reads; reference must be BWA-indexed for Blackbird)
- **Truvari** (Computes recall and precision of each callset against the ground-truth SV list)
- **SPAdes** (Local assembly engine underlying Blackbird (spades_for_blackbird fork)) — https://github.com/1dayac/spades_for_blackbird

## Examples

```
blackbird -b simulated_SLR_chr1.bam -r GRCh38.fa -o out_hybrid_5x -l hifi_5x.bam -m hifi_5x.fastq -t 32 -s spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- Downsampled BAMs are position-sorted and indexed at every coverage point from 5× to 30×
- Truvari reports per-type recall and precision; expected Blackbird hybrid-mode recall ≈85–87% (deletions) and 75–77% (insertions) with 90–93% precision across coverage levels
- Blackbird hybrid-mode F1 at 5× long-read coverage reaches 0.835 (deletions) and 0.808 (insertions), approaching PBSV/Sniffles2 results at 10×
- Only VCF calls longer than 50 bp (out_50.vcf) enter the Truvari comparison, matching the ground-truth event size range
- Simulated SLR BAM carries BX barcode tags required by Blackbird's barcode-aware assembly

## Limitations

- Results are on a simulated dataset; simulation may not capture real SLR/HiFi error profiles
- Only chromosome 1 and the 1437 insertion ground truth from Huddleston & Eichler 2016 were used, limiting scope of generalization
- Calls in out.vcf (≤50 bp or otherwise unreliable) are not dependable and are excluded from benchmarking

## Evidence

- [full_text] On the simulated 37× SLR dataset, Blackbird in hybrid mode achieved 85–87% recall for deletions, 75%–77% recall for insertions, and 90%–93% precision across all long-read coverage levels (5×–30×).: [evidence span withheld — anchor status: not_found]
- [abstract] Blackbird requires only 5× coverage to achieve F1-scores of 0.835 (deletions) and 0.808 (insertions), similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi coverage.: "Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi long-read coverage."
- [readme] Input options require position-sorted indexed BAM with BX tags and BWA-indexed reference.: "`--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags"
- [readme] Output VCF contains only reliable calls longer than 50 bp.: "`out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [methods] Blackbird uses barcode-aware sliding window local assembly rather than whole-genome assembly.: "Blackbird uses a barcode-aware
sliding window approach to assemble small segments of the target genome and sensitively call
SVs in these segments."
