---
name: polya-tail-detection-in-inserted-sequences
description: Use when filtering merged long-read TE insertion callsets (e.g., cuteSV-Sniffles2)
  by retrotransposition hallmarks, or when verifying that candidate insertions show
  terminal polyA tails before accepting them as true retrotransposition events.
when_to_use_negative:
- Input is raw nanopore FASTQ reads rather than reassembled insertion sequences —
  the tail detection operates on reassembled insertions, not on individual reads
- The callset targets non-retrotransposon structural variants (deletions, inversions),
  where polyA tails and L1 motifs are not expected filtering features
- Precision is already sufficient without filtering and any recall loss from hallmark
  filtering is unacceptable for the analysis goal
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RetroInspector
  role: Snakemake pipeline that reassembles inserted regions from supporting reads
    and annotates retrotransposition hallmarks (polyA tails, L1 endonuclease motif)
    used for call filtering
  repo: https://github.com/javiercguard/retroinspector
- name: RapidFuzz
  role: Fuzzy text search (initial similarity ratio 60%) for the L1 endonuclease consensus
    motif TTTTT/AA and its reverse complement on both strands of the inserted sequence
- name: pROC
  role: Used in evaluating precision/recall of the hallmark-filtered callset
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:ef7da53a3a918d48b6f6f277fbfa9a969bed31637269a329a5208ccbd37447c1
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
  - RapidFuzz
  - RetroInspector
  - pROC
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# polya-tail-detection-in-inserted-sequences

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects polyA/polyT tails at the termini of reassembled TE insertion sequences (and the L1 endonuclease target motif TTTTT/AA) as retrotransposition hallmarks, which RetroInspector uses to filter insertion calls and raise precision above 90% at 5 and 7 supporting reads.

## When to use

Use when filtering merged long-read TE insertion callsets (e.g., cuteSV-Sniffles2) by retrotransposition hallmarks, or when verifying that candidate insertions show terminal polyA tails before accepting them as true retrotransposition events.

## When NOT to use

- Input is raw nanopore FASTQ reads rather than reassembled insertion sequences — the tail detection operates on reassembled insertions, not on individual reads
- The callset targets non-retrotransposon structural variants (deletions, inversions), where polyA tails and L1 motifs are not expected filtering features
- Precision is already sufficient without filtering and any recall loss from hallmark filtering is unacceptable for the analysis goal

## Inputs

- Reassembled inserted sequences from supporting reads (RetroInspector reassembly step)
- GRCh38 reference sequence ±20 bp around each insertion site
- cuteSV-Sniffles2 merged insertion callset (VCF) with supporting-read counts

## Outputs

- Per-insertion hallmark annotations (polyA tail presence/position, L1 motif TTTTT/AA hit)
- Filtered TE insertion callset retaining only calls with retrotransposition hallmarks
- Precision/recall metrics with and without hallmark filters at 5 and 7 supporting reads

## How to apply

Retrieve each reassembled inserted sequence from the cuteSV-Sniffles2 merged insertion callset together with the GRCh38 reference sequence ±20 bp around the insertion site. Detect polyA/polyT tails by recovering adenine/thymine runs on the inserted sequence, merging candidates separated by gaps of ≤2 bp, and selecting the candidate closest to the sequence end (or start for the opposite strand); require the tail to start within the last (or first) 10% of the inserted sequence. Optionally combine this with a RapidFuzz fuzzy search (initial similarity ratio 60%) for the L1 endonuclease consensus motif TTTTT/AA and its reverse complement TT/AAAAA on both strands, keeping the hit closest to the insertion coordinates when multiple matches occur. Recompute precision and recall of the callset with and without the hallmark filters at supporting-read thresholds of 5 and 7 reads, and test hallmark differences between true and false positives with a χ-squared test. The filter is successful when precision rises above 90% at both thresholds with acceptable (reported) recall loss, as in the paper's 90.38% and 90.94% precision at 5 and 7 reads.

## Related tools

- **RetroInspector** (Snakemake pipeline that reassembles inserted regions from supporting reads and annotates retrotransposition hallmarks (polyA tails, L1 endonuclease motif) used for call filtering) — https://github.com/javiercguard/retroinspector
- **RapidFuzz** (Fuzzy text search (initial similarity ratio 60%) for the L1 endonuclease consensus motif TTTTT/AA and its reverse complement on both strands of the inserted sequence)
- **pROC** (Used in evaluating precision/recall of the hallmark-filtered callset)

## Examples

```
snakemake --use-conda -np --config referenceGenome="/home/yourname/hg38.fa" outputPath="/home/yourname/experiment_results" fastq_directory="/home/yourname/data/samples"
```

## Evaluation signals

- Precision of the filtered TE insertion callset exceeds 90% at supporting-read thresholds of 5 and 7 reads (paper: 90.38% and 90.94%)
- PolyA tail candidate starts within the last (or first) 10% of the inserted sequence, with merged A/T runs separated by ≤2 bp gaps
- χ-squared test reports a significant difference in hallmark frequency between true and false positives (paper: χ², p < 2e−16)
- Filtering with the L1 motif (TTTTT/AA and reverse complement TT/AAAAA, ≥60% fuzzy similarity) keeps the hit closest to the insertion coordinates when multiple matches occur
- Recall loss from filtering is measured and reported rather than silently accepted

## Limitations

- HGSVC truth-set data were basecalled with an older basecaller, so even after reassembly, sequences may be imperfect, particularly for homopolymers — this directly affects polyA/polyT run detection
- Hallmark discrimination is imperfect; the proportions differed significantly but the filter trades recall for precision
- Exact statistical test details (χ² implementation) and some comparison parameters are not fully specified in the paper

## Evidence

- [results] Hallmark filter features: [evidence span withheld — anchor status: not_found]
- [results] Hallmark filter precision gains: [evidence span withheld — anchor status: not_found]
- [results] Statistical significance of hallmark discrimination: [evidence span withheld — anchor status: not_found]
