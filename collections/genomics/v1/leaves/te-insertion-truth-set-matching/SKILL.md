---
name: te-insertion-truth-set-matching
description: Use when benchmarking a TE/SV insertion caller's VCF output against a
  curated truth set and you need precision, recall, and F1 computed from position-matched
  variants at a defined base-pair margin.
when_to_use_negative:
- The callsets are not insertion callsets in a matchable variant format (e.g., raw
  FASTQ or unassembled contigs) — caller output must first be converted to VCF
- The goal is genotyping concordance rather than insertion detection benchmarking
  (the article evaluated HGSV2 genotyping separately from the truth-set matching)
- Tools were run with incomparable compute resources — timing comparisons are invalid
  (the paper excluded GraffiTE because it used 96 threads when assigned 32)
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: SURVIVOR
  role: Matches each tool's called variants against the truth set using 100 bp and
    500 bp distance margins to derive true positives
- name: RetroInspector
  role: Pipeline under benchmark; generates the strict cuteSV+Sniffles2 insertion
    callset (default minimum read support of 3) that serves as the reference point
  repo: https://github.com/javiercguard/retroinspector
- name: PALMER
  role: Comparative benchmark tool run once per TE type, filtered to variants with
    at least 1 high-confidence read and 2 supporting reads before matching
- name: GraffiTE
  role: Comparative benchmark tool using Dfam 3.8 converted with FamDB 1.0.2 and the
    'pangenie' graph method on ONT long reads
  repo: https://github.com/Dfam-consortium/FamDB
- name: cuteSV
  role: Internal variant caller whose output (with Sniffles2) defines the strict RetroInspector
    callset evaluated against the truth set
  repo: https://github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Internal variant caller whose output (with cuteSV) defines the strict RetroInspector
    callset evaluated against the truth set
  repo: https://github.com/fritzsedlazeck/Sniffles
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:47606ed8c9236c3dccfba2c72ebf378d4fa6621391524e9dbd6bc3eb14ce9c75
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
  - FamDB toolkit
  - GNU parallel
  - GraffiTE
  - PALMER
  - PanGenIE
  - RetroInspector
  - SURVIVOR
  - Sniffles2
  - cuteSV
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# te-insertion-truth-set-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Matches TE insertion callsets (VCF) from long-read detection tools against a curated truth set with SURVIVOR at a strict 100 bp and a permissive 500 bp distance margin, then computes precision, recall, and F1 per tool, as done to benchmark RetroInspector against PALMER and GraffiTE on the HGSV nanopore samples.

## When to use

Use when benchmarking a TE/SV insertion caller's VCF output against a curated truth set and you need precision, recall, and F1 computed from position-matched variants at a defined base-pair margin.

## When NOT to use

- The callsets are not insertion callsets in a matchable variant format (e.g., raw FASTQ or unassembled contigs) — caller output must first be converted to VCF
- The goal is genotyping concordance rather than insertion detection benchmarking (the article evaluated HGSV2 genotyping separately from the truth-set matching)
- Tools were run with incomparable compute resources — timing comparisons are invalid (the paper excluded GraffiTE because it used 96 threads when assigned 32)

## Inputs

- VCF callsets of TE insertions from each benchmarked tool (RetroInspector, PALMER-converted VCF, GraffiTE)
- Truth-set VCF of comprehensively characterized TE insertions (HGSVC three-sample set, 5,630 insertions)
- Per-tool runtime statistics (e.g., RetroInspector --stats flag)
- SURVIVOR for interval matching at 100 bp and 500 bp margins

## Outputs

- Match counts (true positives) per tool per margin from SURVIVOR comparisons
- Precision, recall, and F1-score table per tool (e.g., RetroInspector 80.90%/87.10%/83.89%; PALMER 71.77%/52.61%/60.72%; GraffiTE 32.33%/96.18%/48.40%)
- Per-sample runtime comparison table

## How to apply

Detect insertions with the candidate tool(s) (e.g., RetroInspector's Snakemake pipeline using minimap2 alignment against GRCh38 and cuteSV+Sniffles2 with the default minimum support of 3 reads), then check the presence of each tool's variants in the truth set with SURVIVOR using a 100 bp distance margin, repeating the comparison at 500 bp as the more permissive setting. Count SURVIVOR matches as true positives against the truth set (here 5,630 reported TE insertions across three HGSV samples) and compute precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 per tool. Apply consistent filtering to each tool before matching (e.g., for PALMER, keep variants with at least 1 high-confidence read and 2 supporting reads, i.e., 10% of depth coverage rounded down). Present the strict cuteSV+Sniffles2 callset as the reference point (precision 80.90%, recall 87.10%, F1 83.89%) and report runtime separately, excluding any tool that ran on a different thread allocation (GraffiTE used 96 threads when assigned 32).

## Related tools

- **SURVIVOR** (Matches each tool's called variants against the truth set using 100 bp and 500 bp distance margins to derive true positives)
- **RetroInspector** (Pipeline under benchmark; generates the strict cuteSV+Sniffles2 insertion callset (default minimum read support of 3) that serves as the reference point) — https://github.com/javiercguard/retroinspector
- **PALMER** (Comparative benchmark tool run once per TE type, filtered to variants with at least 1 high-confidence read and 2 supporting reads before matching)
- **GraffiTE** (Comparative benchmark tool using Dfam 3.8 converted with FamDB 1.0.2 and the 'pangenie' graph method on ONT long reads) — https://github.com/Dfam-consortium/FamDB
- **cuteSV** (Internal variant caller whose output (with Sniffles2) defines the strict RetroInspector callset evaluated against the truth set) — https://github.com/tjiangHIT/cuteSV
- **Sniffles2** (Internal variant caller whose output (with cuteSV) defines the strict RetroInspector callset evaluated against the truth set) — https://github.com/fritzsedlazeck/Sniffles

## Examples

```
SURVIVOR merge tool_calls.vcf truth_set.vcf 100 1 0 0 0 0
```

## Evaluation signals

- The strict cuteSV+Sniffles2 callset reproduces ~80.9% precision / 87.1% recall / 83.9% F1 against the 5,630-insertion HGSVC truth set
- PALMER's reproduced metrics fall below RetroInspector's F1 of 83.89%, with a maximum F1 of 60.72% (precision 71.77%, recall 52.61%)
- GraffiTE shows the expected asymmetric profile: high recall (96.18%) but low precision (32.33%), F1 48.40%
- Both margins were applied: strict 100 bp and permissive 500 bp SURVIVOR comparisons were run and reported
- Runtime table reflects per-sample ordering relative to PALMER, with GraffiTE excluded from timing

## Limitations

- An incomplete truth set can depress apparent precision — the article notes PALMER's low precision 'could stem from an incomplete truth set'
- Truth-set-derived performance depends on data age/basecalling: 'the publicly available HGSVC data is a few years old, meaning it was basecalled with an older basecaller, and even after reassembly, sequences may be imperfect, particularly for homopolymers'
- Runtime comparisons are only valid under matched thread allocations; mismatched resources (GraffiTE at 96 vs 32 threads) must be excluded

## Evidence

- [methods] Check the presence of each tool's variants in the truth set with SURVIVOR using a distance margin of 100 bp, and repeat the comparison with a 500 bp limit as the more permissive setting.: [evidence span withheld — anchor status: not_found]
- [methods] Compute precision, recall, and F1 per tool from the SURVIVOR matches (strict cuteSV+Sniffles2 reference point: precision 80.90%, recall 87.10%, F1 83.89%): [evidence span withheld — anchor status: not_found]
- [results] PALMER demonstrated lower performance than RetroInspector (F1 83.89%), achieving a maximum F1-score of 60.72%, with precision of 71.77% and recall of 52.61% (Table 1): [evidence span withheld — anchor status: not_found]
