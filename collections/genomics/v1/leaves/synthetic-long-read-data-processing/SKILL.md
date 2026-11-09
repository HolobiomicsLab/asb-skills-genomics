---
name: synthetic-long-read-data-processing
description: Use when raw linked-read data from 10x Genomics, TELL-Seq or stLFR platforms
  must be aligned to GRCh38 and made LongRanger-compatible before running barcode-based
  tools like Novel-X, or when comparing molecule coverage across platforms.
when_to_use_negative:
- Data are regular short reads without barcodes (e.g. standard Illumina WGS from the
  1000 Genomes pilot) — barcode-aware synthetic long-read conversion does not apply
- Input is already a LongRanger-aligned 10x BAM — no conversion is needed, proceed
  directly to the analysis pipeline
- The analysis needs whole-genome diploid assembly from stLFR reads rather than insertion
  calling — use stlfr2supernova/SuperNova directly instead
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Novel-X
  role: Downstream Snakemake pipeline consuming LongRanger-aligned 10x BAMs; requires
    recursive git clone with submodules
  repo: github.com/1dayac/Novel-X
- name: LongRanger
  role: Aligns 10x Genomics and TELL-Seq linked reads to GRCh38 producing the LongRanger-compatible
    BAM input
- name: stlfr2supernova
  role: Converts BGI stLFR raw reads into 10X-compatible format for Supernova-based
    processing
  repo: https://github.com/BGI-Qingdao/stlfr2supernova_pipeline
- name: SuperNova
  role: De novo diploid whole-genome assembler used on converted stLFR/synthetic long-read
    data
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:2b96e3cac4985353a8b84e40af267dceb7aa95b46965b0ebdd9e269e1d94999d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LongRanger
  - Novel-X
  - Paftools
  - SuperNova
  - stlfr2supernova
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# synthetic-long-read-data-processing

## Summary

Converts raw synthetic long-read sequencing data (10x Genomics Chromium, TELL-Seq, stLFR) into LongRanger-compatible BAM files aligned to GRCh38 so they can feed downstream barcode-aware analyses such as Novel-X NRS insertion calling.

## When to use

Use when raw linked-read data from 10x Genomics, TELL-Seq or stLFR platforms must be aligned to GRCh38 and made LongRanger-compatible before running barcode-based tools like Novel-X, or when comparing molecule coverage across platforms.

## When NOT to use

- Data are regular short reads without barcodes (e.g. standard Illumina WGS from the 1000 Genomes pilot) — barcode-aware synthetic long-read conversion does not apply
- Input is already a LongRanger-aligned 10x BAM — no conversion is needed, proceed directly to the analysis pipeline
- The analysis needs whole-genome diploid assembly from stLFR reads rather than insertion calling — use stlfr2supernova/SuperNova directly instead

## Inputs

- Raw paired-end FASTQ/BAM from 10x Genomics Chromium linked-read sequencing
- Raw paired-end FASTQ/BAM from TELL-Seq (UST) sequencing
- Raw paired-end FASTQ/BAM from BGI stLFR sequencing
- GRCh38 reference genome

## Outputs

- LongRanger-compatible BAM with barcode tags aligned to GRCh38
- Molecule coverage distribution summaries per platform (e.g. for NA12878)
- Pipeline-ready dataset for downstream NRS insertion calling

## How to apply

Align each platform's paired-end reads to GRCh38 so barcode (BX) tags are preserved: run LongRanger for 10x Genomics and TELL-Seq datasets, and the stlfr2supernova pipeline (BGI-Qingdao) to convert stLFR raw reads into 10X-compatible format before assembly/alignment. Verify LongRanger BAMs can be consumed directly by the downstream pipeline (e.g. Novel-X Snakemake with novel-x.py run --10x --high-coverage on the NA19240 BAM). Before running, compare molecule coverage distributions across platforms (as done for NA12878 10x/TELL-Seq/stLFR) to check that coverage is in the expected range for the tool. Note platform quirks: only stLFR provides paired reads with equal-length ends, which affects which tools can run on which datasets.

## Related tools

- **Novel-X** (Downstream Snakemake pipeline consuming LongRanger-aligned 10x BAMs; requires recursive git clone with submodules) — github.com/1dayac/Novel-X
- **LongRanger** (Aligns 10x Genomics and TELL-Seq linked reads to GRCh38 producing the LongRanger-compatible BAM input)
- **stlfr2supernova** (Converts BGI stLFR raw reads into 10X-compatible format for Supernova-based processing) — https://github.com/BGI-Qingdao/stlfr2supernova_pipeline
- **SuperNova** (De novo diploid whole-genome assembler used on converted stLFR/synthetic long-read data)

## Examples

```
git clone --recursive git@github.com:1dayac/Novel-X.git && novel-x.py run --10x --high-coverage
```

## Evaluation signals

- Output BAM is LongRanger-format with intact barcode (BX) tags usable by novel-x.py run with --10x
- Molecule coverage distribution per platform falls in the expected range when plotted against the NA12878 platform comparison
- stLFR conversion yields 10X-compatible reads accepted by the downstream pipeline without barcode-splitting errors
- Downstream Novel-X runs complete all stages (NRS insertion assembly, barcode list extraction, insertions reassembly, location on the reference) without failure at filter_target_contig

## Limitations

- Only stLFR technology provides paired reads with equal-length ends, which restricts which tools can run on additional datasets (e.g. Pamir)
- Novel-X requires a substantial number of distinct barcodes per insertion; small insertions often lack sufficient barcode coverage for anchor assembly
- There is a known problem on the filter_target_contig stage of the Novel-X pipeline
- SuperNova exposes no user-controllable parameters; its defaults are tuned for whole human genome diploid assembly

## Evidence

- [methods] The input for Novel-X is a reference genome (e.g. GRCh38): "The input for Novel-X is a reference genome (e.g. GRCh38)"
- [methods] For stLFR data, use this pipeline (github.com/BGI-Qingdao/stlfr2supernova_pipeline): "For stLFR data, use this [pipeline](https://github.com/BGI-Qingdao/stlfr2supernova_pipeline)"
- [methods] molecule coverage distribution of different synthetic long-read platforms on the NA12878 genome: "molecule coverage distribution of different synthetic long-read platforms on the NA12878 genome"
- [methods] only stLFR technology provides paired reads with equal length ends. In order to run Pamir on additional datasets: "only stLFR technology provides paired reads with equal length ends. In order to run Pamir on additional datasets"
- [readme] If you clone repository non-recursively Novel-X will not work.: "If you clone repository non-recursively Novel-X will not work. To fix this run from Novel-X folder:

```
git submodule update --init --recursive
```"
