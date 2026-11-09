---
name: variant-caller-combination-comparison
description: Decision support for choosing which SV caller pairing and minimum supporting-read
  threshold to configure in RetroInspector (or a similar nanopore TE-detection pipeline)
  when performance must be quantified against a curated truth set such as HGSVC.
when_to_use_negative:
- When the goal is TE detection performance of RetroInspector itself against external
  pipelines (PALMER, GraffiTE) rather than an internal caller-combination ablation
- When short-read data is the input — this comparison is grounded in nanopore long-read
  alignment against GRCh38
- When a quick production run is needed with the pipeline defaults, since the article
  already establishes cuteSV+Sniffles2 with at least 3 supporting reads as the default
  (and recommended) settings of RetroInspector
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: RetroInspector
  role: Snakemake pipeline whose caller combination and supporting-read threshold
    configuration this skill benchmarks
  repo: https://github.com/javiercguard/retroinspector
- name: minimap2
  role: Aligns basecalled nanopore reads against the GRCh38 reference genome
  repo: https://github.com/lh3/minimap2
- name: SVIM
  role: Structural-variant caller substituted for cuteSV in the Sniffles2+SVIM combination
  repo: https://github.com/eldariont/svim
- name: Sniffles2
  role: Structural-variant caller used in both tested combinations
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: cuteSV
  role: Structural-variant caller paired with Sniffles2 in the comparison baseline
  repo: https://github.com/tjiangHIT/cuteSV
- name: SAMtools
  role: Sorts and indexes minimap2 BAM alignments before variant calling
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:4269b6b9d72563420309a463cbc89f4ddffb477865a4c5f6293a090d4885f966
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/retroinspector
  repo_url: https://github.com/javiercguard/retroinspector
  tools:
  - RetroInspector
  - SAMtools
  - SVIM
  - Sniffles2
  - cuteSV
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# variant-caller-combination-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark structural-variant caller combinations (cuteSV+Sniffles2 versus Sniffles2+SVIM) for transposable-element insertion detection in nanopore data, filtering calls by a supporting-read threshold and scoring precision/recall/F1 against the HGSV truth set. In the RetroInspector study, Sniffles2+SVIM at 5 supporting reads reached 84.46% F1 versus 84.28% for cuteSV+Sniffles2.

## When to use

Decision support for choosing which SV caller pairing and minimum supporting-read threshold to configure in RetroInspector (or a similar nanopore TE-detection pipeline) when performance must be quantified against a curated truth set such as HGSVC.

## When NOT to use

- When the goal is TE detection performance of RetroInspector itself against external pipelines (PALMER, GraffiTE) rather than an internal caller-combination ablation
- When short-read data is the input — this comparison is grounded in nanopore long-read alignment against GRCh38
- When a quick production run is needed with the pipeline defaults, since the article already establishes cuteSV+Sniffles2 with at least 3 supporting reads as the default (and recommended) settings of RetroInspector

## Inputs

- Basecalled nanopore FASTQ/FASTA reads for HGSV samples HG00733, HG00514, NA19240
- GRCh38 (hg38) reference genome FASTA
- HGSVC three-sample TE insertion truth set (5,630 insertions)
- VCF callsets from SVIM, Sniffles2, and cuteSV

## Outputs

- One structural-variant VCF per caller (SVIM, Sniffles2, cuteSV)
- Strict-criterion (intersection) and lax callsets per caller combination at 3- and 5-supporting-read thresholds
- SURVIVOR match results at 100 bp (and supplementary 500 bp) distance margins
- Per-combination precision, recall, and F1-score table (e.g., Sniffles2+SVIM 84.46% F1 vs cuteSV+Sniffles2 84.28% F1 at 5 reads)

## How to apply

Align basecalled nanopore reads of the three HGSV samples (HG00733, HG00514, NA19240) against GRCh38 with minimap2, then sort and index the BAMs with samtools. Call structural variants independently with SVIM (substituted for cuteSV) and with Sniffles2, producing one VCF per caller, and retain the strict-criterion (intersection-of-callers) callsets for each combination. Filter both callsets by read evidence — compare the default and recommended 3 supporting reads against a stricter 5-supporting-read threshold. Match called TE insertions against the HGSVC truth set (5,630 reported insertions) using SURVIVOR with a 100 bp distance margin (plus a supplementary 500 bp comparison), then compute precision, recall, and F1 per combination. Validate the comparison by checking that the reproduced table reports Sniffles2+SVIM at roughly 84.46% F1 and cuteSV+Sniffles2 at roughly 84.28% F1 (precision 82.25%, recall 86.41%) at the 5-read threshold, consistent with the article's Table 1.

## Related tools

- **RetroInspector** (Snakemake pipeline whose caller combination and supporting-read threshold configuration this skill benchmarks) — https://github.com/javiercguard/retroinspector
- **minimap2** (Aligns basecalled nanopore reads against the GRCh38 reference genome) — https://github.com/lh3/minimap2
- **SVIM** (Structural-variant caller substituted for cuteSV in the Sniffles2+SVIM combination) — https://github.com/eldariont/svim
- **Sniffles2** (Structural-variant caller used in both tested combinations) — https://github.com/fritzsedlazeck/Sniffles
- **cuteSV** (Structural-variant caller paired with Sniffles2 in the comparison baseline) — https://github.com/tjiangHIT/cuteSV
- **SAMtools** (Sorts and indexes minimap2 BAM alignments before variant calling)

## Examples

```
snakemake --use-conda -c 8 --config referenceGenome=/path/hg38.fa outputPath=/path/results fastq_directory=/path/hgsv_samples
```

## Evaluation signals

- At the 5-supporting-read threshold, cuteSV+Sniffles2 strict callset should reproduce precision 80.90% (TE; 85.63% all), recall 87.10% (16.34%), F1 83.89%
- Sniffles2+SVIM F1 should land at approximately 84.46% and cuteSV+Sniffles2 at approximately 84.28% at the 5-read threshold, matching Table 1
- Callset matching against the HGSVC truth set should use SURVIVOR with a 100 bp distance margin, with a supplementary 500 bp comparison showing consistent ranking
- The intersection (strict) callsets should be derived independently for both combinations so the only differing variable is the caller pairing
- Reported F1 differences should be marginal (within ~0.2 percentage points), consistent with the article's note that the three combinations performed similarly

## Limitations

- xTea was not used in the comparative evaluation, so caller-combination conclusions exclude it
- HGSVC truth-set data was basecalled with an older basecaller, so sequences may be imperfect, particularly for homopolymers
- No software version numbers are stated for cuteSV, Sniffles2, or SVIM, complicating exact reproduction; settings must be consulted at the pipeline repository's template configuration file

## Evidence
